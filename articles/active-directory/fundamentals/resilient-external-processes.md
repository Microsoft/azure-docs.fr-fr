---
title: Interfaces résilientes avec des processus externes utilisant Azure AD B2C | Microsoft Docs
description: Méthodes pour créer des interfaces résilientes avec des processus externes
services: active-directory
ms.service: active-directory
ms.subservice: fundamentals
ms.workload: identity
ms.topic: how-to
author: gargi-sinha
ms.author: gasinh
manager: martinco
ms.reviewer: ''
ms.date: 11/30/2020
ms.custom: it-pro
ms.collection: M365-identity-device-management
ms.openlocfilehash: c96856c988cae891e64ddf460d61851102e4666c
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95918970"
---
# <a name="resilient-interfaces-with-external-processes"></a>Interfaces résilientes avec processus externes

Dans cet article, nous vous proposons des conseils sur la façon de planifier et d’implémenter les API RESTFul dans le parcours utilisateur et de rendre votre application plus résiliente vis-à-vis des défaillances de l’API.

![Image montrant les interfaces avec des composants de processus externes](media/resilient-external-processes/external-processes-architecture.png)

## <a name="ensure-correct-placement-of-the-apis"></a>Garantir le positionnement correct des API

Les stratégies Identity Experience Framework (IEF) vous permettent d’appeler un système externe à l’aide d’un [profil technique de l’API RESTful](https://docs.microsoft.com/azure/active-directory-b2c/restful-technical-profile). Les systèmes externes ne sont pas contrôlés par l’environnement du runtime IEF et constituent un possible point de défaillance.

### <a name="how-to-manage-external-systems-using-apis"></a>Gestion des systèmes externes à l’aide d’API

- Lorsque vous appelez une interface pour accéder à certaines données, vérifiez si les données vont motiver la décision d’authentification. Évaluez si les informations sont essentielles pour les fonctionnalités principales de l’application. Par exemple, un e-commerce par rapport à une fonctionnalité secondaire comme une administration. Si les informations ne sont pas nécessaires à l’authentification et ne sont requises que pour les scénarios secondaires, envisagez de déplacer l’appel vers la logique d’application.

- Si les données nécessaires pour l’authentification sont relativement statiques et peu volumineuses et qu’elles n’ont aucune autre raison commerciale d’être externalisées du répertoire, envisagez de les faire figurer dans le répertoire.

- Si possible, supprimez les appels d’API du chemin d’accès préauthentifié. Si vous ne le pouvez pas, vous devez mettre en place des protections strictes contre les attaques par déni de service (DoS) et par déni de service distribué (DDoS) devant vos API. Les attaquants peuvent charger la page de connexion et tenter de saturer votre API avec des attaques DoS et paralyser votre application. Par exemple, l’utilisation de CAPTCHA dans votre flux de connexion ou d’inscription peut vous aider.

- Utilisez les [connecteurs d’API du flux d’utilisateurs intégré à l’inscription](https://docs.microsoft.com/azure/active-directory-b2c/api-connectors-overview) chaque fois que cela est possible pour intégrer les API web, soit après l’inscription auprès d’un fournisseur d’identité, soit avant la création de l’utilisateur. Comme les flux d’utilisateurs sont déjà largement testés, il est probable que vous n’ayez pas à effectuer de tests fonctionnels, de performance ou d’échelle au niveau des flux d’utilisateurs. Vous devez toujours tester vos applications sur le plan des fonctionnalités, des performances et de la mise à l’échelle.

- Les [profils techniques](https://docs.microsoft.com/azure/active-directory-b2c/restful-technical-profile) des API RESTFul Azure AD ne fournissent aucun comportement de mise en cache. Au lieu de cela, le profil d’API RESTFul implémente une logique de nouvelle tentative et un délai d’expiration intégré à la stratégie.

- Pour les API qui ont besoin d’écrire des données, mettez en file d’attente une tâche pour que ces tâches soient exécutées par un Worker en arrière-plan. Vous pouvez utiliser des services tels que [Files d’attente Azure](https://docs.microsoft.com/azure/storage/queues/storage-queues-introduction). Ainsi, l’API retourne efficacement, ce qui augmentera les performances d’exécution de la stratégie.  

## <a name="api-error-handling"></a>Gestion des erreurs d’API

Comme les API se trouvent en dehors du système Azure AD B2C, il est nécessaire de disposer d’une gestion des erreurs appropriée au sein du profil technique. Assurez-vous que l’utilisateur final est informé de manière appropriée et que l’application peut traiter les échecs correctement.

### <a name="how-to-gracefully-handle-api-errors"></a>Gestion efficace des erreurs d’API

- Une API pouvant échouer pour diverses raisons, faites en sorte que votre application soit résiliente face à de tels échecs. [Renvoyez un message d’erreur HTTP 4XX](https://docs.microsoft.com/azure/active-directory-b2c/restful-technical-profile#returning-validation-error-message) si l’API ne parvient pas à terminer la requête. Dans la stratégie d’Azure AD B2C, essayez de gérer correctement l’indisponibilité de l’API et peut-être de rendre une expérience réduite.

- [Gérez correctement les erreurs temporaires](https://docs.microsoft.com/azure/active-directory-b2c/restful-technical-profile#error-handling). Le profil d’API RESTFul vous permet de configurer des messages d’erreur pour différents [disjoncteurs](https://docs.microsoft.com/azure/architecture/patterns/circuit-breaker).

- Surveillez et utilisez de manière proactive l’intégration continue et la livraison continue (CI/CD) et alternez les informations d’identification d’accès à l’API, telles que les mots de passe et les certificats utilisés par le [moteur de profil technique](https://docs.microsoft.com/azure/active-directory-b2c/restful-technical-profile).

## <a name="api-management---best-practices"></a>Gestion des API – meilleures pratiques

Pendant que vous déployez les API REST et que vous configurez le profil technique RESTful, le respect des meilleures pratiques recommandées vous aidera à ne pas commettre d’erreurs courantes et à ne pas négliger certains aspects.

### <a name="how-to-manage-apis"></a>Gestion des API

- Gestion des API (APIM) publie, gère et analyse vos API. APIM gère également l’authentification pour fournir un accès sécurisé aux services principaux et aux microservices. Utilisez une passerelle API pour effectuer un scale-out des déploiements d’API, de la mise en cache et de l’équilibrage de charge.

- Il est recommandé d’obtenir le bon jeton au début du parcours utilisateur au lieu d’appeler plusieurs fois pour chaque API et de [sécuriser une API Azure APIM](https://docs.microsoft.com/azure/active-directory-b2c/secure-api-management?tabs=app-reg-ga).

## <a name="next-steps"></a>Étapes suivantes

- [Ressources de résilience pour les développeurs Azure AD B2C](resilience-b2c.md)
  - [Expérience résiliente de l’utilisateur final](resilient-end-user-experience.md)
  - [Résilience grâce aux meilleures pratiques des développeurs](resilience-b2c-developer-best-practices.md)
  - [Résilience par la surveillance et l’analytique](resilience-with-monitoring-alerting.md)
- [Renforcer la résilience de votre infrastructure d’authentification](resilience-in-infrastructure.md)
- [Augmenter la résilience de l’authentification et de l’autorisation dans vos applications](resilience-app-development-overview.md)
