# Docker und Container in der Praxis

Dieses Repository ist Teil eines Online-Workshops von c't für [Heise Events](https://heise-events.de).


## Materialien im Browser anzeigen

Um Präsentation und Dokumentation auf einer Entwicklungsmaschine zu starten:

* installieren Sie Docker
* installieren Sie Docker-Compose
* fahren Sie die Zusammenstellung hoch: `docker-compose up -f docker-compose-dev.yml -d`


## Anregungen in diesem Repo

Dieses Repository darf man auch gern als Anregung und Inspiration genommen werden. Einige Dateien und Ideen exemplarisch:

* GitHub Action zum automatischen Bau: ./github/workflows/docker-docs.yml
* Multi-Stage-Image bauen: ./docs/Dockerfile-prod
* Eine containerbasierte Entwicklungsumgebung: docker-compose-dev.yml