---
title: Durées de vie des jetons configurables
titleSuffix: Microsoft identity platform
description: Découvrez comment définir des durées de vie pour des jetons émis par la Plateforme d’identité Microsoft.
services: active-directory
author: rwike77
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 12/14/2020
ms.author: ryanwi
ms.custom: aaddev, identityplatformtop40, content-perf, FY21Q1, contperf-fy21q1
ms.reviewer: hirsin, jlu, annaba
ms.openlocfilehash: f73186612fe79af88e84956bb4d0f0b374f4c986
ms.sourcegitcommit: 63d0621404375d4ac64055f1df4177dfad3d6de6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/15/2020
ms.locfileid: "97507793"
---
# <a name="configurable-token-lifetimes-in-microsoft-identity-platform-preview"></a>Durées de vie des jetons configurables dans la plateforme d’identité Microsoft (préversion)

> [!IMPORTANT]
> Après mai 2020, les locataires ne seront plus en mesure de configurer la durée de vie des jetons d’actualisation et de session.  Azure Active Directory cessera d’honorer la configuration actuelle des jetons d’actualisation et de session dans les stratégies après le 30 janvier 2021. Vous pourrez toujours configurer la durée de vie des jetons d'accès après la dépréciation.
>
> Si vous devez continuer à définir la période de temps avant qu’un utilisateur soit invité à se connecter à nouveau, configurez la fréquence de connexion dans Accès conditionnel. Pour en savoir plus sur l’accès conditionnel, consultez [Configurer la gestion de session d’authentification avec l’accès conditionnel](/azure/active-directory/conditional-access/howto-conditional-access-session-lifetime).
>
> Pour les locataires qui ne souhaitent pas utiliser l’accès conditionnel après la date de mise hors service, ils peuvent s’attendre à ce qu’Azure AD honore la configuration par défaut décrite dans la section suivante.

## <a name="configurable-token-lifetime-properties-after-the-retirement"></a>Propriétés des durées de vie des jetons configurables après la mise hors service
La configuration des jetons d’actualisation et de session est affectée par les propriétés suivantes et leurs valeurs définies respectives. Après la mise hors service de la configuration des jetons d’actualisation et de session, Azure AD honorera uniquement la valeur par défaut décrite ci-dessous, que les stratégies aient des valeurs personnalisées configurées ou non. Vous pourrez toujours configurer la durée de vie des jetons d’accès après la mise hors service. 

|Propriété   |Chaîne de propriété de stratégie    |Éléments affectés |Default |
|----------|-----------|------------|------------|
|Délai d’inactivité maximale de jeton d’actualisation |MaxInactiveTime  |Jetons d’actualisation |90 jours  |
|Âge maximal de jeton d’actualisation à facteur unique  |MaxAgeSingleFactor  |Jetons d’actualisation (pour tous les utilisateurs)  |Jusqu’à révocation  |
|Âge maximal de jeton d’actualisation multifacteur  |MaxAgeMultiFactor  |Jetons d’actualisation (pour tous les utilisateurs) |180 jours  |
|Âge maximal de jeton de session à facteur unique  |MaxAgeSessionSingleFactor |Jetons de session (persistants et non persistants)  |Jusqu’à révocation |
|Âge maximal de jeton de session multifacteur  |MaxAgeSessionMultiFactor  |Jetons de session (persistants et non persistants)  |180 jours |

## <a name="identify-configuration-in-scope-of-retirement"></a>Identifier la configuration dans le cadre de la mise hors service

Pour commencer, suivez les étapes ci-dessous :

