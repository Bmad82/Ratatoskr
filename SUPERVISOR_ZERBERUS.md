# SUPERVISOR_ZERBERUS.md – Zerberus Pro 4.0

<!-- STAND-ANKER:START -->
**Stand:** P221-v2b - Phase 5b - 2026-05-21 - 2900+ Tests
**Commits:** Zerberus `4edfd35` - Ratatoskr `6bf7936` - Claude `9a47b1d`
_Single Source of Truth: STAND.json - propagiert von scripts/propagate_stand.py._
<!-- STAND-ANKER:END -->

*Strategischer Stand für die Supervisor-Instanz (claude.ai Chat). Historische Patch-Einträge liegen in [`SUPERVISOR_ZERBERUS_ARCHIV_2026-05-21.md`](SUPERVISOR_ZERBERUS_ARCHIV_2026-05-21.md) (kompletter Schnitt vor v2b-Entschlackung) und in [`docs/PROJEKTDOKUMENTATION.md`](docs/PROJEKTDOKUMENTATION.md). Diese Datei trägt nur den heißen Stand.*

---

## Aktueller Patch

**FR 2026-05-23 orchestrate-mvp (2026-05-23)** — ORCHESTRATE als dritter Pipeline-Pfad. Pre-call Detector (`zerberus/core/orchestrate_intent.py`, Zwei-Pfad-Logik) + `HuginnIntent.ORCHESTRATE`-Enum, OrchestratorService mit Plan-Call + Synthese (`zerberus/core/orchestrate.py`, robustes JSON-Parsing, Cost-Estimation), HitL-Gate (`zerberus/core/orchestrate_hitl.py`, P206-Pattern + Per-Step-Opt-out), Worker-Dispatcher (`zerberus/core/orchestrate_worker.py`, sequenzielle Kontext-Kette, SSE-Progress, Guard-DI), Wiring in `legacy.py` chat_completions VOR `load_system_prompt` + Endpoints `/v1/orchestrate/poll` und `/v1/orchestrate/resolve`. Alle LLM-Calls ueber OpenRouter / DeepSeek V3.2 — KEIN Claude-Call im neuen Pfad (Anthropic-Billing-Split 15.06.2026). 87 neue Tests gruen (Intent 17, Service 22, HitL 19, Worker 15, Legacy-Wiring 14), kein Regress. Schritt 6 (Sandbox-Integration) und Frontend-Plan-Karte als Backlog B-080..B-084.

## Vorheriger Patch

**Sammel-Doku FR 2026-05-23 Pipeline-Diagnose (2026-05-23)** — Sidebar NULL-safe NOT IN, Modell-Pille re-hydrate, chat_history_max_messages 40, DECISIONS D-007, RAG-Eval Phase 5b Audit. Details in `mjolnir.md`-Vorlauf-Eintrag und HANDOVER.

## Vorvorheriger Patch

**Komponenten-Migration N+5..N+10 (2026-05-21)** — Kintsugi-Typografie auf Reasoning/Input/Scroll-Nav/Project-View/Settings + Final Cleanup. 759/759 Tests grün. Detail im Archiv-Eintrag.

## Phase 5 — Status

Phase 5a (P194-P203) ABGESCHLOSSEN. Phase 5b läuft — Warm-up komplett ERLEDIGT (B-073 WhatsApp-Removal, B-074 Lessons-Cron, DECISIONS_ZERBERUS, Whisper-INT8-Werkzeug; A/B-Messung INT8 vs. FP16 offen als D-004). Power-Features-Roadmap (Cluster 1–3) in [`MARATHON_WORKFLOW_ZERBERUS.md`](MARATHON_WORKFLOW_ZERBERUS.md) als Single Source of Truth — die frühere separate `PHASE_5B_ROADMAP_ZERBERUS.md` wurde per FR 2026-05-22 in den Workflow eingehängt und gelöscht. Phase 5c (UI-Redesign P-UI-1..11) ABGESCHLOSSEN.

## Architektur-Referenz

**Tech-Stack:** DeepSeek V3.2 (Cloud-LLM, OpenRouter) | Mistral Small 3 (Guard) | Gemma 4 E2B (Prosodie, lokal ~3.4 GB) | faster-whisper large-v3 FP16 (ASR, Docker Port 8002) | cross-en-de-roberta (Embeddings DE, GPU) | bge-reranker-v2-m3 | SQLite (`bunker_memory.db`, WAL, Alembic seit P92) | Nala (Mobile-first) + Hel (Admin) | Huginn-Telegram (Long-Polling, Tailscale). **VRAM** (Nala+Prosodie): ~10.3 GB / 12 GB (RTX 3060). **Repos** (alle drei synchron via `scripts/sync_repos.ps1`): `C:\Users\chris\Python\{Zerberus, Rosa\Nala_Rosa\Ratatoskr, Claude}`.

