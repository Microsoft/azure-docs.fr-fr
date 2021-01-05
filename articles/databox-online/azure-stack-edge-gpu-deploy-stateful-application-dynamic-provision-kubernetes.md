---
title: Utiliser kubectl pour déployer une application avec état Kubernetes via un partage approvisionné de manière dynamique sur un appareil Azure Stack Edge Pro avec GPU | Microsoft Docs
description: Décrit comment créer et gérer un déploiement d’application avec état Kubernetes via un partage approvisionné de manière dynamique à l’aide de kubectl sur un appareil Microsoft Azure Stack Edge Pro avec GPU.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 08/26/2020
ms.author: alkohli
ms.openlocfilehash: 81a52b26c5291f788ac81caeb2ca5416a2f58d36
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96448876"
---
# <a name="use-kubectl-to-run-a-kubernetes-stateful-application-with-storageclass-on-your-azure-stack-edge-pro-gpu-device"></a>Utiliser kubectl pour exécuter une application avec état Kubernetes à l’aide d’une StorageClass sur votre appareil Azure Stack Edge Pro avec GPU

Cet article explique comment déployer une application avec état à instance unique dans Kubernetes à l’aide d’une classe StorageClass, pour approvisionner le stockage de manière dynamique, et d’un déploiement. Le déploiement utilise des commandes `kubectl` sur un cluster Kubernetes existant et déploie l’application MySQL. 

