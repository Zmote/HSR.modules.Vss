======================================
VSS FS13 Repetitionsfragen Antworten
======================================

Dieses Dokument wird vorzu erweitert. Ergänzungen und Antworten sind herzlich willkommen.
Repetitionsfragen: https://github.com/moonline/Vss/blob/master/RepetitionQuestions.Vss.tex


Systemmodelle
=============
1) Architekturmodelle zeigen wie die (verteilten) Komponenten zusammenarbeiten

2)
	Client / Server
		Clients stellen Anfragen an einen zentralen Server / initiieren einen Prozess, der Server Antwortet / liefert den Clients Ergebnisse
	Proxy Server & Cache
		Clients stellen Anfragen, diese laufen über einen Proxyserver mit einem Cache. Teilweise erhalten die Clients die Antwort, teilsweise leitet der Proxy Server die Anfrage and den originalen Server weiter.
	P2P
		Alle Kommunikationsteilnehmer sind gleichberechtigt. Oft gibt es einen Leader, dieser ist jedoch austauschbar
	Applets
		Der Client lädt sich vom Server ein Applet herunter. Anschliessend kommuniziert der Client nur noch mit dem Applet und das Applet kommuniziert im Hintergrund mit dem Server, ohne das der Client dies explizit wahrnimmt.
	Thin Client / Fat Server
		Die Clients besitzen selbst keine Software und Daten oder nur wenige. Der Grossteil der Rechenarbeit und Daten stellen die Server bereit. Je nach Ausführung schwank die Verteilung zwischen "Server stellt nur Daten bereit" und "Server stellt Daten und Applikationen bereit, Client ist nur noch UI".
	MVC Pattern
		Die Daten, Model und Controller liegen auf dem Server, die Präsentation beim Client
	Three Tier
		::
		
			Client     [ Präsentation  ]
			- - - - - - - - - -|- - - - - - 
			Server     [ Businesslogik ]
			                   |
			           [     Daten     ]
	
	
	ISO OSI Layer Model
		Aufteilung in Prozesse (Anwendungen), Transport und Netzwerk
		
3) Allgegenwärtige Systeme, immer verfügbar und überall. Dies zieht Kensequenzen wie stark schwankende Netzqualität, Verfügbarkeit, Energieversorgung, ... mit sich.

4) Interaktionsmodell: beschreibt die Kommunikation zwischen den Komponenten. Es gibt keine globale Uhr, nur Nachrichten.

5) Fehlermodell: Beschreibt, was bei einem Fehler passiert und wie die Komponenten darauf reagieren

6) Sicherheitsmodell: beschreibt Bedrohungen und Sicherheitsmassnahmen zur Sicherung der Kommunikation und der Komponenten

7) RM-ODP: Metamodell für verteilte Systeme. Beschreibt ein verteiltes System von verschiedenen Viewpoints aus um die Komplexität zu verringern. Mögliche View Points:
	* Informations / Datensicht: Datenmodell, Constraints
	* Computational- / Logiksicht: Komponenten und Interfaces
	* Engineeringsicht: Netzarchitektur, Client- / Server Aufteilung
	* Technologysicht: Abbildung der Architektur auf echte Hardware
	
	
Interprozesskommunikation
=========================
8)
	synchron
		Sender sendet Message und wartet, bis Antwort eintrifft. -> blockiert
	asynchron
		Sender sendet Message, legt Buffer für Antwort an und holt sie später ab. -> blockiert nicht
		
9)
	Ports
		Schnittstellen an einem Host
	Sockets
		Endpunkt der IP, TCP/UDP Kommunikation, ist immer an einen Port gebunden, wird durch ein File im Hintergrund repräsentiert.
		
10)
	IP
		Verbindungslose Kommunikation: Best Effort, Keine Übermittlungsbestätigung. Keine Übertragungsgarantie. Pakete können verloren gehen, doppelt oder in falscher Reihenfolge ankommen. Fehler aus tieferen Layern treten auch in IP auf.
	TCP
		Verbindungsbehaftete Kommunikation: Bestätigung und Retransmit bei verlorengegangenen. Reihenfolge garantiert.
	UDP
		Verbindungslose Kommunikation: Übertragung nicht garantiert, keine Bestätigung dafür weniger Overhead.
		
11) Pakete können verloren gehen, verworfen werden, doppelt oder in falscher Reihenfolge ankommen
		
12) UDP besitzt wesentlich weniger Overhead als TCP. Spielt die übertragene Datenmenge eine grössere Rolle als die vollständige Übertragung, so ist UDP geeignet. z.B. Multimedia Streaming, VoIP

13)
	UDP:
		.. code-block:: java
		
			// sender
			try {
				DatagramSocket socket = new DatagramSocket();
				String message = "Something to send...";
				DatagramPacket p = new DatagrammPacket(message.getBytes(), (message.getBytes()).length, InetAddress.getByName(receiverHost), receiverPort);
				try {
					socket.send(p);
				} catch (IOException e) { e.printStackTrace(); }
				socket.close();
			} catch (Exception e) { System.err.print(e); }
			
			// receiverHost
			try {
				DatagramSocket socket = new DatagramSocket(port);
				buffer byte[] new byte[512];
				String message = "";
				String messagePart;
				do {
					DatagramPacket p = new DatagramPacket(buffer, buffer.length);
					socket.receive(p);
					messagePart = new String(buffer).trim();
					message += messagePart;
				} while (!messagePart.equals("."));
				System.out.println(message);
				socket.close();
			} catch (Exception e) { System.err.print(e); }
			
			
	TCP
		.. code-block:: java
		
			// sender
			try (Socket socket = new Socket()) {
				BufferedOutputStream bos = new BufferedOutputStream(socket.getOutputStream());
				socket.connect();
				String message = "Somethin to send...";
				bos.write(message);
			} catch (Exception e) { System.err.print(e); }
			
			// receiver
			try (ServerSocket socket = new ServerSocket(serverPort)) {
				while(true) {
					try (ClientSocket client = socket.accept()) {
						DataInputStream in = client.getInputStream();
						DataOutputStream out = client.getOutputStream();
						
						String message = in.readUTF();
						out.write("Successfull received: "+message);
						
						System.out.println(message);
						client.close();
					} catch (Exception e) { System.err.println(e); }
				}
				socket.close();
			} catch (Exception e) { System.err.print(e); }
			
			
