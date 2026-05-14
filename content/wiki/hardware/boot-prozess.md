---
date: '2026-04-13T07:13:52Z'
draft: false
title: 'Boot Process'
image: '/images/boot.webp'
tags: ["hardware","boot_process"]
---

Der **Bootprozess** ist ein festgelegter Ablauf, den die CPUs beim Start eines Computers durchläuft, um das System betriebsbereit zu machen. Dieses Wissen ist besonders hilfreich für die Fehlerdiagnose während des Startvorgangs.

{{<mermaid>}}
flowchart LR
    CPU[CPU Reset] --> 
    Firmware["Firmware<br>(BIOS/UEFI)"] -->
    POST -->
    Devices["Boot Device<br>Suche"] -->
    MBR["MBR / EFI<br>Application"] -->
    Bootloader -->
    Kernel
{{</mermaid>}}

Vor allem **AMD** und **Intel** achten bei ihrer **x86**-Architektur stark auf Rückwärtskompatibilität. Das bedeutet, dass Programme die für den allerersten x86 Prozessor aus 1978 geschrieben wurden immer noch auf aktuellen Prozessoren laufen. Um den Vorgang vollständig zu verstehen, lohnt sich daher auch ein Blick in die Vergangenheit.

Andere Architekturen wie **ARM** und **RISC V** sind meist deutlich schlanker, da sie nicht mit den Altlasten von x86 belastet sind. Allerdings funktionieren sie im Wesentlichen sehr ähnlich. Da **ARM** und **RISC V** allerdings nur Spezifikationen darstellen, hängt der genaue Bootprozess stark von der jeweiligen CPU-Implementierung ab. Deshalb betrachten wir den Ablauf exemplarisch anhand von x86 genauer.

![x86 Boot](/assets/x86_boot.svg)

## Firmware

Der **Intel 8086** von 1978 ist der erste Prozessor, und der Namensgeber, der **x86** Familie. Mit einer `16 Bit` Wortbreite definierte er die Basis moderner Intel und AMD CPUs. In späteren Architekturerweiterungen wird dieses Verhalten als **Real Mode** bezeichnet.

Neben CPU und Arbeitsspeicher befindet sich auf jedem Motherboard Firmware (BIOS/UEFI). Wenn der Computer startet wird die Firmware an die Adresse `0xF0000` eingeblendet und die CPU beginnt bei der **Reset Vector Adresse** `0xFFFF0` mit der Ausführung.

> [!TIP]
> 16 Bit x86 Prozessoren haben einen 20 Bit Adressbus. Segmente (`16 bit`) werden mit `16` multipliziert um die Adresse zu berechnen. Das erlaubt es `16 bit` Prozessoren statt `64kB` \(2^{16}\) bis zu `1MB` \(2^{20}\) zu adressieren.

## BIOS

Das BIOS initialisiert zunächst alle wichtigen Komponenten und führt zunächst einen **Power On Self Test (POST)** durch, bei dem es die wichtigsten Komponenten wie RAM und Video testet. Ohne RAM läuft das System nicht, ohne Video können weitere Fehler nicht kommuniziert werden. Wenn dabei etwas schief geht meldet der POST das mittels akustischen Signalen über einen Piezolautsprecher. Ältere Systeme testen zudem Tastaturen und starten nicht, solange keine Tastatur angeschlossen ist. Moderne Motherboards haben teilweise Segmentdisplays, auf denen sie numerische Fehlercodes statt Akustiksignale ausgeben.

Je nach konfiguration durchsucht das BIOS anschließend alle Speichermedien in einer festgelegten Reihenfolge. Der erste Block eines Speichermediums, der **Master Boot Record (MBR)**, beinhaltet Informationen über die darauf enthaltenen Daten, darunter ob es sich um ein Bootmedium handelt oder nicht. Bootmedien sind Speichermedien die vom BIOS gestartet werden können und dann selbstständig laufen. Das BIOS erkennt das, wenn die letzten zwei Bytes des MBR `0x55AA` entsprechen. Das erste gefundene Bootmedium wird gestartet. Dabei wird der MBR in den Arbeitsspeicher an die Stelle `0x7C00` geladen und anschließend ausgeführt.

> [!TIP]
> Das klassische BIOS arbeitet blockbasiert und behandelt moderne Massenspeicher zunächst über Kompatibilitätsschnittstellen ähnlich wie klassische Festplatten. Moderne Massenspeicher wie SSDs werden beim klassischen BIOS-Boot über kompatible Blockgeräte-Schnittstellen angesprochen.

## First Booter

Der MBR besteht nur aus `512 Byte`, darunter die Partitionstabelle und Bootsignatur, effektiv `446 Byte`. Da das nicht ausreicht um ernsthafte Systeme zu bauen, liegt hier nur der **Urlader** (First Booter auf Englisch).

Der Urlader lädt je nach Bedarf weitere Sektoren von der Festplatte und springt dann in den geladenen Code. Dieser Code war ursprünglich das **Betriebssystem**, später allerdings eher ein **Bootloader**.

