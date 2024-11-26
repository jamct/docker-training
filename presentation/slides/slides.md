## Herzlich Willkommen

![ ]([https://images.ctfassets.net/3ujuzjed3id8/3QFe3Y7dkXiQfQkkb8dh0V/8c3057fad1b302c96e4e66791c064443/Docker-Academy-Shop-3072x768_hero.jpg)

26. und 27.11.2024

---

## Ihr Dozent

- Jan Mahn
- ab 2017 c't-Redakteur im Ressort Systeme und Sicherheit
- zuständig für Server-Systeme, Container, Software-Entwicklung
- seit 2024 stellv. Chefredakteur heise medien
- (begeisterter) Docker-Nutzer seit 2017

---

# "Es geht nicht um Docker"

Es geht um: <!-- .element: class="fragment" data-fragment-index="1" -->

- Flexibilität <!-- .element: class="fragment" data-fragment-index="2" -->
- Ersetzbare Server <!-- .element: class="fragment" data-fragment-index="3" -->
- Weniger Ärger mit Abhängigkeiten <!-- .element: class="fragment" data-fragment-index="4" -->

---

![ ](https://www.heise.de/select/ct/2021/24/2127409262474574019/ct2321dockerfur_albert_hulm_117229_jam_a_16zu9.jpg)

---

## Status Quo

"Server sind wie Haustiere. Sie werden gepflegt, für viel Geld am Leben erhalten. Stirbt ein Server, bricht für den Besitzer eine Welt zusammen."

---

## Typische Probleme

"Schlimmer als die Frage, warum etwas **nicht** funktioniert, ist ist Frage, warum es funktioniert."

- Abhängigkeiten in bestimmten Versionen festgenagelt <!-- .element: class="fragment" data-fragment-index="1" -->
- Mehrere (virtuelle) Maschinen für parallele alte Abhängigkeiten <!-- .element: class="fragment" data-fragment-index="2" -->

---

## Wunschzustand

- Die Anwendung läuft heute im eigenen RZ, morgen vielleicht in "der Cloud". <!-- .element: class="fragment" data-fragment-index="1" -->
- Es gibt eine 1:1-Kopie auf meiner Entwicklermaschine. <!-- .element: class="fragment" data-fragment-index="2" -->
- Ich sichere die Daten und das Rezept, um ein Produktivsystem zu erzeugen. <!-- .element: class="fragment" data-fragment-index="3" -->
- Wenn ich mehrere Kunden habe, die meine Anwendung nutzen, sollen auch die schnell eine identische Umgebung bekommen. <!-- .element: class="fragment" data-fragment-index="4" -->

---

## Cloud-Native !== Alles in der Cloud

"Cloud-Native heißt: Nutze die Techniken großer Anwendungen, die (auch) in der Cloud laufen. Wächst die Anwendung, ist sie von Anfang an skalierbar konzipiert."

---

## Pets vs. Cattle

"In der Container-Welt sind Container und ihre Hosts wie Nutzvieh. Sie sind ersetzbar! Stirbt einer, übernimmt der nächste. Kein Grund zur Trauer."

---

## Container !== Virtualisierung

![ ](./slides/header.jpg)

---

![ ](./slides/docker.png)

---

![ ](./slides/docker2.png)

---

## Wege zum Ziel

- Dienste in Container verpacken <!-- .element: class="fragment" data-fragment-index="1" -->
- Container-Abbilder automatisch bauen<!-- .element: class="fragment" data-fragment-index="2" -->
- Container-Host automatisch einrichten<!-- .element: class="fragment" data-fragment-index="3" -->
- Reproduzierbare Zusammenstellungen<!-- .element: class="fragment" data-fragment-index="4" -->

---

# Ihre Pläne mit Docker?

---

## Lab 1: Erste Schritte

Alle Workshop-Inhalte finden Sie unter [docs.liefer.it](https://docs.liefer.it)

![ ](https://heise.cloudimg.io/width/900/q65.png-lossy-65.webp-lossy-65.foil1/_www-heise-de_/select/ct/2016/5/1456733697045992/contentimages/image-145552165478819.jpg)

---

## Lab 2: Arbeiten mit Images

![ ](https://heise.cloudimg.io/width/900/q65.png-lossy-65.webp-lossy-65.foil1/_www-heise-de_/select/ct/2017/15/1500578738258740/contentimages/image-1499146982969054.jpg)

---

## Was gehört in ein Image

- die Anwendung <!-- .element: class="fragment" data-fragment-index="1" -->
- ein minimales Userland <!-- .element: class="fragment" data-fragment-index="2" -->
- ggf. Laufzeitumgebung / Interpreter <!-- .element: class="fragment" data-fragment-index="3" -->
- alle Abhängigkeiten <!-- .element: class="fragment" data-fragment-index="4" -->

---

## Herzlich Willkommen

![ ](https://www.heise-events.de/uploads/OnuAG8xJ/766x0_2560x0/Docker_2000x500.jpg)

Tag 2, 27.11.2024

---

## Zur Historie

- [2006 schlägt ein Google-Mitarbeiter die Funktion cgroups im Kernel vor](https://docs.kernel.org/admin-guide/cgroup-v1/cgroups.html)
- 2007 im Kernel
- 2013 erscheint Docker
- 2015 erscheint Kubernetes
- [2017 wird die OCI gegründet](https://opencontainers.org)

---

## Images bestehen aus Schichten

- jede Schicht verändert etwas am Dateisystem
- Beim Start des Containers setzt der Docker-Daemon alle Schichten zu einem System zusammen
- identische Schichten liegen nur einmal auf der Festplatte des Hosts

---

## Ein Blick in ein Image

```bash
mkdir nginx-inside
docker pull nginx:alpine
docker save nginx:alpine -o nginx.tar
tar -xf nginx.tar
```

---

## Ein Beispiel aus der Praxis

```dockerfile
FROM python:3-alpine
RUN apk add build-base
COPY ./mkdocs/ /mkdocs/
WORKDIR /mkdocs/
RUN pip install --upgrade pip && pip install mkdocs mkdocs-bootswatch
EXPOSE 8080
CMD ["mkdocs", "serve"]
```

---
