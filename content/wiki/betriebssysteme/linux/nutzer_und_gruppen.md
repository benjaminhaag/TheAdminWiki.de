---
date: '2026-04-19T12:07:53Z'
draft: true
title: 'Nutzer und Gruppen'
image: "/images/linux_nutzer_und_gruppen.webp"
tags: ["betriebssysteme","linux","nutzer", "gruppen", "benutzer"]
---

Linux ist ein Mehrbenutzersystem. Mehrere Personen, Dienste und Anwendungen können gleichzeitig auf demselben System arbeiten. Damit dabei nicht jeder uneingeschränkt auf alle Dateien, Prozesse oder Systemeinstellungen zugreifen kann, verwendet Linux ein Berechtigungssystem auf Basis von Benutzern und Gruppen.

Jeder Prozess läuft dabei im Kontext eines Benutzers. Dadurch kann das System unterscheiden:

 - Wer Dateien lesen oder verändern darf
 - Wer Programme ausführen darf
 - Wer administrative Rechte besitzt
 - Welche Dienste voneinander getrennt werden sollen

Besonders auf Servern ist dieses Konzept essenziell. Webserver, Datenbanken oder Backup-Dienste erhalten häufig eigene Benutzerkonten, um Systeme sicher voneinander zu isolieren. Gruppen erleichtern zusätzlich die gemeinsame Verwaltung von Berechtigungen, beispielsweise wenn mehrere Benutzer Zugriff auf dieselben Dateien benötigen.

## Benutzertypen

Linux kennt technisch zwei Arten von Nutzern, logisch sogar drei. Die meisten Distributionen verwenden folgende Aufteilung:

 - **UID 0**: root
 - **UIDs 1-999**: Systemnutzer
 - **UIDs 1000+**: normale Nutzer

Technisch unterscheidet der Linuxkernel nur zwischen UID 0 (root) und anderen UIDs. Der root Nutzer gilt als systemweiter Administrator und besitzt nahezu uneingeschränkte Rechte. Das System baut hier und da leichte Hürden ein um gröbere Versehen zu vermeiden, diese sind aber leicht zu umgehen.
Nutzer aller anderen UIDs sind unprivilegierte Nutzer und dürfen nur sehr wenig, oder mit ausdrücklicher Erlaubnis.

Zur leichteren Übersicht teilen die meisten Distributionen zudem zwischen Systemnutzer und normale Nutzer. Normale Nutzer sind die, die vorm Bildschirm sitzen und mit dem System arbeiten. Allerdings macht es auch Sinn bestimmten Diensten wie Webservern, Datenbanken, und anderen Services nur die Berechtigungen zu geben die sie wirklich brauchen. Dafür erstellt man Systemnutzer. Oft besitzen Systemnutzer keine interaktive Anmeldung und kein Homeverzeichnis. Sie existieren hauptsächlich, damit Prozesse unter einer eigenen UID laufen können und der Kernel Berechtigungen sauber trennen kann.

### Login Prozess

Wenn sich ein als normaler Benutzer am System anmelden möchte wird man meist mit einem Login begrüßt. Das kann eine grafische Benutzeroberfläche sein, ein textbasiertes Terminal, oder eine Remoteverbindung. Wann immer das System nicht weiß wer der Nutzer ist, wird er gebeten sich zu authentifizieren. Meist mit Username und Passwort. Sobald der Nutzer seine Anmeldeinformationen eingegeben hat liegt es am Loginprozess sie zu verifizieren und zu einer UID aufzulösen.

Anschließend fordert der Loginprozess einen neuen Prozess beim Kernel an. Der Loginprozess läuft typischerweise mit der UID 0 (root) und darf deshalb beim Kernel einen neuen Prozess mit der UID des angemeldeten Benutzers starten. Prozesse mit UIDs anders als 0 dürfen das aus offensichtlichen Sicherheitsgründen nicht und bekommen immer einen Prozess mit der selben UID. Danach läuft der neue Prozess unter der UID des angemeldeten Benutzers und hat die entsprechenden Berechtigungen. 

