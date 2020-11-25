# Personal media server

## Installation

Install `helm` and `fluxctl`

```
brew install helm fluxctl
```

Create a new VM from the [`k3os`][k3os] installation ISO.

[k3os]: https://github.com/rancher/k3os

Log in as the `rancher` user and execute `sudo k3os install`. Select "Install to disk". Select "Config with cloud-init file". Use the path

```
https://raw.githubusercontent.com/cjlarose/media-server/master/cloud-init-server.yaml
```

The VM will install `k3os` to disk and restart.

Stop the VM. Remove the installation media from the VM. Start it back up.

Now, let's ensure we can connect to the VM removely via `kubectl`.

```sh
K3OS_IP=192.168.50.143
scp rancher@"$K3OS_IP":/etc/rancher/k3s/k3s.yaml ~/.kube/k3os.yaml
ln -sfn ~/.kube/k3os.yaml ~/.kube/config
sed -i .bak 's/127.0.0.1/'"$K3OS_IP"'/g' ~/.kube/k3os.yaml
```

Now, execute `kubectl version` to make sure you can connect. All further instructions are to be executed remotely.

Install Calico:

```sh
kubectl create -f https://docs.projectcalico.org/manifests/tigera-operator.yaml
cat <<EOF | kubectl create -f -
# This section includes base Calico installation configuration.
# For more information, see: https://docs.projectcalico.org/v3.15/reference/installation/api#operator.tigera.io/v1.Installation
apiVersion: operator.tigera.io/v1
kind: Installation
metadata:
  name: default
spec:
  # Configures Calico networking.
  calicoNetwork:
    # Note: The ipPools section cannot be modified post-install.
    ipPools:
    - blockSize: 26
      cidr: 10.42.0.0/16
      encapsulation: VXLANCrossSubnet
      natOutgoing: Enabled
      nodeSelector: all()
EOF
```

Apply Calico network policies

```sh
find ./calico -name '*.yaml' -exec calicoctl apply -f {} \;
```

Install MetalLB:

```sh
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.3/manifests/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.9.3/manifests/metallb.yaml
# On first install only
kubectl create secret generic -n metallb-system memberlist --from-literal=secretkey="$(openssl rand -base64 128)"
```

Configure MetalLB:

```
cat <<EOF | kubectl apply -f -
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 172.16.20.17/32
      - 172.16.20.20/32
EOF
```

Install flux and the helm operator:

```sh
helm repo add fluxcd https://charts.fluxcd.io

kubectl create namespace flux

helm upgrade -i flux-homelab fluxcd/flux \
--set git.url=git@github.com:cjlarose/homelab \
--set git.path="cert-manager\,configmaps\,namespaces\,releases\,secrets\,workloads" \
--set syncGarbageCollection.enabled=true \
--namespace flux

helm upgrade -i helm-operator fluxcd/helm-operator \
--set helm.versions=v3 \
--set allowNamespace=media \
--namespace media
```

Wait until the containers come up. Then, use `fluxctl` to get the SSH public key for deployments

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

To monitor flux logs

```sh
kubectl -n flux logs deployment/flux-homelab -f
```

To monitor helm operator logs

```sh
kubectl -n media logs deployment/helm-operator -f
```
