---
title: Modification de vos mots de passe StorSimple | Microsoft Docs
description: Explique comment utiliser le service StorSimple Device Manager pour modifier vos mots de passe du Gestionnaire d’instantanés StorSimple et d’administrateur de l’appareil.
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: ''
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 07/03/2017
ms.author: alkohli
ms.openlocfilehash: 038084ba9ae43e14bc2eb42bf258912be27d062c
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96023752"
---
# <a name="use-the-storsimple-device-manager-service-to-change-your-storsimple-passwords"></a>Utilisation du service StorSimple Device Manager pour modifier vos mots de passe StorSimple

## <a name="overview"></a>Vue d’ensemble
L’option **Configurer** du portail Azure contient tous les paramètres d’appareil que vous pouvez reconfigurer sur un appareil StorSimple géré par un service StorSimple Device Manager. Ce didacticiel explique comment utiliser l’option **Sécurité** sous **Paramètres de l’appareil** pour modifier le mot de passe d’administrateur de l’appareil ou le mot de passe du Gestionnaire d’instantanés StorSimple.

## <a name="change-the-device-administrator-password"></a>Modification du mot de passe d’administrateur de l’appareil
Quand vous utilisez l’interface Windows PowerShell pour accéder à l’appareil StorSimple, vous devez entrer un mot de passe d’administrateur de l’appareil. À l’inscription du premier appareil StorSimple auprès d’un service, le mot de passe par défaut de cette interface est *Password1*. Pour assurer la sécurité de vos données, il vous est demandé de modifier ce mot de passe à la fin de l’inscription. Vous ne pouvez pas quitter le processus d'inscription sans modifier ce mot de passe. Pour plus d’informations, voir [Étape 3 : Configuration et inscription de l’appareil via Windows PowerShell pour StorSimple](storsimple-8000-deployment-walkthrough-u2.md#step-3-configure-and-register-the-device-through-windows-powershell-for-storsimple).

Le mot de passe qui a été défini initialement dans l’interface Windows PowerShell au moment de l’inscription peut être modifié ultérieurement via le portail Azure. Pour modifier le mot de passe d’administrateur de l’appareil, procédez comme suit.

#### <a name="to-change-the-device-administrator-password"></a>Pour modifier le mot de passe d’administrateur de l’appareil
1. Accédez à votre service StorSimple Device Manager et cliquez sur **Appareils**.

2. Dans la liste tabulaire d’appareils, sélectionnez et cliquez sur l’appareil dont vous souhaitez modifier le mot de passe.

    ![Capture d’écran montrant le service Gestionnaire de périphériques StorSimple. Sous Gestion, Appareils est sélectionné. Dans la liste des appareils, un appareil est sélectionné.](./media/storsimple-8000-change-passwords/changepwd1.png)