Wichtig zu verstehen ist, dass der Benutzer selbst nie direkt am System arbeitet. Der Nutzer vorm Bildschirm öffnet keine Dateien, führt keine Programme aus und verändert keine Einstellungen. Stattdessen gibt er Befehle an eine Shell (Grafisch durch klicken oder Textuell durch Kommandos). Die Shell ist ein Prozess mit der UID des angemeldeten Nutzers, die dann den Befehl ausführt und die Datei öffnet, das Programm ausführt, oder Einstellungen ändert. 

> [!IMPORTANT]
> Der Kernel prüft also nicht wer gerade auf einen Knopf gedrückt hat, sondern ob der Prozess der die Anfrage stellt die entsprechenden Berechtigungen hat.

> [!TIP]
> Bei Systemnutzern fällt die Authentifizierung weg. Das Initsystem weiß, welche Prozesse es mit welcher UID starten soll und startet sie analog zum Loginprozess.

## Gruppen

Benutzer alleine reichen für die Rechteverwaltung eines Mehrbenutzersystems oft nicht aus. Häufig sollen mehrere Benutzer gemeinsam auf bestimmte Dateien oder Ressourcen zugreifen können, ohne sich denselben Benutzeraccount teilen zu müssen. Dafür verwendet Linux Gruppen.

Auch Gruppen werden intern über numerische IDs verwaltet, den sogenannten **Group IDs (GID)**. Jeder Benutzer, und damit Prozess, besitzt neben einer Benutzer-ID auch mindestens eine primäre Gruppe sowie optional weitere zusätzliche Gruppen. Diese Gruppenmitgliedschaften bestimmen, auf welche gemeinsam genutzten Ressourcen ein Prozess zugreifen darf. Die Gruppenzugehörigkeit der Prozesse ergibt sich analog zur UID aus den Zugehörigkeiten des Benutzers.

Während Benutzer hauptsächlich festlegen, wer ein Prozess ist, definieren Gruppen häufig, auf welche gemeinsamen Ressourcen ein Prozess zugreifen darf. Wenn beispielsweise mehrere Benutzer gemeinsam an Dateien arbeiten sollen, können sie und ihre Prozesse Mitglied derselben Gruppe sein. Anschließend lassen sich Berechtigungen direkt für diese Gruppe vergeben, statt jeden Benutzer einzeln zu konfigurieren.

### Primäre und Sekundäre Gruppen

Die Primäre Gruppe wirkt ersteinmal unnötig, da Berechtigungen in der Praxis fast immer über sekundäre Gruppen organisiert werden. Sie erfüllt jedoch einen wichtigen Zweck: Jede Datei unter Linux besitzt neben einem Besitzer auch genau eine Besitzergruppe.

Neue Dateien erhalten standardmäßig die primäre Gruppe des Prozesses, der sie erstellt hat.

Damit Benutzer Dateien nicht versehentlich einer großen gemeinsamen Gruppe zuordnen, verwenden viele Distributionen sogenannte User Private Groups (UPG). Dabei erhält jeder Benutzer automatisch eine eigene primäre Gruppe mit demselben Namen wie der Benutzer selbst. Dadurch gehören neu erstellte Dateien standardmäßig nur dem jeweiligen Benutzer und dessen privater Gruppe.

> [!TIP]
> Rootprozesse wie Init Prozesse oder Login Prozesse gehören meist der Primären Gruppe 0 (root) an. Allerdings ist die Gruppe nur symbolisch, da jeder Prozess eine Primäre Gruppe braucht. In Gruppe 0 sein hat keine besondere Bedeutung wie UID 0.

## PAM und NSS

