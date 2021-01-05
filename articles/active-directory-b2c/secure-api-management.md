---
title: Sécuriser une API Gestion des API Azure avec Azure Active Directory B2C
description: Découvrez comment utiliser les jetons d’accès émis par Azure Active Directory B2C pour sécuriser un point de terminaison de l’API Gestion des API Azure.
services: active-directory-b2c
author: msmimart
manager: celestedg
ms.service: active-directory
ms.workload: identity
ms.topic: how-to
ms.date: 07/30/2020
ms.author: mimart
ms.subservice: B2C
ms.openlocfilehash: 60b7bb33dfbf29b7e448887ce992d03009133b2e
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94953486"
---
# <a name="secure-an-azure-api-management-api-with-azure-ad-b2c"></a>Sécuriser une API Gestion des API Azure avec Azure AD B2C

Découvrez comment restreindre l’accès à votre API Gestion des API (APIM) Azure aux clients qui se sont authentifiés avec Azure AD B2C (Azure Active Directory B2C). Suivez les étapes décrites dans cet article pour créer et tester une stratégie de trafic entrant dans APIM qui limite l’accès uniquement aux requêtes qui incluent un jeton d’accès émis par Azure AD B2C valide.

## <a name="prerequisites"></a>Prérequis

Avant de poursuivre les étapes de cet article, vous devez disposer des ressources suivantes :

