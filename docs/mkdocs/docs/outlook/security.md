# Benutzer im Container

Die Arbeit mit Benutzerrechten im Container ist leider alles andere als intuitiv. Um das Konzept zu verstehen, muss man sich zunächst kurz mit Benutzern unter Linux befassen. Jeder Benutzer (gleiches gilt für Gruppen) hat eine ID, bei Benutzern heißt sie UID. Verwaltet werden die UIDs vom Kernel – und das verrät auch schon, wo die Besonderheit liegt: In Docker-Umgebungen gibt es nur einen Kernel, den alle Container ansprechen (Container sind keine VMs).

Zum Nachvollziehen nehmen Sie eine frische Linux-Maschine. Alle Benutzer listen Sie auf mit:

```
cat /etc/passwd
```

Das sieht dann zum Beispiel so aus (Liste gekürzt):

```
root:x:0:0:root:/root:/bin/bash
daemon:x:1:1:daemon:/usr/sbin:/usr/sbin/nologin
bin:x:2:2:bin:/bin:/usr/sbin/nologin
sys:x:3:3:sys:/dev:/usr/sbin/nologin
sync:x:4:65534:sync:/bin:/bin/sync
games:x:5:60:games:/usr/games:/usr/sbin/nologin
man:x:6:12:man:/var/cache/man:/usr/sbin/nologin
lp:x:7:7:lp:/var/spool/lpd:/usr/sbin/nologin
mail:x:8:8:mail:/var/mail:/usr/sbin/nologin
news:x:9:9:news:/var/spool/news:/usr/sbin/nologin
uucp:x:10:10:uucp:/var/spool/uucp:/usr/sbin/nologin
proxy:x:13:13:proxy:/bin:/usr/sbin/nologin
www-data:x:33:33:www-data:/var/www:/usr/sbin/nologin
backup:x:34:34:backup:/var/backups:/usr/sbin/nologin
[...]
```

Die meisten Informationen sind unwichtig, relevant sind die ersten Spalten (getrennt durch Doppelpunkte). Zunächst der Username, dann das Kennwort (das `x` verweist auf die `etc/shadow` und interessiert hier nicht). Dann folgt die ID des Benutzers und die ID der Gruppe, in der e sich befindet. Hier findet die Zuordnung von Name und ID statt. Im Hintergrund wird immer mit der ID gearbeitet, die Nummer ist für Menschen.

Einige Benutzer sind in vielen Distros von Anfang an dabei, zum Beispiel www-data für Webserver. Sie bekommen auch immer wieder dieselben IDs.

Genug vom Host, jetzt zum Container: Wenn man nichts unternimmt und im Dockerfile nichts weiter angegeben ist, startet ein Prozess immer mit dem User `root`. Das klingt für langjährige Linux-Nutzer gruselig, ist es aber gar nicht: Schließlich kann der Prozess nur in seinem begrenzten Bereich wüten. Maximal also in seinem flüchtigen Dateisystem und in den Volumes, die Sie ihm übergeben haben. Und genau an der Schnittstelle wird es spannend.

Sie können auf zwei Arten steuern unter welcher UID ein Prozess im Container läuft. Entweder beim Bau im Dockerfile mit `USER` oder beim Start mit dem Parameter `--user`. Letzteres kann man bequem per Docker-Compose übergeben. Zeit für ein kleines Experiment. Legen Sie folgende Docker-Compose-Datei an:

```
version: "3.8"
services:
  test:
    image: busybox
    command: sleep infinity
    user: "33"
```

Gestartet werden soll ein Container, der mit `sleep` am Leben gehalten wird, aber nichts weiter tut. Mit `user:` übergeben Sie den Benutzer, der im Container laufen soll. Statt der `33` könnten Sie hier auch den Namen `www-data` ausschreiben. Was dann passiert: Docker fragt beim Gastgeber nach, löst den Namen zur UID auf und übergibt die UID in den Container. Fahren Sie das ganze hoch und springen Sie in den Container:

```
docker compose up -d
docker compose exec test sh
```

Zeigen Sie sich dann an, wer den Prozess ausführt. Es ist `www-data`. Dass dieser Name hier angezeigt wird, ist aber purer Zufall – weil im Busybox-Container in `etc/passwd` ebenfalls dieser Name angelegt ist. Sie könnten im Container aber auch einen ganz anderen Namen für diese UID vergeben, das spielt keine Rolle. Wichtig ist: Die Syscalls aus dem Container-Prozess kommen mit der UID beim Kernel des Gasgebers an. Und nur darauf kommt es an. Das funktioniert übrigens auch, wenn es die UID im Container gar nicht gibt. Verlassen Sie den Container, fahren ihn mit `docker compose down` herunter und ändern das Dockerfile auf `user: 1005`. Wieder hochfahren, reintunneln und mit `ps` die UID anzeigen – angzeigt wird nur die Nummer, es gibt keine Namensauflösung, der Prozess funktioniert dennoch. Ein `whoami` im Container sagt ebenfalls, dass es den User hier nicht gibt. Das Fazit: Der Nutzer im Container ist nur was fürs Auge (vielleicht schön, wenn Sie Logging im Container eingebaut haben und da der richtige Name stehen soll), relevant für Volumes ist die ID des Gastgeber-Kernels!

## Konsequenz für die Praxis

Angenommen, Sie möchten den Nutzer `myapp-data` benutzen, der in ein Volume schreibt. Legen Sie ihn auf dem Gastgeber an (`useradd myapp-service -u 1099`). Geben ihm die Rechte auf seinen Datenordner. Wenn Sie möchten, können Sie ins Dockerfile ebenfalls diese Zeile als Befehl einbauen:

```
RUN useradd myapp-service -u 1099
[...]
USER 1099
```

Das ist aber beides optional, wenn Sie das später mit der Anweisung `user:` im Compose-File überschreiben. Bekommt der Container jetzt sein Volume, darf er darin das, was die `1099` auf dem Gastgeber darf.

### Randnotiz zu NFS

Mit der Angabe von `user: ` im Compose-File kann man auch das Problem mit NFS lösen. Um zu verhindern, dass die Macher eines Images in böswilliger Absicht eine beliebige UID ins Dockerfile schreiben und somit Rechte erschleichen, kann man das in letzter Instanz im Compose-File überschreiben. Natürlich muss der Betreiber des Docker-Hosts, der die Compose-Files schreibt und hochfährt, vertrauenswürdig sein.