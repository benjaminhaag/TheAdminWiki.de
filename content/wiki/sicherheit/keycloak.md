---
date: '2026-03-07T11:01:23Z'
draft: true
title: 'Keycloak'
image: '/images/keycloak.webp'
tags: ["sicherheit","keycloak", "identity and access management", "iam"]
---

[Identity und Access Management](./identity-and-access-management.md) ist ein zentraler Baustein moderner Software-Architekturen. Eine weit verbreitete Open-Source-Lösung dafür ist Keycloak. Die Plattform ermöglicht Single Sign-On, Benutzerverwaltung und sichere Authentifizierung für Anwendungen.

## Architektur

![Diagram of the Keycloak Architecture](/assets/keycloak_architecture.svg)

### Keycloak Server

Der **Keycloak Server** ist die zentrale Laufzeitkomponente der Plattform. Er fungiert als **Identity Provider** und übernimmt sämtliche Aufgaben rund um Authentifizierung, Benutzerverwaltung und Token-Ausstellung.

Zu seinen zentralen Aufgaben gehören:

 - Durchführung von Authentifizierungsprozessen mittels der **Authorization Engine**
 - Verwaltung von **Benutzern**, **Gruppen** und **Rollen**
 - Ausgabe und Signierung von Tokens
 - Bereitstellung von Login-Flows und Sicherheitsrichtlinien in der **Authentication Engine**
 - Integration externer Identity Provider
 - Anbindung externer Benutzerverzeichnisse

Der Server stellt eine Reihe standardisierter **Endpunkte** bereit, die von Clients genutzt werden. Diese umfassen insbesondere:

 - Authorization Endpoint
 - Token Endpoint
 - UserInfo Endpoint
 - Logout Endpoint

Diese Endpunkte bilden die Grundlage für die Authentifizierungsflows gemäß SAML, OpenID Connect und OAuth 2.0.

### Realm

Ein **Realm** ist die grundlegende organisatorische Einheit innerhalb von Keycloak. Er stellt eine logisch isolierte Sicherheitsdomäne dar, innerhalb derer sämtliche Identitäts- und Zugriffsinformationen verwaltet werden.

Ein Realm enthält typischerweise:

 - Benutzer
 - Rollen
 - Gruppen
 - Clients
 - Authentifizierungsflows
 - Identity Provider
 - Sicherheitseinstellungen

Realms sind vollständig voneinander getrennt. Benutzer, Tokens oder Konfigurationen eines Realms sind nicht in anderen Realms sichtbar oder nutzbar.

Dieses Konzept erlaubt beispielsweise:

 - Trennung verschiedener Anwendungen oder Plattformen
 - Multi-Tenant-Setups
 - getrennte Entwicklungs-, Test- und Produktionsumgebungen

> [!TIP]
> Der standardmäßig vorhandene Realm master dient primär der Administration des Keycloak-Systems selbst.

> [!TIP]
> Die meisten erstellen eher zu viele Realms als zu wenige. Ein Realm pro Abteilung macht zum Beispiel keinen Sinn, wenn jede Abteilung die selbe Anwendung nutzt. Dann hätte die Anwendung pro Abteilung einen Login Button. Auch pro Anwendung macht nur Sinn, wenn die Nutzer unabhängig voneinander sind. Ansonsten muss ein Nutzer wieder pro Anwendung einmal angelegt werden.
> Ein konkretes Beispiel für Realms sind Mitarbeiter und Kunden, wenn beide distinkte Sicherheitsdomänen darstellen und auf unterschiedliche Anwendungen zugreifen. Z.B. Kunden nutzen das Kundenportal, während Mitarbeiter die Supportoberfläche verwenden.

### Clients

Ein **Client** repräsentiert eine Anwendung oder einen Service, der Authentifizierung über Keycloak nutzt. Der Client möchte wissen wer der Nutzer ist (Authentifizierung) und vom Client die Berechtigung haben sich beim **Resource Server** informationen zu holen.

Typische Clients sind:

 - Webanwendungen
 - Mobile Anwendungen
 - Single Page Applications
 - Backend-Services
 - APIs

Clients werden innerhalb eines Realms registriert und konfigurieren, wie die Anwendung mit Keycloak interagiert. Clients kommunizieren durch den **Protocol Adapter** mit dem Keycloak Server.

### Resource Server

