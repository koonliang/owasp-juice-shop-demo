# Git Workflow Command

Universal git workflow for all project types including branching, security checks, commits, and pushing.

## Usage Note
When applying this workflow, specify the project path you're working on:
- Navigate to the project directory first, OR
- Specify the full path to the project when running git commands

## Step 0: Initial Branch Decision

**IMPORTANT: Always start by asking the user about their branching preference**

After checking git status, use the AskUserQuestion tool to ask:

```
Question: "You are currently on branch '<current-branch>'. How would you like to proceed?"
Options:
  1. "Continue on current branch" - Work directly on the current branch
  2. "Create new feature branch" - Create and switch to a new branch following naming conventions
```

**When to recommend each option:**

**Continue on current branch:**
- Personal projects / solo development
- Already on a feature branch
- Quick fixes or documentation updates
- Training/learning projects

**Create new feature branch:**
- Team projects / collaborative development
- Working from main/master/develop branch
- Need code review via pull request
- Want to test changes before merging

After user selects their preference, proceed accordingly:
- If "Continue on current branch" → Skip to Section 2 (Security Pre-Commit Check)
- If "Create new feature branch" → Continue with Section 1 (Branch Management)

## 1. Branch Management

### Common branch naming conventions:

**Standard Software Development:**
- `feature/user-authentication` - New feature implementation
- `bugfix/login-error` - Bug fixes
- `hotfix/security-patch` - Critical production fixes
- `refactor/database-schema` - Code refactoring
- `docs/api-documentation` - Documentation updates
- `test/unit-tests` - Test additions

**Infrastructure as Code (IaC):**
- `infra/aws-vpc-setup` - Infrastructure provisioning
- `infra/terraform-modules` - Terraform module development
- `config/k8s-deployment` - Kubernetes configurations
- `iac/cloudformation-stack` - CloudFormation templates
- `pipeline/ci-cd-setup` - CI/CD pipeline changes

**DevOps/Platform:**
- `deploy/staging-environment` - Deployment configurations
- `monitoring/prometheus-setup` - Monitoring and observability
- `security/secrets-rotation` - Security improvements

**Mobile/Frontend:**
- `ui/navigation-redesign` - UI/UX changes
- `feature/offline-mode` - Mobile features
- `perf/lazy-loading` - Performance optimizations

### Branch operations:
1. Check current branch and status
2. Ensure working directory is clean
3. Create new branch from appropriate base (main/develop)
4. Switch to new branch
5. Verify you're on the new branch

### Useful commands:
- List all branches: `git branch -a`
- Delete local branch: `git branch -d <branch-name>`
- Rename branch: `git branch -m <old-name> <new-name>`
- Check tracking: `git branch -vv`
- Switch branches: `git checkout <branch-name>` or `git switch <branch-name>`

### Best practices:
- Create separate branch for each feature/fix/task
- Use descriptive, meaningful branch names
- Keep branches focused on single responsibility
- Delete branches after merging to keep repo clean
- Pull latest changes from base branch before creating new branch
- Rebase or merge regularly to avoid conflicts

## 2. Security Pre-Commit Check

### Checks to perform before committing:
1. Scan for potential secrets (API keys, passwords, tokens)
2. Check for hardcoded credentials
3. Verify .gitignore is working correctly
4. Scan for sensitive file extensions
5. Check commit message quality
6. Validate IaC configurations (if applicable)

### Security patterns to detect:
- `API_KEY=`
- `PASSWORD=`
- `SECRET=`
- `aws_access_key_id`
- `private_key`
- `credential`
- `token=`
- `authorization:`
- Database connection strings with passwords

### Files that should NEVER be committed:

**Universal:**
- `.env`, `.env.local`, `.env.production` - Environment files
- `credentials.json`, `secrets.json` - Credential files
- `id_rsa`, `*.pem`, `*.key` - Private keys
- `*.p12`, `*.pfx` - Certificate files
- Config files with secrets or credentials
- Database files with real/production data

