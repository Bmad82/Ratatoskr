# lessons_ZERBERUS.md – Zerberus Pro 4.0 (projektspezifisch)
*Gelernte Lektionen. Nach jeder Korrektur ergänzen.*

Universelle Erkenntnisse: https://github.com/Bmad82/Claude/lessons/

---

## Tuple-Return statt Dict-Augmentation erzwingt Caller-Disziplin (Whisper-Voice-Path DEGRADE, FR 2026-05-24 Pipeline-Bereinigung Session 4)

`whisper_client.transcribe()` lieferte `Dict[str, Any]` mit `text`-Key, warf bei Fehler|Caller fingen NUR `WhisperSilenceGuard` und liefen `result.get("text", "")` — leeres Transcript war ambivalent (User schwieg vs Whisper-Timeout)|Reflex-Fix: `result["error_type"] = "timeout"` augmentieren — non-breaking, jeder Caller kann optional auf das neue Feld branchen|Falle: genau diese Optionalitaet ist der Bug — heute IGNORIEREN die Caller den Error-Type, naive Augmentation aendert das nicht. Wer den Field nicht liest, schweigt weiter|Loesung: neuer Wrapper `transcribe_voice() -> Tuple[str, Optional[str]]` als DEGRADE-Layer. Tuple-Unpacking (`text, err = await transcribe_voice(...)`) ist syntaktisch ZWINGEND — wer den Error-Type weglaesst, bekommt `UnpackingError`, kein silent fail|Innerer `transcribe()` bleibt fuer Caller die echten Exceptions brauchen (Tests, Future-Pfad ohne DEGRADE-Semantik)|Whitelist-Konstante `WHISPER_ERROR_TYPES = ("silence", "timeout", "http_error", "transport_error", "unknown")` haelt Drift sichtbar — wer einen neuen String einfuehrt ohne die Konstante zu aendern, schlaegt der `TestWhisperErrorTypesWhitelist` rot|Regel: bei API-Aenderungen die Caller-Behavior aendern sollen, ist Dict-Augmentation eine Falle — Tuple/Dataclass erzwingt die Caller-Migration, sonst aendert sich nichts

---

## Fire-and-Forget braucht Outer-Timeout + Done-Callback, sonst crasht es still (HitL-Deferred-Send, FR 2026-05-24 Pipeline-Bereinigung Session 4)

`asyncio.create_task(deferred_coroutine())` ohne Task-Reference und ohne `add_done_callback` ist ein Silent-Failure-Pfad|Wenn der Task crasht, erscheint nur `Task exception was never retrieved` als RuntimeWarning beim Garbage-Collect — in der Logflut nicht sichtbar|Im konkreten Fall (`_deferred_file_send_after_hitl`): User klickt ✅, aber `send_document` haengt 30 min in einem TCP-Timeout. User sieht nichts, der Bot ist scheinbar tot|Direkter `await` ist keine Loesung — der `long_polling_loop` blockiert dann sequenziell und die Click-Antwort die das Gate aufloest, wuerde nie verarbeitet (Deadlock)|Loesung: `asyncio.create_task(asyncio.wait_for(deferred(...), timeout=N))` als Outer-Hard-Timeout (N = innerer Erwartungs-Timeout + Send-Margin), plus `add_done_callback(callback)` der Exception explizit branched|Callback muss selbst `try/except` haben (sonst crasht der Callback und die Exception verschwindet erneut) — `task.exception()` kann selbst `CancelledError` werfen|TimeoutError ist branchen-wuerdig: dem User einen Fallback-Hinweis schicken ("⚠️ ... abgebrochen"), nicht still sterben|Regel: jedes `asyncio.create_task` ohne `await` braucht (a) sinnvollen `timeout`-Wrap und (b) `add_done_callback` mit Exception-Inspect — sonst ist es Fire-and-Forget-and-Forget

---

## Passive Registry zuerst, Active Decorator-Pattern spaeter (Error-Policy, FR 2026-05-24 Pipeline-Bereinigung Session 4)

FR-Spec verlangt "Error-Policy-Registry mit FailMode-Enum"|Naive Lesart: `@enforce_fail_mode(FailMode.OPEN)` als Decorator-Wrapper der Try/Except automatisch um die Funktion legt — Verhalten wird zentral durchgesetzt|Problem: Decorator-Stacking bricht Test-Mocks (`patch.object(module, "func", AsyncMock(...))` umgeht den Decorator), Async-vs-Sync-Inferenz wird brittle, Decorator-Reihenfolge wird zur Schulden-Klasse|Loesung: passive Registry als Schritt 1. `MODULE_POLICY: Mapping[str, FailMode]` dokumentiert Soll-Verhalten, `get_fail_mode(name)` liefert Lookup, jeder Modul-Author implementiert seine Policy selbst und kann via `assert get_fail_mode("sentiment") is FailMode.OPEN` self-auditen|Vorteil: Bestandsverhalten bleibt unangetastet, Aenderung ist 100 Zeilen statt 1000, Tests pruefen Mapping statt verschachtelte Decorator-Stack-Reihenfolge|Active-Pattern kann in Session 7 oder spaeter erfolgen, dann mit klarerem Refactor-Scope und Audit der existierenden Try/Except-Pfade|`get_fail_mode(unknown_module)` wirft `KeyError` statt Default zurueckzugeben — wer ein unbekanntes Modul liest hat sich vermutlich verschrieben, kein silent Pass|Regel: bei Registry-Pattern-Einfuehrung in einer bestehenden Codebase ist passive Lookup-Layer der konservative Schritt 1 — sichtbare Durchsetzung (Decorator/Wrapper) erst wenn das Mapping geuebt + stabil ist

---

## Lazy-Load + Lifecycle-Watchdog statt Eager-Load entfernen (FR 2026-05-24 Pipeline-Bereinigung Session 3)

BERT-Sentiment lud bei Modul-Import eager auf CUDA — 1.4 GB VRAM reserviert beim Server-Start, auch wenn Sentiment in der Session nicht aufgerufen wird|Reflex: Eager-Load entfernen, jeder Call laedt selbst, Modell-Lifecycle dem Garbage-Collector ueberlassen|Problem: Re-Load-Latenz pro Call (~3 s fuer german-sentiment-bert) ist zu teuer fuer einen interaktiven Pfad|Loesung: Drei-Schritt-Pattern — (a) `_ensure_loaded()` mit `threading.Lock` als idempotenter Lazy-Loader, (b) `last_used_ts` Timestamp im Modul-Global beim Call setzen, (c) externer Watchdog (`zerberus/core/lifecycle_watchdog.py`) prueft alle 30 s ob `(now - last_used_ts) >= idle_seconds` und ruft dann `unload()` + `torch.cuda.empty_cache()`|Pre-Warm-Schutz: `last_used_ts=None` (geladen aber nie benutzt) zaehlt NICHT als idle → kein Unload, weil das ein bewusster Warm-Up-Zustand sein kann|Registry-Pattern statt Klassen-Vererbung: `@dataclass LifecycleTarget(name, is_loaded, last_used_ts, unload)` als Funktions-Pointer-Bundle — passt zu den existierenden Modulen die Modul-globalen State halten und keine Klassen-Instanzen haben|Anti-Brittle: pure function `_should_unload(target, now, idle_seconds)` separat von der async Loop testbar; Exception-isolated pro Target (ein fehlgeschlagener Unload stoppt nicht den Loop)|Regel: Bei teuren-Reload-Modellen niemals Eager-Load-rip-out — Lazy-Load + externer Watchdog gibt aktiven Sessions volle Geschwindigkeit, idlen Maschinen das VRAM zurueck

---

## Python-Plumbing VOR Infrastruktur (Whisper Dual-Compute-Type, FR 2026-05-24 Pipeline-Bereinigung Session 3)

FR-Spec verlangt "Whisper Dual-Model (Nala int8 / Dictate float16)"|Echte Dual-Variante braucht entweder zwei Container oder einen Server, der `compute_type` aus dem Multipart-Form-Feld pro Request honoriert — heute ist es ein Single-Slot-`faster-whisper-server`|Reflex: erst Container umbauen, dann Code anpassen|Problem: das schiebt das Code-Delivery hinter die Infrastruktur-Arbeit, und die Container-Aenderung ist nicht trocken testbar wenn der Code noch gar keine Felder durchreicht|Loesung umgekehrt: Code-Plumbing JETZT, beide Pfade defaulten auf `whisper-1`, Server ignoriert `compute_type` harmlos wenn unbekannt — sobald der Container Dual-Variante kann, schalten die Pfade ueber Config ohne zweiten Coding-Roundtrip|Source-Audit-Tests pinnen die Verkabelung („Nala-Pfad nutzt `compute_type_nala`, Dictate-Pfad nutzt `compute_type_dictate`, NIE umgekehrt") — wenn jemand spaeter aus Versehen die Felder vertauscht, kracht der Test|`compute_type=None`-Default ist die Backward-Compat-Spur: Telegram-Pfad (out of FR scope) bleibt unangetastet, sendet kein Feld, Server-Verhalten identisch|Regel: Code-Pfade duerfen vor der Infrastruktur fertig sein — Source-Audit-Tests verhindern dass spaetere Refactors die Verkabelung silently brechen, und Default-Werte halten den heutigen Zustand stabil

---

## Legacy-Config-Datei-Migration: Rename-only, kein Merge (FR 2026-05-24 Pipeline-Bereinigung Session 2)

`config.yaml` war seit Patch 105/112 SSoT, aber `config.json` lag noch im Repo-Root und wurde nur per Warn-Log registriert — Split-Brain-Falle, wenn jemand die JSON manuell editiert hat|Naive FR-Lesart "Migration-Pfad ... beim Start einmalig nach config.yaml migrieren" koennte als 3-Wege-Merge missverstanden werden|Tatsaechlich enthaelt eine alte `config.json` per Definition VERALTETE Werte (Hel-Schreibungen aus der Pre-P105-Aera) — Merge wuerde den Split-Brain reanimieren (alte JSON-Werte koennten frische YAML-Werte ueberschreiben)|Loesung: idempotenter Rename `config.json` → `config.json.deprecated` mit drei expliziten Returns (`"absent"` no-op, `"renamed"` erfolgreich, `"conflict"` beide vorhanden → WARN + manueller Eingriff, KEIN Overwrite)|Hook in `invariants.check_config_consistency()` weil Boot-Time-Selfchecks dort ohnehin versammelt sind|Test-Backstop: Source-Audit ueber das gesamte aktive Paket nach `"config.json"`-String-Literalen mit gestrippten Docstrings/Kommentaren — Tests + Migrations-Modul ausgenommen, sonst null Treffer|Regel: bei Legacy-File-Migration immer Rename-only mit `.deprecated`-Suffix, nie Auto-Merge; die kanonische Datei darf nie zugunsten der veralteten ueberschrieben werden

---

## Cache-Invalidate-Decorator-Konsistenz schlaegt manuelle reload-Calls (Hel post_config, FR Session 2 2026-05-24)

Patch 156 fuehrte `@invalidates_settings` als Decorator-Konvention fuer alle Hel-YAML-Writer ein|Ausnahme blieb `post_config` mit manuellem `if changed: reload_settings()`-Aufruf — funktional korrekt, aber inkonsistent mit den 5 anderen Writern (`post_huginn_config`, `post_vision_config`, `post_pacemaker_*`, `post_provider_blacklist`)|Risiko: wenn jemand spaeter den Endpoint refactort und den Call vergisst, schweigender Cache-Drift (Hel-UI zeigt alten Wert nach Save)|Fix: Decorator vorhaengen, `reload_settings()`-Call entfernen — Verhalten identisch (beide Pfade laden den Singleton nach jedem Write neu), aber jetzt eine Konvention statt zwei|Test-Backstop: `re.search(r"@invalidates_settings[^\n]*\n(?:[^\n]*\n)?async def post_config\b", text)` pinnt den Decorator UND ein zweiter Test verbietet `reload_settings()` im Funktionskoerper (sonst doppelt geladen)|Regel: bei Decorator-basierten Konventionen sind Ausnahmen Schulden — beim ersten Anlass alle Stellen auf die Konvention bringen, sonst wandern Bug-Fix-Patterns zwischen "manuell" und "decorator" hin und her

---

## Per-Request-Disk-Read fuer Templates kostet nichts und liefert Live-Edit (Hel-Template-Extraktion, FR 2026-05-24 Pipeline-Bereinigung Session 1)

Modul-Konstante `ADMIN_HTML = open(...).read()` faengt das Template einmalig beim Import|Frontend-Edit wird erst sichtbar nach Python-Reload (in Produktion: Server-Restart, in Dev: Watcher-Bounce)|Per-Request-Read im Endpoint-Body kostet einen 50-KB-File-I/O — auf SSD < 1 ms, kein messbarer Latenz-Hit|Live-Edit-UX wiegt das auf: Markup-Patch im Editor speichern + Browser-F5 = sofort sichtbar, kein Restart|Trick: die Modul-Konstante als Backward-Compat-Symbol behalten (alte Source-Audit-Tests asserten weiter auf `ADMIN_HTML`), der Endpoint liest aber von Disk|Regel: bei statischen HTML-Templates ohne Variable-Injection (alle Daten kommen via XHR/SSE post-Render) ist per-Request-Read der Standard, nicht die Optimierung

---

## data-action-Pattern statt eval/new-Function fuer Inline-Handler-Migration (Hel DOM-Cleanup, 2026-05-24)

94 Inline-onclick/onchange/oninput zu konvertieren ist mechanische Arbeit — der Reflex ist `eval(handlerString)` oder `new Function(handlerString)` als Mini-Dispatcher|Beides ist CSP-aequivalent zu inline-handlers (`unsafe-eval` statt `unsafe-inline`), bringt also nichts|Sauber: `data-{event}-action="funcName"` + `data-{event}-args='[...]'` als JSON, plus drei Top-Level-Listener (`click`/`change`/`input`) mit `event.target.closest('[data-' + evt + '-action]')` als Dispatcher|`this.value`-Sentinel als `__VALUE__`-String in den Args, beim Dispatch durch `el.value` ersetzt — die haeufigste Inline-Spezial-Form (Slider-Display-Update)|Komplexe Inline-Expressions (`if(x) x.method()`, `window.open(...)`) zu **Named-Helpers** promoten — kein Versuch das im Generator zu parsen|Regel: Wenn eine Inline-Migration "nur noch eval"-rufgegen schreit, eine Stufe zurueck und das data-action-Schema bauen — kostet eine Stunde, spart Wartung + CSP-Tightening-Pfad

---

## Multi-Session-Marathon braucht STATUS-Header-Disziplin im mjolnir.md (FR Pipeline-Bereinigung Session 1, 2026-05-24)

Multi-Session-FRs (8 Sessions, eine pro Coda-Aufruf) brauchen klare Marker fuer den Workflow|`STATUS: FERTIG` in mjolnir.md = die naechste Session loescht es und faengt komplett neu an|`STATUS: IN_ARBEIT` = der Multi-Session-Auftrag laeuft noch, naechste Session fortsetzen|Falle: vergisst man IN_ARBEIT zu setzen, sieht die naechste Coda-Session „FERTIG" und ignoriert den noch offenen FR — Marathon bricht ab|Doppel-Belt: zusaetzlich im mjolnir.md eine fettgesetzte WARNUNG an Chris („KEINEN neuen FEATURE_REQUEST schicken"), weil der Supervisor sonst parallele FRs reinkippen koennte und der Multi-Session-Workflow kollidiert|Regel: bei Multi-Session-FR IMMER vier Marker setzen — (a) `STATUS: IN_ARBEIT`, (b) `FORTSCHRITT: X von Y Sessions`, (c) `NAECHSTE SESSION:` mit konkretem Session-Namen, (d) WARNUNG-Block an Chris

---

## Claude Design ist kein Dialog-Partner — Briefing als vollständiger Block (Code Cat Wireframes, 2026-05-24)

Anlass: Code Cat Wireframes, 24.05.2026|Claude Design stellt Rückfragen, wartet aber nur ~3 Sekunden auf Antwort und macht dann selbstständig weiter|Dialog-Workflow fuehrt zu ungewollten Eigeninterpretationen|Loesung: Vollstaendiges Briefing als zwei Dateien — Struktur-JSON (Layout-Skelett) + DESIGN_KINTSUGI.md (Aesthetik)|Keine Luecken lassen, Rueckfragen sofort als Block beantworten|Regel: Bei Claude Design immer alles vorher fertig haben, kein iterativer Dialog

---

## Mobile und Desktop sind getrennte Welten — kein Responsive-Kompromiss (Code Cat vs. Nala, 2026-05-24)

Versuchung: ein „responsive" Frontend bauen das beides kann|Realitaet: Mobile (Single-Column, Touch, 44px Targets, :active) und Desktop (Three-Column, Hover, Keyboard, Drag-Resize, Min-Width 1024px) folgen unvereinbaren Interaktions-Patterns|Loesung: getrennte Frontends — Nala (Mobile-only) und Code Cat (Desktop-only)|Gleiche Design-Tokens (Kintsugi), verschiedene Layout-Patterns|Regel: Neue Frontends: zuerst entscheiden ob Mobile oder Desktop, nie beides in einer Datei

---

## Guard-Wiring war DI-Slot ohne Caller — orchestrate-mvp Schritt 0 (FR code-cat-mvp, 2026-05-24)

`dispatch_worker_chain()` hatte schon einen `guard_callable=None`-Slot|Aber legacy.py rief ohne Argument auf — kein Guard pro Worker-Output|Folge: 87 Tests gruen, ORCHESTRATE-Pfad sauber, aber Halluzinations-Schutz nur theoretisch verfuegbar|Fix: 5-Zeilen-Wrapper in legacy.py der `hallucination_guard.check_response` auf die `async (user_msg, answer) -> dict | None` Signatur des Workers abbildet|Lesson: DI-Slots ohne Default-Caller sind versteckte Schulden — beim Schreiben des Slots gleich den ersten Default-Caller mit verdrahten, sonst sammelt sich Tech-Debt im naechsten FR

---

## Session-Auffüll-Regel — Bootstrap-Overhead nicht pro Mini-Patch zahlen (2026-05-21, Kintsugi-Migration Token-Audit)

Sessions schlossen bei ~120k ab obwohl 300k+ Budget frei war|3 Sessions à 120k statt 1 à 360k = 200k verschwendet|Mega-Patch-Ära (P122–P152) bewies: 24k/Patch bei Auffüll-Logik vs 100k+/Patch ohne|Fix: Primärer Auftrag fertig UND < 300k verbraucht → nächstes Item nehmen (FEATURE_REQUEST > MARATHON_WORKFLOW > BACKLOG > Test-Schulden > Doku-Hygiene), Stopp bei ~350k|Nur sichere unabhängige Items als Auffüller, destruktive Ops nie|Ein HANDOVER am Ende statt pro Zwischen-Patch

---

## Gist-Sync (Schritt 12) ist gleichrangige Session-End-Pflicht — Faulheits-Catch #2 Derivat (Gist-Drift 2026-05-21)

Vier Sessions in Folge (B-074, Nala-Frontend-Separation, Hel-Frontend-Separation, FEATURE_REQUEST Teil-Lieferung) haben Schritt 12 des Session-Zyklus (Push + Gist-PATCH) übersprungen|Local-Commit + Push wurden gemacht, der Gist-PATCH für den Projekt-Gist (HANDOVER/MJOLNIR/STATUS/LESSONS/REPO_INDEX) fiel aus|Gist-Stand blieb auf B-aufraeumen (2026-05-18) eingefroren, drei große Sessions später war er vollständig veraltet|Folge: Supervisor (Chat-Instanz) bekommt beim ZUSAMMENFASSUNG-Fetch über Gist (Raw-Links kann Supervisor nicht fetchen, nur Gists) eine vier Sessions alte Realität — Mjölnir-Round-Trip auf Supervisor-Seite kaputt

---

## Claude-Design-Export ist UI-only — Migration in Backend-getriebenes Frontend braucht Foundation-Layer-Trennung (FEATURE_REQUEST 2026-05-20 → 2026-05-21)

Aus Claude Design exportierte UI-Prototypen sind pure UI-Code ohne jegliches Backend-Wiring (kein `fetch`, kein `EventSource`, kein `MediaRecorder`, kein `jsPDF`, kein Pacemaker)|Class-Names und IDs weichen typischerweise von der produktiven Implementierung ab (`#chat-list` vs `#chatMessages`, `.msg-bot` vs `.bot-message`)|Blindes Mergen oder Ueberschreiben der existierenden Frontend-Dateien bricht alle laufenden Features|Loesung: Multi-Step-Prozess **Foundation-Layer zuerst** (Design-Tokens als separate Datei, per CSS-Kaskade nur auf der Ziel-Seite ueberschrieben), **Komponenten-Migration spaeter** in einer Folge-Session

---

## Telegram Long-Polling braucht PID-Lock gegen Zombie-Caller (B-04, 2026-05-20 → 2026-05-21)

Telegram-Bot-API erlaubt genau einen aktiven `getUpdates`-Caller pro Bot-Token|Doppel-Start (vergessener alter Prozess, Hot-Reload, Worktree parallel) → HTTP-409-Flood im Terminal|Cleanly-handled-Shutdown reicht nicht (SIGKILL, Crash, OS-Restart umgehen das Shutdown-Hook)|Korrekte Loesung: PID-Lock-File, Cross-Platform Liveness-Check, einmalige WARN bei 409 + groesseres Backoff statt Log-Flood

---

## Inline-Monolith-Anti-Pattern: Frontend in Python-Triple-Quoted-Strings (Frontend-Separation, 2026-05-19; Hel bestaetigt 2026-05-20)

Wenn ein Modul Python-Logik UND HTML+CSS+JS-Bytes gemeinsam traegt, waechst es organisch zur Unwartbarkeit|Jeder Mini-Patch ist lokal als „schnellstes Vorgehen" rational (Edit nur im bestehenden `<style>`-Block, keine neue Datei, keine Static-Mount-Pruefung)|Source-Audit-Tests, die per `Path.read_text()` + Substring auf Inline-Bytes assertieren, **zementieren** die Monolith-Form (jede Separation bricht zig Tests)|Compound-Interest-Anti-Pattern: jeder Mini-Schritt rational, das Gesamtergebnis nicht mehr

---

## Lone Surrogates bei UTF-8-Datei-Schreibung aus Triple-Quoted-Source (Hel-Separation, 2026-05-20)

Wenn ein extrahierter String-Body lone Unicode-Surrogates enthaelt (typischerweise wenn 4-Byte-Emojis als Surrogate-Pair in Python-Source-Triple-Quoted-Strings landen und ein Halbpaar verloren geht), crasht `Path.write_text(..., encoding='utf-8')` mit `UnicodeEncodeError`|Fix: `body.encode('utf-8', errors='replace').decode('utf-8', errors='replace')` vor dem Schreiben — mirror der `_sanitize_unicode()`-Semantik|Loese das Problem genau dort wo der Endpoint es auch loest (am Output-Rand), nicht im Source

---

## Python-String-Literale → JS-Bytes via AST (Frontend-Separation, 2026-05-19)

Bei String-Literal-Extraktion aus Python-Source NIE ueber `Path.read_text()` arbeiten|Python escaped Backslashes anders als JS interpretiert|`ast.literal_eval()` ueber AST-Knoten liefert decoded Bytes|`node --check` ist die schnellste Verifikation

---

## OBERSTES GEBOT (P-umzug, 2026-05-16)