14) Sie führen dazu, dass das Programm auf send() und receive() warten muss und blockiert ist. Besser: asynchrones send() benutzen.

15) Sind Timeout und maximale Anzahl Retransmit ausgeschöpft, bricht TCP ab. Somit können auch bei TCP Pakete verloren gehen.

16) 
	Multicast
		Empfänger melden sich bei einer Gruppe an. Nachrichten an die Gruppe werden automatisch allen Teilnehmern zugestellt. Gruppenkommunikation basiert oft auf Multicasting. Multicast muss von den Netzwerkkomponenten unterstützt werden.
	IP Multicast
		Verwendung von speziellen Adressen (224.0.0.1 bis 239.255.255.254) um Gruppen zu identifizieren. Verteilt wird die Nachricht von speziellen Multicastroutern. Begrenzung möglich durch Angabe der maximal durchlaufbaren Router.
		
17)
	Marshalling
		Daten mit einer internen Struktur werden in eine externe Struktur übersetzt (z.B. für den Versand in einem UDP Paket):
	Un-Marshalling
		Daten einer externen Struktur werden in eine interne übersetzt.
		
18)
	direkte Socketnutzung
		Exakte Steuerung der Sockets, möglicherweise effizienter
	Nutzung durch Middleware
		Abstraktion, keine loq-level Programmierung, universeller

19) Die Objekte werden zerlegt und als Textform repräsentiert. Serialisierung kann genutzt werden, um Objekte über das Netzwerk zu übertragen.
		Nachteile: Private Member sind sichtbar, Objektversionen können Ärger machen, Referenzen Serialisieren ist schwierig (möglicherweise sind die Referenzierten Objekte nicht mehr da bei der reSerialisierung)
		
20)

21) 
	* Serialisierbare Klassen müssen Serializable sein. 
	* Mit transient können nicht zu serialisierbare Attribute gekennzeichnet werden. 
	* Durch Überschreiben der Methode writeObject() und readObject() kann die Serialisierung gesteuert werden.
	* Unterklassen von serialisierbaren Klassen sind ebenfalls serialisierbar!
	
22) Java Versionsnummer von Klassen (beinhaltet class-hash): Wird die Klasse verändert, können keine alten Objekte mehr deserialisiert werden! Mittels einer eigen vergebenenSerialVerUid kann die Versionierung gesteuert werden.

23) Java Objekte als XML exportieren. Vorteil: Standartisiertes Datenformat, das auch andern Applikationen erlaubt, die Objekte weiterzuverwenden.

24) Die Referenzen können ungültig werden und die gleiche Referenz darf nicht wiederverwendet werden.

25) Die Java Non-blocking IO Channels dienen zum nicht blockierenden Datenaustausch anstelle der normalen (blocking) Channels.


Verteilte Objekte und entfernte Aufrufe
=======================================
26) Damit der Benutzer entsprechende Massnahmen treffen kann gegen blockieren (z.B. asynchronen Aufruf).

27)
	* Remote Garbage Collection
	* Remote Procedure Call
		
28) 
	RRA
		* Clients stellen Requests an einen Server (R). 
		* Der Server antwortet falls notwendig (R).
		* Der Client sendet je nach dem ein Acklowledge (A).
	Probleme
		* Nachricht/Reply/Ack kann verloren gehen
		* Message/Request kann mehrfach ankommen
		* Message / Reply kann mehrfach interpretiert/ausgeführt werden mit unterschiedlichen Ergebnissen, falls Message/Reply den Status von Server oder Client verändern.
		* Server/Client kann ausfallen, überlastet sein
		* Kommandreihenfolge kann durcheinander geraten
		
29) siehe 28.Probleme

30) 
	* Remote Call wird 1 mal durchgeführt (Maybe)
	* Remote Call mindestens einmal durchgeführt (at least once) -> mehrmals möglich mit untersch. Resultaten
	* Remote Call wird höchstens einmal durchgeführt (at most once) -> Reply wird gespeichert: immer gleiches Res.
	* Remote Call wird genau einmal durchgeführt
		
31) Es soll entfernt eine Methode aufgerufen werden, genau so wie es lokal geschieht.
		Aufruf
			Unbedingt Call by Value, da Referenzen auf Server und Client unterschiedlich sind
		lokale Memer
			Zugriff auf lokale Member remote ist zu unterlassen, die Referenz könnte sich geändert haben in der Zwischenzeit.
			
32) Interface definition Language. Definiert Signatur der Methoden sowie in und out parameter.
	.. code-block:: corba
	
		interface NodeList {
			void addNode(in Node node);
			void removeNode(in int position, out Node node);
		}
		
		
33) 
	* Remote Procedure Call: Data Structs + die Nummer der aufzurufenden Methode werden übermittelt. 
	* Retourniert werden Structs.
	* Communication Moduls auf beiden Seiten sind für die Übermittlung zuständig
	
34) Procedure ist grösser, beinhaltet u.U. mehrere Methoden, Methode call ist eher simpel	
	
35) 
	* Remote Method Invocation. 
	* Biete Remote method calls, Übertragung von serialisierten Java Objekten und remote Garbage Collection.
	* Der Client kommuniziert mit einem Proxy, der Server mit einem Skeleton. Proxy und Skeleton kommunizieren miteinandern und übernehmen Garbage Collecton, Remote Calls, ...
	
	
RMI
===
36) 
	Interface Server
		Interface zur Verfügung stellen
	Registry
		Registry initieren
	Server
		Server erzeugen
		Serverinterface implementieren
		Server anmelden bei Registry
	Client
		Client erzeugen
		RMI Server loockup in Registry machen
		Entfernte Methode auf lokalem Serverproxy aufrufen
		
37) 
	Client
		* besitzt lokale Objekte
		* besitzt einen Proxy für jedes Remote Objekt
		* besitzt ein Kommunikationsmodule, führt Request/Reply Protokoll aus
		* besitzt ein Remote Reference Module, das lokale und remote Referenzen zueinander übersetzt
		* Der Client spricht nur mit dem Proxy
	Server
		* besitzt lokale Objekte (Dienstobjekte, die der Client aufruft)
		* besitzt ein Kommunkationsmodule
		* besitzt Skeleton & Dispatcher für die lokalen Objekte
		* besitzt ein Remote Reference Module
		
