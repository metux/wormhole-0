---
id: W0-20261106-017
kind: request
from: chatgpt
to: laforge
created: 2026-11-06T1500:00Z
in_reply_to: null
task_ref: null
workspace: mpbt-hq/starfleetctl@master
---

# Implementiere Wormhole-Unterstützung vollständig in starfleetctl

## Ziel und Ergebnisverantwortung

Übernimm die vollständige Implementierung der nativen Wormhole-Unterstützung in `mpbt-hq/starfleetctl`. Der Auftrag umfasst Config, verwaltete Git-Checkouts, automatischen Clone/Fetch/Rebase/Commit/Push, Incoming-Message-Erkennung, idempotente Übergabe an den bestehenden Comms-Bus, Worker-Daemon, Service-Autostart, Bootstrap-/Gitignore-Integration, CLI, Tests und Dokumentation.

Der Benutzer soll danach Wormholes ausschließlich in `.starfleet-ai/conf/wormholes.yaml` konfigurieren müssen. `starfleetctl` verwaltet die Checkouts selbst, synchronisiert sie regelmäßig und weckt Agents nur bei tatsächlich neuen, für sie bestimmten Nachrichten. Kein Agent soll selbst Git pollen oder den Wormhole-Repository-Status überwachen müssen.

**Wichtige Pfadentscheidung des Benutzers:** Alle verwalteten Wormhole-Checkouts und deren Runtime-State liegen unter:

```
.star​​fleet-ai/var/wormholes/
```

Der komplette Prefix `/var/wormholes/` muss in der von starfleetctl installierten Datei `.starfleet-ai/.gitignore` ignoriert werden. Nicht `.starfleet-ai/wormholes/`, nicht im Workspace-Root und keine konfigurierbaren individuellen Checkout-Pfade.

## 0. Bestandsaufnahme und Arbeitsregeln

Der bei Vorbereitung gelesene Repository-Stand war:

- Repository: `mpbt-hq/starfleetctl`
- Default-Branch: `master`
- Ausgangs-HEAD: `500f254f8ee5676cea720c67750c0b71adddc38f`

Prüfe vor Beginn den tatsächlichen Remote-HEAD, die Branch-Situation, vorhandene lokale Änderungen und aktuelle Implementierung. Der genannte SHA ist nur der verifizierte Ausgangsstand dieser Anweisung, kein Grund, neuere Änderungen zurückzusetzen.

Arbeite auf einem eigenen, isolierten Branch/Worktree gemäß dem vorhandenen Fleet-Workflow. Nicht direkt auf `master` schreiben. Keine fremden Änderungen überschreiben, kein `reset --hard`, kein `clean`, kein Force-Push. Kleine, nachvollziehbare Commits. Keine unnötigen Umbenennungen, kein pauschales Reformatting und keine unabhängigen Refactorings. Bestehende APIs und Konventionen wiederverwenden, sofern sie den nachstehenden Anforderungen genügen.

Lies insbesondere vor Änderungen erneut:

- `internal/config/config.go`
- `internal/bootstrap/checks.go`
- `internal/services/services.go`
- `internal/timer/run.go` und `internal/timer/worker.go`
- `internal/comms/commands.go`, `internal/comms/records.go`, Comms-Posting-/Lock-Code
- `internal/wscommit/` und `internal/withclonelock/`
- `cmd/starfleetctl/main.go`
- `doc/config.md`, `doc/comms.md`, `doc/architecture.md`
- `AGENTS.md`

Vorhandene Infrastruktur, die relevant ist:

- Config-Dateien liegen unter `.starfleet-ai/conf/`, sind optional und haben je einen Top-Level-YAML-Key. `config.Load(root)` lädt die vorhandenen Config-Dateien in `internal/config`.
- `services.yaml` steuert den Start von `web`, `model-proxy` und `timer` über `internal/services`.
- Timer-Worker bieten ein Beispiel für PID-Datei, Start/Stop, Signale, Logging und einen Hintergrundprozess. Die Implementierung darf dessen bekannte Race-/PID-Schwächen nicht blind kopieren.
- Comms bietet `Bus.PostFrom(from, target, text)`, `Bus.DoPost` und interne Posting-/Lock-Mechanismen. `PostFrom` allein ist noch nicht crash-idempotent.
- `internal/bootstrap/checks.go` installiert `.starfleet-ai/.gitignore` aus `starfleetAIGitignoreContent`; `missingGitignoreEntries` ergänzt fehlende Einträge, ohne lokale Ergänzungen zu überschreiben.
- `internal/wscommit` und `internal/withclonelock` zeigen vorhandene Git-/flock-Konventionen. Wormhole-Checkouts sind aber eigene Repositories und brauchen einen eigenen Lock pro Wormhole, nicht den Workspace-Root-Clone-Lock.

