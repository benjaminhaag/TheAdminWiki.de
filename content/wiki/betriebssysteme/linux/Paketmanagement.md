---
date: '2026-04-24T13:04:42Z'
draft: false
title: 'Paketmanagement'
image: "/images/linux_paketmanagement.webp"
tags: ["betriebssysteme","linux","Paketmanagement"]
---

Um software zu installieren müssen bestimmte **Dateien**, **Programme** und **Abhängigkeiten** an bestimmte Stellen im System abgelegt werden. Zudem müssen oft noch weitere Schritte vorgenommen werden wie **Einstellungen** ändern, **Benutzer** anlegen und vieles mehr.

Damit der Benutzer nicht alles selbst machen muss gibt es grundsätzlich zwei Konzepte. **Installer** sind kleine Programme die einmal ausgeführt werden und sich um alles kümmern. Das war Jahrelang der Hauptweg um unter Windows Software zu installieren, kommt aber mit erheblichen Nachteilen. Zum einen muss der Nutzer die Software von fremden, potentiell virenbefallenen, Quellen Programme herunterladen, und zum anderen gibt es keine zentrale Stelle die weiß welche Software installiert ist.

Linux ist das Problem schon immer anders angegangen und Nutzt sogenannte **Pakete**. Pakete sind Archive mit sämtlichen notwendigen Dateien, Programmen, maschinenlesbaren (De-)Installationsanleitungen und Metadaten die automatisches Updaten, Dependency Management, und vieles mehr ermöglichen. Pakete lösen an sich zwar nicht das Problem mit fremden Quellen, bieten aber eine Zentrale Stelle im System die weiß, welche Pakete installiert wurden. Zudem ermöglichen Pakete eine Einheitliche Art und Weise mit Software umzugehen und sogenannte **Repositories** bereitzustellen. Repositories sind Vertrauenswürdige Quellen mit getesteter und gescannter Software. Das macht die Installation und das Management wesentlich einfacher.

## Pakete

Pakete sind archive mit Dateien, Installationsanleitungen und Metadaten. 

### Debian
Wer selbst ein Paket bauen möchte, oder eins anschauen will, kann beliebige Pakete entpacken

```
dpkg-deb -R <FILE> <TARGET>
```

und wieder verpacken

```
dpkg-deb --build 
```

```
.
|-- DEBIAN
|   |-- control
|   |-- postinst
|   |-- postrm
|   |-- preinst
|   `-- prerm
`-- etc
    `-- myapp
        `-- test.txt
```

Der Aufbau von Paketen ist immer ähnlich und Besteht aus einem `DEBIAN` ordner und weiteren Dateien. Die weiteren Dateien werden von dpkg so ins system gelegt wie sie im Paket sind. Wer also eine Datei namens `test.txt` in `/etc/myapp` ablegen will, muss die datei in `./etc/myapp/test.txt` anlegen. Zusätzlich kann das packet die Skripte `preinst`, `postinst`, `prerm` und `postrm` enthalten. Sie werden jeweils vor (pre) und nach (post) der Installation (inst), beziehungsweise Deinstallation (rm) ausgeführt. Also bevor die Dateien an die entsprechenden stellen gelegt, beziehungsweise entfernt werden. Damit das klappt, müssen die Skripte ausführungsberechtigungen gesetzt haben.

Die control datei speichert die Metadaten des Packetes

```
Package: myapp
Version: 1.0
Section: utils
Priority: optional
Architecture: all
Depends: python (>= 3.10), curl | wget
Maintainer: John Doe <john.doe@theadminwiki.de>
Description: My little test application
```

**Section** ist frei wählbar, sollte sich aber an gängigen Sektionen der Distribution orientieren wie zum Beispiel:

| Sektion   | Beschreibung |
| - | - |
| admin     | Administrationstools
| utils     | Werkzeuge |
| base      | Grundlegende Distributionspakete |
| devel     | Entwicklerwerkzeuge |
| libs      | Bibliotheken |
| net       | Netzwerktools |
| mail      | Emailtools |
| web       | Webserver und Tools |
| gnome     | Gnome DE relevante Pakete |
| kde       | KDE relevante Pakete |
| sound     | Soundtools |
| video     | Grafiktools |
| games     | Spiele |
| doc       | Dokumentationen und Hilfen |
| misc      | Unspezifische Pakete |

