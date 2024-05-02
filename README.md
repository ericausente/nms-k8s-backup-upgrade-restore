# nms-upgrade

Personal note on approach to upgrading the NGINX Management Suite (NMS) and NGINX Agent within a Kubernetes environment

## Prerequisites

Before proceeding with the upgrade, ensure the following conditions are met:

- Confirm root access to the Kubernetes cluster where NMS is installed.
- Helm 3.10.0 or later 	https://helm.sh/docs/intro/install/
-  Verify the installed versions of NGINX Management Suite and Instance Manager.
- SQLite Configuration: Check that SQLite is installed and properly configured according to the NGINX documentation https://docs.nginx.com/nginx-management-suite/admin-guides/maintenance/sqlite-installation/.
-  Ensure Helm is installed and configured for your Kubernetes cluster.
- Documentation Review: Familiarize yourself with the NMS documentation (https://docs.nginx.com/nginx-management-suite/installation/kubernetes/deploy-instance-manager/) and release notes (https://docs.nginx.com/nginx-management-suite/nim/releases/release-notes/) for version-specific instructions or changes.


## Upgrade Process

Download the Upgrade Package
1. Add the NGINX stable Helm repository and update it:
```
helm repo add nginx-stable https://helm.nginx.com/stable
helm repo update
```
2. Pull the NGINX Management Suite Helm chart for the new version:
```
helm pull nginx-stable/nms --version <new_version>
tar zxvf nms-<version>.tgz
```
3. Navigate to the Backup Directory and Execute the Backup Script:
```
cd /nms/charts/nms-hybrid/backup-restore
./k8s-backup.sh
```

Sample Output: 
```
# sudo KUBECONFIG=~/.kube/config ./k8s-backup.sh
--- NGINX Management Suite k8s backup script ---

####
#### Parse arguments:
####
Using tmp directory: /tmp/k8s-backup-1710847841
Creating dirs: /tmp/k8s-backup-1710847841
####
#### Query user for info
####
Please provide the NGINX Instance Manager Helm namespace:
nms
####
#### Collect NMS info
####
collecting k8s configmaps
Creating dirs: /tmp/k8s-backup-1710847841/configmaps
collecting k8s secrets
Creating dirs: /tmp/k8s-backup-1710847841/secrets
pod information collected
collecting core secrets
Creating dirs: /tmp/k8s-backup-1710847841/core-secrets
tar: Removing leading `/' from member names
Creating dirs: /tmp/k8s-backup-1710847841/dqlite/core
####
#### Dump dqlite: core
####
nms-core database address is 0.0.0.0:7891
Collecting nms-core DQlite information...
Defaulted container "core" out of: core, change-mount-ownership (init)
flag provided but not defined: -n
Usage of /etc/nms/scripts/dqlite-backup:
  -address string
        DB address
  -config string
        config file (default is /etc/nms/nms.conf) (default "/etc/nms/nms.conf")
  -name string
        DB name (default "dpm")
  -output string
        output directory for Dqlite data (default "/tmp/dqlite/")
command terminated with exit code 2
Defaulted container "core" out of: core, change-mount-ownership (init)
rm: cannot remove '/etc/nms/scripts/core': No such file or directory
command terminated with exit code 1
nms-core DQlite information collected
Creating dirs: /tmp/k8s-backup-1710847841/dqlite/dpm
####
#### Dump dqlite: dpm
####
nms-dpm database address is 0.0.0.0:7890
Collecting nms-dpm DQlite information...
Defaulted container "dpm" out of: dpm, change-mount-ownership (init)
flag provided but not defined: -n
Usage of /etc/nms/scripts/dqlite-backup:
  -address string
        DB address
  -config string
        config file (default is /etc/nms/nms.conf) (default "/etc/nms/nms.conf")
  -name string
        DB name (default "dpm")
  -output string
        output directory for Dqlite data (default "/tmp/dqlite/")
command terminated with exit code 2
tar: Removing leading `/' from member names
tar: /etc/nms/scripts/dpm: Cannot stat: No such file or directory
tar: Exiting with failure status due to previous errors
Defaulted container "dpm" out of: dpm, change-mount-ownership (init)
rm: cannot remove '/etc/nms/scripts/dpm': No such file or directory
command terminated with exit code 1
nms-dpm DQlite information collected
Creating dirs: /tmp/k8s-backup-1710847841/dqlite/integrations
####
#### Dump dqlite: integrations
####
nms-integrations database address is 0.0.0.0:7892
Collecting nms-integrations DQlite information...
Defaulted container "integrations" out of: integrations, change-mount-ownership (init)
flag provided but not defined: -n
Usage of /etc/nms/scripts/dqlite-backup:
  -address string
        DB address
  -config string
        config file (default is /etc/nms/nms.conf) (default "/etc/nms/nms.conf")
  -name string
        DB name (default "dpm")
  -output string
        output directory for Dqlite data (default "/tmp/dqlite/")
command terminated with exit code 2
tar: Removing leading `/' from member names
tar: /etc/nms/scripts/integrations: Cannot stat: No such file or directory
tar: Exiting with failure status due to previous errors
Defaulted container "integrations" out of: integrations, change-mount-ownership (init)
rm: cannot remove '/etc/nms/scripts/integrations': No such file or directory
command terminated with exit code 1
nms-integrations DQlite information collected
Creating dirs: /tmp/k8s-backup-1710847841/dqlite/acm
####
#### Dump dqlite: acm
####
nms-acm database address is 0.0.0.0:9300
WARNING: /etc/nms/scripts/dqlite-backup is not available in pod/acm-5956f9cd8b-ncnjr. Skipping...
Creating dirs: /tmp/k8s-backup-1710847841/dqlite/license
####
#### Dump dqlite: license
####
nms-license database address is 0.0.0.0:7893
Collecting nms-license DQlite information...
Defaulted container "integrations" out of: integrations, change-mount-ownership (init)
flag provided but not defined: -n
Usage of /etc/nms/scripts/dqlite-backup:
  -address string
        DB address
  -config string
        config file (default is /etc/nms/nms.conf) (default "/etc/nms/nms.conf")
  -name string
        DB name (default "dpm")
  -output string
        output directory for Dqlite data (default "/tmp/dqlite/")
command terminated with exit code 2
Defaulted container "integrations" out of: integrations, change-mount-ownership (init)
rm: cannot remove '/etc/nms/scripts/license': No such file or directory
command terminated with exit code 1
nms-license DQlite information collected
####
#### Package output
####
Archive /mnt/c/Users/ausente/Downloads/nms/charts/nms-hybrid/backup-restore/k8s-backup-1710847841.tar.gz ready
```

4. Use Helm to upgrade the NGINX Management Suite deployment, specifying the path to your values.yaml file and setting any necessary parameters, such as the admin password hash.

To upgrade Instance Manager, take the following steps:
- A. Download Helm Package  -  Go to the MyF5 website, then select Resources > Downloads.
- B. Load Docker Images     - https://github.com/ericausente/nms-acm-on-k8s-via-helm/blob/main/docker-tag-push.sh
- C. Push Images to Private Docker Registry - - https://github.com/ericausente/nms-acm-on-k8s-via-helm/blob/main/docker-tag-push.sh
- D. Prepare your yaml file (Configure values.yaml file to pull from your private Docker registry.)

```
# values.yaml
global:
nms-hybrid:
    imagePullSecrets:
        - name: regcred
    apigw:
        image:
            repository: ausente/nms-apigw
            tag: 2.15.1
    core:
        image:
            repository: ausente/nms-core
            tag: 2.15.1
    dpm:
        image:
            repository: ausente/nms-dpm
            tag: 2.15.1
    ingestion:
        image:
            repository: ausente/nms-ingestion
            tag: 2.15.1
    integrations:
        image:
            repository: ausente/nms-integrations
            tag: 2.15.1
```
Important:
Make sure to copy and save the password for future reference. Only the encrypted password is stored in Kubernetes. There’s no way to recover or reset a lost password.
(Optional) Replace <nms-chart-version> with the desired version; see the table below for the available versions. Alternatively, you can omit this flag to install the latest version.
    
5. Failed Scenario and Rollback (I forgot to specify the Loadbalancer Type of Service)
```
helm upgrade -n nms --set nms-hybrid.adminPasswordHash=$(openssl passwd -1 'YourPassword123#') -f values.yaml  nms nginx-stable/nms --version 1.12.0 --wait
```

If the upgrade fails:
Identify the release name and revision number for rollback.

```
helm list -n nms
helm history nms -n nms
```

Rollback to a previous stable revision.
```
helm rollback nms <revision> -n nms
```

Sample output: 
```
eric [ ~ ]$ helm status nms -n nms
NAME: nms
LAST DEPLOYED: Sun Mar 24 00:08:45 2024
NAMESPACE: nms
STATUS: pending-upgrade
REVISION: 2
TEST SUITE: None
eric [ ~ ]$ 

eric [ ~ ]$ helm rollback nms -n nms 1
Rollback was a success! Happy Helming!
```

After the rollback, it's a good practice to verify that the rollback was successful and the release is now running the expected version. You can do this by running:

    helm status RELEASE_NAME to check the current state and version of your release.
    helm history RELEASE_NAME to see the updated history and ensure the rollback appears as the latest action.

Additional Options:

    If you're not sure which revision to roll back to, use helm history RELEASE_NAME to review the revisions and their statuses.
    Helm also allows for additional flags like --timeout to specify the time to wait for the rollback to complete and --force to force resource update through delete/recreate if needed.

6. Successful Scenario:
```
# helm upgrade -n nms --version 1.12.0 --set nms-hybrid.adminPasswordHash=$(openssl passwd -1 'YourPassword123#') -f values.yaml nms nginx-stable/nms --set nms-hybrid.apigw.service.type=LoadBalancer --wait

$ kubectl get pods -n nms
NAME                            READY   STATUS                  RESTARTS   AGE
acm-5956f9cd8b-ht6bs            0/1     ImagePullBackOff        0          7m18s
adm-678cb6b487-mzk64            0/1     Init:ImagePullBackOff   0          7m17s
apigw-fbdc7dbb6-2gspj           1/1     Running                 0          34s
clickhouse-654cc8d5d7-vk7h8     1/1     Running                 0          28s
core-c8656676-mxfh9             1/1     Running                 0          32s
dpm-75fb57c8b9-l95wh            1/1     Running                 0          31s
ingestion-7d44d65f86-kpqlw      1/1     Running                 0          31s
integrations-598b97cc76-8zs25   1/1     Running                 0          29s

# kubectl get svc -n nms
NAME           TYPE           CLUSTER-IP     EXTERNAL-IP    PORT(S)                      AGE
acm            ClusterIP      10.0.206.118   <none>         8037/TCP                     10m
adm            ClusterIP      10.0.103.213   <none>         8039/TCP                     10m
apigw          LoadBalancer   10.0.41.44     4.246.52.142   443:31492/TCP                4d6h
clickhouse     ClusterIP      10.0.195.64    <none>         9000/TCP,8123/TCP            4d6h
core           ClusterIP      10.0.249.9     <none>         8033/TCP,8038/TCP,7891/TCP   4d6h
dpm            ClusterIP      10.0.74.74     <none>         8034/TCP,8036/TCP,9100/TCP   4d6h
ingestion      ClusterIP      10.0.194.159   <none>         8035/TCP                     4d6h
integrations   ClusterIP      10.0.237.189   <none>         8037/TCP                     4d6h

```

8. Upgrade NGINX Agent



# Using the Restoration Script in AKS

Utility pod has to be scheduled on the nodes where the PVs which the NMS Components are in use.  
Given the error message from the initial kubectl describe pod command indicating a "volume node affinity conflict" (for our utility pod), I’ve identified a requirement to adjust the PV and PVCs used. 
It appears we need to ensure all necessary PVs are accessible within a single Azure zone where the node resides. 
This task presents challenges, especially if the data on these PVs is actively in use and tied to specific zones.

Please note that this situation isn't managed by NGINX but falls under the purview of AKS maintainers. 

As a best effort, in my lab tests and through insights gathered from Azure documentation and community forums, I found that creating a new Storage Class specific to a single zone (westus2-2 in my case) was a necessary step. 
Below is the configuration used for the Storage Class, named single-zone-storage-class: 
```
# cat sc.yaml
kind: StorageClass
apiVersion: storage.k8s.io/v1
metadata:
  name: single-zone-storage-class
provisioner: kubernetes.io/azure-disk
parameters:
  skuName: Premium_LRS
  location: westus2
  # Set this to true for single-zone or remove for multi-zone
  zoned: "true"
reclaimPolicy: Retain
volumeBindingMode: WaitForFirstConsumer
allowedTopologies:
- matchLabelExpressions:
  - key: failure-domain.beta.kubernetes.io/zone
    values:
    - westus2-2
```


I applied the above Storage Class and adjusted our helm deployment to utilize it.
This ensured that our utility pod was scheduled correctly within the westus2-2 zone: 

Here's the the values used for my helm install, specifically highlighting the storage class adjustment for some components:
```
# cat values.yaml
# values.yaml
global:
    nmsModules:
        nms-acm:
            enabled: false
            configs: []
            services: []
        nms-adm:
            enabled: false
            configs: []
            services: []
nms-hybrid:
    imagePullSecrets:
        - name: regcred
    apigw:
        image:
            repository: ausente/nms-apigw
            tag: 2.12.0
    core:
        image:
            repository: ausente/nms-core
            tag: 2.12.0
        persistence:
          enabled: true
          storageClass: single-zone-storage-class
    dpm:
        image:
            repository: ausente/nms-dpm
            tag: 2.12.0
        persistence:
          enabled: true
          storageClass: single-zone-storage-class
    ingestion:
        image:
            repository: ausente/nms-ingestion
            tag: 2.12.0
    integrations:
        image:
            repository: ausente/nms-integrations
            tag: 2.12.0
        persistence:
          enabled: true
          storageClass: single-zone-storage-class
```

After creating some dummy user roles, users, and instance groups within NMS, I initiated a backup using my script. 
NOTE: Instead of proceeding with the usual uninstall-install cycle, I opted to delete the newly created resources directly from the UI to hasten the process.

Subsequently, I performed a helm upgrade with the below values to ensure the deployment of the utility pod, as shown by the following kubectl command output  below: 

```
# cat values_sc.yaml
# values.yaml
global:
    utility: true
    nmsModules:
        nms-acm:
            enabled: false
            configs: []
            services: []
        nms-adm:
            enabled: false
            configs: []
            services: []
nms-hybrid:
    imagePullSecrets:
        - name: regcred
    apigw:
        image:
            repository: ausente/nms-apigw
            tag: 2.12.0
    core:
        image:
            repository: ausente/nms-core
            tag: 2.12.0
        persistence:
          enabled: true
          storageClass: single-zone-storage-class
    dpm:
        image:
            repository: ausente/nms-dpm
            tag: 2.12.0
        persistence:
          enabled: true
          storageClass: single-zone-storage-class
    ingestion:
        image:
            repository: ausente/nms-ingestion
            tag: 2.12.0
    integrations:
        image:
            repository: ausente/nms-integrations
            tag: 2.12.0
        persistence:
          enabled: true
          storageClass: single-zone-storage-class
    utility:
        image:
            repository: ausente/utility
            tag: latest
```

```
# kubectl get deployment -n nms
NAME           READY   UP-TO-DATE   AVAILABLE   AGE
apigw          1/1     1            1           22h
clickhouse     1/1     1            1           22h
core           1/1     1            1           22h
dpm            1/1     1            1           22h
ingestion      1/1     1            1           22h
integrations   1/1     1            1           22h
utility        0/0     0            0           22h
```


To prepare for the restoration process, I scaled the utility deployment to 1 replica:
```
kubectl -n nms scale deployment/utility --replicas=1
```

I also ensured the utility pod was scheduled in the desired westus2-2 zone: 
```
kubectl patch deployment utility -n nms --type='json' -p='[{"op": "add", "path": "/spec/template/spec/affinity", "value": {"nodeAffinity": {"requiredDuringSchedulingIgnoredDuringExecution": {"nodeSelectorTerms": [{"matchExpressions": [{"key": "topology.kubernetes.io/zone", "operator": "In", "values": ["westus2-2"]}]}]}}}}]'
```

With the utility pod now correctly scheduled and running, I proceeded to run the restoration script, anticipating a recovery of the NMS configurations and data.
It was able to restore my instance group list at this point. 






