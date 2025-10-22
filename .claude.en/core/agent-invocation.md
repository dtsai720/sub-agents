# Sub-Agent Invocation Mechanism

## Dispatch Mechanism Overview

**⚠️ Important: For complete invocation examples and best practices, refer to:**
→ `.claude/ORCHESTRATOR_USAGE_TEMPLATE.md`

### Core Invocation Principles

1. Always **load** `templates/sub-agent-runtime-core.md` first (execution rules, not Sub-Agent)
2. Then **load** the specific Sub-Agent definition file (actual role definition)
3. Provide complete context (Sub-Agent cannot access conversation history)
4. Clear output requirements and artifact paths
5. The actual invocation uses `general-purpose` agent, executing with loaded content

---

## Dispatch Steps

### Step 1: Read Necessary Files

```
→ Read .claude/templates/sub-agent-runtime-core.md (execution rules document, must load)
→ Read .claude/agents/{AgentName}.md (Sub-Agent role definition)
Confirm Agent's role, capabilities, and output specifications
```

### Step 2: Compose Task Prompt

**Standard Prompt Structure:**

```
[=== Sub-Agent Runtime Core ===]
{Core constraints and execution rules loaded from templates/sub-agent-runtime-core.md}

[=== Agent Role Definition ===]
{Sub-Agent role definition loaded from agents/{AgentName}.md}

[=== Current Task ===]
{Specific task defined based on workflow stage}

[=== Input Data ===]
{Previous stage artifacts content or paths}
{User-provided requirements or data}
{All necessary context information}

[=== Output Requirements ===]
{Expected artifact format and content requirements}
{Quality standards and acceptance criteria}
{File output path (e.g., docs/PROD.md)}

[=== Additional Context ===] (Optional)
{Technical constraints, timeline requirements, or other limitations}
```

Note: Standard reporting format is already defined in templates/sub-agent-runtime-core.md, no need to repeat

### Step 3: Invoke Task Tool

**Actual Invocation Example:**

```javascript
Task(
  subagent_type: "general-purpose",
  description: "Invoke Product Manager Agent to analyze subscription system requirements",
  prompt: `
You are a professional Product Manager Agent.

## Role Definition
[Content read from .claude/agents/product-manager.md]

## Current Task
User wants to develop a subscription system with core features including:
- Users can subscribe to content of interest
- Receive content update notifications

Target users: B2C individual users
Expected scale: Medium (1000-10000 users)

## Output Requirements
Please produce a complete PROD.md including:
1. Product overview and value proposition
2. User stories and use cases
3. Feature requirements list (prioritized)
4. Non-functional requirements (performance, security)
5. MVP scope definition

File output: docs/PROD.md

## Reporting Format
Use standard Sub-agent reporting format
`
)
```

### Step 4: Handle Sub-agent Report

- Verify artifact completeness (file exists, content complete)
- Check if quality meets standards
- Analyze if technical decisions are reasonable
- Evaluate if suggested next steps are appropriate
- Decide next stage dispatch strategy (sequential or parallel)

---

## Visual Feedback Specification

Must provide clear visual prompts when invoking Sub-agents:

### Before Invocation (BEFORE)

```
## 🟢 About to Invoke: [Agent Name]

**📋 Task:** [Brief task description]
**📄 Expected Output:** [Expected artifact file path]
**⏱️ Estimated Time:** [Estimated execution time]

**Input Information:**
- [Key input information 1]
- [Key input information 2]
- ...
```

### After Invocation (AFTER)

```
## ✅ [Agent Name] Execution Complete

**📄 Produced:** [Actual artifact file path]
**⏱️ Execution Time:** [Actual execution time]
**📊 Artifact Summary:** [Brief summary of artifacts]

**Next Steps Suggested:**
- [Suggested next Agent or action]
```

---

## Sub-Agent Execution Failure Handling

### Detection Method

1. After Sub-agent reports completion, Orchestrator **must** verify artifacts exist
2. Use Read/Glob/Bash tools to check if files are actually created
3. If artifacts don't exist → determine as "execution failed"

### Failure Cause Analysis

