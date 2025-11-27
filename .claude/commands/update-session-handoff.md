---
description: Auto-capture session work and update all context files - no questions asked
tags: [project, gitignored]
---

You are tasked with automatically capturing everything that happened during the current Claude Code session and updating all context preservation files.

## Your Mission

Analyze the ENTIRE conversation history from this session and automatically extract:
1. What tasks were worked on
2. Files created/modified
3. Commands that were executed
4. Technical decisions made
5. Issues found or fixed
6. What should happen next

Then update ALL context files without asking the user anything.

## Step 1: Analyze Current Session

Review the conversation history and extract:

### A. Main Tasks/Work Done
- What was the user trying to accomplish?
- What features were built?
- What bugs were fixed?
- What was refactored?

Look for:
- User's initial requests
- Features discussed
- Problems solved
- Code written/modified

### B. Files Changed
Auto-detect using:
```bash
git status --short
git diff --name-only
git diff --staged --name-only
```

Categorize files by:
- Created (new files)
- Modified (existing files changed)
- Deleted (if any)

For each file, infer the purpose from:
- File name
- Conversation context
- Tool calls (Read/Write/Edit)

### C. Commands Executed
Extract ALL Bash commands from the session:
- Look through all Bash tool calls
- Capture the command and what it was for
- Note which commands succeeded
- Skip failed/error commands

Categorize commands:
- AWS/Cloud commands (aws, gcloud, azure)
- Docker commands (docker, docker-compose)
- Git commands (git add, commit, push, etc.)
- Build commands (npm, make, mvn, cargo, etc.)
- Database commands (psql, mysql, mongo, etc.)
- Testing commands (pytest, npm test, etc.)
- Deployment commands

### D. Technical Decisions Made
Look for decisions in conversation:
- "Let's use X instead of Y because..."
- "We should implement this as..."
- "The best approach is..."
- Library/framework choices
- Architecture decisions
- Design pattern selections

For each decision, capture:
- What was decided
- Why (rationale from conversation)
- Alternatives mentioned (if any)
- Trade-offs discussed

### E. Issues Found or Fixed
Scan conversation for:

**Issues Found:**
- Error messages discussed
- Bugs discovered
- Problems encountered
- Workarounds applied

**Issues Fixed:**
- "Fixed X by doing Y"
- Problems that were resolved
- Solutions implemented

### F. Next Steps
Look for:
- "Next, we should..."
- "TODO: ..."
- "In the next session..."
- Incomplete tasks
- Future improvements mentioned
- Things left to do

### G. Important Context
Capture:
- Environment details (AWS profiles, databases, etc.)
- Credentials references (never actual secrets!)
- Special configurations
- Critical reminders
- Constraints or limitations discussed

## Step 2: Detect Project Root

Find project root by checking (in order):
1. Git repository root: `git rev-parse --show-toplevel`
2. Directory with package.json
3. Directory with CLAUDE.md
4. Current working directory

## Step 3: Update SESSION_HANDOFF.txt

Location: `<project-root>/docs/SESSION_HANDOFF.txt`

**PREPEND** (add to top) this new entry:

```
=== Session End: [CURRENT_TIMESTAMP] ===

Session Duration: [If detectable, otherwise "N/A"]

What was accomplished:
[Bullet list of main tasks from Step 1.A]
- [Task 1 with specific details]
- [Task 2 with specific details]
- [Task 3 with specific details]

Files created/modified:
[From Step 1.B - git status output]
- path/to/file1.js: [Purpose inferred from context]
- path/to/file2.py: [Purpose inferred from context]

Commands that worked:
[From Step 1.C - successful commands with descriptions]
# Description of what this does
command line here

# Another command description
another command here

Tests/Validation performed:
[Extract any testing done from conversation]
- [Test 1]: [Result]
- [Test 2]: [Result]

Current blockers:
[From Step 1.E - unresolved issues, or "NONE"]

Next session should start with:
1. Read /docs/CURRENT_STATE.txt
2. Read /docs/SESSION_HANDOFF.txt
3. [Specific files to review if relevant]
4. Continue with: [First next step from Step 1.F]

Files to reference:
[Key files from Step 1.B with their purpose]
- file.js: [Purpose]
- file.py: [Purpose]

Critical reminders for next session:
[From Step 1.G - important context]
- [Reminder 1]
- [Reminder 2]

Open questions to address:
[Unanswered questions from conversation]
- [Question 1]
- [Question 2]

---

[Keep 3-4 most recent previous sessions, remove older ones]
```

