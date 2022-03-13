# Set up a High Availability etcd Cluster with kubeadm

This will set up a highly available etcd cluster, and a single-master Kubernetes cluster using kubeadm
* 3 etcd nodes, each etcd has 2 CPUs and 2 GBs RAM
* 1 master has 2 CPUs and 2 GBs RAM
* 1 worker has 2 CPUs and 2 GBs RAM

![setup.png](setup.png?raw=true "setup.png")

## Prerequisites
Before you want to try this on your local, here are requirements
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

## Generate SSH key pair for CI user
```
cd ~
ssh-keygen -t rsa
```

Remember to enter `ci` as suffix in they key filename: `id_rsa_ci`

## Provision VMs
```
cd setup-ha-etcd-cluster
vagrant up
```

Check if we can ssh to the master VM, be aware that we use `~/.ssh/id_rsa_ci` key to ssh
```
ssh -i ~/.ssh/id_rsa_ci ci@172.16.1.11
```

## Setup etcd cluster
This playbook can be run in parallel with load balancer playbook
```
ansible-playbook -i playbook/etcd_cluster_inventory.yml playbook/etcd_cluster_playbook.yml
```

Check etcd cluster health
```
ssh -i ~/.ssh/id_rsa_ci ci@172.16.3.11
export ETCD_TAG=3.5.1-0
export HOST0=172.16.3.11
docker run --rm -it \
--net host \
-v /etc/kubernetes:/etc/kubernetes k8s.gcr.io/etcd:${ETCD_TAG} etcdctl \
--cert /etc/kubernetes/pki/etcd/peer.crt \
--key /etc/kubernetes/pki/etcd/peer.key \
--cacert /etc/kubernetes/pki/etcd/ca.crt \
--endpoints https://${HOST0}:2379 endpoint health --cluster
```

## Setup Kubernetes cluster
```
ansible-playbook -i playbook/k8s_cluster_inventory.yml playbook/k8s_cluster_playbook.yml
```

Download kubeconfig to your local, and verify the Kubernetes cluster
```
rm -rf ~/.kube-local
mkdir ~/.kube-local
scp -i ~/.ssh/id_rsa_ci ci@172.16.1.11:/home/ci/.kube/config ~/.kube-local/config
export KUBECONFIG=$HOME/.kube-local/config && kubectl get nodes
```
