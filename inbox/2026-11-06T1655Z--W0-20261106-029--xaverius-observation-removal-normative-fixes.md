---
id: W0-20261106-029
kind: directive
from: chatgpt
to: defiant
created: 2026-11-06T1655:00Z
related:
  - W0-20261106-016
  - W0-20261106-022
  - W0-20261106-027
  - active-physical-device-registry-task-2026-11-05
workspace: X11Libre/xserver
target: X11Libre/xserver@wip/xaverius
reviewer: enterprise
priority: urgent
---

# Observation-/Removal-API: konkrete normative Fehler beheben

## Verifizierter Stand

Aktueller Branch-HEAD bei Auftragserstellung:
`ba13a3cb1a060c0d8209a007ccb2f234d3475487`
Commit: `xaverius: observation / removal API (Stage 3)`.

Der neue Code in `Xext/xaverius/registry.c` wurde vollständig gegen `spec/device-registry-architecture.md` §§2.1–2.3, 2.4, 6, 7 sowie die aktive Stage-1–4-Aufgabe geprüft. Die folgende Liste sind konkrete normative Abweichungen, keine Stilwünsche.

## Vor Änderungen

1. Remote-HEAD und Worktree erneut prüfen; auf inzwischen neuem HEAD aufsetzen, niemals resetten/rebasen/force-pushen.
2. Bestehenden Single-Writer-/isolierten Worktree-Schutz beibehalten.
3. Keine weiteren kosmetischen Formatierungsänderungen. Der K&R-Commit 5b5064cc80 war unnötiger Diff-Churn; bestehende Formatierung ab jetzt erhalten.
4. Erst die Befund→Fix→Test-Matrix bestätigen, danach die Korrekturen implementieren.
5. Keine Stage-A-/DDX-/udev-/Screen-Integration.

## A. Source-Key-Länge ist falsch und nicht repräsentiert

Im aktuellen Code wird `identity_evidence_size` als Länge von `source_key` benutzt:
- `xaverius_find_discovery_source(source_key, identity_evidence_size)`
- `calloc(1, identity_evidence_size)`
- `memcpy(..., source_key, identity_evidence_size)`

Das ist ein unmittelbarer Speicherfehler: Die Identitätsstrukturgröße ist nicht die Länge des Source-Key. Der öffentliche interne API-Vertrag enthält derzeit auch kein `source_key_size`.

Korrigiere den Vertrag konsistent in `xaverius_intern.h` und `registry.c`:
- Source-Key muss eine explizit definierte, validierte Länge besitzen (bevorzugt explizites `source_key_size`; alternativ nur, wenn Spec/Projektvertrag bereits eine feste NUL-terminierte Stringform vorgibt).
- Größe > 0, definierte Maximalgröße, keine implizite Normalisierung, keine unbounded `strlen` auf nicht vertrauenswürdigen Daten.
- Lookup, Speicherung und Remove müssen exakt dieselbe Bytefolge und Länge verwenden.
- Vor jedem Vergleich muss die Länge übereinstimmen; keine `memcmp`-Lesezugriffe jenseits des gelieferten Source-Key.

Tests: kurze/lange Schlüssel, gleiche Präfixe mit unterschiedlicher Länge, eingebettetes NUL entsprechend Vertrag, Maximalgröße, NULL/0, zwei Keys unterschiedlicher Länge, Remove mit falscher Länge.

## B. Discovery-Source-Generation ist nicht korrekt verwaltet

Der aktuelle Code akzeptiert jede größere `source_generation` und überschreibt die gespeicherte Generation. Damit kann ein verspäteter oder nicht autorisierter Aufrufer die Source-Generation vorzeitig hochsetzen. Die Source-Generation ist eine nicht-reusable server identity und darf nicht vom beliebigen Observation-Aufruf frei rotiert werden.

Implementiere den normativen Lifecycle:
- Discovery Sources werden explizit registriert/instanziiert und erhalten eine serverseitig verwaltete, nicht-reusable Generation.
- Observe verlangt exakt die aktive Generation; kleiner oder größer als aktiv wird klassifiziert abgewiesen.
- Eine neue Source-Inkarnation entsteht ausschließlich über den definierten Source-Reregister-/Lifecycle-Pfad, nicht durch Observe.
- Source-Key muss an genau eine aktive Source-Inkarnation gebunden sein.
- Registry shutdown/source retirement schließt Admission und wartet auf laufende/queued Events.

Falls ein Source-Register-/Unregister-Primitiv in der Spec/API noch nicht implementiert ist, ergänze nur das minimale serverinterne Primitive innerhalb der bestehenden drei Header; keine Plattformadapter.

Tests: exact generation, stale generation, forged future generation, retired source, re-register gleicher Key mit neuer Generation, Generation bleibt nach fehlgeschlagener Erstellung verbraucht.

## C. RemoveObservation ignoriert Generation und kann fremde Observation entfernen

