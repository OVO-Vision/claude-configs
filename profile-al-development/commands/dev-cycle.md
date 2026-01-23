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

## Step 0: Create Task Structure

**At workflow start, create tasks with dependencies:**

```
TaskCreate: "Requirements Analysis"
  - description: "Extract requirements for: [user request]"
  - activeForm: "Analyzing requirements"

TaskCreate: "Solution Planning"
  - description: "Design architecture and implementation plan"
  - activeForm: "Designing solution"

TaskCreate: "Development"
  - description: "Implement AL code per solution plan"
  - activeForm: "Implementing code"

TaskCreate: "Code Review"
  - description: "Review code for quality and standards"
  - activeForm: "Reviewing code"

TaskCreate: "Diagnostics"
  - description: "Fix compilation issues"
  - activeForm: "Fixing diagnostics"

TaskCreate: "Testing"
  - description: "Create test codeunits"
  - activeForm: "Writing tests"

TaskCreate: "Test Review"
  - description: "Review test coverage and quality"
  - activeForm: "Reviewing tests"

TaskCreate: "Documentation"
  - description: "Generate feature and API documentation"
  - activeForm: "Writing documentation"
```

**Then set up dependencies:**
```
TaskUpdate: "Solution Planning" → addBlockedBy: ["Requirements Analysis"]
TaskUpdate: "Development" → addBlockedBy: ["Solution Planning"]
TaskUpdate: "Code Review" → addBlockedBy: ["Development"]
TaskUpdate: "Diagnostics" → addBlockedBy: ["Code Review"]
TaskUpdate: "Testing" → addBlockedBy: ["Diagnostics"]
TaskUpdate: "Test Review" → addBlockedBy: ["Testing"]
TaskUpdate: "Documentation" → addBlockedBy: ["Test Review"]
```

## Checking for Existing Tasks/Docs

**Before creating tasks, check if resuming:**

```
TaskList → Check for existing tasks for this feature
  - If tasks exist with some completed → Resume from first pending task
  - If no tasks → Create fresh task structure (Step 0)

Also check:
ls .dev/01-requirements.md .dev/02-solution-plan.md
  - If planning docs exist → Ask: Resume or start fresh?
```

## First: Check for Existing Planning

**Before starting, check if planning docs exist:**

```bash
# Check for existing planning documents
ls .dev/01-requirements.md .dev/02-solution-plan.md
```

### If Planning Docs Exist (from previous /plan)

**Skip to Phase 3: Development**
- Show summary: "Found existing plan, continuing from development phase"
- Show brief summary of existing 02-solution-plan.md
- Ask: "Use existing plan or start fresh?"
  - Use existing → Jump to Phase 3
  - Start fresh → Continue with Phase 1

### If No Planning Docs

**Start from Phase 1**

## Workflow with Approval Gates

### Phase 1: Requirements Analysis

1. **TaskUpdate:** "Requirements Analysis" → status: "in_progress"
2. **Spawn requirements-engineer** with user's request
3. **TaskUpdate:** "Requirements Analysis" → status: "completed"
4. **Show `.dev/01-requirements.md` summary**
5. **🛑 APPROVAL GATE:** Ask user:
   - ✅ Approve and continue to solution planning
   - 🔄 Refine requirements (provide feedback, re-run agent)
   - ❌ Stop here

### Phase 2: Solution Planning

6. **TaskUpdate:** "Solution Planning" → status: "in_progress"
7. **Spawn solution-planner** (reads requirements, uses MCP tools)
8. **TaskUpdate:** "Solution Planning" → status: "completed"
9. **Show `.dev/02-solution-plan.md` summary**
10. **🛑 APPROVAL GATE:** Ask user:
    - ✅ Approve and start implementation
    - 🔄 Revise plan (provide feedback, re-run agent)
    - ❌ Stop here

### Phase 3: Development (Iterative)

**Initial Implementation:**
11. **TaskUpdate:** "Development" → status: "in_progress"
12. **Spawn al-developer** (reads solution plan)
13. **TaskUpdate:** "Development" → status: "completed"

