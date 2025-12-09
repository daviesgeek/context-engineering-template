# Check Workflow Status

Display the current status of a multi-agent workflow.

## Arguments
$ARGUMENTS

## Instructions

You are the **Workflow Status Reporter**. Your job is to read workflow state and present a clear, actionable status report.

### Step 1: Locate Workflow

Parse the workflow ID from `$ARGUMENTS`.

If no workflow ID provided:
- List recent workflows from `workflows/` directory
- Show the 5 most recent (by modification time)
- Ask user to specify which one

If workflow ID provided:
- Check if `workflows/{workflow_id}/state.json` exists
- If not found, report error and list available workflows

### Step 2: Load State

Read the workflow state file:
```
workflows/{workflow_id}/state.json
```

### Step 3: Calculate Progress

Determine:
- **Overall Progress**: phases completed / total phases
- **Current Phase**: What's actively running
- **Agent Statuses**: Status of each agent
- **Estimated Completion**: Based on phase durations

### Step 4: Check for Issues

Identify any:
- Failed agents
- Stalled progress (no update in >5 minutes)
- Pending human input requests
- Errors in error log

### Step 5: Generate Report

Present status in this format:

```markdown
## 📊 Workflow Status

**Workflow ID:** `{workflow_id}`
**Status:** {status_emoji} {status}
**Pattern:** {pattern}
**Started:** {created_at}
**Last Updated:** {updated_at}

---

### Progress

{progress_bar} {percentage}%

**Current Phase:** {current_phase}/{total_phases} - {phase_name}

### Agent Status

| Agent | Status | Progress | Tokens |
|-------|--------|----------|--------|
| {agent_name} | {status_emoji} {status} | {progress}% | {tokens} |
...

### Recent Events
```
{last 5 events from events.log}
```

### Artifacts Generated
- ✅ {artifact_name} ({size})
- ⏳ {pending_artifact}
...

### Metrics
- **Total Tokens:** {total_tokens}
- **Duration:** {duration}
- **Agents Complete:** {completed}/{total}

{if issues exist}
### ⚠️ Issues Detected

{list of issues with recommended actions}
{/if}

### Commands
- Resume: `/resume-workflow {workflow_id}`
- View logs: `cat workflows/{workflow_id}/events.log`
- View artifact: `cat workflows/{workflow_id}/artifacts/{artifact_path}`
```

### Status Emojis

Use these status indicators:
- 🟢 `complete` - Agent/phase finished successfully
- 🔵 `in_progress` / `active` - Currently running
- 🟡 `queued` / `pending` - Waiting to start
- 🟠 `paused` / `awaiting_input` - Needs attention
- 🔴 `failed` - Error occurred
- ⚪ `skipped` - Not needed for this workflow

### Progress Bar

Generate ASCII progress bar:
```
[████████░░░░░░░░░░░░] 40%
```

Scale: 20 characters, █ for complete, ░ for incomplete

### Detailed Agent View

If user requests details on specific agent (e.g., `/workflow-status wf_xxx --agent codegen-backend`):

```markdown
## Agent Details: {agent_name}

**Status:** {status}
**Started:** {started_at}
**Duration:** {duration}

### Input Artifacts
- {artifact}: {path}

### Output
{if complete}
**Artifact:** {output_path}
**Size:** {size}
**Confidence:** {confidence_score}
{/if}

{if in_progress}
**Current Task:** {current_task}
**Progress:** {progress}%
{/if}

{if failed}
**Error:** {error_message}
**Retries:** {retries}/{max_retries}
{/if}

### Logs
```
{relevant log entries for this agent}
```
```

## Error Cases

**Workflow not found:**
```markdown
## ❌ Workflow Not Found

Could not find workflow `{workflow_id}`.

### Available Workflows
| ID | Status | Created | Request |
|----|--------|---------|---------|
{list of available workflows}

Run `/workflow-status {valid_id}` to check status.
```

**No workflows exist:**
```markdown
## ℹ️ No Workflows Found

No workflows have been created yet.

To start a new workflow:
1. Create a request file in `PRPs/00-Requests/`
2. Run `/init-workflow PRPs/00-Requests/your-request.md`
```

## Example Output

```markdown
## 📊 Workflow Status

**Workflow ID:** `wf_20241125_143022`
**Status:** 🔵 in_progress
**Pattern:** concurrent-standard
**Started:** 2024-11-25 14:30:22
**Last Updated:** 2024-11-25 14:35:15 (2 minutes ago)

---

### Progress

[████████████░░░░░░░░] 60%

**Current Phase:** 4/6 - generation

### Agent Status

| Agent | Status | Progress | Tokens |
|-------|--------|----------|--------|
| orchestrator | 🔵 monitoring | - | 2,100 |
| requirements-analyst | 🟢 complete | 100% | 2,500 |
| research-codebase | 🟢 complete | 100% | 10,300 |
| research-docs | 🟢 complete | 100% | 8,800 |
| architecture | 🟢 complete | 100% | 6,000 |
| codegen-backend | 🔵 in_progress | 67% | 5,600 |
| test-generation | 🟡 queued | 0% | - |
| validation | 🟡 pending | 0% | - |

### Recent Events
```
[14:35:15] AGENT_PROGRESS | agent=codegen-backend | progress=0.67
[14:33:00] AGENT_STARTED | agent=codegen-backend
[14:32:55] PHASE_STARTED | phase=4 | name=generation
[14:32:50] AGENT_COMPLETED | agent=architecture | tokens=6000
[14:30:22] WORKFLOW_INITIALIZED | pattern=concurrent-standard
```

### Artifacts Generated
- ✅ requirements/requirements.json (5.2KB)
- ✅ research/codebase.json (12.1KB)
- ✅ research/docs.json (8.4KB)
- ✅ architecture/architecture.json (7.8KB)
- ⏳ code/backend/ (generating...)

### Metrics
- **Total Tokens:** 35,300
- **Duration:** 4m 53s
- **Agents Complete:** 5/9

### Commands
- Resume: `/resume-workflow wf_20241125_143022`
- View logs: `cat workflows/wf_20241125_143022/events.log`
```
