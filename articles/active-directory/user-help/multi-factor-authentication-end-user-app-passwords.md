---
title: Gestion des mots de passe d’application – Azure Active Directory | Microsoft Docs
description: Découvrez les mots de passe d’application et le rôle de ces derniers dans le cadre de la vérification en deux étapes.
services: active-directory
author: curtand
manager: daveba
ms.reviewer: richagi
ms.assetid: 345b757b-5a2b-48eb-953f-d363313be9e5
ms.workload: identity
ms.service: active-directory
ms.subservice: user-help
ms.topic: end-user-help
ms.date: 05/28/2020
ms.author: curtand
ms.custom: user-help, seo-update-azuread-jan
ms.openlocfilehash: c9d8bb85aac87503ca641f6633668a6dc582882f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90056079"
---
# <a name="manage-app-passwords-for-two-step-verification"></a>Gérer les mots de passe d’application pour la vérification en deux étapes

>[!Important]
>Votre administrateur peut ne pas vous autoriser à utiliser des mots de passe d’application. Si vous ne voyez pas l’option **Mots de passe d’application**, cela signifie qu’elle n’est pas disponible dans votre organisation.

Lorsque vous utilisez des mots de passe d’application, gardez à l’esprit les points suivants :

- Les mots de passe d’application sont générés automatiquement. Ils doivent être créés et entrés une fois par application.

