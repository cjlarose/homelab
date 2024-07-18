# Personal media server

## Installation

## Sealed Secrets

```sh
kubectl apply -f https://github.com/bitnami-labs/sealed-secrets/releases/download/v0.18.0/controller.yaml
```

## cert-manager

```sh
kubectl apply -f https://github.com/cert-manager/cert-manager/releases/download/v1.11.0/cert-manager.yaml
kubectl -n cert-manager patch deployment cert-manager --patch-file photos/cert-manager/deployment-patch.yaml
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

## PhotoPrism

```sh
kubectl apply -f photos/mariadb/secret.yaml
kubectl apply -f photos/mariadb/deployment.yaml
kubectl apply -f photos/mariadb/service.yaml
kubectl apply -f photos/photoprism/ingress.yaml
kubectl apply -f photos/photoprism/secret.yaml
kubectl apply -f photos/photoprism/stateful-set.yaml
kubectl apply -f photos/photoprism/service.yaml
```

## restic

```sh
kubectl apply -f photos/restic/backblaze-application-key-secret.yaml
kubectl apply -f photos/restic/restic-secret.yaml
kubectl apply -f photos/restic/backup-cronjob.yaml
```

To get restic logs

```sh
kubectl get jobs.batch
kubectl logs job/restic-backup-photoprism-28350720
```
