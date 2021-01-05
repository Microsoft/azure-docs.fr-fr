---
title: Configurer une clé de sécurité comme méthode de vérification - Azure AD
description: Comment configurer votre page Informations de sécurité (préversion) pour vérifier votre identité au moyen d’une clé de sécurité Fast Identity Online (FIDO2) comme méthode de vérification.
services: active-directory
author: curtand
manager: daveba
ms.reviewer: librown
ms.service: active-directory
ms.workload: identity
ms.subservice: user-help
ms.topic: end-user-help
ms.date: 07/18/2019
ms.author: curtand
ms.openlocfilehash: c056e439deac71417ff14dcfc3f2c3c95db41946
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88797664"
---
# <a name="set-up-a-security-key-as-your-verification-method"></a>Configurer la clé de sécurité comme méthode de vérification

Vous pouvez utiliser des clés de sécurité comme méthode de connexion avec mot de passe dans votre organisation. Une clé de sécurité est un appareil physique qui est utilisé avec un code confidentiel unique pour se connecter à votre compte professionnel ou scolaire. Étant donné que les clés de sécurité nécessitent que vous disposiez de l’appareil physique et d’un nom que vous seul connaissez, il s’agit d’une méthode de vérification plus puissante qu’un nom d’utilisateur et un mot de passe.

[!INCLUDE [preview-notice](../../../includes/active-directory-end-user-preview-notice-security-key.md)]

>[!Note]
> Si vous ne voyez pas l’option de clé de sécurité, il est possible que votre organisation ne vous autorise pas à l’utiliser à des fins de vérification. Dans ce cas, vous devez choisir une autre méthode ou contacter le support technique de votre organisation pour obtenir de l’aide.

## <a name="security-verification-versus-password-reset-authentication"></a>Vérification de la sécurité ou authentification avec réinitialisation du mot de passe

Les méthodes relatives aux informations de sécurité sont utilisées pour la vérification de la sécurité à deux facteurs et la réinitialisation du mot de passe. Toutefois, ces techniques d’authentification ne prennent pas en charge toutes les méthodes.

| Méthode | Utilisé pour |
| ------ | -------- |
| Application Authenticator | Vérification à deux facteurs et réinitialisation de mot de passe. |
| SMS | Vérification à deux facteurs et réinitialisation de mot de passe. |
| Appels téléphoniques | Vérification à deux facteurs et réinitialisation de mot de passe. |
| Clé de sécurité | Vérification à deux facteurs et réinitialisation de mot de passe. |
| Compte e-mail | Authentification par réinitialisation de mot de passe uniquement. Vous devez choisir une autre méthode pour la vérification à deux facteurs. |
| Questions de sécurité | Authentification par réinitialisation de mot de passe uniquement. Vous devez choisir une autre méthode pour la vérification à deux facteurs. |

## <a name="what-is-a-security-key"></a>Qu’est-ce qu’une clé de sécurité ?

