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

### Use local image

Build an image and import it to the cluster.  
For example, the user microservice:

```bash
docker build . --tag fiufit/users:latest
k3d image import fiufit/users:latest --cluster=taller2
```

The tag is important, it is used in the K8s deployment.

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

## Misc

### Useful commands

```bash
kubectl get pods -n fiufit
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

