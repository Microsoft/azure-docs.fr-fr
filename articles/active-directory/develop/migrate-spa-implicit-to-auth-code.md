---
title: Migrer une application monopage JavaScript de l’octroi implicite vers le flux de code d’autorisation | Azure
titleSuffix: Microsoft identity platform
description: Explique comment mettre à jour une application monopage JavaScript utilisant MSAL.js 1.x et le flux d’octroi implicite vers MSAL.js 2.x et le flux de code d’autorisation avec prise en charge de PKCE et CORS.
services: active-directory
author: hahamil
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: how-to
ms.workload: identity
ms.date: 07/17/2020
ms.author: hahamil
ms.custom: aaddev, devx-track-js
ms.openlocfilehash: 05258e201c65138e53e861f0631eb33e08c9c199
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92673590"
---
# <a name="migrate-a-javascript-single-page-app-from-implicit-grant-to-auth-code-flow"></a>Migrer une application monopage JavaScript de l’octroi implicite vers le flux de code d’autorisation

La bibliothèque d’authentification Microsoft pour JavaScript (MSAL.js) v2.0 offre une prise en charge du flux de code d’autorisation avec PKCE et CORS pour les applications monopages sur la plateforme des identités Microsoft. Effectuez les étapes décrites dans les sections ci-dessous pour migrer votre application MSAL.js 1.x utilisant l’octroi implicite vers MSAL.js 2.0+ (nommé *2.x* dans la suite de cet article) et le flux de code d’authentification.

MSAL.js 2.x offre une amélioration par rapport à MSAL.js 1.x en prenant en charge le flux de code d’autorisation dans le navigateur au lieu du flux d’octroi implicite. MSAL.js 2.x ne prend **PAS** en charge le flux implicite.

## <a name="migration-steps"></a>Étapes de la migration

Pour mettre à jour votre application vers MSAL.js 2.x et le flux de code d’authentification, trois étapes principales sont nécessaires :

1. Basculer vos URI de redirection d’ [inscription d’application](#switch-redirect-uris-to-spa-platform) de la plateforme **web** vers la plateforme **Application monopage**
1. Mettre à jour votre [code](#switch-redirect-uris-to-spa-platform) de MSAL.js 1.x vers la version  **2.x**
1. Désactiver l’[octroi implicite](#disable-implicit-grant-settings) dans votre inscription d’application quand toutes les applications qui partagent l’inscription ont été mises à jour vers MSAL.js 2.x et le flux de code d’authentification

Les sections suivantes décrivent en détail chaque étape.

## <a name="switch-redirect-uris-to-spa-platform"></a>Basculer les URI de redirection vers la plateforme SPA

Si vous souhaitez continuer à utiliser votre inscription d’application existante pour vos applications, utilisez le portail Azure pour mettre à jour les URI de redirection de l’inscription vers la plateforme d’application monopage (SPA). Cela active le flux de code d’autorisation avec prise en charge de PKCE et CORS pour les applications qui utilisent l’inscription (vous devez quand même mettre à jour le code de votre application vers MSAL.js v2.x).

Effectuez ces étapes pour les inscriptions d’applications qui sont actuellement configurées avec des URI de redirection de plateforme **web**  :

1. Connectez-vous au [portail Azure](https://portal.azure.com) et sélectionnez votre locataire **Azure Active Directory** .
1. Dans **Inscriptions d’applications** , sélectionnez votre application, puis **Authentification** .
1. Dans la vignette de plateforme **web** sous **URI de redirection** , sélectionnez la bannière d’avertissement indiquant que vous devez migrer vos URI.

    :::image type="content" source="media/migrate-spa-implicit-to-auth-code/portal-01-implicit-warning-banner.png" alt-text="Bannière d’avertissement de flux implicite sur la vignette d’application web dans le portail Azure":::
1. Sélectionnez *uniquement* les URI de redirection dont les applications utiliseront MSAL.js 2.x, puis sélectionnez **Configurer** .

    :::image type="content" source="media/migrate-spa-implicit-to-auth-code/portal-02-select-redirect-uri.png" alt-text="Bannière d’avertissement de flux implicite sur la vignette d’application web dans le portail Azure":::

Ces URI de redirection doivent maintenant apparaître dans la vignette de plateforme **Application monopage** , indiquant que la prise en charge de CORS avec le flux de code d’autorisation et PKCE est activée pour ces URI.

:::image type="content" source="media/migrate-spa-implicit-to-auth-code/portal-03-spa-redirect-uri-tile.png" alt-text="Bannière d’avertissement de flux implicite sur la vignette d’application web dans le portail Azure":::

Vous pouvez également [créer une inscription d’application](scenario-spa-app-registration.md) au lieu de mettre à jour les URI de redirection dans votre inscription existante.

## <a name="update-your-code-to-msaljs-2x"></a>Mettre à jour votre code vers MSAL.js 2.x

Dans MSAL 1.x, pour créer une instance d’application vous deviez initialiser un [UserAgentApplication][msal-js-useragentapplication] comme suit :

```javascript
// MSAL 1.x
import * as msal from "msal";

const msalInstance = new msal.UserAgentApplication(config);
```

Dans MSAL 2.x, vous devez à la place initialiser un [PublicClientApplication][msal-js-publicclientapplication] :

```javascript
// MSAL 2.x
import * as msal from "@azure/msal-browser";

const msalInstance = new msal.PublicClientApplication(config);
```

Pour obtenir une vue d’ensemble complète de l’ajout de MSAL 2.x à votre application, consultez [Tutoriel : Connecter les utilisateurs et appeler l’API Microsoft Graph à partir d’une application monopage (SPA) JavaScript à l’aide du flux de code d’authentification](tutorial-v2-javascript-auth-code.md).

Pour connaître les modifications supplémentaires que vous devrez peut-être apporter à votre code, consultez le [guide de migration](https://github.com/AzureAD/microsoft-authentication-library-for-js/blob/dev/lib/msal-browser/docs/v1-migration.md) sur GitHub.

## <a name="disable-implicit-grant-settings"></a>Désactiver les paramètres d’octroi implicite

Une fois que vous avez mis à jour toutes vos applications de production qui utilisent cette inscription d’application et son ID client vers MSAL 2.x et le flux de code d’autorisation, vous devez désactiver les paramètres d’octroi implicite dans l’inscription d’application.

Quand vous désactivez les paramètres d’octroi implicite dans l’inscription d’application, le flux implicite est désactivé pour toutes les applications utilisant l’inscription et son ID client.

**Ne désactivez pas** le flux d’octroi implicite avant d’avoir mis à jour toutes vos applications vers MSAL.js 2.x et [PublicClientApplication][msal-js-publicclientapplication].

## <a name="next-steps"></a>Étapes suivantes

Pour en savoir plus sur le flux de code d’autorisation, notamment les différences entre les flux de code implicite et d’authentification, consultez [Plateforme d’identités Microsoft et flux de code d’autorisation OAuth 2.0](v2-oauth2-auth-code-flow.md).

Si vous souhaitez approfondir le développement d’applications monopages JavaScript sur la plateforme d’identités Microsoft, la série en plusieurs parties [Scénario : Application monopage](scenario-spa-overview.md) peut vous aider à commencer.

<!-- LINKS - external -->
[msal-js-useragentapplication]: https://azuread.github.io/microsoft-authentication-library-for-js/ref/msal-core/modules/_useragentapplication_.html
[msal-js-publicclientapplication]: https://azuread.github.io/microsoft-authentication-library-for-js/ref/msal-browser/classes/_src_app_publicclientapplication_.publicclientapplication.html
