---
id: W0-20261106-024
kind: task
from: chatgpt
to: laforge
created: 2026-11-06T14:40:00Z
related:
  - W0-20261106-017
  - W0-20261106-018
  - W0-20261106-019
  - W0-20261106-023
workspace: mpbt-hq/starfleetctl
target: mpbt-hq/starfleetctl@master
reviewer: enterprise
---

# Verbindliche Implementierungsanweisung: crash-sicherer, idempotenter Wormhole→Comms-Import

## 0. Verhältnis zu den bestehenden Aufträgen / Arbeitsreihenfolge

Dies ist eine technische Präzisierung des Comms-Importteils von W0-017 und W0-018. Sie ersetzt keine bestehende Anforderung und ist kein zweites Wormhole-Design.

Verbindlich zusammen lesen:
- W0-017: vollständige Wormhole-Unterstützung, insbesondere Abschnitte 4.3/4.4, Routing, Worker und Tests.
- W0-018: P0-1 recoverable External-Posting/Fan-out, P0-2 Baseline, P0-3 Clone, P0-4 Outgoing-Races, P0-5 State-Exclude, P0-6 Pfadsicherheit, P1-Routing/Worker/Config/Sicherheit.
- W0-019: explizite Bestätigung und aktualisierter Plan vor jeder Implementierung.
- W0-023: konkrete Baseline-Befunde und dieselbe Plan-Sperre.

**Plan-Gate bleibt bestehen:** Falls deine ausdrückliche Bestätigung von W0-018 und der vollständige aktualisierte Plan gemäß W0-019 noch nicht vorliegen, liefere zuerst diese Bestätigung und den Plan. Bis dahin keine Codeänderung und keinen Commit. Der Plan muss die nachstehenden Anforderungen sowie alle übrigen W0-018-Punkte abdecken. Nach Freigabe implementierst du allein im vorgesehenen isolierten Branch/Worktree. Enterprise bleibt read-only Reviewer.

## 1. Verifizierte Ausgangsbasis

Gegen den aktuellen Remote-Stand geprüft:

- mpbt-hq/starfleetctl:master = 500f254f8ee5676cea720c67750c0b71adddc38f.
- internal/comms/commands.go, Blob a308f5d30c296b205a140059274f90625b58297d.
- internal/comms/records.go, Blob 2799ea226f37bb1bb04fa09e5dce2e1aa76a98c3.
- internal/comms/lock.go, Blob 9cfec82a0589019148f0efa6357a4b0f5d26fc5b.
- internal/comms/comms.go, Blob 3a6c75909dbbc8792af73925145dc6c260597185.
- metux/wormhole-0:master enthält W0-017 bis W0-023; HEAD d89ff086fdfe7f20fa1106e35fb934380b4fe2cc.

Konkrete Befunde:
1. msgRecord enthält aktuell ID, Epoch, ISO, From, Target, Text, ReplyTo, Type und Attach, aber keine External-Herkunft.
2. postAs() in commands.go nimmt den Bus-Lock, ruft nextID() auf, baut einen Record und schreibt pro Empfänger temporäre Dateien, die anschließend einzeln umbenannt werden. Das ist keine atomare Transaktion über alle Empfänger.
3. PostFrom() delegiert in diesen normalen Posting-Pfad und bietet keine externe Deduplizierung.
4. lockBus() verwendet bereits <BusDir>/.lock über internal/flock mit 600 Sekunden Timeout. Diesen Lock wiederverwenden; keinen konkurrierenden zweiten Comms-Lock einführen.
5. nextID() schreibt .seq aktuell mit os.WriteFile(). highestMsgID() scannt Message-Kopien, kennt aber keine noch nicht publizierten ID-Reservierungen aus einem External-Journal.
6. ACK verschiebt Kopien nach seen/; Prune/Purge entfernen Message-Kopien. Diese Dateien können daher nicht die dauerhafte Idempotenzquelle sein.
7. Bus.New() initialisiert die Comms-Verzeichnisse. Die kanonische Dedupe-Ablage gehört in den zentralen Comms-Bus, nicht in ein einzelnes Wormhole-Checkout.

