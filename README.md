# Demo Argo Workflow & ArgoCD App

## Prerequisites

### Install WSL (Windows Only)
Install WSL https://learn.microsoft.com/en-us/windows/wsl/install

### Install CLI tools
install argo, argocd, kubernetes-helm, kubectl, k9s, kubens, kubectx, docker, neovim, jq, mktemp, yq (yamlq), argocd-autopilot

Via NixOS:

    curl -L https://nixos.org/nix/install | sh
    nix-shell --packages argo argocd kubernetes-helm kubectl k9s kubectx docker neovim jq mktemp yq argocd-autopilot

To stop using installed packages, just type `exit` command and your Nix session will stop.
Search for more packages on https://search.nixos.org to try them out.

To free up Nix storage cache run:

    nix-collect-garbage

### Setup local k8s cluster
- Install Rancher Desktop from https://rancherdesktop.io/
- Enable Kubernetes cluster feature
  ![Rancher Desktop Enable K8s](/docs/RancherDesktopEnableK8s.png)

Windows Only:
- Forward cluster to WSL via: Preferences -> WSL -> Integrations -> Ubuntu
  ![Rancher Desktop Forward K8s](/docs/RancherDesktopForwardK8s.png)

### Activate Rancher K8s Cluster Context
To work with local Rancher Desktop K8s cluster please execute following command:

    kubectx rancher-desktop

### Install Argo Workflows into the cluster

    kubectl create namespace argo
    kubectl apply -n argo -f https://github.com/argoproj/argo-workflows/releases/download/v3.5.5/quick-start-minimal.yaml

### Install Argo Events
https://argoproj.github.io/argo-events/quick_start/

    kubectl create namespace argo-events
    kubectl apply -f https://raw.githubusercontent.com/argoproj/argo-events/stable/manifests/install.yaml

### Install Argo CD into the cluster
https://argo-cd.readthedocs.io/en/stable/getting_started/

    kubectl create namespace argocd
    kubectl apply -n argocd -f https://raw.githubusercontent.com/argoproj/argo-cd/stable/manifests/install.yaml

## Setup Docker Hub credentials

    REGISTRY_SERVER='https://index.docker.io/v1/'
    REGISTRY_USER='your-username'
    REGISTRY_PASSWORD='your-password'
    REGISTRY_EMAIL='your-email'

    temp_dir=$(mktemp -d)
    kubectl create secret docker-registry registry-creds \
    --docker-server=$REGISTRY_SERVER \
    --docker-username=$REGISTRY_USER \
    --docker-password=$REGISTRY_PASSWORD \
    --docker-email=$REGISTRY_EMAIL \
    --dry-run=client -o json | jq -r '.data.".dockerconfigjson"' | base64 --decode > $temp_dir/config.json

    kubectl create secret generic config.json --from-file=$temp_dir/config.json -n argo
    rm -rf $temp_dir

## Setup Github Credentials

    GIT_ACCESS_TOKEN='your-access-token'
    kubectl create secret generic git-credentials-secret --from-literal=.git-credentials="https://$GIT_ACCESS_TOKEN@github.com" -n argo


## Access Argo Workflow UI

    kubectl -n argo port-forward service/argo-server 2746:2746

In your browser open: https://localhost:2746

![Argo Workflow](/docs/ArgoWorkflow.png)

## Access ArgoCD UI

    kubectl port-forward svc/argocd-server -n argocd 8080:443

In your browser open: https://localhost:8080

![Argo CD](/docs/ArgoCD.png)

## Create ArgoCD app

    kubectl port-forward svc/argocd-server -n argocd 8080:443
    argocd login localhost:8080 
    argocd app create cicd-automation-demo --repo https://github.com/majoferenc/demo-cicd-automation-app.git  --dest-server https://kubernetes.default.svc --dest-namespace default  --path chart
