---
title: Déployer l’application PHP Guestbook sur Kubernetes avec Arc sur un appareil Azure Stack Edge Pro avec GPU | Microsoft Docs
description: Décrit comment déployer une application sans état PHP Guestbook avec Redis en utilisant GitOps sur un cluster Kubernetes avec Arc de votre appareil Azure Stack Edge Pro.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 08/25/2020
ms.author: alkohli
ms.openlocfilehash: 82b786f3c90a7ef482ccbcd6e66fb9abfc4eea5f
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97561934"
---
# <a name="deploy-a-php-guestbook-stateless-application-with-redis-on-arc-enabled-kubernetes-cluster-on-azure-stack-edge-pro-gpu"></a>Déployer une application sans état PHP Guestbook avec Redis sur un cluster Kubernetes avec Arc sur Azure Stack Edge Pro avec GPU

Cet article explique comment créer et déployer une simple application web multiniveau à l’aide de Kubernetes et d’Azure Arc. Cet exemple est constitué des composants suivants :

- un maître Redis à instance unique pour stocker les entrées de guestbook ;
- plusieurs instances Redis répliquées pour servir les lectures ;
- plusieurs instances frontales web.

Le déploiement s’effectue à l’aide de GitOps sur le cluster Kubernetes avec Arc sur votre appareil Azure Stack Edge Pro. 

Cette procédure est destinée aux personnes qui ont examiné les [charges de travail Kubernetes sur un appareil Azure Stack Edge Pro](azure-stack-edge-gpu-kubernetes-workload-management.md) et connaissent les concepts de [Kubernetes avec Azure Arc (préversion)](../azure-arc/kubernetes/overview.md).


## <a name="prerequisites"></a>Prérequis

Avant de pouvoir déployer l’application sans état, assurez-vous d’avoir rempli les conditions préalables suivantes sur votre appareil et le client que vous utiliserez pour accéder à celui-ci :

> [!NOTE]
> Cet article contient des références au terme esclave, un terme que Microsoft n’utilise plus. Lorsque le terme sera supprimé du logiciel, nous le supprimerons de cet article.

### <a name="for-device"></a>Pour l’appareil

1. Vous avez les informations d’identification de connexion à un appareil Azure Stack Edge Pro à 1 nœud.
    1. L’appareil est activé. Voir [Activer l’appareil](azure-stack-edge-gpu-deploy-activate.md).
    1. Le rôle de calcul est configuré sur l’appareil via le portail Azure et l’appareil dispose d’un cluster Kubernetes. Voir [Configurer le calcul](azure-stack-edge-gpu-deploy-configure-compute.md).

1. Vous avez activé Azure Arc sur le cluster Kubernetes existant sur votre appareil, et vous disposez d’une ressource Azure Arc correspondante dans le portail Azure. Pour obtenir des instructions détaillées, consultez [Activer Azure Arc sur un appareil Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-arc-kubernetes-cluster.md).

### <a name="for-client-accessing-the-device"></a>Pour un client accédant à l’appareil

