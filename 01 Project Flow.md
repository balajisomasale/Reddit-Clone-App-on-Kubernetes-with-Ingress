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
- Once the code is there, check if you have the `Dockerfile` which builds `Docker Image`
- Build Docker image by using : `docker build . -t balu9198/balaji-reddit-clone`
- check if the image is created : `docker images`

  ![image](https://github.com/balajisomasale/Reddit-Clone-App-on-Kubernetes-with-Ingress/assets/35003840/978f622e-5dea-4b95-9188-1f533a289576)

- Login into Docker Hub : `docker login` and enter the details
- Push the docker image to Docker Hub : `docker push balu9198/balaji-reddit-clone`

  ![image](https://github.com/balajisomasale/Reddit-Clone-App-on-Kubernetes-with-Ingress/assets/35003840/f38ea0dd-3ac4-4b33-ba30-062344575c11)

- checking the Docker Hub :

  ![image](https://github.com/balajisomasale/Reddit-Clone-App-on-Kubernetes-with-Ingress/assets/35003840/dcfd9ad6-c9fe-4fa9-936d-3e42d791b1e7)

### Create a Kubernetes-Deployment-Server:

- create a new ec2 instance with Ubuntu and `T2.medium` size

![image](https://github.com/balajisomasale/Reddit-Clone-App-on-Kubernetes-with-Ingress/assets/35003840/c3b70cd6-cb52-4f1f-a9c8-2bf8234456dc)



```
### To install Docker, Minukube and kubectl : 

```
we need to install docker also to run the minikube and kubectl

# For Docker Installation
sudo apt-get update
sudo apt-get install docker.io -y
sudo usermod -aG docker $USER && newgrp docker

# For Minikube & Kubectl
curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64
sudo install minikube-linux-amd64 /usr/local/bin/minikube 

sudo snap install kubectl --classic
minikube start --driver=docker
```
To check if kubernetes is installed : `kubectl get pods`

Now, we need to `Deploy` the reddit-clone from one server to other 

- we need to write the `kubernetes Manifest file` for the deployment.

Let's Create a Deployment File For our Application. Use the following code for the Deployment.yml file.

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: reddit-clone-deployment
  labels:
    app: reddit-clone
spec:
  replicas: 2
  selector:
    matchLabels:
      app: reddit-clone
  template:
    metadata:
      labels:
        app: reddit-clone
    spec:
      containers:
      - name: reddit-clone
        image: trainwithshubham/reddit-clone
        ports:
        - containerPort: 3000
```
- In above `deployment.yml` file, we need to aware of how many `replicas` we are giving interms of scalability in real time
- `spec` is very important parameter where we need to provide which `container,image and ports` we need the deployment.

On deployment server, create a folder : `mkdir k8s` and `cd k8s` and `vim deployment.yml`

- To deploy we use : `kubectl apply -f deployment.yml`
- To check its status : `kubectl get deployment`

![image](https://github.com/balajisomasale/Reddit-Clone-App-on-Kubernetes-with-Ingress/assets/35003840/ac4f94a5-80db-405a-9033-c85ca122c333)

Here Ready `2/2` specifies the `replicas` and if the traffic is very high, we can change the replicas to 4 and then `kubectl apply -f deployment.yml`
and then `get` the deployments and it will show ready as `4/4`

- But How to access the pods, we need `service.yml` file which creates an `IP` for all the pods in the deployment

```
apiVersion: v1
kind: Service
metadata:
  name: reddit-clone-service
  labels:
    app: reddit-clone
spec:
  type: NodePort
  ports:
  - port: 3000
    targetPort: 3000
    nodePort: 31000
  selector:
    app: reddit-clone

```
- Run the service.yml file : `kubectl apply -f service.yml`
- check the service pod : `kubectl get service`
  
![image](https://github.com/balajisomasale/Reddit-Clone-App-on-Kubernetes-with-Ingress/assets/35003840/c56767dd-b0d5-44c8-b7e2-4d13c98c941c)

- Now, we can access this cluster using : `minikube service reddit-clone-service --url`
- we can curl that address : `curl -L http://192.168.49.2:31000`

Note : we are skipping the `Ingress` part and can be integrated after the deployment 

- So, we cannot yet access the website as we did not expose it to the world
- we need to change the security group settings : `SG > Inbound > 3000 `

![image](https://github.com/balajisomasale/Reddit-Clone-App-on-Kubernetes-with-Ingress/assets/35003840/aef99d9d-3690-4ef5-b136-2b870bb2adc9)

Then We have to expose our app service kubectl port-forward svc/reddit-clone-service 3000:3000 --address 0.0.0.0 &

![image](https://github.com/balajisomasale/Reddit-Clone-App-on-Kubernetes-with-Ingress/assets/35003840/7c7b4388-187a-423f-9a53-b7b83104af44)

Output : 

![image](https://github.com/balajisomasale/Reddit-Clone-App-on-Kubernetes-with-Ingress/assets/35003840/10cc2715-8842-46aa-9a3a-f20df2563be2)

---------------------

Integrating with `Ingress` service: 

Pods and services in Kubernetes have their own IP; however, it is normally not the interface you'd provide to the external internet. Though there is a service with node IP configured, the port in the node IP can't be duplicated among the services. It is cumbersome to decide which port to manage with which service. Furthermore, the node comes and goes, it wouldn't be clever to provide a static node IP to an external service. Ingress defines a set of rules that allows the inbound connection to access Kubernetes cluster services. It brings the traffic into the cluster at L7 and allocates and forwards a port on each VM to the service port. This is shown in the following figure. We define a set of rules and post them as source type ingress to the API server. When the traffic comes in, the ingress controller will then fulfill and route the ingress by the ingress rules. As shown in the following figure, ingress is used to route external traffic to the Kubernetes endpoints by different URLs:

![image](https://github.com/balajisomasale/Reddit-Clone-App-on-Kubernetes-with-Ingress/assets/35003840/03bd4710-e65a-487c-8d6c-7c60ac245b52)


Let's write ingress.yml and put the following code in it:

```
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: ingress-reddit-app
spec:
  rules:
  - host: "domain.com"
    http:
      paths:
      - pathType: Prefix
        path: "/test"
        backend:
          service:
            name: reddit-clone-service
            port:
              number: 3000
  - host: "*.domain.com"
    http:
      paths:
      - pathType: Prefix
        path: "/test"
        backend:
          service:
            name: reddit-clone-service
            port:
              number: 3000

```

------

- Now, we need to `enable` Ingress with minikube as it is not done by default

![image](https://github.com/balajisomasale/Reddit-Clone-App-on-Kubernetes-with-Ingress/assets/35003840/334bd9ab-c8f3-4bde-8f01-49cfa2690404)

Now, we apply the changes using :

![image](https://github.com/balajisomasale/Reddit-Clone-App-on-Kubernetes-with-Ingress/assets/35003840/c4891dc0-3f0c-4ff6-99bf-1229623ef820)

We can access all the code by this endpoint url itself : 

![image](https://github.com/balajisomasale/Reddit-Clone-App-on-Kubernetes-with-Ingress/assets/35003840/71ce93c4-af90-4efd-86b8-5d9cea2bc885)

--------------------------- EOF----- 