Chris terminalisiert NICHTS was Coda kann|NIEMALS git/pytest/pip/robocopy-Befehle an Chris delegieren|Coda merged Branches SELBST auf main + pusht SELBST vor Session-Ende|mjolnir.md enthält NUR was physisch unmöglich ist (Touch-Test, echtes Gerät, Docker Desktop UI)|Verstoß = Session-Abbruch + Korrektur

---

## Multi-Session-Status-Header für mjolnir.md (B-mjolnir-multisession, 2026-05-17)

mjolnir.md hat IMMER einen STATUS-Header als ersten Block|STATUS = FERTIG|IN_ARBEIT|BLOCKIERT|Bei IN_ARBEIT: WARNUNG fett oben + FEATURE_REQUEST NICHT umbenennen|Schutz gegen Chris-überschreibt-laufenden-Auftrag: neuer Request wird zu _QUEUED.md

---

## mjolnir.md-Round-Trip-Pflicht (B-mjolnir-fix, 2026-05-16; verschärft B-mjolnir-fix-2, 2026-05-17)

mjolnir.md wird am Ende JEDER Session überschrieben — ausnahmslos|Kein if/else, kein optional, kein "wenn Token übrig"|Gleichrangig mit HANDOVER und Push|Ohne mjolnir.md ist der Mjölnir-Round-Trip kaputt

---

## Datei-Konvention (projektübergreifend, Konsolidierung 2026-05-15)

- **Alle projektspezifischen Doku-Dateien tragen Suffix `_ZERBERUS`.** Umbenannt: `HANDOVER.md → HANDOVER_ZERBERUS.md`, `lessons.md → lessons_ZERBERUS.md`, `ZERBERUS_MARATHON_WORKFLOW.md → MARATHON_WORKFLOW_ZERBERUS.md` (Suffix-Position vereinheitlicht: PROJEKTNAME steht hinten, nicht vorne). Ausnahmen: `mjolnir.md` (ephemere Übergabe), `FEATURE_REQUEST_ZERBERUS.md` (Suffix bereits im Namen), `README.md`, `CHANGELOG.md`, `PROJEKTDOKUMENTATION.md`, `docs/DESIGN.md` (Sub-Doku in docs/ — Test-Suite + ein knappes Dutzend Module pinnen den Pfad, Umbenennung wäre invasiv und die Konvention zielt auf Root-Doku). Grund: Mehrere Projekte (Zerberus, Mjolnir, Rosa, Kintsugi) halten parallel ähnliche Dateien — Suffix verhindert Verwechslung in Mjolnir-Default-Prompts und cross-projekt-Suchen. Session-Start-Pflicht in `CLAUDE_ZERBERUS.md` und Session-Zyklus in `MARATHON_WORKFLOW_ZERBERUS.md` zeigen explizit auf die suffixierten Namen. Historische Patch-Notes in `MARATHON_WORKFLOW_ZERBERUS.md` und `SUPERVISOR_ZERBERUS.md` referenzieren weiterhin die alten Dateinamen — bewusst nicht angefasst, sonst werden Geschichts-Logs verfälscht.
- **`sync_repos.ps1` musste mitgezogen werden.** Die Datei pinnt `lessons.md` als Quelle für Ratatoskr- und Claude-Repo-Sync. Nach der Umbenennung zeigt sie auf `lessons_ZERBERUS.md`. Folgewirkung: das Ziel im Ratatoskr-Repo bleibt aus historischen Gründen `lessons.md` (kein Rename dort, sonst zerschießt es die Ratatoskr-Sync-History) — die neue Quell-Datei wird auf den alten Zielpfad gemappt. Bei nächster sync_repos-Iteration könnte man auch das Ziel umstellen, aber das geht über die Konsolidierung hinaus.
- **`docs/DESIGN.md` bleibt absichtlich ohne Suffix.** Tests in `test_design_system.py` und Code in `scripts/feature_audit.py`/`zerberus/core/reasoning_router.py` hartcodieren den Pfad. Eine Umbenennung würde mehrere Test-Files brechen + erfordert eine Lawine an Such-und-Ersetz-Edits. Da DESIGN.md im `docs/`-Subordner liegt (nicht im Root) und die Konsolidierungs-Konvention auf Root-Doku zielt, wird `docs/DESIGN.md` als Ausnahme dokumentiert.

---

## Remote-Control via Dateikonventionen statt API-Endpoints (P-mjolnir-workflow)

- **Mjoelnir (Hammerfall) muss Coda-Sessions remote steuern koennen: Sessions starten, Feature-Requests einreichen, Session-Zusammenfassungen abrufen.** Naheliegender Ansatz: API-Endpoints im Coda-Agent. Tatsaechliche Loesung in P-mjolnir-workflow (2026-05-15): **Dateikonventionen im Repo-Root**, der Marathon-Workflow prueft sie im Session-Zyklus.
- **Konkretes Vorgehen:** Zwei Dateinamen sind reserviert — `FEATURE_REQUEST_ZERBERUS.md` (priorisierter Arbeitsauftrag, Mjoelnir legt ihn per beliebigem Weg im Repo-Root ab) und `mjolnir.md` (Session-Zusammenfassung, Coda schreibt sie am Session-Ende, Mjoelnir fetcht sie via ZUSAMMENFASSUNG-Button). Beide werden im Session-Zyklus geprueft (Schritt 0 + Schritt 1 + Schritt 9 + Schritt 10). Nach Abarbeitung des Feature-Requests wird die Datei zu `_ERLEDIGT.md` umbenannt (Marker im Dateinamen, nicht im Inhalt — ueberlebt Refactors besser).
- **Vorteil gegenueber API-Endpoint im Agent:** keine neuen Endpoints, kein Auth-Mechanismus zu pflegen, die Konvention ist git-versioniert und damit auditierbar; der Steuer-Client (Mjoelnir) kann die Datei via beliebigem Pfad ablegen (eigene API, manuelles Editieren in VS Code, `gh` Push); der Agent prueft Dateien wenn er ohnehin Doku liest — null neuer Code-Pfad, null Risiko fuer den laufenden Server.
- **Backstop (Pflicht):** Die Konvention muss im **allerersten** Schritt des Session-Zyklus verankert sein, nicht prompt-abhaengig. Sonst kann ein Standard-Prompt sie nicht aktivieren — der Prompt-Text ist optimiert fuer den haeufigen Fall (kein Feature-Request offen), der Zyklus deckt den seltenen Fall ab. In P-mjolnir-workflow ist die `FEATURE_REQUEST`-Pruefung Schritt 0 (vor HANDOVER, vor MARATHON_WORKFLOW), die `mjolnir.md`-Pruefung Schritt 1.
- **Lesson generalisierbar:** Wer einen lokalen Agent-Workflow remote steuern will, fragt sich zuerst: „Welche Dateien liest der Agent ohnehin?" Wenn die Liste schon Files im Repo-Root enthaelt (HANDOVER, WORKFLOW, README), dann sind Dateikonventionen der billigste Pfad. API-Endpoints sind die richtige Antwort wenn (a) der Steuer-Client nicht ins Repo schreiben kann oder (b) die Steuerung Echtzeit-Feedback braucht (Status-Polling, Streaming). Fuer asynchrone Triggers + Polling-Abrufe ist Datei-Konvention robuster.

---

## Eskalation statt Endlos-Workaround (P-start-stable)

- Wenn ein Test physische Ressourcen braucht die Coda nicht hat (SSL-Certs, laufender Server mit echtem Dateisystem, Browser, Hardware) → ESKALIEREN, nicht Workaround bauen|"Das ist ein manueller Test für Chris" ist eine gültige Antwort

---

## Test-zementierte Doku-Anker muessen Doku-Refactors ueberleben (P-debt-12)

- **DESIGN.md hatte seit P-UI-meta (2026-05-07) zwei verlorene Anker (`Leitregel`, `projektübergreifend`, `shared-design.css`) durch komplette Datei-Neufassung.** Der Test `test_design_system::test_design_md_enthaelt_regel` aus Patch 151 zementierte diese drei Strings als Spec, der Doku-Refactor hat sie versehentlich rausgeschnitten — Test rot, drei Wochen unbemerkt. Anker waren urspruenglich in einer expliziten `## Leitregel`-Sektion mit Inhalt; der Refactor hat die Sektion gestrichen, aber den Test-Anker nicht migriert.
- **Loesung P-debt-12: kompakter Leitregel-Block direkt nach der Einleitung wieder eingefuegt.** Der Block ist die ehrliche Re-Interpretation der ursprunglichen Spec aus Patch 151 (L-001) und enthaelt alle drei Anker organisch im Inhalt: `**projektübergreifend**` als Adverb, `Leitregel` als Sektions-Headline, `shared-design.css` als Pfad-Verweis im Token-Verteilungs-Hinweis. Keine kuenstliche Marker-Wand, sondern echte Spec.
- **Lesson generalisierbar:** Wer eine Doku-Datei komplett neu fasst (full rewrite), muss die test-zementierten Anker explizit pruefen — `grep -E 'assert.*"' tests/test_<doku>.py` zeigt welche Strings die Tests erwarten. Beim Schreiben der neuen Doku diese Strings als Pflicht-Inhalt mitnehmen oder den Test mit-aktualisieren. Nicht beides offen lassen, sonst haelt sich das Failure jahrelang als pre-existing Schuld. Backstop: bei jedem Doku-Refactor die zugehoerige Test-Suite mitlaufen lassen, bevor der Refactor ausgeliefert wird.

---

## Splitscreen-Bug ohne Live-Diagnose: defensives Hardening + Verifikations-Schritt (P-UI-Hel-Split)

- **Bug-Reports im UI-Bereich brauchen oft eine Live-Diagnose, die der Coda-Agent nicht hat.** Chris hat den Hel-Splitscreen-Bug 2026-05-07 gemeldet ("Hel wird in der Mitte abgeschnitten — auch bei reichlich Breite/Hoehe"), aber Coda-Worktree hat keine Chrome-Extension verbunden, kann den Bug nicht visuell reproduzieren. Der direkte Weg "Bug starren bis Diagnose klar ist" ist verschlossen. Drei pragmatische Optionen: (a) raten und blind fixen — wahrscheinlich falsch, blast radius unklar, (b) auf naechste Session verschieben — Schuld bleibt offen, (c) **defensiv haerten + Verifikations-Schritt fuer User**. Option (c) ist die ehrliche Loesung wenn die Symptome typisch genug sind.
- **Konkretes Vorgehen P-UI-Hel-Split:** systematische Suche nach Splitscreen-Anti-Patterns im Code (Inline-Style-Grids ohne Stack-Override, sticky-Position mit fixem negativen Margin, Container-width ohne explizites `width: 100%`, fehlendes `overflow-x: hidden` auf html/body) — fixe alle als CSS-Defense-Schicht. Source-Audit-Tests zementieren die Sub-Selektoren (36 Tests in 12 Klassen). Klare visuelle Verifikations-Anweisung in HANDOVER + UI_BUG_HEL_SPLITSCREEN.md (5-Punkte-Checkliste fuer Chris). Wenn die Verifikation Restprobleme zeigt: Folge-Patch P-UI-Hel-Split-2 mit Live-Diagnose (Chris liefert Screenshot).
- **Defensive Hardening hat einen viel kleineren Blast-Radius als gezielte Edits.** `html { overflow-x: hidden }` plus Splitscreen-Breakpoint kann nichts schlimmer machen — entweder es war kein Bug (kein Effekt) oder es loest den Bug. Die Anti-Regress-Klasse `TestKeineRegressionVorgaengerCSS` (8 Tests) zementiert dass alle bekannten Hel-CSS-Strukturen unangetastet bleiben — der Patch ist additiv, nicht modifikativ.
- **Lesson generalisierbar:** Bug-Reports ohne Live-Diagnose-Pfad sind nicht "blockiert bis User mehr Daten liefert". Wenn der Bug in eine bekannte Symptom-Klasse faellt (Splitscreen-Layout, Fokus-Drift, Race-Condition bei Async-Init), ist der konstruktive Weg: alle plausiblen Anti-Patterns systematisch absichern + Verifikations-Schritt in HANDOVER definieren. Backstop: Anti-Regress-Tests damit die Defense-Schicht nicht spaeter weg-refactored wird.

---

## Inline-Style-Grids brauchen Attribute-Selectors fuer Responsive-Override (P-UI-Hel-Split)

- **Legacy-HTML mit Inline-Style-Grids (`<div style="display:grid;grid-template-columns:1fr 1fr;...">`) ist der schwierigste responsive Fall.** Vier Stellen in `zerberus/app/routers/hel.py` haben das genau so. Klassen-Selektoren erreichen sie nicht. Refactor zu eigenen Helper-Klassen waere die saubere Loesung — aber bedeutet acht Edits an HTML, Risiko fuer subtile Layout-Drift. Memory-Regel "Don't add abstractions beyond what the task requires" sagt: nimm den minimalen Pfad.
- **Loesung: CSS-Defense-Schicht via Attribute-Selector mit `!important`.** `[style*="grid-template-columns:1fr 1fr"] { grid-template-columns: 1fr !important }` matched alle Inline-Style-Grids mit `1fr 1fr`. `!important` ist Pflicht, weil Inline-Styles sonst die CSS-Spezifitaets-Schlacht gewinnen.
- **Reihenfolge-Falle: Substring-Match.** `[style*="1fr 1fr"]` matched auch `1fr 1fr 1fr`. Daher MUSS der spezifischere 3-Spalten-Selektor VOR dem allgemeinen 2-Spalten-Selektor stehen — sonst ueberschreibt CSS-Source-Order das spezifischere Verhalten. Test `TestSplitscreenGridStacking::test_grid_3col_stack` zementiert diese Reihenfolge per `block.find()`-Vergleich (idx_3 < idx_2).
- **Lesson generalisierbar:** Inline-Styles sind kein Refactor-Trigger, sondern ein Defense-Trigger — wenn du sie nicht via Klasse adressieren kannst, gibt es Attribute-Selectors. Substring-Match-Fallen sind ueberall wo CSS-Selektoren mit `[attr*=...]` arbeiten — der spezifischere Selektor MUSS vor dem allgemeineren stehen. Test der die Reihenfolge zementiert ist die Doku-Schicht.

---

## RAG-Doku mit Stand-Anker: kompakt halten und FAQ in den ersten 4000 Zeichen (P-debt-11)

- **Stand-Anker waechst, wenn Patch-Beschreibungen volle Aenderungs-Listen tragen — RAG-Chunking driftet vom ersten Chunk weg.** `huginn_kennt_zerberus.md` hatte ueber zehn aktive Patch-Eintraege als Bullet-Liste mit jeweils 4-7 KB Detail-Beschreibung im "Aktueller Stand"-Block. Folge: der FAQ-Block (woertliche Stand-Fragen plus Antworten — der Embedding-Match-Garant aus P213-pre-4) wurde aus dem ersten 4000-Zeichen-Fenster geschoben. `test_p213_pre_4_huginn_ranking::test_faq_block_appears_near_top` schlug an. Plus: `test_doc_does_not_use_code_blocks` schlug an, weil ein Patch-Eintrag den Triple-Backtick als Code-Marker-Beispiel enthielt — ` ``` ` im Stand-Anker pollutet den ersten RAG-Chunk mit Code-Block-Syntax, die kein semantischer Inhalt ist.
- **Loesung: alte Patch-Eintraege ausduennen, FAQ-Block direkt nach dem letzten Bullet platzieren.** P-debt-11 hat den Stand-Anker auf eine kompakte Liste reduziert: 1-2 Saetze pro aktivem Patch (Letzter Patch + Vorletzter + Letzter Code-Patch + Letzter Maintenance + Letzter Tooling), eine ein-Zeilen-Liste fuer "Fruehere Phase-5c-Patches" (UI-1..UI-10 als reine Stichworte), Phase-Status, Test-Zahl, Datum. Detail-Beschreibungen wandern in `docs/PROJEKTDOKUMENTATION.md` (Patch-Historie, vollstaendige Doku). Plus: VRAM-Code-Block in Inline-Text umgeschrieben (kein ` ``` ` mehr in der Doku). Plus: neue Sektion "Aktuelle Konfiguration (Live-Werte zur Laufzeit)" verweist auf den dynamisch generierten `[Aktuelle System-Informationen]`-Block aus P185 — statt Modell-Versionsnummern statisch zu pflegen, wird das laufende System gefragt.
- **Lesson generalisierbar:** RAG-Doku unterliegt anderen Pflege-Regeln als Code-Doku. Code-Doku darf vollstaendig sein (mehr Detail = besseres LLM-Verstaendnis), aber RAG-Doku ist Chunk-getrieben: der erste Chunk ist der Embedding-Match-Hot-Spot, alles dahinter rankt schlechter. Stand-Anker + FAQ MUSS in den ersten ~4000 Zeichen liegen — alles andere ist optional. Bei jedem neuen Patch der den Stand-Anker erweitert: pruefen ob die FAQ-Position noch passt (`python -c "p = open(...).read(); print(p.find('Bei welchem Patch sind wir'))"`). Wenn nicht: alten Patch-Eintrag in eine ein-Zeilen-Bullet-Liste reduzieren, Detail nach `PROJEKTDOKUMENTATION.md` verlagern. Backstop: `test_faq_block_appears_near_top` zementiert die 4000-Zeichen-Grenze als harte Spec.

---

## Recovery-Patches darf man nicht einfrieren — sie sind Zwischenzustaende (P217)

- **P218-pre hat einen destruktiven Recovery eingefuehrt: bei Dim-Mismatch wurde der gesamte Slot ueber ``_reset_index_inplace`` neu mit der neuen Dimension aufgebaut, alte Chunks gingen verloren.** Das war ein "Bug-Patch" (akuter Schmerz: HTTP 500 im Upload-Pfad), kein architektonischer Fix. Die Tests zementierten das destruktive Verhalten als Erwartung — das ist der gefaehrliche Teil. Wer Recovery als Spec einbetoniert, schuetzt den Bug vor seiner eigentlichen Loesung. P217 hat das aufgeloest: wenn der DualEmbedder-Switch einen anderen Slot adressieren kann (DE-Vec → DE-Slot, EN-Vec → EN-Slot), darf der falsche Slot nicht zerstoert werden — er ist der Heimat-Slot des anderen Embedders. **Lesson generalisierbar:** Recovery-Tests duerfen das Recovery-Verhalten zementieren, aber sie duerfen NICHT als Spec-Definition gelten. Beim naechsten architektonischen Fix muss klar sein: das destruktive Verhalten war nur ein Notnagel, kein Endzustand. Bei Recovery-Patches gehoert in den Tests ein WHY-Docstring der die Architektur-Schuld benennt — sonst wird sie zur Spec.
- **Sprach-Routing als architektonischer Fix fuer einen vermeintlichen Recovery-Pfad.** Vor P217 hatte die globale RAG-Pipeline EINEN Add-Pfad (`_add_to_index`) und ZWEI Indizes (`_index` DE, `_en_index` EN). Adds gingen immer in den DE-Slot, EN-Vektoren mit 1024 Dim sprengten den DE-Slot (Dim 768) → Reset → DE-Daten weg. P217 fuegt einen `language`-Parameter hinzu und routet pro Vektor. Das ist eine kleine Code-Aenderung (eine Verzweigung in `_add_to_index`), aber strukturell wichtig: der Slot ist jetzt eine SPRACH-EINHEIT, kein BLINDER-CONTAINER. **Lesson generalisierbar:** wenn zwei "Reaktoren" einen einzigen Container teilen und periodisch ineinander krachen, ist der Fix nicht "Container groesser machen" oder "Reset-Recovery". Der Fix ist: zwei Container, eine Routing-Regel. FAISS-Indizes pro Sprache sind hier nur ein Beispiel — gilt auch fuer Caches mit zwei verschiedenen TTL-Klassen, fuer Logging-Streams pro Severity, etc.
- **DELETE-Pfad fuer Soft-Delete-Stores muss physische Bereinigung integrieren.** Vor P217 setzte `rag_document_delete` nur `deleted=True` in DE-Metadata und schloss aus dem Retrieval aus — das ist O(1)-Soft-Delete, billig. Aber: `_index.ntotal` zaehlte weiter alle Vektoren mit, und (Bug 2) EN-Metadata wurde gar nicht beruehrt. User klickte "alle Doks loeschen" → Hel zeigte 0 Doks aber 47 Chunks. Eindeutiges UX-Bug-Signal. P217 macht zusaetzlich einen physischen Reindex pro Sprach-Slot nach dem Soft-Delete: `_rebuild_index_atomic(verbliebene_chunks, settings, language=...)` — atomar mit Pre-Backup, FAISS-BAK-Muster. Bei leerer Chunk-Liste (alle Doks geloescht) wird der Slot leer-reindiziert (Dim bleibt erhalten, ntotal=0). **Lesson generalisierbar:** Soft-Delete-Stores brauchen einen Cleanup-Trigger der den Soft-Delete physisch bereinigt — entweder bei jeder DELETE-Operation (synchron, einfach) oder via Cron-Job (asynchron, billiger fuer grosse Stores). Ohne Cleanup driftet der Soft-Delete-State zwischen Listen-Filter (zeigt nur sichtbare) und Total-Counter (zaehlt physisch alles), und der User merkt das als "die Geister-Chunks haben mich gestorben".
- **`for_write=True`-Param in Resolver-Helpern: kleine API, grosser Sicherheitsgewinn.** Pre-P217 hatte `_resolve_paths(language="en", dual_aware=True)` ein None-Fallback: wenn `_en_index is None` (Slot noch nicht geladen), wurde stattdessen der DE-Pfad zurueckgegeben. Das war fuer READ-Operationen sinnvoll (EN-Index existiert nicht? → DE-Index als Fallback retrievaln), aber fuer WRITE-Operationen toedlich (erster EN-Schreibvorgang landet in DE-Pfad → Bug-1-Kern). Lesson: Resolver-Helper sollten zwischen Read- und Write-Semantik unterscheiden. P217 fuegt `for_write=True` hinzu: bei Write wird `language` strikt angewandt, kein Fallback. **Lesson generalisierbar:** in jedem Helper, der Pfade/Resourcen aufloest, eine explizite Read/Write-Trennung anbieten — Read kann Fallback nutzen (Robustheit), Write muss strikt sein (Korrektheit). Nicht selten ist genau diese Trennung der naechste Bugfix.

---

## Polish-Patches duerfen Struktur erweitern, wenn sie Bestands-API wiederverwenden (P-UI-Polish-2)

- **P-UI-Polish (1) war reines CSS-Wert-Polish**, ohne neuen DOM, ohne neues JS. P-UI-Polish-2 hat zusaetzlich eine **kompakte Sidebar-Projekt-Liste** eingebaut — das ist neue HTML- + neue CSS- + neue JS-Funktion. Auf den ersten Blick ein "strukturelles" Add, also eigentlich ausserhalb des "Polish"-Scopes. Aber: die neue Funktion `renderSidebarProjects()` nutzt **100 % der existierenden Daten- und State-API aus P-UI-5** — denselben `window.__nalaProjectsCache`, denselben Pin-State (`getPinnedProjectIds`), denselben Active-State (`getActiveProjectId`), dieselbe `projectsSortMode`-Variable. Kein neuer Daten-Pfad, kein neuer Cache, kein neues localStorage-Key. Nur ein neuer Render-Pfad + Display-Container.
- **Hooks halten die Liste mit der View synchron** — `setActiveProject` / `clearActiveProject` / `toggleProjectPin` / `setProjectsSortMode` / `loadNalaProjects` / `toggleSidebar` rufen alle zusaetzlich `renderSidebarProjects()` (mit `typeof === 'function'`-Guard, fail-quiet). Das ist die kritische Eigenschaft: User-Action triggert IMMER beide Render-Pfade, also keine Drift zwischen Sidebar-Liste und View.
- **Lesson generalisierbar:** "Polish vs. Struktur" ist keine harte Linie, sondern ein Spektrum. Der bessere Test: bringt der Patch eigene Daten/State/Cache mit? Wenn JA → strukturell (Phase-Patch). Wenn NEIN (nur neuer Render auf existing State) → Polish ist OK, weil der Blast-Radius klein bleibt. Die Sidebar-Projekt-Liste ist genau dieser Fall: 50 Zeilen JS, kein neuer Daten-Code, fail-quiet integration. Backstop: Anti-Regress-Klasse `TestPhase5cMechanikIntakt` zementiert dass die existierende API unangetastet bleibt.

