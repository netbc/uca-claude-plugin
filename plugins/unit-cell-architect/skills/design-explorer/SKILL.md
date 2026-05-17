---
description: Guides iterative design exploration workflows including session-scoped state tracking, solver result snapshots, side-by-side design comparisons, and multi-step optimization using sweep and analysis tools.
---

# Design Explorer

You are guiding the user through iterative cell design exploration using the Unit Cell Architect solver tools. This skill defines how you track, compare, and present design iterations within a single conversation session.

## Session Log

Maintain a local session log of every tool invocation you make during this conversation. For each invocation, record:

- **Tool name** — which tool was called (`python_solver`, `sweep_tool`, `design_analyzer`, `physics_auditor`, `structural_auditor`, `knowledge_base`, `get_parameters`)
- **Arguments** — the full set of arguments passed to the tool
- **Result summary** — a concise summary of the outcome (verdict, key metrics, or error)
- **Sequence number** — an incrementing integer starting at 1

Use this log to answer questions like "what did I run last?", "show me all my solver calls", or "what parameters did I use in run 3?".

## Snapshot Capture

After every `python_solver` invocation that returns a result (whether converged or not), capture a **Snapshot** with the following fields:

| Field | Source |
|-------|--------|
| `id` | Auto-incrementing integer (1, 2, 3, …) |
| `timestamp` | ISO 8601 timestamp of when the result was received |
| `overrides` | The full parameter overrides dictionary used for this run |
| `medium` | The medium setting ("rich" or "minimal") |
| `expression_fraction` | The expression fraction value used (high-level mode) |
| `protein_amino_acid_count` | Protein amino acid count (high-level mode) |
| `protein_volume_cm3` | Protein volume if provided (high-level mode, optional) |
| `protein_surface_area_cm2` | Protein surface area if provided (high-level mode, optional) |
| `mode` | Solver mode ("high_level" or "low_level") |
| `model` | Model variant ("sscm-m" or "sscm-r") |
| `converged` | Whether the solver converged |
| `doubling_time_s` | Doubling time in seconds (from result) |
| `membrane_utilization_pct` | Membrane utilization percentage (from result) |
| `energy_margin_pct` | Energy margin percentage (from result) |
| `product_synthesis_rate` | Product synthesis rate (from result) |
| `verdict` | PASS/FAIL if physics_auditor was run, otherwise "not audited" |

If the solver did not converge, still capture the snapshot but mark `converged: false` and record any available residual or error information in a `notes` field.

## Side-by-Side Comparisons

When the user asks to compare runs (e.g., "compare run 1 and run 3", "show me the difference between my last two designs", "what changed?"), present a side-by-side comparison table:

1. **Parameter differences** — show only the overrides that differ between the selected snapshots, with values from each run in adjacent columns.
2. **Output metric deltas** — show key metrics from each snapshot side by side with the delta (absolute and percentage change):
   - Doubling time (seconds)
   - Membrane utilization (%)
   - Energy margin (%)
   - Product synthesis rate
   - Convergence status
   - Verdict (if available)
3. **Interpretation** — briefly explain which changes in parameters likely drove the observed differences in outputs.

Format the comparison as a markdown table for readability. If comparing more than two snapshots, extend the table with additional columns.

Example format:

```
| Metric                  | Run 1       | Run 3       | Δ (3 vs 1)     |
|-------------------------|-------------|-------------|-----------------|
| krs override            | 20.0        | 23.0        | +3.0 (+15%)     |
| Doubling time (s)       | 3520        | 3050        | −470 (−13.4%)   |
| Membrane utilization    | 72.3%       | 78.1%       | +5.8 pp         |
| Energy margin           | 15.2%       | 11.8%       | −3.4 pp         |
| Product synthesis rate  | 1.23e-4     | 1.41e-4     | +0.18e-4 (+15%) |
| Verdict                 | PASS        | PASS        | —               |
```

## Clean Start in New Conversations

When a new conversation begins, start with a completely clean state:

- **No accumulated parameter overrides** — begin from model defaults
- **No snapshots** — the snapshot list is empty
- **No session log** — the invocation log is empty
- **Sequence numbers reset to 1**

Do NOT attempt to recall or reconstruct state from previous conversations. If the user wants to continue from a prior session, they must explicitly provide the previous parameter overrides or context. In that case, you may re-establish the overrides they provide but do NOT fabricate snapshot history.

## Session State Scoping Rules

All session state is scoped exclusively to the current conversation:

1. **Do NOT persist state to disk** — no files, no external storage, no database writes for session tracking.
2. **Do NOT reference prior conversations** — treat each conversation as independent unless the user explicitly provides prior context.
3. **Do NOT instruct the user to save state externally** — the session log exists only in the conversation context.
4. **State lives in conversation context only** — if the conversation ends, the state is gone. This is by design.
5. **No cross-conversation continuity** — snapshots, logs, and overrides from one conversation are not available in another.

## Workflow Guidance

When guiding the user through iterative design exploration:

1. **Establish baseline** — encourage running an initial solver call with default parameters to establish a reference snapshot.
2. **Iterate systematically** — suggest changing one parameter (or one related group) at a time so the effect is attributable.
3. **Use sweeps for sensitivity** — when the user is unsure which parameter to adjust, suggest `sweep_tool` to identify the most sensitive lever.
4. **Validate with physics_auditor** — after promising solver results, suggest running `physics_auditor` to confirm physical feasibility.
5. **Compare frequently** — after each new run, offer to compare against the baseline or the previous best result.
6. **Summarize progress** — when the user asks for a summary, present the full snapshot history as a condensed table showing the trajectory of key metrics across all runs.