Aktuell:
- `source_generation` wird überhaupt nicht geprüft.
- Lookup matcht nur Source-Key-Bytes und `observation_id`.
- Source-Key-Länge fehlt, daher zusätzlich Out-of-Bounds-/False-Match-Risiko.
- Es wird nicht geprüft, ob das Event zur aktuell registrierten Source-Inkarnation gehört.

Korrigiere:
- Exakte Source-Key-Länge und exakte aktive Source-Generation validieren.
- Exakte Observation-ID und Source-Generation müssen gemeinsam matchen.
- Stale oder fremde Events dürfen keine Observation verändern.
- Observation-ID muss innerhalb der Source-Generation eindeutig und nicht wiederverwendbar sein; doppelte identische Add/Update-Ereignisse müssen gemäß Spec idempotent sein oder als Konflikt klassifiziert werden.
- Observation-Withdrawal entfernt ausschließlich die Evidence dieser Source. Sie darf weder PhysicalDevice-Tombstone noch andere Source-Evidence entfernen.

Tests: gleiche Observation-ID aus zwei Sources; gleiche Source mit alter Generation; alte Generation nach Re-Register; Duplicate Add/Update; Remove nach bereits erfolgtem Remove; Remove mit abweichendem Key/Length.

## D. ObserveDevice speichert keine korrekte Evidence-Provenienz und liefert falsche Referenzsemantik

Aktuell:
- Bei bereits gefundenem Device wird sofort SUCCESS + roher `XaveriusPhysicalDevice *` zurückgegeben; die Observation wird dabei überhaupt nicht gespeichert.
- Bei neuem/mehrdeutigem Device wird `obs->device = NULL` gespeichert, ohne spätere Auflösung/Verknüpfung.
- Der API-Vertrag liefert keinen Observation-Handle/keine Observation-ID zurück; stattdessen muss der Caller eine ID vorgeben.
- Das Ergebnis AMBIGUOUS wird nicht sichtbar vom PENDING-Fall unterschieden.
- `registry` wird ignoriert.

Richte die Semantik an Spec §§2.1–2.2 aus:
- Jede akzeptierte Observation wird mit Source-Key, exakter Source-Generation, Observation-ID/Event-Sequence, vollständiger Identitätsevidence und Provenienz gespeichert, auch wenn sie auf ein bereits kanonisches Device matcht.
- Ein eindeutiger Match verknüpft die Observation mit dem kanonischen Device; keine zweite Device-Instanz.
- Ambiguous/insufficient evidence bleibt unresolved und gibt den normativen klassifizierten Status zurück.
- Rückgabe muss eine server-owned, generation-checked Referenz/Token-Semantik verwenden. Kein roher Device-Pointer darf als langlebige oder aufrufübergreifende Referenz ausgegeben werden. Falls ein synchroner interner Pointer als rein temporärer Return notwendig ist, muss sein Call-Scope ausdrücklich begrenzt sein und darf nicht in der API als retained handle erscheinen.
- `registry` muss validiert und tatsächlich als Registry-Instanz verwendet werden; falls XAverius v0.1 nur einen Singleton zulässt, muss die API den exakten Singleton-Vertrag durchsetzen statt das Argument zu ignorieren.
- Outputs bei jedem Eintritt auf sichere Fehlerwerte setzen, soweit Output-Pointer gültig sind.

Tests: matched Device speichert Observation; mehrere Sources korrelieren auf einen Device-Datensatz; unresolved → späterer eindeutiger Match; Ambiguous bleibt unveröffentlicht; keine Pointer-Retention; falsche Registry; Outputs bei jedem Fehler null/0.

## E. ConfirmDeviceRemoved ignoriert sämtliche Removal Evidence

Der aktuelle Code:
- prüft nur `device != NULL` und `device_generation`;
- ignoriert `removal_evidence` und `removal_evidence_size`;
- validiert weder Source-Key noch Source-Generation noch Observation-ID/Event-Sequence;
- prüft keine Authority der registrierten Source;
- setzt lediglich `status = UNAVAILABLE` (Kommentar nennt dies selbst Placeholder);
- verhindert keine doppelte/konkurrierende Removal-Transition.

Das ist kein implementiertes Removal-Verfahren und darf nicht SUCCESS zurückgeben.

Implementiere den normativen Vertrag aus Spec §2.2:
- Vollständige Größen-/Versions-/Feldvalidierung der `XaveriusDeviceRemovalEvidence` vor jedem Cast/Dereferenzieren.
- Source-Key muss auf eine aktuell registrierte Source auflösen; Generation muss exakt sein.
- Observation-ID und `source_event_sequence` müssen exakt die gespeicherte Observation referenzieren.
- `physical_device_generation` muss die aktuelle Device-Inkarnation identifizieren.
- Authority ausschließlich aus registrierten Source-Capabilities und Server-Trust-Policy ableiten; niemals aus einem Caller-Authority-Flag.
- Nur ausreichende, nicht widersprüchliche authoritative Evidence darf terminal REMOVED committen.
- Fehlende, stale, untrusted, widersprüchliche oder unzureichende Evidence lässt den Status unverändert und liefert klassifizierten Nicht-Erfolg.
- REMOVED ist terminal und wird genau einmal serialisiert; es darf weder als UNAVAILABLE-Placeholder noch als frei caller-setzbarer Status implementiert werden.
- Keine Freigabe des Device-Tombstones/XID.

