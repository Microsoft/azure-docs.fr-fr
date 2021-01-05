---
title: La machine virtuelle ne répond pas pendant l’attente du service Gestionnaire de session locale
description: Cet article explique comment résoudre les problèmes liés au fait que le système d’exploitation invité est bloqué en attendant que le Gestionnaire de session locale termine le traitement lors du démarrage d’une machine virtuelle Azure.
services: virtual-machines-windows
documentationcenter: ''
author: mibufo
editor: ''
tags: azure-resource-manager
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.topic: troubleshooting
ms.date: 10/22/2020
ms.author: v-mibufo
ms.openlocfilehash: 8af8d7695c48c6ac682109bb38935e98921fa9e4
ms.sourcegitcommit: c157b830430f9937a7fa7a3a6666dcb66caa338b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94681905"
---
# <a name="vm-is-unresponsive-while-waiting-for-the-local-session-manager-service"></a>La machine virtuelle ne répond pas pendant l’attente du service Gestionnaire de session locale

Cet article explique comment résoudre les problèmes liés au fait que le système d’exploitation invité est bloqué en attendant que le Gestionnaire de session locale termine le traitement lors du démarrage d’une machine virtuelle Azure.

## <a name="symptoms"></a>Symptômes

Quand vous utilisez les [diagnostics de démarrage](./boot-diagnostics.md) pour visualiser la capture d’écran de la sortie de la machine virtuelle, vous constatez que cette capture affiche une invite avec le message « Attendez le Gestionnaire de session locale ».

![Capture d’écran montrant le système d’exploitation invité bloqué dans Windows Server 2012 R2, avec le message « Attendez le Gestionnaire de session locale » .](media/vm-unresponsive-wait-local-session-manager/vm-unresponsive-wait-local-session-manager-1.png)

## <a name="cause"></a>Cause

Il peut y avoir plusieurs raisons pour lesquelles une machine virtuelle est bloquée en attente du Gestionnaire de session locale. Si ce problème persiste, vous devez collecter un fichier de vidage pour analyse.

## <a name="solution"></a>Solution

Dans certains cas, il suffit d’attendre que le processus se termine pour résoudre le problème. Si votre machine virtuelle ne répond pas et reste bloquée sur l’écran d’attente pendant plus d’une heure, vous devez collecter une image mémoire, puis contacter le Support Microsoft.

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
> [Résoudre les erreurs de démarrage de machine virtuelle Azure](boot-error-troubleshoot.md)