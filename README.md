# k8sclusterkubeadm

## Prerequisites
- Install VirtualBox
-  Install Vagrant
- `git clone https://github.com/mrbilalshafiq/k8sclusterkubeadm.git && cd k8sclusterkubeadm`


## Spin up VMs 

1. Go the folder containing the Vagrantfile and run:
` vagrant status`

2. When you can see the VM's it intends to create, run:
`vagrant up`

3. Then to ssh into the kubemaster, type:
`vagrant ssh kubemaster`

## Forwarding IPv4 and letting iptables see bridged traffic ######
> Run on all VMs

4. Verify that the br_netfilter module is loaded by running:
`lsmod | grep br_netfilter`

5. To load it explicitly, run:
`sudo modprobe br_netfilter`

6. In order for a Linux node's iptables to correctly view bridged traffic, verify that net.bridge.bridge-nf-call-iptables is set to 1 in your sysctl config. For example: 
```
cat <<EOF | sudo tee /etc/modules-load.d/k8s.conf
overlay
br_netfilter
EOF

sudo modprobe overlay
sudo modprobe br_netfilter
```

7. sysctl params required by setup, params persist across reboots
```
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-iptables  = 1
net.bridge.bridge-nf-call-ip6tables = 1
net.ipv4.ip_forward                 = 1
EOF
```

8. Apply sysctl params without reboot
`sudo sysctl --system`

## INSTALL DOCKER 

9. Older versions of Docker went by the names of docker, docker.io, or docker-engine. Uninstall any such older versions before attempting to install a new version:
`sudo apt-get remove docker docker-engine docker.io containerd runc`

10. Update the apt package index and install packages to allow apt to use a repository over HTTPS:
```
sudo apt-get update
sudo apt-get install -y \
    ca-certificates \
    curl \
    gnupg \
    lsb-release
```

11. Add Docker’s official GPG key:
```
sudo mkdir -p /etc/apt/keyrings
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /etc/apt/keyrings/docker.gpg
```

12. Use the following command to set up the repository:
```
echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.gpg] https://download.docker.com/linux/ubuntu \
  $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
```

13. Update the apt package index:
`sudo apt-get update`

14. If error:
```
sudo chmod a+r /etc/apt/keyrings/docker.gpg
sudo apt-get update
```

15. For the latest version:
`sudo apt-get install -y docker-ce docker-ce-cli containerd.io docker-compose-plugin`

16. Create the docker group:
`sudo groupadd docker`

17. Add your user to the docker group:
`sudo usermod -aG docker $USER`

18. Run the following command to activate the changes to groups:
`newgrp docker`

19. Verify that you can run docker commands without sudo:
`docker run --rm hello-world`

## INSTALLING kubeadm, kubelet, & kubectl

20. Update the apt package index and install packages needed to use the Kubernetes apt repository:
```
sudo apt-get update
sudo apt-get install -y apt-transport-https ca-certificates curl
```

21. Download the Google Cloud public signing key:
`sudo curl -fsSLo /usr/share/keyrings/kubernetes-archive-keyring.gpg https://packages.cloud.google.com/apt/doc/apt-key.gpg`

22. Add the Kubernetes apt repository:
`echo "deb [signed-by=/usr/share/keyrings/kubernetes-archive-keyring.gpg] https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee /etc/apt/sources.list.d/kubernetes.list`

23. Update apt package index, install kubelet, kubeadm and kubectl, and pin their version:
```
sudo apt-get update
sudo apt-get install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
```

24. Start the cluster on the master node
`kubeadm init --pod-network-cidr 10.244.0.0/16 --apiserver-advertise-address=192.168.56.2`

25. If having issues then comment out disabled_plugins = ["cri"] in /etc/containerd/config.toml:
```
sudo vi /etc/containerd/config.toml
sudo systemctl restart containerd
```

26. To start using your cluster, you need to run the following as a regular user:
```
mkdir -p $HOME/.kube
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

27. Verify the cluster
`kubectl get nodes`

28. You must deploy a Container Network Interface (CNI) based Pod network add-on so that your Pods can communicate with each other.
> Weave Net can be installed onto your CNI-enabled Kubernetes cluster with a single command:
`kubectl apply -f https://github.com/weaveworks/weave/releases/download/v2.8.1/weave-daemonset-k8s.yaml`
29. Join the nodes
```
kubeadm join 192.168.56.2:6443 --token xy1ftf.e6p1zgyvv8pg84ky \
        --discovery-token-ca-cert-hash sha256:0cec514854af79291486c0cb017f3e0b7974a7136fb327f10aa8f346df6956f5
```

30. See the nodes
`kubectl get nodes`