**IaC-Specific:**
- `terraform.tfstate`, `terraform.tfstate.backup` - Terraform state files (use remote backend)
- `*.tfvars` with sensitive values (use encrypted or remote)
- AWS credentials files
- `kubeconfig` with production credentials
- Ansible vault files (unencrypted)

**Development:**
- `node_modules/`, `vendor/` - Dependencies (use package managers)
- `.vscode/`, `.idea/` - IDE-specific files (unless team standard)
- Build artifacts, log files, temporary files

### Pre-commit security checklist:
□ No secrets in code
□ No real API keys or credentials
□ .gitignore updated appropriately
□ Sensitive files excluded
□ Test/mock data only (no production data)
□ Commit message follows guidelines
□ IaC files validated (terraform validate, yamllint, etc.)
□ No hardcoded IP addresses or endpoints (for IaC)

### If secrets detected:
1. Remove from working tree immediately
2. Add to .gitignore
3. Use environment variables instead
4. Use secrets management solutions:
   - **AWS**: Secrets Manager, Parameter Store, KMS
   - **Azure**: Key Vault
   - **GCP**: Secret Manager
   - **HashiCorp**: Vault
   - **Kubernetes**: Secrets, Sealed Secrets
5. For IaC: Use variables, data sources, or secret backends
6. **Never commit and push secrets!**
7. If already committed: Rotate credentials, use git-filter-repo to remove history

### Project-Specific Considerations:

**IaC Projects:**
- Validate syntax before committing (terraform validate, cfn-lint)
- Check for security misconfigurations (tfsec, checkov, terrascan)
- Review resource costs (infracost)
- Document infrastructure changes in commit message

**Application Projects:**
- Run linters and formatters
- Ensure tests pass locally
- Check for dependency vulnerabilities
- Review code for security issues (SAST tools)

## 3. Git Commit

### IMPORTANT: Check for project-specific commit guidelines:
- Look for `CLAUDE.md`, `CONTRIBUTING.md`, or `.github/CONTRIBUTING.md`
- Follow team conventions for commit messages
- Respect any pre-commit hooks configured
- Never include Claude Code signature unless explicitly allowed by project

### Commit workflow:
1. Check git status to see what files have changed
2. Review the changes with git diff
3. Stage appropriate files with git add (or git add -p for partial staging)
4. Create commit with descriptive message
5. Verify commit was created successfully

### Commit message format (Conventional Commits):
```
<type>(<scope>): <subject>

<body>

<footer>
```

**Types:**
- `feat:` - New feature
- `fix:` - Bug fix
- `docs:` - Documentation only
- `style:` - Formatting, missing semicolons, etc.
- `refactor:` - Code change that neither fixes a bug nor adds a feature
- `perf:` - Performance improvement
- `test:` - Adding or updating tests
- `chore:` - Maintenance tasks, dependencies
- `ci:` - CI/CD changes
- `build:` - Build system changes

**Subject line:**
- Brief summary (50 chars or less)
- Imperative mood ("Add" not "Added")
- No period at the end
- Capitalize first letter

### Example commit messages:

**Application Development:**
- `feat(auth): add JWT token validation`
- `fix(api): resolve null pointer in user service`
- `refactor(database): optimize query performance`
- `docs(readme): update installation instructions`
- `test(user): add unit tests for user model`

**Infrastructure as Code:**
- `feat(terraform): add VPC module for multi-az deployment`
- `fix(k8s): correct resource limits in deployment manifest`
- `chore(cloudformation): update AMI IDs for EC2 instances`
- `docs(iac): add architecture diagram and runbook`
- `ci(pipeline): integrate Terraform validation step`

**DevOps/Platform:**
- `feat(monitoring): add Prometheus alerting rules`
- `fix(cicd): resolve docker build caching issue`
- `chore(deps): update kubernetes to v1.28`
- `perf(build): optimize Docker layer caching`

### DO NOT include in commits (unless project allows):
- Emoji or special characters (unless team convention)
- Claude Code attribution
- Co-authored tags (unless actually co-authored)
- Excessive details (use PR description instead)

## 4. Git Push