Nous prenons actuellement en charge plusieurs conceptions et fournisseurs de clés de sécurité qui utilisent les protocoles d’authentification sans mot de passe (FIDO2) [ Fast Indentity Online (FIDO)](https://fidoalliance.org/fido2/). Ces clés vous permettent de vous connecter à votre compte professionnel ou scolaire pour accéder à vos ressources informatiques d’organisation situées sur un support et un navigateur web pris en charge.

Votre administrateur ou votre organisation vous fournit une clé de sécurité si elle en a besoin pour votre compte professionnel ou scolaire. Il existe différents types de clés de sécurité que vous pouvez utiliser, par exemple une clé USB que vous connectez à votre appareil ou une clé NFC que vous connectez à un lecteur NFC. Vous pouvez obtenir plus d’informations sur votre clé de sécurité, notamment son type, dans la documentation du fabricant.

> [!Note]
> Si vous ne parvenez pas à utiliser une clé de sécurité FIDO2, vous pouvez utiliser d’autres méthodes de vérification par mot de passe, telles que l’application Microsoft Authenticator ou Windows Hello. Pour plus d’informations sur l’application Microsoft Authenticator, consultez [Qu’est-ce que l’application Microsoft Authenticator ?](user-help-auth-app-overview.md) Pour plus d’informations sur Windows Hello, consultez [Vue d’ensemble de Windows Hello](https://www.microsoft.com/windows/windows-hello).

## <a name="before-you-begin"></a>Avant de commencer

Avant de pouvoir inscrire votre clé de sécurité, les conditions suivantes doivent être remplies :

- Votre administrateur a activé cette fonctionnalité pour une utilisation au sein de votre organisation.

- Vous êtes sur un appareil qui a la mise à jour Windows du 10 mai 2019 et qui utilise un navigateur pris en charge.

- Vous avez reçu une clé de sécurité physique approuvée par votre administrateur ou par votre organisation. Votre clé de sécurité doit être à la fois conforme à FIDO2 et à Microsoft. Si vous avez des questions sur votre clé de sécurité et pour vérifier si elle est compatible, contactez le support technique de votre organisation.

## <a name="register-your-security-key"></a>Inscrire votre clé de sécurité

Vous devez créer votre clé de sécurité et lui associer un PIN unique avant de pouvoir vous connecter à votre compte professionnel ou scolaire via cette clé. Vous pouvez avoir jusqu’à 10 clés enregistrées avec votre compte. 

1. Accédez à la page **Mon profil** à l’adresse https://myaccount.microsoft.com et connectez-vous si vous ne l’avez pas déjà fait.

2. Sélectionnez **Informations de sécurité**, **Ajouter une méthode**, puis **Clé de sécurité** dans la liste déroulante **Ajouter une méthode**.

    ![Zone Ajouter une méthode, avec la clé de sécurité sélectionnée](media/security-info/security-info-security-key-add-method.png)

3. Sélectionnez **Ajouter**, puis le type de clé de sécurité dont vous disposez : **appareil USB** ou **appareil NFC**.

    ![Indiquer si vous disposez d’un type de clé de sécurité USB ou NFC](media/security-info/security-info-security-key-choose-type.png)

    > [!Note]
    > Si vous ne savez pas de quel type de clé de sécurité vous disposez, reportez-vous à la documentation du fabricant. Si vous n’êtes pas sûr du fabricant, contactez le support technique de votre organisation pour obtenir de l’aide.

4. Mettez votre clé de sécurité dans un emplacement facilement accessible, puis dans la zone **Clé de sécurité**, sélectionnez **Suivant**.

    ![Zone Démarrer l’inscription de la clé de sécurité](media/security-info/security-info-security-key-start-setup.png)

    Une nouvelle zone s’affiche pour vous guider dans la configuration de votre nouvelle méthode de connexion.

5. Dans la zone **Configuration de votre nouveau mode de connexion**, sélectionnez **Suivant**, puis :

    - Si votre clé de sécurité est un appareil USB, insérez-la dans le port USB de votre appareil.

    - Si votre clé de sécurité est un appareil NFC, connectez-la à votre lecteur.

6. Entrez le code confidentiel unique de votre clé de sécurité dans la zone **Sécurité Windows**, puis sélectionnez **OK**.

    Vous revenez alors à la zone **Configuration de votre nouveau mode de connexion**.

7. Sélectionnez **Suivant**.

8. Revenez à la page **Informations de sécurité**, tapez un nom que vous reconnaîtrez plus tard pour votre nouvelle clé de sécurité, puis sélectionnez **Suivant**.

    ![Page Informations de sécurité, nommer votre clé de sécurité](media/security-info/security-info-security-key-name.png)

    Votre clé de sécurité est inscrite et vous pouvez l’utiliser pour vous connecter à votre compte professionnel ou scolaire.

9. Sélectionnez **Terminé** pour fermer la zone **Clé de sécurité**.

    La page **Informations de sécurité** est mise à jour avec les informations de votre clé de sécurité.

    ![Page Informations de sécurité, avec toutes les méthodes inscrites affichées](media/security-info/security-info-security-key-configured.png)

## <a name="delete-a-security-key-from-your-security-info"></a>Supprimer une clé de sécurité de vos informations de sécurité

Si vous avez perdu votre clé de sécurité ou que vous ne souhaitez plus l’utiliser, vous pouvez la supprimer de vos informations de sécurité. Bien que cela empêche l’utilisation de la clé de sécurité avec votre compte professionnel ou scolaire, la clé de sécurité continue de stocker vos données et informations d’identification. Pour supprimer vos données et vos informations d’identification de la clé de sécurité elle-même, vous devez suivre les instructions de la section [Réinitialiser une clé de sécurité compatible Microsoft](#reset-your-security-key) dans cet article.

1. Sélectionnez le lien **Supprimer** de la clé de sécurité à supprimer.

2. Sélectionnez **OK** dans la zone **Supprimer la clé de sécurité**.

    Votre clé de sécurité est supprimée et vous ne pouvez plus l’utiliser pour vous connecter à votre compte professionnel ou scolaire.

>[!Important]
>Si vous avez supprimé cette clé de sécurité par erreur, vous devez l’enregistrer à nouveau en suivant les instructions de la section [Comment enregistrer une clé de sécurité](#register-your-security-key) dans cet article.

## <a name="manage-your-security-key-settings-from-windows-settings"></a>Gérer vos paramètres de clé de sécurité à partir des paramètres Windows

Vous pouvez gérer vos paramètres de clé de sécurité à partir de l’application **Paramètres Windows**, notamment la réinitialisation de votre clé de sécurité et la création d’un code confidentiel pour celle-ci.

### <a name="reset-your-security-key"></a>Réinitialiser votre clé de sécurité

Si vous souhaitez supprimer toutes les informations de compte stockées sur votre clé de sécurité physique, vous devez rétablir les paramètres d’usine de la clé. La réinitialisation de votre clé de sécurité supprime tous les éléments de la clé, ce qui vous permet de recommencer.

>[!IMPORTANT]
>La réinitialisation de votre clé de sécurité entraîne la suppression de tous les éléments de la clé et sa réinitialisation aux paramètres d’usine.
>
> **Toutes les données et toutes les informations d’identification sont effacées.**

#### <a name="to-reset-your-security-key"></a>Pour réinitialiser votre clé de sécurité

1. Ouvrez l’application Paramètres Windows, sélectionnez **Comptes**, **Options de connexion**, **Clé de sécurité**, puis **Gérer**.

2. Insérez votre clé de sécurité dans le port USB ou connectez votre lecteur NFC pour vérifier votre identité.

3. Suivez les instructions à l’écran en fonction du fabricant de votre clé de sécurité. Si le fabricant de votre clé n’est pas listé dans les instructions à l’écran, reportez-vous au site du fabricant pour plus d’informations.

4. Sélectionnez **Fermer** pour fermer l’écran **Gérer**.

### <a name="create-a-new-security-key-pin"></a>Créer un nouveau code confidentiel pour une clé de sécurité

Vous pouvez créer un nouveau code confidentiel pour votre clé de sécurité.

#### <a name="to-create-a-new-security-key-pin"></a>Pour créer un nouveau code confidentiel pour une clé de sécurité

1. Ouvrez l’application Paramètres Windows, sélectionnez **Comptes**, **Options de connexion**, **Clé de sécurité**, puis **Gérer**.

2. Insérez votre clé de sécurité dans le port USB ou connectez votre lecteur NFC pour vérifier votre identité.
3. Sélectionnez **Ajouter** dans la zone **Security Key PIN (Code confidentiel de la clé de sécurité)** , tapez et confirmez le nouveau code confidentiel de la clé de sécurité, puis sélectionnez **OK**.

     La clé de sécurité est mise à jour avec le nouveau code confidentiel de clé de sécurité à utiliser avec votre compte professionnel ou scolaire. Si vous décidez à nouveau de modifier votre code confidentiel, vous pouvez sélectionner le bouton **Modifier**.
4. Sélectionnez **Fermer** pour fermer l’écran **Gérer**.

## <a name="additional-security-info-methods"></a>Autres méthodes d’informations de sécurité

Pour enregistrer une clé de sécurité, vous devez avoir au moins une méthode de vérification de sécurité supplémentaire enregistrée. Pour plus d’informations, consultez la [Section vue d’ensemble](./security-info-setup-auth-app.md). 

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur les méthodes de vérification sans mot de passe, lisez le blog [Azure AD de Microsoft démarre une préversion publique des clés de sécurité FIDO2, en activant des connexions sans mot de passe](https://www.onmsft.com/news/microsofts-azure-ad-begins-public-preview-of-fido2-security-keys-enabling-passwordless-logins) ou lisez les articles [Qu’est-ce que l’application Microsoft Authenticator ?](user-help-auth-app-overview.md) et [Aperçu Windows Hello](https://www.microsoft.com/windows/windows-hello).

- Plus d’informations sur les [clés de sécurité conformes Microsoft](/windows/security/identity-protection/hello-for-business/microsoft-compatible-security-key).

- Si vous avez perdu ou oublié votre mot de passe, réinitialisez-le à partir du [portail de réinitialisation de mot de passe](https://passwordreset.microsoftonline.com/), ou suivez les étapes de l’article [Réinitialiser votre mot de passe professionnel ou scolaire](active-directory-passwords-update-your-own-password.md).

- Obtenez des conseils de dépannage et de l’aide pour les problèmes de connexion en consultant l’article [Quand vous ne pouvez pas vous connecter à votre compte Microsoft](https://support.microsoft.com/help/12429/microsoft-account-sign-in-cant).