# STATE_CONSUMER_AUDIT.md — Cross-Domain State Consumer Audit

**Auftrag:** Abhängigkeitskarte der von Core State und Media State veröffentlichten
fachlichen Zustände, vor jeder Änderung an Namen, Werten oder Semantik.
**Stand:** 2026-09-16 · **Rolle:** Claude Opus 5 (read-only Analyse)
**Maßgeblicher IST:** ausschließlich `origin/main` bzw. der jeweilige Remote-Default,
exportiert per `git archive` in ein Scratchpad. Keine lokalen Checkouts verändert,
keine Live-Zugriffe, keine Commits/PRs/Issues/Releases.

Detailartefakte:
[`STATE_CONSUMER_EDGES.csv`](STATE_CONSUMER_EDGES.csv) ·
[`STATE_CONSUMER_MATRIX.csv`](STATE_CONSUMER_MATRIX.csv) ·
[`STATE_CATALOG.csv`](STATE_CATALOG.csv) ·
[`DECISION_QUESTIONS.csv`](DECISION_QUESTIONS.csv)

---

## 1. Geprüfte Repo-Stände (Preflight)

`git fetch --all --prune` für alle 20 Repositories; Remote-Default via
`ls-remote --symref` verifiziert, Remote-Head gegen Remote-Tracking-Ref geprüft
(identisch für alle).

| Repository | Remote Default | analysierter Commit | Datum | Version (manifest) |
|---|---|---|---|---|
| benni-core-state | main | `c9408fa` | 2026-09-02 | 0.11.8 |
| benni_media_state | main | `c371b32` | 2026-09-07 | 0.14.5 |
| benni_media_policy | main | `3392cce` | 2026-08-30 | 0.18.2 |
| benni_media_apply | main | `639ba1b` | 2026-09-13 | 0.19.15 |
| benni_media | main | `6b4b5db` | 2026-09-09 | 0.7.8 |
| benni-core-devices | main | `52c5e89` | 2026-08-28 | 0.5.4 |
| core-contracts | main | `5160aef` | 2026-09-14 | 0.2.3 |
| blind_control | main | `47065ba` | 2026-09-10 | 0.7.4 |
| benni_blind_policy | main | `9cbd1f0` | 2026-08-31 | 0.8.6 |
| benni_light_policy | main | `c7004ba` | 2026-08-30 | 0.3.7 |
| benni_scene_presets | main | `ab89944` | 2026-09-14 | 0.1.4 |
| benni_climate_policy | main | `0ad085b` | 2026-08-13 | 0.1.9 |
| benni_door_policy | main | `31dc06d` | 2026-08-13 | 0.2.6 |
| plug_policy_engine | main | `7d61ba6` | 2026-08-13 | 0.3.4 |
| benni_notification_router | main | `f880185` | 2026-08-13 | 0.1.0 |
| ha_wake_planner | main | `9f94d2a` | 2026-08-13 | 1.1.3 (domain `wake_planner`) |
| einhornzentrale | main | `0640ffc` | 2026-09-13 | (HA-Konfiguration) |
| haos_eltern | main | `66be668` | 2026-09-02 | (HA-Konfiguration) |
| benni_media_context | main | `9919ffa` | 2026-08-13 | 0.1.1 (Legacy-Vorgänger) |
| control | main | `c96d3d9` | 2026-09-10 | (Governance) |

**Preflight-Befund, der die Methode rechtfertigt:** Der lokale Checkout von
`einhornzentrale` stand zum Analysezeitpunkt auf dem Feature-Branch
`codex/control-28-media-priority-spec` (1 ahead / 10 behind `origin/main`, zwei
ungetrackte bzw. geänderte Dateien unter `docs/`). Ein Audit gegen die sichtbaren
Arbeitsverzeichnis-Dateien hätte hier einen falschen IST abgebildet. Alle Aussagen
dieses Dokuments stammen aus `git archive origin/main`; die Checkouts wurden nicht
verändert (alle übrigen 11 geprüften Repositories: `main`, 0 geänderte Dateien).

