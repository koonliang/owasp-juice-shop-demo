---
description: Initialize context preservation files for any project - Usage /init-context-files <project-root-path>
tags: [project, gitignored]
---

You are tasked with creating generic context preservation template files to help maintain project state across multiple Claude Code sessions.

## User Input
The user will provide: `/init-context-files <project-root-path>`

Example: `/init-context-files /Users/rupeshpanwar/Documents/Project/Bundle`
Example: `/init-context-files .` (for current directory)

## Steps to Execute

### 1. Parse Project Path
- Extract the project root path from user input
- Resolve relative paths (like `.` or `..`) to absolute paths
- Validate that the path exists

### 2. Create /docs Directory
- Create `<project-root>/docs/` directory if it doesn't exist
- Set appropriate permissions (755)

### 3. Create 5 Template Files
Create these files in `<project-root>/docs/`:

#### File 1: CURRENT_STATE.txt
```
Last Updated: YYYY-MM-DD HH:MM
Current Project: [Project Name - Auto-detect from directory name]
Active Phase: [Not started]

=== QUICK CONTEXT ===
Working on: [Brief description of what you're building]
Key Services: [List main services/components]
Access Method: [How to access resources - AWS profile, SSH, API keys, etc.]
Environment: [Development/Staging/Production]

=== COMPLETED RECENTLY ===
- [Task completed]
- [Another completed task]

=== IN PROGRESS ===
- [Current active work]
- Status: [Progress/blockers]

=== NEXT SESSION ===
Start with: [What to do when resuming work]
First task: [Specific next task]
Files to read:
  - [Filename with line numbers if applicable]
  - [Another file]

=== IMPORTANT NOTES ===
- [Critical constraint or requirement]
- [Security/access consideration]
- [Environment-specific info]

=== FILE LOCATIONS ===
Config files: [Path]
Scripts: [Path]
Data/Artifacts: [Path]
Documentation: [Path]

=== CREDENTIALS & ACCESS ===
Note: Never store actual credentials here - only references
- AWS Profile: [Profile name or "See ~/.aws/credentials"]
- Database: [Connection info location or Secrets Manager path]
- APIs: [Where keys are stored]

---
```

#### File 2: SESSION_HANDOFF.txt
```
# Session Handoff Log
# Purpose: Track session endings and prepare for next session startup
# Usage: Update this at the END of each session

=== Session End: YYYY-MM-DD HH:MM ===

Session Duration: [X hours Y minutes]

What was accomplished:
- [Completed item 1]
- [Completed item 2]
- [Completed item 3]

Files created/modified:
- [Filename]: [What changed]
- [Filename]: [What changed]

Tests/Validation performed:
- [Test 1]: [Result]
- [Test 2]: [Result]

Current blockers:
- [Blocker 1 or "NONE"]
- [Blocker 2 or "NONE"]

Next session should start with:
1. Read CURRENT_STATE.txt
2. Read SESSION_HANDOFF.txt
3. Review: [Specific files/sections]
4. Continue with: [Specific task/phase]

Files to reference:
- [File]: [Purpose - e.g., "Implementation plan"]
- [File]: [Purpose - e.g., "Current codebase"]

Critical reminders for next session:
- [Reminder about environment/credentials]
- [Reminder about constraints]
- [Reminder about decisions made]

Open questions to address:
- [Question 1]
- [Question 2]

---

=== Session: YYYY-MM-DD ===
[Keep 3-5 most recent sessions for history]

Accomplished: [Summary]
Next: [What's next]

---

=== Session: YYYY-MM-DD ===
[Previous session]

---
```

#### File 3: COMMANDS_LOG.txt
```
# Commands Log
# Purpose: Track successful commands, scripts, and operations
# Usage: Add commands here as you discover what works

Project: [Auto-detected project name]
Last Updated: YYYY-MM-DD

=== Quick Reference Commands ===
# Most commonly used commands for this project

[Your frequently used commands will go here]

---

=== Environment Setup ===
# Commands to set up development environment

# Example:
# export ENV_VAR=value
# source .env

---

=== Build & Deploy ===
# Build, compile, package, deploy commands

# Example:
# npm run build
# docker build -t app:latest .
# kubectl apply -f deployment.yaml

---

=== Database Operations ===
# Database connection, migration, backup commands

# Example:
# psql -h hostname -U user -d database
# npm run migrate
# pg_dump database > backup.sql

---

=== Testing & Validation ===
# Commands to run tests and validate functionality

# Example:
# npm test
# pytest tests/
# curl -X GET http://localhost:8080/health

---

=== Debugging & Logs ===
# Commands to view logs, debug issues

# Example:
# tail -f /var/log/app.log
# kubectl logs pod-name
# docker logs container-id

---

=== Git Operations ===
# Git workflow commands specific to this project

# Example:
# git checkout -b feature/new-feature
# git commit -m "message"
# git push origin branch-name

---

=== Cloud/Infrastructure (AWS/GCP/Azure) ===
# Cloud provider specific commands

# Example:
# aws s3 ls s3://bucket-name
# aws ecs describe-services --cluster name
# gcloud compute instances list

---

=== Docker/Container Commands ===
# Container management

# Example:
# docker-compose up -d
# docker exec -it container sh
# docker ps -a

---

=== File Locations ===
Scripts: [Path to scripts directory]
Config: [Path to config files]
Logs: [Where logs are stored]
Artifacts: [Where build outputs go]

---

=== Connection Strings (Redacted) ===
# Template connection strings (no real credentials!)

Database: [connection string template]
API: [endpoint template]
Cache: [redis/memcached connection]

---

=== Troubleshooting Notes ===
Common Issue: [Description]
Solution: [Command or fix]

Common Issue: [Description]
Solution: [Command or fix]

---
```