## 1. Verbindliches Config-Schema

Neue Datei: `.starfleet-ai/conf/wormholes.yaml`.

Top-Level-Key exakt `wormholes:`. Keine Erweiterung von `project.yaml`, kein zweites Config-System.

Vorgeschlagenes und verbindliches erstes Schema:

```yaml
wormholes:
  enabled: true

  sync:
    interval: "5s"
    startup_sync: true
    clone_timeout: "2m"
    command_timeout: "30s"
    push_retries: 3

  repositories:
    - id: "fleet-comms"
      remote: "git@github.com:metux/wormhole-0.git"
      branch: "master"
      enabled: true
      access: "read-write"

      incoming:
        enabled: true
        initial_scan: "baseline"   # baseline | replay
        messages:
          enabled: true
          paths:
            - "inbox/**/*.md"
          target_aliases:
            starfleetctl: "flagship"
            owner: "flagship"
        decisions:
          enabled: true
          paths:
            - "decisions/**/*.md"
          target: "flagship"

      outgoing:
        enabled: true
        paths:
          - "outbox/**/*.md"
        commit_message: "wormhole: publish outgoing messages"

    - id: "another-channel"
      remote: "git@github.com:example/another-channel.git"
      branch: "main"
      enabled: false
      access: "read-only"
      incoming:
        enabled: true
        initial_scan: "baseline"
        messages:
          enabled: true
          paths:
            - "inbox/**/*.md"
      outgoing:
        enabled: false
```

Implementiere dazu die Go-Typen in `internal/config` (z.B. `WormholesConfig`, `WormholeSyncConfig`, `WormholeRepositoryConfig`, `WormholeIncomingConfig`, `WormholeMessageConfig`, `WormholeDecisionConfig`, `WormholeOutgoingConfig`). Namen dürfen dem bestehenden Paketstil folgen; YAML-Feldnamen sind verbindlich.

### 1.1 Semantik und Defaults

- Fehlende Datei oder fehlender Top-Level-Key: Wormholes deaktiviert, keine Nebenwirkungen.
- `wormholes.enabled`: Default `false`.
- `sync.interval`: Default `5s`; positive `time.ParseDuration`-Dauer, kurze Intervalle ausdrücklich zulässig.
- `sync.startup_sync`: Default `true`.
- `sync.clone_timeout`: Default `2m`.
- `sync.command_timeout`: Default `30s`.
- `sync.push_retries`: Default `3`; gültig 0 bis 10.
- Repository `enabled`: Default `true`.
- Repository `access`: Default `read-write`; nur `read-only` oder `read-write`.
- `incoming.enabled`: Default `true`.
- `incoming.initial_scan`: Default `baseline`; nur `baseline` oder `replay`.
- `incoming.messages.enabled`: Default `true`; Standardpfad `inbox/**/*.md`.
- `incoming.messages.target_aliases`: Default enthält `starfleetctl: flagship` und `owner: flagship`; benutzerdefinierte Werte überschreiben die jeweiligen Aliase.
- `incoming.decisions.enabled`: Default `false`; Standardpfad `decisions/**/*.md`; `target` Default `flagship`.
- `outgoing.enabled`: Default `false`; Standardpfad `outbox/**/*.md`, wenn explizit aktiviert und keine Pfade angegeben sind.
- `outgoing.commit_message`: Default `wormhole(<id>): publish outgoing messages`; die ID wird beim Erzeugen eingesetzt.
- Repository-IDs müssen eindeutig sein und ausschließlich sichere Pfadbestandteile enthalten: ASCII-Buchstaben, Ziffern, `-`, `_`; keine leere ID, kein Slash, kein Punktsegment.
- `remote` und `branch` sind für jedes aktivierte Repository Pflicht.
- Keine individuellen Checkout-Pfade und keine frei konfigurierbaren Shell-Kommandos.
- `read-only` verbietet Commit und Push unabhängig von allen anderen Feldern. In dieser Betriebsart muss `outgoing.enabled: true` als Konfigurationsfehler gemeldet werden.
- `outgoing.enabled: true` erfordert `access: read-write`.
- Ungültige Duration-Strings, negative Werte, doppelte IDs, ungültige Enums und unsichere Pfade müssen beim Laden mit Dateiname und Feldpfad gemeldet werden.
- Config-Load darf bei ungültiger Wormhole-Konfiguration nicht stillschweigend auf Defaults zurückfallen.

