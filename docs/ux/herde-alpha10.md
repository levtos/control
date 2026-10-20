> A2-Freigabe durch Benni am 20.10.2026 einschließlich K1–K8. Originale jetzt unter assets/herde/ übernommen; Merge/Release/Live bleiben gesperrt. Die folgende A1-Planung bleibt als Provenienz.

# Einhorn-Herde — Alpha-10-Design und Übernahmeplan

20.10.2026 · [control#60](https://github.com/Levtos/control/issues/60)
und [wuschel#24](https://github.com/Levtos/wuschel/issues/24).
Agent `agent:codex`; **A1 zur Review, kein Assetbau und keine Veröffentlichung**.
Dieser ergänzende Draft-PR bleibt bis zur jeweiligen Benni-Freigabe offen;
die Herdenübernahme in A2 wird in demselben PR fortgeführt.

## Aktuelles Soll und Ist

Alle neun Figuren und neun PNG-Icons liegen in Bennis bereitgestelltem ZIP.
Geprüft: Figuren 1024×1536 RGBA, Icons 1254×1254 RGBA.
Die Icon-Ecken sind bereits bereinigt: unsichtbare Pixel tragen Kachel-Petrol.
Keine SVGs, keine neue Bildgenerierung oder Nachbearbeitung der Originale.
Der mitgelieferte Steckbrief ist Version 4 und berücksichtigt Schlummer als
jungen Erwachsenen (ca. 20). Die ältere Teenager-Angabe aus dem Issue ist
durch den Kommentar vom 20.10.2026 überholt.

Nach A2-Freigabe werden unter `assets/herde/` alle 18 Original-PNGs und
`herde-steckbrief.md` übernommen. PNGs bytegleich zum ZIP; SHA256-Manifest
und README verweisen auf #60 und die Herkunft. Dateinamen mit ausgeschriebenen
Umlauten; keine Anpassung der Bilder für ein vermeintliches einheitliches
Originalformat.

Einzige geplante Steckbrief-Textkorrektur:

> Icon als PNG 1254×1254, Petrol-Kachel, abgerundete transparente Ecken,
> unsichtbare Pixel im Kachel-Petrol

Sie ersetzt die Checklisten-Zeile „SVG-Icon zeichnen“.

## Namens- und Verwendungsvorlage

| Name | Modul | Symbol |
|---|---|---|
| Wuschel | App / Core Contracts | Vertragsrolle mit Regenbogensiegel |
| Traber | HA-Integration; Funktion in der UI „HA-Anbindung“ | gestapelte Briefe mit bunten Siegeln |
| Schnüffel | Anwesenheit, später | Lupe mit Haus |
| Kuschel | Klima, später | Thermostat-Drehknopf |
| Funkel | Licht, später | Glühbirne |
| Blinzel | Rollos, später | Sonnenbrille |
| Schlummer | Schlaf/Wecken, später | Wecker |
| Träller | Medien, später | Popcorn-Eimer |
| Pförtchen | Tür, später | Schlüssel |

Weitere Module sind hier ausschließlich Vorlage, keine Implementierung,
Repoanlage, Umbenennung oder Freigabe ihres späteren Fachscopes.
Mähnenfarben sind Erkennungsfarben, getrennt von Grün/Gelb/Rot als Status.
Icons dienen Kachel, Store, Favicon und eigener Navigation; Figuren dienen
Seitenkopf, Leerständen, Einrichtung und Fehlerseiten. Listen bleiben symbolisch.
HA-Seitenleiste nutzt `mdi:script-text-outline`, kein eigenes PNG.
Die konkret beauftragte Alpha-9-Oberfläche bleibt Grundlage, kein neues
Framework oder allgemeiner Designsystem-Cutover.

## Ableitungen und Branding

Wuschel enthält in A2 ein wiederholbares Skript mit gepinnten Werkzeugversionen,
Quellhashes und Zuschnitt-/Hufankermanifest. Beim Skalieren linear RGB und
vormultipliziertes Alpha; Originale bleiben unverändert.
Gemeinsame Randregel und Hufgrundlinie, keine Verzerrung oder abgeschnittenen
Bildteile; WebP 512/1024 px Breite. Originale nicht im Frontend-Bundle.
Wuschel: App-Icon 128, Logo, Favicon 32 plus 16-px-Siegelausschnitt,
Apple Touch 180; eigene Navigation nur Icon.

Traber bringt sein Icon/Logo in `custom_components/traber/brand/` mit.
Seit HA 2026.3 unterstützt Home Assistant lokale Brandbilder für
Custom-Integrationen; sie haben Vorrang vor den zentralen Brands.
Kein brands-PR für diesen unterstützten Weg nötig.
Quellen: [HA-Ankündigung](https://developers.home-assistant.io/blog/2026/02/24/brands-proxy-api/),
[aktuelle Integrations-Brand-Doku](https://contributors.home-assistant.io/core/integrations/brands/).
Tatsächlich installierte HA-Version erst vor Teil B über sicheren Status
ohne App-Optionen verifizieren; kein automatisches Upgrade.

Detailentscheidungen und Prüfplan:
[Wuschel ADR 0024](https://github.com/Levtos/wuschel/blob/agent/codex-alpha10-wuschel/docs/adr/0024-herd-assets.md),
[A1-Einstieg](https://github.com/Levtos/wuschel/blob/agent/codex-alpha10-wuschel/docs/alpha10/README.md).
Diese Branchlinks werden beim späteren Merge auf den finalen Commit aktualisiert.

## Nachweise und Gates

A1: nur dieser Designplan, Dokumentations-/Linkprüfung und Draft-PR.
A2: Originalhashvergleich aller 18 PNGs, korrekte Steckbriefzeile, Manifest,
README und Wuschel-Ableitungen; reproduzierbarer zweiter Lauf und Icon-
Kontaktbogen auf hell/dunkel. Dann gemeinsamer Review-STOPP.
Kein Merge, Release, HACS-/HA-Schritt aus A1. Technische Abnahme ist nicht Live.
