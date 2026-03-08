---
date: '2026-03-07T15:53:22Z'
draft: false
title: 'OAuth2 - OpenID Connect - SAML'
image: '/images/ocs.webp'
tags: ["sicherheit","oauth2 openid connect saml", "oauth2", "openid connect", "oidc", "saml", "sso"]
---

Single Sign-On (SSO) beschreibt ein Authentifizierungsverfahren, bei dem sich ein Benutzer einmal anmeldet und anschließend auf mehrere Dienste zugreifen kann, ohne sich erneut anmelden zu müssen. Moderne, Web-basierte Implementierungen basieren meist auf **OAuth2**, **OpenID Connect (OIDC)** oder **SAML**.

Während OAuth2 primär Autorisierung bereitstellt, erweitert OpenID Connect OAuth2 um Authentifizierung. SAML ist zur reinen Authentifizierung.

| Feature | OAuth2 | OpenID Connect | SAML |
|---|---|---|---|
| Zweck | Autorisierung | Authentifizierung + Autorisierung | Authentifizierung |
| Basis | HTTP + JSON | OAuth2 Erweiterung | XML |
| Token | Access Token | Access Token + ID Token | Security Assertions |
| Token Format | meist JWT | JWT | XML Assertion |
| Typischer Einsatz | API Zugriff | Web Login / SSO | Enterprise SSO |
| Redirect Login | optional | üblich | üblich |
| Moderne Nutzung | sehr verbreitet | Standard für Web SSO | eher Legacy |
| Mobile / SPA geeignet | ja | ja | eher ungeeignet |

## OAuth2

OAuth2 ist ein **Autorisierungsframework**, das Anwendungen erlaubt im Namen eines Benutzers auf Ressourcen zuzugreifen, ohne dessen Passwort zu kennen.

Der Zugriff erfolgt über **Tokens**, die vom **Identity Provider (Authorization Server)** ausgestellt werden.

OAuth2 definiert vier zentrale Rollen:

 - **Resource Owner** – der Benutzer, dem die Ressourcen gehören  
 - **Client** – die Anwendung, die Zugriff auf Ressourcen möchte  
 - **Authorization Server / Identity Provider (IdP)** – authentifiziert Benutzer und stellt Tokens aus  
 - **Resource Server** – stellt geschützte Ressourcen bereit

Der Client fordert Token an um auf die Resourcen zuzugreifen die er benötigt. Anschließend autorisiert der Resource Owner den Zugriff und der IdP stellt zwei Token aus:

 - **access_token** - kann zeitlich begrenzt genutzt werden um sich dem Resource Server gegenüber zu autorisieren
 - **refresh_token** - kann genutzt werden um neue access token anzufordern, da diese aus Sicherheitsgründen schnell ablaufen

Der Zugriff auf APIs erfolgt typischerweise über ein **Bearer Token**:

```
Authorization: Bearer <access_token>
```

Für die Autorisierung sieht OAuth2 folgende Flows (Abläufe) vor:

### Direct Grant

Der **Direct Grant** (Resource Owner Password Grant) erlaubt dem Client, Benutzername und Passwort direkt an den Identity Provider zu senden. Anschließend stellt der IdP **access_token** und **refresh_token** aus. Dazu muss der Resource Owner sein Passwort mit dem Client teilen, weshalb dieser Flow zwar sehr einfach ist, aber als veraltet und unsicher gilt.

### Implicit Flow

Der **Implicit Flow** wurde für reine Browser-Anwendungen entwickelt.

Der Client leitet den Benutzer zum Identity Provider weiter. Nach erfolgreicher Anmeldung wird der Benutzer zurück zum Client umgeleitet und erhält das Token direkt über die URL.

```
https://client.example/#access_token=...
```

Das Problem dabie ist, dass Tokens über den Browser gehen und leicht abgefangen werden können. Daher gilt der Implicit Flow heute als **veraltet**.

### Standard Flow

