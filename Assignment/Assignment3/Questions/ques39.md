An IoT gateway is a hardware/software bridge that sits between resource-constrained sensor nodes and the internet/cloud. It aggregates data from multiple sensor nodes (which often use short-range, low-power protocols the internet doesn't understand directly), performs protocol translation, local buffering, and sometimes edge processing/filtering, before forwarding data onward using internet-compatible protocols.
Data flow diagram:
[IoT Sensor Nodes]  --(BLE / Zigbee / LoRa)-->  [Gateway]  --(Wi-Fi / Ethernet / Cellular)-->  [Cloud]  --(HTTPS / WebSocket)-->  [User Application]
     (e.g., ESP32 +          Local protocol         (e.g., Raspberry Pi         MQTT/HTTPS           (AWS IoT Core,          Dashboard / Mobile App
      DHT11, MQ-2)            translation            or ESP32 as edge          to broker/API         Firebase, ThingSpeak)   (Grafana, custom app)
                                                       hub)
Typical protocols at each layer:
Sensor Node → Gateway: BLE, Zigbee, LoRa, or simple UART/I2C/analog for very short-range wired sensor-to-microcontroller links.
Gateway → Cloud: MQTT (over TLS) is most common for telemetry; HTTPS/REST for occasional API calls or firmware updates.
Cloud → User Application: HTTPS REST APIs for on-demand data, WebSockets or MQTT-over-WebSocket for real-time dashboard updates.
