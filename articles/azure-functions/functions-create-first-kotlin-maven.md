---
title: Créer votre première fonction dans Azure avec Kotlin et Maven
description: Créez et publiez une fonction déclenchée par HTTP sur Azure avec Kotlin et Maven.
author: dglover
ms.service: azure-functions
ms.topic: quickstart
ms.date: 03/25/2020
ms.author: dglover
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: acb4290a90532b38f44a957e33c69f7d2b3f11f8
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92104816"
---
# <a name="quickstart-create-your-first-function-with-kotlin-and-maven"></a>Démarrage rapide : Créer votre première fonction avec Kotlin et Maven

Cet article vous guide dans l’utilisation de l’outil en ligne de commande Maven pour générer et publier un projet de fonction Kotlin sur Azure Functions. Lorsque vous avez terminé, le code de votre fonction s’exécute sur le [plan de consommation](functions-scale.md#consumption-plan) dans Azure et peut être déclenché à l’aide d’une requête HTTP.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

Pour développer des fonctions en utilisant Kotlin, les éléments suivants doivent être installés :

- [Java Developer Kit (JDK)](/azure/developer/java/fundamentals/java-jdk-long-term-support), version 8
- [Apache Maven](https://maven.apache.org), version 3.0 ou ultérieure
- [Azure CLI](/cli/azure)
- [Azure Functions Core Tools](./functions-run-local.md#v2) version 2.6.666 ou ultérieure

> [!IMPORTANT]
> Pour pouvoir effectuer ce démarrage rapide, vous devez définir la variable d’environnement JAVA_HOME sur l’emplacement d’installation du JDK.

## <a name="generate-a-new-functions-project"></a>Générer un nouveau projet Functions

Dans un dossier vide, exécutez la commande suivante pour générer le projet Functions à partir d’un [archétype Maven](https://maven.apache.org/guides/introduction/introduction-to-archetypes.html).

# <a name="bash"></a>[bash](#tab/bash)
```bash
mvn archetype:generate \
    -DarchetypeGroupId=com.microsoft.azure \
    -DarchetypeArtifactId=azure-functions-kotlin-archetype
```

> [!NOTE]
> Si vous rencontrez des problèmes avec l’exécution d’une commande, jetez un coup d’œil à la version `maven-archetype-plugin` utilisée. Étant donné que vous exécutez la commande dans un répertoire vide dépourvu de fichier `.pom`, celle-ci essaie peut-être d’utiliser un plug-in de l’ancienne version à partir de `~/.m2/repository/org/apache/maven/plugins/maven-archetype-plugin` si vous avez mis à niveau Maven à partir d’une version antérieure. Dans ce cas, essayez de supprimer le répertoire `maven-archetype-plugin` et de réexécuter la commande.

# <a name="powershell"></a>[PowerShell](#tab/powershell)
```powershell
mvn archetype:generate `
    "-DarchetypeGroupId=com.microsoft.azure" `
    "-DarchetypeArtifactId=azure-functions-kotlin-archetype"
```

# <a name="cmd"></a>[Cmd](#tab/cmd)
```cmd
mvn archetype:generate ^
    -DarchetypeGroupId=com.microsoft.azure ^
    -DarchetypeArtifactId=azure-functions-kotlin-archetype
```
---

Maven vous demande d’entrer les valeurs nécessaires pour terminer la génération du projet. Pour les valeurs de _groupId_, _artifactId_ et _version_, consultez les informations de référence sur les [conventions de nommage de Maven](https://maven.apache.org/guides/mini/guide-naming-conventions.html). La valeur de _appName_ doit être unique dans Azure : par défaut, Maven génère donc un nom d’application basé sur la valeur précédemment entrée pour _artifactId_. La valeur de _packageName_ détermine le package Kotlin pour le code de fonction généré.

Les identificateurs `com.fabrikam.functions` et `fabrikam-functions` ci-dessous sont utilisés en tant qu’exemples et pour faciliter la lecture des étapes ultérieures de ce démarrage rapide. Dans cette étape, vous êtes invité à fournir vos propres valeurs pour Maven.

<pre>
[INFO] Parameter: groupId, Value: com.fabrikam.function
[INFO] Parameter: artifactId, Value: fabrikam-function
[INFO] Parameter: version, Value: 1.0-SNAPSHOT
[INFO] Parameter: package, Value: com.fabrikam.function
[INFO] Parameter: packageInPathFormat, Value: com/fabrikam/function
[INFO] Parameter: appName, Value: fabrikam-function-20190524171507291
[INFO] Parameter: resourceGroup, Value: java-functions-group
[INFO] Parameter: package, Value: com.fabrikam.function
[INFO] Parameter: version, Value: 1.0-SNAPSHOT
[INFO] Parameter: groupId, Value: com.fabrikam.function
[INFO] Parameter: appRegion, Value: westus
[INFO] Parameter: artifactId, Value: fabrikam-function
</pre>

Maven crée les fichiers projet dans un nouveau dossier avec le nom de _artifactId_, dans cet exemple `fabrikam-functions`. Le code généré, et prêt à être utilisé, dans le projet est une fonction simple [déclenchée par HTTP](./functions-bindings-http-webhook.md) qui renvoie le corps de la requête :

```kotlin
class Function {

    /**
     * This function listens at endpoint "/api/HttpTrigger-Java". Two ways to invoke it using "curl" command in bash:
     * 1. curl -d "HTTP Body" {your host}/api/HttpTrigger-Java&code={your function key}
     * 2. curl "{your host}/api/HttpTrigger-Java?name=HTTP%20Query&code={your function key}"
     * Function Key is not needed when running locally, it is used to invoke function deployed to Azure.
     * More details: https://aka.ms/functions_authorization_keys
     */
    @FunctionName("HttpTrigger-Java")
    fun run(
            @HttpTrigger(
                    name = "req",
                    methods = [HttpMethod.GET, HttpMethod.POST],
                    authLevel = AuthorizationLevel.FUNCTION) request: HttpRequestMessage<Optional<String>>,
            context: ExecutionContext): HttpResponseMessage {

        context.logger.info("HTTP trigger processed a ${request.httpMethod.name} request.")

        val query = request.queryParameters["name"]
        val name = request.body.orElse(query)

        name?.let {
            return request
                    .createResponseBuilder(HttpStatus.OK)
                    .body("Hello, $name!")
                    .build()
        }

        return request
                .createResponseBuilder(HttpStatus.BAD_REQUEST)
                .body("Please pass a name on the query string or in the request body")
                .build()
    }
}
```

## <a name="run-the-function-locally"></a>Exécuter la fonction localement

Accédez au dossier du projet nouvellement créé, puis générez et exécutez la fonction avec Maven :

```
cd fabrikam-function
mvn clean package
mvn azure-functions:run
```

> [!NOTE]
> Si vous rencontrez cette exception : `javax.xml.bind.JAXBException` avec Java 9, consultez la solution de contournement sur [GitHub](https://github.com/jOOQ/jOOQ/issues/6477).

Vous voyez cette sortie lorsque la fonction s’exécute localement sur votre système et quand elle est prête à répondre aux requêtes HTTP :

<pre>
Now listening on: http://0.0.0.0:7071
Application started. Press Ctrl+C to shut down.

Http Functions:

        HttpTrigger-Java: [GET,POST] http://localhost:7071/api/HttpTrigger-Java
</pre>

Déclenchez la fonction à partir de la ligne de commande en utilisant cURL dans une nouvelle fenêtre de terminal :

```
curl -w '\n' -d LocalFunction http://localhost:7071/api/HttpTrigger-Java
```

<pre>
Hello LocalFunction!
</pre>

Utilisez `Ctrl-C` dans le terminal pour arrêter le code de la fonction.

## <a name="deploy-the-function-to-azure"></a>Déployer la fonction sur Azure

Le processus de déploiement sur Azure Functions utilise les informations d’identification du compte provenant d’Azure CLI. [Connectez-vous à Azure CLI](/cli/azure/authenticate-azure-cli?view=azure-cli-latest) avant de continuer.

```azurecli
az login
```

Déployez votre code dans une nouvelle application de fonction en utilisant la cible Maven `azure-functions:deploy`.

> [!NOTE]
> Quand vous utilisez Visual Studio Code pour déployer votre application de fonction, n’oubliez pas de choisir un abonnement payant, sinon vous obtenez une erreur. Votre abonnement est indiqué sur le côté gauche de l’IDE.

```
mvn azure-functions:deploy
```

Quand le déploiement est terminé, vous voyez l’URL que vous pouvez utiliser pour accéder à votre application de fonction Azure :

<pre>
[INFO] Successfully deployed Function App with package.
[INFO] Deleting deployment package from Azure Storage...
[INFO] Successfully deleted deployment package fabrikam-function-20170920120101928.20170920143621915.zip
[INFO] Successfully deployed Function App at https://fabrikam-function-20170920120101928.azurewebsites.net
[INFO] ------------------------------------------------------------------------
</pre>

Testez l’application de fonction en cours d’exécution sur Azure avec `cURL`. Vous devez modifier l’URL de l’exemple ci-dessous pour qu’elle corresponde à l’URL déployée de votre application de fonction à l’étape précédente.

> [!NOTE]
> Veillez à définir les **Droits d’accès** sur `Anonymous`. Lorsque vous choisissez le niveau par défaut de `Function`, vous êtes invité à présenter la [clé de fonction](functions-bindings-http-webhook-trigger.md#authorization-keys) dans les demandes d’accès à votre point de terminaison de fonction.

```
curl -w '\n' https://fabrikam-function-20170920120101928.azurewebsites.net/api/HttpTrigger-Java -d AzureFunctions
```

```Output
Hello AzureFunctions!
```

## <a name="make-changes-and-redeploy"></a>Effectuer des modifications et redéployer

Modifiez le fichier source `src/main.../Function.java` dans le projet généré pour modifier le texte retourné par votre application de fonction. Remplacez cette ligne :

```kotlin
return request
        .createResponseBuilder(HttpStatus.OK)
        .body("Hello, $name!")
        .build()
```

Par le code suivant :

```kotlin
return request
        .createResponseBuilder(HttpStatus.OK)
        .body("Hi, $name!")
        .build()
```

Enregistrez les modifications et redéployez l’application en exécutant `azure-functions:deploy` à partir du terminal comme vous l’avez fait précédemment. L’application de fonction sera mise à jour et cette requête :

```
curl -w '\n' -d AzureFunctionsTest https://fabrikam-functions-20170920120101928.azurewebsites.net/api/HttpTrigger-Java
```

La sortie mise à jour s’affiche :

<pre>
Hi, AzureFunctionsTest
</pre>


## <a name="reference-bindings"></a>Liaisons de référence

Pour travailler avec des [déclencheurs et liaisons Functions](functions-triggers-bindings.md) autres que le déclencheur HTTP et le déclencheur de minuteur, vous devez installer des extensions de liaison. Même si cet article ne le demande pas, vous devrez savoir comment activer des extensions lorsque vous utiliserez d’autres types de liaisons.

[!INCLUDE [functions-extension-bundles](../../includes/functions-extension-bundles.md)]

## <a name="next-steps"></a>Étapes suivantes

Vous avez créé une application de fonction Kotlin avec un déclencheur HTTP simple, puis vous l’avez déployée sur Azure Functions.

- Pour plus d’informations sur le développement de fonctions Java et Kotlin, consultez le [Guide du développeur de fonctions Java](functions-reference-java.md).
- Ajoutez des fonctions supplémentaires avec différents déclencheurs à votre projet en utilisant la cible Maven `azure-functions:add`.
- Écrivez et déboguez des fonctions en local avec [Visual Studio Code](https://code.visualstudio.com/docs/java/java-azurefunctions), [IntelliJ](functions-create-maven-intellij.md) et [Eclipse](functions-create-maven-eclipse.md). 
- Déboguez les fonctions déployées dans Azure avec Visual Studio Code. Pour obtenir des instructions, consultez la documentation relative aux [applications Java sans serveur](https://code.visualstudio.com/docs/java/java-serverless#_remote-debug-functions-running-in-the-cloud) Visual Studio Code.