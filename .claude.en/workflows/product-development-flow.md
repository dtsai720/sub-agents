# Product Development Workflow

**Trigger Condition:** User describes product concept or idea

---

## Workflow

### Step 1: Product Requirements Analysis

**Invoke Agent:** Product Manager Agent

**Input:** User requirement description

**Output:** `docs/PROD.md`

**Content Includes:**
- Product overview and value proposition
- Target users and use cases
- Feature requirements list (prioritized)
- Non-functional requirements (performance, security)
- MVP scope definition
- **⭐ UI/UX requirement markers** (new)

---

### Step 2: Feature Duplication Check

**Invoke Agent:** Backend Developer Agent (analysis mode)

**Orchestrator Determines:**
- Check existing codebase language (Go/Java/Python)
- → Read `.claude/agents/backend-developer-{go|java|python}.md`
- If new project or cannot determine: Ask user about tech stack

**Task:** Search if similar features already exist in project

**Output:** Existing feature analysis report

**Decision Branch:**
```
IF (similar features found):
  THEN:
    → Confirm with user:
      - Option 1: Enhance existing feature → Switch to "Existing Project Enhancement Flow"
      - Option 2: Build new feature → Continue this flow
      - Option 3: Use as-is → Provide documentation, end
ELSE:
  → Continue development
ENDIF
```

Detailed check logic:
→ Read `.claude/workflows/feature-duplication-check.md`

---

### Step 3: UI/UX Design Decision ⭐ New Optional Logic

**Orchestrator Decision Logic:**

```
→ Read docs/PROD.md
→ Look for UI/UX requirement markers:
  - "UI Type: API Backend Only"
  - "UI Type: Bootstrap/Tailwind Simple UI"
  - "UI Type: Custom Design"
  - "Frontend: None"
  - "Frontend: React/Vue/Angular"

IF (UI Type = "API Backend Only" OR Frontend = "None"):
  THEN:
    → Skip UI/UX Designer Agent
    → Record decision: "This project is pure API Backend, using Swagger UI as developer interface"
    → Proceed to Step 4

ELSE IF (UI Type = "Bootstrap/Tailwind Simple UI"):
  THEN:
    → Skip UI/UX Designer Agent
    → Record decision: "Using ready-made UI Framework, no custom design needed"
    → Proceed to Step 4

ELSE IF (UI Type = "Custom Design" OR user provides Figma link):
  THEN:
    → Invoke UI/UX Designer Agent
    → Input: docs/PROD.md + Figma link (if any)
    → Output: docs/UI_UX_DESIGN.md (design specification)
    → Proceed to Step 4

ELSE (PROD.md not explicitly marked):
  THEN:
    → Ask user:
      "What are the UI requirements for this project?

      **Option A: Pure API Backend (Recommended for backend services)**
      - Use Swagger UI as developer interface
      - Frontend team develops independently based on OPENAPI.yaml
      - Suitable for: Microservices, Mobile Backend, B2B API

      **Option B: Use Ready-made UI Framework**
      - Bootstrap 5 / Tailwind CSS rapid interface
      - Focus on functionality rather than visual design
      - Suitable for: Internal tools, Admin Dashboard, MVP validation

      **Option C: Need Complete Visual Design**
      - Need to provide Figma designs or detailed design requirements
      - Invoke UI/UX Designer Agent to produce design specification
      - Suitable for: Consumer-facing products, brand-important projects

      Please choose A, B, or C, or directly describe your UI requirements."

    → Update PROD.md UI Type marker based on user answer
    → Execute corresponding branch logic
ENDIF
```

---

### Step 4: Cloud Architecture Design

**Invoke Agent:** Cloud Architect Agent

**Input:**
- `docs/PROD.md`
- `docs/UI_UX_DESIGN.md` (if exists)

**Output:**
- `docs/CLOUD_ARCHITECTURE.md`
- `docs/API_ENDPOINTS.md`
- `docs/ER_DIAGRAM.md`

**Important:** CLOUD_ARCHITECTURE.md must include explicit database type markers:
- Database Type: SQL (PostgreSQL/MySQL/Azure SQL, etc.)
- Database Type: NoSQL (MongoDB/DynamoDB/Cosmos DB, etc.)
- Database Type: Hybrid (SQL + NoSQL - need to invoke both DBA Agents separately)

---

### Step 5: API and Database Design (Can Run in Parallel)

#### 5.1 API Design

**Invoke Agent:** API Designer Agent

**Input:**
- `docs/CLOUD_ARCHITECTURE.md`
- `docs/API_ENDPOINTS.md`

**Output:** `docs/OPENAPI.yaml` (extends API_ENDPOINTS.md)

#### 5.2 Database Design

**Orchestrator Decides Based on Database Type in CLOUD_ARCHITECTURE.md**

