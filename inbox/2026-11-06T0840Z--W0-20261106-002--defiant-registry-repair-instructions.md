# W0-20261106-002 — Defiant: konkrete Reparaturanweisung für den PhysicalDevice-Registry-Kern

Date: 2026-11-06
To: Defiant (einziger Writer auf `X11Libre/xserver:wip/xaverius`)
CC: Enterprise (Reviewer/Koordination, read-only gegenüber dem Xserver-Worktree)
Priority: HIGH — die Registry-Reparatur ist die Hauptaufgabe

Dieses Schreiben konkretisiert den Audit W0-20261106-001. Es ersetzt nicht die normative Spec. Maßgeblich bleiben ausschließlich `metux/xav-design:docs/restructure-2026-10-21` und dort insbesondere:

- `implementation-notes/active-physical-device-registry-task-2026-11-05.md`
- `spec/device-registry-architecture.md`
- `spec/registry-api-lifetime-addendum.md`
- `spec/provider-operation-matrix.md`
- `implementation-notes/xserver-registry-rebuild-plan-2026-11-05.md`

Der Audit hat den auf GitHub sichtbaren Stand als nicht konform mit den Stages 1–4 bewertet. Das ist kein Auftrag, schnell weitere Features auf die alte Provider-Implementierung zu setzen. Ziel ist ein kleiner, reviewbarer, tatsächlich funktionsfähiger Registry-Kern gemäß aktueller Architektur.

## 0. Zuerst Ist-Zustand sichern — noch keine Änderungen

1. Ermittle den aktuellen Remote-HEAD von `wip/xaverius` und den lokalen HEAD samt Parent-SHA.
2. Melde:
   - vollständige HEAD- und Parent-SHA;
   - `git status --short`;
   - alle geänderten/untracked Dateien;
   - lokale Commits, die noch nicht auf dem Remote liegen;
   - Remote-Commits, die lokal fehlen;
   - welche Dateien seit dem Audit W0-20261106-001 verändert wurden.
3. Vergleiche die aktuellen Inhalte von `xaverius_registry.c`, `xaverius_priv.h`, `xaverius_registry.h`, `xaverius_intern.h`, `init.c` und `meson.build` mit den dort dokumentierten Blob-SHAs.
4. Enterprise prüft diesen Statusbericht unabhängig und bestätigt, dass Defiant tatsächlich auf dem aktuellen Remote-Stand arbeitet. Enterprise darf nicht parallel in den Xserver-Worktree schreiben.
5. Bei Divergenz, unbekannten Commits oder unerwarteten lokalen Änderungen: anhalten und melden. Kein Reset, kein Clean, kein Force-Push, kein blindes Pull/Rebase und kein Cherry-pick aus alten Branches. Uncommitted Work sichern, nicht verwerfen.

## 1. Reparaturstrategie: vertikale, testbare Stages

Nicht versuchen, alle acht Audit-Befunde in einem großen Rewrite-Commit zu beheben. Zuerst einen kurzen Architektur-/Dateiplan an Enterprise melden. Danach in dieser Reihenfolge implementieren. Jeder Schritt muss bauen oder als eng begründete, unmittelbar folgende Commit-Serie nachvollziehbar sein.

### Commit-Gruppe A — kanonische Typen und API (Stage 1)

Ersetze die alte Provider-/Capabilities-Abstraktion durch genau das aktuelle Modell:

- `GpuPhysicalDevice`: eine kanonische serverweite physische Geräteinstanz; der immutable Snapshot ist Bestandteil dieses Objekts.
- `GpuDeviceBackendBinding`: direkte Bindung genau eines DDX-registrierten Geräts an dessen native Handle-/Context-Werte und immutable typed ops.
- `GpuDeviceBackendOps`: versionierter Top-Level-Descriptor mit den vier normativen typed operation tables.
- `GpuDeviceBackendRegistration`: vollständiger Registrierungsdeskriptor mit strukturierter Identität, exaktem nativen Handle, per-device Context, Ops-Descriptor und exakter ModuleRole-Generation.
- `GpuModuleRole`: gemeinsamer Code-/Modul-Lifetime- und Quiescence-Datensatz; ausdrücklich kein per-device Dispatch-Proxy.