---

## Save-Button-Wanderung Header → Sidebar zeigt warum Anker-Tests Spec-Verschiebungen explizit dokumentieren muessen (P-UI-Polish-2)

- **`test_settings_umbau::test_schraubenschluessel_nicht_in_topbar`** zementierte fuer Patch-142 die Aussage "Schraubenschluessel weg AUS dem Header — aber Export-Button bleibt im Header" (`assert "openExportMenu()" in header`). P-UI-Polish-2 hat den Export-Button (💾) aus dem Header in die Sidebar verschoben (Spec: Save-Button wandert ins Hamburger-Menue). Der Test fiel rot — die Spec hatte sich verschoben.
- **Pattern: alter `assert X in header` → neuer `assert X NICHT in <button> im Header` plus neuer Test `assert X in sidebar`**. Die Wanderung ist explizit in beide Richtungen dokumentiert: was vorher galt (Negativ-Anti-Assert mit WHY-Docstring der die Wanderung erklaert) UND was jetzt gilt (Positive-Assert auf der neuen Position). Plus: WHY-Docstring im alten Test-Methode dokumentiert die Spec-Verschiebung — `UPDATED P-UI-Polish-2 (2026-05-09): Spec-Verschiebung — der Save-Button wandert aus dem Header in die Sidebar`.
- **Lesson generalisierbar:** Element-Wanderung (DOM-Position aendert sich, Funktion bleibt gleich) ist haeufiger als ein neues Feature. Bei Wanderungen: nicht nur den alten Test rot lassen oder still umpolen, sondern das **Wo** explizit machen — alter Ort: Anti-Assert. Neuer Ort: Positive-Assert. Damit hat der Test in einem Jahr immer noch Doku-Wert: Coda XYZ kann nachvollziehen warum der Button da ist wo er ist (und warum nicht mehr da wo er mal war). Generalisierung von P-UI-Polish-1 Lesson: WHY-Docstring + Anti-Asserts gilt auch fuer Wanderungen.

---

## Visuelles Polish kann reine CSS-Wert-Aenderung sein, wenn die Mechanik durchdacht war (P-UI-Polish)

- **Phase 5c hatte 11 Schritte (UI-1..UI-11) — alle strukturell**: Layout-Grundregel (P-UI-1), Collapse (P-UI-2/3), Sidebar-Shift (P-UI-4), Projektseite (P-UI-5), Reasoning-Block (P-UI-6), LLM-Icon (P-UI-7), Scroll-Nav (P-UI-8), 2-Achsen-Skalierung (P-UI-9), Sentiment-Ambient (P-UI-10), Reasoning-Mapping (P-UI-11). Die Mechanik war stimmig, aber das **visuelle Gewicht** der Default-Werte (Bot-Bubble mit `rgba(26,47,78,0.85)` Background + `box-shadow: 0 3px 8px rgba(0,0,0,0.3)`, User-Bubble mit `rgba(236,64,122,0.88)` + Tail-Radius) war im Kontrast zum cleanen Mockup [`docs/NalaMockup.jsx`](docs/NalaMockup.jsx). P-UI-Polish hat den Look auf Mockup-Niveau gebracht — **OHNE strukturelle Aenderung**, reine CSS-Werte (Background, Border, Padding, Box-Shadow, Margin/Gap, Font-Size).
- **Defense-in-Depth gegen Refactor-Regression: explizite Anti-Regress-Klasse in den neuen Tests.** `test_p_ui_polish_visual_weight.py` hat eine eigene Klasse `TestPhase5cMechanikIntakt` mit acht Tests, die explizit pruefen, dass P-UI-1 (max-width: 100%, align-self: stretch), P-UI-6 (Reasoning-Block-Klasse), P-UI-7 (LLM-Icon-Klasse), P-UI-8 (Scroll-Nav-Klasse) und P-UI-10 (Ambient-Layer + Stage-Override-Selektor) unangetastet bleiben. Wenn jemand spaeter den Polish-Patch zurueckrolliert oder versehentlich strukturell drueberbuegelt, schlagen die Mechanik-Tests sofort an.
- **Lesson generalisierbar:** Polish-Patches sollten klar deklarieren, was sie NICHT machen — und das per Anti-Regress-Tests zementieren. Der Test-File-Name signalisiert "visual_weight" (Polish-Bereich), die Anti-Regress-Klasse prueft den Mechanik-Bereich (UI-1..11). Reine CSS-Polish-Patches haben einen **viel kleineren Blast-Radius** als strukturelle UI-Patches, sind aber genauso oft Angriffsziel von "verbesser mal eben"-Refactors. Backstop: explizite Tests, die die Spec-Trennung halten.

---

## Anti-Invariante 1 + Anti-Invariante 2 koexistieren wenn ihre Geltungsbereiche getrennt sind (P-UI-Polish)

- **Pre-P-UI-Polish hatte Anti-Invariante 1: "Bubble-Backgrounds duerfen NIE `#000000` oder `transparent` als Default haben — nie schwarz auf schwarz".** Patch-183-Sanitizer (`_isBubbleBlack()`) zementiert das durch Regex-Match auf `#000`, `rgb(0,0,0)`, `rgba(0,0,0,A)`, `transparent`, `none`. Ziel: User-Customization (HSL-Slider, Color-Picker, Theme-Reset) darf nie unlesbare Bubble produzieren.
- **P-UI-Polish fuehrt Anti-Invariante 2 ein: "`.bot-message` hat KEINEN Bubble-Hintergrund".** Spec-Vorbild Mockup zeigt freistehenden LLM-Text wie ChatGPT/Claude/Mistral. Erster Reflex: `--bubble-llm-bg: transparent` im `:root`-Block. **Aber das wuerde Anti-Invariante 1 brechen** — der Sanitizer erkennt `transparent` als Black-Bug und wuerde die Variable zurueckaendern.
- **Loesung: getrennte Geltungsbereiche.** Variable `--bubble-llm-bg` bleibt `rgba(26, 47, 78, 0.85)` im `:root` — Anti-Invariante 1 weiter aktiv fuer typing-indicator (Z. ~1862) und Color-Picker-Fallback. `.bot-message` overridet **nur lokal** auf `background: transparent` — Anti-Invariante 2 nur fuer den LLM-Antwort-Block. Beide Anti-Invarianten sind gleichzeitig wahr, weil sie auf verschiedenen Geltungsbereichen operieren (globale Variable vs. spezifische Bubble-Klasse).
- **Lesson generalisierbar:** wenn eine neue Anti-Invariante einer existing zu widersprechen scheint, ist das oft ein Geltungsbereich-Problem, kein Konflikt. Statt die existing zu lockern: getrennte Layer einfuehren (`:root`-Variable vs. spezifische Klassen-Override). Backstop: Anti-Asserts in den Tests die explizit machen welche Variable wo gilt — z.B. `assert "background: var(--bubble-llm-bg)" not in bot_block` (.bot-message darf nicht mehr die Variable nutzen) plus `assert "--bubble-llm-bg: rgba(26, 47, 78, 0.85)" in root_block` (Variable bleibt fuer typing-indicator).

---

## Bei Test-Anker-Updates wegen Spec-Verschiebung: WHY-Docstring + alte Werte als `not in`-Anti-Asserts (P-UI-Polish)

- **Wenn ein Test seine Spec-Werte aendert (Spec hat sich verschoben), reicht das nicht, einfach den `assert`-Wert anzupassen** — der Test verliert sonst die Doku-Spur. P-UI-Polish hatte drei Tests in `test_p_ui_1_layout.py` (`test_user_bubble_box_shadow_bleibt`, `test_bot_bubble_box_shadow_bleibt`, `test_bubble_border_radius_tail_bleibt`), die ehemals "P-UI-1 fordert visuellen Tail + Shadow" zementierten. P-UI-Polish hat den Tail entfernt (User: 18/18/4/18 → 10px gleichmaessig; Bot: 18/18/18/4 → 0) und den Shadow auf Default `none` gesetzt — die Tests muessten umgeschrieben werden.
- **Pattern: alter Test → neuer Test mit explizitem WHY-Docstring**, in dem die Spec-Verschiebung dokumentiert ist (P-UI-1-Anforderung → P-UI-Polish-Aenderung) plus die Begruendung (Mockup-Vorbild) plus die neuen Werte. Plus: alte Werte als **`not in`-Anti-Asserts** stehenlassen — `assert "18px 18px 4px 18px" not in user_block`. Damit ist der Test selbst-dokumentierend: der naechste Refactor sieht, was vorher galt UND was jetzt gilt.
- **Lesson generalisierbar:** P-debt-10-Pattern (WHY-Docstring + harte `assert`-Fehlermeldungen) gilt nicht nur bei broken Tests, sondern auch bei Spec-Verschiebung. Die alte Form als Anti-Assert plus die neue Form als Positive-Assert macht den Test zur kleinen Zeitkapsel — naechster Coda sieht, was P-UI-1 wollte UND was P-UI-Polish geaendert hat. Spec-Drift ist kein Bug, ist eine Verschiebung; die Tests muessen die Verschiebung mit-dokumentieren, nicht still uebernehmen.

---

## Fluechtige HTML-Wrapper als Test-Anker sind brüchig — stabile IDs auf Root-Level-Elementen oder selbst-dokumentierende Comments (P-debt-10)

- **`test_settings_umbau::test_mein_ton_nicht_mehr_in_sidebar` war seit P-UI-4 rot** weil sein End-Anker `</div>\n        <div class="overlay"` ein flüchtiges Layout-Element war (Overlay-Backdrop). P-UI-4 hat den Backdrop entfernt — der Test fiel still durch, weil ein Fallback `find("overlay", sidebar_start)` den nächsten lowercase-`overlay`-String traf (JS-Comment Z. 5810 in nala.py), den `sidebar_block` viel zu groß zog und das Settings-Modal mitsamt `id="my-prompt-area"` einschloss. Test rot ohne dass jemand merkte WARUM — der Schulden-Eintrag war drei Wochen alt. P-debt-10-Fix nutzt jetzt `<div id="ee-modal"` (Easter-Egg-Modal Patch 100) als End-Anker: Root-Level-Geschwister direkt nach dem schließenden Sidebar-Tag, unique in nala.py, keine Layout-Bedeutung — verschwindet nicht beim nächsten UI-Refactor.
- **Lesson generalisierbar:** Source-Audit-Tests die Substring-Slices oder `find()`-Anker nutzen, brauchen Anker die (a) unique in der ganzen Datei sind, (b) eine bewusste Identitaet haben (eindeutige IDs, semantische HTML-Comments wie `Patch 142 (B-013)`, oder Patch-Marker wie `// P-UI-X`), (c) NICHT von Layout-Refactors weggewischt werden koennen. Layout-Wrapper-Klassen wie `.overlay`, `.container`, `.wrapper` sind verboten als Anker — sie sterben beim ersten CSS-Refactor. Plus: keine `if not found: fallback to broader search`-Patterns — Fallback maskiert den Bruch und verzoegert die Diagnose um Wochen. Lieber harter `assert anker_gefunden, "WARUM"` mit klarer Fehlermeldung — Test bricht beim ersten Refactor sofort und erklaert sich selbst.

---

## Pre-Call-Heuristik schlaegt Two-Pass-Routing bei modell-konditionalem LLM-Switch (P-UI-11)