Der **Authorization Code Flow (Standard Flow)** ist heute der empfohlene OAuth2 Flow für Web-Anwendungen.

Ablauf:

 1. Client leitet Benutzer zum Identity Provider weiter
 2. Benutzer authentifiziert sich
 3. IdP redirectet zurück zum Client mit einem **Authorization Code**
 4. Client tauscht den Code und ein Client Secret gegen Tokens

Da der Authorization Code single use ist und nur mit einem Client Secret eingetauscht werden kann, kann ein Angreifer auch dann keine Token bekommen, wenn er vollen Zugriff auf den Browser hat.

> [!WARNING]
> Wenn der Angreifer vollen Zugriff auf den Browser hat kann er zwar keinen Token abgreifen, potentiell aber sehr wohl Username und Passwort bei der Anmeldung beim Identity Provider.

### PKCE

**Proof Key for Code Exchange (PKCE)** erweitert den Authorization Code Flow und wird vor allem für **Public Clients** verwendet (z. B. Mobile Apps oder SPAs). Public Clients können von jedem eingesehen werden und deshalb kein Client Secret geheim halten. Das sorgt dafür, dass jeder den Authorization Code gegen einen Token eintauschen kann und kompromitiert die Sicherheit des Standard Flow. PCKE lößt das Problem bestmöglich.

Statt einem statischen Client Secret generiert der Client einen zufälligen verifier und eine dazugehörige Challenge

```
verifier := random(32)
challenge := base64url(sha256(verifier))
```

Beim Legin wird die Challenge mitgesendet. Beim Token Request mit Authorization Code sendet der Client dann den Verifier mit. Die Challenge wird per GET übermitteln, ist also anfällig. Da die challenge gehashed ist, kann der IdP den Client später verifizieren, ein Angreifer aber keine Token anfragen, da er den verifier nicht kennt. Der Token Request ist ein POST und gilt somit als sicherer, da der verifier nicht mehr in Logfiles auftritt. So kann sich der Client sicher(er) dem IdP gegenüber authentifizieren trotz fehlendem Client Secret.

### DPoP

**Demonstrating Proof of Possession (DPoP)** schützt Access Tokens davor, von anderen Clients missbraucht zu werden.

Der Client generiert ein Keypair und signiert alle Anfragen mit seinem Private Key. Der IdP kann jetzt den Access Token an die Bedingung knüpfen, dass die Anfrage signiert ist. Dieses verknüpften Token nennt man DPoP bound Token.

Der Client signiert jeden Request mit seinem **Private Key** und sendet einen `DPoP` Header. Der Resource Server prüft, ob der Fingerprint des Keys mit dem des Token übereinstimmt.

### Service Account Roles

Dieser Flow wird verwendet, wenn **kein Benutzer beteiligt ist**, z. B.:

 - Backend → API
 - Microservice → Microservice

Der Client authentifiziert sich direkt beim Identity Provider mittels Client ID und Client Secret. Der IdP liefert ein Access Token, das anschließend für API Requests verwendet werden kann. Dabei kann der Client nur auf Resourcen zugreifen, auf die er selbst zugreifen darf ohne einen Nutzer um erlaubnis zu bitten.

### Device Authentication

Der **Device Flow** wird für "Geräte mit eingeschränkter Eingabemöglichkeit" verwendet. Das bedeutet fast immer, dass das gerät nicht über einen Browser besitzt und eine Weiterleitung nicht möglich ist.

Beispiele:

 - Smart TVs
 - CLI Tools
 - IoT Geräte

Ablauf:

1. Gerät fordert `device_code` und `user_code` beim IdP an
2. Benutzer erhält einen `user_code`
3. Benutzer öffnet eine Login-Seite auf einem anderen Gerät
4. Benutzer gibt den User Code ein
5. Gerät fragt regelmäßig `/token` ab ob die Autorisierung abgeschlossen ist

### CIBA

