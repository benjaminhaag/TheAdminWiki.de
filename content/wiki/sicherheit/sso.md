---
date: '2026-03-07T15:47:25Z'
draft: false
title: 'SSO'
image: '/images/sso.webp'
tags: ["sicherheit","sso", "identity and access management", "iam"]
---

**Single Sign-On (SSO)** ist ein Authentifizierungsverfahren, bei dem ein Benutzer sich einmal anmeldet und anschließend Zugriff auf mehrere Systeme oder Anwendungen erhält, ohne sich erneut anmelden zu müssen. SSO verbessert sowohl die Benutzerfreundlichkeit als auch die Sicherheit, da weniger Passwörter verwaltet werden müssen.

In modernen IT-Umgebungen verwenden Benutzer zahlreiche Anwendungen und Systeme. Dies führt zu mehreren Herausforderungen:

 - **Viele Apps, viele Datenstände**: Informationen werden in unterschiedlichen Anwendungen unabhängig gespeichert, was Dateninkonsistenzen verursachen kann.
 - **Viele Login-Prozesse pro Benutzer**: Jeder Benutzer muss sich in mehreren Systemen anmelden, was Zeit kostet und die Benutzerfreundlichkeit beeinträchtigt.
 - **Sicherheitsprobleme**: Mehrere Passwörter erhöhen das Risiko von unsicheren Passwörtern, Passwort-Wiederverwendung oder Phishing-Angriffen.

**Single Sign-On (SSO)** als teil von [Identity und Access Management (IAM)](./identity-and-access-management.md) adressiert diese Probleme, indem es die Authentifizierung zentralisiert und vereinheitlicht. So müssen Administratoren nurnoch eine Identität verwalten und Nutzer benötigen nurnoch einen Log-In Prozess und haben Zugang zu allen Anwendungen.

## Wie SSO funktioniert

SSO basiert darauf, dass ein zentrales System die Authentifizierung übernimmt und andere Anwendungen diesem System vertrauen.

Typischerweise gibt es drei beteiligte Komponenten:

 - Benutzer
 - Anwendung (Client / Service Provider)
 - Identity Provider (IdP)

Der Ablauf sieht vereinfacht so aus:

 1. Der Benutzer greift auf eine Anwendung zu.
 2. Die Anwendung erkennt, dass der Benutzer nicht angemeldet ist.
 3. Der Benutzer wird zum Identity Provider weitergeleitet.
 4. Dort authentifiziert sich der Benutzer (Passwort, MFA, Smartcard etc.).
 5. Der Identity Provider bestätigt der Anwendung die Identität des Benutzers.
 6. Der Benutzer erhält Zugriff auf die Anwendung.

Greift der Benutzer anschließend auf eine weitere Anwendung zu, erkennt der Identity Provider, dass bereits eine aktive Sitzung existiert. Die zweite Anwendung erhält sofort eine Bestätigung der Identität, ohne dass sich der Benutzer erneut anmelden muss.

## SSO im Web

