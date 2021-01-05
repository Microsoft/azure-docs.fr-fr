---
title: Pare-feu du système d’exploitation invité de la machine virtuelle Azure mal configuré | Microsoft Docs
description: Découvrez comment utiliser la console série ou la méthode hors connexion pour diagnostiquer et corriger un pare-feu de système d’exploitation invité mal configuré sur une machine virtuelle Azure distante.
services: virtual-machines-windows
documentationcenter: ''
author: Deland-Han
manager: dcscontentpm
editor: ''
tags: ''
ms.service: virtual-machines
ms.topic: troubleshooting
ms.workload: infrastructure-services
ms.tgt_pltfrm: vm-windows
ms.devlang: azurecli
ms.date: 11/22/2018
ms.author: delhan
ms.openlocfilehash: e6f42bdf462ac5261f77bc05c62e50500345fe37
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "80422530"
---
# <a name="azure-vm-guest-os-firewall-is-misconfigured"></a>Le pare-feu du système d'exploitation invité de la machine virtuelle Azure est mal configuré

Cet article explique comment corriger un pare-feu de système d'exploitation invité mal configuré sur une machine virtuelle Azure.

## <a name="symptoms"></a>Symptômes

1.  L’écran d’accueil de la machine virtuelle indique que la machine virtuelle est entièrement chargée.

2.  Selon la configuration du système d’exploitation invité, un trafic réseau peut ou non atteindre la machine virtuelle.

## <a name="cause"></a>Cause

Une configuration incorrecte du pare-feu du système invité peut bloquer certains ou tous les types de trafic réseau vers la machine virtuelle.

## <a name="solution"></a>Solution

Avant de suivre ces étapes, prenez un instantané du disque système de la machine virtuelle affectée comme sauvegarde. Pour plus d’informations, consultez [Créer un instantané](../windows/snapshot-copy-managed-disk.md).

Pour résoudre ce problème, utilisez la console série ou [réparez la machine virtuelle en mode hors connexion](troubleshoot-rdp-internal-error.md#repair-the-vm-offline) en attachant le disque système de la machine virtuelle à une machine virtuelle de récupération.

## <a name="online-mitigations"></a>Atténuations en ligne

Connectez-vous à la [console série et ouvrez une instance PowerShell](serial-console-windows.md#use-cmd-or-powershell-in-serial-console). Si la console série n’est pas activée sur la machine virtuelle, accédez à la section « Réparer la machine virtuelle en mode hors connexion » de l’article Azure suivant :

 [Erreur interne en cas de connexion à une machine virtuelle Azure avec le Bureau à distance](troubleshoot-rdp-internal-error.md#repair-the-vm-offline)

Les règles suivantes peuvent être modifiées pour activer l’accès à la machine virtuelle (via RDP) ou faciliter la résolution du problème :

*   Bureau à distance (TCP-In) : il s’agit de la règle standard qui fournit l’accès principal à la machine virtuelle en autorisant RDP dans Azure.

*   Gestion à distance de Windows (HTTP-In) : cette règle vous permet de vous connecter à la machine virtuelle à l’aide de PowerShell. Dans Azure, ce type d’accès vous permet d’utiliser l’aspect de script et le dépannage à distance.

*   Partage de fichiers et d’imprimantes (SMB-In) : cette règle permet l’accès au partage réseau comme option de résolution des problèmes.

*   Partage de fichiers et d’imprimantes (Requête Echo - ICMPv4-In) : Cette règle permet d’effectuer un test ping sur la machine virtuelle.

Dans l’instance d’accès à la console série, vous pouvez interroger l’état actuel de la règle de pare-feu.

*   Interrogation avec le nom d’affichage en tant que paramètre :

    ```cmd
    netsh advfirewall firewall show rule dir=in name=all | select-string -pattern "(DisplayName.*<FIREWALL RULE NAME>)" -context 9,4 | more
    ```

*   Interrogation avec le port local utilisé par l’application :

    ```cmd
    netsh advfirewall firewall show rule dir=in name=all | select-string -pattern "(LocalPort.*<APPLICATION PORT>)" -context 9,4 | more
    ```

*   Interrogation avec le port local utilisé par l’application :

    ```cmd
    netsh advfirewall firewall show rule dir=in name=all | select-string -pattern "(LocalIP.*<CUSTOM IP>)" -context 9,4 | more
    ```

*   Si vous voyez que la règle est désactivée, vous pouvez l’activer en exécutant la commande suivante :

    ```cmd
    netsh advfirewall firewall set rule name="<RULE NAME>" new enable=yes
    ```

*   Pour le dépannage, vous pouvez DÉSACTIVER les profils de pare-feu :

    ```cmd
    netsh advfirewall set allprofiles state off
    ```

    Si vous procédez ainsi pour régler correctement le pare-feu, activez-le à nouveau après avoir résolu le problème.

    > [!Note]
    > Vous n’êtes pas obligé de redémarrer la machine virtuelle pour appliquer ce changement.

*   Essayez à nouveau de vous connecter à la machine virtuelle via RDP.

### <a name="offline-mitigations"></a>Atténuations hors connexion

1.  Pour activer ou désactiver des règles de pare-feu, consultez [Activer ou désactiver une règle de pare-feu sur le système d’exploitation invité d’une machine virtuelle Azure](enable-disable-firewall-rule-guest-os.md).

2.  Vérifiez si vous êtes dans le [scénario de blocage du trafic entrant par le pare-feu du système d’exploitation invité](guest-os-firewall-blocking-inbound-traffic.md).

3.  Si vous soupçonnez toujours le pare-feu de bloquer votre accès, consultez [Désactiver le pare-feu du système d’exploitation invité dans une machine virtuelle Azure](disable-guest-os-firewall-windows.md), puis réactivez le pare-feu du système invité à l’aide des règles correctes.

