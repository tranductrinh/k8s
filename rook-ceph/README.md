# Set up Rook Ceph on a bare-metal cluster

## Prerequisites
* [Virtualbox](https://www.virtualbox.org/)
* [Vagrant](https://www.vagrantup.com/)
* [Ansible](https://www.ansible.com/)
* [Helm](https://helm.sh/)

## Set up Kubernetes cluster
Bring up VMs: 1 master and 3 workers.
```shell
cd rook-ceph
vagrant up
```

SSH into `k8s-worker-1`, then list block devices.
```shell
ssh ci@172.16.2.11
lsblk
```

Provision a cluster
```shell
cd ../create-single-master-cluster
ansible-playbook -i playbook/inventory.yml playbook/playbook.yml
```

Download kubeconfig to  your local
```shell
mkdir ~/.kube-local
scp ci@172.16.1.11:/home/ci/.kube/config ~/.kube-local/config
export KUBECONFIG=~/.kube-local/config && kubectl get nodes
```

## Deploy Ceph Operator
```shell
helm repo add rook-release https://charts.rook.io/release
helm upgrade --install --create-namespace --namespace rook-ceph rook-ceph rook-release/rook-ceph
```

```shell
kubectl --namespace rook-ceph get pods -l "app=rook-ceph-operator"
```

## Deploy Ceph Cluster
```shell
helm repo add rook-release https://charts.rook.io/release
helm upgrade --install --create-namespace --namespace rook-ceph rook-ceph-cluster \
   --set operatorNamespace=rook-ceph rook-release/rook-ceph-cluster
```

```shell
kubectl --namespace rook-ceph get pods
```

Verify that the cluster is in a healthy state.
```shell
cd ../rook-ceph
kubectl create -f toolbox.yml
kubectl -n rook-ceph rollout status deploy/rook-ceph-tools
kubectl -n rook-ceph exec -it deploy/rook-ceph-tools -- bash
```

```shell
ceph status
```