Der **Resource Server** stellt sensitive Daten bereit. Mittels **Policy Enforcer** redet er mit dem Keycloak Server um Autorisierungen und Policies durchzusetzen.

### Storage Adapter

Keycloak bietet zu Entwicklungs und Testzwecken eine integrierte H2 Datenbank. Der Storage Adapter sorgt aber für eine agnostische Schnittstelle, sodass andere Datenbanken, wie Postgres oder MySQL, anzubinden.

## Benutzerverwaltung

Innerhalb eines Realms verwaltet Keycloak Benutzeridentitäten. Ein Benutzer besitzt Attribute wie, welche über das **Profil** definiert werden. Typische Attribute sind:

 - Benutzername
 - E-Mail
 - Passwort oder andere Credentials
 - Rollen und Gruppen
 - benutzerdefinierte Attribute

Zusätzlich können weitere Sicherheitsmechanismen aktiviert werden, beispielsweise:
 - Multi-Factor Authentication
 - Passwort-Policies
 - Account-Locking
 - Required Actions (z. B. Passwortänderung beim ersten Login)

Benutzer können direkt in Keycloak gespeichert oder über externe Systeme wie andere Identity Provider und User Federation bereitgestellt werden.

### Gruppen

Gruppen sind eine organisatorische Struktur innerhalb eines Realms, die dazu dient, Benutzer logisch zu bündeln und Zugriffsrechte zentral zu verwalten. Sie ermöglichen es, Berechtigungen und Rollen nicht individuell jedem Benutzer zuzuweisen, sondern kollektiv auf eine Gruppe anzuwenden.

Jede Gruppe kann:

 - Mitglieder (Benutzer) enthalten
 - Rollen und Berechtigungen zugewiesen bekommen
 - Verschachtelte Untergruppen besitzen, um komplexe Organisationsstrukturen abzubilden

Die Mitgliedschaft in einer Gruppe vererbt automatisch alle der Gruppe zugewiesenen Rollen und Berechtigungen an die enthaltenen Benutzer. Das vereinfacht das Management von Zugriffsrechten erheblich, besonders in größeren Umgebungen mit vielen Benutzern und differenzierten Berechtigungsanforderungen.

Typische Einsatzszenarien für Gruppen sind:

 - Abbildung von Teams, Abteilungen oder Projektgruppen
 - Rollenzuweisungen für gemeinsame Verantwortlichkeiten
 - Steuerung von Zugriff auf Applikationen oder Ressourcen auf Basis der Gruppenzugehörigkeit

Durch die Kombination von Gruppen und Rollen kann Keycloak flexible und skalierbare Zugriffskonzepte realisieren, die sich an die organisatorischen Gegebenheiten von Unternehmen anpassen.

### User Federation
Die **User Federation** ermöglicht die Integration externer Benutzerverzeichnisse in Keycloak.
Ein häufiges Szenario ist die Anbindung eines Unternehmensverzeichnisses über **LDAP** oder **Active Directory**.

In diesem Modell bleibt das externe System die primäre Quelle für Benutzerinformationen. Keycloak fungiert lediglich als Authentifizierungs- und Token-Service.

Je nach Konfiguration kann Keycloak:

 - Benutzer dynamisch aus dem Verzeichnis lesen
 - Benutzer lokal synchronisieren
 - Authentifizierung an das externe System delegieren

Das erlaubt eine Integration in bestehende Identity-Infrastrukturen ohne Migration aller Benutzer in Keycloak.

### Identity Provider

Neben der User Federation unterstützt Keycloak auch die Integration externer Identity Provider (IdP).

Dabei authentifiziert sich der Benutzer nicht direkt bei Keycloak, sondern bei einem externen Anbieter. Keycloak übernimmt anschließend die Identität und erstellt eine eigene Session.

Unterstützt werden verschiedene Protokolle und Anbieter, darunter:

 - OpenID Connect
 - SAML 2.0

Typische externe Anbieter sind beispielsweise:

 - Google
 - Microsoft

Diese Integration ermöglicht unter anderem:

 - Social Login
 - Enterprise [Single Sign-On](./sso.md)
 - Föderation zwischen Organisationen

## Keycloak als Open-Source-Lösung {#keycloak-als-open-source-loesung}

Keycloak ist ein Open-Source-[Identity- und Access-Management-System (IAM)](./identity-and-access-management.md), das häufig für Single-Sign-On und Identity-Provider-Szenarien eingesetzt wird.