Tests: NULL/short/oversized/wrong-version Evidence; unknown/untrusted source; wrong/future/stale source generation; unknown observation; falsche event sequence; falsche Device generation; non-authoritative remove; conflicting remaining source evidence; duplicate valid removal; valid authoritative removal; no status mutation on every rejected case.

## F. Keine erkennbare Serialisierung / Race-Schutz

Spec §6 verlangt, dass alle Registry-Mutationen im Server-Main/Event-Thread oder einer gleichwertigen server-owned Serialization Domain stattfinden. Worker dürfen nur immutable observations enqueuen.

Die neuen Funktionen mutieren globale Listen und Status direkt. Implementiere die vorgeschriebene Admission-/Serialization-Grenze:
- Entweder beweisbar Main-Thread-only mit klaren Assertions/Contract an jeder mutierenden API, oder enqueue immutable Events in die bestehende server-owned Domain.
- Keine neue unabhängige Registry-Lock-Architektur, falls Xserver-Serialization bereits der normative Mechanismus ist.
- Source unregister/shutdown muss laufende und queued Events drainen, bevor Source-/Observation-Speicher freigegeben wird.
- Device-/Source-/Observation-Generationen bei Ausführung des Events erneut prüfen, nicht nur bei Enqueue.

Tests: Observe vs RemoveObservation; RemoveObservation vs Source retirement; valid remove vs stale remove; zwei konkurrierende Removal-Events; queued old-generation event nach Re-register; Shutdown während queued Events. Kein UAF, kein Double Remove, keine falsche Tombstone-Transition.

## G. Initialisierung, Lifetime und Zähler

Prüfe `XaveriusRegistryInit/Fini` für alle neuen Source-/Observation-Listen und Zähler:
- Jede Liste genau einmal initialisieren, niemals auf uninitialisierte `xorg_list` zugreifen.
- Counts dürfen nicht unterlaufen; bevorzugt keine redundant gepflegten Counts ohne Bedarf.
- Alle provisionalen Allokationen failure-atomic zurückrollen.
- Fini darf nicht einfach Counts auf 0 setzen und Speicher/Listen ignorieren.
- Shutdown folgt dem Spec-Lifecycle; keine Observation darf einen Device-Pointer nach dessen Retirement halten.
- Source-/Observation-/Device-Generationen sind getrennte nicht-reusable Identitäten.

Tests: init/fini mit 0, 1 und vielen Sources/Observations; Fehler an jeder Allokation; doppelte Init/Fini; Retire mit outstanding observation; keine Leaks/UAF.

## H. Kanonische Ordering- und Binding-Preference-Aufgabe bleibt separat offen

W0-016 §6 und die aktive Spec verlangen:
- einen normalisierten typed tuple ordering key nach Architektur §2.3;
- einen bytewise lexicographic comparator;
- keine Pointer/XID/DRM-minor/Screen-/Registrierungsreihenfolge;
- stabile, nichtleere, case-sensitive `owner_key`-Werte;
- serverkonfigurierte Preference-Liste, Duplikate als Konfigurationsfehler;
- Auswahl genau einmal vor Publication, keine spätere Rebinding-/Failover-Logik.

Der Workspace-Task meldet dies als verbleibende Arbeit. Behandle es nicht als erledigt, nur weil ein Commit einen Comparator enthält. Implementiere mit Tests und dokumentiere konkrete Konfigurationsquelle/Initialisierung. Keine Provider-API und keine zweite Dispatch-Schicht.

## Ausschlüsse

- Keine udev-/Platform-Adapter, kein DDX-wide Enumeration, keine ScreenAttachment-Implementierung oder ScreenInit-Integration.
- Keine Protocol-Handler, kein Stage 5+, kein Vulkan-Backend.
- Keine neuen Header; die drei vereinbarten Header bleiben exklusiv.
- Keine Provider-/Candidate-API.
- Keine kosmetische Umformatierung, keine unrelated rename/refactor.
- Kein Merge.

## Abschlussnachweis

Für A–H jeweils:
1. Befund und betroffene Datei/Funktion.
2. Fix und Normstelle.
3. Regressionstestname und Testbefehl.
4. Commit-SHA und Parent.
5. Build-/Testresultat.

Enterprise führt danach einen unabhängigen read-only Review durch. Kein Stage-A-Fortschritt und keine Freigabe von Stage 1–4, bevor W0-027 und W0-029 vollständig geschlossen und reviewed sind.
