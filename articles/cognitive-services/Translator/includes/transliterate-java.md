---
author: erhopf
ms.service: cognitive-services
ms.topic: include
ms.date: 08/06/2019
ms.custom: devx-track-java
ms.author: erhopf
ms.openlocfilehash: baf74969bdec495fad201a5f489e93edfabf53b6
ms.sourcegitcommit: a76ff927bd57d2fcc122fa36f7cb21eb22154cfa
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 07/28/2020
ms.locfileid: "87375516"
---
[!INCLUDE [Prerequisites](prerequisites-java.md)]

[!INCLUDE [Set up and use environment variables](setup-env-variables.md)]

## <a name="initialize-a-project-with-gradle"></a>Initialiser un projet avec Gradle

Commencez par créer un répertoire de travail pour ce projet. À partir de la ligne de commande (ou d’une session Terminal Server), exécutez cette commande :

```console
mkdir transliterate-sample
cd transliterate-sample
```

Ensuite, initialisez un projet Gradle. Cette commande crée les fichiers de build nécessaires pour Gradle, dont le plus important est le fichier `build.gradle.kts`, qui est utilisé au moment de l’exécution pour créer et configurer votre application. Exécutez cette commande à partir de votre répertoire de travail :

```console
gradle init --type basic
```

Quand vous êtes invité à choisir un **DSL**, sélectionnez **Kotlin**.

## <a name="configure-the-build-file"></a>Configurer le fichier de build

Recherchez le fichier `build.gradle.kts` et ouvrez-le dans votre IDE ou éditeur de texte habituel. Copiez-y ensuite cette configuration de build :

```
plugins {
    java
    application
}
application {
    mainClassName = "Transliterate"
}
repositories {
    mavenCentral()
}
dependencies {
    compile("com.squareup.okhttp:okhttp:2.5.0")
    compile("com.google.code.gson:gson:2.8.5")
}
```

Notez que cet exemple présente des dépendances sur OkHttp pour les requêtes HTTP, et sur Gson pour gérer et analyser JSON. Pour en savoir plus sur les configurations de build, consultez [Creating New Gradle Builds](https://guides.gradle.org/creating-new-gradle-builds/).

## <a name="create-a-java-file"></a>Créer un fichier Java

Créez maintenant un dossier pour votre exemple d’application. À partir de votre répertoire de travail, exécutez cette commande :

```console
mkdir -p src\main\java
```

Ensuite, dans ce dossier, créez un fichier nommé `Transliterate.java`.

## <a name="import-required-libraries"></a>Importer les bibliothèques nécessaires

Ouvrez `Transliterate.java` et ajoutez-y les instructions import suivantes :

```java
import java.io.*;
import java.net.*;
import java.util.*;
import com.google.gson.*;
import com.squareup.okhttp.*;
```


## <a name="define-variables"></a>Définir des variables

Vous devez tout d’abord créer une classe publique pour votre projet :

```java
public class Transliterate {
  // All project code goes here...
}
```

Ajoutez ces lignes à la classe `Transliterate`. D’abord, la clé d’abonnement et le point de terminaison sont lus à partir des variables d’environnement. Ensuite, vous voyez qu’en plus du paramètre `api-version`, deux paramètres supplémentaires ont été ajoutés à `url`. Ces paramètres servent à définir la langue d’entrée et les scripts de translittération. Dans cet exemple, il s’agit du japonais (`jpan`) et du latin (`latn`). 

```java
private static String subscriptionKey = System.getenv("TRANSLATOR_TEXT_SUBSCRIPTION_KEY");
private static String endpoint = System.getenv("TRANSLATOR_TEXT_ENDPOINT");
String url = endpoint + "/transliterate?api-version=3.0&language=ja&fromScript=jpan&toScript=latn";
```
Si vous utilisez un abonnement multiservice Cognitive Services, vous devez également inclure la `Ocp-Apim-Subscription-Region` dans vos paramètres de requête. [En savoir plus sur l’authentification sur l’abonnement multiservice](https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-reference#authentication).

## <a name="create-a-client-and-build-a-request"></a>Créer un client et générer une requête

Ajoutez cette ligne à la classe `Transliterate` pour instancier `OkHttpClient` :

```java
// Instantiates the OkHttpClient.
OkHttpClient client = new OkHttpClient();
```

Ensuite, générez la requête POST. Au besoin, changez le texte pour la translittération.

```java
// This function performs a POST request.
public String Post() throws IOException {
    MediaType mediaType = MediaType.parse("application/json");
    RequestBody body = RequestBody.create(mediaType,
            "[{\n\t\"Text\": \"こんにちは\"\n}]");
    Request request = new Request.Builder()
            .url(url).post(body)
            .addHeader("Ocp-Apim-Subscription-Key", subscriptionKey)
            .addHeader("Content-type", "application/json").build();
    Response response = client.newCall(request).execute();
    return response.body().string();
}
```

## <a name="create-a-function-to-parse-the-response"></a>Créer une fonction pour analyser la réponse

Cette fonction simple analyse et agrémente la réponse JSON retournée par le service Translator.

```java
// This function prettifies the json response.
public static String prettify(String json_text) {
    JsonParser parser = new JsonParser();
    JsonElement json = parser.parse(json_text);
    Gson gson = new GsonBuilder().setPrettyPrinting().create();
    return gson.toJson(json);
}
```

## <a name="put-it-all-together"></a>Assemblage

La dernière étape consiste à effectuer une requête et à obtenir une réponse. Ajoutez ces lignes à votre projet :

```java
public static void main(String[] args) {
    try {
        Transliterate transliterateRequest = new Transliterate();
        String response = transliterateRequest.Post();
        System.out.println(prettify(response));
    } catch (Exception e) {
        System.out.println(e);
    }
}
```

## <a name="run-the-sample-app"></a>Exécution de l'exemple d'application

Voilà, vous êtes prêt à exécuter votre exemple d’application. À partir de la ligne de commande (ou d’une session Terminal Server), accédez à la racine de votre répertoire de travail, puis exécutez cette commande :

```console
gradle build
```

Une fois la génération terminée, exécutez :

```console
gradle run
```

## <a name="sample-response"></a>Exemple de réponse

```json
[
  {
    "text": "konnichiwa",
    "script": "latn"
  }
]
```

## <a name="next-steps"></a>Étapes suivantes

Consultez les informations de référence sur l’API pour comprendre tout ce que vous pouvez faire avec Translator.

> [!div class="nextstepaction"]
> [Informations de référence sur l'API](https://docs.microsoft.com/azure/cognitive-services/translator/reference/v3-0-reference)
