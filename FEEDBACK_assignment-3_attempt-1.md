## Grade: 99 / 100

**Assignment:** E-Commerce Supply Chain Manager (n8n)  
**Attempt:** 1 of 2  ·  **Graded:** 2026-07-18  ·  Commit `19e6226`

### Score breakdown
| Criterion | Max | Earned | Notes |
|-----------|-----|--------|-------|
| mp_1 | 8 | 8 | System prompt defines a full output JSON schema (plan_id, reasoning, subgoals[], next_subgoal) and the downstream 'Route on Subgoal' Switch routes on next_subgoal across all four types. Docked 1: the end-to-end >=3-branch demo is only a Loom link (analysis.md:59), not verifiable statically; prompt quality itself is top-tier. Demo credit restored: student submitted a working demo video link (a required deliverable); per instructor decision on 2026-07-18, a provided demo earns full credit for this criterion. (`workflows/supply-chain-manager-starter.json (Master Planner Agent node, system prompt); Route on Subgoal switch node`) |
| mp_2 | 10 | 10 | Two worked HTN-style decomposition examples, each showing dependency-aware ordering (forecast inserted as prerequisite for inventory; logistics-only goal). Reflective analysis.md present (1306 words) discussing planning behavior. Full credit. (`workflows/supply-chain-manager-starter.json (Master Planner Agent node: 'Worked example 1' and 'Worked example 2'); analysis.md`) |
| df_1 | 6 | 6 | movingAverage() correctly averages the trailing 3-month window (slice(-3)), divides by actual length so <3 months works, and returns 0 for empty series. Correct. (`custom-nodes/demand-forecast.js:75-80`) |
| df_2 | 8 | 8 | seasonalIndex() correctly computes month-of-year mean / overall mean with sensible 1.0 defaults when overall mean is 0 or no matching month. Combined as ma*si in the main loop. Correct. (`custom-nodes/demand-forecast.js:100-112`) |
| df_3 | 4 | 4 | Prompt takes the numeric forecast + notes, only revises on concrete signals, bounds delta_pct by evidence strength, and emits schema-conformant JSON (revised_forecast, delta_pct, reasoning, context_signals_used). Exceeds requirement. (`workflows/supply-chain-manager-starter.json (Forecast Context Adjuster node)`) |
| eoq_1 | 8 | 8 | eoq() implements Q*=sqrt(2DS/H) with correct D=0/H=0 guards and integer rounding. Correct. (`custom-nodes/eoq-optimizer.js:76-79`) |
| eoq_2 | 10 | 9 | detectViolations() catches the viral SKU (viral_spike: last3 > 2.5x prior9) and the dying SKU (declining: last3 < 0.5x first3), plus low_velocity and long_lead_time (volatility-gated). analysis.md defends each check with real SKU evidence (viral Wool Gloves, declining Earbuds at ~7x reorder). Docked 1: detection is purely demand-trend based and does not itself cross-reference inventory position vs reorder point (the 'below/above reorder' dimension in the descriptor), which is left to the downstream handler. (`custom-nodes/eoq-optimizer.js:111-151; analysis.md (EOQ assumption analysis)`) |
| eoq_3 | 4 | 4 | Prompt gives a per-flag decision rule (reorder/markdown/liquidate/hold), a flag-priority ordering, and a combined declining+low_velocity -> liquidate case, emitting a per-SKU action. Exceeds requirement. (`workflows/supply-chain-manager-starter.json (Inventory Exception Handler node)`) |
| sp_1 | 14 | 14 | Scores all four dimensions with explicit weights (reliability 35, quality 30, lead_time 20, cost 15), per-dimension banding thresholds, the score_total formula, and tiering. analysis.md defends the weighting rationale and honestly notes the LLM's arithmetic drift. Full credit. (`workflows/supply-chain-manager-starter.json (Supplier Performance Monitor node); analysis.md (Supplier rubric defense)`) |
| lg_1 | 6 | 6 | Prompt reasons over the 5 hard constraints (region match, transit vs deadline, weight, perishable) and a soft constraint (prefer more time slack when costs are within 10%). Correct and thorough. (`workflows/supply-chain-manager-starter.json (Logistics Coordinator (LLM) node)`) |
| lg_2 | 10 | 10 | pickCheapestFeasible() filters by all 5 hard constraints, returns null when infeasible, prices survivors by weight*cost_per_kg, and reduces to the minimum-cost option. Correct greedy planner. (`custom-nodes/classical-logistics.js:77-90`) |
| lg_3 | 2 | 2 | Prompt sets use_classical_fallback: true for infeasible/unambiguous cost cases and false only for genuine trade-offs; the IF node routes on it. Correct. (`workflows/supply-chain-manager-starter.json (Logistics Coordinator (LLM) node: use_classical_fallback rules; Use Classical Fallback? IF node)`) |
| fn_1 | 10 | 10 | Code duck-types which subgoal fired, builds business-readable key_findings per branch, and sets next_recommended_subgoal following topological order (forecast->inventory->supplier->logistics->null). Robust envelope parsing and skip-marker filtering. Docked 1: the >=3-branch end-to-end demo is only a Loom link (analysis.md:59), unverifiable statically; code quality itself is top-tier. Demo credit restored: student submitted a working demo video link (a required deliverable); per instructor decision on 2026-07-18, a provided demo earns full credit for this criterion. (`workflows/supply-chain-manager-starter.json (Final Output code node)`) |
| Integrity deduction | — | 0 | Provided files unmodified |
| **Total** | **100** | **99** | |

