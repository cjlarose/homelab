# Personal media server

## Installation

Install `helm` and `fluxctl`

```
brew install helm fluxctl
```

Create a new VM running [`k3os`][k3os]. Install it `sudo k3os install`, copy the kubeconfig file onto your client machine (replace the IP address), and ensure you can connect to it via `kubectl`.

[k3os]: https://github.com/rancher/k3os

Install `flux` and `helm-operator` onto the k3s cluster.

```sh
kubectl create namespace flux

helm repo add fluxcd https://charts.fluxcd.io

helm upgrade -i flux fluxcd/flux \
--set git.url=git@github.com:cjlarose/media-server \
--set syncGarbageCollection.enabled=true \
--namespace flux

helm upgrade -i helm-operator fluxcd/helm-operator \
--set git.ssh.secretName=flux-git-deploy \
--set helm.versions=v3 \
--namespace flux
```

Wait until the container come up. Then, use `fluxctl` to get the SSH public key for deployments

```sh
fluxctl identity --k8s-fwd-ns flux
```

Add that as a "deploy key" for this repository. Allow write access.

## Deploying changes

Just push to this repository. `flux` will update resources in the cluster automatically.

If you're impatient, trigger changes manually with

```sh
fluxctl --k8s-fwd-ns=flux sync
```
