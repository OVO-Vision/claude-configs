---
description: Run planning phase only (requirements → solution plan) with approval gates
allowed-tools: ["Task", "Read", "AskUserQuestion", "TaskCreate", "TaskUpdate", "TaskList"]
---

# Planning Phase

Run only the planning phase with approval gates between each step.

## Usage

```
/plan "Add customer credit limit validation feature"
```

## What It Does

Runs the planning AND test specification phases with approval gates:
1. Requirements engineering
2. Solution planning (architecture + implementation + testability)
3. Test specification (test strategy before implementation)

Creates Tasks for tracking (can be resumed or continued with `/develop`).

## ⚠️ CRITICAL: How This Command Works

**You are the ORCHESTRATOR. Agents are the WORKERS.**

Your job:
1. Create tasks (for tracking)
2. **SPAWN AGENTS** to do the work
3. Update task status and get approvals

**DO NOT write requirements or plans yourself. SPAWN requirements-engineer, solution-planner, plan-reviewer, test-engineer.**

## Step 0: Create Planning Tasks

Extract feature name from request (e.g., "Add credit limit validation" → "credit limit validation").

```
TaskCreate: "Analyze requirements for [feature]"
TaskCreate: "Design architecture for [feature]"
TaskCreate: "Review solution plan for [feature]"
TaskCreate: "Verify plan assumptions for [feature]"
TaskCreate: "Check test infrastructure"
TaskCreate: "Design test specification for [feature]"

# Set dependencies
TaskUpdate: "Design architecture for [feature]" → addBlockedBy: ["Analyze requirements for [feature]"]
TaskUpdate: "Review solution plan for [feature]" → addBlockedBy: ["Design architecture for [feature]"]
TaskUpdate: "Verify plan assumptions for [feature]" → addBlockedBy: ["Review solution plan for [feature]"]
TaskUpdate: "Check test infrastructure" → addBlockedBy: ["Verify plan assumptions for [feature]"]
TaskUpdate: "Design test specification for [feature]" → addBlockedBy: ["Check test infrastructure"]
```

## Workflow

### Step 1: Requirements Analysis

1. TaskUpdate: "Analyze requirements for [feature]" → in_progress
2. **→ SPAWN requirements-engineer** with user's request
3. TaskUpdate: "Analyze requirements for [feature]" → completed
4. Read and show `.dev/01-requirements.md` summary (3-5 bullets)
5. **🛑 APPROVAL GATE:** Use AskUserQuestion: Approve / Refine / Stop

### Step 2: Solution Planning

6. TaskUpdate: "Design architecture for [feature]" → in_progress
7. **→ SPAWN solution-planner** (reads requirements, uses MCP tools)
8. TaskUpdate: "Design architecture for [feature]" → completed
9. TaskUpdate: "Review solution plan for [feature]" → in_progress
10. **→ SPAWN plan-reviewer** (adversarial review)
11. **Evaluate** `.dev/02a-plan-review.md`:
    - **CHANGES REQUIRED:** Loop back to solution-planner (max 3 cycles)
    - **APPROVED:** Continue
12. TaskUpdate: "Review solution plan for [feature]" → completed
13. TaskUpdate: "Verify plan assumptions for [feature]" → in_progress
14. **Verify [VERIFY]-tagged assumptions** using Glob/Grep/MCP
    - If wrong: loop back to solution-planner
15. TaskUpdate: "Verify plan assumptions for [feature]" → completed
16. Read and show `.dev/02-solution-plan.md` summary (3-5 bullets)
17. **🛑 APPROVAL GATE:** Use AskUserQuestion: Approve / Refine / Stop

### Step 3: Test Specification

18. TaskUpdate: "Check test infrastructure" → in_progress
19. **Check test infrastructure** (grep for test runner and dependencies)
20. TaskUpdate: "Check test infrastructure" → completed
21. TaskUpdate: "Design test specification for [feature]" → in_progress
22. **→ SPAWN test-engineer** (designs test specifications from plan)
23. TaskUpdate: "Design test specification for [feature]" → completed
24. Read and show `.dev/05-test-specification.md` summary (3-5 bullets)
25. **✅ Done!** Planning complete. User can now run `/develop` for TDD implementation.

## Critical Rules

**ALWAYS stop and ask after requirements and after planning:**
1. Read the output file
2. Show summary (3-5 bullets)
3. Use AskUserQuestion: Approve / Refine / Stop
4. WAIT for user response

If user selects "Refine": Re-spawn agent with feedback.

## Output Files

- `.dev/01-requirements.md`
- `.dev/02-solution-plan.md`
- `.dev/02a-plan-review.md`
- `.dev/05-test-specification.md`
- `.dev/session-log.md`

## After Planning

User can run `/develop` to implement with TDD workflow.

---

**Remember:** Spawn agents. Don't write requirements or plans yourself.