- **Two-Pass-Switch verdoppelt Token-Kosten und Latenz, ohne dass der Praezisionsgewinn das rechtfertigt.** P-UI-11 musste entscheiden, wann der Intent-Router auf eine Reasoning-Variante umschaltet. Naive Idee: erster `llm_service.call(...)` mit Standard-Modell, parsen des `effort`-Headers (`intent_parser.ParsedResponse`, Range 1..5), wenn `effort >= 4` → zweiter Call mit Reasoning-Variante. Klar implementierbar, klar Spec-konform — aber zwei Calls fuer JEDE Reasoning-Switch-Entscheidung, plus mindestens 1× extra Roundtrip-Latenz fuer den User. Das macht den UX-Gewinn ("der User merkt automatisch hochgeschaltete Reasoning") kaputt durch UX-Verlust ("Antwort kommt 2× langsamer").
- **Pre-Call-Heuristik ist die billigere Variante, die fuer den 90 %-Fall reicht.** P-UI-11 schaetzt `effort_score` (0..10) PRE-Call aus der User-Message: Wortzahl-Buckets, Reasoning-Trigger-Wortliste (`warum`, `step by step`, `begruende`, `analysiere`, deutsche + englische), Code-Marker (` ``` `, `def`, `class`), Frage-Wort + `?`, Negativ-Marker (`kurz`, `tldr`). Schwellwert `> 7` schaltet auf Reasoning-Variante um. Heuristik ist weniger praezise als die LLM-Selbsteinschaetzung — false-positives bei langen aber simplen Anfragen, false-negatives bei kurzen aber komplexen — aber kostenneutral. Spec-Garantie ist erfuellt: "Router switcht API-Call auf Reasoning-Variante" impliziert Pre-Switch.
- **Lesson generalisierbar:** wenn ein Routing-Entscheid einen LLM-Call beeinflusst, ist die Pre-Call-Heuristik der pragmatische Default. Two-Pass nur dann, wenn (a) die Heuristik beweisbar (Messung erforderlich) so unpraezise ist, dass die Falsch-Routing-Rate die zusaetzlichen Token-Kosten kompensiert, ODER (b) der Routing-Output selbst ein Wert ist, den nur das LLM zuverlaessig liefern kann (Tool-Call-Detection, Inhalt-Klassifikation jenseits Keyword-Matching). Backstop: Source-Audit-Test, der den Hook-Block `select_reasoning_for_message(...)` VOR dem ersten `llm_service.call(...)` zementiert.

---

## 0..10-Heuristik-Range entkoppelt vom 1..5-Header-Feld vermeidet Spec-Drift-Verwirrung (P-UI-11)

- **Spec sagte `effort_score > 7`, existing `effort` ist 1..5 → erst Spec-Bug-Verdacht, dann Aha-Moment.** `intent_parser.ParsedResponse.effort` ist ein 1..5-Integer aus dem LLM-Output-JSON-Header (Patch 164). Spec-Vorgabe NALA_UI_REDESIGN_PROMPT.md 12 sagt `> 7`. Erste Reflex-Annahme: Spec ist falsch, sollte `> 4` sein. Aber: das existing `effort` 1..5 wird vom LLM SELBST gesetzt — also erst NACH dem Call bekannt. Pre-Call-Heuristik ist eine ANDERE Score-Welt, weil die Datenquelle (User-Input) und der Berechnungs-Zeitpunkt (vor dem Call) anders sind.
- **Loesung: zwei Score-Welten halten, beide passen zur Spec.** P-UI-11 fuehrt einen separaten 0..10-Heuristik-Score ein (`compute_effort_score` in `reasoning_router.py`), der das Spec-`> 7` direkt umsetzt. Der existing 1..5-Header-`effort` bleibt unangetastet — `intent_parser.py` und `pipeline.ParsedResponse` sind nicht touched. Wer den Header-`effort` weiter nutzt (z.B. fuer HitL-Eskalation oder Cost-Tracking), bekommt seine Range; wer den Reasoning-Switch macht, bekommt den anderen Score. Modul-Docstring dokumentiert die Trennung explizit.
- **Lesson generalisierbar:** wenn ein neuer Score eingefuehrt wird, der semantisch zu einem existing Score "gehoert" aber technisch entkoppelt ist (verschiedene Datenquellen, verschiedene Berechnungs-Zeitpunkte), getrennte Range vermeidet Verwirrung. Spec-Vorgabe `> 7` plus existing Range 1..5 ist NICHT "Spec-Bug" — es sind zwei verschiedene Score-Welten. Naming-Pflicht: nicht den existing Variable-Namen wiederverwenden, sondern explizit machen (`effort_score` Pre-Call vs. `effort` Post-Call). Backstop: dass beide Welten in unterschiedlichen Modulen liegen (`zerberus.core.reasoning_router` vs. `zerberus.core.intent_parser`) plus Tests die NICHT crossover-importieren.

---

## Whitelist-Mapping mit explicit-None-Eintraegen ist sicherer als Default-Allow (P-UI-11)

- **Drei-Fall-Whitelist statt Default-Allow.** P-UI-11s `REASONING_MAPPING` ist ein Dict, in dem jeder Eintrag drei Faelle erlaubt: (a) `key: dict` → Switch erlaubt, dieses Modell hat eine Reasoning-Variante; (b) `key: None` → kein Switch, dieses Modell ist EXPLIZIT geprueft worden, hat keine Reasoning-Variante (Spec: "llama/3.1-70b → reasoning: null"); (c) `key NICHT vorhanden` → kein Switch, dieses Modell ist UNBEKANNT, bewusste Default-Block. Aus Caller-Sicht sehen (b) und (c) identisch aus (`select_reasoning_variant` returnt `None`), aber semantisch sind sie unterschiedlich: (b) ist Doku ("ich habe das Modell geprueft, kein Reasoning"), (c) ist Default ("ich kenne das Modell nicht, ich switche nicht").
- **Default-Allow-Alternative waere unsicher.** Naive Implementation waere `if base_model in DEEPSEEK_MODELS: switch_to_r1; elif base_model in MISTRAL_MODELS: thinking; else: try_thinking_anyway`. Das wuerde bei einem neu hinzugefuegten Modell (z.B. `xai/grok-4`) versuchen, ein `thinking`-Parameter zu uebergeben, das das Modell nicht kennt — entweder Crash oder stilles Ignorieren mit Spec-Verletzung ("Wenn `reasoning_variant: null` → kein Switch, kein Fehler"). Whitelist-Stil schliesst das aus.
- **Lesson generalisierbar:** wenn ein Mapping zwischen zwei Welten Sicherheit verlangt (z.B. "Standard-Modell ↦ Reasoning-Variante darf NUR bei explizit erlaubten Modellen schalten"), drei-Fall-Whitelist ist der saubere Pfad. Die `None`-Eintraege sind Doku-wertvoll: sie zeigen, dass das Modell bedacht wurde, nicht nur weggefallen. Backstop: Source-Audit-Tests die alle vier Spec-Modelle namentlich pruefen (`deepseek/v3.2`, `mistral/large`, `claude/sonnet`, `llama/3.1-70b`) plus einen Test der bei unbekanntem Modell `None` erwartet (`test_kein_switch_bei_unknown_model`). Dasselbe Pattern ist auch fuer Permission-Matrices, Tool-Routing und Plugin-Loading nuetzlich.

---

## `isolation: isolate` ist die minimale Aenderung, um einen Container zum eigenen stacking context zu machen (P-UI-10)

- **P-UI-10 brauchte einen Container mit eigenem stacking context, damit ein Child mit `z-index: -1` *innerhalb* der Container-Grenzen sitzt** und nicht hinter dem Container-Background nach body-Ebene durchschlaegt. Drei mainstream-Optionen waeren moeglich: `position: relative` mit zusaetzlichem `z-index: 0` (aendert das Layout potentiell, da Static-→-Relative auch andere Containing-Block-Effekte triggert), `transform: translateZ(0)` (kompositorischer Trick, kann Sub-Pixel-Rendering oder Hidden-Overflow-Effekte verursachen), oder `isolation: isolate` (CSS-Property speziell dafuer designed, KEINE Layout-Side-Effects, keine Transform-Compositing-Surprise). P-UI-10 hat `isolation: isolate` gewaehlt — der Patch berichtigt eine einzige Property, das Verhalten ist im Code-Review klar als "ich will einen stacking context" lesbar. **Lesson generalisierbar:** wenn ein Element zum Container fuer einen `z-index: -1`-Child werden soll und keine bestehende Stacking-Context-Trigger-Property gesetzt ist (transform, opacity < 1, filter, mix-blend-mode, will-change), `isolation: isolate` ist der saubere Pfad — kein Layout-Side-Effect, klare Intention. Bonus: dieser Trick ist auch nuetzlich fuer Komponenten, die `mix-blend-mode`-Children haben und nicht wollen dass deren Mischung auf body-Ebene durchgreift.

---

## Substring-Splits in Source-Audit-Tests sind robust nur, wenn die Substring-Form unique im Source ist (P-UI-10)

- **Erste P-UI-10-Iteration nutzte `body.pui10-ambient-stage-N .message,` als Selektor-Liste** — die Test-Slices in `test_p_ui_1_layout.py` (`nala_src.split(".message {")[1]`) traten auf den P-UI-10-Block statt auf die reale `.message`-Definition, weil `.message {` als Substring der Selektor-Liste-End-Form vorkam (`.message {` am Ende von `... .message,\n        body.pui10-ambient-stage-5 .message {`). Folge: 8 pre-existing Source-Audit-Tests fielen, weil sie den falschen Block lasen.
- **Fix war minimal-invasive**: `.message[class]` als Attribut-Match — semantisch identisch (jedes `.message`-Element hat ein `class`-Attribut, das ist immer wahr), aber der literale Substring `.message {` taucht im pui10-Block nicht mehr auf. Plus: der erklaerende Kommentar musste auch von `.message {` auf "Bubble-Selektor" umformuliert werden, weil sonst der Kommentar-Text selbst als Substring matchte.
- **Lesson generalisierbar:** wenn Source-Audit-Tests Substring-Splits nutzen, jeder neue Patch muss seine Selektoren so formulieren, dass die "kanonische Form" des Bestand-Selektors NICHT als Substring im neuen Block auftaucht. Backstop: einmal `grep ".message {" nala.py` (oder das jeweilige Sentinel) nach jedem Patch ausfuehren, um zu verifizieren dass es nur an der erwarteten EINEN Stelle vorkommt — und das gilt auch fuer Kommentar-Text. Alternative robustere Test-Slices (Lookahead/Negative-Match in Regex statt simples `split`) sind aufwaendiger zu schreiben aber widerstandsfaehiger gegen kuenftige Ergaenzungen.

---

## State-Machines mit Last-Wins-Tie-Break sind weniger flackerig als First-Wins (P-UI-10)

- **Der `pui10_majority`-Voter iteriert RUECKWAERTS durch die History.** Bei Gleichstand (z.B. zwei mal joy + zwei mal anger nach 4 Klassifikationen mit History-max 3) gewinnt die juengste Klassifikation, weil das `for (let i = history.length - 1; i >= 0; i--)`-`if (counts[c] > bestCount)`-Konstrukt erst die letzte Kategorie als Default nimmt und dann nur noch echte Mehrheiten kippen darf — nicht Gleichstaende.
- **First-Wins (vorwaerts-Iteration mit `>`) wuerde bei Stimmungswechsel manchmal noch die alte Stimmung halten.** Edge-Case mit echter Tie: `[anger, joy]` (History 2 nach zwei Klassifikationen) vs `[joy, anger]`: vorwaerts gewinnt der erste (anger bzw. joy), rueckwaerts gewinnt der letzte (joy bzw. anger) — beide haben count=1 (`>` 0). Bei `[joy, anger]` rueckwaerts: anger ist juenger → State folgt der akuten Klassifikation, nicht dem Anfangsmoment. Weniger Flacker.
- **P-UI-10-Faustregel:** bei einer State-Machine, die ein Live-Signal interpretiert (Stimmung, Aktivitaet, Tonalitaet), tie-break auf das juengste Sample. Der State soll der aktuellen Realitaet folgen, nicht der historischen Mehrheit. Backstop: `test_majority_voter_iteriert_rueckwaerts` zementiert die Iteration mit explizitem `for (let i = history.length - 1; i >= 0; i--)`-Source-Check.

---

## Body-Anker `body { font-size: calc(<base> * var(--scale)) }` ist der saubere Pfad fuer "skaliere die UI, nicht den Content" (P-UI-9)

- **Patch-142-Versuch beide Variablen zu koppeln war ein latentes Anti-Pattern.** Das alte `applyUiScale(val)` setzte `--ui-scale` UND `--font-size-base = (16 * f) + 'px'` gleichzeitig. Folge: der User konnte die Bubble-Schriftgröße nicht unabhängig von der UI-Skalierung verändern, weil jeder Scale-Step beide CSS-Variablen tauschte. Plus: das `calc()`-Versprechen aus dem Patch-142-Kommentar ("Dank calc() in der CSS skalieren alle Texte/Buttons/Paddings anteilig") wurde im CSS nirgends eingelöst — keine einzige `calc(... * var(--ui-scale))`-Verwendung im Bestand. Die UI-Scale-Wirkung kam ausschließlich über die gekoppelte `--font-size-base`-Aenderung, was die Bubbles mitzog. **Lesson generalisierbar:** wenn ein Skalierungssystem zwei Klassen von Elementen unterscheiden soll (UI vs. Content), gehören BEIDE Klassen an explizite, getrennte CSS-Variablen. Der Body bekommt einen `calc()`-Anker auf eine davon (die "Default-Klasse" — typisch UI), der Content nutzt seine eigene Variable als `font-size: var(--<content-base>)` und ist damit unabhängig vom body-font-size.
- **Body-Anker plus em-Kette ist die billigste Wirkungskette.** P-UI-9-Implementation: ein einziger Edit am body-Block (`body { font-size: calc(15px * var(--ui-scale)); }`), und alle UI-Elemente die KEIN explizites `font-size` haben (Buttons, Sidebar-Items, Settings-Labels, Login-Form) skalieren ueber den em-Default mit. Bubbles und `#text-input` haben explizite `font-size: var(--font-size-base)` und sind damit unbetroffen — bewusst. Kein Refactoring von hunderten CSS-Regeln noetig. **Lesson:** ein Body-Schrift-Anker mit `calc()` ist der minimalinvasive Weg zu UI-Skalierung — vorausgesetzt der Content-Bereich hat eine eigene `font-size`-Variable (was hier gegeben war).
- **Backstop-Test gegen Anti-Pattern-Rueckfall.** `test_zwei_achsen_unabhaengig` prueft im `pui9_setUiScale`-Slice, dass `--font-size-base` UND `16 * f` NICHT vorkommen. Wenn ein zukuenftiger Refactor versehentlich die alte Kopplung wiedereinfuehrt (z.B. weil jemand "applyUiScale war ja vorher fertig"), schlaegt der Test sofort an. Plus symmetrisch `test_pui9_set_font_size_setzt_nur_font_size_base` (kein `--ui-scale` im FontSize-Setter). **Lesson:** fuer entkoppelte Achsen braucht es zwei symmetrische Anti-Pattern-Tests — pro Achse einen, der die andere ausschliesst.

---

## Backwards-Compat-Shims sind billig genug, dass sie sich gegen "Bestands-Tests anpassen"-Risiko lohnen (P-UI-9)

- **Zwei drei-Zeilen-Shims halten ein ganzes Bestands-Test-Set lauffaehig.** P-UI-9 koennte `applyUiScale`/`resetUiScale` komplett entfernen und alle bestehenden Tests aktualisieren (`test_settings_umbau::TestUiSkalierung` 5 Tests, `test_loki_mega_patch` Z. 681 ein Selektor). Stattdessen bleiben beide Funktionsnamen als duenne Shims:
  ```js
  function applyUiScale(val) { pui9_setUiScale(val); }
  function resetUiScale() { pui9_resetUiScale(); }
  ```
  Bestands-Tests, die auf `function applyUiScale` und `function resetUiScale` splitten, bleiben gruen — die ANTI-Pattern-Kopplung mit `--font-size-base` ist im Implementations-Pfad weggefallen, das Shim selber ist semantisch sauber. **Lesson generalisierbar:** wenn ein Patch ein Bestands-API umbaut (gleicher Name, anderes Verhalten), behalte den Namen als Shim — wenn das Shim selbst NICHT irrefuehrend ist (z.B. keine Side-Effects, die der Caller noch erwartet), kostet es nichts, reduziert die Diff-Groesse, und macht den Patch reviewbar.
- **Wann das Shim ENTFERNT werden muss.** Gegenbeispiel: P-UI-9 hat `setFontSize`/`resetFontSize`/`markActiveFontPreset` (Patch 86) komplett entfernt — keine Shims. Begruendung: diese Funktionen haben nicht nur die CSS-Variable gesetzt, sondern auch den `.font-preset-btn`-Klassen-State synchronisiert. Mit dem Removal des HTML-Buttons und CSS-Blocks (4-Preset-System) waere ein Shim irrefuehrend (`markActiveFontPreset` haette nichts mehr zum Markieren). Stattdessen Aufrufer direkt migriert: `resetTheme` ruft jetzt `pui9_resetFontSize`, `loadFav` ruft `pui9_setFontSize`/`pui9_resetFontSize`. **Faustregel:** Shim wenn der alte Name eine reine Setter-Funktion war und das neue Verhalten dieselbe Pre-/Post-Condition erreicht. Direkt-Migration wenn das alte Verhalten Side-Effects hatte, die im neuen System nicht mehr existieren (z.B. UI-State der entfernten Buttons).

---

## Pre-Render-IIFE-Doppelschreibung beim Skalierungs-Boot vermeidet sichtbares Snapping (P-UI-9)

- **Body-Schrift haengt von einer CSS-Variable ab → die muss VOR dem ersten Paint gesetzt sein.** Wenn ich nur die spaete `restoreUiScale`-IIFE haette (laeuft nach dem `<script>`-Block am Ende des Dokuments, also nach DOMContentLoaded), wuerde der Browser:
  1. Erst Paint mit Default `--ui-scale: 1` → body-font-size 15px → Buttons/Sidebar Default-Groesse
  2. Dann IIFE feuert → setzt `--ui-scale: 1.15` (z.B. aus localStorage) → body-font-size 17.25px → Layout-Snap
  Sichtbar als kurzes Flackern beim Reload. P-UI-9 fixt das mit einer zweiten Pre-Render-IIFE im `<head>`-Block (die laeuft VOR dem ersten Paint — sie ist Teil des HTML-Doku-Loads, nicht des Body-Scripts), die `nala_ui_scale` direkt aus localStorage liest und `--ui-scale` setzt. Erster Paint ist schon korrekt skaliert. **Lesson generalisierbar:** wenn ein State-Restore von einer CSS-Variable abhaengt, die `body { font-size: calc(...) }` (oder jede andere Layout-relevante Eigenschaft) mitkocht, IMMER zwei Restore-Punkte: einen pre-render fuer die CSS-Wirkung (ueblicherweise im `<head>`), einen post-render fuer alle abhaengigen UI-Elemente (Stepper/Slider/Indikatoren).
- **Beide IIFEs muessen den Favoriten-Pfad UND den Plain-Pfad lesen.** P-UI-9-Pre-Render-IIFE liest sowohl `fav.uiScale` (im aktiven-Favoriten-Pfad, falls `nala_last_active_favorite` gesetzt) als auch `nala_ui_scale` (im Plain-Pfad). Wenn nur ein Pfad behandelt wuerde, wuerde der andere mit Default `1` rendern und dann auf den richtigen Wert snappen. **Lesson:** State-Restore-Logik in zwei IIFEs muss in BEIDEN den vollen Lese-Pfad-Baum spiegeln — sonst gibt es einen Spezialfall mit Snapping. Symmetrie-Pruefung als Source-Audit-Test (`test_pre_render_iife_restoring_ui_scale` prueft beide Pfade).

---

## addMessage-internal-Hook vs. Caller-Side-Hook — wann welches Pattern (P-UI-8)

- **Beide Patterns halten die Funktions-Signatur invariant — die Wahl hängt davon ab, ob der Hook externe Daten braucht.** P-UI-7-Lesson dokumentierte: bei einem neuen Verhalten an einer DOM-Stelle, deren Wrapper aus `addMessage` kommt, ist der Caller-Side-Hook (Aufruf NACH `addMessage(reply, 'bot')` aus dem Caller wie sendMessage heraus) sauberer als der Callee-Side-Hook (vierter Param `modelId`), weil Source-Audit-Tests auf der addMessage-Signatur splitten. P-UI-8 ergänzt die Faustregel: **Caller-Side-Hook ist nur dann der saubere Pfad, wenn der Caller einzigartige Daten hat, die addMessage selbst nicht kennt** (z.B. `data.model` aus der Chat-Response bei P-UI-7). Wenn der Hook **keine externen Daten** braucht (P-UI-2 nur `text`+`sender`, P-UI-6 nur `text`, P-UI-8 nur den frisch erstellten `wrapper`), ist der **addMessage-internal-Hook am Ende der Funktion vor `return wrapper;`** kürzer und vollständiger: er deckt alle 5 Aufruf-Stellen automatisch ab (sendMessage, loadSession-Reload, late-replay, STT-Fehler-Bubble, Greeting-Bubble) ohne dass jeder Caller den Hook duplizieren muss. P-UI-7 konnte das nicht — der Caller hatte Daten, die addMessage nicht kennt. P-UI-8 kann es — alle Bot-Bubbles brauchen denselben Anchor unabhängig von der Aufruf-Quelle.
- **Source-Audit-Tests bleiben in beiden Patterns stabil — entscheidend ist die Signatur-Invariante.** Bei P-UI-7 splittete `_reply_model_block` auf `const replyModel =` (im Caller). Bei P-UI-8 splittet `_add_message_block` auf `function addMessage(text, sender, tsOverride)` und nimmt den ganzen Body bis `return wrapper;`. Beide funktionieren weil die Signatur unangetastet ist. **Lesson generalisierbar:** für jede DOM-Manipulation in Bot-Bubbles entscheide zuerst ob der Hook externe Daten braucht — Caller-Side bei "ja", Callee-Internal bei "nein". Beide halten die Signatur invariant; das Callee-Internal-Pattern ist einfacher zu warten weil weniger Aufruf-Stellen synchronisiert werden müssen.
- **Idempotenz via WeakMap macht den addMessage-internal-Hook robust.** `pui8_anchorMap = new WeakMap()` mappt botWrapperEl → anchorEl. Bei jedem Aufruf prüft `pui8_addAnchor(botWrapperEl)` zuerst `pui8_anchorMap.has(botWrapperEl)` und returnt vorzeitig wenn schon vorhanden. WeakMap statt Map weil: (1) automatische GC bei Wrapper-Removal — wenn die Bot-Bubble aus dem DOM entfernt wird (Chat-Wechsel via loadSession), wird der WeakMap-Eintrag automatisch geräumt. (2) Kein Memory-Leak bei langen Sessions. Bonus: WeakMap-Idempotenz schützt auch vor Doppel-Hooks die durch zukünftige Patches versehentlich entstehen könnten. **Lesson:** für DOM-Element → State-Element-Maps in Frontend-Code IMMER WeakMap statt Map verwenden — sie sind GC-freundlich und Element-Identität-basiert (Reference-Equality, kein Cleanup nötig).

---

## Body-Child-Position ist Pflicht für `position: fixed`-Layer auf transformed Containern (P-UI-8)

- **CSS-Spec-Detail (containing-block-Regel) wirkt auch in P-UI-8 — gleiches Pattern wie P-UI-4-Sidebar.** Bei `body.pui4-sidebar-open` aktiviert `.app-container { transform: translateX(280px); }`. Jedes `position: fixed`-Element innerhalb von `.app-container` würde durch den Transform mitgeshifted (CSS Transforms Module Level 1, Sektion 6.1: `transform` formt einen neuen containing block für fixed-positioned Descendants). P-UI-8 wollte die Scroll-Nav als linker fixed-Layer — wenn sie als `#chat-screen`-Descendant gebaut wäre, würde sie bei offener Sidebar mitshiften: aus `left: 6px` würde im Sichtbereich-Frame plötzlich `left: 286px` werden, weil der containing block sich verschoben hat. Lösung: Body-Child via `document.body.appendChild(navEl)` — Geschwister von `.app-container`, nicht Descendant. Body bleibt der initial containing block, Scroll-Nav bleibt Viewport-fixed.
- **Plus zusätzlicher CSS-State-Mutex via `body.pui4-sidebar-open .pui8-scroll-nav { display: none; }`.** Selbst wenn die Scroll-Nav als Body-Child korrekt Viewport-fixed bleibt, würde sie bei offener Sidebar links neben den 280-px-shifted Content schwebend sichtbar sein — semantisch falsch (User bedient gerade die Sidebar, nicht den Chat) und visuell unschön. Die CSS-Regel `body.pui4-sidebar-open .pui8-scroll-nav { display: none; }` blendet die Leiste komplett aus während der Sidebar-Open-Phase. Re-Erscheinen passiert beim nächsten Scroll-Event nach Sidebar-Close. **Lesson generalisierbar:** wenn ein Body-Child fixed-Layer mit einem anderen Body-Child fixed-Layer kollidieren könnte (Sidebar + Scroll-Nav), CSS-State-Mutex via `body.<state>` einbauen — billiger als IntersectionObserver oder JS-State-Sync, und automatisch synchron mit dem CSS-State der Vorgänger-Patches.

---

## `createElementNS` ist Pflicht für SVG-Inline-DOM, nicht `createElement('svg')` (P-UI-8)

- **Browser akzeptiert `createElement('svg')` ohne Crash — rendert aber nichts.** Erste Idee für die Bezier-Gabelung der Scroll-Nav-Anchor-Punkte war:
  ```js
  const svg = document.createElement('svg');
  svg.setAttribute('width', '16');
  svg.setAttribute('height', '10');
  // ... appendChild paths
  ```
  Browser akzeptiert das, parst es aber als HTML-Element (nicht als SVG). Der Container hat im DOM-Tree denselben Tag-Namen "svg", aber die Pfade werden nicht gezeichnet — der Container bleibt visuell leer. Kein Console-Error, kein Crash, einfach kein Rendering. Korrekt: `document.createElementNS('http://www.w3.org/2000/svg', 'svg')` — explizit den SVG-Namespace mitgeben. Dasselbe gilt für die `<path>`-Children. P-UI-8 nutzt eine Konstante `PUI8_SVG_NS = 'http://www.w3.org/2000/svg'` und der Helper `pui8_buildForkSvg(direction)` ruft `createElementNS(PUI8_SVG_NS, 'svg')` und `createElementNS(PUI8_SVG_NS, 'path')`.
- **Inline-SVG ist die richtige Wahl für theme-fähige, sehr kleine Vektor-Grafiken — `<img>` mit SVG-URL wäre verschwendet.** Die Fork-Gabelung ist 16x10 px klein und besteht aus 2 Bezier-Pfaden. Stroke-Color via `currentColor` (inheriting vom CSS-Parent), Fill via `fill: none` — beide CSS-driven. Mit `<img src="data:image/svg+xml,...">` wäre die Pfade-Geometrie statisch im URL-encoded data, Theme-Wechsel würde ein neues Render brauchen. Inline-SVG via DOM erlaubt CSS-driven Stroke-Color → Theme-Wechsel kostenlos. **Lesson generalisierbar:** für sehr kleine, theme-fähige Vektor-Grafiken (<50 Path-Punkte) ist Inline-SVG via `createElementNS` der saubere Pfad — `<img>` ist Overkill (Network-Request, Render-Pipeline-Overhead) und entkoppelt die Grafik vom CSS-Theming. Faustregel: alles unter 50 Pfad-Punkten ist Inline-SVG-Material.
- **Source-Audit-Test als Backstop gegen Namespace-Vergesslichkeit.** `test_build_fork_createelementns` prüft im pui8_buildForkSvg-Slice die Anwesenheit von `createElementNS(PUI8_SVG_NS`. Wenn ein zukünftiger Refactor-Patch versehentlich auf `createElement('svg')` zurückfällt (Copy-Paste, Refactor-Helper), schlägt der Test sofort an — der Patch-Reviewer sieht "createElementNS fehlt" und kann nachfragen. **Lesson:** für stille Failure-Modi (Namespace-Verwechslung, falsche Spelling, missing parameter) braucht es Source-Audit-Tests, weil Live-Verhalten nur unter bestimmten Render-Pfaden auffällt — manchmal erst Wochen später wenn jemand das erste Mal die SVG-betroffene UI öffnet.

---

## `onerror`-Fallback + localStorage-Cache verwandelt unsichere CDN-Bilder in robuste UI-Komponenten (P-UI-7)

- **Browser-`onerror` ist der saubere Fallback-Trigger für unsichere Image-URLs.** OpenRouter-CDN liefert Provider-Logos unter `https://openrouter.ai/images/icons/<provider>.svg`, aber nicht für jedes Provider-Kürzel — manche Modelle haben keinen Eintrag, andere wechseln Pfad-Konventionen. Die naive Lösung wäre eine kuratierte Whitelist im Frontend (`PUI7_KNOWN_PROVIDERS = ['deepseek', 'anthropic', 'openai', ...]`) — die altert schnell und muss bei jedem neuen Modell nachgepflegt werden. P-UI-7 macht es anders: jedes `<img>`-Element bekommt einen `addEventListener('error')`-Handler, der bei Lade-Fehler `pui7-icon-fallback`-Klasse setzt (CSS kippt Image versteckt → Badge sichtbar) und die fehlerhafte URL in localStorage als `'fail'` cacht. Der `addEventListener('load')`-Handler cacht erfolgreiche URLs als `'ok'`. Wenn die URL beim nächsten Render bereits als `'fail'` gecacht ist, wird das Image-Element gar nicht erst erzeugt — direkt Badge. **Lesson generalisierbar:** bei externen Bild-Quellen mit unklarer Coverage ist ein `<img>` mit `onerror`-Fallback + Persistenz-Cache der saubere Ersatz für eine zentrale Whitelist. Der Cache wächst organisch mit der tatsächlichen User-Nutzung, statt eine veraltende Liste zu pflegen. Plus: pro fehlerhafte URL feuert exakt ein einziger Network-Roundtrip pro Browser-Session — danach Cache-Hit, kein Retry-Storm.
- **Cache-Persistenz pro URL, nicht pro Provider-Slug.** Erster Reflex: Cache-Key = Provider-Slug (`pui7_iconCache['deepseek'] = 'ok'`). Problem: wenn die Pfad-Konvention wechselt (z.B. von `/images/icons/<provider>.svg` zu `/icons/<provider>.png`), bleibt der alte Slug-Cache-Eintrag stehen und versteckt das Update. Mit URL-als-Key (`pui7_iconCache['https://openrouter.ai/images/icons/deepseek.svg'] = 'ok'`) wird ein Pfad-Wechsel zur neuen URL → kein Cache-Eintrag → frischer Versuch. **Lesson:** Cache-Keys sollten die volle Quelle adressieren, nicht eine Abstraktion darüber — sonst wird der Cache zur Falle bei Quell-Wechseln.

---

## Hierarchische Hooks ohne Signatur-Änderung halten Source-Audit-Tests stabil (P-UI-7)

- **Caller-Side-Hooks > Callee-Side-Hooks bei Tests, die an Funktions-Signaturen splitten.** Die intuitivste Architektur für UI-7 wäre ein vierter Param `modelId` an `addMessage(text, sender, tsOverride, modelId)` — Caller übergibt das Modell, addMessage hängt das Icon im Bot-Branch an. ABER: P-UI-6-Tests splitten exakt auf `function addMessage(text, sender, tsOverride)` (`src.split("function addMessage(text, sender, tsOverride)", 1)[1]`) — eine Signatur-Erweiterung würde 7 Tests in `TestAddMessageHook` brechen. Workaround "Tests anpassen" hieße: jedes Mal wenn ein neuer Patch der addMessage-Funktion einen Param dazustellt, muss der Slice-String in den Tests aller vorigen Patches mitwachsen. Das ist ein Wartbarkeits-Albtraum. P-UI-7 löst es anders: separater `pui7_attachLLMIcon(wrapperEl, modelId)`-Helper, der **nach** `addMessage(reply, 'bot')` aus dem Caller heraus gerufen wird. addMessage-Signatur bleibt invariant, alle vorigen Tests stabil, der Helper ist idempotent (mehrfacher Aufruf ist no-op via "alte Icons entfernen vor Insert"). Bonus: UI-11 kann später denselben Helper bei Reasoning-Switch einfach erneut rufen, ohne addMessage anzufassen. **Lesson generalisierbar:** wenn Source-Audit-Tests an einer Funktions-Signatur splitten und ein neuer Patch zusätzliches Verhalten an dieselbe DOM-Stelle koppeln will, ist der Caller-Side-Hook (vom Aufrufer aus) sauberer als der Callee-Side-Hook (zusätzlicher Param) — die Signatur bleibt invariant, die Tests bleiben stabil, der Hook wird mehrfach-aufrufbar.
- **Idempotenz im Hook macht ihn future-proof.** `pui7_attachLLMIcon` macht zuerst `msgDiv.querySelector('.pui7-llm-icon')` und entfernt vorhandene Icons, bevor das neue eingefügt wird. Das wirkt im normalen send-Flow überflüssig (es gibt nie ein altes Icon, weil die Bubble frisch erzeugt wurde) — aber für UI-11 wird's relevant: wenn der Intent-Router mid-conversation auf ein anderes Modell switcht und das Icon wechselt, kann derselbe Helper einfach erneut gerufen werden. Mehrfach-Aufrufe sind no-op-tolerant. **Lesson:** Hook-Helper, die DOM-Elemente an existierende Wrapper anhängen, sollten idempotent sein — selbst wenn der erste Aufrufer das Element nur einmal ruft. Das schützt vor Double-Render-Bugs und macht den Helper für Folge-Patches wiederverwendbar.

---

## Inline-flex + `flex: none` löst das "Icon vor Inline-Element"-Problem ohne `position: absolute` (P-UI-7)

- **Wenn ein neues Inline-Element vor einem bestehenden Inline-Element platziert werden soll, ist `display: inline-flex` + `flex: none` der saubere Pfad.** P-UI-2 setzt `.bubble-collapse-toggle` als `display: inline-block; margin-right: 6px;` — das Element sitzt im natürlichen Flow als erstes Child von `msgDiv`. Für UI-7 wollte ich das Icon **vor** dem Toggle haben, ohne den Toggle anzufassen. Erste Idee: `position: absolute; top: 6px; left: 6px;` für das Icon, Toggle dann `left: 36px;`. Problem: zwei absolute Positionen wären gegenseitig blind und brechen sofort, wenn Padding/Spacing der Bubble sich ändert (z.B. UI-9 2-Achsen-Skalierung wird padding auf `calc(var(--ui-scale) * 6px)` umstellen). Lösung: `display: inline-flex` + `flex: none` + `margin-right: 6px` macht das Icon zu einem **Inline-Container mit fester Größe**, der im natürlichen Flow vor dem Toggle landet — ohne absolute Positionierung, ohne Layout-Brüche. `flex: none` verhindert Schrumpfen bei beengten Bubbles (sonst würde flex-shrink 1 das Icon kompromittieren). **Lesson generalisierbar:** wenn ein neues Inline-Element vor einem bestehenden Inline-Element platziert werden soll, ist `display: inline-flex` + `flex: none` der saubere Pfad — `position: absolute` ist eine Coupling-Falle und sollte nur für tatsächlich überlappende Layer (Tooltips, Popovers, Modal-Overlays) verwendet werden. Der Inline-Flow ist robust gegen Skalierungs- und Padding-Änderungen.
- **`object-fit: contain` für Provider-Logos respektiert Aspect-Ratio.** Provider-Logos kommen in unterschiedlichen Aspect-Ratios (manche quadratisch, manche breit). `<img>` mit `width: 100%; height: 100%` ohne `object-fit` würde manche Logos verzerrt strecken. `object-fit: contain` skaliert das Logo so, dass es vollständig in die 22-px-Box passt, ohne die Proportionen zu ändern — eventuell mit Letterboxing oben/unten. Für ein ICON-Element ist das das richtige Verhalten (besser ein bisschen Whitespace als verzerrte Logos). **Lesson:** Bei Image-Containern mit fester Größe (Icons, Avatare) immer `object-fit: contain` (oder `cover` wenn Crop akzeptabel) explizit setzen — Default `fill` verzerrt, ist fast nie das gewollte Verhalten.

---

## Disjunkter CSS-Klassen-Namespace verhindert Kollision mit ähnlich benannten Bestands-Klassen (P-UI-6)

- **Wenn zwei Patches semantisch verwandte UI-Elemente bauen, lieber Namespace-Präfix als Suffix-Variation.** Phase-5a-P213 hat eine Reasoning-Stepper-Card mit `.reasoning-card`/`.reasoning-toggle`/`.reasoning-list`/`.reasoning-step` etc. — sie zeigt Pipeline-Schritte (Polling von `/v1/reasoning/poll` während des LLM-Calls), eigene CSS-Animation, eigener DOM-Lebenszyklus. P-UI-6 baut die UI für den tatsächlichen Modell-Reasoning-Output (`<think>`-Block aus DeepSeek R1, OpenRouter-Reasoning) — semantisch ähnlich (Reasoning-Block mit Toggle), aber eine andere Datenebene. Statt `.reasoning-block`/`.reasoning-toggle-2` (was sofort verwirrt) wurde der Namespace explizit `pui6-reasoning-...` gewählt: `.pui6-reasoning-block`, `.pui6-reasoning-toggle`, `.pui6-reasoning-content`, `.pui6-reasoning-arrow`, `.pui6-reasoning-label`. Plus `data-pui6-reasoning="1"`/`data-pui6-toggle="1"` als Audit-Hooks für Tests/DevTools. **Lesson generalisierbar:** wenn ein neuer Patch in derselben Datei ein UI-Element baut, das thematisch einem Bestands-Element ähnelt, ist Namespace-Präfix die saubere Lösung — das Source-Diff-Auditing wird trivial (alle Selektoren mit `pui6-`-Präfix gehören zum Patch), Anti-Pattern-Tests können explizit prüfen dass die alten Klassen nicht im neuen Block auftauchen. Suffix-Variation (`-2`/`-new`/`-v2`) ist nichtssagend und altert schlecht.
- **Source-Audit-Test als Backstop gegen Namespace-Verwischung.** `test_pui6_namespace_disjunkt_zu_p213` in `test_p_ui_6_reasoning_block.py` slice't den CSS-Block ab `.pui6-reasoning-block {` und prüft, dass im 1500-Zeichen-Slice keine der P213-Klassen (`reasoning-card`, `reasoning-step`, `reasoning-list`, `reasoning-toggle`) als Selektor-Einleitung steht. Wenn ein zukünftiger Patch versehentlich beide Namespaces vermischt (Copy-Paste, Refactor), schlägt der Test sofort an. **Lesson:** für Patches mit Namespace-Disziplin braucht es einen Anti-Pattern-Test, der genau die Kollisions-Pattern ausschließt — nicht nur den Positiv-Selektor.

---

## Frontend-Extraktion via Regex statt Backend-Schema erweitern, wenn die Information schon im Datenstrom mitkommt (P-UI-6)

- **Defensive Input-Handling statt Schema-Annahmen.** `pui6_extractReasoning(text)` prüft `!text || typeof text !== 'string'` und liefert `{ reasoning: '', content: text || '' }` — kein Throw, keine Annahme dass text immer ein String ist. Beim Match wird der `reasoning`-String getrimmt (sonst klebt der Newline vor/nach `<think>` mit), und der `content` wird ebenfalls von führenden/abschließenden Whitespaces befreit (sonst startet die Antwort mit einem leeren Absatz). **Lesson:** Frontend-Extraktoren müssen defensive Inputs handhaben, weil sie aus mehr Pfaden gerufen werden als ein Backend-Schema-Feld — der Roh-Text kommt aus User-Input, LLM-Output, Replay, Re-Render etc.

---

## Defensive `stopPropagation` an Toggle-Buttons innerhalb klick-aktiver Wrapper ist Pflicht (P-UI-6)

- **Inner-Element-Klick kaskadiert durch alle Listener-Layer der Bubble.** Der `.pui6-reasoning-toggle`-Button sitzt innerhalb der `.bubble-full`/`.message`-Bubble. Diese Bubble hat mehrere Click-Listener: P139-Action-Toolbar-Toggle (`attachActionToggle` macht 5 s-Sichtbarkeit auf Tap), P-UI-2-User-Bubble-Auto-Expand (capture-phase-Listener, der `collapsed-v2` toggelt). Ohne `ev.stopPropagation()` im Click-Listener des Reasoning-Toggles würde ein Klick auf "Gedankengang" GLEICHZEITIG die Action-Toolbar einblenden und (bei Bubbles mit collapsed-v2) das Auto-Collapse togglen. Visuell unschön, semantisch falsch. P-UI-6 fügt `ev.stopPropagation()` als erste Anweisung im Click-Handler ein — das verhindert das Bubbling. **Lesson generalisierbar:** jeder Click-Listener auf einem Inner-Element innerhalb einer click-aktiven Bubble braucht `stopPropagation` — sonst kaskadiert der Click durch die ganze Listener-Kette. Pattern aus P-UI-5 wiederverwendet: dort macht der `pui5_buildProjectCardHtml`-Pin-Button-onclick auch `event.stopPropagation()` aus exakt dem gleichen Grund.
- **Source-Audit-Test als Backstop für `stopPropagation`-Forderung.** `test_build_reasoning_block_toggle_click_listener` in `test_p_ui_6_reasoning_block.py` prüft drei Anweisungen im pui6_buildReasoningBlock-Slice: `addEventListener('click'`, `stopPropagation`, `classList.toggle('pui6-collapsed')`. Wenn ein zukünftiger Patch das `stopPropagation` versehentlich entfernt (Refactor, Listener-Konsolidierung), schlägt der Test sofort an — und der Patch-Reviewer kann nachfragen, warum die Schutzschicht weg sein soll. **Lesson:** sicherheitskritische Handler-Anweisungen (stopPropagation, preventDefault, defensive guards) brauchen Source-Audit-Tests, weil sie in der Live-Demo nur unter bestimmten Klick-Pfaden auffallen.

---

## Eine View als Geschwister im transformed Container shiftet automatisch mit — kein Extra-Setup (P-UI-5)

- **Layout-Modi auf Container-Ebene gelten kostenlos für alle Geschwister-Screens.** P-UI-4 hatte `body.pui4-sidebar-open .app-container { transform: translateX(280px); }` als Trigger für den Container-Shift bei offener Sidebar — ursprünglich für `#chat-screen`. P-UI-5 fügt `#projects-screen` als Geschwister im selben `.app-container` hinzu — und der Shift greift automatisch, weil das Layout-Child mitgeshifted wird. Kein eigener Sidebar-Open-State-Listener nötig, kein doppelter Transform, kein z-Index-Fix. Die Geometrie kaskadiert. **Lesson generalisierbar:** wenn ein Container-Modi auf Layout-Ebene wirkt (Transform, Filter, Opacity, Visibility), gelten die Modi für alle Layout-Children — neue Screens lassen sich als Geschwister einfügen ohne Refactor des Modi-Systems. Das ist auch der Grund warum Spec 13 explizit "eigene View" sagt und nicht "Modal" — Modal wäre nicht im `.app-container`, würde nicht mitgeshifted.
- **Source-Audit-Test zementiert die Geschwister-Position.** `test_projects_screen_steht_im_app_container` in `test_p_ui_5_project_view.py` prüft dass `<div id="projects-screen">` VOR der Sidebar-DIV (body-Child) im Source steht — das ist die Markup-Form der Geschwister-Beziehung im `.app-container`. Wenn ein zukünftiger Patch die View versehentlich nach außen verschiebt, schlägt der Test an. **Lesson:** Layout-Beziehungen die per Markup-Position kodiert sind, brauchen Source-Audit-Tests — sonst kann ein DOM-Move sie unbemerkt brechen.

---

## Defensive `typeof === 'function'`-Checks an Cross-Module-Render-Calls vermeiden Init-Order-Probleme (P-UI-5)

- **Setter-Funktionen werden aus mehreren Pfaden gerufen, nicht alle haben die Render-Targets im DOM.** Die alten `renderNalaProjectsList`/`renderNalaProjectsActive`-Aufrufe in `setActiveProject`/`clearActiveProject` waren globale Calls ohne Defensive — wenn die Render-Targets nicht existieren (User noch nicht zur View navigiert, `handle401` mitten im Login feuert, Init-Reihenfolge anders als erwartet), warf der DOM-Zugriff. P-UI-5 ersetzt das durch `if (typeof renderProjectsView === 'function') { try { renderProjectsView(); } catch (_) {} }` — der Aufruf ist no-op-tolerant, der Cache wird trotzdem aktualisiert. **Lesson generalisierbar:** bei jedem Aufruf einer Render-Funktion aus einer Setter/Mutator-Funktion (statt aus einem Init-Hook) Defensive-Checks vorschalten. Setter werden oft aus verschiedenen Init-Phasen gerufen (init, login, logout, refresh, error-recovery), und die Render-Targets existieren nicht überall. `typeof === 'function'` ist eine billige Prüfung; `try/catch` schützt vor DOM-Zugriff-Fehlern beim Render-Body.
- **Defensive Hide bei Auth-Pfaden ist mehr als ein nice-to-have.** Bei `doLogout` und `handle401` muss `#projects-screen` explizit auf `display: none` gesetzt werden — sonst hängt die View nach Logout sichtbar über dem Login-Screen. Der Login zeigt sich, aber die Project-View liegt darüber im Layer-Stack. Pattern: `const _projEl = document.getElementById('projects-screen'); if (_projEl) _projEl.style.display = 'none';` an jedem Auth-Reset-Pfad einfügen. **Lesson generalisierbar:** wenn ein neuer Screen eingeführt wird, der kein implizites "show only one"-Modell hat, müssen ALLE Screen-Reset-Pfade um den neuen Screen erweitert werden (`doLogout`, `handle401`, `showChatScreen` falls relevant). Sonst lebt der Screen weiter sichtbar in einem inkonsistenten App-State.

---

## `autouse=True`-Fixtures koppeln Source-Audit-Tests an Backend-Setup (P-UI-5)

- **Source-Audit-Tests brauchen kein DB/Config — autouse-Fixture macht's trotzdem.** `test_nala_projects_tab.py` hatte einen `_disable_auto_template`-Fixture mit `autouse=True`, der `cfg.get_settings()` ruft und dabei `config.yaml` lädt. Im Worktree-Setup ist `config.yaml` gitignored, also wird der Loader feuert mit `FileNotFoundError`. Ergebnis: alle 18 reinen HTML/JS-Source-Audit-Tests fielen als ERROR aus, obwohl sie nur den NALA_HTML-String brauchen, nicht die Settings. Die Endpoint-Tests brauchen die Settings tatsächlich — aber nur die. **Lesson generalisierbar:** `autouse=True` ist nur sicher für Fixtures, die garantiert auf jedem Test-Pfad funktionieren (logging, monkeypatch von Standard-Modulen, in-memory-Setup). Sobald ein Fixture externe Files/Configs/DBs lädt, wird er Pflicht-Fixture mit explizitem Param — die Tests, die ihn brauchen, fordern ihn an, die anderen laufen ohne. Vermeidet Pre-existing-Failures auf Tests, die in Wahrheit gar nicht vom Setup abhängen.
- **Pre-existing config.yaml-Drift im Worktree ist Memory-bekannt.** Memory `config_yaml_gitignored` dokumentiert die Drift. Nala-Endpoint-Tests in `test_nala_projects_tab` (TestNalaProjectsEndpoint) bleiben ERROR im Worktree, weil sie `tmp_db` brauchen, das config.yaml liest. Das ist nicht durch P-UI-5 verursacht — der Endpoint selbst ist unangetastet. **Lesson:** bekannte Setup-Drift-Failures dokumentieren (HANDOVER + Memory) statt zu verstecken — Verifizierung mit `git stash; pytest; git stash pop` zeigt zuverlässig ob ein Failure pre-existing ist.

---

## `transform` auf einem Vorfahren formt einen neuen containing block fuer fixed-positioned Descendants (P-UI-4)

- **CSS-Spec-Detail mit grosser Auswirkung**: jeder Vorfahre mit `transform`/`filter`/`perspective` ungleich `none` wird zum containing block fuer all seine fixed-positioned Descendants — nicht mehr der initial containing block (Viewport). CSS Transforms Module Level 1, Sektion 6.1. P-UI-4 wollte `.app-container` per `transform: translateX(280px)` shiften und die `.sidebar` (`position: fixed; left: -300px`) sass darin. Folge: Sidebar wuerde mit dem Container shiften — exakt entgegen der Sidebar-Mechanik (sie soll Viewport-bound bleiben). Loesung war kein neues CSS, sondern DOM-Refactor: Sidebar als body-Child platzieren (Geschwister von `.app-container`). **Lesson generalisierbar:** vor jedem `transform`-Edit auf Container-Element die fixed-positioned Descendants identifizieren — wenn welche darunter sind, entweder DOM-Refactor (Descendant raushebeln) oder containing-block-aware-Variante (z.B. `margin-left` statt `transform`, was kein neues containing block formt). Falls man nicht refactor will, ist `margin-left` der Plan-B — ABER auf Mobile-First-Layouts bricht das die `width: 100%`-Containment-Logik (Container wird breiter als Viewport, Children rendern sich nach der schmaleren Container-Breite statt off-screen-rechts zu laufen). Auf Mobile ist `transform` der einzige saubere Weg fuer "Content rutscht nach rechts und wird teilweise off-screen sichtbar".
- **Source-Audit-Test als Backstop gegen DOM-Move-Rueckfall.** `test_sidebar_steht_ausserhalb_chat_screen` in `test_p_ui_4_sidebar_content_shift.py` prueft drei Bedingungen: Sidebar-DIV-Position liegt VOR `<!-- ...Easter-Egg-Overlay`, NACH dem .app-container-Close-Marker, und der app-container-Close-Marker liegt zwischen chat-screen-Open und Sidebar. Wenn ein zukuenftiger Patch versehentlich die Sidebar wieder in `#chat-screen` verschiebt (Copy-Paste aus altem Code, Merge-Konflikt, Refactor), schlaegt der Test sofort an. **Lesson:** DOM-Position-Invarianten brauchen Source-Audit-Tests genauso wie CSS-Selektor-Invarianten — beides kann durch Refactor unbeabsichtigt brechen, beides ist nicht im Live-Verhalten sichtbar bis das transformations-Detail wirkt.

---

## Capture-phase Listener auf `document` ersetzt Backdrop-Click-Trap (P-UI-4)

- **Backdrop-Element ist Backdrop UND Click-Trap in einem.** Das alte `.overlay`-Element war ein vollflaechiges fixed-positioned DIV mit `onclick="toggleSidebar()"` — schickt jeden Click ausserhalb der Sidebar an den Sidebar-Toggle. Mit Content-Shift-Mechanik (Spec sagt KEIN Overlay) ist das Backdrop weg, aber die Click-Outside-Erkennung muss erhalten bleiben. Loesung: `document.addEventListener('click', handler, true)` (capture-phase, drittes Argument `true`). Capture-phase feuert auf jeden Click vor allen anderen Listenern, inklusive Bubble-phase auf den shifted Children. Mit `event.preventDefault() + event.stopPropagation()` konsumiert der Listener den Click — keine versehentliche Aktion auf dem shifted Content. Hamburger-Whitelist via `event.target.closest('.hamburger')` (sonst toggelt der Hamburger-onclick gleichzeitig ZUSAETZLICH, Bilanz wäre no-op).
- **Whitelist-Pattern fuer Click-Outside-Listener.** Der document-level Listener muss zwei Faelle ausschliessen: (1) Click innerhalb der Sidebar selbst (`sidebar.contains(event.target)`), und (2) Click auf das toggling-Element (Hamburger). Fuer (2) reicht `closest('.hamburger')` weil der Hamburger ein einzelnes Element mit eindeutiger Klasse ist. Bei mehreren toggle-faehigen Elementen waere eine Liste der Klassen sauberer. **Lesson generalisierbar:** Click-Outside-Listener brauchen IMMER eine Whitelist mit (a) dem zu schuetzenden Element selbst und (b) dem oder den Trigger-Elementen die selbst togglen koennen — sonst doppel-toggle. Auf capture-phase + preventDefault + stopPropagation gehoert ein Test der prueft dass der Listener den Event konsumiert (Source-Audit `test_outside_click_konsumiert_event`).

---

## DOM-Move ist transparenter Refactor wenn die Bestands-Selektoren ID-basiert sind (P-UI-4)

- **ID-Selektoren sind position-unabhaengig, klassen-basierte Ancestor-Pfade nicht.** Der HTML-Move der Sidebar von `#chat-screen`-innen nach body-Child funktioniert ohne weitere Aenderungen, weil ALLE Bestands-Listener `getElementById('sidebar')`, `getElementById('archive-search')`, `getElementById('pinned-list')`, `getElementById('session-list')` etc. nutzen. Plus die CSS-Definition fuer `.sidebar`/`.sidebar-actions`/`.sidebar-footer` ist via Klassen-Selektor ohne Ancestor-Pfad — funktioniert auch nach dem Move. Vor dem Move per Grep verifiziert dass kein `#chat-screen .sidebar`/`#chat-screen #sidebar`/`closest('.app-container')` etc. im Source vorkommt. **Lesson generalisierbar:** bei DOM-Move-Refactors immer zuerst per Grep auf Ancestor-relative Selektoren pruefen (z.B. `#parent .child`, `parent > .child`, `closest('.ancestor')`, `:scope > .descendant`). ID-basierte Selektoren (`#child`) sind transparent fuer DOM-Position, klassen-basierte sind es nur wenn der Selektor selbst keinen Ancestor-Pfad enthaelt. Falls ja: entweder Selektor refactoren oder DOM-Move verwerfen.
- **`querySelectorAll` ist genauso position-unabhaengig wie getElementById, wenn der Selektor unconditioned ist.** Im JS-Code wird haeufig auch `querySelectorAll('.sidebar-action-btn')` etc. genutzt — diese funktionieren nach dem DOM-Move trotzdem, weil der Selektor `.sidebar-action-btn` keinen Ancestor-Constraint hat. Erst wenn jemand `document.querySelectorAll('.app-container .sidebar')` schreiben wuerde, waere der Selektor brittle. **Lesson:** beim Code-Audit fuer DOM-Move-Refactors auch `querySelector*` mit Ancestor-Pfad pruefen (z.B. `'.parent .child'` oder `'#parent .child'`).

---

## CSS-only fuer Hoehen-Wechsel ist stabiler als JS-getriebene autoresize-Logik (P-UI-3)

- **Wenn ein UI-Element zwischen zwei Hoehen-Zustaenden wechselt und beide CSS-driven beschrieben werden koennen, ist CSS-only der stabilste Pfad.** Die Patch-67-autoresize-Logik im Nala-Eingabefeld setzte `Math.min(textInput.scrollHeight, 140)` an vier Stellen (input-Listener, focus-Listener, editMessage, fullscreenClose) und schrieb `style.height` als Inline-Wert. Inline-Style hat hoehere Spezifitaet als CSS-Klassen — die schoene CSS-Transition feuerte gar nicht, weil JS jedes Mal direkt setzte. P-UI-3 entfernte alle vier Inline-Setter und zentralisierte die Hoehe in `#text-input` (default 44px) und `#text-input:focus` (33dvh). CSS-Transition feuert jetzt zuverlaessig, der Code ist viel kuerzer. **Lesson generalisierbar:** wenn ein Default-Zustand und ein State-Zustand beide CSS-driven beschreibbar sind (Pseudo-Klasse `:focus`/`:hover`/`:checked`, oder eine State-Klasse die per JS toggelt), ist CSS die alleinige Hoehen-Quelle. JS-Inline-Setter sind versteckte Schalter, die jeder Refactor-Schritt aus Versehen rausreissen oder unbeabsichtigt brechen kann
- **Source-Audit-Test als Backstop gegen Rueckfall.** `test_keine_alte_autoresize_logik` in `test_p_ui_3_textarea_collapse.py` prueft drei Anti-Pattern-Strings: `Math.min(textInput.scrollHeight, 140)`, `Math.max(textInput.scrollHeight, 96)`, und `textInput.style.height` (egal in welchem Kontext). Wenn ein zukuenftiger Patch versehentlich einen Inline-Setter wieder einfuehrt (Copy-Paste, Merge-Konflikt, Refactor), schlaegt der Test sofort an. **Lesson:** wenn du ein Anti-Pattern aus dem Code entfernst, einen Source-Audit-Test schreiben der das Anti-Pattern NICHT mehr findet — pro entferntem Pattern ein assert-not-in. Nicht nur eine Variante (`textInput.style.height = '...'`) sondern alle Schreibweisen (`textInput.style.height` als globaler Substring)

---

## Klassen-basiertes Disabled ist orthogonal zu native `disabled`-Attribut (P-UI-3)

- **Zwei Disabled-Mechanismen am selben Button koennen koexistieren — aber muessen explizit synchronisiert werden.** Send-Button hat im Nala-Frontend zwei Disabled-Pfade: (1) `lockInput()` setzt `sendBtn.disabled = true` plus inline `opacity: 0.5` waehrend des Sendens — Native-Browser-Disabled, blockt Click + Tab-Focus + Form-Submit. (2) `pui3_updateSendBtnState()` togglet `.send-btn.pui3-empty` je nach Eingabefeld-Inhalt — CSS-Pseudo-Disabled via `pointer-events: none` + `opacity: 0.4`. Beide koennen gleichzeitig aktiv sein (Send laeuft + Eingabefeld leer), und das ist OK. ABER: nach `unlockInput()` (Send fertig oder abgebrochen) wird inline-opacity geleert — der Native-Disabled ist weg, aber die Klasse ist noch der Klassen-Stand vor dem Send. Wenn der User waehrend des Sendens nichts geaendert hat, ist das OK — ABER falls er getippt hat (`textInput.disabled = true` blockt die Eingabe nicht in allen Browsern), kann der Text-State driften. Loesung: `unlockInput()` ruft `pui3_updateSendBtnState()` am Ende auf — Klasse synchronisiert mit aktuellem Inhalt. **Lesson generalisierbar:** wenn zwei Disabled-Mechanismen am selben Button koexistieren, beide explizit synchronisieren bei Lock-Aufhebung — nicht annehmen, dass einer den anderen "von alleine" trifft. Ein einzelner Source-Audit-Test (`test_unlock_input_ruft_helper`) zementiert die Synchronisation
- **`pointer-events: none` + `opacity: 0.4` als CSS-Pseudo-Disabled-Pattern.** Native `disabled` setzt im Browser per Default `pointer-events: none`, taucht den Button optisch ab und entfernt ihn aus Tab-Focus-Reihenfolge. Eine CSS-Klasse die das gleiche Verhalten via `pointer-events: none + opacity` simuliert ist dann sinnvoll, wenn ein Button NICHT permanent disabled ist (Send waehrend Sending) sondern abhaengig von State (leer vs. Inhalt) — `disabled` ist binaer und steht im DOM-Attribut, das ist umstaendlich zu togglen. CSS-Klasse plus `classList.add/remove` ist eleganter. ABER `disabled = false` plus `.pui3-empty` heisst der Button ist DOM-aktiv aber CSS-pseudo-tot — also Tab-Focus klappt, das ist semantisch nicht ganz sauber. Pragmatisch: fuer Mobile + Touch-First-Apps ohne Tab-Focus ist das OK. **Lesson:** CSS-Klassen-disabled ist ein State-Toggle-Pattern, nativ-disabled ist ein DOM-Attribut. Beide haben unterschiedliche Semantik — fuer State-driven Disabled die Klasse, fuer "echt aus der Tab-Reihenfolge raus" das Attribut

---

## `scrollTop = scrollHeight` macht im 44px-Collapsed-Textarea die letzte Zeile sichtbar (P-UI-3)

- **Browser-Textarea respektiert `scrollTop` auch wenn `overflow: hidden`.** Spec verlangte: das collapsed Eingabefeld zeigt die letzte eingegebene Zeile (nicht die erste). Browser-Textarea scrollt ohne Hilfe nicht zur letzten Zeile, wenn der Cursor woanders steht — die Default-Scroll-Position bleibt am Anfang. Ein einzelnes `textInput.scrollTop = textInput.scrollHeight` im Blur-Handler setzt den Scroll-Container ans Ende des virtuellen Inhalts. Mit `white-space: nowrap; text-overflow: ellipsis; overflow: hidden` wird der Rest des Inhalts sauber abgeschnitten — User sieht die letzte Zeile als Preview, der Anfang ist truncated. **Lesson generalisierbar:** Browser-`<textarea>`-Elemente speichern `scrollTop` intern, auch wenn `overflow: hidden` ist — die Scroll-Position-Wert wird bei `overflow: visible/scroll`-Wechsel wieder angewendet. Pragmatischer Trick um Truncation-Verhalten an Position-Praeferenzen anzupassen
- **Try/catch um den Scroll-Setter, weil Browser-Edge-Cases.** `textInput.scrollTop = scrollHeight` kann in seltenen Faellen werfen (Detached Elements, defektes Layout-Engine, sehr alter Browser). Wrapping in `try { ... } catch (_) {}` macht den Blur-Handler robust gegen unerwartete Browser-Situationen. **Lesson:** bei DOM-Property-Settern, die in 99% der Faelle stumm passen, aber in 1% werfen, ist try/catch billiger als sich Browser-Mismatch-Bugs zu fangen. Anti-Pattern: blanker Setter ohne Schutz — wenn der Bug spaeter auftritt, faellt er stumm in einer beliebigen UI-Aktion auf

---

## Newline-Strings in NALA_HTML brauchen `\\n`, nicht `\n` (P-UI-2)

- **`NALA_HTML` ist ein Python-Triple-String ohne `r`-Prefix — `\n` darin wird beim Modul-Load zu echtem Newline-Char.** Erste P-UI-2-Variante hatte `text.split('\n')` direkt im Inline-JS — Python machte daraus beim Auswerten ein echtes Newline-Zeichen mitten im JS-String, wodurch die JS-String-Literale ungeschlossen blieben. Der `node --check`-Test (P203d3 `TestJsSyntaxIntegrity`) fing das ab: „Inline-Block hat Syntax-Fehler — Invalid or unexpected token". **Lesson generalisierbar:** jeder JS-Inline-Block in einem Python-`"""`-String muss `\n`/`\t`/`\r` als `\\n`/`\\t`/`\\r` schreiben (Backslash escapen). Gilt für `text.split('\\n')`, `text.indexOf('\\n')`, Regex `/\\n+$/` etc. Der `node --check`-Test ist der Backstop ohne Browser-Roundtrip — vor Commit immer durchlaufen lassen
- **`r"""..."""`-Strings wären eine Alternative, aber kollidieren mit Python-Backslash-Escapes anderswo.** Wenn man `NALA_HTML` als Raw-String definiert, würden alle `\n` literal sein — bequemer für JS, aber Python-Sprach-Strings im selben Block (`"ä"`, `\\` für Pfade) bräuchten dann andere Schreibweisen. Pragmatisch besser: bei der bestehenden Triple-String-Konvention bleiben und JS-`\n` doppelt escapen. **Lesson:** bei String-Mehrsprachigkeit (Python-File enthält JS-Code) eine Konvention für die Container-Sprache (Python) wählen und die Gast-Sprache (JS) konsequent escapen — nicht zwischen Modi wechseln
- **Source-Audit ist nicht ausreichend für JS-Syntax — `node --check` braucht's zusätzlich.** Mein Test `test_collapse_thresholds_im_source` prüft, dass `PUI2_COLLAPSE_CHAR_THRESHOLD = 160` im Source steht — aber nicht, ob der umgebende JS-Block parsen kann. P203d3 `TestJsSyntaxIntegrity::test_alle_inline_scripts_parsen` extrahiert alle inline `<script>`-Bodies und ruft `node --check` darauf — das ist die einzige Backstop-Stelle die Python-Escape-Bugs in inline-JS findet. **Lesson:** für jede Sprache die in einer anderen Sprache eingebettet ist, einen sprachspezifischen Parser-Test haben. Source-Audit auf String-Existenz reicht nicht, wenn die String-Boundaries selbst kaputt sein können

---

## Capture-phase-Listener priorisiert vor existierenden Bubble-Listeners (P-UI-2)

- **Bei mehreren Listenern auf demselben Element ist die Reihenfolge nicht definiert.** Mein P-UI-2 User-Tap-Listener (Auto-Collapse → expand) und P139 `attachActionToggle` (Tap → Action-Toolbar 5s) hängen beide an `msgDiv`. Wenn beide in der Bubble-phase feuern, läuft erst eine Action-Toolbar-Anzeige UND dann das Aufklappen — User sieht visuelle Inkonsistenz. **Lesson generalisierbar:** Listener auf demselben Event-Target werden in Registrierungsreihenfolge gefeuert, aber das ist fragil — wenn jemand die Reihenfolge der `addEventListener`-Aufrufe ändert (z.B. durch einen Refactor), bricht der Effekt. Capture-phase ist die explizite Lösung: dritter Parameter `true` macht den Listener priority. Plus `stopPropagation()` damit die Bubble-phase gar nicht erst feuert
- **`addEventListener('click', handler, true)` plus `stopPropagation()` als saubere Vorrang-Mechanik.** Capture-phase läuft top-down (Document → Target), Bubble-phase läuft bottom-up (Target → Document). Standard-Listener registrieren in Bubble-phase (Default `false` als dritter Parameter). Capture-phase-Listener feuern garantiert vor Bubble-phase-Listenern auf demselben Element — und mit `event.stopPropagation()` kann ich die Bubble-phase ganz unterdrücken, falls die Capture-Logik bereits gehandelt hat. **Lesson:** wenn ein bestehender Listener nicht angefasst werden soll (P139 ist verifiziertes Verhalten), aber ein neuer Listener Vorrang braucht, ist Capture + stopPropagation der einzige saubere Weg. KEINE direkten Modifikationen am bestehenden Listener
- **Whitelist `a, button, select, input, textarea` von Tap-Logik ausschließen.** Tap auf einen Button innerhalb der Bubble (z.B. Copy-Button, TTS-Button) darf nicht das Collapse togglen — der Button hat sein eigenes onclick. Identisch zur P139-Whitelist `target.closest('a, button, select, input, textarea')` als Early-Return. **Lesson:** wenn man Click-Listeners auf Container-Elementen registriert, immer eine Interactive-Whitelist als Early-Return prüfen — sonst wird jeder Inline-Button vom Container-Listener gestört

---

## Source-Audit-Test mit `.split("}")` frisst die schließende Klammer (P-UI-2)

- **`.split(delim)` entfernt das delim-Zeichen aus den Stücken — wenn das delim im gesuchten Pattern selbst vorkommt, frisst der Split die Pattern-Bytes.** Mein erster Test-Versuch für „User-Tap-Listener nutzt capture phase" splittete `addmsg_block.split("}")[0:6]` und jointe zurück — aber `}, true);` (Listener-Ende) wurde dadurch zu `, true);` weil das `}` direkt davor der Splitter war. Test failte fälschlicherweise auf einen korrekten Code. **Lesson generalisierbar:** beim Source-Splitten den Delimiter so wählen, dass er NICHT im gesuchten Suffix-Pattern vorkommt. Bei Listener-Heuristiken NIE nach `}`/`)` splitten — die kommen im JS-Listener-Schema überall vor
- **Pro-Pattern: erst nach einem eindeutigen Marker splitten, dann fixe Slice-Länge.** Statt `.split("}")[0:N]` und join: `block.split(unique_marker)[1][:1500]`. Das schneidet ab `unique_marker` und nimmt 1500 Zeichen — `[:1500]` frisst keine Zeichen, ist deterministisch, und 1500 reichen für jeden Listener-Block. **Lesson:** Source-Audit-Heuristiken auf Slice basieren statt auf Split-Strukturzeichen — Slice ist non-destruktiv

---

## Volle Bubble-Breite braucht ZWEI CSS-Edits, nicht einen (P-UI-1)

- **Wenn ein äußerer Wrapper die Breite limitiert, hilft es nicht, die innere Bubble auf 100% zu setzen.** Im Nala-Frontend sass die `.message`-Bubble in einem `.msg-wrapper`, der selbst `max-width: 92%` (Mobile) bzw. `80%` (Desktop) hatte. Erste Variante des Layout-Umbaus änderte nur `.message` auf `100%` — visuell hatte sich nichts geändert, weil der Wrapper die Bubble bei 92% gehalten hat. Erst nach Änderung von **beiden** (Wrapper UND Bubble) auf `max-width: 100%` und `align-self: stretch` greift der volle-Breite-Effekt. **Lesson generalisierbar:** bei Layout-Edits in geschachtelten Containern alle Layer prüfen — einen Selector zu ändern reicht selten. Vor dem Edit die Container-Hierarchie ablaufen (`.message` ⊂ `.msg-wrapper` ⊂ `.chat-messages` ⊂ `.app-container`) und entscheiden, auf welcher Ebene der Cap aktuell sitzt
- **`align-self` allein reicht nicht — `max-width` muss mitziehen.** `align-self: stretch` macht ein Flex-Item so breit wie der Container, ABER nur wenn keine `max-width` davorsteht. Wenn `max-width: 92%` gesetzt ist und `align-self: stretch` dazu, bleibt das Element bei 92%. Beide CSS-Eigenschaften gemeinsam ändern. **Lesson:** Stretch ist eine "wenn nichts begrenzt"-Aussage; `max-width` ist eine "härtere" Begrenzung, die Vorrang hat. Beim Layout-Audit beide Werte gemeinsam lesen
- **Source-Audit-Test als Anti-Refactor-Backstop, auch bei reinen CSS-Patches.** `test_no_92_or_80_percent_cap_on_message` und `test_no_avatar_class_in_css` zementieren das neue Layout. Wenn ein zukünftiger Patch versehentlich den 80%-Cap zurückbringt (Copy-Paste, Merge-Konflikt) oder ein `.avatar`-Element einführt, schlagen die Tests sofort an statt erst beim nächsten Browser-Test. **Lesson:** Source-Audit-Pattern ist nicht nur für Anti-Patterns in Python-Code wertvoll, sondern auch für Anti-Patterns in inline-CSS — gerade wenn das CSS in einer Python-Datei lebt (`nala.py`) und reguläre CSS-Linter es nicht erfassen
- **`P-UI-1`-Inline-Marker im Source als Diff-Hinweis.** Bei jedem geänderten CSS-Block wurde ein Kommentar mit `P-UI-1 (Phase 5c, 2026-05-07)` und kurzer Begründung eingebaut. Wenn jemand in einem späteren Patch eine der Regeln rückrolliert, ist im Diff sofort sichtbar, dass es sich um einen P-UI-1-Effekt handelt. **Lesson:** Patch-Tag-Kommentare sind die Brücke zwischen Code und Patch-Doku — sie überleben das Vergessen der Begründung viel besser als externe Doku

---

## System-Maintenance-Cron-Jobs nicht in der DB-Task-Tabelle, sondern direkt am APScheduler (P213-pre-4)

- **Wenn ein Cron-Job System-Maintenance ist (GC, Cleanup, Health-Check), gehoert er NICHT in die User-konfigurierbare Task-Tabelle.** Erste Versuchung war, den FAISS-Backup-GC als P214-Task `kind=shell` mit DB-Eintrag zu bauen — analog dem HANDOVER-Vorschlag. Schnell aufgefallen: `_execute_kind_shell` braucht `project_id` (Workspace-Mount-Defense, P214-pre-3), und ein globaler Maintenance-Job hat keinen Projekt-Workspace. Plus: User soll System-GC nicht versehentlich aus der Hel-UI disablen koennen — das ist kein "kannst du frei waehlen"-Task, das ist Infrastruktur. **Lesson generalisierbar:** Task-Systeme haben oft zwei Ebenen — User-konfigurierbare Tasks (DB + UI + Disable-fuer-User) und System-Maintenance-Tasks (Code + immer-an + nicht User-sichtbar). Letztere registriert man direkt am Scheduler analog dem P57-Overnight-Job, ohne DB-Umweg. Spart auch eine Wahl von Task-Kind und einen Default-Seed-Mechanismus
- **Pure-Function-Schicht plus Cron-Hook als minimaler Zwei-File-Patch.** `zerberus/utils/backup_gc.py` enthaelt `find_stale_backups`/`delete_backup_files`/`run_backup_gc` (alle pure, kein APScheduler-Import) — testbar mit `tmp_path` + `os.utime` ohne `time.sleep`. `zerberus/main.py`-Lifespan registriert `_scheduler.add_job(run_backup_gc, trigger="cron", day_of_week="sun", hour=3, minute=0, ...)` direkt. Source-Audit-Test prueft nur dass die Lifespan-Sektion `from zerberus.utils.backup_gc import run_backup_gc` und `add_job(...)` mit dem GC-Helper enthaelt. **Lesson:** wenn ein Patch nur eine Pure-Function-Logik plus eine Verdrahtungs-Stelle hat, brauchen die Tests keine Integration-Fixtures — Pure-Function-Tests fuer die Logik plus Source-Audit-Tests fuer die Verdrahtung reichen. Ein 31-Test-File ohne ein einziges DB-Setup
- **`os.utime` plus `now_seconds`-Override ist der Trick fuer GC-Tests ohne `time.sleep`.** Klassisches Anti-Pattern: ein Test wartet 8 Sekunden mit `time.sleep(8)` und prueft dass eine 7-Sekunden-Cutoff-Regel greift. Resultat: langsame Test-Suite, flaky bei CI-Last-Spitzen. Stattdessen: `os.utime(path, (now-age_seconds, now-age_seconds))` setzt die mtime kuenstlich, plus optionaler `now_seconds`-Parameter in der Pure-Function damit der Test "die jetzige Zeit" ohne `freezegun` ueberschreiben kann. **Lesson generalisierbar:** jeder Pfad, der `time.time()` ODER `datetime.utcnow()` aufruft, sollte einen `now`-Parameter mit Default `None` (faellt auf live-time zurueck) anbieten — dann bleiben die Tests deterministisch und schnell ohne externe Mock-Lib. Plus: `os.utime` ist der portable Weg ein File-mtime fuer Tests zu fakeen — auch unter Windows

---

## Reset-vor-Rebuild auf persistente Stores ist immer Datenverlust-Risiko (P213-pre-3)

- **"Erst loeschen, dann neu aufbauen" ist verfuehrerisch einfach und immer falsch wenn der Store persistent ist.** Der alte `rag_reindex`-Pfad lief `_reset_sync` (Index + Metadata leeren + auf Disk persistieren) gefolgt von einem Encoding-Loop. Crash mitten im Loop → leerer Index auf Disk → Userdaten weg. Der Spalt zwischen den Schritten kann mikrofein sein, aber er ist real. **Lesson generalisierbar:** bei jeder "alten Zustand loeschen → neuen aufbauen"-Operation auf persistenten Stores muss der neue Zustand VORHER vollstaendig in einer separaten Variable existieren, dann atomar geswappt werden. Gilt fuer FAISS-Indices, DB-Migrationen, Config-Files, Workspace-Snapshots, Cache-Verzeichnisse. Anti-Pattern: "wir leeren das Ziel, dann fuellen wir es neu" — Pro-Pattern: "wir bauen das neue Ziel parallel auf, dann tauschen wir es atomar gegen das alte"
- **FAISS-BAK-MUSTER auch fuer planmaessige Operationen, nicht nur fuer Recovery.** P218-pre nutzte das Pattern bei Dim-Mismatch-Recovery (impliziter Datenverlust), P213-pre-3 nutzt es bei jedem geplanten Reindex (zur Sicherheit auch wenn alles gut geht). Das Backup zahlt sich aus bei dem 1% der Faelle wo der Swap fehlschlaegt — und in den 99% normalen Laeufen kostet es nur eine `shutil.copy2`. **Lesson:** bei jedem Pfad der ein FAISS-Index-File ueberschreibt, das Pre-Image nach `<file>.<grund>_<UTC>.bak` sichern. Backup-Garbage-Collection ist eine eigene Schuldenposition (Cron-Job alter `.bak`-Dateien)
- **Anti-Pattern-Check als Source-Audit-Test ist genauso wertvoll wie "diese Funktion existiert".** `test_endpoint_no_longer_calls_reset_or_add_to_index` prueft das Endpoint-Source darauf, dass das alte Anti-Pattern (`_reset_sync(...)` direkt vor einem `_add_to_index(...)`-Loop) NICHT mehr vorkommt. Wenn jemand in einem zukuenftigen Refactor versehentlich das alte Pattern wieder einfuehrt (Copy-Paste, Merge-Konflikt), schlaegt der Test sofort an. **Lesson:** Source-Audit-Tests sind nicht nur fuer "diese Funktion existiert" gut, sondern auch fuer "dieses Anti-Pattern kommt nicht mehr vor" — beides kostengeguenstige Backstops gegen Regression. Bei jedem Bugfix der ein Anti-Pattern aus dem Code entfernt: einen Test schreiben der das Anti-Pattern NICHT mehr findet

---

## "quasi" ist semantischer Qualifier, kein Filler-Word (P219-pre)

- **Hedge-Woerter veraendern Aussagen — sie raus zu filtern macht Saetze falsch.** "quasi", "sort of", "kind of", "annaehernd", "ungefaehr" sind Bedeutungs-relevant: "Das ist quasi Reasoning" ≠ "Das ist Reasoning". Wer sie als Filler streicht, produziert eine staerkere Aussage als der User meinte. Im Whisper-Cleaner kam "quasi" in der Filler-Regex `(?i)\b(eigentlich|irgendwie|quasi|uebrigens|prinzipiell|im endeffekt)\b` und im prompt_compressor `_STOPWORDS`-Set. Beide entfernt. **Lesson generalisierbar:** vor jedem Listen-Eintrag in einer Filler-/Stop-Word-Liste fragen: ist das Wort funktional (Diskursmarker, Pausen-Filler — "ehm", "tja", "nun") oder semantisch (Qualifier, Verstaerker, Negation, Modal-Partikel mit Bedeutung)? Funktional → weg, semantisch → drin lassen, auch wenn es statistisch prominent ist

---

## Supervisor-Bug-Sammelstelle als Buffer-Schicht zwischen Beobachtung und Aktion (P219-pre)

- **Direktes Beobachtung→Patch zerstuckelt Aufmerksamkeit.** Vor P219-pre haette der Supervisor jeden gemeldeten Bug sofort in einen Coda-Auftrag uebersetzt — fuenf Bugs in einer Sitzung = fuenf Patches mit fuenf Commit-Sequenzen + fuenf Sync-Wellen. Mit Sammelstelle: ein Sammel-Patch pro Trigger-Phrase ("los", "pack zusammen") mit einem Commit + einer Sync-Welle. **Lesson generalisierbar:** in agentischen Multi-Loop-Systemen lohnt sich eine Buffer-Schicht zwischen Beobachtung und Aktion. Der "los"-Trigger ist der Flush-Befehl, vorher bleibt die Liste passiv und sichtbar (visueller Druck) ohne Aktivismus
- **Bug-Sammelstand-Anzeige als visueller Druck.** Wenn die Sammelstelle am Ende jeder Antwort als nummerierte Liste sichtbar ist, sieht Chris automatisch wenn sie waechst und gibt von sich aus den Flush-Befehl. Ohne Anzeige bleibt die Liste unsichtbar — entweder die Sammelstelle wird vergessen, oder der Supervisor muss aktiv erinnern (was wieder Aktivismus waere). **Lesson:** bei passiven Buffer-Schichten muss der Stand sichtbar sein, sonst verlaeuft sich der Buffer
- **Trigger-Phrasen explizit machen.** "los", "pack zusammen", "mach einen Patch draus" sind die Flush-Trigger. NICHT-Trigger sind impliziter Hinweis ("waere gut zu fixen"), Zustimmung ("ja stimmt") oder Mehrfach-Nennung. Wer einen Bug mehrfach erwaehnt, ist immer noch in der Sammelphase — nicht in der Aktions-Phase. **Lesson:** bei UX-Trigger-Mechanik immer Whitelist statt Blacklist — und die Whitelist explizit dokumentieren, sonst eskaliert sie still

---

## FAISS Dim-Mismatch-Recovery muss am Add-Pfad sitzen (P218-pre)

- **Pattern direkt aus P199 uebertragbar** — Projekt-RAG (`projects_rag.index_project_file`) hat das Problem schon geloest: bei Dim-Mismatch im vstack-Versuch wird der Index neu aufgebaut statt fehlertraechtig erweitert (lessons.md "Embedder-Dim-Wechsel zwischen Sessions toleriert"). P218-pre kopiert das auf den globalen Pfad (`modules/rag/router.py::_add_to_index`): Helper `_reset_index_inplace(target_dim, settings)` baut FAISS mit der neuen Dim neu, leert Metadata, persistiert via dual-aware-Pfad. WARN-Log mit `[RAG-218]`-Tag und alter+neuer Dim, sonst wundert sich die Bug-Suche|Lesson generalisierbar: wenn ein Pure-Function-Pattern in der Projekt-Variante schon getestet ist, lohnt der 1:1-Transfer auf die globale Variante mehr als eine Neuentwicklung — gleicher Code, halbe Reviewzeit

---

## Autonome Prioritätsliste — fragen nur bei echtem Architektur-Risiko (FR-AUTONOME-PRIORITÄT)

- **AUTONOME-PRIORITÄT — Coda fragt nicht was als nächstes kommt; er liest HANDOVER+WORKFLOW und entscheidet** | "Wie möchtest du weitermachen?" ist ein Fehler wenn die Antwort aus Doku ableitbar ist | P216-Ende war der Lernmoment: Phase 5a war komplett, der HANDOVER nannte Doku-Catchup als höchste Prio, trotzdem hat Coda nach dem nächsten Schritt gefragt — die Frage war Reibung ohne Nutzen

---

## Loki / Fenrir / Vidar sind kein Extra — sie sind der Beweis (P215)

- **Wenn die drei Suiten nicht gegen den Live-Server gelaufen sind, ist nicht bewiesen dass das System funktioniert.** Coda hat seit P93 (als Loki/Fenrir/Vidar gebaut wurden) Dutzende UI-/Auth-/Pipeline-Patches gemacht ohne sie konsequent auszufuehren — und am Ende geschrieben "Chris muss noch testen". Das ist dasselbe Anti-Pattern wie bei Unit-vs-Integration-Tests aus P213-pre-6: was bequem ist (lokale Mocks, Unit-Tests) wird gebaut, was notwendig ist (Live-E2E, Live-Chaos) wird vermieden|**Lesson generalisierbar:** wenn ein System eine Smoke-/E2E-/Chaos-Suite hat, IST diese Suite das Pflicht-Gate vor "Patch ist fertig". Nicht "wenn ich Zeit habe", nicht "wenn der Patch gross genug ist" — bei jedem Patch der den von der Suite abgedeckten Bereich anfasst. Suite gruen ODER dokumentierter Skip-Grund mit Schulden-Eintrag, sonst ist der Patch nicht durch

---

## Cron-Shell-Runs: Snapshot-vor-Run ist fail-CLOSED, nicht best-effort (P214-pre-3)

- **Beim Verdrahten eines neuen schreibenden Pfads gegen einen bestehenden Snapshot-Layer entscheiden: fail-open oder fail-closed?** Das hat KEINE neutrale Antwort — es haengt davon ab, was der Snapshot tatsaechlich tut. Bei P214-pre-3 (Cron-Shell-Run mit `writable=True`-Workspace-Mount) macht der Pre-Snapshot zwei Dinge gleichzeitig: 1) Tar-Archiv des Workspace-Stands fuer Rollback, 2) Vergleichsanker fuer den After-Snapshot-Diff. Wenn der Pre-Snapshot crasht, hat man WEDER Rollback NOCH Diff — der writable Run waere ein nicht-rueckgaengig-machbarer Schreibzugriff ohne Audit-Spur. Lesson: bei jedem Pfad der den Workspace verändern darf gilt fail-CLOSED — Pre-Snapshot-Crash → Run abgelehnt, status=error|Anti-Pattern: "best-effort, kein Snapshot ist auch OK" — verleiht ein falsches Sicherheitsgefuehl, weil der nominelle Pfad mit Snapshot lief, der Crash-Pfad aber nicht

---

## Sandbox-Sprache erweitern: Default belassen, Operator opt-in lassen (P214-pre-3)

- **Neue Sandbox-Sprache hinzufuegen heisst NICHT, sie automatisch in `allowed_languages` aufnehmen.** Bei P214-pre-3 hatte ich das Versuchung — `bash` zu `["python", "javascript", "bash"]` als Default machen. Wäre falsch: shell-Cron-Runs sind ein neues Risiko-Profil (Build/Test/Log mit writable Mount + langer Laufzeit) gegenueber Code-Snippet-Ausfuehrung im Chat (read-only Mount + 30s default-Timeout). Default-Inclusion wuerde jedes bestehende Deployment ohne Operator-Eingriff aktivieren — nicht zumutbar|Pattern: SandboxConfig kriegt `bash_image: str = "debian:bookworm-slim"` (Default ist da, Image faellt nicht runter), aber `allowed_languages` Default bleibt `["python", "javascript"]`. Operator muss bewusst beide Felder aktivieren: `allowed_languages` erweitern + Image ziehen. Sonst lehnt Sandbox mit klarer Fehlermeldung ab|Lesson generalisierbar: bei jeder neuen Capability die nur durch Verkettung mehrerer Schichten Wirkung entfaltet, gilt: Defaults so setzen dass die Verkettung GENAU EIN explizites Opt-In braucht. "Image-Default da" und "Sprache nicht in Whitelist" zusammen heisst: harmlos bis Operator zustimmt

---

## Inline-onclick mit verschachtelten Quotes ist eine Falle (P214-pre-2 + P203b-Recall)

- **Python-String mit inline `onclick="fn(\\'' + var + '\\')"` zerbricht im Browser-JS und im node-Syntax-Check.** Beim Erzeugen einer Tabelle in Hel-UI hatte ich `'<button onclick="runSchedTaskNow(\\'' + t.task_id + '\\')">'` geschrieben — die `\\'`-Sequenz im Python-Source-String wird zu `\'`, dann zu `'` im finalen ADMIN_HTML, und das ergibt im JS-Output `onclick="runSchedTaskNow('' + t.task_id + '')"` — zwei leere Strings statt der erwarteten Quote-Verschachtelung|Symptom: `test_alle_inline_scripts_parsen` aus P203b knallt mit `SyntaxError: Unexpected string` weil der JS-Parser ein leeres String-Literal sieht|Fix: P203b-Pattern mit `data-Attribute` + `addEventListener` statt inline `onclick` — Tabellen-Zellen kriegen `data-sched-task-id="..."` plus `data-sched-action="run|edit|delete"`, der Listener wird nach dem `innerHTML`-Set per `tbody.querySelectorAll(...)` verdrahtet|Lesson generalisierbar: in HTML das Python erzeugt, das JS enthaelt das Strings concatenated, NIEMALS Inline-onclick mit Variablen-Werten — IMMER data-Attribute + addEventListener. Quote-Eskalation ist nicht intuitiv vorhersagbar zwischen Python-Source-Quoting, Python-String-Wert und JavaScript-Parser

---

## Scheduler-Handler-Signatur als ScheduledTask statt str (P214-pre-2)

- **Wenn Handler-Funktionen mehr Kontext brauchen, dispatch das ganze DB-Row, nicht nur eine Spalte.** P214-pre-1 hat `_execute_kind_*(payload: str)` definiert — funktionierte fuer notice (nur payload noetig), aber bei pre-2 brauchte der prompt-Pfad `task.project_id` fuer den Persona-+RAG-Lookup|Refactor: Handler-Signatur einheitlich auf `(task: ScheduledTask) -> tuple[bool, str, Optional[str]]`, der Dispatcher gibt das ganze ORM-Row weiter|Vorteil: spaetere Erweiterungen (z. B. Per-Task-Modell-Override) brauchen kein zweites Refactor, weil das ganze Row schon verfuegbar ist|Lesson generalisierbar: bei Dispatch-Tabellen (kind→handler-Pattern), GLEICH am Anfang den vollen Kontext-Datentyp uebergeben, auch wenn der erste Handler nur eine Spalte braucht. Spaetere Schichten zahlen den Refactor-Preis sonst doppelt

---

## Scheduler-Pfad: Late-Binding auf Modul-Globale + 1-Loop-pro-Test (P214-pre-1)

- **`from module import _global_var` snapshottet die Referenz beim Import.** Im P214-pre-1-Runner war `from zerberus.core.database import _async_session_maker` die naheliegende Importform — aber das produziert ein internes Modul-lokales Symbol, das beim Import-Zeitpunkt einmal `None` ist und sich NIE wieder aktualisiert, auch wenn jemand später `db_mod._async_session_maker = new_maker` setzt|Symptom: Integration-Tests, die `_async_session_maker` patchen, sehen weiterhin `None` und der Runner antwortet "DB nicht initialisiert"|Fix: `from zerberus.core import database as _db_mod` plus zur Laufzeit `_db_mod._async_session_maker` lesen — Late-Binding gibt jedem Aufrufer den aktuellen Modul-Stand|Lesson generalisierbar: bei Modul-Globalen, die zur Laufzeit oder im Test gepatcht werden koennen, IMMER ueber das Modul-Objekt zugreifen, nicht direkt importieren — sonst patcht der Test eine Referenz, die niemand sieht

---

## Mock-Tests pruefen Code, Integration-Tests pruefen System (P213-pre-6)

- **Mock-Tests = "der Code laeuft korrekt", Integration-Tests = "das System liefert das richtige Ergebnis" — beides ist noetig, keiner ersetzt den anderen.** Die P213-pre-Saga (vier Code-Patches) hat das in Reinform vorgefuehrt: Coda hat Code + Doku + 17 neue Mock-Tests verifiziert, ohne EIN MAL gegen den laufenden Server zu pruefen, ob `_huginn_rag_lookup("Bei welchem Patch sind wir gerade?")` jetzt wirklich den aktuellen Patch liefert. Die Mock-Tests waren alle gruen, der echte Server hat in dieser Zeit weiterhin "Patch 178" halluziniert|Lesson generalisierbar: bei jedem Patch der Pipeline / RAG / Auth / Sync beruehrt, MUSS es mindestens einen Integration-Test geben der das Endergebnis pruefen, nicht nur die Code-Logik|Anti-Pattern: "Tests sind gruen, also Patch fertig" — gruen heisst nur "das was getestet wird, funktioniert", nicht "das System liefert was der User erwartet"

---

## Embedder-Drift bei Para­phrasen-Queries: FAQ-Form als Anker-Verstaerker (P213-pre-4)

- **Stand-Anker mit Erklaer-Text rankt schlecht auf Para­phrasen — wortgleiche Frage-Strings sind die Loesung.** Diagnose `tools/diag_p213_pre_4.py` zeigte: der Stand-Anker-Chunk (200 Woerter, "## Aktueller Stand" plus Erklaerung) lag bei der Frage `"bei welchem Patch sind wir gerade"` NICHT IN TOP-20 der L2-Distanzen, dabei rankt er bei wortgenauer Anfrage `"letzter Patch"` auf Rang 1|Ursache: der DE-Embedder (T-Systems Roberta-Sentence-Transformer) misst Satz-Aehnlichkeit, nicht Frage-Antwort-Match — eine Frage als Token-String matcht semantisch besser auf eine andere Frage als auf eine erklaerende Aussage, selbst wenn die Aussage die gleiche Information traegt|Fix-Pattern: FAQ-Form im Anker einbauen — die wahrscheinlichen User-Fragen WOERTLICH auflisten plus jeweils kurze Antwort. Embedder findet wortgleiche Frage-Tokens auf Rang 1, das `min_chunk_words=30`-Ziel der system-Kategorie ist trotzdem locker erfuellt|Lesson generalisierbar: bei jedem RAG-Doku das LLM-Antworten zu spezifischen User-Fragen liefern soll, NICHT auf Embedder-Magic vertrauen — die wahrscheinlichen Fragen wortgleich im Doku ablegen. Embedder-Distanz zwischen "Frage A" und "Frage A" ist null, zwischen "Frage A" und "Antwort A in Erzaehlform" ist nicht-null und kann ueber 1.5 L2-Einheiten betragen|Anti-Pattern: "der Embedder versteht das schon" — bei kurzen Queries (< 5 Tokens) verstehen Sentence-Transformer fast nichts ueber Bedeutung, sondern nur Token-Ueberlapp.

---

## uvicorn --reload-Flut bei Coda-Sessions: Debounce + Watch-Scope (P213-pre-4 / FR-Reload)

- **`--reload-delay 2.0` reduziert die Reload-Flut massiv.** Default-Verhalten: jede Datei-Aenderung triggert sofort einen Reload — bei einer Coda-Session mit 40 Edits in 2 Minuten = 40 Reloads, jeder haengt 30-60s an offenen Browser-Keepalive-Connections|Mit 2-Sekunden-Debounce: nur wenn 2 Sekunden Stille auf das letzte Edit folgen, wird reloaded — typische Coda-Burst von 5 Edits in 10 Sekunden = 1 Reload statt 5|Voraussetzung: uvicorn >= 0.20 (vorhanden in 0.24.0). Bei aelteren Versionen `pip install --upgrade uvicorn` autonom moeglich

---

## Sync-Tool-Defaults muessen Realitaet widerspiegeln + Bug-Stacking (P213-pre)

- **Tool-Defaults muessen zu der Server-Konfiguration passen, gegen die das Tool spricht.** `tools/sync_huginn_rag.py` hatte als Default `http://localhost:5000`, aber `start.bat` startet den Server mit `--ssl-keyfile`/`--ssl-certfile` auf Port 5000 (HTTPS). Folge: `RemoteProtocolError: Server disconnected` — das Tool sprach Plain-HTTP gegen einen TLS-Port|Drei Marathon-Sessions (P210/P211/P212) lang scheiterte der Sync mit "Server nicht erreichbar" — ohne dass jemand den Default als Bug erkannt hat, weil "scheitert wenn Server nicht laeuft" als Symptom akzeptiert wurde|Lesson: bei jedem CLI-Tool das gegen einen lokalen Server spricht IMMER einmalig den Happy-Path mit laufendem Server testen — sonst bleibt der Default-Bug latent bis jemand den Sync wirklich braucht|Fix-Pattern: Default an `start.bat`-Realitaet anpassen (`https://localhost:5000`), `ZERBERUS_URL=http://...` als explizites Override fuer alternative Setups

