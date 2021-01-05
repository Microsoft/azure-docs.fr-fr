---
title: Exécuter Linux sur des nœuds de calcul de machine virtuelle
description: Découvrez comment traiter des charges de travail de calcul parallèles sur des pools de machines virtuelles Linux dans Azure Batch.
ms.topic: how-to
ms.date: 11/10/2020
ms.custom: H1Hack27Feb2017, devx-track-python, devx-track-csharp
ms.openlocfilehash: 0a9c801a13af05f077b87f296992da7f50742e4b
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94533495"
---
# <a name="provision-linux-compute-nodes-in-batch-pools"></a>Configurer des nœuds de calcul Linux dans des pools Batch

Vous pouvez utiliser Azure Batch pour exécuter des charges de travail de calcul parallèles sur les machines virtuelles Linux et Windows. Cet article explique comment créer des pools de nœuds de calcul Linux dans le service Batch à l’aide de bibliothèques clientes [Batch Python](https://pypi.python.org/pypi/azure-batch) et [Batch .NET](/dotnet/api/microsoft.azure.batch). 

## <a name="virtual-machine-configuration"></a>Configuration de la machine virtuelle

Lorsque vous créez un pool de nœuds de calcul dans Batch, vous avez deux options pour sélectionner la taille du nœud et le système d’exploitation : Configuration des services cloud et Configuration de la machine virtuelle. La plupart des pools de nœuds de calcul Windows utilisent la [configuration des services cloud](nodes-and-pools.md#cloud-services-configuration), qui spécifie que le pool est composé de nœuds Azure Cloud Services. Ces pools fournissent uniquement des nœuds de calcul Windows.

En revanche, la [configuration d’ordinateur virtuel](nodes-and-pools.md#virtual-machine-configuration) spécifie que le pool est composé de machines virtuelles Azure, qui peuvent être créées à partir d’images Linux ou Windows. Lorsque vous créez un pool avec une configuration d’ordinateur virtuel, vous devez spécifier une [taille de nœud de calcul disponible](../virtual-machines/sizes.md), la référence d’image de machine virtuelle et la référence SKU de l’agent de nœud Batch (un programme qui s’exécute sur chaque nœud et fournit une interface entre le nœud et le service batch) et la référence d’image de machine virtuelle qui sera installée sur les nœuds.

### <a name="virtual-machine-image-reference"></a>Référence de l’image de la machine virtuelle

Le service Batch utilise des [groupes de machines virtuelles identiques](../virtual-machine-scale-sets/overview.md) pour fournir des nœuds de calcul dans la configuration de la machine virtuelle. Vous pouvez spécifier une image à partir de la [Place de marché Azure](https://azuremarketplace.microsoft.com/marketplace/apps/category/compute?filters=virtual-machine-images&page=1) ou [utiliser Shared Image Gallery pour préparer une image personnalisée](batch-sig-images.md).

Lorsque vous créez une référence d’image de machine virtuelle, vous devez spécifier les propriétés suivantes :

| **Propriété de référence d’image** | **Exemple** |
| --- | --- |
| Serveur de publication |Canonical |
| Offre |UbuntuServer |
| SKU |18.04-LTS |
| Version |latest |

> [!TIP]
> Pour plus d’informations sur ces propriétés et sur la manière de spécifier des images de la Place de marché dans [Rechercher des images de machine virtuelle Linux sur la Place de marché Microsoft Azure avec Azure CLI](../virtual-machines/linux/cli-ps-findimage.md). Notez que toutes les images Marketplace ne sont pas compatibles avec Batch pour le moment.

### <a name="node-agent-sku"></a>Référence de l’agent de nœud

[L’agent de nœud Batch](https://github.com/Azure/Batch/blob/master/changelogs/nodeagent/CHANGELOG.md) est un programme qui s’exécute sur chacun des nœuds du pool. Il constitue l’interface de commande et de contrôle entre le nœud et le service Batch. Il existe différentes implémentations de l’agent de nœud pour différents systèmes d’exploitation, connues sous le nom de références. Essentiellement, lorsque vous créez une configuration de machine virtuelle, vous spécifiez d’abord la référence de l’image de la machine virtuelle, puis spécifiez l’agent de nœud à installer sur l’image. En règle générale, chaque référence d’agent de nœud est compatible avec plusieurs images de machine virtuelle. Voici quelques exemples de références d’agent de nœud :

- batch.node.ubuntu 18.04
- batch.node.centos 7
- batch.node.windows amd64

### <a name="list-of-virtual-machine-images"></a>liste des images de machine virtuelle

Toutes les images de la Place de marché ne sont pas compatibles avec les agents de nœud Batch actuellement disponibles. Pour répertorier la liste de toutes les images de machine virtuelle de la Place de marché prises en charge pour le service Batch et les références SKU des agents de nœud correspondants, utilisez [list_supported_images](/python/api/azure-batch/azure.batch.operations.AccountOperations#list-supported-images-account-list-supported-images-options-none--custom-headers-none--raw-false----operation-config-) (Python), [ListSupportedImages](/dotnet/api/microsoft.azure.batch.pooloperations.listsupportedimages) (Batch.NET) ou l’API correspondante dans le Kit de développement logiciel (SDK) pour l’autre langue.

## <a name="create-a-linux-pool-batch-python"></a>Création d’un pool Linux : Python Batch

L’extrait de code suivant offre un exemple d’utilisation de la [bibliothèque cliente Microsoft Azure Batch pour Python](https://pypi.python.org/pypi/azure-batch) pour créer un pool de nœuds de calcul de serveur Ubuntu. Pour plus d’informations sur le module Batch Python, consultez la [documentation de référence](/python/api/overview/azure/batch).

Cet extrait de code crée explicitement un paramètre [ImageReference](/python/api/azure-mgmt-batch/azure.mgmt.batch.models.imagereference) et spécifie chacune de ses propriétés (éditeur, offre, référence SKU, version). Toutefois, dans un code de production, nous vous recommandons toutefois d’utiliser la méthode [list_supported_images](/python/api/azure-batch/azure.batch.operations.AccountOperations#list-supported-images-account-list-supported-images-options-none--custom-headers-none--raw-false----operation-config-) pour opérer une sélection parmi les combinaisons disponibles de références SKU d’image et de nœud d’agent au moment du runtime.

```python
# Import the required modules from the
# Azure Batch Client Library for Python
import azure.batch.batch_service_client as batch
import azure.batch.batch_auth as batchauth
import azure.batch.models as batchmodels

# Specify Batch account credentials
account = "<batch-account-name>"
key = "<batch-account-key>"
batch_url = "<batch-account-url>"

# Pool settings
pool_id = "LinuxNodesSamplePoolPython"
vm_size = "STANDARD_D2_V3"
node_count = 1

# Initialize the Batch client
creds = batchauth.SharedKeyCredentials(account, key)
config = batch.BatchServiceClientConfiguration(creds, batch_url)
client = batch.BatchServiceClient(creds, batch_url)

# Create the unbound pool
new_pool = batchmodels.PoolAddParameter(id=pool_id, vm_size=vm_size)
new_pool.target_dedicated = node_count

# Configure the start task for the pool
start_task = batchmodels.StartTask()
start_task.run_elevated = True
start_task.command_line = "printenv AZ_BATCH_NODE_STARTUP_DIR"
new_pool.start_task = start_task

# Create an ImageReference which specifies the Marketplace
# virtual machine image to install on the nodes
ir = batchmodels.ImageReference(
    publisher="Canonical",
    offer="UbuntuServer",
    sku="18.04-LTS",
    version="latest")

# Create the VirtualMachineConfiguration, specifying
# the VM image reference and the Batch node agent
# to install on the node
vmc = batchmodels.VirtualMachineConfiguration(
    image_reference=ir,
    node_agent_sku_id="batch.node.ubuntu 18.04")

# Assign the virtual machine configuration to the pool
new_pool.virtual_machine_configuration = vmc

# Create pool in the Batch service
client.pool.add(new_pool)
```

Comme nous l’avons indiqué, nous vous recommandons d’utiliser la méthode [list_supported_images](/python/api/azure-batch/azure.batch.operations.AccountOperations#list-supported-images-account-list-supported-images-options-none--custom-headers-none--raw-false----operation-config-) afin d’opérer une sélection de manière dynamique parmi les combinaisons d’image d’agent de nœud/marketplace actuellement prises en charge (au lieu de créer explicitement une [ImageReference](/python/api/azure-mgmt-batch/azure.mgmt.batch.models.imagereference)). L’extrait de code Python suivant illustre l’utilisation de cette méthode.

```python
# Get the list of supported images from the Batch service
images = client.account.list_supported_images()

# Obtain the desired image reference
image = None
for img in images:
  if (img.image_reference.publisher.lower() == "canonical" and
        img.image_reference.offer.lower() == "ubuntuserver" and
        img.image_reference.sku.lower() == "18.04-lts"):
    image = img
    break

if image is None:
  raise RuntimeError('invalid image reference for desired configuration')

# Create the VirtualMachineConfiguration, specifying the VM image
# reference and the Batch node agent to be installed on the node
vmc = batchmodels.VirtualMachineConfiguration(
    image_reference=image.image_reference,
    node_agent_sku_id=image.node_agent_sku_id)
```

## <a name="create-a-linux-pool-batch-net"></a>Création d’un pool Linux : .NET Batch

L’extrait de code suivant offre un exemple d’utilisation de la bibliothèque cliente [Batch .NET](https://www.nuget.org/packages/Microsoft.Azure.Batch/) pour créer un pool de nœuds de calcul de serveur Ubuntu. Pour plus d’informations sur Batch .NET, consultez la [documentation de référence](/dotnet/api/microsoft.azure.batch).

L’extrait de code suivant utilise la méthode [PoolOperations.ListSupportedImages](/dotnet/api/microsoft.azure.batch.pooloperations.listsupportedimages) pour opérer une sélection dans la liste des combinaisons d’image de la Place de marché et de référence SKU d’agent de nœud actuellement prises en charge. Cette technique est recommandée car la liste des combinaisons prises en charge peut changer de temps à autre. En règle générale, les combinaisons prises en charge sont ajoutées.

```csharp
// Pool settings
const string poolId = "LinuxNodesSamplePoolDotNet";
const string vmSize = "STANDARD_D2_V3";
const int nodeCount = 1;

// Obtain a collection of all available node agent SKUs.
// This allows us to select from a list of supported
// VM image/node agent combinations.
List<ImageInformation> images =
    batchClient.PoolOperations.ListSupportedImages().ToList();

// Find the appropriate image information
ImageInformation image = null;
foreach (var img in images)
{
    if (img.ImageReference.Publisher == "Canonical" &&
        img.ImageReference.Offer == "UbuntuServer" &&
        img.ImageReference.Sku == "18.04-LTS")
    {
        image = img;
        break;
    }
}

// Create the VirtualMachineConfiguration for use when actually
// creating the pool
VirtualMachineConfiguration virtualMachineConfiguration =
    new VirtualMachineConfiguration(image.ImageReference, image.NodeAgentSkuId);

// Create the unbound pool object using the VirtualMachineConfiguration
// created above
CloudPool pool = batchClient.PoolOperations.CreatePool(
    poolId: poolId,
    virtualMachineSize: vmSize,
    virtualMachineConfiguration: virtualMachineConfiguration,
    targetDedicatedComputeNodes: nodeCount);

// Commit the pool to the Batch service
await pool.CommitAsync();
```

Bien que l’extrait de code ci-dessus utilise la méthode [PoolOperations.istSupportedImages](/dotnet/api/microsoft.azure.batch.pooloperations.listsupportedimages) pour répertorier et sélectionner de manière dynamique des combinaisons d’images et de références SKU d’agent de nœud prises en charge (recommandé), vous pouvez également configurer explicitement [ImageReference](/dotnet/api/microsoft.azure.batch.imagereference) :

```csharp
ImageReference imageReference = new ImageReference(
    publisher: "Canonical",
    offer: "UbuntuServer",
    sku: "18.04-LTS",
    version: "latest");
```

## <a name="connect-to-linux-nodes-using-ssh"></a>Se connecter à des nœuds Linux via SSH

Pendant le développement ou lors de la résolution des problèmes, il peut s’avérer nécessaire de se connecter aux nœuds de votre pool. Contrairement aux nœuds de calcul Windows, vous ne pouvez pas utiliser le protocole RDP (Remote Desktop Protocol) pour se connecter à des nœuds Linux. Au lieu de cela, le service Batch autorise l’accès SSH sur chaque nœud de connexion à distance.

L’extrait de code Python suivant crée un utilisateur sur chaque nœud d’un pool, nécessaire à la connexion à distance. Il imprime ensuite les informations de connexion SSH pour chaque nœud.

```python
import datetime
import getpass
import azure.batch.batch_service_client as batch
import azure.batch.batch_auth as batchauth
import azure.batch.models as batchmodels

# Specify your own account credentials
batch_account_name = ''
batch_account_key = ''
batch_account_url = ''

# Specify the ID of an existing pool containing Linux nodes
# currently in the 'idle' state
pool_id = ''

# Specify the username and prompt for a password
username = 'linuxuser'
password = getpass.getpass()

# Create a BatchClient
credentials = batchauth.SharedKeyCredentials(
    batch_account_name,
    batch_account_key
)
batch_client = batch.BatchServiceClient(
    credentials,
    base_url=batch_account_url
)

# Create the user that will be added to each node in the pool
user = batchmodels.ComputeNodeUser(username)
user.password = password
user.is_admin = True
user.expiry_time = \
    (datetime.datetime.today() + datetime.timedelta(days=30)).isoformat()

# Get the list of nodes in the pool
nodes = batch_client.compute_node.list(pool_id)

# Add the user to each node in the pool and print
# the connection information for the node
for node in nodes:
    # Add the user to the node
    batch_client.compute_node.add_user(pool_id, node.id, user)

    # Obtain SSH login information for the node
    login = batch_client.compute_node.get_remote_login_settings(pool_id,
                                                                node.id)

    # Print the connection info for the node
    print("{0} | {1} | {2} | {3}".format(node.id,
                                         node.state,
                                         login.remote_login_ip_address,
                                         login.remote_login_port))
```

Le résultat pour ce code ressemble à l’exemple suivant. Dans ce cas, le pool contient quatre nœuds Linux.

```
Password:
tvm-1219235766_1-20160414t192511z | ComputeNodeState.idle | 13.91.7.57 | 50000
tvm-1219235766_2-20160414t192511z | ComputeNodeState.idle | 13.91.7.57 | 50003
tvm-1219235766_3-20160414t192511z | ComputeNodeState.idle | 13.91.7.57 | 50002
tvm-1219235766_4-20160414t192511z | ComputeNodeState.idle | 13.91.7.57 | 50001
```

Au lieu d’un mot de passe, vous pouvez spécifier une clé publique SSH lorsque vous créez un utilisateur sur un nœud. Dans le SDK Python, utilisez le paramètre **ssh_public_key** sur [ComputeNodeUser](/python/api/azure-batch/azure.batch.models.computenodeuser). Dans .NET, utilisez la propriété [ComputeNodeUser.SshPublicKey](/dotnet/api/microsoft.azure.batch.computenodeuser.sshpublickey#Microsoft_Azure_Batch_ComputeNodeUser_SshPublicKey).

## <a name="pricing"></a>Tarifs

Azure Batch est basé sur la technologie d’Azure Cloud Services et des machines virtuelles Azure. Le service Batch lui-même est proposé gratuitement, ce qui signifie que vous payez uniquement les ressources de calcul (et les coût associés qu’elles impliquent) que vos solutions Batch utilisent. Si vous sélectionnez la **Configuration de la machine virtuelle**, vous êtes facturé en fonction de la structure de [tarification des machines virtuelles](https://azure.microsoft.com/pricing/details/virtual-machines/).

Si vous déployez des applications sur vos nœuds Batch à l’aide de [packages d’application](batch-application-packages.md), vous êtes également facturé pour les ressources Stockage Azure que vos packages d’application consomment.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez les [exemples de code Python](https://github.com/Azure/azure-batch-samples/tree/master/Python/Batch) dans le [Dépôt GitHub azure-batch-samples](https://github.com/Azure/azure-batch-samples) pour découvrir comment exécuter des opérations Batch courantes, telles que la création de pools, de travaux et de tâches. Le fichier [Lisez-moi](https://github.com/Azure/azure-batch-samples/blob/master/Python/Batch/README.md) qui accompagne les exemples de code Python contient des informations sur l’installation des packages nécessaires.
- En savoir plus sur l’utilisation de [machines virtuelles de faible priorité](batch-low-pri-vms.md) avec Batch.
