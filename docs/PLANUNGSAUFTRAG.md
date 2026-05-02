# Poind Planungsauftrag

## Zweck

Dieser Planungsauftrag fasst die bisherige Konzeptarbeit zusammen und definiert den Arbeitsauftrag fuer die ersten Poind-Slices. Er ist kein finales Gamedesign-Dokument, sondern ein belastbarer Startvertrag fuer Architektur, Methodik und die erste spielbare Territoriums-Simulation.

## Ausgangspunkt

Die erste Konzeptphase drehte sich um individuelle Einheiten:

- Punkte, Kreise oder Quadrate als Koerper
- Kontaktkampf, Massegewinn und Masseschaden
- Rasterkoerper mit Zellen, Waffen, Antrieb und Trefferstatus
- viele kleine Suchprojektile, die zuerst gegnerische Projektile abfangen und sonst Schiffe angreifen
- Wachstum oder Anbau einzelner Koerperteile

Diese Richtung bleibt als Ideenpool erhalten, ist aber nicht der aktuelle Startpunkt. Der unmittelbare Projektfokus wurde auf eine abstraktere Kollektivsimulation verschoben.

## Aktueller Fokus

Poind startet als Kolonie- und Territoriums-Simulation:

- Die Welt ist ein feines quadratisches Raster.
- Mehrere Kolonien konkurrieren um Raum, Ressourcen und eingeschlossene Flaechen.
- Kolonien bestehen zuerst nicht aus Individuen, sondern aus Territoriumsbloecken.
- Konflikt entsteht durch Blockieren, Einkreisen, Abschneiden und Ressourcenkontrolle.
- Lernen oder komplexe Agenten kommen erst nach deterministischen Regeln und scripted baselines.

## Kernregeln Der Ersten Spielbaren Version

Start:

- Die Simulation startet mit mehreren Kolonien, default 20.
- Jede Kolonie platziert genau einen Startblock.
- Startpositionen muessen eindeutig und innerhalb der Karte sein.

Platzierung:

- Ein Block darf nur auf einem leeren Feld platziert werden.
- Ein Block darf nicht auf gegnerische oder eigene Bloecke gesetzt werden.
- Die erste Wachstumsregel nutzt nur den zuletzt erfolgreich platzierten Block als Wachstumspunkt.
- Von diesem Wachstumspunkt werden nur die 8 direkten Nachbarn geprueft.
- Wenn kein Nachbarfeld frei ist, ist die Kolonie im ersten Modell blockiert.

Capture:

- Nach jeder Platzierung wird lokal geprueft, ob dadurch eine leere Flaeche eingeschlossen wurde.
- Geprueft werden nur leere Nachbarregionen um den neu gesetzten Block.
- Eine Region ist gewonnen, wenn sie den Kartenrand nicht erreicht und nur von Bloecken derselben Kolonie begrenzt wird.
- Gewonnene leere Zellen werden der Kolonie zugeordnet.
- Gegnerische Zellen werden in der ersten Version nie ueberschrieben.
- Platzierung nutzt 8-neighbor adjacency.
- Capture-Flood-Fill nutzt 4-neighbor connectivity.

Ressourcen und Energie:

- Ressourcen kommen nach dem deterministischen Kern.
- Kolonien sollen Energie aus eigenen Ressourcenfeldern erhalten.
- Platzierungen kosten Energie.
- Ressourcen in eingeschlossenen Flaechen werden dadurch strategisch wertvoll.

Agenten:

- Zuerst entstehen einfache scripted strategies.
- Beispiele: random, resource-seeking, enclosure-seeking, blocking.
- Learning agents kommen erst, wenn Regeln, Tests und Baselines stabil sind.

## Architekturauftrag

Die Architektur folgt bewusst dem Baktorium-Vorbild, aber mit Poind-spezifischen Regeln:

- `core/` enthaelt reine Grid-Mathematik und deterministische Hilfen.
- `sim/` besitzt die Simulation Truth: Grid, Ownership, Kolonien, Placement, Capture, Ressourcen, Energie.
- `runtime/` erzeugt spaeter read-only snapshots fuer Renderer, HUD und Debug.
- `rendering/` zeichnet Snapshots, aber mutiert keine Simulation.
- `scenes/` komponieren Lab, Input und Tick Driver.
- `ui/` und `debug/` beobachten oder steuern nur ueber explizite Sim-APIs.

Harte Invarianten:

- Ein Territory Block ist Datenmodell, kein Godot Node.
- Die Grid-Ownership wird nur durch den Simulation Service geaendert.
- Die geplante oeffentliche Placement-API ist `TerritorySimulationService.place_block(colony_id, coord)`.
- Ordinary growth scannt kein komplettes Territorium.
- Capture wird durch neue Platzierung ausgeloest und bleibt lokal zum neuen Block.
- Visualisierung darf niemals Simulation Truth werden.

## Arbeitsmethodik

Der Projektstart ist dokumentations- und testgetrieben:

- `AGENTS.md` ist der Arbeitsvertrag fuer Agenten.
- `docs/ARCHITEKTUR.md` beschreibt Layer und Ownership.
- `docs/DECISIONS.md` haelt ADRs fest.
- `docs/SIM_RULES.md` beschreibt aktuelle Sim-Regeln.
- `docs/STATUS.md` und `docs/NEXT_STEPS.md` halten Zustand und naechste Arbeit synchron.
- `docs/FINDINGS.md` sammelt Risiken, offene Fragen und Review-Befunde.

