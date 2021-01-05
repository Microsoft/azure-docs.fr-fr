---
title: Utiliser kubectl pour déployer une application avec état Kubernetes via un partage provisionné de manière statique sur un appareil Azure Stack Edge Pro | Microsoft Docs
description: Décrit comment créer et gérer le déploiement d’une application avec état Kubernetes via un partage provisionné de manière statique à l’aide de kubectl sur un appareil GPU Azure Stack Edge Pro.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 09/22/2020
ms.author: alkohli
ms.openlocfilehash: c2a14c12baac29d73754bb17e3ca386cc48e1ba0
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96449238"
---
# <a name="use-kubectl-to-run-a-kubernetes-stateful-application-with-a-persistentvolume-on-your-azure-stack-edge-pro-device"></a>Utiliser kubectl pour exécuter une application avec état Kubernetes à l’aide d’une ressource PersistentVolume sur votre appareil Azure Stack Edge Pro

Cet article explique comment déployer une application avec état à instance unique dans Kubernetes à l’aide d’un volume persistant PersistentVolume (PV) et d’un déploiement. Le déploiement utilise des commandes `kubectl` sur un cluster Kubernetes existant et déploie l’application MySQL. 

Cette procédure est destinée à ceux qui ont étudié le [stockage Kubernetes sur un appareil Azure Stack Edge Pro](azure-stack-edge-gpu-kubernetes-storage.md) et connaissent les concepts du [stockage Kubernetes](https://kubernetes.io/docs/concepts/storage/).

Azure Stack Edge Pro prend également en charge l’exécution de conteneurs Azure SQL Edge. Ceux-ci peuvent être déployés de la façon détaillée ici pour MySQL. Pour plus d’informations, consultez [Azure SQL Edge](../azure-sql-edge/overview.md).


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
        - Vérifiez la compatibilité de ces deux versions à partir du mappage fourni dans la version Kubernetes prise en charge <!-- insert link-->. 


Vous êtes prêt à déployer une application avec état sur votre appareil Azure Stack Edge Pro. 

## <a name="provision-a-static-pv"></a>Approvisionner un PV statique

Pour approvisionner un PV de manière statique, vous devez créer un partage sur votre appareil. Procédez comme suit pour provisionner une ressource PV sur votre partage SMB. 

> [!NOTE]
> L’exemple spécifique utilisé dans ce guide pratique ne fonctionne pas avec les partages NFS. En général, les partages NFS peuvent être provisionnés sur votre appareil Azure Stack Edge avec des applications sans base de données.

1. Indiquez si vous souhaitez créer un partage Edge ou un partage local Edge. Suivez les instructions dans [Ajouter un partage](azure-stack-edge-manage-shares.md#add-a-share) pour créer un partage. Veillez à cocher la case **Utiliser le partage avec le computing en périphérie**.

    ![Partage local Edge pour PV](./media/azure-stack-edge-gpu-deploy-stateful-application-static-provision-kubernetes/edge-local-share-static-provision-1.png)

    1. Au lieu de créer un nouveau partage, si vous pouvez monter un partage existant.
    
        Sur le portail Azure de votre ressource Azure Stack Edge, accédez à **Partages**. Dans la liste des partages, sélectionnez un partage que vous souhaitez utiliser en cliquant dessus.

        ![Sélectionner un partage local existant pour PV](./media/azure-stack-edge-gpu-deploy-stateful-application-static-provision-kubernetes/mount-edge-share-1.png)

    1. Sélectionnez **Monter** et confirmez le montage quand vous y êtes invité.  

        ![Monter un partage local existant pour PV](./media/azure-stack-edge-gpu-deploy-stateful-application-static-provision-kubernetes/mount-edge-share-2.png)

1. Notez le nom du partage. Une fois ce partage créé, un objet de volume persistant est automatiquement créé dans le cluster Kubernetes correspondant au partage SMB que vous avez créé. 

## <a name="deploy-mysql"></a>Déployer MySQL

Vous allez maintenant exécuter une application avec état en créant un déploiement Kubernetes et en le connectant au PV que vous avez créé à l’étape précédente à l’aide d’une revendication de volume persistant PersistentVolumeClaim (PVC). 

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

1. Vous utiliserez les fichiers YAML suivants. Le fichier `mysql-deployment.yml` décrit un déploiement qui exécute MySQL et fait référence à la PVC. Le fichier définit un montage de volume pour `/var/lib/mysql`, puis crée une PVC qui recherche un volume de 20 Go. 

    Cette revendication est satisfaite par les PV existants approvisionnés de manière statique lors de la création du partage à l’étape précédente. Sur votre appareil, un grand PV de 32 To est créé pour chaque partage. Le PV répond aux exigences stipulées par la PVC, et la PVC doit être liée à ce PV.

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
              claimName: mysql-pv-claim
    ```
    
2. Copiez et enregistrez sous un fichier `mysql-pv.yml` dans le même dossier où vous avez enregistré `mysql-deployment.yml`. Pour utiliser le partage SMB créé précédemment avec `kubectl`, définissez le champ `volumeName` de l’objet PVC sur le nom du partage. 

    > [!NOTE] 
    > Assurez-vous que les fichiers YAML ont une mise en retrait correcte. Vous pouvez vérifier cela à l’aide de [YAML Lint](http://www.yamllint.com/) pour les valider et les enregistrer.
   
    ```yml
    apiVersion: v1
    kind: PersistentVolumeClaim
    metadata:
      name: mysql-pv-claim
    spec:
      volumeName: <smb-share-name-here>
      storageClassName: ""
      accessModes:
        - ReadWriteOnce
      resources:
        requests:
          storage: 20Gi
    ```

3. Déployez le fichier `mysql-pv.yaml`.

    `kubectl apply -f <URI path to the mysql-pv.yml file> -n <your-user-namespace>`
    
    Voici un exemple de sortie du déploiement.

    
    ```powershell
    C:\Users\user>kubectl apply -f "C:\stateful-application\mysql-pv.yml" -n userns1
    persistentvolumeclaim/mysql-pv-claim created
    
    C:\Users\user>
    ```
   Notez le nom de la PVC créée. Vous l’utiliserez dans une étape ultérieure. 

4. Déployez le contenu du fichier `mysql-deployment.yml`.

    `kubectl apply -f <URI path to mysql-deployment.yml file> -n <your-user-namespace>`

    Voici un exemple de sortie du déploiement.
    
    ```powershell
    C:\Users\user>kubectl apply -f "C:\stateful-application\mysql-deployment.yml" -n userns1
        service/mysql created
        deployment.apps/mysql created
    ```
    
5. Affiche des informations sur le déploiement.

    `kubectl describe deployment <app-label> -n <your-user-namespace>`
    
    ```powershell
    C:\Users\user>kubectl describe deployment mysql -n userns1
    Name:               mysql
    Namespace:          userns1
    CreationTimestamp:  Tue, 18 Aug 2020 09:44:58 -0700
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
        ClaimName:  mysql-pv-claim
        ReadOnly:   false
    Conditions:
      Type           Status  Reason
      ----           ------  ------
      Progressing    True    NewReplicaSetAvailable
      Available      True    MinimumReplicasAvailable
    OldReplicaSets:  <none>
    NewReplicaSet:   mysql-c85f7f79c (1/1 replicas created)
    Events:
      Type    Reason             Age   From                   Message
      ----    ------             ----  ----                   -------
      Normal  ScalingReplicaSet  10m   deployment-controller  Scaled up replica set mysql-c85f7f79c to 1
    
    C:\Users\user>
    ```
    

6. Répertorie les pods créés par le déploiement.

    `kubectl get pods -l <app=label> -n <your-user-namespace>`

    Voici un exemple de sortie.

    
    ```powershell
    C:\Users\user>kubectl get pods -l app=mysql -n userns1
    NAME                    READY   STATUS    RESTARTS   AGE
    mysql-c85f7f79c-vzz7j   1/1     Running   1          14m
    
    C:\Users\user>
    ```
    
7. Inspectez la ressource PersistentVolumeClaim.

    `kubectl describe pvc <your-pvc-name>`

    Voici un exemple de sortie.

    
    ```powershell
    C:\Users\user>kubectl describe pvc mysql-pv-claim -n userns1
    Name:          mysql-pv-claim
    Namespace:     userns1
    StorageClass:
    Status:        Bound
    Volume:        mylocalsmbshare1
    Labels:        <none>
    Annotations:   kubectl.kubernetes.io/last-applied-configuration:
                     {"apiVersion":"v1","kind":"PersistentVolumeClaim","metadata":{"annotations":{},"name":"mysql-pv-claim","namespace":"userns1"},"spec":{"acc...
                   pv.kubernetes.io/bind-completed: yes
    Finalizers:    [kubernetes.io/pvc-protection]
    Capacity:      32Ti
    Access Modes:  RWO,RWX
    VolumeMode:    Filesystem
    Mounted By:    mysql-c85f7f79c-vzz7j
    Events:        <none>
    
    C:\Users\user>
    ```
    

## <a name="verify-mysql-is-running"></a>Vérifier que MySQL est en cours d’exécution


Pour exécuter une commande sur un conteneur dans un pod qui exécute MySQL, saisissez :

`kubectl exec <your-pod-with-the-app> -i -t -n <your-namespace> -- mysql`

Voici un exemple de sortie.

```powershell
C:\Users\user>kubectl exec mysql-c85f7f79c-vzz7j -i -t -n userns1 -- mysql
Welcome to the MySQL monitor.  Commands end with ; or \g.
Your MySQL connection id is 1
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
C:\Users\user>kubectl delete pvc mysql-pv-claim -n userns1
persistentvolumeclaim "mysql-pv-claim" deleted
C:\Users\user>
```                                                                                         

Le PV n’est plus lié à la PVC, car celle-ci a été supprimée. Comme le PV a été approvisionné lors de la création du partage, vous devrez supprimer le partage. Procédez comme suit :

1. Démontez le partage. Dans Portail Azure, accédez à votre **ressource Azure Stack Edge > Partages**, puis sélectionnez le partage que vous souhaitez démonter en cliquant dessus. Sélectionnez **Démonter** et confirmez l’opération. Attendez que le partage soit démonté. Le démontage libère le partage (et donc le PersistentVolume associé) du cluster Kubernetes. 

    ![Démonter le partage local pour PV](./media/azure-stack-edge-gpu-deploy-stateful-application-static-provision-kubernetes/unmount-edge-local-share-1.png)

1. Vous pouvez maintenant sélectionner **Supprimer** et confirmer la suppression pour supprimer votre partage. Cela doit également supprimer le partage et le PV correspondant.

    ![Supprimer le partage local pour PV](./media/azure-stack-edge-gpu-deploy-stateful-application-static-provision-kubernetes/delete-edge-local-share-1.png)


## <a name="next-steps"></a>Étapes suivantes

Pour comprendre comment provisionner le stockage de manière dynamique, consultez [Déployer une application avec état via le provisionnement dynamique sur un appareil Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-stateful-application-dynamic-provision-kubernetes.md)