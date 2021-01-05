---
title: Ajouter un disque de données à une machine virtuelle Linux avec Azure CLI
description: Découvrir comment ajouter un disque de données persistant à votre machine virtuelle Linux avec l’interface Azure CLI
author: cynthn
ms.service: virtual-machines-linux
ms.topic: how-to
ms.date: 08/20/2020
ms.author: cynthn
ms.subservice: disks
ms.openlocfilehash: 9520196c8dce9ea511c2f3b799bd12b34c6f988f
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96499745"
---
# <a name="add-a-disk-to-a-linux-vm"></a>Ajouter un disque à une machine virtuelle Linux

Cet article vous explique comment attacher un disque persistant à votre machine virtuelle afin de conserver vos données, et ce, même si votre machine virtuelle est remise en service en raison d’une opération de maintenance ou de redimensionnement.


## <a name="attach-a-new-disk-to-a-vm"></a>Attacher un nouveau disque à une machine virtuelle

Si vous souhaitez ajouter un nouveau disque vide sur votre machine virtuelle, utilisez la commande [az vm disk attach](/cli/azure/vm/disk?view=azure-cli-latest) avec le paramètre `--new`. Si votre machine virtuelle est dans une Zone de disponibilité, le disque est automatiquement créé dans la même zone que la machine virtuelle. Pour plus d'informations, consultez [Vue d’ensemble des zones de disponibilité](../../availability-zones/az-overview.md). L’exemple suivant crée un disque nommé *myDataDisk* avec une taille de 50 Go :

```azurecli
az vm disk attach \
   -g myResourceGroup \
   --vm-name myVM \
   --name myDataDisk \
   --new \
   --size-gb 50
```

## <a name="attach-an-existing-disk"></a>Association d'un disque existant

Pour attacher un disque existant, rechercher l’ID de disque et transmettez-le à la commande [az vm disk attach](/cli/azure/vm/disk?view=azure-cli-latest). L’exemple suivant interroge pour un disque nommé *myDataDisk* dans *myResourceGroup*, puis l’attache à la machine virtuelle nommée *myVM* :

```azurecli
diskId=$(az disk show -g myResourceGroup -n myDataDisk --query 'id' -o tsv)

az vm disk attach -g myResourceGroup --vm-name myVM --name $diskId
```

## <a name="format-and-mount-the-disk"></a>Formater et monter le disque

Vous devez exécuter SSH dans votre machine virtuelle Azure afin de partitionner, de formater et de monter votre nouveau disque pour que votre machine virtuelle Linux puisse l’utiliser. Pour plus d’informations, consultez l’article [Utilisation de SSH avec Linux sur Azure](mac-create-ssh-keys.md). L'exemple suivant permet de se connecter à une machine virtuelle avec l'adresse IP publique *10.123.123.25* et le nom d'utilisateur *azureuser* :

```bash
ssh azureuser@10.123.123.25
```

### <a name="find-the-disk"></a>Recherche du disque

Une fois connecté à votre machine virtuelle, vous devez rechercher le disque. Dans cet exemple, nous utilisons `lsblk` pour répertorier les disques. 

```bash
lsblk -o NAME,HCTL,SIZE,MOUNTPOINT | grep -i "sd"
```

Le résultat ressemble à l’exemple suivant :

```bash
sda     0:0:0:0      30G
├─sda1             29.9G /
├─sda14               4M
└─sda15             106M /boot/efi
sdb     1:0:1:0      14G
└─sdb1               14G /mnt
sdc     3:0:0:0      50G
```

`sdc` correspond au disque que nous recherchons, car il fait 50G. Si vous ne savez pas quel disque est basé uniquement sur la taille, vous pouvez accéder à la page des machines virtuelles du portail, sélectionner **Disques** et vérifier le numéro d'unité logique (LUN) du disque sous **Disques de données**. 


### <a name="format-the-disk"></a>Formater le disque