**Einschränkungen, die für alle Aussagen gelten**

- `benni_scene_presets` und `benni_blind_policy` enthalten auf main **keine**
  Referenz auf Core-State- oder Media-State-Zustände (`benni_blind_policy` nur
  als Prefill-Konstanten; es ist laut Betriebsstand durch `blind_control`
  abgelöst). Sie erscheinen deshalb nicht als Consumer.
- `haos_eltern` konsumiert **nichts** aus diesem Stack (eigene
  `presence_state_*`-Templates, gleicher Name, andere Herkunft) → Spalte
  `eltern_haos` in der Matrix durchgehend `n/a`.
- Die Prefills in den Config-Flows sind **Code-Evidenz, nicht Live-Evidenz**.
  Wo eine Bindung live abweichen kann (gespeicherte `entry.data`/`options`), ist
  das in der jeweiligen Zeile vermerkt. **Nicht live verifiziert** — siehe §9.
- `einhornzentrale/benni_core_devices/import.yaml` ist ein Config-Artefakt. Der
  Bulk-Import ist nach vorliegendem Betriebswissen nicht live gemergt; die
  daraus abgeleiteten Kanten sind als solche markiert.

---

## 2. Zahlen

| Kennzahl | Wert |
|---|---|
| Inventarisierte fachliche Zustände (Katalog) | **38** |
| Consumer-Kanten (Edge List) | **139** |
| Cross-Domain-Zustände (mind. 2 Domänen) | **21** von 38 |
| Zustände mit `SEMANTIC_CONFLICT` | **8** |
| Offene Decision Questions | **18** (alle `benni_decision = OPEN`) |
| Consumer-Repositories mit mindestens einer Kante | **12** |

Verteilung der Decision Levels: L0 25 · L1 20 · L2 12 · **L3 30** · **L4 42** · L5 2 · L6 8.
Der Schwerpunkt auf L4 ist kein Zufall: Der Stack besteht überwiegend aus
**Gates** (darf etwas passieren?), nicht aus Zielberechnungen.

Verteilung `change_impact`: CRITICAL 29 · HIGH 53 · MEDIUM 34 · LOW 23.

Kanten je Consumer: media_policy 18 · core_state 18 (intern) · light_policy 17 ·
blind_control 15 · media_apply 14 · climate_policy 14 · einhornzentrale 13 ·
media_state 10 · notification_router 7 · door_policy 5 · plug_policy 5 ·
benni_media 2.

---

## 3. Die wichtigsten Consumer-Drifts

### 3.1 `bio_state` — fünf Sleep-Definitionen (DQ-01)

| Consumer | Sleep-Menge | Fundstelle |
|---|---|---|
| media_policy | `{provisional_sleep, sleep, sleeping, asleep}` | `const.py:86` |
| media_apply `_bio_sleep` | `{sleep}` | `coordinator.py:517` |
| media_apply `BIO_SLEEP_CONTEXT_VALUES` | `{provisional_sleep, sleep}` | `const.py:109` |
| media_state | `{sleep, asleep}` | `const.py:502` |
| light_policy | `{provisional_sleep, sleep}` | `const.py:102` |
| blind_control | `{provisional_sleep, sleep}` | `engine.py:46` |
| notification_router | `{sleep}` | `const.py:36` |
| plug_policy_engine | `{sleep}` | `engine.py:64` |
| **climate_policy** | `{sleep, waking}` | `policy.py:906` |
| toolbox_readiness.yaml | kennt `provisional_sleep` gar nicht | `toolbox_readiness.yaml:11` |

Praktische Folge in einer normalen Nacht (`bio_state = provisional_sleep`): Licht
ist aus, Rollladen ist zu, die HomePods sind pausiert — **gleichzeitig** darf
media_apply Radio automatisch starten, der Denon-Nachlauf läuft ungebremst weiter,
die Steckdosen werden nicht geschnitten, Benachrichtigungen klingeln und leuchten,
die Heizung heizt normal, und `binary_sensor.system_benni_context_ready` meldet
„nicht bereit". Innerhalb von media_apply existieren beide Lesarten in derselben
Datei.

