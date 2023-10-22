# Reverse-Proxy und Load Balancer

## Warum Revers-Proxy nutzen?
* optimiert auf Caching   
* man kann Sicherheit Primär auf Web<=>Reverse-Proxy setzen, da der rest isoliert und nicht erreichbar von außen ist   
* Single URL-Entrypoint möglich (eine URL mit x Services dahinter)   

## Reverse-Proxy
* leitet eine Anfrage 1:1 an einen dahinter liegenden Dienst weiter   
* muss nicht auf dem gleichen Server sein, nur erreichbar   

## Load Balancer
* jeder Dienst muss mindestens 2x vorhanden sein   
* verteilt die Last möglichst gleichmäßig (je nach Einstellung)   
* man kann einzelne Server/Services/Dienste aus dem System nehmen, Updaten und danach wieder hinzufügen   

### Load Balancer mit Session persistence
Z.B. wird ein Formular immer auf genau ein einzigen Server weiter geleitet.   
Geeignet wenn man z.B. einen dedizierten Login-Server braucht/will.   

## Beispie anhand von PHP-Script mit verschiedenen Ports
3 php-Scripte erstellen (server7001.php / server7002.php / server7003.php)   
```
<?php
echo date('h:i:s') . PHP_EOL;
echo '<h1>Server Port 7001</h1>';
```
Hierbei unterschiedliche Ports bei den scripten eintragen.   
   
Nun die Scripte via Dev-Server starten und im huntergrund legen   
```
php -S localhost:7001 server7001.php &
php -S localhost:7002 server7002.php &
php -S localhost:7003 server7003.php &
```

nginx.conf anpassen   
```
server {
	...

	# schicke Headerinfos mit
	add_header proxyby nginx-proxy;
	proxy_set_header proxyby nginx-proxy;
	proxy_set_header HOST $host;
	proxy_set_header X-Forwarded-Proto $scheme;
	proxy_set_header X-Real-IP $remote_addr;
	proxy_set_header X-Forwarde-For $proxy_add_x_forwarded_for;

	# server7001.php
	location /server1 {
		proxy_pass 'http://localhost:7001/';
	}

	# server7002.php
	location /server2 {
		proxy_pass 'http://localhost:7002/';
	}

	# server7003.php
	location /server3 {
		proxy_pass 'http://localhost:7003/';
	}
}
```

## Load Balancer Direktive
Mit Direktiven wird festgelegt, wie die Verteilung vorgenommen werden soll.   

| Methode | Direktive | Beschreibung |
| -------- | -------- | -------- |
| RoundRobin | - | Ein Server nach dem anderen |
| Geringste Verbindungen | least_conn | Der Serever, mit der wenigsten Verbindung wird bevorzugt |
| Gewichtung | weight | Der Server erhalten eine Gewichtung/Priorität |
| IP Hash | ip_hash | Ein Besucher wird immer zum gleichen Server verbunden (basierend über die IP-Adresse) |


## Load Balancer konfiguration

### RoundRobin
```
http {
	upstream xyz_alias {
		server localhost:7001;
		server localhost:7002;
		server localhost:7003;
	}

	server {
		location /loadbalance {
			proxy_pass 'http://xyz_alias/';
		}
	}
}
```

### Geringste Verbindungen
```
http {
	upstream xyz_alias {
		least_conn;
		server localhost:7001;
		server localhost:7002;
		server localhost:7003;
	}

	server {
		location /loadbalance {
			proxy_pass 'http://xyz_alias/';
		}
	}
}

### Gewichtung
```
http {
	upstream xyz_alias {
		server localhost:7001;
		server localhost:7002 weight=2;
		server localhost:7003;
	}

	server {
		location /loadbalance {
			proxy_pass 'http://xyz_alias/';
		}
	}
}
```

### IP-Hash
```
http {
	upstream xyz_alias {
		ip_hash;
		server localhost:7001;
		server localhost:7002;
		server localhost:7003;
	}

	server {
		location /loadbalance {
			proxy_pass 'http://xyz_alias/';
		}
	}
}
```