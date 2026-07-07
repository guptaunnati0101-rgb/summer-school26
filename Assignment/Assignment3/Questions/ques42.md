Approach: The MQ-2 gas sensor gives an analog reading proportional to smoke/CO concentration. We read this alongside DHT11 temperature/humidity, classify the reading into SAFE / MODERATE / DANGER bands using two thresholds, drive an RGB LED and a buzzer with distinct tones per level, and log everything to Serial in CSV format so it survives without internet connectivity (fully local logic — no cloud dependency for core function).
cpp
#include <DHT.h>


#define MQ2_PIN     34      // analog input
#define DHT_PIN     4
#define DHT_TYPE    DHT11
#define LED_RED     25
#define LED_GREEN   26
#define LED_BLUE    27
#define BUZZER_PIN  14


// Thresholds (analog units, calibrate against clean air baseline)
const int MODERATE_THRESHOLD = 350;
const int DANGER_THRESHOLD   = 600;


DHT dht(DHT_PIN, DHT_TYPE);
unsigned long lastRead = 0;
const unsigned long INTERVAL = 1000; // 1 second


void setup() {
  Serial.begin(115200);
  pinMode(LED_RED, OUTPUT);
  pinMode(LED_GREEN, OUTPUT);
  pinMode(LED_BLUE, OUTPUT);
  pinMode(BUZZER_PIN, OUTPUT);
  dht.begin();
  Serial.println("Timestamp,MQ-2 Value,DHT11 Temp,DHT11 Humidity,Severity Level");
}


void setRGB(bool r, bool g, bool b) {
  digitalWrite(LED_RED, r);
  digitalWrite(LED_GREEN, g);
  digitalWrite(LED_BLUE, b);
}


void soundAlarm(String level) {
  if (level == "SAFE") {
    noTone(BUZZER_PIN);
  } else if (level == "MODERATE") {
    tone(BUZZER_PIN, 1000, 200);
  } else {
    tone(BUZZER_PIN, 2500, 500);
  }
}


String formatTimestamp() {
  unsigned long totalSec = millis() / 1000;
  int mm = (totalSec / 60) % 60;
  int ss = totalSec % 60;
  char buf[6];
  sprintf(buf, "%02d:%02d", mm, ss);
  return String(buf);
}


void loop() {
  if (millis() - lastRead >= INTERVAL) {
    lastRead = millis();


    int gasValue = analogRead(MQ2_PIN);
    float temp = dht.readTemperature();
    float hum  = dht.readHumidity();


    String severity;
    if (gasValue >= DANGER_THRESHOLD) {
      severity = "DANGER";
      setRGB(true, false, false);
    } else if (gasValue >= MODERATE_THRESHOLD) {
      severity = "MODERATE";
      setRGB(false, false, true);
      digitalWrite(LED_GREEN, HIGH);
    } else {
      severity = "SAFE";
      setRGB(false, true, false);
    }


    soundAlarm(severity);


    Serial.print(formatTimestamp()); Serial.print(",");
    Serial.print(gasValue); Serial.print(" ppm,");
    Serial.print(temp); Serial.print("C,");
    Serial.print(hum); Serial.print("%,");
    Serial.println(severity);


    // Bonus: Bluetooth alert (via HC-05/HC-06 on Serial2 on ESP32)
    // Serial2.println(severity + ": Gas=" + String(gasValue));
  }
}