Der Linuxkernel selbst kennt nur die UID und GID, und arbeitet intern nicht mit Usernamen und Passwort. Er überlasst die authentifizierung den Prozessen wie SSH, Logind, grafischen Benutzeroberflächen, FTPd, und vielen mehr. Ohne eine gemeinsame Schnittstelle müsste jede Anwendung ihre eigene Authentifizierungslogik implementieren und unterschiedliche Verfahren wie die lokale Anmeldung, Active Directory, LDAP, Kerberos, NTLM, und viele mehr, selbst unterstützen. Um hier Ordnung ins Chaos zu bringen bieten die **Pluggable Authentication Modules (PAM)** eine einheitliche Lösung. Prozesse wie SSH und Logins übergeben den Authentifizierungsprozess an die PAM Bibliothek und bekommen dann nurnoch zurück ob der Nutzer erfolgreich authentifizieurt wurde oder nicht. PAM selbst ruft zur authentifizierung Module auf und bleibt so flexibel. Die Module implementieren dann die lokale Authentifizierung, AD, LDAP, 2FA, und vieles mehr. Der Administrator kann PAM konfigurieren um eine andere Authentifizierungsmethode zu konfigurieren.

Ähnlich zur Authentifizierung überlässt der Linuxkernel auch die Auflösung von Nutzername zu UID und Gruppenzugehörigkeiten den Prozessen selbst. Da die Informationen dazu auch aus mehreren Quellen stammen können, gibt es in Linux den **Name Service Switch (NSS)**. Der NSS wird zur Namensauflösung genutzt und dient neben der Auflösung von Nutzernamen auch der Auflösung von Gruppennamen, Hostnamen und vielem mehr.

> [!TIP]
> Dadurch kann ein Benutzer existieren, obwohl kein entsprechender Eintrag in `/etc/passwd` vorhanden ist.

## Benutzer und Gruppenverwaltung

Im Standardfall (ohne AD, LDAP, Kerberos und co) liegen die Benutzer nach POSIX Standard im Dateisystem. Historisch bedingt teilt sich die Nutzerverwaltung in die Dateien `/etc/passwd` für Benutzer und `/etc/group` für Gruppen auf, wo alle Informationen über Benutzer und Gruppen gespeichert waren. Das hat allerdings ein großes Problem. Damit Benutzer wissen wem eine Datei gehört und nicht mit einer zufällig wirkenden Zahl konfrontiert werden, müssen sie UIDs und GIDs zu Klarnamen auflösen können. Dafür brauchen sie Zugriff auf die beiden Dateien. Obwohl Passwörter heutzutage gehashed gespeichert werden, wäre es ein Sicherheitsrisiko wenn jeder Nutzer Zugriff darauf hätte. Deshalb wurden beide Dateien nachmal aufgeteilt. Öffentlich bekannte Daten über Benutzer liegen in `/etc/passwd`, bzw `/etc/group` über Gruppen, und Sensible Daten wie Passwörter in `/etc/shadow` für Benutzer, bzw. `/etc/gshadow` für Gruppen.

`/etc/passwd` sind ist folgt aufgebaut:
```txt
username:password:UID:GID:GECOS:home:shell
```
**username** steht dabei wenig überraschend für den Usernamen. Beim **password** hingegen wird man auf modernen Systemen ein `x` finden. Das bedeutet, dass das Passwort in `/etc/shadow` ausgeladert wurde. Alternativ kann hier ein Hash, oder Klartext stehen. Moderne Implementierungen von PAM ignorieren das allerdings aus Sicherheitsgründen und erzwingen die Nutzung von Shodow.