38) 
	Proxy
		repräsentiert das entfernte Objekt, implementiert die Schnittstelle des entferten Objektes. Der Client spricht immer nur mit dem Proxy.
	Skeletton
		repräsentiert den Client, implementiert die Methoden der entfernten Schnittstelle. Der Server spricht immer nur mit dem Skeletton. Nur der Proxy des Clients und der Skelleton des Servers sprechend über das Netz direkt miteinander.
		
39) Der Dispatcher wählt die gewünschte Methode auf dem Skeleton aus.

40) 
	Call by Value
		Objekt Serialisieren, mitgeben
	Call by Reference (pseudo)
		Dem Server den Client als Parameter mitgeben, damit der Server auf dem Client entsprechende Methoden aufrufen kann
		
41) Remote Garbage Collection übernimmt das Entsorgen von nicht mehr benötigten Objekten in einem RMI System. Client und Server wissen nicht, wann auf der andern Seite ein Objekt stirbt und sie es abräumen können. Daher muss der GC regelmässig remote vorbeischauen und überprüfen, welche Objekte nicht mehr benötigt werden.

42) Das Interface wird auf einen Interface Server geladen. Client und Server laden sich das Interface herunter und implementieren es. Gegenüber dem normalen RMI ändert sich für den Client nichts, dem Server muss eine remote Codebase hinterlegt werden.

43) Dem RMI Server wird ein Daemon vorgeschaltet. Der Server geht schlafen sobald seine aktuellen Requests abgehandelt wurden. Der Client spricht den Server an, der Daemon weckt den Server und übergibt den Call.


Messaging
=========
44) 
	* Messaging ist Technologie unabhängig und es können beliebige Formate zum Einsatz kommen (JSON, XML, ...).
	* Sender und Empfänger müssen nicht zwingend zur gleichen Zeit aktiv sein.
	
45) Sender und Empfänger sind entkoppelt. Die Kommunikation läuft über eine Zwischenstelle/Vermittler.

46)	+--------------------+--------------------------------------------+--------------------------------------------+
	|                    | räuml. gekoppelt                           | räumlich nicht gekoppelt                   |
	+--------------------+--------------------------------------------+--------------------------------------------+
	| zeitlich gekoppelt | Sender und Empfänger müssen gleichzeitig   | Sender und Empfänger müssen gleichzeitig   | 
	|                    | aktiv sein, damit eine Nachricht über-     | aktiv sein zur Nachrichtenübertragung. Die |
	|                    | mittelt werden kann. Die Kommunikation ist | Kommunikation ist ungerichtet. z.B. IP     |
	|                    | gerichtet. z.B RMI                         | Multicast                                  |
	+--------------------+--------------------------------------------+--------------------------------------------+
	| zeitlich nicht gek.| Sender und Empfänger müssen nicht gleich-  | Sender und Empfänger brauchen sich nicht   |
	|                    | zeitig aktiv sein. Die Kommunikation ist   | zu kennen und haben eigene Zeitsysteme.    |
	|                    | gerichtet.                                 |                                            |
	+--------------------+--------------------------------------------+--------------------------------------------+

	
Gruppenkommunikation
--------------------
47) Multimediastreaming, Multimediachat mit mehreren Teilnehmern, Nachrichtensystem für mehrere Empfänger

48) 
	offene Gruppe
		Kommunikation von aussen in die Gruppe möglich
	geschlossene Gruppe
		Kommunikation nur innerhalb der Gruppe
	überlappende Gruppe
		Mitglieder können in mehreren Gruppen seine
		
49)
	Anforderungen an Zuverlässigkeit und Ordnung
		P2P
			* Integrität: versendete und empfangene Nachricht sind identisch
			* Validität: Nachricht wird eventuell abgeliefert
		Pub / Sub
			* Integrität
			* Validität
			* Agreement: Wird die Nachricht an jemanden ausgeliefert, wird sie an alle ausgeliefert
	Ordnung
		FIFO
			Nachrichten werden an alle in der gleichen Reihenfolge ausgeliefert
		Kausale Ordnung
			Die Ordnung der Ereignisse beeinflusst die Reihenfolge der Nachrichten
		Totale Ordnung
			Wenn eine Nachricht vor einer anderen einem Mitglied ausgeliefert wird, wird sie das an alle
			
50) JGroups erlaubt das erzeugen von Groups und das Publishen / Subscripen in eine Gruppe.
		* Channels: Gruppen, denen man beitreten kann
		* Building Blocks: Gruppieren Channels
		
		
Publisher / Subscriber
----------------------
51
.. 
* Publisher bringen Nachrichten in einen Pool eine
* Subscriber werden benachrichtigt, sobald neue Nachrichten da sind
* Rollen:
	* Ereignisdienst: generiert Ereignisse
	* relevantes Objekt
	* Beobachter: Vermittler
* Mit Filter können Subscriber die Nachrichten filtern

52
.. 
Topic
	Nachrichten können an ein Thema adressiert werden. Es werden nur die Abonnenten des Themas über die neue Nachricht benachrichtigt
Typen
	Subscribers werden anhand des Nachrichtentyps benachrichtigt.
		
53
..
Die Queue muss dies übernehmen. Kennt die Queue die Subscribers nicht, so ist das unmöglich.

54
..
Der Broker übersetzt die Nachricht vom Format eines Systems für das Format eines andern Systems
			
55
..
P/S System::

	                       Subscribers
	
	subscribe(t1)            subscribe(t2)          notify(e1)
         |                       |                      ^
	     v                       v                      |
	+----------------------------------------------------------+
	|              Public / Subscribe system                   |
	+----------------------------------------------------------+
	     ^                    ^                         ^
	     |                    |                         |
	publish(e1)          publish(e2)               publish(e3)

	                     Publishers


Message Queues
--------------

56
..
* MQ: Queue, in die Nachrichtein von Producern eingefügt und von Consumern abgeholt werden.
* Unterschied P/S: Keine Richtung definiert, jeder Kann Message in Queue einfügen oder abholen
* Arten
	* Lesen blockiert bis Message eingetroffen
	* Pollen, ob Nachricht da
	* Notify, wenn Nachricht da