Ein wesentlicher Vorteil von Keycloak ist, dass es ohne Lizenzkosten betrieben werden kann. Als Community-getriebenes Open-Source-Projekt wird es kontinuierlich weiterentwickelt und kann vollständig selbst gehostet werden. Zudem lässt sich die Plattform durch Erweiterungen und Custom Provider stark anpassen.

Neben der Authentifizierung unterstützt Keycloak auch eine integrierte Authorization-Engine, mit der sich Zugriffsregeln zentral definieren lassen. Dadurch kann Keycloak nicht nur als Identity-Provider, sondern auch als Policy Decision Point für Zugriffskontrolle eingesetzt werden.

### Alternativen zu Keycloak

Es existieren zahlreiche Alternativen zu Keycloak. Viele dieser Lösungen konzentrieren sich primär auf **Authentifizierung** und **Token-Ausstellung**, während komplexere Autorisierungslogik häufig in Anwendungen oder APIs implementiert wird.

#### Open-Source Alternativen

Beispiele für Open-Source-Identity-Provider sind **ZITADEL** oder **authentik**. Diese Systeme sind oft leichtergewichtig und moderner aufgebaut, fokussieren sich jedoch stärker auf die Rolle als Identity-Provider.

#### Cloud-basierte Identity-Provider

Kommerzielle Plattformen wie **Auth0**, **Okta**, **Amazon Cognito** oder **Microsoft Entra ID** bieten Identity-Services vollständig aus der Cloud an. Dadurch entfällt der Betrieb eigener Infrastruktur, allerdings entstehen Lizenz- oder Nutzungsgebühren.

#### Developer-First-Lösungen

Tools wie **Logto** oder **SuperTokens** richten sich speziell an Entwickler moderner Web- und Microservice-Architekturen. Sie stellen Identity-Funktionen primär über APIs bereit und lassen sich leicht in bestehende Anwendungen integrieren.

### Red Hat Single Sign-On

Eine besondere Rolle spielt **Red Hat Single Sign-On (RH-SSO)**. Dabei handelt es sich um eine Enterprise-Distribution von Keycloak, die von Red Hat gepflegt und unterstützt wird.

RH-SSO basiert technisch auf Keycloak, wird jedoch von Red Hat stabilisiert, getestet und mit **kommerziellem Support** sowie **langfristigen Wartungszyklen** angeboten. Unternehmen, die auf Support-Verträge oder zertifizierte Integrationen angewiesen sind, setzen daher häufig auf diese Variante.

## Keycloak im Kontext des BSI-Grundschutzes

Innerhalb eines nach [BSI-Grundschutz](./bsi-grundschutz.md) zertifizierten oder ausgerichteten IT-Systems übernimmt Keycloak eine zentrale Rolle im Bereich [**Identitäts- und Zugriffsmanagement (IAM)**](./identity-and-access-management.md).

Die Verwendung von Keycloak unterstützt dabei die Umsetzung mehrerer BSI-Anforderungen, unter anderem:

 - **Zugangskontrolle**: Keycloak stellt sicher, dass nur autorisierte Benutzer Zugriff auf geschützte Anwendungen und Ressourcen erhalten.
 - **Authentifizierung und Autorisierung**: Durch standardisierte Verfahren (z. B. OpenID Connect, OAuth 2.0) gewährleistet Keycloak sichere und nachvollziehbare Login-Prozesse.
 - **Rollen- und Rechtemanagement**: Die granulare Zuweisung von Rollen und Gruppen entspricht den Vorgaben zur differenzierten Zugriffsbeschränkung.
 - **Protokollierung und Nachvollziehbarkeit**: Keycloak kann Login- und Token-Transaktionen protokollieren, was eine Überwachung und Nachvollziehbarkeit von Zugriffen ermöglicht.
 - **Integration in bestehende Sicherheitsinfrastrukturen**: Mit Funktionen wie User Federation und Identity Federation lässt sich Keycloak nahtlos an Verzeichnisdienste und externe Identity Provider anbinden, was den Betrieb in komplexen, sicherheitskritischen Umgebungen erleichtert.

Damit trägt Keycloak maßgeblich zur Einhaltung der Sicherheitsanforderungen des BSI-Grundschutzes bei und unterstützt Organisationen dabei, ihre IT-Landschaft sicher und regelkonform zu gestalten.