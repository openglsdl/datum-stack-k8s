# datum-stack-k8s

## Deploying datum and knots via kubectl

### 1. Configure your environment
- Setup storage classes
- Setup namespaces
- etc.

### 2. Configure a kustomize overlay for your environment
Note that a container image will need to be set as a minimum customization.

Datum example:
```bash
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: datum-gateway
spec:
  template:
    spec:
      containers:
        - name: datum-gateway
          image: retropex/datum-docker:latest
```

Knots example:
```bash
apiVersion: apps/v1
kind: StatefulSet
metadata:
  name: knots-bitcoind
spec:
  replicas: 1
  template:
    spec:
      containers:
        - name: knots-bitcoind
          image: retropex/docker-bitcoind:28.1
```

Also be sure to configure a volume claim template that matches your environment.

See `deploy/datum/overlays/example-dev` and `deploy/knots/overlays/example-dev` for overlay examples.

### 3. Setup your datum and knots config files
- Place datum_gatway_config.json in `deploy/datum/base/datum_gatway_config.json`
- Place bitcoin.conf in `deploy/knots/base/bitcoin.conf`

### 4. Verify that your kustomize overlay generates valid configuration
```
$ kubectl kustomize /path/to/your/overlay
```

Repeat this for both of your datum and knots overlays.

### 5. Deploy

```bash
$ kubectl create namespace YOUR_NAMESPACE
$ kubectl apply -n YOUR_NAMESPACE -k /path/to/your/knots/overlay
$ kubectl apply -n YOUR_NAMESPACE -k /path/to/your/datum/overlay
```

Depending on your environment, it may take knots hours to days to sync with the network.

### 6. Verify the deployment and services
Check the status of your StateFulSet:

```bash
$ kubectl get sts -n YOUR_NAMESPACE
```

Check the status of your pods:

```bash
$ kubectl get pods -n YOUR_NAMESPACE
```

Check that your services are created:

```bash
$ kubectl get services -N YOUR_NAMESPACE
```

### 7. Connect miners to datum

Once knots has synchronized and datum is connected via RPC, you can point miners to the sv1 port of datum-gateway-lb using the IP of any node in the cluster.