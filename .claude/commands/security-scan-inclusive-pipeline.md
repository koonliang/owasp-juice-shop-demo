# Security Scan Pipeline Setup

You are tasked with integrating security scanning tools into an existing CI/CD pipeline. This slash command will:

1. **Accept user input** for the project summary file path
2. **Analyze the project** to determine appropriate security scanning tools
3. **Find the existing pipeline** configuration
4. **Create an enhanced pipeline** with security scanning stages integrated into the CI/CD flow

## Instructions

### Step 1: Gather Information

First, ask the user for the project summary file path using the AskUserQuestion tool:

```
Question: "Which project summary file should I use to determine the appropriate security scanning tools?"
Options:
  - "docs/juice-shop-app.txt" (Current Juice Shop project)
  - "Provide custom path" (Other project documentation)
```

### Step 2: Read and Analyze Files

Read the following files in parallel:
1. Project summary file (user-provided path)
2. `/docs/security-scan/security-scan-tools.txt` (security scanning tools reference)
3. Find existing pipeline files:
   - `.github/workflows/*.yml` (GitHub Actions)
   - `.gitlab-ci.yml` (GitLab CI)
   - `Jenkinsfile` (Jenkins)
   - Other CI/CD configurations

### Step 3: Create TODO List

Create a TODO list with the following tasks:
1. Analyzing project summary and technology stack
2. Reading security scan tools reference
3. Finding existing pipeline configuration
4. Archiving existing pipeline to workflows-archive
5. Designing enhanced pipeline with security stages
6. Creating enhanced pipeline file
7. Validating pipeline YAML syntax
8. Creating session summary document

### Step 3.5: Archive Existing Pipeline

Before creating the enhanced pipeline, archive the existing pipeline to avoid duplicate runs:

