---
name: course-correct
description: >-
  Audit recent project direction and implementation trajectory against stated
  objectives and user intent by spawning a fresh, clean-room, read-only
  subagent. Use when the user asks to course-correct, sanity-check direction,
  or audit for drift, or proactively on long-horizon tasks when workarounds,
  hardcoded special cases, or model biases may have pulled work off track.
---

# Course Correct

Spawn an independent, zero-history auditor subagent to evaluate recent work
against the project's real objectives and current user intent—breaking long-
context drift, sunk-cost momentum, and trained model biases.

## Core Principle: Clean-Room Firewall

Self-auditing inside a long conversation fails because the current context is
anchored on past rationalizations. You **must** delegate the audit to a **fresh
subagent** (no forked conversation history) and enforce an information firewall:

- **Pass only raw inputs:** verbatim user intent/feedback, pointers to
  canonical project docs (README, specs, plans, decisions), and the workspace /
  git scope to inspect (branch, base ref, or commit range).
- **Withhold your rationalizations:** do **not** send a pre-digested defense of
  *why* recent shortcuts or abstractions were "necessary," and do **not** ask
  leading questions ("confirm that X is reasonable").
- **Require read-only inspection:** the auditor must inspect actual code, diffs,
  and tests—never edit files, mutate state, or take summaries at face value.

---

## Workflow

### Step 1: Assemble the Minimal Audit Brief (No Spin)

Gather four factual items before spawning the auditor:

1. **Stated Objective (Verbatim):** Copy the user's relevant goal statements,
   architectural preferences, or recent pushback word-for-word. Do not soften or
   rephrase.
2. **Ground-Truth Docs:** List paths to the repo's authoritative goal/design
   documents (e.g., `README.md`, `AGENTS.md`, spec/plan/decision docs), if any.
3. **Inspection Scope:** Provide the workspace root, active branch/worktree, and
   recent git range (e.g., `<base>..HEAD` or last `N` commits, plus uncommitted
   diff if non-empty).
4. **Focus Question (Optional):** If the user raised a specific concern, include
   it verbatim; otherwise keep the audit open-ended across the rubric below.

### Step 2: Spawn a Fresh Read-Only Auditor Subagent

Use your harness's subagent tool with a **fresh conversation context** (never
fork/inherit your current conversation history) and **read-only** permissions if
supported. Prefer a strong reasoning model over a lightweight/fast model.

Use this prompt template for the auditor:

```markdown
You are an independent, read-only technical auditor performing a course-
correction review. You have no prior context and no stake in defending existing
work. Be direct, skeptical, and evidence-backed.

## Stated Objectives & User Input (Verbatim)
<paste verbatim user goals, constraints, and recent input>

## Ground-Truth Project Docs
<list paths to specs, README, architecture/decision docs, or "None specified">

## Scope to Inspect
- Workspace: <path>
- Branch / ref range: <e.g., main..HEAD, last 10 commits, plus working tree>

## Audit Protocol
1. **Anchor on the goal first.** Read the stated objectives and ground-truth
   project docs *before* inspecting recent implementation code or diffs.
2. **Inspect actual artifacts, not claims.** Run read-only git commands
   (`git status`, `git log --stat`, `git diff`) and read the actual modified or
   newly created code, interfaces, configs, and tests.
3. **Audit against these concrete drift and bias failure modes:**
   - **Proxy / Spirit Drift:** Optimizing a local metric, milestone, or test
     while missing the underlying goal or violating the intended architecture.
   - **Hardcoding & Megakernel Bias:** Baking task-, model-, or example-specific
     arrangements into code/primitives where the goal calls for generality,
     parameterization, composition, or search.
   - **Glue & Sunk-Cost Patching:** Layering adapters, bypass scripts, special
     cases, or wrappers around a broken abstraction instead of fixing or
     replacing the abstraction.
   - **Silent Pruning & Heuristic Shortcuts:** Inserting arbitrary caps, greedy
     cutoffs, whitelists, or early pruning that silently exclude better general
     solutions.
   - **Evidence / Verification Theater:** Treating mocks, analytical models,
     unwired stubs, or narrow unit tests as end-to-end validation, or adding
     heavy process around an unvalidated core.
   - **Scope Drift & Constraint Decay:** Wandering into tangential work while
     the core loop stays open, or silently dropping earlier user constraints
     and repo rules.
4. **Stay read-only.** Do not modify files, run mutating commands, or commit.

## Required Output Format
Return a concise, blunt report with exact `file:line` or commit citations for
every claim:

1. **Verdict:** One of `ON_TRACK`, `MINOR_DRIFT`, `SIGNIFICANT_DRIFT`, or
   `OFF_TRACK`, followed by a 2–3 sentence plain-English bottom line.
2. **Alignment Check:** Briefly state what *is* genuinely aligned and verified
   (separating verified reality from modeled/untested claims).
3. **Findings (Ordered by Severity: Critical / High / Medium):**
   For each finding:
   - **Failure mode** (from the taxonomy above or another concrete issue)
   - **Evidence** (`file:line`, symbol, diff, or test gap)
   - **Why it conflicts with the stated objective**
   - **Root cause vs. symptom**
4. **Course-Correction Options:** 2–3 concrete paths forward (e.g., surgical
   fix, architectural reset/cutover, or deliberate trade-off to keep), stating
   what to cut/delete—not just what to add—for each option.
```

### Step 3: Relay Findings Without Defensive Filtering

When the auditor returns:

1. **Present the auditor's verdict and findings directly to the user.** Do not
   bury critical findings, soften severity ratings, or silently edit out
   criticism.
2. **Separate any rebuttal explicitly.** If you believe the auditor misread a
   file or missed a hard constraint, add a short, clearly labeled
   `Parent Context Note` with concrete `file:line` counter-evidence rather than
   altering the auditor's report.
3. **Recommend a concrete pivot and wait before major changes.** Highlight the
   recommended option from the auditor's list (including what code/bloat should
   be deleted) and ask the user to confirm before executing destructive
   refactors or architectural pivots—unless the user already asked you to
   course-correct autonomously.

---

## Guardrails

- **Never fake independence:** Do not perform the audit yourself and pretend a
  fresh agent ran it. If your harness genuinely cannot spawn subagents, state
  that explicitly and ask the user whether to run a fresh session or proceed
  with a self-audit flagged as non-independent.
- **No leading context:** If you catch yourself writing paragraphs explaining
  why the current implementation "had to be done this way" in the subagent
  prompt, delete them. Let the code speak for itself.
- **Keep findings grounded:** Reject vague advice ("improve modularity"); if
  the auditor returns uncited claims, ask it in one follow-up to cite exact
  files, lines, and symbols before presenting to the user.
