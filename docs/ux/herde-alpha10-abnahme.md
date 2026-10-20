# Einhorn-Herde — A2-Nachweis

20.10.2026 · [Issue #60](https://github.com/levtos/control/issues/60),
[Control-PR #61](https://github.com/levtos/control/pull/61),
[Wuschel-PR #27](https://github.com/levtos/wuschel/pull/27).
Bennis A1-Review hat A2 einschließlich K1–K8 ausdrücklich freigegeben.

Unter `assets/herde/` liegen alle neun Originalfiguren (1024×1536), alle neun
Originalicons (1254×1254), der Steckbrief und das Quellmanifest. Alle 18 PNGs
wurden erneut bytegleich gegen Bennis freigegebenes ZIP geprüft. Die Icons
einschließlich der unsichtbaren Petrol-Pixel wurden nicht verändert.
Am Steckbrief ist genau eine Zeile geändert:

> Icon als PNG 1254×1254, Petrol-Kachel, abgerundete transparente Ecken, unsichtbare Pixel im Kachel-Petrol

Sie ersetzt die Checklisten-Zeile zum Zeichnen eines SVG-Icons. Weitere Module
bleiben Namens-/Motivvorlagen. Keine Installation oder Fachimplementierung
dieser Module gehört zur Herdenübernahme.

Wuschels Pipeline bindet die Originale an Control-Commit
`70b6e83b26a930632c7b2430fb6d5b955ec63158` und prüft deren SHA-256-Werte.
Zwei vollständige Ableitungsläufe liefern identische Hashes. Ein eigener
Wuschel-CI-Job wiederholt diese Prüfung mit dem gepinnten Quellcommit.
Figurenbudgets, Alpha-Verarbeitung und Versionen, Kontaktbögen und Firefox-
Nachweise stehen in [Wuschels A2-Abnahme](https://github.com/levtos/wuschel/blob/agent/codex-alpha10-wuschel/docs/alpha10/ABNAHME.md).

Traber verwendet lokale Bilder unter `custom_components/traber/brand/`.
Der von Benni/Claude gelieferte Befund HA 2026.10.2 erfüllt die Mindestversion
2026.3; keine erneute Live-Abfrage in A2. Die HA-Seitenleiste verwendet weiterhin
`mdi:script-text-outline`. Erkennungsfarben bleiben von Statusfarben getrennt.

Keine Änderungen an Home Assistant, LXC, Geheimnissen oder Originalbildern.
Technische Prüfung ist keine Live-Abnahme. Beide PRs bleiben für Bennis A2-
Review offen; Merge, Release und Betriebsmaßnahmen bleiben gesperrt.