57
..
Die Queue muss wissen, wer die Nachricht alles bekommen soll, damit sie den Zeitpunkt der Entsorgung weis.

58
..
Es wurde eine Session aufgebaut, weil TCP genutzt wurde. Dies ist jedoch komplett unsinnig, weil eine Message eine einmahlige, gerichtete Nachricht ist.

59
..
MQ::

	           Consumers
	
	   [A]       [B]         [C]
	receive()   poll()     notify()
		|         |           ^
	    v         v           |
	+------------------------------+
	|  |o|       |o|         |o|   | Message System
	|  |o|       | |         |o|   |
	|  |o|       | |         | |   |
	|  | |       | |         | |   |
	+------------------------------+
	    ^          ^         ^
	    |           \       /
	  send()      send() send()
	   [X]             [Y]

	          Producers


60
..
Java Messaging System. Plattformunabhängiges Message System.

61
..
Message Oriented Middleware

62
..
Ein Routerknoten verbindet sich mit jedem Teilnehmer, dadurch sind alle Teilnehmer über den Router miteinander verknüpft, ohne zu jedem andern Teilnehmer eine Verbindung aufbauen zu müssen.


Verteilte Dateisysteme
======================

63
--
* Datenkonsistenz
* Funktion trotz Komponentenausfall
* Fehlertolerant
* Skalierbarkeit

64
--
Replikation der Daten. Kein Datenverlust, wenn ein Knoten ausfällt.

65
--
* Concurrency
* Accesssecurity
* Wieder aktiven Knoten ohne Systemneustart integrieren
* Übertragen der Verantwortlichkeiten bei Komponentenausfall
* Caching der Daten

66
--
Ist eine Komponente nicht erreichbar, müssen die Operationen später ausgeführt werden können


NFS
---

67
..
Der Server liefert die Daten an die Clients aus. Dabei werden nur Teile der Daten übertragen.

68
..
Der Server leiht dem Client die Datei aus. Sobald ein anderer die Datei auch will, fordert er sie mit den gemachten Änderungen zurück.

69
..
Auf Grosse Dateien.


AFS
---

70
..
AFS liefert immer ganze Daten an den Client.

71
..
Auf viele aber kleine Daten.

72
..
AFS überträgt immer ganze Dateien, NFS nur Teile davon.


HDFS
----

73
..
* Effizientes Client Caching
* Geschwindigkeit vergleichbar mit lokalem Dateisysteme, Hoher Durchsatz
* Absturzresistenz
* Skalierbareit

74
..
* Setzt auf die jeweiligen Dateisysteme der Komponenten auf
* Für riesige Dateien ausgelegt
* Grosse Blöcke
* Für Write Once, Read Many ausgelegt (Streaming)
* Delegation bei Knotenausfall

75
..
Es setzt auf die jeweiligen bestehenden Dateisysteme auf und ist für riesige Dateien ausgelegt

76
..
Ausfallsicherheit
	Das System darf nicht ausfallen, auch wenn einzelne Knoten ausfallen.
Data Recovery
	Fällt ein Knoten aus, müssen die Daten wiederhergestellt werden können
Component Recovery
	Ausgefallene Komponenten müssen sich wieder ins System einfügen lassen, ohne das dies einen Einfluss auf das System hat
Konsistenz
	Die Daten müssen jederzeit Konsistenz sein
Skalierbarkeit
	Das System muss um weitere Knoten erweitert werden können

77
..
* Lokal anfallende Daten werden auch lokal behandelt
* Berechnungen finden da statt, wo die Daten liegen

78
..
Er wird wieder hochgefahren und ins System integriert. Anschliessend werden die Daten aktualisiert.

79
..
Es dient dazu, grosse und verteilte Datenmengen zusammenzuführen, zu verknüpfen und zu berechnen.

**WordCount**

Input
	Ein- oder mehrere Sätze
Map
	Es wird eine List aufgestellt mit den Wörtern (für jedes Wort gibt es so viele {Word, 1} - Einträge, wie Wörter )
Shuffle
	Die einzelnen Einträge werden zusammengesammelt {Wort, 1, 1, 1}
Reduce
	Die EInträge werden berechnet: {Wort, 3}
Output
	Eine Liste mit den Wörtern und deren Anzahl vorkommen wird zurückgegeben.

80
..
Systemarchitektur Hadoop::

	                                   [ Client ]
	                                       ^
	                                       |
	                                       v
	                     +-------------------------------------+
	                     | Naming Node     [ Job Tracker ]     |           Master Node
	                     +-------------------------------------+
	
	                   ^          ^                   ^         ^
	                  /           |                   |          \
	                 v            v                   v           v
	------------------- ------------------- ------------------- -------------------
	|  Node 1         | | Node 2          | | Node 3          | | Node 4          |     Slaves Nodes
	| [ Task Tracker] | | [ Task Tracker] | | [ Task Tracker] | | [ Task Tracker] |
	| - Map           | | - Map           | | - Map           | | - Map           |
	| - Reduce        | | - Reduce        | | - Reduce        | | - Reduce        |
	------------------- ------------------- ------------------- -------------------


81
..
Der Naming Node weis, wo die Daten liefern und Holt diese ab. Anschliessend werden sie dem Client ausgeliefert.

82
..
* Es gibt einen Job Tracker, mehrere Task Trackers, eine Naming Note. Die restlichen Nodes sind Data Nodes.
* Die Task Tracker fragen über den Naming Node den Node ab, auf dem die Daten liegen.
* Der Client kommuniziert immer mit dem Job Tracker.

