# Initialize Multi-Agent Workflow

Initialize a new multi-agent workflow from a GitHub issue, GitLab issue, or prompt file.

## Arguments
$ARGUMENTS

## Instructions

You are the **Workflow Initializer**. Your job is to fetch/parse a feature request from various sources and orchestrate a multi-agent workflow with approval checkpoints after each phase.

---

## Directory Structure

```
prompts/                              # User-authored prompt files
├── jwt-auth.md
└── user-profile.md

workflows/                                 # Generated artifacts (all workflow output)
└── 2024-12-08_jwt-auth/
    ├── prompt.md                     # Copy of original prompt or fetched issue
    ├── state.json
    ├── events.log
    └── artifacts/
        ├── requirements/
        ├── research/
        ├── architecture/
        ├── code/
        ├── tests/
        ├── validation/
        └── docs/
```

**Key principles:**
- `prompts/` is for authoring — you write here
- `workflows/` is for output — agents write here
- Each run is self-contained with its own prompt copy
- Run folders named: `{YYYY-MM-DD}_{slug}` (slug auto-generated from title, max 50 chars)

---

## Step 1: Parse Input & Fetch Source

Detect the input type and fetch content:

### GitHub Issue
```
Pattern: gh#123, github#123, or full URL https://github.com/owner/repo/issues/123
```

```bash
# Extract issue number and fetch
gh issue view 123 --json title,body,labels,state --jq '{title: .title, body: .body}'
```

### GitLab Issue
```
Pattern: gl#123, gitlab#123, or full URL https://gitlab.com/owner/repo/-/issues/123
```

```bash
# Using glab CLI
glab issue view 123 --output json | jq '{title: .title, body: .description}'

# Or via API if glab not available
curl --header "PRIVATE-TOKEN: $GITLAB_TOKEN" \
  "https://gitlab.com/api/v4/projects/$PROJECT_ID/issues/123" | \
  jq '{title: .title, body: .description}'
```

### File Path
```
Pattern: Any .md file path (typically in prompts/)
```

```bash
view $ARGUMENTS
```

### No Input Provided
```bash
view prompts/
```
List available prompts and ask user which one to use.

---

## Step 2: Generate Slug and Run ID

### Slug Generation Rules

From the title, generate a URL-safe slug:

1. Lowercase the title
2. Replace spaces and special chars with hyphens
3. Remove consecutive hyphens
4. Truncate to 50 characters max (break at word boundary)
5. Remove trailing hyphens

**Examples:**
| Title | Slug |
|-------|------|
| "Add JWT-based authentication for our API" | `add-jwt-based-authentication-for-our-api` |
| "Fix bug in user login flow" | `fix-bug-in-user-login-flow` |
| "Implement comprehensive data export functionality with multiple format support" | `implement-comprehensive-data-export-functionality` |

### Run ID Format

```bash
RUN_ID="$(date +%Y-%m-%d)_${SLUG}"
# Example: 2024-12-08_add-jwt-authentication
```

---

## Step 3: Create Run Structure

```bash
# Create run directory and subdirectories
mkdir -p "workflows/${RUN_ID}/artifacts/"{requirements,research,architecture,code,tests,validation,docs}
```

**Created structure:**
```
workflows/{RUN_ID}/
├── prompt.md           # Copy of original input
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

---

## Step 4: Copy/Create Prompt File

**Use `create_file` tool** to write `workflows/{RUN_ID}/prompt.md`:

```markdown
# {Title}

**Source:** {github|gitlab|file}
**Reference:** {issue URL or file path}
**Fetched:** {ISO-8601 timestamp}

---

