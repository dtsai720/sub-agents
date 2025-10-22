# Quality Control & Validation

## Quality Control Principles

For detailed error handling procedures, refer to:
→ `.claude/workflows/error-handling.md`

---

## Validation Checklist

### Artifact Validation

- Are artifacts present and complete
- Do file formats and content meet specifications
- Are technical decisions reasonable
- **Is test coverage adequate**

---

## Test Coverage Validation Rules

After Developer Agent completes development stage, Orchestrator must perform the following checks:

### 1. Execute Tests and Check Coverage

```bash
# Run complete test suite and generate coverage report
go test ./... -v -coverprofile=coverage.out -timeout 600s
go tool cover -func=coverage.out | tail -1
```

### 2. Read Implementation Plan to Confirm Coverage Targets

Read coverage targets for each layer from `docs/IMPLEMENTATION_PLAN_BACKEND_{GO|JAVA|PYTHON}.md`

For example:
- Repository > 80%
- Service > 80%
- Handler > 75%

### 3. Compare Actual Coverage with Targets

```
IF (actual coverage < target coverage):
  THEN:
    → Record modules that didn't meet target and the gap
    → Calculate number of additional tests needed
    → Re-invoke Developer Agent with explicit requirements:
      - "Current {module} coverage is {actual}%, below target {target}%"
      - "Please add test cases, focusing on: {uncovered functions/branches}"
      - "Goal: Increase coverage to above {target}%"
ELSE:
  → Coverage meets target, proceed to next stage (Code Review)
ENDIF
```

### 4. Coverage Improvement Retry Mechanism

- Maximum 2 retry attempts
- Re-verify coverage after each retry
- If still below target after 2 attempts → Report to user, ask whether to:
  * Option A: Continue adding tests (3rd attempt)
  * Option B: Adjust coverage target (needs to update Implementation Plan)
  * Option C: Proceed with Code Review first, mark coverage issue

### 5. Coverage Check Example Output

```
❌ Test Coverage Below Target

| Module     | Current Coverage | Target Coverage | Gap     | Status |
|-----------|-----------------|-----------------|---------|--------|
| Repository | 72.3%           | >80%            | -7.7%   | ❌     |
| Service    | 13.9%           | >80%            | -66.1%  | ❌     |
| Handler    | 55.8%           | >75%            | -19.2%  | ❌     |

**Decision:** Re-invoke Backend Developer Agent to add tests
```

---

## Common Error Codes

- **AGENT_EXEC_FAIL** - Task invocation failed
- **QUALITY_CHECK_FAIL** - Insufficient quality
- **REQUIREMENT_CHANGED** - Requirements changed
- **TECHNICAL_CONFLICT** - Technical conflict
- **AGENT_NOT_FOUND** - Missing Sub-agent
- **COVERAGE_INSUFFICIENT** - Insufficient test coverage

---

## Iteration Principles

Maximum 3 improvements, each with clear improvement direction

---

## Sub-agent Output Requirements

Each Sub-agent must complete tasks in a single execution and output in the following format:

```
## 📋 Task Completion Report
**Agent Identity:** [Agent Name]

**Completed Tasks:**
[Specific work completed, including key decisions and implementation details]

**Artifacts:**
- Document 1: {file path or content summary}
- Document 2: {file path or content summary}
- ...

**Quality Self-Check:**
✅ Completed Items:
- [Item 1]
- [Item 2]

⚠️ Points to Note:
- [Note 1]
- [Note 2]

**Technical Decisions:**
- [Important technical choices, architecture decisions, or design considerations]

**Suggested Next Steps:**
- Recommended Agent: [Agent Name]
- Reason: [Why this Agent is needed]
- Required Input: [What documents or information needed]
```

⚠️ Important Reminder:
- Sub-agents cannot wait or engage in multi-turn interactions
- Must complete all work in a single response
- If task is too complex, should split into multiple independent Sub-agent tasks

---

## DBA Agent Dispatch Decision

### SQL DBA vs NoSQL DBA Selection Mechanism

Orchestrator must invoke the correct DBA Agent based on database type.

**Step 1: Read CLOUD_ARCHITECTURE.md**
```
→ Read docs/CLOUD_ARCHITECTURE.md
→ Look for "Database Type:" marker or database service name
```

**Step 2: Identify Database Type**

**SQL Database Keywords:**
- PostgreSQL, MySQL, MariaDB
- AWS RDS, Aurora (PostgreSQL/MySQL)
- Azure SQL Database, Azure Database for PostgreSQL/MySQL
- Google Cloud SQL, Spanner

**NoSQL Database Keywords:**
- MongoDB, MongoDB Atlas
- AWS DynamoDB, DocumentDB
- Azure Cosmos DB (NoSQL API, MongoDB API)

**Step 3: Invoke Corresponding DBA Agent**

```
IF SQL database identified:
  THEN:
    → Read .claude/agents/sql-dba.md
    → Compose Task Prompt (runtime-core + sql-dba + task)
    → Invoke SQL DBA Agent
    → Expected output:
      - docs/SCHEMA.sql
      - docs/migrations/*.sql
      - docs/QUERY_OPTIMIZATION.md

ELSE IF NoSQL database identified:
  THEN:
    → Read .claude/agents/nosql-dba.md
    → Compose Task Prompt (runtime-core + nosql-dba + task)
    → Invoke NoSQL DBA Agent
    → Expected output:
      - docs/NOSQL_SCHEMA.md
      - docs/INDEX_STRATEGY.md
      - docs/DATA_MODEL.json

ELSE IF both SQL and NoSQL identified (Hybrid):
  THEN:
    → Invoke SQL DBA + NoSQL DBA in parallel
    → In Task Prompt, explicitly inform each of their responsible database portion
    → Expected output: Output files from both

ELSE (cannot identify):
  THEN:
    → Ask user:
      "What database type does this project use?
       - SQL database (PostgreSQL, MySQL, Azure SQL, etc.)
       - NoSQL database (MongoDB, DynamoDB, Cosmos DB, etc.)
       - Hybrid architecture (SQL + NoSQL)"
    → Invoke corresponding DBA Agent based on user response
ENDIF
```

**Step 4: Verify Output**

- SQL DBA → Check if SCHEMA.sql is executable, Migration scripts are complete
- NoSQL DBA → Check if Document Schema, Partition Key design is reasonable

**Common Error Handling:**
- If CLOUD_ARCHITECTURE.md doesn't mark Database Type → Infer from database service name
- If cannot infer → Ask user
- If invoked wrong DBA Agent → Identify from error message, re-invoke correct Agent

---

## Project Status Management

For detailed PROJECT_STATUS.md management rules, refer to:
→ `.claude/CLAUDE.md` `[Project Status Management]` section

**Core Principles:**
- Orchestrator maintains `docs/PROJECT_STATUS.md`
- Automatically update after each Sub-Agent execution
- Support interruption recovery
- Record current stage, completed stages, pending stages
- Provide clear next step instructions
