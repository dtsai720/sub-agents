# Development Guidelines

## Philosophy

### Core Beliefs

- **Incremental progress over big bangs** - Small changes that compile and pass tests
- **Learning from existing code** - Study and plan before implementing
- **Pragmatic over dogmatic** - Adapt to project reality
- **Clear intent over clever code** - Be boring and obvious

### Simplicity Means

- Single responsibility per function/class
- Avoid premature abstractions
- No clever tricks - choose the boring solution
- If you need to explain it, it's too complex

## Process

### 1. Planning & Staging

Break complex work into 3-5 stages. Document in `IMPLEMENTATION_PLAN_{AGENT_NAME}.md`:

**Naming Convention:**

- Backend (Go): `IMPLEMENTATION_PLAN_BACKEND_GO.md`
- Backend (Java): `IMPLEMENTATION_PLAN_BACKEND_JAVA.md`
- Backend (Python): `IMPLEMENTATION_PLAN_BACKEND_PYTHON.md`
- Frontend: `IMPLEMENTATION_PLAN_FRONTEND.md`

**Planning Workflow:**

#### Phase 1: Plan Generation (Sub-Agent)

1. **Generate Plan** - Sub-Agent creates implementation plan with 3-5 stages
2. **Output Plan** - Save as `IMPLEMENTATION_PLAN_{AGENT_NAME}.md`
3. **Report to Orchestrator** - Return plan in completion report, STOP execution

#### Phase 2: User Review (Orchestrator)

4. **Present Plan** - Orchestrator shows plan to user
5. **Wait for Confirmation** - User approves/modifies the plan
6. **Decision Point**:
   - If approved → Proceed to Phase 3
   - If modified → Orchestrator updates plan file

#### Phase 3: Implementation (Sub-Agent, second invocation)

7. **Execute Plan** - Orchestrator re-invokes Sub-Agent with approved plan
8. **Implement Stages** - Sub-Agent follows plan, updates status
9. **Update Progress** - Mark stages as completed during implementation
10. **Cleanup** - Remove plan file when all stages are done

**Plan Template:**

```markdown
# Implementation Plan - [Agent Name]

**Agent:** [Backend-Go / Backend-Java / Backend-Python / Frontend]
**Created:** [Date]
**Status:** [Planning / Approved / In Progress / Complete]

---

## Stage 1: [Name]
**Goal**: [Specific deliverable]
**Success Criteria**: [Testable outcomes]
**Tests**: [Specific test cases]
**Files to Create/Modify**: [List of files]
**Status**: [Not Started|In Progress|Complete]

## Stage 2: [Name]
...

---

## Review Checklist
- [ ] Plan reviewed by user
- [ ] All stages have clear goals
- [ ] Tests are well-defined
- [ ] No missing dependencies
```

**Important Rules:**

**For Sub-Agents:**

- ⚠️ **CANNOT wait for user interaction** - Must complete in single execution
- When generating plan: Output plan, report to Orchestrator, STOP
- When implementing: Follow approved plan from Orchestrator input
- Update stage status during implementation
- Each stage should be independently testable

**For Orchestrator:**

- ⚠️ **MUST present plan to user before implementation**
- Read plan file, show to user, wait for approval
- Re-invoke Sub-Agent only after user confirms
- Pass approved plan as context to Sub-Agent

### 2. Implementation Flow

1. **Understand** - Study existing patterns in codebase
2. **Test** - Write test first (red)
3. **Implement** - Minimal code to pass (green)
4. **Refactor** - Clean up with tests passing
5. **Prepare for commit** - Ensure code is ready with clear changes documented

### 3. When Stuck (After 3 Attempts)

**CRITICAL**: Maximum 3 attempts per issue, then STOP.

1. **Document what failed**:
   - What you tried
   - Specific error messages
   - Why you think it failed

2. **Research alternatives**:
   - Find 2-3 similar implementations
   - Note different approaches used

