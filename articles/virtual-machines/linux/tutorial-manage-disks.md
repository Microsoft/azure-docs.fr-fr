---
title: Didacticiel - Gestion des disques Azure avec l’interface de ligne de commande Azure
description: Dans ce tutoriel, vous allez apprendre à utiliser Azure CLI afin de créer et gérer des disques Azure pour des machines virtuelles
author: cynthn
ms.service: virtual-machines-linux
ms.topic: tutorial
ms.workload: infrastructure
ms.date: 08/20/2020
ms.author: cynthn
ms.custom: mvc, devx-track-azurecli
ms.subservice: disks
ms.openlocfilehash: 948a4ae8c329d69e404ef8d0f609748b955b0ecc
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89078847"
---
# <a name="tutorial---manage-azure-disks-with-the-azure-cli"></a>Didacticiel - Gestion des disques Azure avec l’interface de ligne de commande Azure

Les machines virtuelles utilisent des disques pour stocker leur système d’exploitation, leurs applications et leurs données. Lorsque vous créez une machine virtuelle, il est important de choisir une taille de disque et une configuration adaptées à la charge de travail prévue. Ce tutoriel vous montre comment déployer et gérer des disques de machine virtuelle. Vous en apprendrez davantage sur les points suivants :

> [!div class="checklist"]
> * Disques de système d’exploitation et disques temporaires
> * Disques de données
> * Disques Standard et Premium
> * Performances des disques
> * Attachement et préparation des disques de données
> * Captures instantanées de disque


## <a name="default-azure-disks"></a>Disques Azure par défaut

Lorsqu’une machine virtuelle Azure est créée, deux disques sont automatiquement attachés à celle-ci.

**Disque de système d’exploitation** : la taille des disques de système d’exploitation peut atteindre 2 To ; ces disques hébergent le système d’exploitation des machines virtuelles. Le disque de système d’exploitation est nommé */dev/sda* par défaut. La configuration de la mise en cache de disque de système d’exploitation est optimisée pour les performances du système d’exploitation. En raison de cette configuration, le disque de système d’exploitation **ne doit pas** être utilisé pour les applications ou les données. Pour héberger ce type de contenu, utilisez plutôt des disques de données, qui sont décrits plus loin dans ce tutoriel.

**Disque temporaire** : les disques temporaires utilisent un disque SSD qui se trouve sur le même hôte Azure que la machine virtuelle. Les disques temporaires sont extrêmement performants et peuvent être utilisés pour des opérations telles que le traitement de données temporaires. Toutefois, si la machine virtuelle est déplacée vers un nouvel hôte, toutes les données stockées sur un disque temporaire sont supprimées. La taille du disque temporaire est déterminée par la taille de la machine virtuelle. Les disques temporaires sont nommés */dev/sdb* et ont un point de montage */mnt*.

## <a name="azure-data-disks"></a>Disques de données Azure

Des disques de données supplémentaires peuvent être ajoutés pour installer des applications et stocker des données. Les disques de données doivent être utilisés dans les cas où un stockage des données durable et réactif est souhaité. La taille de la machine virtuelle détermine le nombre de disques de données pouvant être attachés à cette machine virtuelle.

## <a name="vm-disk-types"></a>Type de disque de machine virtuelle

Azure propose deux types de disque.

**Disques Standard** : ils s’appuient sur des disques durs et offrent un stockage économique qui n’en est pas moins performant. Les disques Standard constituent la solution idéale pour une charge de travail de développement et de test économique.

**Disques Premium** : ils reposent sur un disque SSD à faible latence et hautes performances. Ils conviennent parfaitement aux machines virtuelles exécutant une charge de travail en production. Les machines virtuelles dont le [nom de la taille](../vm-naming-conventions.md) contient un **S** prennent généralement en charge le stockage Premium. Par exemple, les machines virtuelles Azure des séries DS, DSv2, GS et FS prennent en charge le stockage Premium. Lorsque vous sélectionnez une taille de disque, la valeur est arrondie au type suivant. Par exemple, si la taille du disque est supérieure à 64 Go, mais inférieure à 128 Go, le type de disque est P10. 

