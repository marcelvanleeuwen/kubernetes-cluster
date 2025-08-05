# Tips & Trick
<br>

### Connect old k8s-worker to new control plane
<br>
Remove old configuration data

```sh
sudo kubeadm reset
```

```sh
sudo rm -rf /etc/cni/net.d
```

```sh
sudo rm -rf ~/.kube
```

```sh
sudo rm -rf /var/lib/kubelet/*
```
<br>
Done with wipping old configuration data
<br>
<br>

Command to add node. Execute this on the control plane.
<br>

```sh
kubeadm token create --print-join-command --ttl 0
```
<br>
Use output to add node. So execute this command on the node to add.
<br>
<br>


### Install Bash 5.x, Bash Completion, KubeCtl on MacOS
<br>

##### Install HomeBrew
<br>

```sh
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```
<br>

add the following to
<br>
```sh
nano ~/.bash_profile
```

```sh
#Homebrew pad toevoegen (voor ARM Macs = Apple Silicon)
eval "$(/opt/homebrew/bin/brew shellenv)"
```
<br>

##### Install Bash
<br>

```sh
brew install bash
```
<br>

Set Bash 5.x as default
<br>

```ssh
sudo bash -c 'echo /opt/homebrew/bin/bash >> /etc/shells'
```

```sh
chsh -s /opt/homebrew/bin/bash
```
<br>

Check if it is default
<br>
```sh
echo $SHELL
```

```sh
bash --version
```

```sh
which bash
```
<br>
Open and close terminal to see if everything keeps working
<br>

<br>

##### Install Bash Completion and Kubectl Completion 
<br>

```sh
brew install auto-completion@2
```
<br>

add the following to
<br>
```sh
nano ~/.bash_profile
```

```sh
# bash-completion v2
if [ -f "$(brew --prefix)/etc/profile.d/bash_completion.sh" ]; then
  source "$(brew --prefix)/etc/profile.d/bash_completion.sh"
fi

# kubectl completion
source <(kubectl completion bash)
```
<br>

##### Install Kubectl
<br>

```sh
brew install kubectl
```
<br>

##### complete ~/.bash_profile

```sh
# Homebrew pad toevoegen (voor ARM Macs = Apple Silicon)
eval "$(/opt/homebrew/bin/brew shellenv)"

# bash-completion v2
if [ -f "$(brew --prefix)/etc/profile.d/bash_completion.sh" ]; then
  source "$(brew --prefix)/etc/profile.d/bash_completion.sh"
fi

# kubectl completion
source <(kubectl completion bash)
```
<br>

### Add an alias to shorten the kubectl command

add the folowing to ~/.bashrc

```sh
alias k='kubectl'
```
<br>


### Show ETCD options

```sh
sudo cat /etc/kubernetes/manifests/etcd.yaml
```
<br>

### Show Kube-scheduler options

```sh
sudo cat /etc/kubernetes/manifests/kube-scheduler.yaml
```
<br>

### Show Kube-controller options

```sh
sudo cat /etc/kubernetes/manifests/kube-controller-manager.yaml 
```
<br>

### Check if Kubelet is runing and show options

```sh
ps -aux | grep kubelet
```