{Original body/description content}
```

If source is a file from `prompts/`, copy content directly. If from GitHub/GitLab, include the fetched title and body.

---

## Step 5: Analyze Request & Select Pattern

Parse the request and classify:

| Aspect | Options |
|--------|---------|
| **Type** | NEW_FEATURE, BUG_FIX, REFACTOR, ENHANCEMENT, DOCUMENTATION |
| **Complexity** | SIMPLE, MEDIUM, COMPLEX, LARGE |
| **Domain** | BACKEND, FRONTEND, FULLSTACK, DATABASE, INFRASTRUCTURE |

**Pattern Selection:**
- **sequential-simple**: SIMPLE bugs/docs → 4 phases
- **sequential-standard**: MEDIUM features → 6 phases (default)
- **sequential-full**: COMPLEX/LARGE features → 6 phases with expanded agents

---

## Step 6: Initialize State File

**CRITICAL: Use `create_file` tool**

Create `workflows/{RUN_ID}/state.json`:

```json
{
  "run_id": "{RUN_ID}",
  "slug": "{SLUG}",
  "status": "initialized",
  "pattern": "{selected_pattern}",
  "created_at": "{ISO-8601 timestamp}",
  "updated_at": "{ISO-8601 timestamp}",
  
  "source": {
    "type": "{github|gitlab|file}",
    "reference": "{issue URL or file path}",
    "issue_number": "{number or null}"
  },
  
  "request": {
    "title": "{extracted title}",
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
        "status": "pending",
        "checkpoint": {
          "required": true,
          "type": "review",
          "approved": false
        }
      },
      {
        "phase": 2,
        "name": "research",
        "agents": ["research-codebase", "research-docs"],
        "status": "pending",
        "checkpoint": {
          "required": true,
          "type": "quick",
          "approved": false
        }
      },
      {
        "phase": 3,
        "name": "architecture",
        "agents": ["architecture"],
        "status": "pending",
        "checkpoint": {
          "required": true,
          "type": "review",
          "approved": false
        }
      },
      {
        "phase": 4,
        "name": "generation",
        "agents": ["{domain-specific}"],
        "status": "pending",
        "checkpoint": {
          "required": true,
          "type": "review",
          "approved": false
        }
      },
      {
        "phase": 5,
        "name": "integration",
        "agents": ["integration"],
        "status": "pending",
        "checkpoint": {
          "required": true,
          "type": "quick",
          "approved": false
        }
      },
      {
        "phase": 6,
        "name": "finalization",
        "agents": ["validation", "documentation"],
        "status": "pending",
        "checkpoint": {
          "required": true,
          "type": "summary",
          "approved": false
        }
      }
    ],
    "current_phase": 0,
    "total_phases": 6
  },
  
  "checkpoints": [],
  
  "config": {
    "slim_mode": true,
    "auto_approve_quick": false
  },
  
  "metrics": {
    "total_tokens": 0,
    "agents_completed": 0,
    "agents_failed": 0,
    "checkpoints_passed": 0
  }
}
```

**Checkpoint Types:**
- `quick`: Simple y/n confirmation, minimal details
- `review`: Show artifact summary, request explicit approval
- `summary`: End-of-workflow summary with all outputs

**Domain-specific agents for phase 4:**
- BACKEND: `["codegen-backend", "database", "test-generation"]`
- FRONTEND: `["codegen-frontend", "test-generation"]`
- FULLSTACK: `["codegen-backend", "codegen-frontend", "database", "test-generation"]`
- DATABASE: `["database", "test-generation"]`
- DOCUMENTATION: `["documentation"]` (skip phases 4-5)

---

## Step 7: Create Event Log

**Use `create_file` tool:**

Create `workflows/{RUN_ID}/events.log`:

```
[{ISO-8601}] WORKFLOW_INITIALIZED | run_id={RUN_ID} | source={type} | pattern={pattern}
```

---

## Step 8: Execute Phase 1 - Requirements Analysis

### 8.1 Load Agent (Slim Mode)

When `slim_mode: true`, load only essential agent sections:

```bash
# Load agent definition
view .claude/agents/requirements-analyst/agent.md
```

**Slim Mode Context Extraction:**
Only pass to the agent:
- Role & Core Responsibility (first ~50 lines)
- Output Artifact Schema
- Run ID and input path

Skip:
- Detailed examples
- Full process descriptions (agent knows these)

### 8.2 Invoke Requirements Analyst

Invoke with:
- **Input**: `workflows/{RUN_ID}/prompt.md`
- **Run ID**: `{RUN_ID}`
- **Output Path**: `workflows/{RUN_ID}/artifacts/requirements/requirements.json`

### 8.3 Handle Clarifications

After requirements-analyst completes, check `clarifications_needed`:

**If blocking clarifications exist:**

```markdown
## ⚠️ Clarifications Required (Phase 1: Requirements)

**Run:** `{RUN_ID}`

### Blocking Questions (must answer)

**CLR-001**: {question}
> Context: {context}
> Suggested: {default}

Your answer: _______

---

### Optional (defaults will be used)

