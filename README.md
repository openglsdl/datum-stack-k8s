# datum-stack-k8s

## Deploying datum and knots-bitcoind via kubectl

### 1. Copy Manifests to Your MicroK8s Server
Ensure the following files are available on your MicroK8s server:
- `deployments/datum-gateway.yaml`
- `deployments/knots-bitcoind.yaml`

### 2. Apply the Manifests
From the project root, run:
```bash
microk8s kubectl apply -f knots-bitcoind/knots-bitcoind-deployment.yaml
microk8s kubectl apply -f datum/datum-gateway-deployment.yaml
```

### 3. Verify the Deployments and Services
Check that the pods are running:

```bash
microk8s kubectl get pods
```

Check that the services are created:

```bash
microk8s kubectl get services
```

### 4. (Optional) Expose Services Externally
If you want to access the services from outside the cluster, you may need to edit the `Service` type from `ClusterIP` to `NodePort` or `LoadBalancer` in the manifests.

### 5. (Optional) Provide Configuration
If `datum_gateway` requires a custom config file, mount it as a Kubernetes ConfigMap or volume and update the deployment manifest accordingly.

## Deploying to Google Cloud Platform (GCP) with GKE

### 1. Create a GKE Cluster
Follow the official GCP documentation to create your desired type of Kubernetes cluster:
- https://cloud.google.com/kubernetes-engine/docs/how-to/creating-a-zonal-cluster
- https://cloud.google.com/kubernetes-engine/docs/how-to/creating-a-regional-cluster

Note that instructions below are shown as a simple zonal deployment example.

### 2. Configure kubectl to Use Your GKE Cluster
```bash
gcloud container clusters get-credentials datum-cluster --zone us-central1-a
```

### 3. Apply the Manifests
From the project root, run:
```bash
kubectl apply -f knots-bitcoind/knots-bitcoind-deployment.yaml
kubectl apply -f datum/datum-gateway-deployment.yaml
```

### 4. Expose Services Externally
Edit the `Service` manifests to use `type: LoadBalancer` for external access. Example:
```yaml
spec:
  type: LoadBalancer
  ports:
    # ...
```
Re-apply the service manifests if you make changes:
```bash
kubectl apply -f knots-bitcoind/knots-bitcoind-deployment.yaml
kubectl apply -f datum/datum-gateway-deployment.yaml
```

### 5. Check External IPs
After a few minutes, get the external IPs:
```bash
kubectl get services
```

### 6. (Optional) Use Google Secret Manager or ConfigMaps
For sensitive configuration, use GCP Secret Manager or Kubernetes ConfigMaps/Secrets and mount them into your pods.

### 7. Clean Up
To delete the cluster and all resources:
```bash
gcloud container clusters delete datum-cluster --zone us-central1-a
```

## Deploying to AWS with EKS

### 1. Create an EKS Cluster
Follow the official AWS documentation or use eksctl:
https://docs.aws.amazon.com/eks/latest/userguide/create-cluster.html

Example with eksctl:
```bash
eksctl create cluster --name datum-cluster --region us-west-2 --nodes 3
```

### 2. Configure kubectl for EKS
```bash
aws eks --region us-west-2 update-kubeconfig --name datum-cluster
```

### 3. Apply the Manifests
```bash
kubectl apply -f knots-bitcoind/knots-bitcoind-deployment.yaml
kubectl apply -f datum/datum-gateway-deployment.yaml
```

### 4. Expose Services Externally
Edit the `Service` manifests to use `type: LoadBalancer` for external access. Then re-apply the manifests.

### 5. Get External IPs
```bash
kubectl get services
```

### 6. (Optional) Use AWS Secrets Manager or ConfigMaps
For sensitive configuration, use AWS Secrets Manager or Kubernetes ConfigMaps/Secrets and mount them into your pods.

### 7. Clean Up
```bash
eksctl delete cluster --name datum-cluster --region us-west-2
```

## Deploying to Azure with AKS

### 1. Create an AKS Cluster
Follow the official Azure documentation:
https://docs.microsoft.com/en-us/azure/aks/kubernetes-walkthrough

Example command:
```bash
az aks create --resource-group myResourceGroup --name datum-cluster --node-count 3 --generate-ssh-keys
```

### 2. Configure kubectl for AKS
```bash
az aks get-credentials --resource-group myResourceGroup --name datum-cluster
```

### 3. Apply the Manifests
```bash
kubectl apply -f knots-bitcoind/knots-bitcoind-deployment.yaml
kubectl apply -f datum/datum-gateway-deployment.yaml
```

### 4. Expose Services Externally
Edit the `Service` manifests to use `type: LoadBalancer` for external access. Then re-apply the manifests.

### 5. Get External IPs
```bash
kubectl get services
```

### 6. (Optional) Use Azure Key Vault or ConfigMaps
For sensitive configuration, use Azure Key Vault or Kubernetes ConfigMaps/Secrets and mount them into your pods.

### 7. Clean Up
```bash
az aks delete --resource-group myResourceGroup --name datum-cluster --yes
```

