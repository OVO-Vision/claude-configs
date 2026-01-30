# TDD Workflow - Shared Standards

**Version:** 2.19.0

This document defines the Test-Driven Development (TDD) workflow used across the AL development profile. It is referenced by agents and commands to ensure consistent TDD discipline.

---

## Core TDD Principles

### The TDD Cycle

```
RED → GREEN → REFACTOR → (repeat)
```

1. **RED:** Write a failing test
2. **GREEN:** Make it pass with minimal code
3. **REFACTOR:** Improve code quality without changing behavior

### Critical Rules

⛔ **NEVER** implement logic before user confirms test FAILS (RED phase)
⛔ **NEVER** skip approval gates between phases
⛔ **NEVER** batch multiple cycles - one test at a time
⛔ **ALWAYS** use AskUserQuestion at phase gates (BLOCKING)

---

## Phase 1: RED - Write Failing Test

### Goal
Create a test that fails because the production code doesn't exist yet.

### Steps

1. **Read test specification** from `.dev/05-test-specification.md`
2. **Implement test codeunit** following the spec:
   ```al
   codeunit 50200 "Credit Limit Tests"
   {
       Subtype = Test;

       [Test]
       procedure Test_ValidateCreditLimit_WithinLimit()
       var
           Validator: Codeunit "Credit Limit Validator";
           MockRepo: Codeunit "Mock Customer Repository";
           Result: Boolean;
       begin
           // [GIVEN] Customer with 10000 limit, 5000 outstanding
           MockRepo.AddMockCustomer('C001', 10000, 5000);

           // [WHEN] Validating order of 3000
           Result := Validator.ValidateCreditLimit('C001', 3000, MockRepo);

           // [THEN] Should allow (5000 + 3000 < 10000)
           Assert.IsTrue(Result, 'Should allow order within limit');
       end;
   }
   ```

3. **Implement mock repositories** for dependency injection
4. **Create MINIMAL production code stubs** (compilation only - NO logic):
   ```al
   codeunit 50100 "Credit Limit Validator"
   {
       procedure ValidateCreditLimit(
           CustomerNo: Code[20];
           OrderAmount: Decimal;
           Repo: Interface ICustomerRepository
       ): Boolean
       begin
           // ⛔ NO LOGIC! Just return default so test compiles and FAILS
           exit(false);  // or exit(true) - whichever makes test FAIL
       end;
   }
   ```

5. **⛔ HARD STOP - Use AskUserQuestion:**
   ```markdown
   RED Phase Complete: [TestName]

   Please deploy to BC and run the test. Did it FAIL as expected?

   Options:
   - Yes - Test FAILED → Continue to GREEN phase
   - No - Test PASSED → ⚠️ TDD VIOLATION! Test is wrong or code exists
   - Deployment issue → Need troubleshooting
   ```

6. **Wait for user response** - DO NOT proceed to GREEN until user confirms FAIL

### RED Phase Violations

**If test PASSES in RED phase:**
```
❌ TDD VIOLATION: Test passed before production code exists!

This means:
- Test assertions are vacuous (testing nothing)
- Production code already exists somewhere
- Test specification is incorrect

⛔ CANNOT proceed to GREEN phase.
⛔ DO NOT write any production code.

Please investigate the test.
```

**STOP HERE. Wait for user.**

---

## Phase 2: GREEN - Make It Pass

### Goal
Write minimal production code to make the test pass.

### Steps

1. **NOW implement ACTUAL production logic** (replace RED phase stubs):
   ```al
   codeunit 50100 "Credit Limit Validator"
   {
       procedure ValidateCreditLimit(
           CustomerNo: Code[20];
           OrderAmount: Decimal;
           Repo: Interface ICustomerRepository
       ): Boolean
       var
           Customer: Record Customer;
           Outstanding: Decimal;
       begin
           // ✅ ACTUAL LOGIC (minimal to make test pass)
           if not Repo.TryGetCustomer(CustomerNo, Customer) then
               Error('Customer not found');

           if Customer.CreditLimit = 0 then
               exit(true); // Unlimited

           Outstanding := Repo.GetOutstandingAmount(CustomerNo);
           exit(Outstanding + OrderAmount <= Customer.CreditLimit);
       end;
   }
   ```

2. **Implement real interface implementations** (repositories, services)
3. **⛔ HARD STOP - Use AskUserQuestion:**
   ```markdown
   GREEN Phase Complete: [TestName]

   Please deploy to BC and run the test. Did it PASS?

   Options:
   - Yes - Test PASSED → Continue to REFACTOR
   - No - Test still FAILS → Fix implementation
   - Deployment issue → Need troubleshooting
   ```

4. **Wait for user response** - DO NOT proceed to REFACTOR until user confirms PASS

