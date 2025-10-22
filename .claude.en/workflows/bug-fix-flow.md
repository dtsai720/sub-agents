# Bug Fix Flow - Hybrid Approach Based on Impact Scope

**Version**: 1.0
**Created**: 2025-10-10
**Applicable Scenarios**: Bugs found after Code Review, technical debt, performance issues

---

## Flow Overview

When Code Review or Codebase Analysis discovers issues, Orchestrator chooses different handling flows based on **impact scope**:

```
Code Review Finds Issues → Orchestrator Classifies →
  ├─ Critical/High (Affects Users) → PM Tracking Flow
  ├─ Medium/Low (Technical Debt) → Orchestrator Quick Fix Flow
  └─ Security (Security Vulnerabilities) → Orchestrator Emergency Fix Flow
```

---

## Classification Standards

### Critical/High Priority (Affects Users) → PM Tracking

**Criteria:**
- ✅ Directly impacts user experience (UI bugs, feature errors, data errors)
- ✅ Causes service unavailability (timeout, crash, hang)
- ✅ Legal/compliance risks (accessibility, GDPR, privacy)
- ✅ Data integrity issues (data loss, inconsistency)

**Examples:**
- HTTP timeout causing users to wait without response
- UI state management errors causing display confusion
- Accessibility violations (WCAG non-compliance)
- Goroutine leak causing application slowdown

**Handling Flow:**
1. Orchestrator creates FIXES_TRACKING.md (temporary tracking)
2. Directly invoke Developer Agents for quick fixes
3. **After fixes**, invoke PM Agent to update PROD.md (post-recording)
4. PM converts technical issues to product improvement descriptions

---

### Medium/Low Priority (Technical Debt) → Orchestrator Tracking

**Criteria:**
- ✅ Code quality issues (code smell, duplicate code)
- ✅ Performance optimization (non-urgent)
- ✅ Test coverage improvement
- ✅ Documentation supplementation
- ✅ Dependency updates (no security risk)

**Examples:**
- Increase test coverage from 35% → 70%
- Refactor duplicate code
- Optimize algorithm efficiency (no user complaints)
- Supplement API documentation

**Handling Flow:**
1. Orchestrator creates FIXES_TRACKING.md
2. Directly invoke Developer Agents for fixes
3. **No need** for PM involvement, no PROD.md update
4. Record technical improvements in CHANGELOG.md

---

### Security Issues (Security Vulnerabilities) → Orchestrator Emergency Flow

**Criteria:**
- ✅ Security vulnerabilities (SQL injection, XSS, CSRF)
- ✅ Sensitive data leakage
- ✅ Permission control issues
- ✅ Dependency package security updates

**Examples:**
- SQL injection vulnerability
- API without permission validation
- Password stored in plaintext
- Dependencies with known CVEs

**Handling Flow:**
1. Orchestrator **immediately** creates SECURITY_FIX_TRACKING.md
2. Highest priority to invoke Developer Agents
3. **After fixes**, invoke PM Agent to assess user notification needs
4. Record fixes in SECURITY.md

---

## Orchestrator Decision Tree

```
[Code Review Finds Issues]
         ↓
[Read Issue List and Severity Assessment]
         ↓
    ┌────┴────┐
    │ Classify Issues │
    └────┬────┘
         ↓
    Does it affect users?
    ├─ YES → Critical/High Priority
    │         ├─ Create FIXES_TRACKING.md
    │         ├─ Invoke Developer Agents (quick fix)
    │         ├─ Invoke Code Reviewer (validation)
    │         ├─ Invoke QA (testing)
    │         └─ Invoke PM Agent (post-record to PROD.md)
    │
    ├─ NO → Is it a security vulnerability?
    │        ├─ YES → Security Issues
    │        │         ├─ Create SECURITY_FIX_TRACKING.md
    │        │         ├─ Highest priority fix
    │        │         └─ PM assesses user notification
    │        │
    │        └─ NO → Medium/Low Priority
    │                  ├─ Create FIXES_TRACKING.md
    │                  ├─ Invoke Developer Agents
    │                  └─ Record to CHANGELOG.md
    │
    └─ Complete
```

---

## FIXES_TRACKING.md Format

Orchestrator automatically creates this file to track fix progress:

```markdown
# Bug Fixes Tracking

**Created**: {YYYY-MM-DD}
**Source**: {CODE_REVIEW_REPORT.md | FRONTEND_CODE_REVIEW_REPORT.md}
**Branch**: {branch_name}
**Priority**: {Critical | High | Medium | Low | Security}

---

## Classification

**Type**: {Critical/High - Affects Users | Medium/Low - Technical Debt | Security - Security Vulnerability}

**PM Tracking Required**: {YES | NO}
- YES: Needs PROD.md update after fix
- NO: Only record to CHANGELOG.md

---

## Issues to Fix

### Critical Priority (Must Fix)

1. **[Backend] HTTP Client Timeout Missing**
   - **Severity**: High
   - **Impact**: Service may hang causing user wait
   - **Location**: `internal/api/winfittsclient.go:33`
   - **Estimated Effort**: 2 hours
   - **Assigned Agent**: Backend Developer (Go)
   - **Status**: ⏳ Pending
   - **Fix**:
     ```go
     client.Timeout = 30 * time.Second
     ```

2. **[Backend] Goroutine Leak in Config Watcher**
   - **Severity**: High
   - **Impact**: Resource leak, application slowdown
   - **Location**: `internal/config/config.go:85`
   - **Estimated Effort**: 2 hours
   - **Assigned Agent**: Backend Developer (Go)
   - **Status**: ⏳ Pending

### High Priority (Should Fix Soon)

3. **[Frontend] Mode State Management Bug**
   - **Severity**: High
   - **Impact**: UI may display incorrect state
   - **Location**: `frontend/src/pages/HomePage.jsx`
   - **Estimated Effort**: 5 hours
   - **Assigned Agent**: Frontend Developer
   - **Status**: ⏳ Pending

4. **[Frontend] Accessibility Violations**
   - **Severity**: Critical
   - **Impact**: Cannot meet WCAG 2.1 Level AA, legal risk
   - **Location**: Global (missing ARIA labels)
   - **Estimated Effort**: 8 hours
   - **Assigned Agent**: Frontend Developer
   - **Status**: ⏳ Pending

---

## Progress Summary

- **Total Issues**: 4
- **Completed**: 0
- **In Progress**: 0
- **Pending**: 4
- **Total Estimated Effort**: 17 hours

---

## Testing Plan

- [ ] Backend: Run `make test` (target: maintain 89.2% coverage)
- [ ] Frontend: Run `npm test` (target: maintain 35% coverage, don't break existing)
- [ ] Integration: Manual testing of sync flow
- [ ] Accessibility: axe DevTools scan

---

## Post-Fix Actions

### If PM Tracking Required (YES):
1. Call PM Agent with context:
   - Input: FIXES_TRACKING.md + implementation details
   - Output: Updated PROD.md section "Recent Improvements"
   - PM converts technical fixes to product language

### If PM Tracking Not Required (NO):
1. Update CHANGELOG.md with technical details
2. No PROD.md update needed

---

## Completion Criteria

- [ ] All issues marked as ✅ Completed
- [ ] All tests passing
- [ ] Code review completed
- [ ] PROD.md updated (if PM tracking required)
- [ ] CHANGELOG.md updated
- [ ] Git branch merged to main
```

---

## PM Agent Post-Recording Mode

When `PM Tracking Required = YES`, Orchestrator invokes PM Agent:

### Input (Prompt)
```markdown
## Task: Convert Technical Fixes to Product Improvement Records

You are a Product Manager who needs to convert the following technical bug fixes to product language and update PROD.md.

### Input Data
- FIXES_TRACKING.md (technical issue list)
- Implementation details (fix content)

### Task
1. Read FIXES_TRACKING.md to understand fix content
2. Read existing PROD.md
3. Add "Recent Improvements" section to PROD.md
4. Describe improvements in product language (no technical jargon)

### Output Format
Updated PROD.md with new section:

## Recent Improvements

**Version**: {version}
**Release Date**: {date}

### Stability Enhancements
- ✅ Improved application stability, resolved performance degradation after long runtime
- ✅ Enhanced network connection timeout handling, reduced user wait time

### Accessibility Improvements
- ✅ Added complete keyboard navigation support, meets accessibility standards
- ✅ Improved screen reader compatibility (WCAG 2.1 Level AA)

### UI/UX Refinements
- ✅ Fixed state display issues, improved interface consistency
```

### PM Agent Conversion Examples

**Technical Description** → **Product Description**

| Technical Issue | Product Improvement Description |
|---------|-------------|
| HTTP client timeout missing | Improved network connection handling, avoid long user wait |
| Goroutine leak in config watcher | Enhanced application stability, resolved long-runtime performance issues |
| Mode state boolean bug | Fixed interface state display issue, improved user experience consistency |
| WCAG accessibility violations | Added complete accessibility support, meets international standards (WCAG 2.1 Level AA)|

---

## Complete Flow Example

### Scenario: Code Review Finds 4 Issues

#### Step 1: Orchestrator Classification

```
Read CODE_REVIEW_REPORT.md + FRONTEND_CODE_REVIEW_REPORT.md

Classification Results:
- HTTP timeout → Critical (affects users)
- Goroutine leak → Critical (affects users)
- Mode state bug → High (affects users)
- Accessibility → Critical (legal risk + affects users)

Decision: All Critical/High → Use PM tracking flow
```

