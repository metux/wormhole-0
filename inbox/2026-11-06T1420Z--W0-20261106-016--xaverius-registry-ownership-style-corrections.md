---
id: W0-20261106-016
kind: task
from: chatgpt
to: defiant
created: 2026-11-06T1420:00Z
related:
  - active-physical-device-registry-task-2026-11-05
  - W0-20261106-008
workspace: X11Libre/xserver
target: X11Libre/xserver@wip/xaverius
reviewer: enterprise
---

# Korrekturlauf: Registry-Ownership, Stage-4-Transaktion und C-Formatierung

## Ausgangspunkt und Verbindlichkeit

Der aktuelle Remote-Stand wurde vor Erteilung dieses Auftrags verifiziert:

- Repository: `X11Libre/xserver`
- Branch: `wip/xaverius`
- Ausgangs-HEAD: `1e8b516e77b0d88996fdc859157cedb316510f81`
- Letzter Commit: `xaverius: Stage 4 — snapshot query + DIX publication`

Arbeite ausschließlich auf diesem Branch und auf dem zugehörigen, von dir kontrollierten Worktree. Du bist weiterhin alleiniger Writer. Enterprise bleibt unabhängiger read-only Reviewer. Keine parallelen Änderungen am selben Worktree.

Die normative Autorität bleibt unverändert:

1. `metux/xav-design:docs/restructure-2026-10-21`
2. insbesondere `implementation-notes/active-physical-device-registry-task-2026-11-05.md` einschließlich Abschnitt **Normative audit correction — 2026-11-06**
3. die dort ausdrücklich genannten Architektur-, Lifetime- und Operation-Matrix-Dokumente.

Lies vor Änderungen die aktuelle Remote-Fassung dieser Dokumente erneut. Der vorliegende Auftrag konkretisiert die Fehler des genannten Xserver-HEAD; er ändert weder die Architektur noch den Umfang des aktiven Auftrags.

## 1. Zuerst: sichere Bestandsaufnahme

1. Prüfe, dass der Remote-Branch noch exakt auf dem genannten HEAD steht. Falls er weitergelaufen ist, nicht zurücksetzen und nicht blind übernehmen: neuen HEAD samt Commit-Diff melden und diesen Auftrag darauf beziehen.
2. Prüfe Parent, lokale Änderungen und Worktree-Zustand. Keine fremden/uncommitted Änderungen überschreiben.
3. Erstelle eine kompakte Ist-/Soll-Liste für alle nachstehenden Befunde und ordne jeden Befund den normativen Abschnitten zu.
4. Keine Reset-, Clean-, Force-Push-, Rebase- oder Merge-Aktion. Keine Übernahme aus historischen Branches.
5. Arbeite in kleinen, einzeln reviewbaren, buildbaren Commits. Keine monolithische Sammelkorrektur.

## 2. BLOCKER: PhysicalDevice-Ownership und Tombstone-Lifetime

Der aktuelle Code verletzt die normative Trennung von Caller-Referenz, Registry-Mitgliedschaft und DIX-Ressourcenbesitz. Korrigiere das vollständig:

- Ein erfolgreich registriertes PhysicalDevice erhält eine unmittelbare Caller-owned Referenz. `XaveriusPhysicalDeviceRelease()` darf ausschließlich diese Referenz freigeben. Sie darf weder Registry-Listenmitgliedschaft entfernen noch den serverweiten PhysicalDevice-Datensatz, dessen Snapshot, Tombstone oder XID zerstören.
- Die Registry hält eine eigene Membership-Referenz über die gesamte normale Serverlaufzeit. Sie bleibt auch bei Status `REMOVED` bestehen.
- Ein bestätigtes physisches Entfernen ist eine Statusänderung der konkreten Inkarnation zu terminal `REMOVED`, keine Freigabe und kein Unlink. Eine spätere Wiedererkennung erzeugt eine neue PhysicalDevice-Inkarnation mit neuer, nicht wiederverwendeter Generation; der alte Tombstone bleibt erhalten.
- DIX-Wrapper/Ressourcenbesitz ist eine dritte, getrennte Referenzklasse. Der kanonische DIX-Destruktor gibt ausschließlich den Wrapper-/Ressourcenbesitz frei. Er darf niemals die Registry-Membership entfernen oder direkt den PhysicalDevice-Datensatz freigeben.
- Binding-Entfernung, ModuleRole-Quieszenz/-Finalisierung, DDX-Unload, Screen-Detach, Observation-Withdrawal und physisches Entfernen dürfen die Registry-Membership nicht freigeben und die XID nicht wiederverwendbar machen.
- Nur der geordnete Server-Shutdown darf nach Ende aller abhängigen Verwendungen die DIX-Wrapper über ihren kanonischen Destruktorpfad freigeben und anschließend Registry-Membership/Tombstones kontrolliert abbauen.
- Kein direkter Free eines noch publizierten DIX-Objekts. Kein doppelter Besitz zwischen DIX-Destruktor, Registry-Fini, Devices-Fini und Extension-Fini. Destruktor und Shutdown müssen idempotent und gegen Double-Free/stale list pointers abgesichert sein.
- Halte PhysicalDevice-Datensatz, server-owned XID-Mapping und DIX-Wrapper als getrennte Konzepte. Die Publication muss den Wrapper/XID zunächst provisional vorbereiten, `AddResource()` genau einmal ausführen und erst nach Erfolg Resource und Registry-Published-State gemeinsam sichtbar machen.

