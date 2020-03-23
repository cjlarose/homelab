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
https://raw.githubusercontent.com/cjlarose/media-server/master/cloud-init.yaml
```

The VM will install `k3os` to disk and restart.

I'm using [UEFI "Default Boot Behavior"][uefi-fix] in byhve to boot the VM, so I'll fix the boot partition now.

[uefi-fix]: https://www.ixsystems.com/community/threads/howto-how-to-boot-linux-vms-using-uefi.54039/

```sh
sudo su -
mkdir /mnt
mount /dev/vda1 /mnt
mkdir /mnt/EFI/BOOT
cp /mnt/EFI/alpine/grubx64.efi /mnt/EFI/BOOT/bootx64.efi
```

Stop the VM. Remove the installation media from the VM. Start it back up.

Now, let's ensure we can connect to the VM removely via `kubectl`.

```sh
K3OS_IP=192.168.50.143
scp rancher@"$K3OS_IP":/etc/rancher/k3s/k3s.yaml ~/.kube/k3os.yaml
ln -sfn ~/.kube/k3os.yaml ~/.kube/config
sed -i .bak 's/127.0.0.1/'"$K3OS_IP"'/g' ~/.kube/k3os.yaml
```

Now, execute `kubectl version` to make sure you can connect. All further instructions are to be executed remotely.

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
