Q48. Design Challenge: IoT for Jammu Smart City (500–700 words)
Selected systems: Street Lighting, Traffic Flow, Waste Bin Fill Levels
1. Smart Street Lighting Jammu's street lighting currently runs on fixed timers, wasting energy on empty roads late at night and under-lighting during heavy fog or early monsoon darkness. An IoT-based adaptive lighting system would use an LDR (ambient light sensor) paired with a PIR motion sensor at each pole to detect approaching pedestrians or vehicles and brighten LEDs only when needed, dimming to a low standby level otherwise.
Sensors: LDR (ambient light), PIR (motion presence).
Microcontroller/connectivity: ESP32 (Wi-Fi) at each pole cluster, or a LoRa module where poles are far from Wi-Fi coverage, reporting to a central gateway.
Data flow: LDR/PIR to ESP32 (local dimming decision) to LoRa/Wi-Fi to Cloud (fault/energy dashboard) to Municipal Dashboard (map view of lit/unlit/faulty poles).
Estimated cost per node: ESP32 (Rs.450) + LDR (Rs.20) + PIR (Rs.80) + driver circuit (Rs.150) is approximately Rs.700–900 per pole, excluding the LED fixture itself.
2. Traffic Flow Monitoring Jammu's key junctions suffer congestion with no adaptive signal timing. An ultrasonic/IR vehicle-counting array at each approach lane can estimate queue length and vehicle throughput, feeding a signal-timing optimization service.
Sensors: IR break-beam or ultrasonic (HC-SR04) sensors per lane, or a low-cost camera module with on-device object counting for busier junctions.
Microcontroller/connectivity: ESP32-CAM (where vision is used) or ESP32 with ultrasonic sensors, connected via 4G/LTE for junctions outside stable Wi-Fi range, or Wi-Fi where municipal networks exist.
Data flow: Sensor array to ESP32 (vehicle count/queue estimate) to MQTT over 4G/Wi-Fi to Cloud traffic-analytics service to Traffic Control Dashboard (adjusts signal timing recommendations) and public traffic app.
Estimated cost per node: ESP32-CAM (Rs.1,200) or ESP32 + 4x HC-SR04 (Rs.450+Rs.400) + 4G module (Rs.900) is approximately Rs.1,500–2,500 per junction lane set.
3. Waste Bin Fill-Level Monitoring Municipal waste collection in Jammu often runs on a fixed schedule regardless of actual bin fill level, leading to overflowing bins in high-traffic areas and wasted collection trips to near-empty bins elsewhere. Mounting an ultrasonic sensor inside each public bin lid to measure fill height solves this.
Sensors: HC-SR04 ultrasonic sensor (mounted at the lid, facing down into the bin).
Microcontroller/connectivity: ESP8266 (low-cost, low-power) with deep-sleep between readings (e.g., hourly), reporting over Wi-Fi where available or LoRa in outlying areas, since bins don't need constant connectivity.
Data flow: HC-SR04 to ESP8266 (fill % calculation, deep-sleep cycle) to LoRa/Wi-Fi to Cloud (MQTT ingestion) to Waste Management Dashboard (route optimization for collection trucks, prioritizing bins over 80% full).
Estimated cost per node: ESP8266 (Rs.250) + HC-SR04 (Rs.80) + battery/solar trickle charger (Rs.300) is approximately Rs.600–700 per bin.
Common architecture: All three systems funnel into a shared MQTT broker hosted on a municipal cloud instance, which routes topic-specific data (streetlight/#, traffic/#, wastebin/#) to a unified city-operations dashboard (e.g., Grafana or a custom Node-RED interface). This shared backbone reduces integration cost, lets the city add new sensor categories later without redesigning the pipeline, and keeps per-node hardware cost low (Rs.600–2,500 depending on system) — a realistic, incrementally deployable path to a Jammu smart city pilot.

