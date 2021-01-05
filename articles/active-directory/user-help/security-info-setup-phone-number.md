---
title: Configurer votre numéro de téléphone comme méthode de vérification – Azure AD
description: Guide pratique pour configurer votre page Informations de sécurité (préversion) et vérifier votre identité avec votre numéro de téléphone et votre appareil mobile comme méthode de vérification.
services: active-directory
author: curtand
manager: daveba
ms.reviewer: sahenry
ms.service: active-directory
ms.workload: identity
ms.subservice: user-help
ms.topic: end-user-help
ms.date: 02/13/2019
ms.author: curtand
ms.openlocfilehash: 7500b6b146a627dcdebacd9d2e2c7c61bc43c105
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "83744448"
---
# <a name="set-up-a-phone-number-as-your-verification-method"></a>Configurer un numéro de téléphone comme méthode de vérification

Suivez ces étapes pour ajouter vos méthodes de réinitialisation de mot de passe et de vérification en deux étapes. Au terme de la configuration initiale, vous pouvez revenir à la page **Informations de sécurité** pour ajouter, mettre à jour ou supprimer vos informations de sécurité.

Si vous êtes invité à les configurer immédiatement après vous être connecté à votre compte professionnel ou scolaire, reportez-vous aux étapes détaillées de l’article [Configurer vos informations de sécurité à partir de l’invite de la page de connexion](security-info-setup-signin.md).

[!INCLUDE [preview-notice](../../../includes/active-directory-end-user-preview-notice-security-info.md)]

> [!Note]
> Les informations de sécurité ne prennent pas en charge les numéros de poste. Même si vous ajoutez le bon format, +33 4 25 55 51 23-12345, les numéros de poste sont supprimés avant l’appel.
>
> Si vous ne voyez pas d'option téléphonique, cela peut indiquer que votre organisation ne vous autorise pas à utiliser cette option à des fins de vérification. Dans ce cas, vous devez choisir une autre méthode ou contacter le support technique de votre organisation pour obtenir de l'aide.

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

## <a name="set-up-phone-calls-from-the-security-info-page"></a>Configurer la méthode de l'appel téléphonique à partir de la page Informations de sécurité

Selon les paramètres de votre organisation, vous pourrez peut-être utiliser l'appel téléphonique comme méthode d'informations de sécurité.

>[!Note]
>Si vous préférez recevoir un SMS plutôt qu'un appel téléphonique, suivez les étapes de l'article [Configurer les informations de sécurité pour utiliser la messagerie texte (SMS)](security-info-setup-text-msg.md).

### <a name="to-set-up-phone-calls"></a>Pour configurer la méthode de l'appel téléphonique

1. Connectez-vous à votre compte professionnel ou scolaire, puis accédez à votre page https://myaccount.microsoft.com/.

    ![Page Mon profil, avec les liens des informations de sécurité mis en évidence](media/security-info/securityinfo-myprofile-page.png)