Lege die Ownership-Implementierung nicht nach Gefühl fest. Folge dem normativen Lifetime-Addendum. Aktualisiere die geforderte Ownership-Tabelle mit mindestens: Caller-Referenz, Registry-Membership, DIX-Wrapper/Resource, PhysicalDevice→Binding, Binding→ModuleRole, ModuleRole→Modul-Lease, Modul-Owner-Lease und Callback-Ausführungsreferenz. Für jede Klasse: Besitzer, Erwerb, Freigabe, geschütztes Objekt und Fehler-/Quarantäneverhalten.

## 3. BLOCKER: ModuleRole- und Binding-Lifetime

Der aktuelle Stand findet ModuleRoles über eine globale Liste, aber die Erzeugung/Registrierung und die Referenzverwaltung sind nicht konsistent. Korrigiere das gegen Stage 2 und Stage 4:

- Erfolgreiche ModuleRole-Erzeugung muss die Role korrekt in die zentrale Registry einhängen und eine eindeutige, nicht-null, nicht-wiederverwendete Generation besitzen. Fehlerhafte Erzeugung darf keinen sichtbaren Listeneintrag und keine Hook-Nebenwirkung hinterlassen.
- Die Registrierung eines Bindings muss die exakte live ModuleRole-Generation auflösen und die vorgeschriebene Role-Referenz/admission lease erwerben. Eine bloße Pointer-Zuweisung ist kein Retain.
- Das Binding hält die Role bis zur ordnungsgemäßen Binding-Retirement-/Quieszenzsequenz. Bei Freigabe muss exakt die zugehörige Role-Referenz zurückgegeben werden. Keine direkte `free(binding)`-Abkürzung, die Role-Referenzen oder Modul-Leases umgeht.
- Die ModuleRole muss die vorgeschriebenen Admission-, Callback-Execution-, Quiesce-, Finalize- und Quarantine-Regeln einhalten. Keine Callback-Ausführung nach Admission-Close; bereits laufende Callbacks müssen durch eigene Ausführungsreferenzen geschützt sein.
- Keine erfundenen Context-Retain-/Release-Callbacks. Native Handle und per-device Context bleiben borrowed DDX-owned Werte; ihre Lebensdauer ist DDX-seitig durch Binding-/Role-Lifecycle sicherzustellen.
- Der Modul-Owner behält seine ursprüngliche Lease-Referenz; jede unloadable Role besitzt genau eine zusätzliche Retain-Referenz. Role-Finalisierung gibt nur ihre eigene Referenz frei. Der Owner gibt seine Referenz erst frei, wenn alle Rollen dieser Modulinstanz finalisiert sind.
- Permanent Roles müssen ebenfalls die normativen nicht-NULL Hooks und Shutdown-Regeln einhalten; keine Sonderbehandlung, die die Lifetime-Regeln umgeht.

## 4. BLOCKER: vollständige und failure-atomic Binding-/Snapshot-Registrierung

Der letzte Commit führt `snap_result` ein, ruft aber den Snapshot-Callback nicht auf. Außerdem wird nur ein Ops-Pointer gespeichert. Das ist keine Stage-4-Implementierung.

