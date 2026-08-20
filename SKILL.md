---
name: code-diet
description: "code, diet, codebase, scan, dead, find, duplicate, 找重複程式碼, 清理冗餘, 抽共用邏輯, 找沒用到的函式"
allowed-tools: Agent Bash Read Write Edit Grep Glob
io:
  input:
    - mime: "text/plain"
      description: "Optional scope restriction (module name, directory, or 'full')"
  output:
    - mime: "text/markdown"
      description: "Diet report with findings, recommendations, and action items"
disable-model-invocation: true
---

# Code Diet — Periodic Codebase Slimming

Scan for duplication, dead code, and extraction opportunities.
Propose minimal shared abstractions following **composition > inheritance**.

## Philosophy

```
重複只是「醜」，錯誤抽象會「卡住未來演進」。
只抽確定有 2+ 消費者的 pattern，其餘留在模組內。
```

## FSM

```
SCAN → CLASSIFY → EVALUATE → PROPOSE → RECORD
```

## Agent Delegation

| Phase | Agent | Notes |
|-------|-------|-------|
| SCAN | 3 parallel Explore agents | Duplication / Dead code / Pattern divergence |
| EVALUATE | 1 Plan agent | Apply decision framework, prioritize |
| PROPOSE | inline | Generate report + optional execution |

## Phase 1: SCAN — Three Parallel Scans

Launch 3 Explore agents in parallel:

### Scan A: Duplication Detection
Find near-identical code blocks across modules:
- Functions with same logic in 2+ files (>10 lines similar)
- Identical import boilerplate patterns
- Copy-pasted ILIKE/search/CRUD patterns
- Same constant values defined in multiple places

Tools: `Grep` for pattern matching, `Glob` for file discovery, `Read` for confirmation.

### Scan B: Dead Code Detection
Find unused code that should be removed:
- Functions/classes never imported or called
- Stubbed functions that return `[]` or `pass` with TODO (from migrations)
- Deprecated code blocks behind `if False:` or `# LEGACY`
- Imports that are unused (cross-reference with ruff output)

### Scan C: Pattern Divergence
Find places where modules solve the same problem differently:
- Different error handling patterns for the same external service
- Inconsistent retry/fallback strategies
- Mixed naming conventions for similar concepts
- `shared/` utilities that exist but aren't used by eligible modules

## Phase 2: CLASSIFY — Categorize Findings

For each finding from Phase 1, classify:

| Category | Symbol | Action |
|----------|--------|--------|
| Extract to shared | `→shared` | Create shared utility, modules delegate |
| Inline dead code removal | `✂️ dead` | Delete safely |
| Adopt existing shared | `→adopt` | Module should use existing shared fn |
| Keep as-is (domain-specific) | `✓ keep` | Document why it's intentionally separate |

## Phase 3: EVALUATE — Decision Framework

For each `→shared` candidate, apply the 4-question test:

```
1. 2+ 模組有幾乎相同的程式碼？  → NO = keep
2. 邏輯跟特定 domain model 無關？ → NO = keep
3. 函式簽名可以用 generic type？  → NO = keep
4. 未來模組大概率會用？           → NO = keep (unless 1-3 all YES)

All 4 YES → extract to shared/
```

**Anti-patterns to avoid:**
- Builder/Pipeline classes for <5 consumers
- Unified result schemas when module result structures differ
- Abstract base classes when a plain function suffices
- Moving domain-specific logic (Weibull decay, noise filter, domain routing) to shared

**Extraction principles:**
- Function composition > class hierarchy
- Opt-in > mandatory interface
- Convention > abstraction
- `shared/` only if 2+ modules use it

## Phase 4: PROPOSE — Generate Report

Output format (save to `~/.claude/outputs/code-diet/`):

```markdown
# Code Diet Report — {date}

## Summary
- Duplication findings: N
- Dead code findings: N
- Pattern divergence: N
- Recommended extractions: N
- Estimated lines removable: N

## Priority 1: Quick Wins (< 30 min each)
| Finding | Files | Lines | Action |
|---------|-------|-------|--------|
| ... | ... | ... | →shared / ✂️ dead / →adopt |

## Priority 2: Moderate Effort (1-3 hours)
...

## Priority 3: Future Consideration
...

## Not Recommended (Intentionally Separate)
| Code | Reason |
|------|--------|
| ... | Domain-specific / <2 consumers / would create coupling |
```

## Phase 5: RECORD — Store Results

```bash
# Optional: file the report into a report store, if you run one.
# Skip this block entirely when you do not - the report already exists on disk.
# <your-report-cli> --json reports create \
  --title "Code Diet Report: {date}" \
  --query "code duplication shared layer" \
  --skill "code-diet" \
  --tags "code-diet,refactoring,{scope}" \
  --content "<report markdown>"
```

## Scope Control

Default: scan `core/src/` (modules + shared).
Optional narrowing via argument:
- `code-diet full` — core/ + stations/ + mcp/ + libs/
- `code-diet memvault` — only memvault module vs shared
- `code-diet search` — only search-related code paths

## Scheduling Suggestion

Run monthly or after major feature completion. Add to Cronicle as a reminder (not automated execution — human should review proposals before acting).

## Additional Resources

### Reference Files
- **`references/decision-framework.md`** — Extended examples of extract vs keep decisions from 2026-03-17 refactoring
