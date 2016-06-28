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