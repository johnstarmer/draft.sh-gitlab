# draft.sh-gitlab
Setting up draft.sh with a gitlab repository

### Deploy a Kubernetes cluster
Note: Gitlab recommends a Kubernetes cluster, version 1.8 or higher. 6vCPU and 16GB of RAM.

To start you will want to install Minikube and kubectl following the instructions here: 
https://kubernetes.io/docs/tasks/tools/install-minikube/

then launch minikube with 8 GB RAM

minikube start --memory 8192

If you want to set this as default on minikube, 
```
minikube config set memory 8192
```
You will need to delete and restart minikube to initialize this setting.

### Install and configure Helm
- Helm is a Kubernetes package manager. With it, you can install and manage Kubernetes manifests, aka "Helm charts", on your Kubernetes cluster.
- Download the Helm binary using [Homebrew](https://brew.sh/) via `brew install kubernetes-helm` or the [official releases page](https://github.com/kubernetes/helm/releases).
- Once you have Helm installed and Kubernetes running on your machine:
```
helm init --upgrade
```
Install RBAC rules:
```
kubectl create -f rbac.yaml
```
### Install Draft
Follow the draft install instructions here:
https://github.com/Azure/draft/blob/master/docs/quickstart.md

Verify Draft has been installed correctly:

```
draft version
```
Initialize Draft
```
draft init
```
### Setup your application repository
If you just want to try out Draft, the following will allow Draft to build images directly using Minikube's Docker daemon without needing to set up a remote/external container registry.
```
eval $(minikube docker-env)
```

