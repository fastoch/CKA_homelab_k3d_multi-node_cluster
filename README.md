# Introduction

The goal of this tutorial is to set up a homelab on a Fedora 43 machine for preparing the **CKA exam**.  

## Learning Resources

- K3D Kubernetes Tutorial: Local Clusters in Seconds - https://www.youtube.com/watch?v=ErhVmAEOUBM
- https://www.udemy.com/course/certified-kubernetes-administrator-with-practice-tests/
- https://kodekloud.com/
- https://killercoda.com

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
- verify with `k3d --version`, notice that the previous command also installed k3s.

## 3. Install kubectl

kubectl is the Kubernetes command-line tool.  
```bash
sudo dnf install kubectl
```
This will actually install the latest `kubernetes1.xx-client` package. Current version being 1.34.

---

# Create a cluster with one command

```bash
k3d cluster create mycluster
```

This command creates a k3d cluster, but it actually does multiple things:
  - it creates a docker network
  - it spins up a server node (or control plane)
  - it sets up a load balancer
  - it configures our kubeconfig file to connect kubectl to this cluster

With only one command, K3d has configured for us a fully functional k3s cluster in a matter of seconds.  

# Default pods in a K3d cluster

The above command has created a single-node cluster which contains 7 pods:
- coredns: handles DNS resolution within the cluster
- helm-install-traefik-crd & helm-install-traefik: required to install the Traefik ingress controller 
- local-path-provisioner: so we can provisoin persistent volumes
- metrics-server: provides container resource metrics like CPU and memory usage 
- svclb-traefik: provides external IPs for load balancers when we create them
- traefik: the Traefik ingress controller itself

## Playing with our first cluster

- We can run a test deployment: `kubectl create deployment nginx --image=nginx`
- Then, we can see the corresponding container creation by running `kubectl get pods`
- And we can expose that service on port 80: `kubectl expose deployment nginx --port=80 --type=ClusterIP`
- We can now see our newly created service by running `kubectl get svc`

# Essential commands for managing k3d

We've created a cluster and deployed our first application to it. Now, let's learn some useful k3d commands:
- to list all clusters: `k3d cluster list`
- to stop a cluster without deleting it: `k3d cluster stop mycluster`
- to start a cluster: `k3d cluster start mycluster`
- to delete a cluster: `k3d cluster delete mycluster`

# Creating a multi-node k3d cluster

To create a multi-node cluster with 1 control plane and 2 worker nodes:
```bash
k3d cluster create multinode-cluster --servers 1 --agents 2
```
**Reminder**:
- server node = control plane
- agent node = worker node

## Create a cluster from a config file

```bash
k3d cluster create --config my-cluster-config.yaml
```

You can then check that your config was properly applied with:
```bash
kubectl get nodes
docker ps
```

## Adding or removing workers

To add a worker node to an existing cluster: 
```bash
k3d node create new-worker --cluster multinode-cluster --role agent
```

To remove a specific worker node from a cluster:
```bash
kubectl get nodes
k3d node delete k3d-multinode-cluster-agent-1
```
