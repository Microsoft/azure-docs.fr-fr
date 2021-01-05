---
title: Inscrire des applications monopages | Azure
titleSuffix: Microsoft identity platform
description: Découvrir comment créer une application monopage (inscription d’application)
services: active-directory
author: hahamil
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 05/19/2020
ms.author: hahamil
ms.custom: aaddev
ms.openlocfilehash: e42b6b7ffa39110ce356094de690bb353a3fb10f
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97107785"
---
# <a name="single-page-application-app-registration"></a>Application monopage : Inscription d'application

Pour inscrire une application monopage dans la plateforme d’identité Microsoft, procédez comme suit. Les étapes d’inscription diffèrent entre MSAL.js 1.0, qui prend en charge le flux d’octroi implicite, et MSAL.js 2.0, qui prend en charge le flux du code d’autorisation avec PKCE.

## <a name="create-the-app-registration"></a>Créer l’inscription d’application

Pour les applications basées sur MSAL.js 1.0 et 2.0, commencez par suivre les étapes suivantes pour créer l’inscription d’application initiale.

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Si vous avez accès à plusieurs locataires, utilisez le filtre **Répertoire + abonnement** :::image type="icon" source="./media/common/portal-directory-subscription-filter.png" border="false"::: dans le menu du haut pour sélectionner le locataire dans lequel vous voulez inscrire une application.
1. Recherchez et sélectionnez **Azure Active Directory**.
1. Sous **Gérer**, sélectionnez **Inscriptions d’applications** > **Nouvelle inscription**.
1. Entrez un **nom** pour votre application. Les utilisateurs de votre application peuvent voir ce nom, et vous pouvez le changer ultérieurement.
1. Choisissez les **Types de comptes pris en charge** pour l’application. N’entrez **PAS** d’**URI de redirection**. Pour obtenir une description des différents types de comptes, consultez [Inscrire une application](quickstart-register-app.md).
1. Sélectionnez **Inscrire** pour l’inscription d’application.

Ensuite, configurez l’inscription d’application avec un **URI de redirection** pour spécifier l’emplacement vers où la plateforme d’identité Microsoft doit rediriger le client, ainsi que tous les jetons de sécurité. Suivez les étapes appropriées pour la version de MSAL.js que vous utilisez dans votre application :

- [MSAL.js 2.0 avec flux de code d’autorisation](#redirect-uri-msaljs-20-with-auth-code-flow) (recommandé)
- [MSAL.js 1.0 avec flux implicite](#redirect-uri-msaljs-10-with-implicit-flow)

## <a name="redirect-uri-msaljs-20-with-auth-code-flow"></a>URI de redirection : [MSAL.js 2.0 avec flux de code d’autorisation](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-browser)

Procédez comme suit pour ajouter un URI de redirection pour une application qui utilise MSAL.js 2.0 ou version ultérieure. MSAL.js 2.0+ prend en charge le flux de code d’autorisation avec PKCE et CORS en réponse aux [restrictions de cookies tierces du navigateur](reference-third-party-cookies-spas.md). Le flux d’octroi implicite n’est pas pris en charge dans MSAL.js 2.0+.

1. Dans le portail Azure, sélectionnez l’inscription d’application que vous avez créée à l’étape [Créer l’inscription d’application](#create-the-app-registration).
1. Sous **Gérer**, sélectionnez **Authentification** > **Ajouter une plateforme**.
1. Sous **Applications web**, sélectionnez la vignette **Application monopage**.
1. Sous **URI de redirection**, entrez un [URI de redirection](reply-url.md). N’activez **AUCUNE** des cases à cocher sous **Octroi implicite**.
1. Sélectionnez **Configurer** pour terminer l’ajout de l’URI de redirection.

Vous avez maintenant terminé l’inscription de votre application monopage, et avez configuré un URI de redirection vers lequel le client sera redirigé et auquel tous les jetons de sécurité seront envoyés. Si vous configurez votre URI de redirection à l’aide de la vignette **Application monopage** dans le volet **Ajouter une plateforme**, l’inscription de votre application est configurée pour prendre en charge le flux de code d’autorisation avec PKCE et CORS.

Pour plus de conseils, suivez le [tutoriel](tutorial-v2-javascript-auth-code.md).

## <a name="redirect-uri-msaljs-10-with-implicit-flow"></a>URI de redirection : [MSAL.js 1.0 avec flux implicite](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-core)

Procédez comme suit pour ajouter un URI de redirection pour une application monopage qui utilise MSAL.js, version 1.3 ou antérieure, et le flux d’octroi implicite. Les applications qui utilisent MSAL.js, version 1.3 ou antérieure, ne prennent pas en charge le flux de code d’autorisation.

1. Dans le portail Azure, sélectionnez l’inscription d’application que vous avez créée à l’étape [Créer l’inscription d’application](#create-the-app-registration).
1. Sous **Gérer**, sélectionnez **Authentification** > **Ajouter une plateforme**.
1. Sous **Applications web**, sélectionnez la vignette **Application monopage**.
1. Sous **URI de redirection**, entrez un [URI de redirection](reply-url.md).
1. Activez le **Flux implicite** :
    - Si votre application connecte des utilisateurs, sélectionnez **Jetons d’ID**.
    - Si votre application doit également appeler une API web protégée, sélectionnez **Jetons d’accès**. Pour plus d’informations sur ces types de jetons, consultez [Jetons d’ID](id-tokens.md) et [Jetons d’accès](access-tokens.md).
1. Sélectionnez **Configurer** pour terminer l’ajout de l’URI de redirection.

Vous avez maintenant terminé l’inscription de votre application monopage, et avez configuré un URI de redirection vers lequel le client sera redirigé et auquel tous les jetons de sécurité seront envoyés. En sélectionnant l’une des options **Jetons d’ID** ou **Jetons d’accès**, vous avez activé le flux d’octroi implicite.

Pour plus de conseils, suivez le [tutoriel](tutorial-v2-javascript-spa.md).

## <a name="note-about-authorization-flows"></a>Remarque à propos des flux d’autorisation

Par défaut, une inscription d’application créée via une configuration de plateforme d’application monopage active le flux de code d’autorisation. Pour tirer parti de ce flux, votre application doit utiliser MSAL.js, version 2.0 ou ultérieure.

Comme mentionné précédemment, les applications monopages utilisant MSAL.js version 1.3 sont limitées au flux d’octroi implicite. Les [Meilleures pratiques d’OAuth 2.0](v2-oauth2-auth-code-flow.md) actuelles recommandent d’utiliser le flux code d’autorisation plutôt que le flux implicite pour les applications monopages. Le fait d’avoir des jetons d’actualisation à durée de vie limitée permet également à votre application de s’adapter aux [restrictions de confidentialité des cookies des navigateurs modernes](reference-third-party-cookies-spas.md), tels que Safari ITP.

Lorsque toutes vos applications monopages de production représentées par une inscription d’application utilisent MSAL.js 2.0 et le flux de code d’autorisation, désactivez les paramètres d’octroi implicite dans le volet **Authentification** de l’inscription d’application sur le portail Azure. Il se peut néanmoins que des applications utilisant MSAL.js 1.x et le flux implicite continuent de fonctionner si vous laissez le flux implicite activé (coché).

## <a name="next-steps"></a>Étapes suivantes

Configurez le code de votre application pour utiliser l’inscription d’application que vous avez créée aux étapes précédentes : [Configuration du code de l’application](scenario-spa-app-configuration.md).
