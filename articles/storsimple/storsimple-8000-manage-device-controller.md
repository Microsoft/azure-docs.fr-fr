---
title: Gestion des contrôleurs d’appareil de la gamme StorSimple 8000 | Microsoft Docs
description: Découvrez comment arrêter, redémarrer, éteindre ou réinitialiser vos contrôleurs d’appareil StorSimple.
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
ms.date: 06/19/2017
ms.author: alkohli
ms.openlocfilehash: 090797549cc61aa27945114e5ef8b666226b66e2
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94956991"
---
# <a name="manage-your-storsimple-device-controllers"></a>Gestion de vos contrôleurs d’appareil StorSimple

## <a name="overview"></a>Vue d’ensemble

Ce didacticiel décrit les différentes opérations qui peuvent être effectuées sur vos contrôleurs d’appareil StorSimple. Les contrôleurs de votre appareil StorSimple sont des contrôleurs redondants (homologues) en mode actif/passif. À un moment donné, un seul contrôleur est actif et procède au traitement de toutes les opérations de disque et du réseau. L’autre contrôleur est en mode passif. Si le contrôleur actif est défaillant, le contrôleur passif devient automatiquement actif.

Ce didacticiel inclut des instructions détaillées pour gérer les contrôleurs d’appareil à l’aide de :

* Panneau **Contrôleurs** de votre appareil dans le service StorSimple Device Manager.
* Windows PowerShell pour StorSimple

Nous vous recommandons de gérer les contrôleurs d’appareil via le service StorSimple Device Manager. Si une action ne peut être effectuée qu’à l’aide de Windows PowerShell pour StorSimple, le didacticiel l’indique.

Après avoir lu ce didacticiel, vous pourrez :

* redémarrer ou arrêter un contrôleur d’appareil StorSimple
* arrêter un appareil StorSimple
* rétablir les paramètres d’usine de votre appareil StorSimple

## <a name="restart-or-shut-down-a-single-controller"></a>Redémarrage ou arrêt d’un contrôleur unique
Le redémarrage ou l’arrêt d’un contrôleur n’est pas nécessaire si le système fonctionne normalement. Les opérations d’arrêt d’un contrôleur d’appareil unique ne sont courantes que lorsqu’un composant matériel de l’appareil est défaillant et qu’il doit être remplacé. Le redémarrage d’un contrôleur peut également être nécessaire lorsque les performances sont affectées par une utilisation excessive de la mémoire ou un dysfonctionnement du contrôleur. Vous devrez peut-être également redémarrer un contrôleur qui a été remplacé, si vous souhaitez l’activer et le tester.

Le redémarrage d’un appareil n’est pas sans interruption pour les initiateurs connectés, en supposant que le contrôleur passif est disponible. Si un contrôleur passif est indisponible ou bien désactivé, son redémarrage peut entraîner l’interruption du service.

