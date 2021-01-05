---
title: Erreur d’arrêt de Windows – 0x0000007E Exception de thread système non gérée
description: Cet article décrit les étapes à suivre lorsque le système d’exploitation invité rencontre un problème et souhaite redémarrer votre machine virtuelle Azure. Le message indique qu’une exception de thread système n’a pas été gérée.
services: virtual-machines-windows
documentationcenter: ''
author: mibufo
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.topic: troubleshooting
ms.date: 11/04/2020
ms.author: v-mibufo
ms.openlocfilehash: 1ce594d9e3ffddf781c61717ae4534f0c7bd40f8
ms.sourcegitcommit: c157b830430f9937a7fa7a3a6666dcb66caa338b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94681888"
---
# <a name="windows-stop-error---0x0000007e-system-thread-exception-not-handled"></a>Erreur d’arrêt de Windows – 0x0000007E Exception de thread système non gérée

Cet article décrit les étapes à suivre lorsque le système d’exploitation invité rencontre un problème et tente de redémarrer votre machine virtuelle Azure. Le message d’erreur indique qu’une exception de thread système n’a pas été gérée.

## <a name="symptoms"></a>Symptômes

Lorsque vous utilisez les [diagnostics de démarrage](./boot-diagnostics.md) pour afficher une capture d’écran de la machine virtuelle, vous constatez que Windows a besoin de redémarrer avec le code d’arrêt « EXCEPTION DE THREAD SYSTÈME NON GÉRÉE » ou le code d’erreur « 0x0000007E ».

![Capture d’écran montrant un blocage de Windows lors du démarrage, avec le message « Votre PC a rencontré un problème et doit redémarrer. Nous allons le redémarrer pour vous. » et un code d’arrêt « EXCEPTION DE THREAD SYSTÈME NON GÉRÉE ».](media/windows-stop-error-system-thread-exception-not-handled/windows-stop-error-system-thread-exception-not-handled-1.png)

## <a name="cause"></a>Cause

Il n’est pas possible de déterminer la cause tant qu’un fichier d’image mémoire n’a pas été analysé. Continuez pour collecter le fichier d’image mémoire.

## <a name="solution"></a>Solution

Pour résoudre ce problème, vous devez d’abord recueillir le fichier de vidage de la mémoire de l’incident, puis l’envoyer au support Microsoft. Pour collecter le fichier de vidage, suivez les instructions indiquées dans les deux sections ci-dessous.

### <a name="attach-the-os-disk-to-a-new-repair-vm"></a>Joindre le disque du système d’exploitation à une machine virtuelle de réparation

1. Pour préparer une machine virtuelle de réparation, suivez les étapes 1 à 3 des [commandes de réparation de machine virtuelle](./repair-windows-vm-using-azure-virtual-machine-repair-commands.md).
1. Connectez-vous à la machine virtuelle de réparation à l’aide de la connexion Bureau à distance.

### <a name="locate-the-dump-file-and-submit-a-support-ticket"></a>Rechercher le fichier de l’image mémoire et envoyer un ticket de support

1. Sur la machine virtuelle de réparation, accédez au dossier Windows sur le disque du système d'exploitation attaché. Par exemple, si la lettre de lecteur affectée au disque du système d’exploitation attaché est *F*, accédez à `F:\Windows`.
1. Recherchez le fichier *memory.dmp*, puis [envoyez un ticket de support](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) en joignant le fichier de l’image mémoire.
1. Si vous ne parvenez pas à localiser le fichier *memory.dmp*, suivez le guide pour [générer un fichier d’image mémoire sur incident à l’aide d’appels d’interruption non masquable (NMI)](/windows/client-management/generate-kernel-or-complete-crash-dump).

Pour plus d’informations sur les appels NMI, consultez le guide de l’utilisateur relatif aux [appels NMI dans une console série Azure](./serial-console-windows.md#use-the-serial-console-for-nmi-calls).

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Résoudre les erreurs de démarrage de machine virtuelle Azure](./boot-error-troubleshoot.md)
