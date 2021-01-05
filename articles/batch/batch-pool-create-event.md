---
title: Événement de création de pool Azure Batch
description: Informations de référence pour l’événement de création de pool Batch, qui est émis après la création d’un pool. Le contenu du journal fournit des informations générales sur le pool.
ms.topic: reference
ms.date: 10/08/2020
ms.openlocfilehash: f8b020401443b2a9e80837599d6fd4b2a3001d27
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97609001"
---
# <a name="pool-create-event"></a>Événement de création de pool

 Cet événement est émis après la création d’un pool. Le contenu du journal fournit des informations générales sur le pool. Notez que, si la taille cible du pool est supérieure à 0 nœud de calcul, un événement de démarrage de redimensionnement de pool suit immédiatement cet événement.

 L'exemple suivant montre le corps d'un événement de création de pool pour un pool créé à l'aide de la propriété `CloudServiceConfiguration`.

```
{
    "id": "myPool1",
    "displayName": "Production Pool",
    "vmSize": "Standard_F1s",
    "imageType": "VirtualMachineConfiguration",
    "cloudServiceConfiguration": {
        "osFamily": "3",
        "targetOsVersion": "*"
    },
    "networkConfiguration": {
        "subnetId": " "
    },
    "virtualMachineConfiguration": {
          "imageReference": {
            "publisher": " ",
            "offer": " ",
            "sku": " ",
            "version": " "
          },
          "nodeAgentId": " "
        },
    "resizeTimeout": "300000",
    "targetDedicatedNodes": 2,
    "targetLowPriorityNodes": 2,
    "taskSlotsPerNode": 1,
    "vmFillType": "Spread",
    "enableAutoScale": false,
    "enableInterNodeCommunication": false,
    "isAutoPool": false
}
```

