---
date: '2026-04-13T07:13:52Z'
draft: true
title: 'Boot Process'
image: '/images/boot.webp'
tags: ["hardware","boot_process"]
---

Der **Bootprozess** ist ein festgelegter Ablauf, den die CPUs beim Start eines Computers durchläuft, um das System betriebsbereit zu machen. Dieses Wissen ist besonders hilfreich für die Fehlerdiagnose während des Startvorgangs.

Vor allem **AMD** und **Intel** achten bei ihrer **x86**-Architektur stark auf Rückwärtskompatibilität. Das bedeutet, dass Programme die für den allerersten x86 Prozessor aus 1978 geschrieben wurden immernoch auf aktuellen Prozessoren laufen. Um den Vorgang vollständig zu verstehen, lohnt sich daher auch ein Blick in die Vergangenheit.

Andere Architekturen wie **ARM** und **RISC V** sind meist deutlich schlanker, da sie nicht mit den Altlasten von x86 belastet sind. Allerdings funktionieren sie im Wesentlichen sehr ähnlich. Da **ARM** und **RISC V** allerdings nur Spezifikationen darstellen, hängt der genaue Bootprozess stark von der jeweiligen CPU-Implementierung ab. Deshalb betrachten wir den Ablauf exemplarisch anhand von x86 genauer.