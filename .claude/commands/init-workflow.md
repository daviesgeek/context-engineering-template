# Initialize Multi-Agent Workflow

Initialize a new multi-agent workflow from a feature request file.

## Arguments
$ARGUMENTS

## Instructions

You are the **Workflow Initializer**. Your job is to analyze a feature request and set up a coordinated multi-agent workflow to implement it.

### Step 1: Validate Input

First, read the feature request file provided in `$ARGUMENTS`:

```
Read the file at: $ARGUMENTS
```

If the file doesn't exist or path is invalid:
- List available files in `PRPs/00-Requests/`
- Ask the user which request they want to process
- Stop and wait for clarification

### Step 2: Generate Workflow ID

Create a unique workflow ID using this format:
```
wf_YYYYMMDD_HHMMSS
```

Example: `wf_20241125_143022`

Use the current date and time.

### Step 3: Analyze the Request

Parse the feature request and extract:

1. **Request Type Classification:**
   - `NEW_FEATURE` - Building something new
   - `BUG_FIX` - Fixing existing functionality
   - `REFACTOR` - Restructuring existing code
   - `ENHANCEMENT` - Improving existing feature
   - `DOCUMENTATION` - Documentation only

2. **Complexity Assessment:**
   - `SIMPLE` - Single component, straightforward
   - `MEDIUM` - Multiple components, some integration
   - `COMPLEX` - Many components, significant integration
   - `LARGE` - Requires hierarchical breakdown

3. **Domain Focus:**
   - `BACKEND` - Server-side logic
   - `FRONTEND` - UI components
   - `FULLSTACK` - Both frontend and backend
   - `DATABASE` - Data layer focused
   - `INFRASTRUCTURE` - DevOps/deployment

4. **Required Agents** - Determine which specialized agents are needed based on the request.

### Step 4: Select Workflow Pattern

Based on your analysis, select the appropriate orchestration pattern:

**Sequential Pattern** (`sequential-simple`)
- Use for: Simple bug fixes, documentation updates, single-component changes
- Agents run one after another
- Minimal parallelization

**Concurrent Pattern** (`concurrent-standard`)
- Use for: New features, enhancements with multiple components
- Research agents run in parallel
- Code generation agents run in parallel where dependencies allow

**Concurrent Full Pattern** (`concurrent-full`)
- Use for: Complex features requiring all agent types
- Maximum parallelization
- All research, then all generation, then validation

**Hierarchical Pattern** (`hierarchical-modular`)
- Use for: Large refactors, multi-module changes
- Creates sub-workflows for each module
- Sub-orchestrators manage each section

### Step 5: Create Workflow Directory

Create the workflow directory structure:

```
workflows/{workflow_id}/
├── state.json          # Workflow state
├── request.md          # Copy of original request
├── artifacts/          # Agent outputs
│   ├── requirements/
│   ├── research/
│   ├── architecture/
│   ├── code/
│   ├── tests/
│   └── validation/
├── logs/               # Per-agent logs
└── events.log          # Event history
```

### Step 6: Initialize State

Create `state.json` with initial workflow state:

```json
{
  "workflow_id": "{workflow_id}",
  "status": "initialized",
  "pattern": "{selected_pattern}",
  "created_at": "{ISO timestamp}",
  "updated_at": "{ISO timestamp}",
  "request": {
    "file": "{original file path}",
    "type": "{request_type}",
    "complexity": "{complexity}",
    "domain": "{domain_focus}"
  },
  "agents": {
    "orchestrator": {
      "status": "active",
      "started_at": "{ISO timestamp}"
    }
  },
  "execution_plan": {
    "phases": [],
    "current_phase": 0,
    "total_phases": 0
  },
  "artifacts": {},
  "events": [],
  "metrics": {
    "total_tokens": 0,
    "agents_completed": 0,
    "agents_failed": 0
  }
}
```

### Step 7: Create Execution Plan

Based on the selected pattern and required agents, create a detailed execution plan.

**For `concurrent-standard` pattern (most common):**

```json
{
  "phases": [
    {
      "phase": 1,
      "name": "requirements",
      "agents": ["requirements-analyst"],
      "parallel": false,
      "blocking": true
    },
    {
      "phase": 2,
      "name": "research",
      "agents": ["research-codebase", "research-docs"],
      "parallel": true,
      "blocking": true
    },
    {
      "phase": 3,
      "name": "architecture",
      "agents": ["architecture"],
      "parallel": false,
      "blocking": true
    },
    {
      "phase": 4,
      "name": "generation",
      "agents": ["codegen-backend", "codegen-frontend", "database", "test-generation"],
      "parallel": true,
      "blocking": true,
      "note": "Only include agents relevant to domain focus"
    },
    {
      "phase": 5,
      "name": "integration",
      "agents": ["integration"],
      "parallel": false,
      "blocking": true
    },
    {
      "phase": 6,
      "name": "finalization",
      "agents": ["validation", "documentation"],
      "parallel": true,
      "blocking": true
    }
  ]
}
```

Customize the phases based on:
- Remove `codegen-frontend` if domain is `BACKEND`
- Remove `codegen-backend` if domain is `FRONTEND`
- Remove `database` if no data layer changes needed
- For `SIMPLE` complexity, may skip architecture phase
- For `DOCUMENTATION` type, only need documentation agent

### Step 8: Log Initialization Event

Add the initialization event to `events.log`:

```
[{ISO timestamp}] WORKFLOW_INITIALIZED | workflow_id={workflow_id} | pattern={pattern} | request_type={type}
```

### Step 9: Begin Orchestration

Now invoke the **Requirements Analyst** agent to begin the workflow.

Read the requirements analyst agent definition:
```
.claude/agents/requirements-analyst/agent.md
```

Execute the requirements analyst with:
- The original feature request content
- The workflow ID for tracking
- The path to store the requirements artifact

### Step 10: Report Status

After initialization, report to the user:

```markdown
## ✅ Workflow Initialized

**Workflow ID:** `{workflow_id}`
**Pattern:** {pattern_name}
**Status:** In Progress

### Request Analysis
- **Type:** {request_type}
- **Complexity:** {complexity}
- **Domain:** {domain_focus}

### Execution Plan
{List phases with agent names}

### Current Status
- Phase 1/{total_phases}: Requirements Analysis
- Active Agent: requirements-analyst

### Commands
- Check status: `/workflow-status {workflow_id}`
- Resume if paused: `/resume-workflow {workflow_id}`
- View logs: `cat workflows/{workflow_id}/events.log`

---
*Workflow started at {timestamp}*
```

---

## Error Handling

**If request file is empty or malformed:**
- Report the specific issue
- Suggest the correct format (FEATURE, EXAMPLES, DOCUMENTATION, OTHER CONSIDERATIONS sections)
- Ask user to fix and retry

**If workflow directory creation fails:**
- Check permissions
- Report the error
- Suggest manual creation if needed

**If agent invocation fails:**
- Log the error to events.log
- Update state.json with error status
- Report to user with recovery options

---

## Important Notes

1. **State Persistence:** Always write state changes to `state.json` immediately after each action
2. **Event Logging:** Log every significant action to `events.log`
3. **Idempotency:** If the same request is initialized twice, warn the user about existing workflow
4. **Artifacts:** All agent outputs go in `artifacts/` subdirectory with clear naming

---

## Example Invocation

```
/init-workflow PRPs/00-Requests/user-authentication.md
```

This will:
1. Read `PRPs/00-Requests/user-authentication.md`
2. Analyze and classify the request
3. Create `workflows/wf_20241125_143022/`
4. Initialize workflow state
5. Begin requirements analysis phase
6. Report status to user
