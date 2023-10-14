# NGINX Grundladen

## Kommentare
`nginx -v` -> zeigt Versions und Konfigurations-Argumente an
`nginx -t` -> testet die aktuelle Konfiguration   
`nginx -s reload` -> läd die aktuelle Konfiguration (bei fehlern bleit die alte bestehen)   
`systemctl status nginx` -> status von nginx (zeigt auch die Worker an)   

## Konfiguration
### Minimal Konfiguration
`user` definiert den nginx user (optional | default: nobody)   
`worker_processes` definiert die anzahl der laufenden Worker (optional | default: 2) -> Auto für Workeranzahl = CPU-Kerne
`events`-Block ist mandatory   
`http`-Block definiert die http Kommunication   
`server`-Block definiert die Virtual-Hosts   
`types`-Block definitiert Mime-Type to File-Endung   

```
events {}

http {

	include mime.types;				# mit gelieferte Mime-Types to File-Endung

	server {
		listen 80;					# port (http/https)
		server_name: *.foobar.com  	# wie ist der server erreichar
		root /var/www            	# wo liegt die Webseite
	}
}
```

### Location Direktive
`location`-Block definifert eigene Einstellungen (Abweichend vom `server`-Block)   

```
...
server {
	...

	root /var/www

	location ~ /name {
		return 200 'Hier wird /name als prefix interpretiert';
	}

	location = /haus {
		return 200 'Dies ist eine expliziten location';
	}

	location = /strasse[0-9] {
		return 200 'Auch Regulere Ausdrücke sind möglich (benötigt pcre Erweiterung)';
	}

	location ~* /hausnummer[A-D] {
		return 200 'Hierbei sind die Reguleren Ausdrücke nicht mehr Case-Sensitiv';
	}

	location ^= /strasse5 {
		return 200 'Dieser Eintrag vor bevorzugt (anstelle der Reguleren Ausdrücke) verwenden.';
	}

	# eine Location kann auch die Basis-Konfifuration überschreiben
	location ~ .(png|jpg|jpeg|gif|ico)$ {
		root /var/storage;

	}
}
```

### Variablen
Verfürbare Basis-Variablen sind unter nginx.org zu finden.   

```
location ~ /variablen {
	return 200 "$host - $connection - $date_local - $arg_firma";
}
```
`$arg_` entspricht den GET-Parametern in der URL  
Basis-Variablen können nicht überschrieben werden (invalide Konfiguratoin)   

Eigene Varaiblen können mit `set` definirt werden   
```
server {
	...

	set $number 0;
	location = /elf {
		set $number 11;
	}

	return 200 "Number: $number";
}
``` 

### Kondition
```
server {
	root /var/www;

	if ($data_locale ~ "Saturday|Sunday") {
		root /var/www/angebote
	}
}
```

### Rewrite
```
server {
	...
	location ~ /hidden-rewrite {
		rewrite ^/$ https://www.foobar.de;
	}

	location ~ /temp-rewrite {
		rewrite ^/$ https://www.foobar.de redirect;
	}

	location ~ /perm-rewrite {
		rewrite ^/$ https://www.foobar.de permanent;
	}

	location ~ /with-pages {
		rewrite ^/(.*)$ https://www.foobar.de/$1 permanent;
	}
}
```

### Try Files
`try_files` versucht die Dateien nach und nach zu finden   
```
server {
	try_files /bilder/foobar.png /error.png;
}
```
   
Es kann auch auf locations verwiesen werden   
```
server {
	try_files $uri /index.html /404;

	location /404 {
		return 404 "Page not found";
	}
}
```

Auch verlinkungen sind erlaubt   
```
server {
	try_files $uri @404;

	location @404 {
		return 404 "Page not found \n$uri";
	}
}
```

## Logging

`access.log` -> enthält jeder Zugriff
`error.log` -> enthält jeder Error ab einem bestimmtes Level (kein 404)
   
### Log-Files
```
server {
	access_log /var/log/nginx/access.log;
	access_log /var/log/nginx/domain-specific-access.log;	# schreibt den Eintrag ZUSÄTZLICH in ein anderes Log rein

	error_log /var/log/nginx/error.log;
	error_log /var/log/nginx/important.log crit;			# schreibt alle CRIT-Meldungen (und höher) in ein andere Logs

	# logs deaktivieren
	access_log off;
	error_log off;
}
```

### Log-Format
```
server {
	log_format	dsgvo	'xxx.xxx.xxx.xxx - $remote_user [time_local] "$request" '
						'$status $body_bytes_send "$http_referer" '
						'"$http_user_agent" "$http_x_forwared_for"';

	assecc_log /var/log/nginx/clear.log dsgvo;
}
```

### Logrotation
Logrotate für das jeweilige System installieren   
Konfiguration zu finden unter `/etc/logrotate.d/`   
   
`/etc/logrotate.d/nginx`
```
/var/log/nginx/*.log {					# Pfad der Datei
        daily							# Täglich Rotieren
        missingok						# kein fehler bei fehlenden Log-Files
        rotate 7						# Anzahl der Rotation (Anzahl der Logfiles)
        compress 						# gz-Komplemieren
        notifempty 						# leere Dateien aussehen/überspringen
        create 0640 www-data adm 		# neue Dateien werden mit den Rechten/User/Gruppe angelegt
        sharedscripts 					# unterbindet multiple ausführung
        prerotate 						# wird vor dem ausführen gestartet
                if [ -d /etc/logrotate.d/httpd-prerotate ]; then \
                        run-parts /etc/logrotate.d/httpd-prerotate; \
                fi \
        endscript
        postrotate 						# wird nach dem ausführen gestartet
                invoke-rc.d nginx rotate >/dev/null 2>&1
        endscript
}
```


## PHP
1. PHP-FPM für das jeweilige System installieren   
2. den php-fpm Socket finden `ls /var/run/php`
3. in der NGINX-Konfiguration angeben
```
server {
	index index.php index.html;

	location ~\.php {
		fastcgi_pass unix:/var/run/php/php8.1-fpm.sock;
		fastcgi_index index.php;
	}
}
```
auf die Bereichtigungen der Prozesse achten   
`ps aux | grep nginx` und `ps aux | grep php`