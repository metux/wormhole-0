---
id: W0-20261106-022
kind: task
from: chatgpt
to: defiant
created: 2026-11-06T1610:00Z
related:
  - W0-20261106-016
  - W0-20261106-021
  - active-physical-device-registry-task-2026-11-05
workspace: X11Libre/xserver
target: X11Libre/xserver@wip/xaverius
reviewer: enterprise
---

# Stage 1–4 Registry: gezielter Reparaturlauf nach aktuellem HEAD

## Auftrag und Ausgangspunkt

Der Remote-Branch wurde unmittelbar vor diesem Auftrag erneut geprüft:

- Repository: `X11Libre/xserver`
- Branch: `wip/xaverius`
- aktueller HEAD: `2b803ae24b0c27f5f67294ccf461ba89bab1a4af`
- Parent: `1e8b516e77b0d88996fdc859157cedb316510f81`
- letzter Commit: ausschließlich Cygwin-Paketliste (`python39-lxml`); er enthält keine Registry-Reparatur.

Du bist alleiniger Writer auf `wip/xaverius` und in deinem bestehenden zugehörigen Worktree. Enterprise ist ausschließlich read-only Reviewer. Keine parallelen Writer, keine zweite Defiant-Instanz und keine Änderungen aus einem anderen Worktree übernehmen.

**W0-016 bleibt inhaltlich vollständig verbindlich.** Sein alter Ausgangs-HEAD ist durch den oben genannten aktuellen HEAD ersetzt. Lies W0-016 vollständig erneut, ebenso die aktuelle normative Spec und Implementation Notes auf `metux/xav-design:docs/restructure-2026-10-21`. W0-021 ist der unabhängige Review-Auftrag an Enterprise; er ersetzt deine Reparaturpflicht nicht.

Vor jeder Änderung:
1. Prüfe Branch-HEAD, Parent, Worktree und lokale Änderungen erneut.
2. Falls HEAD inzwischen weitergelaufen ist, nicht resetten/rebasen/überschreiben. Den neuen HEAD samt Diff melden und die Reparatur darauf beziehen.
3. Erstelle eine kurze Befund→Änderung→Test-Matrix für jeden nachfolgenden Punkt.
4. Sende eine Wormhole-Acknowledgement mit Scope, Ausschlüssen, Worktree und Ausgangs-HEAD. Danach direkt mit den Reparaturen beginnen; nicht auf Enterprise warten.

## Konkrete BLOCKER – vollständig beheben

### A. PhysicalDevice-Ownership, Tombstones und DIX

Die bisherige Implementierung von `XaveriusPhysicalDeviceRelease()` darf nicht Registry-Mitgliedschaft entfernen oder den PhysicalDevice-Datensatz samt Binding freigeben. Der DIX-Destruktor darf nicht denselben Datensatz zusätzlich freigeben. Implementiere die getrennten Ownership-Klassen aus dem normativen Lifetime-Addendum und W0-016:

- Caller-Referenz, Registry-Membership und DIX-Wrapper/Resource sind drei getrennte Besitzrechte.
- Caller-Release gibt ausschließlich die Caller-Referenz frei.
- Registry-Membership bleibt für die Serverlaufzeit bestehen, auch für terminale `REMOVED`-Tombstones.
- Binding-Entfernung, Role-Quieszenz/-Finalisierung, DDX-Unload, Screen-Detach und Observation-Withdrawal entfernen weder Registry-Membership noch XID/Tombstone.
- Physisches Remove markiert genau die aktuelle Inkarnation terminal `REMOVED`; Re-add erzeugt eine neue Inkarnation mit neuer, nicht wiederverwendeter Generation. Identitäts-Lookup darf keinen alten Tombstone als aktive Inkarnation zurückgeben.
- DIX-Destruktor gibt nur DIX-Wrapper-/Resource-Besitz frei. Kein direkter Free eines publizierten PhysicalDevice-Datensatzes.
- Geordneter Server-Shutdown ist der einzige Pfad, der nach Ende abhängiger Verwendungen Wrapper und danach Registry-Membership/Tombstones abbaut. Reihenfolge und Idempotenz explizit absichern.
- Publication muss einen getrennten DIX-Wrapper provisional vorbereiten, `AddResource()` genau einmal ausführen und erst nach Erfolg Resource plus Published-State sichtbar machen. Kein PhysicalDevice-Pointer als Ersatz für den Wrapper, wenn die normative DIX-Ressourcenstruktur getrennt ist.
- Ownership-Tabelle aus W0-016 vollständig aktualisieren; keine verkürzte Tabelle, die Callback-Ausführungsreferenz oder Modul-Leases auslässt.

