---
title: "Gedanken zur Fahrradrückgabe, Benachrichtigungen und Authentifizierung"
layout: post
author: Constantin Müller, Maximilian Richt, Stefan Kaufmann
excerpt: Gedanken über Datensparsame Benachrichtigungssysteme für OpenBike
---

![](/assets/images/blog/20200707_bikepost.jpg)

In den letzten Wochen haben wir uns immer wieder Gedanken über die **Rückgabe** von Fahrrädern gemacht. Das ist ein Themengebiet, das erstmal recht einfach erscheint, wir durch den Bikesharing-Betrieb aber schon einiges gelernt haben und jetzt mehr spannende Fragen sehen als vorher.

Momentan ist es so, dass unsere Räder kostenlos ausgeliehen werden können. Wir beschränken dabei auch die Länge der Ausleihe/Fahrt nicht. Bei kommerziellen Anbietern gibt es meist den Anreiz möglichst kurz zu fahren (weil z.B. die ersten 30 Minuten kostenfrei sind), oder bei Nichtbenutzung die Leihe auch schnellstmöglich wieder zu beenden, da minütlich abgerechnet wird. Bei unserer kostenfreien Ausleihe bestehen diese Anreize zurzeit nicht - was dazu führt, dass wir immer wieder mehrere stunden- bis tagelange Ausleihen gesehen haben, ohne dass sich das Rad bewegt hat. Leute haben also vergessen, die Ausleihe des Rads zu beenden und somit anderen wieder zur Verfügung zu stellen.

Am naheliegensten wäre nun bei Ausleihen, die deutlich länger als die Durchschnittsausleihe dauern, die ausleihende Person zu benachrichtigen und zu erinnern, bei Nichtbenutzung das Rad auch wieder zurückzugeben. Unsere Oberfläche ist aber aktuell eine [Webapp](voorwiel). Diese können nur unter Android überhaupt (Web-)Push-Nachrichten auslösen, unter iOS ist dies aktuell gar nicht möglich.

Eine weitere Möglichkeit Nutzerïnnen über noch offene Leihen zu informieren, wären bereits bestehende Kanäle wie E-Mail, SMS oder diverse Messenger. Da wir in [cykel](cykel) aber keine Registrierung und Login selbst anbieten, sondern dies über externe Dienste per oAuth2 abbilden, bekommen wir diese Kontaktmöglichkeiten nur in seltensten Fällen. Zudem werfen wir diese Daten - falls doch übermittelt - sogar aktiv weg. Dies hilft uns dabei, möglichst datensparsam mit persönlichen Daten umzugehen, denn eine wunderbare Möglichkeit für Datenschutz ist es, wenn man Daten gar nicht erst erfasst.

Nun stellt sich jedoch die Frage, ob wir diese Prinzipien über Bord werfen müssten, um über vergessene Rückgaben informieren zu können. Zudem würden uns mehr personenbezogene Daten nur noch mehr Probleme bereiten, wenn wir das zukünftig angedachte Roaming zwischen verschiedenen OpenBike-Instanzen einbauen. In dem Fall müssten diese personenbezogenen Kontaktdaten dann mitwandern.

_Wir suchen also einen Weg, Benutzerïnnen Nachrichten zukommen zu lassen, ohne dass wir zwingenderweise einen Bezug zwischen einer Ausleihe, der Nachricht, dem Nachrichtenweg und der Person haben müssen._

Momentan benutzt die Kernkomponente von OpenBike zur Authentifizierung oAuth2. Einige Dienste, mit denen eine oAuth2-Authentifizierung möglich ist, bieten selbst auch eine Messenging-Funktion (wie z.B. Twitter mit Direktnachrichten), mit der wir benachrichtigen könnten.
Aber nicht alle Dienste bieten das an und zudem gibt es bei den einzelnen Diensten dafür auch keine einheitliche Schnittstelle. Dies ist aber auch gar nicht das Ziel von oAuth2 - dieser (und verwandte) Standards wie OpenID Connect sind ja eigentlich für die Absicherung von Schnittstellen gedacht.

Noch dazu besitzen viele Menschen ja bereits eine Menge an Möglichkeiten, über die sie erreichbar wären - neben klassisch E-Mail und SMS kämen da noch diverse Messenger, Soziale Netzwerke und andere Dienste in Betracht, die vielleicht sogar schon auf den Smartphones installiert sind und Push-Nachrichten ausspielen.

Unsere Idee wäre es nun eine weitere Komponente zu bauen, die zwischen OpenBike und den Authentifizierungsprovidern liegt und sich einerseits um die Abwicklung der Authentifizierung mit verschiedenen Providern und andererseits um das Weiterleiten von Nachrichten über diese kümmert. 
Diese Komponente sollte also können:

* selbst oAuth Provider sein
* mehrere andere oAuth Provider für den Login unterstützen
* für die Provider die es anbieten, Messaging unterstützen
* für alle anderen die Hinterlegung einer E-Mail-Adresse oder Handynummer (für SMS) ermöglichen
* der Benutzer:in wählen lassen, über welchen Weg sie kontaktiert werden will
* die Zurordnung zu einer Mailadresse, Handynummer oder Loginprovider aufheben können (für E-Mail-Addresswechsel, neue Handynummer, oder gesperrtem/neuen Account bei einem Provider)
* selbst eine Schnittstelle für Benachrichtigungen anbieten

Damit schaffen wir zwar wieder ein zentrales System, das personenbezogene Daten hortet, aber dieses wäre zumindest von den Ausleihvorgängen auf unterschiedlichen OpenBike-Instanzen unabhängig. Das ist auch der Grund, warum wir dies nicht direkt in [cykel](cykel) einbauen möchten - einerseits sollte diese Kommunikationskomplexität mit zig unterschiedlichen Anbietern nicht in die eigentliche Bikesharingsoftware rein, andererseits sollen auch die personenbezogenen Daten möglichst weit weg davon. Nehmen wir einen kollektiven Betrieb von vielen kleinen OpenBike-Instanzen in unterschiedlichen Nachbarschaften und Städten an, würde ein zentraler Service, der sich um dieses Problemfeld kümmert immerhin den Fokus auf Datenschutz und entsprechende Regelungen an dieser einen Stelle bündeln. Trotzdem können natürlich unterschiedliche Instanzen auch ihre eigenen Login/Notificationservices hosten.

Wir sind uns aber auch nach viel drüber grübeln noch nicht ganz sicher, ob das die sinnvollste Variante ist, das Notificationproblem zu lösen. Auch welche Protokolle dort in Betracht kommen könnten - aktuell klingen zumindest oAuth2 und ActivityPub interessant - ist noch offen.

Wir freuen uns daher über weitere Ideen, Anmerkungen oder auch Vorschläge das Ganze komplett anders zu machen. Gerne per Mail an openbike@ulm.dev - oder sogar besser im öffentlichen [`#transport-bikesharing:matrix.org` Matrix-Channel](https://matrix.to/#/!ghOLficeAycydtkZtA:matrix.org?via=matrix.org). 
