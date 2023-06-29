# Local K8s environment with Kustomization

## Pre-requirements

Install:

- [k3d](https://k3d.io/v5.4.9/#installation)
- [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)
- [Helm](https://helm.sh/docs/intro/install/)

Checkout all repositories corresponding to the projects listed in `resources:`.

## Deployment

### Create cluster

```bash
k3d cluster create taller2
```

### Create namespace

```bash
kubectl create ns fiufit
```

### Deploy postgresql

```bash
helm repo add bitnami-repo https://charts.bitnami.com/bitnami
helm install postgres-release bitnami-repo/postgresql --values overlays/postgres/values.yaml -n fiufit
```

Validate it is deployed:

```bash
kubectl get pods -n fiufit
NAME                            READY   STATUS    RESTARTS   AGE
postgres-release-postgresql-0   1/1     Running   0          12s
```

### Deploy MongoDB

```bash
helm install mongodb-release bitnami-repo/mongodb --values overlays/mongodb/values.yaml -n fiufit
```

### Deploy Redis

[Documentation](https://artifacthub.io/packages/helm/bitnami/redis)

```bash
helm install redis-release oci://registry-1.docker.io/bitnamicharts/redis --values overlays/redis/values.yaml -n fiufit
```

### Deploy

```bash
kubectl apply -k overlays/ -n fiufit
```

### Expose services

```bash
# Postgres available at localhost:5432 (user: postgres password: postgres)
kubectl port-forward svc/postgres-release-postgresql 5432:5432 -n fiufit
# MongoDB available at localhost:27017 (user: fiufit password: fiufit database: fiufit)
kubectl port-forward svc/mongodb-release 27017:27017 -n fiufit
# Users service available at localhost:8000
kubectl port-forward svc/users-service 8000 -n fiufit
```

**Note** with:

```bash
kubectl get services -n fiufit
```

You can list all services that can be exposed.

## Use local image

By default, images from docker hub will be used, you can build an image locally.
and import it to the cluster.

Build an image and import it to the cluster.  
For example, the user microservice:

```bash
docker build . --tag fiufit/users:latest
k3d image import fiufit/users:latest --cluster=taller2
```

The tag is important, it is used in the K8s deployment.

### Override deployments

In the file `kustomization.yaml` there are commented out overrides:

```yaml
# patchesJson6902:
#   - target:
#      group: apps
#      version: v1
#      kind: Deployment
#      name: auth
#    path: set_auth_deployment.yaml
#  - target:
#      group: apps
#      version: v1
#      kind: Deployment
#      name: users
#    path: set_users_deployment.yaml
```

When comments are removed the images pushed to the local cluster will be used.

## Configuration

### Set firebase credentials for trainings

Create a file named `set_trainings_firebase_credential.yaml` (it is ignored and
will not be pushed) with:

```yaml
---
apiVersion: v1
kind: ConfigMap
metadata:
  name: trainings-configuration
data:
  TRAININGS_FIREBASE_PRIVATE_KEY: "FIREBASE_PRIVATE_KEY"
```

Where `FIREBASE_PRIVATE_KEY` is the private key from `credentials.json`
downloaded from firebase.

## Useful commands

### Show containers

```bash
kubectl get pods -n fiufit
```

### Show services

```bash
kubectl get services -n fiufit
```

### Show configmap

```bash
kubectl get configmap
```

### Get config map content

```bash
kubectl describe configmap CONFIGMAP-NAME
```

### SSH to pod


```bash
kubectl exec -it <your_pod> -- /bin/bash
```

### Delete postgres release

```bash
helm uninstall postgres-release -n fiufit
# A bug https://github.com/helm/charts/issues/16251#issuecomment-564964236
# Does not delete the PVC with uninstall, run:
kubectl delete pvc data-postgres-release-postgresql-0 -n fiufit
```

### Delete deployment

```bash
kubectl delete -k overlays/ -n fiufit
```