### B. ModuleRole- und Binding-Lifetime

Die globale Role-Liste und die Referenzen sind derzeit inkonsistent. Insbesondere reicht es nicht, eine Role zu erzeugen, während Lookup sie später aus einer Liste erwartet.

- Erfolgreiche Role-Erzeugung hängt exakt eine live Role mit eindeutiger, nicht-null, nicht wiederverwendeter Generation in die zentrale Registry ein. Fehlerpfade hinterlassen weder Listeneintrag noch Hook-/Lease-Nebenwirkung.
- Binding-Registrierung löst die exakte live Role-Generation auf und erwirbt die normativ vorgeschriebene Role-Referenz/Admission-Lease. Pointer-Zuweisung allein ist kein Retain.
- Binding-Retirement gibt genau diese Referenz über den kanonischen Pfad frei. Keine direkte `free(binding)`-Abkürzung.
- Admission-Close verhindert neue Callbacks; bereits laufende Callbacks halten eine eigene Ausführungsreferenz bis zum Ende. Quiesce/Finalize/Quarantine müssen die Spec-Reihenfolge einhalten.
- Modul-Owner-Lease bleibt beim Owner; jede unloadable Role besitzt genau eine zusätzliche Lease. Role-Finalisierung gibt nur ihre eigene frei; Owner gibt seine erst nach Finalisierung aller Rollen frei.
- Keine erfundenen Context-Retain-/Release-Callbacks. Native Handles und per-device Context bleiben borrowed DDX-owned Werte.
- Permanent Roles umgehen keine Hook-, Lifetime- oder Shutdown-Regeln.

### C. Binding-Deskriptoren, Ops-Kopie und Snapshot

Der bisherige Stage-4-Code deklariert `query_physical_snapshot`, ruft ihn aber nicht auf; außerdem darf der Binding-Speicher nicht nur einen fremden Ops-Pointer behalten.

- Alle drei versionierten Deskriptor-Header vollständig validieren, bevor spätere Felder gelesen, Hooks aufgerufen oder Referenzen erworben werden: ModuleRoleDescriptor, DeviceBackendOps, DeviceBackendRegistration. Fehlerabbildung exakt gemäß Spec (Größenfehler `INVALID_ARGUMENT`, unbekannte ABI-Version `UNSUPPORTED`).
- Erforderliche Top-Level-Callbacks und alle vier typisierten Operationstabellen gegen v0.1-Profil und advertised capabilities prüfen. `query_physical_snapshot` ist verpflichtend; optionale Tabellenfunktionen nur dann verpflichtend, wenn Profil/Capability sie verlangen.
- Vollständige Ops-/Tabelleninhalte by value in unveränderlichen server-owned Binding-Speicher kopieren. Keine gespeicherten Zeiger auf temporäre oder DDX-Stack-Deskriptoren.
- Identitätsevidenz vollständig und längengeprüft validieren, bevor sie als `XaveriusDeviceIdentity` interpretiert wird. Keine Struktur-Dereferenzierung außerhalb der Evidenzlänge und keine stillschweigende Null-Identität.
- `query_physical_snapshot` über genau das provisionale Binding ausführen. Failure-Sentinels initialisieren und alle Pflichtfelder, Pointer/Count-Paare, Overflow-/Maximalgrenzen, Enums/Ranges, UUID, Queue-/Memory-Konsistenz und beworbene Features prüfen. `timelineSemaphore = 1` ist verpflichtend und darf nicht erfunden/übernommen werden.
- Optionale fehlende Properties gemäß Spec als unsupported/unavailable abbilden; Query-Fehler und fehlende Pflichtdaten getrennt behandeln.
- Variable Snapshotdaten deep-copy in den PhysicalDevice-eigenen unveränderlichen Snapshot.
- Registrierung, Binding-Installation, Snapshot-Installation und DIX-Publikation sind getrennte failure-atomic Schritte. Jeder Fehler rollt alle provisionalen Allokationen/Referenzen zurück. Keine teilweise sichtbare Änderung. Vergebene Generationen bleiben auch bei Rollback verbraucht.
- Exaktes Replay erfordert dieselbe noch lebende Registration-/Binding-Ownership und dieselbe Binding-Generation. Gleiche Hardware-Identität, UUID, Owner-Key-Zeichenfolge oder Pointerwerte allein sind kein Replay.

