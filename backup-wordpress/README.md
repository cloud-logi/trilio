# Instalacion de Trilio 
## Instalamos los drivers de HostPath CSI
## https://docs.trilio.io/kubernetes/appendix/csi-drivers/hostpath-for-tvk

Clonar el repositorio hostpath GitHub
```
git clone https://github.com/kubernetes-csi/csi-driver-host-path.git
```

Segun la version del cluster ir al directorio de instalacion
```
# cd csi-driver-host-path/deploy/kubernetes-1.19
```

Omita este paso si los crds de volumenapshot, volumenapshotcontent, volumenapshotclass ya están presentes en el clúster
```
# Change to the latest supported snapshotter version
$ SNAPSHOTTER_VERSION=v2.0.1

# Apply VolumeSnapshot CRDs
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/${SNAPSHOTTER_VERSION}/config/crd/snapshot.storage.k8s.io_volumesnapshotclasses.yaml
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/${SNAPSHOTTER_VERSION}/config/crd/snapshot.storage.k8s.io_volumesnapshotcontents.yaml
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/${SNAPSHOTTER_VERSION}/config/crd/snapshot.storage.k8s.io_volumesnapshots.yaml

# Create snapshot controller
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/${SNAPSHOTTER_VERSION}/deploy/kubernetes/snapshot-controller/rbac-snapshot-controller.yaml
$ kubectl apply -f https://raw.githubusercontent.com/kubernetes-csi/external-snapshotter/${SNAPSHOTTER_VERSION}/deploy/kubernetes/snapshot-controller/setup-snapshot-controller.yaml
```

Asegurarse que nos encontramos en el proyecto "default" y ejecutar el script de instalacion
```
# ./deploy.sh
```

Chequear que esten corriendo los pods
```
# oc get pod 
NAME                    READY   STATUS    RESTARTS   AGE
csi-hostpath-socat-0    1/1     Running   0          6d22h
csi-hostpathplugin-0    9/9     Running   0          6d22h
snapshot-controller-0   1/1     Running   0          6d22h
``` 

Crear un Storage Class para Hostpath
```
# oc create -f install/storageclass.yaml
```

Crear un VolumeSnapshot Class para Hostpath
```
# oc create -f install/volumesnapshotclass.yaml
```

Crear un volumen de test
```
# oc create -f install/pv-test.yaml
```

Crear un snapshot para task-pv-claim
```
oc create -f install/snap-vpc-test.yaml
```

Verifique si el snapshot y el contenido de la instantánea del volumen muestra la instantánea creada.Verifique si la instantánea y el contenido de la instantánea del volumen muestra la instantánea creada.
```
oc get volumesnapshot
oc get volumesnapshotcontent
```

# Se necesita para este caso un servidor NFS.
Creamos el Target
```
# oc create -f nfstarget.yaml
targets.triliovault.trilio.io/sample-target created
```
Creamos el Retention Policy
```
# oc create -f Policy.yaml 
policy.triliovault.trilio.io/demo-policy created
```

Crear app de Test
```
# oc create -f app-demo-trilio.yaml 
secret/mysql-pass created
persistentvolumeclaim/mysql-pv-claim created
deployment.apps/k8s-demo-app-mysql created
service/k8s-demo-app-mysql created
deployment.apps/k8s-demo-app-frontend created
service/k8s-demo-app-frontend created
```

Crear Backup Plan
```
# oc create -f BackupPlan.yaml 
backupplan.triliovault.trilio.io/mysql-label-backupplan created
```

Crear Backup 
```
# oc create -f backup.yaml 
backup.triliovault.trilio.io/mysql-label-backup created
```

Verificar el porcentaje del Backup
```
# oc get backups
NAME                 BACKUPPLAN               BACKUP TYPE   STATUS       DATA SIZE   START TIME             END TIME   PERCENTAGE COMPLETED   BACKUP SCOPE
mysql-label-backup   mysql-label-backupplan   Full          InProgress   0           2021-05-28T15:28:42Z              20                     App
```

Backup completo cuando llega a 100%
```
# oc get backups
NAME                 BACKUPPLAN               BACKUP TYPE   STATUS      DATA SIZE   START TIME             END TIME               PERCENTAGE COMPLETED   BACKUP SCOPE
mysql-label-backup   mysql-label-backupplan   Full          Available   261033984   2021-05-28T15:28:42Z   2021-05-28T15:31:35Z   100                    App

```

Con watch monitoreamos el proceso
```
# watch 'oc get backups.triliovault.trilio.io '
```

Para restorear primero borramos la app.
```
# oc delete -f app-demo-trilio.yaml 
secret "mysql-pass" deleted
persistentvolumeclaim "mysql-pv-claim" deleted
deployment.apps "k8s-demo-app-mysql" deleted
service "k8s-demo-app-mysql" deleted
deployment.apps "k8s-demo-app-frontend" deleted
service "k8s-demo-app-frontend" deleted
```

Hacemos el restore de la app
```
# oc create -f restore.yaml 
restore.triliovault.trilio.io/demo-restore created
```
Monitoreamos el restore 
```
# watch 'oc get restore '
demo-restore   mysql-label-backup   Completed   120869458   2021-05-28T15:34:42Z   2021-05-28T15:36:40Z   100                    default             App
```



