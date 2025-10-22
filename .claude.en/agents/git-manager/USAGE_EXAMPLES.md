# Git Manager Agent - Usage Examples

本文檔提供 Git Manager Agent 的實際使用範例。

---

## 場景 1：創建 Pull Request（最常見）

### 背景
- Backend Developer 完成了用戶認證功能
- Backend Code Reviewer 審查通過
- 現在需要創建 PR 提交審查

### Orchestrator 調用

```markdown
我將調用 Git Manager 創建 Pull Request。

**Context:**
- Current branch: feature/user-authentication
- Base branch: main
- Changes completed:
  - POST /auth/register endpoint (email validation, bcrypt hashing)
  - POST /auth/login endpoint (JWT generation)
  - POST /auth/logout endpoint (JWT blacklist)
  - Unit tests: 26 tests passing
  - Integration tests: 8 tests passing
- Code review: ✅ Passed (CODE_REVIEW_REPORT.md available)

**Task:**
Please create a pull request with:
1. Appropriate title following [type] format
2. Comprehensive PR description including:
   - Summary of changes
   - Test results
   - Breaking changes (if any)
3. Add labels: "enhancement", "backend"
4. Assign to @me
5. Link to issue #123 (if exists)
```

### Git Manager 執行流程

**Step 1: 環境檢查**
```bash
$ git status
On branch feature/user-authentication
nothing to commit, working tree clean

$ git branch --show-current
feature/user-authentication

$ git remote -v
origin  https://github.com/user/repo.git (fetch)
origin  https://github.com/user/repo.git (push)
```

**Step 2: 分析變更**
```bash
$ git log origin/main...HEAD --oneline
a1b2c3d [test] Add integration tests for auth endpoints
b2c3d4e [feat] Add logout endpoint with JWT blacklist
c3d4e5f [feat] Add login endpoint with JWT generation
d4e5f6g [feat] Add register endpoint with email validation
e5f6g7h [refactor] Extract password hashing to utility

$ git diff origin/main...HEAD --stat
internal/handler/auth.go           | 180 +++++++++++++++++++
internal/handler/auth_test.go      |  95 ++++++++++
internal/service/auth.go           | 220 +++++++++++++++++++++++
internal/service/auth_test.go      | 130 ++++++++++++++
internal/middleware/jwt.go         |  85 +++++++++
pkg/utils/password.go              |  45 +++++
cmd/api/main.go                    |  12 +-
go.mod                             |   3 +
go.sum                             |   8 +
9 files changed, 775 insertions(+), 3 deletions(-)
```

**Step 3: Push 分支**
```bash
$ git push -u origin feature/user-authentication
Total 15 (delta 8), reused 0 (delta 0), pack-reused 0
To https://github.com/user/repo.git
 * [new branch]      feature/user-authentication -> feature/user-authentication
Branch 'feature/user-authentication' set up to track remote branch 'feature/user-authentication' from 'origin'.
```

**Step 4: 創建 PR**
```bash
$ gh pr create \
  --title "[feat] Add user authentication API endpoints" \
  --body "$(cat <<'EOF'
## Summary
Implement complete user authentication flow with JWT-based token management.

## Changes
- **POST /auth/register** - User registration with email validation and bcrypt password hashing
- **POST /auth/login** - User login with JWT token generation
- **POST /auth/logout** - JWT token invalidation using Redis blacklist
- **JWT Middleware** - Token validation for protected routes
- **Password Utilities** - Secure password hashing and validation

## Related Issues
- Closes #123

## Test Results
✅ **Unit Tests:** 26/26 passing
✅ **Integration Tests:** 8/8 passing
✅ **Code Coverage:** 87%

### Test Details
- Email validation (valid/invalid formats)
- Password strength validation
- JWT token generation and validation
- Token expiry handling
- Blacklist functionality
- Concurrent request handling

## Breaking Changes
None - This is a new feature addition.

## API Documentation
Updated in `docs/API.md`:
- POST /auth/register
- POST /auth/login
- POST /auth/logout

## Code Review
✅ Passed backend code review (see CODE_REVIEW_REPORT.md)
- No critical issues
- 2 minor suggestions implemented
- All best practices followed

## Checklist
- [x] Code compiles successfully
- [x] All tests pass (34/34)
- [x] No linting errors
- [x] API documentation updated
- [x] Backward compatible
- [x] Code reviewed

EOF
)" \
  --base main \
  --head feature/user-authentication \
  --label "enhancement" \
  --label "backend" \
  --assignee "@me"

✓ Created pull request #456 for user:feature/user-authentication into main
```

