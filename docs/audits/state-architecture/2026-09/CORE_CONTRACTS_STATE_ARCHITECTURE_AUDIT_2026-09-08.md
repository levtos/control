# Architektur-Audit: Core Contracts als zentrale Daten- und State-Schicht

**Stand: 08.09.2026. Ergebnis: Hybrid – zentraler Datenzugang, eindeutige fachliche Owner und eine schrittweise Konsolidierung geeigneter Berechnungen.**

**Read-only eingehalten:** Keine Dateien, GitHub-Objekte oder Home-Assistant-Zustände wurden verändert. Es wurden keine Tests, Workflows oder Deployments gestartet.

Der Bericht unterscheidet:

- **Belegt:** durch untersuchten Default-Branch-Code, Spezifikationen oder GitHub-Entscheidungen.
- **CONTRACT CONFLICT:** widersprüchliche Vorgaben oder Abweichungen zwischen Vertrag und Implementierung.
- **Empfehlung:** vorgeschlagene Architektur, noch nicht beschlossen.
- **Nicht belegt:** insbesondere aktuelles Live-Verhalten, Performance und vollständige Migrationsparität.

Die bestehenden Audit-Ergebnisse wurden als Ausgangspunkt verwendet und entscheidungsrelevante Stellen gezielt vertieft. Die Default-Branches der untersuchten Repositories wurden verifiziert. Für Core Contracts, Core State, Media State, Climate, Wake Planner und Control wurden die maßgeblichen Quellen und relevanten Diskussionen herangezogen. Historische Test- und Live-Nachweise bleiben historische Nachweise.

---

# A. Executive Summary

## Entscheidung: Hybrid

**Core Contracts sollte die kanonische Zugriffsschicht für gemeinsam genutzte fachliche Zustände werden. Ich empfehle gegenwärtig jedoch nicht, Core State und Media State vollständig in seine Runtime zu verschieben.**

Dafür sprechen folgende Befunde:

1. **Die vorhandene Foundation ist substantiell.** Registry, Revisionen, profilbezogene LKG, typisierte Consumer API, Anforderungen, Quality, Freshness, Lineage und Subscriptions sind vorhanden. Ein Ersatz dieser Foundation wäre nicht gerechtfertigt.

2. **Core Contracts ist heute keine allgemeine State-Runtime.** Seine Fusionen ersetzen weder die Bio-State-Machine noch Media-Arbitration, Kalenderbeschaffung oder restartfeste Ereignisverarbeitung.

3. **Zentraler Zugriff und zentraler Berechnungsort sind unterschiedliche Entscheidungen.** Ein externer, eindeutig zuständiger Owner kann einen kanonischen Contract liefern. Dafür fehlt gegenwärtig allerdings eine geeignete typisierte Producer-Anbindung.

4. **Die wichtigsten Probleme sind semantisch und zeitlich.** Konkurrierende Media-Bedeutungen, Rückkopplungen, Kalenderqualität und uneinheitliche Fallbacks verschwinden nicht durch das Zusammenlegen von Repositories.

5. **Deine Fenster-Sicherheitsvorgabe ist im untersuchten Core-Contracts-Pfad nicht vollständig umgesetzt.** Der aktuelle Opening-Vertrag lehnt einen physischen Ersatzwert ab und liefert bei fehlender Evidenz `unknown`. Eine Batterieabhängigkeit, die zentral den von dir geforderten wirksamen Zustand „offen“ erzeugt, ist dort nicht vorhanden.

6. **Auch der gewünschte zentrale Quellenwechsel hat noch konkrete Grenzen.** Der vorhandene generische Source Listener liest den Entity-State. Eine allgemeine Attributauswahl und Einheitenumrechnung sind darin nicht implementiert.

7. **Ein Teil der heutigen Climate-Berechnungen gehört tatsächlich außerhalb einer Heizentscheidung.** Das betrifft beispielsweise Messwertaufbereitung und bestimmte physikalische Ableitungen. Andere Werte sind bereits heizungsspezifische Bewertungsgrößen und dürfen nicht als neutrale Umweltwahrheit umetikettiert werden.

8. **Wake ist bereits teilweise konsolidiert.** Core State besitzt eine interne Planberechnung. Eine erneute Verlagerung nach Core Contracts würde eine bestehende Ownership-Entscheidung ändern. Die offene Baustelle ist vor allem vollständige Quellen-, Qualitäts-, Event- und Cutover-Parität.

9. **Benni und Eltern können dieselbe Architektur verwenden, der gesamte Stack ist dafür aber noch nicht bereit.** Core Contracts trennt Profile bereits deutlich besser als mehrere Consumer ihre Räume, Geräte und optionalen Fähigkeiten.

10. **Empfohlen ist Variante C mit begrenzten internen Resolvern:** gemeinsame Bindings und Contracts in Core Contracts; komplexe Engines zunächst als eigenständige Owner; kleine, tatsächlich gemeinsam benötigte Ableitungen können später in fachlich isolierten internen Modulen laufen.

11. **Akute Fehler bleiben unabhängige Arbeitspakete.** TV-Wiederstart und verspätete Musikpause dürfen nicht auf eine vollständige Registry-Migration warten.

12. **Es gibt keinen technischen Beleg dafür, dass die Anzahl der Integrationen das Hauptproblem ist.** Belegt sind hingegen unklare Bedeutungsgrenzen und mehrere Transport- beziehungsweise Auswertungspfade.

