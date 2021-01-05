---
title: Authentification et autorisation pour l’API - Azure Time Series Insights | Microsoft Docs
description: Cet article décrit comment configurer l’authentification et l’autorisation pour une application personnalisée qui appelle l’API Azure Time Series Insights.
ms.service: time-series-insights
services: time-series-insights
author: deepakpalled
ms.author: shresha
manager: dpalled
ms.reviewer: v-mamcge, jasonh, kfile
ms.devlang: csharp
ms.workload: big-data
ms.topic: conceptual
ms.date: 10/02/2020
ms.custom: seodec18, has-adal-ref
ms.openlocfilehash: d1bd3c5796658663b6111723829cbe620346002c
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95016239"
---
# <a name="authentication-and-authorization-for-azure-time-series-insights-api"></a>Authentification et autorisation pour l’API Insights Azure Time Series

Ce document explique comment inscrire une application dans Azure Active Directory à l'aide du nouveau panneau Azure Active Directory. Les applications inscrites dans Azure Active Directory permettent aux utilisateurs de s'authentifier et d'utiliser l'API Azure Time Series Insight associée à un environnement Azure Time Series Insights.

## <a name="service-principal"></a>Principal du service

Les sections suivantes expliquent comment configurer une application pour accéder à l’API Azure Time Series Insights pour le compte d’une application. L’application peut ensuite interroger ou publier des données de référence dans l’environnement Azure Time Series Insights avec ses propres informations d’identification par le biais d’Azure Active Directory.

## <a name="summary-and-best-practices"></a>Résumé et meilleures pratiques

Le flux d'inscription de l'application Azure Active Directory implique trois étapes principales.