::

	                           ,- Data Node
	              Task Tracker -- Data Node
	            /             \
	Job Tracker                Naming Node
	           \              /
	             Task Tracker -- Data Node
	                         `- Data Node


Namensdienste
=============

83
--
Ein Namesdienst ist ein Ausunftsdienst, wo sich Objekte, denen ein bestimmter Namen zugeordnet wurde, befinden. Die Aufgabe des Namensdienstes ist es, Kontexte und Subkontexte aufzulösen und die Adresse des Objektes zu liefern.

84
--
Ein Kontext ist die Umgebung eines Objektes. Beispiel www.hsr.ch. "ch" ist der Kontext, "hsr" der Subkontext.

85
--
Einen Kontext einem Objekt zuzuordnen.

86
--
Eine Gruppierung von Kontexten. Z.B. ftp://upload.hsr.ch. Ftp ordnet diese Adresse dem Namensraum ftp zu.

87
--
Ein System zur Auflösung von Namen und zum Auffinden deren gebundenen Objekte.

88
--
Im Unterschied zum reinen Namensdienst gibt es noch Attribute, nach deren die Objekte gefunden werden können.

89
--
* Fehlertolerant
* Ein Fehler darf sich nicht global auswirken
* Hohe und langzeitliche Verfügbarkeit
* Verwalten einer grossen Anzahl von Namen
* Tolerierung von Misstrauen (Nicht jeder kennt jeden und nicht jeder hat auf alles Zugriff)

90
--
Sind Namen, die auf andere Namen zeigen. z.B. www.goo.gl/erdif zeigt auf search.google.com/query=erdif

91
--
Iterativ
	* Client fragt Nameserver nach Adresse zu Namen
	* Nameserver liefert Antwort oder Server für Kontext.
	* Client fragt diesen Server
	* Dieser liefet wiederum die Antwort oder einen Server für den Subkontext.
	* ...
Rekursiv
	* Client fragt Nameserver nach Adresse zu Namen
	* Nameserver liefert Antwort oder fragt andern Nameserver
	* Dieser liefert Antwort oder fragt wieder andern Nameserver
	* ...

92
--
Erlaubt nicht nur das Finden von Namen, sondern von ganzen Services. Services melden sch selbstständig an und Clients verbinden sich automatisch mit den Services.

93
--
Global Naming System: Verteiltes, langlebiges, robustes und anpassbares Namenssystem, das mit Baumstrukturen arbeitet. Bsp: <EU/DE/HB/IBM, Hans.Mustermann/phone>

94
--
Verzeichnisdienst, die Grundlage von LDAP. Zu jede, Eintrag ist die Klasse spezifiziert (z.B. organozation). Es können eignen Klassen definiert werden. Einträge können nocht Attribute enthalten.


LDAP
----

95
..
LDAP ist ein auf X.500 basierender Verzeichnisdienst, dessen Einträge Klassen zugeordnet sind. Mit Schematas werden die Beziehungen zwischen Unter- und Oberklassen definiert.

96
..
OSI DAP setzt auf dem Presentation Layer auf, LDAP auf dem Transport Layer. LDAP ist wesentlich schlanker.

97
..
Ein Directory ist ein Subtree im Verzeichnisbaum. z.B::

	o=Swisscom


98
..
Ein Eintragsteil ist Typ einer Objektklasse.

Beispiel cn=Müller, sn=Hans, ou=People, o=SBB, c=ch::

	c=ch
		o=SBB
			ou=People
				uid=123
					sn=Hans
					cn=Müller


99
..
Objektklassen sind allgemeine Beschreibungen für gleichartige Objekte. z.B:
* ou Organization Unit
* o Organisation
* c Country

100
...
* add Entry
* remove Entry
* modify Entry Value
* search Entry
* compare Entry

101
...
* Jeder Name setzt sich aus einer Sequenz von Teilen zusammen.
* Die Einträge in einem Verzeichnis können hirarchisch in einer Baumstruktur geordnet werden.
* Aliase ermöglichen Verknüpfungen
* Ein Verzeichnisbaum kann sich über mehrere Server erstrecken,

102
...
Zwischen Client- und Server übermittelte TCP/IP Nachrichten spezifizieren die Operationen, die der Client ausführen möchte.

103
...
Zuerst wird der lokale LDAP verwendet, dann externe.

104
...
LDAP Objekte sind über Schemen definiert und deklarieren, welche Pflichtattribute und optionale Attribute definieren.


JNDI
----

105
...
Java Naming and Directory Interface API: Einheitliche Schnittstelle zum Zugriff auf verschiedenste Verzeichnisdienste.

106
...
JNDI ist ein Kontextgraph und besitzt Schnittstellen zu verschiedensten Diensten, wie z.B. LDAP.


P2P
===

107
---
Vorteile
	* Daten verteilt, redundant
	* Teilnehmer gleichberechtigt
	* Keine Zentrale kontrolle
	* kein zentrales Backup
	* Weniger gut angreifbar
Nachteile
	* Schwieriger, Informationen aufzufinden
	* Netz verändert sich dauernd
	* Suche nach Informationen kann lange dauern
	* Datenkonsistenz schwierig
	* Keine Zentrale zugriffskontrolle

108
---
Das Netz organisiert sich neu, andere Knoten übernehmen dessen aufgabe.

109
---
* Routen von Anfragen
* Verteilte Datenspeicherung

110
---
* Verteilter Algorithmus für das Routing in einem P2P Netz

111
---


Routing
-------

112
...
Jeder Knoten weiss nur über einen Teil der Objekte, wo sie sind. Trotzdem muss es möglich sein, über Routing das gesuchte Objekt zu identifizieren.

113
...
* Alle Knoten sind in einem Kreis angeordnet.
* Es wird an einem Punkt eingestiegen und zirkulär von einem Knoten zum nächsten gehüpft, der dem gesuchten Ziel am nächsten kommt.
* Nicht sehr effizient

114
...
* Jeder Knoten weis, wo die nächsten Knoten zu finden sind (Knoten der nächsten Stelle in der ID)
* Es wird von einem Knoten aus ein Knoten angesprungen, dessen erste Stele mit dem Ziel übereinstimt
* Es wird der nächste Knoten angesprungen, dessen nächste Stelle mit dem Ziel übereinstimmt
* Stelle für Stelle nähert man sich so dem Ziel

115
...
* Jeder Node beinhaltet ein Set mit numerisch und eines mit physisch nahen Knoten.
* die Routing Tabelle basiert auf Präfix basierte Routing

116
...
* Berücksichtigung der Entfernung (Latenz) von Knoten
* Information auf nahen Knoten wird bevorzugt
* weniger Trafic, schneller am Ziel

117
...
einfügen
	* Beim veröffentlichen eines neuen Objektes benachrichtigt der Knoten seinen Root Knoten
	* Alle Knoten auf dem Weg speichern einen Verweis auf das Objekt
lokalisieren
	* Anfrage geht an Root Node des Objektes
	* Passiert die Anfrage auf dem Weg einen Verweis zu dem Objekt, wird sie direkt dorthin umgeleitet
	* Desto naher der Knoten dem gesuchten Objekt ist, desto früher kreuzt sich sein Suchpfad mit dem Verweisfad.
	* Anfragen laden nur im schlechtesten Fall beim Root Node
löschen
	?


DHT
---

118
...
* DHT's sind über viele Knoten verteilt und beinhalten Referenzen zu Objekten
* Eine verteilte Hash Tabelle kann genau gleich viel wie eine lokale.

119
...
* Die Information ist beim Node gespeichert, dessen ID die nächst grössere Zahl als der Value Hash darstellt.
* Von einem Startpunkt aus wird im Kreis herum gegangen, bis der Knoten mit der Information gefunden wurde.
* Nachteil: Ineffizient.

120
...
* Jeder Knoten besitzt eine Successortabelle mit so vielen Einträgen wie Knoten: (index, z=k+2^(i-1), successor).
* Die Information wird bei der nächst höheren Knoten id (successor) als z gespeichert.
* Der Successor ist der nächstgrossere Knoten von z = {actual Node}+2^({Index}-1)
* Abfrage: Jeder Knoten sendet eine Anfrage an den Successor weiter, dessen Id kleiner als die gesuchte ist, wenn er selbst das Objekt nicht besitzt.
* Die Hops einer Query ähneln der Abfrage in einer Baumstruktur: O(log(N))

121
...
* Der nächst grössere Knoten wird gesucht
* Der Successor Pointer des neuen Knoten zeigt auf den gefundenen Knoten
* Elemente werden reorganisiert
* Der nächstkleinere Knoten wird aktualisert (Sucessor Pointer)
* Alle Finger Pointer werden angepasst


Zeit und globale Zustände
=========================

122
---
Zeit ist die definierung einer Reihenfolge. Die Zeit als physikalische Grösse ist gerichtet und unumkehrbar.

In Informatiksystemen sorgt Zeit dafür, dass Ereignisse in eine Reihenfolge gebracht werden können. Stimmen die Zeiten zweier Systeme nicht überein (was immer der Fall ist, die Frage ist wie viel), so gibt es ein Chaos, weil Ereignisse vor andern erfasst werden, obwohl sie in umgekehrter Reihenfolge passiert sind.

123
---
Transaktionen überwachen mittels der Zeit das sich keine Operationen überlagern. Stimmen diese Zeiten nicht, so sind Transaktionen vermischt mit andern, obwohl dies nicht sein darf und nicht protokolliert wird.

124
---
Sein eigener Zustand + der Inhalt der Kommunikationskanäle

125
---
Ereignisse & Messages
	Ein Prozess schickt eine Zustandsänderung über einen Kanal an einen andern. Das Absenden und Empfangen sind Ereignisse, die Transaktionsnachricht die Message Z.B. Konto A sendet Transaktion von 200.- an Konto B.
Marker
	Teilen allen Prozessen im System mit, das der Zustand aufgezeichnet wird.
Unterschiedliche Laufzeit der Messages
	Führt zur Vermischung der Reihenfolge (Nachrichten können andere überholen) beim Empfänger. Darum kann nicht einfach mit einem einfachen Cut der globale Zustand erfasst werden.

126
---
physische Uhr
	Feste Reihenfolge von gleichen Zeitabständen.
logische Uhr
	Reihenfolge wird definiert über Reihenfolge der Ereignisse. Keine festen Zeitabstände.

127
---
Weil dies zum kompletten Chaos führen würde. Nachrichten würden plötzlich ankommen, bevor se abgeschickt wurden.

128
---
Jeder Prozess zählt seine Ereignisse und teilt diese Anzahl jeweils in jeder Nachricht mit. Beim Empfangen aktualisiert jeder Prozess seine logische Uhr, indem er das max(receiveTime, localTime)+1 als neue lokale Zeit übernimmt.

Die logische Uhr kann aus Sicht des einzelnen Prozesses Sprünge aufweisen, weil ein anderer Prozess viel mehr Ereignisse erhält.

Die Prozesse besitzen zwar immer unterschiedliche Zeiten, doch sie befinden sich in einer fest definierten Reihenfolge.

129
---
Durch die schwankende Übertragungszeit beim senden des Zeitstempels von einem Zeitserver zu einem Client lässt sich die Zeit nicht genau bestimmen. Die Übertragungszeit ist unbekannt. Sie kann angenähert werden, indem die Nachricht jeweils zurückgesendet wird und der Server die RountTripTime dem Client bekannt gibt. Wird das ganze mehrmals wiederholt, so kann die Zeit angenähert werden aber eben nicht genau bestimmt werden. Alle verteilten Komponenten haben somit eine leicht andere Zeit.

130
---
* Gestartet wird bei 1
* Jeder Prozess inkrementiert bei einem lokalen Ereignis seinen Zähler.
* Bei einem Empfangsereignis wählt der Prozess max(receiveTime, localTime)+1 als neue lokalZeit.
* Damit ist die Reihenfolge der Ereignisse festgelegt.

131
---
::

	Z1 1 -----*2--------^4------------*5-----------^6----------
	            `      ´                `        ´
	Z2 1 ---------v3----------*4---------------*5-^7-----------
	                 ´          `           `    ´
	Z3 1 -- ^3--------*4-----------v5--------------------------
	       ´       ´           `             ´  `
	Z4 1 *2------*3------------------v5--*6--------v7----------

132
---
Aus der logischen Zeit ( C(a)->C(b) ) kann nicht auf die physische Zeit ( a->b ) geschlossen werden.

* "Kausalität impliziert Uhrzeitordnung"
* "Uhrzeitordnung impliziert NICHT Kausalität"


133
---
* Jeder Prozess führt ein Array mit den Zeiten aller Prozesse.
* Nur eigene Ereignisse inkrementieren die lokale Zeit
* Von andern Prozessen wird damit jeweils nur der Status übernommen
* von C(a)->C(b) kann auf a->b geschlossen werden

134
---
Die Physische Zeit ist die Menge aller Zuständer aller Prozesse.

**Vergleich Lamport CLock LC und Vector CLock LV**
Initialisierung
	* LC: 1
	* VC: 0

lokales Ereignis
	* LC: time+1
	* VC: time[actualProcess] +1

Sendeereignis
	* LC: send(time)
	* VC: send(time[])

Empfangsereignis
	* LC: max(localTime, receiveTime)
	* VC: Zeiten anderer Prozesse aktualisieren auf max(localTime[pi], receiveTime[pi])

135
---
Weil sie eine strenge Ordnungsrelation beschreibt

136
---
Nein. Sie ist dazu noch unflexibler und funktioniert nur, wenn die Anzahl Prozesse im System festliegt.


Globale Zustände
----------------

137
...
* Bank: Tagesauszug / Abrechnung
* Datenbank Backup
* Multiplayergame: Spielzustand

138
...
Die Historie eines Prozesses definiert sich aus der Menge seiner Ereignisse.

139
...
Die Zustände werden so erfasst, das das System als totales stimmt. z.B. Bank: die Summe aller Konti sollte unabhängig der Transaktionen immer gleich sein (ohne Transaktionen zu andern Banken).

Mit einem Ereignis enthält der Schnitt auch alle Ereignisse, die vorher statgefunden haben (mit dem Ereignis in einer happend-before Beziehung stehen).

140
...
* Um einen Snapshot zu machen, wird von einem Prozess eine Markernachricht versandt.
* Die andern Prozesse starten die Kanalaufzeichnung.
* Sobald die Prozesse erneut einen Marker erhalten, beenden sie die Aufzeichnung
* Der Globale Zustand ermittelt sich aus den Prozesszuständen+Kanalzustände (Aufgezeichnete, empfangene Nachrichten, die vor der Aufzeichnung abgesandt wurden)

::

	Z1 [1000] ----o--------------*----------^---^----o-^--*---------------
	                 ` 200        ° m1     °   ´      °`    ° 
	                     `         °     ° m2 ´     °  400`   °
	                         `      °  °     ´    °          `  °
	Z2 [500]  -----------^------v----v*--o----------------------v-v-------
	                    ´          300    `´   ° m3                °
	                 ´                 ° ´ ` °                      ´°
	              ´ 150            200 ,°  °`                  ´       °
	Z3 [750]  -o----------------------*--v*---v-----------o--------------v-

	Z1  o - - >  Z2: von Z1 nach Z2
	Z1  * ° ° >  Z2: Marker
	e12: Kanal1, 2. Nachricht im Beispiel, die abgeschickt wird
	
	Konsistenter Zustand (2250 Total ursprünglich):
	Z1: z1 + e32 = (1000-200) + 200 = 1000
	Z2: z2 + {}  = (500+150+200) = 650
	Z3: z3 + {}  = (750-150-200) = 400
	Total konsistenter Zustand: 2250
	
	Gehört e21 auch noch dazu, weil sie während der Messung von Z3 ankommt, oder nur vor der Messung abgesandte ?


