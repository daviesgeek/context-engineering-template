# Initialize Multi-Agent Workflow

Initialize a new multi-agent workflow from a feature request file.

## Arguments
$ARGUMENTS

## Instructions

You are the **Workflow Initializer**. Your job is to analyze a feature request and set up a coordinated multi-agent workflow to implement it.

---

## Step 1: Validate Input

Read the feature request from `$ARGUMENTS`:

**If path is provided:**
```bash
view $ARGUMENTS
```

**If no path or file doesn't exist:**
```bash
view PRPs/00-Requests/
```
Then list available requests and ask user which one to use.

---

## Step 2: Generate Workflow ID and Create Structure

Create workflow ID and directory in one step:

```bash
# Generate ID
WORKFLOW_ID="wf_$(date +%Y%m%d_%H%M%S)"

# Create structure (REQUIRED COMMAND - use exactly as shown)
mkdir -p "workflows/${WORKFLOW_ID}/artifacts/"{requirements,research,architecture,code,tests,validation,docs}
mkdir -p "workflows/${WORKFLOW_ID}/logs"

# Copy request file
cp "$ARGUMENTS" "workflows/${WORKFLOW_ID}/request.md"
```

**Required directories created:**
- `workflows/{workflow_id}/artifacts/{requirements,research,architecture,code,tests,validation,docs}`
- `workflows/{workflow_id}/logs/`
- `workflows/{workflow_id}/request.md` (copy of original)

---

## Step 3: Analyze Request & Select Pattern

Parse the request and classify:

| Aspect | Options |
|--------|---------|
| **Type** | NEW_FEATURE, BUG_FIX, REFACTOR, ENHANCEMENT, DOCUMENTATION |
| **Complexity** | SIMPLE, MEDIUM, COMPLEX, LARGE |
| **Domain** | BACKEND, FRONTEND, FULLSTACK, DATABASE, INFRASTRUCTURE |

**Pattern Selection:**
- **sequential-simple**: SIMPLE bugs/docs
- **concurrent-standard**: MEDIUM/COMPLEX features (most common)
- **concurrent-full**: COMPLEX FULLSTACK features
- **hierarchical-modular**: LARGE multi-module refactors

---

## Step 4: Initialize State File

**CRITICAL: Use create_file tool, not cat or echo**

Create `workflows/{workflow_id}/state.json`:

```json
{
  "workflow_id": "{workflow_id}",
  "status": "initialized",
  "pattern": "{selected_pattern}",
  "created_at": "{ISO-8601 timestamp}",
  "updated_at": "{ISO-8601 timestamp}",
  "request": {
    "file": "{relative_path_to_request}",
    "type": "{NEW_FEATURE|BUG_FIX|etc}",
    "complexity": "{SIMPLE|MEDIUM|COMPLEX|LARGE}",
    "domain": "{BACKEND|FRONTEND|etc}"
  },
  "agents": {
    "orchestrator": {
      "status": "active",
      "started_at": "{ISO-8601 timestamp}"
    }
  },
  "execution_plan": {
    "phases": [
      {
        "phase": 1,
        "name": "requirements",
        "agents": ["requirements-analyst"],
        "parallel": false,
        "blocking": true,
        "status": "pending"
      },
      {
        "phase": 2,
        "name": "research",
        "agents": ["research-codebase", "research-docs"],
        "parallel": true,
        "blocking": true,
        "status": "pending"
      },
      {
        "phase": 3,
        "name": "architecture",
        "agents": ["architecture"],
        "parallel": false,
        "blocking": true,
        "status": "pending"
      },
      {
        "phase": 4,
        "name": "generation",
        "agents": ["{domain-specific agents}"],
        "parallel": true,
        "blocking": true,
        "status": "pending"
      },
      {
        "phase": 5,
        "name": "integration",
        "agents": ["integration"],
        "parallel": false,
        "blocking": true,
        "status": "pending"
      },
      {
        "phase": 6,
        "name": "finalization",
        "agents": ["validation", "documentation"],
        "parallel": true,
        "blocking": true,
        "status": "pending"
      }
    ],
    "current_phase": 1,
    "total_phases": 6
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

**Customize phase 4 agents based on domain:**
- BACKEND: `["codegen-backend", "database", "test-generation"]`
- FRONTEND: `["codegen-frontend", "test-generation"]`
- FULLSTACK: `["codegen-backend", "codegen-frontend", "database", "test-generation"]`
- DATABASE: `["database", "test-generation"]`
- DOCUMENTATION: `["documentation"]` (skip other phases)

---

## Step 5: Create Event Log

**CRITICAL: Use create_file tool to write, not cat/echo**

Create `workflows/{workflow_id}/events.log`:

```
[{ISO-8601}] WORKFLOW_INITIALIZED | workflow_id={id} | pattern={pattern} | type={type}
```

---

## Step 6: Invoke Requirements Analyst

Load the agent definition:

```bash
view .claude/agents/requirements-analyst/agent.md
```

Then invoke the requirements-analyst agent with:
- **Input**: Content from `workflows/{workflow_id}/request.md`
- **Workflow ID**: `{workflow_id}`
- **Output Path**: `workflows/{workflow_id}/artifacts/requirements/requirements.json`

The agent will produce a requirements artifact following `requirements.schema.json`.

---

## Step 7: Handle Clarifications

After requirements-analyst completes:

1. **Read the requirements artifact:**
   ```bash
   view workflows/{workflow_id}/artifacts/requirements/requirements.json
   ```

2. **Check the clarifications_needed array:**

   **ALWAYS pause and show clarifications to the user** (even if all are non-blocking).

3. **Display clarifications in priority order:**

   Show blocking questions first, then non-blocking questions with defaults:
   
   ```markdown
   ## ⚠️ Clarifications Required
   
   The requirements analysis identified the following questions. 
   Please review and provide answers, or say "use defaults" to accept suggestions.
   
   ---
   
   ### BLOCKING QUESTIONS (Must be answered)
   
   **CLR-001 (BLOCKING)**
   Q: Should JWT tokens be stored in httpOnly cookies or localStorage?
   Context: Security implications differ significantly
   Suggested: httpOnly cookies
   
   Your answer: _______
   
   ---
   
   ### OPTIONAL CLARIFICATIONS (Can use defaults)
   
   **CLR-002**
   Q: What should the access token expiration time be?
   Context: Shorter times are more secure but less convenient
   Default: 24 hours ✓
   
   **CLR-003**
   Q: Should we implement "remember me" functionality?
   Context: Extends refresh token lifetime
   Default: No ✓
   
   ---
   
   **To continue, please:**
   - Answer all BLOCKING questions
   - Optionally override any defaults
   - Or say "use defaults" to accept all suggestions
   ```

   Update state.json status to `"awaiting_input"` and **STOP**.
   
   **Log event:**
   ```
   [{timestamp}] AWAITING_CLARIFICATIONS | blocking={count} | total={count}
   ```

4. **When user responds:**
   
   a. Parse the user's answers
   
   b. Validate that all BLOCKING questions are answered
   
   c. For non-blocking questions without explicit answers, use suggested defaults
   
   d. Update requirements.json with all resolutions:
      ```json
      "clarifications_needed": [
        {
          "id": "CLR-001",
          "blocking": true,
          "resolved": true,
          "resolution": "httpOnly cookies",
          "resolved_at": "{timestamp}"
        },
        {
          "id": "CLR-002",
          "blocking": false,
          "resolved": true,
          "resolution": "24 hours",
          "resolved_at": "{timestamp}",
          "used_default": true
        }
      ]
      ```
   
   e. Update state.json to `"in_progress"`
   
   f. Append to events.log:
      ```
      [{timestamp}] CLARIFICATIONS_RESOLVED | blocking_answered={count} | defaults_used={count}
      ```
   
   g. Continue to Phase 2

5. **If clarifications_needed array is empty:**
   
   Log event and continue directly:
   ```
   [{timestamp}] NO_CLARIFICATIONS_NEEDED | proceeding_to_research
   ```

---

## Step 8: Report Status (Concise)

Display a compact status report:

```markdown
✅ **Workflow Initialized: `{workflow_id}`**