Cette procédure est destinée aux personnes qui ont examiné le [stockage Kubernetes sur un appareil Azure Stack Edge Pro](azure-stack-edge-gpu-kubernetes-storage.md) et connaissent les concepts du [stockage Kubernetes](https://kubernetes.io/docs/concepts/storage/).


## <a name="prerequisites"></a>Prérequis

Avant de pouvoir déployer l’application avec état, assurez-vous d’avoir rempli les conditions préalables suivantes sur votre appareil et le client que vous utiliserez pour accéder à ce dernier :

### <a name="for-device"></a>Pour l’appareil

- Vous avez les informations d’identification de connexion à un appareil Azure Stack Edge Pro à 1 nœud.
    - L’appareil est activé. Voir [Activer l’appareil](azure-stack-edge-gpu-deploy-activate.md).
    - Le rôle de calcul est configuré sur l’appareil via Portail Azure et l’appareil dispose d’un cluster Kubernetes. Voir [Configurer le calcul](azure-stack-edge-gpu-deploy-configure-compute.md).

### <a name="for-client-accessing-the-device"></a>Pour le client qui accède à l’appareil

- Vous disposez d’un système client Windows qui sera utilisé pour accéder à l’appareil Azure Stack Edge Pro.
    - Le client exécute Windows PowerShell 5.0 ou une version ultérieure. Pour télécharger la dernière version de Windows PowerShell, accédez à [Installation de Windows PowerShell](/powershell/scripting/install/installing-windows-powershell?view=powershell-7).
    
    - Vous pouvez également utiliser un autre client avec un [système d’exploitation pris en charge](azure-stack-edge-gpu-system-requirements.md#supported-os-for-clients-connected-to-device). Cet article décrit la procédure à utiliser avec un client Windows. 
    
    - Vous avez terminé la procédure décrite dans [Accéder au cluster Kubernetes sur un appareil Azure Stack Edge Pro](azure-stack-edge-gpu-create-kubernetes-cluster.md). Vous avez :
      - Création d’un espace de noms `userns1` via la commande `New-HcsKubernetesNamespace`. 
      - Création d’un utilisateur `user1` via la commande `New-HcsKubernetesUser`. 
      - Octroi de l’accès `user1` à `userns1` via la commande `Grant-HcsKubernetesNamespaceAccess`.       
      - Installation de `kubectl` sur le client et enregistrement du fichier `kubeconfig` avec la configuration de l’utilisateur sur C:\\Users\\&lt;username&gt;\\.kube. 
    
    - Assurez-vous que la version du client `kubectl` n’est pas décalée de plus d’une version par rapport à la version principale de Kubernetes exécutée sur votre appareil Azure Stack Edge Pro. 
        - Utilisez `kubectl version` pour vérifier la version de kubectl en cours d’exécution sur le client. Prenez note de la version complète.
        - Dans l’interface utilisateur locale de votre appareil Azure Stack Edge Pro, accédez à **Vue d’ensemble** et notez le numéro de version du logiciel Kubernetes. 
        - Vérifiez la compatibilité de ces deux versions à partir du mappage fourni dans la version Kubernetes prise en charge<!-- insert link-->. 


Vous êtes prêt à déployer une application avec état sur votre appareil Azure Stack Edge Pro. 


## <a name="deploy-mysql"></a>Déployer MySQL

Vous allez maintenant exécuter une application avec état en créant un déploiement Kubernetes et en le connectant à la classe de stockage StorageClass intégrée à l’aide d’une revendication de volume persistant PersistentVolumeClaim (PVC). 

Toutes les commandes `kubectl` que vous utilisez pour créer et gérer des déploiements d’application avec état doivent spécifier l’espace de noms associé à la configuration. Pour spécifier l’espace de noms dans une commande kubectl, utilisez `kubectl <command> -n <your-namespace>`.

1. Obtenez la liste des pods en cours d’exécution sur votre cluster Kubernetes dans votre espace de noms. Un pod est un conteneur d’application, ou processus, s’exécutant sur votre cluster Kubernetes.

   ```powershell
   kubectl get pods -n <your-namespace>
   ```
    
   Voici un exemple d’utilisation de la commande :
    
   ```powershell
    C:\Users\user>kubectl get pods -n "userns1"
    No resources found in userns1 namespace.    
    C:\Users\user>
   ```
    
   La sortie doit indiquer qu’aucune ressource (pod) n’a été trouvée, car aucune application n’est en cours d’exécution sur votre cluster.

1. Vous utiliserez les fichiers YAML suivants. Le fichier `mysql-deployment.yml` décrit un déploiement qui exécute MySQL et fait référence à la PVC. Le fichier définit un montage de volume pour `/var/lib/mysql`, puis crée une PVC qui recherche un volume de 20 Go. Un PV dynamique est approvisionné, et la PVC y est liée.

    Copiez et enregistrez le fichier `mysql-deployment.yml` suivant dans un dossier sur le client Windows que vous utilisez pour accéder à l’appareil Azure Stack Edge Pro.
    
    ```yml
    apiVersion: v1
    kind: Service
    metadata:
      name: mysql
    spec:
      ports:
      - port: 3306
      selector:
        app: mysql
      clusterIP: None
    ---
    apiVersion: apps/v1 # for versions before 1.9.0 use apps/v1beta2
    kind: Deployment
    metadata:
      name: mysql
    spec:
      selector:
        matchLabels:
          app: mysql
      strategy:
        type: Recreate
      template:
        metadata:
          labels:
            app: mysql
        spec:
          containers:
          - image: mysql:5.6
            name: mysql
            env:
              # Use secret in real usage
            - name: MYSQL_ROOT_PASSWORD
              value: password
            ports:
            - containerPort: 3306
              name: mysql
            volumeMounts:
            - name: mysql-persistent-storage
              mountPath: /var/lib/mysql
          volumes:
          - name: mysql-persistent-storage
            persistentVolumeClaim:
              claimName: mysql-pv-claim-sc
    ```
    
2. Copiez et enregistrez sous un fichier `mysql-pvc.yml` dans le même dossier où vous avez enregistré `mysql-deployment.yml`. Pour utiliser la StorageClass intégrée de l’appareil Azure Stack Edge Pro sur un disque de données attaché, définissez le champ `storageClassName` de l’objet PVC sur `ase-node-local`, et accessModes doit être `ReadWriteOnce`. 

    > [!NOTE] 
    > Assurez-vous que les fichiers YAML ont une mise en retrait correcte. Vous pouvez vérifier cela à l’aide de [YAML Lint](http://www.yamllint.com/) pour les valider et les enregistrer.
   
    ```yml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: mysql-pv-claim-sc
    spec:
      storageClassName: ase-node-local
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 20Gi
    ```

3. Déployez le fichier `mysql-pvc.yaml`.

    `kubectl apply -f <URI path to the mysql-pv.yml file> -n <your-user-namespace>`
    
    Voici un exemple de sortie du déploiement.

    
    ```powershell
    C:\Users\user>kubectl apply -f "C:\stateful-application\mysql-pvc.yml" -n userns1
    persistentvolumeclaim/mysql-pv-claim-sc created
    C:\Users\user>
    ```
   Notez le nom de la PVC créée, ici `mysql-pv-claim-sc`. Vous l’utiliserez dans une étape ultérieure. 

4. Déployez le contenu du fichier `mysql-deployment.yml`.

    `kubectl apply -f <URI path to mysql-deployment.yml file> -n <your-user-namespace>`

    Voici un exemple de sortie du déploiement.
    
    ```powershell
    C:\Users\user>kubectl apply -f "C:\stateful-application\mysql-deployment.yml" -n userns1
    service/mysql created
    deployment.apps/mysql created
    C:\Users\user>
    ```
    
5. Affiche des informations sur le déploiement.

    `kubectl describe deployment <app-label> -n <your-user-namespace>`
    
    ```powershell
    C:\Users\user>kubectl describe deployment mysql -n userns1
    Name:               mysql
    Namespace:          userns1
    CreationTimestamp:  Thu, 20 Aug 2020 11:14:25 -0700
    Labels:             <none>
    Annotations:        deployment.kubernetes.io/revision: 1
                        kubectl.kubernetes.io/last-applied-configuration:
                          {"apiVersion":"apps/v1","kind":"Deployment","metadata":{"annotations":{},"name":"mysql","namespace":"userns1"},"spec":{"selector":{"matchL...
    Selector:           app=mysql
    Replicas:           1 desired | 1 updated | 1 total | 1 available | 0 unavailable
    StrategyType:       Recreate
    MinReadySeconds:    0
    Pod Template:
      Labels:  app=mysql
      Containers:
       mysql:
        Image:      mysql:5.6
        Port:       3306/TCP
        Host Port:  0/TCP
        Environment:
          MYSQL_ROOT_PASSWORD:  password
        Mounts:
          /var/lib/mysql from mysql-persistent-storage (rw)
      Volumes:
       mysql-persistent-storage:
        Type:       PersistentVolumeClaim (a reference to a PersistentVolumeClaim in the same namespace)
        ClaimName:  mysql-pv-claim-sc
        ReadOnly:   false
    Conditions:
      Type           Status  Reason
      ----           ------  ------
      Available      True    MinimumReplicasAvailable
      Progressing    True    NewReplicaSetAvailable
    OldReplicaSets:  <none>
    NewReplicaSet:   mysql-695c4d9dcd (1/1 replicas created)
    Events:
      Type    Reason             Age   From                   Message
      ----    ------             ----  ----                   -------
      Normal  ScalingReplicaSet  24s   deployment-controller  Scaled up replica set mysql-695c4d9dcd to 1
    C:\Users\user>
    ```
    

6. Répertorie les pods créés par le déploiement.

    `kubectl get pods -l <app=label> -n <your-user-namespace>`

    Voici un exemple de sortie.

    
    ```powershell
    C:\Users\user>kubectl get pods -l app=mysql -n userns1
    NAME                     READY   STATUS    RESTARTS   AGE
    mysql-695c4d9dcd-rvzff   1/1     Running   0          40s
    C:\Users\user>
    ```
    
7. Inspectez la ressource PersistentVolumeClaim.

    `kubectl describe pvc <your-pvc-name>`

    Voici un exemple de sortie.

    
    ```powershell
    C:\Users\user>kubectl describe pvc mysql-pv-claim-sc -n userns1
    Name:          mysql-pv-claim-sc
    Namespace:     userns1
    StorageClass:  ase-node-local
    Status:        Bound
    Volume:        pvc-dc48253c-82dc-42a4-a7c6-aaddc97c9b8a
    Labels:        <none>
    Annotations:   kubectl.kubernetes.io/last-applied-configuration:
                     {"apiVersion":"v1","kind":"PersistentVolumeClaim","metadata":{"annotations":{},"name":"mysql-pv-claim-sc","namespace":"userns1"},"spec":{"...
                   pv.kubernetes.io/bind-completed: yes
                   pv.kubernetes.io/bound-by-controller: yes
                   volume.beta.kubernetes.io/storage-provisioner: rancher.io/local-path
                   volume.kubernetes.io/selected-node: k8s-3q7lhq2cl-3q7lhq2
    Finalizers:    [kubernetes.io/pvc-protection]
    Capacity:      20Gi
    Access Modes:  RWO
    VolumeMode:    Filesystem
    Mounted By:    mysql-695c4d9dcd-rvzff
    Events:
      Type    Reason                 Age                From                                                                                                Message
      ----    ------                 ----               ----                                                                                                -------
      Normal  WaitForFirstConsumer   71s (x2 over 77s)  persistentvolume-controller                                                                         waiting for first consumer to be created before binding
      Normal  ExternalProvisioning   62s                persistentvolume-controller                                                                         waiting for a volume to be created, either by external provisioner "rancher.io/local-path" or manually created by system administrator
      Normal  Provisioning           62s                rancher.io/local-path_local-path-provisioner-6b84988bf9-tx8mz_1896d824-f862-4cbf-912a-c8cc0ca05574  External provisioner is provisioning volume for claim "userns1/mysql-pv-claim-sc"
      Normal  ProvisioningSucceeded  60s                rancher.io/local-path_local-path-provisioner-6b84988bf9-tx8mz_1896d824-f862-4cbf-912a-c8cc0ca05574  Successfully provisioned volume pvc-dc48253c-82dc-42a4-a7c6-aaddc97c9b8a
    C:\Users\user>
    ```
    

## <a name="verify-mysql-is-running"></a>Vérifier que MySQL est en cours d’exécution

Pour vérifier que l’application s’exécute, saisissez :

`kubectl exec <your-pod-with-the-app> -i -t -n <your-namespace> -- mysql -p`

Lorsque vous y êtes invité, fournissez le mot de passe. Le mot de passe se trouve dans votre fichier `mysql-deployment`.

Voici un exemple de sortie.

```powershell
C:\Users\user>kubectl exec mysql-695c4d9dcd-rvzff -i -t -n userns1 -- mysql -p
Enter password:
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 3
Server version: 5.6.49 MySQL Community Server (GPL)

Copyright (c) 2000, 2020, Oracle and/or its affiliates. All rights reserved.

Oracle is a registered trademark of Oracle Corporation and/or its
affiliates. Other names may be trademarks of their respective
owners.

Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.
mysql>
```

## <a name="delete-a-deployment"></a>Supprimer un déploiement

Pour supprimer le déploiement, supprimez les objets déployés par leur nom. Ces objets incluent le déploiement, le service et la PVC.
 
```powershell
kubectl delete deployment <deployment-name>,svc <service-name> -n <your-namespace>
kubectl delete pvc <your-pvc-name> -n <your-namespace>
```

Voici un exemple de sortie du moment où vous supprimez le déploiement et le service.

```powershell
C:\Users\user>kubectl delete deployment,svc mysql -n userns1
deployment.apps "mysql" deleted
service "mysql" deleted
C:\Users\user>
```
Voici un exemple de sortie du moment où vous supprimez la PVC.

```powershell
C:\Users\user>kubectl delete pvc mysql-pv-claim-sc -n userns1
persistentvolumeclaim "mysql-pv-claim-sc" deleted
C:\Users\user>
```                                                                                         


## <a name="next-steps"></a>Étapes suivantes

Pour comprendre comment configurer la mise en réseau via kubectl, consultez [Déployer une application sans état sur un appareil Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-stateless-application-iot-edge-module.md)