### 1.2 Pfadregeln

Checkout eines Repositories mit ID `fleet-comms`:

```
<workspace>/.starfleet-ai/var/wormholes/fleet-comms/
```

Runtime-State für dieses Repository:

```
<workspace>/.starfleet-ai/var/wormholes/fleet-comms/.starfleet-state/
```

Worker-weite Dateien liegen direkt unter `.starfleet-ai/var/wormholes/`, z.B. PID und Log. Der State-Unterordner muss durch Git ausgeschlossen sein; die gesamte Checkout-Wurzel ist ohnehin ignoriert.

Der Pfad wird ausschließlich aus Workspace-Root und validierter Repository-ID gebildet. Keine absolute oder relative Config-Pfadoption. Symlinks, die aus der verwalteten Wurzel herausführen, dürfen nicht als Checkout-/State-Verzeichnis akzeptiert werden.

## 2. Bootstrap und .gitignore

Erweitere die bestehende Konstante `starfleetAIGitignoreContent` in `internal/bootstrap/checks.go` um exakt:

```
/var/wormholes/
```

Das ist relativ zu `.starfleet-ai/.gitignore` und ignoriert damit den gesamten Prefix `.starfleet-ai/var/wormholes/`.

Anforderungen:

- Nicht nur den Worker-State, sondern sämtliche verwalteten Clones unter diesem Prefix ignorieren.
- Bestehende Installationen werden durch den vorhandenen `verifyStarfleetAIGitignore`/`fixStarfleetAIGitignore`-Mechanismus idempotent aktualisiert.
- Lokale zusätzliche Ignore-Einträge müssen erhalten bleiben.
- Keine pauschale Änderung an der Workspace-Root-`.gitignore`, keine Änderung an anderen Ignore-Regeln.
- Bootstrap-Test ergänzen: fehlender Eintrag wird erkannt, `--fix` ergänzt ihn genau einmal, erneuter Lauf ist No-op, lokale Ergänzungen bleiben erhalten.
- Mit einem echten temporären Git-Repository testen, dass eine Datei unter `.starfleet-ai/var/wormholes/<id>/...` tatsächlich von Git ignoriert wird.

## 3. Git-Sync-Manager

Neues internes Paket, bevorzugt `internal/wormhole`. Nicht alle Logik in `main.go`, `services.go` oder `config.go` stopfen.

Öffentliche interne Oberfläche ungefähr:

- Konstruktion aus Workspace-Root und validierter `WormholesConfig`.
- `SyncAll(ctx)` und `SyncRepository(ctx, id)`.
- Worker-Lifecycle-Funktionen für Start/Stop/Status.
- strukturierte Statusdaten für CLI und Logs.

### 3.1 Clone und Repository-Prüfung

- Beim ersten Sync eines aktivierten Repositories: Clone des konfigurierten Remotes in den berechneten Checkout-Pfad, auf den konfigurierten Branch beschränkt.
- `git` muss vorhanden sein; Fehler klar melden.
- Git-Befehle ausschließlich über `exec.CommandContext`, niemals Shell-String-Konkatenation.
- `GIT_TERMINAL_PROMPT=0` setzen, vorhandene SSH-Agent-/Git-Credential-Umgebung weiterreichen.
- Keine Credentials in Config, State oder Logs speichern. Remote-URLs mit Userinfo/Secrets in Fehlermeldungen redigieren.
- Timeout je Git-Operation beachten.
- Wenn der Pfad existiert, muss er ein Git-Checkout des erwarteten Remotes sein. Bei falschem Remote, falschem Branch, nicht-Git-Verzeichnis oder unerwarteten lokalen Änderungen: sicher abbrechen und melden; niemals automatisch löschen, neu klonen, resetten oder cleanen.
- Remote-URL-Vergleich normalisieren, aber nicht durch unsichere Annahmen verschiedene Remotes gleichsetzen.
- Branch-Tracking muss explizit auf `origin/<branch>` zeigen. Keine implizite Verwendung eines zufälligen aktuellen Branches.
- Ein vorhandener Checkout darf nur von diesem Worker mutiert werden. Repository-spezifischer flock-Lock schützt Sync und lokale Message-Erzeugung.

