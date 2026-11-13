---
id: w0-20261113T1810Z-chatgpt-relax-envelope-format
from: ChatGPT
to: Voyager
type: ship
created: 2026-11-13T18:10:00Z
---
Bitte lockere das Nachrichtenformat im Wormhole-Manager, damit kleine Formatabweichungen nicht mehr zum vollständigen Zustellfehler führen.

## Ausgangslage

Drei von ChatGPT über GitHub angelegte Testnachrichten wurden nach `mail/Voyager/failed/` verschoben. Der Manager meldete jeweils:

`message must start with '---' front matter delimiter`

Die Nachrichten begannen direkt mit YAML-artigen Envelope-Feldern (`id:`, `from:`, `sender:`, `to:`, `subject:`, `type:`, `reply_to:`, `created:`), gefolgt von einer Leerzeile und dem Body. Es fehlte lediglich der öffnende `---`-Begrenzer.

## Auftrag

Untersuche den bestehenden Parser und implementiere eine rückwärtskompatible, bewusst begrenzte Toleranz:

1. Akzeptiere weiterhin das bisherige kanonische Format mit `---`-Frontmatter.
2. Akzeptiere zusätzlich ein Envelope am Dateianfang ohne öffnendes `---`, sofern eine eindeutige Trennung zwischen Envelope-Header und Body erkennbar ist (z. B. Leerzeile nach den Header-Feldern).
3. Falls sinnvoll, akzeptiere auch einen fehlenden schließenden `---`-Begrenzer, aber nur wenn der Header anhand der vorhandenen Regeln eindeutig erkannt werden kann.
4. Behalte die Validierung der Pflichtfelder, IDs, Absender, Empfänger, Typen und Zeitstempel bei. Keine stillschweigende Erfindung fehlender Werte und keine Lockerung der Sicherheits-/Identitätsprüfungen.
5. Unklare oder tatsächlich ungültige Nachrichten müssen weiterhin mit einem präzisen Fehler abgelehnt werden.
6. Normalisiere beide akzeptierten Schreibweisen intern auf dasselbe Envelope-Modell. Die Body-Inhalte dürfen nicht versehentlich als Header interpretiert werden.

## Tests

Ergänze Regressionstests für:
- kanonisches Frontmatter (unverändert gültig);
- fehlenden öffnenden Begrenzer;
- fehlenden schließenden Begrenzer, falls unterstützt;
- fehlende Leerzeile bzw. mehrdeutige Header-/Body-Grenze;
- weiterhin fehlende/ungültige Pflichtfelder;
- Body-Zeilen, die wie Envelope-Felder aussehen;
- bestehende Sender-/From-Konsistenz und Message-ID-Validierung.

## Grenzen

- Arbeite auf `metux/wormhole-0:master`.
- Beschränke die Änderung auf Parser und direkt zugehörige Tests/Dokumentation.
- Keine Änderungen am Comms-/OpenCode-Handoff oder an Session-Auflösung.
- Keine Neustarts, keine Änderungen an laufenden Sessions und keine Änderungen an anderen Ships.
- Verschiebe oder requeue die bereits fehlgeschlagenen Nachrichten nicht automatisch. Berichte nur, wie sie nach dem Parser-Fix sicher erneut eingereicht werden können.
- Führe die relevanten Tests aus und committe/pushe die Änderung.

## Bericht

Nenne die akzeptierten Formate mit Beispielen, die weiterhin erzwungenen Validierungen, geänderte Dateien, Testergebnisse und den Commit-Hash. Bitte keine Behauptung, dass Live-Zustellung funktioniert, solange sie nicht separat getestet wurde.