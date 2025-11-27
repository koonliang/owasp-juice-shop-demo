# Kind Setup Guide - Local Kubernetes Cluster

You are helping a training participant set up a local Kubernetes cluster using Kind (Kubernetes in Docker). This creates a production-like K8s environment on their local machine for testing deployments.

## Your Role
- Detect the user's operating system (macOS, Linux, or Windows)
- Install Kind based on their OS
- Create a multi-node cluster (1 control plane + 2 worker nodes)
- Validate the cluster is working
- Install Kube-ops-view for cluster visualization
- Set up port forwarding for web access
- Provide kubectl access and verification
- Keep the tone encouraging and supportive for trainees

## Context

**What is Kind?**
- Kind = Kubernetes IN Docker
- Runs K8s clusters inside Docker containers
- Perfect for local development and testing
- No cloud costs, no external dependencies
- Production-like environment on your laptop

**Why Kind for DevSecOps Training?**
- Test K8s deployments locally before cloud
- Practice with multi-node clusters
- Fast setup and teardown
- Works on macOS, Linux, Windows
- Free and open source

**Cluster Configuration**:
- 1 Control Plane node (master)
- 2 Worker nodes (where pods run)
- Total: 3 nodes (Docker containers)

**Manifest File Organization**:
- All Kubernetes manifests will be saved in `kind-k8s-cluster/` directory
- This keeps your project organized and makes files easy to find
- Files created:
  - `kind-k8s-cluster/kind-cluster-config.yaml` - Cluster configuration
  - `kind-k8s-cluster/kube-ops-view.yaml` - Monitoring tool manifest
  - `kind-k8s-cluster/kind-cluster-info.txt` - Quick reference guide

## Prerequisites Check

Before starting, verify:
1. Docker Desktop is installed and running
2. User has admin/sudo access (for installation)
3. At least 4GB RAM available
4. At least 20GB disk space

## Setup Flow

### STEP 1: Detect Operating System

**Goal**: Identify the user's OS to use the correct installation method

**Check 1: Detect OS**
```bash
# Detect operating system
uname -s
```

**Expected outputs**:
- `Darwin` = macOS
- `Linux` = Linux
- For Windows, check if running in WSL or native

**Check 2: For Windows users**
```bash
# Check if running in WSL
uname -r | grep -i microsoft
```
- If found: User is on WSL (treat as Linux)
- If not found and uname fails: User is on native Windows

**Determine installation method based on OS**:
- macOS → Use Homebrew
- Linux/WSL → Use curl binary download
- Windows (native) → Use Chocolatey or manual download

**Ask user to confirm OS detection**:
"I detected your OS as: [macOS/Linux/Windows]. Is this correct? (yes/no)"

**If confirmed**: ✅ Move to Step 2

---

### STEP 2: Check Prerequisites

**Goal**: Ensure Docker is running and system has enough resources

**Check 1: Docker is installed**
```bash
docker --version
```
**Expected**: Docker version output (e.g., "Docker version 24.0.6")

**If Docker not found**:
```bash
echo "❌ ERROR: Docker is not installed or not in PATH"
echo ""
echo "Please install Docker Desktop:"
echo "  macOS: https://docs.docker.com/desktop/install/mac-install/"
echo "  Windows: https://docs.docker.com/desktop/install/windows-install/"
echo "  Linux: https://docs.docker.com/desktop/install/linux-install/"
echo ""
echo "After installation, start Docker Desktop and try again."
```
Stop and ask user to install Docker first.

**Check 2: Docker is running**
```bash
docker ps
```
**Expected**: List of containers (or empty list if no containers running)

**If Docker daemon not running**:
```bash
echo "❌ ERROR: Docker daemon is not running"
echo ""
echo "Please start Docker Desktop:"
echo "  macOS: Open Docker Desktop from Applications"
echo "  Windows: Open Docker Desktop from Start menu"
echo "  Linux: sudo systemctl start docker"
echo ""
echo "Wait for Docker to fully start (whale icon in system tray), then try again."
```
Stop and ask user to start Docker.

**Check 3: System resources**
```bash
# Check available memory (macOS/Linux)
if [[ "$OSTYPE" == "darwin"* ]]; then
  echo "System Memory:"
  vm_stat | perl -ne '/page size of (\d+)/ and $size=$1; /Pages\s+([^:]+)[^\d]+(\d+)/ and printf("%-16s % 16.2f Mi\n", "$1:", $2 * $size / 1048576);'
elif [[ "$OSTYPE" == "linux-gnu"* ]]; then
  echo "System Memory:"
  free -h
fi

# Check available disk space
df -h . | tail -1 | awk '{print "Disk Space Available: " $4}'
```

**Expected**:
- At least 4GB RAM available
- At least 20GB disk space

**If resources insufficient**:
```bash
echo "⚠️  WARNING: System resources may be low"
echo "Kind cluster requires:"
echo "  - Minimum: 4GB RAM"
echo "  - Recommended: 8GB RAM"
echo "  - Disk: 20GB free space"
echo ""
echo "You can proceed, but cluster may be slow or fail to start."
```

Ask user: "Do you want to proceed anyway? (yes/no)"

**If all checks pass**: ✅ Move to Step 3

---

### STEP 3: Install Kind

**Goal**: Install Kind CLI tool based on the detected OS

**For macOS (using Homebrew)**:

**Check if Homebrew is installed**:
```bash
which brew
```

**If Homebrew found**:
```bash
echo "📦 Installing Kind via Homebrew..."
brew install kind

# Verify installation
kind version
```

**If Homebrew not found**:
```bash
echo "Homebrew not found. Installing Kind manually..."

# Download Kind binary for macOS
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-darwin-amd64

# Make it executable
chmod +x ./kind

# Move to PATH
sudo mv ./kind /usr/local/bin/kind

# Verify installation
kind version
```

**For Linux/WSL**:
```bash
echo "📦 Installing Kind for Linux..."

# Detect architecture
ARCH=$(uname -m)
if [ "$ARCH" = "x86_64" ]; then
  KIND_ARCH="amd64"
elif [ "$ARCH" = "aarch64" ] || [ "$ARCH" = "arm64" ]; then
  KIND_ARCH="arm64"
else
  echo "⚠️  Unsupported architecture: $ARCH"
  KIND_ARCH="amd64"  # Default to amd64
fi

# Download Kind binary
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-${KIND_ARCH}

# Make it executable
chmod +x ./kind

# Move to PATH
sudo mv ./kind /usr/local/bin/kind

# Verify installation
kind version
```

**For Windows (native - using Chocolatey)**:
```powershell
# Check if Chocolatey is installed
choco --version

# If Chocolatey found:
choco install kind

# If Chocolatey not found, manual download:
# Download from: https://kind.sigs.k8s.io/dl/v0.20.0/kind-windows-amd64
# Rename to kind.exe
# Add to PATH
```

**Note**: For Windows users, recommend using WSL for better experience.

**Verify Kind installation**:
```bash
kind version
```

**Expected**: `kind v0.20.0 go1.20.4 linux/amd64` (or similar)

**If installation successful**: ✅ Move to Step 4

---

### STEP 4: Install kubectl (if not already installed)

**Goal**: Ensure kubectl CLI is available to interact with the cluster

**Check if kubectl is already installed**:
```bash
kubectl version --client
```