* [Locataire Azure AD B2C](tutorial-create-tenant.md)
* [Application inscrite](tutorial-register-applications.md) dans votre locataire
* [Flux d’utilisateurs créés](tutorial-create-user-flows.md) dans votre locataire
* [API publiée](../api-management/import-and-publish.md) dans Gestion des API Azure
* [Postman](https://www.getpostman.com/) pour tester l’accès sécurisé (facultatif)

## <a name="get-azure-ad-b2c-application-id"></a>Obtenir l’ID d’application Azure AD B2C

Lorsque vous sécurisez une API dans Gestion des API Azure avec Azure AD B2C, vous avez besoin de plusieurs valeurs pour la [stratégie de trafic entrant](../api-management/api-management-howto-policies.md) que vous créez dans APIM. Tout d’abord, enregistrez l’ID d’une application que vous avez précédemment créée dans votre locataire Azure AD B2C. Si vous utilisez l’application que vous avez créée dans les prérequis, utilisez l’ID d’application pour *webbapp1*.

Pour inscrire une application dans votre locataire Azure AD B2C, vous pouvez utiliser notre nouvelle expérience unifiée **Inscriptions d'applications** ou notre expérience héritée **Applications (héritées)** . [En savoir plus sur la nouvelle expérience](./app-registrations-training-guide.md).

#### <a name="app-registrations"></a>[Inscriptions des applications](#tab/app-reg-ga/)

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sélectionnez le filtre **Annuaire et abonnement** dans le menu supérieur, puis l’annuaire qui contient votre locataire Azure AD B2C.
1. Dans le menu de gauche, sélectionnez **Azure AD B2C**. Ou sélectionnez **Tous les services**, puis recherchez et sélectionnez **Azure AD B2C**.
1. Sélectionnez **Inscriptions d'applications**, puis sélectionnez l'onglet **Applications détenues**.
1. Enregistrez la valeur dans la colonne **ID d’application (cliente)** pour *webapp1* ou une autre application que vous avez créée précédemment.

#### <a name="applications-legacy"></a>[Applications (héritées)](#tab/applications-legacy/)

1. Connectez-vous au [portail Azure](https://portal.azure.com).
1. Sélectionnez le filtre **Annuaire et abonnement** dans le menu supérieur, puis l’annuaire qui contient votre locataire Azure AD B2C.
1. Dans le menu de gauche, sélectionnez **Azure AD B2C**. Ou sélectionnez **Tous les services**, puis recherchez et sélectionnez **Azure AD B2C**.
1. Sous **Gérer**, sélectionnez **Applications (héritées)** .
1. Enregistrez la valeur dans la colonne **ID D’APPLICATION** pour *webapp1* ou une autre application que vous avez créée précédemment.

* * *

## <a name="get-token-issuer-endpoint"></a>Obtenir le point de terminaison de l’émetteur de jeton

Procurez-vous ensuite l’URL de configuration connue pour l’un de vos flux d’utilisateurs Azure AD B2C. Vous avez également besoin de l’URI de point de terminaison de l’émetteur de jeton que vous souhaitez prendre en charge dans Gestion des API Azure.

1. Accédez à votre locataire Azure AD B2C dans le [portail Azure](https://portal.azure.com).
1. Sous **Stratégies**, sélectionnez **Flux utilisateur**.
1. Sélectionnez une stratégie existante, par exemple *B2C_1_signupsignin1*, puis **Exécuter le flux d’utilisateur**.
1. Enregistrez l’URL dans le lien hypertexte affiché sous le titre **Exécuter le flux d’utilisateur** près du haut de la page. Cette URL est le point de terminaison de détection OpenID Connect bien connu pour le flux d’utilisateurs et vous l’utilisez dans la section suivante lorsque vous configurez la stratégie de trafic entrant dans Gestion des API Azure.

    ![Lien hypertexte URI bien connu dans la page Exécuter maintenant du Portail Azure](media/secure-apim-with-b2c-token/portal-01-policy-link.png)

1. Sélectionnez le lien hypertexte pour accéder à la page de configuration OpenID Connect bien connue.
1. Dans la page qui s’ouvre dans votre navigateur, enregistrez la valeur `issuer`, par exemple :

    `https://<tenant-name>.b2clogin.com/xxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx/v2.0/`

    Vous utilisez cette valeur dans la section suivante lorsque vous configurez votre API dans Gestion des API Azure.

Vous devez maintenant avoir deux URL enregistrées pour une utilisation dans la section suivante : l’URL de point de terminaison de configuration OpenID Connect bien connue et l’URI de l’émetteur. Par exemple :

```
https://<tenant-name>.b2clogin.com/<tenant-name>.onmicrosoft.com/B2C_1_signupsignin1/v2.0/.well-known/openid-configuration
https://<tenant-name>.b2clogin.com/99999999-0000-0000-0000-999999999999/v2.0/
```

## <a name="configure-inbound-policy-in-azure-api-management"></a>Configurer la stratégie de trafic entrant dans Gestion des API Azure

Vous êtes maintenant prêt à ajouter la stratégie de trafic entrant dans Gestion des API Azure qui valide les appels d’API. En ajoutant une stratégie de [validation JWT](../api-management/api-management-access-restriction-policies.md#ValidateJWT) qui vérifie l’audience et l’émetteur dans un jeton d’accès, vous pouvez vous assurer que seuls les appels d’API avec un jeton valide sont acceptés.

1. Accédez à votre instance Gestion des API Azure dans le [portail Azure](https://portal.azure.com).
1. Sélectionnez **API**.
1. Sélectionnez l’API que vous souhaitez sécuriser avec Azure AD B2C.
1. Sélectionnez l’onglet **Conception**.
1. Sous **Traitement entrant**, sélectionnez **\</\>** pour ouvrir l'éditeur de code de stratégie.
1. Placez la balise `<validate-jwt>` suivante à l’intérieur de la stratégie `<inbound>`.

    1. Mettez à jour la valeur `url` de l’élément `<openid-config>` avec l’URL de configuration connue de votre stratégie.
    1. Mettez à jour l’élément `<audience>` avec l’ID de l’application que vous avez créée précédemment dans votre locataire B2C (par exemple, *webapp1*).
    1. Mettez à jour l’élément `<issuer>` avec le point de terminaison de l’émetteur de jeton que vous avez enregistré précédemment.

    ```xml
    <policies>
        <inbound>
            <validate-jwt header-name="Authorization" failed-validation-httpcode="401" failed-validation-error-message="Unauthorized. Access token is missing or invalid.">
                <openid-config url="https://<tenant-name>.b2clogin.com/<tenant-name>.onmicrosoft.com/B2C_1_signupsignin1/v2.0/.well-known/openid-configuration" />
                <audiences>
                    <audience>44444444-0000-0000-0000-444444444444</audience>
                </audiences>
                <issuers>
                    <issuer>https://<tenant-name>.b2clogin.com/99999999-0000-0000-0000-999999999999/v2.0/</issuer>
                </issuers>
            </validate-jwt>
            <base />
        </inbound>
        <backend> <base /> </backend>
        <outbound> <base /> </outbound>
        <on-error> <base /> </on-error>
    </policies>
    ```

## <a name="validate-secure-api-access"></a>Valider l’accès sécurisé à l’API

Pour vous assurer que seuls les appelants authentifiés peuvent accéder à votre API, vous pouvez valider votre configuration de Gestion des API Azure en appelant l’API avec [Postman](https://www.getpostman.com/).

Pour appeler l’API, vous avez besoin d’un jeton d’accès émis par Azure AD B2C et d’une clé d’abonnement APIM.

### <a name="get-an-access-token"></a>Obtention d’un jeton d’accès

Vous avez d’abord besoin d’un jeton émis par Azure AD B2C à utiliser dans l’en-tête `Authorization` dans Postman. Vous pouvez en obtenir un à l’aide de la fonctionnalité **Exécuter maintenant** du flux d’utilisateurs d’inscription/de connexion que vous devez avoir créé comme l’un des prérequis.

1. Accédez à votre locataire Azure AD B2C dans le [portail Azure](https://portal.azure.com).
1. Sous **Stratégies**, sélectionnez **Flux utilisateur**.
1. Sélectionnez un flux d’utilisateurs d’inscription/de connexion existant, par exemple *B2C_1_signupsignin1*.
1. Pour **Application**, sélectionnez *webapp1*.
1. Pour **URL de réponse**, choisissez `https://jwt.ms`.
1. Sélectionnez **Exécuter le flux utilisateur**.

    ![Page Exécuter le flux d’utilisateur pour le flux d’utilisateurs d’inscription/de connexion dans le portail Azure](media/secure-apim-with-b2c-token/portal-03-user-flow.png)

1. Terminez le processus de connexion. Vous devez être redirigé vers `https://jwt.ms`.
1. Enregistrez la valeur de jeton encodée affichée dans votre navigateur. Vous utilisez cette valeur de jeton pour l’en-tête d’autorisation dans Postman.

    ![Valeur de jeton encodée affichée sur jwt.ms](media/secure-apim-with-b2c-token/jwt-ms-01-token.png)

### <a name="get-api-subscription-key"></a>Obtenir la clé d’abonnement d’API

Une application cliente (dans ce cas, Postman) qui appelle une API publiée doit inclure une clé d’abonnement de gestion des API valide dans ses requêtes HTTP adressées à l’API. Pour obtenir une clé d’abonnement à inclure dans votre requête HTTP Postman :

1. Accédez à votre instance de service Gestion des API Azure dans le [portail Azure](https://portal.azure.com).
1. Sélectionnez **Abonnements**.
1. Sélectionnez les points de suspension pour **Produit : Illimité**, puis sélectionnez **Afficher/masquer les clés**.
1. Enregistrez la **CLÉ PRIMAIRE** du produit. Vous utilisez cette clé pour l’en-tête `Ocp-Apim-Subscription-Key` dans votre requête HTTP dans Postman.

![Page de clé d’abonnement avec l’option Afficher/masquer les clés sélectionnée dans le portail Azure](media/secure-apim-with-b2c-token/portal-04-api-subscription-key.png)

### <a name="test-a-secure-api-call"></a>Tester un appel d’API sécurisé

Une fois le jeton d’accès et la clé d’abonnement APIM enregistrés, vous êtes maintenant prêt à tester si vous avez correctement configuré l’accès sécurisé à l’API.

1. Créez une requête `GET` dans [Postman](https://www.getpostman.com/). Pour l’URL de requête, spécifiez le point de terminaison de la liste des intervenants de l’API que vous avez publiée comme l’un des prérequis. Par exemple :

    `https://contosoapim.azure-api.net/conference/speakers`

1. Ajoutez ensuite les en-têtes suivants :

    | Clé | Valeur |
    | --- | ----- |
    | `Authorization` | Valeur de jeton encodée que vous avez enregistrée précédemment, avec le préfixe `Bearer ` (inclure l’espace après « Bearer ») |
    | `Ocp-Apim-Subscription-Key` | Clé d’abonnement APIM que vous avez enregistrée précédemment |

    L’URL de requête **GET** et les **en-têtes** doivent ressembler à ce qui suit :

    ![Interface utilisateur Postman qui indique l’URL de requête GET et les en-têtes](media/secure-apim-with-b2c-token/postman-01-headers.png)

1. Sélectionnez le bouton **Envoyer** dans Postman pour exécuter la requête. Si vous avez tout configuré correctement, vous devez obtenir une réponse JSON avec un ensemble d’intervenants à la conférence (illustré ici tronqué) :

    ```json
    {
      "collection": {
        "version": "1.0",
        "href": "https://conferenceapi.azurewebsites.net:443/speakers",
        "links": [],
        "items": [
          {
            "href": "https://conferenceapi.azurewebsites.net/speaker/1",
            "data": [
              {
                "name": "Name",
                "value": "Scott Guthrie"
              }
            ],
            "links": [
              {
                "rel": "http://tavis.net/rels/sessions",
                "href": "https://conferenceapi.azurewebsites.net/speaker/1/sessions"
              }
            ]
          },
    [...]
    ```

### <a name="test-an-insecure-api-call"></a>Tester un appel d’API non sécurisé

Maintenant que vous avez effectué une requête réussie, testez le cas d’échec pour vous assurer que les appels à votre API avec un jeton *non valide* sont rejetés comme prévu. Une façon d’effectuer le test consiste à ajouter ou à modifier quelques caractères dans la valeur du jeton, puis à exécuter la même requête `GET` que précédemment.

1. Ajoutez plusieurs caractères à la valeur de jeton pour simuler un jeton non valide. Par exemple, ajoutez « NON VALIDE » à la valeur de jeton :

    ![Section des en-têtes de l’interface utilisateur Postman montrant les mots NON VALIDE ajoutés au jeton](media/secure-apim-with-b2c-token/postman-02-invalid-token.png)

1. Sélectionnez le bouton **Envoyer** pour exécuter la requête. Avec un jeton non valide, le résultat attendu est un code d’état Non autorisé `401` :

    ```json
    {
        "statusCode": 401,
        "message": "Unauthorized. Access token is missing or invalid."
    }
    ```

Si vous voyez le code d’état `401`, vous avez vérifié que seuls les appelants disposant d’un jeton d’accès valide émis par Azure AD B2C peuvent effectuer des requêtes réussies à votre API Gestion des API Azure.

## <a name="support-multiple-applications-and-issuers"></a>Prendre en charge plusieurs applications et émetteurs

Plusieurs applications interagissent généralement avec une seule API REST. Pour permettre à votre API d’accepter des jetons destinés à plusieurs applications, ajoutez leurs ID d’application à l’élément `<audiences>` dans la stratégie de trafic entrant APIM.

```xml
<!-- Accept tokens intended for these recipient applications -->
<audiences>
    <audience>44444444-0000-0000-0000-444444444444</audience>
    <audience>66666666-0000-0000-0000-666666666666</audience>
</audiences>
```

De même, pour prendre en charge plusieurs émetteurs de jetons, ajoutez leurs URI de point de terminaison à l’élément `<issuers>` dans la stratégie de trafic entrant APIM.

```xml
<!-- Accept tokens from multiple issuers -->
<issuers>
    <issuer>https://<tenant-name>.b2clogin.com/99999999-0000-0000-0000-999999999999/v2.0/</issuer>
    <issuer>https://login.microsoftonline.com/99999999-0000-0000-0000-999999999999/v2.0/</issuer>
</issuers>
```

## <a name="migrate-to-b2clogincom"></a>Migrer vers b2clogin.com

Si vous disposez d’une API APIM qui valide les jetons émis par le point de terminaison `login.microsoftonline.com` hérité, vous devez migrer l’API et les applications qui l’appellent pour utiliser les jetons émis par [b2clogin.com](b2clogin.md).

Vous pouvez suivre ce processus général pour effectuer une migration intermédiaire :

1. Ajoutez la prise en charge dans votre stratégie de trafic entrant APIM pour les jetons émis par b2clogin.com et login.microsoftonline.com.
1. Mettez à jour vos applications une à la fois pour obtenir des jetons à partir du point de terminaison b2clogin.com.
1. Une fois que toutes vos applications obtiennent correctement des jetons à partir de b2clogin.com, supprimez la prise en charge des jetons émis par login.microsoftonline.com à partir de l’API.

L’exemple de stratégie de trafic entrant APIM suivant illustre comment accepter des jetons émis par b2clogin.com et login.microsoftonline.com. En outre, il prend en charge les demandes d’API de deux applications.

```xml
<policies>
    <inbound>
        <validate-jwt header-name="Authorization" failed-validation-httpcode="401" failed-validation-error-message="Unauthorized. Access token is missing or invalid.">
            <openid-config url="https://<tenant-name>.b2clogin.com/<tenant-name>.onmicrosoft.com/B2C_1_signupsignin1/v2.0/.well-known/openid-configuration" />
            <audiences>
                <audience>44444444-0000-0000-0000-444444444444</audience>
                <audience>66666666-0000-0000-0000-666666666666</audience>
            </audiences>
            <issuers>
                <issuer>https://login.microsoftonline.com/99999999-0000-0000-0000-999999999999/v2.0/</issuer>
                <issuer>https://<tenant-name>.b2clogin.com/99999999-0000-0000-0000-999999999999/v2.0/</issuer>
            </issuers>
        </validate-jwt>
        <base />
    </inbound>
    <backend> <base /> </backend>
    <outbound> <base /> </outbound>
    <on-error> <base /> </on-error>
</policies>
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les stratégies dans Gestion des API Azure, consultez l’[index de référence des stratégies APIM](../api-management/api-management-policies.md).

Vous trouverez des informations sur la migration des API web basées sur OWIN et de leurs applications vers b2clogin.com dans [Migrer une API web OWIN vers b2clogin.com](multiple-token-endpoints.md).