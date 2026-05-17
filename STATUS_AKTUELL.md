# STATUS_AKTUELL.md — Zerberus Pro 4.0
*Erstellt: 2026-05-17 von Coda (B-status-snapshot). Quellen: HANDOVER_ZERBERUS.md, SUPERVISOR_ZERBERUS.md, BACKLOG_ZERBERUS.md, HANDOVER_PHASE_5.md, lessons_ZERBERUS.md, docs/PROJEKTDOKUMENTATION.md (letzter Block), git log.*

---

## 1. Patch- und Branch-Stand

- **Letzter Zerberus-Commit:** `20a5727` — "HANDOVER: Commit-Hash + Sync-Status nach B-mjolnir-multisession nachgetragen" (2026-05-17)
- **Letzter Code-Patch:** P217 (Backend-Bugfix, DualEmbedder-Slot-Routing, DELETE-Pfad Reindex-atomar, `for_write=True` Resolver). Davor für UI: P-UI-11 (Reasoning-Mapping). Für Huginn: B-072 (Voice→Whisper-Pipeline, 2026-05-16).
- **Letzter doku-only Patch:** B-mjolnir-multisession (Multi-Session-Status + QUEUED-Pattern, 2026-05-17)
- **Aktuelle Phase:** Phase 5a ✅ vollständig abgeschlossen (P194–P203), Phase 5c ✅ vollständig abgeschlossen (P-UI-1..UI-11). Phase 5b noch nicht gestartet.
- **Branch-Status:** alles auf main. Keine offenen Feature-Branches.
- **Letzter Ratatoskr-Sync:** 2026-05-17 — Commit `48f9815` ("Sync: B-mjolnir-multisession: STATUS-Header + QUEUED-Pattern + dreiphasiger Selbsttest")

---

## 2. Was in Phase 5a tatsächlich gelandet ist

### LLM-Routing
- **Chat-LLM:** DeepSeek V3.2 via OpenRouter
- **Guard-LLM:** Mistral Small 3 (`mistralai/mistral-small-24b-instruct-2501`, OpenRouter) — seit P120/P180
- **Prosody-LLM:** Gemma 4 E2B (lokal, llama-mtmd-cli, Q4_K_M ~3.4 GB) — opt-in
- **Llama Guard 3 8B:** unklar (Quelle: HANDOVER_PHASE_5.md und SUPERVISOR_ZERBERUS.md nennen ausschließlich Mistral Small 3 als Guard — kein Llama-Guard-Eintrag in gelesenen Files)

### Guard-Layer
- Mistral Small 3 aktiv als Response-Guard
- `caller_context` (Persona, seit P180) + `rag_context` (RAG-Chunks, seit P180) werden übergeben — verhindert Halluzinations-Flag auf korrekten RAG-Antworten
- Fail-open Default (Guard-Timeout → Request wird nicht geblockt)
- **Veto-Pfad geprüft:** unklar (Quelle: kein expliziter Veto-Pfad-Test-Report in gelesenen Files)

