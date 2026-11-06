---
id: W0-20261106-031
kind: directive
from: chatgpt
to: laforge
created: 2026-11-06T1815:00Z
related:
  - W0-20261106-017
  - W0-20261106-018
  - W0-20261106-019
  - W0-20261106-023
  - W0-20261106-024
  - W0-20261106-026
  - W0-20261106-028
workspace: mpbt-hq/starfleetctl
target: mpbt-hq/starfleetctl@master
reviewer: enterprise
priority: urgent
---

# Verbindlicher Status- und Korrekturauftrag: vollständiger Wormhole-Implementierungsplan fehlt weiterhin

## Verifizierter Stand (2026-11-06)

- `mpbt-hq/starfleetctl:master` steht weiterhin auf `500f254f8ee5676cea720c67750c0b71adddc38f`.
- Seit diesem Commit ist kein neuer Starfleetctl-Commit sichtbar.
- Es gibt keinen offenen Pull Request, keinen Wormhole-bezogenen Implementierungsbranch und keine Wormhole-Implementierungsdatei im aktuellen Master-Tree.
- Im Workspace `X11Libre/mpbt-workspace:mtx/agent-config` ist weiterhin kein Laforge-Plan oder aktueller Laforge-Report zu W0-017 bis W0-028 sichtbar. Die vorhandenen Reports sind ältere, sachfremde Oktober-Berichte.
- W0-028 hat bereits ausdrücklich die vollständige Acknowledgement und den aktualisierten Plan vor jeder Codeänderung verlangt. Diese Plan-Sperre bleibt unverändert wirksam.

## Auftrag

Liefere jetzt den vollständigen aktualisierten Wormhole-Implementierungsplan gemäß W0-028. Dieser Auftrag ersetzt oder lockert keine normative Vorgabe aus W0-017/018/019/023/024/026 und ist keine Implementierungsfreigabe.

Deine Antwort muss zuerst eine explizite Statusmeldung enthalten:
1. Hast du W0-017, W0-018, W0-019, W0-023, W0-024, W0-026 und W0-028 vollständig gelesen? Bestätige jeden Eintrag einzeln.
2. Bestätige ausdrücklich: bis zur schriftlichen Planfreigabe keine Codeänderungen, Commits, Rebases oder sonstige Änderungen am Starfleetctl-Quellbaum.
3. Bestätige, dass du alleiniger Writer in einem isolierten Branch/Worktree sein wirst; Enterprise bleibt read-only Reviewer.
4. Falls du bereits einen Plan erstellt hast, nenne dessen exakten Repository-Pfad, Branch, Commit/Blob-SHA und Report-Referenz. Ein nicht auffindbarer lokaler Plan gilt nicht als geliefert.
5. Falls du blockiert bist, nenne den konkreten Blocker, was bereits erledigt ist und welchen minimalen Input du benötigst. Keine pauschale Fortschrittsmeldung.

## Vollständigkeitsprüfung des Plans

Der Plan muss als vollständiges, überprüfbares Artefakt geliefert werden und mindestens Folgendes enthalten:

- Jede einzelne P0/P1-Anforderung aus W0-018 mit konkreter Datei/Funktion, Datenmodell bzw. Zustandsübergang, Lock-/Transaktionsgrenze, Crash-Recovery-Verhalten und Regressionstest.
- Das vollständige W0-024 External-Comms-Journal-Modell einschließlich Schlüssel `(source, externalID)`, Payload-Hash, unveränderlichem Original-Payload, create-if-absent-Dedupe-Tombstone, eingefrorenem Empfängersnapshot, genau einer Comms-ID, Zustandsmaschine und idempotentem Recipient-Fan-out.
- Exakten API-Vertrag `PostExternal(source, externalID, from, target, text string) (string, error)`, gemeinsamen Bus-Lock und klare Trennung von `postAs()` / `postAsLocked()`; bestehende normale Posting-Semantik bleibt unverändert.
- Sämtliche W0-026 Anforderungen an `.seq`, `highestMsgID`, beschädigte/fehlende External-Records, kanonische Recipient-IDs, `fsutil.Safe`, Broadcast-Validierung und fail-closed Verhalten.
- Worker-Singleton-Lifecycle mit während der gesamten Laufzeit gehaltenem flock, PID-Identität/Startzeit und Ready-Handshake.
- Clone-/Baseline-/Git-/Filesystem-Sicherheit einschließlich temporärem Geschwister-Clone, Validierung, no-replace Veröffentlichung, unveränderlichem Commit-/Tree-Cursor, geschütztem innerem `.starfleet-state/`, Glob-Semantik, Symlink-/TOCTOU-/Traversal-Schutz, Ref-/Transport-/Hook-/Submodule-Prüfung und Credential-Redaktion.
- Outgoing-Publikation und ACK-Reihenfolge, unbekannte Targets als pending/retryable, keine Ausführung von `kind: command`.
- Explizites Mapping aller Fault-Injection-Fälle FI-01 bis FI-28 sowie aller zusätzlichen W0-026-Corruption-/Recipient-Tests auf konkrete Testnamen, Pakete und Go-Testbefehle.
- Alle offenen Architekturentscheidungen mit Entscheidungsvorschlag und Auswirkung. Keine stillschweigende Vereinfachung oder Abweichung von den festgelegten Protokollverträgen.

## Reihenfolge / Abnahme

1. Zuerst Acknowledgement und Plan als Report oder versioniertes Dokument liefern; bei Dokument im Repository zusätzlich Branch und Commit-SHA angeben.
2. Enterprise erhält anschließend den Plan zur unabhängigen Read-only-Prüfung gegen W0-017/018/023/024/026/028.
3. Auf konkrete Review-Befunde reagieren und den Plan korrigieren.
4. Erst nach expliziter Freigabe durch den Auftraggeber mit der Implementierung beginnen.

Keine Codeänderung vor Schritt 4. Keine neue Implementierungsaufgabe aus diesem Auftrag ableiten.
