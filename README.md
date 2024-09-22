# Kubernetes on AWS - EKS

## problem description
Right after you setup the cluster on LKE or Minikube and deployed your application inside, your manager comes to you to tell you that the company also wants to run Kubernetes on AWS. Again, with less overhead when managing just one platform. So they ask you to reconfigure your cluster on AWS and deploy your application there instead.

See: https://github.com/sidor2/devops-module10-k8s.git

## proposed solution

### STEP 1: Create EKS cluster
- With eksctl you create an EKS cluster with 3 Nodes and 1 Fargate profile

```
eksctl create cluster \
  --name m11-cluster \
  --region us-west-2 \
  --nodegroup-name m11-nodegroup \
  --nodes 3 \
  --nodes-min 3 \
  --nodes-max 3 \
  --node-type t3.medium \
  --managed
```

```
eksctl create fargateprofile \
  --cluster m11-cluster \
  --name m11-fargate-profile \
  --namespace fargate-namespace
```


### STEP 2: Deploy Mysql with 2 replicas using Helm
- You deploy mysql and phpmyadmin on the EC2 nodes using the same setup as before.

- https://github.com/bitnami/charts/tree/main/bitnami%2Fmysql

```
helm repo add bitnami https://charts.bitnami.com/bitnami
helm repo update
```
```
helm install mysql bitnami/mysql -f mysql-helm/values.yaml
```

### STEP 3: Deploy phpmyadmin

```
kubectl apply -f phpmyadmin/phpmyadmin.yaml
```


### STEP 4: Deploy your Java Application with 2 replicas

- You deploy your Java application using Fargate with 3 replicas using the same setup as before.

```
kubectl apply -f java-app/secret.yaml
kubectl apply -f java-app/configmap.yaml
kubectl apply -f java-app/deployment.yaml
kubectl apply -f java-app/service.yaml
```

### STEP 5: Configure Autoscaling

- attach ClusterAutoScaler policy to the IAM role that your worker nodes use

```
aws autoscaling describe-auto-scaling-groups --query 'AutoScalingGroups[*].{Name:AutoScalingGroupName}' --output table
```

```
aws autoscaling create-or-update-tags --tags \
  Key=k8s.io/cluster-autoscaler/enabled,Value=true,PropagateAtLaunch=true,ResourceType=auto-scaling-group,ResourceId=eks-m11-nodegroup-b2c90af1-aad7-b258-5c5b-9f11570eeb38 \
  Key=k8s.io/cluster-autoscaler/m11-cluster,Value=true,PropagateAtLaunch=true,ResourceType=auto-scaling-group,ResourceId=eks-m11-nodegroup-b2c90af1-aad7-b258-5c5b-9f11570eeb38
```

```
wget https://raw.githubusercontent.com/kubernetes/autoscaler/master/cluster-autoscaler/cloudprovider/aws/examples/cluster-autoscaler-autodiscover.yaml
```

```
kubectl apply -f cluster-autoscaler-autodiscover.yaml
```

```
kubectl annotate deployment cluster-autoscaler \
    -n kube-system \
    cluster-autoscaler.kubernetes.io/safe-to-evict="false"
```
- edit the deployment container commands
```
        - --balance-similar-node-groups
        - --skip-nodes-with-system-pods=false
        image: registry.k8s.io/autoscaling/cluster-autoscaler:v1.30.0
```