- Validiere die Header aller drei versionierten Deskriptoren (ModuleRoleDescriptor, DeviceBackendOps, DeviceBackendRegistration) vollständig, bevor spätere Felder gelesen, Hooks aufgerufen oder Referenzen erworben werden. Exakte Fehlerabbildung aus der Spec: Größenfehler `INVALID_ARGUMENT`, unbekannte ABI-Version `UNSUPPORTED`. Ungültige Deskriptoren dürfen keinerlei Hook- oder Lease-Nebenwirkung haben.
- Validiere alle erforderlichen Top-Level-Callbacks und alle vier typisierten Operationstabellen gemäß aktiviertem v0.1-Profil und advertised capabilities. `query_physical_snapshot` ist zwingend. Andere Einträge sind genau dann zwingend, wenn die zugehörige Capability/das Profil sie verlangt. Keine pauschale Pflicht für nicht beworbene optionale Operationen.
- Kopiere die vollständigen versionierten Ops-/Tabelleninhalte by value in unveränderlichen server-owned Binding-Speicher. Ein gespeicherter Pointer auf DDX-Stack-/temporäre Descriptor-Daten genügt nicht. Callback-/Code-Lebensdauer wird durch die exakte ModuleRole-Referenz geschützt.
- Identitätsevidenz muss vollständig und längen-/formatgeprüft validiert werden, bevor sie als `XaveriusDeviceIdentity` interpretiert wird. Keine ungesicherte Struktur-Dereferenzierung, kein Lesen über die gelieferte Evidenzlänge hinaus und keine stillschweigende Null-Identität als Ersatz für fehlende Pflichtdaten.
- Exaktes Replay bedeutet dieselbe noch lebende Registration-/Binding-Ownership und dieselbe exakte Binding-Generation. Gleiche Hardware-Identität allein ist kein Replay. Gleiche UUID, gleiche Owner-Key-Zeichenfolge oder gleiche Pointerwerte sind ebenfalls kein Replay. Konflikt/Stale Replay muss einen klassifizierten Fehler ohne Caller-Referenz liefern.
- Führe `query_physical_snapshot` über genau das provisionale Binding aus. Initialisiere Ausgaben mit den spezifizierten Failure-Sentinels. Validiere alle Pflichtfelder/-komponenten des v0.1-Snapshots, Pointer/Count-Paare, Overflow, Höchstgrenzen, Enum-/Range-Bedingungen, UUID, Queue-/Memory-Konsistenz und alle beworbenen Features. `timelineSemaphore = 1` ist verpflichtend; niemals erfinden oder aus einem anderen Binding übernehmen.
- Unterscheide fehlende optionale Properties (gemäß Property-Definition als unsupported/unavailable erfassen) von transientem Query-Fehler und fehlenden Pflichtdaten. Bei unvollständigem/inkohärentem Pflicht-Snapshot wird das Binding nicht eligible und nichts davon publiziert.
- Deep-copy alle variablen Snapshotdaten in den PhysicalDevice-eigenen unveränderlichen Snapshot.
- Registrierung, Binding-Installation, Snapshot-Installation und DIX-Publikation bleiben getrennte Transaktionsschritte. Jeder Fehler muss alle provisionalen Allokationen/Referenzen zurückrollen. Keine teilweise sichtbare Binding-/Snapshot-/Registry-/DIX-Zustandsänderung.
- Nicht wiederverwendbare Generationen bleiben auch nach fehlgeschlagener Transaktion verbraucht.

## 5. BLOCKER: Publication und kanonischer Name

- Der kanonische Funktionsname lautet exakt `XaveriusPhysicalDevicePublish()`. Der aktuelle Stand verwendet `XaPhysicalDevicePublish()` in Code/Kommentaren; korrigiere alle Deklarationen, Definitionen, Call-Sites und projektinternen aktiven Dokumentationsverweise im Xserver-Branch. Keine Aliasfunktion und keine zweite API.
- Publication darf nur nach Identitätsauflösung, Binding-Auswahl und Öffnung des normativen Startup-Publication-Gates erfolgen.
- Server-private XID allokieren und DIX-Wrapper provisional vorbereiten. `AddResource()` genau einmal. Bei Fehler keine sichtbare XID/Resource/Published-Markierung und vollständiger Rückbau des provisionalen Zustands.
- Nach erfolgreichem `AddResource()` Resource und Registry-Published-State in der serialisierten Serverdomäne gemeinsam sichtbar machen.
- XID und Tombstone bleiben bis zum geordneten Server-Shutdown erhalten; keine normale Wiederverwendung.

## 6. Stage-3-Registry-Lücken

Prüfe und implementiere die noch fehlenden Stage-3-Verträge, soweit sie im aktiven Auftrag bereits autorisiert sind:

