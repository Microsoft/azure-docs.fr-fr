---
title: Création d’un pool d’images personnalisées à l’aide d’une image managée
description: Créez un pool d’images personnalisées à partir d’une image managée pour provisionner les nœuds de calcul avec les logiciels et les données de votre application.
ms.topic: conceptual
ms.date: 11/18/2020
ms.openlocfilehash: 0a357a1d8a22341297f3bee73fb0867fb03f374f
ms.sourcegitcommit: f6236e0fa28343cf0e478ab630d43e3fd78b9596
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94916574"
---
# <a name="use-a-managed-image-to-create-a-custom-image-pool"></a>Création d’un pool d’images personnalisées à l’aide d’une image managée

Pour créer un pool d’images personnalisées destiné aux machines virtuelles de votre pool Batch, vous pouvez utiliser une image managée de façon à élaborer une [image Shared Image Gallery](batch-sig-images.md). L’utilisation d’une seule image gérée est également prise en charge, mais uniquement pour les versions d’API allant jusqu’à 2019-08-01.

> [!IMPORTANT]
> Dans la plupart des cas, vous devez créer des images personnalisées à l’aide de Shared Image Gallery. À l’aide de Shared Image Gallery, vous pouvez approvisionner des pools plus rapidement, mettre à l’échelle de grandes quantités de machines virtuelles et améliorer la fiabilité lors de la configuration des machines virtuelles. Pour en savoir plus, consultez [Utiliser Shared Image Gallery pour créer un pool personnalisé](batch-sig-images.md).

Cette rubrique explique comment créer un pool d’images personnalisées en utilisant seulement une image managée.

## <a name="prerequisites"></a>Prérequis

- **Une ressource d’image managée**. Pour créer un pool de machines virtuelles à l’aide d’une image personnalisée, vous devez avoir ou créer une ressource d’image managée dans le même abonnement et la même région Azure que le compte Batch. L’image doit être créée à partir d’instantanés du disque de système d’exploitation de la machine virtuelle et, éventuellement, ses disques de données associés.
  - Utilisez une image personnalisée unique pour chaque pool que vous créez.
  - Pour créer un pool avec l’image à l’aide des API Batch, spécifiez **l’ID de ressource** de l’image, qui est au format `/subscriptions/xxxx-xxxxxx-xxxxx-xxxxxx/resourceGroups/myResourceGroup/providers/Microsoft.Compute/images/myImage`.
  - La ressource d’image managée doit exister pour faire monter la durée de vie du pool en puissance. Elle peut être supprimée une fois que le pool est supprimé.

- **Authentification Azure Active Directory (Azure AD)** . L’API du client Batch doit utiliser l’authentification Azure AD. La prise en charge d’Azure Batch pour Azure AD est documentée dans [Authentifier les solutions de service Batch avec Active Directory](batch-aad-auth.md).

## <a name="prepare-a-managed-image"></a>Préparer une image managée

Dans Azure, vous pouvez préparer une image managée à partir de ce qui suit :

- Captures instantanées du système d'exploitation et des disques de données d'une machine virtuelle Azure
- Machine virtuelle Azure généralisée dotée de disques managés
- Disque dur virtuel local généralisé chargé dans le cloud

Pour mettre à l’échelle des pools Batch de manière fiable avec une image managée, nous vous recommandons de créer une image managée *uniquement* à l’aide de la première méthode : en utilisant des captures instantanées des disques de la machine virtuelle. Les étapes suivantes montrent comment préparer une machine virtuelle, prendre un instantané et créer une image managée à partir de l’instantané.

### <a name="prepare-a-vm"></a>Préparer une machine virtuelle

Si vous créez une machine virtuelle pour l'image, utilisez une image propriétaire de la Place de Marché Azure prise en charge par Batch comme image de base pour votre image managée. Seules les images propriétaires peuvent être utilisées comme image de base. Pour obtenir la liste complète des références d'image de la Place de marché Azure prises en charge par Azure Batch, consultez l'opération [Lister les références SKU d'agent de nœud](/java/api/com.microsoft.azure.batch.protocol.accounts.listnodeagentskus).