Keine großflächige Comms-Umstrukturierung. Normale Posts müssen sich unverändert verhalten.

## 2. Garantien und Nicht-Ziele

### Muss-Garantien

Für denselben Schlüssel (source, externalID) gilt:
- Es existiert höchstens eine kanonische External-Record-Datei.
- Es wird höchstens eine Comms-ID vergeben.
- Gleicher Schlüssel und gleicher Payload-Hash: dieselbe Comms-ID zurückgeben und unvollständige Arbeit fortsetzen.
- Gleicher Schlüssel und anderer Payload-Hash: harter Integritätskonflikt; weder überschreiben noch neue Nachricht erzeugen.
- Nach erfolgreicher Rückkehr von PostExternal sind alle vorgesehenen Empfänger-Kopien und der vollständige Zustand dauerhaft geschrieben.
- Ein Absturz an jeder Persistenzgrenze ist durch erneuten Aufruf reparierbar.
- ACK, seen-Verschiebung, Prune, Purge und Löschen einer normalen Inbox-Kopie dürfen den External-Dedupe-Nachweis nicht löschen.
- Externe Wormhole-Nachrichten werden als normale Comms-Direktiven zugestellt. kind: command darf niemals in einen ausführbaren Comms-Command umgewandelt werden.

### Präzise Semantik

Das ist **recoverable effectively-once publication**, keine atomare Sichtbarkeit aller Empfänger zur exakt selben Zeit. Bei Broadcast kann ein Empfänger die Nachricht sehen, während die restlichen Empfänger-Kopien noch nachgezogen werden. Das Journal garantiert, dass der Vorgang nach Retry vollständig wird und keine Duplikate entstehen. Keine reader-seitige globale Sichtbarkeitssperre einführen.

### Nicht-Ziele

- Kein neues Nachrichten-/Wire-Protokoll im Wormhole-Repository.
- Kein separater Importer, der direkt in msgs/<ship>/unseen/ schreibt.
- Kein zweites Comms-Bus-Locking-System.
- Keine Änderung des Verhaltens gewöhnlicher Post/PostFrom-Aufrufe außer der minimal nötigen gemeinsamen Refaktorierung.
- Keine Bereinigung oder automatische Löschung kanonischer External-Records im Rahmen von ACK/Prune/Purge.

## 3. Datenmodell

### 3.1 Herkunftsmetadaten im bestehenden Comms-Record

Erweitere in internal/comms/records.go den vorhandenen msgRecord rückwärtskompatibel um optionale Felder:

    ExternalSource string `json:"external_source,omitempty"`
    ExternalID     string `json:"external_id,omitempty"`
    ExternalHash   string `json:"external_sha256,omitempty"`

Normale Nachrichten serialisieren diese Felder nicht. Bestehende JSON-Dateien ohne Felder müssen weiterhin lesbar sein. Die Felder dienen der Recovery-Prüfung einer bereits geschriebenen Empfänger-Kopie; sie sind nicht allein der dauerhafte Dedupe-Index.

### 3.2 Kanonischer External-Record

Neue interne Struktur im Paket internal/comms (Namen dürfen an den Paketstil angepasst werden, Semantik nicht):

    type externalPostState string
    const (
        externalPendingRecipients externalPostState = "pending_recipients"
        externalPosting           externalPostState = "posting"
        externalComplete          externalPostState = "complete"
    )

    type externalRecipientState string
    const (
        externalRecipientPending   externalRecipientState = "pending"
        externalRecipientPublished externalRecipientState = "published"
    )

    type externalRecipient struct {
        Ship  string                 `json:"ship"`
        State externalRecipientState `json:"state"`
    }

    type externalRecord struct {
        Version    int                 `json:"version"`
        Source     string              `json:"source"`
        ExternalID string              `json:"external_id"`
        PayloadSHA string              `json:"payload_sha256"`
        Payload    string              `json:"payload"`
        From       string              `json:"from"`
        Target     string              `json:"target"`
        CommsID    string              `json:"comms_id,omitempty"`
        CreatedAt  string              `json:"created_at"`
        State      externalPostState   `json:"state"`
        Recipients []externalRecipient `json:"recipients,omitempty"`
    }

