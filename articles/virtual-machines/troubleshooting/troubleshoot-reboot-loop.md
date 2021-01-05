---
title: Boucle de redémarrage Windows sur une machine virtuelle Azure | Microsoft Docs
description: Découvrir comment résoudre une boucle de redémarrage Windows | Microsoft Docs
services: virtual-machines-windows
documentationCenter: ''
author: genlin
manager: dcscontentpm
editor: ''
ms.service: virtual-machines-windows
ms.topic: troubleshooting
ms.tgt_pltfrm: vm-windows
ms.workload: infrastructure
ms.date: 10/15/2018
ms.author: genli
ms.openlocfilehash: 4c336fe9a65d7bcc44790a4bfb02bed44f028733
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86500920"
---
# <a name="windows-reboot-loop-on-an-azure-vm"></a>Boucle de redémarrage Windows sur une machine virtuelle Azure
Cet article décrit la boucle de redémarrage que vous pouvez rencontrer sur une machine virtuelle Windows dans Microsoft Azure.

## <a name="symptom"></a>Symptôme

Quand vous utilisez les [diagnostics de démarrage](./boot-diagnostics.md) pour obtenir les captures d’écran d’une machine virtuelle, vous constatez que la machine virtuelle démarre. Toutefois, le processus de démarrage est interrompu et recommence.

![Écran de démarrage 1](./media/troubleshoot-reboot-loop/start-screen-1.png)

## <a name="cause"></a>Cause

La boucle de redémarrage se produit pour les raisons suivantes :

### <a name="cause-1"></a>Cause 1

Un service tiers marqué comme étant critique ne peut pas être démarré. Ce problème entraîne le redémarrage du système d’exploitation.

### <a name="cause-2"></a>Cause 2

Des changements ont été apportés au système d’exploitation. En général, ceux-ci sont liés à l’installation d’une mise à jour, à l’installation d’une application ou à l’application d’une nouvelle stratégie. Vous devrez peut-être consulter les journaux d’activité suivants pour obtenir plus d’informations :

- Journaux des événements
- CBS.logWindows
- Update.log

### <a name="cause-3"></a>Cause 3

Le système de fichiers a peut-être été endommagé. Toutefois, il est difficile de diagnostiquer et d’identifier la cause de l’endommagement du système d’exploitation.

## <a name="solution"></a>Solution

Pour résoudre ce problème, [sauvegardez le disque du système d’exploitation](../windows/snapshot-copy-managed-disk.md) et [attachez-le à une machine virtuelle de secours](./troubleshoot-recovery-disks-portal-windows.md), puis suivez les étapes de la solution correspondant à votre problème ou essayez les solutions une par une.

### <a name="solution-for-cause-1"></a>Solution pour la cause 1

1. Une fois le disque du système d’exploitation attaché à une machine virtuelle opérationnelle, vérifiez que le disque est marqué comme étant **en ligne** dans la console Gestion des disques, puis notez la lettre de lecteur de la partition dans laquelle se trouve le dossier **\Windows**.

2. Si le disque est **hors connexion**, faites-le passer **en ligne**.

3. Créez une copie du dossier **\Windows\System32\config** au cas où vous souhaiteriez annuler les changements.

4. Sur la machine virtuelle de secours, ouvrez l’Éditeur du Registre Windows (regedit).

5. Sélectionnez la clé **HKEY_LOCAL_MACHINE**, puis **Fichier** > **Charger la ruche** à partir du menu.

6. Accédez au fichier SYSTEM dans le dossier **\Windows\System32\config**.

7. Sélectionnez **Ouvrir**, tapez le nom **BROKENSYSTEM**, puis développez la clé **HKEY_LOCAL_MACHINE**. Vous devriez voir une clé supplémentaire appelée **BROKENSYSTEM** .

8. Déterminez à partir de quel ControlSet l’ordinateur démarre. Son numéro de clé apparaît dans la clé de Registre suivante.

    `HKEY_LOCAL_MACHINE\BROKENSYSTEM\Select\Current`

9. Déterminez la criticité du service de l’agent de la machine virtuelle à l’aide de la clé de Registre suivante.

    `HKEY_LOCAL_MACHINE\BROKENSYSTEM\ControlSet00x\Services\RDAgent\ErrorControl`

10. Si la clé de Registre n’a pas la valeur **2**, accédez à l’atténuation suivante.

11. Si la clé de Registre a la valeur **2**, remplacez la valeur **2** par **1**.

12. Si toutes les clés suivantes existent et qu’elles ont la valeur **2** ou **3**, remplacez ces valeurs par **1** comme ceci :

    - `HKEY_LOCAL_MACHINE\BROKENSYSTEM\ControlSet00x\Services\AzureWLBackupCoordinatorSvc\ErrorControl`
    - `HKEY_LOCAL_MACHINE\BROKENSYSTEM\ControlSet00x\Services\AzureWLBackupInquirySvc\ErrorControl`
    - `HKEY_LOCAL_MACHINE\BROKENSYSTEM\ControlSet00x\Services\AzureWLBackupPluginSvc\ErrorControl`

13. Sélectionnez la clé **BROKENSYSTEM**, puis **Fichier** > **Décharger Hive** à partir du menu.

14. Détachez le disque du système d’exploitation de la machine virtuelle de dépannage.

15. Retirez le disque de la machine virtuelle de dépannage et patientez environ 2 minutes le temps qu’Azure libère ce disque.

16. [Créez une machine virtuelle à partir du disque du système d’exploitation](../windows/create-vm-specialized.md).

17. Si le problème est résolu, il se peut que vous deviez réinstaller [RDAgent](/archive/blogs/mast/install-the-vm-agent-on-an-existing-azure-vm) (WaAppAgent.exe).

### <a name="solution-for-cause-2"></a>Solution pour la cause 2

Restaurez la machine virtuelle à la dernière configuration valide connue. Pour cela, consultez [Guide pratique pour démarrer une machine virtuelle Windows Azure avec la dernière configuration valide connue](https://support.microsoft.com/help/4016731/).

### <a name="solution-for-cause-3"></a>Solution pour la cause 3
>[!NOTE]
>La procédure suivante ne doit être utilisée qu’en dernier recours. Bien que la restauration à partir de regback puisse rétablir l’accès à l’ordinateur, le système d’exploitation n’est pas considéré comme stable en raison de la perte de données dans le Registre entre l’horodatage de la ruche et le jour actuel. Vous devez générer une nouvelle machine virtuelle et planifier la migration des données.

1. Une fois le disque attaché à une machine virtuelle de dépannage, vérifiez que le disque est marqué comme étant **en ligne** dans la console Gestion des disques.

2. Créez une copie du dossier **\Windows\System32\config** au cas où vous souhaiteriez annuler les changements.

3. Copiez les fichiers dans le dossier **\Windows\System32\config\regback** et remplacez ceux du dossier **\Windows\System32\config**.

4. Retirez le disque de la machine virtuelle de dépannage et patientez environ 2 minutes le temps qu’Azure libère ce disque.

5. [Créez une machine virtuelle à partir du disque du système d’exploitation](../windows/create-vm-specialized.md).
