---
title: Activer Azure Arc sur Kubernetes sur un appareil avec GPU Azure Stack Edge Pro | Microsoft Docs
description: Décrit comment activer Azure Arc sur un cluster Kubernetes existant sur votre appareil avec GPU Azure Stack Edge Pro.
services: databox
author: alkohli
ms.service: databox
ms.subservice: edge
ms.topic: how-to
ms.date: 11/12/2020
ms.author: alkohli
ms.openlocfilehash: 342f6a2c4761104823694f2181b3ffa8726a441e
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96449419"
---
# <a name="enable-azure-arc-on-kubernetes-cluster-on-your-azure-stack-edge-pro-gpu-device"></a>Activer Azure Arc sur un cluster Kubernetes sur votre appareil avec GPU Azure Stack Edge Pro

Cet article explique comment activer Azure Arc sur un cluster Kubernetes existant sur votre appareil Azure Stack Edge Pro. 

Cette procédure est destinée aux personnes qui ont examiné les [charges de travail Kubernetes sur un appareil Azure Stack Edge Pro](azure-stack-edge-gpu-kubernetes-workload-management.md) et connaissent les concepts de [Kubernetes avec Azure Arc (préversion)](../azure-arc/kubernetes/overview.md).


## <a name="prerequisites"></a>Prérequis

Avant d’activer Azure Arc sur un cluster Kubernetes, assurez-vous d’avoir rempli les conditions préalables suivantes sur votre appareil Azure Stack Edge Pro et le client que vous utiliserez pour accéder à celui-ci :

### <a name="for-device"></a>Pour l’appareil

1. Vous avez les informations d’identification de connexion à un appareil Azure Stack Edge Pro à 1 nœud.
    1. L’appareil est activé. Voir [Activer l’appareil](azure-stack-edge-gpu-deploy-activate.md).
    1. Le rôle de calcul est configuré sur l’appareil via le portail Azure et l’appareil dispose d’un cluster Kubernetes. Voir [Configurer le calcul](azure-stack-edge-gpu-deploy-configure-compute.md).

1. Vous disposez d’un accès de propriétaire à l’abonnement. Vous aurez besoin de cet accès au cours de l’étape d’attribution de rôle pour votre principal de service.
 

### <a name="for-client-accessing-the-device"></a>Pour un client accédant à l’appareil