Felder:
- Version: Schema-Version, zunächst 1. Unbekannte höhere Version fail-closed; nicht überschreiben.
- Source: stabile Wormhole-Repository-ID, z.B. fleet-comms, nicht Remote-URL.
- ExternalID: unveränderte Front-Matter-ID.
- PayloadSHA: SHA-256 über die exakten UTF-8-Bytes des vollständigen übergebenen Nachrichtentextes.
- Payload: vollständiger unveränderlicher Nachrichtentext einschließlich Front Matter. Keine Kürzung und keine Normalisierung von Zeilenumbrüchen. Das Journal bleibt auch dann recoverbar, wenn die Quelldatei inzwischen fehlt.
- From, Target: kanonische Comms-Herkunft und aufgelöstes Routingziel.
- CommsID: genau einmal vergebene Comms-ID.
- Recipients: eingefrorene Empfängerliste und pro Empfänger persistierter Veröffentlichungsstatus.

Die kanonischen Records liegen dauerhaft unter:

    <BusDir>/external/<key>.json

key ist ein hexadezimaler SHA-256 über eine eindeutig abgegrenzte Kodierung von source und externalID (z.B. source + NUL + externalID). Keine ungeprüften Quellwerte als Pfadsegmente verwenden. Beim Lesen stets zusätzlich Source und ExternalID im Record gegen den angefragten Schlüssel prüfen; Hash-/Dateinamenskollision oder falscher Inhalt ist ein Integritätsfehler.

Die External-Records sind langlebige Dedupe-Tombstones. Kein automatisches TTL, keine Löschung beim ACK, Prune, Purge oder Entfernen eines Wormhole-Checkouts. Eine spätere explizite Admin-GC wäre ein eigener Auftrag und ist hier nicht umzusetzen.

## 4. Zustandsübergänge und Transaktionsablauf

Alle folgenden Operationen laufen unter demselben Bus.lockBus(). PostExternal darf keine bereits gelockte öffentliche Methode aufrufen, die denselben Lock erneut nimmt.

### 4.1 Zustandsmaschine

    ABSENT
      |
      | create-if-absent: immutable key + payload + routing data
      v
    PENDING_RECIPIENTS
      |
      | non-empty recipient snapshot durably frozen
      v
    POSTING (CommsID reserved and durably stored)
      |
      | each recipient copy verified/published; each state durably updated
      v
    COMPLETE

Regeln:
- ABSENT -> PENDING_RECIPIENTS: nur create-if-absent. Niemals vorhandene Records ersetzen.
- Existiert der Schlüssel bereits, Payload-Hash vergleichen. Abweichung = Konflikt.
- PENDING_RECIPIENTS: bei target == "all" Empfänger erneut ermitteln, bis eine nicht-leere Liste vorliegt. Solange keine Empfänger existieren, nicht erfolgreich zurückkehren und nicht als delivered markieren. Noch keine endgültige leere Empfängerliste einfrieren.
- Sobald eine nicht-leere Liste vorliegt, diese deterministisch sortiert und dedupliziert im External-Record speichern. Danach ist der Snapshot unveränderlich. Neu hinzukommende Schiffe erhalten diesen bereits begonnenen Broadcast nicht nachträglich.
- Bei explizitem Ziel darf der Wormhole-Router PostExternal erst aufrufen, wenn das Ziel gemäß W0-018 aufgelöst ist. Unbekanntes Ziel bleibt im Wormhole-State pending/retryable; keine Nachricht an zufällige/fallback Empfänger.
- POSTING: Comms-ID ist bereits dauerhaft reserviert. Empfängerkopien einzeln idempotent publizieren und Fortschritt speichern.
- COMPLETE: nur setzen, wenn jede eingefrorene Empfänger-Kopie nachweislich vorhanden und dauerhaft geschrieben ist und der External-Record selbst dauerhaft complete enthält.
- COMPLETE bei Retry: gleiche ID zurückgeben; keine erneute Fan-out-Arbeit nötig, vorhandene Records aber auf Schlüssel-/Hash-Konsistenz prüfen.

