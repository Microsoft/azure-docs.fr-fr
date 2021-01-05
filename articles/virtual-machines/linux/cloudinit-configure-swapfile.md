---
title: Utiliser cloud-init pour configurer une partition d'échange sur une machine virtuelle Linux
description: Utilisation de cloud-init pour configurer une partition d'échange sur une machine virtuelle Linux lors de la création à l'aide de l'interface Azure CLI
author: rickstercdn
manager: gwallace
ms.service: virtual-machines-linux
ms.topic: how-to
ms.date: 11/29/2017
ms.author: rclaus
ms.openlocfilehash: 1f8746e67ba712ac4c2c3e832fcb5ffefb170d59
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87371938"
---
# <a name="use-cloud-init-to-configure-a-swap-partition-on-a-linux-vm"></a>Utiliser cloud-init pour configurer une partition d'échange sur une machine virtuelle Linux
Cet article vous explique comment utiliser [cloud-init](https://cloudinit.readthedocs.io) pour configurer la partition d'échange sur différentes distributions Linux. La partition d'échange est généralement configurée par l'Agent Linux (WALA) en fonction des distributions qui en ont besoin.  Ce document décrit le processus de création de la partition d'échange à la demande au moment de l'approvisionnement à l'aide de cloud-init.  Pour plus d’informations sur le fonctionnement de cloud-init en mode natif dans Azure et sur les versions de Linux prises en charge, consultez [Présentation de cloud-init](using-cloud-init.md)

## <a name="create-swap-partition-for-ubuntu-based-images"></a>Créer une partition d'échange pour des images basées sur Ubuntu
Par défaut sur Azure, les images de la galerie d'Ubuntu ne créent pas de partitions d'échange. Pour activer la configuration de la partition d'échange au moment de l'approvisionnement de la machine virtuelle à l'aide de cloud-init, consultez le [document AzureSwapPartitions](https://wiki.ubuntu.com/AzureSwapPartitions) sur le wiki Ubuntu.

## <a name="create-swap-partition-for-red-hat-and-centos-based-images"></a>Créer une partition d'échange pour les images basées sur Red Hat et CentOS

Dans l'interpréteur de commandes actuel, créez un fichier nommé *cloud_init_swappart.txt* et collez la configuration suivante. Pour cet exemple, créez le fichier dans Cloud Shell et non sur votre ordinateur local. Vous pouvez utiliser l’éditeur de votre choix. Entrez `sensible-editor cloud_init_swappart.txt` pour créer le fichier et afficher la liste des éditeurs disponibles. Choisissez le n°1 pour utiliser l’éditeur **nano**. Vérifiez que l’intégralité du fichier cloud-init est copiée, en particulier la première ligne.  

```yaml
#cloud-config
disk_setup:
  ephemeral0:
    table_type: gpt
    layout: [66, [33,82]]
    overwrite: true
fs_setup:
  - device: ephemeral0.1
    filesystem: ext4
  - device: ephemeral0.2
    filesystem: swap
mounts:
  - ["ephemeral0.1", "/mnt"]
  - ["ephemeral0.2", "none", "swap", "sw", "0", "0"]
```

Avant de déployer cette image, vous devez créer un groupe de ressources avec la commande [az group create](/cli/azure/group). Un groupe de ressources Azure est un conteneur logique dans lequel les ressources Azure sont déployées et gérées. L’exemple suivant crée un groupe de ressources nommé *myResourceGroup* à l’emplacement *eastus*.

```azurecli-interactive 
az group create --name myResourceGroup --location eastus
```

Maintenant, créez une machine virtuelle avec [az vm create](/cli/azure/vm) et spécifiez le fichier cloud-init avec `--custom-data cloud_init_swappart.txt` comme suit :

```azurecli-interactive 
az vm create \
  --resource-group myResourceGroup \
  --name centos74 \
  --image OpenLogic:CentOS:7-CI:latest \
  --custom-data cloud_init_swappart.txt \
  --generate-ssh-keys 
```

## <a name="verify-swap-partition-was-created"></a>Vérifier que la partition d'échange a été créée
Établissez une connexion SSH à l’adresse IP publique de votre machine virtuelle indiquée dans la sortie de la commande précédente. Entrez votre propre **publicIpAddress** comme suit :

```bash
ssh <publicIpAddress>
```

Une fois le protocole SSH appliqué à la machine virtuelle, vérifiez que la partition d'échange a été créée.

```bash
swapon -s
```

La sortie de cette commande doit ressembler à ceci :

```text
Filename                Type        Size    Used    Priority
/dev/sdb2  partition   2494440 0   -1
```

> [!NOTE] 
> Si vous disposez d'une image Azure existante sur laquelle une partition d'échange est configurée et que vous souhaitez modifier la configuration de la partition d'échange pour de nouvelles images, vous devez supprimer la partition d'échange existante. Veuillez consulter « Customize Images to provision by cloud-init» (Personnaliser des images à approvisionner par cloud-init) pour plus d’informations.

## <a name="next-steps"></a>Étapes suivantes
Pour obtenir des exemples cloud-init supplémentaires de modifications de configuration, consultez les rubriques suivantes :
 
- [Ajouter un utilisateur Linux supplémentaire à une machine virtuelle](cloudinit-add-user.md)
- [Exécuter un gestionnaire de package pour mettre à jour les packages existants au premier démarrage](cloudinit-update-vm.md)
- [Use cloud-init to set hostname for a Linux VM in Azure](cloudinit-update-vm-hostname.md) (Utiliser cloud-init pour définir un nom d’hôte pour une machine virtuelle Linux dans Azure) 
- [Installer un package d’application, mettre à jour des fichiers de configuration et injecter des clés](tutorial-automate-vm-deployment.md)
