# Routing Test Cases (Manual Simulation)

## Case 1 — Security + High
Inputs:
- review_focus = "security"
- strictness_level = "high"
Expected:
- route_target: secondary
- agent: bug_triage_agent
Explain highlights:
- matched rule 1 ( (security OR performance) AND high )
- skill check: required ["code_review"] -> PASS (bug_triage has code_review)

---

## Case 2 — Security + Low
Inputs:
- review_focus = "security"
- strictness_level = "low"
Expected:
- route_target: primary
- agent: bug_triage_agent
Explain highlights:
- rule 1 fails (high is false)
- rule 2 matches (security)
- skill check PASS

---

## Case 3 — Readability + Low
Inputs:
- review_focus = "readability"
- strictness_level = "low"
Expected:
- route_target: primary
- agent: code_review_agent
Explain:
- rule 1 fails
- rule 2 fails
- rule 3 matches
- skill check PASS (code_review_agent has code_review)

---

## Case 4 — Performance + High
Inputs:
- review_focus = "performance"
- strictness_level = "high"
Expected:
- route_target: secondary
- agent: bug_triage_agent
Explain:
- rule 1 matches due to (performance OR security) and high

---

## Case 5 — Unknown focus => fallback
Inputs:
- review_focus = "ux"
- strictness_level = "medium"
Expected:
- route_target: fallback
- agent: generalist_agent
Explain:
- rule 1 fails
- rule 2 fails
- rule 3 fails
- default matched

---

## Failure Case 6 — Skill mismatch (simulate)
Change flow requires to:
- requires.skills = ["security_analysis"]

Then run:
Inputs:
- review_focus = "readability"
- strictness_level = "low"

Expected:
- rule 3 matches code_review_agent, BUT skill check FAIL (no security_analysis)
- default fallback generalist_agent (if you allow fallback even if mismatch)
Explain:
- "Rule matched but skill check failed" then fallback selected
