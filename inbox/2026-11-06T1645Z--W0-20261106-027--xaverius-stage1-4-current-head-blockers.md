---
id: W0-20261106-027
kind: directive
from: chatgpt
to: defiant
created: 2026-11-06T1645:00Z
related:
  - W0-20261106-016
  - W0-20261106-022
  - active-physical-device-registry-task-2026-11-05
workspace: X11Libre/xserver
target: X11Libre/xserver@wip/xaverius
reviewer: enterprise
priority: urgent
---

# Stage 1–4: konkrete Restfehler im aktuellen Registry-HEAD beheben

## Verifizierter Stand

Remote-Branch `X11Libre/xserver:wip/xaverius` wurde gelesen; aktueller HEAD bei Auftragserstellung:
`24858501c5` (vollständige SHA bitte vor Änderung erneut auflösen).
Neueste Commits gegenüber dem vorigen Audit:
- `2e69ba726a`: Caller-Release soll nur Caller-Referenz freigeben
- `5716ce694f`: Publish-Funktionsname auf Xaverius-Namespace
- `deee1921ea`: ModuleRole-Lifetime/Binding-Retention
- `f640840daa`: failure-atomic Registration/Snapshot
- `24858501c5`: Retain nach Identity-Check verschoben

W0-022 und W0-016 bleiben vollständig verbindlich. Dieser Auftrag ist eine konkrete Ergänzung aufgrund des tatsächlich sichtbaren aktuellen Quellstands, keine Freigabe für Stage A/DDX-Discovery oder spätere Phasen.

## Vor Änderungen

1. HEAD, Parent und Worktree erneut prüfen. Bei neuem HEAD auf diesem Stand weiterarbeiten, niemals resetten/rebasen/force-pushen.
2. Bestehenden alleinigen Writer-/Worktree-Schutz beibehalten.
3. Zuerst die folgende Befund→Fix→Test-Matrix bestätigen. Danach direkt korrigieren; nicht auf Enterprise warten.
4. Kleine, getrennte Commits. Kein Formatierungsumbau außerhalb geänderter Stellen.

## Konkrete, im aktuellen Quelltext sichtbare Blocker

### A. ModuleRole-Retain wird bei Fehlern nicht zurückgegeben

In `Xext/xaverius/registry.c`, `XaveriusRegistryRegisterDdxDevice()`:
- `XaveriusModuleRoleRetain(role)` erfolgt vor `xaverius_binding_alloc()`.
- Bei fehlgeschlagener Binding-Allokation wird direkt zurückgegeben, ohne `XaveriusModuleRoleRelease(role)`.
- Auch die späteren Fehlerpfade (PhysicalDevice-Allokation, Identity-Evidence-Allokation, Snapshot-Query/Validierung) geben den erworbenen Role-Ref nicht zurück.
- `free(binding)` umgeht den kanonischen Binding-/Role-Retirement-Pfad.

Korrigiere das durch einen klaren provisionalen Ownership-Zustand und einen gemeinsamen Rollback-/Retirement-Pfad. Jede erfolgreiche Retain-Operation muss bei Fehlschlag exakt einmal ausgeglichen werden; bei Erfolg geht genau diese Referenz in das installierte Binding über. Keine direkte Binding-Freigabe, die die Role-Referenz umgeht.

Tests: Fehlerinduzierte Allokationsfehler an jeder Stufe, Snapshot-Callback-Fehler, ungültige Identität; Retain/Release-Zähler müssen exakt ausgeglichen sein. Erfolgsfall: Binding hält genau eine Role-Referenz bis zum Retirement.

### B. Descriptor-Validierung ist doppelt und liefert falsche Fehlerklasse

Der aktuelle Code prüft den Ops-Header zweimal identisch. Zudem werden Größenfehler und unbekannte ABI-Versionen gemeinsam als `INVALID_ARGUMENT` zurückgegeben. Das widerspricht dem Lifetime-Addendum:
- Size mismatch → `INVALID_ARGUMENT`
- unbekannte ABI-Version → `UNSUPPORTED`
- keine Hook-/Lease-Nebenwirkung vor vollständiger Validierung.

Prüfe die drei Descriptoren vollständig und jeweils mit dem zugehörigen ABI-Vertrag: `XaveriusModuleRoleDescriptor`, `XaveriusDeviceBackendOps`, `XaveriusDeviceBackendRegistration`. Verifiziere insbesondere, dass die Registration nicht versehentlich mit dem Ops-ABI-Namen validiert wird. Entferne die doppelte Prüfung. Setze alle Output-Parameter unmittelbar nach Eintritt auf sichere Fehlerwerte (insbesondere `*out_device = NULL`, soweit der Pointer gültig ist), bevor ein Fehler zurückgegeben werden kann.

Tests je Descriptor: NULL, undersized, oversized gemäß exaktem Spec-Vertrag, unbekannte Version; exakte Result-Klasse; kein Retain/Callback bei ungültigem Descriptor.

### C. Binding speichert weiterhin fremde Ops-Pointer

`xaverius_binding_alloc()` weist `binding->ops = registration->ops` zu. Das ist kein server-owned immutable copy und kann auf temporäre DDX-Descriptoren zeigen. Das Lifetime-Addendum verlangt vollständige by-value-Kopien des Top-Level-Descriptors und der vier typisierten Operationstabellen.