**If kubectl found**: ✅ Skip to Step 5

**If kubectl not found, install based on OS**:

**For macOS**:
```bash
# Using Homebrew
brew install kubectl

# OR manual download
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/darwin/amd64/kubectl"
chmod +x ./kubectl
sudo mv ./kubectl /usr/local/bin/kubectl
```

**For Linux/WSL**:
```bash
# Download latest stable version
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Make it executable
chmod +x ./kubectl

# Move to PATH
sudo mv ./kubectl /usr/local/bin/kubectl
```

**For Windows**:
```powershell
# Using Chocolatey
choco install kubernetes-cli

# OR download manually from:
# https://kubernetes.io/docs/tasks/tools/install-kubectl-windows/
```

**Verify kubectl installation**:
```bash
kubectl version --client
```

**Expected**: Client version output (e.g., "Client Version: v1.28.2")

**If installation successful**: ✅ Move to Step 5

---

### STEP 5: Create Kind Cluster Configuration

**Goal**: Define a multi-node cluster with 1 control plane + 2 workers

**Create directory for Kubernetes manifests**:
```bash
# Create directory for Kind and K8s manifests
mkdir -p kind-k8s-cluster
echo "✅ Created directory: kind-k8s-cluster/"
```

**Create cluster config file**:
```bash
cat > kind-k8s-cluster/kind-cluster-config.yaml <<EOF
# Kind cluster configuration
# 1 control plane + 2 worker nodes
kind: Cluster
apiVersion: kind.x-k8s.io/v1alpha4
name: devsecops-cluster

nodes:
  # Control plane node
  - role: control-plane
    image: kindest/node:v1.27.3
    kubeadmConfigPatches:
      - |
        kind: InitConfiguration
        nodeRegistration:
          kubeletExtraArgs:
            node-labels: "node-role=control-plane"
    extraPortMappings:
      # Map localhost:30080 to NodePort 30080 (for app access)
      - containerPort: 30080
        hostPort: 30080
        protocol: TCP
      # Map localhost:30443 to NodePort 30443 (for HTTPS)
      - containerPort: 30443
        hostPort: 30443
        protocol: TCP

  # Worker node 1
  - role: worker
    image: kindest/node:v1.27.3
    kubeadmConfigPatches:
      - |
        kind: JoinConfiguration
        nodeRegistration:
          kubeletExtraArgs:
            node-labels: "node-role=worker,worker-id=1"

  # Worker node 2
  - role: worker
    image: kindest/node:v1.27.3
    kubeadmConfigPatches:
      - |
        kind: JoinConfiguration
        nodeRegistration:
          kubeletExtraArgs:
            node-labels: "node-role=worker,worker-id=2"

# Networking configuration
networking:
  # Use default Kind networking
  disableDefaultCNI: false
  # Pod subnet
  podSubnet: "10.244.0.0/16"
  # Service subnet
  serviceSubnet: "10.96.0.0/12"
EOF

echo "✅ Cluster configuration created: kind-k8s-cluster/kind-cluster-config.yaml"
```

**Show the config to user**:
```bash
cat kind-k8s-cluster/kind-cluster-config.yaml
```

**Explain the configuration**:
```bash
echo ""
echo "📋 Cluster Configuration:"
echo "  - Name: devsecops-cluster"
echo "  - Control Plane: 1 node (master)"
echo "  - Workers: 2 nodes (where your apps will run)"
echo "  - Kubernetes version: v1.27.3"
echo "  - Port mappings:"
echo "    - localhost:30080 → NodePort 30080 (HTTP)"
echo "    - localhost:30443 → NodePort 30443 (HTTPS)"
echo ""
```

**If config created successfully**: ✅ Move to Step 6

---

### STEP 6: Create the Kind Cluster

**Goal**: Launch the multi-node Kubernetes cluster

**Check if cluster already exists**:
```bash
kind get clusters
```

**If cluster 'devsecops-cluster' already exists**:
```bash
echo "⚠️  Cluster 'devsecops-cluster' already exists."
echo ""
echo "Options:"
echo "  1. Delete existing cluster and recreate"
echo "  2. Use existing cluster"
echo ""
```

Ask user: "Delete and recreate cluster? (yes/no)"

**If yes, delete existing cluster**:
```bash
echo "🗑️  Deleting existing cluster..."
kind delete cluster --name devsecops-cluster
echo "✅ Cluster deleted"
```

**Create the cluster**:
```bash
echo "🚀 Creating Kind cluster with 1 control plane + 2 workers..."
echo "This may take 2-5 minutes..."
echo ""

# Create cluster using config file
kind create cluster --config kind-k8s-cluster/kind-cluster-config.yaml

echo ""
echo "✅ Cluster creation completed!"
```

**Expected output**:
- Creating cluster "devsecops-cluster"
- Ensuring node image (kindest/node:v1.27.3)
- Preparing nodes
- Writing configuration
- Starting control-plane
- Installing CNI
- Installing StorageClass
- Set kubectl context to "kind-devsecops-cluster"

**If cluster creation fails**:
- Check Docker is running: `docker ps`
- Check disk space: `df -h`
- Check Docker memory allocation (Docker Desktop → Settings → Resources)
- Try deleting and recreating: `kind delete cluster --name devsecops-cluster`

**If cluster created successfully**: ✅ Move to Step 7

---

### STEP 7: Validate the Cluster

**Goal**: Verify the cluster is healthy and all nodes are ready

**Check kubectl context**:
```bash
echo "🔍 Verifying cluster setup..."
echo ""

# Show current context
kubectl config current-context
```

**Expected**: `kind-devsecops-cluster`

**Check cluster info**:
```bash
echo ""
echo "📊 Cluster Information:"
kubectl cluster-info

echo ""
echo "📋 Nodes in cluster:"
kubectl get nodes -o wide
```

**Expected**: 3 nodes
- 1 control-plane (Ready)
- 2 workers (Ready)

**Check system pods**:
```bash
echo ""
echo "🔍 System Pods Status:"
kubectl get pods -n kube-system
```

**Expected**: All pods in Running or Completed status
- coredns-* (2 pods)
- etcd-*
- kube-apiserver-*
- kube-controller-manager-*
- kube-proxy-* (3 pods, one per node)
- kube-scheduler-*
- kindnet-* (3 pods, one per node)

**Wait for all nodes to be Ready**:
```bash
echo ""
echo "⏳ Waiting for all nodes to be Ready..."

# Wait up to 5 minutes for nodes
kubectl wait --for=condition=Ready nodes --all --timeout=300s

if [ $? -eq 0 ]; then
  echo "✅ All nodes are Ready!"
else
  echo "⚠️  Timeout waiting for nodes to be Ready"
  echo "Check status: kubectl get nodes"
fi
```

**Check Docker containers (the nodes)**:
```bash
echo ""
echo "🐳 Docker Containers (Kind nodes):"
docker ps --filter "name=devsecops-cluster"
```

**Expected**: 3 containers running
- devsecops-cluster-control-plane
- devsecops-cluster-worker
- devsecops-cluster-worker2

**Test cluster connectivity**:
```bash
echo ""
echo "🔌 Testing cluster connectivity..."

# Try to list namespaces
kubectl get namespaces
```

**Expected**: List of default namespaces
- default
- kube-node-lease
- kube-public
- kube-system

