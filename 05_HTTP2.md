# HTTP/2

## Vorbereitung - SSL
* SSL Zertifikat erstellen/benötigt
* Openssl 1.0.2 oder höher

nginx.conf
```
http {
	...
	listen 443 ssl;
	ssl_certificate /etc/ssl/zertifikat.crt;
	ssl_certificate_key /etc/ssl/Zertifikats_key.key;
	...
}
```

## HTTP/2 aktivieren & konfigurieren
Beim selbst copelieren von nginx muss ssl_module und https_v2_module mit installiert werden.   

### Vorteile - Funktionen
* Binäres Datenübertragung -> schneller verarbung
* Parallele Verarbeitungen -> mehrere Anfragen wird als gesammtpaket ausgeliefert
* Header Komprimierung -> große Header schnell und effizient ausliefern
* Priorisierung von Streams -> z.B. nur das ausliefert was sichtbar ist
* Server-Push -> Notwendige Daten mit schicken die eh benötigt werden, ohne auf Browseranfrage zu warten

nginx.conf
```
http {
	...
	listen 443 ssl http2;
	...
}
```

## HTTP/2 Preload & Server-Push

ohne Server-Push
```
[Browser]			           [Server]
    |                             |
    | --- Anfrage index.html ---> |
    | <--- Anwort index.html ---- |
    |                             |
    | ---- Anfrage CSS-Datei ---> |
    | <--- Antwort CSS-Datei ---- |
    |                             |
    | ------ Anfrage Logo ------> |
    | <------ Antwort Logo ------ |
    V 				              V
```


mit Server-Push
```
[Browser]			           [Server]
    |                             |
    | --- Anfrage index.html ---> |
    | <--- Anwort index.html ---- |
    | <--- Antwort CSS-Datei ---- |
    | <------ Antwort Logo ------ |
    |                             |
    V 				              V
```

Pro
* optimierte Antworten
* schnelle verarbeitung
* weniger Kommunikation -> schnellerer Seitenaufbau

Kontra
* Caching greift hier auf alle Dateien
* kann nicht jeder Browser (muss HTTP/2 unterstützen)


### Preload
preload   
```
location = /index.html {
	add_header Link "<style.css>; as=style; rel=preload, <logo.png>; as=image; rel=preload";
}
```

preload mit cache
```
server {
	location = /index.html {
		add_header Set-Cookie "session=1";
		add_header Link $resources;
	}

	map $http_cookie $resources {
		"~*session=1" "";
		default "<style.css>; as=style; rel=preload, <logo.png>; as=image; rel=preload";
	}
}
```

### Server-Push
```
server {
	http2_push /style.css;
	httpe_push /logo.png

	location = /index.html {
		http2_push /style2.css;
		httpe_push /logo2.png
	}
}
```
