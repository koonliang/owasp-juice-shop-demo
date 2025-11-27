# Deploy Application to Kubernetes Cluster

You are helping a user deploy their application to a Kubernetes cluster and add an automated deployment stage to their existing GitHub Actions pipeline.

## Your Role - INTELLIGENT AUTOMATION

**IMPORTANT:** This command should be SMART and minimize questions by auto-detecting information from existing setup.

## Intelligence First Approach

### Step 1: Auto-Discovery (DO THIS FIRST!)

Before asking ANY questions, automatically detect:

1. **GitHub Actions Workflow Analysis**
   - Search for `.github/workflows/*.yml` files
   - Parse workflow to extract:
     - Docker image name and repository
     - Docker Hub username/organization
     - Image tags being used
     - Application name (from repo/image name)
     - Whether self-hosted runner is configured

2. **Kubernetes Cluster Detection**
   - Check if kubectl is configured: `kubectl cluster-info`
   - Detect cluster type: `kind get clusters`, `minikube status`, or cloud context
   - Get cluster name and current context
   - Verify cluster accessibility
   - **CRITICAL: For Kind clusters, check kind-cluster-config.yaml for extraPortMappings**
   - Parse Kind config to find which NodePorts are actually mapped to localhost
   - Example: If config shows `containerPort: 30080`, only suggest 30080 (not 30000!)

3. **Application Configuration Detection**
   - Read Dockerfile to find `EXPOSE` port
   - Check for existing K8s manifests in `k8s/`, `kubernetes/`, or `.k8s/` directories
   - Look for docker-compose.yml for port mappings
   - Check package.json or server files for port configuration

4. **Self-Hosted Runner Detection**
   - Check for `.github/workflows/` runs-on values
   - Look for self-hosted runner configuration
   - **IMPORTANT:** Self-hosted runners need kubectl installed
   - Pipeline MUST include kubectl installation step (auto-install if missing)
   - Verify runner has kubeconfig access to target cluster

### Step 2: Build Configuration Object

After auto-discovery, create a configuration object with:
- ✅ Auto-detected values (show these to user)
- ❓ Missing values that need user input (ask ONLY these)

### Step 3: Interactive Questions (ONLY for Missing Info)

Ask user ONLY for information that couldn't be auto-detected:
1. **Number of replicas** (default: 1 for dev/training, 2+ for production HA)
2. **Service type** (NodePort for local, LoadBalancer for cloud)
3. **NodePort** (if NodePort selected):
   - **For Kind clusters**: ONLY suggest ports from kind-cluster-config.yaml extraPortMappings
   - Example: If config has 30080 and 30443, suggest 30080 (first available)
   - **For other clusters**: Suggest 30000 (standard NodePort range)
4. **Resource limits** (suggest based on app type)
5. **Health check endpoint** (default: `/` or auto-detect from code)

### Validation Rules

- App name: lowercase alphanumeric with hyphens
- Port: 1-65535
- Replicas: 1-10 recommended
- NodePort:
  - **Kind clusters**: MUST match extraPortMappings in kind-cluster-config.yaml
  - **Other clusters**: 30000-32767 (standard range)
- Resources: Use standard K8s notation (100m, 256Mi, 1Gi, etc.)

### CRITICAL: Kind Port Mapping Issue

**Problem:** Kind clusters only expose specific ports to localhost via extraPortMappings.
If you suggest NodePort 30000 but Kind config only maps 30080, the service won't be accessible!

**Solution:**
1. Always check for `kind-cluster-config.yaml` or `kind-config.yaml`
2. Parse `extraPortMappings` section to find mapped containerPorts
3. ONLY suggest NodePort values that exist in the mapping
4. If no Kind config found, check Docker container ports:
   ```bash
   docker ps --filter name=<cluster>-control-plane --format "{{.Ports}}"
   ```

**Example Kind Config:**
```yaml
extraPortMappings:
  - containerPort: 30080  # ← This port is mapped
    hostPort: 30080
  - containerPort: 30443  # ← This port is mapped
    hostPort: 30443
```
**Result:** Only suggest NodePort 30080 or 30443 (not 30000!)

## Deployment Process

### Step 1: Show Auto-Detected Configuration
Display what was automatically discovered:
```
🔍 Auto-detected configuration:
✅ App name: juice-shop
✅ Docker image: rupeedev/owasp-juice-shop:latest
✅ Container port: 3000
✅ K8s cluster: devsecops-cluster (Kind)
✅ Cluster status: Running
✅ GitHub workflow: .github/workflows/simple-pipeline.yml
✅ Pipeline stages: lint → test → build → push-to-dockerhub
```

### Step 2: Verify and Fill Gaps
- Confirm auto-detected values with user
- Ask ONLY for missing configuration (replicas, service type, resources)
- Provide smart defaults based on detected environment

### Step 2: Create Kubernetes Manifests

Generate in `k8s/` directory:

**Deployment manifest** (`<app-name>-deployment.yaml`):
- Replicas as specified
- Image pull policy: Always
- Resource requests and limits
- Liveness and readiness probes (HTTP GET on health path)
- Environment variables (if provided)
- Container port exposed
- Labels for service selection

