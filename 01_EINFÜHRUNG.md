# Einführung

## Warum NGING nutzen?
* Geschwindigkeitsvorteil
* Leichtgewicht
* Einfache Konfiguration
* wachsende Popularität

## Wann kein NGINX nutzen?
* wenn Apache zwingend benötigt wird (z.B. für bestimmte Module)
* keine/schlechte Möglichkeit für Konfiguration auf Verzeichnisebene
* bei Shred Hosting Dienste (wegen eigenständige Konfiguration)
* kein eigenständiges PHP-Module (muss extra configuriert werden)

## night nur ein Webserver
* Modular aufgebaut
* als Reverse Proxy einsetzbar (Web und Mail)
* als Load Balancer einsetzbar
* Integriertes Caching
* Streaming möglich

## NGINX vs. Apache
|NGINX|Apache|
|-----|------|
| Ereignisgesteuer | Prozessgesteuer |
| Feste Prozessanzahl | neue Prozesse bei mehr anfragen |
| optimitert für Statischen Content | optimiert für dynamischen Content |
| Leichtgewicht | großer Funktionsumfang |
| Hardware schonen | stark von Hardware/Resourcen abhängig |
| URL-Basierte LoKationen | Dateibasierte Lokationen |
| erster Match in Konfiguration greift | letzter Match in Konfiguration greift |