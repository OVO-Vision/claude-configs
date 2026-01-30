---
description: Quick bug fix workflow - analyze, fix, verify (no planning phase)
allowed-tools: ["Task", "Read", "Grep", "Glob"]
---

# Quick Fix

Fast workflow for small bug fixes and error messages. Skips planning phase.

## Usage

```
/fix "Error: Customer email validation fails for john.doe@example.com"
```

```
/fix "Compilation error AL0132 at line 45 in CustomerProcessor.al"
```

```
/fix "Runtime error when posting sales order with credit limit exceeded"
```

## What It Does

Lightweight 3-step workflow:
1. **Locate the issue** - Find relevant code
2. **Fix the code** - Apply the fix directly
3. **Verify** - Run diagnostics to confirm

**No planning, no testing phase** - just fix and verify.

## Workflow

### Step 1: Analyze & Locate

Use grep/glob to find the relevant code:
- Search for error message text
- Search for file names mentioned in error
- Search for function/procedure names
- Identify the specific issue

Respond with:
```
Found issue in: src/codeunits/CustomerProcessor.Codeunit.al:411
Issue: Email validation logic rejects valid emails with dots
Root cause: StrPos check is too restrictive
```

### Step 1.5: Write Failing Test (Recommended for Non-Trivial Bugs)

For bugs beyond simple compiler errors, write a test that reproduces the bug before fixing it:

1. **Write a test** that exercises the buggy behavior
2. **Expect it to FAIL** — confirming the bug exists
3. **If the test PASSES immediately** — your diagnosis is wrong, re-investigate

**Why:** A failing test proves you understand the bug. If the test passes, you're looking at the wrong code.

**Skip this step for:**
- Compiler errors (AL0132, etc.) — the compiler IS the failing test
- Typos, missing properties, syntax issues
- Obvious one-line fixes

**Use this step for:**
- Logic bugs (wrong calculation, wrong condition)
- Validation errors (rejects valid input, accepts invalid input)
- Integration issues (events not firing, wrong data flow)

### Step 2: Fix Code

Directly edit the file with the fix:
- Use Edit tool to apply the fix
- Keep changes minimal and focused
- Add comment explaining the fix if not obvious

Respond with:
```
Fixed: Removed problematic dot position check
Changed: Lines 411-417
File: src/codeunits/CustomerProcessor.Codeunit.al
```

### Step 3: Verify with Diagnostics

Spawn diagnostics-fixer to verify:
- Compiles without errors
- No new warnings introduced
- Fix doesn't break anything else

**If diagnostics-fixer finds complex issues:**
- May need to iterate back to fix additional errors
- Follows same iteration rules as `/develop` command

Respond with:
```
Verification complete → .dev/05-diagnostics.md
- Compilation: ✓ Success
- New errors: 0
- New warnings: 0

Fix is ready to commit.
```

Or if issues remain:
```
Verification found issues → .dev/05-diagnostics.md
- Initial fix applied
- Diagnostics found N additional errors
- Applying additional fixes...
[Iterates to resolve]
```

## When to Use /fix

**✓ Good for:**
- Compiler errors (AL0132, AL0254, etc.)
- Runtime errors with stack traces
- Specific validation errors
- Small logic bugs
- Quick corrections

**✗ Not for:**
- New features (use `/plan` or `/dev-cycle`)
- Architectural changes (use `/plan`)
- Multiple related issues (use `/develop`)
- Unclear requirements (use `/plan`)

## Output

Quick fixes write minimal documentation:
- `.dev/quickfix-[timestamp].md` - What was fixed
- `.dev/05-diagnostics.md` - Verification results

No full planning or testing docs.

## Example Session

```
User: /fix "Email validation rejects john.doe@example.com"

Claude: Searching for email validation code...

Found issue:
- File: src/codeunits/CustomerProcessor.Codeunit.al
- Line: 411
- Function: ValidateEmailFormat
- Problem: Dot position check rejects valid emails

Applying fix... [uses Edit tool]

Fixed: Removed overly restrictive StrPos check
- Before: Rejected any email with dot in username
- After: Only validates @ and . presence

Verifying... [spawns diagnostics-fixer]

Verification complete:
- Compilation: ✓ Success
- No new errors
- Fix is ready to commit

Quick fix complete! Git diff:
  src/codeunits/CustomerProcessor.Codeunit.al | 6 +-----
  1 file changed, 1 insertion(+), 5 deletions(-)
```

## Approval Gate

After showing the fix, ask:
```
Review the fix above.

Options:
- Apply and verify (spawn diagnostics-fixer)
- Revise fix (I'll provide feedback)
- Cancel
```

Only proceed to verification if user approves the fix.

## Comparison to Full Workflow

| Feature | /fix (Quick) | /dev-cycle (Full) |
|---------|-------------|-------------------|
| Planning phase | ❌ Skip | ✓ Full |
| Requirements doc | ❌ No | ✓ Yes |
| Design doc | ❌ No | ✓ Yes |
| Implementation plan | ❌ No | ✓ Yes |
| Code changes | ✓ Direct | ✓ Planned |
| Code review | ❌ Skip | ✓ Full |
| Diagnostics | ✓ Basic | ✓ Full |
| Testing phase | ❌ Skip | ✓ Full |
| Duration | ~5 min | ~30-60 min |

## Documentation Generated

Minimal docs in `.dev/`:
```markdown
# Quick Fix: [Issue]

**Date:** 2026-01-14 16:30
**Issue:** Email validation rejects valid emails

## Problem
Customer emails like john.doe@example.com were rejected

## Root Cause
Line 411: StrPos check was overly restrictive

## Fix Applied
Removed dot position validation, kept essential checks

## Files Changed
- src/codeunits/CustomerProcessor.Codeunit.al (lines 411-417)

## Verification
- Compilation: ✓ Success
- New errors: 0
- Ready for commit
```

## Resuming to Full Workflow

If quick fix reveals larger issues:
```
User: "Actually this needs more work"
Claude: "Let's switch to full workflow:"

/plan "Redesign email validation with comprehensive rules"
```

## Tips for Good Fix Descriptions

**✓ Good descriptions:**
- "Error AL0132: Undeclared variable 'TotalAmount' in line 245"
- "Customer posting fails with 'Credit limit exceeded' even when under limit"
- "Email validation rejects valid emails with dots before @"

**✗ Vague descriptions:**
- "Something's broken"
- "It doesn't work"
- "Fix the customer code"

**Better with context:**
- Include error codes (AL0132, AA0001, etc.)
- Include file names and line numbers
- Include error messages
- Include what you were trying to do

---

**Remember:** /fix is for surgical strikes on specific bugs. For anything complex, use /dev-cycle.
