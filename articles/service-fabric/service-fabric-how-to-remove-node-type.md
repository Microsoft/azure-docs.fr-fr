---
title: Supprimer un type de nœud dans Azure Service Fabric | Microsoft Docs
description: Découvrez comment supprimer type de nœud d’un cluster Service Fabric dans Azure.
author: inputoutputcode
manager: sridmad
ms.topic: conceptual
ms.date: 08/11/2020
ms.author: chrpap
ms.openlocfilehash: ede999bee9ce1a4a9dd10652a2c52a840d5b24be
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88163575"
---
# <a name="how-to-remove-a-service-fabric-node-type"></a>Supprimer un type de nœud Service Fabric
Cet article décrit comment mettre à l’échelle un cluster Azure Service Fabric en supprimant un type de nœud existant d’un cluster. Un cluster Service Fabric est un groupe de machines virtuelles ou physiques connectées au réseau, sur lequel vos microservices sont déployés et gérés. Une machine ou une machine virtuelle faisant partie d’un cluster est appelée un nœud. Les groupes de machines virtuelles identiques constituent une ressource de calcul Azure que vous utilisez pour déployer et gérer une collection de machines virtuelles en tant que groupe. Chaque type de nœud défini dans un cluster Azure est [ configuré comme un groupe identique distinct](service-fabric-cluster-nodetypes.md). Chaque type de nœud peut alors faire l’objet d’une gestion séparée. Après avoir créé un cluster Service Fabric, vous pouvez faire évoluer un cluster horizontalement en supprimant un type de nœud (groupe de machines virtuelles identiques) et tous ses nœuds.  Une mise à l’échelle peut s’effectuer à tout moment, même lorsque des charges de travail sont en cours d’exécution sur le cluster.  Lorsque vous mettez vos nœuds à l’échelle, vos applications sont automatiquement mises à l’échelle.

> [!WARNING]
> Nous vous déconseillons d'utiliser cette approche trop fréquemment pour supprimer un type de nœud d'un cluster de production. Il s’agit d’une commande dangereuse, car elle supprime le groupe de machines virtuelles identiques derrière le type de nœud. 

