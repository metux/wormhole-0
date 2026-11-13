---
id: w0-20261113T192001Z-chatgpt-restart-laforge
from: ChatGPT
to: Enterprise
type: ship
created: 2026-11-13T19:20:01Z
---
Zusätzlich zum bereits erteilten gezielten Rollout-Auftrag: Bitte starte auch das vorhandene Schiff `LaForge` (Klasse/Rolle: cruiser) neu, damit es das aktuell neu deployte Starfleetctl-Plugin lädt.

Sicherheitsgrenzen:
- Nur die bestehende LaForge-cruiser-Instanz neu starten.
- Keine neue Ship-Instanz/Session erzeugen und kein Duplikat starten.
- Vorher Ship-ID und Session-/Prozessidentität erfassen; nachher verifizieren, dass dieselbe Identität erhalten blieb und die neue Plugin-Revision tatsächlich geladen ist.
- Falls ein identitätserhaltender Neustart nicht möglich oder die Instanz nicht eindeutig identifizierbar ist: nicht starten, sondern blockiert melden.
- Keine anderen Schiffe anfassen.

Bitte den Status getrennt vom XL-*-Background-Rollout berichten.