## Step 4: Update CURRENT_STATE.txt

Location: `<project-root>/docs/CURRENT_STATE.txt`

Update these sections:

```
Last Updated: [CURRENT_TIMESTAMP]
```

```
=== COMPLETED RECENTLY ===
[Add completed items from Step 1.A]
[Keep last 5 items total, remove older ones]
```

```
=== IN PROGRESS ===
[Update based on partial completions and next steps]
[From Step 1.F - what's actively being worked on]
```

```
=== NEXT SESSION ===
Start with: [First action from Step 1.F]
First task: [Specific next task]
Files to read:
  [List relevant files from Step 1.B]
```

```
=== IMPORTANT NOTES ===
[Add any new critical info from Step 1.G]
[Preserve existing important notes]
```

Preserve other sections but ensure they're current.

## Step 5: Update COMMANDS_LOG.txt

Location: `<project-root>/docs/COMMANDS_LOG.txt` or `<project-root>/docs/security/COMMANDS_LOG.txt`

For each command from Step 1.C:

1. **Check if command already exists** - don't duplicate
2. **Categorize automatically**:
   - AWS commands → "Cloud/Infrastructure (AWS/GCP/Azure)"
   - Docker commands → "Docker/Container Commands"
   - Git commands → "Git Operations"
   - npm/yarn/build → "Build & Deploy"
   - psql/mysql → "Database Operations"
   - pytest/test → "Testing & Validation"
   - logs/tail → "Debugging & Logs"

3. **Add with description**:
```
# [What this command does - inferred from context]
command here

# [Expected output or result]
```

4. Update timestamp:
```
Last Updated: [CURRENT_TIMESTAMP]
```

## Step 6: Update DECISIONS.txt

Location: `<project-root>/docs/DECISIONS.txt` or `<project-root>/docs/security/DECISIONS.txt`

For each decision from Step 1.D:

```
Decision: [Clear statement]
Date: [CURRENT_DATE]
Category: [Auto-categorize: Architecture/Technology/Process/Security/Performance]

Context & Requirements:
[From conversation - why this was needed]

Chosen Solution:
[What was decided]

Alternatives Considered:
[If mentioned in conversation, otherwise "Not discussed"]

Trade-offs Accepted:
+ Pros: [Benefits mentioned]
- Cons: [Drawbacks mentioned or "None identified"]

Impact:
- Affected components: [Infer from files changed]
- Dependencies added: [From package.json changes or conversation]

Review/Revisit:
[If mentioned, otherwise "As needed"]

---
```

Update timestamp:
```
Last Updated: [CURRENT_TIMESTAMP]
```

## Step 7: Update ISSUES.txt

Location: `<project-root>/docs/ISSUES.txt` or `<project-root>/docs/security/ISSUES.txt`

### For New Issues Found (from Step 1.E):

```
Issue #[auto-increment]: [Descriptive title from error/problem]
Severity: [Infer from context: Critical/High/Medium/Low]
Impact: [What breaks - from conversation]
Environment: [Where it occurred - infer from context]
First seen: [CURRENT_DATE]

Description:
[Problem description from conversation]

Reproduction steps:
[If mentioned, otherwise "See description"]

Workaround:
[If any workaround was applied, otherwise "NONE"]

Root cause:
[If identified in conversation, otherwise "Under investigation"]

Permanent fix needed:
[If discussed, otherwise "To be determined"]

Status: [Open/In Progress based on conversation]
Assigned to: Unassigned
Related issues: [If mentioned]

---
```