## Bootloader

Das BIOS war zwar auch bei `32 Bit` Systemen noch die Primäre Firmware um die CPU zu starten, wurde aber auf `16 Bit` ausgelegt und entsprach schnell nicht mehr den modernen Anforderungen. Bootloader erweitern die Möglichkeiten von BIOS Firmware um genau diese Anforderungen. Unter anderem bieten sie **Multiboot**, sodass mehrere Betriebssysteme auf einem Rechner installiert werden können und der Nutzer beim Start die Möglichkeit hat aus diesen auszuwählen. Des weiteren lässt der Bootloader das Betriebssystem auf einer `32 Bit` Architektur nicht im `16 Bit` Modus zurück wie das BIOS.

## Protected Mode

Der **Intel 80286** von 1982 führte den **Protected Mode** ein. Ein Ringbasiertes Privilegiensystem, bei dem äußere Ringe weniger Rechte haben als Innere. Vorher konnte jedes Programm direkt auf Hardware zugreifen, das ist jetzt dem Kernel im **Ring 0** vorbehalten. Der Ring 0 wird auch **Kernelspace** genannt und hat vollen Zugriff auf das gesamte System. Die Architektur sieht vier Privilegierungsringe vor, praktisch werden auf modernen Systemen meist nur Ring 0 und Ring 3 verwendet. Ringe 1 und 2 sind für Treiber gedacht. Je nachdem wieviel Zugriff ein Treiber benötigt, wird er in einen der beiden Ringe gesetzt. Allerdings haben sich die Treiberringe nie durchgesetzt. Monolitische Kernel wie *Linux*, *Windows NT* und *XNU (MacOS)* lassen die Treiber im Kernelspace um Komplexitäten zu vermeiden. **Ring 3** wird auch Userspace genannt und Programme die hier laufen haben nur Zugriff auf den Arbeitsspeicher und notwendige Teile der CPU. In der Theorie sollte das die Sicherheit von Betriebssystemen erhöhen.

![X86 Privilegien](/assets/rings.svg)

> [!TIP]
> In der Praxis ist das Segmentierungsmodell von x86 kompliziert und Prozessisolation war zu unflexibel. Das hat die Speicherverwaltung schwieriger gemacht und der Protected Mode hat sich erst im Zusammenspiel mit Paging durchgesetzt.

## i386

Der **Intel 80386** von 1985 markiert den nächsten Meilenstein und führt **Paging** ein, was den *Protected Mode* erst wirklich nützlich gemacht hat, und erweitert die *x86* Architektur auf `32 Bit` und somit den Adressraum auf `4 GB`. `32 Bit` x86-Architekturen werden oft als **x86_32**, **IA-32** oder **i386** bezeichnet.

Beim **Paging**, oder **Virtual Memory** arbeiten Userspace Programme mit virtuellen Adressen. Diese Virtuelle Adressen werden dann mittels der **Memory Management Unit (MMU)** in Physische Adressen umgerechnet. Das hat zwei signifikante Vorteile gegenüber Segmentierung. Da jeder Prozess einen eigenen Adressraum hat ist dieser Linear und der Prozess muss nicht mehr hin und her springen um zu seinem Speicher zu gelangen. Zudem sind die Adressräume isoliert. Jetzt kann also kein Prozess mehr auf Speicher von anderen zugreifen ohne dass das explizit vom Betriebssystem so arrangiert wurde.

> [!TIP]
> Paging wird meist vom Betriebssystem gemacht, nicht vom Bootloader. Im `64 Bit` Modus ist Segmentierung keine Option mehr, dann würde der Bootloader Paging so einstellen, dass alle Logischen Adressen 1:1 auf Physische Adressen zeigen. Das ist die Standardeinstellung im Kernelspace.

### A20 Gate

Die *x86* Architektur hatte `20 Bits` für den Adressbus dank Segmentation. Also Lines 0 bis 19. Da sich das BIOS ganz oben und Interrupts ganz unten befanden haben viele Entwickler aus Effizienzgründen absichtlich mit Integer Overflows gearbeitet. Bei einer erweiterung des Adressbusses auf `32 Bit` bedeutet das, dass 16 Bit Programme plötzlich keinen Overflow mehr bekommen, falsche Speicherbereiche adressieren, und nicht mehr funktionieren. Deshalb gibt es das `A20` Gate, welches Bits auf der Line 20 nicht zulässt, um einen Overflow zu forcieren. Wenn der Bootloader von 16 Bit auf 32 Bit wechselt, muss er die A20-Leitung freischalten, damit Adressen oberhalb von `1 MB` erreichbar sind.

## AMD64

Der **AMD Opteron** von 2003 ist der erste `64 Bit` Prozessor auf Basis der `x86` Architektur. Da die `64 Bit` Erweiterung auf AMD zurückgeht, wird sie als **AMD 64** bezeichnet. Allerdings wird sie bei Intel oft auch als **x86-64**, oder bei Betriebssystemen als **x64** bezeichnet.

