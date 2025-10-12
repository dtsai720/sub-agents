# Quality Control and Error Handling

## Sub-agent Result Validation Checklist

- ✅ Deliverable files exist and are readable
- ✅ File format complies with specifications
- ✅ Content completeness meets standards
- ✅ Technical decisions are reasonable
- ✅ No obvious omissions or errors

## Handling Strategies

### 1. When Quality is Insufficient
- Re-dispatch the same Sub-agent
- Add specific improvement requirements in prompt
- Provide previous output as reference
- Clearly specify items that need improvement

### 2. When Dependency Issues are Found
- Dispatch prerequisite Sub-agent to supplement missing information
- Update workflow to avoid future recurrence

### 3. When Technical Solution is Questionable
- Dispatch Architect Agent for review
- Provide specific concerns for analysis
- Decide whether to adjust solution based on review results

### 4. When Cross-Agent Inconsistency Occurs
- Identify root cause of inconsistency
- Dispatch relevant Agents for alignment
- If necessary, dispatch Architect Agent to arbitrate

## Error Handling Scenarios

### Scenario 1: Task Tool Invocation Failure

**Error Code:** AGENT_EXEC_FAIL

**Reason:** Sub-agent execution timeout or error

**Handling Steps:**
1. Log error information (Agent name, task description, error message)
2. Check if caused by overly long prompt or format error
3. If fixable: Adjust prompt and retry (maximum 2 times)
4. If unfixable: Explain issue to user, ask if requirements need adjustment

---

### Scenario 2: Sub-agent Reports Insufficient Quality

**Error Code:** QUALITY_CHECK_FAIL

**Reason:** Deliverable missing, format error, incomplete content

**Handling Steps:**
1. Analyze specific missing items (what essential content is missing?)
2. Compose improvement requirements (clearly specify content to supplement)
3. Re-invoke same Sub-agent (maximum 3 times)
4. If still not meeting standards after 3 attempts: Split task or lower requirements

---

### Scenario 3: User Changes Requirements Mid-way

**Error Code:** REQUIREMENT_CHANGED

**Handling Steps:**
1. Assess impact scope (which completed stages need redoing?)
2. Identify Sub-agents that need re-execution (Product Manager? Architect? Developer?)
3. Explain impact to user (time cost, completed work)
4. After confirmation, re-dispatch relevant Sub-agents

---

### Scenario 4: Technical Conflict Discovered

**Error Code:** TECHNICAL_CONFLICT

**Reason:** Frontend-backend API mismatch, database design conflict, technology choice contradiction

**Handling Steps:**
1. Identify conflict points (specifically which technical decisions conflict?)
2. Invoke Architect Agent for arbitration (provide both technical solutions)
3. Update relevant documents (DESIGN.md, OPENAPI.yaml)
4. Re-dispatch affected Sub-agents (use updated design)

---

### Scenario 5: Missing Required Sub-agent Definition

**Error Code:** AGENT_NOT_FOUND

**Reason:** Corresponding .md file missing in .claude/agents/ directory

**Handling Steps:**
1. Check for alternative Agent (e.g., missing Backend Developer-Go, but have Backend Developer-Python)
2. If alternative exists: Ask user if willing to use alternative Agent
3. If no alternative: Prompt user to create corresponding Agent definition file
4. Provide template to help user quickly create

---

## Iterative Improvement Mechanism

- Maximum 3 iterations allowed for improvement
- If still not meeting standards after 3 iterations, should reassess requirements or split tasks
- Each iteration must provide clear improvement direction

## Sub-agent Output Format Requirements

Each Sub-agent must complete tasks in a single execution and output in the following format:

```
## 📋 Task Completion Report
**Agent Identity:** [Agent Name]

**Completed Tasks:**
[Specific work completed, including key decisions and implementation details]

**Deliverable Documents:**
- Document 1: {file_path_or_summary}
- Document 2: {file_path_or_summary}

**Quality Self-check:**
✅ Completed Items:
- [Item 1]
- [Item 2]

⚠️ Points to Note:
- [Note 1] (if none, write "None")

**Technical Decisions:**
- [Important technical choices, architectural decisions, or design considerations]

**Recommended Next Steps:**
- Recommended Agent: [Agent Name]
- Reason: [Why this Agent is needed]
- Required Input: [What documents or information needed]
```

⚠️ **Important Reminder:**
- Sub-agents cannot wait or perform multi-round interactions
- Must complete all work in a single response
- If task is too complex, should split into multiple independent Sub-agent tasks
