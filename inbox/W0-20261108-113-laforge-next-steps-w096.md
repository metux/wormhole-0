---
id: W0-20261108-113
kind: owner-instruction
from: owner-coordination
to: LaForge
created: 2026-11-08T18:45:00Z
in_reply_to: W0-20261108-111
task_ref: W0-20261107-096
correlation_id: laforge-w096-next-steps-113
---

# LaForge — konkrete nächste Schritte für W0-096

Bitte fahre mit der bestehenden Implementierung auf `mpbt-hq/starfleetctl:work/wormhole-manager-w0-017` fort. Keine neue Architektur und keine zusätzlichen Features. Der zuletzt verifizierte Branch-HEAD war `7ecc11220995ad5e3b61bde49ff95b238a77903f`.

Arbeite die beiden noch offenen Punkte in dieser Reihenfolge ab:

## 1. YAML-Validierung kontextabhängig machen
- Ersetze die globale Key-Allowlist durch erlaubte Schlüssel je YAML-Mapping bzw. Konfigurationsstruktur.
- Ein bekannter Schlüssel am falschen Ort muss als Fehler erkannt werden, nicht stillschweigend durch `yaml.Node.Decode` verschwinden.
- Prüfe auch unbekannte, doppelte und falsch typisierte Schlüssel sowie ungültige verschachtelte Werte.
- Ergänze Regressionstests: mindestens ein Schlüssel, der global bekannt, aber im jeweiligen Mapping unzulässig ist; unbekannter Key; gültige Konfiguration.
- Achte darauf, dass die Validierung auch bei deaktiviertem Worker ausgeführt wird.

## 2. Fehler aus RunLoop zurückgeben
- Prüfe den gesamten Kontrollfluss von `RunLoop` und den Sync-/Ingestion-Aufrufen.
- Aggregierte Fehler dürfen nicht nur geloggt und danach verschluckt werden. Gib sie an den Aufrufer zurück, soweit die Lifecycle-Semantik das zulässt.
- Falls ein dauerhaft laufender Worker Fehler bewusst intern behandelt, dokumentiere die konkrete Grenze und stelle sicher, dass der Aufrufer einen geeigneten Fehler-/Statuskanal erhält.
- Ergänze Tests, die nachweisen, dass Sync- und Ingestion-Fehler den vorgesehenen Aufrufer erreichen und nicht nur im Log auftauchen.

## Commit- und Prüfregeln
- Vor Änderungen: Branch, HEAD, Merge-Base und `git status --short` dokumentieren; vorhandene Änderungen erhalten.
- YAML-Validierung und RunLoop-Fehlerweitergabe als getrennte, kleine Commits mit jeweils passenden Regressionstests umsetzen.
- Nur explizit benötigte Pfade committen; keine Broad-Globs.
- Tests tatsächlich ausführen und die exakten Befehle samt Ergebnissen festhalten.
- Danach Push-SHA und Parent-SHA melden. Keine Erfolgsmeldung ohne Testergebnis.
- Kein Reset, Force-Push, Stash oder Verwerfen vorhandener Arbeit.

Bitte beginne mit Punkt 1 und melde nach dessen Commit einen kurzen Checkpoint, bevor du Punkt 2 angehst. W0-111 ist die begleitende Checkpoint-Anfrage; diese Nachricht präzisiert die Reihenfolge und den erwarteten Umfang, ersetzt W0-096/W0-101 nicht.
