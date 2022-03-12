# Setup a multi-master Kubernetes cluster with kubeadm

This will set up a new multi-master Kubernetes cluster that has
* 3 masters, each master has 2 CPUs and 2 GBs RAM
* 3 workers, each worker has 2 CPUs and 2 GBs RAM
* 2 load balancers, each load balancer has 1 CPU and 1 GB RAM

![setup.png](setup.png?raw=true "setup.png")

## Prerequisites
Before you want to try this on your local, here are requirements
* Your computer must have at least 14 CPUs, 14 GBs RAM
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
cd create-ha-cluster
vagrant up
```

Let's ssh into the first master VM
```
ssh ci@172.16.1.11
```

## Setup load balancer
```
ansible-playbook -i playbook/lb_inventory.yml playbook/lb_playbook.yml --extra-vars "cluster_vip=172.16.0.16"
```

Let's ssh into the load balancer VM by the VIP, and check haproxy service status
```
ssh ci@172.16.0.16
systemctl status haproxy
```

## Setup Kubernetes cluster
```
ansible-playbook -i playbook/cluster_inventory.yml playbook/cluster_playbook.yml --extra-vars "cluster_vip=172.16.0.16"
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
