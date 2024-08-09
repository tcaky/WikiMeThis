# Install VirtualBox
Installed VirtualBox on Windows 11

# Download Mint Linux ISO
1. Download [Linux Mint](https://linuxmint.com/edition.php?id=316)
2. Create new VM in VirtualBox and use Mint ISO



# Getting K8s realted artifacts installed.
Log onto the VM.

```shell
# From: https://dzone.com/articles/install-docker-kubernetes-and-minikube-on-linux-mi

sudo apt update

sudo apt install neovim
sudo apt install docker*

# add user to docker group if not the root user.
sudo usermod -aG docker $USER

# setup the key for kubernetes packages
curl -fsSL https://pkgs.k8s.io/core:/stable:/v1.28/deb/Release.key | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
sudo echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/v1.28/deb/ /" | sudo tee /etc/apt/sources.list.d/kubernetes.list

sudo apt update
sudo apt-get install -y kubectl

# Install Minikube
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube

# In a terminal...
minikube start --driver=docker

# Make Docker the default driver
https://dzone.com/articles/install-docker-kubernetes-and-minikube-on-linux-mi

```
