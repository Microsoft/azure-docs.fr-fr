---
title: Créer des images de machines virtuelles Azure Linux avec Packer
description: Découvrez comment utiliser Packer pour créer des images de machines virtuelles Linux dans Azure
author: cynthn
ms.service: virtual-machines-linux
ms.subservice: imaging
ms.topic: how-to
ms.workload: infrastructure
ms.date: 05/07/2019
ms.author: cynthn
ms.openlocfilehash: 29a0c47bf24ecd916fb9402ffcb2a3ff13a36a84
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96016418"
---
# <a name="how-to-use-packer-to-create-linux-virtual-machine-images-in-azure"></a>Comment utiliser Packer pour créer des images de machines virtuelles Linux dans Azure
Chaque machine virtuelle dans Azure est créée à partir d’une image qui définit la distribution Linux et la version du système d’exploitation. Les images peuvent inclure des configurations et des applications pré-installées. La Place de marché Microsoft Azure fournit de nombreuses images internes et de tiers pour les distributions et environnements d’application les plus courants. Vous pouvez également créer vos propres images personnalisées selon vos besoins. Cet article explique comment utiliser l’outil open source [Packer](https://www.packer.io/) pour définir et générer des images personnalisées dans Azure.

> [!NOTE]
> Azure propose désormais un service, le générateur d’images Azure (préversion), pour définir et créer vos propres images personnalisées. Le générateur d’images Azure repose sur Packer. Vous pouvez donc même utiliser vos scripts d’approvisionnement de shell Packer existants. Pour vous familiariser avec le générateur d’images Azure, voir [Créer une machine virtuelle Linux avec le générateur d’images Azure](image-builder.md).


## <a name="create-azure-resource-group"></a>Créer un groupe de ressources Azure
Pendant le processus de génération, Packer crée des ressources Azure temporaires lorsqu’il génère la machine virtuelle source. Pour capturer cette machine virtuelle source afin de l’utiliser en tant qu’image, vous devez définir un groupe de ressources. La sortie du processus de génération Packer est stockée dans ce groupe de ressources.

Créez un groupe de ressources avec la commande [az group create](/cli/azure/group). L’exemple suivant crée un groupe de ressources nommé *myResourceGroup* à l’emplacement *eastus* :

```azurecli
az group create -n myResourceGroup -l eastus
```


## <a name="create-azure-credentials"></a>Créer des informations d’identification Azure
Packer s’authentifie auprès d’Azure à l’aide d’un principal de service. Un principal de service Azure est une identité de sécurité que vous pouvez utiliser avec des applications, des services et des outils d’automatisation comme Packer. Vous contrôlez et vous définissez les opérations que le principal du service est autorisé à effectuer dans Azure.

Créez un principal de service avec la commande [az ad sp create-for-rbac](/cli/azure/ad/sp) et affichez les informations d’identification nécessaires à Packer :

```azurecli
az ad sp create-for-rbac --query "{ client_id: appId, client_secret: password, tenant_id: tenant }"
```

Voici un exemple de la sortie des commandes précédentes :

```output
{
    "client_id": "f5b6a5cf-fbdf-4a9f-b3b8-3c2cd00225a4",
    "client_secret": "0e760437-bf34-4aad-9f8d-870be799c55d",
    "tenant_id": "72f988bf-86f1-41af-91ab-2d7cd011db47"
}
```

Pour s’authentifier sur Azure, vous devez également obtenir votre ID d’abonnement Azure avec [az account show](/cli/azure/account) :

```azurecli
az account show --query "{ subscription_id: id }"
```

Vous utilisez la sortie de ces deux commandes à l’étape suivante.


## <a name="define-packer-template"></a>Définition du modèle Packer
Pour générer les images, vous devez créer un modèle en tant que fichier JSON. Dans le modèle, vous définissez des générateurs et des fournisseurs pour mener à bien le processus de génération réel. Packer possède un [fournisseur pour Azure](https://www.packer.io/docs/builders/azure.html) qui vous permet de définir des ressources Azure, telles que les informations d’identification du principal de service créées à l’étape précédente.

Créez un fichier nommé *ubuntu.json* et collez le contenu suivant : Saisissez vos propres valeurs comme suit :

| Paramètre                           | Emplacement |
|-------------------------------------|----------------------------------------------------|
| *client_id*                         | Première ligne de la sortie de la commande create `az ad sp` - *appId* |
| *client_secret*                     | Deuxième ligne de la sortie de la commande create `az ad sp` - *password* |
| *tenant_id*                         | Troisième ligne de la sortie de la commande create `az ad sp` - *tenant* |
| *subscription_id*                   | Sortie de la commande `az account show` |
| *managed_image_resource_group_name* | Nom du groupe de ressources créé lors de la première étape |
| *managed_image_name*                | Nom de l’image de disque géré créée |


```json
{
  "builders": [{
    "type": "azure-arm",

    "client_id": "f5b6a5cf-fbdf-4a9f-b3b8-3c2cd00225a4",
    "client_secret": "0e760437-bf34-4aad-9f8d-870be799c55d",
    "tenant_id": "72f988bf-86f1-41af-91ab-2d7cd011db47",
    "subscription_id": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxx",

    "managed_image_resource_group_name": "myResourceGroup",
    "managed_image_name": "myPackerImage",

    "os_type": "Linux",
    "image_publisher": "Canonical",
    "image_offer": "UbuntuServer",
    "image_sku": "16.04-LTS",

    "azure_tags": {
        "dept": "Engineering",
        "task": "Image deployment"
    },

    "location": "East US",
    "vm_size": "Standard_DS2_v2"
  }],
  "provisioners": [{
    "execute_command": "chmod +x {{ .Path }}; {{ .Vars }} sudo -E sh '{{ .Path }}'",
    "inline": [
      "apt-get update",
      "apt-get upgrade -y",
      "apt-get -y install nginx",

      "/usr/sbin/waagent -force -deprovision+user && export HISTSIZE=0 && sync"
    ],
    "inline_shebang": "/bin/sh -x",
    "type": "shell"
  }]
}
```

Ce modèle génère une image Ubuntu 16.04 LTS, installe NGINX, puis déprovisionne la machine virtuelle.

> [!NOTE]
> Si vous développez ce modèle pour approvisionner les informations d’identification utilisateur, définissez la commande du fournisseur qui déprovisionne l’agent Azure de sorte qu’elle indique `-deprovision` et non `deprovision+user`.
> L’indicateur `+user` supprime tous les comptes d’utilisateur de la machine virtuelle source.


## <a name="build-packer-image"></a>Génération de l’image Packer
Si Packer n’est pas encore installé sur votre ordinateur local, [suivez les instructions d’installation de Packer](https://www.packer.io/docs/install).

Générez l’image en spécifiant votre fichier de modèle Packer comme suit :

```bash
./packer build ubuntu.json
```

Voici un exemple de la sortie des commandes précédentes :

```output
azure-arm output will be in this color.

==> azure-arm: Running builder ...
    azure-arm: Creating Azure Resource Manager (ARM) client ...
==> azure-arm: Creating resource group ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-swtxmqm7ly’
==> azure-arm:  -> Location          : ‘East US’
==> azure-arm:  -> Tags              :
==> azure-arm:  ->> dept : Engineering
==> azure-arm:  ->> task : Image deployment
==> azure-arm: Validating deployment template ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-swtxmqm7ly’
==> azure-arm:  -> DeploymentName    : ‘pkrdpswtxmqm7ly’
==> azure-arm: Deploying deployment template ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-swtxmqm7ly’
==> azure-arm:  -> DeploymentName    : ‘pkrdpswtxmqm7ly’
==> azure-arm: Getting the VM’s IP address ...
==> azure-arm:  -> ResourceGroupName   : ‘packer-Resource-Group-swtxmqm7ly’
==> azure-arm:  -> PublicIPAddressName : ‘packerPublicIP’
==> azure-arm:  -> NicName             : ‘packerNic’
==> azure-arm:  -> Network Connection  : ‘PublicEndpoint’
==> azure-arm:  -> IP Address          : ‘40.76.218.147’
==> azure-arm: Waiting for SSH to become available...
==> azure-arm: Connected to SSH!
==> azure-arm: Provisioning with shell script: /var/folders/h1/ymh5bdx15wgdn5hvgj1wc0zh0000gn/T/packer-shell868574263
    azure-arm: WARNING! The waagent service will be stopped.
    azure-arm: WARNING! Cached DHCP leases will be deleted.
    azure-arm: WARNING! root password will be disabled. You will not be able to login as root.
    azure-arm: WARNING! /etc/resolvconf/resolv.conf.d/tail and /etc/resolvconf/resolv.conf.d/original will be deleted.
    azure-arm: WARNING! packer account and entire home directory will be deleted.
==> azure-arm: Querying the machine’s properties ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-swtxmqm7ly’
==> azure-arm:  -> ComputeName       : ‘pkrvmswtxmqm7ly’
==> azure-arm:  -> Managed OS Disk   : ‘/subscriptions/guid/resourceGroups/packer-Resource-Group-swtxmqm7ly/providers/Microsoft.Compute/disks/osdisk’
==> azure-arm: Powering off machine ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-swtxmqm7ly’
==> azure-arm:  -> ComputeName       : ‘pkrvmswtxmqm7ly’
==> azure-arm: Capturing image ...
==> azure-arm:  -> Compute ResourceGroupName : ‘packer-Resource-Group-swtxmqm7ly’
==> azure-arm:  -> Compute Name              : ‘pkrvmswtxmqm7ly’
==> azure-arm:  -> Compute Location          : ‘East US’
==> azure-arm:  -> Image ResourceGroupName   : ‘myResourceGroup’
==> azure-arm:  -> Image Name                : ‘myPackerImage’
==> azure-arm:  -> Image Location            : ‘eastus’
==> azure-arm: Deleting resource group ...
==> azure-arm:  -> ResourceGroupName : ‘packer-Resource-Group-swtxmqm7ly’
==> azure-arm: Deleting the temporary OS disk ...
==> azure-arm:  -> OS Disk : skipping, managed disk was used...
Build ‘azure-arm’ finished.

==> Builds finished. The artifacts of successful builds are:
--> azure-arm: Azure.ResourceManagement.VMImage:

ManagedImageResourceGroupName: myResourceGroup
ManagedImageName: myPackerImage
ManagedImageLocation: eastus
```

La génération de la machine virtuelle, l’exécution des fournisseurs et le nettoyage du déploiement par Packer ne prennent que quelques minutes.


## <a name="create-vm-from-azure-image"></a>Création d’une machine virtuelle à partir d’une image Azure
Vous pouvez à présent créer une machine virtuelle à partir de votre image à l’aide de la commande [az vm create](/cli/azure/vm). Spécifiez l’image que vous avez créée avec le paramètre `--image`. L’exemple suivant crée une machine virtuelle nommée *myVM* à partir de *myPackerImage* et génère des clés SSH si elles n’existent pas déjà :

```azurecli
az vm create \
    --resource-group myResourceGroup \
    --name myVM \
    --image myPackerImage \
    --admin-username azureuser \
    --generate-ssh-keys
```

Si vous souhaitez créer des machines virtuelles dans un autre groupe de ressources ou dans une autre région que votre image Packer, spécifiez l’ID de l’image plutôt que son nom. Pour obtenir l’ID d’image, exécutez la commande [az image show](/cli/azure/image#az-image-show).

La création de la machine virtuelle ne nécessite que quelques minutes. Une fois la machine virtuelle créée, notez la valeur de `publicIpAddress` qui s’affiche dans l’interface Azure CLI. Cette adresse est utilisée pour accéder au site NGINX à l’aide d’un navigateur web.

Pour autoriser le trafic web à accéder à votre machine virtuelle, ouvrez le port 80 à partir d’Internet à l’aide de la commande [az vm open-port](/cli/azure/vm) :

```azurecli
az vm open-port \
    --resource-group myResourceGroup \
    --name myVM \
    --port 80
```

## <a name="test-vm-and-nginx"></a>Test de la machine virtuelle et de NGINX
Vous pouvez maintenant ouvrir un navigateur web et entrer `http://publicIpAddress` dans la barre d’adresse. Indiquez votre propre adresse IP publique à partir du processus de création de la machine virtuelle. La page NGINX par défaut s’affiche comme dans l’exemple suivant :

![Site par défaut NGINX](./media/build-image-with-packer/nginx.png) 


## <a name="next-steps"></a>Étapes suivantes
Vous pouvez également utiliser les scripts d’approvisionnement Packer existants avec le [générateur d’images Azure](image-builder.md).
