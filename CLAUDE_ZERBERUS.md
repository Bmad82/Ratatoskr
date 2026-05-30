# CLAUDE_ZERBERUS.md — Zerberus Pro 4.0 (Kern-Bibel)

> **mw-v2b Paket 2 (2026-05-21):** Diese Datei enthaelt nur noch den Kern (OBERSTES GEBOT + Faulheits-Catches + Workflow). Task-spezifische Regeln in [`playbooks/`](playbooks/), pfadspezifische Regeln in [`docs/claude_rules/`](docs/claude_rules/) (opt-in via Variante A/B). Patch-Historie P155-P217 per `python scripts/lessons_lookup.py --task '<Stichwort>'`.

## Regel 0 — OBERSTES GEBOT: Chris terminalisiert NICHTS was Coda kann

- **Coda fuehrt jede Operation SELBST aus die ueber das Terminal moeglich ist.** NIEMALS `git`/`pytest`/`pip`/`robocopy`/`venv`/`spacy`/`python`/`gh`-Befehle an Chris delegieren. Coda hat Shell + Bash + PowerShell.
- **Coda merged Branches SELBST auf main und pusht SELBST vor Session-Ende.** `git merge --ff-only` + `git push origin main` gehoeren in die Coda-Verantwortung, nicht in eine Mjoelnir-Liste.
- **`mjolnir.md` enthaelt NUR was physisch unmoeglich ist fuer Coda:** echtes Geraet, Touch-UX-Gefuehl, Browser-Login am realen Endpunkt, Tailscale vom Mobilgeraet, Whisper-Spracheingabe am Mikrofon, Docker-Desktop-UI-Klicks, Hardware. KEIN `git`, KEIN `pytest`, KEIN `pip install`.
- **Supervisor (Chat) gibt Chris ebenfalls KEINE Terminal-Befehle, sondern baut Coda-Prompts.**
- **Verstoss = Session-Abbruch + Korrektur.** Wenn Coda merkt dass er gerade Terminal-Arbeit an Chris delegiert hat → STOPP, zurueck-rollen, selbst machen.
- **Warum:** Delegation an Chris bricht den Kontrakt und macht Coda zu einem teuren Ratgeber statt einem Mitarbeiter.

## Autonome Prioritaetsliste — Coda fragt nicht „wie weiter?"

Wenn alle Phasen-Ziele ✅ und kein expliziter naechster Schritt vom User: 1) Doku-Catchup wenn ≥3 Patches ohne SUPERVISOR/PROJEKTDOKU/huginn-Update → nachholen + Huginn-RAG-Sync | 2) Offene Schulden aus HANDOVER (kleinsten zuerst) | 3) WORKFLOW.md pruefen ob naechste Phase-Ziele definiert | 4) Sonst in HANDOVER „Wartet auf Phase-Spec" + STOPP.

NIEMALS „wie moechtest du weitermachen?" fragen, wenn aus HANDOVER+WORKFLOW ableitbar. Ausnahme nur bei echtem Architektur-Risiko (Scope-Sprengung, irreversible Aenderung).

## Faulheits-Catches (Pflicht — alle 6)

### 1. Destruktive Operationen — Pflicht-Stopp
Vor `rm -rf` | `DROP TABLE` | `faiss.write_index` | `volume rm` | `git push --force` | `git reset --hard` (main) | `branch -D` | Skripte mit `destroy`/`nuke`/`wipe`/`purge`/`overwrite`: STOPP, beschreiben was passiert, „Bestaetigung?" abwarten. Detail + Backup-Muster: [`playbooks/database.md`](playbooks/database.md), pfad-getriggerte Regel: [`docs/claude_rules/destructive_ops.md`](docs/claude_rules/destructive_ops.md).

### 2. Deliverable-Smoke (Pflicht)
Kein Deliverable ohne funktionalen Smoke. HTML/JSON/Script → Datei oeffnen/parsen/Ergebnis pruefen VOR Patch-Abschluss. „Syntax valide" ≠ Smoke. Fehlende Smoke = Patch NICHT abgeschlossen. Anlass: Feature-Audit-HTML (Post-P-debt-12) hatte valides JS aber kaputtes inline-JSON — User sah leere Seite.