141
...
* Zuverlässigkeit: Jede Nachricht wird genau einmal übermittelt
* Die Kanäle sind unidirektional und implementieren FIFO

142
...
Weil sich die lokalen Zuständer bereits während dem Ermitteln des globalen schon wieder verändert haben.

Der globale Zustand ist deshalb konsistent, weil nur die Nachrichten bis zu einem bestimmten Zeitpunkt und die Nachrichten, die noch unterwegs sind, berücksichtigt werden.


Verteilte Synchronisation
=========================

Verteilten Ausschluss
---------------------

143
...
Die Prozesse müssen einen Weg finden, auszuhandeln, wer in die Critical Session CS eintreten darf, und wer schon drin ist.

Zentraler Koordinator
	* Ein zentraler Koordinator gibt den Zugang in die CS frei
	* In einem verteilten System wird der Koordinator gewählt, ansonsten Client-/Server Prinzip

Multicast
	* Über Multicast wird herausgefunden, ob gerade jemand in der CS ist
	* Sobald derjenige in der CS wieder draussen ist, benachrichtigt er den wartenden


144
...
* Es tritt immer nur ein Prozess in die CS ein
* Kein Prozess verhungert in der Warteliste
* Prozesse überholen einandern nicht in der Warteschlange

145
...
Alle Prozesse stellen eine Anfrage an einen zentralen Server. Sobald die CS frei ist, erhält er vom Server ein Token, um in die CS einzutreten.

