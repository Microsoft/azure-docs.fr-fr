---
title: Charger ou copier une machine virtuelle Linux personnalisée avec Azure CLI
description: Charger ou copier une machine virtuelle personnalisée en utilisant le modèle de déploiement Resource Manager et Azure CLI
services: virtual-machines-linux
documentationcenter: ''
author: cynthn
manager: gwallace
tags: azure-resource-manager
ms.assetid: a8c7818f-eb65-409e-aa91-ce5ae975c564
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: azurecli
ms.topic: how-to
ms.date: 10/10/2019
ms.author: cynthn
ms.openlocfilehash: df2b58e0067932edd9dfa21ee1a6fbb2a5c1fdf7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87289763"
---
# <a name="create-a-linux-vm-from-a-custom-disk-with-the-azure-cli"></a>Créer une machine virtuelle Linux à partir d’un disque personnalisé avec Azure CLI

<!-- rename to create-vm-specialized -->

Cet article vous explique comment charger un disque dur virtuel (VHD) personnalisé et comment copier un VHD existant dans Azure. Le VHD nouvellement créé permet ensuite de créer d’autres machines virtuelles Linux. Vous pouvez installer et configurer une distribution Linux selon vos besoins, puis utiliser ce VHD pour créer une machine virtuelle Azure.

Pour créer plusieurs machines virtuelles à partir de votre disque personnalisé, commencez par créer une image à partir de votre machine virtuelle ou VHD. Pour plus d’informations, consultez l’article [Créer une image personnalisée d’une machine virtuelle Azure avec Azure CLI](tutorial-custom-images.md).

Pour créer un disque personnalisé, vous disposez de deux méthodes :
* Télécharger un disque dur virtuel
* Copier une machine virtuelle Azure existante


## <a name="requirements"></a>Spécifications
Pour exécuter la procédure ci-après, vous avez besoin des éléments suivants :

