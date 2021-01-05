---
title: Point de terminaison UserInfo de la plateforme d’identités Microsoft | Azure
titleSuffix: Microsoft identity platform
description: Découvrez le point de terminaison UserInfo sur la plateforme d’identités Microsoft.
services: active-directory
author: hpsin
manager: CelesteDG
ms.service: active-directory
ms.subservice: develop
ms.workload: identity
ms.topic: conceptual
ms.date: 09/21/2020
ms.author: hirsin
ms.reviewer: hirsin
ms.custom: aaddev
ms.openlocfilehash: 8f3fd462a52b035cd5b5447560e5472b41f237fa
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91653228"
---
# <a name="microsoft-identity-platform-userinfo-endpoint"></a>Point de terminaison UserInfo de la plateforme d’identités Microsoft

Le point de terminaison UserInfo fait partie de la [norme OpenID Connect](https://openid.net/specs/openid-connect-core-1_0.html#UserInfo) (OIDC), conçue pour retourner des revendications sur l’utilisateur qui s’est authentifié.  Pour la plateforme d’identités Microsoft, le point de terminaison UserInfo est hébergé sur Microsoft Graph (https://graph.microsoft.com/oidc/userinfo). 

## <a name="find-the-well-known-configuration-endpoint"></a>Rechercher le point de terminaison de configuration bien connu

Vous pouvez découvrir par programmation le point de terminaison UserInfo à l’aide du document de découverte OpenID Connect, à cet emplacement `https://login.microsoftonline.com/common/v2.0/.well-known/openid-configuration`. Il est indiqué dans le champ `userinfo_endpoint` ; ce modèle peut être utilisé dans les clouds pour aider à pointer vers le point de terminaison approprié.  Nous déconseillons de coder en dur le point de terminaison UserInfo dans votre application, utilisez plutôt le document de découverte OIDC pour rechercher ce point de terminaison au moment de l’exécution.

Dans le cadre de la spécification OpenID Connect, le point de terminaison UserInfo est souvent appelé automatiquement par des [bibliothèques conformes OIDC](https://openid.net/developers/certified/) pour obtenir des informations sur l’utilisateur.  Sans héberger un tel point de terminaison, la plateforme d’identités Microsoft ne serait pas conforme aux normes, et certaines bibliothèques échoueraient.  À partir de la [liste des revendications identifiées dans la norme OIDC](https://openid.net/specs/openid-connect-core-1_0.html#StandardClaims), nous produisons les revendications de nom, la revendication de l’objet et l’adresse e-mail, lorsqu’elles sont disponibles et consenties.  

## <a name="consider-use-an-id-token-instead"></a>Vous devez : Utiliser un jeton d’ID à la place

Les informations disponibles dans le jeton d’ID que votre application peut recevoir sont un sur-ensemble des informations qu’il peut obtenir du point de terminaison UserInfo.  Dès lors que vous pouvez obtenir un jeton d’ID en même temps que vous obtenez un jeton permettant d’appeler le point de terminaison UserInfo, nous vous suggérons d’utiliser ce jeton d’ID pour obtenir des informations sur l’utilisateur au lieu d’appeler le point de terminaison UserInfo.  L’utilisation du jeton d’ID supprimera une à deux requêtes réseau du lancement de votre application, ce qui réduira la latence dans votre application.

Si vous avez besoin de détails supplémentaires sur l’utilisateur, appelez l’[API Microsoft Graph `/user` ](/graph/api/user-get) pour obtenir des informations telles que le numéro de téléphone du bureau ou le poste.   Vous pouvez également utiliser des [revendications facultatives](active-directory-optional-claims.md) pour inclure des informations utilisateur supplémentaires dans vos jetons d’ID et d’accès.

## <a name="calling-the-userinfo-endpoint"></a>Appel du point de terminaison UserInfo

UserInfo est une API de jeton de porteur OAuth standard, appelée comme n’importe quelle autre API Microsoft Graph, à l’aide du jeton d’accès reçu lors de l’obtention d’un jeton pour Microsoft Graph. Elle retourne une réponse JSON contenant les revendications relatives à l’utilisateur.

### <a name="permissions"></a>Autorisations

Utilisez les [autorisations OIDC](v2-permissions-and-consent.md#openid-connect-scopes) suivantes pour appeler l’API UserInfo. `openid` est obligatoire, et les étendues `profile` et `email` garantissent que des informations supplémentaires sont fournies dans la réponse.

|Type d'autorisation      | Autorisations    |
|:--------------------|:---------------------------------------------------------|
|Déléguée (compte professionnel ou scolaire) | openid (obligatoire), profil, e-mail |
|Déléguée (compte Microsoft personnel) | openid (obligatoire), profil, e-mail |
|Application | Non applicable |

> [!TIP]
> Copiez cette URL dans votre navigateur afin d’obtenir un jeton pour le point de terminaison UserInfo ainsi qu’un [jeton d’ID](id-tokens.md), puis remplacez l’ID client et l’URI de redirection par les vôtres. Notez qu’elle ne demande que des étendues pour les étendues OpenID ou Graph, et rien d’autre.  Cela est nécessaire, car vous ne pouvez pas demander d’autorisations pour deux ressources différentes dans la même demande de jeton.
>
> `https://login.microsoftonline.com/common/oauth2/v2.0/authorize?client_id=<yourClientID>&response_type=token+id_token&redirect_uri=<YourRedirectUri>&scope=user.read+openid+profile+email&response_mode=fragment&state=12345&nonce=678910`
>
> Vous pouvez utiliser ce jeton d’accès à la section suivante.

Comme pour tout autre jeton Microsoft Graph, le jeton que vous recevez ici peut ne pas être un jeton JWT. Si vous avez connecté un utilisateur de compte Microsoft, il s’agira d’un format de jeton chiffré. Cela est dû au fait que Microsoft Graph dispose d’un modèle d’émission de jeton spécial. Cela n’a aucune incidence sur votre capacité à utiliser le jeton d’accès pour appeler le point de terminaison UserInfo.

### <a name="calling-the-api"></a>Appel de l’API

L’API UserInfo prend en charge GET et POST, conformément à la spécification OIDC.

```http
GET or POST /oidc/userinfo HTTP/1.1
Host: graph.microsoft.com
Authorization: Bearer eyJ0eXAiOiJKV1QiLCJub25jZSI6Il…
```

### <a name="userinfo-response"></a>Réponse UserInfo

```jsonc
{
    "sub": "OLu859SGc2Sr9ZsqbkG-QbeLgJlb41KcdiPoLYNpSFA",
    "name": "Mikah Ollenburg", // names all require the “profile” scope.
    "family_name": " Ollenburg",
    "given_name": "Mikah",
    "email": "mikoll@contoso.com" //requires the “email” scope.
}
```

Les revendications répertoriées ici sont toutes les revendications que le point de terminaison UserInfo peut retourner.  Il s’agit des mêmes valeurs que celles que l’application voit dans le [jeton d’ID](id-tokens.md) émis à l’application.  

## <a name="notes-and-caveats-on-the-userinfo-endpoint"></a>Remarques et avertissements sur le point de terminaison UserInfo

* Si vous souhaitez appeler ce point de terminaison UserInfo, vous devez utiliser le point de terminaison v2.0.  Si vous utilisez le point de terminaison v1.0, vous obtiendrez un jeton pour le point de terminaison UserInfo v1.0, qui est hébergé sur login.microsoftonline.com.  Nous recommandons que toutes les applications et bibliothèques conformes à OIDC utilisent le point de terminaison v2.0 pour garantir la compatibilité.
* La réponse du point de terminaison UserInfo ne peut pas être personnalisée.  Si vous souhaitez personnaliser les revendications, utilisez le [mappage de revendications]( active-directory-claims-mapping.md) pour modifier les informations retournées dans les jetons.
* Il est impossible d’ajouter quelque chose à la réponse du point de terminaison UserInfo.  Si vous souhaitez obtenir des revendications supplémentaires sur l’utilisateur, utilisez les [revendications facultatives]( active-directory-optional-claims.md) pour ajouter de nouvelles revendications aux jetons.

## <a name="next-steps"></a>Étapes suivantes

* [Examiner le contenu des jetons d’ID](id-tokens.md)
* [Personnaliser le contenu d’un jeton d’ID à l’aide des revendications facultatives](active-directory-optional-claims.md)
* [Demander un jeton d’accès et un jeton d’ID à l’aide du protocole OAuth2](v2-protocols-oidc.md)