---

## Reasoning-Sichtbarkeit: sync-emit + async-audit + Frontend-Polling (P213)

- **Sync-emit + async-audit ist die richtige Asymmetrie fuer Telemetrie im Hot-Path.** `emit_step` MUSS sync sein, weil der Chat-Pfad nicht auf einen DB-Insert warten darf — der Step landet im In-Memory-Buffer und der Caller faehrt sofort weiter|Audit-Insert ist asynchron als Hintergrund-Task auf dem laufenden Event-Loop|Wenn kein Loop verfuegbar ist (Sync-Test), wird die Coroutine GAR NICHT ERST erzeugt (sonst RuntimeWarning "coroutine was never awaited") — `loop = asyncio.get_running_loop()` mit RuntimeError-Catch + None-Fallback ist das Pattern|Generalisierbar fuer jede Telemetrie-Pipeline: niemals einen `asyncio.create_task` ausserhalb eines aktiven Loops erzeugen.

---

## GPU-Queue: statisches Budget + FIFO + Audit-Tabelle als Tuning-Grundlage (P211)

- **Statisches VRAM-Budget schlaegt dynamische Detection.** `nvidia-smi`-Polling ist unzuverlaessig (Treiber-Caching, Race-Conditions zwischen Read und Allocate, Abhaengigkeit zu CUDA-Headern). Statische Tabelle aus produktiven Messungen plus konservativer 11-von-12-GB-Budget liefert die gleiche Schutzwirkung mit 30 Zeilen Code statt 200|Lesson generalisierbar: bei Resource-Limits IMMER zuerst pruefen ob ein statischer Wert reicht, bevor man eine dynamische Detection baut — die ist fast immer fragiler|`compute_vram_budget(consumer_name)`-Pattern: einer Whitelist mit `_KNOWN_CONSUMERS` plus `DEFAULT_*_BUDGET_MB` mit Warning-Log fuer unbekannte Namen — Tippfehler in der Verdrahtung werden so loud detektierbar

