---
title: Tutoriel - Créer et utiliser des disques pour les groupes identiques avec Azure CLI
description: Découvrez comment utiliser Azure CLI pour créer et utiliser la fonctionnalité Disques managés avec un groupe de machines virtuelles identiques, notamment comment ajouter, préparer, répertorier et détacher les disques.
author: ju-shim
ms.author: jushiman
ms.topic: tutorial
ms.service: virtual-machine-scale-sets
ms.subservice: disks
ms.date: 03/27/2018
ms.reviewer: mimckitt
ms.custom: mimckitt, devx-track-azurecli
ms.openlocfilehash: adaa7d1c2cf4a78a680ef4fbbec06975ceda812b
ms.sourcegitcommit: 5e5a0abe60803704cf8afd407784a1c9469e545f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96433494"
---
# <a name="tutorial-create-and-use-disks-with-virtual-machine-scale-set-with-the-azure-cli"></a>Tutoriel : Créer et utiliser des disques avec un groupe de machines virtuelles identiques avec Azure CLI
Les groupes de machines virtuelles identiques utilisent des disques pour stocker le système d’exploitation, les applications et les données de l’instance de machine virtuelle. Lorsque vous créez et gérez un groupe identique, il est important de choisir une taille de disque et une configuration appropriées à la charge de travail prévue. Ce didacticiel explique comment créer et gérer des disques de machine virtuelle. Ce didacticiel vous montre comment effectuer les opérations suivantes :

> [!div class="checklist"]
> * Disques de système d’exploitation et disques temporaires
> * Disques de données
> * Disques Standard et Premium
> * Performances des disques
> * Attacher et préparer des disques de données

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Cet article nécessite la version 2.0.29 ou ultérieure d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="default-azure-disks"></a>Disques Azure par défaut
Lorsqu’un groupe identique est créé ou mis à l’échelle, deux disques sont automatiquement attachés à chaque instance de machine virtuelle.

**Disque de système d’exploitation** : la taille des disques de système d’exploitation peut atteindre 2 To. Ces disques hébergent le système d’exploitation de l’instance de machine virtuelle. Le disque de système d’exploitation est nommé */dev/sda* par défaut. La configuration de la mise en cache de disque de système d’exploitation est optimisée pour les performances du système d’exploitation. En raison de cette configuration, le disque de système d’exploitation **ne doit pas** héberger d’applications ou de données. Pour héberger ce type de contenu, utilisez plutôt des disques de données, qui sont décrits plus loin dans cet article.

**Disque temporaire** : les disques temporaires utilisent un disque SSD qui se trouve sur le même hôte Azure que l’instance de machine virtuelle. Ce sont des disques extrêmement performants qui peuvent être utilisés pour diverses opérations telles que le traitement de données temporaires. Toutefois, si l’instance de machine virtuelle est déplacée vers un nouvel hôte, toutes les données stockées sur le disque temporaire concerné sont supprimées. La taille du disque temporaire est déterminée par la taille de l’instance de machine virtuelle. Les disques temporaires sont nommés */dev/sdb* et ont un point de montage */mnt*.

### <a name="temporary-disk-sizes"></a>Tailles du disque temporaire
| Type | Tailles courantes | Taille maximale du disque temporaire (Gio) |
|----|----|----|
| [Usage général](../virtual-machines/sizes-general.md) | Séries A, B et D | 1 600 |
| [Optimisé pour le calcul](../virtual-machines/sizes-compute.md) | Série F | 576 |
| [Mémoire optimisée](../virtual-machines/sizes-memory.md) | Séries D, E, G et M | 6144 |
| [Optimisé pour le stockage](../virtual-machines/sizes-storage.md) | Série L | 5630 |
| [GPU](../virtual-machines/sizes-gpu.md) | Série N | 1440 |
| [Hautes performances](../virtual-machines/sizes-hpc.md) | Séries A et H | 2000 |


## <a name="azure-data-disks"></a>Disques de données Azure
Des disques de données supplémentaires peuvent être ajoutés si vous avez besoin d’installer des applications et de stocker des données. Les disques de données doivent être utilisés dans les cas où un stockage des données durable et réactif est souhaité. Chaque disque de données offre une capacité maximale de 4 To. La taille de l’instance de machine virtuelle détermine le nombre de disques de données pouvant être attachés. Pour chaque processeur virtuel de machine virtuelle, vous pouvez attacher deux disques de données à un maximum de 64 disques par machine virtuelle.

