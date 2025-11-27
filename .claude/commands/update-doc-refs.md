---
description: Update CLAUDE.md with all txt files from docs directory
---

You are tasked with updating the "Documents To Refer" section in CLAUDE.md with all available .txt documentation files.

## Command Purpose

This command automatically discovers all documentation files in the /docs directory and updates CLAUDE.md to reference them, ensuring Claude Code CLI is aware of all available documentation during sessions.

## Execution Steps

### Step 1: Find All Documentation Files

Use Glob to find all .txt files in the docs directory:
- Path: /Users/rupeshpanwar/Documents/AI-Projects/owasp-juice-shop/docs
- Pattern: Include all .txt files in docs/ and subdirectories
- Include files from security-scan/ subdirectory

### Step 2: Filter and Organize Files

**EXCLUDE these files/directories:**
- Files in Summary/ subdirectory (session summaries, not primary docs)
- ChangeLog.txt (mentioned separately in "Start Every Session With" section)
- CURRENT_STATE.txt (mentioned in session workflow)
- SESSION_HANDOFF.txt (mentioned in session workflow)
- COMMANDS_LOG.txt (internal tracking file)
- DECISIONS.txt (internal tracking file)
- ISSUES.txt (internal tracking file)
- Files in docker/ subdirectory (if they exist and are duplicates)
- Files in LocalStack/ subdirectory (if they exist and are duplicates)

**INCLUDE these priority files:**
- All primary documentation files in /docs root
- Files in security-scan/ subdirectory
- Any deployment guides, training docs, workflow guides

**Sort Strategy:**
1. Sort alphabetically by filename
2. Maintain relative paths starting with /docs/

### Step 3: Read Current CLAUDE.md

Read the current CLAUDE.md file from:
/Users/rupeshpanwar/Documents/AI-Projects/owasp-juice-shop/CLAUDE.md

**Identify the section to update:**
- Section: "## Documents To Refer"
- Starts at line ~11
- Includes intro text: "ALWAYS refer below documents while working on this application"
- Followed by numbered list (1., 2., 3., etc.)

### Step 4: Build New Documentation List

Create a numbered list with:
- Intro line: "ALWAYS refer below documents while working on this application"
- Empty line
- Numbered list starting from 1
- Format: `1. /docs/FILENAME.txt`
- Relative paths starting with /docs/
- Sequential numbering (1., 2., 3., etc.)

**Example output format:**
```
## Documents To Refer

ALWAYS refer below documents while working on this application

1. /docs/AWS_DEPLOYMENT_PLAN.txt
2. /docs/AWS_DEPLOYMENT_PLAN_ECS.txt
3. /docs/DEVSECOPS_TRAINING_WITH_JUICE_SHOP.txt
4. /docs/Deploy-App-2-K8s-Cluster.txt
5. /docs/FRONTEND_BACKEND_DATABASE_FLOW.txt
...
10. /docs/security-scan/security-scan-tools.txt
```

### Step 5: Update CLAUDE.md

Use the Edit tool to replace ONLY the "Documents To Refer" section:
- Find the old section: From "## Documents To Refer" to the next "##" heading
- Replace with the new numbered list
- Preserve all other sections unchanged:
  - Project Overview
  - Git Commit Guidelines
  - Documentation Guidelines
  - Start Every Session With
  - End Every Session With
  - Create Session Reports Guidelines
  - Technology Stack
  - All other sections

**CRITICAL: Do NOT modify these sections:**
- Git Commit Guidelines (lines ~26-28)
- Documentation Guidelines (lines ~30-39)
- Start Every Session With (lines ~41-54)
- End Every Session With (lines ~56-67)
- Create Session Reports Guidelines (lines ~69+)

### Step 6: Verify Update

After updating:
1. Count total documentation files found
2. Identify any NEW files added since last update
3. Identify any files REMOVED since last update
4. Show the complete updated list to the user

### Step 7: Report to User

Provide a clear summary:

```
✅ CLAUDE.md updated successfully!

Documentation files indexed: [count]

Added files:
- [new file 1]
- [new file 2]
(or "None - all files were already indexed")

Removed files:
- [removed file 1]
(or "None")

Complete documentation list:
1. /docs/file1.txt
2. /docs/file2.txt
...
[full numbered list]

Next steps:
- Claude Code CLI can now reference all documentation files
- Use "Read /docs/FILENAME.txt" in your prompts
- Documentation is automatically discovered in future sessions
```

## Important Notes

**Preserve CLAUDE.md Structure:**
- Only update the "Documents To Refer" section
- Keep exact formatting and spacing
- Maintain all other sections unchanged
- Use consistent indentation (no tabs, 2 spaces for sub-items if needed)

**File Discovery Priority:**
- Primary documentation files (deployment, training, guides)
- Security scanning documentation
- Technical architecture documentation
- Workflow and process documentation

**Exclusion Rationale:**
- Context preservation files (CURRENT_STATE.txt, etc.) are referenced in session workflow sections, not in primary documentation list
- Summary files are session-specific outputs, not reference documentation
- ChangeLog.txt is referenced separately in "Start Every Session With" section

## Error Handling

**If no .txt files found:**
- Error message: "No documentation files found in /docs directory"
- Suggestion: "Run /init-context-files and /doc-generator first"

**If CLAUDE.md doesn't exist:**
- Error message: "CLAUDE.md not found at expected path"
- Suggestion: "Ensure you're in the owasp-juice-shop project directory"

**If Glob fails:**
- Error message: "Failed to search for documentation files"
- Show error details
- Suggest manual verification of /docs directory

## Example Full Workflow

```
User: /update-doc-refs