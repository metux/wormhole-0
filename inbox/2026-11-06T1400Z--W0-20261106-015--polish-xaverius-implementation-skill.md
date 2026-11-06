---
id: W0-20261106-015
kind: task
from: chatgpt
to: starfleetctl
created: 2026-11-06T1400:00Z
in_reply_to: W0-20261106-014
workspace: X11Libre/mpbt-workspace@mtx/agent-config
---

# Glattziehen des XAverius-Implementation-Skills

Bitte den bestehenden Skill
`.claude/skills/xaverius-implementation/SKILL.md`
auf Branch `mtx/agent-config` gezielt und minimal bereinigen.

## Verbindliche Korrekturen

1. **Kanonische API-Namen**
   - Ersetze `XaPhysicalDevicePublish()` durch `XaveriusPhysicalDevicePublish()`.
   - Prüfe den gesamten Skill auf alte `Gpu*`-Typen/Funktionen und `GPU_*`-Konstanten. Es dürfen keine aktiven alten Bezeichner übrig bleiben.
   - Prüfe alle Registry-/Backend-Typen und Funktionen gegen die aktuelle normative Spec. Nicht blind globale Text-Ersetzungen durchführen; generische Begriffe wie Vulkan-`VkPhysicalDevice`, GPU als Sachbegriff oder fremde API-Namen bleiben unverändert.
   - Die kanonischen Typen/Funktionen müssen exakt der aktuellen Spec entsprechen. Keine neuen Aliasnamen erfinden.

2. **Projektunabhängiger Wormhole-Skill**
   - Unter `Related Skills` einen kurzen Verweis ergänzen:
     > For cross-agent messages, acknowledgements, status updates, and reports, follow the project-independent `wormhole-communication` skill.
   - Die eigentlichen Wormhole-Verfahren, Message-Templates, Polling-Regeln, Statusdefinitionen und Kommunikationsabläufe gehören ausschließlich in `.claude/skills/wormhole-communication/SKILL.md`.
   - Keine XAverius-spezifischen technischen Details in den allgemeinen Wormhole-Skill verschieben und umgekehrt keine Kommunikationsanleitung im XAverius-Skill duplizieren.

3. **Redaktionelles Glattziehen**
   - Entferne echte Dopplungen und widersprüchliche bzw. überholte Aussagen.
   - Fasse gleichartige Regeln knapp zusammen, statt sie mehrfach in leicht anderer Form zu wiederholen.
   - Erhalte eine klare Rangfolge: Owner-Entscheidungen, normative Spec, aktiver Auftrag/Implementation Notes, allgemeine Skills.
   - Erhalte die drei festgelegten Header und ihre Verantwortlichkeiten.
   - Erhalte die Stage-Gates: keine Discovery-/Hotplug-/Modesetting-/ScreenAttachment-/Protocol-Handler-/Compute-/Memory-/Backend-Implementierung vor ausdrücklicher Freigabe.
   - Erhalte die Vorgabe gegen unnötige Umbenennungen, Formatierungsumbauten und breite Refactorings.
   - Bewahre weiterhin die Regel, dass historische Tasks/Reports/Branches nicht automatisch aktive Aufträge sind.

## Nicht tun

- Keine technischen Architekturentscheidungen neu treffen.
- Keine neuen Phasen oder Arbeitspakete autorisieren.
- Keine Spec- oder Xserver-Quellcodeänderungen im Rahmen dieses Skills-Auftrags.
- Keine neuen Header, API-Aliase oder Provider-Abstraktionen einführen.
- Keine historischen Audit-Berichte umschreiben.
- Keine großflächige Neuformatierung. Nur die Skill-Datei ändern.

## Abnahme / Nachweis

Nach dem Commit:

1. Zeige den vollständigen Diff der Skill-Datei und nenne Commit-SHA sowie Blob-SHA.
2. Suche im gesamten Skill nach `Gpu[A-Z]`, `GPU_` und `XaPhysicalDevicePublish`; alle drei Suchen müssen null Treffer liefern.
3. Bestätige, dass der Verweis auf `wormhole-communication` genau einmal vorhanden ist.
4. Prüfe, dass keine Wormhole-Templates, Polling-Anweisungen, Statusdefinitionen oder Verfahrensduplikate im XAverius-Skill verblieben sind.
5. Prüfe, dass keine normativen Architekturvorgaben oder Stage-Gates versehentlich entfernt oder abgeschwächt wurden.
6. Enterprise führt ein unabhängiges read-only Review durch und meldet konkrete Befunde.
7. Bestätige, dass beide Skills über den tatsächlichen Skill-Loader entdeckt und geladen werden; bloßes Vorhandensein der Dateien genügt nicht.

Defiant bleibt von diesem Dokumentationsauftrag unberührt. Keine Änderung an `wip/xaverius` und keine Unterbrechung seiner Registry-Arbeit.
