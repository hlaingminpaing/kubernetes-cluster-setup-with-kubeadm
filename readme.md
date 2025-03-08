# Master Node Setup

### Update the package index
```sh
sudo apt-get update 
```
### Update packages required for HTTPS package repository access

```sh
sudo apt-get install -y 
apt install ca-certificates curl software-properties-common gnupg lsb-release
```

### Allow forwarding IPv4 by loading the br_netfilter module with the follow commands:

```sh
#Load br_netfilter module
sudo modprobe overlay
sudo modprobe br_netfilter
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```


### sysctl params required by setup, params persist across reboots
```sh
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
# Apply sysctl params without reboot
sudo sysctl --system
```

### Install containerd using the DEB package distributed by Docker with the following commands:

#### Add Docker’s official GPG key
```sh
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
# Set up the repository
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
# Install containerd
sudo apt-get update
sudo apt-get install -y containerd.io=1.7.19-1
```

Note: This is only one way of installing containerd. Please refer https://github.com/containerd/containerd/blob/main/docs/

### Configure the systemd cgroup driver with the following commands:

Configure the systemd cgroup driver

```sh
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
sudo systemctl restart containerd
This is required to mitigate the instability of having two cgroup managers. Please refer https://kubernetes.io/docs/setup/production-environment/container-runtimes/
```

### Install kubeadm, kubectl, and kubelet from the official Kubernetes package repository:

##### Add the public signing key for the Kubernetes package repositories
```sh
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
# Add the Kubernetes release repository
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
# Update the package index to include the Kubernetes repository
sudo apt-get update
# Install the packages
sudo apt-get install -y kubelet kubeadm kubectl
```

### Prevent automatic updates to the installed packages with the following command:

```sh
sudo apt-mark hold kubelet kubeadm kubectl
```

Display the help page for kubeadm:

```sh
kubeadm
```
### Initialize the control-plane node using the init command:

```sh
sudo kubeadm init --pod-network-cidr=192.168.0.0/16 

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

kubectl get componentstatuses

kubectl get nodes
kubectl describe nodes
```
### Create the Calico network plugin for pod networking

```sh
kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
```

### Worker Node Setup

### Update the package index
```sh
sudo apt-get update 
```
### Update packages required for HTTPS package repository access

```sh
sudo apt-get install -y 
apt-transport-https ca-certificates curl software-properties-common gnupg lsb-release
```

### Allow forwarding IPv4 by loading the br_netfilter module with the follow commands:

```sh
#Load br_netfilter module
sudo modprobe overlay
sudo modprobe br_netfilter
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF
```


### sysctl params required by setup, params persist across reboots
```sh
cat <<EOF | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf
net.bridge.bridge-nf-call-iptables  = 1
net.ipv4.ip_forward                 = 1
net.bridge.bridge-nf-call-ip6tables = 1
EOF
# Apply sysctl params without reboot
sudo sysctl --system
```

### Install containerd using the DEB package distributed by Docker with the following commands:

#### Add Docker’s official GPG key
```sh
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
# Set up the repository
echo \
  "deb [arch=amd64 signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
# Install containerd
sudo apt-get update
sudo apt-get install -y containerd.io=1.7.19-1
```

Note: This is only one way of installing containerd. Please refer https://github.com/containerd/containerd/blob/main/docs/

### Configure the systemd cgroup driver with the following commands:

Configure the systemd cgroup driver

```sh
sudo mkdir -p /etc/containerd
containerd config default | sudo tee /etc/containerd/config.toml
sudo sed -i 's/SystemdCgroup = false/SystemdCgroup = true/' /etc/containerd/config.toml
sudo systemctl restart containerd
This is required to mitigate the instability of having two cgroup managers. Please refer https://kubernetes.io/docs/setup/production-environment/container-runtimes/
```

### Install kubeadm, kubectl, and kubelet from the official Kubernetes package repository:

##### Add the public signing key for the Kubernetes package repositories
```sh
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.30/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
# Add the Kubernetes release repository
echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.30/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
# Update the package index to include the Kubernetes repository
sudo apt-get update
# Install the packages
sudo apt-get install -y kubelet kubeadm kubectl
```

### Prevent automatic updates to the installed packages with the following command:

```sh
sudo apt-mark hold kubelet kubeadm kubectl
kubeadm join 10.0.0.100:6443 --token mtcv1t.pvyu0ij061oake7t --discovery-token-ca-cert-hash sha256:991981f37a96591e8e4fe57ce761ab7b7832a8a90c76e612234fc1c8b9fcbb55 
```
