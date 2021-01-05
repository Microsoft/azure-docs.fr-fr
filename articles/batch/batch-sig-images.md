---
title: Création d’un pool d’images personnalisées avec Shared Image Gallery
description: Les pools d’images personnalisées représentent un moyen efficace de configurer les nœuds de calcul pour exécuter des charges de travail Batch.
ms.topic: conceptual
ms.date: 11/18/2020
ms.custom: devx-track-python, devx-track-azurecli
ms.openlocfilehash: eb21a9e0d355274142e34fbb5c90a4d293c88ef1
ms.sourcegitcommit: 4295037553d1e407edeb719a3699f0567ebf4293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96327302"
---
# <a name="use-the-shared-image-gallery-to-create-a-custom-image-pool"></a>Création d’un pool d’images personnalisées avec Shared Image Gallery

Quand vous créez un pool Azure Batch à l’aide de Configuration de la machine virtuelle, vous spécifiez une image de machine virtuelle qui fournit le système d’exploitation pour chaque nœud de calcul dans le pool. Vous pouvez créer un pool de machines virtuelles en utilisant une image prise en charge de la Place de Marché Azure ou en élaborant une image personnalisée avec une [image Shared Image Gallery](../virtual-machines/windows/shared-image-galleries.md).

## <a name="benefits-of-the-shared-image-gallery"></a>Avantages de Shared Image Gallery

Lorsque vous utilisez Shared Image Gallery pour votre image personnalisée, vous contrôlez le type et la configuration du système d’exploitation ainsi que le type de disque de données. Votre image partagée peut inclure des applications et des données de référence qui deviennent disponibles sur tous les nœuds du pool Batch dès qu’ils sont approvisionnés.

Vous pouvez également avoir plusieurs versions d’une image en fonction des besoins de votre environnement. Quand vous utilisez une version d’image pour créer une machine virtuelle, la version d’image permet de créer des disques pour la machine virtuelle.

L’utilisation d’une image partagée réduit le temps de préparation des nœuds de calcul du pool appelés à exécuter votre charge de travail Batch. Il est possible d’utiliser une image de la Place de Marché et d’installer des logiciels sur chaque nœud de calcul après l’approvisionnement, mais l’utilisation d’une image partagée est généralement plus efficace. En outre, vous pouvez spécifier plusieurs réplicas pour l’image partagée. Ainsi, vous gagnerez du temps lors de la création de pools avec de nombreuses machines virtuelles (plus de 600 machines virtuelles).

Le recours à une image partagée pour votre scénario peut offrir plusieurs avantages :

