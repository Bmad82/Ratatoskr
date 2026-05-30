# SUPERVISOR_ZERBERUS.md – Zerberus Pro 4.0

<!-- STAND-ANKER:START -->
**Stand:** FR 2026-05-30 RESTART-Hotfix (uvicorn stale → Kill+Restart+Auto-Hook) - Phase 5b - 2026-05-30 - ~4810 Tests
**Commits:** siehe HANDOVER_ZERBERUS.md für aktuelle Hashes
_Single Source of Truth: STAND.json - propagiert von scripts/propagate_stand.py._
<!-- STAND-ANKER:END -->

*Strategischer Stand für die Supervisor-Instanz (claude.ai Chat). Historische Patch-Einträge: [`SUPERVISOR_ZERBERUS_ARCHIV_2026-05-21.md`](SUPERVISOR_ZERBERUS_ARCHIV_2026-05-21.md) + [`docs/PROJEKTDOKUMENTATION.md`](docs/PROJEKTDOKUMENTATION.md). Diese Datei trägt nur den heißen Stand.*

---

## Aktueller Patch

**FR 2026-05-30 CodeCat-Inventur (Bestandsaufnahme, 2026-05-30)** — Reine Inventur-Session, KEIN Code-Patch, KEIN Fix, KEINE Architektur-Aenderung. Acht Blöcke A–H als Vorbereitung für die kommenden Folge-Feature-Requests (OpenRouter-Kosten, Projekt-Sync-Sichtbarkeit, Modell-Picker, Token-Anzeige, Hel-Kontostand-Fix). **Ergebnis-Datei: [`docs/CODECAT_INVENTUR.md`](docs/CODECAT_INVENTUR.md)** (Bibel-Format, pipe-separated, alle Befunde mit Datei:Zeile-Pointern). **Wichtigste Befunde:** (1) **HTTP-401 in CodeCat-Sidebar** ist eine JWT-Exclusion-Lücke für `/archive/` — die Liste `_JWT_EXCLUDED_PREFIXES` in `middleware.py:95-109` enthält `/code-cat` (PATH-03) und `/v1/` (Dictate-Bypass), aber NICHT `/archive/`. Frontend ruft `fetch('/archive/sessions?limit=30')` ohne JWT → 401. NICHT GEFIXT, Fix-Optionen in `CODECAT_INVENTUR.md` Block A-3. (2) **Hel-Kontostand-Bug Wurzel-Grund identifiziert:** `/api/v1/credits` wird mit `OPENROUTER_API_KEY` (Inference-Key) gerufen, braucht aber `OPENROUTER_MANAGEMENT_KEY` (separater Provisioning-Key). `hel.py:431-432` schluckt die Exception silent (`except Exception: pass`), Kontostand bleibt `None`, UI zeigt nichts. NICHT GEFIXT, Fix-Skizze in `CODECAT_INVENTUR.md` Block D-2. (3) **`20250514`-Deadline-Check sauber:** 0 Treffer in aktivem Code (`.py`/`.json`/`.yaml`/`.yml`/`.js`) in Zerberus UND Claude-Repo. Alle Treffer in Doku/Planern/Lessons. KEINE Exposure vor 15. Juni 2026. (4) **`orchestrator.py` ist NICHT tot** (Memory-Drift, früherer Eintrag muss aktualisiert werden); `legacy.py` und `orchestrator.py` sind beide live mit eigenen Aufgaben (`main.py:532,534,542`). (5) **Modell-Label "DEEPSEEK/V3.2" in CodeCat ist Hardcoded-Default** (`code_cat.html:670`), wird nur aktualisiert wenn ein Chat-Response zurückkommt; real verwendetes Modell kommt aus `config.yaml.legacy.models.cloud_model` mit möglichem P-UI-11-Override; Verdacht "Label lügt, Coda läuft auf Opus" ist FALSCH — Opus 4.7 ist Coda-Domain (Mjölnir-CLI), CodeCat-Orchestrate-Pfad sperrt Anthropic-Modelle explizit (FR 2026-05-23 orchestrate-mvp Schritt 5, Anthropic-Billing-Split 15.06.2026). (6) **CTX 0%** ist statischer Platzhalter (`code_cat.html:710`, kein `updateCtx()`-Handler). Token-/$-Anzeige im Footer akkumuliert clientseitig aus dem `data.usage`-Feld der Chat-Response — Werte gehen beim Reload verloren, kein Read aus `costs`-Tabelle. (7) **Nala und CodeCat teilen die Projekt-Datenschicht** (`projects_repo.py` Single Source of Truth, gemeinsame `projects`-/`project_files`-Tabellen, gemeinsamer `data/projects/`-Filesystem-Pfad); CodeCat hat aber heute keine eigene Upload-UI — Uploads laufen über Hel `POST /admin/projects/{id}/files`. (8) **Agent-FS-Zugriff** existiert als Sandbox-Workspace-Materialisierung (`projects_workspace.py:338-407`, Docker-Volume-Mount `-v workspace:/workspace[:ro|:rw]`, Patch 171/203c/207); das ist das "kopier-und-fummel-an-der-Kopie"-Prinzip in Zerberus, NICHT identisch mit Coda's git-worktree-Pattern. **Code-Veto-Layer aktiv** (`code_veto.py:114-140`, Patch 209 Sancho Panza). **HITL-Permission-Gate aktiv** nur für `channel="telegram"` (`orchestrator.py:161 _HITL_PROTECTED_CHANNELS`). (9) **Reasoning-Mapping (P-UI-11)** ist Auto-Heuristik aus User-Message, NICHT von Mjölnir steuerbar — keine Anbindung in `hel_settings.json`, keine UI in CodeCat. Folge-FR-Bedarf falls Mjölnir-Slider greifen soll. **KEIN Server-Restart** (kein Code geändert). **Folge-FR-Reihenfolge** (Empfehlung in `CODECAT_INVENTUR.md` Block I-1): (a) /archive/-JWT-Fix, (b) OPENROUTER_MANAGEMENT_KEY, (c) CodeCat-Kosten aus `costs`-Tabelle, (d) generation_id-Logging + Alembic-Migration, (e) CodeCat-Modell-Picker + CTX-Berechnung, (f) Mjölnir → Reasoning-Slider-Durchreichung.

