# Circio DSL — Pseudo Engine (Play Mode)

Dit is een simpele “runner” (in woorden/pseudo-code) om te snappen hoe je DSL uitgevoerd wordt.

## 1) Parse
- Lees flow blokken: goal/context/inputs/requires/routing/explain/agents/steps/constraints
- Valideer syntax volgens grammar

## 2) Resolve agents
- Lees agent-definities uit agents/*.yaml
- Maak lookup: agent_id -> agent object (skills/tools/policies)

## 3) Evaluate routing (in volgorde)
Pseudo:

run(flow, inputs):
  trace = new ExplainTrace()

  requiredSkills = flow.requires.skills
  routingRules = flow.routing.rules_in_order

  for rule in routingRules:
    if rule.type == "when":
      result = evalCondition(rule.condition, inputs)

      trace.addCondition(rule.condition, result)

      if result == true:
        candidate = getAgent(rule.agentName)
        trace.setMatchedRule(rule)

        # Skill check
        skillOK = hasSkills(candidate, requiredSkills)
        trace.addSkillCheck(candidate.id, requiredSkills, skillOK)

        if skillOK:
          trace.select(rule.target, candidate.id)
          return executeSteps(flow, candidate, inputs, trace)

        # skill mismatch => continue routing or fallback later
        trace.addNote("Rule matched but skill check failed; continue.")

    if rule.type == "default":
      fallbackAgent = getAgent(rule.agentName)
      trace.setMatchedRule(rule)
      trace.select(rule.target, fallbackAgent.id)
      return executeSteps(flow, fallbackAgent, inputs, trace)

  # If no default rule exists:
  trace.fail("No routing rule matched and no default provided.")
  return trace

## 4) Execute steps
executeSteps(flow, agent, inputs, trace):
  for step in flow.steps:
    if violatesConstraints(step, flow.constraints):
      trace.addConstraintBlock(step, "blocked")
      continue
    trace.addStep(step, "executed_by", agent.id)

  return { result: "done", explain: trace.render(flow.explain) }

# Circio Flow Execution Model (v0.1)

Dit document beschrijft hoe een `flow/*.yaml` runtime wordt uitgevoerd (conceptueel).
Doel: voorspelbare execution + bruikbaar voor visualisatie (OR1) en correctness (OR4).

## 1. Terminologie
- **FlowSpec**: YAML definitie (inputs, nodes, connections, outputs).
- **RuntimeContext**: key-value store met alle tussenresultaten (typed).
- **NodeState**: `idle | running | success | error`.
- **ActiveNode**: node die nu uitgevoerd wordt.
- **Terminal**: expliciet eindpunt (`result: success|error`).

## 2. Runtime data model
### 2.1 Context
Context is een dictionary:
- key: string (bijv. `user_profile`, `validation_result`, `account_id`)
- value: object
- type: type-id (bijv. `UserProfile`, `ValidationResult`)

### 2.2 Node states
Elke node heeft:
- state: `idle|running|success|error`
- started_at / ended_at (optioneel)
- error (optioneel)

## 3. Execution stappen (high-level)
1. **Load FlowSpec**
2. **Validate FlowSpec (OR4 baseline)**
3. **Initialize Context**
   - vul flow `inputs` met runtime input values
4. **Determine Start Node**
   - default: eerste node in `nodes`
   - (later uitbreidbaar: `start: <nodeId>`)
5. **Execute loop**
   - voer nodes uit totdat:
     - een `terminal` node bereikt is, of
     - er geen volgende node bestaat (error), of
     - max-steps overschreden is (safety)

## 4. Node execution semantics
### 4.1 function
- leest `input` key uit Context
- roept `function` aan (FMAP)
- schrijft `output` key naar Context
- set state: running -> success (of error)

### 4.2 api_call
- leest `input` uit Context
- roept `api` aan
- schrijft `output` naar Context

### 4.3 router
- leest `input` object uit Context
- evalueert `routes[].when` in volgorde
- eerste match bepaalt `goto`
- geen match -> error (of later: default route)

### 4.4 terminal
- stopt de flow
- zet flow result = success|error
- message is optional (logging/UI)

## 5. Next-node rules
- Bij `router`: next = `goto`
- Bij overige nodes: next = via `connections`:
  - match op `from: <nodeId>.<output>` -> `to: <nodeId>.<input>`
  - precies 1 volgende connection verwacht (v0.1)
- Geen connection gevonden -> error tenzij node een `terminal` is.

## 6. Error handling (v0.1)
Als een node faalt (exception/type mismatch/api failure):
- check `error_handling`:
  - eerste regel met `on: <nodeId>` matcht
  - jump naar `goto`
- anders: flow stopt met error.

## 7. OR1 hooks (visualisatie)
Tijdens execution publiceer events:
- NodeStarted(nodeId)
- NodeSucceeded(nodeId)
- NodeFailed(nodeId, error)
- ContextUpdated(key)
- FlowFinished(result)

UI kan hiermee real-time states tonen.
