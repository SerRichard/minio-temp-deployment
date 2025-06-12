# minio-temp-deployment
Just a place to hold the yamls and steps for spinning up a short term minio instance which can be used to mirror data, where data exists in disks from a previous deployment.

This assumes 2 disks from a previous deployment need to be mirror'd into a new deployment. Check the order of the old disks by checking the set order found at:
```
# Assuming moumt paths as minio-1 and minio-2
cat minio-1/.minio.sys/format.json
cat minio-2/.minio.sys/format.json
```

Ensure the mount paths are ordered in their set order, to the order of the persistent volumes. So if minio-2 actually is first in the set order, it wants to be the local path
for the old-pv-1, not old-pv-2.

### Install using helm

```
helm repo add minio https://charts.min.io/
helm repo update
```

```
kubectl create ns minio-tmp
helm install minio-old minio/minio -n minio-tmp -f minio-values.yaml
```

### Mirror old data to another deployment

Assuming we have a new minio service in the cluster as minio-sc. We need to register the two alias's, and then we can mirror the data.

```
mc alias set oldminio http://minio-old.minio-tmp.svc.cluster.local:9000 ACCESS_KEY ACCESS_SECRET
mc alias set newminio http://minio-svc.monitoring.svc.cluster.local:9000 ACCESS_KEY ACCESS_SECRET
```

Mirror the bucket called 'chunks', from the old deployment to the new deployment.
```
mc mirror oldminio/chunks newminio/chunks
```

### Uninstall

```
helm uninstall minio-old -n minio-tmp
```
