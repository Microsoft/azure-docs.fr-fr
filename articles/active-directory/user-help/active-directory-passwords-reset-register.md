---
title: Inscrire des informations d’authentification pour réinitialiser son propre mot de passe – Azure AD
description: Inscrivez vos informations de méthode de vérification destinées à la réinitialisation de mot de passe en libre-service Azure AD pour pouvoir réinitialiser votre propre mot de passe sans l’aide d’un administrateur.
services: active-directory
author: curtand
manager: daveba
ms.reviewer: sahenry
ms.assetid: ''
ms.service: active-directory
ms.subservice: user-help
ms.workload: identity
ms.topic: end-user-help
ms.date: 05/28/2020
ms.author: curtand
ms.openlocfilehash: e460f3fd383286b1e61a979b87a9dfa09f272313
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88799024"
---
# <a name="register-your-verification-method-info-to-reset-your-own-password"></a>Inscrire vos informations de méthode de vérification pour réinitialiser votre mot de passe

Si vous avez oublié votre mot de passe professionnel ou scolaire, que votre organisation ne vous en a jamais envoyé un ou que votre compte a été verrouillé, vous pouvez utiliser vos informations de sécurité et votre appareil mobile pour le réinitialiser.

Votre administrateur doit activer cette fonctionnalité pour que vous puissiez enregistrer vos informations et réinitialiser votre mot de passe. Si l’option **J’ai oublié mon mot de passe** n’apparaît pas, cela signifie que votre administrateur n’a pas encore activé cette fonctionnalité pour votre organisation. Si vous pensez que ce n’est pas normal, contactez votre support technique pour obtenir de l’aide.

>[!Important]
>Cet article est destiné aux utilisateurs qui essaient d’utiliser une inscription pour la réinitialisation de mot de passe en libre-service. Cela signifie que vous serez en mesure de réinitialiser votre mot de passe professionnel ou scolaire (par exemple, alain@contoso.com) sans l’aide de votre administrateur. Si vous êtes administrateur et recherchez des informations sur la façon d’activer la réinitialisation de mot de passe en libre-service pour vos employés ou d’autres utilisateurs, voir [Déployer la réinitialisation de mot de passe en libre-service Azure AD et autres articles](../authentication/howto-sspr-deployment.md).

## <a name="set-up-your-password-reset-verification-method"></a>Configurer votre méthode de vérification de réinitialisation de mot de passe