`climate_policy` ist der einzige Consumer, der **`waking` als Schlaf** wertet
(Heizprofil `off`), und der einzige mit einem **Fail-safe-Default `"sleep"`**:
fehlt der Kontext, geht die Heizung aus (E-200).

### 3.2 `day_state` — zwei Enum-Generationen nebeneinander (DQ-10)

Core State publiziert neun Phasen. `media_policy` und `climate_policy` rechnen
mit der alten Acht-Phasen-Generation:

- **`midday`, `late_afternoon`, `evening`** haben in `HOMEPODS_BASELINES` /
  `DENON_BASELINES` **keinen Eintrag** → die Phasen-Baseline fällt still auf den
  flachen Fallback (0.35 / 0.40). Dieselben drei Werte sind **nicht** in
  `SUB_ALLOWED_PHASES` → der Subwoofer bleibt mittags, am späten Nachmittag und
  am Abend gesperrt (E-054, E-055).
- **`late_morning`, `early_evening`** werden von media_policy und climate_policy
  erwartet, von core_state aber nie erzeugt, und von `blind_control` aktiv mit
  `ValueError` abgelehnt. Das Frei-Tag-Bad-Preheat-Fenster (`late_morning`) kann
  damit nie feuern (E-056).
- Nur `blind_control` (strikt 9) und `light_policy` (9+8 vereinigt) sind
  konsistent mit dem Producer.

### 3.3 `presence` — zwei parallele Away-Ketten (DQ-08)

Core State publiziert `binary_sensor.*_core_state_presence_away` und bezeichnet
es im Code als „canonical away gate for the whole fleet" (inkl. Activity-Hold).
**Auf `origin/main` hat dieser Gate keinen einzigen Produktiv-Consumer** (E-076).

Stattdessen leitet `media_state` Away **selbst** aus `presence_personal` ab
(`away_from_presence` + 25-s-ON-Debounce, E-070/E-071) und beliefert damit
media_policy, media_apply — und sehr wahrscheinlich auch `blind_control`, dessen
Auto-Suggestion Entities mit einem `away_gate`-**Attribut** bevorzugt, das nur
media_state publiziert (E-168). Der Activity-Hold von Core State (E-043/E-044)
wirkt in dieser Kette also **nicht**.

Der Code-Kommentar in `benni_media_state/logic.py` nennt als Quelle
`binary_sensor.benni_core_state_away` — das ist falsch; der Code liest
`presence_personal`.

### 3.4 `bei_eltern` — anwesend oder abwesend? (DQ-09)

`light_policy` behandelt `bei_eltern` als **abwesend** (hard_off /
Anwesenheitssimulation). media_state, door_policy, plug_policy_engine und
notification_router behandeln es als **home-äquivalent**. door_policy ist der
feinste Fall: home-äquivalent fürs Auto-Lock, aber ausdrücklich „nicht zuhause"
fürs Auto-Unlock. Beide Lesarten sind begründbar — es fehlt die gemeinsame Aussage.

### 3.5 `media_device` — drei Bedeutungen in einem Enum (DQ-14)

Derselbe Wert trägt gleichzeitig **Identität** (light_policy: „ist meine
konfigurierte Gaming-Quelle aktiv?"), **Audio-Routing** (media_apply:
`DENON_CONSUMER_DEVICES`, media_policy: `is_pc_gaming`) und **Screen-Intent**
(media_apply: `SCREEN_DEVICES` → TV einschalten). Konkrete Lücke daraus:
`appletv` ist für Apply ein Screen-Device, in `blind_control`s `tv_platforms`
aber nicht enthalten — ein Apple-TV-Abend liefert keine explizite Blend-Klasse
(E-140).

### 3.6 Harte String-Literale statt Konstanten

Drei Stellen vergleichen `bio_state` gegen inline-Literale statt gegen die
Modulkonstante — sie würden ein Rename still überleben und danach falsch
entscheiden:

