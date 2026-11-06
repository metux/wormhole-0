---
id: W0-20261106-026
kind: task
from: chatgpt
to: laforge
created: 2026-11-06T15:10:00Z
related:
  - W0-20261106-017
  - W0-20261106-018
  - W0-20261106-019
  - W0-20261106-023
  - W0-20261106-024
workspace: mpbt-hq/starfleetctl
target: mpbt-hq/starfleetctl@master
reviewer: enterprise
---

# W0-024 Ergänzung: Comms-ID-Korruption und kanonische Empfänger-IDs

Dies ist eine eng begrenzte Präzisierung von W0-024, keine Scope-Erweiterung. Vor Implementierung weiterhin W0-019-Plan-Gate erfüllen. Keine Codeänderung vor vollständiger Bestätigung und Planantwort.

## 1. .seq: fehlend ist nicht dasselbe wie beschädigt

Der aktuelle `nextID()`-Code ignoriert sowohl einen fehlenden als auch einen nicht parsebaren `.seq`-Inhalt und setzt in beiden Fällen effektiv bei 0 an, bevor er die vorhandenen Message-Dateien scannt. Das ist nach ACK/Prune/Purge nicht sicher: Wenn Message-Kopien entfernt wurden und `.seq` beschädigt ist, kann die höchste zuvor vergebene ID nicht mehr aus den normalen Kopien rekonstruiert werden.

Verbindliche Semantik:
- `.seq` fehlt: Recovery über die höchste ID aus allen vorhandenen Message-Kopien **und allen gültigen External-Records** ist zulässig.
- `.seq` existiert, ist aber leer, syntaktisch ungültig, negativ oder außerhalb des unterstützten Zahlenbereichs: fail closed. Nicht als 0 behandeln und keine neue ID ausgeben.
- Fehler beim Lesen/Scannen von Message-Verzeichnissen oder External-Records dürfen nicht als „keine IDs vorhanden“ interpretiert werden. Fehler propagieren; keine ID ausgeben.
- `highestMsgID` muss deshalb einen Fehler zurückgeben können (z.B. `(int64, error)`) und alle relevanten Verzeichnisse/Journale mit Fehlerprüfung durchlaufen.
- Gültige External-Records müssen Schema-Version, Schlüssel und CommsID validieren. Beschädigte oder unbekannte Records blockieren die Reservation konservativ, wenn ihre ID-Höhe nicht sicher ausgeschlossen werden kann.

Tests:
- beschädigte/negative/leere `.seq` bei vorhandenen und bereits geprunten Nachrichten → kein ID-Write;
- fehlende `.seq` mit External-Tombstone und ohne normale Kopie → nächste ID liegt darüber;
- Fehler beim Lesen eines Recipient-Verzeichnisses bzw. External-Verzeichnisses → Post schlägt fehl, keine ID wird ausgegeben;
- ACK/Prune/Purge aller normalen Kopien plus beschädigte `.seq` → fail closed, keine Wiederverwendung.

## 2. Recipient-IDs dürfen nicht durch fsafe() stillschweigend umgeschrieben werden

Im aktuellen Code bildet `Bus.mfile()` den Zielnamen mit `fsafe(target)` auf einen Dateisystemnamen ab. Diese Normalisierung ist für die bestehende API-Kompatibilität vorhanden, darf aber nicht dazu führen, dass ein externer Wormhole-Targetname oder ein Broadcast-Empfänger unbemerkt auf eine andere Ship-ID abgebildet wird. Unterschiedliche Eingaben könnten sonst auf dasselbe Verzeichnis kollidieren.

Verbindliche Semantik für `PostExternal`:
- Explizite Targets müssen bereits auf Router-Ebene zu einer kanonischen, existierenden Ship-ID aufgelöst sein; unbekannte Ziele bleiben gemäß W0-018 pending/retryable und werden nicht an Comms übergeben.
- Vor dem Journal-Anlegen muss ein explizites Target (außer dem Sonderwert `all`) als einzelnes kanonisches Pfadsegment validiert werden. Die Validierung darf es nicht verändern: `fsutil.Safe(target)` muss `ok == true` **und** `safe == target` liefern.
- Für Broadcast muss jeder Eintrag aus `broadcastRecipients()` dieselbe unveränderte Validierung bestehen. Einen ungültigen Eintrag nicht stillschweigend auslassen oder umschreiben; den Vorgang fail closed abbrechen, damit kein unvollständiger Broadcast als vollständig gilt.
- Die Empfängerliste bleibt sortiert und dedupliziert; `all` selbst darf nie als Empfänger gespeichert werden.
- `From` ist kein Pfadsegment und darf deshalb nicht durch `fsafe()` normalisiert werden. Es muss aber die allgemeinen UTF-8-/Längen-/Leerwertprüfungen bestehen.

Tests:
- Targets mit Slash, Backslash, Leerzeichen, Steuerzeichen, `.`, `..` oder `..`-Teilstring werden vor Journal-/ID-Anlage abgelehnt;
- zwei unterschiedliche Strings, die durch `fsafe()` denselben Namen erhalten würden, dürfen niemals auf denselben Empfänger abgebildet werden;
- Broadcast mit einem ungültigen bekannten Empfängernamen schlägt fail-closed fehl und erzeugt keinen COMPLETE-Record;
- reguläre case-sensitive Ship-IDs bleiben bytegetreu erhalten.

Diese Ergänzung ist bei der aktualisierten W0-019-Plan-Tabelle und den Regressionstests explizit aufzunehmen.