Abgesehen vom erweiterten Adressbus auf `64 Bit` und somit theoretisch `16EB` Adressierbaren Speicherplatz. `16EB` sind eine riesige Menge, die aktuell niemand benötigt. Zugriff darauf würde die Systeme auch extrem verlangsamen und aufgrund der komplexeren Decodertechnologie wesentlich teurer machen. Aktuelle Rechner sind daher wesentlich begrenzter. Server sind für mehr ausgelegt als PCs, aber bei weitem nicht soviel wie theoretisch möglich wäre. Typischerweise verwenden aktuelle CPUs Adressbreiten von `48-57 Bits`.

AMD64 enthält einen **Compatibility Mode**, mit dem ein `64 Bit` Betriebssystem weiterhin `32 Bit` Anwendungen ausführen kann.

Zudem erzwingt AMD64 Paging.

## EFI/UEFI

Das **Extensible Firmware Interface (EFI)** war eine Firmware von Intel für den Itanium Prozessor mit dem Ziel das BIOS zu ersetzen um einen moderneren Bootprozess bereitzustellen. Kurz gesagt, es übernimmt viele Aufgaben von modernen Bootloadern. Das **Unified Extensible Firmware Interface** ist eine Standardisierung um Herstellerübergreifende Kompatibilität zu gewährleisten und hat das spezifische EFI abgelöst.

UEFIs sind komplex und passen oft nicht in den Speicherbereich der für die Firmware vorgesehen ist. Gemapped wird also zunächst nur der Startercode der dann das eigentliche UEFI lädt. Die Prozesse sind ähnlich zu modernen Bootloadern. Zudem können erweiterungen und Treiber als Module hinterlegt werden um das UEFI zu erweitern. Moderne UEFIs haben **Grafische Benutzeroberflächen**, **Grundlegende Netzwerkstacks**, oft **Browser** und unterstützen einfache Dateisysteme wie **FAT(12/16/32)** und **ISO 9660**.

Statt einem MBR erwarten UEFIs einen GPT, eine modernere Alternative. Im GPT suchen sie nach **EFI System Partitions (ESP)** die EFI Anwendungen (`.efi` binaries, meist Bootloader) bereithält. Je nach Konfiguration lädt das UEFI eine dieser Anwendungen. Dabei ist die CPU bereits in der Nativen Architektur. Also 32 Bit CPUs sind im Protected 32 modus und 64 Bit CPUs im 64 Bit Long Mode.

Das UEFI sucht nicht zufällig nach EFI Anwendungen. Im einfachsten Fall ist eine bevorzugte Anwendung im NVRAM vom UEFI gespeichert, dann läd es diese direkt. Alternativ sucht es auf allen Massenspeichermedien in konfigurierter Reihenfolge nach ESPs. Wurde ein ESP gefunden, wird die Anwendung unter `\EFI\BOOT\BOOT\x32.EFI` beziehungsweise `\EFI\BOOT\BOOT\x64.EFI` erwartet. Andere Anwendungen werden ignoriert.

> [!TIP]
> Ein UEFI kann zunächst keine Bootloader und Kernel laden die für BIOS geschrieben wurden. Allerdings gibt es in fast jedem UEFI die Option **Legacy Boot** oder ähnlich um ein BIOS zu emulieren und mit älterer Software kompatibel zu sein.

### Bootloader und UEFI

Das UEFI nimmt dem Bootloader viele Aufgaben ab, ersetzt es aber nicht komplett. Einer der bekanntesten Bootloader damals wie heute ist **GRUB**. GRUB (heute GRUB LEGACY) ist für BIOS (bzw. Legacy Boot) geschrieben und übernimmt den gesamten Bootprozess bis zu 32 Bit. Der moderne **GRUB2** ist für UEFI geschrieben und stellt ein Menü zur Betriebssystemwahl bereit, Timeouts, Default Kernel sowie komplexere konfigurationen wie Festplattenverschlüsselung, LVM und RAIDs mit denen UEFI nativ nicht ohne weiteres zurechtkommt. Viele Kernel können aber auch direkt von UEFI gebooted werden.

Beim Installieren von einem Bootloader wie GRUB registriert sich der Bootloader selbst als bevorzugte Anwendung im UEFI. GRUB liegt in der ESP meist unter `\EFI\BOOT\BOOTX86.EFI`

### Secureboot

Eine erweiterung in modernen UEFIs ist **Seruceboot**. Dabei werden im UEFI öffintliche kryptographische Schlüssel hinterlegt denen das System vertraut. Ein Boot findet nur dann statt, wenn die EFI Anwendung vom zugehörigen privaten Schlüssel signiert wurde. Das sorgt dafür, dass ein Angreifer nicht einfach die Anwendung verändern, und schadsoftware bereits im bootprozess einschleusen kann.