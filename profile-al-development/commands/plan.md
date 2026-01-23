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

Runs the two planning agents with approval gates:
1. Requirements engineering
2. Solution planning (architecture + implementation)

Creates Tasks for tracking (can be resumed or continued with `/develop`).

## Step 0: Create Tasks

**At start, create planning tasks:**

```
TaskCreate: "Requirements Analysis"
  - description: "Extract requirements for: [user request]"
  - activeForm: "Analyzing requirements"

TaskCreate: "Solution Planning"
  - description: "Design architecture and implementation plan"
  - activeForm: "Designing solution"

TaskUpdate: "Solution Planning" → addBlockedBy: ["Requirements Analysis"]
```

## Workflow with Approval Gates

### Step 1: Requirements Analysis

1. **TaskUpdate:** "Requirements Analysis" → status: "in_progress"
2. **Spawn requirements-engineer** with user's request
3. **TaskUpdate:** "Requirements Analysis" → status: "completed"
4. **Show `.dev/01-requirements.md` summary**
5. **🛑 APPROVAL GATE:** Ask user:
   - ✅ Approve and continue to solution plan
   - 🔄 Refine requirements
   - ❌ Stop here

### Step 2: Solution Planning

6. **TaskUpdate:** "Solution Planning" → status: "in_progress"
7. **Spawn solution-planner** (reads requirements, uses MCP tools)
8. **TaskUpdate:** "Solution Planning" → status: "completed"
9. **Show `.dev/02-solution-plan.md` summary**
10. **✅ Done!** Planning complete

**Note:** Tasks remain for `/develop` or `/dev-cycle` to continue.

## Critical Rules

### ALWAYS Stop and Ask

After requirements:
1. **Read the output file**
2. **Summarize key points** (3-5 bullets)
3. **Use AskUserQuestion** to get approval
4. **Only proceed if approved**

### Approval Question Format

```json
{
  "questions": [{
    "question": "Review .dev/01-requirements.md. Ready to proceed to solution planning?",
    "header": "Next Step",
    "multiSelect": false,
    "options": [
      {
        "label": "Approve - Continue",
        "description": "Looks good, proceed to solution planning"
      },
      {
        "label": "Refine - Need changes",
        "description": "I'll provide feedback to improve"
      },
      {
        "label": "Stop here",
        "description": "End the planning workflow"
      }
    ]
  }]
}
```

## Output

Planning documents in `.dev/`:
- `.dev/01-requirements.md`
- `.dev/02-solution-plan.md`
- `.dev/session-log.md`

## After Planning

User can then:
- Review solution plan carefully
- Proceed to `/develop` to implement the plan
- Run `/test` after development
- Or start over with feedback

## When to Use

- Want to plan before committing to implementation
- Need to review and approve approach first
- Creating documentation for approval
- Exploring different design options

## Example

```
User: /plan "Add email validation"

Claude: Spawning requirements-engineer...
[Agent runs]

Claude: Requirements complete → .dev/01-requirements.md

Summary:
- Email format validation
- Block invalid on save
- User-friendly errors

[AskUserQuestion: Approve / Refine / Stop]

User: [Approves]

Claude: Spawning solution-planner...
[Agent runs]

Claude: Solution plan complete → .dev/02-solution-plan.md

Architecture summary:
- Table extension approach
- Validation codeunit
- Event subscriber pattern

Implementation summary:
- 4 files to create
- 3 implementation phases
- Estimated 60 minutes

Ready to implement? Run: /develop
```

---

**Remember:** Stop after requirements for user approval before continuing to solution planning.