- `benni_media_policy/logic.py:350` → `== "waking"` (Wake-Fenster, E-003)
- `benni_media_apply/logic.py:487` → `not in ("waking","awake")` (E-006)
- `benni_climate_policy/policy.py:906` → `in ("sleep","waking")` (E-200)

---

## 4. Duplicate Truth vs. legitime Projektion

Klassifikation nach §11 des Auftrags. Nicht jede Mehrfachdarstellung ist ein Fehler.

| Klasse | Beispiele |
|---|---|
| `SAME_TRUTH_PROJECTION` (legitim) | `raw_presence` als Attribut von `presence_effective` (door_policy liest Beobachtung und Stabilisierung aus einer Entity, E-084) · `entertainment_active` als Projektion von `media_context` · `master_context` als Verkettung · `media_block_reason()` in policy und apply: textgleiche Lesung, **unterschiedliche Folgen je Ebene** (E-165/E-167) |
| `COMPATIBILITY_ALIAS` | `sleeping`/`asleep` in media_policy und media_state · `free_time` im Activity-Feed · `movie`/`video` in plug_policy · DE/EN-Synonyme für `day_context` in blind_control · `LEGACY_DAY_PHASES` in light_policy |
| `DERIVED_DIFFERENT_SCOPE` (legitim) | `activity_context` (Media-Hälfte) vs. `activity_state` (Gesamtentscheidung) · `quiet_mode` als Volume-Overlay (policy) vs. Exec-Modus (apply) · `sleep_tv_evidence` als Execution-Evidenz, die einen State-Übergang autorisiert |
| `DUPLICATE_CALCULATION` | `import.yaml: context_presence_preheat_active` berechnet neu, was core_state bereits publiziert (E-096) · zwei Bias-Light-Wahrheiten: `plug_policy_engine` und `import.yaml: media_bias_light_should_be_on` (E-126/E-160) · `evaluate_quiet` erkennt Schlaf über `activity_state` statt über `bio_state` (E-039) |
| `SEMANTIC_CONFLICT` | `bio_state` (5 Sleep-Mengen) · `day_state` (2 Enum-Generationen) · `presence_personal`/`bei_eltern` · `presence_away` vs. `media_state.away_gate` · `media_device` (3 Bedeutungen) · `audio_owner` (Soll-Priorität + Ist-Beobachtung) |
| `LEGACY_SECOND_TRUTH` | Alle `sensor.benni_combined_context_*`-Bindungen in climate_policy (**vollständig**), plug_policy (presence/bio/activity) und media_policy (day_state) — `core_state/mapping.py` führt diese IDs ausdrücklich als `legacy_references` mit `legacy_resolution = replace_after_cutover`; `light_policy` hat den Cutover bereits vollzogen · zwei Wake-Planner publizieren dieselbe Entscheidung |
| `UNCLEAR` | blind_controls Away-Bindung (Auto-Suggestion bevorzugt das nur bei media_state vorhandene `away_gate`-Attribut, E-168) |

Die Private-Time-Kette (Actual `media_context=private_time` → Desired
`audio_owner=private_stack` → Execution `private_active`) ist **kein** Duplicate
Truth, sondern eine korrekte Ebenentrennung.

---

## 5. Rename- / Enum-Migrations-Impact

Beispiel `bio_state.sleep → sleeping` (nur Impact, keine Änderung):