## Offene Items / Strukturelle Schulden

→ Konsolidiert in [`BACKLOG_ZERBERUS.md`](BACKLOG_ZERBERUS.md). Hier nur strukturelle Schulden:

- **Persona-Hierarchie** Hel ↔ Nala — löst sich mit B-071 (SillyTavern/ChatML-Wrapper).
- **`interactions`-Tabelle** ohne User-Spalte — Per-User-Metriken erst nach Alembic-Fix vertrauenswürdig.
- **Voice-Messages** in Telegram-DM (P182 antwortet höflich, echte Whisper-Pipeline = B-072).

## Architektur-Warnungen

- **Rosa Security Layer:** NICHT implementiert — Dateien im Projektordner sind Vorbereitung.
- **JWT** blockiert externe Clients komplett — `static_api_key` ist der einzige Workaround.
- **/v1/-Endpoints** MÜSSEN auth-frei bleiben (Dictate-Tastatur kann keine Custom-Headers) — Bypass via `_JWT_EXCLUDED_PREFIXES`.
- **Chart.js / zoom-plugin / hammer.js** via CDN — bei Air-Gap ist das Metriken-Dashboard tot.
- **Prosodie-Audio-Bytes** dürfen NICHT in `interactions`-Tabelle landen (Worker-Protection P191).

## Sync-Pflicht (Patch 164+)

`scripts/sync_repos.ps1` muss nach jedem `git push` laufen. Coda pusht zuverlässig nach Zerberus, vergisst aber den Sync regelmäßig (Ratatoskr + Claude driften). Mit v2a Paket 2 ist das in `scripts/session_end.ps1` gebündelt. Falls der Sync scheitert: explizit melden („⚠️ sync_repos.ps1 nicht ausgeführt"), nicht stillschweigend überspringen.

## Langfrist-Vision

- **Phase 5 — Nala-Projekte:** Zerberus = persönliche Code-Werkstatt, Nala vermittelt zwischen Chris und LLMs/Sandboxes.
- **Metric Engine** = kognitives Tagebuch + Frühwarnsystem für Denkmuster-Drift.
- **Rosa Corporate Security Layer** = letzter Baustein vor kommerziellem Einsatz.
- **Telegram-Bot** als Zero-Friction-Frontend für Dritte (keine Tailscale-Installation nötig).

## Supervisor-Verhalten — Bug-Sammelstelle (P219-pre)

Wenn Chris Bugs diktiert ohne explizit „fixen"/„los"/„pack zusammen": still sammeln, **nicht** sofort patchen. Am Ende jeder Antwort die offene Sammlung als nummerierte Liste anzeigen (visueller Druck). Wenn leer: nichts anzeigen. Bei ≥3 Bugs gelegentlich an Sammel-Patch-Trigger erinnern. Trigger-Phrasen für Coda-Auftrag: „los", „pack zusammen", „mach einen Patch draus", „Coda kann jetzt", „Sammel-Patch".

## Don'ts für Supervisor

- **PROJEKTDOKUMENTATION.md NICHT vollständig laden** (5000+ Zeilen) — nur gezielt nach Patch-Nummern grep'en.
- **Memory-Edits** max 500 Zeichen pro Eintrag.
- **Session-ID ≠ User-Trennung** — Metriken pro User erst nach DB-Architektur-Fix vertrauenswürdig.
- **Patch-Prompts IMMER als `.md`-Datei** — NIE inline im Chat (Patch 101).
- **Dateinamen `CLAUDE_ZERBERUS.md` und `SUPERVISOR_ZERBERUS.md` sind FINAL** — alte Namen (`CLAUDE.md`, `HYPERVISOR.md`) nicht mehr referenzieren.
- **Lokale Pfade:** Zerberus `C:\Users\chris\Python\Zerberus\`, Ratatoskr `C:\Users\chris\Python\Rosa\Nala_Rosa\Ratatoskr\`, Claude `C:\Users\chris\Python\Claude\` — Patch-Prompts mit falschen Pfaden immer verifizieren.
- **Bug-Sammelstelle NICHT als Erstreaktion in einen Patch verwandeln** (P219-pre) — siehe Sektion oben.

## Lessons-Quellen (Coda vs Supervisor)

- **Coda** liest `lessons_ZERBERUS.md` (Pipe-only, ~Top-3 via `scripts/lessons_lookup.py`) und `lessons_ZERBERUS_KONTEXT.md` nur on-demand.
- **Supervisor** (Chat-Instanz) liest `GLOBAL_LESSONS.md` aus dem Claude-KB-Gist + `lessons_ZERBERUS.md` aus dem Zerberus-Gist.
- **Archiv** (passiv): `LESSONS_KONSOLIDIERT.md` im Claude-Repo.
