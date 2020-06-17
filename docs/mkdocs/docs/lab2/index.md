# :fa-flask: Lab 2: Arbeiten mit Images

Im ersten Teil haben Sie bereits zwei Images benutzt: `busyboy` und `nginx`. Aber wo kommen diese überhaupt her?

![ ](https://heise.cloudimg.io/width/900/q65.png-lossy-65.webp-lossy-65.foil1/_www-heise-de_/select/ct/2017/15/1500578738258740/contentimages/image-1499146982969054.jpg){: class="noprint"}

Der Reihe nach. Ursprung eines Images ist ein Rezept, das *Dockerfile*. Ein kleines Beispiel:

```dockerfile
FROM debian:buster-slim
RUN apt-get update && apt-get install -y iputils-ping
CMD ping -c 4 heise.de
```

In der ersten Zeile wird das Basis-Image definiert. Damit bekommt der Container das Dateisystem eines leeren Debian-Systems. Anschließend können beliebige Linux-Operationen ausgeführt werden, zum Beispiel Installationen per Paketmanager.

Am Ende des Dockerfile folgt das Schlüsselwort `CMD`. Es definiert den Prozess, der im Container läuft. Erinnern Sie sich an den Grundsatz:

**"Ein Container, ein Prozess"**

Solange dieser Prozess läuft, existiert der Container.


|Schlüsselwort|Funktion|Beispiel|
|---|---|---|
|`FROM`|definiert das Basis-Image|`FROM debian:buster-slim`|
|`RUN`|führt einen Befehl während des Bauprozesses aus|`RUN apt update && apt install ping`|
|`COPY`|kopiert Dateien und Ordner in den Container|`COPY settings.conf /etc/test/settings.conf`|
|`CMD`|definiert den Prozess, der im Container laufen soll|`CMD ping heise.de`|


## 1. Das erste eigene Image

Legen Sie eine Datei `Dockerfile` an und kopieren Sie den Inhalt des obenstehenden Beispiels in die Datei. Dann kann der Bau beginnen.

* Starten Sie den Bau mit:

```
docker build .
```

Der Punkt am Ende sagt: "Suche ein Dockerfile im aktuellen Verzeichnis". Sie können den Bauprozess live verfolgen. Am Ende erfahren Sie die ID, die sich der Docker-Daemon ausgedacht hat:

```
Successfully built 72a2c8964f90
```

Kopieren Sie Ihre ID heraus und starten Sie den Container:

```
docker run <id>
```

Der Container hat eine kurze Lebenszeit. Nach vier Pings stoppt der Prozess und Docker stoppt den Container.

### 1.2 Images verwalten

Alle Images auf dem System sehen Sie mit

```
docker image ls
```

Dort finden Sie auch Ihr frisch gebautes Image. Sie können ihm einen sprechenden Namen geben:

```
docker tag <id> simpleping:latest
```

Danach können Sie das Image unter seinem Namen ansprechen:

```
docker run simpleping
```

Das Taggen können Sie mit dem Parameter `-t` auch direkt mit dem Bauprozess verbinden:

```
docker build -t simpleping .
```

Zum Löschen eines Images nutzen Sie:

```
docker image rm <id oder name>
```

Der Docker-Daemon wird sich beschweren, solange noch ein Container auf Basis des Images existiert (auch gestoppte Container zählen dazu). Nutzen Sie [Ihr Wissen aus Lab 1](../lab1/#4-zusammenfassung), um die gestoppten Container zu identifizieren und alle Images zu entsorgen.

!!! note "Images aufräumen"
    Ungenutzte Images kosten Platz auf der Platte. Sie müssen sie nicht per Hand einzeln löschen. Für diesen Zweck gibt es:

    ```
    docker image prune
    ```

Damit ein gestoppter Container nicht herumliegt und sofort abgeräumt wird, gibt es den Parameter `--rm`:


```
docker run --rm simpleping
```


## 2. Gute Images

!!! note "Gute Images erkennen"

    1. Überprüfen Sie, ob es ein offizielles Image für die Software gibt (ohne / im Namen).

    2. Wählen Sie einen geeigneten Tag. Abgespeckte Images sind kleiner und bieten weniger Angriffsfläche. Alpine ist oft eine gute Alternative.

    3. Gibt es kein offizielles Image, schauen Sie sich direkt beim Entwickler der Software um. Einige bieten eigene Docker-Images an.

    4. Gibt es keine Images der Entwickler, suchen Sie im Hub oder in der Suchmaschine.

    5. Abrufzahlen, Dokumentation und Sterne im Docker Hub geben einen Hinweis, ob das Projekt aktiv benutzt wird.

    6. Kennen Sie den Ersteller nicht, bevorzugen Sie Images mit automatischen Builds und schauen Sie ins Dockerfile.