| Betroffen | Umfang |
|---|---|
| Producer | `benni-core-state/const.py:162` (`BIO_SLEEP`), `models.py` (Default `PersistentState.bio_state = BIO_SLEEP`), `logic.py:1911/1928/2013` (live_status-Label, Farb-/Prioritätsmaps), `mapping.py` `allowed_states` |
| Consumer-Konstanten | media_policy `const.py:86` (toleriert bereits) · media_apply `const.py:108`+`109` · media_state `const.py:502` · light_policy `const.py:100` · blind_control `engine.py:46` + `binding_suggestions.py:248` (options-Prüfung!) · notification_router `const.py:36` · plug_policy `const.py:72` |
| Harte Literale | media_apply `logic.py:487`, climate_policy `policy.py:906`, media_policy `logic.py:350` (waking) |
| YAML/Templates | `toolbox_readiness.yaml:11,28` (Wertebereich) · `radio.yaml:126-131` (`['sleep','sleeping']`) · `import.yaml:385` (`${bio} == "sleep"`) |
| Gespeicherte Zustände | `PersistentState.bio_state` in `.storage/benni_core_state_state_<entry_id>` — ein Restore nach dem Rename lädt den alten Wert; `models.from_dict` validiert **nicht** gegen `BIO_STATES` |
| Enum-Metadaten | `SensorDeviceClass.ENUM` + `_attr_options` an `sensor.*_bio_state`: HA protokolliert Werte außerhalb `options` als Fehler; `blind_control._is_bio_state` verlangt genau `{sleep, provisional_sleep, waking, awake}` im `options`-Attribut → **blind_control verliert bei einem Rename seine Auto-Bindung** |
| Contracts | keiner — es existiert kein publizierter Core Contract für `bio_state` |
| Compatibility-Pfade | media_policy und `radio.yaml` würden `sleeping` bereits akzeptieren; alle übrigen nicht |

Kurz: Ein `bio_state`-Rename ist **nicht** rein mechanisch. Der teuerste Teil
sind die drei Literale, das `options`-Attribut (blind_control) und der
persistierte Restore-Wert.

Weitere Rename-Risiken in absteigender Schärfe:
1. `day_state` — jede Änderung trifft zwei Enum-Generationen gleichzeitig (§3.2).
2. `activity_state` — `blind_control._convert_activity` wirft bei einem
   **unbekannten** Wert `ValueError`; die Observation wird unbrauchbar und der
   gesamte Blendschutz fällt aus (fail-closed, aber wirksam).
3. `presence_personal` — `toolbox_readiness.yaml` prüft den Wertebereich hart.
4. `media_device` — drei Bedeutungen, drei Wertelisten (§3.5).
5. `action` (media_policy → media_apply) — das Enum ist in `apply/const.py`
   **kopiert**, nicht importiert.

---

## 6. Core Contracts — Klassifikation je Zustand (§14)

**IST auf `core-contracts@main` (v0.2.3):** Es gibt fünf Schemata
(`room_climate`, `opening`, `weather_environment`, `technical_device`,
`presence` mit genau einem Feld `present: boolean`). Publizierbar ist
ausschließlich **ein** Pilot-Contract: `benni.opening.kitchen_patio_door`.
`media.activity.v1` existiert **nur als Dokumentationsbeispiel** in
`docs/consumer-api-v1.md` — kein Code, kein Registry-Eintrag.
Core Contracts ist damit heute **keine** State Machine und auch kein
State-Transport für diesen Stack.

| Klassifikation | Zustände |
|---|---|
| `CONTRACT_READY` (12) | `activity_state`, `day_state`, `day_context`, `presence_effective`, `presence_away`, `presence_household`, `presence_transition`, `apply_ready`, `media_context`, `activity_context`, `entertainment_active`, `sleep_tv_evidence` |
| `CONTRACT_AFTER_SEMANTIC_DECISION` (10) | `bio_state`, `presence_personal`, `media_device`, `quiet_mode`, `presence_state`, `away_gate`, `wake_state`, `next_wake`, `wake_needed`, `holiday_active` |
| `INTERNAL_STATE` (7) | `presence_effective_transition`, `presence_band`, `presence_preheat_active`, `media_subcontext`, `gaming_source`, `gaming_platform`, `headset_active` |
| `POLICY_OUTPUT` (5) | `audio_owner`, `audio_scenario`, `action`, die Apply-Freigaben, die Volume-Ziele |
| `EXECUTION_STATE` (1) | `playback_health` / `playback_recovery_stage` / `denon_nachlauf_active` / … |
| `NOT_SUITABLE` (3) | `master_context`, `live_status`, `private_time_manual` |

