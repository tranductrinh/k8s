# Create a single-master Kubernetes cluster with kubeadm

This will set up a new single-master Kubernetes cluster that has
* 1 master, has 2 CPUs and 2 GBs RAM
* 3 workers, each worker has 2 CPUs and 2 GBs RAM

![setup.png](setup.png?raw=true "setup.png")

## Prerequisites
Before you want to try this on your local, here are requirements
* A public key in .ssh/id_rsa.pub in your home directory
* Directly download [Virtualbox](https://www.virtualbox.org/) and install, or use homebrew
    ```
    brew install --cask virtualbox
    ```
* Vagrant
    ```
    brew install --cask vagrant
    ```

## Provision VMs
```
cd create-single-master-cluster
vagrant up
```

## Create Kubernetes cluster
```
ansible-playbook -i playbook/inventory.yml playbook/playbook.yml
```

Let's ssh into the master VM, and verify the cluster
```
ssh ci@172.16.1.11
kubectl get nodes
```

Download kubeconfig to  your local
```
mkdir ~/.kube-local
scp ci@172.16.1.11:/home/ci/.kube/config ~/.kube-local/config
export KUBECONFIG=~/.kube-local/config && kubectl get nodes
```

## Creating and exploring an nginx deployment
Source: https://kubernetes.io/docs/tasks/run-application/run-stateless-application-deployment/

### Create a Deployment
```
kubectl apply -f https://k8s.io/examples/application/deployment.yaml
```

### Display information about the Deployment
```
kubectl describe deployment nginx-deployment
```

### List the Pods created by the deployment
```
kubectl get pods -l app=nginx
```