### 3. Loki / Fenrir / Vidar — Pflicht-Lauf
Bei JEDEM Patch der UI / Auth / Chat-Pipeline / RAG / Guard / Huginn / Endpoint anfasst: Server via `start_stable.bat`, dann Vidar → Loki → Fenrir mit `-m e2e`. Failures = Blocker. Detail: [`playbooks/testing.md`](playbooks/testing.md).

### 4. Auto-Test-Policy & Integration-Test-Pflicht
Alles was Coda testen kann → Coda testet. Vor jedem „Chris muss noch testen"-Vermerk gilt: kann ich das selbst testen? JA → Integration-Test schreiben, NICHT eskalieren. Detail: [`playbooks/testing.md`](playbooks/testing.md).

### 5. Pflicht nach jedem Patch
SUPERVISOR_ZERBERUS.md aktualisieren (Nr | Datum | 3-5 Zeilen). Offene Items pflegen. PROJEKTDOKUMENTATION.md anhaengen (Patch-Nr+Titel | Datum | Was | Dateien | Teststand). [`docs/huginn_kennt_zerberus.md`](docs/huginn_kennt_zerberus.md) bei neuen Features/Endpoints/Architektur aktualisieren — **KEIN automatischer RAG-Upload mehr** (FR 2026-05-22): Datei bleibt im Repo, Chris laedt sie bei Bedarf manuell ueber Hel hoch. Detail: [`playbooks/rag_pipeline.md`](playbooks/rag_pipeline.md).

### 6. Repo-Sync + Live-Deploy (Pflicht-Letztschritt)
Nach `git push`: `sync_repos.ps1` DANN `scripts/verify_sync.ps1` DANN `scripts/verify_live_deploy.ps1` — alle drei Pflicht. `sync_repos.ps1` ruft intern `scripts/deploy_to_live.ps1` auf, der den Live-Server-Pfad (`C:\Users\chris\Python\Zerberus` — identisch mit dem Arbeits-Pfad, ein einziger Klon seit B-061/2026-05-16) per `git pull --ff-only` synchronisiert UND bei requirements.txt-Diff `pip install` in das Live-`venv` laeuft. `verify_live_deploy.ps1` ist der finale Beweis: Live-HEAD == Remote-HEAD. Bei ❌ Exit 1 von einem der drei: NICHT weitermachen, Sync-Problem loesen. Patch gilt erst bei ✅ Exit 0 von ALLEN drei als abgeschlossen. **FR 2026-05-30 PATH-01:** B-076-Default-Pfad und DEPLOY-01-Korrektur zeigten beide auf falsche Pfade (Arbeits-Pfad zu eng, dann Rosa-Klon der gar nicht live war). Beweis: Traceback aus Live-Log `File "C:\Users\chris\Python\Zerberus\zerberus\core\llm.py", line 91`. Default ist jetzt der echte Live-Pfad; Worktree-Pushes (`PWD = .claude/worktrees/<id>`) triggern Pull regulaer, Self-Skip greift nur bei Aufruf aus dem Live-Pfad selbst (dann ist nichts zu tun). Fallback: `scripts/install_auto_deploy_task.ps1` (Scheduled Task alle 10 min). **Chris terminalisiert NICHTS — keinen `git pull`, keinen `pip install`, NICHTS.** Detail: [`playbooks/observability.md`](playbooks/observability.md).

## Marathon-Workflow (Phase 5+)

