---
title: Générer une application mobile qui appelle des API web | Azure
titleSuffix: Microsoft identity platform | Azure
description: Apprenez à générer une application mobile qui appelle des API web (vue d’ensemble)
services: active-directory
author: jmprieur
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/07/2019
ms.author: jmprieur
ms.reviewer: brandwe
ms.custom: aaddev, identityplatformtop40
ms.openlocfilehash: 853ab82ab40cc7ebee1c74feaf6aad87adf39ece
ms.sourcegitcommit: 6109f1d9f0acd8e5d1c1775bc9aa7c61ca076c45
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94443107"
---
# <a name="scenario-mobile-application-that-calls-web-apis"></a>Scénario : Application mobile appelant des API web

Découvrez comment générer une application mobile qui appelle des API web.

## <a name="getting-started"></a>Prise en main

Si ce n’est déjà fait, créez votre première application en effectuant un démarrage rapide :

- [Démarrage rapide : Acquérir un jeton et appeler l’API Microsoft Graph à partir d’une application Android](./quickstart-v2-android.md)
- [Démarrage rapide : Acquérir un jeton et appeler l’API Microsoft Graph à partir d’une application iOS](./quickstart-v2-ios.md)
- [Démarrage rapide : Acquérir un jeton et appeler l’API Microsoft Graph à partir d’une application Xamarin iOS et Android](https://github.com/Azure-Samples/active-directory-xamarin-native-v2) (GitHub)

## <a name="overview"></a>Vue d’ensemble

Une expérience utilisateur transparente et personnalisée est essentielle pour les applications mobiles.  La Plateforme d’identité Microsoft permet aux développeurs mobiles de créer cette expérience pour les utilisateurs iOS et Android. Votre application peut connecter des utilisateurs Azure Active Directory utilisateurs (Azure AD), des utilisateurs de compte Microsoft personnels et des utilisateurs Azure AD B2C. Elle peut également acquérir des jetons pour appeler une API web en leur nom. Pour implémenter ces flux, nous allons utiliser la Bibliothèque d’authentification Microsoft (MSAL). MSAL implémente la norme [Flux du code d’autorisation OAuth2.0 ](v2-oauth2-auth-code-flow.md).

![Applications démon](./media/scenarios/mobile-app.svg)

Éléments à prendre en considération pour les applications mobiles :

- **L’expérience utilisateur est essentielle** : Autorisez les utilisateurs à voir la valeur de votre application avant de demander la connexion. Demandez uniquement les autorisations requises.
- **Prise en charge de toutes les configurations utilisateur** : De nombreux utilisateurs professionnels mobiles doivent adhérer à des stratégies d’accès conditionnel et à des stratégies de conformité des appareils. Veillez à prendre en charge ces scénarios clés.
- **Implémenter l’authentification unique (SSO)** : La Bibliothèque d’authentification Microsoft (MSAL) et la Plateforme d’identités Microsoft vous permettent d’activer l’authentification unique via le navigateur de l’appareil ou Microsoft Authenticator (et le Portail d’entreprise Intune sur Android).
- **Implémenter le mode d’appareil partagé** : Autorisez l’utilisation de votre application dans des scénarios d’appareils partagés, par exemple, dans les hôpitaux, les usines, les magasins et la finance. [Découvrez-en plus sur la prise en charge du mode d’appareil partagé](msal-shared-devices.md).

## <a name="specifics"></a>Spécificités

Gardez les considérations suivantes à l’esprit lorsque vous générez une application mobile sur la Plateforme d’identités Microsoft :

- Selon la plateforme, une interaction de l’utilisateur peut être nécessaire la première fois que celui-ci se connecte. Par exemple, iOS exige des applications qu’elles affichent l’interaction de l’utilisateur lorsque ce dernier utilise l’authentification unique pour la première fois via Microsoft Authenticator (et le Portail d’entreprise Intune sur Android).
- Sur iOS et Android, la bibliothèque MSAL peut utiliser un navigateur externe pour connecter les utilisateurs. Le navigateur externe peut apparaître en haut de votre application.
- N’utilisez jamais de clé secrète dans une application mobile. Dans ces applications, les secrets sont accessibles à tous les utilisateurs.

## <a name="recommended-reading"></a>Lectures recommandées

[!INCLUDE [recommended-topics](../../../includes/active-directory-develop-scenarios-prerequisites.md)]

## <a name="next-steps"></a>Étapes suivantes

Passez à l’article suivant de ce scénario, [Inscription d’application](scenario-mobile-app-registration.md).
