# Context Engineering Agentic Workflow

This project implements a multi-agent workflow system for AI-assisted software development.

## Project Structure

```
.claude/
├── agents/              # Specialized agent definitions
├── commands/            # User-facing slash commands
├── orchestration/       # Workflow patterns and coordination
└── protocols/           # Communication schemas and standards

workflows/               # Active workflow state and artifacts
PRPs/                   # Feature requests and outputs
examples/               # Code examples for pattern matching
```

## Agent Architecture

This system uses **specialized agents** rather than a monolithic approach:

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

## Workflow Commands

- `/init-workflow <request-file>` - Start a new multi-agent workflow
- `/workflow-status <workflow-id>` - Check workflow status
- `/resume-workflow <workflow-id>` - Resume a paused workflow

## Global Rules

### Code Quality Standards

1. **Type Hints** - All Python code must include type hints
2. **Docstrings** - All public functions/classes need docstrings
3. **Error Handling** - Use custom exceptions, never bare except
4. **Testing** - Minimum 80% coverage for new code
5. **Linting** - Code must pass ruff with default settings

### File Organization

1. **One Responsibility Per File** - Keep files focused (<300 lines)
2. **Clear Naming** - Use descriptive names for files and functions
3. **Consistent Structure** - Follow existing patterns in codebase

### API Design

1. **RESTful** - Follow REST conventions for API endpoints
2. **Standard Responses** - Use consistent response formats
3. **Validation** - Validate all input at API boundary

### Security

1. **No Secrets in Code** - Use environment variables
2. **Input Validation** - Sanitize all user input
3. **Authentication** - Protect sensitive endpoints

## Artifact Standards

All agent outputs must:
- Follow the JSON schema in `.claude/protocols/artifact-schemas/`
- Include metadata (agent, timestamp, confidence score)
- Be valid and parseable by downstream agents

## State Management

- Workflow state persisted to `workflows/{id}/state.json`
- State updated after every significant action
- Events logged to `workflows/{id}/events.log`

## Error Handling

- All errors logged with context
- Recoverable errors trigger retry (max 3)
- Non-recoverable errors pause workflow for human input

## Token Efficiency

- Agents receive only necessary context
- Use artifact summaries when possible
- Full artifacts loaded on-demand only
- Target: <100k total tokens per feature