**Decision Logic:**
```
→ Read docs/CLOUD_ARCHITECTURE.md
→ Look for "Database Type:" marker

IF Database Type = SQL:
  THEN:
    → Read .claude/agents/sql-dba.md
    → Invoke SQL DBA Agent
    → Expected output: SCHEMA.sql + Migration scripts

ELSE IF Database Type = NoSQL:
  THEN:
    → Read .claude/agents/nosql-dba.md
    → Invoke NoSQL DBA Agent
    → Expected output: NOSQL_SCHEMA.md + INDEX_STRATEGY.md + DATA_MODEL.json

ELSE IF Database Type = Hybrid:
  THEN:
    → Invoke SQL DBA + NoSQL DBA in parallel
    → SQL DBA handles relational database part
    → NoSQL DBA handles NoSQL database part

ELSE:
  THEN:
    → Ask user to choose database type
    → Invoke corresponding DBA Agent based on user answer
ENDIF
```

Detailed decision logic:
→ Read `.claude/core/quality-control.md` (DBA Agent dispatch decision)

---

### Step 5.5: Database Operations Design (If Production Environment)

**Trigger Conditions:**
- Schema design complete (SCHEMA.sql or NOSQL_SCHEMA.md produced)
- Environment is production (or user explicitly requests ops planning)
- User mentions: backup, HA, disaster recovery, security, monitoring keywords

**DB Ops Agent Invocation:**
```
→ Read .claude/agents/db-ops.md
→ Invoke DB Ops Agent
→ Input:
  - docs/CLOUD_ARCHITECTURE.md
  - docs/SCHEMA.sql or docs/NOSQL_SCHEMA.md
  - Business requirements (RPO, RTO, availability targets)
→ Expected output:
  - docs/BACKUP_STRATEGY.md (backup strategy)
  - docs/HA_DR_PLAN.md (high availability and disaster recovery)
  - docs/MONITORING_SETUP.md (monitoring and alerting)
  - docs/SECURITY_HARDENING.md (security hardening)
  - docs/DB_OPS_RUNBOOK.md (operations manual)
```

**Notes:**
- Development environment can skip this step
- If user doesn't mention ops requirements, Orchestrator should proactively ask:
  "Does this project need production environment database operations planning? (backup, HA, monitoring)"

---

### Step 5.6: Deliverable Completeness Check (Mandatory Before Development)

Orchestrator must check the following required files before entering development phase:

**Required Files List:**
```
✅ Must exist:
- docs/CLOUD_ARCHITECTURE.md (Architect output)
- docs/OPENAPI.yaml or docs/API_ENDPOINTS.md (API Designer output)
- docs/SCHEMA.sql (SQL DBA) or docs/NOSQL_SCHEMA.md (NoSQL DBA)

⚠️ Production environment must have:
- docs/BACKUP_STRATEGY.md (DB Ops output)
- docs/HA_DR_PLAN.md (DB Ops output)
```

**Check Logic:**
```
IF (missing OPENAPI.yaml AND missing API_ENDPOINTS.md):
  THEN:
    → Pause development flow
    → Invoke API Designer Agent to produce OPENAPI.yaml
    → Verify file produced
    → Continue development flow

IF (missing SCHEMA.sql AND missing NOSQL_SCHEMA.md):
  THEN:
    → Pause development flow
    → Invoke corresponding DBA Agent based on Database Type
    → Verify file produced
    → Continue development flow

IF (production environment AND missing DB Ops documents):
  THEN:
    → Ask user: "Need production environment operations planning?"
    → If yes → Invoke DB Ops Agent
ENDIF
```

---

### Step 6: Development Phase (Execute in Two Stages)

#### Stage 6.1: Implementation Plan Production

**Invoke Agent:** Backend Developer Agent (first time)

**Task:** Produce IMPLEMENTATION_PLAN

**Output:** `docs/IMPLEMENTATION_PLAN_{AGENT_NAME}.md`

**After Completion:** Developer Agent reports to Orchestrator, STOP execution

#### Stage 6.2: User Reviews Plan

**Orchestrator Actions:**
- Read IMPLEMENTATION_PLAN file
- Present to user for review (3-5 development stages, test plan, file list)
- Wait for user confirmation or modification

**Decision Point:**
- Approve → Proceed to Stage 6.3
- Modify → Orchestrator updates Plan, confirm again
- Reject → Adjust requirements, re-plan

#### Stage 6.3: Actual Development (Can Run in Parallel)

**Invoke Agent:** Backend Developer Agent (second time)

**Input:** Approved IMPLEMENTATION_PLAN

**Tasks:**
- Frontend Developer Agent executes IMPLEMENTATION_PLAN_FRONTEND.md (if needed)
- Backend Developer Agent executes IMPLEMENTATION_PLAN_BACKEND_{GO|JAVA|PYTHON}.md
- Developer Agent updates Stage Status in Plan
- Clean up IMPLEMENTATION_PLAN file after completion

---

### Step 6.5: API Change Sync Check (Auto Trigger)

**Orchestrator Actions:**
- Read `docs/CHANGE_SUMMARY.md` to detect API changes
- If API changes → Invoke API Designer Agent to sync OPENAPI.yaml
- If no API changes → Skip this step