- **Session-Start-Pflicht** (mw-v2a 2026-05-21): FEATURE_REQUEST_ZERBERUS.md → mjolnir.md → HANDOVER_ZERBERUS.md → MARATHON_WORKFLOW_ZERBERUS.md → `python scripts/lessons_lookup.py --task '<Aufgabe>'` (NICHT lessons_ZERBERUS.md komplett laden — TF-IDF Top-3; bei 0 Treffern: Aufgabe ist neu). Dann loslegen.
- Ziele statt Rezepte | WAS nicht WIE | eigene Architektur-Entscheidungen erwuenscht.
- Stopp bei ~400k Token oder Kontextvergiftung | aktuellen Patch sauber fertig → Doku → Handover → STOPP.
- Blockiert → Frage in `DECISIONS_PENDING.md` parken → naechsten unabhaengigen Patch nehmen.
- **Worktree-Branch IMMER nach main mergen vor Session-Ende** | sonst laeuft Server auf altem Code (P217-Anlass). Konkret: `cd C:\Users\chris\Python\Zerberus && git merge <branch> --ff-only && git push origin main`. Wenn ausnahmsweise NICHT: im HANDOVER „Branch noch nicht gemergt — Patch NICHT aktiv".
- **Session-Start: NICHT fragen „was soll ich machen"** (P219-pre) — die HANDOVER-Empfehlung am Ende des „Naechster Schritt"-Abschnitts ist der Default. Coda laedt HANDOVER, liest Empfehlung, legt los. Nur wenn Chris in der Eroeffnungs-Message explizit eine andere Aufgabe reingibt, davon abweichen.
- **HANDOVER „Naechster Schritt" — GENAU EIN Default als Feststellung**, KEINE Optionsliste. „Naechster Schritt ist X (Begruendung)." — Punkt. Alternativen die Chris entscheiden muss → DECISIONS_PENDING (nur bei echtem Architektur-Risiko). Umkehr-Exklusiv: Coda entscheidet IMMER selbst, AUSSER User hat explizite Prioritaet gesetzt.

## Session-Auffuell-Regel (2026-05-21)

Primaerer Auftrag erledigt UND < 300k Token verbraucht → weiterarbeiten, nicht abschliessen. Auffuell-Reihenfolge: (1) FEATURE_REQUEST Restpunkte (2) MARATHON_WORKFLOW offene Items (3) BACKLOG (4) Test-Schulden (5) Doku-Hygiene. Stopp bei ~350k (50k Reserve fuer Doku). Zwischen-Patches: eigener Commit, ABER kein separater HANDOVER — ein HANDOVER am Session-Ende fuer alle. AUSNAHME: destruktive/riskante Patches NIE als Auffueller. Anti-Pattern: „Patch fertig bei 120k → Doku → STOPP" = 80% Overhead.

## Coda-Autonomie (P176)

Coda uebernimmt ALLES was er kann: `docker pull`, `pip install`, `curl`, Testdaten, Sync. Chris nur fuer physisch Unmoegliches. Vor Patch-Abschluss: Server startet sauber? Images da? Dependencies aktuell? Nicht hoffen — verifizieren. Neue Dependencies: `pip install` im venv (nicht `--break-system-packages` global). Docker-Images: `docker pull`. Healthchecks ausfuehren, Probleme fixen. Erst wenn unfixbar (Auth/Hardware/UX) → an Chris eskalieren.

## Token-Effizienz

- Datei bereits im Kontext → NICHT nochmal lesen | nur lesen wenn (a) nicht sichtbar ODER (b) direkt vor Write
- Doku-Updates am Patch-Ende | ein Read→Write-Zyklus pro Datei
- CLAUDE_ZERBERUS.md + lessons_ZERBERUS.md: Bibel-Fibel-Format (Pipes | Stichpunkte | ArtikelWeg)
- SUPERVISOR/PROJEKTDOKU/README/Patch-Prompts: Prosa (menschliche Leser)
- Neue Eintraege IMMER im komprimierten Format

## Globale Wissensbasis

- Repo: https://github.com/Bmad82/Claude (PUBLIC | keine Secrets/Keys/IPs/interne URLs)
- `lessons/` nur bei Bedarf pruefen | nicht rituell bei jedem Patch
- Nach Abschluss: universelle Erkenntnisse dort eintragen

