---
title: Créer des mots de passe d’application à partir de la page Informations de sécurité (préversion) - Azure AD
description: Créez des mots de passe générés automatiquement (mots de passe d’application) à utiliser avec toute application sans navigateur ou toute application qui ne prend pas en charge la vérification à 2 facteurs dans votre organisation. Ce mot de passe d’application est différent d’un mot de passe normal et peut être configuré à partir de la page Informations de sécurité.
services: active-directory
author: curtand
manager: daveba
ms.reviewer: sahenry
ms.service: active-directory
ms.workload: identity
ms.subservice: user-help
ms.topic: end-user-help
ms.date: 02/13/2018
ms.author: curtand
ms.openlocfilehash: 84588130788e9be8d3be52a8ea0f3988dce7b952
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91537002"
---
# <a name="create-app-passwords-from-the-security-info-preview-page"></a>Créer des mots de passe d’application à partir de la page Informations de sécurité (préversion)

Certaines applications, telles qu’Outlook 2010, ne prennent pas en charge la vérification en deux étapes. Cela signifie que si vous utilisez la vérification en deux étapes dans votre organisation, l’application ne fonctionnera pas. Pour contourner ce problème, vous pouvez créer un mot de passe généré automatiquement, distinct de votre mot de passe normal, que vous devrez utiliser avec chaque application sans navigateur.

[!INCLUDE [preview-notice](../../../includes/active-directory-end-user-preview-notice-security-info.md)]

>[!Important]
>Votre administrateur peut ne pas vous autoriser à utiliser des mots de passe d’application. Si vous ne voyez pas l’option **Mots de passe d’application**, cela signifie qu’elle n’est pas disponible dans votre organisation.

Lorsque vous utilisez des mots de passe d’application, gardez à l’esprit les points suivants :

- Les mots de passe d’application sont générés automatiquement. Ils doivent être créés et entrés une fois par application.

- Un utilisateur peut posséder jusqu’à 40 mots de passe. Si vous essayez de créer un mot de passe supplémentaire au-delà de cette limite, vous serez invité à supprimer un mot de passe existant avant d’être autorisé à en créer un autre.

    >[!Note]
    >Les clients Office 2013 (y compris Outlook) prennent en charge de nouveaux protocoles d’authentification et peuvent être utilisés dans le cadre de la vérification en deux étapes. Cela signifie qu’après l’activation de la vérification en deux étapes, vous n’aurez plus besoin de mots de passe d’application pour les clients Office 2013. Pour plus d’informations, consultez l’article [Fonctionnement de l’authentification moderne pour les applications clientes Office 2013 et Office 2016](https://support.office.com/article/how-modern-authentication-works-for-office-2013-and-office-2016-client-apps-e4c45989-4b1a-462e-a81b-2a13191cf517).

## <a name="create-new-app-passwords"></a>Créer de nouveaux mots de passe d’application

Si vous utilisez la vérification en deux étapes avec votre compte professionnel ou scolaire et que votre administrateur a activé la fonctionnalité d’informations de sécurité, vous pouvez créer et supprimer des mots de passe d’application à l’aide de la page **Informations de sécurité**.

>[!Note]
>Si votre administrateur n’a pas activé la fonctionnalité d’informations de sécurité, vous devez suivre les instructions et les informations contenues dans la section [Gérer les mots de passe pour la vérification en deux étapes](multi-factor-authentication-end-user-app-passwords.md).

### <a name="to-create-a-new-app-password"></a>Pour créer un nouveau mot de passe d’application

1. Connectez-vous à votre compte professionnel ou scolaire, puis accédez à votre page https://myaccount.microsoft.com/.

    ![Page Mon profil, avec les liens des informations de sécurité mis en évidence](media/security-info/securityinfo-myprofile-page.png)

2. Sélectionnez **Informations de sécurité** à partir du volet de navigation de gauche ou du lien du bloc **Informations de sécurité**, puis sélectionnez **Ajouter une méthode** dans la page **Informations de sécurité**.

    ![Page Informations de sécurité, avec l'option Ajouter une méthode en surbrillance](media/security-info/securityinfo-myprofile-addmethod-page.png)

3. Sur la page **Ajouter une méthode**, choisissez **Mot de passe d’application** dans la liste déroulante, puis sélectionnez **Ajouter**.

    ![Zone Ajouter une méthode, avec l’option Mot de passe d’application sélectionnée](media/security-info/securityinfo-myprofile-addpassword.png)

4. Tapez le nom de l’application qui requiert le mot de passe, puis sélectionnez **Suivant**.

    ![Capture d’écran montrant la page « Mot de passe d’application » avec le nom de l’application entré.](media/security-info/securityinfo-myprofile-password-appname.png)

5. Copiez le texte à partir de la zone **Mot de passe**, collez le mot de passe dans la zone de mot de passe de l’application (dans cet exemple, Outlook 2010), puis sélectionnez **Terminé**.

    ![Page Mot de passe d’application, avec le nom de l’application](media/security-info/securityinfo-myprofile-password-copytext.png)

    Le mot de passe est ajouté et vous pourrez vous connecter correctement à votre application à l’avenir.

## <a name="delete-your-app-passwords"></a>Supprimer vos mots de passe d’application

Si vous n’avez plus besoin d’utiliser une application qui requiert un mot de passe d’application, vous pouvez supprimer le mot de passe d’application associé. Le fait de supprimer le mot de passe d’application libère l’un des emplacements de mot de passe d’application disponibles pour une utilisation ultérieure.

>[!Important]
>Si vous supprimez un mot de passe d’application par erreur, il n’existe aucun moyen d’annuler la suppression. Vous devrez créer un nouveau mot de passe d’application et l’entrer à nouveau dans l’application, en suivant les étapes décrites dans la section [Créer de nouveaux mots de passe d’application](#create-new-app-passwords) de cet article.

### <a name="to-delete-an-app-password"></a>Pour supprimer un mot de passe d’application

1. Sur la page **Informations de sécurité**, sélectionnez le lien **Supprimer** en regard de l’option **Mot de passe d’application** pour l’application spécifique.

    ![Lien permettant de supprimer la méthode Mot de passe d’application sur la page Informations de sécurité](media/security-info/securityinfo-myprofile-password-appdelete.png)

2. Sélectionnez **Oui** dans la zone de confirmation pour supprimer le **Mot de passe d’application**. Une fois le mot de passe d’application supprimé, celui-ci disparaît de vos informations de sécurité et de la page **Informations de sécurité**.

## <a name="for-more-information"></a>Informations supplémentaires

- Pour plus d’informations sur la page **Informations de sécurité** et sur sa configuration, consultez [Vue d’ensemble des informations de sécurité](./security-info-setup-signin.md)