Im Webbereich werden SSO-Mechanismen hauptsächlich durch die Protokolle [OAuth2, bzw OpenID Connect](oauth2_openid_connect.md) und [SAML](oauth2_openid_connect.md#saml) umgesetzt.

Diese Protokolle funktionieren in der Regel über HTTP-Redirects und Tokens zwischen Browser, Anwendung und Identity Provider.

Typische Eigenschaften von Web-SSO:

 - Browserbasierte Weiterleitungen
 - Token-basierte Authentifizierung
 - Föderation zwischen unterschiedlichen Organisationen möglich
 - Gut geeignet für Webanwendungen, APIs und Cloud-Services

Ein Beispiel für einen Web-SSO Flow:

 1. Benutzer öffnet eine Webanwendung.
 2. Die Anwendung leitet den Benutzer zum Identity Provider weiter.
 3. Der Benutzer meldet sich dort an.
 4. Der Identity Provider stellt ein Token oder eine Assertion aus.
 5. Die Anwendung akzeptiert diese und erstellt eine lokale Session.

Dieses Verfahren wird heute von vielen Cloud-Diensten und Plattformen verwendet.

Das Problem dabei ist, dass diese Protokolle Webbasiert sind und nur begrenzt für native Systeme wie Betriebssysteme, klassische Desktopanwendungen, WiFi und co geeignet sind.

## SSO in Systemen

SSO für native Systeme, Betriebssysteme und Unternehmensanwendungen unterscheidet sich grundlegend von Web-SSO. Während Web-SSO auf Protokollen wie OAuth2, OIDC oder SAML basiert, greifen native Systeme meist auf Verzeichnisdienste und Netzwerkprotokolle zurück. 

Die wichtigsten Technologien sind 
 - **Lightweight Directory Access Protocol (LDAP)**
 - **Kerberos**
 - **Active Directory (AD)**

Diese Technologien werden vor allem in Unternehmensnetzwerken eingesetzt, um Benutzer zentral zu authentifizieren und zu autorisieren.

### LDAP

**Lightweight Directory Access Protocol (LDAP)** ist ein Protokoll zum Zugriff auf Verzeichnisdienste.

Ein Verzeichnisdienst speichert Informationen über **Benutzer**, **Gruppen**, **Computer** und andere **Ressourcen** in einer **hierarchischen Struktur**.

LDAP selbst stellt jedoch kein vollständiges SSO-Protokoll dar. Es dient hauptsächlich dazu:

 - Benutzerinformationen zu speichern
 - Benutzer zu authentifizieren
 - Gruppen und Berechtigungen zu verwalten

Viele Anwendungen nutzen LDAP, um Benutzer direkt gegen einen zentralen Verzeichnisdienst zu authentifizieren. Dabei enthält der Verzeichnusdienst die UserID und das passwort zur authentifizierung, sowie optional Gruppen und Rollen zur späteren Autorisierung durch das system selbst.

### Kerberos

Kerberos ist ein Netzwerkprotokoll für sichere Authentifizierung.

Es basiert auf einem Ticket-System:

 - Benutzer meldet sich einmal am Netzwerk an.
 - Ein Key Distribution Center (KDC) stellt ein Ticket aus.
 - Dieses Ticket kann anschließend verwendet werden, um sich bei weiteren Diensten zu authentifizieren.

Der Vorteil von Kerberos ist, dass Passwörter nicht ständig über das Netzwerk übertragen werden müssen wie bei LDAP. Zudem ist das Ticket für alle Anwendungen gültig und der Nutzer muss sich nur einmal anmelden (SSO). So stellt Kerberos die Grundlage für viele SSO-Mechanismen in Unternehmensnetzwerken dar.

> [!TIP]
> Kerberos kann unabhängig betrieben werden, wird aber oft in Tandem mit LDAP genutzt. Kerberos führt die Authentifizierung durch und stellt Tickets aus, während LDAP weitere Informationen über die Nutzer liefert wie z.B. Gruppen und Rollen.

### Active Directory

**Active Directory (AD)** ist Microsofts zentraler Verzeichnisdienst, der LDAP, Kerberos und zusätzliche Dienste kombiniert. Mittlerweile gibt es auch open source ienste von Drittanbietern, wie zum Beispiel **Samba 4 Active Directory**.

### NTLM

**New Technology LAN Manager (NTLM)** ist ein älteres Authentifizierungsprotokoll aus der Windows-Welt.

Es wird heute größtenteils durch Kerberos ersetzt, kommt aber weiterhin in bestimmten Szenarien vor, zum Beispiel:

 - ältere Anwendungen
 - Workgroup-Umgebungen
 - Fallback-Authentifizierung

NTLM gilt als weniger sicher als moderne Kerberos-basierte Verfahren.

### GSSAPI

**Generic Security Services API (GSSAPI)** ist eine standardisierte Schnittstelle, über die Anwendungen Authentifizierungsmechanismen wie Kerberos und NTLM verwenden können, ohne sich auf eine bestimmte Technologie festzulegen.

Anwendungen müssen dadurch nicht direkt mit Kerberos oder anderen Protokollen implementiert werden, sondern können über GSSAPI verschiedene Authentifizierungsmechanismen nutzen.

### SPNEGO

**Simple and Protected GSSAPI Negotiation Mechanism (SPNEGO)** ist ein Mechanismus zur Aushandlung eines Authentifizierungsprotokolls zwischen Client und Server (meist via HTTP).

Im Web wird SPNEGO häufig verwendet, um Kerberos-basierte SSO-Authentifizierung im Browser zu ermöglichen. Dabei einigen sich Server und Client auf die Technologie (meist Kerberos) und können dann tickets austauschen.

Dies wird beispielsweise in Unternehmensnetzwerken genutzt, damit Benutzer automatisch im Browser angemeldet sind, wenn sie bereits am Betriebssystem angemeldet sind. SPNEGO gibt das ticket an den Webserver weiter, da dieser sich das nicht direkt vom PC holen kann wie native Anwendungen.

## Hybridlösungen {#hybridloesungen}

Native Systeme für SSO funktionieren gut innerhalb betriebssystemnaher Umgebungen. Über einen Browser ist die Nutzung solcher Protokolle zwar mittels SPNEGO möglich, aber deutlich schwerer und funktioniert nur wirklich gut bei homogenen Ökosystemen wie reine Microsoft Lösungen, zum Beispiel Office 365 im Browser. Allerdings bietet Microsoft bei weitem nicht für alles eine Lösung, und oft reichen diese dann nichtmal aus. 

Die meisten Unternehmen kombinieren beide Ansätze und nutzen Identity Provider wie Keycloak zur Anbindung von Webanwendungen und AD zur Anbindung betriebssystemnaher Software. 

> [!TIP]
> Keycloak lässt sich zwar theoretisch mittels GSSAPI und SPNEGO an ein AD anbinden sodass Nutzer mit dem Betriebssystem Login sofort auch bei Keycloak authentifiziert sind, allerdings ist das oft ein administrativer Albtraum, weshalb viele Unternehmen zwei Logins voraussetzen. Einmal im Betriebssystem und einmal im Browser.