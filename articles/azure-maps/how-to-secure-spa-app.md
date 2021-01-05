---
title: Guide pratique pour sécuriser une application monopage avec connexion non interactive
titleSuffix: Azure Maps
description: Comment configurer une application monopage avec un contrôle d’accès en fonction du rôle Azure (RBAC Azure) non interactif et le Kit de développement logiciel (SDK) web Azure Maps.
author: anastasia-ms
ms.author: v-stharr
ms.date: 06/12/2020
ms.topic: how-to
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.custom: devx-track-js
ms.openlocfilehash: c39104912c99b199d38cf489bb61d64e83b89286
ms.sourcegitcommit: 4064234b1b4be79c411ef677569f29ae73e78731
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92895594"
---
# <a name="how-to-secure-a-single-page-application-with-non-interactive-sign-in"></a>Guide pratique pour sécuriser une application monopage avec connexion non interactive

Le guide suivant se rapporte à une application qui utilise Azure Active Directory (Azure AD) pour fournir un jeton d’accès à des applications Azure Maps quand l’utilisateur ne peut pas se connecter à Azure AD. Ce processus nécessite l’hébergement d’un service web qui doit être sécurisé de façon à être accessible uniquement par l’application web monopage. Il existe plusieurs implémentations qui peuvent effectuer l’authentification auprès d’Azure AD. Ce guide s’appuie sur le produit Azure Functions pour acquérir des jetons d’accès.

[!INCLUDE [authentication details](./includes/view-authentication-details.md)]

> [!Tip]
> Azure Maps peut prendre en charge les jetons d’accès à partir des flux interactifs/d’authentification de l’utilisateur. Les flux interactifs permettent de bénéficier d’une étendue de révocation d’accès et de gestion des secrets plus restreinte.

## <a name="create-azure-function"></a>Créer une fonction Azure

Créez une application de service web sécurisée qui est responsable de l’authentification auprès d’Azure AD. 

1. Créez une fonction dans le portail Azure. Pour plus d’informations, consultez [Créer une fonction Azure](../azure-functions/functions-create-first-azure-function.md).

2. Configurez la stratégie CORS sur la fonction Azure pour qu’elle soit accessible par l’application web monopage. Cela permet de sécuriser les clients du navigateur aux origines autorisées de votre application web. Voir [Ajouter des fonctionnalités CORS](../app-service/app-service-web-tutorial-rest-api.md#add-cors-functionality).

3. [Ajoutez une identité affectée par le système](../app-service/overview-managed-identity.md?tabs=dotnet#add-a-system-assigned-identity) sur la fonction Azure afin d’autoriser la création d’un principal de service pour l’authentification auprès d’Azure AD.  

4. Accordez l’accès en fonction du rôle pour l’identité affectée par le système au compte Azure Maps. Pour plus d’informations, consultez [Accorder l’accès en fonction du rôle](#grant-role-based-access).

5. Écrivez du code pour la fonction Azure afin d’obtenir des jetons d’accès Azure Maps à l’aide d’une identité affectée par le système avec l’un des mécanismes pris en charge ou le protocole REST. Voir [Obtenir des jetons pour les ressources Azure](../app-service/overview-managed-identity.md?tabs=dotnet#add-a-system-assigned-identity).

    Exemple de protocole REST :

    ```http
    GET /MSI/token?resource=https://atlas.microsoft.com/&api-version=2019-08-01 HTTP/1.1
    Host: localhost:4141
    ```

    Exemple de réponse :

    ```http
    HTTP/1.1 200 OK
    Content-Type: application/json

    {
        "access_token": "eyJ0eXAi…",
        "expires_on": "1586984735",
        "resource": "https://atlas.microsoft.com/",
        "token_type": "Bearer",
        "client_id": "..."
    }
    ```

6. Configurer la sécurité pour la fonction Azure HttpTrigger

   * [Créer une clé d’accès à une fonction](../azure-functions/functions-bindings-http-webhook-trigger.md?tabs=csharp#authorization-keys)
   * [Sécurisez le point de terminaison HTTP](../azure-functions/functions-bindings-http-webhook-trigger.md?tabs=csharp#secure-an-http-endpoint-in-production) pour la fonction Azure en production.
   
7. Configurez le SDK web Azure Maps de l’application web. 

    ```javascript
    //URL to custom endpoint to fetch Access token
    var url = 'https://<APP_NAME>.azurewebsites.net/api/<FUNCTION_NAME>?code=<API_KEY>';

    var map = new atlas.Map('myMap', {
                center: [-122.33, 47.6],
                zoom: 12,
                language: 'en-US',
                view: "Auto",
            authOptions: {
                authType: "anonymous",
                clientId: "<insert>", // azure map account client id
                getToken: function(resolve, reject, map) {
                    fetch(url).then(function(response) {
                        return response.text();
                    }).then(function(token) {
                        resolve(token);
                    });
                }
            }
        });

        // use the following events to debug, you can remove them at any time.
        map.events.add("tokenacquired", function () {
            console.log("token acquired");
        });
        map.events.add("error", function (err) {
            console.log(JSON.stringify(err.error));
        });
    ```

## <a name="grant-role-based-access"></a>Accorder un accès en fonction du rôle

Vous accordez le *contrôle d’accès en fonction du rôle (Azure RBAC)* en affectant l’identité affectée par le système à une ou plusieurs définitions de rôles Azure. Pour afficher les définitions de rôle Azure disponibles pour Azure Maps, accédez à **Contrôle d’accès (IAM)** . Sélectionnez **Rôles** , puis recherchez les rôles qui commencent par *Azure Maps* .

1. Accédez à votre **compte Azure Maps** . Sélectionnez **Contrôle d’accès (IAM)**  > **Attributions de rôles** .

    > [!div class="mx-imgBorder"]
    > ![Accorder l’accès à l’aide d’Azure RBAC](./media/how-to-manage-authentication/how-to-grant-rbac.png)

2. Sous l’onglet **Attributions de rôle** , sou **Rôle** , sélectionnez une définition de rôle Azure Maps intégrée, comme **Lecteur de données Azure Maps** ou **Contributeur aux données Azure Maps** . Sous **Attribuer l’accès à** , sélectionnez **Application de fonction** . Sélectionnez le principal par nom. Ensuite, sélectionnez **Enregistrer** .

   * Pour plus d’informations, consultez [Ajouter ou supprimer des attributions de rôles](../role-based-access-control/role-assignments-portal.md).

> [!WARNING]
> Les définitions de rôles intégrées Azure Maps fournissent un accès d’autorisation très large à de nombreuses API REST Azure Maps. Pour limiter l’accès aux API à un minimum, consultez cet [article](../role-based-access-control/custom-roles.md) expliquant comment créer une définition de rôle personnalisée et affecter l’identité affectée par le système à la définition de rôle personnalisée. Cela permet d’activer les privilèges minimum nécessaires à l’application pour accéder à Azure Maps.

## <a name="next-steps"></a>Étapes suivantes

Apprenez-en davantage sur le scénario d’application monopage :
> [!div class="nextstepaction"]
> [Application monopage](../active-directory/develop/scenario-spa-overview.md)

Recherchez les métriques d’utilisation de l’API pour votre compte Azure Maps :
> [!div class="nextstepaction"]
> [Afficher les métriques d’utilisation](how-to-view-api-usage.md)

Explorez d’autres exemples qui montrent comment intégrer Azure AD à Azure Maps :
> [!div class="nextstepaction"]
> [Exemples Azure Maps](https://github.com/Azure-Samples/Azure-Maps-AzureAD-Samples/tree/master/src/ClientGrant)