# Feature Duplication Check Mechanism

When entering Product Development Flow, Orchestrator must execute feature duplication check to avoid redundant development.

## Trigger Timing

- After Product Manager Agent produces PROD.md
- Before entering UI/UX design phase
- Purpose: Confirm whether similar features already exist in the system

## Execution Steps

### Step 1: Invoke Backend Developer Agent for Feature Search

```javascript
Task(
  subagent_type: "general-purpose",
  description: "Backend Developer Agent executes feature duplication check",
  prompt: `
You are a professional Backend Developer Agent.

## Special Task: Feature Duplication Check

Please analyze whether similar features already exist in the project.

## Input
Core feature description from PROD.md:
- Feature: Close issue and notify team members
- User Story: When user closes issue, system should automatically notify relevant team members

Project path: {project_path}

## Analysis Requirements
1. Search project's API endpoints, services, modules
2. Compare based on functional semantics (not just names)
3. Assess similarity (High > 70% / Medium 40-70% / Low < 40%)
4. Explain differences

## Output Format

### Existing Feature Analysis Report

**Search Keywords:** close issue, notify, team

**Similar Features Found:**

1. **Feature Name:** close issue API
   - **Path:** src/api/issues.go:45
   - **Description:** Close issue and update database status
   - **Similarity:** High (80%)
   - **Differences:**
     * Existing: Close issue + update status
     * Missing: Team notification functionality
   - **Tech Stack:** Go + Gin

2. **Feature Name:** notification service
   - **Path:** src/services/notification.go:20
   - **Description:** Send various types of notifications
   - **Similarity:** Medium (50%)
   - **Differences:**
     * Existing: General notification service
     * Missing: Integration with issue closing

**Recommendation:**
- **Recommended Solution:** Enhance existing close issue API, integrate notification service
- **Reason:** Features highly overlap (80%), avoid redundant development
- **Estimated Effort:** Modify existing API (2-3 hours) vs Build from scratch (1-2 days)

**If No Similar Features Found:**

### Existing Feature Analysis Report

**Search Keywords:** [Extracted from PROD.md]

**Search Result:** No similar features found

**Confirmation:** Safe to proceed with new feature development, no duplication risk
`
)
```

### Step 2: Orchestrator Analyzes Results and Makes Decision

```
If similar features found (similarity > 40%):
  → Show analysis results to user
  → Provide choices: Enhance existing / Build new / Use as-is

If no similar features found:
  → Confirm no duplication
  → Continue product development flow
```

### Step 3: Confirm with User (if similar features found)

```
Orchestrator response:
"I found the following similar features in the system:

📍 **close issue API** (src/api/issues.go:45)
- Function: Close issue and update database status
- Similarity: 80% (highly similar)
- Difference: Currently no team notification functionality

📍 **notification service** (src/services/notification.go:20)
- Function: Send various types of notifications
- Similarity: 50% (moderately similar)
- Difference: Needs integration into issue closing workflow

💡 **Recommended Solution:** Enhance existing close issue API, integrate notification service

**Estimated Effort:**
- Enhance existing: 2-3 hours
- Build from scratch: 1-2 days

**Please Choose:**
1. [ ] Enhance existing close issue API (recommended)
   → Will switch to "Existing Project Enhancement Flow"

2. [ ] Build entirely new API
   → Continue "Product Development Flow"
   → Architect will ensure no conflict with existing features

3. [ ] Use existing feature as-is
   → I will provide documentation for existing API
"
```

### Step 4: Execute Flow Based on User Choice

```
Option 1 (Enhance existing):
  → Switch to "Existing Project Enhancement Flow"
  → Skip UI/UX and architecture design (foundation exists)
  → Proceed directly to development phase

Option 2 (Build new feature):
  → Continue "Product Development Flow"
  → Pass "Existing Feature Analysis Report" to Architect
  → Architect ensures no conflict when designing

Option 3 (Use as-is):
  → Produce documentation for existing API usage
  → End flow
```

## Key Principles

- ✅ Automated detection reduces manual oversight
- ✅ Transparent analysis results help user understand recommendations
- ✅ User retains final decision authority
- ✅ Provide effort estimates to aid decision-making
- ✅ Avoid technical debt and duplicate code