**If all validations pass**: ✅ Move to Step 8

---

### STEP 8: Install Kube-ops-view

**Goal**: Deploy Kube-ops-view for cluster visualization via web interface

**What is Kube-ops-view?**
- Visual monitoring tool for Kubernetes clusters
- Real-time view of nodes, pods, and resource usage
- Shows cluster health at a glance
- Accessible via web browser
- Perfect for training and demos

**Create Kube-ops-view manifest**:
```bash
cat > kind-k8s-cluster/kube-ops-view.yaml <<EOF
---
apiVersion: v1
kind: ServiceAccount
metadata:
  name: kube-ops-view
  namespace: default
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: kube-ops-view
rules:
- apiGroups: [""]
  resources: ["nodes", "pods", "services", "replicationcontrollers", "persistentvolumeclaims", "persistentvolumes"]
  verbs: ["list", "get"]
- apiGroups: ["apps"]
  resources: ["deployments", "replicasets", "statefulsets", "daemonsets"]
  verbs: ["list", "get"]
- apiGroups: ["batch"]
  resources: ["jobs", "cronjobs"]
  verbs: ["list", "get"]
- apiGroups: ["extensions"]
  resources: ["deployments", "replicasets", "daemonsets"]
  verbs: ["list", "get"]
---
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: kube-ops-view
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: kube-ops-view
subjects:
- kind: ServiceAccount
  name: kube-ops-view
  namespace: default
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kube-ops-view
  namespace: default
  labels:
    app: kube-ops-view
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kube-ops-view
  template:
    metadata:
      labels:
        app: kube-ops-view
    spec:
      serviceAccountName: kube-ops-view
      containers:
      - name: kube-ops-view
        image: hjacobs/kube-ops-view:20.4.0
        ports:
        - containerPort: 8080
          protocol: TCP
        readinessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 5
          timeoutSeconds: 1
        livenessProbe:
          httpGet:
            path: /health
            port: 8080
          initialDelaySeconds: 30
          periodSeconds: 30
          timeoutSeconds: 10
          failureThreshold: 3
        resources:
          limits:
            cpu: 200m
            memory: 200Mi
          requests:
            cpu: 50m
            memory: 50Mi
---
apiVersion: v1
kind: Service
metadata:
  name: kube-ops-view
  namespace: default
  labels:
    app: kube-ops-view
spec:
  type: ClusterIP
  ports:
  - port: 80
    targetPort: 8080
    protocol: TCP
  selector:
    app: kube-ops-view
EOF

echo "✅ Kube-ops-view manifest created: kind-k8s-cluster/kube-ops-view.yaml"
```

**Deploy Kube-ops-view**:
```bash
echo "📊 Deploying Kube-ops-view..."
echo ""

# Apply the manifest
kubectl apply -f kind-k8s-cluster/kube-ops-view.yaml

echo ""
echo "⏳ Waiting for Kube-ops-view pod to be ready..."

# Wait for pod to be ready
kubectl wait --for=condition=Ready pod -l app=kube-ops-view --timeout=120s

if [ $? -eq 0 ]; then
  echo "✅ Kube-ops-view is ready!"
else
  echo "⚠️  Timeout waiting for Kube-ops-view"
  echo "Check status: kubectl get pods -l app=kube-ops-view"
fi
```

**Verify deployment**:
```bash
echo ""
echo "📋 Kube-ops-view Status:"
kubectl get deployment kube-ops-view
kubectl get pods -l app=kube-ops-view
kubectl get service kube-ops-view
```

**Expected output**:
- Deployment: 1/1 ready
- Pod: Running status
- Service: ClusterIP type on port 80

**If deployment successful**: ✅ Move to Step 9

---

### STEP 9: Set Up Port Forwarding for Web Access

**Goal**: Enable access to Kube-ops-view web interface from your browser

**What is Port Forwarding?**
- Maps a local port to a service running in the cluster
- Allows accessing cluster services from localhost
- Secure way to access internal services
- No need to expose services externally

**Start port forwarding**:
```bash
echo "🌐 Setting up port forwarding for Kube-ops-view..."
echo ""
echo "This will forward http://localhost:8080 to the Kube-ops-view service"
echo ""
echo "Starting port forward in background..."

# Start port forwarding in background
kubectl port-forward service/kube-ops-view 8080:80 > /dev/null 2>&1 &

# Save the process ID
PORT_FORWARD_PID=$!

# Give it a moment to start
sleep 3

# Check if port forward is running
if ps -p $PORT_FORWARD_PID > /dev/null 2>&1; then
  echo "✅ Port forwarding active!"
  echo ""
  echo "🌐 Access Kube-ops-view at: http://localhost:8080"
  echo ""
  echo "To stop port forwarding later, run:"
  echo "  kill $PORT_FORWARD_PID"
  echo ""
  echo "Or find the process and kill it:"
  echo "  ps aux | grep 'port-forward'"
  echo "  kill <PID>"
else
  echo "⚠️  Port forwarding failed to start"
  echo ""
  echo "Try manually:"
  echo "  kubectl port-forward service/kube-ops-view 8080:80"
fi
```

**Alternative: Manual port forwarding**:
```bash
# If you want to run it in foreground and see the logs:
kubectl port-forward service/kube-ops-view 8080:80

# This will show connection logs
# Press Ctrl+C to stop
```

**Test the web interface**:
```bash
echo ""
echo "🧪 Testing web interface..."
sleep 2

# Test if the service responds
curl -s -o /dev/null -w "%{http_code}" http://localhost:8080

if [ $? -eq 0 ]; then
  echo ""
  echo "✅ Kube-ops-view web interface is accessible!"
  echo ""
  echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
  echo "🎉 Open your browser and visit:"
  echo ""
  echo "    http://localhost:8080"
  echo ""
  echo "You'll see a visual representation of your cluster:"
  echo "  • 1 Control Plane node (master)"
  echo "  • 2 Worker nodes"
  echo "  • All running pods"
  echo "  • Real-time resource usage"
  echo "━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━━"
else
  echo ""
  echo "⚠️  Web interface not responding yet"
  echo "Wait a moment and try: http://localhost:8080"
fi
```

**Understanding the Kube-ops-view Interface**:
- **Nodes**: Shows all 3 nodes (control-plane + 2 workers)
- **Pods**: Visual representation of pods on each node
- **Colors**: Green = healthy, Red = issues
- **CPU/Memory**: Resource usage bars
- **Auto-refresh**: Updates in real-time

**Troubleshooting port forwarding**:
```bash
# Check if port 8080 is already in use
lsof -i :8080

# If port is busy, use a different port
kubectl port-forward service/kube-ops-view 8081:80

# Check if service exists
kubectl get service kube-ops-view

# Check if pod is running
kubectl get pods -l app=kube-ops-view

# View pod logs if there are issues
kubectl logs -l app=kube-ops-view
```

**If port forwarding successful**: ✅ Move to Step 10

---

### STEP 10: Deploy Test Application

**Goal**: Deploy a simple app to verify the cluster works (and see it in Kube-ops-view!)

**Create a test deployment**:
```bash
echo "🧪 Deploying test application..."
echo ""

# Deploy nginx as a test
kubectl create deployment nginx-test --image=nginx:alpine

# Expose it via NodePort
kubectl expose deployment nginx-test --type=NodePort --port=80 --target-port=80

# Wait for deployment to be ready
kubectl wait --for=condition=Available deployment/nginx-test --timeout=120s

echo ""
echo "✅ Test application deployed!"
```