a)
~~
Dann steht das ganze System still (Single Point of Failure).

b)
~~
* Token kann verloren gehen (Stillstand)
* Token kann doppelt vorkommen (Konflikt)
* Prozess kann in CS abstürzen und verlässt diese nicht mehr (blockade)
* Server kann ausfallen (Stillstand)

c)
~~
Der Server muss regelmässig überprüfen, ob das Token noch da ist und ansonsten ein neues generieren

146
...
Alle Teilnehmer sind in einem Ring angeordnet. Der Token wird herumgegeben. Wer keinen Eintritt in CS will, gibt ihn weiter. Wer Zutritt will gibt ihn nach dem Verlassen weiter.

a)
~~
Das Token stirbt mit ihm und er verlässt die CS nie. (Stillstand)

b)
~~
Jemand müsste ein neues generieren. (Stillstand)

c)
~~
* Mehrere Tokens im Umlauf
* Token verloren
* Jemand gibt Token nicht weiter
* Prozess 2 stibt, 1 kennt 3 nicht um Token weiterzugeben

d)
~~
Extreme hoher Bandbreitenbedarf. Das Token wird auch im Kreis herumgegeben, wenn niemand in die CS will. Desto weniger in die CS wollen, desto mehr Nachrichten werden verdandt.

147
...
Wer in die CS will, sendet einen Multicast und tritt ein, wenn er von allen Antwort erhalten hat. Wer in der CS ist, gibt erst Antwort, wenn er wieder draussen ist.

* Es muss bekannt sein, wie viele Teilnehmer es gibt -> Bei Multicast nicht der Fall

148
...
Gleiches Verfahren wie bei Ricart, nur dass die Anfrage jeweils nur an die Prozesse geht, die in die CS eintreten wollen. Wer gar nicht in die CS will, wird gar nicht gefragt.


Wahlen
------

149
...
* Der Wahlauslöser sendet ein Election-Nachricht an den Nachbarn mit einer List, die ihn enthält.
* Jeder trägt sich in die Liste ein und sendet die Nachricht weiter
* Erhält der Wahlauslöser die Nachricht und ist schon eingetragen, so bestimmt er die höchste ID, merkt sich dies als Communicator und sendet die Nachricht als Elected-Message weiter.
* Jeder berechnet den Communicator, wenn er die Message erhält