Normative API-Namen verwenden: `GpuRegistryRegisterDdxDevice`, `GpuPhysicalDeviceRelease`, `GpuRegistryRemoveDeviceBackendBinding` sowie die getrennte interne Publikationsoperation `XaPhysicalDevicePublish`.

Konkret:
1. Führe die Typen in den vorhandenen passenden privaten Headern ein. Keine neuen Convenience-Header und keine doppelten parallelen Typmodelle ohne zwingenden Grund.
2. Entferne bzw. entkopple die alten produktiven APIs/Typen `XaveriusProviderOps`, `XaveriusProviderCapabilities`, `XaveriusRegisterDeviceDirect` und die nicht implementierte `XaveriusRegisterProvider`. Keine Kompatibilitätsfassade, die das alte Modell weiter als Registry-Kern verwendet.
3. Halte die vorhandenen Header-Scope-Grenzen ein. PhysicalDevice-XID und DIX-Wrapper bleiben serverintern und dürfen nicht in DDX-sichtbare Header gelangen.
4. Setze `struct_size` und `abi_version` an den vorgeschriebenen Descriptoren um. Für v0.1 gilt exakt die in der Spec festgelegte Größe und Version. Prüfe den Header, bevor du nachfolgende Felder oder Callback-Pointer liest.
5. Size mismatch => `INVALID_ARGUMENT`; unbekannte Version => `UNSUPPORTED`. Bei diesen Fehlern darf kein Callback und kein Retain/Release-Hook aufgerufen werden.
6. Dokumentiere Ownership an den tatsächlichen Feldern/Übergängen. Native Handle und per-device Context sind borrowed DDX-owned values. Registry darf sie niemals freigeben und darf keine erfundenen Context-Retain/Release-Callbacks ergänzen.

Stage-1-Tests:
- NULL-Descriptor;
- zu kleiner und zu großer v1-Descriptor;
- unbekannte ABI-Version;
- sicherstellen, dass bei ungültigem Header kein Callback/Hook aufgerufen wird;
- statische Typ-/Layoutprüfungen, soweit sinnvoll.

### Commit-Gruppe B — ModuleRole und Lifetime (Stage 2)

Implementiere ModuleRole als eigenständigen Lifetime-Datensatz, nicht als Backend-Dispatcher.

Muss-Verhalten:
1. Nicht wiederverwendbare Role-Generation und exakte Generation-Prüfung.
2. Admission schließen, sodass nach Beginn des Unload/Shutdown keine neuen Callbacks angenommen werden.
3. Jeder tatsächlich laufende Callback hält eine Execution-Reference; beim Verlassen wird sie zuverlässig freigegeben.
4. Quiesce-Ergebnis unterscheidet mindestens PENDING, SUCCESS und FAILURE. Bei FAILURE oder unklarem Zustand wird die Role/Binding quarantänisiert; kein optimistisches Freigeben von Code, Handle oder Context.
5. Permanent Roles haben ebenfalls nicht-NULL retain/release/quiesce Hooks. Sie haben keine Loader-Lease; Shutdown darf erst im Registry-Shutdown-Pfad beginnen.
6. Lease-Paarung exakt einhalten: Modul-Owner behält seine ursprüngliche Lease-Referenz; jede unloadbare Role hält genau eine zusätzliche Referenz; Role-Finalisierung gibt nur ihre eigene frei; der Owner gibt seine Referenz erst frei, wenn alle Roles dieser Modulinstanz finalisiert sind.
7. Hooks sind infallible, non-blocking Lifetime-Primitives. Keine Screen-Lifetime, kein `DriverPtr.refCount`, kein roher `ModuleDesc *` und keine per-Screen duplizierte ModuleDesc als Ersatz verwenden.

Stage-2-Tests:
- Registrierung schlägt vor Retain fehl => kein Hook;
- erfolgreicher Role-Aufbau => Retain genau einmal;
- mehrere Roles teilen eine Module-Lease korrekt;
- Callback gegen gleichzeitig beginnendes Admission-Close;
- PENDING -> später SUCCESS;
- FAILURE -> Quarantäne, kein vorzeitiges Release;
- Finalisierung -> Release genau einmal;
- Permanent-Role-Shutdown.

### Commit-Gruppe C — Registry, Identität und Generationen (Stage 3)

Implementiere einen serverweiten Registry-Dienst mit serialisierten Mutationen im Server-Main-/Event-Thread. Keine Registry pro Screen.