---

## RAG-Selbstwissen muss explizite Stand-Anker tragen UND auto-synct werden (P210)

- **Doku ohne expliziten Stand-Anker laesst das LLM raten.** `huginn_kennt_zerberus.md` beschrieb bewusst nur den "aktuellen Zustand", nicht die Patch-Historie — dadurch existierte keine Zeile wie "Letzter Patch: P209" im Index. Bei "welcher Patch?"-Fragen griff Huginn auf die prominentesten Patch-Tags im Doku-Text — `[HUGINN-178]` als Logging-Tag aus einer Audit-Tag-Liste — und antwortete konsistent "178". Lesson: wenn Huginn auf eine spezifische Frage IMMER falsch antwortet, ist die Antwort wahrscheinlich gar nicht im RAG, sondern wird halluziniert auf Basis prominenter Strings in den Chunks die zufaellig hochranken|Fix-Pattern: Pflicht-Header `## Aktueller Stand` ganz oben, vier Bullet-Points (Patch / Phase / Tests / Datum), Source-Audit-Test prueft Existenz UND Mindestnummer

---

## Zweite Meinung vor Ausfuehrung — Veto-Layer als Pre-Filter zum HitL (P209)

- Layered Defense: Veto (P209) **VOR** HitL (P206) **VOR** Sandbox (P203c). Drei unabhaengige Schichten, jede mit eigenem Audit-Trail (`code_vetoes` / `code_executions`)|Veto ist die einzige Schicht, in der das System eine **Read-Only**-Entscheidung trifft (User kann nicht ueberschreiben — bei false-positive muss er die Frage neu formulieren)|HitL ist User-Interaktiv (approve/reject), Sandbox ist Code-Execution|Logisch saubere Trennung: Veto schuetzt vor "Code-Vorschlag passt nicht zum Wunsch" oder "Code ist gefaehrlich", HitL schuetzt vor "ich will das gerade nicht ausfuehren", Sandbox schuetzt vor "Code soll nichts kaputtmachen koennen"