**Check the deployment**:
```bash
echo ""
echo "📦 Deployment Status:"
kubectl get deployment nginx-test

echo ""
echo "🔍 Pods:"
kubectl get pods -l app=nginx-test -o wide

echo ""
echo "🌐 Service:"
kubectl get service nginx-test
```

**Get the NodePort**:
```bash
echo ""
NODE_PORT=$(kubectl get service nginx-test -o jsonpath='{.spec.ports[0].nodePort}')
echo "📍 Service is available at: http://localhost:$NODE_PORT"
echo ""
echo "Test it with: curl http://localhost:$NODE_PORT"
```

**Test the application**:
```bash
echo ""
echo "🧪 Testing application..."
sleep 5  # Give it a moment

curl -s http://localhost:$NODE_PORT | head -5

if [ $? -eq 0 ]; then
  echo ""
  echo "✅ Application is responding!"
else
  echo ""
  echo "⚠️  Application not responding yet. Wait a moment and try:"
  echo "   curl http://localhost:$NODE_PORT"
fi
```

**Cleanup test deployment** (optional):
```bash
echo ""
echo "🧹 Cleanup test deployment? (We'll keep it for now)"
echo "To clean up later, run:"
echo "  kubectl delete deployment nginx-test"
echo "  kubectl delete service nginx-test"
```

**If test successful**: ✅ Move to Step 11

