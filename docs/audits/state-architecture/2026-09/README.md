# State-/Architecture-Audit-Archiv 2026-09

## Zweck

Dieser Ordner enthält repoübergreifende Architektur- und State-Audits als
gemeinsame Referenz für:

- Architekturarbeit
- Core Contracts
- Core State
- Media State
- Consumer-Migrationen
- Cross-Domain-State-Analysen

Er ist eine Dokumentationsablage, keine neue fachliche Analyse. Die enthaltenen
Artefakte wurden unverändert aus ihrer jeweiligen Quelle übernommen.

## Aktualitätshierarchie

Die Ablage enthält Artefakte mit unterschiedlichem Verbindlichkeitsgrad. Bei
Widersprüchen gilt diese Reihenfolge:

1. **Aktueller Remote-Default-Code** (`origin/main` bzw. der jeweilige
   Default-Branch) — der technische IST. Kein Dokument ersetzt ihn.
2. **`STATE_CONSUMER_*` (2026-09-16)** — verifizierte Cross-Domain-Inventur.
   Gegen den Remote-Default-Code neu geprüft; beschreibt, wer welchen Zustand
   wie konsumiert.
3. **`STATE_DECISION_REVIEW.*` (2026-09-16)** — fachliche
   Entscheidungsaufbereitung auf Basis von (2). **Empfehlungen, kein Soll-Vertrag.**
4. **`MEDIA_SYSTEM_AUDIT_OPUS_*`** — Media-Deep-Dive. Für Media-Semantik
   weiterhin die detaillierteste Quelle.
5. **Audits vom 08.09.2026** — historische Seeds. Nicht automatisch als
   aktueller IST verwenden.

> **Wichtig:** `STATE_DECISION_REVIEW` ist **nicht** automatisch Soll-Vertrag.
> Jede dort ausgesprochene Empfehlung steht auf `benni_decision = OPEN` und wird
> erst durch eine ausdrückliche Entscheidung von Benni verbindlich. Eine
> Zielarchitektur oder ein Enum-Wechsel gilt nicht deshalb als beschlossen, weil
> er in diesem Ordner dokumentiert ist.

## Dokumentstatus

### `STATE_CONSUMER_AUDIT.md`

- **Datum:** 2026-09-16
- **Rolle:** `CURRENT CROSS-DOMAIN STATE CONSUMER AUDIT`
- **Versionsstatus:** gegen den Remote-Default-Branch der 20 beteiligten
  Repositories neu verifiziert (`git archive origin/main`, keine lokalen
  Checkouts verändert); Repo-Stände und Commits sind im Dokumentkopf tabelliert
- **Verwendung:** Aktueller Überblick über die von Core State und Media State
  veröffentlichten fachlichen Zustände und deren Consumer. Enthält die
  Preflight-Tabelle, die Consumer-Drifts, die Duplicate-Truth-Klassifikation,
  den Rename-Impact und den Abgleich gegen die älteren Audits
  (`CONFIRMED_CURRENT` / `OUTDATED` / `NEW_FINDING`). Für Fragen nach dem
  aktuellen Cross-Domain-IST ist dies das Einstiegsdokument.

### `STATE_CONSUMER_EDGES.csv`

- **Datum:** 2026-09-16
- **Rolle:** `CANONICAL CONSUMER EDGE DATASET`
- **Versionsstatus:** an `STATE_CONSUMER_AUDIT.md` gekoppelt
- **Verwendung:** Normalisierte technische Grundlage. Eine Zeile je fachlich
  relevanter Consumer-Kante:
  Producer → State/Value → Consumer → Decision Level → Interpretation → Effect,
  jeweils mit Fallback- und Unknown-Verhalten, Quellenreferenz, Testabdeckung,
  Legacy-Einordnung und Change-Impact. Die maßgebliche Detailquelle, auf die
  alle übrigen Artefakte über Edge-IDs (`E-xxx`) verweisen.

### `STATE_CONSUMER_MATRIX.csv`

- **Datum:** 2026-09-16
- **Rolle:** `HUMAN CROSS-DOMAIN CONSUMER MATRIX`
- **Versionsstatus:** an `STATE_CONSUMER_AUDIT.md` gekoppelt
- **Verwendung:** Breite Arbeitsansicht pro State bzw. konkretem Wert und
  Integration. Beantwortet auf einen Blick, wie jede Domäne einen einzelnen
  Enum-Wert behandelt, inklusive kanonischem Owner, Semantic Conflict,
  Rename-Impact und Architekturnotiz. Für Architekturgespräche gedacht, nicht
  für die maschinelle Auswertung.

### `STATE_CATALOG.csv`

- **Datum:** 2026-09-16
- **Rolle:** `CURRENT STATE INVENTORY`
- **Versionsstatus:** an `STATE_CONSUMER_AUDIT.md` gekoppelt
- **Verwendung:** Inventar der publizierten fachlichen Zustände mit Owner,
  Wertebereich, Semantic Scope, Transport, Statefulness, Persistenz,
  Quality-/Freshness-Verfügbarkeit, Consumer-Anzahl, Cross-Domain-Nutzung,
  Duplicate-Truth-Klasse und Contract-Eignung.

### `DECISION_QUESTIONS.csv`

- **Datum:** 2026-09-16
- **Rolle:** `RAW DECISION QUESTION INVENTORY`
- **Versionsstatus:** direktes Ergebnis des Consumer-Audits; **noch nicht**
  fachlich verdichtet
- **Verwendung:** Die offenen Fragen in der Form, in der sie unmittelbar aus der
  Kantenanalyse entstanden sind. Für die entscheidbare Ebene ist
  `STATE_DECISION_REVIEW` vorzuziehen; diese Datei bleibt als Rohstand und
  Nachvollziehbarkeitsbeleg erhalten.