## Projektpfad & Server

```
C:\Users\chris\Python\Zerberus
```
(Umzug B-061 / 2026-05-16. Ratatoskr + Claude-Repo bleiben am alten Ort; `sync_repos.ps1` + `verify_sync.ps1` sind portabel.)

```bash
cd C:\Users\chris\Python\Zerberus
venv\Scripts\activate
uvicorn zerberus.main:app --host 0.0.0.0 --port 5000 --reload
```

Fuer Test-Sweeps: `start_stable.bat` (kein Reload, Pflicht fuer Playwright).

## Regeln (Basics)

1. Erst lesen, dann schreiben | keine blinden Ueberschreibungen
2. `bunker_memory.db` niemals loeschen/aendern (Detail: [`playbooks/database.md`](playbooks/database.md))
3. `.env` niemals leaken/loggen
4. `config.yaml` = Single Source of Truth | `config.json` NICHT als Konfig-Quelle
5. Module mit `enabled: false` nicht anfassen
6. Dateinamen `CLAUDE_ZERBERUS.md`/`SUPERVISOR_ZERBERUS.md` FINAL | nicht umbenennen | gilt auch fuer Ratatoskr-Kopien
7. Mobile-first + Touch-Target ≥44px + `keydown` statt `keypress` + `:active` statt nur `:hover` → Detail in [`docs/claude_rules/frontend_mobile.md`](docs/claude_rules/frontend_mobile.md)
8. `/v1/`-Endpoints auth-frei (Dictate-Pipeline) → Detail in [`playbooks/auth_security.md`](playbooks/auth_security.md)
9. **User-Entscheidungen als klickbare Box** — Aktionen mit User-Input (Datei loeschen, Index leeren, Settings) → IMMER klickbare Entscheidungsbox in Nala-UI. Buttons + klare Optionen + „Soll ich das fuer dich uebernehmen?". Kein Freitext-Dialog bei binaerer/ternaerer Entscheidung.

## Playbook-Index

| Playbook | Wann lesen |
|---|---|
| [`testing.md`](playbooks/testing.md) | UI/Auth/Pipeline/RAG/Guard/Endpoint-Patch — Loki/Fenrir/Vidar + Auto-Test-Policy + Integration-Tests |
| [`rag_pipeline.md`](playbooks/rag_pipeline.md) | `/hel/admin/rag/*` | FAISS-Index | Embeddings | Huginn-Sync |
| [`auth_security.md`](playbooks/auth_security.md) | `middleware.py` | `/v1/*` | JWT | HTTP-Header | Dictate-App |
| [`database.md`](playbooks/database.md) | `bunker_memory.db` | Alembic | FAISS-Backup | destruktive DB-Ops |
| [`observability.md`](playbooks/observability.md) | Logging | Watchdog | Pacemaker | Sync-Tools | Doku-Konsistenz |

## Weiterfuehrende Doku

- Projektspezifische Lessons: `lessons_ZERBERUS.md` (Pipe-only) + `lessons_ZERBERUS_KONTEXT.md` (Prosa)
- Globale Lessons: https://github.com/Bmad82/Claude/tree/main/lessons
- Patch-Archiv: `docs/PROJEKTDOKUMENTATION.md` (nicht vollstaendig laden, gezielt grep'en)
- Supervisor-Patch-Prompts: IMMER als `.md`-Datei vom Supervisor, nie inline Chat-Text

## Don'ts (Erinnerung)

- PROJEKTDOKUMENTATION.md NICHT vollstaendig laden (5000+ Zeilen)
- Alte Namen `CLAUDE.md` / `HYPERVISOR.md` nicht mehr referenzieren — final sind `CLAUDE_ZERBERUS.md` / `SUPERVISOR_ZERBERUS.md`
- Patch-Prompts mit falschen lokalen Pfaden immer verifizieren (Zerberus, Ratatoskr, Claude — Pfade siehe oben)
- `--break-system-packages` NIE — immer im venv installieren