1. Vous disposez d’un système client Windows qui sera utilisé pour accéder à l’appareil Azure Stack Edge Pro.
  
    - Le client exécute Windows PowerShell 5.0 ou une version ultérieure. Pour télécharger la dernière version de Windows PowerShell, accédez à [Installation de Windows PowerShell](https://docs.microsoft.com/powershell/scripting/install/installing-powershell-core-on-windows).
    
    - Vous pouvez également utiliser un autre client avec un [système d’exploitation pris en charge](azure-stack-edge-gpu-system-requirements.md#supported-os-for-clients-connected-to-device). Cet article décrit la procédure à utiliser avec un client Windows. 
    
1. Vous avez terminé la procédure décrite dans [Accéder au cluster Kubernetes sur un appareil Azure Stack Edge Pro](azure-stack-edge-gpu-create-kubernetes-cluster.md). Vous avez :
    
    - Installé `kubectl` sur le client.    
    - Assurez-vous que la version du client `kubectl` n’est pas décalée de plus d’une version par rapport à la version principale de Kubernetes exécutée sur votre appareil Azure Stack Edge Pro. 
      - Utilisez `kubectl version` pour vérifier la version de kubectl en cours d’exécution sur le client. Prenez note de la version complète.
      - Dans l’interface utilisateur locale de votre appareil Azure Stack Edge Pro, accédez à **Mise à jour de logiciel** et notez le numéro de version du serveur Kubernetes. 
    
        ![Vérifier le numéro de version du serveur Kubernetes](media/azure-stack-edge-gpu-connect-powershell-interface/verify-kubernetes-version-1.png)      
      
      - Vérifiez que ces deux versions sont compatibles. 


## <a name="register-kubernetes-resource-providers"></a>Inscrire des fournisseurs de ressources Kubernetes

Avant d’activer Azure Arc sur le cluster Kubernetes, vous devez activer et inscrire `Microsoft.Kubernetes` et `Microsoft.KubernetesConfiguration` pour votre abonnement. 

1. Pour activer un fournisseur de ressources, dans le portail Azure, accédez à l’abonnement que vous prévoyez d’utiliser pour le déploiement. Accédez à **Fournisseurs de ressources**. 
1. Dans le volet droit, recherchez les fournisseurs que vous souhaitez ajouter. Dans cet exemple, `Microsoft.Kubernetes` et `Microsoft.KubernetesConfiguration`.

    ![Inscrire des fournisseurs de ressources Kubernetes](media/azure-stack-edge-gpu-connect-powershell-interface/register-k8-resource-providers-1.png)

1. Sélectionnez un fournisseur de ressources, puis, dans la partie supérieure de la barre de commandes, sélectionnez **Inscrire**. L’inscription prend quelques minutes. 

    ![Inscrire des fournisseurs de ressources Kubernetes 2](media/azure-stack-edge-gpu-connect-powershell-interface/register-k8-resource-providers-2.png)

1. Actualisez l’interface utilisateur jusqu’à ce que le fournisseur de ressources soit inscrit. Répétez le processus pour les deux fournisseurs de ressources.
    
    ![Inscrire des fournisseurs de ressources Kubernetes 3](media/azure-stack-edge-gpu-connect-powershell-interface/register-k8-resource-providers-4.png)

Vous pouvez également inscrire des fournisseurs de ressources via `az cli`. Pour plus d’information, consultez [Inscrire les deux fournisseurs pour Kubernetes à extension Azure Arc](../azure-arc/kubernetes/connect-cluster.md#register-the-two-providers-for-azure-arc-enabled-kubernetes).

## <a name="create-service-principal-assign-role"></a>Créer un principal du service, attribuer un rôle

1. Assurez-vous que vous disposez du `Subscription ID` et du nom du groupe de ressources que vous avez utilisé pour le déploiement de ressource pour votre service Azure Stack Edge. Pour accéder à l’ID d’abonnement, accédez à votre ressource Azure Stack Edge dans le portail Azure. Accédez à **Vue d’ensemble > Essentiels**.

    ![Obtenir l’ID d’abonnement](media/azure-stack-edge-gpu-connect-powershell-interface/get-subscription-id-1.png)

    Pour obtenir le nom du groupe de ressources, accédez à **Propriétés**.

    ![Obtenir le nom du groupe de ressources](media/azure-stack-edge-gpu-connect-powershell-interface/get-resource-group-name-1.png)

1. Pour créer un principal de service, utilisez la commande suivante via `az cli`.

    `az ad sp create-for-rbac --skip assignment --name "<Informative name for service principal>"`  

    Pour plus d’informations sur la façon de se connecter à `az cli`, [démarrez Cloud Shell dans le portail Azure](../cloud-shell/quickstart-powershell.md#start-cloud-shell).

    Voici un exemple. 
    
    ```azurecli
    PS /home/user> az ad sp create-for-rbac --skip-assignment --name "https://azure-arc-for-ase-k8s"
    {
      "appId": "aa8a082e-0fa1-4a82-b51c-e8b2a9fdaa8b",
      "displayName": "azure-arc-for-ase-k8s",
      "name": "https://azure-arc-for-ase-k8s",
      "password": "<password>",
      "tenant": "72f988bf-86f1-41af-91ab-2d7cd011db47"
    }
    PS /home/user>
    ```

1. Prenez note des valeurs `appID`, `name`, `password` et `tenantID` que vous utiliserez comme entrée dans la commande suivante.

1. Après avoir créé le principal de service, attribuez le rôle `Kubernetes Cluster - Azure Arc Onboarding` au principal nouvellement créé. Il s’agit d’un rôle Azure intégré (utilisez l’ID de rôle dans la commande) avec des autorisations limitées. Utilisez la commande suivante :

    `az role assignment create --role 34e09817-6cbe-4d01-b1a2-e0eac5743d41 --assignee <appId-from-service-principal> --scope /subscriptions/<SubscriptionID>/resourceGroups/<Resource-group-name>`

    Voici un exemple.
    
    ```azurecli
    PS /home/user> az role assignment create --role 34e09817-6cbe-4d01-b1a2-e0eac5743d41 --assignee aa8a082e-0fa1-4a82-b51c-e8b2a9fdaa8b --scope /subscriptions/062c67a6-019b-40af-a775-c4dc1abe56ed/resourceGroups/myaserg1
    {
      "canDelegate": null,
      "id": "/subscriptions/062c67a6-019b-40af-a775-c4dc1abe56ed/resourceGroups/myaserg1/providers/Microsoft.Authorization/roleAssignments/59272f92-e5ce-4aeb-9c0c-62532d8caf25",
      "name": "59272f92-e5ce-4aeb-9c0c-62532d8caf25",
      "principalId": "b045b3fe-8745-4097-9674-91cb0afaad91",
      "principalType": "ServicePrincipal",
      "resourceGroup": "myaserg1",
      "roleDefinitionId": "/subscriptions/062c67a6-019b-40af-a775-c4dc1abe56ed/providers/Microsoft.Authorization/roleDefinitions/34e09817-6cbe-4d01-b1a2-e0eac5743d41",
      "scope": "/subscriptions/062c67a6-019b-40af-a775-c4dc1abe56ed/resourceGroups/myaserg1",
      "type": "Microsoft.Authorization/roleAssignments"
    }
    PS /home/user>
    ```
    Pour plus d’informations sur la création d’un principal du service et l’exécution de l’attribution de rôle, consultez les étapes décrites dans [Créer un principal de service d’intégration compatible avec Azure Arc](../azure-arc/kubernetes/create-onboarding-service-principal.md).


## <a name="enable-arc-on-kubernetes-cluster"></a>Activer Arc sur le cluster Kubernetes

Pour configurer le cluster Kubernetes pour la gestion d’Azure Arc, procédez comme suit :

1. [Connectez-vous à l’interface PowerShell](azure-stack-edge-gpu-connect-powershell-interface.md#connect-to-the-powershell-interface) de votre appareil.

1. Tapez :

    `Set-HcsKubernetesAzureArcAgent -SubscriptionId "<Your Azure Subscription Id>" -ResourceGroupName "<Resource Group Name>" -ResourceName "<Azure Arc resource name (shouldn't exist already)>" -Location "<Region associated with resource group>" -TenantId "<Tenant Id of service principal>" -ClientId "<App id of service principal>" -ClientSecret "<Password of service principal>"`


    > [!NOTE]
    > - Pour déployer Azure Arc sur votre appareil, vous devez utiliser une [Région prise en charge pour Azure Arc](../azure-arc/kubernetes/overview.md#supported-regions). 
    > - Utilisez la commande `az account list-locations` pour connaître le nom exact de l'emplacement à transmettre dans la cmdlet `Set-HcsKubernetesAzureArcAgent`. Les noms d'emplacement ne comportent généralement pas d'espaces.
    
    Voici un exemple :
   
    ```powershell
    [10.128.44.240]: PS>Set-HcsKubernetesAzureArcAgent -SubscriptionId "062c67a6-019b-40af-a775-c4dc1abe56ed" -ResourceGroupName "myaserg1" -ResourceName "myasetestresarc" -Location "westeurope" -TenantId "72f988bf-86f1-41af-91ab-2d7cd011db47" -ClientId "aa8a082e-0fa1-4a82-b51c-e8b2a9fdaa8b" -ClientSecret "<password>"
        [10.128.44.240]: PS>
    ```
    
    Dans le portail Azure, une ressource doit être créée avec le nom que vous avez fourni dans la commande précédente.

    ![Accéder à la ressource Azure Arc](media/azure-stack-edge-gpu-connect-powershell-interface/verify-azure-arc-enabled-1.png)

1. Pour vérifier que le service Azure Arc est correctement activé, exécutez la commande suivante à partir de l’interface PowerShell :

    `kubectl get deployments -n azure-arc`

    Cette commande recherche toutes les applications qui déployées dans l’espace de noms `azure-arc` correspondant à Azure Arc.

    Voici un exemple de sortie illustrant les agents Azure Arc qui ont été déployés sur votre cluster Kubernetes dans l’espace de noms `azure-arc`. 


    ```powershell
    [10.128.44.240]: PS>kubectl get deployments -n azure-arc
    NAME                        READY   UP-TO-DATE   AVAILABLE   AGE
    cluster-metadata-operator   1/1     1            1           45m
    clusteridentityoperator     1/1     1            1           45m
    config-agent                1/1     1            1           45m
    connect-agent               1/1     1            1           45m
    controller-manager          1/1     1            1           45m
    flux-logs-agent             1/1     1            1           45m
    metrics-agent               1/1     1            1           45m
    resource-sync-agent         1/1     1            1           45m
    [10.128.44.240]: PS>
    ```

    Obtenez la liste des pods en cours d’exécution sur votre cluster Kubernetes dans votre espace de noms `azure-arc`. Un pod est un conteneur d’application, ou processus, s’exécutant sur votre cluster Kubernetes. 

    Utilisez la commande suivante :
    
    `kubectl get pods -n azure-arc`
    
    Voici un exemple de sortie.
    
    ```powershell
    [10.128.44.240]: PS>kubectl get pods -n azure-arc
    NAME                                         READY   STATUS    RESTARTS   AGE
    cluster-metadata-operator-64cbdf95b4-s2q52   2/2     Running   0          16m
    clusteridentityoperator-6f6dbccf7-nwnxg      3/3     Running   0          16m
    config-agent-7df5bf497b-mjm8k                3/3     Running   0          16m
    connect-agent-5d4c766764-m7h46               1/1     Running   0          16m
    controller-manager-777555fb57-t7tdp          3/3     Running   0          16m
    flux-logs-agent-845476c899-zcmtj             2/2     Running   0          16m
    metrics-agent-84d6fc8f4d-g9jkm               2/2     Running   0          16m
    resource-sync-agent-8f88dbf96-zgxjj          3/3     Running   0          16m
    [10.128.44.240]: PS>
    ```


Comme le montre la sortie précédente, un Kubernetes à extension Azure Arc se compose de quelques agents (opérateurs) qui s’exécutent dans votre cluster déployé dans l’espace de noms `azure-arc`.

- `config-agent` : surveille le cluster connecté pour les ressources de configuration de contrôle de code source appliquées au cluster, et met à jour l’état de conformité.
- `controller-manager` : opérateur d’opérateurs orchestrant les interactions entre les composants Azure Arc.
- `metrics-agent` : collecte les métriques d’autres agents Arc pour s’assurer que ces agents offrent des performances optimales.
- `cluster-metadata-operator` : collecte des métadonnées de cluster : version du cluster, nombre de nœuds et version de l’agent Azure Arc
- `resource-sync-agent` : synchronise les métadonnées de cluster précitées dans Azure.
- `clusteridentityoperator`: Kubernetes avec Azure Arc prend actuellement en charge l’identité attribuée par le système. clusteridentityoperator gère le certificat d’identité de service managé (MSI) utilisé par d’autres agents pour la communication avec Azure.
- `flux-logs-agent` : collecte des journaux des opérateurs de flux déployés dans le cadre de la configuration du contrôle de code source.
- `connect-agent` : communique avec la ressource Azure Arc.

### <a name="remove-arc-from-the-kubernetes-cluster"></a>Supprimer Arc du cluster Kubernetes

Pour supprimer la gestion d’Azure Arc, procédez comme suit :

1. 1. [Connectez-vous à l’interface PowerShell](azure-stack-edge-gpu-connect-powershell-interface.md#connect-to-the-powershell-interface) de votre appareil.
2. Tapez :

    `Remove-HcsKubernetesAzureArcAgent` 


> [!NOTE]
> Par défaut, lorsque les ressources `yamls` sont supprimées du référentiel Git, les ressources correspondantes ne sont pas supprimées du cluster Kubernetes. Vous devez définir `--sync-garbage-collection` dans Arc OperatorParams pour autoriser la suppression des ressources lorsqu'elles sont supprimées du référentiel Git. Pour plus d'informations, consultez [Supprimer une configuration](../azure-arc/kubernetes/use-gitops-connected-cluster.md#additional-parameters).

## <a name="next-steps"></a>Étapes suivantes

Pour comprendre comment exécuter un déploiement Azure Arc, consultez [Déployer une application PHP Guestbook sans état avec Redis via GitOps sur un appareil Azure Stack Edge Pro](azure-stack-edge-gpu-deploy-stateless-application-git-ops-guestbook.md).