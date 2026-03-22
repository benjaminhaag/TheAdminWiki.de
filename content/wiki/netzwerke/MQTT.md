---
date: '2026-03-22T15:56:01Z'
draft: true
title: 'MQTT'
tags: ["netzwerke","MQTT"]
---

[**Message Queuing Telemetry Transfort (MQTT)**](https://docs.oasis-open.org/mqtt/mqtt/v5.0/mqtt-v5.0.html) ist ein Machine-To-Machine (M2M) Protokoll zur übertragung von Telemetriedaten in IoT und IIoT Netzwerken.

Im Gegensatz zu [CoAP](./CoAP.md) folgt es nicht dem Request-Response mit Clients und Servern, sondern einem Publisher-Subscriber Modell. Nachrichten werden vom Publisher an einen **Message Broker** gesendet der dann sicherstell, dass sie an alle Subscriber weitergeleitet wird.

## Protocol

MQTT setzt eine Fehlertollerante Netzworkverbindung wie **TCP/IP**, **TLS** oder **WebSockets** ([RFC 6455](https://www.rfc-editor.org/rfc/rfc6455.html)) voraus und ist somit schwergewichtiger als zum Beispiel [CoAP](./CoAP.md), aber deutlich leichter als REST. Es nutzt die Standartports **1883** über TCP/IP, **8883** über TLS, bwz. **80/443** über WebSockets.
**MQTT for Sensor Networks (MQTT-SN)** bietet eine Möglichkeit MQTT über alternative Netzworke wie **Zigbee** zu betreiben.


### Topics

Clients eine dauerhafte Verbindung zum Message Broker auf. Sie haben dann die Möglichkeit **Topics** zu abonieren (subscribe), bzw. zu veröffentlichen (publish).

Bei einer **Subscription** sendet der Client eine Nachricht an den Broker mit dem Topic, über welches er in Zukunft informiert werden will. Der Broker speichert die Subscription.

Wenn ein Client einen Wert zu einem Topic **Publischen** möchte, sendert er eine Publish Nachricht an den Broker mit dem topic und dem Payload. Der Broker sendet diese Nachricht dann an alle Clients, die das Topic aboniert haben.

### Nachrichtenformat