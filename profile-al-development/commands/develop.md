---
description: Run development phase only (implement → review → fix diagnostics) with iteration loops
allowed-tools: ["Task", "Read", "TaskCreate", "TaskUpdate", "TaskList", "TaskGet"]
---

# Development Phase

Run only the development phase with automatic iteration loops for quality.

## Usage

```
/develop
```

**Prerequisites:**
- Must have `.dev/02-solution-plan.md` from planning phase
- Should have `.dev/05-test-specification.md` for TDD workflow (recommended)

## ⚠️ CRITICAL: How This Command Works

**You are the ORCHESTRATOR. Agents are the WORKERS.**

Your job:
1. Create/check tasks (for tracking)
2. **SPAWN AGENTS** to do the work
3. Update task status

**DO NOT implement code yourself. SPAWN al-developer, code-reviewer, diagnostics-fixer.**

## Step 0: Check/Create Development Tasks

```
TaskList → Check existing tasks

# Extract feature name, create if needed:
TaskCreate: "TDD implementation for [feature]" (parent - al-developer creates child tasks)
TaskCreate: "Review code for [feature]"
TaskCreate: "Fix compilation issues for [feature]"

# Set dependencies
TaskUpdate: "TDD implementation for [feature]" → addBlockedBy: ["Design test specification for [feature]"]
TaskUpdate: "Review code for [feature]" → addBlockedBy: ["TDD implementation for [feature]"]
TaskUpdate: "Fix compilation issues for [feature]" → addBlockedBy: ["Review code for [feature]"]
```

## Workflow

**Step 1: Detect TDD vs Traditional**

```bash
ls .dev/05-test-specification.md
# If exists → Use TDD workflow (steps 2-4)
# If not → Use traditional workflow (steps 5-7)
```

**Steps 2-4: TDD Implementation (if test spec exists)**

2. TaskUpdate: "TDD implementation for [feature]" → in_progress
3. **→ SPAWN al-developer** - Tell agent the TDD discipline:

   **CRITICAL TDD WORKFLOW - FOR EACH TEST:**

   **RED Phase (Write Failing Test):**
   - Write the test code
   - Add ONLY minimal code to compile (empty procedures, default returns, no logic)
   - Create child task: "TDD RED: [test name]"
   - **STOP → Ask user to deploy and run test → Verify test FAILS**
   - Only proceed to GREEN after user confirms FAIL

   **GREEN Phase (Make It Pass):**
   - NOW implement the actual production logic
   - Create child task: "TDD GREEN: [test name]"
   - **STOP → Ask user to deploy and run test → Verify test PASSES**
   - Only proceed to REFACTOR after user confirms PASS

   **REFACTOR Phase (Improve Code):**
   - Improve code quality (extract helpers, add docs, optimize)
   - Create child task: "TDD REFACTOR: [test name]"
   - **STOP → Ask user to run ALL tests → Verify ALL tests PASS**
   - Only proceed to next test after user confirms ALL PASS

   **Rules:**
   - NEVER implement logic before user verifies test fails
   - NEVER skip verification gates
   - Document all cycles in `.dev/03-tdd-log.md`

4. TaskUpdate: "TDD implementation for [feature]" → completed

**Steps 5-7: Traditional Implementation (no test spec)**

5. TaskUpdate: "Implement [feature]" → in_progress
6. **→ SPAWN al-developer** - Tell agent:
   - "Implement from `.dev/02-solution-plan.md`"
7. TaskUpdate: "Implement [feature]" → completed

**Steps 8-11: Code Review Loop**

8. TaskUpdate: "Review code for [feature]" → in_progress
9. **→ SPAWN code-reviewer** (reviews test + production code)
10. **Evaluate:**
    - **Critical/High issues:** Create iteration task, loop back to step 3 (or 6 if traditional)
    - **Medium/Low issues:** Continue to step 11
11. TaskUpdate: "Review code for [feature]" → completed

**Steps 12-15: Compilation Loop**

12. TaskUpdate: "Fix compilation issues for [feature]" → in_progress
13. **→ SPAWN diagnostics-fixer** (auto-fixes simple issues)
14. **Evaluate:**
    - **Complex errors (3+ or logic):** Create iteration task, loop back to step 3 (or 6 if traditional)
    - **Minor/no errors:** Done
15. TaskUpdate: "Fix compilation issues for [feature]" → completed

## Iteration Logic

**Code review triggers iteration:** Critical/High issues (security, performance, DRY, missing functionality)
**Diagnostics triggers iteration:** 3+ complex errors or logic issues
**No iteration:** Only Medium/Low issues or 1-2 trivial errors

## Output Files

- `.dev/03-tdd-log.md` (if TDD) or production code
- `.dev/03-code-review.md`
- `.dev/04-diagnostics.md`
- `.dev/session-log.md`

## When to Use

- Have `.dev/02-solution-plan.md` ready
- Continuing from `/plan`
- Want automatic quality checks with iteration loops

---

**Remember:** Spawn agents. Don't implement code yourself.