<br>


[!INCLUDE [disk-storage-premium-ssd-sizes](../../../includes/disk-storage-premium-ssd-sizes.md)]

Lorsque vous configurez un disque de stockage Premium, contrairement au stockage standard, la capacité, les E/S par seconde et le débit de ce disque sont assurés. Par exemple, si vous créez un disque P50, Azure configure une capacité de stockage de 4 095 Go, 7 500 E/S par seconde et un débit de 250 Mo/s pour ce disque. Votre application peut utiliser tout ou partie de la capacité et des performances. Les disques SSD Premium sont conçus pour fournir des latences faibles de quelques millisecondes ainsi que l’IOPS et le débit cibles décrits dans le précédent tableau 99,9 % du temps.

Bien que le tableau ci-dessus identifie le nombre max. d’E/S par seconde par disque, un niveau de performances plus élevé est possible en entrelaçant plusieurs disques de données. Par exemple, 64 disques de données peuvent être attachés à la machine virtuelle Standard_GS5. Si chacun de ces disques est de type P30, vous pouvez atteindre un nombre maximum d’E/S par seconde de 80 000. Pour plus d’informations sur le nombre maximal d’E/S par seconde par machine virtuelle, consultez [Types et tailles des machines virtuelles](../sizes.md).

## <a name="launch-azure-cloud-shell"></a>Lancement d’Azure Cloud Shell

Azure Cloud Shell est un interpréteur de commandes interactif et gratuit que vous pouvez utiliser pour exécuter les étapes de cet article. Il contient des outils Azure courants préinstallés et configurés pour être utilisés avec votre compte.

