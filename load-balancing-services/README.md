# Set up load balancing requests to Kubernetes services with Ingress NGINX

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

## Setup Kubernetes cluster
Follow [Create a HA Kubernetes cluster with kubeadm, keepalived, and haproxy by Ansible](../create-ha-cluster) to set up a HA Kubernetes cluster.

## Install NGINX ingress controller
```shell
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.1.1/deploy/static/provider/baremetal/deploy.yaml
kubectl -n ingress-nginx  get pods
```

Patch `ingress-nginx-controller` service to set `nodePort` to `30100` and `30101`
```shell
kubectl -n ingress-nginx patch svc ingress-nginx-controller --patch \
  '{"spec": { "type": "NodePort", "ports": [ { "port": 80, "nodePort": 30100 }, { "port": 443, "nodePort": 30101 } ] } }'
kubectl -n ingress-nginx get svc
```

Patch `ingress-nginx-controller` configmap to for [use-proxy-protocol](https://kubernetes.github.io/ingress-nginx/user-guide/nginx-configuration/configmap/#use-proxy-protocol)
```shell
kubectl -n ingress-nginx patch configmap ingress-nginx-controller --patch-file patch-configmap.yaml
```

## Configure load balancing requests to ingress-nginx service
See task `Configure haproxy` in [lb_playbook.yml](../create-ha-cluster/playbook/lb_playbook.yml)

## Verify by deploying a nginx into Kubernetes cluster
Source: https://kubernetes.io/docs/concepts/services-networking/connect-applications-service/

Deploy
```shell
kubectl apply -f my-nginx-deploy.yaml
kubectl get pods -l run=my-nginx
```

Expose service
```shell
kubectl apply -f my-nginx-svc.yaml
kubectl get svc my-nginx
```

[nip.io](https://nip.io/) allows us to map any IP Address to a hostname using the defined formats, then we do not need to edit `etc/hosts` file 
```shell
kubectl apply -f my-nginx-ingress.yaml
```

Open on browser [http://172.16.0.16.nip.io](http://172.16.0.16.nip.io), or
```shell
open http://172.16.0.16.nip.io
```