Identitätsregeln:
1. PCI: vollständige Domain:Bus:Device.Function plus kanonischer aufgelöster sysfs-/Kernel-Device-Pfad und Incarnation.
2. Primary-/Render-DRM-Nodes sind über die zugrunde liegende Kernel-DRM-Identität zu korrelieren. Major/minor, DRM-Minor, Node-Name oder Pfad allein sind keine kanonischen Identitäten.
3. Vulkan deviceUUID ist starke Korrelations-/Bestätigungsevidenz, aber kein Ersatz für widerspruchsfreie Plattformidentität. UUID-Konflikt bei gleicher autoritativer Plattformidentität ist ein Konflikt/Quarantänefall, nicht einfach „zwei Geräte“.
4. Nicht-PCI-Geräte benötigen passende Bus-/Subsystem-/Parent-Evidenz oder eine ausdrücklich DDX-scoped stabile, instanz-eindeutige Fallback-ID. Zwei Geräte ohne solche Evidenz dürfen niemals allein deshalb gleich sein, weil beide keine PCI-Koordinaten und keine UUID haben.
5. Abgeleitete Geräte (VF, Partition, MIG etc.) benötigen eine eigene Instanzidentität; Parent-ID allein darf Geschwister nicht zusammenführen.
6. Vendor-/Device-ID, Name, Modell, Treiberversion oder API-Version dürfen Identität nicht begründen.
7. Evidenz muss Provenienz und Generation behalten. Verspätete Events einer alten Generation dürfen aktuelle Zustände nicht überschreiben.
8. Identitätsambiguität bleibt unveröffentlicht. Nicht raten, nicht zusammenführen, nicht zwei unaufgelöste Kandidaten als gesicherte Geräte publizieren.

Registrierungssemantik:
- Nur exaktes Replay derselben noch lebenden Registrierung/Binding-Ownership ist idempotent.
- Exaktes Replay liefert dieselbe Binding-Generation und eine zusätzliche caller-owned PhysicalDevice-Referenz.
- Gleiche Hardwareidentität mit konkurrierender/abweichender Binding-Ownership ist ein klassifizierter Konflikt, kein pauschales `true`.
- Jede Device-, Binding- und Role-Generation ist nicht wiederverwendbar. Generationen, die in einem fehlgeschlagenen Versuch bereits reserviert wurden, bleiben verbraucht.
- Kein Mischen von Snapshots, Ops, Handles oder Contexts verschiedener Registrierungen.
- Deterministische Binding-Auswahl gemäß konfigurierter DDX-Role-Priorität, niemals nach Registrierungsreihenfolge, Screen-Reihenfolge, Pointerwert, DRM-Minor oder Modul-Ladereihenfolge. Kein transparentes Failover in v0.1.

Stage-3-Tests:
- PCI-Gerät und mehrere unterstützte Non-PCI-Fälle;
- Primary-/Render-Alias;
- fehlende und widersprüchliche Evidenz;
- gleiche UUID mit widersprüchlicher Plattformidentität;
- zwei DDX-Registrierungen für dasselbe Gerät;
- deterministische Role-Priorität unabhängig von Registrierungsreihenfolge;
- exaktes Replay vs. Konflikt;
- Remove/Re-add mit neuer Incarnation;
- verspätete/stale Generation wird abgewiesen.

### Commit-Gruppe D — failure-atomare Registrierung und Snapshot (Stage 4, Teil 1)

Die Registrierung muss über die exakte provisorische Binding laufen:

1. Vollständigen Registration-Descriptor-Header prüfen.
2. Identität, erforderliche non-NULL Handle-/Context-Werte, exakte lebende ModuleRole-Generation und Ops-ABI prüfen.
3. Role-Admission/Reference sicher erwerben. Bei quiescing/retired Role abbrechen.
4. Alle erforderlichen Top-Level-Callbacks und alle vier typed operation tables für das aktivierte v0.1-Profil validieren.
5. Descriptor und Operationstabellen vollständig by-value in immutable server-owned Binding-Storage kopieren. Kein teilweise sichtbares Binding.
6. Identität kanonisieren und Generationen reservieren.
7. Snapshot-Ausgabe vor dem Callback mit Failure-Sentinels initialisieren.
8. `query_physical_snapshot` ausschließlich über die exakte provisorische Binding aufrufen; Callback-Execution-Reference halten.
9. Alle Snapshot-Felder validieren: Pointer/Count-Paare, Obergrenzen, Wertebereiche, UUID, Queue-/Memory-Konsistenz, Feature-Konsistenz. Keine ungültigen oder nur teilweise gefüllten Daten übernehmen.
10. Alle dynamischen Arrays vollständig tief kopieren. Snapshot ist danach vollständig server-owned und unveränderlich.
11. Erst wenn alle Schritte erfolgreich sind, Binding + Snapshot atomar als interne, noch nicht notwendigerweise veröffentlichte PhysicalDevice-Instanz installieren.
12. Jeder Fehler läuft über einen gemeinsamen Provisional-Object-Cleanup-Pfad. Keine halben Listen-Einträge, Bindings, Caller-Refs oder freistehenden Snapshot-Pointer. Borrowed Handle/Context werden auch beim Fehler nicht freigegeben.

Wichtig: Ein erfolgreicher Register-Aufruf bedeutet nur „Binding und Snapshot installiert“. Er bedeutet weder XID-Zuteilung noch DIX-Publikation.

Tests:
- Fehler-Injektion an jeder Allokation und jedem Callback;
- Callback-Fehler und ungültige Snapshot-Felder;
- NULL+positive Count und non-NULL+Count 0;
- übergroße Counts und inkonsistente Queue-/Memory-Angaben;
- Snapshot vollständig kopiert, auch wenn DDX die Quelldaten direkt nach Rückkehr ändert;
- kein Leak und kein sichtbarer Registry-Eintrag nach jedem Fehler;
- Handle/Context werden niemals von Registry/cleanup freigegeben.

### Commit-Gruppe E — getrennte DIX-Publikation (Stage 4, Teil 2)

Publikation ist ein eigener serverinterner Vorgang, nicht Teil von `GpuRegistryRegisterDdxDevice`.

1. Vor Publikation müssen Identität aufgelöst, eligible binding deterministisch gewählt und Startup-Publikations-Gate geöffnet sein.
2. `XaPhysicalDevicePublish` bereitet serverprivaten XID und DIX-Wrapper vor, während das Objekt noch provisional/unpublished ist.
3. `AddResource()` wird exakt einmal aufgerufen.
4. Erst wenn `AddResource()` erfolgreich war, werden XID, DIX-Resource-Ownership und Published-State im serialisierten Registry-Kontext gemeinsam sichtbar.
5. Bei `AddResource()`-Fehler: kein sichtbarer XID, kein Published-State, kein stale Listeneintrag; provisional Ressourcen vollständig aufräumen.
6. DIX-Resource-Destructor ist der kanonische Pfad zum Entfernen der veröffentlichten Resource-Membership. Keine parallele direkte Freigabe derselben Resource durch RegistryFini/DevicesFini.
7. `RegistryFini` darf nicht über bereits freigegebene oder stale Einträge laufen und darf nicht ein zweites Mal destruktieren.

Tests:
- AddResource-Aufrufszähler == 1 bei erfolgreicher Publikation;
- AddResource-Fehler hinterlässt keinerlei Publikation;
- Registrierung vor Öffnung des Gates liefert noch keine XID;
- Gate-Öffnung publiziert nur vollständig vorbereitete, eindeutig aufgelöste Geräte;
- Destructor genau einmal;
- RegistryFini nach DIX-Destruction und umgekehrt ohne Double-Free/UAF;
- Referenzzählung für Caller, Registry, DIX und Binding korrekt.

## 2. Verbindlicher Ownership-Bericht

Mit Stage D/E einen konkreten Tabellenbericht abliefern. Spalten: Referenzklasse, Owner, Erwerb, Freigabe, Schutzwirkung, Verhalten bei Fehler/Quarantäne.

Mindestens eigene Zeilen für:
- unmittelbare Caller-Reference aus erfolgreicher Registrierung;
- Registry-Membership-Reference;
- DIX-Resource-Ownership/Reference;
- PhysicalDevice -> ausgewählte Binding-Reference;
- Binding -> exakte ModuleRole-Generation;
- ModuleRole -> Module-Lease und Modul-Owner -> ursprüngliche Lease;
- synchrone Callback-Execution-Reference;
- spätere LogicalDevice-/Backend-/Operation-Token-Referenzen (als ausdrücklich deferred markieren, nicht vorzeitig implementieren).