**Code Review Loop:**
14. **TaskUpdate:** "Code Review" → status: "in_progress"
15. **Spawn code-reviewer** (reviews code)
16. **Evaluate findings:**
    - **If Critical/High issues:**
      - TaskUpdate: "Code Review" → status: "completed", metadata: {passed: false}
      - TaskUpdate: "Development" → status: "in_progress" (reset for fixes)
      - ITERATE back to al-developer → Go to step 12
    - **If only Medium/Low issues:**
      - TaskUpdate: "Code Review" → status: "completed", metadata: {passed: true}
      - Continue to step 17

**Compilation Loop:**
17. **TaskUpdate:** "Diagnostics" → status: "in_progress"
18. **Spawn diagnostics-fixer** (fixes auto-fixable issues)
19. **Evaluate compilation:**
    - **If complex errors remain (3+ or logic issues):**
      - TaskUpdate: "Diagnostics" → status: "completed", metadata: {clean: false}
      - ITERATE back to al-developer → Go to step 11
    - **If minor issues or clean:**
      - TaskUpdate: "Diagnostics" → status: "completed", metadata: {clean: true}
      - Continue to step 20

**Approval:**
20. **Show `.dev/03-code-review.md` + `.dev/04-diagnostics.md` summary**
21. **🛑 APPROVAL GATE:** Ask user:
    - ✅ Approve and continue to testing
    - 🔄 Manual adjustments needed (provide feedback)
    - ❌ Stop here

### Phase 4: Testing (Automated)

22. **TaskUpdate:** "Testing" → status: "in_progress"
23. **Spawn test-engineer** (creates tests)
24. **TaskUpdate:** "Testing" → status: "completed"
25. **TaskUpdate:** "Test Review" → status: "in_progress"
26. **Spawn test-reviewer** (reviews tests)
27. **TaskUpdate:** "Test Review" → status: "completed"
28. **Show `.dev/06-test-review.md` summary**

### Phase 5: Documentation (Automated)

29. **TaskUpdate:** "Documentation" → status: "in_progress"
30. **Spawn docs-writer** (generates documentation)
31. **TaskUpdate:** "Documentation" → status: "completed"
32. **Show documentation summary**
33. **✅ Done!** All tasks completed.

## Critical Rules

### ALWAYS Stop and Ask

After each major phase output:
1. **Read the output file**
2. **Summarize key points** (3-5 bullets)
3. **Use AskUserQuestion** to get approval
4. **Only proceed if approved**

### Never Auto-Continue

❌ **DON'T:**
```
Spawning requirements-engineer...
Spawning bc-solution-designer...  ← NO! Wait for approval!
```

✅ **DO:**
```
Requirements analysis complete → .dev/01-requirements.md

Key findings:
- 5 functional requirements
- 2 BC integration points
- Credit limit validation on sales posting

[Use AskUserQuestion: Approve / Refine / Stop]

[Wait for user response before continuing]
```

## Approval Question Format

Use AskUserQuestion with these options:

```markdown
{
  "questions": [{
    "question": "Review .dev/01-requirements.md. Ready to proceed to design phase?",
    "header": "Next Step",
    "multiSelect": false,
    "options": [
      {
        "label": "Approve - Continue to design",
        "description": "Requirements look good, proceed to BC solution design"
      },
      {
        "label": "Refine - Need changes",
        "description": "I'll provide feedback to improve the requirements"
      },
      {
        "label": "Stop here",
        "description": "End the workflow at this phase"
      }
    ]
  }]
}
```

## Handling User Feedback

### If "Approve"
- Continue to next phase

### If "Refine"
- Ask: "What changes do you need?"
- Re-spawn same agent with user feedback
- Show updated output
- Ask for approval again

### If "Stop"
- End workflow
- Summarize what was completed
- User can resume later with individual commands

## Output

All phases write to `.dev/` directory:
- `.dev/01-requirements.md`
- `.dev/02-solution-plan.md`
- `.dev/03-code-review.md`
- `.dev/04-diagnostics.md`
- `.dev/05-test-plan.md`
- `.dev/06-test-review.md`
- `.dev/session-log.md`

## Expected Duration

