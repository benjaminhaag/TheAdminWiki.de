---
date: "2026-02-02T14:14:23Z"
draft: false
name: "Github Pages"
---

Kostenlos für Öffentliche Repositories (mit Quota)

1. .github/workflows/....yml für Autodeploy anlegen.

Settings (Repository) -> Pages -> Build and deploy -> Source -> `GitHub Actions`
browse all workflows -> ... (Workflow auswählen, anpassenm committen)

Unter Actions sollte der Job jetzt laufen und dann eine Page erstellen.

2. Domain Anlegen (optional)

! DNS wird heftig gecached und Änderungen können sehr lange dauern bis sie von allen Systemen gesehen werden.

Standardmäßig läuft die Webseite jetzt unter einer subdomain mit subfolder. Das macht absolute pfade kaputt und kann deshalb kaputt aussehen (CSS und Bilder werden nicht geladen). Lösung: eigene (sub-)domain.

Settings (Profile) -> Pages -> Verified domains -> Add a domain -> Schritte Folgen

DNS Einträge:

CNAME www username.github.io
A und AAA mit IPs von der [Anleitung](https://docs.github.com/en/pages/configuring-a-custom-domain-for-your-github-pages-site/managing-a-custom-domain-for-your-github-pages-site#configuring-an-apex-domain)

Settings (Repository) -> Pages -> Custom domain -> Eintragen

! Bis dann alles propagiert ist und funktioniert kann etwas Zeit vergehen.
