# STATE_DECISION_REVIEW.md — Fachliche Entscheidungsschicht

**Gegenstand:** nicht der Sensor und nicht der Enum-Wert, sondern die fachliche
**Decision**. Verdichtung der 139 Consumer-Kanten und 18 Rohfragen aus dem
Cross-Domain State Consumer Audit (2026-09-16, `origin/main`) zu einer für
Menschen entscheidbaren Ebene.

**Stand:** 2026-09-16 · **Rolle:** `CURRENT HUMAN DECISION BASELINE`
**Status aller Empfehlungen:** Vorschlag. Verbindlich wird eine Regel erst durch
eine ausdrückliche Entscheidung von Benni. Die Spalte `Benni` steht deshalb
überall auf `OPEN`.

**Grundlage:** [`STATE_CONSUMER_EDGES.csv`](STATE_CONSUMER_EDGES.csv) ·
[`STATE_CONSUMER_MATRIX.csv`](STATE_CONSUMER_MATRIX.csv) ·
[`STATE_CATALOG.csv`](STATE_CATALOG.csv) ·
[`DECISION_QUESTIONS.csv`](DECISION_QUESTIONS.csv) ·
[`STATE_CONSUMER_AUDIT.md`](STATE_CONSUMER_AUDIT.md)
Maschinenlesbare Fassung: [`STATE_DECISION_REVIEW.csv`](STATE_DECISION_REVIEW.csv)

Es wurde **kein neuer Audit** durchgeführt. Alle Aussagen stammen aus den
genannten Artefakten; es wurde kein Produktivcode gelesen, der dort nicht schon
belegt ist.

---

## Lesart: Decision statt Sensor

Jede Zeile beschreibt einen Knoten in der Kette

```
Input State/Value  →  Decision Node  →  Interpretation  →  Output  →  nächster Consumer
```

Entscheidungen auf verschiedenen Ebenen werden **nicht** zusammengezogen. Dass
`provisional_sleep` für das Licht Schlaf ist und für den Sleep-TV-Timer der
Beginn eines Lifecycles, ist kein Widerspruch, sondern L3 gegen L6.

Die Leitfrage lautet deshalb nirgends „ist `provisional_sleep` gleich `sleep`?“,
sondern immer „**für welche konkrete Decision** soll `provisional_sleep` wie
`sleep` behandelt werden?“ Die Familie `BIO / SLEEP` ist genau deshalb in
19 Einzelentscheidungen aufgeteilt.

**Decision Levels:** `L0 FACT` · `L1 NORMALIZE` · `L2 CLASSIFY` · `L3 POLICY` ·
`L4 ARBITRATE/GATE` · `L5 EXECUTE` · `L6 LIFECYCLE/RECOVERY`

---

## Kennzahlen

| | |
|---|---|
| Decisions gesamt | **54** |
| Semantic Conflict | JA 32 · teilweise 9 · NEIN 11 · unklar/Folgefrage 2 |
| Familien | 6 |
| Priorität | P1 12 · P2 22 · P3 20 |
| Klassifikation | ARCHITECTURE_DECISION 13 · BUG 9 · DRIFT 8 · LEGACY 5 · NO_PROBLEM 9 · PRODUCT_DECISION 10 |
| Migrationstyp | CONTRACT_CUTOVER 4 · CROSS_DOMAIN_HOTFIX 2 · DOCUMENT_ONLY 12 · ENUM_MIGRATION 1 · LOCAL_FIX 22 · NO_CHANGE 7 · SEMANTIC_SPLIT 6 |
| Decision Level | L0 10 · L1 5 · L2 10 · L3 11 · L4 12 · L5 1 · L6 5 |

Familien: BIO / SLEEP 19 · PRESENCE / AWAY 8 · DAY / ACTIVITY 8 · MEDIA CONTEXT / DEVICE / ACTIVITY 10 · SHARED / DEBUG OUTPUTS 5 · WAKE / TRANSITIONS 4

---

## Haupttabelle