- **Requirements:** ~2 min agent + approval
- **Solution planning:** ~4 min agent + approval (combines design + implementation planning)
- **Development:** ~15-30 min (includes iteration loops if needed)
  - al-developer: ~5-10 min
  - code-reviewer: ~3-5 min
  - If iteration needed: +5-10 min per loop
  - diagnostics-fixer: ~2-5 min
- **Testing:** ~10-15 min (all agents)

**With approvals:** 35-65 minutes total (depending on iterations)

## When to Use

- Starting a new feature
- Want to review each phase before proceeding
- Need complete documentation
- Collaborative development

## When NOT to Use

- Quick bug fixes (too heavy)
- You trust agents fully (use faster workflow)
- Already have requirements/design (start at `/develop`)

## Two Common Workflows

### Workflow A: All-in-One
```bash
# Run everything in one go
/dev-cycle "Add feature X"
# Goes through all phases with approval gates
```

### Workflow B: Plan First, Implement Later (Recommended)
```bash
# Session 1: Planning (review carefully)
/plan "Add feature X"
# Creates planning docs, stops after 02-solution-plan.md
# Review at your leisure

# Session 2: Implementation (same day or later)
/dev-cycle "Add feature X"
# Detects existing plan
# Asks: "Use existing plan or start fresh?"
# Choose: Use existing → skips to development

# OR just:
/develop
# Uses existing 02-solution-plan.md

# Session 3: Testing
/test
# Creates comprehensive tests
```

**Workflow B is recommended** because:
- Planning can be reviewed carefully offline
- Implementation can happen in a separate session
- No rush to approve each phase immediately
- Can share plan with team before implementing

## Resuming After Stop

If user stops at any phase, they can resume:

```
Stopped after requirements:
→ /plan "Continue with [feedback]"

Stopped after solution planning:
→ /develop  (or /dev-cycle with existing plan)

Stopped after development:
→ /test
```

## Example Session

```
User: /dev-cycle "Add customer email validation"

Claude: Spawning requirements-engineer...
[Agent runs]

Claude: Requirements analysis complete → .dev/01-requirements.md

Summary:
- Validate email format on customer record
- Block invalid emails on save
- Support multiple email formats
- Show user-friendly error messages

[AskUserQuestion: Approve / Refine / Stop]

User: [Selects "Approve - Continue to solution planning"]

Claude: Spawning solution-planner...
[Agent runs]

Claude: Solution plan complete → .dev/02-solution-plan.md

Architecture summary:
- Table extension on Customer table
- Separate validation codeunit
- Event subscriber for additional checks

Implementation summary:
- 4 files to create
- 3 implementation phases
- Estimated 60 minutes

[AskUserQuestion: Approve / Refine / Start Implementation / Stop]

User: [Selects "Refine - Need changes"]

Claude: What changes would you like to the solution plan?

User: "Use FlowFields instead of calculated fields for better performance"

Claude: Re-running solution-planner with your feedback...
[Agent runs with constraint]

Claude: Solution plan revised → .dev/02-solution-plan.md

Updated:
- Added FlowFields for real-time calculations
- Updated implementation plan for FlowField setup
- Performance improved with database-level calculation

[AskUserQuestion: Approve / Refine / Start Implementation / Stop]

User: [Selects "Approve - Start implementation"]

[Workflow continues to development phase...]
```

## Implementation Notes

### Reading Output Files

After each agent, **always read the output file** and show key points:

```
Read .dev/01-requirements.md
Extract:
- Number of requirements
- Key functionality
- Integration points
- Assumptions

Read .dev/02-solution-plan.md
Extract:
- Architecture approach
- Number of objects/files
- Implementation phases
- Estimated effort
```

### Concise Summaries

Show 3-5 bullet points, not full file contents:

✅ **Good:**
```
Key findings:
- 3 table extensions needed
- Event subscriber for posting validation
- 2 page extensions for UI
```

❌ **Too verbose:**
```
[dumps entire 200-line file contents]
```

### Trust the User

If user approves, don't second-guess:
- Don't say "Are you sure?"
- Just continue to next phase

---

**Remember:** This is a collaborative workflow. The user reviews and approves each major phase before proceeding.