**Client Initiated Backchannel Authentication (CIBA)** erlaubt ebenfalls eine Autorisierung ohne Browser Redirect, im Gegensatz zur Device Authentication muss der Nutzer aber keine Autorisierung anfordern, sondern der Client initiiert die Anfrage.

Typisches Beispiel:

 - Login wird über eine **Banking-App bestätigt**
 - Anmeldung bei Adobe via App

 1. Client startet Authentifizierung mit `login_hint` (welcher Nutzer)
 2. IdP sendet Anfrage an Nutzer über einen Backchannel
 3. Benutzer bestätigt Login auf seinem Gerät
 4. Client fragt regelmäßig `/token` ab ob die Autorisierung abgeschlossen ist

## OpenID Connect

**OpenID Connect (OIDC)** erweitert OAuth2 um eine standardisierte **Authentifizierungsschicht**.

Zusätzlich zu OAuth Tokens wird ein **ID Token** ausgestellt.

Der IdP liefert:

 - access_token
 - refresh_token
 - **id_token**

Das **ID Token** ist ein **JWT**, das Informationen über den Benutzer enthält und an den Client als Audience gebunden ist.

 > [!TIP]
 > OAuth2 kann theoretisch zur Authentifizierung genutzt werden, da sich der Resource Owner authentifizieren muss und der Access Token genutzt werden kann um User Details vom IdP abzufragen. Allerdings kann der Client dann den access_token nutzen um sich fälschlicherweise dem Resource Server gegenüber als Nutzer auszugeben und somit mehr berechtigungen erhalten als er sollte. Der Identity Token von OIDC ist an den anfragenden Client gebunden und kann daher nicht misbraucht werden. Deshalb sollten die Rollen der beiden Protokolle respektiert werden.

### Opaque Token

Statt JWT können auch **Opaque Tokens** als **Access Token** und **Refresh Token** verwendet werden. Statt sinnvollem JSON enthalten diese nur zufällige Zeichenfolgen und müssen vom Resource Server erst beim Identity Provider gegen sinnvolle Informationen eingetauscht werden.

## SAML

**Security Assertion Markup Language (SAML)** funktioniert vom Prinzip her ähnlich wie der [Implicit Flow](#implicit-flow) von [OpenID Connect](#openid-connect) ohne zusätzlicher Authorization, also ohne _Access Token_ und ohne _Refresh Token_ von OAuth2. Das liegt daran, dass SAML ein reines Authentifizierungsprotokoll ist und nicht zur Autorizierung gegenüber dritten Systemen gedacht.

Wie der Name schon vermuten lässt, basiert SAML auf **XML** statt JSON und nutzt **Security Assertions** statt JWT. XML hat jedoch das problem, dass es keine standardisierte form gibt, externe resourcen referenziert werden können, und Implementierungen stärker aufeinander abgestimmt werden müssen. Daher wird SAML vorwiegend in legacy Anwendungen benutzt und ist größtenteils durch OIDC ersetzt worden.

### Rollen

Da SAML ein reines Authentifizierungsprotokoll ist gibt es keinen _Resource Server_. Rollen in SAML sind:

 - **Identity Provider (IdP)** - analog zum IdP in OAuth2/OIDC
 - **Service Provider** - analog zum Client in OAuth2/OIDC

### Security Assertions

Security Assertions sind lediglich signierte XML Dokumente und funktionieren somit technisch analog genau wie JWT.

```
<saml:Assertion>
    <saml:Subject>
        alice@wonderland.de
    </saml:Subject>
    <saml:AttributeStatement>
        <Attribute Name="email">alice@wonderland.de</Attribute>
        <Attribute Name="Name">Kingsleigh</Attribute>
        <Attribute Name="Vorname">Alice</Attribute>
        <Attribute Name="role">administrator</Attribute>
        <Attribute Name="role">IT Department</Attribute>
    </saml:AttributeStatement>
</saml:Assertion>
```