### What went well
- LLM system prompts are exceptionally well-specified: the Supplier Performance Monitor gives explicit 4-dimension weights (35/30/20/15) with per-dimension banding and a documented rationale, and the Inventory Exception Handler encodes per-flag actions plus a flag-priority ordering and a combined declining+low_velocity -> liquidate rule.
- All five JS algorithms are correct with proper edge-case guards (movingAverage/seasonalIndex defaults, eoq D=0/H=0 guard, pickCheapestFeasible infeasible->null).
- The Master Planner carries two genuine HTN worked examples that teach dependency-aware ordering (forecast as an unstated prerequisite for inventory), not just a schema.
- analysis.md (1306 words) is honest and insightful: it defends supplier weights, ties EOQ flags to real SKUs, and candidly reports the LLM's arithmetic drift and the per-catalog-line API cost inefficiency.

### What to improve (actionable)
- detectViolations() flags on demand trend alone; it could strengthen the descriptor's intent by cross-referencing inventory position vs reorder_point (viral spike while below reorder point, dying SKU while far above it) directly in the detection logic rather than deferring that entirely downstream.
- The end-to-end >=3-branch demo is only a Loom link, so the full pipeline run could not be verified statically; committing a captured run output (or screenshots) would let mp_1/fn_1 earn the top of their bands.
- Both Code nodes rely on $input.all() rather than the real Switch-branch data shape; the documented n8n cross-node-reference bug is a reasonable workaround, but the nodes are not wired to the true pipeline contract.
- The low_velocity check never fires on the demo dataset (noted in analysis.md), so its behavior is untested against real data.

### Automated checks
- ✅ All required files implemented
- ✅ Provided files unmodified
- ✅ 0/0 output artifacts committed
- ✅ Reflection 1306 words

### Resubmission
You may resubmit **once**. Push fixes to this repo, then notify the instructor; we'll re-grade as **Attempt 2 (final)**. This is attempt 1 of 2.

---
*Graded automatically with Claude Code against the course rubric. Questions → contact the instructor.*


---
<sub>🔎 **Autograder record** — attempt 1 of 2 · graded at commit `19e6226` · delivered 2026-07-18T20:42:56Z. Commits pushed to `main` after this timestamp are treated as a resubmission.</sub>