### Prosody
- Gemma 4 E2B: lokal, Q4_K_M, llama-mtmd-cli, Modelle unter `C:\Users\chris\models\gemma4-e2b\`
- Opt-in via Consent-UI (P191)
- Worker-Protection vorhanden (P191) — Worker-Schutz gegen parallele Audio-Anfragen
- Whisper + Gemma parallel via `asyncio.gather` (P190)

### Embedding-Stack
- **Bi-Encoder DE:** `T-Systems-onsite/cross-en-de-roberta-sentence-transformer` (GPU)
- **Bi-Encoder EN:** `intfloat/multilingual-e5-large` (CPU, optionaler Index)
- **Cross-Encoder Reranker:** `BAAI/bge-reranker-v2-m3` (seit P89)
- DualEmbedder mit Sprach-Routing (DE-Slot / EN-Slot getrennt, seit P217)

### Secrets-Filter
- P212 — Longest-First-Masking vorhanden. Audit-Trail: unklar (Quelle: P212 wird in BACKLOG_ZERBERUS.md unter B-016-Description erwähnt, aber kein expliziter Audit-Trail-Beleg in gelesenen Files)

### Sentiment-Triptychon
- **BERT:** `oliverguhr/german-sentiment-bert` — aktiv (lokal)
- **Prosodie:** Gemma 4 E2B — aktiv (opt-in via Consent-UI)
- **Konsens:** Triptychon-Integration in P192 gelandet
- **Incongruence-Indicator:** unklar (Quelle: P192-Details nicht vollständig in gelesenen Files — Symptom-Indikator in UI nicht bestätigt)

### RAG-Pipeline
- FAISS over-fetch → min-chunk 120w (P88) → cross-encoder rerank (P89) → top_k
- Sprach-getrennte FAISS-Indizes: DE-Slot + EN-Slot (P217)
- **Aktuelle top_k-Werte:** unklar (Quelle: kein aktueller Config-Dump in gelesenen Files)
- Soft-Delete mit physischem Reindex (P217) — DELETE löscht Chunks aus FAISS nach Delete-Operation

### UI
- **Nala:** Mobile-first, HSL-Slider, TTS + Auto-TTS (P186), Katzenpfoten, Feuerwerk, Sentiment-Triptychon (P192). Eingabefeld-Collapse CSS-only (P-UI-3). Sidebar-Projektliste (P-UI-Polish-2). Reasoning-Block (P-UI-11). Alle Phase-5c-Items ✅.
- **Hel:** Admin-Dashboard, Splitscreen gefixt (P-UI-Hel-Split, von Chris visuell verifiziert 2026-05-10). Tab-Nav, Pacemaker, Metriken (Charts mit Chart.js seit P91). Scheduler-Tab. RAG-Tab.
- **Mobile-tauglich:** Nala ✅ (mobile-first), Hel bedingt (primär Desktop-Dashboard; Touch-Targets 44px überall in Nala per shared-design.css)

### Self-Knowledge-RAG (Huginn)
- `docs/huginn_kennt_zerberus.md` + Spiegel `docs/RAG Testdokumente/huginn_kennt_zerberus.md` als RAG-Material
- `tools/sync_huginn_rag.py` als Sync-Tool (Stand-Anker-Parser + Huginn-Doc-Update-Tool)
- Stand-Anker-Trim auf kompakte Bullet-Liste (P-debt-11) — FAQ-Block in ersten 4000 Zeichen
- **Auto-Update am Session-Ende:** unklar (Quelle: kein "Cron läuft stabil" Beleg in gelesenen Files — sync ist ein manuelles Tool, kein automatischer Cron)

### Easter Egg Weiher 1967/68
- ✅ EINGEBAUT (B-070, 2026-05-16)
- Ort: `zerberus/main.py`, Funktion `create_app()`, zwischen `app.mount("/static"...)` und `@app.get("/")`
- Kommentar: `// make love ... not war — W.F. Weiher, Stanford AI Lab, 1967`
- Trigger: kein funktionaler Trigger — rein versteckter Kommentar, null Impact

---

## 3. Test-Stand