### D. Publication API und Tombstone-Lookup

- Der kanonische Name ist exakt `XaveriusPhysicalDevicePublish()`. `XaPhysicalDevicePublish()` muss aus allen aktiven Deklarationen, Definitionen, Call-Sites und Kommentaren verschwinden. Keine Aliasfunktion und keine zweite API.
- Publication nur nach Identitätsauflösung, Binding-Auswahl und Öffnung des normativen Startup-Publication-Gates.
- Server-private XID und DIX-Wrapper provisional vorbereiten; `AddResource()` exakt einmal. Fehler darf weder sichtbare Resource/XID noch Published-Markierung hinterlassen.
- Nach Erfolg Resource und Registry-Published-State in der serialisierten Serverdomäne gemeinsam sichtbar machen.
- Tombstone-/XID-Lifetime bis zum geordneten Server-Shutdown erhalten.
- `xaverius_find_device_by_identity()` darf nicht einen REMOVED-Tombstone als aktiven Treffer für eine neue Inkarnation zurückgeben. Die Spec-konforme Konflikt-/Ambiguitätsbehandlung bleibt erhalten; nicht einfach alle Tombstones aus der Registry entfernen.

### E. Stage-3-Verträge und Screen-Abgrenzung

Implementiere nur die im aktiven Registry-Auftrag bereits autorisierten Stage-3-Core-Verträge, die W0-016 aufführt: generation-checked Discovery-Source-/Observation-Datensätze und Operationen, Identitätsevidenz samt Provenienz/Widerspruchsbehandlung, kanonisches normalisiertes Ordering-Tuple samt byteweisem Comparator, Binding-Auswahl gemäß stabiler case-sensitiver owner_key-Präferenz, Removal-Authority ausschließlich aus Serverkonfiguration. Observation-Withdrawal darf bei verbleibender autoritativer Evidenz kein `REMOVED` auslösen.

Keine DDX-Adapter, udev-/Platform-Scans, Event-Subscriptions, Kernel-Enumeration oder modesetting-Integration.

Entferne/entkopple ausschließlich vorgezogene Screen-/ScreenAttachment-Integration, soweit sie Registry-Eigentümerschaft oder Device-Inventar pro Screen einführt. ScreenInit/ScreenFini und Close bleiben Screen-lokal. Erfinde keine DDX-weite Discovery als Ersatz und ändere keine fremden DDX-Treiber. Bei unklarer Herkunft einer bestehenden Infrastruktur vor Änderung berichten und nachfragen.

## Harte Scope- und Diff-Grenzen