Pour ouvrir Cloud Shell, sélectionnez **Essayer** en haut à droite d’un bloc de code. Vous pouvez également lancer Cloud Shell dans un onglet distinct du navigateur en accédant à [https://shell.azure.com/powershell](https://shell.azure.com/bash). Sélectionnez **Copier** pour copier les blocs de code, collez-les dans Cloud Shell, puis appuyez sur Entrée pour les exécuter.

## <a name="create-and-attach-disks"></a>Créer et attacher des disques

Des disques de données peuvent être créés et attachés lors de la création d’une machine virtuelle, mais ils peuvent également être créés et attachés à une machine virtuelle existante.

### <a name="attach-disk-at-vm-creation"></a>Attacher le disque lors de la création d’une machine virtuelle

Créez un groupe de ressources avec la commande [az group create](/cli/azure/group#az-group-create).

```azurecli-interactive
az group create --name myResourceGroupDisk --location eastus
```

Créez une machine virtuelle avec la commande [az vm create](/cli/azure/vm#az-vm-create). L’exemple suivant crée une machine virtuelle nommée *myVM*, ajoute un compte d’utilisateur appelé *azureuser* et génère des clés SSH si elles n’existent pas. L’argument `--datadisk-sizes-gb` est utilisé pour spécifier qu’un disque supplémentaire doit être créé et attaché à la machine virtuelle. Pour créer et attacher plusieurs disques, utilisez une liste séparée par des espaces des valeurs de taille de disque. Dans l’exemple suivant, une machine virtuelle est créée avec deux disques de données, tous deux de 128 Go. La taille des disques étant de 128 Go, ces disques sont configurés en tant que disques P10, qui fournissent 500 E/S par seconde maximum par disque.

```azurecli-interactive
az vm create \
  --resource-group myResourceGroupDisk \
  --name myVM \
  --image UbuntuLTS \
  --size Standard_DS2_v2 \
  --generate-ssh-keys \
  --data-disk-sizes-gb 128 128
```

### <a name="attach-disk-to-existing-vm"></a>Attacher un disque à une machine virtuelle existante

Pour créer et attacher un nouveau disque à une machine virtuelle existante, utilisez la commande [az vm disk attach](/cli/azure/vm/disk#az-vm-disk-attach). L’exemple suivant crée un disque Premium de 128 Go et l’attache à la machine virtuelle créée à l’étape précédente.

```azurecli-interactive
az vm disk attach \
    --resource-group myResourceGroupDisk \
    --vm-name myVM \
    --name myDataDisk \
    --size-gb 128 \
    --sku Premium_LRS \
    --new
```

## <a name="prepare-data-disks"></a>Préparer les disques de données

Une fois qu’un disque a été attaché à la machine virtuelle, le système d’exploitation doit être configuré pour utiliser le disque. L’exemple suivant montre comment configurer manuellement un disque. Ce processus peut également être automatisé à l’aide de cloud-init, qui est présenté dans un [prochain didacticiel](./tutorial-automate-vm-deployment.md).


Créez une connexion SSH avec la machine virtuelle. Remplacez l’exemple d’adresse IP par l’adresse IP publique de la machine virtuelle.

```console
ssh 10.101.10.10
```

Partitionnez le disque avec `parted`.

```bash
sudo parted /dev/sdc --script mklabel gpt mkpart xfspart xfs 0% 100%
```

Écrivez un système de fichiers dans la partition avec la commande `mkfs`. Utilisez `partprobe` pour que le système d’exploitation prenne en compte la modification.

```bash
sudo mkfs.xfs /dev/sdc1
sudo partprobe /dev/sdc1
```

Montez le nouveau disque afin qu’il soit accessible dans le système d’exploitation.

```bash
sudo mkdir /datadrive && sudo mount /dev/sdc1 /datadrive
```

Le disque est désormais accessible via le point de montage `/datadrive`, qui peut être vérifié en exécutant la commande `df -h`.

```bash
df -h | grep -i "sd"
```

La sortie indique le nouveau lecteur monté sur `/datadrive`.

```bash
Filesystem      Size  Used Avail Use% Mounted on
/dev/sda1        29G  2.0G   27G   7% /
/dev/sda15      105M  3.6M  101M   4% /boot/efi
/dev/sdb1        14G   41M   13G   1% /mnt
/dev/sdc1        50G   52M   47G   1% /datadrive
```

Pour vous assurer que le lecteur est remonté automatiquement après un redémarrage, vous devez l’ajouter au fichier */etc/fstab*. Pour ce faire, obtenez l’UUID du disque avec l’utilitaire `blkid`.

```bash
sudo -i blkid
```

La sortie affiche l’UUID du lecteur, `/dev/sdc1` dans cet exemple.

```bash
/dev/sdc1: UUID="33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e" TYPE="xfs"
```

> [!NOTE]
> si vous ne modifiez pas correctement le fichier **/etc/fstab** , il se peut que le système ne puisse plus démarrer. En cas de doute, reportez-vous à la documentation de la distribution pour obtenir des informations sur la modification adéquate de ce fichier. Il est par ailleurs vivement recommandé de créer une sauvegarde du fichier /etc/fstab avant de le modifier.

Ouvrez le fichier `/etc/fstab` dans un éditeur de texte comme suit :

```bash
sudo nano /etc/fstab
```

Ajoutez une ligne semblable à la suivante au fichier */etc/fstab* en remplaçant la valeur UUID par la vôtre.

```bash
UUID=33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e   /datadrive  xfs    defaults,nofail   1  2
```

Une fois que vous avez fini de modifier le fichier, utilisez `Ctrl+O` pour écrire le fichier et `Ctrl+X` pour quitter l’éditeur.

Maintenant que le disque a été configuré, fermez la session SSH.

```bash
exit
```

## <a name="take-a-disk-snapshot"></a>Prendre un instantané d’un disque

Lors d’une capture instantanée du disque, Azure crée une copie en lecture seule à un moment donné du disque. Les captures instantanées de machine virtuelle Azure sont utiles pour enregistrer rapidement l’état d’une machine virtuelle avant d’apporter des modifications à la configuration. En cas de problème ou d’erreur, la machine virtuelle peut être restaurée à l’aide d’un instantané. Lorsqu’une machine virtuelle a plusieurs disques, une capture instantanée de chaque disque est prise. Pour les sauvegardes cohérentes des applications, pensez à arrêter la machine virtuelle avant de prendre des captures instantanées du disque. Vous pouvez également utiliser le [service Sauvegarde Azure](../../backup/index.yml), qui vous permet d’effectuer des sauvegardes automatisées alors que la machine virtuelle est en cours d’exécution.

### <a name="create-snapshot"></a>Créer une capture instantanée

Avant de créer une capture instantanée, vous avez besoin de l’ID ou du nom du disque. Utilisez [az vm show](/cli/azure/vm#az-vm-show) pour afficher l’ID du disque. Dans cet exemple, l’ID du disque est stocké dans une variable pour qu’il puisse être utilisé dans une étape ultérieure.

```azurecli-interactive
osdiskid=$(az vm show \
   -g myResourceGroupDisk \
   -n myVM \
   --query "storageProfile.osDisk.managedDisk.id" \
   -o tsv)
```

Maintenant que vous avez l’ID, utilisez [az snapshot create](/cli/azure/snapshot#az-snapshot-create) pour créer une capture instantanée du disque.

```azurecli-interactive
az snapshot create \
    --resource-group myResourceGroupDisk \
    --source "$osdiskid" \
    --name osDisk-backup
```

### <a name="create-disk-from-snapshot"></a>Création d’un disque à partir d’une capture instantanée

Cette capture instantanée peut alors être convertie en disque à l’aide de [az disk create](/cli/azure/disk#az-disk-create), qui peut être utilisé pour recréer la machine virtuelle.

```azurecli-interactive
az disk create \
   --resource-group myResourceGroupDisk \
   --name mySnapshotDisk \
   --source osDisk-backup
```

### <a name="restore-virtual-machine-from-snapshot"></a>Restauration de la machine virtuelle à partir de l’instantané

Pour illustrer la récupération de machine virtuelle, supprimez la machine virtuelle existante à l’aide de [az vm delete](/cli/azure/vm#az-vm-delete).

```azurecli-interactive
az vm delete \
--resource-group myResourceGroupDisk \
--name myVM
```

Créez une nouvelle machine virtuelle à l’aide du disque d’instantané.

```azurecli-interactive
az vm create \
    --resource-group myResourceGroupDisk \
    --name myVM \
    --attach-os-disk mySnapshotDisk \
    --os-type linux
```

### <a name="reattach-data-disk"></a>Rattacher un disque de données

Tous les disques de données doivent être rattachés à la machine virtuelle.

Trouvez le nom du disque de données à l’aide de la commande [az disk list](/cli/azure/disk#az-disk-list). Cet exemple place le nom du disque dans une variable nommée `datadisk`, qui est utilisée dans l’étape suivante.

```azurecli-interactive
datadisk=$(az disk list \
   -g myResourceGroupDisk \
   --query "[?contains(name,'myVM')].[id]" \
   -o tsv)
```

Utilisez la commande [az vm disk attach](/cli/azure/vm/disk#az-vm-disk-attach) pour attacher le disque.

```azurecli-interactive
az vm disk attach \
   –g myResourceGroupDisk \
   --vm-name myVM \
   --name $datadisk
```

## <a name="next-steps"></a>Étapes suivantes

Ce didacticiel vous a apporté des connaissances concernant les disques de machine virtuelle, notamment concernant les points suivants :

> [!div class="checklist"]
> * Disques de système d’exploitation et disques temporaires
> * Disques de données
> * Disques Standard et Premium
> * Performances des disques
> * Attachement et préparation des disques de données
> * Captures instantanées de disque

Passez au didacticiel suivant pour en apprendre davantage sur l’automatisation de la configuration de machine virtuelle.

> [!div class="nextstepaction"]
> [Automatiser la configuration de machine virtuelle](./tutorial-automate-vm-deployment.md)
