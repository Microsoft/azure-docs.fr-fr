---
title: Barrières de sécurité relatives au déploiement d’un cluster Service Fabric via Azure Resource Manager
description: Cet article fournit une vue d’ensemble des erreurs courantes commises lors du déploiement d’un cluster Service Fabric par le biais de Azure Resource Manager, et explique comment les éviter.
services: service-fabric
documentationcenter: .net
author: peterpogorski
ms.topic: conceptual
ms.date: 02/13/2020
ms.author: pepogors
ms.openlocfilehash: aa1499c57ead28bfcee90a2f224ef9c3bb1d7f58
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86247822"
---
# <a name="service-fabric-guardrails"></a>Barrières de sécurité relatives à Service Fabric 
Lors du déploiement d’un cluster Service Fabric, des barrières de sécurité sont appliquées et entraînent l’échec du déploiement d’Azure Resource Manager si la configuration du cluster n’est pas valide. Les sections suivantes fournissent une vue d’ensemble des problèmes de configuration courants du cluster et des étapes requises pour les atténuer. 

## <a name="durability-mismatch"></a>Incompatibilité de la durabilité
### <a name="overview"></a>Vue d’ensemble
La valeur de durabilité d’un type de nœud Service Fabric est définie dans deux sections différentes d’un modèle Azure Resource Manager : la section sur l’extension de groupes de machines virtuelles identiques de la ressource de groupe de machines virtuelles identiques et la section sur le type de nœud de la ressource de cluster Service Fabric. Il est impératif que la valeur de durabilité de ces sections corresponde. Dans le cas contraire, le déploiement de la ressource échoue.

La section suivante contient un exemple d’incompatibilité de durabilité entre le paramètre de durabilité de l’extension du groupe de machines virtuelles identiques et celui du type de nœud Service Fabric :  

**Paramètre de durabilité de l’extension du groupe de machines virtuelles identiques**
```json 
{
  "extensions": [
    {
      "name": "[concat('ServiceFabricNodeVmExt','_vmNodeType0Name')]",
      "properties": {
        "type": "ServiceFabricNode",
        "publisher": "Microsoft.Azure.ServiceFabric",
        "settings": {
          "durabilityLevel": "Silver",
        }
      }
    }
  ]
}
```

**Paramètre de durabilité du type de nœud Service Fabric** 
```json
{
  "nodeTypes": [
    {
      "name": "[variables('vmNodeType0Name')]",
      "durabilityLevel": "Bronze",
      "isPrimary": true,
      "vmInstanceCount": "[parameters('nt0InstanceCount')]"
    }
  ]
}
```

### <a name="error-messages"></a>Messages d’erreur
* Le niveau de durabilité du groupe de machines virtuelles identiques ne correspond pas à celui du type de nœud Service Fabric en cours
* Le niveau de durabilité du groupe de machines virtuelles identiques ne correspond pas à celui du type de nœud Service Fabric cible
* Le niveau de durabilité du groupe de machines virtuelles identiques ne correspond pas au niveau de durabilité Service Fabric en cours ou au niveau de durabilité du type de nœud Service Fabric cible 

### <a name="mitigation"></a>Limitation des risques
Pour corriger un problème d’incompatibilité entre des durabilités, comme indiqué dans les messages d’erreur ci-dessus, procédez comme suit :
1. Mettez à jour le niveau de durabilité dans la section de l’extension du groupe de machines virtuelles identiques ou du type de nœud Service Fabric dans le modèle Resource Manager, afin de vous assurer que les valeurs correspondent.
2. Redéployez le modèle Azure Resource Manager avec les valeurs à jour.


## <a name="seed-node-deletion"></a>Suppression des nœuds initiaux 
### <a name="overview"></a>Vue d’ensemble
Un cluster Service Fabric a une propriété de [niveau de fiabilité](./service-fabric-cluster-capacity.md#reliability-characteristics-of-the-cluster) qui est utilisée pour déterminer le nombre de réplicas des services système qui s’exécutent sur le type de nœud principal du cluster. Le nombre de réplicas nécessaire détermine le nombre minimal de nœuds qui doivent être maintenus dans le type de nœud principal du cluster. Si le nombre de nœuds dans le type de nœud principal est inférieur au minimum requis pour le niveau de fiabilité, le cluster devient instable.  

### <a name="error-messages"></a>Messages d’erreur 
L’opération de suppression du nœud initial a été détectée et sera rejetée. 
* Cette opération entraînerait uniquement la conservation de {0} nœuds initiaux potentiels dans le cluster, alors que {1} sont nécessaires au minimum.
* La suppression de {0} nœuds initiaux sur {1} entraînerait une défaillance du cluster en raison d’une perte de quorum de nœuds initiaux. Le nombre maximal de nœuds initiaux pouvant être supprimés à la fois est de {2}.
 
### <a name="mitigation"></a>Limitation des risques 
Assurez-vous que votre type de nœud principal dispose de suffisamment de machines virtuelles pour la fiabilité spécifiée sur votre cluster. Vous ne pourrez pas supprimer une machine virtuelle si cela met le groupe de machines virtuelles identiques sous le nombre minimal de nœuds du niveau de fiabilité donné.
* Si le niveau de fiabilité est spécifié correctement, veillez à avoir suffisamment de nœuds dans le type de nœud principal pour le niveau de fiabilité. 
* Si le niveau de fiabilité est incorrect, apportez un changement sur la ressource Service Fabric pour réduire le niveau de fiabilité avant de lancer des opérations de groupe de machines virtuelles identiques et attendez qu’il soit effectué.
* Si le niveau de fiabilité est Bronze, suivez ces [étapes](./service-fabric-cluster-scale-in-out.md#manually-remove-vms-from-a-node-typevirtual-machine-scale-set) pour effectuer normalement un scale-in de votre cluster.

## <a name="next-steps"></a>Étapes suivantes
* Créez un cluster sur des machines virtuelles ou des ordinateurs exécutant Windows Server : [Création de clusters Service Fabric pour Windows Server](service-fabric-cluster-creation-for-windows-server.md)
* Créez un cluster sur des machines virtuelles ou des ordinateurs exécutant Linux : [Créer un cluster Linux](service-fabric-cluster-creation-via-portal.md)
* Résoudre les problèmes liés à Service Fabric : [Guides de résolution des problèmes](https://github.com/Azure/Service-Fabric-Troubleshooting-Guides)
