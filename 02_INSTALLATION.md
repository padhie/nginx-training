# Installation

## CentOS
* installation muss via EPEL erfolgen (RedHead hat Nginx nicht im default Repository)

## Debian
* `apt update && apt upgrade && apt install nginx`

## via SourceCode
> für Benutzerdefinierte Einstellungen oder wenn nginx nicht für die Distrubtion verfügbar ist
* ningx.org -> Freeware
* nginx.com -> Commerzial
* tar-Datei herunterladen und entpacken
* Entwicklertools installieren
** Debian: apt install build-essential libpcre3 libpcre3-dev zlib1g zlib1g-dev libssl-dev
** CentOS: yum groupinstall "Development Tools" && yam install pcre pcre-devel zlib zlib-dev openssl openssl-devel
* `./configure`
** für Benutzedefinierte Einstellungen `./configure --....`
** Options können auf nginx.org/nginx.com nachgelesen werden
* `make`
* `make install`
* Autostartservice (z.B. SystemD) einrichten
** fertige Konfiguration kann auf nginx.org/nginx.com gefunden werden