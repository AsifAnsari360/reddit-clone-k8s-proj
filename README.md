# Reddit Clone App on Kubernetes with Ingress
This project demonstrates how to deploy a Reddit clone app on Kubernetes with Ingress and expose it to the world using Minikube as the cluster.
Below is an overview of the architecture of this Reddit Clone App running on Kubernetes with Ingress.
![Architecture Diagram](https://github.com/LondheShubham153/reddit-clone-k8s-ingress/assets/71492927/e1eec5f2-1983-445b-8966-e9acfdea7f8e)

- In `CI Server`(Instance type: t2.micro, for testing on a small scale), the code will be build and an image will be created. The image will be pushed onto the **Docker hub**.
- In `Deployment Server`(Instance type: t2.xlarge, for testing and production Scale) Kubernetes will be set up to deploy the image on a scale.


## Prerequisites
Before you begin, you should have the following tools installed on your local machine: 

- Docker
- Minikube cluster ( Running )
- kubectl
- Git
- Also create account on Docker hub. To create account. [click here](https://hub.docker.com/)

You can install Prerequisites by doing these steps. And these steps are for Ubuntu AMI.
```bash
# Steps:-

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

## Installation
Follow these steps to install and run the Reddit clone app on your local machine:

### In CI Server:

1) First, Install Docker in the CI server.
```bash
# For Docker Installation
sudo apt-get update
sudo apt-get install docker.io -y
sudo usermod -aG docker $USER && newgrp docker
```
2) Clone this repository to your local machine(AWS):
```bash
git clone https://github.com/AsifAnsari360/reddit-clone-k8s-proj.git
```
3) Navigate to the project directory:
```bash
cd reddit-clone-k8s-proj
```
4) Build the Docker image for the Reddit clone app:
```bash
docker build . -t asifansarihot896000/reddit-clone    #To build Image
```
5) Login into docker hub account:
```bash
docker login
```
Add credentials and press Enter.

6) To push the Image to docker hub:
```bash
docker push asifansarihot896000/reddit-clone:latest
```

![dockerhub](https://github.com/user-attachments/assets/650fc54b-06c6-487c-949e-5c99eef17476)

**Here the First half of project is completed**

### In Deployment Server:

1) Install all the Prerequisites that is mentioned below:
```bash
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
2) Create New directory and change the directory:
```bash
mkdir k8s
cd k8s
```
3) Create **Deployment.yml** file and add following code:
```bash
apiVersion: apps/v1
kind: Deployment
metadata:
  name: reddit-clone-deployment
  labels:
    app: reddit-clone
spec:
  replicas: 1
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
        image: asifansarihot896000/reddit-clone    #You can also add your docker hub image
        ports:
        - containerPort: 3000
```
- Now RUN following command to deploy:
```bash
kubectl apply -f Deployment.yml
```
4)  Create **Service.yml** file and add following code:
```bash
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
- Now RUN following command to create service:
```bash
kubectl apply -f Service.yml
```
5) To Access application RUN following command:
```bash
minikube service reddit-clone-service --url
```
- Copy the URL provided and RUN it using the command:
```bash
curl -L <YOUR_URL>
```
The application will RUN in the terminal.
6) To Expose the application (To RUN it globally):
```bash
kubectl expose deployment reddit-clone-deployment --type=NodePort
```
- After running this command, add 3000 port in your instance inbound rules.
7) To Expose the service:
```bash
kubectl port-forward svc/reddit-clone-service 3000:3000 --address 0.0.0.0 &
```
- **&** at the end of the command is used to run the application in the background and you can continue working in your terminal. 
- After running the command, copy your instance public IP address and paste it into any web browser.
```bash
http://<Public IP>:3000/
```
The reddit-clone Web page will be visible.

![reddit clone](https://github.com/user-attachments/assets/34ab122b-99f7-4041-b1ba-1e6feca2aa83)

## Ingress

- In ingress, all the pods running are clustered in one service and we get a new cluster IP.
The IP comes with a particular domain name. Rules are created in ingress like for the frontend and backend with different domain name.

1) To enable Ingress controller:
```bash
minikube addons enable ingress
```
2) Create **ingress.yml** file and add following code:
```bash
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

3) To apply ingress settings RUN the following command:
```bash
kubectl apply -f ingress.yml
```
4) Verify that the ingress resource is running correctly by using:
```bash
kubectl get ingress ingress-reddit-app
```
5) Now It's time to test your ingress so use the command in the terminal:
```bash
curl -L domain.com/test
```

![ingress](https://github.com/user-attachments/assets/5337ea14-e37c-4dea-a914-e729409e6a56)

- The Application will RUN, You can also see the deployed application on `Ec2_ip:3000`.


**Note:-** Make sure you open the 3000 port in a security group of your Ec2 Instance.