**UID** ist die UID des Nutzers, während **GID** die Primäre GID ist. Zusätzliche Gruppen werden in `/etc/group` bwz. `/etc/gshadow` für sensible Zugehörigkeiten festgelegt. Der Name **General Electric Comprehensive Operating System (GECOS)** stammt aus den Anfängen von UNIX und dem Bedarf die Nutzerdefinierung mit dem Betriebssystem GECOS konsistent und kompatibel zu halten. Der Inhalt dieses Feldes kann frei gewählt werden und bietet zusätzliche Informationen wie den vollen Namen, Beschreibungen, Telefonnummer, etc. **home** ist das Verzeichnis in das der Benutzer gesetzt wird wenn er sich einloggt. Hier befinden sich meist die Privaten Dateien des Benutzers, oder die Dateien des Systemdienstes. **shell** definiert die Loginshell. Technisch gesehen ist das das Programm was zum Beispiel vom Loginprozess oder SSH gestartet wird, wenn der Nutzer sich erfolgreich angemeldet hat. Bei den meisten normalen Benutzern ist das `/bin/bash`. `/usr/sbin/nologin` ist ein Programm das sich sofort wieder beendet. Es dient dazu Systemnutzer sofort wieder rauszuschmeißen falls sie sich doch irgendwie einloggen sollten. Zum Beispiel aufgrund einer Fehlkonfiguration.

`/etc/shadow` ist wie folgt aufgebaut:
```txt
username:password:DateOfLastChange:MinAge:MaxAge:Warn:Inactivity:Expiration:Reserved
```
**username** ist der Nutzername und dient der Zuordnung zum Nutzer in `/etc/passwd`. **password** speichert das tatsächliche Passwort als Hash. Alternativ kann hier auch `*` stehen, dann ist kein Passwort gesetzt und der Nutzer kann sich nicht einloggen. Ein `!` vor dem Passwort heißt, dass es gesperrt ist. Auch dann kann sich der Nutzer nicht einloggen, allerdings ist das Passwort noch vorhanden. Ein `*` kann nicht entsperrt werden ohne ein neues Passwort zu setzen. **DateOfLastChange** gibt an, wann das Passwort zuletzt geändert wurde. Das wird vor allem in Verbindung mit **MinAge** und **MaxAge** genutzt, welche angeben nach wie vielen Tagen das Passwort erneut geändert werden kann (MinAge) oder muss (MaxAge). **Warn** gibt an wieviele Tage vor Ablauf des Passwortes der Nutzer eine Warnung beim Einloggen erhält. Nach Ablauf von MaxAge ist der Account nicht gesperrt, der Nutzer wird aber nur nach einer Passwortänderung reingelassen. **Inactivity** gibt an, wieviele Tage nach Ablauf des Passwortes der Account gesperrt werden soll. Ist der Account gesperrt, muss das Passwort von einem Administrator zurückgesetzt werden. **Expiration** sperrt den Account automatisch nach einer bestimmten Anzahl an Tagen unabhängig von Passwortänderungen. Das ist vor allem bei Gastaccounts sinnvoll. **Reserved** ist ein (noch) Unspezifiziertes Feld und sollte frei bleiben.

> [!TIP]
> Standardmäßig laufen weder Passwörter noch Accounts ab. Ein Administrator muss das explizit konfigurieren. Passwörter können auch direkt wieder geändert werden.

`/etc/group` ist wie folgt aufgebaut:
```txt
groupname:password:GID:members
```
**groupname** und **GID** definieren den Klarnamen und die ID der Gruppe. Normalerweise weist der Administrator (root) Gruppenzugehörigkeiten zu. Allerdings unterstützt Linux auch die Möglichkeit Gruppenpasswörter zu vergeben, mit denen ein Nutzer einer Gruppe vorübergehend beitreten kann. Auch hier ist in den meisten modernen Distributionen nur ein `x` eingetragen. **members** ist eine kommagetrennte liste an Nutzernamen, den Mitgliedern. Im Memberfeld sind keine Benutzer eingetragen die die Gruppe bereits als primäre Gruppe haben.