### Push workflow:
1. Check current branch
2. Verify remote is configured correctly
3. Check if local branch tracks remote branch
4. Pull latest changes if behind remote (optional)
5. Push commits to remote
6. Verify push was successful

### Safety checks:
- **Never force push to main/master/production branches**
- Check for any uncommitted changes first
- Ensure you're pushing to correct remote (origin vs upstream)
- Verify branch name before pushing
- Review commits being pushed (git log origin/main..HEAD)
- For IaC: Double-check you're not pushing state files

### Common push scenarios:
- **First push**: `git push -u origin <branch-name>` (sets upstream tracking)
- **Regular push**: `git push`
- **New branch**: `git push -u origin <new-branch>`
- **Push with force (use with caution)**: `git push --force-with-lease` (safer than --force)
- **Push tags**: `git push --tags`

### If push is rejected:
1. Check if remote has changes you don't have
2. Pull latest changes: `git pull --rebase` (preferred) or `git pull`
3. Resolve any conflicts
4. Run tests if applicable
5. Push again

### Protected branches:
- Some repos have branch protection rules
- May require pull requests instead of direct push
- May need code review approvals
- CI/CD checks must pass
- Consider using `gh pr create` for creating pull requests

## Complete Workflow Steps

### Basic Flow:
1. **Navigate to Project** → `cd /path/to/project`
2. **Create Branch** → Create and switch to appropriate branch for your work
3. **Make Changes** → Edit files and implement features/fixes/infrastructure
4. **Security Check** → Run security validation before staging
5. **Stage Changes** → Add files to staging area
6. **Commit** → Create commit following guidelines
7. **Push** → Push commits to remote repository
8. **Pull Request** → Create PR for code review (if required)

### Extended Flow (with validation):
1. **Navigate to Project** → `cd /path/to/project`
2. **Update Base Branch** → `git checkout main && git pull`
3. **Create Branch** → `git checkout -b feature/my-feature`
4. **Make Changes** → Edit files
5. **Run Tests** → Ensure tests pass locally
6. **Security Check** → Scan for secrets and vulnerabilities
7. **Validate** → For IaC: terraform validate, yamllint, etc.
8. **Review Changes** → `git diff`
9. **Stage Changes** → `git add <files>`
10. **Commit** → `git commit -m "feat: add feature"`
11. **Update from Base** → `git rebase main` (if needed)
12. **Push** → `git push -u origin feature/my-feature`
13. **Create PR** → Use GitHub/GitLab UI or CLI
14. **Address Review** → Make changes based on feedback
15. **Merge** → Merge PR after approval
16. **Cleanup** → Delete feature branch

## Project Type Specific Notes

### IaC Projects (Terraform, CloudFormation, Kubernetes):
- Always validate before committing
- Use remote state backends (S3, Terraform Cloud)
- Never commit state files or plan files with sensitive data
- Document resource dependencies
- Use variables for environment-specific values
- Version lock providers and modules
- Run security scans (tfsec, checkov)
- Test in non-production first

### Web Applications:
- Run linters and formatters
- Ensure tests pass
- Build successfully before committing
- Check for console errors
- Review bundle size changes

### Mobile Applications:
- Test on multiple devices/simulators
- Check for backward compatibility
- Review permissions changes
- Test offline functionality

### API/Backend Services:
- Update API documentation
- Test endpoints locally
- Check for breaking changes
- Update changelog if needed

### Microservices:
- Consider service dependencies
- Update service contracts
- Version APIs appropriately
- Document inter-service changes

## Best Practices Summary

1. **Always work on feature branches**, never directly on main/master
2. **Pull before push** to avoid conflicts
3. **Commit often** with meaningful messages
4. **Never commit secrets** or sensitive data
5. **Review your changes** before committing
6. **Run tests** before pushing
7. **Keep commits atomic** - one logical change per commit
8. **Rebase to keep history clean** (when appropriate)
9. **Delete merged branches** to keep repo clean
10. **Follow team conventions** and project guidelines
11. **For IaC: validate, scan, test** before committing
12. **Document breaking changes** clearly in commit messages

Always follow this workflow order for secure, organized, and professional version control across all project types.
