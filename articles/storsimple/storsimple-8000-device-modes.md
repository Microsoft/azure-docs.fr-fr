---
title: Changer le mode de l’appareil StorSimple | Microsoft Docs
description: Décrit les modes de l’appareil StorSimple et explique comment utiliser Windows PowerShell for StorSimple pour changer le mode de l’appareil.
services: storsimple
documentationcenter: ''
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: na
ms.topic: how-to
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 06/29/2017
ms.author: alkohli
ms.openlocfilehash: c7b0ea489c1d70ab86d677aad666ea6728fa76b4
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96017081"
---
# <a name="change-the-device-mode-on-your-storsimple-device"></a>Changement du mode de votre appareil StorSimple

Cet article fournit une brève description des modes avec lesquels votre appareil StorSimple peut fonctionner. Votre appareil StorSimple peut fonctionner dans trois modes : Normal, Maintenance et Récupération.

À la fin de cet article, vous :

* Description des modes de l'appareil StorSimple
* Comment déterminer le mode dans lequel se trouve l'appareil StorSimple
* Comment passer du mode Normal au mode Maintenance, et *inversement*

Les tâches de gestion ci-dessus peuvent uniquement être effectuées via l’interface Windows PowerShell de votre appareil StorSimple.

## <a name="about-storsimple-device-modes"></a>À propos des modes de l’appareil StorSimple

Votre appareil StorSimple peut fonctionner en mode Normal, Maintenance ou Récupération. Chacun de ces modes est brièvement décrit ci-dessous.

### <a name="normal-mode"></a>Mode Normal

Il s’agit du mode de fonctionnement normal pour un appareil StorSimple entièrement configuré. Par défaut, votre appareil doit être en mode Normal.

### <a name="maintenance-mode"></a>Mode Maintenance

Parfois, l’appareil StorSimple doit être mis en mode Maintenance. Ce mode vous permet d’assurer la maintenance de l’appareil et d’installer des mises à jour perturbatrices, telles que celles du microprogramme de disque.

Vous ne pouvez mettre le système en mode Maintenance qu’à l’aide de Windows PowerShell pour StorSimple. Dans ce mode, toutes les demandes d’E/S sont suspendues. Les services tels que la mémoire vive non volatile (NVRAM) ou le service de cluster sont également arrêtés. Les deux contrôleurs sont redémarrés lorsque vous entrez dans ce mode ou que vous le quittez. Lorsque vous quittez le mode Maintenance, tous les services reprennent et doivent être sains. Cela peut prendre quelques minutes.

> [!NOTE]
> **Le mode Maintenance est uniquement pris en charge sur un appareil en état de fonctionnement. Il n’est pas pris en charge sur un appareil dont l’un ou les deux contrôleurs ne fonctionnent pas.**


### <a name="recovery-mode"></a>Mode Récupération

Le mode Récupération peut être décrit comme un « mode sans échec avec prise en charge du réseau Windows ». Le mode Récupération engage l’équipe de support technique Microsoft et leur permet d’effectuer des diagnostics sur le système. L’objectif principal objectif du mode Récupération consiste à récupérer les journaux d’activité système.

Si votre système passe en mode Récupération, vous devez contacter le support technique Microsoft pour les étapes suivantes. Pour plus d’informations, accédez à [Contacter le support technique Microsoft](storsimple-8000-contact-microsoft-support.md).

> [!NOTE]
> **Vous ne pouvez pas mettre l’appareil en mode Récupération. Si l’appareil est en mauvais état, ce mode tente de le rétablir dans un état permettant au support technique Microsoft de l’examiner.**

## <a name="determine-storsimple-device-mode"></a>Détermination du mode de l’appareil StorSimple

#### <a name="to-determine-the-current-device-mode"></a>Pour déterminer le mode actuel de l’appareil