## <a name="vm-disk-types"></a>Type de disque de machine virtuelle
Azure propose deux types de disque.

### <a name="standard-disk"></a>Disque Standard
Le stockage Standard s’appuie sur des disques HDD et offre un stockage économique et performant. Les disques Standard constituent la solution idéale pour une charge de travail de développement et de test économique.

### <a name="premium-disk"></a>Disque Premium
Les disques Premium reposent sur un disque SSD à faible latence et hautes performances. Ces disques sont recommandés pour les machines virtuelles qui exécutent des charges de travail de production. Le stockage Premium prend en charge les machines virtuelles des séries DS, DSv2, GS et FS. Lorsque vous sélectionnez une taille de disque, la valeur est arrondie au type suivant. Par exemple, si la taille du disque est inférieure à 128 Go, le type de disque est P10. Si la taille du disque est entre 129 Go et 512 Go, le type de disque est P20. Au-dessus de 512 Go, le type de disque est P30.

### <a name="premium-disk-performance"></a>Performances du disque Premium
|Type de disque de stockage Premium | P4 | P6 | P10 | P20 | P30 | P40 | P50 |
| --- | --- | --- | --- | --- | --- | --- | --- |
| Taille du disque (arrondie) | 32 Go | 64 Go | 128 Go | 512 Go | 1 024 Go (1 To) | 2 048 Go (2 To) | 4 095 Go (4 To) |
| Nb max. d'E/S par seconde par disque | 120 | 240 | 500 | 2 300 | 5 000 | 7 500 | 7 500 |
Débit par disque | 25 Mo/s | 50 Mo/s | 100 Mo/s | 150 Mo/s | 200 Mo/s | 250 Mo/s | 250 Mo/s |

Bien que le tableau ci-dessus identifie le nombre max. d’E/S par seconde par disque, un niveau de performances plus élevé est possible en entrelaçant plusieurs disques de données. Par exemple, une machine virtuelle Standard_GS5 peut atteindre un nombre maximum d’E/S par seconde de 80 000. Pour plus d’informations sur le nombre max. d’E/S par seconde par machine virtuelle, consultez [Tailles des machines virtuelles Linux dans Azure](../virtual-machines/sizes.md).


## <a name="create-and-attach-disks"></a>Créer et attacher des disques
Vous pouvez créer et attacher des disques lorsque vous créez un groupe identique, ou avec un groupe identique existant.

