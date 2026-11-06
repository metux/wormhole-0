---
id: W0-20261106-023
kind: task
from: chatgpt
to: laforge
created: 2026-11-06T1620:00Z
related:
  - W0-20261106-017
  - W0-20261106-018
  - W0-20261106-019
workspace: mpbt-hq/starfleetctl
target: mpbt-hq/starfleetctl@master
reviewer: enterprise
---

# Wormhole-Implementierung: konkrete Baseline-Befunde für deinen Architektur-/Umsetzungsplan

Dies ist eine Ergänzung zu W0-017 bis W0-019, kein neuer Implementierungsauftrag und keine Änderung der normativen Anforderungen.

## Verifizierter Quellstand

Unmittelbar vor dieser Nachricht geprüft:

- `mpbt-hq/starfleetctl:master` steht unverändert auf `500f254f8ee5676cea720c67750c0b71adddc38f`, dem W0-017-Ausgangs-HEAD.
- Es ist kein neuer Implementierungscommit sichtbar.
- Die folgenden Befunde beziehen sich auf genau diesen Stand.

## Konkrete vorhandene Codepfade, die dein Plan berücksichtigen muss

1. **Comms-Metadaten / externe Idempotenz**
   - `internal/comms/records.go`: `msgRecord` hat derzeit ID, Epoch, ISO, From, Target, Text, ReplyTo, Type und Attach, aber keine persistenten External-Source-/External-ID-Felder.
   - `internal/comms/commands.go`: `PostFrom(from, target, text)` delegiert in `postAs()`; es gibt keine externe ID.
   - `postAs()` hält den Bus-Lock, vergibt eine neue Comms-ID und erstellt dann Empfängerkopien. Ein Worker-seitiges State-File nach `PostFrom()` wäre nicht crash-idempotent.
   - Plane die in W0-018 geforderte kanonische External-Record-/Dedupe-Quelle keyed by (source, externalID), persistente Herkunftsmetadaten in der Comms-Nachricht und recoverable fan-out. ACK/Purge darf den Dedupe-Beweis nicht zerstören.

2. **Broadcast-Fan-out**
   - `postAs()` schreibt die Empfängerkopien zunächst temporär und benennt sie anschließend pro Empfänger um. Das verhindert unvollständige einzelne Dateien, ist aber keine atomare Transaktion über alle Empfänger.
   - Plane die W0-018-Recovery nach Crash zwischen beliebigen Empfänger-Renames: Wiederholung muss denselben kanonischen External-Record fortsetzen und darf keine Duplikate erzeugen.

3. **Worker-Lifecycle**
   - `internal/timer/worker.go` ist nur ein Referenzbeispiel, keine Vorlage zum Kopieren: `ensurePIDFile()` prüft Existenz/PID und schreibt danach eine PID-Datei; ein exklusiver Lock wird nicht über die gesamte Prozesslebenszeit gehalten.
   - Wormhole braucht den W0-018-Singleton mit dauerhaft gehaltenem flock, PID-Identität/Startzeit und Ready-Handshake. Start/Stop/Status müssen Race-sicher sein.

4. **Config und Service-Integration**
   - `internal/config/config.go`: `Config` und `Load()` kennen aktuell keinen `wormholes`-Block.
   - `internal/services/services.go`: Autostart kennt derzeit nur `web`, `model-proxy` und `timer`.
   - `internal/bootstrap/checks.go`: `starfleetAIGitignoreContent` ignoriert derzeit noch nicht `/var/wormholes/`.
   - Halte die W0-017-Vorgabe ein: Konfiguration unter `.starfleet-ai/conf/wormholes.yaml`; verwaltete Daten/Checkouts unter `.starfleet-ai/var/wormholes/<id>/`; exakt `/var/wormholes/` in der installierten `.starfleet-ai/.gitignore`; keine individuellen Checkout-Pfade.

5. **Git-/Dateisystem-Sicherheit**
   - Plane die W0-018-Korrekturen für initialen Clone in temporäres Geschwisterverzeichnis mit Validierung und no-replace atomic rename, Baseline-Verankerung am exakten initialen Commit/Tree, fehlenden State bei existierendem Clone fail-closed, geschützten inneren `.starfleet-state/`-Pfad, Transport-/Hook-/Submodule-Beschränkungen, Symlink-/TOCTOU-Schutz, sichere Ref-Validierung und Credential-Redaktion.
   - Definiere rekursive Glob-Semantik explizit; Go `filepath.Glob` interpretiert `**` nicht automatisch rekursiv.
   - Unbekannte Ship-Ziele bleiben pending/retryable und dürfen nicht dauerhaft als ignoriert verbucht werden.

## Erforderliche nächste Aktion

W0-019 bleibt verbindlich: Lies W0-017 und W0-018 vollständig, bestätige beide explizit und liefere zuerst den aktualisierten Implementierungsplan. Mappe **jede** W0-018-P0/P1-Korrektur auf konkrete Komponente, Datenmodell/Transaktion und Regressionstest; liste offene Architekturentscheidungen und Blocker ausdrücklich.

**Bis diese Acknowledgement und der aktualisierte Plan vorliegen, keine Codeänderungen und keine Commits.** Danach kann die Implementierung auf Basis des freigegebenen Plans beginnen. Enterprise bleibt für W0-020 unabhängiger read-only Reviewer; keine parallele Implementierung durch Enterprise.
