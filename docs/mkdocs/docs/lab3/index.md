# :fa-flask: Lab 3: Komplexe Zusammenstellungen mit Docker-Compose

Bisher haben Sie Docker-Container mit `docker run` gestartet. Schon in den ersten Beispielen, in denen Volumes und Ports zum Einsatz kamen, wurden die Befehle lang und unübersichtlich.

Ein Befehl wie 

```bash
docker run \
-p 80:80 \
-p 443:443 \
-v /path/to/file:/etc/file \
-v /config.conf:/etc/config.conf \
-e DATABASE='foo' \
-e DATABASE_HOST='bar' \
-e DATABASE_PASSWORD='secret' \
--name testserver nginx:latest
```

ist auf Dauer keine Lösung. Das Grundprinzip "Ein Prozess, ein Container" hat zur Folge, dass man für die meisten Anwendungen mehr als einen Container braucht. Zum Glück gibt es Docker-Compose – das ist der Weg, wie Sie zukünftig mit Docker arbeiten möchten.

Prüfen Sie, ob Docker-Compose installiert ist:

```
docker-compose version
```


Docker-Compose arbeitet mit YAML-Dateien. YAML ist eine Auszeichnungssprache für Objekte und in der Cloud-Native-Welt. YAML arbeitet mit Einrückungen zur Auszeichnung von Verschachtelungen:

```yaml
auto:
  name: Golf
  hersteller:
    name: Volkswagen
    ort:
      name: Wolfsburg
      plz: 38440
  tags:
   - auto
   - kfz
```


 Ein gutes Beispiel ist ein WordPress-Blog. WordPress braucht:

* einen Container mit dem Webserver (Apache)
* einen Datenbankserver (MariaDB oder MySQL)



```yaml
version: '3.8'

services:
   db:
     image: mysql:5.7
     volumes:
       - ./data/db/:/var/lib/mysql
     restart: always
     environment:
       MYSQL_ROOT_PASSWORD: secret123root
       MYSQL_DATABASE: wordpress
       MYSQL_USER: wordpress
       MYSQL_PASSWORD: 12345wordpress54321

   wordpress:
     depends_on:
       - db
     image: wordpress:latest
     ports:
       - "80:80"
     restart: always
     environment:
       WORDPRESS_DB_HOST: db:3306
       WORDPRESS_DB_USER: wordpress
       WORDPRESS_DB_PASSWORD: 12345wordpress54321
     working_dir: /var/www/html
     volumes:
       - ./data/wordpress:/var/www/html/wp-content
```