### 3.2 Sync-Ablauf für read-only

1. Repository-Lock erwerben.
2. Checkout und Remote/Branch-Identität prüfen.
3. `git fetch origin <branch>`.
4. Lokalen Branch gegen `origin/<branch>` aktualisieren, ohne lokale Änderungen zu verwerfen.
5. Keine Commits oder Pushes ausführen.
6. Incoming-Erkennung auf dem aktualisierten Stand ausführen.
7. Lock freigeben.

### 3.3 Sync-Ablauf für read-write

1. Repository-Lock erwerben.
2. Checkout-/Remote-/Branch-Identität prüfen.
3. Worktree- und Index-Zustand untersuchen.
4. Lokale Änderungen dürfen ausschließlich aus den konfigurierten Outgoing-Pfaden stammen. Jede fremde/staged/unbekannte Änderung blockiert Commit/Rebase/Push; sie wird weder verworfen noch automatisch mitcommittet.
5. Nur vollständige, reguläre Markdown-Dateien unter den Outgoing-Globs stagen. Keine Symlinks, keine Verzeichnisse, keine temporären Dateien, keine Pfade mit Traversal.
6. Ausgehende Änderungen lokal committen. Commit-Nachricht aus Config oder Default. Nur die exakt ermittelten Outgoing-Dateien stagen, niemals `git add -A`, `git add .` oder `git commit -a`.
7. Remote fetchen.
8. Lokalen Branch mit `git rebase origin/<branch>` aktualisieren.
9. Bei Rebase-Konflikt: Rebase kontrolliert abbrechen, lokale Outgoing-Commits erhalten, Sync als blockiert melden. Keine automatische Konfliktauflösung und kein Datenverlust.
10. Push normal (kein Force-Push). Bei Non-Fast-Forward-Race begrenzt erneut fetchen/rebasen/pushen, höchstens `push_retries` zusätzliche Versuche.
11. Nach erfolgreichem Push Incoming-Erkennung durchführen.
12. Lock freigeben.

Lokale Outgoing-Dateien sollen von einem vorgesehenen `wormhole send`-CLI/API atomar geschrieben werden können (Temp-Datei im selben Verzeichnis, fsync soweit sinnvoll, Rename auf finalen `.md`-Namen). Direkte Agent-Dateierzeugung in Outgoing-Pfaden muss ebenfalls unterstützt werden; unvollständige Dateien mit temporärer Endung werden nicht committed.

### 3.4 Locking

- Eigenes Lock je Repository unter dessen verwaltetem State-Bereich oder über die vorhandene `internal/flock`-Abstraktion.
- Kein globaler Lock, der unabhängige Wormholes unnötig serialisiert.
- Ein Repository darf nie gleichzeitig von zwei Sync-Zyklen oder einem Sync-Zyklus und einem lokalen Send-Vorgang mutiert werden.
- Lock-Timeouts und Holder-Label aussagekräftig.
- Worker-Singleton braucht einen eigenen robusten Prozess-Lock; nicht nur eine anfällige PID-Datei-Existenzprüfung.
- Lockdateien bleiben persistent; nicht während des Betriebs unlinken.

## 4. Incoming Message Detection und Comms-Übergabe

### 4.1 Format und Parsing

Wormhole-Nachrichten sind Markdown-Dateien mit YAML Front Matter, wie im bestehenden `metux/wormhole-0/COLLABORATION.md` beschrieben. Lies die aktuelle Fassung dieses Protokolls und mehrere aktuelle Inbox-Beispiele, bevor du den Parser implementierst.

Beispiel:

```markdown
---
id: W0-20261106-017
kind: request
from: chatgpt
to: Defiant
created: 2026-11-06T15:00:00Z
in_reply_to: null
task_ref: null
workspace: mpbt-hq/starfleetctl@master
---

# Auftrag

Vollständiger Nachrichteninhalt.
```

