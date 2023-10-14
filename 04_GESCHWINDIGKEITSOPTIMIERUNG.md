# Geschwindigkeitsoptimierung

## Browser-Caching
```
server {
	location ~* \.(jpg|jpeg|png|gif|ico) {
		expire 30d;			# 30 Tage
		expire 1m;			# 1 Monat

		add_header Cache-Controll public;		# HTTP 1.1 und neuer
		add_header Pragma public;				# vor HTTP 1.1
		add_header Vary Accept-Encoding;		# neue Browser bekommen compremierte Version, ältere Browser plain Version
	}
}
```

## Komplremierung
```
http {
	gzip on;
	gzip_vary on;
	gzip_proxied any;
	gzip_disable "msie6";				# ms-ie6 hat keine komplremierung (alle vorherigen werden mit deaktiviert)
	gzip_http_version 1.1;
	gzip_min_length 250;
	gzip_buffers 32 16k;
	gzip_comp_level 3;					# hohe Kompression = hohe Serverauslastung
	gzip_types text/html text/plain;	# mehrere Möglich | default: text/html 
}
```

## FastCGI Cache
```
http {
	fastcgi_cache_path /var/cache/nginx level=1:2 key_zone=APPLICATION:100m inactive=60m;
	fastcgi_cache_key "$scheme$request_method$host$request_uri";

	server {
		location ~\.php {
			fastcgi_cache APPLICATION;						# key_zone aus der definition
			fastcgi_cache_valid 200 404 10m;				# mit "any" werden alle Statuscode gespeichert | default statuscode; 200, 302, 404

			add_header X-Cache $upstream_cache_status;		# für die Clients (Browser)
		}
	}
}
```

Beachten, dass nicht alles gecacht wird (z.B. Admin-Oberfläche oder Formulare)   
```
http {
	server {
		set $no_cache 0;

		# kein Cache für POST
		if ($request_method = POST) {
			set $no_cache 1;
		}

		# kein Cache bei URL-Parameter
		if ($query_string != "") {
			set $no_cache 1;
		}

		# kein Cache für Admin
		if ($request_uri ~* "/admin") {
			set $no_cache 1;	
		}

		location ~\.php {
			fastcgi_cache_bypass $no_cache;
			fastcgi_no_cache $no_cache;
		}
	}
}
```