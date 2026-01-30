---
description: Run/analyze existing tests or validate test coverage (v2.18 TDD workflow)
allowed-tools: ["Task", "TaskCreate", "TaskUpdate", "TaskList", "Read"]
---

# Testing Phase

## IMPORTANT CHANGE in v2.18

**Old behavior (v2.17):** Create tests for existing code
**New behavior (v2.18):** Tests created during TDD (in /develop). This command validates coverage or runs existing tests.

## Usage

```
/test
```

## What This Command Does (v2.18+)

### If TDD Workflow Was Used (tests exist)

1. **Validate test coverage** against `.dev/05-test-specification.md`
2. **Run test-reviewer** to verify:
   - All test specifications were implemented
   - Test quality is high
   - Coverage is comprehensive
3. **Output:** `.dev/06-test-review.md`

### If Tests Don't Exist (Traditional Workflow)

**Error message:**
```
❌ No tests found and no test specification exists.

In v2.18+, tests are created during /develop using TDD workflow.

Options:
1. Run /plan to create test specification
2. Run /develop with test specification for TDD implementation
3. (Not recommended) Create tests after-the-fact

For TDD workflow:
  /plan "your feature" → /develop
```

## Step 0: Check Test Status

```
# Check for test specification
Read .dev/05-test-specification.md

# Check for test codeunits
Glob pattern: "src/Tests/**/*.al" OR "**/*Test*.al"

# Check for TDD log
Read .dev/03-tdd-log.md
```

### Scenarios

**Scenario A: TDD workflow completed (recommended)**
- Test specification exists: `.dev/05-test-specification.md`
- Tests exist: `src/Tests/*.al`
- TDD log exists: `.dev/03-tdd-log.md`
- **Action:** Validate coverage with test-reviewer

**Scenario B: Tests exist, no specification**
- Tests exist but no `.dev/05-test-specification.md`
- **Action:** Review test quality only (cannot validate against spec)

**Scenario C: No tests exist**
- No test codeunits found
- **Action:** Guide user to use TDD workflow or create tests first

## Step 0: Check/Create Tasks

**Check for existing tasks:**

```
TaskList → Check if Test Validation task exists
  - If exists → Continue
  - If not exists → Create test validation task
```

**Create task if needed:**

```
TaskCreate: "Test Validation"
  - description: "Validate test coverage matches specification"
  - activeForm: "Validating test coverage"
```

## Workflow

### Scenario A: TDD Workflow (Validate Coverage)

1. **TaskUpdate:** "Test Validation" → status: "in_progress"
2. **Read test specification:** `.dev/05-test-specification.md`
3. **Find test codeunits:** Glob for test files
4. **Spawn test-reviewer** with instructions:
   - Read test specification
   - Read implemented tests
   - Verify all specs were implemented
   - Check test quality
   - Report coverage percentage
5. **TaskUpdate:** "Test Validation" → status: "completed"
6. **Show summary** of `.dev/06-test-review.md`

### Scenario B: No Specification (Quality Review Only)

1. **TaskUpdate:** "Test Validation" → status: "in_progress"
2. **Find test codeunits:** Glob for test files
3. **Spawn test-reviewer** with instructions:
   - Review test quality
   - Cannot validate against specification (doesn't exist)
   - Report on test patterns and quality
4. **TaskUpdate:** "Test Validation" → status: "completed"
5. **Show summary**

### Scenario C: No Tests (Error)

```
❌ No tests found.

Detected workflow: [Traditional / Unknown]

In v2.18+, tests are created during /develop via TDD:

Recommended workflow:
1. /plan "your feature"
   - Creates requirements, solution plan, test specification
2. /develop
   - Implements tests + code via TDD
3. /test (optional)
   - Validates coverage

If you want to add tests to existing code:
1. Manually run test-engineer agent to create test specification
2. Run al-developer with TDD workflow to implement tests
3. Or manually write tests (not recommended)

TDD workflow ensures tests are correct (RED phase proves test can fail).
Adding tests after code doesn't provide this confidence.
```

## Output

**TDD Workflow:**
- `.dev/06-test-review.md` - Coverage validation report
- Shows: specs implemented, coverage %, test quality

**Traditional Workflow:**
- `.dev/06-test-review.md` - Test quality report only

## When to Use

### Use /test when:
- **After TDD development** - Validate final coverage
- **Testing existing tests** - Want quality review of test code
- **Debugging test failures** - Analyze why tests are failing

### Don't use /test when:
- **No tests exist and no specification** - Use /plan then /develop instead
- **Want to create tests** - Tests are created in /develop (TDD workflow)

## Example Sessions

### Example 1: After TDD Workflow (Success)

```
User: /test

Claude: Checking test status...
- Test specification found: .dev/05-test-specification.md
- Test codeunits found: 3 files (Cod50200, Cod50201, Cod50202)
- TDD log found: .dev/03-tdd-log.md

Spawning test-reviewer to validate coverage...

Test validation complete → .dev/06-test-review.md

Summary:
- Test specifications: 12 total
- Implemented tests: 12 (100%)
- Coverage: 100% of specifications
- Test quality: Excellent
- All tests passing: ✅

✅ Test coverage validated successfully.
```

### Example 2: No Tests Found (Error)

```
User: /test

Claude: Checking test status...
- Test specification: Not found
- Test codeunits: Not found
- TDD log: Not found

❌ No tests found.

In v2.18+, tests are created during /develop using TDD workflow.

Recommended next steps:
1. /plan "your feature" - Creates test specification
2. /develop - Implements tests via TDD

Or if you already have production code and want to add tests:
- You can manually run test-engineer to create specification
- Then run al-developer with TDD workflow

Note: TDD (tests before code) provides higher confidence than
tests after code, because RED phase proves tests can fail.
```

### Example 3: Tests Exist, No Specification

```
User: /test

Claude: Checking test status...
- Test specification: Not found
- Test codeunits found: 2 files
- TDD log: Not found

Warning: Tests exist but no test specification found.
Cannot validate coverage against specification.
Will review test quality only.

Spawning test-reviewer...

Test quality review complete → .dev/06-test-review.md

Summary:
- Test codeunits: 2
- Test procedures: 8
- Test quality: Good
- Patterns used: [GIVEN]/[WHEN]/[THEN]
- Mocks: Some tests use mocks

Note: Without test specification, cannot verify if all
requirements are covered. Consider creating specification
for future development.
```

## Migration from v2.17 to v2.18

**If you have existing projects without test specifications:**

1. **Option A: Continue without TDD** (not recommended)
   - Use /test as before
   - Tests reviewed for quality only
   - No coverage validation

2. **Option B: Adopt TDD for new features** (recommended)
   - Use /plan then /develop for new work
   - TDD workflow creates tests + code together
   - High confidence in test correctness

3. **Option C: Retrofit test specifications** (optional)
   - Manually create `.dev/05-test-specification.md` for existing code
   - Document what existing tests should cover
   - Use /test to validate against specification

## Key Takeaway

**v2.18 Changes:**
- Tests created DURING development (TDD), not AFTER
- /test command validates coverage, doesn't create tests
- Test specifications drive implementation
- Higher confidence: RED phase proves test can fail

**For new projects:** Use /plan → /develop workflow
**For existing projects:** /test still works but only reviews quality
