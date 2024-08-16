# devops-module10-k8s
Container Orchestration with Kubernetes

## problem description
Your company’s java-mysql application is running with docker-compose on a server. This application is used often internally and by your company clients too. You noticed that the server isn't very stable: Often a database container dies or the application itself, or docker daemon must be restarted. During this time people can't access the app!

So when this happens, the users write to you to tell you that the app is down and ask you to fix it. You SSH into the server, restart the containers with docker-compose and containers start again.

But this is annoying work, plus it doesn't look good for your company that your clients often can't access the app. So you want to make your application more reliable and highly available. You want to replicate both the database and the app, so if one container goes down, there is always a backup. Also you don't want to rely on a single server, but have multiple, in case 1 whole server goes down or gets rebooted etc.

So you look into different solutions and decide to use the container orchestration tool Kubernetes to solve the issue. For now you want to configure it and deploy your application manually, since it's a new tool and want to try it out manually before automating.

## propose solution

### STEP 1: Create a Kubernetes cluster
```
minikube start --driver=docker
```



### STEP 2: Deploy Mysql with 2 replicas using Helm
- https://github.com/bitnami/charts/tree/main/bitnami%2Fmysql

```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```
```
helm install mysql bitnami/mysql --set replicaCount=2 --set persistence.enabled=false
```
```
helm install mysql bitnami/mysql -f mysql-helm/values.yaml
```


### STEP 3: Deploy your Java Application with 2 replicas

```
kubectl apply -f java-app/secret.yaml
kubectl apply -f java-app/configmap.yaml
kubectl apply -f java-app/deployment.yaml
```


### STEP 4: Deploy phpmyadmin

```
kubectl apply -f phpmyadmin/phpmyadmin.yaml
```

### STEP 5: Deploy Ingress Controller
```
helm repo add ingress-nginx https://kubernetes.github.io/ingress-nginx
helm repo update
```
```
helm install ingress-nginx ingress-nginx/ingress-nginx 
```

### STEP 6: Create Ingress rule

> make sure the name of the service matches in the ingress.yaml and in the deployment.yaml
```
kubectl apply -f java-app/ingress.yaml
```
```
minikube addons enable ingress
```
```
minikube ip
```
```/etc/hosts
<ip> my-java-app.com
```
```
curl http://my-java-app.com
```

### STEP 7: Port-forward for phpmyadmin
1. Temporary port-forward
```
kubectl port-forward services/phpmyadmin 8080:80
```

2. Permanent - updated the phpmyadmin service to LoadBalancer type and redeployed.

3. Run a DaemonSet for Port Forwarding (not implemented yet)

### STEP 8: Create Helm Chart for Java App

```
helm install mysql bitnami/mysql -f mysql-helm/values.yaml
```
```
helm install ingress-nginx ingress-nginx/ingress-nginx 
```
```
helm install java-app java-app-helm
```
```
minikube ip
```
```/etc/hosts
<ip> my-java-app.com
```
```
curl http://my-java-app.com
```