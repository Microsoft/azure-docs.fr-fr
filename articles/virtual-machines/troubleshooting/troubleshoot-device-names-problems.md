---
title: Résoudre les problèmes liés aux modifications des noms de périphérique de machine virtuelle Linux dans Azure | Microsoft Docs
description: Explique pourquoi les noms de périphérique de machine virtuelle Linux changent et comment résoudre ce problème.
services: virtual-machines-linux
documentationcenter: ''
author: genlin
manager: dcscontentpm
editor: ''
tags: ''
ms.service: virtual-machines-linux
ms.topic: troubleshooting
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.devlang: azurecli
ms.date: 11/01/2018
ms.author: genli
ms.openlocfilehash: 31f64a504156134b1d622705d5301d9cd5a5f5b1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88756824"
---
# <a name="troubleshoot-linux-vm-device-name-changes"></a>Résoudre les problèmes liés aux modifications des noms de périphérique de machine virtuelle Linux

Cet article explique pourquoi les noms de périphérique changent une fois que vous redémarrez une machine virtuelle Linux ou rattachez les disques de données. L’article fournit également des solutions pour résoudre ce problème.

## <a name="symptoms"></a>Symptômes
Vous pourriez rencontrer les problèmes suivants lors de l’exécution de machines virtuelles Linux dans Microsoft Azure :

- La machine virtuelle ne parvient pas à démarrer après un redémarrage.
- Quand vous détachez et rattachez des disques de données, les noms de périphérique de disque changent.
- Une application ou un script qui fait référence à un disque en utilisant le nom de périphérique échoue, car le nom du périphérique a changé.

## <a name="cause"></a>Cause

Les chemins d’accès des périphériques dans Linux ne sont pas toujours cohérents entre les redémarrages. Les noms de périphérique se composent de numéros principaux (lettre) et secondaires. Quand le pilote du périphérique de stockage Linux détecte un nouveau périphérique, il lui attribue un numéro principal et un numéro secondaire à partir de la plage disponible pour lui. Quand vous retirez un périphérique, ses numéros peuvent être réutilisés.

Le problème se produit parce que l’analyse des périphériques dans Linux, telle qu’elle est planifiée par le sous-système SCSI, est asynchrone. Par conséquent, le nom du chemin d’un périphérique peut varier d’un redémarrage à l’autre.

## <a name="solution"></a>Solution

Pour résoudre ce problème, utilisez un dispositif d’affectation de noms persistants. Il existe quatre façons d’utiliser l’affectation de noms persistants : étiquette de système de fichiers, UUID, ID ou chemin. Nous vous recommandons d’utiliser l’étiquette de système de fichiers ou un UUID pour les machines virtuelles Linux Azure.

