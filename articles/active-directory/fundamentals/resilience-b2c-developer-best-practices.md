---
title: Résilience grâce aux meilleures pratiques de développeur à l’aide d’Azure AD B2C | Microsoft Docs
description: Résilience grâce aux meilleures pratiques de développeur en matière de gestion des identités et des accès client à l’aide d’Azure AD B2C
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
ms.openlocfilehash: ecde474abf3c814b7c3afa4ae18d044868785cf5
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "95919011"
---
# <a name="resilience-through-developer-best-practices"></a>Résilience grâce aux meilleures pratiques des développeurs

Dans cet article, nous partageons quelques enseignements basés sur notre expérience de travail avec des clients importants. Vous pouvez prendre en compte ces recommandations dans la conception et l’implémentation de vos services.

![Image montrant les composants de l’expérience de développeur](media/resilience-b2c-developer-best-practices/developer-best-practices-architecture.png)

## <a name="use-the-microsoft-authentication-library-msal"></a>Utiliser la bibliothèque d’authentification Microsoft (MSAL)

La [bibliothèque d’authentification Microsoft (MSAL)](https://docs.microsoft.com/azure/active-directory/develop/msal-overview) et la [bibliothèque d’authentification Microsoft Identity Web pour ASP.NET](https://docs.microsoft.com/azure/active-directory/develop/reference-v2-libraries) simplifient l’acquisition, la gestion, la mise en cache et l’actualisation des jetons requis par une application. Ces bibliothèques sont optimisées spécifiquement pour prendre en charge Microsoft Identity, notamment les fonctionnalités qui améliorent la résilience des applications.

Les développeurs doivent adopter les dernières versions de MSAL et rester à jour. Découvrez [comment accroître la résilience de l’authentification et de l’autorisation](resilience-app-development-overview.md) dans vos applications. Dans la mesure du possible, évitez d’implémenter votre propre pile d’authentification et utilisez des bibliothèques bien établies.

## <a name="optimize-directory-reads-and-writes"></a>Optimiser les lectures et écritures dans les répertoires

Le service d’annuaire Microsoft Azure AD B2C prend en charge des milliards d’authentifications par jour. Il est conçu pour un taux élevé de lectures par seconde. Optimisez vos écritures pour réduire les dépendances et augmenter la résilience.

### <a name="how-to-optimize-directory-reads-and-writes"></a>Optimisation des lectures et écritures dans les répertoires

- **Évitez d’écrire des fonctions dans le répertoire lors de la connexion** : N’exécutez jamais d’écriture lors de la connexion sans précondition (clause IF) dans vos stratégies personnalisées. Un cas d’usage nécessitant une écriture au moment de la connexion est [la migration juste-à-temps des mots de passe utilisateur](https://github.com/azure-ad-b2c/user-migration/tree/master/seamless-account-migration). Évitez tout scénario qui nécessite une écriture à chaque connexion.

  - Les [conditions préalables](https://docs.microsoft.com/azure/active-directory-b2c/userjourneys) d’un parcours utilisateur se présentent comme suit :

  ``
  <Precondition Type="ClaimEquals" ExecuteActionsIf="true"> 
  <Value>requiresMigration</Value>
  ...
  < Precondition/>
  ``
  - Renforcez la résistance vis-à-vis des [inscriptions par des bots en intégrant un système CAPTCHA](https://github.com/azure-ad-b2c/samples/tree/master/policies/captcha-integration).

  - Utilisez un [exemple de test de charge](https://docs.microsoft.com/azure/active-directory-b2c/best-practices#testing) pour simuler l’inscription et la connexion. 

- **Comprenez la limitation** : Le répertoire implémente les règles de limitation au niveau de l’application et du locataire. Il existe d’autres limites de taux pour les opérations de lecture/GET, d’écriture/POST, de mise à jour/PUT et de suppression/DELETE, et chaque opération a des limites différentes.

  - Une écriture au moment de la connexion se fera sous une opération POST pour les nouveaux utilisateurs ou PUT pour les utilisateurs existants.

  - Une stratégie personnalisée qui crée ou met à jour un utilisateur à chaque connexion peut atteindre une limite de taux PUT ou POST au niveau de l’application. Les mêmes limites s’appliquent lors de la mise à jour des objets annuaire via Azure AD ou Microsoft Graph. De même, examinez les lectures afin de réduire au minimum le nombre de lectures à chaque connexion.

  - Estimez le pic de charge pour prédire le taux d’écritures sur le répertoire et éviter la limitation. Les estimations des pics de trafic doivent inclure des estimations pour des actions telles que l’inscription, la connexion et l’authentification multifacteur (MFA). Veillez à tester à la fois le système Azure AD B2C et votre application pour les pics de trafic. Il est possible qu’Azure AD B2C puisse gérer la charge sans limitation, lorsque vos applications ou services en aval ne le feront pas.

  - Comprenez et planifiez la chronologie de votre migration. Lorsque vous envisagez de migrer des utilisateurs vers Azure AD B2C à l’aide de Microsoft Graph, tenez compte des limites de l’application et du locataire pour calculer le temps nécessaire à la migration des utilisateurs. Si vous divisez votre travail de création d’utilisateur ou votre script en utilisant deux applications, vous pouvez utiliser la limite par application. Il faudrait néanmoins rester sous le seuil par locataire.

  - Comprenez les effets de votre tâche de migration sur d’autres applications. Tenez compte du trafic actif traité par d’autres applications de confiance pour vous assurer de ne pas causer de limitation au niveau du locataire ni de privation de ressources pour votre application active. Pour plus d’informations, consultez les [conseils sur la limitation de Microsoft Graph](https://docs.microsoft.com/graph/throttling).
  
## <a name="extend-token-lifetimes"></a>Prolonger la durée de vie des jetons

Dans le cas peu probable où le service d’authentification Azure AD B2C ne parvient pas à effectuer de nouvelles inscriptions et connexions, vous pouvez toujours fournir une atténuation pour les utilisateurs qui sont connectés. Grâce à la [configuration](https://docs.microsoft.com/azure/active-directory-b2c/configure-tokens), vous pouvez permettre aux utilisateurs déjà connectés de continuer à utiliser l’application sans percevoir de perturbation jusqu’à ce qu’ils se déconnectent de l’application ou que la [session](https://docs.microsoft.com/azure/active-directory-b2c/session-behavior) expire en raison d’une inactivité.

Les besoins de votre entreprise et l’expérience souhaitée pour l’utilisateur final dicteront la fréquence d’actualisation des jetons pour les applications web et les applications monopages (SPA).

### <a name="how-to-extend-token-lifetimes"></a>Prolongation de la durée de vie des jetons

- **Applications web** : Pour les applications web où le jeton d’authentification est validé au début de la connexion, l’application dépend du cookie de session pour continuer à prolonger la validité de la session.

  - Permettez aux utilisateurs de rester connectés en implémentant des durées de session mobiles qui continueront à renouveler les sessions en fonction de l’activité de l’utilisateur. En cas d’interruption prolongée de l’émission de jetons, ces durées de session peuvent être augmentées en tant que configuration unique sur l’application. Maintenez la durée de vie de la session au maximum autorisé.

- **SPA** : Une SPA peut dépendre de jetons d’accès pour effectuer des appels aux API. Une SPA utilise traditionnellement le flux implicite qui n’entraîne pas de jeton d’actualisation. La SPA peut utiliser une iframe masquée pour effectuer de nouvelles demandes de jeton auprès du point de terminaison d’autorisation si le navigateur a toujours une session active auprès d’Azure AD B2C. Pour les SPA, quelques options sont disponibles pour permettre à l’utilisateur de continuer à utiliser l’application.

  - Prolongez la durée de validité du jeton d’accès pour répondre aux besoins de votre entreprise.

  - Construisez votre application de manière à utiliser une passerelle API comme proxy d’authentification. Dans cette configuration, la SPA se charge sans authentification et les appels d’API sont effectués vers la passerelle API. La passerelle API envoie l’utilisateur à travers un processus de connexion utilisant un [octroi de code d’autorisation](https://oauth.net/2/grant-types/authorization-code/) en fonction d’une stratégie et authentifie l’utilisateur. Par la suite, la session d’authentification entre la passerelle API et le client est maintenue à l’aide d’un cookie d’authentification. Les API sont gérées à partir de la passerelle API à l’aide du jeton obtenu par la passerelle API ou d’une autre méthode d’authentification directe telle que les certificats, les informations d’identification du client ou les clés API.

  - [Migrez votre SPA de l’octroi implicite](https://developer.microsoft.com/identity/blogs/msal-js-2-0-supports-authorization-code-flow-is-now-generally-available/) vers le [flux d’octroi de code d’autorisation](https://docs.microsoft.com/azure/active-directory-b2c/implicit-flow-single-page-application) avec la prise en charge de la clé de preuve pour l’échange de code (PKCE) et du partage des ressources cross-origin (CORS). Migrez votre application de MSAL.js 1.x vers MSAL.js 2.x pour réaliser la résilience des applications web.

  - Pour les applications mobiles, il est recommandé de prolonger à la fois la durée de vie des jetons d’actualisation et des jetons d’accès.

- **Applications back-end ou de microservices** : Comme les applications back-end (démon) ne sont pas interactives et ne sont pas dans un contexte utilisateur, la probabilité de vol de jetons est fortement réduite. Il est recommandé de trouver un juste équilibre entre la sécurité et la durée de vie et de définir une durée de vie longue pour les jetons.

## <a name="configure-single-sign-on"></a>Configurer l’authentification unique

Avec l’[authentification unique](https://docs.microsoft.com/azure/active-directory/manage-apps/what-is-single-sign-on), les utilisateurs se connectent une seule fois avec un seul compte et accèdent à plusieurs applications. L’application peut être une application web, mobile ou monopage (SPA), indépendamment de la plateforme ou du nom de domaine. Lorsque l’utilisateur se connecte pour la première fois à une application, Azure AD B2C maintient une [session basée sur un cookie](https://docs.microsoft.com/azure/active-directory-b2c/session-overview).

Lors des demandes d’authentification suivantes, Azure AD B2C lit et valide la session basée sur un cookie et émet un jeton d’accès sans inviter l’utilisateur à se reconnecter. Si l’authentification unique est configurée avec une étendue limitée au niveau d’une stratégie ou d’une application, l’accès ultérieur à d’autres stratégies et applications nécessitera une nouvelle authentification.

### <a name="how-to-configure-sso"></a>Configuration de l’authentification unique

[Configurez l’authentification unique](https://docs.microsoft.com/azure/active-directory/hybrid/how-to-connect-sso-quick-start) de manière à ce qu’elle s’applique à l’ensemble du locataire (par défaut) pour permettre à plusieurs applications et flux d’utilisateurs de votre locataire de partager la même session utilisateur. La configuration à l’échelle du locataire offre une meilleure résilience à une nouvelle authentification.  

## <a name="safe-deployment-practices"></a>Pratiques de déploiement sécurisé

Les interruptions de service les plus courantes sont les modifications de code et de configuration. L’adoption de processus et d’outils d’intégration continue et de livraison continue (CI/CD) permet un déploiement rapide à grande échelle et réduit les erreurs humaines pendant les tests et le déploiement en production. Adoptez CI/CD à des fins de réduction des erreurs, d’efficacité et de cohérence. [Azure Pipelines](https://docs.microsoft.com/azure/devops/pipelines/apps/cd/azure/cicd-data-overview) est un exemple de CI/CD.

## <a name="web-application-firewall"></a>Pare-feu d’applications web

Protégez vos applications contre les vulnérabilités connues, telles que les attaques par déni de service distribué (DDoS), les injections SQL, les scripts intersites, l’exécution de code à distance et bien d’autres, comme indiqué dans [Les 10 principales vulnérabilités de l’OWASP](https://owasp.org/www-project-top-ten/). Le déploiement d’un pare-feu d’applications web (WAF) peut permettre de se défendre contre des codes malveillants exploitant une faille de sécurité et des vulnérabilités courantes.

- Utilisez Azure [WAF](https://docs.microsoft.com/azure/web-application-firewall/overview), qui offre une protection centralisée contre les attaques.

- Utilisez WAF avec [Azure AD Identity Protection et l’accès conditionnel pour fournir une protection multicouche](https://docs.microsoft.com/azure/active-directory-b2c/conditional-access-identity-protection-overview) lors de l’utilisation d’Azure AD B2C.  

## <a name="secrets-rotation"></a>Rotation des secrets

Azure AD B2C utilise des secrets pour les applications, les API, les stratégies et le chiffrement. Les secrets sécurisent l’authentification, les interactions externes et le stockage. Le National Institute of Standards and Technology (NIST) appelle « cryptoperiod » l’intervalle de temps pendant lequel une clé spécifique est autorisée à être utilisée par des entités légitimes. Choisissez la durée appropriée de [cryptoperiod](https://csrc.nist.gov/publications/detail/sp/800-57-part-1/rev-5/final) pour répondre aux besoins de votre entreprise. Les développeurs doivent définir manuellement l’expiration et alterner les secrets bien avant leur expiration.

### <a name="how-to-implement-secret-rotation"></a>Implémentation de la rotation des secrets

- Utilisez des [identités managées](https://docs.microsoft.com/azure/active-directory/managed-identities-azure-resources/overview) pour les ressources prises en charge afin de vous authentifier auprès d’un service qui prend en charge l’authentification Azure AD. Lorsque vous utilisez des identités managées, vous pouvez gérer les ressources automatiquement, notamment la rotation des informations d’identification.

- Effectuez un inventaire de l’ensemble des [clés et certificats configurés](https://docs.microsoft.com/azure/active-directory-b2c/policy-keys-overview) dans Azure AD B2C. Cette liste est susceptible d’inclure des clés utilisées dans des stratégies personnalisées, des [API](https://docs.microsoft.com/azure/active-directory-b2c/secure-rest-api), un jeton d’ID de signature et des certificats pour SAML.

- À l’aide de CI/CD, alternez les secrets qui sont sur le point d’expirer dans les deux mois suivant la saison de pointe prévue. La durée maximale recommandée de cryptoperiod des clés privées associées à un certificat est d’un an.

- Surveillez et alternez de manière proactive les informations d’identification d’accès à l’API, telles que les mots de passe et les certificats.

## <a name="test-rest-apis"></a>Tester des API REST

Dans le contexte de la résilience, les tests des API REST doivent inclure la vérification des codes HTTP, de la charge utile de réponse, des en-têtes et des performances. Les tests ne doivent pas seulement inclure des tests de « bon chemin », mais aussi vérifier si l’API gère correctement les scénarios de problème.

### <a name="how-to-test-apis"></a>Test des API

Nous vous recommandons d’inclure dans votre plan de test des [tests d’API complets](https://docs.microsoft.com/azure/active-directory-b2c/best-practices#testing). Si vous prévoyez une augmentation de la demande en raison du trafic induit par une promotion ou des vacances, vous devez revoir vos tests de charge en fonction des nouvelles estimations. Procédez à un test de charge de vos API et du réseau de distribution de contenu (CDN) dans un environnement de développement et non en production.

## <a name="next-steps"></a>Étapes suivantes

- [Ressources de résilience pour les développeurs Azure AD B2C](resilience-b2c.md)
  - [Expérience résiliente de l’utilisateur final](resilient-end-user-experience.md)
  - [Interfaces résilientes avec processus externes](resilient-external-processes.md)
  - [Résilience par la surveillance et l’analytique](resilience-with-monitoring-alerting.md)
- [Renforcer la résilience de votre infrastructure d’authentification](resilience-in-infrastructure.md)
- [Augmenter la résilience de l’authentification et de l’autorisation dans vos applications](resilience-app-development-overview.md)
