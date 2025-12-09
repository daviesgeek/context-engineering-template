# Context Engineering: Agentic Workflow System

This originally started from coleam00's [https://github.com/coleam00/context-engineering-intro](context-engineering-intro), but I've changed a LOT, especially and specifically passing off context into smaller and more specialized agents to do work, leaving the main context for orchestration.

## Overview

A multi-agent workflow system for AI-assisted software development. Transforms monolithic AI interactions into coordinated, specialized agent workflows.

Instead of one large AI agent trying to do everything, this system uses **specialized agents** that each handle a specific concern:

```
┌─────────────────────────────────────────────────────────────────────┐
│                         ORCHESTRATOR                                 │
│                    (Coordinates everything)                          │
└────────────────────────────┬────────────────────────────────────────┘
                             │
         ┌───────────────────┼───────────────────┐
         ▼                   ▼                   ▼
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│  Requirements   │ │    Research     │ │   Architecture  │
│    Analyst      │ │   (Codebase)    │ │      Agent      │
└─────────────────┘ └─────────────────┘ └─────────────────┘
                             │
         ┌───────────────────┼───────────────────┐
         ▼                   ▼                   ▼
┌─────────────────┐ ┌─────────────────┐ ┌─────────────────┐
│    Codegen      │ │    Codegen      │ │      Test       │
│   (Backend)     │ │   (Frontend)    │ │   Generation    │
└─────────────────┘ └─────────────────┘ └─────────────────┘
                             │
                             ▼
                   ┌─────────────────┐
                   │   Validation    │
                   │     Agent       │
                   └─────────────────┘
```

## Key Benefits

| Monolithic Approach      | Multi-Agent Approach         |
| ------------------------ | ---------------------------- |
| 100k+ tokens per feature | ~70k tokens (42% reduction)  |
| Sequential execution     | Parallel where possible      |
| Single point of failure  | Fault-tolerant with recovery |
| Generic prompts          | Specialized expertise        |
| Difficult to debug       | Clear agent boundaries       |

## Quick Start

### 1. Create a Feature Request

Create a file in `PRPs/00-Requests/`:

```markdown
## FEATURE:

Describe what you want to build...

## EXAMPLES:

- examples/relevant/file.py - What to reference

## DOCUMENTATION:

- https://docs.example.com/api

## OTHER CONSIDERATIONS:

Any constraints, gotchas, or special requirements
```

### 2. Initialize Workflow

```bash
/init-workflow PRPs/00-Requests/my-feature.md
```

This will:

- Analyze your request
- Select the appropriate workflow pattern
- Create a workflow with unique ID
- Begin executing agents

### 3. Monitor Progress

```bash
/workflow-status wf_20241125_143022
```

### 4. Resume if Paused

```bash
/resume-workflow wf_20241125_143022
```

## Commands

| Command                 | Description                    |
| ----------------------- | ------------------------------ |
| `/init-workflow <file>` | Start new multi-agent workflow |
| `/workflow-status [id]` | Check workflow status          |
| `/resume-workflow <id>` | Resume paused/failed workflow  |

## Workflow Patterns

### Sequential Simple

Best for: Bug fixes, documentation, simple changes

```
Requirements → Research → Codegen → Validation
```

### Concurrent Standard

Best for: New features, enhancements

```
Requirements → [Research || Research] → Architecture →
[Backend || Frontend || Tests] → Integration → Validation
```

### Concurrent Full

Best for: Complex full-stack features

```
Requirements → [Research || Docs] → Architecture →
[Backend || Frontend || Database || Tests] →
Integration → [Validation || Documentation]
```

### Hierarchical Modular

Best for: Large refactors, multi-module changes

```
Requirements → Architecture →
  [Sub-workflow A] ||
  [Sub-workflow B] ||
  [Sub-workflow N] →
Integration → Validation
```

## Project Structure

```
context-engineering-agentic/
├── .claude/
│   ├── commands/                  # User-facing commands
│   │   ├── init-workflow.md
│   │   ├── workflow-status.md
│   │   └── resume-workflow.md
│   ├── agents/                    # Agent definitions
│   │   ├── orchestrator/
│   │   ├── requirements-analyst/
│   │   └── ...
│   ├── orchestration/             # Workflow patterns
│   │   └── patterns/
│   └── protocols/                 # Schemas & standards
│       └── artifact-schemas/
├── workflows/                     # Active workflow state
│   └── {workflow_id}/
│       ├── state.json
│       ├── artifacts/
│       ├── logs/
│       └── events.log
├── PRPs/                          # Feature requests & outputs
│   ├── 00-Requests/
│   ├── 01-Requirements/
│   └── ...
├── examples/                      # Code examples
├── CLAUDE.md                      # Project rules
└── README.md
```

## Agent Reference

| Agent                | Responsibility         | Context Budget |
| -------------------- | ---------------------- | -------------- |
| orchestrator         | Coordinate workflow    | 2k-5k tokens   |
| requirements-analyst | Structure requirements | 3k-8k tokens   |
| research-codebase    | Find code patterns     | 5k-15k tokens  |
| research-docs        | Gather documentation   | 5k-20k tokens  |
| architecture         | Design solution        | 8k-20k tokens  |
| codegen-backend      | Generate backend code  | 5k-15k tokens  |
| codegen-frontend     | Generate frontend code | 5k-15k tokens  |
| database             | Handle data layer      | 4k-12k tokens  |
| test-generation      | Create test suites     | 5k-15k tokens  |
| integration          | Connect components     | 8k-20k tokens  |
| validation           | Verify quality         | 5k-15k tokens  |
| documentation        | Generate docs          | 5k-12k tokens  |

## State Management

Workflow state is persisted to `workflows/{id}/state.json` after every significant action:

- Agent starts/completes
- Phase transitions
- Errors occur
- Human input received

This enables:

- **Recovery**: Resume from any failure point
- **Debugging**: See exactly what happened
- **Monitoring**: Track progress in real-time

## Error Handling

| Error Type             | Recovery Strategy                      |
| ---------------------- | -------------------------------------- |
| Agent timeout          | Retry with extended timeout (up to 3x) |
| Validation failure     | Re-run affected codegen agents         |
| Missing artifact       | Re-run producer agent                  |
| Context overflow       | Pause, suggest simplification          |
| Blocking clarification | Pause for human input                  |

## Token Efficiency

The multi-agent approach is more token-efficient because:

1. **Context Isolation**: Each agent only sees what it needs
2. **Artifact Compression**: Research agents create summaries
3. **No Redundancy**: Information loaded once, passed via artifacts
4. **Parallel Processing**: Multiple agents don't share context

Typical savings: **40-60%** compared to monolithic approach.

## Contributing

To add a new agent:

1. Create directory in `.claude/agents/{agent-name}/`
2. Add `agent.md` with role definition
3. Add `capabilities.json` with metadata
4. Update workflow patterns to include agent
5. Add artifact schema if new output type

## License

MIT
