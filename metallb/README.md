# MetalLB

## Prerequisites
* [Virtualbox](https://www.virtualbox.org/)
* [Vagrant](https://www.vagrantup.com/)
* [Ansible](https://www.ansible.com/)
* [Helm](https://helm.sh/)

## Setup Kubernetes cluster
If you don't have a running cluster, follow [Create a single-master Kubernetes cluster with kubeadm by Ansible](../create-single-master-cluster) to set up a Kubernetes cluster.

## Deploy a nginx into Kubernetes cluster
Source: https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/

Deploy
```shell
kubectl apply -f my-nginx-deploy.yaml
kubectl get pods -l run=my-nginx
```

Expose service type `LoadBalancer`
```shell
kubectl apply -f my-nginx-svc.yaml
kubectl get svc my-nginx
```

You will see that `EXTERNAL-IP` is always `pending`.

## Install MetalLB with Helm
Source: https://metallb.universe.tf/installation/#installation-with-helm

Install
```shell
helm repo add metallb https://metallb.github.io/metallb
helm install metallb metallb/metallb -f values.yaml
```

If you want to uninstall MetalLB, run `helm uninstall metallb`.

List all releases
```shell
helm list --all -A
```

## Verify
Get services again
```shell
kubectl get svc my-nginx
```

You will see that `EXTERNAL-IP` is assigned with an IP.

In order to be able to connect to the IP, you will need to be inside VM networks. SSH into the first master VM, then try to `curl` to the `EXTERNAL-IP` that you see above.
```shell
ssh ci@172.16.1.11
```

If you would like to connect to the IP from your host, you can set up `Bridged Networking` on VirtualBox VMs, which is not covered here!