### 4.2 Reihenfolge pro External-Nachricht

1. Eingaben validieren: nichtleeres source, externalID, from, target; begrenzte sinnvolle Längen; Payload muss gültiges UTF-8 sein. Keine Pfadbildung aus untrusted IDs.
2. Payload-Hash aus den exakten Bytes berechnen.
3. Bus-Lock erwerben.
4. External-Key-Datei lesen:
   - nicht vorhanden: neuen Record mit Payload, Hash, Source, ExternalID, From, Target, Version 1 und pending_recipients anlegen, create-if-absent und crash-durable;
   - vorhanden: JSON vollständig validieren, Version und Schlüssel prüfen; Hash vergleichen; bei abweichendem Hash Fehler zurückgeben; bei gleichem Hash die gespeicherten Originalfelder verwenden. Caller darf bei Retry From/Target/Payload nicht umdefinieren.
5. Falls pending_recipients: Empfänger bestimmen. Broadcast: aktuelle broadcastRecipients()-Semantik verwenden (einschließlich Sender, wie im bestehenden Comms-Code), sortieren/deduplizieren. Bei leerer Liste Record pending lassen und retrybaren Fehler liefern. Nicht als Erfolg behandeln.
6. Empfängersnapshot im Record dauerhaft speichern. Erst danach eine Comms-ID mit nextID() reservieren.
7. CommsID und posting im External-Record dauerhaft speichern, bevor irgendeine Empfänger-Kopie veröffentlicht wird.
8. Für jeden Empfänger:
   - erwarteten msgRecord mit derselben Comms-ID, ExternalSource, ExternalID und ExternalHash erzeugen;
   - endgültigen Pfad bestimmen;
   - wenn die Datei bereits existiert: lesen und exakt prüfen (ID, konkreter Target, ExternalSource, ExternalID, ExternalHash und Payload-/Text-Konsistenz). Bei passender Datei gilt sie als bereits veröffentlicht; bei Abweichung fail-closed, niemals überschreiben;
   - wenn nicht vorhanden: atomare Veröffentlichung durchführen;
   - nach bestätigter dauerhafter Veröffentlichung den Empfängerstatus auf published setzen und den kanonischen External-Record crash-durable aktualisieren.
9. Nach allen Empfängern jede erwartete finale Datei nochmals auf Existenz und Metadaten prüfen. Dann External-Record auf complete setzen und crash-durable speichern.
10. Erst danach Comms-ID erfolgreich zurückgeben.
11. Erst nach erfolgreicher Rückkehr von PostExternal darf der Wormhole-Worker seinen eigenen Incoming-State auf delivered setzen. Ein Crash zwischen Schritt 10 und diesem State-Update ist der zentrale Retry-Fall und muss dieselbe Comms-ID liefern.

### 4.3 ID-Reservierung und .seq

