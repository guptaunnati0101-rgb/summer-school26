Scenario: An ESP8266 connects to public Wi-Fi with hardcoded SSID/password, uses plain HTTP, publishes to a public unauthenticated MQTT broker, and never validates incoming commands.
Five vulnerabilities and their risks:
Hardcoded SSID/password in source code. Anyone with access to the code (e.g., a public GitHub repo) obtains the Wi-Fi credentials directly, potentially compromising the entire network the device sits on, not just the device itself.
Unencrypted HTTP instead of HTTPS. All data — sensor readings, any tokens, command payloads — travels in plaintext. An attacker on the same public Wi-Fi network can trivially sniff this traffic (e.g., with Wireshark) or perform a man-in-the-middle attack to alter data in transit.
Publishing to a public MQTT broker with no authentication. Anyone who knows (or guesses) the broker address and topic name can subscribe to the device's data feed, read sensitive sensor data, or — worse — publish spoofed messages on the same topic, corrupting or hijacking the data stream.
No validation of incoming commands. If the device blindly executes any command it receives on its subscribed topic (e.g., "unlock door", "activate relay"), an attacker who reaches that public broker can issue arbitrary commands to the device with zero authorization checks — a direct path to unauthorized physical actuation.
No firmware/update integrity checking (implied by "never validates"). If the device also accepts firmware updates without checking a signature or checksum, an attacker could push malicious firmware disguised as a legitimate update, achieving full device takeover.
Corrected code for two vulnerabilities:
(1) Moving credentials out of source code, into a git-ignored config file:
cpp
// config.h  (added to .gitignore — never committed)
#define WIFI_SSID "your_network_name"
#define WIFI_PASS "your_secure_password"
#define MQTT_USER "device_client_01"
#define MQTT_PASS "strong_broker_password"


// main.ino
#include "config.h"
#include <ESP8266WiFi.h>


void setup() {
  WiFi.begin(WIFI_SSID, WIFI_PASS); // credentials no longer visible in shared code
}
(3) Requiring authentication on MQTT connect, and (4) validating commands before acting on them:
cpp
#include <PubSubClient.h>
WiFiClientSecure espClient; // TLS-capable client instead of plain WiFiClient
PubSubClient mqttClient(espClient);


void connectMQTT() {
  // Authenticated, encrypted connection instead of anonymous plaintext
  while (!mqttClient.connected()) {
    mqttClient.connect("esp8266_client_01", MQTT_USER, MQTT_PASS);
  }
}


void callback(char* topic, byte* payload, unsigned int length) {
  String message;
  for (unsigned int i = 0; i < length; i++) message += (char)payload[i];


  // Validate command against an allow-list before ever acting on it
  const String allowedCommands[] = {"STATUS", "PING", "RELAY_ON", "RELAY_OFF"};
  bool valid = false;
  for (String cmd : allowedCommands) {
    if (message == cmd) { valid = true; break; }
  }


  if (valid) {
    executeCommand(message);
  } else {
    Serial.println("REJECTED unrecognized command: " + message);
  }
}