2. Sélectionnez **Informations de sécurité** à partir du volet de navigation de gauche ou du lien du bloc **Informations de sécurité**, puis sélectionnez **Ajouter une méthode** dans la page **Informations de sécurité**.

    ![Page Informations de sécurité, avec l'option Ajouter une méthode en surbrillance](media/security-info/securityinfo-myprofile-addmethod-page.png)

3. Sur la page **Ajouter une méthode**, choisissez **Téléphone** dans la liste déroulante, puis sélectionnez **Ajouter**.

    ![Zone Ajouter une méthode, avec l’option Téléphone sélectionnée](media/security-info/securityinfo-myprofile-addphonetext.png)

4. Sur la page **Téléphone**, entrez le numéro de téléphone de votre appareil mobile, choisissez **M'appeler**, puis sélectionnez **Suivant**.

    ![Ajouter le numéro de téléphone et choisir la méthode de l’appel téléphonique](media/security-info/securityinfo-myprofile-phonecall-addnumber.png)

5. Répondez à l'appel téléphonique de vérification envoyé au numéro de téléphone que vous avez entré, puis suivez les instructions.

    La page change pour indiquer que vous avez réussi.

    ![Notification de réussite, connectant le numéro de téléphone, le choix de recevoir un appel téléphonique et votre compte](media/security-info/securityinfo-myprofile-phonetext-success.png)

    Vos informations de sécurité sont mises à jour et vous pouvez opter pour un appel téléphonique afin de contrôler votre identité lorsque vous utilisez la vérification en deux étapes ou la réinitialisation du mot de passe. Si vous souhaitez que l'appel téléphonique constitue votre méthode par défaut, reportez-vous à la section [Changer de méthode d'informations de sécurité par défaut](#change-your-default-security-info-method) de cet article.

## <a name="delete-phone-calls-from-your-security-info-methods"></a>Supprimer l'appel téléphonique de vos méthodes d'informations de sécurité

Si vous ne souhaitez plus utiliser l'appel téléphonique comme méthode d'informations de sécurité, vous pouvez supprimer celle-ci de la page **Informations de sécurité**.

>[!Important]
>Si vous supprimez cette méthode par erreur, il n'existe aucun moyen d'annuler la suppression. Vous devrez à nouveau l'ajouter en suivant les étapes de la section [Configurer la méthode de l'appel téléphonique](#set-up-phone-calls-from-the-security-info-page) de cet article.

### <a name="to-delete-phone-calls"></a>Pour supprimer la méthode de l'appel téléphonique

1. Sur la page **Informations de sécurité**, sélectionnez le lien **Supprimer** en regard de l’option **Téléphone**.

    ![Lien permettant de supprimer la méthode Téléphone sur la page Informations de sécurité](media/security-info/securityinfo-myprofile-phonetext-delete.png)

2. Sélectionnez **Oui** dans la zone de confirmation pour supprimer le numéro de **téléphone**. Une fois votre numéro de téléphone supprimé, celui-ci disparaît de vos informations de sécurité et de la page **Informations de sécurité**. Si **Téléphone** est votre méthode par défaut, celle-ci est remplacée par une autre méthode disponible.

## <a name="change-your-default-security-info-method"></a>Changer de méthode d’informations de sécurité par défaut

Pour utiliser l'appel téléphonique comme méthode par défaut lorsque vous vous connectez à votre compte professionnel ou scolaire à l'aide de la vérification en deux étapes ou pour les demandes de réinitialisation de mot de passe, il vous suffit de définir cette méthode sur la page **Informations de sécurité**.

### <a name="to-change-your-default-security-info-method"></a>Pour changer de méthode d’informations de sécurité par défaut

1. Dans la page **Informations de sécurité**, sélectionnez le lien **Changer** en regard des informations relatives à la **Méthode de connexion par défaut**.

    ![Lien Changer pour la méthode de connexion par défaut](media/security-info/securityinfo-myprofile-phonetext-defaultchange.png)

2. Sélectionnez **Téléphone - Appeler (*_votre_numéro_de_téléphone_*)** dans la liste déroulante des méthodes disponibles, puis sélectionnez **Confirmer**.

    ![Choisir la méthode de connexion par défaut](media/security-info/securityinfo-myprofile-phonecall-changeddefault.png)

    La méthode par défaut utilisée pour la connexion devient **Téléphone - Appeler (*_votre_numéro_de_téléphone_*)** .

## <a name="additional-security-info-methods"></a>Autres méthodes d’informations de sécurité

Vous avez la possibilité de choisir la façon d’être contacté par votre organisation pour la vérification de votre identité, en fonction de ce que vous essayez de faire. Ces options sont les suivantes :

- **Application d’authentification.** Téléchargez et utilisez une application d’authentification pour obtenir une notification d’approbation ou un code d’approbation généré de manière aléatoire pour la réinitialisation du mot de passe ou la vérification en deux étapes. Pour obtenir des instructions détaillées sur la configuration et l’utilisation de l’application Microsoft Authenticator, consultez [Configurer les informations de sécurité pour utiliser une application d’authentification](security-info-setup-auth-app.md).

- **SMS sur appareil mobile.** Entrez votre numéro de téléphone mobile et recevez un code par SMS, à utiliser pour la vérification en deux étapes ou la réinitialisation de mot de passe. Pour obtenir des instructions détaillées sur la vérification de votre identité par SMS, consultez [Configurer les informations de sécurité pour utiliser la messagerie texte (SMS)](security-info-setup-text-msg.md).

- **Clé de sécurité.** Enregistrez votre clé de sécurité compatible Microsoft et utilisez-la avec un code PIN pour la vérification en deux étapes ou la réinitialisation du mot de passe. Pour obtenir des instructions détaillées sur la vérification de votre identité au moyen d’une clé de sécurité, consultez [Configurer des informations de sécurité pour utiliser une clé de sécurité](security-info-setup-security-key.md).

- **Adresse e-mail.** Entrez votre adresse e-mail professionnelle ou scolaire pour recevoir un e-mail de réinitialisation de mot de passe. Cette option n’est pas disponible pour la vérification en deux étapes. Pour des instructions pas à pas sur la façon de configurer votre e-mail, consultez [Configurer les informations de sécurité pour utiliser l’e-mail](security-info-setup-email.md).

- **Questions de sécurité.** Répondez à certaines questions de sécurité créées par votre administrateur pour votre organisation. Cette option est uniquement disponible pour la réinitialisation du mot de passe et non pour la vérification en deux étapes. Pour des instructions pas à pas sur la façon de configurer vos questions de sécurité, consultez l’article [Configurer les informations de sécurité pour utiliser les questions de sécurité](security-info-setup-questions.md).

    >[!Note]
    >Si certaines de ces options ne sont pas disponibles, il est très probable que votre organisation n’autorise pas ces méthodes. Si tel est le cas, vous devez choisir une autre méthode ou contacter votre administrateur pour obtenir de l’aide.

## <a name="next-steps"></a>Étapes suivantes

- Si vous avez perdu ou oublié votre mot de passe, réinitialisez-le à partir du [portail de réinitialisation de mot de passe](https://passwordreset.microsoftonline.com/), ou suivez les étapes de l’article [Réinitialiser votre mot de passe professionnel ou scolaire](active-directory-passwords-update-your-own-password.md).

- Obtenez des conseils de dépannage et de l’aide pour les problèmes de connexion en consultant l’article [Quand vous ne pouvez pas vous connecter à votre compte Microsoft](https://support.microsoft.com/help/12429/microsoft-account-sign-in-cant).
