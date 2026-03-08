https://kubernetes.io/ru/docs/tasks/tools/install-minikube/

minikube start --driver=docker --cpus=4 --memory=16g

minikube status

minikube addons enable ingress

minikube dashboard

https://kubernetes.io/ru/docs/tasks/tools/install-kubectl/

kubectl version


kubectl create ns notes