Pour formater le disque, utilisez `parted`. Si la taille du disque est supérieure ou égale à 2 tébioctets (Tio), vous devez utiliser le partitionnement GPT. Si elle est inférieure à 2 Tio, vous pouvez utiliser le partitionnement MBR ou GPT. 

> [!NOTE]
> Il est recommandé d'utiliser la dernière version `parted` disponible pour votre distribution.
> Si la taille du disque est supérieure ou égale à 2 tébioctets (Tio), vous devez utiliser le partitionnement GPT. Si la taille du disque est inférieure à 2 Tio, vous pouvez utiliser le partitionnement MBR ou GPT.  


L'exemple suivant utilise `parted` sur `/dev/sdc`, qui correspond à l'emplacement du premier disque de données sur la plupart des machines virtuelles. Remplacez `sdc` par l'option appropriée pour votre disque. Nous le formatons également à l'aide du système de fichiers [XFS](https://xfs.wiki.kernel.org/).

```bash
sudo parted /dev/sdc --script mklabel gpt mkpart xfspart xfs 0% 100%
sudo mkfs.xfs /dev/sdc1
sudo partprobe /dev/sdc1
```

Utilisez l'utilitaire [`partprobe`](https://linux.die.net/man/8/partprobe) pour vérifier que le noyau tient compte de la nouvelle partition et du nouveau système de fichiers. Si vous ne parvenez pas à utiliser `partprobe`, les commandes blkid ou lslbk ne renvoient pas immédiatement l'UUID du nouveau système de fichiers.


### <a name="mount-the-disk"></a>Monter le disque

À présent, créez un répertoire afin de monter le système de fichiers à l’aide de `mkdir`. L'exemple suivant crée un répertoire sous `/datadrive` :

```bash
sudo mkdir /datadrive
```

Utilisez `mount` pour monter le système de fichiers. L'exemple suivant monte la partition `/dev/sdc1` sur le point de montage `/datadrive` :

```bash
sudo mount /dev/sdc1 /datadrive
```

### <a name="persist-the-mount"></a>Rendre le montage permanent

Pour vous assurer que le lecteur est remonté automatiquement après un redémarrage, vous devez l’ajouter au fichier */etc/fstab*. Il est aussi vivement recommandé d'utiliser l'UUID (identificateur unique universel) dans */etc/fstab* pour faire référence au lecteur plutôt que de se contenter du nom de l'appareil (par exemple, */dev/sdc1*). Si le système d’exploitation détecte une erreur disque pendant le démarrage, l’utilisation de l’UUID évite que le disque incorrect ne soit monté sur un emplacement donné. Les disques de données restants reçoivent alors les mêmes ID d’appareil. Pour rechercher l’UUID du nouveau lecteur, utilisez l’utilitaire `blkid` :

```bash
sudo blkid
```

La sortie doit ressembler à celle-ci :

```bash
/dev/sda1: LABEL="cloudimg-rootfs" UUID="11111111-1b1b-1c1c-1d1d-1e1e1e1e1e1e" TYPE="ext4" PARTUUID="1a1b1c1d-11aa-1234-1a1a1a1a1a1a"
/dev/sda15: LABEL="UEFI" UUID="BCD7-96A6" TYPE="vfat" PARTUUID="1e1g1cg1h-11aa-1234-1u1u1a1a1u1u"
/dev/sdb1: UUID="22222222-2b2b-2c2c-2d2d-2e2e2e2e2e2e" TYPE="ext4" TYPE="ext4" PARTUUID="1a2b3c4d-01"
/dev/sda14: PARTUUID="2e2g2cg2h-11aa-1234-1u1u1a1a1u1u"
/dev/sdc1: UUID="33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e" TYPE="xfs" PARTLABEL="xfspart" PARTUUID="c1c2c3c4-1234-cdef-asdf3456ghjk"
```

> [!NOTE]
> si vous ne modifiez pas correctement le fichier **/etc/fstab** , il se peut que le système ne puisse plus démarrer. En cas de doute, reportez-vous à la documentation de la distribution pour obtenir des informations sur la modification adéquate de ce fichier. Il est par ailleurs vivement recommandé de créer une sauvegarde du fichier /etc/fstab avant de le modifier.

Ensuite, ouvrez le fichier */etc/fstab* dans un éditeur de texte, comme suit :

```bash
sudo nano /etc/fstab
```

Dans cet exemple, utilisez la valeur UUID de l'appareil `/dev/sdc1` créé lors des étapes précédentes et le point de montage `/datadrive`. Ajoutez la ligne suivante à la fin du fichier `/etc/fstab` :

```bash
UUID=33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e   /datadrive   xfs   defaults,nofail   1   2
```

Dans cet exemple, nous utilisons l'éditeur nano. Par conséquent, lorsque vous avez fini de modifier le fichier, utilisez `Ctrl+O` pour écrire le fichier et `Ctrl+X` pour quitter l'éditeur.

> [!NOTE]
> La suppression ultérieure d’un disque de données sans modifier fstab pourrait entraîner l’échec du démarrage de la machine virtuelle. La plupart des distributions fournissent les options fstab *nofail* et/ou *nobootwait*. Ces options permettent à un système de démarrer même si le disque n’est pas monté au moment du démarrage. Pour plus d’informations sur ces paramètres, consultez la documentation de votre distribution.
>
> L’option *nofail* garantit que la machine virtuelle démarre même si le système de fichiers est endommagé ou si le disque n’existe pas au moment du démarrage. Sans cette option, vous pouvez être confronté au comportement décrit dans [Cannot SSH to Linux VM due to FSTAB errors](/archive/blogs/linuxonazure/cannot-ssh-to-linux-vm-after-adding-data-disk-to-etcfstab-and-rebooting) (Connexion SSH vers machine virtuelle Linux impossible en raison d’erreurs FSTAB)
>
> La console série de machine virtuelle Azure peut servir pour accéder à la console sur votre machine virtuelle si la modification de fstab a entraîné un échec de démarrage. Plus de détails dans la [documentation relative à la console série](../troubleshooting/serial-console-linux.md).

### <a name="trimunmap-support-for-linux-in-azure"></a>Prise en charge de TRIM/UNMAP pour Linux dans Azure
Certains noyaux Linux prennent en charge les opérations TRIM/UNMAP pour ignorer les blocs inutilisés sur le disque. Cette fonctionnalité est particulièrement utile dans le stockage standard pour informer Azure que des pages supprimées ne sont plus valides et peuvent être ignorées. Elle peut vous permettre d’économiser de l’argent si vous créez des fichiers volumineux, puis que vous les supprimez.

Il existe deux façons d’activer la prise en charge de TRIM sur votre machine virtuelle Linux. Comme d’habitude, consultez votre distribution pour connaître l’approche recommandée :

* Utilisez l’option de montage `discard` dans */etc/fstab*, par exemple :

    ```bash
    UUID=33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e   /datadrive   xfs   defaults,discard   1   2
    ```
* Dans certains cas, l’option `discard` peut avoir un impact sur la performance. Vous pouvez également exécuter la commande `fstrim` manuellement à partir de la ligne de commande ou l’ajouter à votre crontab pour l’exécuter régulièrement :

    **Ubuntu**

    ```bash
    sudo apt-get install util-linux
    sudo fstrim /datadrive
    ```

    **RHEL/CentOS**

    ```bash
    sudo yum install util-linux
    sudo fstrim /datadrive
    ```

## <a name="troubleshooting"></a>Dépannage

[!INCLUDE [virtual-machines-linux-lunzero](../../../includes/virtual-machines-linux-lunzero.md)]

## <a name="next-steps"></a>Étapes suivantes

* Pour vous assurer que votre machine virtuelle Linux est correctement configurée, passez en revue les recommandations visant à [optimiser les performances de votre machine virtuelle Linux](optimization.md) .
* Développez votre capacité de stockage en ajoutant des disques supplémentaires et [configurez RAID](/previous-versions/azure/virtual-machines/linux/configure-raid) pour augmenter les performances.