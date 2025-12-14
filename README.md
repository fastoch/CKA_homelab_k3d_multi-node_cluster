# Introduction

The goal of this tutorial is to set up a homelab on a Fedora 43 machine for preparing the **CKA exam**.  

## Useful resources

- https://kodekloud.com/
- https://killercoda.com
- https://www.youtube.com/watch?v=ErhVmAEOUBM

## What is K3s? 

K3s is a **lightweight**, easy-to-install, deploy, and manage **version** of stock **Kubernetes** (K8s).  
No, it is not a fork of Kubernetes, K3s is a certified Kubernetes distribution.

## What is K3d? 

K3d is a tool for running K3s single- and multi-node clusters in **Docker**.  
Coupling it with **kubectl** makes a fully functional homelab for preparing the CKA exam.  
This combo supports context switching between clusters to mimic exam scenarios.  

**k3d** can spin up conformant clusters in seconds via simple commands like `k3d cluster create cka-lab --servers 1 --agents 2`, covering CKA domains such as 
scheduling, RBAC (Role-Based Access Control), and storage without needing VMs.  
While **kubectl** handles all exam-required operations (e.g., kubectl apply -f, imperative edits, crictl inspections) on these clusters.  

## Limitations

k3d uses lightweight k3s, skipping full **kubeadm** bootstrapping or advanced **HA** (High-Availability) setups rarely tested in CKA.  

---

# Setting up my CKA homelab

## 1. Install Docker on Fedora 43

- check if docker is already installed: `docker version`
- if not, add the docker repo, install docker components, enable docker service, and add your user to the docker group:
```bash
sudo dnf config-manager addrepo --from-repofile=https://download.docker.com/linux/fedora/docker-ce.repo
sudo dnf install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin
sudo systemctl enable --now docker
sudo usermod -aG docker $USER
```
- Reboot
- Verify installation with `docker version`​

## 2. Install K3d

- Run the official install script for the latest version:
```bash
curl -s https://raw.githubusercontent.com/k3d-io/k3d/main/install.sh | bash
```
- verify with `k3d --version`, notice the previous command also installed k3s

## 3. Install kubectl

kubectl is the Kubernetes command-line tool.  
```bash
sudo dnf install kubectl
```
This will actually install the latest `kubernetes1.xx-client` package. Current version being 1.34.

## 4. Create a test cluster 

```bash
k3d cluster create mycluster
```
This command creates a k3d cluster, but it actually does multiple things:
  - it creates a docker network
  - 
