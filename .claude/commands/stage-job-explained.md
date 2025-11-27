---
description: Explain any pipeline stage/job with concise, skim-able format
---
# Stage/Job Explanation Command

Your job is to explain the pipeline stage/job mentioned by the user.

## Task

1. Identify which stage/job the user is asking about
2. Check if `docs/STAGE_EXPLAINED.txt` exists
3. If exists, add the new stage section following the established pattern
4. If doesn't exist, create it with the "Key Insight" header first
5. Keep information **concise and skim-able** (limit words, maximize clarity)

---

## File Structure Pattern

### First Time Creating File:

```
PIPELINE EXPLAINED
===================

Key Insight:

  • [Tool 1]  → Finds [what it detects] ([category])
  • [Tool 2]  → Finds [what it detects] ([category])
  • [Tool 3]  → Finds [what it detects] ([category])

[Then add individual stage sections...]
```

### For Each Stage Section:

**Use this exact format** (from STAGE_EXPLAINED.txt):

```
================================================================================

================================================================================

STAGE X: [Tool Name] - [Brief Description]
===========================================

TL;DR
-----
[Tool] = [One-line explanation]. Think of it as [simple analogy].


Command Execution:
------------------
[Actual command from pipeline]

Meaning:
  • [flag/option 1]  → [what it does]
  • [flag/option 2]  → [what it does]
  • [output]         → [where results go]




PURPOSE IN CI/CD PIPELINE
============================

Why [Tool] in Stage X?
----------------------
✓ [Benefit 1] → [Brief explanation]
✓ [Benefit 2] → [Brief explanation]
✓ [Benefit 3] → [Brief explanation]

Pipeline Position:
------------------
Stage X: [Tool] (Parallel with [others] OR Sequential after [previous])
             ↓
         Runs after Stage Y


INTERPRETATION OF OUTPUT
==========================

Result: [SUCCESS/WARNING/FAILURE] ([context])
----------------------------------------------
[Brief description of typical output]

Typical [App Name] Findings ([context if intentionally vulnerable]):

[Show actual example output - keep it real and concise]

✓ [Finding 1] (CVE-XXXX)      → [Severity]
  Location: [file:line]
  "[Brief description]"

✓ [Finding 2] (CVE-XXXX)      → [Severity]
  Location: [file:line]
  "[Brief description]"

What this means:
• [Implication 1]
• [Implication 2]
• [Pipeline behavior - continue/fail]




================================================================================

Key Insight: [Tool-specific insight about what it detects and limitations]
```

---

## Word Limits (Keep It Skim-able)

**TL;DR:**

- Max 2 lines
- One analogy

**Command Execution:**

- Show command
- 3-5 bullet points for meaning
- No lengthy explanations

**Purpose:**

- 3-4 benefits max
- Each benefit: 5-10 words
- Use "→" for quick explanation

**Output Interpretation:**

- Show 2-3 real examples
- Each example: 3 lines max
- "What this means": 3 bullets, 1 line each

**Common Errors:**

- 2-3 examples max
- Brief "What these mean" (1 line per error)

---

## Important Guidelines

**DO:**

- ✅ Extract actual console output from user's context
- ✅ Use real command examples from pipeline
- ✅ Show actual file:line locations
- ✅ Include CVE IDs when relevant
- ✅ Note continue-on-error context
- ✅ Mention parallel execution
- ✅ Reference artifact uploads
- ✅ Include runner type (ubuntu-latest vs self-hosted)
- ✅ Keep it **concise and skim-able**
- ✅ Update "Key Insight" section at top when adding new tools

**DON'T:**

- ❌ Write lengthy paragraphs
- ❌ Add detailed remediation sections
- ❌ Include verbose best practices
- ❌ Duplicate information
- ❌ Add multiple examples of same error type
- ❌ Go over word limits

---

## Stage-Specific Adaptations

**SAST (CodeQL, SonarQube):**

- Focus on vulnerability types (SQL injection, XSS, etc.)
- Show data flow tracking
- Include CWE/CVE references

**Secret Detection (TruffleHog, Gitleaks):**

- Focus on credential types (AWS keys, GitHub tokens, etc.)
- Show verified vs unverified distinction
- Include detector names

**Dependency Scan (npm audit, Trivy, RetireJS):**

- Focus on package vulnerabilities
- Show severity distribution
- Include CVE references
- Note transitive dependencies

**DAST (OWASP ZAP, Nikto):**

- Focus on runtime vulnerabilities
- Show HTTP endpoint testing
- Include OWASP category references

**Container Scan (Trivy, Grype):**

- Focus on base image vulnerabilities
- Show OS package issues
- Include layer analysis

**Policy Validation (OPA/Conftest):**

- Focus on configuration issues
- Show policy violations
- Include best practice references

**Attack Surface (OWASP Noir):**

- Focus on API endpoint discovery
- Show technology detection
- Include endpoint counts

---

## Example: Applying Word Limits

**❌ Too Verbose:**

```
TL;DR
-----
SAST stands for Static Application Security Testing. CodeQL is a powerful
semantic code analysis engine that analyzes your source code to find security
vulnerabilities such as SQL injection, cross-site scripting, and many other
types of security issues without actually running the application. It uses
data flow analysis to trace how user input flows through your application
to potentially vulnerable code paths.
```

**✅ Concise (Following Pattern):**

```
TL;DR
-----
SAST = Static Application Security Testing. CodeQL analyzes source code for
security vulnerabilities (SQL injection, XSS, etc.) WITHOUT running the app.
Think of it as a security-focused X-ray of your code.
```

---

## Key Insight Pattern (Top of File)

Always maintain this summary at the top of STAGE_EXPLAINED.txt:

```
Key Insight:

  • npm audit  → Finds vulnerable dependencies (supply chain)
  • CodeQL     → Finds vulnerable code (application logic)
  • TruffleHog → Finds leaked secrets (credentials)
  • Trivy      → Finds container vulnerabilities (base images)
  • OWASP ZAP  → Finds runtime vulnerabilities (DAST)
  • Noir       → Finds API attack surface (endpoints)
```

Update this section whenever a new scanning tool is added.

---

## When User Asks

**User says:** "Explain the npm audit stage"

**You do:**

1. Read `docs/STAGE_EXPLAINED.txt`
2. Check if npm audit section exists
3. If not, add it following the concise pattern
4. Update "Key Insight" section if needed
5. Extract actual output from pipeline if available
6. Keep it **brief and skim-able**

**User says:** "Explain this error: [shows output]"

**You do:**

1. Identify the stage from output
2. Add to COMMON ERRORS section
3. Keep explanation to 1-2 lines max
4. Show actual error code/message

---

## Output Format

Always output:

1. Confirmation: "Added [Stage Name] to docs/STAGE_EXPLAINED.txt"
2. Brief summary: "[Tool] scans for [what] in [where]"
3. Key finding: "Typical findings: [1-2 examples]"
4. Location reference: "See lines [X-Y] in STAGE_EXPLAINED.txt"

Keep response concise - let the file speak for itself!
