---
title: Résoudre un problème netvsc.sys quand vous vous connectez à distance à une machine virtuelle Windows 10 ou Windows Server 2016 dans Azure | Microsoft Docs
description: Découvrez comment résoudre un problème RDP lié à netsvc.sys quand vous vous connectez à une machine virtuelle Windows 10 ou Windows Server 2016 dans Azure.
services: virtual-machines-windows
documentationCenter: ''
author: genlin
manager: dcscontentpm
editor: v-jesits
ms.service: virtual-machines-windows
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 11/19/2018
ms.author: genli
ms.openlocfilehash: 0f5a414f00ffa50114f090fc19f37b8a85428547
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86508999"
---
# <a name="cannot-connect-remotely-to-a-windows-10-or-windows-server-2016-vm-in-azure-because-of-netvscsys"></a>Impossible de se connecter à distance à une machine virtuelle Windows 10 ou Windows Server 2016 dans Azure en raison de netvsc.sys

Cet article explique comment résoudre un problème dans lequel vous n’avez aucune connexion réseau quand vous vous connectez à une machine virtuelle Windows 10 ou Windows Server 2016 Datacenter sur un hôte Hyper-V Server 2016.

## <a name="symptoms"></a>Symptômes

Vous n’arrivez pas à vous connecter à une machine virtuelle Azure Windows 10 ou Windows Server 2016 à l’aide de RDP (Remote Desktop Protocol). Dans [Diagnostics de démarrage](boot-diagnostics.md), l’écran affiche une croix rouge sur la carte d’interface réseau. Cela indique que la machine virtuelle n’a aucune connectivité alors que le système d’exploitation est entièrement chargé.

En règle générale, ce problème se produit dans Windows [build 14393](https://support.microsoft.com/help/4093120/) et [build 15063](https://support.microsoft.com/help/4015583/). Si la version de votre système d’exploitation est postérieure à ces versions, cet article ne s’applique pas à votre scénario. Pour vérifier la version du système, ouvrez une session CMD dans [la fonctionnalité Console d’accès série](serial-console-windows.md), puis exécutez **Ver**.

## <a name="cause"></a>Cause

Ce problème risque de se produire si la version du fichier système netvsc.sys installé est **10.0.14393.594** ou **10.0.15063.0**. Ces versions de netvsc.sys risquent d’empêcher le système d’interagir avec la plateforme Azure.


## <a name="solution"></a>Solution

Avant de suivre ces étapes, [prenez un instantané du disque système](../windows/snapshot-copy-managed-disk.md) de la machine virtuelle affectée comme sauvegarde. Pour résoudre ce problème, utilisez la console série ou [réparez la machine virtuelle en mode hors connexion](#repair-the-vm-offline) en attachant le disque système de la machine virtuelle à une machine virtuelle de récupération.


### <a name="use-the-serial-console"></a>Utiliser la console série

Connectez-vous à [la console série, ouvrez une instance PowerShell](serial-console-windows.md), puis suivez ces étapes.

> [!NOTE]
> Si la console série n’est pas activée sur votre machine virtuelle, accédez à la section [Réparer la machine virtuelle en mode hors connexion](#repair-the-vm-offline).

1. Exécutez la commande suivante dans une instance PowerShell pour obtenir la version du fichier (**c:\windows\system32\drivers\netvsc.sys**) :

   ```
   (get-childitem "$env:systemroot\system32\drivers\netvsc.sys").VersionInfo.FileVersion
   ```

2. Téléchargez la mise à jour appropriée sur un disque de données nouveau ou existant qui est attaché à une machine virtuelle fonctionnelle de la même région :

   - **10.0.14393.594** : [KB4073562](https://support.microsoft.com/help/4073562)  ou mise à jour ultérieure
   - **10.0.15063.0** : [KB4016240](https://support.microsoft.com/help/4016240) ou mise à jour ultérieure

3. Détachez le disque utilitaire de la machine virtuelle fonctionnelle, puis attachez-le à la machine virtuelle qui ne fonctionne plus.

4. Exécutez la commande suivante pour installer la mise à jour sur la machine virtuelle :

   ```
   dism /ONLINE /add-package /packagepath:<Utility Disk Letter>:\<KB .msu or .cab>
   ```

5. Redémarrez la machine virtuelle.

### <a name="repair-the-vm-offline"></a>Réparer la machine virtuelle en mode hors connexion

1. [Attachez le disque système à une machine virtuelle de récupération](./troubleshoot-recovery-disks-portal-windows.md).

2. Établissez une connexion Bureau à distance avec la machine virtuelle de récupération.

3. Vérifiez que le disque est marqué comme étant **En ligne** dans la console Gestion des disques. Notez la lettre de lecteur affectée au disque système attaché.

4. Créez une copie du dossier **\Windows\System32\config** au cas où vous devriez annuler les changements.

5. Dans la machine virtuelle de secours, démarrez l’Éditeur du Registre (regedit.exe).

6. Sélectionnez la clé **HKEY_LOCAL_MACHINE**, puis **Fichier** > **Charger la ruche** dans le menu.

7. Accédez au fichier SYSTEM dans le dossier **\Windows\System32\config**.

8. Sélectionnez **Ouvrir**, tapez le nom **BROKENSYSTEM**, développez la clé **HKEY_LOCAL_MACHINE**, puis localisez la clé supplémentaire appelée **BROKENSYSTEM** .

9. Accédez à l’emplacement suivant :

   ```
   HKLM\BROKENSYSTEM\ControlSet001\Control\Class\{4d36e972-e325-11ce-bfc1-08002be10318}
   ```

10. Dans chaque sous-clé (par exemple, 0000), examinez la valeur **DriverDesc** qui est affichée comme **Microsoft HYPER-V Network Adapter**.

11. Dans la sous-clé, examinez la valeur **DriverVersion** qui est la version du pilote de la carte réseau de la machine virtuelle.

12. Téléchargez la mise à jour appropriée :

    - **10.0.14393.594** : [KB4073562](https://support.microsoft.com/help/4073562)  ou mise à jour ultérieure
    - **10.0.15063.0** : [KB4016240](https://support.microsoft.com/help/4016240) ou mise à jour ultérieure

13. Attachez le disque système comme disque de données sur une machine virtuelle de secours sur laquelle vous pouvez télécharger la mise à jour.

14. Exécutez la commande suivante pour installer la mise à jour sur la machine virtuelle :

    ```
    dism /image:<OS Disk letter>:\ /add-package /packagepath:c:\temp\<KB .msu or .cab>
    ```

15. Exécutez la commande suivante pour démonter les ruches :

    ```
    reg unload HKLM\BROKENSYSTEM
    ```

16. [Détachez le disque système et recréez la machine virtuelle](./troubleshoot-recovery-disks-portal-windows.md).

## <a name="need-help-contact-support"></a>Vous avez besoin d’aide ? Contacter le support technique

Si vous avez encore besoin d’aide, [contactez le support Azure](https://portal.azure.com/?#blade/Microsoft_Azure_Support/HelpAndSupportBlade) pour résoudre votre problème rapidement.
