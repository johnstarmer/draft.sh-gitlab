# draft.sh-gitlab
Setting up draft.sh with a gitlab repository
### Clone the lab repository to your computer
```
git clone https://github.com/kumulustech/draft.sh-gitlab draft
```
### Deploy a Kubernetes cluster
Note: Gitlab recommends a Kubernetes cluster, version 1.8 or higher. 6vCPU and 16GB of RAM.

To start you will want to install Minikube and kubectl following the instructions here: 
https://kubernetes.io/docs/tasks/tools/install-minikube/

then launch minikube with 8 GB RAM
```
minikube start --memory 8192
```
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
### Install Gitlab 

#WIP

First we will install the core Gitlab components. You will need to specify:
```global.hosts.domain```: the base domain of the wildcard host entry. (e.g. kumul.us for a wild card entry of ```*.kumul.us```.
```global.hosts.externalIP```: the external IP that the wildcard DNS resolves to.
```certmanager-issuer.email```: The email address to use when requesting new SSL certificates from Let's Encrypt.
First run
```
helm repo add gitlab https://charts.gitlab.io/
```
```
helm update
```
Before running the following, replace the details in {brackets} with your data.
```
helm upgrade --install gitlab gitlab/gitlab \
  --timeout 600 \
  --set global.hosts.domain={example.local} \
  --set global.hosts.externalIP={10.10.10.10} \
  --set certmanager-issuer.email={me@example.local}
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
### Using Draft - Draft Create
Switch to the folder containing the example application code.
```
cd example-app
```
Now we'll have draft create a helm chart, Docker file and draft.toml file for our application with 
```
draft create
```
Verify the new files were created
```
ls -a
```
Take a look at the draft.toml file that basic configuration details about our app
```
cat draft.toml
```
Other files that were created by Draft are

```.draftignore``` - specifies files for Draft to ignore (with a similar syntax to .helmignore)

```.dockerignore``` - specifies files for Docker to ignore

```.draft-tasks.toml``` - allows you to specify tasks to run before or after running ```draft up``` or cleanup tasks to run after ```draft delete```

### Deploy your application to Kubernetes
Finally, it's time to run 
```
draft up
```
If you get success messages on both the Docker build process and application release process, your application should be good. If not, run the "inspect logs" command provided in the launch output. 
Verify your application pod is running with 
```
kubectl get pods
```
### Interact with your application
```
draft connect
```
Note the {LOCALHOST_PORT} your application has been assigned and in a new terminal window run:
```curl localhost:{LOCALHOST_PORT}```
You should get the expected ```"Hello, World!"``` output.
You can now ```CTRL-C``` to cancel the ```draft connect``` session.

### Update your application
Let's change the application to output "Hello, Draft!" instead of "Hello, World!"

```
cat <<EOF > app.py
from flask import Flask

app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello, Draft!\n"

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=8080)
EOF
```
### Draft Up(grade)
This time when we call ```draft up```, Draft recognizes the Helm release already exists and will run a ```helm upgrade``` rather than a new install. This time, let's add the --auto-connect flag to our ```draft up``` command to automatically connect after the upgrade completes.
```
draft up --auto-connect
```
### Cleanup with Draft Delete
If you are done running the application, you can now run
```
draft delete
```
to delete the application from your Kubernetes cluster. Check on the application's status with:
```
kubectl get pods
```

You will initially see the pod status as "TERMINATING" and finally get a "No resources found" message when the deletion is complete.

Note: ```draft delete``` does _not_ remove images created for the deployment from your Docker registry.