**Service manifest** (`<app-name>-service.yaml`):
- Type as specified (NodePort/LoadBalancer/ClusterIP)
- Port mappings
- Selector matching deployment labels
- NodePort if specified

### Step 3: Create Kubernetes Manifests

Generate in `k8s/` or `kind-k8s-cluster/` directory (check for existing K8s directories first):

**Deployment manifest** (`<app-name>-deployment.yaml`):
- Use auto-detected image from GitHub workflow
- Use auto-detected port from Dockerfile
- Set replicas as user specified or default
- Add resource limits as specified
- Include liveness/readiness probes on detected health endpoint
- Use proper labels for service selection

**Service manifest** (`<app-name>-service.yaml`):
- Use detected service type (NodePort for Kind/local, LoadBalancer for cloud)
- Map to detected container port
- Set NodePort if specified and service type is NodePort

### Step 4: Deploy to Kubernetes
- Apply deployment manifest: `kubectl apply -f k8s/<app>-deployment.yaml`
- Apply service manifest: `kubectl apply -f k8s/<app>-service.yaml`
- Wait for rollout: `kubectl rollout status deployment/<app>`
- Verify pods: `kubectl get pods -l app=<app>`
- Check service: `kubectl get svc <app>`
- Show access information based on service type

### Step 5: Extend GitHub Actions Workflow

**IMPORTANT:** Automatically extend the detected workflow file:
- Identify the last job in pipeline (e.g., `push-to-dockerhub`)
- Add new `deploy` job that:
  - Depends on the last job (needs: [last-job])
  - Runs on self-hosted runner (if detected) or ubuntu-latest
  - Uses the same Docker image variables from the workflow
  - Includes steps to:
    - **Install kubectl if not present (CRITICAL for self-hosted runners!)**
    - Verify kubectl access to cluster
    - Apply K8s manifests
    - Wait for rollout completion
    - Verify deployment
    - Show access URL

Example deploy job structure:
```yaml
deploy:
  name: Deploy to Kubernetes
  runs-on: self-hosted  # or ubuntu-latest
  needs: push-to-dockerhub  # depends on previous job
  env:
    DOCKER_USERNAME: <detected>
    DOCKER_REPO: <detected>
  steps:
    - Checkout code
    - Install kubectl (if not present on runner)
    - Verify kubectl connectivity
    - Apply K8s manifests
    - Update deployment image
    - Wait for rollout
    - Verify and show access info
```

**CRITICAL: kubectl Installation for Self-Hosted Runners**

Self-hosted runners may not have kubectl pre-installed. Always add this step:

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
      echo "✅ kubectl already installed: $(kubectl version --client --short 2>/dev/null || kubectl version --client)"
    fi
```

This prevents "kubectl: command not found" errors (exit code 127).

### Step 6: Create Documentation
Generate `k8s/README.md` or update existing with:
- Auto-detected configuration summary
- Access instructions (with actual URLs/commands)
- Pipeline flow diagram
- Management commands (scale, update, logs)
- Troubleshooting guide
- Cleanup instructions

### Step 7: Test Deployment
- Verify app is accessible
- Show actual access command/URL
- Check pod logs for startup
- Verify health endpoints

## Success Criteria

User should have:
- ✅ Custom K8s manifests for their app
- ✅ Application deployed and running
- ✅ Service accessible (based on type)
- ✅ GitHub Actions workflow updated with kubectl auto-install
- ✅ Pipeline successfully deploys on push
- ✅ Complete documentation
- ✅ Understanding of how to manage deployment

## Common Issues and Solutions

### Issue 1: kubectl command not found (Exit Code 127)
**Problem:** Self-hosted runner doesn't have kubectl installed
**Solution:** Pipeline should include kubectl installation step (auto-detected and added)
**Prevention:** Always check for kubectl and install if missing

### Issue 2: NodePort not accessible on Kind cluster
**Problem:** NodePort doesn't match Kind's extraPortMappings
**Solution:** Parse kind-cluster-config.yaml and only use mapped ports
**Prevention:** Auto-detect Kind port mappings before suggesting NodePort

### Issue 3: Image pull failures
**Problem:** Deployment tries to pull image before Docker Hub push completes
**Solution:** Use job dependencies (needs: [push-to-dockerhub])
**Prevention:** Ensure proper job ordering in pipeline

## Your Instructions - AUTOMATION FIRST

1. **AUTO-DETECT FIRST** (DO NOT SKIP THIS!)
   - Run discovery commands silently
   - Parse GitHub workflow files
   - Read Dockerfile and configs
   - Detect K8s cluster
   - **CRITICAL: If Kind cluster, find and parse kind-cluster-config.yaml for port mappings**
   - Build complete configuration object

2. **SHOW DISCOVERY RESULTS**
   - Display what was auto-detected in a clear summary
   - Mark items with ✅ (detected) or ❓ (needs input)
   - Build user confidence in automation

3. **ASK MINIMAL QUESTIONS**
   - ONLY ask for values that couldn't be detected
   - Provide smart defaults based on environment
   - Group related questions together

4. **CONFIRM AND PROCEED**
   - Show complete configuration (detected + user input)
   - Get final confirmation before deployment
   - Proceed with deployment

5. **EXECUTE AUTOMATICALLY**
   - Create K8s manifests
   - Deploy to cluster
   - Extend GitHub workflow (include kubectl installation step!)
   - Test deployment
   - Generate documentation

6. **VALIDATE AND VERIFY**
   - Check deployment status
   - Verify pod health
   - Test service accessibility
   - Show access information

7. **PROVIDE SUMMARY**
   - What was deployed
   - How to access it
   - Pipeline flow (before → after)
   - Next steps

## Start Message

Begin with:

"I'll help you deploy your application to Kubernetes by intelligently analyzing your existing setup and automating the deployment configuration.

**Intelligent Automation:**
- 🔍 Auto-detect configuration from GitHub Actions workflow
- 🔍 Extract Docker image and port from Dockerfile
- 🔍 Detect local Kubernetes cluster
- ❓ Ask only what's needed (minimal questions)

Let me start by analyzing your current setup..."

Then immediately run auto-discovery before asking any questions.

## Auto-Detection Examples

### Example 1: Complete Auto-Detection
```
🔍 Analyzing your setup...

