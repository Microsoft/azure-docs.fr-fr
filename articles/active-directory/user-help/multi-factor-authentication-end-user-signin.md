---
title: Connexion à l’aide de l’authentification avec un compte professionnel ou scolaire-Azure AD
description: Découvrez comment vous connecter à votre compte professionnel ou scolaire avec les différentes méthodes de vérification à deux facteurs.
services: active-directory
author: curtand
manager: daveba
ms.assetid: b310b762-471b-4b26-887a-a321c9e81d46
ms.workload: identity
ms.service: active-directory
ms.subservice: user-help
ms.topic: end-user-help
ms.date: 04/02/2017
ms.author: curtand
ms.reviewer: librown
ms.custom: end-user, seo-update-azuread-jan
ms.openlocfilehash: 2f27dd7daf310e425b09db7905ad2f7c37fcff81
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94834164"
---
# <a name="sign-in-to-your-work-or-school-account-using-your-two-factor-verification-method"></a>Se connecter à un compte professionnel ou scolaire à l’aide de votre méthode de vérification à deux facteurs

> [!NOTE]
> Cet article vise à présenter de manière détaillée une expérience de connexion classique. Pour obtenir de l’aide concernant la connexion ou pour résoudre des problèmes, voir [Difficultés avec Azure AD Multi-Factor Authentication](multi-factor-authentication-end-user-troubleshoot.md).

## <a name="what-will-your-sign-in-experience-be"></a>Quelle sera votre expérience de connexion ?
Votre expérience d’authentification diffère selon ce que vous choisissez d’utiliser comme second facteur : un appel téléphonique, une application d’authentification ou des messages texte. Choisissez l’option qui décrit le mieux votre utilisation :

| Comment vous connectez-vous ? |
| --- |
| [Avec un appel téléphonique sur mon téléphone mobile ou de bureau](#signing-in-with-a-phone-call) |
| [Avec un message texte sur mon téléphone mobile](#signing-in-with-a-text-message)
| [Avec des notifications envoyées par l’application Microsoft Authenticator](#to-sign-in-with-a-notification-from-the-microsoft-authenticator-app) |
| Avec des codes de vérification envoyés par l’application Microsoft Authenticator |
| [Avec une autre méthode, car je ne peux pas utiliser ma méthode préférée à l’heure actuelle](#signing-in-with-an-alternate-method) |

## <a name="signing-in-with-a-phone-call"></a>Connexion avec un appel téléphonique
Les informations suivantes décrivent l’expérience de vérification en deux étapes avec un appel sur votre téléphone mobile ou de bureau.

1. Connectez-vous à une application ou un service comme Microsoft 365 à l’aide de votre nom d’utilisateur et de votre mot de passe.  
2. Microsoft vous appelle.  
3. Répondez au téléphone et appuyez sur la touche #.  

## <a name="signing-in-with-a-text-message"></a>Connexion avec un message texte
Les informations suivantes décrivent l’expérience de vérification en deux étapes avec un message texte sur votre téléphone mobile :

1. Connectez-vous à une application ou un service comme Microsoft 365 à l’aide de votre nom d’utilisateur et de votre mot de passe.
2. Microsoft vous envoie un message texte qui contient un code à chiffres.
3. Entrez le code dans la zone appropriée sur la page de connexion.

## <a name="signing-in-with-the-microsoft-authenticator-app"></a>Connexion avec l’application Microsoft Authenticator
Les informations suivantes décrivent l’utilisation de l’application Microsoft Authenticator pour les vérifications en deux étapes. Il existe deux façons différentes d’utiliser l’application. Vous pouvez recevoir des notifications Push sur votre appareil ou ouvrir l’application pour obtenir un code de vérification.

### <a name="to-sign-in-with-a-notification-from-the-microsoft-authenticator-app"></a>Pour vous connecter avec des notifications envoyées par l’application Microsoft Authenticator
1. Connectez-vous à une application ou un service comme Microsoft 365 à l’aide de votre nom d’utilisateur et de votre mot de passe.
2. Microsoft envoie une notification à l’application Microsoft Authenticator sur votre appareil.

   ![Microsoft envoie une notification](./media/multi-factor-authentication-end-user-signin/notify.png)

3. Ouvrez la notification sur votre téléphone, puis sélectionnez la clé **Vérifier**. Si votre entreprise requiert un code confidentiel, vous pouvez l’entrer ici.
4. Vous devez maintenant être connecté.

### <a name="to-sign-in-using-a-verification-code-with-the-microsoft-authenticator-app"></a>Pour vous connecter à l’application Microsoft Authenticator à l’aide d’un code de vérification

Si vous utilisez l’application Microsoft Authenticator pour obtenir des codes de vérification, lorsque vous ouvrez l’application vous voyez un nombre sous le nom de votre compte. Ce nombre change toutes les 30 secondes afin que vous n’utilisiez pas deux fois le même. Lorsque vous êtes invité à entrer un code de vérification, ouvrez l’application et utilisez le nombre qui est actuellement affiché.

1. Connectez-vous à une application ou un service comme Microsoft 365 à l’aide de votre nom d’utilisateur et de votre mot de passe.
2. Microsoft vous demande un code de vérification.

   ![Entrer le code de vérification](./media/multi-factor-authentication-end-user-signin/verify3.png)

3. Ouvrez l’application Microsoft Authenticator sur votre téléphone et entrez le code dans la zone où vous vous connectez.

## <a name="signing-in-with-an-alternate-method"></a>Connexion avec une autre méthode
Parfois, vous n’avez pas le téléphone ou l’appareil que vous avez configuré en tant que méthode de vérification par défaut avec vous. C’est la raison pour laquelle nous vous recommandons de définir des méthodes de sauvegarde pour votre compte. La section suivante vous montre comment vous connecter avec une autre méthode quand votre méthode principale n’est pas disponible.

1. Connectez-vous à une application ou un service comme Microsoft 365 à l’aide de votre nom d’utilisateur et de votre mot de passe.
2. Sélectionnez **Utiliser une autre option de vérification**. Vous voyez autant d’options de vérification que vous en avez configurées.
3. Choisissez une autre méthode et connectez-vous.

   ![Utiliser une autre méthode](./media/multi-factor-authentication-end-user-signin/alt.png)

## <a name="next-steps"></a>Étapes suivantes
- Si vous rencontrez des problèmes de connexion avec la vérification en deux étapes, obtenez plus d’informations sur la page [Difficultés avec Azure AD Multi-Factor Authentication](multi-factor-authentication-end-user-troubleshoot.md).

- Apprenez comment [gérer les paramètres de la vérification en deux étapes](multi-factor-authentication-end-user-manage-settings.md).

- Découvrez comment [prendre en main l’application Microsoft Authenticator](user-help-auth-app-download-install.md) afin que vous puissiez utiliser les notifications pour vous connecter, plutôt que des SMS et des appels téléphoniques.