3. Dans le panneau **Paramètres**, accédez à **Paramètres de l’appareil > Sécurité**.

    ![Capture d'écran montrant le panneau Paramètres du service Gestionnaire de périphériques. Sous Paramètres de l’appareil, Sécurité est sélectionné.](./media/storsimple-8000-change-passwords/changepwd2.png)

4. Dans le panneau **Paramètres de sécurité**, cliquez sur **Mot de passe** pour modifier le mot de passe d’administrateur de l’appareil.

    ![Capture d'écran montrant les paramètres pour le panneau Paramètres de sécurité. Le bouton Mot de passe est mis en surbrillance.](./media/storsimple-8000-change-passwords/changepwd3.png)

5. Dans le panneau **Mot de passe**, indiquez un mot de passe d’administrateur contenant entre 8 et 15 caractères. Le mot de passe doit contenir une combinaison d’au moins 3 caractères en majuscules, en minuscules, numériques et spéciaux.

6. Confirmez le mot de passe.

    ![Capture d’écran montrant le panneau Mot de passe. Sous Mot de passe administrateur du Gestionnaire d’instantanés, les zones Nouveau mot de passe et Confirmer le mot de passe sont renseignées.](./media/storsimple-8000-change-passwords/changepwd4.png)

7. Cliquez sur **Enregistrer** puis, lorsque vous êtes invité à confirmer l’opération, cliquez sur **Oui**.

    ![Capture d’écran montrant le panneau Mot de passe. Le bouton Enregistrer est en surbrillance.](./media/storsimple-8000-change-passwords/changepwd6.png)

Le mot de passe Administrateur d’appareil doit maintenant être à jour. Vous pouvez utiliser ce mot de passe modifié pour accéder à l’interface Windows PowerShell.

## <a name="set-the-storsimple-snapshot-manager-password"></a>Définition du mot de passe du Gestionnaire d’instantanés StorSimple
Le Gestionnaire d’instantanés StorSimple réside sur l’ordinateur hôte Windows et permet aux administrateurs de gérer les sauvegardes de votre appareil StorSimple sous la forme d’instantanés cloud ou locaux.

Au moment de configurer un appareil dans le Gestionnaire d’instantanés StorSimple, vous êtes invité à fournir l’adresse IP et le mot de passe de votre appareil de stockage afin de l’authentifier.

Vous pouvez définir ou modifier le mot de passe du Gestionnaire d’instantanés StorSimple via le portail Azure. Procédez comme suit pour définir ou modifier le mot de passe du Gestionnaire d’instantanés StorSimple.

#### <a name="to-set-the-storsimple-snapshot-manager-password"></a>Pour définir le mot de passe du Gestionnaire d’instantanés StorSimple
1. Accédez à votre service StorSimple Device Manager et cliquez sur **Appareils**.

2. Dans la liste tabulaire d’appareils, sélectionnez et cliquez sur l’appareil dont vous souhaitez définir ou modifier le mot de passe du Gestionnaire d’instantanés StorSimple.

     ![Capture d’écran montrant le service Gestionnaire de périphériques StorSimple. Sous Gestion, Appareils est sélectionné. Dans la liste des appareils, un appareil est sélectionné.](./media/storsimple-8000-change-passwords/changepwd1.png)

3. Dans le panneau **Paramètres**, accédez à **Paramètres de l’appareil > Sécurité**.

     ![Capture d'écran montrant le panneau Paramètres du service Gestionnaire de périphériques. Sous Paramètres de l’appareil, Sécurité est sélectionné.](./media/storsimple-8000-change-passwords/changepwd2.png)

4. Dans le panneau **Paramètres de sécurité**, cliquez sur **Mot de passe** pour définir ou modifier le mot de passe du Gestionnaire d’instantanés StorSimple.

     ![Capture d'écran montrant les paramètres pour le panneau Paramètres de sécurité. Le bouton Mot de passe est mis en surbrillance.](./media/storsimple-8000-change-passwords/changepwd3.png) 

5. Dans le panneau **Mot de passe**, entrez un mot de passe de 14 ou 15 caractères. Assurez-vous que le mot de passe contient une combinaison d’au moins 3 caractères en majuscules, en minuscules, numériques et spéciaux.

6. Confirmez le mot de passe.

     ![Capture d’écran montrant le panneau Mot de passe. Sous Mot de passe Snapshot Manager, les zones Nouveau mot de passe et Confirmer le mot de passe sont renseignées.](./media/storsimple-8000-change-passwords/changepwd5.png)

7. Cliquez sur **Enregistrer** puis, lorsque vous êtes invité à confirmer l’opération, cliquez sur **Oui**.

     ![Capture d’écran montrant le panneau Mot de passe. Le bouton Enregistrer est en surbrillance.](./media/storsimple-8000-change-passwords/changepwd6.png)

Le mot de passe du Gestionnaire d’instantanés StorSimple doit maintenant être mis à jour.

## <a name="next-steps"></a>Étapes suivantes
* En savoir plus sur la [sécurité StorSimple](storsimple-8000-security.md).
* En savoir plus sur la [modification de la configuration de votre appareil](storsimple-8000-modify-device-config.md).
* En savoir plus sur l’[utilisation du service StorSimple Device Manager pour gérer votre appareil StorSimple](storsimple-8000-manager-service-administration.md).

