---
title: Modifier les paramètres DATA 0 sur un appareil de la gamme StorSimple 8000 | Microsoft Docs
description: Découvrez comment utiliser Windows PowerShell for StorSimple pour reconfigurer l'interface réseau DATA 0 sur votre appareil StorSimple.
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
ms.date: 03/27/2017
ms.author: alkohli
ms.openlocfilehash: 85d7114f419266124d0d23368b24700af025758a
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94961037"
---
# <a name="modify-the-data-0-network-interface-settings-on-your-storsimple-8000-series-device"></a>Modifier les paramètres d’interface réseau DATA 0 sur votre appareils de la gamme StorSimple 8000

## <a name="overview"></a>Vue d’ensemble

Votre appareil Microsoft Azure StorSimple dispose de six interfaces réseau, de DATA 0 à DATA 5. L’interface DATA 0 est toujours configurée via l'interface Windows PowerShell ou la console série, et elle est automatiquement activée pour le cloud. Vous ne pouvez pas configurer l’interface réseau DATA 0 via le portail Azure.

L’interface DATA 0 est d'abord configuré via l’Assistant Installation lors du déploiement initial de l’appareil StorSimple. Si l’appareil est en cours d’exécution, vous devrez peut-être reconfigurer les paramètres DATA 0. Ce didacticiel fournit deux méthodes permettant de modifier les paramètres de réseau DATA 0 via Windows PowerShell for StorSimple.

Après avoir lu ce didacticiel, vous pourrez :

* Modification des paramètres réseau de DATA 0 via l'Assistant Installation
* Modifier les paramètres réseau DATA 0 via l’applet de commande `Set-HcsNetInterface`

## <a name="modify-data-0-network-settings-through-setup-wizard"></a>Modification des paramètres réseau de DATA 0 via l’Assistant Installation
Vous pouvez reconfigurer les paramètres réseau de DATA 0 en vous connectant à l'interface Windows PowerShell de votre appareil StorSimple et en ouvrant une session de l'Assistant Installation. Pour modifier les paramètres de DATA 0, procédez comme suit :

#### <a name="to-modify-data-0-network-settings-through-setup-wizard"></a>Pour modifier les paramètres réseau de DATA 0 via l’Assistant Installation
1. Dans le menu de la console série, sélectionnez l’option 1, **Ouvrir une session avec un accès total**. Lorsque vous y êtes invité, fournissez le **mot de passe administrateur de l’appareil**. Le mot de passe par défaut est `Password1`.
2. À l’invite de commandes, tapez :
   
    `Invoke-HcsSetupWizard`
3. Un assistant d’installation s’affiche pour vous aider à configurer l’interface DATA 0 de votre appareil. Indiquez les nouvelles valeurs de l'adresse IP, de la passerelle et du masque réseau.

> [!NOTE]
> Les adresses IP fixes des contrôleurs doivent être reconfigurées via le panneau **Paramètres réseau** de l’appareil StorSimple dans le portail Azure. Pour plus d'informations, consultez la rubrique [Modification des interfaces réseau](storsimple-8000-modify-device-config.md#modify-network-interfaces).

## <a name="modify-data-0-network-settings-through-set-hcsnetinterface-cmdlet"></a>Modification des paramètres réseau de DATA 0 via l’applet de commande Set-HcsNetInterface
Vous pouvez également reconfigurer l’interface réseau DATA 0 via l’applet de commande `Set-HcsNetInterface` . L’applet de commande est exécutée à partir de l’interface Windows PowerShell de votre appareil StorSimple. Lorsque vous utilisez cette procédure, les adresses IP fixes du contrôleur peuvent également être configurées. Pour modifier les paramètres de DATA 0, procédez comme suit : 

#### <a name="to-modify-data-0-network-settings-through-the-set-hcsnetinterface-cmdlet"></a>Pour modifier les paramètres réseau de DATA 0 via l’applet de commande Set-HcsNetInterface
1. Dans le menu de la console série, sélectionnez l’option 1, **Ouvrir une session avec un accès total**. Lorsque vous y êtes invité, fournissez le mot de passe administrateur de l’appareil. Le mot de passe par défaut est `Password1`.
2. À l’invite de commandes, tapez :
   
    `Set-HCSNetInterface -InterfaceAlias Data0 -IPv4Address <> -IPv4Netmask <> -IPv4Gateway <> -Controller0IPv4Address <> -Controller1IPv4Address <> -IsiScsiEnabled 1 -IsCloudEnabled 1`
   
    Entrez, entre crochets, les valeurs suivantes de DATA 0 :
   
   * Adresse IPv4
   * Passerelle IPv4
   * Masque de sous-réseau IPv4
   * Adresse IPv4 fixe du contrôleur 0
   * Adresse IPv4 fixe du contrôleur 1
     
     Pour plus d’informations sur l’utilisation de cette applet de commande, consultez le [document de référence sur les applets de commande Windows PowerShell pour StorSimple](/previous-versions/windows/powershell-scripting/dn688161(v=wps.630)).

## <a name="next-steps"></a>Étapes suivantes
* Pour configurer des interfaces réseau autres que DATA 0, vous pouvez utiliser la page [Configurer les paramètres réseau du portail Azure](storsimple-8000-modify-device-config.md). 
* Si vous rencontrez des problèmes lors de la configuration de vos interfaces réseau, consultez la page [Résolution des problèmes de déploiement](./storsimple-8000-troubleshoot-deployment.md).