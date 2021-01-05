---
title: 'Démarrage rapide : Bibliothèque de client Java Recherche d’entités Bing'
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 03/06/2020
ms.custom: devx-track-java
ms.author: aahi
ms.openlocfilehash: f69b9b989a93949f9a0441676c81af7480fb968f
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "87375263"
---
Utilisez ce guide de démarrage rapide pour commencer à rechercher des entités avec la bibliothèque de client Recherche d’entités Bing pour Java. Si l’outil Recherche d’entités Bing a une API REST compatible avec la plupart des langages de programmation, la bibliothèque de client offre quant à elle un moyen facile d’intégrer le service à vos applications. Le code source de cet exemple est disponible sur [GitHub](https://github.com/Azure-Samples/cognitive-services-java-sdk-samples/tree/master/Search/BingEntitySearch).

## <a name="prerequisites"></a>Prérequis

* Le [JDK (Java Development Kit)](https://www.oracle.com/technetwork/java/javase/downloads/)

* La bibliothèque de client Recherche d’entités Bing pour Java

Installez les dépendances de bibliothèque de client Recherche d’entités Bing en utilisant Maven, Gradle ou un autre système de gestion des dépendances. Le fichier POM Maven nécessite la déclaration suivante :

```xml
<dependency>
  <groupId>com.microsoft.azure.cognitiveservices</groupId>
  <artifactId>azure-cognitiveservices-entitysearch</artifactId>
  <version>1.0.2</version>
</dependency>
```

[!INCLUDE [cognitive-services-bing-news-search-signup-requirements](~/includes/cognitive-services-bing-entity-search-signup-requirements.md)]


## <a name="create-and-initialize-a-project"></a>Créer et initialiser un projet

1. Créez un projet Java dans votre éditeur ou IDE favori, puis importez les bibliothèques suivantes.

    ```java
    import com.microsoft.azure.cognitiveservices.entitysearch.*;
    import com.microsoft.azure.cognitiveservices.entitysearch.implementation.EntitySearchAPIImpl;
    import com.microsoft.azure.cognitiveservices.entitysearch.implementation.SearchResponseInner;
    import com.microsoft.rest.credentials.ServiceClientCredentials;
    import okhttp3.Interceptor;
    import okhttp3.OkHttpClient;
    import okhttp3.Request;
    import okhttp3.Response;
    
    import java.io.IOException;
    import java.util.ArrayList;
    import java.util.List;
    ```

2. Créer une variable pour votre clé d’abonnement

    ```java
    String subscriptionKey = "your-key-here"
    ```

## <a name="create-a-search-client"></a>Créer un client de recherche

1. Implémentez le client `dominantEntityLookup`, qui nécessite votre point de terminaison d’API et une instance de la classe `ServiceClientCredentials`. Vous pouvez utiliser le point de terminaison global ci-dessous, ou le point de terminaison de [sous-domaine personnalisé](../../../../cognitive-services/cognitive-services-custom-subdomains.md) affiché dans le portail Azure pour votre ressource.

    ```java
    public static EntitySearchAPIImpl getClient(final String subscriptionKey) {
        return new EntitySearchAPIImpl("https://api.cognitive.microsoft.com/bing/v7.0/",
                new ServiceClientCredentials() {
                //...
                }
    )};
    ```

    Pour implémenter `ServiceClientCredentials`, effectuez les étapes suivantes :

   1. Remplacez la fonction `applyCredentialsFilter()` avec un objet `OkHttpClient.Builder` en tant que paramètre. 
        
       ```java
       //...
       new ServiceClientCredentials() {
               @Override
               public void applyCredentialsFilter(OkHttpClient.Builder builder) {
               //...
               }
       //...
       ```
    
   2. Au sein de `applyCredentialsFilter()`, appelez `builder.addNetworkInterceptor()`. Créez un objet `Interceptor` et remplacez sa méthode `intercept()` pour prendre un objet intercepteur `Chain`.

       ```java
       //...
       builder.addNetworkInterceptor(
           new Interceptor() {
               @Override
               public Response intercept(Chain chain) throws IOException {
               //...    
               }
           });
       ///...
       ```

   3. Au sein de la fonction `intercept`, créez des variables pour votre requête. Utilisez `Request.Builder()` pour générer votre requête. Ajoutez votre clé d’abonnement à l’en-tête `Ocp-Apim-Subscription-Key` et retournez `chain.proceed()` sur l’objet de requête.
            
       ```java
       //...
       public Response intercept(Chain chain) throws IOException {
           Request request = null;
           Request original = chain.request();
           Request.Builder requestBuilder = original.newBuilder()
                   .addHeader("Ocp-Apim-Subscription-Key", subscriptionKey);
           request = requestBuilder.build();
           return chain.proceed(request);
       }
       //...
       ```
## <a name="send-a-request-and-receive-a-response"></a>Envoyer une requête et recevoir une réponse

1. Créez une instance du client de recherche avec votre clé d’abonnement. Utilisez `client.entities().search()` pour envoyer une demande pour la requête de recherche `satya nadella` et obtenir une réponse. 
    
    ```java
    EntitySearchAPIImpl client = getClient(subscriptionKey);
    SearchResponseInner entityData = client.entities().search(
            "satya nadella", null, null, null, null, null, null, "en-us", null, null, SafeSearch.STRICT, null);
    ```

1. Si des entités sont retournées, convertissez-les en liste. Itérez en leur sein et affichez l’entité dominante.

    ```java
    if (entityData.entities().value().size() > 0){
        // Find the entity that represents the dominant entity
        List<Thing> entries = entityData.entities().value();
        Thing dominateEntry = null;
        for(Thing thing : entries) {
            if(thing.entityPresentationInfo().entityScenario() == EntityScenario.DOMINANT_ENTITY) {
                System.out.println("\r\nSearched for \"Satya Nadella\" and found a dominant entity with this description:");
                System.out.println(thing.description());
                break;
            }
        }
    }
    ```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Créer une application web monopage](../../tutorial-bing-entities-search-single-page-app.md)

* [Qu’est-ce que l’API Recherche d’entités Bing ?](../../overview.md)
