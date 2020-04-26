---
title: Eine Stadtverwaltung lernt mobiles Arbeiten, Teil 3. BigBlueButton in groß
layout: post
author: Stefan Kaufmann, Constantin Müller, Maximilian Richt, Katharina Schweiger
excerpt: (die #coronalearnings von z/da, eine blogpostserie)
---

(die #coronalearnings von z/da, eine blogpostserie)

Dieser Teil sollte schon vor einer Woche erscheinen, aber es war einfach so viel zu tun, dass für das Aufschreiben zu wenig Zeit war. [Auf verschwoerhaus.de](https://verschwoerhaus.de/was-wir-gerade-tun/) wurde es schon kurz angeteasert und [auf netzpolitik.org konnten wir das Projekt auch vorstellen:](https://netzpolitik.org/2020/ulm-baut-offene-bildungsinfrastruktur-fuer-schulen/) Gemeinsam mit der [Abteilung Bildung und Sport](https://www.ulm.de/global/datenpool/organisationseinheiten/stadt-ulm/bildung-und-soziales/bildung-und-sport) der Stadt Ulm rollen wir derzeit als Lückenfüller einen [BigBlueButton](https://en.wikipedia.org/wiki/BigBlueButton)-Pool für diejenigen Schulen in Ulm aus, die nicht über andere Werkzeuge wie z.B. den derzeit landesweit ausgerollten Pool abgedeckt werden.

Der Aufschlag dazu fand Anfang der Osterferien statt, und der Leiter der Abteilung war recht pragmatisch in seinem Auftrag. Ein neuer Kollege bei Bildung und Sport, der am 1. April mit der Arbeit anfing, fand folgenden Auftrag für das kleine, gemeinsame Team:

> Es wurde […] die Apollo 13 Metapher bemüht. Wir sollen jetzt in einem kleinen Team den „CO2 Filter“ bauen und […] eine Einkaufsliste geben was wir brauchen

Die Einkaufsliste war schnell gestrickt, und so fingen wir an, den Rollout umzusetzen. Auf dem Weg durften wir uns intensiv mit [den Leuten](https://social.tchncs.de/@angry/104037839682796290) austauschen, die den [doch recht mächtigen Rollout](https://www.kuketz-blog.de/bigbluebutton-als-virtuelles-klassenzimmer-in-baden-wuerttemberg/) des Baden-Württemberg-weiten BBB-Systems praktisch umsetzen, und auch von Seiten des [OMI](https://www.uni-ulm.de/in/omi/) und des [kiz der Uni Ulm](https://www.uni-ulm.de/einrichtungen/kiz/) gab es wertvollen Input, wie deren unieigenes BBB-Pool ausgerollt und verwaltet wird.

(Einschub: An der Stelle möchte ich noch einmal den riesigen Vorteil von Freier Software und Open Infrastructure hervorheben: Hier arbeiten sehr viele helle Köpfe auf hohem Niveau an ihren jeweiligen Infrastrukturen, und durch den gemeinsamen Austausch und das gegenseitige voneinander-lernen wird sehr viel mehr daraus als die Summe der einzelnen Teile!)

Schon seitdem wir unser „kleines“ BBB im März aufgebaut hatten, haben wir Anfragen auf verschiedensten Kanälen bekommen, wie andere ein BBB-Setup nachbauen können und wo die Grenzen liegen. Wir haben uns gedacht, dass wir an dieser Stelle sowohl einmal zeigen, wie wir unser Einzel-Server-Setup gebaut haben, und wie skalierende Varianten mit Pools aussehen können. Danach folgen eine kleine FAQ und die Probleme, denen wir begegnet sind.

Zwei Dinge sind vorneweg jedoch wichtig: Wir betreuen und betrachten mit unserer Brille vorwiegend die _technische_ Seite. Zu guter Lehre gehört viel viel mehr als eine Online-Lernumgebung – so wie allein ein Klassenzimmer auch keinen guten Unterricht garantiert. Online-Lehre ist ein ganz eigenes Format, das separater Vorbereitung bedarf. Den Unterricht 1:1 ins Netz zu übertragen, wird in der Praxis nur mäßig zufriedenstellende Ergebnisse erzielen. Das wiederum ist aber kein spezifisches BBB-Problem, sondern es trifft auf quasi alle vergleichbaren Systeme zu.  
Zweitens, wir waren in der letzten Woche vom großen Andrang auf das BBB-Schulsystem sehr überwältigt. Wir dachten anfangs, hier vor allem eine Stopgap-Lösung zu bauen. Ich bin _sehr_ auf die kommenden Tage gespannt, in denen wir Stück für Stück die Anmeldungen onboarden werden – und dann sehen, wie die Lastverteilung im Tagesgang wirklich aussieht. Das ist der Part, der sich nicht vorab testen lässt. Mal schauen, wie gut ich heute abend schlafen kann.

Nun aber zur kleinen, quick-and-dirty-Lösung.

## Ein Server, ein BBB, viele Leute.

Das erste Setup war recht schnell installiert. Es handelt sich um eine kleine [Proxmox](https://de.wikipedia.org/wiki/Proxmox_VE)-virtualisierte Maschine im Verschwörhauskeller, mit sehr überschaubaren Ressourcen (4 CPUs, 8 GB RAM). Die zugrundeliegende Maschine, auf der die VM läuft, ist ein Dell R610 mit 8× Intel Xeon X5570 bei 2.93 GHz. Insgesamt hat die Host-Maschine 48 GB RAM.

[Max](https://robbi5.de) sagt scherzhaft, dass die VM im Vergleich zu anderen Hostsystemen auf Raspberry-Pi-Niveau ist (wenngleich wir nicht empfehlen würden, solch eine Umgebung auf einem RasPi umzusetzen). In der Spitze waren auf dieser Maschine 58 NutzerInnen größtenteils mit aktiver eigener Webcam aktiv, und das war auch das einzige Mal, dass die CPU an ihre Grenzen kam. Der Traffic lag in der Spitze bei 45 Mbit/s ausgehend und 9,6 Mbit/s eingehend. Wenn alle Teilnehmenden mit Video in der Konferenz sind, sind sowohl der Datenratenbedarf als auch die CPU-Auslastung relativ hoch; wir hatten auch schon Sessions mit 23 Leuten bei 10 Mbit/s outbound und 2.25 Mbit/s inbound. Your mileage may vary.

Installiert wurde die Maschine mit dem [bbb-install](https://github.com/bigbluebutton/bbb-install)-Script auf Ubuntu 16.04 LTS – ja, BBB funktioniert derzeit offiziell exakt nur auf dieser veralteten Version, was Folgeprobleme z.B. mit Python-Versionen nach sich zieht. Dazu später mehr.

Wichtig beim Rollout ist auch, die _Möglichkeit_ zur Aufzeichnung von Sessions von vorneherein zu deaktivieren. Tatsächlich [finden nämlich standardmäßig Aufzeichnungen aller Sessions statt](https://github.com/bigbluebutton/bigbluebutton/issues/92029), und der Recording-Knopf setzt hier nur Schnittmarken und startet nach Ende der Session eine Nachbearbeitungspipeline. Das von vorneherein zu deaktivieren schafft bessere DSGVO-Compliance und schützt auch vor vollaufenden Platten.

### TURN und STUN zur Firewalldurchbohrung

Da nicht nur „unsere“ Verwaltung in Ulm, sondern auch andere Behörden und Projektpartner mittlerweile diese Instanz nutzen, ist es sehr wichtig, durch Firewalls durchzukommen, die noch auf Sicherkeitskonzepten von vor 30+ Jahren stammen (alles ist aus Prinzip erstmal zugenagelt). Nach einigem Herumprobieren läuft daher gerade auf einer seperaten VM ein [coturn](https://github.com/coturn/coturn), das [STUN](https://de.wikipedia.org/wiki/STUN) und [TURN](https://en.wikipedia.org/wiki/Traversal_Using_Relays_around_NAT) auf Port 443 abwickelt. Damit ließen sich zum Erstaunen mancher Gegenstelle sogar bundesministerielle Firewalls durchbohren, bei denen selbst Jitsi bislang nicht funktionierte. Das ist natürlich andererseits wieder der Vorteil einer so eindimensionalen behördlichen IT-Sicherheit ;)

(Mehr zu dieser Besonderheit im skalierenden Ansatz weiter unten, hier gibt es ggf. Fallstricke).

### Dialin via SIP

Da im Hintergrund von BBB ein [FreeSWITCH](https://en.wikipedia.org/wiki/Freeswitch) läuft, lassen sich relativ einfach Einwahlnummern für diejenigen Menschen anbinden, die nicht so einfach ein Headset an ihren Rechner anschließen oder aus anderen Gründen Probleme mit dem Rechner-Audio haben. Hier gibt es eine große Auswahl an SIP-Telefonie-Anbietern mit lokalen Einwahlnummern. Letztlich ist aber die Zahl der eingehenden, gleichzeitig nutzbaren Kanäle begrenzt, weswegen sich hier nur einwählen sollte, wer nicht anders kann. An den Server wird _eine_ Rufnummer gebunden; die Auswahl des Konferenzraums erfolgt dann über eine PIN.

### Einfach flott rein
Im Gegensatz zu verschlossenen Konferenzlösungen für bestimmte Nutzergruppen, die meist bei der Person, die das Meeting erstellt einen Account erzwingt, haben wir bei der kleinen Lösung versucht, es so einfach wie möglich zu gestalten. Der Flow, der auch z.B. so bei Jitsi Meet funktioniert, besteht aus der Eingabe eines Raumnamens, der dann angelegt wird, falls er noch nicht existiert. Danach folgt die Eingabe des eigenen Namens. Das Meeting bekommt einen URL, den die Teilnehmenden besuchen können und nur noch ihren Namen eingeben müssen. Ein Vorteil dieser Lösung im Gegensatz beispielsweise zu [greenlight](https://github.com/bigbluebutton/greenlight) liegt auch darin, dass man sich mit dieser Schmalspurlösung nicht sofort Docker und Datenbanken und E-Mail-Versand und sonstige weitere Abhängigkeiten eintritt.

Das BBB-Frontend dafür ist im Wesentlichen wenig anderes als eine angepasste Variante der HTML5-Demo von BBB. Wir haben es als [bbb-easy-join](https://github.com/stadtulm/bbb-easy-join) veröffentlicht.

### Monitoring, ein Auge drauf haben

![Monitoring-Graph](/assets/images/blog/20200426_mon1.png)

[Monitoring](https://mon.service.verschwoer.haus/d/HIbd_CXZz2/bigbluebutton-server-instance) mit hübschen Graphen ist _immer_ toll! Deswegen haben wir auch in der kleinen Lösung Prometheus und Grafana laufen, die ihre Daten über [bbb-exporter](https://github.com/greenstatic/bigbluebutton-exporter) bekommen. Dort gab es vor Kurzem auch [einen Pull Request](https://github.com/greenstatic/bigbluebutton-exporter/pull/10) eines ~alten~ jungen Bekannten aus dem Jugend-hackt-Kosmos, um zu sehen, wie viele Menschen per SIP eingewählt sind.

## Viele Server, viele BBBs, noch mehr Leute

Irgendwann ist aber auch auf „handelsüblichen“ Servern die Kapazität zu Ende. Das lässt sich auch nur begrenzt durch mehr CPU-Power beheben. Aufgrund der Systemarchitektur mit [node.js](https://de.wikipedia.org/wiki/Node.js) – einer Kernkomponente, die sich um den Chat, das Whiteboard, aber auch das Signaling der Status der Teilnehmenden kümmert – laufen hier wesentliche Teile des Setups weiterhin single-threaded. Es gibt ein [Issue zum Aufteilen](https://github.com/bigbluebutton/bigbluebutton/pull/8788) und ein weiteres Issue, um die [veraltete nodejs-version hochzuziehen](https://github.com/bigbluebutton/bigbluebutton/issues/8937), was das Gesamtsetup hoffentlich etwas performanter macht.

Egal wie sie aber nun beschaffen sind: Irgendwann braucht es mehr als einen BBB-Server. Aus der BBB-Welt gibt es dafür [scalelite](https://github.com/blindsidenetworks/scalelite) für die Lastverteilung. Wie das in etwa aussieht, zeigt das folgende Bild:

![Andreas Grupp / CC BY-SA (https://creativecommons.org/licenses/by-sa/4.0)](/assets/images/blog/20200426_scalelite.png) 
Skizze von [Andreas Grupp auf Wikimedia Commons](https://commons.wikimedia.org/wiki/File:BBB-Scalelite-Infrastruktur.png) / [CC BY-SA](https://creativecommons.org/licenses/by-sa/4.0)

Im Hintergrund steht ein Pool an Servern, die sich jeweils am scalelite anmelden. Der Einstieg der Teilnehmenden erfolgt dann über das Frontend der Wahl – das kann z.B. das erwähnte greenlight sein, aber auch Moodle oder andere passende Plattformen. Scalelite weist dann im Hintergrund der angefragten Session einen Poolserver mit Kapazität zu und dann kann es losgehen.

### Automatisieren, automatisieren, automatisieren (und vielleicht virtualisieren)

Um diesen Pool hochzuziehen, haben wir uns diverse Ansible-Playbooks zum Vorbild genommen. Neben den [Playbooks der Uni Ulm](https://release.bbb.uni-ulm.de) und [der Landeslösung](https://codeberg.org/DigitalSouveraeneSchule/bbb) haben wir vor allem [das Skript von n0emis](https://github.com/n0emis/ansible-role-bigbluebutton) integriert – noch so ein junger Bekannter aus dem Jugend-hackt-Kosmos :) Das [letzliche Playbook](https://github.com/stadtulm/a13-ansible) ermöglicht es uns, innerhalb von rund 20 Minuten aus einer blanken Maschine einen zusätzlichen Poolserver zu bauen. Für weitergehende Ansätze mit Virtualisierung und Orchestrierung fehlte bislang leider die Zeit.

![Monitoringgraph von ulmlernt](/assets/images/blog/20200426_mon2.png)

Auch hier gibt es ein [Monitoring](https://mon.ulmlernt.org/d/HIbd_CXZz/bigbluebutton-all-servers), um einen Überblick über den Status und die Auslastung der Pool-Server zu haben. Wir werden uns hier Schritt für Schritt mit den praktischen Daten herantasten müssen – wir wissen grob, wie viel Last ein einzelner Server abkann (die Doku spricht von rund 150 Teilnehmenden, die Uni Ulm meldete Erfahrungen mit bis zu 300 TN), die tatsächliche Last ist aber eine Funktion aus zeitlicher Verteilung, tatsächlicher Benutzung der Räume und wie viele Menschen dabei ihr Video laufen haben. Das wird sich nur in der Praxis ermitteln lassen.

### Grenzen, nichttechnisch

Zunächst das Offensichtliche: Alles, was wir hier machen, ist ein Notbehelf. Wir können nicht binnen einer Woche Onlinelehre aus dem Nichts schaffen – und das ist keine Werkzeug- sondern eine Strukturfrage. Ich bin mir sicher, dass in den kommenden Wochen Kiebitzereien von der Seitenlinie kommen werden, warum nicht $XYZ genutzt wurde, weil das ja viel besser gehe. 

Kern des Problems ist und bleibt aber: Eine Klassenzimmersituation _kann_ nicht 1:1 ins Netz übertragen werden, und Onlinedidaktik erfordert ganz andere Vorgehensweisen als Klassenzimmerunterricht. Diese Erfahrung wird nicht viel anders sein als die in der Geschäftswelt, wo in den vergangenen Wochen versucht wurde, klassische Präsenzarbeit mit den Werkzeugen verteilten Arbeitens 1:1 umzusetzen – das geht nur mäßig gut, und _effizientes_ verteiltes Arbeiten erfordert ganz andere Paradigmen.

Ich habe aber viel Begeisterung und Umsetzungswillen von den ersten alphatestenden LehrerInnen gesehen, und hoffe, dass die Kollegien der teilnehmenden Schulen sich intern und untereinander weiterhelfen und gegenseitig fortbilden. Dass das leichter gesagt als getan ist, ist mir schmerzhaft bewusst. [Bei der LehrerInnenfortbildung BW entsteht derzeit ein Katalog an Handreichungen](https://lehrerfortbildung-bw.de/st_digital/medienwerkstatt/dossiers/bbb/), den ich sehr empfehlen kann. Der Fokus auf gezielt eingesetzte Webinare und Gruppenarbeiten sowie Flipped Classrooms, ergänzt durch Sprechstunden, erscheint mir ein guter Ansatz – denn jetzt kommen wir ins technische.

### Grenzen, technisch, allgemein

Die Annahme, dass Online-Unterricht analog zum Klassenzimmer funktionieren kann, kommt sehr schnell schon technisch an Grenzen. Ich zitiere hier verkürzt aus einer Überschlagsrechnung, die mir zur Verfügung gestellt wurde: In ganz Baden-Württemberg finden pro Woche rund 2,2 Millionen Unterrichtsstunden statt. Selbst bei gleichmäßiger Verteilung auf 8 Stunden am Tag über 5 Tage die Woche entspräche dies 55.000 gleichzeitiger Videokonferenz-Sessions. Die oben in der simplen Lösung beobachteten Werte decken sich mit dem, was mir vorliegt: Selbst wenn nur die Lehrkraft an 25 SchülerInnen streamt, liegen wir bei 5–10 Mbit/s outbound. Selbst wenn nun alle Unterrichtsstunden wirklich gleichverteilt über den Tag stattfinden und nur gelegentlich SchülerInnen ebenfalls ihr Video zuschalten, kommen wir so schnell in den Terabit-Bereich, um Lastspitzen abzufangen.

Und da reden wir noch gar nicht von den Peering-Problematiken zu Access-Netzen oder der Situation, wenn mehrere SchülerInnen sich einen Anschluss teilen – und schon gar nicht von Fragen sozialer Ungleichheit und wer überhaupt genügend passende Endgeräte hat.

Ich denke, dass diese Fragen auch keine der anderen Webconferencing-Lösungen beantworten kann.

Nach diesem kleinen ~Downer~ Realitätscheck nun noch ein paar andere Limitationen.

### Grenzen, technisch, BBB

Der Bauprozess für BBB ist… problematisch. Die Plattform an sich finde ich für Onlinelehre enorm stark. Man merkt der Developer-Community aber an, dass sie lange Jahre ein Nischendasein fristete, und die größere Free-Software-Welt erst jetzt – notgedrungen – so richtig auf sie aufmerksam geworden war. Wie oben beschrieben baut BBB auf Ubuntu 16.04 LTS ab, derweil seit dieser Woche schon die _übernächste_ LTS-Version veröffentlicht wurde. Die Voraussetzungen für den Bau und die Paketierung des Systems sind nur spärlich dokumentiert, [was mittlerweile durch die neugewonnene Aufmerksamkeit auch umfangreich diskutiert wird](https://github.com/bigbluebutton/bigbluebutton/issues/8861)

Daraus folgende Effekte sind auch zu sehen. Das Projekt BBB macht gerade durch die vorher nie dagewesene Aufmerksamkeit rapide Entwicklungsschritte. Alle paar Tage erscheinen neue Package-Releases, und nicht immer ist im Changelog oder durch Release-Notes erkennbar, dass für das Ausrollen der neuen Pakete Änderungen auf die URLs der passenden Repositories nötig sind.  
Ich halte es auch für wichtig, hier offen mit [bereits erkannten](https://github.com/tchenu/CVE-2020-12112/) und möglicherweise künftig noch auftretenden CVEs umzugehen. Wir arbeiten weiter daran, eine technisch möglichst umfassende Absicherung unseres BBB-Pools gewährleisten zu können. Es kann aber seriöserweise ebensowenig wie bei anderen – auch proprietären – Werkzeugen versprochen werden, dass BBB grundsätzlich „sicher“ ist. Das ist mit ein Grund, warum wir so darauf pochen, dass Webconferencing-Werkzeuge kein 1:1-Transfer von Präsenzklassenräumen sein können.

Wir hoffen alle darauf, dass BBB in der nahen Zukunft schrittweise einen Security-Audit bekommt und die vermutlich unerwartete und ungewollte Rolle als kritische Infrastruktur aktuell noch viel mehr Augen und Aufmerksamkeit auf das Projekt lenkt – damit am Ende wir alle mehr davon haben. Die Liste an Pull Requests und Verbesserungsvorschlägen ist in der Tat gleichermaßen vielfältig wie Anlass zur Freude.

Eine Baustelle, die [nicht nur uns betrifft,](https://github.com/bigbluebutton/bigbluebutton/issues/8133) sind Audioprobleme bei iOS-Geräten. Die großartigen Datenzauberer und -hexen in unserem Team arbeiten gerade an einer besseren STUN/TURN-Konfiguration, um auch dieses Problem zu lösen und danach auch zu dokumentieren. Auch hier sind aber nicht nur wir am Ball, glücklicherweise. Wenn ich nach Twitter gehe, sieht sich gerade gefühlt ein Drittel der Tech-Welt BBB näher an.

## Mehr

Gerade diese gesammelte Aufmerksamkeit ist im deutschsprachigen Raum gerade erfreulich gebündelt. [Holger](https://twitter.com/systemagie), der schon eine tragende Rolle bei der Weiterentwicklung von digitransit im verteilten Transportkollektiv einnimmt, hatte [vergangenen Montag schon zu einem Austausch](https://twitter.com/systemagie/status/1251141205494239232) im Rahmen des [digital verteilten Onlinechaos](https://di.c3voc.de/) eingeladen. Morgen abend soll es eine nächste Runde des Austauschs geben, und es existiert auch [ein Matrix/Riot-Channel](https://matrix.to/#/#bigbluebutton-de:matrix.org), der auch in den Slack der Open Knowledge Foundation gespiegelt wird. Ich kann nur alle Interessierten einladen, sich einzubringen – gemeinsam können wir noch viel mehr aus diesem Projekt machen :)

Das nun als sonntagabendlicher sehr später Aufschrieb des Status Quo. Morgen wissen wir mehr, ob uns das Ding um die Ohren fliegt. Stay tuned.
