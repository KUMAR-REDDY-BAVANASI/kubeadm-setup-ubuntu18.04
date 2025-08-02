How to Install Kubernetes on Ubuntu 18.04
=========================================

Kubernetes is an open source platform for managing container technologies such as Docker

Prerequisites
=============
2 or more Linux servers running Ubuntu 18.04

Access to a user account on each system with sudo or root privileges

The apt package manager, included by default

Command-line/terminal window (Ctrl–Alt–T)


Steps to Install Kubernetes on Ubuntu
=====================================
Step 1: Install Docker
----------------------
`sudo apt-get install docker.io -y

sudo systemctl enable docker

sudo systemctl start docker

sudo apt update`

Install Kubernetes
Step 2: Add Kubernetes Signing Key
--------------------------------------
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add


Step 3: Add Software Repositories:
----------------------------------
sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"


Step 4: Kubernetes Installation Tools
---------------------------------------
sudo apt-get install kubeadm kubelet kubectl -y

sudo apt-mark hold kubeadm kubelet kubectl

[OR]

Specific version:
-----------------
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add - && \
  echo "deb http://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list && \
  sudo apt-get update -q && \
  sudo apt-get install -qy kubelet=1.21.0-00 kubectl=1.21.0-00 kubeadm=1.21.0-00



Kubernetes Deployment
Step 5: Begin Kubernetes Deployment
Start by disabling the swap memory on each server
---------------------------------------------------
sudo swapoff -a

sudo sed -i '/ swap / s/^/#/' /etc/fstab


Step 6: Assign Unique Hostname for Each Server Node 
----------------------------------------------------
sudo hostnamectl set-hostname master-node

sudo hostnamectl set-hostname worker-node01

********************Repeat 1-6 steps each server********************

Step 7: Initialize Kubernetes on Master Node
--------------------------------------------
sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --ignore-preflight-errors all


---------------------------------------------------------------------------------------------------------
If any error like kubelet unhealthy:  [SOLVED]
----------------------------------------------
cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2",
  "storage-opts": [
    "overlay2.override_kernel_check=true"
  ]
}
EOF

mkdir -p /etc/systemd/system/docker.service.d

systemctl daemon-reload

systemctl restart docker

kubeadm reset 

---------------------------------------------------------------------------------------------------------


sudo kubeadm init --pod-network-cidr=10.244.0.0/16 --ignore-preflight-errors all

mkdir -p $HOME/.kube

sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config

sudo chown $(id -u):$(id -g) $HOME/.kube/config


Step 8: Deploy Pod Network to Cluster:
---------------------------------------
*******************[CALICO NETWORK PLUGIN]****************************************

kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml


*******************[FLANNEL NETWORK PLUGIN]****************************************

sudo kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml


kubectl get pods --all-namespaces



Step 9: Join Worker Node to Cluster
------------------------------------

kubeadm token create --print-join-command      # in master node

In worker node you have to use kubeadm join command like below:
---------------------------------------------------------------

kubeadm join --discovery-token abcdef.1234567890abcdef --discovery-token-ca-cert-hash sha256:1234..cdef 1.2.3.4:6443


Step 10: Switch to master node and enter
----------------------------------------

kubectl get nodes




===========================================

Note:
-----
To destroy kubeadm setup:
-------------------------

kubeadm reset -y

sudo apt-get purge kubeadm kubectl kubelet kubernetes-cni kube* -y

sudo apt-get autoremove -y

sudo rm -rf ~/.kube
