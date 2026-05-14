---
date: '2026-05-14T11:13:56Z'
draft: false
title: 'Linux'
image: "/images/linux.webp"
tags: ["betriebssysteme","linux"]
---

Linux selbst ist kein Betriebssystem, sondern ein Betriebssystemkernel. Das wissen vermutlich die meisten. Allerdings Wissen viele nicht' was genau der Unterschied ist.

## Kernel


Moderne Rechnerarchitekturen sind mit Privilegienstufen ausgestattet. x86 nennt diese Privilegienstufen Ringe. Ring 0 (Kernelspace) bedeutet hierbei vollen Zugriff auf sämtliche Hardwarekomponenten, während Ring 3 (Userspace) nur den nötigsten Zugriff für Nutzerprogramme bietet. Ring 1 und 2 existieren zwar ebenfalls, werden von modernen Betriebssystemen jedoch kaum genutzt. Gerätetreiber laufen in der Praxis meist ebenfalls im Kernelspace (Ring 0).

![X86 Privilegien](/assets/rings.svg)

> [!TIP]
> Moderne Prozessoren besitzen zusätzlich weitere Privilegierungsmechanismen wie Hypervisor-Modi (oft informell als „Ring -1“ bezeichnet) oder System Management Mode (SMM).

Der Kernel muss also im Kernelspace sämtliche Hardware und Zugriffe von "oben" verwalten. Um mehr Sicherheit in Multiusersystemen zu gewährleisten ist der Kernel auch für Benutzerverwaltung und Berechtigungen zuständig. Die Benutzer eines Betriebssystems sind grundsätzlich nicht am Kernel selbst interessiert, sondern an Nutzerprogrammen. Selbst Administratoren reden mit dem Kernel und konfigurieren ihn nur über Nutzerprogramme. Das schon allein aus Sicherheitsgründen, da sonst jeder direkt in den Kernel gehen und das System übernehmen könnte.

Wenn ein Nutzerprogramm auf Arbeitsspeicher zugreifen will, kann es das direkt. Arbeitsspeicherzugriffe sind in modernen Systemen dank der **Memory Management Unit (MMU)** und **Paging** isoliert und somit sicher. Für alles was bestimmte Berechtigungen braucht, wie zum Beispiel das **Spawnen neuer Prozesse**, **Zugriffe auf Dateien** oder andere **Treiber** muss das Programm allerdings auf den Kernel zurückgreifen. Dieser entscheidet dann, ob das Programm darf was es versucht und führt die Aktion sicher durch.

Diese Aufrufe an den Kernels heißen **Systemaufrufe**, **System Calls** oder **Syscalls**. Die CPU bietet dafür **Interrupts** die das Programm auslösen kann. Interrupts und Exceptions sorgen dafür, dass die CPU in den Kernelspace wechselt. Beispiele für Exceptions sind Division durch Null oder Zugriffe auf ungültigen Arbeitsspeicher. Wird ein Interrupt ausgelöst wechselt die CPU sofort in den Kernelspace und springt an eine Adresse im Kernel den der Kernel vorher selbst angegeben hat.

![OS Architektur](/assets/OSArchitecture.svg)

> [!TIP]
> Historisch wurden unter Linux Systemaufrufe oft über `int 0x80` durchgeführt. Moderne Systeme verwenden meist effizientere CPU-Instruktionen wie `syscall` oder `sysenter`.

```c
// include libc
#include <unistd.h>

int main() {
    write(1, "Hello, World\n", 14);
    return 0;
}
```

```asm
# write(1, msg, 14)
mov eax, 4      # sys_write
mov ebx, 1      # stdout
mov ecx, msg    # message
mov edx, 14     # strlen(msg)
int 0x80        # Interrupt No. 80 (syscall)

```

Der Kernel kann dann seine Aufgabe erledigen, die CPU wieder in den Userspace bringen und die Kontrolle wieder an das Nutzerprogramm übergeben.

> [!TIP]
> Damit Programmierer Syscalls nicht jedes Mal selbst schreiben müssen gibt es Bibliotheken wie die **glibc** unter Linux die die Aufrufe in einfach nutzbare C Funktionen abstrahiert.

