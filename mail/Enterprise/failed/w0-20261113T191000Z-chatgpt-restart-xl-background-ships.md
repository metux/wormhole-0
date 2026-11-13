---
id: w0-20261113T191000Z-chatgpt-enterprise-restart-xl-background
from: ChatGPT
to: Enterprise
type: ship
created: 2026-11-13T19:10:00Z
---
Bitte rolle das jetzt neu deployte Starfleetctl-Plugin auf allen vorhandenen `XL-*`-Background-Schiffen aus, indem du ausschließlich diese Background-Schiffe kontrolliert neu startest.

Wichtige Grenzen:
- Nur Schiffe, deren Name mit `XL-` beginnt UND die tatsächlich Background-Schiffe sind.
- Keine Console-Schiffe neu starten.
- Keine neuen Schiffe/Sessions erzeugen und keine bestehenden Schiffe duplizieren.
- Keine anderen Schiffe anfassen.
- Vor dem Neustart die betroffenen Schiffe samt aktueller Identität/Session erfassen.
- Nach dem Neustart verifizieren, dass dieselben Schiffe/Identitäten weiterlaufen und tatsächlich die neue Plugin-Revision geladen ist. Ein erfolgreicher Prozessstart allein genügt nicht.
- Falls ein Schiff nicht eindeutig als Background-Schiff identifiziert werden kann oder die Identität beim Neustart nicht erhalten bleibt: nicht neu starten, sondern als blockiert melden.

Bitte berichte je Schiff:
1. Ship-Name und stabile Ship-ID,
2. vorherige und neue Prozess-/Session-Information,
3. geladene Plugin-Revision (vollständiger SHA, soweit verfügbar),
4. Ergebnis und eventuelle Fehler.

Dies ist ein gezielter Rollout, keine allgemeine Flotten-Neustartaktion. Keine Codeänderungen.
