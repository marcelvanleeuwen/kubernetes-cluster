# Install Kubernetes bare metal
<br>

#### Before you begin
<br>
<ul>
  <li>A compatible Linux host (I use the prefered OS Ubuntu LTS 24.04 because most</li>
  <li>Kubernetes development is done on Ubuntu)</li>
  <li>2GB or more RAM</li>
  <li>2 CPUs or more for control plane machines</li>
  <li>Full network connectivity</li>
  <li>Unique hostname, MAC address</li>
  <li>Certain ports open</li>
</ul>
<br>

#### Prefered OS 
<br>
Ubuntu Server LTS 24.04 (<i>because most Kubernetes development is done on Ubuntu</i>)
<br>
<br>

#### Install Ubuntu Server
<br>
<ul>
  <li>Install Ubuntu Server</li>
  <li>Set static ip during installation</li>
  <li>Reboot</li>
</ul>
<ul>
<li>Login</li>
<li>Update Ubuntu Server</li>
</ul>
<br>

```sh
sudo apt-get update && sudo apt-get upgrade -y 
```
<br>

<li>Set hostname</li>
<br>

```sh
set hostname {name}
```
<i>example: k8s-control.plane-1</i>
<br>

#### disable linux swap and remove any existing swap partitions
<br>

```sh
sudo swapoff -a
```
```sh
sudo sed -i '/\sswap\s/ s/^\(.*\)$/#\1/g' /etc/fstab
```
<br>

#### Setting up container runtime prereq
<br>

```sh
echo -e "overlay\nbr_netfilter" | sudo tee /etc/modules-load.d/containerd.conf > /dev/null
```
<br>

```sh
sudo modprobe overlay
```
```sh
sudo modprobe br_netfilter
```
<br>

#### Setup required sysctl params, these persist across reboots.
<br>

```sh
echo -e "net.bridge.bridge-nf-call-iptables = 1\nnet.ipv4.ip_forward = 1\nnet.bridge.bridge-nf-call-ip6tables = 1" | sudo tee /etc/sysctl.d/99-kubernetes-cri.conf > /dev/null
```
<br>

#### Apply sysctl params without reboot
<br>

```sh
sudo sysctl --system
```
<br>

#### install containerd
<br>

```sh
sudo apt-get install docker.io containerd
```
<br>

#### install containerd latest version over apt-installed-version
<br>

```sh
sudo systemctl stop containerd
```
```sh
sudo wget https://github.com/containerd/containerd/releases/download/v2.1.1/containerd-2.1.1-linux-amd64.tar.gz
```
```sh
sudo tar xvf containerd-2.1.1-linux-amd64.tar.gz
```
```sh
sudo mv bin/* /usr/bin
```
```sh
sudo rm -rf bin containerd-2.1.1-linux-amd64.tar.gz
```
<br>

#### containerd config
<br>

```sh
sudo mkdir -p /etc/containerd
```
<br>

```sh
sudo tee /etc/containerd/config.toml > /dev/null <<EOF
version = 2
[plugins]
  [plugins."io.containerd.grpc.v1.cri"]
    [plugins."io.containerd.grpc.v1.cri".containerd]
      discard_unpacked_layers = true
      [plugins."io.containerd.grpc.v1.cri".containerd.runtimes]
        [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc]
          runtime_type = "io.containerd.runc.v2"
          [plugins."io.containerd.grpc.v1.cri".containerd.runtimes.runc.options]
            SystemdCgroup = true
EOF
```

```sh
sudo systemctl start containerd
```
<br>

#### install required packages
<br>

```sh
sudo apt-get install apt-transport-https
```
<br>

#### install kubernetes packages
<br>

```sh
sudo curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.33/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
```
```sh
sudo echo 'deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.33/deb/ /' | sudo tee /etc/apt/sources.list.d/kubernetes.list
```
```sh
sudo apt-get update
```
```sh
sudo apt-get install kubelet kubeadm kubernetes-cni kubectl #### control plane
```
```sh
sudo apt-get install kubelet kubeadm kubernetes-cni #### worker
```
<br>

#### crictl uses containerd as default
<br>

```sh
echo "runtime-endpoint: unix:///run/containerd/containerd.sock" | sudo tee /etc/crictl.yaml
```
<br>

#### kubelet should use containerd
<br>

```sh
echo 'KUBELET_EXTRA_ARGS="--container-runtime-endpoint=unix:///run/containerd/containerd.sock"' | sudo tee /etc/default/kubelet > /dev/null
```
<br>

#### start services
<br>

```sh
sudo systemctl daemon-reload
```
```sh
sudo systemctl enable containerd
```
```sh
sudo systemctl restart containerd
```
```sh
sudo systemctl enable kubelet && systemctl start kubelet
```
<br>

#### init k8s control plane
<br>

```sh
sudo kubeadm init 
```
<br>

To start using your cluster, you need to run the following as a regular user:
<br>
```sh
mkdir -p $HOME/.kube
```
```sh
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
```
```sh
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```
<br>

Alternatively, if you are the root user, you can run:
<br>
```sh
export KUBECONFIG=/etc/kubernetes/admin.conf
```
<br>

#### CNI (install calico network plugin ** required **)
<br>

```sh
kubectl apply -f https://raw.githubusercontent.com/projectcalico/calico/v3.27.2/manifests/calico.yaml
```
<br>

#### COMMAND TO ADD A WORKER NODE
<br>

```sh
kubeadm token create --print-join-command --ttl 0
```
<br>
Use output to add node. So execute this command on the node to add.
<br>
<br>

#### Change roles label
<br>

```sh
kubectl label node k8s-worker-1 node.role.kubernetes.io/worker=worker
```
<i>For some reason the "roles" doesn't change after this command</i>