150
...
2(n-1)

151
...
* Jeder kennt seine grösseren Nachbarn
* bei der Wahl sender der Wahlauslöser eine Anfrage an die grösseren (grösste ID's') Nachbarn
* Jeder sendet wieder eine Anfrage an die grösseren Nachbarn
* Der lebende Prozess mit der grössten ID, der keine grösseren Nachbarn kennt, ruft sich als Communicator aus indem er dies an alle sendet.


Multicast Kommunikation
-----------------------

152
...
Bei Multicast ist es nicht vorgesehen, das die Anzahl Teilnehmer ermittelt werden kann.

153
...
* Ein Knoten wurde sendet einen Nachricht mit seinem Zeitstempel und seiner ID als Identifier an alle
* Jeder Teilnehmer sendet einen Sequenznummervorschlag zurück
* Der Initialknoten wählt aus den Vorschlägen die höchste aus und sendet diese an alle


Konsens
-------

154
...
Weil der Prozess, der die Schlussentscheidung treffen muss, zwei Resultate hat von den beiden andern Prozessen. Sind diese unterschiedlich, so lässt sich keine Entscheidung treffen (50/50).

155
...
* Der Kommandant teil den andern seine Entscheidung mit
* Jeder teilt den andern seine und die erhaltene Entscheidung mit
* Der Letzte trifft eine mehrheitsentscheidung

**Beispiel mit 4 Knoten**
* 1->2,3,4: Angriff
* 2->3,4: Angriff
* 3->2,4: Unbekannt (möchte Rückzug, da a+r= unbekannt)
* 4->2,3: Angriff
* 2 entscheidet sich 2:1 für Angriff


Modellierung verteilter Systeme
===============================

156
---
Nebenläufig
	Zwei unabhängige Prozessen können (müssen aber nicht) parallel ausgeführt werden
Parallel
	Zwei prozesse laufen parallel ab (sind nicht mehr unabhängig)

157
---
Producer
	Produziert Tokens und reiht diese in eine Queue ein
Consumer
	Nimmt die Tokens aus der Queue und verbraucht diese

::

	 Producer           Queue            Consumer
	   P1       T1       P2       T2       P3
	   (*) ---> [ ] ---> ( ) ---> [ ] ---> (*)
	    ^--------'                 ^--------'
	N = [P,T,F,m0]
	P = {P1,P2,P3}
	T = {T1, T2}
	F = {(P1,T1), (T1,P1), (T1,P2), (P2,T2), (T2,P3), (P3,T2)}
	m0 = {1,0,1}


158
---
Vorbereich
	Positionen, die im Falle einer Transition Tokens liefern, bzw. von denen die Transaktion entspringt. Im Beispiel 157 sind P3 und P2 im Vorbereich von T2
Nachbereich
	Positionen, die im Falle einer Transition Tokens erhalten. Im Beispiel 157 sind P1 und P2 im Nachbereich von T1
Konzession
	Möglichkeit einer Transition zu schalten (Vorbereich ist so belegt, das ein Schalten möglich ist)

159
---
Die Verteilung der Tokens auf die Stellen im Netz. Die Markierung m0 beispielsweise stellt den Ausgangszustand dar. m1 im System bei 157: m1 = {1,1,1}

160
---
::

	   {1,0,1}
	 t1 |   ^
	    v   | t2
	   {1,1,1}
	 t1 |   ^
	    v   | t2
	   {1,2,1}
	 t1 |   ^
	    v   | t2
	   {1,3,1}
		...


161
---
::

	P1                                   P6
	(*)-->[ ]<----.         .---->[ ]<--(*)
	 ^     |       `.     .´       |     ^
	 |     v         `. .´         v     |
	[ ]   ( ) P2      (*) P4   P5 ( )   [ ]
	 ^     |          ^ ^          |     ^
	 |     v        .´   `.        v     |
	( )<--[ ]------´       `------[ ]-->( )
	P3                                   P7
	
	Kritical Section: P2, P5
	Local Work: P3, P7
	Waiting: P1, P6


Möchte der linke Prozess in die kritische Section P2 eintreten, so holt sie bei P4 das Token und gibt es anschliessend wieder frei.

162
---
Der Crosstalk Algorithmus bildet das Senden und Empfangen von Nachrichten zwischen zwei Teilnehmern ab, wobei immer Bestätigungen versandt werden, wenn die Nachricht erfolgreich angekommen ist. Bedingung: Es darf jeweils nur eine Nachricht im Kanal sein.

Beim Sendern einer Nachricht kommt diese entweder an und der Empfänger sendet eine Bestätigung oder sie überschneidet sich mit einer andern Nachticht (Crosstalk) und der Empfänger sendet keine Bestätigung.

163
---
Eingabesignale an das System werden sofot wieder nach aussen weitergegeben. z.B. Kunde drückt auf die Klingel -> System lässt Klingel läuten.

164
---
Petri Netz, mit Aktionen bezeichnet. z.B. on, Betrag erhöhen, ...


CSP
---

165
...
Prozessalgebra zur Beschreibung von Prozessen.

166
...
Stop
	Hält das System an.
Skip
	Stoppt das System. Anschliessend kann gar nichts mehr gemacht werden.

167
...
::

	x: Event
	P: Prozess
	
	( )--x-->(P)   entspricht   x -> P
	
	Das Ereignis x wartet darauf, von P akzeptiert zu werden.


168
...
Die Reihenfolge von Operationen des CSP.

169
...
Der Automat wird selbst in die Ereigniskette eingebunden:

::

	CHF : (in5 → out2 → out1 → out2 → CHF )


170
...
::

	(a -> P|b -> Q)


171
...
Traces sind eine Art Historie (Prozessspuren).

::

	a -> b -> STOPP
	
	Traces: <a>,<a,b>,<>
	Tracelänge: #<a,b> = 2


172
...
::

	Channel: c
	Ausgabe: c!e
	Eingabe: c?e
	
	Beispiel Wert der Eingabe nach Ausgabe kopieren:
	Copy = in?e -> out!e -> Copy


173
...





