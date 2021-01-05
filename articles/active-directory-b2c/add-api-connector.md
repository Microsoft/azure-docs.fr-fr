---
title: Ajouter des connecteurs d’API à des flux d’utilisateurs (version préliminaire)
description: Configurer un connecteur d’API à utiliser dans un workflow utilisateur.
services: active-directory-b2c
ms.service: active-directory
ms.subservice: B2C
ms.topic: how-to
ms.date: 10/15/2020
ms.author: mimart
author: msmimart
manager: celestedg
ms.custom: it-pro
ms.openlocfilehash: 9117474c3cbf5087a5b63512fcc17c4771bf7aa6
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96343873"
---
# <a name="add-an-api-connector-to-a-sign-up-user-flow-preview"></a>Ajouter un connecteur d’API à un flux d’utilisateur d’inscription (version préliminaire)

> [!IMPORTANT]
> Les connecteurs d’API pour l’inscription sont une fonctionnalité en préversion publique d’Azure AD B2C. Pour plus d’informations sur les préversions, consultez [Conditions d’utilisation supplémentaires pour les préversions de Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

Pour utiliser un [connecteur d’API](api-connectors-overview.md), vous devez d’abord créer le connecteur d’API, puis l’activer dans un workflow utilisateur.

## <a name="create-an-api-connector"></a>Créer un connecteur d'API

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Sous **Services Azure**, sélectionnez **Azure AD B2C**.
4. Sélectionnez **Connecteurs d’API (version préliminaire)** , puis sélectionnez **Nouveau connecteur d’API**.

   ![Ajoutez un nouveau connecteur d'API](./media/add-api-connector/api-connector-new.png)

5. Indiquez un nom d’affichage pour l’appel. Par exemple, **Valider les informations utilisateur**.
6. Indiquez l’**URL du point de terminaison** pour l’appel d’API.
7. Saisissez les informations d’authentification pour l’API.

   - Seule l’authentification de base est actuellement prise en charge. Si vous souhaitez utiliser une API sans authentification de base à des fins de développement, saisissez simplement un **nom d’utilisateur** et un **mot de passe** « factices » que votre API peut ignorer. Pour vous en servir avec une fonction Azure et une clé API, vous pouvez inclure le code en tant que paramètre de requête dans l’**URL du point de terminaison** (par exemple https []()://contoso.azurewebsites.net/api/endpoint <b>?code=0123456789</b>).

   ![Configurer un nouveau connecteur d’API](./media/add-api-connector/api-connector-config.png)
8. Sélectionnez **Enregistrer**.

## <a name="the-request-sent-to-your-api"></a>Demande envoyée à votre API
Un connecteur d’API est matérialisé en tant que requête **HTTP POST**, en envoyant les attributs utilisateur (« revendications ») en tant que paires clé-valeur dans un corps JSON. Les attributs sont sérialisés de la même façon que les propriétés utilisateur [Microsoft Graph](/graph/api/resources/user#properties). 

**Exemple de requête**
```http
POST <API-endpoint>
Content-type: application/json

{
 "email": "johnsmith@fabrikam.onmicrosoft.com",
 "identities": [
     {
     "signInType":"federated",
     "issuer":"facebook.com",
     "issuerAssignedId":"0123456789"
     }
 ],
 "displayName": "John Smith",
 "givenName":"John",
 "surname":"Smith",
 "jobTitle":"Supplier",
 "streetAddress":"1000 Microsoft Way",
 "city":"Seattle",
 "postalCode": "12345",
 "state":"Washington",
 "country":"United States",
 "extension_<extensions-app-id>_CustomAttribute1": "custom attribute value",
 "extension_<extensions-app-id>_CustomAttribute2": "custom attribute value",
 "ui_locales":"en-US"
}
```

Seules les propriétés utilisateur et les attributs personnalisés répertoriés dans l’expérience **Azure AD B2C** > **Attributs utilisateur** peuvent être envoyés dans la demande.

Les attributs personnalisés existent au format **extension_\<extensions-app-id>_CustomAttribute** dans le répertoire. Votre API doit s’attendre à recevoir des revendications dans ce même format sérialisé. Pour plus d’informations sur les attributs personnalisés, consultez [Définir des attributs personnalisés dans Azure Active Directory B2C](user-flow-custom-attributes.md).

De plus, la revendication **Paramètres régionaux de l’interface utilisateur (« ui_locales »)** est envoyée par défaut dans toutes les demandes. Elle fournit les paramètres régionaux d’un utilisateur tels qu’ils sont configurés sur son appareil et ils peuvent être utilisés par l’API pour renvoyer des réponses internationalisées.

> [!IMPORTANT]
> Si une revendication n’a pas de valeur au moment où le point de terminaison de l’API est appelé, la revendication n’est pas envoyée à l’API. Votre API doit être conçue pour vérifier et gérer explicitement le cas où une revendication ne figure pas dans la requête.

> [!TIP] 
> Les revendications [**identités (« identities »)**](/graph/api/resources/objectidentity) et **Adresse e-mail (« email »)** peuvent être utilisées par votre API pour identifier un utilisateur avant qu’il n’ait un compte dans votre locataire. 

## <a name="enable-the-api-connector-in-a-user-flow"></a>Activer le connecteur d’API dans un workflow utilisateur

Procédez comme suit pour ajouter un connecteur d’API à un workflow d’utilisateur d’inscription.

1. Connectez-vous au [portail Azure](https://portal.azure.com/).
2. Sous **Services Azure**, sélectionnez **Azure AD B2C**.
4. Sélectionnez **Flux d’utilisateurs**, puis sélectionnez le flux utilisateur auquel vous souhaitez ajouter le connecteur d’API.
5. Sélectionnez **Connecteurs d’API**, puis sélectionnez les points de terminaison d’API que vous souhaitez appeler aux étapes suivantes dans le workflow d’utilisateur :

   - **Après la connexion avec un fournisseur d’identité**
   - **Avant de créer l’utilisateur**

   ![Ajouter des API au flux de l’utilisateur](./media/add-api-connector/api-connectors-user-flow-select.png)

6. Sélectionnez **Enregistrer**.

## <a name="after-signing-in-with-an-identity-provider"></a>après la connexion avec un fournisseur d’identité

Un connecteur d’API à cette étape du processus d’inscription est appelé immédiatement après que l’utilisateur s’est authentifié auprès d’un fournisseur d’identité (comme Google, Facebook et Azure AD). Cette étape précède la **_page de collection d’attributs_* _, qui est le formulaire présenté à l’utilisateur pour collecter des attributs utilisateur. Cette étape n’est pas appelée si un utilisateur s’inscrit auprès d’un compte local.

### <a name="example-request-sent-to-the-api-at-this-step"></a>Exemple de demande envoyée à l’API à cette étape
```http
POST <API-endpoint>
Content-type: application/json

{
 "email": "johnsmith@fabrikam.onmicrosoft.com",
 "identities": [ 
     {
     "signInType":"federated",
     "issuer":"facebook.com",
     "issuerAssignedId":"0123456789"
     }
 ],
 "displayName": "John Smith",
 "givenName":"John",
 "lastName":"Smith",
 "ui_locales":"en-US"
}
```

Les revendications exactes envoyées à l’API dépendent des informations fournies par le fournisseur d’identité. « email » est toujours envoyé.

### <a name="expected-response-types-from-the-web-api-at-this-step"></a>Types de réponses attendus de l’API Web à cette étape

Lorsque l’API Web reçoit une requête HTTP de Azure AD pendant un workflow utilisateur, elle peut renvoyer les réponses suivantes :

- Réponse de continuation
- Réponse de blocage

#### <a name="continuation-response"></a>Réponse de continuation

Une réponse de continuation indique que le flux de l’utilisateur doit passer à l’étape suivante : la page de collecte d’attribut.

Dans une réponse de continuation, l’API peut renvoyer des revendications. Si une revendication est retournée par l’API, la revendication effectue les opérations suivantes :

- Renseigne le champ d’entrée dans la page de collections d’attributs.

Consultez un exemple de [réponse de continuation](#example-of-a-continuation-response).

#### <a name="blocking-response"></a>Réponse de blocage

Une réponse de blocage quitte le workflow de l’utilisateur. Il peut être intentionnellement émis par l’API pour arrêter la continuation du workflow de l’utilisateur en affichant une page de blocage à l’utilisateur. La page de blocage affiche les `userMessage` fournies par l’API.

Consultez un exemple de [réponse de blocage](#example-of-a-blocking-response).

## <a name="before-creating-the-user"></a>avant la création de l’utilisateur

Un connecteur d’API à cette étape du processus d’inscription est appelé après la page de collection d’attributs, si elle est inclue. Cette étape est toujours appelée avant qu’un compte d’utilisateur ne soit créé.

<!-- The following are examples of scenarios you might enable at this point during sign-up: -->
<!-- 
- Validate user input data and ask a user to resubmit data.
- Block a user sign-up based on data entered by the user.
- Perform identity verification.
- Query external systems for existing data about the user and overwrite the user-provided value. -->

### <a name="example-request-sent-to-the-api-at-this-step"></a>Exemple de demande envoyée à l’API à cette étape

```http
POST <API-endpoint>
Content-type: application/json

{
 "email": "johnsmith@fabrikam.onmicrosoft.com",
 "identities": [
     {
     "signInType":"federated",
     "issuer":"facebook.com",
     "issuerAssignedId":"0123456789"
     }
 ],
 "displayName": "John Smith",
 "givenName":"John",
 "surname":"Smith",
 "jobTitle":"Supplier",
 "streetAddress":"1000 Microsoft Way",
 "city":"Seattle",
 "postalCode": "12345",
 "state":"Washington",
 "country":"United States",
 "extension_<extensions-app-id>_CustomAttribute1": "custom attribute value",
 "extension_<extensions-app-id>_CustomAttribute2": "custom attribute value",
 "ui_locales":"en-US"
}
```
Les revendications exactes envoyées à l’API dépendent des informations collectées par l’utilisateur ou fournies par le fournisseur d’identité.

### <a name="expected-response-types-from-the-web-api-at-this-step"></a>Types de réponses attendus de l’API Web à cette étape

Lorsque l’API Web reçoit une requête HTTP de Azure AD pendant un workflow utilisateur, elle peut renvoyer les réponses suivantes :

- Réponse de continuation
- Réponse de blocage
- Réponse de validation

#### <a name="continuation-response"></a>Réponse de continuation

Une réponse de continuation indique que le flux de l’utilisateur doit passer à l’étape suivante : créer l’utilisateur dans le répertoire.

Dans une réponse de continuation, l’API peut renvoyer des revendications. Si une revendication est retournée par l’API, la revendication effectue les opérations suivantes :

- Remplace toute valeur qui a déjà été attribuée à la revendication à partir de la page de collection d’attributs.

Consultez un exemple de [réponse de continuation](#example-of-a-continuation-response).

#### <a name="blocking-response"></a>Réponse de blocage
Une réponse de blocage quitte le workflow de l’utilisateur. Il peut être intentionnellement émis par l’API pour arrêter la continuation du workflow de l’utilisateur en affichant une page de blocage à l’utilisateur. La page de blocage affiche les `userMessage` fournies par l’API.

Consultez un exemple de [réponse de blocage](#example-of-a-blocking-response).

### <a name="validation-error-response"></a>Validation-réponse d’erreur
 Lorsque l’API répond avec une réponse d’erreur de validation, le workflow de l’utilisateur reste sur la page de collection d’attributs et un `userMessage` est présenté à l’utilisateur. L’utilisateur peut ensuite modifier et renvoyer le formulaire. Ce type de réponse peut être utilisé pour la validation d’entrée.

Consultez un exemple de [réponse d’erreur de validation](#example-of-a-validation-error-response).

## <a name="example-responses"></a>Exemples de réponses

### <a name="example-of-a-continuation-response"></a>Exemple de réponse de continuation

```http
HTTP/1.1 200 OK
Content-type: application/json

{
    "version": "1.0.0",
    "action": "Continue",
    "postalCode": "12349", // return claim
    "extension_<extensions-app-id>_CustomAttribute": "value" // return claim
}
```

| Paramètre                                          | Type              | Obligatoire | Description                                                                                                                                                                                                                                                                            |
| -------------------------------------------------- | ----------------- | -------- | -------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| version                                            | String            | Oui      | Version de l’API.                                                                                                                                                                                                                                                                |
| action                                             | String            | Oui      | La valeur doit être `Continue`.                                                                                                                                                                                                                                                              |
| \<builtInUserAttribute>                            | \<attribute-type> | Non       | Les valeurs retournées peuvent remplacer des valeurs collectées à partir d’un utilisateur. Elles peuvent également être retournées dans le jeton si elles sont sélectionnées en tant que _*Revendication d’application**.                                              |
| \<extension\_{extensions-app-id}\_CustomAttribute> | \<attribute-type> | Non       | La revendication n’a pas besoin de contenir `_<extensions-app-id>_`. Les valeurs retournées peuvent remplacer des valeurs collectées à partir d’un utilisateur. Elles peuvent également être retournées dans le jeton si elles sont sélectionnées en tant que **Revendication d’application**.  |

### <a name="example-of-a-blocking-response"></a>Exemple de réponse de blocage

```http
HTTP/1.1 200 OK
Content-type: application/json

{
    "version": "1.0.0",
    "action": "ShowBlockPage",
    "userMessage": "There was a problem with your request. You are not able to sign up at this time.",
}

```

| Paramètre   | Type   | Obligatoire | Description                                                                |
| ----------- | ------ | -------- | -------------------------------------------------------------------------- |
| version     | String | Oui      | Version de l’API.                                                    |
| action      | String | Oui      | La valeur doit être `ShowBlockPage`                                              |
| userMessage | String | Oui      | Message à afficher à l’utilisateur.                                            |

**Expérience de l’utilisateur final avec une réponse de blocage**

![Exemple de page de blocage](./media/add-api-connector/blocking-page-response.png)

### <a name="example-of-a-validation-error-response"></a>Exemple de réponse d’erreur de validation



```http
HTTP/1.1 400 Bad Request
Content-type: application/json

{
    "version": "1.0.0",
    "status": 400,
    "action": "ValidationError",
    "userMessage": "Please enter a valid Postal Code."
}
```

| Paramètre   | Type    | Obligatoire | Description                                                                |
| ----------- | ------- | -------- | -------------------------------------------------------------------------- |
| version     | String  | Oui      | Version de l’API.                                                    |
| action      | String  | Oui      | La valeur doit être `ValidationError`.                                           |
| status      | Integer | Oui      | La valeur doit être `400` pour une réponse ValidationError.                        |
| userMessage | String  | Oui      | Message à afficher à l’utilisateur.                                            |

> [!NOTE]
> Le code d’état HTTP doit être « 400 » en plus de la valeur « Status » dans le corps de la réponse.

**Expérience de l’utilisateur final avec une réponse d’erreur de validation**

![Exemple de page de validation](./media/add-api-connector/validation-error-postal-code.png)


## <a name="best-practices-and-how-to-troubleshoot"></a>Meilleures pratiques et résolution des problèmes

### <a name="using-serverless-cloud-functions"></a>Utilisation des fonctions cloud serverless
Les fonctions serverless, comme les déclencheurs HTTP dans Azure Functions, fournissent une méthode simple pour créer des points de terminaison d’API à utiliser avec le connecteur d’API. Vous pouvez utiliser la fonction cloud serverless pour, [par exemple](code-samples.md#api-connectors), effectuer une logique de validation et limiter les inscriptions à des domaines de courrier spécifiques. La fonction cloud serverless peut également appeler et invoquer d’autres API web, magasins d’utilisateurs et autres services cloud dans le cade de scénarios plus complexes.

### <a name="best-practices"></a>Meilleures pratiques
Assurez-vous que :
* Votre API suit les contrats de demande et de réponse d’API comme indiqué ci-dessus. 
* L’**URL du point de terminaison** du connecteur d’API pointe vers le point de terminaison d’API approprié.
* Votre API recherche explicitement les valeurs null des revendications reçues.
* Votre API répond aussi rapidement que possible pour garantir une expérience utilisateur fluide.
    * Si vous utilisez une fonction serverless ou un service web scalable, utilisez un plan d’hébergement qui conserve l’API dans un état « de veille » ou « dynamique ». en production. Pour Azure Functions, il est recommandé d’utiliser le [plan Premium](../azure-functions/functions-scale.md)


### <a name="use-logging"></a>Utiliser la journalisation
En général, il est judicieux d’utiliser les outils de journalisation activés par votre service API Web, comme [Application Insights](../azure-functions/functions-monitoring.md), pour surveiller votre API en cas de codes d’erreur inattendus, d’exceptions et de performances médiocres.
* Analysez les codes d’état HTTP autres que HTTP 200 ou 400.
* Un code d’état HTTP 401 ou 403 indique généralement un problème avec votre authentification. Vérifiez la couche d’authentification de votre API et la configuration correspondante dans le connecteur d’API.
* Si nécessaire, utilisez des niveaux de journalisation plus agressifs (par exemple, « trace » ou « debug ») lors du développement.
* Surveillez votre API en cas de temps de réponse longs.

## <a name="next-steps"></a>Étapes suivantes
<!-- - Learn how to [add a custom approval workflow to sign-up](add-approvals.md) -->
- Bien commencer avec nos [exemples](code-samples.md#api-connectors).
