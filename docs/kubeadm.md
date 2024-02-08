# KubeAdm 

## KubeADM Install Script - SingleNode Cluster

Guides
- Install KubeAdm on [Multi-Node](https://www.linuxtechi.com/install-kubernetes-on-ubuntu-22-04/) Cluster
- Install KubeAdm on [Single-Node](https://blog.radwell.codes/2022/07/single-node-kubernetes-cluster-via-kubeadm-on-ubuntu-22-04/)

Minimum Specs
- 4GB RAM + 2 CPU's + 20GB Free Space 

## Pre-requisites

1. Ensure the name of Host is consistent with the `/etc/hosts` file you will modify below
2. Default name is `k8s-control`

## Install General Dependencies
```bash
sudo nano /etc/hostname
# or
sudo hostnamectl set-hostname "k8s-cluster1"
```

## Disable Swap and Add Kernal Parameters

```bash
sudo swapoff -a
#comment last line out
sudo nano /etc/fstab 
```

```bash
# Write the lines overlay and br_netfilter to the .conf file
sudo tee /etc/modules-load.d/containerd.conf <<EOF
overlay
br_netfilter
EOF

modprobe overlay
modprobe br_netfilter

# Add kernal parameters
sudo tee /etc/sysctl.d/kubernetes.conf <<EOF
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
net.ipv4.ip_forward = 1
EOF

# Reload changes
sudo sysctl --system

```

## Install Containerd Runtime

`sudo apt install -y curl gnupg2 software-properties-common apt-transport-https ca-certificates`

### Enable Docker Repo and Install Containerd
```bash
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/docker.gpg

sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"

sudo apt update

sudo apt install -y containerd.io

# Start as cgroup w/ systemd

containerd config default | sudo tee /etc/containerd/config.toml >/dev/null 2>&1

sudo sed -i 's/SystemdCgroup \= false/SystemdCgroup \= true/g' /etc/containerd/config.toml

sudo systemctl restart containerd
sudo systemctl enable containerd
```

## Add Apt Repository for Kubernetes

```bash
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo gpg --dearmour -o /etc/apt/trusted.gpg.d/kubernetes-xenial.gpg

sudo apt-add-repository "deb http://apt.kubernetes.io/ kubernetes-xenial main"

# Install Kubectl, Kubeadm and Kubelet
sudo apt update
sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

## Initialize Single-Node Cluster
`sudo kubeadm init --pod-network-cidr=10.244.0.0/16`

Ensure you run the below commands
```bash
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

### Untaint Node for Single-Cluster Pod Deployments

```bash
kubectl taint nodes --all node-role.kubernetes.io/control-plane-
```

### Install Calico CNI Plugin
`kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.25.0/manifests/calico.yaml`

### Simple Nginx Pod Test
```bash
kubectl run nginx --image=nginx
kubectl expose pod nginx --port=80 --type=NodePort

# Alternate
kubectl create deployment nginx-app --image=nginx --replicas=2

kubectl expose deployment nginx-app --type=NodePort --port=80

curl http://192.168.1.230:<NodePort>

```

## Installing Add-Ons

### Install Helm
`curl https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 | bash`

### Install CSI Driver - OpenBS
```bash
helm repo add openebs https://openebs.github.io/charts

kubectl create namespace openebs

helm --namespace=openebs install openebs openebs/openebs

# Add bitnami repo to helm
helm repo add bitnami https://charts.bitnami.com/bitnami

helm install wordpress bitnami/wordpress --set=global.storageClass=openebs-hostpath
```


[Original Link](https://www.linkedin.com/pulse/how-easily-setup-multi-node-kubernetes-cluster-single-quattrocchi/)

# How to Easily Setup a Multi-node Kubernetes Cluster on a Single Machine with Terraform, libvirt and Fedora CoreOS

**Luigi Quattrocchi**  
_Senior Software Engineer at TIM_

Published on December 28, 2022

This article describes a simple procedure to setup a multi-node Kubernetes cluster for development/testing/learning purposes on a single Linux machine using Terraform, libvirt and Fedora CoreOS virtual machines.

## Requirements

On the target Linux machine the following software must be installed:

- **libvirt library**: On Fedora, the installation can be done using the following command:
  ```
  sudo dnf group install --with-optional virtualization
  ```
- **Terraform**: On Fedora, the installation can be done using these commands:
  ```
  sudo dnf install -y dnf-plugins-core
  sudo dnf config-manager --add-repo https://rpm.releases.hashicorp.com/fedora/hashicorp.repo
  sudo dnf -y install terraform
  ```
- **Butane**: On Fedora, the installation can be done using this command:
  ```
  sudo dnf install -y butane
  ```

Clone the GitHub repository:
```
git clone https://github.com/luigiqtt/dev-multinode-k8s.git
```

## Configuration

### Terraform Variables

Modify the `k8s.auto.tfvars` and `k8s.secret.auto.tfvars` files according to your requirements.

### Butane Files

Modify the Butane YAML-formatted files for each node in the `config` directory as needed.

## Cluster Creation

1. Initialize Terraform:
   ```
   cd dev-multinode-k8s
   terraform init
   ```
2. Apply the configuration with Terraform:
   ```
   sudo terraform apply
   ```
3. Initialize the Control Plane node:
   ```
   ssh admin@192.168.40.162
   cd setup
   ./init.sh
   ```
4. Copy the join command from the script execution log and initialize the worker nodes.

## Add/Remove Worker Nodes

### Remove Worker Nodes

1. Modify the `worker_count` parameter in `k8s.auto.tfvars` file, setting a smaller value, then run the Terraform apply command:
```
sudo terraform apply
```
Note that in this way the virtual machines corresponding to the removed nodes will be destroyed and this could have some unwanted impacts on the running pods. See Safely Drain a Node for recommendations on how to properly remove a node from the cluster.

### Add New Worker Nodes

1. Modify the `worker_count` parameter in `k8s.auto.tfvars`, setting a higher value.
2. Create additional Butane files in the `config` folder for each new worker if required.
3. Execute `sudo terraform apply` to apply the changes.

### Nodes Configuration Updates

There are two ways to modify the nodes' configuration parameters:

1. **Using Terraform**: Modify the parameters in the configuration files and execute `sudo terraform apply`.
2. **Using libvirt (Virtual Machine Manager or virsh command)**: This method is recommended if Terraform plans to destroy any resources.

#### Example: Changing the Startup Behavior

- To change the automatic startup of the nodes, modify the `autostart` parameter in `k8s.auto.tfvars`.

## Control the Cluster from the Host Machine

- Install `kubectl` on the host machine.
- Copy the `.kube/config` file from the Control Plane node to the host machine.

## Deploy an NGINX Ingress Controller

```bash
kubectl apply -f https://raw.githubusercontent.com/kubernetes/ingress-nginx/controller-v1.6.4/deploy/static/provider/cloud/deploy.yaml
```

- Check the status of the controller:
  ```bash
  kubectl describe -n ingress-nginx deploy/ingress-nginx-controller
  ```

## Deploy and Access the Kubernetes Dashboard

- To deploy and access the Kubernetes Dashboard, visit [Kubernetes Dashboard Documentation](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/).

## Destroy the Cluster

- To destroy the cluster and remove all the created virtual machines:
  ```bash
  sudo terraform destroy
  ```

## References

- Creating a Kubernetes Cluster with Fedora CoreOS 36
- Fedora CoreOS - Basic Kubernetes Setup
- Creating a cluster with kubeadm

---

*Published by Luigi Quattrocchi, Senior Software Engineer at TIM*

# Tags

\#terraform \#kubernetes \#k8s \#virtualization \#libvirt \#linux \#fedora
```

This Markdown representation provides a structured and readable format of the article, suitable for documentation or publishing in platforms that support Markdown.