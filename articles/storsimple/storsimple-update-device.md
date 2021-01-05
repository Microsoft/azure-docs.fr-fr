---
title: Mettre à jour votre appareil StorSimple | Microsoft Docs
description: Explique comment utiliser la fonctionnalité de mise à jour de StorSimple pour installer les mises à jour et les correctifs ordinaires et en mode Maintenance.
services: storsimple
documentationcenter: NA
author: alkohli
manager: carmonm
editor: ''
ms.assetid: 786059f5-2a38-4105-941d-0860ce4ac515
ms.service: storsimple
ms.devlang: NA
ms.topic: article
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 01/23/2018
ms.author: alkohli
ms.openlocfilehash: faf06775b78c5a7c90cea000ac0a1eb768107ef4
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94956838"
---
# <a name="update-your-storsimple-8000-series-device"></a>Mettre à jour votre appareil StorSimple 8000 Series
> [!NOTE]
> Le portail Azure Classic pour StorSimple est déconseillé. Vos instances de StorSimple Device Manager seront automatiquement déplacées vers le nouveau portail Azure à la date de désapprobation planifiée. Vous serez prévenu de ce déplacement par un e-mail et une notification du portail. Ce document sera également bientôt retiré. Si vous avez des questions concernant le déplacement, consultez [FAQ : Déplacement vers le portail Azure](./index.yml).

## <a name="overview"></a>Vue d’ensemble
Les fonctionnalités de mise à jour StorSimple permettent de maintenir facilement à jour votre appareil StorSimple. Selon le type de mise à jour, vous pouvez appliquer des mises à jour à l’appareil par le biais du portail Azure Classic ou de l’interface Windows PowerShell. Ce didacticiel décrit les différents types de mise à jour et leur installation.

Vous pouvez appliquer deux types de mises à jour d’appareil : 

* Mises à jour ordinaires (ou en mode Normal)
* Mises à jour en mode Maintenance

Vous pouvez installer des mises à jour périodiques au moyen du portail Azure Classic ou de Windows PowerShell. Toutefois, vous devez utiliser Windows PowerShell pour installer les mises à jour en mode Maintenance. 

Chaque type de mise à jour est décrit séparément, ci-dessous.

### <a name="regular-updates"></a>Mises à jour ordinaires
Les mises à jour ordinaires s’effectuent sans interruption du service et peuvent être installées lorsque l’appareil est en mode Normal. Ces mises à jour sont appliquées à chaque contrôleur d’appareil via le site web Microsoft Update. 

> [!IMPORTANT]
> Un basculement de contrôleur peut se produire pendant le processus de mise à jour. Mais cela n’aura aucune incidence sur la disponibilité ou le fonctionnement du système.
> 
> 