### Git Manager 回報

```
✅ Pull Request 創建成功！

**PR Details:**
- PR #456: https://github.com/user/repo/pull/456
- Title: [feat] Add user authentication API endpoints
- Base: main ← Head: feature/user-authentication
- Labels: enhancement, backend
- Assignee: @me

**Change Summary:**
- 9 files changed, 775 insertions(+), 3 deletions(-)
- 3 new API endpoints (register, login, logout)
- JWT middleware for protected routes
- 34 tests (all passing)
- 87% code coverage

**Next Steps:**
1. ⏳ Wait for CI/CD checks to complete
2. 👥 Request code review (suggested reviewers: @tech-lead, @security-team)
3. 📝 Address review feedback (if any)
4. ✅ Merge after approval

**CI/CD Status:**
- Build: Pending
- Tests: Pending
- Security Scan: Pending

Monitor PR status: https://github.com/user/repo/pull/456/checks
```

---

## 場景 2：Commit 變更

### 背景
- Backend Developer 修復了一個 bug
- 需要 commit 變更並推送

### Orchestrator 調用

```markdown
我將調用 Git Manager commit 變更。

**Context:**
- Modified files:
  - internal/service/auth.go (password validation bug fix)
  - internal/service/auth_test.go (added edge case tests)
- Change type: Bug fix
- All tests passing: ✅

**Task:**
Please:
1. Analyze the changes
2. Write appropriate commit message following project conventions
3. Commit the changes
4. Report commit SHA
```

### Git Manager 執行

```bash
# Step 1: 分析變更
$ git status
On branch feature/password-fix
Changes not staged for commit:
  modified:   internal/service/auth.go
  modified:   internal/service/auth_test.go

$ git diff internal/service/auth.go
@@ -45,7 +45,7 @@ func (s *AuthService) ValidatePassword(password string) error {
-    if len(password) < 8 {
+    if len([]rune(password)) < 8 {
         return errors.New("password must be at least 8 characters")
     }

# Step 2: 查看 commit 風格
$ git log -5 --pretty=format:"%s"
[test] Add integration tests for login endpoint
[feat] Add JWT token refresh endpoint
[refactor] Extract validation logic to separate package
[fix] Handle edge case in email validation
[docs] Update API documentation

# Step 3: Commit
$ git add internal/service/auth.go internal/service/auth_test.go

$ git commit -m "$(cat <<'EOF'
[fix] Fix password validation for multi-byte characters

Update password length validation to correctly count Unicode characters:
- Use len([]rune(password)) instead of len(password)
- Fixes issue where emoji/CJK characters counted as multiple bytes
- Add test cases for Unicode password validation

Resolves edge case where passwords with emoji were incorrectly rejected.

EOF
)"

[feature/password-fix a1b2c3d] [fix] Fix password validation for multi-byte characters
 2 files changed, 15 insertions(+), 5 deletions(-)
```

### Git Manager 回報

```
✅ Commit 創建成功！

**Commit Details:**
- SHA: a1b2c3d4e5f6g7h8i9j0
- Message: [fix] Fix password validation for multi-byte characters
- Branch: feature/password-fix
- Files: 2 changed (internal/service/auth.go, internal/service/auth_test.go)

**Changes:**
- 15 insertions(+)
- 5 deletions(-)

**Next Actions:**
1. Continue development, or
2. Push to remote: `git push origin feature/password-fix`
3. Create PR when ready

**Suggested PR Title:**
[fix] Fix password validation for multi-byte characters
```

---

## 場景 3：創建新分支並開始開發

### Orchestrator 調用

```markdown
我將調用 Git Manager 創建新的 feature 分支。

**Context:**
- Current branch: main
- New feature: Email verification
- Branch naming convention: feature/[feature-name]

**Task:**
1. Pull latest main
2. Create branch: feature/email-verification
3. Switch to new branch
4. Report status
```

### Git Manager 執行

```bash
# Step 1: 確保在 main 並更新
$ git checkout main
Switched to branch 'main'

$ git pull origin main
Already up to date.

# Step 2: 創建新分支
$ git checkout -b feature/email-verification
Switched to a new branch 'feature/email-verification'

# Step 3: 確認
$ git status
On branch feature/email-verification
nothing to commit, working tree clean

$ git log -1 --oneline
e5f6g7h [feat] Add JWT token refresh endpoint
```

### Git Manager 回報