**Priorität** gibt die wichtigkeit des Pakets für ein System an. Sie kann `required`, `important`, `standard` oder `optional` sein, ist aber in den aller meisten Fällen `optional`

**Architekturue** muss ein bekanntes Alias für die Architektur sein die unterstützt wird. Architekturunabhängige Pakete wie reine Dateien oder Python Skripte können `all` nutzen. Architekturabhängige Pakete sollten aus `amd64`, `i386`, `arm64` oder `armhf` wählen. Im zweifel kennt das System die architektur selbst `dpkg --print-architecture`. Das wird auch genutzt um die Kompatibilität des Paketes zu ermitteln.

**Depends** listed alle Abhängigkeiten auf. Es können auch alternative Abhängigkeiten angegeben werden (`curl | wget`) und konkrete Versionen
| Angabe | Bedeutung |
| - | - |
| =      | Exakte Version nötig |
| >=    | Mindestens die angegebene Version |
| <=    | Maximal die angegebene Version |
| >>    | Strikt größer |
| <<    | Strikt kleiner |

## Einfache Paketmanager

Einfache (low-level) Paketmanager kennen nur installierte Pakete und das, was gerade installiert werden soll. Sie Installieren, Deinstallieren und Konfigurieren Pakete. Um ein Paket zu installieren muss es erst heruntergeladen werden. Dabei stellen Paketmanager sicher dass bei der Installation alle notwendigen Abhängigkeiten installiert sind und bei der Deinstallation kein noch installiertes Paket vom deinstallierten Paket abhängt. Anderenfalls wirft der Paketmanager einen Fehler und listet die betroffenen Pakete auf.

> [!WARNING]
> Jede Linux Distribution hat eine andere Vorstellung davon wo, was, wie gemacht wird. RPM Pakete können nicht mit DPKG verwaltet werdet. DPKG Pakete für Ubuntu müssen auch nicht auf Debian mit DPKG Funktionieren.
> Wenn möglich sollte ein Paket immer aus dem Korrekten Repository verwendet werden.

> [!TIP]
> Mit `-f` bei dpkg, bzw `--nodeps` bei rpm lassen sich Pakete installieren und entfernen ohne den Dependency Check. Aber nur weil es geht muss man es nicht machen.

### DPKG

**Debian Package (DPKG)** ist ein low-level Package Manager der von Debian entwickelt wird und auch auf nahezu jedem Derivat (wie Ubuntu) zum Einsatz kommt.

| Beschreibung                                  | dpkg                      |
| --------------------------------------------- | ------------------------- |
| Paket installieren                           | `dpkg -i <FILE>`          |
| Paket deinstallieren                         | `dpkg -r <PACKAGE>`       |
| Paket inklusive aller Dateien entfernen      | `dpkg -P <PACKAGE>`       |
| Paketinformationen anzeigen                  | `dpkg -I <PACKAGE>`       |
| Paketinformationen anzeigen                  | `dpkg -I <FILE>`          |
| Installierte Dateien auflisten                | `dpkg -L <PACKAGE>`       |
| Installierte Dateien auflisten                | `dpkg -L <FILENAME>`       |
| Installierte Pakete auflisten                 | `dpkg --get-selections`    |
| Paket zu einer installierten Datei finden    | `dpkg-query -s <FILE>`    |
| Paketeinstellungen Zurücksetzen               | `dpkg-reconfigure <PACKAGE>`  |

### RPM

**RedHat Package Manager (RPM)** ist ein low-level Package Manager von RedHat der auch auf nahezu jedem Derivat (wie Fedora) zum Einsatz kommt. Zudem wird RPM von SUSE verwendet, obwohl es kein RedHat Derivat ist.

