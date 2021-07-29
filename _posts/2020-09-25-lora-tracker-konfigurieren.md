---
title: "LoraWAN GPS-Tracker konfigurieren"
layout: post
author: Constantin Müller, Maximilian Richt, Stefan Kaufmann
excerpt: Wie wir unsere Lora-Tracker konfigurieren
---

Neben den [Wifi-Trackern](/2020/04/24/wifi-tracker-bauanleitung/) benutzen wir als GPS-Tracker hauptsächlich die LoRaWAN GPS-Tracker [Dragino LGT 92](https://www.dragino.com/products/lora/item/142-lgt-92.html) in der Ausführung mit eingebautem Akku. Zwar testen wir auch andere Trackermodelle, möchten aber in diesem Artikel kurz beschreiben, wie wir die Tracker konfiguriert haben. Der Vorteil dieser Tracker ist, dass sie als fertiges Produkt gekauft werden können, aber trotzdem komplett [Open Hardware](https://de.wikipedia.org/wiki/Open-Source-Hardware) sind und die Firmware als Freie/Open-Source-Software veröffentlicht wird. Die Annahme von [Bugfixes auf GitHub](https://github.com/dragino/LGT-92_-LoRa_GPS_Tracker/pulls) scheint aber noch nicht so gut zu funktionieren. Der große Vorteil, dass man die Firmware selber anpassen und auch Firmwareupdates selbstständig einspielen kann, bleibt natürlich bestehen. Und das ist bei solchen Geräten nicht selbstverständlich, wie wir bei Experimenten mit Trackern anderer Hersteller schmerzlich feststellen mussten.

Wir möchten in diesem Artikel festhalten wie wir unsere Tracker konfiguriert haben. Die Konfiguration findet, wie in der [Anleitung ab Seite 20](http://www.dragino.com/downloads/downloads/LGT_92/LGT-92_LoRa_GPS_Tracker_UserManual_v1.5.5.pdf) beschrieben mit AT-Kommandos über die serielle Konsole statt.

Eine Liste aller AT-Kommandos findet sich in der [AT-Anleitung des Herstellers](http://www.dragino.com/downloads/downloads/LGT_92/DRAGINO_LGT92_AT_Commands_v1.5.3.pdf). Hier solltet ihr überprüfen ob die Firmware auf dem Stand der Dokumentation ist, und diese im zweifel aktualisieren.

## Akkulaufzeit optimieren

![Dragino Tracker beim Laden](/assets/images/blog/20200925_tracker-laden.jpg)

Da GPS sehr stromhungrig ist, bieten die Tracker eine Reihe von Konfigurationsmöglichkeiten um für verschiedene Anwendungsfälle die bestmögliche Laufzeit herauszuholen. Wichtigster Punkt: Wie oft wird der GPS-Chip gestartet um die Position zu erfassen. 

Das Gute an den Dragino-Trackern ist, dass diese einen Beschleunigungssensor eingebaut haben und ab Firmwareversion 1.5 lässt der Tracker so konfigurieren, dass dieser uns beim Stromsparen sehr hilft: Unsere Räder stehen die meiste Zeit des Tages ja an einem Fahrradständer. Solange sie stehen brauchen wir keine regelmäßigen Positionsupdates im System. (Vielleicht alle paar Stunden mal, um sicher zu gehen, dass das Rad noch dort steht und kein Lora-Paket zwischendurch verloren gegangen ist.) Mit dem Beschleunigungssensor kann der Tracker erfassen, wann das Rad bewegt wird, und dann anfangen seine Position häufiger zu senden.

Mit `AT+TDC` kann man konfigurieren, in welchem Intervall in *Milli*sekunden der Tracker seine Position sendet, wenn er bewegt wird. Und mit `AT+KAT` den Abstand zwischen Aussendungen in *Milli*sekunden, solange keine Bewegung registriert wurde. Momentan nutzen wir die Werte `AT+TDC=90000` und `AT+KAT=1800000`. `AT+KAT` ist bei uns mit 30 Minuten sehr gering eingestellt, da wir das Feature mit der neuen Firmware erst einmal testen wollten. Wahrscheinlich ist es aber sinnvoll, diesen Wert auf drei Stunden erhöhen. `AT+TDC` ist bei uns mit 90 Sekunden auch relativ gering, weil wir überlegen in Zukunft [MDS-Daten](https://radforschung.org/log/mds-fuer-kommunen-erklaert/) zu generieren.

Ein Zweiter, für die Akkulaufzeit relevanter Faktor ist, wie lange der GPS-Chip versucht, eine Position zu ermitteln. Sucht dieser nicht lang genug nach Satelliten, ist die Genauigkeit der Position möglicherweise sehr schlecht. Stehen Fahrzeuge öfter unter Brücken, unter Dächern oder an sonstigen Orten mit schlechtem GPS-Empfang, verbraucht der GPS-Chip bei zu langer erfolgloser Suche sehr viel Strom. Diese Zeit lässt sich mit `AT+FTIME` konfigurieren. (Wert in Sekunden, in unserem Fall `AT+FTIME=180`)
Damit im Zusammenhang steht auch die konfigurierbare Mindestgenauigkeit. Das heißt, wir können konfigurieren, ab welcher Genauigkeit sich der Tracker mit der Position zufrieden gibt und diese sendet. Da der Empfänger für eine genauere Position länger braucht und damit mehr Strom verbraucht, müssen wir hier also ein gutes Verhältnis zwischen Genauigkeit und Akkulaufzeit finden. Dies kann sich je nach Anwendungsfall unterscheiden. Nach unserer Erfahrung ist die vorkonfigurierte Genauigkeit von `AT+PDOP=3.00` teilweise erschreckend ungenau. Unsere Tracker sind mit `AT+PDOP=2.00` konfiguriert. Bei [DOP](https://de.wikipedia.org/wiki/Dilution_of_Precision) handelt es sich nicht um eine Genauigkeit in Metern, sondern um eine dimensionslose Maßzahl für die Streuung der erfassten Positionswerte. Wenn die Genauigkeit `AT+PDOP` in der Erfassungszeit `AT+FTIME` nicht erreicht wurde, wird die bis dahin am genausten erfasste Position gesendet. `AT+PDOP` und `AT+FTIME` stehen hier also in einer gewissen Abhänigkeit zueinander - wir haben unser Optimum dabei auch noch nicht gefunden.

Ein Problem mit der aktuellen Firmware ist, dass leider die [Genauigkeit der GPS-Position nicht mit per Lora übermittelt wird](https://github.com/dragino/LGT-92_-LoRa_GPS_Tracker/issues/13). Andere GPS-Chips in neueren Hardwarerevisionen könnten sich anders verhalten. Wir haben Tracker in der Hardware-Version 1.4 mit dem Quectel L70-RL GPS-Chip. Es gibt wohl auch eine Nachfolgegeneration auf dem ein anderer GPS-Chip verbaut ist.


## Reichweitenoptimierungen

Für Orte die nicht so gut mit LoRaWAN abgedeckt sind, gibt es zwei relevante Einstellungen.

### Datenrate

![Datarate Calculator](/assets/images/blog/20200925_datarte-calcultaor.png)

Die Datenrate (oder auch Spreading Factor): LoRaWAN-Nodes können ihre Daten in verschiedenen Geschwindigkeiten senden. Um so langsamer die Daten gesendet werden, um so höher ist die Wahrscheinlichkeit, dass das Gateway zwischen vielen Störfaktoren das gesendete Signal noch herausfiltern kann. Dementsprechend steigt mit niedrigerer Datenrate (höherem Spreadingfactor) also auch in der Regel die Reichweite des Signals. Allerdings ist dabei zu beachten, dass wir uns das Übertragungsmedium mit allen anderen LoRa-Geräten und noch vielen anderen Sendern teilen, die diese offene Frequenz benutzen. Das heißt, um so länger wir senden, um so länger beeinträchtigen wir die Frequenz für alle anderen Teilnehmenden. Neben den rechtlichen Regelungen, die aus diesem Grund die Bundesnetzagentur getroffen hat, gibt es auch noch [restriktivere Beschränkungen, die sich die TTN-Community selbst auferlegt hat](https://www.thethingsnetwork.org/forum/t/limitations-data-rate-packet-size-30-seconds-uplink-and-10-messages-downlink-per-day-fair-access-policy-guidelines/1300). Das soll dafür sorgen, möglichst vielen Geräten die Teilnahme am Netzwerk zu ermöglichen. In der Praxis bedeutet dies: Es gibt einen Richtwert, wie lange jedes Gerät pro Tag maximal senden sollte. Wie viele Daten das sind, wie oft ein GPS-Tracker also seine Daten übermitteln kann, hängt dementsprechend auch mit der Datenrate zusammen. 

Um auszurechnen, wie häufig der Dragino-Tracker bei welcher Datenrate senden darf, gibt es diesen großartigen Rechner: [https://avbentem.github.io/airtime-calculator/ttn/eu868](https://avbentem.github.io/airtime-calculator/ttn/eu868). Hier sehen wir, dass die Tracker bei einer Payload-Größe von 21 Byte und der voreingestellten Datenrate (DR5 / SF7) 16 mal pro Stunde, also etwa alle 4 Minuten, ihre Position senden könnten. Damit stoßen wir an keine realen Grenzen, denn schon allein aus Akkulaufzeitgründen  ist es nicht sinnvoll dauerhaft so oft die Position zu übermitteln.
Selbst bei der niedrigsten festeinstellbaren Datenrate (DR2/SF10) dürfen wir noch mehr als zwei Geolocations pro Stunde senden, was in gut ausgelasteten System in denen die Räder mehrere Fahrten pro Tag machen noch deutlich ausreicht, da die Fahrräder sich ja trotzdem die meiste Zeit nicht bewegen.

Mit dem AT-Kommando `AT+DR=4` kann man somit die Datenrate auf 4 setzen, was dem Spreadingfaktor 8 entspricht. DR1 und DR2 dürfen nur benutzt werden, wenn [ADR (Adaptive Data Rate)](https://www.thethingsnetwork.org/docs/lorawan/adaptive-data-rate.html) aktiviert ist, was in unserem Anwendungsfall eher wenig praktikabel ist.

### Single Channel Gateways

![Selbstbau Single-Channel-Gateway aus der gleichen Hardware, die wir auch für unsere Wifi-Tracker verwenden](/assets/images/blog/20200925_selbstbau-single-channel-gateway.jpg)

LoRaWAN benutzt im Normalfall 8 Unterfreuqenzen (Channels/Kanäle). Dies dient dazu das geteilte Frequenzband besser auszunutzen. Das heißt auch, dass standardkonforme Gateways auch gleichzeitig auf alle 8 Kanäle hören sollten, um alle gesendeten Nachrichten empfangen zu können. Die Clients wählen sich zufällig einen Kanal, um das Band möglichst gleichmäßig auszunutzen. 

Solche vollwertige Gateways sind nicht für alle einfach erschwinglich, auch wenn der Preis für manche Geräte mittlerweile auf unter 100€ gefallen ist. Deswegen existieren Bauanleitungen für Selbstbau-Gateways, die nur auf einem Kanal empfangen und senden. An Orten ohne systematisch aufgebaute TTN-Infrastruktur sind dies womöglich die ersten und einzigen verfügbaren Gateways, oder auch im eigenen Arbeitszimmer für Experimente.  
Da diese Gateways im Durchschnitt aber nur jede achte Nachricht überhaupt empfangen, kann der Effekt solch eines Single-Channel-Gateways erst mal sehr enttäuschend sein. Um temporär Abhilfe zu schaffen, können die Dragino-Tracker in den Single-Channel-Mode versetzt werden. Wenn man die eingesetzte Frequenz des Single-Channel-Gateways kennt, kann der Tracker dann mit z.B. `AT+CHS=868100000` auf diese Frequenz festgesetzt werden. Das Problem ist, dass dadurch die vorhin angesprochene gleichmäßige Verteilung von Aussendungen auf alle Channels unterwandert wird. Deswegen sollte der Tracker wieder in den Multi-Channel-Mode gesetzt werden (`AT+CHS=0`), sobald er in einem „richtigen“ TTN-Netz mit ausreichend vielen standardkonformen Gateways eingesetzt wird.