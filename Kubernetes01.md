# Kubernetes Setup

## First time setup

>> GOpen the below link:

https://minikube.sigs.k8s.io/docs/start/


`curl -LO https://storage.googleapis.com/minikube/releases/latest/minikube-linux-amd64`

``sudo install minikube-linux-amd64 /usr/local/bin/minikube && rm minikube-linux-amd64``

```sudo usermod -aG docker $USER && newgrp docker```

```minikube start -- driver=docker```

```sudo snap install kubectl --classic```

```kubectl get pods -A```

## Docker containerisation and run

```git clone https://github.com/LondheShubham153/django-todo-cicd.git```

```sudo docker build -t kubernetes/test:v1 .```

```sudo docker run -p 8000:8000 <image id>``` 


## Pushing on Docker Hub

For isolation create a directoy

```mkdir k8s`` 

```cd k8s```

Push to docker hub repository (We can push on any repository nexus, artifactory etc)

Create a repository on a docker hub. In my case it is kubernetes_test


```docker tag kubernetes/test:v1 krishnasachin/kubernetes_test:v1```

```docker push krishnasachin/kubernetes_test:v1```

Can Pull the images from docker hub to local or any server ec2

```docker pull krishnasachin/kubernetes_test:v1``

## Kubernetes pods

Create pod yaml file

```vim pod.yaml```

Open below link and copy this

https://kubernetes.io/docs/concepts/workloads/pods/


```
apiVersion: v1
kind: Pod
metadata:
  name: k8s-test
spec:
  containers:
  - name: k8s-test
    image: kubernetes/test
    ports:
    - containerPort: 8000
```

Befor running below command make sure minikube is running

```kubectl apply -f pod.yaml```

```kubectl get pods```

To run the application use wide detail and get ip

```kubectl get pods -o wide```

Then

```curl -L http://10.244.0.4:8000```


Let's we deleted the pods or any crash happened. 

```kubectl delete pod k8s-test```

No pods is there then how k8s is handling differently from docker. 

```kubectl get pods```

Let see...

We are not deploying pods in production. We are using Kubernetes deployment for production.

Click on the below link

https://kubernetes.io/docs/concepts/workloads/controllers/deployment/

```vim deploy.yaml```

```
apiVersion: apps/v1
kind: Deployment
metadata:
  name: k8s-test
  labels:
    app: nginx
spec:
  replicas: 3
  selector:
    matchLabels:
      app: nginx
  template:
    metadata:
      labels:
        app: nginx
    spec:
      containers:
      - name: k8s-test
        image: kubernetes/test
        ports:
        - containerPort: 8000
```

```kubectl apply -f deploy.yaml```

```kubectl get pods```

Now delete pods and check: 

```kubectl delete pods k8s-test-589dd657f9-7hngw```

New Pods is auto created. This process is called auto healing

```kubectl get pods```

How we can access it from outside of hosted server:

1) First is tunnling but for production it is not

2) For production use Kubernetes service 

Part1: Manage network and service

```vim service.yaml```

```
apiVersion: v1
kind: Service
metadata:
  name: my-service
spec:
  type: NodePort
  selector:
    app: MyApp
  ports:
    - protocol: TCP
      port: 80
      targetPort: 8000
      nodePort: 30007
```

Create service pods:

```kubectl apply -f service.yaml```

Get service detail

```kubectl get svc```

Get public ip of the service from:

```minikube profile list```


Access from public ip:

```curl -L http://192.168.49.2:30007```


Part 2: Manage host ip:

in etc/hosts map the public ip with any domain.

```nano etc\hosts```

```
192.168.49.2 infytechai.com
```

```curl -L infytechai.com:30007```