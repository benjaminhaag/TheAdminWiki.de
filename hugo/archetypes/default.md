{{- $path := .Path | strings.TrimSuffix ".md" -}}
---
date: '{{ .Date }}'
draft: true
title: '{{ replace .File.ContentBaseName "-" " " | title }}'
tags: {{ after 1 (strings.Split $path "/") | jsonify }}
---