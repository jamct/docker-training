# :fa-flask: Lab 6: HTTP-Routing mit Traefik

Bisher haben wir die Server-Anwendungen alle nacheinander gestartet. Es darf ja immer nur ein Prozess Port 80 belegen. So kann man eine Maschine natürlich nicht effizient nutzen. Man braucht einen Router, der eingehenden (HTTP-)Verkehr entgegennimmt und an den richtigen Container weitergibt. Nur der Router lauscht an Port 80 – und an Port 443, weil er ganz nebenbei noch für TLS und Zertifikatsbeschaffung zuständig ist.

Ein solcher Router, der wahrhaft "Cloud-Native" ist und sich gut auf einem Docker-Host macht, ist Træfik (oder Traefik). Wie alle verwendeten Tools ist auch der Open-Source, es gibt auch eine Enterprise-Edition mit Supportvertrag.

## Traefik einrichten

![ ](https://heise.cloudimg.io/width/900/q65.png-lossy-65.webp-lossy-65.foil1/_www-heise-de_/select/ct/2019/17/1565693316870479/contentimages/image-1564472350617001.jpg){: class="noprint"}

Für die Traefik-Eintrichtung sollten Sie am besten ein eigenes Repository mit dem Compose-File und einigen Config-Files anlegen. In die Compose-Datei stecken Sie ggf. noch Watchtower und Portainer. Ins Repo kann man dann noch ein Bash-Script legen, das Docker installiert. Dann hat man einen Ordner mit allen Rezepten für einen neuen Docker-Server.

Die Arbeit beginnt mit dem Compose-File `docker-compose.yml`:

```yaml
version: "3.7"
services:
  traefik:
    image: traefik:v2.2
    command: --providers.docker
    restart: always
    ports:
      - 80:80
      - 443:443
    volumes:
      - /var/run/docker.sock:/var/run/docker.sock:rw
      - ./traefik/static.yml:/etc/traefik/traefik.yml
      - ./traefik/dynamic.yml:/etc/traefik/dynamic/dynamic.yml
      - ./traefik/acme.json:/etc/traefik/acme/acme.json
    networks:
      - router
    
# [...] Ergänzen Sie ggf. Watchtower, Portainer und andere Tools nach Belieben...

networks:
  router:
    external: 
      name: router-network

```

Hier kommt erstmals ein neues Konzept zum Einsatz: Der Router wird an ein Netzwerk mit dem Namen `router` gebunden. Am Ende der Datei wird dieses Netzwerk bekannt gemacht: Docker-Compose soll ein extern erzeugtes Netzwerk, das draußen `router-network` heißt, innerhalb des Files als `router` bekannt machen.

Bevor Sie das Netzwerk nutzen können, muss es einmalig auf dem Docker-Host eingerichtet werden (auch das gehört in ein Installations-Skript):

```
docker network create --driver=bridge --attachable --internal=false router-network
```

Jetzt fehlen noch drei Dateien, mit denen Traefik gesteuert wird. Legen Sie innerhalb des Repos (oder für die Experimentierphase zunächst auf dem Server) den Unterordner "traefik" an und dort die Datei "static.yml":

```yaml
entryPoints:
  web:
    address: ":80"
  web-secure:
    address: ":443"

providers:
  docker:
    watch: true
  file:
    directory: /etc/traefik/dynamic
    watch: true
    filename: dynamic.yml

certificatesResolvers:
  default:
    acme:
      email: jam@ct.de
      storage: /etc/traefik/acme/acme.json
      httpChallenge:
        entryPoint: web
```

Traefik arbeitet wie folgt: Beim Start liest es die Static-Yaml-Datei ein. Darin stehen Dinge für den Grundbetrieb. Ändert man hier etwas, muss man Traefik neu starten (und den HTTP-Router im Produktivsystem will man nicht ständig neu starten).

Traefik soll Zertifikate bei Let's Encrypt bestellen und sie automatisch austauschen (nie wieder Arbeit mit Zertifikaten). In der Static-Datei ist nur eine Änderung nötig: Ersetzen Sie die Mailadresse durch Ihre. Die Mailadresse wird von Let's Encrypt benutzt, Sie per Mail zu warnen, wenn ein Zertifikat ausläuft und nicht automatisch erneuert wurde (nach meiner Erfahrung nie). Die Adresse wird nicht im Zertifikat veröffentlicht!

In der statischen Datei wird auf eine dynamische referenziert. Legen Sie `dynamic.yml` an:

```yml
http:
  routers:
    https-redirect:
      rule: "HostRegexp(`{any:.*}`)"
      middlewares:
        - https-redirect
      service: redirect-all
  middlewares:
    https-redirect:
      redirectScheme:
        scheme: https
  services:
    redirect-all:
      loadBalancer:
        servers:
          - url: ""
```

Diesen Block müssen Sie jetzt nicht Zeile für Zeile verstehen. Er kümmert sich darum, dass Anfragen an `http://` automatisch auf `https://` umgeleitet werden.

Jetzt fehlt noch eine letzte Datei, die am Anfang leer ist. Legen Sie die `./traefik/acme.json` einfach an und setzen Sie dann die Rechte:

```
chmod 600 acme.json
```

Dieser Schritt ist dringend notwendig. In dieser Datei speichert Traefik die Zertifikate.


Das Repository sieht jetzt so aus:

```
docker-compose.yml
traefik/
  acme.json
  static.yml
  dynamic.yml
```

Wenn Sie das Netzwerk angelegt haben, ist es an der Zeit, Traefik zu starten. Traefik steht auf `restart: always`, wird nach einem Neustart also zurückkommen.

Fahren Sie die Zusammenstellung hoch:

```
docker compose up -d
```

Bisher passiert noch nichts. Wenn Sie eine beliebige Subdomain Ihres Servers im Browser öffnen, sehen Sie eine Fehlermeldung. Es gibt ja noch keinen Dienst, den Traefik veröffentlichen könnte.

Es ist an der Zeit, Ihr selbstgebautes Image zu veröffentlichen (mit TLS und unter einem schönen Hostnamen). Wechseln Sie dazu das Verzeichnis (Traefik arbeitet einfach still im Hintergrund und braucht unsere Aufmerksamkeit heute nicht mehr) und legen Sie eine neue Datei `docker-compose.yml` an:

```yaml
version: "3.8"
services:
  docs:
    image: docker.pkg.github.com/jamct/demo-docs/docs:latest
    restart: always
    labels:
      - "traefik.http.routers.docs.rule=Host(`docs.00.liefer.it`)"
      - "traefik.http.routers.docs.tls.certResolver=default"
      - "traefik.http.routers.docs.tls=true"
      - "com.centurylinklabs.watchtower.enable=true"
    networks:
      - router
networks:
  router:
    external: 
      name: router-network
```

Diesen Schnipsel können Sie als Referenz für weitere Projekte nutzen. Wichtig ist folgendes:

* Traefik wird über die Labels konfiguriert. Jeder Container bekommt einen `router`, danach folgt ein Name, der hier `docs` lautet.
* Für weitere Container müssen Sie einen anderen Router-Namen vergeben
* Der Router bekommt eine Regel, nach der Traefik den eingehenden Verkehr filtern soll. Hier ist die Regel auf die Subdomain gesetzt
* Andere Filter-Regeln erklärt die (recht gute) [Doku von Traefik](https://docs.traefik.io/routing/routers/) 





Jetzt sollte auch klar werden, warum wir das Kunststück mit dem Netzwerk eingebaut haben. Wenn Sie ein Compose-Projekt mit der gesamten Infrastruktur und Traefik haben, kann das dauerhaft laufen. Davon unabhängig können Sie andere Projekte in eigenen Compose-Files hoch- und runterfahren.

Fahren Sie Ihre Doku mit Docker-Compose hoch. Am Anfang wird sich der Browser beklagen – Traefik fängt sofort an, ein Zertifikat passend zur Host-Regel zu bestellen. Das dauert maximal fünf Minuten.