| Beschreibung                                  | rpm                   |
| --------------------------------------------- | --------------------- |
| Paket installieren                           | `rpm -i <FILE>`       |
| Paket deinstallieren                         | `rpm -e <PACKAGE>`    |
| Paket inklusive aller Dateien entfernen      | *nicht möglich*       |
| Paketinformationen anzeigen                  | `rpm -qi <PACKAGE>`   |
| Paketinformationen anzeigen                  | `rpm -qip <FILE>`     |
| Installierte Dateien auflisten                | `rpm -ql <PACKAGE>`   |
| Installierte Dateien auflisten                | `rpm -qlp <FILE>`     |
| Installierte Pakete auflisten                 | `rpm -qa`             |
| Paket zu einer installierten Datei finden    | `rpm qf <FILE>`       |
| Paketeinstellungen Zurücksetzen               | *nicht möglich*   |

> [!TIP]
> RPM kommt standardmäßig nicht mit einer Fortschrittsanzeige und wenig output. Die Flags `-h` zum zeigen einer Fortschrittsanzeige und `-v` für verbosen Output sind daher fast immer sinnvoll.

## Repository Basierte Paketmanager

Einfache Paketmanager kennen nur installierte Pakete und das, was gerade installiert werden soll. Sie können also prüfen ob alle Abhängigkeiten installiert sind, aber keine weiteren Pakete aus dem Nichts installieren. Repository basierte (high-level) Paketmanager erfinden das Rad nicht neu und bauen deshalb meist auf bereits bestehende einfachere Paketmanager auf.
Um Pakete zu verwalten reden sie mit den daruntergelegenen Paketmanagern, haben aber vollen Zugriff auf Repositories um nach fehlenden Abhängigkeiten zu suchen und diese einfach mit zu installieren. Die Repositories sind nich fest definiert. Stattdessen bieten Repository basierte Paketmanager eine Stelle an, an der Paketquellen hinterlegt sind.

Um nicht jedes mal das komplette Repositoryverzeichnis zu durchsuchen wird einmal ein Manifest heruntergeladen. Quasi eine Liste an verfügbaren Paketen, versionen und weiteren Metainformationen. Diese Liste kann sehr schnell veralten. Zum Beispiel, wenn ein neues Paket hinzugefügt wird, Software aktualisiert wird, oder eine Sicherheitslücke gepatched wird. Um sicherzugehen dass der Paketmanager immer über die aktuellsten Informationen verfügt *kann* es notwendig sein die **Metadaten zu aktualisienen**. 

Ein weiterer wesentlicher Unterschied zu einfachen Paketmanagern ist, dass sich Repository basierte Paketmanager selbst um Abhängigkeiten kümmern. In seltenen fällen, vor allem in verbindung mit einigen Befehlen die `--force` oder `-f` beinhalten, aber auch bei Systemabstürzen oder anderen Problemen, kann es zu inkonsistenzen kommen. Abhängigkeiten fehlen oder sind in der falschen Version vorhanden. In dem fall ist es oft hilfreich die **Systemkonsistenz wiederherzustellen**.

> [!TIP]
> Dass nicht jedes mal die Quellen automatisch aktualisiert werden hat praktische gründe. Repositories sind riesige Verzeichnisse mit massen an Metainformationen. Jedes mal alles zu aktualisieren dauert lang und kostet viele Resourcen. Selbst wenn man es einstellen kann ist es nicht immer sinnvoll aus Bequemlichkeit jedes mal die Metadaten neu zu ziehen.