3. **Question fundamentals**:
   - Is this the right abstraction level?
   - Can this be split into smaller problems?
   - Is there a simpler approach entirely?

4. **Try different angle**:
   - Different library/framework feature?
   - Different architectural pattern?
   - Remove abstraction instead of adding?

## Technical Standards

### Architecture Principles

- **Composition over inheritance** - Use dependency injection
- **Interfaces over singletons** - Enable testing and flexibility
- **Explicit over implicit** - Clear data flow and dependencies
- **Test-driven when possible** - Never disable tests, fix them

### Code Quality

- **Every commit must**:
  - Compile successfully
  - Pass all existing tests
  - Include tests for new functionality
  - Follow project formatting/linting

- **Before committing**:
  - Run formatters/linters
  - Self-review changes
  - Ensure commit message explains "why"

### Error Handling

- Fail fast with descriptive messages
- Include context for debugging
- Handle errors at appropriate level
- Never silently swallow exceptions

## Clean Code & Design Principles

### Core Philosophy

1. **Start Simple** - Write the simplest code that works
2. **YAGNI** - You Aren't Gonna Need It (don't add features speculatively)
3. **Rule of Three** - Duplicate once, refactor on third occurrence
4. **Test-Driven** - If it's hard to test, refactor for dependency injection
5. **Readable > Clever** - Code is read 10x more than written
6. **Patterns Emerge** - Don't force patterns, let them emerge from refactoring

### SOLID Principles

#### S - Single Responsibility Principle (SRP)

**Definition:** A class/function should have one reason to change.

**When to Apply:**

- ✅ Function > 30 lines → Consider splitting
- ✅ Class has multiple unrelated dependencies
- ✅ Test setup requires mocking unrelated services

**When NOT to Apply:**

- ❌ Simple CRUD with < 3 operations
- ❌ Creating interfaces for single implementation
- ❌ Splitting logic that always changes together

#### O - Open/Closed Principle (OCP)

**Definition:** Open for extension, closed for modification.

**When to Apply:**

- ✅ You have 3+ similar implementations (use Strategy Pattern)
- ✅ New features will be added frequently
- ✅ Business rules vary by context

**When NOT to Apply:**

- ❌ Only 1-2 implementations exist and unlikely to grow
- ❌ Logic is simple and stable (if/else is fine)
- ❌ Premature abstraction (YAGNI violation)

#### L - Liskov Substitution Principle (LSP)

**Definition:** Subtypes must be substitutable for their base types.

**When to Apply:**

- ✅ Inheritance hierarchy feels forced
- ✅ Subclass needs to disable parent methods
- ✅ Tests fail when swapping implementations

**When NOT to Apply:**

- ❌ Not using inheritance (favor composition)
- ❌ Interface has single implementation

#### I - Interface Segregation Principle (ISP)

**Definition:** Clients shouldn't depend on interfaces they don't use.

**When to Apply:**

- ✅ Interface has > 5 methods
- ✅ Implementations need to stub/panic methods
- ✅ Different clients use different subsets of methods

**When NOT to Apply:**

- ❌ All clients use all methods
- ❌ Interface has < 3 methods
- ❌ Breaking interfaces would cause more harm

#### D - Dependency Inversion Principle (DIP)

**Definition:** Depend on abstractions, not concretions.

**When to Apply:**

- ✅ Need to swap implementations (testing, multi-DB support)
- ✅ Logic depends on external services (DB, API, filesystem)
- ✅ Want to test without real dependencies

**When NOT to Apply:**

- ❌ Single implementation, no testing needed
- ❌ Standard library types already abstract
- ❌ Simple utilities with no external dependencies

### Design Patterns - When to Use

#### Pattern Selection Process

```text
Is the problem recurring?
  NO  → Don't use a pattern, solve directly
  YES ↓

Do you have 3+ similar implementations?
  NO  → Wait (YAGNI)
  YES ↓

Use Pattern
```

#### Common Patterns

**Strategy Pattern:**

