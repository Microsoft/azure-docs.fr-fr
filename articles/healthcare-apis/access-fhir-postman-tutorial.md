---
title: Serveur FHIR Postman dans Azure – API Azure pour FHIR
description: Dans ce tutoriel, nous allons parcourir une à une les étapes nécessaires pour accéder à un serveur FHIR à l’aide de Postman. Postman est utile pour déboguer les applications qui accèdent à des API.
services: healthcare-apis
ms.service: healthcare-apis
ms.subservice: fhir
ms.topic: tutorial
ms.reviewer: dseven
ms.author: matjazl
author: matjazl
ms.date: 02/07/2019
ms.openlocfilehash: 37086a925d65a80f219c8e0a3358c834a2691701
ms.sourcegitcommit: 4b76c284eb3d2b81b103430371a10abb912a83f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/01/2020
ms.locfileid: "93145648"
---
# <a name="access-azure-api-for-fhir-with-postman"></a>Accéder à l’API Azure pour FHIR avec Postman

Une application cliente accède à une API FHIR par l’intermédiaire d’une [API REST](https://www.hl7.org/fhir/http.html). Vous pouvez aussi interagir directement avec le serveur FHIR pendant que vous créez des applications, par exemple, à des fins de débogage. Dans ce tutoriel, nous allons parcourir une à une les étapes nécessaires pour accéder à un serveur FHIR à l’aide de [Postman](https://www.getpostman.com/). Postman est un outil souvent utilisé pour déboguer les applications en phase de développement et qui accèdent à des API.

## <a name="prerequisites"></a>Prérequis

- Un point de terminaison FHIR dans Azure. Vous pouvez le configurer à l’aide de l’API Azure pour FHIR managée ou du serveur FHIR pour Azure open source. Configurez l’API Azure pour FHIR managée à partir du [portail Azure](fhir-paas-portal-quickstart.md), de [PowerShell](fhir-paas-powershell-quickstart.md) ou d’[Azure CLI](fhir-paas-cli-quickstart.md).
- Une [application cliente](register-confidential-azure-ad-client-app.md) que vous utiliserez pour accéder au service FHIR.
- Postman installé. Vous pouvez vous le procurer à l’adresse [https://www.getpostman.com](https://www.getpostman.com).

## <a name="fhir-server-and-authentication-details"></a>Serveur FHIR et informations d’authentification

Pour pouvoir utiliser Postman, les informations suivantes sont nécessaires :

- L’URL de votre serveur FHIR, par exemple `https://MYACCOUNT.azurehealthcareapis.com`
- L’élément `Authority` du fournisseur d’identité de votre serveur FHIR, par exemple, `https://login.microsoftonline.com/{TENANT-ID}`
- L’élément `audience` configuré. Il s’agit généralement de l’URL du serveur FHIR, par exemple `https://MYACCOUNT.azurehealthcareapis.com` ou simplement `https://azurehealthcareapis.com`.
- L’élément `client_id` (ou ID d’application) de l’[application cliente](register-confidential-azure-ad-client-app.md) que prévoyez d’utiliser pour accéder au service FHIR.
- L’élément `client_secret` (ou secret d’application) de l’application cliente.

Enfin, vous devez vérifier que `https://www.getpostman.com/oauth2/callback` est une URL de réponse inscrite pour votre application cliente.

## <a name="connect-to-fhir-server"></a>Se connecter au serveur FHIR

À partir de Postman, adressez une demande `GET` à `https://fhir-server-url/metadata` :

![Énoncé de capacité des métadonnées Postman](media/tutorial-postman/postman-metadata.png)

L’URL des métadonnées de l’API Azure pour FHIR est `https://MYACCOUNT.azurehealthcareapis.com/metadata`. Dans cet exemple, l’URL du serveur FHIR est `https://fhirdocsmsft.azurewebsites.net` et l’énoncé de capacité du serveur est disponible à l’adresse `https://fhirdocsmsft.azurewebsites.net/metadata`. Ce point de terminaison est normalement accessible sans authentification.

Si vous tentez d’accéder à des ressources restreintes, vous devriez obtenir une réponse de type « Échec de l’authentification » :

![Échec de l’authentification](media/tutorial-postman/postman-authentication-failed.png)

## <a name="obtaining-an-access-token"></a>Obtention d’un jeton d’accès

Pour obtenir un jeton d’accès valide, sélectionnez « Authorization » et choisissez le TYPE « OAuth 2.0 » :

![Définir OAuth 2.0](media/tutorial-postman/postman-select-oauth2.png)

Cliquez sur « Get New Access Token » (Obtenir un nouveau jeton d’accès) ; une boîte de dialogue apparaît alors :

![Demander un nouveau jeton d’accès](media/tutorial-postman/postman-request-token.png)

Vous avez besoin de fournir certains détails :

| Champ                 | Exemple de valeur                                                                                                   | Commentaire                    |
|-----------------------|-----------------------------------------------------------------------------------------------------------------|----------------------------|
| Token Name (Nom du jeton)            | MYTOKEN                                                                                                         | Nom de votre choix          |
| Grant Type (Type d’autorisation)            | Code d’autorisation                                                                                              |                            |
| Callback URL (URL de rappel)          | `https://www.getpostman.com/oauth2/callback`                                                                      |                            |
| Auth URL (URL d’authentification)              | `https://login.microsoftonline.com/{TENANT-ID}/oauth2/authorize?resource=<audience>` | `audience` a la valeur `https://MYACCOUNT.azurehealthcareapis.com` pour l’API Azure pour FHIR |
| Access Token URL (URL du jeton d’accès)      | `https://login.microsoftonline.com/{TENANT ID}/oauth2/token`                                                      |                            |
| Client ID (ID client)             | `XXXXXXXX-XXX-XXXX-XXXX-XXXXXXXXXXXX`                                                                            | ID de l'application             |
| Client Secret (Clé secrète client)         | `XXXXXXXX`                                                                                                        | Clé client secrète          |
| Scope (Étendue) | `<Leave Blank>` |
| State (État)                 | `1234`                                                                                                            |                            |
| Client Authentication (Authentification du client) | Envoyer les informations d’identification du client dans le corps                                                                                 |                 

Cliquez sur « Request Token » (Demander un jeton). Vous êtes alors guidé pendant le déroulement de l’authentification Azure Active Directory et un jeton est retourné à Postman. Si vous rencontrez des problèmes, ouvrez la console Postman (à partir de l’élément de menu « View->Show Postman Console »).

Au bas de l’écran du jeton retourné, cliquez sur « Use Token » (Utiliser le jeton) :

![Utiliser le jeton](media/tutorial-postman/postman-use-token.png)

Le jeton figure maintenant dans le champ « Access Token » (Jeton d’accès) et vous pouvez sélectionner des jetons à partir de « Available Tokens » (Jetons disponibles). Si vous cliquez à nouveau sur « Send » (Envoyer) pour répéter la recherche de ressources `Patient`, vous obtenez l’état `200 OK` :

![200 OK](media/tutorial-postman/postman-200-OK.png)

Cela signifie qu’il n’y a aucun patient dans la base de données et que la recherche est vide.

Si vous inspectez le jeton d’accès avec un outil tel que [https://jwt.ms](https://jwt.ms), vous obtenez un contenu de ce type :

```jsonc
{
  "aud": "https://MYACCOUNT.azurehealthcareapis.com",
  "iss": "https://sts.windows.net/{TENANT-ID}/",
  "iat": 1545343803,
  "nbf": 1545343803,
  "exp": 1545347703,
  "acr": "1",
  "aio": "AUQAu/8JXXXXXXXXXdQxcxn1eis459j70Kf9DwcUjlKY3I2G/9aOnSbw==",
  "amr": [
    "pwd"
  ],
  "appid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "oid": "xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx",
  "appidacr": "1",

  ...// Truncated
}
```

Si vous cherchez à résoudre un problème, nous vous recommandons de vérifier en premier lieu que vous disposez de l’audience appropriée (revendication `aud`). Si votre jeton provient de l’émetteur approprié (revendication `iss`), que sont audience est correcte (revendication `aud`), mais que vous ne parvenez toujours pas à accéder à l’API FHIR, il est probable que l’utilisateur ou le principal de service (revendication `oid`) n’a pas accès au plan de données FHIR. Nous vous recommandons d’[utiliser le contrôle d’accès en fonction du rôle Azure (RBAC Azure)](configure-azure-rbac.md) pour attribuer des rôles de plan de données aux utilisateurs. Si vous utilisez un locataire Azure Active Directory externe secondaire pour votre plan de données, vous devez [configurer des attributions de rôles RBAC locales](configure-local-rbac.md).

Il est également possible d’[obtenir un jeton pour l’API Azure pour FHIR à partir d’Azure CLI](get-healthcare-apis-access-token-cli.md). Si vous utilisez un jeton obtenu avec Azure CLI, vous devez utiliser le type d’autorisation « Bearer Token » (Jeton du porteur) et coller le jeton directement.

## <a name="inserting-a-patient"></a>Insertion d’un patient

Maintenant que vous disposez d’un jeton d’accès valide, vous pouvez insérer un nouveau patient. Passez à la méthode « POST » et ajoutez le document JSON suivant dans le corps de la demande :

[!code-json[](samples/sample-patient.json)]

Cliquez sur « Send » (Envoyer) ; vous pouvez alors observer que le patient a bien été créé :

![Capture d’écran montrant que le patient est correctement créé.](media/tutorial-postman/postman-patient-created.png)

Si vous répétez cette recherche de patient, vous obtenez maintenant l’enregistrement du patient :

![Patient créé](media/tutorial-postman/postman-patient-found.png)

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez accédé à une API FHIR à l’aide de Postman. Pour en savoir plus sur les fonctionnalités d’API prises en charges, consultez la section y afférente.
 
>[!div class="nextstepaction"]
>[Fonctionnalités prises en charge](fhir-features-supported.md)
