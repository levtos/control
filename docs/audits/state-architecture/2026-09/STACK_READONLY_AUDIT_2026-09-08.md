# Read-only Audit des Levtos-/Home-Assistant-Stacks

**Stand: 8. September 2026. Ergebnis: Der Stack besitzt bereits tragfähige fachliche Schichten, ist aber noch keine durchgängig contractbasierte, allgemein profilfähige Plattform.**

Die wichtigsten Probleme sind **konkurrierende Media-Wahrheiten, veraltete Consumer-Annahmen, unvollständige Intent-Verträge und weiterhin installationsspezifische Topologie**. Core Contracts ist technisch deutlich weiter als seine produktive Nutzung.

**Das Read-only-Gate wurde eingehalten:** keine Dateien, Konfigurationen, Issues, Branches, Commits, PRs, Releases oder Workflows verändert; keine Geräte gesteuert und keine HA-Live-Änderung ausgeführt.

Die Untersuchung verbindet aktuelle GitHub-Quellen, Entscheidungsverläufe und ausschließlich lesende Stichproben der **Einhornzentrale**. Die Elterninstallation wurde anhand des GitHub-Repositories untersucht. Eine vollständige Live-Storage-/Dashboard-Inventur und mehrere Integration-Diagnostics waren über die verfügbaren Schnittstellen nicht zugänglich. Deshalb sind insbesondere negative Aussagen wie „es existiert garantiert kein weiterer Live-Consumer“ **nicht belegt**. Die folgenden Feststellungen sind keine pauschale Live-Zertifizierung.

---

## 1. Executive Summary

1. **Die tatsächliche zentrale Architektur besteht heute aus Core Devices, Core State und Media State – ergänzt durch Core Contracts.** Core Contracts liegt noch nicht durchgängig zwischen physischen Quellen und fachlichen Engines.

2. **Core Contracts ist als Fundament grundsätzlich verwendbar.** Profilbezogene Registry-Revisionen, LKG, typisierte Consumer-Anforderungen, Quality/Freshness und Subscriptions sind implementiert. Das aktuelle Repository ist `benni-core-contracts`, nicht das nahezu leere `core-contracts`.

3. **Blind Control besitzt den klarsten echten Core-Contracts-Consumer.** Die Nutzung ist selektiv; ungewählte Bereiche bleiben explizite Compatibility-Bindings. Ein flächendeckender produktiver Cutover ist damit nicht bewiesen.

4. **Eine doppelte Media-Wahrheit ist live belegt:** Core Devices meldete `entertainment_active=true`, während Media State gleichzeitig `false` meldete. Ursache sind unterschiedliche Definitionen, nicht bloß zwei Namen derselben Projektion.

5. **Media State ist noch keine rein beobachtende Fachschicht.** Es konsumiert Core-State-Presence/Activity/Bio und unterdrückt bei Away seine Media-Wahrheit. Gleichzeitig konsumiert Core State den Media-Activity-Feed. Das widerspricht der entschiedenen Richtung.

6. **Der TV-Wiederstart ist ein konkreter P0-Fehler:** R12 rearmt bei „TV an“ und interpretiert späteres bewusstes Ausschalten bei noch vorhandenem Screen-Kontext erneut als Einschaltanlass. Das ist in Apply #46 dokumentiert und im aktuellen Code nachvollziehbar.

7. **Apple-TV-Foreground und Playback sind noch nicht vollständig als unabhängige Verträge konsumierbar.** Der aktuelle Streaming-Hold ist verbessert; die frühe HomePod-Pause bleibt ein eigener offener Übergangsvertrag.

8. **Es gibt aktuellen Enum-/Consumer-Drift:** Media Policy verwendet weiterhin acht alte Tagesphasen; Core State liefert neun andere. Light-Musiklogik akzeptiert `free_time`/`idle`, aber nicht das aktuelle `music`. Climate besitzt ebenfalls einen alten `free_time`-Sonderzweig.

9. **Safety-Contract-Lücken bestehen bei Door und Climate.** Door fehlt positive Opening-Closed-Evidence vor Auto-Lock. Climate wertet alte Werte-/Attributformen ohne vollständigen Quality-/Freshness-Vertrag aus. Eine aktuell unsichere Live-Aktion wurde dabei nicht nachgewiesen.

10. **Die Elternfähigkeit ist kein reines Binding-Thema.** Die Klima-Engine setzt drei feste Raumrollen voraus; Media setzt eine feste Gerätekonstellation voraus; Core State unterstützt ein bewusst begrenztes gemeinsames Elternmodell.

11. **Die Persistenzbasis ist besser als die Topologieabstraktion.** Entry-bezogene Stores und getrennte HA-Instanzen verhindern viele Kollisionen. Das beweist noch keine beliebige Mehrprofil-Nutzung innerhalb derselben HA-Instanz – diese ist aber auch nicht der bereits entschiedene Zielumfang.

12. **Umbrella fehlt ein vollständiger Playback-Capability-Vertrag.** Die fehlenden Transportfunktionen sind im aktuellen Backend-/Frontend-Vertrag angelegt und nicht ausschließlich ein Button-Problem.

13. **Mehrere frühere Fehler sind bereits technisch behoben und sollten nicht erneut als offene Architekturdefekte behandelt werden:** Resume-Reconciliation, begrenztes Unmute, Light-Startup-Lux, Blind-Bewegungszuordnung und `provisional_sleep`-Consumer.

14. **Empfohlen ist eine schrittweise Migration:** zuerst Ownership-/Safety-/Drift-Korrekturen, danach vorhandene Opening-/Climate-/Weather-/Presence-Contracts nutzen; fehlende Media- und Capability-Verträge gezielt ergänzen.

15. **Ein Big-Bang-Refactor ist weder erforderlich noch durch die bestehenden Entscheidungen gedeckt.**

