# Kubernetes Installation (v1.31)
## ubuntu guest addition:
```
sudo apt install open-vm-tools open-vm-tools-desktop -y
```
```
sudo reboot
```
## 1.1. Device minimum requirement
- Must have at least 2xCPU on Ubuntu
- Must have a static IP

## 1.2. Control plane / master node
Login as root  
Export following two variables in your current terminal:
```bash
KUBERNETES_VERSION=v1.31
CRIO_VERSION=v1.31
```


Install the dependencies for adding repositories:  
```bash
apt-get update
apt-get install -y software-properties-common curl
```


Add the Kubernetes repository:
```
curl -fsSL https://pkgs.k8s.io/core:/stable:/$KUBERNETES_VERSION/deb/Release.key |
    gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/$KUBERNETES_VERSION/deb/ /" | tee /etc/apt/sources.list.d/kubernetes.list
```

Add the CRI-O repository:

```
curl -fsSL https://pkgs.k8s.io/addons:/cri-o:/stable:/$CRIO_VERSION/deb/Release.key | gpg --dearmor -o /etc/apt/keyrings/cri-o-apt-keyring.gpg

echo "deb [signed-by=/etc/apt/keyrings/cri-o-apt-keyring.gpg] https://pkgs.k8s.io/addons:/cri-o:/stable:/$CRIO_VERSION/deb/ /" | tee /etc/apt/sources.list.d/cri-o.list
```


Install the packages:
```
apt-get update
apt-get install -y cri-o kubelet kubeadm kubectl
```


Start CRI-O:
```
systemctl start crio.service
```


Disable swap space:
```
swapoff -a
```


Check if all swap value becomes zero
```
free -h
```

Edit the swap file:
```
nano /etc/fstab
```

    Comment out last line like this:  
    # /swapfile


Change network config:
```
modprobe br_netfilter
sysctl -w net.ipv4.ip_forward=1
```

Or, to change network setting permanently:
```
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
EOF
sudo sysctl --system
sysctl net.ipv4.ip_forward
```
Bootstrap a cluster: If you have a single network interface, the Kubernetes cluster can be initiated with the following command.
```
kubeadm init
```

If the VM has multiple network interfaces, you may specify each interface's IP address and port number when initializing the Kubernetes cluster.
```
kubeadm init --control-plane-endpoint 192.168.56.101:6443
```


### kubeadm init will provide the following message:
```
To start using your cluster, you need to run the following as a regular user:

mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config

Alternatively, if you are the root user, you can run:

export KUBECONFIG=/etc/kubernetes/admin.conf

You should now deploy a pod network to the cluster.
Run "kubectl apply -f [podnetwork].yaml" with one of the options listed at:
https://kubernetes.io/docs/concepts/cluster-administration/addons/  
Then you can join any number of worker nodes by running the following on each as root:  

kubeadm join 192.168.119.134:6443 --token 26uac9.vq08kykgru1if5j1 \ --discovery-token-ca-cert-hash sha256:5b37d58b6c6e232dc379863f17cce84be69358c9f4887441cd1588b4809c17bd
```

  
So run the following commands:
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

## 1.3. Worker Node
Log in as root
Export follwoing two variables in your current terminal:  
```
KUBERNETES_VERSION=v1.31
CRIO_VERSION=v1.31
```
Install the dependencies for adding repositories:
```
apt-get update
apt-get install -y software-properties-common curl
```
Add the Kubernetes repository:
```
curl -fsSL https://pkgs.k8s.io/core:/stable:/$KUBERNETES_VERSION/deb/Release.key |
    gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg

echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/$KUBERNETES_VERSION/deb/ /" | tee /etc/apt/sources.list.d/kubernetes.list
```

Add the CRI-O repository:
```
curl -fsSL https://pkgs.k8s.io/addons:/cri-o:/stable:/$CRIO_VERSION/deb/Release.key | gpg --dearmor -o /etc/apt/keyrings/cri-o-apt-keyring.gpg

echo "deb [signed-by=/etc/apt/keyrings/cri-o-apt-keyring.gpg] https://pkgs.k8s.io/addons:/cri-o:/stable:/$CRIO_VERSION/deb/ /" |
    tee /etc/apt/sources.list.d/cri-o.list
```
Install the packages:
```
apt-get update
apt-get install -y cri-o kubelet kubeadm kubectl
```
Start CRI-O:
```
systemctl start crio.service
```

Disable swap space:
```
swapoff -a
```

Check if all swap value becomes zero:
```
free -h 
```

Edit the swap file:
```
nano /etc/fstab
```

    Comment out last line like this:  
    # /swapfile

Change network config:
```
modprobe br_netfilter
sysctl -w net.ipv4.ip_forward=1
```

Or, to change network setting permanently:
```
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.ipv4.ip_forward = 1
EOF
sudo sysctl --system
sysctl net.ipv4.ip_forward
```

Copy "kubeadm join" command from master node and paste here in worker node:
```
kubeadm join 192.168.119.134:6443 --token 26uac9.vq08kykgru1if5j1 \ --discovery-token-ca-cert-hash sha256:5b37d58b6c6e232dc379863f17cce84be69358c9f4887441cd1588b4809c17bd
```

## On Master Node  
Once joing command is successful, then check cluster status.
```
kubectl get node
```

## If any connectivity issue occurs:  
a. Port 6443 must be open on master note  
```
sudo ss -tuln | grep 6443
```

b. Check connectivity from worker node:
```
nc -zv 192.168.119.134 6443
```

c. Allow firewall for port 6443
```
sudo ufw allow 6443/tcp 
```

