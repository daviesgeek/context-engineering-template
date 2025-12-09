# Resume Workflow

Resume a paused or failed multi-agent workflow.

## Arguments
$ARGUMENTS

## Instructions

You are the **Workflow Recovery Coordinator**. Your job is to resume workflows that have been paused, failed, or interrupted.

### Step 1: Parse Arguments

Parse `$ARGUMENTS` for:
- `workflow_id` (required) - The workflow to resume
- `--from-agent <agent>` (optional) - Restart from specific agent
- `--skip-failed` (optional) - Skip failed agents and continue
- `--force` (optional) - Force restart even if workflow appears active

Examples:
```
/resume-workflow wf_20241125_143022
/resume-workflow wf_20241125_143022 --from-agent codegen-backend
/resume-workflow wf_20241125_143022 --skip-failed
```

### Step 2: Load Workflow State

Read workflow state:
```
workflows/{workflow_id}/state.json
```

If not found:
- List available workflows
- Ask user to specify valid ID
- Stop

### Step 3: Validate Resumption

Check if workflow can be resumed:

**Can Resume:**
- Status: `paused`, `awaiting_input`, `failed`
- Has at least one incomplete phase
- Not already actively running (unless `--force`)

**Cannot Resume:**
- Status: `completed` - Workflow already done
- Status: `cancelled` - Workflow was cancelled
- Status: `in_progress` (without `--force`) - Already running

### Step 4: Determine Resume Point

Based on state and arguments, determine where to resume:

**Default Behavior (no flags):**
1. Find the current phase
2. Identify incomplete/failed agents in that phase
3. Resume those agents

**With `--from-agent`:**
1. Find the specified agent
2. Reset that agent's status to `pending`
3. Re-run from that agent forward

**With `--skip-failed`:**
1. Mark failed agents as `skipped`
2. Continue with next phase if dependencies allow
3. Note: May result in incomplete feature

### Step 5: Handle Pending Human Input

If workflow status is `awaiting_input`:

Check for pending clarifications:
```
workflows/{workflow_id}/state.json → human_inputs[]
```

If clarifications exist:
```markdown
## 🔔 Human Input Required Before Resuming

**Workflow:** {workflow_id}

### Pending Questions

{for each pending question}
**{question_id}:** {question}
- Context: {context}
- Suggested Default: {suggested_default}
{/for}

### To Continue

1. Answer the questions above in your response
2. Run `/resume-workflow {workflow_id}` again

Or to use defaults:
`/resume-workflow {workflow_id} --use-defaults`
```

### Step 6: Prepare Resume

Before resuming:

1. **Update State:**
   ```json
   {
     "status": "in_progress",
     "updated_at": "{current_timestamp}"
   }
   ```

2. **Log Event:**
   ```
   [{timestamp}] WORKFLOW_RESUMED | from_status={previous_status} | resume_point={agent_or_phase}
   ```

3. **Reset Failed Agents (if applicable):**
   ```json
   {
     "agents": {
       "{failed_agent}": {
         "status": "pending",
         "retries": 0,
         "error": null
       }
     }
   }
   ```

### Step 7: Execute Resume

Based on resume point:

**Resume from Phase:**
1. Set `execution_plan.current_phase` to target phase
2. Invoke orchestrator to continue from that phase
3. Orchestrator handles agent invocations

**Resume from Agent:**
1. Identify required inputs for agent
2. Verify input artifacts exist
3. Invoke the specific agent
4. On completion, continue normal orchestration

**After Validation Failure:**
1. Check which components failed validation
2. Re-run only the affected codegen agents
3. Re-run validation after regeneration

### Step 8: Report Status

After initiating resume:

```markdown
## ▶️ Workflow Resumed

**Workflow ID:** `{workflow_id}`
**Previous Status:** {previous_status}
**Resume Point:** {resume_point}
**Current Status:** 🔵 in_progress

### Resume Details
{description of what's being resumed and why}

### Preserved Artifacts
{list of artifacts that will be reused}

### Agents to Run
{list of agents that will execute}

### Monitoring
- Status: `/workflow-status {workflow_id}`
- Logs: `cat workflows/{workflow_id}/events.log`

---
*Resumed at {timestamp}*
```

### Recovery Scenarios

#### Scenario 1: Agent Timeout

**Detection:** Agent status = `failed`, error contains "timeout"

**Recovery:**
1. Check if agent has retries remaining
2. If yes: Reset and retry with extended timeout
3. If no: Report failure, suggest manual intervention

