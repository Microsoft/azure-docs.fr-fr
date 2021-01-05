---
title: Réinitialiser votre mot de passe à l’aide des informations de sécurité - Azure Active Directory | Microsoft Docs
description: Comment réinitialiser votre mot de passe si vous l’oubliez, à l’aide des informations de sécurité et d’une vérification en deux étapes.
services: active-directory
author: curtand
manager: daveba
ms.reviewer: sahenry
ms.service: active-directory
ms.workload: identity
ms.subservice: user-help
ms.topic: end-user-help
ms.date: 09/10/2020
ms.author: curtand
ms.openlocfilehash: c47d573dab3e72a956f6f0b9d09a1dafadf0225c
ms.sourcegitcommit: d2222681e14700bdd65baef97de223fa91c22c55
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/07/2020
ms.locfileid: "91827766"
---
# <a name="reset-your-work-or-school-password-using-security-info"></a>Réinitialiser votre mot de passe professionnel ou scolaire avec des informations de sécurité

Si vous avez oublié votre mot de passe professionnel ou scolaire, que votre organisation ne vous en a jamais envoyé un ou que votre compte a été verrouillé, vous pouvez utiliser vos informations de sécurité et votre appareil mobile pour le réinitialiser. Votre administrateur doit activer cette fonctionnalité pour vous permettre de configurer vos informations et [réinitialiser votre propre mot de passe](./active-directory-passwords-reset-register.md).