**For GitHub Actions:**
1. Create archive directory: `.github/workflows-archive/` (if it doesn't exist)
2. Find existing pipeline files in `.github/workflows/`
3. Move old pipeline(s) to archive folder using Bash tool:
   ```bash
   mkdir -p .github/workflows-archive
   mv .github/workflows/simple-pipeline.yml .github/workflows-archive/
   ```

**For GitLab CI:**
1. Create archive directory: `.gitlab/workflows-archive/` (if it doesn't exist)
2. Rename `.gitlab-ci.yml` to `.gitlab-ci.yml.disabled` or move to archive

**For Jenkins:**
1. Create archive directory: `jenkins/archive/` (if it doesn't exist)
2. Move `Jenkinsfile` to archive folder

**Important Notes:**
- Use the Bash tool to execute the mkdir and mv commands
- Inform the user which pipeline was archived
- Mark the TODO item "Archiving existing pipeline to workflows-archive" as completed after archiving
- If no existing pipeline is found, skip this step and inform the user

### Step 4: Analyze Project & Determine Tools

Based on the project summary, determine:

**Technology Stack Analysis:**
- Backend framework (Node.js, Python, Java, Go, etc.)
- Frontend framework (Angular, React, Vue, etc.)
- Database type (SQLite, PostgreSQL, MongoDB, etc.)
- Container usage (Docker, Kubernetes, etc.)
- Cloud deployment (AWS, GCP, Azure, K8s, etc.)

**Deployment Path Selection:**
- If Kubernetes deployment detected → Use PATH 1 (K8s Security)
- If AWS ECS deployment detected → Use PATH 2 (ECS Security)
- Default → PATH 1 (K8s Security)

**Security Tools Selection** (from security-scan-tools.txt):

**TEST Stage (Blocking):**
1. SAST (CodeQL, Semgrep, or language-specific)
2. Secret Detection (TruffleHog, detect-secrets, or Gitleaks)
3. Unit Tests (existing)

**SECURITY-SCAN Stage (Individual Jobs - Parallel execution):**
4. npm-audit job (dependency vulnerability scanning)
5. retirejs job (JavaScript library scanning)
6. gitleaks job (git history secret scanning)
7. dockerfile-policy job (OPA/Conftest policy enforcement)

**IMPORTANT NOTE - Trivy Placement:**
- ⚠️  **Trivy should NOT be in Stage 3 (parallel with code scans)**
- ✅ **Correct placement: Stage 5b (after Build, before Push)**
- **Why**: Build once, scan the actual image, then push the scanned image
- **Optimal Flow**: Build → Trivy (security gate) → Push → Deploy

**SONAR Stage (Manual gate):**
9. SonarQube/SonarCloud (code quality + security)

**K8S-SECURITY Stage (for K8s deployments):**
10. OWASP ZAP (DAST baseline scan)
11. Security Headers check

**Optional K8s Tools (Reference):**
- KubeSec (K8s manifest security)
- KubeBench (CIS benchmark)
- KubeScan (RBAC analysis)
- Falco (runtime security)

### Step 5: Design Enhanced Pipeline

Create an enhanced pipeline that integrates security scanning into the existing flow:

**Existing Flow (from simple-pipeline.yml):**
```
Lint → Test → Build → Push → Deploy
```

**Enhanced Flow with Security Scanning (Parallel Optimized):**
```
Lint
  ↓
  ├─ SAST (parallel) ──────────┐
  ├─ Secret Detection (parallel)┼─→ Security-Scan (parallel) → Sonar → Build → Push → Deploy → K8s-Security → Consolidate Reports
  ├─ Unit Tests (parallel) ─────┤
  └─────────────────────────────┘
```

**Note:** Dockerfile Policy is now part of Security-Scan matrix (runs in parallel with other security scans)

**Stage Breakdown:**

1. **Lint Stage** (existing) - Keep as is
2. **Stage 2 - Parallel Execution** (optimized for performance):
   - **2a. SAST** - Static analysis with npm audit (with artifact upload)
   - **2b. Secret Detection** - Git history secret scanning (with artifact upload)
   - **2c. Unit Tests** - Existing test suite
3. **Security-Scan Stage** (NEW) - Individual jobs running in parallel:
   - **npm-audit** - Dependency vulnerability scanning
   - **retirejs** - JavaScript library vulnerability scanning
   - **gitleaks** - Git history secret scanning
   - **dockerfile-policy** - OPA/Conftest with comprehensive security rules
   - **Each job uploads its own artifact independently**
   - **Note**: Trivy is NOT in this stage (see Stage 5b below)
4. **Sonar Stage** (NEW) - SonarQube analysis with manual gate
5. **Build Stage** (existing) - Docker image build
5b. **Trivy Container Scan** (NEW - Security Gate) - Scan built image BEFORE push
   - ✅ **Best Practice**: Scans the actual image that will be deployed
   - Loads image from Build stage artifact
   - Scans for OS vulnerabilities (Alpine, Debian, Ubuntu packages)
   - Runs in warning mode (continue-on-error: true)
   - Blocks push if critical issues found (optional strict mode)
6. **Push Stage** (existing) - Push to Docker Hub (only if Trivy passes)
7. **Deploy Stage** (existing) - K8s deployment
8. **K8s-Security Stage** (NEW) - Post-deployment DAST:
   - K8s manifest security scanning (with artifact upload)
   - OWASP ZAP baseline scan (with artifact upload)
   - Security headers check
9. **Consolidate Reports Stage** (NEW) - Aggregate all security findings:
   - Download all security scan artifacts
   - Run consolidation script to parse findings
   - Generate HTML dashboard and Markdown report
   - Upload consolidated reports (90-day retention)

**Performance Optimization:**
- Stage 2 jobs run in parallel (40-60% faster than sequential)
- Stage 3 uses individual jobs (not matrix) for clearer visibility (5 scans run simultaneously)
- Each security scan uploads its own artifact independently
- Consolidation runs even if previous stages fail (if: always())
- Health checks use continue-on-error to ensure scan steps always run

### Step 6: Create Enhanced Pipeline File

Create a new pipeline file: `.github/workflows/enhanced-pipeline.yml`

**Pipeline Structure Guidelines:**

```yaml
name: Enhanced Pipeline with Security Scanning

on:
  push:
    branches: [main, master]
  pull_request:
    branches: [main, master]

jobs:
  # Stage 1: Lint (existing)
  lint:
    name: Lint Code
    runs-on: ubuntu-latest
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '22'
      - name: Install dependencies
        run: npm install --prefer-offline --no-audit
        # Note: Use 'npm install' if package-lock.json is not committed
        #       Use 'npm ci' if package-lock.json exists for faster, reproducible builds
      - name: Run linter
        run: npm run lint
        continue-on-error: true

  # Stage 2a: SAST (Parallel)
  sast:
    name: SAST - Static Analysis
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '22'

      - name: Install dependencies
        run: npm install --prefer-offline --no-audit

      - name: Run SAST (npm audit)
        run: |
          echo "🔍 Running SAST scan..."
          # TODO: Implement CodeQL or Semgrep
          # For now, use npm audit as basic SAST
          npm audit --audit-level=high > npm-audit-sast.txt 2>&1 || echo "⚠️  Vulnerabilities found (warning mode)"
          cat npm-audit-sast.txt
        continue-on-error: true  # Warning mode for training

      - name: Upload SAST Report
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: sast-report
          path: npm-audit-sast.txt
          retention-days: 30

  # Stage 2b: Secret Detection (Parallel)
  secret-detection:
    name: Secret Detection
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0  # Full history for secret scanning

      - name: Run Secret Detection
        run: |
          echo "🔐 Scanning for secrets in git history..."
          # TODO: Implement TruffleHog or detect-secrets
          # For now, use basic git secrets check
          git log --all --pretty=format: --name-only | sort -u | xargs grep -i "password\|secret\|key" > secret-detection.txt 2>&1 || echo "✅ No obvious secrets found"
          cat secret-detection.txt || echo "No secrets file generated"
        continue-on-error: true  # Warning mode for training

      - name: Upload Secret Detection Report
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: secret-detection-report
          path: secret-detection.txt
          retention-days: 30

  # Stage 2c: Unit Tests (Parallel)
  test:
    name: Unit Tests
    runs-on: ubuntu-latest
    needs: lint
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '22'

      - name: Install dependencies
        run: |
          npm install --prefer-offline --no-audit
          cd frontend && npm install --prefer-offline --no-audit

      - name: Run unit tests
        run: npm test
        continue-on-error: true  # Warning mode for training

  # Stage 3: Security Scan (Individual Jobs - runs after all Stage 2 jobs)
  # Stage 3a: npm Audit
  npm-audit:
    name: npm Audit
    runs-on: ubuntu-latest
    needs: [sast, secret-detection, test]
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '22'

      - name: Run npm audit
        run: |
          echo "📦 Running npm audit..."
          npm audit --audit-level=moderate > npm-audit.txt 2>&1 || echo "⚠️  Vulnerabilities found (warning mode)"
          cat npm-audit.txt
        continue-on-error: true

      - name: Upload npm Audit Report
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: security-scan-npm-audit
          path: npm-audit.txt
          retention-days: 30

  # Stage 3b: RetireJS
  retirejs:
    name: RetireJS
    runs-on: ubuntu-latest
    needs: [sast, secret-detection, test]
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '22'

      - name: Run RetireJS
        run: |
          echo "📚 Running RetireJS scan..."
          npm install -g retire
          retire --path . > retirejs-scan.txt 2>&1 || echo "⚠️  Vulnerable libraries found (warning mode)"
          cat retirejs-scan.txt
        continue-on-error: true

      - name: Upload RetireJS Report
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: security-scan-retirejs
          path: retirejs-scan.txt
          retention-days: 30

  # Stage 3c: Gitleaks
  gitleaks:
    name: Gitleaks Secrets
    runs-on: ubuntu-latest
    needs: [sast, secret-detection, test]
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Run Gitleaks
        run: |
          echo "🔐 Running Gitleaks secret scan..."
          docker run --rm -v $(pwd):/repo zricethezav/gitleaks:latest detect --source /repo --verbose > gitleaks-report.txt 2>&1 || echo "⚠️  Secrets found (warning mode)"
          cat gitleaks-report.txt || echo "No secrets detected"
        continue-on-error: true

      - name: Upload Gitleaks Report
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: security-scan-gitleaks
          path: gitleaks-report.txt
          retention-days: 30

  # Stage 3d: Dockerfile Policy
  dockerfile-policy:
    name: Dockerfile Policy (OPA)
    runs-on: ubuntu-latest
    needs: [sast, secret-detection, test]
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Run Dockerfile Policy Scan
        run: |
          echo "📋 Running OPA/Conftest policy validation..."
          echo "Using comprehensive security policy from docs/docker/dockerfile-security.rego"
          echo "Using docker-based conftest (no binary installation needed)"
          echo ""

          echo "🔍 Testing Dockerfile against comprehensive security policies..." | tee dockerfile-policy.txt
          echo "" | tee -a dockerfile-policy.txt
          echo "Security checks include:" | tee -a dockerfile-policy.txt
          echo "  ✓ No secrets in ENV variables" | tee -a dockerfile-policy.txt
          echo "  ✓ Use trusted base images only" | tee -a dockerfile-policy.txt
          echo "  ✓ No 'latest' tag for base images" | tee -a dockerfile-policy.txt
          echo "  ✓ Avoid curl bashing" | tee -a dockerfile-policy.txt
          echo "  ✓ No system package upgrades in Dockerfile" | tee -a dockerfile-policy.txt
          echo "  ✓ Use COPY instead of ADD" | tee -a dockerfile-policy.txt
          echo "  ✓ Don't run as root user" | tee -a dockerfile-policy.txt
          echo "  ✓ Don't use sudo command" | tee -a dockerfile-policy.txt
          echo "  ✓ Use multi-stage builds" | tee -a dockerfile-policy.txt
          echo "" | tee -a dockerfile-policy.txt

          # Use docker-based conftest with comprehensive policy
          set +e  # Disable exit on error for proper handling
          docker run --rm -v $(pwd):/project \
            openpolicyagent/conftest:latest \
            test --policy ./docs/docker/dockerfile-security.rego Dockerfile 2>&1 | tee -a dockerfile-policy.txt
          CONFTEST_EXIT_CODE=$?
          set -e  # Re-enable exit on error

          echo "" | tee -a dockerfile-policy.txt
          if [ $CONFTEST_EXIT_CODE -ne 0 ]; then
            echo "⚠️  Policy violations found (exit code: $CONFTEST_EXIT_CODE)" | tee -a dockerfile-policy.txt
            echo "📝 This is expected for training - Juice Shop is intentionally vulnerable" | tee -a dockerfile-policy.txt
            echo "📚 Reference: https://cloudberry.engineering/article/dockerfile-security-best-practices/" | tee -a dockerfile-policy.txt
            echo "✅ Pipeline continues in warning mode" | tee -a dockerfile-policy.txt
          else
            echo "✅ All Dockerfile policies passed!" | tee -a dockerfile-policy.txt
          fi
        continue-on-error: true

      - name: Upload Dockerfile Policy Report
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: security-scan-dockerfile-policy
          path: dockerfile-policy.txt
          retention-days: 30

  # Stage 4: SonarQube (NEW - Manual gate)
  sonar:
    name: SonarQube Code Quality
    runs-on: ubuntu-latest
    needs: [npm-audit, retirejs, gitleaks, dockerfile-policy]
    steps:
      - name: Checkout code
        uses: actions/checkout@v4
        with:
          fetch-depth: 0

      - name: Setup Node.js
        uses: actions/setup-node@v4
        with:
          node-version: '22'

      - name: Install dependencies
        run: npm install

      - name: SonarQube Scan (placeholder)
        run: |
          echo "📊 SonarQube analysis would run here"
          echo "⚠️  Note: SonarQube requires server setup"
          echo "   For now, this is a placeholder for future integration"
          # TODO: Implement SonarCloud or self-hosted SonarQube
        continue-on-error: true

  # Stage 5: Build (existing)
  build:
    name: Build & Tag Docker Image
    runs-on: ubuntu-latest
    needs: sonar
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Set up Docker Buildx
        uses: docker/setup-buildx-action@v3

      - name: Build Docker image
        run: |
          docker build -t juice-shop:latest .
          echo "✅ Docker image built successfully!"

      - name: Tag images
        run: |
          docker tag juice-shop:latest juice-shop:${{ github.sha }}
          docker tag juice-shop:latest juice-shop:enhanced
          echo "🏷️  Images tagged"

      - name: Save Docker image as artifact
        run: |
          docker save juice-shop:latest -o juice-shop-image.tar
          echo "💾 Docker image saved"

      - name: Upload Docker image artifact
        uses: actions/upload-artifact@v4
        with:
          name: docker-image
          path: juice-shop-image.tar
          retention-days: 1

  # Stage 5b: Trivy Container Scan (Security Gate Before Push)
  trivy:
    name: Trivy - Scan Built Image
    runs-on: ubuntu-latest
    needs: build
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Download Docker image artifact
        uses: actions/download-artifact@v4
        with:
          name: docker-image

      - name: Load Docker image
        run: |
          echo "📦 Loading built Docker image..."
          docker load -i juice-shop-image.tar
          echo "✅ Image loaded: juice-shop:latest"

      - name: Run Trivy Container Scan
        run: |
          echo "🐳 Running Trivy container security scan..."
          echo "⚠️  BEST PRACTICE: Scanning the actual image that will be pushed"
          echo "   (Not building a separate image - more efficient and accurate)"
          echo ""

          echo "=== Trivy Scan - HIGH Severity (Warning) ===" | tee trivy-scan.txt
          docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
            -v $HOME/.cache:/root/.cache/ \
            aquasec/trivy:latest image --exit-code 0 --severity HIGH --light juice-shop:latest \
            2>&1 | tee -a trivy-scan.txt \
            || echo "⚠️  HIGH severity vulnerabilities found"

          echo "" | tee -a trivy-scan.txt
          echo "=== Trivy Scan - CRITICAL Severity ===" | tee -a trivy-scan.txt
          docker run --rm -v /var/run/docker.sock:/var/run/docker.sock \
            -v $HOME/.cache:/root/.cache/ \
            aquasec/trivy:latest image --exit-code 0 --severity CRITICAL --light juice-shop:latest \
            2>&1 | tee -a trivy-scan.txt \
            || echo "⚠️  CRITICAL vulnerabilities found (warning mode for training)"

          echo "" | tee -a trivy-scan.txt
          echo "📊 Trivy scan completed" | tee -a trivy-scan.txt
          echo "⚠️  Note: Pipeline continues even with vulnerabilities (warning mode)"
          echo "🚀 Proceeding to push stage..."
        continue-on-error: true

      - name: Upload Trivy Report
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: container-scan-trivy
          path: trivy-scan.txt
          retention-days: 30

  # Stage 6: Push (existing)
  push-to-dockerhub:
    name: Push to Docker Hub
    runs-on: ubuntu-latest
    needs: trivy
    env:
      DOCKER_USERNAME: rupeedev
      DOCKER_REPO: owasp-juice-shop
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Download Docker image artifact
        uses: actions/download-artifact@v4
        with:
          name: docker-image

      - name: Load Docker image
        run: docker load -i juice-shop-image.tar

      - name: Log in to Docker Hub
        uses: docker/login-action@v3
        with:
          username: ${{ env.DOCKER_USERNAME }}
          password: ${{ secrets.DOCKER_PASSWORD }}

      - name: Tag and push images
        run: |
          docker tag juice-shop:latest ${{ env.DOCKER_USERNAME }}/${{ env.DOCKER_REPO }}:latest
          docker tag juice-shop:latest ${{ env.DOCKER_USERNAME }}/${{ env.DOCKER_REPO }}:${{ github.sha }}
          docker tag juice-shop:latest ${{ env.DOCKER_USERNAME }}/${{ env.DOCKER_REPO }}:enhanced

          docker push ${{ env.DOCKER_USERNAME }}/${{ env.DOCKER_REPO }}:latest
          docker push ${{ env.DOCKER_USERNAME }}/${{ env.DOCKER_REPO }}:${{ github.sha }}
          docker push ${{ env.DOCKER_USERNAME }}/${{ env.DOCKER_REPO }}:enhanced

  # Stage 7: Deploy (existing)
  deploy:
    name: Deploy to Kubernetes
    runs-on: self-hosted
    needs: push-to-dockerhub
    env:
      DOCKER_USERNAME: rupeedev
      DOCKER_REPO: owasp-juice-shop
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Install kubectl
        run: |
          if ! command -v kubectl &> /dev/null; then
            curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
            chmod +x kubectl
            sudo mv kubectl /usr/local/bin/
          fi

      - name: Apply manifests
        run: |
          kubectl apply -f kind-k8s-cluster/juice-shop-deployment.yaml
          kubectl apply -f kind-k8s-cluster/juice-shop-service.yaml

      - name: Update deployment
        run: |
          kubectl set image deployment/juice-shop juice-shop=${{ env.DOCKER_USERNAME }}/${{ env.DOCKER_REPO }}:${{ github.sha }}

      - name: Wait for rollout
        run: |
          kubectl rollout status deployment/juice-shop --timeout=180s

      - name: Verify deployment
        run: |
          kubectl get pods -l app=juice-shop
          kubectl get svc juice-shop
          kubectl get deployment juice-shop

      - name: Get service endpoint
        run: |
          echo "🌐 Application deployed at: http://localhost:30080"
        id: endpoint

  # Stage 8: K8s Security (NEW - DAST with Parallel Execution)
  # Note: Split into multiple jobs for parallel execution

  # Stage 8a: K8s Manifest Security (Parallel - doesn't need app running)
  k8s-manifest-scan:
    name: K8s Manifest Security
    runs-on: ubuntu-latest  # Use ubuntu-latest for Docker access
    needs: deploy
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Run K8s Manifest Security Scan
        run: |
          echo "🔐 Scanning Kubernetes manifests for security issues..." | tee k8s-manifest-scan.txt
          echo "" | tee -a k8s-manifest-scan.txt

          # Scan deployment and service manifests with OPA Conftest
          echo "=== Scanning K8s Deployment Manifest ===" | tee -a k8s-manifest-scan.txt
          docker run --rm -v $(pwd):/project \
            openpolicyagent/conftest:latest \
            test --policy ./policy kind-k8s-cluster/juice-shop-deployment.yaml \
            2>&1 | tee -a k8s-manifest-scan.txt \
            || echo "⚠️  Policy violations in deployment manifest" | tee -a k8s-manifest-scan.txt

          echo "" | tee -a k8s-manifest-scan.txt
          echo "=== Scanning K8s Service Manifest ===" | tee -a k8s-manifest-scan.txt
          docker run --rm -v $(pwd):/project \
            openpolicyagent/conftest:latest \
            test --policy ./policy kind-k8s-cluster/juice-shop-service.yaml \
            2>&1 | tee -a k8s-manifest-scan.txt \
            || echo "⚠️  Policy violations in service manifest" | tee -a k8s-manifest-scan.txt

          echo "" | tee -a k8s-manifest-scan.txt
          echo "📝 K8s manifest scan checks for:" | tee -a k8s-manifest-scan.txt
          echo "   - Containers running as non-root (runAsNonRoot)" | tee -a k8s-manifest-scan.txt
          echo "   - Security context settings (allowPrivilegeEscalation)" | tee -a k8s-manifest-scan.txt
          echo "   - Service type configurations" | tee -a k8s-manifest-scan.txt
          echo "   - Resource limits and requests" | tee -a k8s-manifest-scan.txt
          echo "   - Image tag best practices (no 'latest' tag)" | tee -a k8s-manifest-scan.txt
          echo "📚 Reference: Best practices from DevSecOps patterns" | tee -a k8s-manifest-scan.txt
        continue-on-error: true

      - name: Upload K8s Manifest Scan Report
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: k8s-security-k8s-manifest
          path: k8s-manifest-scan.txt
          retention-days: 30

  # Stage 8b: Security Headers & DAST (Parallel - needs app running)
  security-headers-scan:
    name: Security Headers Analysis
    runs-on: self-hosted  # Needs access to deployed app
    needs: deploy
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Install kubectl (if not present)
        run: |
          if ! command -v kubectl &> /dev/null; then
            echo "📥 Installing kubectl..."
            curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
            chmod +x kubectl
            sudo mv kubectl /usr/local/bin/
            echo "✅ kubectl installed"
          else
            echo "✅ kubectl already installed"
          fi

      - name: Wait for application readiness
        run: |
          echo "⏳ Waiting for application to be ready..."
          sleep 30
          kubectl wait --for=condition=ready pod -l app=juice-shop --timeout=120s

      # Application Health Check
      - name: Application Health Check
        run: |
          echo "🏥 Checking application health..."
          echo "Waiting for application to start responding..."

          MAX_RETRIES=10
          RETRY_COUNT=0
          SUCCESS=false

          while [ $RETRY_COUNT -lt $MAX_RETRIES ]; do
            echo "Attempt $((RETRY_COUNT + 1))/$MAX_RETRIES..."

            # Capture both HTTP code and curl exit code
            HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:30080 2>&1 || echo "000")

            if [ "$HTTP_CODE" = "200" ]; then
              echo "✅ Application is responding (HTTP $HTTP_CODE)"
              SUCCESS=true
              break
            elif [ "$HTTP_CODE" = "000" ]; then
              echo "⚠️  Connection failed - application not ready yet"
            else
              echo "⚠️  Application returned HTTP $HTTP_CODE"
            fi

            RETRY_COUNT=$((RETRY_COUNT + 1))
            if [ $RETRY_COUNT -lt $MAX_RETRIES ]; then
              echo "Waiting 10 seconds before retry..."
              sleep 10
            fi
          done

          if [ "$SUCCESS" = false ]; then
            echo "❌ Application health check failed after $MAX_RETRIES attempts"
            echo "Checking pod status:"
            kubectl get pods -l app=juice-shop
            kubectl describe pod -l app=juice-shop | tail -50
            exit 1
          fi

      # K8s Manifest Security Scan
      - name: K8s Manifest Security Scan
        run: |
          echo "🔐 Scanning Kubernetes manifests for security issues..." | tee k8s-manifest-scan.txt
          echo "" | tee -a k8s-manifest-scan.txt

          # Scan deployment and service manifests with OPA Conftest
          echo "=== Scanning K8s Deployment Manifest ===" | tee -a k8s-manifest-scan.txt
          docker run --rm -v $(pwd):/project \
            openpolicyagent/conftest:latest \
            test --policy ./policy kind-k8s-cluster/juice-shop-deployment.yaml \
            2>&1 | tee -a k8s-manifest-scan.txt \
            || echo "⚠️  Policy violations in deployment manifest" | tee -a k8s-manifest-scan.txt

          echo "" | tee -a k8s-manifest-scan.txt
          echo "=== Scanning K8s Service Manifest ===" | tee -a k8s-manifest-scan.txt
          docker run --rm -v $(pwd):/project \
            openpolicyagent/conftest:latest \
            test --policy ./policy kind-k8s-cluster/juice-shop-service.yaml \
            2>&1 | tee -a k8s-manifest-scan.txt \
            || echo "⚠️  Policy violations in service manifest" | tee -a k8s-manifest-scan.txt

          echo "" | tee -a k8s-manifest-scan.txt
          echo "📝 K8s manifest scan checks for:" | tee -a k8s-manifest-scan.txt
          echo "   - Containers running as non-root (runAsNonRoot)" | tee -a k8s-manifest-scan.txt
          echo "   - Security context settings" | tee -a k8s-manifest-scan.txt
          echo "   - Service type configurations" | tee -a k8s-manifest-scan.txt
          echo "   - Resource limits and requests" | tee -a k8s-manifest-scan.txt
          echo "📚 Reference: Best practices from DevSecOps patterns" | tee -a k8s-manifest-scan.txt
        continue-on-error: true

      # Security Headers Check
      - name: Security Headers Analysis
        run: |
          echo "🔒 Analyzing HTTP security headers..."
          echo ""
          echo "=== Current Security Headers ==="
          curl -s -D - http://localhost:30080 -o /dev/null 2>&1 | grep -E "^X-|^Content-Security|^Strict-Transport" || true
          echo ""
          echo "=== Security Header Assessment ==="

          HEADERS=$(curl -s -D - http://localhost:30080 -o /dev/null 2>&1)

          # Check for present headers
          echo "$HEADERS" | grep -q "X-Content-Type-Options" && echo "✓ X-Content-Type-Options: Present" || echo "✗ X-Content-Type-Options: Missing"
          echo "$HEADERS" | grep -q "X-Frame-Options" && echo "✓ X-Frame-Options: Present" || echo "✗ X-Frame-Options: Missing"
          echo "$HEADERS" | grep -q "Content-Security-Policy" && echo "✓ Content-Security-Policy: Present" || echo "✗ Content-Security-Policy: Missing (expected)"
          echo "$HEADERS" | grep -q "Strict-Transport-Security" && echo "✓ Strict-Transport-Security: Present" || echo "✗ Strict-Transport-Security: Missing (expected)"
          echo "$HEADERS" | grep -q "X-XSS-Protection" && echo "✓ X-XSS-Protection: Present" || echo "✗ X-XSS-Protection: Missing (expected)"
          echo ""
          echo "⚠️  Note: Juice Shop is intentionally vulnerable - missing headers are expected"
        continue-on-error: true

      # OWASP ZAP Baseline Scan (Optional - can be resource intensive)
      - name: OWASP ZAP Baseline Scan
        run: |
          echo "🔍 Running OWASP ZAP baseline scan..."
          echo "Note: This may take 2-5 minutes..."

          # Use correct ZAP docker image: zaproxy/zap-stable (NOT owasp/zap2docker-stable)
          timeout 300 docker run --rm --network host \
            zaproxy/zap-stable:latest \
            zap-baseline.py -t http://localhost:30080 -I -r zap-report.html \
            || echo "⚠️  Security issues found or scan timed out (warning mode)"

          echo "📊 ZAP scan completed"
          echo "⚠️  Findings are expected for intentionally vulnerable application"
        continue-on-error: true

      - name: Security scan complete
        run: |
          echo ""
          echo "✅ K8s Security scans completed"
          echo "📊 Review DAST results for security findings"
          echo "⚠️  Note: Juice Shop is intentionally vulnerable - findings are expected"
          echo ""

      - name: Upload K8s Security Reports
        uses: actions/upload-artifact@v4
        if: always()
        with:
          name: k8s-security-reports
          path: |
            k8s-manifest-scan.txt
            zap-report.html
          retention-days: 30

  # Stage 9: Security Report Consolidation
  consolidate-reports:
    name: Consolidate Security Reports
    runs-on: ubuntu-latest
    needs: k8s-security
    if: always()  # Run even if previous stages have failures
    steps:
      - name: Checkout code
        uses: actions/checkout@v4

      - name: Create reports directory
        run: mkdir -p security-reports

      - name: Download SAST Report
        uses: actions/download-artifact@v4
        continue-on-error: true
        with:
          name: sast-report
          path: security-reports/

      - name: Download Secret Detection Report
        uses: actions/download-artifact@v4
        continue-on-error: true
        with:
          name: secret-detection-report
          path: security-reports/

      - name: Download Security Scan Reports
        uses: actions/download-artifact@v4
        continue-on-error: true
        with:
          pattern: security-scan-*
          path: security-reports/
          merge-multiple: true

      - name: Download K8s Security Reports
        uses: actions/download-artifact@v4
        continue-on-error: true
        with:
          name: k8s-security-reports
          path: security-reports/

      - name: List downloaded reports
        run: |
          echo "📋 Downloaded security reports:"
          ls -lah security-reports/
          echo ""
          echo "Report files:"
          find security-reports/ -type f

      - name: Run consolidation script
        run: |
          chmod +x .github/scripts/consolidate-security-reports.sh
          .github/scripts/consolidate-security-reports.sh security-reports consolidated-reports

      - name: Display consolidation summary
        run: |
          echo ""
          echo "================================================================"
          echo "📊 Security Report Consolidation Summary"
          echo "================================================================"
          echo ""
          if [ -f "consolidated-reports/summary.txt" ]; then
            echo "Tool Summary:"
            cat consolidated-reports/summary.txt
          fi
          echo ""
          echo "Generated Reports:"
          ls -lh consolidated-reports/
          echo ""

      - name: Upload Consolidated Reports
        uses: actions/upload-artifact@v4
        with:
          name: consolidated-security-reports
          path: |
            consolidated-reports/security-dashboard.html
            consolidated-reports/security-report.md
            consolidated-reports/summary.txt
          retention-days: 90  # Keep consolidated reports longer

  # Final success message
  pipeline-complete:
    name: Pipeline Complete
    runs-on: ubuntu-latest
    needs: consolidate-reports
    steps:
      - name: Success message
        run: |
          echo "🎉 Enhanced Pipeline completed successfully!"
          echo ""
          echo "✅ Pipeline Stages (Parallel Execution Optimized):"
          echo "   Stage 1: Lint ✓"
          echo "   Stage 2: SAST + Secret Detection + Unit Tests (parallel) ✓"
          echo "   Stage 3: Security Scan (Gitleaks, RetireJS, npm-audit - parallel) ✓"
          echo "           + Dockerfile Policy Validation ✓"
          echo "   Stage 4: SonarQube Analysis ✓"
          echo "   Stage 5: Build & Tag Docker Image ✓"
          echo "   Stage 5b: Trivy Container Scan (Security Gate) ✓"
          echo "   Stage 6: Push to Docker Hub ✓"
          echo "   Stage 7: Deploy to Kubernetes ✓"
          echo "   Stage 8: K8s Security (DAST with OWASP ZAP) ✓"
          echo "   Stage 9: Security Report Consolidation ✓"
          echo ""
          echo "⚡ Parallel execution: SAST, Secrets, Tests, and Policy checks run simultaneously"
          echo "🔒 Security scanning integrated at multiple stages"
          echo "📦 Application deployed to Kubernetes"
          echo "🌐 Access at: http://localhost:30080"
          echo ""
          echo "📊 Consolidated Security Reports Available:"
          echo "   Download the 'consolidated-security-reports' artifact to view:"
          echo "   • security-dashboard.html - Interactive HTML dashboard"
          echo "   • security-report.md - Markdown summary report"
          echo "   • summary.txt - Quick statistics"
```

### Step 7: Validate Pipeline

After creating the enhanced pipeline:
1. Validate YAML syntax using yamllint
2. Check for common pipeline issues
3. Verify all job dependencies are correct
4. Ensure environment variables and secrets are properly referenced

### Step 8: Create Session Summary

Create a comprehensive session summary document at `/docs/Summary/security-scan-setup-summary.txt`:

**Summary Structure:**
```
================================================================================
SECURITY SCAN PIPELINE SETUP - SESSION SUMMARY
================================================================================

TL;DR
=====
Integrated comprehensive security scanning with consolidated reporting into existing
CI/CD pipeline for [PROJECT_NAME]. Created enhanced pipeline with 9 stages including
SAST, SCA, container scanning, policy validation, DAST, and automated report
consolidation. All security stages configured in warning mode for training/educational
purposes. Generates downloadable HTML dashboard and Markdown reports.

PROJECT ANALYSIS
================
Technology Stack:
- [List technologies from project summary]

Deployment Path:
- [K8s or ECS]

PIPELINE TRANSFORMATION
=======================
Before (Simple Pipeline):
Lint → Test → Build → Push → Deploy (5 stages)

After (Enhanced Pipeline with Security - Parallel Optimized):
Lint
  ↓
  ├─ SAST (parallel) ──────────┐
  ├─ Secret Detection (parallel)┼─→ Security-Scan (parallel) → Sonar → Build → Push → Deploy → K8s-Security → Consolidate Reports
  ├─ Unit Tests (parallel) ─────┤
  └─────────────────────────────┘

**Note**: Dockerfile Policy now runs in Security-Scan matrix (parallel with other scans)

Performance:
- Stage 2 jobs run in parallel (40-60% faster than sequential)
- Security-Scan uses matrix strategy (5 scans simultaneously)
- Consolidation runs even if previous stages fail (if: always())

Pipeline Archiving:
- Moved existing pipeline to .github/workflows-archive/ to avoid duplicate runs
- Only the enhanced pipeline will run automatically on push/PR events

SECURITY TOOLS INTEGRATED
==========================
[List all tools with their stages and purposes]

FILES CREATED/MODIFIED
=======================
1. .github/workflows/enhanced-pipeline.yml - Enhanced pipeline with 9 security stages
2. .github/scripts/consolidate-security-reports.sh - Security report consolidation script
3. docs/docker/dockerfile-security.rego - Comprehensive Dockerfile security policy (9 checks)
4. docs/docker/README.md - Dockerfile security policy documentation
5. .github/workflows-archive/ - Archive directory (created)
6. .github/workflows-archive/simple-pipeline.yml - Archived old pipeline (moved)
7. docs/Summary/security-scan-setup-summary.txt - This summary

**Consolidated Reports Generated** (downloadable artifacts):
- security-dashboard.html - Interactive HTML dashboard with severity breakdown
- security-report.md - Markdown summary for GitHub-friendly viewing
- summary.txt - Quick statistics per tool

NEXT STEPS
==========
[List recommended next steps]

================================================================================
END OF SUMMARY
================================================================================
```

### Step 9: Present Results to User

Show the user:
1. Summary of changes made
2. Pipeline archiving details (what was moved to .github/workflows-archive/)
3. New enhanced pipeline file location
4. Security tools integrated
5. How to trigger the enhanced pipeline
6. Expected results and warnings
7. Next steps for advanced security integration

## Important Notes

1. **Training Mode**: All security scans use `continue-on-error: true` for educational purposes
2. **K8s Focus**: Pipeline uses K8s-focused security tools (as per security-scan-tools.txt)
3. **Parallel Execution**:
   - Stage 2 jobs (SAST, Secrets, Tests, Policy) run in parallel for 40-60% faster execution
   - Security-Scan stage uses matrix strategy for parallel tool execution
   - Stage 8 split into parallel jobs (K8s Manifest, Security Headers, OWASP ZAP)
4. **Blocking vs Warning**:
   - SAST, Secret Detection, Tests: Warning mode (educational)
   - SECURITY-SCAN stage: Warning mode (non-blocking)
   - K8S-SECURITY stage: Warning mode (DAST findings)
5. **Intentional Vulnerabilities**: Juice Shop is intentionally vulnerable - security findings are expected
6. **OPA Policy Files** (IMPORTANT):
   - **Syntax**: Use OPA Rego v1 syntax (OPA 0.50+)
   - **Required keywords**: `contains` and `if` in all deny/warn rules
   - **Assignments**: Use `:=` instead of `=`
   - **Dockerfile policies**: `policy/dockerfile.rego` (container build security)
   - **K8s policies**: `policy/kubernetes.rego` (deployment manifest security)
   - **Validation**: Test locally with `conftest test` before pipeline run
7. **K8s Security Stage** (DAST with Parallel Execution):
   - **Stage 8a - K8s Manifest Scan**:
     - Runs on `ubuntu-latest` (has Docker pre-installed)
     - Scans deployment and service manifests with OPA/Conftest
     - Validates runAsNonRoot, allowPrivilegeEscalation, resource limits
     - Checks for 'latest' image tags, LoadBalancer exposure
     - Doesn't need access to running application
   - **Stage 8b - Security Headers**:
     - Runs on `self-hosted` (needs app access)
     - Analyzes HTTP security headers
     - Checks X-Frame-Options, CSP, HSTS, etc.
   - **Stage 8c - OWASP ZAP**:
     - Uses correct image: `zaproxy/zap-stable` (NOT `owasp/zap2docker-stable`)
     - Includes application health check (HTTP 200 validation)
     - 5-minute timeout to prevent hanging
   - All three jobs run in parallel for faster execution

## Pipeline Best Practices & Code Generation Requirements

When generating pipeline code, ALWAYS follow these requirements to ensure reliability and avoid common errors:

### 1. Dependency Installation Best Practices

**ALWAYS check package-lock.json existence before choosing install method:**

```yaml
- name: Install dependencies
  run: npm install --prefer-offline --no-audit
  # Use 'npm install' if package-lock.json is not committed
  # Use 'npm ci' only if package-lock.json exists for reproducible builds
```

**Why**: `npm ci` requires `package-lock.json` and will fail with EUSAGE error if missing

---

### 2. Error Handling in Security Scans

**ALWAYS use `set +e` for security scan steps that may exit with non-zero codes:**

```yaml
- name: Run OPA/Conftest
  run: |
    # Install conftest...

    set +e  # Disable exit on error
    conftest test Dockerfile --policy ./policy
    CONFTEST_EXIT_CODE=$?
    set -e  # Re-enable exit on error

    if [ $CONFTEST_EXIT_CODE -ne 0 ]; then
      echo "⚠️  Policy violations found"
      echo "✅ Pipeline continues in warning mode"
    fi
    exit 0  # Always succeed
```

**Why**: GitHub Actions uses `bash -e` which exits immediately on errors, before error handlers (`|| {}`) can execute

**Apply to**: Conftest, npm audit, Trivy, Security Headers, OWASP ZAP, Noir

---

### 3. Application Health Checks

**ALWAYS implement retry logic for health checks (NOT just kubectl wait):**

```yaml
- name: Application Health Check
  run: |
    echo "🏥 Checking application health..."

    MAX_RETRIES=10
    RETRY_COUNT=0
    SUCCESS=false

    while [ $RETRY_COUNT -lt $MAX_RETRIES ]; do
      echo "Attempt $((RETRY_COUNT + 1))/$MAX_RETRIES..."

      HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:30080 2>&1 || echo "000")

      if [ "$HTTP_CODE" = "200" ]; then
        echo "✅ Application is responding (HTTP $HTTP_CODE)"
        SUCCESS=true
        break
      elif [ "$HTTP_CODE" = "000" ]; then
        echo "⚠️  Connection failed - application not ready yet"
      fi

      RETRY_COUNT=$((RETRY_COUNT + 1))
      if [ $RETRY_COUNT -lt $MAX_RETRIES ]; then
        sleep 10
      fi
    done

    if [ "$SUCCESS" = false ]; then
      echo "❌ Health check failed"
      kubectl get pods -l app=juice-shop
      exit 1
    fi
  continue-on-error: true
```

**Why**: Pod readiness doesn't guarantee HTTP server is listening. Exit code 7 (connection refused) is common.

---

### 4. Security Headers Scan Requirements

**ALWAYS pre-check application accessibility before scanning headers:**

```yaml
- name: Run Security Headers Analysis
  if: always()
  run: |
    set +e  # Don't exit on error

    echo "🔒 Analyzing HTTP security headers..." | tee security-headers-scan.txt

    # Check if application is accessible
    HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:30080 2>&1 || echo "000")

    if [ "$HTTP_CODE" != "200" ]; then
      echo "❌ Application not accessible (HTTP $HTTP_CODE)" | tee -a security-headers-scan.txt
      echo "Creating diagnostic report..." | tee -a security-headers-scan.txt
      kubectl get pods -l app=juice-shop 2>&1 | tee -a security-headers-scan.txt || true
      exit 0  # Exit successfully to allow artifact upload
    fi

    echo "✅ Application accessible (HTTP $HTTP_CODE)" | tee -a security-headers-scan.txt
    curl -s -D - http://localhost:30080 -o /dev/null 2>&1 | grep -E "^X-|^Content-Security|^Strict-Transport" | tee -a security-headers-scan.txt
    # ... rest of header checks
  continue-on-error: true
```

**Why**: Prevents exit code 7 errors when application isn't fully accessible

---

### 5. OWASP ZAP Scan Prerequisites

**ALWAYS check Docker availability before running ZAP:**

```yaml
- name: Run OWASP ZAP Baseline Scan
  if: always()
  run: |
    set +e  # Don't exit on error

    echo "🔍 Running OWASP ZAP baseline scan..."

    # Check 1: Docker installed?
    if ! command -v docker &> /dev/null; then
      echo "❌ Docker not installed"
      echo "<html><body><h1>ZAP Scan - Docker Not Available</h1>" > zap-report.html
      echo "<p>Docker is not installed on this runner.</p></body></html>" >> zap-report.html
      exit 0
    fi

    # Check 2: Docker daemon running?
    if ! docker ps &> /dev/null; then
      echo "❌ Docker daemon not running"
      echo "<html><body><h1>ZAP Scan - Docker Daemon Not Running</h1>" > zap-report.html
      echo "<p>Start with: sudo systemctl start docker</p></body></html>" >> zap-report.html
      exit 0
    fi

    # Check 3: Application accessible?
    HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" http://localhost:30080 2>&1 || echo "000")
    if [ "$HTTP_CODE" != "200" ]; then
      echo "❌ Application not accessible"
      echo "<html><body><h1>ZAP Scan - Application Not Accessible</h1>" > zap-report.html
      exit 0
    fi

    echo "✅ All prerequisites met"

    # Run ZAP with correct image name and timeout
    mkdir -p ${{ github.workspace }}/zap-reports
    timeout 300 docker run --rm --network host \
      -v ${{ github.workspace }}/zap-reports:/zap/wrk:rw \
      zaproxy/zap-stable:latest \
      zap-baseline.py -t http://localhost:30080 -I -r zap-report.html \
      || echo "⚠️  Security issues found (warning mode)"

    # Handle report
    if [ -f "${{ github.workspace }}/zap-reports/zap-report.html" ]; then
      mv ${{ github.workspace }}/zap-reports/zap-report.html ${{ github.workspace }}/
    else
      echo "<html><body><h1>ZAP scan did not complete</h1></body></html>" > zap-report.html
    fi
  continue-on-error: true
```

**Why**: Self-hosted runners may not have Docker. Creates diagnostic reports for troubleshooting.

**IMPORTANT**: Use `zaproxy/zap-stable:latest` NOT `owasp/zap2docker-stable` (deprecated)

---

### 6. K8s Manifest Scan Runner Selection

**ALWAYS use ubuntu-latest for K8s manifest scanning (NOT self-hosted):**

```yaml
k8s-manifest-scan:
  name: K8s Manifest Security
  runs-on: ubuntu-latest  # ✅ Has Docker pre-installed
  needs: deploy
  steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Run K8s Manifest Security Scan
      run: |
        echo "🔐 Scanning Kubernetes manifests..." | tee k8s-manifest-scan.txt

        docker run --rm -v $(pwd):/project \
          openpolicyagent/conftest:latest \
          test --policy ./policy kind-k8s-cluster/juice-shop-deployment.yaml \
          2>&1 | tee -a k8s-manifest-scan.txt \
          || echo "⚠️  Policy violations found"
      continue-on-error: true
```

**Why**: GitHub-hosted ubuntu-latest has Docker pre-installed. Self-hosted may not.

**Note**: K8s manifest scanning doesn't need running application, only Docker

---

### 7. kubectl Installation in K8s-Security Jobs

**ALWAYS check kubectl availability before use (self-hosted runners may not have it):**

```yaml
- name: Install kubectl (if not present)
  run: |
    if ! command -v kubectl &> /dev/null; then
      echo "📥 Installing kubectl..."
      curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
      chmod +x kubectl
      sudo mv kubectl /usr/local/bin/
      echo "✅ kubectl installed"
    else
      echo "✅ kubectl already installed"
    fi
```

**Why**: Self-hosted runners don't have kubectl pre-installed (unlike GitHub-hosted)

---

### 8. OPA Rego Policy Syntax (CRITICAL)

**ALWAYS use OPA Rego v1 syntax (OPA 0.50+) for all policy files:**

```rego
package main

# OPA Rego v1 syntax (OPA 0.50+)
import rego.v1

# ✅ CORRECT: Use 'contains' and 'if' keywords
deny contains msg if {
  input[i].Cmd == "from"
  val := split(input[i].Value[0], ":")
  contains(val[1], "latest")
  msg := sprintf("Line %d: do not use 'latest' tag", [i])
}

# ✅ CORRECT: Use ':=' for assignments
has_user_directive if {
  input[_].Cmd == "user"
}

# ❌ WRONG: Old syntax (will fail)
deny[msg] {
  msg = "error"
}
```

**Required Changes:**
- Add `contains` keyword: `deny[msg]` → `deny contains msg`
- Add `if` keyword: `deny contains msg {` → `deny contains msg if {`
- Use `:=` for assignments: `msg =` → `msg :=`
- Add `import rego.v1` at top

**Policy File Structure:**
```
policy/
├── dockerfile.rego    # Dockerfile security policies
└── kubernetes.rego    # K8s manifest security policies
```

---

### 9. Parallel Execution Strategy

**ALWAYS split Stage 2 (SAST, Secrets, Tests) into parallel jobs:**

```yaml
# Stage 2a: SAST (Parallel)
sast:
  needs: lint

# Stage 2b: Secret Detection (Parallel)
secret-detection:
  needs: lint

# Stage 2c: Tests (Parallel)
test:
  needs: lint

# Stage 3: Depends on all Stage 2 jobs
security-scan:
  needs: [sast, secret-detection, test]
```

**Why**: 40-60% faster execution than sequential

**ALWAYS split Stage 8 (K8s Security) into parallel jobs:**
- 8a: K8s Manifest (ubuntu-latest, no app needed)
- 8b: Security Headers (self-hosted, needs app)
- 8c: OWASP ZAP (self-hosted, needs app)

---

### 10. Dependency Order

**ALWAYS install dependencies before running npm commands:**

```yaml
# ✅ CORRECT Order:
- name: Install dependencies
  run: npm install --prefer-offline --no-audit

- name: Run npm audit
  run: npm audit --audit-level=high || echo "⚠️  Vulnerabilities found"

# ❌ WRONG: audit before install will fail
```

**Why**: npm audit requires `node_modules` or `package-lock.json`

---

### 11. OWASP Noir Scan (Stage 9)

**ALWAYS install jq for JSON processing in Noir scans:**

```yaml
- name: Install jq for JSON processing
  run: |
    sudo apt-get update -qq
    sudo apt-get install -y jq

- name: Run OWASP Noir Attack Surface Scan
  run: |
    echo "🔍 Running OWASP Noir..." | tee noir-scan.txt

    # Run Noir
    docker run --rm -v $(pwd):/target \
      ghcr.io/owasp-noir/noir:latest \
      noir -b /target -f json > noir-results.json 2>&1 || echo "⚠️  Scan completed"

    # Parse with jq (NOT grep)
    if [ -f "noir-results.json" ] && [ -s "noir-results.json" ]; then
      ENDPOINT_COUNT=$(jq '.endpoints | length' noir-results.json 2>/dev/null || echo "0")
      echo "   Total Endpoints: $ENDPOINT_COUNT" | tee -a noir-scan.txt

      # Display endpoint details
      jq -r '.endpoints[:20] | .[] | "  • [\(.method)] \(.url)"' noir-results.json 2>/dev/null | tee -a noir-scan.txt

      # Methods distribution
      jq -r '.endpoints | group_by(.method) | .[] | "  • \(.[0].method): \(length) endpoints"' noir-results.json 2>/dev/null | tee -a noir-scan.txt
    fi
  continue-on-error: true
```

**Why**: jq provides proper JSON parsing. grep causes "Broken pipe" errors and only shows partial data.

---

### 12. Artifact Upload Strategy

**ALWAYS use `if: always()` for artifact uploads:**

```yaml
- name: Upload Security Report
  uses: actions/upload-artifact@v4
  if: always()  # Upload even if step fails
  with:
    name: security-report
    path: report.txt
    retention-days: 30
```

**Why**: Ensures diagnostic reports upload even when scans fail

---

### 13. continue-on-error Usage

**ALWAYS add `continue-on-error: true` to security scan steps in training mode:**

```yaml
- name: Run Security Scan
  run: |
    # scan commands
  continue-on-error: true  # Warning mode for training
```

**Why**: Allows pipeline to complete even with vulnerabilities (educational purposes)

---

### 14. Kind Cluster Connectivity for Self-Hosted Runners

**CRITICAL**: Self-hosted runners accessing Kind clusters need special network handling.

Kind clusters run inside Docker containers. NodePort services don't work via `localhost` by default - you need to detect and use the correct access method.

#### Stage 7: Deploy - Add Connectivity Verification

**ALWAYS add multi-method connectivity testing after deployment:**

```yaml
deploy:
  runs-on: self-hosted
  steps:
    # ... existing deployment steps ...

    - name: Verify Kind cluster connectivity
      run: |
        echo "🔍 Checking Kind cluster status..."
        kubectl cluster-info
        kubectl get nodes

        echo ""
        echo "📋 Checking existing deployments..."
        kubectl get deployments -A || true
        kubectl get pods -A || true

    - name: Get service endpoint and test connectivity
      run: |
        echo "🌐 Getting Kind cluster access information..."

        # Get the Kind node IP (Docker container IP)
        KIND_NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')
        echo "Kind Node IP: $KIND_NODE_IP"

        # Test multiple access methods
        echo ""
        echo "Testing connectivity methods:"

        # Method 1: Via localhost (Kind port mapping)
        echo "1. Testing localhost:30080..."
        curl -s -o /dev/null -w "HTTP Status: %{http_code}\n" http://localhost:30080 || echo "localhost:30080 not accessible"

        # Method 2: Via Kind node IP
        echo "2. Testing ${KIND_NODE_IP}:30080..."
        curl -s -o /dev/null -w "HTTP Status: %{http_code}\n" http://${KIND_NODE_IP}:30080 || echo "${KIND_NODE_IP}:30080 not accessible"

        # Method 3: Via Kind container name
        echo "3. Testing kind-control-plane:30080..."
        curl -s -o /dev/null -w "HTTP Status: %{http_code}\n" http://kind-control-plane:30080 || echo "kind-control-plane:30080 not accessible"

        echo ""
        echo "📍 Application endpoints:"
        echo "   - NodePort: http://localhost:30080"
        echo "   - Kind Node: http://${KIND_NODE_IP}:30080"
        echo "   - Container: http://kind-control-plane:30080"
      id: endpoint
```

**Why**: Provides diagnostic information showing which connection method works

---

#### Stage 8b: Security Headers - Dynamic Access Detection

**ALWAYS detect and use the working access method dynamically:**

```yaml
security-headers-scan:
  runs-on: self-hosted
  needs: deploy
  steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Install kubectl (if not present)
      run: |
        if ! command -v kubectl &> /dev/null; then
          echo "📥 Installing kubectl..."
          curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
          chmod +x kubectl
          sudo mv kubectl /usr/local/bin/
          echo "✅ kubectl installed"
        else
          echo "✅ kubectl already installed"
        fi

    - name: Wait for application readiness
      run: |
        echo "⏳ Waiting for application to be ready..."
        sleep 30
        kubectl wait --for=condition=ready pod -l app=juice-shop --timeout=120s

    - name: Detect Kind cluster access method
      id: detect-access
      run: |
        echo "🔍 Detecting best access method for Kind cluster..."

        # Get Kind node IP
        KIND_NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')
        echo "Kind Node IP: $KIND_NODE_IP"

        # Test access methods in order of preference
        APP_URL=""

        # Method 1: localhost (if Kind has port mapping)
        if curl -s -o /dev/null -w "%{http_code}" http://localhost:30080 2>/dev/null | grep -q "200"; then
          APP_URL="http://localhost:30080"
          echo "✅ Access via localhost:30080"
        # Method 2: Kind node IP
        elif curl -s -o /dev/null -w "%{http_code}" http://${KIND_NODE_IP}:30080 2>/dev/null | grep -q "200"; then
          APP_URL="http://${KIND_NODE_IP}:30080"
          echo "✅ Access via ${KIND_NODE_IP}:30080"
        # Method 3: Docker container name
        elif curl -s -o /dev/null -w "%{http_code}" http://kind-control-plane:30080 2>/dev/null | grep -q "200"; then
          APP_URL="http://kind-control-plane:30080"
          echo "✅ Access via kind-control-plane:30080"
        else
          echo "❌ No accessible endpoint found"
          APP_URL="http://localhost:30080"  # Fallback
        fi

        echo "app_url=$APP_URL" >> $GITHUB_OUTPUT
        echo "Selected URL: $APP_URL"

    - name: Application Health Check
      env:
        APP_URL: ${{ steps.detect-access.outputs.app_url }}
      run: |
        echo "🏥 Checking application health at ${APP_URL}..."

        MAX_RETRIES=10
        RETRY_COUNT=0
        SUCCESS=false

        while [ $RETRY_COUNT -lt $MAX_RETRIES ]; do
          echo "Attempt $((RETRY_COUNT + 1))/$MAX_RETRIES..."

          HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" ${APP_URL} 2>&1 || echo "000")

          if [ "$HTTP_CODE" = "200" ]; then
            echo "✅ Application is responding (HTTP $HTTP_CODE)"
            SUCCESS=true
            break
          elif [ "$HTTP_CODE" = "000" ]; then
            echo "⚠️  Connection failed - application not ready yet"
          else
            echo "⚠️  Application returned HTTP $HTTP_CODE"
          fi

          RETRY_COUNT=$((RETRY_COUNT + 1))
          if [ $RETRY_COUNT -lt $MAX_RETRIES ]; then
            echo "Waiting 10 seconds before retry..."
            sleep 10
          fi
        done

        if [ "$SUCCESS" = false ]; then
          echo "❌ Application health check failed"
          kubectl get pods -l app=juice-shop
          kubectl describe pod -l app=juice-shop | tail -50
          exit 1
        fi
      continue-on-error: true

    - name: Run Security Headers Analysis
      if: always()
      env:
        APP_URL: ${{ steps.detect-access.outputs.app_url }}
      run: |
        set +e  # Don't exit on error

        echo "🔒 Analyzing HTTP security headers..." | tee security-headers-scan.txt
        echo "Target: ${APP_URL}" | tee -a security-headers-scan.txt
        echo "" | tee -a security-headers-scan.txt

        # Check if application is accessible
        HTTP_CODE=$(curl -s -o /dev/null -w "%{http_code}" ${APP_URL} 2>&1 || echo "000")

        if [ "$HTTP_CODE" != "200" ]; then
          echo "❌ Application not accessible (HTTP $HTTP_CODE)" | tee -a security-headers-scan.txt
          # ... diagnostic output ...
          exit 0
        fi

        echo "✅ Application accessible (HTTP $HTTP_CODE)" | tee -a security-headers-scan.txt

        # Use ${APP_URL} instead of hardcoded localhost:30080
        curl -s -D - ${APP_URL} -o /dev/null 2>&1 | grep -E "^X-|^Content-Security|^Strict-Transport" | tee -a security-headers-scan.txt

        HEADERS=$(curl -s -D - ${APP_URL} -o /dev/null 2>&1)
        # ... rest of header checks using HEADERS variable ...
      continue-on-error: true
```

**Why**:
- Automatically detects working connection method
- Kind node IP often works when localhost doesn't
- Passes URL between steps via GitHub Actions outputs
- Handles different Kind networking configurations

---

#### Stage 8c: OWASP ZAP - Docker Network Configuration

**CRITICAL**: ZAP runs in Docker and needs special network handling for Kind access.

```yaml
owasp-zap-scan:
  runs-on: self-hosted
  needs: deploy
  steps:
    - name: Checkout code
      uses: actions/checkout@v4

    - name: Install kubectl (if not present)
      run: |
        if ! command -v kubectl &> /dev/null; then
          echo "📥 Installing kubectl..."
          curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"
          chmod +x kubectl
          sudo mv kubectl /usr/local/bin/
          echo "✅ kubectl installed"
        else
          echo "✅ kubectl already installed"
        fi

    - name: Wait for application readiness
      run: |
        echo "⏳ Waiting for application to be ready..."
        sleep 30
        kubectl wait --for=condition=ready pod -l app=juice-shop --timeout=120s

    - name: Detect Kind cluster access method for ZAP
      id: detect-zap-access
      run: |
        echo "🔍 Detecting best access method for ZAP scan..."

        # Get Kind node IP
        KIND_NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')
        echo "Kind Node IP: $KIND_NODE_IP"

        # Get Kind container name
        KIND_CONTAINER=$(docker ps --filter "name=kind-control-plane" --format "{{.Names}}" | head -1)
        echo "Kind Container: $KIND_CONTAINER"

        # For ZAP running in Docker, we need an address accessible from Docker network
        ZAP_TARGET_URL=""
        ACCESS_METHOD=""

        # Method 1: Kind container IP (most reliable for Docker-to-Docker)
        if [ -n "$KIND_NODE_IP" ]; then
          if curl -s -o /dev/null -w "%{http_code}" http://${KIND_NODE_IP}:30080 2>/dev/null | grep -q "200"; then
            ZAP_TARGET_URL="http://${KIND_NODE_IP}:30080"
            ACCESS_METHOD="kind-node-ip"
            echo "✅ Will use Kind Node IP for ZAP: ${KIND_NODE_IP}:30080"
          fi
        fi

        # Method 2: host.docker.internal (works on Docker Desktop)
        if [ -z "$ZAP_TARGET_URL" ]; then
          if docker run --rm curlimages/curl:latest curl -s -o /dev/null -w "%{http_code}" http://host.docker.internal:30080 2>/dev/null | grep -q "200"; then
            ZAP_TARGET_URL="http://host.docker.internal:30080"
            ACCESS_METHOD="host-docker-internal"
            echo "✅ Will use host.docker.internal for ZAP"
          fi
        fi

        # Method 3: Kind container name via Docker network
        if [ -z "$ZAP_TARGET_URL" ] && [ -n "$KIND_CONTAINER" ]; then
          ZAP_TARGET_URL="http://${KIND_CONTAINER}:30080"
          ACCESS_METHOD="kind-container-name"
          echo "✅ Will use Kind container name for ZAP: ${KIND_CONTAINER}:30080"
        fi

        # Fallback
        if [ -z "$ZAP_TARGET_URL" ]; then
          ZAP_TARGET_URL="http://${KIND_NODE_IP}:30080"
          ACCESS_METHOD="fallback-kind-ip"
          echo "⚠️  Using fallback: ${KIND_NODE_IP}:30080"
        fi

        echo "zap_target_url=$ZAP_TARGET_URL" >> $GITHUB_OUTPUT
        echo "access_method=$ACCESS_METHOD" >> $GITHUB_OUTPUT
        echo "kind_network=$(docker network ls --filter name=kind --format '{{.Name}}' | head -1)" >> $GITHUB_OUTPUT
        echo "Selected ZAP target: $ZAP_TARGET_URL (Method: $ACCESS_METHOD)"

    - name: Application Health Check
      run: |
        echo "🏥 Checking application health from runner host..."

        MAX_RETRIES=10
        RETRY_COUNT=0
        SUCCESS=false

        KIND_NODE_IP=$(kubectl get nodes -o jsonpath='{.items[0].status.addresses[?(@.type=="InternalIP")].address}')

        while [ $RETRY_COUNT -lt $MAX_RETRIES ]; do
          echo "Attempt $((RETRY_COUNT + 1))/$MAX_RETRIES..."

          # Try multiple endpoints
          if curl -s -o /dev/null -w "%{http_code}" http://localhost:30080 2>/dev/null | grep -q "200"; then
            echo "✅ Application accessible via localhost:30080"
            SUCCESS=true
            break
          elif curl -s -o /dev/null -w "%{http_code}" http://${KIND_NODE_IP}:30080 2>/dev/null | grep -q "200"; then
            echo "✅ Application accessible via ${KIND_NODE_IP}:30080"
            SUCCESS=true
            break
          else
            echo "⚠️  Application not ready yet"
          fi

          RETRY_COUNT=$((RETRY_COUNT + 1))
          if [ $RETRY_COUNT -lt $MAX_RETRIES ]; then
            echo "Waiting 10 seconds before retry..."
            sleep 10
          fi
        done

        if [ "$SUCCESS" = false ]; then
          echo "❌ Application health check failed"
          kubectl get pods -l app=juice-shop
          kubectl describe pod -l app=juice-shop | tail -50
          exit 1
        fi
      continue-on-error: true

    - name: Run OWASP ZAP Baseline Scan
      if: always()
      env:
        ZAP_TARGET_URL: ${{ steps.detect-zap-access.outputs.zap_target_url }}
        ACCESS_METHOD: ${{ steps.detect-zap-access.outputs.access_method }}
        KIND_NETWORK: ${{ steps.detect-zap-access.outputs.kind_network }}
      run: |
        set +e  # Don't exit on error

        echo "🔍 Running OWASP ZAP baseline scan..."
        echo "Target: ${ZAP_TARGET_URL}"
        echo "Access Method: ${ACCESS_METHOD}"
        echo "Note: This may take 2-5 minutes..."

        # Check if Docker is available
        if ! command -v docker &> /dev/null; then
          echo "❌ Docker not installed"
          echo "<html><body><h1>ZAP Scan - Docker Not Available</h1></body></html>" > zap-report.html
          exit 0
        fi

        # Check if Docker daemon is running
        if ! docker ps &> /dev/null; then
          echo "❌ Docker daemon not running"
          echo "<html><body><h1>ZAP Scan - Docker Daemon Not Running</h1></body></html>" > zap-report.html
          exit 0
        fi

        echo "✅ Docker available and daemon running"

        # Create directory for ZAP reports
        mkdir -p ${{ github.workspace }}/zap-reports

        # Determine network mode based on access method
        if [ "$ACCESS_METHOD" = "kind-node-ip" ] || [ "$ACCESS_METHOD" = "fallback-kind-ip" ]; then
          # Use host network mode for Kind node IP access
          NETWORK_MODE="host"
          echo "Using Docker network mode: host"
        elif [ -n "$KIND_NETWORK" ]; then
          # Use Kind network for container-to-container communication
          NETWORK_MODE="$KIND_NETWORK"
          echo "Using Docker network mode: $KIND_NETWORK"
        else
          # Fallback to host network
          NETWORK_MODE="host"
          echo "Using Docker network mode: host (fallback)"
        fi

        # Run ZAP scan with appropriate network configuration
        echo "Starting ZAP scan targeting: ${ZAP_TARGET_URL}"
        timeout 300 docker run --rm --network ${NETWORK_MODE} \
          -v ${{ github.workspace }}/zap-reports:/zap/wrk:rw \
          zaproxy/zap-stable:latest \
          zap-baseline.py -t ${ZAP_TARGET_URL} -I -r zap-report.html \
          || ZAP_EXIT_CODE=$?

        if [ -n "$ZAP_EXIT_CODE" ]; then
          echo "⚠️  ZAP scan exited with code: $ZAP_EXIT_CODE (warning mode)"
        fi

        # Move report to workspace root
        if [ -f "${{ github.workspace }}/zap-reports/zap-report.html" ]; then
          mv ${{ github.workspace }}/zap-reports/zap-report.html ${{ github.workspace }}/
          echo "✅ ZAP report saved"
        else
          echo "⚠️  ZAP report not generated, creating diagnostic report"
          echo "<html><body><h1>ZAP Scan - Report Not Generated</h1>" > zap-report.html
          echo "<p>Target: ${ZAP_TARGET_URL}</p>" >> zap-report.html
          echo "<p>Access Method: ${ACCESS_METHOD}</p>" >> zap-report.html
          echo "<p>Network Mode: ${NETWORK_MODE}</p>" >> zap-report.html
          echo "</body></html>" >> zap-report.html
        fi

        echo "📊 ZAP scan process completed"
      continue-on-error: true

    - name: Upload OWASP ZAP Scan Report
      uses: actions/upload-artifact@v4
      if: always()
      with:
        name: k8s-security-owasp-zap
        path: zap-report.html
        retention-days: 30
```

**Why**:
- ZAP container needs Docker network access to Kind cluster
- Kind node IP is accessible via host network mode
- Automatically selects correct network mode based on access method
- Provides comprehensive diagnostics if scan fails
- Supports multiple Docker networking configurations

**Key Concepts**:
- **host network**: ZAP container shares host network, can access localhost and Kind node IP
- **Kind network**: Direct container-to-container communication
- **Kind node IP**: Docker container's internal IP (most reliable method)

---

### 15. Summary: Kind Cluster Connectivity Pattern

**For all self-hosted stages accessing Kind cluster:**

1. **Detect access method** - Test localhost, Kind node IP, container name
2. **Pass URL via outputs** - Use GitHub Actions step outputs
3. **Use environment variables** - Reference detected URL in subsequent steps
4. **Handle Docker networking** - Select appropriate network mode for containerized tools
5. **Provide diagnostics** - Create diagnostic reports when connectivity fails

**Applies to**:
- Stage 7: Deploy (verification)
- Stage 8b: Security Headers (curl from host)
- Stage 8c: OWASP ZAP (Docker-to-Docker)

---

## Error Handling

If any step fails:
- Ask the user for clarification or additional information
- Provide clear error messages
- Suggest alternatives or workarounds
- Continue with other steps if possible

## Completion Criteria

The slash command is complete when:
- ✅ Project summary analyzed
- ✅ Existing pipeline found and read
- ✅ Existing pipeline archived to workflows-archive directory
- ✅ Enhanced pipeline created
- ✅ Policy files created (if needed)
- ✅ YAML validation passed
- ✅ Session summary document created
- ✅ All TODOs marked as completed
- ✅ User presented with next steps