La plupart des distributions fournissent les paramètres `fstab` **nofail** ou **nobootwait**. Ces paramètres permettent à un système de démarrer quand le disque n’est pas monté au moment du démarrage. Pour plus d’informations sur ces paramètres, consultez la documentation de votre distribution. Pour plus d’informations sur la manière de configurer une machine virtuelle Linux pour qu’elle utiliser un UUID lorsque vous ajoutez un disque de données, consultez la section [Se connecter à la machine virtuelle Linux afin de monter le nouveau disque](../linux/add-disk.md#format-and-mount-the-disk).

Quand l’agent Linux Azure est installé sur une machine virtuelle, il utilise des règles Udev pour construire un ensemble de liens symboliques sous le chemin /dev/disk/azure. Les applications et les scripts utilisent des règles Udev pour identifier les disques attachés à la machine virtuelle, ainsi que le type de disque et les numéros d’unités logiques de disque.

Si vous avez déjà modifié votre fichier FSTAB de telle sorte que votre machine virtuelle ne démarre pas et que vous n'êtes pas en mesure d'établir une connexion SSH avec elle, vous pouvez utiliser la [console série de machine virtuelle](./serial-console-linux.md) pour passer en [mode mono-utilisateur](./serial-console-grub-single-user-mode.md) et modifier votre fichier FSTAB.

### <a name="identify-disk-luns"></a>Identifier les numéros d’unité logique des disques

Les applications utilisent des numéros d’unités logiques pour trouver tous les disques attachés et pour construire des liens symboliques. L’agent Linux Azure inclut des règles Udev permettant de configurer des liens symboliques entre un numéro d’unité logique et les périphériques :

```console
$ tree /dev/disk/azure

/dev/disk/azure
├── resource -> ../../sdb
├── resource-part1 -> ../../sdb1
├── root -> ../../sda
├── root-part1 -> ../../sda1
└── scsi1
    ├── lun0 -> ../../../sdc
    ├── lun0-part1 -> ../../../sdc1
    ├── lun1 -> ../../../sdd
    ├── lun1-part1 -> ../../../sdd1
    ├── lun1-part2 -> ../../../sdd2
    └── lun1-part3 -> ../../../sdd3
```

Pour récupérer les informations de numéro d’unité logique du compte invité Linux, utilisez `lsscsi` ou un outil similaire :

```console
$ sudo lsscsi

[1:0:0:0] cd/dvd Msft Virtual CD/ROM 1.0 /dev/sr0

[2:0:0:0] disk Msft Virtual Disk 1.0 /dev/sda

[3:0:1:0] disk Msft Virtual Disk 1.0 /dev/sdb

[5:0:0:0] disk Msft Virtual Disk 1.0 /dev/sdc

[5:0:0:1] disk Msft Virtual Disk 1.0 /dev/sdd
```

Les informations de numéro d’unité logique de l’invité sont utilisées avec les métadonnées d’abonnement Azure pour localiser le disque dur virtuel dans le stockage Azure qui contient les données de partition. Par exemple, vous pouvez utiliser `az` CLI :

```azurecli
$ az vm show --resource-group testVM --name testVM | jq -r .storageProfile.dataDisks
[
{
"caching": "None",
  "createOption": "empty",
"diskSizeGb": 1023,
  "image": null,
"lun": 0,
"managedDisk": null,
"name": "testVM-20170619-114353",
"vhd": {
  "uri": "https://testVM.blob.core.windows.net/vhd/testVM-20170619-114353.vhd"
}
},
{
"caching": "None",
"createOption": "empty",
"diskSizeGb": 512,
"image": null,
"lun": 1,
"managedDisk": null,
"name": "testVM-20170619-121516",
"vhd": {
  "uri": "https://testVM.blob.core.windows.net/vhd/testVM-20170619-121516.vhd"
  }
  }
]
```

### <a name="discover-filesystem-uuids-by-using-blkid"></a>Découvrir les UUID de système de fichiers à l’aide de l’utilitaire blkid

Les applications et les scripts lisent la sortie de `blkid`, ou des sources d’information similaires, pour construire des liens symboliques dans le chemin /dev. La sortie montre les UUID de tous les disques attachés à la machine virtuelle et leur fichier de périphérique associé :

```console
$ sudo blkid -s UUID

/dev/sr0: UUID="120B021372645f72"
/dev/sda1: UUID="52c6959b-79b0-4bdd-8ed6-71e0ba782fb4"
/dev/sdb1: UUID="176250df-9c7c-436f-94e4-d13f9bdea744"
/dev/sdc1: UUID="b0048738-4ecc-4837-9793-49ce296d2692"
```

Les règles Udev de l’agent Linux Azure construisent un ensemble de liens symboliques sous le chemin /dev/disk/azure :

```console
$ ls -l /dev/disk/azure

total 0
lrwxrwxrwx 1 root root  9 Jun  2 23:17 resource -> ../../sdb
lrwxrwxrwx 1 root root 10 Jun  2 23:17 resource-part1 -> ../../sdb1
lrwxrwxrwx 1 root root  9 Jun  2 23:17 root -> ../../sda
lrwxrwxrwx 1 root root 10 Jun  2 23:17 root-part1 -> ../../sda1
```

Les applications utilisent les liens pour identifier le périphérique de disque de démarrage et le disque de ressources (éphémère). Dans Azure, les applications doivent rechercher dans les chemins /dev/disk/azure/root-part1 ou /dev/disk/azure-resource-part1 pour découvrir ces partitions.

Toutes les partitions supplémentaires figurant dans la liste `blkid` résident sur un disque de données. Les applications gèrent les UUID de ces partitions et utilisent un chemin pour découvrir le nom du périphérique au moment de l’exécution :

```console
$ ls -l /dev/disk/by-uuid/b0048738-4ecc-4837-9793-49ce296d2692

lrwxrwxrwx 1 root root 10 Jun 19 15:57 /dev/disk/by-uuid/b0048738-4ecc-4837-9793-49ce296d2692 -> ../../sdc1
```


### <a name="get-the-latest-azure-storage-rules"></a>Obtenir les dernières règles de stockage Azure

Pour obtenir les dernières règles de stockage Azure, exécutez les commandes suivantes :

```console
# sudo curl -o /etc/udev/rules.d/66-azure-storage.rules https://raw.githubusercontent.com/Azure/WALinuxAgent/master/config/66-azure-storage.rules
# sudo udevadm trigger --subsystem-match=block
```

## <a name="see-also"></a>Voir aussi

Pour plus d’informations, consultez les articles suivants :

- [Ubuntu : Utilisation des UUID](https://help.ubuntu.com/community/UsingUUID)
- [Red Hat : Attribution de noms persistants](https://access.redhat.com/documentation/en-US/Red_Hat_Enterprise_Linux/7/html/Storage_Administration_Guide/persistent_naming.html)
- [Linux : Ce que les UUID peuvent faire pour vous](https://www.linux.com/news/what-uuids-can-do-you)
- [Udev : Présentation de la gestion des périphériques dans les systèmes Linux modernes](https://www.linux.com/news/udev-introduction-device-management-modern-linux-system)