**Analysis:** {type} | {complexity} | {domain}
**Pattern:** {pattern_name}
**Phase:** 1/6 - Requirements Analysis

---

**Next Steps:**
- Requirements analyst is running
- Check status: `/workflow-status {workflow_id}`
- View logs: `view workflows/{workflow_id}/events.log`

⏱️ Started: {timestamp}
```

**That's it. Keep it brief.**

---

## Allowed Bash Commands

You MUST use these specific commands for file operations:

### Directory Creation
```bash
mkdir -p workflows/{workflow_id}/artifacts/{requirements,research,architecture,code,tests,validation,docs}
mkdir -p workflows/{workflow_id}/logs
```

### File Operations
**DO NOT use cat, echo, or >> for file creation/editing**

Instead:
- **Creating new files**: Use `create_file` tool
- **Editing existing files**: Use `str_replace` tool
- **Reading files**: Use `view` tool

### Copying Files
```bash
cp PRPs/00-Requests/{request}.md workflows/{workflow_id}/request.md
```

---

## Error Handling

| Error | Action |
|-------|--------|
| Request file missing | List available files in PRPs/00-Requests/, ask user |
| Request file empty | Show format example, ask user to fix |
| Directory creation fails | Check permissions, report error |
| Agent invocation fails | Log to events.log, update state.json, report to user |

---

## Simplified Directory Structure

**We use two directories with clear purposes:**

1. **PRPs/00-Requests/** (INPUT ONLY)
   - Simple folder for feature request markdown files
   - User creates requests here
   - Never modified by workflow

2. **workflows/{workflow_id}/** (EXECUTION)
   - All workflow state and artifacts
   - Created and managed by agents
   - Self-contained per workflow

**We do NOT use:**
- PRPs/01-Requirements/ ❌ (use workflows/{id}/artifacts/requirements/)
- PRPs/02-Research/ ❌ (use workflows/{id}/artifacts/research/)
- PRPs/03-Architecture/ ❌ (use workflows/{id}/artifacts/architecture/)
- etc.

Keep it simple: **Requests in PRPs, execution in workflows.**

---

## Example Invocation

```
/init-workflow PRPs/00-Requests/jwt-authentication.md
```

Creates:
```
workflows/wf_20241203_154530/
├── request.md
├── state.json
├── events.log
├── artifacts/
│   ├── requirements/
│   ├── research/
│   ├── architecture/
│   ├── code/
│   ├── tests/
│   ├── validation/
│   └── docs/
└── logs/
```

Invokes requirements-analyst, checks for clarifications, reports status.
