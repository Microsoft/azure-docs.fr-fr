---
title: Augmenter la résilience de l’authentification et de l’autorisation dans les applications démon que vous développez
titleSuffix: Microsoft identity platform
description: Conseils pour l’amélioration de la résilience de l’authentification et de l’autorisation dans une application démon à l’aide de la plateforme d’identités Microsoft
services: active-directory
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: how-to
author: knicholasa
ms.author: nichola
manager: martinco
ms.date: 11/23/2020
ms.openlocfilehash: 74bfc9eeeb8375fca2c88a3fd3c31f17e130fc99
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95919000"
---
# <a name="increase-the-resilience-of-authentication-and-authorization-in-daemon-applications-you-develop"></a>Augmenter la résilience de l’authentification et de l’autorisation dans les applications démon que vous développez

Cet article fournit des conseils sur la façon dont les développeurs peuvent utiliser la plateforme d’identités Microsoft et Azure Active Directory pour augmenter la résilience des applications démon. Cela comprend les processus d’arrière-plan, les services, les applications serveur à serveur et les applications sans utilisateurs.

![Application démon effectuant un appel vers Microsoft Identity](media/resilience-daemon-app/calling-microsoft-identity.png)

## <a name="use-managed-identities-for-azure-resources"></a>Utiliser des identités managées pour des ressources Azure

Les développeurs qui créent des applications démon sur Microsoft Azure peuvent utiliser des [identités managées pour ressources Azure](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview). Les identités managées éliminent la nécessité pour les développeurs de gérer les secrets et les informations d’identification. La fonctionnalité améliore la résilience en évitant les erreurs relatives à l’expiration des certificats, aux erreurs de rotation ou à la confiance. Elle offre également plusieurs fonctionnalités intégrées, spécifiquement conçues pour accroître la résilience.

Les identités managées utilisent des jetons d’accès de longue durée et des informations de Microsoft Identity afin d’acquérir de manière proactive de nouveaux jetons dans un délai important avant l’expiration du jeton existant. Votre application peut continuer à fonctionner pendant que vous essayez d’acquérir un nouveau jeton.

Les identités managées utilisent également des points de terminaison régionaux pour améliorer les performances et la résilience face aux défaillances hors région. L’utilisation d’un point de terminaison régional permet de maintenir l’ensemble du trafic à l’intérieur d’une zone géographique. Par exemple, si votre ressource Azure se trouve en WestUS2, tout le trafic, y compris le trafic généré par Microsoft Identity, doit rester en WestUS2. Cela permet d’éliminer les éventuels points de défaillance en consolidant les dépendances de votre service.

## <a name="use-the-microsoft-authentication-library"></a>Utiliser Microsoft Authentication Library

Les développeurs d’applications démon qui n’utilisent pas d’identités managées peuvent utiliser la [bibliothèque d’authentification Microsoft (MSAL)](https://docs.microsoft.com/azure/active-directory/develop/msal-overview), ce qui facilite l’implémentation de l’authentification et de l’autorisation et utilise automatiquement les meilleures pratiques en matière de résilience. MSAL facilitera le processus de fourniture des informations d’identification du client requises. Par exemple, votre application n’a pas besoin d’implémenter la création et la signature des assertions JSON Web Token lors de l’utilisation d’informations d’identification basées sur des certificats.

### <a name="use-microsoftidentityweb-for-net-developers"></a>Utiliser Microsoft.Identity.Web pour les développeurs .NET

Les développeurs qui créent des applications démon sur ASP.NET Core peuvent utiliser la bibliothèque [Microsoft.Identity.Web](https://docs.microsoft.com/azure/active-directory/develop/microsoft-identity-web). Cette bibliothèque repose sur MSAL pour faciliter l’implémentation de l’autorisation pour les applications ASP.NET Core. Elle comprend plusieurs stratégies de [cache de jetons distribués](https://github.com/AzureAD/microsoft-identity-web/wiki/token-cache-serialization#distributed-token-cache) pour les applications distribuées qui peuvent s’exécuter dans plusieurs régions.

## <a name="cache-and-store-tokens"></a>Cache et stockage de jetons

Si vous n’utilisez pas MSAL pour implémenter l’authentification et l’autorisation, vous pouvez implémenter quelques-unes des meilleures pratiques pour la mise en cache et le stockage des jetons. MSAL implémente et suit automatiquement ces meilleures pratiques.

Une application acquiert des jetons auprès d’un fournisseur d’identité pour autoriser l’application à appeler des API protégées. Lorsque votre application reçoit des jetons, la réponse qui contient les jetons contient également une propriété « expires\_in » qui indique à l’application la durée de mise en cache et de réutilisation du jeton. Il est important que les applications utilisent la propriété « expires\_in » pour déterminer la durée de vie du jeton. L’application ne doit jamais tenter de décoder un jeton d’accès d’API. L’utilisation du jeton mis en cache empêche tout trafic inutile entre votre application et Microsoft Identity. Votre utilisateur peut rester connecté à votre application pendant toute la durée de vie de ce jeton.

## <a name="properly-handle-service-responses"></a>Gérer correctement les réponses de service

Enfin, alors que les applications doivent gérer toutes les réponses d’erreur, certaines réponses peuvent avoir un impact sur la résilience. Si votre application reçoit un code de réponse HTTP 429, Trop de requêtes, Microsoft Identity limite vos demandes. Si votre application continue à faire trop de requêtes, elle continuera d’être limitée, ce qui empêchera votre application de recevoir des jetons. Votre application ne doit pas tenter d’acquérir à nouveau un jeton avant que le temps, en secondes, indiqué dans le champ de réponse Retry-After ne soit écoulé. Recevoir une réponse 429 est souvent une indication que l’application ne met pas en cache ni ne réutilise les jetons correctement. Les développeurs doivent examiner la façon dont les jetons sont mis en cache et réutilisés dans l’application.

Quand une application reçoit un code de réponse HTTP 5xx, l’application ne doit pas entrer dans une boucle de nouvelle tentative rapide. Le cas échéant, l’application doit respecter le même traitement « Retry-After », comme c’est le cas pour une réponse 429. Si aucun en-tête « Retry-After » n’est fourni par la réponse, nous vous recommandons d’implémenter une nouvelle tentative de backoff exponentiel avec la première tentative au moins cinq secondes après la réponse.

Quand une requête expire, les applications ne doivent pas faire de nouvelle tentative immédiatement. Implémentez une nouvelle tentative de backoff exponentiel avec la première tentative au moins cinq secondes après la réponse.

## <a name="next-steps"></a>Étapes suivantes

- [Renforcer la résilience des applications qui permettent aux utilisateurs de se connecter](resilience-client-app.md)
- [Renforcer la résilience de votre infrastructure de gestion des identités et des accès](resilience-in-infrastructure.md)
- [Renforcer la résilience de vos systèmes CIAM](resilience-b2c.md)
