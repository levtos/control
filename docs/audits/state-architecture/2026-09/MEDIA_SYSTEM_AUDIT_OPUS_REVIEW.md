# MEDIA_SYSTEM_AUDIT_OPUS_REVIEW.md — Unabhängige Verifikation des Sonnet-IST-Audits

**Reviewer:** Claude Opus 5 (Red-Team-Rolle) · **Stand:** 2026-09-16
**Geprüftes Objekt:** `MEDIA_SYSTEM_AUDIT.md` + 11 CSVs (Sonnet, 2026-09-15)
**Primärquellen:** Code und Tests auf `origin/main` (exportiert per `git archive`, keine Checkout-Änderung), Runtime-Konfiguration `einhornzentrale@origin/main`, dokumentierte Repros/Handoffs, GitHub-Issues (read-only via `gh`), Live-Zustand der Einhornzentrale (ausschließlich lesend über den eigenen HA-Zugang, markiert als `LIVE_READONLY`).
**Nicht getan:** kein Produktivcode, keine PRs/Releases/Issues, keine Contracts, keine Zielarchitektur, keine Änderung an bestehenden Audit-Dateien oder Lastenheften.

Zeilenangaben beziehen sich auf `origin/main`, sofern nicht `@audited` (= von Sonnet gelesener Snapshot) vermerkt ist. Die Detailtabelle aller Einzelbefunde steht in `MEDIA_SYSTEM_AUDIT_OPUS_FINDINGS.csv` (IDs `OPR-xxx`).

**Bewertungsachsen (pro Befund beide):**
- Verifikation: `CONFIRMED` · `PARTIALLY_CONFIRMED` · `INCORRECT` · `MISSING_FROM_AUDIT` · `AMBIGUOUS` · `INSUFFICIENT_EVIDENCE` · `AUDIT_INTERNAL_CONFLICT`
- Version: `CONFIRMED_CURRENT` · `OUTDATED_SNAPSHOT` · `INCORRECT_AT_SOURCE` · `CURRENT_NEW_FINDING` · `AMBIGUOUS_VERSION`
- Aussageart: `IST_FACT` · `INTERPRETATION` · `ARCHITECTURE_ASSUMPTION`

---

## 1. Executive Verification Result

### 1.1 Versionsbasis — der wichtigste Einzelbefund

Der Sonnet-Audit beschreibt **nicht** den aktuellen IST-Stand. Er wurde gegen lokale Feature-Branch-Checkouts vom 14.–31. Juli 2026 erstellt, obwohl er sich als aktueller IST ausweist. Einer davon (`media_policy`) war ein **nie gemergter** Branch.