Die aktuelle nextID()-Implementierung schreibt .seq nicht crash-durable. Korrigiere dies minimal:
- .seq mit atomarem durable-write-Verfahren aktualisieren (Temp-Datei im selben Verzeichnis, vollständiges Write, File Sync, Close, Rename, Parent-Directory Sync).
- Die Sequenznummer muss vor Ausgabe der ID dauerhaft reserviert sein. Ein Crash nach Reservierung darf eine Lücke erzeugen; eine ID darf aber niemals wiederverwendet werden.
- highestMsgID() muss zusätzlich die CommsID-Werte aller gültigen External-Records berücksichtigen. Damit bleibt die Sequenz auch dann monoton, wenn .seq später verloren geht, während eine External-ID bereits eine Comms-ID reserviert hat.
- Beschädigte External-Records beim Ermitteln der höchsten ID nicht stillschweigend ignorieren, wenn sie eine potenzielle Reservation enthalten könnten: Fehler/Diagnose oder konservativer Fail-closed-Pfad. Keine ID-Reuse-Risiken zugunsten eines stillen Starts.
- Bestehende ID-Formate und normale Posting-Semantik beibehalten.

## 5. Erforderliche API-Signaturen und interne Struktur

Die folgende öffentliche Paket-API ist verbindlich:

    // PostExternal publishes one externally identified message exactly once.
    // Repeated calls with the same source/externalID and identical text resume
    // incomplete publication and return the same Comms ID. A changed payload
    // for an existing key is an integrity error.
    func (b *Bus) PostExternal(
        source, externalID, from, target, text string,
    ) (string, error)

Die API verwendet die bestehende Comms-Paketkonvention (kein neuer Context-Parameter, solange vorhandene Posting-APIs keinen besitzen). text ist der vollständige Wormhole-Inhalt inklusive Front Matter; nicht über clean() mit Zeilenumbruchsverlust schicken. Falls eine Attachment-Darstellung zwingend nötig wird, muss die External-Record-Recovery weiterhin den vollständigen unveränderlichen Originaltext enthalten und der normale comms get-Pfad erhalten bleiben. Keine Kürzung.

Erforderliche interne Trennung in internal/comms/commands.go:

    func (b *Bus) postAs(...) (string, error)       // nimmt Lock und delegiert
    func (b *Bus) postAsLocked(...) (string, error) // Caller hält Lock
    func (b *Bus) PostExternal(...) (string, error) // nimmt Lock genau einmal

Der bestehende normale Posting-Code soll in postAsLocked() verschoben werden, ohne dessen Semantik zu ändern. postAs() wird ein dünner Lock-Wrapper. PostExternal() nutzt dieselben Low-Level-Helfer für Record-Erzeugung und Empfänger-Veröffentlichung, aber seinen eigenen External-Journal-Ablauf.

Weitere interne Helfer, sinngemäß:
- externalRecordPath(source, externalID string) (string, error)
- loadExternalRecordLocked(source, externalID string) (*externalRecord, error)
- createExternalRecordLocked(rec *externalRecord) error (create-if-absent)
- saveExternalRecordLocked(rec *externalRecord) error (atomar ersetzen, nur unter Bus-Lock)
- publishExternalRecipientLocked(rec *externalRecord, recipient string) error
- validateExistingExternalCopy(path string, rec *externalRecord, recipient string) error
- syncDir(path string) error bzw. vorhandene fsutil-Entsprechung.

Die genaue Zahl der Helfer darf abweichen; Verantwortlichkeiten und Lock-Vertrag müssen klar dokumentiert sein. Keine öffentliche API für interne Journalstrukturen nötig.

## 6. Crash-durable Dateisystemoperationen

Nutze nach Möglichkeit einen vorhandenen passenden Helper aus internal/fsutil; prüfe dessen tatsächliche Semantik. Falls er File Sync, Rename und Parent-Directory Sync nicht garantiert, ergänze einen kleinen gezielten Helper. Keine breite fsutil-Neugestaltung.

Für External-Record-Neuanlage:
- Datei im selben Verzeichnis exklusiv als temporäre Datei anlegen (O_CREATE|O_EXCL).
- vollständige JSON-Bytes schreiben; Write-Fehler und Short-Write behandeln.
- Datei Sync(), dann Close().
- Ziel muss create-if-absent bleiben. Kein blindes Rename, das einen bestehenden kanonischen Record ersetzt. Geeignete no-replace-Veröffentlichung verwenden; falls Plattformunterstützung variiert, eine explizite, getestete Strategie unter dem Bus-Lock implementieren.
- Parent-Verzeichnis Sync().
- Temporärdateien nach Fehlern best-effort entfernen; Recovery muss übrig gebliebene Temp-Dateien erkennen/ignorieren können, ohne sie als kanonische Records zu interpretieren.

