---
title: Types de nœud et groupes de machines virtuelles identiques
description: Découvrez la relation entre les types de nœuds Azure Service Fabric et les groupes de machines virtuelles identiques, ainsi que la méthode permettant de se connecter à distance à une instance de groupe identique ou à un nœud de cluster.
ms.topic: conceptual
ms.date: 03/23/2018
ms.author: pepogors
ms.openlocfilehash: 7df7ed9fa1f872cd9554dcf4c8880b7c9d5e53c4
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96576398"
---
# <a name="azure-service-fabric-node-types-and-virtual-machine-scale-sets"></a>Types de nœuds Azure Service Fabric et groupes de machines virtuelles identiques

Les [groupes de machines virtuelles identiques](../virtual-machine-scale-sets/index.yml) représentent une ressource de calcul Azure. Vous pouvez utiliser des groupes identiques pour déployer et gérer une collection de machines virtuelles comme un groupe. Chaque type de nœud que vous définissez dans un cluster Azure Service Fabric configure exactement un groupe identique : plusieurs types de nœuds ne peuvent pas être sauvegardés par le même groupe identique et un type de nœud ne doit pas (dans la plupart des cas) être sauvegardé par plusieurs groupes identiques. Une exception à cette règle est observée dans le rare cas de la [mise à l’échelle verticale](service-fabric-best-practices-capacity-scaling.md#vertical-scaling-considerations) d’un type de nœud, lorsque vous avez temporairement deux groupes identiques avec la même valeur `nodeTypeRef`, tandis que les réplicas sont migrés de l’original au groupe identique mis à niveau.

Le runtime Service Fabric est installé sur chaque machine virtuelle dans le groupe identique par l’extension de machine virtuelle *Microsoft.Azure.ServiceFabric*. Vous pouvez faire monter ou descendre en puissance chaque type de nœud de manière indépendante, modifier la référence SKU du système d’exploitation s’exécutant sur chaque nœud de cluster, avoir différents ensembles de ports ouverts et utiliser différentes métriques de capacité.

La figure suivante illustre un cluster constitué de deux types de nœuds nommés *FrontEnd* et *BackEnd*. Chaque type de nœud compte cinq nœuds.

![Cluster constitué de deux types de nœuds][NodeTypes]

## <a name="map-virtual-machine-scale-set-instances-to-nodes"></a>Mapper des instances de groupe de machines virtuelles identiques à des nœuds

Comme l’illustre la figure précédente, les instances de groupe identique commencent à l’instance 0 et augmentent de 1. Les noms de nœuds reflètent la numérotation. Par exemple, le nœud BackEnd_0 représente l’instance 0 du groupe identique BackEnd. Ce groupe identique spécifique compte cinq instances nommées BackEnd_0, BackEnd_1, BackEnd_2, BackEnd_3 et BackEnd_4.

Quand vous effectuez un scale-out d’un groupe identique, une nouvelle instance est créée. En règle générale, le nom de la nouvelle instance du groupe identique est le nom du groupe identique suivi du numéro d’instance suivant. Dans notre exemple, il s’agit de BackEnd_5.

## <a name="map-scale-set-load-balancers-to-node-types-and-scale-sets"></a>Mapper des équilibreurs de charge de groupe identique à des types de nœuds et des groupes identiques

Si vous avez déployé votre cluster dans le portail Azure ou si vous avez utilisé l’exemple de modèle Azure Resource Manager, toutes les ressources d’un groupe de ressources sont répertoriées. Vous pouvez voir les équilibreurs de charge de chaque groupe identique ou type de nœud. Le nom d’un équilibreur de charge utilise le format suivant : **LB-&lt;nom du type de nœud&gt;** . La figure suivante en propose un exemple (LB-sfcluster4doc-0) :

![Capture d’écran montrant un groupe de ressources avec deux équilibreurs de charge mis en surbrillance.][Resources]

## <a name="service-fabric-virtual-machine-extension"></a>Extension de machine virtuelle Service Fabric

L’extension de machine virtuelle Service Fabric est utilisée pour démarrer Service Fabric sur des machines virtuelles Azure et configurer la sécurité des nœuds.

Voici un extrait de code d’extension de machine virtuelle Service Fabric :

```json
"extensions": [
  {
    "name": "[concat('ServiceFabricNodeVmExt','_vmNodeType0Name')]",
    "properties": {
      "type": "ServiceFabricLinuxNode",
      "autoUpgradeMinorVersion": true,
      "protectedSettings": {
        "StorageAccountKey1": "[listKeys(resourceId('Microsoft.Storage/storageAccounts', variables('supportLogStorageAccountName')),'2015-05-01-preview').key1]",
       },
       "publisher": "Microsoft.Azure.ServiceFabric",
       "settings": {
         "clusterEndpoint": "[reference(parameters('clusterName')).clusterEndpoint]",
         "nodeTypeRef": "[variables('vmNodeType0Name')]",
         "durabilityLevel": "Silver",
         "enableParallelJobs": true,
         "nicPrefixOverride": "[variables('subnet0Prefix')]",
         "dataPath": "D:\\\\SvcFab",
         "certificate": {
           "commonNames": [
             "[parameters('certificateCommonName')]"
           ],
           "x509StoreName": "[parameters('certificateStoreValue')]"
         }
       },
       "typeHandlerVersion": "1.1"
     }
   },
```

Voici les descriptions des propriétés :

| **Nom** | **Valeurs autorisées** | **Conseils ou brève description** |
| --- | --- | --- | --- |
| name | string | Nom unique de l’extension |
| type | "ServiceFabricLinuxNode" ou "ServiceFabricWindowsNode" | Identifie le système d’exploitation sur lequel démarre Service Fabric |
| autoUpgradeMinorVersion | True ou False | Active la mise à niveau automatique des versions mineures du runtime SF |
| publisher | Microsoft.Azure.ServiceFabric | Nom de l’éditeur d’extension Service Fabric |
| clusterEndpont | string | URI : port vers le point de terminaison de gestion |
| nodeTypeRef | string | Nom du type de nœud |
| durabilityLevel | bronze, silver, gold, platinum | Temps alloué pour suspendre l’infrastructure Azure immuable |
| enableParallelJobs | True ou False | Active le calcul de ParallelJobs, par exemple supprimer une machine virtuelle et redémarrer une machine virtuelle dans le même groupe identique en parallèle |
| nicPrefixOverride | string | Préfixe du sous-réseau, par exemple "10.0.0.0/24" |
| commonNames | string[] | Noms communs des certificats de cluster installés |
| x509StoreName | string | Nom du Store où se trouve le certificat de cluster installé |
| typeHandlerVersion | 1.1 | Version de l’extension. Il est recommandé de mettre à niveau la version classique d’extension 1.0 vers la version 1.1 |
| dataPath | string | Chemin d’accès au lecteur utilisé pour enregistrer l’état des données d’application et services système Service Fabric.

## <a name="next-steps"></a>Étapes suivantes

* Consultez [Vue d’ensemble de la fonction « Déployer n’importe où » et comparaison avec les clusters gérés par Azure](service-fabric-deploy-anywhere.md).
* Découvrez plus en détail la [sécurité des clusters](service-fabric-cluster-security.md).
* [Se connecter à distance](service-fabric-cluster-remote-connect-to-azure-cluster-node.md) à une instance de groupe identique spécifique
* [Mettre à jour les valeurs de plages de port RDP](./scripts/service-fabric-powershell-change-rdp-port-range.md) sur des machines virtuelles de cluster après le déploiement
* [Modifier le nom d’utilisateur et le mot de passe administrateur](./scripts/service-fabric-powershell-change-rdp-user-and-pw.md) pour les machines virtuelles de cluster

<!--Image references-->
[NodeTypes]: ./media/service-fabric-cluster-nodetypes/NodeTypes.png
[Resources]: ./media/service-fabric-cluster-nodetypes/Resources.png
[InboundNatPools]: ./media/service-fabric-cluster-nodetypes/InboundNatPools.png
