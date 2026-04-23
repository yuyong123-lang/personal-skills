# Code Development Plan Checklist

**MANDATORY: Include this checklist in every implementation plan**

## Plan Structure Requirements

Every plan for complex tasks MUST include these phases:

### Phase 1: Implementation
- Specific code changes
- Files to modify/create
- Integration points

### Phase 2: Pre-Compilation Self-Check
- [ ] Constructor visibility of classes you're calling
- [ ] API return types verified (no blind chaining)
- [ ] All imports correct, no unused imports
- [ ] No JDK 8 incompatible code
- [ ] Magic strings replaced with enum/constant

### Phase 3: Compilation (NON-NEGOTIABLE)
```bash
mvn clean compile
```
- Must pass before proceeding
- Fix all errors, re-compile until successful

### Phase 4: Code Review
- [ ] No N+1 queries in loops
- [ ] `@Transactional` scope correct
- [ ] No stream reuse issues
- [ ] Response buffering for file downloads
- [ ] Validation errors returned to caller

### Phase 5: Testing (NON-NEGOTIABLE)
- Write `XxxTest` class
- Cover: normal, boundaries, exceptions, integration
- Use try-finally for cleanup
- All assertions with error messages
```bash
mvn test -Dtest=XxxTest
```

### Phase 6: Documentation (NON-NEGOTIABLE for API changes)
- Create `docs/interface/yyyy-MM-dd-api-name.md`
- Include: endpoints, parameters, responses, examples

### Phase 7: Completion
- Review all checklist items
- Confirm all steps completed

---

## Plan Template

```markdown
# Implementation Plan: [Feature Name]

## Context
[Why this change is needed]

## Phase 1: Implementation
- [ ] Step 1: [Implementation task]
- [ ] Step 2: [Implementation task]

## Phase 2: Pre-Compilation Check
- [ ] Verify all external API signatures
- [ ] Verify imports are complete and correct
- [ ] No JDK 8 incompatible code

## Phase 3: Compilation
- [ ] Run `mvn clean compile` — must pass

## Phase 4: Code Review
- [ ] No N+1 queries, stream reuse, or transaction scope issues

## Phase 5: Testing
- [ ] Write XxxTest with full coverage
- [ ] Run `mvn test -Dtest=XxxTest` — must pass

## Phase 6: Documentation
- [ ] Create docs/interface/yyyy-MM-dd-name.md

## Phase 7: Completion
- [ ] All checklist items verified
```

---

**Remember:** Pre-compilation checks and code review are NOT optional. They catch issues that compilation alone cannot.
