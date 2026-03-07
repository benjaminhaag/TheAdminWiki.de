---
date: '2026-03-06T10:58:10Z'
draft: false
title: 'Identity and Access Management'
image: "/images/iam.webp"
tags: ["sicherheit","identity and access management", "iam"]
---

Seit Anbeginn der Computer wächst die Anzahl an Anwendungen mit denen Nutzer täglich konfrontiert werden stetig. Die herkömmliche eine Identität pro Nutzer pro Anwendung sorgte für

 - Zu viele Identitäten für eine Person
 - Viele Passwörter
 - Viele Log In Prozesse
 - Viel Administrativer Aufwand
 - Verwaiste Accounts
 - Sicherheitsvorfälle
 - Verteilte Rechteverwaltung

Um das in den Griff zu bekommen, sorgt Identity und Access Management für einen zentralen, systematischen Ansatz.

## Definition

**Identity and Access Management (IAM)** sind **Konzepte**, **Prozesse** und **technische Systeme**, mit denen digitale Identitäten verwaltet und deren Zugriffsrechte auf Ressourcen kontrolliert werden.

IAM beantwortet vier zentrale Fragen:

 - Bist du es wirklich? *(Authentication)*
 - Was darfst du tun? *(Autorisation)*
 - Was hast du getan? *(Auditing)*

## Konzept

### Identität

Eine **Identität** ist eine **digitale Repräsentation** eines Akteurs.

Das kann sein:
 - ein Mensch *(Mitarbeiter, Kunde, Admin)*
 - ein Service *(Microservice, Cronjob)*
 - ein Gerät *(IoT, Server, Laptop)*

Eine Identität hat:
 - Attribute *(Name, E-Mail, Rolle, Abteilung, Status)*
 - einen Lebenszyklus *(Anlegen → Ändern → Deaktivieren → Löschen)*

### Authentifizierung

Ist der Akteur wirklich, wer er vorgibt zu sein?

Authentifizierung erfolgt mittels:
 - Etwas das ich wiess *(z.B. Passwort, PIN, Sicherheitsfrage)*
 - Etwas das ich habe *(z.B. Smartcard, Smartphone, Zertifikat, Token)*
 - Etwas das ich bin *(Biometrie, z.B. Retina, Fingerabdruck, Gesichtserkennung)*

> [!WARNING]
> Authentifizierung bestätigt nur die Identität und sagt **nichts** darüber aus was sie darf.

### Autorisierung

Ist der Akteur berechtigt zu tun, was er versucht?

Autorisierung erfolgt mittels:
 - MAC *(Mandatory Access Control)*
 - RBAC *(Role Based Access Control)*
 - ABAC *(Attribute Based Access Control)*
 - RuBAC *(Rule Based Access Control)*
 - DAC *(Discretionary Access Control)*

Die Autorisierungsmodelle können auch beliebig kombiniert werden.

#### Mandatory Access Control

Jeder Akteur *(Subject)* hat eine Sicherheitsstufe *(Clearance)* und jedes Objekt hat ein Sicherheitslabel *(Label)*. MAC prüft, ob die Clearance des Subjekts höher ist als der Label des Objekts.

MAC wird primär in militärischen Bereichen eingesetzt, zum Beispiel: VS-NfD. Rüstungsunternehmen und Dienstleister, die für das Militär arbeiten kommen zwangsläufig damit in Berührung. MAC bietet sich für die Umsetzung der Gesetze an.

| Stufe              | Gefahr bei Veröffentlichung                   |
| ------------------ | --------------------------------------------- |
| **VS-NfD**         | Kann öffentliches Interesse schädigen         |
| **VS-Vertraulich** | Kann öffentliches Interesse konkret schädigen |
| **Geheim**         | Kann öffintliche Sicherheit gefährden         |
| **Streng Geheim**  | Kann öffentliche Sicherheit schwer schädigen  |

Akteure bekommen Zugang zu VS-NfD Material nur, wenn sie Teil des Projektes sind, und die notwendige Sicherheitsüberprüfung, Schulung und Stufe haben.

