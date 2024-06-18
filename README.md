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
To install latest docker on Ubuntu 24.04 LTS, first we need to add docker docker repository GPG key using below set of commands. So, start the terminal and execute these commands one after the another.
```
$ sudo apt update
$ sudo apt install ca-certificates curl -y
$ sudo install -m 0755 -d /etc/apt/keyrings
$ sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
$ sudo chmod a+r /etc/apt/keyrings/docker.asc
```
### Add Docker Official APT Repository