---
description: Run complete development lifecycle from requirements to testing with approval gates
allowed-tools: ["Task", "Read", "AskUserQuestion", "TaskCreate", "TaskUpdate", "TaskList", "TaskGet"]
---

# Full Development Cycle

Run the complete AL development lifecycle with approval gates between phases.

## Usage

```
/dev-cycle "Add customer credit limit validation feature"
```

## What It Does

Orchestrates all development phases with **mandatory approval gates** between major steps.

**Task Coordination:** Uses native Tasks for tracking progress, dependencies, and multi-session coordination.

**Smart Resume:** If tasks exist from previous session, resume from where you left off.

## ⚠️ CRITICAL: How This Command Works

**You are the ORCHESTRATOR. Agents are the WORKERS.**

Your job:
1. Create tasks (for tracking progress)
2. **SPAWN AGENTS** to do the actual work
3. Update task status as agents complete work
4. Show summaries and get user approval

**DO NOT:**
- Implement code yourself
- Write requirements yourself
- Do any of the actual work

**DO:**
- Spawn specialized agents for each phase
- Update tasks before/after each agent
- Present results and ask for approval

## Step 0: Create Tasks

Extract feature name from request (e.g., "Add credit limit validation" → "credit limit validation").

Create feature-specific tasks:

```
TaskCreate: "Analyze requirements for [feature]"
TaskCreate: "Design architecture for [feature]"
TaskCreate: "Review solution plan for [feature]"
TaskCreate: "Verify plan assumptions for [feature]"
TaskCreate: "Check test infrastructure"
TaskCreate: "Design test specification for [feature]"
TaskCreate: "TDD implementation for [feature]" (parent - al-developer creates child tasks)
TaskCreate: "Review code for [feature]"
TaskCreate: "Fix compilation issues for [feature]"
TaskCreate: "Validate test coverage for [feature]"
TaskCreate: "Generate documentation for [feature]"

# Set dependencies
TaskUpdate: "Design architecture for [feature]" → addBlockedBy: ["Analyze requirements for [feature]"]
TaskUpdate: "Review solution plan for [feature]" → addBlockedBy: ["Design architecture for [feature]"]
TaskUpdate: "Verify plan assumptions for [feature]" → addBlockedBy: ["Review solution plan for [feature]"]
TaskUpdate: "Check test infrastructure" → addBlockedBy: ["Verify plan assumptions for [feature]"]
TaskUpdate: "Design test specification for [feature]" → addBlockedBy: ["Check test infrastructure"]
TaskUpdate: "TDD implementation for [feature]" → addBlockedBy: ["Design test specification for [feature]"]
TaskUpdate: "Review code for [feature]" → addBlockedBy: ["TDD implementation for [feature]"]
TaskUpdate: "Fix compilation issues for [feature]" → addBlockedBy: ["Review code for [feature]"]
TaskUpdate: "Validate test coverage for [feature]" → addBlockedBy: ["Fix compilation issues for [feature]"]
TaskUpdate: "Generate documentation for [feature]" → addBlockedBy: ["Validate test coverage for [feature]"]
```

**Note:** al-developer creates child tasks (`"TDD RED: [test]"`, `"TDD GREEN: [test]"`, `"TDD REFACTOR: [test]"`) dynamically during TDD cycles.

## Check for Existing Work

```bash
# Check tasks and planning docs
TaskList
ls .dev/01-requirements.md .dev/02-solution-plan.md
```

**If planning docs exist:** Ask user: "Use existing plan (→ Phase 3) or start fresh?"
**If tasks exist:** Resume from first pending task
**Otherwise:** Create fresh tasks (Step 0) and start from Phase 1

## Workflow with Approval Gates

### Phase 1: Requirements Analysis

1. TaskUpdate: "Analyze requirements for [feature]" → in_progress
2. **→ SPAWN requirements-engineer** with user's request
3. TaskUpdate: "Analyze requirements for [feature]" → completed
4. Read and show `.dev/01-requirements.md` summary (3-5 bullets)
5. **🛑 APPROVAL GATE:** Use AskUserQuestion:
   - ✅ Approve → Continue to Phase 2
   - 🔄 Refine → Re-spawn agent with feedback
   - ❌ Stop

### Phase 2: Solution Planning

6. TaskUpdate: "Design architecture for [feature]" → in_progress
7. **→ SPAWN solution-planner** (reads requirements, uses MCP tools)
8. TaskUpdate: "Design architecture for [feature]" → completed
9. TaskUpdate: "Review solution plan for [feature]" → in_progress
10. **→ SPAWN plan-reviewer** (adversarial review)
11. **Evaluate** `.dev/02a-plan-review.md`:
    - **CHANGES REQUIRED:** Loop back to solution-planner (max 3 cycles)
    - **APPROVED:** Continue to step 12
12. TaskUpdate: "Review solution plan for [feature]" → completed
13. TaskUpdate: "Verify plan assumptions for [feature]" → in_progress
14. **Verify [VERIFY]-tagged assumptions** using Glob/Grep/MCP
    - If wrong: loop back to solution-planner with corrections