## Vorheriger Patch

**FR 2026-05-30 RESTART-Hotfix** — uvicorn-Prozess (Port 5000) war stale, alle PATH-01..05-Fixes lagen brach im Speicher. Hartes Kill+Restart, neues `scripts/restart_live.ps1`, Auto-Restart-Hook in `deploy_to_live.ps1` (triggert bei Code-Änderungen unter `zerberus/**`, `requirements.txt`, `start*.bat`). 7 Smoke-Tests (4 server-side OK, 3 UI-only für Chris). Live-Server läuft jetzt mit aktuellem HEAD. `/code-cat` HTTP 200, Chat-Slugs live, Log-Rotation aktiv.

## Vorvorheriger Patch (Tageskontext 2026-05-30)

**Fünf Sessions am 30. Mai:** (1) Rosa-Readiness Sessions 9-13 KOMPLETT — alle 6 Blöcke, 4779→4810 Tests. (2) Slug-Rot-Hotfix — mistral 3.1→3.2, qwen2.5-vl→qwen3-vl, 404 jetzt retryable mit Startup-Slug-WARN. (3) Marathon-Sammlung — Guard-Registry, DB-Crypto, UI-Polish 6 Items. (4) DEPLOY-Marathon — Auto-Sync-Mechanismus (`deploy_to_live.ps1`). (5) PATH-Hotfix — Live-Server-Pfad korrigiert: `Rosa\Nala_Rosa\` → `C:\Users\chris\Python\Zerberus`.

## Rosa-Readiness (FR 2026-05-25, Multi-Session)

16 Marathon-Items in 13 Coda-Sessions. **Block 0 Bugs (3/3):** B-075 Dictate-Sentiment-Skip, B-076 Code-Cat-Drift, B-077 TLS-Cert-Watchdog. **Block 1 Rosa (5/5):** R-00 System-Tab Cache-Control-Hardening (7-Session Bughunt), R-01 Tab-ID Rename, R-02 Body-Move `_chat_completions_body` → `chat_pipeline.py` (legacy.py 2026→211 LOC), R-03 Sub-Endpoint-Refactor 14→5 Router, R-04 Hel-Toggles, R-05 Executor-Sandbox. **Block 3 Stabilität (5/5):** E-04 VRAM Auto-Unload, E-05 Reranker Hard-Limit, E-06 HitL transaktional (DB-Write FIRST), E-07 Config Split-Brain Test, E-08 `/v1/` Auth-Bypass Regression-Test. **Block 4 Architektur (2/2):** E-09 Pipeline-Exception-Hierarchie, E-10 GLOSSARY.md. **Offen:** E-01 Guard-Prompts externalisieren, E-02 `/v1/` Auth-Härtung (braucht Dictate-Kompat-Studie), E-03 Hel Basic-Auth → Session/JWT.

## CodeCat + Orchestrate (FR 2026-05-23 → 2026-05-24)

**orchestrate-mvp** — ORCHESTRATE als dritter Pipeline-Pfad. Pre-call Detector + HuginnIntent.ORCHESTRATE, OrchestratorService (Plan+Synthese), HitL-Gate (Per-Step-Opt-out), Worker-Dispatcher (sequentielle Kontext-Kette, SSE, Guard-DI). Alle LLM-Calls über OpenRouter/DeepSeek V3.2, KEIN Claude-Call (Billing-Split-Vorbereitung). 87 Tests.

**code-cat-mvp** — Desktop-Frontend "Code Cat" für ORCHESTRATE-Backend. Single-File Vanilla HTML/CSS/JS (`code_cat.html` ~750 LOC), eigener Router ohne `/v1`-Prefix. Three-Column-Layout, Plan-Karte mit Step-Cards/Approve/Reject, SSE via `/nala/events`, Log-Panel, Session-Sidebar, Kostenfooter. 48 Tests.

## Phase 5 — Status

Phase 5a (P194-P203) + Phase 5c (UI-Redesign P-UI-1..11) **ABGESCHLOSSEN**. Phase 5b läuft — Warm-up erledigt (B-073 WhatsApp-Removal, B-074 Lessons-Cron, DECISIONS, Whisper-INT8-Werkzeug). Power-Features-Roadmap in [`MARATHON_WORKFLOW_ZERBERUS.md`](MARATHON_WORKFLOW_ZERBERUS.md).

## Architektur-Referenz

**Tech-Stack:** DeepSeek V3.2 (OpenRouter) | Mistral Small 3 (Guard) | Gemma 4 E2B (Prosodie, ~3.4 GB) | faster-whisper large-v3 FP16 (Docker Port 8002) | cross-en-de-roberta (Embeddings DE) | bge-reranker-v2-m3 | SQLite WAL + Alembic | Nala (Mobile-first) + CodeCat (Desktop-Coding) + Hel (Admin) | Huginn-Telegram (Long-Polling, Tailscale). **VRAM:** ~10.3 GB / 12 GB (RTX 3060). **Repos:** `C:\Users\chris\Python\{Zerberus, Rosa\Nala_Rosa\Ratatoskr, Claude}` — sync via `scripts/sync_repos.ps1`.

**Importvertrag (R-02):** Neuer Code → `zerberus.core.chat_pipeline`, NICHT `legacy`. Orchestrator-Imports am Datei-ENDE (Zirkular). Monkeypatches auf `chat_pipeline`, nicht `legacy`.

**Layer-Policy (E-06/E-09):** `hitl_ownership` CLOSED. DB-Write FIRST + Cache NUR nach Commit. Pipeline-Exceptions tragen Layer-Tag, Policy via `MODULE_POLICY`-Registry.

## Offene Items

→ [`BACKLOG_ZERBERUS.md`](BACKLOG_ZERBERUS.md). Strukturelle Schulden: Persona-Hierarchie (B-071), `interactions` ohne User-Spalte, Voice-Messages Telegram (B-072).

## Architektur-Warnungen

- **Rosa Security Layer** NICHT implementiert — Dateien sind Vorbereitung.
- **JWT** blockiert externe Clients — `static_api_key` einziger Workaround.
- **/v1/-Endpoints** MÜSSEN auth-frei bleiben (Dictate-Tastatur) — `_JWT_EXCLUDED_PREFIXES`.
- **Chart.js/zoom/hammer** via CDN — Air-Gap = Metriken-Dashboard tot.
- **Prosodie-Audio-Bytes** NICHT in `interactions` (Worker-Protection P191).

## Sync-Pflicht

`scripts/sync_repos.ps1` nach jedem Push. Gebündelt in `scripts/session_end.ps1`. Falls Sync scheitert: explizit melden, nicht stillschweigend überspringen.

## Langfrist-Vision

Zerberus = persönliche Code-Werkstatt | Metric Engine = kognitives Tagebuch + Frühwarnsystem | Rosa = letzter Baustein vor Kommerz | Telegram = Zero-Friction-Frontend für Dritte.

## Supervisor-Verhalten — Bug-Sammelstelle (P219-pre)

Chris diktiert Bugs ohne "fixen"/"los" → still sammeln. Am Antwort-Ende offene Sammlung als nummerierte Liste (visueller Druck). Bei ≥3 an Sammel-Patch erinnern. Trigger: „los", „pack zusammen", „mach einen Patch draus", „Coda kann jetzt".

## Don'ts für Supervisor

- PROJEKTDOKUMENTATION.md NICHT voll laden (5000+ Zeilen) — gezielt greppen.
- Memory-Edits max 500 Zeichen.
- Patch-Prompts IMMER als `.md`-Datei, NIE inline.
- Dateinamen `CLAUDE_ZERBERUS.md` / `SUPERVISOR_ZERBERUS.md` sind FINAL.
- Bug-Sammelstelle NICHT als Erstreaktion patchen.

## Lessons-Quellen

- **Coda:** `lessons_ZERBERUS.md` (Top-3 via `scripts/lessons_lookup.py`), `lessons_ZERBERUS_KONTEXT.md` on-demand.
- **Supervisor:** `GLOBAL_LESSONS.md` (Claude-KB-Gist) + `lessons_ZERBERUS.md` (Zerberus-Gist).
- **Archiv:** `LESSONS_KONSOLIDIERT.md` (Claude-Repo).

---

# 📦 SUPERVISOR-MEMORY-OFFLOAD

> **Dieser Block ersetzt Supervisor-Memory-Slots.** Supervisor merkt sich nur EINEN Zeiger: "Lies den Memory-Offload-Block." Wenn Memory voll → neue Erkenntnisse hier anhängen, Slot freigeben.

## Architektur-Gesamtbild

Zerberus = EIN System | drei Oberflächen auf EINER Projekt-Datenschicht:
- **Nala** | Chat-Frontend | modell-abhängig | aktiver Pfad: `legacy.py` (NICHT `orchestrator.py`)
- **CodeCat** | Coding-Oberfläche | gleiche Projekte + Docker/Festplatten-Zugriff | kein eigenes Repo | soll Chris' eigenes "Claude Code" werden
- **Hel** | HITL-Admin-Dashboard | Steuereinheit

Roadmap: erst CodeCat fertig → dann Mjölnir umbauen (Handy-Knöpfe → CodeCat-Endpoints statt Claude-Code-CLI) = To-Go-Fernbedienung | Konsequenz: CodeCat-Features mit sauberer API bauen (Mjölnir-tauglich)

**Mjölnir** | separates Handy-Bedienpult | FastAPI Port 8855 | Vanilla HTML/JS/CSS | tmux-Session-Manager | Tailscale ohne Auth | Industrial-Cockpit-Design | triggert Claude Code per CLI | FEATURE_REQUEST rein → mjolnir.md zurück

## Coda-Settings

Opus 4.8 | Max Reasoning | 1M Kontext | Permission Level 4 (Accept Edits/bypass) | Self-Stop ~450k Token (nur Stop + Doku) | Start: "Go." | Plan-Modus nur für unsichere/große erste Patches | Coda bekommt Ziele, entscheidet Implementierung selbst

## OBERSTES GEBOT

Chris terminalisiert NICHTS was Coda kann | NIEMALS Terminal-Befehle an Chris | Supervisor baut IMMER Coda-Prompt (.md) | Coda merged Branches SELBST + pusht SELBST vor Session-Ende | Verstoß = sofortige Korrektur

## Feature-Request-Disziplin

Atomaritäts-Override bei Teilschritten | "Alle Schritte = logische ARBEITSPAKETE, EINE Session, ein Commit pro Schritt, EIN Sammel-HANDOVER" | Ohne Override: ~100k Token Verschwendung pro Mikro-Session
Bug-Sammelstelle | Chris diktiert ohne "fixen" → still sammeln | alle paar Prompts erinnern | vor Kontextfenster-Ende erinnern
FR immer als .md-Datei | jedes Deliverable separate Datei | .md braucht kein Skill-Template

## Whisper/Diktat

~75% der Eingaben per Whisper | Android Pixel 10 Pro | bei phonetisch seltsamen Wörtern IMMER nachfragen
Artefakte: "Stadt"=start.bat | "Serberos"=Zerberus | "Fastabi"=FastAPI | "Süßkittel"=Sysctl | "Cloud/Wolke"=Claude

## Repos + Sync

1. **Zerberus** | `C:\Users\chris\Python\Zerberus` | github.com/Bmad82/Zerberus | ALTER PFAD `...\Rosa\Nala_Rosa\Zerberus` VERALTET
2. **Ratatoskr** | Docs-Spiegel | `C:\Users\chris\Python\Rosa\Nala_Rosa\Ratatoskr` | public | Bmad82/Ratatoskr
3. **Claude** | globale Templates+Lessons | `C:\Users\chris\Python\Claude` | public | Bmad82/Claude
Sync nach jedem Patch: Zerberus→Ratatoskr + universelle Lessons→Claude

## Supervisor-Onboarding

1. Index-Gist: https://gist.github.com/Bmad82/6fb4ba84419edc0e2d8290cacef1faeb (html_extraction_method: markdown)
2. SUPERVISOR_ZERBERUS.md aus Ratatoskr per web_fetch
3. PROJEKTDOKUMENTATION.md NIE voll laden (1900+ Zeilen)
4. Memory-Edits max 500 Zeichen

## Hardware + Mobile

Ryzen 7 2700X | RTX 3060 12GB | 32GB DDR4 | Windows | volle Pipeline ~10.3GB VRAM | Guard/LLM via OpenRouter (nie lokal) | blockiert: Chutes, Targon | Tailscale (Jojo iPhone, Chris Android)
Mobile-First Pflicht | :active statt :hover | Desktop zweitrangig | Chris Viewport: CSS ~427×850px | Safe-Area top ~48-56px / bottom ~16-20px

## OpenRouter-Kosten (für Block-C-Feature)

Kontostand: `GET /api/v1/credits` → total_credits − total_usage | **braucht Management-Key** (NICHT Inference-Key — wahrscheinlicher Grund warum Hel-Kontostand nie ging)
Session-Kosten: `usage`-Feld jeder non-streaming-Completion + `id` für `GET /api/v1/generation`
Chris will: OR-Kontostand + Session-Kosten + Reset-Button (Tageskilometer) + Diagramm (Supervisor vs. Subagents vs. kumuliert) | Anthropic-Coda-Kosten egal "außer easy"

## Deadline 15. Juni 2026

Anthropic zieht `claude-sonnet-4-20250514` + `claude-opus-4-20250514` zurück | API-Calls schlagen hart fehl | kein Failover | Agent-SDK aliased nicht automatisch | Nachfolger: claude-sonnet-4-6 / claude-opus-4-7 (neuestes: Opus 4.8)
Billing-Split am selben Tag: programmatische Claude-Code-Nutzung → separater Credit-Topf zu API-Raten | Interaktiv (Terminal, Web, Cowork) bleibt auf Subscription

## Personen

Jojo = Juana Schreen (NICHT Joana) | Chris' Partnerin | "Sonnenblume" | primäre Nala-Nutzerin (iPhone/Tailscale) | Katze Nala = Projektname
Rosa = Security-Layer-Codename | KEINE Person
