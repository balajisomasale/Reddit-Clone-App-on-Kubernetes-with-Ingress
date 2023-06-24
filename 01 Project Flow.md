## Reddit Clone 


## Prequisites : 

- Once 2 AWS EC2 servers are created, `CI-server`: with Ubuntu-T2 micro and other : `Deployment Server` with ubuntu-t2.medium
- Run below steps :

### To install Docker in CI server : 
```

# For Docker Installation
sudo apt-get update
sudo apt-get install docker.io -y
sudo usermod -aG docker $USER && newgrp docker

```
### Cloning the git project into the server: 

git clone https://github.com/balajisomasale/Reddit-Clone-App-on-Kubernetes-with-Ingress.git
```
completed until here --- 6 min 


```
### To install Minukube and kubectl : 

```
# For Minikube & Kubectl
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube 

sudo snap install kubectl --classic
minikube start --driver=docker
```