1. [Inscrire une application](#azure-active-directory-app-registration) dans Azure Active Directory.
1. Autoriser l’application à accéder aux [données dans l’environnement Azure Time Series Insights](#granting-data-access).
1. Utiliser l'**ID d’application** et la **clé secrète client** pour obtenir un jeton depuis `https://api.timeseries.azure.com/` dans votre [application cliente](#client-app-initialization). Le jeton permet ensuite d’appeler l’API Azure Time Series Insights.

Comme indiqué à l'**étape 3**, séparer les informations d'identification de l'utilisateur et de l'application permet ce qui suit :

* Attribuer à l’identité de l’application des autorisations différentes de vos propres autorisations. En règle générale, ces autorisations sont strictement limitées à ce que l’application nécessite. Par exemple, vous pouvez autoriser l’application à lire uniquement des données dans un environnement Azure Time Series Insights particulier.
* Isoler la sécurité de l’application de la création des informations d'identification d'un utilisateur en utilisant une **clé secrète client** ou un certificat de sécurité. Ainsi, les informations d’identification de l’application ne dépendent pas des informations d’identification d’un utilisateur spécifique. Si le rôle de l’utilisateur évolue, l’application ne requiert pas nécessairement de nouvelles informations d’identification ou une configuration supplémentaire. Si l’utilisateur modifie son mot de passe, tous les accès à l’application ne requièrent pas de nouvelles informations d’identification ou clés.
* Exécuter un script sans assistance en utilisant une **clé secrète client** ou un certificat de sécurité plutôt que les informations d’identification d’un utilisateur spécifique (ce qui implique qu'il soit présent).
* Utiliser un certificat de sécurité plutôt qu’un mot de passe pour sécuriser l’accès à votre API Azure Time Series Insights.

> [!IMPORTANT]
> Suivez le principe de **séparation des problèmes** (décrit dans ce scénario ci-dessus) lorsque vous configurez votre stratégie de sécurité Azure Time Series Insights.

> [!NOTE]

> * L'article se concentre sur une application à client unique conçue pour s’exécuter au sein d’une seule organisation.
> * Les applications à client unique sont généralement utilisées pour des applications métier exécutées au sein de votre organisation.

## <a name="detailed-setup"></a>Configuration détaillée

### <a name="azure-active-directory-app-registration"></a>Inscription d'application Azure Active Directory

[!INCLUDE [Azure Active Directory app registration](../../includes/time-series-insights-aad-registration.md)]

### <a name="granting-data-access"></a>Octroi d'un accès aux données

1. Pour l’environnement Azure Time Series Insights, sélectionnez **Stratégies d’accès aux données**, puis **Ajouter**.

   [![Ajouter une nouvelle stratégie d’accès aux données à l’environnement Azure Time Series Insights](media/authentication-and-authorization/time-series-insights-data-access-policies-add.png)](media/authentication-and-authorization/time-series-insights-data-access-policies-add.png#lightbox)

1. Dans la boîte de dialogue **Sélectionner un utilisateur**, collez le **Nom de l'application** ou l'**ID de l'application** à partir de la section d'inscription de l'application Azure Active Directory.

   [![Rechercher une application dans la boîte de dialogue Sélectionner un utilisateur](media/authentication-and-authorization/time-series-insights-data-access-policies-select-user.png)](media/authentication-and-authorization/time-series-insights-data-access-policies-select-user.png#lightbox)

1. Sélectionnez le rôle. Sélectionnez **Lecteur** pour interroger des données ou **Contributeur** pour interroger des données et modifier les données de référence. Sélectionnez **OK**.

   [![Choisir Lecteur ou Contributeur dans la boîte de dialogue Sélectionner un rôle utilisateur](media/authentication-and-authorization/time-series-insights-data-access-policies-select-role.png)](media/authentication-and-authorization/time-series-insights-data-access-policies-select-role.png#lightbox)

1. Enregistrez la stratégie en sélectionnant **OK**.

   > [!TIP]
   > Pour plus d’informations sur les options avancées d’accès aux données, consultez [Octroi d’un accès aux données](./concepts-access-policies.md).

### <a name="client-app-initialization"></a>Initialisation de l'application cliente

* Les développeurs peuvent utiliser la Bibliothèque d’authentification Microsoft (MSAL) pour s’authentifier auprès d’Azure Time Series Insights.

* Pour s’authentifier à l’aide de la bibliothèque MSAL :

   1. Utilisez l'**ID de l'application** et la **clé secrète client** (clé de l'application) à partir de la section d'inscription de l'application Azure Active Directory afin d'obtenir le jeton pour le compte de l’application.

   1. Dans C#, le code suivant permet d'obtenir le jeton pour le compte de l’application. Pour obtenir un exemple complet sur la façon d’interroger des données à partir d’un environnement Gen1, consultez [Interroger des données à l’aide de C#](time-series-insights-query-data-csharp.md).

        Consultez le référentiel [Azure Time Series Insights](https://github.com/Azure-Samples/Azure-Time-Series-Insights/blob/master/gen1-sample/csharp-tsi-gen1-sample/Program.cs)] pour accéder au code C#.

   1. Le jeton peut ensuite être passé dans l’en-tête `Authorization` lorsque l’application appelle l’API Azure Time Series Insights.

> [!IMPORTANT]
> Si vous utilisez la [Bibliothèque d'authentification Active Directory (ADAL)](../active-directory/azuread-dev/active-directory-authentication-libraries.md) consultez la section [Migration vers MSAL](../active-directory/develop/msal-net-migration.md).

## <a name="common-headers-and-parameters"></a>Paramètres et en-têtes communs

Cette section décrit les en-têtes de requête HTTP et les paramètres qui sont couramment utilisés pour exécuter des requêtes sur les API Azure Time Series Insights Gen1 et Gen2. Les exigences spécifiques aux API sont décrites plus en détail dans la [documentation de référence sur l’API REST Azure Time Series Insights](/rest/api/time-series-insights/).

> [!TIP]
> Lisez les [Informations de référence sur l’API REST Azure](/rest/api/azure/) pour en savoir plus sur l’utilisation des API REST, la création de requêtes HTTP et la gestion des réponses HTTP.

### <a name="authentication"></a>Authentification

Pour exécuter des requêtes authentifiées sur les [API Azure Time Series Insights](/rest/api/time-series-insights/), un jeton de porteur OAuth 2.0 valide doit être passé dans l’[en-tête d’autorisation](/rest/api/apimanagement/2019-12-01/authorizationserver/createorupdate) à l’aide du client REST de votre choix (Postman, JavaScript, C#).

> [!TIP]
> Pour savoir comment s’authentifier par programmation auprès des API Azure Time Series Insights à l’aide du [Kit de développement logiciel (SDK) JavaScript Client](https://github.com/microsoft/tsiclient/blob/master/docs/API.md), lisez l’[exemple de visualisation hébergé du Kit de développement logiciel (SDK) client](https://tsiclientsample.azurewebsites.net/) Azure Time Series Insights ainsi que des graphes et autres graphiques.

### <a name="http-headers"></a>En-têtes HTTP

Les en-têtes de requête obligatoires sont décrits ci-dessous.

| En-tête de requête obligatoire | Description |
| --- | --- |
| Autorisation | Pour s’authentifier auprès d’Azure Time Series Insights, un jeton de porteur OAuth 2.0 valide doit être passé dans l’en-tête d’**autorisation**. |

> [!IMPORTANT]
> Le jeton doit être émis exactement vers la ressource `https://api.timeseries.azure.com/` (également appelée « audience » du jeton).

> * Votre **AuthURL** [Postman](https://www.getpostman.com/) sera donc : `https://login.microsoftonline.com/microsoft.onmicrosoft.com/oauth2/authorize?scope=https://api.timeseries.azure.com//.default`
> * `https://api.timeseries.azure.com/` est valide, mais `https://api.timeseries.azure.com` ne l’est pas.

Les en-têtes de requête facultatifs sont décrits ci-dessous.

| En-tête de demande facultatif. | Description |
| --- | --- |
| Content-Type | Seul `application/json` est pris en charge. |
| x-ms-client-request-id | ID de requête client. Le service enregistre cette valeur. Permet au service de suivre l’opération d’un service à l’autre. |
| x-ms-client-session-id | ID de session du client. Le service enregistre cette valeur. Permet au service de suivre un groupe d’opérations connexes d’un service à l’autre. |
| x-ms-client-application-name | Nom de l’application qui a généré cette requête. Le service enregistre cette valeur. |

Les en-têtes de réponse facultatifs mais recommandés sont décrits ci-dessous.

| En-tête de réponse | Description |
| --- | --- |
| Content-Type | Seul `application/json` est pris en charge. |
| x-ms-request-id | ID de requête généré par le serveur. Peut être utilisé pour demander à Microsoft d’examiner une requête. |
| x-ms-property-not-found-behavior | En-tête de réponse facultatif de l’API GA. Les valeurs possibles sont `ThrowError` (par défaut) ou `UseNull`. |

### <a name="http-parameters"></a>Paramètres HTTP

> [!TIP]
> Pour plus d’informations sur les informations de requête obligatoires et facultatives, consultez la [documentation de référence](/rest/api/time-series-insights/).

Les paramètres de chaîne de requête d’URL obligatoires dépendent de la version de l’API.

| Libérer | Valeurs de version d’API possibles |
| --- |  --- |
| Première génération | `api-version=2016-12-12`|
| Deuxième génération | `api-version=2020-07-31` et `api-version=2018-11-01-preview`|

> [!IMPORTANT]
>
> La version de `api-version=2018-11-01-preview` sera bientôt dépréciée. Nous recommandons aux utilisateurs de passer à la version la plus récente.

Les paramètres de chaîne de requête d’URL facultatifs incluent la définition d’un délai d’expiration pour les durées d’exécution des requêtes HTTP.

| Paramètre de requête facultatif | Description | Version |
| --- |  --- | --- |
| `timeout=<timeout>` | Délai d’attente côté serveur pour l’exécution des requêtes HTTP. S’applique uniquement aux API [Get Environment Events](/rest/api/time-series-insights/dataaccess(preview)/query/getavailability) et [Get Environment Aggregates](/rest/api/time-series-insights/gen1-query-api#get-environment-aggregates-api). La valeur du délai d’attente doit être au format de durée ISO 8601, par exemple `"PT20S"`, et doit être comprise dans la plage `1-30 s`. La valeur par défaut est `30 s`. | Première génération |
| `storeType=<storeType>` | Pour les environnements Gen2 avec le magasin chaud activé, la requête peut être exécutée sur `WarmStore` ou `ColdStore`. Ce paramètre dans la requête définit le magasin sur lequel la requête doit être exécutée. S’il n’est pas défini, la requête est exécutée sur le magasin Cold. Pour interroger le magasin Warm, **storeType** doit être défini sur `WarmStore`. S’il n’est pas défini, la requête est exécutée par rapport au magasin Cold. | Deuxième génération |

## <a name="next-steps"></a>Étapes suivantes

* Pour un exemple de code qui appelle l’API Azure Time Series Insights Gen1 mise à la disposition générale, lisez [Interroger des données Gen1 à l’aide de C#](./time-series-insights-query-data-csharp.md).

* Pour un exemple de code qui appelle l’API Azure Time Series Insights Gen2, lisez [Interroger des données Gen2 à l’aide de C#](./time-series-insights-update-query-data-csharp.md).

* Pour obtenir des informations de référence sur l’API, lisez la documentation de [référence sur l’API de requête](/rest/api/time-series-insights/gen1-query-api).

* Découvrez comment [créer un principal de service](../active-directory/develop/howto-create-service-principal-portal.md).