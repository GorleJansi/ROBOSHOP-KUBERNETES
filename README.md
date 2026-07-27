# ROBOSHOP-KUBERNETES

Kubernetes manifests for practicing RoboShop service deployment on EKS.

This repo currently focuses on stateful backend services and their Kubernetes objects:

- `mongodb/manifest.yaml`
- `mysql/manifest.yaml`
- `rabbitmq/manifest.yaml`
- `redis/manifest.yaml`

Each service manifest generally includes:

- `Service` for stable in-cluster access
- Headless `Service` for StatefulSet pod DNS
- `StorageClass` for dynamic EBS volume provisioning
- `StatefulSet` for stable pod identity and persistent storage

## Prerequisites

Create the namespace first:

```bash
kubectl create namespace roboshop
```

For EBS-backed PVCs, the AWS EBS CSI driver must be installed and healthy:

```bash
kubectl get pods -n kube-system | grep ebs
aws eks describe-addon \
  --cluster-name roboshop-dev \
  --addon-name aws-ebs-csi-driver \
  --region us-east-1 \
  --query 'addon.status'
```

Expected add-on status:

```text
ACTIVE
```

## Apply Manifests

Apply one service:

```bash
kubectl apply -f mongodb/manifest.yaml
```

Apply all current service manifests:

```bash
kubectl apply -f mongodb/manifest.yaml
kubectl apply -f mysql/manifest.yaml
kubectl apply -f rabbitmq/manifest.yaml
kubectl apply -f redis/manifest.yaml
```

## Check Resources

```bash
kubectl get svc -n roboshop
kubectl get statefulset -n roboshop
kubectl get pods -n roboshop -o wide
kubectl get pvc -n roboshop
kubectl get pv
kubectl get storageclass
```

Describe a failing pod:

```bash
kubectl describe pod <pod-name> -n roboshop
```

Check StatefulSet-owned PVCs:

```bash
kubectl get pvc -n roboshop
```

## StatefulSet Notes

StatefulSets keep stable pod names:

```text
mongodb-statefulset-0
mongodb-statefulset-1
```

With `volumeClaimTemplates`, each pod gets its own PVC:

```text
mongodb-volume-mongodb-statefulset-0
mongodb-volume-mongodb-statefulset-1
```

Some StatefulSet fields cannot be changed after creation, including `serviceName`, `selector`, and `volumeClaimTemplates`. If those fields change during practice, delete and recreate the StatefulSet.

```bash
kubectl delete statefulset <statefulset-name> -n roboshop
kubectl apply -f <service-folder>/manifest.yaml
```

Deleting a StatefulSet does not automatically delete PVCs. Delete PVCs manually only when you intentionally want to reset storage.

## Cleanup

Delete workload resources in the namespace:

```bash
kubectl delete all --all -n roboshop
```

Delete PVCs:

```bash
kubectl delete pvc --all -n roboshop
```

Delete StorageClasses created by these manifests:

```bash
kubectl delete storageclass mongodb-storageclass mysql-storageclass rabbitmq-storageclass redis-storageclass
```

For a full namespace reset:

```bash
kubectl delete namespace roboshop
kubectl create namespace roboshop
```