```markdown
## ⏱️ Agent Timeout Recovery

Agent `{agent}` timed out after {duration}.

### Options
1. **Retry** (recommended): `/resume-workflow {id}` - Will retry with 1.5x timeout
2. **Skip**: `/resume-workflow {id} --skip-failed` - Continue without this agent
3. **Manual**: Review agent output at `workflows/{id}/artifacts/`

Retries remaining: {retries}/{max_retries}
```

#### Scenario 2: Validation Failure

**Detection:** Validation agent complete but success_criteria not met

**Recovery:**
1. Parse validation results to identify failing components
2. Map failing components to responsible codegen agents
3. Re-run only those agents
4. Re-run validation

```markdown
## ❌ Validation Failure Recovery

Validation found issues with {n} components.

### Failing Components
{for each failure}
- **{component}**: {failure_reason}
  - Responsible Agent: {agent}
  - Will Re-generate: {yes/no}
{/for}

### Recovery Plan
1. Re-running {list of agents}
2. Will re-validate after generation

Starting recovery...
```

#### Scenario 3: Context Overflow

**Detection:** Error type = `context_overflow`

**Recovery:**
1. Cannot auto-recover
2. Suggest breaking into smaller requests
3. Offer to continue with reduced scope

```markdown
## 📊 Context Overflow

Agent `{agent}` exceeded context window limit.

### Cause
{description of what caused overflow}

### Recommendations
1. **Split Request**: Break feature into smaller pieces
2. **Reduce Scope**: Remove non-essential requirements
3. **Use Hierarchical Pattern**: Switch to modular approach

Cannot auto-resume. Please simplify the request and create a new workflow.
```

#### Scenario 4: Missing Artifact

**Detection:** Required input artifact doesn't exist

**Recovery:**
1. Identify which agent should have produced the artifact
2. Check if that agent completed successfully
3. If yes: Re-run artifact generation
4. If no: Re-run the producer agent

```markdown
## 📁 Missing Artifact

Required artifact `{artifact}` not found.

### Expected Producer
Agent: {agent}
Status: {agent_status}

### Recovery
{if agent completed}
Re-generating artifact from cached state...
{else}
Re-running {agent} to produce artifact...
{/if}
```

### Error Handling

**Workflow Not Found:**
```markdown
## ❌ Cannot Resume

Workflow `{workflow_id}` not found.

### Available Workflows
{list of workflows}
```

**Already Complete:**
```markdown
## ✅ Already Complete

Workflow `{workflow_id}` has already completed successfully.

To view results: `/workflow-status {workflow_id}`
To start new workflow: `/init-workflow <request-file>`
```

**Already Running:**
```markdown
## ⚠️ Already Running

Workflow `{workflow_id}` is already in progress.

Current Status: {status}
Active Agents: {list}

To force restart: `/resume-workflow {workflow_id} --force`
To check status: `/workflow-status {workflow_id}`
```

**Unrecoverable Failure:**
```markdown
## 🛑 Unrecoverable Failure

Workflow `{workflow_id}` cannot be resumed due to critical error.

### Error Details
{error_message}

### Recommendations
1. Review error logs: `workflows/{id}/events.log`
2. Check artifacts for partial progress: `workflows/{id}/artifacts/`
3. Create new workflow with adjusted requirements

### Salvageable Artifacts
{list of completed artifacts that can be reused}
```

## Example Successful Resume

```markdown
## ▶️ Workflow Resumed

**Workflow ID:** `wf_20241125_143022`
**Previous Status:** failed (validation)
**Resume Point:** Phase 4 - Regenerating `codegen-backend`
**Current Status:** 🔵 in_progress

### Resume Details
Validation found 2 test failures in the auth service. Re-generating
the affected component with additional error handling.

### Preserved Artifacts
- ✅ requirements/requirements.json
- ✅ research/codebase.json
- ✅ research/docs.json
- ✅ architecture/architecture.json
- ✅ code/frontend/ (unchanged)
- ⚠️ code/backend/ (regenerating)
- ⚠️ tests/ (will regenerate)

### Agents to Run
1. codegen-backend (retry #2)
2. test-generation (will re-run)
3. validation (will re-run)
4. documentation (will re-run)

### Monitoring
- Status: `/workflow-status wf_20241125_143022`
- Logs: `cat workflows/wf_20241125_143022/events.log`

---
*Resumed at 2024-11-25T14:45:00Z*
```
