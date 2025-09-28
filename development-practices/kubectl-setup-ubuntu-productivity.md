# kubectl Setup on Ubuntu: Installation and Productivity Tips

## Overview

Setting up kubectl efficiently can significantly improve your Kubernetes workflow. This guide covers installing kubectl on Ubuntu and configuring productivity-enhancing aliases and shortcuts that streamline common Kubernetes operations. Based on the comprehensive [kubectl setup guide](https://github.com/nathanfox/nginx-dev-gateway/blob/develop/docs/kubectl-setup.md) and [setup script](https://github.com/nathanfox/nginx-dev-gateway/blob/develop/scripts/setup-kubectl-aliases.sh) from the nginx-dev-gateway repository.

## The Problem

Working with Kubernetes clusters via kubectl involves typing lengthy commands repeatedly:
- `kubectl get pods -n my-namespace` becomes tedious when executed dozens of times daily
- Switching between namespaces requires remembering full command syntax
- Common operations like following logs or executing commands in pods require verbose command-line arguments
- Managing multiple clusters and contexts adds complexity

These repetitive tasks slow down development and increase the cognitive load of Kubernetes operations.

## Installation Methods

There are four primary ways to install kubectl on Ubuntu:

### 1. Official apt Repository (Recommended)

```bash
# Install prerequisites
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl

# Add Google Cloud public signing key
curl -fsSL https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-archive-keyring.gpg

# Add Kubernetes apt repository
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list

# Install kubectl
sudo apt-get update
sudo apt-get install -y kubectl
```

### 2. Direct Binary Download

```bash
# Download latest release
curl -LO "https://dl.k8s.io/release/$(curl -L -s https://dl.k8s.io/release/stable.txt)/bin/linux/amd64/kubectl"

# Make executable and move to PATH
chmod +x kubectl
sudo mv kubectl /usr/local/bin/
```

### 3. Snap Package

```bash
sudo snap install kubectl --classic
```

### 4. Via Azure CLI (for AKS users)

```bash
az aks install-cli
```

## Productivity Enhancement with Aliases

The real power comes from configuring comprehensive aliases and helper functions. The [setup-kubectl-aliases.sh](https://github.com/nathanfox/nginx-dev-gateway/blob/develop/scripts/setup-kubectl-aliases.sh) script provides a complete set of productivity enhancements:

### Core Aliases

```bash
# Basic kubectl alias
alias k='kubectl'

# Resource retrieval
alias kgp='kubectl get pods'                    # Get pods in current namespace
alias kgpa='kubectl get pods --all-namespaces'  # Get all pods across namespaces
alias kgpw='kubectl get pods -o wide'           # Get pods with additional info
alias kgs='kubectl get svc'                     # Get services
alias kgd='kubectl get deployment'              # Get deployments
alias kgn='kubectl get nodes'                   # Get nodes
alias kgi='kubectl get ingress'                 # Get ingress resources

# Describe resources
alias kdp='kubectl describe pod'                # Describe pods
alias kds='kubectl describe svc'                # Describe services
alias kdd='kubectl describe deployment'         # Describe deployments

# Logs
alias kl='kubectl logs'                         # Get logs
alias klf='kubectl logs -f'                     # Follow logs
alias klp='kubectl logs -p'                     # Previous container logs

# Execution and management
alias ke='kubectl exec -it'                     # Execute interactive command
alias kaf='kubectl apply -f'                    # Apply configuration
alias kdf='kubectl delete -f'                   # Delete configuration
alias kdel='kubectl delete'                     # Delete resources
```

### Advanced Helper Functions

The setup script also includes powerful helper functions:

```bash
# Switch or show namespace
kn() {
    if [ -z "$1" ]; then
        kubectl config view --minify | grep namespace | cut -d" " -f6
    else
        kubectl config set-context --current --namespace="$1"
    fi
}

# Get logs by partial pod name
klg() {
    kubectl logs $(kubectl get pods | grep "$1" | head -1 | awk '{print $1}')
}

# Execute into pod by partial name
kex() {
    kubectl exec -it $(kubectl get pods | grep "$1" | head -1 | awk '{print $1}') -- /bin/bash
}

# Port forward to service
kpf() {
    kubectl port-forward svc/$1 ${2:-8080}:${3:-80}
}

# Watch pods
kwp() {
    watch -n 1 kubectl get pods ${1:+-n $1}
}

# Show resource usage
ktop() {
    kubectl top ${1:-pods}
}

# Restart deployment
krestart() {
    kubectl rollout restart deployment/$1
}

# Decode Kubernetes secret
kdecode() {
    kubectl get secret $1 -o jsonpath="{.data.$2}" | base64 -d
}
```


## AKS Integration

For Azure Kubernetes Service users, additional setup steps ensure smooth integration:

### Authentication with AKS

```bash
# Login to Azure
az login

# Get credentials for your AKS cluster
az aks get-credentials --resource-group <resource-group> --name <cluster-name>

# Optional: Install kubelogin for Azure AD authentication
wget https://github.com/Azure/kubelogin/releases/download/v0.0.31/kubelogin-linux-amd64.zip
unzip kubelogin-linux-amd64.zip
sudo mv bin/linux_amd64/kubelogin /usr/local/bin/
```

### Managing Multiple Clusters

```bash
# List available contexts
kubectl config get-contexts

# Switch between clusters
kubectl config use-context <context-name>

# Set default namespace for a context
kubectl config set-context --current --namespace=<namespace>
```

## Quick Setup with the Script (Bash/Ubuntu)

The fastest way to get all these productivity enhancements is to use the provided setup script (designed for Bash on Ubuntu/Linux):

```bash
# Download and run the setup script
curl -O https://raw.githubusercontent.com/nathanfox/nginx-dev-gateway/develop/scripts/setup-kubectl-aliases.sh
chmod +x setup-kubectl-aliases.sh
./setup-kubectl-aliases.sh

# Source your shell configuration
source ~/.bashrc
```

This script will:
1. Add all kubectl aliases to your ~/.bash_aliases file
2. Set up helper functions for common operations
3. Create a backup of your existing configuration

**Note:** For Zsh, Fish, or other shells, you'll need to manually adapt the aliases and functions to your shell's syntax. The alias definitions shown above can be added to your shell's configuration file (e.g., `~/.zshrc` for Zsh) with appropriate syntax modifications.

## Common Workflows

### Daily Development Tasks

```bash
# Quick namespace switch
kn dev-yourname

# Watch pods in your namespace
kwp

# Get logs from a pod (partial name match)
klg api-server

# Execute into a pod
kex frontend

# Port forward to a service
kpf my-service 8080:80

# Apply configuration changes
kaf deployment.yaml

# Restart a deployment
krestart my-app
```

### Debugging and Troubleshooting

```bash
# Get detailed pod information
kdp problematic-pod

# Check events in namespace
kev

# View resource usage
ktop

# Decode a secret value
kdecode my-secret password

# Get pod logs from previous container
klp crashed-pod
```

## Additional Resources

For complete documentation and updates:
- [Full kubectl setup guide](https://github.com/nathanfox/nginx-dev-gateway/blob/develop/docs/kubectl-setup.md)
- [Setup script with all aliases](https://github.com/nathanfox/nginx-dev-gateway/blob/develop/scripts/setup-kubectl-aliases.sh)
- [nginx-dev-gateway](https://github.com/nathanfox/nginx-dev-gateway) for Kubernetes development workflows

## Why This Matters

Investing time in proper kubectl setup pays dividends through faster command execution, fewer typos, and reduced context-switching overhead during daily development work.