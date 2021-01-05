---
title: Rechercher des images GIF à l’aide de l’API Recherche d’images Bing
titleSuffix: Azure Cognitive Services
description: L’API Recherche d’images Bing vous permet également de rechercher les images .gif les plus pertinentes sur tout le web.
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: bing-image-search
ms.topic: conceptual
ms.date: 04/24/2018
ms.author: aahi
ms.custom: seodec2018
ms.openlocfilehash: d1fcac891db240def2e7bbdcb45b45caf7f49a82
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96341901"
---
# <a name="search-for-gif-images"></a>Rechercher des images GIF 

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020**, toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Accord Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

L’API Recherche d’images Bing vous permet également de rechercher les images .gif les plus pertinentes sur tout le web.  Les développeurs peuvent intégrer des fichiers gif attrayants dans divers scénarios de conversation. 

L’URL suivante est une requête pour des images .gif animées.
```
https://api.cognitive.microsoft.com/bing/v7.0/images/search?q=interesting&imageType=AnimatedGif&mkt=en-us
```
Le paramètre [q](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#query) spécifie les termes de recherche.  La requête précédente spécifie également `animatedGif` à l’aide du paramètre de filtre [imageType](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#imagetype).

Pour consulter des exemples de résultats, utilisez l’URL suivante pour effectuer une recherche sur bing.com.
```
https://www.bing.com/images/search?q=interesting&qft=%20filterui%3Aphoto-animatedgif

```
## <a name="query-parameters"></a>Paramètres de requête

Pour plus d’informations sur les options et paramètres de requête, consultez la [référence de l’API Recherche d’images](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#query-parameters). Vous trouverez un exemple sous le titre [Exemple de recherche d’image gif animée à l’aide de Java](#gifExample).

## <a name="tips-and-suggestions"></a>Conseils et suggestions

- Vous pouvez spécifier les paramètres [maxFileSize](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#maxfilesize) et [minFileSize](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#minfilesize). Nous vous recommandons de définir maxFileSize sur 2 000 000 dans la mesure où la majorité des fichiers gif de notre index font moins de 2 Mo.  Cela permet également de contrôler la taille des données si la bande passante est un problème, comme dans les scénarios cellulaires mobiles.
- Pour améliorer les performances perçues, chargez la miniature avant de charger l’URL source.  
- Pour la première exécution ou la page d’accueil, lorsque vous n’avez pas encore de requête utilisateur, essayez d’utiliser nos recherches de fichiers gif tendance à l’aide de l’[API de recherche d’images tendance](trending-images.md).
- Il existe trois paramètres pour [safeSearch](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#safesearch).  L’option `strict` bloque le contenu pour adultes.
- Consultez [mkt](./language-support.md) pour obtenir une liste complète des langues et emplacements pris en charge.
- *AnimatedGifHttps* renvoie uniquement les images gif animées provenant d’une adresse https. Pour des raisons de sécurité, de nombreuses applications nécessitent la connexion à des liens web externes via le protocole https. Par exemple, l’App Store d’Apple requiert une connexion aux services web par HTTPS, qui chiffre les données utilisateur de manière sécurisée lors de leur transit.

<a name="gifExample"></a>

## <a name="example-search-for-animated-gif-using-java"></a>Exemple de recherche d’images gif animées à l’aide de Java

L’URL suivante recherche les images .gif animées : `q=interesting`
```
https://api.cognitive.microsoft.com/bing/v7.0/images/search?q=interesting&imageType=AnimatedGif&mkt=en-us

```
Comme indiqué dans l’exemple suivant, la requête d’URL nécessite l’en-tête [Ocp-Apim-Subscription-Key](/rest/api/cognitiveservices-bingsearch/bing-images-api-v7-reference#headers).

L’exemple Java suivant crée et envoie la requête.

```
package gifSearch;
import java.net.*;
import java.util.*;
import java.io.*;
import javax.net.ssl.HttpsURLConnection;

import com.google.gson.Gson;
import com.google.gson.GsonBuilder;
import com.google.gson.JsonObject;
import com.google.gson.JsonParser;

/*
 * Gson: https://github.com/google/gson
 * Maven info:
 *     groupId: com.google.code.gson
 *     artifactId: gson
 *     version: 2.8.1
 *
 * Once you have compiled or downloaded gson-2.8.1.jar, assuming you have placed it in the
 * same folder as this file (BingImageSearch.java), you can compile and run this program at
 * the command line as follows.
 *
 * javac GIFsearch.java -classpath .;gson-2.8.1.jar -encoding UTF-8
 * java -cp .;gson-2.8.1.jar GIFsearch
 */


public class GIFsearch {

    // Replace the subscriptionKey string value with your valid subscription key.
    static String subscriptionKey = "YOUR-ACCESS-KEY";

    static String host = "https://api.cognitive.microsoft.com";
    static String path = "/bing/v7.0/images/search";

    static String searchTerm = "interesting";

    public static SearchResults SearchImages (String searchQuery) throws Exception {
        // construct URL of search request (endpoint + query string)
        URL url = new URL(host + path + "?q=" +  URLEncoder.encode(searchQuery, "UTF-8") + "&imageType=AnimatedGif&mkt=en-us");
        HttpsURLConnection connection = (HttpsURLConnection)url.openConnection();
        connection.setRequestProperty("Ocp-Apim-Subscription-Key", subscriptionKey);

        // receive JSON body
        InputStream stream = connection.getInputStream();
        String response = new Scanner(stream).useDelimiter("\\A").next();

        // construct result object for return
        SearchResults results = new SearchResults(new HashMap<String, String>(), response);

        // extract Bing-related HTTP headers
        Map<String, List<String>> headers = connection.getHeaderFields();
        for (String header : headers.keySet()) {
            if (header == null) continue;      // may have null key
            if (header.startsWith("BingAPIs-") || header.startsWith("X-MSEdge-")) {
                results.relevantHeaders.put(header, headers.get(header).get(0));
            }
        }

        stream.close();
        return results;
    }

    // pretty-printer for JSON; uses GSON parser to parse and re-serialize
    public static String prettify(String json_text) {
        JsonParser parser = new JsonParser();
        JsonObject json = parser.parse(json_text).getAsJsonObject();
        Gson gson = new GsonBuilder().setPrettyPrinting().create();
        return gson.toJson(json);
    }

    public static void main (String[] args) {
        if (subscriptionKey.length() != 32) {
            System.out.println("Invalid Bing Search API subscription key!");
            System.out.println("Please paste yours into the source code.");
            System.exit(1);
        }

        try {
            System.out.println("Searching the Web for: " + searchTerm);

            SearchResults result = SearchImages(searchTerm);

            System.out.println("\nRelevant HTTP Headers:\n");
            for (String header : result.relevantHeaders.keySet())
                System.out.println(header + ": " + result.relevantHeaders.get(header));

            System.out.println("\nJSON Response:\n");
            System.out.println(prettify(result.jsonResponse));
        }
        catch (Exception e) {
            e.printStackTrace(System.out);
            System.exit(1);
        }
    }

}

//Container class for search results encapsulates relevant headers and JSON data
class SearchResults{
 HashMap<String, String> relevantHeaders;
 String jsonResponse;
 SearchResults(HashMap<String, String> headers, String json) {
     relevantHeaders = headers;
     jsonResponse = json;
 }
}

```

## <a name="results"></a>Résultats
Le code obtient les résultats suivants en tant qu’objets JSON :

```json
    {
      "webSearchUrl": "https://www.bing.com/images/search?view\u003ddetai...",
      "name": "Very Interesting GIF - Thats Very Interesting - ...",
      "thumbnailUrl": "https://tse1.mm.bing.net/th?id\u003dOIP.yJX6Vz345JPK...",
      "datePublished": "2017-03-12T01:35:00.0000000Z",
      "contentUrl": "https://media.contoso.co/images/c895fa573df8e493ca8d0dec7d93b/raw",
      "hostPageUrl": "https://www.contoso.co/view/thats-very-interesting-christi...",
      "contentSize": "1295633 B",
      "encodingFormat": "animatedgif",
      "hostPageDisplayUrl": "https://www.contoso.co/view/thats-very-christian...",
      "width": 440,
      "height": 186,
      "thumbnail": {
        "width": 474,
        "height": 200
      },
      "imageInsightsToken": "ccid_yJX6Vz34*mid_9FF0FFA42AADA1357F042443D2103B40EA...",
      "insightsMetadata": {
        "recipeSourcesCount": 0,
        "bestRepresentativeQuery": {
          "text": "That\u0027s Very Interesting",
          "displayText": "That\u0027s Very Interesting",
          "webSearchUrl": "https://www.bing.com/images/search?q\u003dThat..."
        },
        "pagesIncludingCount": 19,
        "availableSizesCount": 2
      },
      "imageId": "9FF0FFA42AADA1357F042443D21030EAAA225F",
      "accentColor": "62452D"
    },

```

## <a name="next-steps"></a>Étapes suivantes
- [Démarrage rapide C#](quickstarts/csharp.md)
- [Tutoriel d’une application à page unique Recherche d’images](tutorial-bing-image-search-single-page-app.md)