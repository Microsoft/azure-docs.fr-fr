---
title: Créer un pool avec le chiffrement de disque activé
description: Découvrez comment utiliser la configuration du chiffrement de disque pour chiffrer des nœuds avec une clé gérée par la plateforme.
author: pkshultz
ms.topic: how-to
ms.date: 10/08/2020
ms.author: peshultz
ms.custom: references_regions
ms.openlocfilehash: a61e87c660bf2d2f0f4c8d02bd1699c58f8da667
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96350668"
---
# <a name="create-a-pool-with-disk-encryption-enabled"></a>Créer un pool avec le chiffrement de disque activé

Quand vous créez un pool Azure Batch en utilisant une configuration de machine virtuelle, vous pouvez chiffrer des nœuds de calcul dans le pool avec une clé gérée par la plateforme en spécifiant la configuration du chiffrement de disque.

Cet article explique comment créer un pool Batch avec le chiffrement de disque activé.

## <a name="why-use-a-pool-with-disk-encryption-configuration"></a>Pourquoi utiliser un pool avec une configuration du chiffrement de disque ?

Avec un pool Batch, vous pouvez accéder aux données et les stocker sur le système d’exploitation et des disques temporaires du nœud de calcul. Le chiffrement du disque côté serveur à l’aide d’une clé gérée par la plateforme va permettre de protéger ces données avec une faible surcharge et de façon très pratique.

Batch va appliquer l’une de ces technologies de chiffrement de disque sur les nœuds de calcul, en fonction de la configuration du pool et de la prise en charge régionale.

- [Chiffrement de disque managé au repos avec des clés gérées par la plateforme](../virtual-machines/disk-encryption.md#platform-managed-keys)
- [Chiffrement sur l’hôte à l’aide d’une clé gérée par la plateforme](../virtual-machines/disk-encryption.md#encryption-at-host---end-to-end-encryption-for-your-vm-data)
- [Azure Disk Encryption](../security/fundamentals/azure-disk-encryption-vms-vmss.md)

> [!IMPORTANT]
> La prise en charge du chiffrement sur l’hôte à l’aide d’une clé gérée par la plateforme dans Azure Batch est actuellement en préversion publique pour les régions USA Est, USA Ouest 2, USA Centre Sud, US Gov Virginie et US Gov Arizona.
> Cette préversion est fournie sans contrat de niveau de service et n’est pas recommandée pour les charges de travail de production. Certaines fonctionnalités peuvent être limitées ou non prises en charge.
> Pour plus d’informations, consultez [Conditions d’Utilisation Supplémentaires relatives aux Évaluations Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Vous ne pouvez pas spécifier la méthode de chiffrement à appliquer aux nœuds de votre pool. Au lieu de cela, vous indiquez les disques cibles à chiffrer sur leurs nœuds, et Batch peut choisir la méthode de chiffrement appropriée, en veillant à ce que les disques spécifiés soient chiffrés sur le nœud de calcul.

## <a name="azure-portal"></a>Portail Azure

Quand vous créez un pool Batch dans le portail Azure, sélectionnez **TemporaryDisk** ou **OsAndTemporaryDisk** sous **Configuration du chiffrement de disque**.

:::image type="content" source="media/disk-encryption/portal-view.png" alt-text="Capture d’écran de l’option Configuration du chiffrement de disque dans le portail Azure.":::

Une fois le pool créé, vous pouvez voir les cibles de configuration du chiffrement de disque dans la section **Propriétés** du pool.

:::image type="content" source="media/disk-encryption/configuration-target.png" alt-text="Capture d’écran montrant les cibles de configuration du chiffrement de disque dans le portail Azure.":::

## <a name="examples"></a>Exemples

Les exemples suivants montrent comment chiffrer le système d’exploitation et les disques temporaires sur un pool Batch à l’aide du SDK Batch pour .NET, de l’API REST Batch et d’Azure CLI.

### <a name="batch-net-sdk"></a>SDK Batch pour .NET

```csharp
pool.VirtualMachineConfiguration.DiskEncryptionConfiguration = new DiskEncryptionConfiguration(
    targets: new List<DiskEncryptionTarget> { DiskEncryptionTarget.OsDisk, DiskEncryptionTarget.TemporaryDisk }
    );
```

### <a name="batch-rest-api"></a>API REST Batch

URL DE L’API REST :
```
POST {batchURL}/pools?api-version=2020-03-01.11.0
client-request-id: 00000000-0000-0000-0000-000000000000
```
Corps de la requête :
```
"pool": {
    "id": "pool2",
    "vmSize": "standard_a1",
    "virtualMachineConfiguration": {
        "imageReference": {
            "publisher": "Canonical",
            "offer": "UbuntuServer",
            "sku": "18.04-LTS"
        },
        "diskEncryptionConfiguration": {
            "targets": [
                "OsDisk",
                "TemporaryDisk"
            ]
        }
        "nodeAgentSKUId": "batch.node.ubuntu 18.04"
    },
    "resizeTimeout": "PT15M",
    "targetDedicatedNodes": 5,
    "targetLowPriorityNodes": 0,
    "taskSlotsPerNode": 3,
    "enableAutoScale": false,
    "enableInterNodeCommunication": false
}
```

### <a name="azure-cli"></a>Azure CLI

```azurecli-interactive
az batch pool create \
    --id diskencryptionPool \
    --vm-size Standard_DS1_V2 \
    --target-dedicated-nodes 2 \
    --image canonical:ubuntuserver:18.04-LTS \
    --node-agent-sku-id "batch.node.ubuntu 18.04" \
    --disk-encryption-targets OsDisk TemporaryDisk
```

## <a name="next-steps"></a>Étapes suivantes

- Découvrez-en plus sur le [chiffrement côté serveur de Stockage sur disque Azure](../virtual-machines/disk-encryption.md).
- Pour obtenir une vue d’ensemble détaillée de Batch, consultez [flux de travail et ressources du service Batch](batch-service-workflow-features.md).