Vor Gameplay-Komplexitaet kommen:

- klare Regeln
- kleine deterministische Slices
- headless tests
- reproduzierbare Seeds
- einfache sichtbare Lab-Pruefung

## Slice-Plan

### Slice 0: Planung Und Projektvertrag

Ziel: Projektmethodik, Architektur und Sim-Regeln dokumentieren.

Status: gestartet und weitgehend umgesetzt.

Ergebnis:

- `AGENTS.md`
- Architekturdocs
- Decision Log
- Sim Rules
- Status, Next Steps, Findings
- dieser Planungsauftrag

### Slice 1: Deterministischer Core

Ziel: reine Datenbasis ohne Visualisierung.

Scope:

- Grid-Koordinaten und Neighbor-Helfer
- WorldGrid mit leer/owner occupancy
- ColonyState mit `growth_origin`
- Placement-Legalitaet
- deterministische Startpositionen fuer 20 Kolonien
- headless tests fuer Start, Placement und Occupancy

Nicht enthalten:

- Capture
- Ressourcen
- Rendering
- Agent Learning

### Slice 2: Capture-Algorithmus

Ziel: Einkreisungen deterministisch und performant pruefen.

Scope:

- lokale Neighbor-Regionen nach Placement
- Flood Fill mit Early Exit am Kartenrand
- Boundary-Owner-Pruefung
- Fill gewonnener leerer Regionen
- Tests fuer Ringe, Lecks, diagonale Ecken und Mixed-Owner-Boundaries

Nicht enthalten:

- visuelle Politur
- Ressourcen
- komplexe Agenten

### Slice 3: Erste Lab-Ansicht

Ziel: Simulation sichtbar und manuell pruefbar machen.

Scope:

- einfache Grid-Darstellung
- Kolonie-Farben
- Pause, Step, Reset, Seed
- Anzeige von Growth Origin und letzter Platzierung
- optionaler Capture-Highlight

Nicht enthalten:

- Beauty Rendering
- Animation
- ML

### Slice 4: Ressourcen Und Energie

Ziel: strategischen Druck erzeugen.

Scope:

- Ressourcenfelder
- Kolonie-Energie
- Placement-Kosten
- Einkommen aus eigenen Ressourcenfeldern
- erste scripted strategies mit Ressourcenbezug

Nicht enthalten:

- komplexe Oekonomie
- Learning Agents

### Slice 5: Strategien Und Agenten

Ziel: Kolonien vergleichbar handeln lassen.

Scope:

- Random Agent
- Resource-Seeking Agent
- Enclosure-Seeking Agent
- Blocking Agent
- dokumentierter Observation/Action Contract

Nicht enthalten:

- Neural Training Loop vor stabilen Baselines

## Akzeptanzkriterien Fuer Den Start

Vor Slice 1 sollten bestaetigt sein:

- Der Projektfokus ist Territorium statt individueller Koerper.
- Strict `last placed block` bleibt die erste Wachstumsregel.
- Bloecke duerfen nur auf leere Felder gesetzt werden.
- Capture ueberschreibt keine gegnerischen Zellen.
- 8-neighbor placement und 4-neighbor capture fill sind als erster Default akzeptiert.
- Scripted baselines kommen vor learning agents.

Slice 1 ist fertig, wenn:

- 20 Kolonien reproduzierbar starten koennen.
- Platzierung nur auf legalen leeren Nachbarfeldern moeglich ist.
- Occupancy nicht direkt ausserhalb des Simulation Service mutiert wird.
- Headless tests die Kernregeln abdecken.

## Bekannte Risiken

- Strict `last placed block` kann Kolonien zu schnell blockieren.
- Lokale Capture-Checks koennen trotzdem grosse offene Regionen flood-fillen.
- 4-neighbor capture macht diagonale Ecken zu geschlossenen Grenzen.
- Passive Capture-Flaechen erzeugen nicht automatisch neue Wachstumsfronten.
- Learning Agents ohne scripted baselines waeren nicht sinnvoll bewertbar.

Diese Risiken sind in `docs/FINDINGS.md` dokumentiert und sollen nicht stillschweigend geloest oder ignoriert werden.

## Offene Entscheidungen

- konkrete Kartengroesse fuer Slice 1
- ob Karten spaeter wraparound haben duerfen
- ob blockierte Kolonien sterben, warten oder spaeter alternative growth origins erhalten
- ob Capture-Flaechen dauerhaft passiv bleiben
- welche Ressourcen zuerst existieren
- welche Metrik spaeter Erfolg misst: Flaeche, Ressourcen, Ueberleben, Score oder Mischform

## Auftrag An Den Naechsten Umsetzungsschritt

Der naechste sinnvolle Umsetzungsschritt ist Slice 1.

Arbeitsauftrag:

1. Keine Visualisierung bauen.
2. Keine Capture-Regeln implementieren.
3. Den deterministischen Grid- und Placement-Core aufbauen.
4. Simulation Truth strikt in `sim/` halten.
5. Placement nur ueber den geplanten Service erlauben.
6. Headless tests fuer Startpositionen, Legalitaet und Occupancy schreiben.
7. Dokumente nach Abschluss synchronisieren.
