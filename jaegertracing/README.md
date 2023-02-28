# Distributed Tracing Platform
Jaeger: open source, end-to-end distributed tracing

## Prerequisites
* A k8s cluster
* [Helm](https://helm.sh/)

## Setup Kubernetes cluster
Follow * [Load balancing requests to Kubernetes services with external haproxy and NGINX Ingress Controller](../load-balancing-services)

## Install cert-manager and trust-manager
```shell
helm repo add jetstack https://charts.jetstack.io --force-update
helm upgrade -i -n cert-system cert-manager jetstack/cert-manager --set installCRDs=true --wait --create-namespace
helm upgrade -i -n cert-system trust-manager jetstack/trust-manager --wait --set app.trust.namespace=cert-system
```

```shell
NAME              	NAMESPACE         	REVISION	UPDATED                             	STATUS  	CHART                                	APP VERSION
cert-manager      	cert-system       	1       	2023-02-26 11:18:08.553011 +0700 +07	deployed	cert-manager-v1.11.0                 	v1.11.0
trust-manager     	cert-system       	1       	2023-02-26 11:19:08.579392 +0700 +07	deployed	trust-manager-v0.4.0                 	v0.4.0
```

## Create a Bundle using the default CAs
```shell
kubectl apply -f trust-bundle.yml
kubectl get bundle trust-bundle
kubectl get configmap trust-bundle -o "jsonpath={.data['trust-bundle\.pem']}"
```

## Create cluster issuer
```shell
kubectl apply -f cluster-issuer.yml
```

## Install Jaeger Operator
```shell
kubectl create ns observability-system
kubectl apply -f jaeger-operator-v1.42.0.yml
```

## Deploy the AllInOne image
Replace `172.16.0.16` with IP from your cluster.
```shell
kubectl apply -f simplest.yaml
```

Open on browser [http://jaegerui.172.16.0.16.nip.io/](http://jaegerui.172.16.0.16.nip.io/).

## Deploy Rides on Demand
https://github.com/jaegertracing/jaeger/tree/main/examples/hotrod

Replace `172.16.0.16` with IP from your cluster.
```shell
kubectl apply -f example-hotrod.yaml
```

Open on browser [http://example-hotrod.172.16.0.16.nip.io/](http://example-hotrod.172.16.0.16.nip.io/).