### APT
**APT** und **apt-get** basieren auf **DPKG** Beide sind sehr beliebt in Debian basierten Systemen. apt-get ist älter und verändert sich heute kaum noch. Daher wird es primär in skripten verwendet, da man dort ein stabiles Interface braucht. apt ist ein modernen Makeover von apt-get`, vereint mehrere Tools die um apt-get herum gebaut wurden und bietet ein angenehmeres Interface. Da sich apt aber keinem stabilen Interface verschrieben hat wird abgeraten es in Skripten zu verwenden.

| Beschreibung                                  | apt-get                                       | apt                         |
| --------------------------------------------- | --------------------------------------------- | --------------------------- |
| Metadaten Aktualisieren                       | `apt-get update`                              | `apt update`                |
| Paket Installieren                           | `apt-get install <...PACKAGES>`               | `apt install <...PACKAGES>` |
| Paket Deinstallieren                         | `apt-get remove <...PACKAGES>`                | `apt remove <...PACKAGES>`  |
| Paket mit Dateien entfernen                  | `apt-get purge <...PACKAGES>`                 | `apt purge <...PACKAGES>`   |
| Paket suchen                                 | `apt-cache search <PATTERN>`                  | `apt search <PATTERN>`      |
| Details zum Paket anzeigen                   | `apt-cache show <Paket>`                     | `apt show <PACKAGE>`        |
| Dateien in einem Paket anzeigen              | *nicht möglich*                               | *nicht möglich*             |
| Paket zu einer Datei suchen                  | `apt-file update && apt-file search <FILE>`   | *mit apt-file möglich*      |
| Aktualisierungen aufzeigen                    | `apt-get -s upgrade`                          | `apt list --upgradable`     |
| Alle Pakete aktualisieren                     | `apt-get upgrade`                             | `apt upgrade`               |
| Aktualisierungen zu einem Paket aufzeigen    | *nicht direkt möglich*                        | *nicht direkt möglich*      |
| Paket aktualisieren                          | `apt-get install <...PACKAGES>`               | `apt install <...PACKAGES>` |
| Installierte Pakete auflisten                 | *nicht möglich*                               | `apt list --installed`      |
| Systemkonsistenz herstellen                   | `apt-get install --fix-broken`                | `apt install --fix-broken`  |
| Cache Leeren                                  | `apt-get clean`                               | `apt clean`                 |

Cache Dateien liegen unter `/var/cache/apt/archives`.

#### Repository Management

Quellen für apt und apt-get wurden in `/etc/apt/sources.list` definiert, allerdings werden heute zur besseren Übersicht fast nurnoch `.list` Dateien in `/etc/apt/sources.list.d/` genutzt. Beide Paketmanager respektieren aber beide Wege. Um Quellen zu definieren gibt es zwei formate innerhalb dieser Dateien. APT kommt mit beiden klar, allerdings präferieren manche Distributionen das ein oder andere. Beide formate definieren den **Typ** der Pakete, meist *deb*, **URIs** zum Repository, meist *ftp* oder *http*, **Suites** und **Components** zum logischen Trennen von Paketen und optional einen **GPG Key** der die Signatur der Pakete prüfen kann.

> [!TIP]
> Repositories sind oft unverschlüsselt (HTTP statt HTTPS, FTP statt FTPs). Stattdessen stellt GPG die sicherheit bereit. Trotzdem schadet es nicht beide Mechanismen zu nutzen.

```
 deb  http://de.archive.ubuntu.com/ubuntu noble main restricted universe multiverse
^^^^^ ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^ ^^^^^ ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
TYPE                URI                   SUITE             COMPONENTS

 deb [signed-by=/usr/share/keyring/ubuntu-archive-keyring.gpg] http://de.archive.ubuntu.com/ubuntu noble main restricted universe multiverse
     ^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^^
                        GPG KEY
