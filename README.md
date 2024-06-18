# Installations-Guide für Ansible AWX.

## Voraussetzungen:
* 4GB RAM
* 2 CPU Cores
* Mindestens 20 GB freier Speicherplatz
* Internetverbindung
* Ubuntu Server 24.04 LTS
* Sudo Berechtigung

## Docker installieren

### Add Docker official GPG Key
To install latest docker on Ubuntu 24.04 LTS, first we need to add docker docker repository GPG key using below set of commands. 
So, start the terminal and execute these commands one after the another.
```
$ sudo apt update
$ sudo apt install ca-certificates curl -y
$ sudo install -m 0755 -d /etc/apt/keyrings
$ sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
$ sudo chmod a+r /etc/apt/keyrings/docker.asc
```
### Add Docker Official APT Repository
After Installing docker gpg key, add its official apt repository by running the following echo command.
```
$ echo \
  "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \
  $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \
  sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
  ```
### Install Docker on Ubuntu 24.04
As we have enabled the docker official apt repository, so we are good to start the docker installation. Run following apt command to install latest version of docker on your Ubuntu 24.04 system.
```
$ sudo apt update
$ sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
```
Once docker and its dependencies are installed then add your local user to docker group so that local user can run docker command with sudo.
```
$ sudo usermod -aG docker $USER
$ newgrep docker
$ docker --version
```
### Test Docker Installation
In order to test docker installation, let’s try to spin up a container using hello-world image. Run following docker command.
```
$ docker run hello-world
```

## Minikube installieren

### Apply Updates
Install all updates of existing packages of your system by executing the following apt commands from the terminal.
```
$ sudo apt update
$ sudo apt upgrade -y
```
Once all the updates are installed then reboot your system.
```
$ sudo reboot
```
### Install Minikube Dependencies
Run the following to Install minikube dependencies.
```
$ sudo apt install -y curl wget apt-transport-https
```
### Download and Install Minikube Binary
Use the following curl command to download latest minikube binary,
```
$ curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
```
Once the binary is downloaded then install it under the path /usr/local/bin.
```
$ sudo install minikube-linux-amd64 /usr/local/bin/minikube
```
Verify the minikube version
```
$ minikube version
minikube version: v1.33.1
commit: 5883c09216182566a63dff4c326a6fc9ed2982ff
$
```
Note: At the time of writing this tutorial, latest version of minikube was v1.33.1.
###  Install Kubectl Utility
Kubectl is a command line utility which is used to interact with Kubernetes cluster. It is used for managing deployments, service and pods etc. Use below curl command to download latest version of kubectl.
```
$ curl -LO https://storage.googleapis.com/kubernetes-release/release/`curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt`/bin/linux/amd64/kubectl
```
Once kubectl is downloaded then set the executable permissions on kubectl binary and move it to the path /usr/local/bin.
```
$ chmod +x kubectl
$ sudo mv kubectl /usr/local/bin/
```
###  Start Minikube
As we are already stated in the beginning that we would be using docker as base for minikue, so start the minikube with the docker driver, run
```
$ minikube start --driver=docker
```
In case you want to start minikube with customize resources and want installer to automatically select the driver then you can run following command,
```
$ minikube start --addons=ingress --cpus=2 --cni=flannel --install-addons=true --kubernetes-version=stable --memory=6g
```
Run below minikube command to check status,
```
$ minikube status
minikube
type: Control Plane
host: Running
kubelet: Running
apiserver: Running
kubeconfig: Configured
```
Run following kubectl command to verify the Kubernetes version, node status and cluster info.
```
$ kubectl cluster-info
$ kubectl get nodes
```
To enable Ingress controller addon, run
```
$ minikube addons enable ingress
```
## AWX installieren
### Install Required Packages
Log in to your Ubuntu system and install following required packages
```
$ sudo apt install git make -y
```
### Start minikube cluster
To start the minikube, run following minikube command as regular user,
```
$ minikube start --vm-driver=docker --addons=ingress
```
Execute beneath commands to verify minikube and pods status
```
$ minikube status
$ kubectl get pods -A
```
### Deploy Ansible AWX via Operator
Use following git command to get AWX operator,
```
$ git clone https://github.com/ansible/awx-operator.git
$ cd awx-operator/
$ git checkout 2.4.0
```
Note: At the time of writing this post, latest version of AWX operator was 2.4.0. You can check operator version from “https://github.com/ansible/awx-operator/releases”.
Next , set the namespace “ansibe-awx” and run make deploy command
```
$ export NAMESPACE=ansible-awx
$ make deploy
```
check the status pods from ansible-awx namespace,
```
$ kubectl get pods -n ansible-awx
```
Deploy AWX using following command,
```
$ kubectl create -f awx-demo.yml -n ansible-awx
```
Monitor pods and service status of ansible-awx namespace,
```
$ kubectl get pods -n ansible-awx
$ kubectl get svc -n ansible-awx
```
It will take 5 minutes to create all containers. The Installation is completed, if all these containers are running:
```
NAME                                               READY   STATUS             RESTARTS         AGE
awx-demo-postgres-13-0                             1/1     Running            0                50m
awx-demo-task-5bc94c64f4-xfmqt                     3/4     CrashLoopBackOff   11 (4m52s ago)   49m
awx-demo-web-7b694ddd7f-hkjpj                      2/3     CrashLoopBackOff   11 (4m36s ago)   47m
awx-operator-controller-manager-7f77fc6964-s9cnn   2/2     Running            0                54m
```
You can track the installation for AWX from pod,
```
$ kubectl logs awx-operator-controller-manager-6c58d59d97-vvkvc -n ansible-awx -f
```
Replace pod name as per your setup.
### Access AWX Dashboard
To access the dashboard from Ubuntu system itself, run following command to get dashboard url,
```
$ minikube service awx-demo-service --url -n ansible-awx
http://192.168.49.2:30182
$
```
In case, you are trying to access outside of your ubuntu system then run following kubectl command,
```
$ kubectl port-forward service/awx-demo-service -n ansible-awx --address 0.0.0.0 10445:80 &< /dev/null
```
Open the web broswer, type following URL
http://your servers ip:10445
To retrieve admin user password, run the following kubectl command,
```
$ kubectl get secret awx-demo-admin-password -o jsonpath="{.data.password}" -n ansible-awx | base64 --decode; echo
```
Login with the user: admin and the passwordkey from above.