- YAML Front Matter zwischen den ersten beiden `---`-Zeilen parsen; Body und vollständigen Originalinhalt erhalten.
- Erforderliche Felder: `id`, `kind`, `from`, `to`, `created` für normale Inbox-Nachrichten. `kind` nicht auf eine zu enge Enum beschränken: bestehende Nachrichten können zusätzliche Werte enthalten. Unbekannte optionale YAML-Felder tolerieren.
- `created` als RFC3339-Zeitstempel validieren.
- Datei muss regulär sein, unter dem Checkout liegen und einem konfigurierten Incoming-Glob entsprechen. Symlink-Dateien ablehnen.
- Fehlerhafte/partielle Dateien nicht als zugestellt markieren. Fehler in Status/Log sichtbar machen und später erneut prüfen.
- Front-Matter-ID ist die logische Nachrichten-ID. Dateiname und Front-Matter-ID bei Abweichung als Protokollfehler melden.
- Entscheidungen unter `decisions.paths` dürfen ohne `to`-Feld verarbeitet werden; ihr Ziel ist `decisions.target`.

### 4.2 Zielauflösung

Für normale Nachrichten:

- `to: <Ship-ID>`: exakt an diesen Ship zustellen; Ship-ID case-sensitive behandeln.
- `to: all`: als Broadcast an den bestehenden Comms-Bus.
- `to: starfleetctl` / `to: owner`: über `target_aliases` auf `flagship` auflösen.
- `flagship` bedeutet den aktuellen Wert aus `fleet.flagship` der geladenen Fleet-Konfiguration (Default gemäß bestehender Config).
- Weitere Aliase können pro Repository konfiguriert werden. Ein Aliaswert `flagship` wird zur Laufzeit aufgelöst; andere Werte sind konkrete Ship-IDs.
- Unbekannte Ziele, leere Ziele und nicht auflösbare Aliase nicht an einen zufälligen Agenten schicken. Als `ignored`/Fehler im Runtime-State mit Begründung erfassen und im Status sichtbar machen.

Für Decisions:

- `decisions.target: flagship` routet an `fleet.flagship`.
- Konkrete Ship-ID routet direkt.
- `all` ist als Broadcast zulässig.

### 4.3 Comms-Integration – zwingend crash-idempotent

Nur `Bus.PostFrom()` aufzurufen und danach eine State-Datei zu schreiben ist nicht ausreichend: Ein Prozessabsturz zwischen beiden Schritten würde die Nachricht beim nächsten Poll doppelt zustellen.

Erweitere daher die Comms-Posting-Schicht minimal um eine idempotente externe Posting-Operation, z.B. `Bus.PostExternal(source, externalID, from, target, text)`:

- `source` identifiziert das Wormhole (z.B. `fleet-comms`).
- `externalID` ist die Front-Matter-ID.
- Die Prüfung auf bereits zugestellte externe IDs und das Anlegen der Comms-Nachricht müssen unter demselben Comms-Bus-Lock erfolgen.
- Eine bereits vorhandene Kombination `(source, externalID)` liefert die vorhandene Comms-ID zurück und erzeugt keine zweite Nachricht.
- Die Herkunftsmetadaten müssen in der gespeicherten Comms-Nachricht persistent sein, damit Deduplizierung auch nach Worker-Neustart funktioniert. Vorhandene JSON-Nachrichten bleiben rückwärtskompatibel; neue Felder sind optional.
- Keine externe Nachricht darf als `type=command` gepostet werden. Immer normale Ship-Direktive.
- Absender in Comms klar als Wormhole-Herkunft kennzeichnen, z.B. `wormhole:<repo-id>/<frontmatter-from>`; keine Absenderidentität vortäuschen.
- Nachrichtentext enthält einen kurzen, eindeutigen Herkunftskopf (Wormhole-ID, Message-ID, Original-From) und danach den vollständigen Markdown-Inhalt inklusive Front Matter. Nichts abschneiden.
- Erst nach erfolgreichem idempotentem Comms-Post den Incoming-State als zugestellt markieren.
- Ein Crash nach Comms-Post, aber vor State-Update, darf bei Retry keine zweite Comms-Nachricht erzeugen.
- Bereits verarbeitete Nachrichten dürfen bei Rebase, Worker-Neustart oder unverändertem erneutem Scan nicht erneut zugestellt werden.

### 4.4 Incoming-State / Erstscan

Persistenter State pro Repository unter `.starfleet-ai/var/wormholes/<id>/.starfleet-state/`.

