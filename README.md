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