Detailed rules:
→ Read `.claude/core/quality-control.md` (API change detection and sync rules)

---

### Step 7: Backend Code Review

**Invoke Agent:** Backend Code Reviewer Agent

**Input:** `docs/CHANGE_SUMMARY.md`

**Output:** `docs/CODE_REVIEW_REPORT.md`

**Decision:**
- If Critical Issues → Return to Developer Agent for fixes
- If passed → Continue next step

---

### Step 8: QA Testing

**Invoke Agent:** QA Agent

**Tasks:**
- Execute test suite
- Verify feature completeness

**Output:** `docs/QA_TEST_REPORT.md`

**Decision:**
- Failed → Return to Developer Agent for modifications
- Passed → Continue next step

---

### Step 9: DevOps Deployment Configuration (If Needed)

**Trigger Conditions (Any one met):**
- User explicitly requests production deployment
- Selected "Production environment" in initial requirements analysis
- QA tests passed and user asks "how to deploy"
- User mentions: Terraform, Kubernetes, ECS, CI/CD, deployment

**Invoke Agent:** DevOps Agent

**Output:**
- `terraform/` directory (IaC for AWS resources)
- `.github/workflows/deploy.yml` (CI/CD pipeline)
- `k8s/` directory (if using Kubernetes)
- `docs/DEPLOYMENT_GUIDE.md`
- `docs/INFRASTRUCTURE.md`

**Notes:**
- Development environment can skip this step (use docker-compose)
- If user doesn't mention deployment, Orchestrator should ask after QA passes:
  "QA tests passed! Need to deploy to production environment?
   - Option A: Yes (I will invoke DevOps Agent to produce Terraform + CI/CD)
   - Option B: No (local development/testing only)"

**⚠️ IaC Security Constraints (DevOps Agent Must Follow):**
- ✅ Terraform: Only allow `terraform plan` (validate configuration)
- ✅ Helm: Only allow `helm install --dry-run --debug` (simulate deployment)
- ✅ Kubernetes: Only allow `kubectl apply --dry-run=client` (client-side validation)
- ❌ Forbidden: Any actual deployment commands (`terraform apply`, `helm install`, `kubectl apply`)

Detailed security specifications:
→ Read `.claude/agents/devops.md` `[IaC Testing Security Constraints]` section

---

### Step 10: Document Review and Delivery

**Invoke Agent:** Document Review Agent (if implemented)

**Task:** Accept all deliverables

**Complete:** Project delivery

---

## UI/UX Decision Examples

### Example 1: Pure API Backend Project

**PROD.md Marker:**
```markdown
## UI/UX Requirements

**UI Type:** API Backend Only

**Frontend:** None (API only, will be consumed by mobile apps and web clients)

**Developer Interface:** Swagger UI (auto-generated from OPENAPI.yaml)

**Rationale:** This is a microservice providing authentication functionality.
Frontend teams will develop their own UI based on the OpenAPI specification.
```

**Orchestrator Behavior:**
- ✅ Skip UI/UX Designer Agent
- ✅ Focus on API quality and documentation completeness
- ✅ Ensure OPENAPI.yaml includes complete examples and descriptions

### Example 2: Bootstrap Quick MVP

**PROD.md Marker:**
```markdown
## UI/UX Requirements

**UI Type:** Bootstrap/Tailwind Simple UI

**Frontend:** React with Bootstrap 5

**Design Approach:** Use Bootstrap components for rapid MVP development

**Rationale:** This is an internal admin dashboard. Focus on functionality
over visual design. Use standard Bootstrap components to save time.
```

**Orchestrator Behavior:**
- ✅ Skip UI/UX Designer Agent
- ✅ Frontend Developer uses Bootstrap default styles
- ✅ Focus on feature implementation rather than visual design

### Example 3: Need Complete Design

**PROD.md Marker:**
```markdown
## UI/UX Requirements

**UI Type:** Custom Design

**Frontend:** React with custom design system

**Design Files:** https://www.figma.com/file/abc123...

**Rationale:** This is a customer-facing SaaS product. Brand consistency
and user experience are critical. Full UI/UX design required.
```

**Orchestrator Behavior:**
- ✅ Invoke UI/UX Designer Agent
- ✅ Produce design specification based on Figma designs
- ✅ Frontend Developer strictly follows design specification

---

## Workflow Visualization

```
Product Manager (PROD.md)
  ↓
Feature Duplication Check
  ↓
UI/UX Decision ⭐ New
  ├─ Option A: Skip (API Only)
  ├─ Option B: Skip (Bootstrap)
  └─ Option C: UI/UX Designer → UI_UX_DESIGN.md
  ↓
Cloud Architect (CLOUD_ARCHITECTURE.md)
  ↓
API Designer + DBA (parallel)
  ↓
DB Ops (if production environment)
  ↓
Deliverable Check
  ↓
Development (Plan → Review → Implementation)
  ↓
API Sync Check
  ↓
Backend Code Reviewer
  ↓
QA Testing
  ↓
DevOps (if production environment)
  ↓
Complete
```