1. Ouvrez le navigateur web sur votre appareil et accédez à la [page des informations de sécurité](https://account.activedirectory.windowsazure.com/PasswordReset/Register.aspx?regref=ssprsetup).

2. Selon la façon dont votre administrateur a configuré votre organisation, une ou plusieurs des options suivantes sont disponibles pour vous permettre de configurer votre méthode de vérification de sécurité. Si plusieurs options sont disponibles, nous vous recommandons vivement d’en utiliser plusieurs comme méthode de vérification de sécurité, au cas où l’une de vos méthodes deviendrait indisponible.

    - **Application d’authentification.** Choisissez d’utiliser l’application Microsoft Authenticator ou une autre application d’authentification comme méthode de vérification de sécurité. Pour plus d’informations sur la configuration de l’application, voir [Configurer l’application Microsoft Authenticator comme méthode de vérification](security-info-setup-auth-app.md).

    - **Messagerie texte.** Choisissez d’envoyer des SMS à votre appareil mobile. Pour plus d’informations sur la configuration de la messagerie texte, voir [Configurer l’envoi de SMS comme méthode de vérification](security-info-setup-text-msg.md).

    - **Appels téléphoniques.** Choisissez de recevoir un appel téléphonique sur votre numéro de téléphone inscrit. Pour plus d’informations sur la configuration des appels téléphoniques, voir [Configurer un numéro de téléphone comme méthode de vérification](security-info-setup-phone-number.md).

    - **Clé de sécurité.** Choisissez d’utiliser une clé de sécurité compatible avec Microsoft. Pour plus d’informations, voir [Configurer une clé de sécurité comme méthode de vérification](security-info-setup-security-key.md).

    - **Adresse e-mail.** Choisissez d’utiliser une autre adresse e-mail utilisable sans votre mot de passe oublié ou manquant. Cela ne fonctionne que pour la réinitialisation de mot de passe, pas comme méthode de vérification de sécurité. Pour plus d’informations sur la configuration d’une adresse e-mail, voir [Configurer une adresse e-mail comme méthode de vérification](security-info-setup-email.md).

    - **Questions de sécurité.** Choisissez de configurer et de répondre à des questions de sécurité prédéfinies formulées par votre administrateur. Cela ne fonctionne que pour la réinitialisation de mot de passe, pas comme méthode de vérification de sécurité. Pour plus d’informations sur les questions de sécurité, voir [Configurer des questions de sécurité comme méthode de vérification](security-info-setup-questions.md).

3. Après avoir sélectionné et configuré vos méthodes, choisissez **Terminer** pour achever le processus.

    > [!Note]
    > Les informations ajoutées pour votre numéro de téléphone ou votre adresse e-mail ne sont pas partagées avec l’annuaire global de votre organisation. Les seules personnes qui peuvent voir ces informations sont votre administrateur et vous-même. Vous êtes la seule personne à pouvoir voir les réponses à vos questions de sécurité.

## <a name="common-problems-and-their-solutions"></a>Problèmes courants et leurs solutions

 Voici quelques cas d'erreur courants et leurs solutions :

| Message d’erreur |  Solution possible |
| --- | --- | --- |
| Contactez votre administrateur.<br>Nous avons détecté que votre mot de passe de compte d'utilisateur n'est pas géré par Microsoft. Par conséquent, nous ne pouvons pas réinitialiser automatiquement votre mot de passe.<br>Contactez votre service informatique pour une aide supplémentaire.| Si vous recevez ce message d’erreur après avoir tapé votre ID d’utilisateur, cela signifie que votre organisation gère votre mot de passe en interne et ne veut pas que vous réinitialisiez votre mot de passe à partir du lien **Vous ne pouvez pas accéder à votre compte ?** . Pour réinitialiser votre mot de passe dans ce cas, vous devez contacter le support technique de votre organisation ou votre administrateur pour obtenir de l’aide. |
| Votre compte n’est pas activé pour la réinitialisation du mot de passe.<br>Nous sommes désolés, mais votre service informatique n’a pas configuré votre compte pour utiliser ce service.<br>Si vous le souhaitez, nous pouvons contacter un administrateur de votre organisation pour réinitialiser le mot de passe à votre place. | Si vous recevez ce message d’erreur après avoir tapé votre ID d’utilisateur, cela signifie que votre organisation n’a pas activé la fonctionnalité de réinitialisation de mot de passe ou que vous n’êtes pas autorisé à l’utiliser. Pour réinitialiser votre mot de passe dans ce cas, vous devez sélectionner le lien **Contacter un administrateur**. Une fois que vous avez cliqué sur le lien, un message électronique est envoyé au support technique ou à l’administrateur de votre organisation, les informant que vous souhaitez réinitialiser votre mot de passe. |
| Nous n'avons pas pu vérifier votre compte.<br>Si vous le souhaitez, nous pouvons contacter un administrateur de votre organisation pour réinitialiser le mot de passe à votre place. | Si vous recevez ce message d’erreur après avoir tapé votre ID d’utilisateur, cela signifie que votre organisation a activé la réinitialisation de mot de passe et que vous pouvez l’utiliser, mais que vous ne vous êtes pas inscrit au service. Dans ce cas, vous devez contacter le support technique ou l’administrateur de votre organisation pour réinitialiser votre mot de passe. Pour plus d’informations sur l’inscription à la réinitialisation de mot de passe après retour sur votre appareil, voir le processus décrit plus haut dans cet article. |

## <a name="next-steps"></a>Étapes suivantes

- [Changer votre mot de passe avec la réinitialisation de mot de passe en libre-service](active-directory-passwords-update-your-own-password.md)

- [Page d’informations de sécurité](https://mysignins.microsoft.com/security-info)

- [Portail de réinitialisation du mot de passe](https://passwordreset.microsoftonline.com/)

- [Quand vous ne pouvez pas vous connecter à votre compte Microsoft](https://support.microsoft.com/help/12429/microsoft-account-sign-in-cant)