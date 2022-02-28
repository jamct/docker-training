# :fa-wrench: Docker einrichten

Die Software Docker hat das Unternehmen Docker Inc. ursprünglich in zwei Varianten angeboten: *Docker Enterprise* und *Docker Community*. Nachdem die Enterprise-Sparte an Mirantis verkauft wurde, sind nur noch die Open-Source-Variante *Docker Community* und *Docker Desktop* interessant.

## Für Windows und macOS

Es gibt die Docker Desktop mit grafischer Oberfläche für Windows und macOS. Wählen Sie am besten die Stable-Variante, Edge enthält Beta-Funktionen:

* [Download bei Docker Inc.](https://www.docker.com/products/docker-desktop)

![ ](macos.png){: style="width:50%"}

!!! note "Bezahlen für Docker Desktop"
    Docker Desktop (mit der GUI) ist für kleine Unternehmen kostenlos. Ab 250 Mitarbeitern oder mehr als 10 Millionen US-Dollar Umsatz muss man monatlich pro Benutzer zahlen. Preise: [https://www.docker.com/pricing](https://www.docker.com/pricing)


## Für Linux und Linux-Server

Linux-Desktop-Nutzer müssen ohne grafische Oberfläche auskommen (was nicht schwerfällt). Auf dem Server gibt es ohnehin keinen Grund für grafische Oberflächen.

Die Paketquellen der gängigen Linux-Distributionen sind keine geeignete Anlaufstelle. Die Container-Welt ist zu schnelllebig.

Es gibt Paketquellen (.deb und .rpm) [direkt von Docker Inc.](https://docs.docker.com/engine/install/). Der schnellste Weg zur Docker-Engine auf dem Server ist das Installations-Skript:

```bash
curl -fsSL https://get.docker.com -o get-docker.sh
sudo sh get-docker.sh
```

Das Skript lädt Docker herunter und installiert es. Anschließend sollten Sie den aktuellen Benutzer zur Gruppe `docker` hinzufügen, damit Sie auch als normaler Benutzer auf den Docker-Daemon und die laufenden Container einwirken können:

```
sudo usermod -aG docker $USER
```

### Docker-Compose

Unter Windows und macOS müssen Sie Docker-Compose nicht separat installieren, es ist Teil der Desktop-Umgebung.

Unter Linux war Docker-Compose früher ein separates Programm, das in Python geschrieben ist.

Neuerdings gibt es Compose als Erweiterung der Docker-CLI. Installiert wird diese mit folgenden drei Zeilen

```
mkdir -p ~/.docker/cli-plugins/
curl -SL https://github.com/docker/compose/releases/download/v2.2.3/docker-compose-linux-x86_64 -o ~/.docker/cli-plugins/docker-compose
chmod +x ~/.docker/cli-plugins/docker-compose
```

Wenn Sie irgendwo Anleitungen mit `docker-compose` finden, lassen Sie den Bindestrich weg und schreiben: `docker compose`
