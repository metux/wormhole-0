# Starfleetctl / Wormhole – offene Themen und Arbeitsplan

Stand: 2026-11-12
Zweck: Persistenter Projekt-Notizzettel für die Zusammenarbeit mit ChatGPT und der Starfleet-Flotte. Dieser Plan ist ein Arbeitsindex, keine Behauptung über den aktuellen Laufzeitstatus. Statusänderungen nur anhand konkreter Commits, Tests, Nachrichten oder Laufzeitbelege eintragen.

## Aktuell in Arbeit

### W0-RM-READONLY – `read-only`-Option entfernen
- Ziel: Die nicht benötigte read-only-Konfiguration vollständig entfernen.
- Auftrag an Voyager: alle Konfigurationsfelder, Parser, Defaults, Dokumentation und Tests zu `read-only` suchen und entfernen; keine Ersatzoption einführen.
- Den schreibenden Maildir-Lifecycle beibehalten: `new -> cur -> processed/failed`, Commit/Push-Fehler sichtbar behandeln.
- Enger Scope: keine sonstigen Transport- oder Konfigurationsumbauten.
- Abnahme: Suche nach verbliebenen Referenzen; fokussierte Tests und `go test ./...` bzw. relevante Pakettests auf exakt dokumentiertem HEAD; Commit-SHA und vollständige Testausgaben.
- Status: Auftrag an Voyager publiziert; Bearbeitung/Antwort noch nicht bestätigt.
- Korrigierter Auftrag: https://github.com/metux/wormhole-0/blob/master/mail/Voyager/new/w0-20261112T1905Z-chatgpt-remove-read-only-voyager.md

## Priorität 1 – Transport und Comms belastbar abschließen

### W0-134 – Wormhole-Manager-Abnahme
Noch anhand des aktuellen Branch-HEAD zu prüfen, nicht aus früheren Implementierungsberichten ableiten:
- Vertrag von `PostExternal`: tatsächliche API-Signatur (5 vs. 7 Argumente) und eindeutige Semantik für Payload/Text.
- Konfigurations-/Read-only-Reste nach Entfernung der Option: Konfiguration soll den tatsächlich gepushten Remote-Zustand verwenden; keine lokale-only Erfolgsannahme.
- Crash-sichere Fehlertransition nach `failed/` einschließlich dauerhaftem Fehlergrund (z. B. `.reason`).
- Integrierter Test für `PostExternal -> Crash -> Recovery`: keine verlorene oder doppelt importierte Nachricht, Lifecycle-Zustand dauerhaft gepusht.
- Exakte HEAD-SHA und vollständige Ausgaben mindestens:
  - `go test ./internal/wormhole/... -count=1 -timeout 120s`
  - `go test ./internal/comms/... -count=1`
  - `go build ./...`
- Frühere 10s-Timeouts waren nicht beweiskräftig; nicht als Testfehler oder Erfolg werten.
- Abnahme erst nach Quellcodeprüfung und reproduzierbaren Ergebnissen.

### Comms-Routing und Zustellbelege
- Lokale Adressen ohne Domain sowie mit `@local` testen.
- Ship-Namen/Empfänger case-insensitive behandeln und Kollisionen eindeutig regeln.
- Mehrere Remote-Domains über Routing Table testen; keine Sonderbehandlung für den Namen ChatGPT.
- Sicherstellen, dass nicht-lokale Domains keinen Zugriff auf lokale Workspaces erhalten.
- Voyagers Route `pegasus -> wormhole/fleet-comms` dauerhaft und nachvollziehbar konfigurieren.
- Zustellstatus unterscheiden: veröffentlicht, remote importiert, vom Empfänger gelesen/verarbeitet, beantwortet. Ein Move nach `processed/` belegt nur den Import durch den Manager, nicht das Lesen durch das Modell.
- Reverse Routing und 38.6-KB-Payload sind bereits end-to-end mit passendem SHA-256-Bericht belegt; daraus keine unbegrenzte Payload-Grenze ableiten.
- Offener separater Test: explizite Bestätigung, dass die frische Spool-Testnachricht tatsächlich in der Voyager-Console ausgegeben wurde.

## Priorität 2 – Ship-Identität und Task-Lifecycle

### Console-Ship-Identität bei Neustart stabilisieren
- Anforderung: Wird ein Console-Schiff im selben Terminal neu gestartet, muss es denselben Ship-Namen/Identität behalten.
- Anlass: Nach einem Neustart wurde statt Discovery plötzlich Voyager verwendet.
- Nicht mit einem neuen Hintergrund-Schiff lösen; niemals ein bestehendes Console-Schiff zusätzlich im Hintergrund starten.
- Vor Implementierung untersuchen: Quelle der Identität (Terminal-/Session-Kontext, persistenter Zustand, CLI-Argumente), Namensvergabe beim Restart und Verhalten bei explizitem Rename.
- Abnahmetests: gleicher Terminal-Kontext + Restart => gleiche Identität; zwei unterschiedliche Terminals => keine unbeabsichtigte Kollision; explizite Namenswahl bleibt wirksam.
- Als eigenständigen Bug behandeln, getrennt von der Entfernung von `read-only`.