- Use: Multiple algorithms for same task, need to swap behavior at runtime
- Don't: Only 1-2 implementations, behavior never changes

**Factory Pattern:**

- Use: Complex object creation logic, creation depends on configuration
- Don't: Simple constructor suffices, no conditional creation logic

**Repository Pattern:**

- Use: Abstracting data access layer, supporting multiple databases
- Don't: Simple CRUD with no business logic, ORM already abstracts DB

**Dependency Injection:**

- Use: ALWAYS for services with external dependencies
- Don't: Simple utilities with no dependencies

### Over-Engineering Red Flags

**🚩 Warning Signs:**

1. **Interfaces with Single Implementation** - "Just in case" abstractions
2. **Too Many Layers** - Excessive indirection (Controller → Service → Manager → Provider → Repository → DAO)
3. **Generic/Abstract Names** - DataManager, BaseHandler, AbstractFactory, Helper, Util
4. **Premature Optimization** - Complex caching before profiling
5. **Configuration Hell** - 50+ config parameters, no sensible defaults

**Acceptable Thresholds:**

- Function: < 50 lines (ideally < 20)
- Class: < 300 lines
- Parameters: < 5 (ideally < 3)
- Nesting depth: < 4
- Dependencies per class: < 7

### Before Writing Code - Decision Process

1. **Understand Requirement** - What problem am I solving? Is there existing code?
2. **Check Complexity** - Can this be solved in < 30 lines?
   - YES → Write simple code, no patterns
   - NO → Continue evaluation
3. **Identify Variability** - Will there be multiple implementations?
   - NO → No interface needed
   - YES → Define interface (Strategy, DIP)
4. **Check Testability** - Can I test without mocking?
   - YES → No DI needed
   - NO → Use dependency injection
5. **Write Simplest Code** - Start concrete, add abstraction only when needed

### After Writing Code - Self-Review

- [ ] Function < 50 lines? (SRP)
- [ ] Class < 300 lines? (SRP)
- [ ] Can I explain it in one sentence? (Simplicity)
- [ ] Can I test it easily? (DIP)
- [ ] No duplicated code? (DRY)
- [ ] No premature abstractions? (YAGNI)
- [ ] Variable/function names are clear?
- [ ] No magic numbers/strings?
- [ ] Error handling is explicit?

**Pattern Verification:**

- Do I have 3+ similar implementations? → Use Strategy
- Do I need to swap implementations? → Use DI + Interface
- Do I need complex object creation? → Use Factory
- Otherwise → Keep it simple

## Decision Framework

When multiple valid approaches exist, choose based on:

1. **Testability** - Can I easily test this?
2. **Readability** - Will someone understand this in 6 months?
3. **Consistency** - Does this match project patterns?
4. **Simplicity** - Is this the simplest solution that works?
5. **Reversibility** - How hard to change later?

## Project Integration

### Learning the Codebase

- Find 3 similar features/components
- Identify common patterns and conventions
- Use same libraries/utilities when possible
- Follow existing test patterns

### Tooling

- Use project's existing build system
- Use project's test framework
- Use project's formatter/linter settings
- Don't introduce new tools without strong justification

## Quality Gates

### Definition of Done

- [ ] Tests written and passing
- [ ] Code follows project conventions
- [ ] No linter/formatter warnings
- [ ] Commit messages are clear
- [ ] Implementation matches plan
- [ ] No TODOs without issue numbers

### Test Guidelines

- Test behavior, not implementation
- One assertion per test when possible
- Clear test names describing scenario
- Use existing test utilities/helpers
- Tests should be deterministic

## Important Reminders

**NEVER**:

- Use `--no-verify` to bypass commit hooks
- Disable tests instead of fixing them
- Commit code that doesn't compile
- Make assumptions - verify with existing code

**ALWAYS**:

- Prepare working code incrementally for review
- Update plan documentation as you go
- Learn from existing implementations
- Stop after 3 failed attempts and reassess