---

## Spec-Contract / Ambiguitaets-Heuristik vor LLM-Call (P208)

- Heuristik-Score VOR LLM ist die billigste Filter-Stufe: `compute_ambiguity_score` ist Pure-Function, keine DB, kein I/O, keine Tokens|Cost-Profil: 0$ pro nicht-ambiger Message (Score < Threshold), 1 Probe-Call pro ambiger Message (~50 Tokens) plus Long-Poll-Fenster|Threshold tunen ueber `clarifications`-Audit-Tabelle: hoher Bypass-Rate → Threshold zu niedrig (zu viele Probe-Calls), hoher Cancelled-Rate → Frage formulierung nicht hilfreich

---

## Workspace-Snapshots / Diff / Rollback (P207)

- Snapshot-Tar als atomare Einheit: `materialize_snapshot` schreibt mit Tempname + `os.replace` — paralleler Reader sieht nie ein halb-geschriebenes Tar|`restore_snapshot` raeumt Workspace-Inhalt komplett (Root bleibt stehen, sonst Watcher-Konfusion) und extrahiert mit Member-Validation|Tar-Format `ustar` (Default) reicht; `data_filter` aus Python 3.12 nicht nutzen, wenn 3.10/3.11 unterstuetzt werden — `_is_safe_member` macht es portabel

---

## Frontend-Toasts / dezente UI-Notifications (P205)

- Replace-Pattern statt Stacking fuer Status-Toasts: ein einziger `#ragToast`-DOM-Container, CSS-State-Toggle (`.visible`) fuer Show/Hide|Singleton-Timeout-Slot via Function-Property (`_showRagToast._t`) cancelt den vorigen Auto-Hide bevor der neue startet|bei Multi-Item-Uploads gewinnt der letzte Toast — die Item-Liste oben zeigt eh den per-File-Status, der Toast ist additiv fuer EINE Zusatzinfo (hier: RAG-Indizierung)

---

## LLM-Kontext-Brücken (Prosodie, RAG, Persona, Code-Synthese)

- LLM-Brücken-Blöcke IMMER mit eindeutigem Marker-Paar bauen (P204): `[PROSODIE — ...]` / `[/PROSODIE]` analog `[PROJEKT-RAG — Kontext aus Projektdateien]` (P199), `[PERSONA-Marker]` (P197) und `[CODE-EXECUTION — Sprache: ... | exit_code: ...]` / `[/CODE-EXECUTION]` (P203d-2, im Synthese-Prompt)|Marker-Substring in `system_prompt` ist der Idempotenz-Check (zweiter Aufruf hängt nicht doppelt an)|Test `TestMarkerUniqueness` verifiziert dass alle Marker disjoint sind (kein Substring-Match zwischen ihnen)

---

## Konfiguration

- config.yaml = Single Source of Truth|config.json NIE als Konfig-Quelle (Split-Brain P34)

---

## Datenbank

- interactions-Tabelle hat KEINE User-Spalte|User-Trennung nur per Session-ID (unzuverlässig, vor Metrik-Auswertung klären)

---

## RAG

- Orchestrator Auto-Indexing aus|erzeugt sonst „source: orchestrator"-Chunks nach manuellem Clear (P68)|verdrängt echte Dokument-Chunks|nach jedem Clear prüfen

---

## Nala-UI / Header-Injektion (P201)

- Auth-Bridge zwischen Hel-CRUD (Basic-Auth) und Nala-User (JWT): NICHT versuchen. Statt Hel-Endpoint wiederverwenden lieber eigenen schlanken Endpoint im Nala-Router bauen — JWT-Auth via `request.state.profile_name` aus dem Middleware|Bonus: man kann das Response-Dict für den User slimmen (kein `persona_overlay` rausgeben — das ist Admin-Geheimnis und kann Prompt-Engineering-Spuren enthalten)

---

## PWA / Service Worker (P200)

- Router-Level-Dependencies in FastAPI gelten NUR für Routes desselben Routers|Wenn ein Endpoint denselben URL-Prefix wie ein auth-gated Router hat, aber öffentlich sein muss (z.B. `/hel/manifest.json` für PWA-Install): in einen separaten `APIRouter()` ohne Dependencies legen UND in `main.py` VOR dem auth-gated Router via `include_router` einhängen|Symptom bei falscher Reihenfolge: Browser zeigt keinen "App installieren"-Prompt, weil der Manifest-Fetch eine 401-Auth-Challenge bekommt

---

## Service Worker und Browser-Auth-Prompts (P202)

- **SW darf navigation-Requests NICHT via `respondWith` abfangen wenn die Page Basic-Auth (oder allg. WWW-Authenticate) nutzt**|Symptom: User sieht statt Auth-Prompt nur den 401-JSON-Body als sichtbare Response|Ursache: Bei SW-vermittelten Antworten ignoriert der Browser den `WWW-Authenticate: Basic`-Header — der native Auth-Prompt-Pfad wird umgangen|Fix: früh-return im fetch-Handler bei `event.request.mode === 'navigate'`, dann läuft die Navigation durch den nativen Browser-Stack inkl. Auth-Prompt + HTTPS-Indikatoren|Alternative (cache-first für Offline-Modus) ist möglich, aber dann braucht der SW eigene Auth-Logik — selten den Aufwand wert für Heimserver-Use-Cases

---

## Security / Auth

- JWT blockiert externe Clients (Dictate, SillyTavern)|`static_api_key` in config.yaml als Workaround (P59/64)

---

## Deployment

- `%~dp0` = Script-Directory|ersetzt hardcodierte Pfade in .bat|in Anführungszeichen bei Leerzeichen: `cd /d "%~dp0"` (P117)

---

## Pacemaker

- Double-Start: `update_interaction()` muss async sein|alle Call-Sites brauchen await (P59)

---

## Architektur / Bekannte Schulden

- RAG Lazy-Load-Guard in `_encode()`: `_model` war None bei erstem Upload|immer auf None prüfen vor Modell-Nutzung

---

## Pipeline / Routing

- Nala-Frontend routet über `/v1/chat/completions` (legacy.py), NICHT Orchestrator|Fixes nur in orchestrator.py wirken nicht auf Haupt-Chat

---

## Dialekt-Weiche (P103)

- Marker-Länge ×5 (Teclado/Dictate-Pattern)|×2-Variante in `core/dialect.py` produzierte 400/500 (Offset-Bug)|IMMER ×5

---

## Frontend / JS in Python-Strings

