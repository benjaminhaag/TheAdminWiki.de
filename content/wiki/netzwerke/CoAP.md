---
date: '2026-03-22T13:02:12Z'
draft: false
title: 'CoAP'
image: "/images/coap.webp"
tags: ["netzwerke","CoAP"]
---

**CoAP (Constrained Application Protocol)** [RFC 7252](https://datatracker.ietf.org/doc/html/rfc7252) ist ein Protokoll was im Wesentlichen REST sehr ähnlich ist und ähnliche Ziele verfolgt. Im Gegensatz zu HTTP wurde CoAP mit dem Ziel entwickelt auf **"eingeschränkten Geräten" (constrained devices)** zu laufen. Ein typisches Beispiel und Hauptanwendungsgebiet für solche Geräte sind **Sensoren** und **Aktuatoren** im **IoT** Umfeld. Dort sind vor allem **Speicherplatz**, **Arbeitsspeicher**, **Akkulaufzeit**, und demzufolge auch **Nachrichtengröße**, **Wartezeiten** und weitere Metriken stark beschränkt. 

Das macht HTTP mit relativ großen Nachrichten und den darunterliegenden TCP Stack mit großen Wartezeiten viel zu schwergewichtig. Denn große Nachrichten bedeuten Latenz und Resourcenverbrauch. Lanke Wartezeiten bedeuten mehr Uptime und dadurch mehr Stromverbrauch. CoAP lößt diese Probleme durch bestimmte Designentscheidungen und Kompromisse, die im IoT Umfeld eingegangen werden können.

## Grundlagen

CoAP basiert auf **UDP** mit dem Standardport **5683**. Ansonsten ist CoAP, wie [hier](#protokoll) beschrieben, im Wesentlichen stark an HTTP und REST orientiert, lediglich auf die beschränktheit von IoT Geräten optimiert.

IoT und IIoT Geräte laufen oft Batteriebetrieben. Um dennoch maximale Laufzeit zu gewährleisten schlafen vor allem Sensoren die meiste Zeit und werden nur gestartet um einen Wert zu messen, an einen Server zu senden und sich wieder schlafen zu legen. Um diese Uptime zu minimieren ist es vor allem wichtig die Latenz zum Server und [Nachrichtengröße](#nachrichtenformat) zu minimieren.

Von CoAP profitieren vor allem Sensoren, da sie mehr schlafen können. Aber auch Aktuatoren profitieren von dem einfacheren Netzwerkstack den CoAP im vergleich zu alternativen wie HTTP oder MQTT mit sich bringen.

## Protokoll

Da sich CoAP stark an HTTP orientiert hat basiert das Protokoll ebenfalls aus **Request** vom **Client** zum Server und einer **Response** vom **Server** zum Client.

Sowohl Request als auch Response haben eine **Request Line** bzw **Status Line**, optionale **Header** und einen optionalen **Body**. Methoden, Header und Body stimmen im Wesentlichen mit HTTP und REST überein, lediglich das [Nachrichtenformat](#nachrichtenformat) ist wesentlich kompakter gestaltet. So gibt es **GET**, **POST**, **PUT** und **DELETE**, auf weitere Optimierungen von HTTP wie zum Beispiel *Options* und *Preflights* wurde jedoch verzichtet, da diese nicht dem Ziel dienen die Latenz zu minimieren.

## Nachrichtenformat

{{< mermaid >}}
---
title: "CoAP Message"
---
packet
0-1: "Version"
2-3: "Type"
4-7: "Token length"
8-15: "Code"
16-31: "Message ID"
32-63: "Token (Variable Length)"
64-95: "Options (Multiple, Variable Length)"
96-103: "11111111"
104-127: "Payload (Variable Length)"
{{< /mermaid >}}

 - **VER** *(2 Bits)* gibt die CoAP Version (aktuell `0b01`) an
 - **TYPE** *(2 Bits)* gibt den [Message Type](#message-type) für die [Fehlertolleranz](#fehlertolleranz) an
 - **Tokenlen** *(4 Bits)* gibt die länge des [Token](#token) an
 - **Code** *(8 Bits)* gibt den [Code](#code) entspricht der **Method** bzw dem **Status** bei HTTP
 - **Message ID** *(16 Bits)* ist eine eindeutige zuordnung, um mehrfach gesendete Nachrichten nicht mehrfach zu interpretieren.
 - **Token** *(0-8 Bytes)* enthält den [Token](#token)
 - **Options** *(variable)* enthalten [Options](#options), die den Headern in HTTP entsprechen
 - **11111111** *(1 Byte) 0xFF* trennt den **Payload** (Body in HTTP) von den Options.
 - **Payload** *(variabel)* enthält den Payload

> [!TIP]
> Die Länge des Payloads wird nicht explizit angegeben. Diese wird aus der Länge des UDP Datagrams und der bisherigen länge von Header und Options berechnet.

### Message Type

 - **CON** `0b00`
 - **NON** `0b01`
 - **ACK** `0b10`
 - **RST** `0b11`

### Code

{{< mermaid >}}
---
title: "Code"
---
packet
0-2: "Class"
3-7: "Detail"
{{< /mermaid >}}

Der Code entspricht der Methode im HTTP Request, bzw. dem Status in der HTTP Response. Er besteht aus Class und Detail.

 - **Class** gibt an ob es sich um was es sich handelt und ist auch hier an HTTP orientiert, aber kürzer gefasst.
    - **Request** `0b000` *(0)*
    - **Success** `0b010` *(2)*
    - **Client Error** `0b100` *(4)*
    - **Server Error** `0b101` *(5)*
 - **Details** können weitere details über Fehlermeldungen kodieren, wie z.B. *Created* oder *Not Found*, oder im Fall vom Request die **Method** angeben
   - **GET** `0b00001` *(1)*
   - **POST** `0b00010` *(2)*
   - **PUT** `0b00011` *(3)*
   - **DELETE** `0b00100` *(4)*

### Token

Der Token dient der Zuordnung von Response zu Request auf Client Seite. Die Message ID ist nur um doppelte Verarbeitung zu vermeiden und bei Request und Response nicht grundsätzlich identisch.

> [!TIP]
> Dass der Token optional ist wirkt ersteinmal kontraproduktiv. Allerdings ist eine Zuordnung von Request und Response trivial, wenn immer nur eine Nachricht gesendet werden kann. Dann gehört das Response zum vorangegangenen Request und benötigt keinen Token und das Protokoll kann noch mehr resourcen sparen.

### Options

{{< mermaid >}}
---
title: "Code"
---
packet
0-2: "Delta"
3-7: "Length"
8-15: "Delta Extended (Optional 0-2 Bytes)"
16-23: "Length Extended (Optional 0-2 Bytes)"
24-31: "Value (Optional ≥0 Bytes)"
{{< /mermaid >}}

Um Options möglichst klein zu halten, werden namen nicht im Klartext als Strings gesendet, sondern Zahlen zugeordnet. Zum Beispiel:

 - **Uri-Host** *3* Hostname des Servers
 - **Uri-Port** *7* Port des Servers
 - **Uri-Path** *11* Pfad der Anfrage
 - **Uri-Query** *15* Query Parameter

### Delta

Die Zahl, die die Option kodiert wird als *delta* mitgegeben, um große Zahlen effizient zu kodieren. Die Idee ist, dass sowieso mehrere Optionen angegeben werden und daher nicht die komplette (potentiell sehr große) Zahl, sondern der Abstand zum Vorherigen Header angegeben wird.

Will man zum Beispiel Host, Port, Path und Query senden, müsste die Option ID mindestens 5 Bits sein um die 15 zu kodieren. Da Option IDs viel größer sein können, würde das ein komplettes Byte verschwenden. Stattdessen geben wir die IDs *3*, *7*, *11* und *15* als Differenz an. Also *3*, *4*, *4*, *4*. So bräuchte man nurnoch 3 statt 5 bits. CoAP sieht **4 bits** für das Delta vor um Abstände von **0** (gleiche Option nochmal) bis **12** zu kodieren.

Wenn der Abstand 12 nicht reicht, kann man mit einer **13** kodieren, dass man das Delta in **1 Byte** in **Delta Extended** kodiert. Dort kann man dann Delta **13** *(0)* bis **255** *(268)* kodieren. *0-12* werden aus Effizienzgründen hier nicht zugelassen.

Wenn 268 immernoch nicht reichen, dann zeigt die **14** im Delta Feld an, dass **2 Byte** in **Delta Extended** den Abstand kodieren. Bei 13 ist der mindest abstand **269** *(0)* und geht bis **65535** **(65804)** hoch.

Die **15** kann zwar in delta kodiert werden, zeigt aber an, dass danach **Payload** kommt und darf deshalb nicht genutzt werden. Anderenfalls kann es passieren, dass der Server diese Option als Start für den Payload interpretiern, oder, falls nicht noch eine 15 folgt, mit einem Fehler abbricht.

## Fehlertolleranz

CoAP basiert auf UDP und hat demnach keine Garantien ob die Nachricht angekommen ist. Um das nachträglich sicherzustellen bietet CoAP eine einfache implementierung von Timeouts und Retransmits an. Nachrichten können als **CON** *Confirmable* oder **NON** *Non-Confirmable* deklariert werden. Das wird im [Message Type](#message-type) enkodiert.

**Non-Confirmable** messages sind **Best Effort**, bzw. **Fire and Forget** und damit einmal gesendet und (hoffentlich) angekommen.

Wenn eine Nachricht mehr Sicherheit braucht, kann sie als **Confirmable** markiert werden. Dann wartet der Client auf ein **ACK** *Acknowledgement* vom Empfänger, sodass die Nachricht sicher zugestellt wurde. Sollte das Acknowledgement nicht, oder nicht rechzeitig eintreffen, wird die Nachricht erneut gesendet.

Wenn ein Empfänger eine confirmable, aber kaputte Nachricht empfängt, oder aus anderen Gründen nicht fähig ist die Nachricht zu verarbeiten, sendet er ein **RST** *Reset*. In TCP wird darauf verzichtet, da der Sender die Nachricht nach einem Timeout ohnehin wiederholt, bei CoAP sorgt dieser mechanismus aber für kürzere Wartezeiten.

> [!TIP]
> Acknowledgement und Reset sind unabhängig von einer Potentiellen Antwort. Wenn Request und Reply beide confirmable sind, dann werden also insgesamt **4** Nachrichten ausgetauscht.

## Congestion Control

Damit Geräte nicht unnötig auf ausgefallene Server warten und das Netzwerk nicht überlastet, gibt es ähnlich wie bei TCP Congestion Control Mechanismen. Allerdings wesentlich simpler.

Um Überflutung zu vermeiden, sendet ein Sender (Client oder Server) maximal\(NSTART\) parallele **CON** Nachrichten. Bei Antworten auf Broadcast Nachrichten, wie zum Beispiel bei [Service Discovery](#service-discovery), besteht die gefahr dass viele (in großen Netzen hunderte) Services gleichzeitig antworten. Um das zu vermeiden wartet ein Service bei der Antwort auf Broadcast Nachrichten zwischen\(0\) und\(DEFAULT\_LEISURE\).

Bei Anfragen kann es sein, dass der Empfänger aktuell nicht antwortet. Zum Beispiel kann ein Sensol schlafen oder ein Server ausgefallen sein. Wenn der Zustand des Empfängers unbekannt ist, gilt eine Anfrage an ihn als *Probing*. Die Anfrage unterscheidet sich nicht von einer Regulären Anfrage, allerdings dürfen solche Anfragen nur sparsam gesendet werden um das Netzwerk nicht durch schlafende Geräte zu überlasten. Dabei muss der Sender eine Datenrate von unter\(PROBING\_RATE\) einhalten. Proktisch wird das so gemacht, dass ein Datagramm der Länge\(l\) gesendet wird und dann mindestens\(l/PROBING\_RATE\) gewartet wird.

Bei **CON** Nachrichten wartet ein Sender\(ACK\_TIMEOUT\) bevor er davon ausgeht, dass die Nachricht beim Empfänger nicht angekommen ist. Da das an einer Kollision liegen kann, wartet er vor dem erneuten Senden zufällig zwischen\(ACK\_TIMEOUT\) und\(ACK\_TIMEOUT*ACK\_RANDOM\_FACTOR\) (das warten auf den Timeout ist dier schon enthalten). Nach maximal\(MAX\_RETRANSMIT\) gibt der Sender auf.
Es kann ermittelt werden, wielange es dauert bis der letzte Retransmit (letzter Versuch) gesendet wurde \(MAX\_TRANSMIT\_SPAN\)) und wielange es dauert bis das ACK zum letzten Versuch eintreffen sollte \(MAX\_TRANSMIT\_WAIT\)). Danach kann der Sender eine Fehlermeldung an die Anwendung geben.

Um nicht unnötig auf nie kommende Antworten *(nicht Acknowledgements!)* zu warten, muss der Client eine Obergrenze setzen für die Zeit die ein Datagram zum server benötigt \(MAX\_LATENCY\)) und wielange der Server maximal zum Verarbeiten benötigt \(MAX\_PROCESSING\)). Daraus kann dann abgeleitet werden, wielange ein Antwort maximal dauert \(MAX\_RTT\)).

Mit Hilfe dieser Werte kann ermittelt werden, wielange es maximal dauert bis eine Antwort auf eine Anfrage kommt \(EXCHANGE\_LIFETIME\)). Damit kann der Client die Anfrage verwerfen, wenn bis dahin keine Antwort kam und einen Fehler an die Anwendung geben statt ewig zu warten. Zudem wissen wir, wielange es dauert um eine NON nachricht zuzustellen \(NON\_LIFETIME\)) damit können wir Nachrichten löschen, da wir sie sicher nicht mehr zur Duplikaterkennung benötigen.


| Name              | Standard Wert | Formel                                                            |
| ----------------- | ------------- | ----------------------------------------------------------------- |
| NSTART            | 1             | Konfigurierbar                                                    |
| DEFAULT_LEISURE   | 5 s           | Konfigurierbar                                                    |
| PROBING_RATE      | 1 byte/s      | Konfigurierbar                                                    |
| ACK_TIMEOUT       | 2 s           | Konfigurierbar                                                    |
| ACK_RANDOM_FACTOR | 1.5           | Konfigurierbar                                                    |
| MAX_RETRANSMIT    | 4             | Konfigurierbar                                                    |
| MAX_TRANSMIT_SPAN | 45 s          |\(ACK\_TIMEOUT*(2^{MAX\_RETRANSMIT}-1)*ACK\_RANDOM\_FACTOR\)      |
| MAX_TRANSMIT_WAIT | 93 s          |\(ACK\_TIMEOUT*(2^{MAX\_RETRANSMIT+1}-1)*ACK\_RANDOM\_FACTOR\)    |
| MAX_LATENCY       | 100 s         | Konfigurierbar                                                    |
| PROCESSING_DELAY  | 2 s           | Konfigurierbar                                                    |
| MAX_RTT           | 202 s         |\(2*MAX\_LATENCY+PROCESSING\_DELAY\)                              |
| EXCHANGE_LIFETIME | 247 s         |\(MAX\_TRANSMIT\_SPAN+MAX\_RTT\)                                |
| NON_LIFETIME      | 145 s         |\(MAX\_TRANSMIT\_SPAN+MAX\_LATENCY\)                            |

## Observe

Mit der **Observe** Option bietet CoAP ein Subscriber Modell als Alternative zum Polling in HTTP. Wenn in einem Request die Observe Option *(6)* auf 0 gesetzt ist, aboniert der Client die Resource und wird vom Server mit Updates versorgt. Der Server antwortet dann regelmäßig mit Replies, die ebenfalls die Observe Option haben, den Wert davon aber hochzählen um eine eindeutige Reihenfolge zu gewährleisten.

## Resource Discovery

CoAP stellt eine Standardisierte Möglichkeit bereit um herauszufinden, welche Resourcen (Sensoren und Aktuatoren) ein CoAP Gerät zur Verfügung Stellt. Nach dem Standard liefert eine Anfrage an `GET coap://device.local/.well-known/core` eine standardisierte Liste an Resourcen. Die Antwort auf diese Discovery Anfrage wirt im **CoRE Link** ([RFC 6690](https://datatracker.ietf.org/doc/html/rfc6690)) Format gesendet, welche an das **Web Linking** ([RFC 5988](https://datatracker.ietf.org/doc/html/rfc5988)) von HTTP angelehnt wurde.

```
</sensor/temp>;rt="temperature-c";if="sensor",
</light>;rt="light";if="actuator"
```

Die Antwort gibt an welche Endpunkte es gibt `</sensor/temp>`, und was an dem Endpunkt hängt:

 - **rt** Resource Type *(Temperatur, Licht, Thermostat, ...)*
 - **if** Interface *(Aktuator, Sensor)*
 - *cf* Content-Format *(JSON, XML, ...)*
 - ...

### Service Discovery

Resource Discovery funktioniert auf Gerätebasis. Also jedes Gerät muss eine Anfrage erhalten. Um Geräte selbst zu entdecken, unterstützt CoAP Multicast Requests `GET coap://[ff02::1]/.well-known/core`. Daraufhin erhält der Client eine Antwort pro Gerät mit detailierter Resourcenbeschreibung.