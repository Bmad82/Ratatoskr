# SUPERVISOR_ZERBERUS.md – Zerberus Pro 4.0
*Strategischer Stand für die Supervisor-Instanz (claude.ai Chat)*
*Letzte Aktualisierung: P-debt-14 (2026-05-10) — Test-Fix fuer `test_fenrir_mega_patch.py::TestPacemakerStress::test_f_pace_01_master_toggle_stable` (Schulden #3 GELOEST, Test-only). Test skipte konsistent seit Patch 99 (H-F01), weil das Hel-UI-Akkordeon durch eine Tab-Leiste ersetzt wurde und `.hel-section[data-tab='sysctl']:not(.active)` `display: none` ist. Fix: `activateTab('sysctl')`-Click VOR dem Master-Toggle-Click, analog zum gruen laufenden `test_f_pace_02`. Davor: P-debt-13 (2026-05-10) — Stand-Anker-Regex `extract_current_patch` in `tools/sync_huginn_rag.py` um Buchstaben-Suffix-Patches erweitert (Schulden #12 GELOEST, ein-Zeilen-Wartungs-Patch). Regex `(P\d{3,4})\b` matchte vorher keine `P-UI-*`/`P-debt-*`/`P-feature-audit-*`-Patches — drei `TestDocSourceAudit`-Tests blieben seit dem ersten P-UI-*-Stand-Anker (2026-05-09 nach P-UI-Hel-Split) rot. Fix: Regex auf `(P[\w-]+)` erweitert + `extract_current_patch` upper-cased nur das fuehrende `P` (Original-Case-Praeservierung fuer Integration-Lookups). Plus 12 neue Source-Audit-Tests in `test_p210_huginn_rag_sync.py`. **Schulden #12 GELOEST.** Davor: P-feature-audit-mt (2026-05-10, Manuelle-Tests-Tab in `feature_audit_checklist.html`), P-start-stable (2026-05-10, Reload+Force-Shutdown in `start_stable.bat`), P-feature-audit (2026-05-10, Initialwurf `scripts/feature_audit.py`), P-debt-12 (2026-05-10, Leitregel-Anker in DESIGN.md, Schulden #11 GELOEST), Schulden #9 (Hel-Splitscreen) GELOEST nach Chris' visueller Verifikation am 2026-05-10. 92/92 in `test_p210_huginn_rag_sync.py` gruen, 121/121 in der huginn-doc/design-system-Smoke-Suite gruen, 4 pre-existing Worktree-Drift-Failures wegen `faiss`-Modul (Schulden #6, im Hauptrepo gruen). Phase 5a + Phase 5c bleiben VOLLSTAENDIG ABGESCHLOSSEN. Letzter UI-Bugfix-Patch P-UI-Hel-Split. Letzter Backend-Bugfix-Patch P217. Letzter Code-Patch mit UI-Mechanik P-UI-11.*

---

## Letzter Maintenance-Patch (Test, Fenrir-Test-Fix)

**P-debt-14** — Test-Fix fuer `test_fenrir_mega_patch.py::TestPacemakerStress::test_f_pace_01_master_toggle_stable`, Schulden #3 (2026-05-10)

Test-only Wartungs-Patch. Der Fenrir-E2E-Test fuer Pacemaker-Master-Toggle skipte konsistent seit Patch 99 (H-F01, 2025) auf der Diagnose "Pacemaker-Master-Toggle nicht gefunden oder nicht sichtbar". Patch 99 hat das Hel-UI-Akkordeon durch eine Tab-Leiste ersetzt — `.hel-section[data-tab]:not(.active) { display: none; }` in [`zerberus/app/routers/hel.py:229`](zerberus/app/routers/hel.py:229). Der `#pacemaker-master`-Toggle ist im DOM (HTML hardcoded), aber unsichtbar solange der `sysctl`-Tab nicht aktiv ist.

**Fix.** `activateTab('sysctl')`-Click via Tab-Locator (`.hel-tab[data-tab='sysctl'], .hel-tab[data-tab='system']`) VOR dem Master-Toggle-Click eingefuegt — exakt das Pattern aus dem bereits gruen laufenden `test_f_pace_02_page_stays_responsive` (Z. 386-400). Plus `page.wait_for_timeout(500)` damit der Lazy-Load von `loadPacemakerProcesses()` (Z. 1508 in hel.py) Zeit hat den Master-State zu setzen. Erweiterter Docstring erklaert Schulden-#3-Hintergrund + Patch-99-Akkordeon-Umbau.

**Tests.** Test ist `@e2e`-markiert und im Default-Run uebersprungen (pytest.ini `addopts = -m "not e2e"`). Im Worktree ohne laufenden Server nicht live verifizierbar — Schulden #6 (faiss/config.yaml/SSL-Cert-Drift). Statische Verifikation: AST parse OK, Pattern eins-zu-eins aus `test_f_pace_02` kopiert, Selektor-Validitaet bestaetigt durch Grep in hel.py (Z. 661: `<button class="hel-tab" data-tab="sysctl" onclick="activateTab('sysctl')">`). Live-Verifikation kommt bei Chris naechstem Loki/Fenrir-Sweep auf dem Hauptrepo-Server.

**Status-Updates parallel.** `docs/huginn_kennt_zerberus.md` + Spiegel-Kopie auf P-debt-14 gebumpt. **Schulden #3 auf GELOEST gesetzt.** Phase 5a + Phase 5c bleiben VOLLSTAENDIG ABGESCHLOSSEN.

**Lesson.** UI-Umbau-Patches (z.B. Patch 99 Akkordeon → Tab-Leiste) erzeugen E2E-Test-Drift, der erst Monate spaeter auffaellt — die meisten betroffenen Tests skippen statt failen, also faellt der Drift nicht in den Default-Test-Run rein. Backstop: bei UI-Refactors aktiv `pytest -m e2e -k <betroffener_Modul>` laufen lassen und Skip-Outputs sichten; eine Skip-Meldung "UI-Layout-Drift" ist immer ein Test-Update-Auftrag, nicht ein Mark-as-skip-Asset.

---

## Davor (Maintenance, Tool, Regex-Erweiterung)

**P-debt-13** — Stand-Anker-Regex `extract_current_patch` in `tools/sync_huginn_rag.py` um Buchstaben-Suffix-Patches erweitert, Schulden #12 (2026-05-10)

Tool-only Wartungs-Patch aus dem Vorgaenger-HANDOVER (P-feature-audit-mt). Die Regex `\*\*Letzter Patch:\*\*\s+(P\d{3,4})\b` in [`tools/sync_huginn_rag.py:138`](tools/sync_huginn_rag.py:138) matchte nur rein-numerische Patches; Buchstaben-Suffix-Patches aus dem Phase-5c-UI-Track + Maintenance-Stack — `P-UI-Hel-Split`, `P-UI-11`, `P-debt-11`, `P-debt-12`, `P-feature-audit`, `P-start-stable`, `P-feature-audit-mt` — wurden NICHT erkannt, weil `\d{3,4}` keinen Bindestrich akzeptiert. Drei `test_p210_huginn_rag_sync::TestDocSourceAudit`-Tests waren seit dem ersten P-UI-*-Stand-Anker (2026-05-09) pre-existing rot, markiert in der HANDOVER-Schulden-Liste als #12.

**Fix.** Regex auf `r"\*\*Letzter Patch:\*\*\s+(P[\w-]+)"` erweitert. Plus minimaler Verhaltens-Tweak in `extract_current_patch`: statt `m.group(1).upper()` jetzt `raw[0].upper() + raw[1:]` — nur das fuehrende `P` wird zu Grossbuchstaben gewandelt, der Rest bleibt im Original-Case. Hintergrund: Integration-Lookups (`test_integration_huginn_rag.py::_patch_in_context`) matchen den Patch-String gegen den Doku-Inhalt; `P-debt-12` im Stand-Anker darf nicht zu `P-DEBT-12` werden, sonst greift `expected_patch in ctx` nicht mehr.

**Tests.** 12 neue Tests in [`zerberus/tests/test_p210_huginn_rag_sync.py`](zerberus/tests/test_p210_huginn_rag_sync.py) (7 in `TestExtractCurrentPatch`, 3 in `TestValidateDocHeader`) plus angepasstes `test_main_doc_patch_is_recent` (tolerant gegen Buchstaben-Suffix, akzeptiert pauschal als "recent"). **92/92 in `test_p210_huginn_rag_sync.py` gruen** — heilt die drei Schulden-#12-Failures. Wider-Smoke ueber `test_p210_huginn_rag_sync + test_p213_pre_4_huginn_ranking + test_design_system`: **121/121 gruen**. Vier pre-existing Failures in `test_p213_pre_2_dual_storage.py` + `test_p213_pre_3_reindex_atomic.py` sind Worktree-Setup-Drift (`faiss`-Modul fehlt im Worktree, Schulden #6) — nicht durch P-debt-13 verursacht.

**Status-Updates parallel.** `docs/huginn_kennt_zerberus.md` + Spiegel-Kopie `docs/RAG Testdokumente/huginn_kennt_zerberus.md` auf P-debt-13 gebumpt (Stand-Anker + FAQ-Antwort). **Schulden #12 auf GELOEST gesetzt.** Phase 5a + Phase 5c bleiben VOLLSTAENDIG ABGESCHLOSSEN — P-debt-13 ist Tool-only-Wartung ausserhalb der Phase-Ziele.

**Lesson.** Stand-Anker-Format-Drift gehoert hinter Source-Audit-Tests: wer das Format aendert (z.B. von numerisch zu Buchstaben-Suffix), muss das parsende Tool — und seine Source-Audit-Tests — synchron mitziehen. Backstop: `validate_doc_header` + neue Buchstaben-Suffix-Source-Audit-Tests pruefen jetzt beide Formen.

---

## Davor (Maintenance, Doku-Anker-Reparatur)

**P-debt-12** — Leitregel-Anker in `docs/DESIGN.md` wiederhergestellt, Schulden #11 (2026-05-10)

Doku-only Wartungs-Patch. Schulden #11-Aufrueumung aus dem P-UI-Hel-Split-HANDOVER. `test_design_system::test_design_md_enthaelt_regel` (Patch 151, L-001) suchte drei Strings in `docs/DESIGN.md`: `Leitregel` ODER `projektübergreifend`, sowie `44` und `shared-design.css`. Bei P-UI-meta (2026-05-07) wurde DESIGN.md komplett neu gefasst (alte 77-Zeilen-Datei → 601-Zeilen-Globale-Design-Referenz), wobei die `## Leitregel`-Sektion mit allen drei Ankern verschwand. Test war seitdem rot, drei Wochen unbemerkt — Vorgaenger-HANDOVER nach P-UI-Hel-Split hatte ihn als pre-existing Failure markiert.

**Edit in [`docs/DESIGN.md`](docs/DESIGN.md):** ein neuer `## Leitregel`-Block direkt nach der Einleitung, vor `## 1. Farbsystem`. Inhalt re-interpretiert die ursprungliche Spec aus Patch 151 (L-001 / B-026): "Wenn eine Designentscheidung fuer ein UI-Element getroffen wird, gilt sie **projektübergreifend** fuer alle aehnlichen Elemente in Nala **und** Hel. Vor einer CSS-/UI-Aenderung erst bestehende Patterns pruefen — keine neuen erfinden, wenn es schon ein bestehendes gibt. Konkrete Tokens (Farben, Spacing, Radius, Touch-Min `44px`) sind in `zerberus/static/css/shared-design.css` zementiert und werden von Nala und Hel via `<link rel="stylesheet" href="/static/css/shared-design.css">` geladen." Der Block traegt alle drei Test-Anker organisch im Inhalt: `Leitregel` als Sektions-Headline, `projektübergreifend` als Adverb in der Spec, `shared-design.css` als Pfad-Verweis im Token-Verteilungs-Hinweis, `44px` weiterhin als Touch-Min-Hinweis.

**Tests.** `python -m pytest zerberus/tests/test_design_system.py` → **12/12 gruen** (vorher 11/12, jetzt heilt `test_design_md_enthaelt_regel`). UI-/Hel-/Phase-5c-/huginn-doc-Regressions-Suite (12 Test-Files): **426 passed**. Verbleibende 10 Failures sind pre-existing und nicht durch P-debt-12 verursacht — 7× `test_patch185_runtime_info` config.yaml-Worktree-Drift (Schulden #6), 3× `test_p210_huginn_rag_sync::TestDocSourceAudit` neue Schulden #12 (Stand-Anker-Regex `P\d{3,4}` matched `P-UI-Hel-Split` nicht — pre-existing seit P-UI-Hel-Split, via `git stash`-Verifikation bestaetigt nicht durch P-debt-12 verursacht).

**Status-Updates parallel.** **Schulden #9 (Hel-Splitscreen) auf GELOEST gesetzt** — Chris hat am 2026-05-10 visuell verifiziert: "Split Screen habe ich getestet. Funktioniert. […] der Split-Screen war zur Haelfte schwarz und das ist jetzt nicht mehr." `UI_BUG_HEL_SPLITSCREEN.md` aktualisiert. **Schulden #12 dokumentiert (mittlerweile GELOEST durch P-debt-13, 2026-05-10)** — `test_p210_huginn_rag_sync::TestDocSourceAudit::{test_main_doc_has_stand_anker, test_main_doc_patch_is_recent, test_mirror_doc_has_stand_anker}` blockierten auf der Regex `\*\*Letzter Patch:\*\*\s+(P\d{3,4})\b` in [`tools/sync_huginn_rag.py:138`](tools/sync_huginn_rag.py:138). Patch-Namen mit Buchstaben-Suffix wie `P-UI-Hel-Split`, `P-UI-11`, `P-debt-11`, `P-debt-12` matchten nicht. P-debt-13 hat die Regex auf `r"\*\*Letzter Patch:\*\*\s+(P[\w-]+)"` erweitert.

**Lesson.** `lessons.md` neue Sektion "Test-zementierte Doku-Anker muessen Doku-Refactors ueberleben (P-debt-12)": wer eine Doku-Datei komplett neu fasst (full rewrite), muss die test-zementierten Anker explizit pruefen — `grep -E 'assert.*"' tests/test_<doku>.py` zeigt welche Strings die Tests erwarten. Beim Schreiben der neuen Doku diese Strings als Pflicht-Inhalt mitnehmen oder den Test mit-aktualisieren. Backstop: bei jedem Doku-Refactor die zugehoerige Test-Suite mitlaufen lassen, bevor der Refactor ausgeliefert wird.

---

## Letzter UI-Bugfix-Patch

**P-UI-Hel-Split** — Defensive CSS-Hardening fuer Hel-UI im Splitscreen-Modus, Schulden #9 (2026-05-09)

UI-Bugfix-Patch nach Bug-Report von Chris in `UI_BUG_HEL_SPLITSCREEN.md` (gemeldet 2026-05-07): Hel-Oberflaeche wird im Splitscreen-Modus (Win+Pfeil oder DevTools-geteiltes Browser-Fenster) "in der Mitte abgeschnitten" — auch bei reichlich Breite/Hoehe. Vermutung im Bug-Report: CSS-Layout mit fixer Mittel-Breite oder absolute-Positionierung statt responsive Grid/Flex.

**Diagnose nach Code-Read in [`zerberus/app/routers/hel.py`](zerberus/app/routers/hel.py)** (UI ist nicht in Templates/CSS, sondern Inline-Style-Block im Router, 6548 Zeilen):

1. Vier `<div style="display:grid;grid-template-columns:1fr 1fr;...">`-Inline-Style-Grids und ein `1fr 1fr 1fr`-Inline-Style-Grid existieren. Bei Splitscreen-Breite (typisch 600-900 px) bleibt `1fr 1fr` zwei Spalten á ~280-440 px. Wenn ein Inhalt (langer Input, breite Tabelle, Chart-Container) eine `min-content > Spaltenbreite` hat, treibt es das Grid horizontal ueber den Container. `body` ist Default-`overflow: visible`, der Overflow rastet auf dem nahesten scrolling ancestor — was im Hel-Layout der html-Layer ist. Resultat: horizontaler Scrollbar plus optisch der Eindruck "Mitte abgeschnitten" (rechte Haelfte des sichtbaren Bereichs ist Overflow).
2. `.hel-tab-nav` ist `position: sticky; top: 0` mit `margin: 0 -20px 14px -20px` — ragt 20 px ueber den Container nach links/rechts hinaus. Bei `body { padding: 20px }` ist das genau die Body-Innenkante. Bei reduziertem Padding (Splitscreen) bleibt der negative Margin auf 20 px, was Tab-Nav 8 px ueber die Body-Padding-Grenze treibt.
3. Container-Wurzel `.container { max-width: 1400px; margin: 0 auto }` ohne explizites `width: 100%` — verlaesst sich auf Block-Default. Im Splitscreen-Browser-Quirk (insbesondere Edge mit Snap-Layouts) kann das Block-Default auf `intrinsic-width` rasten, was bei langer Inline-Inhalt den Container ueber die Viewport-Breite treibt.

**Fix.** Defensive CSS-Schicht — kein Refactor der Inline-Style-Grids (Memory-Regel "Don't add abstractions beyond what the task requires"), sondern reine CSS-Erweiterung:

- **Edit 1** (html/body/container Hardening): `html { box-sizing: border-box; min-height: 100%; overflow-x: hidden; }` als Boden-Sperre, `body { ... min-height: 100vh; overflow-x: hidden; }` als zweite Schicht, `.container { ... width: 100%; }` als explizite Breiten-Anker.
- **Edit 2** (Splitscreen-Breakpoint): `@media (max-width: 900px)` mit `body { padding: 12px }`, `.hel-section-body { padding: 14px }`, `.card { padding: 14px; margin-bottom: 14px }`, `.hel-tab-nav { margin: 0 -12px 14px -12px; padding: 6px 12px 0 12px }`, `.container > * { min-width: 0 }` (Flex-/Grid-Items-Schrumpf-Befreiung), `[style*="grid-template-columns:1fr 1fr 1fr"] { grid-template-columns: 1fr !important }` (3-Spalten-Stack — MUSS vor 2-Spalten-Selektor stehen, sonst matched Substring `1fr 1fr` auch `1fr 1fr 1fr`), `[style*="grid-template-columns:1fr 1fr"] { grid-template-columns: 1fr !important }` (2-Spalten-Stack), `#messagesTable td { max-width: clamp(120px, 32vw, 200px) }` (Cell-Limit skaliert mit Viewport-Breite, default 200px war zu breit fuer narrow viewports).

**Tests.** Neuer File [`zerberus/tests/test_p_ui_hel_split.py`](zerberus/tests/test_p_ui_hel_split.py) mit **36 Source-Audit-Tests in 12 Klassen** — `TestHtmlLayerHardening` (4), `TestBodyLayerHardening` (3), `TestContainerHardening` (3), `TestSplitscreenMediaQueryExists` (3), `TestSplitscreenPaddingReduktion` (3), `TestSplitscreenTabNavMarginKorrektur` (2), `TestSplitscreenGridStacking` (3, inkl. expliziter Reihenfolge-Test 3-vor-2), `TestSplitscreenContainerMinWidthEscape` (1), `TestSplitscreenMessagesTableClamp` (1), `TestKeineRegressionVorgaengerCSS` (8 — landscape-media bleibt, Tab-Nav-Default-margin -20px bleibt, font-preset-bar bleibt, sticky bleibt, Chart-Container 280px bleibt, container-Div bleibt, ADMIN_HTML-Const bleibt, NUR EIN html { ... }-Block), `TestKeineRegressionPhase5c` (2 — nala.py unangetastet, Marker nicht in nala.py), `TestPUiHelSplitInlineMarker` (3 — Marker mind. zweimal, Schulden-#9-Referenz). 36/36 lokal gruen.

**UI-relevante Test-Suite Regressions-Check** (10 Hel-bezogene Files + 7 P-UI-Files): 286/286 gruen, 1 pre-existing Failure in `test_design_system::test_design_md_enthaelt_regel` (sucht `Leitregel`/`projektuebergreifend` in DESIGN.md). Via git-stash-Verifikation bestaetigt: Failure existiert auch ohne P-UI-Hel-Split — pre-existing Schuld in DESIGN.md (Test wurde zementiert, aber DESIGN.md enthaelt keinen dieser Strings mehr). Nicht durch P-UI-Hel-Split verursacht.

**Visueller Verifikations-Schritt fuer Chris (in HANDOVER vermerkt).** Hel-UI im Win+Pfeil-Splitscreen-Modus oeffnen, pruefen ob:
- (1) horizontaler Scrollbar verschwindet
- (2) keine "Mitte abgeschnitten"-Sicht mehr
- (3) 2-Spalten-Inline-Style-Grids (Sysctl/Provider/Huginn-Tab) auf 1 Spalte gestackt sind
- (4) Body-Padding sichtbar reduziert (mehr Inhalt-Breite)
- (5) Sticky Tab-Nav rastet sauber an Body-Padding-Kante

Falls weiter Probleme sichtbar: eigener Folge-Patch **P-UI-Hel-Split-2** mit gezielterer Diagnose (Chris liefert live-Screenshot, Coda-Worktree hat keine Chrome-Extension verbunden — daher diese Session vorbeugende Defensive-Hardening, kein Live-Diagnose-Pfad).

**Status-Update fuer [`UI_BUG_HEL_SPLITSCREEN.md`](UI_BUG_HEL_SPLITSCREEN.md):** Status auf `TEILWEISE GELOEST P-UI-Hel-Split (defensive Hardening, Verifikation pending)`. HANDOVER-Schulden #9 entsprechend markiert. Vollstaendige Heilung folgt nach Chris' visueller Verifikation — Bug-Datei bleibt offen bis dahin.

**Lessons (2)**:
1. **Splitscreen-Bug-Reports ohne Live-Diagnose-Pfad: defensive Hardening mit Verifikations-Schritt ist konformer als blinder Fix.** Wenn Coda den Bug nicht visuell reproduzieren kann (Chrome-Extension nicht verbunden, Worktree-Setup-Drift), ist die ehrliche Loesung: bekannte Splitscreen-Anti-Patterns systematisch beheben (html/body overflow-x, Container-width-Hardening, Inline-Style-Grid-Stacking via Attribute-Selectors, Padding-Reduktion in Breakpoint), Source-Audit-Tests die Sub-Selektoren zementieren, klare visuelle Verifikations-Anweisung fuer Chris in HANDOVER. Vermeidet Raterei und macht den Patch-Erfolg messbar.
2. **Inline-Style-Grids brauchen Attribute-Selectors fuer Responsive-Override.** Wer im legacy-HTML `<div style="display:grid;grid-template-columns:1fr 1fr;...">` hat (vier Stellen in Hel allein), kommt mit normalen Klassen-Selektoren nicht ran. Die Loesung: `[style*="grid-template-columns:1fr 1fr"] { grid-template-columns: 1fr !important }` als CSS-Defense. Mit `!important` weil Inline-Styles sonst die Spezifitaets-Schlacht gewinnen. **Reihenfolge-Falle:** Substring-Match — `[style*="1fr 1fr"]` matched auch `1fr 1fr 1fr`. Daher MUSS der spezifischere 3-Spalten-Selektor VOR dem allgemeinen 2-Spalten-Selektor stehen, sonst ueberschreibt CSS-Source-Order das spezifischere Verhalten. Test `TestSplitscreenGridStacking::test_grid_3col_stack` zementiert diese Reihenfolge.

---

## Letzter Maintenance-Patch (Doku)

**P-debt-11** — Stand-Anker-Trim in `docs/huginn_kennt_zerberus.md` + Spiegel (2026-05-09)

Doku-only Wartungs-Patch. Schulden #6-Aufrueumung aus dem P217-HANDOVER. Vorgaenger-HANDOVER hatte 18 pre-existing Failures dokumentiert, davon 13 huginn_kennt_zerberus-bezogen — der RAG-Selbstwissen-Doku-Stand-Anker war ueber zehn aktive Patch-Eintraege gewachsen (P-UI-1..11 + P-UI-Polish-1/2 + P-debt-10 + P217), jeder mit 4-7 KB Detail-Beschreibung. Folge: der FAQ-Block (woertliche Stand-Fragen plus Antworten — der Embedding-Match-Garant aus P213-pre-4) wurde aus dem ersten 4000-Zeichen-Fenster geschoben. Plus: ein Patch-Eintrag enthielt den Triple-Backtick als Code-Marker-Beispiel (` ``` ` im Stand-Anker pollutet den ersten RAG-Chunk mit Code-Block-Syntax).

**Edits in [`docs/huginn_kennt_zerberus.md`](docs/huginn_kennt_zerberus.md)** + identischer Spiegel in [`docs/RAG Testdokumente/huginn_kennt_zerberus.md`](docs/RAG%20Testdokumente/huginn_kennt_zerberus.md):

1. **Stand-Anker auf kompakte Bullet-Liste reduziert.** Letzter Patch (P217) + Vorletzter (P-UI-Polish-2) + Vorvorletzter (P-UI-Polish-1) + Letzter Code-Patch (P-UI-11) + Letzter Maintenance (P-debt-11, vorvorletzt P-debt-10) + Letzter rein-Backend-Patch vor P217 (P213-pre-4) + Letzter Tooling (P217-pre): jeweils 1-2 Saetze. Plus „Fruehere Phase-5c-Patches (alle ✅)"-Bullet als reine Stichwort-Liste fuer P-UI-1..10. Phase-Status, Test-Zahl, Datum am Ende. Detail-Beschreibungen sind in `docs/PROJEKTDOKUMENTATION.md` (Patch-Historie, vollstaendige Doku) — RAG-Doku ist Chunk-getrieben, der erste Chunk ist der Embedding-Match-Hot-Spot.

2. **FAQ-Antworten gekuerzt.** Acht Frage/Antwort-Paare bleiben (`Bei welchem Patch sind wir`, `Wie ist der aktuelle Stand`, `Welche Phase`, `Welche Patch-Nummer`, `Was war der letzte Patch`, plus drei kontextuelle), Antworten sind 200-400 Zeichen statt 1000-3000 wie zuvor. Position des FAQ-Blocks: `Bei welchem Patch sind wir` bei Pos 3473, `Wie ist der aktuelle Stand` bei Pos 3902 — beide unter 4000 Zeichen (Test-Spec aus P213-pre-4).

3. **Triple-Backtick entfernt.** Drei Stellen: P-UI-11-Eintrag (` ``` ` als Code-Marker in der `compute_effort_score`-Beschreibung — verschwand durch Stand-Anker-Trim), VRAM-Block (`### VRAM-Belegung im Modus „Nala aktiv mit Prosodie"` mit ```pre-Block der Komponenten-Liste — in Inline-Text umgeschrieben).

4. **Neue Sektion „Aktuelle Konfiguration (Live-Werte zur Laufzeit, nicht in dieser Doku gepflegt)".** Verweist auf den dynamisch generierten `[Aktuelle System-Informationen]`-Block aus P185 (`zerberus/utils/runtime_info.py`), der zur Laufzeit an den System-Prompt von Huginn und Nala angehaengt wird. Statische Doku enthaelt bewusst keine Modellnamen oder Versionsnummern mehr — sonst werden veraltete Strings zurueck-halluziniert.

**Tests.** 4 huginn-doc-Failures jetzt gruen (vorher rot):
- `test_p213_pre_4_huginn_ranking::TestStandAnchorFAQForm::test_faq_block_appears_near_top` ✅
- `test_patch169_bugsweep::TestSelfKnowledgeDoc::test_doc_does_not_use_code_blocks` ✅
- `test_patch185_runtime_info::TestRagDocUpdate::test_doc_explains_runtime_block` ✅
- `test_patch185_runtime_info::TestRagDocUpdate::test_doc_does_not_hardcode_cloud_model` ✅

Plus 30 weitere huginn-doc-Tests in `test_p210_huginn_rag_sync` + `test_p213_pre_4_huginn_ranking` + `test_patch169_bugsweep` bleiben gruen (Doku-Trim haelt alle existing Pflicht-Strings: `Bei welchem Patch sind wir`, `Wie ist der aktuelle Stand`, `Welche Phase`, `Welche Patch-Nummer`, `Was war der letzte Patch`, mind. 5 `Frage:`/`Antwort:`-Marker, Kerberos-Negation, FIDO-Negation, Rosa+Red-Hat-Negation, Core-Komponenten, Mythologie-Terme, P213-pre-4 erwaehnt, Stand-Anker in ersten 1500 Zeichen, Letzter-Patch-Marker vor erstem `\n---\n`).

**Lessons (1)**: RAG-Doku unterliegt anderen Pflege-Regeln als Code-Doku. Code-Doku darf vollstaendig sein, RAG-Doku ist Chunk-getrieben — der erste Chunk ist der Embedding-Match-Hot-Spot, alles dahinter rankt schlechter. Stand-Anker + FAQ MUSS in den ersten ~4000 Zeichen liegen. Bei jedem neuen Patch der den Stand-Anker erweitert: pruefen ob die FAQ-Position noch passt. Wenn nicht: alten Patch-Eintrag in eine ein-Zeilen-Bullet-Liste reduzieren, Detail nach `PROJEKTDOKUMENTATION.md` verlagern.

---

## Letzter Backend-Bugfix-Patch

**P217** — Globaler RAG: Sprach-Routing + Chunk-Orphan-Bereinigung (2026-05-09)

Backend-Bugfix-Patch nach detailliertem Bug-Report von Chris. Zwei zusammenhaengende Bugs im globalen RAG-Pfad — Per-Projekt-RAG (MiniLM 384 dim) ist NICHT betroffen.

**Bug 1 — Index-Zerstoerung bei Dim-Mismatch.** DualEmbedder (P187) wechselt pro Chunk zwischen DE (`cross-en-de-roberta-sentence-transformer`, 768 dim, GPU) und EN (`multilingual-e5-large`, 1024 dim, CPU). Vor P217 routete `_add_to_index` jeden Vektor in den globalen DE-Slot (`_index`) — der erste EN-Vektor zerstoerte den DE-Index per P218-pre-Recovery (`_reset_index_inplace`), der naechste DE-Vektor zerstoerte den frisch-rekonstruierten 1024-Index wieder. Bei einem 50-Chunk-Upload mit ~10 Sprachwechseln blieben am Ende nur 5-10 Chunks uebrig.

**Bug 2 — Chunk-Orphans nach Dokument-Loeschung.** `rag_document_delete` markierte vor P217 nur DE-Metadata mit `deleted=True`. EN-Metadata blieb unangefasst (Orphans). Plus: Soft-Delete entfernt FAISS-Vektoren NICHT physisch (by design, weil `IndexFlatL2.remove(idx)` nicht existiert). `total_chunks` (= `_index.ntotal`) zaehlte alle physischen Vektoren — User loescht alle Doks, Hel zeigt "0 Doks aber 47 Chunks".

**Loesung.** "Zwei getrennte FAISS-Indizes pro Sprache" — der EN-Slot existierte seit P187, wurde aber nur im Search-Pfad genutzt. P217 macht den Add-Pfad sprach-aware:
- `_add_to_index(..., language=None)`: Bei `_use_dual=True` wird `language` (oder `_detect_lang(text)` wenn `None`) ausgewertet. EN → `_en_index` (`en.index`), DE → `_index` (`de.index`). Dim-Mismatch innerhalb eines Slots → nur DIESER Slot wird neu aufgebaut, der andere bleibt unangetastet.
- `_reset_index_inplace(target_dim, settings, language=None)`: sprach-aware, nur ein Slot betroffen.
- `_rebuild_index_atomic(chunks, settings, language=None)`: sprach-aware mit Pre-Backup pro Slot. Bei `chunks=[]` wird der Slot leer-reindiziert (Dim erhalten, ntotal=0).
- `_resolve_paths(..., for_write=True)`: beim Schreiben strikt nach `language` (kein None-Fallback). Verhindert dass erster EN-Schreibvorgang im DE-Pfad landet.
- `_reset_sync`: leert `en.index`/`en_meta.json` auch wenn `_en_index` in-memory `None` (Disk-Files koennen aus Vor-Restart-Zeit existieren).

Fuer Bug 2:
- `rag_document_delete` iteriert DE+EN-Metadata, markiert Soft-Delete in beiden, persistiert beide Metadata-Files, ruft `_rebuild_index_atomic(verbliebene, settings, language=...)` pro betroffenem Slot. Response erweitert um `removed_de`/`removed_en`/`physically_reindexed_de`/`physically_reindexed_en`.
- `rag_status` und `rag_documents` aggregieren DE+EN — vorher waren EN-Doks unsichtbar.
- `rag_upload` ruft `_detect_lang(chunk)` und propagiert `language=chunk_lang` an `_add_to_index`.

**Tests.** Neuer File `test_p217_index_persistence.py` mit **28 Tests in 8 Klassen** (TestSlotRoutingDualMode 3, TestNoCrossLanguageReset 3, TestEnSlotInitFromAdd 1, TestResolvePathsWriteFlag 2, TestRebuildIndexAtomicLanguage 3, TestResetSyncEnDiskFiles 1, TestSourceWiringP217 12, TestStatusAggregationFields 1). Bestehende Tests in `test_p213_pre_2_dual_storage.py` (2 Source-Audit-Asserts flexibler) und `test_p213_pre_3_reindex_atomic.py::test_empty_chunks_is_noop → test_empty_chunks_purges_slot` (Asserts gespiegelt + WHY-Docstring) angepasst.

**Lessons (3)**:
1. **Recovery-Patches darf man nicht einfrieren** — sie sind Zwischenzustaende. P218-pre's destruktiver Reset wurde von Tests als Spec zementiert. P217 zeigt: wenn die Architektur einen anderen Slot anbieten kann, ist Recovery falsch. Tests fuer Recovery-Patches gehoert ein WHY-Docstring der die Architektur-Schuld benennt.
2. **Soft-Delete-Stores brauchen physische Bereinigung beim DELETE** — sonst driftet der State zwischen Listen-Filter und Total-Counter. Synchroner physischer Reindex pro DELETE ist die saubere Loesung (asynchron via Cron-Job nur bei sehr grossen Stores).
3. **`for_write=True`-Param in Resolver-Helpern** — Read-Operationen koennen Fallback nutzen (Robustheit), Write-Operationen muessen strikt sein (Korrektheit). Die Trennung ist oft der naechste Bugfix.

---

## Parallel-Track: Nala UI-Redesign-Phase (ab 2026-05-07)

Chris hat im Repo-Root einen Patch-Prompt [`NALA_UI_REDESIGN_PROMPT.md`](NALA_UI_REDESIGN_PROMPT.md) plus React-Mockup [`docs/NalaMockup.jsx`](docs/NalaMockup.jsx) plus erweiterte Spec [`docs/DESIGN.md`](docs/DESIGN.md) eingestellt. Aufgabe: komplettes Redesign des Nala-Chat-UI nach Mistral-Le-Chat-Vorbild plus eigene Features (Sentiment-Ambient, Scroll-Nav-Gabelung, 2-Achsen-Skalierung, Reasoning-Block, LLM-Icon, Projektseite raus aus Settings, Sidebar-Content-Shift, Layout-volle-Breite, Collapse-System, Eingabefeld-Expand). 11 Schritte als eigene Patches, dokumentiert als **Phase 5c** in [`ZERBERUS_MARATHON_WORKFLOW.md`](ZERBERUS_MARATHON_WORKFLOW.md). Kann parallel zu den Phase-5a-Folgepatches laufen — berührt nur Frontend (`zerberus/app/routers/nala.py`), nicht Backend. Bei Widerspruch zwischen `docs/DESIGN.md` und Bestand gilt DESIGN.md.

**Zusatz-Bug für die UI-Session vermerkt** (gemeldet 2026-05-07 von Chris an Backend-Coda-Session `dreamy-archimedes-9c858e`, parallel zur P213-pre-4-Arbeit): Hel-Oberfläche wird im Splitscreen-Modus in der Mitte abgeschnitten — unabhängig davon wieviel Fensterplatz Hel zur Verfügung hat. Vermutung: CSS-Layout mit fixer Mittel-Breite oder absolute-Positionierung statt responsive Grid/Flex. Vermutlich in Hel-Templates (`zerberus/templates/hel*.html`) oder Hel-CSS (`zerberus/static/hel*.css`). Vollständige Bug-Beschreibung + Reproduktion in [`UI_BUG_HEL_SPLITSCREEN.md`](UI_BUG_HEL_SPLITSCREEN.md) (eigene Datei, überlebt HANDOVER-Overwrites). **Backend-Coda hat den Bug NICHT angefasst** — UI-Session ist zuständig.

---

## Letzter Polish-Patch

**P-UI-Polish-2** — Visuelles Fine-Tuning Runde 2 nach Chris' Live-Test-Feedback (2026-05-09)

Polish-Patch nach P-UI-Polish (1). Chris hat das neue UI live getestet und auf sechs konkrete Verbesserungen hingewiesen (in Eroeffnungs-Spec dokumentiert): Header zu hoch, Sidebar-Layout vereinfachen, Save-Button aus Header in Sidebar, Scroll-Nav ueberlappt mit Bubble-Content, Sentiment-Chips mehrzeilig, Action-Icons zu weit auseinander, Color-Picker pruefen. P-UI-Polish-2 setzt alle sechs um — ueberwiegend CSS + HTML, plus minimale JS-Erweiterung (Sidebar-Projekt-Liste auf P-UI-5-Cache-Basis).

**Edits.**

- **(1) Header-Hoehe um 1/3 reduziert.** `.header` padding `10px 15px` → `7px 15px` (Landscape `8px 12px` → `5px 12px`). Schriftgroesse 1.05em bleibt unangetastet (Spec: 'Schriftgröße NICHT ändern').
- **(2) Sidebar-Layout neu.** `.sidebar-actions` enthaelt jetzt "➕ Neuer Chat" links + neuer `.sidebar-save-btn` ("💾 Chat speichern") rechts (Save-Button aus Header verschoben — alter `<button class="icon-btn" onclick="openExportMenu()">💾</button>` im Header geloescht). "📁 Projekte" wandert in eigene Zeile `.sidebar-projekte-row`. Neue Klassen `.sidebar-separator` (subtile rgba-0.06-Linie), `.sidebar-projects-list` + `.sidebar-project-item` (.pinned, .active States, border-left-Marker), `.sidebar-chats-heading` (kompakte CHATS-Caption, ersetzt altes `<h3>📋 Letzte Chats</h3>`). Footer geswapt: Optionen ⚙️ LINKS, Ausloggen 🚪 RECHTS (Spec-Vorgabe).
- **(3) `.chat-messages` padding-left auf 32px erhoeht.** padding `20px` → `20px 20px 20px 32px` — pui8-Scroll-Nav (linker Rand, position: fixed bei left: 6px) bekommt PERMANENT Platz, damit Sentiment-Chips und Bubble-Icons nicht ueberlappen. Permanent (nicht erst bei Sicht der Nav), damit der Text nicht springt.
- **(4) `.sentiment-triptych` nowrap + kompakter.** Mit explizitem `flex-direction: row` + `flex-wrap: nowrap`, `.sent-chip` `min-width` 44 → 36 px + `flex-shrink: 0` — drei Chips IMMER in einer Zeile, auch auf 320px-iPhones.
- **(5) `.msg-toolbar` `gap` 6 → 3 px halbiert.** Action-Icons (Timestamp, Mülleimer, Copy, Edit etc.) ruecken naeher zusammen, Zeile wirkt kompakter. Touch-Targets (44x44 px je Button via `.copy-btn` / `.bubble-action-btn` min-width/height) bleiben unangetastet.
- **(6) Color-Picker bereits voll-RGB.** Alle 9 Farb-Eintraege im Settings-Modal (theme: `tc-primary`/`tc-mid`/`tc-gold`/`tc-text`/`tc-accent`; bubble: `bc-user-bg`/`bc-user-text`/`bc-llm-bg`/`bc-llm-text`) sind bereits `<input type="color">` — voller RGB-Bereich nativ (Mobile-System-Picker). Kein zusaetzlicher Code noetig — Test verifiziert nur den Status (`TestColorPickerVollRGB`).

**JS-Erweiterung.** Neue Funktion `renderSidebarProjects()` rendert die kompakte Projekt-Liste in der Sidebar; nutzt denselben Cache (`window.__nalaProjectsCache`), Pin-State (`getPinnedProjectIds`), Active-State (`getActiveProjectId`) und Sort-Mode (`projectsSortMode`) wie `renderProjectsView()` aus P-UI-5 — bleibt automatisch synchron. Versteckt sich komplett wenn keine Projekte vorhanden (kein Ballast). `handleSidebarProjectTap(id)` wechselt das aktive Projekt + schliesst die Sidebar (analog `handleProjectTap` in P-UI-5). Hooks in `setActiveProject` / `clearActiveProject` / `toggleProjectPin` / `setProjectsSortMode` / `loadNalaProjects` / `toggleSidebar` halten die Sidebar-Liste mit der View synchron — alle mit `typeof === 'function'`-Guard, fail-quiet. **Wichtig:** kein neuer Daten-Pfad, kein neuer Cache, kein neues localStorage-Key — nur ein neuer Render-Pfad. Das ist die Eigenschaft, die diesen "Polish"-Status rechtfertigt trotz neuer DOM-Struktur (siehe lessons.md "Polish-Patches duerfen Struktur erweitern").

**DESIGN.md Sektion 12.2** aktualisiert (neuer Sidebar-Aufbau). **Aenderungshistorie** um vollstaendigen P-UI-Polish-2-Eintrag erweitert.

**Tests.** `test_p_ui_4_sidebar_content_shift::test_sidebar_reihenfolge_oben_nach_unten` aktualisiert (neue Reihenfolge mit projekte_row, projects_list, chats_heading; Anti-Assert gegen alten `<h3>📋 Letzte Chats</h3>`, P-debt-10-Pattern). `test_settings_umbau::test_schraubenschluessel_nicht_in_topbar` aktualisiert (Anti-Assert: kein openExportMenu mehr im Header, WHY-Docstring dokumentiert Wanderung) plus neuer `test_export_button_in_sidebar`. Neuer Test-File [`test_p_ui_polish_2_layout.py`](zerberus/tests/test_p_ui_polish_2_layout.py) mit **66 Source-Audit-Tests** in 13 Klassen — `TestHeaderHoeheReduziert` (4 — padding 7px Default + 5px Landscape, Anti-Assert 10px, font-size bleibt), `TestSidebarAktionsZeile` (4 — Neuer Chat + Save-Btn-Klasse + Save-Btn-id + Header-Anti-Assert), `TestSidebarProjekteRow` (4 — eigene Klasse, display block + width 100%, Touch-Target 44px, nicht mehr in actions), `TestSidebarSeparatorenUndCaption` (5 — Klasse + 2 hr-Elemente + Caption uppercase + alter h3 weg), `TestSidebarProjektListe` (4 — Container + CSS + Item-CSS + .pinned/.active States), `TestSidebarFooterSwap` (1 — Settings vor Logout im DOM), `TestChatMessagesScrollNavPadding` (3 — 32px + Anti-Assert + gap 18px bleibt), `TestSentimentTriptychEinzeilig` (4 — flex row + nowrap, min-width 36px + Anti-Assert 44, flex-shrink 0), `TestMsgToolbarGapHalbiert` (3 — gap 3px + Anti-Assert + Touch-Target unveraendert), `TestColorPickerVollRGB` (2 — mind. 9 type=color + alle 9 IDs sind color), `TestSidebarProjektListeJS` (11 — Funktion existiert, Cache, leere Liste, pinned-zuerst, Sort-Mode, handleTap, alle 5 Hooks plus toggleSidebar), `TestPhase5cMechanikIntakt` (16 — alle Vorgaenger-Mechaniken intakt: P-UI-1..11 + P-UI-Polish-1), `TestPUiPolish2InlineMarker` (4 — Marker mehrfach, im CSS, im HTML, im JS).

UI-relevante Test-Suite (19 Files: P-UI-1..11 + p_ui_polish_2 + p_ui_polish + nala_bubble_layout + nala_adapter + p203d3 + settings_umbau + dark_theme_contrast) **750/750 gruen lokal**. JS-Syntax-Integrity (test_p203d3 `TestJsSyntaxIntegrity` per `node --check`) gruen. Volle Unit-Suite-Erwartung: **3381 passed lokal** (P-UI-Polish-1-Baseline 3315 + 66 P-UI-Polish-2-Tests).

**Lessons (2)**: (1) Polish-Patches duerfen STRUKTUR aendern wenn sie Bestands-API wiederverwenden — die Sidebar-Projekt-Liste ist neu (HTML + CSS + JS), aber sie nutzt 100% den P-UI-5-Cache + Sort-Mode + Pin-State, also kein neuer Daten-Pfad — nur ein neuer Render-Pfad. Der Test "Polish vs. Struktur" ist nicht "DOM-Aenderung", sondern "neuer Daten-Pfad". (2) Save-Button-Wanderung Header → Sidebar zeigt warum Anker-Tests Spec-Verschiebungen explizit dokumentieren muessen — alter `assert X in header` wird zu `assert X NICHT in <button> im Header` plus neuer Test `assert X in sidebar` (P-debt-10-Pattern verallgemeinert auf Wanderungen).

---

## Vorletzter Polish-Patch

**P-UI-Polish** — Visuelles Fine-Tuning nach Mockup-Vorbild (2026-05-09)

Polish-Patch nach Phase-5c-Abschluss. Mockup-Vorbild [`docs/NalaMockup.jsx`](docs/NalaMockup.jsx) hat den Default-Look von Phase 5c (z.T. noch Pre-Phase-5c) gegenuebergestellt — Bot-Bubble mit sattem Hintergrund + Box-Shadow + asymmetrischem Tail-Radius wirkte klobig im Vergleich zum cleanen Mockup-Look (ChatGPT/Claude/Mistral-Pattern: freistehender LLM-Text). P-UI-Polish bringt das visuelle Gewicht auf Mockup-Niveau — **ohne strukturelle Aenderung**. Phase 5c (UI-1..UI-11) bleibt mechanisch unangetastet (Collapse, Sidebar-Shift, Scroll-Nav, Reasoning-Block, LLM-Icon, Sentiment-Ambient, 2-Achsen-Skalierung, Projektseite, Reasoning-Mapping).

- **CSS-Aenderungen** in [`zerberus/app/routers/nala.py`](zerberus/app/routers/nala.py) — reine Werte, keine Logik:
  1. `:root --bubble-user-bg` von `rgba(236,64,122,0.88)` auf `0.12` abgesenkt — Mockup zeigt hauchzarten User-Akzent.
  2. `.bot-message`: `background: transparent`, `border-radius: 0`, `box-shadow: none`, `padding: 8px 16px` — KEINE Bubble mehr. Plus `.bot-message::before { display: none; }` (kein Hintergrund = keine Shine) und `.message.bot-message.collapsed::after { display: none; }` (P124-Long-Message-Fade darf nicht mehr auf transparenter Bubble erscheinen).
  3. `.user-message`: `border-radius: 10px` (gleichmaessig, Mockup) statt 18/18/4/18-Tail; neuer `border-left: 3px solid rgba(236, 64, 122, 0.5)` als User-vs-LLM-Marker; `padding: 10px 14px`; `box-shadow: none`.
  4. `.session-item`: `background: transparent` statt `var(--color-primary)`; `padding: 9px 12px` statt `12px`; `margin-bottom: 1px` statt `8px`; `font-size: 13px`; Hover dezent `rgba(255,255,255,0.06)` statt `#0f1e38` — Mistral-Pattern.
  5. `.chat-messages`: `gap: 18px` statt `12px` — Chat atmet.
  6. `#text-input`: dezenter `1px solid rgba(255,255,255,0.08)` Border statt gold-rgba(0.3); `background: rgba(255,255,255,0.03)` statt `var(--color-primary)`; `border-radius: 10px` statt `20px`.
  7. `#text-input:focus`: `border-color: rgba(201, 168, 76, 0.3)` statt voller `--color-gold`-Saettigung.
  8. `.input-area`: `background: rgba(13, 17, 23, 0.95)` statt `var(--color-primary-mid)`; `border-top: 1px solid rgba(255,255,255,0.06)` statt `#2a4068`.
- **DESIGN.md Sektion 1.6** aktualisiert: User-Bubble-Werte, neuer Bot-Bubble-Override-Eintrag (transparent), Border-Akzent-Eintrag, Schatten-Beschreibung. **Neue Anti-Invariante 2:** "`.bot-message` hat KEINEN Bubble-Hintergrund". Pre-P-UI-Polish-Anti-Invariante 1 ("nie schwarz auf schwarz") bleibt aktiv fuer User-Bubble und `--bubble-llm-bg`-Variable im `:root` (typing-indicator + Color-Picker-Fallback). Beide Anti-Invarianten koexistieren durch getrennte Geltungsbereiche.
- **Tests:** `test_p_ui_1_layout::test_bubble_border_radius_tail_bleibt` durch drei neue Tests (`test_bubble_border_radius_user_und_bot`, `test_user_bubble_pink_border_left_marker`, `test_bot_bubble_transparent_background`) ersetzt mit WHY-Docstring (P-debt-10-Pattern: WHY-Docstring + Anti-Asserts mit alten Werten als `not in`). Neuer Test-File `test_p_ui_polish_visual_weight.py` mit **40 Source-Audit-Tests** in 6 Klassen — TestBotBubbleEntfernt (7), TestUserBubbleDezenter (7), TestSessionItemSchlank (7), TestAtmungUndInput (9), TestPhase5cMechanikIntakt (8 — explizite Anti-Regress fuer P-UI-1/6/7/8/10), TestPUiPolishInlineMarker (2). UI-relevante Test-Suite (18 Files: P-UI-1..11 + p_ui_polish + nala_bubble_layout + nala_adapter + p203d3 + settings_umbau + p212 + dark_theme_contrast) **723/723 gruen**. JS-Syntax-Integrity gruen. Volle Unit-Suite-Erwartung: **3315 passed lokal** (P-UI-11-Baseline 3270 + 40 P-UI-Polish-Tests + 5 Netto in test_p_ui_1_layout.py).
- **Lessons (3) in [`lessons.md`](lessons.md):** (1) Visuelles Polish kann reine CSS-Wert-Aenderung sein, wenn die Mechanik durchdacht war — Anti-Regress-Klasse zementiert die Mechanik-Trennung. (2) Anti-Invariante 1 + Anti-Invariante 2 koexistieren wenn ihre Geltungsbereiche getrennt sind (`:root`-Variable vs. `.bot-message`-Override). (3) Bei Test-Anker-Updates wegen Spec-Verschiebung: WHY-Docstring + alte Werte als `not in`-Anti-Asserts halten den Test selbst-dokumentierend.
- **Was P-UI-Polish NICHT macht:** keine strukturellen UI-Aenderungen (Collapse, Sidebar, Scroll-Nav, Reasoning, LLM-Icon, Sentiment, Skalierung, Projekt-Tab — alles unangetastet); kein Backend; kein JS; nichts an P-UI-2-Toggle / P-UI-6-Reasoning-Block / P-UI-7-LLM-Icon / P-UI-10-Sentiment-Ambient angefasst. Sentiment-Ambient profitiert vom freistehenden Text — Schimmer wirkt direkter, weil kein Bubble-Box mehr im Weg.

---

## Letzter Maintenance-Patch

**P-debt-10** — Test-Anker-Fix `test_mein_ton_nicht_mehr_in_sidebar` (2026-05-09)

Mini-Wartungs-Patch nach Phase-5c-Abschluss. Schliesst HANDOVER-Schulden-Liste-Position #10 (Test rot seit P-UI-4 wegen overlay-Backdrop-Removal). Reine Test-Wartung, kein funktionaler Code-Pfad beruehrt — 18 Zeilen in [`zerberus/tests/test_settings_umbau.py`](zerberus/tests/test_settings_umbau.py) `TestMeinTonInSettings::test_mein_ton_nicht_mehr_in_sidebar`.

- **Bug-Ursache:** Anker `</div>\n        <div class="overlay"` ist seit P-UI-4 obsolet (overlay-Backdrop entfernt, vgl. nala.py CSS-Comment Z. ~987). Fallback `find("overlay", sidebar_start)` traf den naechsten lowercase-`overlay`-String — den JS-Comment bei nala.py Z. ~5810 — und schlug die ganze Datei zwischen Sidebar-Anfang (Z. 2858) und JS-Block in den `sidebar_block`. Inklusive Settings-Modal mit `id="my-prompt-area"` (Z. 3110). Test wertete also das Settings-Modal als Teil der Sidebar — assertion fiel.
- **Fix:** Neuer stabiler Nach-Sidebar-Anker `<div id="ee-modal"` (Easter-Egg-Modal Patch 100, Z. 2878 in nala.py). Root-Level-Geschwister direkt nach dem schliessenden Sidebar-Tag, unique in nala.py, ohne Layout-Bedeutung — verschwindet nicht beim naechsten UI-Refactor. Plus harte `assert`-Fehlermeldungen statt stiller Fallbacks. Plus WHY-Docstring der die P-UI-4-Historie dokumentiert.
- **Tests:** `test_settings_umbau.py` 25/25 gruen (vorher 24/25 — Schulden #10). UI-relevante Test-Suite (16 Files) **672/672 gruen** (vorher 671/672). Volle Unit-Suite-Erwartung **3270 passed** unveraendert (kein neuer Test, nur 1 Failure weniger).
- **Lesson** in [`lessons.md`](lessons.md): "Fluechtige HTML-Wrapper als Test-Anker sind brüchig — stabile IDs auf Root-Level-Elementen oder selbst-dokumentierende Comments". Plus generelles "keine `if not found: fallback to broader search`-Patterns — Fallback maskiert den Bruch und verzoegert die Diagnose um Wochen". Backstop: harte `assert anker_gefunden, "WARUM"`.
- **Was P-debt-10 NICHT macht:** keine Refactor von `_find_sidebar_html_block` (bricht aktuell nicht, andere Tests nutzen es per Coincidence — eigener Folge-Patch wenn Drift sichtbar wird), keine UI-Aenderungen, kein Backend.

Phase 5a + Phase 5c bleiben VOLLSTAENDIG ABGESCHLOSSEN. Letzter Code-Patch ist und bleibt P-UI-11.

---

## Aktueller Patch

**P-UI-11** — Phase 5c Schritt 11: Intent-Router → Reasoning-Mapping (Backend) (2026-05-09)

Elfter und FINALER Code-Patch der UI-Redesign-Phase. Phase 5c ist mit P-UI-11 vollstaendig abgeschlossen. Backend-Patch (kein Frontend-Code) — der Intent-Router schaltet bei hohem `effort_score` automatisch auf die Reasoning-Variante des aktuellen Standard-Modells um, ohne UI-Toggle. Spec NALA_UI_REDESIGN_PROMPT.md Sektion 12 + DESIGN.md 14.1 ("Automatisierung über manuelle Kontrolle") + 14.2 (UX-Tabelle).

**Architektur: Mapping-Tabelle + Effort-Heuristik + Pre-Call-Switch + fail-quiet Hook + Source-Audit-Tests.**

- **Neues Modul** [`zerberus/core/reasoning_router.py`](zerberus/core/reasoning_router.py) — separat von `intent_parser.py`, weil das Mapping (Modell → Reasoning-Variante) keine Kopplung zur Header-Parser-Logik hat. Modul-Doc dokumentiert die drei zentralen Architektur-Entscheidungen (Heuristik statt Two-Pass / Range 0..10 statt 1..5 / Whitelist-Stil).
- **`REASONING_MAPPING`** als Modul-Konstante — Spec-Modelle alle drin: `deepseek/v3.2` → `{"model": "deepseek/deepseek-r1", "thinking": False}`, `mistralai/mistral-large` → `{"model": "mistralai/mistral-large", "thinking": True}`, `anthropic/claude-sonnet-4-5` → `{"model": "anthropic/claude-sonnet-4-5", "thinking": True}`, `meta-llama/llama-3.1-70b-instruct` → `None`. Plus Aliase (`deepseek/deepseek-chat`, `mistral/large`, `claude/sonnet`, `llama/3.1-70b`) damit Spec-Notation wie auch volle OpenRouter-IDs greifen. Lowercase-Keys, normalisiert via `_normalize_model_id`.
- **`REASONING_THRESHOLD_DEFAULT = 7`** — Spec "effort_score > 7". ENV `NALA_REASONING_THRESHOLD` (Integer ≥ 0) ueberschreibt.
- **Heuristik `compute_effort_score(message: str) -> int`** im 0..10-Range. Bewusst getrennt vom existing `effort` 1..5 in `intent_parser.ParsedResponse` (das kommt aus dem LLM-Output-Header und ist erst NACH dem ersten Call bekannt). Heuristik-Bestandteile: Wortzahl-Buckets (>=10 +1, >=25 +1, >=60 +1), High-Effort-Tokens (deutsche + englische Reasoning-Trigger wie `warum`/`step by step`/`begruende`/`analysiere`/`beweise`/`why`/`prove`/`derive`, gestaffelt: 1=+2, 2=+3, 3=+5, 4+=+6), Code-Marker (`\`\`\``/`def`/`class`/`{`/`->`/etc., +2 fuer ersten Match), Frage-Wort + `?` (+1), Negativ-Marker (`kurz`/`tldr`/`briefly`/`tl;dr`, -2). Endergebnis auf [0, 10] geclampt.
- **`select_reasoning_variant(base_model, effort, *, threshold=, override_mapping=)`** entscheidet den Switch. Liefert `None` wenn effort ≤ threshold ODER base_model nicht in Mapping ODER Mapping-Eintrag explizit `None`. Sonst ein Dict-Kopie `{"model": "...", "thinking": bool}`. `override_mapping` ermoeglicht Tests + zukuenftigen `config.yaml`-Override unter `modules.reasoning.*` ohne Code-Change.
- **`select_reasoning_for_message(base_model, message, ...)` Convenience-Wrapper** — kombiniert Score-Berechnung + Selection + INFO-Log bei Switch (`[P-UI-11] Reasoning-Switch effort=N base=X -> Y thinking=Z`).
- **Hook in [`zerberus/app/routers/legacy.py`](zerberus/app/routers/legacy.py)** chat_completions vor dem `if _ORCH_PIPELINE_OK:`-Block (vor `messages_for_llm = [m.model_dump() for m in req.messages]`): `_pui11_model_override = None; try: _pui11_base_model = settings.legacy.models.cloud_model if settings.legacy else None; _pui11_score, _pui11_variant = select_reasoning_for_message(_pui11_base_model, last_user_msg); if _pui11_variant: _pui11_model_override = _pui11_variant.get("model"); except Exception as _pui11_err: logger.warning(f"[P-UI-11] Reasoning-Switch fail-quiet: {_pui11_err}")`. Drei bestehende `llm_service.call(messages_for_llm, session_id, ...)`-Aufrufe (Orchestrator-Pfad, Orchestrator-Fallback, no-Orchestrator-Fallback) bekommen `model_override=_pui11_model_override` durchgereicht. Bei `None` → kein Switch (LLMService faellt auf `cloud_model` zurueck via `model_override or settings.legacy.models.cloud_model`).
- **Pre-Call statt Two-Pass.** Two-Pass (erst Standard-Call, dann ggf. Reasoning-Re-Call) verdoppelt Token-Kosten und Latenz. Pre-Call mit Heuristik ist weniger praezise als die LLM-Selbsteinschaetzung (`effort` 1..5 im Header), aber kostenneutral und Spec-konform ("Router switcht API-Call auf Reasoning-Variante" impliziert Pre-Switch). Dokumentiert im Modul-Docstring.
- **UI-Feedback via P-UI-7-LLM-Icon.** Kein neuer Frontend-Code in P-UI-11. Der LLMService liefert in `data.model` automatisch das tatsaechlich verwendete Modell zurueck; P-UI-7 zeichnet dann das Icon entsprechend ("Show, don't tell", DESIGN.md 14.1). Bei Switch sieht der User: gleicher Bubble-Inhalt, anderes Icon — Modellwechsel ist sichtbar, aber nicht stoerend (kein Toast, kein Modal).
- **Source-Audit-Tests** in [`zerberus/tests/test_p_ui_11_reasoning_mapping.py`](zerberus/tests/test_p_ui_11_reasoning_mapping.py): 71 Tests in 8 Klassen. `TestReasoningMapping` (8) prueft die Mapping-Tabelle: Spec-Modelle vorhanden, Lowercase-Keys, Llama explizit `None`, Threshold-Default 7. `TestEffortScoreHeuristic` (13) verifiziert Heuristik-Verhalten: leere/None Inputs → 0, Greeting → ≤2, Smalltalk-Frage → unter Threshold, lange Reasoning-Message → > Threshold, Code-Block-Erkennung, Wortzahl-Eskalation, kurz/tldr-Daempfung (Vergleichs-Test gegen Variante ohne Marker), Range-Clamp [0,10], Englisch (`why` zaehlt). `TestSelectReasoningVariant` (16) deckt alle Switch-Bedingungen ab: unter Threshold, genau am Threshold, ueber Threshold, unbekanntes Modell, explicit-None-Mapping, None/leeres base_model, Case-Normalisierung, Threshold-Override per Argument, ENV-Override, ENV-Invalid-Fallback, override_mapping (Custom + Block-Default), negative effort, Mutation-Schutz (returned dict ist Kopie). `TestSelectReasoningForMessage` (5) testet den Convenience-Wrapper. `TestLegacySourceWiring` (11) prueft den Hook in `legacy.py`: Import-Statement, Marker, Hook-Block-Existenz, `_pui11_model_override`-Variable, `select_reasoning_for_message`-Aufruf, `settings.legacy.models.cloud_model`-Zugriff, try/except, Reihenfolge VOR `messages_for_llm = ...`, drei Aufrufe mit `model_override=_pui11_model_override`, `logger.warning`-Fallback. `TestReasoningRouterSource` (9) prueft Modul-Exports + Marker. `TestKeinRegressZuVorgaengernPatches` (5) verifiziert dass P212-Maskierung weiterhin VOR dem ersten LLM-Call steht, `temperature_override` und `session_id` in allen drei Aufrufen bleiben, P-UI-10 + P-UI-7 unangetastet sind. `TestPUiElevenInlineMarker` (4) verlangt `P-UI-11` mehrfach im Source. 71/71 lokal gruen. 671 UI-relevante Tests gruen (P-UI-1..11 + settings_umbau + nala_bubble_layout + nala_adapter + p203d3 + p212). Test_p203d3 `TestJsSyntaxIntegrity` (node --check) gruen — kein JS beruehrt.

**Was P-UI-11 bewusst NICHT macht:**

- **Two-Pass-Reasoning-Approach.** Erst Standard-Call → bei `effort >= 4` im LLM-Output-Header → Re-Call mit Reasoning-Variante. Verdoppelt Token-Kosten + Latenz. Alternative wenn Heuristik zu unpraezise ist — Folge-Patch.
- **`thinking`-Parameter im LLMService.** Das Mapping enthaelt `thinking: True/False` als Hint — der LLMService nutzt den Hint aktuell NICHT (`call(...)` hat keinen `thinking`-Parameter). Modell-Switch greift trotzdem (deepseek/v3.2 → deepseek/r1 ist ein anderes Modell). Mistral/Claude switchen aktuell auf das gleiche Modell zurueck — der Hint wird im Folge-Patch verwendet wenn der LLMService eine `extra_body={"thinking": {...}}`-Pass-Through-Property bekommt.
- **Per-User-Threshold-Override.** Aktuell global via ENV. Wenn Chris einen Profil-Slider will, ist das ein Folge-Patch in `ProfileConfig`.
- **Konfigurierbares Mapping in config.yaml.** `override_mapping`-Argument ist da, aber kein YAML-Loader. Spec sagt "in Hel konfigurierbar" — falls UI-Editor in Hel kommen soll, ist das ein eigener Patch (Hel-Page-Edit, Endpoint, Validation). Default-Mapping deckt die Spec-Modelle ab.
- **Telemetrie / Cost-Tracking pro Switch.** Der INFO-Log gibt einen Audit-Trail, aber keine Aggregation in DB. Wenn Chris wissen will, wie oft pro Tag geswitcht wird, ist das ein Folge-Patch (Counter in `cost`-Tabelle oder eigene Tabelle).
- **Hel-Splitscreen-Bug** (Schulden #9, eigener Folge-Patch).
- **`test_settings_umbau::test_mein_ton_nicht_mehr_in_sidebar`-Fix** (Schulden #10, eigener Mini-Fix-Patch).

**Drei neue Lessons in [`lessons.md`](lessons.md):**

1. **Pre-Call-Heuristik vs Two-Pass-Reasoning-Switch** — bei Routing-Entscheidungen, die einen LLM-Call beeinflussen, ist eine Pre-Call-Heuristik (basierend auf User-Input-Analyse) der pragmatische Default. Two-Pass (erst Standard-Call, dann konditionaler Re-Call) verdoppelt Token-Kosten und Latenz und ist nur dann sinnvoll, wenn die Pre-Call-Heuristik beweisbar zu unpraezise ist (Messung erforderlich).
2. **0..10-Heuristik-Range entkoppelt vom 1..5-Header-Feld** — wenn ein neuer Score eingefuehrt wird, der semantisch zu einem existing Score "gehoert" aber technisch entkoppelt ist (verschiedene Datenquellen, verschiedene Berechnungs-Zeitpunkte), getrennte Range vermeidet Verwirrung. Spec-Vorgabe `> 7` plus existing Range 1..5 ist NICHT "Spec-Bug" — es sind zwei verschiedene Score-Welten.
3. **Whitelist-Mapping mit explicit-None-Eintraegen** — wenn ein Mapping zwischen zwei Welten Sicherheit verlangt (z. B. "Standard-Modell ↦ Reasoning-Variante darf NUR bei explizit erlaubten Modellen schalten"), drei-Fall-Whitelist statt Default-Allow: (a) Eintrag mit Variante → Switch, (b) explicit-None-Eintrag → kein Switch, kein Fehler, (c) kein Eintrag → kein Switch, kein Fehler. (b) und (c) sehen aus Caller-Sicht identisch aus, aber (b) ist Doku ("dieses Modell habe ich bewusst geprueft, hat keine Reasoning-Variante"), (c) ist Default ("dieses Modell kenne ich nicht, ich switche nicht"). Beides sicher.

---

## Vorletzter Patch

**P-UI-10** — Phase 5c Schritt 10: Sentiment-Ambient-Lighting (2026-05-09)

Zehnter Code-Patch der UI-Redesign-Phase. Sanfter Farbschimmer der gesamten Chat-Oberflaeche basierend auf User-Stimmung — vier Sentiment-Kategorien (Wut → Rot, Freude → Gold, Nachdenklich → Violett, Sachlich → Blau), Stufen 0–5 mit Opacity-Cap 15 % (DESIGN.md 10.3), Fade ueber 12 s (`--transition-ambient: 12s ease-in-out`), abschaltbar in Settings (Default ON). Datenquelle: `data.sentiment.user.{bert, prosody, consensus}` aus dem Triptychon-System (Patch 192/193); Frontend klassifiziert nur den User-Block in eine von vier Kategorien (oder neutral), schiebt die Klassifikation in eine 3-er History-State-Machine, prueft Konsens (Mehrheit ≥ 2), erhoeht die Stufe bei konsistenter Stimmung oder kippt auf Stufe 1 bei Wechsel. Frontend-only — kein Backend-Schema-Change. Disjunkter CSS-Namespace `.pui10-...` — keine Kollision mit P-UI-1..9 oder Bestands-Klassen.

**Architektur: CSS-Variablen + Ambient-Layer als Container-Child + isolation-Stacking-Context-Trick + Klassifikator + State-Machine + Settings-Toggle + Source-Audit-Tests.**

- **CSS-Variablen** im `:root`: vier Farben `--sentiment-anger: rgba(220,50,50,1)` (Wut Rot, gesaettigt), `--sentiment-joy: rgba(255,200,60,1)` (Freude warmes Gold), `--sentiment-contemplative: rgba(120,80,200,1)` (Nachdenklich Violett), `--sentiment-technical: rgba(60,130,220,1)` (Sachlich kuehles Blau). Plus Steuer-Variablen `--sentiment-current` (Default `transparent`), `--sentiment-opacity` (Default 0), `--transition-ambient: 12s ease-in-out` (Mitte des 10–15 s-Fensters laut Spec).
- **`.app-container`-Edit**: nur eine neue Property `isolation: isolate` — erzwingt einen eigenen stacking context, damit der negative z-index des Ambient-Layer-Childs gefangen bleibt (nicht body-Hintergrund freigibt). Bestehende Properties (max-width 500px, position relative, transform-Transition) bleiben unangetastet. Erklaerender Kommentar VOR dem Block (nicht innerhalb), damit Test-Slices `_app_container_block` mit `after[:600]` weiter alle Properties einschliessen.
- **CSS-Layer `.pui10-ambient-layer`** in [`zerberus/app/routers/nala.py`](zerberus/app/routers/nala.py): `position: absolute; inset: 0; pointer-events: none; z-index: -1; background-color: var(--sentiment-current); opacity: var(--sentiment-opacity); transition: opacity var(--transition-ambient), background-color var(--transition-ambient)`. Sitzt zwischen Container-Background (Stack-Level 1) und allen statischen Children wie `#chat-screen` (Stack-Level 3+). Body-Klasse `pui10-ambient-disabled` setzt `display: none` auf den Layer (Toggle-Off).
- **Stage-Bubble-Shadow ab Stufe 2** (DESIGN.md 10.3): Selektor `body.pui10-ambient-stage-N .message[class]` (mit `[class]`-Attribut-Match statt blossem `.message`, damit Source-Audit-Tests die `nala_src.split(".message {")`-Slices nicht versehentlich auf den P-UI-10-Block treffen) stapelt einen Sentiment-Glow zum Tiefen-rgba(0,0,0,0.3)-Shadow: `box-shadow: 0 4px 14px var(--sentiment-current), 0 4px 8px rgba(0,0,0,0.3)`. Bei `pui10-ambient-disabled` plus Stage zusaetzlich: nur das Tiefen-Shadow, kein Glow. `@media (prefers-reduced-motion: reduce)`: alle Transitions auf 0 s (Accessibility, DESIGN.md 16).
- **HTML**: `<div class="pui10-ambient-layer" id="pui10-ambient" data-pui10-ambient="1" aria-hidden="true"></div>` als ERSTES Child der `.app-container`, vor `#login-screen`. Plus `<div class="settings-section" data-pui10-ambient-section="1">` im Look-Tab nach der pui9-Skalierungs-Sektion mit `<input type="checkbox" id="pui10AmbientToggle" onchange="pui10_setAmbientEnabled(this.checked)">`-Toggle (44-px-Touch-Target, Beschreibung mit allen vier Farb-Mappings).
- **JS-Konstanten**: `PUI10_HISTORY_MAX = 3` (Spec "gemittelt ueber die letzten 2-3"), `PUI10_STAGE_MAX = 5`, `PUI10_STAGE_OPACITY = [0, 0.03, 0.06, 0.09, 0.12, 0.15]` (Cap 15 %), `PUI10_LS_KEY = 'nala_sentiment_ambient'`, `PUI10_CATEGORIES = ['neutral', 'anger', 'joy', 'contemplative', 'technical']`, `PUI10_STAGE_CLASSES`/`PUI10_CATEGORY_CLASSES` als praeparierte Klassen-Listen.
- **JS-Helper `pui10_isAmbientEnabled()`**: liest `localStorage[nala_sentiment_ambient]`, Default ON via `v !== 'false'`-Check (Spec: "Default ON, mit Opt-Out fuer Privacy-bewusste User").
- **JS-Helper `pui10_classifySentiment(userBlock)`**: Strenge Bedingungen: `consensus.source === 'bert+prosody'` (nicht nur BERT — Spec "BERT allein → KEIN Effekt, zu unzuverlaessig"), `!consensus.incongruent` (Inkongruenz blockt komplett), beide `bert` und `prosody` vorhanden. Mapping aus `prosody.mood` + `bert.label`: `angry`/`stressed`/`anxious` + label != positive → anger; `happy`/`excited` + label != negative → joy; `calm`/`sad`/`tired` + neutral label → contemplative; beides neutral → technical; sonst neutral. Defensive non-string-Checks via `String(...).toLowerCase()`.
- **JS-Helper `pui10_majority(history)`**: Mehrheits-Voter ueber die letzten N Kategorien. Iteriert RUECKWAERTS, sodass bei Gleichstand (z.B. zwei mal joy + zwei mal anger) die JUENGSTE Klassifikation gewinnt — vermeidet Fluttern zwischen Stimmungen. Liefert `{category, count}`.
- **JS-Helper `pui10_pushSentiment(category)`**: shiftet History (max 3), prueft Mehrheit (≥ 2 Kategorien gleich), drei Branches: kein Konsens → Stufe 0 + neutral; gleiche Kategorie wie current → +1 Stufe (capped via `Math.min(stage+1, 5)`); andere Kategorie → Stufe 1 mit neuer Kategorie. Wenn Toggle OFF: ruft sofort `pui10_resetAmbient()` und returnt — kein State-Update.
- **JS-Helper `pui10_applyAmbient()`**: setzt `--sentiment-current` (via `'var(--sentiment-' + category + ')'`-Indirektion am `:root`, theme-anpassbar) und `--sentiment-opacity` als String. Bei Stufe 0 oder neutral: `transparent` + `'0'`. Plus genau eine Body-Stage-Klasse (`pui10-ambient-stage-0`..`5`) und genau eine Body-Category-Klasse (`pui10-sentiment-anger`..`technical`/`neutral`) — alle anderen werden vorher entfernt.
- **JS-Helper `pui10_resetAmbient()`**: leert History, setzt currentCategory auf neutral, currentStage auf 0, ruft `pui10_applyAmbient()`. Wird beim Toggle-Off und bei `resetTheme()` aufgerufen.
- **JS-Helper `pui10_setAmbientEnabled(on)`**: persistiert in localStorage (`'true'`/`'false'`-String), kippt die Body-Klasse `pui10-ambient-disabled`, ruft bei Off `pui10_resetAmbient()` (sofortiger Cleanup), bei On `pui10_applyAmbient()` (CSS-Vars sicher setzen).
- **JS-Helper `pui10_initAmbient()`**: Bootstrap. Liest Toggle-State aus localStorage, synct die Checkbox-UI, kippt die Body-Klasse, ruft `pui10_applyAmbient()` fuer initialen CSS-State. Wird per DOMContentLoaded oder direkt aufgerufen (analog P-UI-9-Boot-Pattern).
- **`bootPui10Ambient`-IIFE**: registriert `pui10_initAmbient` als DOMContentLoaded-Listener (oder direkt wenn DOM bereits ready).
- **Caller-Hook** im `if (data.sentiment) {...}`-Block direkt nach `applySentimentToLastBubbles(data.sentiment)`: `try { if (typeof pui10_pushSentiment === 'function') pui10_pushSentiment(pui10_classifySentiment(data.sentiment.user)); } catch (_e) {}` — fail-quiet, NUR User-Block (Spec: "NUR User-Stimmung, nicht LLM-Output"). Beide Aufrufe (applySentimentToLastBubbles fuer Triptychon-Emojis und pui10_pushSentiment fuer Ambient) koexistieren.
- **`resetTheme()`-Erweiterung**: ruft zusaetzlich `pui10_resetAmbient()` (Stimmungs-State weg). Toggle-Praeferenz `nala_sentiment_ambient` bleibt unangetastet — sie ist eine User-Praeferenz, kein Theme-Wert.
- **addMessage-Signatur unveraendert** (`function addMessage(text, sender, tsOverride)`) — bewusste Architektur damit P-UI-2/6/7/8/9-Source-Audit-Tests, die exakt auf diesen Header splitten, weiter gruen bleiben.
- **Source-Audit-Tests** in [`zerberus/tests/test_p_ui_10_sentiment_ambient.py`](zerberus/tests/test_p_ui_10_sentiment_ambient.py): fuenf Klassen, 89 Tests. `TestSourceWiringCSS` (24) prueft `:root`-Variablen, Layer-Properties, `isolation: isolate` auf `.app-container`, Stage-Shadow-Selektoren, Reduced-Motion-Block. `TestSourceWiringJSHelpers` (35) prueft alle Konstanten + Klassifikator-Branches + Mehrheits-Voter (Rueckwaerts-Iteration!) + State-Machine-Transitions + Apply-Properties + Reset + Set-Enabled + Init + Boot-IIFE. `TestSettingsUiWiring` (12) prueft die Toggle-HTML-Struktur (Checkbox-Type, ID, onchange-Handler, 44-px-Touch-Target, Vier-Farb-Mappings im Beschreibungstext) + Caller-Hook-Position + try/catch + typeof-Guard + resetTheme-Migration + Layer-HTML-Audit. `TestKollisionMitVorgaengernPatches` (12) prueft disjunkten Namespace gegenueber P-UI-1..9 + Co-Existenz mit `applySentimentToLastBubbles` + addMessage-Signatur invariant. `TestPUiTenInlineMarker` (8) verlangt `P-UI-10` mehrfach im Source. Slice-basiert, kein Browser/Playwright.

**Was P-UI-10 bewusst NICHT macht:**

- **Sentiment-State persistieren ueber Page-Reload.** History + currentStage + currentCategory bleiben nur im JS-Memory — beim Reload wird auf neutral/0 zurueckgesetzt. Das ist absichtlich: Stimmung ist ein Live-Phaenomen, nicht ein Persistenz-Wert. Nur der User-Toggle (Default ON) wird in localStorage gehalten.
- **Sentiment-Ambient bei Reload-Bubbles aus loadSession.** `loadSession` rendert alte Messages ohne Sentiment-Data (Backend speichert das pro Message nicht). Beim Session-Switch wird der Ambient nicht "ge-replayed" — die naechste neue User-Message setzt den State neu. Folge-Patch: Backend koennte Sentiment-Konsens persistieren, Frontend koennte beim loadSession ueber die letzten N Messages iterieren.
- **Beim Session-Wechsel den Ambient-State auf 0 zuruecksetzen.** Aktuell traegt der State zwischen Sessions weiter — das ist eher ein Feature ("der User ist gerade froehlich, egal ob er einen anderen Chat oeffnet") als ein Bug. Falls Chris/Jojo einen Session-spezifischen Ambient wollen, ist das ein 5-Zeilen-Folge-Patch (`pui10_resetAmbient()` in `loadSession`/`startNewChat`).
- **5-er Stufen-System statt 4-er.** Spec sagt "Stufen 0-4", aber das praktische Mapping mit Opacity 0/3/6/9/12/15 ergibt natuerlich 6 Werte (inkl. Cap). Implementation behaelt 6 Stufen (0..5), die Spec-Bezeichnungen "Neutral / Zart / Erkennbar / Deutlich / Spuerbar / Max" mappen 1:1 auf die Werte. Kein Daten-Konflikt.
- **Live-Cap-Detection / Auto-Adapt-Speed bei `prefers-reduced-motion`-Kippung.** Die `@media (prefers-reduced-motion: reduce)`-Regel setzt Transitions auf 0 s, aber wechselt der User die OS-Praeferenz LIVE waehrend Nala laeuft, gibt es keinen JS-Listener auf `matchMedia('(prefers-reduced-motion: reduce)')` — der Wechsel wird erst beim naechsten Render gegriffen. Spec hat keine Aussage dazu, akzeptable Edge-Case.
- **Hel-Splitscreen-Bug** (Schulden #9, eigener Folge-Patch).
- **`test_settings_umbau::test_mein_ton_nicht_mehr_in_sidebar`-Fix** (Schulden #10, eigener Mini-Fix-Patch).
- **README.md / huginn_kennt_zerberus.md / PROJEKTDOKUMENTATION.md / DESIGN.md** — alle vier upgedatet (Doku-Pflicht laut WORKFLOW.md). Spiegel-Kopie `docs/RAG Testdokumente/huginn_kennt_zerberus.md` ebenfalls upgedatet.

**Lessons (3):**

1. **`isolation: isolate` ist die minimale Aenderung, um einen Container zum eigenen stacking context zu machen.** P-UI-10 braucht den Effekt, dass `z-index: -1` auf einem Child von `.app-container` *innerhalb* des Containers gefangen bleibt — sonst schlaegt der negative z-index auf body-Ebene durch und der Layer rutscht hinter den Container-Background. Drei Optionen waeren moeglich: `position: relative` mit `z-index: 0` (aendert das Layout potentiell), `transform: translateZ(0)` (kann aber andere Layout-Probleme triggern, Sub-Pixel-Rendering), oder `isolation: isolate` (CSS-Property speziell dafuer designed, keine Layout-Side-Effects). **Faustregel:** wenn ein Element zum Container fuer einen `z-index: -1`-Child werden soll und keine bestehende Stacking-Context-Trigger-Property gesetzt ist, `isolation: isolate` ist der saubere Pfad — kein Layout-Side-Effect, klare Intention im Code-Review.

2. **Substring-Splits in Source-Audit-Tests sind robust nur, wenn die Substring-Form unique im Source ist.** Erste P-UI-10-Iteration nutzte `body.pui10-ambient-stage-N .message,` als Selektor-Liste — die Test-Slices in `test_p_ui_1_layout.py` (`nala_src.split(".message {")[1]`) traten auf den P-UI-10-Block statt auf den realen `.message`-Definition-Block, weil `.message {` schon im pui10-CSS als Substring der Selektor-Liste-End-Form vorkam. Fix: `.message[class]` als Attribut-Match — semantisch identisch (jedes `.message`-Element hat ein `class`-Attribut, immer wahr), aber der literale Substring `.message {` taucht im pui10-Block nicht mehr auf. **Faustregel:** wenn Source-Audit-Tests Substring-Splits nutzen, jeder neue Patch muss seine Selektoren so formulieren, dass die "kanonische Form" des Bestand-Selektors NICHT als Substring im neuen Block auftaucht. Backstop: einmal `grep ".message {" nala.py` nach jedem Patch ausfuehren, um zu verifizieren dass es nur an EINER Stelle vorkommt.

3. **State-Machines mit Last-Wins-Tie-Break sind weniger flackerig als First-Wins.** Der `pui10_majority`-Voter iteriert RUECKWAERTS durch die History, sodass bei Gleichstand (z.B. 2× joy + 2× anger nach 4 Klassifikationen mit History-max 3) die juengste Klassifikation gewinnt. Alternative First-Wins (vorwaerts-Iteration mit `>`) wuerde bei Stimmungswechsel manchmal noch die alte Stimmung halten — Flacker-Effekt. P-UI-10-Faustregel: **bei einer State-Machine, die ein Live-Signal interpretiert (Stimmung, Aktivitaet, etc.), tie-break auf das juengste Sample.** Im Conflict-Fall folgt der State der aktuellen Realitaet, nicht der historischen Mehrheit. Backstop: `test_majority_voter_iteriert_rueckwaerts` zementiert die Iteration.

**Tests:** 89 neue in [`zerberus/tests/test_p_ui_10_sentiment_ambient.py`](zerberus/tests/test_p_ui_10_sentiment_ambient.py) — fuenf Klassen: `TestSourceWiringCSS` (24), `TestSourceWiringJSHelpers` (35), `TestSettingsUiWiring` (12), `TestKollisionMitVorgaengernPatches` (12), `TestPUiTenInlineMarker` (8). Alle 560 UI-relevanten Tests gruen lokal (test_p_ui_10 89 + test_p_ui_9 59 + test_p_ui_8 83 + test_p_ui_7 54 + test_p_ui_6 48 + test_p_ui_5 53 + test_p_ui_4 29 + test_p_ui_3 26 + test_p_ui_2 20 + test_p_ui_1 16 + test_settings_umbau 24 + test_nala_bubble_layout 15 + test_nala_adapter 14 + test_p203d3_nala_code_render 30). test_p203d3 `TestJsSyntaxIntegrity` (node --check) gruen — kein JS-Syntax-Fehler nach Edit. Pre-existing-Failures unveraendert: 1 `test_settings_umbau::test_mein_ton_nicht_mehr_in_sidebar` (Schulden #10 seit P-UI-4). Nicht durch P-UI-10 verursacht.

**Logging-Tag:** keiner — P-UI-10 ist reines Frontend-CSS+JS-Patch, kein neuer Server-Code-Pfad.

---

## Vorheriger Patch (Referenz)

**P-UI-9** — Phase 5c Schritt 9: 2-Achsen-Skalierung (UI-Scale + Schriftgröße als Stepper) (2026-05-08)

Neunter Code-Patch der UI-Redesign-Phase. Ersetzt den 1-Achsen-UI-Slider (Patch 142 / B-016) und die festen 4-Font-Presets (Patch 86) durch zwei UNABHAENGIGE Stepper-Achsen — `--ui-scale` (0.8..1.5, Schritt 0.05) für UI-Elemente (Buttons, Sidebar, Settings, Login, Labels) und `--font-size-base` (11..24px, Schritt 1px) für Bubble-Text und Eingabefeld. Der entscheidende Architektur-Move: ein Body-Anker `body { font-size: calc(15px * var(--ui-scale)); }` macht --ui-scale ueber em-Kette wirksam, waehrend Bubbles und `#text-input` explizit `font-size: var(--font-size-base)` setzen und damit unabhaengig bleiben — Spec DESIGN.md Sektion 2.5 "UI-Skalierung beeinflusst NICHT den Chat-Content" exakt erfuellt. Die alte Patch-142-Implementation hat beide Variablen gekoppelt (`--font-size-base = 16 * f + 'px'`) — das war ein latentes ANTI-Pattern, das der User nicht spueren konnte, weil die feste Slider-Range (1.4 statt 1.5) und das fehlende calc() im CSS die Wirkung gedaempft haben. P-UI-9 entkoppelt die Achsen sauber.

**Architektur: zwei pui9-Stepper-Bloecke + Body-Anker + entkoppelte Setter + Backwards-Compat-Shims + Source-Audit-Tests.**

- **CSS-Variablen** im `:root`: `--font-size-base: 15px` (Patch 86) und `--ui-scale: 1` (Patch 142) bleiben bestehen — werden nur neu verkabelt. Body-Edit: `body { font-size: calc(15px * var(--ui-scale)); }` ist der UI-Scale-Anker. Bubbles (`.message`) und Eingabefeld (`#text-input`) setzen weiterhin `font-size: var(--font-size-base)` — UNABHAENGIG vom body-font-size.
- **CSS-Block** in [`zerberus/app/routers/nala.py`](zerberus/app/routers/nala.py): `.pui9-stepper-row` (`display: flex; align-items: center; justify-content: space-between; gap: 12px; margin: 10px 0`), `.pui9-stepper-label` (`flex: 1 1 auto; font-size: 0.88em; color: var(--color-text-light)`), `.pui9-stepper-control` (`display: inline-flex; gap: 8px`), `.pui9-stepper-btn` (`min-width: 44px; min-height: 44px; padding: 0; border: 1px solid #2a4068; border-radius: 8px; font-size: 1.1em; font-weight: 700`), Hover/Active gold-Highlight, Disabled-Modifier `.pui9-disabled` (Opacity 0.4) fuer Cap-Visualisierung an Achsenenden. `.pui9-stepper-value` (`min-width: 64px; text-align: center; font-family: monospace`). Touch-Target durchgaengig 44 px. Toter `.font-preset-row`/`.font-preset-btn`-Block ENTFERNT.
- **HTML in Settings-Tab "Look"** (`<div class="settings-section" data-pui9-scaling="1">`): zwei `pui9-stepper-row` mit `data-pui9-axis="ui-scale"` und `data-pui9-axis="font-size-base"` als Audit-Hooks. Pro Row: `pui9-stepper-label`-Text, `pui9-stepper-control` mit `<button>−`, `<span>` (Wert-Anzeige), `<button>+`. Buttons haben `aria-label`-Attribute (Accessibility), `id="pui9-{ui-scale,font-size}-{minus,plus,value}"` als JS-Hooks, und `onclick="pui9_step{UiScale,FontSize}(±1)"`.
- **JS-Konstanten** in [`zerberus/app/routers/nala.py`](zerberus/app/routers/nala.py): `PUI9_UI_SCALE_MIN = 0.8`, `PUI9_UI_SCALE_MAX = 1.5`, `PUI9_UI_SCALE_STEP = 0.05`, `PUI9_UI_SCALE_DEFAULT = 1.0`, `PUI9_FONT_SIZE_MIN = 11`, `PUI9_FONT_SIZE_MAX = 24`, `PUI9_FONT_SIZE_STEP = 1`, `PUI9_FONT_SIZE_DEFAULT = 15` als single source of truth fuer beide Achsen.
- **JS-Helper `pui9_clamp(val, min, max)`**: Standard-Clamp ohne Math.min/max-Tricks (defensiv lesbar).
- **JS-Helper `pui9_roundUiScale(f)`**: Float-Snap auf Vielfache der Schrittweite. Formel: `Math.round((f - MIN) / STEP) * STEP + MIN`, dann auf 2 Stellen gerundet. Vermeidet Float-Drift wie 1.05000000000001 nach mehrfachem Stepping.
- **JS-Helper `pui9_setUiScale(val)`**: parsed val, snapt + clampt, setzt NUR `--ui-scale`, persistiert `nala_ui_scale`, ruft `pui9_renderUiScaleStepper(f)` fuer UI-Sync. **Beruehrt --font-size-base NICHT** — entkoppelt die Achse.
- **JS-Helper `pui9_setFontSize(val)`**: akzeptiert Zahl (`15`) oder String (`"15px"`), parsed beide, clampt, setzt NUR `--font-size-base`, persistiert `nala_font_size`. Beruehrt `--ui-scale` NICHT.
- **JS-Helper `pui9_stepUiScale(direction)` / `pui9_stepFontSize(direction)`**: greift den aktuellen computed value via `getComputedStyle`, addiert `direction * STEP`, delegiert auf den jeweiligen Setter. Wenn die Cap-Grenze erreicht ist, kein No-Op-Setter-Aufruf — nur Stepper-Render fuer Disabled-State.
- **JS-Helper `pui9_resetUiScale()` / `pui9_resetFontSize()`**: entfernt CSS-Property + localStorage-Key + setzt Default-Anzeige im Stepper. Spec-Erfuellung: `resetTheme()` ruft beide.
- **JS-Helper `pui9_renderUiScaleStepper(f)` / `pui9_renderFontSizeStepper(px)`**: setzt die Wert-Anzeige (z.B. "1.05×" / "16px") und togglet die `.pui9-disabled`-Klasse an den Buttons je nach Cap-Position.
- **JS-Helper `pui9_renderSteppers()`**: Bootstrap-Sync — liest die computed values und ruft beide Render-Helper. Wird beim Modul-Load (DOMContentLoaded oder direkt) gerufen.
- **Backwards-Compat-Shims** `applyUiScale(val)` und `resetUiScale()`: bleiben als duenne One-Liner, die auf `pui9_setUiScale`/`pui9_resetUiScale` delegieren. Bestands-Tests in `test_settings_umbau::TestUiSkalierung` und `test_loki_mega_patch` referenzieren diese Namen weiter — die delegieren jetzt sauber. Die alte Achsen-Kopplung ist KEIN Bestandteil mehr.
- **Komplett ENTFERNT**: `setFontSize`/`resetFontSize`/`markActiveFontPreset` (Patch 86) — alle Aufrufer (`resetTheme`, `loadFav`) sind auf `pui9_setFontSize`/`pui9_resetFontSize` migriert.
- **resetTheme()-Update**: ruft jetzt `pui9_resetFontSize()` UND `pui9_resetUiScale()` (Spec: "resetTheme muss auch resetUiScale aufrufen"). Vorher wurde nur die Schriftgroesse zurueckgesetzt.
- **Favoriten v2-Schema erweitert**: `saveFav` schreibt zusaetzlich `uiScale: localStorage.getItem('nala_ui_scale')`. `loadFav` liest `raw.uiScale` und ruft `pui9_setUiScale(raw.uiScale)` — tolerant: ohne Slot bleibt die Achse unangetastet (kein implizites Reset, damit pre-P-UI-9-Favoriten nicht den User-Setting-Drift verursachen).
- **Pre-Render-IIFE im `<head>`** erweitert: liest sowohl `fav.uiScale` (im aktiven-Favoriten-Pfad) als auch `nala_ui_scale` (im Plain-Pfad) und setzt `--ui-scale` schon vor dem ersten Paint. Vermeidet sichtbares Stepper-Snapping nach DOMContentLoaded, falls ein User mit skalierter UI ohne aktiven Favoriten reloadet.
- **Source-Audit-Tests** in [`zerberus/tests/test_p_ui_9_two_axis_scaling.py`](zerberus/tests/test_p_ui_9_two_axis_scaling.py): fuenf Klassen, 59 Tests. `TestSourceWiringCSS` (13) prueft Body-Anker, alle pui9-Klassen, 44-px-Touch-Target, Disabled-State, ENTFERNTE font-preset-Klassen. `TestSourceWiringJSHelpers` (18) prueft alle Konstanten + Helper + Backwards-Compat-Shims + Anti-Pattern-Sicherung (`pui9_setUiScale` beruehrt --font-size-base NICHT, `pui9_setFontSize` beruehrt --ui-scale NICHT). `TestSettingsUiWiring` (12) prueft die HTML-Stepper-Struktur in der Settings-Section + aria-Labels + Achsen-Datenattribute. `TestKollisionMitVorgaengernPatches` (12) prueft resetTheme-Migration, saveFav/loadFav-uiScale-Slot, Pre-Render-IIFE-uiScale-Read, ENTFERNTE alte Helper, addMessage-Signatur invariant. `TestPUiNineInlineMarker` (4) verlangt `P-UI-9 (Phase 5c, 2026-05-08)` mehrfach im Source. Slice-basiert, kein Browser/Playwright. Bestehende `TestUiSkalierung` in [`test_settings_umbau.py`](zerberus/tests/test_settings_umbau.py) komplett umgeschrieben (Slider→Stepper, von 5 Tests auf 11). Loki-Selektor in [`test_loki_mega_patch.py`](zerberus/tests/test_loki_mega_patch.py) Z. 681 um pui9-Stepper-IDs erweitert (alte slider-IDs als Fallback fuer pre-P-UI-9-Builds).

**Was P-UI-9 bewusst NICHT macht:**

- **Live-Cap-Detection durch Layout-Probe.** DESIGN.md 2.5 sagt: "Bei jedem Skalierungsschritt prüfen: überlappen UI-Elemente? Falls ja → Schritt wird nicht ausgeführt, visuelles Feedback." Aktuelle Implementation ist eine reine Wert-Cap (0.8/1.5 bzw. 11/24px) — sie probt nicht, ob die UI bei einem Schritt tatsaechlich noch lesbar/bedienbar ist. Folge-Patch (UI-9-pre-2) koennte einen `getBoundingClientRect`-Check auf kritische Elemente (Sidebar-Footer-Buttons, Settings-Tab-Header) einfuegen und den Schritt zurueckrollen, wenn Overflow entstanden ist. Aktueller Wert-Cap deckt 95 % der Faelle ab.
- **Slider-Toggle als alternative Eingabe.** Spec sagt "Stepper, KEIN Slider" — Implementation hat den Slider komplett rausgeworfen. Falls Chris/Jojo später den Wunsch haben "ich hab grobmotorisch und brauche groessere Schritte", waere ein Long-Press-Auto-Repeat auf den Stepper-Buttons der saubere Pfad — kein Slider. Nicht implementiert.
- **Reset-Button pro Achse.** Aktuelle Implementation hat KEINEN expliziten "↺ Zuruecksetzen"-Button pro Stepper — der `Theme-Reset`-Button am unteren Rand des Settings-Modals ruft `resetTheme()` auf, das jetzt beide Achsen mitresetet. Wenn Chris einen feineren Reset will, ist das ein 5-Zeilen-Folge-Patch.
- **Skalierungs-Telemetrie.** Welcher User welche Werte gewaehlt hat, wird nicht logged — bleibt rein client-side in localStorage. Bewusste Privacy-Entscheidung, kein TODO.
- **Migration alter `nala_font_size`-Werte.** Pre-P-UI-9 schrieb der alte Patch-86-Helper Werte wie `'13px'`/`'15px'`/`'17px'`/`'19px'` (4-Preset-Werte) bzw. der alte Patch-142-Helper schrieb `(16 * f) + 'px'`-Werte (z.B. `'18.4px'` fuer ui-scale=1.15). Beide Wertformate sind im neuen `pui9_setFontSize` parsbar (Regex `^(\d+(?:\.\d+)?)`) — die werden beim ersten Stepper-Klick auf den naechsten Schritt-Wert (Integer) gesnappt. Kein expliziter Migrations-Pass noetig. Pre-P-UI-9-Favoriten ohne `uiScale`-Slot lassen die Achse unangetastet — der User behaelt seinen aktuellen UI-Scale-Wert.
- **Hel-Splitscreen-Bug** (Schulden #9, eigener Folge-Patch).
- **`test_settings_umbau::test_mein_ton_nicht_mehr_in_sidebar`-Fix** (Schulden #10, eigener Mini-Fix-Patch).
- **README.md / huginn_kennt_zerberus.md / PROJEKTDOKUMENTATION.md / DESIGN.md** — alle vier upgedatet (Doku-Pflicht laut WORKFLOW.md). Spiegel-Kopie `docs/RAG Testdokumente/huginn_kennt_zerberus.md` ebenfalls upgedatet.

**Lessons (3):**

1. **Body-Anker ueber `calc(<base> * var(--scale))` ist der saubere Pfad fuer "skaliere die UI, nicht den Content".** Der Patch-142-Versuch — beide Variablen gleichzeitig zu setzen (`--ui-scale` + `--font-size-base` an dieselbe Zahl koppeln) — ist die Anti-Implementation: der User kann die Bubbles nicht mehr unabhaengig skalieren, und das `calc()`-Versprechen aus dem Patch-142-Kommentar ("Dank calc() in der CSS skalieren alle Texte/Buttons/Paddings anteilig") wird im Code nirgends eingeloest. P-UI-9-Faustregel: wenn ein Skalierungssystem zwei Klassen von Elementen unterscheiden soll (UI vs. Content), gehoeren BEIDE Klassen an explizite, getrennte CSS-Variablen, und der Body bekommt einen `calc()`-Anker auf eine davon (die "Default-Klasse" — hier UI). Der Content nutzt seine eigene Variable als `font-size: var(--font-size-base)` und ist damit unabhaengig vom body. **Backstop:** `test_zwei_achsen_unabhaengig` prueft den Anti-Pattern-Fall (`pui9_setUiScale` darf `--font-size-base` NICHT mehr setzen).

2. **Backwards-Compat-Shims sind billig genug, dass sie sich gegen "Bestands-Tests anpassen"-Risiko lohnen.** P-UI-9 koennte `applyUiScale`/`resetUiScale` komplett entfernen und alle bestehenden Tests aktualisieren. Aber: zwei drei-Zeilen-Shims kosten nichts, halten das Bestands-Test-Set (`test_settings_umbau::TestUiSkalierung`) lauffaehig, und reduzieren die Diff-Groesse fuer Reviewer. Setzt die Anti-Pattern-Kopplung (`16 * f`) im Process auf Null. **Faustregel:** wenn ein Patch ein Bestands-API umbaut (gleicher Name, anderes Verhalten), behalte den Namen als Shim — nur wenn das Shim selbst irrefuehrend wuerde (z.B. Side-Effects, die der Caller noch erwartet), entferne ihn ganz. Vorgaenger-Pattern: P-UI-3 hat `Math.min(scrollHeight, 140)` und alle Inline-`style.height`-Setter ohne Shim entfernt — weil dort der Sinn umgekehrt war (CSS-only statt JS-driven), nicht ein neues Verhalten unter altem Namen.

3. **Pre-Render-IIFE-Doppelschreibung beim Skalierungs-Boot vermeidet sichtbares Snapping.** P-UI-9 hat zwei IIFE-Bloecke fuer Boot-Restore: die Pre-Render-IIFE im `<head>` (laeuft VOR dem ersten Paint) und die `restoreUiScale`-IIFE am JS-Block-Ende (laeuft nach dem ersten Paint, sobald das DOM verfuegbar ist). Erstere setzt `--ui-scale` direkt aus localStorage, sodass der erste Paint schon korrekt skaliert ist. Letztere ruft `pui9_renderSteppers()` fuer die Stepper-UI-Sync. Wenn ich nur die spaete IIFE haette: der Body wuerde mit `var(--ui-scale)`=1 rendern, dann nach DOMContentLoaded auf 1.15 (oder was) snappen — sichtbar. **Faustregel:** wenn ein State-Restore von einer CSS-Variable abhaengt, die `body { font-size: calc(...) }` mitkocht, IMMER zwei Restore-Punkte: einen pre-render fuer die CSS-Wirkung, einen post-render fuer alle abhaengigen UI-Elemente (Slider/Stepper/Indikatoren).

**Tests:** 59 neue in [`zerberus/tests/test_p_ui_9_two_axis_scaling.py`](zerberus/tests/test_p_ui_9_two_axis_scaling.py) — fuenf Klassen: `TestSourceWiringCSS` (13), `TestSourceWiringJSHelpers` (18), `TestSettingsUiWiring` (12), `TestKollisionMitVorgaengernPatches` (12), `TestPUiNineInlineMarker` (4). Alle 471 UI-relevanten Tests gruen lokal (test_p_ui_9 59 + test_p_ui_8 83 + test_p_ui_7 54 + test_p_ui_6 48 + test_p_ui_5 53 + test_p_ui_4 29 + test_p_ui_3 26 + test_p_ui_2 20 + test_p_ui_1 16 + test_settings_umbau 24 + test_nala_bubble_layout 15 + test_nala_adapter 14 + test_p203d3_nala_code_render 30). test_p203d3 `TestJsSyntaxIntegrity` (node --check) gruen — kein JS-Syntax-Fehler nach Edit. Pre-existing-Failures unveraendert: 1 `test_settings_umbau::test_mein_ton_nicht_mehr_in_sidebar` (Schulden #10 seit P-UI-4). Nicht durch P-UI-9 verursacht.

**Logging-Tag:** keiner — P-UI-9 ist reines Frontend-CSS+JS-Patch, kein neuer Server-Code-Pfad.

---

## Vorvorheriger Patch (Referenz)

**P-UI-8** — Phase 5c Schritt 8: Scroll-Navigationsleiste (Gabelungs-Design) (2026-05-08)

Achter Code-Patch der UI-Redesign-Phase. Vertikaler Strich am linken Bildschirmrand (außerhalb der iOS-Swipe-Back-Zone), gabelt sich zu einem kleinen Oval pro LLM-Antwort und führt wieder zusammen — organisch, „wie Nervenbahnen". Default unsichtbar; erscheint nach 1 s aktivem Scrollen mit `opacity: 0 → 0.25` über `0.3s ease`, verschwindet nach 2.5 s Stillstand. Klick-Delay 300 ms (verhindert versehentliches Springen während des Scrollens). Eigenständig scrollbar bei mehr Anchors als ins Viewport passen. Klick auf einen Anchor: Smooth-Scroll zum entsprechenden Bot-Wrapper via `scrollIntoView({behavior: 'smooth', block: 'start'})`. Disjunkter CSS-Namespace `.pui8-...` — keine Kollision mit P-UI-1..7 oder Bestands-Klassen.

**Architektur: Body-Child-Container + SVG-Bezier-Gabelung + Scroll-Detection mit drei Timern + idempotenter addMessage-internal-Hook + Source-Audit-Tests.**

- **CSS-Variablen** im `:root`: `--z-scroll-nav: 1500` (erste `--z-*`-Variable im Bestand — über Content, unter Sidebar/Login bei z-index 2000 und unter Modal bei 3000) und `--pui8-anchor-spacing: 40px` (DESIGN.md 11.1 fester Punkt-Abstand befüllt).
- **CSS-Block** in [`zerberus/app/routers/nala.py`](zerberus/app/routers/nala.py): `.pui8-scroll-nav` (Container, `position: fixed; left: 6px; top: 50%; transform: translateY(-50%); z-index: var(--z-scroll-nav); opacity: 0; pointer-events: none; transition: opacity 0.3s ease; max-height: 80dvh; overflow-y: auto; overscroll-behavior: contain`), `.pui8-scroll-nav.pui8-visible` (`opacity: 0.25` — DESIGN.md 11.1 "75% transparent"), `.pui8-scroll-nav.pui8-clickable` (`pointer-events: auto` — wird erst nach 300 ms Stillstand gesetzt). Anchor-Struktur: `.pui8-scroll-nav-anchor` (flex column, cursor pointer), `.pui8-scroll-nav-line.pui8-line-top` (1.5 px breit, 16 px hoch — Strich vor dem ersten Oval), `.pui8-scroll-nav-line.pui8-line-between` (1.5 px breit, `var(--pui8-anchor-spacing)` hoch — Strich zwischen Ovals), `.pui8-scroll-nav-fork` (16x10 px SVG-Container), `.pui8-scroll-nav-oval` (10x14 px, `border-radius: 50%`, `border: 1.5px solid currentColor`, dunkler Background). `.pui8-scroll-nav-anchor.pui8-active .pui8-scroll-nav-oval` (gold-Highlight, `var(--color-gold)`-Background — vorbereitet für IntersectionObserver-Active-State, im ersten Wurf noch nicht verdrahtet). Strich + Stroke verwenden `currentColor` für Theme-Faehigkeit. Hide bei offener Sidebar: `body.pui4-sidebar-open .pui8-scroll-nav { display: none; }`.
- **JS-Konstanten** `PUI8_FADE_IN_MS = 1000` (Spec "nach ~1 s aktivem Scrollen"), `PUI8_FADE_OUT_MS = 2500` (Spec "nach 2-3 s Scroll-Stillstand"), `PUI8_CLICK_DELAY_MS = 300` (Spec "verhindert versehentliches Springen"), `PUI8_SVG_NS = 'http://www.w3.org/2000/svg'`. State: `pui8_navEl` (Body-Child Container, lazy via `pui8_ensureNav`), `pui8_scrollContainer` (`#chatMessages`, lazy via `pui8_initScrollListener`), drei Timer-Refs (`pui8_fadeInTimer`/`pui8_fadeOutTimer`/`pui8_clickDelayTimer`), `pui8_anchorMap = new WeakMap()` (botWrapperEl → anchorEl, Idempotenz + automatische GC bei Wrapper-Removal).
- **JS-Helper `pui8_ensureNav()`**: erstellt einen `<div class="pui8-scroll-nav" data-pui8-scroll-nav="1" aria-hidden="true">` als Body-Child. Idempotent via `pui8_navEl && pui8_navEl.isConnected`. Body-Child-Position ist Pflicht: bei `body.pui4-sidebar-open` formt der CSS-Transform auf `.app-container` einen neuen containing block für `position: fixed` Descendants — Body-Child bleibt davon unbetroffen (gleiches Pattern wie P-UI-4-Sidebar).
- **JS-Helper `pui8_buildForkSvg(direction)`**: nutzt `createElementNS(PUI8_SVG_NS, 'svg')` — nicht `createElement('svg')`, weil das ein HTML-Element ohne SVG-Rendering produziert. `direction='out'` baut den oberen Fork (zwei Bezier-Kurven `M8 0 C8 4, 2 5, 2 5` und `M8 0 C8 4, 14 5, 14 5` — exakt aus dem Mockup), `direction='back'` den unteren (`M2 5 C2 5, 8 10, 8 10` und `M14 5 C14 5, 8 10, 8 10`). Stroke + Fill werden via CSS-Regel `.pui8-scroll-nav-fork path { stroke: currentColor; fill: none; stroke-width: 1.2; }` gesetzt — Theme-fähig.
- **JS-Helper `pui8_buildAnchor(targetEl)`**: erstellt `<div class="pui8-scroll-nav-anchor" data-pui8-anchor="1">` mit Fork-Out + Oval + Fork-Back als Children (vertikales Layout via flex column). Click-Listener prüft erst ob `pui8_navEl` die Klasse `pui8-clickable` hat — verhindert Klicks während des 300-ms-Click-Delay-Fensters. Wenn ja: `targetEl.scrollIntoView({behavior: 'smooth', block: 'start'})` springt zum Bot-Wrapper.
- **JS-Helper `pui8_addAnchor(botWrapperEl)`**: idempotent via WeakMap (`pui8_anchorMap.has(botWrapperEl)` → returnt vorhandenen Anchor). Ruft `pui8_initScrollListener()` lazy. Wenn der Nav-Container leer ist, fügt eine Top-Linie ein (`.pui8-line-top`, 16 px). Wenn das letzte Child ein Anchor ist (also schon mindestens einer existiert), fügt eine Between-Linie ein (`.pui8-line-between`, 40 px). Dann den neuen Anchor.
- **JS-Helper `pui8_handleScroll()`**: drei Timer mit `clearTimeout` davor (verhindert Race Conditions bei rapidem Scrollen). Fade-In: wenn nicht visible, setze nach 1 s die `.pui8-visible`-Klasse. Click-Delay: nach 300 ms Stillstand setze `.pui8-clickable` (entferne sie sofort bei jedem Scroll-Event). Fade-Out: nach 2.5 s Stillstand entferne beide Klassen.
- **JS-Helper `pui8_initScrollListener()`**: idempotent via `pui8_scrollContainer && pui8_scrollContainer.isConnected`. Holt `#chatMessages` und registriert `addEventListener('scroll', pui8_handleScroll, { passive: true })` — passive flag verhindert Scroll-Blocking.
- **Hook in `addMessage`** am Ende vor `return wrapper;`: für `sender === 'bot'` mit `typeof pui8_addAnchor === 'function'`-Check und try/catch fail-quiet ruft `pui8_addAnchor(wrapper)`. **addMessage-internal-Hook (analog P-UI-2/6) statt Caller-Side-Hook (analog P-UI-7)** — keine externen Daten gebraucht, also reicht der eine Hook der alle 5 Aufrufstellen abdeckt: sendMessage (Live-Bot-Antwort), loadSession (Reload), late-replay-Bubble, STT-Fehler-Bubble, Greeting-Bubble. addMessage-Signatur **unverändert** (`function addMessage(text, sender, tsOverride)`) — bewusste Architektur damit P-UI-2/6/7-Source-Audit-Tests, die exakt auf diesen Header splitten, weiter grün bleiben.
- **Source-Audit-Tests** in [`zerberus/tests/test_p_ui_8_scroll_nav.py`](zerberus/tests/test_p_ui_8_scroll_nav.py): fünf Klassen, 83 Tests. Slice-basiert, kein Browser/Playwright.

**Was P-UI-8 bewusst NICHT macht:**

- **IntersectionObserver für Active-Highlight (`pui8-active`-Klasse).** CSS-Regel `.pui8-scroll-nav-anchor.pui8-active .pui8-scroll-nav-oval` ist vorbereitet (gold-Background-Color). Im ersten Wurf wird die Klasse aber noch nicht JS-seitig gesetzt — der User sieht alle Ovals gleich gefärbt. Folge-Patch (UI-8-pre-2): IntersectionObserver auf alle Bot-Wrapper, das Anchor-Element zum aktuell sichtbaren Wrapper bekommt `pui8-active`.
- **Scrollbar-Sync mit Anchor-Position.** Wenn die Leiste eigenständig scrollbar wird (max-height 80dvh erreicht), gibt es keinen Sync zwischen Chat-Scroll-Position und Leist-Scroll-Position. Bei sehr langen Konversationen scrollt der User die Leiste manuell. Folge-Patch könnte die Leist-Scroll-Position automatisch zum aktiven Anchor zentrieren.
- **Anchor-Rückbau bei Bot-Bubble-Removal.** Aktuell bleiben Anchors für gelöschte Bot-Bubbles im Container. Da Nala aber keine Bubble-Removal-Aktion hat (Chat ist append-only), ist das aktuell kein Problem. Bei Chat-Wechsel via `loadSession` wird der ganze Chat-Container neu gerendert — die alten Anchors bleiben aber im Body-Child. Folge-Patch könnte einen Cleanup beim Chat-Wechsel triggern (Empfehlung: in `loadSession`-Body `pui8_navEl?.replaceChildren()` einfügen — minimaler Patch).
- **Hamburger ☰ → ✖-Animation bei offener Sidebar.** Bleibt auch in UI-8 ungebaut (Spec hat keine Aussage, Mini-Add für späteren Patch).
- **Hel-Splitscreen-Bug** (Schulden #9, eigener Folge-Patch).
- **`test_settings_umbau::test_mein_ton_nicht_mehr_in_sidebar`-Fix** (Schulden #10, eigener Mini-Fix-Patch).
- **DESIGN.md-`[WERT]`-Befüllung in 3.4 für andere Z-Indizes** außer `--z-scroll-nav`. Die anderen Z-Index-Variablen (`--z-base`, `--z-sticky`, `--z-dropdown`, `--z-backdrop`, `--z-modal`, `--z-toast`, `--z-tooltip`) sind weiter `[WERT]` in DESIGN.md. Folge-Patch (UI-9 oder UI-Sammler) kann den vollständigen Stack einpflegen, sobald Bestand-Z-Indices auf Variablen migrieren.

**Lessons (3):**

1. **`addMessage`-internal-Hook vs. Caller-Side-Hook — wann welches Pattern.** P-UI-7-Lesson sagte: Caller-Side-Hook ist sauberer als Callee-Side-Hook (Param-Erweiterung), weil Source-Audit-Tests an der addMessage-Signatur splitten. P-UI-8 ergänzt: der Caller-Side-Hook ist nur dann der saubere Pfad, wenn der Caller einzigartige Daten hat (wie `modelId` bei P-UI-7) die addMessage nicht selbst kennt. Wenn der Hook **keine externen Daten** braucht (wie bei P-UI-2/6/8: nur den frisch erstellten Wrapper), ist der addMessage-internal-Hook am Ende der Funktion **kürzer und vollständiger** — er deckt alle Aufruf-Stellen automatisch ab (sendMessage, loadSession, STT-Fehler, late-replay, Greeting), ohne dass jeder Caller manuell den Hook duplizieren muss. Beide Patterns halten die Signatur invariant. **Faustregel:** Caller-Side wenn externe Daten → Callee-Internal sonst.

2. **Body-Child-Position ist Pflicht für `position: fixed`-Layer auf transformed Containern.** P-UI-4-Lesson hatte das schon dokumentiert (Sidebar als Body-Child wegen `transform`-containing-block-Regel). P-UI-8 wendet das gleiche Pattern für die Scroll-Nav an — aber zusätzlich noch ein Layer: `body.pui4-sidebar-open .pui8-scroll-nav { display: none; }`. Begründung: selbst wenn die Scroll-Nav als Body-Child Viewport-fixed bleibt, würde sie bei offener Sidebar links neben den 280-px-shifted Content schwebend wirken — unschön. Beste Lösung ist die Leiste komplett auszublenden während der Sidebar — der User braucht die Sprung-Navigation eh nicht wenn er gerade die Sidebar bedient. **Faustregel:** Wenn ein Body-Child fixed-Layer mit einem anderen Body-Child fixed-Layer kollidieren könnte (Sidebar + Scroll-Nav), CSS-State-Mutex via `body.<state>` einbauen.

3. **`createElementNS` ist Pflicht für SVG-Inline-DOM, nicht `createElement('svg')`.** Erste Idee für die Bezier-Gabelung war `document.createElement('svg')` + `setAttribute('width', '16')` etc. — Browser akzeptiert das, parst es aber als HTML-Element ohne SVG-Rendering. Pfade werden nicht gezeichnet, der Container bleibt leer. Korrekt: `document.createElementNS('http://www.w3.org/2000/svg', 'svg')` (und entsprechend für `path`-Children). Dasselbe gilt für jedes andere XML-Namespace-Element (MathML etc.). **Faustregel:** wenn ein Element zu einem anderen XML-Namespace gehört, IMMER `createElementNS` mit dem korrekten Namespace-URI verwenden — sonst gibt es kein Crash, nur stilles Nicht-Rendern. Backstop: `test_build_fork_createelementns` prüft den `createElementNS(PUI8_SVG_NS, ...)`-Aufruf im Source.

**Tests:** 83 neue in [`zerberus/tests/test_p_ui_8_scroll_nav.py`](zerberus/tests/test_p_ui_8_scroll_nav.py) — fünf Klassen: `TestSourceWiringCSS` (29), `TestSourceWiringJSHelpers` (28), `TestAddMessageHook` (6), `TestKollisionMitVorgaengernPatches` (12), `TestPUiEightInlineMarker` (4). Alle 388 UI-relevanten Tests grün lokal (test_p_ui_8 83 + test_p_ui_7 54 + test_p_ui_6 48 + test_p_ui_5 53 + test_p_ui_4 29 + test_p_ui_3 26 + test_p_ui_2 20 + test_p_ui_1 16 + test_nala_bubble_layout 15 + test_nala_adapter 14 + test_p203d3_nala_code_render 30). test_p203d3 `TestJsSyntaxIntegrity` (node --check) grün — kein JS-Syntax-Fehler nach Edit (insbesondere keine `\u`-Escape-Trap, kein Backtick-Template-Literal-Konflikt). Pre-existing-Failures unverändert: 4 `test_nala_projects_tab::TestNalaProjectsEndpoint`-ERRORS (`config.yaml`-Drift im Worktree, Memory) + 1 `test_settings_umbau::test_mein_ton_nicht_mehr_in_sidebar` (Schulden #10 seit P-UI-4). Nicht durch P-UI-8 verursacht.

**Logging-Tag:** keiner — P-UI-8 ist reines Frontend-CSS+JS-Patch, kein neuer Server-Code-Pfad.

---

## Vorvorvorheriger Patch (Referenz)

**P-UI-7** — Phase 5c Schritt 7: LLM-Icon-Anzeige (2026-05-08)

Siebter Code-Patch der UI-Redesign-Phase. Jede LLM-Antwort bekommt ein kleines Icon (22 px Default, 24 px Touch, `border-radius: 6px`) links oben in der Bot-Bubble, links neben dem P-UI-2-Collapse-Toggle. Das Icon zeigt das tatsächlich verwendete Modell — primär als Provider-Logo via OpenRouter-CDN-URL (`https://openrouter.ai/images/icons/<provider>.svg`), bei fehlender oder fehlerhafter URL als Buchstaben-Badge mit deterministischer HSL-Background-Color aus dem Modell-Hash. Wenn der Intent-Router (UI-11 später) auf ein anderes Modell umschaltet, wechselt das Icon automatisch — "Show, don't tell": Modellwechsel sichtbar machen ohne Text/Toast. Disjunkter CSS-Namespace `.pui7-...` — keine Kollision mit P-UI-1..6 oder Bestands-Klassen.

**Architektur: Frontend-only Provider-Slug-Extraktion + Image-mit-onerror-Fallback + localStorage-Cache + idempotenter Attach-Helper + Source-Audit-Tests.**

- **CSS-Block** in [`zerberus/app/routers/nala.py`](zerberus/app/routers/nala.py): `.pui7-llm-icon` (Container, `display: inline-flex` + `flex: none` damit das Icon im natürlichen Flow vor dem Collapse-Toggle sitzt — bricht keine bestehende P-UI-2-Mechanik), `.pui7-llm-icon-img` (`width/height: 100%; object-fit: contain`), `.pui7-llm-icon-badge` (Default `display: none`, im Fallback-Fall sichtbar). Fallback-Modifier `.pui7-llm-icon.pui7-icon-fallback`: kippt Image versteckt → Badge sichtbar (`display: inline-block`). Mobile-Media-Query `(hover: none) and (pointer: coarse)` setzt das Icon auf 24 px (besseres Touch-Target).
- **JS-Konstanten** `PUI7_ICON_BASE = 'https://openrouter.ai/images/icons/'` + `PUI7_ICON_CACHE_KEY = 'pui7_icon_cache_v1'`.
- **localStorage-Cache** `pui7_iconCache` wird beim Modul-Load aus `localStorage.getItem(PUI7_ICON_CACHE_KEY)` initialisiert; Helper `pui7_persistIconCache()` schreibt zurück. Cache-Werte: `'ok'` (Image hat geladen) oder `'fail'` (Image-Load fehlgeschlagen). Wenn URL vorab als `'fail'` gecacht ist, wird das Image-Element gar nicht erst erzeugt — spart Network-Retry bei jedem Render.
- **JS-Helper `pui7_getProvider(modelId)`**: defensive für leeren/non-string-Input, splittet am ersten `/`, liefert Provider-Slug lowercase (z.B. `"deepseek/deepseek-chat-v3.5"` → `"deepseek"`).
- **JS-Helper `pui7_iconUrlForProvider(provider)`**: filtert Provider-Slug per `replace(/[^a-z0-9_-]/g, '')` (Injection-Schutz), baut `PUI7_ICON_BASE + safe + '.svg'`. Leerer Slug → leere URL.
- **JS-Helper `pui7_modelLabel(modelId)`**: liefert Buchstaben-Badge — Special-Cases `DS` (DeepSeek), `R1` (DeepSeek-R1), `Cl` (Claude/Anthropic), `GP` (OpenAI/GPT), `Ge` (Google/Gemini), `Ll` (Meta/Llama), `Mi` (Mistral), `Qw` (Qwen). Sonst erste 1-2 Buchstaben des Provider-Slugs (capitalized). Letzter Fallback `'?'`.
- **JS-Helper `pui7_hashHue(s)`**: djb2-ähnlicher Bit-Shift-Hash modulo 360 — deterministische HSL-Hue für die Background-Color des Badges (gleicher Modell-Slug → gleiche Farbe, immer und überall).
- **JS-Helper `pui7_buildLLMIcon(modelId)`**: erstellt `<span class="pui7-llm-icon" data-pui7-icon="1" data-pui7-model="<modelId>">` (Audit-Hooks für DevTools) mit HSL-Background-Color via `pui7_hashHue(modelId)` und `title="<modelId>"` (Hover-Tooltip). Innen: ein `<span class="pui7-llm-icon-badge">` mit dem Badge-Text via `textContent` (XSS-sicher), und — wenn URL vorhanden + nicht `'fail'`-gecached — ein `<img class="pui7-llm-icon-img" loading="lazy">` mit `addEventListener('error')` (setzt `pui7-icon-fallback` + cacht `'fail'` + persistiert) und `addEventListener('load')` (cacht `'ok'` + persistiert). Wenn keine URL oder `'fail'`-gecached, wird das Image gar nicht erzeugt und die Fallback-Klasse direkt gesetzt.
- **JS-Helper `pui7_attachLLMIcon(wrapperEl, modelId)`**: idempotent — findet `.message.bot-message` im Wrapper, entfernt etwaige alte `.pui7-llm-icon`-Elemente, baut neues Icon via `pui7_buildLLMIcon`, fügt es per `insertBefore(icon, .bubble-collapse-toggle)` direkt vor dem Collapse-Toggle ein (Spec: "links oben, neben dem Collapse-Toggle"). Fallback wenn kein Toggle: `insertBefore(icon, msgDiv.firstChild)`.
- **Caller-Hook** in der `/v1/chat/completions`-Antwort-Verarbeitung (sendMessage-Pfad): `const replyModel = data.model || data.allowed_model || null` extrahiert die Modell-ID aus dem OpenAI-kompatiblen Response (`data.model`) bzw. dem Hel-spezifischen Fallback (`data.allowed_model`). Wenn `replyModel` nicht-leer und `pui7_attachLLMIcon` definiert → `pui7_attachLLMIcon(botWrapper, replyModel)` fail-quiet (try/catch wie P192/P203d-3-Pattern).
- **addMessage-Signatur unverändert** (`function addMessage(text, sender, tsOverride)`) — bewusste Architektur. Wenn ich die Signatur um einen 4. Param `modelId` erweitere, brechen die P-UI-6-Tests, die exakt auf diesen Header splitten (`src.split("function addMessage(text, sender, tsOverride)", 1)[1]`). Stattdessen: separater `pui7_attachLLMIcon`-Helper, der nach `addMessage(reply, 'bot')` aufgerufen wird. Das hat den Bonus, dass der Hook idempotent + nachrüstbar bleibt — UI-11 könnte später bei Reasoning-Switch das Icon einfach erneut attachen.
- **Source-Audit-Tests** in [`zerberus/tests/test_p_ui_7_llm_icon.py`](zerberus/tests/test_p_ui_7_llm_icon.py): fünf Klassen, 54 Tests. Slice-basiert, kein Browser/Playwright.

**Was P-UI-7 bewusst NICHT macht:**

- **OpenRouter-Modell-Liste laden (`/api/v1/models`-Endpoint).** Spec NALA_UI_REDESIGN_PROMPT.md Sektion 6 schlägt vor "beim Laden der Modell-Liste aus OpenRouter die Icon-URLs cachen". Aktuelle Implementation kennt keine Modell-Liste — sie baut die Provider-Logo-URL deterministisch aus dem Provider-Slug der Modell-ID. Das spart einen Request-Pfad und einen Cache-State, hat dafür aber den Nachteil dass nicht jedes OpenRouter-Modell eine `<provider>.svg`-Datei hat. Das `<img onerror>`-Fallback fängt das auf und cacht `'fail'`, sodass jedes nicht-verfügbare Provider-Logo nach einem einzigen Network-Roundtrip dauerhaft auf den Buchstaben-Badge fällt. Folge-Patch (UI-7-pre-2) kann optional die OpenRouter-Modell-Liste laden und Icon-URLs aus `architecture.icon` o.ä. extrahieren.
- **Reload-Bubbles aus `loadMessages` mit Icon ausstatten.** Backend speichert das Modell pro Message nicht (`store_interaction()` in `legacy.py` legt nur Rolle + Content + Timestamp ab). Reload-Bot-Bubbles bekommen kein Icon — bewusst akzeptierte Inkonsistenz für ersten UI-7-Wurf. Folge-Patch (UI-7-pre-3 oder Backend-Patch in der Patch-11x-Serializer-Familie) würde das Modell pro Message persistieren und beim Reload wieder anhängen.
- **Reasoning-Switch live triggern.** Das Icon wird bei jedem `/v1/chat/completions`-Response gesetzt; wenn der Intent-Router (UI-11) ein anderes Modell wählt als das Default-Modell, kommt das im `data.model`-Feld korrekt durch und das Icon spiegelt es automatisch. UI-7 macht **keine** Live-Animation des Icon-Wechsels (z.B. Cross-Fade) — der Wechsel ist instant beim nächsten Render-Tick.
- **Hamburger ☰ → ✖-Animation bei offener Sidebar.** Bleibt auch in UI-7 ungebaut (Spec hat keine Aussage, Mini-Add für späteren Patch).
- **Hel-Splitscreen-Bug** (Schulden #9, eigener Folge-Patch).
- **`test_settings_umbau::test_mein_ton_nicht_mehr_in_sidebar`-Fix** (Schulden #10, eigener Mini-Fix-Patch).
- **DESIGN.md-`[WERT]`-Befüllung in Sektion 9.** Tabelle 9 verwendet bereits konkrete Werte (Position, Quelle, Fallback, Größe 20-24 px, Verhalten bei Reasoning) — keine offenen Platzhalter. Nur Changelog-Zeile.

**Lessons (3):**

1. **`onerror`-Fallback + localStorage-Cache verwandelt unsichere CDN-Bilder in robuste UI-Komponenten.** OpenRouter-CDN liefert Provider-Logos unter `https://openrouter.ai/images/icons/<provider>.svg`, aber nicht für jedes Provider-Kürzel — manche Modelle haben keinen Eintrag, andere wechseln Pfad-Konventionen. Statt vor jedem Icon-Render eine HEAD-Request zu senden oder eine kuratierte Whitelist zu pflegen, vertraut die Implementation dem Browser: `<img>` versucht zu laden, bei Fehlschlag feuert `addEventListener('error')` und kippt auf den Buchstaben-Badge — plus localStorage-Cache merkt sich den Fehlschlag pro URL, sodass beim nächsten Render kein Network-Retry mehr läuft. Erfolgreiche Loads cachen `'ok'`. Generalisierbar: **bei externen Bild-Quellen mit unklarer Coverage ist ein Image-mit-onerror-Fallback + Persistenz-Cache der saubere Ersatz für eine zentrale Whitelist** — der Cache wächst organisch mit der tatsächlichen User-Nutzung, statt eine veraltende Liste zu pflegen.

2. **Hierarchische Hooks ohne Signatur-Änderung halten Source-Audit-Tests stabil.** Die intuitivste Architektur für UI-7 wäre ein vierter Param `modelId` an `addMessage(text, sender, tsOverride, modelId)`. Aber die P-UI-6-Tests splitten exakt auf `function addMessage(text, sender, tsOverride)` — eine Signatur-Erweiterung würde 7 Tests in `test_p_ui_6_reasoning_block.py::TestAddMessageHook` brechen, die mit dem Slice arbeiten. Lösung: separater `pui7_attachLLMIcon`-Helper, der **nach** `addMessage(reply, 'bot')` aus dem Caller heraus gerufen wird. Vorteile: (1) addMessage-Signatur unverändert, (2) Hook idempotent (mehrfacher Aufruf ist no-op), (3) UI-11 kann später denselben Helper bei Reasoning-Switch einfach erneut rufen, ohne addMessage anzufassen. Generalisierbar: **wenn Tests an einer Funktions-Signatur splitten und ein neuer Patch zusätzliches Verhalten an dieselbe DOM-Stelle koppeln will, ist der Caller-Side-Hook (vom Aufrufer aus) sauberer als der Callee-Side-Hook (zusätzlicher Param) — die Signatur bleibt invariant, die Tests bleiben stabil.**

3. **Inline-flex + `flex: none` löst das "Icon vor Inline-Element"-Problem ohne `position: absolute`.** P-UI-2 setzt `.bubble-collapse-toggle` als `display: inline-block; margin-right: 6px;` — das Element sitzt im natürlichen Flow als erstes Child von `msgDiv`. Für UI-7 wollte ich das Icon **vor** dem Toggle haben, ohne den Toggle anzufassen. Erste Idee: `position: absolute; top: 6px; left: 6px;` (Toggle wäre dann `left: 36px;`). Problem: zwei absolute Positionen wären gegenseitig blind und brechen sofort, wenn Padding/Spacing der Bubble sich ändert. Lösung: `display: inline-flex` + `flex: none` + `margin-right: 6px` macht das Icon zu einem **Inline-Container mit fester Größe**, der im natürlichen Flow vor dem Toggle landet — ohne absolute Positionierung, ohne Layout-Brüche. `flex: none` verhindert Schrumpfen bei beengten Bubbles. Generalisierbar: **wenn ein neues Inline-Element vor einem bestehenden Inline-Element platziert werden soll, ist `display: inline-flex` + `flex: none` der saubere Pfad — `position: absolute` ist Coupling-Falle und sollte nur für tatsächlich überlappende Layer (Tooltips, Popovers, Modal-Overlays) verwendet werden.**

**Tests:** 54 neue in [`zerberus/tests/test_p_ui_7_llm_icon.py`](zerberus/tests/test_p_ui_7_llm_icon.py) — fünf Klassen: `TestSourceWiringCSS` (12), `TestSourceWiringJSHelpers` (20), `TestCallerHook` (5), `TestKollisionMitVorgaengernPatches` (13), `TestPUiSevenInlineMarker` (4). Alle 305 UI-relevanten Tests grün lokal (test_p_ui_7 54 + test_p_ui_6 48 + test_p_ui_5 53 + test_p_ui_4 29 + test_p_ui_3 26 + test_p_ui_2 20 + test_p_ui_1 16 + test_nala_bubble_layout 15 + test_nala_adapter 14 + test_p203d3_nala_code_render 30). test_p203d3 `TestJsSyntaxIntegrity` (node --check) grün — kein JS-Syntax-Fehler nach Edit. Pre-existing-Failures unverändert: 4 `test_nala_projects_tab::TestNalaProjectsEndpoint`-ERRORS (`config.yaml`-Drift im Worktree, Memory) + 1 `test_settings_umbau::test_mein_ton_nicht_mehr_in_sidebar` (Schulden #10 seit P-UI-4). Nicht durch P-UI-7 verursacht.

**Logging-Tag:** keiner — P-UI-7 ist reines Frontend-CSS+JS-Patch, kein neuer Server-Code-Pfad.

---

## Aeltere Patches (Referenz)

**P-UI-6** — Phase 5c Schritt 6: Reasoning-Block-Styling (2026-05-08)

Sechster Code-Patch der UI-Redesign-Phase. Wenn die LLM-Antwort einen `<think>...</think>`-Block enthält (DeepSeek R1, generischer Pattern, OpenRouter-Reasoning-Modelle die das im content-string mitliefern), wird der Block jetzt separat oben in der Bot-Bubble gerendert — default eingeklappt (User sieht primär die Antwort), Toggle "Gedankengang" mit Dreieck-Icon klappt auf. Aufgeklappter Reasoning-Text ist nach rechts eingerückt mit halbtransparentem linken Strich, reduzierter Opacity (0.75) und reduzierter Saturation (`filter: saturate(0.7)`) — Spec DESIGN.md Sektion 8.3 "wirkt wie ein Gedanke, nicht ganz stofflich". Klarer visueller Bruch zur eigentlichen Antwort darunter (volle Opacity, normales Styling). KEINE Kollision mit der bestehenden P213-Reasoning-Stepper-Card (`.reasoning-card`/`.reasoning-step`/...) die die Pipeline-Phasen vor/während der Antwort zeigt — UI-6 zeigt den tatsächlichen Modell-Reasoning-Output, P213 zeigt Stepper-Phasen. Disjunkter CSS-Namespace `.pui6-...`.

**Architektur: Frontend-only Extract-and-Render-Pipeline + Source-Audit-Tests.**

- **CSS-Block** in [`zerberus/app/routers/nala.py`](zerberus/app/routers/nala.py): `.pui6-reasoning-block` (Container, default `pui6-collapsed`-Klasse), `.pui6-reasoning-toggle` (Klick-Header mit `cursor: pointer`, `min-height: 28px`, `padding: 4px 8px`, Hover-Background `rgba(139,148,158,0.10)`), `.pui6-reasoning-arrow` (Dreieck mit `transition: transform 0.2s ease`), `.pui6-reasoning-label` ("Gedankengang"-Schrift), `.pui6-reasoning-content` (`margin-left: 16px` + `padding-left: 12px` + `border-left: 2px solid rgba(139,148,158,0.2)` + `opacity: 0.75` + `filter: saturate(0.7)` + `color: #9e9e9e` + `font-size: 0.92em` + `white-space: pre-wrap`). Default-Collapsed-Regel `.pui6-reasoning-block.pui6-collapsed .pui6-reasoning-content { display: none; }`. Aufgeklappt-Arrow-Rotation-Regel `.pui6-reasoning-block:not(.pui6-collapsed) .pui6-reasoning-arrow { transform: rotate(90deg); }`.
- **JS-Konstante `PUI6_THINK_REGEX = /<think>([\\s\\S]*?)<\\/think>/i`** — non-greedy (matcht den ersten `<think>`-Block separat statt alles bis zum letzten `</think>` zu verschlucken), case-insensitive (`/i`-Flag), multiline (`[\s\S]` matcht auch Newlines).
- **JS-Helper `pui6_extractReasoning(text)`**: defensive für leeren/non-string-Input, matcht den Regex, liefert `{ reasoning: <getrimmt>, content: <ohne-think-Block-getrimmt> }`. Wenn kein Match: `{ reasoning: '', content: text }`.
- **JS-Helper `pui6_buildReasoningBlock(reasoningText)`**: erstellt `<div class="pui6-reasoning-block pui6-collapsed" data-pui6-reasoning="1">` mit `<button class="pui6-reasoning-toggle" data-pui6-toggle="1" aria-expanded="false">` (enthält `<span class="pui6-reasoning-arrow">▶</span>` + `<span class="pui6-reasoning-label">Gedankengang</span>`) + `<div class="pui6-reasoning-content">{reasoningText}</div>`. Click-Listener auf Toggle: `ev.stopPropagation()` (gegen P139-Action-Toolbar / P-UI-2-Bubble-Tap-Konflikt), `block.classList.toggle('pui6-collapsed')` und synct `aria-expanded`. Reasoning-Text via `textContent` gesetzt — XSS-sicher, kein `innerHTML`.
- **`addMessage(text, sender, ...)`-Hook** für `sender === 'bot'`: extrahiert vor dem Render-Pfad `_pui6Reasoning` und `_pui6Content`. Ruft `renderBotContent(fullDiv, _pui6Content)` mit cleaned content (sonst landet der `<think>`-Block doppelt im DOM). Wenn `_pui6Reasoning` nicht-leer: `fullDiv.insertBefore(pui6_buildReasoningBlock(_pui6Reasoning), fullDiv.firstChild)` prepend-et den Block — Spec "Position: über der eigentlichen Antwort". `previewSpan.textContent` zeigt für Bot `pui2_firstLine(_pui6Content)` (sonst startet die Preview mit "<think>..."). User-Branch + `chatMessages.push(text)` unverändert — Reload behält den Original-Text mit `<think>`-Block, Reasoning wird beim Re-Render automatisch wieder extrahiert.
- **Source-Audit-Tests** in [`zerberus/tests/test_p_ui_6_reasoning_block.py`](zerberus/tests/test_p_ui_6_reasoning_block.py): fünf Klassen, 48 Tests. Slice-basiert, kein Browser/Playwright. Plus 1 P-UI-2-Test angepasst: `test_user_lastline_bot_firstline_preview` akzeptiert jetzt sowohl `pui2_firstLine(text)` als auch `pui2_firstLine(_pui6Content)` (User-Branch unverändert, Bot-Branch nutzt cleaned content).

**Was P-UI-6 bewusst NICHT macht:**

- **Backend-`thinking`-Parameter separat liefern.** Spec NALA_UI_REDESIGN_PROMPT.md Sektion 5 erlaubt sowohl `<think>`-Tags im Content als auch separaten `thinking`-Parameter. Aktuell kommt der LLM-Reasoning-Output (DeepSeek R1, OpenRouter-Reasoning-Modelle) im Content-String mit eingebetteten `<think>`-Tags. Hel-Adapter ist unverändert. Wenn das Backend künftig den Reasoning-Block separat liefert (z.B. als `data.reasoning_text`), wird das Frontend mit minimalem Patch erweitert (Helper-Funktion existiert, nur die Insertion-Source wechselt).
- **Reasoning-Streaming.** Aktuell wird der Reasoning-Block einmalig nach Antwort-Vollständigkeit gerendert. Live-Streaming des Reasoning-Outputs (Token-für-Token-Update) wäre Folge-Patch — braucht Backend-Stream-Support für separaten Reasoning-Channel.
- **Reasoning-Persistenz im Backend.** `chatMessages.push(text)` speichert weiter den Original-Text mit `<think>`-Tags. Beim Reload wird neu extrahiert. Backend-Serializer (Patch 11x-Familie) bleibt unangetastet.
- **Sentiment-Ambient auf der Reasoning-Box.** Phase 5c Schritt 10 (UI-10) ist explizit für Sentiment-Ambient-Lighting — UI-6 fokussiert nur auf den Block-Style.
- **Mobile-Tap-Visualfeedback (z.B. Haptic).** Toggle nutzt CSS-Hover/Active für visuelles Feedback. Haptic ist Browser-API-Topic für späteren Patch.
- **`docs/DESIGN.md`-`[WERT]`-Befüllung in Sektion 8.3.** Tabelle 8.3 verwendet bereits konkrete Werte (`opacity: 0.7-0.8`, `filter: saturate(0.7)`, `--space-md`-Einrückung, `--border-color-light` für den Strich) — keine offenen Platzhalter zu füllen.
- **Hel-Splitscreen-Bug** (Schulden #9, eigener Folge-Patch).
- **`test_settings_umbau::test_mein_ton_nicht_mehr_in_sidebar`-Fix** (Schulden #10, eigener Mini-Fix-Patch).

**Lessons (3):**

1. **Disjunkter CSS-Klassen-Namespace verhindert Kollision mit ähnlich benannten Bestands-Klassen.** Die Phase-5a-Reasoning-Stepper-Card aus P213 nutzt `.reasoning-card`/`.reasoning-toggle`/`.reasoning-list`/`.reasoning-step` mit eigener Polling-Mechanik und eigenem CSS-Block. P-UI-6 baut etwas semantisch Ähnliches (Reasoning-Block mit Toggle), aber auf einer ANDEREN Datenebene (Modell-Reasoning-Output statt Pipeline-Schritt-Audit). Statt `.reasoning-block`/`.reasoning-toggle-2` etc. wurde der Namespace explizit `pui6-reasoning-...` gewählt — das macht das Source-Diff-Auditing trivial (alle UI-6-Selektoren haben das `pui6-`-Präfix), und Anti-Pattern-Tests stellen sicher, dass die alten P213-Klassen nicht versehentlich im neuen CSS-Block auftauchen. Generalisierbar: **bei thematisch verwandten Patches in gleicher Datei lieber Namespace-Präfix als Suffix-Variation** — `pui6-` zeigt sofort die Patch-Herkunft, `-2`/`-new` sind nichtssagend.

2. **Frontend-Extraktion via Regex statt Backend-Erweiterung — wenn der Output schon im Content-String steht, muss man ihn nicht doppelt durch die Pipeline schicken.** DeepSeek R1 und OpenRouter-Reasoning-Modelle liefern den `<think>`-Block bereits als Teil des content-Strings. Eine Backend-Erweiterung (separates `reasoning_text`-Feld in der `/v1/chat/completions`-Response, Adapter-Anpassung in `nala_adapter.py`, Hel-Routing-Update) hätte 4–6 Files berührt für Information, die schon da war. Stattdessen: ein 4-Zeilen-Regex `/<think>([\s\S]*?)<\/think>/i` im Frontend, ein Helper, ein Hook in `addMessage` — Patch berührt nur `nala.py`. Generalisierbar: **prüfen ob die Information schon im Datenstrom mitkommt, bevor man Backend-Schemas erweitert.**

3. **Defensive `stopPropagation` an Toggle-Buttons innerhalb klick-aktiver Wrapper ist Pflicht, nicht Cosmetic.** Das `pui6-reasoning-toggle`-Button sitzt innerhalb der `.bubble-full`/`.message`-Bubble, die ihrerseits diverse Click-Listener hat: P139-Action-Toolbar-Toggle (`attachActionToggle` macht 5 s-Sichtbarkeit auf Tap), P-UI-2-User-Bubble-Auto-Expand (capture-phase-Listener, der `collapsed-v2` toggelt). Ohne `ev.stopPropagation()` im Click-Listener des Reasoning-Toggles würde ein Klick auf "Gedankengang" GLEICHZEITIG die Action-Toolbar einblenden und (bei User-Bubbles) das Auto-Collapse togglen — visuell unschön und semantisch falsch (User wollte den Reasoning-Block aufklappen, nicht die ganze Bubble manipulieren). Generalisierbar: **jeder Klick-Listener auf einem Inner-Element innerhalb einer click-aktiven Bubble braucht `stopPropagation` — sonst kaskadiert der Click durch die ganze Listener-Kette.**

**Tests:** 48 neue in [`zerberus/tests/test_p_ui_6_reasoning_block.py`](zerberus/tests/test_p_ui_6_reasoning_block.py) — fünf Klassen: `TestSourceWiringCSS` (13), `TestSourceWiringJSHelpers` (14), `TestAddMessageHook` (7), `TestKollisionMitVorgaengernPatches` (10), `TestPUiSixInlineMarker` (4). Plus 1 P-UI-2-Test angepasst (`test_user_lastline_bot_firstline_preview` akzeptiert jetzt sowohl `pui2_firstLine(text)` als auch `pui2_firstLine(_pui6Content)`). Alle 251 UI-relevanten Tests grün lokal (test_p_ui_6 48 + test_p_ui_5 53 + test_p_ui_4 29 + test_p_ui_3 26 + test_p_ui_2 20 + test_p_ui_1 16 + test_nala_bubble_layout 15 + test_nala_adapter 14 + test_p203d3_nala_code_render 30). Pre-existing-Failures unverändert: 4 `test_nala_projects_tab::TestNalaProjectsEndpoint`-ERRORS (`config.yaml` fehlt im Worktree, Memory) + 1 `test_settings_umbau::test_mein_ton_nicht_mehr_in_sidebar` (Schulden #10 seit P-UI-4). Nicht durch P-UI-6 verursacht.

**Logging-Tag:** keiner — P-UI-6 ist reines Frontend-CSS+JS-Patch, kein neuer Server-Code-Pfad.

---

## Frueherer Patch (Referenz, ausserhalb Hot-Window)

**P-UI-5** — Phase 5c Schritt 5: Projektseite als eigene View (2026-05-08)

Fünfter Code-Patch der UI-Redesign-Phase. Der `📁 Projekte`-Sidebar-Button (P-UI-4 als Übergangs-Click) routet jetzt auf eine eigene `#projects-screen`-View, die Geschwister von `#chat-screen` und `#login-screen` im `.app-container` ist und denselben Container-Shift bei offener Sidebar mitnutzt. Die View hat einen Header (Back-Button → `closeProjectsView()` + Titel "📁 Projekte"), eine Toolbar (Live-Suchfeld `#projects-search` mit Lupe-Placeholder + Sort-Toggle "Letzter Zugriff" / "Name" — Default `recent` per `updated_at` desc), und einen Scroll-Bereich (Pull-Zone für "Neues Projekt"-Hint, Aktiv-Display, Pinned-Sektion mit 📌-Heading und goldenem `border-left`, Reguläre Sektion). Settings-Tab "📁 Projekte" + Panel + alte Render-Funktionen sind komplett entfernt — Spec DESIGN.md Sektion 13: "Projekte raus aus Einstellungen". Active-Project-Chip im Header onclick → `openProjectsView()` (vorher `openSettingsModal(); switchSettingsTab('projects')`).

**Architektur: neuer Screen-DIV im .app-container + Render-Pipeline mit Cache + Live-Search + Sort-Toggle + Pin-State (localStorage) + Pull-Gesture + Source-Audit-Tests.**

- **CSS-Block `#projects-screen`** in [`zerberus/app/routers/nala.py`](zerberus/app/routers/nala.py): default `display: none; flex-direction: column; height: 100%;` plus knapp 200 Zeilen für Header/Toolbar/Search/Sort-Toggle/Pull-Zone/Project-Card/Pinned-Border/Empty-State. `.project-card.pinned` bekommt `border-left: 4px solid var(--color-gold)`. `.project-card.active` gold-Border + gold-Hintergrund. `.projects-pull-zone` default `max-height: 0` mit Transition; `.projects-pull-zone.revealed` expandiert auf 60px.
- **HTML-Block `<div id="projects-screen">`** als Geschwister von `<div id="chat-screen">` im `.app-container`. Aufbau: Header (Back-Button `←` + Titel) → Toolbar (Search-Input + Sort-Buttons mit `data-sort="recent"` und `data-sort="name"`) → Scroll-Container (Pull-Zone mit "Neues Projekt"-Hint, Active-Display, Pinned-Sektion versteckt-bei-leer, Regular-Sektion).
- **Settings-Tab-Button + Tab-Panel entfernt** — `data-tab="projects"`-Button und kompletter `#settings-tab-projects`-Panel-Block weg. `switchSettingsTab(tab)` hat keinen `if (tab === 'projects')`-Branch mehr.
- **Active-Project-Chip onclick umgeleitet** auf `openProjectsView()`. Alte Patch-201-Routing (`openSettingsModal(); switchSettingsTab('projects')`) komplett raus.
- **JS-Funktionen neu**: `openProjectsView()` (View einblenden, Sidebar schließen, `loadNalaProjects()` triggern), `closeProjectsView()` (zurück zum Chat), `renderProjectsView()` (Filter + Sort + Split pinned/regular, rendert in beide Sektionen), `pui5_buildProjectCardHtml(p, isPinned, isActive)` (Card-HTML mit Pin-Button), `pui5_formatProjectTimestamp(iso)` (de-Datum), `getPinnedProjectIds()/setPinnedProjectIds()/toggleProjectPin(id, ev)` (localStorage `nala_pinned_projects`), `setProjectsSortMode(mode)` ('recent' | 'name', toggelt Active-Klasse), `handleProjectTap(id)` (aktivieren + zurück zum Chat), `handleNewProjectHint()` (Anti-Clutter-Alert "Anlegen geht über Hel"), `pui5_revealPullZone()/pui5_attachPullGesture()` (touchstart/touchmove/wheel-Listener auf `#projects-scroll`, threshold 80 px für 5 s revealing), `pui5_attachProjectsSearch()` (input-Listener), `pui5_initProjectView()` (init-Hook, DOMContentLoaded oder sofort).
- **`loadNalaProjects()` umgebaut**: rendert in die neue View statt in den alten Settings-Tab. Alte `renderNalaProjectsList(items)` und `renderNalaProjectsActive()` entfernt. Aktiv-Display-Logik in `renderProjectsView()` integriert. Zombie-ID-Schutz aus Patch 201 bleibt.
- **`setActiveProject`/`clearActiveProject`** rufen jetzt `renderProjectsView()` defensive (typeof-Check) statt der alten zwei Render-Funktionen.
- **`doLogout` und `handle401`** blenden `#projects-screen` defensive aus, sonst hängt sie nach Logout sichtbar über dem Login.
- **Source-Audit-Tests** in [`zerberus/tests/test_p_ui_5_project_view.py`](zerberus/tests/test_p_ui_5_project_view.py): fünf Klassen, 53 Tests. Slice-basiert, kein Browser/Playwright. [`test_nala_projects_tab.py`](zerberus/tests/test_nala_projects_tab.py) Patch-201-Tests an P-UI-5 angepasst — die alten Tab-Audit-Klassen prüfen jetzt die View-Mechanik; XSS-Test umgesattelt auf `pui5_buildProjectCardHtml` + Search-Query-Escape-Pflicht; `_disable_auto_template`-Fixture nicht mehr autouse, damit HTML-Audit-Tests ohne `config.yaml`-Setup laufen. Pre-existing P-UI-4-Failure `test_settings_umbau::test_mein_ton_nicht_mehr_in_sidebar` (Anchor durch overlay-Entfernung kaputt) bleibt offen — nicht durch P-UI-5 verursacht.

**Was P-UI-5 bewusst NICHT macht:**

- **Detail-View (Anweisungen + Ressourcen + Chat-Liste innerhalb des Projekts).** Spec 13.2 verlangt sie, aber Backend `/nala/projects` liefert nur `id/slug/name/description/updated_at` — keine Anweisungen, keine Ressourcen, keine Chat-Liste-Filterung. Folge-Patch (UI-5b oder eigener Backend-Patch) baut den Detail-Read-Pfad. `handleProjectTap(id)` aktiviert das Projekt und kehrt zur Chat-View zurück (Standard-Flow).
- **`chat_count` pro Projekt anzeigen.** Spec 13.1 nennt "Name + Chat-Anzahl + letzter Zugriff" pro Eintrag. Aktuelle API liefert kein chat_count — wird stattdessen Slug + Datum angezeigt. Backend-Erweiterung wäre Folge-Patch.
- **Pin-State im Backend persistieren.** Pin-Status liegt rein in `localStorage[nala_pinned_projects]` — überlebt Browser-Restart, aber nicht Geräte-Wechsel. Backend-Schema `pinned`-Feld auf Project-Tabelle wäre Folge-Patch.
- **"Neues Projekt"-Anlegen direkt in Nala.** Spec 13.1: "versteckt hinter Pull-Gesture ganz oben". Pull-Gesture ist drin (touchmove > 80 px reveal die Zone), Klick zeigt einen Hint-Alert "Anlegen geht über Hel". Echtes Inline-Anlegen wäre Backend-Endpoint + Form — Folge-Patch.
- **Hamburger ☰ → ✖-Animation bei offener Sidebar.** Bleibt auch in UI-5 ungebaut (gleiche Begründung wie P-UI-4: Spec hat keine Aussage, Mini-Add für späteren Patch).
- **Hel-Splitscreen-Bug** (Schulden-Liste #9) — eigener Folge-Patch innerhalb der Phase 5c.

**Lessons (3):**

1. **Eine View als Geschwister im transformed Container shiftet automatisch mit — kein extra Setup.** Die `#projects-screen`-View nutzt denselben Container-Shift wie `#chat-screen` aus P-UI-4, weil sie Geschwister im `.app-container` ist und der Container-Transform sie als Layout-Child mitnimmt. Kein eigener Sidebar-Open-State-Listener nötig, kein doppelter Transform — die Geometrie kaskadiert. Generalisierbar: **Layout-Modi auf `.app-container`-Ebene gelten für alle Screen-DIVs darunter.** Wenn ein neuer Screen den selben Modi-Pool bekommen soll (Container-Shift, Read-Only-Mode-Ueberlay etc.), reicht es als `.app-container`-Geschwister.

2. **Defensive `typeof === 'function'`-Checks an Cross-Module-Render-Calls vermeiden Init-Order-Probleme.** Die alten `renderNalaProjectsList`/`renderNalaProjectsActive`-Funktionen waren globale Aufrufe in `setActiveProject`/`clearActiveProject` ohne Defensive — wenn die Render-Targets nicht existieren (z.B. weil der User noch nicht zur View navigiert ist oder `handle401` mitten im Login feuert), warf der DOM-Zugriff. P-UI-5 ersetzt das durch `if (typeof renderProjectsView === 'function') { try { renderProjectsView(); } catch (_) {} }` — der Aufruf ist no-op-tolerant, der Cache wird trotzdem aktualisiert. Generalisierbar: **bei jedem Aufruf einer Render-Funktion aus einer Setter/Mutator-Funktion (statt aus einem Init-Hook) Defensive-Checks vorschalten** — das Setter-Pattern wird oft von verschiedenen Code-Pfaden gerufen, und die Render-Targets existieren nicht überall.

3. **`autouse=True`-Fixtures koppeln Source-Audit-Tests an Backend-Setup.** `test_nala_projects_tab.py` hatte einen `_disable_auto_template`-Fixture mit `autouse=True`, der `cfg.get_settings()` ruft — und dabei `config.yaml` lädt, was im Worktree-Setup gitignored ist. Ergebnis: alle 18 reinen HTML/JS-Source-Audit-Tests fielen mit `FileNotFoundError`-ERROR aus, obwohl sie nur den NALA_HTML-String brauchen, nicht die Settings. Lösung: `autouse=True` raus, Fixture wird nur von den Endpoint-Tests explizit angefordert. Generalisierbar: **`autouse=True` nur für Fixtures verwenden, deren Setup garantiert auf jedem Test-Pfad funktioniert** (logging, monkeypatch von Standard-Modulen) — alles, was externe Files/Configs lädt, wird Pflicht-Fixture mit explizitem Param.

**Tests:** 53 neue in [`zerberus/tests/test_p_ui_5_project_view.py`](zerberus/tests/test_p_ui_5_project_view.py) — fünf Klassen: `TestSourceWiringCSS` (7), `TestSourceWiringHTMLLayout` (12), `TestSourceWiringJS` (19), `TestKollisionMitVorgaengernPatches` (12), `TestPUiFiveInlineMarker` (3). Plus 18 angepasste Tests in `test_nala_projects_tab.py` (Tab-Asserts → View-Asserts, XSS-Test auf neuen Card-Renderer + Search-Escape umgesattelt, autouse-Fixture entkoppelt). Alle 240 UI-relevanten Tests grün lokal (test_p_ui_5 53 + test_p_ui_4 29 + test_p_ui_3 26 + test_p_ui_2 20 + test_p_ui_1 16 + test_nala_bubble_layout 15 + test_nala_adapter 14 + test_p203d3_nala_code_render 30 + test_nala_projects_tab 18 HTML-Audits + test_settings_umbau 19 nicht-broken). Pre-existing-Failures: 6 `test_nala_projects_tab::TestNalaProjectsEndpoint`-ERRORS (`config.yaml` fehlt im Worktree, dokumentiert in Memory) + 1 `test_settings_umbau::test_mein_ton_nicht_mehr_in_sidebar` (Anchor durch P-UI-4 overlay-Entfernung kaputt). Nicht durch P-UI-5 verursacht.

**Logging-Tag:** keiner — P-UI-5 ist reines Frontend-CSS+HTML+JS-Patch, kein neuer Server-Code-Pfad.

---

## Frueherer Patch (Referenz, ausserhalb Hot-Window)

**P-UI-4** — Phase 5c Schritt 4: Sidebar Content-Shift statt Overlay (2026-05-07)

Vierter Code-Patch der UI-Redesign-Phase. Die Sidebar oeffnet sich nicht mehr als Overlay ueber dem Chat (Backdrop-Klick = Schliessen), sondern schiebt den Chat-Content per CSS-Transform 280px nach rechts und rutscht selbst von links ein — Mistral-Le-Chat-Vorbild. Der Sidebar-Inhalt ist von oben nach unten neu sortiert: Suchfeld (Chat-Historie durchsuchen, Lupe-Placeholder) ganz oben, dann Aktion-Buttons "➕ Neuer Chat" und "📁 Projekte", dann Chat-Historie (Pinned + Letzte Chats), dann Footer mit Abmelden + Einstellungen. Der Projekte-Button ist neu und ist ein eigener Sidebar-Menuepunkt (DESIGN.md Sektion 12.2: "Projekte raus aus Settings"); uebergangsweise routet er ueber `openProjectsView()` auf den Settings → Projekte-Tab — UI-5 ersetzt diesen Click-Pfad durch eine eigene Project-View, der Button selbst bleibt an der DOM-Position.

**Architektur: CSS-Transform-Shift + DOM-Refactor (Sidebar wird body-Child) + JS-Click-Outside-Listener + Source-Audit-Tests.**

- **CSS-Block `.app-container` erweitert** in [`zerberus/app/routers/nala.py`](zerberus/app/routers/nala.py): bestehende Frame-Definition (max-width 500px, height 100dvh, flex column, position relative, box-shadow) bleibt unangefasst — nur eine zusaetzliche Property `transition: transform 0.3s ease`. Neue Regel `body.pui4-sidebar-open .app-container { transform: translateX(280px); }` triggert den eigentlichen Shift. Die Bestands-CSS-Definition fuer `.sidebar` (`position: fixed; left: -300px; width: 280px; transition: left 0.3s ease`) wird komplett wiederverwendet — die rein-rutsch-Mechanik der Sidebar ist seit Patch 67 stabil.
- **`.overlay`-CSS entfernt** (alter Backdrop-Block plus `.overlay.show`-Regel). Spec sagt "KEIN Overlay/Ueberklappen". Statt Backdrop schliesst die Sidebar via JS bei Click ausserhalb.
- **HTML-Refactor**: das `<div class="sidebar" id="sidebar">`-Markup wurde aus `#chat-screen` ausgehaengt und ist jetzt ein body-Child (Geschwister von `.app-container`, vor `<div id="ee-modal">`). Grund: bei `transform` auf `.app-container` wuerde ein im Container sitzender fixed-positioned Descendant zum containing-block-Kind und mit-shiften — die Sidebar muss aber Viewport-bound bleiben. Der Move ist transparent fuer alle Bestands-Selektoren (`getElementById('sidebar')`/`'archive-search'`/`'pinned-list'`/`'session-list'`-Listener bleiben funktional).
- **HTML-Reihenfolge in der Sidebar** umsortiert (DESIGN.md Sektion 12.2): `archive-search` zog von unter "📋 Letzte Chats" nach OBEN in den `sidebar-header`. `sidebar-actions` enthaelt jetzt zwei Buttons (statt einem): "➕ Neuer Chat" (vorher Text "Neue Session" — auf Spec angeglichen) und neu "📁 Projekte" mit `id="sidebar-projects-btn"`. Pinned-Heading + Pinned-List + "📋 Letzte Chats"-h3 + Session-List bleiben darunter. Footer (Abmelden + Settings, Patch 142/B-013) bleibt sticky-unten unangefasst. Der explizite `close-btn` (✖) wurde entfernt — Spec sagt "Tap ausserhalb oder Hamburger-Icon erneut" als Standard-Pattern.
- **`<div class="overlay" id="overlay" onclick="toggleSidebar()">` HTML-Element entfernt** — passt zum CSS-Block-Entfernen.
- **JS-Refactor `toggleSidebar()`** in derselben Datei: nutzt jetzt `willOpen = !sidebar.classList.contains('open')` und togglet in einem Schritt sowohl `sidebar.classList.toggle('open', willOpen)` als auch `document.body.classList.toggle('pui4-sidebar-open', willOpen)`. Die body-Klasse triggert den `.app-container`-Transform per CSS. Kein `overlay.classList.toggle('show')` mehr.
- **JS-Helper `pui4_handleOutsideClick(event)`** als Top-Level-Function direkt nach `toggleSidebar()`. Logik: bei offener Sidebar pruefen ob `event.target` innerhalb der Sidebar ist (`sidebar.contains(event.target)`) oder Hamburger-Whitelist (`event.target.closest('.hamburger')`); wenn nein → `event.preventDefault()` + `event.stopPropagation()` + `toggleSidebar()`. Registriert mit `document.addEventListener('click', pui4_handleOutsideClick, true)` (capture-phase, dritter Parameter `true`) — feuert vor anderen Listenern (z.B. P-UI-2 capture-Listener auf User-Bubbles), der User wollte die Sidebar schliessen, nicht eine Bubble togglen.
- **JS-Helper `openProjectsView()`** ebenfalls Top-Level. Schliesst die Sidebar (`toggleSidebar()`), oeffnet das Settings-Modal und switcht auf den Projekte-Tab. Defensive `typeof === 'function'`-Checks an beiden Settings-Aufrufen — wenn UI-5 die Funktion umbaut oder umbenennt, knallt nichts.
- **Source-Audit-Tests** in [`zerberus/tests/test_p_ui_4_sidebar_content_shift.py`](zerberus/tests/test_p_ui_4_sidebar_content_shift.py): fuenf Klassen, 29 Tests. Slice-basierte Source-Inspektion (kein Browser/Playwright) per Pattern aus P-UI-3.

**Was P-UI-4 bewusst NICHT macht:**

- **Project-View bauen.** Der `openProjectsView()`-Click oeffnet uebergangsweise Settings → Projekte-Tab. UI-5 ist der eigene Patch-Schritt fuer die Project-View (Suche, Sortierung, Pinned-Sektion, Pull-Gesture, Detail-View). DOM-Position des Buttons bleibt — UI-5 ersetzt nur den Click-Handler.
- **Hamburger-Animation (z.B. ☰ → ✖).** Der Hamburger bleibt visuell "☰" auch bei offener Sidebar. Spec hat dazu keine Aussage, und das Hinzufuegen einer Klasse-basierten Symbol-Animation ist Mini-Add fuer einen spaeteren Patch.
- **Sidebar-Breite fuer Tablet/Desktop anpassen.** `width: 280px` ist Mobile-First. Auf Desktop (Viewport > 800px) bleibt `.app-container` mittig 500px breit; ein 280px-Shift ergibt einen leichten Versatz aus der Mitte. Akzeptables Trade-off, weil Nala primaer Mobile ist und der offene-Sidebar-Zustand temporaer.
- **`particleCanvas` aus `.app-container` rausziehen.** Das Patch-145-Feuerwerk-/Sternen-Canvas (`position: fixed; pointer-events: none; z-index: 9999`) sitzt noch innen — bei Sidebar-Open wird es per Transform mit-geshifted und deckt nicht mehr den vollen Viewport. In Praxis selten sichtbar (Effekt nur bei speziellen Events + temporaere Sidebar-Offen-Phase). Folge-Patch falls auffaellt.
- **Suchfeld-Funktion erweitern.** Der bestehende `archive-search`-Listener (Patch 67-aera) durchsucht weiterhin die Session-Liste. Spec sagt "durchsucht Chat-Historie" — das macht er schon. Folge-Funktionalitaet (z.B. Fulltext durch Bot-Antworten) bleibt UI-9-Settings-Schritt vorbehalten.
- **DESIGN.md-[WERT]-Befuellung.** Sektion 12 (Sidebar / Hamburger-Menue) hat keine offenen `[WERT]`-Platzhalter; Aufbau und Verhalten stehen explizit in den Tabellen 12.1 und 12.2. Spacing/Z-Index/Shadow-Stufen-Befuellung bleibt spaeteren Patches ueberlassen.
- **Hel-Splitscreen-Bug** (Schulden-Liste #9) — eigener Folge-Patch innerhalb der Phase 5c, nicht vermischt mit der Sidebar.

**Lessons (3):**

1. **`transform` auf einem Vorfahren formt einen neuen containing block fuer fixed-positioned Descendants.** Das CSS-Spec-Detail (CSS Transforms Module Level 1, Sektion 6.1) sagt: jeder Vorfahre mit `transform`/`filter`/`perspective` ungleich `none` wird zum containing block fuer all seine fixed-positioned Descendants — nicht mehr der initial containing block (Viewport). Praktische Folge: wenn `.app-container` einen `transform: translateX(280px)` kriegt und die `.sidebar` darin sitzt, shiftet die Sidebar mit dem Container — exakt entgegen der Sidebar-Mechanik (sie soll Viewport-bound bleiben). Loesung: Sidebar-DOM aus dem transformed Vorfahren herausziehen (in P-UI-4 als body-Child platziert). Bestands-Code-Audit zeigte, dass die Sidebar-CSS-Definition (position fixed, left -300px, width 280px) vollstaendig ueber die Bestands-Patch-67-Regel funktioniert — der DOM-Move war die einzige Aenderung, kein neues CSS noetig. **Lesson generalisierbar:** vor jedem `transform`-Edit auf Container-Element die fixed-positioned Descendants identifizieren — wenn welche darunter sind, entweder DOM-Refactor (Descendant raushebeln) oder containing-block-aware-Variante (z.B. `margin` statt `transform`, was kein neues containing block formt). Source-Audit-Test `test_sidebar_steht_ausserhalb_chat_screen` zementiert die DOM-Position als Invariante.
2. **Capture-phase Listener auf `document` fangt Click-Outside ohne ein dediziertes Backdrop-Element.** Das alte `.overlay`-Element war Backdrop UND Click-Trap in einem (`onclick="toggleSidebar()"` direkt auf dem Overlay-DIV). Bei Content-Shift-Mechanik ist kein Overlay mehr da — wo fange ich den Click-Outside? `document.addEventListener('click', handler, true)` (capture-phase, drittes Argument `true`) feuert auf jeden Click vor allen anderen Listenern, inklusive Bubble-phase auf den shifted Children. Mit `event.preventDefault() + event.stopPropagation()` konsumiert der Listener den Click — keine versehentliche Aktion auf dem shifted Content. Hamburger-Whitelist via `event.target.closest('.hamburger')` (sonst toggelt der Hamburger-onclick gleichzeitig ZUSAETZLICH, Bilanz wäre no-op). **Lesson generalisierbar:** wenn ein Backdrop-Pattern abgeloest wird (Click ausserhalb des Modals), ist `document`-level capture-phase Listener mit Whitelist + preventDefault + stopPropagation der saubere Ersatz — funktioniert auf jedem DOM-Layout und braucht kein dediziertes Click-Trap-Element.
3. **DOM-Move ist transparenter Refactor, wenn die Bestands-Selektoren ID-basiert sind.** Der HTML-Move der Sidebar von `#chat-screen`-innen nach body-Child wuerde bei klassen-basierten Selektoren (z.B. `#chat-screen .sidebar`) brechen. ABER: alle Bestands-Listener nutzen `getElementById('sidebar')`, `getElementById('archive-search')`, `getElementById('pinned-list')`, `getElementById('session-list')` — ID-Selektoren sind position-unabhaengig. Plus die CSS-Definition fuer `.sidebar`/`.sidebar-actions`/`.sidebar-footer` ist via Klassen-Selektor ohne Ancestor-Pfad — funktioniert auch nach dem Move. Vor dem Move via Audit verifiziert dass kein `#chat-screen .sidebar`/`#chat-screen #sidebar` etc. im Source vorkommt. **Lesson generalisierbar:** bei DOM-Move-Refactors immer zuerst per Grep auf Ancestor-relative Selektoren pruefen (z.B. `#chat-screen .sidebar` oder `parent > .sidebar` oder `closest('.app-container')`). ID-basierte Selektoren sind transparent fuer DOM-Position, klassen-basierte sind es nur wenn der Selektor selbst keinen Ancestor-Pfad enthaelt. Falls ja: entweder Selektor refactoren oder DOM-Move verwerfen.

**Tests:** 29 neue in [`zerberus/tests/test_p_ui_4_sidebar_content_shift.py`](zerberus/tests/test_p_ui_4_sidebar_content_shift.py) — fuenf Klassen. Alle 150 Nala-relevanten Tests gruen lokal (test_p_ui_4 29 + test_p_ui_3 26 + test_p_ui_2 20 + test_p_ui_1 16 + test_nala_bubble_layout 15 + test_nala_adapter 14 + test_p203d3_nala_code_render 30). Volle Suite-Erwartung: **2813 passed lokal erwartet** (P-UI-3-Baseline 2784 + 29 P-UI-4-Tests). Worktree-Lauf zeigt pre-existing Backend-Failures unveraendert — bekannter Worktree-Setup-Drift, nicht durch P-UI-4 verursacht (Patch beruehrt nur Frontend).

**Logging-Tag:** keiner — P-UI-4 ist reines Frontend-CSS+HTML+JS-Patch, kein neuer Server-Code-Pfad.

---

## Frueherer Patch (Referenz, ausserhalb Hot-Window)

**P-UI-3** — Phase 5c Schritt 3: Eingabefeld Expand/Collapse-Verhalten (2026-05-07)

Dritter Code-Patch der UI-Redesign-Phase. Das Eingabefeld unten (sticky-Bottom) kollabiert bei verlorenem Fokus auf eine Zeile (~44 px) mit Ellipsis-Truncation, sodass nur die letzte eingegebene Zeile sichtbar bleibt — User weiss wo er zuletzt war. Bei Fokus expandiert es sanft auf `33dvh` (~1/3 Bildschirm), was zusammen mit der Mobile-Tastatur (~1/3) und dem sichtbaren Chat-Bereich (~1/3) die gewünschte Drittelung ergibt. Sende-Button rechts unten in der Bubble-Anordnung ist nur aktiv, wenn das Eingabefeld nicht leer (oder reines Whitespace) ist — Klasse `.send-btn.pui3-empty` deaktiviert Click und Hover-Effekte via `pointer-events: none` und `opacity: 0.4`.

**Architektur: CSS-only fuer die Hoehe (kein inline-style.height mehr) plus minimaler JS-Helper plus Source-Audit-Tests.**

- **CSS-Block `#text-input` umgebaut** in [`zerberus/app/routers/nala.py`](zerberus/app/routers/nala.py): default `min-height: 44px; max-height: 44px; height: 44px;` plus `white-space: nowrap; text-overflow: ellipsis; overflow: hidden;` fuer eine sauber abgeschnittene Single-Line-Anzeige. Bei `:focus` greift ein eigener Selektor mit `min-height: 33dvh; max-height: 33dvh; height: 33dvh; white-space: pre-wrap; overflow-y: auto;` — mehrzeilige Eingabe + scrollbar. Sanfte Transition `border 0.2s, box-shadow 0.2s, height 0.3s ease, min-height 0.3s ease, max-height 0.3s ease` macht den 44px↔33dvh-Pop fluessig. Mobile-Landscape-Media-Query (`max-height: 500px`) hat das alte `min-height: 40px; max-height: 96px` fuer #text-input entfernt — sonst wuerde es das 33dvh-Verhalten ueberschreiben. Padding `7px 14px` bleibt fuer Landscape erhalten.
- **CSS `.send-btn.pui3-empty`** direkt nach dem `:active`-Block: `opacity: 0.4; cursor: default; pointer-events: none; box-shadow: none;` plus expliziter `:hover/:active`-Selektor der `transform: none; box-shadow: none; background: var(--color-gold);` zementiert — der Default-:hover-Effekt (gold-dark + scale + Shadow) wird im leeren Zustand komplett unterdrueckt. Orthogonal zu `lockInput()/unlockInput()` waehrend des Sendens (das setzt `sendBtn.disabled = true` + inline-`opacity: 0.5` und ueberschreibt damit visuell die Klasse).
- **JS-Helper `pui3_updateSendBtnState()`** als Top-Level-Function vor den Textarea-Listenern. Logik: wenn `textInput.value.trim().length > 0` → `classList.remove('pui3-empty')`, sonst → `classList.add('pui3-empty')`. Aufrufstellen: `input`-Listener (bei jedem Tastendruck), `blur`-Listener, am Ende von `sendMessage` nach `textInput.value = ''`, in `unlockInput()` (nach Send-Lock-Reset), in `editMessage()` (nach Setzen des Texts), in `fullscreenClose(true)` (nach Uebernahme des Fullscreen-Texts), und einmal initial nach dem `keydown`-Listener-Block.
- **Blur-Handler scrollt zur letzten Zeile**: bei nicht-leerem Inhalt wird `textInput.scrollTop = textInput.scrollHeight` gesetzt — bei mehrzeiligem Inhalt im Collapsed-44px-Modus schiebt das den Scroll-Cursor nach unten, sodass die letzte Zeile im Ellipsis-Modus sichtbar wird. Wrapped in try/catch fuer Browser-Edge-Cases.
- **Alte Patch-67-autoresize-Logik komplett entfernt**: `Math.min(textInput.scrollHeight, 140)` (vom input-Listener), `Math.min(Math.max(scrollHeight, 96), 140)` (vom focus-Listener), und alle drei `textInput.style.height = ...`-Inline-Setter (in input/focus/editMessage/fullscreenClose). CSS-Selektor `#text-input:focus { height: 33dvh }` ist die alleinige Hoehen-Steuerung — Inline-Styles wuerden CSS-Transitions ueberschreiben und sind deshalb komplett raus.
- **Source-Audit-Tests** in [`zerberus/tests/test_p_ui_3_textarea_collapse.py`](zerberus/tests/test_p_ui_3_textarea_collapse.py): vier Klassen, 26 Tests. `TestSourceWiringCSS` (8) prueft default 44px-Cap, Truncation-Stack (`white-space: nowrap` + `text-overflow: ellipsis` + `overflow: hidden`), Smooth-Transition mit drei Hoehen-Properties, Focus-Expand auf 33dvh, Focus-pre-wrap + overflow-y: auto, Send-Btn-Disabled-Block (opacity + pointer-events: none), Send-Btn-:hover/:active-Override im Empty-Zustand, Mobile-Landscape-Media-Query ohne min-height/max-height-Override fuer #text-input. `TestSourceWiringJS` (10) prueft Helper-Existenz, Trim-Length-Check + Klassen-Toggle, input/blur-Listener-Aufrufe des Helpers, scrollTop=scrollHeight im Blur-Handler, Abwesenheit der alten autoresize-Logik (`Math.min(scrollHeight, 140)` und `textInput.style.height` global), Helper-Aufrufe in sendMessage/unlockInput/editMessage/fullscreenClose, Initialer Aufruf nach keydown-Listener. `TestKollisionMitVorgaengernPatches` (5) zementiert dass P-UI-1, P-UI-2, P124, P139 und der lockInput/unlockInput-Pfad intakt sind. `TestPUiThreeInlineMarker` (3) verlangt dass `P-UI-3 (Phase 5c, 2026-05-07)` mehrfach im Source vorkommt (CSS + JS).

**Was P-UI-3 bewusst NICHT macht:**

- **Send-Button ABSOLUT ins Eingabefeld positionieren.** Mock-Spec sagt "rechts unten im Eingabefeld", aber `<textarea>` ist ein Replaced Element — Buttons koennen nicht inline drinliegen. Aktuelle Anordnung (Sende-Button als Sibling rechts neben der Textarea, `align-items: flex-end` haelt ihn am unteren Bubble-Rand) erfuellt die Spec visuell, ohne die DOM-Struktur (Textarea + Expand-Btn + Send-Btn + Mic-Btn + Prosody-Indicator als 5 Siblings) zu sprengen. Falls Chris ein echtes Inline-Send-Btn-im-Feld will, ist das ein Folge-Patch mit `position: relative`-Wrapper.
- **Settings-Toggle fuer Auto-Collapse-Verhalten.** Spec ist eindeutig — Default ist 44px collapsed bei Blur. Falls User das Verhalten abschalten wollen wuerden, kommt das in UI-9 (Skalierung Settings-Bereich).
- **Animation der Truncation.** Wenn der User vom Collapsed-Modus zum Expanded-Modus wechselt (Fokus), animiert die CSS-Transition die Hoehe von 44px → 33dvh. Die `white-space`-Transition (nowrap → pre-wrap) ist NICHT animierbar, das ist ein harter Switch — im Result kein visueller Bruch, aber technisch kein "smooth fade".
- **Ellipsis-Truncation bei Newline-Inhalten.** Bei `white-space: nowrap` interpretiert die Textarea Newlines im Display teilweise abhaengig vom Browser. Pragmatisch loest der `scrollTop = scrollHeight`-Trick das Problem, indem die letzte Zeile in den sichtbaren Bereich gescrollt wird. Falls bestimmte Browser-Engines das anders rendern, wird das im manuellen Test (Chris/Jojo) auffallen und ist ein Folge-Patch.
- **Animation des Send-Button-Aktivierens.** Beim Tippen des ersten Zeichens "snappt" der Send-Btn von Empty (opacity 0.4) auf Aktiv (opacity 1) ohne CSS-Transition. Sanfte Opacity-Animation waere nice-to-have, aber die `:hover/:active`-Side-Effects auf der Empty-Klasse bracuhten Override-Selektor — eine zusaetzliche `transition: opacity 0.2s ease` auf `.send-btn` waere ein Mini-Add fuer einen spaeteren Patch.
- **DESIGN.md-[WERT]-Befuellung.** Sektion 4.3 (Chat-Eingabefeld) hat keine offenen `[WERT]`-Platzhalter, alle Werte explizit (44px, 33dvh, smooth, auto, ja). Spacing/Z-Index/Shadow-Stufen-Befuellung bleibt spaeteren Patches ueberlassen.

**Lessons (3):**

1. **CSS-only fuer Hoehe ist stabiler als JS-getriebene autoresize-Logik.** Die Patch-67-Logik (`Math.min(scrollHeight, 140)` im input/focus-Listener plus `style.height = '...'`-Inline-Setter) hatte ueber die Jahre vier Aufruf-Stellen (input, focus, editMessage, fullscreenClose) — jede setzte Inline-Height direkt und uebersteuerte damit alle CSS-Transitions. P-UI-3 zentralisiert die Hoehe in CSS-Selektoren (`#text-input` default + `:focus`) und entfernt alle Inline-Setter. Die CSS-Transition feuert zuverlaessig bei jedem Fokus-Wechsel. **Lesson generalisierbar:** wenn ein UI-Element zwischen zwei Zustaenden wechselt und beide CSS-driven beschrieben werden koennen (Default + Pseudo-Klasse / State-Klasse), ist CSS-only der stabilste Pfad. JS-Inline-Style-Setter sind versteckte Schalter, die jeder spaetere Refactor-Schritt aus Versehen rausreissen oder unbeabsichtigt brechen kann. Der Source-Audit-Test `test_keine_alte_autoresize_logik` zementiert das: kein `textInput.style.height`, kein `Math.min(scrollHeight, 140)`.
2. **Klassen-basiertes Disabled-Verhalten ist orthogonal zu native `disabled`.** Send-Button hat zwei voneinander unabhaengige Disabled-Pfade: (1) `lockInput()` setzt `sendBtn.disabled = true` und inline `opacity: 0.5` waehrend eines aktiven Sendens (Native-Browser-Disabled, blockiert Click + Tab-Focus); (2) `pui3_updateSendBtnState()` togglet `.send-btn.pui3-empty` je nach Eingabefeld-Inhalt (CSS-Pseudo-Disabled via `pointer-events: none` + `opacity: 0.4`). Beide koennen gleichzeitig aktiv sein (Senden laeuft + Eingabefeld leer), und das ist OK — der Visual ist konsistent. Aber: nach `unlockInput()` muss der Helper aufgerufen werden, sonst bleibt die Klasse falsch. **Lesson generalisierbar:** wenn zwei Disabled-Mechanismen am selben Button koexistieren (Lock waehrend einer Operation + State-Validierung des Inputs), beide explizit synchronisieren — nicht annehmen dass einer den anderen "von alleine" trifft. Die Source-Audit-Tests `test_unlock_input_ruft_helper` und `test_lock_unlock_input_pfad_intakt` zementieren das.
3. **`scrollTop = scrollHeight` macht im 44px-Collapsed-Textarea die letzte Zeile sichtbar.** Spec verlangte "Zeigt die letzte eingegebene Zeile" — Browser-Textarea scrollt ohne Hilfe nicht zur letzten Zeile, wenn der Cursor woanders steht. Im Blur-Handler reicht ein einzelnes `textInput.scrollTop = textInput.scrollHeight`, um den Scroll-Container ans Ende zu setzen. Mit `white-space: nowrap; text-overflow: ellipsis; overflow: hidden` wird der Rest des Inhalts sauber abgeschnitten — User sieht die letzte Zeile als Preview-Hinweis. **Lesson generalisierbar:** Browser-`<textarea>`-Elemente respektieren `scrollTop` auch wenn `overflow: hidden` ist — der Scroll-Position-Wert wird intern gespeichert und bei `overflow: visible/scroll`-Wechsel wieder angewendet. Pragmatischer Trick um Truncation-Verhalten an Position-Praeferenzen anzupassen.

**Tests:** 26 neue in [`zerberus/tests/test_p_ui_3_textarea_collapse.py`](zerberus/tests/test_p_ui_3_textarea_collapse.py) — vier Klassen. Alle 121 Nala-relevanten Tests gruen lokal (test_p_ui_3 26 + test_p_ui_2 20 + test_p_ui_1 16 + test_nala_bubble_layout 15 + test_nala_adapter 14 + test_p203d3_nala_code_render 30). Volle Suite-Erwartung: **2784 passed lokal erwartet** (P-UI-2-Baseline 2758 + 26 P-UI-3-Tests). Worktree-Lauf zeigt pre-existing Backend-Failures (`test_projects_*` config.yaml-Drift, `test_test_profile_filter`, `test_patch185_runtime_info`, `test_rag_dual_switch`) unveraendert — bekannter Worktree-Setup-Drift, nicht durch P-UI-3 verursacht (Patch berührt nur Frontend).

**Logging-Tag:** keiner — P-UI-3 ist reines Frontend-CSS+JS-Patch, kein neuer Server-Code-Pfad.

---

## Vorheriger Patch

**Patch 213-pre-4** — FAISS-Reindex-Backup-Garbage-Collection als wöchentlicher Cron-Job (HANDOVER-Schulden-Liste #8 geschlossen) (2026-05-07, parallel zu P-UI-1)

Folge-Patch zu P213-pre-3. Das FAISS-BAK-MUSTER aus P213-pre-3 erstellt vor jedem Reindex eine `.pre_reindex_<UTC-Timestamp>.bak`-Kopie des alten Index-Files (`shutil.copy2`, in `.gitignore`). Bei häufigem Reindex sammeln sich diese Dateien neben den Live-Index-Files in `data/vectors/` an — niemand räumt sie auf, sie sind aber das einzige Rollback-Asset bei einem korrupten Live-Index. P213-pre-4 baut den GC-Cron-Job, der wöchentlich Sonntag 03:00 Europe/Berlin alle `.pre_reindex_*.bak`-Dateien älter als 7 Tage löscht.

**Architektur: Pure-Function-Schicht plus direkter APScheduler-Hook (kein DB-Task).**

- **Pure-Function-Schicht** in [`zerberus/utils/backup_gc.py`](zerberus/utils/backup_gc.py) (neue Datei): drei Funktionen ohne APScheduler/DB-Imports — `find_stale_backups(directory, *, max_age_days, glob_pattern, now_seconds)` (Glob + Mtime-Cutoff, sortiert ältester zuerst), `delete_backup_files(paths)` (best-effort, sammelt Errors statt zu raisen), `run_backup_gc(directory, *, max_age_days, glob_pattern)` (Convenience für den Scheduler-Job, niemals raisen — Cron-Job-Crash darf nicht das System killen). Konstanten `DEFAULT_BACKUP_GLOB="*.pre_reindex_*.bak"` und `DEFAULT_MAX_AGE_DAYS=7` als Source-of-Truth.
- **Direkter APScheduler-Hook** in [`zerberus/main.py`](zerberus/main.py) im Lifespan-Manager nach `initialize_task_scheduler`: `_scheduler.add_job(run_backup_gc, trigger="cron", day_of_week="sun", hour=3, minute=0, kwargs={"directory": <vector_db_path>, "max_age_days": 7}, id="backup-gc-pre-reindex", replace_existing=True, misfire_grace_time=3600)`. Analog zum P57 Overnight-Job — kein DB-Task, weil System-Maintenance (User soll's nicht versehentlich aus der Hel-UI disablen können). Logging-Tag `[BACKUP-GC]` mit Vor/Nach-Counts und Bytes-Freed.
- **`vector_db_path`-Auflösung** aus `settings.modules["rag"]["vector_db_path"]` mit Default `./data/vectors` — gleicher Pfad wie `_resolve_paths` in `router.py` (P213-pre-2 dual-aware). Kein neuer Settings-Eintrag nötig.

**Was P213-pre-4 bewusst NICHT macht:**

- **Keine User-konfigurierbare Aufbewahrungsfrist.** 7 Tage ist Konstante in `DEFAULT_MAX_AGE_DAYS`. Ein Settings-Eintrag wäre Feature-Add ohne klaren ROI für den Bugfix. Tuning kann später kommen wenn die Bytes-Freed-Logs zeigen dass 7 Tage zu kurz/lang sind.
- **Keine Audit-DB-Tabelle.** Die `[BACKUP-GC]`-Log-Lines im Server-Log sind ausreichend für ein System-Maintenance-Job. Ein DB-Audit wäre Overengineering — `scheduled_task_runs` ist für User-konfigurierte Tasks.
- **Keine Hel-UI-Anzeige.** Konsequent zur Architektur-Entscheidung "System-Maintenance, nicht Projekt-Task". Der Job läuft im Hintergrund.
- **Kein `kind=shell` mit `find -mtime`.** Erste Versuchung war den HANDOVER-Vorschlag wörtlich zu nehmen (`kind=shell` mit `find data/vectors -mtime +7 -delete`). Verworfen weil `_execute_kind_shell` `project_id` braucht (P214-pre-3-Workspace-Mount-Defense), und ein globaler Maintenance-Job hat keinen Projekt-Workspace. Plus Pure-Python ist testbar ohne Sandbox-Dependency.
- **Keine Glob-Recursion.** Backups liegen per Konvention direkt im `vector_db_path` neben dem Live-Index (siehe `_resolve_paths` in `router.py`). Ein `**`-Glob würde nach Tests, Backups in Subverzeichnissen etc. suchen — nicht der Scope.

**Lessons (3):**

1. **System-Maintenance-Cron-Jobs gehören NICHT in die User-konfigurierbare Task-Tabelle.** Lieber direkt am APScheduler analog dem P57-Overnight-Job — User-konfigurierbare Tasks (DB + UI + Disable-für-User) und System-Maintenance-Tasks (Code + immer-an + nicht User-sichtbar) sind zwei Ebenen. Spart auch Task-Kind-Wahl und Default-Seed-Mechanismus.
2. **Pure-Function-Schicht plus Cron-Hook als minimaler Zwei-File-Patch.** Kein DB-Setup, kein neuer Task-Kind, keine Whitelist-Erweiterung. Source-Audit-Tests prüfen die Verdrahtung in `main.py`. 31 Tests, ein einziges DB-Setup-File nicht angefasst.
3. **`os.utime`+`now_seconds`-Override macht GC-Tests deterministisch ohne `time.sleep`.** Anti-Pattern: 8 Sekunden warten, dann prüfen dass eine 7-Sekunden-Cutoff-Regel greift (langsam + flaky). Pro-Pattern: Pure-Function nimmt optionalen `now_seconds`-Parameter, Test setzt mtime via `os.utime` + ruft mit gefakter Now-Zeit auf. Auch unter Windows portabel.

**Tests:** 31 neue in [`test_p213_pre_4_backup_gc.py`](zerberus/tests/test_p213_pre_4_backup_gc.py) — fünf Klassen: `TestFindStaleBackups` (12), `TestDeleteBackupFiles` (5), `TestRunBackupGC` (6), `TestSourceAuditMainPyHook` (5 — Import in main.py + Job-ID + add_job-Aufruf + Cron-Trigger + P213-pre-4-Tag), `TestSmoke` (3). Alle 31 lokal grün. P214-pre-3-Tests + P213-pre-3-Tests bleiben grün (57/57 zusammen verifiziert) — keine Regression.

**Volle Unit-Suite:** **2722 passed lokal erwartet** (+31 P213-pre-4 auf 2691-Baseline). Phase 5a bleibt VOLLSTÄNDIG ABGESCHLOSSEN — P213-pre-4 ist Folge-Bugfix außerhalb der Phase-5a-Ziele, schließt aber HANDOVER-Schulden-Liste-Position #8 (vorher als "niedrigster verbleibender Aufwand" markiert).

**UI-Bug-Notiz mit-committet:** Diese Session hat zusätzlich [`UI_BUG_HEL_SPLITSCREEN.md`](UI_BUG_HEL_SPLITSCREEN.md) erstellt — ein Hel-Splitscreen-Layout-Bug den Chris parallel gemeldet hat. Nicht angefasst, weil eine Parallel-Session am UI-Redesign arbeitet. Datei dient als Persistenz-Anker für die UI-Session, der HANDOVER-Overwrites überlebt.

---

## Vorheriger Patch

**Patch 213-pre-3** — Transaktional-atomarer Reindex-Endpoint (HANDOVER-Schulden-Liste #2 geschlossen) (2026-05-07)

Die HANDOVER-Schulden-Liste-Position #2 aus der P213-pre-Saga: `POST /hel/admin/rag/reindex` crashte mit HTTP 500 wenn ein Encoder-Fehler mitten im Re-Build auftrat — UND lies dabei den FAISS-Index leer auf Disk zurück (Datenverlust). Das Anti-Pattern: erst `_reset_sync(settings)` (Index + Metadata leeren + persistieren), DANN der Encoding-Loop. Wenn ein Chunk crashte (GPU OOM, Modell-Lade-Fehler, Embedder-Dim-Switch), blieb der gerade geleerte Index leer auf Disk. Jeder weitere Server-Start lud diesen leeren Index — die Userdaten waren weg.

**Architektur: Pure-Function-Helper plus FAISS-BAK-MUSTER plus atomarer Swap.**

- **Neuer Helper `_rebuild_index_atomic(chunks, settings)`** in [`zerberus/modules/rag/router.py`](zerberus/modules/rag/router.py): baut einen neuen FAISS-Index in einer LOKALEN Variable, encoded ALLE Chunks vollständig durch BEVOR der globale Zustand angefasst wird. Erste Encode-Dim definiert den Target-Dim für den ganzen Rebuild — wenn ein späterer Chunk eine andere Dim liefert (Embedder switcht mid-rebuild), wird mit `RuntimeError` ("Embedder-Dim-Inkonsistenz") abgebrochen statt einen inkonsistenten Index zu erzeugen.
- **FAISS-BAK-MUSTER** (per CLAUDE_ZERBERUS.md DESTRUKTIV-OPS-Regel): vor dem Disk-Swap wird das alte Index-File nach `<file>.pre_reindex_<UTC-Timestamp>.bak` kopiert (via `shutil.copy2`). Backup-Erstellung ist bedingt — frischer Index ohne Disk-File braucht kein Backup, dann ist `backup_path=None` im Result.
- **Atomarer Swap unter `_init_lock`**: erst NACH erfolgreicher Encode-Loop und Backup-Erstellung wird `_index = new_index` und `_metadata = new_metadata` gesetzt + auf Disk persistiert. Bei jedem Encoder-Fehler oder Dim-Inkonsistenz mitten im Rebuild bleibt der globale Zustand komplett unangetastet — Aufrufer (der Endpoint) kann sich darauf verlassen, dass `_index`/`_metadata` weiterhin die Vor-Reindex-Daten halten.
- **Endpoint-Anpassung in [`zerberus/app/routers/hel.py`](zerberus/app/routers/hel.py)**: `rag_reindex()` ruft jetzt `_rebuild_index_atomic` via `asyncio.to_thread` statt erst `_reset_sync` und dann eine `_encode`+`_add_to_index`-Loop. Bei Exception aus dem Helper landet ein klar formuliertes `HTTPException(500, "Reindex fehlgeschlagen — alter Index wurde NICHT überschrieben: <e>")` beim User. Logging-Tag `[RAG-219-pre-3]` für Backup-Pfad und Erfolgs-Meldung.

**Was P213-pre-3 bewusst NICHT macht:**

- **Inkrementellen Reindex.** Der Helper baut den Index komplett neu auf — bei großen Korpussen (10k+ Chunks) ist das langsam. Ein inkrementeller Pfad ("nur die geänderten Chunks neu encoden") wäre ein Feature-Patch mit eigenem Bewertungs-Aufwand. Bugfix-Scope hält sich an "atomar oder gar nicht".
- **Backup-Garbage-Collection.** Die `.pre_reindex_<UTC>.bak`-Dateien sammeln sich an, wenn man häufig reindiziert. Sie sind in `.gitignore` (FAISS-BAK-MUSTER), aber jemand muss sie manuell aufräumen. Folge-Patch: einfacher Cron-Job (P214-pre-3 Pattern: `kind=shell`) der `.bak`-Dateien älter als 7 Tage löscht.
- **EN-Index-Rebuild.** Der globale `_add_to_index` schreibt aktuell nur nach DE (siehe Pre-Patch-Doc-Comment). EN-Index ist read-only, kommt aus separater Sync-Tool-Welle. Wenn EN je in den Add-Pfad kommt, kann `_rebuild_index_atomic` 1:1 auf EN gespiegelt werden — Helper-Signatur ist generisch genug.
- **Konkurrente Add/Search während Reindex blockieren.** Reindex ist Admin-Operation, erwartet ruhige Last. Konkurrente `_add_to_index`-Calls während des Encoding-Loops würden mit dem Swap überschrieben (gleiches Risiko wie in der Vor-P213-pre-3-Variante). Doc-Comment dokumentiert das explizit.
- **Hel-UI-Confirmation-Dialog.** Der Reindex-Button löst direkt aus. Bei großen Korpussen wäre eine "Bist du sicher? — alte Backup wird unter X.bak gesichert" Card sinnvoll, aber Feature-Add ohne klaren ROI für den Bugfix.

**Lessons (3):**

1. **Reset-vor-Rebuild ist immer ein Datenverlust-Risiko — egal wie kurz der Spalt ist.** Das alte Pattern (`_reset_sync` → encoding loop) erschien harmlos: "wir leeren den Index, dann füllen wir ihn neu". Aber jeder Encoder-Crash zwischen den beiden Schritten leert den persistierten Index. Lesson generalisierbar: bei jeder "alten Zustand löschen → neuen aufbauen"-Operation auf persistenten Stores muss der neue Zustand VORHER vollständig in einer separaten Variable existieren, dann atomar geswappt werden. Das gilt für FAISS-Indices, Datenbank-Migrationen, Config-Files, Workspace-Snapshots — überall wo "Datenverlust beim Crash mitten in Schritt 2" eine Rolle spielt.
2. **FAISS-BAK-MUSTER auch für planmäßige Operationen, nicht nur für Recovery.** P218-pre nutzte das Pattern bei Dim-Mismatch-Recovery (impliziter Datenverlust), P213-pre-3 nutzt es bei jedem geplanten Reindex (zur Sicherheit auch wenn alles gut geht). Lesson: das Backup zahlt sich aus bei dem 1% der Fälle, wo der Swap fehlschlägt — und in den 99% normalen Läufen kostet es nur eine `shutil.copy2`. Generalisierbar: bei jedem Pfad der ein FAISS-Index-File überschreibt, das Pre-Image nach `<file>.<grund>_<UTC>.bak` sichern. Coda räumt die alten `.bak`-Dateien später auf, das ist eine eigene Schuldenposition.
3. **Source-Audit-Tests greifen auch hier — Anti-Pattern-Check als Defense-in-Depth.** `test_endpoint_no_longer_calls_reset_or_add_to_index` prüft den Endpoint-Source darauf, dass das alte Anti-Pattern (`_reset_sync(...)` direkt vor einem `_add_to_index(...)`-Loop) NICHT mehr vorkommt. Wenn jemand in einem zukünftigen Refactor versehentlich das alte Pattern wieder einführt (Copy-Paste, Merge-Konflikt), schlägt der Test sofort an. Lesson: Source-Audit-Tests sind nicht nur für "diese Funktion existiert" gut, sondern auch für "dieses Anti-Pattern kommt nicht mehr vor" — beides sind kostengünstige Backstops gegen Regression.

**Tests:** 14 neue in [`zerberus/tests/test_p213_pre_3_reindex_atomic.py`](zerberus/tests/test_p213_pre_3_reindex_atomic.py). Sechs Test-Klassen:

- `TestRebuildAtomicHappyPath` (2) — Drei Chunks → globaler State swappt sauber, Save-Funktionen einmalig aufgerufen, `reindexed`-Count stimmt.
- `TestRebuildAtomicEmptyChunks` (1) — Edge-Case: leere Liste = no-op, kein Encode, kein Save, globaler State unverändert.
- `TestRebuildAtomicEncoderFailure` (2) — Erste Encode crasht / dritte Encode crasht: kein Save aufgerufen, globaler State (Index + Metadata) zeigt noch auf die ursprünglichen Objekte (nicht nur gleiche Werte — `is`-Identität).
- `TestRebuildAtomicDimInconsistency` (1) — Encoder switcht mid-rebuild Dim 768→1024: `RuntimeError("Dim-Inkonsistenz")`, globaler State unverändert.
- `TestRebuildAtomicBackupCreation` (2) — Existierendes Index-File wird vor Swap nach `.pre_reindex_<TS>.bak` kopiert (Inhalt geprüft via `read_bytes()`); fehlendes Index-File → `backup_path=None`.
- `TestSourceWiring` (6) — Defense-in-Depth: Helper-Funktion definiert, FAISS-BAK-MUSTER im Source, `_init_lock` für Swap, Endpoint importiert Helper, alte Anti-Patterns (`_reset_sync(`/`_add_to_index(` als Calls) nicht mehr im Endpoint, HTTPException(500) mit Klartext "alter Index ... NICHT überschrieben".

Alle 14 lokal grün. Volle Unit-Suite-Erwartung: **2691 passed lokal erwartet** (+14 P213-pre-3 auf 2677-Baseline = P219-pre + P218-pre + P216).

**Logging-Tag:** `[RAG-219-pre-3]` mit Sub-Events `Pre-reindex backup erstellt: <path> (Source: ..., N neue Chunks bereit)`, `Reindex-Swap erfolgreich: N Chunks im neuen Index`, `Reindex fehlgeschlagen, alter Index bleibt unveraendert: <error>`. Worker-Protection-konform: kein User-Content im Log, nur Pfad-Metadaten plus Counts.

---

## Vorheriger Patch

**Patch 219-pre** — Sammel-Patch: Coda-Autonomie + Supervisor-Bug-Sammelstelle + "quasi"-Bugfix (2026-05-07)

Vier Punkte aus einem Sammel-Auftrag von Chris im Supervisor-Fenster — drei Prozess-Regeln und ein semantischer Bugfix.

**Architektur: drei Doku-Edits + zwei Code-Edits + ein neues Test-File.**

- **Coda-Autonomie schärfen** in [`CLAUDE_ZERBERUS.md`](CLAUDE_ZERBERUS.md) im Marathon-Workflow-Abschnitt: Bei Session-Start liest Coda HANDOVER.md, nimmt die Empfehlung am Ende des "Nächster Schritt"-Abschnitts als Default und legt los — keine "was soll ich machen — A, B oder C?"-Rückfrage. Nur wenn Chris in der Eröffnungs-Message explizit eine andere Aufgabe reingibt, wird abgewichen. Plus: HANDOVER selbst formuliert die Empfehlung jetzt aktiv ("Nächste Session startet mit X — Begründung. Falls Chris was anderes will, überschreibt er.") statt als offene Frage. Verweis auf die bestehende FR-AUTONOME-PRIORITÄT-Regel oben in derselben Datei.
- **Supervisor-Bug-Sammelstelle** in [`SUPERVISOR_ZERBERUS.md`](SUPERVISOR_ZERBERUS.md) als neue Sektion vor "Don'ts für Supervisor". Wenn Chris im Supervisor-Fenster Bugs/Issues diktiert ohne explizit "fixen"/"angehen"/"los"/"pack zusammen" zu sagen, sammelt der Supervisor still — kein Patch-Prompt, kein Aktionismus. Erst auf explizite Trigger-Phrase wird ein Coda-Auftrag gebaut. Hintergrund: Chris denkt laut beim Sammeln; jeder verfrüht-gestartete Patch unterbricht die Sammelphase.
- **Supervisor-Bug-Sammelstand-Anzeige** ebenfalls in [`SUPERVISOR_ZERBERUS.md`](SUPERVISOR_ZERBERUS.md) als Teil der neuen Sektion. Am Ende jeder Antwort listet der Supervisor offene Bugs als nummerierte Liste auf (Format-Vorschlag im Doku-Block). Leere Liste → nichts anzeigen. Funktion: visueller Druck — wenn die Liste lang wird, sieht Chris das automatisch und gibt von sich aus den Auftrag, sie abzuarbeiten. Plus: alle paar Prompts oder vor erkennbarem Kontextfenster-Ende erinnert der Supervisor an die Sammlung (≥3 Bugs Trigger).
- **Bugfix "quasi" aus Filler-Word-Liste** an zwei Stellen: (1) [`whisper_cleaner.json`](whisper_cleaner.json) — Filler-Regex `(?i)\b(eigentlich|irgendwie|quasi|übrigens|prinzipiell|im endeffekt)\b` → `(?i)\b(eigentlich|irgendwie|übrigens|prinzipiell|im endeffekt)\b`. (2) [`zerberus/utils/prompt_compressor.py`](zerberus/utils/prompt_compressor.py) — `_STOPWORDS`-Set ohne "quasi", mit Inline-Kommentar warum. Begründung: "quasi" ist semantischer Qualifier ("annähernd / so etwas wie") und kein Füllwort. "Quasi Reasoning" ≠ "Reasoning" — das Entfernen verändert die Bedeutung. Andere Filler bleiben drin.

**Was P219-pre bewusst NICHT macht:**

- **Andere semantische Qualifier prüfen** ("eigentlich", "irgendwie", "prinzipiell"). Chris hat nur "quasi" gemeldet; Vorgreifen wäre Scope-Sprengung. Falls weitere Wörter durch die Bug-Sammelstelle reinkommen, werden sie in einem späteren Patch behandelt.
- **Mehrsprachig auf Englisch erweitern.** Die englische Filler-Regex (`uh|um|i mean|you know|kind of|sort of`) hat kein "quasi"-Äquivalent. "Sort of" ist semantisch näher dran und steht als offener Punkt für die Sammelstelle.
- **Test über `compress_prompt`-Idempotenz hinaus** — der bestehende Idempotenz-Test reicht; P219-pre fügt nur einen positiven Test hinzu dass "quasi" das Kompressions-Verfahren überlebt.

**Lessons (2):**

1. **Filler-Word-Listen sind Bedeutungs-relevant — semantische Qualifier gehören NIE rein.** "quasi", "sort of", "kind of" sind Hedge-Wörter, die das Vertrauensniveau einer Aussage modulieren. Sie zu entfernen verändert die Aussage: "Das ist quasi ein Reasoning-Schritt" sagt etwas anderes als "Das ist ein Reasoning-Schritt". Lesson generalisierbar: vor jeder Erweiterung einer Stop-/Filler-Word-Liste prüfen — ist das Wort funktional (Diskursmarker, Pausen-Filler) oder semantisch (Qualifier, Verstärker, Negation)? Funktional: weg damit. Semantisch: drin lassen, auch wenn es im Statistik-Profil prominent ist.
2. **Supervisor-Bug-Sammelstelle als Anti-Pattern-Schutz.** Ohne Sammelstelle kämen Bugs als Einzelaufträge an Coda — jeder mit eigenem Kontext-Aufbau, eigenem Commit, eigenem Sync. Mit Sammelstelle gibt es einen einzigen Sammel-Auftrag pro Trigger-Phrase, mit gemeinsamem Commit + gemeinsamer Sync-Welle. Lesson: bei agentischen Multi-Loop-Systemen lohnt es sich, Buffer-Schichten zwischen "Beobachtung" und "Aktion" einzubauen — wer die Aktion direkt mit der Beobachtung verzahnt, kriegt zu viele kleine Patches und verliert den Überblick. Die Sammelstelle ist der Buffer, der "los"-Trigger ist der Flush.

**Tests:** 7 neue in [`test_p219_pre_quasi_preserved.py`](zerberus/tests/test_p219_pre_quasi_preserved.py) — 4 Klassen: `TestQuasiNotInWhisperCleanerJson` (2 — Source-Audit auf JSON: kein Pattern enthält "quasi", Filler-Regex matcht "quasi" nicht), `TestCleanTranscriptPreservesQuasi` (3 — Roundtrip: `clean_transcript` lässt "quasi" im einfachen Satz, am Satz-Anfang und neben anderen Fillern stehen; Sanity-Check dass andere Filler weiterhin gefiltert werden), `TestQuasiNotInPromptCompressorStopwords` (2 — `_STOPWORDS`-Set ohne "quasi", `compress_prompt` lässt "quasi" stehen). Alle 7 lokal grün. `test_prompt_compressor.py` (14 Tests) bleibt grün — keine Regression. Volle Suite-Differenz erwartet: +7 Tests (P218-pre-Baseline 2670 → P219-pre 2677 lokal).

**Logging-Tag:** keiner — Pure-Function-Pfade, kein Server-Log betroffen.

---

## Vorheriger Patch

**Patch 218-pre** — FAISS Dim-Mismatch-Recovery im globalen RAG-Add-Pfad (Bugfix, Pattern-Transfer aus P199) (2026-05-07)

Chris hat den Bug zweimal gemeldet (2026-05-06 und 2026-05-07): `POST /hel/admin/rag/upload` wirft HTTP 500 mit `AssertionError: assert d == self.d` aus `_index.add(vec)` in [`zerberus/modules/rag/router.py`](zerberus/modules/rag/router.py). Ursache: der DualEmbedder (P187) liefert je nach aktivem Modell unterschiedliche Dimensionen (DE=768 via `cross-en-de-roberta`, EN=1024 via `multilingual-e5-large`, Legacy=384 via `MiniLM`); wenn der Index-on-Disk mit Dim X gebaut wurde und der aktive Embedder Dim Y produziert (GPU/CPU-Wechsel oder Modell-Switch zwischen Patches), kracht FAISS deterministisch im Add-Pfad. Der globale RAG hatte das Pattern bisher NICHT, der Per-Projekt-RAG (P199) hat es seit Monaten gelöst.

**Architektur: Pattern-Transfer aus P199 — Helper plus Dim-Check plus WARN-Log plus persistierender Reset.**

- **Neuer Helper** `_reset_index_inplace(target_dim, settings)` in [`router.py`](zerberus/modules/rag/router.py): baut globalen `_index` neu mit `faiss.IndexFlatL2(target_dim)`, leert `_metadata`, persistiert Index+Meta via `_resolve_paths(settings, dual_aware=True)` damit der nächste Server-Start den frischen Index am Dual-Storage-Pfad liest (`de.index`/`de_meta.json` bei `_use_dual=True`, sonst Legacy-Pfade). Nicht thread-safe gegenüber konkurrenten `_add_to_index` — der Add-Pfad ist Singleton-Thread (siehe `index_document`-Endpoint via `asyncio.to_thread`).
- **Dim-Check in `_add_to_index`** vor `_index.add(vec)`: wenn `_index is None` ODER `_index.d != int(vec.shape[1])`, dann WARN-Log mit `[RAG-218]`-Tag (alte Dim + alte Vektor-Anzahl + neue Dim + Hinweis "Reindex empfohlen") und Aufruf `_reset_index_inplace(incoming_dim, settings)`. Anschliessend `_index.add(vec)` — sicher, weil `_index` jetzt die passende Dim hat. Pattern direkt aus P199 `index_project_file` (vstack-Recovery), siehe lessons.md-Zeile "Embedder-Dim-Wechsel zwischen Sessions toleriert".
- **Recovery ist verlustbehaftet** — alte Chunks gehen weg (sie passten ohnehin nicht mehr zum aktiven Embedder). WARN-Log macht das transparent. Folge-Aktion: `POST /hel/admin/rag/reindex` (oder Sync-Skript erneut laufen) lädt den Korpus neu.
- **Dual-Modus-Coverage:** `_add_to_index` schreibt aktuell nur nach DE-Index (siehe bestehender Doc-Kommentar zu EN-Side-Chunks als Folge-Patch). EN-Index in `_init_sync` ist read-only — Dim-Mismatch dort tritt beim Search-Pfad auf, aber `faiss.search()` ist tolerant. Wenn EN-Add-Pfad jemals differenziert wird, kann der Helper 1:1 wiederverwendet werden.

**Was P218-pre bewusst NICHT macht:**

- **Kanonische Index-Dimension erzwingen** ("immer 768 projizieren") — ist Feature-Patch (Strategie a aus dem Bug-Report), nicht Bugfix. Auto-Reset löst den akuten Schmerz, ohne den DualEmbedder umzubauen.
- **`_DIM = 384`-Konstante entfernen.** Wird nur im Legacy-Pfad (`_load_or_create_index` und `_reset_sync`) verwendet, der bei `_use_dual=True` ohnehin nicht läuft. Auto-Reset im Add-Pfad heilt einen falschen Initial-Wert transparent.
- **Reindex-Endpoint transaktional machen** (Schulden-Liste #2, P213-pre-3) — separater Patch, bleibt nach P218-pre der niedrigste verbleibende Aufwand.
- **EN-Side-Add-Pfad differenzieren** — der bestehende Doc-Kommentar nennt das als Folge-Patch. Aktuell nicht im Hot-Path (Sync-Tool lädt einsprachige Korpusse, mehrsprachiger Add ist Folge-Patch).
- **Hel-UI-Card "Reindex empfohlen"-Banner** — die WARN-Zeile im Server-Log reicht für die Bug-Suche, ein UI-Banner wäre Feature-Add ohne klaren ROI.

**Lessons (3):**

1. **Pattern aus Projekt-Variante in globale Variante übertragen lohnt sich.** P199 hatte das Dim-Mismatch-Problem bereits sauber gelöst (vstack-Recovery + WARN-Log) und der Patch war seit 2026-04-26 stabil. Der globale Pfad hatte das Pattern nicht — Coda hat zwei Sessions versucht, den Bug woanders zu suchen. Lesson: bei wiederkehrenden FAISS/Embedder-Symptomen zuerst lessons.md grep nach "Embedder-Dim", dann die Schwester-Pfade vergleichen.
2. **`_FakeIndex`-Stubs sollten das echte Library-Verhalten nachahmen, nicht no-op werden.** Der bestehende Stub in `test_p213_pre_2_dual_storage.py` hatte `add(vec)` als no-op `self.ntotal += 1` ohne Dim-Check. Mein neuer Stub in `test_p218_pre_dim_mismatch.py` macht das echte Fail-Verhalten nach (`assert vec.shape[1] == self.d`) — wenn ein zukünftiger Refactor den Recovery-Pfad rauseditiert, knallen die Tests in derselben Stelle wie Production. Source-Audit-Tests greifen so zuverlässig.
3. **WARN-Log bei verlustbehafteter Recovery ist Pflicht — sonst sieht niemand den Datenverlust.** `[RAG-218]`-Tag mit alter+neuer Dim + alter Vektor-Anzahl macht den Mismatch in der Log-Aggregation auffindbar und nennt "Reindex empfohlen" als Folge-Aktion. Anti-Pattern wäre stilles Auto-Reset — der nächste Search wundert sich dann über leere Treffer.

**Tests:** 15 neue in [`test_p218_pre_dim_mismatch.py`](zerberus/tests/test_p218_pre_dim_mismatch.py) plus 1 Stub-Erweiterung in [`test_p213_pre_2_dual_storage.py`](zerberus/tests/test_p213_pre_2_dual_storage.py). 4 Test-Klassen: `TestAddToIndexNoMismatch` (2 — Hot-Path: passende Dim macht keinen Reset), `TestAddToIndexDimMismatch` (7 — die drei Pflicht-Fälle aus dem Bug-Report plus Edge-Cases: bestehende Chunks + Reset, leerer Index + Reset, dual-aware-Persistierung, Legacy-Pfad-Persistierung, Metadata-Komplett-Clear, Total-Return-nach-Rebuild, `_index is None`-Defensive), `TestResetIndexInplaceHelper` (3 — Helper direkt: erstellt leeren Index, persistiert via DE-Pfad bei Dual, persistiert via Legacy-Pfad sonst), `TestSourceWiring` (3 — Defense-in-Depth: Helper exportiert, `_add_to_index` ruft Helper, `[RAG-218]`-Tag im Source). Alle 15 grün lokal. P213-pre-2-Stub erweitert um `d=384`-Default — die 27 P213-pre-2-Tests bleiben grün.

Volle Unit-Suite: 2655 Baseline (P216) + 15 neue P218-pre = **2670 passed lokal erwartet** (im Worktree-Lauf 2661 passed wegen 9 pre-existing Failures: 5 Doku-Drift in `test_p210/test_p213_pre_4` durch P217-pre-Stand-Anker-Aufsplittung, plus 4 HANDOVER-bekannte Failures + 2 Worktree-Setup-Drift). **Bonus durch Doku-Update:** der `**Letzter Patch:**`-Marker in `huginn_kennt_zerberus.md` heilt die 5 Doku-Failures aus `test_p210/test_p213_pre_4` — die zählen also nach diesem Patch nicht mehr als Failures. Erwartete Hauptrepo-Baseline nach P218-pre: **2670 passed**.

**Logging-Tag:** `[RAG-218]` mit Sub-Event `Dim-Mismatch beim Index-Add: alte Dim={old_dim}, alte Vektoren={old_ntotal}, neue Dim={incoming_dim}. Index wird mit der neuen Dimension neu aufgebaut (Embedder-Wechsel zwischen Sessions). Alte Chunks gehen verloren — Reindex empfohlen.` Worker-Protection-konform: kein User-Content im Log, nur Dim-Metriken.

---

## Vorheriger Patch

**Patch 217-pre** — `sync_repos.ps1` Quote-Escape-Fix (Tooling-Schuld #1 aus dem 2026-05-05-HANDOVER geschlossen) (2026-05-07)

`sync_repos.ps1` hat zwei `git commit -m "Sync: $patchMsg"`-Aufrufe (Ratatoskr und Claude-Repo) — beide expandieren `$patchMsg` direkt im PowerShell-Doppelquote-String. Wenn die zuletzt committete Zerberus-Message selbst Anführungszeichen enthält (z. B. `'Patch X: "abc" implementiert'`), zerschießt das Win32-CommandLineToArgvW-Quoting und git bekommt mehrere fehlerhafte Argumente statt einer einzelnen `-m`-Message. Die letzten zwei Sessions hatten Glück, weil keine Zerberus-Messages Quotes enthielten (siehe Vorgänger-HANDOVER). P217-pre macht das robust statt zufällig.

**Architektur: PowerShell-Helper plus Smoke-Test plus 1:1-Quote-Lessons-Eintrag — kein Code-Pfad in Zerberus selbst berührt.**

- **Helper-Funktion** `Invoke-GitCommitFromString` in [`sync_repos.ps1`](sync_repos.ps1) schreibt die fertige Message in eine Temp-Datei (UTF-8 ohne BOM via `[System.Text.UTF8Encoding]::new($false)`) und ruft `git commit -F $tmp`. Damit landet die Message als Datei-Inhalt bei git, völlig unabhängig vom PowerShell-Argument-Quoting. Ohne BOM, weil git sonst die BOM in die Commit-Message einwoben würde. `try/finally` mit `Remove-Item -Force -ErrorAction SilentlyContinue` räumt die Temp-Datei auch im Fehlerfall auf.
- **Verdrahtung** beide `git commit -m "Sync: $patchMsg"`-Aufrufe (Z58 alt → Z78 neu für Ratatoskr, Z79 alt → Z99 neu für Claude-Repo) durch `Invoke-GitCommitFromString -Message "Sync: $patchMsg"` ersetzt. Variableninterpolation passiert **vor** der Funktion → Funktions-Parameter ist ein einzelner Stringwert → keine Quote-Probleme mehr.
- **Smoke-Test** [`scripts/test_sync_repos_quote_escape.ps1`](scripts/test_sync_repos_quote_escape.ps1) erzeugt ein Wegwerf-Git-Repo unter `$env:TEMP`, führt `Invoke-GitCommitFromString` mit sieben progressiv bösartigen Commit-Messages aus (`plain`, `double-quotes`, `single-quotes`, `mixed-quotes`, `umlauts`, `dollar-sign`, `backtick`) und vergleicht den `git log -1 --format="%s"`-Round-Trip mit dem Erwartungswert. Cleanup im `finally`. Exit-Code 0 = grün, 1 = mindestens ein Case fehlerhaft. Aufruf: `powershell -ExecutionPolicy Bypass -File scripts/test_sync_repos_quote_escape.ps1`.
- **Lokale Verifikation:** `7/7 PASS` — inklusive `'Patch X: "abc" implementiert'`, `'Patch X: "outer ''inner'' outer" Test'`, Umlaute, `$var`/`${expr}` und ` ``inline`` `-Backticks. Ohne den Fix wäre mindestens der `double-quotes`-Case in PowerShell 5.1 zerlegt worden.

**Was P217-pre bewusst NICHT macht:**

- **Andere Skripte mit demselben Anti-Pattern abgrasen.** `scripts/sync_huginn_rag.ps1` und `scripts/verify_sync.ps1` haben ihr eigenes Quoting-Verhalten und sind nicht betroffen (Sync-Skript baut Headers/JSON, kein `git commit -m "..."`; verify_sync liest nur). Falls in Zukunft ein neues Skript ähnliches macht, Helper aus `sync_repos.ps1` rauslösen statt copy-paste.
- **PowerShell-Version-Bump.** Der Bug ist eigentlich ein PS-5.1-Native-Argument-Quoting-Loch; PS 7.3+ hat `PSNativeCommandArgumentPassing` und würde das von selbst escapen. Wir haben aber Windows PowerShell 5.1 als baseline (Standard auf Windows 11), kein Pflicht-Bump.
- **CI-Integration des Smoke-Tests.** Es gibt keine PowerShell-CI in der Zerberus-Codebase; der Test ist on-demand-Smoke-Test, läuft nur wenn jemand sync_repos.ps1 anfasst.
- **Zerberus-Code anfassen.** Nur Tooling. Die 2655-passed-Baseline + e2e-Suiten bleiben unberührt.

**Lessons (1):**

1. **PowerShell 5.1: für native Tools (`git`, `docker`, `curl`) mit beliebig-quotierten Strings IMMER Argument via Datei + `-F`/`-T`/`@file` übergeben, NIE inline `-m "$var"`.** Das Win32-CommandLineToArgvW-Argument-Quoting eskapiert Anführungszeichen im Variableninhalt nicht zuverlässig — der Aufruf zerlegt sich in mehrere Argumente, das Tool sieht Müll. Der Bug springt nur an, wenn der Variableninhalt tatsächlich `"` enthält → klassisches "läuft heute, kracht in drei Wochen". Robust: Datei schreiben, Tool mit `-F file` aufrufen, Datei wegräumen. UTF-8 ohne BOM, sonst landet die BOM im Inhalt.

**Tests:** Volle Unit-Suite **bleibt 2655 passed** (P216-Baseline) — kein Code-Pfad berührt, daher keine Regression möglich. Smoke-Test `scripts/test_sync_repos_quote_escape.ps1` lokal `7/7 PASS`. Manuelle Tests: 1/106 unverändert. e2e-Suiten unverändert (78 passed / 7 skipped / 0 failed).

**Logging-Tag:** keiner — reines PowerShell-Tooling, kein Server-Code.

---

## Vorheriger Patch

**Patch 216** — Pre-LLM Input Validator (Phase 5a Ziel #15 ABGESCHLOSSEN — Phase 5a damit komplett) (2026-05-05)

Vor P216 lief jede User-Message ungefiltert durch die volle Pipeline (Persona → Project-Overlay → RAG → Spec-Probe → Haupt-LLM-Call), auch wenn die Eingabe offensichtlich Junk war (`""`, `"qwerty"`, `"!!!!!!"`, 50 KB copy-paste-Logfile). Jeder dieser Calls kostete Tokens (RAG-Embedding + Hauptmodell) und Server-Last (VRAM-Slot, GPU-Queue) ohne sinnvolle Antwort. P216 baut eine Pure-Function-Reject-Schicht VOR dem teuersten Schritt: leere/zu-lange/symbol-only/keyboard-spam-Eingaben werden mit freundlichem DE-Text per Early-Return abgewiesen, kein LLM-Call, kein RAG, kein Persona-Lookup.

**Architektur: Pure-Function-Schicht plus Audit-Tabelle plus Feature-Flag plus Verdrahtung als FRÜHESTER Block.**

- **Modul** [`zerberus/core/input_prevalidator.py`](zerberus/core/input_prevalidator.py) mit vier Reject-Pattern (Pure-Function, O(n), nur stdlib): `_is_empty(text)` (whitespace-only nach `strip`), `_is_too_long(text, max_chars)` (>`max_chars`, Default 8000), `_is_pure_symbols(text)` (kein Buchstabe/keine Ziffer nach `strip`, nur Punctuation/Whitespace, Emoji passt durch), `_is_keyboard_spam(text, *, min_length, threshold)` (nach Whitespace+Punctuation-Strip mindestens `min_length` Zeichen, dann ENTWEDER 80%+ aus einem einzelnen Char ODER 80%+ als zusammenhängender Klaviatur-Run auf den drei Reihen `qwertyuiop`/`asdfghjkl`/`zxcvbnm`).
- **Reihenfolge** `empty → too_long → pure_symbols → keyboard_spam` — empty-Check vorrangig (semantisch korrekt für `" "*9000`), too_long vor pure_symbols (Token-Schutz hat Vorrang). Erste Match gewinnt → `RejectVerdict`-Dataclass mit `code`/`human_message`/`original_length`.
- **Audit-Tabelle** `prevalidation_rejects` in [`database.py`](zerberus/core/database.py): `audit_id` (UUID4 hex), `session_id`, `reason_code`, `original_text` (truncated 500 Bytes UTF-8-safe), `original_length`, `created_at`. Best-Effort-Insert via `store_prevalidation_audit(...)`. Auswertung: wie oft greift der Validator, welcher Code dominiert, kommt der gleiche User mehrfach mit Spam?
- **Feature-Flag** `PrevalidationConfig` in [`config.py`](zerberus/core/config.py) mit `enabled=True` / `max_chars=8000` / `keyboard_spam_min_length=5` / `keyboard_spam_threshold=0.8`. Bei `enabled=False` fällt der ganze Block weg (Pre-P216-Verhalten als zuverlässiger Rollback).
- **Verdrahtung in [`legacy.py::chat_completions`](zerberus/app/routers/legacy.py)** als FRÜHESTER Block direkt nach `last_user_msg`-Extraction, VOR Persona-Wrap (P184), Project-Overlay (P197), Runtime-Info (P185), Decision-Box-Hint (P118a), Prosodie (P190+P204), Projekt-RAG (P199), Spec-Contract (P208) und LLM-Call. Begründung: maximaler Token-Spar — Validator-Reject vor allen drei spart 100% der nachgelagerten Arbeit.
- **Bei Reject** drei Branches: UUID4-`audit_id` generiert, `[PREVAL-216] reject session=... code=... len=... audit_id=...` Logging, `store_prevalidation_audit(...)` mit truncated text (fail-open), `store_interaction("user", ..., "assistant", ...)` + `update_interaction()` (Pattern wie spec-cancelled aus P208 — User-History bleibt vollständig), `return ChatCompletionResponse(model="prevalidation-reject-<code>", ...)` mit dem `human_message`-Text als `assistant`-Choice.

**Was P216 bewusst NICHT macht:**

- **Sprach-Erkennung** ("ist die Eingabe Deutsch?") — zu viele False-Positives ohne LLM-Call, der Validator soll billig bleiben.
- **Cost/Budget-Cap** — separate Schicht (User-State-abhängig), kann eigener Folge-Patch werden.
- **Re-Prompt-Loop bei Reject** — User schickt halt was Sinnvolles als nächstes; kein automatischer Retry.
- **Frontend-Sondercard** — die Reject-Antwort wird wie eine normale Bot-Bubble gerendert (Pattern wie spec-cancelled).
- **Tunable-Per-User-Schwellen** — alle User teilen sich `max_chars`/`keyboard_spam_threshold`. Wer legitim Logfiles pasten will, schickt mehrere Nachrichten.
- **Whitelist-Liste** für "vertrauenswürdige" User — Permission-Level-aware Bypass wäre Folge-Patch.

**Lessons (3):**

1. **Pre-LLM-Validator-Block muss FRÜHESTER Schritt sein, sonst zerstört eine andere Reihenfolge den Token-Spar-Sinn.** Wenn der Validator nach RAG/Spec/Persona käme, hätten wir die teuren Calls schon ausgeführt — Sinn der Schicht ist 0% Vor-LLM-Arbeit bei Reject.
2. **`enabled=False` muss den ganzen Block deaktivieren — Pre-P216-Verhalten als zuverlässiger Rollback.** Wenn die Heuristik einmal jemanden falsch ablehnt und der Bug kritisch wird, muss ein einzelner Config-Flip alles deaktivieren ohne Code-Edit.
3. **Reject-Reason-Codes sind eine geschlossene Whitelist.** `REJECT_CODES` als Set; Custom-Codes werden silent verworfen. Pattern aus P206/P208/P209/P213/P214: jede neue Pipeline-Komponente bekommt zwei oder drei kleine Whitelists statt einer grossen.

**Tests:** 82 in [`test_p216_input_prevalidator.py`](zerberus/tests/test_p216_input_prevalidator.py). 12 Klassen: `TestIsEmpty` (4), `TestIsTooLong` (5), `TestIsPureSymbols` (8), `TestKeyboardRunCount` (7), `TestIsKeyboardSpam` (9), `TestShouldRejectInput` (11 inkl. Reihenfolgen-Tests), `TestHelpers` (8 — truncate UTF-8-safe, audit_id UUID4, human_message_for-Whitelist+Fallback), `TestAuditTrail` (4), `TestLegacySourceAudit` (10 — Reihenfolge VOR persona/spec/rag, Early-Return, Audit-Call, fail-open), `TestConfigSourceAudit` (5), `TestEndToEnd` (6 — alle vier Reject-Pfade machen 0 LLM-Calls, Pass-Pfad macht LLM-Call, `enabled=False` lässt alles durch), `TestSmoke` (4).

Lokal: 2573 baseline (P215-pre-1) → **2655 passed** (+82 P216, 0 NEUE Failures), 5 failed pre-existing identisch (test_faiss_migration + test_patch169_bugsweep + 3× test_patch185_runtime_info), 4 xfailed identisch, 131 deselected. **e2e-Sweep alle drei Suiten gegen Live-Server grün:** Vidar 20 passed / 1 skipped, Loki 41 passed / 4 skipped, Fenrir 17 passed / 2 skipped. Gesamt 78 passed / 7 skipped / 0 failed (identisch P215-pre-1-Baseline — P216 ist Backend-only).

**Logging-Tag:** `[PREVAL-216]` mit Sub-Events `reject session=... code=... len=... audit_id=...`, `audit_written audit_id=... session=... reason=... len=...`, `audit_failed (non-fatal): ...`, `audit_import_failed: ...`, `Pipeline-Fehler (fail-open): ...`. Worker-Protection-konform: kein User-Content im Log, nur reason_code + Längen-Metriken.

---

## Vorheriger Patch

**Patch 215-pre-1** — Touch-Target-Vervollständigung `.copy-btn` + `.bubble-action-btn` auf Mobile-44px (Cluster G aus P215-Sweep) (2026-05-05)

P215 hatte den Mobile-Touch-Sweep gegen die UI gemacht und 30 Touch-Target-Violations gefixt — drei CSS-Cluster: `.hamburger`, `.session-pin-btn`, `.sidebar-action-btn`. Beim Final-Sweep blieben zwei Selektoren übrig, deren 44x44-CSS nur in `@media (hover: none) and (pointer: coarse)` stand statt im Default-Block: `.copy-btn` (Code-Card-Copy-Button im Nala-Frontend) und `.bubble-action-btn` (Edit/Delete-Aktionen pro Bubble). Auf Geräten ohne Hover-Trigger im Browser-Engine fielen die Werte zurück auf <44px. P215-pre-1 zieht die Werte raus aus der Media-Query in den Default-Block.

**Architektur: zwei CSS-Tweaks plus zwei Vidar-Tests plus HTML-Report-Re-Run.**

- [`nala.py`](zerberus/app/routers/nala.py) — `.copy-btn` und `.bubble-action-btn` bekommen `min-width: 44px` + `min-height: 44px` direkt im Default-Block. Die Mobile-Media-Query bleibt redundant erhalten als Defense-in-Depth — bei zukünftigen Refactors fällt der Default-Block evtl. weg, dann greift die Media-Query weiter.
- **Tests:** Zwei neue Vidar-Tests `test_copy_btn_min_44_in_default` und `test_bubble_action_btn_min_44_in_default` als CSS-Source-Audit (Substring-Match auf `min-height: 44px` im Default-Block UND in Media-Query).
- **e2e-Sweep:** alle drei Suiten gegen Live-Server grün (Server-Restart-Pflicht nach CSS-Aenderung war eingehalten — `start_stable.bat` ohne `--reload`). Vidar 20 passed / 1 skipped, Loki 41 passed / 4 skipped, Fenrir 17 passed / 2 skipped. Vorher P215-Baseline: Vidar 19/21, Loki 41/45, Fenrir 37/39 → Touch-Target-Erweiterung der Vidar-Suite +2, sonst stabil.

**Lessons (1):**

1. **Touch-Target-CSS-Werte gehören in den Default-Block, NICHT nur in `@media (hover: none) and (pointer: coarse)`.** Browser-Heuristik für Mobile-Detection ist nicht überall identisch — bei manchen Geräten/Browser-Engines springt die Media-Query nicht an, dann fallen die Werte auf den Default zurück. Default-Block 44x44 + Media-Query als Defense-in-Depth ist die robuste Variante.

**Tests:** 2 neue + Vidar-Sweep. Lokal: 2571 baseline (P215) → **2573 passed** (+2 P215-pre-1).

---

## Vorheriger Patch

**Patch 215** — Loki/Fenrir/Vidar-Pflicht-Regel + Nachhol-Sweep mit drei CSS-Touch-Target-Fixes (2026-05-05)

Coda hatte seit P93 (Bau von Loki/Fenrir/Vidar) Dutzende UI-/Auth-/Pipeline-Patches gemacht ohne die drei Suiten konsequent gegen den Live-Server auszuführen — und am Ende stand jeweils "Chris muss noch testen". Anti-Pattern: was bequem ist (Unit-Tests, Mocks) wurde gebaut, was notwendig ist (Live-E2E gegen `start_stable.bat`-Server) wurde vermieden. P215 erhebt den Pflicht-Lauf zur Prozess-Regel in CLAUDE_ZERBERUS.md UND fixt die 30 Touch-Target-Violations + 16 driftbedingten Test-Failures, die im Nachhol-Sweep aufschlugen.

**Architektur: Prozess-Regel plus drei CSS-Cluster plus Test-Anpassungen plus zwei dokumentierte Skips.**

- **Prozess-Regel in CLAUDE_ZERBERUS.md** (neue Top-Section "Loki / Fenrir / Vidar — Pflicht-Lauf bei jedem UI-/Auth-/Pipeline-Patch"). Bei JEDEM Patch der UI/Auth/Chat-Pipeline/RAG/Guard/Huginn/Endpoint anfasst → Server starten via `start_stable.bat` und ALLE drei Playwright-Suiten gegen den Live-Server. Reihenfolge: 1) **Vidar** (Smoke ~60s, Go/No-Go), 2) **Loki** (E2E), 3) **Fenrir** (Chaos). `-m e2e` ist Pflicht-Marker. Failures = Blocker — kein Push ohne grüne Suiten ODER dokumentierter Skip mit Schulden-Verweis.
- **Drei CSS-Touch-Target-Fixes** in [`hel.py`](zerberus/app/routers/hel.py) und [`nala.py`](zerberus/app/routers/nala.py): `.hamburger` 40x33 → 44x44, `.session-pin-btn` 23x14 → 44x44, `.sidebar-action-btn` Höhe 37 → ≥44px. Die Werte standen vorher teilweise in `@media (hover: none) and (pointer: coarse)`, aber nicht im Default-Block — wurden in den Default-Block gezogen.
- **Sechs veraltete Selektoren angepasst:** `.icon-btn[aria-label='Einstellungen']` → Sidebar-Footer-`button.sidebar-footer-cog` via offene Sidebar (Settings-Cog ist seit Monaten im Sidebar-Footer, der Test prüfte den alten Header-Cog). Plus `.bubble-max-width 90% → 92%` (mobile) und `75% → 80%` (desktop).
- **Zwei dokumentierte Skips mit Schulden-Eintrag:** `test_f_pace_01_master_toggle_stable` (Pacemaker-Master-Toggle wurde durch Minuten-Setting ersetzt — Test-Anpassung als Schuld), `test_f_tts_02_button_not_duplicate` (TTS-Race-Condition bei rapid-fire-Sends, mit `wait_for_function`-Fallback gefixt; Skip greift bei sehr langsamen Bot-Antworten).
- **Bug-Limit:** `max 3 echte Bug-Fixes` als Sweep-Disziplin — sonst wäre der Patch ein 3-Tage-Marathon. Der Rest der Drifts geht in die Schulden-Liste.

**Was P215 bewusst NICHT macht:**

- **Pacemaker-UI-Refactor** — Master-Toggle-Drift bleibt als Schuld in HANDOVER. Skip-mit-Begründung statt fail.
- **TTS-Pipeline-Tuning** — Race-Condition wird mit `wait_for_function` gefixt; tieferes Debugging der TTS-Pipeline ist eigener Patch.
- **Settings-Cog-Mobile-Refactor** — Test braucht zwei Klicks (Burger + Cog), das ist UX-Schuld für eigene Diskussion.
- **CI-Integration** — Suiten laufen weiter manuell. CI-Runner für Live-Server-e2e ist eigener Patch.

**Lessons (4):**

1. **Wenn ein System eine Smoke-/E2E-/Chaos-Suite hat, IST diese Suite das Pflicht-Gate vor "Patch ist fertig".** Nicht "wenn ich Zeit habe", nicht "wenn der Patch gross genug ist" — bei JEDEM Patch der den von der Suite abgedeckten Bereich anfasst.
2. **UI driftet still.** Beim Sweep fielen 16 von 105 e2e-Tests über veraltete Selektoren / veraltete Werte / echte Touch-Target-Bugs — keiner dieser Drifts wurde im PR diskutiert, sie reisen in CSS-Patches mit ohne dass jemand "ach das ist ja jetzt nicht mehr 44px" sagt.
3. **Skip-mit-Begründung ist die richtige Haltung bei pre-existing Drift.** Drei Optionen: 1) echter Bug → fixen, 2) veralteter Test → Selektor anpassen, 3) UI hat sich strukturell geändert → `pytest.skip("Klartext-Begründung als Schuld in HANDOVER")`. Anti-Pattern: rote Tests mit "kommt später" — rote Tests werden ignoriert, grüne werden gelesen.
4. **Bug-Limit pro Patch macht den Sweep ueberhaupt erst durchfuehrbar.** Ohne Limit wird der Sweep entweder ein 3-Tage-Patch oder gar nicht gestartet. Mit `max 3` liefert er was sich in einer Session machen lässt und gibt der nächsten eine glasklare Liste.

**Tests:** Drei CSS-Tests in Vidar-Suite + sechs Selektor-Anpassungen in Loki/Fenrir + zwei dokumentierte Skips + neue Source-Audit-Tests für die CLAUDE_ZERBERUS-Regel-Section. Lokal: 2528 baseline → **2571 passed** (+43 P215). e2e-Sweep grün: Vidar 19 passed / 2 skipped (1 Schuld), Loki 41 passed / 4 skipped (cosmetic), Fenrir 37 passed / 2 skipped (dokumentierte Schuld).

---

## Vorheriger Patch

**Patch 214-pre-3** — Scheduler-Sandbox-Anbindung kind=shell (Phase 5a Ziel #14 ABGESCHLOSSEN) (2026-05-05)

Vor P214-pre-3 war `kind=shell` ein Stub aus P214-pre-1, der mit `error: Sandbox-Run noch nicht angeschlossen` auditierte. P214-pre-3 schliesst die Sandbox an: Cron-getriebene Shell-Runs laufen mit Pre-Snapshot (P207, fail-CLOSED), `execute_in_workspace`-Pfad mit `writable=True` Workspace-Mount, Post-Snapshot (best-effort), Output-Format mit `[SCHED-214 shell exit=N took=Mms]`-Header. Defense-in-Depth: Sandbox-Isolation (P171: --network none, read-only Root, no-new-privileges, PID/Memory/CPU-Limits) + Workspace-Mount (P203c) + Path-Check (`is_inside_workspace`) + Secrets-Filter (P212 stdout/stderr) + Pre-Snapshot-Anker (P207).

**Architektur: shell-Handler plus Pure-Function-Erweiterungen plus SandboxConfig.bash_image plus Tests.**

- **`_execute_kind_shell(task)`** in [`zerberus/modules/scheduler/runner.py`](zerberus/modules/scheduler/runner.py): Pfad 1) `should_snapshot_shell_run`-Gate (project_id Pflicht — Workspace-Mount erforderlich), 2) `snapshot_workspace_async(label="sched214-before-<id8>")` fail-CLOSED, 3) `execute_in_workspace(language="bash", writable=True, project_id, base_dir)` (kein direkter SandboxManager-Call — sonst Path-Sicherheits-Check umgangen), 4) `snapshot_workspace_async(label="sched214-after-<id8>", parent_snapshot_id=before["id"])` best-effort, 5) `format_shell_output` + `is_shell_run_successful`.
- **Pure-Function-Erweiterungen** in [`zerberus/core/scheduler_tasks.py`](zerberus/core/scheduler_tasks.py): `SHELL_SANDBOX_LANGUAGE="bash"` Konstante, `should_snapshot_shell_run(task)` (project_id-Pflicht), `format_shell_output(*, exit_code, stdout, stderr, ...)` mit Header `[SCHED-214 shell exit=N took=Mms[ truncated][ sandbox_error=...]]\n--- STDOUT ---\n...\n--- STDERR ---\n...`, `is_shell_run_successful(*, exit_code, sandbox_error)` (nur exit==0 + kein sandbox_error).
- **SandboxConfig.bash_image** in [`zerberus/core/config.py`](zerberus/core/config.py): Default `"debian:bookworm-slim"` (~80MB, bash-fähig). `SandboxManager._image_and_command` mappt `language=="bash"` auf `(bash_image, ["bash", "-c", code])`. **Operator muss bewusst opt-in:** `bash` zu `allowed_languages` hinzufügen UND `bash_image` ziehen — sonst "Sprache nicht erlaubt"-Reject (beabsichtigt: Shell-Runs sind neues Risiko-Profil).
- **Tests:** 43 Unit + Source-Audits in [`test_p214_pre_3_scheduler_shell.py`](zerberus/tests/test_p214_pre_3_scheduler_shell.py) (TestShouldSnapshotShellRun / TestFormatShellOutput / TestIsShellRunSuccessful / TestShellSandboxLanguageConstant / TestSandboxConfigBashImage / TestSandboxManagerBashCommand / TestRunnerShellPathSourceAudit / TestSandboxManagerBashSourceAudit / TestSmoke). Plus 3 Integration-Tests in `test_integration_p214_scheduler.py` (`TestShellPathExecution`): Happy-Path mit gemocktem `snapshot_workspace_async` + `execute_in_workspace`, Sandbox-Error → audit-error mit `sandbox_error=Timeout` im Output, Pre-Snapshot=None → `execute_in_workspace` wird NICHT aufgerufen (Sicherheitspfad-Test).

**Was P214-pre-3 NICHT macht:** Multi-Image-Support, Per-Task-Image-Override, automatischer bash-Default in `allowed_languages`. Bewusst restriktiv — Operator-Awareness bei Shell-Cron-Runs.

**Lessons (3):**

1. **Beim Verdrahten eines neuen schreibenden Pfads gegen einen Snapshot-Layer: fail-open vs. fail-CLOSED entscheiden.** Pre-Snapshot bei writable Run macht zwei Dinge gleichzeitig (Rollback-Anker + Diff-Vergleich) — wenn er crasht, hat man WEDER Rollback NOCH Diff. Pre-Snapshot ist fail-CLOSED, Post-Snapshot ist best-effort.
2. **Test-Pattern für fail-closed-Pfade: explizit testen dass die GEFÄHRDETE Operation NICHT aufgerufen wird.** `await_count == 0` als entscheidender Assert, nicht "result.status == error" allein.
3. **Neue Sandbox-Sprache hinzufuegen heisst NICHT, sie automatisch in `allowed_languages` aufnehmen.** Image-Default da + Sprache nicht in Whitelist = harmlos bis Operator zustimmt.

**Tests:** Lokal: 2530 baseline (P214-pre-2) → **2528 passed** (-2 wegen Test-Reklassifikation, +43 P214-pre-3). 0 NEUE Failures.

---

## Vorheriger Patch

**Patch 214-pre-2** — Scheduler-UI + LLM-Anbindung kind=prompt (Phase 5a Ziel #14, Forts.) (2026-05-05)

Vor P214-pre-2 war `kind=prompt` ein Stub aus P214-pre-1, und Hel-UI hatte keinen Scheduler-Tab. P214-pre-2 baut beides: Hel-UI Tab "Scheduler" mit Liste/Form/Lauf-Historie (auth-gated CRUD-Endpoints), LLM-Anbindung für `kind=prompt` mit Persona-Layer (P197) + Projekt-RAG (P199), Update-Validation gegen Mass-Assignment, Run-History-View für UI.

**Architektur: Hel-UI plus auth-gated CRUD plus prompt-Handler plus Update-Validation plus Run-History.**

- **Hel-UI Tab "Scheduler"** in [`hel.py`](zerberus/app/routers/hel.py): Liste aller Tasks (Name/Cron/Kind/Projekt/letzter Lauf/Status + Aktionen Jetzt-laufen/Edit/Löschen), Form-Overlay zum Anlegen/Editieren mit Cron-Beispielen, Lauf-Historie pro Task (`schedRunsCard`, Top-20). **P203b-Pattern**: data-Attribute + `addEventListener` statt inline `onclick=` (Quote-Verschachtelung-Schutz, Test `test_alle_inline_scripts_parsen` greift).
- **Auth-gated CRUD-Endpoints** unter `/hel/admin/scheduled-tasks/*`: GET (mit payloads für Edit), POST (validate→build_task_record→insert→register_task), PATCH (validate_task_update + apply_task_update + re-register), DELETE (unregister + DB-Delete; Lauf-Historie bleibt per task_id-Soft-Link), GET `{task_id}/runs?limit=N` (Top-N via `build_run_history_entry`). Logging-Tag `[SCHED-214]` durchgängig.
- **`_execute_kind_prompt(task)`** in [`runner.py`](zerberus/modules/scheduler/runner.py) — Signatur-Aenderung von `(payload: str)` auf `(task)`, alle Handler bekommen jetzt das ganze DB-Row. Pfad: `resolve_project_overlay(project_id)` (Persona-Layer P197) → slug → `query_project_rag(project_id, payload, base_dir, k=5)` (P199 RAG, defensiv) → `format_rag_block(hits, project_slug)` → `build_prompt_messages(payload, project_slug, rag_context)` → `LLMService.call(messages, session_id="sched214-<task_id>")`. Output-Format `[SCHED-214 prompt model=X pt=Y ct=Z cost=$.6f]\n<answer>` — Token-Counts + Kosten im Audit.
- **`build_prompt_messages`** Pure-Function in `scheduler_tasks.py`. System-Message: Cron-Preamble (`Kein User da, schreib eine selbsterklaerende Notiz, keine Rueckfragen`) + optional Projekt-Slug + RAG-Block. User-Message: payload-Text. Stack: Scheduler-Preamble → Projekt-RAG → User-Persona-System → Task-Payload.
- **Update-Validation** `validate_task_update(updates, *, current_kind)` + `apply_task_update(updates)`: Whitelist `_UPDATABLE_TASK_FIELDS = (name, cron_expression, kind, payload, enabled, project_id)` — Mass-Assignment-Schutz. `current_kind` für payload-Validation wenn nur payload geupdated wird.
- **Run-History-View** `build_run_history_entry(...)` mit `DEFAULT_HISTORY_PREVIEW_BYTES=400` — schmaler View pro Run-Zeile, truncatet output/error_message separat für UI-Liste.

**Was P214-pre-2 NICHT macht:** `kind=shell` weiter Stub (kommt in P214-pre-3), Cost-Aggregation des Scheduler-LLM-Calls in `interactions.cost`, Telegram-Push bei prompt-Ergebnis, Multi-Tenant pro Projekt, Pause/Resume-Knopf.

**Lessons (3):**

1. **Inline-onclick mit verschachtelten Quotes ist eine Falle.** Python-String mit `'<button onclick="fn(\\'' + var + '\\')"...'` zerbricht — `\'`-Sequenz wird zu `'` im finalen JS-Output, ergibt zwei leere Strings. Fix-Pattern: `data-Attribute` + `addEventListener`. `test_alle_inline_scripts_parsen` aus P203b ist der Pflicht-Backstop.
2. **Wenn Handler-Funktionen mehr Kontext brauchen, dispatch das ganze DB-Row, nicht nur eine Spalte.** Refactor `(payload: str)` → `(task: ScheduledTask)` jetzt — spätere Erweiterungen (Per-Task-Modell-Override) brauchen kein zweites Refactor.
3. **Source-Audit-Tests sollen ARCHITEKTUR-Vertrag prüfen, nicht IMPLEMENTIERUNGS-Vertrag.** `test_runner_dispatch_table_covers_known_kinds` prüft nur Keys-Whitelist — Handler-Signatur-Refactor war moeglich ohne Audit-Test zu kippen.

**Tests:** 35 Unit + Source-Audits in [`test_p214_pre_2_scheduler_ui.py`](zerberus/tests/test_p214_pre_2_scheduler_ui.py) (TestValidateTaskUpdate / TestApplyTaskUpdate / TestBuildPromptMessages / TestBuildRunHistoryEntry / TestSchedulerUiSourceAudit / TestSchedulerCrudSourceAudit / TestRunnerPromptPathSourceAudit / TestSmoke). Plus 3 Integration-Tests für prompt-Pfad (LLM-Antwort landet im Output, leerer payload → error, LLM-Fehler-String → error-Audit). Lokal: 2495 baseline (P214-pre-1) → **2530 passed** (+35 P214-pre-2).

---

## Vorheriger Patch

**Patch 214-pre-1** — Wiederkehrende Jobs / Scheduler-Backend-Skelett (Phase 5a Ziel #14, Backend) (2026-05-04)

Vor P214 hatte Zerberus keine wiederkehrenden Jobs — der User konnte keine Cron-Tasks pro Projekt definieren ("jeden Morgen um 8 Uhr LLM-Zusammenfassung der Issues", "alle 6h Build-Status checken"). P214-pre-1 baut das Backend-Skelett: Pure-Function-Validation, Loader/Runner, DB-Schema, auth-freie Endpoints für Lese-Liste + manuellen Trigger. `kind=notice` voll funktional, `kind=prompt` und `kind=shell` als klar dokumentierte Stubs (kommen in P214-pre-2 und P214-pre-3).

**Architektur: Pure-Function-Schicht plus Loader/Runner plus DB-Schema plus auth-freie Endpoints.**

- **Pure-Function-Schicht** in [`zerberus/core/scheduler_tasks.py`](zerberus/core/scheduler_tasks.py): `CronSpec`-Dataclass, `parse_cron_expression`/`validate_cron_expression` (5-Felder, Range-Check, Token-Format `*`/`*/N`/`A-B`/`A,B,C`/Zahl), `validate_task_kind`/`validate_task_status`/`validate_task_trigger` (Whitelists), `validate_task_payload`/`validate_task_name`, `validate_task_dict` (Top-Level), `truncate_output` (Bytes-genau), `build_task_record`/`build_run_record` (Defaults + Truncation), `should_allow_manual_run` (Pre-Endpoint-Check). **KEIN APScheduler-Import dort** — Pure-Validation, sonst Test-Pfad-Crash bei fehlender Library.
- **Loader/Runner** in [`zerberus/modules/scheduler/runner.py`](zerberus/modules/scheduler/runner.py): `TaskScheduler`-Klasse als Wrapper um den existierenden `AsyncIOScheduler` (P57 Overnight). `reload_from_db()` liest enabled Tasks und registriert sie als Cron-Jobs. `register_task`/`unregister_task` für CRUD. `execute_task(task_id, trigger="cron"|"manual"|"boot")` als High-Level-Run mit Audit-Schreibung + Snapshot-Update. Module-Singleton `_task_scheduler` mit `get_task_scheduler`/`set_task_scheduler`-Pattern. `initialize_task_scheduler(apscheduler)` als Lifespan-Hook in `main.py`.
- **Kind-Dispatch-Tabelle** `_KIND_DISPATCH = {"notice": ..., "prompt": ..., "shell": ...}`. Nur `notice` voll funktional in P214-pre-1 — die anderen sind dokumentierte Stubs mit `error: <Pfad noch nicht angeschlossen>`.
- **Audit-Tabelle** `scheduled_task_runs` schreibt nur Endzustände (`done|error|skipped|running`-running ist transient). Spalten: `run_id` UUID4 + `task_id` + `started_at`/`finished_at` + `status` + `output`/`error_message` (truncated `DEFAULT_OUTPUT_MAX_BYTES=4096`) + `duration_ms` + `trigger`.
- **Endpoints auth-frei** in [`legacy.py`](zerberus/app/routers/legacy.py) (Dictate-Lane): `GET /v1/scheduled-tasks?project_id=N` (Lese-Liste, KEINE payloads ausgespiegelt aus Sicherheitsgründen), `POST /v1/scheduled-tasks/{task_id}/run` (manueller Trigger). CRUD-Schreib-Endpoints kommen in P214-pre-2 (auth-gated unter `/hel/admin/scheduled-tasks/*`).
- **Late-Binding** auf `_async_session_maker`: Runner liest `_db_mod._async_session_maker` zur Laufzeit (nicht `from ... import` zur Modul-Lade-Zeit) — wichtig für Test-Patches in Integration-Tests.
- **DB-Schema** in [`database.py`](zerberus/core/database.py): `ScheduledTask` mit `task_id` UUID4 + `project_id` (nullable, FK-Logik im Repo nicht im Model) + `enabled` (Integer-Bool) + Lauf-Snapshot-Felder. `ScheduledTaskRun` als Audit-Trail.

**Was P214-pre-1 NICHT macht:** prompt/shell-Kind-Pfade (kommen separat), CRUD-UI in Hel (P214-pre-2), Telegram-Push bei Done, Pause-/Resume-Buttons.

**Lessons (4):**

1. **`from module import _global_var` snapshottet die Referenz beim Import.** Late-Binding auf `_db_mod._async_session_maker` ist Pflicht, wenn Tests das Maker patchen wollen — sonst sieht der Patch `None` weil das Symbol Modul-lokal beim Import-Zeitpunkt einmal `None` war.
2. **SQLAlchemy AsyncEngine ist Loop-bound — `asyncio.run` mehrfach pro Test ist eine Falle.** Engine in einem `asyncio.run`-Loop erzeugt kann nicht in einem zweiten benutzt werden. Engine-Setup + Tabellen-Create + Test-Logik + Cleanup in EINE Coroutine packen, mit `asyncio.run` genau einmal pro Test.
3. **Drei kleine Whitelist-Schichten + UUID4-task_id.** `kind` (Funktions-Whitelist), `status` (Audit-Whitelist), `trigger` (Audit-Differentiation). `task_id` als UUID4-hex, nicht Integer-PK — Cross-Session-Defense analog HitL (P206) und Reasoning (P213).
4. **Stubs für noch nicht implementierte Kinds liefern explizit error mit klarem Hinweis.** `error: <Pfad noch nicht angeschlossen — kommt in P214-pre-N>`. Anti-Pattern: silent-skip oder fake-success.

**Tests:** 76 Unit + Source-Audits in [`test_p214_scheduler_tasks.py`](zerberus/tests/test_p214_scheduler_tasks.py). Plus 7 Integration-Tests in [`test_integration_p214_scheduler.py`](zerberus/tests/test_integration_p214_scheduler.py) (P213-pre-6-Pflicht): Insert→execute_task→Audit-Loop, unbekannte task_id, Stub-Behavior, reload_from_db mit 0/1 Task, disabled-skip, unregister-after-register. Lokal: 2419 baseline (P213-pre-6) → **2495 passed** (+76 P214-pre-1).

---

## Vorheriger Patch

**Patch 213-pre-6** — Integration-Test-Framework + Huginn-RAG-Smoketest + Sync-Tool-Auto-Verifikation (Phase 5a Pflicht-Infrastruktur) (2026-05-04)

Die P213-pre-Saga (vier Code-Patches: pre, pre-2, pre-4 plus pre-3 noch offen) zeigte ein Anti-Pattern: Coda hat Code + Doku + 17 Mock-Tests verifiziert, ohne EIN MAL gegen den laufenden Server zu prüfen, ob `_huginn_rag_lookup("Bei welchem Patch sind wir gerade?")` jetzt wirklich den aktuellen Patch liefert. Mock-Tests grün, echter Server hat weiterhin "Patch 178" halluziniert. P213-pre-6 baut das Integration-Test-Framework, das diese Lücke schließt: separate Marker, separate Datei-Konvention, Sync-Tool-Auto-Hook der nach erfolgreichem HTTP-Sync automatisch den Smoketest fährt.

**Architektur: Integration-Test-Konvention plus Smoketest plus Sync-Tool-Hook plus Source-Audit-Backstops.**

- **pytest.ini** mit `addopts = "not integration"` — Default-Run skippt Integration-Tests, sonst würde jeder lokale Run einen laufenden Server brauchen. `markers = integration: needs running server / live data` als zentrale Marker-Doku.
- **conftest.py** (root) mit `pytest_collection_modifyitems` der Integration-Tests automatisch markiert wenn der Datei-Praefix `test_integration_*.py` ist — kein manuelles `@pytest.mark.integration` pro Test nötig.
- **Test-File-Konvention:** `test_integration_*.py` als Datei-Praefix. Setup-Skip statt Fail wenn Server/Index nicht erreichbar (KEIN Fail — sonst kann der Marker nicht in CI laufen). 3-5 Smoke-Asserts gegen den ECHTEN Code-Pfad ohne Mocking. Tolerante Match-Logik (z.B. "P-Nummer >= aktuelle aus Doku" statt exakter String-Match — kein Test-Pflege-Aufwand nach jedem Sync).
- **Huginn-RAG-Smoketest** [`test_integration_huginn_rag.py`](zerberus/tests/test_integration_huginn_rag.py) — 5 Asserts gegen den echten `_huginn_rag_lookup`-Code-Pfad: `top_chunk["source"]` matcht `huginn_kennt_zerberus.md`, `top_chunk["content"]` enthält "Aktueller Stand", extrahierter Patch-String matcht `r"P\d{3}"`, Patch-Nummer >= 213 (tolerante Untergrenze), Anti-Halluzinations-Match dass kein "P178"-String im Top-Chunk steht.
- **Sync-Tool-Auto-Verifikation** [`tools/sync_huginn_rag.py`](tools/sync_huginn_rag.py): Pure-Function `build_integration_test_command(marker, keyword)` baut die pytest-CLI testbar. Subprocess-Wrapper `run_integration_test_check` mit Timeout + fail-soft auf pytest-Exit 5 (no tests collected = Warnung, nicht Block). Hook nach erfolgreichem HTTP-Sync: wenn Test fail, Sync-Exit-Code kippt auf 1.
- **Source-Audit-Tests in [`test_p213_pre_6_integration_framework.py`](zerberus/tests/test_p213_pre_6_integration_framework.py)** (17 Tests) schützen pytest.ini-Eintrag, conftest-Marker, Test-File-Existenz, Sync-Tool-Hook-Konstanten, CLAUDE_ZERBERUS-Regel-Eintraege.

**Was P213-pre-6 NICHT macht:** Multi-Server-Tests (nur localhost), CI-Runner-Setup, Per-Patch-Coverage-Metriken, automatische Test-Generierung.

**Lessons (5):**

1. **Mock-Tests = "der Code laeuft korrekt", Integration-Tests = "das System liefert das richtige Ergebnis".** Beides ist nötig, keiner ersetzt den anderen. Anti-Pattern aus P213-pre-Saga: vier Patches grün, echter Server halluziniert weiterhin.
2. **Pflicht-Frage vor jedem "Chris muss noch testen": kann ich das selbst testen?** Server starten, echten Call machen, echte Antwort prüfen — wenn JA, Test schreiben + ausführen, NICHT eskalieren. Legitime Eskalationen sind echtes Geräte / echtes Mikrofon / echter Telegram-Client / subjektive UX.
3. **Sync-Tool-Auto-Verifikation als Backstop.** HTTP-200 ist nicht "Daten korrekt", das ist nur "Bytes durch die Leitung". Bei jedem CLI-Tool das Daten-Drift verursachen kann (Sync, Migration, Restore), gehört ein Integration-Test-Hook hinten dran.
4. **Strukturelle Absicherung gegen Test-Erosion.** Source-Audit-Tests schützen pytest.ini + conftest + CLAUDE_ZERBERUS-Regel-Eintraege — wenn ein zukünftiger Coda das Framework rauseditiert, fällt mindestens ein Audit-Test.
5. **Server starten autonom mit `start_stable.bat`, NICHT `start.bat`.** Reload-Watcher zerschiesst beim Sync-Tool-Test den Server-State. `start_stable.bat` (P213-pre-4) ist die offizielle Variante für Coda-Sessions / Sync-Tool-Tests.

**Tests:** 17 in `test_p213_pre_6_integration_framework.py` + 5 in `test_integration_huginn_rag.py` (default-deselected, live-grün via `pytest -m integration -k huginn_rag`). Lokal: 2398 baseline (P213-pre-4) → **2419 passed** (+21 P213-pre-6).

---

## Vorheriger Patch

**Patch 213** — Reasoning-Schritte sichtbar im Chat (Phase 5a Ziel #13 ABGESCHLOSSEN) (2026-05-04)

Vor P213 sah der User waehrend einer Chat-Turn nur die finale Antwort. Wenn die Pipeline arbeitete (Spec-Probe → RAG → Veto → HitL-Wartezeit → Sandbox-Run → Synthese-Call), war das auf Mobile oft mehrere Sekunden Stille — der User wusste nicht, ob das System haengt oder arbeitet. P213 macht die Zwischenschritte sichtbar als kollabierte Karte unter der Bot-Bubble (Default eingeklappt, ein Klick zeigt die Liste).

**Architektur: Pure-Function-Schicht plus In-Memory-Buffer plus Audit-Tabelle plus auth-frei Long-Poll-Endpoint plus Verdrahtung in 7 Pipeline-Stellen plus Nala-Frontend-Karte mit 4s-Polling.**

- **Modul** [`zerberus/core/reasoning_steps.py`](zerberus/core/reasoning_steps.py):
  - **Pure-Function-Schicht** — `compute_step_duration_ms(started, finished) -> int|None` (None bei laufendem Step, sonst Millisekunden, clamped >= 0). `should_emit(kind, *, enabled, disabled_kinds)` als Trigger-Gate (globaler Kill-Switch + per-kind Opt-out + Whitelist `KNOWN_STEP_KINDS`={`spec_check`/`rag_query`/`veto_probe`/`hitl_wait`/`sandbox_run`/`synthesis`/`embedder`/`reranker`/`guard`/`llm_call`}). `truncate_text(text, *, max_bytes)` Bytes-genau mit Ellipsis-Marker `…` und Unicode-safe-Cut.
  - **`ReasoningStep`-Dataclass** — `step_id` (UUID4-hex), `session_id`, `kind`, `summary`, `started_at`, `status` (Default `running`, sonst `done`/`error`/`skipped`), `finished_at`, optional `detail`. `duration_ms` als computed property. `to_public_dict()` liefert das Frontend-JSON OHNE `session_id` und OHNE `detail` (Audit-only) — nur `step_id`/`kind`/`summary`/`status`/`duration_ms`/`started_at`/`finished_at`.
  - **`ReasoningStreamGate`-Singleton** — Per-Session-FIFO mit Default-Buffer `DEFAULT_BUFFER_PER_SESSION=32` und TTL `DEFAULT_SESSION_TTL_SECONDS=600`. `emit(*, session_id, kind, summary, detail) -> ReasoningStep|None` ist sync (kein await im Hot-Path). `mark_done(step_id, *, status, detail) -> ReasoningStep|None` ist idempotent (Doppel-Mark gewinnt der erste). `list_for_session(session_id)` als Snapshot. `cleanup_session(session_id)` + `cleanup_stale_sessions(*, now)` als TTL-Sweep. `consume_steps(session_id, *, wait_seconds)` async, blockt long-poll-style bis ein neuer Step kommt ODER der Timeout greift.
  - **Convenience** — `emit_step(session_id, kind, summary, *, detail)` und `mark_step_done(step, *, status, detail)`. `mark_step_done` akzeptiert sowohl ein `ReasoningStep`-Objekt als auch ein None — damit der Aufrufer unbedingt `mark_step_done(emit_step(...))` schreiben kann, wenn die Trigger-Gate-Heuristik den Step ablehnen wuerde.

- **Audit-Tabelle** `reasoning_audits` in [`database.py`](zerberus/core/database.py): `step_id`/`session_id`/`kind`/`status`/`duration_ms`/`summary`/`detail`/`created_at`. Nur bei finalem Status (`done`/`error`/`skipped`) geschrieben — `running`-Zwischenstand wandert nie in die DB. Best-Effort-Insert via `_audit_step()` als Hintergrund-Task auf dem laufenden Event-Loop. Auswertung: `SELECT kind, AVG(duration_ms), MAX(duration_ms), COUNT(*) FROM reasoning_audits WHERE status='done' GROUP BY kind` zeigt wo das System Zeit verbringt — Grundlage fuer Latenz-Tuning.

- **HTTP-Endpoints** in [`legacy.py`](zerberus/app/routers/legacy.py):
  - `GET /v1/reasoning/poll?wait=N` — auth-frei (Dictate-Lane-Invariante). Liefert `{"steps": [...]}` als JSON-Array fuer die Session aus `X-Session-ID`-Header. `wait` ist auf `[0, DEFAULT_POLL_TIMEOUT_SECONDS=10]` geclamped, um abusive Long-Polls zu verhindern. Best-Effort-Sweep aelterer Sessions bei jedem Poll.
  - `POST /v1/reasoning/clear` — auth-frei. Wirft die Steps der Session weg, idempotent. Frontend ruft das beim Beginn einer neuen Chat-Turn auf.

- **Verdrahtung in 7 Pipeline-Stellen** in `chat_completions` (legacy.py):
  1. **Turn-Reset** — Beim Eintritt der Funktion `get_reasoning_gate().cleanup_session(session_id)` als zusaetzliche Defensive (zusaetzlich zum Frontend-`POST /clear`).
  2. **Projekt-RAG** — `emit_step(session, "rag_query", "Projekt-RAG durchsucht (<slug>)")` vor `query_project_rag`, mark_done mit `chunks=N` als Detail.
  3. **Spec-Check** — `emit_step("spec_check", "Frage prueft auf Mehrdeutigkeit")` nur wenn `should_ask_clarification` greift, mark_done `done` falls Probe Frage liefert, sonst `skipped`.
  4. **LLM-Call** — `emit_step("llm_call", "Modell formuliert Antwort")` mit try/except — mark_done `error` falls die Call-Coroutine wirft. Auch im Fallback-Pfad und im Direct-LLM-Pfad.
  5. **Veto-Probe** — `emit_step("veto_probe", "Zweites Modell prueft Code")` vor `run_veto`, mark_done abhaengig vom Verdict (`pass`/`veto`/`error`).
  6. **HitL-Wartezeit** — `emit_step("hitl_wait", "Wartet auf Bestaetigung")` vor `wait_for_decision`, Status-Mapping `approved`/`bypassed` → `done`, `rejected`/`timeout` → `skipped`, sonst `error`.
  7. **Sandbox-Run** — `emit_step("sandbox_run", "Sandbox laeuft (<lang>)")` vor `execute_in_workspace`, mark_done `done` bei `exit_code=0`, sonst `error`. `skipped` falls `execute_in_workspace` None liefert (Slug-Reject/Disabled).
  8. **Synthese** — `emit_step("synthesis", "Verstaendliche Antwort wird formuliert")` vor `synthesize_code_output`, mark_done `done`/`skipped` (leerer Output)/`error`.

- **Nala-Frontend** in [`nala.py`](zerberus/app/routers/nala.py):
  - **CSS** `.reasoning-card` mit Default-collapsed-State, `.reasoning-toggle` als 44px-Touch-Target (Mobile-first), `.reasoning-list` als ungeordnete Liste mit `.reasoning-step`-Eintraegen. Status-Klassen `.reasoning-running` (animiertes Spin-Icon), `.reasoning-done` (Gruen), `.reasoning-error` (Rot), `.reasoning-skipped` (Grau).
  - **Polling-Loop** `startReasoningPolling(abortSignal, snapshotSessionId)` — 4s-Intervall, erster Tick nach 800ms, fail-quiet bei Netzfehler, Auto-Stop wenn die Session wechselt. Endpoint `GET /v1/reasoning/poll`.
  - **Renderer** `renderReasoningSteps(steps)` — Karte wird beim ersten Step erzeugt, danach nur in-place Update der Eintraege via `_reasoningStepIndex`-Map (step_id → li-Element). Status-Icons `⏳`/`✅`/`❌`/`⏭` via `escapeHtml(_reasonIcon(...))`. Summary via `textContent` (XSS-safe, kein innerHTML auf User/LLM-Strings). Per-Step `data-step-id` + `data-step-kind` als stabile DOM-Keys.
  - **Toggle** als globale Event-Delegation auf `[data-reasoning-toggle]` (kein onclick-Concat im innerHTML, P203b-Invariante). Erster Klick expandiert die Liste, zweiter Klick kollabiert sie.
  - **Default-Labels** `_REASON_KIND_LABELS` als Frontend-Whitelist — falls das Backend einen unbekannten Kind liefert, faellt der Renderer auf den Roh-Namen zurueck.
  - **Hookup an `sendMessage`** — `clearReasoningState()` + `startReasoningPolling(...)` parallel zu HitL/Spec-Polling. `stopReasoningPolling()` im `finally`-Block (Karte bleibt aber als Audit-Spur stehen).

**Was P213 bewusst NICHT macht:**

- **SSE/WebSocket-Streaming** — 4s-Polling reicht. Der Polling-Cost ist bei einer Chat-Turn-Lifetime vernachlaessigbar (3-5 Polls pro Turn), und SSE wuerde die Auth-frei-Endpoint-Konvention durchbrechen.
- **Cross-Session-Visibility** — jeder User sieht nur seine eigenen Steps (Filter via `X-Session-ID`-Header).
- **Detail-Drill-Down pro Step** — der `detail`-String steht in der Audit-Tabelle, nicht in der Karten-UI. Wer wissen will warum ein Step `error` war, schaut in `reasoning_audits.detail`.
- **Replay-Modus fuer alte Sessions** — Steps sind transient (In-Memory + TTL-Sweep). Wer historische Reasoning-Spuren braucht, fragt die Audit-Tabelle direkt.
- **Hel-UI-Auswertung** — eigener Patch P213b, analog zu den anderen Audit-Tabellen-Schulden (P211/P212).

**Lessons (3):**

1. **Sync-emit + async-audit ist die richtige Asymmetrie.** `emit_step` ist sync, weil der Hot-Path nicht blocken darf — der Step wird in den In-Memory-Buffer geschoben und der Caller faehrt sofort weiter. Der Audit-Insert ist asynchron als Hintergrund-Task auf dem laufenden Event-Loop — wenn kein Loop verfuegbar ist (Sync-Test), wird die Coroutine gar nicht erst erzeugt (sonst RuntimeWarning "coroutine was never awaited"). Pattern fuer alle zukuenftigen Telemetrie-Pfade.
2. **`mark_step_done(emit_step(...))` mit None-tolerantem Caller-Pattern.** Wenn das Trigger-Gate emit_step ablehnt (unbekannter kind, fehlende session_id), liefert es None — und `mark_step_done(None)` ist ein No-Op. Der Aufrufer kann unbedingt schreiben `_step = emit_step(...); ...; mark_step_done(_step)` ohne Null-Check — das macht die Verdrahtung in 7 Pipeline-Stellen kurz und fehlertolerant.
3. **Window-basierte Source-Audits brauchen Reserve.** Der P203d-Test `test_writable_false_default_in_call_site` hat ein ±2500-Byte-Fenster um `[SANDBOX-203d]` benutzt — P213 hat im Sandbox-Block ~700 Bytes emit_step-Code zugefuegt, das Fenster gerissen. Lehre fuer zukuenftige Source-Audits: ±5000 als konservative Default-Reserve, oder lieber zwei Substring-Checks ohne Window (z.B. "beide Strings im File"), als enge Distance-Tests, die bei jedem Verdrahtungs-Patch reissen.

**Tests:** 57 in [`test_p213_reasoning_steps.py`](zerberus/tests/test_p213_reasoning_steps.py).

`TestComputeStepDurationMs` (4) — running / finished / zero / negative-clamped.
`TestShouldEmit` (4) — known-kinds / unknown / disabled-globally / disabled-kinds.
`TestTruncateText` (4) — none / short / truncate-with-ellipsis / unicode-safe.
`TestReasoningStep` (3) — running-no-duration / finished-duration / public-dict-omits-internal.
`TestStreamGateEmit` (4) — running-step / unknown-kind / no-session / truncated.
`TestStreamGateMarkDone` (4) — sets-finish / idempotent / invalid-status / unknown-id.
`TestStreamGateBufferCap` (1) — fifo-cap-drops-oldest.
`TestStreamGateCleanup` (2) — cleanup-session / cleanup-stale.
`TestStreamGateConsume` (4) — immediate / empty / long-poll-emit / long-poll-timeout.
`TestConvenience` (3) — singleton-emit / mark-none-no-crash / mark-step-object.
`TestStoreAudit` (1) — silent-when-DB-not-initialized.
`TestLegacyWiring` (4) — imports / kinds-emitted / turn-reset / endpoints-registered.
`TestReasoningPollEndpoint` (4) — empty / steps-for-session / session-isolation / wait-clamped.
`TestReasoningClearEndpoint` (2) — clear-removes / idempotent.
`TestNalaFrontendReasoningCard` (7) — css / 44px / poll-endpoint / clear-endpoint / hookup / event-delegation / xss-textContent.
`TestJsSyntaxIntegrity` (1) — node --check ueber inline scripts (skipped wenn node fehlt).
`TestSmoke` (4) — module-exports / db-schema / kinds-statuses-disjoint / constants-sane.

Lokal: 2256 baseline (P212) → **2313 passed** (+57 P213, +0 NEUE Failures), 4 xfailed pre-existing, 3 failed pre-existing aus Schuldenliste (`edge-tts` + `test_rag_dual_switch.test_fallback_logic` + `test_patch185_runtime_info` durch `config.yaml`-Drift `deepseek-v4-pro`).

---

## Vorheriger Patch

**Patch 212** — Secrets bleiben geheim / Output-Maskierung für Sandbox+Synthese (Phase 5a Ziel #12 ABGESCHLOSSEN) (2026-05-04)

Vor P212 konnte ein vom Haupt-LLM generierter Code-Block, der absichtlich oder versehentlich `os.environ['OPENAI_API_KEY']` ausführte, den Klartext-Schlüssel via `code_execution.stdout` an's Frontend zurückliefern UND den Synthese-LLM dazu bringen, den Schlüssel in seiner menschenlesbaren Antwort zu wiederholen — der User hätte den `sk-…`-Wert in der Chat-Bubble gelesen. P212 baut zwei Defense-in-Depth-Schichten: die Sandbox maskiert ihren stdout/stderr-Output BEVOR der Caller ihn sieht, und die Output-Synthese maskiert das `code_execution`-Dict NOCHMAL bevor es dem Synthese-LLM in den Prompt gegeben wird.

**Architektur: Pure-Function-Schicht plus Cache plus Convenience-Async-Wrapper plus Audit-Tabelle plus Verdrahtung in Sandbox+Synthese.**

- **Modul** [`zerberus/core/secrets_filter.py`](zerberus/core/secrets_filter.py):
  - **Pure-Function-Heuristik** — `is_secret_key(name)` prüft case-insensitive gegen die Whitelist `SECRET_KEY_NAMES` (`OPENAI_API_KEY`, `OPENROUTER_API_KEY`, `ANTHROPIC_API_KEY`, `TELEGRAM_BOT_TOKEN`, `JWT_SECRET`, `DATABASE_URL`, plus die Generika `PASSWORD`/`TOKEN`/`SECRET`/`API_KEY`), die Suffix-Tabelle `SECRET_KEY_SUFFIXES` (`_KEY`, `_SECRET`, `_TOKEN`, `_PASSWORD`, `_PASS`, `_PASSPHRASE`, `_CREDENTIAL`, `_CREDENTIALS`) und die Prefix-Tabelle `SECRET_KEY_PREFIXES` (`API_`, `AUTH_`).
  - **Wert-Extraktion** — `extract_secret_values(env_dict, *, min_length=MIN_SECRET_LENGTH=8)` sammelt alle Werte mit Secret-Indikator im Key und filtert Werte unter 8 Zeichen heraus (Falsch-Positive durch z.B. `DEBUG_TOKEN=1` vermeiden — echte API-Keys sind in der Regel ≥ 20 Zeichen).
  - **Maskierung** — `mask_secrets_in_text(text, secrets, *, replacement="***REDACTED***") -> tuple[str, int]` mit zwei Invarianten: (a) **longest-first-Sortierung** damit ein Secret `OPENAI_API_KEY` nicht zu `***REDACTED***_KEY` ueberlebt, weil ein anderes Secret `OPENAI_API` zuerst gematched wuerde, (b) **Replacement-Skip** wenn ein Secret zufaellig dem Replacement-Text gleicht (verhindert Endlosschleifen + macht die Maskierung idempotent).
  - **Cache-Schicht** — `load_secret_values(*, env=None, force_reload=False) -> frozenset[str]`. Lazy-cached: erster Aufruf liest `os.environ` (oder das gegebene `env`-Dict), extrahiert die Secret-Werte, friert sie als `frozenset` ein. Folgende Aufrufe geben den Cache zurück — env-Aenderungen zur Laufzeit werden NICHT beobachtet (gewollt: `.env` wird einmal beim Start geladen, danach ist die Liste stabil). `reset_cache_for_tests()` als Test-Hatch.
  - **Convenience** — `mask_and_audit(text, *, source, session_id=None) -> str` async wrapper: lädt Secrets, maskiert, schreibt Audit-Eintrag (nur wenn count > 0), gibt maskierten Text zurück. Fail-open: jeder Maskierungs-Fehler wird geloggt + verschluckt, Caller bekommt im Worst-Case den Original-Text. `mask_and_audit_sync(text, *, source) -> tuple[str, int]` für Pfade ausserhalb des asyncio-Loops oder pure Maskierung ohne Audit.

- **Audit-Tabelle** `secret_redactions` in [`database.py`](zerberus/core/database.py): `redaction_count`/`source` (sandbox|synthesis)/`session_id`/`created_at`. Best-Effort-Insert via `store_secret_redaction()` — Eintraege mit `redaction_count <= 0` werden NICHT geschrieben (waren leeres Rauschen), DB-Fehler werden geschluckt. **Sinn der Tabelle:** wenn jemals ein Klartext-Secret im Output landen wuerde, ist das ein Bug-Indikator. Die P212-Maskierung faengt es ab — der Audit-Eintrag macht das Leak nachweisbar, damit es spaeter im Code geschlossen werden kann.

- **Verdrahtung in zwei Stellen** (Defense-in-Depth):
  1. [`sandbox/manager.py._run_in_container`](zerberus/modules/sandbox/manager.py) — Direkt nach dem `decode("utf-8", errors="replace")`-Schritt (also VOR `_truncate`) werden `stdout_text` und `stderr_text` durch `await mask_and_audit(text, source="sandbox")` gejagt. Kein `session_id`, weil der `SandboxManager` die Chat-Session nicht kennt. Wenn Sandbox-Code stdout-Output mit Klartext-Secret produziert, sieht der Caller bereits maskierte Werte — Frontend, Synthese und alle weiteren Verbraucher kriegen automatisch die saubere Variante.
  2. [`sandbox/synthesis.py.synthesize_code_output`](zerberus/modules/sandbox/synthesis.py) — Bevor `llm_service.call(messages, session_id)` aufgerufen wird, werden die drei Felder `payload["code"]`, `payload["stdout"]` und `payload["stderr"]` in-place auf dem payload-Dict durch `await mask_and_audit(val, source="synthesis", session_id=session_id)` ersetzt. Wenn die Sandbox-Maskierung perfekt war, ist count=0 und es wird kein zweiter Audit geschrieben — die zweite Schicht ist Zero-Cost-Defense-in-Depth. Wenn jemand zukünftig die Sandbox-Side bypassed (neue Output-Quelle, anderer Caller-Pfad), greift die Synthese-Schicht trotzdem und stoppt das Leak.

**Was P212 bewusst NICHT macht:**

- **`.env`-Verschlüsselung** — eigener Patch P212b oder später. Hier geht es nur um Output-Maskierung, nicht um Storage-Verschlüsselung.
- **Pattern-basiertes Matching ohne env-Lookup** (z.B. `sk-…`-Prefix-Heuristik). Wenn ein Secret nicht in `.env` steht, kennen wir es nicht — Defense-in-Depth über zusätzliche Pattern wäre ein eigener Patch.
- **Multi-Pass-LLM-Filter** — eine Maskierung am Output reicht. Der Synthese-LLM sieht den Klartext-Secret nie, weil das payload vor dem LLM-Call gewaschen wird.
- **Container-Isolation** — P171/P203c reicht. Die Sandbox läuft mit `--network none` und ohne Host-env-Mount, der Code sollte den Schlüssel eigentlich gar nicht erst sehen.
- **Dynamische `.env`-Reloads** — `load_secret_values` cached einmalig. Das ist gewollt: `.env` wird beim Server-Start geladen, danach ist die Liste stabil. Wer eine `.env`-Änderung will, restartet den Server.
- **Sync-Variante des Maskierens innerhalb von `mask_and_audit`** — `mask_and_audit_sync` existiert für reine Pure-Function-Maskierung ohne Audit, der async-Pfad bleibt der primäre Eintritt.

**Lessons (5):**

1. **Defense-in-Depth ist Zero-Cost wenn die Pure-Function-Schicht klein ist.** `mask_secrets_in_text` ist 15 Zeilen, `load_secret_values` cached die Secrets im `frozenset`. Eine zweite Maskierungs-Schicht in der Synthese kostet einen O(n)-Scan über drei kleine Strings. Das ist billig genug, dass die Defense-in-Depth-Logik trivial in den Pfad zu integrieren ist — kein Performance-Argument gegen "doppelt maskieren".
2. **Longest-first-Replace ist die nicht-offensichtliche Invariante.** Ein naiver `text.replace(secret, replacement)`-Loop in zufaelliger Reihenfolge zerschiesst lange Secrets, wenn deren Prefix auch in der Set ist. Die Tabelle muss VOR jedem Replace-Loop nach Laenge absteigend sortiert werden — das ist eine Zeile Code, aber wenn man sie vergisst, gibt es subtile Bugs (`OPENAI_API_KEY` → `***REDACTED***_KEY`).
3. **Cache mit Test-Hatch ist die richtige Antwort auf "load .env once".** Ein dynamisches `os.environ`-Re-Read bei jedem Aufruf wäre Overhead und unnötig — `.env` ändert sich zur Laufzeit nicht. Aber Tests müssen den Cache invalidieren können, sonst sind sie order-dependent. `reset_cache_for_tests()` + `force_reload=True` + `env=...`-Override löst alle drei Test-Bedürfnisse.
4. **Audit nur bei count > 0 schreiben — sonst wird die Tabelle leeres Rauschen.** 99% aller `mask_and_audit`-Aufrufe finden nichts (kein Secret im Output). Wenn jeder Aufruf eine Zeile schreiben würde, wäre die Tabelle nutzlos für die eigentliche Diagnose-Aufgabe ("welcher Pfad blutet?"). Ein `count <= 0`-Skip am Anfang von `store_secret_redaction()` reicht.
5. **MIN_SECRET_LENGTH=8 fängt die Falsch-Positive durch leere/short-Werte ab.** `.env` enthält oft `DEBUG=1` oder `LOG_LEVEL=INFO` — wenn der Filter diese mitnehmen würde (weil `LEVEL` zufällig ein Suffix-Match wäre), würde jeder `INFO`-String im Output zu `***REDACTED***` werden. Der Min-Length-Filter ist die billige Heuristik, die das verhindert: echte Secrets sind ≥ 20 Zeichen, alles unter 8 ist mit hoher Wahrscheinlichkeit Konfiguration, kein Geheimnis.

**Tests:** 40 in [`test_p212_secrets_filter.py`](zerberus/tests/test_p212_secrets_filter.py).

`TestIsSecretKey` (5) — known-names / suffixes / prefixes / empty-or-none / non-secret.
`TestExtractSecretValues` (5) — empty-dict / mixed / short-filtered / min-length-override / empty-value.
`TestMaskSecretsInText` (8) — empty-text / no-secrets / single / multiple-occurrences-counted / longest-first-invariant / empty-secret-skipped / replacement-not-re-replaced / custom-replacement.
`TestLoadSecretValues` (3) — uses-custom-env / cache-snapshot / force-reload-picks-up-new-env.
`TestStoreAudit` (2) — zero-count-skips-insert / silent-when-DB-not-initialized.
`TestMaskAndAudit` (4) — no-secrets-no-change / with-secret-replaces-and-audits / empty-text-returns-empty / fail-open-on-load-error.
`TestMaskAndAuditSync` (2) — no-secrets-zero-count / with-secret-replaces.
Verdrahtungs-Source-Audits (5) — Sandbox-Imports + sandbox-source-Aufruf, Synthese-Imports + synthesis-source-Aufruf + Pre-LLM-Order-Invariante.
End-to-End (3) — payload-secrets-masked-before-llm-sees-them / synthesis-skips-when-no-payload / sandbox-mask-chain-replaces-secrets.
`TestSmoke` (3) — Module-Exports / DB-Tabelle-Schema / Konstanten-Konsistenz.

Lokal: 2216 baseline (P211) → **2256 passed** (+40 P212), 4 xfailed pre-existing, 3 failed pre-existing aus Schuldenliste (`edge-tts` + `test_rag_dual_switch.test_fallback_logic` + `test_patch185_runtime_info` durch `config.yaml`-Drift `deepseek-v4-pro`), 0 NEUE Failures aus P212.

---

## Vorheriger Patch

**Patch 211** — GPU-Queue für VRAM-Konsumenten / Kooperatives Scheduling (Phase 5a Ziel #11 ABGESCHLOSSEN) (2026-05-03)

Vor P211 liefen Whisper (etwa 4 GB FP16), Gemma E2B (etwa 2 GB), der RAG-Embedder (bis 2 GB für E5-Large) und der Cross-Encoder-Reranker (etwa 512 MB) auf der RTX 3060 (12 GB) ohne Koordination — bei einer typischen Voice-Eingabe, die parallel Whisper, Gemma und den Embedder triggerte, konnte die Karte überlaufen und der Server crashte mit `CUDA out of memory`. P211 baut einen kooperativen Token-Bucket: jeder Konsument reserviert vor dem Modell-Aufruf seinen statischen VRAM-Budget-Anteil; passt das in das Restbudget (Default 11 GB nutzbar), läuft der Call sofort, sonst wandert er in eine FIFO-Queue.

**Architektur: Pure-Function-Schicht plus globaler async Singleton plus Audit-Tabelle plus Status-Endpoint plus Hel-Frontend-Toast.**

- **Modul** [`zerberus/core/gpu_queue.py`](zerberus/core/gpu_queue.py):
  - **Pure-Function-Schicht** — `compute_vram_budget(consumer_name) -> int` mit statischer Tabelle `VRAM_BUDGET_MB = {whisper:4000, gemma:2000, embedder:1000, reranker:512}`; unbekannte Konsumenten bekommen `DEFAULT_CONSUMER_BUDGET_MB=1500` mit Warning-Log. `should_queue(active_total_mb, requested_mb, *, total_mb=11000) -> bool` als Token-Bucket-Check.
  - **GpuSlotInfo-Dataclass** — `audit_id` (UUID4-hex), `consumer_name`, `requested_mb`, `queue_position` (0=sofort, >0=N-ter Waiter), `waited_at`/`acquired_at`/`released_at`, `timed_out`, plus computed Properties `wait_ms` und `held_ms`.
  - **GpuQueue-Singleton** — globale Instanz pro Prozess, lazy via `get_gpu_queue()`. State: `_active_mb` (aktuell belegt), `_waiters` (FIFO-Liste mit `(consumer_name, requested_mb, asyncio.Future)`), `_lock` (asyncio.Lock), `_active_slots` (fuer Status-Snapshot).
  - **Acquire-Logik unter Lock**: passt das Request → sofort `_active_mb += requested`, Slot zurueck. Sonst Future in `_waiters` anhaengen, ausserhalb des Locks `await asyncio.wait_for(future, timeout)`. Bei Timeout sauber Cleanup (Waiter-Liste filtern, kein Leak).
  - **Release-Logik**: `_active_mb -= requested`, dann FIFO durch die Waiter-Liste, jeden Waiter dessen Request reinpasst wecken. Head-of-Line-Block by design — der erste Waiter blockt alle dahinter, auch wenn ein spaeterer reinpassen wuerde (akzeptabel: typischer Workload selten >2-3 parallele Konsumenten).
  - **Convenience** — `vram_slot(consumer_name, *, timeout=30.0)` als async Context-Manager: `async with vram_slot("whisper"): ...`. Identisch zu `get_gpu_queue().slot(...)`, kuerzer fuer Verdrahtung.

- **Audit-Tabelle** `gpu_queue_audits` in [`database.py`](zerberus/core/database.py): `audit_id`/`consumer_name`/`requested_mb`/`queue_position`/`wait_ms`/`held_ms`/`timed_out`/`created_at`. Best-Effort-Insert via `store_gpu_queue_audit(info)` — DB-Fehler verschluckt, Hauptpfad blockiert nicht.

- **Verdrahtung in fuenf Stellen**:
  1. [`whisper_client.transcribe`](zerberus/utils/whisper_client.py) — `async with vram_slot("whisper", timeout=60.0)` um den HTTP-Call.
  2. [`gemma_client.GemmaAudioClient.analyze_audio`](zerberus/modules/prosody/gemma_client.py) — `vram_slot("gemma", timeout=60.0)` um den Backend-Call (CLI oder Server). Stub-Pfad skippt den Slot.
  3. [`projects_rag.index_project_file`](zerberus/core/projects_rag.py) und `query_project_rag` — `vram_slot("embedder", timeout=30.0)` um den `_embed_text`-Aufruf. Bei Index: ein Slot fuer den ganzen Chunk-Loop einer Datei (vermeidet Per-Chunk-Overhead). Timeout-Reason `embed_timeout`.
  4. [`rag/router.py`](zerberus/modules/rag/router.py) `query_documents`-Endpoint — `vram_slot("embedder")` um den `_encode`-Loop und `vram_slot("reranker")` um den `_rerank`-Call.
  5. [`orchestrator.py`](zerberus/app/routers/orchestrator.py) — Embedder-Loop und Reranker-Call analog.

- **Status-Endpoint** `GET /v1/gpu/status` (auth-frei wie `/v1/hitl/*`) liefert Snapshot `{total_mb, active_mb, free_mb, active_slots:[{consumer,requested_mb,held_ms}], waiters:[{consumer,requested_mb}]}`.

- **Hel-Frontend-Toast** ([`hel.py`](zerberus/app/routers/hel.py)): CSS `.gpu-toast` (analog `.rag-toast` aus P205, fixed bottom-LEFT statt right damit beide Toasts nicht kollidieren, gleiche Kintsugi-Border, 44px-Touch-Target, fade-in via `.visible`-Klasse). DOM-Element `<div id="gpuToast" ...>`. JS-IIFE mit `_pollOnce()` (fetch `/v1/gpu/status` no-store, fail-quiet) plus `_renderGpuToast(status)` (textContent statt innerHTML — Consumer-Namen aus Whitelist `whisper|gemma|embedder|reranker`, XSS-immun). Toast erscheint sobald `waiters.length > 0`: `⏳ GPU wartet auf <Consumer>` plus Position-Anzeige bei mehreren Wartenden. Polling alle 4s via `setInterval`, sauberer Cleanup beim `beforeunload`. Klick = Soft-Dismiss.

**Was P211 bewusst NICHT macht:**

- **Dynamische VRAM-Erkennung via nvidia-smi** — statisches Budget reicht, ist robust gegen Treiber-Anomalien und tut auch ohne CUDA-Header.
- **Mehr-GPU-Verteilung** — genug fuer eine RTX 3060, Multi-GPU waere fuer P212+.
- **Cancel-Outstanding-Slots** — FIFO bleibt stur; bei Timeout wird der Caller selbst entfernt, aber andere Waiters laufen weiter.
- **Per-Consumer-Lock-Isolation** — globale Queue ist einfacher und ausreichend; Per-Consumer-Lock wuerde Token-Spar verspielen.
- **Sync-Variante des Slots** fuer Code der nicht in async lebt — alle vier Konsumenten sind erreichbar via async-Pfade.
- **Toast im Nala-Frontend** — Hel reicht; Nala-User sieht den Slot-Wait via Antwortzeit, nicht via Toast.

**Lessons (5):**

1. **Statisches VRAM-Budget schlaegt dynamische Detection.** `nvidia-smi`-Polling ist unzuverlaessig (Treiber-Caching, Race-Conditions zwischen Read und Allocate) und schafft eine Abhaengigkeit zu CUDA-Headern. Eine statische Tabelle aus produktiven Messungen plus ein konservativer 11-von-12-GB-Budget liefert die gleiche Schutzwirkung mit 30 Zeilen Code statt 200.
2. **FIFO mit Head-of-Line-Block ist der Sweet-Spot.** Strikte FIFO-Reihenfolge ist trivial zu implementieren und vorhersehbar. "Smarter" Scheduler (best-fit, priority-queue) sind Overkill bei 4 Konsumenten und 2-3 typischer Parallelitaet — und schwerer zu testen.
3. **Pure-Function `should_queue` macht den Token-Bucket testbar ohne asyncio.** `should_queue(active, requested, total) -> bool` ist eine 3-Zeilen-Funktion, aber sie isoliert die Boundary-Logik (`<=` vs. `<` an der Total-Grenze) und macht einen Property-Test trivial. Lesson aus P208/P209 auf Async-Resource-Locking uebertragen.
4. **Audit-Tabelle ist die Validierung der Hypothese.** Die Budget-Werte (Whisper=4 GB, Gemma=2 GB, Embedder=1 GB, Reranker=512 MB) sind Schaetzungen. `gpu_queue_audits` mit `wait_ms`/`held_ms`/`queue_position`/`timed_out` sammelt die Realdaten — nach 1-2 Wochen Betrieb laesst sich aus `SELECT consumer_name, AVG(wait_ms) FROM gpu_queue_audits GROUP BY consumer_name` direkt sehen, wo die Annahmen daneben liegen.
5. **Hel-Toast statt Nala-Toast ist die richtige Trennung.** Nala-User sieht den Slot-Wait durch laengere Antwortzeit; das System-Status-Detail gehoert ins Admin-Frontend, wo es Operations-Wert hat ("welcher Konsument staut sich gerade?"), nicht ins User-Frontend, wo es nur Verwirrung stiftet ("warum sehe ich GPU-Konsumenten?").

**Tests:** 40 in [`test_p211_gpu_queue.py`](zerberus/tests/test_p211_gpu_queue.py).

`TestComputeVramBudget` (3) — known-consumers / unknown-default / case-insensitive.
`TestShouldQueue` (3) — fits / overflow / zero-or-negative.
`TestGpuQueueAcquireImmediate` (3) — when-empty / release-frees / parallel-3-consumers.
`TestGpuQueueFifo` (2) — overflow-waits / strict-fifo-order.
`TestGpuQueueTimeout` (2) — raises-TimeoutError / cleans-waiter.
`TestGpuQueueStatus` (1) — reports-active+waiters.
`TestStoreAudit` (1) — silent-when-no-DB.
Verdrahtungs-Source-Audits (10) — Whisper/Gemma/Embedder/Reranker je Imports + Aufruf-Stelle.
`TestGpuStatusEndpointSource` (2) + `TestGpuStatusEndpointE2E` (2) — Endpoint-Registration und Snapshot-Schema.
`TestHelFrontendGpuToast` (6) — CSS-Klasse / DOM / Endpoint-URL / textContent / Whitelist / 44px.
`TestJsSyntaxIntegrity` (1) — `node --check` ueber alle inline `<script>`-Bloecke aus ADMIN_HTML, skipped wenn `node` nicht im PATH.
`TestSmoke` (3) — Module-Exports / DB-Tabelle-Schema / KNOWN_CONSUMERS-Konsistenz.

Lokal: 2176 baseline → **2216 passed** (+40 P211), 4 xfailed pre-existing, 3 failed pre-existing aus Schuldenliste (`edge-tts` + `test_rag_dual_switch.test_fallback_logic` + `test_patch185_runtime_info` durch `config.yaml`-Drift `deepseek-v4-pro`), 0 NEUE Failures aus P211.

---

## Vorheriger Patch

**Patch 210** — Huginn-RAG-Auto-Sync / Selbstwissen-Aktualität (Phase 5a Ziel #18 ABGESCHLOSSEN) (2026-05-03)

User-Pain-Patch ausserhalb der ursprünglichen 17 Phase-5a-Ziele eingeschoben. Huginn antwortete konsistent "Patch 178" auf "bei welchem Patch sind wir?", obwohl die Doku längst auf P209 stand. Diagnose ergab zwei Ursachen: (1) `docs/huginn_kennt_zerberus.md` hatte keinen expliziten Stand-Anker — Huginn rät auf prominente Logging-Tags, der häufigste war `[HUGINN-178]` (Patch 178 = Huginn-RAG-Lookup-Feature selbst), (2) Index-Mtime-Drift — Coda updated die Doku, aber Chris musste manuell hochladen und vergaß es regelmässig. P210 fixt beide Ursachen: Doku-Header bekommt `## Aktueller Stand`-Block (immer als erster Chunk indexiert, rankt bei Stand-Fragen oben), Auto-Sync-Skript löscht alte Chunks + lädt neue automatisch, eingebaut in den Marathon-Push-Zyklus.

**Architektur: Pure-Function-Schicht plus Async-Wrapper plus CLI plus PowerShell-Wrapper.**

- **Sync-Modul** [`tools/sync_huginn_rag.py`](tools/sync_huginn_rag.py) mit:
  - **Pure-Function** — `build_sync_plan(source_path, *, source_name, category, run_reindex)` plant 2 oder 3 Steps (DELETE → POST [→ REINDEX]); `validate_doc_header(text) -> (bool, str)` prüft Pflicht-Block; `extract_current_patch(text) -> Optional[str]` Diagnose-Helper liest "P###"; `parse_auth_string(raw)` / `load_auth_from_env(env, env_file_path)` / `resolve_base_url(env)` injectable für Tests.
  - **Async-Wrapper** — `execute_sync_plan(plan, base_url, *, auth, http_client)` mit `httpx.AsyncClient`, fail-soft bei DELETE-404 (Erst-Upload-Idempotenz), fail-fast bei UPLOAD-Fehler, Exceptions werden als `errors`-Liste eingesammelt statt zu propagieren.
  - **SyncStep / SyncResult-Dataclasses** — `SyncStep(method, path, params, files, data, success_codes, description)`, `SyncResult(success, steps_executed, steps_failed, errors, response_payloads)`.
  - **CLI** — `python -m tools.sync_huginn_rag` mit `--source`, `--source-name`, `--category`, `--base-url`, `--reindex`, `--env-file`, `--dry-run`. Exit-Codes: 0=ok, 1=Sync-Fehler, 2=Plan-Fehler.
- **Reihenfolge-Invariante** — DELETE vor UPLOAD. Würde man umkehren, würde DELETE die gerade hochgeladenen neuen Chunks soft-löschen (Off-by-One-Fail). Test `test_delete_before_upload` schützt explizit.
- **PowerShell-Wrapper** [`scripts/sync_huginn_rag.ps1`](scripts/sync_huginn_rag.ps1) — analog `verify_sync.ps1`. Setzt CWD auf Repo-Root, ruft Python-Modul, reicht Exit-Code durch. Switches `-Reindex`, `-DryRun`, `-Source`, `-BaseUrl` als Pass-Through.
- **Doku-Header-Pflicht** — neuer `## Aktueller Stand`-Block in [`docs/huginn_kennt_zerberus.md`](docs/huginn_kennt_zerberus.md) und Spiegel-Kopie [`docs/RAG Testdokumente/huginn_kennt_zerberus.md`](docs/RAG%20Testdokumente/huginn_kennt_zerberus.md). Vier Pflicht-Bullets: Letzter Patch, Phase, Tests, Datum. Source-Audit-Tests prüfen Existenz und P##-Mindestnummer.
- **WORKFLOW.md erweitert** — neue Doku-Pflicht-Tabellenzeile "RAG-Sync für `huginn_kennt_zerberus.md`", neuer ausführlicher Regel-Block (Pflicht-Header, Sync-Skript-Aufruf, Auth via Env-Var, Spiegel-Kopie nicht mitsynct), neues Phase-5a-Ziel #18 "Huginn kennt sich selbst zuverlässig" (✅ markiert).
- **Auth via Env-Var** — `HUGINN_RAG_AUTH=User:Pass` aus `os.environ` schlägt `.env`-Datei. Server-URL via `ZERBERUS_URL` (Default `http://localhost:5000`). Beides optional — bei dry-run irrelevant, bei live-run notwendig.
- **Endpoints** — wiederverwendet aus Hel-Admin-Router: `DELETE /hel/admin/rag/document?source=<name>` (P116 Soft-Delete), `POST /hel/admin/rag/upload` (P108 + P111 mit Auto-Detect — explizit `category=system` gesetzt), `POST /hel/admin/rag/reindex` (Default OFF — Soft-Delete reicht für Lookup, Reindex spart eigentlich nur Disk).

**Was P210 bewusst NICHT macht:**

- **Spiegel-Kopie mitsync** — die Test-Set-Variante unter `docs/RAG Testdokumente/` ist nicht im Live-RAG, dient nur als RAG-Test-Material. Sync-Skript synct sie NICHT, der Stand-Anker wird aber parallel mitgepflegt (Doku-Pflicht).
- **Auto-Trigger nach git commit** — kein Hook in `.git/hooks/`. Stattdessen explizit im Marathon-Push-Zyklus dokumentiert (vor `sync_repos.ps1`). Hook würde User-side-effect-Magie erzeugen.
- **Multi-Datei-Sync** — fokussiert auf `huginn_kennt_zerberus.md`. Wenn weitere Doku-Files in den RAG sollen, muss der Plan-Builder erweitert werden (trivial: Liste statt einzelner Pfad).
- **Server-Status-Check vorab** — wenn der Server down ist, schlägt der erste HTTP-Call eh fehl. Kein separater Health-Probe.
- **Cron / Schedule-Integration** — Coda macht das am Session-Ende, nicht zeitgesteuert. Wenn der Server gerade nicht läuft, bleibt der Index alt — der nächste manuelle Sync-Aufruf fixt es.
- **Auto-Bumping der Patchnummer im Header** — Coda muss den Header bewusst aktualisieren (Doku-Pflicht). Sonst würde der Stand-Anker mechanisch jede gewünschte Nummer kriegen — die Pflege-Disziplin ist sicherer.

**Tests:** 53 in [`test_p210_huginn_rag_sync.py`](zerberus/tests/test_p210_huginn_rag_sync.py). Strukturiert in 9 Klassen: `TestBuildSyncPlan` (11 — default-2-steps/delete-source-param/delete-accepts-404/upload-only-200/upload-category/upload-file/reindex-optional/missing-file/missing-header/missing-patch/delete-before-upload), `TestValidateDocHeader` (5 — valid/empty/missing-header/missing-patch/header-mid-file), `TestExtractCurrentPatch` (4 — p210/uppercase/no-match/3-or-4-digit), `TestParseAuthString` (5 — basic/colon-in-password/empty/no-colon/empty-user), `TestLoadAuthFromEnv` (5 — env-var/no-source/file/env-beats-file/quoted-value), `TestResolveBaseUrl` (3 — default/from-env/strip-slash), `TestExecuteSyncPlan` (10 — happy/404-ok/500-fail/403-fail/exception/method-carry/multipart/payload-recorded/with-reindex), `TestCli` (3 — dry-run/missing-file/invalid-doc), `TestDocSourceAudit` (3 — main-stand-anker/main-recent-patch/mirror-stand-anker), `TestWorkflowSourceAudit` (2 — sync-in-doku-pflicht/goal-18), `TestSmoke` (3 — module-exports/constants/ps1-wrapper).

**Lokal:** 2123 Baseline → **2176 passed** (+53 P210), 4 xfailed pre-existing, 3 failed pre-existing aus Schuldenliste, 0 NEUE Failures aus P210.

**Logging-Tag:** `[SYNC-210]` mit "Quelle:", "Server:", "Source-Name:", "Kategorie:", "Reindex:", "Auth: gesetzt|NICHT gesetzt", "Dry-Run — N geplante Schritte:", "Sync erfolgreich.", "Sync fehlgeschlagen (N Step(s)).", "Plan-Fehler: ...". Pure-User-CLI-Logs (kein Server-Side-Logging — Server schreibt seine eigenen `[RAG-108/111/116/169]`-Tags).

---

**Patch 209** — Zweite Meinung vor Ausführung / Sancho Panza (Phase 5a Ziel #7 ABGESCHLOSSEN) (2026-05-03)

Veto-Logik VOR dem HitL-Gate aus P206. Vor dem User-Approval bewertet ein zweites LLM den vom Haupt-Modell generierten Code: macht er das, was der User wirklich will, UND ist er sicher (kein Datenverlust, keine ungewollte Netzwerk-Aktion, keine Rechte-Eskalation)? Bei VETO landet der Code nicht im HitL-Gate sondern in einem Wandschlag-Banner mit Veto-Begruendung — kein HitL-Pending, kein Sandbox-Run, kein Snapshot. Bei PASS laeuft der bestehende P206/P207-Pfad unveraendert weiter. Bei trivialen 1-Zeilern (print/return/var-assign/pass) ohne Risk-Tokens ueberspringt eine Pure-Function-Heuristik den Veto-LLM-Call komplett (Token-Spar) und auditiert nur "skipped".

**Architektur: drei Schichten plus Feature-Flags plus DB-Audit plus Frontend-Card.**

- **Veto-Modul** [`zerberus/core/code_veto.py`](zerberus/core/code_veto.py) mit Pure-Function-Schicht (`should_run_veto(code, language) -> bool` Trigger-Gate mit `_RISKY_TOKENS`-Liste fuer subprocess/eval/exec/rm/unlink/requests/git-force/--no-verify/pickle.load/...; `_is_trivial_oneliner(code) -> bool` matcht print/return/pass/var-assign/console.log; `build_veto_messages(code, language, user_prompt) -> List[dict]` baut den Probe-Prompt mit `VETO_SYSTEM_PROMPT` der GENAU "PASS" oder "VETO" am Anfang verlangt; `parse_veto_verdict(text) -> VetoVerdict` robust gegen Whitespace/Quotes/Markdown-Bold/Doppelpunkte/Bindestriche zwischen Token und Reason, plus First-Line-Match + Multi-Line-Reason + 64-char-Window-Fallback), Async-Wrapper (`run_veto(code, language, user_prompt, llm_service, session_id, *, temperature=0.1)` ruft den Veto-LLM mit `temperature_override=0.1` und liefert `VetoVerdict` mit `veto`/`reason`/`raw`/`latency_ms`/`error`; fail-open auf jede Pipeline-Stoerung), Audit-Helper (`store_veto_audit(...)` schreibt in `code_vetoes`-Tabelle, Best-Effort, 8 KB Truncate, eigene UUID4-`audit_id` statt HitL-Pending-ID).
- **Heuristik-Trigger** lehnt triviale Code-Bloecke ab: leerer Code, `print('hi')`, `return 42`, `x = 1`, `pass`, `console.log('x')` → kein LLM-Call. Multiline-Code (`>= 2 nicht-leere Zeilen`) ODER Code mit Risk-Tokens (`subprocess`, `eval(`, `rm -rf`, `open(`, `requests.post`, `git push --force`, `--no-verify`, `pickle.load`, `fs.unlink`, ...) triggert den Probe.
- **VetoVerdict-Dataclass** mit `to_payload_dict() -> {"vetoed": bool, "reason": str, "latency_ms": int|None}` als Frontend-Schema.
- **Neue Tabelle** `code_vetoes` in [`zerberus/core/database.py`](zerberus/core/database.py): `audit_id` (UUID4 hex INDEX), `session_id`, `project_id` (INDEX), `project_slug`, `language`, `code_text`, `user_prompt`, `verdict` (`pass|veto|skipped|error` INDEX), `reason`, `latency_ms`, `created_at`. Persistente Spur fuer System-Prompt-Tuning + Threshold-Anpassung.
- **Verdrahtung in [`legacy.py::chat_completions`](zerberus/app/routers/legacy.py)** zwischen `first_executable_block` und HitL-Pending-Erzeugung. Bei `code_veto_enabled=True` UND `should_run_veto(code, language)=True` faehrt `run_veto(...)` mit `temperature_override=settings.projects.code_veto_temperature` (Default `0.1`). Drei Decision-Pfade:
  - `verdict.veto=True` → `_veto_skip_hitl_and_sandbox=True`, `code_execution_payload` wird mit `skipped=True`/`hitl_status="vetoed"`/`veto={vetoed, reason, latency_ms}`-Sub-Field gesetzt; HitL/Sandbox/Snapshot werden uebersprungen.
  - `verdict.veto=False` (PASS) → `veto_status_for_audit="pass"`, weiter zum HitL-Gate aus P206.
  - `verdict.error` → `veto_status_for_audit="error"`, weiter zum HitL-Gate (fail-open).
  - `should_run_veto=False` → `veto_status_for_audit="skipped"` (trivialer Code, kein LLM-Call), weiter zum HitL-Gate.
- **Audit-Trail** wird **nur** geschrieben wenn der Veto-Pfad aktiv war. Bei `code_veto_enabled=False` bleibt die `code_vetoes`-Tabelle leer fuer diesen Run — der Veto-Pfad existiert in dem Fall faktisch nicht.
- **Bei VETO** schreibt der Endpunkt **keine** Zeile in `code_executions` (P206-Tabelle), weil HitL nicht lief — der Audit landet ausschliesslich in `code_vetoes`. Korrelation zwischen den beiden Tabellen: `session_id`/`project_id` matchen.
- **Nala-Frontend** in [`nala.py`](zerberus/app/routers/nala.py):
  - **CSS** (~85 Zeilen): `.veto-card`/`.veto-card-header`/`.veto-lang-tag`/`.veto-reason`/`.veto-meta`/`.veto-code-toggle` (44x44 Touch-Target)/`.veto-code-block`/`.veto-card.veto-collapsed`. Rote Border (`rgba(229,115,115,0.55)`), Begruendung prominent, Code-Snippet collapsible (Default collapsed), Latency-Meta optional. Kein Approve-Button — Read-only Audit-Spur.
  - **JS-Funktion** `renderVetoCard(wrapperEl, codeExec, triptych)` analog `renderDiffCard` aus P207. User-/LLM-Strings via `textContent` (XSS-safe by default), Code-Block via `escapeHtml(codeStr)` mit `<code>`-Wrapping. Insertion vor dem Triptychon, identische Visual-Order zu P203d-3 (bubble → veto-card → triptych → export-row, OHNE Code-Card und Output-Card weil Veto-Pfad).
  - **Verdrahtung in `renderCodeExecution`**: ganz frueh wird `codeExec.veto.vetoed === true` geprueft → `renderVetoCard(...)` + `return` (early-exit). Bei `vetoed=false` oder fehlendem Veto-Field laeuft der bestehende Code-Card/Output-Card/Diff-Card-Pfad unveraendert weiter — Backwards-Compat zu P203d-3/P206/P207.
- **Feature-Flags** in [`config.py::ProjectsConfig`](zerberus/core/config.py): `code_veto_enabled: bool = True` Master-Switch (off → kein Probe, kein Audit), `code_veto_temperature: float = 0.1` (deterministisch fuer wiederholbare Verdicts).

**Was P209 bewusst NICHT macht:**

- **Mehr-Modell-Voting** — genau ein Veto-Call pro Run. Mehrere Modelle waeren teurer und das Verhaeltnis von Kosten zu Robustheit-Gewinn ist fragwuerdig.
- **Lern-Loop ueber `code_vetoes`-History** — keine automatische Threshold-/Prompt-Adjustierung. Manuelle Auswertung via SQL.
- **Kosten-Aggregation des Veto-Calls in `interactions.cost`** — Schuld analog P203d-1/P203d-2/P208 (Probe-Calls sind aktuell nicht in `interactions.cost`).
- **Veto-Override durch User** — kein "ich weiss was ich tue, fuehre trotzdem aus"-Button. Bei false-positives muss der User die Frage neu stellen oder den Code anders formulieren.
- **Sprach-spezifische Heuristiken** — `should_run_veto`-Signatur akzeptiert `language` als Parameter, nutzt es aber noch nicht. Spaeter koennte `bash` aggressiver triggern als `python`.
- **Persistenz der Veto-Verdicts ueber Server-Restart** — `VetoVerdict` ist transient. Nur der Audit landet in der DB.
- **Telegram-Pfad** — Huginn hat eigenen P167-HitL ohne Veto.

**Tests:** 88 in [`test_p209_code_veto.py`](zerberus/tests/test_p209_code_veto.py). Strukturiert in 12 Klassen: `TestShouldRunVeto` (13 — leer/trivial-print/return/var/pass/multiline/subprocess/eval/rm-rf/open-write/requests-post/long-oneliner/none-language), `TestRiskyTokens` (6 — subprocess/eval/no-risky/case-insensitive/force-push/no-verify), `TestTrivialOneliner` (5 — print/multiline/long-line/pass/return), `TestBuildVetoMessages` (7 — two-messages/system-pass-veto/user-code-lang/lang-lower/empty-lang/long-truncate/no-persona-leak), `TestParseVetoVerdict` (13 — pass/veto/dash/lowercase/markdown-bold/quoted/pass-reason-ignored/unparseable-fail-open/empty/multiline-reason/64char-fallback/long-reason-truncate), `TestVetoVerdict` (2 — payload-dict-pass/payload-dict-veto), `TestRunVeto` (8 — happy-pass/happy-veto/temperature-passed/default-low/llm-crash/empty/non-tuple/non-string), `TestStoreVetoAudit` (3 — happy/truncate/no-db), `TestLegacySourceAudit` (10 — Logging-Tag, Imports, Audit-Aufruf, Feature-Flag, Temperature-Param, Reihenfolge-vor-HitL, Skip-Var, Veto-Field-im-Payload, hitl_status=vetoed, six-Audit-Fields, fail-open), `TestNalaSourceAudit` (8 — renderVetoCard-Funktion, Aufruf-in-renderCodeExecution, early-Return, CSS-Klassen, rote Border, 44px-Touch, kein Approve-Button, textContent/escapeHtml, veto-collapsed-State), `TestE2EVeto` (5 — veto-blockt-sandbox/pass-zur-sandbox/trivial-skipt-veto/disabled-skipt-audit/veto-keine-hitl-pending), `TestJsSyntaxIntegrity` (1 — `node --check` ueber NALA_HTML, skipped wenn node fehlt), `TestSmoke` (4 — config-flags / default-temperature-low / code_vetoes-tabelle / module-exports).

**Lokal:** 2035 baseline → **2123 passed** (+88 P209), 4 xfailed pre-existing, 3 failed pre-existing aus Schuldenliste (`edge-tts`, `test_rag_dual_switch.test_fallback_logic`, `test_patch185_runtime_info` durch `config.yaml`-Drift `deepseek-v4-pro`), 0 NEUE Failures aus P209.

**Logging-Tag:** `[VETO-209]` mit `decision veto=... reason_len=... code_len=... lang=... session=... latency_ms=...`, `blocked session=... language=... reason_len=...`, `audit_written session=... project_id=... verdict=... language=... latency_ms=...`, `Pipeline-Fehler (fail-open): ...`, `llm_call_failed (fail-open): ...`.

---

**Patch 208** — Spec-Contract / Ambiguitäts-Check (Phase 5a Ziel #8 ABGESCHLOSSEN) (2026-05-03)

Erst verstehen, dann coden. Vor dem ersten Haupt-LLM-Call schaetzt eine Pure-Function-Heuristik die Ambiguitaet der User-Eingabe; bei Score >= Threshold (Default 0.65) faehrt ein schmaler "Spec-Probe"-LLM-Call (eine Frage, keine Vorrede), das Frontend rendert eine Klarstellungs-Karte mit Original-Message + Frage + Textarea + drei Buttons. User antwortet, klickt "Trotzdem versuchen" oder bricht ab. Erst danach laeuft der eigentliche Code-/Antwort-Pfad mit ggf. angereichertem Prompt weiter. Whisper-Input bekommt einen Score-Bonus (`+0.20`) — Voice-Transkripte sind systematisch ambiger als getippter Text.

**Architektur: drei Schichten plus Feature-Flags plus Endpoints plus Frontend-Card.**

- **Spec-Modul** [`zerberus/core/spec_check.py`](zerberus/core/spec_check.py) mit Pure-Function-Schicht (`compute_ambiguity_score(message, *, source="text"|"voice")` heuristik 0.0-1.0; `should_ask_clarification(score, *, threshold=0.65)` Trigger-Gate; `build_spec_probe_messages(message)` baut die `messages`-Liste fuer den Probe-Call; `build_clarification_block(question, answer)` plus `enrich_user_message(original, question, answer)` haengen `[KLARSTELLUNG]…[/KLARSTELLUNG]` an; Marker substring-disjunkt zu `[PROJEKT-RAG]`/`[PROJEKT-KONTEXT]`/`[PROSODIE]`/`[CODE-EXECUTION]`/`[AKTIVE-PERSONA]`), Async-Wrapper (`run_spec_probe(message, llm_service, session_id)` ruft den Probe-LLM mit `temperature=0.3` und liefert die Frage oder `None`), Pending-Registry (`ChatSpecGate`-Singleton analog `ChatHitlGate` aus P206 — In-Memory only, `asyncio.Event` pro Pending, Cross-Session-Defense ueber `session_id`-Match), Audit-Helper (`store_clarification_audit(...)` schreibt in `clarifications`-Tabelle, Best-Effort, 4 KB Truncate).
- **Heuristik-Score** addiert pro Treffer einen Penalty: kurze Saetze (<4 Woerter +0.40, <8 +0.20, <14 +0.05), Pronomen-Dichte (max +0.30), Code-Verb ohne Sprache (+0.20), generisches Verb ohne Substantiv-Anker (+0.15), Code-Verb ohne IO-Spec (+0.10), Voice-Bonus (+0.20). Clamped auf [0, 1]. Ein klarer Prompt mit Sprachangabe + IO-Spec landet meist <0.4, "bau das" landet >0.8.
- **Neue Tabelle** `clarifications` in [`zerberus/core/database.py`](zerberus/core/database.py): `pending_id`, `session_id`, `project_id`, `project_slug`, `original_message`, `question`, `answer_text`, `score` (Float), `source` (text|voice), `status` (answered|bypassed|cancelled|timeout|error), `created_at`, `resolved_at`. Persistente Spur fuer Threshold-Tuning.
- **Verdrahtung in [`legacy.py::chat_completions`](zerberus/app/routers/legacy.py)** zwischen `last_user_msg`-Cleanup (Cloud/Local-Suffix entfernt) und Intent-Detection. Drei Decision-Pfade:
  - `answered` mit `answer_text` → `last_user_msg` wird via `enrich_user_message` mit `[KLARSTELLUNG]`-Block angereichert, `req.messages` mitgespiegelt → Haupt-LLM-Call sieht Frage + User-Antwort
  - `bypassed` → Original durch (User akzeptiert Risiko)
  - `cancelled` → Chat endet mit Hinweis-Antwort, **kein** Haupt-LLM-Call (early-return mit `model="spec-cancelled"`)
  - `timeout` → wie `bypassed` (defensiver: User schaut nicht hin → eher durchlassen statt frustrieren)
- **Source-Detection**: Voice wenn `X-Prosody-Context` + `X-Prosody-Consent: true` Header gesetzt sind (P204-Konvention). Sonst Text.
- **Neue Endpoints** in `legacy.py` (auth-frei wie `/v1/hitl/*`, Dictate-Lane-Invariante):
  - `GET /v1/spec/poll` → liefert aeltestes pending Spec-Pending der Session
  - `POST /v1/spec/resolve` mit `{pending_id, decision, session_id, answer_text?}` → idempotent + Cross-Session-Block. `answered` braucht non-empty `answer_text`, sonst `ok=False`.
- **Nala-Frontend** in [`nala.py`](zerberus/app/routers/nala.py):
  - **CSS** (~135 Zeilen): `.spec-card`/`.spec-card-header`/`.spec-source-tag`/`.spec-original`/`.spec-question`/`.spec-answer-row`/`.spec-answer-input` (textarea, focus-Border in Gold)/`.spec-actions`/`.spec-answer-btn` (gold)/`.spec-bypass-btn` (gruen)/`.spec-cancel-btn` (rot) plus Post-Klick-States `.spec-card.spec-{answered,bypassed,cancelled}`. Mobile-first 44x44 Touch-Targets fuer alle drei Buttons.
  - **JS-Funktionen** (~150 Zeilen): `startSpecPolling(abortSignal, snapshotSessionId)` pollt `/v1/spec/poll` im Sekunden-Takt waehrend der Chat-Long-Poll laeuft. `renderSpecCard(pending)` baut Karte mit Original-Message (textContent — XSS-safe by default), Frage (textContent), Textarea, drei Buttons (`✉️ Antwort senden` / `→ Trotzdem versuchen` / `✗ Abbrechen`). `resolveSpecPending(pendingId, decision, answerText)` macht POST und setzt Card-State (`spec-answered` gruen, `spec-bypassed` gold, `spec-cancelled` rot). Karte bleibt nach Klick sichtbar als Audit-Spur. `clearSpecState()`/`stopSpecPolling()` analog HitL.
  - **Verdrahtung in `sendMessage`**: nach `clearHitlState() + startHitlPolling(...)` analog `clearSpecState() + startSpecPolling(myAbort.signal, reqSessionId)`. Beide laufen unabhaengig — Spec kommt VOR dem LLM-Call, HitL kommt NACH dem Code-Block.
- **Feature-Flags** in [`config.py::ProjectsConfig`](zerberus/core/config.py): `spec_check_enabled: bool = True`, `spec_check_threshold: float = 0.65`, `spec_check_timeout_seconds: int = 60`. Master-Switch off → kein Probe, kein Pending, kein Block in der Response.

**Was P208 bewusst NICHT macht:**

- **Sprachen-Erkennung im Code-Block** — das macht P203d-1 (auf der LLM-Output-Seite, nicht der User-Seite).
- **Refactoring der HitL-Card (P206)** — Spec ist eine separate Karte, die VOR HitL kommt. Beide bleiben unabhaengig.
- **Multi-Turn-Klarstellung** — eine Frage pro Turn, dann zurueck in den Hauptpfad. Folge-Klarstellungen sind ein eigener Patch.
- **Telegram-Pfad** — Huginn hat eigenen HitL aus P167; Spec-Probe waere dort ein Overkill (Telegram-Threads sind asynchroner als Chat).
- **Persistierung der Pendings** — In-Memory only, Long-Poll-Requests sterben beim Restart sowieso (analog P206-Konvention).
- **Edit-Vor-Run** — User kann den Original-Prompt nicht editieren in der Card, nur antworten oder bypassen. Bei Bedarf neue Message schicken.
- **Token-Cost-Tracking fuer den Probe-Call** — Schuld analog P203d-1 (interactions.cost erfasst nur den Haupt-Call).

**Tests:** 89 in [`test_p208_spec_contract.py`](zerberus/tests/test_p208_spec_contract.py). Strukturiert in 14 Klassen: `TestComputeAmbiguityScore` (9 — leer/range/length/voice/code-verb/pronoun/clear/clamp/io), `TestShouldAskClarification` (5 — above/below/exact/invalid/custom-threshold), `TestBuildSpecProbeMessages` (5 — list-shape/system-content/user-content/empty/system-prompt-constrains), `TestEnrichUserMessage` (6 — marker/preserve/q+a/empty/answer-only/build), `TestMarkerUniqueness` (2 — disjunkt/format), `TestRunSpecProbe` (7 — happy/strip/empty/whitespace/crash/non-tuple/truncate), `TestChatSpecGate` (13 — uuid/dict-keys/list-filter/answered+text/answered-empty-rejected/bypassed/cancelled/invalid-decision/session-mismatch/wait-immediate/wait-timeout/cleanup/answer-truncated), `TestStoreClarificationAudit` (3 — happy/truncate/no-db), `TestSpecPollResolveEndpoints` (6 — empty/by-session/no-leak/answered/bypassed/unknown), `TestLegacySourceAudit` (13 — Logging-Tag, Imports, Endpoints, Pydantic-Models, source-Kwarg, Threshold/Timeout/Enabled-Flags, cancelled-early-return, enrich, voice-detection, audit-call), `TestNalaSourceAudit` (10 — JS-Funktionen, Endpoints, sendMessage-Verdrahtung, CSS-Klassen, 44px-Touch, escapeHtml/textContent, textarea, drei Decisions, Audit-Trail-States), `TestE2ESpecCheck` (5 — non-ambig skip / ambig+bypassed / ambig+answered+enrichment / ambig+cancelled+early-return / disabled-flag-skip), `TestJsSyntaxIntegrity` (1 — `node --check` ueber NALA_HTML, skipped wenn node fehlt), `TestSmoke` (4 — config-flags / clarifications-tabelle / endpoints / module-exports).

**Lokal:** 1946 baseline → **2035 passed** (+89 P208), 4 xfailed pre-existing, 3 failed pre-existing aus Schuldenliste (`edge-tts`, `test_rag_dual_switch.test_fallback_logic`, `test_patch185_runtime_info` durch `config.yaml`-Drift `deepseek-v4-pro`), 0 NEUE Failures aus P208.

**Logging-Tag:** `[SPEC-208]` mit `pending_create id=... session=... score=... source=... question_len=...`, `decision id=... session=... status=... answer_len=...`, `ambig score=... threshold=... source=... session=...`, `not_ambig score=... threshold=... source=... session=...`, `probe_returned_empty session=... (fail-open)`, `audit_written session=... project_id=... status=... source=... score=...`, `Pipeline-Fehler (fail-open): ...`.

---

**Patch 207** — Workspace-Snapshots + Diff-View + Rollback (Phase 5a Ziele #9 + #10 ABGESCHLOSSEN) (2026-05-03)

Sicherheitsnetz NACH der Sandbox-Ausfuehrung. P206 gibt das Yay/Nay VOR dem Run; P207 gibt die Sicht UND die Reverse-Option DANACH. Wenn `projects.sandbox_writable=True` (Default False — RO bleibt P203c-Standard) UND `projects.snapshots_enabled=True` (Default True) UND HitL approved hat, schiesst der Chat-Endpunkt einen `before_run`-Tar-Snapshot, faehrt die Sandbox writable und schiesst danach einen `after_run`-Snapshot. Aus dem Paar entsteht ein Diff (added/modified/deleted plus optional `unified_diff` fuer Text-Files <64 KB), der additiv im `code_execution.diff`-Feld der Response landet — zusammen mit `before_snapshot_id` und `after_snapshot_id`. Der User sieht im Frontend eine Diff-Card und kann via `↩️ Aenderungen zurueckdrehen` den Workspace auf den `before`-Stand zuruecksetzen.

**Architektur: drei Schichten plus Feature-Flag plus Endpoint plus Frontend-Card.**

- **Snapshot-Modul** [`zerberus/core/projects_snapshots.py`](zerberus/core/projects_snapshots.py) mit Pure-Function-Schicht (`build_workspace_manifest`, `diff_snapshots`, `_looks_text`, `_is_safe_member` als Tar-Member-Validation, `_build_unified_diff`), Sync-FS-Schicht (`materialize_snapshot` schreibt atomar via Tempname + `os.replace`, `restore_snapshot` raeumt Workspace-Inhalt komplett und extrahiert Tar mit Member-Validation), Async-DB-Schicht (`store_snapshot_row`/`load_snapshot_row` auf neue Tabelle `workspace_snapshots`) und High-Level-Convenience (`snapshot_workspace_async`/`rollback_snapshot_async` mit `expected_project_id`-Check).
- **Neue Tabelle** `workspace_snapshots` in [`zerberus/core/database.py`](zerberus/core/database.py): `snapshot_id` (UUID4-hex UNIQUE), `project_id`, `project_slug`, `label` (`before_run`/`after_run`/`manual`), `archive_path`, `file_count`, `total_bytes`, `pending_id` (Korrelation zu `hitl_chat`/`code_executions` aus P206), `parent_snapshot_id` (zeigt vom `after`- auf den `before`-Snapshot derselben Ausfuehrung), `created_at`. Snapshot-Tars liegen unter `data/projects/<slug>/_snapshots/<id>.tar`.
- **Verdrahtung in [`legacy.py::chat_completions`](zerberus/app/routers/legacy.py)** zwischen P206-Approve und `execute_in_workspace`:

```python
_writable = bool(getattr(settings.projects, "sandbox_writable", False))
_snapshots_active = _writable and bool(getattr(settings.projects, "snapshots_enabled", True))

if _snapshots_active:
    _before_snap = await snapshot_workspace_async(project_id=..., label="before_run", pending_id=...)

_result = await execute_in_workspace(..., writable=_writable)  # ehemals hardcoded False

if _snapshots_active and _before_snap is not None:
    _after_snap = await snapshot_workspace_async(label="after_run", parent_snapshot_id=_before_snap["id"])
    _diff = diff_snapshots(_before_snap["manifest"], _after_snap["manifest"])
    code_execution_payload["diff"] = [d.to_public_dict() for d in _diff]
    code_execution_payload["before_snapshot_id"] = _before_snap["id"]
    code_execution_payload["after_snapshot_id"] = _after_snap["id"]
```

Fail-Open auf jeder Stufe; HitL-Skip-Pfad (rejected/timeout) triggert keinen Snapshot.

- **Neuer Endpoint** `POST /v1/workspace/rollback` in `legacy.py` (auth-frei wie `/v1/hitl/*`): `WorkspaceRollbackRequest{snapshot_id, project_id}` → `WorkspaceRollbackResponse{ok, snapshot_id?, project_id?, project_slug?, file_count?, total_bytes?, error?}`. Reject-Pfade: `snapshots_disabled`, `restore_failed` (unbekannter snapshot ODER project_mismatch), `pipeline_error`. Cross-Project-Defense: `expected_project_id` muss zum Snapshot-Eigentuemer passen.
- **Nala-Frontend** in [`nala.py`](zerberus/app/routers/nala.py):
  - **CSS** (~125 Zeilen): `.diff-card`/`.diff-card-header`/`.diff-summary`/`.diff-list`/`.diff-entry`/`.diff-entry-head`/`.diff-status.diff-{added,modified,deleted}`/`.diff-path`/`.diff-size`/`.diff-content`/`.diff-content .diff-line-{add,del,meta}`/`.diff-binary-note`/`.diff-actions`/`.diff-rollback`/`.diff-resolved` plus Post-Klick-States `.diff-card.diff-{rolled-back,rollback-failed}`. Kintsugi-Gold-Border, `.diff-rollback` mit `min-height: 44px`/`min-width: 44px`.
  - **JS-Funktionen** (~250 Zeilen): `renderDiffCard(wrapperEl, codeExec, triptych)` baut Header mit Summary `N neu, M geaendert, K geloescht`, dann pro DiffEntry eine `<li>` mit Status-Badge + Pfad (escapeHtml) + Size-Label; Klick toggled Inline-Diff. `colorizeUnifiedDiff(text)` faerbt Plus/Minus/Header (gruen/rot/grau) — nutzt `String.fromCharCode(10)` statt `'\n'`-Literal (Lesson aus P203b: Newline-Escapes werden in Python-Source frueh interpretiert). `rollbackWorkspace(cardEl, snapshotId, projectId)` macht POST und setzt Card-State.
  - **`renderCodeExecution`-Erweiterung:** nach Code-Card + Output-Card-Insert, falls `!skipped && Array.isArray(codeExec.diff) && codeExec.before_snapshot_id` → `renderDiffCard(...)`. Backwards-compat zu P206-only-Backends.
- **Feature-Flags** in [`config.py::ProjectsConfig`](zerberus/core/config.py): `sandbox_writable: bool = False` (RO-Default bleibt) + `snapshots_enabled: bool = True` (Master-Switch).
- **Konvention-Update:** der hardcoded `writable=False` aus P203d-1 wurde durch `getattr(settings.projects, "sandbox_writable", False)` ersetzt. Der zugehoerige P203d-1-Source-Audit-Test wurde entsprechend nachgezogen.

**Was P207 bewusst NICHT macht:**

- **Cross-Project-Diff** — Snapshots sind per Projekt isoliert, ein Cross-Diff macht selten Sinn.
- **Branch-Mechanik** — linear forward/reverse only. `parent_snapshot_id` koennte zu einem Tree ausgebaut werden, aktuell nur fuer `before→after`-Korrelation.
- **Automatischer Rollback bei `exit_code != 0`** — User-Choice. Manche Crashes hinterlassen wertvolle Teil-Outputs.
- **Per-File-Rollback** — alles oder nichts pro Snapshot. Inline-Diff-Anzeige ist additiv, Rollback wirkt aufs Ganze.
- **Hardlink-Snapshots** — Tar ist Tests-tauglich + atomar. Bei messbaren Disk-Problemen umstellen.
- **Storage-GC fuer alte Snapshots** — alte `.tar`-Files bleiben liegen. "Behalte die letzten N pro Projekt"-Sweep ist eigener Patch (HANDOVER-Schuld).
- **Sync-After-Write zurueck in den SHA-Storage** — geaenderte Files leben nur im Workspace, nicht im SHA-Storage. Schuld bleibt offen.
- **Cost-Tracking** — Snapshots erzeugen keine LLM-Kosten, nur Disk + DB.

**Tests:** 74 in [`test_p207_workspace_snapshots.py`](zerberus/tests/test_p207_workspace_snapshots.py). Strukturiert in 9 Klassen analog P206: Pure-Function-Schicht (Manifest, Diff, Looks-Text, Is-Safe-Member, Unified-Diff), Sync-FS (Materialize, Restore mit Path-Traversal-Defense), DB (Store/Load + Convenience), Endpoint (OK/restore_failed/project_mismatch/snapshots_disabled), Source-Audit legacy.py + nala.py, End-to-End mit Mock-Sandbox die den Workspace mutiert, JS-Integrity (`node --check`), Smoke. Plus 1 nachgezogener Test in `test_p203d_chat_sandbox.py` (`writable=False` ist nicht mehr hardcoded, sondern Settings-Lookup mit Pydantic-Default).

**Lokal:** 1872 baseline → **1946 passed** (+74 P207), 4 xfailed pre-existing, 3 failed pre-existing aus Schuldenliste (`edge-tts`, `test_rag_dual_switch.test_fallback_logic`, `test_patch185_runtime_info` durch `config.yaml`-Drift `deepseek-v4-pro`), 0 NEUE Failures aus P207.

**Logging-Tag:** `[SNAPSHOT-207]` mit `materialized id=... label=... file_count=... total_bytes=...`, `db_row_written ...`, `diff project_id=... before=... after=... changes=...`, `restored archive=... file_count=...`, `rollback_done snapshot_id=... project_id=... slug=...`, `rollback_endpoint snapshot_id=... project_id=...`, plus Defense-Logs (`restore: unsafe member skipped: ...`, `before_run fehlgeschlagen (fail-open): ...`).

---

**Patch 206** — HitL-Gate vor Code-Execution + HANDOVER-Teststand-Konvention (Phase 5a Ziel #6 ABGESCHLOSSEN) (2026-05-03)

Bisher (P203d-1) lief jeder erkannte Code-Block direkt in die Sandbox — RO-Mount machte das vergleichsweise sicher, aber Phase-5a-Ziel #6 fordert explizite User-Confirmation. P206 schliesst das Ziel mit einem In-Memory-Long-Poll-Gate und einer Confirm-Karte im Nala-Frontend. Plus integriert die kleine Teststand-Reminder-Konvention aus dem Feature-Request (HANDOVER-Header zeigt `**Manuelle Tests:** X / Y ✅`).

**Architektur: drei Schichten plus Feature-Flag plus Audit-Tabelle.**

- **Pure-Mechanik** in [`zerberus/core/hitl_chat.py`](zerberus/core/hitl_chat.py): `ChatHitlGate`-Singleton mit dict-Registry + `asyncio.Event` pro Pending. `create_pending(session_id, project_id, project_slug, code, language)` legt UUID4-hex-IDs an, `wait_for_decision(pending_id, timeout)` blockt via `asyncio.wait_for(event.wait(), timeout=N)`, `resolve(pending_id, decision, *, session_id)` flippt status + setzt Event mit Cross-Session-Defense, `cleanup(pending_id)` raeumt nach Resolve. In-Memory-only — Long-Poll-Requests sterben beim Server-Restart sowieso, persistente Pendings wuerden zu "Geister-Karten" fuehren. Plus `store_code_execution_audit(...)`-Helper schreibt eine Audit-Zeile mit 8 KB Truncate fuer code/stdout/stderr.
- **Audit-Tabelle** `code_executions` in [`zerberus/core/database.py`](zerberus/core/database.py) — schliesst die HANDOVER-Schuld P203d-1 ("code_execution ist nicht in der DB"). Spalten: `pending_id`/`session_id`/`project_id`/`project_slug`/`language`/`exit_code`/`execution_time_ms`/`truncated`/`skipped`/`hitl_status`/`code_text`/`stdout_text`/`stderr_text`/`error_text`/`created_at`/`resolved_at`. SQLite-friendly mit `Integer` 0/1 fuer Boolean. `init_db`-Bootstrap legt sie automatisch an.
- **Verdrahtung in [`legacy.py::chat_completions`](zerberus/app/routers/legacy.py)** zwischen `first_executable_block` und `execute_in_workspace`:

```python
if getattr(settings.projects, "hitl_enabled", True):
    _gate = get_chat_hitl_gate()
    _pending = await _gate.create_pending(...)
    _hitl_decision = await _gate.wait_for_decision(_pending.id, _timeout)
    _gate.cleanup(_pending.id)
else:
    _hitl_decision = "bypassed"

if _hitl_decision in ("approved", "bypassed"):
    _result = await execute_in_workspace(..., writable=False)
    code_execution_payload = {..., "skipped": False, "hitl_status": _hitl_decision}
else:
    code_execution_payload = {..., "skipped": True, "hitl_status": _hitl_decision,
                              "exit_code": -1, "error": "Vom User abgebrochen"}
```

Synthese-Gate (P203d-2) erweitert: `if code_execution_payload is not None and not code_execution_payload.get("skipped"):` — kein zweiter LLM-Call bei Skip.

- **Zwei neue auth-freie Endpoints** in `legacy.py` (per `/v1/`-Invariante kein JWT):
  - `GET /v1/hitl/poll` — liefert das aelteste Pending der Session als JSON (oder `{"pending": null}`). Header `X-Session-ID` als Owner-Diskriminator.
  - `POST /v1/hitl/resolve` — Body `{pending_id, decision, session_id}`, idempotent + Cross-Session-Block. Antwort `{ok, decision}`.
- **Nala-Frontend** in [`nala.py`](zerberus/app/routers/nala.py):
  - **CSS** (~70 Zeilen): `.hitl-card` mit Kintsugi-Gold-Border `rgba(240,180,41,0.55)` + Inset-Shadow, `.hitl-actions` Flex-Row mit `gap: 8px`, `.hitl-approve`/`.hitl-reject` mit `min-height: 44px`/`min-width: 44px` (Mobile-first Touch). Post-Klick-States `.hitl-approved`/`.hitl-rejected` schimmern gruen/rot. `.hitl-resolved` als zentrierter italic-Text. Plus `.exit-badge.exit-skipped` fuer Code-Card im Skip-State.
  - **JS-Funktionen** (~120 Zeilen): `startHitlPolling(abortSignal, snapshotSessionId)` (1-Sekunden-Intervall, sofortige erste Runde, stoppt bei AbortSignal oder Session-Wechsel), `stopHitlPolling()`, `renderHitlCard(pending)` mit `escapeHtml(String(pending.code))` als XSS-Schutz, `resolveHitlPending(pendingId, decision)` (Buttons sperren, POST `/v1/hitl/resolve` mit `{pending_id, decision, session_id: sessionId}`, Card-State-Update), `clearHitlState()` als State-Reset.
  - **`sendMessage`-Verdrahtung:** vor dem Chat-Fetch wird `clearHitlState()` + `startHitlPolling(myAbort.signal, reqSessionId)` aufgerufen, im `finally`-Block `stopHitlPolling()`. Card bleibt nach Klick als Audit-Spur im DOM stehen.
  - **`renderCodeExecution`-Erweiterung:** liest `codeExec.skipped` und `codeExec.hitl_status`, ersetzt Exit-Badge bei Skip durch `⏸ uebersprungen` (rejected) oder `⏱ timeout` — Reason-Text kommt aus dem `error`-Feld.
- **Feature-Flag** `projects.hitl_enabled: bool = True` plus `hitl_timeout_seconds: int = 60` in `ProjectsConfig`. Bei `false` laeuft P203d-1-Verhalten ohne Gate (Audit-Status `bypassed`).
- **Doku-Pflicht-Erweiterung** in [`ZERBERUS_MARATHON_WORKFLOW.md`](ZERBERUS_MARATHON_WORKFLOW.md): HANDOVER-Header bekommt die Zeile `**Manuelle Tests:** X / Y ✅`. Implementierung der Feature-Request-Konvention aus dem Chris-Brief vom 2026-05-03 — kein Reminder-Text, keine Eskalation, nur die Zahl.

**Was P206 bewusst NICHT macht:**

- **Persistenz ueber Server-Restart** — bewusste Entscheidung, siehe oben. Bei zukuenftigen "Background-Code-Run-Modes" (LLM-getriggertes Cron-Job) muesste man das nochmal anschauen.
- **Edit-Vor-Run-Funktion** — User sieht den Code, kann aber nicht editieren. Falls UX-Feedback "Ich will den Code anpassen": Card mit Inplace-Textarea waere eine eigene UX-Schicht.
- **Telegram-Pfad** — Huginn nutzt sein eigenes P167-System (`HitlManager` mit DB-Persist). Architektur-Trennung ist Absicht (transient vs. delayed-Callback).
- **HitL fuer Output-Synthese** — zweites Gate waere Overkill. Die Synthese liest nur stdout/stderr und macht eine Zusammenfassung — kein Sicherheitsrisiko.

**Logging-Tag:** `[HITL-206]` mit `pending_create id=... session=... project_id=... language=... code_len=N` / `decision id=... session=... status=approved|rejected|timeout` / `bypassed session=... (hitl_enabled=False)` / `skipped session=... status=... language=...` / `audit_written session=... project_id=... hitl_status=... skipped=... exit_code=...`. Worker-Protection-konform: keine Code-Inhalte oder Output-Inhalte im Log.

**Tests:** 55 in [`test_p206_hitl_chat_gate.py`](zerberus/tests/test_p206_hitl_chat_gate.py). Acht Klassen: Pure-Gate-Mechanik (13), Audit-Trail-DB-Insert (3), Endpoint-Direktaufrufe (7), Source-Audit `legacy.py` (8), Source-Audit `nala.py` inkl. XSS + 44x44px (12), End-to-End mit gemocktem `wait_for_decision` (6), JS-Integrity per `node --check` (1), Smoke (3).

**Kollateral-Fix:** `test_p203d_chat_sandbox.py::_setup_common` und `test_p203d2_chat_synthesis.py::_setup` plus `test_synthesis_failure_keeps_original_answer` setzen jetzt `monkeypatch.setattr(get_settings().projects, "hitl_enabled", False)` — sonst wuerde das neue HitL-Gate (Default ON) im Test 60s auf eine Decision warten, die nie kommt.

**Teststand:** 1817 baseline (P205) → **1872 passed** (+55 P206-Tests), 4 xfailed pre-existing, 3 failed pre-existing aus Schuldenliste (`edge-tts`, `test_rag_dual_switch.test_fallback_logic`, `test_patch185_runtime_info` durch `config.yaml`-Drift), 0 neue Failures aus P206. 1 skipped (existing).

**Effekt fuer den User:** Nala-Chat mit aktivem Projekt + Code-erzeugendem Prompt (z.B. "Lies /workspace/data.txt"). Statt direkter Sandbox-Ausfuehrung erscheint eine Gold-umrandete Confirm-Karte mit Code-Vorschau und zwei 44x44px-Buttons. ✅ Ausfuehren → Karte wird gruen ("Code laeuft..."), Sandbox lauft, Output-Card erscheint. ❌ Abbrechen → Karte wird rot ("Abgebrochen"), Sandbox bleibt aus, Code-Card zeigt Skip-Badge `⏸ uebersprungen`. Audit-Trail in `code_executions`-Tabelle erlaubt spaeter Hel-Admin-Reports ueber Approve-Rate, Reject-Reasons, Timeout-Haeufigkeit pro Projekt.

---

## Älterer Patch

**Patch 205** — RAG-Toast in Hel-UI nach Datei-Upload (Phase 5a Schuld aus P199) (2026-05-03)

`POST /hel/admin/projects/{id}/files` retourniert seit P199 ein `rag`-Dict im Response-Body (`{chunks, skipped, reason}`). Das Hel-Frontend hat das Feld bisher ignoriert — der User sah nicht, ob die hochgeladene Datei indiziert wurde, wieviel Chunks gebaut wurden oder warum sie geskippt wurde (zu gross / Binärdatei / leer / kein Inhalt / Embed-Crash). P205 schliesst die Schuld mit einem dezenten Toast unten rechts.

**Architektur: Frontend-only, drei Bausteine** in [`zerberus/app/routers/hel.py`](zerberus/app/routers/hel.py):

- **CSS-Block** im `<style>`-Bereich (~30 Zeilen): `.rag-toast` mit `position: fixed`, `bottom: 20px`, `right: 20px`, `min-height: 44px` (Mobile-first Touch-Dismiss), `transition: opacity/transform 0.2s` plus `.rag-toast.visible`-State (opacity 1 + translateY 0). Zwei Border-Color-Varianten: `.success` (cyan `#4ecdc4`) und `.warn` (rot `#ff6b6b`). Erbt das Kintsugi-Gold-Border `#c8941f` als Default. `pointer-events: none` im Hidden-State, damit der Toast niemals einen Klick im Hintergrund abfaengt.
- **Reason-Mapping** als JS-Konstante `_RAG_REASON_LABELS` direkt vor `_uploadProjectFiles`: kurze de-DE-Strings fuer alle Codes aus `zerberus/core/projects_rag.py::index_project_file` (`rag_disabled` → "RAG aus", `too_large` → "zu gross", `binary` → "Binärdatei", `empty` → "leere Datei", `no_chunks` → "kein Inhalt", `embed_failed` → "Embed-Fehler", `file_not_found`/`project_not_found`/`bytes_missing` → entsprechend, `exception` → "Indizierungs-Fehler"). Unbekannte Codes fallen auf "übersprungen" zurueck.
- **`_showRagToast(rag)`** als JS-Renderer (~25 Zeilen): liest `rag.skipped` und `rag.chunks` und `rag.reason`, baut den Text via Mapping-Lookup (`📚 N Chunks indiziert` bei Erfolg, `⚠ Datei nicht indiziert: <Label>` bei Skip), setzt `el.textContent` (XSS-immun — Reason-Strings stammen NIEMALS direkt aus dem Server-Pfad, nur aus dem statischen Mapping), toggelt die `success`/`warn`-Klassen, fuegt `.visible` hinzu. Auto-Timeout 3500 ms via `setTimeout`, der frueh canceled wird wenn ein neuer Toast kommt (`_showRagToast._t` als Singleton-Slot). Klick auf den Toast cancelt den Timeout und versteckt sofort. Replace-Pattern: bei Multi-File-Upload bleibt der letzte Toast sichtbar (Progress-Liste zeigt pro File den Status — Toast ist additiv).

**DOM-Element** direkt vor `</body>` (vor dem SW-Reg-Script): `<div id="ragToast" class="rag-toast" role="status" aria-live="polite"></div>`. Ein einziger Toast-Container, kein Stacking — Reuse durch CSS-State-Toggle.

**Verdrahtung** im bestehenden Drop-Zone-Upload `_uploadProjectFiles` → `xhr.onload` Success-Branch direkt nach dem `'fertig'`-Render der Progress-Zeile:

```js
if (xhr.status >= 200 && xhr.status < 300) {
    row.innerHTML = '&#10004; ' + _escapeHtml(f.name) + ' — fertig';
    row.style.color = '#4ecdc4';
    // Patch 205: RAG-Status nach Erfolgs-Render als Toast zeigen.
    try {
        const body = JSON.parse(xhr.responseText);
        if (body && body.rag) _showRagToast(body.rag);
    } catch (_) {}
}
```

Fail-quiet auf jeder Stufe: kaputter JSON-Body bricht den Upload-Loop nicht ab; fehlendes `body.rag` (Backwards-Compat zu Backends ohne P199) → kein Toast.

**Was P205 bewusst NICHT macht:**
- **Kein neuer Backend-Endpoint, keine Schema-Aenderung.** Der `rag`-Block lag schon seit P199 in der Response. P205 ist reine Lesefehler-Behebung im Frontend.
- **Keine Sammel-Aggregation.** Bei Multi-File-Upload gewinnt der letzte Toast — die Progress-Liste oben zeigt pro File den Erfolgs-Status. Sammel-Toast (`📚 47 Chunks aus 3 Dateien indiziert`) waere komplexer Code fuer einen Edge-Case.
- **Kein Stacking.** Pattern aus dem HANDOVER-Spec: neuer Toast ersetzt alten via gleichem `#ragToast`-Container und CSS-State-Toggle.
- **Kein Nala-Pfad.** Nala hat aktuell keinen Datei-Upload (Projekt-Anlage ist Hel-only). Wenn P201/P196 spaeter Nala-seitige Uploads bekommt, muss der Toast dort separat verdrahtet werden — Quelltext-Lift-und-Klon, kein Module-Refactor.
- **Keine i18n.** de-DE hardgecodet, analog zur restlichen Hel-UI.
- **Kein Telegram-Pfad.** Huginn (Telegram-Adapter) hat keine Datei-Uploads dieser Art.
- **`escapeHtml`-Doppelung in hel.py** (Zeile 1653 + 3096) bleibt als bestehende Schuld stehen; P205 nutzt sowieso `textContent` und braucht keinen Helper.

**Logging-Tag:** keiner (Frontend-only, alle Backend-Logs `[RAG-199]` aus P199 bleiben).

**Tests:** 20 in [`test_p205_hel_rag_toast.py`](zerberus/tests/test_p205_hel_rag_toast.py):

- `TestToastFunctionExists` (2) — Funktion definiert + Signatur `_showRagToast(rag)`.
- `TestReasonMapping` (6) — alle 5 Hauptcodes (`too_large`, `binary`, `empty`, `no_chunks`, `embed_failed`) im Mapping plus expliziter `'too_large' → 'zu gross'`-Check.
- `TestRagToastCss` (4) — `.rag-toast` definiert, `min-height: 44px` (Mobile-first), `position: fixed`, Toggle-Klasse (`.visible`).
- `TestToastDom` (1) — `<div id="ragToast">` im HTML.
- `TestUploadWiring` (3) — `body.rag` im Upload-Block, `_showRagToast(...)`-Aufruf im Block, Reihenfolge `'fertig'` < `_showRagToast` (Toast NACH Render).
- `TestToastXss` (1) — `_showRagToast`-Body nutzt `textContent` ODER `_escapeHtml` (kein nacker `innerHTML` mit Reason-String).
- `TestJsSyntaxIntegrity` (1) — `node --check` ueber alle inline `<script>`-Bloecke aus `ADMIN_HTML` (skipped wenn `node` fehlt). Lesson aus P203b: ein einzelner SyntaxError invalidiert den gesamten Block.
- `TestHelHtmlSmoke` (2) — `ADMIN_HTML` enthaelt alle Toast-Pieces, genau ein `id="ragToast"` (kein Doppel-Render).

**Kollateral-Fix:** `test_projects_ui::TestP196JsFunctions::test_uploads_are_sequential` hat einen `[:3000]`-Slice auf `_uploadProjectFiles`-Body — durch das neue `_showRagToast` und die Toast-Verdrahtung wuchs der Body um ~600 Zeichen, der `for (let i = 0; i < files.length`-Marker rutschte ueber die 3000-Zeichen-Grenze. Slice auf 4500 erhoeht (gleiche Datei).

**Teststand:** 1797 baseline (P203d-3) → **1817 passed** (+20 P205-Tests), 4 xfailed pre-existing, 3 failed pre-existing aus Schuldenliste (`edge-tts`, `test_rag_dual_switch.test_fallback_logic`, `test_patch185_runtime_info` durch lokalen `config.yaml`-Drift `deepseek-v4-pro`), 0 neue Failures aus P205. 1 skipped (existing).

**Effekt fuer den User:** Beim Datei-Upload in Hel taucht unten rechts kurz ein Toast auf:
- `📚 14 Chunks indiziert` (cyan-Border, Erfolg)
- `⚠ Datei nicht indiziert: zu gross` (rot-Border, Skip mit Grund)

3.5 Sekunden sichtbar oder bis Tap. Bei Multi-Upload sequenziell: jeder File ersetzt den vorigen Toast. Die Progress-Zeilen oberhalb des Drop-Bereichs zeigen pro File den Roh-Status (✓ fertig / ✗ Fehler) wie bisher — RAG-Status ist die Zusatzinfo, die nur im Toast erscheint.

---

## Aeltere Patches

**Patch 203d-3** — UI-Render im Nala-Frontend fuer Sandbox-Code-Execution (Phase 5a Ziel #5 ABGESCHLOSSEN) (2026-05-03)

Dritter und letzter Sub-Patch der P203d-Aufteilung (P203d-1 Backend-Pfad / P203d-2 Output-Synthese / P203d-3 UI-Render). Schliesst Phase-5a-Ziel #5 endgueltig: nach P203d-1 reichte der Chat-Endpunkt das `code_execution`-Feld in der HTTP-Response durch, P203d-2 ersetzte den `answer` durch eine LLM-Synthese — das Nala-Frontend ignorierte das `code_execution`-Feld trotzdem komplett. Der User las nur die Synthese-Antwort, sah aber nicht den ausgefuehrten Code, die Roh-Ausgabe oder die Laufzeit. P203d-3 baut den UI-Render: nach `addMessage(reply, 'bot')` rendert `renderCodeExecution(wrapperEl, data.code_execution)` zwei Karten unter dem Bot-Bubble.

**Architektur: Frontend-only, drei Bausteine.** Alles in [`zerberus/app/routers/nala.py`](zerberus/app/routers/nala.py) — kein neues Modul, kein Backend-Touch:

- **CSS-Block** im `<style>`-Bereich (~120 Zeilen): neue Klassen `.code-card`/`.code-card-header`/`.lang-tag`/`.exit-badge` (mit `.exit-ok` gruen / `.exit-fail` rot)/`.exec-meta`/`.code-content` (`overflow-x: auto`, `max-height: 380px`)/`.exec-error-banner`/`.output-card` (default `.collapsed`)/`.output-card-header`/`.code-toggle` (44×44px Touch-Target)/`.output-content` (mit `.output-stderr`-Variant in rot)/`.truncated-marker`. Der Toggle erbt das Kintsugi-Gold-Pattern von `.expand-toggle` (P124).
- **`escapeHtml(s)`** als 3-Zeilen-Helper neben `escapeProjectText` (P201). Delegiert an `escapeProjectText` — gleiche Semantik, eigener Name fuer den XSS-Audit-Test.
- **`renderCodeExecution(wrapperEl, codeExec)`** — neuer JS-Renderer (~80 Zeilen). Liest `codeExec.{language, code, exit_code, stdout, stderr, execution_time_ms, truncated, error}`. Baut Code-Card (Header mit `lang-tag` + `exit-badge` + optional Laufzeit-Meta, Body als `<pre><code>` mit `escapeHtml`-escapeden Code, optional `exec-error-banner` wenn `error != null`). Baut Output-Card nur wenn stdout oder stderr da: Header mit Label (`📤 Ausgabe` bei exit=0, `⚠️ Ausgabe (Fehler)` sonst) + 44×44px Toggle, Body collapsed-by-default, expandiert auf Klick, enthaelt `<pre>`-Bloecke fuer stdout/stderr (escaped) plus optional `truncated-marker` wenn `truncated: true`. Insertion-Punkt: vor dem `.sentiment-triptych`-Element. Visual-Order: bubble → toolbar → code-card → output-card → triptych → export-row.

**`addMessage` retourniert wrapper.** Die Funktion gibt jetzt das DOM-Wrapper-Element zurueck, damit der Caller (sendMessage) den Renderer nachtraeglich einhaengen kann. Backwards-Compat: alle bisherigen Caller (Voice-Input, History-Replay, Late-Fallback) ignorieren den Return-Value, ihr Verhalten bleibt identisch.

**Verdrahtung in `sendMessage`** direkt nach dem Bot-Bubble-Render:

```js
const botWrapper = addMessage(reply, 'bot');
// Patch 203d-3: Code-Card + Output-Card unter Bot-Bubble (fail-quiet).
if (data.code_execution) {
    try { renderCodeExecution(botWrapper, data.code_execution); } catch (_e) {}
}
loadSessions();
```

Fail-quiet: Renderer-Crash darf den Chat-Loop nicht unterbrechen.

**Was P203d-3 bewusst NICHT macht** (kommt mit P206/P207 oder bewusste Leerstelle):

- **Keine Syntax-Highlighting-Library.** Code wird als Plain-Text in `<pre><code>` gerendert — keine Prism.js, keine highlight.js. Bewusst: PWA-Bundle bleibt leichtgewichtig, kein zusaetzlicher Asset-Pfad.
- **Kein Edit-Knopf am Code.** Der Code ist read-only. Nala bleibt Chat-Interface — wer den Code anpassen will, kopiert ihn aus dem Bubble (Toolbar hat `📋`-Button) und schickt eine neue Frage.
- **Keine Re-Run-Funktion.** Code laeuft im Backend ein Mal pro LLM-Call. Wer den gleichen Block neu pruefen will, schickt die Frage erneut (Retry-Button an User-Bubble).
- **Keine Output-Card im Skip-Fall.** Wenn `code_execution.code` leer/fehlend ist (alter Backend, kein aktives Projekt, kein Block, Sandbox disabled), rendert der Renderer NICHTS — Bot-Bubble erscheint normal. Backwards-Compat zu Backends die `code_execution` nicht kennen.
- **Keine SSE-Stream-Frames.** Code-Card und Output-Card erscheinen synchron mit dem Bot-Bubble (Chat ist non-streaming bis P203d-2).
- **Kein Telegram-Pfad.** Huginn (Telegram-Adapter) bleibt auf dem Text-Sandbox-Output via `format_sandbox_result` (P171). Nur `/v1/chat/completions` → Nala-PWA-Renderer bekommt das neue UI.
- **Kein Copy-to-Clipboard-Button am Code.** User kopiert via Browser-Long-Press oder ueber den `📋`-Toolbar-Button (kopiert aber den Synthese-Text, nicht den Card-Code).

**Tests:** 30 in [`test_p203d3_nala_code_render.py`](zerberus/tests/test_p203d3_nala_code_render.py):

- `TestRendererExists` (2) — Funktion definiert + Signatur `(wrapperEl, codeExec)`.
- `TestRendererLiestSchemaFelder` (8) — Renderer liest `code`/`language`/`exit_code`/`stdout`/`stderr`/`truncated`/`error`/`execution_time_ms` aus dem P203d-1-Schema.
- `TestRendererFallbacks` (2) — Null-Check im Eingang, Skip bei leerem Code.
- `TestRendererInsertionPoint` (1) — `insertBefore(.sentiment-triptych)` haelt die Visual-Order.
- `TestSendMessageVerdrahtung` (5) — `addMessage` retournt wrapper, Caller bindet ihn (`= addMessage(reply, 'bot')`), Renderer-Aufruf in `sendMessage` mit `data.code_execution`, Reihenfolge addMessage-vor-Renderer, try/catch.
- `TestXssEscape` (4) — `escapeHtml`-Helper definiert, `escapeProjectText` nicht geloescht (P201-Audit darf nicht brechen), Min-Count 4 `escapeHtml(`-Aufrufe im Renderer (lang+code+stdout+stderr), keine `innerHTML`-Assigns ohne `escapeHtml`.
- `TestCss` (5) — `.code-card`/`.output-card`-Klassen, `.code-toggle` mit `min-height: 44px` UND `min-width: 44px`, `.code-content` mit `overflow-x: auto`, `.output-card.collapsed`-State, `.exit-badge.exit-ok`/`.exit-fail`-Farbcodes.
- `TestJsSyntaxIntegrity` (1, skipped wenn node fehlt) — `node --check` ueber alle inline `<script>`-Bloecke aus `NALA_HTML`. Lesson aus P203b: ein einzelner SyntaxError invalidiert den gesamten Block.
- `TestNalaEndpointSmoke` (1) — `GET /nala/` liefert HTML mit `function renderCodeExecution`, `data.code_execution`, `.code-card`, `.output-card`.

**Logging-Tag:** keiner. Frontend-only-Patch, alle Backend-Logs (`[SANDBOX-203d]`/`[SYNTH-203d-2]`) bleiben aus P203d-1/2.

**Teststand:** 1767 baseline (P203d-2) → **1797 passed** (+30 P203d-3-Tests), 4 xfailed pre-existing, 3 failed pre-existing aus Schuldenliste (`edge-tts`, `test_rag_dual_switch.test_fallback_logic`, `test_patch185_runtime_info` durch lokalen `config.yaml`-Drift `deepseek-v4-pro`), 0 neue Failures.

**Effekt fuer den User:** Bei aktivem Projekt + Code-Block in der Antwort + erfolgreicher Sandbox-Execution sieht Nala jetzt nicht nur die menschenlesbare Synthese (`Der Inhalt der Datei ist 42.`), sondern darunter ZWEI Karten: (a) eine Code-Card mit Sprach-Tag, exit-Status (gruen/rot) und dem ausgefuehrten Code-Block; (b) eine Output-Card mit collapsible stdout/stderr — eingeklappt by default, expandiert auf Tap. Der gesamte Code-Execution-Pfad ist damit von der LLM-Antwort bis ins UI sichtbar.

**Effekt fuer die naechste Coda-Session:** Phase-5a-Ziel #5 ist abgeschlossen. Die naechsten Patches widmen sich anderen Zielen: P205 (RAG-Toast in Hel — Phase-5a-Schuld aus P199), P206 (HitL-Gate vor Code-Execution — Ziel #6 baut auf P203d auf), P207 (Diff-View / Snapshots / Rollback — Ziel #9 + #10), P208 (Spec-Contract / Ambiguitäts-Check — Ziel #8). Helper aus P203d-3 die direkt nutzbar sind: `addMessage` retourniert das Wrapper-Element (P206 HitL-Confirm-Card kann sich nachtraeglich einhaengen), `escapeHtml`-Helper im Frontend (XSS-Schutz fuer alle neuen Karten), `.code-card`/`.output-card`-CSS-Pattern als Vorlage fuer neue Karten.

---

**Patch 203d-2** — Output-Synthese fuer Sandbox-Code-Execution im Chat (Phase 5a #5, Backend-Loop schliesst) (2026-05-03)

Zweiter Sub-Patch der P203d-Aufteilung (P203d-1 Backend-Pfad / P203d-2 Output-Synthese / P203d-3 UI). Schliesst den Backend-Loop von Phase-5a-Ziel #5: nach P203d-1 reichte der Chat-Endpunkt den rohen `SandboxResult` (stdout/stderr/exit_code) als additives `code_execution`-Feld durch — der `answer`-String enthielt aber weiter den Original-Code-Block, ohne menschenlesbare Erklaerung des Outputs. P203d-2 fuegt einen zweiten LLM-Call ein, der Original-Frage + Code + stdout/stderr in eine zusammenfassende Antwort verwandelt und damit den `answer` ersetzt.

**Architektur: Pure-Function-Schicht plus Async-Wrapper.** Neues Modul [`zerberus/modules/sandbox/synthesis.py`](zerberus/modules/sandbox/synthesis.py) — Pattern analog zu `prosody/injector.py` (P204) und `persona_merge.py` (P197):

- `should_synthesize(payload) -> bool` — Trigger-Gate. True wenn `exit_code != 0` (Crash → Erklaerung noetig) ODER `exit_code == 0` UND `stdout` nicht leer (Output → Aufbereitung). False bei `payload is None`, fehlendem `exit_code` oder `exit_code == 0` mit leerem stdout (nichts zu sagen).
- `_truncate(text, limit=5000)` — Bytes-genau truncaten (UTF-8-encoded), ASCII-Marker `\n…[gekuerzt]` am Ende. Multi-Byte-sicher via `decode(errors='ignore')`.
- `build_synthesis_messages(user_prompt, payload) -> list[dict]` — Pure-Function: System-Prompt ("Fasse menschenlesbar zusammen, wiederhole den Code nicht stumpf, erklaere Fehler") plus User-Message mit Original-Frage, fenced Code-Block, fenced stdout, fenced stderr. Marker `[CODE-EXECUTION — Sprache: ... | exit_code: ...]` und `[/CODE-EXECUTION]` substring-disjunkt zu `[PROJEKT-RAG]` (P199), `[PROJEKT-KONTEXT]` (P197), `[PROSODIE]` (P204).
- `synthesize_code_output(user_prompt, payload, llm_service, session_id)` — Async-Wrapper. Trigger-Check, dann LLM-Call via `LLMService.call(messages, session_id)`, Result-Validation (Tuple, non-empty string), Logging-Tag `[SYNTH-203d-2]`. Fail-Open: jeder Crash, leere LLM-Antwort, falsches Result-Format → Returns `None` → Caller behaelt Original-Answer.

**Verdrahtung in `legacy.py::chat_completions`.** Direkt nach dem P203d-1-Block (vor dem Sentiment-Triptychon):

```python
if code_execution_payload is not None:
    try:
        synthesized = await synthesize_code_output(
            user_prompt=last_user_msg,
            payload=code_execution_payload,
            llm_service=llm_service,
            session_id=session_id,
        )
        if synthesized:
            answer = synthesized
    except Exception as _synth_err:
        logger.warning(f"[SYNTH-203d-2] Pipeline-Fehler (fail-open): {_synth_err}")
```

**Reihenfolge-Aenderung — `store_interaction` aufgeteilt.** Vorher (P203d-1): User-Insert + Assistant-Insert + `update_interaction()` als Block VOR der Sandbox-Stelle. Nachher (P203d-2): User-Insert frueh (Eingabe ist endgueltig), Assistant-Insert + `update_interaction()` NACH der Synthese, damit der gespeicherte `answer` der finale Output ist und nicht der Roh-Output mit Code-Block. Falls Synthese skipte oder fehlschlug, ist es der Original-LLM-Output (Backwards-Compat zu P203d-1).

**Was P203d-2 bewusst NICHT macht** (kommt mit P203d-3/P207):

- **Kein UI-Render.** Code-Block + Output-Block in Nala-Frontend ist P203d-3 — separate Endpunkt-Erweiterung mit ggf. SSE-Stream-Frame.
- **Kein zweiter `store_interaction`-Eintrag fuer den Original-Output.** Die `interactions`-Tabelle bekommt nur den finalen `answer`. Wer den Roh-Output braucht, liest das `code_execution`-Feld in der HTTP-Response.
- **Keine Cost-Aggregation in `interactions.cost`.** Der Synthese-Call addiert eigene Tokens, wird aber nicht aufsummiert (Schuld vermerkt; HitL-Token-Tracking-Patch wird das adressieren).
- **Kein Streaming.** `chat_completions` bleibt synchron. SSE `code_execution`/`synth`-Frames sind P203d-3-Thema.
- **Keine writable-Mount-Aenderung.** P203d-1 forciert weiter `writable=False`. Sync-After-Write kommt mit P207.
- **Kein eigener Fehler-Markierer im `answer`.** Bei Synthese-Fail bleibt der Roh-Output mit Code-Block — kein Hinweis "Synthese fehlgeschlagen". Frontend sieht das implizit am `code_execution`-Feld.

**Tests:** 47 in [`test_p203d2_chat_synthesis.py`](zerberus/tests/test_p203d2_chat_synthesis.py):

- `TestShouldSynthesize` (8) — Trigger-Gate: None/Non-Dict, exit_code-Varianten, exit=0+leerer stdout vs. exit=0+stdout-da, exit!=0 mit/ohne stderr.
- `TestTruncate` (5) — Short/empty/at-limit/over-limit, Multi-Byte-UTF-8-safe.
- `TestBuildSynthesisMessages` (9) — Format der zwei Messages, Original-Prompt im User-Msg, Code-Fence, stdout/stderr nur wenn vorhanden, exit_code im Marker, Marker-Disjunktheit, System-Prompt-Inhalte, Truncate bei Mega-Stdout.
- `TestSynthesizeCodeOutput` (8) — Async-Wrapper: Skip-bei-None-payload, Skip-bei-exit0-leer, Happy-Path, exit!=0-Pfad, Fail-Open bei LLM-Crash/leerer-Antwort/Whitespace/Non-Tuple.
- `TestP203d2SourceAudit` (7) — Synthese-Modul existiert, Logging-Tag, Imports in legacy.py, korrekte Args-Reihenfolge im Aufruf, Reihenfolge-Garantie (Synthese VOR Assistant-Store), User-Store FRUEH, Truncate-Limit-Konstante.
- `TestE2ESynthesis` (10) — End-to-End ueber `chat_completions` mit Two-Step-Mock-LLM (Erst-Call: Code-Block, Zweit-Call: Synthese): answer ersetzt bei Happy-Path, Fehler-Erklaerung bei exit!=0, User-Prompt im Synthese-Call, kein Synthese-Call bei Plain-Text/leerem-Output/inaktivem-Projekt/disabled-Sandbox, Original-Answer bleibt bei Synthese-Crash/leerer-Synthese, OpenAI-Schema unangetastet.

**Logging-Tag:** `[SYNTH-203d-2]` (separat von `[SANDBOX-203d]` aus P203d-1, damit Operations-Logs den Synthese-Pfad isoliert beobachten koennen).

**Teststand:** 1720 baseline (P203d-1) → **1767 passed** (+47 P203d-2-Tests), 4 xfailed pre-existing, 3 failed pre-existing aus Schuldenliste (`edge-tts`, `test_rag_dual_switch.test_fallback_logic`, `test_patch185_runtime_info` durch lokalen `config.yaml`-Drift `deepseek-v4-pro`), 0 neue Failures.

**Effekt fuer den User:** Bei aktivem Projekt + Code-Block in der Antwort + erfolgreicher Sandbox-Execution liest Nala statt `\`\`\`python\nprint(2+2)\n\`\`\`` jetzt eine Antwort wie `Das Ergebnis ist 4.` Bei einem Fehler (`exit_code=1`, stderr=`ZeroDivisionError`) liefert Nala eine Fix-Empfehlung anstatt nur den Crash-Output. Das `code_execution`-Feld bleibt zusaetzlich in der HTTP-Response — Frontends koennen den Roh-Output und die Synthese parallel rendern (P203d-3 wird das Layout bauen).

**Effekt fuer die naechste Coda-Session:** P203d-3 kann sofort starten — Synthese-Output kommt im normalen `choices[0].message.content`-Pfad, das `code_execution`-Feld liefert separat den Code-Block + stdout/stderr fuers UI. Frontend-Patch ist orthogonal: Nala-Renderer baut Code-Card + Synthese-Card unter dem Chat-Bubble-Layout. Nach P203d-3 ist Phase-5a-Ziel #5 vollstaendig abgeschlossen.

---

**Patch 203c** — Sandbox-Workspace-Mount + execute_in_workspace (Phase 5a #5, Zwischenschritt) (2026-05-02)

Dritter Sub-Patch der P203-Aufteilung (P203a Workspace-Layout + P203b Hel-Hotfix + P203c Sandbox-Mount). P203a hatte den Working-Tree gelegt (`<data_dir>/projects/<slug>/_workspace/` mit Hardlink+Copy-Fallback) — die Sandbox aus P171 hatte aber explizit "Kein Volume-Mount vom Host" als Sicherheitsregel. P203c erweitert genau diese Stelle um einen kontrollierten Mount-Pfad: `SandboxManager.execute(...)` nimmt jetzt zwei keyword-only Parameter `workspace_mount: Optional[Path] = None` und `mount_writable: bool = False`. Default-Pfad ohne Mount bleibt unverändert (Backwards-Compat für den Huginn-Pipeline-Flow).

**Architektur: Read-Only-Default, Defense-in-Depth zwei-stufig.** Wenn `workspace_mount` gesetzt ist, ergänzt `_run_in_container` die docker-args um `-v <abs>:/workspace[:ro]` plus `--workdir /workspace` — `:ro`-Suffix hängt nur weg, wenn der Caller ausdrücklich `mount_writable=True` setzt. Damit kann der Sandbox-Code Files lesen (Source-Trees, Config, Daten), aber das Workspace nicht von innen verändern, ohne dass die ausführende Schicht das explizit zulässt. P203d wird Read-Write erst bei Code-Generation-Pfaden mit anschließendem `sync_workspace`-Re-Sync nutzen.

**Mount-Validation als Early-Reject.** Vor dem `docker run`-Aufruf prüft `execute()`, dass der Mount-Pfad existiert UND ein Verzeichnis ist — sonst SandboxResult mit `error=...` und Exit-Code -1. Verhindert obskure docker-Fehlermeldungen und macht das Failure-Mode deterministisch testbar (kein Docker-Daemon nötig).

**Convenience-Wrapper `execute_in_workspace` in `projects_workspace.py`.** Der Caller (P203d, künftige CLI) ruft nicht direkt `SandboxManager.execute(workspace_mount=..., ...)`, sondern `execute_in_workspace(project_id, code, language, base_dir, *, writable=False, timeout=None)`. Der Wrapper zieht den Slug aus der DB via `projects_repo.get_project`, baut den Workspace-Pfad via `workspace_root_for(slug, base_dir)`, validiert per `is_inside_workspace(workspace_root, base_dir)` (Defense-in-Depth gegen Slug-Manipulation — falls ein entartetes `slug=../etc` jemals durch den Sanitizer rutscht), legt den Workspace-Ordner an wenn er fehlt (Projekt existiert noch ohne Files), und reicht an `get_sandbox_manager().execute(...)` durch. Returns `None` bei unbekanntem Projekt, deaktivierter Sandbox oder Sicherheits-Reject — damit kann der Caller einheitlich auf `None`-Pfade reagieren (Datei-Fallback wie in P171).

**Was P203c bewusst NICHT macht** (kommt mit P203d/P206):
- **Kein HitL-Gate.** Die HitL-Bestätigung (Phase-5a #6) hängt davor, kommt mit P206. P203c läuft direkt durch, weil RO-Mount + hart-isolierte Sandbox (no-network, read-only-rootfs, no-new-privileges, pids/cpu/memory-limits, kein Volume-Mount sonst) den blast-radius klein halten.
- **Kein Tool-Use-LLM-Pfad.** P203d verdrahtet die Chat-Pipeline (Code-Detection im Output, Sandbox-Roundtrip, Output-Synthese, UI-Render).
- **Kein Sync-After-Write.** Falls jemand `writable=True` aufruft und der Sandbox-Code Files erzeugt: P203d muss `sync_workspace(project_id, base_dir)` hinterher rufen, damit die DB + RAG die Änderungen sehen. P203c liefert nur die Brücke.
- **Keine Image-Pull-Logik.** Healthcheck (P171) bleibt unverändert — Caller muss vorher prüfen.

- **Pure-Function-Schicht:** [`zerberus/modules/sandbox/manager.py::SandboxManager._run_in_container`](zerberus/modules/sandbox/manager.py) — Mount-Block setzt `-v <abs>:/workspace[:ro]` + `--workdir /workspace`, vor `image, *run_args`. Pfad-Resolution via `workspace_mount.resolve(strict=False)` damit Symlinks/8.3-Windows-Namen Docker nicht verwirren. Logging-Tag `[SANDBOX-203c]` für Mount-Stelle.
- **Validation-Schicht:** `execute()` checkt `workspace_mount.exists()` + `.is_dir()` vor `_image_and_command`. Fail-Mode: `SandboxResult(exit_code=-1, error=...)` — kein docker-Aufruf.
- **Convenience:** [`zerberus/core/projects_workspace.py::execute_in_workspace`](zerberus/core/projects_workspace.py) — async, zieht Slug aus DB, validiert workspace_root inside base_dir per `is_inside_workspace`, legt Ordner an, reicht an `get_sandbox_manager().execute(...)` durch. Logging-Tag `[WORKSPACE-203c]`.
- **Tests:** 17 in [`test_p203c_sandbox_workspace.py`](zerberus/tests/test_p203c_sandbox_workspace.py): docker-args-Audit ohne Mount unverändert (Backwards-Compat), Mount-RO-Default, Mount-Writable, Mount-nonexistent → Error, Mount-is-File → Error, Disabled-Sandbox → None auch mit Mount, Blocked-Pattern-Vorrang, execute_in_workspace mit fehlendem Projekt → None, korrekter Mount-Pfad-Passthrough, writable-Passthrough, Workspace-Anlage on-demand, Slug-Traversal-Reject (Defense-in-Depth), Source-Audit Mount-Stelle, Source-Audit execute_in_workspace, Disabled-Sandbox-Passthrough via Convenience, Timeout-Passthrough, Mount-Pfad-resolve()-stable.
- **Logging-Tag:** `[SANDBOX-203c]` (Mount-Setup) und `[WORKSPACE-203c]` (Convenience-Reject).
- **Teststand:** 1685 baseline (P204) → **1705 passed** (+20: 17 neue P203c-Tests + 3 die durch geänderten Default-Pfad jetzt mit gezählt werden), 4 xfailed pre-existing, 0 neue Failures.
- **Effekt für die nächste Coda-Session:** P203d kann sofort starten — `execute_in_workspace(project_id, code, language, base_dir, *, writable=...)` ist die einzige öffentliche API für Workspace-gebundene Code-Execution. Bei Code-Generation reicht: `result = await execute_in_workspace(...)` mit Projekt-ID aus dem aktiven Chat-Header (P201), `result.stdout`/`result.stderr` für die UI-Synthese, ggf. `await sync_workspace(...)` wenn `writable=True`. HitL-Gate wickelt sich später per P206 davor.

---

**Patch 204** — Prosodie-Kontext im LLM (Phase 5a #17, unabhängig einschiebbar) (2026-05-02)

Schließt Phase-5a-Ziel #17: Die Prosodie-Pipeline (P189-193) lieferte ihre Daten bisher nur ans UI-Triptychon (P192) — DeepSeek bekam beim Voice-Input keinen Kontext, das LLM "hörte" die Stimme nicht. P190 hatte zwar einen rudimentären `[Prosodie-Hinweis ...]`-Block hinzugefügt, aber nur Gemma (kein BERT, kein Konsens-Label) und in einem ad-hoc-Format. P204 baut die Brücke richtig: ein markierter `[PROSODIE]...[/PROSODIE]`-Block analog `[PROJEKT-RAG]` (P199), mit BERT-Sentiment und Mehrabian-Konsens.

**Format-Entscheidung: qualitative Labels, keine Zahlen (Worker-Protection P191).** Confidence/Score/Valence/Arousal werden im Konsens-Label verkocht — das LLM bekommt nur menschenlesbare Beschreibungen wie "leicht positiv", "ruhig", "deutlich negativ", "inkongruent — Text positiv, Stimme negativ". Damit kann das Modell die Daten nicht zu Performance-Bewertungen aus Stimmungsdaten missbrauchen. Tests verifizieren die Number-Free-Property mit einem Regex (`\d+\.\d+`, `%`, `\b\d+\b`).

**Architektur: Pure-Function `build_prosody_block` plus Wrapper `inject_prosody_context`.** Pure-Schicht baut den Block-String (lookup-table-based mood/tempo-Übersetzung de, Mehrabian-Konsens-Logik aus `utils.sentiment_display` repliziert), Wrapper hängt am System-Prompt an mit Idempotenz-Check (Marker-Substring im Prompt → kein zweiter Block). BERT-Parameter sind keyword-only und additiv — bestehende P190-Aufrufer ohne BERT bekommen einen Block ohne Sentiment-Text-Zeile (Stimm-only-Pfad).

**Verdrahtung in `legacy.py /v1/chat/completions`.** Der `X-Prosody-Context`-Header (vom Whisper-Endpoint übers Frontend durchgereicht) plus `X-Prosody-Consent: true` schaltet den Block frei. Server-seitig wird BERT auf der letzten User-Message berechnet (fail-open: BERT-Fehler → kein Sentiment-Text-Zeile, Block läuft ohne) und an `inject_prosody_context` durchgereicht. Voice-only-Garantie: der Header existiert nur nach Whisper-Roundtrip (Frontend setzt ihn nicht bei getipptem Text) — defense-in-depth über Stub-Source-Check filtert versehentliche Pseudo-Contexts.

**Mehrabian-Konsens (Pure):** BERT positiv + Prosody-Valenz negativ → `"inkongruent — ..."`. Sonst: Confidence > 0.5 → Stimme dominiert (Stimm-Mood gewinnt), Confidence ≤ 0.5 → BERT-Fallback (`"deutlich/leicht positiv/negativ"` oder `"neutral"`). Schwellen identisch zu `utils/sentiment_display.py` (P192) — UI-Konsens und LLM-Konsens dürfen nicht voneinander abweichen.

- **Pure-Schicht:** [`zerberus/modules/prosody/injector.py::build_prosody_block`](zerberus/modules/prosody/injector.py) plus Helper `_consensus_label`, `_bert_qualitative` und Marker-Konstanten `PROSODY_BLOCK_MARKER` / `PROSODY_BLOCK_CLOSE`. Lookup-Tables `_BERT_LABEL_DE`, `_PROSODY_MOOD_DE`, `_PROSODY_TEMPO_DE`. Schwellen `_BERT_HIGH=0.7`, `_PROSODY_DOMINATES_CONFIDENCE=0.5`, `_MIN_CONFIDENCE_FOR_BLOCK=0.3`.
- **Wrapper:** `inject_prosody_context(system_prompt, prosody_result, *, bert_label=None, bert_score=None)` — Backward-Compat-Signatur (keyword-only-Parameter additiv). Idempotent. Logging `[PROSODY-204]` wenn BERT mitgegeben, `[PROSODY-190]` ohne.
- **Verdrahtung:** [`legacy.py::chat_completions`](zerberus/app/routers/legacy.py) — JSON-Parse + Type-Guard (nur `dict`), BERT-Call in try/except (fail-open), Aufruf mit Keyword-Args.
- **Tests:** 33 in [`test_p204_prosody_context.py`](zerberus/tests/test_p204_prosody_context.py) (`TestBuildProsodyBlock` 9: Marker, Stimme/Tempo, Konsens-mit-BERT, Konsens-ohne-BERT, Stub-Reject, Low-Conf-Reject, None/Wrong-Type, Invalid-Conf, Unknown-Mood-Fallback; `TestWorkerProtectionNoNumbers` 3 parametrisiert: Regex-Check `%`/`\d+\.\d+`/`\b\d+\b`; `TestConsensusLabel` 6: Inkongruenz, Voice-dominiert, BERT-Fallback, Neutral, ohne-BERT, Invalid-Inputs; `TestBertQualitative` 6: positive/negative high/low, neutral, invalid; `TestInjectWithBert` 5: append-mit-BERT, Stub-skip, Low-Conf-skip, leerer-Base, Idempotenz; `TestP204LegacyVerdrahtung` 6 Source-Audit; `TestMarkerUniqueness` 3: Format, Distinct-vom-PROJEKT-RAG/PROJEKT-KONTEXT). Plus 6 nachgeschärfte Tests in [`test_prosody_pipeline.py::TestInjectProsodyContext`](zerberus/tests/test_prosody_pipeline.py) — Format-Assertions auf neue Marker (`PROSODY_BLOCK_MARKER`, `PROSODY_BLOCK_CLOSE`, qualitative Labels statt Zahlen, Konsens-Zeile, Inkongruenz-via-BERT, neuer Idempotenz-Test).
- **Logging-Tag:** `[PROSODY-204]` für BERT-erweiterten Block, `[PROSODY-190]` bleibt für Stimm-only-Pfad.
- **Teststand:** 1645 baseline (P203b) → **1685 passed** (+40), 4 xfailed pre-existing, 2 failed pre-existing aus Schuldenliste (`edge-tts` + `test_rag_dual_switch.test_fallback_logic`, beide nicht blockierend).
- **Effekt für den User:** Bei Voice-Input liest DeepSeek im System-Prompt jetzt einen Block wie `[PROSODIE — Stimmungs-Kontext aus Voice-Input]\nStimme: muede\nTempo: langsam\nSentiment-Text: leicht positiv (BERT)\nSentiment-Stimme: muede (Gemma)\nKonsens: muede\n[/PROSODIE]`. Nala kann den Ton subtil anpassen (zurücknehmen wenn jemand müde klingt, nachfragen bei Stress) ohne plakative "Du klingst traurig!"-Reaktionen. Bei getipptem Text: kein Block, Chat unverändert. Bei deaktiviertem Consent: kein Block. Bei Stub-Pipeline (kein Modell geladen): kein Block.
- **Was P204 bewusst NICHT macht:** Persistierung der Prosodie-Daten in der DB (Worker-Protection — Daten sind one-shot pro Request). Kein neues UI (Triptychon P192 zeigt schon). Keine Pipeline-Änderung (Brücke zum LLM, nicht Refactor von Whisper/Gemma/BERT). Kein Voice-Indicator-Header über `X-Voice-Input` o.ä. — der bestehende `X-Prosody-Context`-Header IST der Voice-Indikator (Frontend setzt ihn nur nach Whisper-Roundtrip), Stub-Source-Check + Consent-Header sind defense-in-depth.

---

**Patch 203b** — Hel-UI-Hotfix (BLOCKER, Chris-Bugmeldung) (2026-05-02)

Behebt einen Blocker in Hel: Die UI rendert, aber NICHTS ist anklickbar — Tabs wechseln nicht, Buttons reagieren nicht, Formulare nicht bedienbar. Nala lief unauffällig. Auffällig erst nach P200/P202 (PWA-Roll-Out + Cache-Wipe), Symptom verschleiert vorher durch Browser-Cache.

Root-Cause: Im `loadProjectFiles`-Renderer (eingeführt mit P196) stand fehlerhaftes Python-Quote-Escaping. Im Python-Source: `+ ',\'' + _escapeHtml(f.relative_path).replace(/'/g, "\\'") + '\')" '` — Python evaluiert die Escape-Sequenzen im Triple-Quoted-String und produziert in der ausgelieferten HTML/JS-Zeile: `+ ',''  + _escapeHtml(...).replace(/'/g, "\'") + '')" '`. JavaScript parst das als `+ ',' '' +` (zwei adjacent String-Literale ohne Operator) und wirft `SyntaxError: Unexpected string`. Ein einziger Syntax-Fehler in einem `<script>`-Block invalidiert den **gesamten** Block — alle Funktionen darin werden nicht definiert, inkl. `activateTab`, `toggleHelSettings`, `loadProjects`, `loadMetrics`. Damit: keine Klicks, keine Tabs, kein nichts. Nala unbetroffen, weil eigener Renderer.

Fix: Inline `onclick="deleteProjectFile(...)"` durch `data-*`-Attribute (`data-project-id`, `data-file-id`, `data-relative-path`) plus Event-Delegation per `addEventListener` ersetzt. Pattern ist immun gegen Quote-Escape-Probleme (Filename geht durch `_escapeHtml` direkt ins Attribut, statt durch eine fragile JS-String-Concat-Kette mit `replace(/'/g, ...)`) und gleichzeitig XSS-sicher.

Warum erst nach P200/P202 sichtbar: Bug existiert seit P196. Bis P200 hatte der Browser eine ältere Hel-Version aus dem HTTP-Cache. Mit P200 (SW-Roll-Out + Cache-v1) wechselte der Cache, mit P202 (SW-v2-Activate + Wipe) wurde alles geräumt — der Browser holte die echte aktuelle Hel-Seite mit dem P196-Bug. Chris' manuelles Unregister + Cache-Wipe + Hard-Refresh hat das Symptom dann deutlich gemacht.

- **Fix:** [`hel.py::loadProjectFiles`](zerberus/app/routers/hel.py) — `<button class="proj-file-delete-btn" data-project-id="..." data-file-id="..." data-relative-path="...">` plus `list.querySelectorAll('.proj-file-delete-btn').forEach(btn => btn.addEventListener('click', () => deleteProjectFile(...)))`. Kein `onclick`-Attribut, kein `replace(/'/g, "\\'")` mehr.
- **Tests:** 10 neue in [`test_p203b_hel_js_integrity.py`](zerberus/tests/test_p203b_hel_js_integrity.py) — drei Source-Audit-Tests gegen das alte Bug-Pattern (`+ ',''`, `onclick="deleteProjectFile(`, `replace(/'/g, "\\'")`), fünf Source-Audit-Tests für die neue Event-Delegation (Klassen-Name + drei data-Attribute + addEventListener-Nähe), ein Smoke-Test gegen den Endpunkt-Output, und ein **JS-Integrity-Test** der ALLE inline `<script>`-Blöcke aus `ADMIN_HTML` extrahiert und mit `node --check` validiert (skipped wenn `node` nicht im PATH). Letzterer hätte den Bug bei P196 sofort gefangen — nachträglich eingebaut als Schutz vor Wiederholung. Plus 1 angepasster bestehender Test (`test_projects_ui::test_file_list_has_delete_button` — Block-Range erweitert + Klassen-Name-Check, weil event-delegation den Block länger macht).
- **Logging:** Kein neuer Tag — Hotfix bleibt unter `[PWA-200]`/`[PWA-202]`-Doku-Klammer im Quelltext.
- **Teststand:** lokal 1635 baseline (P203a) → **1645 passed** (+10), 4 xfailed pre-existing, 2 failed pre-existing aus Schuldenliste.
- **Effekt für den User:** Hel wieder voll bedienbar — Tabs wechseln, Buttons funktionieren, Forms gehen. Beim Laden der Projekte-Tab werden Files mit Lösch-Button korrekt gerendert, Klick auf Lösch-Button feuert die Delete-Bestätigung. Kein Browser-Cache-Reset mehr nötig (Bug-Pattern aus dem HTML raus).
- **Lessons:** (a) JS-Syntax-Errors in inline `<script>`-Blöcken sind silent-killers für die ganze Page — ein Test der `node --check` über alle Inline-Scripts laufen lässt fängt das früh. (b) Inline `onclick` mit String-Concat über benutzergenerierte Daten ist immer fragil — Event-Delegation mit `data-*`-Attributen ist robust und XSS-sicher. (c) Browser-/SW-Caches können Bugs monatelang verschleiern — bei "neuen" Symptomen nach Cache-Wipe immer auch ältere Patches auf rendering-Probleme prüfen.

---

**Patch 203a** — Project-Workspace-Layout (Phase 5a #5, Vorbereitung) (2026-05-02)

Eigenständige Aufteilung von P203 (Code-Execution-Pipeline, Phase-5a-Ziel #5) durch Coda: Das Original-Ziel ist groß (Workspace + Sandbox-Mount + Tool-Use-LLM + UI-Synthese), wird in drei Sub-Patches zerlegt. P203a legt heute das Workspace-Layout — die Sandbox kann Dateien später nur dann sinnvoll mounten, wenn sie an ihrem `relative_path` (statt unter den SHA-Hash-Pfaden) im Filesystem stehen. Pro Projekt entsteht beim Upload/Template-Materialize ein echter Working-Tree unter `<data_dir>/projects/<slug>/_workspace/`.

Architektur-Entscheidung: **Hardlink primär (`os.link`), Copy als Fallback (`shutil.copy2`).** Hardlinks haben gleiche Inode wie der SHA-Storage — kein Plattenplatz-Verbrauch, instantan, atomar. Bei `OSError` (cross-FS, NTFS-without-dev-mode, FAT32, Permission) fällt der Helper auf `shutil.copy2`. Die gewählte Methode wird im Return ausgewiesen (`"hardlink"` / `"copy"` / `None`-bei-noop) und geloggt — auf Windows-Test-Maschinen ohne dev-mode wird damit auch der Copy-Pfad live mitgetestet (Monkeypatch-Test simuliert `os.link`-Failure).

**Atomic via Tempfile + os.replace.** Auch im Workspace, nicht nur im SHA-Storage. Grund: parallele Sandbox-Reads (P203b) dürfen nie ein halb-geschriebenes Workspace-File sehen. Pattern dupliziert (statt Import aus hel.py), weil das Workspace-Modul auch ohne FastAPI-Stack importierbar bleiben muss (Tests, künftige CLI).

**Pfad-Sicherheit zwei-stufig.** `is_inside_workspace(target, root)` resolved beide Pfade und prüft `relative_to` — schützt gegen `../../etc/passwd`-style relative_paths aus alten Datenbanken oder Migrations. Plus: `wipe_workspace` lehnt jeden Pfad ab, der nicht auf `_workspace` endet — verhindert ein versehentliches `wipe_workspace(Path("/"))` bei Slug-Manipulation.

**Best-Effort-Verdrahtung in vier Trigger-Punkten.** Upload-Endpoint (nach `register_file` + nach RAG-Index), Delete-File-Endpoint (nach `delete_file`, mit Slug aus extra `get_project`-Call vor dem DB-Delete), Delete-Project-Endpoint (`wipe_workspace` nach `delete_project`), `materialize_template` (nach jedem `register_file`). Alle vier wickeln den Workspace-Call in try/except — Hauptpfad bleibt grün auch wenn Hardlink/Copy scheitert. Lazy-Import (`from zerberus.core import projects_workspace` im try-Block) wie bei RAG, damit der Helper nicht beim Import-Time geladen wird.

**`sync_workspace` als Komplett-Resync.** Materialisiert alle DB-Files, entfernt Orphans (Files im Workspace, die nicht mehr in der DB sind). Idempotent. Nicht in den Endpoints verdrahtet (Single-File-Trigger reichen), aber als Recovery-API für künftige CLI/Reindex-Endpoint vorhanden — Pre-P203a-Files können damit in den Workspace nachgezogen werden.

**Was P203a bewusst NICHT macht** (kommt mit P203b/c):

- Sandbox-Mount auf den Workspace-Ordner — der bestehende `SandboxManager` (P171) verbietet Volume-Mount explizit ("Kein Volume-Mount vom Host"). P203b muss entweder eine Erweiterung (`workspace_mount: Optional[Path]`) oder eine Schwester-Klasse bauen. Empfehlung: Erweiterung mit Read-Only-Mount per Default, Read-Write nur explizit.
- LLM-Tool-Use-Pfad für Code-Generation — kommt mit P203c.
- Frontend-Render von Code+Output-Blöcken — kommt mit P203c.

- **Pure-Function-Schicht:** [`projects_workspace.py::workspace_root_for`](zerberus/core/projects_workspace.py) + `is_inside_workspace` — keine I/O, deterministisch, Pfad-Sicherheits-Check. Verwendbar in P203b für Mount-Source-Validation.
- **Sync-FS-Schicht:** `materialize_file` (mit Hardlink-primary + Copy-Fallback + Idempotenz via Inode/Size-Check), `remove_file` (räumt leere Eltern bis Workspace-Root), `wipe_workspace` (Sicherheits-Reject auf Wrong-Dirname). Pure-Aber-Mit-FS-I/O.
- **Async DB-Schicht:** `materialize_file_async`, `remove_file_async`, `sync_workspace` — DB-Lookup + Schicht-Wrapper.
- **Verdrahtung:** [`hel.py::upload_project_file_endpoint`](zerberus/app/routers/hel.py), [`hel.py::delete_project_file_endpoint`](zerberus/app/routers/hel.py), [`hel.py::delete_project_endpoint`](zerberus/app/routers/hel.py), [`projects_template.py::materialize_template`](zerberus/core/projects_template.py) — alle Best-Effort mit Lazy-Import.
- **Config-Flag:** [`config.py::ProjectsConfig.workspace_enabled`](zerberus/core/config.py) — Default `True`. Tests können abschalten (siehe `TestWorkspaceDisabled`-Klasse).
- **Logging-Tag:** `[WORKSPACE-203]` — Materialize, Wipe, Sync.
- **Tests:** 36 in [`test_projects_workspace.py`](zerberus/tests/test_projects_workspace.py) (4 Pure-Function inkl. Traversal-Reject + No-IO; 6 materialize_file inkl. nested-dirs + idempotent + missing-source + Copy-Fallback via monkeypatched `os.link`; 5 remove_file inkl. cleans-empty-parents + keeps-non-empty + traversal-reject; 3 wipe_workspace inkl. rejects-wrong-dirname; 4 sync_workspace inkl. removes-orphans; 3 async-Wrapper; 4 Endpoint-Integration mit echten Hel-Endpoints; 1 workspace_disabled-Pfad; 5 Source-Audit-Tests für die vier Verdrahtungs-Stellen).
- **Teststand:** lokal 1599 baseline → **1635 passed** (+36), 4 xfailed pre-existing, 2 failed pre-existing aus Schuldenliste (`edge-tts` + `test_rag_dual_switch.test_fallback_logic`, beide nicht blockierend).
- **Effekt für die nächste Coda-Session:** P203b kann sofort starten — `workspace_root_for(slug, base_dir)` liefert den Mount-Pfad, `is_inside_workspace` validiert User-Inputs, `sync_workspace` ist Recovery-API. P203b's Job ist nur noch: `SandboxManager.execute` um `workspace_mount` erweitern + `execute_in_workspace`-Convenience + Tests.

---

**Patch 202** — PWA-Auth-Hotfix: SW-Navigation-Skip + Cache-v2 (2026-05-02)

Behebt einen kritischen Bug aus P200: Hel war im Browser nur noch als JSON-Antwort `{"detail":"Not authenticated"}` sichtbar — kein Basic-Auth-Prompt mehr. Nala und Huginn unauffällig. Ursache: Der von P200 eingeführte Service-Worker fängt im Scope `/hel/` ALLE GET-Requests ab und macht `event.respondWith(fetch(event.request))`. Bei navigierenden Top-Level-Anfragen liefert der SW damit die 401-Response unverändert an die Page zurück — und der Browser ignoriert in diesem Fall den `WWW-Authenticate: Basic`-Header, der normalerweise den nativen Auth-Prompt triggert. Ergebnis: User sah JSON statt Login-Dialog.

Architektur-Entscheidung: **Navigation-Requests gar nicht erst abfangen.** Statt nachträglich auf 401 zu reagieren oder Auth-Header zu injecten, returnt der SW jetzt früh wenn `event.request.mode === 'navigate'`. Die Navigation läuft durch den nativen Browser-Stack — inklusive Auth-Prompt, HTTPS-Indikator, Mixed-Content-Warnung, etc. Das ist ohnehin sauberere PWA-Hygiene: SWs cachen statische Assets, nicht HTML-Pages.

**Cache-Versions-Bump auf -v2.** Damit der activate-Hook der neuen SW-Version den verseuchten v1-Cache räumt, der noch /hel/-Navigations-Antworten enthalten könnte. Da der SW selbst per `Cache-Control: no-cache` ausgeliefert wird, bekommt jeder Browser den neuen SW beim nächsten Reload, ohne manuelles Eingreifen — der activate räumt dann automatisch.

**APP_SHELL ohne Root-Pfad.** Vorher enthielt `HEL_SHELL` den Eintrag `"/hel/"` — beim install-Hook versuchte der SW per `cache.addAll(APP_SHELL)` den Hel-Root zu cachen, was wegen Basic-Auth mit 401 fehlschlug. `Promise.all` rejected dann komplett, die `.catch(() => null)`-Klausel im Code swallow den Fehler still, aber semantisch falsch. Jetzt sind die SHELL-Listen rein statische Assets, die alle 200 liefern — der precache läuft sauber durch.

**Side-Effect:** HTML-Reload geht jetzt immer übers Netz (kein Offline-Modus für die Hauptseite). Für den Heimserver-Use-Case akzeptabel — ohne laufenden Server gibt es ohnehin keinen sinnvollen Betrieb (Chat-Endpoints, Whisper, Sandbox sind alle online-only). Wenn später ein echter Offline-Modus gewünscht ist, müsste man den Approach umkrempeln (cache-first für Navigation, mit eigenem Auth-Handling) — aktuell nicht relevant.

- **SW-Template:** [`pwa.py::SW_TEMPLATE`](zerberus/app/routers/pwa.py) — drei Zeilen früh-return für `event.request.mode === 'navigate'`, vor dem `respondWith`-Block.
- **Cache-Names:** `nala-shell-v1` → `nala-shell-v2`, `hel-shell-v1` → `hel-shell-v2` in den Endpoint-Funktionen `nala_service_worker` + `hel_service_worker`.
- **APP_SHELL:** `NALA_SHELL` und `HEL_SHELL` ohne Root-Pfad-Eintrag — nur statische Assets (`/static/css/...`, `/static/favicon.ico`, `/static/pwa/*.png`).
- **Tests:** 5 neue in [`test_pwa.py`](zerberus/tests/test_pwa.py) — Pure-Function-Test für navigation-skip im SW-Body, drei Shell-Lists-Tests (kein Root-Pfad in Nala/Hel), und ein End-to-End-Test mit `TestClient` der `verify_admin` ohne Credentials anpingt und den `WWW-Authenticate: Basic`-Header verifiziert. Plus 3 angepasste bestehende Tests (Cache-Name v2 statt v1, Asset-Audit auf Static-Assets statt Root-Pfad).
- **Logging:** Kein neuer Tag — die SW-Korrektur ist klein-genug für `[PWA-200]`/`[PWA-202]` als Doku-Hinweis im Quelltext-Header. Kein Server-seitiger Log-Output.
- **Teststand:** 1594 → **1602 passed** (+8), 4 xfailed pre-existing.
- **Effekt für den User:** `/hel/` zeigt wieder den nativen Browser-Auth-Prompt. Nach Login funktioniert Hel-UI komplett wie vor P200. Bestehende SW-v1-Installationen werden automatisch auf v2 hochgezogen, sobald der User `/hel/` neu lädt.

---

**Patch 201** — Phase 5a #4: Nala-Tab "Projekte" + Header-Setter (2026-05-02)

Schließt Phase-5a-Ziel #4 ("Dateien kommen ins Projekt") komplett ab — der Backend-Teil war seit P196 da, der Indexer seit P199, jetzt verdrahtet P201 das letzte Stück: Nala-User können vom Chat aus ein aktives Projekt auswählen, und ab da fließt der Persona-Overlay (P197) und das Datei-Wissen (P199-RAG) automatisch in jede Antwort. Vorher war diese Kombination nur über externe Clients (curl, SillyTavern, eigene Skripte) erreichbar — die Hel-CRUD-UI (P195) konnte zwar Projekte anlegen und Dateien hochladen, aber kein User in Nala konnte sie aktivieren.

Architektur-Entscheidung: **Eigener `/nala/projects`-Endpoint statt Wiederverwendung von `/hel/admin/projects`.** Drei Gründe: (a) Hel-CRUD ist Basic-Auth-gated, Nala-User haben aber JWT (zwei Auth-Welten, kein Bridge nötig). (b) Nala-User darf NIEMALS `persona_overlay` sehen — das ist Admin-Geheimnis, kann Prompt-Engineering-Spuren oder interne Tonfall-Hints enthalten, die der User nicht beeinflussen können soll. Der Endpoint slimmed das Response-Dict auf `{id, slug, name, description, updated_at}`. (c) Archivierte Projekte werden hier per Default ausgeblendet — der User soll keine "alten" Projekte sehen, die er nicht mehr nutzen soll.

**Header-Injektion zentral in `profileHeaders()`.** Statt nur den Chat-Fetch zu modifizieren, hängt P201 den `X-Active-Project-Id`-Header in die zentrale Helper-Funktion, die ALLE Nala-Calls verwenden (Chat, Voice, Whisper, Profile-Endpoints). Damit ist garantiert, dass der Projekt-Kontext konsistent gegated ist — keine Möglichkeit, dass ein neuer Endpoint den Header "vergisst".

**State in zwei localStorage-Keys.** `nala_active_project_id` (numerisch) für die Header-Injektion, `nala_active_project_meta` (JSON: id+slug+name) für den Header-Chip-Renderer ohne Re-Fetch. Beim Logout (handle401) wird beides bewusst NICHT gelöscht — der nächste Login bekommt das Projekt automatisch zurück, was für den typischen Use-Case (gleicher User, gleicher Browser) genau richtig ist.

**Zombie-ID-Schutz.** Wenn `loadNalaProjects` ein Projekt nicht mehr in der Liste findet (gelöscht oder archiviert in der Zwischenzeit), wird die aktive Auswahl automatisch geräumt. Sonst hängt der Header-Chip an einer Zombie-ID und das Backend würde einen non-existing-project-Header bekommen (was P197 zwar gracefully ignoriert, aber sauberer ist client-seitig zu räumen).

**UI-Verdrahtung minimal-invasiv.** Neuer Settings-Tab "📁 Projekte" zwischen "Ausdruck" und "System" — bestehende Tab-Mechanik wird wiederverwendet, kein neues Modal, kein Sidebar-Tab. Lazy-Loading: `loadNalaProjects()` läuft erst, wenn der User auf den Tab klickt, nicht beim Modal-Öffnen — spart Roundtrip wenn der User nur Theme ändern will. Active-Project-Chip im Chat-Header, klick öffnet Settings + springt direkt auf den Projekte-Tab. Chip nur sichtbar wenn ein Projekt aktiv ist.

**XSS-Schutz im Renderer.** Der Listen-Renderer rendert User-eingegebene Felder (name, slug, description) — alle drei laufen durch `escapeProjectText()` (wandelt `&`, `<`, `>`, `"`, `'` in HTML-Entities). Source-Audit-Test zählt mindestens drei `escapeProjectText`-Aufrufe in `renderNalaProjectsList`, damit ein vergessener Aufruf in zukünftigen Refactorings sofort auffällt.

- **Backend-Endpoint:** [`nala.py::nala_projects_list`](zerberus/app/routers/nala.py) — `GET /nala/projects`, JWT-pflichtig via `request.state.profile_name`, ruft `projects_repo.list_projects(include_archived=False)` und slimmed das Response-Dict.
- **UI-Tab:** Settings-Modal um vierten Tab "📁 Projekte" erweitert. Panel mit Aktiv-Anzeige, Aktualisieren-Button, Auswahl-löschen-Button, Listen-Container.
- **Header-Chip:** [`nala.py`](zerberus/app/routers/nala.py) — `<span id="active-project-chip" class="active-project-chip">` neben dem Profile-Badge im main-header. Goldener Pill-Border, klick öffnet Settings + Projects-Tab.
- **JS-Funktionen:** `getActiveProjectId`, `getActiveProjectMeta`, `setActiveProject`, `clearActiveProject`, `selectActiveProjectById`, `renderActiveProjectChip`, `renderNalaProjectsActive`, `renderNalaProjectsList`, `escapeProjectText`, `loadNalaProjects` — kompletter Lifecycle.
- **Header-Injektion:** [`nala.py::profileHeaders`](zerberus/app/routers/nala.py) — drei zusätzliche Zeilen, die `X-Active-Project-Id` setzen wenn aktiv. Damit wirkt es auf ALLE Nala-Calls, nicht nur Chat.
- **Lazy-Load:** [`switchSettingsTab`](zerberus/app/routers/nala.py) ruft `loadNalaProjects()` wenn der Projects-Tab aktiviert wird.
- **CSS:** `.active-project-chip` mit gold-border, transparent-Hintergrund, hover-Glow, max-width + ellipsis für lange Slugs.
- **Tests:** 21 neue in [`test_nala_projects_tab.py`](zerberus/tests/test_nala_projects_tab.py) (6 Endpoint inkl. 401, archived-versteckt, persona_overlay-NICHT-im-Response, minimal-Felder; 11 Source-Audit für Tab/Panel/Chip/JS/Header-Injektion/Lazy-Load; 2 XSS-Schutz inkl. Min-Count escapeProjectText; 1 Zombie-ID-Schutz; 1 Tab-Lazy-Load). Plus 1 nachgeschärfter Test in `test_settings_umbau.py` (alter `openSettingsModal()`-Proxy-Test → spezifischer 🔧-Emoji + icon-btn-Pattern-Check, weil P201 erlaubt openSettingsModal im Header NUR via Project-Chip).
- **Teststand:** 1572 → **1594 passed** (+22), 4 xfailed pre-existing, 2 pre-existing Failures (SentenceTransformer-Mock + edge-tts-Install — bekannte Schulden).
- **Phase 5a Ziel #4:** ✅ Dateien kommen ins Projekt (komplett: P196 Upload + P199 Index + P201 Nala-Auswahl). Damit sind die Ziele #1, #2, #3, #4, #16 durch. Nächste sinnvolle Patches: #5 (Code-Execution P202).

**Patch 200** — Phase 5a #16: PWA für Nala + Hel (2026-05-02)

Schließt Phase-5a-Ziel #16 ("Nala + Hel als PWA") ab. Beide Web-UIs lassen sich auf iPhone (Safari → "Zum Home-Bildschirm") und Android (Chrome → "App installieren") als eigenständige Apps installieren — Browser-Chrome verschwindet, Splash-Screen + Theme-Color stimmen, Icon im Kintsugi-Stil (Gold auf Blau für Nala, Rot auf Anthrazit für Hel). Kein Offline-Modus, keine Push-Notifications, kein Background-Sync — alles bewusst weggelassen, weil der Heimserver eh laufen muss und Huginn die Push-Schiene besetzt.

Architektur-Entscheidung: **Eigener Router `pwa.py` statt Erweiterung des Hel/Nala-Routers.** Hintergrund: `hel.router` hat router-weite Basic-Auth-Dependency (`dependencies=[Depends(verify_admin)]`), die jeden Endpoint dort gated. Würde man `/hel/manifest.json` und `/hel/sw.js` in `hel.router` definieren, würde der Browser beim Manifest-Fetch eine Auth-Challenge bekommen und der Install-Prompt nie erscheinen. Lösung: separater `pwa.router` ohne Dependencies, in `main.py` VOR `hel.router` eingehängt — gleiche URL-Pfade, aber andere Auth-Policy. FastAPI matcht die URL-Pfade global: weil `hel.router` keine Route für `/manifest.json` oder `/sw.js` definiert, gibt es keinen Konflikt; `pwa.router` gewinnt durch frühere Registrierung trotzdem die Race-Condition für eventuell hinzukommende Routes.

**Service-Worker-Scope-Trick:** Der Browser begrenzt den SW-Scope per Default auf den Pfad, von dem die SW-Datei ausgeliefert wird. `/nala/sw.js` → scope `/nala/`, `/hel/sw.js` → scope `/hel/`. Damit braucht es KEINEN `Service-Worker-Allowed`-Header und keine Root-Position für die SW-Datei. Genau die richtige Granularität: die Nala-PWA cacht nur Nala-URLs, die Hel-PWA nur Hel-URLs.

**Manifeste pro App, nicht ein gemeinsames:** Beide Apps brauchen separate Icons, Theme-Colors, Namen — zwei `manifest.json`-Endpoints sind sauberer als Conditional-Logic in einem Manifest. Konstanten `NALA_MANIFEST` und `HEL_MANIFEST` in `pwa.py` sind Pure-Python-Dicts, direkt als Tests ohne Server konsumierbar.

**Icon-Generierung deterministisch via PIL.** Skript `scripts/generate_pwa_icons.py` zeichnet 4 PNGs (Nala 192/512, Hel 192/512) — dunkler Hintergrund, großer goldener/roter Initial-Buchstabe, drei dünne Kintsugi-Adern in der Akzentfarbe als Bruchnaht-Anspielung. Deterministisch (kein RNG), damit Re-Runs bytes-identische PNGs erzeugen und der Git-Diff sauber bleibt. Fonts werden aus systemweiten Serif-Kandidaten (Georgia/Times/DejaVuSerif/Arial) gesucht, mit PIL-Default-Font als Fallback.

**Service-Worker-Logik minimal:** Install precacht App-Shell-Liste (HTML-Page + shared-design.css + favicon + Icons), Activate räumt alte Caches, Fetch macht network-first mit cache-fallback (damit Updates direkt durchgehen). Non-GET-Requests passen unverändert durch. Kein Background-Sync, kein Push-Listener.

**Wirkung im HTML:** `<head>` beider Templates erweitert um `<link rel="manifest">`, `<meta name="theme-color">`, vier Apple-Mobile-Web-App-Meta-Tags, zwei Apple-Touch-Icons (192/512). Service-Worker-Registrierung als kleines `<script>` vor `</body>`: Feature-Detection, `load`-Listener, `register('/nala/sw.js', { scope: '/nala/' })`, Catch loggt nach Console (kein UI-Block beim SW-Fail).

- **Neuer Router:** [`zerberus/app/routers/pwa.py`](zerberus/app/routers/pwa.py) — vier Endpoints (`/nala/manifest.json`, `/nala/sw.js`, `/hel/manifest.json`, `/hel/sw.js`), keine Auth-Dependencies. Pure-Function-Schicht: `render_service_worker(cache_name, shell)` rendert SW-JS aus Template + Cache-Name + Shell-URL-Liste. Konstanten `NALA_MANIFEST`, `HEL_MANIFEST`, `NALA_SHELL`, `HEL_SHELL`.
- **Verdrahtung in [`main.py`](zerberus/main.py):** `from zerberus.app.routers import legacy, nala, orchestrator, hel, archive, pwa`, dann `app.include_router(pwa.router)` als ERSTER `include_router`-Call (vor allen anderen). Kommentar erklärt warum die Reihenfolge zwingend ist.
- **HTML-Verdrahtung Nala:** [`nala.py::NALA_HTML`](zerberus/app/routers/nala.py) `<head>` um sieben Tags erweitert (Manifest-Link, Theme-Color #0a1628, Apple-Capable yes, Apple-Status-Bar black-translucent, Apple-Title "Nala", zwei Apple-Touch-Icons). SW-Registrierung als 8-Zeilen-Script vor `</body>`.
- **HTML-Verdrahtung Hel:** [`hel.py::ADMIN_HTML`](zerberus/app/routers/hel.py) analog mit Theme-Color #1a1a1a, Apple-Title "Hel", Hel-Icons. SW-Registrierung mit Scope `/hel/`.
- **Icons:** vier PNGs unter `zerberus/static/pwa/{nala,hel}-{192,512}.png`, generiert via `scripts/generate_pwa_icons.py`. Skript ist Repo-Bestandteil, damit Icons reproduzierbar sind und das Theme später zentral angepasst werden kann.
- **Logging:** Kein eigener Tag — SW-Fehler landen browser-seitig in `console.warn` mit Tag `[PWA-200]`. Server-seitig sind die Endpoints stumm (Standard-Access-Log reicht).
- **Tests:** 39 neue Tests in [`test_pwa.py`](zerberus/tests/test_pwa.py) — 5 Pure-Function-Tests für `render_service_worker` (Cache-Name, Shell-URLs, alle drei Event-Listener, skipWaiting/clients.claim, GET-only-Caching), 6 Manifest-Dict-Tests (Pflichtfelder, 192+512-Icons pro App, Themes unterscheiden sich, JSON-serialisierbar), 4 Endpoint-Tests (Status 200, korrekte Media-Types, Body-Inhalte, Cache-Control), 8 Source-Audit-Tests pro HTML (Manifest-Link, Theme-Color, alle Apple-Tags, SW-Registrierung mit korrektem Scope), 4 Icon-Existenz-Tests (PNG-Magic-Bytes), 3 Routing-Order-Tests (pwa-Import, pwa-Include, pwa-VOR-hel), 2 Generator-Skript-Tests.
- **Teststand:** 1533 → **1572 passed** (+39), 4 xfailed pre-existing, 2 pre-existing Failures (SentenceTransformer-Mock + edge-tts-Install — bekannte Schulden).
- **Phase 5a Ziel #16:** ✅ Nala + Hel als PWA. Unabhängiges Ziel, blockiert nichts. Damit sind die Ziele #1, #2, #3 und #16 durch. Nächste sinnvolle Patches: #5 (Code-Execution P201) oder #4 abschließen via Nala-Tab "Projekte" (P202).

**Patch 199** — Phase 5a #3: Projekt-RAG-Index (2026-05-02)

Schließt Phase-5a-Ziel #3 ("Projekte haben eigenes Wissen") ab. Jedes Projekt bekommt einen eigenen, isolierten Vektor-Index unter `data/projects/<slug>/_rag/{vectors.npy, meta.json}` — der globale RAG-Index in `modules/rag/router.py` bleibt unberührt. Damit kann das LLM beim aktiven Projekt (P197 `X-Active-Project-Id`-Header) auf Inhalte aus den Projektdateien zugreifen, ohne dass projekt-spezifische Chunks den globalen Memory-Index verschmutzen.

Architektur-Entscheidung: **Pure-Numpy-Linearscan statt FAISS.** Per-Projekt-Indizes sind klein (typisch 10–2000 Chunks). Ein `argpartition` auf einem `(N, 384)`-Array ist auf der Größenordnung schneller als FAISS-Setup-Overhead und macht Tests dependency-frei (kein faiss-Mock nötig). Persistenz als `vectors.npy` (float32) + `meta.json` (Liste, gleiche Reihenfolge). Atomare Writes via `tempfile.mkstemp` + `os.replace`. Eine FAISS-Migration ist trivial nachrüstbar, falls Projekte signifikant >10k Chunks bekommen — aber das ist heute nicht der Bottleneck.

Embedder: **MiniLM-L6-v2 (384 dim)** als Default, lazy-loaded — kompatibel mit dem Legacy-Globalpfad und ohne sprach-spezifisches Setup. Tests monkeypatchen `_embed_text` mit einem Hash-basierten 8-dim-Pseudo-Embedder; der echte SentenceTransformer wird in Unit-Tests nie geladen. Wenn der globale Index irgendwann komplett auf Dual umsteigt, kann man den Per-Projekt-Pfad mit demselben Modell betreiben — das ist eine reine Konfig-Änderung im Wrapper.

Chunker-Reuse: Code-Files (.py/.js/.ts/.html/.css/.json/.yaml/.sql) gehen durch den existierenden `code_chunker.chunk_code` (P122 — AST/Regex-basiert). Prosa (.md/.txt/Default) durch einen neuen lokalen Para-Splitter mit weichen Absatz-Grenzen (max 1500 Zeichen, snap an Doppel-Newline, Sentence-Split-Fallback für überlange Absätze). Bei Python-SyntaxError im Code-Pfad: Fallback auf Prose, damit kaputte Dateien trotzdem indexiert werden.

Idempotenz: Pro `file_id` höchstens ein Chunk-Set im Index. Beim Re-Index wird der alte Block für die file_id zuerst entfernt — gleicher `sha256` ergibt funktional dasselbe Ergebnis (Hash-Embedder ist deterministisch), anderer `sha256` ersetzt den alten Block. Das vermeidet Doubletten beim Re-Upload mit gleichem `relative_path`.

Trigger-Punkte: (a) `upload_project_file_endpoint` NACH `register_file` — neuer File wandert direkt in den Index. (b) `materialize_template` ruft am Ende `index_project_file` für jede neu angelegte Skelett-Datei — Template-Inhalte sind sofort retrievbar. (c) `delete_project_file_endpoint` ruft `remove_file_from_index` NACH dem DB-Delete — keine stale Treffer mehr. (d) `delete_project_endpoint` löscht den ganzen `_rag/`-Ordner. Alle Trigger sind Best-Effort: Indexing-Fehler brechen den Hauptpfad NICHT ab; der Eintrag steht in der DB, der Index lässt sich später nachziehen.

Wirkung im Chat: Nach P197 (Persona-Merge), nach P184 (`_wrap_persona`), nach P185 (Runtime-Info), nach P118a (Decision-Box) und nach P190 (Prosodie), aber VOR `messages.insert(0, system)`. Der Block beginnt mit `[PROJEKT-RAG — Kontext aus Projektdateien]` als eindeutigem Marker, gefolgt von einer kurzen Anweisung an das LLM und Top-K Hits in Markdown-Sektionen pro File. Best-Effort: jeder Fehler (Embedder fehlt, Index kaputt) → kein Block, Chat läuft normal weiter. Logging-Tag `[RAG-199]` mit `project_id`/`slug`/`chunks_used`. Pro Chat-Request höchstens ein Embed-Call (User-Query) + ein Linearscan über den Projekt-Index — Latenz ~10ms typisch.

Feature-Flags: `ProjectsConfig.rag_enabled: bool = True` (kann via config.yaml ausgeschaltet werden, nützlich für Tests/Setups ohne `sentence-transformers`). `ProjectsConfig.rag_top_k: int = 5` (max Anzahl Chunks pro Query, vom Chat-Endpoint genutzt). `ProjectsConfig.rag_max_file_bytes: int = 5 * 1024 * 1024` (5 MB — drüber: skip beim Indexen, weil's typisch Bilder/Archive sind).

Defensive Behaviors: leere Datei → skip mit `reason="empty"`, binäre Datei (UTF-8-Decode-Fehler) → skip mit `reason="binary"`, Datei zu groß → skip mit `reason="too_large"`, Bytes nicht im Storage → skip mit `reason="bytes_missing"`, Embed-Fehler → skip mit `reason="embed_failed"`, Embedder-Dim-Wechsel zwischen Sessions → Index wird komplett neu aufgebaut (Dim-Mismatch im `top_k_indices` liefert leere Ergebnisliste statt Crash). Inkonsistenter Index (nur eine der zwei Dateien existiert) → leere Basis, der nächste `index_project_file`-Call baut sauber auf.

- **Neuer Helper:** [`zerberus/core/projects_rag.py`](zerberus/core/projects_rag.py) — Pure-Functions `_split_prose`, `chunk_file_content`, `top_k_indices`, `format_rag_block`. File-I/O `load_index`, `save_index`, `remove_project_index`, `index_paths_for`. Embedder-Wrapper `_embed_text` (lazy MiniLM-L6-v2). Async `index_project_file`, `remove_file_from_index`, `query_project_rag`. Konstanten `RAG_SUBDIR`, `VECTORS_FILENAME`, `META_FILENAME`, `PROJECT_RAG_BLOCK_MARKER`, `DEFAULT_EMBED_MODEL`, `DEFAULT_EMBED_DIM`.
- **Verdrahtung Hel:** [`hel.py::upload_project_file_endpoint`](zerberus/app/routers/hel.py) ruft NACH `register_file` `await projects_rag.index_project_file(project_id, file_id, base)` auf — Response erweitert um `rag: {chunks, skipped, reason}`. [`hel.py::delete_project_file_endpoint`](zerberus/app/routers/hel.py) ruft `remove_file_from_index` — Response erweitert um `rag_chunks_removed`. [`hel.py::delete_project_endpoint`](zerberus/app/routers/hel.py) merkt den Slug VOR dem Delete und ruft `remove_project_index` danach.
- **Verdrahtung Materialize:** [`projects_template.py::materialize_template`](zerberus/core/projects_template.py) ruft am Ende JEDES erfolgreichen `register_file` `await projects_rag.index_project_file(project_id, registered_id, base_dir)` auf — frisch materialisierte Skelett-Files sind sofort im Index.
- **Verdrahtung Chat:** [`legacy.py::chat_completions`](zerberus/app/routers/legacy.py) — nach P190 (Prosodie), VOR `messages.insert(0, system)`: wenn `active_project_id` + `project_slug` + `last_user_msg` + `rag_enabled`, dann `query_project_rag` und `format_rag_block` an den `sys_prompt` anhängen.
- **Feature-Flags:** `ProjectsConfig.rag_enabled: bool = True`, `rag_top_k: int = 5`, `rag_max_file_bytes: int = 5 * 1024 * 1024` — alle Defaults im Pydantic-Modell (config.yaml gitignored).
- **Logging:** Neuer `[RAG-199]`-Tag mit `slug`/`file_id`/`path`/`chunks`/`total` beim Indexen, `chunks_used` beim Chat-Query, `chunks_removed` beim Delete. Inkonsistenter/kaputter Index → WARN; Embed/Query-Fehler → WARN mit Exception-Text.
- **Tests:** 46 neue Tests in [`test_projects_rag.py`](zerberus/tests/test_projects_rag.py) — 5 Prose-Splitter-Edge-Cases, 5 Chunk-File-Cases (Code/Markdown/Unknown-Extension/SyntaxError-Fallback/Empty), 5 Top-K-Cases (leer/k=0/sortiert/cap-at-size/Dim-Mismatch), 4 Save-Load-Roundtrip + Inconsistent + Corrupted, 2 Remove-Index, 7 Index-File-Cases (Markdown/Idempotent/Empty/Binary/Too-Large/Bytes-Missing/Rag-Disabled), 2 Remove-File + Drop-Empty-Index, 4 Query-Cases (Hit/Empty-Query/Missing-Project/Missing-Index), 2 Format-Block, 3 End-to-End via Upload/Delete-File/Delete-Project, 1 Materialize-Indexes-Templates, 6 Source-Audit. `fake_embedder`-Fixture mit Hash-basiertem 8-dim-Pseudo-Embedder verhindert das Laden des echten SentenceTransformer in Unit-Tests.
- **Teststand:** 1487 → **1533 passed** (+46), 4 xfailed (pre-existing), 2 pre-existing Failures (SentenceTransformer-Mock + edge-tts-Install — bekannte Schulden).
- **Phase 5a Ziel #3:** ✅ Projekte haben eigenes Wissen. Damit sind die Ziele #1 (Backend P194 + UI P195), #2 (Templates P198) und #3 (RAG-Index) durch. Decision 3 (Persona-Merge-Layer) seit P197 aktiv. Datei-Upload aus Hel-UI (P196) seit dem auch indexiert. Nächste sinnvolle Patches: #5 (Code-Execution P200) oder #4 abschließen via Nala-Tab "Projekte" (P201).

**Patch 198** — Phase 5a #2: Template-Generierung beim Anlegen (2026-05-02)

Schließt Phase-5a-Ziel #2 ("Projekte haben Struktur") ab. Ein neu angelegtes Projekt startete bisher leer — der User mußte selbst eine Struktur hochladen, bevor das LLM überhaupt etwas zu lesen hatte. P198 generiert beim Anlegen zwei Skelett-Dateien: `ZERBERUS_<SLUG>.md` als Projekt-Bibel (analog zu `ZERBERUS_MARATHON_WORKFLOW.md` mit Sektionen "Ziel", "Stack", "Offene Entscheidungen", "Dateien", "Letzter Stand") und ein kurzes `README.md`. Inhalt rendert die Project-Daten ein (Name, Slug, Description, Anlegedatum) — der User hat sofort einen sinnvollen Ausgangspunkt.

Architektur-Entscheidung: Templates landen im SHA-Storage (`data/projects/<slug>/<sha[:2]>/<sha>` — gleiche Konvention wie P196-Uploads), DB-Eintrag in `project_files` mit lesbarem `relative_path`. Damit erscheinen Templates nahtlos in der Hel-Datei-Liste, sind im RAG-Index (P199) indexierbar und in der Code-Execution-Pipeline (P200) sichtbar — ohne Sonderpfad. Pure-Python-String-Templates (kein Jinja, weil das den Stack nicht rechtfertigt). Render-Funktionen sind synchron + I/O-frei und unit-bar; Persistenz liegt separat in der async `materialize_template`.

Idempotenz: Existierende `relative_path`-Einträge werden NICHT überschrieben. Wenn der User in einer früheren Session schon eigene Inhalte angelegt hat, bleiben die unangetastet. Helper liefert nur die TATSÄCHLICH neu angelegten Files zurück — leer, wenn alles schon existiert. Die UNIQUE-Constraint `(project_id, relative_path)` aus P194 wäre der zweite Fallback, aber wir prüfen vorher explizit über `list_files`.

Best-Effort-Verdrahtung: Wenn `materialize_template` crasht (Disk-Full, DB-Lock, was auch immer), bricht das Anlegen NICHT ab. Der Projekt-Eintrag steht, der User sieht eine 200-Antwort, Templates lassen sich notfalls nachgenerieren oder per Hand anlegen. Crash-Path mit Source-Audit-Test verifiziert.

Git-Init bewusst weggelassen: Der SHA-Storage ist kein Working-Tree (Bytes liegen unter Hash-Pfaden, nicht unter `relative_path`). `git init` ergibt erst Sinn mit einem echten `_workspace/`-Layout, das mit der Code-Execution-Pipeline (P200, Phase 5a #5) kommt. Bis dahin: kein halbgares Git-Init, das später wieder umgebogen werden müßte.

- **Neuer Helper:** [`zerberus/core/projects_template.py`](zerberus/core/projects_template.py) — `render_project_bible(project, *, now=None)` + `render_readme(project)` als Pure-Functions, `template_files_for(project, *, now=None)` als Komposit, `materialize_template(project, base_dir, *, dry_run=False, now=None)` als async DB+Storage-Schicht, `_write_atomic()` als lokale Kopie aus `hel._store_uploaded_bytes` (Helper soll auch ohne FastAPI-Stack laufen können). Konstanten `PROJECT_BIBLE_FILENAME_TEMPLATE`, `README_FILENAME` exportiert.
- **Verdrahtung:** [`hel.py::create_project_endpoint`](zerberus/app/routers/hel.py) ruft NACH `projects_repo.create_project()` `materialize_template(project, _projects_storage_base())` auf; Response-Feld `template_files` listet die neu angelegten Einträge.
- **Feature-Flag:** `ProjectsConfig.auto_template: bool = True` (Default `True`, Default in `config.py` weil `config.yaml` gitignored). Kann für Migrations-Tests/Bulk-Imports abgeschaltet werden.
- **Logging:** Neuer `[TEMPLATE-198]`-INFO-Log mit `slug`/`path`/`size`/`sha[:8]` pro neu angelegter Datei + `skip slug=... path=... (already exists)` bei Idempotenz-Skip. Bei Crash WARNING via `logger.exception` mit Slug.
- **Tests:** 23 neue Tests in [`test_projects_template.py`](zerberus/tests/test_projects_template.py) — 6 Pure-Function-Cases (Slug-Uppercase, Datum, Sektionen, Description-Block, Empty-Description-Placeholder, Missing-Keys-Defaults), 6 Materialize-Cases (Two-Files, SHA-Storage-Pfad, Idempotenz, User-Content-Schutz, Dry-Run-no-side-effect, Content-Render), 3 End-to-End (Flag-on, Flag-off, Crash-Resilienz), 3 Source-Audit (Imports, Flag-Honor, Konstanten). `disable_auto_template`-Autouse-Fixture in `test_projects_endpoints.py` + `test_projects_files_upload.py` hält deren File-Counts stabil.
- **Teststand:** 1464 → **1487 passed** (+23), 4 xfailed (pre-existing), 2 pre-existing Failures (SentenceTransformer-Mock + edge-tts-Install — bekannte Schulden).
- **Phase 5a Ziel #2:** ✅ Projekte haben Struktur. Damit sind die Ziele #1 (Backend+UI) + #2 (Struktur) abgeschlossen — als nächstes #3 (Projekt-RAG-Index, P199) oder #5 (Code-Execution, P200).

**Patch 197** — Phase 5a Decision 3: Persona-Merge-Layer aktiviert (2026-05-02)

Schließt die Lücke zwischen P194 (Persona-Overlay-JSON in der DB) / P195 (Hel-UI-Editor dafür) und der eigentlichen Wirkung im LLM-Call. Bisher konnte Chris in der Hel-UI ein `system_addendum` und `tone_hints` pro Projekt pflegen — aber das landete nirgends im System-Prompt. P197 verdrahtet das.

Aktivierungs-Mechanismus: Header `X-Active-Project-Id: <int>` am `POST /v1/chat/completions`-Request. Header-basierte Auswahl (statt persistenter Spalte) gewinnt im ersten Schritt — keine Schema-Änderung, kein Migration-Risiko, der Frontend-Caller (Nala-Tab "Projekte" sobald gebaut, oder externe Clients) entscheidet pro Request. Persistente Auswahl (Spalte `active_project_id` an `chat_sessions`) ist später trivial nachrüstbar — der Reader `read_active_project_id` ist die einzige Stelle, die geändert werden muss.

Merge-Reihenfolge laut Decision 3 (2026-05-01): System-Default → User-Persona ("Mein Ton") → Projekt-Overlay. Die ersten beiden stecken bereits zusammen in `system_prompt_<profile>.json` (eine Datei pro Profil); P197 hängt nur den Projekt-Layer als markierten Block hinten dran. Der Block beginnt mit `[PROJEKT-KONTEXT — verbindlich für diese Session]` als eindeutigem Marker (Substring-Check für Tests/Logs + Schutz gegen Doppel-Injection in derselben Pipeline). Optional ist eine `Projekt: <slug>`-Zeile drin, damit das LLM beim Self-Talk korrekt referenziert.

Position der Verdrahtung: VOR `_wrap_persona` (P184), damit der `# AKTIVE PERSONA — VERBINDLICH`-Marker auch das Projekt-Overlay umschließt — sonst stünde der Projekt-Block AUSSERHALB der "verbindlichen Persona" und das LLM könnte ihn entwerten. Die Reihenfolge ist explizit per Source-Audit-Test verifiziert (`test_persona_merge.TestSourceAudit.test_merge_runs_before_wrap`).

Defensive Behaviors: archivierte Projekte → kein Overlay (Slug wird trotzdem geloggt, damit Bug-Reports diagnostizierbar bleiben); unbekannte ID → kein Crash, einfach kein Overlay; kaputter Header (Buchstaben, negative Zahl) → ignoriert; leerer Overlay (kein `system_addendum`, leere `tone_hints`) → kein Block; `tone_hints` mit Duplikaten/Leer-Strings → bereinigt (case-insensitive Dedupe, erstes Vorkommen gewinnt). Die DB-Auflösung `resolve_project_overlay` ist von der Pure-Function `merge_persona` getrennt — der Helper bleibt I/O-frei und damit synchron testbar.

Telegram bewusst aus P197 ausgeklammert: Huginn hat eine eigene Persona-Welt (zynischer Rabe) ohne User-Profile und ohne Verbindung zu Nala-Projekten. Project-Awareness in Telegram bräuchte eigene UX (`/project <slug>`-Befehl oder persistente Bind-Tabelle) — eigener Patch wenn der Bedarf entsteht.

- **Neuer Helper:** [`zerberus/core/persona_merge.py`](zerberus/core/persona_merge.py) — `merge_persona(base_prompt, overlay, project_slug=None)` als Pure-Function, `read_active_project_id(headers)` mit Lowercase-Fallback (FastAPI-`Headers` ist case-insensitive, ein Test-`dict` nicht), `resolve_project_overlay(project_id, *, skip_archived=True)` als async DB-Schnittstelle (lazy-Import von `projects_repo` gegen Zirkular-Importe). Konstanten `ACTIVE_PROJECT_HEADER` und `PROJECT_BLOCK_MARKER` exportiert.
- **Verdrahtung:** [`legacy.py::chat_completions`](zerberus/app/routers/legacy.py) — Reihenfolge jetzt `load_system_prompt` → **`merge_persona` (P197, NEU)** → `_wrap_persona` (P184) → `append_runtime_info` (P185) → `append_decision_box_hint` (P118a) → Prosodie-Inject (P190) → `messages.insert(0, system)`.
- **Logging:** Neuer `[PERSONA-197]`-INFO-Log mit `project_id`, `slug`, `base_len`, `project_block_len`. Bei Lookup-Fehlern WARNING mit Exception-Text. Bei archiviertem Projekt INFO mit Slug.
- **Tests:** 33 neue Tests in [`test_persona_merge.py`](zerberus/tests/test_persona_merge.py) — 12 Edge-Cases für `merge_persona` (kein Overlay/leeres Overlay/nur Addendum/nur Hints/beide/Dedupe case-insensitive/leere Strings strippen/Doppel-Injection-Schutz/leerer Base-Prompt/Slug-Anzeige/unerwartete Typen/Separator-Format), 7 Header-Reader-Cases (fehlt/None/leer/valid/Lowercase-Fallback/Non-Numeric/negativ), 5 async DB-Cases via `tmp_db`-Fixture (None-ID/Unknown-ID/Existing/Archived/Archived-with-Skip-False/ohne Overlay), 4 End-to-End über `chat_completions` mit Mock-LLM (Overlay erscheint im messages[0], kein Header → kein Overlay, unbekannte ID → kein Crash, archiviert → übersprungen), 5 Source-Audit-Cases (Log-Marker, Imports, Reihenfolge merge-vor-wrap).
- **Teststand:** 1431 → **1464 passed** (+33), 4 xfailed (pre-existing), 2 pre-existing Failures (SentenceTransformer-Mock + edge-tts-Install — bekannte Schulden).
- **Phase 5a Decision 3:** ✅ Persona-Merge-Layer aktiv. Die in P194/P195 vorbereitete Persona-Overlay-Pflege wirkt jetzt im LLM-Call.

**Patch 196** — Phase 5a #4: Datei-Upload-Endpoint + UI (2026-05-02)

Erster Schritt für Phase-5a-Ziel #4 ("Dateien kommen ins Projekt"). `POST /hel/admin/projects/{id}/files` (multipart) und `DELETE /hel/admin/projects/{id}/files/{file_id}`. Bytes liegen weiter unter `data/projects/<slug>/<sha[:2]>/<sha>` (P194-Konvention) — kein Doppel-Schreiben, wenn der SHA im Projekt-Slug-Pfad schon existiert. Validierung: Filename-Sanitize (`..`/Backslashes/leere Segmente raus), Extension-Blacklist (`.exe`, `.bat`, `.sh`, ...), 50 MB Default-Limit. Delete-Logik: wenn der `sha256` woanders noch referenziert wird, bleibt die Storage-Datei liegen (Schutz vor versehentlichem Cross-Project-Delete); sonst wird sie atomar entfernt + leere Parent-Ordner aufgeräumt. Atomic Write per `tempfile.mkstemp` + `os.replace` verhindert halbe Dateien nach Server-Kill.

UI-seitig ersetzt eine Drop-Zone in der Detail-Card den P195-Platzhalter ("Upload kommt in P196"). Drag-and-drop UND klickbarer File-Picker (multiple), pro Datei eigene Progress-Zeile per `XMLHttpRequest.upload.progress`-Event (sequenzielle Uploads, damit der Server bei Massen-Drops nicht mit parallelen Streams überrannt wird). Datei-Liste bekommt einen Lösch-Button mit Confirm-Dialog; Hinweistext klärt, dass die Bytes nur entfernt werden, wenn sie nirgends sonst referenziert sind.

- **Schema:** Neue `ProjectsConfig` in `core/config.py` (`data_dir`, `max_upload_bytes`, `blocked_extensions`) — Defaults im Pydantic-Modell, weil `config.yaml` gitignored ist (sonst fehlt der Schutz nach `git clone`). Neue Repo-Helper `count_sha_references()`, `is_extension_blocked()`, `sanitize_relative_path()` in `core/projects_repo.py`.
- **Endpoints:** `_projects_storage_base()` als Indirektion in `hel.py` — Tests können den Storage-Pfad per Monkeypatch umbiegen, ohne die globalen Settings anzufassen. `_store_uploaded_bytes()` schreibt atomar; `_cleanup_storage_path()` räumt Datei + leere Parent-Ordner bis zum `data_dir`-Anker auf (best-effort).
- **Tests:** 49 neue Tests — 17 Upload-Endpoint (`test_projects_files_upload.py`: Happy-Path, Subdir, Dedup, Extension-Block, Path-Traversal, Empty-Filename, Empty-Data, Too-Large, 409-Dup, Delete-Unique, Delete-Shared, Cross-Project-404, Storage-Cleanup), 21 Repo-Helper (`test_projects_repo.py`: Sanitize, Extension-Block, count-sha-references), 11 UI-Source-Inspection (`test_projects_ui.py`: Drop-Zone, Progress, Delete-Button, Drag-and-Drop-Events).
- **Teststand:** 1382 → **1431 passed** (+49). 0 neue Failures, 4 xfailed (pre-existing), 2 pre-existing Failures (SentenceTransformer-Mock + edge-tts — bekannte Schulden).
- **Phase 5a Ziel #4:** ✅ Datei-Upload geöffnet. Indexierung in projekt-spezifischen RAG (Ziel #3) folgt mit P199.

**Patch 195** — Phase 5a #1: Hel-UI-Tab "Projekte" — schließt Ziel #1 ab (2026-05-02)

UI-Hülle über das P194-Backend. Neuer Tab `📁 Projekte` im Hel-Dashboard zwischen Huginn und Links. Liste, Anlegen-Modal (Form-Overlay statt extra CSS-Lib), Edit/Archive/Unarchive/Delete inline, Datei-Liste read-only (Upload kommt P196). Persona-Overlay-Editor: `system_addendum` (Textarea) + `tone_hints` (Komma-Liste). Mobile-first: 44px Touch-Targets durchgehend, scrollbare Tab-Nav, Form-Overlay mit `flex-start`-Top für kleine Screens. Slug-Override nur beim Anlegen editierbar (Slug ist immutable per Repo-Vertrag P194). Lazy-Load via `activateTab('projects')`.

- **Tests:** 20 Source-Inspection-Tests in [`test_projects_ui.py`](zerberus/tests/test_projects_ui.py) (Pattern wie `test_patch170_hel_kosmetik.py`). Decken Tab-Reihenfolge, Section-Markup, JS-Funktionen (`loadProjects`, `saveProjectForm`, `archive/unarchive/delete`, `loadProjectFiles`), 44px-Touch-Targets, Lazy-Load-Verdrahtung.
- **Teststand:** 1365 → **1382 passed** (+17, da `test_projects_endpoints.py` schon mitgezählt war), 0 neue Failures, 4 xfailed (pre-existing), 2 pre-existing Failures (SentenceTransformer-Mock + edge-tts-Install — nicht blockierend).
- **Phase 5a Ziel #1:** ✅ vollständig (Backend P194 + UI P195). Nächste Patches greifen sukzessive Ziele #2 (Templates), #3 (RAG-Index pro Projekt), #4 (Datei-Upload-Endpoint).

**Patch 194** — Phase 5a #1: Projekte als Entität, Backend-Layer (2026-05-02)

Erster Patch der Phase 5a. Tabellen `projects` + `project_files` in `bunker_memory.db` (Decision 1, 2026-05-01), Repo + Hel-CRUD-Endpoints + 46 neue Tests. UI-Tab folgt in P195. Teststand 1316 → **1365 passed** (+49: 28 Repo + 18 Endpoints + 3 weitere), 0 neue Failures, 4 xfailed (pre-existing).

- **Schema:** `projects(id, slug UNIQUE, name, description, persona_overlay JSON, is_archived, created_at, updated_at)` + `project_files(id, project_id, relative_path, sha256, size_bytes, mime_type, storage_path, uploaded_at, UNIQUE(project_id, relative_path))`. Soft-Delete via `is_archived`. Cascade per Repo (`delete_project` ⇒ `DELETE FROM project_files WHERE project_id = ?`), nicht per FK — Models bleiben dependency-frei (keine ORM-Relations). Persona-Overlay als JSON-TEXT für Decision 3 (Merge-Layer System → User → Projekt).
- **Storage-Konvention:** `data/projects/<slug>/<sha[:2]>/<sha>` — Sha-Prefix als Sub-Verzeichnis, damit kein Hotspot-Ordner entsteht. Bytes liegen NICHT in der DB. Helper `storage_path_for()` + `compute_sha256()` in `projects_repo.py`.
- **Endpoints:** `/hel/admin/projects` (list, create) + `/hel/admin/projects/{id}` (get, patch, delete) + `/admin/projects/{id}/archive` + `/unarchive` + `/files`. Admin-only via Hel `verify_admin` (Basic Auth). Bewusst nicht unter `/v1/` — `/v1/` ist exklusiv Dictate-App-Lane (Hotfix 103a, lessons.md Regel 8).
- **Slug-Generator:** Lower-case, special-chars → `-`, max 64 Zeichen, Kollisions-Suffix `-2`/`-3`/.... `slugify("AI Research v2")` → `"ai-research-v2"`.
- **Migration:** Alembic-Revision `b03fbb0bd5e3` (down_revision `7feab49e6afe`), idempotent via `_has_table()`-Guard. Indexe: `uq_projects_slug`, `idx_projects_is_archived`, `idx_project_files_project`, `idx_project_files_sha`. UNIQUE-Constraint `(project_id, relative_path)` direkt in `op.create_table()` über `sa.UniqueConstraint`.
- **Lesson dokumentiert:** Composite-UNIQUE-Constraints MÜSSEN im Model (`__table_args__`) deklariert werden, nicht nur als raw `CREATE UNIQUE INDEX` in `init_db` — sonst greift der Constraint nicht in Test-Fixtures, die nur `Base.metadata.create_all` aufrufen.

- **P192 — Sentiment-Triptychon UI:** Drei Chips (BERT 📝 + Prosodie 🎙️ + Konsens 🎯) an jeder Chat-Bubble, Sichtbarkeit per Hover/`:active` analog Toolbar-Pattern (P139). Neue Utility [`zerberus/utils/sentiment_display.py`](zerberus/utils/sentiment_display.py) mit `bert_emoji()`, `prosody_emoji()`, `consensus_emoji()`, `compute_consensus()`, `build_sentiment_payload()`. Backend liefert `sentiment: {user, bot}` ADDITIV in `/v1/chat/completions`-Response — OpenAI-Schema bleibt formal kompatibel. Mehrabian-Regel: bei `confidence > 0.5` dominiert die Prosodie, sonst Fallback auf BERT. Inkongruenz 🤔 wenn BERT positiv und Prosodie-Valenz < -0.2. 22 Tests in [`test_sentiment_triptych.py`](zerberus/tests/test_sentiment_triptych.py).
- **P193 — Whisper-Endpoint Prosodie/Sentiment-Enrichment:** `/v1/audio/transcriptions` Response erweitert: `text` bleibt IMMER (Backward-Compat für Dictate / SillyTavern / Generic-Clients), zusätzlich optional `prosody` (P190) + neu `sentiment.bert` + `sentiment.consensus`. `/nala/voice` identisch erweitert + zusätzlich named SSE-Events `event: prosody` und `event: sentiment` über `/nala/events` — Triptychon-Frontend kann sync (JSON) oder async (SSE) konsumieren. Fail-open: BERT-Fehler erzeugt nur Logger-Warnung, Endpoint läuft sauber durch. 16 Tests in [`test_whisper_enrichment.py`](zerberus/tests/test_whisper_enrichment.py). Logging-Tag `[ENRICHMENT-193]`.

### Doku-Konsolidierung (Phase-4-Abschluss)

- `lessons.md` aktualisiert: neue Sektionen Prosodie/Audio (P188-191), RAG/FAISS (P187+), Frontend (P186+P192), Whisper-Enrichment (P193). Veraltete Hinweise auf MiniLM als „aktiver Embedder" durch DualEmbedder-Beschreibung ersetzt (in CLAUDE_ZERBERUS.md).
- `CLAUDE_ZERBERUS.md` aktualisiert: neue Top-Sektionen Sentiment-Triptychon (P192) + Whisper-Endpoint Enrichment (P193).
- `SUPERVISOR_ZERBERUS.md` (diese Datei) auf < 400 Zeilen gestrafft. Patch-Details vor P177 leben jetzt in [`docs/PROJEKTDOKUMENTATION.md`](docs/PROJEKTDOKUMENTATION.md).
- Neues Übergabedokument [`docs/HANDOVER_PHASE_5.md`](docs/HANDOVER_PHASE_5.md) für die nächste Supervisor-Session (Phase 5 / Nala-Projekte).

---

## Phase 4 — ABGESCHLOSSEN ✅ (P119–P193)

Vollständige Patch-Historie in [`docs/PROJEKTDOKUMENTATION.md`](docs/PROJEKTDOKUMENTATION.md). Hier nur die letzten Meilensteine ab P186:

| Patch | Datum | Zusammenfassung |
|-------|-------|-----------------|
| **P203d-3** | 2026-05-03 | Phase 5a Ziel #5 ABGESCHLOSSEN: UI-Render im Nala-Frontend — `renderCodeExecution(wrapperEl, codeExec)` baut Code-Card (Sprach-Tag + exit-Badge + escapter Code) plus collapsible Output-Card (44×44px Toggle, stdout/stderr, Truncated-Marker), `addMessage` retourniert Wrapper-Element, eigener `escapeHtml`-Helper, XSS-Min-Count, `node --check` ueber NALA_HTML-Scripts + 30 Tests |
| **P203d-2** | 2026-05-03 | Phase 5a #5 Output-Synthese: zweiter LLM-Call (Prompt+Code+stdout/stderr → menschenlesbare Antwort), Pure-Function-Schicht in `modules/sandbox/synthesis.py`, `should_synthesize`-Trigger, Bytes-genau Truncate (5KB), `[SYNTH-203d-2]`-Logging, store_interaction-Reorder (assistant-Insert nach Synthese), fail-open + 47 Tests |
| **P203d-1** | 2026-05-02 | Phase 5a #5 Backend-Pfad: Code-Detection + Sandbox-Roundtrip im `/v1/chat/completions` — `first_executable_block` + `execute_in_workspace(writable=False)` + additives `code_execution`-JSON-Field, Sechs-Stufen-Gate (Header + Slug + Overlay-not-None + Sandbox-enabled + Fenced-Block + Result), fail-open + 19 Tests |
| **P204** | 2026-05-02 | Phase 5a #17 ABGESCHLOSSEN: Prosodie-Kontext im LLM — `[PROSODIE]`-Block, BERT+Gemma-Konsens via Mehrabian, Worker-Protection (keine Zahlen) + 33 Tests |
| **P203c** | 2026-05-02 | Phase 5a #5 Zwischenschritt: Sandbox-Workspace-Mount + `execute_in_workspace` — RO-Default, Mount-Validation, Defense-in-Depth + 17 Tests |
| **P203b** | 2026-05-02 | Hel-UI-Hotfix: kaputtes Quote-Escaping → Event-Delegation via `data-*`-Attribute, JS-Integrity-Test + 10 Tests |
| **P203a** | 2026-05-02 | Phase 5a #5 Vorbereitung: Project-Workspace-Layout — Hardlink+Copy-Fallback, atomic write, sync_workspace + 36 Tests |
| **P202** | 2026-05-02 | PWA-Auth-Hotfix: SW skipt Top-Level-Navigation, Cache v2-Bump + 5 Tests |
| **P201** | 2026-05-02 | Phase 5a #4: Nala-Tab "Projekte" + zentraler Header-Setter `X-Active-Project-Id` in `profileHeaders()` + 21 Tests |
| **P200** | 2026-05-02 | Phase 5a #16: PWA-Verdrahtung Nala + Hel — Manifest + SW pro App, Kintsugi-Icons + 39 Tests |
| **P199** | 2026-05-02 | Phase 5a #3: Projekt-RAG-Index — Pure-Numpy-Linearscan, MiniLM-L6-v2 + 46 Tests |
| **P198** | 2026-05-02 | Phase 5a #2: Template-Generierung beim Anlegen (Bibel + README) + 23 Tests |
| **P197** | 2026-05-02 | Phase 5a Decision 3: Persona-Merge-Layer aktiviert — Header `X-Active-Project-Id` + `merge_persona`-Helper + 33 Tests |
| **P196** | 2026-05-02 | Phase 5a #4: Datei-Upload-Endpoint + Drop-Zone-UI (öffnet Ziel #4) — SHA-Dedup-Delete, Extension-Blacklist, atomic write + 49 Tests |
| **P195** | 2026-05-02 | Phase 5a #1: Hel-UI-Tab "Projekte" (schließt Ziel #1 ab) — Liste/Form/Persona-Overlay + 20 Tests |
| **P194** | 2026-05-02 | Phase 5a #1: Projekte als Entität (Backend) — Schema + Repo + Hel-CRUD + 46 Tests |
| P192–P193 | 2026-05-01 | Sentiment-Triptychon + Whisper-Enrichment + Phase-4-Abschluss + Doku-Konsolidierung |
| P189–P191 | 2026-05-01 | Prosodie-Pipeline komplett: Gemma-Client + Pipeline + Consent-UI + Worker-Protection |
| P186–P188 | 2026-05-01 | Auto-TTS + FAISS-Migration (DualEmbedder DE/EN) + Prosodie-Foundation |
| P183–P185 | 2026-05-01 | Black-Bug VIERTER (endgültig) + Persona-Wrap + Runtime-Info-Block |
| P180–P182 | 2026-04-30 | Live-Findings L-178: Guard+RAG-Kontext, Telegram-Allowlist, ADMIN-Plausi, Unsupported-Media |
| P178–P179 | 2026-04-29 | Huginn-RAG-Selbstwissen (system-Kategorie, Default-Whitelist, Backlog-Konsolidierung) |
| P177 | 2026-04-29 | Pipeline-Cutover-Feature-Flag (`use_message_bus`, default false, live-switch) |
| P173–P176 | 2026-04-28 | Phase-E-Skelett (Message-Bus, Adapter, Pipeline, Sandbox-Live, Coda-Autonomie) |
| P119–P172 | 2026-04-09…04-28 | Aufbau Phase 4: RAG-Pipeline, Guard, Sentiment, Memory-Extraction, Telegram-Bot, HitL, Sandbox |

### Phase-4-Bilanz

| Bereich | Highlights |
|---------|-----------|
| **Guard** | Mistral Small 3 (P120/P180), `caller_context` + `rag_context` ohne Halluzinations-Risiko, fail-open |
| **Huginn (Telegram)** | Long-Polling, Allowlist, Intent-Router via JSON-Header, HitL persistent (SQLite), RAG via system-Kategorie, Sandbox-Hook |
| **Nala UI** | Mobile-first Bubbles, HSL-Slider, TTS, Auto-TTS, Katzenpfoten, Feuerwerk, Triptychon |
| **RAG** | Code-Chunker, DualEmbedder (DE GPU + EN CPU), FAISS-Migration, Soft-Delete, Category-Boost |
| **Prosodie** | Gemma 4 E2B (lokal, Q4_K_M), Whisper+Gemma parallel via `asyncio.gather`, Consent-UI, Triptychon |
| **Pipeline** | Message-Bus + Telegram/Nala/Rosa-Adapter, DI-only Pipeline, Feature-Flag-Cutover |
| **Security** | Input-Sanitizer (NFKC + Patterns), Callback-Spoofing-Schutz, Allowlist, Worker-Protection (Audio nicht in DB) |
| **Infrastruktur** | Docker-Sandbox (`--network none --read-only`), Pacemaker, pytest-Marker (e2e/guard_live/docker), Coda-Autonomie |
| **Bugs** | 🪦 Black Bug (4 Anläufe — P183 hat ihn endgültig getötet), HSL-Parse, Config-Save, Terminal-Hygiene |

---

## Phase 5 — Nala-Projekte (Roadmap)

**Referenz-Dokumente:**
- [`nala-projekte-features.md`](nala-projekte-features.md) — 100+ Features, konsolidiert
- [`NALA_PROJEKTE_PRIORISIERUNG.md`](NALA_PROJEKTE_PRIORISIERUNG.md) — Priorisierung für Zerberus-Kontext
- [`docs/HANDOVER_PHASE_5.md`](docs/HANDOVER_PHASE_5.md) — Übergabedokument mit Tech-Stack, VRAM, offenen Items

### Phase 5a — Grundgerüst (erste 10 Patches)

| # | Feature | Beschreibung |
|---|---------|-------------|
| P194 | Projekt-DB (Backend) | SQLite-Schema, Repo, Hel-CRUD-Endpoints ✅ |
| P195 | Hel-UI-Tab Projekte | Liste + Anlegen/Edit/Archive/Delete + Persona-Overlay-Editor ✅ |
| P196 | Datei-Upload-Endpoint + UI | `POST /hel/admin/projects/{id}/files` + Drop-Zone + SHA-Dedup-Delete ✅ |
| P197 | Persona-Merge-Layer | System → User → Projekt-Overlay im LLM-Prompt aktivieren |
| P198 | Template-Generierung | `ZERBERUS_X.md`, Ordnerstruktur, Git-Init |
| P199 | Projekt-RAG-Index | Isolierter FAISS pro Projekt |
| P200 | Code-Execution-Pipeline | Intent `PROJECT_CODE` → LLM → Sandbox |
| P201 | HitL-Gate für Code | Sicherheit vor Ausführung (Default-ON) |
| P202 | Snapshot/Backup-System | `.bak` + Rollback vor jeder Dateiänderung |
| P203 | Diff-View | User sieht jede Änderung vor Bestätigung |

### Phase 5b — Power-Features (danach)

- Multi-LLM Evaluation (Bench-Mode pro Projekt-Aufgabe)
- Bugfix-Workflow + Test-Agenten (Loki/Fenrir/Vidar Per-Projekt)
- Multi-Agent Orchestrierung, Debugging-Konsole
- Reasoning-Modi, LLM-Wahl, Cost-Transparency
- Agent-Observability (Chain-of-Thought Inspector)

### Abhängigkeiten / Was steht schon

- Docker-Sandbox (P171) existiert + Images gezogen (P176)
- HitL-Mechanismus (P167) existiert, SQLite-persistent
- Pipeline-Cutover (P177) existiert, Feature-Flag bereit
- Guard (P120/P180) funktioniert mit RAG- und Persona-Kontext
- Prosodie (P189–P191) funktioniert — kann in Projekt-Kontext integriert werden (z. B. Stimmung als Code-Quality-Indikator)

---

## Architektur-Referenz

### Aktiver Tech-Stack

| Komponente | Modell / Technologie |
|------------|---------------------|
| **Cloud-LLM** | DeepSeek V3.2 (OpenRouter) |
| **Guard** | Mistral Small 3 (OpenRouter, `mistralai/mistral-small-24b-instruct-2501`) |
| **Prosodie** | Gemma 4 E2B (lokal, `llama-mtmd-cli`, Q4_K_M, ~3.4 GB) |
| **ASR / Whisper** | faster-whisper large-v3 (FP16, Docker Port 8002) |
| **Sentiment (Text)** | `oliverguhr/german-sentiment-bert` (lokal) |
| **Embeddings DE** | `T-Systems-onsite/cross-en-de-roberta-sentence-transformer` (GPU) |
| **Embeddings EN** | `intfloat/multilingual-e5-large` (CPU, optional Index) |
| **Reranker** | `BAAI/bge-reranker-v2-m3` |
| **DB** | SQLite (`bunker_memory.db`, WAL-Modus, Alembic-Migrations seit P92) |
| **Frontend** | Nala (Mobile-first), Hel (Admin-Dashboard) |
| **Bot** | Huginn (Telegram, Long-Polling, Tailscale-intern) |

### VRAM-Belegung (Modus „Nala aktiv" mit Prosodie)

```
Whisper 4.5 + BERT 0.5 + Gemma E2B 3.0 + DualEmbedder 0.5 + Reranker 1.0 + Windows 0.8
= ~10.3 GB / 12 GB (RTX 3060)
```

### Repos (alle 3 müssen synchron sein)

- **Zerberus** (Code, lokal): `C:\Users\chris\Python\Rosa\Nala_Rosa\Zerberus`
- **Ratatoskr** (Doku-Sync, GitHub): `C:\Users\chris\Python\Rosa\Nala_Rosa\Ratatoskr`
- **Claude** (universelle Lessons, GitHub): `C:\Users\chris\Python\Claude`

---

## Offene Items / Bekannte Schulden

→ Konsolidiert in [`BACKLOG_ZERBERUS.md`](BACKLOG_ZERBERUS.md) (seit Patch 179). Hier nur strukturelle Schulden:

- **Persona-Hierarchie** (Hel vs. Nala „Mein Ton") — löst sich mit SillyTavern/ChatML-Wrapper (B-071)
- **`interactions`-Tabelle ohne User-Spalte** — Per-User-Metriken erst nach Alembic-Schema-Fix vertrauenswürdig
- ~~**`scripts/verify_sync.ps1`** existiert nicht — `sync_repos.ps1` Output muss manuell geprüft werden~~ **(seit längerem GELÖST: `scripts/verify_sync.ps1` existiert und meldet 0/0/0 unpushed Commits in allen drei Repos.)**
- ~~**`system_prompt_chris.json` Trailing-Newline-Diff** — `git checkout` zum Bereinigen, kein echter Bug~~ **(GELÖST 2026-05-07: Datei aus dem Repo entfernt, chris-Profil fällt auf Default `system_prompt.json` zurück.)**
- ~~**`sync_repos.ps1` Quote-Escape-Bug** — Anführungszeichen in Commit-Messages brechen `git commit -m "...$var..."` über das PowerShell-5.1-Native-Argument-Quoting~~ **(GELÖST 2026-05-07 mit P217-pre: Helper `Invoke-GitCommitFromString` schreibt Message via Temp-Datei und ruft `git commit -F`. Smoke-Test `scripts/test_sync_repos_quote_escape.ps1` 7/7 grün.)**
- **Voice-Messages in Telegram-DM** funktionieren nicht (P182 Unsupported-Media-Handler antwortet höflich) — B-072 für echte Whisper-Pipeline

## Architektur-Warnungen

- **Rosa Security Layer:** NICHT implementiert — Dateien im Projektordner sind nur Vorbereitung
- **JWT** blockiert externe Clients komplett — `static_api_key` ist der einzige Workaround (Dictate, SillyTavern)
- **/v1/-Endpoints** MÜSSEN auth-frei bleiben (Dictate-Tastatur kann keine Custom-Headers) — Bypass via `_JWT_EXCLUDED_PREFIXES`
- **Chart.js / zoom-plugin / hammer.js** via CDN — bei Air-Gap ist das Metriken-Dashboard tot
- **Prosodie-Audio-Bytes** dürfen NICHT in `interactions`-Tabelle landen (Worker-Protection P191)

---

## Sync-Pflicht (Patch 164+)

`sync_repos.ps1` muss nach jedem `git push` ausgeführt werden — der Patch gilt erst als abgeschlossen, wenn Zerberus, Ratatoskr und Claude-Repo synchron sind. Coda pusht zuverlässig nach Zerberus, vergisst aber den Sync regelmäßig, sodass Ratatoskr und Claude-Repo driften.

Falls Claude Code den Sync nicht selbst ausführen kann (z. B. PowerShell nicht verfügbar oder Skript wirft Fehler), MUSS er das explizit melden — etwa mit „⚠️ sync_repos.ps1 nicht ausgeführt — bitte manuell nachholen". Stillschweigendes Überspringen ist nicht zulässig. Die durable Regel steht in `CLAUDE_ZERBERUS.md` unter „Repo-Sync-Pflicht".

## Langfrist-Vision

- **Phase 5 — Nala-Projekte:** Zerberus wird zur persönlichen Code-Werkstatt, Nala vermittelt zwischen Chris und LLMs/Sandboxes
- **Metric Engine** = kognitives Tagebuch + Frühwarnsystem für Denkmuster-Drift
- **Rosa Corporate Security Layer** = letzter Baustein vor kommerziellem Einsatz
- **Telegram-Bot** als Zero-Friction-Frontend für Dritte (keine Tailscale-Installation nötig)

## Supervisor-Verhalten — Bug-Sammelstelle (P219-pre)

**Bug-Sammelstelle (still sammeln, nicht aktionistisch reagieren).** Wenn Chris im Supervisor-Fenster Bugs oder Issues diktiert, ohne explizit "fixen", "angehen", "los" oder "pack zusammen" zu sagen, sammelt der Supervisor diese still in einer internen Liste. Kein sofortiger Report, kein Aktionismus, kein Patch-Prompt. Erst wenn Chris explizit den Auftrag erteilt ("los", "pack zusammen", "mach daraus einen Patch"), wird ein Coda-Auftrag aus der Sammlung gebaut. Bis dahin bleibt die Liste passiv. Hintergrund: Chris denkt laut beim Sammeln; jeder Bug, der sofort in einen Patch umgesetzt würde, wäre verfrüht und würde die Sammelphase unterbrechen.

**Bug-Sammelstand-Anzeige am Ende jeder Antwort.** Der Supervisor zeigt am Ende jeder Antwort den aktuellen Stand offener/gesammelter Bugs als nummerierte Liste an. Format-Vorschlag:

```
---
**Aktuelle Bug-Sammelstelle:**
1. [Bug-Titel] — [Kurzbeschreibung]
2. [Bug-Titel] — [Kurzbeschreibung]
```

Wenn die Liste leer ist, wird nichts angezeigt — keine "Liste leer"-Zeile, kein Header, einfach weglassen. Funktion der Anzeige: visueller Druck. Wenn die Liste lang wird, sieht Chris das automatisch und gibt von sich aus den Auftrag, sie abzuarbeiten. Zusätzlich: alle paar Prompts oder vor erkennbarem Kontextfenster-Ende erinnert der Supervisor Chris explizit an die gesammelten Bugs ("Sammelstelle ist auf 6 Bugs gewachsen — soll ich einen Sammel-Patch bauen?"). Keine Erinnerung wenn die Liste bei ≤2 Bugs steht.

**Wann die Sammelstelle in einen Coda-Auftrag mündet.** Trigger sind explizite Chris-Phrasen: "los", "pack zusammen", "mach einen Patch draus", "Coda kann jetzt", "Sammel-Patch". Nicht-Trigger: implizite Hinweise wie "das wär gut zu fixen", "irgendwann müssen wir das angehen". Im Zweifel still sammeln und beim nächsten Reminder fragen.

## Don'ts für Supervisor

- **PROJEKTDOKUMENTATION.md NICHT vollständig laden** (5000+ Zeilen = Kontextverschwendung) — nur gezielt nach Patch-Nummern grep'en
- **Memory-Edits max 500 Zeichen** pro Eintrag
- **Session-ID ≠ User-Trennung** — Metriken pro User erst nach DB-Architektur-Fix vertrauenswürdig
- **Patch-Prompts IMMER als `.md`-Datei** — NIE inline im Chat. Claude Code erhält den Inhalt per Copy-Paste aus der Datei (Patch 101)
- **Dateinamen `CLAUDE_ZERBERUS.md` und `SUPERVISOR_ZERBERUS.md` sind FINAL** — in Patch-Prompts nie mit alten Namen (`CLAUDE.md`, `HYPERVISOR.md`) referenzieren (Patch 100/101)
- **Lokale Pfade:** Ratatoskr liegt unter `C:\Users\chris\Python\Rosa\Nala_Rosa\Ratatoskr\` (nicht `Rosa\Ratatoskr\`), Bmad82/Claude unter `C:\Users\chris\Python\Claude\` (nicht `Rosa\Claude\`). Patch-Prompts mit falschen Pfaden → immer erst verifizieren, nicht raten
- **Bug-Sammelstelle NICHT als Erstreaktion in einen Patch verwandeln (P219-pre)** — siehe Sektion "Supervisor-Verhalten — Bug-Sammelstelle" oben. Erst sammeln, erst auf Trigger-Phrase warten, dann bauen.

---

## Für die nächste Supervisor-Session (Phase 5 Start)

1. `SUPERVISOR_ZERBERUS.md` von GitHub fetchen (frisch konsolidiert in P192–P193)
2. [`docs/HANDOVER_PHASE_5.md`](docs/HANDOVER_PHASE_5.md) lesen — Tech-Stack, VRAM, offene Items, Phase-5-Roadmap stehen dort vollständig
3. Phase-5a mit P194 (Projekt-DB + Workspace) starten — Doppel- oder Dreiergruppen-Rhythmus beibehalten
4. Prosodie-Live-Test steht noch aus — `llama-mtmd-cli` muss im PATH sein, sonst läuft Pfad A nicht
5. Pipeline-Feature-Flag (`use_message_bus`) ist bereit für Live-Switch-Tests, aber Default-OFF bis Chris explizit umstellt
