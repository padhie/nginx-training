# Sicherheit mit nginx

## Grundlagen
* Systeme & Dienste immer auf den neusten Stand halten
* Passwörter und Sensible Informationen schützen
* SSL und Verschlüsselungen (HTTPS) verwenden
* Zugänge Beschränken (Passwort, IP)

## Datei-/Orderrechte
* 644 für Dateien empfohlen (`find . -type f -exec chmod 0644 {} \;`)
* 755 für Dateien empfohlen (`find . -type d -exec chmod 0755 {} \;`)
* nginx oder www-data als Benutzer/Gruppe 

## nginx Server-Informationen verstecken
```
http {
	server_tokens off;
}
```

## IP-Restriction
```
location /admin {
	allow xxx.xxx.xxx.xxx;
	allow xxx.xxx.xxx.0/24;
	deny all;
}
```

## Eigene Error-Seite
```
server {
	error_page 403 /403.html;
	location /403.html {
		# root /var/www/error;
		internal;
	}
}
```

## Verzeichniss-Listen - Autoindex
Autoindex zeigt alle Inhalt eines Verzeichnis an, sofern keine index.html existiert.   
Default hier ist `off`.   
```
server {
	autoindex on;
}
```

## htpasswd
```
server {
	location /admin {
		auth_basic "Benuzterlogin erforderlich";
		auth_basic_user_file /etc/nginx/htpasswd;
	}
} 
```

## Browser ausschließen
Kann z.B. genutzt werden um nur Kompatible Browser zu unterstützen (oder auch Suchmaschienen).   
```
server {
	if ($http_user_agend ~* Google|Chrome|wget|Bot) {
		return 403;
	}
}
```

## HTTP -> HTTPS
```
server {
	listen 80;
	server_name	your.domain.bla;
	return 301 https://$server_name$request_uri;
}

server {
	listen 443;
	...
}
```