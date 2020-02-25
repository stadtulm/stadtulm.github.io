---
title: Smarte Schlösser tracken
layout: post
author: Maximilian Richt
excerpt: Wir finden heraus, wie wir möglichst einfach mit Schlössern kommunizieren können.
---

Aktuell verbauen wir für den Test an unseren Rädern simple, durchaus schwere, Zahlenschlösser. Jedoch wären Schlösser, die sich von selbst öffnen können, ziemlich praktisch -- und so haben wir doch wieder ein Auge auf [die fertigen "smarten" Schlösser, die wir vor einiger Zeit schonmal zerlegt hatten](https://radforschung.org/log/oh-zwei-schloesser/) geworfen.

Leider hatten wir bei dem Schloss wieder ein Befestigungsproblem, mit dem auch schon andere Bikesharer zu kämpfen hatten — es passte in unseren Versuchen einfach nicht an den Rahmen am Hinterrad, zudem war bei unseren Fahrradmodellen auch wieder die Hinterradbremse im Weg. 

![](/assets/images/blog/20200225-bikelock.jpg)

In Unterstützung durch den ADFC wurde dann ein Stück Gepäckträger gekürzt und das Rücklicht nach unten versetzt, sodass das Schloss jetzt einigermaßen stabil hinten am Fahrrad hängt.

An der Integration in den modularen OpenBike-Software-Stack sind wir gerade dran. So bauen wir gerade einen Adapter für [cykel](https://github.com/stadtulm/cykel), um die Kommunikation mit dem Schloss unabhängig von der eigentlichen Sharingsoftware betreiben zu können. Das Repository und erste Tests für die Protokollimplementation gibt es unter [cykel-lock-b10](https://github.com/stadtulm/cykel-lock-bl10).

---

Wer jedoch nur mit dem Schloss reden möchte, ohne gleich ein ganzes cykel + adaptern aufsetzen zu müssen, oder einfach flott experimentieren möchte, kann auf [Traccar](https://www.traccar.org) zurückgreifen. Das ist ein Open Source System zur Kommunikation mit einer ganzen Reihe von unterschiedlichen GPS-Trackern, meist eher mit (Auto)fahrzeug-Fokus. 

Traccar hat uns auch geholfen, um das Protokoll des Schlosses grundlegend zu verstehen. In den Logs taucht dort die komplette Kommunikation als Hexdumps auf, was auch super für test-driven Development ist, um herauszufinden, ob unser Parser schritt-für-schritt die Pakete richtig geparst bekommt.

Um selbst mit Traccar und einem Schloss loslegen zu können, reicht die [Installation von Traccar nach Anleitung](https://www.traccar.org/quick-start/) auf einer Maschine mit öffentlicher IPv4-Adresse.  
Zudem braucht das Schloss eine SIM-Karte, die in den Netzen am Verwendungsort funktioniert und Konfiguration für die Adresse und der Port des eigenen Traccar-Servers. Das wechseln der Simkarte hat [der Hersteller gut bebildert dokumentiert](https://drive.google.com/file/d/1lWF9dB44C2lA412ZPyIhHhqCryF2EM_7/view). 

Die Konfiguration erfolgt entweder [per SMS](https://drive.google.com/file/d/10_dVsOtDC9BFYPz77m9BPHLC086wCIoR/view) (haben wir bislang nicht getestet) oder mit dem mitgeliefertem USB-Serial-Kabel: dazu den blauen Draht an den schwarzen halten, um das Schloss neuzustarten und die serielle Schnittstelle zu aktivieren. Mit 115200,8N1 verbinden, danach mittels `AT^GT_CM=SERVER,0,<IP>,5023,0` (_Format: Server, IP(0) oder DNS(1), IP/DNS-Name, Port, TCP(0) oder UDP(1)_) den eigenen Traccar-Server hinterlegen. Traccar verwendet für das BL10 den Port 5023 um das [kompatible gt06-Protokoll](https://www.traccar.org/devices/) zu sprechen.

Das Schloss kann dann als neues Gerät in Traccar angelegt werden, die Gerätekennung ist die IMEI.

![](/assets/images/blog/20200225-traccar.jpg)

Über die Upload-Symbol-Schaltfläche über der Geräteliste lassen sich Befehle an das Schloss senden. Im auftauchenden Dialog kommt man über _Neu > Benutzerdefinierer Befehl_ dann zum Eingabefeld für das eigentliche Kommando: `UNLOCK#`

---

Lust darauf, selbst mehr vom Schlossprotokoll zu verstehen? In [cykel-lock-b10](https://github.com/stadtulm/cykel-lock-bl10) gibt es den Protokollaufbau und Tests dafür. Mehr Infos zu den Innereien und der verbauten Firmware [haben wir schon mal aufgeschrieben](https://radforschung.org/log/schloss-am-handgelenk/). Wir sind auch weiterhin noch auf der Suche nach möglichst allen AT-Kommandos, die das Schloss versteht - falls ihr noch mehr findet, freuen wir uns gern über eine Nachricht.