✅ GitHub Actions workflow found: .github/workflows/simple-pipeline.yml
✅ Docker image: rupeedev/owasp-juice-shop:latest
✅ Container port: 3000 (from Dockerfile)
✅ App name: juice-shop
✅ K8s cluster: devsecops-cluster (Kind, local)
✅ Kind config: kind-k8s-cluster/kind-cluster-config.yaml
✅ Kind mapped ports: 30080, 30443 (IMPORTANT: These are the ONLY accessible ports!)
✅ Current pipeline: lint → test → build → push-to-dockerhub

❓ Need your input for:
  - Number of replicas (default: 1 for dev/training, 2+ for production)
  - Service type (default: NodePort for local cluster)
  - NodePort (suggest: 30080 from Kind config - IMPORTANT: Not 30000!)
  - Resource limits (default: 512Mi RAM, 0.5 CPU for Node.js)
```

### Example 2: GitHub Workflow Parsing
From `.github/workflows/pipeline.yml`, extract:
- `DOCKER_USERNAME` env var → Docker Hub username
- `DOCKER_REPO` env var → Docker repository name
- Image tags being pushed → Use for deployment
- `runs-on` value → Detect self-hosted runner
- Job dependencies → Identify last job for deployment dependency

### Example 3: Kind Port Mapping Detection (CRITICAL!)

**Check for Kind config file:**
```bash
# Look for Kind cluster config
find . -name "kind-cluster-config.yaml" -o -name "kind-config.yaml"
```

**Parse extraPortMappings:**
```yaml
# Example: kind-k8s-cluster/kind-cluster-config.yaml
nodes:
  - role: control-plane
    extraPortMappings:
      - containerPort: 30080  # ← Parse this!
        hostPort: 30080
      - containerPort: 30443  # ← Parse this!
        hostPort: 30443
```

**Result:** Available NodePorts = [30080, 30443]
**Suggest:** 30080 (first available)

**Fallback (if no config file):**
```bash
# Check Docker container ports
docker ps --filter name=devsecops-cluster-control-plane --format "{{.Ports}}"
# Output: 0.0.0.0:30080->30080/tcp, 0.0.0.0:30443->30443/tcp
# Extract: 30080, 30443
```

### Example 4: Kubernetes Manifests Directory Detection
Check in order:
1. `kind-k8s-cluster/` (if exists, use this)
2. `k8s/` (common location)
3. `kubernetes/` (alternative)
4. `.k8s/` (hidden directory)
5. Create `k8s/` if none exist

## Implementation Notes

**Tools to Use:**
- `Glob` - Find workflow files, Dockerfiles, and Kind config files
- `Read` - Parse workflow YAML, Dockerfile, and Kind config
- `Bash` - Run kubectl commands for cluster detection
- `Grep` - Search for port configurations if needed

**Kind Config Detection:**
```bash
# Find Kind config
find . -name "kind-cluster-config.yaml" -o -name "kind-config.yaml" -o -name "kind.yaml"

# Or use Glob
Glob pattern: **/kind*config*.yaml
```

Read the config file and parse extraPortMappings section.

**Workflow Parsing:**
Parse YAML to extract:
```yaml
env:
  DOCKER_USERNAME: rupeedev          # Extract this
  DOCKER_REPO: owasp-juice-shop      # Extract this

jobs:
  push-to-dockerhub:                 # Last job name
    runs-on: ubuntu-latest           # Runner type
```

**Dockerfile Parsing:**
Look for:
- `EXPOSE <port>` → Container port
- `FROM <image>` → Base image (context)
- `CMD` or `ENTRYPOINT` → Startup command

**Smart Defaults Based on Environment:**
- Local cluster (Kind/minikube) → NodePort service
- Cloud cluster (EKS/GKE/AKS) → LoadBalancer service
- Node.js app → 512Mi RAM, 0.5 CPU
- Java app → 1Gi RAM, 1 CPU
- Python app → 256Mi RAM, 0.25 CPU
