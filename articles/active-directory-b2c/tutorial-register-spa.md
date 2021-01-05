---
title: 'Tutoriel : Inscrire une application monopage'
titleSuffix: Azure AD B2C
description: Ce tutoriel explique comment inscrire une application monopage (SPA) dans Azure Active Directory B2C à l’aide du portail Azure.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: tutorial
ms.date: 08/19/2020
ms.custom: project-no-code
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: d2eaf1dce432821dcfc693dc69dcf975a3d8be8d
ms.sourcegitcommit: 59f506857abb1ed3328fda34d37800b55159c91d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/24/2020
ms.locfileid: "92503859"
---
# <a name="tutorial-register-a-single-page-application-spa-in-azure-active-directory-b2c"></a>Tutoriel : Inscrire une application monopage (SPA) dans Azure Active Directory B2C

Pour que vos [applications](application-types.md) puissent interagir avec Azure Active Directory B2C (Azure AD B2C), elles doivent être inscrites dans un locataire que vous gérez. Ce tutoriel vous montre comment inscrire une application monopage (« SPA ») à l’aide du portail Azure.

## <a name="overview-of-authentication-options"></a>Vue d’ensemble des options d’authentification

De nombreuses applications web modernes sont créées en tant qu’applications monopages (« SPA ») côté client. Les développeurs les écrivent à l’aide de JavaScript ou d’un framework d’application monopage, comme Angular, Vue et React. Ces applications s’exécutent sur un navigateur web et présentent des caractéristiques d’authentification différentes de celles des applications web classiques côté serveur.

Azure AD B2C propose **deux** options qui permettent aux applications monopages d’effectuer la connexion des utilisateurs, et d’obtenir des jetons pour accéder aux services back-end ou aux API web :

### <a name="authorization-code-flow-with-pkce"></a>Flux de code d’autorisation (avec PKCE)
- [Flux de code d’autorisation OAuth 2.0 (avec PKCE)](./authorization-code-flow.md). Le flux de code d’autorisation permet à l’application d’échanger un code d’autorisation pour obtenir des jetons d’ **ID** afin de représenter l’utilisateur authentifié et des jetons d’ **accès** nécessaires pour appeler des API protégées. De plus, il retourne des jetons d’ **actualisation** qui fournissent à votre application un accès à long terme à des ressources au nom d’utilisateurs sans nécessiter l’intervention de ces utilisateurs. 

Il s’agit de l’approche **recommandée** . Le fait d’avoir des jetons d’actualisation à durée de vie limitée permet à votre application de s’adapter aux [restrictions de confidentialité des cookies des navigateurs modernes](../active-directory/develop/reference-third-party-cookies-spas.md), tels que Safari ITP.