### GREEN Phase Iterations

**If test still FAILS:**
1. Ask user for error message
2. Analyze failure
3. Fix production code
4. Re-ask AskUserQuestion for GREEN phase
5. Repeat until test PASSES

**Do NOT move to REFACTOR until test PASSES.**

---

## Phase 3: REFACTOR - Improve Quality

### Goal
Improve code quality without changing behavior.

### What to Refactor

✅ **Do refactor:**
- Extract helper procedures
- Improve variable names
- Add XML documentation
- Separate pure from impure functions
- Optimize performance
- Remove duplication

❌ **Don't refactor:**
- Behavior changes (creates new test failures)
- Add new features (requires new tests first)
- Change interfaces (breaks other tests)

### Steps

1. **Improve code quality:**
   ```al
   /// <summary>
   /// Validates that a new order does not exceed customer credit limit.
   /// </summary>
   procedure ValidateCreditLimit(...): Boolean
   var
       Customer: Record Customer;
   begin
       Customer := GetCustomer(CustomerNo, Repo);

       if IsUnlimitedCredit(Customer) then
           exit(true);

       exit(IsWithinCreditLimit(Customer, OrderAmount, Repo));
   end;

   local procedure GetCustomer(...): Record Customer
   local procedure IsUnlimitedCredit(...): Boolean
   local procedure IsWithinCreditLimit(...): Boolean
   ```

2. **⛔ HARD STOP - Use AskUserQuestion:**
   ```markdown
   REFACTOR Phase Complete

   Please run ALL tests. Do they all PASS?

   Options:
   - Yes - All tests PASS → Continue to next test
   - No - Some tests FAIL → Revert refactoring
   - Deployment issue → Need troubleshooting
   ```

3. **Wait for user response** - DO NOT continue until user confirms all tests PASS

### REFACTOR Phase Failures

**If refactoring broke tests:**
1. **Revert** refactoring immediately
2. Document: "Refactoring reverted - broke tests"
3. Skip further refactoring for this test
4. Continue to next test (refactoring is optional)

---

## TDD Log Documentation

**Every cycle MUST be documented in `.dev/03-tdd-log.md`:**

```markdown
## Feature: Credit Limit Validation

### Cycle 1: Basic Within-Limit Validation

#### RED Phase ✓
- Test: `Test_ValidateCreditLimit_WithinLimit`
- Status: ❌ FAILS as expected
- Error: "Method ValidateCreditLimit not found"
- Reason: Production code stub returns wrong value

#### GREEN Phase ✓
- Production code: Implemented Credit Limit Validator
- Interfaces: ICustomerRepository defined
- Test status: ✅ PASSES

#### REFACTOR Phase ✓
- Extracted helper procedures
- Added XML documentation
- All tests: ✅ PASS (1/1 tests)

**Cycle 1 Complete** ✓
```

---

## Task Management for TDD Cycles

**Agents should create granular tasks for each TDD cycle:**

```
TaskCreate: "TDD RED: [test name]"
  description: "Write failing test for [brief description]"
  activeForm: "Writing failing test: [test name]"

TaskCreate: "TDD GREEN: [test name]"
  description: "Implement production code to make test pass"
  activeForm: "Implementing: [production procedure name]"

TaskCreate: "TDD REFACTOR: [test name]"
  description: "Refactor code for quality"
  activeForm: "Refactoring: [test name]"

# Set dependencies
TaskUpdate: "TDD GREEN: [test]" → addBlockedBy: ["TDD RED: [test]"]
TaskUpdate: "TDD REFACTOR: [test]" → addBlockedBy: ["TDD GREEN: [test]"]
```

**Update task status at each gate:**
- After RED confirmation → mark RED complete, start GREEN
- After GREEN confirmation → mark GREEN complete, start REFACTOR
- After REFACTOR confirmation → mark REFACTOR complete, start next test

---

## Test Execution Context

**AL tests cannot be executed automatically by Claude Code:**
- BC requires server deployment to run tests
- User must manually deploy and run tests at each phase
- Each TDD cycle includes manual approval gates
- User reports PASS/FAIL back to agent

**This ensures TDD discipline is followed** - the agent cannot skip verification gates.

---

## Summary: The Hard Stops

```
1. Write test code + minimal stubs
2. ⛔ STOP → User deploys, runs test, confirms FAIL
3. Write production code
4. ⛔ STOP → User deploys, runs test, confirms PASS
5. Refactor code
6. ⛔ STOP → User runs ALL tests, confirms ALL PASS
7. Move to next test →
```

**Three hard stops per test. No exceptions.**

---

**Remember:** TDD's value comes from seeing tests fail BEFORE implementing logic. Skipping RED phase verification destroys this value.
