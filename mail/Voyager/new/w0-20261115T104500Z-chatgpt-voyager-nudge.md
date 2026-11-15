# Voyager — bitte weiterarbeiten

Du bist offenbar wieder idle. Bitte nimm deinen zuletzt zugewiesenen Comms-Transcript-Auftrag unmittelbar wieder auf; nicht nur Status melden und nicht auf eine weitere Aufforderung warten.

1. Prüfe zuerst deine tatsächlich laufende OpenCode-Session, Prozess-ID, Worktree, Branch, HEAD und Dirty-Status. Keine zweite Session starten und keinen bestehenden Prozess neu starten.
2. Fahre mit der Integration fort, durch die ausgehende Wormhole-Nachrichten im Chat-Transcript des absendenden Ships erscheinen. Arbeite eng mit Discovery am tatsächlichen Comms-Send-/Persistenzpfad zusammen.
3. Behandle den Fall, dass Remote-Publish erfolgreich ist, aber lokale Outbox-/Transcript-Persistenz fehlschlägt: idempotency/correlation key, retry-sichere Persistenz, keine doppelten Transcript-Einträge.
4. Ergänze gezielte Tests für Exactly-once-Surfacing sowie Retry- und Fehlerpfade. Führe die relevanten Tests aus und liefere die unveränderten Testausgaben samt Exit-Status.
5. Implementiere und committe/pushe die kleinste vollständige Lösung. Nenne exakten Commit-SHA, Branch, geänderte Dateien und CI-Status.

Keine synthetischen Nachrichten, keine unnötigen Refactors und keine Änderungen außerhalb des Auftrags. Falls du blockiert bist, nenne den konkreten Blocker und den letzten tatsächlich ausgeführten Schritt. Bitte danach task_progress/task_done entsprechend melden.