- Un utilisateur peut posséder jusqu’à 40 mots de passe. Si vous essayez de créer un mot de passe supplémentaire au-delà de cette limite, vous serez invité à supprimer un mot de passe existant avant d’être autorisé à en créer un autre.

    >[!Note]
    >Les clients Office 2013 (y compris Outlook) prennent en charge de nouveaux protocoles d’authentification et peuvent être utilisés dans le cadre de la vérification en deux étapes. Cela signifie qu’après l’activation de la vérification en deux étapes, vous n’aurez plus besoin de mots de passe d’application pour les clients Office 2013. Pour plus d’informations, consultez l’article [Fonctionnement de l’authentification moderne pour les applications clientes Office 2013 et Office 2016](https://support.office.com/article/how-modern-authentication-works-for-office-2013-and-office-2016-client-apps-e4c45989-4b1a-462e-a81b-2a13191cf517).

## <a name="create-new-app-passwords"></a>Créer de nouveaux mots de passe d’application

À l’occasion du processus initial d’inscription à la vérification à deux facteurs, un mot de passe d’application unique vous est fourni. Si vous en avez besoin de plusieurs, vous devez les créer par vos propres moyens. Vous pouvez créer des mots de passe d’application de plusieurs endroits, selon la façon dont la vérification à deux facteurs est configurée dans votre organisation. Pour plus d’informations sur l’inscription à la vérification à deux facteurs en vue de l’utiliser avec votre compte professionnel ou scolaire, consultez [Vue d’ensemble de la vérification à deux facteurs et de votre compte professionnel ou scolaire](multi-factor-authentication-end-user-first-time.md) ainsi que les articles associés.

### <a name="where-to-create-and-delete-your-app-passwords"></a>Emplacements de création et de suppression de vos mots de passe d’application

Vous pouvez créer et supprimer des mots de passe d’application, selon la façon dont vous utilisez la vérification à deux facteurs :

- **Votre organisation utilise la vérification à deux facteurs et la page Vérification de sécurité supplémentaire.** Si vous utilisez votre compte professionnel ou scolaire (par exemple, alain@contoso.com) avec la vérification à deux facteurs dans votre organisation, vous pouvez gérer vos mots de passe d’application à partir de la [page Vérification de sécurité supplémentaire](https://account.activedirectory.windowsazure.com/Proofup.aspx). Pour obtenir des instructions détaillées, consultez [Créer et supprimer des mots de passe d’application à partir de la page Vérification de sécurité supplémentaire](#create-and-delete-app-passwords-from-the-additional-security-verification-page) dans cet article.

- **Votre organisation utilise la vérification à deux facteurs et le portail Office 365.** Si vous utilisez votre compte professionnel ou scolaire (par exemple, alain@contoso.com), la vérification à deux facteurs et des applications Microsoft 365 dans votre organisation, vous pouvez gérer vos mots de passe d’application à partir de la [page du portail Office 365](https://www.office.com). Pour obtenir des instructions détaillées, consultez [Créer et supprimer des mots de passe d’application à partir du portail Office 365](#create-and-delete-app-passwords-using-the-office-365-portal) dans cet article.

- **Vous utilisez la vérification à deux facteurs avec un compte Microsoft personnel.** Si vous utilisez un compte Microsoft personnel (comme alain@outlook.com) avec la vérification à deux facteurs, vous pouvez gérer vos mots de passe d’application à partir de la [page Security basics](https://account.microsoft.com/security/) (Fonctions de base de la sécurité). Pour obtenir des instructions détaillées, consultez [Utilisation de mots de passe d’application avec des applications qui ne prennent pas en charge la vérification en deux étapes](https://support.microsoft.com/help/12409/microsoft-account-app-passwords-and-two-step-verification).

## <a name="create-and-delete-app-passwords-from-the-additional-security-verification-page"></a>Créer et supprimer des mots de passe d’application à partir de la page Vérification de sécurité supplémentaire

Vous pouvez créer et supprimer des mots de passe d’application à partir de la page **Vérification de sécurité supplémentaire** pour votre compte professionnel ou scolaire.

1. Connectez-vous à la [page Vérification de sécurité supplémentaire](https://account.activedirectory.windowsazure.com/Proofup.aspx), puis sélectionnez **Mots de passe d’application**.

    ![Page Mots de passe d’application, avec l’onglet Mots de passe d’application mis en évidence](media/multi-factor-authentication-end-user-app-passwords/mfa-app-passwords-page.png)

2. Sélectionnez **Créer**, tapez le nom de l’application qui demande le mot de passe d’application, puis sélectionnez **Suivant**.

    ![Page Créer un mot de passe d’application où figure le nom de l’application qui nécessite un mot de passe](media/multi-factor-authentication-end-user-app-passwords/mfa-create-app-password-page.png)

3. Copiez le mot de passe à partir de la page **Votre mot de passe d’application**, puis sélectionnez **Fermer**.

    ![Page Votre mot de passe d’application contenant le mot de passe de l’application spécifiée](media/multi-factor-authentication-end-user-app-passwords/mfa-your-app-password-page.png)

4. Dans la page **Mots de passe d’application**, vérifiez que votre application est listée.

     ![Page Mots de passe d’application dans laquelle figure la nouvelle application](media/multi-factor-authentication-end-user-app-passwords/mfa-app-passwords-page-with-new-password.png)  

5. Ouvrez l’application pour laquelle vous avez créé le mot de passe d’application (en l’occurrence, Outlook 2010), puis collez le mot de passe d’application quand vous y êtes invité. Vous devez effectuer cette opération une seule fois par application.

### <a name="to-delete-an-app-password-using-the-app-passwords-page"></a>Pour supprimer un mot de passe d’application à partir de la page Mots de passe d’application

1. Dans la page **Mots de passe d’application**, sélectionnez **Supprimer** en regard du mot de passe d’application à supprimer.

   ![Supprimer un mot de passe d’application](media/multi-factor-authentication-end-user-app-passwords/mfa-app-passwords-page-delete.png)

2. Sélectionnez **Oui** pour confirmer la suppression du mot de passe, puis sélectionnez **Fermer**.

    Le mot de passe d’application est supprimé.

## <a name="create-and-delete-app-passwords-using-the-office-365-portal"></a>Créer et supprimer des mots de passe d’application à l’aide du Portail Office 365

Si vous utilisez la vérification en deux étapes avec votre compte professionnel ou scolaire et vos applications Microsoft 365, vous pouvez créer et supprimer des mots de passe d’application à l’aide du portail Office 365.

### <a name="to-create-app-passwords-using-the-office-365-portal"></a>Pour créer des mots de passe d’application à l’aide du Portail Office 365

1. Connectez-vous à votre compte professionnel ou scolaire, accédez à la [page Mon compte](https://portal.office.com), sélectionnez **Sécurité et confidentialité**, puis développez **Vérification de sécurité supplémentaire**.

    ![Portail Office sur lequel la zone Vérification de sécurité supplémentaire est développée](media/multi-factor-authentication-end-user-app-passwords/mfa-app-passwords-o365-my-account-page.png)

2. Sélectionnez le texte **Créer et gérer des mots de passe d’application** pour ouvrir la page **Mots de passe d’application**.

    ![Page Mots de passe d’application, avec l’onglet Mots de passe d’application mis en évidence](media/multi-factor-authentication-end-user-app-passwords/mfa-app-passwords-page.png)

3. Sélectionnez **Créer**, tapez le nom de l’application qui demande le mot de passe d’application, puis sélectionnez **Suivant**.

    ![Page Créer un mot de passe d’application où figure le nom de l’application qui nécessite un mot de passe](media/multi-factor-authentication-end-user-app-passwords/mfa-create-app-password-page.png)

4. Copiez le mot de passe à partir de la page **Votre mot de passe d’application**, puis sélectionnez **Fermer**.

    ![Page Votre mot de passe d’application contenant le mot de passe de l’application spécifiée](media/multi-factor-authentication-end-user-app-passwords/mfa-your-app-password-page.png)

5. Dans la page **Mots de passe d’application**, vérifiez que votre application est listée.

     ![Page Mots de passe d’application dans laquelle figure la nouvelle application](media/multi-factor-authentication-end-user-app-passwords/mfa-app-passwords-page-with-new-password.png)  

6. Ouvrez l’application pour laquelle vous avez créé le mot de passe d’application (en l’occurrence, Outlook 2010), puis collez le mot de passe d’application quand vous y êtes invité. Vous devez effectuer cette opération une seule fois par application.

### <a name="to-delete-app-passwords-using-the-app-passwords-page"></a>Pour supprimer des mots de passe d’application à partir de la page Mots de passe d’application

1. Dans la page **Mots de passe d’application**, sélectionnez **Supprimer** en regard du mot de passe d’application à supprimer.

   ![Supprimer un mot de passe d’application](media/multi-factor-authentication-end-user-app-passwords/mfa-app-passwords-page-delete.png)

2. Sélectionnez **Oui** dans la boîte de dialogue de confirmation, puis sélectionnez **Fermer**.

    Le mot de passe d’application est supprimé.

## <a name="if-your-app-passwords-arent-working-properly"></a>Si vos mots de passe d’application ne fonctionnent pas correctement

Vérifiez que vous avez correctement tapé le mot de passe. Si vous êtes sûr d’avoir entré le mot de passe correct, vous pouvez essayer de vous reconnecter et de créer un autre mot de passe d’application. Si aucune de ces options ne résout votre problème, contactez le support technique de votre organisation pour demander la suppression de vos mots de passe d’application existants, ce qui vous permettra d’en créer des nouveaux.

## <a name="next-steps"></a>Étapes suivantes

- [Gérer les paramètres de la vérification en deux étapes](multi-factor-authentication-end-user-manage-settings.md)

- Essayez l’[application Microsoft Authenticator](user-help-auth-app-download-install.md) pour vérifier vos connexions grâce aux notifications d’application, plutôt que de recevoir des SMS ou des appels.
