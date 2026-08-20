[English](README.md) | [繁體中文](README.zh.md)

# code-diet

Scan the codebase for duplication, dead code, and abstraction opportunities — then propose minimal shared extractions.

## Description

Code Diet runs three parallel scans across the Workshop codebase to find near-identical logic across modules, unused functions and imports, and places where modules solve the same problem differently. It applies a strict four-question decision framework before recommending any extraction to shared code, ensuring that only patterns with two or more confirmed consumers ever move to `shared/`. The output is a prioritized diet report saved to `~/.claude/outputs/code-diet/`.

## Features

- Three parallel scans: duplication detection, dead code detection, pattern divergence
- Four-question extract-vs-keep decision framework (2+ consumers, domain-agnostic, generic signature, future reuse)
- Classifies findings as `→shared`, `✂️ dead`, `→adopt`, or `✓ keep`
- Prioritized report with effort estimates (quick wins < 30 min, moderate 1-3 hours, future consideration)
- Scope control: target a single module, a keyword, or the full codebase
- Results stored as intelflow intelligence report for trend tracking
- Composition-over-inheritance philosophy: functions, not class hierarchies

## Usage

```
/code-diet [scope]
```

Examples:

- `/code-diet` — scan `core/src/` (default)
- `/code-diet full` — scan core/ + stations/ + mcp/ + libs/
- `/code-diet memvault` — focus on memvault module vs shared
- `/code-diet search` — only search-related code paths
- "整理重複程式碼"
- "找死碼"

## How It Works

Code Diet spawns three parallel explorer agents targeting duplication, dead code, and pattern divergence respectively. A plan agent then applies the four-question framework to each finding, producing a classified and prioritized list. The final report is written to `~/.claude/outputs/code-diet/` and persisted as an intelflow report. Execution of proposed changes is intentionally left to the developer — the skill proposes, humans decide.

## Requirements

- Claude Code CLI
- *(optional)* A report store CLI, if you keep one. Without it the report is simply left on disk.
- Access to `core/src/` and optionally `stations/`, `mcp/`, `libs/`

## License

MIT
