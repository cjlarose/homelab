# Personal media server

## Installation

## Sealed Secrets

```sh
kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.18.0/controller.yaml
```

## cert-manager

```sh
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.11.0/cert-manager.yaml
kubectl apply -f photos/cert-manager/digital-ocean-credentials.yaml
kubectl apply -f photos/cert-manager/letsencrypt-prod-issuer.yaml
kubectl apply -f photos/cert-manager/toothyshouse-com-certificate.yaml
```

## ingress-nginx

```sh
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.7.0/deploy/static/provider/baremetal/deploy.yaml
kubectl -n ingress-nginx delete service ingress-nginx-controller
kubectl -n ingress-nginx patch deployment ingress-nginx-controller --patch-file photos/ingress-nginx/host-network.yaml
```
