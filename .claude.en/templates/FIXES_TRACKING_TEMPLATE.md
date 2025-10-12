# Bug Fixes Tracking

**Created**: {YYYY-MM-DD}
**Source**: {CODE_REVIEW_REPORT.md | FRONTEND_CODE_REVIEW_REPORT.md | User Request}
**Branch**: {branch_name}
**Priority**: {Critical | High | Medium | Low | Security}

---

## Classification

**Type**: {Critical/High - 影響用戶 | Medium/Low - 技術債 | Security - 安全漏洞}

**PM Tracking Required**: {YES | NO}
- **YES**: 修復後需調用 PM Agent 更新 PROD.md
- **NO**: 僅記錄到 CHANGELOG.md

**Rationale**:
{說明為何選擇此分類 - 參考 bug-fix-flow.md 分類標準}

---

## Issues to Fix

### Critical Priority (Must Fix Immediately)

1. **[{Backend/Frontend/Security}] {Issue Title}**
   - **Severity**: Critical
   - **Category**: {Security / Performance / Bug / Data Integrity}
   - **Impact**: {對用戶或系統的實際影響}
   - **Location**: `{file_path}:{line_number}`
   - **Estimated Effort**: {X hours}
   - **Assigned Agent**: {Backend Developer (Go/Java/Python) | Frontend Developer | etc.}
   - **Status**: ⏳ Pending | 🔄 In Progress | ✅ Completed
   - **Description**:
     ```
     {詳細問題描述}
     ```
   - **Recommended Fix**:
     ```
     {建議的修復方案或代碼示例}
     ```
   - **User Impact (產品語言)**:
     {用產品語言描述對用戶的影響 - PM Agent 事後記錄時使用}

---

### High Priority (Should Fix Soon)

2. **[{Backend/Frontend}] {Issue Title}**
   - **Severity**: High
   - **Category**: {Performance / Bug / UX}
   - **Impact**: {影響描述}
   - **Location**: `{file_path}:{line_number}`
   - **Estimated Effort**: {X hours}
   - **Assigned Agent**: {Agent name}
   - **Status**: ⏳ Pending
   - **Description**:
     ```
     {問題描述}
     ```
   - **Recommended Fix**:
     ```
     {修復方案}
     ```
   - **User Impact (產品語言)**:
     {對用戶的影響}

---

### Medium Priority (Plan to Fix)

3. **[{Component}] {Issue Title}**
   - **Severity**: Medium
   - **Category**: {Code Quality / Tech Debt / Optimization}
   - **Impact**: {影響描述}
   - **Location**: `{file_path}:{line_number}`
   - **Estimated Effort**: {X hours}
   - **Assigned Agent**: {Agent name}
   - **Status**: ⏳ Pending
   - **Description**:
     ```
     {問題描述}
     ```
   - **Recommended Fix**:
     ```
     {修復方案}
     ```

---

### Low Priority (Nice to Have)

4. **[{Component}] {Issue Title}**
   - **Severity**: Low
   - **Category**: {Documentation / Code Style / Minor Improvement}
   - **Impact**: {影響描述}
   - **Location**: `{file_path}:{line_number}`
   - **Estimated Effort**: {X hours}
   - **Assigned Agent**: {Agent name}
   - **Status**: ⏳ Pending
   - **Description**:
     ```
     {問題描述}
     ```
   - **Recommended Fix**:
     ```
     {修復方案}
     ```

---

## Progress Summary

- **Total Issues**: {count}
- **Critical**: {count} ({completed}/{total})
- **High**: {count} ({completed}/{total})
- **Medium**: {count} ({completed}/{total})
- **Low**: {count} ({completed}/{total})
- **Completed**: {count}
- **In Progress**: {count}
- **Pending**: {count}
- **Total Estimated Effort**: {XX hours}
- **Actual Effort**: {XX hours} (update as tasks complete)

**Progress**: [███████░░░] {percentage}%

---

## Testing Plan

### Backend Testing
- [ ] Run `make test` (target: maintain {current_coverage}% coverage)
- [ ] Run integration tests
- [ ] Manual testing of affected functionality
- [ ] {其他測試}

### Frontend Testing
- [ ] Run `npm test` (target: maintain {current_coverage}% coverage)
- [ ] Component testing for modified components
- [ ] Manual UI testing
- [ ] {其他測試}

### Cross-cutting Testing
- [ ] Accessibility testing ({if applicable})
- [ ] Performance testing ({if applicable})
- [ ] Security testing ({if applicable})
- [ ] E2E testing ({if critical flows affected})

---

## Post-Fix Actions

### If PM Tracking Required (YES):

**Step 1: Verify All Fixes Complete**
- [ ] All issues marked as ✅ Completed
- [ ] All tests passing
- [ ] Code review completed
- [ ] QA testing completed

**Step 2: Call PM Agent (事後記錄模式)**
- **Input to PM**:
  - This FIXES_TRACKING.md file
  - Implementation details from developers
  - Test results from QA
- **Expected Output**:
  - Updated PROD.md with "Recent Improvements" section
  - Technical issues converted to product language
  - User impact descriptions added

**Step 3: Final Documentation**
- [ ] PROD.md updated by PM Agent
- [ ] CHANGELOG.md updated with technical details
- [ ] Git commit created with clear message
- [ ] Pull Request created (via Git Manager Agent)

---

### If PM Tracking Not Required (NO):

**Step 1: Technical Documentation**
- [ ] Update CHANGELOG.md with fixes
- [ ] Update technical documentation if needed
- [ ] Add code comments where necessary

**Step 2: Git Management**
- [ ] Create git commit
- [ ] Create Pull Request
- [ ] Merge to main branch

---

## Completion Criteria

**All items must be checked before closing:**

- [ ] All issues marked as ✅ Completed
- [ ] All tests passing (backend + frontend)
- [ ] Code review completed by appropriate reviewer
- [ ] QA testing completed and approved
- [ ] PROD.md updated (if PM tracking = YES)
- [ ] CHANGELOG.md updated
- [ ] Git branch created and PR submitted
- [ ] PROJECT_STATUS.md updated

---

## Notes

### Blocked Items
{List any blocked items and reasons}

### Deferred Items
{List any items deferred to future sprints with rationale}

### Additional Context
{Any additional context, dependencies, or considerations}

---

## Reference Documents

- **Source Code Review**: {link to CODE_REVIEW_REPORT.md}
- **Frontend Review**: {link to FRONTEND_CODE_REVIEW_REPORT.md if applicable}
- **QA Test Plan**: {link to QA_TEST_IMPROVEMENT_PLAN.md if applicable}
- **Architecture Docs**: {link to CLOUD_ARCHITECTURE.md}

---

**Template Version**: 1.0
**Created**: {YYYY-MM-DD}
**Last Updated**: {YYYY-MM-DD HH:MM:SS}