1. Vous disposez d’un système client Windows qui sera utilisé pour accéder à l’appareil Azure Stack Edge Pro.
  
    - Le client exécute Windows PowerShell 5.0 ou une version ultérieure. Pour télécharger la dernière version de Windows PowerShell, accédez à [Installation de Windows PowerShell](/powershell/scripting/install/installing-windows-powershell?view=powershell-7).
    
    - Vous pouvez également utiliser un autre client avec un [système d’exploitation pris en charge](azure-stack-edge-gpu-system-requirements.md#supported-os-for-clients-connected-to-device). Cet article décrit la procédure à utiliser avec un client Windows. 
    
1. Vous avez terminé la procédure décrite dans [Accéder au cluster Kubernetes sur un appareil Azure Stack Edge Pro](azure-stack-edge-gpu-create-kubernetes-cluster.md). Vous avez :
    
    - Installé `kubectl` sur le client  <!--and saved the `kubeconfig` file with the user configuration to C:\\Users\\&lt;username&gt;\\.kube. -->
    
    - Assurez-vous que la version du client `kubectl` n’est pas décalée de plus d’une version par rapport à la version principale de Kubernetes exécutée sur votre appareil Azure Stack Edge Pro. 
      - Utilisez `kubectl version` pour vérifier la version de kubectl en cours d’exécution sur le client. Prenez note de la version complète.
      - Dans l’interface utilisateur locale de votre appareil Azure Stack Edge Pro, accédez à **Vue d’ensemble** et notez le numéro de version du logiciel Kubernetes. 
      - Vérifiez la compatibilité de ces deux versions à partir du mappage fourni dans la version Kubernetes prise en charge <!--insert link-->.

1. Vous disposez d’une [configuration GitOps que vous pouvez utiliser pour exécuter un déploiement d’Azure Arc](https://github.com/kagoyal/dbehaikudemo). Dans cet exemple, vous allez utiliser les fichiers `yaml` suivants pour opérer le déploiement sur votre appareil Azure Stack Edge Pro.

    - `frontend-deployment.yaml`<!-- - The guestbook application has a web frontend serving the HTTP requests written in PHP. It is configured to connect to the redis-master Service for write requests and the redis-slave service for Read requests. This file describes a deployment that runs the frontend of the guestbook application.-->
    - `frontend-service.yaml` <!-- - This allows you to configure an externally visible frontend Service that can be accessed from outside the Kubernetes cluster on your device.-->
    - `redis-master-deployment.yaml` <!-- - This deployment file runs a single replica Redis master Pod. The guestbook application uses Redis to store its data. It writes its data to a Redis master instance and reads data from multiple Redis slave instances.-->
    - `redis-master-service.yaml` <!-- - The guestbook application needs to communicate to the Redis master to write its data. You need to apply a Service to proxy the traffic to the Redis master Pod. A Service defines a policy to access the Pods.-->
    - `redis-slave-deployment.yaml` <!-- - Although the Redis master is a single pod, you can make it highly available to meet traffic demands by adding replica Redis slaves. The Redis Slave Deployment specifies two replicas.-->
    - `redis-slave-service.yaml` <!-- - The guestbook application needs to communicate to Redis slaves to read data. To make the Redis slaves discoverable, you need to set up a Service. A Service provides transparent load balancing to a set of Pods.-->

<!-- az cli version requirements-->


## <a name="deploy-configuration"></a>Déployer la configuration

Procédez comme suit pour configurer la ressource Azure Arc afin de déployer une configuration GitOps via le portail Azure : 

1. Dans votre portail Azure, accédez à la ressource Azure Arc que vous avez créée lorsque vous avez activé Azure Arc sur le cluster Kubernetes sur votre appareil. 

    ![Accéder à la ressource Azure Arc](media/azure-stack-edge-gpu-connect-powershell-interface/verify-azure-arc-enabled-1.png)

1. Accédez à **configurations**, puis sélectionnez **+ Ajouter une configuration**.

    ![Capture d’écran montrant le cluster Kubernetes Azure Arc avec l’option Ajouter une configuration sélectionnée.](media/azure-stack-edge-gpu-connect-powershell-interface/select-configurations-1.png)

1. Dans la boîte de dialogue **Ajouter une configuration**, entrez les valeurs appropriées pour les champs, puis sélectionnez **Appliquer**.

    |Paramètre  |Description |
    |---------|---------|
    |Nom de la configuration     | Nom de la ressource de configuration.        |
    |Nom d’instance de l’opérateur     |Nom d’instance de l’opérateur pour identifier une configuration spécifique. Le nom est une chaîne de maximum 253 caractères, composée uniquement de lettres minuscules, de chiffres, de traits et de points.         |
    |Espace de noms de l’opérateur     | Défini sur **demotestguestbook**, car il s’agit de l’espace de noms spécifié dans le `yaml` de déploiement. <br> Le champ définit l’espace de noms dans lequel l’opérateur est installé. Le nom est une chaîne de maximum 253 caractères, composée uniquement de lettres minuscules, de chiffres, de traits et de points.         |
    |URL du dépôt     |<br>Chemin d’accès du dépôt Git au format `http://github.com/username/repo` ou `git://github.com/username/repo` dans lequel se trouve votre configuration GitOps.         |
    |Étendue de l’opérateur     | Sélectionnez **Espace de noms**. <br>Définit l’étendue d’installation de l’opérateur. Sélectionnez cette valeur en tant qu’espace de noms. Votre opérateur sera installé dans l’espace de noms spécifié dans les fichiers yaml de déploiement.       |
    |Type d'opérateur     | Conservez les valeurs par défaut. <br>Spécifie le type de l’opérateur, par défaut, flux.        |
    |Paramètres de l’opérateur     | Laissez ce champ vide. <br>Ce champ contient les paramètres à passer à l’opérateur de flux.        |
    |Helm     | Définissez cette valeur sur **Désactivé**. <br>Activez cette option si vous souhaitez effectuer des déploiements basés sur un graphique.        |


    ![Ajouter une configuration](media/azure-stack-edge-gpu-connect-powershell-interface/add-configuration-1.png)


1. Le déploiement de la configuration démarre et l’**État de l’opérateur** indique **En attente**. 

    ![Capture d’écran montrant le cluster Kubernetes Azure Arc dans un état en attente pendant qu’il s’actualise.](media/azure-stack-edge-gpu-connect-powershell-interface/view-configurations-1.png)

1. Le déploiement prend quelques minutes. Une fois le déploiement terminé, l’**État de l’opérateur** indique **Installé**.

    ![Capture d’écran montrant le cluster Kubernetes Azure Arc dans un état installé.](media/azure-stack-edge-gpu-connect-powershell-interface/view-configurations-2.png)


## <a name="verify-deployment"></a>Vérifier le déploiement

Le déploiement via la configuration GitOps crée un espace de noms `demotestguestbook`, comme indiqué dans les fichiers `yaml` de déploiement situés dans le dépôt Git.

1. Une fois que vous avez appliqué la configuration GitOps, [connectez-vous à l’interface PowerShell de l’appareil](azure-stack-edge-gpu-connect-powershell-interface.md#connect-to-the-powershell-interface).
1. Exécutez la commande suivante pour répertorier les pods en cours d’exécution dans l’espace de noms `demotestguestbook` correspondant au déploiement.

    `kubectl get pods -n <your-namespace>`
  
    Voici un exemple de sortie.
  
    ```powershell
    [10.128.44.240]: PS>kubectl get pods -n demotestguestbook
    NAME                            READY   STATUS    RESTARTS   AGE
    aseoperator1-5569658644-cqtb5   1/1     Running   0          91m
    frontend-6cb7f8bd65-4xb4f       1/1     Running   0          91m
    frontend-6cb7f8bd65-q9cxj       1/1     Running   0          91m
    frontend-6cb7f8bd65-xpzs6       1/1     Running   0          91m
    memcached-86bdf9f56b-5l2fq      1/1     Running   0          91m
    redis-master-7db7f6579f-2z29w   1/1     Running   0          91m
    redis-slave-7664787fbc-lgr2n    1/1     Running   0          91m
    redis-slave-7664787fbc-vlvzn    1/1     Running   0          91m
    [10.128.44.240]: PS>
    ```  

1. Dans cet exemple, le service frontal a été déployé en tant que type:LoadBalancer. Vous devrez trouver l’adresse IP de ce service pour afficher le guestbook. Exécutez la commande suivante :

    `kubectl get service -n <your-namespace>`
    
    ```powershell
    [10.128.44.240]: PS>kubectl get service -n demotestguestbook
    NAME           TYPE           CLUSTER-IP       EXTERNAL-IP     PORT(S)        AGE
    frontend       LoadBalancer   10.96.79.38      10.128.44.245   80:31238/TCP   85m
    memcached      ClusterIP      10.102.47.75     <none>          11211/TCP      85m
    redis-master   ClusterIP      10.104.32.99     <none>          6379/TCP       85m
    redis-slave    ClusterIP      10.104.215.146   <none>          6379/TCP       85m
    [10.128.44.240]: PS>
    ```
1. Le service frontal de `type:LoadBalancer` a une adresse IP externe. Cette adresse IP provient de la plage d’adresses IP que vous avez spécifiée pour les services externes lors de la configuration des paramètres réseau de Compute sur l’appareil. Utilisez cette adresse IP pour afficher le guestbook à l’adresse URL : `https://<external-IP-address>`.

    ![Afficher le guestbook](media/azure-stack-edge-gpu-connect-powershell-interface/view-guestbook-1.png)

## <a name="delete-deployment"></a>Supprimer un déploiement

Pour supprimer le déploiement, vous pouvez supprimer la configuration dans le portail Azure. Cela a pour effet de supprimer les objets créés, déploiements et services compris.

1. Dans le portail Azure, accédez à la ressource Azure Arc > Configurations. 
1. Localisez la configuration à supprimer. Sélectionnez les points de suspension (...) pour appeler le menu contextuel, puis sélectionnez **Supprimer**.
    ![Supprimer une configuration](media/azure-stack-edge-gpu-connect-powershell-interface/delete-configuration-1.png)

La suppression de la configuration peut prendre plusieurs minutes.
 
<!--```powershell
kubectl delete deployment <deployment-name> -n <your-namespace>
kubectl delete service <service-name> -n <your-namespace>
```

Here is a sample output of when you delete the deployments and the services.

```powershell
C:\Users\user>kubectl delete deployment,svc mysql -n userns1
deployment.apps "mysql" deleted
service "mysql" deleted
C:\Users\user>
```-->


## <a name="next-steps"></a>Étapes suivantes

Découvrir comment [Utiliser le tableau de bord Kubernetes pour surveiller les déploiements sur votre appareil Azure Stack Edge Pro](azure-stack-edge-gpu-monitor-kubernetes-dashboard.md)