`/etc/gshadow` ist wie folgt aufgebaut
```txt
groupname:password:administrators:members
```
**groupname** ist analog zur `/etc/shadow` dazu die Informationen der korrekten Gruppe in `/etc/group` zuzuordnen. **password** ist das Gruppenpasswort. Da das selten genutzt wird ist hier meist ein `*` oder sogar `!*` zu sehen. **administrators** ist eine Liste an Nutzernamen. Gruppenadministratoren können auch ohne Rootrechte Mitglieder verwalten und das Gruppenpasswort setzen. Auch das wird in der Praxis selten genutzt. **members** ist analog zu den Members in `/etc/group`, allerdings werden hier Mitglieder aufgelistet von denen nicht jeder wissen soll, dass sie Mitglieder sind.

### Befehle zur Benutzerverwaltung

Auch wenn ein Administrator theoretisch alle Dateien manuell bearbeiten und Homeverzeichnisse anlegen kann ist das nicht empfohlen. Tippfehler können schnell fatale Folgen haben. Stattdessen bietet das GNU Projekt viele Werkzeuge zur Benutzerverwaltung bereit die stattdessen genutzt werden sollten.


#### Benutzer hinzufügen
Um einen Benutzer zu erstellen kann `useradd` genutzt werden. Unter Ubuntu stehen Administratoren das Wrapperskript `adduser LOGIN` zur Verfügung. Es nutzt selbst `useradd` und stellt eine komfortable interaktive Abfrage bereit und integriert besser in das Ubuntu Ökosystem.
```bash
useradd [options] LOGIN
    -c GECOS                # Kommentare hinzufügen
    -d home_dir             # alternatives Homeverzeichnis
    -e expirationdate       # Ablaufdatum
    -f inactive             # Passwortänderung
    -g primary_group        # Primäre Gruppe
    -G secondary_group      # Sekundäre Gruppe
    -k skeleton_path        # Alternatives Skeleton
    -m                      # Homeverzeichnis erstellen
    -M                      # kein Homeverzeichnis erstellen
    -s SHELL                # Login Shell
    -u UID                  # alternative UID
```

#### Benutzer ändern
Ist ein User einmal angelegt, kann man ihn mit `usermod` verändern. die Optionen sind dabei analog zu `useradd` mit ein paar zusätzlichen optionen.
```bash
usermod [options] LOGIN
    -aG secondary_gid       # Sekundäre Gruppe anhängen
    -l LOGIN                # Nutzername ändern
    -L                      # Nutzer Sperren
    -U                      # Nutzer entsperren
```
Die Option `-aG` ist deshalb notwendig, da `-G` bereits existierende Mitgliedschaften überschreiben würde. `-aG` fügt die Gruppe der bereits bestehenden Liste hinzu.

#### Benutzer löschen
Wenn ein Nutzer garnicht mehr benötigt wird, kann er wieder gelöscht werden. Auch hier bietet Ubuntu `deluser LOGIN` als Wrapper an.
```bash
userdel [options] LOGIN
    -r                      # homeverzeichnis löschen
```

#### Informationen abrufen

Benutzer können ihre Gruppen selbst und alternativ öffentliche Informationen Anderer abfragen. `groups` gibt eine simple liste aller Mitgliedschaften zurück, während `id` zudem die entsprechenden GIDs und die UID des Nutzers ausgibt.

`getent` ist das Userprogramm um eine Namensauflösung von NSS zu bekommen. Mit `getent passwd LOGIN` kann der passwd Eintrag eines Nutzers angefragt werden, der auch aus anderen Quellen stammen kann. LOGIN kann hierbei sowohl die GID, als auch der Nutzername sein. `getent group GRUPPE` gibt entsprechend die Gruppendetails zurück. Wenn kein LOGIN und keine GRUPPE angegeben ist wird eine auflistung aller lokalen Gruppen zurückgegeben. Da externe Verzeichnisse wie Active Directory zu groß und teilweise nicht abfragbar sein können ist die Liste bei nicht-lokalen Quellen potentiell verkürzt oder nicht vorhanden.

#### Passwort ändern