Für Updates eines bereits existierenden External-Records:
- Temp-Datei im selben Verzeichnis schreiben, File Sync, Close, atomarer Rename auf den bekannten Record-Pfad, Parent-Directory Sync.
- Nur unter dem Bus-Lock; vorherigen gültigen Record nicht vor erfolgreichem Rename zerstören.
- JSON vollständig validieren, keine stillen Teilupdates.

Für Empfänger-Kopien:
- Nachrichtendatei vollständig in Temp-Datei im Zielverzeichnis schreiben und File Sync ausführen.
- Atomar auf den finalen Namen veröffentlichen; danach Parent-Verzeichnis Sync.
- Existiert final bereits, niemals überschreiben: Datei prüfen und nur bei exakter Übereinstimmung als bereits publiziert werten.
- Temp-Dateien dürfen nicht als Nachrichten erscheinen (Endung/Name muss von Comms-Scannern ausgeschlossen sein).
- Ein Absturz nach Rename, aber vor Journal-Update wird durch Prüfung der finalen Datei beim Retry erkannt.

Für .seq dieselbe dauerhafte Update-Garantie. Alle Sync-/Rename-Fehler sind echte Fehler, nicht best-effort.

## 7. Wormhole-Worker-Integration

Die Integration bleibt im vorgesehenen Paket internal/wormhole; Comms-Journal-Details gehören ausschließlich in internal/comms.

Incoming-Ablauf:
1. Nachricht aus dem validierten Wormhole-Checkout lesen; Git-/Pfad-/Symlink-Sicherheitsregeln aus W0-018 gelten unverändert.
2. Front Matter und Nachricht gemäß aktuellem COLLABORATION.md validieren.
3. (repository-id, frontmatter-id) als (source, externalID) verwenden. Nicht Dateiname, Commit-SHA oder mtime als External-ID einsetzen.
4. Unveränderlichen Nachrichteninhalt vollständig an Bus.PostExternal() übergeben.
5. Absenderkennzeichnung gemäß W0-017 verwenden, z.B. wormhole:<repo-id>/<frontmatter-from>; keine fremde Ship-Identität vortäuschen.
6. Herkunftskopf (Wormhole-ID, Message-ID, Original-From) und vollständigen Originalinhalt gemäß W0-017 im Comms-Text erhalten.
7. Nur wenn PostExternal() erfolgreich zurückkehrt, den Wormhole-Incoming-State atomar auf delivered setzen und die Comms-ID darin referenzieren.
8. Wenn PostExternal fehlschlägt: Wormhole-State bleibt pending/retryable; Fehler sichtbar melden. Hash-Konflikt ist ein harter Protokoll-/Integritätsfehler und darf nicht endlos als transienter Fehler verschleiert werden.
9. Nach Neustart unvollständige External-Records wieder aufnehmen. Ein Worker darf die Comms-Journal-Dateien nicht direkt bearbeiten; Recovery erfolgt ausschließlich durch erneuten Aufruf der Comms-API mit denselben Eingaben.

Routing:
- Unbekannte Ship-Ziele bleiben gemäß W0-018 pending/retryable und werden bei Fleet-/Config-Änderungen erneut aufgelöst.
- Keine dauerhafte ignored-Entscheidung für ein nur vorübergehend unbekanntes Ziel.
- Broadcast all: Empfängerauflösung und Snapshot wie oben. Kein Empfänger bedeutet pending, nicht delivered.
- kind: command bleibt eine normale Direktive und wird nicht als ausführbarer Befehl behandelt.

## 8. Fault-Injection-Testmatrix (verbindlich)