À compter de la version `2019-07-01` de l’API, vous pouvez définir la taille du disque du système d’exploitation dans un groupe de machines virtuelles identiques avec la propriété [storageProfile.osDisk.diskSizeGb](/rest/api/compute/virtualmachinescalesets/createorupdate#virtualmachinescalesetosdisk). Après le provisionnement, il peut être nécessaire d’étendre ou de repartitionner le disque pour utiliser tout l’espace. Découvrez plus en détail [l’extension du disque ici](../virtual-machines/windows/expand-os-disk.md#expand-the-volume-within-the-os).

### <a name="attach-disks-at-scale-set-creation"></a>Attacher des disques lors de la création d’un groupe identique
Tout d’abord, créez un groupe de ressources avec la commande [az group create](/cli/azure/group). Dans cet exemple, un groupe de ressources nommé *myResourceGroupVM* est créé dans la région *eastus*.

```azurecli-interactive
az group create --name myResourceGroup --location eastus
```

Créez un groupe de machines virtuelles identiques avec la commande [az vmss create](/cli/azure/vmss). L’exemple suivant crée un groupe identique nommé *myScaleSet* et génère des clés SSH si elles n’existent pas. Deux disques sont créés avec le paramètre `--data-disk-sizes-gb`. Le premier disque fait *64* Go, et le second disque fait *128* Go :

```azurecli-interactive
az vmss create \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --image UbuntuLTS \
  --upgrade-policy automatic \
  --admin-username azureuser \
  --generate-ssh-keys \
  --data-disk-sizes-gb 64 128
```

La création et la configuration de l’ensemble des ressources et des instances de machine virtuelle du groupe identique prennent quelques minutes.

### <a name="attach-a-disk-to-existing-scale-set"></a>Attacher un disque à un groupe identique existant
Vous pouvez également attacher des disques à un groupe identique existant. Utilisez le groupe identique créé à l’étape précédente pour ajouter un autre disque avec la commande [az vmss disk attach](/cli/azure/vmss/disk). Dans l’exemple suivant, un disque supplémentaire de *128* Go est attaché :

```azurecli-interactive
az vmss disk attach \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --size-gb 128
```


## <a name="prepare-the-data-disks"></a>Préparer les disques de données
Les disques créés et attachés aux instances de machine virtuelle de votre groupe identique sont des disques bruts. Avant que vous ne puissiez les utiliser avec vos données et vos applications, les disques doivent être préparés. Pour préparer les disques, vous devez créer une partition, créer un système de fichiers, et les monter.

Pour automatiser le processus sur plusieurs instances de machine virtuelle dans un groupe identique, vous pouvez utiliser l’extension de script personnalisé Azure. Cette extension peut exécuter des scripts localement sur chaque instance de machine virtuelle, par exemple pour préparer les disques de données attachés. Pour plus d’informations, consultez [Vue d’ensemble de l’extension de script personnalisé](../virtual-machines/extensions/custom-script-linux.md).

L’exemple suivant montre l’exécution d’un script à partir d’un exemple de référentiel GitHub sur chaque instance de machine virtuelle avec la commande [az vmss extension set](/cli/azure/vmss/extension) qui prépare tous les disques de données attachés bruts :

```azurecli-interactive
az vmss extension set \
  --publisher Microsoft.Azure.Extensions \
  --version 2.0 \
  --name CustomScript \
  --resource-group myResourceGroup \
  --vmss-name myScaleSet \
  --settings '{"fileUris":["https://raw.githubusercontent.com/Azure-Samples/compute-automation-configurations/master/prepare_vm_disks.sh"],"commandToExecute":"./prepare_vm_disks.sh"}'
```

Pour confirmer que les disques ont été correctement préparés, utilisez une clé SSH sur l’une des instances de machine virtuelle. Obtenez les informations de connexion pour votre groupe identique avec la commande [az vmss list-instance-connection-info](/cli/azure/vmss) :

```azurecli-interactive
az vmss list-instance-connection-info \
  --resource-group myResourceGroup \
  --name myScaleSet
```

Utilisez vos propres adresse IP publique et numéro de port pour vous connecter à la première instance de machine virtuelle, comme illustré dans l’exemple suivant :

```console
ssh azureuser@52.226.67.166 -p 50001
```

Examinez les partitions sur l’instance de machine virtuelle comme suit :

```bash
sudo fdisk -l
```

L’exemple de sortie suivant montre que trois disques sont attachés à l’instance de machine virtuelle : */dev/sdc*, */dev/sdd* et *sde/dev/* . Chacun de ces disques a une seule partition qui utilise tout l’espace disponible :

```bash
Disk /dev/sdc: 64 GiB, 68719476736 bytes, 134217728 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xa47874cb

Device     Boot Start       End   Sectors Size Id Type
/dev/sdc1        2048 134217727 134215680  64G 83 Linux

Disk /dev/sdd: 128 GiB, 137438953472 bytes, 268435456 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0xd5c95ca0

Device     Boot Start       End   Sectors  Size Id Type
/dev/sdd1        2048 268435455 268433408  128G 83 Linux

Disk /dev/sde: 128 GiB, 137438953472 bytes, 268435456 sectors
Units: sectors of 1 * 512 = 512 bytes
Sector size (logical/physical): 512 bytes / 512 bytes
I/O size (minimum/optimal): 512 bytes / 512 bytes
Disklabel type: dos
Disk identifier: 0x9312fdf5

Device     Boot Start       End   Sectors  Size Id Type
/dev/sde1        2048 268435455 268433408  128G 83 Linux
```

Examinez les systèmes de fichiers et les points de montage sur l’instance de machine virtuelle comme suit :

```bash
sudo df -h
```

L’exemple de sortie suivant montre que les systèmes de fichiers des trois disques sont correctement montés sous */datadisks* :

```output
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        30G  1.3G   28G   5% /
/dev/sdb1        50G   52M   47G   1% /mnt
/dev/sdc1        63G   52M   60G   1% /datadisks/disk1
/dev/sdd1       126G   60M  120G   1% /datadisks/disk2
/dev/sde1       126G   60M  120G   1% /datadisks/disk3
```

Les disques sur chaque instance de machine virtuelle de votre groupe identique sont préparés automatiquement de la même façon. Lorsque votre groupe identique monte en puissance, les disques de données requis sont attachés aux nouvelles instances de machine virtuelle. L’extension de script personnalisé s’exécute aussi pour préparer automatiquement les disques.

Fermez la connexion SSH vers votre instance de machine virtuelle :

```bash
exit
```


## <a name="list-attached-disks"></a>Afficher les disques attachés
Pour afficher plus d’informations sur les disques attachés à un groupe identique, utilisez la commande [az vmss show](/cli/azure/vmss) et exécutez une requête sur *virtualMachineProfile.storageProfile.dataDisks* :

```azurecli-interactive
az vmss show \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --query virtualMachineProfile.storageProfile.dataDisks
```

Les informations sur la taille du disque, le niveau de stockage et le numéro d’unité logique s’affichent. L’exemple de sortie suivant détaille les trois disques de données attachés au groupe identique :

```json
[
  {
    "additionalProperties": {},
    "caching": "None",
    "createOption": "Empty",
    "diskSizeGb": 64,
    "lun": 0,
    "managedDisk": {
      "additionalProperties": {},
      "storageAccountType": "Standard_LRS"
    },
    "name": null
  },
  {
    "additionalProperties": {},
    "caching": "None",
    "createOption": "Empty",
    "diskSizeGb": 128,
    "lun": 1,
    "managedDisk": {
      "additionalProperties": {},
      "storageAccountType": "Standard_LRS"
    },
    "name": null
  },
  {
    "additionalProperties": {},
    "caching": "None",
    "createOption": "Empty",
    "diskSizeGb": 128,
    "lun": 2,
    "managedDisk": {
      "additionalProperties": {},
      "storageAccountType": "Standard_LRS"
    },
    "name": null
  }
]
```


## <a name="detach-a-disk"></a>Détacher un disque
Lorsque vous n’avez plus besoin d’un disque donné, vous pouvez le détacher du groupe identique. Le disque est supprimé de toutes les instances de machine virtuelle dans le groupe identique. Pour détacher un disque à partir d’un groupe identique, utilisez la commande [az vmss disk detach](/cli/azure/vmss/disk) et spécifiez le numéro d’unité logique du disque. Les numéros d’unité logique sont affichés en sortie à partir de la commande [az vmss show](/cli/azure/vmss) indiquée dans la section précédente. L’exemple ci-après montre comment détacher le numéro d’unité logique *2* du groupe identique :

```azurecli-interactive
az vmss disk detach \
  --resource-group myResourceGroup \
  --name myScaleSet \
  --lun 2
```


## <a name="clean-up-resources"></a>Nettoyer les ressources
Pour supprimer votre groupe identique et vos disques, supprimez le groupe de ressources et toutes ses ressources avec [az group delete](/cli/azure/group). Le paramètre `--no-wait` retourne le contrôle à l’invite de commandes sans attendre que l’opération se termine. Le paramètre `--yes` confirme que vous souhaitez supprimer les ressources sans passer par une invite supplémentaire à cette fin.

```azurecli-interactive
az group delete --name myResourceGroup --no-wait --yes
```


## <a name="next-steps"></a>Étapes suivantes
Dans ce tutoriel, vous avez appris à créer et utiliser des disques avec vos groupes identiques au moyen d’Azure CLI :

> [!div class="checklist"]
> * Disques de système d’exploitation et disques temporaires
> * Disques de données
> * Disques Standard et Premium
> * Performances des disques
> * Attacher et préparer des disques de données

Passez à l’étape suivante du didacticiel afin d’apprendre à utiliser une image personnalisée pour les instances de machine virtuelle de votre groupe identique.

> [!div class="nextstepaction"]
> [Utiliser une image personnalisée pour les instances de machine virtuelle d’un groupe identique](tutorial-use-custom-image-cli.md)