#### Step 2: Orchestrator Creates FIXES_TRACKING.md

```
PM Tracking Required: YES
Total Issues: 4
Estimated Effort: 17 hours
```

#### Step 3: Orchestrator Invokes Developer Agents

```
Invoke in parallel:
1. Backend Developer (Go) → Fix HTTP timeout + Goroutine leak
2. Frontend Developer → Fix Mode state + Accessibility

Waiting for completion...
```

#### Step 4: Orchestrator Invokes Code Reviewer

```
Backend Code Reviewer → Validate backend fixes
Frontend Code Reviewer → Validate frontend fixes
```

#### Step 5: Orchestrator Invokes QA

```
QA Agent → Execute test plan
- Backend tests maintain 89.2% coverage
- Frontend tests don't break existing functionality
- Manual accessibility testing
```

#### Step 6: Orchestrator Invokes PM Agent (Post-Recording)

```
Input: FIXES_TRACKING.md + implementation details
Output: Update PROD.md - "Recent Improvements" section

PM converts to product language:
- "Enhanced application stability"
- "Improved network connection handling"
- "Added complete accessibility support"
- "Fixed interface state display issues"
```

#### Step 7: Git Manager Commits Changes

```
Git Manager → Create commit + PR
Commit message: "fix: improve stability and accessibility (4 critical fixes)"
```

---

## Orchestrator Auto-Identifies Trigger Conditions

Orchestrator automatically starts Bug Fix Flow under following conditions:

1. **User Explicit Statement:**
   - "Please fix issues found in Code Review"
   - "Handle bugs in CODE_REVIEW_REPORT.md"
   - "Fix this technical debt"

2. **File Existence Trigger:**
   - Detects `CODE_REVIEW_REPORT.md` exists
   - Detects `FRONTEND_CODE_REVIEW_REPORT.md` exists
   - Detects `FIXES_TRACKING.md` exists (resume interrupted fix)

3. **Keyword Trigger:**
   - "bug fix", "fix", "fix issues"
   - "technical debt", "tech debt"
   - "code review findings", "review findings"

---

## Integration with Other Flows

### Relationship with Product Development Flow

- **Product Development**: New feature development (starts from PROD.md)
- **Bug Fix Flow**: Existing feature improvement (starts from Code Review)

**Differences:**
- Product Development: PM → Architect → API Designer → Developer
- Bug Fix Flow: Developer → Code Reviewer → QA → PM (post-recording)

### Relationship with Codebase Analysis Flow

- **Codebase Analysis**: Produces Code Review Reports
- **Bug Fix Flow**: Fixes issues found in Code Review

**Flow Connection:**
```
Codebase Analysis → Produce Reports →
Orchestrator Identifies Issues → Start Bug Fix Flow →
Fixes Complete → PM Records Improvements
```

---

## File Output Summary

| Priority | Tracking File | PM Record | Technical Record |
|-------|---------|---------|---------|
| Critical/High | FIXES_TRACKING.md | PROD.md (post) | CHANGELOG.md |
| Medium/Low | FIXES_TRACKING.md | Not needed | CHANGELOG.md |
| Security | SECURITY_FIX_TRACKING.md | PROD.md (case by case) | SECURITY.md |

---

## Flow Checklist

Orchestrator's checklist when executing Bug Fix Flow:

### Startup Phase
- [ ] Read Code Review Reports
- [ ] Classify issues (Critical/Medium/Security)
- [ ] Determine if PM tracking needed
- [ ] Create FIXES_TRACKING.md

### Execution Phase
- [ ] Invoke Backend Developer (if backend issues)
- [ ] Invoke Frontend Developer (if frontend issues)
- [ ] Invoke Code Reviewer (validate fixes)
- [ ] Invoke QA (testing)

### Completion Phase
- [ ] If PM tracking needed → Invoke PM Agent to update PROD.md
- [ ] If PM tracking not needed → Update CHANGELOG.md
- [ ] Invoke Git Manager (commit + PR)
- [ ] Update PROJECT_STATUS.md

---

## Resuming Interrupted Fixes

If Bug Fix process is interrupted, when resuming:

```
User says: "Continue from last interruption"

Orchestrator actions:
1. Read FIXES_TRACKING.md
2. Check Status of each issue
3. Find items with Status = "In Progress" or "Pending"
4. Continue invoking corresponding Agents
5. Update Status after completion
```

---

## Success Criteria

Bug Fix Flow completion criteria:

1. ✅ All issues Status = "✅ Completed"
2. ✅ All tests passing
3. ✅ Code Review passed
4. ✅ PROD.md updated (if needed)
5. ✅ Git PR created
6. ✅ PROJECT_STATUS.md updated

---

**Flow Version**: 1.0
**Created**: 2025-10-10
**Last Updated**: 2025-10-10