Si vous connaissez votre mot de passe et que vous voulez le changer, consultez les étapes dans [Comment modifier votre mot de passe](#how-to-change-your-password) de cet article.

>[!Important]
>Cet article s’adresse aux utilisateurs qui tentent de réinitialiser un mot de passe inconnu ou oublié d’un compte professionnel ou scolaire. Si vous êtes administrateur et que vous recherchez des informations sur la façon d’activer la réinitialisation de mot de passe en libre-service pour vos employés ou d’autres utilisateurs, consultez [Déployer la réinitialisation de mot de passe en libre-service Azure AD et autres articles](../authentication/howto-sspr-deployment.md).

## <a name="how-to-reset-or-unlock-your-password-for-a-work-or-school-account"></a>Comment réinitialiser ou déverrouiller le mot de passe d’un compte professionnel ou scolaire

Vous n’avez peut-être pas accès à votre compte Azure Active Directory (Azure AD) pour l’une des raisons suivantes :

- Votre mot de passe ne fonctionne pas et vous souhaitez le réinitialiser, ou

- Vous connaissez votre mot de passe, mais votre compte est verrouillé et vous souhaitez le déverrouiller.

### <a name="to-reset-your-password-and-get-back-into-your-account"></a>Pour réinitialiser votre mot de passe et vous reconnecter à votre compte

1. Dans l’écran de **saisie du mot de passe**, sélectionnez **Mot de passe oublié**.

2. Dans l’écran **Retournez sur votre compte**, saisissez votre **identifiant utilisateur** professionnel ou scolaire (par exemple, votre adresse e-mail), prouvez que vous n’êtes pas un robot en entrant les caractères visibles à l’écran, puis sélectionnez **Suivant**.

   ![Écran Retournez sur votre compte](media/security-info/security-info-back-into-acct.png)

   >[!NOTE]
   >Si votre administrateur n’a pas encore activé la possibilité de réinitialiser votre mot de passe, vous verrez un lien **Contactez votre administrateur** à la place de l’écran **Retournez sur votre compte**. Ce lien vous permet de contacter votre administrateur pour réinitialiser votre mot de passe, que ce soit par e-mail ou un portail web.

3. Choisissez l’une des méthodes suivantes pour vérifier votre identité et modifier votre mot de passe. Selon la façon dont votre administrateur a configuré votre organisation, vous devrez peut-être exécuter ce processus une deuxième fois et entrer des informations supplémentaires pour une deuxième étape de vérification.

    ![Retournez sur votre compte, étape de vérification n°1](media/security-info/security-info-back-into-acct2.png)

    >[!NOTE]
    >Selon la façon dont votre administrateur a configuré votre organisation, certaines de ces options de vérification peuvent ne pas être disponibles. Pour la vérification à l’aide d’au moins une des méthodes suivantes, vous devez avoir préalablement configuré votre appareil mobile.<br><br>En outre, il est possible que votre nouveau mot de passe doive répondre à certaines exigences de sécurité. Les mots de passe forts présentent généralement de 8 à 16 caractères, et contiennent des caractères majuscules et minuscules, au moins un chiffre et au moins un caractère spécial.

- **Réinitialiser votre mot de passe à l’aide d’une adresse e-mail.** Envoie un e-mail à l’adresse de messagerie que vous avez précédemment configurée lors de la vérification en deux étapes ou dans les informations de sécurité. Si votre administrateur a activé l’expérience informations de sécurité, vous trouverez plus d’informations sur la configuration d’une adresse e-mail dans l’article [Set up security info to use email (preview)](security-info-setup-email.md) (Configurer les informations de sécurité pour utiliser l’e-mail [préversion]). Si vous n’utilisez pas encore les informations de sécurité, vous trouverez plus d’informations sur la configuration d’une adresse e-mail dans l’article [Configurer mon compte pour la vérification en deux étapes](multi-factor-authentication-end-user-first-time.md). 

    1. Sélectionnez **Envoyer un e-mail sur mon adresse de messagerie de secours**, puis sélectionnez **E-mail**.

    2. Saisissez le code de vérification obtenu par e-mail dans la zone de texte, puis sélectionnez **Suivant**.

    3. Saisissez et confirmez votre nouveau mot de passe, puis sélectionnez **Terminer**.

- **Réinitialiser votre mot de passe par SMS.** Envoie un SMS au numéro de téléphone précédemment configuré dans les informations de sécurité. Si votre administrateur a activé l’expérience informations de sécurité, vous trouverez plus d’informations sur la configuration de l’envoie de SMS dans l’article [Set up security info to use text messaging (preview)](security-info-setup-text-msg.md) (Configurer les informations de sécurité pour utiliser les SMS [préversion]). Si vous n’utilisez pas encore les informations de sécurité, vous trouverez plus d’informations sur la configuration de l’envoi de SMS dans l’article [Configurer mon compte pour la vérification en deux étapes](multi-factor-authentication-end-user-first-time.md).

    1. Sélectionnez **Envoyer un SMS sur mon téléphone portable**, saisissez votre numéro de téléphone, puis sélectionnez **Envoyer un SMS**.

    2. Saisissez le code de vérification obtenu par SMS dans la zone de texte, puis sélectionnez **Suivant**.

    3. Saisissez et confirmez votre nouveau mot de passe, puis sélectionnez **Terminer**.

- **Réinitialiser votre mot de passe par téléphone.** Passe un appel vocal automatique au numéro de téléphone précédemment configuré dans les informations de sécurité. Si votre administrateur a activé la fonctionnalité Informations de sécurité, vous trouverez plus d’informations sur la configuration d’un numéro de téléphone dans l’article [Set up security info to use a phone call (preview)](security-info-setup-phone-number.md) (Configurer les informations de sécurité pour utiliser l’appel téléphonique [préversion]). Si vous n’utilisez pas encore les informations de sécurité, vous trouverez plus d’informations sur la configuration de l’appel téléphonique dans l’article [Configurer mon compte pour la vérification en deux étapes](multi-factor-authentication-end-user-first-time.md).

    1. Sélectionnez **Effectuer un appel sur mon téléphone portable**, saisissez votre numéro de téléphone, puis sélectionnez **Appeler**.

    2. Répondez à l’appel téléphonique et suivez les instructions pour vérifier votre identité, puis sélectionnez **Suivant**.

    3. Saisissez et confirmez votre nouveau mot de passe, puis sélectionnez **Terminer**.

- **Réinitialiser votre mot de passe à l’aide de questions de sécurité.** Affiche la liste des questions de sécurité que vous avez configurées dans les informations de sécurité. Si votre administrateur a activé la fonctionnalité Informations de sécurité, vous trouverez plus d’informations sur la configuration des questions de sécurité dans l’article [Set up security info to use pre-defined security questions (preview)](security-info-setup-questions.md) (Configurer les informations de sécurité pour utiliser les questions de sécurité prédéfinies [préversion]). Si vous n’utilisez pas encore les informations de sécurité, vous trouverez plus d’informations sur la configuration des questions de sécurité dans l’article [Configurer mon compte pour la vérification en deux étapes](multi-factor-authentication-end-user-first-time.md).

    1. Sélectionnez **Répondre à ma question de sécurité**, répondez aux questions, puis sélectionnez **Suivant**.

    2. Saisissez et confirmez votre nouveau mot de passe, puis sélectionnez **Terminer**.

- **Réinitialiser votre mot de passe à l’aide d’une notification à partir de votre application d’authentification.** Envoie une notification d’approbation à l’application d’authentification. Si votre administrateur a activé la fonctionnalité Informations de sécurité, vous trouverez plus d’informations sur la configuration de l’envoi de notification par une application d’authentification dans l’article [Set up security info to use an authentication app (preview)](security-info-setup-auth-app.md) (Configurer les informations de sécurité pour utiliser l’application d’authentification [préversion]). Si vous n’utilisez pas encore les informations de sécurité, vous trouverez plus d’informations sur la configuration de l’envoi de notification par une application d’authentification dans l’article [Configurer mon compte pour la vérification en deux étapes](multi-factor-authentication-end-user-first-time.md).

    1. Sélectionnez **Approve a notification on my authenticator app** (Valider une notification sur mon application d’authentification), puis sélectionnez **Send Notification** (Envoyer une notification).

    2. Approuvez la connexion depuis votre application d’authentification.

    3. Saisissez et confirmez votre nouveau mot de passe, puis sélectionnez **Terminer**.

- **Réinitialiser votre mot de passe à l’aide d’un code de votre application d’authentification.** Accepte un code aléatoire fourni par votre application d’authentification. Si votre administrateur a activé la fonctionnalité Informations de sécurité, vous trouverez plus d’informations sur la configuration de l’envoi d’un code par une application d’authentification dans l’article [Set up security info to use an authentication app (preview)](security-info-setup-auth-app.md) (Configurer les informations de sécurité pour utiliser l’application d’authentification [préversion]). Si vous n’utilisez pas encore les informations de sécurité, vous trouverez plus d’informations sur la configuration de l’envoi de code par une application d’authentification dans l’article [Configurer mon compte pour la vérification en deux étapes](multi-factor-authentication-end-user-first-time.md).

  1. Sélectionnez **Enter a code from my authenticator app** (Saisir un code provenant de mon application d’authentification), puis sélectionnez **Send Notification** (Envoyer une notification).

  2. Ouvrez votre application d’authentification, saisissez le code de vérification de votre compte dans la zone de texte, puis sélectionnez **Suivant**.

  3. Saisissez et confirmez votre nouveau mot de passe, puis sélectionnez **Terminer**.

  4. Une fois que vous obtenez le message indiquant que votre mot de passe a été réinitialisé, connectez-vous à votre compte à l’aide de votre nouveau mot de passe.

     Si vous ne pouvez toujours pas accéder à votre compte, contactez l’administrateur de votre organisation pour obtenir de l’aide.

Après la réinitialisation du mot de passe, vous pouvez recevoir un e-mail de confirmation provenant d’un compte tel que « Microsoft de la part de \<*your_organization*> ». Si vous recevez un e-mail de ce type sans avoir récemment réinitialisé votre mot de passe, contactez immédiatement l’administrateur de votre organisation.

## <a name="how-to-change-your-password"></a>Comment modifier votre mot de passe

Si vous souhaitez simplement modifier votre mot de passe, vous pouvez le faire par le biais du portail Office 365, du portail Mes applications ou de la page de connexion Windows 10.

### <a name="to-change-your-password-using-the-office-365-portal"></a>Pour modifier votre mot de passe à partir du portail Office 365

Utilisez cette méthode si vous avez l’habitude d’accéder à vos applications à l’aide du portail Office :

1. Connectez-vous à votre [compte Office 365](https://portal.office.com) à l’aide de votre mot de passe existant.

2. Sélectionnez votre profil en haut à droite de la fenêtre, puis sélectionnez **Afficher le compte**.

3. Sélectionnez **Sécurité et confidentialité** > **Mot de passe**.

4. Saisissez votre ancien mot de passe, créez un nouveau mot de passe et confirmez-le, puis cliquez sur **Soumettre**.

### <a name="to-change-your-password-from-the-my-apps-portal"></a>Pour modifier votre mot de passe à partir du portail Mes applications

Appliquez cette méthode si vous avez l’habitude d’accéder à vos applications à partir du portail Mes applications :

1. Connectez-vous au [portail Mes applications](https://myapps.microsoft.com/) en utilisant votre mot de passe existant.

2. Sélectionnez votre profil en haut à droite de la fenêtre, puis sélectionnez **Profil**.

3. Sélectionnez **Modifier le mot de passe**.

4. Saisissez votre ancien mot de passe, créez un nouveau mot de passe et confirmez-le, puis cliquez sur **Soumettre**.

### <a name="to-change-your-password-at-windows-sign-in"></a>Pour modifier votre mot de passe lors de la connexion à Windows

Si votre administrateur a activé la fonctionnalité, vous verrez un lien de **réinitialisation du mot de passe** sur votre écran d’authentification à Windows 7, Windows 8, Windows 8.1 ou Windows 10.

1. Sélectionnez le lien de **réinitialisation du mot de passe** pour lancer le processus de réinitialisation de mot de passe sans avoir à utiliser la fonctionnalité web habituelle.

2. Vérifiez votre nom d’utilisateur et sélectionnez **Suivant**.

3. Sélectionnez la méthode de contact à utiliser pour la vérification et confirmez. Si nécessaire, choisissez une deuxième option de vérification différente de la précédente, et renseignez les informations nécessaires.

4. Sur la page **Créer un mot de passe**, entrez un nouveau mot de passe, confirmez-le, puis sélectionnez **Suivant**.

    Les mots de passe forts présentent généralement de 8 à 16 caractères, et contiennent des caractères majuscules et minuscules, au moins un chiffre et au moins un caractère spécial.

5. Une fois que vous obtenez le message indiquant que votre mot de passe a été réinitialisé, sélectionnez **Terminer**.

    Si vous ne pouvez toujours pas accéder à votre compte, contactez l’administrateur de votre organisation pour obtenir de l’aide.

## <a name="common-problems-and-their-solutions"></a>Problèmes courants et leurs solutions

Voici quelques cas d'erreur courants et leurs solutions :

|Problème|Description|Solution|
| --- | --- | --- |
|J’obtiens une erreur quand j’essaie de changer mon mot de passe. |Votre mot de passe contient un mot, une expression ou un modèle qui rend votre mot de passe facile à deviner.| Réessayez en utilisant un mot de passe plus fort.|
|Après avoir entré mon identifiant utilisateur, j’accède à une page indiquant « Veuillez contacter votre administrateur ».|Microsoft a déterminé que votre mot de passe de compte d’utilisateur est géré par votre administrateur dans un environnement local. De ce fait, vous ne pouvez pas réinitialiser votre mot de passe à partir du lien « Vous ne pouvez pas accéder à votre compte ». |Contactez votre administrateur pour obtenir de l’aide.|
|L’erreur « Votre compte n’est pas activé pour la réinitialisation de mot de passe » s’affiche lorsque j’entre mon identifiant utilisateur.|Votre administrateur n’a pas encore configuré votre compte pour vous permettre de réinitialiser votre mot de passe.|Votre administrateur n’a pas activé la réinitialisation de mot de passe pour votre organisation à partir du lien « Votre compte n’est pas accessible » ou ne vous a pas octroyé de licence vous permettant d’utiliser la fonctionnalité.<br><br> Pour réinitialiser votre mot de passe, vous devez sélectionner le lien « Contacter un administrateur » afin d’envoyer un e-mail à l’administrateur de votre entreprise et l’informer que vous souhaitez réinitialiser votre mot de passe.|
|Après avoir saisi mon identifiant utilisateur, j’obtiens une erreur indiquant « Nous n’avons pas pu vérifier votre compte ».|Le processus d’authentification n’a pas pu vérifier les informations liées à votre compte.|Si vous obtenez ce message, deux raisons possibles :<br><br>1. Votre administrateur a activé la réinitialisation de mot de passe pour votre organisation, mais vous n’êtes pas inscrit pour utiliser le service. Pour vous inscrire à la réinitialisation du mot de passe, consultez l’un des articles suivants, selon votre méthode de vérification : [Configurer les informations de sécurité pour utiliser une application d’authentification (préversion) ](security-info-setup-auth-app.md), [Configurer les informations de sécurité pour utiliser un appel téléphonique (préversion)](security-info-setup-phone-number.md), [Configurer les informations de sécurité pour utiliser l’envoi de SMS (préversion)](security-info-setup-text-msg.md), [Configurer les informations de sécurité pour utiliser les e-mails (préversion)](security-info-setup-email.md) ou [Configurer les informations de sécurité pour utiliser les questions de sécurité (préversion)](security-info-setup-questions.md).<br><br>2. Votre administrateur n’a pas activé la réinitialisation de mot de passe pour votre organisation. Dans ce cas, sélectionnez le lien « Contacter un administrateur » afin d’envoyer un e-mail au service informatique de votre entreprise demandant de réinitialiser votre mot de passe.|

## <a name="next-steps"></a>Étapes suivantes

- Découvrez les informations de sécurité avec l’article [Security info (preview) overview](./security-info-setup-signin.md) (Vue d’ensemble des informations de sécurité [préversion]).

- Si vous tentez de récupérer un compte personnel comme Xbox, hotmail.com ou outlook.com, essayez les suggestions fournies dans l’article [Quand vous ne pouvez pas vous connecter à votre compte Microsoft](https://support.microsoft.com/help/12429/microsoft-account-sign-in-cant).
