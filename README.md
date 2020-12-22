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
ssh master1 bash -xs < k8s-ubuntu-install<br>
ssh worker1 bash -xs < k8s-ubuntu-install

## generic installation procedure
- kubeadm init on <strong>master1</strong>
- kubeadm join on <strong>worker1</strong>

# UPGRADE
## when upgrading point releases
export K8S_VERSION_CURRENT=$(kubectl version --short | awk '/^Server/{print substr($3,2)}')<br>
export K8S_VERSION_LATEST=$(curl -sL https://dl.k8s.io/release/stable-${K8S_VERSION_CURRENT%.*}.txt)

## otherwise
export K8S_VERSION_MAJOR=1<br>
export K8S_VERSION_LATEST=$(curl -sL https://dl.k8s.io/release/stable-${K8S_VERSION_MAJOR}.txt)

### control plane first
kubectl drain master1 --ignore-daemonsets --delete-local-data<br>
ssh master1 sudo apt update

### when upgrading the OS
ssh master1 bash -xs < k8s-ubuntu-update-os

### otherwise
ssh master1 bash -xs < k8s-ubuntu-update-master<br>
kubectl uncordon master1<br>
sleep 60<br>
ssh master1 sudo docker image prune -af

### workload plane, workers one by one
kubectl drain worker1 --ignore-daemonsets --delete-local-data<br>
ssh worker1 sudo apt update

### when upgrading the OS
ssh worker1 bash -xs < k8s-ubuntu-update-os

### otherwise
ssh worker1 bash -xs < k8s-ubuntu-update-worker<br>
kubectl uncordon worker1<br>
sleep 60<br>
ssh worker1 sudo docker image prune -af
