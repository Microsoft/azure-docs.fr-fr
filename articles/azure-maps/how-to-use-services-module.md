---
title: Utiliser le module des services Azure Maps | Microsoft Azure Maps
description: En savoir plus sur le module de service Azure Maps. Découvrez comment charger et utiliser cette bibliothèque d’assistance pour accéder aux services REST Azure Maps dans des applications web ou Node.js.
author: rbrundritt
ms.author: richbrun
ms.date: 03/25/2019
ms.topic: how-to
ms.service: azure-maps
services: azure-maps
manager: cpendleton
ms.custom: devx-track-js
ms.openlocfilehash: 2e07b614e87ed5dad94cf9bc5994e78071187839
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96008597"
---
# <a name="use-the-azure-maps-services-module"></a>Utiliser le module des services Azure Maps

Le Kit de développement logiciel (SDK) web Azure Maps fournit un *module des services*. Ce module est une bibliothèque d’assistance facilitant l’utilisation des services REST Azure Maps dans des applications web ou Node.js en utilisant JavaScript ou TypeScript.

## <a name="use-the-services-module-in-a-webpage"></a>Utiliser le module des services dans une page web

1. Créez un fichier HTML.
1. Chargez le module des services Azure Maps. Vous pouvez le faire de deux façons :
    - Utiliser la version d’Azure Content Delivery Network du module des services Azure Maps hébergée globalement. Ajoutez une référence de script à l’élément `<head>` du fichier :

        ```html
        <script src="https://atlas.microsoft.com/sdk/javascript/service/2/atlas-service.min.js"></script>
        ```

    - Vous pouvez également charger le module de services pour le code source du kit SDK web d’Azure Maps localement à l’aide du package npm [azure-maps-control](https://www.npmjs.com/package/azure-maps-rest), puis l’héberger avec votre application. Ce package inclut aussi des définitions de TypeScript. Utilisez la commande suivante :
    
        > **npm install azure-maps-rest**
    
        Ensuite, ajoutez une référence de script à l’élément `<head>` du fichier :

         ```html
        <script src="node_modules/azure-maps-rest/dist/atlas-service.min.js"></script>
         ```

1. Créez un pipeline d’authentification. Pour pouvoir initialiser un point de terminaison de client d’URL du service, il est nécessaire que le pipeline soit créé au préalable. Pour authentifier un client du service Recherche Azure Maps, utilisez votre propre clé de compte Azure Maps ou vos informations d’identification Azure Active Directory (Azure AD). Dans cet exemple, le client d’URL du service Recherche est créé. 

    Si vous utilisez une clé d’abonnement pour l’authentification :

    ```javascript
    // Get an Azure Maps key at https://azure.com/maps.
    var subscriptionKey = '<Your Azure Maps Key>';

    // Use SubscriptionKeyCredential with a subscription key.
    var subscriptionKeyCredential = new atlas.service.SubscriptionKeyCredential(subscriptionKey);

    // Use subscriptionKeyCredential to create a pipeline.
    var pipeline = atlas.service.MapsURL.newPipeline(subscriptionKeyCredential, {
      retryOptions: { maxTries: 4 } // Retry options
    });

    // Create an instance of the SearchURL client.
    var searchURL = new atlas.service.SearchURL(pipeline);
    ```

    Si vous utilisez Azure AD pour l’authentification :

    ```javascript
    // Enter your Azure AD client ID.
    var clientId = "<Your Azure Active Directory Client Id>";

    // Use TokenCredential with OAuth token (Azure AD or Anonymous).
    var aadToken = await getAadToken();
    var tokenCredential = new atlas.service.TokenCredential(clientId, aadToken);

    // Create a repeating time-out that will renew the Azure AD token.
    // This time-out must be cleared when the TokenCredential object is no longer needed.
    // If the time-out is not cleared, the memory used by the TokenCredential will never be reclaimed.
    var renewToken = async () => {
      try {
        console.log("Renewing token");
        var token = await getAadToken();
        tokenCredential.token = token;
        tokenRenewalTimer = setTimeout(renewToken, getExpiration(token));
      } catch (error) {
        console.log("Caught error when renewing token");
        clearTimeout(tokenRenewalTimer);
        throw error;
      }
    }
    tokenRenewalTimer = setTimeout(renewToken, getExpiration(aadToken));

    // Use tokenCredential to create a pipeline.
    var pipeline = atlas.service.MapsURL.newPipeline(tokenCredential, {
      retryOptions: { maxTries: 4 } // Retry options
    });

    // Create an instance of the SearchURL client.
    var searchURL = new atlas.service.SearchURL(pipeline);

    function getAadToken() {
      // Use the signed-in auth context to get a token.
      return new Promise((resolve, reject) => {
        // The resource should always be https://atlas.microsoft.com/.
        const resource = "https://atlas.microsoft.com/";
        authContext.acquireToken(resource, (error, token) => {
          if (error) {
            reject(error);
          } else {
            resolve(token);
          }
        });
      })
    }

    function getExpiration(jwtToken) {
      // Decode the JSON Web Token (JWT) to get the expiration time stamp.
      const json = atob(jwtToken.split(".")[1]);
      const decode = JSON.parse(json);

      // Return the milliseconds remaining until the token must be renewed.
      // Reduce the time until renewal by 5 minutes to avoid using an expired token.
      // The exp property is the time stamp of the expiration, in seconds.
      const renewSkew = 300000;
      return (1000 * decode.exp) - Date.now() - renewSkew;
    }
    ```

    Pour plus d’informations, voir [Authentification avec Azure Maps](azure-maps-authentication.md).

1. Le code suivant utilise le nouveau client d’URL du service Recherche Azure Maps pour associer un géocode à une adresse : « 1 Microsoft Way, Redmond, WA ». Le code utilise la fonction `searchAddress` et affiche les résultats sous la forme d’une table dans le corps de la page.

    ```javascript
    // Search for "1 microsoft way, redmond, wa".
    searchURL.searchAddress(atlas.service.Aborter.timeout(10000), '1 microsoft way, redmond, wa')
      .then(response => {
        var html = [];

        // Display the total results.
        html.push('Total results: ', response.summary.numResults, '<br/><br/>');

        // Create a table of the results.
        html.push('<table><tr><td></td><td>Result</td><td>Latitude</td><td>Longitude</td></tr>');

        for(var i=0;i<response.results.length;i++){
          html.push('<tr><td>', (i+1), '.</td><td>', 
            response.results[i].address.freeformAddress, 
            '</td><td>', 
            response.results[i].position.lat,
            '</td><td>', 
            response.results[i].position.lon,
            '</td></tr>');
        }

        html.push('</table>');

        // Add the resulting HTML to the body of the page.
        document.body.innerHTML = html.join('');
    });
    ```

    Voici l’exemple complet de code en cours d’exécution :

<br/>

<iframe height="500" style="width: 100%;" scrolling="no" title="Utilisation du module des services" src="//codepen.io/azuremaps/embed/zbXGMR/?height=500&theme-id=0&default-tab=js,result&editable=true" frameborder='no' loading="lazy" allowtransparency="true" allowfullscreen="true">
Voir la page <a href='https://codepen.io/azuremaps/pen/zbXGMR/'>Utilisation du module des services</a> par Azure Maps (<a href='https://codepen.io/azuremaps'>@azuremaps</a>) sur <a href='https://codepen.io'>CodePen</a>.
</iframe>

<br/>

## <a name="azure-government-cloud-support"></a>Prise en charge du cloud Azure Government

Le SDK web d’Azure Maps prend en charge le cloud Azure Government. Toutes les URL JavaScript et CSS utilisées pour accéder au SDK web d’Azure Maps restent les mêmes. Toutefois, les tâches suivantes doivent être effectuées pour se connecter à la version du cloud Azure Government de la plateforme Azure Maps.

Quand vous utilisez le contrôle de carte interactif, ajoutez la ligne de code suivante avant de créer une instance de la classe `Map`. 

```javascript
atlas.setDomain('atlas.azure.us');
```

Veillez à utiliser les détails d’une authentification Azure Maps à partir de la plateforme du cloud Azure Government lors de l’authentification de la carte et des services.

Quand vous utilisez le module de services, le domaine des services doit être défini lors de la création d’une instance d’un point de terminaison d’URL d’API. Par exemple, le code suivant crée une instance de la classe `SearchURL` et pointe le domaine vers le cloud Azure Government.

```javascript
var searchURL = new atlas.service.SearchURL(pipeline, 'atlas.azure.us');
```

Si vous accédez directement aux services REST d’Azure Maps, définissez le domaine de l’URL sur `atlas.azure.us`. Par exemple, si vous utilisez le service API de recherche, remplacez le domaine URL `https://atlas.microsoft.com/search/` par `https://atlas.azure.us/search/`.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les classes et les méthodes utilisées dans cet article :

> [!div class="nextstepaction"]
> [MapsURL](/javascript/api/azure-maps-rest/atlas.service.mapsurl)

> [!div class="nextstepaction"]
> [SearchURL](/javascript/api/azure-maps-rest/atlas.service.searchurl)

> [!div class="nextstepaction"]
> [RouteURL](/javascript/api/azure-maps-rest/atlas.service.routeurl)

> [!div class="nextstepaction"]
> [SubscriptionKeyCredential](/javascript/api/azure-maps-rest/atlas.service.subscriptionkeycredential)

> [!div class="nextstepaction"]
> [TokenCredential](/javascript/api/azure-maps-rest/atlas.service.tokencredential)

Pour d’autres exemples de code qui utilisent le module des services, voir les articles suivants :

> [!div class="nextstepaction"]
> [Afficher les résultats de recherche sur la carte](./map-search-location.md)

> [!div class="nextstepaction"]
> [Obtenir des informations à partir d’une coordonnée](./map-get-information-from-coordinate.md)

> [!div class="nextstepaction"]
> [Afficher des directions de A à B](./map-route.md)