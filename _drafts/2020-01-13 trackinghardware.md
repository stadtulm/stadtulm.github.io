---
title: Wie funktioniert eigentlich das mit den Fahrradtrackern?
layout: post
author: Stefan Kaufmann
excerpt: Nach dem Chaos Communication Congress wurden wir immer wieder gefragt, wie wir derzeit die Standorte der Fahrräder ermitteln. Hier deswegen ein kleiner Aufschrieb, wie das bislang funktioniert.
---

Auf dem 36. Chaos Communication Congress des CCC zwischen Weihnachten und Neujahr gab es einige Ulmer Beteiligung – und unter anderem auch einen Vortrag von unseren Stadtverwaltungs-Fellows Consti und Max, [der es auch bis Heise Online geschafft hat](https://www.heise.de/newsticker/meldung/36C3-Die-Verkehrswende-selber-hacken-4624862.html).

Nebenan [auf radforschung.org](https://radforschung.org/log/verkehrswende-selber-hacken/) gibt es einen begleitenden Artikel samt allen Quellen, den Vortrag selber gibt es natürlich [auf media.ccc.de in ganzer Länge und auch mit englischer Übersetzung anzusehen](https://media.ccc.de/v/36c3-10881-verkehrswende_selber_hacken).

<iframe width="1024" height="576" src="https://media.ccc.de/v/36c3-10881-verkehrswende_selber_hacken/oembed" frameborder="0" allowfullscreen></iframe>

*Einschub: Der Vortrag erwähnt die Stadt als Fellowship-Geberin eher subtil – das ist Absicht und so gewollt, [denn die Regeln des Congress](https://content.events.ccc.de/cfp/36c3/checklist.html) sagen ganz klar, dass dies eine nicht-kommerzielle Bühne ist, auf der Marketing für ArbeitgeberInnen nicht akzeptiert ist.*

Im Nachgang gab es daraufhin viel viel Feedback, Tipps und Fragen. Und eine der meist gestellten Fragen war, wie denn die Positionsübermittlung der Fahrräder derzeit funktioniert. Deswegen hier mal ein Aufschrieb dazu.

## Wie die Daten überhaupt übertragen werden

Eine grundlegende Annahme für uns ist *derzeit*, dass wir uns nicht auf klassische SIM-Kartenbasierte [M2M](https://de.wikipedia.org/wiki/Machine_to_Machine)-Kommunikationsstrukturen verlassen wollen – also irgendwas, was klassische Mobilfunkbetreiber anbieten. 

In Ulm gibt es bereits seit Ende 2016 [ein gut ausgebautes LoRaWAN-Netzwerk](https://lora.ulm-digital.com/) auf Basis des freien [The Things Network](https://lora.ulm-digital.com/). Das ermöglicht uns, ohne laufende Subskriptionskosten kleine Datenmengen zu übertragen und zu experimentieren. Ein Nachteil muss aber erwähnt werden: Das Grundprinzip des Netzwerks ist, dass vorwiegend Daten von den Endgeräten zu den fest installierten Gateways übertragen werden. Die Rückrichtung von der Infrastruktur zu den Endgeräten ist eigentlich mehr die Ausnahme, und sie funktioniert auch nur unter gewissen Voraussetzungen.

Das ist aber im derzeitigen Projektstadium nur mäßig projektkritisch – und es spricht auch nichts dagegen, gegebenenfalls in einem anderen Setting auch M2M-SIM-basierte Geräte einzusetzen.

## Vom MVP zur eigenen Hardware – oder, alles mal probieren

Der aktuelle Stand ist, überhaupt Positionierungsdaten ins Backend zu übertragen. Beim [Feldversuch auf dem Chaos Communication Camp](https://radforschung.org/log/cccamp19-review/) im Sommer 2019 passierte das mit [TTGO T-Beam](https://tinymicros.com/wiki/TTGO_T-Beam) Development Boards. Das sind für ca. 20 EUR pro Stück aus Fernost bestellbare Entwicklungsplatinen, auf denen ein programmierbarer [ESP32-Controller](https://de.wikipedia.org/wiki/ESP32), ein halbwegs passables GPS-Modul und ein LoRaWAN-Modem ([RFM95W](https://www.hoperf.com/modules/lora/RFM95.html)) gemeinsam verbaut sind. Und weil auf der Platine auch gleich eine Lade- und Entladeregelung sowie eine Halterung für eine [18650](https://de.wikipedia.org/wiki/Lithium-Ionen-Akkumulator#Bauformen)-Akkuzelle dabei sind, ist das praktisch eine alles-in-einem-Lösung, die wir einfach so an die Fahrräder binden und benutzen konnten.

Naja, jedenfalls fast: Selbst wenn das Gerät nur alle 10 Minuten aufwacht, um die Position zu bestimmen, brauchen die Platinen auch im Deep-Sleep-Modus vergleichsweise sehr viel Energie. Die Akkus waren indes meist nach gut 30 Stunden komplett leer und mussten regelmäßig getauscht werden. Für den Praxiseinsatz ist das natürlich so gar nicht tauglich.

Der Quellcode, den wir auf dem Camp im Einsatz hatten, ist [selbstverständlich auf Github verfügbar](https://github.com/radforschung/Lora-TTNMapper-T-Beam). Wie man dort erkennen kann, hatten wir auch mit günstigen Beschleunigungssensoren MPU9250 experimentiert, die das System aufwecken, wenn am abgestellten Fahrrad gewackelt wird.

## Positionierung auf WLAN-Basis

Consti hatte eine andere Idee, wie man lange Betriebszeiten mit wenig Energie hinbekommt. Er hat einen eigenen Aufbau auf Basis eines [ESP8266](https://de.wikipedia.org/wiki/ESP8266) getestet, der intervallbasiert die [BSSIDs](https://en.wikipedia.org/wiki/Service_set_(802.11_network)#Basic_service_set_identifier_(BSSID)) umliegender WLAN-Netzwerke erkennt und diese dann ins Backend schickt. Über den offenen [Mozilla Location Service](https://location.services.mozilla.com/) ist dann eine grobe Rückpositionierung möglich – falls genügend Menschen für diesen Ort [Positionierungsdaten](https://wiki.mozilla.org/CloudServices/Location/Software) angelegt haben. Für eine grobe Positionsbestimmung reicht das allemal – denkbar ist natürlich, genau wie bei allen anderen Methoden, die Positionierung über passende [Beacon-Verfahren](https://en.wikipedia.org/wiki/IBeacon) an den festen Abstellstationen noch zu präzisieren, wenn man dafür geeignete Hardware einsetzt.

## Kommerziell erhältliche Tracker

Um schnell Räder auf die Straße zu bringen und Tests fahren zu können, hatten wir zudem einige kommerziell erhältliche GPS-Tracker mit LoRa-Modem gekauft und getestet. Das war hier vor allem der [Dragino LGT92](http://www.dragino.com/products/lora/item/142-lgt-92.html) – der witzigerweise ein ähnliches Feature Set hat wie unser Eigenaufbau aus T-Beams mit Beschleunigungssensor, und dessen Erfassungszyklus auch ähnlich gedacht ist wie das, auf das wir mit den T-Beams kamen. 

Zum Testen reichen die Geräte ganz passabel aus; an den Fahrrädern haben wir sie in Feuchtraum-Abzweigdosen mit Schellen aus dem Baumarkt befestigt. Ein Tracker kostet bei den üblichen Einkaufsplattformen rund 40 €.

## Unsere geplante eigene Lösung

…ist eigentlich grob eine Mischung zwischen dem T-Beam und den Dragino-Trackern: Eine eigene Plattform auf ESP32-Basis mit verbundenem MPU9250-Gyroskop und abschaltbarer Spannungsversorgung für die allzu energiehungrige Sensorik, v.A. das GPS. So kann der Tracker die meiste Zeit im Tiefstschlafmodus verbringen und dabei kaum Energie aufnehmen – und in periodischen Abständen, bzw. wenn die IMU Bewegung erkennt, wacht die Platine auf und ermittelt ggf. die Position.

![](/assets/images/blog/20200113_testaufbau.jpg)

Langfristig möchten wir diese Plattform auch mal testweise mit den Rahmenschlössern verbinden, die Consti und Max bereits analysiert hatten, um damit Entsperrvorgänge anzustoßen. Für den Anfang freuen wir uns aber auch schon um eine sehr energiesparsame Variante, die im Idealfall bei Benutzung des Rads auch über den Nabendynamo erhaltungsgeladen werden kann, so dass die Wartungsintervalle möglichst ausgedehnt werden können.

![](/assets/images/blog/20200113_current.jpg)

Im Testaufbau hat das alles auch schon sehr gut funktioniert. Im Tiefschlaf braucht das System weniger als ein halbes Milliampere, und trotzdem funktioniert die Positionierung vor allem draußen schnell und zuverlässig. Auch hier können Beacon-Systeme an ausgewiesenen Abstellpunkten zur Präzision der Erkennung beitragen.

## Selber mitentwickeln

Wir möchten als Freie Soft- und Hardware entwickeln – und dazu gehört auf jeden Fall, auch anderen den Einstieg in die Mitentwicklung zu erleichtern. Sobald wir die ersten prototypenfähigen PCB-Designs haben, landen die natürlich auf Github. Und wir möchten interessierte Hackspaces auch gerne Platinen aus der ersten Serie zum selbst belöten zuschicken, sobald sie gefertigt sind. Mehr dazu folgt hier, sobald es soweit ist!
