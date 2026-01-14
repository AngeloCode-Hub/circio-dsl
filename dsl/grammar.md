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
---

## 5. Flow DSL (YAML)

### 5.1 Basisschema (v0.2)
Een flow bevat minimaal:
- `id`, `name`
- `inputs` (typed)
- `nodes`
- `connections`
- `outputs`
- optioneel: `error_handling`

### 5.2 Router node (branching)
Een `router` node bepaalt de volgende node op basis van regels:


- id: route_on_validation
  type: router
  input: validation_result
  input_type: ValidationResult
  routes:
    - when: validation_result.is_valid == true
      goto: create_account
    - when: validation_result.is_valid == false
      goto: end_with_error

### 5.3 Condition syntax (v0.2)

Conditions zijn eenvoudige expressies die gebruikt worden in `router` nodes.

Ondersteund:
- Vergelijkingen: `==`, `!=`, `>`, `<`, `>=`, `<=`
- Boolean waarden: `true`, `false`
- Dot-notation voor velden: `validation_result.is_valid`

Voorbeelden:
- `validation_result.is_valid == true`
- `score > 0.8`
- `user_profile.email != null`

### 5.4 Terminal node (v0.2)

Een `terminal` node beëindigt de flow expliciet.

```yaml
- id: end_with_error
  type: terminal
  result: error
  message: "Validation failed"
