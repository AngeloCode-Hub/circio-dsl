## OR4 Error Format (v0.1)

OR4-validatiefouten worden gerapporteerd als gestructureerde objecten
die door de visualisatielaag (OR1) geïnterpreteerd kunnen worden.

### Error object schema

```yaml
code: string            # Unieke foutcode (machine-leesbaar)
message: string         # Menselijk leesbare foutmelding
severity: error|warning
node_id: string|null    # Betrokken node (indien van toepassing)
path: string|null       # DSL-pad (bijv. nodes[2].input_type)

```

## 2️⃣ Concrete OR4 Error Voorbeelden  
(gekoppeld aan `or4_invalid_example.yaml`)

## OR4 Error Voorbeelden

### OR4_TYPE_MISMATCH

```yaml
code: OR4_TYPE_MISMATCH
message: "Input type ValidationResult is not compatible with expected type UserProfile"
severity: error
node_id: create_account
path: nodes.create_account.input_type

```

```yaml
code: OR4_UNKNOWN_NODE
message: "Connection target node 'notify_admin' does not exist"
severity: error
node_id: null
path: connections[1].to

```

```yaml
code: OR4_INVALID_GOTO
message: "Error handling goto target 'non_existing_terminal' does not exist"
severity: error
node_id: create_account
path: error_handling[0].goto
```

## 3️⃣ Hoe OR1 deze OR4 errors visualiseert

Dit stuk kun je **onder OR1–OR4 koppeling zetten**.


## OR4 Errors in OR1 Visualisatie

OR1 visualiseert OR4-validatiefouten vóór runtime-executie.

### Visualisatie-afspraken
- Flows met OR4-errors worden niet uitgevoerd
- OR1 toont een "validation layer" boven de flow

### UI-concept (voorbeeld)
- Node met fout → rood omlijnd
- Edge met fout → rood gestippeld
- Error-panel toont:
  - foutcode
  - boodschap
  - betrokken node

### Mapping
| OR4 element | OR1 visualisatie |
|------------|------------------|
| node_id    | highlight node   |
| path       | tooltip / detail |
| severity   | kleur (rood/geel) |