- Une machine virtuelle Linux qui a été préparée pour une utilisation dans Azure. La section [Préparation de la machine virtuelle](#prepare-the-vm) de cet article explique où trouver les informations propres à la distribution concernant l’installation de l’agent Linux Azure (waagent), qui vous sont nécessaires pour vous connecter à une machine virtuelle avec SSH.
- Le fichier VHD d’une [distribution Linux approuvée par Azure](endorsed-distros.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) existante (ou consultez [les informations relatives aux distributions non approuvées](create-upload-generic.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)) sur un disque virtuel au format VHD. Plusieurs outils permettent de créer une machine virtuelle et un disque dur virtuel :
  - Installez et configurez [QEMU](https://en.wikibooks.org/wiki/QEMU/Installing_QEMU) ou [KVM](https://www.linux-kvm.org/page/RunningKVM), en veillant à utiliser VHD comme format d’image. En cas de besoin, vous pouvez [convertir une image](https://en.wikibooks.org/wiki/QEMU/Images#Converting_image_formats) avec `qemu-img convert`.
  - Vous pouvez également utiliser Hyper-V [sur Windows 10](/virtualization/hyper-v-on-windows/quick-start/enable-hyper-v) ou [sur Windows Server 2012/2012 R2](/previous-versions/windows/it-pro/windows-server-2012-R2-and-2012/hh846766(v=ws.11)).

> [!NOTE]
> Azure ne prend pas en charge le nouveau format VHDX. Lorsque vous créez une machine virtuelle, spécifiez le format de disque dur virtuel (VHD). Si nécessaire, vous pouvez convertir des disques VHDX en VHD avec la commande [qemu-img convert](https://en.wikibooks.org/wiki/QEMU/Images#Converting_image_formats) ou l’applet de commande PowerShell [Convert-VHD](/powershell/module/hyper-v/convert-vhd?view=win10-ps). Azure ne prend pas en charge le chargement de VHD dynamiques. Vous devez donc convertir ces disques en VHD statiques avant le chargement. Vous pouvez utiliser des outils tels que les [utilitaires de disque dur virtuel Azure pour GO](https://github.com/Microsoft/azure-vhd-utils-for-go) pour convertir les disques dynamiques lors de leur chargement dans Azure.
> 
> 


- Vérifiez que vous avez installé la dernière version [d’Azure CLI](/cli/azure/install-az-cli2) et que vous êtes connecté à un compte Azure avec [az login](/cli/azure/reference-index#az-login).

Dans les exemples suivants, remplacez les noms de paramètres par vos propres valeurs, telles que `myResourceGroup`, `mystorageaccount` et `mydisks`.

<a id="prepimage"> </a>

## <a name="prepare-the-vm"></a>Préparation de la machine virtuelle

Azure prend en charge diverses distributions de Linux (voir [Distributions Linux approuvées](endorsed-distros.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)). Les articles ci-après expliquent comment préparer les diverses distributions Linux prises en charge sur Azure :

* [Distributions CentOS](create-upload-centos.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [Debian Linux](debian-create-upload-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [Oracle Linux](oracle-create-upload-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [Red Hat Enterprise Linux](redhat-create-upload-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [SLES et openSUSE](suse-create-upload-vhd.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [Ubuntu](create-upload-ubuntu.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)
* [Autres : Distributions non approuvées](create-upload-generic.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json)

Consultez également les [notes d’installation Linux](create-upload-generic.md#general-linux-installation-notes) pour obtenir d’autres conseils généraux sur la préparation d’images Linux pour Azure.

> [!NOTE]
> Le [Contrat de niveau de service (SLA) de la plateforme Azure](https://azure.microsoft.com/support/legal/sla/virtual-machines/) ne s’applique aux machines virtuelles exécutant Linux que lorsque l’une des distributions approuvées est utilisée avec les détails de configuration définis sous la section « Distributions et versions prises en charge » de l’article [Linux sur les distributions approuvées par Azure](endorsed-distros.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).
> 
> 

## <a name="option-1-upload-a-vhd"></a>Option 1 : Télécharger un disque dur virtuel

Vous pouvez désormais charger un disque dur virtuel directement dans un disque managé. Pour plus d’instructions, consultez [Charger un disque dur virtuel dans Azure à l'aide d'Azure CLI](disks-upload-vhd-to-managed-disk-cli.md).

## <a name="option-2-copy-an-existing-vm"></a>Option n°2 : Copier une machine virtuelle existante

Vous pouvez également créer une machine virtuelle personnalisée dans Azure, puis copier le disque du système d’exploitation et l’attacher à une nouvelle machine virtuelle pour créer une autre copie. Cette approche est bien adaptée à un test, mais si vous souhaitez utiliser une machine virtuelle Azure existante comme modèle pour plusieurs nouvelles machines virtuelles, créez plutôt une *image*. Pour plus d’informations sur la création d’une image à partir d’une machine virtuelle Azure existante, consultez l’article [Créer une image personnalisée d’une machine virtuelle Azure avec Azure CLI](tutorial-custom-images.md).

Si vous souhaitez copier une machine virtuelle existante dans une autre région, vous souhaiterez peut-être utiliser azcopy pour [créer une copie d’un disque dans une autre région](disks-upload-vhd-to-managed-disk-cli.md#copy-a-managed-disk). 

Dans le cas contraire, vous devez prendre un instantané de la machine virtuelle, puis créer un nouveau disque dur virtuel de système d’exploitation à partir de l’instantané.

### <a name="create-a-snapshot"></a>Créer un instantané

Cet exemple crée l’instantané d’une machine virtuelle nommée *myVM* dans le groupe de ressources *myResourceGroup* et crée un instantané nommé *osDiskSnapshot*.

```azurecli
osDiskId=$(az vm show -g myResourceGroup -n myVM --query "storageProfile.osDisk.managedDisk.id" -o tsv)
az snapshot create \
    -g myResourceGroup \
    --source "$osDiskId" \
    --name osDiskSnapshot
```
###  <a name="create-the-managed-disk"></a>Créer le disque géré

Créez un disque géré à partir de l’instantané.

Obtenez l’ID de l’instantané. Dans cet exemple, l’instantané est nommé *osDiskSnapshot* et se trouve dans le groupe de ressources *myResourceGroup*.

```azurecli
snapshotId=$(az snapshot show --name osDiskSnapshot --resource-group myResourceGroup --query [id] -o tsv)
```

Créez le disque géré. Dans cet exemple, nous allons créer à partir de notre instantané un disque managé nommé *myManagedDisk*, qui se trouve dans le stockage standard et présente une taille de 128 Go.

```azurecli
az disk create \
    --resource-group myResourceGroup \
    --name myManagedDisk \
    --sku Standard_LRS \
    --size-gb 128 \
    --source $snapshotId
```

## <a name="create-the-vm"></a>Création de la machine virtuelle

Créez votre machine virtuelle avec la commande [az vm create](/cli/azure/vm#az-vm-create) et attachez (--attach-os-disk) le disque managé en tant que disque du système d’exploitation. L’exemple ci-après crée une machine virtuelle nommée *myNewVM* à l’aide du disque managé que vous avez créé à partir du VHD chargé :

```azurecli
az vm create \
    --resource-group myResourceGroup \
    --location eastus \
    --name myNewVM \
    --os-type linux \
    --attach-os-disk myManagedDisk
```

Vous devriez pouvoir utiliser SSH pour vous connecter à la machine virtuelle avec les informations d’identification de la machine virtuelle source. 

## <a name="next-steps"></a>Étapes suivantes
Après avoir préparé et chargé votre disque virtuel personnalisé, vous pouvez découvrir plus d’informations sur [l’utilisation de Resource Manager et des modèles](../../azure-resource-manager/management/overview.md). Vous pouvez également [ajouter un disque de données](add-disk.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json) à vos nouvelles machines virtuelles. Si vous avez besoin d’accéder à des applications qui s’exécutent sur vos machines virtuelles, veillez à [ouvrir les ports et points de terminaison](nsg-quickstart.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).