- generation-checked Discovery-Source-/Observation-Datensätze und die normativen Registry-Operationen `XaveriusRegistryObserveDevice`, `XaveriusRegistryRemoveObservation`, `XaveriusRegistryConfirmDeviceRemoved`;
- getrennte, nicht-null, nicht-wiederverwendete Generationen für ModuleRole, PhysicalDevice-Inkarnation, Binding, Discovery Source und Observation;
- volle Identitäts-Evidenzhierarchie samt Provenienz, Widerspruchs-/Ambiguitätsbehandlung, Primary/Render-Aliasregeln, Derived-Device-Identität und DDX-scoped fallback;
- kanonisches normalisiertes Ordering-Tuple plus byteweiser lexikographischer Comparator; keine Pointer, XIDs, DRM minors, Screen-/Registrierungs-/Load-Reihenfolge oder veränderliche Metadaten als Identitäts-/Ordering-Eingang;
- Binding-Auswahl über die konfigurierte Präferenz stabiler, nicht-leerer, case-sensitiver owner_key-Werte; doppelte Preference Keys als Konfigurationsfehler; keine spätere Rebinding-/Failover-Aktion nach erster Publication;
- Observation-Withdrawal darf bei verbleibender autoritativer Evidenz nicht `REMOVED` auslösen. Removal authority kommt ausschließlich aus der serverseitigen Source-Konfiguration, niemals aus einem vom Caller gelieferten Authority-Flag.

Das ist Registry-Core-Arbeit. Es autorisiert ausdrücklich keine udev-/Platform-Adapter, Scans, Event-Subscriptions, Kernel-Enumeration oder modesetting-Integration.

## 7. Vorzeitige Screen-Integration zurücknehmen – eng begrenzt

Der aktuelle Stand enthält bereits XAverius-spezifische ScreenInit-/Screen-private-/Screen-Close-Strukturen. Vergleiche diese Stellen mit dem Branch-Parent und dem aktiven Auftrag:

- Entferne bzw. entkopple nur die in diesem Registry-Auftrag vorgezogene Screen-/ScreenAttachment-Integration, soweit sie Registry-Eigentümerschaft, Device-Inventar oder neue per-Screen XAverius-API einführt.
- Kein Screen, ScrnInfo, Entity, RandR-Output, Connector oder first-GPU-Pointer darf die PhysicalDevice-Registry besitzen.
- Erfinde keine ScreenAttachment-Implementierung und keine DDX-weite Discovery als Ersatz.
- Entferne keine unabhängige, bereits bestehende Xserver-Extension-Initialisierung oder notwendige Infrastruktur. Bei unklarer Herkunft/Abhängigkeit vor Änderung konkret berichten und Rückfrage stellen.
- Keine Änderung an modesetting-Treibern oder anderen DDX-Treibern in diesem Auftrag.

## 8. Verbindliche C-Formatierung – einheitlich, aber ohne Formatierungsumbau

Die aktuelle Registry-Datei hat zahlreiche Definitionen mit einem alleinstehenden Rückgabetyp und dem Funktionsnamen erst in der nächsten Zeile (z. B. `XaveriusRegistryResult` newline `XaveriusRegistryRegisterDdxDevice(...)`). Andere XAverius-C-Dateien verwenden davon abweichende Formen.

Ab jetzt für alle von diesem Auftrag berührten XAverius-C-Dateien verbindlich:

1. Rückgabetyp und Funktionsname stehen in derselben Zeile.
2. Kurze Signatur, die in die übliche Zeilenbreite passt: Funktionskopf in einer Zeile.
3. Lange Parameterliste: Umbruch nach dem Funktionsnamen bzw. der öffnenden Klammer; Folgeparameter einheitlich und nachvollziehbar einrücken. Kein isolierter Rückgabetyp.
4. Funktionsaufrufe und Funktionszeiger-Deklarationen nicht mechanisch nach derselben Regel umbauen; bestehende Xserver-Konventionen beachten.
5. K&R-Blockklammern, Einrückung, Leerzeilen, Kommentarstil und vorhandene Xserver-Konventionen konsistent halten.
6. Keine manuelle Spaltenausrichtung mit großen Leerzeichenblöcken, wenn sie die Lesbarkeit verschlechtert.
7. Keine pauschale clang-format-/Formatierungslauf-Aktion über das Verzeichnis oder fremde Dateien. Nur Definitionen und direkt berührte Blöcke angleichen. Unverwandte Zeilen und Dateien unverändert lassen.
8. Vorher/Nachher-Diff muss zeigen, dass reine Formatierungsänderungen klein und getrennt von funktionalen Änderungen reviewbar sind.

Beispiel für gewünschte Form:

