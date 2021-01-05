---
title: Attacher un disque de données à une machine virtuelle Linux
description: Utilisez le portail pour attacher un disque de données nouveau ou existant à une machine virtuelle Linux.
author: cynthn
ms.service: virtual-machines-linux
ms.topic: how-to
ms.date: 08/28/2020
ms.author: cynthn
ms.subservice: disks
ms.openlocfilehash: 8f60c83417e9c614ca30f140e6acbbf08e5643cf
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96500646"
---
# <a name="use-the-portal-to-attach-a-data-disk-to-a-linux-vm"></a>Utiliser le portail pour attacher un disque de données à une machine virtuelle Linux 
Cet article vous explique comment attacher des disques nouveaux et existants à une machine virtuelle Linux par le biais du portail Azure. Vous pouvez également [attacher un disque de données à une machine virtuelle Windows dans le Portail Azure](../windows/attach-managed-disk-portal.md?toc=%2fazure%2fvirtual-machines%2fwindows%2ftoc.json). 

Avant d’attacher des disques à votre machine virtuelle, lisez les conseils suivants :

* La taille de la machine virtuelle détermine le nombre de disques de données que vous pouvez attacher . Pour en savoir plus, voir la rubrique [Tailles de machines virtuelles](../sizes.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).
* Les disques attachés aux machines virtuelles sont en réalité des fichiers .vhd stockés dans Azure. Pour plus d’informations, consultez [Introduction aux disques managés](../managed-disks-overview.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json).
* Après avoir attaché le disque, vous devez vous [connecter à la machine virtuelle Linux pour monter le nouveau disque](#connect-to-the-linux-vm-to-mount-the-new-disk).


## <a name="find-the-virtual-machine"></a>Recherchez la machine virtuelle.
1. Accédez au [Portail Azure](https://portal.azure.com/) pour rechercher la machine virtuelle. Recherchez et sélectionnez **Machines virtuelles**.
2. Choisissez la machine virtuelle dans la liste.
3. Sur la page **Machines virtuelles**, sous **Paramètres**, sélectionnez **Disques**.


## <a name="attach-a-new-disk"></a>Attacher un nouveau disque

1. Dans le volet **Disques**, sous **Disques de données**, sélectionnez **Créer et attacher un disque**.

1. Entrez un nom pour votre disque géré. Examinez les paramètres par défaut et mettez à jour **Type de stockage**, **Taille (Gio)** , **Chiffrement** et **Mise en cache de l'hôte**, si nécessaire.
   
   :::image type="content" source="./media/attach-disk-portal/create-new-md.png" alt-text="Passez en revue les paramètres du disque.":::


1. Lorsque vous avez terminé, sélectionnez **Enregistrer** en haut de la page pour créer le disque managé et mettre à jour la configuration de la machine virtuelle.


## <a name="attach-an-existing-disk"></a>Association d'un disque existant
1. Dans le volet **Disques**, sous **Disques de données**, sélectionnez **Attacher des disques existants**.
1. Cliquez sur le menu déroulant **Nom du disque** et sélectionnez un disque dans la liste des disques managés disponibles. 

1. Cliquez sur **Enregistrer** pour attacher le disque géré existant et mettre à jour la configuration de la machine virtuelle :
   

## <a name="connect-to-the-linux-vm-to-mount-the-new-disk"></a>Se connecter à la machine virtuelle Linux afin de monter le nouveau disque
Vous devez exécuter SSH dans votre machine virtuelle Azure afin de partitionner, de formater et de monter votre nouveau disque pour que votre machine virtuelle Linux puisse l’utiliser. Pour plus d’informations, consultez l’article [Utilisation de SSH avec Linux sur Azure](mac-create-ssh-keys.md). L’exemple suivant établit une connexion à une machine virtuelle à l’aide de l’adresse IP publique *10.123.123.25* avec le nom d’utilisateur *azureuser* : 

```bash
ssh azureuser@10.123.123.25
```

## <a name="find-the-disk"></a>Recherche du disque

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
sdc     3:0:0:0       4G
```

Dans cet exemple, le disque ajouté est `sdc`. Il s’agit d’un numéro d'unité logique 0 avec 4 Go.

Pour un exemple plus complexe, voici à quoi ressemble plusieurs disques de données dans le portail :

:::image type="content" source="./media/attach-disk-portal/create-new-md.png" alt-text="Passez en revue les paramètres du disque.":::

Dans l’image, vous pouvez voir qu’il existe 3 disques de données : 4 Go sur le numéro d'unité logique 0, 16 Go sur le numéro d'unité logique 1 et 32 Go sur le numéro d'unité logique 2.

Voici à quoi peut ressembler l’utilisation de `lsblk` :

```bash
sda     0:0:0:0      30G
├─sda1             29.9G /
├─sda14               4M
└─sda15             106M /boot/efi
sdb     1:0:1:0      14G
└─sdb1               14G /mnt
sdc     3:0:0:0       4G
sdd     3:0:0:1      16G
sde     3:0:0:2      32G
```

À partir de la sortie de `lsblk` vous pouvez voir que le disque de 4 Go sur le numéro d’unité logique 0 correspond à `sdc`, le disque de 16 Go sur le numéro d’unité logique 1 correspond à `sdd` et le disque 32 Go sur le numéro d'unité logique 2 correspond à `sde`.

### <a name="partition-a-new-disk"></a>Partitionnement d’un nouveau disque

Si vous utilisez un disque existant contenant des données, passez au montage du disque. Si vous attachez un nouveau disque, vous devez partitionner ce disque.

L’utilitaire `parted` peut être utilisé pour partitionner et formater un disque de données.

> [!NOTE]
> Il est recommandé d’utiliser la dernière version de `parted` disponible pour votre distribution.
> Si la taille du disque est supérieure ou égale à 2 Tio, vous devez utiliser le partitionnement GPT. Si la taille du disque est inférieure à 2 Tio, vous pouvez utiliser le partitionnement MBR ou GPT.  


L’exemple suivant utilise `parted` sur `/dev/sdc`, où le premier disque de données se trouve généralement sur la plupart des machines virtuelles. Remplacez `sdc` par l’option adaptée à votre disque. Nous le formatons également à l’aide du système de fichiers [XFS](https://xfs.wiki.kernel.org/) .

```bash
sudo parted /dev/sdc --script mklabel gpt mkpart xfspart xfs 0% 100%
sudo mkfs.xfs /dev/sdc1
sudo partprobe /dev/sdc1
```

Utilisez l’utilitaire [`partprobe`](https://linux.die.net/man/8/partprobe) pour vérifier que le noyau tient compte de la nouvelle partition et du nouveau système de fichiers. Si vous ne parvenez pas à utiliser `partprobe`, les commandes blkid ou lslbk ne retournent pas immédiatement l’UUID du nouveau système de fichiers.

### <a name="mount-the-disk"></a>Monter le disque

Créez un répertoire afin de monter le système de fichiers à l’aide de `mkdir`. L’exemple suivant crée un répertoire dans `/datadrive` :

```bash
sudo mkdir /datadrive
```

Utilisez `mount` pour monter le système de fichiers. L’exemple suivant monte la partition */dev/sdc1* pour le point de montage `/datadrive` :

```bash
sudo mount /dev/sdc1 /datadrive
```

Pour vous assurer que le lecteur est remonté automatiquement après un redémarrage, vous devez l’ajouter au fichier */etc/fstab*. Il est aussi vivement recommandé d’utiliser l’UUID (identificateur global unique) dans */etc/fstab* comme référence au lecteur, plutôt que le nom d’appareil uniquement (par exemple, */dev/sdc1*). Si le système d’exploitation détecte une erreur disque pendant le démarrage, l’utilisation de l’UUID évite que le disque incorrect ne soit monté sur un emplacement donné. Les disques de données restants reçoivent alors les mêmes ID d’appareil. Pour rechercher l’UUID du nouveau lecteur, utilisez l’utilitaire `blkid` :

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

Dans cet exemple, utilisez la valeur UUID pour l’appareil `/dev/sdc1` créé lors des étapes précédentes et le point de montage `/datadrive`. Ajoutez la ligne suivante à la fin du fichier `/etc/fstab` :

```bash
UUID=33333333-3b3b-3c3c-3d3d-3e3e3e3e3e3e   /datadrive   xfs   defaults,nofail   1   2
```

Nous avons utilisé l’éditeur nano et dès lors, une fois que vous avez fini de modifier le fichier, utilisez `Ctrl+O` pour écrire le fichier et `Ctrl+X` pour quitter l’éditeur.

> [!NOTE]
> La suppression ultérieure d’un disque de données sans modifier fstab pourrait entraîner l’échec du démarrage de la machine virtuelle. La plupart des distributions fournissent les options fstab *nofail* et/ou *nobootwait*. Ces options permettent à un système de démarrer même si le disque n’est pas monté au moment du démarrage. Pour plus d’informations sur ces paramètres, consultez la documentation de votre distribution.
> 
> L’option *nofail* garantit que la machine virtuelle démarre même si le système de fichiers est endommagé ou si le disque n’existe pas au moment du démarrage. Sans cette option, vous pouvez être confronté au comportement décrit dans [Cannot SSH to Linux VM due to FSTAB errors](/archive/blogs/linuxonazure/cannot-ssh-to-linux-vm-after-adding-data-disk-to-etcfstab-and-rebooting) (Connexion SSH vers machine virtuelle Linux impossible en raison d’erreurs FSTAB)


## <a name="verify-the-disk"></a>Vérifier le disque

Vous pouvez maintenant utiliser `lsblk` à nouveau pour voir le disque et le point de montage.

```bash
lsblk -o NAME,HCTL,SIZE,MOUNTPOINT | grep -i "sd"
```

Vous obtenez un résultat semblable à ce qui suit :

```bash
sda     0:0:0:0      30G
├─sda1             29.9G /
├─sda14               4M
└─sda15             106M /boot/efi
sdb     1:0:1:0      14G
└─sdb1               14G /mnt
sdc     3:0:0:0       4G
└─sdc1                4G /datadrive
```

Vous pouvez constater que `sdc` est maintenant monté sur `/datadrive`.

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

## <a name="next-steps"></a>Étapes suivantes
Vous pouvez également [attacher un disque de données](add-disk.md) à l’aide d’Azure CLI.