`passwd` ist ein tool zum Ändern des Passworts. Ohne Parameter und LOGIN ändert ein Nutzer sein eigenes Passwort nach eingabe des aktuellen. Root kann das Passwort von jedem Nutzer ändern ohne das alte zu kennen. Flags können genutzt werden um Passwörter stattdessen zu sperren, entsperren, und vieles mehr.
```bash
passwd [options] [LOGIN]
    -d              # Passwort löschen
    -e              # Passwortänderung erzwingen
    -i INACTIVITY   # inactivity setzen
    -l              # Passwort sperren
    -u              # Passwort entsperren
    -n MinAge       # MinAge setzen
    -x MaxAge       # MaxAge setzen
    -w Warn         # Warn setzen
    -S              # Passwortdetails anzeigen
```

Eine mächtigere alternative zum Ändern der Metadaten bietet `chage`.
```bash
chage [options] LOGIN
    -l              # Informationen Anzeigen
    -d last_day     # Letzte Änderung setzen
    -E expiration   # Ablaufdatum setzen
    -I inactive     # Inaktivität setzen
    -m min_age      # MinAge setzen
    -M max_age      # MaxAge setzen
    -W warn         # Warn setzen
```

### Befehle zur Gruppenverwaltung

#### Gruppe hinzufügen

Um eine Gruppe zu erstellen kann `groupadd` genutzt werden. Unter Ubuntu stehen Administratoren das Wrapperskript `addgroup LOGIN` zur Verfügung. Es nutzt selbst `useradd` und stellt eine konfortable interaktive Abfrage bereit und integriert besser in das Ubuntu Ökosystem.
```bash
groupadd [options] GRUPPE
    -g GID          # GID setzen
```

#### Gruppe löschen

Um eine Gruppe zu entfernen kann `groupdel` genutzt werden, oder `delgroup` unter Ubuntu.
```bash
groupdel GRUPPE
```

#### Gruppenadministration

Gruppen unter Linux haben das Konzept der Gruppenadministratoren. Gruppenadministratoren können Mitglieder verwalten und Gruppenpasswörter setzen um temporäre Mitgliedschaften zu gewähren.
```bash
gpasswd -A LOGIN    # Füge LOGIN als Administrator hinzu
```

Anschließend kann der Administrator Mitglieder verwalten und Gruppenpasswörter verwalten
```bash
gpasswd [options] GRUPPE
                # (keine optionen) Gruppenpasswort setzen
    -r          # Gruppenpasswort entfernen
    -a LOGIN    # Nutzer hinzufügen
    -d LOGIN    # Nutzer entfernen
```

Anschließend können Nicht-Mitglieder der Gruppe mit dem Gruppenpasswort temporär beitreten
```bash
newgrp GRUPPE
```
Die neue Gruppenzugehörigkeit gilt nur für die aktuelle Shell und deren Kindprozesse.

## Defaultwerte Einstellen

Mithilfe der Befehle können Details wie Ablaufdatum, Homeverzeichnus, Shell und weiteres eingestellt werden. Statt jedem einen Haufen Optionen mitzugeben können allerdings auch die Standardwerte eingestellt werden.

### Skeleton

Beim Erstellen des Homeverzeichnisses wird nicht einfach ein leeres Verzeichnis erstellt. Meist ist es bereits mit diversen Konfigurationsdateien für zum Beispiel Bash gefüllt. Unter einigen grafischen Benutzeroberflächen wird auch ein Ordnersystem mit Ordnern wie `Dokumente`, `Downloads` und anderen erstellt. 

Nach dem Erstellen des Ordners wird ein sogenanntes **Skeleton** hineinkopiert. Standardmäßig liegt es unter `/etc/skel`. Dort können weitere Dateien hinterlegt werden die dem neuen Nutzer von Anfang an bereitgestellt werden sollen.

### Defaults

Die Standardwerte von `useradd` und `groupadd` liegen unter `/etc/login.defs` und beinhalten neben den bekannten Optionen auch optionen wie `UID_MIN` um anzugeben ab welcher UID normale Benutzer losgehen.