|Élément|Type|Notes|
|-------------|----------|-----------|
|`id`|String|ID du pool.|
|`displayName`|String|Nom d’affichage du pool.|
|`vmSize`|String|Taille des machines virtuelles dans le pool. Toutes les machines virtuelles d’un pool ont la même taille. <br/><br/> Pour plus d’informations sur les tailles disponibles de machines virtuelles pour les pools de Services Cloud (pools créés avec cloudServiceConfiguration), voir [Tailles de Services Cloud](../cloud-services/cloud-services-sizes-specs.md). Le service Batch prend en charge toutes les tailles de machines virtuelles des Services Cloud, à l’exception de `ExtraSmall`.<br/><br/> Pour plus d’informations sur les tailles de machines virtuelles pour les pools utilisant des images de la Place de marché Machines Virtuelles (pools créés avec virtualMachineConfiguration), voir [Tailles de machines virtuelles](../virtual-machines/sizes.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) (Linux) ou [ailles de machines virtuelles](../virtual-machines/sizes.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json) (Windows). Le service Batch prend en charge l’ensemble des tailles de machine virtuelle Azure, à l’exception de `STANDARD_A0` et de celles comprises dans Premium Storage (série `STANDARD_GS`, `STANDARD_DS`, et `STANDARD_DSV2`).|
|`imageType`|String|Méthode de déploiement de l'image. Valeurs prises en charge : `virtualMachineConfiguration` ou `cloudServiceConfiguration`|
|[`cloudServiceConfiguration`](#bk_csconf)|Type complexe|Configuration de service cloud pour le pool.|
|[`virtualMachineConfiguration`](#bk_vmconf)|Type complexe|Configuration de machine virtuelle pour le pool.|
|[`networkConfiguration`](#bk_netconf)|Type complexe|Configuration réseau pour le pool.|
|`resizeTimeout`|Temps|Délai d’attente pour l’allocation de nœuds de calcul au pool spécifié pour la dernière opération de redimensionnement sur le pool.  (dimensionnement initial quand la création du pool est considérée comme un redimensionnement).|
|`targetDedicatedNodes`|Int32|Nombre de nœuds de calcul dédiés demandés pour le pool.|
|`targetLowPriorityNodes`|Int32|Nombre de nœuds de calcul de faible priorité demandés pour le pool.|
|`enableAutoScale`|Bool|Spécifie si la taille du pool s’ajuste automatiquement au fil du temps.|
|`enableInterNodeCommunication`|Bool|Spécifie si le pool est configuré pour une communication directe entre les nœuds.|
|`isAutoPool`|Bool|Spécifie si le pool a été créé via un mécanisme AutoPool du travail.|
|`taskSlotsPerNode`|Int32|Nombre maximal de tâches pouvant s’exécuter simultanément sur un nœud de calcul unique au sein du pool.|
|`vmFillType`|String|Définit la manière dont le service Batch distribue les tâches entre les nœuds de calcul au sein du pool. Les valeurs valides sont Spread ou Pack.|

###  <a name="cloudserviceconfiguration"></a><a name="bk_csconf"></a> cloudServiceConfiguration

|Nom de l'élément|Type|Notes|
|------------------|----------|-----------|
|`osFamily`|String|Famille de systèmes d’exploitation invités d’Azure à installer sur les machines virtuelles au sein du pool.<br /><br /> Les valeurs possibles sont les suivantes :<br /><br /> **2**: famille de systèmes d’exploitation 2, équivalent à Windows Server 2008 R2 SP1.<br /><br /> **3**: famille de systèmes d’exploitation 3, équivalent à Windows Server 2012.<br /><br /> **4**: famille de systèmes d’exploitation 4, équivalent à Windows Server 2012 R2.<br /><br /> Pour plus d’informations, voir [Publications de système d’exploitation invité d’Azure](../cloud-services/cloud-services-guestos-update-matrix.md#releases).|
|`targetOSVersion`|String|Version du système d’exploitation invité d’Azure à installer sur les machines virtuelles du pool.<br /><br /> La valeur par défaut est **\*** qui spécifie la dernière version du système d’exploitation pour la famille spécifiée.<br /><br /> Pour les autres valeurs autorisées, voir [Publications de système d’exploitation invité d’Azure](../cloud-services/cloud-services-guestos-update-matrix.md#releases).|

###  <a name="virtualmachineconfiguration"></a><a name="bk_vmconf"></a> virtualMachineConfiguration

|Nom de l'élément|Type|Notes|
|------------------|----------|-----------|
|[`imageReference`](#bk_imgref)|Type complexe|Spécifie des informations sur la plateforme ou l’image de la Place de marché à utiliser.|
|`nodeAgentId`|String|Référence (SKU) de l’agent du nœud Batch configuré sur le nœud de calcul.|
|[`windowsConfiguration`](#bk_winconf)|Type complexe|Spécifie les paramètres du système d’exploitation Windows sur la machine virtuelle. Cette propriété ne doit pas être spécifiée si la propriété imageReference fait référence à une image de système d’exploitation Linux.|

###  <a name="imagereference"></a><a name="bk_imgref"></a> imageReference

|Nom de l'élément|Type|Notes|
|------------------|----------|-----------|
|`publisher`|String|Éditeur de l’image.|
|`offer`|String|Offre de l’image.|
|`sku`|String|Référence (SKU) de l’image.|
|`version`|String|La version de l’image.|

###  <a name="windowsconfiguration"></a><a name="bk_winconf"></a> windowsConfiguration

|Nom de l'élément|Type|Notes|
|------------------|----------|-----------|
|`enableAutomaticUpdates`|Boolean|Indique si la machine virtuelle est activée pour les mises à jour automatiques. Si cette propriété n’est pas spécifiée, la valeur par défaut est true.|

###  <a name="networkconfiguration"></a><a name="bk_netconf"></a> networkConfiguration

|Nom de l'élément|Type|Notes|
|------------------|--------------|----------|
|`subnetId`|String|Spécifie l’identificateur de ressource du sous-réseau dans lequel les nœuds de calcul du pool sont créés.|