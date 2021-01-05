---
title: Créer un ticket ou cas de support pour la gamme StorSimple 8000
description: Découvrez comment créer une demande de support et démarrer une session de support sur votre appareil de la gamme StorSimple 8000.
author: alkohli
ms.service: storsimple
ms.topic: how-to
ms.date: 01/09/2018
ms.author: alkohli
ms.openlocfilehash: c288f59b1cb9a85ed4bc978d1c3fba18ba30b572
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96017200"
---
# <a name="contact-microsoft-support"></a>Contacter le support Microsoft

StorSimple Device Manager offre la possibilité de **créer une demande de support** à partir du panneau de synthèse du service. Si vous rencontrez des problèmes avec votre solution StorSimple, vous pouvez créer une demande de service pour obtenir un support technique. Lors d’une session en ligne avec votre ingénieur de support, vous devrez également démarrer une session de support sur votre appareil StorSimple. Cet article vous guide tout au long des procédures suivantes :

* Création d’une demande de support
* Gestion du cycle de vie d’une demande de support à partir du portail
* Démarrage d’une session de support dans l’interface Windows PowerShell de votre appareil StorSimple

Examinez les [informations et les contrats de niveau de service relatifs à la prise en charge de la gamme StorSimple 8000](./storsimple-8000-support-options.md) avant de créer une demande de support.

## <a name="create-a-support-request"></a>Création d’une demande de support

Selon votre [plan de support](https://azure.microsoft.com/support/plans/), vous pouvez créer des tickets de support pour un problème affectant votre appareil StorSimple directement à partir du panneau de synthèse du service StorSimple Device Manager. Procédez comme suit pour créer une demande de support.

#### <a name="to-create-a-support-request"></a>Création d’une demande de support

1. Accédez à votre service StorSimple Device Manager. Dans les paramètres du panneau de synthèse du service, accédez à la section **SUPPORT + DÉPANNAGE**, puis cliquez sur **Nouvelle demande de support**.
     
    ![Contacter le support MS avec le nouveau portail](./media/storsimple-8000-contact-microsoft-support/contactsupport1.png)
   
2. Dans le panneau **Nouvelle demande de support**, cliquez sur **De base**. Dans le panneau **De base**, procédez comme suit :
   1. Dans la liste déroulante **Type de problème**, sélectionnez **Technique**.
   2. L’**abonnement**, le type de **service** et la **ressource** (service StorSimple Device Manager) actuels sont automatiquement sélectionnés. 
   3. Sélectionnez un **plan de support** dans la liste déroulante si plusieurs plans sont associés à votre abonnement. Vous avez besoin d'un plan de support payant pour bénéficier du support technique.
   4. Cliquez sur **Suivant**.

       ![Contacter le support MS avec le nouveau portail 2](./media/storsimple-8000-contact-microsoft-support/contactsupport2.png)

3. Dans le panneau **Nouvelle demande de support**, sélectionnez **2 Problème**. Dans le panneau **Problème**, procédez comme suit :
    
    1. Choisissez la **gravité**.
    2. Indiquez si le problème est lié à l’appliance ou au service StorSimple Device Manager.
    3. Choisissez une **catégorie** pour le problème et fournissez des **détails** supplémentaires sur ce dernier.
    4. Fournissez la date et l’heure de début du problème.
    5. Sous **Chargement de fichier**, cliquez sur l’icône du dossier pour sélectionner votre package de support.
    6. Sélectionnez la case à cocher **Partager les informations de diagnostic**.
    7. Cliquez sur **Suivant**.

       ![Contacter le support MS avec le nouveau portail 3](./media/storsimple-8000-contact-microsoft-support/contactsupport3.png) 

4. Dans le panneau **Nouvelle demande de support**, cliquez sur **3 Informations de contact**. Dans le panneau **Informations de contact**, procédez comme suit :

   1. Dans la section **Options de contact**, fournissez votre méthode de contact préférée (téléphone ou e-mail) et votre langue. Le temps de réponse est automatiquement sélectionné en fonction de votre plan d’abonnement.
   2. Dans la section Informations de contact, fournissez votre nom, votre adresse e-mail, un contact secondaire (facultatif) et votre pays/région. Cochez la case **Enregistrer les modifications de contact pour les futures demandes de support** .
   3. Cliquez sur **Créer**.
   
       ![Contacter le support MS avec le nouveau portail 4](./media/storsimple-8000-contact-microsoft-support/contactsupport5.png)   

      Le Support Microsoft vous contactera à l’aide de ces informations pour approfondir le diagnostic et vous aider à résoudre le problème.
      Une fois votre demande envoyée, un ingénieur de support vous contactera dès que possible pour traiter votre demande.

## <a name="manage-a-support-request"></a>Gérer une demande de support

Après avoir créé un ticket de support, vous pouvez gérer son cycle de vie depuis le portail.

#### <a name="to-manage-your-support-requests"></a>Pour gérer vos demandes de support

1. Pour accéder à la page d’aide et de support, cliquez sur **Parcourir > Aide + support**.

    ![Gérer les demandes de support](./media/storsimple-8000-contact-microsoft-support/managesupport1.png)

2. Une liste tabulaire de toutes les demandes de support est affichée dans le panneau **Aide + support**.

    ![Gérer les demandes de support 2](./media/storsimple-8000-contact-microsoft-support/managesupport2.png)

3. Sélectionnez une demande de support. Vous pouvez afficher l’état et les détails de cette demande. Si vous souhaitez donner suite à cette demande, cliquez sur **+ Nouveau message**.

    ![Gérer les demandes de support 3](./media/storsimple-8000-contact-microsoft-support/managesupport3.png)

## <a name="start-a-support-session-in-windows-powershell-for-storsimple"></a>Démarrage d’une session de support dans Windows PowerShell pour StorSimple

Pour résoudre les problèmes que vous pouvez rencontrer avec l’appareil StorSimple, vous devrez contacter l'équipe de Support technique de Microsoft. Le support Microsoft devra peut-être utiliser une session de support pour se connecter à votre appareil.

Procédez comme suit pour démarrer une session de support :

#### <a name="to-start-a-support-session"></a>Démarrage d’une session de support

1. Accédez directement à l’appareil à l'aide de la console série ou via une session telnet à partir d'un ordinateur distant. Pour cela, suivez les étapes de la section [Utilisation de PuTTY pour se connecter à la console série de l’appareil](storsimple-8000-deployment-walkthrough-u2.md#use-putty-to-connect-to-the-device-serial-console).
2. Dans la session qui s’ouvre, appuyez sur la touche **Entrée** pour afficher une invite de commandes.
3. Dans le menu de la console série, sélectionnez l’option 1, **Ouvrir une session avec un accès total**.
4. À l'invite de commande, entrez le mot de passe suivant :
   
    `Password1`
5. À l'invite de commande, tapez la commande suivante :
   
    `Enable-HcsSupportAccess`
6. Une chaîne chiffrée s'affiche. Copiez cette chaîne dans un éditeur de texte comme le Bloc-notes.
7. Enregistrez cette chaîne et envoyez-la par e-mail au support Microsoft.

> [!IMPORTANT]
> Vous pouvez désactiver l’accès au support en exécutant `Disable-HcsSupportAccess`. L’appareil StorSimple tentera également de désactiver l'accès au support 8 heures après le début de la session. Il est recommandé de modifier vos informations d'identification de l’appareil StorSimple après le lancement d'une session de support.


## <a name="next-steps"></a>Étapes suivantes

Découvrez comment [diagnostiquer et résoudre les problèmes liés à votre appareil de la gamme StorSimple 8000](storsimple-8000-troubleshoot-deployment.md).