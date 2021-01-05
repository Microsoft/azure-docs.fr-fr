---
author: alkohli
ms.service: storsimple
ms.topic: include
ms.date: 10/26/2018
ms.author: alkohli
ms.openlocfilehash: 867cdc97ff91d5932230b733dee4d7660d499c39
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96025729"
---
#### <a name="to-complete-the-minimum-storsimple-device-setup"></a>Pour mener à bien la configuration minimale d’un appareil StorSimple

   > [!NOTE]
   > Vous ne pouvez pas modifier le nom de l’appareil une fois la configuration de l’appareil terminée.
   
1. Sélectionnez votre appareil et cliquez dessus dans la liste tubulaire des appareils du panneau **Appareils**. L’appareil se trouve à l’état **Prêt pour la configuration**. Le panneau **Configurer l’appareil** s’ouvre.

     ![Interfaces réseau de la configuration minimale d’un appareil StorSimple](./media/storsimple-8000-complete-minimum-device-setup-u2/step4minconfig1.png)

2. Dans le panneau **Configurer l’appareil** :
   
   1. Entrez un **Nom convivial** pour votre appareil. Le nom de l’appareil par défaut reflète des informations telles que le modèle de l’appareil et son numéro de série. Vous pouvez attribuer un nom convivial allant jusqu'à 64 caractères pour gérer l’appareil.
   2. Définissez le **fuseau horaire** en fonction de l'emplacement géographique de l’appareil déployé. Votre appareil utilise ce fuseau horaire pour toutes les opérations planifiées.
   3. Sous les **paramètres DATA 0** :

       1. Votre interface réseau DATA 0 s’affiche comme étant activée avec les paramètres réseau (IP, sous-réseau, passerelle) configurés via l’Assistant d’installation. DATA 0 est également automatiquement activé pour le cloud, ainsi que pour l’iSCSI.

       2. Entrez les adresses IP fixes pour les contrôleurs 0 et 1. **Les adresses IP fixes des contrôleurs doivent être disponibles au sein du sous-réseau accessible par l'adresse IP de l’appareil.** Si l’interface du réseau DATA 0 est configurée pour le protocole IPv4, les adresses IP fixes doivent être fournies au format IPv4. Si vous avez fourni un préfixe pour la configuration du protocole IPv6, ces champs sont automatiquement remplis par des adresses IP fixes.

            ![Interfaces réseau de la configuration minimale d’un appareil StorSimple 2](./media/storsimple-8000-complete-minimum-device-setup-u2/step4minconfig2.png)

            Les adresses IP fixes du contrôleur servent à traiter les mises à jour de l’appareil et à nettoyer la mémoire. Par conséquent, les adresses IP fixes doivent être routables et être en mesure de se connecter à Internet. Vous pouvez vérifier que vos adresses IP de contrôleur fixe sont routables à l’aide de l’applet de commande [Test-HcsmConnection][Test]. Dans l’exemple suivant, les adresses IP de contrôleur fixe sont routées à Internet et peuvent accéder aux serveurs Microsoft Update.

            ![Test-HcsmConnection indiquant les adresses IP routables](./media/storsimple-8000-complete-minimum-device-setup-u2/step4minconfig3.png)

1. Cliquez sur **OK**. La configuration de l’appareil démarre. Lorsque la configuration de l’appareil est terminée, vous êtes averti. L’état de l’appareil devient **En ligne** dans le panneau **Appareils**.

    ![Interfaces réseau de la configuration minimale d’un appareil StorSimple 3](./media/storsimple-8000-complete-minimum-device-setup-u2/step4minconfig4.png)

<!--Link reference-->
[Test]: /previous-versions/windows/powershell-scripting/dn715782(v=wps.630)