- `'\n'` in Python-HTML-Strings wird als Newline gerendert + bricht JS-String-Literale (`SyntaxError: unterminated string literal`)|innerhalb `ADMIN_HTML="""..."""`/`NALA_HTML="""..."""` immer `'\\n'`/`\\r`/`\\t`|betrifft jeden Python-String mit JS|Erstmals P69c (`exportChat`), erneut P100 (`showMetricInfo`)

---

## SSE / Streaming Resilience (P109)

- Frontend-Timeout ≠ Backend-Timeout: 45s-Frontend-Timeout bricht `fetch()` ab|Backend arbeitet weiter + speichert Antwort via `store_interaction()`|Naiver Retry = doppelte LLM-Calls (Kosten + Session-Historie verwirrt)

---

## Theme-Defaults (P109)

- Anti-Invariante „nie schwarz auf schwarz": Bubble-Background-Defaults in `:root` müssen lesbare Werte haben, auch ohne Theme/Favorit|`rgba(…, 0.85-0.88)` mit Theme-Hex als Basis|User-Bubble `rgba(236, 64, 122, 0.88)` + LLM-Bubble `rgba(26, 47, 78, 0.85)` = Chris's Purple/Gold

---

## RAG GPU-Acceleration (P111)

- torch-Variante checken: `pip list | grep torch` zeigt `+cpu` wenn CUDA fehlt|RTX 3060 ungenutzt bis `pip install torch --index-url https://download.pytorch.org/whl/cu121`|Device-Helper aus P111 fällt defensiv auf CPU zurück (keine Regression, aber kein Speed-Up)

---

## Query-Router / Category-Boost (P111)

- Wortgrenzen-Matching IMMER: naives Substring-`in` findet `api` in `Kapitel` → false positive|`re.search(r'(?<!\w)kw(?!\w)', text.lower())` Pflicht|Multi-Wort-Keys (`"ich habe"`) fallen auf `in` zurück (Leerzeichen=Wortgrenze)|analog P103

---

## Auto-Category-Detection (P111)

- Extension-Map statt Content-Analyse: `Path(filename).suffix.lower()` → dict|Content-LLM-Call in Phase 4|`.json`/`.yaml`/`.md`→technical|`.csv`→reference|`.pdf`/`.txt`/`.docx`→general (Chris kann overriden)

---

## DB-Dedup / Insert-Guard (P113a)

- Dedup-Scope ≠ alle Rollen: `whisper_input` hat oft `session_id=NULL` (Dictate-Direct-Logging)|nur `role IN ('user','assistant')` mit konkreter session_id deduplizieren|Insert-Guard in `store_interaction` prüft session_id vorher → Whisper-Pipeline unberührt

---

## Whisper Sentence-Repetition (P113b / W-001b)

- Wort-Dedup ≠ Satz-Dedup: `detect_phrase_repetition` (P102) cappt N-Gramme bis 6 Wörter|ganze Sätze >6 Wörter rutschen durch|`detect_sentence_repetition` splittet an `(?<=[.!?])\s+` + dedupliziert konsekutive Sätze (case-insensitive, whitespace-collapsed)

---

## Whisper Timeout-Hardening (P160)

- httpx Default zu kurz: lange Aufnahmen (>10s) brauchen RTX 3060 bis 30s|Vor P160: `httpx.AsyncClient(timeout=60.0)` ohne Connect-Timeout|Fix: `httpx.Timeout(120, connect=10)` aus Config (`settings.whisper.request_timeout_seconds`)|Connect kurz (Docker-nicht-erreichbar ≠ Whisper-rechnet)

---

## Background Memory Extraction (P115)

- Overnight-Erweiterung statt eigener Cron: 04:30-APScheduler-Job aus P57 (BERT-Sentiment) wird um Memory-Extraction erweitert|Reihenfolge Sentiment→Extraction egal|read-only auf 24h-Nachrichten

---

## XSS / URL-Encoding (P116)

- `encodeURIComponent` für Query-Param mit Leerzeichen: `source=Neon Kadath.txt` bricht|`?source=Neon%20Kadath.txt` funktioniert|JS: `fetch('/…?source=' + encodeURIComponent(source))` — nie Template-Strings direkt

---

## Decision-Boxes + Feature-Flags (P118a)

- Marker-Parsing ohne `eval()`/innerHTML-Injection: Regex findet `[DECISION]…[/DECISION]`|`[OPTION:wert]` Label|Text außerhalb Marker → `document.createTextNode`|nur `<button>` strukturell gebaut|Kein `innerHTML` mit User/LLM-Content

---

## Dateinamen-Konvention (P100)

- Projektspezifische CLAUDE.md IMMER `CLAUDE_[PROJEKTNAME].md`|Supervisor `SUPERVISOR_[PROJEKTNAME].md`

---

## sys.modules-Test-Isolation (P169)

- `sys.modules["..."] = X` direkt setzen ist eine Falle|der Eintrag bleibt nach dem Test gesetzt, nachfolgende Tests sehen das Fake-Objekt statt des echten Moduls

---

## RAG-Status Lazy-Init (P169)

- Reine Read-Endpoints (`GET /admin/rag/status`, `GET /admin/rag/documents`) liefen ohne `_ensure_init`-Aufruf|Folge: Hel-RAG-Tab zeigte „0 Dokumente" bis zum ersten Schreibvorgang, danach erschien plötzlich der ganze Bestand

---

## HitL-Gate aus dem Long-Polling-Loop (P168)

- Direkt-`await` auf `wait_for_decision` im Telegram-Handler = Deadlock|Long-Polling-Loop awaited Updates SEQUENZIELL|der Click der das Gate auflöst kommt erst durch wenn der vorherige Handler returned|aber der vorherige Handler wartet auf den Click → Hängt bis Sweep-Timeout (5 min)

---

## Telegram sendDocument (P168)

- httpx-Multipart/form-data baut sich automatisch wenn man `files={"document": (filename, bytes, mime)}` + `data={...}` an `client.post(...)` gibt|kein zusätzlicher Encoder nötig

---

## Telegram-Bot hinter Tailscale (P155)

- Webhooks brauchen öffentliche HTTPS-URL|Tailscale MagicDNS (`*.tail*.ts.net`) löst nur intern|Self-Signed-Certs helfen nicht (DNS-Lookup scheitert vorher)

---

## Telegram Group Privacy (P155)

- BotFather GroupPrivacy default AN|AN = Bot sieht in Gruppen nur `@`-Mention oder `/`-Start|Für `respond_to_name`+`autonomous_interjection` MUSS AUS|sonst kommen Updates gar nicht per `getUpdates`, ganzer Gruppen-Flow läuft ins Leere, Symptom lautlos

---

## Vidar: Post-Deployment Smoke-Test (P153)

- 3 Test-Agenten: Loki=E2E (Happy-Path)|Fenrir=Chaos (Edge/Stress)|Vidar=Smoke (Go/No-Go)

---

## Design-Konsistenz (L-001, P151)

- `zerberus/static/css/shared-design.css` mit `--zb-*`-Namespace|Design-Tokens: Farben/Spacing/Radien/Schatten/Touch/Typography|`docs/DESIGN.md` als Referenz

---

## Settings-Singleton + YAML-Writer (P156)

- Jede `config.yaml`-Schreibfunktion MUSS Singleton invalidieren|sonst stale Wert

---

## Guard-Kontext pro Einsatzort (P158)

- Problem: Halluzinations-Guard (Mistral Small 3) zustandslos, kennt weder Antwortenden noch Umgebung|Persona-Selbstreferenzen (Huginn=Rabe, Nala=Zerberus) als Halluzination eingestuft|Antwort im Huginn-Flow komplett unterdrückt = Persona-Unterdrückung, kein Sicherheitsgewinn

---

## Huginn-Persona als technisches Werkzeug (P158)

- Sarkasmus = kalibrierte Beratung, kein Gimmick|zynisch-bissiger Ton der [`DEFAULT_HUGINN_PROMPT`](zerberus/modules/telegram/bot.py) = Kanal für Aufwands-Rückmeldung|„Schreib Pong in Rust" → Aufwand kommentieren, vernünftigeres Format vorschlagen (Python, 30 Zeilen), nach Bestätigung trotzdem liefern|Ton korreliert mit Absurdität/Aufwand

---

## Ratatoskr Sync-Reihenfolge (P158)

- Zerberus ERST commit+push, DANN `sync_repos.ps1`|nicht andersherum

---

## Huginn-Roadmap + Review-Referenz (P161)

- Ab P162 referenzieren Patch-Prompts Finding-IDs (`K1`, `O3`, `D8` etc.) statt Beschreibung|Nachschlagen [`docs/huginn_review_final.md`](docs/huginn_review_final.md)|Sequenz+Phasen [`docs/huginn_roadmap_v2.md`](docs/huginn_roadmap_v2.md)

---

## Input-Sanitizer + Telegram-Hardening (P162)

- `structlog` NICHT installiert|Patch-Prompts aus 7-LLM-Review schlagen oft `structlog.get_logger()` vor|Zerberus nutzt `logging.getLogger(...)`|nicht blind übernehmen, auf vorhandenes Setup adaptieren (Stil wie [`bot.py`](zerberus/modules/telegram/bot.py)/[`hitl.py`](zerberus/modules/telegram/hitl.py))

---

## Token-Effizienz bei Doku-Reads (P163)

- Keine rituellen File-Reads|`SUPERVISOR_ZERBERUS.md`/`lessons.md`/`CLAUDE_ZERBERUS.md` werden via CLAUDE.md schon in den Kontext geladen|Re-Read = 2-4k Token verschwendet pro Patch

---

## Intent-Router via JSON-Header (P164)

- Architektur-Entscheidung: Intent kommt vom Haupt-LLM via JSON-Header in der eigenen Antwort, NICHT via Regex/Classifier|Whisper-Transkriptionsfehler machen Regex unbrauchbar|Extra-Classifier-Call verdoppelt Latenz

---

## Auto-Test-Policy (P165)

- Mensch=unzuverlässiger Tester|Coda=systematisch+unbestechlich

---

## HitL-Hardening (P167)

- HitL-State im RAM = Datenverlust bei Restart|jede Reservierung muss in SQLite|`hitl_tasks`-Tabelle mit UUID4-Hex-PK ist Source-of-Truth|In-Memory-Cache nur Beschleuniger + `asyncio.Event`-Notifizierung

---

## Log-Hygiene + Repo-Sync-Verifikation (P166)

- Routine-Heartbeats fluten Terminal|Pacemaker-Puls/Watchdog-Healthcheck/Audio-Transkripte → DEBUG, nicht INFO|sichtbar bleibt nur Start/Stop/Problem

---

## Sync-Pflicht nach jedem Push (P164)

- Coda-Setup pusht zuverlässig nach Zerberus, vergisst aber `sync_repos.ps1`|Ratatoskr+Claude-Repo driften unbemerkt

---

## Rate-Limiting + Graceful Degradation (P163)

- Per-User-Limit gegen Telegram-Spam: 10 msg/min/User|Cooldown 60s|InMemory-Singleton in [`core/rate_limiter.py`](zerberus/core/rate_limiter.py) mit Interface `RateLimiter` (Rosa-Skelett für Redis-Variante) + `InMemoryRateLimiter` (Huginn-jetzt)|`first_rejection`-Flag liefert genau EIN „Sachte, Keule"-Reply, danach still ignorieren

---

## Mega-Patch-Erkenntnisse (Sessions 122-152, 2026-04-23/24)

- 33k Token/Patch bei 8 Patches → 24k/Patch bei 16 Patches|je mehr Patches im selben Fenster, desto weniger Overhead (Codebasis nur einmal eingelesen)

---

## Live-Test-Erkenntnisse Patch 178 (Huginn-Selbstwissen)

- L-178a Guard kennt RAG nicht|Guard-Call (Mistral Small 3) prüft LLM-Antworten ohne Wissen über RAG-Inhalte|Korrekte RAG-Antworten (z.B. "Patch 178", "981 Tests") werden als Halluzination geflaggt|Fix: Guard-Call braucht RAG-Chunks als Referenz-Kontext (stateless bleiben)

---

## Guard + RAG-Kontext (P180)

- Guard muss wissen, was dem Antwortenden als Referenz zur Verfügung stand|Sonst: jede RAG-basierte Antwort liest sich für den Guard wie erfundene Fakten ("woher kommt 'Tailscale'? steht nicht in der User-Frage")|Fix in P180: `check_response(rag_context=...)` reicht den RAG-Lookup-String an den System-Prompt durch, mit explizitem Marker "Fakten aus diesem Referenz-Wissen sind KEINE Halluzinationen"|Truncation bei 1500 Zeichen schützt das knappe Mistral-Small-Token-Budget — der Guard braucht keine vollen Chunks, nur genug um Material wiederzuerkennen

---

## Telegram-User-Allowlist (P181)

- Allowlist mit leerer `allowed_users`-Liste = alle erlaubt (Safety-Fallback)|Sonst wäre eine vergessene Liste in `mode: allowlist` ein Total-Lock-Out — niemand kommt rein, auch der Admin nicht (wenn er nicht selbst die Liste pflegt)|Wer wirklich nur den Admin will, soll auf `mode: admin_only` umstellen|Logging-Konsequenz: leere Liste loggt KEINEN Block, weil intern als "open" behandelt — sieht in Logs aus, als wäre die Allowlist inaktiv (ist sie effektiv auch)

---

## ADMIN-Intent Plausibilitäts-Heuristik (P182)

- ADMIN-Verdict wird auf CHAT downgegradet wenn der User-Text keine Admin-Marker hat (P182)|Vorher klassifizierte das LLM auch "Wie geht's dir?" als ADMIN, was den HitL-Button auslöste — nervt mehr als es schützt (False-Positive-Rate hoch)|Heuristik: Slash-Prefix (`/status`, `/help`) ODER Admin-Keyword (`status`, `restart`, `config`, ...) → bleibt ADMIN, sonst → CHAT|Backward-Compat: Plausi-Check greift nur wenn der Caller `user_message` mitgibt — alle bestehenden Tests bleiben grün, nur der Telegram-Router profitiert (er gibt user_msg explizit mit)

---

## Unsupported-Media-Handler (P182)

- Voice/Audio/Sticker/Document/Video → freundliche Absage statt lautloses Verschlucken (P182)|Vorher fiel ein Voice-Message-Update durch alle Filter und kam in `_process_text_message` als "empty" raus — der User sah keine Reaktion und wartete ins Leere|Jetzt: kurze Erklärung "kann ich noch nicht verarbeiten" + `reply_to_message_id` damit die Absage als Antwort auf die Voice-Message erscheint|Telegram-API-Detail: `message.voice`/`message.audio`/`message.sticker` etc. sind eigene Top-Level-Felder, nicht in `message.text`/`caption`

---

## Architektur-Mismatch zwischen Patch-Spec und Code (P182)

- Patch-Spec sprach von "ADMIN-Confidence ≥ 0.85", aber der Intent-Parser liefert keine Confidence (P182)|Architektur: das LLM emittiert harte Labels (`{"intent": "ADMIN", "needs_hitl": true}`), keine Wahrscheinlichkeiten|Konsequenz für die Umsetzung: das Konzept "höhere Schwelle" wurde umgedeutet auf "höhere Plausibilität" — Heuristik-basierter Downgrade statt numerischer Threshold|Lehre: bei Patches die eine Architektur voraussetzen die nicht existiert, lieber pragmatisch reinterpretieren als vortäuschen die Architektur sei da|Im Patch-Body explizit dokumentieren, dass die Spec angepasst wurde + Begründung

---

## Prosodie / Audio (P188-191)

- Whisper verwirft Prosodie (Pitch/Tempo/Stress)|Parallel-Pipeline mit Gemma E2B noetig|Whisper NICHT ersetzen (spezialisiert auf Wort-Erkennung, schlecht in Stimm-Analyse)

---

## RAG / FAISS (P187+)

- DualEmbedder aktiv seit P187|DE: `T-Systems-onsite/cross-en-de-roberta-sentence-transformer` (GPU)|EN: `intfloat/multilingual-e5-large` (CPU)|Flag `modules.rag.use_dual_embedder` (Default `false` in `config.yaml.example`, lokal seit 2026-05-01 `true`)|MiniLM-Pfad bleibt als Fallback erhalten

---

## Frontend (P186+P192)

- Auto-TTS triggert NACH `addMessage(reply, 'bot')` (entspricht SSE-done-Moment im non-streaming Pfad)|nicht pro Chunk|Audio-Stop bei: `loadSession`, `doLogout`, `handle401`, Toggle-OFF|`window.__nalaAutoTtsAudio`-Pattern analog SSE-Watchdog

---

## Whisper-Endpoint Enrichment (P193)

- `/v1/audio/transcriptions` Response erweitert: `text` bleibt IMMER (Backward-Compat fuer Dictate, SillyTavern, Generic-Clients)|optional `prosody` (P190) und `sentiment` mit `bert.{label,score}` + optional `consensus.{emoji,incongruent,source}`|`compute_consensus(bert_label, bert_score, prosody)` zentrale Helfer-Funktion fuer alle Pfade

---

## Datei-Storage / Multipart-Upload (P196)

- Beim Schreiben von Upload-Bytes IMMER atomar: `tempfile.mkstemp(dir=target.parent)` + `os.replace(tmp, target)`|`tempfile.mkstemp` muss IM Ziel-Ordner liegen, sonst macht `os.replace` einen Cross-Volume-Move statt atomarem Rename|Server-Kill mitten im Schreiben hinterlaesst dann hoechstens ein `.upload_*.tmp`, kein halbgeschriebenes Ziel|Im Fehlerfall Temp-File im `except` aufraeumen, sonst sammelt sich Muell

---

## 🪦 Der Schwarze Bug — Post Mortem (P109 → P153 → P169 → P183)

- Bei wiederkehrenden Bugs NIEMALS an der Symptomstelle patchen|erst ALLE Codepfade inventarisieren

---

## Persona-Layer-Merge (P197)

- Mehrstufige Persona (System → User → Projekt) NICHT als monolithischen String aus 3 Quellen kombinieren|stattdessen einen markierten Block pro Layer dranhängen|`[PROJEKT-KONTEXT — verbindlich für diese Session]` als eindeutiger Marker → Substring-Check für Tests/Logs + Schutz vor Doppel-Injection in derselben Pipeline

---

## Template-Generierung beim Anlegen (P198)

- Templates als regulaere `project_files`-Eintraege im SHA-Storage (nicht als separater Sonder-Pfad)|gleicher Storage-Layout wie Uploads → Hel-Datei-Liste, RAG-Index, Code-Execution-Pipeline sehen sie ohne Spezialfall|spart eine zweite Persistenz-Schicht und verhindert Drift zwischen "Template-Files" und "User-Files"

---

## Per-Projekt-RAG-Index (P199)

- Pure-Numpy-Linearscan ist fuer kleine Per-Projekt-Indizes (≤10k Chunks) FAISS ueberlegen|`argpartition(-sims, k-1)[:k]` + `argsort` liefert exakt das gleiche Top-K wie ein FAISS-FlatL2-Search auf normalisierten Vektoren|Tests laufen dependency-frei (kein faiss-Mock), Setup-Overhead ist Null, Persistierung als `vectors.npy` + `meta.json` ist trivial|FAISS-Wechsel spaeter ist 10-Zeilen-Tausch in `top_k_indices` + `load_index`/`save_index`, Public-API bleibt

---

## Project-Workspace-Layout (P203a)

- Hardlink-primary + Copy-Fallback bei `OSError` ist das richtige Pattern fuer FS-Spiegelung von SHA-Storage in begehbares Layout|`os.link` ist instantan + 0-Byte-Overhead (gleiche Inode), aber scheitert bei cross-FS, FAT32, NTFS-without-dev-mode, Permission-Denied|`shutil.copy2` als sicherer Fallback|gewaehlte Methode IM RETURN ausweisen (`"hardlink"` / `"copy"` / `None`-noop) damit Tests verifizieren koennen WELCHER Pfad lief — auf Windows-Maschinen ohne dev-mode wird der Copy-Pfad damit live mitgetestet (Monkeypatch-Test simuliert `os.link`-Failure)

---

## Sandbox-Workspace-Mount (P203c)

- Container-Mounts: Read-Only ist der konservative Default, Read-Write nur explizit per separatem Flag|`mount_writable: bool = False` zwingt den Caller, sich Gedanken ueber Sync-Back zu machen|der Sandbox-Code kann Files lesen (Source-Trees, Config, Daten), aber das Workspace nicht von innen veraendern, ohne dass die ausfuehrende Schicht es ausdruecklich zulaesst|gilt analog fuer DB-Connections (read-only-Replica vs. write-Master), Cloud-Storage (signed-GET vs. signed-PUT), File-Handles (`'r'` vs. `'w'`) — Default-RO ist Industrie-Standard

---

## Code-Detection im Chat-Endpunkt (P203d-1)

- Sechs-Stufen-Gates sind gut testbar|Wenn jeder Skip-Pfad eine eigene Stufe hat, kann jeder Skip-Fall einzeln getestet werden — bei P203d-1 sieben Skip-Tests + zwei Happy-Path-Tests + zwei Edge-Cases statt einem monolithischen "wenn X dann Y"-Block|Pattern: jede Bedingung als eigener `if`/`return None`-Schritt, nicht ein langer `if A and B and C and D and E and F`-Ausdruck — der Debug-Output bei einem Skip ist dann sofort klar (welches Gate hat geblockt?), und Tests koennen je Gate gezielt mocken

---

## HitL-Gate vor Code-Execution (P206)

- Long-Poll-innerhalb-Request + separater Resolve-Endpoint ist das saubere Pattern wenn die UI eine Bestaetigung braucht und das Backend non-streaming ist|Chat-Completions blockt via `await asyncio.wait_for(event.wait(), timeout=N)` bis Frontend `POST /v1/hitl/resolve` sendet|gleiches Pattern fuer jede zukuenftige User-Bestaetigung in einer einzelnen Request-Pipeline (Diff-Approval bei P207, Spec-Confirmation bei P208) — kein WebSocket-Setup, kein SSE-Frame, nur ein zusaetzlicher Auth-freier Endpoint

---

## Destruktive Operationen — Industrie-Lektionen (2025-2026)

KERN-ERKENNTNIS: LLM-Agenten haben kein Bauchgefühl|führen aus was logisch

---

## PowerShell 5.1 Native-Argument-Quoting (P217-pre)

- Inline-`-m "$var"` an native Tools (`git commit`, `docker run`, `curl -d`) ist in PowerShell 5.1 ein Quoting-Loch|wenn der Variableninhalt selbst `"` enthaelt, escaped der Win32-CommandLineToArgvW-Pfad das nicht|der Aufruf zerlegt sich in mehrere Argumente, das Tool sieht Muell|Bug springt nur an wenn der Inhalt zufaellig Quotes enthaelt — klassisches "laeuft heute, kracht in drei Wochen"|robust: Inhalt in Temp-Datei schreiben + Tool mit `-F file`/`-T file`/`@file`-Loader aufrufen|Datei-Inhalt geht NICHT durch den Argument-Quoter, kein Escape-Problem mehr|UTF-8 OHNE BOM (`[System.Text.UTF8Encoding]::new($false)`), sonst landet die BOM im Inhalt|`try/finally` mit `Remove-Item -Force -ErrorAction SilentlyContinue` raeumt die Temp-Datei auch im Fehlerfall auf|PowerShell 7.3+ hat `PSNativeCommandArgumentPassing` und wuerde das von selbst escapen — wer auf 5.1 baselined ist (Default auf Windows 11), muss explizit den Datei-Pfad waehlen