#### File 4: DECISIONS.txt
```
# Technical Decisions Log
# Purpose: Document WHY choices were made, not just WHAT
# Usage: Add entry whenever making architectural or technical decision

Project: [Auto-detected project name]
Last Updated: YYYY-MM-DD

=== How to Use This File ===
When making a decision, document:
1. What was decided
2. Why this choice (context/requirements)
3. Alternatives considered
4. Trade-offs accepted
5. When to revisit

---

=== DECISION TEMPLATE (Copy this for new entries) ===

Decision: [Clear statement of what was decided]
Date: YYYY-MM-DD
Category: [Architecture/Technology/Process/Security/Performance]

Context & Requirements:
- [Requirement 1]
- [Requirement 2]

Chosen Solution:
[Describe the solution/approach selected]

Alternatives Considered:
1. [Alternative 1]: Rejected because [reason]
2. [Alternative 2]: Rejected because [reason]

Trade-offs Accepted:
+ Pros: [Benefits of this choice]
- Cons: [Drawbacks we're accepting]

Impact:
- Affected components: [List]
- Migration needed: [Yes/No - details]
- Dependencies added: [List]

Review/Revisit:
- When: [Condition or date to reconsider]
- Why: [What might change this decision]

---

=== Active Decisions ===

[Your project decisions will be documented here]

---

=== Technical Constraints ===
# Non-negotiable constraints for this project

Constraint: [Description]
Reason: [Why this constraint exists]
Impact: [How it affects design]

---

=== Patterns & Standards ===
# Coding patterns, naming conventions, architectural patterns

Pattern: [Name]
Where: [Which parts of codebase]
Why: [Reason for this pattern]
Example: [Code or file reference]

---

=== Technology Stack ===
# Key technologies and WHY they were chosen

Technology: [Name - e.g., PostgreSQL]
Version: [Version number]
Purpose: [What it's used for]
Why chosen: [Reason over alternatives]
Alternatives considered: [List with brief why not]

---

=== Dependencies ===
# Major dependencies and justification

Dependency: [Package/library name]
Version: [Version or range]
Purpose: [What it does]
Why this one: [Why not alternatives]
Risk: [Any concerns about this dependency]

---
```

#### File 5: ISSUES.txt
```
# Known Issues & Workarounds
# Purpose: Track problems, bugs, and their solutions
# Usage: Add issues as discovered, update when resolved

Project: [Auto-detected project name]
Last Updated: YYYY-MM-DD

=== How to Use This File ===
- Add new issues to "Active Issues" section
- Move to "Resolved" when fixed
- Keep "Common Errors" updated as a reference

---

=== ISSUE TEMPLATE (Copy for new issues) ===

Issue #[number]: [Brief descriptive title]
Severity: [Critical/High/Medium/Low]
Impact: [What breaks or doesn't work]
Environment: [Where it occurs - dev/staging/prod/all]
First seen: YYYY-MM-DD

Description:
[Detailed description of the problem]

Reproduction steps:
1. [Step 1]
2. [Step 2]
3. [Observe problem]

Workaround:
[Temporary solution if available, or "NONE"]

Root cause:
[If known, describe the underlying cause]

Permanent fix needed:
[What needs to happen to truly resolve this]

Status: [Open/In Progress/Blocked/Need Info]
Assigned to: [Person or "Unassigned"]
Related issues: [Links to related issues]

---

=== Active Issues ===

[Current problems will be tracked here]

---

=== Resolved Issues ===

Issue: [Title]
Resolution: [What fixed it]
Date resolved: YYYY-MM-DD
Lessons learned: [What we learned from this]
Prevention: [How to avoid in future]

---

=== Environment-Specific Issues ===
# Problems that only occur in certain environments

Environment: [Dev/Staging/Production]
Issue: [Description]
Cause: [Why it happens in this env]
Workaround: [How to handle it]

---

=== Common Errors & Solutions ===
# Quick reference for frequently encountered errors

Error Message: [Exact error text or pattern]
Cause: [What causes this error]
Solution: [How to fix it]
Prevention: [How to avoid it]

---

Error Message: [Another common error]
Cause: [What causes it]
Solution: [Fix]
Command: [Exact command to run]

---

=== Performance Issues ===

Component: [Service/module/function]
Symptom: [Slow response, high CPU, etc.]
Threshold: [What's the acceptable performance]
Current: [Current performance]
Optimization needed: [What to try]
Status: [Open/In Progress/Resolved]

---

=== Security Concerns ===
# Security-related issues or considerations

Concern: [Description]
Severity: [Critical/High/Medium/Low]
Mitigation: [Current mitigation if any]
Long-term fix: [What's needed]
Compliance: [If related to compliance requirement]

---

=== Technical Debt ===
# Things that work but need refactoring

Item: [Description]
Location: [File/module]
Why debt: [Why it's suboptimal]
Impact: [How it affects development]
Effort to fix: [Estimated effort]
Priority: [High/Medium/Low]

---
```

