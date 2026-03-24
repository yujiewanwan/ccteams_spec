# Flow Spec Suite

Spec-driven development workflow for Claude Code. **Five** commands from idea to merge.

> 🌏 [中文版说明](README_zh.md) / Chinese version

## Why not open-spec / spec-kit?

| | open-spec / spec-kit | flow-spec |
|---|---|---|
| Era | Early 2023 | 2024+ |
| Core Assumption | AI needs strong constraints | AI is capable, needs freedom |
| Alignment | Template-based, one-way | Three agents debate in parallel |
| Development | Human writes code | Agent-teams fully automated |
| Best For | Conservative enterprises | Efficiency-focused teams |

## Installation

```bash
# Claude Code Plugin
/plugin marketplace add yujiewanwan/ccteams_spec
/plugin install flow-spec-suite@ccteams_spec

# Or manual
cp -r skills/* .claude/skills/
```

## Update

```bash
# Check for updates
/plugin list

# Update to latest version
/plugin update flow-spec-suite@ccteams_spec

# Or reinstall to force update
/plugin uninstall flow-spec-suite@ccteams_spec
/plugin install flow-spec-suite@ccteams_spec
```

For manual install:
```bash
# Remove old version
rm -rf .claude/skills/flow-*

# Copy new version
cp -r /path/to/ccteams_spec/skills/* .claude/skills/
```

## Uninstall

```bash
# Via plugin manager
/plugin uninstall flow-spec-suite@ccteams_spec

# Or manual
rm -rf .claude/skills/flow-init
rm -rf .claude/skills/flow-new
rm -rf .claude/skills/flow-continue
rm -rf .claude/skills/flow-sync
rm -rf .claude/skills/flow-ff
```

## Quick Start

```bash
# 1. Create spec from scratch (human alignment)
/flow:new
# → PM + Architect + QA agents ask questions in parallel
# → Generates docs/specs/SPEC-001/spec.yaml

# OR: Bootstrap from existing plan/document
/flow:init                    # Auto-detect from context
/flow:init from docs/plan.md  # Explicit file path

# 2. Develop (fully automated)
/flow:continue        # Start agent-teams (FE/BE/QA/Review)
/flow:continue        # Auto test, then human verification
/flow:continue        # Confirm human testing done

# 3. Finish
/flow:sync            # Update docs, output PR checklist
```

**Status flow:** `draft → approved → in-progress → human-testing → done`

## Commands

**Entry points** (choose one):
| Command | What it does | When to use |
|---------|--------------|-------------|
| `/flow:init` | Bootstrap spec from existing docs/context | **Already have a plan/document** |
| `/flow:new` | Three agents align in parallel → generate spec | **Starting from scratch** |

**Development flow** (after init/new):
| Command | What it does |
|---------|--------------|
| `/flow:continue` | Auto-execute next step based on spec.yaml status |
| `/flow:sync` | Update docs, generate PR checklist |
| `/flow:ff` | Fast-forward mode (when all ACs are automated) |

## Artifacts

```
docs/specs/<SPEC-ID>/
├── requirement.md    # Requirements (human-readable)
├── test-cases.md     # Test cases (human-readable)
└── spec.yaml         # Machine contract (drives workflow)
```

## License

MIT