> [!NOTE]
> Vous ne pouvez pas, comme image de base, utiliser une image de fournisseurs tiers qui comporte des conditions de licence et d’achat supplémentaires. Pour plus d’informations sur ces images de la Place de marché, consultez les recommandations émises pour les machines virtuelles [Linux](../virtual-machines/linux/cli-ps-findimage.md#deploy-an-image-with-marketplace-terms) ou [Windows](../virtual-machines/windows/cli-ps-findimage.md#deploy-an-image-with-marketplace-terms).

- Assurez-vous que la machine virtuelle est créée avec un disque managé. Il s’agit du paramètre de stockage par défaut quand vous créez une machine virtuelle.
- N’installez pas d’extensions Azure, comme l’extension de script personnalisé, sur la machine virtuelle. Si l’image contient une extension préinstallée, Azure peut rencontrer des problèmes lors du déploiement du pool Batch.
- Lorsque vous utilisez des disques de données attachés, vous devez monter et formater les disques à partir d'une machine virtuelle pour les utiliser.
- Vérifiez que l’image du système d’exploitation de base que vous fournissez utilise le lecteur temporaire par défaut. L’agent de nœud Batch s’attend actuellement à ce que le lecteur temporaire par défaut soit utilisé.
- Assurez-vous que le disque du système d’exploitation n’est pas chiffré.
- Une fois que la machine virtuelle s’exécute, connectez-la via le protocole RDP (pour Windows) ou SSH (pour Linux). Le cas échéant, installez les logiciels nécessaires ou copiez les données souhaitées.  

### <a name="create-a-vm-snapshot"></a>Créer un instantané de la machine virtuelle

Une capture instantanée est une copie complète en lecture seule d’un disque dur virtuel. Pour créer un instantané des disques de système d’exploitation ou de données d’une machine virtuelle, vous pouvez utiliser les outils en ligne de commande ou le portail Azure. Pour les étapes et les options permettant de créer un instantané, consultez les conseils pour les machines virtuelles [Linux](../virtual-machines/linux/snapshot-copy-managed-disk.md) ou [Windows](../virtual-machines/windows/snapshot-copy-managed-disk.md).

### <a name="create-an-image-from-one-or-more-snapshots"></a>Créer une image à partir d’un ou plusieurs instantanés

Pour créer une image managée à partir d’un instantané, utilisez les outils en ligne de commande Azure comme la commande [az image création](/cli/azure/image). Vous pouvez créer une image en spécifiant l’instantané d’un disque du système d’exploitation et éventuellement un ou plusieurs instantanés de disque de données.

## <a name="create-a-pool-from-a-managed-image"></a>Création d’un pool à partir d’une image managée

Une fois que vous avez trouvé l’ID de ressource de votre image managée, créez un pool d’images personnalisé à partir de cette image. Les étapes suivantes vous montrent comment créer un pool d’images personnalisé à l’aide de Batch Service ou de Batch Management.

> [!NOTE]
> Assurez-vous que l’identité que vous utilisez pour l’authentification Azure AD dispose des autorisations sur la ressource d’image. Consultez [Authentifier des solutions de service Batch avec Active Directory](batch-aad-auth.md).
>
> La ressource destinée à l'image managée doit exister pendant toute la durée de vie du pool. Si la ressource sous-jacente est supprimée, le pool ne peut pas être mis à l'échelle.

### <a name="batch-service-net-sdk"></a>Kit de développement logiciel (SDK) Batch Service .NET

```csharp
private static VirtualMachineConfiguration CreateVirtualMachineConfiguration(ImageReference imageReference)
{
    return new VirtualMachineConfiguration(
        imageReference: imageReference,
        nodeAgentSkuId: "batch.node.windows amd64");
}

private static ImageReference CreateImageReference()
{
    return new ImageReference(
        virtualMachineImageId: "/subscriptions/{sub id}/resourceGroups/{resource group name}/providers/Microsoft.Compute/images/{image definition name}");
}

private static void CreateBatchPool(BatchClient batchClient, VirtualMachineConfiguration vmConfiguration)
{
    try
    {
        CloudPool pool = batchClient.PoolOperations.CreatePool(
            poolId: PoolId,
            targetDedicatedComputeNodes: PoolNodeCount,
            virtualMachineSize: PoolVMSize,
            virtualMachineConfiguration: vmConfiguration);

        pool.Commit();
    }
```

### <a name="batch-management-rest-api"></a>API REST Batch Management

URI de l’API REST

```http
 PUT https://management.azure.com/subscriptions/{sub id}/resourceGroups/{resource group name}/providers/Microsoft.Batch/batchAccounts/{account name}/pools/{pool name}?api-version=2020-03-01
```

Corps de la requête

```json
 {
   "properties": {
     "vmSize": "{VM size}",
     "deploymentConfiguration": {
       "virtualMachineConfiguration": {
         "imageReference": {
           "id": "/subscriptions/{sub id}/resourceGroups/{resource group name}/providers/Microsoft.Compute/images/{image name}"
         },
         "nodeAgentSkuId": "{Node Agent SKU ID}"
       }
     }
   }
 }
```

## <a name="considerations-for-large-pools"></a>Considérations relatives aux grands pools

Si vous envisagez de créer un pool avec des centaines de machines virtuelles ou plus, à l’aide d’une image personnalisée, il est important de suivre les instructions précédentes pour utiliser une image créée à partir d’un instantané de machine virtuelle.

Tenez également compte des points suivants :

- **Limites de taille** : Batch limite la taille du pool à 2 500 nœuds de calcul dédiés ou 1 000 nœuds de faible priorité, lorsque vous utilisez une image personnalisée.

  Si vous utilisez la même image (ou plusieurs images basées sur le même instantané sous-jacent) pour créer plusieurs pools, les nœuds de calcul totaux dans les pools ne peuvent pas dépasser les limites précédentes. Nous ne recommandons pas l’utilisation d’une image ou de son instantané sous-jacent au-delà d’un seul pool.

  Les limites peuvent être réduites si vous configurez le pool avec des [pools NAT de trafic entrant](pool-endpoint-configuration.md).

- **Redimensionner le délai d’attente** : si votre pool contient un nombre de nœuds fixe (pas de mise à l’échelle), augmentez la propriété de l’élément resizeTimeout du pool jusqu’à une valeur telle que 20 à 30 minutes. Si votre pool n’atteint pas sa taille cible dans le délai imparti, effectuez une autre [opération de redimensionnement](/rest/api/batchservice/pool/resize).

  Si vous prévoyez un pool avec plus de 300 nœuds de calcul, vous devrez peut-être redimensionner le pool plusieurs fois afin d’atteindre la taille cible.
  
À l’aide de [Shared Image Gallery](batch-sig-images.md), vous pouvez créer des pools plus grands avec vos images personnalisées ainsi qu’avec d’autres réplicas d’image partagée. À l’aide d’images partagées, le pool atteint jusqu’à 25 % plus rapidement l’état stable et la latence d’inactivité de la machine virtuelle est inférieure à 30 %.

## <a name="considerations-for-using-packer"></a>Considérations relatives à l'utilisation de Packer

Pour créer directement une ressource d'image managée à l'aide de Packer, vous devez impérativement utiliser un compte Batch en mode Abonnement utilisateur. Avec un compte en mode Service Batch, vous devez d'abord créer un disque dur virtuel, puis l'importer dans une ressource d'image managée. En fonction du mode d'allocation de pool (Abonnement utilisateur ou Service Batch), la procédure de création d'une ressource d'image managée varie.

Veillez à ce que la ressource utilisée pour créer l'image managée existe pendant toute la durée de vie du pool qui référence l'image personnalisée, faute de quoi des échecs d'allocation de pool et/ou des échecs de redimensionnement peuvent survenir.

Si l'image ou la ressource sous-jacente est supprimée, une erreur semblable à la suivante peut apparaître : `There was an error encountered while performing the last resize on the pool. Please try resizing the pool again. Code: AllocationFailed`. Si vous obtenez cette erreur, assurez-vous que la ressource sous-jacente n’a pas été supprimée.

Pour plus d'informations sur l'utilisation de Packer pour créer une machine virtuelle, consultez [Créer une image Linux avec Packer](../virtual-machines/linux/build-image-with-packer.md) ou [Créer une image Windows avec Packer](../virtual-machines/windows/build-image-with-packer.md).

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment utiliser [Shared Image Gallery](batch-sig-images.md) pour créer un pool personnalisé.
- Pour obtenir une vue d’ensemble détaillée de Batch, consultez [flux de travail et ressources du service Batch](batch-service-workflow-features.md).