**CLR-002**: {question}
> Default: {default} ✓

---

**Reply with your answers, or say "use defaults" to continue.**
```

Update `state.json`:
- Set `status: "awaiting_input"`
- Set `execution_plan.phases[0].status: "awaiting_clarification"`

**STOP and wait for user response.**

---

## Step 9: Phase Checkpoint - Requirements

After clarifications resolved (or if none needed):

### 9.1 Generate Checkpoint Summary

```markdown
## ✓ Phase 1 Complete: Requirements Analysis

**Run:** `{RUN_ID}`
**Duration:** {phase_duration}
**Confidence:** {confidence_score}

### Summary
{2-3 sentence summary of what was extracted}

### Key Requirements
- **Functional:** {count} requirements identified
- **Non-Functional:** {count} constraints
- **Integrations:** {list of systems}

### Clarifications
- Blocking resolved: {count}
- Defaults used: {count}

### Artifacts
- `artifacts/requirements/requirements.json`

---

## Ready for Phase 2: Research

The next phase will:
1. Search codebase for relevant patterns and existing code
2. Fetch external documentation referenced in requirements

**Continue to Phase 2?** (yes/no/show details)
```

### 9.2 Wait for Approval

Update `state.json`:
- Set `status: "checkpoint"`
- Set `execution_plan.phases[0].checkpoint.pending: true`

**STOP and wait for user approval.**

### 9.3 On Approval

When user responds `yes`, `y`, `continue`, or similar:

```bash
# Log checkpoint passed
echo "[{timestamp}] CHECKPOINT_PASSED | phase=1 | name=requirements" >> workflows/{RUN_ID}/events.log
```

Update `state.json`:
- Set `execution_plan.phases[0].status: "complete"`
- Set `execution_plan.phases[0].checkpoint.approved: true`
- Set `execution_plan.phases[0].checkpoint.approved_at: "{timestamp}"`
- Set `execution_plan.current_phase: 2`
- Increment `metrics.checkpoints_passed`

---

## Step 10: Execute Remaining Phases

Repeat the pattern for each phase:

### Phase 2: Research
1. Run `research-codebase` agent
2. Run `research-docs` agent  
3. Combine findings into summary
4. **Checkpoint (quick)**: "Research complete. Found {x} relevant files, {y} patterns. Continue?"

### Phase 3: Architecture
1. Run `architecture` agent with requirements + research artifacts
2. **Checkpoint (review)**: Show component diagram, file structure, key decisions
3. Wait for approval

### Phase 4: Code Generation
1. Run domain-specific codegen agents
2. Run `test-generation` agent
3. **Checkpoint (review)**: Show files created, test coverage plan
4. Wait for approval

### Phase 5: Integration
1. Run `integration` agent
2. **Checkpoint (quick)**: "Integration complete. {x} files modified. Continue to validation?"

### Phase 6: Finalization
1. Run `validation` agent
2. Run `documentation` agent
3. **Checkpoint (summary)**: Full workflow summary with all outputs

---

## Step 11: Workflow Completion

After all phases approved:

```markdown
## ✅ Workflow Complete: `{RUN_ID}`