### `STATE_DECISION_REVIEW.md`

- **Datum:** 2026-09-16
- **Rolle:** `CURRENT HUMAN DECISION BASELINE`
- **Versionsstatus:** Verdichtung von `STATE_CONSUMER_EDGES.csv` und
  `DECISION_QUESTIONS.csv`; kein eigener Audit
- **Verwendung:** Fachliche Entscheidungsebene für Benni. Gegenstand ist die
  einzelne **Decision**, nicht der Sensor oder Enum-Wert — die Sleep-Frage ist
  deshalb nicht als „ist `provisional_sleep` gleich `sleep`?“ gefasst, sondern
  pro betroffener Entscheidung getrennt (Auto-Start, Resume, HomePod-Pause,
  Denon-Nachlauf, Sleep-TV, Notifications, Climate, Light, Blind, Wake,
  Readiness). Enthält Haupttabelle, Detailblöcke je Decision, Decision
  Dependency Graph, Priorisierung (P1/P2/P3) und die Fusion-Evidence-Bewertung.
  **Empfehlungen sind Vorschläge, bis Benni sie bestätigt.**

### `STATE_DECISION_REVIEW.csv`

- **Datum:** 2026-09-16
- **Rolle:** `STRUCTURED DECISION DATASET`
- **Versionsstatus:** an `STATE_DECISION_REVIEW.md` gekoppelt
- **Verwendung:** Maschinenlesbare Decision-Liste inklusive Empfehlung,
  Klassifikation, Migrationstyp, Change Type, Compatibility Requirement,
  Contract Readiness, Abhängigkeiten, Priorität und Entscheidungsstatus. Die
  Spalte `benni_decision` steht in allen Zeilen auf `OPEN`.

### `MEDIA_SYSTEM_AUDIT_OPUS_REVIEW.md`

- **Datum:** siehe Dokumentkopf
- **Rolle:** `VERIFIED MEDIA REVIEW / CURRENT REFERENCE FOR MEDIA FINDINGS`
- **Versionsstatus:** verifiziert gegen den zum Erstellungszeitpunkt
  aktuellen `origin/main`
- **Verwendung:** Der Review verifiziert den ursprünglichen (Sonnet-)
  Media-Audit gegen den damals aktuellen `origin/main` und unterscheidet
  aktuelle Befunde, historische Snapshot-Befunde und fehlerhafte
  Audit-Aussagen. Für Media-Semantik ist dieser Review gegenüber dem
  ursprünglichen Audit vorzuziehen. Der Cross-Domain-Audit vom 16.09.2026 hat
  seine für den Stack relevanten Befunde erneut geprüft und einzeln als
  `CONFIRMED_CURRENT` oder `OUTDATED` eingeordnet.

### `MEDIA_SYSTEM_AUDIT_OPUS_FINDINGS.csv`

- **Datum:** siehe zugehöriger Review
- **Rolle:** `STRUCTURED FINDING INDEX`
- **Versionsstatus:** an den Opus-Review gekoppelt
- **Verwendung:** Maschinenlesbare Detailbefunde zum Opus-Review.

### `STACK_READONLY_AUDIT_2026-09-08.md`

- **Datum:** 2026-09-08
- **Rolle:** `HISTORICAL CROSS-DOMAIN SEED`
- **Versionsstatus:** historischer Snapshot des Levtos-/Home-Assistant-Stacks
- **Verwendung:** Repoübergreifender Read-only-Audit vom 08.09.2026. Nicht
  automatisch als aktueller IST verwenden. Für neue Arbeiten müssen
  relevante Aussagen gegen aktuellen `origin/main` geprüft werden. Für
  Cross-Domain-Fragen ist `STATE_CONSUMER_AUDIT.md` der aktuelle Nachfolger.

### `CORE_CONTRACTS_STATE_ARCHITECTURE_AUDIT_2026-09-08.md`

- **Datum:** 2026-09-08
- **Rolle:** `HISTORICAL ARCHITECTURE / CONTRACT SEED`
- **Versionsstatus:** historischer Architekturvorschlag
- **Verwendung:** Untersucht Core Contracts, Core State, Media State und
  mögliche Zielarchitekturen zum Stand 08.09.2026. Enthält Empfehlungen und
  damalige Architekturannahmen. Nicht als automatisch beschlossene
  Zielarchitektur behandeln.

## Source-of-Truth-Regel

Audit-Dokumente sind Referenz- und Entscheidungsartefakte, aber kein Ersatz
für aktuellen Quellcode. Bei Aussagen über den aktuellen IST gilt nach
Repository-Preflight der Remote-Default-Branch (`origin/main` bzw.
entsprechender Default-Branch des jeweiligen Repositories) als technische
Primärquelle. Historische Audit-Aussagen müssen bei zeitabhängigen Fragen
erneut gegen aktuellen Code geprüft werden.

Das gilt ausdrücklich auch für den Audit vom 16.09.2026: Seine Bindungsaussagen
stammen aus den Prefills im Code, nicht aus der laufenden Anlage. Gespeicherte
`entry.data`/`options` können abweichen; die betroffenen Stellen sind im
Dokument benannt.

## Keine „latest“-Kopie

Es gibt bewusst keine automatisch überschreibbare Datei wie `latest.md`
oder `current_audit.md`. Die Originalartefakte bleiben datiert bzw.
identifizierbar; diese `README.md` ist der Index, der ihre aktuelle Rolle
beschreibt. Das verhindert, dass historische Evidenz durch spätere
Versionen still überschrieben wird.
