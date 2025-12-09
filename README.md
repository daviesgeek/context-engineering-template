# Context Engineering: Agentic Workflow System

A multi-agent workflow system for AI-assisted software development. Based on [coleam00's context-engineering-intro](https://github.com/coleam00/context-engineering-intro), heavily modified to pass context into smaller, specialized agents while keeping the main context for orchestration.

## Quick Start

1. **Create a feature request** in `PRPs/00-Requests/`:
   ```markdown
   ## FEATURE:
   Describe what you want to build...

   ## EXAMPLES:
   - examples/relevant/file.py

   ## DOCUMENTATION:
   - https://docs.example.com/api
   ```

2. **Initialize workflow**: `/init-workflow PRPs/00-Requests/my-feature.md`

3. **Monitor progress**: `/workflow-status <workflow-id>`

4. **Resume if paused**: `/resume-workflow <workflow-id>`

## Overview

Instead of one large AI agent trying to do everything, this system uses **specialized agents** that each handle a specific concern:

```
┌─────────────────────────────────────────────────────────────────────┐
│                         ORCHESTRATOR                                │
│                    (Coordinates everything)                         │
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
| 100k+ tokens per feature | ~70k tokens (40-60% savings) |
| Sequential execution     | Parallel where possible      |
| Single point of failure  | Fault-tolerant with recovery |
| Generic prompts          | Specialized expertise        |

## Project Structure

```
├── .claude/
│   ├── agents/              # Specialized agent definitions (agent.md)
│   ├── commands/            # Slash commands (init-workflow, status, resume)
│   ├── orchestration/       # Workflow patterns (sequential, concurrent, etc.)
│   └── protocols/           # Artifact schemas for agent communication
├── PRPs/                    # Feature requests & outputs
│   ├── 00-Requests/         # Initial feature requests
│   ├── 01-Requirements/     # Structured requirements
│   ├── 02-Research/         # Research artifacts
│   ├── 03-Architecture/     # Design documents
│   ├── 10-Generated-Code/   # Code outputs
│   ├── 20-Validated/        # Passed validation
│   ├── 30-Deployed/         # Shipped features
│   └── 99-Archive/          # Completed/abandoned
├── use-cases/               # Example implementations
├── research/                # Research documents
└── workflows/               # Runtime state (created when workflows run)
```

## Workflow Patterns

| Pattern | Best For | Flow |
|---------|----------|------|
| Sequential Simple | Bug fixes, docs, small changes | Requirements → Research → Codegen → Validation |
| Concurrent Standard | New features | Requirements → [Research ‖ Research] → Architecture → [Backend ‖ Frontend ‖ Tests] → Validation |
| Concurrent Full | Complex full-stack | All agents in parallel where possible |
| Hierarchical Modular | Large refactors | Sub-workflows for each module |

## Workflow Versions

This system includes three workflow initialization commands with progressive capabilities:

| Version | Command | Input Sources | Output Directory | Checkpoints |
|---------|---------|---------------|------------------|-------------|
| V1 | `/init-workflow` | File path | `workflows/wf_*` | None |
| V2 | `/init-workflow-v2` | File path | `workflows/wf_YYYYMMDD_HHMMSS/` | None |
| V3 | `/init-workflow-v3` | GitHub, GitLab, files | `workflows/YYYY-MM-DD_{slug}/` | Yes |

### V1: Basic (`/init-workflow`)

The original implementation with basic state tracking.

```bash
/init-workflow PRPs/00-Requests/my-feature.md
```

### V2: Enhanced (`/init-workflow-v2`)

Adds structured clarification handling and improved state management.

**Key Features:**
- Clarification handling (blocking vs optional questions)
- Event logging to `events.log`
- Domain-specific agent selection for code generation
- 6-phase execution plan

**Directory Structure:**
```
workflows/wf_20241208_154530/
├── request.md          # Copy of original request
├── state.json          # Workflow state
├── events.log          # Event history
└── artifacts/
    ├── requirements/
    ├── research/
    ├── architecture/
    ├── code/
    ├── tests/
    ├── validation/
    └── docs/
```

**Usage:**
```bash
/init-workflow-v2 PRPs/00-Requests/my-feature.md
```

### V3: Full Featured (`/init-workflow-v3`) - Recommended

The most complete implementation with human-in-the-loop checkpoints and multi-source input support.

**Key Features:**
- **Multi-source input**: GitHub issues, GitLab issues, or local files
- **Human-readable run IDs**: `2024-12-08_add-jwt-authentication`
- **Checkpoint gates**: Approval required after each phase
- **Slim mode**: Token optimization by passing only essential context to agents
- **Auto-increment**: Handles duplicate runs on same day (`-2`, `-3`, etc.)

**Input Sources:**
```bash
# GitHub issue
/init-workflow-v3 gh#123
/init-workflow-v3 https://github.com/owner/repo/issues/123

# GitLab issue
/init-workflow-v3 gl#456
/init-workflow-v3 https://gitlab.com/owner/repo/-/issues/456

# Local file
/init-workflow-v3 prompts/my-feature.md
```

**Directory Structure:**
```
runs/2024-12-08_add-jwt-authentication/
├── prompt.md           # Copy of original prompt or fetched issue
├── state.json          # Workflow state with checkpoints
├── events.log          # Event history
└── artifacts/
    ├── requirements/
    ├── research/
    ├── architecture/
    ├── code/
    ├── tests/
    ├── validation/
    └── docs/
```

**Checkpoint Types:**
| Type | When Used | Description |
|------|-----------|-------------|
| `quick` | Research, Integration | Simple y/n confirmation |
| `review` | Requirements, Architecture, Generation | Detailed artifact review |
| `summary` | Finalization | Comprehensive workflow report |

**Slim Mode:**

When enabled (`config.slim_mode: true`), agents receive only:
- Role & core responsibility
- Output artifact schema
- Run ID and paths
- Previous phase artifact **summaries** (not full content)

This reduces token usage while maintaining agent effectiveness.

## Agents

| Agent | Responsibility |
|-------|----------------|
| orchestrator | Coordinate workflow execution |
| requirements-analyst | Extract and structure requirements |
| research-codebase | Find patterns in existing code |
| research-docs | Gather external documentation |
| architecture | Design technical solutions |
| codegen-backend | Generate backend code |
| codegen-frontend | Generate frontend code |
| database | Handle data layer code |
| test-generation | Create test suites |
| integration | Connect components |
| validation | Verify quality |
| documentation | Generate docs |

## State & Recovery

- Workflow state persists to `workflows/{id}/state.json`
- Errors trigger retry (max 3) or pause for human input
- Resume any workflow with `/resume-workflow <id>`

## Contributing

To add a new agent:
1. Create `.claude/agents/{agent-name}/agent.md`
2. Update workflow patterns to include agent
3. Add artifact schema if new output type

## License

MIT
