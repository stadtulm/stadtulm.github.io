---
title: Wifi-Tracker bauen
layout: post
author: Constantin Müller
excerpt: Wie wir die Wifi-Geolocation-Tracker zusammenbauen.
---



Wir hatten [im Januar schon einmal beschrieben](/2020/01/13/trackinghardware/), dass wir auch selbst entwickelte Wifi-Tracker zur Positionsbstimmung benutzen. Das [Hardwaredesign](https://github.com/stadtulm/Lora-Wifi-Location-Tracker/blob/master/hardware/README.md) und die [Firmware](https://github.com/stadtulm/Lora-Wifi-Location-Tracker), welche wir dafür entwickelt haben ist natürlich als OpenSource bzw. OpenHardware auf GitHub verfügbar.

Allerdings ist es sicher auch interessant, wie genau wir die Tracker momentan einsetzen. Theoretisch funktioniert die Platine ja einfach, wenn man sie zusammen baut, aber irgendwie muss sie ja auch ans Fahrrad und mit Strom versorgt werden. Wie wir das aktuell umsetzen, beschreiben wir hier.

## Flashen

![](/assets/images/blog/20200423-wifi-tracker/ESP-12F-flashtool.jpg)

In dem Hardwaredesign sind Kontakte für Datentransfer und Buttons für Flash und Reset vorgesehen, um die Firmware auf dem fertig gelötetem Board aufzuspielen. Allerdings flashen wir die Firmware schon vor dem Auflöten mit einem [testboard programmer](https://de.aliexpress.com/wholesale?SearchText=burner+esp12) auf das ESP12-Modul. Zum einen vermeidet man dadurch, bei vielen Platinen immer wieder Kontaktkabel an und wieder abzulöten, andererseits kann man auch so die Buttons weg lassen, was im Zweifel zusätzlich nochmal Platz spart.

## Zusammenlöten

![](/assets/images/blog/20200423-wifi-tracker/ESP-und-RFM-loeten.jpg)

Beim Zusammenlöten starten wir mit dem ESP12-Modul. Es empfiehlt sich dieses zum Löten mit einer [dritten Hand](https://de.wikipedia.org/wiki/Dritte_Hand) oder einer Klemme an der Platine in der richtigen Kontaktposition zu fixieren. Danach das Gleiche noch mal mit dem RFM-96 LoRa-Modul.

![](/assets/images/blog/20200423-wifi-tracker/platine-backside.jpg)

Sind diese beiden Bauteile angelötet, kommen die Kleinteile auf der Rückseite dran. Dabei sind eigentlich nur die fünf `10k`-Widerstände und der `100uF`-Kondensator relevant. Die `1k`-Widerstände sind nur nötig, wenn die serielle Schnittstelle zum Flashen oder Debuggen benutzt werden soll. Die `100k`- und `680k`-Widerstände waren ursprünglich zum Messen der Spannung gedacht, verbrauchen ohne weitere Hardwaremodifikation  aber zusätzlich Strom im Schlafzustand und werden deswegen momentan auch von der Firmware nicht unterstützt. Dieses Problem wollen wir in zukünftigen Hardwareversionen aber lösen, da die Information zum Batteriezustand im Betrieb durchaus wichtig ist. Den Spannungsregler `AMS111733` können wir in unserem Fall auch komplett weg lassen, da wir den Tracker mit zwei AA Batterien betreiben, welche die gewünschte Spannung schon von Haus aus liefern.

![](/assets/images/blog/20200423-wifi-tracker/antenne.jpg)

An dem Antennen-Pin vom RFM sollte noch ein Kabel als Antenne angelötet werden. Dieses kann dann später nach dem Einbau gekürzt werden. Billige Spiralantennen haben sich bei uns als schwer zu löten und nicht sehr effektiv herausgestellt. Das kann aber bei anderen Antennen durchaus anders sein.

## Gehäuse

![](/assets/images/blog/20200423-wifi-tracker/gehaeuse-original.jpg)

Nach langem Experimentieren und langer Suche nach einem günstigen Gehäuse, welches den Tracker und die Batterien platzsparend fassen kann, haben wir uns für ein modifiziertes 3×AA Batteriegehäuse entschieden. Die Verkabelung im Gehäuse wird so modifiziert, dass nur zwei Batterien genutzt werden. Im dritten Fach findet dann der Tracker seinen Platz.

Von diesen Gehäusen gibt es leicht abweichende Ausführungen, die Verkabelung ist aber in der Regel immer gleich. Preislich liegen die Gehäuse bei AliExpress je nach Menge und Umrechnungskurs bei etwa 40-70 Cent.

![](/assets/images/blog/20200423-wifi-tracker/abdeckung-offen.jpg)

Als erster Schritt wird die Abdeckung über dem Schalter gelöst (manchmal geschraubt, manchmal gesteckt). Dadurch kann der Schalter zusammen mit dem schwarzen Kabel und der Spirale nach oben raus gezogen werden.

![](/assets/images/blog/20200423-wifi-tracker/kabel-nach-oben-biegen.jpg)

Im nächsten Schritt kann das rote Kabel bis zum Metallplättchen heraus gezogen, und das Kabel nach oben gebogen werden. 

![](/assets/images/blog/20200423-wifi-tracker/schwarzes-kabel-anloeten.jpg)

Als Nächstes muss der Doppelkontakt nach oben aus der Halterung gezogen werden. An die Seite ohne Feder kann nun ein Kabel angelötet werden. (Ich verwende da immer ein Stück des vorher gezogenen schwarzen Kabels.) 

![](/assets/images/blog/20200423-wifi-tracker/verbindung-anloeten.jpg)

Danach müssen die beiden Kabel nur noch so eingekürzt werden, dass sie bequem an die Platine gelötet werden können. Zum Abschätzen des Abstandes empfiehlt es sich, die Platine schon mal in die endgültige Stelle einzulegen. Das rote Kabel wird nun an den `3V` Kontakt gelötet und das schwarze auf `GND`.

![](/assets/images/blog/20200423-wifi-tracker/beschriftet-und-batterien.jpg)

Außerdem beschriften wir die Platinen noch mit einer Kurzform der TTN-Device-ID, welche wir vorher in der Firmware konfiguriert haben, damit es später keine Verwechslungen gibt. Nun können die Batterien rein und der Deckel drauf. Falls die Platine beim Schütteln klappert, kann noch ein bisschen Füllmaterial ins Gehäuse geklemmt werden. 

## Verkleben

![](/assets/images/blog/20200423-wifi-tracker/zugeklebt.jpg)

Jetzt kommen wir zum unausgereiftesten Teil des ganzen: Das Gehäuse ist alles andere als wasserdicht. Deswegen haben wir es einfach mit ordentlich Gaffa verklebt. Dabei unbedingt auf die Ecken achten. Das ganze ist dann leider beim Batterien tauschen etwas unbequem, da das alles wieder aufgeschnitten werden muss. Bei uns halten die Batterien aber so etwa 3 bis 4 Monate.

![](/assets/images/blog/20200423-wifi-tracker/kondenzwasser1.jpg)

Bis jetzt hatten wir noch keine Fälle in denen nennenswert Wasser rein geflossen wäre, wie wir das bei den Abzweigdosen für die GPS-Tracker schon erlebt haben. Allerdings hatten wir jetzt schon Fälle, in denen sich Korrosion gebildet hat und sich Kondenswasser abgesetzt hat. Hier könnte vielleicht ein kleiner Beutel Silica-Gel nicht schaden. Über Vorschläge und Ideen zur besseren Abdichtung freuen wir uns immer.

![](/assets/images/blog/20200423-wifi-tracker/kondenzwasser2.jpg)

## Klett

![](/assets/images/blog/20200423-wifi-tracker/klett.jpg)

Nach dem Verkleben werden auf einer Seite noch zwei Streifen Hakenklett angeklebt. Auf der Innenseite der Beschriftungsplatten unserer Fahrräder befinden sich auch zwei Flauschklettstreifen - und da wird der Tracker dann einfach angeklettet. Das hält! Außerdem sollte auch außen auf den Tracker noch einmal die Tracker-ID drauf geschrieben werden.

## Mitentwickeln

Für alle, die Interesse haben sich auch einen Wifi-only Geolocation-Tracker zu bauen, hätten wir auch noch ein paar unbestückte Platinen, die wir gern verschicken können. Schickt uns dafür einfach eine Mail an openbike@ulm.dev

