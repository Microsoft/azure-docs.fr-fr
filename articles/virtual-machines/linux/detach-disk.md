---
title: Détacher un disque de données d’une machine virtuelle Linux - Azure
description: Apprenez à détacher un disque de données d’une machine virtuelle dans Azure à l’aide d’Azure CLI ou du portail Azure.
author: roygara
ms.service: virtual-machines-linux
ms.topic: how-to
ms.date: 07/18/2018
ms.author: rogarana
ms.subservice: disks
ms.custom: devx-track-azurecli
ms.openlocfilehash: 41f70a476e5cc562207614c4e04658d5f0fc6d09
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87499515"
---
# <a name="how-to-detach-a-data-disk-from-a-linux-virtual-machine"></a>Comment détacher un disque de données d’une machine virtuelle Linux

Lorsque vous n’avez plus besoin d’un disque de données qui est attaché à une machine virtuelle, vous pouvez le détacher facilement. Cela supprime le disque de la machine virtuelle, mais pas du stockage. Dans cet article, nous utilisons une distribution Ubuntu LTS 16.04. Si vous utilisez une autre distribution, les instructions permettant de démonter le disque peuvent être différentes.

> [!WARNING]
> Si vous détachez un disque, il n’est pas supprimé automatiquement. Si vous êtes abonné au stockage Premium, vous continuerez à engager des frais de stockage pour le disque. Pour plus d’informations, consultez [Tarifs et facturation du stockage Premium](https://azure.microsoft.com/pricing/details/storage/page-blobs/).

Si vous souhaitez réutiliser les données du disque, vous pouvez l’attacher à la même machine virtuelle ou à une autre.  


## <a name="connect-to-the-vm-to-unmount-the-disk"></a>Se connecter à la machine virtuelle pour démonter le disque

Avant de pouvoir détacher le disque à l’aide de l’interface de ligne de commande ou du portail, vous devez démonter le disque et supprimer les références à celui-ci à partir de votre fichier fstab.

Connectez-vous à la machine virtuelle. Dans cet exemple, l’adresse IP publique de la machine virtuelle est *10.0.1.4* avec le nom d’utilisateur *azureuser* : 

```bash
ssh azureuser@10.0.1.4
```

Tout d’abord, recherchez le disque de données que vous souhaitez détacher. L’exemple suivant utilise dmesg pour filtrer les disques SCSI :

```bash
dmesg | grep SCSI
```

Le résultat ressemble à l’exemple suivant :

```bash
[    0.294784] SCSI subsystem initialized
[    0.573458] Block layer SCSI generic (bsg) driver version 0.4 loaded (major 252)
[    7.110271] sd 2:0:0:0: [sda] Attached SCSI disk
[    8.079653] sd 3:0:1:0: [sdb] Attached SCSI disk
[ 1828.162306] sd 5:0:0:0: [sdc] Attached SCSI disk
```

Ici, *sdc* est le disque que nous voulons détacher. Vous devez également récupérer l’UUID du disque.

```bash
sudo -i blkid
```

Le résultat ressemble à l’exemple qui suit :

```bash
/dev/sda1: UUID="11111111-1b1b-1c1c-1d1d-1e1e1e1e1e1e" TYPE="ext4"
/dev/sdb1: UUID="22222222-2b2b-2c2c-2d2d-2e2e2e2e2e2e" TYPE="ext4"
/dev/sdc1: UUID="33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e" TYPE="ext4"
```


Modifiez le fichier */etc/fstab* pour supprimer les références au disque. 

> [!NOTE]
> si vous ne modifiez pas correctement le fichier **/etc/fstab** , il se peut que le système ne puisse plus démarrer. En cas de doute, reportez-vous à la documentation de la distribution pour obtenir des informations sur la modification adéquate de ce fichier. Il est par ailleurs vivement recommandé de créer une sauvegarde du fichier /etc/fstab avant de le modifier.

Ouvrez le fichier */etc/fstab* dans un éditeur de texte comme suit :

```bash
sudo vi /etc/fstab
```

Dans cet exemple, la ligne suivante doit être supprimée du fichier */etc/fstab* :

```bash
UUID=33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e   /datadrive   ext4   defaults,nofail   1   2
```

Utilisez `umount` pour démonter le disque. L’exemple suivant démonte la partition */dev/sdc1* du point de montage */datadrive* :

```bash
sudo umount /dev/sdc1 /datadrive
```


## <a name="detach-a-data-disk-using-azure-cli"></a>Détacher un disque de données à l’aide d’Azure CLI 

Cet exemple détache le disque *myDataDisk* de la machine virtuelle nommée *myVM* dans *myResourceGroup*.

```azurecli
az vm disk detach \
    -g myResourceGroup \
    --vm-name myVm \
    -n myDataDisk
```

Le disque reste dans le stockage, mais il n’est plus attaché à une machine virtuelle.


## <a name="detach-a-data-disk-using-the-portal"></a>Détacher un disque de données avec le portail

1. Dans le menu de gauche, sélectionnez **Machines virtuelles**.
1. Dans le panneau de la machine virtuelle, sélectionnez **Disques**.
1. En haut du panneau **Disques**, sélectionnez **Modifier**.
1. Dans le panneau **Disques**, à l’extrême droite du disque de données que vous souhaitez détacher, cliquez sur le bouton détacher ![image du bouton détacher](./media/detach-disk/detach.png).
1. Une fois que le disque a été supprimé, cliquez sur **Enregistrer** en haut du panneau.

Le disque reste dans le stockage, mais il n’est plus attaché à une machine virtuelle.



## <a name="next-steps"></a>Étapes suivantes
Si vous souhaitez réutiliser le disque de données, vous pouvez simplement l’[attacher à une autre machine virtuelle](add-disk.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).