```
✅ 新分支創建成功！

**Branch Details:**
- New branch: feature/email-verification
- Based on: main (commit e5f6g7h)
- Current HEAD: e5f6g7h [feat] Add JWT token refresh endpoint

**Status:**
- Working tree: clean
- Ready for development

**Next Steps:**
1. Start implementing email verification feature
2. Make commits as you develop
3. Push branch when ready: `git push -u origin feature/email-verification`
4. Create PR when feature is complete

**Suggested Commit Format:**
[feat] Add email verification endpoint
[test] Add tests for email verification
[docs] Update API docs for email verification
```

---

## 場景 4：處理 Merge Conflict（通知用戶）

### Orchestrator 調用

```markdown
我將調用 Git Manager 嘗試 rebase main 分支。

**Context:**
- Current branch: feature/user-profile
- Base branch: main
- Need to sync with latest main changes

**Task:**
Rebase current branch on latest main
```

### Git Manager 執行

```bash
# Step 1: Fetch latest
$ git fetch origin main

# Step 2: Attempt rebase
$ git rebase origin/main
Auto-merging internal/service/user.go
CONFLICT (content): Merge conflict in internal/service/user.go
Automatic merge failed; fix conflicts and then commit the result.
```

### Git Manager 回報（錯誤處理）

```
⚠️ Merge Conflict 發生

**狀況:**
Rebase 過程中發現 merge conflict，無法自動解決。

**衝突文件:**
- internal/service/user.go

**詳細資訊:**
```
<<<<<<< HEAD (Current Change: feature/user-profile)
func (s *UserService) UpdateProfile(userID int64, data UpdateProfileRequest) error {
    // Your implementation
=======
func (s *UserService) UpdateUserProfile(userID int64, profile ProfileData) error {
    // Main branch implementation
>>>>>>> origin/main
```

**解決步驟:**
1. **手動編輯衝突文件:**
   ```bash
   # 編輯 internal/service/user.go
   # 移除 <<<<<<, =======, >>>>>> 標記
   # 保留需要的代碼
   ```

2. **標記為已解決:**
   ```bash
   git add internal/service/user.go
   ```

3. **繼續 rebase:**
   ```bash
   git rebase --continue
   ```

**或者放棄 rebase:**
```bash
git rebase --abort
```

**需要協助嗎?**
- 使用 IDE 的 merge 工具（推薦）
- 或者手動編輯文件解決衝突

**當前狀態:**
Rebase 已暫停，等待衝突解決。
```

---

## 場景 5：檢查 Git 歷史和分支狀態

### Orchestrator 調用

```markdown
我將調用 Git Manager 檢查專案的 git 狀態。

**Task:**
1. 列出所有本地分支
2. 顯示最近 10 筆 commit
3. 檢查是否有未提交的變更
```

### Git Manager 執行

```bash
# Local branches
$ git branch
  develop
  feature/email-verification
* feature/user-authentication
  main

# Recent commits
$ git log -10 --pretty=format:"%h - %s (%an, %ar)"
a1b2c3d - [test] Add integration tests for auth (Alice, 2 hours ago)
b2c3d4e - [feat] Add logout endpoint (Alice, 3 hours ago)
c3d4e5f - [feat] Add login endpoint (Bob, 5 hours ago)
d4e5f6g - [feat] Add register endpoint (Bob, 6 hours ago)
e5f6g7h - [refactor] Extract JWT logic (Charlie, 1 day ago)
f6g7h8i - [test] Add unit tests for password (Alice, 2 days ago)
g7h8i9j - [fix] Fix email validation regex (Bob, 2 days ago)
h8i9j0k - [docs] Update API documentation (Charlie, 3 days ago)
i9j0k1l - [feat] Add password reset endpoint (Alice, 4 days ago)
j0k1l2m - [chore] Update dependencies (Bob, 5 days ago)

# Status
$ git status
On branch feature/user-authentication
Your branch is up to date with 'origin/feature/user-authentication'.

nothing to commit, working tree clean
```

### Git Manager 回報

```
📊 Git 狀態報告

**當前分支:**
- Active: feature/user-authentication
- Up to date with remote: ✅
- Working tree: clean ✅

**本地分支 (4):**
- develop
- feature/email-verification
- feature/user-authentication (current)
- main

**最近活動 (Last 10 commits):**
1. a1b2c3d - [test] Add integration tests for auth (Alice, 2 hours ago)
2. b2c3d4e - [feat] Add logout endpoint (Alice, 3 hours ago)
3. c3d4e5f - [feat] Add login endpoint (Bob, 5 hours ago)
4. d4e5f6g - [feat] Add register endpoint (Bob, 6 hours ago)
5. e5f6g7h - [refactor] Extract JWT logic (Charlie, 1 day ago)
...

**Commit 類型統計:**
- [feat]: 4 commits
- [test]: 2 commits
- [refactor]: 1 commit
- [fix]: 1 commit
- [docs]: 1 commit
- [chore]: 1 commit

**團隊貢獻:**
- Alice: 4 commits
- Bob: 4 commits
- Charlie: 2 commits

**建議:**
✅ Everything looks good! Ready to create PR or continue development.
```

---

## 場景 6：清理已 Merge 的分支

### Orchestrator 調用

```markdown
我將調用 Git Manager 清理已合併的舊分支。

**Context:**
- PR #450 (feature/old-feature) 已 merged 到 main
- 需要清理本地和遠端分支

**Task:**
1. 確認 PR 已 merged
2. 刪除本地分支
3. 刪除遠端分支
```

### Git Manager 執行

```bash
# Step 1: 確認分支已 merged
$ git checkout main
$ git pull origin main

$ git branch --merged
  feature/old-feature
  main

# Step 2: 刪除本地分支
$ git branch -d feature/old-feature
Deleted branch feature/old-feature (was a1b2c3d).

# Step 3: 刪除遠端分支
$ git push origin --delete feature/old-feature
To https://github.com/user/repo.git
 - [deleted]         feature/old-feature
```

### Git Manager 回報

```
✅ 分支清理完成！

**已刪除:**
- Local branch: feature/old-feature (was a1b2c3d)
- Remote branch: origin/feature/old-feature

**確認:**
- PR #450 已成功 merged 到 main
- 分支已安全刪除（已完整 merge）

**剩餘分支:**
- main (current)
- develop
- feature/user-authentication
- feature/email-verification

**建議:**
定期清理已 merged 的分支以保持倉庫整潔。
```

---

## 進階場景：與其他 Agent 協作

### 完整開發流程

```
1. Backend Developer
   ↓ (完成代碼)

2. Git Manager
   ↓ (commit 變更)

3. Backend Code Reviewer
   ↓ (審查通過)

4. Git Manager
   ↓ (創建 PR)

5. QA
   ↓ (測試通過)

6. Git Manager
   ↓ (Merge PR / 打 tag)

7. DevOps
   ↓ (部署)
```

### Orchestrator 協調範例

```markdown
Phase 1: Development
→ Backend Developer: 實作 User Profile API
→ Git Manager: Commit 變更

Phase 2: Review
→ Backend Code Reviewer: 審查代碼
→ Git Manager: 根據 review 修改並 commit

Phase 3: PR Creation
→ Git Manager: 創建 PR，包含所有 commit 和 test 結果

Phase 4: Testing
→ QA: 執行測試
→ Git Manager: 更新 PR 狀態

Phase 5: Merge
→ Git Manager: Merge PR 並創建 release tag

Phase 6: Deployment
→ DevOps: 部署到 staging/production
```

---

## 錯誤處理範例

### 敏感檔案檢查

```bash
$ git diff --name-only
.env
internal/config/database.go

$ grep -E '\.(env|key|pem)$' <<< ".env"
.env

⚠️ 警告：發現敏感檔案

**檢測到:**
- .env (環境變數檔案)

**建議動作:**
1. 將 .env 添加到 .gitignore
2. 使用 .env.example 替代
3. 移除 staged 變更:
   ```bash
   git reset HEAD .env
   ```

**是否繼續?** 需要用戶確認後才能 commit。
```

---

## 使用提示

### 何時調用 Git Manager

✅ **應該調用:**
- 完成一個功能/修復後
- 需要創建 PR 時
- 需要同步分支時
- 清理舊分支時

❌ **不需要調用:**
- 只是查看文件內容
- 運行測試
- 編譯代碼

### 最佳實踐

1. **頻繁 commit** - 每個邏輯變更一個 commit
2. **清晰訊息** - 遵循 [type] format
3. **小 PR** - 一次只處理一個功能
4. **及時同步** - 定期 rebase/merge main

---

## 工具檢查

在調用 Git Manager 前，確保：

```bash
# Git 已安裝
$ git --version
git version 2.39.0

# GitHub CLI 已安裝（PR 功能需要）
$ gh --version
gh version 2.40.0

# 已登入 GitHub
$ gh auth status
✓ Logged in to github.com as username
```

如果工具缺失，Git Manager 會提供安裝指引。
