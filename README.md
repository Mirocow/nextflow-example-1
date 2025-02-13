### Запуск Nextflow + Wave с помощью Kubernetes

#### Проблема: при запуске Nextflow pipeline с помощью Kubernetes возникает ошибка "Process terminated for an unknown reason"

#### Настройки запуска:

Настройки kubernetes:  
Для успешного запуска необходимо создать Persistent Volume Claime (PVC) с режимом доступа ReadWriteMany и указать Storage Class с RWX правами, также необходимо создать Service Account.

#### nextflow-pvc.yaml
```
apiVersion: v1
kind: PersistentVolumeClaim
metadata:
  name: nextflow-pvc
  namespace: nextflow
spec:
  accessModes:
  - ReadWriteMany
  resources:
    requests:
      storage: 20Gi
  storageClassName: nfs-rwx
```
#### nfs-rwx-storageclass.yaml

```yaml
allowVolumeExpansion: true
apiVersion: storage.k8s.io/v1
kind: StorageClass
metadata:
  annotations:
    meta.helm.sh/release-name: nfs-provisioner
    meta.helm.sh/release-namespace: default
  creationTimestamp: "2025-02-06T08:25:50Z"
  labels:
    app: nfs-subdir-external-provisioner
    app.kubernetes.io/managed-by: Helm
    chart: nfs-subdir-external-provisioner-4.0.18
    heritage: Helm
    release: nfs-provisioner
  name: nfs-rwx
  resourceVersion: "7814"
  uid: 527ce739-9533-4ba0-ad94-042dc30712e3
parameters:
  archiveOnDelete: "true"
provisioner: cluster.local/nfs-provisioner-nfs-subdir-external-provisioner
reclaimPolicy: Delete
volumeBindingMode: Immediate
```

#### nextflow-sa.yaml
```yaml
apiVersion: v1
kind: ServiceAccount
metadata:
  annotations:
    kubectl.kubernetes.io/last-applied-configuration: |
      {"apiVersion":"v1","kind":"ServiceAccount","metadata":{"annotations":{},"name":"nextflow-sa","namespace":"nextflow"}}
  creationTimestamp: "2025-02-06T08:45:38Z"
  name: nextflow-sa
  namespace: nextflow
  resourceVersion: "13164"
  uid: 53ec03bf-3cae-4a95-b74f-5c4662ec03d9
```

#### Запуск

```bash
$ nextflow kuberun https://github.com/Mirocow/nextflow-example-1 -r main -head-image 'nextflow/nextflow:22.10.8' -v nextflow-pvc:/mnt -profile kubernetes --outdir /tmp
```

### Запуск Nextflow + Wave с помощью Docker

#### Запуск

```bash
$ nextflow run main.nf -profile docker --outdir /tmp
```
