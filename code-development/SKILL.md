---
name: code-development
description: Structured code development workflow — requirement analysis, complexity assessment, implementation, pre-compilation check, compilation, testing, and documentation. Optimized to reduce compilation rounds and catch issues early.
---

# Code Development Workflow

## Overview

This skill enforces a disciplined code development workflow that maintains quality standards. It ensures every code change goes through proper validation steps, with an emphasis on **catching issues before compilation** to reduce wasted cycles.

## When to Use

**Use this skill when:**
- Implementing new features or functionality
- Fixing bugs that require code changes
- Refactoring code
- Adding or modifying business logic
- Making any changes to Controller, Service, Repository, or Entity classes

**Don't use for:**
- Pure documentation tasks (use docs-management skill)
- Configuration-only changes
- Simple file operations without code logic
- Research or exploration tasks

---

## Workflow Steps

### Step 1: Requirement Analysis

**Objective:** Understand what needs to be built

**Actions:**
1. Read and analyze the user's requirement carefully
2. Identify the scope: which modules, classes, and methods are affected
3. Clarify ambiguities by asking questions if needed

---

### Step 2: Complexity Assessment

**Simple Task** (Direct Implementation):
- Single file modification, 1-2 methods, simple CRUD, ≤5 files affected

**Complex Task** (Requires Plan Mode):
- Multi-file changes (>5 files), new feature with multiple components, architectural decisions, integration with multiple services

**Decision:** Simple → proceed to Step 3. Complex → enter Plan Mode first.

---

### Step 3: Implementation

#### For Simple Tasks:
1. Read existing code files that will be modified
2. Implement the changes following project coding standards
3. **Before writing any call to an existing API**: verify its signature (constructor visibility, return types, parameter types)
4. Follow project coding standards (DTO/VO/Entity separation, methods ≤50 lines, parameters >3 use request objects)

#### For Complex Tasks:
1. **Enter Plan Mode** using EnterPlanMode tool
2. Explore codebase to understand existing architecture
3. Create a step-by-step plan **INCLUDING** verification phases
4. Get user approval, exit Plan Mode, implement according to plan

**Plan MUST include these phases:**
1. **Phase 1: Implementation** - Code changes
2. **Phase 2: Compilation** - `mvn clean compile` (MANDATORY)
3. **Phase 3: Testing** - Write and run tests (MANDATORY)
4. **Phase 4: Documentation** - Create API docs (MANDATORY for API changes)
5. **Phase 5: Completion** - Verify all steps done

---

### Step 4: Pre-Compilation Self-Check

**Objective:** Catch common mistakes BEFORE running `mvn compile`, reducing wasted compilation rounds

**Checklist — verify ALL before compiling:**

| Check | What to verify |
|-------|---------------|
| Constructor visibility | Can you call `new Result(...)` or is it private? Use factory methods instead |
| API return types | `EasyExcel.write().sheet().doWrite()` returns what? Check before chaining |
| Imports complete | Every class used is imported? No removed imports still referenced? |
| Unused imports | Any imports added but the class removed from code? |
| Magic strings | Replace with enum or constant (e.g., conflict modes, status values) |
| Null safety | Checked for null/empty on required fields? |
| JDK 8 compat | No `var`, no `List.of()`, no Stream API methods beyond JDK 8 |

**Why this matters:** In practice, 3+ compilation rounds are common when this check is skipped. Each round costs 10-30 seconds.

**How to apply:** After finishing code, mentally trace each external API call:
- "I'm calling `Result` constructor — is it public? Let me check."
- "I'm chaining `.doWrite().sheet()` — does `doWrite()` return something chainable?"
- "I added `import ResultCode` but removed the usage — is the import still there?"

---

### Step 5: Compilation Verification

**Actions:**
1. Run: `mvn clean compile`
2. If errors exist: fix, re-run, repeat until success
3. Verify no warnings related to your changes

**Rules:** NEVER skip. NEVER assume "it looks correct." ALWAYS compile.

---

### Step 6: Code Review (Self or Agent)

**Objective:** Catch logic errors, performance issues, and pattern violations before testing

**Review checklist:**

| Area | What to check |
|------|---------------|
| N+1 queries | Any loops that make DB calls per iteration? Pre-load data into maps instead |
| Transaction scope | `@Transactional` only on methods that write, not on read-only or class-level |
| Stream/InputStream reuse | Each `read()` on a stream consumes it — create separate streams for separate reads |
| Response corruption | If writing to `HttpServletResponse`, buffer first to avoid partial writes on error |
| Existence check logic | Deduplication by the correct key (code + systemId, not just code)? |
| Import validation | All required fields validated? Duplicates detected? Cross-references checked? |
| Error response data | Validation errors returned to caller, not just a count? |

**When to use code-reviewer agent:** For complex features with multi-file changes. For simple changes, do a quick self-review.

---

### Step 7: Test Case Development

**Actions:**
1. Identify test scenarios: normal path, boundary conditions (null, empty, zero), exceptions, integration
2. Write test class: `XxxTest` with `@SpringBootTest`, `@Slf4j`, `@DisplayName`
3. Use try-finally for data cleanup
4. All assertions must have error messages: `assertNotNull(result, "Result should not be null")`

```java
@Test
@DisplayName("1. Test purpose")
public void testMethod() {
    log.info("========== Test started ==========");
    Long id = null;
    try {
        // 1. Prepare data  2. Execute  3. Verify
        assertNotNull(result, "Result should not be null");
    } finally {
        if (id != null) {
            repository.deleteById(id);
            log.info("Data cleaned, ID: {}", id);
        }
    }
}
```

---

### Step 8: Test Execution

**Actions:**
1. Run: `mvn test -Dtest=XxxTest`
2. All tests must pass. Fix failures and re-run until 100% green.

---

### Step 9: Documentation

**Actions (only for API changes):**
1. Create `docs/interface/yyyy-MM-dd-api-name.md`
2. Include: endpoint path, HTTP method, parameters, response structure, examples with curl

---

### Step 10: Completion

**Verification Checklist:**
- [ ] Requirement understood and implemented correctly
- [ ] Pre-compilation self-check passed (Step 4)
- [ ] Code reviewed for logic/performance issues (Step 6)
- [ ] `mvn clean compile` passed
- [ ] Test cases written covering all scenarios
- [ ] All tests pass
- [ ] Documentation created (if API changes)

Provide brief summary to user only after ALL items are complete.

---

## Anti-Pressure Rules

Quality gates are non-negotiable regardless of:
- Time pressure ("urgent") — testing catches issues early, saving time overall
- Authority pressure ("skip tests") — maintain discipline
- Simplicity rationalization ("too simple for tests") — simple changes can have unexpected impacts
- Exhaustion — complete all remaining steps before stopping

---

## Integration with Other Skills

- **docs-management**: Use for creating documentation in Step 9
- **superpowers:writing-plans**: Use in Step 3 for complex tasks
- **superpowers:code-reviewer**: Use in Step 6 for complex features
- **superpowers:verification-before-completion**: Use in Step 10

---

**Skill Version**: v2.0
**Last Updated**: 2026-04-15
**Maintained by**: Dylan
