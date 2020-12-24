# scripts to deploy / upgrade Kubernetes lab clusters on Ubuntu

### I use Ubuntu (Server 20.04) in order to make use of up-to-date cluster and kernel packages.

# ASSUMPTIONS
- 2+ available role-dedicated lab nodes (ideally 1 master and 2+ workers) running Ubuntu (I use 4 (1+3) KVM guest domains created via libvirt, YMMV)
- SSH access to all nodes with pubkey auth
- sudo as root without password
- variables named <i>K8S_VERSION_*</i> sent and accepted via SSH
- one doesn't know Ansible (half-joking there, this is more of an educational, "quick 'n' dirty" effort)
- one knows there are other, possibly better ways

# INSTALLATION
```bash
ssh master1 bash -xs < k8s-ubuntu-install
ssh worker1 bash -xs < k8s-ubuntu-install
```

## generic installation procedure
- kubeadm init on <strong>master1</strong>
- kubeadm join on <strong>worker1</strong>

# UPGRADE
```bash
## when upgrading point releases
export K8S_VERSION_CURRENT=$(kubectl version --short | awk '/^Server/{print substr($3,2)}')
export K8S_VERSION_LATEST=$(curl -sL https://dl.k8s.io/release/stable-${K8S_VERSION_CURRENT%.*}.txt)

## otherwise
export K8S_VERSION_MAJOR=1
export K8S_VERSION_LATEST=$(curl -sL https://dl.k8s.io/release/stable-${K8S_VERSION_MAJOR}.txt)

### control plane first
kubectl drain master1 --ignore-daemonsets --delete-local-data
ssh master1 sudo apt update

### when upgrading the OS
ssh master1 bash -xs < k8s-ubuntu-update-os

### otherwise
ssh master1 bash -xs < k8s-ubuntu-update-master
kubectl uncordon master1
sleep 60
ssh master1 sudo docker image prune -af

### workload plane, workers one by one
kubectl drain worker1 --ignore-daemonsets --delete-local-data
ssh worker1 sudo apt update

### when upgrading the OS
ssh worker1 bash -xs < k8s-ubuntu-update-os

### otherwise
ssh worker1 bash -xs < k8s-ubuntu-update-worker
kubectl uncordon worker1
sleep 60
ssh worker1 sudo docker image prune -af
```
