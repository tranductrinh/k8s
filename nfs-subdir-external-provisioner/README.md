# NFS Subdir External Provisioner

![Setup.png](Setup.png?raw=true "Setup.png")

## Prerequisites
* [Virtualbox](https://www.virtualbox.org/)
* [Vagrant](https://www.vagrantup.com/)
* [Ansible](https://www.ansible.com/)
* [Helm](https://helm.sh/)

## Setup Kubernetes cluster
Follow [Create a single-master Kubernetes cluster with kubeadm by Ansible](../create-single-master-cluster) to set up a Kubernetes cluster.

## Setup NFS Server
### Provision NFS VM
```
cd nfs-subdir-external-provisioner
vagrant up
```

### Manual setup NFS Server
Source: https://www.tecmint.com/install-nfs-server-on-ubuntu/

#### Install NFS Kernel Server
```
ssh ci@172.16.4.11
sudo apt update
sudo apt install nfs-kernel-server
```

#### Create an NFS Export Directory
```
sudo mkdir -p /mnt/nfs_share
sudo chown -R nobody:nogroup /mnt/nfs_share/
sudo chmod 777 /mnt/nfs_share/
```

If you run `sudo ls /mnt/nfs_share/`, it should return empty directory.

#### Grant NFS Share Access to Client Systems
```shell
sudo vi /etc/exports
```

Add following lines to grant access from workers
```text
/mnt/nfs_share 172.16.0.0/16(rw,insecure,sync,no_subtree_check)
```

#### Export the NFS Share Directory
```shell
sudo exportfs -a
sudo systemctl restart nfs-kernel-server
```

#### Allow NFS Access through the Firewall
Check firewall status
```shell
sudo ufw status
```

If your firewall is enabled, then allow workers to connect to NFS port.
```shell
sudo ufw allow from 172.16.0.0/16 to any port nfs
sudo ufw status numbered
```

#### Install the NFS-Common Package on Workers
Worker 1
```
ssh ci@172.16.2.11
sudo apt update
sudo apt install nfs-common
```

Worker 2
```
ssh ci@172.16.2.12
sudo apt update
sudo apt install nfs-common
```

Worker 3
```
ssh ci@172.16.2.13
sudo apt update
sudo apt install nfs-common
```

## Install NFS Subdir External Provisioner with Helm
Source: https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner

Install
```shell
helm repo add nfs-subdir-external-provisioner https://kubernetes-sigs.github.io/nfs-subdir-external-provisioner/
helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
    --set nfs.server=172.16.4.11 \
    --set nfs.path=/mnt/nfs_share
```

If you want to uninstall `nfs-subdir-external-provisioner`, run `helm uninstall nfs-subdir-external-provisioner`.

List StorageClass
```shell
kubectl get storageclass
```
There is `nfs-client` StorageClass created.

## Verify
```shell
kubectl apply -f my-nginx-pvc.yaml
kubectl get pvc
```

`my-nginx-pvc` PersistentVolumeClaim is in `Bound` status. You should note the `VOLUME` hash.

```shell
kubectl apply -f my-nginx-deploy.yaml
kubectl get pod
```

SSH to NFS Server, and check shared directory.
```shell
ssh ci@172.16.4.11
cd /mnt/nfs_share
ls -alh
```

There is a directory inside `/mnt/nfs_share`, it has the `VOLUME` hash that you see above.

```shell
cd _VOLUME_HASH_DIRECTORY_
echo "Hello from a PV on NFS Server" >> index.html
```

Create service
```shell
kubectl apply -f my-nginx-svc.yaml
kubectl get svc
```

Open `http://172.16.1.11:30080/` on browser.
