# datum-stack-k8s

## Deploying datum and knots via kubectl

### 1. Configure your environment
- Setup storage classes
- Setup namespaces
- etc.

### 2. Configure a kustomize overlay for your environment
Note that an image will need to be set as a minimum customization.

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

See `deploy/datum/overlays/dev` and `deploy/knots/overlays/dev` for a full overlay example.

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
$ kubectl apply -n YOUR_NAMESPACE -k /path/to/your/overlay
```

Repeat this for both of your datum and knots overlays.

### 6. Verify the Deployments and Services
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