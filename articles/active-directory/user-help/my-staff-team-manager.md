---
title: Gérer les mots de passe et les numéros de téléphone avec Mon personnel (préversion) - Azure AD | Microsoft Docs
description: Gérez les mots de passe et les numéros de téléphone de vos utilisateurs avec Mon personnel
services: active-directory
documentationcenter: ''
author: curtand
manager: daveba
ms.topic: end-user-help
ms.service: active-directory
ms.subservice: user-help
ms.workload: identity
ms.date: 04/14/2020
ms.author: curtand
ms.reviewer: sahenry
ms.custom: oldportal;it-pro;
ms.openlocfilehash: 56ee7df52abf97c920dfe2fa4ba5d1d858975f7d
ms.sourcegitcommit: f5580dd1d1799de15646e195f0120b9f9255617b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/29/2020
ms.locfileid: "91537088"
---
# <a name="delegate-user-management-with-my-staff-preview"></a>Déléguer la gestion des utilisateurs avec Mon personnel (préversion)

Votre organisation peut avoir recours à **Mon personnel** pour déléguer les tâches de gestion des utilisateurs à un responsable (gérant de magasin ou chef d'équipe, par exemple) afin d'aider les membres du personnel à accéder aux applications dont ils ont besoin. Si un membre de votre équipe n'a pas accès à une application parce qu'il a oublié un mot de passe, sa productivité en pâtira. Cela se traduira également par une augmentation des coûts liés au support et par un goulot d'étranglement au niveau de vos processus administratifs.  Avec Mon personnel, un membre d'équipe qui ne parvient pas à accéder à son compte peut récupérer cet accès en quelques clics, sans intervention de l'administrateur.

## <a name="manage-your-staff-in-my-staff"></a>Gérer votre personnel dans Mon personnel

La gestion des membres de votre équipe dans Mon Personnel est simple. Pour commencer, [accédez à Mon personnel](https://aka.ms/mystaff), sélectionnez une équipe ou un emplacement, puis sélectionnez un utilisateur. Les emplacements et les membres de l'équipe d'un emplacement donné sont déterminés par votre administrateur informatique et vous ne pouvez pas les modifier.

Si vous gérez plusieurs emplacements, lorsque vous accédez à Mon personnel, vous devez en sélectionner un pour afficher les membres d'équipe attribués à celui-ci.

Si vous ne disposez pas encore des autorisations suffisantes pour accéder à Mon personnel, le message suivant s’affiche : « Désolé, vous n’êtes pas autorisé à consulter Mon personnel pour le moment. Pour plus d'informations, contactez votre administrateur. »

### <a name="find-a-staff-member-in-my-staff"></a>Rechercher un membre du personnel dans Mon personnel

Vous devez ouvrir le profil d'un membre du personnel avant de pouvoir commencer à le gérer.

1. [Ouvrez Mon personnel](https://aka.ms/mystaff) et, si nécessaire, sélectionnez un emplacement.

    ![Sélectionner un emplacement pour un membre d'équipe dans Mon personnel](media/my-staff-team-manager/allaus.png)

1. Ouvrez le profil d'un membre de l'équipe.

    ![Sélectionner un utilisateur associé à un emplacement dans Mon personnel](media/my-staff-team-manager/aupage.png)

## <a name="reset-a-user-password"></a>Réinitialiser le mot de passe d’un utilisateur

Si votre organisation vous en a donné l'autorisation, vous pouvez réinitialiser les mots de passe des membres de votre personnel.

1. [Ouvrez Mon personnel](https://aka.ms/mystaff).
1. Ouvrez le profil d'un membre du personnel.
1. Sélectionnez **Réinitialiser le mot de passe**.

    ![Réinitialiser le mot de passe d'un utilisateur dans Mon personnel](media/my-staff-team-manager/resetpassword1.png)

1. Générez ou entrez le nouveau mot de passe. Un mot de passe temporaire généré automatiquement peut s'afficher ou vous pouvez être invité à entrer celui-ci pour l'utilisateur.

    ![Copier le mot de passe temporaire de l'utilisateur après une réinitialisation dans Mon personnel](media/my-staff-team-manager/resetpassword2.png)

Après avoir réinitialisé le mot de passe de l'utilisateur, attribuez-lui le mot de passe temporaire. Lorsque l'utilisateur se connecte avec son mot de passe temporaire, il doit le modifier.

## <a name="manage-a-users-phone-number"></a>Gérer le numéro de téléphone d'un utilisateur

Si votre organisation vous en a donné l'autorisation, vous pouvez gérer les numéros de téléphone des membres de votre personnel.

### <a name="add-a-phone-number"></a>Ajouter un numéro de téléphone

1. [Ouvrez Mon personnel](https://aka.ms/mystaff).
1. Ouvrez le profil d'un membre du personnel.
1. Sélectionnez **Ajouter un numéro de téléphone**.

    ![Ajouter le numéro de téléphone d'un utilisateur dans Mon personnel](media/my-staff-team-manager/addphone1.png)

1. Ajoutez le numéro de téléphone et sélectionnez **Enregistrer**.

    ![Enregistrer le numéro de téléphone de l'utilisateur ajouté dans Mon personnel](media/my-staff-team-manager/addphone2.png)

Une fois que vous avez enregistré le numéro de téléphone d'un utilisateur, celui-ci peut l'utiliser pour se connecter par SMS, effectuer une vérification en deux étapes ou réinitialiser lui-même son mot de passe, selon les paramètres de votre organisation.

![Nouveau numéro de téléphone enregistré auprès de Mon personnel](media/my-staff-team-manager/addphone3.png)

### <a name="edit-a-phone-number"></a>Modifier un numéro de téléphone

1. [Ouvrez Mon personnel](https://aka.ms/mystaff).
1. Ouvrez le profil d'un membre du personnel.
1. Sélectionnez **Modifier le numéro de téléphone**.

    ![Sélectionner Modifier à partir du profil utilisateur dans Mon personnel](media/my-staff-team-manager/editphone2.png)

1. Entrez le nouveau numéro de téléphone, puis sélectionnez **Enregistrer**.

    ![Modifier le numéro de téléphone d'un membre du personnel dans Mon personnel](media/my-staff-team-manager/editphone1.png)

### <a name="enable-phone-number-sign-in-for-a-user"></a>Activer la connexion à l'aide d'un numéro de téléphone pour un utilisateur

Si la connexion à l'aide d'un numéro de téléphone en guise de nom d'utilisateur (connexion par SMS) est activée dans votre organisation, vous pouvez ajouter cette authentification à un numéro de téléphone d'utilisateur existant.

1. [Ouvrez Mon personnel](https://aka.ms/mystaff).
1. Ouvrez le profil d'un membre du personnel.
1. Si un message indiquant que vous pouvez vous connecter avec votre numéro de téléphone en guise de nom d'utilisateur apparaît en bas de l'écran, sélectionnez **Activer** pour lancer le processus. Ce message apparaît si l'utilisateur a été autorisé à se connecter avec son numéro de téléphone.

    ![Voir le message lorsque la connexion par téléphone est prise en charge à un emplacement donné dans Mon personnel](media/my-staff-team-manager/enableforms1.png)

1. Lorsque vous avez terminé, sélectionnez **OK**.

    ![Capture d’écran de la fenêtre « Activer le numéro de téléphone pour la connexion ? » avec le bouton « OK » sélectionné](media/my-staff-team-manager/enableforms2.png)

### <a name="remove-a-phone-number"></a>Supprimer un numéro de téléphone

1. [Ouvrez Mon personnel](https://aka.ms/mystaff).
1. Ouvrez le profil d'un membre du personnel.
1. Sélectionnez **Supprimer le numéro de téléphone**.
1. Lorsque vous avez terminé, sélectionnez **Supprimer**.

    ![Supprimer le numéro de téléphone d'un membre du personnel dans Mon personnel](media/my-staff-team-manager/deletephone1.png)