| Decision ID | Family | Level | Fachliche Frage | Inputs | heutige Interpretationen | Konflikt | Empfohlene kanonische Regel | Konfidenz | Benni |
|---|---|---|---|---|---|---|---|---|---|
| `D-B01` | BIO / SLEEP | L2 | Soll Core State einen publizierten Sleep-Kontext-BEGRIFF anbieten, auf den Consumer binden, statt dass jeder Consumer seine eigene Wertemenge pflegt? | bio_state | **(heute: keiner)**: es gibt keinen publizierten Praedikat-Begriff; jeder Consumer definiert eine eigene Wertemenge; **benni-core-state (Producer)**: publiziert bereits die Attribute sleep_confirmed, sleep_source, sleep_reference_start | JA - Ursache aller nachfolgenden BIO-Decisions | ein zusaetzliches boolesches Attribut sleep_context am bestehenden bio_state-Sensor (true fuer provisional_sleep UND sleep), additiv, ohne Enum-Aenderung. Consumer, die 'Schlaf' meinen, binden auf das Attribut; Consumer, die bewusst nur bestaetigten Schlaf … | hoch | `OPEN` |
| `D-B02` | BIO / SLEEP | L4 | Darf waehrend provisional_sleep automatisch Musik/Radio gestartet werden? | bio_state, radio_ready, presence_state, manual_playback | **benni_media_policy**: PS zaehlt als Schlaf (BIO_SLEEP_VALUES); **benni_media_apply**: _bio_sleep prueft nur == sleep | JA - die beiden Ebenen derselben Kette widersprechen sich | waehrend PS KEIN automatischer Start. PS ist ein Schutzkorridor; ein selbst gestarteter Radiostream ist genau das, was der Korridor verhindern soll. Konkret: media_apply auf das Sleep-Praedikat (D-B01) ziehen. | hoch | `OPEN` |
| `D-B03` | BIO / SLEEP | L4 | Darf waehrend provisional_sleep ein Resume bzw. ein Playback-Repair laufen? | bio_state, homepods_resume_allowed, volume_apply_allowed, quiet_mode | **benni_media_policy**: PS zaehlt als Schlaf; **benni_media_apply**: BIO_SLEEP_CONTEXT_VALUES enthaelt PS; zusaetzlich fail-closed auf waking/awake | NEIN - beide Ebenen blocken, Apply zusaetzlich strenger | unveraendert lassen. Das fail-closed-Muster von Apply (positiver Wachbeweis statt Abwesenheit von Schlaf) ist die sicherere Variante und sollte als Muster fuer alle Aktuations-Gates dokumentiert werden. | hoch | `OPEN` |
| `D-B04` | BIO / SLEEP | L3 | Sollen die HomePods bei provisional_sleep pausiert werden? | bio_state, media_context, audio_owner | **benni_media_policy**: PS zaehlt als Schlaf -> competes=True; **benni_media_apply**: fuehrt die Policy-Entscheidung aus | NEIN | unveraendert lassen. Pausieren ist die konservative Richtung und deckt sich mit Licht und Rollladen. | hoch | `OPEN` |
| `D-B05` | BIO / SLEEP | L4 | Soll der Denon-Nachlauf (R13/R14) bei provisional_sleep pausieren? | bio_state, media_device, PC-/TV-Power | **benni_media_apply (_bio_sleep)**: nur == sleep; **benni_media_apply (BIO_SLEEP_CONTEXT_VALUES)**: PS + sleep | JA - modulinterner Widerspruch in derselben Datei | die Fundstellen vereinheitlichen. Fachlich sprechen beide Richtungen: der Denon soll im Schlaf aus, aber waehrend PS laeuft per Definition noch der TV. Vorschlag: Nachlauf im PS NICHT pausieren (der TV ist ja an), aber die Entscheidung explizit an das Sleep… | hoch | `OPEN` |
| `D-B06` | BIO / SLEEP | L6 | Soll der Sleep-TV-Abschalttimer (R24) bereits bei provisional_sleep laufen? | bio_state, sleep_source, sleep_reference_start, TV-Power | **benni_media_apply**: BIO_SLEEP_CONTEXT_VALUES (PS + sleep); **benni-core-state**: konsumiert off_confirmed als PS->S-Beweis | NEIN - beide Seiten derselben Kette sind konsistent | unveraendert lassen. Genau hier ist PS-als-Schlaf zwingend, weil der Timer den Uebergang PS->S ueberhaupt erst erzeugt. | hoch | `OPEN` |
| `D-B07` | BIO / SLEEP | L4 | Sollen Benachrichtigungen bei provisional_sleep gedaempft werden? | bio_state, quiet_mode, activity_state | **benni_notification_router**: BIO_SLEEP = nur sleep; const.BIO_STATES kennt PS nicht; **benni_media_state (indirekt)**: quiet_mode kann ueber activity_state==sleep entstehen | JA - PS ist beim Router schlicht unbekannt | PS in BIO_STATES aufnehmen und wie sleep daempfen. Ein TV-Abend im Schutzkorridor ist genau die Situation, in der ein Licht-Ring oder ein Media-Ton stoert. | hoch | `OPEN` |
| `D-B08` | BIO / SLEEP | L3 | Soll die Heizung bei provisional_sleep absenken? | bio_state (ueber sensor.benni_combined_context_bio_state) | **benni_climate_policy**: bio in (sleep, waking) -> profile=off; PS ist nicht enthalten | JA gegenueber Licht/Rollladen/Media-Policy | PS wie sleep behandeln (Heizung aus). Waehrend PS liegt der Nutzer bereits; die thermische Traegheit macht ein spaeteres Absenken wirkungsarm. | mittel | `OPEN` |
| `D-B09` | BIO / SLEEP | L3 | Soll waking als Schlaf gelten und die Heizung ausschalten? | bio_state | **benni_climate_policy**: bio in (sleep, waking) -> profile=off; **alle uebrigen Consumer**: waking ist eine Wach-/Weckphase | JA - climate_policy steht gegen alle anderen Domaenen | waking aus der Heizungs-Aus-Menge entfernen. Der Weckzeitpunkt ist genau der Moment, an dem geheizt werden soll; die Traegheit spricht sogar fuer ein Vorheizen davor. | mittel | `OPEN` |
| `D-B10` | BIO / SLEEP | L3 | Soll das Licht bei provisional_sleep hart aus sein? | bio_state | **benni_light_policy**: BIO_SLEEP_CONTEXTS = {provisional_sleep, sleep} | NEIN | unveraendert lassen. Deckt sich mit Rollladen und Media-Policy und ist die konservative Richtung. | hoch | `OPEN` |
| `D-B11` | BIO / SLEEP | L3 | Soll der Rollladen bei provisional_sleep in Schlafposition fahren? | bio_state | **blind_control**: EFFECTIVE_SLEEP_STATES = {provisional_sleep, sleep} | NEIN | unveraendert lassen. | hoch | `OPEN` |
| `D-B12` | BIO / SLEEP | L3 | Sollen Steckdosen-Cuts (Bias Light, Diffuser, PC-Idle) bei provisional_sleep greifen? | bio_state (ueber sensor.benni_combined_context_bio_state), media_context | **plug_policy_engine**: asleep = (bio == sleep) | JA gegenueber Licht/Rollladen | Bias Light und Diffuser wie bei sleep abschalten, den PC-Idle-Cut hingegen NICHT auf PS ausweiten. Bias Light folgt ohnehin dem TV-Stack, und der laeuft im PS noch - die tv_active-Gegenprobe regelt das bereits. | mittel | `OPEN` |
| `D-B13` | BIO / SLEEP | L6 | Soll der manuelle private_time-Latch bei provisional_sleep geraeumt werden? | bio_state, private_time_manual, pc_active | **benni_media_state**: BIO_SLEEP_VALUES = {sleep, asleep}; Flanke nur in diese Menge | teilweise | unveraendert lassen (nur bei bestaetigtem sleep raeumen). Der Latch braucht ohnehin PC-Aktivitaet; PS entsteht am TV. Zusaetzlich greift der 4-Stunden-Timeout. | mittel | `OPEN` |
| `D-B14` | BIO / SLEEP | L2 | Zaehlt provisional_sleep als Aufweck-Kandidat (context_bio_wake_candidate)? | bio_state, Kaffee/PC/PS5/Tuer/Dusche | **einhornzentrale import.yaml**: any(bio == sleep, bio == waking) | teilweise | PS aufnehmen, WENN dieser Combined ueberhaupt live ist. Die Wachsignal-Logik in core_state selbst akzeptiert im PS bereits jedes starke Signal - der Combined bildet damit ein anderes Verhalten ab als der Owner. | niedrig | `OPEN` |
| `D-B15` | BIO / SLEEP | L0 | Ist provisional_sleep ein gueltiger Wert fuer die System-Readiness? | bio_state | **einhornzentrale toolbox_readiness.yaml**: states(...) in ['sleep','waking','awake'] | JA - der Producer emittiert einen Wert, den die Readiness fuer ungueltig haelt | provisional_sleep in die Liste aufnehmen. Alternativ die Wertebereichs-Pruefung durch eine reine Existenzpruefung ersetzen, wie sie fuer day_state, day_context, activity_state und master_context in derselben Datei bereits verwendet wird. | hoch | `OPEN` |
| `D-B16` | BIO / SLEEP | L6 | Darf der Uebergang provisional_sleep -> awake als Wach-FLANKE gelten und damit die Wake-Sequenz (R23) sowie den Stop-Latch-Reset ausloesen? | bio_state (Flanke) | **benni_media_apply**: prev not in (awake, waking) AND cur in (awake, waking) | teilweise - fuer den Latch bewusst beschlossen, fuer die Wake-Sequenz nicht dokumentiert | die beiden Folgen trennen. Der Stop-Latch-Reset auf jeder Wach-Flanke ist gewollt (ausdrueckliche Entscheidung in media_apply#52). Die Wake-Sequenz mit Musikstart sollte dagegen an einen GEPLANTEN Weckzeitpunkt gebunden sein, nicht an ein spontanes Aufstehe… | mittel | `OPEN` |
| `D-B17` | BIO / SLEEP | L2 | Soll Quiet (Ducking) weiterhin ueber activity_state == sleep erkannt werden, also an bio_state vorbei? | activity_state, bio_state, Tuer, Anruf, Musik-Enum | **benni_media_state**: activity_state in (sleep, asleep, quiet) -> quiet_mode; **benni-core-state**: activity=sleep entsteht nur bei Bio S/PS OHNE TV; mit TV gewinnt entertainment | JA - zweiter, undokumentierter Schlafkanal | den Schlaf-Zweig aus evaluate_quiet entfernen und - falls das Verhalten gewollt ist - durch eine explizite bio_state-Bedingung ersetzen. Die toten Werte asleep und quiet entfallen dabei. | hoch | `OPEN` |
| `D-B18` | BIO / SLEEP | L0 | Sollen die drei harten bio_state-String-Literale durch Modulkonstanten ersetzt werden? | bio_state | **benni_media_policy logic.py:350**: == "waking" inline; **benni_media_apply logic.py:487**: not in ("waking","awake") inline neben BIO_AWAKE_VALUES; **benni_climate_policy policy.py:906**: in ("sleep","waking") inline | NEIN (heute verhaltensgleich) | auf die jeweilige Modulkonstante ziehen. Rein mechanisch, kein Verhaltenswechsel. | hoch | `OPEN` |
| `D-B19` | BIO / SLEEP | L0 | Sollen die nie emittierten Alias-Werte sleeping und asleep entfernt werden? | bio_state | **benni_media_policy**: BIO_SLEEP_VALUES enthaelt sleeping und asleep; **benni_media_state**: BIO_SLEEP_VALUES enthaelt asleep; **einhornzentrale radio.yaml**: bio in ['sleep','sleeping'] | NEIN | erst entfernen, NACHDEM ueber einen moeglichen bio_state-Rename entschieden ist. Solange ein Rename sleep -> sleeping im Raum steht, sind genau diese drei Stellen die einzigen, die ihn ueberleben wuerden. | hoch | `OPEN` |
| `D-P01` | PRESENCE / AWAY | L4 | Wer besitzt das Away-EXECUTION-GATE - Core State oder Media State? | presence_personal, presence_effective, presence_away (core), away_gate (media) | **benni-core-state**: publiziert binary_sensor.*_presence_away als 'canonical away gate for the whole fleet' (abwesend UND kein Activity-Hold); **benni_media_state**: leitet Away eigenstaendig aus presence_personal ab (ohne Activity-Hold, mit 25-s-Debounce); **benni_media_policy / benni_media_app… | JA - der deklarierte Kanon hat keinen Consumer, die faktische Wahrheit liegt in einer Nachbardomaene | Owner-Wechsel zurueck zu Core State. Away ist eine Personen-, keine Medienwahrheit. Konkret: media_state konsumiert binary_sensor.*_core_state_presence_away statt presence_personal und projiziert es nur noch (presence_state/away_gate bleiben als Media-Anzei… | hoch (Code-Evidenz); mittel fuer blind_controls Live-Bindung | `OPEN` |
| `D-P02` | PRESENCE / AWAY | L4 | Soll der Activity-Hold von Core State im Media-Away-Gate wirken? | presence_personal, activity_state, effective_hold_active | **benni-core-state**: starke lokale Aktivitaet (private_time/gaming/entertainment/work_home/music=hard, pc_active/household=mid) haelt presence_effective auf assumed home; **benni_media_state**: kennt den Hold nicht; liest nur presence_personal | JA - genau der Fall, fuer den der Hold gebaut wurde, wird im Media-Stack nicht wirksam | ja, der Hold soll wirken. Er wurde in core_state PR3 ausdruecklich gebaut, damit ein GPS-Aussetzer bei aktiver lokaler Nutzung die away-gegateten Consumer (ausdruecklich: Media, Tuer) nicht abreisst. Umsetzung faellt mit D-P01 zusammen. | hoch | `OPEN` |
| `D-P03` | PRESENCE / AWAY | L1 | Wo gehoert der 25-Sekunden-Away-Debounce hin - in den Producer oder in den Consumer? | presence_personal, away_gate | **benni_media_state**: ON-Debounce 25 s im eigenen Coordinator; Rueckkehr wirkt sofort; **benni-core-state**: eigene Stabilisierung ueber presence_effective (arriving 5 s, leaving 60 s, stable_away 120 s) | teilweise - beide Mechanismen sind begruendet, aber unabgestimmt | der Debounce bleibt beim Consumer (Media), aber als dokumentierter Consumer-Parameter, nicht als zweite Away-Wahrheit. Medien haben ein legitimes Eigeninteresse an einer traegen Reaktion (Audiokette nicht abreissen); Tuer und Licht haben es nicht. | mittel | `OPEN` |
| `D-P04` | PRESENCE / AWAY | L2 | Was bedeutet bei_eltern fachlich - Wohnung leer und Person sicher, oder schlicht abwesend? | presence_personal | **benni_light_policy**: PRESENCE_SIM_TRIGGERS = {abwesend, bei_eltern}; **benni_media_state**: bei_eltern -> away_raw False; **plug_policy_engine**: is_home_like; **benni_notification_router**: HOME_EQUIVALENT_PRESENCE; **benni_door_policy**: home-aequivalent fuer Auto-Lock, aber NICHT 'zuhause' … | JA - Licht steht gegen alle uebrigen Domaenen | bei_eltern heisst 'Wohnung leer, Person sicher und erreichbar'. Unter dieser Definition sind ALLE heutigen Verhaltensweisen korrekt, auch die des Lichts - eine Anwesenheitssimulation ist genau dann sinnvoll, wenn die Wohnung leer ist. Es fehlt nur die gemei… | hoch | `OPEN` |
| `D-P05` | PRESENCE / AWAY | L3 | Soll die Anwesenheitssimulation auch bei bei_eltern laufen (nicht nur bei abwesend)? | presence_personal, presence_transition, day_state | **benni_light_policy**: bei_eltern loest hard_off bzw. Simulation aus; coming_home beendet sie sofort | Folgefrage aus D-P04 | unveraendert lassen, sofern D-P04 wie empfohlen entschieden wird. Die Wohnung ist bei bei_eltern leer - genau dafuer ist die Simulation da. | hoch | `OPEN` |
| `D-P06` | PRESENCE / AWAY | L0 | Welche Entity-ID ist die kanonische Bindung - clean slug oder system_-Praefix? | alle publizierten States | **benni_door_policy**: bindet sensor.system_benni_core_state_presence_effective (+ Legacy-Repoint vom clean slug); **einhornzentrale toolbox_readiness.yaml**: bindet sensor.benni_core_state_presence_effective (clean); **benni_media_policy / benni_media_apply**: binden system_ fuer media_states Pr… | JA - dieselbe Wahrheit wird unter zwei IDs gebunden | eine einmalige Live-Inventur der tatsaechlichen Entity-IDs, danach eine Regel festlegen, welche Seite der Standard ist (Producer-Rename oder Consumer-Repoint). Das Muster von light_policy (Migration mit beiden Varianten) ist die einzige Loesung im Stack, di… | mittel (Code-Evidenz sicher, Live-Zustand ungeprueft) | `OPEN` |
| `D-P07` | PRESENCE / AWAY | L1 | Soll blind_control seine Away-Quelle explizit binden statt per Attribut-Heuristik zu erraten? | presence_away (core), presence_state + away_gate (media) | **blind_control**: _is_presence_contract akzeptiert Entities mit away_gate-Attribut oder passendem slug-Attribut; core_states presence_away publiziert beides nicht | JA - der Auto-Binder waehlt eine andere Quelle, als der Reason-String behauptet | die Away-Bindung in blind_control explizit setzen (Options-Flow) und den irrefuehrenden Reason-String korrigieren. Welche Quelle richtig ist, entscheidet D-P01. | hoch (Code); Live-Bindung ungeprueft | `OPEN` |
| `D-P08` | PRESENCE / AWAY | L4 | Wie soll unbekannte Presence behandelt werden - zurueckhalten oder Beobachtung verwerfen? | presence_state, presence_effective | **benni_media_policy**: unknown haelt Auto-Start zurueck, pausiert aber NICHT; **blind_control**: unknown -> ValueError -> Observation unusable; **benni_door_policy**: uncertain/stale -> expliziter Blocker | NEIN - drei unterschiedliche, aber jeweils konservative Auspraegungen | als bewusstes Muster dokumentieren, nicht angleichen. Alle drei erhalten die Unsicherheit als sichtbaren Ausgang, statt sie stillschweigend durch einen Default zu ersetzen - genau das ist die Regel aus der Prozessvorgabe. | hoch | `OPEN` |
| `D-D01` | DAY / ACTIVITY | L1 | Welche Tagesphasen-Generation ist verbindlich - die neun kanonischen Core-State-Phasen oder die alten acht? | day_state | **benni-core-state (Producer)**: neun Phasen (seasonal-accordion-v2), mapping.py fuehrt sie als kanonisch; **blind_control**: strikt die neun; alles andere -> ValueError; **benni_light_policy**: akzeptiert beide Generationen (CORE_DAY_PHASES + LEGACY_DAY_PHASES); **benni_media_policy**: Tabellen … | JA - zwei Generationen im produktiven Betrieb | die neun Core-State-Phasen sind verbindlich (sie sind in mapping.py bereits als kanonisch erklaert und von zwei Consumern korrekt umgesetzt). Daraus folgen zwei getrennte Schritte: (1) Bindungen auf core_state umstellen (D-D05), (2) die 8er-Tabellen fachlic… | hoch | `OPEN` |
| `D-D02` | DAY / ACTIVITY | L1 | Welche HomePods-/Denon-Lautstaerke-Baseline gilt in midday, late_afternoon und evening? | day_state | **benni_media_policy**: HOMEPODS_BASELINES / DENON_BASELINES kennen diese drei Werte nicht | JA - drei von neun Phasen haben faktisch keine Baseline | die Tabellen um die drei Werte ergaenzen. Als Ausgangspunkt (Benni entscheidet die Zahlen): midday nahe afternoon (0.45 / 0.30), late_afternoon zwischen afternoon und evening (0.45 / 0.30), evening am bisherigen early_evening (0.40 / 0.30). Zusaetzlich soll… | hoch (Befund); niedrig (konkrete Zahlen - reine Produktentscheidung) | `OPEN` |
| `D-D03` | DAY / ACTIVITY | L4 | Darf der Subwoofer mittags, am spaeten Nachmittag und am Abend laufen? | day_state, lokale Uhrzeit, entertainment_active, headset_active | **benni_media_policy**: SUB_ALLOWED_PHASES = (late_morning, forenoon, afternoon, early_evening, late_evening) + Wanduhr-Floor 09:00 | JA - der Subwoofer ist mitten am Tag und am Abend gesperrt | midday, late_afternoon und evening aufnehmen. Fachlich war das Fenster als 'ab 09:00 bis einschliesslich spaeter Abend' gemeint; die drei Werte fallen genau in dieses Fenster und fehlen nur, weil sie zur Entstehungszeit nicht existierten. | hoch | `OPEN` |
| `D-D04` | DAY / ACTIVITY | L3 | Wann soll das Bad an freien Tagen vorgeheizt werden? | day_state, day_context | **benni_climate_policy**: (frei AND day_state == late_morning); **benni_climate_policy**: (Werktag AND early_morning) OR late_evening | JA - eine von drei Preheat-Regeln ist tot | late_morning durch forenoon ersetzen (der naechste Nachfolger in der Neun-Phasen-Ordnung). Ob das fachlich der gewuenschte Zeitpunkt ist, entscheidet Benni - die Regel ueberhaupt wieder scharf zu schalten, ist unstrittig. | hoch (Befund); mittel (Ersatzwert) | `OPEN` |
| `D-D05` | DAY / ACTIVITY | L0 | Sollen die Legacy-Bindungen auf sensor.benni_combined_context_* abgeloest werden? | bio_state, activity_state, day_state, day_context, presence_personal, presence_household, presence_band, presence_transition | **benni_climate_policy**: bindet AUSSCHLIESSLICH auf die Combined-IDs (8 von 8 Kontextwerten); **plug_policy_engine**: presence, bio, activity ueber Combined-IDs; **benni_media_policy**: day_state ueber die Combined-ID; **benni_light_policy**: Cutover vollzogen, migration.py dokumentiert ihn; **b… | JA - drei Module haengen an einer als abloesbar markierten Zwischenschicht | ja, ablosen - aber modulweise und in dieser Reihenfolge: (1) plug_policy_engine (einfachste Bindung), (2) climate_policy zusammen mit D-B08/D-B09/D-D04, (3) media_policy ZWINGEND erst nach D-D02 und D-D03. Vorher ist eine Live-Pruefung noetig, ob die Combin… | hoch (Code); niedrig (Live-Existenz der Combined-Entities) | `OPEN` |
| `D-D06` | DAY / ACTIVITY | L3 | Gilt der Abend-Komfortzuschlag der Heizung noch, nachdem free_time aufgefaechert wurde? | activity_state, day_state | **benni_climate_policy**: activity == free_time AND day_state == early_night AND kein Sommermonat | JA - die Regel wurde durch eine Producer-Aenderung faktisch entwertet | die Menge auf die Nachfolger von free_time erweitern (entertainment, music, gaming, pc_active, free_time, idle) - also auf 'Nutzer ist abends zuhause und nicht im Arbeits- oder Haushaltsmodus'. | mittel | `OPEN` |
| `D-D07` | DAY / ACTIVITY | L2 | Sollen work_home und work_away produzierbar gemacht oder als Werte zurueckgezogen werden? | activity_state, homeoffice_ping | **benni-core-state**: work_home braucht CONF_HOMEOFFICE_PING, das bewusst ungebunden ist; work_away ist nicht in ACTIVITY_PRECEDENCE; **benni_light_policy**: work_home -> CCT 5000K; **benni_notification_router**: work_home -> Unterdrueckung; **benni_media_policy**: work_home/work_away -> Boost-Block | NEIN - konsistent, aber unerreichbar | entweder eine Homeoffice-Quelle binden (dann werden drei implementierte Zweige lebendig) oder die Werte als 'geplant, nicht erzeugt' im Katalog kennzeichnen. Nicht empfohlen: die Consumer-Zweige entfernen - sie sind korrekt und billig zu halten. | hoch | `OPEN` |
| `D-D08` | DAY / ACTIVITY | L4 | Welche Activity-Werte duerfen einen Szenen-Preset im Licht treiben, nachdem free_time aufgefaechert wurde? | activity_state | **benni_light_policy**: ACTIVITY_PRESET_DRIVING = {free_time, idle}; **benni_light_policy**: GAMING_ACTIVITY_STATES = {gaming, free_time, idle} - wurde nachgezogen | teilweise - innerhalb desselben Moduls wurde ein Gate nachgezogen, das andere nicht | pruefen, ob music (und ggf. entertainment) in ACTIVITY_PRESET_DRIVING gehoeren. Musik zu hoeren war frueher free_time und hat Presets getrieben; heute tut es das nicht mehr, ohne dass das entschieden wurde. | mittel | `OPEN` |
| `D-M01` | MEDIA CONTEXT / DEVICE / ACTIVITY | L2 | Was bedeutet media_device - Geraete-IDENTITAET, AUDIO-ROUTING oder SCREEN-INTENT? | media_device | **benni_media_apply (DENON_CONSUMER_DEVICES)**: Audio-Routing: welche Geraete brauchen den Denon als Senke; **benni_media_apply (SCREEN_DEVICES)**: Screen-Intent: tv/appletv brauchen den Bildschirm; **benni_media_policy (is_pc_gaming)**: Audio-Pfad: pc bedeutet Headset statt Raumspeaker; **benni_… | JA - ein Enum traegt drei fachliche Bedeutungen | media_device bleibt das beschreibende Primaergeraet (Identitaet). Die beiden abgeleiteten Bedeutungen werden als eigene Felder publiziert - am sinnvollsten als zusaetzliche Felder des bestehenden activity_context-Feeds, nicht als neue Entities: (a) audio_si… | hoch | `OPEN` |
| `D-M02` | MEDIA CONTEXT / DEVICE / ACTIVITY | L2 | Soll es eine publizierte screen_class geben, statt dass vier Consumer 'welche Bildschirmart laeuft' unabhaengig ableiten? | media_context, media_device, gaming_platform, entertainment_active, activity_state, activity_context | **benni_light_policy**: media_context in {tv,streaming} OR media_device == tv, plus entertainment_stable; **blind_control**: eigener Adapter _convert_activity mit eigener Vorrangregel ueber 6 Felder; **plug_policy_engine**: media_context in {movie,streaming,tv,video} OR (gaming AND gaming_source=… | JA - vier Antworten auf dieselbe Frage aus vier Feldkombinationen | ja. screen_class (none\|tv\|pc\|other) als Feld des activity_context-Feeds, mit einer Evidenzliste im Attribut. Das vorhandene Vorbild ist blind_controls _convert_activity - es ist die vollstaendigste Ableitung und kann als Referenzimplementierung dienen, wand… | hoch | `OPEN` |
| `D-M03` | MEDIA CONTEXT / DEVICE / ACTIVITY | L1 | Soll appletv in blind_control als tv-Plattform gelten? | media_device | **benni_media_apply**: appletv in SCREEN_DEVICES; **blind_control**: tv_platforms = {ps5, playstation, xbox, switch, tv}; appletv fehlt | JA - dasselbe Geraet ist in einer Domaene ein Bildschirm, in der anderen nicht | appletv in tv_platforms aufnehmen. Kurzfristiger Einzeiler; mittelfristig durch D-M02 ohnehin abgeloest. | hoch | `OPEN` |
| `D-M04` | MEDIA CONTEXT / DEVICE / ACTIVITY | L4 | Soll der activity_context-Feed Quality und Freshness selbst publizieren, statt dass Core State ueber last_updated schaetzt? | activity_context, Zeitstempel des Feed-Sensors | **benni_media_state (Producer)**: publiziert reason und hold_strength, aber KEIN quality-/freshness-Feld; **benni-core-state**: consumer-seitiges Alters-Gate 1800 s auf last_updated; explizite Marker haetten Vorrang | JA - Luecke, nicht Widerspruch | ja. Der Feed bekommt quality und freshness als Attribute (das Consumer-Gate liest sie bereits bevorzugt aus - es muss also nichts umgebaut werden, nur befuellt). Das ist der billigste und zugleich nuetzlichste Contract-Schritt des gesamten Audits. Ob das al… | hoch | `OPEN` |
| `D-M05` | MEDIA CONTEXT / DEVICE / ACTIVITY | L4 | Wer besitzt die Hold-Staerke - der Media-Feed oder Core State? | activity_context (attr hold_strength), activity_state | **benni_media_state**: publiziert hold_strength (hard fuer private_time/gaming, soft fuer entertainment/music, none fuer idle); **benni-core-state**: eigenes ACTIVITY_HOLD_STRENGTH (high/mid/none) plus SOFT_HOLD_ACTIVITIES | teilweise - zwei parallele Modelle, heute ohne Widerspruch im Ergebnis | Core State behaelt die Hold-Staerke. Die Hold-Frage ist 'darf lokale Aktivitaet ein rohes abwesend ueberstimmen' - das ist eine Presence-Entscheidung, keine Medienentscheidung. Konsequenz: hold_strength im Feed wird als reines Diagnose-/Anzeigefeld gekennze… | mittel | `OPEN` |
| `D-M06` | MEDIA CONTEXT / DEVICE / ACTIVITY | L0 | Sind die als 'nur Debug' deklarierten Attribute am activity_state-Sensor Teil des Consumer-Contracts? | activity_state (attrs media_context, media_device, entertainment_active, gaming_platform, media_activity_context) | **benni-core-state**: Code-Kommentar: 'Debug-Echo aus media_state (treiben die Entscheidung NICHT mehr)'; **blind_control**: liest genau diese Attribute als Evidenz fuer die Screen-Klasse | JA - ein als Debug gekennzeichnetes Feld ist Produktiv-Input einer anderen Domaene | entweder als Teil des Contracts anerkennen und entsprechend kennzeichnen (Attribute duerfen dann nicht ohne Ankuendigung entfallen), oder blind_control auf den Media-Feed direkt binden. Letzteres ist sauberer und faellt mit D-M02 zusammen. | hoch | `OPEN` |
| `D-M07` | MEDIA CONTEXT / DEVICE / ACTIVITY | L3 | Ist audio_owner eine Soll-Entscheidung oder eine Ist-Beobachtung? | media_context, media_device, homepods_state | **benni_media_policy**: Prioritaet des Soll-Stacks aus dem Kontext PLUS die Beobachtung, dass die HomePods gerade spielen; **benni_media_apply**: liest audio_owner == private_stack als Private-Indikator; **benni_media (Umbrella)**: zeigt audio_scenario (reine Soll-Wahrheit) als Hero | JA - der Name sagt Entscheidung, der Wert ist teilweise Beobachtung | der bereits in control#7 vorgeschlagenen Namenskonvention folgen - Ist/Beobachtung als *_current bzw. *_observed, Soll als *_target bzw. *_desired, Freigabe als *_allowed. Konkret: audio_scenario ist bereits die Soll-Wahrheit; audio_owner wird zur reinen Is… | hoch | `OPEN` |
| `D-M08` | MEDIA CONTEXT / DEVICE / ACTIVITY | L3 | Soll es eine oder zwei Bias-Light-Wahrheiten geben? | media_context, gaming_source, entertainment_active, tv_active | **plug_policy_engine**: _decide_bias_light aus media_context + gaming_source + tv_active-Gegenprobe; **einhornzentrale import.yaml**: media_bias_light_should_be_on aus sensor.benni_combined_media_entertainment_active + tv | JA - DUPLICATE_CALCULATION | eine Wahrheit, und zwar plug_policy_engine (es ist der Owner der Steckdose und hat die tv_active-Gegenprobe aus control#35). Den Combined in import.yaml zurueckbauen - abhaengig davon, ob er live ueberhaupt aktiv ist (D-S04). | mittel | `OPEN` |
| `D-M09` | MEDIA CONTEXT / DEVICE / ACTIVITY | L0 | Sollen die toten media_context-Werte movie und video entfernt werden? | media_context | **plug_policy_engine**: want_on = media in {movie, streaming, tv, video} ... | NEIN | entfernen. Anders als die bio-Aliase (D-B19) haben sie keinen Nutzen als Dual-Read-Reserve - es steht kein media_context-Rename im Raum. | hoch | `OPEN` |
| `D-M10` | MEDIA CONTEXT / DEVICE / ACTIVITY | L2 | Braucht der Title-Classifier-Enum 3 (gaming_grind_preemptible) einen eigenen Subcontext-Wert? | media_subcontext, ps5_enum, pc_enum | **benni_media_state**: ENUM_GAME_GRIND_PREEMPTIBLE = 3 existiert als Konstante; **benni_media_policy**: is_grind prueft subcontext == gaming_grind | teilweise - Luecke zwischen Classifier und Subcontext-Enum | klaeren, ob Enum 3 fachlich wie gaming_grind behandelt werden soll (dann auf denselben Subcontext mappen) oder ob ein eigener Wert noetig ist (dann auch media_policy erweitern). | mittel | `OPEN` |
| `D-S01` | SHARED / DEBUG OUTPUTS | L2 | Soll die YAML-Doppelberechnung von presence_preheat_active zurueckgebaut werden? | presence_band, presence_preheat_active | **benni-core-state**: publiziert binary_sensor.*_presence_preheat_active mit Quelle, Startzeit und Maximaldauer; **einhornzentrale import.yaml**: context_presence_preheat_active = (band == preheat) | JA - DUPLICATE_CALCULATION mit abweichender Semantik (Owner hat ein Zeitfenster, der Combined nicht) | zurueckbauen und Consumer auf den Owner-Output binden - abhaengig davon, ob der Combined live ist (D-S04). | hoch (Befund); niedrig (Live-Wirksamkeit) | `OPEN` |
| `D-S02` | SHARED / DEBUG OUTPUTS | L0 | Sollen master_context und live_status Contract-Status bekommen? | master_context, live_status | **einhornzentrale toolbox_readiness.yaml**: prueft master_context nur auf Existenz; **(live_status)**: kein Produktiv-Consumer auf origin/main | NEIN | nein, beide bleiben NOT_A_CONTRACT. Wichtig ist nur das Bewusstsein, dass master_context eine gepunktete Verkettung aller fuenf Kernwerte ist - jeder Enum-Rename aendert diesen String implizit mit. | hoch | `OPEN` |
| `D-S03` | SHARED / DEBUG OUTPUTS | L5 | Soll das action-Enum zwischen media_policy und media_apply geteilt statt kopiert werden? | action | **benni_media_policy**: definiert none\|pause_homepods\|resume_homepods\|start_radio; **benni_media_apply**: haelt eine KOPIE der Konstanten (kein Import) und ergaenzt turn_off_denon | teilweise - bewusste Entkopplung, aber ohne Versionierung | die Kopie beibehalten (der bewusste Verzicht auf Cross-Modul-Imports ist eine tragende Architekturregel des Stacks), aber die Kopplung sichtbar machen - etwa durch einen Contract-Versionsstring in beiden Modulen und einen Test, der die Wertemengen vergleicht. | mittel | `OPEN` |
| `D-S04` | SHARED / DEBUG OUTPUTS | L0 | Ist benni_core_devices/import.yaml live wirksam oder ein reines Config-Artefakt? | alle Combined-Definitionen in import.yaml | **einhornzentrale import.yaml**: definiert u.a. context_bio_wake_candidate, context_presence_preheat_active, media_bias_light_should_be_on | unklar | einmalig live pruefen, ob die in import.yaml definierten Combineds als Entities existieren. Das Ergebnis entscheidet, ob D-B14, D-M08 und D-S01 ueberhaupt Arbeit sind oder nur Dokumentation. | niedrig (das ist ja die Frage) | `OPEN` |
| `D-S05` | SHARED / DEBUG OUTPUTS | L0 | Was geschieht mit publizierten States ohne jeden Consumer? | presence_effective_transition, presence_preheat_active, live_status, presence_away | **benni-core-state**: vier publizierte Outputs ohne gefundenen Produktiv-Consumer auf main | teilweise | nicht entfernen. live_status und presence_effective_transition sind Anzeige- bzw. Diagnoseprojektionen und billig. presence_away ist ein Sonderfall und gehoert zu D-P01. presence_preheat_active sollte Consumer bekommen statt in YAML nachgebaut zu werden (D-… | hoch | `OPEN` |
| `D-W01` | WAKE / TRANSITIONS | L2 | Wer besitzt die Weckentscheidung - ha_wake_planner oder Core State? | wake_state, next_wake, wake_needed, holiday_active | **ha_wake_planner**: publiziert alle vier Werte; **benni-core-state**: publiziert eigene Werte unter sensor.*_core_state_wake_* und vergleicht sie im Shadow; **benni-core-state mapping.py**: fuehrt alle vier als STATUS_PLANNED mit Ziel core_state; **benni_media_policy**: liest wake_needed als Flanke | JA - Doppelpublikation, aber mit bereits definierter Zielrichtung | der in mapping.py festgelegte Ziel-Owner (Core State) ist plausibel und sollte bestaetigt werden. Offen ist nicht die Richtung, sondern das Freigabe-Gate und die Reihenfolge - dafuer existieren bereits control#27 (Shadow-Paritaet), control#28 (Cutover-Gate)… | hoch | `OPEN` |
| `D-W02` | WAKE / TRANSITIONS | L4 | Was ist der kanonische Wake-TRIGGER - bio_state == waking, die bio-Flanke nach awake, oder ein geplanter Weckzeitpunkt (wake_needed)? | bio_state, wake_needed | **benni_media_apply**: Flanke in {awake, waking} loest die Wake-Sequenz aus; **benni_media_policy**: bio_state == "waking" ODER wake_needed haelt die Musik-Baseline zurueck; **benni_media_policy**: steigende wake_needed-Flanke setzt manual_stop zurueck; **benni_light_policy**: bio_state == waking… | JA - drei verschiedene Trigger fuer dieselbe Lebenslage | waking ist der kanonische ZUSTAND fuer alle Zielentscheidungen (Licht, Rollladen, Baseline-Schweigen). Der geplante Weckzeitpunkt (wake_needed bzw. sein Core-State-Nachfolger) sollte der Trigger fuer die AKTION Wake-Sequenz sein. Damit faellt auch D-B16 aus… | hoch | `OPEN` |
| `D-W03` | WAKE / TRANSITIONS | L6 | Soll der manual_stop-Reset laenger als einen Tick halten? | wake_needed, media_stop_latch | **benni_media_policy**: steigende wake_needed-Flanke setzt manual_stop=False; **benni_media_apply**: loest den externen Latch auf der Bio-Wach-Flanke | JA - der Policy-Reset ist wirkungslos, solange der externe Latch steht | der Policy-interne Reset gehoert entweder entfernt (weil Apply den externen Latch loest) oder er muss den externen Latch mitloesen. Beides ist eine Folgeentscheidung aus D-W04. | hoch | `OPEN` |
| `D-W04` | WAKE / TRANSITIONS | L6 | Wer besitzt den Stop-Latch? | input_boolean.media_stop_latch, manual_stop (media_policy) | **benni_media_policy**: fuehrt ein eigenes manual_stop im RAM, liest zusaetzlich den externen Latch; **benni_media_apply**: loest den externen Latch auf der Bio-Wach-Flanke; **einhornzentrale (Skripte)**: script.system_bedtime_mode setzt ihn; mehrere Altskripte loeschen ihn | JA - kein Owner, drei Schreiber | genau einen Owner festlegen. Naheliegend ist media_policy (dort entsteht die Entscheidung 'Nutzer hat gestoppt'), mit dem externen input_boolean als reinem Bedienelement. Die Altskripte, die denselben Zustand parallel setzen, entfallen. | hoch | `OPEN` |

---

## Detailblöcke

### Familie: BIO / SLEEP

#### `D-B01` — Soll Core State einen publizierten Sleep-Kontext-BEGRIFF anbieten, auf den Consumer binden, statt dass jeder Consumer seine eigene Wertemenge pflegt?

- **Decision Level:** `L2 CLASSIFY`
- **Inputs:** `bio_state`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| (heute: keiner) | es gibt keinen publizierten Praedikat-Begriff; jeder Consumer definiert eine eigene Wertemenge | fuenf verschiedene Sleep-Definitionen im Stack |
| benni-core-state (Producer) | publiziert bereits die Attribute sleep_confirmed, sleep_source, sleep_reference_start | die Bausteine fuer ein Praedikat existieren, es fehlt nur die Veroeffentlichung |

- **Klassifikation:** `ARCHITECTURE_DECISION`
- **Semantic Conflict:** JA - Ursache aller nachfolgenden BIO-Decisions

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: ein zusaetzliches boolesches Attribut sleep_context am bestehenden bio_state-Sensor (true fuer provisional_sleep UND sleep), additiv, ohne Enum-Aenderung. Consumer, die 'Schlaf' meinen, binden auf das Attribut; Consumer, die bewusst nur bestaetigten Schlaf meinen, lesen weiterhin sleep_confirmed bzw. den Enum-Wert.

**Begründung:** Das Problem ist nicht der Enum, sondern dass jeder Consumer die Klassifikation neu trifft. Ein Attribut ist additiv, braucht keine Enum-Migration und macht die verbleibenden Unterschiede (D-B02, D-B05) zu einer bewussten Wahl statt zu einem Zufall.

- **Betroffene Consumer:** `alle BIO-Consumer`
- **Betroffene Domains:** Medien, Licht, Rollladen, Klima, Benachrichtigung, Steckdose, YAML
- **Betroffene Edge-IDs:** `E-001`, `E-004`, `E-005`, `E-011`, `E-014`, `E-016`, `E-020`, `E-022`, `E-200`, `E-024`, `E-025`
- **Migrationstyp:** `DOCUMENT_ONLY`
- **Change Type:** `NEW_CONTRACT`
- **Compatibility Requirement:** `NONE`
- **Contract Readiness:** `CONTRACT_AFTER_DECISION`
- **Hängt ab von:** —
- **Priorität:** `P1` · **Konfidenz:** hoch
- **Benni:** `OPEN`

> Blockiert D-B02 bis D-B14. Erst nach dieser Entscheidung ist klar, ob die Einzelfaelle angeglichen oder bewusst getrennt bleiben.

#### `D-B02` — Darf waehrend provisional_sleep automatisch Musik/Radio gestartet werden?

- **Decision Level:** `L4 ARBITRATE/GATE`
- **Inputs:** `bio_state`, `radio_ready`, `presence_state`, `manual_playback`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni_media_policy | PS zaehlt als Schlaf (BIO_SLEEP_VALUES) | music_baseline_candidate=False -> kein Auto-Start |
| benni_media_apply | _bio_sleep prueft nur == sleep | should_autostart_radio laesst PS durch -> Auto-Start moeglich |

- **Klassifikation:** `PRODUCT_DECISION`
- **Semantic Conflict:** JA - die beiden Ebenen derselben Kette widersprechen sich

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: waehrend PS KEIN automatischer Start. PS ist ein Schutzkorridor; ein selbst gestarteter Radiostream ist genau das, was der Korridor verhindern soll. Konkret: media_apply auf das Sleep-Praedikat (D-B01) ziehen.

**Begründung:** Die Policy-Ebene hat die Entscheidung bereits so getroffen; Apply hebt sie faktisch wieder auf. Ein Auto-Start im PS-Korridor ist die einzige Kombination, die den Nutzer aktiv weckt statt nur nichts zu tun.

- **Betroffene Consumer:** `benni_media_policy`, `benni_media_apply`
- **Betroffene Domains:** Medien
- **Betroffene Edge-IDs:** `E-002`, `E-004`
- **Migrationstyp:** `LOCAL_FIX`
- **Change Type:** `SEMANTIC_CHANGE`, `CODE_FIX_ONLY`
- **Compatibility Requirement:** `NONE`
- **Contract Readiness:** `APPLY_INTERNAL`
- **Hängt ab von:** `D-B01`
- **Priorität:** `P2` · **Konfidenz:** hoch
- **Benni:** `OPEN`

> Kleinster wirksamer Einzelfix im Media-Stack, sobald D-B01 entschieden ist.

#### `D-B03` — Darf waehrend provisional_sleep ein Resume bzw. ein Playback-Repair laufen?

- **Decision Level:** `L4 ARBITRATE/GATE`
- **Inputs:** `bio_state`, `homepods_resume_allowed`, `volume_apply_allowed`, `quiet_mode`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni_media_policy | PS zaehlt als Schlaf | Resume blockiert, HomePods-Ziel 0.0 |
| benni_media_apply | BIO_SLEEP_CONTEXT_VALUES enthaelt PS; zusaetzlich fail-closed auf waking/awake | Repair blockiert (sleep_context) bzw. bio_state_unproven |

- **Klassifikation:** `NO_PROBLEM`
- **Semantic Conflict:** NEIN - beide Ebenen blocken, Apply zusaetzlich strenger

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: unveraendert lassen. Das fail-closed-Muster von Apply (positiver Wachbeweis statt Abwesenheit von Schlaf) ist die sicherere Variante und sollte als Muster fuer alle Aktuations-Gates dokumentiert werden.

**Begründung:** Hier ist die strengere Regel bereits an der richtigen Stelle - unmittelbar vor dem Service-Call.

- **Betroffene Consumer:** `benni_media_policy`, `benni_media_apply`
- **Betroffene Domains:** Medien
- **Betroffene Edge-IDs:** `E-001`, `E-005`, `E-006`
- **Migrationstyp:** `DOCUMENT_ONLY`
- **Change Type:** `CODE_FIX_ONLY`
- **Compatibility Requirement:** `NONE`
- **Contract Readiness:** `APPLY_INTERNAL`
- **Hängt ab von:** `D-B01`
- **Priorität:** `P3` · **Konfidenz:** hoch
- **Benni:** `OPEN`

> Nur das harte Literal in logic.py:487 gehoert auf die Konstante gezogen (siehe D-B18).

#### `D-B04` — Sollen die HomePods bei provisional_sleep pausiert werden?

- **Decision Level:** `L3 POLICY`
- **Inputs:** `bio_state`, `media_context`, `audio_owner`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni_media_policy | PS zaehlt als Schlaf -> competes=True | homepods_should_pause, Volume-Ziel 0.0 |
| benni_media_apply | fuehrt die Policy-Entscheidung aus | pause_homepods |

- **Klassifikation:** `NO_PROBLEM`
- **Semantic Conflict:** NEIN

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: unveraendert lassen. Pausieren ist die konservative Richtung und deckt sich mit Licht und Rollladen.

**Begründung:** Der PS-Eintritt setzt ohnehin einen TV-Abend voraus; ein laufender HomePod-Stream ist dort nicht der Erwartungsfall.

- **Betroffene Consumer:** `benni_media_policy`, `benni_media_apply`
- **Betroffene Domains:** Medien
- **Betroffene Edge-IDs:** `E-001`
- **Migrationstyp:** `NO_CHANGE`
- **Change Type:** `CODE_FIX_ONLY`
- **Compatibility Requirement:** `NONE`
- **Contract Readiness:** `POLICY_INTERNAL`
- **Hängt ab von:** `D-B01`
- **Priorität:** `P3` · **Konfidenz:** hoch
- **Benni:** `OPEN`

#### `D-B05` — Soll der Denon-Nachlauf (R13/R14) bei provisional_sleep pausieren?

- **Decision Level:** `L4 ARBITRATE/GATE`
- **Inputs:** `bio_state`, `media_device`, `PC-/TV-Power`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni_media_apply (_bio_sleep) | nur == sleep | Nachlauf laeuft waehrend PS normal weiter |
| benni_media_apply (BIO_SLEEP_CONTEXT_VALUES) | PS + sleep | andere Fundstellen derselben Datei pausieren bei PS |

- **Klassifikation:** `BUG`
- **Semantic Conflict:** JA - modulinterner Widerspruch in derselben Datei

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: die Fundstellen vereinheitlichen. Fachlich sprechen beide Richtungen: der Denon soll im Schlaf aus, aber waehrend PS laeuft per Definition noch der TV. Vorschlag: Nachlauf im PS NICHT pausieren (der TV ist ja an), aber die Entscheidung explizit an das Sleep-Praedikat plus den TV-Zustand binden statt an zwei unterschiedliche bio-Mengen.

**Begründung:** Der aktuelle Zustand ist nicht das Ergebnis einer Entscheidung, sondern zweier unabhaengig gewachsener Fundstellen. Welches Verhalten gewollt ist, muss Benni sagen; dass beide gleichzeitig gelten, ist in jedem Fall falsch.

- **Betroffene Consumer:** `benni_media_apply`
- **Betroffene Domains:** Medien
- **Betroffene Edge-IDs:** `E-004`, `E-010`
- **Migrationstyp:** `LOCAL_FIX`
- **Change Type:** `SEMANTIC_CHANGE`, `CODE_FIX_ONLY`
- **Compatibility Requirement:** `NONE`
- **Contract Readiness:** `APPLY_INTERNAL`
- **Hängt ab von:** `D-B01`
- **Priorität:** `P2` · **Konfidenz:** hoch
- **Benni:** `OPEN`

> Einziger Fall, in dem EIN Modul zwei Sleep-Definitionen gleichzeitig anwendet.

#### `D-B06` — Soll der Sleep-TV-Abschalttimer (R24) bereits bei provisional_sleep laufen?

- **Decision Level:** `L6 LIFECYCLE/RECOVERY`
- **Inputs:** `bio_state`, `sleep_source`, `sleep_reference_start`, `TV-Power`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni_media_apply | BIO_SLEEP_CONTEXT_VALUES (PS + sleep) | Timer armed, publiziert sleep_tv_evidence |
| benni-core-state | konsumiert off_confirmed als PS->S-Beweis | bio_state PS -> sleep |

- **Klassifikation:** `NO_PROBLEM`
- **Semantic Conflict:** NEIN - beide Seiten derselben Kette sind konsistent

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: unveraendert lassen. Genau hier ist PS-als-Schlaf zwingend, weil der Timer den Uebergang PS->S ueberhaupt erst erzeugt.

**Begründung:** Wuerde der Timer erst bei sleep starten, gaebe es keinen Weg von PS nach S ausser manuell.

- **Betroffene Consumer:** `benni_media_apply`, `benni-core-state`
- **Betroffene Domains:** Medien, Person
- **Betroffene Edge-IDs:** `E-009`, `E-026`
- **Migrationstyp:** `NO_CHANGE`
- **Change Type:** `CODE_FIX_ONLY`
- **Compatibility Requirement:** `NONE`
- **Contract Readiness:** `EVENT_CONTRACT`
- **Hängt ab von:** —
- **Priorität:** `P3` · **Konfidenz:** hoch
- **Benni:** `OPEN`

> Der beste Beleg dafuer, dass PS und S NICHT ueberall gleich behandelt werden duerfen.

#### `D-B07` — Sollen Benachrichtigungen bei provisional_sleep gedaempft werden?

- **Decision Level:** `L4 ARBITRATE/GATE`
- **Inputs:** `bio_state`, `quiet_mode`, `activity_state`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni_notification_router | BIO_SLEEP = nur sleep; const.BIO_STATES kennt PS nicht | waehrend PS klingeln Media-Route und Licht-Ring weiter, Push wird nicht deferred |
| benni_media_state (indirekt) | quiet_mode kann ueber activity_state==sleep entstehen | nur wenn KEIN TV laeuft - waehrend PS laeuft aber definitionsgemaess der TV |

- **Klassifikation:** `DRIFT`
- **Semantic Conflict:** JA - PS ist beim Router schlicht unbekannt

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: PS in BIO_STATES aufnehmen und wie sleep daempfen. Ein TV-Abend im Schutzkorridor ist genau die Situation, in der ein Licht-Ring oder ein Media-Ton stoert.

**Begründung:** Der Router kennt den Wert nicht, weil er vor dessen Einfuehrung geschrieben wurde - das ist kein bewusster Produktentscheid.

- **Betroffene Consumer:** `benni_notification_router`
- **Betroffene Domains:** Benachrichtigung
- **Betroffene Edge-IDs:** `E-020`, `E-021`
- **Migrationstyp:** `LOCAL_FIX`
- **Change Type:** `VALUE_ONLY`, `CODE_FIX_ONLY`
- **Compatibility Requirement:** `NONE`
- **Contract Readiness:** `OWNER_INTERNAL`
- **Hängt ab von:** `D-B01`
- **Priorität:** `P2` · **Konfidenz:** hoch
- **Benni:** `OPEN`

> notification_router ist auf main v0.1.0 und hat keine Tests fuer diesen Pfad.

#### `D-B08` — Soll die Heizung bei provisional_sleep absenken?

- **Decision Level:** `L3 POLICY`
- **Inputs:** `bio_state (ueber sensor.benni_combined_context_bio_state)`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni_climate_policy | bio in (sleep, waking) -> profile=off; PS ist nicht enthalten | waehrend PS heizt die Wohnung normal weiter |

- **Klassifikation:** `PRODUCT_DECISION`
- **Semantic Conflict:** JA gegenueber Licht/Rollladen/Media-Policy

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: PS wie sleep behandeln (Heizung aus). Waehrend PS liegt der Nutzer bereits; die thermische Traegheit macht ein spaeteres Absenken wirkungsarm.

**Begründung:** Die Absenkung ist trage - sie zu spaet auszuloesen kostet die Wirkung, sie zu frueh auszuloesen kostet wenig, weil PS ohnehin nur nachts entsteht.

- **Betroffene Consumer:** `benni_climate_policy`
- **Betroffene Domains:** Klima
- **Betroffene Edge-IDs:** `E-200`
- **Migrationstyp:** `LOCAL_FIX`
- **Change Type:** `VALUE_ONLY`, `SEMANTIC_CHANGE`
- **Compatibility Requirement:** `NONE`
- **Contract Readiness:** `OWNER_INTERNAL`
- **Hängt ab von:** `D-B01`, `D-D05`
- **Priorität:** `P2` · **Konfidenz:** mittel
- **Benni:** `OPEN`

> Haengt zusaetzlich an D-D05, weil climate_policy ausschliesslich auf die Legacy-Combined-ID bindet.

#### `D-B09` — Soll waking als Schlaf gelten und die Heizung ausschalten?

- **Decision Level:** `L3 POLICY`
- **Inputs:** `bio_state`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni_climate_policy | bio in (sleep, waking) -> profile=off | beim Aufwachen wird die Heizung AUSgeschaltet statt hochzufahren |
| alle uebrigen Consumer | waking ist eine Wach-/Weckphase | Weckerlicht an, Rollladen faehrt, Media-Wake-Sequenz startet, Router daempft nur |

- **Klassifikation:** `BUG`
- **Semantic Conflict:** JA - climate_policy steht gegen alle anderen Domaenen

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: waking aus der Heizungs-Aus-Menge entfernen. Der Weckzeitpunkt ist genau der Moment, an dem geheizt werden soll; die Traegheit spricht sogar fuer ein Vorheizen davor.

**Begründung:** waking ist in jedem anderen Modul der Beginn des Tages. Dass ausgerechnet die Heizung dann abschaltet, ist mit hoher Wahrscheinlichkeit ein uebernommenes Zwei-Werte-Relikt aus der Zeit vor der Einfuehrung von waking.

- **Betroffene Consumer:** `benni_climate_policy`
- **Betroffene Domains:** Klima
- **Betroffene Edge-IDs:** `E-200`
- **Migrationstyp:** `LOCAL_FIX`
- **Change Type:** `VALUE_ONLY`, `CODE_FIX_ONLY`
- **Compatibility Requirement:** `NONE`
- **Contract Readiness:** `OWNER_INTERNAL`
- **Hängt ab von:** —
- **Priorität:** `P1` · **Konfidenz:** mittel
- **Benni:** `OPEN`

> Live-Wirkung nicht verifiziert: climate_policy bindet auf benni_combined_context_bio_state; ob diese Entity live existiert und einen Wert liefert, ist offen (D-D05). Bei fehlendem Kontext greift zusaetzlich der Default 'sleep' - dann ist die Heizung dauerhaft aus.

#### `D-B10` — Soll das Licht bei provisional_sleep hart aus sein?

- **Decision Level:** `L3 POLICY`
- **Inputs:** `bio_state`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni_light_policy | BIO_SLEEP_CONTEXTS = {provisional_sleep, sleep} | hard_off aller Gruppen |

- **Klassifikation:** `NO_PROBLEM`
- **Semantic Conflict:** NEIN

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: unveraendert lassen. Deckt sich mit Rollladen und Media-Policy und ist die konservative Richtung.

**Begründung:** PS entsteht nur im TV-Abend-Kontext; Deckenlicht ist dort nicht erwuenscht.

- **Betroffene Consumer:** `benni_light_policy`
- **Betroffene Domains:** Licht
- **Betroffene Edge-IDs:** `E-014`
- **Migrationstyp:** `NO_CHANGE`
- **Change Type:** `CODE_FIX_ONLY`
- **Compatibility Requirement:** `NONE`
- **Contract Readiness:** `POLICY_INTERNAL`
- **Hängt ab von:** —
- **Priorität:** `P3` · **Konfidenz:** hoch
- **Benni:** `OPEN`

#### `D-B11` — Soll der Rollladen bei provisional_sleep in Schlafposition fahren?

- **Decision Level:** `L3 POLICY`
- **Inputs:** `bio_state`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| blind_control | EFFECTIVE_SLEEP_STATES = {provisional_sleep, sleep} | Kandidat 'sleep' mit config.target('sleep') |

- **Klassifikation:** `NO_PROBLEM`
- **Semantic Conflict:** NEIN

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: unveraendert lassen.

**Begründung:** Identisch begruendet wie D-B10; zusaetzlich ist die Beobachtung bei blind_control qualitaetsgegatet (unusable statt falscher Annahme).

- **Betroffene Consumer:** `blind_control`
- **Betroffene Domains:** Rollladen
- **Betroffene Edge-IDs:** `E-016`
- **Migrationstyp:** `NO_CHANGE`
- **Change Type:** `CODE_FIX_ONLY`
- **Compatibility Requirement:** `NONE`
- **Contract Readiness:** `POLICY_INTERNAL`
- **Hängt ab von:** —
- **Priorität:** `P3` · **Konfidenz:** hoch
- **Benni:** `OPEN`

> Achtung bei jedem spaeteren Enum-Rename: blind_control validiert die zulaessigen Werte gegen das options-Attribut des Sensors (D-B19 / Rename-Impact).

#### `D-B12` — Sollen Steckdosen-Cuts (Bias Light, Diffuser, PC-Idle) bei provisional_sleep greifen?

- **Decision Level:** `L3 POLICY`
- **Inputs:** `bio_state (ueber sensor.benni_combined_context_bio_state)`, `media_context`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| plug_policy_engine | asleep = (bio == sleep) | waehrend PS bleiben Bias Light, Diffuser und PC-Steckdose an |

- **Klassifikation:** `PRODUCT_DECISION`
- **Semantic Conflict:** JA gegenueber Licht/Rollladen

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: Bias Light und Diffuser wie bei sleep abschalten, den PC-Idle-Cut hingegen NICHT auf PS ausweiten. Bias Light folgt ohnehin dem TV-Stack, und der laeuft im PS noch - die tv_active-Gegenprobe regelt das bereits.

**Begründung:** Bias Light und Diffuser sind Ambiente und gehoeren zur selben Klasse wie Licht. Ein PC-Cut ist dagegen eine harte Aktion mit Datenverlustrisiko und sollte bestaetigten Schlaf verlangen.

- **Betroffene Consumer:** `plug_policy_engine`
- **Betroffene Domains:** Steckdose
- **Betroffene Edge-IDs:** `E-022`, `E-126`
- **Migrationstyp:** `LOCAL_FIX`
- **Change Type:** `VALUE_ONLY`, `SEMANTIC_CHANGE`
- **Compatibility Requirement:** `NONE`
- **Contract Readiness:** `OWNER_INTERNAL`
- **Hängt ab von:** `D-B01`, `D-D05`
- **Priorität:** `P2` · **Konfidenz:** mittel
- **Benni:** `OPEN`

> Gutes Beispiel dafuer, dass 'PS wie S' pro Decision und nicht pauschal zu beantworten ist.

#### `D-B13` — Soll der manuelle private_time-Latch bei provisional_sleep geraeumt werden?

- **Decision Level:** `L6 LIFECYCLE/RECOVERY`
- **Inputs:** `bio_state`, `private_time_manual`, `pc_active`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni_media_state | BIO_SLEEP_VALUES = {sleep, asleep}; Flanke nur in diese Menge | beim Uebergang awake -> PS bleibt der Latch stehen |

- **Klassifikation:** `PRODUCT_DECISION`
- **Semantic Conflict:** teilweise

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: unveraendert lassen (nur bei bestaetigtem sleep raeumen). Der Latch braucht ohnehin PC-Aktivitaet; PS entsteht am TV. Zusaetzlich greift der 4-Stunden-Timeout.

**Begründung:** Ein Clear auf der PS-Flanke wuerde eine bewusst gesetzte Nutzerabsicht aufgrund eines nur vermuteten Schlafs verwerfen.

- **Betroffene Consumer:** `benni_media_state`
- **Betroffene Domains:** Medien
- **Betroffene Edge-IDs:** `E-011`
- **Migrationstyp:** `NO_CHANGE`
- **Change Type:** `CODE_FIX_ONLY`
- **Compatibility Requirement:** `NONE`
- **Contract Readiness:** `OWNER_INTERNAL`
- **Hängt ab von:** `D-B01`
- **Priorität:** `P3` · **Konfidenz:** mittel
- **Benni:** `OPEN`

> Der tote Alias 'asleep' gehoert unabhaengig davon entfernt (D-B19).

#### `D-B14` — Zaehlt provisional_sleep als Aufweck-Kandidat (context_bio_wake_candidate)?

- **Decision Level:** `L2 CLASSIFY`
- **Inputs:** `bio_state`, `Kaffee/PC/PS5/Tuer/Dusche`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| einhornzentrale import.yaml | any(bio == sleep, bio == waking) | waehrend PS kein wake_candidate |

- **Klassifikation:** `DRIFT`
- **Semantic Conflict:** teilweise

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: PS aufnehmen, WENN dieser Combined ueberhaupt live ist. Die Wachsignal-Logik in core_state selbst akzeptiert im PS bereits jedes starke Signal - der Combined bildet damit ein anderes Verhalten ab als der Owner.

**Begründung:** Ein Wach-Kandidat, der einen der drei Schlafzustaende auslaesst, beschreibt den Owner falsch.

- **Betroffene Consumer:** `einhornzentrale (import.yaml)`
- **Betroffene Domains:** YAML-Konfiguration
- **Betroffene Edge-IDs:** `E-024`
- **Migrationstyp:** `LOCAL_FIX`
- **Change Type:** `VALUE_ONLY`
- **Compatibility Requirement:** `UNKNOWN`
- **Contract Readiness:** `NOT_A_CONTRACT`
- **Hängt ab von:** `D-B01`, `D-S04`
- **Priorität:** `P3` · **Konfidenz:** niedrig
- **Benni:** `OPEN`

> Wirksamkeit unklar: laut Betriebsstand ist der Bulk-Import nicht live gemergt (D-S04).

#### `D-B15` — Ist provisional_sleep ein gueltiger Wert fuer die System-Readiness?

- **Decision Level:** `L0 FACT`
- **Inputs:** `bio_state`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| einhornzentrale toolbox_readiness.yaml | states(...) in ['sleep','waking','awake'] | binary_sensor.system_benni_context_ready geht im legitimen Zustand PS auf OFF |

- **Klassifikation:** `BUG`
- **Semantic Conflict:** JA - der Producer emittiert einen Wert, den die Readiness fuer ungueltig haelt

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: provisional_sleep in die Liste aufnehmen. Alternativ die Wertebereichs-Pruefung durch eine reine Existenzpruefung ersetzen, wie sie fuer day_state, day_context, activity_state und master_context in derselben Datei bereits verwendet wird.

**Begründung:** Eine Readiness, die bei einem regulaeren Producer-Wert Alarm schlaegt, erzeugt jede Nacht ein falsches Negativ und entwertet damit das Signal.

- **Betroffene Consumer:** `einhornzentrale (toolbox_readiness.yaml)`
- **Betroffene Domains:** YAML-Konfiguration, Betrieb
- **Betroffene Edge-IDs:** `E-025`, `E-078`, `E-085`
- **Migrationstyp:** `LOCAL_FIX`
- **Change Type:** `VALUE_ONLY`
- **Compatibility Requirement:** `NONE`
- **Contract Readiness:** `NOT_A_CONTRACT`
- **Hängt ab von:** —
- **Priorität:** `P1` · **Konfidenz:** hoch
- **Benni:** `OPEN`

> Billigster Fix mit sofort sichtbarer Wirkung. Die uebrigen Consumer dieses Sensors sind nicht Teil dieses Audits.

#### `D-B16` — Darf der Uebergang provisional_sleep -> awake als Wach-FLANKE gelten und damit die Wake-Sequenz (R23) sowie den Stop-Latch-Reset ausloesen?

- **Decision Level:** `L6 LIFECYCLE/RECOVERY`
- **Inputs:** `bio_state (Flanke)`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni_media_apply | prev not in (awake, waking) AND cur in (awake, waking) | Wake-Sequenz startet (HomePods-Startlautstaerke, Ramp, Radio-Autostart) UND Stop-Latch wird zurueckgesetzt |

- **Klassifikation:** `PRODUCT_DECISION`
- **Semantic Conflict:** teilweise - fuer den Latch bewusst beschlossen, fuer die Wake-Sequenz nicht dokumentiert

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: die beiden Folgen trennen. Der Stop-Latch-Reset auf jeder Wach-Flanke ist gewollt (ausdrueckliche Entscheidung in media_apply#52). Die Wake-Sequenz mit Musikstart sollte dagegen an einen GEPLANTEN Weckzeitpunkt gebunden sein, nicht an ein spontanes Aufstehen um 23:40 aus dem TV-Abend.

**Begründung:** PS -> awake entsteht schon dann, wenn jemand waehrend des TV-Abends aufsteht und die Kaffeemaschine oder den PC beruehrt. Ein Radiostart ist dort nicht die erwartete Reaktion.

- **Betroffene Consumer:** `benni_media_apply`
- **Betroffene Domains:** Medien
- **Betroffene Edge-IDs:** `E-007`
- **Migrationstyp:** `SEMANTIC_SPLIT`
- **Change Type:** `SEMANTIC_CHANGE`
- **Compatibility Requirement:** `NONE`
- **Contract Readiness:** `APPLY_INTERNAL`
- **Hängt ab von:** `D-W02`
- **Priorität:** `P2` · **Konfidenz:** mittel
- **Benni:** `OPEN`

> Haengt an D-W02: solange nicht entschieden ist, was der kanonische Wake-Trigger ist, kann die Sequenz nicht sauber umgebunden werden.

#### `D-B17` — Soll Quiet (Ducking) weiterhin ueber activity_state == sleep erkannt werden, also an bio_state vorbei?

- **Decision Level:** `L2 CLASSIFY`
- **Inputs:** `activity_state`, `bio_state`, `Tuer`, `Anruf`, `Musik-Enum`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni_media_state | activity_state in (sleep, asleep, quiet) -> quiet_mode | Ducking, Subwoofer aus, R20-Snapshot - ausgeloest durch einen Aktivitaets- statt Koerperzustand |
| benni-core-state | activity=sleep entsteht nur bei Bio S/PS OHNE TV; mit TV gewinnt entertainment | der Zweig feuert genau dann nicht, wenn man ihn am ehesten erwarten wuerde |

- **Klassifikation:** `BUG`
- **Semantic Conflict:** JA - zweiter, undokumentierter Schlafkanal

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: den Schlaf-Zweig aus evaluate_quiet entfernen und - falls das Verhalten gewollt ist - durch eine explizite bio_state-Bedingung ersetzen. Die toten Werte asleep und quiet entfallen dabei.

**Begründung:** Schlaf ueber activity_state zu erkennen umgeht den Owner und erzeugt ein Verhalten, das vom TV-Zustand abhaengt, ohne dass das irgendwo als Absicht steht.

- **Betroffene Consumer:** `benni_media_state`, `benni_media_policy`, `benni_media_apply`, `benni_notification_router`
- **Betroffene Domains:** Medien, Benachrichtigung
- **Betroffene Edge-IDs:** `E-039`, `E-046`, `E-170`, `E-171`, `E-172`
- **Migrationstyp:** `LOCAL_FIX`
- **Change Type:** `SEMANTIC_CHANGE`, `LEGACY_REMOVAL`
- **Compatibility Requirement:** `NONE`
- **Contract Readiness:** `OWNER_INTERNAL`
- **Hängt ab von:** —
- **Priorität:** `P1` · **Konfidenz:** hoch
- **Benni:** `OPEN`

> Bestaetigt OPR-033. Wirkt ueber quiet_mode bis in die Benachrichtigungen durch.

#### `D-B18` — Sollen die drei harten bio_state-String-Literale durch Modulkonstanten ersetzt werden?

- **Decision Level:** `L0 FACT`
- **Inputs:** `bio_state`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni_media_policy logic.py:350 | == "waking" inline | Wake-Fenster-Gate |
| benni_media_apply logic.py:487 | not in ("waking","awake") inline neben BIO_AWAKE_VALUES | Repair-Gate |
| benni_climate_policy policy.py:906 | in ("sleep","waking") inline | Heizprofil-Gate |

- **Klassifikation:** `DRIFT`
- **Semantic Conflict:** NEIN (heute verhaltensgleich)

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: auf die jeweilige Modulkonstante ziehen. Rein mechanisch, kein Verhaltenswechsel.

**Begründung:** Diese drei Stellen wuerden jeden Enum-Rename still ueberleben und danach falsch entscheiden. Genau das ist der teure Teil eines Renames (siehe Rename-Impact im Audit).

- **Betroffene Consumer:** `benni_media_policy`, `benni_media_apply`, `benni_climate_policy`
- **Betroffene Domains:** Medien, Klima
- **Betroffene Edge-IDs:** `E-003`, `E-006`, `E-200`
- **Migrationstyp:** `LOCAL_FIX`
- **Change Type:** `CODE_FIX_ONLY`
- **Compatibility Requirement:** `NONE`
- **Contract Readiness:** `NOT_A_CONTRACT`
- **Hängt ab von:** —
- **Priorität:** `P3` · **Konfidenz:** hoch
- **Benni:** `OPEN`

> Voraussetzung dafuer, dass ein spaeterer Rename ueberhaupt planbar wird.

#### `D-B19` — Sollen die nie emittierten Alias-Werte sleeping und asleep entfernt werden?

- **Decision Level:** `L0 FACT`
- **Inputs:** `bio_state`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni_media_policy | BIO_SLEEP_VALUES enthaelt sleeping und asleep | toter Zweig |
| benni_media_state | BIO_SLEEP_VALUES enthaelt asleep | toter Zweig |
| einhornzentrale radio.yaml | bio in ['sleep','sleeping'] | toter Zweig; vermutliche Herkunft des Alias |

- **Klassifikation:** `LEGACY`
- **Semantic Conflict:** NEIN

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: erst entfernen, NACHDEM ueber einen moeglichen bio_state-Rename entschieden ist. Solange ein Rename sleep -> sleeping im Raum steht, sind genau diese drei Stellen die einzigen, die ihn ueberleben wuerden.

**Begründung:** Tote Werte sind hier ausnahmsweise nuetzlich: sie sind eine unfreiwillige Dual-Read-Faehigkeit. Sie jetzt zu entfernen, verteuert einen spaeteren Rename.

- **Betroffene Consumer:** `benni_media_policy`, `benni_media_state`, `einhornzentrale`
- **Betroffene Domains:** Medien, YAML-Konfiguration
- **Betroffene Edge-IDs:** `E-001`, `E-011`, `E-205`
- **Migrationstyp:** `DOCUMENT_ONLY`
- **Change Type:** `LEGACY_REMOVAL`
- **Compatibility Requirement:** `TEMP_ALIAS`
- **Contract Readiness:** `NOT_A_CONTRACT`
- **Hängt ab von:** `D-B01`
- **Priorität:** `P3` · **Konfidenz:** hoch
- **Benni:** `OPEN`

> core_state hat sleeping/asleep nie emittiert (belegt in OPR-031); die Herkunft ist radio.yaml.

---

### Familie: PRESENCE / AWAY

#### `D-P01` — Wer besitzt das Away-EXECUTION-GATE - Core State oder Media State?

- **Decision Level:** `L4 ARBITRATE/GATE`
- **Inputs:** `presence_personal`, `presence_effective`, `presence_away (core)`, `away_gate (media)`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni-core-state | publiziert binary_sensor.*_presence_away als 'canonical away gate for the whole fleet' (abwesend UND kein Activity-Hold) | auf origin/main KEIN Produktiv-Consumer gefunden |
| benni_media_state | leitet Away eigenstaendig aus presence_personal ab (ohne Activity-Hold, mit 25-s-Debounce) | away_gate + presence_state; beliefert media_policy, media_apply und sehr wahrscheinlich blind_control |
| benni_media_policy / benni_media_apply | lesen media_states Projektion | harter Media-Block |
| blind_control | Auto-Suggestion bevorzugt Entities mit away_gate-Attribut - das hat nur media_state | Away-Kandidat des Rollladens stammt vermutlich aus dem Media-Stack |

- **Klassifikation:** `ARCHITECTURE_DECISION`
- **Semantic Conflict:** JA - der deklarierte Kanon hat keinen Consumer, die faktische Wahrheit liegt in einer Nachbardomaene

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: Owner-Wechsel zurueck zu Core State. Away ist eine Personen-, keine Medienwahrheit. Konkret: media_state konsumiert binary_sensor.*_core_state_presence_away statt presence_personal und projiziert es nur noch (presence_state/away_gate bleiben als Media-Anzeige bestehen). Alternativ - falls das nicht gewollt ist - den Kanonik-Anspruch im Core-State-Code zuruecknehmen und presence_away als reine Diagnose kennzeichnen.

**Begründung:** Ein als kanonisch bezeichneter Gate ohne Consumer ist irrefuehrend. Solange media_state Away selbst berechnet, wirkt der Activity-Hold von Core State in der gesamten Media- und vermutlich auch Rollladen-Kette nicht.

- **Betroffene Consumer:** `benni_media_state`, `benni_media_policy`, `benni_media_apply`, `blind_control`, `benni-core-state`
- **Betroffene Domains:** Person, Medien, Rollladen
- **Betroffene Edge-IDs:** `E-070`, `E-071`, `E-076`, `E-165`, `E-167`, `E-168`
- **Migrationstyp:** `CONTRACT_CUTOVER`
- **Change Type:** `OWNER_CHANGE`, `BINDING_CHANGE`, `SEMANTIC_CHANGE`
- **Compatibility Requirement:** `DUAL_READ`
- **Contract Readiness:** `CONTRACT_READY`
- **Hängt ab von:** —
- **Priorität:** `P1` · **Konfidenz:** hoch (Code-Evidenz); mittel fuer blind_controls Live-Bindung
- **Benni:** `OPEN`

> Blockiert D-P02 und D-P03. Vor der Umsetzung muss die Live-Bindung von blind_control geprueft werden.

#### `D-P02` — Soll der Activity-Hold von Core State im Media-Away-Gate wirken?

- **Decision Level:** `L4 ARBITRATE/GATE`
- **Inputs:** `presence_personal`, `activity_state`, `effective_hold_active`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni-core-state | starke lokale Aktivitaet (private_time/gaming/entertainment/work_home/music=hard, pc_active/household=mid) haelt presence_effective auf assumed home | presence_away bleibt OFF trotz rohem abwesend |
| benni_media_state | kennt den Hold nicht; liest nur presence_personal | ein GPS-Blip bei laufendem Gaming reisst nach 25 s die Audio-Kette ab |

- **Klassifikation:** `BUG`
- **Semantic Conflict:** JA - genau der Fall, fuer den der Hold gebaut wurde, wird im Media-Stack nicht wirksam

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: ja, der Hold soll wirken. Er wurde in core_state PR3 ausdruecklich gebaut, damit ein GPS-Aussetzer bei aktiver lokaler Nutzung die away-gegateten Consumer (ausdruecklich: Media, Tuer) nicht abreisst. Umsetzung faellt mit D-P01 zusammen.

**Begründung:** Der Docstring von PresenceAwayBinarySensor nennt Media als Ziel-Consumer. Die Absicht ist dokumentiert, die Verkabelung fehlt.

- **Betroffene Consumer:** `benni_media_state`, `benni_media_policy`, `benni_media_apply`
- **Betroffene Domains:** Person, Medien
- **Betroffene Edge-IDs:** `E-043`, `E-044`, `E-070`, `E-076`
- **Migrationstyp:** `CROSS_DOMAIN_HOTFIX`
- **Change Type:** `BINDING_CHANGE`, `SEMANTIC_CHANGE`
- **Compatibility Requirement:** `DUAL_READ`
- **Contract Readiness:** `CONTRACT_READY`
- **Hängt ab von:** `D-P01`
- **Priorität:** `P2` · **Konfidenz:** hoch
- **Benni:** `OPEN`

> Der Hold bricht bei bestaetigtem Far-Away fuer weiche Aktivitaeten (music/entertainment) ohnehin - das Risiko einer haengenden Anwesenheit ist damit begrenzt.

#### `D-P03` — Wo gehoert der 25-Sekunden-Away-Debounce hin - in den Producer oder in den Consumer?

- **Decision Level:** `L1 NORMALIZE`
- **Inputs:** `presence_personal`, `away_gate`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni_media_state | ON-Debounce 25 s im eigenen Coordinator; Rueckkehr wirkt sofort | Media reagiert spaeter auf Weggehen als der Rest der Flotte |
| benni-core-state | eigene Stabilisierung ueber presence_effective (arriving 5 s, leaving 60 s, stable_away 120 s) | zwei unabhaengige Zeitkonstanten fuer dieselbe Frage |

- **Klassifikation:** `ARCHITECTURE_DECISION`
- **Semantic Conflict:** teilweise - beide Mechanismen sind begruendet, aber unabgestimmt

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: der Debounce bleibt beim Consumer (Media), aber als dokumentierter Consumer-Parameter, nicht als zweite Away-Wahrheit. Medien haben ein legitimes Eigeninteresse an einer traegen Reaktion (Audiokette nicht abreissen); Tuer und Licht haben es nicht.

**Begründung:** Ein domaenenspezifisches Zeitverhalten ist kein Duplicate Truth, solange die Klassifikation selbst aus einer Quelle kommt. Genau das stellt D-P01 her.

- **Betroffene Consumer:** `benni_media_state`
- **Betroffene Domains:** Medien, Person
- **Betroffene Edge-IDs:** `E-071`
- **Migrationstyp:** `DOCUMENT_ONLY`
- **Change Type:** `SEMANTIC_CHANGE`
- **Compatibility Requirement:** `NONE`
- **Contract Readiness:** `OWNER_INTERNAL`
- **Hängt ab von:** `D-P01`
- **Priorität:** `P2` · **Konfidenz:** mittel
- **Benni:** `OPEN`

> Die 25 s erklaeren, warum Core und Media beim Weggehen kurzzeitig auseinanderlaufen koennen.

#### `D-P04` — Was bedeutet bei_eltern fachlich - Wohnung leer und Person sicher, oder schlicht abwesend?

- **Decision Level:** `L2 CLASSIFY`
- **Inputs:** `presence_personal`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni_light_policy | PRESENCE_SIM_TRIGGERS = {abwesend, bei_eltern} | hard_off bzw. Anwesenheitssimulation |
| benni_media_state | bei_eltern -> away_raw False | kein Medien-Stopp |
| plug_policy_engine | is_home_like | kein Away-Cut |
| benni_notification_router | HOME_EQUIVALENT_PRESENCE | lokale Kanaele bleiben erlaubt |
| benni_door_policy | home-aequivalent fuer Auto-Lock, aber NICHT 'zuhause' fuer Auto-Unlock | kein Auto-Lock, Auto-Unlock moeglich |

- **Klassifikation:** `PRODUCT_DECISION`
- **Semantic Conflict:** JA - Licht steht gegen alle uebrigen Domaenen

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: bei_eltern heisst 'Wohnung leer, Person sicher und erreichbar'. Unter dieser Definition sind ALLE heutigen Verhaltensweisen korrekt, auch die des Lichts - eine Anwesenheitssimulation ist genau dann sinnvoll, wenn die Wohnung leer ist. Es fehlt nur die gemeinsame Aussage, nicht die Angleichung.

**Begründung:** Die Divergenz ist kein Fehler, sondern eine unausgesprochene Definition. Wird sie einmal formuliert, loesen sich vier von fuenf scheinbaren Widerspruechen auf.

- **Betroffene Consumer:** `benni_light_policy`, `benni_media_state`, `plug_policy_engine`, `benni_notification_router`, `benni_door_policy`
- **Betroffene Domains:** Licht, Medien, Steckdose, Benachrichtigung, Tuer
- **Betroffene Edge-IDs:** `E-070`, `E-072`, `E-073`, `E-074`, `E-081`, `E-084`
- **Migrationstyp:** `DOCUMENT_ONLY`
- **Change Type:** `SEMANTIC_CHANGE`
- **Compatibility Requirement:** `NONE`
- **Contract Readiness:** `CONTRACT_AFTER_DECISION`
- **Hängt ab von:** —
- **Priorität:** `P2` · **Konfidenz:** hoch
- **Benni:** `OPEN`

> Billigste der P2-Entscheidungen: vermutlich ohne jede Codeaenderung abschliessbar.

#### `D-P05` — Soll die Anwesenheitssimulation auch bei bei_eltern laufen (nicht nur bei abwesend)?

- **Decision Level:** `L3 POLICY`
- **Inputs:** `presence_personal`, `presence_transition`, `day_state`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni_light_policy | bei_eltern loest hard_off bzw. Simulation aus; coming_home beendet sie sofort | Licht verhaelt sich wie bei echter Abwesenheit |

- **Klassifikation:** `NO_PROBLEM`
- **Semantic Conflict:** Folgefrage aus D-P04

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: unveraendert lassen, sofern D-P04 wie empfohlen entschieden wird. Die Wohnung ist bei bei_eltern leer - genau dafuer ist die Simulation da.

**Begründung:** Unter der Definition aus D-P04 ist das Licht der einzige Consumer, der die Situation korrekt als 'Wohnung leer' behandelt.

- **Betroffene Consumer:** `benni_light_policy`
- **Betroffene Domains:** Licht
- **Betroffene Edge-IDs:** `E-072`, `E-100`
- **Migrationstyp:** `NO_CHANGE`
- **Change Type:** `CODE_FIX_ONLY`
- **Compatibility Requirement:** `NONE`
- **Contract Readiness:** `POLICY_INTERNAL`
- **Hängt ab von:** `D-P04`
- **Priorität:** `P3` · **Konfidenz:** hoch
- **Benni:** `OPEN`

#### `D-P06` — Welche Entity-ID ist die kanonische Bindung - clean slug oder system_-Praefix?

- **Decision Level:** `L0 FACT`
- **Inputs:** `alle publizierten States`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni_door_policy | bindet sensor.system_benni_core_state_presence_effective (+ Legacy-Repoint vom clean slug) | funktioniert live |
| einhornzentrale toolbox_readiness.yaml | bindet sensor.benni_core_state_presence_effective (clean) | andere ID fuer dieselbe Wahrheit |
| benni_media_policy / benni_media_apply | binden system_ fuer media_states Presence, clean fuer alles uebrige | gemischt |
| benni-core-state | Prefill fuer sleep_tv_evidence zeigt auf den clean slug | live existiert nur der system_-Slug (OPR-141) |
| benni_light_policy | migration.py behandelt beide Varianten | einziges Modul mit systematischer Loesung |

- **Klassifikation:** `DRIFT`
- **Semantic Conflict:** JA - dieselbe Wahrheit wird unter zwei IDs gebunden

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: eine einmalige Live-Inventur der tatsaechlichen Entity-IDs, danach eine Regel festlegen, welche Seite der Standard ist (Producer-Rename oder Consumer-Repoint). Das Muster von light_policy (Migration mit beiden Varianten) ist die einzige Loesung im Stack, die beide Faelle ueberlebt.

**Begründung:** Das ist keine Semantik-, sondern eine Betriebsfrage. Sie erzeugt aber stille Fehler (presence_effective_missing, tote Prefills), die wie Logikfehler aussehen.

- **Betroffene Consumer:** `benni_door_policy`, `benni-core-state`, `benni_media_policy`, `benni_media_apply`, `einhornzentrale`
- **Betroffene Domains:** Betrieb, alle
- **Betroffene Edge-IDs:** `E-026`, `E-080`, `E-085`, `E-165`, `E-167`, `E-110`
- **Migrationstyp:** `CROSS_DOMAIN_HOTFIX`
- **Change Type:** `BINDING_CHANGE`
- **Compatibility Requirement:** `DUAL_READ`
- **Contract Readiness:** `NOT_A_CONTRACT`
- **Hängt ab von:** —
- **Priorität:** `P2` · **Konfidenz:** mittel (Code-Evidenz sicher, Live-Zustand ungeprueft)
- **Benni:** `OPEN`

> Betrifft auch D-B06/D-B09: ob climate_policy und die sleep_tv_evidence-Kette live ueberhaupt gebunden sind, haengt an dieser Frage.

#### `D-P07` — Soll blind_control seine Away-Quelle explizit binden statt per Attribut-Heuristik zu erraten?

- **Decision Level:** `L1 NORMALIZE`
- **Inputs:** `presence_away (core)`, `presence_state + away_gate (media)`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| blind_control | _is_presence_contract akzeptiert Entities mit away_gate-Attribut oder passendem slug-Attribut; core_states presence_away publiziert beides nicht | nur media_state erfuellt das Praedikat; der Reason-String heisst trotzdem core_state_away_gate_attribute |

- **Klassifikation:** `BUG`
- **Semantic Conflict:** JA - der Auto-Binder waehlt eine andere Quelle, als der Reason-String behauptet

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: die Away-Bindung in blind_control explizit setzen (Options-Flow) und den irrefuehrenden Reason-String korrigieren. Welche Quelle richtig ist, entscheidet D-P01.

**Begründung:** Eine Heuristik, die eine andere Domaene waehlt als ihr eigener Diagnosetext angibt, macht jede spaetere Fehlersuche teuer.

- **Betroffene Consumer:** `blind_control`
- **Betroffene Domains:** Rollladen
- **Betroffene Edge-IDs:** `E-168`
- **Migrationstyp:** `LOCAL_FIX`
- **Change Type:** `BINDING_CHANGE`, `CODE_FIX_ONLY`
- **Compatibility Requirement:** `NONE`
- **Contract Readiness:** `OWNER_INTERNAL`
- **Hängt ab von:** `D-P01`
- **Priorität:** `P2` · **Konfidenz:** hoch (Code); Live-Bindung ungeprueft
- **Benni:** `OPEN`

> Gilt sinngemaess auch fuer die activity_state-Auto-Suggestion (_is_core_activity).

#### `D-P08` — Wie soll unbekannte Presence behandelt werden - zurueckhalten oder Beobachtung verwerfen?

- **Decision Level:** `L4 ARBITRATE/GATE`
- **Inputs:** `presence_state`, `presence_effective`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni_media_policy | unknown haelt Auto-Start zurueck, pausiert aber NICHT | laufende Musik bleibt |
| blind_control | unknown -> ValueError -> Observation unusable | kein Away-Kandidat, Quality wird durchgereicht |
| benni_door_policy | uncertain/stale -> expliziter Blocker | keine Schloss-Aktion |

- **Klassifikation:** `NO_PROBLEM`
- **Semantic Conflict:** NEIN - drei unterschiedliche, aber jeweils konservative Auspraegungen

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: als bewusstes Muster dokumentieren, nicht angleichen. Alle drei erhalten die Unsicherheit als sichtbaren Ausgang, statt sie stillschweigend durch einen Default zu ersetzen - genau das ist die Regel aus der Prozessvorgabe.

**Begründung:** Die Domaenen haben unterschiedliche Kosten fuer Fehlentscheidungen (laufende Musik abreissen vs. Rollladen fahren vs. Tuer entriegeln); die jeweils gewaehlte Konservativitaet passt dazu.

- **Betroffene Consumer:** `benni_media_policy`, `benni_media_apply`, `blind_control`, `benni_door_policy`
- **Betroffene Domains:** Medien, Rollladen, Tuer
- **Betroffene Edge-IDs:** `E-082`, `E-166`, `E-168`
- **Migrationstyp:** `DOCUMENT_ONLY`
- **Change Type:** `CODE_FIX_ONLY`
- **Compatibility Requirement:** `NONE`
- **Contract Readiness:** `CONTRACT_READY`
- **Hängt ab von:** —
- **Priorität:** `P3` · **Konfidenz:** hoch
- **Benni:** `OPEN`

> Positivbeispiel: so sollte mit unknown/unavailable/stale ueberall umgegangen werden.

---

### Familie: DAY / ACTIVITY

#### `D-D01` — Welche Tagesphasen-Generation ist verbindlich - die neun kanonischen Core-State-Phasen oder die alten acht?

- **Decision Level:** `L1 NORMALIZE`
- **Inputs:** `day_state`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni-core-state (Producer) | neun Phasen (seasonal-accordion-v2), mapping.py fuehrt sie als kanonisch | early_night..late_evening |
| blind_control | strikt die neun; alles andere -> ValueError | Observation unusable |
| benni_light_policy | akzeptiert beide Generationen (CORE_DAY_PHASES + LEGACY_DAY_PHASES) | funktioniert in jedem Fall |
| benni_media_policy | Tabellen im 8-Phasen-Vokabular, keine Validierung | stiller Fallback bei Nichttreffer |
| benni_climate_policy | prueft u.a. auf late_morning und den nirgends existierenden Wert 'night' | Regeln feuern nie |

- **Klassifikation:** `ARCHITECTURE_DECISION`
- **Semantic Conflict:** JA - zwei Generationen im produktiven Betrieb

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: die neun Core-State-Phasen sind verbindlich (sie sind in mapping.py bereits als kanonisch erklaert und von zwei Consumern korrekt umgesetzt). Daraus folgen zwei getrennte Schritte: (1) Bindungen auf core_state umstellen (D-D05), (2) die 8er-Tabellen fachlich auf neun Werte erweitern (D-D02, D-D03, D-D04). Schritt 2 ist eine echte Produktentscheidung und darf NICHT mechanisch abgeleitet werden.

**Begründung:** Der Producer hat den Contract bereits festgelegt; offen ist nur, welche Pegel und Fenster in den drei neuen Phasen gelten sollen.

- **Betroffene Consumer:** `benni_media_policy`, `benni_climate_policy`, `benni_light_policy`, `blind_control`
- **Betroffene Domains:** Medien, Klima, Licht, Rollladen
- **Betroffene Edge-IDs:** `E-050`, `E-051`, `E-054`, `E-055`, `E-056`
- **Migrationstyp:** `ENUM_MIGRATION`
- **Change Type:** `ENUM_RENAME`, `BINDING_CHANGE`, `SEMANTIC_CHANGE`
- **Compatibility Requirement:** `MIGRATION_REQUIRED`
- **Contract Readiness:** `CONTRACT_READY`
- **Hängt ab von:** —
- **Priorität:** `P1` · **Konfidenz:** hoch
- **Benni:** `OPEN`

> Blockiert D-D02, D-D03, D-D04 und (in der Reihenfolge) D-D05 fuer media_policy.

#### `D-D02` — Welche HomePods-/Denon-Lautstaerke-Baseline gilt in midday, late_afternoon und evening?

- **Decision Level:** `L1 NORMALIZE`
- **Inputs:** `day_state`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni_media_policy | HOMEPODS_BASELINES / DENON_BASELINES kennen diese drei Werte nicht | _baseline() faellt still auf den flachen Fallback 0.35 / 0.40 zurueck - ohne Fehler, ohne Diagnose |

- **Klassifikation:** `PRODUCT_DECISION`
- **Semantic Conflict:** JA - drei von neun Phasen haben faktisch keine Baseline

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: die Tabellen um die drei Werte ergaenzen. Als Ausgangspunkt (Benni entscheidet die Zahlen): midday nahe afternoon (0.45 / 0.30), late_afternoon zwischen afternoon und evening (0.45 / 0.30), evening am bisherigen early_evening (0.40 / 0.30). Zusaetzlich sollte _baseline() einen Nichttreffer als Diagnose sichtbar machen statt still zu schlucken.

**Begründung:** Ein stiller Fallback in drei von neun Phasen ist genau die Klasse Fehler, die man im Betrieb nicht bemerkt. Die Zahlen selbst sind Geschmack und gehoeren zu Benni.

- **Betroffene Consumer:** `benni_media_policy`
- **Betroffene Domains:** Medien
- **Betroffene Edge-IDs:** `E-054`
- **Migrationstyp:** `LOCAL_FIX`
- **Change Type:** `VALUE_ONLY`, `CODE_FIX_ONLY`
- **Compatibility Requirement:** `SAME_RELEASE_REQUIRED`
- **Contract Readiness:** `POLICY_INTERNAL`
- **Hängt ab von:** `D-D01`
- **Priorität:** `P1` · **Konfidenz:** hoch (Befund); niedrig (konkrete Zahlen - reine Produktentscheidung)
- **Benni:** `OPEN`

> Muss zusammen mit D-D05 ausgeliefert werden: eine Umbindung auf core_state OHNE erweiterte Tabellen verschlechtert das Verhalten.

#### `D-D03` — Darf der Subwoofer mittags, am spaeten Nachmittag und am Abend laufen?

- **Decision Level:** `L4 ARBITRATE/GATE`
- **Inputs:** `day_state`, `lokale Uhrzeit`, `entertainment_active`, `headset_active`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni_media_policy | SUB_ALLOWED_PHASES = (late_morning, forenoon, afternoon, early_evening, late_evening) + Wanduhr-Floor 09:00 | midday, late_afternoon und evening sind NICHT enthalten -> Subwoofer bleibt aus |

- **Klassifikation:** `PRODUCT_DECISION`
- **Semantic Conflict:** JA - der Subwoofer ist mitten am Tag und am Abend gesperrt

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: midday, late_afternoon und evening aufnehmen. Fachlich war das Fenster als 'ab 09:00 bis einschliesslich spaeter Abend' gemeint; die drei Werte fallen genau in dieses Fenster und fehlen nur, weil sie zur Entstehungszeit nicht existierten.

**Begründung:** Das Lastenheft beschreibt ein zusammenhaengendes Tagesfenster. Die heutige Menge ist kein Fenster mehr, sondern ein Loch.

- **Betroffene Consumer:** `benni_media_policy`
- **Betroffene Domains:** Medien
- **Betroffene Edge-IDs:** `E-055`
- **Migrationstyp:** `LOCAL_FIX`
- **Change Type:** `VALUE_ONLY`
- **Compatibility Requirement:** `SAME_RELEASE_REQUIRED`
- **Contract Readiness:** `POLICY_INTERNAL`
- **Hängt ab von:** `D-D01`
- **Priorität:** `P1` · **Konfidenz:** hoch
- **Benni:** `OPEN`

> Der 09:00-Wanduhr-Floor bleibt davon unberuehrt und traegt die eigentliche Ruhezeit-Absicht.

#### `D-D04` — Wann soll das Bad an freien Tagen vorgeheizt werden?

- **Decision Level:** `L3 POLICY`
- **Inputs:** `day_state`, `day_context`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni_climate_policy | (frei AND day_state == late_morning) | late_morning existiert in core_state nicht -> das Frei-Tag-Fenster kann nie feuern |
| benni_climate_policy | (Werktag AND early_morning) OR late_evening | diese beiden Fenster funktionieren |

- **Klassifikation:** `BUG`
- **Semantic Conflict:** JA - eine von drei Preheat-Regeln ist tot

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: late_morning durch forenoon ersetzen (der naechste Nachfolger in der Neun-Phasen-Ordnung). Ob das fachlich der gewuenschte Zeitpunkt ist, entscheidet Benni - die Regel ueberhaupt wieder scharf zu schalten, ist unstrittig.

**Begründung:** Eine Regel, die per Konstruktion nie feuern kann, ist schlimmer als keine Regel: sie taeuscht abgedeckte Funktionalitaet vor.

- **Betroffene Consumer:** `benni_climate_policy`
- **Betroffene Domains:** Klima
- **Betroffene Edge-IDs:** `E-056`
- **Migrationstyp:** `LOCAL_FIX`
- **Change Type:** `VALUE_ONLY`
- **Compatibility Requirement:** `NONE`
- **Contract Readiness:** `OWNER_INTERNAL`
- **Hängt ab von:** `D-D01`, `D-D05`
- **Priorität:** `P2` · **Konfidenz:** hoch (Befund); mittel (Ersatzwert)
- **Benni:** `OPEN`

> Gleiches Muster wie der tote Wert 'night' in der Nachtmenge derselben Datei.

#### `D-D05` — Sollen die Legacy-Bindungen auf sensor.benni_combined_context_* abgeloest werden?

- **Decision Level:** `L0 FACT`
- **Inputs:** `bio_state`, `activity_state`, `day_state`, `day_context`, `presence_personal`, `presence_household`, `presence_band`, `presence_transition`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni_climate_policy | bindet AUSSCHLIESSLICH auf die Combined-IDs (8 von 8 Kontextwerten) | gesamte Heizlogik haengt an Legacy-IDs |
| plug_policy_engine | presence, bio, activity ueber Combined-IDs | Away-Cuts und Schlaf-Cuts haengen daran |
| benni_media_policy | day_state ueber die Combined-ID | Volume-Baseline und Subwoofer-Fenster haengen daran |
| benni_light_policy | Cutover vollzogen, migration.py dokumentiert ihn | bindet direkt auf core_state |
| benni-core-state mapping.py | fuehrt alle diese IDs als legacy_references mit legacy_resolution=replace_after_cutover | der Zielpfad ist bereits definiert |

- **Klassifikation:** `LEGACY`
- **Semantic Conflict:** JA - drei Module haengen an einer als abloesbar markierten Zwischenschicht

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: ja, ablosen - aber modulweise und in dieser Reihenfolge: (1) plug_policy_engine (einfachste Bindung), (2) climate_policy zusammen mit D-B08/D-B09/D-D04, (3) media_policy ZWINGEND erst nach D-D02 und D-D03. Vorher ist eine Live-Pruefung noetig, ob die Combined-Entities ueberhaupt noch existieren und Werte liefern.

**Begründung:** Der Cutover-Pfad ist in mapping.py definiert und von light_policy bereits vorgemacht. Das Risiko liegt nicht im Cutover selbst, sondern in der Reihenfolge: media_policy wuerde ohne erweiterte Tabellen in drei Tagesphasen schlechter werden.

- **Betroffene Consumer:** `benni_climate_policy`, `plug_policy_engine`, `benni_media_policy`
- **Betroffene Domains:** Klima, Steckdose, Medien
- **Betroffene Edge-IDs:** `E-022`, `E-023`, `E-042`, `E-054`, `E-055`, `E-067`, `E-073`, `E-075`, `E-091`, `E-095`, `E-102`, `E-200`, `E-201`, `E-202`, `E-203`, `E-204`
- **Migrationstyp:** `CONTRACT_CUTOVER`
- **Change Type:** `BINDING_CHANGE`, `LEGACY_REMOVAL`
- **Compatibility Requirement:** `MIGRATION_REQUIRED`
- **Contract Readiness:** `CONTRACT_READY`
- **Hängt ab von:** `D-D01`, `D-D02`, `D-D03`
- **Priorität:** `P1` · **Konfidenz:** hoch (Code); niedrig (Live-Existenz der Combined-Entities)
- **Benni:** `OPEN`

> Groesste einzelne Aufraeumaktion des Audits: 16 Kanten. Vor allem aber: solange climate_policy an einer moeglicherweise toten ID haengt, ist die gesamte Heizlogik in ihrem Verhalten unbestimmt.

#### `D-D06` — Gilt der Abend-Komfortzuschlag der Heizung noch, nachdem free_time aufgefaechert wurde?

- **Decision Level:** `L3 POLICY`
- **Inputs:** `activity_state`, `day_state`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni_climate_policy | activity == free_time AND day_state == early_night AND kein Sommermonat | abends laeuft heute meist entertainment, gaming oder music - free_time kommt kaum noch vor |

- **Klassifikation:** `DRIFT`
- **Semantic Conflict:** JA - die Regel wurde durch eine Producer-Aenderung faktisch entwertet

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: die Menge auf die Nachfolger von free_time erweitern (entertainment, music, gaming, pc_active, free_time, idle) - also auf 'Nutzer ist abends zuhause und nicht im Arbeits- oder Haushaltsmodus'.

**Begründung:** Activity v1 hat den free_time-Sammeltopf bewusst aufgefaechert. Consumer, die auf den Sammeltopf gebaut haben, wurden dabei nicht mitgezogen - das ist derselbe Mechanismus wie beim Licht-Preset-Gate (D-D08).

- **Betroffene Consumer:** `benni_climate_policy`
- **Betroffene Domains:** Klima
- **Betroffene Edge-IDs:** `E-201`
- **Migrationstyp:** `LOCAL_FIX`
- **Change Type:** `VALUE_ONLY`, `SEMANTIC_CHANGE`
- **Compatibility Requirement:** `NONE`
- **Contract Readiness:** `OWNER_INTERNAL`
- **Hängt ab von:** `D-D05`
- **Priorität:** `P2` · **Konfidenz:** mittel
- **Benni:** `OPEN`

> Die eigentliche Frage dahinter: welche Activity-Werte bedeuten 'Nutzer ist entspannt zuhause'? Sie stellt sich in D-D08 noch einmal.

#### `D-D07` — Sollen work_home und work_away produzierbar gemacht oder als Werte zurueckgezogen werden?

- **Decision Level:** `L2 CLASSIFY`
- **Inputs:** `activity_state`, `homeoffice_ping`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni-core-state | work_home braucht CONF_HOMEOFFICE_PING, das bewusst ungebunden ist; work_away ist nicht in ACTIVITY_PRECEDENCE | beide Werte werden auf main nie erzeugt |
| benni_light_policy | work_home -> CCT 5000K | toter Zweig |
| benni_notification_router | work_home -> Unterdrueckung | toter Zweig |
| benni_media_policy | work_home/work_away -> Boost-Block | toter Zweig |

- **Klassifikation:** `PRODUCT_DECISION`
- **Semantic Conflict:** NEIN - konsistent, aber unerreichbar

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: entweder eine Homeoffice-Quelle binden (dann werden drei implementierte Zweige lebendig) oder die Werte als 'geplant, nicht erzeugt' im Katalog kennzeichnen. Nicht empfohlen: die Consumer-Zweige entfernen - sie sind korrekt und billig zu halten.

**Begründung:** Drei Domaenen haben Verhalten fuer einen Zustand implementiert, den niemand erzeugt. Das ist kein Fehler, aber eine offene Produktluecke.

- **Betroffene Consumer:** `benni_light_policy`, `benni_notification_router`, `benni_media_policy`
- **Betroffene Domains:** Licht, Benachrichtigung, Medien
- **Betroffene Edge-IDs:** `E-031`, `E-037`, `E-040`
- **Migrationstyp:** `DOCUMENT_ONLY`
- **Change Type:** `SEMANTIC_CHANGE`
- **Compatibility Requirement:** `NONE`
- **Contract Readiness:** `CONTRACT_AFTER_DECISION`
- **Hängt ab von:** —
- **Priorität:** `P3` · **Konfidenz:** hoch
- **Benni:** `OPEN`

> core_state dokumentiert die Nichtbindung ausdruecklich ('work_home ist geplant, nicht faked') - das ist bereits die saubere Variante.

#### `D-D08` — Welche Activity-Werte duerfen einen Szenen-Preset im Licht treiben, nachdem free_time aufgefaechert wurde?

- **Decision Level:** `L4 ARBITRATE/GATE`
- **Inputs:** `activity_state`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni_light_policy | ACTIVITY_PRESET_DRIVING = {free_time, idle} | seit der Auffaecherung feuert der Zweig deutlich seltener; music und entertainment fallen heraus |
| benni_light_policy | GAMING_ACTIVITY_STATES = {gaming, free_time, idle} - wurde nachgezogen | das Gaming-Gate ist bereits an die neuen Werte angepasst, das Preset-Gate nicht |

- **Klassifikation:** `DRIFT`
- **Semantic Conflict:** teilweise - innerhalb desselben Moduls wurde ein Gate nachgezogen, das andere nicht

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: pruefen, ob music (und ggf. entertainment) in ACTIVITY_PRESET_DRIVING gehoeren. Musik zu hoeren war frueher free_time und hat Presets getrieben; heute tut es das nicht mehr, ohne dass das entschieden wurde.

**Begründung:** Dasselbe Muster wie D-D06: der Producer hat aufgefaechert, ein Teil der Consumer wurde nachgezogen, ein anderer nicht.

- **Betroffene Consumer:** `benni_light_policy`
- **Betroffene Domains:** Licht
- **Betroffene Edge-IDs:** `E-033`, `E-034`
- **Migrationstyp:** `LOCAL_FIX`
- **Change Type:** `VALUE_ONLY`, `SEMANTIC_CHANGE`
- **Compatibility Requirement:** `NONE`
- **Contract Readiness:** `POLICY_INTERNAL`
- **Hängt ab von:** —
- **Priorität:** `P3` · **Konfidenz:** mittel
- **Benni:** `OPEN`

> Der fruehere Befund 'Gaming-Ring feuert nie' ist auf main bereits behoben - dieser Nachbarzweig ist es nicht.

---

### Familie: MEDIA CONTEXT / DEVICE / ACTIVITY

#### `D-M01` — Was bedeutet media_device - Geraete-IDENTITAET, AUDIO-ROUTING oder SCREEN-INTENT?

- **Decision Level:** `L2 CLASSIFY`
- **Inputs:** `media_device`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni_media_apply (DENON_CONSUMER_DEVICES) | Audio-Routing: welche Geraete brauchen den Denon als Senke | Nachlauf-Off blockiert |
| benni_media_apply (SCREEN_DEVICES) | Screen-Intent: tv/appletv brauchen den Bildschirm | TV wird eingeschaltet (WoL) |
| benni_media_policy (is_pc_gaming) | Audio-Pfad: pc bedeutet Headset statt Raumspeaker | HomePods spielen weiter, Denon bleibt aus |
| benni_light_policy | Identitaet: ist MEINE konfigurierte Gaming-Quelle aktiv | Gaming-Look je Quelle; media_device == tv -> Cinema |
| blind_control | Screen-Klasse: pc_platforms vs. tv_platforms | glare_pc / glare_tv; bei Widerspruch ScreenEvidenceConflict |

- **Klassifikation:** `ARCHITECTURE_DECISION`
- **Semantic Conflict:** JA - ein Enum traegt drei fachliche Bedeutungen

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: media_device bleibt das beschreibende Primaergeraet (Identitaet). Die beiden abgeleiteten Bedeutungen werden als eigene Felder publiziert - am sinnvollsten als zusaetzliche Felder des bestehenden activity_context-Feeds, nicht als neue Entities: (a) audio_sink_consumer (bool oder Enum), (b) screen_class (none|tv|pc|other, siehe D-M02).

**Begründung:** Solange die drei Bedeutungen in einem Wert stecken, muss jeder Consumer sie neu ableiten - und jeder tut es anders. Die appletv-Luecke (D-M03) ist die direkte Folge davon.

- **Betroffene Consumer:** `benni_media_apply`, `benni_media_policy`, `benni_light_policy`, `blind_control`
- **Betroffene Domains:** Medien, Licht, Rollladen
- **Betroffene Edge-IDs:** `E-124`, `E-135`, `E-136`, `E-137`, `E-138`, `E-139`, `E-140`
- **Migrationstyp:** `SEMANTIC_SPLIT`
- **Change Type:** `NEW_CONTRACT`, `SEMANTIC_CHANGE`
- **Compatibility Requirement:** `DUAL_PUBLISH`
- **Contract Readiness:** `CONTRACT_AFTER_SPLIT`
- **Hängt ab von:** —
- **Priorität:** `P1` · **Konfidenz:** hoch
- **Benni:** `OPEN`

> Blockiert D-M02 und D-M03. media_device selbst bleibt unveraendert - die Aufteilung ist additiv.

#### `D-M02` — Soll es eine publizierte screen_class geben, statt dass vier Consumer 'welche Bildschirmart laeuft' unabhaengig ableiten?

- **Decision Level:** `L2 CLASSIFY`
- **Inputs:** `media_context`, `media_device`, `gaming_platform`, `entertainment_active`, `activity_state`, `activity_context`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni_light_policy | media_context in {tv,streaming} OR media_device == tv, plus entertainment_stable | Cinema-Look |
| blind_control | eigener Adapter _convert_activity mit eigener Vorrangregel ueber 6 Felder | pc\|tv\|screen\|none -> glare_* |
| plug_policy_engine | media_context in {movie,streaming,tv,video} OR (gaming AND gaming_source==tv) | Bias Light |
| einhornzentrale import.yaml | eigene Combined-Projektion ueber entertainment + tv | Bias Light (zweite Wahrheit) |

- **Klassifikation:** `ARCHITECTURE_DECISION`
- **Semantic Conflict:** JA - vier Antworten auf dieselbe Frage aus vier Feldkombinationen

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: ja. screen_class (none|tv|pc|other) als Feld des activity_context-Feeds, mit einer Evidenzliste im Attribut. Das vorhandene Vorbild ist blind_controls _convert_activity - es ist die vollstaendigste Ableitung und kann als Referenzimplementierung dienen, wandert dann aber in den Producer.

**Begründung:** Der Media-Stack weiss als einziger, welches Geraet gerade den Bildschirm treibt. Dass drei fremde Domaenen das aus Rohfeldern rekonstruieren, ist die Ursache der Uneinheitlichkeit.

- **Betroffene Consumer:** `benni_light_policy`, `blind_control`, `plug_policy_engine`, `einhornzentrale`
- **Betroffene Domains:** Medien, Licht, Rollladen, Steckdose
- **Betroffene Edge-IDs:** `E-035`, `E-036`, `E-125`, `E-126`, `E-128`, `E-140`, `E-148`, `E-155`, `E-159`, `E-160`
- **Migrationstyp:** `SEMANTIC_SPLIT`
- **Change Type:** `NEW_CONTRACT`
- **Compatibility Requirement:** `DUAL_PUBLISH`
- **Contract Readiness:** `CONTRACT_AFTER_SPLIT`
- **Hängt ab von:** `D-M01`
- **Priorität:** `P2` · **Konfidenz:** hoch
- **Benni:** `OPEN`

> Loest zugleich D-M03 und D-M08 auf.

#### `D-M03` — Soll appletv in blind_control als tv-Plattform gelten?

- **Decision Level:** `L1 NORMALIZE`
- **Inputs:** `media_device`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni_media_apply | appletv in SCREEN_DEVICES | TV wird fuer Apple TV eingeschaltet |
| blind_control | tv_platforms = {ps5, playstation, xbox, switch, tv}; appletv fehlt | ein Apple-TV-Abend liefert KEINE explizite Screen-Klasse -> kein glare_tv ueber diesen Pfad |

- **Klassifikation:** `BUG`
- **Semantic Conflict:** JA - dasselbe Geraet ist in einer Domaene ein Bildschirm, in der anderen nicht

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: appletv in tv_platforms aufnehmen. Kurzfristiger Einzeiler; mittelfristig durch D-M02 ohnehin abgeloest.

**Begründung:** Apple TV ist im Haushalt der Hauptweg fuer Streaming. Dass ausgerechnet dabei der Blendschutz nicht ueber die explizite Evidenz greift, ist eine Luecke, kein Entwurf.

- **Betroffene Consumer:** `blind_control`
- **Betroffene Domains:** Rollladen
- **Betroffene Edge-IDs:** `E-140`
- **Migrationstyp:** `LOCAL_FIX`
- **Change Type:** `VALUE_ONLY`
- **Compatibility Requirement:** `NONE`
- **Contract Readiness:** `OWNER_INTERNAL`
- **Hängt ab von:** —
- **Priorität:** `P1` · **Konfidenz:** hoch
- **Benni:** `OPEN`

> Teilweise abgefedert: entertainment_active setzt tv_active auch ohne explizite Klasse - aber erst auf der schwaecheren Evidenzstufe.

#### `D-M04` — Soll der activity_context-Feed Quality und Freshness selbst publizieren, statt dass Core State ueber last_updated schaetzt?

- **Decision Level:** `L4 ARBITRATE/GATE`
- **Inputs:** `activity_context`, `Zeitstempel des Feed-Sensors`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni_media_state (Producer) | publiziert reason und hold_strength, aber KEIN quality-/freshness-Feld | Consumer muss schaetzen |
| benni-core-state | consumer-seitiges Alters-Gate 1800 s auf last_updated; explizite Marker haetten Vorrang | ein eingefrorener Feed-Wert gilt bis zu 30 Minuten als frisch |

- **Klassifikation:** `ARCHITECTURE_DECISION`
- **Semantic Conflict:** JA - Luecke, nicht Widerspruch

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: ja. Der Feed bekommt quality und freshness als Attribute (das Consumer-Gate liest sie bereits bevorzugt aus - es muss also nichts umgebaut werden, nur befuellt). Das ist der billigste und zugleich nuetzlichste Contract-Schritt des gesamten Audits. Ob das als Core Contract (media.activity.v1) oder als Attribut am bestehenden Sensor geschieht, ist eine getrennte Frage.

**Begründung:** Es gibt genau eine Producer-zu-Producer-Kante im Stack, und ausgerechnet sie hat kein Qualitaetssignal. Das Vorbild liegt im selben Repo: activity_decision an core_states activity_state liefert Gewinner, Kandidaten, Freshness je Quelle, quality_status, degraded_reason und Zeitstempel - ganz ohne Contract-Layer.

- **Betroffene Consumer:** `benni-core-state`
- **Betroffene Domains:** Medien, Person
- **Betroffene Edge-IDs:** `E-145`, `E-146`
- **Migrationstyp:** `CONTRACT_CUTOVER`
- **Change Type:** `NEW_CONTRACT`, `CONTRACT_VERSION`
- **Compatibility Requirement:** `DUAL_READ`
- **Contract Readiness:** `CONTRACT_READY`
- **Hängt ab von:** —
- **Priorität:** `P1` · **Konfidenz:** hoch
- **Benni:** `OPEN`

> core-contracts@main dokumentiert media.activity.v1 bereits als Beispiel - es existiert aber weder Code noch Registry-Eintrag.

#### `D-M05` — Wer besitzt die Hold-Staerke - der Media-Feed oder Core State?

- **Decision Level:** `L4 ARBITRATE/GATE`
- **Inputs:** `activity_context (attr hold_strength)`, `activity_state`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni_media_state | publiziert hold_strength (hard fuer private_time/gaming, soft fuer entertainment/music, none fuer idle) | wird von Core State nicht gelesen |
| benni-core-state | eigenes ACTIVITY_HOLD_STRENGTH (high/mid/none) plus SOFT_HOLD_ACTIVITIES | entscheidet den Presence-Hold ausschliesslich selbst |

- **Klassifikation:** `ARCHITECTURE_DECISION`
- **Semantic Conflict:** teilweise - zwei parallele Modelle, heute ohne Widerspruch im Ergebnis

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: Core State behaelt die Hold-Staerke. Die Hold-Frage ist 'darf lokale Aktivitaet ein rohes abwesend ueberstimmen' - das ist eine Presence-Entscheidung, keine Medienentscheidung. Konsequenz: hold_strength im Feed wird als reines Diagnose-/Anzeigefeld gekennzeichnet oder entfernt.

**Begründung:** Solange beide Modelle existieren und nur eines wirkt, sieht ein Leser im Cockpit eine Staerke, die nichts bewirkt.

- **Betroffene Consumer:** `benni-core-state`
- **Betroffene Domains:** Medien, Person
- **Betroffene Edge-IDs:** `E-043`, `E-044`, `E-147`
- **Migrationstyp:** `DOCUMENT_ONLY`
- **Change Type:** `SEMANTIC_CHANGE`
- **Compatibility Requirement:** `NONE`
- **Contract Readiness:** `OWNER_INTERNAL`
- **Hängt ab von:** `D-M04`
- **Priorität:** `P2` · **Konfidenz:** mittel
- **Benni:** `OPEN`

> Falls der Feed spaeter ein Contract wird (D-M04), sollte hold_strength nicht Teil des Contracts sein.

#### `D-M06` — Sind die als 'nur Debug' deklarierten Attribute am activity_state-Sensor Teil des Consumer-Contracts?

- **Decision Level:** `L0 FACT`
- **Inputs:** `activity_state (attrs media_context, media_device, entertainment_active, gaming_platform, media_activity_context)`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni-core-state | Code-Kommentar: 'Debug-Echo aus media_state (treiben die Entscheidung NICHT mehr)' | als entscheidungsfrei deklariert |
| blind_control | liest genau diese Attribute als Evidenz fuer die Screen-Klasse | glare_pc/glare_tv/glare_general haengen produktiv daran |

- **Klassifikation:** `ARCHITECTURE_DECISION`
- **Semantic Conflict:** JA - ein als Debug gekennzeichnetes Feld ist Produktiv-Input einer anderen Domaene

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: entweder als Teil des Contracts anerkennen und entsprechend kennzeichnen (Attribute duerfen dann nicht ohne Ankuendigung entfallen), oder blind_control auf den Media-Feed direkt binden. Letzteres ist sauberer und faellt mit D-M02 zusammen.

**Begründung:** 'Debug' ist eine Zusage, dass man das Feld jederzeit aendern darf. Diese Zusage stimmt heute nicht - ein Entfernen wuerde den Blendschutz stillschweigend deaktivieren.

- **Betroffene Consumer:** `blind_control`, `benni-core-state`
- **Betroffene Domains:** Person, Rollladen
- **Betroffene Edge-IDs:** `E-127`, `E-128`, `E-140`, `E-141`, `E-148`, `E-158`, `E-159`
- **Migrationstyp:** `DOCUMENT_ONLY`
- **Change Type:** `SEMANTIC_CHANGE`, `BINDING_CHANGE`
- **Compatibility Requirement:** `DUAL_READ`
- **Contract Readiness:** `CONTRACT_AFTER_DECISION`
- **Hängt ab von:** `D-M02`
- **Priorität:** `P2` · **Konfidenz:** hoch
- **Benni:** `OPEN`

> Zweiter Hop derselben Media-Wahrheit: media_state -> core_state (Attribut) -> blind_control.

#### `D-M07` — Ist audio_owner eine Soll-Entscheidung oder eine Ist-Beobachtung?

- **Decision Level:** `L3 POLICY`
- **Inputs:** `media_context`, `media_device`, `homepods_state`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni_media_policy | Prioritaet des Soll-Stacks aus dem Kontext PLUS die Beobachtung, dass die HomePods gerade spielen | audio_owner mischt beides; AUDIO_OWNER_SLEEP ist definiert, wird nie emittiert |
| benni_media_apply | liest audio_owner == private_stack als Private-Indikator | Private-Exit-Flanke -> Denon-Off-Delay |
| benni_media (Umbrella) | zeigt audio_scenario (reine Soll-Wahrheit) als Hero | die saubere Trennung existiert bereits daneben |

- **Klassifikation:** `ARCHITECTURE_DECISION`
- **Semantic Conflict:** JA - der Name sagt Entscheidung, der Wert ist teilweise Beobachtung

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: der bereits in control#7 vorgeschlagenen Namenskonvention folgen - Ist/Beobachtung als *_current bzw. *_observed, Soll als *_target bzw. *_desired, Freigabe als *_allowed. Konkret: audio_scenario ist bereits die Soll-Wahrheit; audio_owner wird zur reinen Ist-Beobachtung (audio_owner_current) reduziert oder entsprechend umbenannt. AUDIO_OWNER_SLEEP entfaellt.

**Begründung:** Genau dieser Namensfehler hat laut control#7 bereits zu unmoeglichen Starts gefuehrt (Apply las eine Beobachtung als Start-Vorbedingung). Der Befund ist damit unabhaengig belegt.

- **Betroffene Consumer:** `benni_media_apply`, `benni_media`
- **Betroffene Domains:** Medien
- **Betroffene Edge-IDs:** `E-176`, `E-177`
- **Migrationstyp:** `SEMANTIC_SPLIT`
- **Change Type:** `ENUM_RENAME`, `SEMANTIC_CHANGE`, `LEGACY_REMOVAL`
- **Compatibility Requirement:** `DUAL_PUBLISH`
- **Contract Readiness:** `POLICY_INTERNAL`
- **Hängt ab von:** —
- **Priorität:** `P2` · **Konfidenz:** hoch
- **Benni:** `OPEN`

> Direkter Anschluss an den Naming-Input vom 11.09.2026 in control#7.

#### `D-M08` — Soll es eine oder zwei Bias-Light-Wahrheiten geben?

- **Decision Level:** `L3 POLICY`
- **Inputs:** `media_context`, `gaming_source`, `entertainment_active`, `tv_active`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| plug_policy_engine | _decide_bias_light aus media_context + gaming_source + tv_active-Gegenprobe | DESIRED_ON/OFF fuer die Steckdose |
| einhornzentrale import.yaml | media_bias_light_should_be_on aus sensor.benni_combined_media_entertainment_active + tv | zweiter Combined-Output fuer dieselbe Frage |

- **Klassifikation:** `LEGACY`
- **Semantic Conflict:** JA - DUPLICATE_CALCULATION

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: eine Wahrheit, und zwar plug_policy_engine (es ist der Owner der Steckdose und hat die tv_active-Gegenprobe aus control#35). Den Combined in import.yaml zurueckbauen - abhaengig davon, ob er live ueberhaupt aktiv ist (D-S04).

**Begründung:** Zwei Wahrheiten fuer dieselbe Steckdose sind der klassische Weg zu Flapping, sobald beide eine Aktion ausloesen duerfen.

- **Betroffene Consumer:** `plug_policy_engine`, `einhornzentrale`
- **Betroffene Domains:** Steckdose, YAML-Konfiguration
- **Betroffene Edge-IDs:** `E-126`, `E-160`
- **Migrationstyp:** `LOCAL_FIX`
- **Change Type:** `LEGACY_REMOVAL`
- **Compatibility Requirement:** `UNKNOWN`
- **Contract Readiness:** `NOT_A_CONTRACT`
- **Hängt ab von:** `D-S04`
- **Priorität:** `P3` · **Konfidenz:** mittel
- **Benni:** `OPEN`

> Der Combined bindet zusaetzlich auf eine Legacy-Entertainment-ID, nicht auf media_state.

#### `D-M09` — Sollen die toten media_context-Werte movie und video entfernt werden?

- **Decision Level:** `L0 FACT`
- **Inputs:** `media_context`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| plug_policy_engine | want_on = media in {movie, streaming, tv, video} ... | movie und video werden von media_state nie emittiert |

- **Klassifikation:** `LEGACY`
- **Semantic Conflict:** NEIN

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: entfernen. Anders als die bio-Aliase (D-B19) haben sie keinen Nutzen als Dual-Read-Reserve - es steht kein media_context-Rename im Raum.

**Begründung:** Reine Toolbox-Altlast; erhoeht nur den Leseaufwand.

- **Betroffene Consumer:** `plug_policy_engine`
- **Betroffene Domains:** Steckdose
- **Betroffene Edge-IDs:** `E-126`
- **Migrationstyp:** `LOCAL_FIX`
- **Change Type:** `LEGACY_REMOVAL`
- **Compatibility Requirement:** `NONE`
- **Contract Readiness:** `NOT_A_CONTRACT`
- **Hängt ab von:** —
- **Priorität:** `P3` · **Konfidenz:** hoch
- **Benni:** `OPEN`

#### `D-M10` — Braucht der Title-Classifier-Enum 3 (gaming_grind_preemptible) einen eigenen Subcontext-Wert?

- **Decision Level:** `L2 CLASSIFY`
- **Inputs:** `media_subcontext`, `ps5_enum`, `pc_enum`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni_media_state | ENUM_GAME_GRIND_PREEMPTIBLE = 3 existiert als Konstante | es gibt keinen SUB_GAME_*-Wert dafuer |
| benni_media_policy | is_grind prueft subcontext == gaming_grind | Enum 3 kann die Grind-Sonderbehandlung nicht ausloesen |

- **Klassifikation:** `DRIFT`
- **Semantic Conflict:** teilweise - Luecke zwischen Classifier und Subcontext-Enum

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: klaeren, ob Enum 3 fachlich wie gaming_grind behandelt werden soll (dann auf denselben Subcontext mappen) oder ob ein eigener Wert noetig ist (dann auch media_policy erweitern).

**Begründung:** Ein Classifier-Wert ohne Gegenstueck im Consumer-Enum faellt still in den Default-Zweig.

- **Betroffene Consumer:** `benni_media_state`, `benni_media_policy`
- **Betroffene Domains:** Medien
- **Betroffene Edge-IDs:** `E-123`
- **Migrationstyp:** `LOCAL_FIX`
- **Change Type:** `VALUE_ONLY`
- **Compatibility Requirement:** `SAME_RELEASE_REQUIRED`
- **Contract Readiness:** `OWNER_INTERNAL`
- **Hängt ab von:** —
- **Priorität:** `P3` · **Konfidenz:** mittel
- **Benni:** `OPEN`

> Nicht Teil des Cross-Domain-Scopes, aber beim Lesen von media_state aufgefallen.

---

### Familie: SHARED / DEBUG OUTPUTS

#### `D-S01` — Soll die YAML-Doppelberechnung von presence_preheat_active zurueckgebaut werden?

- **Decision Level:** `L2 CLASSIFY`
- **Inputs:** `presence_band`, `presence_preheat_active`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni-core-state | publiziert binary_sensor.*_presence_preheat_active mit Quelle, Startzeit und Maximaldauer | vollstaendiger Output inkl. Diagnose |
| einhornzentrale import.yaml | context_presence_preheat_active = (band == preheat) | Zweitberechnung ohne die Zeitbegrenzung des Owners |

- **Klassifikation:** `LEGACY`
- **Semantic Conflict:** JA - DUPLICATE_CALCULATION mit abweichender Semantik (Owner hat ein Zeitfenster, der Combined nicht)

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: zurueckbauen und Consumer auf den Owner-Output binden - abhaengig davon, ob der Combined live ist (D-S04).

**Begründung:** Die YAML-Variante ist nicht nur doppelt, sie ist auch fachlich aermer: sie kennt die Preheat-Maximaldauer nicht.

- **Betroffene Consumer:** `einhornzentrale`
- **Betroffene Domains:** YAML-Konfiguration
- **Betroffene Edge-IDs:** `E-096`
- **Migrationstyp:** `LOCAL_FIX`
- **Change Type:** `LEGACY_REMOVAL`, `BINDING_CHANGE`
- **Compatibility Requirement:** `UNKNOWN`
- **Contract Readiness:** `NOT_A_CONTRACT`
- **Hängt ab von:** `D-S04`
- **Priorität:** `P3` · **Konfidenz:** hoch (Befund); niedrig (Live-Wirksamkeit)
- **Benni:** `OPEN`

> core_states presence_preheat_active hat auf main ebenfalls keinen direkten Consumer.

#### `D-S02` — Sollen master_context und live_status Contract-Status bekommen?

- **Decision Level:** `L0 FACT`
- **Inputs:** `master_context`, `live_status`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| einhornzentrale toolbox_readiness.yaml | prueft master_context nur auf Existenz | Readiness-Flag |
| (live_status) | kein Produktiv-Consumer auf origin/main | reine Anzeige |

- **Klassifikation:** `NO_PROBLEM`
- **Semantic Conflict:** NEIN

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: nein, beide bleiben NOT_A_CONTRACT. Wichtig ist nur das Bewusstsein, dass master_context eine gepunktete Verkettung aller fuenf Kernwerte ist - jeder Enum-Rename aendert diesen String implizit mit.

**Begründung:** Beide sind Anzeigeprojektionen ohne eigene Wahrheit. Ein Contract wuerde nur Pflege kosten.

- **Betroffene Consumer:** `einhornzentrale`
- **Betroffene Domains:** Betrieb
- **Betroffene Edge-IDs:** `E-185`, `E-186`
- **Migrationstyp:** `NO_CHANGE`
- **Change Type:** `CODE_FIX_ONLY`
- **Compatibility Requirement:** `NONE`
- **Contract Readiness:** `NOT_A_CONTRACT`
- **Hängt ab von:** —
- **Priorität:** `P3` · **Konfidenz:** hoch
- **Benni:** `OPEN`

#### `D-S03` — Soll das action-Enum zwischen media_policy und media_apply geteilt statt kopiert werden?

- **Decision Level:** `L5 EXECUTE`
- **Inputs:** `action`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni_media_policy | definiert none\|pause_homepods\|resume_homepods\|start_radio | Producer |
| benni_media_apply | haelt eine KOPIE der Konstanten (kein Import) und ergaenzt turn_off_denon | Consumer mit Eigenerweiterung |

- **Klassifikation:** `ARCHITECTURE_DECISION`
- **Semantic Conflict:** teilweise - bewusste Entkopplung, aber ohne Versionierung

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: die Kopie beibehalten (der bewusste Verzicht auf Cross-Modul-Imports ist eine tragende Architekturregel des Stacks), aber die Kopplung sichtbar machen - etwa durch einen Contract-Versionsstring in beiden Modulen und einen Test, der die Wertemengen vergleicht.

**Begründung:** Die Entkopplung ist gewollt und richtig. Nicht gewollt ist, dass eine Enum-Erweiterung auf der einen Seite auf der anderen unbemerkt bleibt.

- **Betroffene Consumer:** `benni_media_apply`
- **Betroffene Domains:** Medien
- **Betroffene Edge-IDs:** `E-175`
- **Migrationstyp:** `DOCUMENT_ONLY`
- **Change Type:** `CONTRACT_VERSION`
- **Compatibility Requirement:** `SAME_RELEASE_REQUIRED`
- **Contract Readiness:** `POLICY_INTERNAL`
- **Hängt ab von:** —
- **Priorität:** `P3` · **Konfidenz:** mittel
- **Benni:** `OPEN`

> Gilt sinngemaess fuer alle KOPIERTEN const-Bloecke zwischen den Media-Modulen.

#### `D-S04` — Ist benni_core_devices/import.yaml live wirksam oder ein reines Config-Artefakt?

- **Decision Level:** `L0 FACT`
- **Inputs:** `alle Combined-Definitionen in import.yaml`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| einhornzentrale import.yaml | definiert u.a. context_bio_wake_candidate, context_presence_preheat_active, media_bias_light_should_be_on | Wirksamkeit unbekannt - laut Betriebsstand fehlt der bulk_import-Service |

- **Klassifikation:** `DRIFT`
- **Semantic Conflict:** unklar

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: einmalig live pruefen, ob die in import.yaml definierten Combineds als Entities existieren. Das Ergebnis entscheidet, ob D-B14, D-M08 und D-S01 ueberhaupt Arbeit sind oder nur Dokumentation.

**Begründung:** Vier Decisions dieses Reviews haengen an dieser einen Tatsachenfrage. Sie ist read-only in wenigen Minuten beantwortbar.

- **Betroffene Consumer:** `einhornzentrale`
- **Betroffene Domains:** YAML-Konfiguration, Betrieb
- **Betroffene Edge-IDs:** `E-024`, `E-092`, `E-096`, `E-130`, `E-160`
- **Migrationstyp:** `DOCUMENT_ONLY`
- **Change Type:** `CODE_FIX_ONLY`
- **Compatibility Requirement:** `UNKNOWN`
- **Contract Readiness:** `NOT_A_CONTRACT`
- **Hängt ab von:** —
- **Priorität:** `P2` · **Konfidenz:** niedrig (das ist ja die Frage)
- **Benni:** `OPEN`

> Billigste Klaerung mit der groessten Hebelwirkung auf die P3-Liste.

#### `D-S05` — Was geschieht mit publizierten States ohne jeden Consumer?

- **Decision Level:** `L0 FACT`
- **Inputs:** `presence_effective_transition`, `presence_preheat_active`, `live_status`, `presence_away`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni-core-state | vier publizierte Outputs ohne gefundenen Produktiv-Consumer auf main | presence_away ist der einzige davon mit Kanonik-Anspruch |

- **Klassifikation:** `NO_PROBLEM`
- **Semantic Conflict:** teilweise

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: nicht entfernen. live_status und presence_effective_transition sind Anzeige- bzw. Diagnoseprojektionen und billig. presence_away ist ein Sonderfall und gehoert zu D-P01. presence_preheat_active sollte Consumer bekommen statt in YAML nachgebaut zu werden (D-S01).

**Begründung:** Ein Output ohne Consumer ist nicht automatisch falsch - aber einer, der sich selbst als kanonisch bezeichnet, schon.

- **Betroffene Consumer:** `-`
- **Betroffene Domains:** Person
- **Betroffene Edge-IDs:** `E-076`, `E-096`, `E-186`
- **Migrationstyp:** `DOCUMENT_ONLY`
- **Change Type:** `CODE_FIX_ONLY`
- **Compatibility Requirement:** `NONE`
- **Contract Readiness:** `OWNER_INTERNAL`
- **Hängt ab von:** `D-P01`, `D-S01`
- **Priorität:** `P3` · **Konfidenz:** hoch
- **Benni:** `OPEN`

---

### Familie: WAKE / TRANSITIONS

#### `D-W01` — Wer besitzt die Weckentscheidung - ha_wake_planner oder Core State?

- **Decision Level:** `L2 CLASSIFY`
- **Inputs:** `wake_state`, `next_wake`, `wake_needed`, `holiday_active`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| ha_wake_planner | publiziert alle vier Werte | heutiger Producer |
| benni-core-state | publiziert eigene Werte unter sensor.*_core_state_wake_* und vergleicht sie im Shadow | zweite Publikation derselben Entscheidung |
| benni-core-state mapping.py | fuehrt alle vier als STATUS_PLANNED mit Ziel core_state | Zielrichtung ist bereits festgelegt |
| benni_media_policy | liest wake_needed als Flanke | manual_stop-Reset |

- **Klassifikation:** `ARCHITECTURE_DECISION`
- **Semantic Conflict:** JA - Doppelpublikation, aber mit bereits definierter Zielrichtung

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: der in mapping.py festgelegte Ziel-Owner (Core State) ist plausibel und sollte bestaetigt werden. Offen ist nicht die Richtung, sondern das Freigabe-Gate und die Reihenfolge - dafuer existieren bereits control#27 (Shadow-Paritaet), control#28 (Cutover-Gate) und control#29 (Repo-Archivierung).

**Begründung:** Dieser Audit liefert dazu keine neue Evidenz; er bestaetigt nur, dass die Doppelpublikation weiter besteht und dass core_state den alten Planner heute nur noch als Fallback-Eingang fuehrt.

- **Betroffene Consumer:** `benni-core-state`, `benni_media_policy`
- **Betroffene Domains:** Zeit, Person, Medien
- **Betroffene Edge-IDs:** `E-115`, `E-116`, `E-117`
- **Migrationstyp:** `CONTRACT_CUTOVER`
- **Change Type:** `OWNER_CHANGE`, `BINDING_CHANGE`
- **Compatibility Requirement:** `DUAL_PUBLISH`
- **Contract Readiness:** `CONTRACT_AFTER_DECISION`
- **Hängt ab von:** —
- **Priorität:** `P2` · **Konfidenz:** hoch
- **Benni:** `OPEN`

> Bereits als eigene Issue-Flotte erfasst (control#27/#28/#29/#30) - hier nur als Abhaengigkeit von D-W02 gefuehrt.

#### `D-W02` — Was ist der kanonische Wake-TRIGGER - bio_state == waking, die bio-Flanke nach awake, oder ein geplanter Weckzeitpunkt (wake_needed)?

- **Decision Level:** `L4 ARBITRATE/GATE`
- **Inputs:** `bio_state`, `wake_needed`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni_media_apply | Flanke in {awake, waking} loest die Wake-Sequenz aus | HomePods-Startlautstaerke, Ramp, Radio-Autostart |
| benni_media_policy | bio_state == "waking" ODER wake_needed haelt die Musik-Baseline zurueck | Baseline schweigt im Wake-Fenster |
| benni_media_policy | steigende wake_needed-Flanke setzt manual_stop zurueck | nur fuer einen Tick |
| benni_light_policy | bio_state == waking | Weckerlicht |
| blind_control | bio_state == waking | waking-Ziel plus Pause aller Umweltanforderungen |

- **Klassifikation:** `PRODUCT_DECISION`
- **Semantic Conflict:** JA - drei verschiedene Trigger fuer dieselbe Lebenslage

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: waking ist der kanonische ZUSTAND fuer alle Zielentscheidungen (Licht, Rollladen, Baseline-Schweigen). Der geplante Weckzeitpunkt (wake_needed bzw. sein Core-State-Nachfolger) sollte der Trigger fuer die AKTION Wake-Sequenz sein. Damit faellt auch D-B16 auseinander: Stop-Latch-Reset auf jede Wach-Flanke, Musikstart nur auf einen geplanten Weckzeitpunkt.

**Begründung:** Benni hat die Frage in control#7 selbst gestellt ('ob der richtige Trigger der Bio-State oder ein Activity-State ist, ist noch offen'). Dieser Audit liefert die Faktenlage dazu: fuenf Consumer, drei Trigger, und ein spontanes Aufstehen um 23:40 startet heute Musik.

- **Betroffene Consumer:** `benni_media_apply`, `benni_media_policy`, `benni_light_policy`, `blind_control`
- **Betroffene Domains:** Medien, Licht, Rollladen, Person
- **Betroffene Edge-IDs:** `E-003`, `E-007`, `E-013`, `E-017`, `E-018`, `E-115`, `E-116`
- **Migrationstyp:** `SEMANTIC_SPLIT`
- **Change Type:** `SEMANTIC_CHANGE`, `BINDING_CHANGE`
- **Compatibility Requirement:** `SAME_RELEASE_REQUIRED`
- **Contract Readiness:** `CONTRACT_AFTER_DECISION`
- **Hängt ab von:** `D-W01`
- **Priorität:** `P2` · **Konfidenz:** hoch
- **Benni:** `OPEN`

> Blockiert D-B16. Beruehrt unmittelbar den Zielbild-Input aus control#7 ('Logiken reagieren auf States, mit eindeutigen Triggerpunkten').

#### `D-W03` — Soll der manual_stop-Reset laenger als einen Tick halten?

- **Decision Level:** `L6 LIFECYCLE/RECOVERY`
- **Inputs:** `wake_needed`, `media_stop_latch`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni_media_policy | steigende wake_needed-Flanke setzt manual_stop=False | steht der externe input_boolean.media_stop_latch weiter auf on, setzt der naechste Tick manual_stop erneut auf True |
| benni_media_apply | loest den externen Latch auf der Bio-Wach-Flanke | der effektive Reset kommt nur von hier |

- **Klassifikation:** `BUG`
- **Semantic Conflict:** JA - der Policy-Reset ist wirkungslos, solange der externe Latch steht

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: der Policy-interne Reset gehoert entweder entfernt (weil Apply den externen Latch loest) oder er muss den externen Latch mitloesen. Beides ist eine Folgeentscheidung aus D-W04.

**Begründung:** Ein Reset, der einen Tick spaeter wieder ueberschrieben wird, sieht im Log aus wie eine funktionierende Freigabe und ist keine.

- **Betroffene Consumer:** `benni_media_policy`, `benni_media_apply`
- **Betroffene Domains:** Medien
- **Betroffene Edge-IDs:** `E-116`
- **Migrationstyp:** `LOCAL_FIX`
- **Change Type:** `CODE_FIX_ONLY`, `SEMANTIC_CHANGE`
- **Compatibility Requirement:** `NONE`
- **Contract Readiness:** `POLICY_INTERNAL`
- **Hängt ab von:** `D-W04`
- **Priorität:** `P2` · **Konfidenz:** hoch
- **Benni:** `OPEN`

> Bestaetigt OPR-063.

#### `D-W04` — Wer besitzt den Stop-Latch?

- **Decision Level:** `L6 LIFECYCLE/RECOVERY`
- **Inputs:** `input_boolean.media_stop_latch`, `manual_stop (media_policy)`

**Aktuelle Consumer-Regeln**

| Consumer | aktuelle Interpretation | Ergebnis |
|---|---|---|
| benni_media_policy | fuehrt ein eigenes manual_stop im RAM, liest zusaetzlich den externen Latch | das Policy-interne Flag wird vom externen Latch ueberschattet |
| benni_media_apply | loest den externen Latch auf der Bio-Wach-Flanke | einziger automatischer Loeser |
| einhornzentrale (Skripte) | script.system_bedtime_mode setzt ihn; mehrere Altskripte loeschen ihn | drei Wege fuer dieselbe Absicht 'Schlafmodus' |

- **Klassifikation:** `ARCHITECTURE_DECISION`
- **Semantic Conflict:** JA - kein Owner, drei Schreiber

**Empfohlene kanonische Regel** *(Vorschlag, nicht beschlossen)*

EMPFEHLUNG: genau einen Owner festlegen. Naheliegend ist media_policy (dort entsteht die Entscheidung 'Nutzer hat gestoppt'), mit dem externen input_boolean als reinem Bedienelement. Die Altskripte, die denselben Zustand parallel setzen, entfallen.

**Begründung:** Der Fall ist in control#7 am 12.09.2026 bereits mit einem konkreten Vorfall belegt (Latch blieb von 02:xx bis zum manuellen Eingriff stehen, weil der einzige automatische Loeser durch eine Regression blockiert war).

- **Betroffene Consumer:** `benni_media_policy`, `benni_media_apply`, `einhornzentrale`
- **Betroffene Domains:** Medien, YAML-Konfiguration
- **Betroffene Edge-IDs:** `E-116`
- **Migrationstyp:** `SEMANTIC_SPLIT`
- **Change Type:** `OWNER_CHANGE`, `SEMANTIC_CHANGE`, `LEGACY_REMOVAL`
- **Compatibility Requirement:** `MIGRATION_REQUIRED`
- **Contract Readiness:** `POLICY_INTERNAL`
- **Hängt ab von:** —
- **Priorität:** `P2` · **Konfidenz:** hoch
- **Benni:** `OPEN`

> Blockiert D-W03. Der Stop-Latch ist kein publizierter State der beiden State-Engines und deshalb im Katalog nicht enthalten - er taucht hier nur als Consumer-Kante auf.

---

## Decision Dependency Graph

Nur echte fachliche Abhängigkeiten — eine Kante bedeutet: *die zweite Entscheidung
ist ohne die erste nicht sinnvoll beantwortbar*. Alles ohne eingehende Kante ist
sofort entscheidbar.

### Strang 1 — Sleep-Semantik

```
D-B01  Publizierter Sleep-Kontext-Begriff
  ├─→ D-B02  Auto-Audio-Start im PS
  ├─→ D-B03  Resume / Repair im PS
  ├─→ D-B04  HomePod-Pause im PS
  ├─→ D-B05  Denon-Nachlauf im PS            (modulinterner Widerspruch)
  ├─→ D-B07  Notifications im PS
  ├─→ D-B08  Heizung im PS                   ──┐
  ├─→ D-B12  Steckdosen-Cuts im PS           ──┤  brauchen zusätzlich D-D05
  ├─→ D-B13  private_time-Latch im PS          │
  ├─→ D-B14  Wake-Kandidat im PS             ──┘  braucht zusätzlich D-S04
  └─→ D-B19  Alias-Werte sleeping/asleep entfernen
```

`D-B06` (Sleep-TV-Timer), `D-B09` (waking = Heizung aus), `D-B10` (Licht),
`D-B11` (Rollladen), `D-B15` (Readiness), `D-B17` (Quiet-Schlafkanal) und
`D-B18` (harte Literale) hängen **nicht** an `D-B01` — sie sind unabhängig
entscheidbar.

### Strang 2 — Away / Presence

```
D-P01  Owner des Away-Execution-Gates
  ├─→ D-P02  Wirkt der Activity-Hold im Media-Away-Gate?
  ├─→ D-P03  Wo gehört der 25-s-Debounce hin?
  └─→ D-P07  blind_control: Away explizit binden

D-P04  Bedeutung von bei_eltern
  └─→ D-P05  Anwesenheitssimulation bei bei_eltern
```

`D-P06` (clean vs. `system_`-Slug) und `D-P08` (unknown-Behandlung) sind
unabhängig.

### Strang 3 — Tagesphasen

```
D-D01  Verbindliche Tagesphasen-Generation
  ├─→ D-D02  Volume-Baseline in midday / late_afternoon / evening  ──┐
  ├─→ D-D03  Subwoofer-Fenster in denselben drei Phasen            ──┤
  └─→ D-D04  Frei-Tag-Bad-Preheat                                    │
                                                                     │
D-D05  Ablösung der benni_combined_context_*-Bindungen  ←────────────┘
  ├─→ D-B08  Heizung im PS
  ├─→ D-B12  Steckdosen-Cuts im PS
  ├─→ D-D04  Frei-Tag-Bad-Preheat
  └─→ D-D06  Abend-Komfortzuschlag (free_time-Auffächerung)
```

**Reihenfolge-Zwang:** `D-D05` darf für `benni_media_policy` erst *nach*
`D-D02` und `D-D03` ausgeliefert werden. Eine Umbindung auf `core_state` ohne
erweiterte Tabellen würde die Lautstärke-Baseline in drei von neun Tagesphasen
auf den flachen Fallback und den Subwoofer in dieselben drei Phasen auf `aus`
setzen — also das Verhalten aktiv verschlechtern.

### Strang 4 — Media-Semantik

```
D-M01  Bedeutung von media_device (Identität / Routing / Screen-Intent)
  └─→ D-M02  Publizierte screen_class
        └─→ D-M06  Sind die „Debug"-Attribute Teil des Contracts?

D-M04  Quality/Freshness am activity_context-Feed
  └─→ D-M05  Owner der Hold-Stärke

D-S04  Ist import.yaml live wirksam?
  ├─→ D-B14  Wake-Kandidat
  ├─→ D-M08  Bias-Light-Doppelwahrheit
  └─→ D-S01  preheat_active-Doppelberechnung
```

`D-M03` (appletv) ist unabhängig und kurzfristig lösbar, wird durch `D-M02`
langfristig überflüssig.

### Strang 5 — Wake

```
D-W01  Owner der Weckentscheidung (bereits control#27/#28/#29/#30)
  └─→ D-W02  Kanonischer Wake-Trigger (Zustand vs. geplanter Zeitpunkt)
        └─→ D-B16  Darf PS → awake die Wake-Sequenz auslösen?

D-W04  Owner des Stop-Latch
  └─→ D-W03  Hält der manual_stop-Reset länger als einen Tick?
```

### Querverbindung

`D-P01` und `D-D05` sind beide **Owner-/Bindungsfragen** und berühren sich nicht
fachlich — sie können parallel entschieden werden. `D-B01`, `D-D01` und `D-M01`
sind die drei **Semantik-Wurzeln**; mit ihren transitiven Folgeentscheidungen
tragen sie 20 der 54 Decisions.

---

## Priorisierung

Reihenfolge nach: (1) andere Decisions hängen davon ab, (2) aktueller Semantic
Conflict bzw. belegtes Fehlverhalten, (3) Cross-Domain-Auswirkung,
(4) Ownership-/Contract-Frage, (5) Naming und Cleanup.

### P1 — zuerst

| ID | Warum P1 |
|---|---|
| `D-B01` | Wurzel: blockiert 10 weitere BIO-Decisions. |
| `D-D01` | Wurzel: blockiert `D-D02`, `D-D03`, `D-D04` und die Reihenfolge von `D-D05`. |
| `D-M01` | Wurzel: blockiert `D-M02` und damit die Screen-Class-Frage in drei Domänen. |
| `D-P01` | Ownership-Wurzel: der als kanonisch deklarierte Away-Gate hat keinen Consumer. |
| `D-D05` | 16 Kanten; solange `climate_policy` an einer möglicherweise toten ID hängt, ist die gesamte Heizlogik in ihrem Verhalten unbestimmt. |
| `D-D02` | Belegtes Fehlverhalten: stiller Fallback der Volume-Baseline in 3 von 9 Phasen. |
| `D-D03` | Belegtes Fehlverhalten: Subwoofer mittags und abends gesperrt. |
| `D-B09` | Belegtes Fehlverhalten: `waking` schaltet die Heizung aus statt sie hochzufahren. |
| `D-B15` | Belegtes Fehlverhalten: Readiness meldet jede Nacht ein falsches Negativ. Billigster Fix im gesamten Review. |
| `D-B17` | Belegter zweiter Schlafkanal an `bio_state` vorbei, wirkt bis in die Benachrichtigungen. |
| `D-M03` | Belegte Lücke: Apple-TV-Abend ohne explizite Blend-Klasse. Einzeiler. |
| `D-M04` | Contract-Wurzel: die einzige Producer-zu-Producer-Kante hat kein Qualitätssignal. |

### P2 — danach

`D-B02` · `D-B05` · `D-B07` · `D-B08` · `D-B12` · `D-B16` ·
`D-P02` · `D-P03` · `D-P04` · `D-P06` · `D-P07` ·
`D-D04` · `D-D06` ·
`D-M02` · `D-M05` · `D-M06` · `D-M07` ·
`D-S04` ·
`D-W01` · `D-W02` · `D-W03` · `D-W04`

Schwerpunkt: die Einzelfälle, die erst nach den P1-Wurzeln beantwortbar sind,
plus die vier Wake-/Stop-Latch-Fragen, die bereits in control#7 belegt sind.

**Sonderfall `D-S04`** (ist `import.yaml` live wirksam?): fachlich eine reine
Tatsachenfrage, read-only in Minuten zu klären — sie entscheidet aber, ob
`D-B14`, `D-M08` und `D-S01` überhaupt Arbeit sind oder nur Dokumentation.

### P3 — Cleanup und Bestätigung

`D-B03` · `D-B04` · `D-B06` · `D-B10` · `D-B11` · `D-B13` · `D-B14` ·
`D-B18` · `D-B19` · `D-P05` · `D-P08` · `D-D07` · `D-D08` ·
`D-M08` · `D-M09` · `D-M10` · `D-S01` · `D-S02` · `D-S03` · `D-S05`

Davon sind neun (`D-B03`, `D-B04`, `D-B06`, `D-B10`, `D-B11`, `D-P05`, `D-P08`,
`D-S02`, `D-S05`) als `NO_PROBLEM` klassifiziert: sie brauchen nur eine Bestätigung, dass
das heutige Verhalten gewollt ist, und keine Änderung.

---

## Fusion Evidence — Core State ↔ Media State

**Bewertung: `PARTIAL_CONSOLIDATION_ONLY`**

Ausschließlich auf Basis der vorhandenen Auditdaten; kein neuer
Architekturvergleich, kein Entwurf einer neuen State Engine.

### Was die Daten zeigen

| Kriterium | Befund |
|---|---|
| Schnittstellenbreite | Genau **eine** Producer-zu-Producer-Kante: `activity_context` (`E-145`). Gerichtet, mit Qualitäts-Gate beim Consumer. |
| Rückkopplungen | Zwei. (a) `bio_state` → media_apply → `sleep_tv_evidence` → `bio_state` (`E-009`/`E-026`), über `sleep_reference_start` sauber korreliert. (b) `presence_personal` → media `away_gate` → `activity_context` → core `activity_state` → Activity-Hold → `presence_effective`, gedämpft weil `presence_personal` roh bleibt. |
| Duplicate Calculation zwischen den Engines | Genau **eine**: `evaluate_quiet` erkennt Schlaf über `activity_state` statt über `bio_state` (`D-B17`) — ein Einzelfix, kein Strukturproblem. |
| Fehlplatzierte Owner | Genau **einer** gut belegt: `presence_state` / `away_gate` in media_state (`D-P01`). |
| Ursachen der teuren Probleme | Semantik (`D-B01`, `D-D01`, `D-M01`, `D-P04`) und Bindungshygiene (`D-D05`, `D-P06`). Keines davon liegt am Modulschnitt. |

### Warum nicht `FUSION_NOT_SUPPORTED`

Der vorangegangene Audit hatte formuliert, eine Fusion werde „nicht gestützt". Das
bleibt richtig für eine **vollständige** Zusammenlegung. Die Decision-Ebene zeigt
aber, dass es sehr wohl eine belastbar begründete **Teil**-Konsolidierung gibt:
die Away-/Presence-Projektion (`D-P01`, `D-P02`) gehört fachlich zu Core State,
und `D-M05` verschiebt die Hold-Stärke in dieselbe Richtung. `PARTIAL_CONSOLIDATION_ONLY`
ist deshalb die präzisere Einordnung derselben Datenlage.

### Warum nicht `FUSION_SUPPORTED`

Eine Fusion würde die einzige saubere Grenze des Stacks aufgeben — eine schmale,
gerichtete, qualitätsgegatete Feed-Kante — und keines der 54 Entscheidungsprobleme
lösen. Von den 12 P1-Decisions würde **keine einzige** durch eine Zusammenlegung
beantwortet.

### Warum nicht `INSUFFICIENT_EVIDENCE`

Die Kantenlage ist vollständig erhoben (139 Kanten über 12 Consumer-Repositories)
und die Schnittstelle zwischen den beiden Engines ist eindeutig identifiziert.
Was fehlt, ist Live-Evidenz zu einzelnen **Bindungen** (`D-P06`, `D-S04`) — nicht
zur Struktur.

### Empfohlener Umfang der Teil-Konsolidierung *(Vorschlag, nicht beschlossen)*

1. `D-P01` / `D-P02` — Away-Ownership zurück zu Core State, Debounce bleibt beim Consumer.
2. `D-M04` — Quality und Freshness am `activity_context`-Feed; damit wird die einzige Engine-Grenze vertraglich sauber.
3. `D-M05` — Hold-Stärke eindeutig bei Core State.

Alles Weitere bleibt beim heutigen Owner.

---

## Abgrenzung

Dieses Dokument ist eine **Entscheidungsvorbereitung**, kein Soll-Vertrag. Es
wurde kein Produktivcode geändert, kein State umbenannt, kein Consumer korrigiert,
kein Contract implementiert und keine Architekturentscheidung als beschlossen
markiert. Alle 54 Einträge stehen auf `benni_decision = OPEN`.
