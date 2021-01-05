---
title: La machine virtuelle Azure ne répond pas lors de l’application de la stratégie de sécurité au système
description: Cet article décrit les étapes à suivre pour résoudre les problèmes de blocage de l’écran de chargement si la machine virtuelle ne répond pas lors de l’application d’une stratégie de sécurité au système sur une machine virtuelle Azure.
services: virtual-machines-windows
documentationcenter: ''
author: TobyTu
manager: dcscontentpm
editor: ''
tags: azure-resource-manager
ms.assetid: a97393c3-351d-4324-867d-9329e31b5629
ms.service: virtual-machines-windows
ms.workload: infrastructure-services
ms.tgt_pltfrm: na
ms.topic: troubleshooting
ms.date: 06/15/2020
ms.author: v-mibufo
ms.openlocfilehash: 6b50bffd1a44c0cf53f15650f5ff4d938f45df4d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "84907973"
---
# <a name="azure-vm-is-unresponsive-while-applying-security-policy-to-the-system"></a>La machine virtuelle Azure ne répond pas lors de l’application de la stratégie de sécurité au système

Cet article décrit les étapes à suivre pour résoudre les problèmes d’arrêt du système d’exploitation et de non-réponse lors de l’application d’une stratégie de sécurité sur une machine virtuelle Azure.

## <a name="symptoms"></a>Symptômes

Lorsque vous utilisez [Diagnostics de démarrage](boot-diagnostics.md) pour afficher la capture d'écran de la machine virtuelle, vous constatez que la capture d'écran affiche le système d’exploitation bloqué au moment du démarrage avec le message :

> « Application de la stratégie de sécurité au système ».

:::image type="content" source="media/unresponsive-vm-apply-security-policy/apply-policy.png" alt-text="La capture d’écran de l’écran de démarrage de Windows Server 2012 R2 est bloquée.":::

:::image type="content" source="media/unresponsive-vm-apply-security-policy/apply-policy-2.png" alt-text="La capture d’écran de l’écran de démarrage de Windows Server 2012 R2 est bloquée.":::

## <a name="cause"></a>Cause

Ce problème peut être dû à une multitude de causes potentielles. Vous ne serez pas en mesure de connaître la source tant qu’une analyse de vidage de la mémoire n’aura pas été effectuée.

## <a name="resolution"></a>Résolution

### <a name="process-overview"></a>Vue d’ensemble du processus

1. [Créez une machine virtuelle de réparation et accédez-y](#create-and-access-a-repair-vm)
2. [Activer la console série et la collecte de l’image mémoire](#enable-serial-console-and-memory-dump-collection)
3. [Régénérer la machine virtuelle](#rebuild-the-vm)
4. [Collecter le fichier de vidage de la mémoire](#collect-the-memory-dump-file)

### <a name="create-and-access-a-repair-vm"></a>Créer une machine virtuelle de réparation et y accéder

1. Suivez les [étapes 1 à 3 des commandes de réparation de machine virtuelle](repair-windows-vm-using-azure-virtual-machine-repair-commands.md#repair-process-example) pour préparer une machine virtuelle de réparation.
2. Connectez-vous à la machine virtuelle de réparation à l’aide de la connexion Bureau à distance.

### <a name="enable-serial-console-and-memory-dump-collection"></a>Activer la console série et la collecte de l’image mémoire

Pour activer la collecte d’image mémoire et la console série, exécutez le script suivant :

1. Ouvrez une session d’invite de commandes avec élévation de privilèges (Exécuter en tant qu’administrateur).
2. Répertoriez les données du Store BCD et déterminez l’identificateur du chargeur de démarrage, que vous utiliserez à l’étape suivante.

     1. Pour une machine virtuelle de 1ère génération, entrez la commande suivante et notez l’identificateur indiqué :

        ```console
        bcdedit /store <BOOT PARTITON>:\boot\bcd /enum
        ```

        Dans la commande, remplacez \<BOOT PARTITON> par la lettre de la partition du disque joint qui contient le dossier de démarrage.

        :::image type="content" source="media/unresponsive-vm-apply-security-policy/store-data.png" alt-text="La capture d’écran de l’écran de démarrage de Windows Server 2012 R2 est bloquée." /v NMICrashDump /t REG_DWORD /d 1 /f
        ```

        Décharger le disque de système d’exploitation corrompu :

        ```console
        REG UNLOAD HKLM\BROKENSYSTEM
        ```

### <a name="rebuild-the-vm"></a>Régénérez la machine virtuelle.

Suivez l’[étape 5 des commandes de réparation de machine virtuelle](repair-windows-vm-using-azure-virtual-machine-repair-commands.md#repair-process-example) pour réassembler la machine virtuelle.

### <a name="collect-the-memory-dump-file"></a>Collecter le fichier de l’image mémoire

Pour résoudre ce problème, vous devez d’abord collecter le fichier de l’image mémoire du plantage et contacter le support technique muni de ce fichier d’image mémoire. Pour collecter le fichier d’image, effectuez les étapes suivantes :

1. Joindre le disque du système d’exploitation à une machine virtuelle de réparation :

    - Suivez les [étapes 1 à 3 des commandes de réparation de machine virtuelle](repair-windows-vm-using-azure-virtual-machine-repair-commands.md#repair-process-example) pour préparer une nouvelle machine virtuelle de réparation.
    - Connectez-vous à la machine virtuelle de réparation à l’aide de la connexion Bureau à distance.

2. Localisez le fichier de vidage sur incident et soumettez un ticket de support :

    - Sur la machine virtuelle de réparation, accédez au dossier Windows sur le disque du système d’exploitation attaché. Si la lettre de lecteur qui est affectée au disque du système d’exploitation attaché est `F`, vous devez accéder à `F:\Windows`.
    - Recherchez le fichier memory.dmp, puis [envoyez un ticket de support](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) avec le fichier de l’image mémoire.
    - Si vous rencontrez des difficultés pour localiser le fichier memory.dmp, vous pouvez utiliser à la place des [appels d’interruption non masquable (NMI) dans la console série](serial-console-windows.md#use-the-serial-console-for-nmi-calls). Vous pouvez suivre le guide pour [générer un fichier de vidage sur incident complet à l’aide d’appels NMI](/windows/client-management/generate-kernel-or-complete-crash-dump).

## <a name="next-steps"></a>Étapes suivantes

Si vous rencontrez des problèmes lorsque vous appliquez la stratégie utilisateurs et groupes locaux, consultez [La machine virtuelle ne répond pas lors de l’application de la stratégie utilisateurs et groupes locaux de la stratégie de groupe](unresponsive-vm-apply-group-policy.md)