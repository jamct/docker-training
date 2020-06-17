# Ausblick und Abschluss

Zunächst ein Tipp zum Schluss: Um wirklich ins Thema einzusteigen, müssen Sie mit Docker arbeiten – viele Grundlagen waren Inhalt des Workshops, eigene Erfahrungen kann das nicht ersetzen. Am besten steigen Sie mit einem nicht-kritischen Teil ein. Ideen für Beispiel-Projekte:

* Eine Doku (Anregungen in diesem Repo)
* Eine Status-Seite für firmeninterne Dienste

## Offene Fragen

Hier noch ein Nachtrag zu offenen Fragen nach der Veranstaltung:

### Frage:

Wie kann ich das Dateisystem für den Container begrenzen, damit ein "verrückt gewordener" Container nicht den Host und damit alle anderen Container beeinflusst?

### Antwort:

Viele Images arbeiten mit dem Nutzer "root". Die Kapselung von Docker ist aber relativ robust (nicht zu 100%, aber zu 99%). Der Prozess im Container sieht das Dateisystem, das der Docker Daemon ihm vorsetzt. Also die Summe aller Schichten und die Volumes, die man ihm reinreicht. Aus diesen Volumes kommt ein Prozess nicht raus.

Wenn Sie eigene Images bauen, können Sie im Dockerfile einen Benutzer anlegen und dann Docker dazu bringen, diesen zu nutzen. Beispiel:

```
FROM ubuntu:latest
RUN groupadd -r postgres && useradd -u 1005 --no-log-init -r -g postgres postgres
USER postgres
```

Der Nutzer hat dann die UID 1005. Wenn Sie denselben Nutzer mit derselben UID auf dem Gastgeber anlegen, erleichtern Sie sich ggf. das Leben, wenn Sie zum Beispiel einen Job erzeugen, der die Daten sichern soll. 


### Frage:

Wie bekomme ich extern gebaute Artefakte (z.B. Maven) in einen automatischen Bau-Prozess?

### Antwort:

Wenn Sie GitHub Actions benutzen möchten, müssen Sie Ihre Artefakte so bereitstellen, dass man sie über das Internet beziehen kann. Im Dockerfile kann man das dann zum Beispiel vn externer Stelle laden:

```
RUN wget https://internal-artifacts.example.org/app.tar.gz \
&& tar zxvpf app.tar.gz && rm app.tar.gz
```


### Frage:

Am ersten Tag wurde erwähnt, dass ein Image auch gar kein Base-Image haben kann. Wie mache ich das konkret, wenn ich wirklich nur eine ausführbare Datei habe?

### Antwort:

Als Beispiel nehmen wir mal Go und bauen ein Multistage-Image (schematische Darstellung, ungetestet!):

```
FROM golang
COPY ./code .
RUN GOOS=linux go build -o app .

FROM scratch
COPY --from=0 app .
CMD ["./app"]  
```

In der ersten Stufe wird der Code kopiert, der Go-Compiler baut das Executable `app`. In der zweiten Stufe wird ein Image `FROM scratch` erzeugt. Das enthält absolut nichts. Darein kopiert Docker dann `app` aus Stufe 1 und führt es aus. Das Image ist so groß wie das Executable. Klappt nur mit Anwendungen, die komplett bedürfnislos sind.

### Frage:

Kann Traefik auch HTTP/2 ausliefern? Wie aktiviere ich das?

### Antwort:

Ja, da gibt es nichts zu aktivieren. Wenn die Seite per TLS ausgeliefert wird, wird auch HTTP/2 unterstützt!

## Weitere Software


Im Workshop wurde an einigen auf Themen und Konzepte verwiesen. Hier noch einmal eine kleine Übersicht und Links:

### 1. Kubernetes

Kubernetes (K8S) ist ein Container-Orchestrator für große Umgebungen. Aber auch in mittleren Umgebungen kann es uns das Leben erleichtern. Kubernetes löst folgende Probleme:

* Cluster aus mehreren Servern
* API mit ausgefeiltem Berechtigungsmodell
* ein riesiges Umfeld mit Software für viele spezielle Probleme
* Managed-Kubernetes als Produkt von Cloud-Anbietern (keine Verwaltung des Gastgebers).
* Rolling Updates (Austausch von Containern ohne Ausfall)
* definierte Reihenfolgen von Starts und Neustarts

Wichtig vorm Einstieg in K8S: Solides Docker-Wissen und – noch wichtiger – Erfahrung. Zu Erfahrungen gehört auch das Scheitern.

### 2. Eigene Registry

Zwei Registries haben wir behandelt: Den Docker-Hub und die GitHub-Registry. Wenn Sie eine eigene Container-Registry brauchen, können folgende Produkte interessant sein:

* [Harbor](https://goharbor.io)
* [GitLab](https://docs.gitlab.com/ee/user/packages/container_registry/)