### For Issues Fixed (from Step 1.E):

Move from Active to Resolved:
```
Issue: [Title/description]
Resolution: [How it was fixed - from conversation]
Date resolved: [CURRENT_DATE]
Lessons learned: [If discussed]
Prevention: [If discussed, otherwise "N/A"]

---
```

Update timestamp.

## Step 8: Git Detection & Smart Inference

Run these to gather data:
```bash
# Detect project root
git rev-parse --show-toplevel 2>/dev/null

# Get changed files
git status --short
git diff --name-only
git diff --staged --name-only

# Get recent commits (if any made during session)
git log --oneline -5

# Check current branch
git branch --show-current
```

Use this to:
- Verify file changes
- Infer what was worked on
- Get commit messages for context
- Understand branch/feature being developed

## Step 9: Conversation Analysis

Review the entire conversation and extract:
- User's initial request/goal
- Key topics discussed
- Solutions provided
- Code snippets created
- Errors encountered
- Explanations given
- Recommendations made

Use this to write comprehensive, accurate updates.

## Step 10: Generate Summary Report

After updating all files, provide this output:

```
✅ Session handoff complete - Auto-captured from session history

📊 Session Analysis:
------------------
Main work: [1-line summary]
Files modified: [count]
Commands logged: [count]
Decisions documented: [count]
Issues tracked: [count new/fixed]

📁 Files Updated:
----------------
✓ /docs/SESSION_HANDOFF.txt
  → Added session entry with [X] accomplishments

✓ /docs/CURRENT_STATE.txt
  → Updated progress: [X] completed, [Y] in progress
  → Next session: [first next step]

[✓ /docs/COMMANDS_LOG.txt
  → Added [X] new commands] (if applicable)

[✓ /docs/DECISIONS.txt
  → Documented [X] decisions] (if applicable)

[✓ /docs/ISSUES.txt
  → Tracked [X] new issues, resolved [Y]] (if applicable)

📝 Summary:
----------
Worked on: [main task/feature]
Status: [overall status]
Next: [first next step]
[Blockers: [list]] (if any)

🔄 Next Session Startup:
-----------------------
Run: "Before we start, read /docs/CURRENT_STATE.txt and summarize what you understand about our current progress."

💡 Tip: All context is preserved. Your next session will have full memory of this work.
```

## Important Guidelines

1. **Be comprehensive** - Capture EVERYTHING from the session
2. **Be accurate** - Only document what actually happened
3. **Infer intelligently** - Use context to understand purpose
4. **No placeholders** - Use real data from the session
5. **No questions** - Fully autonomous, no user input needed
6. **Preserve history** - Don't delete old content, archive it
7. **Cross-reference** - Link related items (issues to decisions, etc.)
8. **Smart categorization** - Auto-categorize everything appropriately
9. **Detect duplication** - Don't add commands/decisions that already exist
10. **Real timestamps** - Use actual current date/time

## Edge Cases

- **No git repository**: Still capture files from conversation, note "git not available"
- **No changes detected**: Document that session was exploratory/planning
- **Files don't exist yet**: Offer to create them or note they need initialization
- **Very short session**: Still capture what happened, even if minimal
- **Only conversation, no code**: Document discussions, decisions, plans

## Quality Checks

Before finishing:
1. ✓ All dates/timestamps are real (not placeholders)
2. ✓ File paths are accurate
3. ✓ Commands are complete and correct
4. ✓ Decisions include rationale
5. ✓ Issues are clearly described
6. ✓ Next steps are actionable
7. ✓ No sensitive data (passwords, keys, etc.)

## Auto-Detection Intelligence

Use these signals:
- Tool calls (Read/Write/Edit) → files worked on
- Bash commands → operations performed
- Error messages → issues found
- "Let's use X" → decisions made
- "Next we should" → next steps
- Package additions → dependencies
- Conversation tone → success/blockers