* Pour plus de détails sur l’installation de mises à jour périodiques par le biais du Portail Azure Classic, consultez [Installation de mises à jour périodiques via le Portail Azure Classic](#install-regular-updates-via-the-azure-classic-portal).
* Vous pouvez également installer les mises à jour ordinaires via Windows PowerShell pour StorSimple Pour des informations détaillées, consultez la page [Installer les mises à jour ordinaires via Windows PowerShell pour StorSimple](#install-regular-updates-via-windows-powershell-for-storsimple).

### <a name="maintenance-mode-updates"></a>Mises à jour en mode Maintenance
Les mises à jour en mode Maintenance provoquent une interruption du service. Il s’agit, par exemple, des mises à niveau du microprogramme du disque. Ces mises à jour nécessitent que l’appareil soit placé en mode Maintenance. Pour plus d’informations, consultez l’[Étape 2 : Passage en mode Maintenance](#step2). Vous ne pouvez pas utiliser le portail Azure Classic pour installer des mises à jour en mode Maintenance. Vous devez utiliser Windows PowerShell pour StorSimple. 

Pour plus d’informations sur l’installation des mises à jour en mode Maintenance, voir la section [Installer les mises à jour en mode Maintenance via Windows PowerShell pour StorSimple](#install-maintenance-mode-updates-via-windows-powershell-for-storsimple).

> [!IMPORTANT]
> Les mises à jour en mode Maintenance doivent être appliquées séparément à chaque contrôleur. 
> 
> 

## <a name="install-regular-updates-via-the-azure-classic-portal"></a>Installer des mises à jour périodiques via le portail Azure Classic
Vous pouvez utiliser le portail Azure Classic pour appliquer des mises à jour à votre appareil StorSimple.

[!INCLUDE [storsimple-install-updates-manually](../../includes/storsimple-install-updates-manually.md)]

## <a name="install-regular-updates-via-windows-powershell-for-storsimple"></a>Installer les mises à jour ordinaires via Windows PowerShell pour StorSimple
Sinon, vous pouvez utiliser Windows PowerShell pour StorSimple pour appliquer les mises à jour ordinaires (mode Normal).

> [!IMPORTANT]
> Bien que vous puissiez installer des mises à jour périodiques à l’aide de Windows PowerShell pour StorSimple, nous vous recommandons vivement de les installer au moyen du portail Azure Classic. À partir de la mise à jour 1, des vérifications préalables seront effectuées avant l’installation de mises à jour à partir du portail. Ces vérifications préalables permettent de prévenir les échecs et de garantir une expérience avec moins de problèmes. 
> 
> 

[!INCLUDE [storsimple-install-regular-updates-powershell](../../includes/storsimple-install-regular-updates-powershell.md)]

## <a name="install-maintenance-mode-updates-via-windows-powershell-for-storsimple"></a>Installer les mises à jour en mode Maintenance via Windows PowerShell pour StorSimple
Vous utilisez Windows PowerShell pour StorSimple pour appliquer les mises à jour en mode Maintenance à votre appareil StorSimple. Dans ce mode, toutes les demandes d’E/S sont suspendues. Les services tels que la mémoire vive non volatile (NVRAM) ou le service de cluster sont également arrêtés. Les deux contrôleurs sont redémarrés lorsque vous entrez dans ce mode ou que vous le quittez. Lorsque vous quittez ce mode, tous les services reprennent et doivent être sains. (Cela peut prendre quelques minutes.)

Si vous devez appliquer des mises à jour en mode Maintenance, vous recevez une alerte par le biais du portail Azure Classic, vous informant que vous avez des mises à jour à installer. Cette alerte inclut des instructions pour installer les mises à jour à l’aide de Windows PowerShell pour StorSimple. Après avoir mis à jour votre appareil, utilisez la même procédure pour faire passer l’appareil en mode Normal. Pour obtenir des instructions détaillées, consultez l’ [Étape 4 : Sortie du mode Maintenance](#step4).

> [!IMPORTANT]
> * Avant de passer en mode Maintenance, vérifiez que les deux contrôleurs d’appareil sont intègres en vérifiant le **Statut matériel** sur la page **Maintenance** du portail Azure Classic. Si le contrôleur n’est pas sain, contactez le support technique Microsoft pour connaître les étapes suivantes. Pour plus d’informations, accédez à Contacter le support technique Microsoft. 
> * En mode Maintenance, vous devez appliquer la mise à jour d’abord à un contrôleur, puis à l’autre contrôleur.
> 
> 

### <a name="step-1-connect-to-the-serial-console"></a>Étape 1 : Connexion à la console série <a name="step1"></a>
Tout d’abord, utilisez une application telle que PuTTY pour accéder à la console série. La procédure suivante explique comment utiliser PuTTY pour se connecter à la console série.

[!INCLUDE [storsimple-use-putty](../../includes/storsimple-use-putty.md)]

### <a name="step-2-enter-maintenance-mode"></a>Étape 2 : Passage en mode Maintenance <a name="step2"></a>
Une fois que vous êtes connecté à la console, déterminez s’il existe des mises à jour à installer, puis passez en mode Maintenance pour les installer.

[!INCLUDE [storsimple-enter-maintenance-mode](../../includes/storsimple-enter-maintenance-mode.md)]

### <a name="step-3-install-your-updates"></a>Étape 3 : Installation des mises à jour <a name="step3"></a>
Ensuite, installez les mises à jour.

[!INCLUDE [storsimple-install-maintenance-mode-updates](../../includes/storsimple-install-maintenance-mode-updates.md)]

### <a name="step-4-exit-maintenance-mode"></a>Étape 4 : Sortie du mode Maintenance <a name="step4"></a>
Pour finir, quittez le mode Maintenance.

[!INCLUDE [storsimple-exit-maintenance-mode](../../includes/storsimple-exit-maintenance-mode.md)]

## <a name="install-hotfixes-via-windows-powershell-for-storsimple"></a>Installer les correctifs logiciels via Windows PowerShell pour StorSimple
Contrairement aux mises à jour pour Microsoft Azure StorSimple, les correctifs logiciels sont installés à partir d’un dossier partagé. Comme pour les mises à jour, il existe deux types de correctifs : 

* Correctifs logiciels ordinaires 
* Correctifs logiciels en mode Maintenance  

Les procédures suivantes expliquent comment utiliser Windows PowerShell pour StorSimple pour installer les correctifs logiciels ordinaires et en mode Maintenance.

[!INCLUDE [storsimple-install-regular-hotfixes](../../includes/storsimple-install-regular-hotfixes.md)]

[!INCLUDE [storsimple-install-maintenance-mode-hotfixes](../../includes/storsimple-install-maintenance-mode-hotfixes.md)]

## <a name="what-happens-to-updates-if-you-perform-a-factory-reset-of-the-device"></a>Que se passe-t-il pour les mises à jour si vous effectuez une réinitialisation de l’appareil aux paramètres d’usine ?
Si un appareil est réinitialisé aux paramètres d’usine, toutes les mises à jour sont perdues. Une fois que l’appareil réinitialisé avec les paramètres par défaut est inscrit et configuré, vous devez installer manuellement les mises à jour par le biais du portail Azure Classic et/ou de Windows PowerShell pour StorSimple. Pour plus d’informations sur la réinitialisation des paramètres d’usine, consultez [Rétablissement des paramètres par défaut de l’appareil](storsimple-8000-manage-device-controller.md#reset-the-device-to-factory-default-settings).

## <a name="next-steps"></a>Étapes suivantes
* En savoir plus sur [l’utilisation de Windows PowerShell pour StorSimple pour administrer votre appareil StorSimple](./storsimple-8000-windows-powershell-administration.md).
* En savoir plus sur [l’utilisation du service StorSimple Manager pour gérer votre appareil StorSimple](./storsimple-8000-manager-service-administration.md).