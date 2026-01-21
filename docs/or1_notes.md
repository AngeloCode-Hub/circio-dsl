# OR1 – State Management & UI Interactions (Working Notes)

## Doel
Een gebruiker moet in één oogopslag kunnen zien:
- welke node nu actief is
- welke nodes al klaar zijn
- waar een fout is ontstaan
- welke data in/uit een node gaat

## State model (DSL)
De flow-DSL definieert een vaste set node states:

- idle (default)
- running
- success
- error

Deze states zijn bedoeld voor visualisatie in de UI. De DSL beschrijft de mogelijke states,
de runtime (engine/backend) levert later de actuele state per node.

## UI mapping (concept)
- idle    → neutraal
- running → highlight / animatie
- success → duidelijke “done” indicator
- error   → error indicator + message

## Data in/out per node
Per node wordt zichtbaar gemaakt:
- input (naam + type)
- output (naam + type)
Optioneel kan een preview van de actuele data getoond worden tijdens execution.

## Acceptatiecriteria (OR1)
- Een flow toont per node altijd exact één state
- Gebruiker kan execution volgorde volgen
- Errors zijn direct zichtbaar op de juiste node

## Koppeling OR1 (Visualisatie) ↔ OR4 (Correctness)

OR1 richt zich op het visualiseren van flow-executie (node states, data passing, routes).
OR4 garandeert dat deze visualisatie betrouwbaar is doordat flows eerst correct en consistent
moeten zijn voordat ze uitgevoerd of getoond worden.

### Waarom OR4 nodig is voor OR1
Zonder OR4 kan een visualisatie misleidend worden:
- een verbinding kan verwijzen naar een niet-bestaande node → UI kan geen pijl tekenen
- type mismatch in dataflow → UI toont data die nooit correct kan bestaan
- router verwijst naar een ongeldige goto → UI “hangt” of toont een pad dat niet kan lopen
- ontbrekende terminal/error-handling → UI weet niet wanneer een flow stopt

Daarom geldt: **OR1 visualiseert alleen flows die OR4-valid zijn.**

---

## OR1 Visualisatie-elementen die afhankelijk zijn van OR4

### 1) Node State Rendering (idle/running/success/error)
- OR1 toont per node een state (bijv. kleur/icoon).
- OR4 zorgt dat elke node uitvoerbaar is (input/output kloppen), waardoor state-transities betekenisvol zijn.

Voorbeeld:
- `validate_profile` → `success`
- `route_on_validation` → `success`
- `create_account` → `success` of `error`

Als OR4 faalt (bijv. type mismatch), stopt de flow vóór execution.
Dan toont OR1 geen runtime states maar een **validatie-foutstatus**.

---

### 2) Edge/Connection Visualisatie (dataflow pijlen)
- OR1 tekent pijlen tussen `from` en `to`.
- OR4 controleert:
  - `from` bestaat
  - `to` bestaat
  - output-type van `from` is compatibel met input-type van `to`

OR4 bepaalt dus of een edge:
- **groen/valid** (type OK)
- **rood/invalid** (type mismatch of ontbrekende node)
- **grijs/unknown** (nog niet gecheckt)

---

### 3) Router Branch Visualisatie (control-flow)
- OR1 wil laten zien welke route gekozen werd (highlighten).
- OR4 garandeert dat alle `goto` targets bestaan en dat conditions syntactisch valide zijn.

Visualisatiegedrag:
- Niet-uitgevoerde routes → grijs
- Gekozen route → highlight
- Ongeldige route (OR4 failure) → rood + foutmelding “invalid goto target”

---

### 4) Data Inspectie per Node (inputs/outputs)
- OR1 toont per node de input/output payloads.
- OR4 garandeert dat die payloads bestaan en typed zijn (TMAP).

Zonder OR4 zou OR1 data kunnen tonen die semantisch niet klopt of onmogelijk is.

---

## Validatie → Visualisatie flow (pipeline)

1. Flow wordt geladen (YAML parsing)
2. OR4-validatie:
   - ids uniek
   - connections geldig
   - types compatibel
   - router goto targets geldig
   - terminal states aanwezig
3. Als OR4 faalt:
   - OR1 toont “validation layer” errors (geen runtime execution)
4. Als OR4 slaagt:
   - OR1 start runtime visualisatie: node states + edges + gekozen routes

---

## Resultaat
OR4 fungeert als "gatekeeper" voor OR1:
- **OR4 = correctness layer**
- **OR1 = execution/visualization layer**

Hierdoor is de visualisatie altijd betrouwbaar, reproduceerbaar en interpreteerbaar.

Flow loaded
   ↓
OR4 validation
   ↓
Errors?
 ├─ Yes → OR1 toont validatie-errors (geen execution)
 └─ No  → OR1 toont runtime states (idle/running/success/error)
