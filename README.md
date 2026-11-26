# course-correct

A harness-agnostic agent skill (`SKILL.md`) for catching long-context drift, sunk-cost patching, and trained model biases during day-to-day engineering work.

## What It Does

When invoked (manually or at long-horizon checkpoints), `course-correct` instructs the active agent to:

1. **Enforce a clean-room firewall** — strip out its own rationalizations and assemble only verbatim user objectives, pointers to ground-truth project docs, and the git/workspace scope.
2. **Spawn a fresh, zero-history, read-only auditor subagent** — anchored on the project goals *before* reading recent code/diffs.
3. **Audit against common agent failure modes** — proxy/spirit drift, hardcoding and premature specialization, glue & sunk-cost patching, silent shortcuts, verification theater, and scope/constraint decay.
4. **Relay findings transparently** — without defensive filtering, with concrete `file:line` citations and actionable pivot options (including what to delete).

## Installation

Clone directly into your harness's skills directory:

```bash
# Claude Code (global)
git clone https://github.com/shisuiki/course-correct.git ~/.claude/skills/course-correct

# Codex (global)
git clone https://github.com/shisuiki/course-correct.git ~/.codex/skills/course-correct

# Antigravity (global)
git clone https://github.com/shisuiki/course-correct.git ~/.gemini/config/skills/course-correct

# Project-local (.agents/skills/)
git clone https://github.com/shisuiki/course-correct.git .agents/skills/course-correct
```

Or copy the single self-contained [`SKILL.md`](./SKILL.md) anywhere your agent discovers skills.
