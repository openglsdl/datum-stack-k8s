# datum-stack-k8s

## Deploying datum and knots via kubectl

### 1. Configure your environment
- Setup storage classes
- Setup namespaces
- etc.

### 2. Configure a kustomize overlay for your environment
Note that a container image will need to be set as a minimum customization.

#### For GKE Autopilot
An example overlay is provided, customize as needed.

knots-bitcoind:
```bash
kubectl apply -k deploy/knots/overlays/example-gke/
```

datum-gateway:
```bash
kubectl apply -k deploy/datum/overlays/example-gke/
```

#### For generic environments
Create custom overlays as needed.

Datum overlay example: [deploy/datum/overlays/example-dev](deploy/datum/overlays/example-dev)
Knots overlay example: [deploy/knots/overlays/example-dev](deploy/knots/overlays/example-dev)

Also be sure to configure a volume claim template that matches your environment.

### 3. Setup your datum and knots config files
- Place your datum_gatway_config.json in `deploy/datum/base/datum_gatway_config.json`
- Place your bitcoin.conf in `deploy/knots/base/bitcoin.conf`

### 4. Verify that your kustomize overlay generates valid configuration
```
$ kubectl kustomize /path/to/your/overlay
```

Repeat this for both of your datum and knots overlays.

### 5. Deploy

#### For GKE Autopilot:

1. Add a router and enable nat if using a cluster with private nodes (see GKE Autopilot troubleshooting section)
```bash
gcloud compute routers create bitcoin-router --network=default --region=YOUR_REGION
gcloud compute routers nats create bitcoin-nat --router=bitcoin-router --region=YOUR_REGION --nat-all-subnet-ip-ranges --auto-allocate-nat-external-ips
```

2. Deploy services
```bash
kubectl create namespace YOUR_NAMESPACE
kubectl apply -n YOUR_NAMESPACE -k deploy/knots/overlays/example-gke/
kubectl apply -n YOUR_NAMESPACE -k deploy/datum/overlays/example-gke/
```

#### For generic environments:
```bash
kubectl create namespace YOUR_NAMESPACE
kubectl apply -n YOUR_NAMESPACE -k /path/to/your/knots/overlay
kubectl apply -n YOUR_NAMESPACE -k /path/to/your/datum/overlay
```

Depending on your environment, it may take knots hours to days to sync with the network.

### 6. Verify the deployment and services
Check the status of your StatefulSet:

```bash
kubectl get sts -n YOUR_NAMESPACE
```

Check the status of your pods:

```bash
kubectl get pods -n YOUR_NAMESPACE
```

Check that your services are created:

```bash
kubectl get services -n YOUR_NAMESPACE
```

### 7. Connect miners to datum

Once knots has synchronized and datum is connected via RPC, you can point miners to the sv1 port of datum-gateway-lb using the IP of any node in the cluster.

## Troubleshooting

### GKE Autopilot

#### Network Configuration Requirements

GKE Autopilot with private nodes requires additional network configuration for Bitcoin P2P connectivity:

1. **Enable Cloud NAT for outbound internet access:**
   ```bash
   # Create a Cloud Router (if not exists)
   gcloud compute routers create bitcoin-router \
       --network=default \
       --region=YOUR_REGION

   # Create Cloud NAT for outbound internet access
   gcloud compute routers nats create bitcoin-nat \
       --router=bitcoin-router \
       --region=YOUR_REGION \
       --nat-all-subnet-ip-ranges \
       --auto-allocate-nat-external-ips
   ```

2. **Configure firewall rules for Bitcoin P2P ports:**
   ```bash
   # Allow outbound Bitcoin traffic
   gcloud compute firewall-rules create allow-bitcoin-egress \
       --direction=EGRESS \
       --priority=1000 \
       --network=default \
       --action=ALLOW \
       --rules=tcp:8333,tcp:18333,tcp:18444 \
       --destination-ranges=0.0.0.0/0 \
       --target-tags=YOUR_CLUSTER_TAG
   ```

3. **Apply permissive NetworkPolicy for Bitcoin connectivity:**
   ```yaml
   apiVersion: networking.k8s.io/v1
   kind: NetworkPolicy
   metadata:
     name: knots-bitcoind-netpol
   spec:
     podSelector:
       matchLabels:
         app: knots-bitcoind
     policyTypes:
     - Egress
     egress:
     - {} # Allow all egress traffic
   ```

#### Security Context Requirements

GKE Autopilot enforces security policies. Ensure your StatefulSets include:

```yaml
spec:
  template:
    spec:
      securityContext:
        runAsNonRoot: true
        runAsUser: 1000
        runAsGroup: 1000
        fsGroup: 1000
        seccompProfile:
          type: RuntimeDefault
      containers:
        - name: your-container
          securityContext:
            allowPrivilegeEscalation: false
            runAsNonRoot: true
            runAsUser: 1000
            runAsGroup: 1000
            capabilities:
              drop:
              - ALL
```

#### Common Issues

**Bitcoin node can't connect to peers:**
- Verify Cloud NAT is configured and operational
- Check firewall rules allow Bitcoin P2P ports (8333, 18333, 18444)
- Ensure NetworkPolicy allows egress traffic

**Pods stuck in CrashLoopBackOff:**
- Check security context is properly configured for non-root execution
- Verify health probes are using TCP checks instead of process checks
- Ensure proper file permissions on mounted volumes

**StatefulSet pods not starting:**
- Check if `podManagementPolicy: Parallel` is set for faster startup
- Verify PVC storage class is available in your cluster
- Check resource requests/limits are within Autopilot constraints

**Datum-gateway can't connect to Bitcoin RPC:**
- Verify knots-bitcoind is fully synced and RPC is enabled
- Check service discovery and DNS resolution between pods
- Ensure RPC credentials match in both services

**Verification Commands:**
```bash
# Check Bitcoin peer connections
kubectl exec knots-bitcoind-0 -- bitcoin-cli getconnectioncount

# Check blockchain sync status  
kubectl exec knots-bitcoind-0 -- bitcoin-cli getblockchaininfo

# Check datum-gateway logs (last 50 lines)
kubectl logs datum-gateway-0 --tail=50

# Check Cloud NAT status
gcloud compute routers nats describe bitcoin-nat \
    --router=bitcoin-router \
    --region=YOUR_REGION
```