1. Ouvrez une session sur la console série de l'appareil en suivant les étapes dans [Utilisation de PuTTY pour se connecter à la console série de l'appareil](storsimple-8000-deployment-walkthrough-u2.md#use-putty-to-connect-to-the-device-serial-console).
2. Examinez le message de bannière situé dans le menu de la console série de l’appareil. Ce message indique explicitement si l’appareil est en mode Maintenance ou Récupération. Si le message ne contient pas d’informations spécifiques sur le mode du système, l’appareil est en mode Normal.

## <a name="change-the-storsimple-device-mode"></a>Changer le mode de l'appareil StorSimple

Vous pouvez mettre l’appareil StorSimple en mode Maintenance (à partir du mode Normal) pour effectuer une maintenance ou installer des mises à jour du mode Maintenance. Effectuez les procédures suivantes pour entrer dans le mode Maintenance ou le quitter.

> [!IMPORTANT]
> Avant d’activer le mode Maintenance, vérifiez que les deux contrôleurs d’appareil fonctionnent correctement en accédant à **Paramètres > Hardware health** (Intégrité matérielle) pour votre appareil dans le portail Azure. Si au moins l’un des deux contrôleurs n’est pas sain, contactez le Support Microsoft pour connaître les étapes suivantes. Pour plus d’informations, accédez à [Contacter le support technique Microsoft](storsimple-8000-contact-microsoft-support.md).
 

#### <a name="to-enter-maintenance-mode"></a>Pour passer en mode Maintenance

1. Ouvrez une session sur la console série de l'appareil en suivant les étapes dans [Utilisation de PuTTY pour se connecter à la console série de l'appareil](storsimple-8000-deployment-walkthrough-u2.md#use-putty-to-connect-to-the-device-serial-console).
2. Dans le menu de la console série, sélectionnez l’option 1, **Ouvrir une session avec un accès total**. Lorsque vous y êtes invité, fournissez le **mot de passe administrateur de l’appareil**. Le mot de passe par défaut est : `Password1`.
3. À l'invite de commandes, tapez 
   
    `Enter-HcsMaintenanceMode`
4. Un message d’avertissement s’affiche pour vous indiquer que le mode maintenance va perturber toutes les demandes d’E/S et annuler la connexion au portail Azure. Vous êtes invité à confirmer l’opération. Tapez **O** pour passer en mode Maintenance.
5. Les deux contrôleurs redémarrent. Une fois le redémarrage terminé, la bannière de la console série indique que l’appareil est en mode maintenance. Voici un exemple de sortie obtenue.

```
    ---------------------------------------------------------------
    Microsoft Azure StorSimple Appliance Model 8100
    Name: 8100-SHX0991003G44MT
    Software Version: 6.3.9600.17820
    Copyright (C) 2014 Microsoft Corporation. All rights reserved.
    You are connected to Controller0 - Passive
    ---------------------------------------------------------------

    Controller0>Enter-HcsMaintenanceMode
    Checking device state...

    In maintenance mode, your device will not service IOs and will be disconnected from the Microsoft Azure StorSimple Manager service. Entering maintenance mode will end the current session and reboot both controllers, which takes a few minutes to complete. Are you sure you want to enter maintenance mode?
    [Y] Yes [N] No (Default is "Y"): Y

    <BOTH CONTROLLERS RESTART>

    -----------------------MAINTENANCE MODE------------------------
    Microsoft Azure StorSimple Appliance Model 8100
    Name: 8100-SHX0991003G44MT
    Software Version: 6.3.9600.17820
    Copyright (C) 2014 Microsoft Corporation. All rights reserved.
    You are connected to Controller0 - Passive
    ---------------------------------------------------------------

    Serial Console Menu
    [1] Log in with full access
    [2] Log into peer controller with full access
    [3] Connect with limited access
    [4] Change language
    Please enter your choice>

```

#### <a name="to-exit-maintenance-mode"></a>Pour quitter le mode Maintenance

1. Connectez-vous à la console série de l’appareil. Vérifiez dans le message de bannière, que votre appareil est en mode Maintenance.
2. À l’invite de commandes, tapez :
   
    `Exit-HcsMaintenanceMode`
3. Un message d’avertissement et un message de confirmation s’affichent. Tapez **O** pour quitter le mode Maintenance.
4. Les deux contrôleurs redémarrent. Une fois le redémarrage terminé, la bannière de la console série indique que l’appareil est en mode normal. Voici un exemple de sortie obtenue.

```
    -----------------------MAINTENANCE MODE------------------------
    Microsoft Azure StorSimple Appliance Model 8100
    Name: 8100-SHX0991003G44MT
    Software Version: 6.3.9600.17820
    Copyright (C) 2014 Microsoft Corporation. All rights reserved.
    You are connected to Controller0
    ---------------------------------------------------------------

    Controller0>Exit-HcsMaintenanceMode
    Checking device state...

    Before exiting maintenance mode, ensure that any updates that are required on both controllers have been applied. Failure to install on each controller could result in data corruption. Exiting maintenance mode will end the current session and reboot both controllers, which takes a few minutes to complete. Are you sure you want to exit maintenance mode?
    [Y] Yes [N] No (Default is "Y"): Y

    <BOTH CONTROLLERS RESTART>

    ---------------------------------------------------------------
    Microsoft Azure StorSimple Appliance Model 8100
    Name: 8100-SHX0991003G44MT
    Software Version: 6.3.9600.17820
    Copyright (C) 2014 Microsoft Corporation. All rights reserved.
    You are connected to Controller0 - Active
    ---------------------------------------------------------------

    Serial Console Menu
    [1] Log in with full access
    [2] Log into peer controller with full access
    [3] Connect with limited access
    [4] Change language
    Please enter your choice>
```

## <a name="next-steps"></a>Étapes suivantes

Découvrez comment [appliquer les mises à jour des modes Normal et Maintenance](storsimple-update-device.md) sur votre appareil StorSimple.