### 4. Set Current Date/Time
- Replace all `YYYY-MM-DD HH:MM` placeholders with actual current timestamp
- Replace `[Auto-detected project name]` with the directory name from path

### 5. Create README for /docs folder
Create `<project-root>/docs/README.md`:

```markdown
# Project Documentation & Context Files

This directory contains context preservation files to maintain project state across Claude Code sessions.

## Files Overview

| File | Purpose | Update Frequency |
|------|---------|------------------|
| `CURRENT_STATE.txt` | Current project status snapshot | Every major milestone |
| `SESSION_HANDOFF.txt` | End-of-session summary | End of each session |
| `COMMANDS_LOG.txt` | Successful commands library | As commands are discovered |
| `DECISIONS.txt` | Why technical choices were made | When decisions are made |
| `ISSUES.txt` | Known problems & workarounds | As issues are found/resolved |

## How to Use

### Starting a New Session
```
Claude, read /docs/CURRENT_STATE.txt and /docs/SESSION_HANDOFF.txt
Then summarize what we should work on next.
```

### Ending a Session
```
Update /docs/SESSION_HANDOFF.txt with:
- What we completed
- Next steps
- Any blockers
```

### During Work
- Add successful commands to `COMMANDS_LOG.txt`
- Document decisions in `DECISIONS.txt`
- Track issues in `ISSUES.txt`
- Update `CURRENT_STATE.txt` when completing major tasks

## Best Practices

1. **Keep CURRENT_STATE.txt updated** - It's your session startup file
2. **Always update SESSION_HANDOFF.txt** - Write it like you're briefing your future self
3. **Document WHY, not just WHAT** - Future you will thank you
4. **Real examples > Abstract templates** - Fill in with actual project data
5. **Review periodically** - Archive old content to keep files manageable

## Tips

- These files are meant to be living documents
- Don't worry about perfect formatting - clarity > perfection
- Use these as conversation starters with Claude Code
- Reference line numbers when telling Claude what to read
- Keep sensitive data out - use references to secret stores instead

---
Generated by `/init-context-files` slash command
```

### 6. Verification
After creating all files:
1. List all created files with full paths
2. Confirm /docs directory creation
3. Show file sizes
4. Display success message with usage instructions

### 7. Output to User

Provide:
```
✅ Context preservation files initialized!

Created in: <absolute-path-to-project>/docs/

Files created:
1. CURRENT_STATE.txt (ready to customize)
2. SESSION_HANDOFF.txt (template ready)
3. COMMANDS_LOG.txt (template ready)
4. DECISIONS.txt (template ready)
5. ISSUES.txt (template ready)
6. README.md (usage guide)

Next steps:
1. Customize CURRENT_STATE.txt with your project details
2. Start working on your project
3. At session end, update SESSION_HANDOFF.txt
4. At next session start: "Read /docs/CURRENT_STATE.txt"

Example usage at start of next session:
"Read <project-path>/docs/CURRENT_STATE.txt and <project-path>/docs/SESSION_HANDOFF.txt, then tell me what we should work on next."
```

## Error Handling
- If path doesn't exist: "Error: Path '<path>' does not exist. Please provide a valid project directory."
- If no path provided: "Usage: /init-context-files <project-root-path>"
- If /docs creation fails: Show error and suggest manual creation

## Important Notes
- All templates are generic and project-agnostic
- Auto-detect project name from directory
- Never hardcode paths or project-specific info
- Create /docs if it doesn't exist
- All files should have clear headers and usage instructions
- Templates should guide users on what to fill in
