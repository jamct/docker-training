# Windows-Container unter Windows

![ ](https://heise.cloudimg.io/width/900/q65.png-lossy-65.webp-lossy-65.foil1/_www-heise-de_/select/ct/2018/5/1519616862422776/contentimages/image-1518182492265127.jpg)

Als Gastgeber für Windows Container brauchen Sie einen Windows Server (2016 oder 2019).

Installiert wird Docker per PowerShell:

```powershell
Install-Module -Name DockerMsftProvider -Repository PSGallery -Force
Install-Package -Name docker -ProviderName DockerMsftProvider
```

Anschließend neustarten:

```
Restart-Computer -Force
```

Die Befehle aus der  Linux-Welt funktionieren ebenfalls. Prüfen Sie die Funktion des Docker Daemons:

```
docker ps
```

## Ein Image finden

Unter Windows ist die Trennung zwischen Kernel und Userland nicht so sauber wie unter Linux. Deshalb muss man beim Wählen des Basis-Images genau aufpassen.

Grundsätzlich gibt es zwei (relevante) Basis-Images:

* [mcr.microsoft.com/windows/nanoserver](https://hub.docker.com/_/microsoft-windows-nanoserver) (~100 MByte)
* [mcr.microsoft.com/windows/servercore](https://hub.docker.com/_/microsoft-windows-servercore) (~2 GByte)

Entscheidend ist die Wahl des Tags. Der Tag muss zum Kernel des Gastgebers passen. Für den Server Core gibt es Tags, die an die 3-jährigen Zyklen des klassischen Windows Server angepasst sind:

* mcr.microsoft.com/windows/servercore:ltsc2019
* mcr.microsoft.com/windows/servercore:ltsc2016

Für den Nanoserver gibt es die nicht. Der Nanoserver wird im halbjährlichen Zyklus ausgeliefert:

* mcr.microsoft.com/windows/nanoserver:2004
* mcr.microsoft.com/windows/nanoserver:1903

Damit das läuft, brauchen Sie den halbjährlich aktualisierten Server. Die schlechte Nachricht: Dafür müssen Sie [Software Assurance bei Microsoft buchen](https://docs.microsoft.com/de-de/windows-server/get-started-19/servicing-channels-19). Es gibt aber einen Ausweg aus der misslichen Lage: Docker unter Windows kann Hyper-V zur Isolierung einsetzen:

```
docker run --isolation=hyperv mcr.microsoft.com/windows/nanoserver:1903
```

Die komplette Auflistung, bis wann welche Basis-Images unterstützt werden, [finden Sie bei Microsoft](https://docs.microsoft.com/en-us/virtualization/windowscontainers/deploy-containers/base-image-lifecycle).


### Nützliche Images aus der Windows-Welt

Bei Windows-Containern geht es meist um Projekte aus den Themenbereichen IIS und ASP.NET. Für solche Projekte müssen Sie nicht beim nackten Nanoserver beginnen. Stattdessen nutzen Sie Images, die Microsoft für diese Zwecke bereits vorbereitet hat:

* [IIS](https://hub.docker.com/_/microsoft-windows-servercore-iis)
* [ASP.NET (mit IIS)](https://hub.docker.com/_/microsoft-dotnet-framework-aspnet)

## Microsofts Sicht auf die Dinge

Eine Anekdote am Rande: In einem Gespräch mit Corey Sanders, Corporate Vice President für Azure bei Microsoft, ging es auch um die Bedeutung von Windows-Containern. Microsofts Meinung zum Thema:

"Windows-Container sind für Legacy-Anwendungen! Wenn Sie eine neue Anwendung planen, nehmen Sie Linux!"

Auch klassische Windows-Server-Software wie den *SQL Server* gibt es als Linux-Abbild, direkt von Microsoft:

* [mcr.microsoft.com/mssql/server](https://hub.docker.com/_/microsoft-mssql-server) (basiert auf Ubuntu)