- **Vous pouvez utiliser les mêmes images dans toutes les régions.** Vous pouvez créer des réplicas d’images partagées dans différentes régions afin que tous vos pools utilisent la même image.
- **Configurer le système d’exploitation.** Vous pouvez personnaliser la configuration du disque de système d’exploitation de l’image.
- **Préinstaller des applications.** La préinstallation des applications sur le disque du système d’exploitation est plus efficace et moins sujet aux erreurs que l’installation d’applications après l’approvisionnement des nœuds de calcul à l’aide d’un début de tâche.
- **Copier de grandes quantités de données en une seule fois.** Intégrez les données statiques à l’image partagée managée en les copiant sur les disques de données d’une image managée. Cette opération ne doit être effectuée qu’une seule fois et rend les données accessibles à chaque nœud du pool.
- **Augmentez la taille des pools.** Avec Shared Image Gallery, vous pouvez créer des pools plus grands avec vos images personnalisées ainsi qu’avec d’autres réplicas d’image partagée.
- **Amélioration des performances par rapport au recours à une seule image managée comme image personnalisée.** Pour un pool d’images personnalisées Shared Image, le temps nécessaire pour atteindre l’état stable peut être réduit de 25 % et la latence d’inactivité de la machine virtuelle peut être réduite de 30 %.
- **Gestion des versions et regroupement d’images pour une gestion simplifiée.** La définition du regroupement d’images contient des informations sur la raison pour laquelle l’image a été créée, le système d’exploitation concerné et l’utilisation de l’image. Le regroupement d’images simplifie la gestion des images. Pour plus d’informations, consultez les [Définitions d’images](../virtual-machines/windows/shared-image-galleries.md#image-definitions).

## <a name="prerequisites"></a>Prérequis

> [!NOTE]
> Vous devez vous authentifier à l’aide d’Azure AD. Si vous utilisez shared-key-auth, une erreur d’authentification se produit.  

- **Un compte Azure Batch.** Pour créer un compte Batch, consultez les démarrages rapides Batch à l’aide du [portail Azure](quick-create-portal.md) ou de l’[interface de ligne de commande Azure](quick-create-cli.md).

- **Une image Shared Image Gallery**. Pour créer une image partagée, vous devez disposer d'une image managée ou en créer une. L’image doit être créée à partir d’instantanés du disque de système d’exploitation de la machine virtuelle et, éventuellement, ses disques de données associés.

> [!NOTE]
> Si l’image partagée n’est pas dans le même abonnement que le compte Batch, vous devez [inscrire le fournisseur de ressources Microsoft.Batch](../azure-resource-manager/management/resource-providers-and-types.md#register-resource-provider) pour cet abonnement. Les deux abonnements doivent se trouver dans le même locataire Azure AD.
>
> L’image peut se trouver dans une autre région, à condition qu’elle ait des réplicas dans la même région que votre compte Batch.

Si vous utilisez une application Azure AD pour créer un pool d’images personnalisées avec une image Shared Image Gallery, l’application doit disposer d’un [rôle intégré Azure](../role-based-access-control/rbac-and-directory-admin-roles.md#azure-roles) qui lui donne accès à l’image Shared Image. Vous pouvez accorder cet accès dans le Portail Azure en accédant à l’image Shared Image, en sélectionnant **Contrôle d’accès (IAM)** et en ajoutant une attribution de rôle pour l’application.

## <a name="prepare-a-shared-image"></a>Préparation d’une image Shared Image

Dans Azure, il est possible de préparer une image partagée à partir d’une image managée, qui peut être créée à partir de différentes sources :

- Captures instantanées du système d'exploitation et des disques de données d'une machine virtuelle Azure
- Machine virtuelle Azure généralisée dotée de disques managés
- Disque dur virtuel local généralisé chargé dans le cloud

> [!NOTE]
> Batch prend uniquement en charge les images partagées généralisées ; une image partagée spécialisée ne peut pas être utilisée pour créer un pool.

Les étapes suivantes expliquent comment préparer une machine virtuelle, en prendre une capture instantanée et créer une image à partir de celle-ci.

### <a name="prepare-a-vm"></a>Préparer une machine virtuelle

Si vous créez une machine virtuelle pour l'image, utilisez une image propriétaire de la Place de Marché Azure prise en charge par Batch comme image de base pour votre image managée. Seules les images propriétaires peuvent être utilisées comme image de base. Pour obtenir la liste complète des références d'image de la Place de marché Azure prises en charge par Azure Batch, consultez l'opération [Lister les références SKU d'agent de nœud](/java/api/com.microsoft.azure.batch.protocol.accounts.listnodeagentskus).

> [!NOTE]
> Vous ne pouvez pas, comme image de base, utiliser une image de fournisseurs tiers qui comporte des conditions de licence et d’achat supplémentaires. Pour plus d’informations sur ces images de la Place de marché, consultez les recommandations émises pour les machines virtuelles [Linux](../virtual-machines/linux/cli-ps-findimage.md#deploy-an-image-with-marketplace-terms) ou [Windows](../virtual-machines/windows/cli-ps-findimage.md#deploy-an-image-with-marketplace-terms).

- Assurez-vous que la machine virtuelle est créée avec un disque managé. Il s’agit du paramètre de stockage par défaut quand vous créez une machine virtuelle.
- N’installez pas d’extensions Azure, comme l’extension de script personnalisé, sur la machine virtuelle. Si l’image contient une extension préinstallée, Azure peut rencontrer des problèmes lors du déploiement du pool Batch.
- Lorsque vous utilisez des disques de données attachés, vous devez monter et formater les disques à partir d'une machine virtuelle pour les utiliser.
- Vérifiez que l’image du système d’exploitation de base que vous fournissez utilise le lecteur temporaire par défaut. L’agent de nœud Batch s’attend actuellement à ce que le lecteur temporaire par défaut soit utilisé.
- Assurez-vous que le disque du système d'exploitation n'est pas chiffré.
- Une fois que la machine virtuelle s’exécute, connectez-la via le protocole RDP (pour Windows) ou SSH (pour Linux). Le cas échéant, installez les logiciels nécessaires ou copiez les données souhaitées.  

### <a name="create-a-vm-snapshot"></a>Créer un instantané de la machine virtuelle

Une capture instantanée est une copie complète en lecture seule d’un disque dur virtuel. Pour créer un instantané des disques de système d’exploitation ou de données d’une machine virtuelle, vous pouvez utiliser les outils en ligne de commande ou le portail Azure. Pour les étapes et les options permettant de créer un instantané, consultez les conseils pour les machines virtuelles [Linux](../virtual-machines/linux/snapshot-copy-managed-disk.md) ou [Windows](../virtual-machines/windows/snapshot-copy-managed-disk.md).

### <a name="create-an-image-from-one-or-more-snapshots"></a>Créer une image à partir d’un ou plusieurs instantanés

Pour créer une image managée à partir d’un instantané, utilisez les outils en ligne de commande Azure comme la commande [az image création](/cli/azure/image). Créez une image en spécifiant l'instantané d'un disque du système d'exploitation et éventuellement un ou plusieurs instantanés de disque de données.

### <a name="create-a-shared-image-gallery"></a>Créer une galerie Shared Image Gallery

Après avoir créé votre image managée, vous devez créer une galerie Shared Image Gallery pour rendre votre image personnalisée disponible. Pour apprendre à créer une galerie Shared Image Gallery pour vos images, consultez [Créer une galerie Shared Image Gallery avec l'interface de ligne de commande Azure](../virtual-machines/shared-images-cli.md) ou [Créer une galerie Shared Image Gallery à l'aide du portail Azure](../virtual-machines/linux/shared-images-portal.md).

## <a name="create-a-pool-from-a-shared-image-using-the-azure-cli"></a>Créer un pool à partir d’une image partagée à l’aide de l’interface de ligne de commande Azure

Pour créer un pool à partir de votre image partagée à l’aide de l’interface de ligne de commande Azure, utilisez la commande `az batch pool create`. Spécifiez l’ID de l’image partagée dans le champ `--image`. Vérifiez que le type de système d’exploitation et la référence SKU correspondent aux versions spécifiées par `--node-agent-sku-id`

> [!NOTE]
> Vous devez vous authentifier à l’aide d’Azure AD. Si vous utilisez shared-key-auth, une erreur d’authentification se produit.  

```azurecli
az batch pool create \
    --id mypool --vm-size Standard_A1_v2 \
    --target-dedicated-nodes 2 \
    --image "/subscriptions/{sub id}/resourceGroups/{resource group name}/providers/Microsoft.Compute/galleries/{gallery name}/images/{image definition name}/versions/{version id}" \
    --node-agent-sku-id "batch.node.ubuntu 16.04"
```

## <a name="create-a-pool-from-a-shared-image-using-c"></a>Créer un pool à partir d’une image partagée à l’aide de C#

Vous pouvez également créer un pool à partir d’une image partagée à l'aide du kit de développement logiciel (SDK) C#.

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
        virtualMachineImageId: "/subscriptions/{sub id}/resourceGroups/{resource group name}/providers/Microsoft.Compute/galleries/{gallery name}/images/{image definition name}/versions/{version id}");
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
    ...
}
```

## <a name="create-a-pool-from-a-shared-image-using-python"></a>Créer un pool à partir d’une image partagée en Python

Vous pouvez également créer un pool à partir d’une image partagée à l’aide du kit SDK Python : 

```python
# Import the required modules from the
# Azure Batch Client Library for Python
import azure.batch as batch
import azure.batch.models as batchmodels
from azure.common.credentials import ServicePrincipalCredentials

# Specify Batch account and service principal account credentials
account = "{batch-account-name}"
batch_url = "{batch-account-url}"
ad_client_id = "{sp-client-id}"
ad_tenant = "{tenant-id}"
ad_secret = "{sp-secret}"

# Pool settings
pool_id = "LinuxNodesSamplePoolPython"
vm_size = "STANDARD_D2_V3"
node_count = 1

# Initialize the Batch client with Azure AD authentication
creds = ServicePrincipalCredentials(
    client_id=ad_client_id,
    secret=ad_secret,
    tenant=ad_tenant,
    resource="https://batch.core.windows.net/"
)
client = batch.BatchServiceClient(creds, batch_url)

# Configure the start task for the pool
start_task = batchmodels.StartTask(
    command_line="printenv AZ_BATCH_NODE_STARTUP_DIR"
)
start_task.run_elevated = True

# Create an ImageReference which specifies the image from
# Shared Image Gallery to install on the nodes.
ir = batchmodels.ImageReference(
    virtual_machine_image_id="/subscriptions/{sub id}/resourceGroups/{resource group name}/providers/Microsoft.Compute/galleries/{gallery name}/images/{image definition name}/versions/{version id}"
)

# Create the VirtualMachineConfiguration, specifying
# the VM image reference and the Batch node agent to
# be installed on the node.
vmc = batchmodels.VirtualMachineConfiguration(
    image_reference=ir,
    node_agent_sku_id="batch.node.ubuntu 18.04"
)

# Create the unbound pool
new_pool = batchmodels.PoolAddParameter(
    id=pool_id,
    vm_size=vm_size,
    target_dedicated_nodes=node_count,
    virtual_machine_configuration=vmc,
    start_task=start_task
)

# Create pool in the Batch service
client.pool.add(new_pool)
```

## <a name="create-a-pool-from-a-shared-image-using-the-azure-portal"></a>Créer un pool à partir d’une image partagée à l’aide du portail Azure

Effectuez les étapes suivantes pour créer un pool à partir d’une image partagée dans le portail Azure.

1. Ouvrez le [portail Azure](https://portal.azure.com).
1. Accédez à **Comptes Batch** et sélectionnez votre compte.
1. Sélectionnez **Pools**, puis **Ajouter** pour créer un pool.
1. Dans la section **Type d’image**, sélectionnez **Galerie d’images partagées**.
1. Complétez les sections restantes avec les informations relatives à votre image managée.
1. Sélectionnez **OK**.

![Créez un pool à partir d’une image partagée avec le portail.](media/batch-sig-images/create-custom-pool.png)

## <a name="considerations-for-large-pools"></a>Considérations relatives aux grands pools

Si vous envisagez de créer un pool avec des centaines ou des milliers de machines virtuelles ou plus à l’aide d’une image partagée, suivez les instructions ci-dessous.

- **Nombre de réplicas de la galerie Shared Image Gallery.**  Pour chaque pool de 300 instances maximum, nous vous recommandons de conserver au moins un réplica. Par exemple, si vous créez un pool de 3 000 machines virtuelles, vous devez conserver au moins 10 réplicas de votre image. Nous suggérons toujours de conserver plus de réplicas que le minimum exigé pour de meilleures performances.

- **Délai d'expiration du redimensionnement.** Si votre pool contient un nombre de nœuds fixe (pas de mise à l'échelle automatique), augmentez la propriété `resizeTimeout` du pool en fonction de la taille de celui-ci. Le délai d’attente de redimensionnement recommandé est d’au moins 15 minutes pour 1 000 machines. Par exemple, le délai d’expiration de redimensionnement recommandé pour un pool de 2 000 machines virtuelles est d’au moins 30 minutes.

## <a name="next-steps"></a>Étapes suivantes

- Pour obtenir une vue d’ensemble détaillée de Batch, consultez [Flux de travail et ressources du service Batch](batch-service-workflow-features.md).
- Découvrez la [Galerie d’images partagées](../virtual-machines/windows/shared-image-galleries.md) (Shared Image Gallery).