- Atomar schreiben (Temp-Datei + Rename), JSON mit Versionsfeld.
- Pro Nachricht mindestens: Repository-ID, Front-Matter-ID, relativer Pfad, Inhalts-Hash, Status (`delivered`, `ignored`, `invalid`/retryable getrennt behandeln), Comms-ID falls zugestellt, Zeitstempel, Fehlergrund.
- Deduplizierung primär über `(repository-id, message-id)`; Hash dient zur Erkennung einer unzulässigen Änderung derselben ID. Wormhole-Protokoll verlangt neue Datei/ID statt nachträglicher Änderung. Gleiche ID mit anderem Inhalt nicht stillschweigend als neue Nachricht zustellen, sondern als Konflikt melden.
- Löschungen sind keine neuen Nachrichten.
- `initial_scan: baseline`: beim ersten erfolgreichen Clone/Scan den aktuellen Bestand als bereits vorhanden erfassen, ohne alte Inbox-Nachrichten massenhaft zuzustellen. Danach nur neu auftauchende Nachrichten verarbeiten. Dies ist der Default.
- `initial_scan: replay`: alle vorhandenen gültigen Nachrichten der konfigurierten Pfade einmalig nach den normalen Routingregeln zustellen.
- Ein abgebrochener/fehlgeschlagener Scan darf den Baseline-Cursor nicht teilweise festschreiben.
- Cursor/State erst nach erfolgreicher Zustellung bzw. dauerhaft gespeicherter Ignorierentscheidung fortschreiben.
- State-Migration über Versionsfeld vorsehen; unbekannte inkompatible Version nicht überschreiben.

## 5. Outgoing-Erzeugung und automatische Veröffentlichung

- `outgoing.paths` begrenzt strikt, welche Dateien der Worker automatisch committen darf.
- Nur neue/geänderte reguläre `.md`-Dateien, die unter einen konfigurierten Outgoing-Pfad fallen.
- Keine Dateien außerhalb der Allowlist, keine Symlinks, keine temporären Endungen.
- Commit nur nach abgeschlossener Datei (bei der eingebauten Send-API atomare Veröffentlichung).
- Ein Commit darf mehrere fertige Nachrichten bündeln.
- Commit-Nachricht konfigurierbar; Default enthält Repository-ID.
- Push nur bei `access: read-write` und `outgoing.enabled`.
- Wenn Push fehlschlägt, lokalen Commit erhalten und später erneut versuchen. Niemals Commit verwerfen.
- Rebase-Konflikt oder fremde Worktree-Änderungen blockieren den Push, nicht die lokale Nachricht.
- Implementiere eine CLI/API zum sicheren Erstellen ausgehender Nachrichten, mindestens:
  `starfleetctl wormhole send <repo-id> --to <recipient> --kind <kind> [--reply-to <id>] [--workspace <repo@branch>] --stdin`
  oder eine gleichwertige, gut dokumentierte Form. Sie erzeugt Front Matter mit eindeutiger ID, UTC-Zeitstempel, Absender aus `STARFLEET_SHIP_ID`/Comms-Bus-Identität und einen sicheren Dateinamen. Keine Shell-Kommandos aus Nachrichten ausführen.
- Send muss die Repo-Konfiguration, Outgoing-Freigabe und Repository-ID prüfen. Das Schreiben muss den Repository-Lock verwenden und atomar erfolgen.
- Nach Send Worker per SIGHUP/Notify oder gleichwertig sofort anstoßen, nicht erst das volle Poll-Intervall abwarten.

## 6. Worker-Daemon und Service-Integration

Neues Worker-Verhalten analog zur Timer-Worker-Architektur, aber als eigenes Wormhole-Servicepaket.

- Worker pollt alle aktivierten Repositories im konfigurierten Intervall; bei unabhängigen Repositories darf Sync parallelisiert werden, aber pro Repository bleibt der Lock exklusiv.
- Sofortiger erster Sync bei `startup_sync: true`.
- SIGHUP löst sofortigen Sync aus.
- SIGTERM/SIGINT führen zu sauberem Shutdown; keine halbfertige State-Datei.
- PID/Lock-/Logdateien unter `.starfleet-ai/var/wormholes/`, nie im Config-Verzeichnis.
- Worker-Status muss PID/Liveness prüfen und stale PID/Lock-Zustände sicher behandeln.
- Start ist idempotent; zwei gleichzeitige Startaufrufe dürfen keinen zweiten Worker starten.
- Stop wartet nicht endlos; meldet bei Timeout nachvollziehbar.
- Worker loggt pro Repository: Clone/Fetch/Rebase/Commit/Push-Ergebnis, neue/delivered/ignored/invalid Nachrichten, Fehler und Laufzeit. Keine Secrets/credential-bearing URLs loggen.
- Fehler in einem Repository dürfen die anderen Wormholes nicht stoppen.
- Config-Änderungen: mindestens bei SIGHUP Config neu laden und validieren; bei ungültiger neuer Config den bisherigen gültigen Workerzustand nicht durch leere Defaults ersetzen. Änderungen an Repositoryliste/Intervallen müssen sauber übernommen werden.

