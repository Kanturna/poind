# Evaluationsauftrag Fuer Claude

## Rolle

Du bist externer Architektur- und Simulationsreviewer fuer Poind. Deine Aufgabe ist keine Implementierung, sondern eine kritische Analyse des aktuellen Projektfundaments, bevor Slice 1 umgesetzt wird.

Bewerte, ob die geplante Territoriums-Simulation, die Architekturregeln und die Slice-Reihenfolge stabil genug sind, um daraus einen testbaren Godot-Prototyp zu bauen.

## Kontext

Poind ist ein Godot 4.6 Projekt. Der aktuelle Fokus ist keine individuelle Koerper-/Kampf-Simulation, sondern eine Kolonie- und Territoriums-Simulation auf einem quadratischen Raster.

Kernidee:

- 20 Kolonien starten mit je einem Block.
- Bloecke duerfen nur auf leeren Zellen platziert werden.
- Die erste Wachstumsregel nutzt nur den zuletzt gesetzten Block als Wachstumspunkt.
- Von diesem Wachstumspunkt werden nur 8 Nachbarn geprueft.
- Nach Platzierung wird lokal geprueft, ob leere Flaechen eingeschlossen wurden.
- Eingeschlossene leere Flaechen werden der Kolonie zugeordnet.
- Gegnerische Zellen werden in der ersten Version nie ueberschrieben.
- Ressourcen, Energie, scripted strategies und learning agents kommen erst spaeter.

## Bitte Lies Diese Dateien

Lies zuerst:

1. `AGENTS.md`
2. `docs/PLANUNGSAUFTRAG.md`
3. `docs/ARCHITEKTUR.md`
4. `docs/SIM_RULES.md`
5. `docs/DECISIONS.md`
6. `docs/NEXT_STEPS.md`
7. `docs/FINDINGS.md`

Nutze diese Autoritaetsreihenfolge, falls Dokumente widersprechen:

```text
ARCHITEKTUR > DECISIONS > SIM_RULES > STATUS > NEXT_STEPS > AGENTS > README
```

## Analysefokus

### 1. Konzept Und Simulationsregeln

Pruefe:

- Ist der Pivot von individueller Koerper-Simulation zu Territoriums-Simulation klar genug dokumentiert?
- Sind die Kernregeln der ersten spielbaren Version eindeutig?
- Erzeugen die Regeln voraussichtlich interessante Strategie, oder droht zu frueh triviales Verhalten?
- Ist `last placed block` als alleiniger Wachstumspunkt fuer Slice 1 sinnvoll, oder ist das Risiko von Dead-Ends zu hoch?
- Ist die Entscheidung "nur leere Felder, kein Overwrite" konsistent mit Capture und Konflikt?
- Ist die Mischung aus 8-neighbor placement und 4-neighbor capture flood fill gut begruendet?

### 2. Architektur Und Datenhoheit

Pruefe:

- Sind Layer und Ownership scharf genug getrennt?
- Ist klar, welche Daten Simulation Truth sind?
- Ist `TerritorySimulationService.place_block(colony_id, coord)` als geplante Placement-Grenze ausreichend?
- Gibt es versteckte Risiken, weil WorldGrid, ColonyState, Capture und Runtime Snapshots noch nicht konkret spezifiziert sind?
- Fehlen Invarianten, die vor Slice 1 dokumentiert werden sollten?

### 3. Slice-Reihenfolge

Pruefe:

- Ist Slice 1 ohne Capture, Ressourcen und Rendering sinnvoll klein genug?
- Ist Slice 2 als separater Capture-Slice richtig abgegrenzt?
- Sollte eine minimale Lab-Ansicht frueher kommen, oder ist erst headless Core besser?
- Sind die Non-Goals pro Slice klar genug?
- Gibt es Abhaengigkeiten, die spaeter teure Umbauten verursachen koennten?

### 4. Tests Und Validierung

Pruefe:

- Sind die geplanten Headless-Tests fuer Slice 1 ausreichend?
- Welche konkreten Edge Cases fehlen fuer Placement?
- Welche konkreten Edge Cases fehlen fuer Capture in Slice 2?
- Welche Performance-Gates sollten frueh definiert werden?
- Wie sollte deterministisches Seeding geprueft werden?

### 5. Agenten Und Lernen

Pruefe:

- Ist die Entscheidung "scripted baselines vor learning agents" stark genug?
- Sind die geplanten Baselines sinnvoll?
- Welche Beobachtungen und Aktionen sollten spaeter dokumentiert werden?
- Gibt es Risiken, dass die Regeln fuer Learning Agents zu arm oder zu instabil sind?

## Gewuenschtes Output-Format

Bitte antworte strukturiert in Deutsch.

Beginne mit einer kurzen Gesamtbewertung:

```text
Gesamturteil: tragfaehig / tragfaehig mit Risiken / nicht tragfaehig
```

Danach liste konkrete Findings nach Prioritaet:

```text
P0 - Blocker
- Titel
- Befund
- Warum das vor Slice 1 blockiert
- Konkrete Empfehlung

P1 - Sollte vor oder in Slice 1 geklaert werden
- Titel
- Befund
- Risiko
- Konkrete Empfehlung

P2 - Kann spaeter geklaert werden
- Titel
- Befund
- Empfehlung
```

Wenn du keine Findings in einer Kategorie hast, schreibe explizit:

```text
Keine P0-Findings.
```

Schliesse mit:

- empfohlenem naechsten Schritt
- Liste der Dokumente, die deiner Meinung nach geaendert werden sollten
- kurze Antwort auf die Frage: "Kann Slice 1 gestartet werden?"

## Grenzen

- Keine Code-Aenderungen vorschlagen, ausser sie sind fuer Slice 1 konkret relevant.
- Keine alternative Grossvision ausarbeiten.
- Keine ML-Architektur vorziehen.
- Keine individuellen Koerper-, Waffen- oder Projektilsysteme wieder in den Startscope ziehen.
- Keine pauschalen Aussagen ohne Bezug auf die aktuellen Dokumente.

## Bewertungsmassstab

Gut ist ein Fundament, wenn:

- die Regeln testbar sind
- die ersten Slices klein genug und nicht kuenstlich klein sind
- Simulation Truth eindeutig bleibt
- Performance-Risiken frueh sichtbar sind
- offene Entscheidungen bewusst dokumentiert sind
- der naechste Umsetzungsschritt ohne Interpretationsarbeit gestartet werden kann
