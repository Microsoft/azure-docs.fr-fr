---
title: 'Démarrage rapide : Bibliothèque de client Java Recherche de vidéos Bing'
titleSuffix: Azure Cognitive Services
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.topic: include
ms.date: 03/19/2020
ms.custom: devx-track-java
ms.author: aahi
ms.openlocfilehash: 2b3d4993406f150b2983d4d820f7d070b5de1e96
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "87375403"
---
Utilisez ce guide de démarrage rapide pour démarrer une recherche de vidéos avec la bibliothèque de client Recherche de vidéos Bing pour Java. Si l’outil Recherche de vidéos Bing dispose d’une API REST compatible avec la plupart des langages de programmation, la bibliothèque de client offre quant à elle un moyen facile d’intégrer le service à vos applications. Le code source de cet exemple est disponible sur [GitHub](https://github.com/Azure-Samples/cognitive-services-java-sdk-samples/tree/master/Search/BingVideoSearch) avec des fonctionnalités et annotations supplémentaires.

## <a name="prerequisites"></a>Prérequis

* Le [JDK (Java Development Kit)](https://www.oracle.com/technetwork/java/javase/downloads/jdk11-downloads-5066655.html)

* La [bibliothèque Gson](https://github.com/google/gson)

[!INCLUDE [cognitive-services-bing-video-search-signup-requirements](~/includes/cognitive-services-bing-video-search-signup-requirements.md)]

Installez les dépendances de bibliothèque de client Recherche de vidéos Bing en utilisant Maven, Gradle ou un autre système de gestion des dépendances. Le fichier POM Maven nécessite la déclaration suivante :

```xml
  <dependencies>
    <dependency>
      <groupId>com.microsoft.azure.cognitiveservices</groupId>
      <artifactId>azure-cognitiveservices-videosearch</artifactId>
      <version>0.0.1-beta-SNAPSHOT</version>
    </dependency>
  </dependencies> 
```

## <a name="create-and-initialize-a-project"></a>Créer et initialiser un projet


Créez un projet Java dans votre éditeur ou IDE favori, puis importez les bibliothèques suivantes.

```java
    import com.microsoft.azure.cognitiveservices.videosearch.*;
    import com.microsoft.azure.cognitiveservices.videosearch.VideoObject;
    import com.microsoft.rest.credentials.ServiceClientCredentials;
    import okhttp3.Interceptor;
    import okhttp3.OkHttpClient;
    import okhttp3.Request;
    import okhttp3.Response;
    import java.io.IOException;
    import java.util.ArrayList;
    import java.util.List; 
```

## <a name="create-a-search-client"></a>Créer un client de recherche

1. Implémentez le client `VideoSearchAPIImpl`, qui nécessite votre point de terminaison d’API et une instance de la classe `ServiceClientCredentials`.

    ```java
    public static VideoSearchAPIImpl getClient(final String subscriptionKey) {
        return new VideoSearchAPIImpl("https://api.cognitive.microsoft.com/bing/v7.0/",
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

## <a name="send-a-search-request-and-receive-the-response"></a>Envoyer une requête de recherche et recevoir la réponse 

1. Créez une fonction nommée `VideoSearch()` qui prend votre clé d’abonnement comme chaîne. Instanciez le client recherche créé précédemment.
    
    ```java
    public static void VideoSearch(String subscriptionKey){
        VideoSearchAPIImpl client = VideoSDK.getClient(subscriptionKey);
        //...
    }
    ```
2. Au sein de `VideoSearch()`, envoyez une requête de recherche de vidéos à l’aide du client, avec `SwiftKey` comme terme de recherche. Si l’API Recherche de vidéos retourne un résultat, obtenez le premier et affichez son ID, son nom et son URL, ainsi que le nombre total de vidéos retournées. 
    
    ```java
    VideosInner videoResults = client.searchs().list("SwiftKey");

    if (videoResults == null){
        System.out.println("Didn't see any video result data..");
    }
    else{
        if (videoResults.value().size() > 0){
            VideoObject firstVideoResult = videoResults.value().get(0);

            System.out.println(String.format("Video result count: %d", videoResults.value().size()));
            System.out.println(String.format("First video id: %s", firstVideoResult.videoId()));
            System.out.println(String.format("First video name: %s", firstVideoResult.name()));
            System.out.println(String.format("First video url: %s", firstVideoResult.contentUrl()));
        }
        else{
            System.out.println("Couldn't find video results!");
        }
    }
    ```

3. Appelez la méthode search depuis votre méthode main.

    ```java
    public static void main(String[] args) {
        VideoSDK.VideoSearch("YOUR-SUBSCRIPTION-KEY");
    }
    ```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Créer une application web monopage](../../tutorial-bing-video-search-single-page-app.md)

## <a name="see-also"></a>Voir aussi 

* [Qu’est-ce que l’API Recherche de vidéos Bing ?](../../overview.md)
* [Exemples du Kit de développement logiciel (SDK) .NET pour Cognitive Services](https://github.com/Azure-Samples/cognitive-services-dotnet-sdk-samples/tree/master/BingSearchv7)
