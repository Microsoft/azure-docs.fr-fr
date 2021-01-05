---
title: Erreur d’arrêt de Windows – Processus du système d’état terminé
description: Cet article décrit les étapes à suivre pour résoudre les problèmes dans lesquels le système d’exploitation rencontre l’erreur d’arrêt 0xC000021A, qui empêche le démarrage d’une machine virtuelle Azure.
services: virtual-machines-windows, azure-resource-manager
documentationcenter: ''
author: v-miegge
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.assetid: 037d0858-4611-4707-bd46-cc8085d011ed
ms.service: virtual-machines-windows
ms.workload: na
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
ms.date: 09/21/2020
ms.author: v-mibufo
ms.openlocfilehash: b13b61aff819271ed1722572f251f9a6d14b17ab
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91976995"
---
# <a name="windows-stop-error---0xc000021a-status-system-process-terminated"></a>Erreur d’arrêt de Windows – 0xC000021A Processus du système d’état terminé

Cet article décrit les étapes à suivre pour résoudre les problèmes dans lesquels le système d’exploitation rencontre l’erreur d’arrêt 0xC000021A, qui empêche le démarrage d’une machine virtuelle Azure.

## <a name="symptom"></a>Symptôme

Lorsque vous utilisez [Diagnostics de démarrage](./boot-diagnostics.md) pour afficher la capture d’écran de la machine virtuelle, la capture d’écran affiche le message indiquant que le système d’exploitation a rencontré une erreur lors du démarrage, à avoir :

**Votre ordinateur a rencontré un problème et doit redémarrer. Nous collectons simplement des informations relatives aux erreurs, vous pourrez ensuite redémarrer l’ordinateur. (##% terminé) Pour en savoir plus, vous pouvez rechercher cette erreur en ligne ultérieurement : 0xC000021a**.

  ![La figure 1 affiche le code d’erreur #0xC000021A avec le message « Votre ordinateur a rencontré un problème et doit redémarrer. Nous collectons simplement des informations relatives aux erreurs, vous pourrez ensuite redémarrer l’ordinateur. »](./media/windows-stop-error-system-process-terminated/1-pc-problem-restart.png)

## <a name="cause"></a>Cause :

L’erreur 0xC000021A indique **STATUS_SYSTEM_PROCESS_TERMINATED**.

Cette erreur se produit quand un processus critique, tel que WinLogon (Winlogon. exe) ou le sous-système d’exécution client/serveur (csrss.exe) échoue. Une fois que le noyau détecte que l’un de ces services s’est arrêté, il déclenche l’erreur **STOP 0xC000021A**. Cette erreur peut avoir plusieurs causes, à savoir :

- Des fichiers système incompatibles ont été installés.
- L’installation d’une mise à jour de Service Pack ou de base de connaissances a échoué.
- Un programme de sauvegarde utilisé pour restaurer un disque dur n’a pas pu restaurer correctement des fichiers qui ont peut-être été utilisés.
- Un programme tiers incompatible a été installé.

## <a name="solution"></a>Solution

### <a name="collect-the-memory-dump-file"></a>Collecter le fichier de l’image mémoire

Pour résoudre ce problème, vous devez analyser le vidage sur incident. Collectez le fichier de vidage de la mémoire pour l’incident et contactez le support technique. Pour collecter le fichier d’image, effectuez les étapes suivantes :

### <a name="attach-the-os-disk-to-a-new-repair-vm"></a>Attacher le disque du système d’exploitation à une machine virtuelle de récupération

1.  Effectuez les étapes 1 à 3 des [commandes de réparation de machine virtuelle](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md) pour préparer une machine virtuelle de réparation.
2.  Utilisez la **Connexion Bureau à distance** pour vous connecter à la machine virtuelle de réparation.

### <a name="locate-the-dump-file-and-submit-a-support-ticket"></a>Rechercher le fichier de l’image mémoire et envoyer un ticket de support

1.  Sur la machine virtuelle de réparation, accédez au dossier Windows sur le disque du système d’exploitation attaché. Si la lettre de lecteur affectée au disque du système d’exploitation attaché est F, accédez à F:\Windows.
2.  Recherchez le fichier memory.dmp, puis [envoyez un ticket de support](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) avec le fichier de l’image mémoire.
3.  Si vous rencontrez des difficultés pour localiser le fichier memory.dmp, vous pouvez utiliser à la place des [appels d’interruption non masquable (NMI) dans la console série](./serial-console-windows.md#use-the-serial-console-for-nmi-calls). Vous pouvez suivre le [guide pour générer un fichier de vidage sur incident complet à l’aide d’appels NMI](/windows/client-management/generate-kernel-or-complete-crash-dump).

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur la résolution des problèmes, voyez la [résolution des erreurs de démarrage courantes](./boot-error-troubleshoot.md) ou [comment résoudre des problèmes d’une machine virtuelle Windows en attachant le disque du système d’exploitation à une machine virtuelle de récupération](./troubleshoot-recovery-disks-windows.md). Vous devez également comprendre [comment utiliser les diagnostics de démarrage pour résoudre des problèmes d’une machine virtuelle](./boot-diagnostics.md).
- Pour plus d’informations sur l’utilisation de Resource Manager, consultez la page [Présentation d’Azure Resource Manager](../../azure-resource-manager/management/overview.md).
- Si vous ne pouvez pas vous connecter à votre machine virtuelle, consultez [Dépannage d’une connexion Bureau à distance à une machine virtuelle Azure](./troubleshoot-rdp-connection.md).