| Repo | Von Sonnet auditiert (rekonstruiert, Manifest-Versionen stimmen mit Sonnet überein) | `origin/main` (= live installiert, `LIVE_READONLY`) | Differenz | Audit-relevante Änderungen seit Snapshot |
|---|---|---|---|---|
| `benni_media_state` | `codex/control-28-state-hotfix` @ `a047734` (2026-07-14, v0.13.3) | `c371b32` (2026-09-07, v0.14.5) | 15 Commits hinter main (19 Dateien, +1 182/−95) | Debounce 4 s → 2 s + 6 s Deckel (PR #19, benni_media#13); Source-Arbitration mit LG-5-s-Grace, 20-s-TV-Startschutz, Priorität ATV > PS5 > TV und explizite `device`-Zuordnung pro Kontext (PRs #22/#27); Konsum der Master-Attribute `is_active` für PC/PS5/Switch/Denon (PR #26); Enum 3 `grind_preemptible` (im Snapshot nicht vorhanden); Apple-TV-Prefill `…_lan` (PR #20, benni_media#16); Artwork-Fallback (PR #28) |
| `benni_media_policy` | `claude/control-45-bio-sleep-modifier` @ `622be27` (2026-07-20, v0.18.0) — **nicht in main enthalten** (1 ahead / 16 behind, Merge-Base `e70c74e`) | `3392cce` (2026-08-30, v0.18.2) | eigener, unveröffentlichter Zweig | Auf main wurde eine **andere** control#45-Umsetzung gemergt (`036fb61`): kein `resume_blocked_by_sleep()` (waking blockiert Resume nicht mehr), Schlaf setzt HomePods-Ziel auf `0.0` statt `None`; #59: `provisional_sleep` zählt als Sleep |
| `benni_media_apply` | `codex/control-28-delayed-reapply` @ `2daafd3` (2026-07-14, v0.16.1) | `639ba1b` (2026-09-13, v0.19.15) | 37 Commits hinter main (28 Dateien +5 965/−203, davon `custom_components` +2 595/−159) | Radiostart-Sperre im Schlaf (control#45, v0.16.2); Denon-Volume sofort (PR #29, benni_media#13); stale-`media_device`-Power-Override (PR #30, benni_media#14); Per-Pod-Volume, Screen-/Owner-Startsperre, Denon nie Volume 0 (PRs #31–#37, benni_media#16); Radio-Dispatch-Cooldown (PR #38); Wake-Single-Flight und Recovery inkl. optionalem MA-Add-on-Restart (PR #42); Sleep-TV-Evidence + Store (PRs #43/#54, core-state#59); Stuck-Mute-Reparatur (PRs #44/#49); Resume-Reconciliation (Issue #41, PRs #45/#50/#51); R12-Episoden mit `episode_id` (Issue #46, PR #48); Stop-Latch-Reset auf Bio-Wake-Flanke (PR #52); state-getriebene Readmission (PR #55) |
| `benni_media` (UX) | `codex/control-28-cockpit-null-hotfix` @ `fc2d5a7` (2026-07-14, v0.7.5) | `6b4b5db` (2026-09-09, v0.7.8) | 13 Commits hinter main | in `custom_components` nur Frontend-Bundle `frontend/app/main.js` und Manifest; `aggregator.py` unverändert |
| Lastenheft (`einhornzentrale`) | `codex/control-28-media-priority-spec` @ `846ccd5` (2026-07-14; nur lokaler Branch, kein Remote-Ref) | `0640ffc` (2026-09-13) | 1 ahead / 10 behind | R23: „PC-Ein" aus der Triggerliste entfernt (`lastenheft.md:270`, PR #34 „remove PC from bio wake contract"); `40_automations.md:94`: PC-Aktivität bleibt Activity-State-Eingang ohne Bio-Wirkung; kleine Änderungen an `packages/media/media_automations.yaml`, `templates/radio.yaml`, `packages/system/manual_bio_scripts.yaml` (zusammen 10 Zeilen) |
| `benni-core-state` | `c9408fa` (2026-09-02) | identisch | 0 | `bio_state` ∈ {`sleep`,`provisional_sleep`,`waking`,`awake`} seit 2026-08-07 — war im gelesenen Checkout bereits vorhanden |
| `core-contracts` | `agent/issue-8-published-options-flow` @ `bc3cb90` (2026-07-31) | `5160aef` (2026-09-14) | 1 ahead / 28 behind | Registry/Exchange-Layer, Consumer-API-Doku mit geplantem Contract `media.activity.v1` |

`LIVE_READONLY`: `update.benni_media_state_update` v0.14.5, `…policy…` v0.18.2, `…apply…` v0.19.15, `update.benni_media_update` v0.7.8 — jeweils `installed == latest`. **`origin/main` ist damit auch der laufende Code.**

Hinweis zur Nachvollziehbarkeit: Während dieses Reviews wurden die lokalen Checkouts der vier Media-Repos und von `core-contracts` von dritter Seite auf `main` umgestellt. Die Sonnet-Snapshots waren zuvor per `git archive` gesichert; die Rekonstruktion bleibt belastbar (Versionen `0.13.3/0.18.0/0.16.1/0.7.5` stimmen mit Sonnets eigenen Angaben überein).

### 1.2 Belastbarkeit

**Belastbar (auch auf main):** das Grundgerüst State → Policy → Apply → UX; die Kontextwerte und Prioritäten; die textuelle Dopplung von `media_block_reason()`; der tote UX-Pfad `toggle_private`; das Fehlen des Anwesenheits-Simulators; R21-Nudge-Reset fehlt; die RAM-Natur des Policy-`OrchestratorState`; der Verlust der R13/R14-Nachlauf-Flanke bei Restart/Reload; die Direktbindungen von Apply an State-Entities.

**Echte Fehler bereits im Snapshot (`INCORRECT_AT_SOURCE`):**
- „`media_apply` ist das einzige Modul mit realen HA-Service-Calls" — falsch: Umbrella ruft `homeassistant.toggle`, YAML-Skripte in `einhornzentrale` schreiben HomePods und Stop-Latch (OPR-010).
- Apply leite Denon-Aktivität bevorzugt aus `powered/is_active/watt_active` ab — falsch: `_denon_power_on()` liest den **äußeren State-String** (`_tri_bool`), die Attribut-Präferenz gilt nur für PC/TV (OPR-111).
- AUD-RULE-008: PS5-Gaming setze einen ETM-Titel voraus — falsch: PS5 an ⇒ Gaming, der Titel verfeinert nur den Subkontext; das Titel-Gate gilt nur für PC (OPR-146).
- „`bio_state=sleeping` erzeugt eine kritische Live-Divergenz" — falsch: Core State hat `sleeping`/`asleep` nie emittiert (auch nicht historisch). Die **reale** Divergenz betrifft `provisional_sleep` und wurde übersehen (OPR-031/032).
- `media_stop_latch`-Schreiber: **beide** widersprüchlichen Sonnet-Aussagen sind falsch (kein Wake-Planner-Schreiber; Apply ist nicht der einzige) (OPR-130).
- Lastenheft-R10: Sonnet gab die Diff-Zeile zum gestrichenen Auto-Latch (`lastenheft.md:465`) nur als „Bewusst gestrichen" wieder; die Ersatzspalte derselben Zeile lautet aber „private_time via **Stash**/manuell" und deckt damit den automatischen Pfad des Codes (OPR-052).

**Überholt (`OUTDATED_SNAPSHOT`, im Snapshot korrekt):** „keine Episode-ID", „Apply ohne Store-Persistenz", „`sleep_tv_evidence` wird nie veröffentlicht", `null`-Doppelbedeutung der Volume-Targets (nur im ungemergten Policy-Branch), „waking blockiert Resume", R12-Semantik, Debounce-Werte, stiller YAML-Script-Fallback beim Radiostart, Sonnets Start-Pfad-Modell (3 Pfade).

**Interpretationsprobleme:** Mehrere von Sonnet als Dubletten markierte Begriffe sind **absichtlich getrennte Ebenen** (Private-Time-Kette Actual → Desired → Execution; Away: Klassifikation vs. Soll-Folge vs. Ausführung; `quiet_mode`-Doppellesung). Umgekehrt wurde `audio_owner` zu sauber und `device` zu harmlos eingeschätzt. Die Core-Contract-Kandidaten enthalten Architekturannahmen, die als IST formuliert sind (z. B. „`media_blocked` sollte von media_state kommen").

### 1.3 Vor jeder Architekturarbeit zwingend zu klären

1. **Sleep-Semantik je Schicht für `provisional_sleep`** (OPR-032): Policy behandelt PS als Schlaf, Apply teils als wach (R14-Nachlaufpause, Autostart-Vorgate), teils als Schlaf (Wake-Unterdrückung, Start-Admission, R24).
2. **Master-Aktivitätsvertrag** (OPR-111): State liest `is_active`, Policy und Apply lesen den äußeren State-String desselben Masters; `benni_media_state#25` (offen, testing) belegt live `unknown` + `is_active=true`.
3. **Start-Autorität** (Abschnitt 6): Auf main gibt es einen konsolidierten Apply-Owner, aber daneben ungegatete bzw. anders gegatete Startwege (manueller UX-Radiostart, YAML-Skripte `system_mark_*`).
4. **Stop-Latch-Ownership** (OPR-130/063): kein Owner; Policy-internes `manual_stop` wird vom externen Latch überschattet.
5. **`audio_owner`- und `device`-Semantik** (Abschnitt 5): beide tragen je zwei Bedeutungen.

---

## 2. Confirmed Major Findings (gegen main verifiziert)

| ID | Befund | Evidenz |
|---|---|---|
| OPR-020 | `media_block_reason()` ist in Policy und Apply textgleich (gleiche Inputs `away_gate`/`presence_state`/`presence_degraded`, gleiche Rückgabewerte). `CONFIRMED` / `CONFIRMED_CURRENT` / `IST_FACT` | `benni_media_policy/…/logic.py:292-306`, `benni_media_apply/…/logic.py:622-635` |
| OPR-140 | UX-Quick-Action „Private Time" ist tot: `aggregator.py` sucht `bindings()["private_manual_entity"]`, der Key existiert seit media_state 0.11.0 nicht mehr; das Frontend auf main ruft die Aktion weiterhin auf. `CONFIRMED` / `CONFIRMED_CURRENT` | `benni_media/…/aggregator.py:289-297`, `const.py:49-51`, `benni_media/frontend-src/src/pages/OverviewPage.tsx:47`, `benni_media_state/…/const.py:298-310` (kein Key in `WATCH_KEYS`) |
| OPR-143 | Anwesenheits-Simulator (R26–R29) fehlt vollständig — auch in der YAML-Konfiguration. `CONFIRMED_CURRENT` | grep `simulator|schwanz|podigee` = 0 in allen Media-Repos und `einhornzentrale@main` |
| OPR-144 | R21 „Nudge-Reset bei Audio-Kontext-Wechsel" nicht implementiert; nur expliziter Reset. `CONFIRMED_CURRENT` | `benni_media_policy/…/coordinator.py:511-524` |
| OPR-084 | Policy-`OrchestratorState`, `_manual_nudge`, `_boost_suppressed`, `_homepods_idle_since` sind RAM; nur die Matrix ist Store-persistiert. `CONFIRMED_CURRENT` | `benni_media_policy/…/coordinator.py:136-148`, `storage.py:13-19` |
| OPR-080 | R13/R14: `NachlaufState` inkl. `last_pc_on/last_tv_on` ist RAM; nach Restart/Reload startet der erste `_compute()` mit `None`, ein bereits ausgeschalteter PC/TV erzeugt keine Flanke, es gibt keinen Startup-Reconcile; laufende Tasks werden beim Unload abgebrochen. `CONFIRMED_CURRENT` (Präzisierung s. Abschnitt 8) | `benni_media_apply/…/logic.py:1020-1130`, `coordinator.py:205, 409-421, 576-625`, `__init__.py:47-71` |
| OPR-100 | `media_device` wird faktisch als Routing-/Capability-Information genutzt (Details Abschnitt 5). `CONFIRMED_CURRENT`, auf main sogar ausgeweitet | `benni_media_apply/…/const.py:87-105`, `logic.py:1269-1305, 1393-1399`, `benni_media_policy/…/logic.py:321-327, 863-871` |
| OPR-148 | `VOL_POLICY_MUTED` und `AUDIO_OWNER_SLEEP` sind definierte, nie emittierte Werte. `CONFIRMED_CURRENT` | `benni_media_policy/…/const.py:73, 116`; `logic.py:578-684` |
| OPR-145 | R23 wird im Code ausschließlich über die Bio-Flanke ausgelöst; `CONF_WAKE_TRIGGERS` ist leer vorbelegt. `CONFIRMED_CURRENT` | `benni_media_apply/…/const.py:241-244`, `coordinator.py:2304-2321` |
| OPR-164 | Die fünf Kontextwerte `idle/tv/streaming/gaming/private_time` entsprechen Lastenheft §4.1. `CONFIRMED_CURRENT` | `benni_media_state/…/const.py:48-53` |
| OPR-151 | Kein Test importiert oder verkettet Logik eines anderen Media-Moduls (State→Policy→Apply). `CONFIRMED_CURRENT` — aber präzisiert, s. Abschnitt 13 | grep über alle `tests/` |

---

## 3. Incorrect or Overstated Findings

| ID | Sonnet-Aussage | Problem | Korrekte Aussage | Evidenz |
|---|---|---|---|---|
| OPR-001 | „aktueller IST-Zustand" | Gelesen wurden Feature-Branches von Juli, einer ungemergt | Audit = historischer Snapshot; main weicht in Apply massiv ab | Abschnitt 1.1 |
| OPR-010 | Apply ist das einzige Modul mit realen HA-Service-Calls (MD §1); zugleich listet MD §2.4 einen Service-Call der Umbrella | interner Widerspruch + unvollständig | Service-Writer: Apply (viele), Umbrella (`homeassistant.toggle`, indirekt manueller Radiostart), YAML-Skripte/-Automationen (`music_assistant.play_media`, `media_player.media_stop`, `input_boolean.turn_on/off`, `wake_on_lan`); Policy/State schreiben nur Config-Einträge | `benni_media/…/aggregator.py:294-296`; `einhornzentrale/packages/media/media_scripts.yaml:1-51`; `packages/system/manual_bio_scripts.yaml:8-47` |
| OPR-031 | `sleeping` in Policy-Set ⇒ „CRITICAL" Divergenz | Wert wird nie emittiert (auch historisch nicht; `git log -S` leer) | latente Dead-Value-Divergenz; Herkunft vermutlich Legacy-Template `sensor.benni_combined_context_bio_state` (`INFERRED`) | `benni-core-state/…/const.py:162-166`; `einhornzentrale/packages/media/templates/radio.yaml:126-131` |
| OPR-111 | Apply bevorzugt `powered/is_active/watt_active` für Denon | falsch gelesen | Apply-Denon: `_tri_bool(CONF_DENON_POWER)` = äußerer State-String; Attribut-Präferenz nur `_powered()` für PC/TV; Policy liest ebenfalls State-String; State (main) liest `is_active` | `benni_media_apply/…/coordinator.py:467-514` (auch `@audited:301-340`); `benni_media_policy/…/coordinator.py:397-398`; `benni_media_state/…/coordinator.py:404-412, 687-688` |
| OPR-146 | AUD-RULE-008: PS5-Gaming benötigt ETM-Titel; 02-CSV: `ps5_raw` gatet PS5-Gaming | falsch | PS5 an (oder Vordergrund PS5) ⇒ Gaming; ohne Titel `gaming_grind` bzw. sticky; Titel-Gate nur PC | `benni_media_state/…/logic.py:548-578` (`@audited:391-421` identisch) |
| OPR-130 | 04: Wake-Planner-getriggerter Schreiber; 07/02/03: nur Apply schreibt | beide falsch | vollständige Liste s. Abschnitt 7 / OPR-130 | Abschnitt 7 |
| OPR-052 | Die Diff-Tabelle (Kap. 12) dokumentiere die bewusste Streichung eines automatischen Eintritts; der Code ähnle diesem gestrichenen Verhalten (MD §13 CONTRADICTION) | unvollständig zitiert | gestrichen ist der „Auto-Latch via Denon+PC+Nacht"; Ersatzspalte derselben Zeile: „via Stash/manuell"; der R10-Text („Kein automatischer Eintritt") widerspricht damit Tabelle **und** Code; der aktuelle Vertrag in `benni_media_state#25` verlangt für Auto Private Time Classifier UND PC UND Denon | `einhornzentrale/docs/lastenhefte/reviewed/media/lastenheft.md:465` (identisch `00_overview.md:236`); `lastenheft.md:209-211`; GitHub `Levtos/benni_media_state#25` |
| OPR-021 | `media_block_reason()` = dieselbe fachliche Entscheidung zweimal, „HIGH" | überzeichnet | geteilt ist nur die Klassifikationslesung; Folgeentscheidungen unterscheiden sich pro Ebene (Abschnitt 5); `presence_state=="abwesend"` ist redundant zu `away_gate` (beide aus demselben Gate in einem Compute) | `benni_media_state/…/logic.py:280-291, 686-698` |
| OPR-060 | drei unabhängige Startpfade | für main unvollständig/überholt; im Snapshot ebenfalls unvollständig (YAML-Skripte, manueller Play fehlen) | siehe Start Authority Map (15 Einträge) | Abschnitt 6 |
| OPR-050 | drei Antworten auf „ist Private Time aktiv" = Red Flag | Ebenen verwechselt | Actual → Desired → Execution-Trigger; echter Mangel nur im Unknown-Handling von Apply (OPR-051) | Abschnitt 5 |
| OPR-040 | `audio_owner` sauberer Single-Owner, `actual_state` | Semantik gemischt | Priorität des fachlichen Stacks aus Kontext **plus** beobachtete HomePods-Wiedergabe | Abschnitt 5 |
| OPR-080 | Denon kann „unbegrenzt" eingeschaltet bleiben | Mechanismus korrekt, Umfang überzeichnet | bleibt an, bis eine andere Off-Quelle greift (Away-Gate-Off, neue PC/TV-Flanke, neuer Private-Exit, manuell) | Abschnitt 8 |
| OPR-023 | CC-06 „media_blocked … am natürlichsten von media_state" | als IST formulierte Architekturannahme | `ARCHITECTURE_ASSUMPTION`; State publiziert die Fakten (`away_gate`) bereits | `benni_media_state/…/entities.py:75-83` |

---

## 4. Missing Findings (von Sonnet komplett übersehen)

| ID | Befund | Version | Evidenz |
|---|---|---|---|
| OPR-032 | **`provisional_sleep` wird in Media uneinheitlich behandelt.** Policy: Sleep (Resume-Block, HomePods-Ziel 0, Pause). State: Private-Latch-Räumung nur bei `sleep`. Apply: `_bio_sleep()` nur `sleep` → R14-Nachlauf wird in PS **nicht** pausiert, `should_autostart_radio` lässt PS durch; gleichzeitig prüfen `decide_wake`, Start-Admission, Repair-Gate und R24 `bio_state ∈ {PS,S}`. Im Snapshot kannte **kein** Media-Modul PS, obwohl Core State es seit 2026-08-07 emittierte. | `CURRENT_NEW_FINDING` (im Snapshot als latente Lücke vorhanden) | `benni_media_policy/…/const.py:83-86`; `benni_media_state/…/const.py:502`; `benni_media_apply/…/coordinator.py:516-523, 1398-1399`, `logic.py:392-408, 484, 1104, 1480, 1612`; `benni-core-state/…/logic.py:1095-1102` |
| OPR-012 | Manueller Radiostart der UX (`async_play_radio`) umgeht nicht nur `apply_enabled`, sondern **alle** Start-Gates (Stop-Latch, Schlaf, Presence, Audio-Owner). Kommentar dokumentiert nur den Shadow-Bypass. | `CONFIRMED_CURRENT` (auch Snapshot) | `benni_media_apply/…/coordinator.py:2688-2707`; `benni_media/…/aggregator.py:279-281` |
| OPR-014 | Config-Schreibpfade lösen einen Reload aus und verwerfen allen RAM-State: Apply-Schalter „Automatik scharf"/`set_apply_enabled`, Policy-`set_apply_enabled`/`set_scalars` (Update-Listener → `async_reload`). Matrix-Patches schreiben nur Store (kein Reload). | beide Stände | `benni_media_apply/switch.py:48-53`, `coordinator.py:2761-2764`, `__init__.py:69, 88-89`; `benni_media_policy/…/coordinator.py:307-317, 506-509`, `__init__.py:59, 78-79` |
| OPR-013 | R12 hängt an einer YAML-Automation: `media_player.turn_on` triggert `automation.living_tv_turn_on_via_wake_on_lan` (`webostv.turn_on`, gated `binary_sensor.benni_core_state_apply_ready`), die das Magic-Packet sendet; Apply sendet selbst nur mit konfigurierter MAC (Default leer). Live: Automation `on`. | `CURRENT_NEW_FINDING` | `einhornzentrale/packages/media/media_automations.yaml:1-14`; `benni_media_apply/…/const.py:382-383`, `coordinator.py:2246-2266`; `docs/issue-46-tv-wol-episode.md:37-40`; `LIVE_READONLY` |
| OPR-092 | Die YAML-Automation `media_radio_resume_after_manual_playback` dupliziert Apply-Trigger B; Paket ist geladen, die Automation live **aus** (zuletzt ausgelöst 2026-05-25). | Config-Dublette, Runtime inaktiv | `packages/_entry/04_media.yaml`; `media_automations.yaml:16-58`; `LIVE_READONLY` |
| OPR-063 | Policy löscht `manual_stop` bei steigender `wake_needed`-Flanke nur für **einen Tick**: steht der externe Latch weiter auf `on`, setzt der nächste Tick `manual_stop=True` erneut. Der effektive Reset kommt nur über Apply (Bio-Wake-Flanke) oder YAML. | `CURRENT_NEW_FINDING` (auch Snapshot) | `benni_media_policy/…/logic.py:464-474` |
| OPR-051 | Apply bildet `private_active = (audio_owner == "private_stack")` als Bool; `unknown/unavailable` wird zu `False` (Docstring behauptet `None`). Ein Policy-Ausfall während Private erzeugt so eine Exit-Flanke → ggf. 15-s-Denon-Off-Delay. Auswirkung `INFERRED`. | `CURRENT_NEW_FINDING` | `benni_media_apply/…/coordinator.py:571`, `logic.py:113-115, 1179-1216` |
| OPR-042 | Frontend markiert den Private-Button aktiv bei `audio_owner === "private"`; der Wert heißt `private_stack` → nie aktiv. | beide Stände | `OverviewPage.tsx:47`; `benni_media_policy/…/const.py:69` |
| OPR-101 | Snapshot-Bug: `device` = erstes erkanntes Gerät (ATV > TV > PS5 > …) unabhängig vom gewählten Kontext; PC-Gaming bei laufendem TV ergab `device=tv` → `is_pc_gaming()` falsch. Auf main behoben (explizite Zuordnung, Priorität ATV > PS5 > TV). | `OUTDATED_SNAPSHOT` (Sonnet übersah ihn) | `@audited benni_media_state/…/logic.py:361-377, 571-578`; main `logic.py:515-534, 743-779` |
| OPR-033 | `evaluate_quiet()` wertet `activity_state ∈ {sleep, asleep, quiet}` als Quiet; Core State kennt nur `sleep`. `activity=sleep` (Bio S/PS ohne TV) aktiviert damit Ducking, Subwoofer-Aus und den R20-Snapshot — ein zweiter, von `bio_state` unabhängiger Schlafkanal. | beide Stände | `benni_media_state/…/logic.py:321-335`; `benni-core-state/…/const.py:191-206`, `logic.py:1590-1599` |
| OPR-121 | PS → `awake` (Aktivitäts-Wake in Core State) zählt in Apply als Wake-Flanke (R23-Sequenz, Trigger-A-Radiostart) und als Stop-Latch-Reset-Flanke. Für den Latch-Reset ausdrücklich so beschlossen (#52); für die Wake-Sequenz nicht explizit dokumentiert. | `CURRENT_NEW_FINDING` | `benni_media_apply/…/coordinator.py:627-661, 2293-2302`; `benni-core-state/…/logic.py:1095-1102`; GitHub PR `benni_media_apply#52` |
| OPR-141 | Live heißt der Sensor `sensor.system_benni_media_apply_sleep_tv_evidence` (Renamed-Device-Präfix); Core-State-Prefill bindet `sensor.benni_media_apply_sleep_tv_evidence` (live 404). Ob eine Options-Bindung das überschreibt: `INSUFFICIENT_EVIDENCE`. Apply-Entities haben gemischte Präfixe (`benni_*` / `system_benni_*`). | `CURRENT_NEW_FINDING` | `benni-core-state/…/const.py:299`; `LIVE_READONLY` |
| OPR-153 | Kommentare widersprechen dem Code: media_state-`Inputs` nennt `binary_sensor.benni_core_state_away` als Away-Quelle (real `presence_personal`); Coordinator-Docstring „Debounce 4 s" (real 2 s); Apply-Modul-Docstrings „R20/R13/R24 folgen" und „start_radio an Script delegiert"; `decide_wake` nennt Private-Time als Trigger, unterdrückt ihn aber; Apply-Prefill-Kommentar „Denon … (is_active)" liest State-String; Umbrella-Kommentar verweist auf nicht existierende Konstante `CONF_PRIVATE_MANUAL`. | `CURRENT_NEW_FINDING` | `benni_media_state/…/logic.py:118-122` vs. `coordinator.py:659`; `coordinator.py:3-6` vs. `const.py:436`; `benni_media_apply/…/logic.py:11-13, 1595-1621`, `coordinator.py:11`, `const.py:233-236`; `benni_media/…/const.py:49-51` |
| OPR-142 | `core-contracts@main` dokumentiert bereits einen geplanten Contract `media.activity.v1` (Owner MediaState, Consumer CoreState) und das Rollenbeispiel `media.audio_player.living_room`; Media-Cutover ausdrücklich separater Auftrag. | `CURRENT_NEW_FINDING` | `core-contracts/docs/consumer-api-v1.md:38-60, 206-222`; `docs/lastenheft-registry-exchange-layer-v1.md:54-66` |
| OPR-004 | Code-Verweise `control#3`/`control#45` sind GitLab-Nummern; GitHub `control#3`/`#45` sind fachfremd. `control#45` → `benni_media_policy#22` (Provenance GitLab-Work-Item 45); für `control#3` keine GitHub-Entsprechung auffindbar. | `INSUFFICIENT_EVIDENCE` für die Entscheidungs-Herkunft | `gh issue view` (Abschnitt 12) |

---

## 5. Actual vs Desired vs Execution Semantic Review

| Begriff | Producer | Tatsächliche Semantik (main) | Klasse | Sonnet-Klasse | Bewertung |
|---|---|---|---|---|---|
| `context` | State | beobachtetes Szenario aus Rohsignalen, Away → `idle` | actual_state | actual | `CONFIRMED` |
| `audio_scenario` | Policy | Soll-Audio aus Konstellation; `owner none` ⇒ `music` | desired_state | desired | `CONFIRMED`; kein Apply-Consumer |
| `audio_owner` | Policy | `private_stack/gaming_stack/tv_denon` = **Vorrang des fachlichen Stacks aus `context`** (unabhängig von realem Gerätezustand: `tv_denon` auch bei Denon aus); `homepods` **nur wenn HomePods beobachtet `playing`**; sonst `none`; Away ⇒ `none` | **gemischt** (desired precedence + observed playback) | actual | `PARTIALLY_CONFIRMED` / `INTERPRETATION`. Apply kompensiert die Mischung: vor dem ersten Play wird `none` künstlich zu `homepods` gesetzt (`coordinator.py:1383-1386, 1473-1478`) |
| `action` | Policy | Instruktion aus Zustandsmaschine | desired (Instruktion) | desired | `CONFIRMED`; Apply filtert erneut (`logic.py:865-901`) und leitet auf main in eigene Episoden um |
| `manual_stop` | Policy | internes Flag, überschrieben durch externen Latch; Wake-Reset nur 1 Tick | execution/user-intent-mirror | execution | `PARTIALLY_CONFIRMED` + OPR-063 |
| `media_block_reason()` Policy | Policy | Klassifikationslesung → **Desired**-Folge: Owner none, Szenario off, Pause, Volumes 0, Sub aus, Resume-Erinnerung behalten | desired | actual | Ebene falsch zugeordnet |
| `media_block_reason()` Apply | Apply | dieselbe Lesung → **Execution**-Folge: Pause, Denon `turn_off`, Sub aus, `EXEC_IMMEDIATE`, Wake/Resume abbrechen, Dispatch/Repair blockieren | execution (+ eine Soll-Entscheidung: Denon aus bei Away, in Policy nicht abgebildet, OPR-022) | actual | Ebene falsch zugeordnet |
| `device` | State | (1) primäre Quelle/Identität; genutzt als (2) Screen-Intent R12 (`tv/appletv`), (3) Denon-Konsument (`tv/appletv/ps5/switch/pc`, mit Power-Gegenprobe nur pc/tv), (4) R12-Episodenende (`none/denon/homepods/pc/ps5/switch`), (5) PC-Headset-Routing (`is_pc_gaming`), (6) Denon-Audiopfad (`device==denon`), (7) Core-State-Eingang | actual + faktisch Routing/Capability | actual (mit Hinweis) | `CONFIRMED_CURRENT`, ausgeweitet. `LIVE_READONLY`: `media_device=pc` bei `audio_owner=homepods` → PC zählt als Denon-Konsument, obwohl Audio über HomePods läuft |
| `private_time_active` | State | Eintritt auto (Classifier+PC+Denon) oder manuell (Switch+PC); sofortiger Exit; Away ⇒ false | actual_state | actual | `CONFIRMED` |
| `private_active` | Apply | `audio_owner == private_stack`; Trigger für Private-Exit-Delay und Wake-Sperre | execution-trigger | „dritte Antwort" | Ebene nicht erkannt; Unknown→False-Mangel (OPR-051) |

### Semantik-Matrix Private Time

| Frage | `private_time_active` (State) | `audio_owner == private_stack` (Policy) | `private_active` (Apply) |
|---|---|---|---|
| Ebene | Actual | Desired (Stack-Vorrang) | Execution-Trigger |
| Eintritt | Classifier ∧ PC ∧ Denon ∨ Switch ∧ PC (`logic.py:370-387`) | `context == private_time` (`logic.py:258-259`) | `_state(audio_owner) == private_stack` (`coordinator.py:571`) |
| Austritt | sofort bei Wegfall einer Pflichtbedingung; Away | Kontextwechsel; Away (`media_block_reason`) | Wert ≠ `private_stack` **inkl. unknown/unavailable** |
| Latenz | State-Debounce 2 s | + Entity-Propagation | + Entity-Propagation; Apply-Tick ungedrosselt |
| Kann A wahr, B falsch sein? | ja: Policy nicht geladen/unavailable | ja: Away mit `presence_state=abwesend` vor Kontext-Update (`INFERRED`, gleiche Compute-Quelle) | ja: `unknown` → falsch trotz aktivem Private |
| Gewollt? | ja (Kette) | ja | Kette ja; Unknown→False nicht dokumentiert (Docstring widerspricht) |

**Fazit:** keine Zusammenführung aus IST ableitbar. Belegter Mangel ist ausschließlich das Unknown-Handling in Apply.

---

## 6. Start Authority Map (origin/main)

Legende Gates: L = Stop-Latch, S = Schlaf, P = Presence (away/unknown), O = konkurrierender Owner/Screen, A = `apply_enabled`.

| # | Integration · Funktion | Trigger | Gates / Conditions | Hidden State | Retry / Delay | L | S | P | Konkurrierende Pfade | Service-Call |
|---|---|---|---|---|---|---|---|---|---|---|
| S1 | Policy · `music_baseline_candidate` + `decide_action` (`logic.py:353-376, 500-511`) | HomePods ≥30 s stabil nicht `playing` (Timer) | owner none, kein Grind, Station gesetzt, kein Quiet, kein Manual, kein Wake-Fenster (`wake_needed` ∨ `waking`), kein `manual_stop` | `_homepods_idle_since`, `_baseline_timer`, `OrchestratorState` | Timer re-armt alle 30,5 s solange idle (`coordinator.py:441-449`) | via `manual_stop` | `bio_sleep` (PS+S) | ja | S2, S5 | keiner (Soll `start_radio`) |
| S2 | Policy · Resume nach Auto-Pause (`logic.py:512-523`) | konkurrierender Stack endet nach Auto-Pause | `pre_pause_mode` radio/manual | `auto_paused`, `pre_pause_mode`, `last_*` | – | via `manual_stop` | PS+S | ja | S1 | keiner (Soll `start_radio`/`resume_homepods`) |
| S3 | Apply · Plan-Filter `decide_apply` → `_execute` → `_schedule_resume_reconciliation(source=policy|tv_resume)` (`logic.py:865-901`, `coordinator.py:1110-1126, 1239-1281`) | Policy-`action` | nicht spielend, kein L, kein Private-Exit-Suppress, `radio_ready` nicht false, kein Manual; danach Admission `_playback_start_block_reason` (A, P, S=`sleep`, L, Policy-Pause, Resume-Erlaubnis — für `policy` überschrieben auf true —, radio_ready true, Manual, O, `bio_state∈{PS,S}`); Wake-Owner unterdrückt Parallelstart (`logic.py:594-602`) | `_pending_plan`, `_wake_start_owned`, `_playback_episode_admitted`, `_resume_episode_finished`, `_resume_content_*` | R2-Debounce 5 s (Deckel 8 s); Episode genau einmal | ja | ja | ja | S5, S7, S4 | `media_player.media_play` (Resume mit gemerktem Content), `music_assistant.play_media` (replace), `media_player.volume_mute` |
| S4 | Apply · `_maybe_schedule_policy_readmission` (`coordinator.py:1333-1365`) | transienter Blocker (`resume_not_allowed`, `volume_apply_not_allowed`, `presence_unknown`, `radio_not_ready`, `audio_owner_unproven`) fällt weg | wie S3 | `_policy_readmission_blocker`, `_last_attempt_at` | Cooldown 30 s | ja | ja | ja | S3 | wie S3 |
| S5 | Apply · Wake-Trigger A `_schedule_radio_autostart` → `_run_radio_autostart` (`coordinator.py:649-661, 709-712, 1213-1237, 1809-2003`) | Bio-Flanke nicht-wach → `awake/waking` (inkl. PS→awake) oder optionale Zusatztrigger | `decide_wake` (P, S∨PS/S, Private), A, Autostart-Option, `should_autostart_radio` (P, S=`sleep`, radio_ready true, kein Manual, geplante Station nicht spielend, O); Admission: **Latch für Wake-Admission ignoriert** (`coordinator.py:1389-1390`), weil Reset auf derselben Flanke asynchron läuft | `_last_bio_state`, `_wake_start_owned`, Recovery-Stage/-Versuche | Lead 1 s; Settle 60 s; Health 3×5 s; genau 1 Soft-Recovery (replace); optional Hard-Recovery nach 300 s | Admission nein, danach ja (Repair-Gate) | ja | ja | S3, S6 | `music_assistant.play_media`; `media_player.volume_mute`; optional `hassio.addon_restart` + erneuter Dispatch |
| S6 | Apply · R23 `_run_wake` (`coordinator.py:2334-2376`) | wie S5 (`wplan.fire` ∧ A) | `media_block_reason` nach Debounce | `_wake_task`, `_ramp_task` | Floor 0,10 blockierend, 5 s, Ramp 16×1 s | – | via S5 | ja | parallel zu S5 (Race-Fix FLEET-42) | `media_player.volume_set` (pro Pod) — **kein Start** |
| S7 | Apply · Trigger B `_schedule_radio_resume` → `_run_radio_resume` (`coordinator.py:713-720, 2213-2228`) | `manual_playback` true → false | A, Autostart, nicht blockiert, kein Wake-Owner, `action≠pause`, `should_autostart_radio`; nach Delay Recheck; dann Reconciliation `source=resume` (erfordert Policy-`homepods_resume_allowed`) | `_last_manual_playback`, `_radio_resume_task` | 10 s | ja | `sleep` im Vorgate, PS erst in Admission | ja | S3, YAML S13 | wie S3 |
| S8 | Apply · Stuck-Mute-Recovery (`coordinator.py:396-406, 1750-1807`) | Pod-State-Event, 30-min-Backstop | Repair-Gate: gemanagte Episode (Wake-Owner ∨ geplante Station spielt), Bio `awake/waking`, Owner `homepods`, kein Quiet, Target > 0 u. a. (`logic.py:460-513`) | `_stuck_mute_task`, `_last_unmute_attempt_at`, `_stuck_mute_retry_not_before` | 2 Versuche × 2 s; Event-Cooldown 30 s; nach Fehlschlag 1800 s | ja | ja | ja | – | `media_player.volume_mute` false — lässt Wiedergabe hörbar werden, startet nicht |
| S9 | Apply · `async_play_radio` (manuell, via Umbrella `play_radio`) (`coordinator.py:2688-2707`) | Nutzeraktion | **nur** gebundener Player + `media_id` | – | – | **nein** | **nein** | **nein** | alle | `music_assistant.play_media` |
| S10 | Apply · Script-Fallback in `_dispatch_automatic_radio` (`coordinator.py:2191-2195`) | `media_id` fehlt und kein Wake-Owner | – | – | – | – | – | – | – | `script.turn_on` → S11. Alle Aufrufer laufen als Owner ⇒ faktisch unerreichbar (`INFERRED`) |
| S11 | YAML · `script.media_radio_start` (`media_scripts.yaml:1-32`) | Aufrufer S12/S13/S10/UI | `radio_ready` on, Latch off, Manual off | – | 2 s zwischen `play_media` und `media_play` | ja | nein | nein | S3/S5 | `music_assistant.play_media`, `media_player.media_play`. `LIVE_READONLY`: zuletzt ausgelöst 2026-09-05 (Aufrufer: `INSUFFICIENT_EVIDENCE`) |
| S12 | YAML · `script.system_mark_waking` / `system_mark_awake` (`manual_bio_scripts.yaml:8-30`) | manuell | keine Media-Gates außer S11 | – | 2 s | **löscht Latch** | setzt Bio | nein | S5 (Bio-Flanke löst zusätzlich Apply-Wake aus) | `benni_core_state.*`, `script.media_radio_clear_stop_latch`, `script.media_radio_start` |
| S13 | YAML · `automation.media_radio_resume_after_manual_playback` (`media_automations.yaml:16-58`) | Manual on → off | radio_ready, Latch, keine Policy-Pause, 10 s, Rechecks | – | 10 s | ja | nein | nein | S7 (Dublette) | `script.media_radio_start`. **Live aus.** |
| S14 | Apply · R20 Quiet-Restore (`logic.py:909-934`) | Quiet-Exit | Snapshot vorhanden, Gruppe adressierbar | `ApplyState.pre_quiet_*` | Ramp | – | – | – | S6 (beide rampen HomePods, `INFERRED` Überlappung bei `activity sleep → waking`, OPR-033) | `media_player.volume_set` — kein Start |
| S15 | Umbrella · `toggle_private` | Nutzeraktion | – | – | – | – | – | – | – | `homeassistant.toggle` — **tot** (OPR-140) |

**Unterschiedlich beantwortete Fragen zwischen Startpfaden (IST_FACT):**
- *Ist Schlaf?* S1/S2: `{PS,S,sleeping,asleep}`; S7-Vorgate: nur `sleep`; S3/S5-Admission: `{PS,S}`; S11/S12: gar nicht.
- *Blockiert der Latch?* S1–S4, S7, S11, S13 ja; S5-Admission bewusst nein; S9 nein; S12 löscht ihn.
- *Braucht es Policy-Resume-Erlaubnis?* S3 für `tv_resume` ja, für `policy`/`wake` überschrieben; S7 ja; S9/S11/S12 nein.
- *Konkurrierender Owner?* S3–S8 ja (`screen_blocks_music_start`); S1 implizit (owner none); S9/S11/S12 nein.

---

## 7. Stop / Block / Suppression Authority Map (origin/main)

| # | Integration · Stelle | Wirkung | Trigger/Bedingung | Evidenz |
|---|---|---|---|---|
| B1 | State · Away-Gate | Kontext `idle`, Entertainment aus | `presence_personal=abwesend` ≥ 25 s | `logic.py:258-277, 686-698` |
| B2 | State · Switch-Dock ignoriert | kein Switch-Gaming | hart `False` (FLEET-95) | `coordinator.py:676-683` |
| B3 | Policy · `media_block_reason` | Owner none, Szenario off, Pause, Volumes 0, Sub aus | away | `logic.py:923-953` |
| B4 | Policy · konkurrierender Stack | `pause_homepods` | Owner private/gaming/tv_denon, außer Grind/PC-Gaming | `logic.py:330-341, 484-487` |
| B5 | Policy · `bio_sleep` (PS+S) | Pause, kein Resume, HomePods-Ziel 0 | Bio | `logic.py:492-493, 609-610, 967` |
| B6 | Policy · `manual_stop` / Latch | kein Resume/Baseline | Latch on | `logic.py:464-468, 494-495` |
| B7 | Policy · Presence unknown | hält Resume, pausiert nicht | degraded/unknown | `logic.py:309-313, 496-499` |
| B8 | Policy · Quiet / R19-Mute | Ducking bzw. HomePods 0 | Tür/Anruf/Mute-Enum/`activity=sleep` | `logic.py:619-625, 642-643` |
| B9 | Apply · `away_block` | Pause, **Denon `turn_off`**, Sub aus, sofort; Wake/Resume abbrechen | away | `logic.py:843-863`, `coordinator.py:600-602, 1115-1116` |
| B10 | Apply · Plan-Filter | kein Start bei Latch/Suppress/radio_ready false/Manual | – | `logic.py:865-901` |
| B11 | Apply · Wake-Single-Flight | entfernt Policy-Startbefehle | Wake-Owner aktiv | `logic.py:594-602`, `coordinator.py:666` |
| B12 | Apply · `screen_blocks_music_start` | bricht verzögerten Resume ab | TV an ∨ Owner ∉ {homepods,none} | `logic.py:287-318`, `coordinator.py:608-609` |
| B13 | Apply · Start-/Repair-Gates | bricht gemanagte Episode ab | 13 bzw. 16 Gründe | `logic.py:411-513`, `coordinator.py:598-599` |
| B14 | Apply · Radio-Dispatch-Circuit | Cooldown 15 s, exp. Backoff ≤ 120 s | automatische Starts | `logic.py:321-389`, `const.py:301-302` |
| B15 | Apply · `homepods_volume_addressable` | kein Volume auf pausierter Gruppe | – | `logic.py:667-693` |
| B16 | Apply · Denon nie Volume 0 | – | Ziel ≤ 0 | `logic.py:972` |
| B17 | Apply · R13/R14 + Private-Exit | Denon `turn_off` nach 90 s bzw. 15 s | Flanke, kein Konsument | `coordinator.py:2574-2679` |
| B18 | Apply · R24 | TV `turn_off` nach absoluter Deadline, Warnung, R12-Consume | Bio PS/S ∧ TV aktiv | `coordinator.py:2495-2553` |
| B19 | Apply · R12-Consume | keine TV-Weckung | Startup, TV-Shutdown-Flanke, unbekannter Intent, Sleep-TV-Off | `logic.py:1331-1390` |
| B20 | Apply · Resume-Episode beendet / Content geändert | kein Re-Arm | – | `coordinator.py:1245-1256, 1434-1457` |
| B21 | YAML · `script.media_radio_stop`, `script.system_bedtime_mode` | Latch **on** + `media_player.media_stop` | manuell bzw. Hue-Dimmer `off_hold` | `media_scripts.yaml:34-43`; `manual_bio_scripts.yaml:32-47`; `system_automations.yaml:1-95`; ADR `2026-08-10-sleep-shutdown-trigger.md` |
| B22 | Alle Module · `apply_enabled` | Shadow | Option (je Modul eigene) | `benni_media_apply/…/coordinator.py:297-299`; `benni_media_policy/…/coordinator.py:159-161` |

### Auflösung des Sonnet-`AUDIT-INTERNAL-CONFLICT` (`input_boolean.media_stop_latch`)

| Aspekt | Befund (main) | Evidenz |
|---|---|---|
| Definition/Persistenz | YAML-`input_boolean` ohne `initial` ⇒ Zustand wird von HA über Neustart wiederhergestellt | `packages/media/input_boolean.yaml:1-3`, geladen via `packages/_entry/04_media.yaml` |
| Writer → **on** | `script.media_radio_stop`; `script.system_bedtime_mode` (ausgelöst durch Automation am Schlafzimmer-Hue-Dimmer) | `media_scripts.yaml:38-40`; `manual_bio_scripts.yaml:42-44`; `system_automations.yaml:1-95` |
| Writer → **off** | `script.media_radio_clear_stop_latch` (aufgerufen von `system_mark_waking`/`system_mark_awake`); `benni_media_apply` auf Bio-Flanke `sleep/PS → waking/awake` bei aktivem Apply (#52). Im Snapshot schrieb Apply `off` unbedingt beim Trigger-A-Start (`@audited coordinator.py:873-877`) | `media_scripts.yaml:45-51`; `benni_media_apply/…/coordinator.py:2269-2291` |
| Wake Planner | **kein** Writer (keine Referenz in `ha_wake_planner@main`) | grep |
| Weitere manuelle Writer (UI-Helper-Toggle, HomeKit, Dashboards) | Dashboards live: keine Referenz; YAML-Automationen/-Skripte sind über die HA-API nicht durchsuchbar; HomeKit-Include nicht nachweisbar → `INSUFFICIENT_EVIDENCE` | `LIVE_READONLY` `ha_search` (partial: YAML nicht scanbar) |
| Reader | Policy (`manual_stop`), Apply (Plan-Filter, Admission, Dispatch, Trigger B), `script.media_radio_start`, Automation S13 (live aus), Template `sensor.media_radio_plan`, Test `einhornzentrale/tests/test_sleep_trigger_contract.py:59` | – |
| Ownership | kein fachlicher Owner; YAML-Paket definiert den Helper, drei Systeme schreiben | – |
| Bewertung Sonnet | 04 (Wake-Planner-Automation) `INCORRECT_AT_SOURCE`; 07/02/03 (nur Apply) `INCORRECT_AT_SOURCE` | – |

---

## 8. Hidden Runtime State & Restart Verification

**Setup-Reihenfolge (main, alle `IST_FACT`):**
- State: `async_load_persisted()` (Pre-ATV) → `async_config_entry_first_refresh()` → `async_start()` (Listener) → Plattformen → Switch `RestoreEntity` stellt `private_manual` wieder her und rechnet neu (`__init__.py:25-45`, `switch.py:55-61`). Folge: der erste Compute läuft ohne Private-Latch; der Kontext kann beim Start kurz ohne `private_time` publiziert werden (`INFERRED` für Sichtbarkeit).
- Policy: Legacy-Migration → `async_load_matrix()` → erster Refresh mit leerem `OrchestratorState` → Listener + 09:00-Tick (`__init__.py:38-60`).
- Apply: Migration → `async_load_stored()` (Sleep-TV-Store) → erster Refresh → Listener, Pod-Listener, 30-min-Backstop → Plattformen; Unload bricht alle Tasks ab (`__init__.py:47-71`, `coordinator.py:266-271, 371-421`).

| Integration | State | Persistenz (main) | Restart / Reload | Sonnet | Bewertung |
|---|---|---|---|---|---|
| State | `_pre_atv` | Store, 5 s delayed save | überlebt | persistiert | `CONFIRMED_CURRENT` |
| State | `_private_manual` + 4-h-Timeout | RestoreEntity; Timeout startet neu | überlebt, Timeout neu | – | `CONFIRMED_CURRENT` |
| State | `_away_since`, `_ps5_on_since`, `_sticky_gaming_sub`, `_last_bio_state`, `_last_pc_active` | RAM | verloren | RAM | `CONFIRMED_CURRENT` |
| State | `_foreground` (LG-Grace 5 s), `_tv_start_since` (20 s), `_debounce_started`, `_source_debounce_fixed`, `streaming_confirmed` aus **vorherigem** publizierten `context` | RAM; `self.data` | nach Restart: TV-Kontext frühestens nach 20 s, Streaming-Bestätigung fehlt | – | `CURRENT_NEW_FINDING` (`coordinator.py:128-159, 614-669`, `logic.py:497-512`) |
| Policy | `OrchestratorState`, Nudge, Boost-Suppress, `_homepods_idle_since` | RAM | verloren — auch bei Options-Speichern (Reload, OPR-014) | RAM | `CONFIRMED_CURRENT`, ergänzt |
| Policy | `_matrix_override` | Store | überlebt | Store | `CONFIRMED_CURRENT` |
| Apply | `SleepTvState` (absolute Deadline, `off_commanded_for_deadline`, `warned_for_deadline`, Off-Bestätigung) | **Store**, vor Service-Call persistiert, Reconcile nach Start | überlebt, keine Doppelausführung derselben Deadline | „kein Store in Apply" | `OUTDATED_SNAPSHOT` (`coordinator.py:266-286, 2425-2543`; `docs/sleep-tv-evidence-contract.md`) |
| Apply | `TvWolState` (Episode, Permission) | RAM, **absichtlich** ohne Persistenz; Start = verbraucht | nach Restart/Reload keine TV-Weckung, bis Episode endet und neu beginnt | `fired`-Flag; nach Restart einmaliges Re-Fire möglich (im Snapshot korrekt, `@audited logic.py:728-780`) | `OUTDATED_SNAPSHOT` + `CURRENT_NEW_FINDING`: main ist fail-closed (`docs/issue-46-tv-wol-episode.md:27-32`) |
| Apply | `NachlaufState`, `PrivateExitState`, laufende Tasks | RAM | Flanke verloren, kein Reconcile | RAM | `CONFIRMED_CURRENT` (s. u.) |
| Apply | Playback-Episode (`_wake_start_owned`, `_playback_episode_admitted`, `_resume_content_*`, Recovery-Stage), `RadioDispatchState`, `_policy_readmission_*`, `_last_hard_recovery_at`, `ApplyState` | RAM | verloren; Recovery/Episode endet | teils | `CURRENT_NEW_FINDING` |
| Alle | `apply_enabled` | ConfigEntry-Option | überlebt; Umschalten ⇒ Reload ⇒ RAM-Verlust | Option | `CONFIRMED` + OPR-014 |

### R13/R14-Denon-Nachlauf nach Neustart — End-to-End

1. Vor Restart: PC→aus-Flanke armt `pc_armed`, Task wartet 90 s (`logic.py:1087-1094`, `coordinator.py:2588-2593`).
2. Unload bricht den Task ab (`coordinator.py:420-421`).
3. Setup: neuer `NachlaufState()` mit `last_pc_on=None`; erster `_compute()` im First-Refresh (`coordinator.py:623-625`).
4. `pc_off_edge = state.last_pc_on is True and inp.pc_power_on is False` ⇒ `False` (weil `None`); `ns.pc_armed` ist `False`, also auch kein Cancel. Folgeticks: `last_pc_on=False` → weiterhin keine Flanke.
5. Kein Startup-Reconcile existiert (vollständiger Coordinator gelesen).
6. Denon geht erst aus durch: Away-Block (`logic.py:850-856`), neue Private-Exit-Flanke, neue PC/TV-an→aus-Flanke, manuell.

**Klassifikation:** `PARTIALLY_CONFIRMED` — Mechanismus `CONFIRMED_CURRENT` (`IST_FACT`); „unbegrenzt" überzeichnet. Zusätzlich: identischer Verlust bei jedem Reload durch den Apply-Scharfschalter.

---

## 9. Trigger / Race / Timing Verification

**Timer/Delays (main, `IST_FACT`):** State: Debounce 2 s / Deckel 6 s (`const.py:436, 446`), LG-Grace 5 s, TV-Start 20 s, Away 25 s, PS5-Hold 90 s, Private-Timeout 4 h, Pre-ATV-Save 5 s. Policy: Baseline-Idle 30 s (+0,5 s Re-Arm), 09:00-Tick. Apply: R2 5 s / Deckel 8 s, Denon sofort, Reapply 30 s, Radio-Resume 10 s, Wake-Lead 1 s, Wake-Debounce 5 s, Ramp 16×1 s, Dispatch-Cooldown 15 s/Backoff ≤ 120 s, Settle 60 s, Recheck 30 s, Health 3×5 s, Unmute 2×2 s, Event-Cooldown 30 s, Backstop 1 800 s, Hard-Recovery nach 300 s/Warten 60 s/Cooldown 1 800 s, Nachlauf 90 s, Private-Exit 15 s, Sleep-TV 2 700 s/Warnung 60 s/Off-Bestätigung 600 s, Readmission-Cooldown 30 s.

| Konstrukt | Status | Evidenz |
|---|---|---|
| FLEET-245 latest-wins Pending-Plan (No-Op aktualisiert Puffer ohne Fensterneustart) | `CONFIRMED_CURRENT` | `logic.py:767-804`, `coordinator.py:754-794` |
| Anti-Starvation-Deckel (State + Apply) | `CURRENT_NEW_FINDING` (#13) | State `logic.py:215-236`; Apply `logic.py:794-801` |
| Denon-Volume am Debounce vorbei, serialisiert über `_exec_lock` | `CURRENT_NEW_FINDING` | `logic.py:738-764`, `coordinator.py:778-780, 885-899` |
| Wake-Floor vs. Radio-Burst (FLEET-42): Floor blockierend + 1 s Lead | `CONFIRMED_CURRENT` | `coordinator.py:1812-1821, 2345-2352` |
| MA-Playback-Lock-Contention durch parallele Starts (control#48 Evidenz 25.–27.08.) → Single-Flight-Owner | `CURRENT_NEW_FINDING` | GitHub `Levtos/control#48`; `coordinator.py:663-666` |
| Veraltete Tasks: R12 prüft Episode-Token vor jedem Seiteneffekt, claimt `pending` vor `await` | `CURRENT_NEW_FINDING` | `coordinator.py:2230-2266` |
| Veraltete Tasks: jeder Recovery-Sleep re-validiert Gates | `CURRENT_NEW_FINDING` | `coordinator.py:1552-1562` |
| Sleep-TV: Marker vor Service-Call persistiert (keine Doppel-Abschaltung nach Restart) | `CURRENT_NEW_FINDING` | `coordinator.py:2528-2540` |
| Stop-Latch-Reset (async Task) und Wake-Admission auf derselben Flanke: Admission ignoriert den alten Latch-Snapshot bewusst | `CURRENT_NEW_FINDING` | `coordinator.py:627-632, 1389-1390`; PR `benni_media_apply#52` |
| Startup-Radio-Restart-Guard entfernt, Schutz liegt in Policy-Baseline-Debounce | `CONFIRMED_CURRENT` | `coordinator.py:620-622` |
| Sonnet: PS5-Hold nach Restart wirkungslos | `CONFIRMED_CURRENT` (RAM-Anker) | `coordinator.py:145, 646-652` |

Keine neuen hypothetischen Races ergänzt. Einzige nicht belegte Überlappung (R20-Restore vs. R23-Ramp beim Übergang `activity sleep → waking`) ist als `INFERRED` in Abschnitt 15 geführt.

---

## 10. Layering Bypass Review

| Direktzugriff Apply → … | Klassifikation | Begründung / Evidenz |
|---|---|---|
| `quiet_mode` (State) | bewusst dokumentiert · Safety/Latenz | Quiet muss sofort durchbrechen (`logic.py:715-735`); auf main zusätzlich bewusst im Repair-Gate einer gemanagten Episode ignoriert (`coordinator.py:1479-1480`) |
| `away_gate`/`presence_state` (State) | bewusst dokumentiert · Safety („Mirror der media_policy") | `logic.py:622-642`; Folge-Aktionen (Denon aus) existieren nur in Apply |
| `media_device` (State) | bewusst dokumentiert, historisch gewachsen | `docs/issue-46-tv-wol-episode.md:7-12`; Capability-Mehrfachnutzung (Abschnitt 5) |
| `bio_state` + Attribute `sleep_source`, `sleep_reference_start` (Core State) | technisch notwendig für R24/#59, aber **unnötige Duplikation** der Schlafklassifikation mit abweichender PS-Behandlung | `coordinator.py:516-564`; OPR-032 |
| `activity_state` | nur State/Policy, nicht Apply | – |
| Denon-/TV-/PC-Master (Core Devices) | technisch notwendig (Ist-Power); **uneinheitlich** (Denon State-String, PC/TV Attribute) | `coordinator.py:467-514`; OPR-111 |
| Stop-Latch (Helper) | Safety, historisch gewachsen; überschattet Policy-`manual_stop` | OPR-063 |
| `audio_owner` roh (Policy) | bewusst (#16, generalisierte Owner-Startsperre) | `logic.py:287-318` |
| HomePods-Pods (einzelne AirPlay-Player) | technisch notwendig (#16) | `const.py:144-149, 221-226` |
| Radio-Helper (`radio_ready`, `manual_playback`, `planned_station_playing`) | historisch gewachsen (YAML-Templates) | `templates/radio.yaml`; Policy liest `planned_radio`/`manual_playback` ebenfalls |
| `wake_needed` | nur Policy; Apply nutzt Bio-Flanke → keine Duplikation, andere Semantik | Abschnitt 11/OPR-120 |
| Umbrella → `hass.data` Coordinator-Methoden (inkl. manueller Play) | bewusst (Write-Gateway), Gate-Bypass nicht dokumentiert | `aggregator.py:255-299`; OPR-012 |
| Core State → `sensor.benni_media_state_*`, `sensor.system_benni_media_state_activity_context`, Sleep-TV-Evidence | bewusst (Feed-/Evidence-Verträge), Slug-Fehlbindung (OPR-141) | `benni-core-state/…/const.py:299-315` |
| YAML-WoL-Automation ← Core State `apply_ready` | historisch gewachsen, nicht in Media-Doku als Abhängigkeit der Kette geführt | OPR-013 |

**Wake-Semantik (Auftrag K):** `binary_sensor.wake_planner_benni_wake_needed` ist ein **Level**: an, solange eine geplante/überschriebene Weckentscheidung existiert **und** jetzt im Weckfenster liegt (`ha_wake_planner/…/entities.py:185-195`). Die Apply-Wake ist eine **Transition** des tatsächlichen Core-State-Lebenszyklus. Core State verwendet `wake_needed` selbst als `planned_wake`, um S/PS → `waking` zu schalten (`benni-core-state/…/logic.py:1043-1102`). Policy nutzt das Level (Baseline-Stille im Fenster) und dessen Flanke (`manual_stop`-Reset, praktisch wirkungslos bei gesetztem Latch, OPR-063); Apply startet die Sequenz über die Bio-Flanke. Fälle „Policy erkennt Wake, Apply nicht": Weckfenster öffnet, während Bio bereits `awake` ist (keine Flanke) — dann schweigt nur die Policy-Baseline. Dokumentiert ist die Baseline-Stille (FLEET-246, `logic.py:344-350`), nicht die unterschiedliche Signalwahl. Bewertung Sonnet: `PARTIALLY_CONFIRMED` / `INTERPRETATION` — keine Gleichheit, sondern vor- und nachgelagerte Ebenen.

---

## 11. Core Contract Candidate Review

Nur Beurteilung der Sonnet-Kandidaten; keine neuen Contracts.

| Kandidat | IST sauber? | Semantik eindeutig? | Owner eindeutig? | Ebene eindeutig? | Heute formalisiert? | Reife |
|---|---|---|---|---|---|---|
| CC-01 Geräte-Power/Aktivität | nein — drei Lesarten (State `is_active`, Policy State-String, Apply State-String für Denon / Attribute für PC/TV) | nein — Headline vs. `is_active` je Geräteklasse offen (`benni_media_state#25`) | ja (Core Devices) | actual | nein | nicht reif |
| CC-02 `audio_owner` | Producer ja | **nein** (Stack-Vorrang + beobachtetes Playback) | ja | gemischt | nein | nicht reif |
| CC-03 `audio_scenario` | ja | weitgehend (Desired); Verhältnis zu `context` im Code dokumentiert | ja | desired | nein | beschreibend nutzbar; kein Execution-Consumer |
| CC-03a Overview-Glättung | – | UX | – | ux | – | Sonnet korrekt: kein Kandidat |
| CC-04 Volume-Targets | teilweise | `null` auf main nur unkonfiguriert/blockiert (Sonnets Overload war ungemergter Branch); **`0.0`** trägt mehrere Bedeutungen (still, kein Denon-Volume, blockiert Starts, Wake hält Floor) | ja | desired | nein | nicht reif |
| CC-05 `subwoofer_allowed` | ja | Name „allowed", genutzt als Soll-an | ja | desired | nein | nahezu reif |
| CC-06 `media_blocked` | – | Folgen je Ebene verschieden | – | – | nein | `ARCHITECTURE_ASSUMPTION`; Fakt `away_gate` existiert |
| CC-07 `bio_sleep` für Media | nein (PS-Divergenz) | Quelle ja: `bio_state` mit `contract_version` 2.1.0 (`LIVE_READONLY`) | ja (Core State) | actual | Quelle ja, Consumer-Mapping nein | Quelle reif, Consumer-Mapping nicht |
| CC-08 `context/subcontext/device` | `context` ja | `device` nein (Identität + Routing) | ja | actual | nein | `context` nahezu reif, `device` nicht |
| CC-09 Private Time | Kette ja | Ebenen erst durch diesen Review geklärt | ja (State) | actual/desired/execution | nein; `private_time_active` nur Attribut | nicht reif |
| CC-10 `apply_enabled` | Config | Name kollidiert | je Modul | execution-config | Option | kein Contract-Kandidat im engeren Sinn |

**Übersehene, bereits formalisierte Artefakte (IST, keine Empfehlung):** `activity_context`-Feed (in `core-contracts@main` als `media.activity.v1` dokumentiert, OPR-142); Sleep-TV-Evidence mit `contract_version` 1.0.0 (Apply, OPR-141); `binary_sensor.benni_core_state_apply_ready` als Gate in YAML (OPR-013).

---

## 12. Lastenheft Verification

Vergleichsquelle: `einhornzentrale@0640ffc` `docs/lastenhefte/reviewed/media/` — **Ordner** mit `lastenheft.md`, `00_overview.md`, `20_helpers.md`, `30_combined.md`, `40_automations.md`, `50_scripts.md`, `activity.md`, `naming.md`, `findings_splitting.md`; Sonnet las nur `lastenheft.md` (Branch-Stand). Zusätzlich ADR `docs/adr/2026-08-10-sleep-shutdown-trigger.md`.

| Punkt | Sonnet | Verifikation | Einordnung |
|---|---|---|---|
| R10 Private Time | CONTRADICTION | R10 (`lastenheft.md:209-211`) und `20_helpers.md:75` („manuell via Dashboard") ≠ Code; die Diff-Tabelle desselben Dokuments (`lastenheft.md:465`, identisch `00_overview.md:236`) nennt als Ersatz „via Stash/manuell"; der aktuelle Vertrag `benni_media_state#25` bestätigt den Auto-Pfad als geltende Regel. Ursprüngliche GitLab-Entscheidung (control#3) auf GitHub nicht auffindbar | **Spec intern widersprüchlich; R10-Text vermutlich veraltet** (`PROBABLY_OUTDATED_SPEC`); Entscheidungs-Herkunft `INSUFFICIENT_EVIDENCE` |
| R23 Wake | PARTIAL | main-Spec: „PC-Ein" entfernt (`lastenheft.md:270`, einhornzentrale PR #34); `40_automations.md:94` führt PC-Aktivität als Activity-Eingang ohne Bio-Wirkung; Kaffeemaschine/Fenster/PS5 nicht als Rohtrigger gebunden (Bio-Flanke stattdessen); **Private-Time-Aktivierung als Trigger in Spec, im Code explizit unterdrückt** (`benni_media_apply/…/logic.py:1615-1621`) | Spec teilweise nachgezogen; beide beschreiben unterschiedliche Ebenen (Rohsignal vs. fusionierte Bio-Flanke); Private-Time: echter Widerspruch |
| R24 Sleep-TV-Off | UNCLEAR | 45 min (2 700 s), Warnung 60 s vorher, Text identisch (`const.py:391-397`); main zusätzlich: PS-bewusste absolute Deadline, 10-min-Off-Bestätigung, Evidence-Sensor, Restart-sicher (core-state#59) | umgesetzt; Code umfangreicher als Spec (`EXTRA_IN_CODE`) — Sonnets UNCLEAR aufgelöst |
| R2 Debounce | PARTIAL (4 s/5 s) | State 2 s/6 s, Apply 5 s/8 s + Denon sofort | weiterhin PARTIAL mit anderen Werten; Kalibrierung laut Spec erlaubt, nicht zurückdokumentiert |
| R26–R29 Simulator | MISSING_IN_CODE | bestätigt, auch in YAML | `MISSING_IN_CODE` |
| Gaming Grind | PARTIAL (`INFERRED`) | Denon-Default −0,10 (`benni_media_policy/…/const.py:299`) liegt im Spec-Bereich −0,10…−0,15 | `MATCH` (Grenzwert), Kalibrierung nicht zurückdokumentiert |
| Sleep als Modifier vs. Owner (R25) | control#45 beschrieben | main: HomePods pausiert/Ziel 0, Denon normale Matrix mit Nacht-Baselines; zusätzlich `activity=sleep` ⇒ Quiet-Ducking; R25 verlangt „TV/Denon mit eigenen Sleep-Lautstärken" | `PARTIAL`; unterschiedliche Ebenen (Spec: Soll-Ergebnis; Code: zwei Kanäle) |
| R12 TV-Power-On | – | episodengebundene Einmal-Weckung über YAML-WoL-Automation | `PARTIAL`/`EXTRA_IN_CODE` |
| Kontextwerte §4.1 | MATCH | bestätigt | `MATCH` |

Keine Aussage, ob Code oder Spec geändert werden soll.

---

## 13. Test Coverage Verification

| Aspekt | Snapshot | main | Bewertung |
|---|---|---|---|
| Testfunktionen State / Policy / Apply | 101 / 125 / 121 | 149 / 116 / 311 | Apply-Abdeckung stark gewachsen |
| Testart Apply | nur pure Logik | zusätzlich **echter Coordinator-Runner mit Fake-HA-Services, Player-States und deterministischer Uhr** (`tests/test_resume_reconciliation.py:1-106`, `docs/issue-41…:44-46`) | Sonnets „nur pure-logic" `OUTDATED_SNAPSHOT` |
| Cross-Integration (Module verkettet) | keine | keine | `CONFIRMED_CURRENT` |
| Vertragsnahe Tests über Modulgrenze | – | Core State testet Feed-/Mapping-Verträge (`tests/test_activity_decision_contract.py`, `test_mapping_contract.py`, `test_private_time_source_contract.py`); `einhornzentrale/tests/test_sleep_trigger_contract.py` prüft Latch im Bedtime-Pfad | Teilabdeckung einzelner Schnittstellen, keine Kette |

**Live-Repros mit Regressionstest (main):** `bei_eltern` (`benni_media_state/tests/test_presence.py:151`); #41/#48 Resume/Wake-Single-Flight (`benni_media_apply/tests/test_resume_reconciliation.py`); #46 R12-Episoden (`test_tv_wol_episode.py`); #14 stale `media_device` (`test_private_exit_denon_cleanup.py`); #16 (`test_per_pod_volume.py`, `test_stale_radio_start_under_tv.py`, `test_homepods_pause_ramp.py`, `test_denon_no_zero.py`); #13 (`test_transition_latency.py`, State `test_debounce.py`); #52 Latch-Reset (`test_resume_reconciliation.py:172-249`); core-state#59 Sleep-TV (`test_logic.py:835-959`).

**Nur innerhalb eines Repos abgesichert:** alle obigen. **Ohne jede Absicherung:** PS-Behandlung von R14/Autostart-Vorgate in Apply gegenüber Policy (Policy testet nur die Mengenzugehörigkeit, `benni_media_policy/tests/test_logic.py:62-63`); Unknown-`audio_owner` → Private-Exit; Umbrella (kein `tests/`); YAML-Start-/Stop-Pfade gegen Apply-Gates; `benni_media_state#25`-Regression für Policy/Apply (nicht geprüft → `INSUFFICIENT_EVIDENCE`).

---

## 14. Sonnet Audit Corrections

**Korrigieren**
1. Versionsbasis: Audit als Snapshot `a047734/622be27(unmerged)/2daafd3/fc2d5a7/846ccd5` kennzeichnen (OPR-001/002/003).
2. „Apply einziger Service-Writer" streichen; Writer-Map ergänzen (OPR-010/011).
3. `media_stop_latch`-Writer/Reader gemäß Abschnitt 7 ersetzen (OPR-130).
4. `bio_sleep` „CRITICAL/`sleeping`" → latente Dead-Value-Divergenz; reale PS-Divergenz aufnehmen (OPR-031/032).
5. Denon-Ableitung Apply = State-String (OPR-111).
6. AUD-RULE-008 und 02-Zeile `ps5_raw`: PS5-Gaming ist geräteebenig (OPR-146).
7. R10-Diff: Tabellen-Ersatz „via Stash/manuell" korrekt wiedergeben (OPR-052).
8. CC-06 und ähnliche Formulierungen als `ARCHITECTURE_ASSUMPTION` markieren (OPR-023).

**Ergänzen**
9. Start Authority Map und Stop/Block-Map (Abschnitte 6/7).
10. Reload-Auslöser durch Config-Writes (OPR-014); manueller Play-Bypass (OPR-012); YAML-WoL- und Resume-Automation (OPR-013/092); `wake_needed`-Reset-Schatten (OPR-063); Unknown-`private_active` (OPR-051); `activity=sleep`-Quiet-Kanal (OPR-033); PS→awake-Wake (OPR-121); Frontend-Owner-Check (OPR-042); Kommentar-Widersprüche (OPR-153); Sleep-TV-Slug-Fehlbindung (OPR-141); `media.activity.v1` in core-contracts (OPR-142); GitLab-Nummern in Code-Verweisen (OPR-004); Snapshot-Bug `device` (OPR-101).
11. Lastenheft als Ordner inkl. Companion-Dokumente und ADR prüfen (OPR-003).

**Präzisieren**
12. `media_block_reason`: gleiche Klassifikation, unterschiedliche Folgeentscheidungen je Ebene (OPR-021/022).
13. Private Time als Actual → Desired → Execution (OPR-050).
14. `audio_owner` als gemischte Semantik (OPR-040); `device` als Identität + Capability (OPR-100).
15. R13/R14-Restart „unbegrenzt" → „bis andere Off-Quelle greift", plus Reload (OPR-080).
16. Wake: Level (Planner) vs. Transition (Bio) (OPR-120).
17. Überholte Snapshot-Aussagen als `OUTDATED_SNAPSHOT` markieren: Episode-ID (OPR-070), Apply ohne Store (OPR-081), Sleep-TV-Evidence „nie veröffentlicht" (OPR-082), Volume-`null`-Doppelbedeutung (OPR-149), `resume_blocked_by_sleep`/waking (OPR-034), R12-Restart-Re-Fire (OPR-157), R24 nur `bio_sleep` (OPR-158), Debounce-Werte (OPR-090), stiller Script-Fallback (OPR-150), Tests „nur pure logic" (OPR-154), „keine Core-Contracts-Referenz" (OPR-142).

**Unverändert bestätigt**
18. Kontextwerte/Prioritäten, `toggle_private` tot, Simulator fehlt, R21 fehlt, Policy-RAM-State, tote Enum-Werte, R23-Triggerquelle, fehlende Cross-Integration-Tests, `quiet_mode`-Doppellesung als bewusstes Design, Core-State-Konsum des Activity-Feeds.

---

## 15. Open Questions After Opus Review

Nur Fragen, die nach Source-, Konfigurations- und Runtime-Prüfung offen bleiben:

1. Ist in `provisional_sleep` die Nicht-Pause des R14-Denon-Nachlaufs in Apply gewollt? core-state#59 regelt Awake-Resume, nicht den Nachlauf.
2. Soll PS → `awake` (Aktivitäts-Wake) die R23-Wake-Sequenz und den Trigger-A-Radiostart auslösen? Für den Latch-Reset beschlossen (#52), für die Sequenz nicht belegt.
3. Ist `private_active = False` bei unbekanntem `audio_owner` gewollt?
4. Sind die Gate-Bypässe des manuellen UX-Radiostarts (Latch, Schlaf, Presence, Owner) gewollt oder nur der dokumentierte Shadow-Bypass?
5. Überschreibt die live gespeicherte Core-State-Konfiguration die Prefill-Bindung `sensor.benni_media_apply_sleep_tv_evidence` auf die tatsächliche Entity `sensor.system_…`? (`INSUFFICIENT_EVIDENCE`)
6. Bedeutet der `benni_media_state#25`-Scope („keine Ersatzlogik in Policy/Apply"), dass Policy/Apply dauerhaft auf dem äußeren Master-State bleiben sollen?
7. Welche Entscheidung lag dem Auto-Private-Pfad ursprünglich zugrunde (GitLab `control#3`)? GitHub-Transfer nicht auffindbar.
8. Wer hat `script.media_radio_start` zuletzt (2026-09-05) ausgelöst? Runtime-Historie nicht ausgewertet.
9. Ist „Private-Time-Aktivierung" als R23-Trigger (Spec) oder deren Unterdrückung (Code) gültig?
10. Überlagern sich R20-Quiet-Restore und R23-Wake-Ramp beim Übergang `activity sleep → waking` hörbar? (`INFERRED`, nicht verifiziert)
11. Stammen `sleeping`/`asleep` in den Policy-/State-Mengen aus dem Legacy-Template `sensor.benni_combined_context_bio_state`, und ist diese Entity noch irgendwo gebunden? (`INFERRED`)
12. Sind weitere manuelle Latch-Writer (HA-UI-Toggle, HomeKit, YAML-Dashboards) produktiv relevant? (`INSUFFICIENT_EVIDENCE`)

---

### Anhang — Arbeitsartefakte
- Snapshots: `scratchpad/opus_review/audited/*` (Sonnet-Stand) und `scratchpad/opus_review/main/*` (`origin/main`), jeweils per `git archive`.
- Einzelbefunde: `MEDIA_SYSTEM_AUDIT_OPUS_FINDINGS.csv` (64 Zeilen).
- In der CSV zusätzlich geführt, im Text oben ohne ID-Marke enthalten: OPR-002 (ungemergter Policy-Branch, Abschnitt 1.1), OPR-011 (Options- und Store-Schreiber, Abschnitt 1.2), OPR-024 (Redundanz `presence_state`, Abschnitt 5), OPR-085 (State-RAM nach Restart, Abschnitt 8), OPR-155 (`apply_enabled`-Namenskollision, Abschnitt 11), OPR-156 (Bypass-Klassifikation, Abschnitt 10), OPR-160 bis OPR-163 (Lastenheft R24, R2, Grind, Sleep, Abschnitt 12), OPR-170 bis OPR-172 (CC-01, CC-05, CC-10, Abschnitt 11).
