---
title: "Converter von IXSI nach GBFS für Carsharing"
layout: post
author: Constantin Müller, Maximilian Richt
excerpt: Ein GBFS-Converter für Carsharing-Daten
---

![Carsharingauto an der Mobilitätsstation am Eselsberg in Ulm](/assets/images/blog/20210903-carsharing.jpg)

Dass wir für Bikesharing, wie unserem [selbstgebauten OpenBike](https://ulm.dev/projects/openbike/), auf GBFS - der internationalen [_General Bikeshare Feed Specification_](https://github.com/NABSA/gbfs) - setzen ist vielleicht bekannt. Aber was genau ist IXSI jetzt? Das _Interface for X-Sharing Information_, kurz IXSI, ist ein mit Fördermitteln unter anderem an der [RWTH Aachen entstandener](https://www.researchgate.net/publication/277020604_IXSI_-_Interface_for_X-Sharing_Information) Standard zum Austausch zwischen Sharingdiensten und Auskunftsdiensten und wird hauptsächlich von der deutschen Carsharing-Industrie benutzt um Daten zwischen den einzelnen Carsharern austauschen zu können.

Im Rahmen der [Mobilitätsstation am Eselsberg](https://www.zukunftsstadt-ulm.de/eselsberg/mobilitaets-station) fordern wir von den Anbietern, welche dort Mobilitätsdienste anbieten wollen (und zukünftig auch für alle Mobilitätsanbieter, die in der gesamten Stadt anbieten wollen) die Bereitstellung von offenen Daten zur Verfügbarkeit der Angebote.

Für Bikesharing und Scootersharing fordern wir bereits [in unserer Kooperationsvereinbarung [PDF]](https://www.ulm.de/-/media/ulm/vgv/vp/downloads/escooter/kooperationsvereinbarung-fr-etretrollersharing-in-ulm.pdf) GBFS. Zwar wurde GBFS ursprünglich für Bikesharing entwickelt, aber es stellt sich schnell raus, dass sich die Auskunft von Bikesharing und anderen Verkehrsmitteln wie Leihautos gar nicht so stark unterscheidet. Zudem sind bereits für die zukünftige GBFS-Version 3.0 [Erweiterungen für Carsharing](https://github.com/NABSA/gbfs/pull/350) und eine [Umbenennung zu weniger »Bike«, mehr »Vehicle«](https://github.com/NABSA/gbfs/pull/354) geplant. Daher eignet sich GBFS aus unserer Sicht bereits jetzt dazu, auch offene Daten zu Carsharing auszuliefern.

Offene Daten im Carsharingmarkt sind wichtig - und leider zurzeit immer noch sehr rar. Die technischen Dienstleister der Carsharer und vorallem der Bundesverband Carsharing befinden sich, wie vor ein paar Jahren auch die meisten Verkehrsverbünde, noch im Glauben, dass offene Daten sie ins Verderben stürzen und ihre Geschäftsmodelle daran zerbrechen würden.
Eine stichhaltige Begründung für diese Sorge konnte uns bis heute niemand liefern und wir glauben sogar, dass das Gegenteil der Fall ist:
Um so mehr Menschen von einem verfügbaren Mobilitätsangebot wissen, um so mehr sind bereit es zu nutzen. Offene Daten könen genau für diese Sichtbarkeit der Angebote in Karten und Mobilitätsapps sorgen. Um so verbreiteter der Standard der Daten, um so einfacher ist es, diese Daten in vorhandene Anwendungen zu integrieren. Genau deswegen setzen wir hier auf das international verbreitete GBFS. Aber auch Prototypen und Nischenanwendungen profitieren hier von Open Data und verbreiteten Standards, da so auf vorhandene Programmierbibliotheken gesetzt werden kann oder Lösungen aus anderen Städten in neue Regionen übertragen werden können ohne dass diese aufwendig neu entwickelt werden müssen.[^digitransit]

Und am Ende sitzt die Kundin dann sowieso im Fahrzeug des Anbieters - von einem vielbeschworenen drohenen »Verlust des Kundenkontakts« kann also wirklich nicht die Rede sein.

Die Software der in Ulm ansässigen Carsharing-Anbieter unterstützt leider noch kein GBFS, und der Softwareanbieter war nicht bereit, selbst GBFS zu implementieren. Um die Bereitstellung von GBFS als Open Data trotzdem zu ermöglichen, haben wir uns dann also darauf geeinigt, dass uns die vorhandene IXSI-Schnittstelle bereitgestellt wird. Auf deren Basis entwickelten wir dann eine Software, welche die IXSI-Daten regelmäßig abfragt, nach GBFS konvertiert und öffentlich zugänglich macht.

Das Ergebnis sind nun zwei GBFS-Endpunkte für die beiden Carsharing-Anbieter, welche ihre Leihautos an der Mobilitätsstation anbieten. Die SWU stellt darüber hinaus die Daten für alle ihre Carsharing-Stationen bereit, der Anbieter Conficars leider nur für die Station am Eselsberg.

Die GBFS-Endpunkte sind unter den folgenden URLs abrufbar:
* Conficars: [`https://gbfs.conficars.de/gbfs.json`](https://gbfs.conficars.de/gbfs.json)
* SWU2go: [`https://ixsi.swu.de/gbfs.json`](https://ixsi.swu.de/gbfs.json)

Den Sourcecode des IXSI-GBFS-Converters stellen wir natürlich auch als OpenSource Software  unter der [freien MIT-Lizenz](https://github.com/stadtulm/ixsi-gbfs-converter/blob/main/LICENSE) auf [https://github.com/stadtulm/ixsi-gbfs-converter](https://github.com/stadtulm/ixsi-gbfs-converter) zur Verfügung.
Ein Einsatz des IXSI-GBFS-Converters ist entweder direkt als node.js-Anwendung aus dem Sourcecode oder mit den vorbereiteten Docker-Images unter [https://hub.docker.com/r/stadtulm/ixsi-gbfs-converter](https://hub.docker.com/r/stadtulm/ixsi-gbfs-converter) möglich.

Wir freuen uns auch über gemeinsame Weiterentwicklung des Adapters. Vorallem bei der aktuellen Entwicklungsgeschwindigkeit des GBFS-Standards werden sicher weitere Datenfelder auftauchen, die aus IXSI-Daten befüllt werden können. Aktuell fehlen auch noch die mit GBFS 2.1 eingeführten [vehicle types](https://github.com/NABSA/gbfs/blob/v2.2/gbfs.md#vehicle_typesjson-added-in-v21). Zudem ist der Adapter aktuell nur für Stationsbasierte Systeme implementiert - mangels Free-Floating-Carsharer in Ulm. Falls also eine IXSI-Schnittstelle mit Freefloatern auftaucht, freuen wir uns besonders über Pull-Requests, die diese auch implementieren.

Außerdem würden wir uns freuen, wenn weitere Kommunen diesem Beispiel folgen und offene Mobilitätsdaten von den Anbietern einfordern und als Vorraussetzung für den Betrieb von Sharingdiensten machen. Eine [Policy-Anleitung](https://mobilitydata.org/gbfs-and-shared-mobility-data-policy-in-europe/) hat MobilityData in den letzten Tagen veröffentlicht, die technischen Vorraussetzungen sind nun mit dem Adapter geschaffen - auch wenn wir hoffen, dass die technischen Dienstleister der Carsharer doch noch einlenken und GBFS direkt in ihre Software einbauen. Bis dahin kann der Adapter auch von den Carsharinganbietern betrieben werden.

Wenn ihr eine Software entwickelt oder betreut, die mit GBFS umgehen kann, integriert doch gern die Daten. Dies hilft auch mehr Bürger\*innen und Entscheider\*innen zu zeigen, was man mit den Daten alles machen kann - und den Anbietern, dass das alles gar nicht weh tut.

Und natürlich freuen wir uns, wenn ihr dieses Beispiel nutzt um in eurer Kommune offene Mobilitätsdaten anzusprechen.


[^digitransit]: Ein Beispiel dafür ist [digitransit](https://ulm.dev/projects/digitransit/), welches wir ohne viel Aufwand oder Neuentwicklung für Ulm anpassen konnten.