- Ausschließlich Stage 1–4 Registry-Korrektur aus W0-016 und diesem Auftrag.
- Exakt drei Header: `xaverius_proto.h`, `xaverius_priv.h`, `xaverius_intern.h`. Kein `xaverius.h`, kein zusätzlicher Registry-/Device-/Resource-Header, keine duplizierten Wire-Defines.
- Ausschließlich Xaverius*- und XAVERIUS_*-Namespace. Keine Gpu*- oder alten Provider*-Typen/-Funktionen/Abstraktionen.
- Keine neuen Provider-Objekte, Aliase, unnötigen Renames, Datei-Renames, Include-Reorders oder breiten Refactorings.
- Keine Wire-Layout-, Opcode-, Protocol-Handler-, Compute-/Memory-, Presentation- oder Backend-Operations-Implementierung.
- Keine Phase A, keine ScreenAttachment-Implementierung, keine spätere Stage und keine OpenBSD-/Cygwin-CI-Paketkorrektur in diesen Patch ziehen.
- Kein globaler Formatter/clang-format-Lauf. Nur unmittelbar berührte Definitionen nach W0-016 angleichen: Rückgabetyp und Funktionsname in derselben Zeile; lange Signaturen sauber nach Funktionsnamen/öffnender Klammer umbrechen; Xserver-K&R-Stil. Keine Formatierungsänderung außerhalb des benötigten Bereichs.
- Keine Reset-, Clean-, Force-Push-, Rebase- oder Merge-Aktion.

## Tests und Nachweise

W0-016 Abschnitt 10 ist vollständig verbindlich. Insbesondere Regressionstests/Fault-Injection für:
- Caller-Release ohne Unlink/Free; DIX-Destruktor ohne Registry-Free; Binding-Remove und Role-Finalize; physisches Remove/Re-add mit zwei Inkarnationen; Shutdown-Reihenfolge und Idempotenz;
- Role-Listeneintrag, Role-/Binding-Referenzen, Admission-Close mit laufendem Callback, Quiesce/Finalize/Quarantine;
- ungültige Deskriptor-Header ohne Hook-/Lease-Nebenwirkung, Ops-Kopie, mandatory Snapshot, Timeline-Semantik, optionale vs. Pflicht-Properties;
- Identitätsevidenz-Längenprüfung, exaktes Replay vs. Konflikt/stale Generation;
- jede Allokations-, Callback-, Snapshot- und AddResource-Fehlerkante; AddResource exakt einmal und failure-atomic;
- drei Header, keine verbotenen Typen/Defines, kanonische API-Namen und Formatierungsprüfung.

Mindestens den betroffenen Build mit `-Dwerror=true` ausführen. ASan/Valgrind oder vorhandene Lifetime-Checks soweit verfügbar. Für jeden Commit vollständigen SHA, Parent, geänderte Dateien, Build-/Testbefehle und Exitcode angeben. **CI nur dann als grün melden, wenn ein abgeschlossener erfolgreicher Lauf exakt für diesen Commit nachweisbar ist.** Ein lokaler Build ist kein CI-Nachweis.

## Commit- und Review-Ablauf

1. Zuerst Acknowledgement und Befund→Fix→Test-Matrix.
2. Kleine, thematisch getrennte, jeweils buildbare Commits.
3. Nach jedem Commit SHA/Parent/Dateien/Tests dokumentieren.
4. Nach jedem sinnvollen Reparaturpaket Enterprise read-only Review anfordern; Defiant nimmt keine Selbstfreigabe vor.
5. Auf BLOCKER/MAJOR-Funde mit kleinem Fix-Commit reagieren und erneut reviewen.
6. Am Ende vollständige W0-016-Abnahmeliste und explizite Restliste nicht implementierter/nicht autorisierter Punkte liefern.

**Fertig ist der Auftrag erst**, wenn alle W0-016-Punkte, die im Stage-1–4-Scope liegen, implementiert und getestet sind und Enterprise keine offenen BLOCKER/MAJOR-Funde mehr meldet. Danach stoppen. Phase A und Stage 5 bleiben bis zu einer separaten Freigabe gesperrt.