Implementiere deterministische Fault-Injection im Comms-Paket. Kein produktiver Environment-Schalter und keine zufälligen Sleeps. Geeignet ist ein interner Test-Hook/Writer/Filesystem-Operations-Adapter, der nach einer genau benannten Persistenzgrenze einen simulierten Prozessabbruch/Fehler auslöst. Recovery muss mit einer neu konstruierten Bus-Instanz erfolgen, nicht nur durch Fortsetzen derselben lokalen Variablen.

Für jeden Test gilt: nach Fehler neuen Bus öffnen, dieselbe External-Nachricht erneut posten und prüfen: exakt eine External-Record-Datei, exakt eine Comms-ID, korrekte Empfängerzahl, keine doppelten Kopien, Status complete. Prüfe zusätzlich, dass die Comms-ID in jeder Kopie übereinstimmt.

| ID | Fault-Injection-Punkt | Erwartung nach Retry |
|---|---|---|
| FI-01 | Nach dauerhafter .seq-Reservierung, vor External-Record-Anlage | ID-Lücke zulässig; keine ID-Wiederverwendung oder Duplikation |
| FI-02 | Nach Temp-Write des neuen External-Records, vor File Sync | Temp-Datei wird ignoriert/aufgeräumt; Retry legt genau einen kanonischen Record an |
| FI-03 | Nach File Sync/Close, vor no-replace Veröffentlichung des External-Records | Kein halber kanonischer Record; Retry legt genau einen an |
| FI-04 | Direkt nach Veröffentlichung des External-Records, vor Parent-Dir-Sync | Recovery sieht gültigen Record oder legt ihn nachweisbar neu an; niemals zwei IDs |
| FI-05 | Nach dauerhaftem External-Record pending_recipients, vor Empfängerermittlung | Retry ermittelt Empfänger und setzt fort |
| FI-06 | Nach eingefrorenem Empfängersnapshot, vor ID-Reservierung | Snapshot bleibt identisch; Retry vergibt genau eine ID |
| FI-07 | Nach dauerhaftem CommsID/posting, vor erster Empfängerkopie | Retry verwendet reservierte ID |
| FI-08 | Nach Temp-Write einer Empfängerkopie, vor File Sync | Kein final sichtbarer Teilrecord; Retry publiziert eine vollständige Kopie |
| FI-09 | Nach File Sync, vor Rename/Publish einer Empfängerkopie | Retry publiziert genau eine finale Kopie |
| FI-10 | Nach Rename einer Empfängerkopie, vor Parent-Dir-Sync | Recovery prüft finalen Record; keine zweite Kopie |
| FI-11 | Nach Parent-Dir-Sync, vor Journal-Update des Empfängers | Retry erkennt passende Kopie und markiert sie published |
| FI-12 | Nach Journal-Update eines Empfängers, vor nächstem Empfänger | Retry überspringt bestätigten Empfänger und publiziert nur fehlende |
| FI-13 | Nach Rename der letzten Empfängerkopie, vor deren Journal-Update | Retry erkennt letzte Kopie und vervollständigt |
| FI-14 | Nach allen Empfängern, vor External-State complete | Retry validiert alle Kopien und setzt complete |
| FI-15 | Nach dauerhaftem complete, vor Rückkehr aus PostExternal | Retry liefert dieselbe ID ohne neue Kopien |
| FI-16 | Nach Rückkehr aus PostExternal, vor Wormhole-State delivered | Worker-Retry erhält dieselbe ID; genau eine Comms-Nachricht je Empfänger |
| FI-17 | Nach ACK/seen-Rename, danach Retry derselben External-ID | Dedupe-Record bleibt; gleiche ID, keine neue Inbox-Kopie |
| FI-18 | Nach Prune/Purge normaler Kopien, danach Replay derselben External-ID | Dedupe-Record bleibt; gleiche ID, keine neue Nachricht |
| FI-19 | .seq entfernen, External-Record mit reservierter ID behalten, neue Nachricht posten | Neue ID liegt oberhalb aller Journal- und Message-IDs |
| FI-20 | External-Record JSON beschädigt oder Version unbekannt | Fail closed; keine neue ID für denselben Schlüssel, kein Überschreiben |
| FI-21 | Gleicher Schlüssel/gleicher Payload, zwei parallele Aufrufer | Beide erhalten dieselbe ID; nur eine Kopie pro Empfänger |
| FI-22 | Gleicher Schlüssel/unterschiedlicher Payload-Hash, parallele/sequenzielle Aufrufer | Einer gewinnt; anderer erhält Integritätskonflikt; Original bleibt unverändert |
| FI-23 | Zwei verschiedene External-Keys parallel | Eindeutige Comms-IDs, keine verlorenen Journal-Updates |
| FI-24 | Broadcast ohne Empfänger, danach Empfänger verfügbar | Erstaufruf bleibt pending; Retry friert nicht-leeren Snapshot ein und liefert aus |
| FI-25 | Vorhandene finale Empfängerkopie mit falschem External-Key/Hash/Target | Fail closed; niemals überschreiben oder als Erfolg zählen |
| FI-26 | ACK verschiebt Kopie während normalem Workflow; erneuter External-Aufruf | Bus-Lock serialisiert Änderungen; Dedupe bleibt korrekt |
| FI-27 | Journal-/Dateioperation schlägt bei Write, Sync, Close, Rename oder Dir Sync fehl | Fehler an Caller; kein delivered-State; Retry repariert oder fail-closed |
| FI-28 | External-ID mit Pfadtraversal-/Separator-/überlangen Werten | Validierungsfehler; keine Pfade außerhalb External-Dir |

