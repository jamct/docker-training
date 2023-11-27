# :material-clock: Lab 7: Die Strategie für die Zukunft

Sie kennen jetzt das Konzept von Containern, wissen, wo brauchbare Images zu finden sind und wie man eigene erzeugen kann. Abschließend eine Checkliste für empfehlenswerte nächste Schritte:

## Für Entwickler

- Die eigene Anwendung in ein Dockerfile verpacken und lokal bauen
- Eine Registry auswählen und die Anwendung mit `docker push` dort ablegen
- Eine Automation auswählen (GitHub Actions, GitLab, Jenkins, ...) und Images automatisch bauen lassen
- Ggf. eine Strategie entwickeln, wo die Images des Unternehmens langfristig liegen sollen (eigene Registry betreiben oder Leistung mieten?)
- Eine Strategie für Tags entwickeln und mit CI/CD umsetzen. Beispiel: `:latest`-Tag bei Pushes in den Main-Branch, `:feature`-Tag bei Pushes in andere Branches
- Ein Installationspaket für Entwickler und Kunden mit Docker-Compose schnüren
- Das Image verfeinern: Schichten optimieren, Healthchecks erforschen, Monitoring ausprobieren

## Für Admins

- Passende Container für die zu betreibende Anwendung suchen. Guter Startpunkt für erste Erfahrung sind statische Webseiten
- Mit Docker eine Dev- oder Staging-Umgebung bauen. Dabei Erfahrungen und Probleme sammeln
- Docker-Compose-Rezepte (am besten in GitHub-Repos) für Alltagsprobleme bereitlegen
- Monitoring und Logging einrichten. Infrage kommen noch Werkzeuge wie [Fluentd](https://www.fluentd.org/guides/recipes/docker-logging).

# Für große Unternehmen

- Solide Erfahrungen mit Docker-Compose in Dev- und Staging-Umgebungen sammeln
- CI/CD- und Registry-Strategie festzurren
- Wenn man mit einzelnen Docker-Hosts an die Grenzen stößt: Einen Blick auf Kubernetes werfen.
