---
description: Enforces Cooper-Helmstetter-Donachie (CHD) model validity domain by checking that parameter combinations do not violate the single-genome assumption (doubling time ≥ 20 minutes) and provides pre-submission heuristic guards with biological explanations.
---

# Validity Guard — CHD Assumption Boundary Enforcement

## Purpose

The Unit Cell Architect uses the 2026 Steady-State Cell Model (SSCM), which assumes a single complete genome copy with non-overlapping replication forks. This assumption derives from the Cooper-Helmstetter-Donachie (CHD) model of the bacterial cell cycle and is only valid for doubling times ≥ 20 minutes (1200 seconds).

This skill instructs you to detect when parameter combinations or solver results violate this fundamental assumption, warn the user with a biological explanation, and allow them to acknowledge and proceed if they choose.

---

## The 20-Minute Boundary: Why It Matters

### Biological Basis

In *E. coli*, chromosomal DNA replication takes approximately 40 minutes (the C period) and cell division requires an additional ~20 minutes after replication completes (the D period). When the doubling time is ≥ 20 minutes, the cell can be modeled with a single genome copy undergoing at most one round of replication at a time.

However, when growth rate increases beyond this threshold (doubling time < 20 minutes), *E. coli* initiates **new rounds of DNA replication before the previous round completes**. This creates **multi-fork replication** — multiple replication forks simultaneously traversing the chromosome, with the cell effectively containing multiple partial genome copies.

### Why the SSCM Cannot Represent This

The SSCM's stoichiometric balance equations assume:
- A single complete genome per unit cell
- Non-overlapping replication forks
- DNA content proportional to one genome copy

Multi-fork replication violates all three assumptions. The actual DNA content, replication machinery requirements, and nucleotide demands scale non-linearly with overlapping forks. Results computed below the 20-minute boundary are **extrapolations beyond the model's validated domain** — they may appear numerically converged but do not reflect biological reality.

---

## Pre-Submission Heuristic Checks

Before invoking the `python_solver` or any tool that accepts parameter overrides, perform the following heuristic checks on the accumulated parameter overrides:

### Check 1: Ribosome Working Rate

If `krs` (ribosome working rate) is set to a value **greater than 25 aa/s**, issue a warning:

> ⚠️ **CHD Boundary Warning**: The ribosome working rate `krs` = {value} aa/s exceeds 25 aa/s. At this translation speed, the cell's overall biosynthetic capacity may push the doubling time below 20 minutes, violating the single-genome CHD assumption. The default value is 20 aa/s; values above 25 aa/s approach the physical maximum for *E. coli* ribosomes and imply very fast growth.

### Check 2: Cell Cycle Length

If `td` (cell cycle doubling time) is set to a value **less than 1200 seconds**, issue a warning:

> ⚠️ **CHD Boundary Warning**: The cell cycle parameter `td` = {value} s is below 1200 seconds (20 minutes). This directly specifies a doubling time in the multi-fork replication regime. The SSCM single-genome model cannot accurately represent cells growing this fast.

### Check 3: Combined Rate Increases

If the user has made **multiple rate increases** that collectively suggest sub-20-minute doubling, issue a warning. Specifically, flag when:

- Two or more working rates (`krp`, `krs`, `klpe`, `kdp`, `kstp`, `ketc`, `ktrna`, `kenz`) are each increased by more than 20% above their defaults, OR
- Any working rate is increased by more than 50% above its default, OR
- The combination of rate increases and pathway length decreases suggests the cell would need to divide faster than every 20 minutes to maintain steady state

> ⚠️ **CHD Boundary Warning**: The combined parameter modifications (increased rates: {list_of_changes}) collectively suggest a growth rate that may exceed the 20-minute doubling threshold. At these biosynthetic speeds, *E. coli* would require multi-fork replication, which the SSCM single-genome model cannot represent.

### Heuristic Check Procedure

1. Before each solver invocation, review the current accumulated overrides.
2. Apply checks 1, 2, and 3 in order.
3. If ANY check triggers, present the warning to the user BEFORE invoking the tool.
4. Wait for user acknowledgment before proceeding (see "User Acknowledgment" below).

---

## Post-Solve Flagging

After receiving results from `python_solver`, check the `doubling_time_seconds` field in the response:

### If `doubling_time_seconds` < 1200:

Flag the result prominently:

> ⚠️ **Result Outside Model Validity Domain**
>
> The solver returned a doubling time of {doubling_time_seconds} seconds ({doubling_time_seconds/60:.1f} minutes), which is below the 20-minute CHD boundary.
>
> **What this means**: At this growth rate, real *E. coli* cells would initiate overlapping rounds of DNA replication (multi-fork replication). The SSCM assumes a single genome copy with non-overlapping forks, so this result is an **extrapolation beyond validated model boundaries**.
>
> The computed molecule counts, membrane utilization, and energy balance may not reflect biological reality at this growth rate. Use these results with caution — they indicate trends but not quantitatively accurate predictions.

This flag should appear regardless of whether pre-submission heuristics were triggered (the solver may produce sub-20-minute doubling from parameter combinations that the heuristics did not catch).

---

## User Acknowledgment and Proceeding Past Warnings

When a CHD boundary warning is issued (either pre-submission or post-solve):

1. **Present the warning clearly** with the ⚠️ indicator and biological explanation.
2. **Ask the user** if they want to proceed:
   - "Would you like me to proceed with this solver invocation anyway? The results will be extrapolations beyond the model's validated domain."
3. **If the user acknowledges and requests execution**:
   - Proceed with the tool invocation.
   - Include a note in the response: "Note: These results are computed outside the CHD validity domain (doubling time < 20 min). The single-genome assumption is violated at this growth rate."
   - Do NOT re-issue the same warning for subsequent invocations in the same conversation unless the parameters change further.
4. **If the user does not acknowledge or asks to adjust**:
   - Suggest parameter modifications that would bring the design back within the validity domain.
   - Offer to reduce the offending rates or increase `td` to ≥ 1200 seconds.

### Remembering Acknowledgment

Once a user has acknowledged the CHD warning for a particular parameter set:
- Do not repeat the full biological explanation for the same parameter set.
- If parameters change further (pushing even faster), issue a brief reminder: "Note: Updated parameters still exceed the CHD boundary (td ≈ {estimated} s)."
- If the user resets parameters to within the valid domain and then exceeds it again, issue the full warning again.

---

## Summary of Thresholds

| Check | Condition | Action |
|-------|-----------|--------|
| Pre-submission | `krs` > 25 aa/s | Warn before solver call |
| Pre-submission | `td` < 1200 s | Warn before solver call |
| Pre-submission | Combined rates suggest td < 1200 s | Warn before solver call |
| Post-solve | `doubling_time_seconds` < 1200 | Flag result prominently |

---

## Key Reference Values

- **CHD boundary**: 1200 seconds (20 minutes)
- **Default `krs`**: 20 aa/s (ribosome elongation rate)
- **Default `td`**: model-dependent (~3520 s for minimal medium)
- **C period**: ~40 minutes (time to replicate the chromosome)
- **D period**: ~20 minutes (time from replication completion to division)
- **Physical maximum `krs`**: ~22 aa/s *in vivo*; values above 25 are supraphysiological