- **Unit-Tests gesamt:** unklar (Quelle: kein aktueller Gesamtlauf in gelesenen Files. Letzte bestätigte Zahl: 2784 passed bei P-UI-3 / 2026-05-07. Seither Increments durch P183 +49, P-debt-11..14, P217, B-072 +16, P-umzug. Schätzung: ~2900+, aber nicht verifiziert)
- **Letzter Lauf mit Gesamtzahl:** P-UI-3 (2026-05-07) → 2784 passed
- **Loki-Suite:** unklar (Quelle: keine Loki-Gesamtzahl in gelesenen Files)
- **Fenrir-Suite:** unklar — `test_fenrir_mega_patch.py` existiert (446+ Zeilen). Einzelne Tests bekannt: `test_f_tts_02_button_not_duplicate` (noch offen, Schulden #4), `test_f_pace_01_master_toggle_stable` (gefixt P-debt-14), `test_f_pace_02_page_stays_responsive` (grün). Gesamtzahl unklar.
- **Vidar-Suite:** unklar (Quelle: nicht explizit in gelesenen Files mit Zahl)
- **Telegram/Huginn/Whisper-Suite:** 50/50 grün (Stand B-072, 2026-05-16)
- **Bekannt flaky/gespringt:**
  - `test_rag_dual_switch::test_fallback_logic` — SentenceTransformer-Mock-Issue (pre-existing seit P193)
  - `test_tts_integration::test_leerer_text_raises_value_error` — edge-tts-Modul nicht installiert (pre-existing)
  - `test_patch185_runtime_info` (7 Tests) — Worktree-Setup-Drift (`config.yaml` fehlt im Worktree, Schulden #6) — im Hauptrepo grün
  - `test_f_tts_02_button_not_duplicate` — TTS-Race-Condition, Schulden #4, mittel-bis-groß
- **Letzte Suite-Laufzeit:** unklar (kein Laufzeit-Log in gelesenen Files)

---

## 4. VRAM-Tetris (RTX 3060 12 GB)

**Komponenten im Modus "Nala aktiv mit Prosodie":**

| Komponente | VRAM |
|-----------|------|
| Whisper large-v3 (FP16, Docker) | ~4.5 GB |
| BERT Sentiment | ~0.5 GB |
| Gemma 4 E2B (Q4_K_M) | ~3.0 GB |
| DualEmbedder (DE GPU) | ~0.5 GB |
| Reranker bge-reranker-v2-m3 | ~1.0 GB |
| Windows-Reserve | ~0.8 GB |
| **Gesamt** | **~10.3 GB / 12 GB** |

- **Reserve im Idle:** ~1.7 GB (Gemma E2B nicht geladen wenn Prosodie opt-out)
- **Reserve mit Prosodie aktiv:** ~1.7 GB (Gemma in obiger Tabelle eingerechnet)
- **Bottleneck:** Whisper (4.5 GB) ist der größte Einzelkonsument
- **Kandidaten für Raus/OpenRouter-Migration:**
  - EN-Embedder (`multilingual-e5-large`) läuft auf CPU — kein VRAM-Problem, aber langsam
  - B-040 (Upgrade Bi-Encoder auf `BAAI/bge-m3`) wäre VRAM-Erhöhung → PRIO RUNTER (Reranker kompensiert)
  - Gemma E2B ist opt-in — bei Prosodie-Deaktivierung ~1.7 GB Reserve zurück
  - Kein Migrationsdruck aktuell — 1.7 GB Reserve ist knapp aber ausreichend

---

## 5. Phase 5b — was als nächstes geplant

**WICHTIG für den Supervisor:** Die in der Feature-Request-Anfrage genannten Items (Projekt-DB, Workspace-Routing, Docker-Sandbox, Sancho-Panza-Veto, HitL, Snapshot/Rollback) sind **ALLE ERLEDIGT** — das war Phase 5a (P194–P203). Der aktuelle Stand ist nach Phase 5a + 5c.

**Unmittelbar nächster Schritt (eine der beiden Optionen, Coda wählt nach Token-Budget):**
1. **Schulden #4: TTS-Race-Condition** — `test_f_tts_02_button_not_duplicate` als P-debt-15 oder Pipeline-Audit P-tts-1. Mittlerer-bis-großer Aufwand. Test in `zerberus/tests/test_fenrir_mega_patch.py:446-481` — prüft kein TTS-Button-Duplikat nach mehrfachem Chat-Senden.
2. **PROJEKTDOKUMENTATION.md-Catchup** — 6 Patches fehlen (P-umzug/B-061, B-072, B-mjolnir-fix, B-global-lessons, B-mjolnir-fix-2, B-mjolnir-multisession). Coda-isolierbar, klar definiert, ~1-2 h pro Session.

**Phase 5b Power-Features (aus HANDOVER_PHASE_5.md, Reihenfolge noch nicht final entschieden):**
- Multi-LLM Evaluation (Bench-Mode pro Projekt-Aufgabe — DeepSeek vs. Sonnet vs. lokal)
- Bugfix-Workflow + Test-Agenten Per-Projekt (Loki/Fenrir/Vidar als Templates)
- Multi-Agent Orchestrierung, Debugging-Konsole
- Reasoning-Modi, LLM-Wahl per Aufgabe, Cost-Transparency in Hel
- Agent-Observability (Chain-of-Thought Inspector mit redacted Sensitive-Patterns)

**Abhängigkeiten:**
- Docker-Sandbox (P171) ✅, HitL (P167) ✅, Pipeline-Cutover (P177) ✅ — alles bereits gebaut
- Schulden #4 (TTS) ist Voraussetzung für sauberen Fenrir-Test-Run, aber kein Blocker für Phase 5b

**Supervisor-Klärungsbedarf vor Phase 5b:** Reihenfolge der 5b-Items (kein expliziter Priorisierungs-Dok in gelesenen Files). `NALA_PROJEKTE_PRIORISIERUNG.md` und `NALA_PROJEKTE_FEATURE_SPEC.md` existieren im Zerberus-Root — nicht in diesem Sync enthalten, aber beim Supervisor abrufbar.

---

## 6. Offene Bugs / Dauerbrenner

**Aktive Schulden (nach B-mjolnir-multisession):**

| # | Item | Aufwand | Regression-Test |
|---|------|---------|----------------|
| 4 | TTS-Race-Condition `test_f_tts_02_button_not_duplicate` | mittel-groß | test existiert, schlägt an |
| 5 | Settings-Cog-Mobile-UX-Frage | braucht User-Input | nein |
| 6 | Worktree-Setup-Drift (config.yaml, faiss fehlt in Worktree) | niedrig | n/a (Hauptrepo grün) |
| 7 | EN-Side-Add-Pfad differenzieren (Hel-UI-Wahl DE/EN/auto) | mittel | nein |
| 13 | Claude-Repo-Drift (2 uncommitted + 1 unpushed) | klein | nein |
| 15 | Embedded-Worktree-Stub im Claude-Repo (`git rm --cached`) | klein | nein |

**Dauerbrenner aus bekannter Bug-Liste (Stand bei Lesen):**
- `/v1/`-Auth-Bypass: **unklar** (Quelle: nicht in HANDOVER, BACKLOG oder SUPERVISOR erwähnt)
- Config-Split-Brain Hel↔LLMService: **unklar** (Quelle: nicht explizit in gelesenen Files als aktiver Bug gelistet)
- legacy.py vs. orchestrator.py Pipeline-Pfad: BACKLOG B-012 related. Feature-Flag `modules.pipeline.use_message_bus` bereit (P177), aber Cut-Over-Status unklar (Quelle: kein "Cut-Over auf neue Pipeline" in HANDOVER_ZERBERUS.md bestätigt)
- Server-Reload-Zombies: `start.bat` hat netstat+taskkill-Schleife (laut BACKLOG L-178f ERLEDIGT). `start_stable.bat` hat Reload+Force-Shutdown (P-start-stable). Kein aktiver Bug-Report in aktuellen Files.

**Weitere offene BACKLOG-Items:** B-010 (FAISS-Migration --execute ausstehend), B-011 (Prosodie-Pipeline), B-013 (Background Memory Extraction Phase 4), B-020 (Kostendelta), B-021 (BERT-Sentiment-Sprünge), B-023 (RAG 500er beim ersten Upload nach Clear), B-025 (manuell getippter Text → DB-Speicherung verifizieren), B-035 (Metriken-Glättungs-Toggle), B-050 (torch-CUDA-Check), B-060 (Whisper Sentence-Repetition).

---

## 7. Erledigt seit letztem Supervisor-Update

*Letztes Supervisor-Update: P-mjolnir-workflow (2026-05-15). Seither folgende Patches:*

| Item | Status |
|------|--------|
| B-061 / P-umzug (Zerberus nach `C:\Users\chris\Python\Zerberus`, relative Pfade Audit, sync_repos.ps1 portabel) | ✅ ERLEDIGT (2026-05-16) |
| B-072 (Huginn Voice→Whisper-Pipeline in Telegram) | ✅ ERLEDIGT (2026-05-16) |
| B-mjolnir-fix (mjolnir.md Round-Trip Pflicht im Workflow verankert) | ✅ ERLEDIGT (2026-05-16) |
| B-global-lessons (4 Marathon-Workflow-Lessons ins Claude-Repo) | ✅ ERLEDIGT (2026-05-16) |
| B-mjolnir-fix-2 (Round-Trip-Pflicht verschärft, gleichrangig mit HANDOVER+Push) | ✅ ERLEDIGT (2026-05-17) |
| B-mjolnir-multisession (STATUS-Header + QUEUED-Pattern + dreiphasiger Selbsttest) | ✅ ERLEDIGT (2026-05-17) |
| **WhatsApp-Modul gelöscht** | **unklar** (Quelle: kein Eintrag in HANDOVER/BACKLOG/SUPERVISOR über WhatsApp-Deletion) |
| **Lessons-Cron eingerichtet** | **unklar** (Quelle: kein Cron-Setup-Eintrag in gelesenen Files) |
| **Self-Knowledge-RAG bestätigt funktionsfähig** | ✅ ja — tools/sync_huginn_rag.py + huginn_kennt_zerberus.md aktiv. Auto-Cron: **unklar** |
| **Weiher-Easter-Egg eingebaut** | ✅ BESTÄTIGT (B-070, zerberus/main.py:create_app()) |

---

## 8. Inkonsistenzen

*Beim Lesen aufgefallen — keine Korrekturen, nur Beobachtungen:*

1. **DECISIONS.md fehlt.** FEATURE_REQUEST_ZERBERUS.md (der Auftrag für diesen Status-Snapshot) nennt DECISIONS.md als zu lesende Datei. Es gibt keine solche Datei im Zerberus-Root und kein DECISIONS*.md in docs/ (Glob-Suche ohne Treffer). Nicht heimlich ersetzt — vermutlich nicht existente oder bereits in HANDOVER/BACKLOG integrierte Datei.

2. **SUPERVISOR_ZERBERUS.md und PROJEKTDOKUMENTATION.md 6 Patches veraltet.** Stand: P-mjolnir-workflow (2026-05-15). Nicht enthalten: B-061, B-072, B-mjolnir-fix, B-global-lessons, B-mjolnir-fix-2, B-mjolnir-multisession. HANDOVER_ZERBERUS.md und lessons_ZERBERUS.md sind aktuell.

3. **BACKLOG_ZERBERUS.md Header veraltet.** Kopfzeile sagt "Stand: Patch 182 (2026-04-30)". Tatsächlicher Code-Stand: P217+. Inhalt (Statusmarkierungen der Items) scheint aktuell, aber der Header-Stempel nicht.

4. **Llama Guard 3 8B unklar.** FEATURE_REQUEST_ZERBERUS.md fragt nach "Llama Guard 3 8B + Mistral Small 3" als Guard-Layer. HANDOVER_PHASE_5.md und SUPERVISOR_ZERBERUS.md nennen ausschließlich Mistral Small 3 als Guard. Kein Llama-Guard-Eintrag in keiner gelesenen Datei — entweder nie eingebaut, oder die Doku hat ihn nicht erwähnt.

5. **Phase-5b-Begrifflichkeit im FEATURE_REQUEST.** Section 5 des Feature-Requests nennt "Projekt-DB, Workspace-Routing, Docker-Sandbox, Sancho-Panza-Veto, HitL, Snapshot/Rollback" als Phase-5b-Items. Das sind alle Phase-5a-Items (P194–P203), komplett erledigt. Der Begriff "Phase 5b" im Feature-Request meint vermutlich die nächste noch nicht gestartete Planungsphase aus Supervisor-Sicht — intern heißt die Phase aber "5b: Power-Features" (Multi-LLM, Multi-Agent etc.). Keine Handlungskonsequenz, nur Begriffsklärung für den Supervisor.

6. **Ratatoskr sync_repos.ps1 Ziel-Name für lessons.md.** sync_repos.ps1 kopiert `lessons_ZERBERUS.md` → Ratatoskr als `lessons_ZERBERUS.md`. Aber Ratatoskr hat auch noch eine ältere `lessons.md` — von vor der Datei-Konvention-Umbennennung (2026-05-15). Die beiden Dateien koexistieren in Ratatoskr ohne Kommentar.

---

## 9. Doku-Sync-Status

| Datei | Status |
|-------|--------|
| `SUPERVISOR_ZERBERUS.md` (Zerberus + Ratatoskr) | **VERALTET** — endet bei P-mjolnir-workflow/2026-05-15. 6 Patches fehlen. |
| `docs/PROJEKTDOKUMENTATION.md` (Zerberus) + `PROJEKTDOKUMENTATION.md` (Ratatoskr) | **VERALTET** — endet bei P-mjolnir-workflow (Zeile 9601). Letzte 6 Patches fehlen. |
| `lessons_ZERBERUS.md` (Zerberus + Ratatoskr) | ✅ **AKTUELL** — neueste Sektion B-mjolnir-multisession 2026-05-17 |
| `HANDOVER_ZERBERUS.md` (Zerberus; Ratatoskr hat `HANDOVER.md`) | ✅ **AKTUELL** — letzter Stand B-mjolnir-multisession 2026-05-17 |
| `huginn_kennt_zerberus.md` in Zerberus/docs | **VERALTET** — Stand-Anker zeigt P-mjolnir-workflow; letzte B-patches nicht drin |
| `BACKLOG_ZERBERUS.md` | **TEILWEISE VERALTET** — Items aktuell, Header-Stempel nicht (P182 statt P217+) |

**sync_repos.ps1 letzter erfolgreicher Lauf:** 2026-05-17, nach B-mjolnir-multisession.
- Zerberus: `20a5727` → Ratatoskr: `48f9815` → Claude-Repo: `08c7e9e`
- `verify_sync.ps1`: alle 3 Repos working-tree clean, 0 unpushed ✅ (Stand nach B-mjolnir-multisession)

**Empfehlung:** Vor der nächsten Supervisor-Session ist ein PROJEKTDOKUMENTATION.md-Catchup (SUPERVISOR_ZERBERUS.md + PROJEKTDOKU + huginn_kennt_zerberus.md) empfehlenswert. Die 6 fehlenden Patches (B-061/P-umzug, B-072, B-mjolnir-fix, B-global-lessons, B-mjolnir-fix-2, B-mjolnir-multisession) sind alle doku-isolierbar. Schätzung: ~1-2 Sessions. Danach ist der Supervisor wieder synchron mit dem tatsächlichen Stand.

---
*Ende STATUS_AKTUELL.md — Zerberus Pro 4.0, Stand 2026-05-17, Coda B-status-snapshot*
