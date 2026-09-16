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

Er ist eine Dokumentationsablage, keine neue fachliche Analyse. Die vier
enthaltenen Artefakte wurden unverändert aus ihrer jeweiligen Quelle
übernommen.

## Dokumentstatus

### `MEDIA_SYSTEM_AUDIT_OPUS_REVIEW.md`

- **Datum:** siehe Dokumentkopf
- **Rolle:** `VERIFIED MEDIA REVIEW / CURRENT REFERENCE FOR MEDIA FINDINGS`
- **Versionsstatus:** verifiziert gegen den zum Erstellungszeitpunkt
  aktuellen `origin/main`
- **Verwendung:** Der Review verifiziert den ursprünglichen (Sonnet-)
  Media-Audit gegen den damals aktuellen `origin/main` und unterscheidet
  aktuelle Befunde, historische Snapshot-Befunde und fehlerhafte
  Audit-Aussagen. Für Media-Semantik ist dieser Review gegenüber dem
  ursprünglichen Audit vorzuziehen.

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
  relevante Aussagen gegen aktuellen `origin/main` geprüft werden.

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

## Keine „latest“-Kopie

Es gibt bewusst keine automatisch überschreibbare Datei wie `latest.md`
oder `current_audit.md`. Die Originalartefakte bleiben datiert bzw.
identifizierbar; diese `README.md` ist der Index, der ihre aktuelle Rolle
beschreibt. Das verhindert, dass historische Evidenz durch spätere
Versionen still überschrieben wird.
