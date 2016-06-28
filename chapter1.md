# Die Konfigurationsdatei

Die Konfigurationsdatei von Squid ist mit rund 125 Einträgen wohl neben der des Apache eine der längsten Konfigurationsdateien, die es bei einem Server gibt. Glücklicherweise ist ein Großteil
des Inhalts eine (englische) Hilfe für die entsprechenden Konfigurationsmöglichkeiten (erkennbar am vorangestellten 'Gartenzaun').

Eine weitaus komplettere (englische) Dokumentation findet sich übrigens auf den [Webseiten](http://www.squid-cache.org/) des Squid-Projektes. 


## Network
In diesem Abschnitt finden Sie alle Parameter, die für den Betrieb von Squid in einem Netzwerk erforderlich sind. So muss Squid u.a. wissen:
* Ob er Daten von lokalen Webservern zwischenspeichern soll oder nicht - bzw. welche Webserver überhaupt im lokalen Netzwerk laufen.
* Ob es andere Proxies im Netzwerk gibt und auf welche Art und Weise Squid mit ihnen Kommunizieren kann.
* Mit welcher Art von Clients es Squid zu tun hat: "normalen" Desktop Browsern oder gar Gateways oder anderen Proxies.

Squid benutzt für die Kommunikation mit anderen Rechnern die Protokolle TCP, UDP  und ICP auf speziellen Netzwerk-Ports.

TCP und UDP werden zur Kommunikation mit Webservern und Clients verwendet, ICP zur Kommunikation mit anderen Proxies. Für jede Verbindung benötigt Squid Angaben, über welche Ports und IP-Adressen diese Verbindungen aufgebaut werden sollen.


### http_port
Mit diesem Parameter wird der Port definiert, auf welchem der Proxyserver auf Anfragen von normalen Clients lauscht.

Wenn zusätzlich eine IP-Adresse oder ein Rechnername angegeben wird, wird Squid nur auf der angegebenen Adresse lauschen. Anderenfalls nimmt Squid Anfragen auf allen angeschlossenen Schnittstellen entgegen, was bei einem Proxyserver durchaus Sicherheitslücken verursachen kann, da dann meist auch andere Nutzer aus dem Internet den Proxy benutzen - ohne das der Administrator davon erfährt.

> Lassen Sie nach Möglichkeit Squid immer nur auf Anfragen des internen Netzwerkes hören, indem Sie ihm hier eine entsprechende IP-Adresse zuweisen.

```
   Standardwert:  http_port 3128
   Mögliche Werte: http_port <Port>
                   <Rechnername> : <Port>
                     <IP-Nummer> : <Port>
```

Beispiele:

Oftmals wird der Port 8080 oder 8000 verwendet. Dies können Sie mit diesem Eintrag erreichen:

 ```
  http_port 172.16.0.1:8080
 ```

Die Einstellung in der Datei squid.conf kann beim Start des Servers überschrieben werden. Dazu dient der Schalter -a beim Programmstart.
Sie können Squid auch auf mehreren Ports und (sollte Ihr Rechner über mehrere Netzwerkschnittstellen verfügen) Netzwerken lauschen lassen. Tragen Sie hierzu einfach die Angaben hintereinander in die Konfigurationsdatei ein.


## OPTIONS WHICH AFFECT THE NEIGHBOR SELECTION ALGORITHM

Hier wird es schon etwas interessanter.  Wenn Sie z.B. einen Provider haben, der eigene Proxy-Server für Sie bereitstellt, dann können Sie durch die zusätzliche Nutzung dieser externen Server einen enormen Geschwindigkeitsgewinn erreichen:

### cache_peer
Mit
```
 cache_peer www-proxy.t-online.de parent 80 0 no-query default no-netdb-exchange
```
verwenden wir z.B. den Proxy-Server von T-Online (Achtung: dieser Proxy ist nur ein Beispiel und nur nach Einwahl mit einem T-Online Account erreichbar).

* Dieser Server soll als "Eltern"-Server (parent) immer befragt werden. Da T-Online keine "ICP-Queries" gestattet, muß als zweite Zahl eine 0 stehen. (Eine 7 wäre auch möglich - dann wird an den Proxy vor der Benutzung ein "Ping" gesendet. Dies entspricht aber nicht mehr dem aktuellen Standard.) Zusätzlich wird dies auch noch mit der Option "no-query" explizit angegeben.
* "default" steht für einen Cache, der als "letzte Reserve" benutzt werden kann. Wenn also kein anderer (besserer) Proxy Server für die Anfrage benutzt werden kann, dann wird dieser "default" Proxy genutzt. Da wir bislang keinen anderen Proxy definiert haben, könnte man sich diese Option natürlich auch schenken.
* "no-netdb-exchange" schließlich bedeutet, dass die Datenbanken dieser beiden Server nicht untereinander ausgetauscht werden sollen. (Dies lässt T-Online auch nicht zu.)

Eine weitere Möglichkeit stellt die zusätzliche Verwendung externer Filter-Programme dar, welche komplett eigenständig laufen (und auch konfiguriert werden müssen). Hier bietet die Wirtschaft eigentlich genügend Möglichkeiten, sein Geld auszugeben. Von Empfehlungen sehe ich hier deshalb einmal ab. Aber bitte nicht diese zusätzlichen Proxies mit den Möglichkeiten von Squid verwechseln, andere "Hilfsprogramme" einzubinden (wie etwa SquidGuard). 

Oft müssen hier allerdings einige zusätzliche Regeln definiert werden, welche Squid anweisen, nur für Webseiten und nicht für FTP-Dateien bei den zusätzlichen Proxy Servern anzufragen. Die zugehörende Konfiguration für ein Filterprogramm auf dem eigenen Rechner (Port 9090) lautet insgesamt (kann so in die squid.conf eingetragen werden):

```
cache_peer localhost parent 9090 7 proxy-only no-query default no-digest
no-delay
acl ftp url_regex -i ftp://*
always_direct allow ftp
```

So lassen sich bei einigen dieser Zusatzfilter wirksam Werbebanner und Pop-Up Fenster verhindern. Wie das mit anderen Open Source Tools ebenfalls geht, erzähle ich weiter hinten.


### cache_host_domain

Mit der folgenden Zeile:
```
cache_host_domain www-proxy.t-online.de .de
```
beschränken wir die Domains, nach denen der Proxy-Server von T-Online befragt wird, auf Domains aus Deutschland.

### cache_stoplist
Mit 
```
cache_stoplist cgi-bin ?
```
sagt man Squid, dass er URL's, die ein "cgi-bin" oder ein "?" enthalten, direkt holen und nicht zwischenspeichern soll. Da es sich hierbei meist um Suchanfragen (z.B. in google) und damit um dynamisch generierte Webseiten handelt ist das auch in Ordnung.

## OPTIONS WHICH AFFECT THE CACHE SIZE

Hier läßt sich wohl am ehesten am Proxy-Server tunen. Immerhin handelt es sich hierbei um Einstellungen, welche die Speichernutzung - sowohl im RAM als auch auf der Festplatte - betreffen.

### cache_mem

Gleich der erste Eintrag ist ein Volltreffer - normalerweise nutzt Squid nur 8 MB RAM (!). Man kann zwar von einer gewissen Kompensation reden, wenn man bedenkt, dass Linux von sich aus oft benötigte Dateien von der Festplatte im RAM zwischenspeichert, aber mit einer höheren Einstellung lassen sich hier mehr Objekte direkt von Squid im Speicher halten, ohne dass zusätzliche Mechanismen benötigt werden.

Hier eine kleine Tabelle mit Werten, die sich in der Praxis bewährt haben:

| RAM im System | Ram für Squid |
| -- | -- |
| bis 32 MB | 8 MB |
| 32 MB | 16 MB |
| 64 MB | 24 MB |
| 128 MB | 96 MB |
| 256 MB | 96 MB |

Darüber hinaus sollte man für Squid ungefähr ein Viertel des vorhandenen RAMs reservieren. 

Bitte beachten Sie, dass Sie laut der offiziellen Dokumentation immer ein Vielfaches von 4 MB verwenden sollten und der gesammte von Squid verwendete Speicherplatz durchaus höher sein kann! (Dies ist hier aber noch nie vorgekommen...)

Wenn Sie über ein System mit ausreichend RAM (>=128 MB) verfügen und auch die weiter unten eingestellten delay_pools vernünftig nutzen möchten, sollten Sie hier mindestens 32 MB vorsehen. Ich habe mich bei meinem Beispiel sogar für 128 MB entschieden:

```
cache_mem 128 MB
```

### maximum_object_size

Dateien über die hier angegebene Größe werden NICHT vom Proxy zwischengespeichert. Eine HTML-Datei erreicht selten die vorgegebene 4 MB Grenze - wenn Sie aber mit ihren Schülern einmal ein Video ansehen wollen oder als Administrator ein Updatepaket für mehrere Rechner aus dem Internet laden müssen, sollten Sie hier höhere Werte einstellen.  (Unter Webmin lautet der passende Eintrag hier übrigens "Maximale Größe zwischengespeicherter Objekte".)

```
maximum_object_size 65536 KB
```

### maximum_object_size_in_memory
Das, was die Übersetzung schon sagt: Dateien, welche größer sind als hier angegeben werden im RAM gehalten.

Standard sind 8 KB. 
```
maximum_object_size_in_memory 4 KB
```

### cache_replacement_policy
Wann sollen Dateien aus dem Cache entfernt werden? Seit der Version 2.5 von Squid stehen dafür mehrere Möglichkeiten zur Verfügung:

| Wert | Beschreibung |
| -- | -- |
| heap LFUDA | (Heap least frequently used with dynamic aging) Häufig angefragte Objekte werden im Cache gehalten, selten angefragte werden freigegeben, unabhängig von deren Grösse. Damit wird ein häufiger angefragtes grosses Objekt ggf. auf Kosten vieler kleiner Objekte im Cache gehalten. Damit steigt die Trefferrate für grosse Objekte auf Kosten kleinerer. Die "Byte"-Trefferrate liegt hier also höher, da nur ein Treffer für eine grosse Datei schon viele Treffer für kleinere Dateien ausgleichen kann. |
| lru | (Last Recently Used) Behält die zuletzt angefragten Objekte im Cache, unabhängig von Grösse und Alter der Objekte. Also werden grosse Objekte genauso behandelt, wie kleine. Wenn neue Objekte angefragt werden, werden ältere Dateien aus dem Cache gelöscht. Dies ist die Standard Einstellung, wenn nichts anderes explizit angegeben wird.|
| heap GDSF | (Greedy-Dual-Size Frequency) Kleine, häufig angefragte Objekte werden auf Kosten grosser weniger häufig angefragter Objekte im Cache gehalten. Damit wird die Warscheinlichkeit eines Treffers für kleinere Objekte gesteigert. |
| heap LRU | (Heap Last frequently used) Erweitertes lru Verfahren. Hier wird zusätzlich noch das Alter der zwischengespeicherten Seiten berücksichtigt.  |

Jetzt stehen wir natürlich vor einem Dilemma: Lieber kurze Antwortzeiten (damit GDSF) oder weniger Traffic-Volumen (damit LFUDA)?

Wenn eine Schule ihren Internetanschluss nach Volumen abrechnet empfielt sich in jedem Fall die Verwendung von "heap least frequently used with dynamic aging LFUDA".

Unter Webmin lautet die entsprechende Einstellung "Dynamic least frequently used". Damit einher geht allerdings auch ein grösserer Wert bei der maximum_objekt_size!

Damit lohnt sich die Umstellung also nur, wenn gleichzeitig auch der Wert für cache_mem erhöht wird. So ab 32 MB RAM für Squid sollte sich die Umstellung bemerkbar machen - darunter fahren Sie mit den alten Werten auf jeden Fall besser. 
```
cache_replacement_policy heap LFUDA
```
Eine andere Möglichkeit wäre noch ein goldener Mittelweg durch Kombination beider Verfahren: Squid kann den Cache-Bereich im Hauptspeicher und denjenigen auf der Festplatte unterschiedlich verwalten. Also kann man im RAM mit "GSDF" möglichst viele kleine Objekte zwischenspeichern und auf der Festplatte die "grossen Brocken" auslagern, indem man dort LFUDA nutzt. In diesem Fall sähe die Konfiguration so aus:
```
cache_replacement_policy heap LFUDA
memory_replacement_policy heap GDSF
```
Zur letzten Zeile erfahren Sie mehr im nächsten Abschnitt.

### memory_replacement_policy
Genauso, wie die Dateien auf der Festplatte verwaltet werden, so funktioniert das auch im RAM. 
```
memory_replacement_policy heap LFUDA
```

## LOGFILE PATHNAMES AND CACHE DIRECTORIES

### cache_dir
 
 Hier stellen Sie u.a. die Größe des verwendeten Festplatten-Caches ein. Bitte achten Sie darauf, dass Sie auch genügend Platz auf der entsprechenden Partition haben. Abhilfe könnte hier das Anlegen eines Unterverzeichnisses auf einer anderen Partition (z.B. im Homeverzeichnis - ungünstig, geht aber) bringen.

Hier haben Sie zusätzlich die Möglichkeit, neben dem normalen UFS-Cache-Dateisystem auch das neue DISKD- oder AUFS-Dateisystem zu verwenden. Dieses soll einen entscheidenden Geschwindigkeitsgewinn bringen, wie auf der Hauptseite von Squid zu lesen ist.

Die drei Zahlen hinter der Verzeichnisangabe betreffen immer die Gesamtgröße des Caches und die Anzahl an Verzeichnissen und Unterverzeichnissen, in welchen die gecachten Seiten gespeichert werden.

Die vielen Verzeichnisse und Unterverzeichnisse müssen sein, da ansonsten zu viele Dateien innerhalb eines einzelnen Verzeichnisses gespeichert werden. Sie sollten die Standardangabe von 16 Verzeichnissen mit 256 Unterverzeichnissen nur bei "ausuferndem" Platzangebot (mehrere GB) für Squids Cache verändern müssen.

Eine Kalkulationsmöglichkeit für optimale Performance gibt Gert Brits:
* x=Grösse des Cache-Verzeichnisses in kB (d.h. 6GB ~ 6,000,000KB);
* y=Durchschnittliche Objektgrösse (im Schnitt meist ~ 13KB)

$$(((x / y) / 256) / 256) * 2 = Anzahl der Verzeichnisse im ersten Verzeichnisbaum$$

Bei einer Cache-Dir Grösse von 6GB sähe das dann ungefähr so aus:
$$6,000,000 / 13 = 461538.5 / 256 = 1802.9 / 256 = 7 * 2 = 14$$

Damit sähe die cache_dir Zeile dann so aus:
```
cache_dir /var/squid/cache 6000 14 256
```

Auch bei der Zugriffsart auf das Cache-Verzeichnis haben Sie wieder die Qual der Wahl:

| Wert | Bemerkung |
| -- | -- |
| UFS (Standard) | Die Standard-Einstellung von Squid (wird auch genommen, wenn nichts weiter angegeben wird). Sollte bei der normalen Installation auch so gelassen werden. Erst ab 1-2 GB lohnt sich eine Umstellung wirklich. |
| Async UFS | Asyncrones UFS - um zu verhindern, dass Squid allzu lange Pausen beim Zugriff auf die Festplatte macht. Hier werden keine Rückmeldungen vom System verlangt, wenn Dateien gespeichert werden. Das ist schnell, führt aber zu Fehlverhalten von Squid, wenn einmal das System nicht nachkommt und Daten nicht schnell genug speichern kann. Sie sollten hier also durchaus mit Datenverlust oder Programmabstürzen (von Squid) rechnen. Wenn es allerdings funktioniert... ;-) |
| diskd | Die "neueste" Errungenschaft. DISKD ist kompatibel zu UFS und kann mit UFS eingerichtete Cache-Verzeichnisse direkt übernehmen. Hier soll - ähnlich dem Async. UFS - ein "Blockieren" von Squid bei Plattenzugriffen verhindert werden. Dafür wird ein separates Programm "diskd" gestartet, welches die Plattenzugriffe von Squid verwaltet. Besonders bei der Verwendung von mehreren Cache-Verzeichnissen auf verschiedenen Platten kann es zu nachweisbaren Geschwindigkeitsgewinnen kommen. Beachten Sie aber, dass "diskd" auch wieder Platz im RAM und einige Ressourcen des Prozessors benötigt. |

