Approach: Read PIR motion state, gate alerts by a configurable time window (set via Serial command), escalate through three alarm stages, flash an LED sequence, and (optionally) notify via a Telegram bot over Wi-Fi.
cpp
#define PIR_PIN      27
#define POT_PIN      34   // sensitivity potentiometer
#define BUZZER_PIN   14
#define LED1 25
#define LED2 26
#define LED3 33


int startHour = 22, endHour = 6; // default night-time window
unsigned long motionStart = 0;
bool motionActive = false;
int stage = 0; // 0=none, 1=warning, 2=alarm, 3=urgent


int currentHour = 23; // replace with RTC module or NTP time in production


void setup() {
  Serial.begin(115200);
  pinMode(PIR_PIN, INPUT);
  pinMode(BUZZER_PIN, OUTPUT);
  pinMode(LED1, OUTPUT); pinMode(LED2, OUTPUT); pinMode(LED3, OUTPUT);
  Serial.println("Ready. Send: SET_HOURS <start> <end>");
}


bool withinWindow() {
  if (startHour < endHour) {
    return currentHour >= startHour && currentHour < endHour;
  } else {
    return currentHour >= startHour || currentHour < endHour;
  }
}


void handleSerialCommand() {
  if (Serial.available()) {
    String cmd = Serial.readStringUntil('\n');
    cmd.trim();
    if (cmd.startsWith("SET_HOURS")) {
      int sh, eh;
      sscanf(cmd.c_str(), "SET_HOURS %d %d", &sh, &eh);
      startHour = sh; endHour = eh;
      Serial.println("Window updated: " + String(startHour) + " - " + String(endHour));
    }
  }
}


void flashSequence() {
  digitalWrite(LED1, HIGH); delay(100); digitalWrite(LED1, LOW);
  digitalWrite(LED2, HIGH); delay(100); digitalWrite(LED2, LOW);
  digitalWrite(LED3, HIGH); delay(100); digitalWrite(LED3, LOW);
}


void loop() {
  handleSerialCommand();


  bool motionDetected = digitalRead(PIR_PIN) == HIGH;


  if (motionDetected && withinWindow()) {
    if (!motionActive) {
      motionActive = true;
      motionStart = millis();
      stage = 1;
      Serial.println("MOTION @ " + String(millis()) + "ms - WARNING stage");
    }


    unsigned long elapsed = millis() - motionStart;
    if (elapsed > 10000 && stage < 3) {
      stage = 3;
      Serial.println("ESCALATED - URGENT stage");
    } else if (elapsed > 4000 && stage < 2) {
      stage = 2;
      Serial.println("ESCALATED - ALARM stage");
    }


    flashSequence();
    if (stage == 1) tone(BUZZER_PIN, 800, 150);
    else if (stage == 2) tone(BUZZER_PIN, 1500, 150);
    else tone(BUZZER_PIN, 3000, 300);


    // ESP32 Telegram bot notification (requires Wi-Fi + bot token in config.h)
    // sendTelegramAlert("Motion detected! Stage: " + String(stage));
  } else if (!motionDetected) {
    motionActive = false;
    stage = 0;
    noTone(BUZZER_PIN);
  }
}
