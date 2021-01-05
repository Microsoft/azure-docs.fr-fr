---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 10/26/2018
ms.author: alkohli
ms.openlocfilehash: 560c9c177bfa693580979101e5b9343fcff7fe40
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "67177259"
---
### <a name="install-maintenance-mode-updates-via-windows-powershell-for-storsimple"></a>Installer les mises à jour en mode maintenance via Windows PowerShell pour StorSimple

Quand vous appliquez des mises à jour en mode maintenance à l’appareil StorSimple, toutes les demandes d’E/S sont interrompues. Les services tels que la mémoire vive non volatile (NVRAM) ou le service de cluster sont arrêtés. Les deux contrôleurs redémarrent lorsque vous entrez dans ce mode ou que vous le quittez. Lorsque vous quittez ce mode, tous les services reprennent et sont sains. (Cela peut prendre quelques minutes.)

> [!IMPORTANT]
> * Avant de passer en mode maintenance, vérifiez que les deux contrôleurs d’appareil sont sains dans le portail Azure. Si le contrôleur n’est pas sain, [contactez le Support Microsoft](../articles/storsimple/storsimple-8000-contact-microsoft-support.md) pour connaître les étapes suivantes.
> * En mode maintenance, vous devez d’abord mettre à jour un premier contrôleur, puis l’autre contrôleur.

1. Utilisez PuTTY pour vous connecter à la console série. Suivez les instructions détaillées de la section [Utilisation de PuTTY pour se connecter à la console série de l’appareil](../articles/storsimple/storsimple-8000-deployment-walkthrough-u2.md#use-putty-to-connect-to-the-device-serial-console). À l'invite de commandes, appuyez sur **Entrée**. Sélectionnez Option 1, **Se connecter avec accès total**.

2. Pour mettre le contrôleur en mode maintenance, tapez :
    
    `Enter-HcsMaintenanceMode`

    Les deux contrôleurs redémarrent en mode maintenance.

3. Installez vos mises à jour en mode maintenance. Tapez :

    `Start-HcsUpdate`

    Vous êtes invité à confirmer l’opération. Une fois que vous avez confirmé les mises à jour, elles sont installées sur le contrôleur auquel vous êtes connecté. Après l’installation des mises à jour, le contrôleur redémarre.

4. Surveillez l’état des mises à jour. Connectez-vous au contrôleur homologue pendant que le contrôleur actuel se met à jour et n’est pas en mesure de traiter d’autres commandes. Tapez :

    `Get-HcsUpdateStatus`

    Si le `RunInProgress` est `True`, la mise à jour est toujours en cours. Si le `RunInProgress` est `False`, cela veut dire que la mise à jour est terminée.

5. Une fois que les mises à jour du microprogramme de disque sont correctement appliquées et que le contrôleur mis à jour a redémarré, vérifiez la version du microprogramme de disque. Sur le contrôleur mis à jour, tapez :

    `Get-HcsFirmwareVersion`
   
    Les versions attendues du microprogramme de disque sont les suivantes : `XMGJ, XGEG, KZ50, F6C2, VR08, N003, 0107`

6. Quittez le mode maintenance. Tapez la commande suivante pour chaque contrôleur d’appareil :

    `Exit-HcsMaintenanceMode`

    Les contrôleurs redémarrent quand vous quittez le mode maintenance.

7. Revenez au Portail Azure. Il se peut que le portail n’affiche pas les mises à jour installées en mode maintenance pendant 24 heures.