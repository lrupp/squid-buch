# Die Konfigurationsdatei

Die Konfigurationsdatei von Squid ist mit rund 125 Einträgen wohl neben der des Apache eine der längsten Konfigurationsdateien, die es bei einem Server gibt. Glücklicherweise ist ein Großteil
des Inhalts eine (englische) Hilfe für die entsprechenden Konfigurationsmöglichkeiten (erkennbar am vorangestellten 'Gartenzaun').

Eine weitaus komplettere (englische) Dokumentation findet sich übrigens auf den [Webseiten](http://www.squid-cache.org/) des Squid-Projektes. 


## Network
In diesem Abschnitt finden Sie alle Parameter, die für den Betrieb von Squid in einem Netzwerk erforderlich sind. So muß Squid u.a. wissen:
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



