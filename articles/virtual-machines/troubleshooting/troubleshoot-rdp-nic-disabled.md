---
title: Impossible de se connecter à distance aux machines virtuelles Azure car la carte réseau est désactivée | Microsoft Docs
description: Apprendre à résoudre un échec de connexion RDP dû à la désactivation de la carte réseau sur des machines virtuelles Azure | Microsoft Docs
services: virtual-machines-windows
documentationCenter: ''
author: genlin
manager: dcscontentpm
editor: ''
ms.service: virtual-machines-windows
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 11/12/2018
ms.author: genli
ms.openlocfilehash: 0e79efc9de43fc0a3044e9ae1e3959f63bb6e69f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90090252"
---
#  <a name="cannot-remote-desktop-to-a-vm-because-the-network-interface-is-disabled"></a>Impossible de connecter un bureau à distance à une machine virtuelle car l'interface réseau est désactivée

Cet article explique comment résoudre un problème où une connexion Bureau à distance à des machines virtuelles Azure Windows est impossible si l’interface réseau est désactivée.


## <a name="symptoms"></a>Symptômes

Vous ne pouvez pas établir de connexion RDP ni aucun autre type de connexion à d'autres ports d'une machine virtuelle Azure car l'interface réseau de la machine virtuelle est désactivée.

![Capture d’écran montrant une machine virtuelle dans laquelle l’interface réseau est déconnectée.](./media/troubleshoot-rdp-nic-disabled/disconnected.png)

![Capture d’écran montrant une machine virtuelle dans laquelle l’interface réseau est désactivée.](./media/troubleshoot-rdp-nic-disabled/disabled.png)


## <a name="solution"></a>Solution

Avant de suivre cette procédure, faites en sauvegarde en prenant un instantané du disque du système d’exploitation de la machine virtuelle affectée. Pour plus d’informations, consultez [Créer un instantané](../windows/snapshot-copy-managed-disk.md).

Pour activer l'interface de la machine virtuelle, utilisez la console série ou [réinitialisez l'interface réseau](#reset-network-interface) de la machine virtuelle.

### <a name="use-serial-control"></a>Utiliser le contrôle série

1. Connectez-vous à la [console série et ouvrez une instance CMD](./serial-console-windows.md#use-cmd-or-powershell-in-serial-console). Si la console série n’est pas activée sur votre machine virtuelle, consultez [Réinitialiser l'interface réseau](#reset-network-interface).
2. Vérifiez l’état de l’interface réseau :

    ```console
    netsh interface show interface
    ```

    Notez le nom de l’interface réseau désactivée.

3. Activez l’interface réseau :

    ```console
    netsh interface set interface name="interface Name" admin=enabled
    ```

    Par exemple, si l’interface réseau s’appelle « Ethernet 2 », exécutez la commande suivante :

    ```console
    netsh interface set interface name="Ethernet 2" admin=enabled
    ```

4.  Vérifiez à nouveau l’état de l’interface réseau pour vous assurer qu'elle est activée.

    ```console
    netsh interface show interface
    ```

    À ce stade, vous n’êtes pas obligé de redémarrer la machine virtuelle. La machine virtuelle sera de nouveau accessible.

5.  Connectez-vous à la machine virtuelle et vérifiez que le problème est résolu.

## <a name="reset-network-interface"></a>Réinitialiser l’interface réseau

Pour réinitialiser l’interface réseau, remplacez l’adresse IP par une autre qui est disponible dans le sous-réseau. Pour ce faire, utilisez le portail Azure ou Azure PowerShell. Pour plus d’informations, consultez [Réinitialiser l'interface réseau](reset-network-interface.md).
