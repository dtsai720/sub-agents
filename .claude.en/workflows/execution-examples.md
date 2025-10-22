# Complete Execution Examples

## Scenario 1: User Wants to Develop an Online Ordering System

### 1. User Input
```
"I want to develop an online ordering system"
```

### 2. Orchestrator Analysis
- Type: Product Development Flow
- Requirement Completeness: Concept stage, needs refinement
- Decision: Start with Product Manager Agent

### 3. Orchestrator Execution
- Read .claude/agents/product-manager.md
- Compose prompt (role + user requirements + output requirements)
- Invoke Product Manager Agent via Task tool

### 4. Product Manager Agent Reports Back
- Deliverable: docs/PROD.md
- Suggestion: Next step - invoke UI/UX Designer Agent

### 5. Orchestrator Validates and Continues
- Validate PROD.md quality
- Read .claude/agents/ui-ux-designer.md
- Compose prompt (role + PROD.md + output requirements)
- Invoke UI/UX Designer Agent via Task tool

### 6. Continue Until Complete Delivery

---

## Scenario 2: PM Proposes Requirement, System Finds Similar Existing Feature

### 1. User Input (PM)
```
"We need a feature that can close issues and notify the team"
```

### 2. Orchestrator Analysis
- Type: Product Development Flow
- Decision: Start with Product Manager Agent

### 3. Orchestrator Invokes Product Manager Agent
- Input: User requirements
- Output: docs/PROD.md
- Content includes: Detailed requirements for closing issues + notifying team members

### 4. Orchestrator Executes Feature Duplication Check
- Read .claude/agents/backend-developer-go.md
- Compose prompt:
  * Role definition + feature duplication check task
  * Input: Feature description from PROD.md
  * Requirement: Search for similar features
- Invoke Backend Developer Agent via Task tool

### 5. Backend Developer Agent Reports Back
```
## Existing Feature Analysis Report

**Similar Features Found:**

1. close issue API (src/api/issues.go:45)
   - Similarity: 80% (High)
   - Difference: Missing notification functionality

2. notification service (src/services/notification.go:20)
   - Similarity: 50% (Medium)
   - Difference: Needs integration with issue workflow

**Recommendation:** Enhance existing close issue API
```

### 6. Orchestrator Confirms with User
```
"I found the following similar features in the system:

📍 close issue API - 80% similarity
Difference: Missing team notification functionality

💡 Recommendation: Enhance existing API (estimated 2-3 hours)
   vs Build new feature (estimated 1-2 days)

Please choose:
1. [ ] Enhance existing API (recommended)
2. [ ] Build new API
3. [ ] Use existing feature as-is
"
```

### 7. User Chooses
Option 1 (Enhance existing API)

### 8. Orchestrator Switches to Existing Project Enhancement Flow
- Skip UI/UX design (foundation already exists)
- Invoke Architect Agent:
  * Input: PROD.md + existing implementation analysis
  * Task: Design how to integrate notification service
  * Output: DESIGN.md
- Invoke Backend Developer Agent:
  * Input: DESIGN.md + existing code analysis
  * Task: Modify close issue API, integrate notification functionality
- Invoke QA Agent (regression testing)
- Delivery

### Key Improvements
- ✅ Automatically discovers duplicate features (PM may not know about existing close issue API)
- ✅ Provides transparent analysis and recommendations
- ✅ Saves development time (2-3 hours vs 1-2 days)
- ✅ Avoids technical debt and duplicate code
