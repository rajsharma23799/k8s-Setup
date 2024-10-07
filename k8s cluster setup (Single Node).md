# 1. Disable Firewall in case of on-prem VMs or Add security group to allow these following ports:
 For MASTER Node:
  - **6443** - K8s API Server used by all.
  - **2379-2380** - etcd server client API used by kube-apiserver, etcd.
  - **10250** - Kubelet API used and self, control plane
  - **10251** - kube-scheduler used by self.
  - **10252** - kube-controller-manager used by self.
 For WORKER Node:
  - 10250 - Kubelet API used by self and control plane
  - 30000-32767 - NodePort Services used by All.

  ```
  systemctl disable firewalld
  systemctl stop firewalld
```

# 2. We need to disable the swap as it is not supported by K8s
  ```
  swapoff -a
  sed -i '/swap/d' /etc/fstab
```

# 3. Disable SELINUX as this will create and ask for explanation at each steps
  ```
  setenforce 0
  sed -i --follow-symlinks 's/^SELINUX=enforcing/SELINUX=disabled/' /etc/sysconfig/selinux
```

# 4. Now we have to setup some configuration for the K8s networking:
  ```
  vi  /etc/sysctl.d/kubernetes.conf
  	>net.bridge.bridge-nf-call-ip6tables = 1
  	>net.bridge.bridge-nf-call-iptables = 1
```

# 5. Reload the system to apply changes
  ```
  sysctl --system
```

# 6. Install docker prerequisites
  ```
  yum install -y yum-utils device-mapper-persistent-data lvm2
```

# 7. Add repo for docker
  ```
  yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo
```

# 8. Install container.io service that will manage the container which were going to be deploy with the help of kubernetes
  ```
  yum intall -y containerd.io
```

# 9. Now we'll setup some configuraiton for the containerd
  ```
  mkdir -p /etc/containerd

  ### Copy the default containderd conf in /etc folder
  containerd config default | sudo tee /etc/containerd/config.toml
```

  ( In this if we can some error like this: W0227 23:16:15.926702 55879 checks.go:835] detected that the sandbox image “registry.k8s.io/pause:3.8 1” of the container runtime is 
    inconsistent with that used by kubeadm. It is recommended that using “registry.k8s.io/pause:3.9 6” as the CRI sandbox image.
	  Just change to registry.k8s.io/pause:3.9 in config.toml and 
	  SystemCgroup set to true )

# 10. Restart and enable containerd
  ```
  systemctl restart containerd
  systemctl enable containerd
```

# 11. Setup repo fo rthe kubernetes
  ```
  vi /etc/yum.repos.d/kubernetes.repo


  Add the following text to the .repo file
    [kubernetes]
	  name=kubernetes
	  baseurl=https://pkgs.k8s.io/core:/stable:/v1.29/rpm/
	  enable=1
	  gpgcheck=1
	  gpgkey=https://pkgs.k8s.io/core:/stable:/v1.29/rpm/repodata/repomd.xml.key
	  exclude=kubelet kubeadm kubectl cri-tools kubernetes-cni
```

# 12. Install the kubelete, kubeadm, kubectl with the --disableexcludes flag set to kubernetes
  ```
  yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes

  if file already exists error appear remove the existing the /kubernetes directory:
  rm -rf /etc/kubernetes
```

# 13. For master node:
  Intialize the control plane 
  ```
  kubeadm init --apiserver-advertise-address=<ADDRESS OF MASTER NODE> --pod-network-cidr=<SUBNET FOR PODS>/16

  and we can get the join command for worker nodes by executing the command on master node:
  kubeadm token create --print-join-command
```
# 14. For worker nodes first enable the kubelet 
  ```
  systemclt enable kubelet
  kubeadm join 10.125.193.136:6443 --token gdze90.i4i1wmtqtzwvtu2h --discovery-token-ca-cert-hash sha256:0408fc4999c96ec15c706ce6c01612d3b7f53713dbc5ccdb875545a67f6f7f1f
  ```











