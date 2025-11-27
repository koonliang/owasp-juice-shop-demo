---
description: Generate comprehensive technical documentation for any project - Usage: /doc-generator <project-directory-path>
---
You are tasked with creating comprehensive technical documentation for a specified project following the CLAUDE.md guidelines.

## Command Usage:

```
/doc-generator <project-directory-path>
```

**Example:**

```
/doc-generator /Users/rupeshpanwar/Documents/Project/Bundle/frontend
/doc-generator ./my-project
/doc-generator /path/to/any/project
```

**Note:** The project directory path is REQUIRED. If not provided, ask the user for the project path.

**Important:** All analysis, file exploration, and documentation creation must be performed on the specified project directory, not the current working directory.

## Documentation Guidelines (MUST FOLLOW):

- **Format:** TXT format with clean, minimal decoration
- **TL;DR:** Include a TL;DR section at the very top for quick reference
- **Layout:** Skim-able layout with clear section headings using "====="
- **Length:** Comprehensive coverage in ~500-600 lines for complex projects (200 lines minimum)
- **Style:** No heavy borders, just clean headings
- **Location:** Create document in /docs directory with descriptive filename

## Tasks to Complete:

1. **Project Overview and Quick Stats**

   - Application name, version, and repository location
   - Purpose and main capabilities (2-3 sentences)
   - Quick statistics: number of dependencies, modules, components, environments
   - Target users or use cases
2. **Technology Stack Breakdown**

   - Framework and build tools (with versions)
   - UI libraries and component frameworks
   - Styling solutions (CSS frameworks, preprocessors)
   - State management approach
   - Routing solution
   - Key libraries organized by category:
     * Authentication & Security
     * API & HTTP clients
     * Data visualization
     * Forms & Validation
     * Testing & Quality tools
     * Development tools
   - Database(s) and ORM/ODM
   - External services and integrations
3. **Project Structure**

   - Complete directory tree with explanations
   - Purpose of each major directory
   - Key files and configuration files
   - Naming conventions
   - Module organization pattern
4. **Main Features & Modules**

   - List all major features (10+ if applicable)
   - For each feature:
     * Brief description
     * Key components involved
     * Routes or entry points
     * Related stores/services
   - Authentication and authorization details
   - Role-based access control (if applicable)
5. **Environment Configuration**

   - API endpoints for each environment (dev/staging/prod)
   - Authentication configuration
   - Third-party service configurations
   - Feature flags
   - Environment variable examples
6. **Build & Deployment Processes**

   - Development commands (install, dev server, build)
   - Testing commands (unit, integration, e2e)
   - Code quality commands (lint, type-check, format)
   - Build output details
   - Deployment steps
7. **CI/CD Pipeline Details**

   - Pipeline stages overview
   - Key jobs in each stage
   - Automated tests and checks
   - Security scanning steps
   - Deployment triggers and approvals
   - Artifact management
8. **Routing Structure**

   - Complete list of all routes
   - Authentication requirements per route
   - Role-based access restrictions
   - Route guards and middleware
   - Route metadata
9. **Architectural Patterns**

   - Component architecture approach
   - State management pattern
   - API service pattern
   - Authentication architecture
   - Code organization principles
   - Security practices implemented
   - Performance optimizations applied
   - Testing strategy
10. **API Integration Flows**

    - Request/response cycle
    - Authentication flow (login, token refresh, MFA)
    - Error handling approach
    - Interceptor logic
11. **Key Dependencies & Purposes**

    - State management stores (purpose of each)
    - API service layer components
    - Authentication services
    - Error handling utilities
    - Utility services
12. **Application Entry Flow**

    - Initialization sequence
    - Application bootstrap process
    - Route resolution flow
    - Component mounting order
13. **Deployment Checklist**

    - Pre-deployment checks
    - Staging deployment steps
    - Production deployment steps
    - Post-deployment validation

## Output Requirements:

### File Creation:

- Create document in: `<project-directory-path>/docs/<project-name>.txt`
- If /docs directory doesn't exist in the project, create it first
- Use descriptive filename based on project name
- Always use the provided project directory path, not the current working directory

### Format Requirements:

- Start with document title and separator (===)
- Include TL;DR section at the very top
- Use section headings with "====" underlines (not borders)
- Use subsection headings with "----" underlines
- Keep line length reasonable (<100 chars where possible)
- Use bullet points and numbered lists for clarity
- Include code examples where relevant (commands, configs)

### Content Requirements:

- Be comprehensive but concise
- Focus on technical details developers need
- Include actual values found in the project (versions, URLs, IDs)
- Use clear, professional technical writing
- Organize information logically
- Make it skim-able with clear hierarchy