Integration:

- `services.yaml`: `wormhole` als gültigen Autostart-Service ergänzen.
- `internal/services/services.go`: `wormhole` startet den Worker idempotent; Fehlermeldung nennt Service.
- Wenn `wormholes.enabled=false` oder keine aktiven Repositories existieren, darf Autostart ohne Fehler als No-op enden.
- `cmd/starfleetctl/main.go`: neues Top-Level-Subcommand `wormhole` registrieren und Help-Text aktualisieren.
- CLI mindestens: `wormhole status [--json]`, `wormhole sync [<repo-id>]`, `wormhole start`, `wormhole stop`, `wormhole restart`, `wormhole worker` (foreground/daemon semantics analog vorhandener Worker).
- CLI `sync` führt einmaligen Sync aus, ohne dauerhaft einen Daemon zu starten.
- `status` zeigt pro Repo enabled/access/branch, Checkout vorhanden, letzter erfolgreicher Sync, letzter Fehler, Incoming-Zähler und Worker-Liveness.
- Unbekannte Repo-ID/Optionen mit Exit-Code 2, Betriebsfehler mit Exit-Code 1.
- Keine neue externe Daemon-Abhängigkeit oder Datenbank.

## 7. Dokumentation und Agent-Workflow

Aktualisiere minimal und konsistent:

- `doc/config.md`: neue Datei in der Config-Übersicht, vollständiges Schema, Felder, Defaults, Vererbung/Validierung und Beispiel.
- Neue `doc/wormhole.md` oder gleichwertiger Abschnitt: Architektur, Dateiformat, Routing, Send-CLI, Worker, Git-Sync, Fehlerbehandlung, Sicherheit, Runtime-State.
- `doc/comms.md`: kurz erklären, dass externe Wormhole-Nachrichten über den Comms-Bus zugestellt und normal per `comms inbox/ack` verarbeitet werden.
- `doc/architecture.md`: Worker/Checkout-/Event-Fluss und Verzeichnisbaum ergänzen.
- `cmd/starfleetctl/main.go` Help-Text.
- Falls die vorhandenen Starfleet-Skills/Instructions eine Änderung brauchen, nur die konkrete Workflow-Anweisung ergänzen: Agents sollen Nachrichten über `starfleetctl wormhole send` erstellen und müssen nicht selbst pollen oder Git-Operationen ausführen. Keine umfangreichen Skill-Umbauten.

## 8. Sicherheit und Robustheit

- Git-Remotes sind Daten, keine Shell-Befehle. Keine `sh -c`.
- Keine Force-Pushes, keine automatischen Hard-Resets, kein Clean, kein Stash-Pop mit möglichem Verlust.
- Kein automatisches Löschen/Reklonieren eines unerwarteten Checkouts.
- Keine Verarbeitung von Symlink-Dateien oder Pfad-Traversal.
- Nachrichten niemals als lokale Shell-Kommandos interpretieren. `kind: command` aus einem Wormhole darf nicht in `comms type=command` übersetzt werden.
- Eingehende Nachrichten sind externe Inhalte. Herkunft im Comms-Text deutlich machen; keine stillschweigende Autoritätssteigerung gegenüber normalen Fleet-Direktiven.
- YAML-/Markdown-Parser müssen Größenlimits haben (z.B. 1 MiB pro Nachricht) und ungültige Eingaben sauber ablehnen.
- Logs dürfen keine Zugangsdaten aus Remote-URLs enthalten.
- Fehlerbehandlung muss zwischen transient (Retry) und permanent (Config/Protokoll/Permission) unterscheiden.
- Worker-Absturz darf keine Nachricht verlieren oder doppelt zustellen.
- State- und Comms-Updates müssen die oben beschriebene idempotente Reihenfolge einhalten.

## 9. Tests – Abnahmepflicht

Schreibe echte Tests, nicht nur Mock-Aufrufe:

### Config
- Datei fehlt → deaktiviert.
- Beispielkonfiguration wird korrekt geladen.
- Defaults und explizite Overrides stimmen.
- Ungültige/duplizierte IDs, Traversal, leere Remote-/Branch-Angaben, ungültige Enums und Duration werden abgewiesen.
- read-only + outgoing enabled wird abgewiesen.
- Unbekannte optionale Front-Matter-Felder bleiben zulässig.

