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
Q40. MQTT QoS Levels
QoS 0 (At most once): "Fire and forget." The message is sent once with no acknowledgment or retry. It may be lost if the network drops it, but there is minimal overhead.
QoS 1 (At least once): The sender keeps the message until it receives a PUBACK from the receiver. If the ACK is lost, the message is resent, so the message is guaranteed to arrive — but it might arrive more than once (duplicates possible).
QoS 2 (Exactly once): Uses a 4-step handshake (PUBREC, PUBREL, PUBCOMP) to guarantee the message is delivered exactly one time, no duplicates, no loss — but with the highest latency and overhead of the three.
For an IoT water level monitoring system: I would choose QoS 1. Losing a single water-level reading (as QoS 0 might) is risky in a flood-warning context — a missed CRITICAL reading could have real consequences. However, QoS 2's extra handshake latency isn't worth it here, because occasional duplicate readings are harmless (the next reading a few seconds later will simply confirm or update the water level) — the application can tolerate duplicates but not silent loss. QoS 1 gives the best balance of reliability and speed for this safety-relevant but duplicate-tolerant use case.