1. Téléchargez la dernière [version préliminaire publique du module Azure AD PowerShell](https://www.powershellgallery.com/packages/AzureADPreview).
1. Exécutez la commande `Connect` pour vous connecter à votre compte Administrateur Azure AD. Exécutez cette commande chaque fois que vous démarrez une nouvelle session.

    ```powershell
    Connect-AzureAD -Confirm
    ```

1. Pour afficher toutes les stratégies qui ont été créées dans votre organisation, exécutez la cmdlet [Get-AzureADPolicy](/powershell/module/azuread/get-azureadpolicy?view=azureadps-2.0-preview&preserve-view=true).  Tout résultat ayant une valeur de propriété définie qui diffère des valeurs par défaut indiquées ci-dessus relève du champ d’application de la mise hors service.

    ```powershell
    Get-AzureADPolicy -All
    ```

1. Pour déterminer les applications et les principaux de service liés à une stratégie spécifique, vous avez identifié la cmdlet [Get-AzureADPolicyAppliedObject](/powershell/module/azuread/get-azureadpolicyappliedobject?view=azureadps-2.0-preview&preserve-view=true) en remplaçant **1a37dad8-5da7-4cc8-87c7-efbc0326cf20** par l’un de vos ID de stratégie. Vous pouvez ensuite décider s’il faut configurer la fréquence de connexion de l’accès conditionnel ou conserver les valeurs Azure AD par défaut.

    ```powershell
    Get-AzureADPolicyAppliedObject -id 1a37dad8-5da7-4cc8-87c7-efbc0326cf20
    ```

Si votre locataire a des stratégies qui définissent des valeurs personnalisées pour les propriétés de configuration des jetons d’actualisation et de session, Microsoft vous recommande de mettre à jour ces stratégies avec des valeurs qui reflètent les valeurs par défaut décrites ci-dessus. Si aucune modification n’est apportée, Azure AD honorera automatiquement les valeurs par défaut.  

## <a name="overview"></a>Vue d'ensemble

Vous pouvez spécifier la durée de vie d’un jeton émis par la Plateforme d’identité Microsoft. Vous pouvez définir les durées de vie des jetons pour toutes les applications de votre organisation, pour une application mutualisée (plusieurs organisations) ou pour un principal de service spécifique de votre organisation. Cependant, nous ne prenons pas en charge actuellement la configuration des durées de vie des jetons pour les [principaux de service d’identité managée](../managed-identities-azure-resources/overview.md).

Dans Azure AD, un objet de stratégie représente un ensemble de règles appliquées sur des applications individuelles ou sur toutes les applications d’une organisation. Chaque type de stratégie comporte une structure unique avec un ensemble de propriétés qui sont ensuite appliquées aux objets auxquels elles sont affectées.

Vous pouvez désigner une stratégie comme stratégie par défaut pour votre organisation. La stratégie est appliquée à toutes les applications de l’organisation tant qu’elle n’est pas remplacée par une stratégie pourvue d’une priorité plus élevée. Vous pouvez également affecter une stratégie à des applications spécifiques. L’ordre de priorité varie par type de stratégie.

Pour obtenir des exemples, consultez des [exemples de configuration des durées de vie des jetons](configure-token-lifetimes.md).

> [!NOTE]
> La stratégie de durée de vie des jetons configurable s’applique seulement aux clients mobiles et de poste de travail qui accèdent aux ressources SharePoint Online et OneDrive Entreprise ; elle ne s’applique pas aux sessions de navigateur web.
> Pour gérer la durée de vie des sessions de navigateur web pour SharePoint Online et OneDrive Entreprise, utilisez la fonctionnalité [Durée de vie de la session d’accès conditionnel](../conditional-access/howto-conditional-access-session-lifetime.md). Reportez-vous au [blog SharePoint Online](https://techcommunity.microsoft.com/t5/SharePoint-Blog/Introducing-Idle-Session-Timeout-in-SharePoint-and-OneDrive/ba-p/119208) pour en savoir plus sur la configuration des délais d’expiration des sessions inactives.

## <a name="token-types"></a>Types de jetons

Vous pouvez définir les stratégies de durée de vie des jetons pour les jetons d’actualisation, les jetons d’accès, les jetons SAML, les jetons de session et les jetons d’ID.

### <a name="access-tokens"></a>Jetons d’accès

Les clients utilisent des jetons d’accès pour accéder à une ressource protégée. Un jeton d’accès peut uniquement être utilisé pour une combinaison spécifique d’utilisateur, de client et de ressource. Les jetons d’accès ne peuvent pas être révoqués et sont valides jusqu’à leur expiration. Un acteur malveillant qui a obtenu un jeton d’accès peut l’utiliser pour prolonger sa durée de vie. L’ajustement de la durée de vie des jetons d’accès représente un compromis entre l’amélioration des performances du système et l’augmentation de la durée pendant laquelle le client conserve un accès une fois son compte désactivé. Les performances du système sont améliorées en réduisant le nombre de fois où un client doit acquérir un nouveau jeton d’accès.  La valeur par défaut est 1 heure. Après 1 heure, le client doit utiliser le jeton d’actualisation pour acquérir (généralement en mode silencieux) un nouveau jeton d’actualisation et un jeton d’accès. 

### <a name="saml-tokens"></a>Jetons SAML

Les jetons SAML sont utilisés par de nombreuses applications SAAS basées sur le Web, et sont obtenus à l’aide du point de terminaison du protocole SAML2 d’Azure Active Directory. Ils sont également consommés par les applications utilisant WS-Federation. La durée de vie par défaut du jeton est d’une heure. Du point de vue d’une application, la période de validité du jeton est spécifiée par la valeur NotOnOrAfter de l’élément `<conditions …>` dans le jeton. Au terme de la période de validité du jeton, le client doit initier une nouvelle requête d’authentification, qui est souvent satisfaite sans connexion interactive en raison du jeton de session d’authentification unique (SSO).

La valeur de NotOnOrAfter peut être modifiée à l’aide du paramètre `AccessTokenLifetime` dans un élément `TokenLifetimePolicy`. Elle sera définie sur la durée de vie configurée dans la stratégie si elle existe, à laquelle sera ajouté un facteur de décalage de l’horloge de cinq minutes.

La valeur NotOnOrAfter de la confirmation d’objet spécifiée dans l’élément `<SubjectConfirmationData>` n’est pas affectée par la configuration de la durée de vie du jeton. 

### <a name="refresh-tokens"></a>Jetons d’actualisation

Lorsqu’un client acquiert un jeton d’accès pour accéder à une ressource protégée, il reçoit aussi un jeton d’actualisation. Le jeton d’actualisation permet d’obtenir de nouvelles paires de jetons d’accès/actualisation à l’expiration du jeton d’accès actuel. Un jeton d’actualisation est lié à une combinaison d’utilisateur et de client. Un jeton d’actualisation peut être [révoqué à tout moment](access-tokens.md#token-revocation), et la validité du jeton est vérifiée à chaque fois qu’il est utilisé.  Les jetons d’actualisation ne sont pas révoqués lorsqu’ils sont utilisés pour récupérer de nouveaux jetons d’accès. Il est cependant fortement recommandé de supprimer l’ancien jeton lorsque vous en obtenez un nouveau.

Il est important de distinguer les clients confidentiels des clients publics, car cela a un impact sur la durée d’utilisation des jetons d’actualisation. Pour plus d’informations sur les différents types de client, consultez [RFC 6749](https://tools.ietf.org/html/rfc6749#section-2.1).

#### <a name="token-lifetimes-with-confidential-client-refresh-tokens"></a>Durées de vie des jetons avec des jetons d’actualisation de client confidentiel
Les clients confidentiels sont des applications qui peuvent stocker un mot de passe client (secret). Ils peuvent prouver que les requêtes proviennent de l’application cliente sécurisée et non d’un acteur malveillant. Par exemple, une application web est un client confidentiel, car elle peut stocker un secret de client sur le serveur web. Elle n’est pas exposée. Comme ces flux sont plus sécurisés, les durées de vie par défaut des jetons d’actualisation émis en direction de ces flux sont de `until-revoked`, ne peuvent pas être modifiées à l’aide de la stratégie et ne peuvent pas être révoquées sur les réinitialisations de mot de passe volontaires.

#### <a name="token-lifetimes-with-public-client-refresh-tokens"></a>Durées de vie des jetons avec des jetons d’actualisation de client public

Les clients publics ne peuvent pas stocker en toute sécurité un mot de passe client (secret). Par exemple, une application iOS/Android ne peut pas masquer un secret au propriétaire de la ressource et est donc considérée comme un client public. Vous pouvez définir des stratégies sur des ressources pour empêcher les jetons d’actualisation des clients publics antérieurs à une période spécifiée d’obtenir une nouvelle paire de jetons d’accès/actualisation. Pour ce faire, utilisez la [propriété Délai d’inactivité maximale de jeton d’actualisation](#refresh-token-max-inactive-time) (`MaxInactiveTime`). Vous pouvez également utiliser des stratégies pour définir un délai au-delà duquel les jetons d’actualisation ne sont plus acceptés. Pour ce faire, utilisez la propriété [Âge maximal de jeton d’actualisation à facteur unique](#single-factor-session-token-max-age) ou la propriété [Âge maximal de jeton de session multifacteur](#multi-factor-refresh-token-max-age). Vous pouvez ajuster la durée de vie des jetons d’actualisation pour contrôler le moment et la fréquence auxquels l’utilisateur doit entrer de nouveau les informations d’identification au lieu d’être authentifié de nouveau en mode silencieux lorsqu’il utilise une application cliente publique.

> [!NOTE]
> La propriété Âge maximal représente la durée pendant laquelle un seul jeton peut être utilisé. 

### <a name="id-tokens"></a>Jetons d’ID
Les jetons d’ID sont transmis aux sites web et clients natifs. Les jetons d’ID contiennent des informations de profil sur un utilisateur. Un jeton d’ID est lié à une combinaison spécifique d’utilisateur et de client. Les jetons d’ID sont considérés comme valides jusqu’à leur expiration. En règle générale, une application web fait correspondre la durée de vie de session d’un utilisateur de l’application à la durée de vie du jeton d’ID émis pour l’utilisateur. Vous pouvez ajuster la durée de vie des jetons d’ID pour contrôler la fréquence à laquelle l’application web arrête la session de l’application et demande à l’utilisateur de s’authentifier à nouveau auprès de la Plateforme d’identité Microsoft (en mode silencieux ou interactif).

### <a name="single-sign-on-session-tokens"></a>Jetons de session d’authentification unique
Quand utilisateur s’authentifie auprès de la Plateforme d’identité Microsoft, une session d’authentification unique (SSO) est établie avec le navigateur de l’utilisateur et la Plateforme d’identité Microsoft. Le jeton SSO représente cette session sous la forme d’un cookie. Le jeton de session SSO n’est pas lié à une application cliente/ressource spécifique. Les jetons de session SSO peuvent être révoqués, et leur validité est vérifiée à chaque fois qu’ils sont utilisés.

La Plateforme d’identité Microsoft utilise deux types de jetons de session d’authentification unique : persistant et non persistant. Les jetons de session persistants sont stockés en tant que cookies persistants par le navigateur. Les jetons de session non persistants sont stockés en tant que cookies de session. (Les cookies de session sont détruits lors de la fermeture du navigateur.) En règle générale, un jeton de session non persistant est stocké. Cependant, quand l’utilisateur sélectionne la case à cocher **Maintenir la connexion** lors de l’authentification, un jeton de session persistant est stocké.

Les jetons de session non persistants ont une durée de vie de 24 heures. Les jetons persistants ont une durée de vie de 90 jours. À chaque fois qu’un jeton de session SSO est utilisé au cours de sa période de validité, celle-ci est prolongée à nouveau de 24 heures ou de 90 jours, en fonction du type de jeton. Si un jeton de session SSO n’est pas utilisé au cours de sa période de validité, il est considéré comme arrivé à expiration et n’est plus accepté.

Vous pouvez utiliser une stratégie pour définir la durée après laquelle le premier jeton de session a été émis au-delà de celle pour laquelle le jeton de session n’est plus accepté. (Pour ce faire, utilisez la propriété Âge maximal de jeton de session). Vous pouvez ajuster la durée de vie des jetons de session pour contrôler le moment et la fréquence auxquels l’utilisateur doit entrer de nouveau les informations d’identification au lieu d’être authentifié de nouveau en mode silencieux lorsqu’il utilise une application web.

### <a name="token-lifetime-policy-properties"></a>Propriétés des stratégies de durée de vie des jetons
Une stratégie de durée de vie des jetons est un type d’objet de stratégie qui contient des règles de durée de vie des jetons. Utilisez les propriétés de la stratégie pour contrôler les durées de vie des jetons spécifiés. Si aucune stratégie n’est définie, le système applique la valeur de durée de vie par défaut.

### <a name="configurable-token-lifetime-properties"></a>Propriétés des durées de vie des jetons configurables
| Propriété | Chaîne de propriété de stratégie | Éléments affectés | Default | Minimum | Maximale |
| --- | --- | --- | --- | --- | --- |
| Durée de vie de jeton d’accès |AccessTokenLifetime<sup>2</sup> |Jetons d’accès, jetons d’ID, jetons SAML2 |1 heure |10 minutes |1 jour |
| Délai d’inactivité maximale de jeton d’actualisation |MaxInactiveTime |Jetons d’actualisation |90 jours |10 minutes |90 jours |
| Âge maximal de jeton d’actualisation à facteur unique |MaxAgeSingleFactor |Jetons d’actualisation (pour tous les utilisateurs) |Jusqu’à révocation |10 minutes |Jusqu’à révocation<sup>1</sup> |
| Âge maximal de jeton d’actualisation multifacteur |MaxAgeMultiFactor |Jetons d’actualisation (pour tous les utilisateurs) | 180 jours |10 minutes |180 jours<sup>1</sup> |
| Âge maximal de jeton de session à facteur unique |MaxAgeSessionSingleFactor |Jetons de session (persistants et non persistants) |Jusqu’à révocation |10 minutes |Jusqu’à révocation<sup>1</sup> |
| Âge maximal de jeton de session multifacteur |MaxAgeSessionMultiFactor |Jetons de session (persistants et non persistants) | 180 jours |10 minutes | 180 jours<sup>1</sup> |

* <sup>1</sup>Une durée explicite maximale de 365 jours peut être définie pour ces attributs.
* <sup>2</sup>Pour s’assurer que le client web Microsoft Teams fonctionne, il est recommandé de conserver AccessTokenLifetime sur une durée supérieure à 15 minutes pour Microsoft Teams.

### <a name="exceptions"></a>Exceptions
| Propriété | Éléments affectés | Default |
| --- | --- | --- |
| Âge maximal de jeton d’actualisation (émis pour les utilisateurs fédérés disposant d’informations de révocation insuffisantes<sup>1</sup>) |Jetons d’actualisation (émis pour les utilisateurs fédérés disposant d’informations de révocation insuffisantes<sup>1</sup>) |12 heures |
| Délai d’inactivité maximale de jeton d’actualisation (émis pour les clients confidentiels) |Jetons d’actualisation (émis pour les clients confidentiels) |90 jours |
| Âge maximal de jeton d’actualisation (émis pour les clients confidentiels) |Jetons d’actualisation (émis pour les clients confidentiels) |Jusqu’à révocation |

* <sup>1</sup> Les utilisateurs fédérés qui disposent d’informations de révocation insuffisantes incluent tous les utilisateurs qui n’ont pas l’attribut « LastPasswordChangeTimestamp » synchronisé. Cette valeur Âge maximal courte est affectée à ces utilisateurs car Azure Active Directory est incapable de vérifier quand révoquer les jetons qui sont liés à d’anciennes informations d’identification (par exemple un mot de passe qui a été changé) et doit vérifier plus fréquemment pour s’assurer que les jetons associés sont toujours conformes. Pour améliorer cette expérience, les administrateurs de locataires doivent s’assurer qu’ils synchronisent l’attribut « LastPasswordChangeTimestamp » (cela peut être défini sur l’objet utilisateur à l’aide de PowerShell ou d’AADSync).

### <a name="policy-evaluation-and-prioritization"></a>Définition des priorités et évaluation de la stratégie
Vous pouvez créer, puis affecter une stratégie de durée de vie à une application spécifique, à votre organisation et à vos principaux de service. Plusieurs stratégies peuvent s’appliquer à une application spécifique. La stratégie de durée de vie du jeton appliquée suit les règles ci-dessous :

* Si une stratégie est explicitement affectée au principal de service, elle est appliquée.
* Si aucune stratégie n’est explicitement affectée au principal de service, une stratégie explicitement affectée à l’organisation parente du principal de service est appliquée.
* Si aucune stratégie n’est explicitement affectée au principal de service ou à l’organisation, la stratégie affectée à l’application est appliquée.
* Si aucune stratégie n’a été affectée au principal du service, à l’organisation ou à l’objet d’application, les valeurs par défaut sont appliquées. (Consultez le tableau dans la section [Propriétés des durées de vie des jetons configurables](#configurable-token-lifetime-properties).)

Pour plus d’informations sur la relation existant entre les objets de principal de service et d’application, consultez [Objets application et principal du service dans Azure Active Directory](app-objects-and-service-principals.md).

La validité d’un jeton est évaluée lors de son utilisation. C’est la stratégie pourvue de la priorité la plus élevée sur l’application ouverte qui est appliquée.

Tous les intervalles de temps utilisés ici sont mis en forme selon C# [TimeSpan](/dotnet/api/system.timespan) object - D.HH:MM:SS.  Par conséquent, 80 jours et 30 minutes s’affichent sous la forme `80.00:30:00`.  La première valeur D peut être supprimée si elle est égale à zéro ; 90 minutes deviennent alors `00:90:00`.  

> [!NOTE]
> Voici un scénario d’exemple.
>
> Un utilisateur souhaite accéder à deux applications web : l’application web A et l’application web B.
> 
> Facteurs :
> * Les deux applications web résident dans la même organisation parente.
> * La stratégie 1 de durée de vie des jetons pourvue d’une propriété Âge maximal de jeton de session de 8 heures est définie comme valeur par défaut de l’organisation parente.
> * L’application web A est une application web d’utilisation régulière, qui n’est liée à aucune stratégie.
> * L’application web B est utilisée pour les processus très sensibles. Son principal de service est lié à la stratégie 2 de durée de vie des jetons pourvue d’une propriété Âge maximal de jeton de session de 30 minutes.
>
> À 12 h 00, l’utilisateur démarre une nouvelle session de navigateur et tente d’accéder à l’application web A. Il est redirigé vers la Plateforme d’identité Microsoft et invité à se connecter. Cette action crée un cookie avec un jeton de session dans le navigateur. L’utilisateur est redirigé vers l’application web A avec un jeton d’ID qui lui permet d’accéder à l’application.
>
> À 12 h 15, l’utilisateur tente d’accéder à l’application web B. Le navigateur le redirige vers la Plateforme d’identité Microsoft qui détecte le cookie de session. Le principal de service de l’application web B est lié à la stratégie 2 de durée de vie des jetons, mais fait également partie de l’organisation parente avec la stratégie 1 de durée de vie des jetons par défaut. La stratégie 2 de durée de vie des jetons est appliquée puisque les stratégies liées aux principaux de service ont une priorité supérieure à celle des stratégies par défaut de l’organisation. Comme le jeton de session a été initialement émis au cours des 30 dernières minutes, il est considéré comme valide. L’utilisateur est redirigé vers l’application web B avec un jeton d’ID qui lui octroie un accès.
>
> À 13 h 00, l’utilisateur tente d’accéder à l’application web A. Il est redirigé vers la Plateforme d’identité Microsoft. L’application web A n’est liée à aucune stratégie, mais comme elle réside dans une organisation avec la stratégie 1 de durée de vie des jetons par défaut, cette stratégie est appliquée. Le cookie de session qui a initialement été émis au cours des huit dernières heures est détecté. L’utilisateur est redirigé en mode silencieux vers l’application web A avec un nouveau jeton d’ID. Aucune authentification de l’utilisateur n’est nécessaire.
>
> Immédiatement après, l’utilisateur tente d’accéder à l’application web B. Il est redirigé vers la Plateforme d’identité Microsoft. Comme avant, la stratégie 2 de durée de vie des jetons est appliquée. Étant donné que le jeton a été émis il y a plus de 30 minutes, l’utilisateur est invité à entrer de nouveau ses informations d’identification. Un nouveau jeton de session et un jeton d’ID sont émis. L’utilisateur peut alors accéder à l’application web B.
>
>

## <a name="configurable-policy-property-details"></a>Présentation des propriétés de stratégie configurables
### <a name="access-token-lifetime"></a>Durée de vie de jeton d’accès
**Chaîne :** AccessTokenLifetime

**Éléments affectés :** Jetons d’accès, jetons d’ID, jetons SAML

**Résumé :** cette stratégie détermine la durée pendant laquelle les jetons d’accès et d’ID sont considérés comme valides. Réduire la propriété Durée de vie de jeton d’accès atténue le risque qu’un jeton d’accès ou jeton d’ID soit utilisé par un acteur malveillant pour une période prolongée. (Ces jetons ne peuvent pas être révoqués.) L’inconvénient est que les performances sont affectées, car les jetons sont remplacés plus souvent.

Pour obtenir un exemple, consultez [Créer une stratégie de connexion Web](configure-token-lifetimes.md#create-a-policy-for-web-sign-in).

### <a name="refresh-token-max-inactive-time"></a>Délai d’inactivité maximale de jeton d’actualisation
**Chaîne :** MaxInactiveTime

**Éléments affectés :** Jetons d’actualisation

**Résumé :** cette stratégie détermine l’âge qu’un jeton d’actualisation peut avoir avant qu’un client ne puisse plus l’utiliser pour récupérer une nouvelle paire de jetons d’actualisation/d’accès lors de la tentative d’accès à cette ressource. Comme un nouveau jeton d’actualisation est généralement renvoyé lorsqu’un jeton d’actualisation est utilisé, la stratégie bloque l’accès si le client tente d’accéder à une ressource à l’aide du jeton d’actualisation en cours pour la période spécifiée.

Cette stratégie force les utilisateurs qui n’ont pas été actifs sur leur client à s’authentifier de nouveau pour récupérer un nouveau jeton d’actualisation.

La propriété Délai d’inactivité maximale de jeton d’actualisation doit être définie sur une valeur inférieure à celles des propriétés Âge maximal de jeton d’actualisation à facteur unique et Âge maximal de jeton d’actualisation multifacteur.

Pour obtenir un exemple, consultez [Créer une stratégie pour une application native qui appelle une API Web](configure-token-lifetimes.md#create-a-policy-for-a-native-app-that-calls-a-web-api).

### <a name="single-factor-refresh-token-max-age"></a>Âge maximal de jeton d’actualisation à facteur unique
**Chaîne :** MaxAgeSingleFactor

**Éléments affectés :** Jetons d’actualisation

**Résumé :** cette stratégie détermine la durée pendant laquelle un utilisateur peut utiliser des jetons d’actualisation pour obtenir de nouvelles paires de jetons d’accès/actualisation après sa dernière authentification réussie avec un seul facteur. Une fois que l’utilisateur est authentifié et qu’il reçoit un nouveau jeton d’actualisation, il peut utiliser le flux de jeton d’actualisation pour la période spécifiée. (Cela est vrai tant que le jeton d’actualisation actuel n’a pas été révoqué et qu’il ne reste pas inutilisé plus longtemps que la durée d’inactivité.) À ce stade, l’utilisateur est contraint de s’authentifier de nouveau pour recevoir un nouveau jeton d’actualisation.

En réduisant l’âge maximal, vous obligez les utilisateurs à s’authentifier plus souvent. Comme l’authentification à facteur unique est considérée comme moins sécurisée qu’une authentification multifacteur, nous vous recommandons de définir cette propriété sur une valeur inférieure ou égale à celle de la propriété Âge maximal de jeton d’actualisation multifacteur.

Pour obtenir un exemple, consultez [Créer une stratégie pour une application native qui appelle une API Web](configure-token-lifetimes.md#create-a-policy-for-a-native-app-that-calls-a-web-api).

### <a name="multi-factor-refresh-token-max-age"></a>Âge maximal de jeton d’actualisation multifacteur
**Chaîne :** MaxAgeMultiFactor

**Éléments affectés :** Jetons d’actualisation

**Résumé :** cette stratégie détermine la durée pendant laquelle un utilisateur peut utiliser des jetons d’actualisation pour obtenir de nouvelles paires de jetons d’accès/actualisation après sa dernière authentification réussie avec plusieurs facteurs. Une fois que l’utilisateur est authentifié et qu’il reçoit un nouveau jeton d’actualisation, il peut utiliser le flux de jeton d’actualisation pour la période spécifiée. (Cela est vrai tant que le jeton d’actualisation actuel n’a pas été révoqué et qu’il ne reste pas inutilisé plus longtemps que la durée d’inactivité.) À ce stade, les utilisateurs sont contraints de s’authentifier de nouveau pour recevoir un nouveau jeton d’actualisation.

En réduisant l’âge maximal, vous obligez les utilisateurs à s’authentifier plus souvent. Comme l’authentification à facteur unique est considérée comme moins sécurisée qu’une authentification multifacteur, nous vous recommandons de définir cette propriété sur une valeur supérieure ou égale à celle de la propriété Âge maximal de jeton d’actualisation à facteur unique.

Pour obtenir un exemple, consultez [Créer une stratégie pour une application native qui appelle une API Web](configure-token-lifetimes.md#create-a-policy-for-a-native-app-that-calls-a-web-api).

### <a name="single-factor-session-token-max-age"></a>Âge maximal de jeton de session à facteur unique
**Chaîne :** MaxAgeSessionSingleFactor

**Éléments affectés :** Jetons de session (persistants et non persistants)

**Résumé :** cette stratégie détermine la durée pendant laquelle un utilisateur peut utiliser des jetons de session pour obtenir un nouvel ID et un nouveau jeton de session après sa dernière authentification réussie avec un seul facteur. Une fois que l’utilisateur est authentifié et qu’il reçoit un nouveau jeton de session, il peut utiliser le flux de jeton de session pour la période spécifiée. (Cela est vrai tant que le jeton de session actuel n’est pas révoqué et qu’il n’a pas expiré.) Après la période de temps spécifié, l’utilisateur est forcé à s’authentifier de nouveau pour recevoir un nouveau jeton de session.

En réduisant l’âge maximal, vous obligez les utilisateurs à s’authentifier plus souvent. Comme l’authentification à facteur unique est considérée comme moins sécurisée qu’une authentification multifacteur, nous vous recommandons de définir cette propriété sur une valeur inférieure ou égale à celle de la propriété Âge maximal de jeton de session multifacteur.

Pour obtenir un exemple, consultez [Créer une stratégie de connexion Web](configure-token-lifetimes.md#create-a-policy-for-web-sign-in).

### <a name="multi-factor-session-token-max-age"></a>Âge maximal de jeton de session multifacteur
**Chaîne :** MaxAgeSessionMultiFactor

**Éléments affectés :** Jetons de session (persistants et non persistants)

**Résumé :** cette stratégie détermine la durée pendant laquelle un utilisateur peut utiliser des jetons de session pour obtenir un nouvel ID et un nouveau jeton de session après sa dernière authentification réussie avec plusieurs facteurs. Une fois que l’utilisateur est authentifié et qu’il reçoit un nouveau jeton de session, il peut utiliser le flux de jeton de session pour la période spécifiée. (Cela est vrai tant que le jeton de session actuel n’est pas révoqué et qu’il n’a pas expiré.) Après la période de temps spécifié, l’utilisateur est forcé à s’authentifier de nouveau pour recevoir un nouveau jeton de session.

En réduisant l’âge maximal, vous obligez les utilisateurs à s’authentifier plus souvent. Comme l’authentification à facteur unique est considérée comme moins sécurisée qu’une authentification multifacteur, nous vous recommandons de définir cette propriété sur une valeur supérieure ou égale à celle de la propriété Âge maximal de jeton de session à facteur unique.

## <a name="cmdlet-reference"></a>Référence des applets de commande

Il s’agit d’applets de commande du [module Azure Active Directory PowerShell pour Graph (préversion)](/powershell/module/azuread/?view=azureadps-2.0-preview#service-principals&preserve-view=true&preserve-view=true).

### <a name="manage-policies"></a>Gérer les stratégies

Vous pouvez utiliser les applets de commande suivantes pour gérer les stratégies.

| Applet de commande | Description | 
| --- | --- |
| [New-AzureADPolicy](/powershell/module/azuread/new-azureadpolicy?view=azureadps-2.0-preview&preserve-view=true) | Permet de créer une stratégie. |
| [Get-AzureADPolicy](/powershell/module/azuread/get-azureadpolicy?view=azureadps-2.0-preview&preserve-view=true) | Permet d’obtenir toutes les stratégies d’Azure AD ou une stratégie spécifiée. |
| [Get-AzureADPolicyAppliedObject](/powershell/module/azuread/get-azureadpolicyappliedobject?view=azureadps-2.0-preview&preserve-view=true) | Permet d’obtenir toutes les applications et tous les principaux de service liés à une stratégie. |
| [Set-AzureADPolicy](/powershell/module/azuread/set-azureadpolicy?view=azureadps-2.0-preview&preserve-view=true) | Met à jour une stratégie existante. |
| [Remove-AzureADPolicy](/powershell/module/azuread/remove-azureadpolicy?view=azureadps-2.0-preview&preserve-view=true) | Supprime la stratégie spécifiée. |

### <a name="application-policies"></a>Stratégies d’application
Vous pouvez utiliser les applets de commande suivantes pour les stratégies d’application.</br></br>

| Applet de commande | Description | 
| --- | --- |
| [Add-AzureADApplicationPolicy](/powershell/module/azuread/add-azureadapplicationpolicy?view=azureadps-2.0-preview&preserve-view=true) | Lie la stratégie spécifiée à une application. |
| [Get-AzureADApplicationPolicy](/powershell/module/azuread/get-azureadapplicationpolicy?view=azureadps-2.0-preview&preserve-view=true) | Permet d’obtenir la stratégie affectée à une application. |
| [Remove-AzureADApplicationPolicy](/powershell/module/azuread/remove-azureadapplicationpolicy?view=azureadps-2.0-preview&preserve-view=true) | Supprime une stratégie d’une application. |

### <a name="service-principal-policies"></a>Stratégies de principal du service
Vous pouvez utiliser les applets de commande suivantes pour les stratégies de principal de service.

| Applet de commande | Description | 
| --- | --- |
| [Add-AzureADServicePrincipalPolicy](/powershell/module/azuread/add-azureadserviceprincipalpolicy?view=azureadps-2.0-preview&preserve-view=true) | Lie la stratégie spécifiée à un principal de service. |
| [Get-AzureADServicePrincipalPolicy](/powershell/module/azuread/get-azureadserviceprincipalpolicy?view=azureadps-2.0-preview&preserve-view=true) | Permet d’obtenir une stratégie liée au principal de service spécifié.|
| [Remove-AzureADServicePrincipalPolicy](/powershell/module/azuread/remove-azureadserviceprincipalpolicy?view=azureadps-2.0-preview&preserve-view=true) | Supprime la stratégie du principal de service spécifié.|

## <a name="license-requirements"></a>Conditions de licence :

L'utilisation de cette fonctionnalité nécessite une licence Azure AD Premium P1. Pour trouver la licence appropriée à vos besoins, consultez [Comparaison des fonctionnalités mises à la disposition générale des éditions Gratuite et Premium](https://azure.microsoft.com/pricing/details/active-directory/).

Les clients avec [des licences Microsoft 365 Business](/office365/servicedescriptions/microsoft-365-service-descriptions/microsoft-365-business-service-description) ont également accès aux fonctionnalités d’accès conditionnel.

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus, consultez des [exemples de configuration des durées de vie des jetons](configure-token-lifetimes.md).