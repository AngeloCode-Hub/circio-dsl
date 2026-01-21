# OR1 – UI State Schema (v0.1)

Dit document beschrijft hoe Circio flows visueel weergegeven worden (OR1).
De visualisatie is gebaseerd op:
- OR4 validatie (pre-execution)
- Runtime node states (during execution)

Doel: een UI kan hiermee consistente kleuren/labels/iconen gebruiken, ongeacht implementatie (SwiftUI/Web).

---

## 1. Twee lagen in de UI

### 1.1 Validation layer (OR4)
Deze laag wordt getoond vóór execution.
Als OR4-errors bestaan, wordt execution niet gestart.

Validation status:
- `valid`  → execution toegestaan
- `invalid` → toon errors, geen execution

### 1.2 Runtime layer (OR1)
Deze laag toont node states tijdens execution.
Runtime states zijn alleen relevant als OR4-validatie geslaagd is.

---

## 2. Node states (runtime)

NodeState enum (v0.1):
- `idle`
- `running`
- `success`
- `error`

### State semantics
- `idle`    → node nog niet uitgevoerd
- `running` → node is actief
- `success` → node uitgevoerd zonder fouten
- `error`   → node faalde of werd naar error afgehandeld

---

## 3. UI mapping (kleuren + iconen)

### 3.1 Node weergave
Per node wordt minimaal getoond:
- node id
- node type (function/api_call/router/terminal)
- state indicator (kleur/icoon)
- optioneel: input/output keys

### 3.2 Kleur-/icoon mapping (conceptueel)
UI gebruikt consistente mapping:

- `idle`    → neutraal (grijs)
- `running` → actief (blauw)
- `success` → positief (groen)
- `error`   → fout (rood)

Iconen (voorbeeld):
- `idle`    → circle
- `running` → spinner / play
- `success` → check
- `error`   → x / warning

(Implementatie kan iconen aanpassen, zolang de betekenis gelijk blijft.)

---

## 4. Edge/connection visualisatie (dataflow)

Edges representeren `connections`.

### 4.1 Edge status (v0.1)
Een edge kan 3 UI-states hebben:

- `inactive`  → nog niet gebruikt (default)
- `active`    → data is overgedragen via deze connection
- `invalid`   → OR4 error op connection (bestaat niet / type mismatch)

Mapping (conceptueel):
- inactive → dun/neutraal
- active   → highlight (bijv. dikker of glow)
- invalid  → rood/gestippeld + fouttooltip

---

## 5. Router branch visualisatie (control-flow)

Router nodes hebben routes (`when` → `goto`).

### 5.1 Route state (v0.1)
- `not_taken` → route bestaat maar werd niet gekozen
- `taken`     → route gekozen tijdens execution
- `invalid`   → OR4 error (goto bestaat niet / condition parse fail)

UI gedrag:
- `taken` highlighten
- `not_taken` dimmen
- `invalid` rood markeren + tooltip

---

## 6. Error panel (OR4 + runtime)

De UI toont een error panel met gestructureerde errors.

### 6.1 OR4 errors (pre-execution)
Als OR4 faalt:
- toon lijst errors (code, message, node_id, path)
- highlight gerelateerde nodes/edges

### 6.2 Runtime errors (during execution)
Als runtime faalt:
- toon laatste node in `error`
- toon runtime error + (optioneel) error_handling route die gekozen werd

---

## 7. Event hooks (voor real-time UI)

Execution engine kan events publiceren:

- NodeStarted(nodeId)
- NodeSucceeded(nodeId)
- NodeFailed(nodeId, error)
- EdgeActivated(from, to)
- RouteTaken(routerId, gotoNodeId)
- ContextUpdated(key)
- FlowFinished(result)

UI update states op basis van events.

---

## 8. Minimal UI requirements (v0.1)

De UI is OR1-compliant als:
- node states zichtbaar zijn (idle/running/success/error)
- router “taken route” zichtbaar is
- errors zichtbaar zijn vóór execution (OR4) en tijdens execution (runtime)
- edges minimaal zichtbaar zijn (connections)
