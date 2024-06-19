# Docker installieren

## Docker GPG-Schlüssel hinzufügen

1. Aktualisieren Sie die Paketliste und installieren Sie erforderliche Pakete:
    ```sh
    sudo apt update
    sudo apt install ca-certificates curl -y
    ```

2. Erstellen Sie das Verzeichnis für die Schlüsselringe:
    ```sh
    sudo install -m 0755 -d /etc/apt/keyrings
    ```

3. Laden Sie den Docker GPG-Schlüssel herunter und speichern Sie ihn:
    ```sh
    sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
    sudo chmod a+r /etc/apt/keyrings/docker.asc
    ```

## Docker-Repository hinzufügen

4. Fügen Sie das offizielle Docker APT-Repository hinzu:
    ```sh
    echo \\
    "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu \\
    $(. /etc/os-release && echo "$VERSION_CODENAME") stable" | \\
    sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    ```

## Docker auf Ubuntu 24.04 installieren

5. Aktualisieren Sie die Paketliste und installieren Sie Docker:
    ```sh
    sudo apt update
    sudo apt install docker-ce docker-ce-cli containerd.io docker-buildx-plugin docker-compose-plugin -y
    ```

6. Fügen Sie den aktuellen Benutzer zur Docker-Gruppe hinzu, um Docker-Befehle ohne `sudo` ausführen zu können:
    ```sh
    sudo usermod -aG docker $USER
    newgrp docker
    docker --version
    ```

## Docker-Installation testen

7. Testen Sie die Docker-Installation, indem Sie einen Container mit dem `hello-world`-Image starten:
    ```sh
    docker run hello-world
    ```

# Minikube installieren

## Updates anwenden

1. Installieren Sie alle Updates der vorhandenen Pakete Ihres Systems:
    ```sh
    sudo apt update
    sudo apt upgrade -y
    sudo reboot
    ```

## Minikube-Abhängigkeiten installieren

2. Installieren Sie die erforderlichen Abhängigkeiten:
    ```sh
    sudo apt install -y curl wget apt-transport-https
    ```

## Minikube-Binary herunterladen und installieren

3. Laden Sie die neueste Minikube-Binary herunter:
    ```sh
    curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
    ```

4. Installieren Sie das Minikube-Binary:
    ```sh
    sudo install minikube-linux-amd64 /usr/local/bin/minikube
    ```

## Minikube-Version überprüfen

5. Überprüfen Sie die Minikube-Version:
    ```sh
    minikube version
    ```

## Kubectl-Dienstprogramm installieren

6. Laden Sie die neueste Version von kubectl herunter:
    ```sh
    curl -LO https://storage.googleapis.com/kubernetes-release/release/$(curl -s https://storage.googleapis.com/kubernetes-release/release/stable.txt)/bin/linux/amd64/kubectl
    ```

7. Setzen Sie die Ausführungsrechte und verschieben Sie die Binary:
    ```sh
    chmod +x kubectl
    sudo mv kubectl /usr/local/bin/
    ```

## Minikube starten

8. Starten Sie Minikube mit dem Docker-Treiber:
    ```sh
    minikube start --driver=docker
    ```

9. Alternativ können Sie Minikube mit angepassten Ressourcen und automatischer Treiberauswahl starten:
    ```sh
    minikube start --addons=ingress --cpus=2 --cni=flannel --install-addons=true --kubernetes-version=stable --memory=6g
    ```

## Minikube-Status überprüfen

10. Überprüfen Sie den Minikube-Status:
    ```sh
    minikube status
    ```

## Kubernetes-Version, Knotenstatus und Clusterinformationen überprüfen

11. Überprüfen Sie die Kubernetes-Clusterinformationen und den Status der Knoten:
    ```sh
    kubectl cluster-info
    kubectl get nodes
    ```

## Ingress-Controller-Addon aktivieren

12. Aktivieren Sie das Ingress-Controller-Addon:
    ```sh
    minikube addons enable ingress
    ```

# AWX installieren

## Erforderliche Pakete installieren

1. Installieren Sie Git und Make:
    ```sh
    sudo apt install git make -y
    ```

## Minikube-Cluster starten

2. Starten Sie den Minikube-Cluster:
    ```sh
    minikube start --vm-driver=docker --addons=ingress
    ```

3. Überprüfen Sie den Minikube-Status und den Status der Pods:
    ```sh
    minikube status
    kubectl get pods -A
    ```

## Ansible AWX über Operator bereitstellen

4. Klonen Sie das AWX-Operator-Repository und wechseln Sie zum entsprechenden Verzeichnis:
    ```sh
    git clone https://github.com/ansible/awx-operator.git
    cd awx-operator/
    git checkout 2.4.0
    ```

5. Setzen Sie den Namespace und führen Sie die Bereitstellung aus:
    ```sh
    export NAMESPACE=ansible-awx
    make deploy
    ```

6. Überprüfen Sie den Status der Pods im Namespace „ansible-awx“:
    ```sh
    kubectl get pods -n ansible-awx
    ```

7. Stellen Sie AWX bereit:
    ```sh
    kubectl create -f awx-demo.yml -n ansible-awx
    ```

8. Überwachen Sie den Status der Pods und Dienste im Namespace „ansible-awx“:
    ```sh
    kubectl get pods -n ansible-awx
    kubectl get svc -n ansible-awx
    ```

9. Verfolgen Sie die Installation von AWX aus dem Pod:
    ```sh
    kubectl logs awx-operator-controller-manager-6c58d59d97-vvkvc -n ansible-awx -f
    ```

## AWX-Dashboard zugreifen

10. Um das Dashboard vom Ubuntu-System aus zuzugreifen, führen Sie den folgenden Befehl aus, um die Dashboard-URL zu erhalten:
    ```sh
    minikube service awx-demo-service --url -n ansible-awx
    ```

11. Falls Sie von außerhalb Ihres Ubuntu-Systems zugreifen möchten, führen Sie den folgenden kubectl-Befehl aus:
    ```sh
    kubectl port-forward service/awx-demo-service -n ansible-awx --address 0.0.0.0 10445:80 &< /dev/null
    ```

12. Öffnen Sie den Webbrowser und geben Sie die folgende URL ein: `http://your servers ip:10445`.

13. Um das Administratorkennwort abzurufen, führen Sie den folgenden kubectl-Befehl aus:
    ```sh
    kubectl get secret awx-demo-admin-password -o jsonpath="{.data.password}" -n ansible-awx | base64 --decode; echo
    ```

14. Melden Sie sich mit dem Benutzer `admin` und dem aus dem obigen Befehl erhaltenen Passwort an