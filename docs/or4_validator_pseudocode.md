# OR4 Validation Pseudo-code (v0.1)

Dit document beschrijft hoe een Circio FlowSpec statisch wordt gevalideerd
volgens OR4-correctness regels, vóór executie of visualisatie.

De validator accepteert een FlowSpec (YAML) en retourneert een lijst OR4-errors.

---

## Data-structuren

- FlowSpec:
  - inputs
  - nodes
  - connections
  - outputs
  - error_handling (optioneel)

- ValidationError:
  - code
  - message
  - severity
  - node_id (optioneel)
  - path (optioneel)

---

## Hoofd-algoritme


function validateFlow(flowSpec):
errors = []

errors += validateRequiredFields(flowSpec)
errors += validateUniqueNodeIds(flowSpec.nodes)
errors += validateNodeTypeRules(flowSpec.nodes)
errors += validateConnections(flowSpec)
errors += validateTypeConsistency(flowSpec)
errors += validateRouterTargets(flowSpec.nodes)
errors += validateErrorHandling(flowSpec)
errors += validateTermination(flowSpec)

return errors


---

## Validatiestappen (detail)

### 1. Required fields


if flowSpec.nodes is empty:
add OR4_MISSING_NODES error


---

### 2. Unieke node-ids


seen = set()
for node in nodes:
if node.id in seen:
add OR4_DUPLICATE_NODE_ID error
else:
seen.add(node.id)


---

### 3. Node-type regels


for node in nodes:
if node.type == "function":
require fields: function, input, input_type, output, output_type

if node.type == "api_call":
    require fields: api, input, input_type, output, output_type

if node.type == "router":
    require fields: input, input_type, routes
    if routes is empty:
        add OR4_EMPTY_ROUTER error

if node.type == "terminal":
    require result in {success, error}


---

### 4. Connections validatie


for connection in connections:
if source node does not exist:
add OR4_UNKNOWN_NODE error

if target node does not exist:
    add OR4_UNKNOWN_NODE error


---

### 5. Type-consistentie (TMAP)


for connection in connections:
sourceType = output_type of source node
targetType = input_type of target node

if sourceType != targetType:
    add OR4_TYPE_MISMATCH error


---

### 6. Router targets


for node in nodes where type == router:
for route in node.routes:
if route.goto does not exist:
add OR4_INVALID_GOTO error


---

### 7. Error handling


for rule in error_handling:
if rule.on does not exist:
add OR4_UNKNOWN_NODE error

if rule.goto does not exist:
    add OR4_INVALID_GOTO error


---

### 8. Termination check


if no terminal node is reachable:
add OR4_NO_TERMINATION error


---

## Resultaat

- Geen errors → FlowSpec is OR4-valid
- ≥1 error → FlowSpec is ongeldig
- OR1 visualisatie wordt alleen gestart bij OR4-valid flows