**Pro Tip**: Open Kube-ops-view in your browser (http://localhost:8080) and watch the test application appear on the cluster visualization!

---

### STEP 11: Provide Cluster Access Information

**Goal**: Give user all the information to work with their cluster

**Create a summary file**:
```bash
cat > kind-k8s-cluster/kind-cluster-info.txt <<EOF
================================================================================
Kind Kubernetes Cluster - Access Information
================================================================================

Cluster Name: devsecops-cluster
Kubernetes Version: v1.27.3
Nodes: 1 control plane + 2 workers (3 total)

KUBECTL CONTEXT
===============
Current context: $(kubectl config current-context)

Switch to this cluster:
  kubectl config use-context kind-devsecops-cluster

CLUSTER ACCESS
==============
Get cluster info:
  kubectl cluster-info

View all nodes:
  kubectl get nodes -o wide

View all pods across all namespaces:
  kubectl get pods --all-namespaces

COMMON COMMANDS
===============
# Create a deployment
kubectl create deployment myapp --image=nginx

# Expose as NodePort service
kubectl expose deployment myapp --type=NodePort --port=80

# Get service URL
kubectl get service myapp

# View logs
kubectl logs -l app=myapp

# Execute command in pod
kubectl exec -it <pod-name> -- /bin/sh

# Delete resources
kubectl delete deployment myapp
kubectl delete service myapp

DEPLOY YOUR JUICE SHOP
=======================
# Deploy the image you pushed to Docker Hub
kubectl create deployment juice-shop --image=rupeedev/owasp-juice-shop:latest

# Expose via NodePort (port 30080 is mapped to localhost)
kubectl expose deployment juice-shop --type=NodePort --port=3000 --target-port=3000

# Or use LoadBalancer (Kind simulates it)
kubectl expose deployment juice-shop --type=LoadBalancer --port=3000 --target-port=3000

# Check status
kubectl get deployment juice-shop
kubectl get pods -l app=juice-shop
kubectl get service juice-shop

# Access the app
# Get the NodePort:
kubectl get service juice-shop -o jsonpath='{.spec.ports[0].nodePort}'

# Then visit: http://localhost:<nodeport>

CLUSTER MANAGEMENT
==================
# View cluster config
kind get clusters

# Export kubeconfig
kind get kubeconfig --name devsecops-cluster

# SSH into a node
docker exec -it devsecops-cluster-control-plane bash
docker exec -it devsecops-cluster-worker bash

# View node logs
docker logs devsecops-cluster-control-plane

# Stop cluster (keep it, just stop containers)
docker stop devsecops-cluster-control-plane devsecops-cluster-worker devsecops-cluster-worker2

# Start cluster again
docker start devsecops-cluster-control-plane devsecops-cluster-worker devsecops-cluster-worker2

# Delete cluster completely
kind delete cluster --name devsecops-cluster

TROUBLESHOOTING
===============
# Cluster not responding
kubectl cluster-info dump

# Node issues
kubectl describe node <node-name>

# Pod issues
kubectl describe pod <pod-name>
kubectl logs <pod-name>

# Events
kubectl get events --all-namespaces --sort-by='.lastTimestamp'

# Check Docker
docker ps --filter "name=devsecops-cluster"

# Recreate cluster
kind delete cluster --name devsecops-cluster
kind create cluster --config kind-k8s-cluster/kind-cluster-config.yaml

PORT MAPPINGS
=============
localhost:30080 → NodePort 30080 (HTTP)
localhost:30443 → NodePort 30443 (HTTPS)

To use these fixed ports, create services with:
  kubectl expose deployment myapp --type=NodePort --port=80 --node-port=30080

NEXT STEPS
==========
1. Deploy Juice Shop from Docker Hub
2. Create Kubernetes manifests (Deployment + Service YAML)
3. Practice scaling: kubectl scale deployment juice-shop --replicas=3
4. Update deployment: kubectl set image deployment/juice-shop juice-shop=rupeedev/owasp-juice-shop:simple
5. Add to CI/CD pipeline: Deploy to Kind after Docker Hub push

RESOURCES
=========
- Kind Docs: https://kind.sigs.k8s.io/
- Kubectl Cheat Sheet: https://kubernetes.io/docs/reference/kubectl/cheatsheet/
- K8s Concepts: https://kubernetes.io/docs/concepts/

================================================================================
Cluster created successfully! Happy Kuberneting! 🚀
================================================================================
EOF

echo "✅ Cluster information saved to: kind-k8s-cluster/kind-cluster-info.txt"
echo ""
cat kind-k8s-cluster/kind-cluster-info.txt
```

**Display final summary**:
```bash
echo ""
echo "╔══════════════════════════════════════════════════════════════════╗"
echo "║           🎉 Kind Cluster Setup Complete! 🎉                     ║"
echo "╚══════════════════════════════════════════════════════════════════╝"
echo ""
echo "✅ Cluster Name: devsecops-cluster"
echo "✅ Nodes: 1 control plane + 2 workers"
echo "✅ Kubernetes version: v1.27.3"
echo "✅ kubectl context set to: kind-devsecops-cluster"
echo ""
echo "Quick Start:"
echo "  kubectl get nodes              # View all nodes"
echo "  kubectl get pods -A            # View all pods"
echo "  kubectl create deployment test --image=nginx"
echo ""
echo "Deploy Juice Shop:"
echo "  kubectl create deployment juice-shop --image=rupeedev/owasp-juice-shop:latest"
echo "  kubectl expose deployment juice-shop --type=NodePort --port=3000"
echo ""
echo "📋 Full cluster info saved to: kind-k8s-cluster/kind-cluster-info.txt"
echo ""
echo "🚀 Your local Kubernetes cluster is ready for DevSecOps training!"
echo ""
```

**If everything successful**: ✅ Cluster Setup Complete!

---

### STEP 12: Create GitHub Personal Access Token (PAT) for Runner

**Goal**: Create authentication token for GitHub Actions Runner to register with your repository

**What is this step?**
- GitHub Actions Runner needs to authenticate with GitHub
- PAT (Personal Access Token) provides secure authentication
- Token is used by runner pod to register automatically
- One-time setup per repository

**Why do we need this?**
- Runner pod will register itself with your GitHub repo
- Enables the runner to receive workflow jobs
- Required for CI/CD automation

**Create GitHub PAT**:
```bash
echo "🔐 Setting up GitHub Authentication for Runner..."
echo ""
echo "📝 Creating GitHub Personal Access Token (PAT):"
echo ""
echo "1. Go to: https://github.com/settings/tokens/new"
echo ""
echo "2. Fill in token details:"
echo "   - Note: 'kind-cluster-runner' (or any descriptive name)"
echo "   - Expiration: 90 days (or as needed)"
echo ""
echo "3. Select scopes (permissions):"
echo "   ✓ repo (Full control of private repositories)"
echo "   ✓ workflow (Update GitHub Action workflows)"
echo "   ✓ admin:org (if using organization runners)"
echo ""
echo "4. Click 'Generate token' button at bottom"
echo ""
echo "5. ⚠️  IMPORTANT: Copy the token NOW!"
echo "   - Token will only be shown once"
echo "   - Format: ghp_xxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxxx"
echo "   - Save it safely for next step"
echo ""
echo "6. Click 'Copy' button"
echo ""
```

**Ask user to confirm**:
"Have you created and copied the GitHub PAT? (yes/no)"

**If no**: Guide them through the process again
**If yes**: Ask for repository details

**Collect repository information**:
```bash
echo ""
echo "📋 Repository Information Needed:"
echo ""
```

**Ask the user**:
"What is your GitHub username or organization? (e.g., 'rupeshpanwar')"

Store response as: `GITHUB_OWNER`

**Ask the user**:
"What is your repository name? (e.g., 'owasp-juice-shop')"

Store response as: `GITHUB_REPO`

**Ask the user**:
"Please paste your GitHub PAT (it will be stored securely as a Kubernetes secret):"

Store response as: `GITHUB_PAT`

**Verify inputs**:
```bash
echo ""
echo "✅ Configuration Summary:"
echo "  GitHub Owner: ${GITHUB_OWNER}"
echo "  Repository: ${GITHUB_REPO}"
echo "  Full Repo: https://github.com/${GITHUB_OWNER}/${GITHUB_REPO}"
echo "  PAT: ghp_***...*** (hidden)"
echo ""
```

**Ask user to confirm**:
"Is this information correct? (yes/no)"

**If yes**: ✅ Move to Step 13

**Security Note**:
```bash
echo ""
echo "🔒 Security Notes:"
echo "  - PAT will be stored as a Kubernetes secret"
echo "  - Secret is encrypted in etcd (K8s datastore)"
echo "  - Only runner pod can access it"
echo "  - Not visible in kubectl describe output"
echo "  - Deleted when cluster is deleted"
echo ""
```

---

### STEP 13: Create Kubernetes Secret with GitHub Credentials

**Goal**: Store GitHub credentials securely in Kubernetes for runner pod to use

**What is a Kubernetes Secret?**
- Encrypted storage for sensitive data
- Mounted as environment variables in pods
- Base64 encoded
- Protected by K8s RBAC

**Create the secret**:
```bash
echo "🔐 Creating Kubernetes secret for GitHub credentials..."
echo ""

# Create secret with GitHub PAT and repo info
kubectl create secret generic github-runner-secret \
  --from-literal=GITHUB_OWNER="${GITHUB_OWNER}" \
  --from-literal=GITHUB_REPO="${GITHUB_REPO}" \
  --from-literal=GITHUB_PAT="${GITHUB_PAT}" \
  --namespace=default

if [ $? -eq 0 ]; then
  echo "✅ Secret 'github-runner-secret' created successfully!"
else
  echo "❌ Failed to create secret"
  echo "Check if secret already exists: kubectl get secret github-runner-secret"
fi
```

**Verify secret creation**:
```bash
echo ""
echo "🔍 Verifying secret..."
kubectl get secret github-runner-secret -n default

echo ""
echo "📋 Secret details (values are hidden):"
kubectl describe secret github-runner-secret -n default
```

**Expected output**:
- Secret exists with type `Opaque`
- Contains 3 data items (GITHUB_OWNER, GITHUB_REPO, GITHUB_PAT)
- Data values are shown as byte counts, not actual values

**Test secret can be read** (optional verification):
```bash
echo ""
echo "🧪 Testing secret access (owner only):"
kubectl get secret github-runner-secret -o jsonpath='{.data.GITHUB_OWNER}' | base64 -d
echo ""
```

**Expected**: Shows your GitHub username

**If secret created successfully**: ✅ Move to Step 14

**Troubleshooting**:
```bash
# If secret already exists and you need to update it:
kubectl delete secret github-runner-secret -n default

# Then recreate with new values
kubectl create secret generic github-runner-secret \
  --from-literal=GITHUB_OWNER="${GITHUB_OWNER}" \
  --from-literal=GITHUB_REPO="${GITHUB_REPO}" \
  --from-literal=GITHUB_PAT="${GITHUB_PAT}" \
  --namespace=default
```

---

### STEP 14: Deploy GitHub Actions Runner Pod to Kind Cluster

**Goal**: Deploy a runner pod that registers with GitHub and executes workflows

**What is the Runner Pod?**
- Containerized GitHub Actions runner
- Runs inside your Kind cluster
- Auto-registers with your GitHub repository
- Listens for workflow jobs
- Has kubectl access to deploy to same cluster

**Create runner manifest file**:
```bash
echo "📝 Creating GitHub Actions Runner manifest..."
echo ""

cat > kind-k8s-cluster/github-runner.yaml <<EOF
---
# ServiceAccount for the runner pod
apiVersion: v1
kind: ServiceAccount
metadata:
  name: github-runner
  namespace: default
---
# ClusterRole with permissions to deploy applications
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRole
metadata:
  name: github-runner
rules:
  # Full access to deployments, pods, services
  - apiGroups: ["apps"]
    resources: ["deployments", "replicasets", "statefulsets"]
    verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
  - apiGroups: [""]
    resources: ["pods", "services", "configmaps", "secrets"]
    verbs: ["get", "list", "watch", "create", "update", "patch", "delete"]
  # Access to view nodes and namespaces
  - apiGroups: [""]
    resources: ["nodes", "namespaces"]
    verbs: ["get", "list", "watch"]
  # Access to rollout status
  - apiGroups: ["apps"]
    resources: ["deployments/status"]
    verbs: ["get", "watch"]
---
# ClusterRoleBinding to grant permissions to ServiceAccount
apiVersion: rbac.authorization.k8s.io/v1
kind: ClusterRoleBinding
metadata:
  name: github-runner
roleRef:
  apiGroup: rbac.authorization.k8s.io
  kind: ClusterRole
  name: github-runner
subjects:
  - kind: ServiceAccount
    name: github-runner
    namespace: default
---
# Deployment for GitHub Actions Runner
apiVersion: apps/v1
kind: Deployment
metadata:
  name: github-runner
  namespace: default
  labels:
    app: github-runner
spec:
  replicas: 1
  selector:
    matchLabels:
      app: github-runner
  template:
    metadata:
      labels:
        app: github-runner
    spec:
      serviceAccountName: github-runner
      containers:
      - name: runner
        image: myoung34/github-runner:latest
        env:
          # GitHub repository URL
          - name: REPO_URL
            value: "https://github.com/\$(GITHUB_OWNER)/\$(GITHUB_REPO)"
          # GitHub owner from secret
          - name: GITHUB_OWNER
            valueFrom:
              secretKeyRef:
                name: github-runner-secret
                key: GITHUB_OWNER
          # GitHub repository name from secret
          - name: GITHUB_REPO
            valueFrom:
              secretKeyRef:
                name: github-runner-secret
                key: GITHUB_REPO
          # GitHub PAT from secret
          - name: ACCESS_TOKEN
            valueFrom:
              secretKeyRef:
                name: github-runner-secret
                key: GITHUB_PAT
          # Runner name (appears in GitHub UI)
          - name: RUNNER_NAME
            value: "kind-cluster-runner"
          # Runner labels (used in runs-on)
          - name: LABELS
            value: "self-hosted,kind-cluster,local"
          # Auto-remove runner on pod termination
          - name: EPHEMERAL
            value: "true"
          # Don't run as root
          - name: RUNNER_WORKDIR
            value: "/tmp/runner"
        resources:
          requests:
            cpu: 100m
            memory: 256Mi
          limits:
            cpu: 500m
            memory: 512Mi
        volumeMounts:
          # Mount docker socket for docker commands
          - name: docker-sock
            mountPath: /var/run/docker.sock
      volumes:
        - name: docker-sock
          hostPath:
            path: /var/run/docker.sock
            type: Socket
      # Restart policy
      restartPolicy: Always
EOF

echo "✅ Runner manifest created: kind-k8s-cluster/github-runner.yaml"
```

**Show manifest to user**:
```bash
echo ""
echo "📋 Runner Configuration:"
cat kind-k8s-cluster/github-runner.yaml
```

**Deploy the runner**:
```bash
echo ""
echo "🚀 Deploying GitHub Actions Runner to cluster..."
echo ""

# Apply the manifest
kubectl apply -f kind-k8s-cluster/github-runner.yaml

if [ $? -eq 0 ]; then
  echo "✅ Runner deployment created!"
else
  echo "❌ Failed to deploy runner"
  exit 1
fi
```

**Wait for runner pod to start**:
```bash
echo ""
echo "⏳ Waiting for runner pod to be ready..."
echo "This may take 1-2 minutes (downloading image, registering with GitHub)..."
echo ""

# Wait up to 3 minutes for pod to be ready
kubectl wait --for=condition=Ready pod -l app=github-runner --timeout=180s -n default

if [ $? -eq 0 ]; then
  echo "✅ Runner pod is ready!"
else
  echo "⚠️  Timeout waiting for runner pod"
  echo "Check status: kubectl get pods -l app=github-runner"
  echo "View logs: kubectl logs -l app=github-runner"
fi
```

**Check runner status**:
```bash
echo ""
echo "📋 Runner Pod Status:"
kubectl get deployment github-runner -n default
kubectl get pods -l app=github-runner -n default
echo ""
```

**View runner logs** (to confirm registration):
```bash
echo "📝 Runner registration logs:"
kubectl logs -l app=github-runner -n default --tail=50
echo ""
```

**Expected in logs**:
- "Runner successfully registered"
- "Listening for Jobs"
- "Runner name: kind-cluster-runner"
- "Labels: self-hosted,kind-cluster,local"

**If deployment successful**: ✅ Move to Step 15

**Troubleshooting**:
```bash
# If pod fails to start
kubectl describe pod -l app=github-runner

# If image pull fails
kubectl get events --sort-by='.lastTimestamp' | grep runner

# If registration fails (check PAT and repo name)
kubectl logs -l app=github-runner --all-containers=true

# Restart runner pod
kubectl rollout restart deployment github-runner

# Delete and recreate
kubectl delete -f kind-k8s-cluster/github-runner.yaml
kubectl apply -f kind-k8s-cluster/github-runner.yaml
```

---

### STEP 15: Verify Runner Registration in GitHub

**Goal**: Confirm the runner appears in your GitHub repository settings

**Check GitHub UI**:
```bash
echo "🔍 Verifying runner registration in GitHub..."
echo ""
echo "📋 Steps to verify:"
echo ""
echo "1. Open your browser and go to:"
echo "   https://github.com/${GITHUB_OWNER}/${GITHUB_REPO}/settings/actions/runners"
echo ""
echo "2. Look for a runner named: 'kind-cluster-runner'"
echo ""
echo "3. Status should be: 🟢 Idle (green dot)"
echo ""
echo "4. Labels should include: self-hosted, kind-cluster, local"
echo ""
echo "5. Runner should show as Active/Online"
echo ""
```

**Open URL automatically** (if possible):
```bash
# Try to open browser (macOS)
if [[ "$OSTYPE" == "darwin"* ]]; then
  open "https://github.com/${GITHUB_OWNER}/${GITHUB_REPO}/settings/actions/runners"
elif [[ "$OSTYPE" == "linux-gnu"* ]]; then
  xdg-open "https://github.com/${GITHUB_OWNER}/${GITHUB_REPO}/settings/actions/runners" 2>/dev/null || echo "Please open the URL manually"
fi
```

**Ask user to confirm**:
"Can you see the 'kind-cluster-runner' in your GitHub repository? (yes/no)"

**If yes**: 🎉 **Runner setup complete!**

**If no**: Troubleshoot

**Troubleshooting runner registration**:
```bash
echo ""
echo "🔧 Troubleshooting runner registration..."
echo ""

# Check runner pod logs for errors
echo "1. Check runner logs for errors:"
kubectl logs -l app=github-runner -n default

echo ""
echo "2. Common issues:"
echo "   - Invalid PAT: Check token has correct scopes (repo, workflow)"
echo "   - Wrong repo name: Verify GITHUB_OWNER and GITHUB_REPO"
echo "   - Token expired: Create new PAT and update secret"
echo "   - Network issues: Check pod can reach github.com"
echo ""

# Test network connectivity from pod
echo "3. Test GitHub connectivity:"
kubectl exec -it deployment/github-runner -- curl -s https://api.github.com/

echo ""
echo "4. Verify secret values:"
kubectl get secret github-runner-secret -o jsonpath='{.data.GITHUB_OWNER}' | base64 -d
echo ""

# Restart runner if needed
echo "5. Restart runner pod:"
echo "   kubectl rollout restart deployment github-runner"
echo ""
```

**Final verification**:
```bash
echo ""
echo "✅ Final Checks:"
echo ""

# Check all runner resources
echo "📦 Runner Resources:"
kubectl get serviceaccount github-runner
kubectl get clusterrole github-runner
kubectl get clusterrolebinding github-runner
kubectl get deployment github-runner
kubectl get pods -l app=github-runner

echo ""
echo "🔐 Runner Secret:"
kubectl get secret github-runner-secret

echo ""
```

**Update cluster info file**:
```bash
echo ""
echo "📝 Updating cluster info with runner details..."

cat >> kind-k8s-cluster/kind-cluster-info.txt <<EOF

================================================================================
GITHUB ACTIONS RUNNER (NEW!)
================================================================================

Runner Pod: github-runner
Status: Check with 'kubectl get pods -l app=github-runner'
GitHub URL: https://github.com/${GITHUB_OWNER}/${GITHUB_REPO}/settings/actions/runners

RUNNER DETAILS
==============
Runner Name: kind-cluster-runner
Labels: self-hosted, kind-cluster, local
Location: Running inside Kind cluster
Access: Can deploy to same cluster via kubectl

USING THE RUNNER
================
In your .github/workflows/*.yml file, use:

  jobs:
    deploy-to-kind:
      runs-on: self-hosted  # This uses your runner!
      steps:
        - name: Deploy to Kind
          run: |
            kubectl apply -f k8s/deployment.yaml
            kubectl rollout status deployment/juice-shop

RUNNER MANAGEMENT
=================
# View runner logs
kubectl logs -l app=github-runner -f

# Restart runner
kubectl rollout restart deployment github-runner

# Scale runner (add more replicas)
kubectl scale deployment github-runner --replicas=2

# Delete runner
kubectl delete -f kind-k8s-cluster/github-runner.yaml

# Update runner secret (if PAT expires)
kubectl delete secret github-runner-secret
kubectl create secret generic github-runner-secret \
  --from-literal=GITHUB_OWNER="..." \
  --from-literal=GITHUB_REPO="..." \
  --from-literal=GITHUB_PAT="..."

TROUBLESHOOTING
===============
# Check runner status
kubectl get pods -l app=github-runner

# View detailed info
kubectl describe pod -l app=github-runner

# Check logs for errors
kubectl logs -l app=github-runner --tail=100

# Test GitHub connectivity
kubectl exec -it deployment/github-runner -- curl https://api.github.com

# Verify secret
kubectl get secret github-runner-secret -o yaml

================================================================================
Runner Setup Complete! Your CI/CD pipeline can now deploy to this cluster!
================================================================================
EOF

echo "✅ Cluster info updated with runner details"
```

**Display success message**:
```bash
echo ""
echo "╔════════════════════════════════════════════════════════════════╗"
echo "║     🎉 GitHub Actions Runner Setup Complete! 🎉                ║"
echo "╚════════════════════════════════════════════════════════════════╝"
echo ""
echo "✅ Runner Pod: Deployed and running"
echo "✅ GitHub Registration: Active"
echo "✅ Cluster Access: Configured with RBAC"
echo "✅ Ready to Deploy: From GitHub Actions workflows"
echo ""
echo "🌐 View runner in GitHub:"
echo "   https://github.com/${GITHUB_OWNER}/${GITHUB_REPO}/settings/actions/runners"
echo ""
echo "📋 Runner Details:"
echo "   Name: kind-cluster-runner"
echo "   Labels: self-hosted, kind-cluster, local"
echo "   Status: 🟢 Idle (waiting for jobs)"
echo ""
echo "🚀 Next Steps:"
echo "  1. Update your workflow to use: runs-on: self-hosted"
echo "  2. Add deployment stage to push Docker images"
echo "  3. Deploy apps from your CI/CD pipeline!"
echo ""
echo "💡 Quick Test:"
echo "   git push → GitHub Actions → Runner Pod → Deploy to Kind"
echo ""
```

**If everything successful**: ✅ **Complete Setup Done!**

---

## Success Criteria

By the end of this setup, the trainee should have:

**Basic Setup (Steps 1-11)**:
□ Kind CLI installed and verified
□ kubectl CLI installed and verified
□ 3-node Kubernetes cluster running (1 control-plane + 2 workers)
□ All nodes in Ready status
□ System pods running correctly
□ Kube-ops-view deployed and accessible
□ Port forwarding configured (http://localhost:8080)
□ Visual cluster monitoring via web browser
□ Test application deployed and accessible
□ kubectl context set to kind-devsecops-cluster
□ Cluster info file saved in kind-k8s-cluster/ directory
□ All manifest files organized in kind-k8s-cluster/ directory

**Runner Setup (Steps 12-15)**:
□ GitHub PAT (Personal Access Token) created with correct scopes
□ Kubernetes secret created with GitHub credentials
□ GitHub Actions Runner pod deployed to cluster
□ Runner registered and visible in GitHub repository settings
□ Runner shows as 🟢 Idle/Active in GitHub UI
□ Runner has labels: self-hosted, kind-cluster, local
□ RBAC configured (ServiceAccount, ClusterRole, ClusterRoleBinding)
□ github-runner.yaml manifest saved in kind-k8s-cluster/
□ Cluster info file updated with runner details

**Understanding & Readiness**:
□ Understanding of how to deploy applications to K8s
□ Understanding of self-hosted runners in K8s
□ Ready to deploy Juice Shop from Docker Hub via CI/CD
□ Ready to use runner in GitHub Actions workflows

---

## What We Built

**Summary**:

**Cluster Infrastructure (Steps 1-11)**:
- ✅ Kind (Kubernetes in Docker) installed
- ✅ kubectl CLI for cluster management
- ✅ Multi-node cluster (production-like)
- ✅ 1 Control Plane node (master)
- ✅ 2 Worker nodes (for app workloads)
- ✅ Kube-ops-view for cluster visualization
- ✅ Port forwarding configured (localhost:8080)
- ✅ Port mappings (localhost:30080, 30443)
- ✅ Cluster validated and tested
- ✅ Zero cloud costs (100% local)

**CI/CD Integration (Steps 12-15)**:
- ✅ GitHub Actions Runner pod deployed
- ✅ Runner registered with GitHub repository
- ✅ RBAC configured for secure deployments
- ✅ Runner can deploy to same cluster
- ✅ Full CI/CD automation enabled
- ✅ Self-hosted runner in Kubernetes
- ✅ Secure credential management (K8s secrets)
- ✅ Ready for automated deployments

**Key Learning**:
- Kubernetes architecture (control plane + workers)
- kubectl command-line tool
- Pods, Deployments, Services concepts
- Visual cluster monitoring with Kube-ops-view
- Port forwarding for accessing cluster services
- NodePort for external access
- Local development before cloud deployment
- Docker containers as K8s nodes
- Multi-node cluster for realistic testing
- Real-time cluster visualization and monitoring

**Architecture**:
```
┌─────────────────────────────────────────────┐
│           Your Local Machine                │
│                                             │
│  ┌────────────────────────────────────┐    │
│  │         Docker Desktop             │    │
│  │                                    │    │
│  │  ┌──────────────────────────────┐ │    │
│  │  │  Kind Cluster: devSecOps     │ │    │
│  │  │                              │ │    │
│  │  │  ┌────────────────────────┐ │ │    │
│  │  │  │  Control Plane Node    │ │ │    │
│  │  │  │  (Master)              │ │ │    │
│  │  │  └────────────────────────┘ │ │    │
│  │  │                              │ │    │
│  │  │  ┌────────────────────────┐ │ │    │
│  │  │  │  Worker Node 1         │ │ │    │
│  │  │  │  (Pods run here)       │ │ │    │
│  │  │  └────────────────────────┘ │ │    │
│  │  │                              │ │    │
│  │  │  ┌────────────────────────┐ │ │    │
│  │  │  │  Worker Node 2         │ │ │    │
│  │  │  │  (Pods run here)       │ │ │    │
│  │  │  └────────────────────────┘ │ │    │
│  │  │                              │ │    │
│  │  └──────────────────────────────┘ │    │
│  │                                    │    │
│  └────────────────────────────────────┘    │
│                                             │
│  kubectl ←→ Kind Cluster                   │
│                                             │
└─────────────────────────────────────────────┘
```

---

## Next Steps After Setup

The participant can now:
1. **Deploy Juice Shop to K8s** - Use image from Docker Hub
2. **Create K8s manifests** - YAML files for Deployment/Service
3. **Practice scaling** - Scale pods up/down
4. **Add to CI/CD** - Deploy to Kind from GitHub Actions
5. **Learn K8s concepts** - Pods, Services, Ingress, ConfigMaps
6. **Prepare for cloud** - Same concepts work on EKS, GKE, AKS

**Quick reference commands**:
```bash
# View cluster
kubectl get nodes
kubectl cluster-info

# Deploy from Docker Hub
kubectl create deployment juice-shop --image=rupeedev/owasp-juice-shop:latest
kubectl expose deployment juice-shop --type=NodePort --port=3000

# Check status
kubectl get deployments
kubectl get pods
kubectl get services

# Scale
kubectl scale deployment juice-shop --replicas=3

# Update image
kubectl set image deployment/juice-shop juice-shop=rupeedev/owasp-juice-shop:simple

# View logs
kubectl logs -l app=juice-shop

# Delete
kubectl delete deployment juice-shop
kubectl delete service juice-shop

# Cluster management
kind get clusters
kind delete cluster --name devsecops-cluster
```

---

## Troubleshooting Guide

### Issue: "Docker is not running"
**Fix**:
```bash
# macOS
open -a Docker

# Linux
sudo systemctl start docker

# Check status
docker ps
```

### Issue: "Kind command not found"
**Fix**:
```bash
# Check installation
which kind

# Reinstall
# macOS
brew install kind

# Linux
curl -Lo ./kind https://kind.sigs.k8s.io/dl/v0.20.0/kind-linux-amd64
chmod +x ./kind
sudo mv ./kind /usr/local/bin/kind
```

### Issue: "Cluster creation failed - not enough memory"
**Fix**:
```bash
# Check Docker Desktop settings
# Docker Desktop → Settings → Resources
# Increase Memory to at least 4GB
# Increase CPUs to at least 2
# Apply & Restart

# Then retry
kind delete cluster --name devsecops-cluster
kind create cluster --config kind-k8s-cluster/kind-cluster-config.yaml
```

### Issue: "Nodes not ready"
**Fix**:
```bash
# Wait a bit longer
kubectl wait --for=condition=Ready nodes --all --timeout=600s

# Check node status
kubectl describe nodes

# Check system pods
kubectl get pods -n kube-system

# Restart cluster
kind delete cluster --name devsecops-cluster
kind create cluster --config kind-k8s-cluster/kind-cluster-config.yaml
```

### Issue: "kubectl: connection refused"
**Fix**:
```bash
# Check cluster is running
kind get clusters

# Check Docker containers
docker ps --filter "name=devsecops-cluster"

# Reset kubectl context
kubectl config use-context kind-devsecops-cluster

# Verify
kubectl cluster-info
```

### Issue: "Cannot access application via NodePort"
**Fix**:
```bash
# Check service
kubectl get service <service-name>

# Get NodePort
kubectl get service <service-name> -o jsonpath='{.spec.ports[0].nodePort}'

# Check pod is running
kubectl get pods

# Check if using correct port
# Kind maps container ports to localhost
# Use: http://localhost:<nodeport>

# For fixed port 30080, ensure service uses it:
kubectl expose deployment myapp --type=NodePort --port=80 --node-port=30080
# Then access: http://localhost:30080
```

### Issue: "Image pull error from Docker Hub"
**Fix**:
```bash
# Check image exists
docker pull rupeedev/owasp-juice-shop:latest

# If pull works, try K8s deployment again
kubectl create deployment juice-shop --image=rupeedev/owasp-juice-shop:latest

# Check pod events
kubectl describe pod <pod-name>

# If private image, create secret:
kubectl create secret docker-registry dockerhub-secret \
  --docker-server=https://index.docker.io/v1/ \
  --docker-username=rupeedev \
  --docker-password=<your-token>

# Then reference in deployment
```

---

## Training Session Best Practices

**For Trainers**:
- Run through setup yourself first
- Ensure Docker Desktop is properly configured (6GB+ memory for runner)
- Have participants do all steps during training
- **Phase 1 (Steps 1-11)**: Allow 15-20 minutes for cluster setup
- **Phase 2 (Steps 12-15)**: Allow 10-15 minutes for runner setup
- **Total time**: 25-35 minutes for complete setup
- Consider splitting into two sessions: Cluster first, then Runner
- Emphasize: This is local, no cloud costs!
- Show the Docker Desktop UI to see Kind containers
- Demonstrate runner appearing in GitHub UI

**For Participants**:
- Follow steps in order
- Don't skip validation steps
- Ask questions if stuck
- Remember: Each node is a Docker container
- Runner pod is just another container in the cluster
- Practice kubectl commands after setup
- Keep kind-k8s-cluster/ directory as reference for all manifests and info files
- Save your GitHub PAT securely (needed if recreating runner)
- **Optional**: Stop after Step 11 if you only need local cluster without CI/CD

---

## Your Instructions as the Assistant

1. **Start with Step 1** - Detect OS first
2. **Wait for user confirmation** after each major step
3. **Validate before proceeding** - Run verification commands
4. **Troubleshoot proactively** - If something fails, help fix it
5. **Explain what's happening** - Don't just give commands
6. **Use code blocks** - Make commands easy to copy
7. **Track progress** - Remind user which step they're on
8. **Celebrate success** - Acknowledge when cluster is ready
9. **Provide next steps** - Guide to deploying Juice Shop
10. **Save artifacts** - Create info files for reference

## Start the Setup

Begin by asking the user:

"Welcome to Kind Setup - Local Kubernetes Cluster with CI/CD! I'll guide you through setting up a production-like Kubernetes environment on your local machine with automated deployment capabilities. This setup includes 15 comprehensive steps.

**What we're building**:
- 3-node Kubernetes cluster (1 control plane + 2 workers)
- Running inside Docker containers
- GitHub Actions Runner pod for CI/CD automation
- Zero cloud costs
- Perfect for testing deployments locally with full CI/CD pipeline

**Two-Phase Setup**:
📦 **Phase 1 (Steps 1-11, ~15-20 minutes)**: Cluster Infrastructure
  - Install Kind & kubectl
  - Create 3-node cluster
  - Deploy Kube-ops-view
  - Validate cluster

🚀 **Phase 2 (Steps 12-15, ~10-15 minutes)**: CI/CD Integration
  - Create GitHub PAT
  - Deploy runner pod
  - Register with GitHub
  - Enable automated deployments

**Prerequisites**:
1. Docker Desktop installed and running
2. At least 4GB RAM available (6GB recommended for runner)
3. At least 20GB disk space
4. Admin/sudo access for installations
5. GitHub account with repository access

**Optional**: You can stop after Step 11 if you only need the cluster, or continue to Step 15 for full CI/CD automation.

Let me start by detecting your operating system..."

Then proceed with Step 1: Detect Operating System