> [!IMPORTANT]
> * **Un contrôleur en cours d’exécution ne doit jamais être physiquement supprimé, car cela entraînerait une perte de redondance et une augmentation des risques d’interruption.**
> * La procédure suivante s’applique uniquement à l’appareil physique StorSimple. Pour plus d’informations sur le démarrage, l’arrêt et le redémarrage de la StorSimple Cloud Appliance, consultez la rubrique [Utilisation de l’appliance cloud](storsimple-8000-cloud-appliance-u2.md#work-with-the-storsimple-cloud-appliance).

Vous pouvez redémarrer ou arrêter un contrôleur d’appareil donné à l’aide du portail Azure du service StorSimple Device Manager ou de Windows PowerShell pour StorSimple.

Pour gérer vos contrôleurs d’appareil à partir du portail Azure, procédez comme suit.

#### <a name="to-restart-or-shut-down-a-controller-in-azure-portal"></a>Pour redémarrer ou arrêter un contrôleur dans le portail Azure
1. Dans votre service StorSimple Device Manager, accédez à **Appareils**. Sélectionnez votre appareil dans la liste des appareils. 

    ![Choisir un appareil](./media/storsimple-8000-manage-device-controller/manage-controller1.png)

2. Accédez à **Paramètres > Contrôleurs**.
   
    ![Vérifiez que les contrôleurs d’appareil StorSimple sont en bon état de fonctionnement](./media/storsimple-8000-manage-device-controller/manage-controller2.png)
3. Dans le panneau **Contrôleurs**, vérifiez que l’état des deux contrôleurs sur votre appareil est **Sain**. Sélectionnez un contrôleur, cliquez dessus avec le bouton droit, puis sélectionnez **Redémarrer** ou **Arrêter**.

    ![Sélectionner Redémarrer ou Arrêter des contrôleurs d’appareil StorSimple](./media/storsimple-8000-manage-device-controller/manage-controller3.png)

4. Un travail est créé pour redémarrer ou arrêter le contrôleur et des avertissements appropriés vous sont présentés, le cas échéant. Pour surveiller le redémarrage ou l’arrêt, accédez à **Service > Journaux d’activité**, puis filtrez par les paramètres spécifiques à votre service. Si un contrôleur a été arrêté, vous devrez alors appuyer sur le bouton d’alimentation pour activer le contrôleur.

#### <a name="to-restart-or-shut-down-a-controller-in-windows-powershell-for-storsimple"></a>Pour redémarrer ou arrêter un contrôleur dans Windows PowerShell pour StorSimple
Pour arrêter ou redémarrer un contrôleur sur votre appareil StorSimple à partir de Windows PowerShell pour StorSimple, procédez comme suit.

1. Accédez à l’appareil via la console série ou une session telnet à partir d’un ordinateur distant. Pour vous connecter au Contrôleur 0 ou au Contrôleur 1, suivez la procédure décrite dans [Utilisation de PuTTY pour se connecter à la console série de l’appareil](storsimple-8000-deployment-walkthrough-u2.md#use-putty-to-connect-to-the-device-serial-console).
2. Dans le menu de la console série, sélectionnez l’option 1, **Ouvrir une session avec un accès total**.
3. Dans le message de bannière, prenez note du contrôleur auquel vous êtes connecté (contrôleur 0 ou contrôleur 1) et vérifiez s’il est actif ou bien passif (en veille).
   
   * Pour arrêter un contrôleur unique, à l’invite de commandes, tapez :
     
       `Stop-HcsController`
     
       Ceci arrête le contrôleur auquel vous êtes connecté. Si vous arrêtez le contrôleur actif, l’appareil bascule vers le contrôleur passif.

   * Pour redémarrer un contrôleur, à l’invite de commandes, tapez :
     
       `Restart-HcsController`
     
       Ceci redémarre le contrôleur auquel vous êtes connecté. Si vous redémarrez le contrôleur actif, l’appareil bascule vers le contrôleur passif avant de redémarrer.

## <a name="shut-down-a-storsimple-device"></a>arrêter un appareil StorSimple

Cette section vous explique comment arrêter un appareil StorSimple en cours d’exécution ou défaillant à partir d’un ordinateur distant. Un appareil est désactivé dès lors que les contrôleurs d’appareil sont arrêtés. L’arrêt d’un appareil est exécuté lorsque l’appareil est physiquement déplacé ou mis hors service.

> [!IMPORTANT]
> Avant d’arrêter l’appareil, vérifiez l’intégrité des composants de l’appareil. Accédez à votre appareil, puis cliquez sur **Paramètres > Intégrité du matériel**. Dans le panneau **État et intégrité du matériel**, assurez-vous que l’état des LED des composants est vert. Un appareil en bon état de fonctionnement a un état vert. Si votre appareil est sur le point d’être arrêté pour procéder au remplacement d’un composant défaillant, vous verrez s’afficher un état Échec (rouge) ou un état Détérioré (jaune) pour les composants respectifs.


#### <a name="to-shut-down-a-storsimple-device"></a>Pour arrêter un appareil StorSimple

1. Utilisez la procédure [Redémarrage ou d’arrêt d’un contrôleur](#restart-or-shut-down-a-single-controller) pour identifier et arrêter le contrôleur passif sur votre appareil. Vous pouvez effectuer cette opération dans le portail Azure ou dans Windows PowerShell pour StorSimple.
2. Répétez l’étape précédente pour arrêter le contrôleur actif.
3. Vous devez maintenant examiner le panneau arrière de l’appareil. Une fois les deux contrôleurs arrêtés, l’état des LED des deux contrôleurs doit être rouge clignotant. Si vous souhaitez désactiver complètement l’appareil à ce stade, assurez-vous que les commutateurs d’alimentation sont en position d’arrêt sur tous les PCM. Ceci désactivera l’appareil.

## <a name="reset-the-device-to-factory-default-settings"></a>Rétablissement des paramètres d’usine de l’appareil

> [!IMPORTANT]
> Si vous devez rétablir les paramètres d’usine de l’appareil, contactez le Support Microsoft. La procédure décrite ci-dessous ne doit être effectuée qu’en coordination avec le Support Microsoft.

Cette procédure explique comment rétablir les paramètres d’usine de votre appareil Microsoft Azure StorSimple à l’aide de Windows PowerShell pour StorSimple.
La réinitialisation d’un appareil supprime toutes les données et tous les paramètres de l’ensemble du cluster par défaut.

Pour rétablir les paramètres d’usine de votre appareil Microsoft Azure StorSimple, procédez comme suit :

### <a name="to-reset-the-device-to-default-settings-in-windows-powershell-for-storsimple"></a>Pour rétablir les paramètres d’usine de l’appareil dans Windows PowerShell pour StorSimple
1. Accédez à l’appareil via sa console série. Vérifiez le message de bannière pour vous assurer que vous êtes connecté au contrôleur **actif**.
2. Dans le menu de la console série, sélectionnez l’option 1, **Ouvrir une session avec un accès total**.
3. À l’invite, tapez la commande suivante pour réinitialiser l’ensemble du cluster et ainsi supprimer toutes les données, toutes les métadonnées et tous les paramètres du contrôleur :
   
    `Reset-HcsFactoryDefault`
   
    Pour réinitialiser un seul contrôleur, utilisez l’applet de commande [Reset-HcsFactoryDefault](/previous-versions/windows/powershell-scripting/dn688132(v=wps.630)) avec le paramètre `-scope`.)
   
    Le système redémarrera plusieurs fois. Un message s’affichera une fois la réinitialisation terminée. Selon le modèle du système, l’opération peut prendre 45 à 60 minutes pour un appareil 8 100 et 60 à 90 minutes pour un appareil 8 600.
   
## <a name="questions-and-answers-about-managing-device-controllers"></a>Questions et réponses sur la gestion des contrôleurs d’appareil
Dans cette section, vous trouverez des réponses à certaines questions fréquemment posées, relatives à la gestion des contrôleurs d’appareil StorSimple.

**Q.** Que se passe-t-il si les deux contrôleurs de mon appareil sont activés et en bon état de fonctionnement, et que je redémarre ou arrête le contrôleur actif ?

**A.** Si les deux contrôleurs de votre appareil sont activés et en bon état de fonctionnement, vous êtes invité à confirmer l’opération. Vous pouvez choisir de :

* **Redémarrer le contrôleur actif** : un message vous avertit que le redémarrage d’un contrôleur actif entraîne un basculement de l’appareil vers le contrôleur passif. Le contrôleur redémarre.
* **Arrêter un contrôleur actif** : un message vous avertit que l’arrêt d’un contrôleur actif entraîne une coupure du service. Vous devez également appuyer sur le bouton d’alimentation de l’appareil pour activer le contrôleur.

**Q.** Que se passe-t-il si le contrôleur passif de mon appareil est indisponible ou désactivé, et que je redémarre ou arrête le contrôleur actif ?

**A.** Si le contrôleur passif de votre appareil est indisponible ou désactivé, et que vous choisissez de :

* **Redémarrer le contrôleur actif** : un message vous avertit que la poursuite de l’opération entraînera une interruption temporaire du service, et vous êtes invité à confirmer l’opération.
* **Arrêter un contrôleur actif** : un message vous avertit que la poursuite de l’opération entraînera une coupure du service. Vous devez également appuyer sur le bouton d’alimentation de l’un ou des deux contrôleurs pour activer l’appareil. Vous êtes invité à confirmer l’opération.

**Q.** À quel moment le redémarrage ou l’arrêt du contrôleur échoue-t-il ?

**A.** Le redémarrage ou l’arrêt d’un contrôleur peut échouer si :

* Une mise à jour de l’appareil est en cours.
* Le redémarrage d’un contrôleur est déjà en cours.
* L’arrêt d’un contrôleur est déjà en cours.

**Q.** Comment pouvez-vous déterminer si un contrôleur a été redémarré ou arrêté ?

**A.** Vous pouvez vérifier l’état du contrôleur dans le panneau Contrôleur. L’état du contrôleur indique si un contrôleur est en cours de redémarrage ou d’arrêt. De plus, le panneau **Alertes** contient une alerte d’information vous indiquant si le contrôleur est redémarré ou arrêté. Les opérations de redémarrage et d’arrêt du contrôleur sont également enregistrées dans les journaux d’activité. Pour plus d’informations sur les journaux d’activité, consultez [Afficher les journaux d’activité](storsimple-8000-service-dashboard.md#view-the-activity-logs).

**Q.** Le basculement du contrôleur a-t-il un impact sur les E/S ?

**A.** Les connexions TCP entre les initiateurs et le contrôleur actif sont réinitialisées à la suite du basculement du contrôleur, mais elles seront rétablies lorsque le contrôleur passif effectuera l’opération. Une interruption temporaire (moins de 30 secondes) peut survenir dans l’activité d’E/S entre les initiateurs et l’appareil au cours de cette opération.

**Q.** Comment puis-je remettre mon contrôleur en service une fois qu’il a été arrêté et supprimé ?

**A.** Pour remettre en service un contrôleur, vous devez l’insérer dans le châssis, comme décrit dans [Remplacement d’un module de contrôleur sur votre appareil StorSimple](storsimple-8000-controller-replacement.md).

## <a name="next-steps"></a>Étapes suivantes
* Si vous rencontrez des problèmes avec vos contrôleurs d’appareil StorSimple, que vous ne pouvez pas résoudre en suivant les procédures décrites dans ce didacticiel, [contactez le support technique Microsoft](storsimple-8000-contact-microsoft-support.md).
* Pour en savoir plus sur l’utilisation du service StorSimple Device Manager, consultez [Utilisation du service StorSimple Device Manager pour gérer votre appareil StorSimple](storsimple-8000-manager-service-administration.md).