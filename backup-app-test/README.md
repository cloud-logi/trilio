# Backup de Trilio con app-demo
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