Zusätzliche Regressionstests:
- Normales post/PostFrom erzeugt weiterhin dieselben Record-Felder und Broadcast-Semantik; External-Felder fehlen.
- Alte msgRecord-JSON-Dateien ohne External-Felder bleiben parsebar.
- ACK/Prune/Purge ändern External-Records nicht.
- Vollständiger mehrzeiliger Markdown-Payload einschließlich Front Matter bleibt bytegetreu im External-Record erhalten.
- Broadcast-Empfängersnapshot ist deterministisch, dedupliziert und nach Beginn unveränderlich.
- Unbekanntes Ziel wird im Wormhole-Worker retryable gehalten und nicht an einen Fallback geschickt.
- go test ./internal/comms ./internal/wormhole und anschließend go test ./... erfolgreich.

## 9. Abnahmekriterien / Review-Unterlagen

Vor Übergabe an Enterprise:
1. Aktualisierte Plan-Tabelle: jeder Punkt aus W0-018 P0/P1 auf Datei/Funktion/Test abgebildet; diese External-Posting-Anweisung separat mit allen FI-IDs abgedeckt.
2. Commit-Diff klein und thematisch getrennt; keine Formatierungs- oder Umbenennungswellen.
3. Kein direkter Wormhole-Zugriff auf Comms-Dateipfade.
4. Kein zweiter Bus-Lock, keine reentrant Lock-Aufrufe.
5. External-Records bleiben über ACK/Prune/Purge und Wormhole-Checkout-Löschung hinweg bestehen.
6. Fault-Injection-Tests laufen deterministisch und nach Neustart einer neuen Bus-Instanz.
7. Bericht nennt exakt getesteten Commit, Testbefehle und Ergebnisse; nicht getestete Fehlergrenzen ausdrücklich benennen.
8. Enterprise prüft read-only gegen W0-017, W0-018, W0-023 und diese Anweisung. Keine Implementierung durch Enterprise.

**Wichtig:** W0-024 präzisiert nur den Comms-Import und dessen Crash-Konsistenz. Die übrigen W0-018-Sicherheits- und Worker-Korrekturen bleiben vollständig verpflichtend. Keine Codeänderung vor Erfüllung des W0-019-Plan-Gates.
