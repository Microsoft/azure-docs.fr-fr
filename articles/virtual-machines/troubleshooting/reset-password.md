---
title: Réinitialiser votre mot de passe Linux local sur les machines virtuelles Azure | Microsoft Docs
description: Introduction des étapes pour réinitialiser votre mot de passe Linux local sur la machine virtuelle Azure
services: virtual-machines-linux
documentationcenter: ''
author: Deland-Han
manager: dcscontentpm
editor: ''
tags: ''
ms.service: virtual-machines-linux
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-linux
ms.topic: troubleshooting
ms.date: 08/20/2019
ms.author: delhan
ms.openlocfilehash: c6bfd5b9ff3626593916533f27c5c2755cebcb13
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87028479"
---
# <a name="how-to-reset-local-linux-password-on-azure-vms"></a>Réinitialiser votre mot de passe Linux local sur les machines virtuelles Azure

Cet article présente plusieurs méthodes pour réinitialiser les mots de passe Linux locaux sur les machines virtuelles. Si le compte utilisateur a expiré ou si vous souhaitez simplement créer un nouveau compte, utilisez les méthodes suivantes pour créer un nouveau compte d’administrateur local pour retrouver l’accès à la machine virtuelle.

## <a name="symptoms"></a>Symptômes

Vous ne pouvez pas vous connecter à la machine virtuelle, et vous recevez un message vous informant que votre mot de passe est incorrect. De plus, vous ne pouvez pas utiliser VMAgent pour réinitialiser votre mot de passe sur le portail Azure.

## <a name="manual-password-reset-procedure"></a>Procédure de réinitialisation manuelle du mot de passe

> [!NOTE]
> Les étapes suivantes ne s’appliquent pas aux machines virtuelles équipées d’un disque non managé.

1. Prenez un instantané du disque du système d’exploitation de la machine virtuelle affectée, créez un disque à partir de l’instantané, puis attachez le disque à une machine virtuelle de dépannage. Pour en savoir plus, consultez [Résoudre les problèmes d’une machine virtuelle Windows en connectant le disque du système d’exploitation à une machine virtuelle de récupération à l’aide du portail Azure](troubleshoot-recovery-disks-portal-linux.md).

2. Connectez-vous à la machine virtuelle de dépannage à l’aide du Bureau à distance.

3.  Exécutez la commande SSH suivante sur la machine virtuelle de dépannage pour devenir un super utilisateur.

    ```bash
    sudo su
    ```

4.  Exécutez **fdisk -l** ou consultez les journaux d’activité système pour trouver le disque joint. Recherchez le nom du lecteur à monter. Sur la machine virtuelle temporaire, cherchez dans le fichier journal correspondant.

    ```bash
    grep SCSI /var/log/kern.log (ubuntu)
    grep SCSI /var/log/messages (centos, suse, oracle)
    ```

    Voici un exemple de sortie de la commande grep :

    ```bash
    kernel: [ 9707.100572] sd 3:0:0:0: [sdc] Attached SCSI disk
    ```

5.  Créer un point de montage nommé **tempmount**.

    ```bash
    mkdir /tempmount
    ```

6.  Monter le disque du système d’exploitation sur le point de montage. Vous devez généralement monter *sdc1* ou *sdc2*. Tout dépend de la partition d’hébergement dans le répertoire */etc* du disque de la machine défaillante.

    ```bash
    mount /dev/sdc1 /tempmount
    ```

7.  Créez des copies des principaux fichiers d’informations d’identification avant d’apporter des modifications :

    ```bash
    cp /etc/passwd /etc/passwd_orig    
    cp /etc/shadow /etc/shadow_orig    
    cp /tempmount/etc/passwd /etc/passwd
    cp /tempmount/etc/shadow /etc/shadow 
    cp /tempmount/etc/passwd /tempmount/etc/passwd_orig
    cp /tempmount/etc/shadow /tempmount/etc/shadow_orig
    ```

8.  Réinitialisez le mot de passe de l'utilisateur dont vous avez besoin :

    ```bash
    passwd <<USER>> 
    ```

9.  Déplacez les fichiers modifiés dans le bon dossier sur le disque de la machine défaillante.

    ```bash
    cp /etc/passwd /tempmount/etc/passwd
    cp /etc/shadow /tempmount/etc/shadow
    cp /etc/passwd_orig /etc/passwd
    cp /etc/shadow_orig /etc/shadow
    ```

10. Revenez à la racine et démontez le disque.

    ```bash
    cd /
    umount /tempmount
    ```

11. Dans le portail Azure, détachez le disque de la machine virtuelle de dépannage.

12. [Changez le disque de système d’exploitation pour la machine virtuelle affectée](troubleshoot-recovery-disks-portal-linux.md#swap-the-os-disk-for-the-vm).

## <a name="next-steps"></a>Étapes suivantes

* [Résoudre les problèmes de machine virtuelle Azure en attachant le disque du système d’exploitation à une autre machine virtuelle Azure](https://social.technet.microsoft.com/wiki/contents/articles/18710.troubleshoot-azure-vm-by-attaching-os-disk-to-another-azure-vm.aspx)

* [Azure CLI : guide pratique pour supprimer et redéployer une machine virtuelle à partir d’un disque dur virtuel](/archive/blogs/linuxonazure/azure-cli-how-to-delete-and-re-deploy-a-vm-from-vhd)
