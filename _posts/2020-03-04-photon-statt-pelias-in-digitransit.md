---
title: Photon statt Pelias in digitransit nutzen
layout: post
author: Constantin Müller
excerpt: Digitransit mit dem Geocoder Photon benutzen.
---

![](/assets/images/blog/20200304-mapmarker.jpg)

Zur Adresssuche benutzt digitransit den OpenSource-Geocoder [Pelias](https://github.com/pelias/pelias). Leider wird dieser nach dem [Ende von Mapzen](https://www.mapzen.com/blog/shutdown/) so gut wie nicht mehr weiter entwickelt. Außerdem hatte Pelias in unserer Instanz Probleme bei der Zuordnung von administrativen Grenzen. Da Pelias die Daten dafür nicht aus Openstreetmap bezieht, erschwerte dies uns die Analyse und Lösung des Problems. 

Glücklicherweise gibt es alternativ zu Pelias noch [Photon](https://github.com/komoot/photon), welcher eine vergleichbare API für suggest-as-you-type-Adresssuche anbietet. Da Photon auf dem Importer des Standard-OpenStreetMaps-Geocoder [Nominatim](http://nominatim.org/) basiert, ist die Auswertung der OpenStreetMap-Daten entsprechend ausgereift.

Leider sind die APIs von Pelias und Photon unterschiedlich, das Featureset aber ähnlich. Um Photon alternativ als Geocoder in digitransit benutzen zu können haben wir deswegen einen [kleinen Adapter](https://github.com/stadtulm/photon-pelias-adapter) gebaut, der die aus digitransit kommenden Anfragen in einen Photon-Request und die Antwort aus Photon wieder zurück in das Pelias-Format übersetzt.

Es werden nicht alle Funktionen von Pelias unterstützt, sondern hauptsächlich die Funktionen, welche von digitransit benutzt werden. Der zusätzliche Verarbeitungsschritt kostet natürlich etwas Zeit, was auf Kosten des Nutzererlebnis geht. Deswegen sollte der Adapter als Provisorium gesehen werden, bis digitransit von Haus aus einen anderen Geocoder unterstützt. Das Projekt ist auf [GitHub](https://github.com/stadtulm/photon-pelias-adapter) zu finden. 
