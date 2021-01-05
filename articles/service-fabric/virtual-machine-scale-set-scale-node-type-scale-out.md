---
title: Ajouter un type de nœud à un cluster Azure Service Fabric
description: Découvrez comment effectuer un scale-out d’un cluster Service Fabric en ajoutant un groupe de machines virtuelles identiques.
ms.topic: article
ms.date: 02/13/2019
ms.openlocfilehash: efd329c07b4881c6710d4173857b4186965438d8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88719323"
---
# <a name="scale-a-service-fabric-cluster-out-by-adding-a-virtual-machine-scale-set"></a>Montez en charge un cluster Service Fabric en ajoutant un groupe de machines virtuelles identiques
Cet article décrit comment mettre à l’échelle un cluster Azure Service Fabric en ajoutant un nouveau type de nœud à un cluster existant. Un cluster Service Fabric est un groupe de machines virtuelles ou physiques connectées au réseau, sur lequel vos microservices sont déployés et gérés. Une machine ou une machine virtuelle faisant partie d’un cluster est appelée un nœud. Les groupes de machines virtuelles identiques constituent une ressource de calcul Azure que vous utilisez pour déployer et gérer une collection de machines virtuelles en tant que groupe. Chaque type de nœud défini dans un cluster Azure est [ configuré comme un groupe identique distinct](service-fabric-cluster-nodetypes.md). Chaque type de nœud peut alors faire l’objet d’une gestion séparée. Après avoir créé un cluster Service Fabric, vous pouvez faire évoluer un cluster horizontalement en ajoutant un nouveau type de nœud (groupe de machines virtuelles identiques) à un cluster existant.  Une mise à l’échelle peut s’effectuer à tout moment, même lorsque des charges de travail sont en cours d’exécution sur le cluster.  Lorsque vous mettez vos nœuds à l’échelle, vos applications sont automatiquement mises à l’échelle.

## <a name="add-an-additional-scale-set-to-an-existing-cluster"></a>Ajouter un groupe identique à un cluster existant
L’ajout d’un nouveau type de nœud (qui s'appuie sur un groupe de machines virtuelles identiques) à un cluster existant est similaire à la [mise à niveau du type de nœud principal](service-fabric-scale-up-primary-node-type.md), sauf que vous n’utilisez pas le même NodeTypeRef. Il est évident que vous n’allez pas désactiver des groupes de machines virtuelles identiques utilisés activement, ni perdre la disponibilité du cluster si vous ne mettez pas à jour le type de nœud principal. 

La propriété NodeTypeRef est déclarée dans les propriétés de l’extension Service Fabric du groupe de machines virtuelles identiques :
```json
<snip>
"publisher": "Microsoft.Azure.ServiceFabric",
     "settings": {
     "clusterEndpoint": "[reference(parameters('clusterName')).clusterEndpoint]",
     "nodeTypeRef": "[parameters('vmNodeType2Name')]",
     "dataPath": "D:\\\\SvcFab",
     "durabilityLevel": "Silver",
<snip>
```

Vous devez en outre ajouter ce nouveau type de nœud à votre ressource de cluster Service Fabric :

```json
<snip>
"nodeTypes": [
      {
      "name": "[parameters('vmNodeType2Name')]",
      "applicationPorts": {
                "endPort": "[parameters('nt2applicationEndPort')]",
                "startPort": "[parameters('nt2applicationStartPort')]"
      },
      "clientConnectionEndpointPort": "[parameters('nt2fabricTcpGatewayPort')]",
      "durabilityLevel": "Silver",
       "ephemeralPorts": {
                "endPort": "[parameters('nt2ephemeralEndPort')]",
                "startPort": "[parameters('nt2ephemeralStartPort')]"
      },
      "httpGatewayEndpointPort": "[parameters('nt2fabricHttpGatewayPort')]",
      "isPrimary": false,
      "vmInstanceCount": "[parameters('nt2InstanceCount')]"
},
<snip>
```

## <a name="next-steps"></a>Étapes suivantes
* Découvrez comment [monter en puissance le type de nœud principal](service-fabric-scale-up-primary-node-type.md).
* Découvrez-en plus sur l’[extensibilité des applications](service-fabric-concepts-scalability.md).
* [Procédez à une montée et une descente en puissance d’un cluster Azure](service-fabric-tutorial-scale-cluster.md).
* [Mettez à l’échelle un cluster Azure par programmation](service-fabric-cluster-programmatic-scaling.md), à l’aide du kit de développement logiciel de calcul Azure.
* [Augmentez ou diminuez la taille des instances d’un cluster autonome](service-fabric-cluster-windows-server-add-remove-nodes.md).