Pour tirer parti de ce flux, votre application peut utiliser une bibliothèque d’authentification qui le prend en charge, comme [MSAL.js 2.x](https://github.com/Azure-Samples/ms-identity-b2c-javascript-spa). 

![Authentification des applications monopages](./media/tutorial-single-page-app/spa-app-auth.svg)

### <a name="implicit-grant-flow"></a>Octroi de flux implicite
- [Flux implicite OAuth 2.0](implicit-flow-single-page-application.md). Certains frameworks, tels que [MSAL.js 1.x](https://github.com/Azure-Samples/active-directory-b2c-javascript-msal-singlepageapp), ne prennent en charge que le flux d’octroi implicite. Le flux d’octroi implicite permet à l’application d’obtenir des jetons d’ **ID** et d’ **accès** . Contrairement au flux de code d’autorisation, le flux d’octroi implicite ne retourne pas de **jeton d’actualisation** . 

![Flux implicite des applications monopages](./media/tutorial-single-page-app/spa-app.svg)

Ce flux d’authentification n’inclut pas de scénarios d’application utilisant des frameworks JavaScript multiplateformes, tels qu’Electron et React-Native. Ces scénarios demandent d’autres fonctionnalités pour interagir avec les plateformes natives.

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

Si vous n’avez pas encore créé votre propre [locataire Azure AD B2C](tutorial-create-tenant.md), créez-en un maintenant. Vous pouvez utiliser un locataire Azure AD B2C existant.

## <a name="register-the-spa-application"></a>Inscrire l’application SPA

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sélectionnez l’icône **Annuaire et abonnement** dans la barre d’outils du portail, puis sélectionnez l’annuaire qui contient votre locataire Azure AD B2C.
1. Dans le portail Azure, recherchez et sélectionnez **Azure AD B2C** .
1. Sélectionnez **Inscriptions d’applications** , puis **Nouvelle inscription** .
1. Entrez un **Nom** pour l’application. Par exemple, *spaapp1* .
1. Sous **Types de comptes pris en charge** , sélectionnez **Comptes dans un fournisseur d’identité ou annuaire organisationnel (pour authentifier les utilisateurs avec des flux d’utilisateurs)** .
1. Sous **URI de redirection** , sélectionnez **Application monopage (SPA)** , puis entrez `https://jwt.ms` dans la zone de texte de l’URL.

    L’URI de redirection est le point de terminaison auquel l’utilisateur est envoyé par le serveur d’autorisation (dans ce cas, Azure AD B2C) après avoir terminé son interaction avec l’utilisateur, et auquel un jeton d’accès ou un code d’autorisation est envoyé en cas d’autorisation réussie. Dans une application de production, il s’agit généralement d’un point de terminaison accessible publiquement dans lequel votre application s’exécute, comme `https://contoso.com/auth-response`. À des fins de test comme dans ce didacticiel, vous pouvez le définir sur `https://jwt.ms`, une application web Microsoft qui affiche le contenu décodé d’un jeton (le contenu du jeton ne quitte jamais votre navigateur). Pendant le développement d’une application, vous pouvez ajouter le point de terminaison sur lequel votre application écoute localement, comme `http://localhost:5000`. Vous pouvez ajouter des URI de redirection à vos applications inscrites à tout moment et les modifier.

    Les restrictions suivantes s’appliquent aux URI de redirection :

    * L’URL de réponse doit commencer par le schéma `https`, sauf en cas d’utilisation de `localhost`.
    * L’URL de réponse respecte la casse. Sa casse doit correspondre à celle du chemin d’URL de votre application en cours d’exécution. Par exemple, si votre application comprend `.../abc/response-oidc` dans son chemin, ne spécifiez pas `.../ABC/response-oidc` dans l’URL de réponse. Étant donné que le navigateur web considère que les chemins respectent la casse, les cookies associés à `.../abc/response-oidc` peuvent être exclus s’ils sont redirigés vers l’URL `.../ABC/response-oidc` qui ne correspond pas à la casse.

1. Sous **Autorisations** , activez la case à cocher *Accorder le consentement administrateur aux autorisations openid et offline_access* .
1. Sélectionnez **Inscription** .


## <a name="enable-the-implicit-flow"></a>Activer le flux implicite
Si vous utilisez le flux implicite, vous devez activer le flux d’octroi implicite dans l’inscription de l’application.

1. Dans le menu de gauche, sous **Gérer** , sélectionnez **Authentification** .
1. Sous **Octroi implicite** , cochez les deux cases **Jetons d’accès** et **Jetons d’ID** .
1. Sélectionnez **Enregistrer** .

## <a name="migrate-from-the-implicit-flow"></a>Migrer à partir du flux implicite

Si vous avez une application qui se sert du flux implicite, nous vous recommandons de migrer avec le flux de code d’autorisation, en utilisant un framework qui le prend en charge comme [MSAL.js 2.x](https://github.com/AzureAD/microsoft-authentication-library-for-js/tree/dev/lib/msal-browser).

Lorsque toutes vos applications monopages de production, représentées par une inscription d’application, utilisent le flux de code d’autorisation, désactivez les paramètres du flux d’octroi implicite. 

1. Dans le menu de gauche, sous **Gérer** , sélectionnez **Authentification** .
1. Sous **Octroi implicite** , décochez les deux cases **Jetons d’accès** et **Jetons d’ID** .
1. Sélectionnez **Enregistrer** .

Il se peut que des applications utilisant le flux implicite continuent de fonctionner si vous laissez le flux implicite activé (coché).

* * *

## <a name="next-steps"></a>Étapes suivantes

Ensuite, découvrez comment créer des flux utilisateur pour permettre à vos utilisateurs de s’inscrire, se connecter et gérer leur profil.

> [!div class="nextstepaction"]
> [Créer des flux utilisateur dans Azure Active Directory B2C >](tutorial-create-user-flows.md)
