---
title: Dépanner une machine virtuelle Azure défaillante à l’aide d’une virtualisation imbriquée dans Azure | Microsoft Docs
description: Guide pratique pour dépanner une machine virtuelle Azure posant problème à l’aide de la virtualisation imbriquée dans Azure
services: virtual-machines-windows
documentationcenter: ''
author: glimoli
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.topic: article
ms.date: 11/19/2019
ms.author: genli
ms.openlocfilehash: f950cb63b5083a85ab5420434abdd9a720115b1a
ms.sourcegitcommit: c2dd51aeaec24cd18f2e4e77d268de5bcc89e4a7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94734546"
---
# <a name="troubleshoot-a-faulty-azure-vm-by-using-nested-virtualization-in-azure"></a>Dépanner une machine virtuelle Azure défaillante à l’aide d’une virtualisation imbriquée dans Azure

Cet article explique comment créer un environnement de virtualisation imbriqué dans Microsoft Azure pour pouvoir ensuite monter le disque de la machine virtuelle défaillante à dépanner sur l’hôte Hyper-V (machine virtuelle de secours).

## <a name="prerequisites"></a>Prérequis

Pour monter la machine virtuelle défaillante, la machine virtuelle de secours doit utiliser le même type de compte de stockage (Standard ou Premium) que la machine virtuelle défaillante.

## <a name="step-1-create-a-rescue-vm-and-install-hyper-v-role"></a>Étape 1 : Créer une machine virtuelle de secours et installer le rôle Hyper-V

1.  Créez une machine virtuelle de secours :

    -  Système d’exploitation : Windows Server 2016 Datacenter

    -  Taille : n’importe quelle machine de la gamme V3 avec au moins deux cœurs prenant en charge la virtualisation imbriquée. Pour plus d’informations, consultez le blog [Introducing the new Dv3 and Ev3 VM sizes](https://azure.microsoft.com/blog/introducing-the-new-dv3-and-ev3-vm-sizes/).

    -  Mêmes emplacement, compte de stockage et groupe de ressources que la machine virtuelle défaillante.

    -  Sélectionnez le même type de stockage (Standard ou Premium) que la machine virtuelle défaillante.

2.  Après avoir créé la machine virtuelle de secours, établissez une connexion Bureau à distance à cette machine virtuelle.

3.  Dans le Gestionnaire de serveur, sélectionnez **Gérer** > **Ajouter des rôles et des fonctionnalités**.

4.  Dans la section **Type d’installation**, sélectionnez **Installation basée sur un rôle ou une fonctionnalité**.

5.  Dans la section **Sélectionner le serveur de destination**, vérifiez que la machine virtuelle de secours est sélectionnée.

6.  Sélectionnez le **rôle Hyper-V** > **Ajouter des fonctionnalités**.

7.  Sélectionnez **Suivant** dans la section **Fonctionnalités**.

8.  Si un commutateur virtuel est disponible, sélectionnez-le. Sinon, sélectionnez **Suivant**.

9.  Dans la section **Migration**, sélectionnez **Suivant**.

10. Dans la section **Emplacements par défaut**, sélectionnez **Suivant**.

11. Cochez la case pour redémarrer automatiquement le serveur si nécessaire.

12. Sélectionnez **Installer**.

13. Autorisez le serveur à installer le rôle Hyper-V. Cette opération prend quelques minutes. Le serveur redémarre ensuite automatiquement.

## <a name="step-2-create-the-faulty-vm-on-the-rescue-vms-hyper-v-server"></a>Étape 2 : créer la machine virtuelle défaillante sur le serveur Hyper-V de la machine virtuelle de secours

1.  [Créez un disque d’instantané](troubleshoot-recovery-disks-portal-windows.md#take-a-snapshot-of-the-os-disk) pour le disque du système d’exploitation de la machine virtuelle qui rencontre un problème, puis joignez le disque d’instantané à la machine virtuelle de secours.

2.  Bureau à distance vers la machine virtuelle de secours.

3.  Ouvrez Gestion des disques (diskmgmt.msc). Vérifiez que le disque de la machine virtuelle défaillante est défini sur **Hors connexion**.

4.  Ouvrez Hyper-V Manager : dans **Gestionnaire de serveur**, sélectionnez le **rôle Hyper-V**. Cliquez avec le bouton droit sur le serveur, puis sélectionnez **Gestionnaire Hyper-V**.

5.  Dans le Gestionnaire Hyper-V, cliquez avec le bouton droit sur la machine virtuelle de secours, puis sélectionnez **Nouvelle** > **Machine virtuelle** > **Suivant**.

6.  Tapez un nom pour la machine virtuelle, puis sélectionnez **Suivant**.

7.  Sélectionnez **Génération 1**.

8.  Définissez la mémoire de démarrage à 1 024 Mo ou plus.

9. Le cas échéant, sélectionnez le commutateur réseau Hyper-V qui a été créé. Sinon, passez à la page suivante.

10. Sélectionnez **Attacher un disque dur virtuel ultérieurement**.

    ![image de l’option Attacher un disque dur virtuel ultérieurement](media/troubleshoot-vm-by-use-nested-virtualization/attach-disk-later.png)

11. Sélectionnez **Terminer** quand la machine virtuelle a été créée.

12. Cliquez avec le bouton droit sur la machine virtuelle que vous avez créée, puis sélectionnez **Paramètres**.

13. Sélectionnez **Contrôleur IDE 0**, sélectionnez **Disque dur**, puis cliquez sur **Ajouter**.

    ![image de l’ajout d’un nouveau disque dur](media/troubleshoot-vm-by-use-nested-virtualization/create-new-drive.png)    

14. Dans **Disque dur physique**, sélectionnez le disque de la machine virtuelle défaillante que vous avez attachée à la machine virtuelle Azure. Si vous ne voyez pas de disques répertoriés, vérifiez si le disque est défini sur Hors connexion à l’aide de la fonction Gestion des disques.

    ![image du montage du disque](media/troubleshoot-vm-by-use-nested-virtualization/mount-disk.png)  


15. Sélectionnez **Apply** (Appliquer), puis **OK**.

16. Double-cliquez sur la machine virtuelle, puis démarrez-la.

17. Vous pouvez maintenant utiliser la machine virtuelle en tant que machine virtuelle locale. Vous pouvez effectuer les étapes de dépannage nécessaires.

## <a name="step-3-replace-the-os-disk-used-by-the-faulty-vm"></a>Étape 3 : Remplacer le disque du système d’exploitation utilisé par la machine virtuelle défaillante

1.  Après avoir remis en ligne la machine virtuelle, arrêtez la machine virtuelle dans le Gestionnaire Hyper-V.

2.  [Démonter et détacher le disque du système d’exploitation réparé](troubleshoot-recovery-disks-portal-windows.md#unmount-and-detach-the-original-virtual-hard-disk
).
3.  [Remplacez le disque du système d’exploitation utilisé par la machine virtuelle avec le disque du système d’exploitation réparé](troubleshoot-recovery-disks-portal-windows.md#swap-the-os-disk-for-the-vm
).

## <a name="next-steps"></a>Étapes suivantes

Si vous rencontrez des problèmes pour vous connecter à votre machine virtuelle, consultez [Dépannage d’une connexion Bureau à distance à une machine virtuelle Azure](troubleshoot-rdp-connection.md). Pour résoudre les problèmes liés à l’accès aux applications exécutées sur votre machine virtuelle, consultez [Résoudre les problèmes de connectivité des applications sur une machine virtuelle Windows](troubleshoot-app-connection.md).