Context und native Handle bleiben DDX-owned borrowed values. Ihre Lebensdauer wird durch den exakten Binding-/Role-Lifecycle-Vertrag geschützt; keine Registry-eigene Context-Refcount-API erfinden.

## 3. Init/Fini und Integrationsgrenzen

- Vorhandenen Extension-Init/Fini-Pfad und die bestehende Init-Registrierung untersuchen; genau einen kanonischen Pfad verwenden.
- PhysicalDevice-DIX-Resource-Type genau einmal initialisieren, bevor er verwendet wird.
- Implementiere jetzt nur Init/Fini-Anteile, die Stages 1–4 benötigen.
- Discovery-Quellen stoppen, ScreenAttachment, spätere LogicalDevices, Backend-Objekte und Protokoll-Handler bleiben deferred und dürfen nicht mit Erfolg-Stubs vorgetäuscht werden.
- Keine zweite Registry- oder Plattform-Inventar-Implementierung und kein globaler `/dev/dri`-Scan.
- Keine Änderungen an modesetting/ScreenInit zur Erzeugung einer scheinbaren Demo. DDX-weite Enumeration/Registrierung ist ein separates Folgepaket nach Review des Registry-Kerns.

## 4. Explizit nicht tun

- Keine Protocol-Handler, Vulkan-Backend-Operationen, Compute-Memory-Handler oder Opcode-Arbeit.
- Keine Screen-gebundene PhysicalDevice-Registry.
- Keine Wiederbelebung des Provider-Candidate-/Provider-Dispatch-Modells.
- Keine Übernahme aus `wip/xaverius_old_1` oder `rfc/xaverius*`; diese Branches sind ausschließlich forensisch.
- Keine unnötigen Umbenennungen, Formatierungswellen, Include-Sortierung, Lizenzheader-Änderungen, breiten Refactors oder neuen Dateien ohne konkreten Interface-Grund.
- Keine CI-Reparatur als Vorwand, die Registry-Arbeit zu pausieren; OpenBSD/Cygwin bleiben getrennte LOW-Priority-Aufgaben.
- Kein Commit, der ungetesteten Code als „Stage complete“ ausweist.

## 5. Review-Gates und Berichtspflicht

Nach jeder Commit-Gruppe:
1. Defiant meldet Commit-SHA + Parent-SHA, genaue Dateien, Symbole/API, Build- und Testbefehle mit Exitcodes, offene Punkte.
2. Enterprise reviewt read-only gegen die oben genannten Spec-Dateien und prüft insbesondere, dass kein alter Provider-Begriff/API als aktive Architektur übrig blieb.
3. Stage erst nach dokumentierter Review-Freigabe als abgeschlossen markieren.
4. Defiant bleibt alleiniger Writer. Enterprise startet keinen parallelen Patch und verändert nicht denselben Branch.
5. Erst nach akzeptierten Stages 1–4 darf ein separater Auftrag DDX-weite Discovery/ScreenAttachment-Integration freigeben.

Für die Gesamtfreigabe sind mindestens erforderlich:
- Descriptor-/ABI-Validierungstests;
- ModuleRole-/Lease-/Quiescence-Tests;
- Identity-/Incarnation-Vektoren;
- exaktes Replay und Konfliktfälle;
- vollständige Snapshot-Validierung und Fehler-Injektion;
- AddResource-exactly-once und DIX-Destructor-/Refcount-Tests;
- Ownership-Tabelle;
- erfolgreicher Build und dokumentierte Testresultate auf dem exakten Commit.

## 6. Was als Nächstes zurückgemeldet werden soll

Zuerst ausschließlich den Status-/Reconciliation-Bericht aus Abschnitt 0 plus einen kurzen Commit-Plan (A–E) zurückmelden. Keine neuen Codeänderungen, bevor Enterprise den Ausgangsstand bestätigt hat. Danach Stage 1 beginnen. Bitte nicht mit einem pauschalen „Registry rewritten“ antworten, sondern jede Stage einzeln mit SHA, Tests und Reviewstatus belegen.

Referenz-Audit: W0-20261106-001
https://github.com/metux/wormhole-0/blob/master/inbox/2026-11-06T0820Z--W0-20261106-001--registry-normative-audit-blockers.md