Implementiere die kleinste korrekte immutable server-owned Kopie gemäß v0.1-ABI. Alle verschachtelten Tabellen müssen ebenfalls kopiert werden; Pointer im Binding müssen auf die eigenen Kopien zeigen. Keine Änderung der Callback-Signaturen und keine neuen Context-Retain-/Release-Hooks.

Tests: DDX-Quellstrukturen nach erfolgreicher Registrierung überschreiben/freigeben; Binding-Aufrufe müssen weiterhin die unveränderten kopierten Funktionszeiger verwenden. Prüfe alle vier Tabellen.

### D. Snapshot wird noch nicht vollständig validiert und übernommen

Der aktuelle Code ruft `query_physical_snapshot` auf, setzt danach aber `snapshot_size = 0` und installiert das Ergebnis ohne sichtbare vollständige Validierung/deep copy. Der Callback darf keine beliebige unvalidierte Struktur als Snapshot in den Registry-Datensatz einschleusen.

Implementiere:
- failure sentinels vor dem Callback;
- Pflicht-Callback-/Capability-zu-Operation-Validierung;
- Prüfung aller Pflichtfelder, Pointer/Count-Paare, Overflow-/Maximalgrenzen, Enums/Ranges, UUID, Queue-/Memory-Konsistenz und beworbenen Features;
- `timelineSemaphore = 1` als Pflichtbedingung;
- Unterscheidung optional fehlender Properties vs. Query-Fehler vs. fehlende Pflichtdaten;
- vollständige Deep-Copy variabler Daten in server-owned Snapshot-Speicher;
- keine Registry-Installation/Publication bei irgendeinem Fehler.

Tests: je Fehlerklasse mindestens ein negativer Fall, Count/Pointer-Mismatch, Overflow, zu große Arrays, ungültige UUID/Enums, Capability ohne passende Operation, Timeline-Semaphore fehlt, optionale Property fehlt.

### E. Identity-Evidence wird vor Längenvalidierung interpretiert

Aktuell wird `identity_evidence` als `XaveriusDeviceIdentity *` dereferenziert, obwohl die Länge nicht vorher exakt geprüft wird. Zusätzlich wird bei fehlender Evidenz eine Null-Identity als Ersatz verwendet.

Validiere zuerst Pflicht-Pointer, exakte/minimale Strukturgröße und Version/Format der Evidenz gemäß Spec. Erst danach darfst du casten/kopieren. Fehlende oder unvollständige Pflicht-Evidenz darf keine Null-Identity erzeugen und keine Registrierung ermöglichen.

Tests: NULL, 0/zu kleine Länge, übergroße/inkompatible Evidenz, ungültige Felder, gültige Evidenz; keine Out-of-Bounds-Lesezugriffe.

### F. Identity-Treffer ist noch kein exaktes Replay

Der aktuelle `existing != NULL`-Zweig gibt bei identischer Hardware-Identity unconditional SUCCESS zurück und liefert das vorhandene Binding zurück. Der Code prüft dort nicht dieselbe live Registration-/Binding-Ownership und exakte Binding-Generation. Das widerspricht dem Lifetime-Addendum und kann einen zweiten DDX-Aufrufer fälschlich als Replay akzeptieren.

Implementiere die normative Unterscheidung:
- exakt dieselbe noch lebende Registration-/Binding-Ownership und Generation → idempotentes Replay, zusätzliche Caller-Referenz, gleiche Generation;
- gleiche Hardwareidentität mit anderer Binding-Ownership/Generation → klassifizierter Konflikt/Mehrdeutigkeit, kein Caller-Ref;
- stale/retired Generation → stale result, kein Caller-Ref.
Keine Pointer-Gleichheit oder gleiche UUID/owner_key allein als Replay-Kriterium.

Tests: echtes Replay, anderer Role-Owner, anderer Context/Native-Handle, neue Binding-Generation, retired Role, gleicher Identitätsschlüssel mit widersprüchlichen Evidenzen.

### G. Failure-atomicity muss auch DIX und Registry abdecken

Die Kommentare behaupten gemeinsame Rollback-Logik; der Code muss sie tatsächlich umsetzen. Verifiziere insbesondere, dass vor erfolgreicher vollständiger Snapshot-Validierung kein Registry-Listeneintrag entsteht und dass DIX-Wrapper/XID/PUBLISHED-Status erst nach erfolgreichem einmaligem `AddResource()` gemeinsam sichtbar werden. Registry-Membership, Caller-Referenz und DIX-Resource bleiben getrennte Ownership-Klassen.

Tests: Fehler an jeder Transaktionsgrenze; keine teilweise sichtbare Registry-/DIX-Änderung, keine wiederverwendete Generation, keine Leaks/Doubles.

## Ausschlüsse

- Keine DDX-wide Discovery, kein modesetting, keine udev-/Platform-Adapter, keine Screen-Integration.
- Keine Protocol-Handler, kein Stage 5+, keine Phase B+.
- Keine neuen Header; die drei vereinbarten Header bleiben exklusiv.
- Keine Provider-/Candidate-/per-Screen-Inventar-Architektur.
- Keine API-/Typ-Renames außerhalb zwingend notwendiger Korrektur zur Spec.
- Keine pauschale Build-/Formatierungsreform.

## Abschlussnachweis

Für jeden Teilcommit: SHA, Parent, geänderte Dateien, konkreter Fix und Tests. Danach vollständiger Testlauf und eine Befund→Fix→Test-Abdeckung gegen W0-016/W0-022/W0-027. Enterprise prüft read-only. Kein Merge und keine Stage-A-Arbeit vor ausdrücklicher Review-Freigabe.
