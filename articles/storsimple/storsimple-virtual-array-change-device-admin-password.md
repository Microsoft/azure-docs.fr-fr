---
title: Modifier le mot de passe administrateur de l’appareil StorSimple Virtual Array | Microsoft Docs
description: Explique comment utiliser le portail Azure ou l’interface utilisateur web de StorSimple Virtual Array pour modifier le mot de passe administrateur de l’appareil.
services: storsimple
documentationcenter: NA
author: alkohli
manager: timlt
editor: ''
ms.assetid: 11490814-d9fd-4dc7-9c3b-55dd2c23eaf1
ms.service: storsimple
ms.devlang: NA
ms.topic: how-to
ms.tgt_pltfrm: NA
ms.workload: TBD
ms.date: 02/27/2017
ms.author: alkohli
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: 8bc8846d546faec194617ccb753cdbd105e16bf9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85513613"
---
# <a name="change-the-storsimple-virtual-array-device-administrator-password-via-storsimple-device-manager"></a>Modifier le mot de passe administrateur de l’appareil StorSimple Virtual Array via StorSimple Device Manager

## <a name="overview"></a>Vue d’ensemble

Quand vous utilisez l’interface Windows PowerShell pour accéder au StorSimple Virtual Array, vous devez entrer un mot de passe administrateur pour l’appareil. Lorsque l'appareil StorSimple est d'abord configuré puis démarré, le mot de passe par défaut est *Password1*. Pour garantir la sécurité de vos données, le mot de passe par défaut expire dès votre première connexion et vous devez donc le modifier.

Vous pouvez également utiliser l’interface utilisateur web locale ou le portail Azure pour modifier à tout moment le mot de passe administrateur une fois l’appareil déployé dans votre environnement de production. Chacune de ces procédures est décrite dans cet article.

 ![Panneau Appareils](./media/storsimple-virtual-array-change-device-admin-password/ova-devices-blade.png)

## <a name="use-the-azure-portal-to-change-the-password"></a>Utiliser le portail Azure pour modifier le mot de passe

Procédez comme suit pour modifier le mot de passe administrateur de l’appareil via le portail Azure.

#### <a name="to-change-the-device-administrator-password-via-the-azure-portal"></a>Pour modifier le mot de passe administrateur de l’appareil via le portail Azure

1. Dans la page d’accueil du service, sélectionnez votre service, double-cliquez sur le nom du service, puis dans la section **Gestion**, cliquez sur **Appareils**. Le panneau **Appareils**, qui répertorie tous vos appareils StorSimple Virtual Array, s’ouvre.

2. Dans le panneau **Appareils**, double-cliquez sur l’appareil dont le mot de passe doit être modifié.

3. Dans le panneau **Paramètres** de votre appareil, cliquez sur **Sécurité**.

4. Dans le panneau **Paramètres de sécurité**, procédez comme suit :
   
   1. Faites défiler l’écran vers le bas jusqu’à la section **Mot de passe de l’administrateur de l’appareil** . Indiquez un mot de passe Administrateur contenant entre 8 et 15 caractères.
   2. Confirmez le mot de passe.
   3. Cliquez sur **Enregistrer** dans la partie supérieure du panneau.

Le mot de passe administrateur de l’appareil est maintenant à jour. Vous pouvez utiliser ce mot de passe modifié pour accéder à l’appareil localement.

![Panneau Paramètres de sécurité](./media/storsimple-virtual-array-change-device-admin-password/ova-change-device-pwd.png)

## <a name="use-the-local-web-ui-to-change-the-password"></a>Utiliser l’interface utilisateur web locale pour modifier le mot de passe

Procédez comme suit pour modifier le mot de passe d’administrateur de l’appareil via l’interface utilisateur web locale.

#### <a name="to-change-the-device-administrator-password-via-the-local-web-ui"></a>Pour modifier le mot de passe administrateur de l’appareil via l’interface utilisateur web locale

1. Dans l’interface utilisateur web locale, cliquez sur **Maintenance** > **Modifier le mot de passe** pour votre appareil.
   
    ![change password1](./media/storsimple-virtual-array-change-device-admin-password/image40.png)
2. Entrez le **mot de passe actuel**.
3. Fournissez un **nouveau mot de passe**. Le mot de passe doit comporter au moins 8 caractères. Il doit contenir 3 caractères sur 4 en majuscules, minuscules, chiffres et caractères spéciaux.
   
    Notez que votre mot de passe ne peut pas être identique aux 24 derniers mots de passe.
4. Entrez de nouveau le mot de passe pour le confirmer.
   
    ![change password2](./media/storsimple-virtual-array-change-device-admin-password/image41.png)
5. En bas de cette page, cliquez sur **Appliquer**. Le nouveau mot de passe est maintenant appliqué. Si la modification du mot de passe échoue, l’erreur suivante s’affiche :
   
    ![password error](./media/storsimple-virtual-array-change-device-admin-password/image42.png)
   
    Vous êtes averti une fois le mot de passe correctement mis à jour. Vous pouvez ensuite utiliser ce mot de passe modifié pour accéder à l’appareil localement.


## <a name="next-steps"></a>Étapes suivantes
Découvrez comment [gérer votre StorSimple Virtual Array](storsimple-ova-web-ui-admin.md).

