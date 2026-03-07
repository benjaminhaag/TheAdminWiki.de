---
date: '2026-03-07T15:47:25Z'
draft: true
title: 'SSO'
tags: ["sicherheit","sso", "identity and access management", "iam"]
---

**Single Sign-On (SSO)** ist ein Authentifizierungsverfahren, bei dem ein Benutzer sich einmal anmeldet und anschließend Zugriff auf mehrere Systeme oder Anwendungen erhält, ohne sich erneut anmelden zu müssen. SSO verbessert sowohl die Benutzerfreundlichkeit als auch die Sicherheit, da weniger Passwörter verwaltet werden müssen.

In modernen IT-Umgebungen verwenden Benutzer zahlreiche Anwendungen und Systeme. Dies führt zu mehreren Herausforderungen:

 - **Viele Apps, viele Datenstände**: Informationen werden in unterschiedlichen Anwendungen unabhängig gespeichert, was Dateninkonsistenzen verursachen kann.
 - **Viele Login-Prozesse pro Benutzer**: Jeder Benutzer muss sich in mehreren Systemen anmelden, was Zeit kostet und die Benutzerfreundlichkeit beeinträchtigt.
 - **Sicherheitsprobleme**: Mehrere Passwörter erhöhen das Risiko von unsicheren Passwörtern, Passwort-Wiederverwendung oder Phishing-Angriffen.

**Single Sign-On (SSO)** als teil von [Identity und Access Management (IAM)](./identity-and-access-management.md) adressiert diese Probleme, indem es die Authentifizierung zentralisiert und vereinheitlicht. So müssen Administratoren nurnoch eine Identität verwalten und Nutzer benötigen nurnoch einen Log-In Prozess und haben Zugang zu allen Anwendungen.

## SSO im Web

Im Webbereich werden SSO-Mechanismen hauptsächlich durch die Protokolle [OAuth2, bzw OpenID Connect](oauth2_openid_connect.md) und [SAML](oauth2_openid_connect.md#saml) umgesetzt. Das Problem dabei ist, dass diese Protokolle Webbasiert sind und nur begrenzt für native Systeme wie Betriebssysteme, klassische Desktopanwendungen, WiFi und co geeignet sind.

## SSO in Systemen

SSO für native Systeme, Betriebssysteme und Unternehmensanwendungen unterscheidet sich grundlegend von Web-SSO. Während Web-SSO auf Protokollen wie OAuth2, OIDC oder SAML basiert, greifen native Systeme meist auf Verzeichnisdienste und Netzwerkprotokolle zurück. Die wichtigsten Technologien sind **Lightweight Directory Access Protocol (LDAP)**, **Kerberos** und **Active Directory (AD)**.

### LDAP

### Kerberos

### Active Directory

**Active Directory (AD)** ist Microsofts zentraler Verzeichnisdienst, der LDAP, Kerberos und zusätzliche Dienste kombiniert. Mittlerweile gibt es auch open source ienste von Drittanbietern, wie zum Beispiel **Samba 4 Active Directory**.

### NTLM

### GSSAPI

### SPNEGO

## Hybridlösungen {#hybridloesungen}

Native Systeme für SSO funktionieren gut innerhalb betriebssystemnaher Umgebungen. Über einen Browser ist die Nutzung solcher Protokolle zwar mittels SPNEGO möglich, aber deutlich schwerer und funktioniert nur wirklich gut bei homogenen Ökosystemen wie reine Microsoft Lösungen, zum Beispiel Office 365 im Browser. Allerdings bietet Microsoft bei weitem nicht für alles eine Lösung, und oft reichen diese dann nichtmal aus. 

Die meisten Unternehmen kombinieren beide Ansätze und nutzen Identity Provider wie Keycloak zur Anbindung von Webanwendungen und AD zur Anbindung betriebssystemnaher Software. 

> [!TIP]
> Keycloak lässt sich zwar theoretisch mittels GSSAPI und SPNEGO an ein AD anbinden sodass Nutzer mit dem Betriebssystem Login sofort auch bei Keycloak authentifiziert sind, allerdings ist das oft ein administrativer Albtraum, weshalb viele Unternehmen zwei Logins voraussetzen. Einmal im Betriebssystem und einmal im Browser.