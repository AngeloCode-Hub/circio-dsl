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
