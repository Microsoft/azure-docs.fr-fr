---
title: Le démarrage d’une machine virtuelle Azure est bloqué au niveau de la mise à jour de Windows | Microsoft Docs
description: Découvrez comment résoudre le blocage du processus de démarrage d’une machine virtuelle Azure au niveau de la mise à jour de Windows.
services: virtual-machines-windows
documentationCenter: ''
author: genlin
manager: dcscontentpm
editor: v-jesits
ms.service: virtual-machines-windows
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 10/09/2018
ms.author: genli
ms.openlocfilehash: fe2d444bbdc09ed5ab0205d4139591e3288e3358
ms.sourcegitcommit: c2dd51aeaec24cd18f2e4e77d268de5bcc89e4a7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94734563"
---
# <a name="azure-vm-startup-is-stuck-at-windows-update"></a>Le démarrage de la machine virtuelle Azure est bloqué au niveau de la mise à jour de Windows

Cet article vous aide à résoudre le blocage de votre machine virtuelle au stade Windows Update durant le démarrage. 


## <a name="symptom"></a>Symptôme

 Une machine virtuelle Windows ne démarre pas. Quand vous examinez les captures d’écran de la fenêtre [Diagnostics de démarrage](../troubleshooting/boot-diagnostics.md), vous constatez que le démarrage est bloqué au niveau du processus de mise à jour. Voici quelques exemples de messages qui peuvent s’afficher :

- Installation de Windows (x %). N’éteignez pas l’ordinateur. Cette opération peut prendre un certain temps. Votre PC va redémarrer à plusieurs reprises.
- Laissez votre PC allumé pendant ce temps. Installation de la mise à jour x sur x... 
- Impossible de terminer les mises à jour. Annulation des modifications. N’éteignez pas l’ordinateur.
- Échec de la configuration des mises à jour de Windows. Rétablissement des modifications. N’éteignez pas l’ordinateur.
- Erreur < code d’erreur > à l’application de la mise à jour x sur x (\Regist...)
- Erreur fatale < code d’erreur > lors de la mise à jour x sur x ($$...)


## <a name="solution"></a>Solution

En fonction du nombre de mises à jour qui sont installées ou annulées, le processus de mise à jour peut prendre un certain temps. Laissez la machine virtuelle dans cet état pendant 8 heures. Si la machine virtuelle est toujours dans cet état après cette période, redémarrez-la à partir du portail Azure et voyez si elle peut démarrer normalement. Si cette étape ne permet pas de résoudre le problème, essayez la solution suivante.

### <a name="remove-the-update-that-causes-the-problem"></a>Supprimer la mise à jour à l’origine du problème

1. Prenez un instantané du disque du système d’exploitation de la machine virtuelle affectée en guise de sauvegarde. Pour plus d’informations, consultez [Créer un instantané](../windows/snapshot-copy-managed-disk.md). 
2. [Attachez le disque du système d’exploitation à une machine virtuelle de récupération](troubleshoot-recovery-disks-portal-windows.md).
3. Une fois que le disque du système d’exploitation est attaché à la machine virtuelle de récupération, exécutez **diskmgmt.msc** pour ouvrir le programme Gestion des disques, puis vérifiez que le disque attaché est **EN LIGNE**. Notez la lettre de lecteur affectée au disque du système d’exploitation attaché contenant le dossier \windows. Si le disque est chiffré, déchiffrez-le avant de passer aux étapes suivantes dans ce document.

4. Ouvrez une instance d’invite de commandes avec élévation de privilèges (Exécuter en tant qu’administrateur). Exécutez la commande suivante pour obtenir la liste des packages de mise à jour présents sur le disque de système d’exploitation attaché :

    ```console
    dism /image:<Attached OS disk>:\ /get-packages > c:\temp\Patch_level.txt
    ```

    Par exemple, si le disque du système d’exploitation attaché est le lecteur F, exécutez la commande suivante :

    ```console
    dism /image:F:\ /get-packages > c:\temp\Patch_level.txt
    ```

5. Ouvrez le fichier C:\temp\Patch_level.txt, puis lisez-le de bas en haut. Recherchez la mise à jour dont l’état est **Installation en attente** ou **Désinstallation en attente**.  Voici un exemple d’état de la mise à jour :

    ```
    Package Identity : Package_for_RollupFix~31bf3856ad364e35~amd64~~17134.345.1.5
    State : Install Pending
    Release Type : Security Update
    Install Time :
    ```
6. Supprimez la mise à jour à l’origine du problème :
    
    ```
    dism /Image:<Attached OS disk>:\ /Remove-Package /PackageName:<PACKAGE NAME TO DELETE>
    ```
    Exemple : 

    ```
    dism /Image:F:\ /Remove-Package /PackageName:Package_for_RollupFix~31bf3856ad364e35~amd64~~17134.345.1.5
    ```

    > [!NOTE] 
    > Selon la taille du package, l’outil DISM prend un certain temps pour traiter la désinstallation. Normalement, le processus est terminé en 16 minutes.

7. [Détachez le disque de système d’exploitation et recréez la machine virtuelle](troubleshoot-recovery-disks-portal-windows.md#unmount-and-detach-the-original-virtual-hard-disk). Ensuite, vérifiez que le problème est résolu.
