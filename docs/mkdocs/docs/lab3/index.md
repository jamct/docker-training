# :material-flask: Lab 3: Komplexe Zusammenstellungen mit Docker-Compose

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
docker compose version
```

## Grundlagen: YAML

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

## 2. WordPress in Containern

Ein gutes Beispiel ist ein WordPress-Blog. WordPress braucht:

- einen Container mit dem Webserver (Apache)
- einen Datenbankserver (MariaDB oder MySQL)
- der Webserver muss mit seiner Datenbank sprechen können.
- Port 80 des Containers muss auf der externen Netzwerkkarte lauschen

Legen Sie eine Datei mit dem Namen "docker-compose.yml" an. Das ist der Standardname, den Docker-Compose erwartet. Die folgende Zusammenstellung beschreibt eine simple WordPress-Zusammenstellung:

```yaml
version: "3.8"

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

Wenn die Datei auf dem Server liegt, können Sie die Zusammenstellung starten:

```
docker compose up
```

Jetzt brauchen Sie etwas Geduld: Docker lädt die unhandlichen Images aus dem Hub herunter und fährt die Datenbank hoch. Das dauert, weil die Datenbank noch leer ist – MySQL richtet sich erstmal ein.

Wie bei `docker run` gibt es auch bei Docker-Compose einen detached-Modus. Ein fertiges Setup würden Sie mit starten mit:

```
docker compose up -d
```

Das Gegenteil von `up` lautet (logisch):

```
docker compose down
```

Und auch andere bekannte Befehle finden Sie wieder:

```
docker compose ps
docker compose exec -it <service> sh
```

## 2.2 Umgebungsvariablen

Die Konfiguration gehört nicht in die Compose-Datei! Sie sollten die Compose-Datei so flexibel wie möglich halten. Dann ist es leicht, eine Dev-, Staging- und Prod-Umgebung aus demselben File zu erzeugen.

Docker-Compose arbeitet dafür mit Ersetzungen:

```yaml
wordpress:
  depends_on:
    - db
  image: "wordpress:${TAG}"
  ports:
    - "80:80"
  restart: always
  environment:
    WORDPRESS_DB_HOST: db:3306
    WORDPRESS_DB_USER: ${DB_USER}
    WORDPRESS_DB_PASSWORD: ${DB_PASS}
  working_dir: /var/www/html
  volumes:
    - ./data/wordpress:/var/www/html/wp-content
```

Diese Ersetzungen kommen aus einer Datei mit dem Namen `.env`, die sich neben dem Compose-File befinden muss:

```
TAG=latest
DB_USER=wordpress
DB_PASS=secret
```

## 2.2 Wenn es mal wieder länger dauert

Ein häufiges Problem: Ich weiß, dass meine Anwendung beim Stoppen länger braucht – weil ich zum Beispiel erst alle Daten aus dem Speicher sichere. Normalerweise fackelt Docker nicht lange und sendet schon 10 Sekunden nach einem `SIGTERM`direkt den `SIGKILL`. Wenn ich meiner Anwendung die Zeit geben will, in Ruhe herunterzufahren, teile ich das im Compose-File mit:

```
slow_service:
  image: slow-image:slow
  stop_grace_period: 2m
```

## 3: Docker-Compose macht vieles einfacher:

- Die Struktur wird schnell sichtbar
- Mit `depends_on` bestimmen Sie (grob), welcher Container zuerst startet
- Pfadangaben funktionieren relativ vom Pfad des Docker-Compose-Files

!!! note "Infrastruktur ist Code"
Um dem Ziel einen wesentlichen Schritt näher zu kommen, eine reproduzierbare Umgebung zu bauen, sollten Sie Ihre Docker-Compose-Files von Anfang an in eine Versionsverwaltung legen (Git). Und auch Dockerfiles von eigenen Images gehören in die Versionsverwaltung. Im nächsten Schritt wird dann automatisch gebaut (Continuous Integration).

    Guter Stil ist es, eine vorbereitete `.env-example` bereitzulegen. Die kann ein Nutzer des Repos umbenennen (`mv .env-example .env`) und direkt starten.