### Bootstrap/Gitignore
- `/var/wormholes/` wird in neue .starfleet-ai/.gitignore geschrieben.
- Vorhandene Datei wird idempotent ergänzt, lokale Einträge bleiben.
- `git check-ignore` bestätigt Ignore für Checkout-Dateien und State-Dateien.

### Git-Sync
Mit temporären lokalen Bare-Repositories testen:
- Erst-Clone des richtigen Branches.
- vorhandener korrekter Checkout.
- falsches Remote/falscher Branch/nicht-Git-Verzeichnis wird nicht zerstört.
- Fetch/Rebase eines Remote-Commits.
- lokale Outgoing-Datei wird nur in erlaubtem Pfad committed und gepusht.
- fremde Änderung blockiert; sie bleibt bytegenau erhalten.
- read-only führt niemals Commit/Push aus.
- Push-Race wird begrenzt mit Fetch/Rebase/Retry behandelt.
- Rebase-Konflikt erhält lokale Commits und meldet Blockade.
- kein Test darf Force-Push, reset --hard oder clean benötigen.

### Incoming/Comms
- baseline liefert alte Nachrichten nicht aus.
- replay liefert vorhandene Nachrichten genau einmal.
- neue Nachricht wird an korrekten Ship zugestellt.
- Aliase `starfleetctl`/`owner` routen an Fleet-Flagship.
- `all` broadcastet über Comms.
- Decisions gehen an konfiguriertes Ziel.
- falsches/unbekanntes Ziel wird nicht falsch zugestellt.
- kaputtes/partielles Front Matter wird nicht als verarbeitet markiert.
- ID-/Hash-Konflikt wird gemeldet.
- Rebase/erneuter Scan/Restart erzeugt keine Duplikate.
- Fault-Injection direkt nach Comms-Post, aber vor Wormhole-State-Update beweist crash-idempotente Deduplizierung.
- Wormhole-Nachricht mit `kind: command` erzeugt ausschließlich normale Ship-Direktive, niemals Comms-Command.

### Worker/CLI/Service
- Start ist singleton/idempotent; Stop beendet sauber.
- SIGHUP löst sofortigen Sync aus.
- Fehler eines Repositories hindert andere nicht.
- Service-Autostart startet `wormhole` nur bei aktiver Config.
- `status --json` ist maschinenlesbar und stabil.
- CLI-Exitcodes und Help sind getestet.

Abschlussprüfungen: `go test ./...`, `make all` und alle vorhandenen relevanten Repository-Checks. Bei nicht ausführbaren Checks konkret angeben warum; nicht als grün ausgeben.

## 10. Nicht-Ziele

- Keine separate MCP-/HTTP-/Webservice-Infrastruktur.
- Keine neue Datenbank.
- Kein Agent-eigenes Polling.
- Keine Änderung am bestehenden Wormhole-Dateiprotokoll ohne zwingenden Grund.
- Kein automatischer Model-Aufruf zur Konfliktlösung.
- Kein Force-Push.
- Keine Umstellung des bestehenden Task-/Report-Systems auf Wormhole.
- Keine breit angelegte Überarbeitung von Comms, Timer oder Config außerhalb der minimal nötigen Erweiterungen.

## 11. Abschlussbericht und Review

Nach Implementierung:

1. Commit-SHAs und Branch nennen.
2. Vollständige Liste der geänderten Dateien und wesentliche Entscheidungen.
3. Testergebnisse mit konkreten Befehlen/Exitcodes.
4. Nachweis, dass `.starfleet-ai/var/wormholes/` unter allen Installationspfaden ignoriert wird.
5. Beispielkonfiguration, die tatsächlich durch den Config-Loader und Validator läuft.
6. Nachweis des Crash-Fault-Tests gegen doppelte Comms-Zustellung.
7. Offene Einschränkungen klar benennen.
8. Unabhängigen Enterprise-Review anfordern; Laforge bleibt Writer, Reviewer arbeitet read-only.
9. Keine Folgeaufgaben außerhalb dieses Umfangs beginnen, bevor der Review abgeschlossen ist.

Ziel ist eine vollständig benutzbare, robuste Wormhole-Funktion – nicht nur ein Config-Struct oder ein Worker-Prototyp.
