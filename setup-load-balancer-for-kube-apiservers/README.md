# Setup load balancer for kuber-apiservers

This will set up a new multi-master Kubernetes cluster that has
* 3 masters, each master has 2 CPUs and 2 GBs RAM
* 3 workers, each worker has 2 CPUs and 2 GBs RAM
* 1 load balancers, each load balancer has 1 CPU and 1 GB RAM

![setup.png](setup.png?raw=true "setup.png")

## Prerequisites
Before you want to try this on your local, here are requirements
* Your computer must have at least 13 CPUs, 13 GBs RAM
* A public key in .ssh/id_rsa.pub in your home directory
* Directly download [Virtualbox](https://www.virtualbox.org/) and install, or use homebrew
    ```
    brew install --cask virtualbox
    ```
* Vagrant
    ```
    brew install --cask vagrant
    ```
* [(Optional) Vagrant Manager](http://vagrantmanager.com/)
    ```
    brew install --cask vagrant-manager
    ```

## Provision VMs
```
cd setup-load-balancer-for-kube-apiservers
vagrant up
```

## Setup load balancer
```
ansible-playbook -i playbook/load_balancer_inventory.yml playbook/load_balancer_playbook.yml
```

Let's ssh into the load balancer VM, and check haproxy service status
```
ssh ci@172.16.0.11
systemctl status haproxy
```

## Setup Kubernetes cluster
```
ansible-playbook -i playbook/cluster_inventory.yml playbook/cluster_playbook.yml
```

Let's ssh into the first master VM, and verify the cluster
```
ssh ci@172.16.1.11
kubectl get nodes
```

Download kubeconfig to  your local
```
mkdir ~/.kube-local
scp ci@172.16.1.11:/home/ci/.kube/config ~/.kube-local/config
export KUBECONFIG=$HOME/.kube-local/config && kubectl get nodes
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