### Sections to EXCLUDE:

- ❌ Common issues and solutions
- ❌ Maintenance and monitoring guidelines
- ❌ General best practices section
- ❌ Contact and support information

### Example Section Format:

```
Section Heading
===============

Subsection Heading
------------------
- Bullet point with details
- Another bullet point
  - Nested details
  - More nested info

Key Point: Value or description
Another Point: Value or description

Code Example:
npm run dev              # Start development server
npm run build            # Production build
```

## Execution Steps (EFFICIENT APPROACH):

**CRITICAL:** Use Read, Glob, Grep tools directly. DO NOT use Task/Explore agent unless absolutely necessary for complex code flow analysis. Build documentation incrementally.

### 1. Extract & Validate Project Path

- Parse project directory path from command arguments
- If no path provided, ask: "Please provide the project directory path"
- Use Bash to validate: `ls <project-path>` or check key files exist

### 2. Detect Project Type (Read key files in parallel)

Read these files simultaneously to identify project type:

- `<project-path>/package.json` (Node.js/JavaScript)
- `<project-path>/pyproject.toml` or `setup.py` (Python)
- `<project-path>/go.mod` (Go)
- `<project-path>/pom.xml` or `build.gradle` (Java)
- `<project-path>/Cargo.toml` (Rust)
- `<project-path>/composer.json` (PHP)
- `<project-path>/Gemfile` (Ruby)
- `<project-path>/.csproj` (C#/.NET)

### 3. Gather Core Information (Targeted reads)

**Step 3a - Project Metadata:**
Read in parallel:

- Project config file (package.json, pyproject.toml, etc.)
- README.md (if exists)
- .env.example or .env.template (environment variables)

**Step 3b - Build & CI/CD:**
Read in parallel:

- .gitlab-ci.yml, .github/workflows/*.yml, Jenkinsfile, etc.
- Dockerfile, docker-compose.yaml
- Makefile (if exists)
- Build config (tsconfig.json, webpack.config.js, etc.)

**Step 3c - Project Structure:**
Use Glob to find key directories:

- `<project-path>/src/**` or `<project-path>/lib/**`
- `<project-path>/tests/**` or `<project-path>/__tests__/**`
- `<project-path>/docs/**`

**Step 3d - Entry Points & Routing:**
Based on project type:

- Node.js: Read `src/index.ts`, `src/app.ts`, `src/router.ts`
- Python: Read `main.py`, `app.py`, `__init__.py`
- Go: Read `main.go`, `cmd/**/*.go`
- Java: Find main class with Grep: `public static void main`

**Step 3e - Module Discovery:**
Use Glob patterns:

- Controllers: `**/*Controller.{ts,js,py}` or `**/controllers/**`
- Models: `**/*Model.{ts,js,py}` or `**/models/**`
- Services: `**/*Service.{ts,js,py}` or `**/services/**`
- Routes: `**/*Router.{ts,js,py}` or `**/routes/**`

**Step 3f - Configuration Files:**
Read test/lint configs:

- `jest.config.js`, `pytest.ini`, `go.mod`
- `eslint.config.js`, `.pylintrc`, etc.

### 4. Extract Project Name

From config file read in Step 3a:

- Node.js: `package.json` → name field
- Python: `pyproject.toml` → name field
- Go: `go.mod` → module name
- Fallback: Directory basename

### 5. Build Documentation Incrementally

**DO NOT wait to gather everything first!**

- Start writing documentation as soon as you have Section 1 data
- Add sections progressively as you read more files
- Use information already in memory from previous reads

### 6. Create Documentation File

- Ensure `<project-path>/docs/` directory exists (create if needed)
- Write to: `<project-path>/docs/<project-name>.txt`
- Follow format guidelines strictly

### 7. Confirm Completion

- Report full path: `<project-path>/docs/<project-name>.txt`
- Brief summary: "Documented X sections covering Y features"

## Performance Guidelines:

**DO:**
✓ Read multiple files in parallel (single message, multiple Read calls)
✓ Use Glob for pattern matching (find all controllers, models, etc.)
✓ Use Grep for specific searches (find route definitions, class declarations)
✓ Build documentation incrementally section-by-section
✓ Skip reading large test files or generated code

**DON'T:**
✗ Use Task/Explore agent (wastes 30K+ tokens)
✗ Read every single file in the project
✗ Wait to gather all info before starting documentation
✗ Read node_modules, vendor, dist, build directories
✗ Use "very thorough" exploration unless truly needed

**Token Budget:** Aim for <8,000 tokens total (vs 45K+ with Task/Explore)

The final document should serve as the definitive technical reference for the project, enabling any developer to understand the architecture, setup, and deployment process quickly.