15. TaskUpdate: "Verify plan assumptions for [feature]" → completed
16. Read and show `.dev/02-solution-plan.md` summary (3-5 bullets)
17. **🛑 APPROVAL GATE:** Use AskUserQuestion:
    - ✅ Approve → Continue to Phase 2.5
    - 🔄 Revise → Re-spawn agent with feedback
    - ❌ Stop

### Phase 2.5: Test Specification

18. TaskUpdate: "Check test infrastructure" → in_progress
19. **Check test infrastructure:** grep for `"isTest": true` and test dependencies
    - If missing: AskUserQuestion to set up, skip, or specify path
20. TaskUpdate: "Check test infrastructure" → completed
21. TaskUpdate: "Design test specification for [feature]" → in_progress
22. **→ SPAWN test-engineer** (designs test specifications from plan)
23. TaskUpdate: "Design test specification for [feature]" → completed
24. Read and show `.dev/05-test-specification.md` summary (3-5 bullets)
25. **🛑 APPROVAL GATE:** Use AskUserQuestion:
    - ✅ Approve → Continue to Phase 3 (TDD)
    - 🔄 Revise → Re-spawn agent with feedback
    - ❌ Stop

### Phase 3: TDD Development (Iterative)

**TDD Implementation:**
26. TaskUpdate: "TDD implementation for [feature]" → in_progress
27. **→ SPAWN al-developer** - Follow TDD discipline from `tdd-workflow.md`:
    - RED → GREEN → REFACTOR cycle for each test
    - Create child tasks for each phase (TDD RED, TDD GREEN, TDD REFACTOR)
    - Hard stops at each phase for user verification
    - ⛔ NEVER implement logic before user confirms test FAILS
    - ⛔ NEVER skip verification gates
    - Document cycles in `.dev/03-tdd-log.md`

    **See `tdd-workflow.md` for complete TDD standards.**

28. TaskUpdate: "TDD implementation for [feature]" → completed

**Code Review Loop:**
29. TaskUpdate: "Review code for [feature]" → in_progress
30. **→ SPAWN code-reviewer** (reviews test + production code)
31. **Evaluate** `.dev/03-code-review.md`:
    - **CHANGES REQUIRED:** Create iteration task, loop back to al-developer (step 27)
    - **Stall check:** After 2 iterations with no progress, escalate to user
    - **APPROVED:** Continue to step 32
32. TaskUpdate: "Review code for [feature]" → completed

**Compilation Loop:**
33. TaskUpdate: "Fix compilation issues for [feature]" → in_progress
34. **→ SPAWN diagnostics-fixer** (auto-fixes simple issues)
35. **Evaluate compilation:**
    - **Complex errors (3+ or logic issues):** Create iteration task, loop back to al-developer (step 27)
    - **Stall check:** After 2 iterations with no progress, escalate to user
    - **Minor/no errors:** Continue to step 36
36. TaskUpdate: "Fix compilation issues for [feature]" → completed

**Development Approval Gate:**
37. Read and show summaries: `.dev/03-tdd-log.md`, `.dev/03-code-review.md`, `.dev/04-diagnostics.md`
38. **🛑 APPROVAL GATE:** Use AskUserQuestion:
    - ✅ Approve → Continue to Phase 4
    - 🔄 Manual fixes → Provide feedback
    - ❌ Stop

### Phase 4: Test Validation

39. TaskUpdate: "Validate test coverage for [feature]" → in_progress
40. **→ SPAWN test-reviewer** (validates coverage matches test specification)
41. TaskUpdate: "Validate test coverage for [feature]" → completed
42. Read and show `.dev/06-test-review.md` summary

### Phase 5: Documentation

43. TaskUpdate: "Generate documentation for [feature]" → in_progress
44. **→ SPAWN docs-writer** (generates feature/API docs)
45. TaskUpdate: "Generate documentation for [feature]" → completed
46. Show documentation summary
47. **✅ Done!** All tasks completed.

## Critical Rules

**ALWAYS stop and ask after each phase:**
1. Read the output file
2. Show summary (3-5 bullets)
3. Use AskUserQuestion with options: Approve / Refine / Stop
4. WAIT for user response

**NEVER auto-continue to next phase without approval.**

If user selects "Refine": Re-spawn agent with feedback, then ask for approval again.

## Output Files

Agents write to `.dev/` directory:
- `.dev/01-requirements.md` - Requirements
- `.dev/02-solution-plan.md` - Solution architecture
- `.dev/02a-plan-review.md` - Plan review
- `.dev/05-test-specification.md` - Test specs
- `.dev/03-tdd-log.md` - TDD cycles
- `.dev/03-code-review.md` - Code review
- `.dev/04-diagnostics.md` - Compilation
- `.dev/06-test-review.md` - Test validation
- `.dev/session-log.md` - Activity log

## When to Use

- Starting a new feature with full lifecycle
- Want approval gates between phases
- Need complete documentation

**Alternative:** Use `/plan` first (review offline), then `/develop` later (recommended for large features).

---

**Remember:** You spawn agents. Agents do the work. You just orchestrate and get approvals.
