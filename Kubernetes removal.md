# Removal of Kubernetes from Ubuntu

To completely delete Kubernetes from Ubuntu, you must remove the installed packages, reset the system to a clean state, and clean up any configuration files or leftover data. Here's a step-by-step guide to ensure you completely remove Kubernetes:
## 1. Stop Kubernetes Services

First, stop the Kubernetes services that are running on your system:
```
sudo systemctl stop kubelet
sudo systemctl stop docker
sudo systemctl stop crio
```

## 2. Reset Kubernetes Cluster

If you initialized the Kubernetes cluster using `kubeadm`, you can use `kubeadm reset` to undo the initialization and remove cluster-related configurations:

```
sudo kubeadm reset
```

This command will:

- Remove the cluster configuration files.
- Reset the Kubernetes control plane.
- Delete Kubernetes-related data from `/etc/kubernetes`.

## 3. Remove Kubernetes Packages

Remove the Kubernetes packages (kubeadm, kubelet, kubectl, and any other related tools) using the following commands:

```
sudo apt-get purge -y kubeadm kubectl kubelet kubernetes-cni
```

After that, remove any remaining Kubernetes-related packages:
```
sudo apt-get autoremove -y
```

## 4. Delete Kubernetes Configuration Files

Remove any leftover Kubernetes configuration and data files:
```
sudo rm -rf ~/.kube
sudo rm -rf /etc/kubernetes
sudo rm -rf /var/lib/etcd
sudo rm -rf /var/lib/kubelet
sudo rm -rf /var/lib/cni
sudo rm -rf /var/run/kubernetes
```

## 5. Disable and Remove Network Interfaces

If you have specific Kubernetes network interfaces (like flannel or Calico) or network-related configurations, disable and remove them:
```
sudo ip link delete cni0
sudo ip link delete flannel.1
```

## 6. Remove Docker (if installed for Kubernetes)

If you installed Docker specifically for Kubernetes, you can remove it as well. If you plan to use Docker for other purposes, skip this step.
```
sudo apt-get purge -y docker docker-engine docker.io containerd runc
```

## 7. Reboot Your System

Finally, reboot your system to apply all changes:
```
sudo reboot
```
### Additional Steps (Optional):

If you want to remove any CNI (Container Network Interface) plugins, you can check and clean up any additional network plugins that were installed in /opt/cni/bin.

After following these steps, Kubernetes will be completely removed from your Ubuntu system. If you decide to reinstall Kubernetes later, you can start with a fresh setup.
