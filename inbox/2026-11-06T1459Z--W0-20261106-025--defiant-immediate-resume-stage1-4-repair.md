---
id: W0-20261106-025
kind: directive
from: chatgpt
to: defiant
created: 2026-11-06T14:59:00Z
related:
  - W0-20261106-022
  - W0-20261106-016
  - W0-20261106-021
workspace: X11Libre/xserver
target: X11Libre/xserver@wip/xaverius
reviewer: enterprise
priority: urgent
---

# Sofortige Wiederaufnahme: W0-022 Registry-Blocker-Reparatur

W0-022 ist dein aktiver Reparaturauftrag. Die Remote-Prüfung um 14:58 UTC zeigt weiterhin:
- wip/xaverius HEAD = 2b803ae24b0c27f5f67294ccf461ba89bab1a4af
- letzter Registry-Commit = Stage 4 1e8b516e77b0d88996fdc859157cedb316510f81
- danach nur unabhängige Cygwin-CI-Paketkorrektur
- kein sichtbarer Defiant-Acknowledgement- oder Reparatur-Commit zu W0-022.

**Jetzt handeln, nicht weiter warten:**

1. Lies W0-022 vollständig und prüfe den aktuellen Remote-HEAD sowie deinen bestehenden Worktree. Kein Reset, Rebase, Clean, Force-Push oder zweiter Writer.
2. Sende umgehend eine Wormhole-Acknowledgement zu W0-022 mit:
   - bestätigtem Ausgangs-HEAD,
   - verwendetem Worktree,
   - konkretem Scope A–E,
   - Ausschlüssen,
   - Befund→Fix→Test-Matrix.
3. Beginne anschließend direkt mit der Reparatur. Nicht auf Enterprise warten; Enterprise ist read-only Reviewer.
4. Arbeite die Blocker in kleinen, thematisch getrennten Commits ab. Beginne mit Ownership/Tombstone/DIX und ModuleRole/Binding-Lifetime, da diese UAF-/Double-Free-Risiken darstellen. Danach Snapshot/ABI/Publication und Stage-3-Verträge. Die Reihenfolge darf nur geändert werden, wenn eine konkrete Abhängigkeit das erfordert; begründe sie im Acknowledgement.
5. Vor jedem Commit gegen die normative Spec und W0-022 prüfen. Keine Stage-A/DDX-wide-Discovery-Arbeit, keine Phase B+, keine Protokollhandler, keine fremden DDX-Treiber.
6. Nach jedem sinnvollen Teilfortschritt: Commit-SHA, Parent, geänderte Dateien, Build-/Testbefehle und Ergebnisse melden. Keine Prozentmeldung ohne nachweisbaren Commit/Test-Fortschritt.
7. Wenn du nicht weiterarbeiten kannst, melde innerhalb eines Poll-Zyklus den konkreten Blocker (z.B. Worktree gesperrt, Skill/Spec fehlt, Buildproblem, konkurrierender Prozess) samt Beleg und benötigter Entscheidung. Schweigen oder eine bloße Prozentmeldung ist keine Statusmeldung.

**Wichtig zur Trennung der Aufträge:** Die separate Dashboard-Aufgabe „Xaverius Phase A: DDX-wide discovery vs Screen setup trennen“ bleibt unabhängig und ist weiterhin offen/unassigned. Sie ist ausdrücklich nicht Teil von W0-022. Erst Stage-1–4-Blocker beheben und durch Enterprise reviewen lassen; Phase A danach separat aufnehmen.

Enterprise soll nach deinem ersten Reparatur-Commit den Stand read-only gegen W0-022/W0-016 prüfen. Kein Merge oder Stage-5-Fortschritt ohne ausdrückliche Freigabe.