Grundlagen: [Control ADR 0002](https://github.com/Levtos/control/blob/28bf515449187c067fe5e23e9c383c0aa0523236/docs/adr/0002-github-only-governance.md), [Architekturentscheidungen in control #37](https://github.com/Levtos/control/issues/37), [Core-Contracts-Ausbau #25](https://github.com/Levtos/benni-core-contracts/issues/25).

---

## 2. Ist-Architektur

### 2.1 Verifizierter Repository-Stand

Die tatsächlichen Default-Branches wurden geprüft und die folgenden Heads am Ende erneut gegen GitHub gelesen. Alle verwenden `main`, außer `discord-game` mit `master`.

| Repository | geprüfter Head | Rolle |
|---|---|---|
| `control` | `28bf5154` | Governance und Entscheidungen |
| `benni-core-contracts` | `def02cdf` | Tatsächliche Contract-Plattform, Manifest 0.2.1 |
| `benni-core-devices` | `52c5e89a` | Geräte-/Domain-Masters, Combined-Kompatibilität |
| `benni-core-state` | `c9408fa3` | Presence, Bio, Zeit, Activity, Wake |
| `benni_media_state` | `c371b328` | Media-Zustand, 0.14.5 |
| `benni_media_policy` | `3392ccef` | Audioentscheidungen, 0.18.2 |
| `benni_media_apply` | `a5296087` | Media-Aktuation, 0.19.8 |
| `benni_media` | `e3085d77` | Umbrella/Frontend, 0.7.6 |
| `Title_classifier` | `cd220cf8` | Titel-/Katalogklassifizierung |
| `blind_control` | `afc2f658` | Neue Blind-Engine, 0.7.2 |
| `benni_blind_policy` | `9cbd1f0d` | Legacy-Blind-Policy |
| `benni_climate_policy` | `0ad085b3` | Klima-Engine, 0.1.9 |
| `benni_light_policy` | `c7004ba6` | Lichtentscheidungen/-Ausführung, 0.3.7 |
| `benni_scene_presets` | `803b54eb` | Looks, Gerätebindings, Rendering |
| `ha_wake_planner` | `9f94d2a8` | Bestehender Wake Planner |
| `plug_policy_engine` | `7d61ba60` | Plug-Schutz und Schaltung |
| `benni_door_policy` | `31dc06df` | Lock-Policy/-Ausführung |
| `benni_notification_router` | `f8801854` | Notification-Policy/-Routing |
| `benni_media_context` | `9919ffa0` | Ältere kombinierte Media-Integration |
| `combined_mediaplayers` | `0ae536ca` | Player-Aggregation |
| `Media_Art_Wrapper` | `16069283` | Artwork-Provider |
| `discord-game` | `20aad14c` | Gaming-/Artwork-Provider |
| `hass-psn` | `71555937` | PSN-Provider |
| `stash-ha` | `8f8739a7` | Stash-Provider |
| `einhornzentrale` | `4d279328` | Benni-YAML, Bindings, Lastenhefte |
| `haos_eltern` | `66be668f` | Eigenständiger Eltern-YAML-Stack |
| `robot_vacuum_control` | `58eed770` | Platzhalter, keine auditierbare Engine |
| `core-contracts` | `dd1317e3` | Platzhalter, nicht die Contract-Plattform |

`bennis_toolbox` ist archiviert. Ein separates aktuelles Warden-Repository oder eine zusätzliche eigenständige Presence-Engine wurde im Repository-Inventar nicht gefunden.

### 2.2 Tatsächlicher Datenfluss

```text
Physische Geräte / HA-Provider
│
├── Core Devices
│   ├── Device Masters: TV, PC, PS5, Denon, Apple TV, ...
│   ├── Opening-/Climate-/weitere Masters
│   ├── eigene Media-Context-Fusion             [zweite Media-Wahrheit]
│   └── Combined-Kompatibilitätsprojektionen
│
├── Core Contracts
│   └── Registry + Bindings + Quality/Freshness + Consumer API
│       └── Blind Control: ausgewählte Schema-Bereiche
│
├── GPS / SSID / Proximity / Kalender ──────────> Core State
│                                                │
│                 Media-Activity-Feed ────────────┤
│                                                ├── Presence / Bio / Day / Activity
│                                                ├── Wake-/Sleep-Lifecycle
│                                                └── Startup-Readiness
│
├── Native Media-Player / MA / Title Classifier ─> Media State
│                      Core Devices ────────────> │
│                      Core State ──────────────> │  [Rückkopplung]
│                                                │
│                                                ├── Media-Context / Device
│                                                ├── Gaming / Entertainment
│                                                └── Activity-Feed ──> Core State
│
│     Media State + Core State + rohe Player/Helper
│                         └──> Media Policy
│                              └──> Media Apply
│                                   ├── MA / HomePods / Denon / TV
│                                   └── TV-Off-Evidence ──> Core State
│                                       [entschiedener, begrenzter Readback]
│
├── Core State + Masters + Rohquellen ──> Climate Policy + Apply ──> Thermostate
├── Core State + Media + Lux/Kalender ──> Light Policy
│                                       ├──> Scene Presets ──> Leuchten
│                                       └──> Hall/Bath/Ring-Adapter
├── Core State + Opening + Umwelt ─────> Blind Control ──> Cover-Adapter
├── Core State + Lock/Batterie ────────> Door Policy + Apply
└── Masters + Plug-/Power-Quellen ─────> Plug Policy + Apply

Core-/Media-/Policy-/Apply-Daten ──> Umbrella und weitere UIs

Parallel:
Einhornzentrale-YAML ──> manuelle Bio-/Radio-/Bedtime-Abläufe
Eltern-YAML ──> eigene Presence-, Heiz- und Apply-Logik
```

Der Rückweg **Media Apply → Core State** ist nicht automatisch ein Architekturfehler: Der TV-Off-Evidence-Vertrag ist ausdrücklich entschieden. Problematisch ist dagegen, dass **Media State seine beobachtete Fachwahrheit anhand von Core-State-Kontext verändert**, während Core State diese Fachwahrheit wieder konsumiert. [Core-State-Entscheidungsdelta](https://github.com/Levtos/benni-core-state/blob/c9408fa33f2914b1c5f7316738cb38b43add33b5/docs/architecture/2026-08-02-core-state-decision-delta.md), [Media-State-Inputs](https://github.com/Levtos/benni_media_state/blob/c371b3282d8bb3a5d35ad58b27179d5e3b4d7f6e/custom_components/benni_media_state/coordinator.py#L670).

### 2.3 Integrationsinventar

„CC“ bedeutet hier Nutzung der **Core-Contracts-Consumer-Schicht**. Ein `sensor.benni_master_*` ist ein stabiler bestehender Vertrag, aber noch kein Nachweis einer Core-Contracts-Migration.

| Integration | Aufgabe / Inputs | Outputs und bekannte Consumer | CC / direkte HA-Inputs |
|---|---|---|---|
| Core Contracts | Technische Quellen über Bindings | Typisierte Werte, Qualität, Revisionen; Blind Control | Fundament; direkte Quellen als Adapter |
| Core Devices | Rohzustände, Leistung, Kontakte, Geräteattribute | Masters und Legacy-Combineds; Core/Media/Policies | Kein flächiger CC-Consumer; viele direkte Adapter |
| Core State | GPS, SSID, Proximity, Kalender, Masters, Media-Feed | Presence/Bio/Day/Activity/Wake/Readiness; fast alle Policies | Kein aktueller CC-Cutover; Roh- und Owner-Entities |
| Media State | Masters, native Player, MA, Classifier, Core State, Helper | Context/Subcontext/Device/Activity/Entertainment; Core, Policy, UX | Kein CC-Cutover; gemischte Inputs |
| Media Policy | Media State, Core State, Player, Opening, Musik-/Intent-Helper | Owner/Szenario/Volume/Pause/Resume/Subwoofer; Apply, UX | Kein CC-Cutover; zusätzliche Rohableitungen |
| Media Apply | Policy-Ziele, Core State, Player-Readback, optionale Wake-Trigger | Geräteaktionen, Health, TV-Off-Evidence; Core, UX | Direkte Aktoren legitim; semantische Zusatzinputs bleiben |
| Umbrella | Coordinator-/Status-APIs der Media-Module | View-Model, Einstellungen, Kommandoweiterleitung | Kein CC nötig für alle Daten; modulbezogene APIs |
| Title Classifier | Titel/Providerdaten, Katalog-/Watcher-Konfiguration | Klassifikation/Enum/Metadaten; Media/Light | Externer Producer, direkte Quellen legitim |
| Blind Control | Core-State-Owner, Opening, Umwelt, Cover | Kandidaten/Ziel/Safety/Movement/Trace; Cover, UX | **Teilweise CC**, sonst explizite Compatibility |
| Legacy Blind Policy | Core/Media/Opening/Umwelt | Ziele, Privacy-/Policy-Zustände, Coveraktionen | Alte Owner-/Entity-Bindings |
| Climate Policy | Core-Kontext, Raum-/Außentemperatur, Fenster, Thermostate | Heizprofile/Ziele/Demand/Debug; eigener Apply | Kein CC-Cutover |
| Light Policy | Core/Media, Classifier, Lux, Kalender, Area-Trigger | Looks/Helligkeit/Area-Aktionen; Presets/Leuchten | Kein CC-Cutover |
| Scene Presets | Look-Inhalt, Zielbindings, explizite Aufträge | Rendering und Geräteaktionen | Geräteadapter; kein Context-Recompute erforderlich |
| Wake Planner | Kalender, Zeit-/Wake-Konfiguration | Wake-Zeit/-Needed/-State; Core, Media | Bestehende Entity-Verträge; Migration offen |
| Plug Policy | Masters, Schalter-/Leistungsdaten, Schutzkontext | Cut-/Supply-Entscheidung und Schaltung | Kein CC-Cutover; gemischte Adapter/Semantik |
| Door Policy | Effective Presence, Lock, Batterie | Lock-Zustand/-Entscheidung/-Aktion | Kein Opening-CC; direkte Lock-Adapter |
| Notification Router | Core-/Media-Kontext, Event-/Safety-Inputs | Routing/DND/Rate-Limits/Notifications | Kein CC-Cutover |
| Eltern-YAML | Elterntracker, Raum-/Fenster-/Thermostatdaten | Eigene Presence-, Ziel- und Apply-States | Eigenständige konkrete Entity-Kette |
| Alte Media Context | Player-/Helperzustände | Kombinierter alter Media-/Volume-Kontext | Legacy; auf Einhornzentrale nicht als Entry geladen |
| Combined Mediaplayers | Konfigurierbare Playerliste | Aggregierter Playerzustand | Adapter; keine zentrale Fachautorität |
| Artwork/Discord/PSN/Stash | Externe Provider | Metadaten und Quellenzustände | Producer, keine pauschale CC-Pflicht |

### 2.4 State, Policy, Aktionen und Persistenz

| Integration | State | Policy | Aktionen | Persistenzform |
|---|---:|---:|---:|---|
| Core Contracts | technische Wahrheit | nein | keine Aktuation | PostgreSQL-Revisionen, profilbezogenes LKG |
| Core Devices | ja | teilweise Legacy-Felder | kein zentraler Policy-Apply | Entry-/Master-Konfiguration und Laufzeit |
| Core State | ja | fachlicher Lifecycle/Arbitration | eigene State-Kommandos | Entry-bezogene State-, Wake- und Command-Stores |
| Media State | ja | Away-/Quiet-/Private-Grenzfälle | primär eigene Zustände | Entry/Latches/Pre-ATV-Zustand |
| Media Policy | Entscheidungszustand | ja | keine Geräteausführung | Entry, Matrix, Orchestratorzustand |
| Media Apply | Ausführungs-/Health-State | zusätzliche R12/R23/R24-Regeln | ja | Entry, Episoden/Timer-/Recoveryzustand |
| Umbrella | Anzeigeprojektion | teilweise Anzeigeheuristik | Gateway-Kommandos | Modulkonfiguration, temporäre Anzeigezustände |
| Title Classifier | Klassifikation | Katalogzuordnung | keine Geräteaktuation | Katalog-/Watcher-Konfiguration, Cache |
| Blind Control | Geräte-/Bewegungsevidence | ja | isolierter Adapter | Entry, Laufzeittracker; frische Restart-Baseline |
| Legacy Blind Policy | ja | ja | ja | Entry und Legacy-Latches |
| Climate | ja | ja | ja | Entry; Hysterese/Boost/Apply-Laufzeit |
| Light | ja | ja | ja | Entry/Subentries, Laufzeittimer |
| Scene Presets | Renderingstatus | keine Context-Policy | ja | Look-/Binding-Daten |
| Wake Planner | ja | Zeitplanung | Wake-Signal | Plan-/Entry-Konfiguration |
| Plug | ja | ja | ja | Geräte-/Modulkonfiguration, Laufzeit |
| Door | Lock-/Applystatus | ja | ja | Entry-bezogener Zustand |
| Notification Router | Routingstatus | ja | ja | Konfiguration, Dedupe-/Routingzustand |
| Eltern-YAML | ja | ja | ja | YAML/Helper/HA-Restore |

**Eine Integration darf Policy und Apply enthalten**, wenn die internen Verantwortlichkeiten eindeutig sind. Eine zusätzliche Repository-Aufspaltung folgt daraus nicht. Für Light/Scene Presets ist die bestehende Teilung sogar ausdrücklich entschieden. [Control #37](https://github.com/Levtos/control/issues/37).

---

## 3. Soll-Architektur aus bereits entschiedenen Verträgen

Aus den geltenden Entscheidungen ergibt sich:

```text
Provider / physische Quellen
    ↓
private Adapter / technische Normalisierung
    ↓
Core Contracts + klar besessene Device-/Domain-Verträge
    ├── Core State: Presence, Bio, Zeit, globale Activity
    └── Media State: beobachtete Media-/Gaming-Wahrheit
              ↓
         Core State darf Media-Fachwahrheit verwenden
              ↓
Policies: fachliche Ziele und Prioritäten
              ↓
Apply: sichere Ausführung, Readback, begrenzte Recovery
              ↓
Geräte

UX / Diagnostics konsumieren versionierte Projektionen der jeweiligen Owner.
```

Bereits entschieden sind insbesondere:

- **Eine Berechnung hat einen Owner.** Projektionen sind erlaubt, parallele Neuberechnungen derselben Bedeutung nicht.
- **Core Contracts erfindet keine physische Sicherheit.** Fehlende Opening-Evidence wird nicht zu „geschlossen“. Fachliches Blockieren gehört zum Consumer.
- **Opening, Lock und Latch sind getrennte Wahrheiten.**
- **Media State besitzt Media-/Gaming-Wahrheit.** Core State besitzt globale Activity; das ist eine legitime Aggregation.
- **Native Apple-TV-Evidence besitzt die Geräte-/Foreground-Wahrheit; MA-Endpunkte sind nicht automatisch deren Ersatz.**
- **Light Policy entscheidet Look und effektive Helligkeit; Scene Presets rendert.**
- **Climate besitzt seine thermischen Ableitungen.** `feels_like`, effektive Außentemperatur oder Floor-Slab-Logik müssen nicht vorsorglich in eine neue globale State-Schicht.
- **`provisional_sleep` und `sleep` sind für die ausdrücklich migrierten Media-/Light-/Blind-Consumer derselbe Sleep-Kontext.** Die Lifecycle-Werte bleiben getrennt.
- **Profile laufen zunächst in getrennten HA-Instanzen.** Vollständige Funktionsparität aller Engines ist nicht bereits beschlossen.
- **Der technische Namespace `benni_` bleibt bestehen.** Er allein ist kein Profilfehler.
- **Migration erfolgt pro Contract/Consumer, mit überprüfter Compatibility und Cutover.**

Quellen: [Control #37](https://github.com/Levtos/control/issues/37), [Core State #59](https://github.com/Levtos/benni-core-state/issues/59), [Core State #62](https://github.com/Levtos/benni-core-state/issues/62), [Core Contracts Consumer API](https://github.com/Levtos/benni-core-contracts/blob/def02cdf6db4daf24bb97ee36678eac090b9f863/custom_components/benni_core_contracts/consumer_api.py).

---

## 4. Shadow-/Grey-Area-Tabelle

**Prioritäten:** P0 = Safety/gefährliche Aktuation oder konkreter Intent-Race; P1 = vor weiterer Feature-Arbeit klären; P2 = Contract-/Profilmigration; P3 = UX/Legacy/Dokumentationskonsistenz.

| ID | Kategorie / Beobachtung | Evidence | Soll dokumentiert? | Risiko | Owner | Prio |
|---|---|---|---|---|---|---|
| G01 | **Duplicate truth:** Core-Devices-Media-Fusion berechnet `entertainment_active` unabhängig von Media State | Live-Widerspruch; [Importdefinition](https://github.com/Levtos/einhornzentrale/blob/4d27932835ce9e72e22b21012589c399f5380d2f/benni_core_devices/import.yaml#L1795) | Ja, Media State ist Owner | Consumer wählen unterschiedliche Bedeutung | Core Devices / Media State | P1 |
| G02 | **CONTRACT CONFLICT:** Media State konsumiert Core-Kontext und setzt bei Away Media-Kontext auf idle | [Away-Zweig](https://github.com/Levtos/benni_media_state/blob/c371b3282d8bb3a5d35ad58b27179d5e3b4d7f6e/custom_components/benni_media_state/logic.py#L679) | Gerichtete Trennung entschieden | Rückkopplung, Verlust beobachteter Wahrheit | Media State | P1 |
| G03 | **Intent-Race:** TV-on rearmt WOL; TV-off bei altem Screen-State löst erneut Einschalten aus | [R12](https://github.com/Levtos/benni_media_apply/blob/a5296087ddb70cece264627c830598c659c47da1/custom_components/benni_media_apply/logic.py#L1301), [#46](https://github.com/Levtos/benni_media_apply/issues/46) | Ja | Bewusstes Ausschalten wird rückgängig gemacht | Media Apply | **P0** |
| G04 | **Missing contract / race:** frühe ATV-Foreground-Evidence erreicht HomePod-Pause nicht als eigener unmittelbarer Pfad | [State #24](https://github.com/Levtos/benni_media_state/issues/24), [Apply #47](https://github.com/Levtos/benni_media_apply/issues/47) | Teilweise, aktueller Issue-Scope | Verzögerte konkurrierende Wiedergabe | State → Policy → Apply | P1 |
| G05 | **Consumer drift:** Media Policy verwendet alte acht Tagesphasen | [Policy-Konstanten](https://github.com/Levtos/benni_media_policy/blob/3392ccefbfaf709253ef0b5efe842da54aa8d20b/custom_components/benni_media_policy/const.py#L125) | Core liefert neun | Fallback-Volume; Subwoofer blockiert gültige neue Phasen | Media Policy | P1 |
| G06 | **Consumer drift:** Light-Musik-Policy akzeptiert `free_time`/`idle`, nicht `music` | [Musik-Gate](https://github.com/Levtos/benni_light_policy/blob/c7004ba66fcd434a96008dbb25cff6aaaa959e61/custom_components/benni_light_policy/policy.py#L597) | Activity-Vertrag entschieden | Musik-Look kann trotz passender Klassifikation ausbleiben | Light Policy | P1 |
| G07 | **Consumer drift:** Climate-Nachtkomfort hängt an Legacy-`free_time` | [Climate-Zweig](https://github.com/Levtos/benni_climate_policy/blob/0ad085b3359f4bef1401a82c9344b07f0e50a093/custom_components/benni_climate_policy/policy.py#L928) | Altregel vorhanden, neue Zuordnung offen | Aktuelle Activity-Buckets erreichen Sonderregel nicht | Climate | P1 |
| G08 | **CONTRACT CONFLICT / Safety:** Door hat kein Opening-Closed-Gate vor Auto-Lock | [Door Policy](https://github.com/Levtos/benni_door_policy/blob/31dc06df7416afcbbb281668d0375aa08e64eee5/custom_components/benni_door_policy/policy.py), [Control #37](https://github.com/Levtos/control/issues/37) | Neuer G4-Vertrag ja; Alt-Spec widerspricht | Verriegelungsentscheidung ohne Tür-Closed-Nachweis | Door | **P0 vor Aktivierung** |
| G09 | **Safety/Adapter:** Climate-Fensterauswertung trägt Quality/Freshness nicht vollständig weiter | [Adapter](https://github.com/Levtos/benni_climate_policy/blob/0ad085b3359f4bef1401a82c9344b07f0e50a093/custom_components/benni_climate_policy/coordinator.py#L215), [WindowState](https://github.com/Levtos/benni_climate_policy/blob/0ad085b3359f4bef1401a82c9344b07f0e50a093/custom_components/benni_climate_policy/models.py#L85) | Fresh positive closed entschieden | Alte/unerwartete Evidence kann permissiv werden | Climate | **P0 vor Ausbau** |
| G10 | **Profile coupling:** Climate besitzt feste Zonen living/kitchen/bath und feste Fensterzuordnung | [ConfigFlow](https://github.com/Levtos/benni_climate_policy/blob/0ad085b3359f4bef1401a82c9344b07f0e50a093/custom_components/benni_climate_policy/config_flow.py#L161) | Allgemeine Profile Ziel | Andere Wohnung braucht strukturelle Anpassung | Climate | P2 |
| G11 | **Profile coupling:** Media-Rollen bleiben TV/ATV/PS5/PC/Denon/HomePods-spezifisch | [State-Konfiguration](https://github.com/Levtos/benni_media_state/blob/c371b3282d8bb3a5d35ad58b27179d5e3b4d7f6e/custom_components/benni_media_state/const.py) | Teilweise | Leeres Eltern-Prefill ersetzt keine Capability-Topologie | Media-Fleet | P2 |
| G12 | **Binding coupling:** Bedeutung hängt teils vom exakten Opening-Entity-Namen ab | [Policy-Adapter](https://github.com/Levtos/benni_media_policy/blob/3392ccefbfaf709253ef0b5efe842da54aa8d20b/custom_components/benni_media_policy/coordinator.py#L375) | Semantische Bindings Ziel | Rename/Profilbindung verändert Interpretation | Media Policy / Climate | P2 |
| G13 | **State reconstruction:** Denon/PS5-Rohplayer ergänzen bzw. überstimmen Master-Aktivität | [Media-State-Inputs](https://github.com/Levtos/benni_media_state/blob/c371b3282d8bb3a5d35ad58b27179d5e3b4d7f6e/custom_components/benni_media_state/coordinator.py#L613) | Gerätewahrheit beim Master | Abweichende Aktivitätsdefinitionen | Media State / Devices | P1 |
| G14 | **State reconstruction:** Policy leitet Denon-Audiopfad auch aus nichtleerer Source ab | [denon_audio_path](https://github.com/Levtos/benni_media_policy/blob/3392ccefbfaf709253ef0b5efe842da54aa8d20b/custom_components/benni_media_policy/logic.py#L863) | Teilweise | Gewählte Source ist nicht zwingend tatsächliches Audio | Media Policy | P1 |
| G15 | **Layer/legacy bypass:** optionale rohe Wake-Trigger können in Apply selbst Wake-Start auslösen | [Apply-Wake](https://github.com/Levtos/benni_media_apply/blob/a5296087ddb70cece264627c830598c659c47da1/custom_components/benni_media_apply/coordinator.py#L1881) | Core besitzt Wake; Extras Legacy | Zweiter Intent-Einstieg | Media Apply / Core | P1 |
| G16 | **Legacy writer:** manuelle Waking/Awake-Skripte starten Radio direkt nach zwei Sekunden | [Bio-Skripte](https://github.com/Levtos/einhornzentrale/blob/4d27932835ce9e72e22b21012589c399f5380d2f/packages/system/manual_bio_scripts.yaml) | Übergangsvertrag, kein finaler zentraler Intent-Pfad | Parallelität zum Core→Policy→Apply-Start | Einhornzentrale / Media | P1 |
| G17 | **Safety-gate gap:** Hall/Bath/Ring prüfen nicht denselben vollständigen Startup-Pfad wie Living | [Area-Controller](https://github.com/Levtos/benni_light_policy/blob/c7004ba66fcd434a96008dbb25cff6aaaa959e61/custom_components/benni_light_policy/areas.py) | Startup-Sperre vorgesehen | Frühe/überholte Area-Aktion möglich | Light Policy | P1 |
| G18 | **UX contract gap:** keine vollständigen Transport-Capabilities für manuelle Musik | [MusicPage](https://github.com/Levtos/benni_media/blob/e3085d7763a45b6460fe55d7cbae5593cc729eec/frontend-src/src/pages/MusicPage.tsx), [Types](https://github.com/Levtos/benni_media/blob/e3085d7763a45b6460fe55d7cbae5593cc729eec/frontend-src/src/types.ts) | Ziel nachvollziehbar, genauer Vertrag offen | UI kann Playerfähigkeiten nicht korrekt anbieten | Umbrella / Apply | P2 |
| G19 | **Diagnostics drift:** Umbrella setzt Lesezeit als Aktualität und `stale=false` | [Aggregator](https://github.com/Levtos/benni_media/blob/e3085d7763a45b6460fe55d7cbae5593cc729eec/custom_components/benni_media/aggregator.py#L52) | Qualität soll Owner-Evidence bleiben | Alte Modulwerte erscheinen frisch | Umbrella | P1 |
| G20 | **Profile gap:** Eltern-YAML ist weiterhin eigene Implementierung, kein Climate-Profil | [Eltern-Stand](https://github.com/Levtos/haos_eltern/tree/66be668fa1bf0eaf5dbc2dddfcc52a44036d2024) | Gemeinsame Engine Ziel | Zwei auseinanderlaufende Regeln | Climate / Eltern | P2 |
| G21 | **Presence drift:** Eltern-YAML behandelt fehlende numerische Trackerwerte über `int(0)` anders als Core-Elternmodell | Eltern-YAML; [Core #62](https://github.com/Levtos/benni-core-state/issues/62) | Neues Core-Modell entschieden | Migration verändert Ausfallverhalten | Core / Eltern | P2 |
| G22 | **Enum ambiguity:** Gaming-Enum 3 und 1 werden im Media-State-Subcontext auf Grind abgebildet | [State-Logik](https://github.com/Levtos/benni_media_state/blob/c371b3282d8bb3a5d35ad58b27179d5e3b4d7f6e/custom_components/benni_media_state/logic.py), [Classifier #83](https://github.com/Levtos/Title_classifier/issues/83) | Nicht vollständig konsolidiert | Bedeutung „preemptible“ nicht eindeutig übergeben | Classifier / Media State | P1 |
| G23 | **Capability workaround:** Switch-Dock wird bewusst hart ignoriert | [switch_dock=False](https://github.com/Levtos/benni_media_state/blob/c371b3282d8bb3a5d35ad58b27179d5e3b4d7f6e/custom_components/benni_media_state/coordinator.py#L676) | Dokumentierter Zwischenzustand | Vorhandenes Gerät ist keine nutzbare Capability | Devices / Media State | P2 |
| G24 | **Project-memory drift:** ältere Backlog-/Testing-Einträge stehen neben bereits supersedierenden Entscheidungen | [Project 1](https://github.com/users/Levtos/projects/1), [Core #59](https://github.com/Levtos/benni-core-state/issues/59), [CC #25](https://github.com/Levtos/benni-core-contracts/issues/25) | Governance verlangt aktuelle Evidence | Folgeagent kann falschen Scope übernehmen | Control / jeweiliger DRI | P3 |

### Wichtige Einschränkungen dieser Findings

- **G08:** Door war in der Live-Stichprobe mit `apply_enabled=false` und ohne nutzbares Lock-Binding gesperrt.
- **G09:** Climate zeigte aktuell ausgeschaltete Heizziele und `window_blocks_heating`. Das beweist den normalen Open-Fall, nicht die Sicherheit bei stale/conflicting Evidence.
- **G16:** Der direkte Skriptpfad ist im aktuellen GitHub-Code vorhanden. Ein tatsächlich doppelter Start durch diesen Pfad wurde in diesem Audit nicht ausgelöst.
- Die zusätzliche YAML-Automation für Radio-Resume war live **ausgeschaltet**. Sie ist ein latenter Legacy-Pfad, kein belegter aktuell laufender Parallelwriter.
- Alte Blind-Control-Criticals wurden durch spätere Versionen und Entscheidungen behandelt. Sie werden hier **nicht unverändert erneut als aktuelle Fehler ausgegeben**. [Blind Control #3](https://github.com/Levtos/blind_control/issues/3).

---

## 5. Source-of-Truth-Matrix

| Semantik | aktueller Producer | weitere Producer/Projektionen | autoritativ laut Vertrag | Consumer / Problem |
|---|---|---|---|---|
| Geräteaktivität `is_active` | Device Master | Media State ergänzt Rohplayer | Device Master | Unterschiedliche Aktivitätskriterien möglich |
| `media_context` | Media State | Core-Devices-Media-Fusion; alte Media Context | Media State | Gleichartige Namen, andere Kategorien |
| `entertainment_active` | Media State | Core-Devices-Fusion und Device-Attribute | Global: Media State | **Live-Widerspruch belegt** |
| globale `activity_state` | Core State | Media State liefert Fachfeed | Core State | Legitimes Aggregat; Rückkopplung vermeiden |
| Media-`activity_context` | Media State | Core-State-Activity | Media State für Media-Anteil | Gleicher Wortstamm, unterschiedlicher Scope |
| Presence personal/effective | Core State | Media-State-Presence/Away-Gate; Eltern-YAML | Core State | Lokale Verzögerung und Bedeutung unterscheiden |
| Household occupancy | Core State / Eltern-YAML | lokale Aktivitätsindikatoren | Profilabhängiger Core-Vertrag | Elternmodell ist kein beliebiges Personenmodell |
| Bio / Sleep / PS | Core State | Policy-`effective_sleep` | Core State | Consumer-Projektion erlaubt |
| Wake-Zeit/-Needed | Wake Planner und neuer Core-Pfad | YAML-Wake-Skripte | Migration noch gated | Parallele Übergangspfade |
| TV-off für Sleep | TV Master → Media Apply Evidence | roher TV-Player für WOL | Master-/Apply-Evidence nach #59 | Bewusst getrennte Zwecke |
| Foreground / Screen ownership | Media State aus LG-Source/TV | `media_device` wird von Apply als Screen-Bedarf verwendet | Media State | Beobachtung und Einschaltintent nicht gleich |
| Audio owner | Media Policy | tatsächlicher Playerzustand | Policy für gewünschtes Routing | Ist-Audio und Soll-Owner nicht vermischen |
| Audio scenario | Media Policy | Umbrella-Anzeige | Policy | `music` kann gelten, obwohl noch nichts spielt |
| Gaming-Klasse | Title Classifier | Media-State-Subcontext | Classifier → Media State | Enum-/Session-Semantik nicht vollständig klar |
| Opening | Opening-Master / CC | alte Combineds, Climate-Adapter, Eltern-YAML | Domain-Owner/CC | Qualität wird nicht überall erhalten |
| Room climate | Sensor-/Climate-Master / CC | Climate-interne thermische Ableitungen | Quellenvertrag; Ableitungen Climate | Kein Grund für pauschale neue Thermal-Schicht |
| Room activity | einzelne Area-/Occupancy-Inputs | keine allgemeine Raum-Activity-Plattform belegt | unvollständig entschieden | Topologie-/Capability-Lücke |
| gewünschter Look | Light Policy | Legacy-/manuelle Aufträge | Light Policy | Scene Presets soll nicht Context rekonstruieren |
| Coverziel | Blind Control bzw. Legacy | manuelle/alte Apply-Einstiege | genau ein aktiver Writer | Aktueller Cutoverzustand separat prüfen |

**Konkreter Live-Befund:**

```text
Core-Devices-Media-Fusion:
    state = pc
    entertainment_active = true

Media State:
    media_context = idle
    media_device = pc
    entertainment_active = false
```

Die Core-Devices-Fusion beantwortet faktisch „irgendeine relevante Quelle aktiv?“. Media State beantwortet eine enger definierte Entertainment-/Szenariofrage.

Dagegen waren `combined_media_context` und `combined_context_master` als **Passthrough-Projektionen** erkennbar. Sie sind nicht allein wegen ihrer Existenz doppelte Berechnungen.

---

## 6. Core-Contracts-Migrationsmatrix

Die aktuelle Standard-Schemaregistry enthält:

- `presence`
- `opening`
- `room_climate`
- `weather_environment`
- `technical_device`

**Nicht vorhanden ist damit automatisch jeder benötigte Media-, Raum-, Wake- oder Transportvertrag.** `technical_device.device_state` ist beispielsweise kein semantisch vollständiger Ersatz für `tv_active`, Foreground oder Audio Ownership. [Aktuelle Schemas](https://github.com/Levtos/benni-core-contracts/blob/def02cdf6db4daf24bb97ee36678eac090b9f863/custom_components/benni_core_contracts/contracts.py).

| Klasse | Consumer / heutiger Input | vorhandener Contract | Empfehlung | Aufwand / Risiko |
|---|---|---|---|---|
| A | Core State: Tracker-Presence | `presence.present` | Trackerbeobachtung pro Binding übernehmen | Mittel; Aggregations-/Freshness-Parität sichern |
| A | Core State: GPS-Distanz, Richtung, SSID-Anker | Nur teilweise abgedeckt | Semantische Evidence-Rollen ergänzen, keine bloße Entity-Typkopie | Mittel–strukturell |
| A | Climate: Raumtemperatur/-feuchte | `room_climate` | Früher Consumer-Cutover | Mittel; Quality in Entscheidung tragen |
| A | Climate: Fensterzustand | `opening` | Hohe Priorität; exakte Closed-/Freshness-Prüfung | Mittel; safety-relevant |
| A | Climate: Außentemperatur/-feuchte | `weather_environment` | Mit bestehender eigener thermischer Engine verbinden | Einfach–mittel |
| A | Climate: Forecast-/Spezialthermik | Nicht vollständig | Nur benötigte technische Felder ergänzen | Mittel; fachliche Formeln bleiben Climate |
| A | Media State: TV/PS5/PC/Denon-Aktivität | `technical_device` reicht semantisch nicht vollständig | Device-Activity-Vertrag präzisieren; Master-Owner erhalten | Mittel |
| A | Media State: Foreground/Playback | Kein vollständiger Media-Vertrag | Beobachtungsdimensionen getrennt veröffentlichen | Strukturell, hoher Consumer-Einfluss |
| A | Media Policy: Opening | `opening` | Exakten Entity-Namen als Semantikschalter entfernen | Einfach–mittel |
| A | Media Policy: tatsächlicher Audiopfad | Lücke | Vom zuständigen Media-Owner beziehen | Mittel |
| A | Light: Außenlux/Wetter | `weather_environment` teilweise passend | Vorhandene Felder nutzen | Einfach–mittel |
| A | Light: Raumlux/Motion/Occupancy | Kein vollständiger Raum-Capability-Vertrag | Kleine Rollenverträge; Außenlux nicht als Raumlux umetikettieren | Mittel |
| A | Door: Opening closed | `opening` | Pflichtinput für automatisches Verriegeln | Mittel; Safety-Gate |
| A | Door: Lock-/Latch-Semantik | `technical_device` nur technisch allgemein | Eigenständige Semantik sauber definieren | Mittel |
| A | Plug: semantische Aktivität/Verfügbarkeit | Teilweise | Master-/Contract-Wahrheit beziehen; Cut-Entscheidung bleibt Plug | Mittel |
| A | Notification Router: Kontext-/Safety-Inputs | Teilweise | Owner-Verträge statt Rohsensorinterpretation | Mittel |
| A | Wake: Kalender-/Wake-Evidence | Kein allgemeiner vollständiger Vertrag | Bestehenden Core-Cutover abschließen, nicht neue Parallelengine | Mittel |
| B | Media Apply: MA, HomePods, Denon, TV-Aktoren | Nicht alles muss CC sein | Direkte Geräteadapter behalten | Niedrig |
| B | Blind: Coverposition/Motion/Aktuation | Bestehender Adapter | Numerische Achse und Geräteausführung dort behalten | Niedrig |
| B | Scene Presets: Light-/Effect-Ziele | Nicht erforderlich | Rendering-Bindings behalten | Niedrig |
| B | Climate Apply: Thermostatdienste | Nicht erforderlich | Zielausführung und Readback behalten | Niedrig |
| C | Title Classifier, Discord, PSN, Stash, Artwork | Producer | Keine künstliche CC-Abhängigkeit im Provider erzwingen | Niedrig |
| D | Blind Control: ausgewählte Opening-/Climate-/Weather-Schemas | Ja | Bereits implementierten API-Pfad nutzen, Rollenmatrix live prüfen | Mittel; kein Rohfallback nach gewähltem defektem Contract |

**Positiv:** Blind Controls CC-Adapter pinnt Schema-Version 1, übernimmt Quality/Freshness und fällt bei einem **gewählten**, aber defekten Contract nicht still auf Rohdaten zurück. Nur **ungewählte** Bereiche bleiben explizite Compatibility. Das ist ein geeignetes Migrationsmuster. [CoreInputs](https://github.com/Levtos/blind_control/blob/afc2f658aac8d2b7ed8a4b47c86ec54f6436e2be/custom_components/blind_control/core_inputs.py).

---

## 7. Profil-Readiness-Matrix

Bewertet wird **dieselbe Codebasis auf getrennten Benni-/Eltern-HA-Instanzen**, mit abweichenden Bindings, Parametern und optionalen Fähigkeiten. Nicht bewertet wird eine unentschiedene allgemeine Mandantenplattform.

| Integration | Benni / Eltern heute | Binding | Policy | Persistenz | optionale Capabilities / Topologie | Bewertung |
|---|---|---|---|---|---|---|
| Core Contracts | Beide Profile modelliert | profilbezogen | fachlich neutral | Registry/Revision/LKG profilbezogen | Schemas erweiterbar, Bestand begrenzt | **READY** als Fundament |
| Core Devices | Benni-Vertragskatalog; konfigurierbare Geräte | weitgehend konfigurierbar | Legacy-Fusion enthält Fachbedeutung | Entry-/Master-Konfiguration | flexibel, aber Benni-Import prägt Realität | **PARTIAL** |
| Core State | Benni plus explizites Eltern-Haushaltsmodell | überschreibbar | einige bewusste Profilzweige | Entry-isolierte Stores | zwei Tracker, gemeinsamer Bio-/Activity-Scope | **PARTIAL** |
| Media State | Benni; Eltern-Prefill leer | konfigurierbar, Fallback-Kopplung | feste Geräte-/Prioritätsrollen | Entry-/Latch-State | keine allgemeine Capability-Topologie | **PARTIAL** |
| Media Policy | Benni; leere Elternroute | überwiegend konfigurierbar | Werte editierbar, Musikbaseline Benni-geprägt | Entry/Matrix | HomePods/Denon-Schema bleibt | **PARTIAL** |
| Media Apply | Benni-Gerätekette | Aktoren konfigurierbar | R12/R23/R24 teils installationsnah | episodische Persistenz vorhanden | MA-/HomePod-/TV-Rollen fest | **PARTIAL** |
| Umbrella | Benni-Media-Modell | Module abstrahiert | Anzeige teilweise rekonstruiert | vor allem Module | Labels/Capabilities nicht allgemein | **PARTIAL** |
| Title Classifier | konfigurierbare Watcher/Kataloge | gut | Klassifikation konfigurierbar | Katalog-/Watcher-State | Anbieter-/Quellenrollen variabel | **MOSTLY READY** |
| Blind Control | konkrete Cover-Engine; CC-Profilwahl | explizit, qualitätsbewusst | viele Kalibrier-/Zielwerte | HA-/Entry-Kontext; frische Baseline | optionale Umweltwerte; kein allgemeines Mehrraumprodukt | **MOSTLY READY** im bestehenden Scope |
| Legacy Blind Policy | Benni-spezifischer Altpfad | konfigurierbar | Legacy-Privacy/Modell | Legacy-Latches | keine Zielplattform für Eltern | **NOT PROFILE SAFE** als Zielarchitektur |
| Climate | drei Benni-Raumrollen; Eltern separat YAML | Slots änderbar | zahlreiche Werte konfigurierbar | Instanzisoliert | feste Zonen/Fensterzuordnung | **NOT PROFILE SAFE** für geforderte Topologie |
| Light | Living plus Hall/Bath/Ring/Subentries | teilweise gut | Profile/Looks konfigurierbar | Entry/Subentry | Hauptmodell weiterhin rollenfest | **PARTIAL** |
| Scene Presets | Look-/Binding-Engine | gut isoliert | keine Presence-/Zeit-Engine | eigene Look-Daten pro HA | Geräte-/Looklisten variabel | **MOSTLY READY** |
| Wake Planner | bestehender persönlicher Planer | Kalender konfigurierbar | Parameter konfigurierbar | eigene Konfiguration | Core-Migration/Haushaltsscope offen | **PARTIAL** |
| Plug Policy | variable Geräte | gut, Benni-Fallbacks | Schutzparameter konfigurierbar | Instanzbezogen | variable Gerätezahl | **MOSTLY READY** mit Contract-Nachzug |
| Door Policy | Benni-Presence-Modell; Elternroute leer | konfigurierbar | persönliche Auto-Unlock-Annahmen | Entry-isoliert | Opening fehlt; Haushaltsscope ungeklärt | **NOT PROFILE SAFE** |
| Notification Router | konfigurierbare Routen | überwiegend konfigurierbar | Regeln/Parameter vorhanden | Instanzbezogen | Kontextrollen noch konkret | **PARTIAL** |
| Provider/Wrapper | quellenbezogen | meist konfigurierbar | keine Haushaltsengine | provider-/entrybezogen | für ihren Adapter-Scope geeignet | **MOSTLY READY** |
| Eltern-YAML | eigene Implementierung | konkret | eigene Regeln | eigene HA | keine wiederverwendbare Engine | **NOT PROFILE SAFE** als gemeinsame Engine |

### Persistenzbewertung

**Belegt gut:** Core Contracts trennt aktive Registry-Revisionen und LKG nach Profil. Consumer-Beobachtungen und Subscriptions führen den Profilbezug mit. Core State verwendet getrennte Entry-Keys für State, Wake-Konfiguration und idempotente UX-Kommandos. [Registry Store](https://github.com/Levtos/benni-core-contracts/blob/def02cdf6db4daf24bb97ee36678eac090b9f863/custom_components/benni_core_contracts/registry_store.py#L45), [Core-State-Stores](https://github.com/Levtos/benni-core-state/blob/c9408fa33f2914b1c5f7316738cb38b43add33b5/custom_components/benni_core_state/const.py).

**Nicht belegt:** eine vollständige gleichzeitige Zwei-Profil-Live-Abnahme aller Engines, Caches und Diagnostics innerhalb derselben HA-Instanz.

### Konkrete Profilblocker

- Raumanzahl und Raumrollen sind nicht durchgängig Daten.
- „Nicht vorhanden“, „nicht gebunden“ und „temporär nicht verfügbar“ sind nicht überall eigenständige Zustände.
- Ein leeres Eltern-Prefill belegt keine Eltern-Businesslogik.
- Medienkonfiguration beschreibt teilweise Produktnamen statt Fähigkeiten.
- Core-Elternmodell ist bewusst ein gemeinsamer Haushalt, kein beliebiges Mehrpersonenmodell.
- Eltern-Heizung ist weiterhin eine andere Implementierung.
- Persistierte Profilwerte sind nicht überall als zusammenhängende versionierte Policy-Konfiguration exportierbar.
- UX zeigt teilweise technische Tracker-/Entity-Slots statt semantischer Rollen.

---

## 8. Media Deep Dive

### 8.1 Begriffe im aktuellen Code

| Begriff | tatsächliche Bedeutung |
|---|---|
| `activity_state` | globale Core-State-Arbitration einschließlich Bio, Arbeit, Haushalt und Media |
| `activity_context` | Media-Anteil für Core State |
| `media_context` | Media-State-Szenario; enthält derzeit auch Away-Unterdrückung |
| `subcontext` | Detail-/Klassifizierungszweig, beispielsweise Streaming-App oder Grind |
| `foreground` | beobachtete LG-/Screen-Quelle |
| `playback` | Playerzustand; nicht gleich Foreground |
| `media_device` | erkanntes/priorisiertes Gerät; kann bei `idle` weiterhin `pc` sein |
| `audio_owner` | Policy-Routing-/Konkurrenzentscheidung |
| `audio_scenario` | gewünschtes Szenario; Music-Baseline kann ohne Playback gelten |
| `entertainment_active` | in Media State enger als „irgendein Gerät aktiv“ |
| `manual playback` | über Session-/Helper-Evidence erkannte Wiedergabe |
| `manual stop` | expliziter Stop-Latch, bewusst nicht aus `playing→idle` abgeleitet |
| Gaming-Klasse | Classifier-Enum plus Media-State-Session-/Subcontext-Regeln |

**Sauber:** Media Policy behandelt einen negativen Player-Edge nicht automatisch als manuellen Stop. Genau diese Unterscheidung zwischen **positivem Intent** und **negativer Gerätebeobachtung** fehlt beim aktuellen TV-WOL-Rearm. [Policy-Logik](https://github.com/Levtos/benni_media_policy/blob/3392ccefbfaf709253ef0b5efe842da54aa8d20b/custom_components/benni_media_policy/logic.py), [Apply #46](https://github.com/Levtos/benni_media_apply/issues/46).

### 8.2 Media-State-/Activity-Matrix

Die Tabelle beschreibt implementierte Pfade bei nutzbaren Inputs. Gleichzeitige konkurrierende Quellen werden zusätzlich durch Prioritäten, Presence und Bio beeinflusst.

| Situation | Activity / Media Context | Foreground / Screen | Audio Owner / HomePods | Producer | Stabilisierung / Mehrdeutigkeit |
|---|---|---|---|---|---|
| Music | Media-Feed `music`, Media-Context oft `idle` | kein Screen nötig | HomePods bzw. Music-Baseline | State + Policy | `idle` bedeutet hier nicht „keine Musik“ |
| Streaming, ATV playing | `entertainment` / `streaming` | Playing-Fast-Path; LG-Source für Arbitration | TV/Denon, HP pausieren | Media State → Policy | normaler kurzer State-Debounce; kein pauschaler 20-s-TV-Wait |
| ATV paused/idle | bestehendes Streaming nur mit passender Foreground-Evidence | TV aktiv + ATV-Source | TV/Denon bleibt | Media State | 5-s-Source-Verlust-Grace; kein neuer Streamingbeweis allein durch idle |
| TV | `entertainment` / `tv` | TV-/Source-Evidence | TV/Denon, HP pausieren | Media State | generischer TV-Startup-Guard 20 s |
| Gaming PC | `gaming` / `gaming` | PC-Gerät bedeutet nicht zwingend LG-Foreground | HP dürfen weiterlaufen; Denon-Audio kann aus | State/Classifier → Policy | Titel-/Session-Evidence und PC-Activity getrennt |
| Gaming PS5 immersiv | `gaming` / `gaming` | PS5-Foreground priorisiert | Gaming-/Denon-Pfad, HP konkurrierend | State/Classifier → Policy | Degraded-Hold nur bei passenden Ausfällen |
| Gaming Grind | `gaming` / `gaming_grind` | Screen kann aktiv sein | HP weiter, Denon Kulisse, Subwoofer aus | Classifier/State/Policy | Enum 1/3 und Preemption nicht vollständig eindeutig |
| Private | `private_time` / `private_time` | kein allgemeiner Screenzwang | Private-Owner, HP verdrängt | Media State, Core arbitriert global | manueller TTL/Latch plus Provider-Evidence |
| Idle | `idle` oder Core `pc_active` | Gerät kann weiter erkannt sein | Music-Baseline möglich | State + Core + Policy | Audio-Soll ist nicht Playback-Ist |
| Sleep / Pre-Sleep | Bio bleibt S/PS; TV-Entertainment kann sichtbar bleiben | TV kann während Sleep-Timer weiterlaufen | HP werden durch Sleep-Kontext gesperrt | Core + Policy + Apply | 45-min-TV-Timer; bestätigtes TV-off separat |
| Manual Apple Music über MA | Media-Feed Music, Context meist idle | kein Screen nötig | laufende manuelle Session schützen | MA → State/Policy/Apply | automatische Recovery darf Session nicht ersetzen; UX-Caps fehlen |

### 8.3 Übergänge und Race-Bewertung

| Übergang | Implementierter Mechanismus | Bewertung |
|---|---|---|
| Music → Apple TV | Playing-/Foreground-Arbitration → Policy-Pause → Apply | Frühe Foreground-Pause und Ausführungsverzögerung offen |
| Apple TV → Music | Kontextverlust → Resume → gemeinsame Recovery-Episode | **Technisch verbessert in 0.19.8**, akustische Live-Abnahme separat |
| Music → PS5 | Aktivität/Foreground + Gaming-Klasse | Nicht jedes Gaming soll Musik stoppen; Klassenvertrag erhalten |
| PS5 → Music | eindeutiges Off beendet Aktivität; Degraded-Fall kann halten | Hold ist nicht universeller 90-s-Off-Delay |
| Apple TV → PS5 | erkannte LG-PS5-Source verdrängt ATV-Hold | Sourcewechsel darf nicht als Sourceverlust-Grace behandelt werden |
| PS5 → Apple TV | LG-Foreground/ATV-Playback priorisieren | Vorhandene PS5-Evidence kann parallel bleiben |
| TV off | State fällt zurück; Apply liest alten Device-Kontext noch kurz | **WOL-Rearm-Race belegt** |
| Sleep-Timer → TV off | Apply schaltet aus, Master liefert Off-Evidence | R12 kann mit noch vorhandenem Screen-State kollidieren |
| PC gaming → idle | Titel-/Klassifikationsverlust und PC-Activity unterschiedlich | PC on ist nicht Gaming; Session-Hold explizit prüfen |
| Gaming-Titel vorübergehend fehlt | vorhandene Session-/Grind-Stabilisierung | verhindert Flapping, kann aber Klassifikation zeitweise konservieren |
| Manual Apple Music | manuelle Session darf automatische Starts blockieren | Transport-/Session-Capability-Vertrag unvollständig |
| Automatic radio | gemeinsamer Single-flight-/Health-Pfad | YAML-Startwege können dieselbe Ownership umgehen |

Die in #41 dokumentierte frühere Gleichsetzung „Service dispatched = Playback erfolgreich“ wurde in 0.19.8 technisch korrigiert. Sie ist **kein unverändert offener Codebefund** dieses Audits. [Aktueller Abschluss #41](https://github.com/Levtos/benni_media_apply/issues/41#issuecomment-5571445353).

---

## 9. Stabilität und Transition Contracts

Die folgenden Werte sind Code-/Konfigurationsdefaults, keine behaupteten universellen Live-Einstellungen.

| Owner / Timer | Dauer | Start | Abbruch / Reset | Überlagerung |
|---|---:|---|---|---|
| Core Startup | 90 s | HA-/Core-Start | neuer Lifecycle | lokale Consumer-Guards können zusätzlich gelten |
| Core Tracker-Freshness | 1.800 s | letzte verwertbare Evidence | neue Evidence | nicht mit Transition-Hold gleichsetzen |
| Core Presence stale | 900 s | Evidence-Alter | frische Evidence | Activity-Hold und profilabhängige Regeln |
| Core Arriving | 5 s | Annäherungs-/Home-Kandidat | Gegenbeweis | Media-Away-Gate ist weitere Schicht |
| Core Leaving / Stable Away | 60 / 120 s | entsprechender Übergang | neue Home-/Gegenevidence | Pfade unterscheiden; nicht pauschal addieren |
| Core Preheat | bis 900 s | Preheat-Situation | Home/Far/Timeout | Climate konsumiert Band/Transition |
| Core Media-Feed-Freshness | 1.800 s | Feed-Zeitstempel | neue/negative Evidence | Producer-Quality hat Vorrang |
| Media State Debounce | 2 s; Rearm-Grenze 6 s | State-Burst | Compute/Stop; decisive Edge begrenzt Rearm | bei Bursts kann Deadline über 6 s liegen |
| Media State LG-Source-Grace | 5 s | expliziter Sourceverlust bei TV on | erkannte Source oder TV off | wird am Eventzeitpunkt geführt, nicht nach Debounce neu gestartet |
| Media State TV-Startup | 20 s | generischer TV-Kaltstart | TV off/qualifizierter anderer Pfad | nicht auf ATV-Playing-Fast-Path aufaddieren |
| Media State PS5-Degraded-Hold | 90 s | geeigneter unknown/unavailable-Ausfall | saubere Off-/neue Evidence | kein allgemeiner PS5-off-Hold |
| Media State Away-Gate | 25 s | vorgelagerte Away-Evidence | Home/Gegenevidence | kann Presence-Stabilisierung nachgelagert sein |
| Media State Private TTL | 4 h | manueller Private-Latch | Clear/Sleep/Timeout | Restart-/Restored-Episode separat behandeln |
| Media Policy Music-Baseline | 30 s, Reevaluation etwa 30,5 s | nicht spielende Baseline | Playback/Stop/Sleep/Away/Konkurrenz | betrifft Start-Recovery, nicht jeden Pause-Pfad |
| Media Apply Debounce | 5 s; Rearm-Grenze 8 s | normaler Plan | Immediate-/Shadow-/No-op-Pfad, Ausführung | normaler Pause-Plan wartet ebenfalls |
| Media Apply Ramp | 16 × 1 s | Volumeziel | neuer Plan/Abbruch | physischer Transportzustand separat |
| Media Apply Radio-Cooldown | 15 s, Backoff bis 120 s | Startversuch/Fehler | neue zulässige Episode | Single-flight bleibt wichtig |
| Media Apply Wake | 5 s; Play-Lead 1 s | Core-Bio-Edge oder optionaler Rohtrigger | Safety-/Episode-Abbruch | YAML startet teils nach 2 s separat |
| Media Apply Settling | 60 s | besessener Start/Resume | Safety-Abbruch/Health-Abschluss | danach begrenzte Recovery |
| Media Apply Health-Samples | 3 × 5 s | Health-Prüfung | fehlerhafte/negative Probe | nicht mit Dispatch-Erfolg gleichsetzen |
| Media Apply Unmute | max. 2 Versuche, 2 s Recheck | bestätigte geeignete Playing-Episode | dauerhafte Safety-Gates | 30-s-Cooldown; 30-min-Backstop |
| Media Apply Hard Recovery | frühestens 300 s; Wait 60 s; Cooldown 30 min | opt-in, anhaltender Fehler | alle Safety-/Ownership-Gates | standardmäßig aus |
| Media Apply Denon-Aus | 90 s | passender PC-/TV-Endpfad | neue Nutzung | separater Geräteschutzpfad |
| Media Apply Private-Ende | 15 s | Private verlässt Kontext | neuer Private-/Blockzustand | kann normalen Music-Pfad verzögern |
| Sleep-TV | 45 min; Warnung 60 s vorher | Sleep-Kontext/TV-Aktivierung | verifiziertes Off, Kontextende; definierte Verlängerung | R12-Race relevant |
| TV-off-Bestätigung | 10 min kontinuierlich | Master bestätigt Off | On/Unknown unterbricht | **ein gemeinsames Evidence-Fenster**, nicht zweimal zehn Minuten |
| Blind Umwelt | Eintritt 10 s / Austritt 120 s | Heat-/Glare-/Cold-Kandidat | Gegenbedingung/harte Gates | zusätzlich Motor-Cooldown |
| Blind Position/Movement | Settling 2 s / Timeout 120 s / Recovery 30 s | Fahrt bzw. Fehler | Bewegung/ungültige Evidence unterbricht Ruhe | keine historische Zielqueue |
| Blind Apply-Cooldown | 60 s | tatsächlicher Dispatch | aktuelle Entscheidung ersetzt Pending | Safety-Bypass nach Vertrag |
| Door | Lock 60 s / Unlock 5 s | zulässiger Kandidat | Recheck/Gegenbedingung | Startup 30 s, Unlock-Cooldown 180 s, Anti-Flap 120 s |
| Light Hall | 120 s; begrenzte Off-Wiederholung | Trigger | Re-Trigger/Stop | Area-Startup-Gates separat |
| Light Bath | konfigurierbar, Default 3.600 s | Licht-/Timerstart | Zustandswechsel/Stop | bis 86.400 s konfigurierbar |
| Climate Fensterhold | bis 300 s im betreffenden Altpfad | bekannte Öffnung | geschlossen/neuer Kontext | Nachtpfad kann unmittelbar blockieren |
| Umbrella Idle-Anzeige | 60 s | Anzeigenwechsel | neue Anzeigeevidence | **nur UX**, keine Aktuationslatenz |

Quellen: [Media-State-Coordinator](https://github.com/Levtos/benni_media_state/blob/c371b3282d8bb3a5d35ad58b27179d5e3b4d7f6e/custom_components/benni_media_state/coordinator.py#L292), [Apply-Debounce](https://github.com/Levtos/benni_media_apply/blob/a5296087ddb70cece264627c830598c659c47da1/custom_components/benni_media_apply/coordinator.py#L746), [Apply-Konstanten](https://github.com/Levtos/benni_media_apply/blob/a5296087ddb70cece264627c830598c659c47da1/custom_components/benni_media_apply/const.py), [Blind-Konfiguration](https://github.com/Levtos/blind_control/blob/afc2f658aac8d2b7ed8a4b47c86ec54f6436e2be/custom_components/blind_control/config.py#L440), [Door-Konstanten](https://github.com/Levtos/benni_door_policy/blob/31dc06df7416afcbbb281668d0375aa08e64eee5/custom_components/benni_door_policy/const.py).

### Relevante Timerketten

**Music → ATV-Pause:**

```text
Media-State-Debounce
    + normaler Apply-Debounce
    + Geräte-/MA-Reaktionszeit
```

Nominal können bereits **2 + 5 Sekunden** entstehen, bevor die physische Reaktion vollständig sichtbar wird. Nicht jeder Pfad durchläuft beide Fenster gleich; die dokumentierte Live-Latenz lässt sich deshalb nicht seriös allein einem Timer zuordnen.

**TV → Music bei fehlgeschlagenem Resume:**

```text
State-Konvergenz
    → Policy-Freigabe
    → Apply
    → 60 s Settling
    → höchstens ein kontrollierter Replace-Versuch
```

Die 60 Sekunden sind aktuell ein bewusst erhaltener Recovery-Wert, kein neu entdeckter unbeabsichtigter Debounce.

**Blind Umweltänderung:**

```text
fachliche Eintritts-/Austrittsstabilisierung
    + gegebenenfalls verbleibender Motor-Cooldown
```

Das ist grundsätzlich legitim. Eine direkte Safety-Anforderung darf diese normale Kette nicht abwarten.

**Zusätzlicher Befund:** Die „max wait“-Werte der Debouncer begrenzen das weitere Rearming, nicht zwingend die absolute Deadline. Bei passenden Bursts sind daher ungefähr **6+2 Sekunden** im State bzw. **8+5 Sekunden** im Apply möglich. Die Bezeichnung sollte diesen Unterschied nicht verdecken.

---

## 10. Consumer-Drift, Legacy und UX

### 10.1 Belegter aktueller Drift

| Producer | Consumer-Annahme | tatsächliche Folge |
|---|---|---|
| Core State: neun Phasen einschließlich `midday`, `late_afternoon`, `evening` | Media Policy: alte acht Phasen mit `late_morning`, `early_evening` | Neue Phasen fallen auf Basiswerte zurück; Subwoofer-Fenster lehnt sie ab |
| Core State: `music` | Light-Musik-Gate: `free_time`/`idle` | Musik-Look wird nicht aufgrund des aktuellen Music-Buckets freigegeben |
| Core State: differenzierte Media-Activity | Climate: `free_time`-Nachtkomfort | Legacy-Sonderregel erreicht aktuelle Media-Buckets nicht |
| Opening-/CC-Quality | Climate: ältere boolesche/Attributformen | Freshness und Konfliktzustände sind nicht gleichwertig integriert |
| Classifier-Enum 1/3 | Media State: beide Grind-Subcontext | Unterschiedliche Prioritätsbedeutung bleibt nicht eindeutig als eigener Contract erhalten |

**Kein aktueller Tagesphasenfehler bei Light:** Light besitzt bereits eine explizite Neun-Phasen-Matrix plus Legacy-Kompatibilität. Dieser Nachzug sollte als Vorbild geprüft werden, nicht erneut umgesetzt werden. [Light-Konstanten](https://github.com/Levtos/benni_light_policy/blob/c7004ba66fcd434a96008dbb25cff6aaaa959e61/custom_components/benni_light_policy/const.py#L64).

### 10.2 Legitimer Compatibility Layer

- Combined-Sensoren, die lediglich einen aktuellen Owner projizieren.
- Legacy-Entity-Migrationen, die gespeicherte Bindings nachvollziehbar weiterführen.
- Native Geräteadapter in Apply und Scene Presets.
- Nicht ausgewählte CC-Bereiche in Blind Control mit sichtbarem Compatibility-Status.
- Wake Planner während eines ausdrücklich noch nicht abgeschlossenen Core-Cutovers.
- Historische Blind-Integration als ausgeschalteter Rollback-Bestand.

### 10.3 Legacy mit zweiter Wahrheit oder zweitem Steuerweg

- Core-Devices-Media-Fusion mit eigener Entertainment-Definition.
- Alte `benni_media_context`-Engine, falls zusätzlich aktiviert.
- Direkte Radio-/Wake-YAML-Sequenzen neben Media Apply.
- Policy-artige Combineds wie Bias-Light-/Plug-Protection-Bedeutungen im Core-Devices-Import.
- Benni-Prefills, deren Fallbacklogik explizites Nicht-Binden nicht eindeutig von „kein gespeicherter Wert“ unterscheidet.
- Historische Enum-/Tagesphasenannahmen in aktuellen Consumer-Engines.

### 10.4 UX als Contract-Consumer

**Umbrella macht bereits einiges richtig:** Es aggregiert Modul-APIs und leitet Befehle an bestehende Backend-Komponenten weiter. Die UI ist nicht generell ein neuer Media-Owner.

Offene Lücken:

1. **Transport-Capabilities fehlen:** Play/Pause, Next, Previous, Shuffle und Repeat benötigen unterstützte Fähigkeiten und einen adressierbaren Session-/Player-Kontext.
2. **Frische wird beschönigt:** `updated_at` beim Lesen und `stale=false` ersetzen keine Producer-Evidence.
3. **Snapshots sind nicht als gemeinsame kausale Revision modelliert:** State, Policy und Apply können unterschiedliche Übergangsstufen zeigen.
4. **Anzeigeheuristiken rekonstruieren Bedeutung:** „Aus“ wird teilweise aus mehreren Modulwerten abgeleitet.
5. **Produktlabels bleiben konkret:** HomePods-/Geräterollen sind nicht vollständig profilneutral.

Die angemessene Lösung ist ein **kleiner versionierter Playback-/Transport-View-Model-Vertrag**. Ein Frontend sollte weder `supported_features` beliebiger Rohplayer selbst interpretieren noch Businessentscheidungen nachbauen. Welche Komponente den normalisierten Capability-Vertrag produziert, muss im Media-Scope ausdrücklich festgelegt werden.

---

## 11. Priorisierte Empfehlungen

### P0 – Aktuation und Safety

**P0.1 TV-WOL an eine echte Intent-Episode binden.**

Bestehendes Apply #46 verwenden. „TV ist jetzt aus“ darf keinen neuen Einschaltintent erzeugen. TV-on darf dieselbe Episode nicht automatisch rearmen. Sleep-Timer-Off gehört in dieselbe Gegenfallmatrix.

**P0.2 Door vor automatischer Aktivierung an Opening-Closed-Evidence binden.**

Alte Tür-Spec und neue G4-Entscheidung als **CONTRACT CONFLICT** konsolidieren. Kein Auto-Lock auf Grundlage von „entriegelt“ allein. Aktuell gesperrte Live-Konfiguration nicht vorschnell aktivieren.

**P0.3 Climate-Opening-Adapter qualitätssicher machen.**

Nur gültige, frische, zuverlässige Closed-Evidence darf die relevante Heizfreigabe tragen. Altwert-/Attributinterpretationen und das bestehende Fensterhold müssen gegen den geltenden Safety-Vertrag abgeglichen werden.

### P1 – Vor weiterer Feature-Arbeit

- Core-Devices-Media-Fusion und Media State auf **einen globalen Media-Owner** zurückführen; zuerst Consumer inventarisieren.
- Media-State-Core-Rückkopplung bereinigen: beobachtete Medienwahrheit und Policy-Unterdrückung trennen.
- Tagesphasen-/Activity-Drift als zusammengehörigen Consumer-Nachzug behandeln.
- Foreground-/Playback-/Intent-Vertrag für die vorhandenen Media-Issues präzisieren.
- Rohe Wake-Extras und manuelle YAML-Radiostarts einem eindeutigen Intent-/Start-Owner zuordnen.
- Light-Area-Startup-/Dispatch-Gates angleichen.
- Umbrella-Quality/Freshness aus den Produzenten übernehmen.

### P2 – Core Contracts und Profile

- Vorhandene Opening-, Room-Climate-, Weather- und Presence-Schemas zuerst nutzen.
- Core-State-Elternmigration mit derselben Fehler-/Freshness-Semantik absichern.
- Climate-Raum-/Fenster-/Capability-Topologie datengetrieben machen.
- Erst danach Eltern-YAML schrittweise ersetzen.
- Media-Capabilities und Geräteaktivität gezielt ergänzen; keine zweite HA-Registry aufbauen.
- Blind Controls bereits vorhandenen CC-Pfad anhand aktiver Rollen und Registry-Revisionen verifizieren.

### P3 – UX und Legacy

- Playback-Capabilities in Umbrella sichtbar machen.
- Legacy-Pfade mit Owner, verbleibenden Consumern und Sunset-Gate kennzeichnen.
- Veraltete Project-Einträge und irreführende Issue-Referenzen konsolidieren.
- Technische Entity-Slots in Profil-UX durch verständliche Rollen ergänzen.
- Keine pauschalen Umbenennungen des `benni_`-Namespaces.

---

## 12. Vorgeschlagene Folge-Issues

**Nur Vorschläge; nichts wurde erstellt oder verändert.** Bestehende Issues sollten erweitert bzw. weiterverwendet werden, wenn der Scope bereits passt.

| Repository / Titel | Problem und Scope | Out of scope | Abhängigkeiten | Agent-Empfehlung |
|---|---|---|---|---|
| `benni_media_apply` – **#46 fortführen: TV-WOL nur aus positiver Screen-Intent-Episode** | Rearm, TV-off, Sleep-off, keine wiederholten Starts | neue Media-Architektur, Ramp-Umbau | vorhandener State-/Intent-Vertrag | bestehender Issue-DRI; sonst Claude gemäß Ownership |
| `benni_media_state` + `benni_media_apply` – **#24/#47: Foreground → Pause als Übergangsvertrag** | frühe Evidence, Cancellation, Latenzpfad, Zwischenzustände | pauschale Verkürzung aller Timer | #46 sauber getrennt halten | jeweiliger bestehender DRI, koordinierter Contract |
| `control`, Umsetzung `benni-core-devices`/`benni_media_state` – **Ein autoritativer Media-/Entertainment-Vertrag** | doppelte Fusion, Definitionen, Consumer, Rückkopplung | neue Mega-State-Engine | vollständige aktive Consumerliste | Codex Devices / Claude Media, getrennte Implementierungsscopes |
| `control`, betroffene Policies – **Consumer-Nachzug für Day-/Activity-Vertrag** | Media-Tagesphasen, Light-Music, Climate-Legacy-Bucket | neue Tagesphasen oder neue Komfortwerte | Produktfragen unten | Claude Media/Light; Codex Climate |
| `benni_door_policy` – **Auto-Lock benötigt positive Opening-Closed-Evidence** | Conflict Alt-Spec/G4, Qualität, Recheck vor Dispatch | neue Auto-Unlock-Strategie, Lock-open | Opening-Vertrag | Claude |
| `benni_climate_policy` – **Opening-Quality und profilneutrale Raumtopologie** | zuerst Safety-Adapter; danach getrennte Topologie-Migration | komplette Heizlogik neu schreiben | Fensterhold-Entscheidung, Eltern-Inventar | Codex; zwei begrenzte Arbeitspakete |
| `benni-core-state` / `haos_eltern` – **Eltern-Contract-Cutover mit Presence-Parität** | Tracker-Evidence, Ausfallverhalten, Haushaltsscope, Bindings | Mehrpersonen-Lifecycle ohne Entscheidung | CC-Presence-Rollen | Claude Core; Eltern-Systemarbeit nach Blockerregel |
| `benni_media_apply` / `einhornzentrale` – **Ein Start-Owner für Wake und manuelle Radio-Intents** | YAML-Startwege/optionale Rohtrigger, Idempotenz | bestehende Single-flight-Recovery ersetzen | Core-Wake-Cutover | Claude bzw. bestehender Apply-DRI |
| `benni_light_policy` – **Gemeinsame Readiness-/Dispatch-Gates für Area-Controller** | Hall/Bath/Ring, Startup, Recheck | Light-/Scene-Presets-Aufspaltung | bestehender Startup-Vertrag | Claude |
| `benni_media` – **Playback-Capabilities und ehrliche Snapshot-Qualität** | Transport-View-Model, Sourcezeit/Quality, keine Roh-UI-Logik | visueller Komplettumbau | Capability-Owner festlegen | Claude |
| `blind_control` – **#3 weiterführen: aktive CC-Rollen und verbleibende Live-Gates** | tatsächliche Registry-/Binding-Evidence, aktueller Shadow-/Writerzustand | neue theoretische Komplettmigration, erneutes Öffnen supersedierter Findings | Benni-Gates und vorhandener API-Pfad | bestehender Issue-DRI |
| `control` – **Project-/Contract-Memory konsolidieren** | supersedierte Backlogs/Entscheidungen verknüpfen | neue Architekturentscheidungen | Ergebnisse der fachlichen Reviews | Governance-Owner |

### BLOCKED – PRODUCT DECISION REQUIRED

Vor den jeweils abhängigen Implementierungen sind diese Fragen konkret zu beantworten:

1. **Climate bei `provisional_sleep`:** Soll PS dieselben Heizfolgen wie `sleep` haben? Der bereits entschiedene Media-/Light-/Blind-Consumer-Vertrag beantwortet diese Klima-Frage nicht automatisch.

2. **Climate-Nachtkomfort:** Welche aktuellen Activity-Werte ersetzen fachlich den alten `free_time`-Sonderfall: Music, Entertainment, Gaming, PC-active oder eine andere Kombination?

3. **Media-Tagesphasen:** Welche Volume-Baselines und Subwoofer-Freigaben gelten für `midday`, `late_afternoon` und `evening`, soweit keine aktuelle verbindliche Zuordnung vorhanden ist?

4. **Gaming-Enum 3:** Ist es gegenüber Enum 1 eine eigenständige Preemption-/Audio-Fähigkeit oder lediglich eine Legacy-Klassifikation? Der gewünschte Unterschied muss Producer und Consumer gemeinsam erreichen.

5. **Eltern-Topologie:** Welche Räume, Fenstergruppen und optionalen Fähigkeiten gehören zum ersten gemeinsamen Climate-Engine-Scope? Keine Annahme, dass drei Benni-Raumrollen genügen.

6. **Eltern-Bio/Wake:** Bleibt es beim bereits implementierten gemeinsamen Haushaltsmodell, oder werden künftig getrennte Personen-Lifecycles benötigt? Letzteres wäre zusätzlicher Produktscope.

7. **Fensterhold Climate:** Wie wird der alte 300-s-Hold gegenüber dem neueren positiven Closed-Safety-Vertrag behandelt? Dokumentationskonflikt ausdrücklich auflösen.

8. **Manuelle Media-Transportkommandos:** Welcher Backend-Owner stellt normalisierte Fähigkeiten und die aktuelle Session bereit? Welche Auswirkungen haben Pause/Stop auf den automatischen Musik-Intent?

**Keine neue Produktentscheidung erforderlich** ist dagegen für: unbekanntes Opening nicht als geschlossen behandeln, TV-off nicht als positiven Einschaltintent interpretieren, erfolgreiche Aktuation nicht allein aus Dispatch ableiten oder fremde Quality nicht beim Lesen frischstempeln.

---

## 13. Ausdrückliche Abschlussantworten

**1. Was ist heute die tatsächliche zentrale State-/Contract-Architektur?**
Ein hybrider Stack: Core Devices liefert Masters, Core State globale Zustände, Media State Medienzustände. Core Contracts bietet das neue technische Fundament, wird aber noch nicht durchgängig konsumiert.

**2. Wo existieren doppelte oder konkurrierende Wahrheiten?**
Vor allem Core-Devices-Media-Fusion versus Media State; außerdem Presence-/Away-Projektionen, rohe Geräte-Rekonstruktionen, Wake-Übergangspfade und die separate Eltern-YAML-Implementierung. Reine Passthrough-Combineds sind davon zu unterscheiden.

**3. Welche Integrationen umgehen Core Contracts noch?**
Core State, Media State/Policy, Climate, Light, Door, Plug und Notification Router verwenden weiterhin eigene Entity-/Master-Bindings. Bei Apply, Scene Presets und externen Providern sind direkte Gerätezugriffe teilweise ausdrücklich legitim.

**4. Welche Schicht besitzt heute fälschlich Verantwortung einer anderen Schicht?**
Media State enthält Policy-Unterdrückung durch Away; Core Devices berechnet globale Media-Bedeutung; Media Policy rekonstruiert Teile des Audiopfads; Media Apply besitzt optionale rohe Wake-Intent-Einstiege; YAML startet Radio parallel zum zentralen Pfad.

**5. Welche fünf Bereiche erzeugen das größte Race-/Regression-Risiko?**

1. TV-WOL-Rearm gegen bewusstes oder timerbedingtes TV-off.
2. Foreground-/Playback-Übergänge mit State-/Apply-Verzögerung.
3. Konkurrierende Media-Definitionen und Core↔Media-Rückkopplung.
4. Wake/Radio/Resume über zentrale Episoden plus Legacy-Startwege.
5. Safety-Rechecks bei Opening, Door, Climate und Area-Startup.

**6. Ist der Stack bereit für `benni` + `eltern` mit derselben Engine?**
**Teilweise, insgesamt nein.** Das Fundament ist geeignet; mehrere Engines sind konfigurierbar. Die gewünschte variable Wohnung-/Raum-/Capability-Topologie ist noch nicht durchgängig umgesetzt.

**7. Was sind die konkreten Blocker?**
Feste Raum-/Geräterollen, unvollständige semantische Bindings, nicht einheitliche Quality-Behandlung, alte Enum-Annahmen, begrenzter Eltern-Haushaltsscope und eine weiterhin separate Eltern-Heizimplementierung.

**8. Welche Core-Contracts-Migrationen sollten als Nächstes erfolgen?**
Opening und Room Climate für Climate/Door, Weather für Climate/Light/Blind sowie Presence-Evidence für Core State. Danach gezielte Device-Activity-/Media-Capability-Verträge. Bestehende Safety- und Ownership-Fehler vorher oder im selben eng begrenzten Consumer-Schritt korrigieren.

**9. Was sollte ausdrücklich nicht verändert werden?**
Die grundsätzliche State→Policy→Apply-Trennung; Device-Master-Ownership; Core State als globaler Activity-Owner; Media State als Media-Fachowner; Scene Presets als Renderer; Climate-eigene thermische Ableitungen; Apply-Single-flight/Health/Unmute; Blind Controls isolierter Adapter und logische Achse; Core-Contracts-Revision/LKG/Quality; der technische `benni_`-Namespace.

**10. Welche Produktentscheidungen fehlen?**
Climate-PS-Verhalten, Ersatz des `free_time`-Komfortfalls, neue Media-Tagesphasenwerte, Gaming-Enum-3-Semantik, erster Eltern-Topologieumfang, gegebenenfalls Mehrpersonen-Bio/Wake, Fensterhold-Konflikt und der manuelle Playback-Capability-/Intent-Vertrag.

**STOP. Keine Implementierung, keine Dateiänderung und keine Issue-/PR-Erstellung begonnen.**
