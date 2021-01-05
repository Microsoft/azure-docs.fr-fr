---
title: Différences entre MSAL.js et ADAL.js | Azure
titleSuffix: Microsoft identity platform
description: Découvrez les différences entre la Bibliothèque d'authentification Microsoft pour JavaScript (MSAL.js) et la Bibliothèque d'authentification Azure AD pour JavaScript (ADAL.js), et apprenez à faire un choix entre les deux.
services: active-directory
author: navyasric
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.topic: conceptual
ms.workload: identity
ms.date: 04/10/2019
ms.author: nacanuma
ms.reviewer: saeeda
ms.custom: aaddev
ms.openlocfilehash: 5d91514af9aea3dd7ea13b94681fbb27c53772fe
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88120896"
---
# <a name="differences-between-msal-js-and-adal-js"></a>Différences entre MSAL JS et ADAL JS

La Bibliothèque d'authentification Microsoft pour JavaScript (MSAL.js) et la Bibliothèque d'authentification Azure AD pour JavaScript (ADAL.js) sont utilisées pour authentifier les entités Azure AD et demander des jetons à Azure AD. Jusqu'à présent, la plupart des développeurs utilisaient Azure AD pour développeurs (v1.0) pour authentifier les identités Azure AD (comptes professionnels et scolaires) en demandant des jetons avec ADAL. À présent, grâce à MSAL.js, vous pouvez authentifier un ensemble plus large d'identités Microsoft (identités Azure AD et comptes Microsoft, et comptes sociaux et locaux par le biais d'Azure AD B2C) via la plateforme d'identités Microsoft.

Cet article compare la Bibliothèque d'authentification Microsoft pour JavaScript (MSAL.js) et la Bibliothèque d'authentification Azure AD pour JavaScript (ADAL.js) et explique comment faire un choix entre les deux.

## <a name="choosing-between-adaljs-and-msaljs"></a>Choix entre ADAL.js et MSAL.js

Dans la plupart des cas, il est préférable d'utiliser la plateforme d'identités Microsoft et MSAL.js, qui correspond à la dernière génération de bibliothèques d'authentification de Microsoft. MSAL.js vous permet d'acquérir des jetons pour les utilisateurs qui se connectent à votre application avec Azure AD (comptes professionnels et scolaires), avec des comptes (personnels) Microsoft (MSA) ou avec Azure AD B2C.

Si vous connaissez déjà le point de terminaison v1.0 (et ADAL.js), n'hésitez pas à consulter l'article consacré aux [différences du point de terminaison v2.0](../azuread-dev/azure-ad-endpoint-comparison.md).

Mais vous devrez quand même utiliser ADAL.js si votre application a besoin de connecter des utilisateurs dotés de versions antérieures des [services de fédération Active Directory (AD FS)](/windows-server/identity/active-directory-federation-services).

## <a name="key-differences-in-authentication-with-msaljs"></a>Principales différences de l'authentification à l'aide de MSAL.js

### <a name="core-api"></a>API Core

* ADAL.js utilise [AuthenticationContext](https://github.com/AzureAD/azure-activedirectory-library-for-js/wiki/Config-authentication-context#authenticationcontext) comme représentation d'une instance de connexion de votre application au serveur d'autorisation ou au fournisseur d'identité via une URL d'autorité. L'API MSAL.js quant à elle est conçue autour d'une application cliente d'agent utilisateur (forme d'application cliente publique dans laquelle le code client est exécuté dans un agent utilisateur, comme un navigateur web). Elle fournit la classe `UserAgentApplication` pour représenter une instance du contexte d'authentification de l'application auprès du serveur d'autorisation. Pour plus d'informations, consultez [Initialiser à l'aide de MSAL.js](msal-js-initializing-client-applications.md).

* Dans ADAL.js, les méthodes d'acquisition de jetons sont associées à une seule autorité définie dans `AuthenticationContext`. Dans MSAL.js, les requêtes d'acquisition de jetons peuvent prendre des valeurs d'autorité différentes de celles définies dans `UserAgentApplication`. Cela permet à MSAL.js de procéder séparément à l'acquisition et à la mise en cache des jetons pour différents locataires et comptes d'utilisateur dans la même application.

* La méthode permettant d'acquérir et de renouveler des jetons en mode silencieux sans inviter les utilisateurs s'appelle `acquireToken` dans ADAL.js. Dans MSAL.js, cette méthode s'appelle `acquireTokenSilent`, pour mieux décrire cette fonctionnalité.

### <a name="authority-value-common"></a>Valeur d'autorité `common`

Dans la version 1.0, l'utilisation de l'autorité `https://login.microsoftonline.com/common` permet aux utilisateurs de se connecter avec n'importe quel compte Azure AD (pour n'importe quelle organisation).

Dans la version 2.0, l'utilisation de l'autorité `https://login.microsoftonline.com/common` permet aux utilisateurs de se connecter avec n'importe quel compte d'organisation Azure AD ou compte personnel Microsoft (MSA). Pour restreindre la connexion aux seuls comptes Azure AD (même comportement qu'avec ADAL.js), vous devez utiliser `https://login.microsoftonline.com/organizations`. Pour plus de détails, reportez-vous à l'option de configuration `authority` dans [Initialiser à l'aide de MSAL.js](msal-js-initializing-client-applications.md).

### <a name="scopes-for-acquiring-tokens"></a>Étendues pour l'acquisition de jetons
* Étendue au lieu du paramètre resource dans les demandes d'authentification pour l'acquisition de jetons

    Le protocole v2.0 utilise des étendues au lieu de ressources dans les demandes. En d'autres termes, lorsque votre application doit demander des jetons avec des autorisations pour une ressource telle que MS Graph, la différence en termes de valeurs transmises aux méthodes de bibliothèque est la suivante :

    v1.0 : ressource = https\://graph.microsoft.com

    v2.0 : étendue = https\://graph.microsoft.com/User.Read

    Vous pouvez demander des étendues pour n'importe quelle API de ressources à l'aide de l'URI de l'API au format suivant : appidURI/scope Par exemple : https:\//mytenant.onmicrosoft.com/myapi/api.read

    Pour l’API MS Graph uniquement, une valeur d’étendue `user.read` est mappée sur https:\//graph.microsoft.com/User.Read et peut être utilisée de manière interchangeable.

    ```javascript
    var request = {
        scopes = ["User.Read"];
    };

    acquireTokenPopup(request);   
    ```

* Étendues dynamiques pour un consentement incrémentiel.

    Lorsque vous génériez des applications à l'aide de la version 1.0, vous deviez enregistrer le jeu complet d'autorisations (étendues statiques) exigé par l'application pour que l'utilisateur donne son consentement au moment de la connexion. Dans la version 2.0, vous pouvez utiliser le paramètre scope pour demander les autorisations au moment où vous le souhaitez. C'est ce qu'on appelle des étendues dynamiques. Cela permet à l'utilisateur de fournir un consentement incrémentiel aux étendues. Par conséquent, si au début vous souhaitez juste que l'utilisateur se connecte à votre application et que vous n'avez besoin d'aucun type d'accès, c'est possible. Si, par la suite, vous devez avoir la possibilité de lire le calendrier de l'utilisateur, vous pouvez alors demander l'étendue de celui-ci dans les méthodes acquireToken et obtenir le consentement de l'utilisateur. Par exemple :

    ```javascript
    var request = {
        scopes = ["https://graph.microsoft.com/User.Read", "https://graph.microsoft.com/Calendar.Read"];
    };

    acquireTokenPopup(request);   
    ```

* Étendues pour les API V1.0

    Lors de l'obtention de jetons à l'aide de MSAL.js pour les API V1.0, vous pouvez demander toutes les étendues statiques enregistrées sur l'API en ajoutant `.default` à l'URI de l'ID d'application de l'API en tant qu'étendue. Par exemple :

    ```javascript
    var request = {
        scopes = [ appidURI + "/.default"];
    };

    acquireTokenPopup(request);
    ```

## <a name="next-steps"></a>Étapes suivantes
Pour plus d'informations, consultez la [comparaison entre v1.0 et v2.0](../azuread-dev/azure-ad-endpoint-comparison.md).