**Wo ein Contract heute den größten realen Nutzen hätte:** `activity_context`.
Der Feed ist die einzige Producer→Producer-Kante des Stacks, und der Producer
publiziert **weder Quality noch Freshness**. Core State behilft sich mit einem
consumer-seitigen 1800-Sekunden-Alters-Gate auf `last_updated` (E-146) — ein
eingefrorener Feed-Wert gilt damit bis zu 30 Minuten als frisch. Genau diese
Lücke ist das, was ein Contract mit Quality/Freshness/Lineage schließen würde.

Das beste bestehende Vorbild ist bereits im Stack: `activity_decision` an
`sensor.*_core_state_activity_state` liefert Gewinner, gültige und unterdrückte
Kandidaten, Quellen, Freshness je Quelle, `quality_status`, `degraded_reason`,
`fallback_reason` und Zeitstempel — ohne jeden Contract-Layer.

---

## 7. Architekturfrage Core State ↔ Media State (§13)

Bewertung **je Kriterium**, nicht nach Anzahl der Integrationen:

| Kriterium | Befund |
|---|---|
| Heutiger Owner sinnvoll? | **Ja** für Medien-Szenarien (media_state) und für Person/Zeit (core_state). **Nein** für `presence_state` und `away_gate`: Away ist keine Medienwahrheit. |
| Cross-domain? | 21 von 38 Zuständen. Die Media-Outputs mit den meisten fremden Consumern sind `media_context` (8) und `media_device` (6). |
| Wird er von beiden Engines interpretiert? | Nur bei **einem** Zustand: `activity_context` (Media produziert, Core arbitriert). Genau **eine** Feed-Kante. |
| Rückkopplung? | **Zwei.** (a) `bio_state` → media_apply → `sleep_tv_evidence` → `bio_state` (PS→S, über `sleep_reference_start` sauber korreliert). (b) `presence_personal` → media_state `away_gate` → `activity_context` → core `activity_state` → Activity-Hold → `presence_effective`. Gedämpft, weil `presence_personal` selbst roh bleibt. |
| Duplicate Calculation? | Ja, aber **nicht zwischen den Engines**: media_state erkennt Schlaf ein zweites Mal über `activity_state == sleep` (E-039), und die YAML-Config berechnet Preheat und Bias Light doppelt. |
| Owner technisch verschiebbar? | Ja für `presence_state`/`away_gate` (media_state → core_state) und für `wake_*` (wake_planner → core_state; in `mapping.py` bereits als Ziel erfasst). |
| Echter Nutzen einer Fusion? | **Nicht belegt.** Die Schnittstelle ist schmal, gerichtet und bereits qualitätsgegatet. |

**Bewertung:** Eine Core-State/Media-State-Fusion wird durch die Daten **nicht
gestützt** — nicht „noch nicht entscheidbar", sondern aktiv nicht nahegelegt. Die
teuren Probleme dieses Stacks sind Semantik (DQ-01, DQ-08, DQ-09, DQ-10, DQ-14)
und Bindungshygiene (DQ-16, DQ-17), nicht Modulschnitt. Eine Fusion würde keines
davon lösen und die einzige saubere Grenze (Feed mit Qualitäts-Gate) aufgeben.

Der einzige gut belegte Owner-Verschiebungskandidat ist die Away-/Presence-
Projektion von media_state zurück zu core_state (DQ-08).

---

## 8. Decision Questions für Benni

Alle 18 in [`DECISION_QUESTIONS.csv`](DECISION_QUESTIONS.csv), `benni_decision = OPEN`.
Die sechs, die andere Entscheidungen blockieren:

1. **DQ-01 — Was bedeutet `provisional_sleep` je Schicht?** Blockiert DQ-02,
   DQ-03, DQ-04 und jede Vereinheitlichung im Media-Stack.
