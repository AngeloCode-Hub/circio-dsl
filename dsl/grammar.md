# Circio DSL – Grammar & Conventions (v0.2)

Deze repo beschrijft een Domain-Specific Language (DSL) voor gestructureerde AI-interactie en flow-definities.
Doel: flows en agents beschrijven op een manier die (1) menselijk leesbaar is, (2) machine/AI-parsebaar is,
en (3) goed aansluit op visualisatie (OR1) en correctness principes (OR4).

## 1. Kernconcepten

- **Agent**: beschrijving van een AI-agent (doel, inputs/outputs, capabilities, constraints).
- **Flow**: beschrijving van een uitvoerbare keten van stappen (nodes) met data- en control-flow.
- **Node**: één stap in een flow (bijv. api_call, function, transform, router).
- **Port**: input/output van een node (impliciet via `input` en `output` velden).
- **Connection**: verbinding tussen node outputs en inputs.
- **Type (TMAP)**: datatype van inputs/outputs (bijv. `UserProfile`, `AccountId`).
- **Function (FMAP)**: semantische transformatie (bijv. `validateUserProfile`, `mapToCRMContact`).
- **State (OR1)**: runtime status van nodes (idle/running/success/error).

## 2. Bestandslocaties

- `agents/*.yaml`  → agent definities
- `flows/*.agl` of `flows/*.yaml` → flow definities (kies 1 format; YAML aanbevolen voor consistentie)
- `docs/*` → concepten, voorbeelden, rationale

## 3. Conventies

### 3.1 Identifiers
- `id` is lowercase met underscores: `onboarding_agent`, `onboarding_flow`, `create_account`
- Node ids zijn uniek binnen een flow.

### 3.2 Types (TMAP)
- Types zijn PascalCase: `UserProfile`, `AccountId`, `OnboardingResult`
- Elke input/output krijgt een `type`.

### 3.3 Functions (FMAP)
- Functions zijn camelCase: `validateUserProfile`, `mapToAccountRequest`
- Function nodes verwijzen naar `function:`.

### 3.4 Node states (OR1)
Standaard states:
- `idle` (default)
- `running`
- `success`
- `error`

UI kan dit vertalen naar kleur/icoon.

### 3.5 Node Types (v0.2)
Toegestane node types (minimaal):
- `function`  → voert een semantische transformatie uit (FMAP), bijv. validatie of mapping
- `api_call`  → roept een externe API aan
- `router`    → kiest een route op basis van een condition (control-flow)
- `terminal`  → eindpunt van de flow (success/error)

## 4. Agent DSL (YAML)

### 4.1 Schema (v0.1)

```yaml
id: onboarding_agent
name: Circio Onboarding Assistant
description: >
  ...

role: assistant
domain: onboarding

inputs:
  - name: user_profile
    type: UserProfile

outputs:
  - name: onboarding_result
    type: OnboardingResult

capabilities:
  - explain_flow
  - validate_inputs

constraints:
  - input.user_profile must_exist
  - output.onboarding_result type_check

```
## 5. Flow DSL (YAML)

### 5.2 Router node (branching)

```yaml
- id: route_on_validation
  type: router
  input: validation_result
  input_type: ValidationResult
  routes:
    - when: validation_result.is_valid == true
      goto: create_account
    - when: validation_result.is_valid == false
      goto: end_with_error

- id: end_with_error
  type: terminal
  result: error
  message: "Validation failed"

```
## 6. Correctness rules (OR4) (v0.1)

Deze regels beschrijven statische validatie van FlowSpec voordat execution start.

### 6.1 Required fields
Flow:
- `id`, `name`, `inputs`, `nodes`, `outputs` zijn verplicht.
- `connections` verplicht voor flows met >1 node (v0.1).

Node:
- elke node heeft `id`, `type`.
- node `id` uniek binnen de flow.

### 6.2 Node type specific rules
function:
- vereist: `function`, `input`, `input_type`, `output`, `output_type`

api_call:
- vereist: `api`, `input`, `input_type`, `output`, `output_type`

router:
- vereist: `input`, `input_type`, `routes[]`
- elke route heeft: `when`, `goto`
- `goto` verwijst naar bestaande node id
- minimaal 1 route

terminal:
- vereist: `result` in `{success,error}`
- `message` optioneel

### 6.3 Connections rules (wiring)
- `from` moet bestaan als `<nodeId>.<outputName>` of `<nodeId>.output` (v0.1: we gebruiken `.output`)
- `to` moet bestaan als `<nodeId>.<inputName>` of `<nodeId>.input` (v0.1: we gebruiken `.input`)
- `from.nodeId` en `to.nodeId` moeten bestaan
- geen duplicate connections met dezelfde `from`

### 6.4 Type consistency (TMAP)
- `connections` moeten type-compatible zijn:
  - output_type van source node == input_type van target node
- flow `outputs[]` moeten bestaan in Context:
  - output key moet geproduceerd worden door een node output (of flow input)

### 6.5 Reachability
- Elke node (behalve start node) moet bereikbaar zijn via:
  - een connection, of
  - een router `goto`
- Nodes die nooit bereikt kunnen worden -> warning/error (policy: error in v0.1 is ok)

### 6.6 Termination
- Flow moet een gegarandeerde stopmogelijkheid hebben:
  - minstens 1 `terminal` node
  - en die terminal moet bereikbaar zijn
- Router zonder match pad is invalid (v0.1) tenzij later `default` bestaat

### 6.7 Error handling validity
- `error_handling[].on` moet verwijzen naar bestaande node id
- `error_handling[].goto` moet verwijzen naar bestaande node id
- `goto` mag terminal zijn (aanbevolen)

### 6.8 Condition syntax (router.when)
Ondersteund in v0.1:
- `==`, `!=`, `>`, `<`, `>=`, `<=`
- boolean literals: `true|false`
- dot-notation op input object: `validation_result.is_valid`
- null check: `!= null` (optioneel)
Als parsing faalt -> invalid FlowSpec.