```

Mit der klassischen Syntax kann man nur eine Suite angeben. Will man aus dem selben Repository mehrere Suites nutzen, braucht es mehrere Zeilen. Zudem ist die Syntax eher unübersichtlich. Die *Deb822* Syntax löst das Problem.

```
Types: deb
URIs: http://de.archive.ubuntu.com/ubuntu
Suites: noble noble-updates noble-backports
Components: main restricted universe multiverse
Signed-By: /usr/share/keyring/ubuntu-archive-keyring.gpg
```

Suites und Components können von den Repositories frei gewählt werden, Debian und Ubuntu haben aber Standards, die von den meisten Repositories respektiert werden.

Als Suite nutzen Ubuntu und Debian den Codename der Version (z.B. noble) und zusätzlich ein suffix zum trennen von Updatezyklen.
| Suite                  | Beschreibung                                                          |
| ---------------------- | --------------------------------------------------------------------- |
| **noble**              | beinhaltet die Pakete wie sie zum Erscheinen der Distribution waren  |
| **noble-updates**      | beinhaltet stabile Versionen die sehr gut getestet sind               |
| **noble-backports**    | beinhaltet aktuellere Versionen, die noch getestet werden             |
| **noble-security**     | beinhaltet sicherheitsrelevante Patches.                              |

Als Komponenten Nutzt Ubuntu folgendes Schema

| Komponente        | Beschreibung                                                                  |
| ----------------- | ----------------------------------------------------------------------------- |
| **main**          | Offiziell unterstützte Pakete die Ubuntu selbst baut oder anpasst.           |
| **restricted**    | Offiziell unterstützte Pakete mit nicht Quelloffener Software (wie Treiber)  |
| **universe**      | Unterstützte Pakete die Ubuntu zwar testet, aber nicht anpasst               |
| **multiverse**    | Unterstützte Pakete die nicht angepasst sind und nicht Quelloffene Software beinhalten.  |

Debian unterscheidet nicht zwischen den Maintainern, aber granularer zwischen der Quelloffenheit.

| Komponente    | Beschreibung                                                      |
| ------------- | ----------------------------------------------------------------- |
| **main**      | Pakete mit Freier und Open Source Software                       |
| **contrib**   | Pakete mit Freier Software, aber Closed Source Abhängigkeiten    |
| **non-free**  | Closed Source Software                                            |


### YUM, DNF
**YUM** und **DNF** basieren auf **RPM**. **YellowDog Updater (YUP)** wurde damals für die (heute nicht mehr existente) **YellowDog Linux** Distribution geschrieben. Da YellowDog Linux ein RedHat Derivat war und YUP an sich gute Software, modifizierte RedHat YUP um es auf alle RedHat Derivate portieren zu können und veröffentlichte den Paketmanager als **YellowDog Updater, Modified (YUM)**. Beide Manager wurden in Python2 geschrieben, was heute veraltet ist. Als ersatz wurde YUM in Python3 neu geschrieben und als **Dandified YUM (DNF)** von RedHat veröffentlicht. Der Befehl `yum` ist zwar heute noch auf einigen RedHat Derivaten, ist aber ein wrapper um DNF und nutzt daher die DNF-Syntax.

| Beschreibung                                  | yum                          | dnf                          |
| --------------------------------------------- | ---------------------------- | ---------------------------- |
| Metadaten Aktualisieren                       | *nicht notwendig*            | *nicht notwendig*            |
| Paket Installieren                           | `yum install <...PACKAGES>`  | `dnf install <...PACKAGES>`  |
| Paket Deinstallieren                         | `yum remove <...PACKAGES>`   | `dnf remove <...PACKAGES>`   |
| Paket mit Dateien entfernen                  | *manuell nötig*              | *manuell nötig*              |
| Paket suchen                                 | `yum search <PATTERN>`       | `dnf search <PATTERN>`       |
| Details zum Paket anzeigen                   | `yum info <PACKAGE>`         | `dnf info <PACKAGE>`         |
| Dateien in einem Paket anzeigen              | `yum repoquery -l <PACKAGE>` | `dnf repoquery -l <PACKAGE>` |
| Paket zu einer Datei suchen                  | `yum whatprovides <FILE>`    | `dnf provides <FILE>`        |
| Aktualisierungen aufzeigen                    | `yum check-update`           | `dnf check-update`           |
| Alle Pakete aktualisieren                    | `yum update`                 | `dnf upgrade`                |
| Aktualisierungen zu einem Paket aufzeigen    | `yum check-update <PACKAGE>` | `dnf check-update <PACKAGE>` |
| Paket aktualisieren                          | `yum update <...PACKAGES>`   | `dnf upgrade <...PACKAGES>`  |
| Installierte Pakete auflisten                | `yum list installed`         | `dnf list --installed`       |
| Systemkonsistenz herstellen                   | `yum-complete-transaction`   | `dnf distro-sync`            |
| Cache Leeren                                  | `yum clean all`              | `dnf clean all`                 |

Cache Dateien liegen unter `/var/cache/yum`, beziehungsweise in `/var/cache/dnf`.

> [!TIP]
> Caches für Metadaten und Pakete können getrennt geleert werdet mit `metadata` bzw. `packages` statt `all`

#### Repository Management

Quellen für YUM und DNF befinden sich in `.repo` Dateien in `/etc/yum.repos.d/`. Allerdings werden repositories durch den config-manager verwaltet. Die Nutzung wurde in DNF Version 5 jedoch deutlich verändert.

| Beschreibung                                  | yum                                   | dnf 4                                 | dnf 5                                             |
| --------------------------------------------- | ------------------------------------- | ------------------------------------- | ------------------------------------------------- |
| Repositories auflisten                        | `yum repolist all`                      | `dnf repolist --all`                    | `dnf repolist --all`                                |
| Repository hinzufügen                         | `yum-config-manager --add-repo <URL>`   | `dnf config-manager --add-repo <URL>`   | `dnf config-manager addrepo --frem-repofile=<URL>`  |
| Repository aktivieren                         | `yum-config-manager --enable <REPO>`    | `dnf config-manager --set-enabled <REPO>`   | `dnf config-manager setopt <REPO>.enabled=1`    |
| Repository deaktivieren                       | `yum-config-manager --disable <REPO>`    | `dnf config-manager --set-disabled <REPO>`   | `dnf config-manager setopt <REPO>.enabled=0`  |

> [!TIP]
> Zum löschen kann die `.repo` datei manuell gelöscht werden. Alternativ wird das Repository nur disabled.

### Zypper
**Zypper** wurde von **SUSE** entwickelt. SUSE Systeme nutzen zwar ebenfalls **RPM** Pakete, die Repositories sind aber vollkommen anders aufgebaut und fordern daher einen eigenen Manager.

| Beschreibung                                  | zypper                          |
| --------------------------------------------- | ------------------------------- |
| Metadaten Aktualisieren                       | `zypper refresh`                |
| Paket Installieren                           | `zypper install <...PACKAGES>`  |
| Paket Deinstallieren                         | `zypper remove <...PACKAGES>`   |
| Paket mit Dateien entfernen                  | *manuell nötig*                 |
| Paket suchen                                 | `zypper search <PATTERN>`       |
| Details zum Paket anzeigen                   | `zypper info <PACKAGE>`         |
| Dateien in einem Paket anzeigen              | `zypper what-provides <PACKAGE>`| 
| Paket zu einer Datei suchen                  | `zypper what-provides <FILE>`   |                          
| Aktualisierungen aufzeigen                    | `zypper list-updates`           |
| Alle Pakete aktualisieren                     | `zypper update`                 |
| Aktualisierungen zu einem Paket aufzeigen    | `zypper list-updates <PACKAGE>` |
| Paket aktualisieren                          | `zypper update <...PACKAGES>`   |
| Installierte Pakete auflisten                 | `zypper search -i`              |
| Systemkonsistenz herstellen                   | `zypper install --force-resolution --force` |
| Cache Leeren                                  | `zypper clean --all`                        |

Cache Dateien liegen unter `/var/cache/zypp`.


> [!TIP]
> Caches für Metadaten und Pakete können getrennt geleert werdet mit `--metagata` bzw. `--packages` statt `--all`

#### Repository Management

Quellen für YUM und DNF befinden sich in `.repo` Dateien in `/etc/zypp/repos.d/`. Allerdings werden repositories durch zypper direkt verwaltet. Zudem kann pro repository eingestellt werden ob beim Nutzen des Repositories automatisch die neuen Metainformatiionen geholt werden sollen. Damit sollte man allerdings vorsichtig sein, da dass den gesamten Prozess deutlich verlangsamen und zu erhöhtem Resourcenverbrauch führen kann.

| Beschreibung                                  | zypper                                |
| --------------------------------------------- | ------------------------------------- |
| Repositories auflisten                        | `zypper repos`                          |
| Repository hinzufügen                         | `zypper addrepo <URL> <NAME>`           |
| Repository löschen                            | `zypper removerepo <NAME>`              |
| Repository aktivieren                         | `zypper modifyrepo -e <NAME>`           |
| Repository deaktivieren                       | `zypper modifyrepo -d <NAME>`           |
| Autorefresh aktivieren                        | `zypper modifyrepo -f <NAME>`           |
| Autorefresh deaktivieren                      | `zypper modifyrepo -F <NAME>`           |

## Containerbasierte Paketmanager

Containerbasierte Paketmanager haben das Ziel komplett unabhängig von der Distribution zu sein und damit eine leichtere Verteilung von Software zu ermöglichen. Das wird erreicht indem zur eigentlichen Software auch noch die meisten **Dependencies** gepackt wird. Zudem wird die Software in einem **Container (Sandbox)** gestartet um unabhängiger von der Distribution zu laufen und einen Zusätzlichen Schutzmechanismus zu bieten. Containerbasierte Paketmanager basieren meist auf einen Hintergrundprozess der unter anderem automatisch nach Updates suchen kann.

Vorteile sind dass die Anwendung **fast überall läuft**, unabhängig von der Linux Distribution. **Kein Dependency Management** braucht und somit schnell und einfach installiert ist. **Mehrere Versionen** gleichzeitig unterstützt, da die Pakete erst in der Sandbox ausgepackt werden. **Automatische Updates** durch den Hintergrundprozessprozess.
Die Nachteile solcher Pakete sind **Riesige Pakete** aufgrund aller Dependencies in dem Paket. Das ist vor allem dann Spürbar, wenn man zum fünften mal die selbe Grafikbibliothek installiert die jede GUI braucht. **Langsamerer Start** und **Höherer Speicherbedarf** durch die Sandbox und das einrichten der Umgebung.

### Snap

Snap ist ein Paketmanager von Ubuntu. Durch die einfache verteilung von Software ist die Nutzung ebenfalls entsprechend einfach. Der Befehl `snap` redet mit dem Hintergrundprozess `snapd`, der dann die Suche, Installation und Updates durchführt.

| Beschreibung | snap |
| ------------ | ---- |
| Paket suchen | `snap find <PATTERN>` |
| Paket installieren | `snap install <PACKAGE>` |
| Paket deinstallieren | `snap remove <PACKAGE>` |

> [!TIP]
> Snap findet außerhalb von Ubuntu nahezu keine Anwendung, da es kein frei wählbares dezentrales Repository zulässt. Ubuntu vertreibt aber wichtige Software wie den Firefox mittels Snap, was oft als beschränkend wahrgenommen wird und ein Grund ist warum viele Ubuntu nicht mögen.

### Flatpak

Flatpak ist eine alternative zu Snap von RedHat mit freier Quellenwahl und viel mehr Unterstützung in einer Vielzahl an Distributionen die nicht nur zu RedHat gehören.

| Beschreibung                          | flatpak                             |
| ------------------------------------- | ----------------------------------- |
| Quellen anzeigen                      | `flatpak remotes`                   |
| Pakete suchen                         | `flatpak search <PATTERN>`          |
| Paket installieren                    | `flatpak install <REMOTE> <APP-ID>` |
| Paket entfernen                       | `flatpak uninstall <APP-ID>`        |
| Installierte Pakete auflisten         | `flatpak list`                      |
| Updates durchführen                   | `flatpak update`                    |
| Informationen zu einem Paket anzeigen | `flatpak info <APP-ID>`             |

Flatpak verwendet sogenannte Remotes, die als Softwarequellen dienen. Diese können hinzugefügt oder entfernt werden. Der wichtigste Remote ist Flathub, welcher eine große Sammlung an Desktop-Anwendungen bereitstellt.

Beispiel zum Hinzufügen von Flathub:
```
flatpak remote-add --if-not-exists flathub https://flathub.org/repo/flathub.flatpakrepo
```