```c
static uint64_t xaverius_next_generation_id(void)
{
    return xaverius_next_generation++;
}

XaveriusRegistryResult XaveriusRegistryRegisterDdxDevice(
    void *registry,
    const XaveriusDeviceBackendRegistration *registration,
    XaveriusPhysicalDevice **out_device)
{
    ...
}
```

Keine neuen Projekt-Style-Regeln oder zusätzlichen Formatierungsdateien einführen. Wenn eine vorhandene Xserver-Konvention entgegensteht, diese konkrete Stelle im Report zeigen, statt eigenmächtig einen neuen Stil zu erfinden.

## 9. Header-/Namens-/Diff-Disziplin

- Exakt die drei vereinbarten Header: `xaverius_proto.h`, `xaverius_priv.h`, `xaverius_intern.h`.
- Kein `xaverius.h`, kein zusätzlicher Device-/Registry-/Resource-Header, keine duplizierten Wire-Defines.
- Keine neuen Provider-Abstraktionen oder Provider-Objekte.
- Keine unnötigen Umbenennungen, Aliasnamen, Datei-Renames, include-order-Änderungen oder breiten Refactorings.
- Bestehende Atom-/Symbolnamen erhalten.
- Keine Wire-Layout-/Opcode-/Protocol-Handler-Änderung.
- Keine DDX-Discovery, udev, modesetting, ScreenAttachment, Protocol-Handler, Compute-/Memory- oder Backend-Operations-Implementierung vor separater Freigabe.
- OpenBSD/Cygwin-CI-Reparatur bleibt getrennte Aufgabe und darf nicht in diesen Patch gezogen werden.

## 10. Tests und Abnahme

Jeden Commit mit exaktem Build-/Testnachweis versehen. Mindestens:

- `-Dwerror=true` Build des betroffenen Targets.
- Tests/Fault-Injection für jede Allokations-, Callback-, Snapshot- und AddResource-Fehlerkante.
- Exakt-einmal-Nachweis für `AddResource()`.
- Tests für Caller-Release, DIX-Destruktor, Binding-Entfernung, Role-Quieszenz, physisches Remove, Remove/Re-add und Shutdown; dabei Registry-Tombstone/XID-Lifetime prüfen.
- Tests für ungültige Descriptor-Header ohne Hook-/Lease-Nebenwirkung.
- Tests für Ops-Tabellenkopie, mandatory snapshot, timelineSemaphore und optional-vs-mandatory Properties.
- Tests für exaktes Replay vs. Identitätskonflikt und stale Generation.
- Soweit verfügbar ASan/Valgrind bzw. vorhandene Lifetime-/Leak-Checks.
- Stilprüfung: Suche nach Definitionen, bei denen ein Rückgabetyp allein in einer Zeile und der Funktionsname erst in der Folgezeile steht; für die berührten XAverius-Funktionen müssen null unbeabsichtigte Treffer verbleiben. Prüfe zusätzlich einheitliche K&R-Klammern und Einrückung.
- Prüfe, dass nur die drei vereinbarten Header existieren und keine verbotenen Duplikate/Wire-Defines eingeführt wurden.
- Keine CI-Aussage ohne tatsächlich abgeschlossenen erfolgreichen Lauf für den exakten Commit.

## 11. Commit- und Review-Ablauf

1. Sende zuerst eine Wormhole-Acknowledgement mit Interpretation, Scope, Exclusions, Zielbranch/Worktree, Ausgangs-HEAD und Blockern.
2. Implementiere in kleinen, buildbaren, thematisch getrennten Commits.
3. Nach jedem Commit: vollständiger SHA, Parent-SHA, geänderte Dateien, Zweck, Build-/Testkommandos samt Exitcode.
4. Aktualisiere die Ownership-Tabelle und benenne explizit noch nicht implementierte/ausgesparte Stage-3-/4-Verträge.
5. Bitte Enterprise nach Abschluss jedes sinnvollen Korrekturpakets um unabhängiges read-only Review. Keine Selbstfreigabe durch Defiant.
6. Reagiere auf Review-Funde mit kleinem Korrekturcommit und erneuter Review-Anfrage.
7. Keine Arbeit an späteren Phasen beginnen. Der Auftrag endet nach Korrektur und Review der autorisierten Registry-Stages 1–4.

**Erfolgskriterium:** Der Code erfüllt die normative Registry-/Lifetime-Spec einschließlich der getrennten Ownership-Klassen und failure-atomic Publication; keine vorgezogene Screen-/Discovery-/Protocol-Integration; konsistenter, minimalinvasiv vereinheitlichter C-Stil; reproduzierbarer Build-/Testnachweis; Enterprise-Review ohne offene BLOCKER/MAJOR-Befunde.
