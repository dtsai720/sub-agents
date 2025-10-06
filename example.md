# Development Guidelines (System Rules)

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

Break complex work into 3-5 stages. Document in `IMPLEMENTATION_PLAN.md`:

```markdown
## Stage N: [Name]
**Goal**: [Specific deliverable]
**Success Criteria**: [Testable outcomes]
**Tests**: [Specific test cases]
**Status**: [Not Started|In Progress|Complete]
```

- Update status as you progress
- Remove file when all stages are done

### 2. Implementation Flow

1. **Understand** - Study existing patterns in codebase
2. **Test** - Write test first (red)
3. **Implement** - Minimal code to pass (green)
4. **Refactor** - Clean up with tests passing
5. **Commit** - With clear message linking to plan

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

### SOLID Principles

Apply SOLID principles consistently:

- **Single Responsibility Principle (SRP)** - Each class/function should have only one reason to change
- **Open/Closed Principle (OCP)** - Open for extension, closed for modification
- **Liskov Substitution Principle (LSP)** - Subtypes must be substitutable for their base types
- **Interface Segregation Principle (ISP)** - Clients should not depend on interfaces they don't use
- **Dependency Inversion Principle (DIP)** - Depend on abstractions, not concretions

### Object-Oriented Programming Concepts

Leverage OOP fundamentals effectively:

- **Encapsulation** - Hide internal implementation details, expose clean interfaces
- **Inheritance** - Use when there's a clear "is-a" relationship, prefer composition otherwise
- **Polymorphism** - Enable flexible behavior through interfaces and abstractions
- **Abstraction** - Focus on essential features, hide unnecessary complexity

### Design Patterns

Reference proven design patterns when appropriate:

- **Creational Patterns** - Factory, Builder, Singleton (use sparingly)
- **Structural Patterns** - Adapter, Decorator, Facade
- **Behavioral Patterns** - Observer, Strategy, Command
- **Architectural Patterns** - Repository, Service Layer, MVC/MVP

Choose patterns that solve actual problems, not for pattern's sake

### Go-Specific Guidelines

- **Type definitions**:
  - Use `any` instead of `interface{}` (Go 1.18+)
  - Prefer specific interface types when possible
  - Use generics for type-safe collections and functions

- **Function design**:
  - **Parameter limit**: Functions should not have more than 5 parameters - use structs or options patterns for complex parameter sets
  - **Context handling**: Never include `context.Context` in structs - always pass as the first parameter
  - **Logger handling**: Never include `logger.ILogger` in structs - always pass as the second parameter after context
  - **Parameter order**: When both `ctx context.Context` and `log logger.ILogger` are present, they must be the first and second parameters respectively
  - **Loop syntax**: Use `for i := range n` instead of `for i := 0; i < n; i++` for simple integer iteration (Go 1.22+)

- **Interface design**:
  - **Interface requirement**: Every struct that provides business logic should have a corresponding interface
  - **Interface naming**: Use `I` prefix for interfaces (e.g., `IService`, `IRepository`, `IProcessor`)
  - **Dependency injection**: Depend on interfaces, not concrete implementations

- **Unit Testing**:
  - **Use testify package** for all unit tests - provides rich assertion and mocking capabilities
  - Import: `github.com/stretchr/testify/assert`, `github.com/stretchr/testify/require`, `github.com/stretchr/testify/mock`
  - **Use table-driven tests** for comprehensive test coverage with multiple test cases - this is mandatory for all unit tests
  - **Use parallel testing**: Add `t.Parallel()` to all unit tests to enable concurrent execution and improve test performance
  - **Exception for environment variables**: Do not use `t.Parallel()` when using `t.Setenv()` as environment variables are global and cause race conditions
  - Prefer `require` for critical assertions that should stop test execution on failure
  - Use `assert` for non-critical assertions that allow test continuation
  - Use `testify/mock` for mocking dependencies and interfaces
  - Follow testify conventions for cleaner, more readable test code
  - **Error assertion pattern**: Use `assert.Equal(t, hasError, err != nil)` instead of `assert.Equal(t, expectedError, err)` for better error testing clarity

- **Mock Data with types.MockItem**:
  - Use `types.MockItem[T]` for consistent test data structure in table-driven tests
  - Structure: `MockItem{Count: int, Error: error, Item: T}`
  - `Count`: Number of times the mock method should be called (use with `Times()`)
  - `Error`: Error to return from mock method (use with `Return()`)
  - `Item`: Typed data payload for the test case (generic type T)
  - **Usage beyond gomock**: Use `types.MockItem[T]` whenever you have `expectedResult` and `expectedError` patterns in table-driven tests, not limited to gomock scenarios
  - **Mock execution pattern**: Use `Times(Count)` directly without checking `Count > 0` - `Times(0)` means the method won't be called
  - **Avoid conditional mock setup**: Never use `if count > 0 { mock().Times(count) }` - always use `mock().Times(count)` directly
  - Example usage:

    ```go
    response: types.MockItem[any]{Count: 1, Error: nil}
    mockService.EXPECT().Method(params).Return(response.Error).Times(response.Count)
    ```

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

### Git and Version Control

- **Commit authorship**:
  - Do not modify commit authors or add tool signatures
  - Keep commit messages clean and professional
  - Never add "Generated with [Tool Name]" or similar signatures
  - Preserve original authorship and ownership

- **Pull requests**:
  - Use clear, descriptive titles that explain the change
  - Write comprehensive descriptions focusing on the "why" and "what"
  - Do not include tool attribution in PR descriptions
  - Focus on technical content and business value

### Error Handling

- Fail fast with descriptive messages
- Include context for debugging
- Handle errors at appropriate level
- Never silently swallow exceptions

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

### Mock Generation

Use `scripts/genmock.sh` to generate mocks for interfaces:

1. **Add interface to SOURCE array** in `scripts/genmock.sh`:

   ```bash
   SOURCE=(
       "github.com/web4ux/path/to/package:InterfaceName"
       # ... other entries
   )
   ```

2. **Ensure interface is defined** in `interface.go` within the package

3. **Run mock generation**:

   ```bash
   ./scripts/genmock.sh
   ```

4. **Output structure**:

   ```text
   mocks/
   ├── package_name/mock.go
   └── another_package/mock.go
   ```

The script generates mocks using `mockgen` with proper package naming and folder structure.

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

- Commit working code incrementally
- Update plan documentation as you go
- Learn from existing implementations
- Stop after 3 failed attempts and reassess

# Task Instructions

When I give you a task (e.g., implement a feature, refactor code, write tests, or review code), you must:

1. **Always comply with the Development Guidelines**:
   - Apply Go-specific rules, testing standards, SOLID principles, and project conventions.
   - Ensure every code sample compiles, passes tests, and follows linting/formatting rules.

2. **Output Structure**:
   - **Code**: Full implementation with proper imports, error handling, and clear structure.
   - **Tests**: Table-driven tests using `testify`, including mocks when necessary.
   - **Explanation**: Rationale of design decisions, referencing relevant rules from the Development Guidelines.

3. **Interaction Rules**:
   - If multiple valid approaches exist, explain options and recommend one based on testability, readability, consistency, simplicity, and reversibility.
   - If uncertain or missing context, always ask clarifying questions instead of making assumptions.
   - Never bypass tests, commit hooks, or coding standards.

---

# 🌐 Language Rules

- **Conversation replies → Traditional Chinese**
- **Structured outputs / documents → English**