**Source:** {github issue #123 | gitlab issue #456 | prompts/file.md}
**Duration:** {total_time}
**Phases:** 6/6 complete

### Summary
{Brief description of what was built}

### Generated Files
```
workflows/{RUN_ID}/
├── prompt.md
├── state.json
├── events.log
└── artifacts/
    ├── requirements/requirements.json
    ├── research/codebase.json
    ├── research/docs.json
    ├── architecture/architecture.json
    ├── code/{generated files}
    ├── tests/{test files}
    ├── validation/validation.json
    └── docs/{documentation}
```

### Metrics
| Metric | Value |
|--------|-------|
| Total Tokens | {total} |
| Agents Run | {count} |
| Checkpoints Passed | 6 |
| Clarifications Resolved | {count} |

### Next Steps
1. Review generated code in `artifacts/code/`
2. Run tests: `pytest workflows/{RUN_ID}/artifacts/tests/`
3. Apply changes to your codebase

---

**Commands:**
- View logs: `view workflows/{RUN_ID}/events.log`
- View state: `view workflows/{RUN_ID}/state.json`
```

---

## Checkpoint Type Details

### Quick Checkpoint
Minimal interruption for low-risk phases:

```markdown
## → Phase {n} Complete: {name}

{One-line summary}

**Continue?** (y/n)
```

### Review Checkpoint
Detailed review for critical phases:

```markdown
## → Phase {n} Complete: {name}

### Summary
{2-3 sentences}

### Key Outputs
{Bullet list of important items}

### Decisions Made
{Any architectural or design decisions}

### Potential Concerns
{Any flags or warnings}

---

**Options:**
- `yes` / `y` - Continue to next phase
- `no` / `n` - Pause workflow
- `details` - Show full artifact
- `modify` - Request changes before continuing
```

### Summary Checkpoint
End-of-workflow comprehensive review:

```markdown
## → Workflow Summary

{Full completion report as shown in Step 11}
```

---

## Slim Mode Agent Loading

When `config.slim_mode: true`, minimize context passed to agents:

### What to Include
- Agent role and core responsibility
- Output schema
- Run ID and artifact paths
- Previous phase artifact **summaries** (not full content)

### What to Exclude
- Detailed examples in agent definitions
- Full previous artifacts (use summaries)
- Process descriptions (agents have internalized these)

### Summary Extraction
For each artifact, extract only:
```json
{
  "artifact": "requirements.json",
  "summary": {
    "title": "...",
    "complexity": "...",
    "functional_requirements_count": 5,
    "key_integrations": ["Redis", "User Model"],
    "blocking_clarifications": 0
  }
}
```

---

## Error Handling

| Situation | Action |
|-----------|--------|
| GitHub/GitLab CLI not installed | Show install instructions, offer file path fallback |
| Issue not found | Report error, ask for correct issue number |
| Network error fetching issue | Retry once, then report and offer file path fallback |
| Agent fails | Log error, pause at checkpoint, offer retry or skip |
| User rejects checkpoint | Pause workflow, ask for guidance |
| `prompts/` doesn't exist | Create it: `mkdir -p prompts` |
| `workflows/` doesn't exist | Create it automatically |

---

## Command Variations

```bash
# GitHub issue
/init-workflow gh#123
/init-workflow github#123
/init-workflow https://github.com/owner/repo/issues/123

# GitLab issue
/init-workflow gl#456
/init-workflow gitlab#456
/init-workflow https://gitlab.com/owner/repo/-/issues/456

# File path (from prompts/ or anywhere)
/init-workflow prompts/jwt-auth.md
/init-workflow ./feature-request.md

# Interactive (no argument)
/init-workflow
```

---

## Example Flow

```
User: /init-workflow gh#42

Claude: Fetching GitHub issue #42...

        ✅ Issue fetched: "Add user authentication with JWT"
        
        📋 Analysis:
        - Type: NEW_FEATURE
        - Complexity: MEDIUM  
        - Domain: BACKEND
        - Pattern: sequential-standard (6 phases)
        
        Created run: 2024-12-08_add-user-authentication-with-jwt
        
        Starting Phase 1: Requirements Analysis...

        [Requirements agent runs]

        ## ⚠️ Clarifications Required
        
        **CLR-001 (blocking)**: Should tokens be stored in cookies or localStorage?
        > Suggested: httpOnly cookies
        
        Your answer: _______

User: use cookies

Claude: ✓ Using httpOnly cookies for token storage.

        ## ✓ Phase 1 Complete: Requirements Analysis
        
        - 4 functional requirements identified
        - 2 integrations: Redis, User Model
        - Confidence: 0.87
        
        **Continue to Phase 2: Research?** (y/n)

User: y

Claude: Starting Phase 2: Research...
        [continues...]
```

---

## Handling Multiple Runs for Same Feature

If you run the same prompt twice on the same day, the slug ensures uniqueness:

```
workflows/
├── 2024-12-08_add-jwt-authentication/      # First run
└── 2024-12-08_add-jwt-authentication-2/    # Second run (auto-increment)
```

**Detection logic:**
```bash
# Check if run folder exists, if so, append -2, -3, etc.
BASE="workflows/$(date +%Y-%m-%d)_${SLUG}"
if [ -d "$BASE" ]; then
  COUNT=2
  while [ -d "${BASE}-${COUNT}" ]; do
    COUNT=$((COUNT + 1))
  done
  RUN_ID="$(date +%Y-%m-%d)_${SLUG}-${COUNT}"
else
  RUN_ID="$(date +%Y-%m-%d)_${SLUG}"
fi
```