#### Role Based Access Control

Akteure haben Rollen und Objekte fordern Rollen für Zugriffe. RBAC ist eines der Verbreitetsten Autorisierungsmodelle, da es einfach zu implementieren und relativ flexibel ist. Zum Beispiel In Unternehmen mit verschiedenen Departments

| Rolle                | Bewerberdaten | Mitarbeiterdaten |
| -------------------- | ------------- | ---------------- |
| **HR-Mitarbeiter**   | CRUD          | R                |
| **Developer**        | -             | R                |
| **IT-Administrator** | R             | CRUD             |

#### Attribute Based Access Control

Statt konkrete Rollen haben Akteure feingranulare Attribute, mit denen das System prüfen kann, ob der Akteur Zugang hat oder nicht. ABAC ist flexibler, aber aufwendiger als RBAC. Zum Beispiel: 

 > Nur Developer mit 2FA können Quellcode veröffentlichen.

ABAC kann auch RBAC abbilden, indem die Rolle dem Akteur als Attribut mitgegeben wird.

#### Rule Based Access Control

RuBAC ist das Flexibelste. Das System evaluiert Berechtigungen anhand programmierter Regeln. Zum Beispiel: 

 > Nur Developer mit 2FA können Quellcode veröffentlichen, falls es zwischen 8:00 Uhr morgens und 17 Uhr ist. Sollte die Veröffentlichung außerhalb dieser Zeit passieren, muss mindestens ein Vorgesetzter zustimmen.

RuBAC kann auch ABAC implementieren, damit auch RBAC und MAC.

#### Discretionary Access Control

DAC implementiert im Gegensatz zu den anderen Modellen keine Zentrale Rechteverwaltung, stattdessen hat jedes Objekt einen Eigentümer (Owner), der die Rechte anderer verwaltet. Die Berechtigungen sind also nicht in einer zentralen Datenbank in Form von Rollen, Regeln und Attributen, sondern am Objekt selbst.

DAC wird meist bei Dateisystemen genutzt. Der Eigentümer einer Datei in Linux kann Rechte für sich selbst, eine Gruppe und alle anderen Nutzer einstellen. Z.B. `rw-rw-r--`. In Windows‘ NTFS und SMB können Rechte sogar noch feingranularer pro Benutzer vergeben werden. 

|         | File 1      | File 2 |
| ------- | ----------- | ------ |
| *Alice* | Read, Write | Read   |
| *Bob*   | Read        | Read   |

#### Auditing

Accounting nimmt auf **Wer** hat **Wann** **Was** **Wo(mit)** gemacht. Im Auditing kann das dann eingesehen werden.

Neben der korrekten Durchsetzung von Authentifizierung und Autorisierung ist es auch wichtig zu wissen wer wann was wo(mit) gemacht hat um Richtigkeit und Compliance nachzuweisen (Auditing), oder im schlimmsten Fall helfen zu können eine Straftat aufzuklären.

## IAM in Modernen Systemen

Im Laufe der letzten Jahrzehnte hat sich die IT-Landschaft stark verändert und IAM eine zentrale Rolle zugeordnet.

### Compliance

Die zunehmende Anzahl an Gesetze fordert immer stärkere Maßnahmen und Kontrollen zur Absicherung aller Systeme. 

Ohne IAM als zentralen Mechanismus ist es nicht mehr möglich dem Nachzukommen.

### Remote Zugriffe

Remote Work und Cloud Plattformen sorgen dafür, dass es kein sicheres Intranet mehr gibt das vom Internet abgeschirmt ist. Stattdessen ist die IT fast aller Unternehmen über das Internet verteilt. IAM erleichtert die Administration und die Sicherheit trotz steigender Bedrohungspotentiale.

### Microservices und App Jungle

Microservices und die wachsende Anzahl an Apps, mit denen heutzutage jeder Mitarbeiter arbeitet, sorgen für exponentiell mehr Zugängen, die das IT-Department managen muss. IAM zentralisiert die Verwaltung und reduziert die Anzahl an Identitäten.