2. **DQ-08 — Ist Away Beobachtung, Klassifikation oder Execution-Gate?**
   Entscheidet, ob `binary_sensor.*_core_state_presence_away` Consumer bekommt
   oder seinen Kanonik-Anspruch verliert.
3. **DQ-09 — Gilt `bei_eltern` als anwesend?** Kleine Frage, vier betroffene Domänen.
4. **DQ-10 — Welche Tagesphasen-Generation gilt?** Muss **vor** DQ-17 entschieden
   sein, sonst bricht beim Cutover die Volume-Baseline in drei Phasen.
5. **DQ-14 — Was bedeutet `media_device`?** Voraussetzung für einen sauberen
   Screen-Class-Contract (DQ-05).
6. **DQ-17 — Bleiben die `benni_combined_context_*`-Bindungen?** Betrifft die
   komplette Klimasteuerung, Teile der Steckdosenlogik und die Media-Lautstärke.

Nicht-blockierend, aber billig zu schließen: DQ-13 (toter Schlafkanal in
`evaluate_quiet`), DQ-16 (clean vs. `system_`-Slug).

---

## 9. Was dieser Audit **nicht** belegt

- **Keine Live-Verifikation.** Alle Bindungsaussagen stammen aus
  `PROFILE_PREFILL`/`ENTITY_PREFILL` und `LEGACY_ENTITY_MAP` im Code. Gespeicherte
  `entry.data`/`options` können abweichen. Besonders zu prüfen:
  blind_controls `away`- und `activity_state`-Bindung (E-035, E-168),
  core_states `sleep_tv_evidence`-Bindung (E-026), climate_policys und
  plug_policys Combined-Bindungen (E-200 ff., E-022).
- **Keine Aussage über Live-Verhalten** von `import.yaml` (Bulk-Import-Stand).
- `benni_media_context` (Legacy-Vorgänger) wurde als historische Quelle gelesen,
  nicht als aktiver Consumer gewertet.

---

## 10. Abgleich mit den vorhandenen Audits (§16)

`MEDIA_SYSTEM_AUDIT_OPUS_REVIEW.md` / `-FINDINGS.csv` (2026-09-16) diente als
Seed. Abgleich der für diesen Audit relevanten Befunde:

| Alter Befund | Status hier |
|---|---|
| OPR-032 `provisional_sleep` uneinheitlich in Media | **CONFIRMED_CURRENT** und erweitert: nicht nur Media — insgesamt **fünf** Sleep-Definitionen inkl. climate_policy (`{sleep, waking}`) und toolbox_readiness (PS ungültig). |
| OPR-033 `evaluate_quiet` liest `activity_state ∈ {sleep,asleep,quiet}` | **CONFIRMED_CURRENT** (E-039). Ergänzung: der Zweig feuert nur ohne TV, weil `activity` bei laufendem TV auf `entertainment` kippt (E-046). |
| OPR-020/021 `media_block_reason()` textgleich in policy und apply | **CONFIRMED_CURRENT**, Einordnung bestätigt: `SAME_TRUTH_PROJECTION`, nicht Duplicate Truth (E-165/E-167). |
| OPR-100 `media_device` als Routing-Information | **CONFIRMED_CURRENT** und erweitert: **drei** Bedeutungen (Identität, Routing, Screen-Intent) plus die `appletv`-Lücke bei blind_control. |
| OPR-141 `sleep_tv_evidence` Slug-Mismatch | **CONFIRMED_CURRENT** als Code-Befund (E-026); Live-Auflösung weiterhin offen. |
| OPR-142 `media.activity.v1` in core-contracts geplant | **CONFIRMED_CURRENT**: nur Dokumentation, kein Code, kein Registry-Eintrag. |
| OPR-063 `manual_stop`-Reset hält nur einen Tick | **CONFIRMED_CURRENT** (E-116). |
| OPR-051 `private_active` verliert `unknown` | **CONFIRMED_CURRENT** (E-176). |
| OPR-031 „`sleeping` erzeugt keine Live-Divergenz, Wert wird nie emittiert" | **CONFIRMED_CURRENT** — und die vermutete Herkunft ist jetzt belegt: `einhornzentrale/packages/media/templates/radio.yaml:126-131` führt `bio in ['sleep','sleeping']`; es ist die einzige Fundstelle im Stack, die beide Werte zusammen kennt (E-205). |
| Memory-Notiz „light_policy `make_gaming_policy` feuert nie, Gate verlangt `free_time`/`idle`" | **OUTDATED** — auf `origin/main` enthält `GAMING_ACTIVITY_STATES` den Wert `gaming` (`const.py:125-127`). Befund behoben. |

