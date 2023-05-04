# Local K8s environment with Kustomization

## Pre-requirements

Install:

- [k3d](https://k3d.io/v5.4.9/#installation)
- [Kubectl](https://kubernetes.io/docs/tasks/tools/install-kubectl-linux/)
- [Helm](https://helm.sh/docs/intro/install/)

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

### Deploy

```bash
kubectl apply -k overlays/ -n fiufit
```

### Expose services

```bash
# Database available at localhost:5432 (user: postgres password: postgres)
kubectl port-forward svc/postgres-release-postgresql 5432:5432 -n fiufit
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

In the file `kustomization.yaml` there are two commented out overrides:

```yaml
patchesJson6902:
  - target:
     group: apps
     version: v1
     kind: Deployment
     name: auth
   path: set_auth_deployment.yaml
 - target:
     group: apps
     version: v1
     kind: Deployment
     name: users
   path: set_users_deployment.yaml
```

When comments are removed the images pushed to the local cluster will be used.

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