### Expliziter Task-Abschluss und Auto-Nudging
- `task_progress` für Fortschritt und `task_done` mit `completed | blocked | yielded`.
- Turn-Lifecycle, Task-Lifecycle und Ship-Lifecycle getrennt halten.
- Nach Ende eines Modell-Turns ohne Task-Abschluss darf ein Watchdog prüfen und ggf. nudgen; niemals nudgen, solange noch ein Modell-Turn läuft.
- Ca. 15 Minuten als zu prüfenden Default erwägen, nicht ungeprüft fest verdrahten.
- Idempotenz, Neustartverhalten, manuelle Unterbrechung und blockierte/yielded Tasks spezifizieren.
- Ziel: Agenten bleiben nicht still mitten in einer Aufgabe stehen und vergessen Commit/Push oder Abschlussmeldung.

## Priorität 3 – spätere Bereinigungen und Erweiterungen

### `cleanup-2o` – direkte JSON-Injection
- Starfleetctl-Kommando liefert JSON direkt an das Plugin.
- Enthält Konfiguration, Polling-Intervall, Modellwechsel und vollständigen Injection-Snippet/JSON.
- Plugin soll nicht selbst Nachrichtentext generieren.
- Session-ID kommt aus der Ship-ID-Umgebung; keine mehreren Sessions für dasselbe Schiff.
- Wormhole-Transportänderungen und Injection-Arbeit in getrennten Commits/PRs halten.

### Dispatch-Lease-Reste
- Prüfen, ob `dispatchClaim()` / `dispatchRelease()` nach Aufgabe der Mehrfach-Session-pro-Ship-Anforderung noch gebraucht werden.
- Erst Aufrufpfade und tatsächliche Konkurrenz-/Crash-Semantik untersuchen; nicht vorschnell entfernen.

### Console-Sicherheits-/Berechtigungsverhalten
- Für Console-Schiffe OpenCode-Ask blockieren, analog zu Background-Schiffen.
- `external_directory`-Zugriff soll weiterhin möglich bleiben.
- Berechtigungsregeln getrennt prüfen; nicht pauschal alle externen Zugriffe sperren.

### Artefakte / große Dateien im Wormhole
- Aktuell kein vereinbartes, implementiertes Artefaktregister voraussetzen.
- Falls benötigt, minimaler Vorschlag:
  - `artifacts/index.md`
  - `artifacts/<artifact-id>/manifest.yaml`
  - Manifest mit Beschreibung, relativen Pfaden, Größen und SHA-256 (MIME-Typ optional).
  - Nachrichten referenzieren Artefakt-ID und Pfad.
- Erst nach konkretem Bedarf umsetzen; für sehr große Binärdateien später Git LFS/externe Ablage prüfen.

## Bereits belegte Fakten (nicht als offene Fehler wiederholen)
- Normaler Comms-Rückweg über `postAs()/resolveTarget()/publishRemote()` und Wormhole-Transport ist im Quellcode vorhanden.
- Voyager hat eine Nachricht über normale Comms an `ChatGPT@pegasus` gesendet; die Nachricht landete im ChatGPT-Wormhole-Spool.
- Eine 38,656-Byte-Payload mit 256 Sequenzzeilen wurde in Voyager importiert und von Voyager mit korrekter SHA-256, Zeilenzahl sowie Start-/Endmarkern bestätigt.
- Das belegt diesen konkreten Ende-zu-Ende-Fall, aber weder eine maximale Payload-Größe noch den separaten Console-Ausgabe-Test.
- Requests an Enterprise und Defiant wurden importiert; Antworten waren zuletzt noch nicht nachgewiesen. Aktuelle Flottenzuordnung: Voyager = Starfleetctl, Defiant = Xaverius, Enterprise = lokale Koordination/Comms-Tests.

## Arbeitsregeln
- Keine Runtime-Aussage nur aus Commit oder `processed/` ableiten.
- Bei jeder Abnahme exakten Branch, HEAD-SHA, konkrete Tests und vollständige Ergebnisse dokumentieren.
- Kleine, atomare Änderungen; keine unnötigen Refactors.
- Vor Restart bestehender Console-Schiffe Identität und Terminal-Kontext erhalten; niemals unbemerkt ein zweites Schiff starten.
- Dieser Plan ist ein Index. Detailentscheidungen und größere Spezifikationen bei Bedarf in eigene Dateien auslagern.