**`NEW_FINDING` (in keinem vorliegenden Audit enthalten):**

1. `binary_sensor.*_core_state_presence_away` ist als kanonisches Flotten-Gate
   deklariert und hat auf main **keinen Consumer** (E-076, DQ-08).
2. media_state leitet Away aus `presence_personal` ab statt aus diesem Gate und
   umgeht damit den Activity-Hold; der Code-Kommentar nennt eine falsche Quelle (E-070).
3. `day_state`: `midday`/`late_afternoon`/`evening` haben in media_policy keine
   Baseline und kein Subwoofer-Fenster; `late_morning`/`early_evening` werden
   erwartet, aber nie erzeugt (E-054, E-055, E-056).
4. climate_policy bindet **ausschließlich** auf `benni_combined_context_*`, hat
   eine eigene Sleep-Menge `{sleep, waking}` und einen Fail-safe-Default
   `"sleep"` → Heizung aus bei fehlendem Kontext (E-200 ff.).
5. `bei_eltern` wird in Licht und allen übrigen Domänen gegensätzlich gewertet (E-072).
6. Als „nur Debug" deklarierte Attribute am `activity_state`-Sensor
   (`media_context`, `media_device`, `entertainment_active`) sind faktischer
   Produktiv-Input von blind_control (E-128, E-140, E-159).
7. Der `activity_context`-Feed publiziert kein Quality/Freshness-Feld; das
   30-Minuten-Alters-Gate lebt beim Consumer (E-146).
8. `hold_strength` (hard/soft) des Feeds wird von Core State ignoriert — es gibt
   zwei parallele Hold-Modelle (E-147).
9. Doppelberechnungen in der YAML-Config: `context_presence_preheat_active` und
   `media_bias_light_should_be_on` (E-096, E-160).
10. `toolbox_readiness.yaml` meldet im legitimen Zustand `provisional_sleep`
    „nicht bereit" (E-025).
11. `appletv` ist für media_apply ein Screen-Device, für blind_control keine
    tv-Plattform (E-140).

**`INCORRECT`:** keine Aussage der vorliegenden Audits wurde im Rahmen dieses
Scopes als sachlich falsch widerlegt.

---

## 11. Definition of Done — Gegenprobe an zwei Beispielen

**`bio_state`:** Producer core_state (§1) · vier Werte · Consumer je Wert in
`STATE_CONSUMER_MATRIX.csv` (vier Zeilen) · Decision Levels L0–L6 in 27 Kanten ·
fachliche Frage je Kante · Folge je Kante · abweichende Interpretationen in §3.1 ·
Rename-Impact in §5 · Rückkopplung über `sleep_tv_evidence` in §7 ·
Contract-Klassifikation `CONTRACT_AFTER_SEMANTIC_DECISION` in §6.

**`media_device`:** Producer media_state · acht Werte · Consumer je Wertgruppe in
fünf Matrix-Zeilen · sechs Kanten über L1/L2/L4/L5 · drei konkurrierende
Bedeutungen in §3.5 und DQ-14 · Rename-Impact `CRITICAL` · Contract-Klassifikation
`CONTRACT_AFTER_SEMANTIC_DECISION`.

---

*Erstellt read-only gegen `origin/main`. Keine Codeänderung, kein Commit, kein
Branch, kein PR, kein Issue, kein Release, kein Entity-Rename, keine
Contract-Migration, keine Config-Änderung, kein HA-Service-Call.*
