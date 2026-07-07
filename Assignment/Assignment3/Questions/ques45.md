void setup() {
  Serial.begin(115200);
  pinMode(TRIG_PIN, OUTPUT);
  pinMode(ECHO_PIN, INPUT);
  pinMode(LED_GREEN, OUTPUT);
  pinMode(LED_YELLOW, OUTPUT);
  pinMode(LED_RED, OUTPUT);
  pinMode(BUZZER_PIN, OUTPUT);
  for (int i = 0; i < SAMPLES; i++) readings[i] = 100; // assume normal clearancApproach: Monitor temperature/humidity (DHT11) and light (LDR with running average), and drive three relays (heater, fan, grow light) using bang-bang control with hysteresis to avoid rapid relay chattering at the boundary.
cpp
#include <DHT.h>
#include <LiquidCrystal.h>


#define DHT_PIN 4
#define DHT_TYPE DHT11
#define LDR_PIN 34
#define HEATER_RELAY 25
#define FAN_RELAY    26
#define LIGHT_RELAY  27


DHT dht(DHT_PIN, DHT_TYPE);
LiquidCrystal lcd(12, 11, 5, 4, 3, 2);


const float TEMP_LOW = 18.0, TEMP_HIGH = 22.0, HYSTERESIS = 1.0;
const float HUM_HIGH = 70.0;


const int SAMPLES = 10;
int ldrReadings[SAMPLES];
int sampleIndex = 0;
long ldrRunningTotal = 0;


unsigned long lastCycle = 0;
int displayPage = 0;


void setup() {
  Serial.begin(115200);
  dht.begin();
  lcd.begin(16, 2);
  pinMode(HEATER_RELAY, OUTPUT);
  pinMode(FAN_RELAY, OUTPUT);
  pinMode(LIGHT_RELAY, OUTPUT);
  for (int i = 0; i < SAMPLES; i++) ldrReadings[i] = 0;
}


int updateLDRAverage() {
  ldrRunningTotal -= ldrReadings[sampleIndex];
  ldrReadings[sampleIndex] = analogRead(LDR_PIN);
  ldrRunningTotal += ldrReadings[sampleIndex];
  sampleIndex = (sampleIndex + 1) % SAMPLES;
  return ldrRunningTotal / SAMPLES;
}


void controlClimate(float temp, float hum) {
  static bool heaterOn = false;
  if (temp < TEMP_LOW - HYSTERESIS) heaterOn = true;
  else if (temp > TEMP_LOW + HYSTERESIS) heaterOn = false;
  digitalWrite(HEATER_RELAY, heaterOn ? HIGH : LOW);


  static bool fanOn = false;
  if (temp > TEMP_HIGH + HYSTERESIS || hum > HUM_HIGH + HYSTERESIS) fanOn = true;
  else if (temp < TEMP_HIGH - HYSTERESIS && hum < HUM_HIGH - HYSTERESIS) fanOn = false;
  digitalWrite(FAN_RELAY, fanOn ? HIGH : LOW);
}


void controlLight(int avgLdr) {
  static bool lightOn = false;
  const int LIGHT_THRESHOLD = 400;
  if (avgLdr < LIGHT_THRESHOLD - 30) lightOn = true;
  else if (avgLdr > LIGHT_THRESHOLD + 30) lightOn = false;
  digitalWrite(LIGHT_RELAY, lightOn ? HIGH : LOW);
}


void loop() {
  float temp = dht.readTemperature();
  float hum  = dht.readHumidity();
  int avgLdr = updateLDRAverage();


  controlClimate(temp, hum);
  controlLight(avgLdr);


  if (millis() - lastCycle >= 3000) {
    lastCycle = millis();
    lcd.clear();
    switch (displayPage) {
      case 0: lcd.print("Temp: " + String(temp) + "C"); break;
      case 1: lcd.print("Hum: " + String(hum) + "%"); break;
      case 2: lcd.print("Light: " + String(avgLdr)); break;
    }
    displayPage = (displayPage + 1) % 3;


    Serial.print("T="); Serial.print(temp);
    Serial.print(" H="); Serial.print(hum);
    Serial.print(" L="); Serial.println(avgLdr);


    // BONUS: MQTT publish (requires Wi-Fi + PubSubClient library)
    // mqttClient.publish("greenhouse/temp", String(temp).c_str());
  }
}