## Distributionen

Was wir als Betriebssystem bezeichnen ist ein Bündel aus Kernel, Treibern und Nutzerprogrammen. Da Linux nur den Kernel stellt und Geschmäcker verschieden sind, haben sich im Laufe der Zeit viele Betriebssysteme mit Linux Kernel gebildet. Wir nennen solche Bündel im Linux kontext auch **Linux Distributionen**.

Viele Linux-Distributionen bestehen nicht nur aus dem Linux-Kernel, sondern auch aus Werkzeugen des GNU-Projekts. GNU hatte ursprünglich das Ziel, ein freies UNIX-ähnliches Betriebssystem zu schaffen. Der eigene Kernel GNU Hurd wurde jedoch nie so verbreitet wie Linux. Deshalb wurden GNU-Werkzeuge häufig mit dem Linux-Kernel kombiniert. Aus diesem Grund sprechen manche von **GNU/Linux**, wenn sie das Gesamtsystem aus Linux-Kernel und GNU-Userland meinen.

Distributionen gibt es viele. Die meisten fangen nicht von Null an, sondern bauen auf andere Distributionen auf und passen lediglich Kleinigkeiten an. Es gibt Tausende Distributionen und es ist unmöglich alle aufzulisten. Tiefere Einblicke bietet [Distrowatch](https://distrowatch.com/), hier sind einmal die wichtigsten aufgelistet die jeder kennen sollte.

### Debian

**Debian** ist eine rein Community getriebene Linux Distribution. Das bedeutet, dass hinter Debian kein kommerzielles Unternehmen steckt, sondern eine Community aus Enthusiasten, Privatpersonen, Organisationen und ja, auch Unternehmen. Die Debian Community setzt auf Stabilität statt Aktualität. Software für Debian ist traditionell eher in älteren Versionen verfügbar, da diese sehr gut getestet ist und garantiert stabil läuft. Natürlich werden Sicherheitsupdates eingespielt, ein risiko besteht daher nicht. Debian ist besonders gut für Server geeignet, da diese über Jahre hinweg stabil und zuverlässig laufen sollen. Weniger geeignet ist Debian hingegen für Nutzer, die aktuellere Versionen bevorzugen.

Eines der wichtigsten Debian Derivate ist **Ubuntu**. Hinter Ubuntu steckt das Unternehmen **Canonical**, was sich zur Aufgabe gemacht hat die Stabilität von Debian nutzerfreundlich zu machen und Unternehmen als echte Alternative anzubieten. Ubuntu teilt sich auf in eine Desktop und eine Server Version, was maßgeblich den Umfang der mitgelieferten Software beeinflusst. Wie der Name schon sagt ist Ubuntu Desktop für Rechner optimiert, an denen Personen arbeiten und kommt zum Beispiel mit einer grafischen Benutzeroberfläche. Ubuntu Server hingegen kommt mit software die für den Betrieb von Servern optimiert ist und enthält standardmäßig eine Kommandozeile als Benutzeroberfläche und einen Remotezugriff. Selbst der Installer unterscheidet sich. Ubiquity, der Installer für die Desktop version ist ebenfalls grafisch, während Subiquity für Ubuntu Server textuell ist und die Installation sogar per Fernzugriff durchgeführt werden kann.

Im Laufe der Zeit haben sich für Linux viele grafische Benutzeroberflächen entwickelt. Einige Ubuntuderivate wie **Lubuntu (LXQt)**, **Xubuntu (XFCE)** und **Kubuntu (KDE Plasma)** sind im Grunde Ubuntu mit alternativem Aussehen.

**Linux Mint** hat sich sowohl beim Aussehen, als auch bei der Nutzung stark an Windows XP orientiert um Umsteigern einen einfachen Einstieg in Linux zu gewährleisten nachdem Microsoft den Support für Windows XP eingestellt hatte. Damit hat Linux Mint auch heute noch große Beliebtheit gewonnen.

{{< mermaid >}}
---
title: "Debian Familie"
config:
    theme: "base"
    gitGraph:
        showCommitLabel: false
        showBranches: true
        mainBranchName: 'Debian'
        parallelCommits: true
---
gitGraph TB:
    commit
    commit
    branch Ubuntu
    commit
    checkout Debian
    commit
    commit
    checkout Ubuntu
    branch "Lubuntu"
    commit
    checkout Ubuntu
    branch "Xubuntu"
    commit
    checkout Ubuntu
    branch "Kubuntu"
    commit
    checkout Ubuntu
    branch "Linux Mint"
    commit
    checkout Ubuntu
    commit
{{< /mermaid >}}

### Red Hat

**Red Hat, Inc.** ist ein Softwareunternehmen, welches Software für Unternehmen bereit stellt. Natürlich haben sie auch frühzeitig angefangen eine eigene Linux Distribution anzubieten, für die ihre Software optimiert ist. Allerdings braucht man kein RedHat Linux um die Software zu nutzen. **Red Hat Enterprise Linux (RHEL)** ist die Hauptversion von Red Hat und wird als kommerzielles Produkt angeboten. Wer Linux im Unternehmen Produktiv einsetzen möchte bekommt von Red Hat nicht nur eine Linux Distribution, sondern auch die neuesten Versionen wertvoller Enterprise Software und Support oben drauf.

**Fedora** dient als Innovationsplattform für neue Technologien, die später teilweise in Red Hat Enterprise Linux übernommen werden.

Linux ist Open Source unter der GPLv2 Lizenz und verlangt, dass sämtliche änderungen ebenfalls frei zur verfügung gestellt werden. Das bedeutet, wer sich sein eigenes RHEL gratis zusammenbauen möchte, kann Red Hat nach dem Source Code fragen. Da das extrem Aufwendig, aber durchaus interessant war, hat sich eine Community gebildet um genau das zu tun. Das Ergebnis war **CentOS**. Da CentOS und RHEL so ähnlich waren konnten Unternehmen, die CentOS im Einsatz haben, offiziellen Support und alle Vorzüge von RHEL dazu kaufen. CentOS wurde allerdings von Red Hat, Inc. aufgekauft, da die andere Pläne hatten. CentOS dient heute als Upstream für RHEL. Das bedeutet, CentOS ist kein Derivat mehr von RHEL, sondern genau andersrum. Red Hat testet mittels CentOS die Stabilität und nutzerfreundlichkeit ihrer Produkte an CentOS Nutzern. CentOS ist zwar weiterhin kostenlos, dient heute als Vorschau auf kommende RHEL-Versionen. Dadurch ist es aktueller, aber weniger konservativ als klassische Enterprise-Distributionen.

Die Community hat reagiert und mittlerweile **RockyLinux** als Nachfolgeprojekt von CentOS ins Leben gerufen.

{{< mermaid >}}
---
title: "Red Hat Familie"
config:
    theme: "base"
    gitGraph:
        showCommitLabel: false
        showBranches: true
        mainBranchName: 'Red Hat'
        parallelCommits: true
---
gitGraph TB:
    commit
    commit
    branch RHEL
    commit
    checkout RHEL
    branch "Fedora"
    commit
    checkout RHEL
    branch "CentOS"
    commit
    checkout RHEL
    branch "RockyLinux"
    commit
    checkout RHEL
    commit
{{< /mermaid >}}

### SUSE

Ähnlich wie Red Hat, Inc. war **Gesellschaft für Software und Systementwicklung mbH (S.u.S.E)** (Ja eine GmbH aus Deutschland, genauergesagt Nürnberg) ein Softwarehaus, das für Unternehmen Software geschrieben hat. Ähnlich zu RHEL haben sie **SUSE Linux** (heute **openSUSE**) als community Version entwickelt und dazu die kommerziellen Versionen **SUSE Linux Enterprise Desktop (SLE Desktop)** für den Desktop und **SUSE Linux Enterprise Server (SLE Server)** für Serversysteme entwickelt. openSUSE dient, ähnlich wie CentOS für RHEL, als upstream für SLE. Heute wurde SUSE (heute **SUSE S.A.**) mehrfach verkauft und umbenannt und sitzt, abgesehen von ein paar Außenstellen, nicht mehr in Deutschland.

{{< mermaid >}}
---
title: "SuSE Familie"
config:
    theme: "base"
    gitGraph:
        showCommitLabel: false
        showBranches: true
        mainBranchName: 'SUSE Linux'
        parallelCommits: true
---
gitGraph TB:
    commit
    commit
    branch openSUSE
    commit
    checkout 'SUSE Linux'
    branch "SLE Desktop"
    commit
    checkout 'SUSE Linux'
    branch "SLE Server"
    commit
{{< /mermaid >}}

### Arch

**Arch** ist eine besondere Linux Distribution, da sie genau wie Debian durch die Community getrieben ist. Im gegensatz zu Debian hat sich Arch vollkommene Freiheit in der Gestaltung auf die Fahne geschrieben und dient primär als Distribution für Bastler. Wer tief in Linux und seine Subsysteme eintauchen will, alles selbst einstellen, und kein Problem damit hat Tage damit zu verbringen ersteinmal alles zum Laufen zu bekommen ist bei Arch gut aufgehoben. Die Möglichkeiten sind unendlich und genau das reizt viele. Die Arbeit die dahinter steckt schreckt jedoch viele ab und ist ein wichtiger Grund, warum man Arch in Unternehmen nicht sieht.

**Manjaro** hat sich zur Aufgabe gemacht Arch wesentlich Benutzerfreundlicher zu machen. Das ist zwar zu großen Teilen gelungen, allerdings finden sich Nutzer dennoch oft vor komplexen Fehlern.

{{< mermaid >}}
---
title: "Arch Familie"
config:
    theme: "base"
    gitGraph:
        showCommitLabel: false
        showBranches: true
        mainBranchName: 'Arch'
        parallelCommits: true
---
gitGraph TB:
    commit
    commit
    branch Manjaro
    commit
    checkout Arch
    commit
{{< /mermaid >}}

### Gentoo

**Gentoo** ist eine sogenannte Metadistribution und damit sogar noch Bastelfreundlicher als Arch. Gentoo selbst trifft keine Annahmen darüber wer du bist und was du möchtest. Stattdessen setzt es dich vor eine leere Festplatte und ein minimalistisches Terminal. Deine erste Aufgabe ist es dann den Kernel selbst zu konfigurieren und kompilieren. Anschließend darfst du die meisten Treiber und Nutzerprogramme kuratieren und kompilieren. Gentoo ist deshalb eine Distribution, weil es Repositories bereitstellt von denen du dir den Quellcode sämtlicher Software ziehen kannst. Gentoo sollte grundsätzlich nur jemand in betracht ziehen der Linux sehr tief verstehen will, viel Zeit, und einen Starken rechner hat. Gentoo braucht zwar nicht viel zum Laufen, aber der Kompiliervorgang wird dadurch extrem beschleunigt.

Das wohl bekannteste Derivat von Gentoo ist **ChromiumOS**, also die Basis von **ChromeOS**. Im Gegensatz zu Gentoo müssen Nutzer hier nichts mehr selbst bauen. Im gegenteil. Ein Chromebook kommt bereits mit ChromeOS ausgeliefert.

{{< mermaid >}}
---
title: "Gentoo Familie"
config:
    theme: "base"
    gitGraph:
        showCommitLabel: false
        showBranches: true
        mainBranchName: 'Gentoo'
        parallelCommits: true
---
gitGraph TB:
    commit
    commit
    branch ChromiumOS
    commit
    checkout Gentoo
    commit
{{< /mermaid >}}

### Android

**Android Open Source Project (AOSP)** wird von der Android Alliance entwickelt. Die Android Alliance ist nicht Google, wobei Google einen großen Platz in ihr hat. AOSP ist zwar ein funktionsfähiges Betriebssystem, kommt aber ohne den meisten nützlichen Funktionen die Androidnutzer kennen. **Android** basiert auf AOSP, beinhaltet aber viele Dienste und Anwendungen von Google, ohne die sich AOSP relativ nutzlos anfühlt.

Da viele in der AOSP Community meinen, dass Android offen und frei vom Google-Zwang sein sollte, haben sie **LineageOS** entwickelt, was mittlerweile eine echte Alternative für Privatsphäreliebende darstellt. Allerdings können viele Apps ohne die Google Services trotzdem nicht verwendet werden. [**MicroG**](https://github.com/microg/GmsCore/wiki) ist ein Projekt um genau das Problem zu lösen. Es stellt Alternativen zu den Google Services bereit um Apps lauffähig zu machen, ohne Google selbst in das System zu lassen.

Aus einem Projekt AOSP sicherer zu machen heraus entstand **GrapheneOS**. Das Ziel von GrapheneOS ist Minimalismus und Sicherheit. Im Gegensatz zu LineageOS wird Benutzerfreundlichkeit als zweitrangig behandelt, aber nicht vernachlässigt.

{{< mermaid >}}
---
title: "AOSP Familie"
config:
    theme: "base"
    gitGraph:
        showCommitLabel: false
        showBranches: true
        mainBranchName: 'AOSP'
        parallelCommits: true
---
gitGraph TB:
    commit
    commit
    branch LineageOS
    commit
    commit
    checkout AOSP
    commit
    branch "GrapheneOS"
    commit
    checkout AOSP
    commit
{{< /mermaid >}}

### NixOS

**NixOS** ist eine relativ neue Distribution und verfolgt einen besonderen Ansatz. Statt das System zu installieren und dann anzupassen, wird alles in ein Systemzustand in einer Konfigurationsdatei festgehalten. Soll neue Software installiert werden, oder ein Update gefahren, wird die Konfigurationsdatei angepasst und das gesamte System neu ausgerollt. Das Prinzip war zwar schon vor NixOS in form von Konfigurationsmanagern wie **Ansible** da, NixOS bietet aber Konsequenz bis hin zur Partitionierung der Festplatte.

{{< mermaid >}}
---
title: "NixOS"
config:
    theme: "base"
    gitGraph:
        showCommitLabel: false
        showBranches: true
        mainBranchName: 'NixOS'
        parallelCommits: true
---
gitGraph TB:
    commit
    commit
{{< /mermaid >}}

### Ubuntu

Ubuntu ist eines der bekanntesten und beliebtesten Distributionen im professionellen Bereich. Es profitiert durch seine Ähnlichkeit zu Debian von einer großen Debian Community, bringt aber auch selbst eine eigene riesige Community mit. **Canonical** hat im Grunde nichts neues entwickelt, sondern lediglich Debian aufgeräumt, die Entwicklung unter einen Hut gebracht, nutzerfreundlich gemacht und neu verpackt. Dadurch funktionieren ~90% der dinge für Debian auch unter Ubuntu. Zudem bietet ubuntu eine große **deutschsprachige Community**, [ubuntuusers](https://ubuntuusers.de/) und ist deshalb im deutschsprachigen Raum unter Anfängern besonders beliebt.

Es gibt auch viele kritiken gegenüber Ubuntu, das meiste sind jedoch bewusste Entscheidungen, die durchaus auch positive Auswirkungen haben.

| Eigenschaften | Pro | Contra |
| - | - | - |
| Viele Features | Es funktioniert einfach | Es ist Bloat und damit ein Sicherheits und Performancerisiko |
| Starres System | Es geht wenig kaputt | Es ist unflexibel |
| Alte Pakete | Gut getestet und stabil | neue Featuren sind nicht verfügbar |
| Abstrahiert | Einfach zu Nutzen | Man nutzt kein klassisches Linux mehr |

Features die oft als Bloat angesehen werden sind zum Beispiel:
 - **SMB**, womit Ubuntu problemlos in Windows Umgebungen arbeitet
 - **Cloud Init**, für einfaches massendeployment
 - **Closed Source Treiber**, für breiten Hardware support
 - **Snap**, isolierte Anwendungen

Meist werden diese Features nicht gebraucht, sind aber ausführbarer Code (und somit potentielle Schwachstellen). Dennoch ist es oft sinnvoll, wenn einfach alles funktioniert.

#### Lifecycle

Neben regulären Sicherheitsupdates bietet Canonical jedes Jahr zwei neue Versionen für Ubuntu Desktop und Ubuntu Server. Die neuen Versionen kommen immer im **April** und **Oktober** und beinhalten das Veröffentlichungsdatum als Versionsnummer. Die April Version von 2026 heißt zum Beispiel **Ubuntu 26.04**. Im Oktober kommt dann *Ubuntu 26.10*. Zusätzlich zu den Versionsnummern haben Ubuntu Versionen, genau wie andere Debian Derivate, Codenamen. Bei Ubuntu werden Codenamen immer aus einem Adjektiv und einem Tier mit gleichem Anfangsbuchstaben zusammengesetzt. Dabei geht Canonical Alphabetisch vor. Ubuntu 25.04 heißt zum Beispiel *Plucky Puffin*, gefolgt von 25.10 *Questing Quokka* und 26.04 *Resolute Raccoon*. Debian benennt seine Distributionen im Übrigen nach Charakteren von Toy Story.

Ubuntu Versionen werden Grundsätzlich **9 Monate** unterstützt. Da das nicht viel ist, vor Allem für Server, stellt Canonical alle zwei Jahre (jedes gerade Jahr) im April eine **Long Term Support (LTS)** bereit, die dann **5 Jahre** unterstützt wird. Zahlende Kunden können nach den 5 Jahren **Extended Security Mainteinance (ESM)** zukaufen, wodurch die Version dann noch eine individuelle Zeit lang mit Sicherheitsupdates versorgt wird. Allerdings ist das Update auf eine neue Version unter Debian Derivaten besonders einfach.

## UNIX und POSIX

1969 wurde in den AT&T Bell Labs das Betriebssystem **UNIX** entwickelt. Die Hauptentwickler waren **Ken Thompson** und **Dennis Ritchie**, weitere wichtige Entwickler waren **Douglas McIlroy**, **Joe Ossanna** und **Rudd Canaday**. UNIX wurde von vielen Universitäten und Unternehmen verwendet. Durch die Vielzahl an Hardware auf denen es zum Einsatz kam gab es viele verschiedene Adaptionen die mit der Zeit auseinanderliefen.

Aus der Not heraus Ordnung ins Chaos zu bringen wurde 1988 vom **IEEE** der Standard **Portable Operating System Interface (POSIX)** entwickelt, der **Befehle**, **Aufbau** und **Verhalten** von Betriebssystemen beschreibt. Die Konsequenz daraus ist, dass POSIX kompatible Programme auf allen POSIX kompatiblen Betriebssystemen laufen. Allerdings gibt es auf fast allen POSIX kompatiblen Betriebssystemen auch erweiterte Schnittstellen. Programme die gegen Erweiterungen programmiert wurden laufen dann auch nur auf diesem Betriebssystem. Es ist also falsch anzunehmen dass Programme für macOS auch auf Linux laufen nur, weil beide Betriebssysteme POSIX kompatibel sind.

POSIX kompatible Betriebssysteme werden auch als **UNIX like** bezeichnet, da sie sich ähnlich wie UNIX verhalten und bedienen lassen. Allerdings sind nur **UNIX Zertifizierte** Betriebssysteme wie *macOS* anerkannte UNIX Systeme. Die Zertifizierung hat sonst keine weitere Bedeutung.

Jedes Betriebssystem was den Standard einhält kann sich gegen Geld UNIX Zertifizieren lassen. Allerdings gibt es auch noch einige Systeme die Nachfolger des originalen **AT&T UNIX** sind. Manche davon sind zertifiziert, aber nicht mehr alle.

{{<mermaid>}}
venn-beta
  set Windows
  set UNIX-like:200
  set ATNTUNIX["AT&T UNIX"]:70
    text BSD
  set UNIXZERTIFIZIERT["UNIX Zertifiziert"]:70
  set Linux:10
  union ATNTUNIX,UNIX-like:100
  union UNIXZERTIFIZIERT,UNIX-like:100
  union ATNTUNIX,UNIXZERTIFIZIERT[""]:15
    text macOS
  union Linux,UNIX-like:10
{{</mermaid>}}


> [!TIP]
> Microsoft Windows spielt natürlich nach seinen eigenen Regeln.