Ein Beispiel gefällig?
```
cache_dir aufs /var/squid/cache 6000 14 256
```
 Wenn Sie den Server neu installieren oder eine weitere, kleine Platte "übrig" haben, dann sollten Sie diese speziell für Squid einbinden. Um die Zugriffe auf die Cache-Dateien noch weiter zu optimieren, kann dann die Partition oder Platte mit der Option "noatime" in der Datei /etc/fstab eingebunden werden. Damit wird bei einem Zugriff auf die Cache-Dateien nicht gleichzeitig jedesmal die Zugriffszeit geändert.

...und wo wir gerade bei der Datei /etc/fstab und der separaten Partition von Squid's Cache sind: mit "noexec" und "nosuid" schliessen Sie vorsorglich Sicherheitslücken. Ein Beispieleintrag in der Datei /etc/fstab könnte also so aussehen:
```
# Device 	Mountpoint 	FStype 	Options 	Dump 	Pass#
... 	/... 	... 	defaults 	1 	2
/dev/hdd2 	/var/squid 	reiserfs 	rw,noatime,noexec,nosuid 	0 	3
# /dev/hdc2 	/var/squid/logs 	reiserfs 	rw,noatime,noexec,nosuid 	0 	3
```
 (Achtung: bei diesem Beispiel werden auch die Logbücher von Squid unter /var/squid/logs auf dem entsprechend eingebunden Filesystem abgelegt. Damit wird diese eine Festplatte natürlich wieder mehr belastet, weil Sie einerseits ja die heruntergeladenen Dateien wegspeichern muss, andererseits aber auch die entsprechenden Logbücher.

In der letzten Spalte ist deshalb noch eine weitere Platte im System (derzeit mit einem '#' auskommentiert), die nur für die Logbücher zuständig ist.

Wenn Sie die "Performance" Ihres Systems auf die Spitze treiben wollen, sollten Sie sowieso über ein RAID-System nachdenken. Es ist auch immer eine gute Idee, wenn alle Logbücher des Systems auf einem anderen System (der Syslog-Daemon lässt übrigens auch das Schreiben über ein Netzwerk zu) oder einer anderen Platte gespeichert werden. Jetzt noch alle ausführbaren Programme auf einer Platte und Partition, welche Read-Only gemountet wird, alle spool- und cache-Verzeichnisse auf einer weiteren Platte, ...

...und irgendwann sollte man über ein Leben ohne Computer und Backups nachdenken...

Übrigens: Wenn man einmal eine bestimmte Datei sucht, die mal im Web vorhanden aber nun gelöscht ist, kann man unter Umständen noch sein Glück im Cache von Squid versuchen. Dazu benötigt man die Datei store.log. Dort wird vom Squid festgehalten, welche Dateien sich gerade im Cache befinden und welchen Status diese haben. Schauen wir mal kurz auf eine Zeile in dieser Datei:
```
1109331307.457 Release -1 FFFFFFFF D6A9EAFAD972F9389675C5B8AB2AD5BF 302 1109331307 -1 -1 text/html -1/204 GET http://www.google.com/
```
Die Einträge sind wie folgt zu verstehen:
* Zeitformat (in UTC mit Millisekunden)
* Die Aktion (hier: RELEASE, d.h. die Datei wurde freigegeben oder nicht gespeichert)
* Das Verzeichnis (-1 steht hier für: nicht gespeichert)
* FFFFFFFF (die Dateinummer - wurde die Datei wie hier nicht gespeichert, wird FFFFFFFF gesetzt - ansonsten findet man hier den Dateinamen)
* Der Statuscode der HTTP Antwort - hier 302, da google auf die lokale Domain umleitet.
* Das Datum aus der HTTP Antwort
* "Last-Modified: " aus der HTTP Antwort
* "Expires: " aus der HTTP Antwort
* Dateityp - hier eine reine Textdatei
* Mit einem / getrennt: "Content-Length: " aus der HTTP Antwort und die tatsächlich übermittelte Grösse
* Die Methode, die diese Datei angefordert hat (meistens GET)
* Die URL zu dieser Datei

Anhand dieser Angaben sollten Sie die benötigte Datei nun schnell im Cache-Speicher finden können - sofern Sie dort gespeichert wurde...

### emulate_httpd_log

Squid speichert seine Logfiles normalerweise in einem eigenen Format, welches auch von entsprechenden Analyse-Tools gelesen werden kann. Wenn Sie - um etwa bei der Verwendung von "Logviewer" - Squid mit diesem Schalter anweisen ein "lesbareres" Logfile-Format zu verwenden, müssen Sie auch die entsprechenden Analyse-Tools (wie z.B. calamaris und webalizer) entsprechend umstellen.
```
emulate_httpd_log on
```
Damit speichert Squid besonders die Datei access.log, welche die Anforderungen der Clients enthält, in einem benutzerfreundlichen Format. So wird u.a. die Zeit in einem lesbaren Format gespeichert und auch der Rest der Einträge lassen sich mit nur wenig Fantasie richtig deuten.

Mit der neuesten 3.x Version ist diese Option übrigens obsolet: Squid speichert jetzt immer im "http_log" Format.

### log_fqdn

Hier können Sie Squid anweisen, die Hostnamen der anfragenden Clients aufzulösen und diese Namen ins Logbuch zu schreiben. Aber Achtung! Zur Auflösung dieser Namen führt Squid einen DNS-Lookup durch, er fragt also zunächst einmal den eigenen Rechner (der diese Daten hoffentlich in der Datei /etc/hosts hat oder über einen eigenen Nameserver verfügt) und wendet sich bei einem Misserfolg an den nächsten DNS-Server. Wenn es sich dabei um einen öffentlichen DNS-Server (etwa den der Telekom) handelt, dann wird Squid die anfragenden IP-Nummern kaum zu Namen auflösen können.

Mit dieser Option kann man also einerseits ein "schöneres" Logbuch produzieren, andererseits aber Squid auch enorm ausbremsen, weil er für jede Anfrage eines Clients dessen IP-Adresse in einen zugehörigen Namen auflösen soll. Die Standardeinstellung von "no" sollten Sie also nur bei der Verwendung eines eigenen Nameservers oder einer gut gefüllten hosts-Datei auf dem Server ändern.
```
log_fqdn on
```

## OPTIONS FOR EXTERNAL SUPPORT PROGRAMS
Hier werden externe Hilfsprogramme aufgeführt, die von Squid während des laufenden Betriebs genutzt werden. Etwa Filter- oder Authentifizierungsprogramme.

### ftp_user
 Gleich der erste Eintrag scheint nicht so richtig zu passen, allerdings meldet sich Squid mit dem hier eingetragenen Namen bei einer Datenübertragung mit FTP am entsprechenden Server an. Wenn sie also einerseits den Serveradmins etwas genauere Daten für ihre Statistik liefern wollen und auch nicht gleich wieder hinausgeschmissen werden möchten, wenn ein FTP-Server nur dem Nutzer "anonymous" mit einer echten Email-Adresse Zugriff gewährt, dann sollten Sie hier einen 'vernünftigen' Eintrag in der Form: nutzer@domain.de machen. Aber Achtung: manchmal werden die so übermittelten Email-Adressen auch für die Versendung von SPAM genutzt.

Unter Webmin können Sie hier einen Eintrag hinter "Anon. FTP-Anmeldung" machen.
```
ftp_user anonymous@schule.de
```

### ftp_passive

Normalerweise verwendet Squid sowieso passives FTP. Wenn Sie das nicht möchten, können Sie es hier aber abstellen:
```
ftp_passive off
```

### ftp_list_width

Gibt die Anzahl der Zeichen an, die Squid pro Spalte bei der Verwendung des FTP-Protokolls an den Client zurückgibt. Dies ist besonders bei der Verwendung eines Browsers hilfreich, um in FTP-Verzeichnissen etwas mehr von den eigentlichen Dateinamen zu sehen.
```
ftp_list_width 45
```

### redirect_program

Hier können externe "Weiterleitungs-"Programme eingebunden werden. Diese Programme bekommen vom Squid sämtliche Anfragen vom Client durchgereicht, überprüfen diese intern und geben dann an Squid entweder die Anfragen zurück, damit dieser diese holt oder Sie reagieren auf die Anfragen und leiten Sie um.

Ein solches Programm ist z.B. das Programm SquidGuard, welches in einem anderen Kapitel besprochen wird.

Weitere Filterprogramme gibt es wie "Sand am Meer" - suchen Sie doch einfach mal in einer Suchmaschine Ihrer Wahl nach den Begriffen "redirect program squid download"...

Dort werden Sie dann auch so nützliche Programme wie "Adware-Blocker" und ähnliches finden.

Beispielhaft aktivieren wir nun einmal SquidGuard (beachten Sie bitte die komplette Pfadangabe zum Programm!):

```
redirect_program /usr/bin/squidGuard
```

Sie können übrigens mehrere "Redirector"-Programme hintereinander angeben. Squid reicht die URLs dann der Reihe nach von einem Redirector zum anderen weiter. D.h. wenn Sie z.B. zuerst einen AddBlocker als Redirector eingestellt haben, so liefert diese ja eine andere URL an Squid zurück. Der nachfolgende Redirector wird dann diese (neue) URL prüfen.

### redirect_children

Der Parameter redirect_children weist Squid an, eine bestimmte Anzahl von Redirect-Programmen im Voraus zu starten, damit Anfragen von mehreren Clients zügig beantwortet werden können. Der Defaultwert von fünf hat sich aber bewährt und sollte nur in begründeten Ausnahmefällen geändert werden. Keine Angst: surfen mehr als Clients über Squid, werden auch eine entsprechende Anzahl an Redirect-Programmen gestartet - Plus 5 zusätzlichen, die schnell einspringen, wenn plötzlich weitere Clients auftauchen.
```
redirect_children 5
```
Sollten ihre Schüler aber über Verbindungsabbrüche klagen oder im Logfile von Squid verdächtigge Einträge im Zusammenhang mit redirect_children auftauchen - dann drehen Sie hier vorsichtig hoch.

### auth_param

Der neue Squid (ab 2.3) arbeitet nicht mehr mit den bisherigen Parametern authenticate_program" und "authenticate_children".

Sollten Sie diese Parameter noch in Ihrer squid.conf eingetragen haben, werden Sie diesbezüglich Fehlermeldungen beim Starten von Squid bekommen und die Nutzerauthentifizierung findet nicht mehr statt!

Geben Sie - um eine Nutzeranmeldung über die Datei /etc/shadow im Browser zu verlangen - folgende 4 Zeilen in die squid.conf ein:
```
auth_param basic program /usr/sbin/ncsa_auth /etc/shadow
auth_param basic children 5
auth_param basic realm Internetzugang
auth_param basic credentialsttl 2 hour
```
und hier die Erklärung zu den einzelnen Zeilen:
| Wert | Erklärung |
| -- | -- |
| auth_param basic program /usr/sbin/ncsa_auth /etc/shadow | Zur Authentifizierung greift Squid hier auf das Programm ncsa_auth zurück. Das ist das Standardprogramm von Squid. Andere Programme sind z.B. smb_auth, yp_auth oder squid_ldap_auth. (ein Hinweis an dieser Stelle: das Programm ldap_auth hat dieselbe Funktion, wird aber nicht mehr weiter Entwickelt und ist von squid_ldap_auth abgelöst worden). All diese Programme können Nutzer authentifizieren:
* ncsa_auth authentifiziert die Nutzer über eine shadow-Datei, wie Sie Linux z.B. in /etc/shadow verwendet. Dort sind normalerweise alle Nutzer des Systems eingetragen. Da diese Datei aber auch für andere Authentifizierungen (z.B. der Benutzeranmeldung) genutzt wird, sollten Sie hier vorsichtig bei Veränderungen sein. (Achtung: Bitte löschen Sie  keine Nutzer aus der Datei /etc/shadow, wenn Sie Ihnen nur das Surfen im Internet verbieten wollen!)
* smb_auth authentifiziert die Nutzer über Ihre Samba-Passwörter in der Datei smbpasswd. Diese können u.U. von denjenigen in /etc/passwd abweichen: nicht jeder Nutzer, der sich an einem Windows-Client anmelden können soll, soll sich ja auch am Server selbst anmelden können. Ansonsten gilt hier dasselbe wie für ncsa_auth: beachten Sie, dass die Datei smbpasswd durchaus auch für andere Authentifizierungen genutzt wird!
* yp_auth Hier werden Nutzer über einen NIS-Server identifiziert. Dabei reicht das Modul normalerweise nur Benutzername und Passwort an den NIS-Server weiter und erhält von diesem entweder ein "ok" oder eine Fehlermeldung zurück. Hier liegen die Möglichkeiten ganz und gar bei der Konfiguration der NIS-Datenbank. So könnte hier einzelnen Nutzern z.B. eine normale Anmeldung, nicht aber das Surfen im Internet erlaubt werden...
* squid_ldap_auth Auch hier reicht das Modul die Daten nur an einen anderen Server weiter, der die eigentlich Authentifizierung vornimmt und eine entsprechende Antwort an das Modul zurücksendet. Da heute immer mehr Administratoren auf LDAP umsatteln, wird diese Art der Authentifizierung zukünftig wohl eine größere Bedeutung gewinnen, da sich hier - ähnlich wie bei der Authentifizierung über einen NIS-Server - gezielt Unterscheidungen vornehmen lassen. So sähe eine gülitge Zeile mit Zusatzattributen hier wie folgt aus:
```
authenticate_program /usr/sbin/squid_ldap_auth -b dc=Schule,dc=de -f (&(!(internetDisabled=true))(!(gidNumber=103))(uid=%s)) -u uid ldap
``
D.h. im LDAP gibt es für jeden Nutzer einen Eintrag "internetDisabled". Sollte dieser nicht auf "true" stehen oder der anmeldende Nutzer die Gruppen-ID 103 besitzen, dann wird Squid die Anmeldung verweigern - sonst zulassen. |
| auth_param basic children 5 | Squid startet - um Anfragen möglichst schnell beantworten zu können - das Authentifizierungsprogramm gleich mehrfach im voraus. Diese Einstellung sollte für ~150 Rechner ausreichen. Bitte erhöhen Sie sie also nur, wenn die Anmeldung wirklich zu lange dauert, da ansonsten unnötig Speicherplatz verbraucht wird.  |
| auth_param basic realm Internetzugang | Das Wort hinter "realm" wird im Dialogfenster bei der Anmeldung angezeigt. Bei mehreren Wörtern diese bitte in Anführungszeichen setzen. |
| auth_param basic credentialsttl 2 hour | Die Authentifizierung wird die eingestellte Zeit lang aufrecht erhalten. Danach muss sich ein Nutzer erneut anmelden. Sollte der Nutzer zwischenzeitlich den Browser schließen, muss er sich allerdings sowieso erneut anmelden. Eine kürzere Zeitspanne dürfte also kaum jemanden auffallen. |

## OPTIONS FOR TUNING THE CACHE

Nachdem wir nun schon im Vorfeld so viel an "Tuning-Möglichkeiten" kennen gelernt haben, bietet uns die Konfigurationsdatei nun sogar noch einen eigenen Abschnitt dafür an.

### request_header_max_size

Hiermit wird die maximale Größe eines HTTP-Headers einer Anfrage festgelegt. Diese Header stehen am Anfang einer HTML-Seite und werden nicht vom Browser angezeigt. Normalerweise haben Header eine Größe von einigen hundert Byte. Diese Option dient dazu, fehlerhafte Anfragen, z.B. durch fehlerhafte Client-Software oder einen DoS-Angriff abzufangen. Die Standardeinstellung von 10 kB ist ein wenig übertrieben: 3 KB reichen da normalerweise aus. Achten Sie aber darauf, dass dies bei den immer mehr in Mode kommenden Groupware-Lösungen durchaus zu Problemen führen kann, wenn Nutzer Dateien von Ihrem Arbeitsplatz auf den Server hochladen wollen.
```
request_header_max_size 3 KB
```

### refresh_pattern

Mit den Refresh-Pattern kann das Verhalten von Squid bei Anfragen eines Clients massiv beeinflusst werden. So können regelwidrige Refresh Pattern auch zur Auslieferung längst veralteter Webseiten oder FTP-Dateien führen. Allerdings ergibt sich hier auch eine optimale Möglichkeit zum Cache-Tuning.

| Anweisung | Option | regulärer Ausdruck | Mindestzeit | Prozentsatz | Maximalzeit | weitere Optionen |
| -- | -- | -- | -- | -- | -- | -- |
| Die "refresh_pattern" Zeilen werden der Reihe nach durchlaufen. Es gilt die erste Zeile, deren regulärer Ausdruck zutrifft. | Normalerweise wird zwischen Groß- und Kleinschreibung unterschieden. Mit der Option -i wird das nicht berücksichtigt. | Der reguläre Ausdruck bestimmt, für welche URLs die Anweisung gilt. | Die bei min angegebene Mindestzeit gibt (in Minuten) die Zeit an, die Objekte ohne ausdrückliches Verfallsdatum mindestens als aktuell (fresh) betrachtet werden. 43200 bedeutet also 30 Tage. | percent ist der Prozentsatz des Objektalters (Zeit seit der letzten Änderung) die Objekte ohne ausdrückliches Verfallsdatum mindestens als aktuell (fresh) betrachtet werden. | max ist der maximale Zeitraum, den Objekte ohne ausdrückliches Verfallsdatum als aktuell (fresh) betrachtet werden. | options beinhaltet eine oder mehrere Optionen, durch Lerraum getrennt. Auf die einzelnen Optionen geht die untere Tabelle ein. |
| refresh_pattern |  | ^ftp: | 43200 | 20% | 43200 | reload-into-ims |
| refresh_pattern |  | ^gopher: | 43200 | 0% | 43200 | reload-into-ims |
| refresh_pattern | -i | \\.jpg | 43200 | 100% | 43200 | reload-into-ims |
| refresh_pattern | -i | \\.gif | 43200 | 100% | 43200 | reload-into-ims |
| refresh_pattern | -i | \\.tif | 43200 | 100% | 43200 | reload-into-ims |
| refresh_pattern | -i | \\.bmp | 43200 | 100% | 43200 | reload-into-ims |
| refresh_pattern | -i | \\.png | 43200 | 100% | 43200 | reload-into-ims |
| refresh_pattern | -i | windowsupdate.com/.*\\.(cab|exe) | 43200 | 100% | 43200 | reload-into-ims |
| refresh_pattern | -i | download.microsoft.com/.*\\.(cab|exe) | 43200 | 100% | 43200 | reload-into-ims |
| refresh_pattern |  | . | 20160 | 50% | 43200 |  |

...und hier noch einmal die möglichen Optionen für die letzte Spalte:

| Option | Bedeutung |
| -- | -- |
| override-expire | erzwingt die unter "min" angegeben "Mindesthaltbarkeit", auch wenn vom Webserver ein "Expires:" im Header gesendet wurde. |
| override-lastmode | erzwingt die unter "min" angegeben "Mindesthaltbarkeit", auch wenn das Objekt häufig geändert wird. |
| reload-into-ims | Ändert eine "no-cache" oder "reload" Anfrage in ein "If-Modified-Since" - das sollte die sinnvollste Voreinstellung sein, da Squid so zwar eine Anfrage ins Internet aussendet - oft aber die Seite aus seinem eigenen Cache holen kann. |
| ignore-reload | ignoriert ein "no-cache" oder "reload" in einer Anfrage. |

Bitte beachten Sie, dass Squid bei einigen Einstellungen gegen die w3d-Konventionen verstößt. Mit den oben gemachten Einstellungen werden aber vorwiegend Grafiken besser zwischengespeichert, die sich - wie die Erfahrung zeigt - kaum verändern.

Die beiden vorletzten refresh-pattern sorgen (als Beispiel für komplexere Ausdrücke) dafür, dass Windows-Updates im Cache zwischengespeichert werden. Damit gelingt dann das Updaten der Windows-Rechner ein wenig schneller. Natürlich könnte man auch die vorherigen Zeilen für die Bilder zusammenfassen - hier habe ich einmal aus Gründen der Übersichtlichkeit darauf verzichtet.

Der letzte Eintrag sorgt mit dem "." (Punkt) dafür, dass alle angeforderten Seiten, Grafiken, etc. für mindestens 14 Tage zwischengespeichert werden, solange sie sich nicht ausdrücklich geändert haben.