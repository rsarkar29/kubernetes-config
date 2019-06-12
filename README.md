![alt text](https://kubernetes.io/images/kubernetes-horizontal-color.png)

# How to easily deploy a Drupal instance on Kubernetes using YAML

## Preparing the data persistence infrastructure

Since Kubernetes containers are not data-persistent, it is necessary to use volumes in order to preserve the data. Otherwise, all the changes written to the database or to any directory will be lost after the container restarts.

In Kubernetes it is recommended to use the PersistentVolume object to retain the data. A PersistentVolume is a representation of persistent storage. It supports many underlying technologies and storage services: NFS, Cinder, Gluster, Ceph, AWS EBS, Google Persistent Disk,…

It is possible to create a PersistentVolume either statically or dynamically. In this article we chose a dynamically-created PersistentVolume since it is easier to manage.

The way to do that is to create a PersistentVolumeClaim. local-volume.yaml

```
apiVersion: v1
kind: PersistentVolume
metadata:
  name: mysql-pv
  labels:
    type: local
spec:
  storageClassName: manual
  capacity:
    storage: 5Gi
  accessModes:
    - ReadWriteOnce
  hostPath:
    path: /tmp/data/lv-1
  persistentVolumeReclaimPolicy: Recycle
```

To apply this configuration to kubernetes Cluster

```
kubectl apply -f local-volume.yaml
```
To check persistentVolume

```
kubectl get pv
```

## PersistentVolume

To claim persistentVolume.
```
.
.
.
volumes:
      - name: drupal-local-storage
        persistentVolumeClaim:
          claimName: drupal-lv-claim
```

To check persistentVolumeClaim

```
kubectl get pvc
```

## Deploying a MySQL instance

To deploy the MySQL instance we will use a Deployment and a Service. Note that we run MySQL from root. When you run it in production, use custom credentials. It is also a good practice to keep MySQL password in a secret, not in the yaml.

```
kubectl apply -f mysql-deployment.yaml
```

## Deploying Drupal Application

The Deployment will be based on a container with a Drupal image from Docker Hub repository. In addition, we will also use a temporary container (InitContainer) whose mission will be to pre-populate the persistent storage with the data used by Drupal

```
kubectl apply -f drupal-deployment.yaml
```

## References

https://medium.com/containerum/how-to-easily-deploy-a-drupal-8-instance-on-kubernetes-b90acc7786b7