## <a name="durability-characteristics"></a>Caractéristiques de durabilité
La sécurité est prioritaire par rapport à la vitesse lors de l’utilisation de Remove-AzServiceFabricNodeType. Le type de nœud doit avoir le [niveau de durabilité](./service-fabric-cluster-capacity.md#durability-characteristics-of-the-cluster) Silver ou Gold, car :
- Bronze ne vous accorde aucune garantie sur l’enregistrement des informations d’état.
- Les niveaux de durabilité Silver et Gold interceptent toutes les modifications du groupe de machines virtuelles identiques.
- Gold vous permet également de contrôler les mises à jour Azure sous le groupe de machines virtuelles identiques.

Service Fabric « gère » les modifications et les mises à jour sous-jacentes afin d’éviter toute perte de données. Toutefois, lorsque vous supprimez un type de nœud doté du niveau de durabilité Bronze, vous pouvez perdre les informations d'état. Si vous supprimez un type de nœud principal et que votre application est sans état, vous pouvez utiliser le niveau Bronze. Lorsque vous exécutez des charges de travail avec état en production, la configuration minimale doit être Silver. De même, pour les scénarios de production, le type de nœud principal doit toujours être Silver ou Gold.

### <a name="more-about-bronze-durability"></a>En savoir plus sur la durabilité Bronze

Lorsque vous supprimez un type de nœud de niveau Bronze, tous les nœuds dans le type de nœud s’arrêtent immédiatement. Service Fabric ne recouvre pas les mises à jour du groupe de machines virtuelles identiques de nœuds Bronze, par conséquent, toutes les machines virtuelles s’arrêtent immédiatement. Si vous aviez quoi que ce soit avec état sur ces nœuds, les données sont perdues. Cependant, même si vous étiez sans état, tous les nœuds dans Service Fabric participent à l’anneau, un voisinage entier peut donc être perdu, ce qui peut déstabiliser le cluster lui-même.

## <a name="remove-a-node-type"></a>Supprimer un type de nœud

1. Les conditions préalables suivantes doivent être réunies avant d'entamer le processus.

    - Le cluster est sain.
    - La capacité (comme le nombre de nœuds disponibles pour placer le nombre de réplicas requis) reste suffisante après la suppression du type de nœud.

2. Déplacez hors du type de nœud tous les services qui présentent des contraintes de placement liées à l'utilisation du type de nœud.

    - Modifiez l'application ou le manifeste de service pour ne plus faire référence au type de nœud.
    - Déployez la modification.

    Puis validez ce qui suit :
    - Tous les services modifiés ci-dessus ne sont plus exécutés sur le nœud appartenant au type de nœud.
    - Tous les services sont sains.

3. Ne marquez pas le type de nœud comme non principal (ignorez pour les types de nœuds non principaux)

    - Localisez le modèle Azure Resource Manager utilisé pour le déploiement.
    - Recherchez la section relative au type de nœud dans la section Service Fabric.
    - Définissez la propriété isPrimary sur false. ** Ne supprimez pas la section relative au type de nœud dans cette tâche.
    - Déployez le modèle Azure Resource Manager modifié. ** Selon la configuration du cluster, cette étape peut prendre un certain temps.
    
    Puis validez ce qui suit :
    - La section Service Fabric du portail indique que le cluster est prêt.
    - Le cluster est sain.
    - Aucun des nœuds appartenant au type de nœud n'est marqué comme nœud initial.

4. Désactivez chaque nœud dans le type de nœud.

    Connectez-vous au cluster à l'aide de PowerShell, puis exécutez l'étape suivante.
    
    ```powershell
    $nodeType = "" # specify the name of node type
    $nodes = Get-ServiceFabricNode
    
    foreach($node in $nodes)
    {
      if ($node.NodeType -eq $nodeType)
      {
        $node.NodeName
     
        Disable-ServiceFabricNode -Intent RemoveNode -NodeName $node.NodeName -Force
      }
    }
    ```

    - Pour la durabilité Bronze, attendez que tous les nœuds soient désactivés.
    - Pour la durabilité Silver ou Gold, certains nœuds seront désactivés et les autres seront en cours de désactivation. Consultez l'onglet des détails des nœuds en cours de désactivation. S'ils sont tous bloqués pour garantir le quorum des partitions du service d'infrastructure, il est possible de continuer en toute sécurité.

5. Arrêtez les données relatives au type de nœud.

    Connectez-vous au cluster à l'aide de PowerShell, puis exécutez l'étape suivante.
    
    ```powershell
    foreach($node in $nodes)
    {
      if ($node.NodeType -eq $nodeType)
      {
        $node.NodeName
     
        Start-ServiceFabricNodeTransition -Stop -OperationId (New-Guid) -NodeInstanceId $node.NodeInstanceId -NodeName $node.NodeName -StopDurationInSeconds 10000
      }
    }
    ```
    
    Attendez que tous les nœuds correspondant au type de nœud soient marqués comme inactifs.

6. Libérer des nœuds dans le groupe de machines virtuelles identiques d’origine
    
    Connectez-vous à l’abonnement Azure dans lequel le groupe identique a été déployé et supprimez le groupe de machines virtuelles identiques. 

    ```powershell
    $scaleSetName="myscaleset"
    $scaleSetResourceType="Microsoft.Compute/virtualMachineScaleSets"
    
    Remove-AzResource -ResourceName $scaleSetName -ResourceType $scaleSetResourceType -ResourceGroupName $resourceGroupName -Force
    ```

    
7. Supprimez les données relatives au type de nœud.

    Connectez-vous au cluster à l'aide de PowerShell, puis exécutez l'étape suivante.
    
    ```powershell
    foreach($node in $nodes)
    {
      if ($node.NodeType -eq $nodeType)
      {
        $node.NodeName
     
        Remove-ServiceFabricNodeState -NodeName $node.NodeName -Force
      }
    }
    ```

    Attendez que tous les nœuds soient supprimés du cluster. Les nœuds ne doivent pas être affichés dans SFX.

8. Supprimez le type de nœud de la section Service Fabric.

    - Localisez le modèle Azure Resource Manager utilisé pour le déploiement.
    - Recherchez la section relative au type de nœud dans la section Service Fabric.
    - Supprimez la section correspondant au type de nœud.
    - Pour les clusters Silver et de durabilité supérieure uniquement, mettez à jour la ressource de cluster dans le modèle et configurez les stratégies d’intégrité de manière à ignorer l’intégrité de l’application fabric:/System en ajoutant `applicationDeltaHealthPolicies` sous la ressource de cluster `properties` comme indiqué ci-dessous. La stratégie ci-dessous doit ignorer les erreurs existantes, mais ne pas autoriser de nouvelles erreurs d'intégrité. 
 
 
     ```json
    "upgradeDescription":  
    { 
      "forceRestart": false, 
      "upgradeReplicaSetCheckTimeout": "10675199.02:48:05.4775807", 
      "healthCheckWaitDuration": "00:05:00", 
      "healthCheckStableDuration": "00:05:00", 
      "healthCheckRetryTimeout": "00:45:00", 
      "upgradeTimeout": "12:00:00", 
      "upgradeDomainTimeout": "02:00:00", 
      "healthPolicy": { 
        "maxPercentUnhealthyNodes": 100, 
        "maxPercentUnhealthyApplications": 100 
      }, 
      "deltaHealthPolicy":  
      { 
        "maxPercentDeltaUnhealthyNodes": 0, 
        "maxPercentUpgradeDomainDeltaUnhealthyNodes": 0, 
        "maxPercentDeltaUnhealthyApplications": 0, 
        "applicationDeltaHealthPolicies":  
        { 
            "fabric:/System":  
            { 
                "defaultServiceTypeDeltaHealthPolicy":  
                { 
                        "maxPercentDeltaUnhealthyServices": 0 
                } 
            } 
        } 
      } 
    },
    ```

    - Déployez le modèle Azure Resource Manager modifié. ** Cette étape prendra un certain temps, mais généralement pas plus de deux heures. Cette mise à niveau modifiera les paramètres d'InfrastructureService. Un redémarrage du nœud est donc nécessaire. Dans ce cas, `forceRestart` est ignoré. 
    Le paramètre `upgradeReplicaSetCheckTimeout` spécifie la durée maximale pendant laquelle Service Fabric doit attendre qu'une partition soit sécurisée, si ce n'est pas encore le cas. Une fois les contrôles de sécurité réussis pour toutes les partitions d'un nœud, Service Fabric procède à la mise à niveau sur ce nœud.
    La valeur du paramètre `upgradeTimeout` peut être réduite à 6 heures, mais pour une sécurité maximale, il convient d'utiliser 12 heures.

    Puis validez ce qui suit :
    - La ressource Service Fabric est prête sur le portail.

9. Supprimez toutes les références aux ressources relatives au type de nœud à partir du modèle Resource Manager.

    - Localisez le modèle Azure Resource Manager utilisé pour le déploiement.
    - Supprimez le groupe de machines virtuelles identiques et les autres ressources associées au type de nœud à partir du modèle.
    - Déployez les modifications.

    Ensuite :
    - Attendez la fin du déploiement.
    
10. Supprimez les ressources associées au type de nœud qui ne sont plus utilisées. Exemple de Load Balancer, et IP publique. 

    - Pour supprimer ces ressources, vous pouvez utiliser la même commande PowerShell que celle utilisée à l’étape 6, en spécifiant le type de ressource spécifique et la version de l’API. 

> [!Note]
> Cette étape est facultative si le Load Balancer et l’adresse IP réutilisés entre les types de nœuds sont les mêmes.

## <a name="next-steps"></a>Étapes suivantes
- En savoir plus sur les [caractéristiques de durabilité](./service-fabric-cluster-capacity.md#durability-characteristics-of-the-cluster) du cluster.
- En savoir plus les [types de nœud et les groupe de machines virtuelles identiques](service-fabric-cluster-nodetypes.md).
- En savoir plus sur la [mise à l’échelle du cluster Service Fabric](service-fabric-cluster-scaling.md).