Grundlagen: [Core-Contracts-Architektur](https://github.com/Levtos/benni-core-contracts/blob/def02cdf6db4daf24bb97ee36678eac090b9f863/docs/architecture.md), [Consumer API](https://github.com/Levtos/benni-core-contracts/blob/def02cdf6db4daf24bb97ee36678eac090b9f863/docs/consumer-api-v1.md), [Control ADR 0002](https://github.com/Levtos/control/blob/28bf515449187c067fe5e23e9c383c0aa0523236/docs/adr/0002-github-only-governance.md).

---

# B. Ist-Architektur

## Untersuchter Repository-Rahmen

„Vorhandenes Repository“ bedeutet hier ausdrücklich nicht „aktuell auf jeder HA-Instanz installiert“.

| Bereich | Tatsächlich vorhandene, untersuchte Repositories |
|---|---|
| Governance | `control` |
| Zentrale Verträge und Zustände | `benni-core-contracts`, `benni-core-state`, `benni-core-devices` |
| Medien | `benni_media_state`, `benni_media_policy`, `benni_media_apply`, `benni_media` |
| Weitere Medienpfade | `benni_media_context`, `combined_mediaplayers`, `Media_Art_Wrapper` |
| Klassifikation und Provider | `Title_classifier`, `discord-game`, `hass-psn`, `stash-ha`, `bgtracker` |
| Wake | `ha_wake_planner` |
| Climate | `benni_climate_policy` |
| Licht | `benni_light_policy`, `benni_scene_presets` |
| Rollo | `blind_control`, `benni_blind_policy` |
| Weitere Consumer/Aktoren | `benni_door_policy`, `plug_policy_engine`, `benni_notification_router` |
| Installationskonfiguration | `einhornzentrale`, `haos_eltern` |

Maßgebliche geprüfte Stände sind unter anderem Core Contracts **0.2.1**, Core State **0.11.8**, Media State **0.14.5**, Media Apply **0.19.8**, Climate Policy **0.1.9** und Blind Control **0.7.2**. Diese Angaben bezeichnen Repository-Stände, keine bestätigten Live-Installationen.

## Tatsächlicher Datenfluss

```text
HA-Geräte / Tracker / Kalender / externe Provider
│
├── Core Contracts
│   ├── SourceBindings → AtomicSignals → begrenzte Fusionen
│   ├── Registry / Revisionen / Quality / Freshness / Lineage
│   ├── interne Consumer API
│   └── begrenzte öffentliche Opening-Projektion
│                │
│                └── Blind Control: ausgewählter Contract-Pfad
│
├── Core Devices / vorhandene YAML-Normalisierung
│   └── Device-/Opening-/Media-Masters
│          ├── Media State
│          └── weitere Policies / Consumer
│
├── Core State
│   ├── Presence / Übergänge / Activity-Hold
│   ├── Bio / Sleep / Waking
│   ├── Tagesphase / Tageskontext
│   └── interne Wake-Planberechnung
│          │
│          ├── Climate / Light / Blind / Door / weitere Consumer
│          └── Media State ───────────────┐
│                    ▲                  │
│                    └── Media-Activity ┘
│
├── alter Wake Planner
│   ├── eigene Kalenderbeschaffung / Cache
│   ├── Regeln / manuelle Zustände
│   └── Entities / Vorschau / Wake-Event
│
├── Climate Policy
│   ├── eigene Weather-Aufbereitung / Forecast / Cache
│   ├── Komfort- und Heizbewertung
│   └── Zielbildung / Ausführungspfade
│
├── Blind Control
│   ├── eigene Solar-/Umweltableitungen
│   └── Policy / Apply
│
└── Media-Provider + Title Classifier
        ↓
    Media State
        ↓
    Media Policy
        ↓
    Media Apply
        ↓
    reale Mediengeräte

Media-/Umbrella-UX liest bestehende Integrations- und HA-Projektionen.
Weitere YAML-/Legacy-Pfade bestehen daneben.
```

**Core Contracts ist heute noch nicht der zentrale Zugriffspunkt der gesamten Flotte.** Die Integrationen konsumieren überwiegend bestehende HA-Entities, Master-Attribute oder andere Integrationen.

Die Rückkopplung zwischen Core State und Media State ist real: Core State verwendet Media-Aktivität; Media State verwendet wiederum Core-State-Zustände für sein Away-Gating. Das ist nicht automatisch eine endlose Programmschleife, aber eine fachliche Abhängigkeitsschleife.

Belege: [Core-State-Coordinator](https://github.com/Levtos/benni-core-state/blob/c9408fa33f2914b1c5f7316738cb38b43add33b5/custom_components/benni_core_state/coordinator.py), [Media-State-Coordinator](https://github.com/Levtos/benni_media_state/blob/c371b3282d8bb3a5d35ad58b27179d5e3b4d7f6e/custom_components/benni_media_state/coordinator.py), [Climate-Coordinator](https://github.com/Levtos/benni_climate_policy/blob/0ad085b3359f4bef1401a82c9344b07f0e50a093/custom_components/benni_climate_policy/coordinator.py).

---

# C. Problemstellen

| ID | Befund | Einordnung | Konsequenz |
|---|---|---|---|
| C01 | Der Opening-Vertrag liefert bei fehlender Evidenz `unknown`; zentraler Batterie→offen-Pfad fehlt. | **CONTRACT CONFLICT** mit deiner bestätigten Vorgabe | Sicherheitssemantik muss zentral und sichtbar erfüllt werden. |
| C02 | Generischer Source Listener liest `state.state`; keine allgemeine Attributauswahl oder Einheitenkonvertierung. | **Belegte Funktionslücke** | Nicht jeder gewünschte Quellenwechsel ist heute allein durch Umbinden möglich. |
| C03 | Keine typisierte externe Producer-API für vollständige Owner-Ergebnisse. | **Belegte Architekturlücke** | Variante C ist nicht allein mit der vorhandenen Consumer API umgesetzt. |
| C04 | Core State und Media State beeinflussen sich gegenseitig. | **Belegte Rückkopplung** | Presence-Hold und Media-Away können dieselbe Evidenz gegenseitig bestätigen oder unterdrücken. |
| C05 | Media State setzt bei Away Kontext und Entertainment zurück, obwohl Geräte weiter aktiv sein können. | **Vermischung von Beobachtung und Zulässigkeit** | „Aktiv beobachtet“ und „für Verhalten freigegeben“ sind nicht mehr eindeutig getrennt. |
| C06 | Globaler Media-Kontext existiert in mehreren historischen beziehungsweise aktuellen Pfaden. | **Konkurrierende Bedeutungen/Legacy** | Consumer müssen wissen, welcher Kontext autoritativ ist. |
| C07 | Core-State-Wake liest Kalenderattribute; alter Wake Planner beschafft Zeiträume mit `calendar.get_events`. | **Belegte Quellenabweichung** | Gleiche Rechenlogik beweist keine vollständige Planungsparität. |
| C08 | Core-State-Wake kann Kalenderqualität bereits anhand vorhandener Konfiguration als frisch markieren. | **Belegte Qualitätslücke** | „Quelle konfiguriert“ wird teilweise mit „Daten frisch vorhanden“ verwechselt. |
| C09 | Climate hält eigene numerische Ersatzwerte; beim erneuten Lesen wird deren Zeitbezug erneuert. | **Abweichende Freshness-Semantik** | Eine stehengebliebene numerische Entity wird dadurch nicht zuverlässig als alte Messung erkannt. |
| C10 | Profiltrennung der Registry steht fest codierten Raum-/Gerätetopologien gegenüber. | **Profile-Coupling** | Zentrale Bindings allein machen Climate und Media nicht elternfähig. |
| C11 | Wake-Event-Deduplizierung und mehrere Media-Holds sind nur im Prozessspeicher. | **Restart-Risiko** | Neustart kann Ereignis- oder Ablaufsemantik verändern. |
| C12 | Ein Teil der übergeordneten Dokumentation beschreibt ältere Stände. | **Dokumentationsdrift** | Architekturentscheidungen dürfen nicht aus einzelnen alten Übersichtsseiten abgeleitet werden. |

## C01: Fenster und Batterie – der konkrete Konflikt

**Dein bestätigter Wunschzustand:** Ist die Batterie des Fensterkontakts leer, muss der wirksame Fensterzustand zentral als **offen** erscheinen und die Heizung blockieren.

**Belegter Implementierungsstand:** Im untersuchten Core-Contracts-Schema sind `opening_state` und `is_open` physische Felder, für die ein Ersatzwert abgelehnt wird. Fehlende beziehungsweise widersprüchliche Kontaktevidenz führt zu `unknown`. Die untersuchte Opening-Verarbeitung enthält keine entsprechende Batterieabhängigkeit.

Damit ist **die von dir geforderte zentrale operative Semantik nicht vollständig vorhanden**. Dass einzelne Consumer bei `unknown` sicher blockieren, ersetzt diesen Vertrag nicht.

**Technische Empfehlung:** Der zentrale Opening-Vertrag muss den wirksamen Sicherheitszustand einschließlich Ersatzwert liefern und gleichzeitig die ursprüngliche Beobachtung, Qualität und Ursache erhalten. Beispielsweise kann die wirksame Aussage „offen/blockierend“ sein, während die Diagnose „physischer Zustand nicht beobachtbar; Batterie leer“ lautet. Die genaue Schemaform ist eine spätere technische Ausarbeitung, keine erneute Frage nach deiner Sicherheitsentscheidung.

Beleg: [aktuelle Contract-Schemata](https://github.com/Levtos/benni-core-contracts/blob/def02cdf6db4daf24bb97ee36678eac090b9f863/custom_components/benni_core_contracts/contracts.py), [Published Opening Contract](https://github.com/Levtos/benni-core-contracts/blob/def02cdf6db4daf24bb97ee36678eac090b9f863/docs/published-opening-contract-v1.md).

## C02: Der zentrale „Symlink“ ist noch nicht vollständig

Das vorhandene Binding benennt eine konkrete Entity und ein semantisches Feld. Dieses Feld ist **kein allgemeiner Attributselektor**.

Ein Wechsel von einem Temperatur-Sensor zu beispielsweise einem Wetter-Entity-Attribut ist deshalb nicht automatisch derselbe technische Vorgang. Auch eine deklarierte Einheit im Schema führt noch keine Umrechnung durch.

Das ist eine begrenzte, sinnvolle Erweiterung der bestehenden Foundation – kein Grund, eine zweite Entity Registry oder eine neue State-Integration zu bauen.

Beleg: [Source Listener](https://github.com/Levtos/benni-core-contracts/blob/def02cdf6db4daf24bb97ee36678eac090b9f863/custom_components/benni_core_contracts/source_listener.py).

## Was ausdrücklich nicht als Doppelberechnung gelten sollte

- Ein Zustand darf über API, Diagnose und HA-Entity **mehrfach dargestellt** werden, solange er einmal berechnet wird.
- `media_context` und `activity_context` dürfen unterschiedliche Dimensionen ausdrücken. Problematisch wird es erst bei uneindeutiger Verwendung.
- Ein Apply-Status wie „Befehl ausgeführt“ ist keine zweite Berechnung des gewünschten Gerätezustands.
- Eine heizungsspezifische Bewertung ist nicht automatisch eine zweite Berechnung der Außentemperatur.

---

# D. Zielarchitektur und Variantenvergleich

## Empfohlener Datenfluss

**Der folgende Aufbau ist eine Empfehlung, kein bereits beschlossener Vertrag.**

```text
Physische HA-Quellen / externe Provider
                  │
                  ▼
       Quellenadapter und Bindings
       Typen / Attribute / Einheiten / Beobachtungszeit
                  │
                  ▼
       Kanonische Eingangs-Contracts
                  │
          ┌───────┴─────────────────────┐
          ▼                             ▼
  Interne, kleine Resolver      Externe fachliche Engines
  und bestehende Fusionen       Core State / Media State /
                                bestehender Wake-Owner
          │                             │
          │                     typisierte Producer-API
          └─────────────┬───────────────┘
                        ▼
              Kanonische Ergebnis-Contracts
              Owner / Version / Quality /
              Freshness / Lineage / Scope
                        │
                   Consumer API
                        │
           ┌────────────┼─────────────┐
           ▼            ▼             ▼
         Policies      UX          Diagnostics
           │
           ▼
       Apply / Geräteadapter
           │
           ▼
       reale Aktoren
```

**Wichtig:** Ein Owner darf seine eigenen Ergebnisse nicht über eine verdeckte Rückkopplung wieder als unabhängige Evidenz konsumieren. Der Graph muss auf Contract-Ebene geprüft werden, nicht nur auf Integrationsebene.

Ein Apply-Ergebnis darf als explizites Ereignis zurückgeführt werden, beispielsweise „TV für Schlafepisode X ausgeschaltet“. Es ist dann eine dokumentierte neue Beobachtung, kein beliebiger Rückkanal.

## Vergleich der drei Varianten

| Kriterium | A: getrennte Owner, heutige Austauschstruktur | B: alle State-Engines in CC | C: zentrale Contracts, externe komplexe Owner |
|---|---|---|---|
| Single Source of Truth | Möglich, aber derzeit lückenhaft | Möglich, nicht durch Zusammenlegung garantiert | Gut erreichbar durch verbindlichen Producer-Vertrag |
| Doppelte Logik | Muss einzeln beseitigt werden | Muss ebenfalls einzeln beseitigt werden | Muss ebenfalls einzeln beseitigt werden |
| Ownership | Sichtbare Integrationsgrenzen | Muss intern streng erzwungen werden | Fachlicher Owner bleibt sichtbar |
| Kopplung | Viele direkte Consumer-Abhängigkeiten | Weniger Integrations-, mehr Runtime-Kopplung | Consumer hängen am Contract |
| Zyklen | Heute teilweise verborgen | Zentral prüfbar, falls alle Reads deklariert | Über Producer- und Contract-Graph prüfbar |
| Testbarkeit | Pure Engines teilweise schon gut testbar | Gute isolierte Tests möglich | Bestehende Engine-Tests plus Contract-Tests |
| Erweiterbarkeit | Neue Integration oft naheliegend | Gefahr wachsender zentraler Plattform | Neue Owner ohne neue Consumer-Anbindung |
| Restart/Restore | Unterschiedliche Mechanismen | Gemeinsamer Neustart trifft alle Engines | Unterschiedliche Owner, vereinbarte Recovery-Verträge |
| Fehlerisolation | Logische Modulgrenzen | Größerer gemeinsamer Ausfallbereich | Ownerfehler können als Contract-Ausfall begrenzt werden |
| Performance | Zusätzliche Entity-Wege | Potenziell weniger Transport | Potenziell weniger Consumer-Transport |
| Performance-Nachweis | Nicht gemessen | Nicht gemessen | Nicht gemessen |
| HA-Startup | Viele Einzelabhängigkeiten | Zentraler kritischer Startpfad | Explizite Producer-Readiness erforderlich |
| Diagnose/Lineage | Fragmentiert | Zentral gut möglich | Zentral gut möglich, wenn Producer Herkunft liefern |
| Quality/Freshness | Unterschiedliche Definitionen | Einheitlich modellierbar | Einheitlich modellierbar |
| Schema-Evolution | Direkte Entity-Annahmen verbreitet | Gefahr gemeinsamer Versionszwänge | Contract-Versionen entkoppeln Consumer |
| Migration | Niedrigster kurzfristiger Aufwand | Höchster Umfang und Zustandsrisiko | Schrittweise pro Contract |
| UX | Mehrere Konfigurationsorte | Eine Oberfläche gut möglich | Ebenfalls eine Oberfläche möglich |
| Wartbarkeit | Viele Verbindungen | Große zentrale Codebasis | Zusätzliche API-Grenze, dafür begrenzte Verantwortungen |
| Monolith-Risiko | Niedrig in CC | Hoch ohne interne Architekturdisziplin | Mittel |
| HA-Entity-Transport | Bleibt weitgehend bestehen | Kann stark sinken | Kann für fachliche Consumer stark sinken |
| Profile | Unterschiedliche Reife | Einheitliche Infrastruktur möglich | Einheitlicher Contract-Scope, Owner müssen mitziehen |

**Bewertung:** Variante C löst dein Hauptziel, ohne gleichzeitig sämtliche langlebigen Zustände migrieren zu müssen. Variante B ist langfristig vertretbar, wenn sie nachweislich Betriebs- und Wartungsvorteile bringt. Der aktuelle Code liefert diesen Nachweis noch nicht.

Außerdem wäre B eine ausdrückliche Architekturänderung gegenüber den heutigen Integrationsgrenzen und bisherigen Entscheidungen. Sie darf nicht als bloße interne Registry-Erweiterung umgesetzt werden. Das betrifft auch die erneute Bewertung von [Control #37](https://github.com/Levtos/control/issues/37) und die Backend-Grenzen aus [ADR 0001](https://github.com/Levtos/control/blob/28bf515449187c067fe5e23e9c383c0aa0523236/docs/adr/0001-ux-frontend-standard.md).

---

# E. Empfohlenes Core-Contracts-Metamodell

## Was heute vorhanden ist

Die untersuchten Schemata decken diese Bereiche ab:

- `room_climate`
- `opening`
- `weather_environment`
- `technical_device`
- `presence`

Die vorhandenen Fusionen bieten eine begrenzte Auswahl fest implementierter Auswahl-, Bool- und Opening-Strategien. Sie sind kein allgemeiner Resolver für beliebige heterogene Inputs.

Ein Media-Contract in einem API-Beispiel ist daher **kein Nachweis eines implementierten Media-Schemas**.

Belege: [Schemata](https://github.com/Levtos/benni-core-contracts/blob/def02cdf6db4daf24bb97ee36678eac090b9f863/custom_components/benni_core_contracts/contracts.py), [Registry-Lastenheft](https://github.com/Levtos/benni-core-contracts/blob/def02cdf6db4daf24bb97ee36678eac090b9f863/docs/lastenheft-registry-exchange-layer-v1.md).

## Benötigte fundamentale Modelle

Nicht jeder Algorithmus benötigt einen eigenen Runtime-Typ.

| Typ | Zweck | Stateful? | Inputs | Output | Persistenz | Reales Beispiel |
|---|---|---:|---|---|---|---|
| **Source Adapter + Binding** | Externe Quelle technisch korrekt lesen und normalisieren | Teilweise | Entity-State, Attribut, Provider | Beobachtung mit Zeit und Qualität | Binding; gegebenenfalls Adapter-Cache | Wettertemperatur aus Attribut |
| **Contract Definition + Instanz** | Bedeutung, Typ, Version und räumlichen/personellen Scope festlegen | Nein | Schema und Scope | Stabile fachliche Schnittstelle | Registry/Revisionen | Temperatur eines bestimmten Raums |
| **Pure Resolver** | Deterministische Ableitung | Nein | Typisierte Contracts und Parameter | Abgeleiteter Zustand | Konfiguration, kein Entscheidungszustand | Taupunkt, Tagesphase, Klassifikation |
| **Stateful Engine** | Vergangenheit und Übergänge verarbeiten | Ja | Snapshots, Ereignisse, Zeit | Zustand und Übergänge | Versionierter Engine-State | Presence-Übergang, Bio, Media-Arbitration |
| **Planner** | Zeitbezogene Vorschau und nächsten relevanten Zeitpunkt berechnen | Berechnung möglichst nein | Kalender, Regeln, Datum, Konfiguration | Plan mit Gültigkeitszeitraum und Begründung | Regeln; externe User-Zustände separat | Wake-Plan |
| **External Producer** | Ergebnis eines anderen fachlichen Owners registrieren und liefern | Ownerabhängig | Typisierte Owner-Ergebnisse | Kanonischer Contract | Ownership-/Versionsdaten; State beim Owner | Media State → CC |
| **User-State-/Command-Grenze** | Explizite Nutzerentscheidungen mit Herkunft und Lebensdauer verwalten | Ja | Zulässiger Nutzerbefehl | Nutzerzustand, nicht Messwert | Dauerhaft, gegebenenfalls mit Ablauf | Manueller Schlafbefehl |
| **Domain Event** | Einmalige fachliche Übergänge mit Identität transportieren | Zustellung ja | Engine-Übergang/Apply-Evidence | Ereignis mit Episode und Herkunft | Je nach Wiederholungsrisiko | Wake-Fenster betreten, TV-Aus bestätigt |
| **Projection/View Model** | Bestehende Ergebnisse anzeigen | Nein | Contracts | UX/HA-/Diagnoseansicht | Allenfalls Präsentationseinstellungen | Live-Status, kompakte Zusammenfassung |

### Was keine zusätzliche fundamentale Engine-Art braucht

- Formel, Klassifikation, Aggregation, Score und Schwellenwert sind **Resolver-Algorithmen**.
- Debounce, Hysterese, Hold, Edge Detection und zeitliche Fenster sind **Bausteine einer Stateful Engine**.
- Regelprioritäten und Schedule-Suche gehören zum **Planner**.
- Fan-in, Fan-out und mehrstufige Ableitungen gehören zum **Dependency-Modell**.
- Quality, Freshness, Lineage und Safety sind **Querschnittsverträge**, keine selbstständigen Fachintegrationen.

Ein „Confidence“-Wert ist außerdem nicht automatisch eine statistisch kalibrierte Wahrscheinlichkeit. Seine Bedeutung muss definiert werden.

## Unverzichtbare Querschnittsfähigkeiten

1. **Explizite Input-/Output-Abhängigkeiten:** keine versteckten `hass.states`-Reads innerhalb fachlicher Resolver.
2. **Scope:** Installation/Profil, Person, Raum und Geräteinstanz dürfen nicht zusammenfallen.
3. **Optionalität:** Fehlende optionale Fähigkeiten müssen von ausgefallenen Pflichtquellen unterscheidbar sein.
4. **Zeitmodell:** Beobachtungszeit, Empfangszeit, Berechnungszeit und Gültigkeitsintervall getrennt.
5. **Konsistente Auswertung:** zusammengehörige Outputs mit nachvollziehbarer Eingangsrevision beziehungsweise Auswertungsversion.
6. **Persistenzmigration:** Registry-LKG ist kein Ersatz für Bio-, Hold- oder Event-State.
7. **Producer-Lifecycle:** Start, bereit, degradiert, beendet; einschließlich stale gewordener Producer.
8. **Graphvalidierung:** auch externe Producer-Abhängigkeiten, nicht nur Fusion-zu-Fusion.
9. **Begrenzte Ausführung:** keine blockierenden Netzwerkoperationen in zentralen Resolvern; Fehler und Rückstau pro Owner begrenzen.
10. **Qualitätsfortpflanzung:** Herkunft der tatsächlich verwendeten Inputs erhalten. Ein frischer Rechenlauf macht alte Eingaben nicht frisch.

**Diese Fähigkeiten sollten als Architekturvertrag beschrieben werden. Es ist nicht notwendig, sie alle vor dem ersten Consumer-Durchstich vollständig als Universalframework zu implementieren.**

---

# F. Canonical Truth Catalog

Die Bezeichnungen in dieser Tabelle sind fachliche Namen, **keine neu beschlossenen Contract-IDs**.

Für die Inventur stehen:

- **Fakt:** beobachtete Information.
- **Ableitung:** aus Evidenz berechneter Zustand.
- **Plan:** berechnete zeitliche Planung.
- **Policy:** gewünschte Folge beziehungsweise Zielbewertung.
- **Projektion:** Darstellung einer bereits vorhandenen Wahrheit.

## F1. Bedeutung, Berechnung und Ownership

| Wahrheit | Heutiger Owner / Inputs / Berechnung | Art | Empfohlener kanonischer Owner |
|---|---|---|---|
| Tatsächliche Außentemperatur | HA-Quelle; Climate wählt Quellen und Ersatzwerte | Fakt | Quellenadapter über vorhandenen Environment-Contract |
| Gefühlte Außentemperatur | Climate Weather Resolver; Providerwert oder Temperatur/RH/Wind-Formel | Ableitung | Ein benannter Environment-Resolver; Methodik explizit |
| Heizungswirksame Außentemperatur | Climate: reale/gefühlte Temperatur, Wetter, Lux, Forecast, Bodenmodell | **Policy-Merkmal** | Climate Policy |
| Raumtemperatur und RH | Physische Quellen, Climate, teilweise Master-/Contract-Pfade | Fakt | Raumbezogene Contracts |
| Gefühlte Raumtemperatur | Climate mit Temperatur, Feuchte und heizungsspezifischem Offset | Ableitung mit Policy-Einfluss | Zunächst Climate; nicht ungeprüft als neutrale Physik veröffentlichen |
| Taupunkt / absolute Feuchte | Bathroom-Berechnung aus Temperatur und RH | Physikalische Ableitung | Reiner Environment-Resolver, wenn tatsächlich gemeinsam benötigt |
| Feuchteanstieg / Trend | Bathroom-Vergleich von Messwerten und Zeitpunkten | Zeitliche Ableitung | Ein zeitlich eindeutig definierter Resolver |
| Forecast | Climate: Wettervorhersage, Cache, Auswahl relevanter Stunden/Tage | Zeitreihe/Prognose | Provideradapter mit Prognose-Contract |
| Sonnenstand / Einfallswinkel | Blind Control: Sonne, Standort, Fenstergeometrie | Ableitung | Standort-/flächenbezogener Resolver |
| Heat-/Glare-Bewertung | Blind Control: Strahlung, Schwellen, Hysterese | **Policy-Merkmal** | Blind Policy |
| Physischer Opening-Zustand | Raw-Kontakte, Core Devices/YAML, CC-Pilot | Fakt/Ableitung | Zentraler Opening-Owner |
| Wirksamer Opening-Sicherheitszustand | Heute unterschiedliche Fallbackpfade | Sicherheitsableitung | Derselbe Opening-Owner; bestätigte Ersatzwertsemantik |
| Presence einer Person | Core State: Tracker, WLAN, Proximity, Profilregeln | Stateful Ableitung | Core-State-Presence-Engine |
| Household Presence | Core State mit Haushaltsquelle/Personenlogik | Ableitung | Expliziter Household-Owner in Core State |
| Raumbelegung | Verschiedene Motion-/Raum-/Policy-Signale | Nicht einheitlich | Erst gemeinsame Bedeutung festlegen; nicht Household Presence umbenennen |
| Presence-Band und Übergang | Core State: Distanz, vorherige Werte, Hysterese | Stateful Ableitung | Presence-Engine |
| Bio/Sleep/Waking | Core State: Episoden, Evidenz, Zeitfenster, manuelle Befehle | Stateful Ableitung | Bio-Engine in Core State |
| Tagesphase | Core State: Datum, lokale Uhrzeit, saisonale Phasen | Reine Ableitung | Bestehender Day-State-Resolver |
| Tageskontext/Feiertag | Core State beziehungsweise Wake-Kalenderpfade | Ableitung | Gemeinsame definierte Kalendergrundlage; eindeutige Projektionen |
| Personenaktivität | Core State: Bio, Media-Aktivität, PC-/weitere Signale und Prioritäten | Ableitung mit Holds | Activity-Owner in Core State |
| Geräteaktivität / Power / Availability | Core Devices; teilweise Raw-Neuinterpretation in Consumern | Fakt/Ableitung | Geräteadapter beziehungsweise technischer Contract-Owner |
| Media-Foreground | Media State: Quellenarbitration und Geräteinformationen | Stateful Ableitung | Media-State-Engine |
| Media-Kontext und Subkontext | Media State: Geräte, Apps, Klassifikation, Arbitration | Stateful Ableitung | Media-State-Engine |
| Gaming-Klasse | Title Classifier, von Media State interpretiert | Klassifikation | Title Classifier; Media konsumiert definierte Bedeutung |
| Headset-Kontext | Media State teilweise aus Gaming-Klasse abgeleitet | Ableitung | Media; deutlich von physischer Headset-Verbindung unterscheiden |
| Private-Kontext | Media State: Klassifikation/Stash/PC/Denon beziehungsweise manueller Zustand | Ableitung + User State | Media-Owner mit getrennten Quellen |
| Quiet-Anforderung | Media State: Anruf/Tür/Bio/Klassifikation beziehungsweise expliziter Input | Teilweise **Policy** | Beobachtung beim State-Owner; gewünschte Ruhe bei zuständiger Policy |
| Wake-Plan / Next Wake | Core State intern und alter Wake Planner | Plan | Bestehender Core-State-Wake-Owner nach vollständigem Cutover |
| Wake-Ereignis | Alter Wake Planner: Fenster betreten | Domain Event | Genau ein Event-Owner mit Deduplizierungsvertrag |
| Soll-Audiopfad / HomePod-Pause / Lautstärke | Media Policy | Policy | Media Policy |
| Heiz-Sollwerte / Lüftungs-/Trocknungsziele | Climate Policy | Policy | Climate Policy |
| Rollo-Ziele | Blind Policy/Control | Policy | Zuständiger Blind-Policy-Owner |
| Licht-Ziele | Light Policy; Scene Presets setzt um | Policy | Bestehende Light-/Scene-Grenze |
| Ausführungsstatus | Apply-Schichten | Technische Beobachtung | Jeweiliger Apply-Owner |
| Titel / Artwork / Transportfähigkeiten | Provider, Classifier, Media-/UX-Projektionen | Providerdaten/Capabilities | Externe Owner; gemeinsame typisierte Projektion bei Bedarf |

Climate-Belege: [Weather Resolver](https://github.com/Levtos/benni_climate_policy/blob/0ad085b3359f4bef1401a82c9344b07f0e50a093/custom_components/benni_climate_policy/weather_resolver.py), [Bathroom](https://github.com/Levtos/benni_climate_policy/blob/0ad085b3359f4bef1401a82c9344b07f0e50a093/custom_components/benni_climate_policy/bathroom.py), [Climate-Logik](https://github.com/Levtos/benni_climate_policy/blob/0ad085b3359f4bef1401a82c9344b07f0e50a093/docs/climate_policy_logic.md).

## F2. Laufzeit, Qualität, Persistenz und Transport

Diese Tabelle ergänzt die zwölf Inventurmerkmale für die oben genannten Familien. Die nachfolgenden Core-/Media-Matrizen differenzieren deren einzelne Felder.

| Familie | Stateful / Persistenz heute | Quality / Freshness / Fallback heute | Consumer und Transport | Konkurrenz |
|---|---|---|---|---|
| CC-Basiswerte | Registry/Revisionen/LKG persistent; Beobachtungen davon getrennt | Zentral modelliert; physischer Opening-Ersatzwert eingeschränkt | Consumer API, begrenzte HA-Projektion | Neben bestehenden Raw-/Master-Pfaden |
| Climate-Wetter | Cache und historische Laufzeitwerte | Eigene Ersatzketten; Messalter nicht durchgängig erhalten | Climate intern, HA-Diagnose-/State-Entities | Mehrere Wetter-/Temperaturinterpretationen |
| Bathroom-Metriken | Vorwerte, Budgets und Timer im Coordinator | Fehlende Werte und Ersatzpfade lokal | Vor allem Climate intern | Keine pauschal belegte zweite identische Formel |
| Blind-Umwelt | Teilweise rein, teilweise Hysterese-/Pending-State | Eigene Providerqualität und Stabilisierung | Blind intern und Diagnose | Ähnliche Wetterinputs; fachlich andere Flächenbewertung |
| Presence/Bio | Core-State-Persistenz und Edge-Bookkeeping | Quellenabhängige Regeln, Retain/Holds; keine CC-weite Lineage | Über HA-Entities an mehrere Policies | Media-Rückkopplung; Raw-Fallbacks |
| Day/Activity | Day weitgehend rein; Activity mit Stateful-Einfluss | Zeit-/Quellenregeln im Core-State-Kontext | HA-Entities | Media-Activity ist eine andere Dimension, muss so konsumiert werden |
| Media | Mischung aus RAM, Restore und verzögertem Speichern | Debounce, Grace, Holds, Away-Gating | HA-Entities/Attribute an Policy und UX | Core-Devices-/Legacy-Kontexte, Raw-Neuinterpretation |
| Title Classifier | Katalog-/Konfigurationspersistenz, Providerzustand | Gemeinsame Effective-Berechnung; Offline-/Titel-Fallback | Media und weitere Consumer | Enum-/Bedeutungsdrift möglich |
| Wake | Core-State-Stores und alter Planner-Store; alter Kalendercache RAM | Unterschiedliche Beschaffung und Qualitätsdarstellung | Entities, Vorschau, Events | Zwei noch relevante Berechnungs-/Transportpfade |
| Policy-Ergebnisse | Je nach Policy Holds, Overrides und Historie | Bewusst policyabhängig | Policy→Apply, HA-/UX-Projektionen | Konkurrenz nur bei mehreren Entscheidern desselben Ziels |
| Apply | Timer, Cooldowns, Recovery-/Episode-State | Gerätebestätigung, Retry, Settling | Reale Services und Status | Direkte YAML-/Legacy-Startwege können konkurrieren |

## Wichtige Climate-Differenzierung

Eine Berechnung gehört nicht allein deshalb in Core Contracts, weil sie einen Temperaturwert ausgibt.

Im aktuellen Climate-Code:

- Die gefühlte Außentemperatur kann eine nachvollziehbare meteorologische Ableitung sein.
- Die **heizungswirksame** Außentemperatur kombiniert zusätzliche heizungsbezogene Korrekturen.
- Das Bodenmodell verwendet unter anderem historische und prognostizierte Temperaturen zur Heizbewertung.
- Der gespeicherte „gestrige“ Wert ist dabei nicht automatisch ein Tagesmittel.
- Die gefühlte Raumtemperatur ist durch diese Bewertung beeinflusst.

**Empfehlung:** Messwerte und wiederverwendbare physikalische Ableitungen zentralisieren. Heizungsbezogene Bewertungsgrößen bei Climate belassen und verständlich benennen.

Damit muss Climate nicht „nichts berechnen“. Es soll keine fremde fachliche Wahrheit erneut berechnen; seine eigenen Heizentscheidungen und dafür notwendigen Bewertungsgrößen bleiben seine Aufgabe.

---

# G. Core-State-Migrationsmatrix

Die Buchstaben entsprechen deinem gewünschten Raster:

**A** Contract/Fusion · **B** reiner Resolver · **C** Stateful Engine · **D** Planner · **E** externer Owner · **F** Policy · **G** Legacy/redundant.

Dabei bezeichnet B/C/D die **Art der Berechnung**. Meine empfohlene erste Betriebsform für diese Core-State-Berechnungen bleibt **E: Core State publiziert nach Core Contracts**.

Grundlage: [Datenmodell](https://github.com/Levtos/benni-core-state/blob/c9408fa33f2914b1c5f7316738cb38b43add33b5/custom_components/benni_core_state/models.py), [Logik](https://github.com/Levtos/benni-core-state/blob/c9408fa33f2914b1c5f7316738cb38b43add33b5/custom_components/benni_core_state/logic.py), [Coordinator](https://github.com/Levtos/benni-core-state/blob/c9408fa33f2914b1c5f7316738cb38b43add33b5/custom_components/benni_core_state/coordinator.py).

| Aktueller Wert/Funktion | Tatsächliche Bedeutung | Klasse | Empfohlener zukünftiger Ort |
|---|---|---|---|
| `presence_personal` | Personenanwesenheit aus profilabhängigen Quellen; Retain bei Evidenzlücken | C/E | Presence-Engine → Personen-Contract |
| `presence_household` | Haushaltsanwesenheit | A/B/E | Household-Contract mit eigener Definition |
| `presence_band` | Distanzbereich mit Übergangslogik | C/E | Presence-Engine |
| `presence_transition` | Kommen/Gehen/weitere Übergangsbedeutung aus Historie | C/E | Presence-Engine |
| `presence_effective` | Wirksame Anwesenheit einschließlich Activity-Hold | C/E | Eigener eindeutig beschriebener Presence-Ausgang |
| `presence_effective_transition` | Übergang der wirksamen Anwesenheit | C/E | Derselbe Owner |
| `effective_reason` | Begründung des wirksamen Zustands | A | Metadatum desselben Contracts |
| `effective_assumed` | Zustand enthält Annahme | A | Qualitäts-/Provenienzmerkmal |
| `effective_hold_strength` | Stärke des Activity-Holds | A | Engine-Diagnose, kein zweiter Presence-Owner |
| `effective_source_activity` | Verwendete Aktivität | A | Lineage |
| `effective_hold_active` | Hold läuft | C/A | Stateful Ergebnis plus Diagnose |
| `preheat_active` | Rückkehr-/Presence-basierte Vorheizfreigabe | **C/F-Grenzfall** | Bedeutung prüfen: Rückkehrfenster bei Presence; Heizfreigabe bei Climate |
| `preheat_source` | Herkunft dieses Ergebnisses | A | Beim endgültigen Owner |
| `preheat_started` | Beginn des Fensters | C/A | Beim endgültigen Owner |
| `bio_state` | Awake/Provisional Sleep/Sleep/Waking | C/E | Bio-Engine |
| Provisional Sleep | Eigenständiger Zustand mit Evidenz-/Episodenlogik | C/E | Bio-Engine; bestehende Heizabsenkung nicht neu verhandeln |
| bestätigter/inferierter Sleep | Zustand mit Herkunft und Bestätigung | C/E | Bio-Engine |
| Waking | Übergang mit spezifischer Wake-Evidenz | C/E | Bio-Engine |
| `last_sleep_start` | Schlafbeginn | C/A | Persistente Bio-Episode |
| `last_awake_start` | Wachbeginn | C/A | Persistente Bio-Episode |
| `last_provisional_sleep_start` | Beginn der vorläufigen Schlafphase | C/A | Persistente Bio-Episode |
| `last_waking_start` | Beginn Waking | C/A | Persistente Bio-Episode |
| `sleep_reference_start` | Gemeinsame Episodenreferenz | C/A | Bio-Episode; für Apply-Evidence erhalten |
| `sleep_source`, `sleep_confirmed` | Herkunft und Bestätigung | A | Bio-Provenienz |
| `inferred_tv_off_at` | Zugeordnete TV-Off-Evidence | C/A | Explizite Episode-/Event-Evidence |
| `observed_signal_states` | Restartfeste Flankenbuchhaltung | C | Privat in Bio-Engine |
| `indicator_active_since` | Beginn relevanter Aktivität | C | Privat in Bio-Engine |
| `opening_states` | Vorzustände für Opening-Evidenz | C | Privat; künftig typisierte Opening-Inputs |
| `minimum_sleep_minutes`, `provisional_lead_minutes` | Fachliche Parameter | Konfiguration | Versionierte Bio-/Profilkonfiguration |
| Presence-Kandidaten, Startzeiten, letzte Home/Away-Zeiten | Stabilisierung und Restore | C | Privat in Presence-Engine |
| letzte Proximity-Distanz und Zeit | Übergangs-/Trendhistorie | C | Privat in Presence-Engine |
| `day_state` | Neunphasiger, saisonaler Tageszustand | B/E | Bestehender Day-State-Resolver; später leicht intern hostbar |
| `day_context` | Kalender-/Wochentagskontext | B/E | Definierter Zeit-/Kalender-Resolver |
| `activity_state` | Priorisierte Personenaktivität | B/C/E | Activity-Engine; Media-Eingang entkoppeln |
| `master_context` | Zusammenfassung mehrerer bereits berechneter Achsen | A/G | Nur Anzeige-/Compatibility-Projektion |
| `wake_state` | Planungszustand | D/E | Bestehender interner Wake-Owner |
| `next_wake` | Nächster geplanter Zeitpunkt | D/E | Wake-Plan-Contract |
| `wake_needed` | Aktuelles Wake-Fenster | D/B/E | Projektion des Plans; nicht gleich `waking` |
| `holiday_active` | Feiertags-/Planungskontext | B/E | Kalendergrundlage/Planprojektion eindeutig benennen |
| interner Sleep-Plan / Sleep-Window | Aus Wake-Plan und Parametern abgeleitetes Zeitfenster | D/B/E | Beim bestehenden Core-State-Planungsowner |
| `live_status` | Deutscher Anzeigetext | A | View Model, keine Business-Eingangsquelle |
| `attrs` / Trace-Felder | Erklärung vorhandener Ergebnisse | A | Contract-Metadaten/Diagnose |
| Apply Readiness | Technischer Startschutz, kein Personenstatus | Technischer C-Mechanismus | Lifecycle-/Readiness-Vertrag; Apply behält eigenes Gate |

### Zwei besonders wichtige Ergebnisse

**`master_context` benötigt keine eigene kanonische Berechnung.** Als lesbare Zusammenfassung kann es bleiben. Consumer sollten daraus keine einzelnen Business-Zustände zurückparsen.

**Presence darf nicht mit Media zu einer gegenseitigen Bestätigung verschmelzen.** Beobachtete Medienaktivität kann eine deklarierte Evidenz sein. Ein bereits aufgrund von Presence unterdrückter Media-Zustand sollte nicht anschließend als unabhängige Presence-Evidenz gelten.

Die aktuelle Bio-Episodenlogik enthält bereits wertvolle restartfeste Flanken- und Herkunftsbehandlung. Diese sollte erhalten werden. Sie ist kein unnötiger Ballast, der durch eine generische Fusion ersetzt werden könnte. Relevante Weiterentwicklung: [Core State #59](https://github.com/Levtos/benni-core-state/issues/59).

---

# H. Media-State-Migrationsmatrix

Grundlagen: [Media-State-Logik](https://github.com/Levtos/benni_media_state/blob/c371b3282d8bb3a5d35ad58b27179d5e3b4d7f6e/custom_components/benni_media_state/logic.py), [Coordinator](https://github.com/Levtos/benni_media_state/blob/c371b3282d8bb3a5d35ad58b27179d5e3b4d7f6e/custom_components/benni_media_state/coordinator.py).

| Wert/Funktion | Aktuelle Bedeutung / Besonderheit | Klasse | Empfehlung |
|---|---|---|---|
| `media_context` / `context` | Medienkontext einschließlich Arbitration und Away-Einfluss | C/E | Kanonischer Media-Contract; Beobachtung und Eligibility trennen |
| `media_subcontext` / `subcontext` | Unterklassifikation, teils mit Sticky-Verhalten | B/C/E | Beim Media-Owner |
| `media_device` / `device` | Für Kontext ausgewähltes Gerät | C/E | Gleiche Auswertungsversion wie Kontext |
| Media-Source / Foreground | Quelleneigentümerschaft aus Geräte-/LG-/App-Evidenz | C/E | Expliziter Ausgang, nicht aus Playback rekonstruieren |
| `gaming_source` | Gaming-Quelle | B/C/E | Media-Projektion mit Classifier-Lineage |
| `platform` | Ausgewählte Gaming-Plattform | B/C/E | Media-Contract |
| `headset_active` | Teilweise aus Gaming-Klasse abgeleiteter Kontext | B/E | Von tatsächlicher Headset-Verbindung unterscheiden |
| `entertainment_active` | Aus Kontext abgeleitet, durch Away-Gate beeinflusst | B/F-Grenze | Beobachtetes Entertainment getrennt von Verhaltensfreigabe |
| `active_reasons` | Begründungen | A | Lineage/Diagnose |
| `quiet_mode` | Mischung aus Kontextbeobachtung und Ruheanforderung | B/F | Beobachtung zentral; gewünschte Ruhe zur Policy |
| `quiet_reason` | Begründung | A | Beim jeweiligen Ergebnis |
| `presence_state`, `presence_source` | Übernommene Core-State-Eingänge | A | Input-Lineage; keine zweite Presence-Wahrheit |
| `away_gate` | Verhaltensbezogene Sperre | F | Als Eligibility/Policy-Gate kenntlich machen |
| `private_time_active` | Automatische oder manuelle Private-Bedeutung | B/C/E | Media-Owner, Quellen getrennt |
| `private_time_source` | Automatisch/manuell | A | Provenienz |
| `private_time_reason` | Ursache | A | Diagnose |
| `private_time_blocked_reason` | Warum Voraussetzungen fehlen | A | Diagnose/Eligibility |
| `activity_context` | Aktivitätsprojektion, unter anderem Music | B/E | Als andere Dimension als `media_context` dokumentieren |
| App-/Streaming-Klassifikation | Provider-/App-Evidenz wird eingeordnet | B/E | Code-definierter Resolver |
| Device Matrix | Sammlung lokaler Geräteinformationen | A/B | Technische Contracts konsumieren; Matrix als Diagnose |
| Gaming-Klasseninterpretation | Enum wird in grind/headset/default übersetzt | B/E | Versionierten Classifier-Vertrag konsumieren |
| Pre-Apple-TV-Zustand | Vorherige Aktivität für Rückkehrverhalten | C | Privat im Media-Owner |
| Sticky-Gaming-Kontext | Kurzfristiger Erhalt | C | Privat mit expliziter Ablauf-/Restore-Semantik |
| manueller Private-Schalter | Nutzerzustand mit Restore/TTL | C | User-State-Vertrag |
| Manual Nudge | Kurzlebiger Nutzerimpuls | Event/C | Expliziter Intent, nicht Gerätebeobachtung |
| Titel/Artist/Artwork | Projektionen externer Provider | A/E | Nicht erneut fachlich berechnen |
| Debounce/Grace/Degraded-Hold | Stabilisierung | C | Jeweils beim Owner des Übergangs |

## Beobachtung, Foreground und Policy müssen auseinandergehalten werden

Diese Aussagen sind nicht gleichbedeutend:

```text
Apple TV ist eingeschaltet.
Apple TV spielt.
Apple TV besitzt den Bildschirm-Foreground.
Streaming-Inhalt ist klassifiziert.
Der TV soll eingeschaltet werden.
HomePods sollen pausieren.
```

Die ersten vier sind unterschiedliche Beobachtungen beziehungsweise Ableitungen. Die letzten beiden sind Policy-Ergebnisse.

**Insbesondere ist eine erkannte Streaming-Aktivität kein ausreichender positiver Auftrag, einen bewusst ausgeschalteten TV erneut einzuschalten.**

## Aktuelle Fehler und Timer

| Mechanismus | Heutiger Owner | Beobachtung |
|---|---|---|
| Media-Debounce | Media State | Rund 2 Sekunden; zusätzliche Re-Arm-Logik kann die Reaktion verlängern |
| TV-only Guard | Media State | Rund 20 Sekunden |
| LG-Source-Grace | Media State | 5 Sekunden für den konkret definierten Verlustfall; kein beliebiger allgemeiner TTL |
| PS5-Degraded-Hold | Media State | Rund 90 Sekunden |
| Away-Stabilisierung | Media State | Rund 25 Sekunden |
| Private-Restore/TTL | Media State | Wiederhergestellter Zustand erhält erneut Laufzeit; nicht gleich absoluter ursprünglicher Ablauf |
| Pause-Debounce | Media Apply | Zusätzliche Verzögerung in der Ausführungskette |
| Recovery/Settling | Media Apply | Gerätebezogene Zustände; gehören weiterhin zu Apply |

Die aktuellen GitHub-Nachweise zeigen die Wirkung mehrerer Schichten:

- Im untersuchten Übergang war LG bereits auf Apple-TV-Quelle.
- Apple-TV-Playback folgte später.
- Media State klassifizierte anschließend Streaming.
- Policy und Apply reagierten nahezu unmittelbar auf diesen späteren State.
- Die HomePods erreichten erst danach den inaktiven Zustand.

Das belegt, dass „Pause zu spät“ nicht allein ein Frontend- oder ein einzelnes Apply-Timerproblem ist. Die frühe Foreground-Evidenz muss mit ihrer eigenen Bedeutung verfügbar sein. Belege: [Media State #24](https://github.com/Levtos/benni_media_state/issues/24), [Media Apply #47](https://github.com/Levtos/benni_media_apply/issues/47).

Der TV-Wiederstart ist separat dokumentiert: [Media Apply #46](https://github.com/Levtos/benni_media_apply/issues/46). Eine zentrale Runtime allein würde die falsche Intent-Ableitung nicht korrigieren.

## Rechtfertigt Media State eine eigene Integration?

**Heute: ja, als bestehender komplexer Owner mit eigener Migration und eigenem Lifecycle.**

**Grundsätzlich dauerhaft zwingend: nein.** Seine Engine könnte später intern gehostet werden, wenn Inputs, Outputs, Persistenz, Zeitverhalten und Fehlergrenzen sauber herausgelöst sind.

Die zuerst notwendige Arbeit ist dieselbe, egal ob der spätere Host Media State oder Core Contracts heißt. Deshalb bringt ein frühzeitiger Repository-Umzug derzeit wenig zusätzlichen Nutzen.

---

# I. Wake-Planner-Entscheidung

## Tatsächlicher Stand

Der alte Wake Planner enthält:

| Bestandteil | Einordnung |
|---|---|
| Personen- und Regelkonfiguration | Konfiguration |
| Wochentage, Intervalle, Zyklen, Prioritäten | Planner |
| Kalenderbeschaffung und Cache | Quellenadapter |
| Feiertagsauflösung | Eingangsaufbereitung/Resolver |
| `next_wake`, `wake_state`, `wake_needed` | Plan und Projektionen |
| Skip/Override im alten Planner | User-owned State |
| `wake_planner_wake_triggered` | Domain Event |
| Vorschau und Panel | UX |
| Services zur eigenen Konfiguration | Command-API, keine Geräte-Actuation |
| `calendar.get_events` | Lesende Quellenbeschaffung, keine Kalenderänderung |

Core State besitzt bereits eine interne Wake-Planberechnung. Der aktuelle Coordinator verwendet interne Plan-/Sleep-Window-Ergebnisse; die pauschale Aussage „alles nur Shadow, ohne Einfluss“ wäre für den heutigen Code falsch.

Gleichzeitig besteht eine relevante Lücke: Die dortige Kalenderbeschaffung entspricht nicht dem Zeitraumsabruf des alten Planners.

Belege: [Core-State-Wake-Berechnung](https://github.com/Levtos/benni-core-state/blob/c9408fa33f2914b1c5f7316738cb38b43add33b5/custom_components/benni_core_state/wake_planning.py), [Wake Calendar Source](https://github.com/Levtos/ha_wake_planner/blob/9f94d2a80a79219db2c557abe8f353c861777d39/custom_components/wake_planner/calendar_source.py), [Wake Calendar Cache](https://github.com/Levtos/ha_wake_planner/blob/9f94d2a80a79219db2c557abe8f353c861777d39/custom_components/wake_planner/calendar_cache.py).

## Bewertung der vier Möglichkeiten

| Option | Bewertung |
|---|---|
| 1. Gesamten Wake Planner nach CC integrieren | Nicht empfohlen: zieht alte UI, Commands, Cache und Legacy-Lifecycle in die Foundation |
| 2. Reine Engine in CC, eigene UX | Technisch möglich; derzeit zusätzlicher Umzug einer bereits integrierten Core-State-Funktion |
| 3. Bestehender Core-State-Wake-Owner publiziert nach CC | **Empfohlen** |
| 4. Heutige parallele Struktur dauerhaft behalten | Nicht empfohlen; Parität und Cutover bleiben sonst dauerhaft offen |

### Konkrete Empfehlung

- **Ein automatischer Wake-Owner:** der bereits beschlossene interne Core-State-Owner.
- CC stellt seinen Plan als versionierten Contract bereit.
- Kalenderqualität, Planhorizont und Event-Semantik werden vor einer Abschaltung des alten Planners abgeglichen.
- Die UI konsumiert denselben Plan und dieselbe Konfiguration.
- Alte manuelle Skip-/Override-Funktionen werden nicht automatisch wieder als Zielumfang eingeführt. Sie sind in den neueren Entscheidungen ausdrücklich vom automatischen Zielmodell getrennt beziehungsweise ausgeschlossen.

Der Inventar-PR [Wake Planner #36](https://github.com/Levtos/ha_wake_planner/pull/36) ist **Draft und nicht gemergt**. Sein Inhalt ist wertvolle Inventur, aber kein Beleg für abgeschlossenen Cutover. Ältere darin beschriebene Reihenfolgen sind gegen die inzwischen weiterentwickelten [Core-State-Entscheidungen #36](https://github.com/Levtos/benni-core-state/issues/36) zu lesen.

Zusätzliche offene Mechanismen:

- Kalender-LKG besitzt im alten Cache keine allgemeine maximale Stale-Grenze.
- Event-Deduplizierung ist nur im RAM.
- Gleicher Plan nach Neustart bedeutet nicht automatisch „Wake-Event darf erneut ausgelöst werden“.
- Leerer Kalender und erfolgreich abgefragter Kalender ohne Termine sind unterschiedliche Zustände.

---

# J. UX- und Terminologie-Vorschlag

## Empfohlene Bezeichnung: „Fachliche Zustände“

**„Scenario Sensor“ empfehle ich nicht.**

Drei Gründe:

1. Nicht jedes Ergebnis braucht eine HA-Sensor-Entity.
2. „Scenario“ wird im Medienbereich bereits für andere, teilweise policybezogene Bedeutungen verwendet.
3. Ein Plan, eine unsichere Presence-Ableitung und ein Messwert sind keine identische Sensorart.

„Fachliche Wahrheiten“ kann als Architekturbegriff funktionieren. Für eine Oberfläche ist **„Fachliche Zustände“** verständlicher und behauptet bei degradierten Daten keine Gewissheit.

Technische Begriffe:

- **Contract:** stabile Schnittstelle und Bedeutung.
- **Binding:** Verbindung zur Quelle.
- **Resolver/Engine:** zuständige Berechnung.
- **Zustand:** aktuelles Ergebnis.
- **Plan:** zeitbezogenes Ergebnis.
- **Diagnose:** Herkunft, Qualität und Erklärung.

## Schlanke Informationsarchitektur

```text
Übersicht

Quellen & Bindings
  Filter: Climate / Blind / Light / gemeinsam genutzt

Fachliche Zustände
  Zustand auswählen
  → Bedeutung
  → Inputs und Herkunft
  → zuständige Berechnung
  → Ergebnis und Qualität
  → verwendende Consumer

Diagnose
  Abhängigkeiten
  Änderungen / Revisionen
  Fehler und veraltete Quellen

Einstellungen
  Profile / zulässige Parameter / Import-Export
```

Ein separater Hauptmenüpunkt für jeden Algorithmustyp würde die technische Implementierung unnötig in die Bedienung übertragen.

## Was editierbar sein sollte

| Nutzereingabe | Empfehlung |
|---|---|
| Physische Entity/Attribut | Ja, typgeprüfter Selektor |
| Zugelassene Einheit/Normalisierung | Möglichst automatisch aus Metadaten; Ausnahmen kontrolliert |
| Räume und vorhandene Fähigkeiten | Ja, innerhalb des jeweiligen Modells |
| Fachliche Parameter | Ja, mit Schema, Grenzen und verständlicher Bedeutung |
| Interne Contract-ID | Nicht im normalen UI editierbar |
| Beliebige Formel/Code | Nein |
| Freies Verdrahten beliebiger Engines | Nein |
| Technische Schema-/ID-Migration | Versionierter Entwicklungsauftrag |

Ein Quellenwechsel muss die betroffenen Consumer zeigen. Modulansichten bleiben Filter auf **dasselbe Binding**.

Hilfetexte sollten aus derselben versionierten fachlichen Definition stammen, die auch den Resolver erklärt. Das Frontend darf keine zweite Formeldokumentation und keine neue Businesslogik pflegen. Info-Hilfe muss per Touch, Klick und Tastatur erreichbar sein.

Das entspricht der bestehenden Richtung aus [ADR 0001](https://github.com/Levtos/control/blob/28bf515449187c067fe5e23e9c383c0aa0523236/docs/adr/0001-ux-frontend-standard.md) und [Control #17](https://github.com/Levtos/control/issues/17).

---

# K. Fehlende Szenarien und Blind Spots

Dieser Abschnitt enthält die Mechanismen, die bei einer einfachen Erweiterung „Fusion plus Resolver“ übersehen würden.

| Blind Spot | Reales Beispiel | Benötigter Mechanismus |
|---|---|---|
| **Messzeit ≠ Lesezeit** | Climate liest denselben numerischen Wert erneut | Beobachtungszeit und Quellengesundheit getrennt von Berechnungszeit |
| **Unveränderter Wert ≠ ausgefallener Sensor** | Temperatur/Fenster bleibt lange gleich | Providerabhängiger Lebensnachweis; kein pauschaler Änderungs-TTL |
| **Forecast ist kein aktueller Messwert** | Climate bewertet kommende Stunden/Tage | Gültigkeitsintervall, Prognosezeitpunkt und Horizont |
| **Zeitfenster brauchen eine genaue Definition** | `bathroom_humidity_rise_5m` vergleicht tatsächlich den vorherigen Messwert innerhalb einer Altersgrenze | Explizite Fenster-/Sampling-Semantik |
| **Historischer Wert ist nicht automatisch Aggregat** | „Gestern“ im Climate-Bodenmodell | Definition letzter Wert versus Tagesmittel |
| **Kalenderkonfiguration beweist keine Datenqualität** | Core-State-Wake | Beschaffungsstatus und Coverage |
| **Leere Daten haben mehrere Ursachen** | Keine Termine versus Abfragefehler | Fehlend/leer/stale getrennt |
| **Plan und Ereignis sind verschieden** | `next_wake` versus Wake-Trigger | Event-ID, Episode, Wiederholungsregel |
| **Restart darf keine neue Flanke erfinden** | Bio-/Wake-/Media-Zustand beim Boot | Persistentes Edge-Bookkeeping |
| **TTL braucht einen Zeitbezug** | Manueller Private-Zustand nach Restore | Absoluter Ablauf oder ausdrücklich erneuerbare Laufzeit |
| **Nutzerabsicht ist keine Gerätebeobachtung** | Bewusst TV aus versus Streaming erkannt | Explizite Intent-/Command-Provenienz |
| **Apply-Evidence kann legitim zurückfließen** | TV-Aus bestätigt eine Schlafepisode | Korrelations-ID und gerichteter Ereignisvertrag |
| **Safety-Ersatzwert ist nicht physische Beobachtung** | Leere Fensterbatterie | Wirksamer Zustand plus erhaltene Ursache/Evidenz |
| **Scores können Scheingenauigkeit erzeugen** | Confidence bei Umwelt-/Quellenauswertung | Definierte Skala und Bedeutung |
| **Mehrere Quellen können dieselbe Evidenz spiegeln** | Provider und Master desselben Geräts | Lineage statt vermeintlich unabhängiger Bestätigung |
| **Raum-/Personen-Scope ist feiner als Profil** | Zwei Fenster, mehrere Bewohner | Instanzschlüssel und Topologie |
| **„Profil“ hat mehrere Bedeutungen** | `benni/eltern`, Wochentag/Wochenende, Heizprofil | Getrennte Typen und Namen |
| **Optionale Fähigkeiten dürfen nicht zum Installationszwang werden** | Eltern ohne Media oder Lux | Required/optional plus definiertes Degradieren |
| **Registry-Rollback ist kein Engine-Rollback** | Alte Konfiguration mit neuer Bio-State-Struktur | Kompatibilitätsprüfung und State-Migration |
| **Ein Producer kann schweigen, obwohl CC läuft** | Externe Media-/Core-Engine fällt aus | Producer-Readiness und Stale-Erkennung |
| **Callbacks können Rückstau erzeugen** | Viele Consumer/Events | Begrenzte Queues, Coalescing, Fehlerisolation |
| **Gemeinsame Werte benötigen konsistente Versionen** | Kontext, Gerät und Foreground aus verschiedenen Zwischenständen | Zusammengehörige Ergebnisrevision |
| **Aktorrolle ist nicht Schaltberechtigung** | Zentral gebundener TV | Fähigkeit/Identität zentral; finaler Writer bei Apply |
| **UI benötigt Capabilities, nicht nur Metadaten** | Manuelle Apple-Music-Wiedergabe | Transportfähigkeiten und aktueller Playerzustand als View-Model-Vertrag |

## Weitere Flottenmechanismen

- **Blind Control:** Geometrie, Solarberechnung und Strahlungsdaten passen zu reinen Resolvern; Heat-/Glare-Hysterese und Zielpositionen bleiben policybezogen.
- **Bathroom:** Taupunkt ist eine Ableitung. Lüftungsbudget, Cooldown, Trocknungsheizung und Diffuserbegrenzung sind Policy-/Steuerungszustände.
- **Light:** Bewegungs-, Ring- und Nachlaufzustände rechtfertigen keine Verlagerung aller Timer in CC. Viele existieren ausschließlich zur Lichtsteuerung.
- **Plug Policy:** Idle-Zeit, manuelle Einschaltung, Schutz und Cut-Sicherheit sind Policy-/Apply-Zustände.
- **Notification Router:** DND, Deduplizierung und Rate Limits sind Routing-/Nutzerzustände, keine universellen Haushaltsfakten.
- **Title Classifier:** Die gemeinsame Effective-Berechnung ist bereits ein gutes Muster. Katalog, Providerintegration und Klassifikationsverwaltung müssen nicht in CC wandern.
- **Door Policy:** Ein Schlosszustand ist keine vollständige Tür-Opening-Evidence. `locked` beweist nicht allein den Zustand eines unabhängigen Türkontakts.

Diese Fälle lassen sich mit den vorgeschlagenen Grundmodellen abdecken. Sie rechtfertigen **keine eigene Runtime-Art für jede einzelne Formel oder jeden Timer**.

---

# L. Risiken und Architekturregeln

## Hauptrisiken

| Risiko | Besonders bei | Erforderliche Begrenzung |
|---|---|---|
| Monolith | Variante B | Fachmodule, explizite APIs, keine globalen Reads |
| Beschädigter oder falsch interpretierter State | Engine-Verlagerung | Versionierte State-Schemata, Migration und Rollback-Vertrag |
| Falsche Neustartübergänge | Bio, Wake, Media | Episode-/Event-Identität und Restore-Tests |
| Consumer sehen Zwischenzustände | Mehrstufige Auswertung | Konsistente Snapshots/Ergebnisrevisionen |
| Contract-Explosion | Veröffentlichung jedes Helpers | Nur tatsächlich gemeinsam benötigte fachliche Outputs |
| Schema-Kopplung | Gemeinsame Releases | Kompatibilitätsregeln pro Contract |
| Abhängigkeitsschleifen | Cross-Domain-Resolver | Vollständiger deklarierter Graph |
| Performanceprobleme | Zentrale periodische Auswertung | Messung, bedarfsorientierte Auswertung, begrenzte Arbeit |
| Schlechte Debugbarkeit | Generische konfigurierbare Engines | Code-definierte Resolver mit gezielter Explainability |
| Unerwartete Consumer-Auswirkung | Geteiltes Binding | Verwendungsnachweis und Änderungsübersicht |
| Sicherheitsverlust beim Cutover | Opening/Heizung/TV | Ein Writer, explizite Gates, Vergleich und Live-Abnahme |
| Scheinbare Profilfähigkeit | Nur andere Entity-IDs | Räume, Personen, Fähigkeiten und Persistenz vollständig scopen |

**Mehrere HA-Integrationen sind keine getrennten Betriebssystemprozesse.** Eine gemeinsame Runtime reduziert deshalb nicht automatisch vorhandene Prozessisolation; umgekehrt kann sie den logischen Ausfallbereich und die Release-Kopplung deutlich vergrößern. Ein Performancegewinn ist ohne Messung nicht belegt.

## Profilbewertung

| Komponente | Bewertung | Konkrete Grenze |
|---|---|---|
| Core Contracts | **MOSTLY READY** | Gute Profiltrennung; allgemeine Attribut-/Adapterfähigkeit und fachliche Lücken bleiben |
| Core State | **PARTIAL** | Eltern-Presence vorhanden, aber profilabhängige Logik und Quellen-/Topologieannahmen |
| Media State | **PARTIAL** | Feste Gerätewelt und spezialisierte Arbitration; optionales Modul möglich |
| Climate Policy | **PARTIAL** | Feste Raum-/Gerätestruktur; Eltern-YAML noch separater Implementierungspfad |
| Blind Control | **PARTIAL** | Gute Contract-Consumer-Grenze in ausgewähltem Pfad; Migration und Konfiguration nicht vollständig vereinheitlicht |
| Gesamter Stack | **Noch nicht bereit** | Bindings, Fähigkeiten, Topologie und Cutover fehlen in mehreren Consumern |

**Eltern benötigt keine Media-Funktion als Voraussetzung für Climate.** Benötigte Presence-/Bio-/Zeitinformationen müssen über die tatsächlich erforderlichen Contracts verfügbar sein. Fehlende Media-Fähigkeit ist ein erlaubter Profilzustand, kein Fehler.

## Entscheidungsregel: State oder Policy?

Eine belastbare Prüfung besteht aus drei Fragen:

1. **Was behauptet das Ergebnis?**
   Beobachtung/Ableitung über die Welt – oder gewünschtes Verhalten?

2. **Welche Parameter beeinflussen es?**
   Quellenkalibrierung und Messmodell – oder Komfortziel, Priorität, Energiepräferenz?

3. **Würde die Aussage bei unveränderter Welt allein durch eine andere Handlungspräferenz wechseln?**
   Dann ist sie häufig Policy oder ein policybezogenes Bewertungsmerkmal.

Beispiele:

| Aussage | Einordnung |
|---|---|
| Raumtemperatur 18 °C | Fakt |
| Taupunkt 12 °C | Physikalische Ableitung |
| Raum vermutlich belegt | Zustandsableitung mit Unsicherheit |
| Fenster wirksam offen wegen Safety-Fallback | Zentraler Sicherheitsvertrag |
| Raum benötigt Heizung | Policy, sofern Komfortziel/Heizstrategie enthalten |
| Heizung soll auf 21 °C | Policy |
| Nächster Wake gemäß geltendem Plan 07:30 | Plan |
| Jetzt Musik als Weckaktion starten | Policy/Apply |
| HA-Service ausführen | Bei Gerätewirkung Apply; eigene Konfiguration oder lesende Datenbeschaffung gesondert einordnen |

**„Führt keine Aktion aus“ reicht als Grenze nicht aus.** Eine Funktion kann bereits eine Policy entscheiden, obwohl sie nur einen Boolean zurückgibt.

---

# M. Das stärkste Gegenargument gegen die Konsolidierung

Die stärkste technisch begründete Gegenposition lautet:

> Core State und Media State besitzen bereits fachlich spezialisierte, teilweise rein testbare Engines. Ihr Verschieben in Core Contracts beseitigt weder falsche Bedeutungen noch fehlende Evidenz. Es erzeugt zunächst eine gemeinsame Release-, Persistenz- und Startup-Verantwortung für besonders empfindliche Zustände. Gleichzeitig muss die heute begrenzte Foundation um eine umfassende Engine-Infrastruktur erweitert werden.

Dieses Gegenargument trifft den aktuellen Stand.

Besonders deutlich wird es bei Wake: Die reine Berechnung ist bereits intern verfügbar. Die offene Kalender- und Event-Parität würde nach einem Umzug unverändert weiterbestehen.

Und bei Media: Ein fälschlich aus Geräteaktivität abgeleiteter Einschaltauftrag bleibt falsch, auch wenn State und Registry dieselbe Integration sind.

**Warum ich trotzdem die Zentralisierung des Datenzugangs empfehle:**

Dein eigentliches Ziel wird dadurch sehr gut erfüllt:

- Eine Rolle kann zentral neu gebunden werden.
- Consumer müssen den physischen Anbieter nicht kennen.
- Jeder Zustand hat einen benannten Owner.
- Alle Consumer sehen dieselbe Bedeutung und Qualität.
- Diagnose und Auswirkungen eines Quellenwechsels werden nachvollziehbar.

Dafür müssen nicht sofort alle Berechnungen im selben Integrationspaket laufen.

**Die sinnvolle Konsolidierung betrifft zuerst Verträge und Zugriffspfade. Der gemeinsame Runtime-Host ist eine nachgelagerte, gesondert zu rechtfertigende Entscheidung.**

---

# N. Entscheidungsvorschlag und mögliche Folgeaufträge

## Mein konkreter Entscheidungsvorschlag

**Variante C als Zielrichtung beschließen, ergänzt um kleine interne, fachlich isolierte Resolver.**

Das bedeutet:

1. Core Contracts wird der verbindliche Datenzugang für gemeinsam genutzte fachliche Zustände.
2. Jeder Contract erhält genau einen fachlichen Berechnungsowner.
3. Core State und Media State bleiben zunächst diese Owner.
4. Ihre Ergebnisse gelangen künftig typisiert nach CC.
5. Wiederverwendbare kleine Ableitungen dürfen innerhalb von CC laufen, wenn Inputs, Bedeutung und Consumer klar sind.
6. Policies und Apply bleiben außerhalb der State-Berechnung.
7. Eine spätere Zusammenlegung kompletter Engines bleibt möglich, ist aber kein vorausgesetzter Teil jeder Contract-Migration.

## Vorgeschlagene Reihenfolge separater Arbeitspakete

**Dies sind Vorschläge; keine Issues wurden erstellt oder verändert. Bestehende Issues müssen vor einem neuen Ticket auf Überschneidung geprüft werden.**

| Reihenfolge | Paket / Repository-Rahmen | Problem und Scope | Nicht enthalten / Abhängigkeiten |
|---|---|---|---|
| Parallel dringend | Bestehende Media-Fehler: `benni_media_apply`, `benni_media_state` | TV-Off-Intent, R12/WOL-Re-Arm, frühe Foreground-Pause; bestehende #46/#47/#24 verwenden | Keine vollständige CC-Migration erforderlich |
| 1 | Architekturentscheidung in `control` | Zentraler Zugriff versus Runtime-Host; Ownership, Policy-Grenze, Producer-Richtung; #37 ausdrücklich bewerten | Kein Code, keine Auflösung bestehender Owner |
| 2 | Opening-Sicherheitsvertrag in `benni-core-contracts` | Bestätigtes Batterie→wirksam offen samt Quality/Ursache; vorhandene Opening-Verträge abgleichen | Keine neue Sicherheitsentscheidung erfinden |
| 3 | Quellenadapter in `benni-core-contracts` | Entity-State/Attribut, Einheiten, Beobachtungszeit, zulässige Normalisierung | Keine universelle HA-Geräteregistry |
| 4 | Climate-Durchstich | Inputs und erzeugte Werte klassifizieren; bestehende Contracts verwenden; neutrale Ableitungen von Heizbewertung trennen | Kein neuer Wochenplan, keine neue Komfortstrategie |
| 5 | Typisierte Producer-Anbindung in `benni-core-contracts` | Ownerregistrierung, Scope, Version, Qualität, Lifecycle, Abhängigkeiten | Noch keine vollständige Core-/Media-Verlagerung |
| 6 | Core-State-Contract-Ausgänge | Presence/Bio/Day/Wake als eindeutige Outputs; Consumer schrittweise umstellen | Bio-/Presence-Produktregeln nicht beiläufig ändern |
| 7 | Eltern-Climate mit derselben Engine | Räume, Fähigkeiten, Parameter und Bindings; Ablösung separater Heiz-YAML mit einem Apply-Owner | Kein Media-Zwang; Live-Gates separat |
| 8 | Media-Contract-Bereinigung | Foreground, Playback, Kontext, Eligibility und Intent trennen; Rückkopplungen auflösen | Kein reiner Repository-Umzug als Ersatz |
| 9 | Wake-Parität und Cutover | Kalenderquellen, Quality, Events, Vorschau; bestehende #34/#35 und Draft #36 einordnen | Keine automatische Wiederaufnahme verworfener manueller Features |
| 10 | Registry-UX | Gemeinsame Bindings, verständliche Rollen, Auswirkungen und Hilfen | Kein visueller Universal-Automation-Editor |
| 11 | Core-Devices-/Legacy-Rückbau | Für jede Funktion Nachfolger, Consumer und Live-Gate nachweisen | Keine Abschaltung allein wegen vorhandener CC-Schemata |
| Später, optional | Runtime-Konsolidierung | Prüfen, ob bestimmte Engines innerhalb CC tatsächlich einfacher und stabiler werden | Nur nach belegtem Nutzen und eigenem Migrationsauftrag |

Die konkrete Implementierungs-Ownership wird später **pro Issue** festgelegt.

## Was ausdrücklich erhalten bleiben sollte

- Die bestehende Core-Contracts-Registry und Consumer API.
- Profilbezogene Revisionen und Konfigurations-LKG.
- Die strenge Contract-Consumer-Grenze des bereits umgestellten Blind-Pfads.
- Die gemeinsame Effective-Klassifikation im Title Classifier.
- Restartfeste Bio-Episoden- und Flankenbuchhaltung.
- Die Trennung von Media Policy und Media Apply.
- Gerätebezogene Recovery-/Settling-Logik bei Apply.
- Die bereits intern vorhandene reine Wake-Planberechnung.
- Bestehende Climate-Produktlogik, soweit kein belegter Fehler beziehungsweise freigegebener Vertragskonflikt vorliegt.

## Welche Entscheidung von dir tatsächlich noch gebraucht wird

Die offene Grundentscheidung ist **nicht** erneut die Fenster-Sicherheit, der Wochenplan oder eine Media-Pflicht für Eltern.

Sie lautet:

> Soll Core Contracts zunächst der verbindliche gemeinsame Datenzugang mit externen fachlichen Ownern werden – oder soll ausdrücklich zusätzlich die vollständige gemeinsame State-Runtime als verbindliches Migrationsziel beschlossen werden?

**Meine begründete Empfehlung ist die erste Variante mit der Möglichkeit späterer gezielter Runtime-Konsolidierung.**

Damit lässt sich die zentrale Registry sinnvoll nutzen, Climate zuerst migrieren und das Elternprofil vorbereiten, ohne vorher sämtliche empfindlichen Zustandsmaschinen umzuziehen.

**Audit abgeschlossen. Keine Änderungen vorgenommen. Weitere Arbeit erst nach deinem Review.**
