---
layout: post
title: 'TYPO3 Sentry Integration: Testing and Debugging with cURL'
date: '2024-0x-xxTxx:xx:xx.000+02:00'
author: Torben Hansen
tags:
- sentry
- typo3
- debugging
modified_time: '2024-01-xxTxx:xx:xx.000+02:00'
permalink: /2024/0x/todo.html
---

Nachteile aktuell
- dev-dependencies in composer.json
- Unnötige Plugins in composer.json
- Einträge (`app-dir` und `web-dir`) in `extra`
- Einträge (`vendor-dir` und `bin-dir`) in `config` 
- Manchmal auch manuelle Symlink-Erzeugung in `post-autoload-dump`
- Für mehrere TYPO3 Versionen pflegt man einfach mehrere composer.jsons in unterschiedlichen verzeichennissen

Vorteile neu:
- Extension wird direkt als Composer Dependency required
- Es ist möglich mehrere Verzeichnisse mit einer composer.json pro TYPO3 version zu haben
- Keine Anpassungen zur Build Laufzeit an composer.json und verschiedene Dependencies zu requiren
- Die composer.json ist aufgeräumter (keine extra config oder web-dir)
- Kein symlinken der Extension für Build
- Keine var/ Verzeichnis nach Test-Ausführung im Projektroot