- Sub-agent may have only "planned" without "executing"
- Sub-agent may have misunderstood task requirements
- Sub-agent may have encountered technical limitations but didn't report clearly

### Handling Process (MUST follow)

```
IF (Sub-agent reports completion && artifacts don't exist):
  THEN:
    1. Record failure reason and missing artifact list
    2. **Re-invoke the same Sub-agent**, and in Task Prompt:
       - Explicitly state "previous execution failed"
       - List "missing artifacts"
       - Emphasize "must actually use Write/Edit tools to create files"
       - Provide more detailed output requirements (file path, content structure)
    3. If second attempt still fails → report to user and pause execution

ELSE IF (artifacts exist but quality doesn't meet standards):
  THEN:
    1. Analyze quality issues (incomplete content, format errors, logic errors)
    2. Re-invoke Sub-agent with specific improvement suggestions
    3. Maximum 2 improvement attempts

ELSE:
  → Execution successful, proceed to next step
ENDIF
```

### Re-invocation Example

```markdown
## 🔄 Re-invoke: Backend Developer (Go) Agent

**Reason:** Previous execution failed - artifacts not produced

**Missing Artifacts:**
- ❌ internal/handler/auth_integration_test.go (integration test file)
- ❌ docs/CHANGE_SUMMARY.md (change summary document)
- ❌ Implementation Plan Status not updated

**Task Requirements (Enhanced):**
You **must** use Write tool to create the following files:
1. `/path/to/auth_integration_test.go` - containing 5 E2E tests
2. `/path/to/CHANGE_SUMMARY.md` - containing complete change summary
3. Use Edit tool to update IMPLEMENTATION_PLAN Status

**Verification Standards:**
- After completion, Orchestrator will use `ls -la /path/to/` to verify file existence
- Use `wc -l /path/to/file` to verify file is not empty
```

### Orchestrator Self-Reminder

- ⚠️ Never assume Sub-agent reporting "complete" means actually complete
- ✅ Must verify existence of each key artifact
- ✅ When failure detected, provide clearer instructions for re-invocation
- ✅ Record failure count to avoid infinite retry (maximum 2 attempts)

---

## Parallel Dispatch Strategy

### When to Use Parallel Dispatch

- When multiple Sub-agents' work has no dependencies
- Can significantly reduce overall execution time
- Example: Frontend development + Backend development + DBA can proceed simultaneously
- Example: Backend Code Reviewer + Document review can proceed simultaneously

### Parallel Dispatch Method

- Use multiple Task tool invocations in a single response
- Claude Code will automatically execute these Tasks in parallel
- After all results return, perform unified analysis

### Parallel Dispatch Example

```
Invoke three Sub-agents simultaneously:
- Task 1: Backend Developer Agent (Go) - Implement API
- Task 2: Frontend Developer Agent - Implement UI
- Task 3: DBA Agent - Design database

After all results return:
- Verify consistency of all three artifacts
- Check if interface definitions match
- Decide next step (usually integration testing)
```

### Considerations

- Ensure no data dependencies between parallel Sub-agents
- If dependencies exist, must invoke sequentially
- Parallel results need cross-validation for consistency

---

## Information Coordination and Transfer

When Architect needs to understand existing implementation:
1. Orchestrator first invokes corresponding Developer Agent to analyze existing code
2. Pass analysis results to Architect Agent
3. Architect designs based on analysis results

**Principles:**
- Sub-agents don't communicate directly, all information passed by Orchestrator
- Orchestrator responsible for identifying "what prerequisite information is needed" and dispatching accordingly

---

## Sub-Agent Constraints

⚠️ Sub-agents must comply with:
- Must complete all tasks in a single execution
- Cannot invoke other Sub-agents (only Orchestrator can dispatch)
- Cannot wait or interact, must complete in one go
- Must output results in standard format
- Ensure artifacts are complete and meet specifications

---

## Technical Limitations

- Sub-agents use Task tool's "general-purpose" type
- Sub-agents cannot access Orchestrator's conversation history
- All necessary information must be provided in the prompt
- Sub-agent responses are fully returned to Orchestrator
- Orchestrator responsible for passing information between Sub-agents
