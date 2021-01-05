---
title: Utiliser Java et Gradle pour publier une fonction sur Azure
description: Créez et publiez une fonction déclenchée par HTTP sur Azure avec Java et Gradle.
author: KarlErickson
ms.custom: devx-track-java
ms.author: karler
ms.topic: how-to
ms.date: 04/08/2020
ms.openlocfilehash: 48a732e3935d78bdbf8b81fe989b59be1fbe2203
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96020430"
---
# <a name="use-java-and-gradle-to-create-and-publish-a-function-to-azure"></a>Utiliser Java et Gradle pour créer et publier une fonction sur Azure

Cet article montre comment générer et publier un projet de fonction Java sur Azure Functions avec l’outil en ligne de commande Gradle. Quand vous avez terminé, votre code de fonction s’exécute dans Azure dans un [plan d’hébergement serverless](functions-scale.md#consumption-plan) et est déclenché par une requête HTTP. 

> [!NOTE]
> Si Gradle n’est pas votre outil de développement préféré, consultez nos tutoriels similaires, destinés aux développeurs Java utilisant [Maven](./create-first-function-cli-java.md), [IntelliJ IDEA](/azure/developer/java/toolkit-for-intellij/quickstart-functions) et [VS Code](./create-first-function-vs-code-java.md).

## <a name="prerequisites"></a>Prérequis

Pour développer des fonctions avec Java, les éléments suivants doivent être installés :

- [Java Developer Kit (JDK)](/azure/developer/java/fundamentals/java-jdk-long-term-support), version 8
- [Azure CLI]
- [Azure Functions Core Tools](./functions-run-local.md#v2) version 2.6.666 ou ultérieure
- [Gradle](https://gradle.org/), version 4.10 ou ultérieure

Vous avez également besoin d’un abonnement Azure actif. [!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

> [!IMPORTANT]
> Pour pouvoir effectuer ce démarrage rapide, vous devez définir la variable d’environnement JAVA_HOME sur l’emplacement d’installation du JDK.

## <a name="prepare-a-functions-project"></a>Préparer un projet Functions

Utilisez la commande suivante pour cloner l’exemple de projet :

```bash
git clone https://github.com/Azure-Samples/azure-functions-samples-java.git
cd azure-functions-samples-java/
```

Ouvrez `build.gradle` et remplacez `appName` dans la section suivante par un nom unique pour éviter les conflits de noms de domaine lors du déploiement sur Azure. 

```gradle
azurefunctions {
    resourceGroup = 'java-functions-group'
    appName = 'azure-functions-sample-demo'
    pricingTier = 'Consumption'
    region = 'westus'
    runtime {
      os = 'windows'
    }
    localDebug = "transport=dt_socket,server=y,suspend=n,address=5005"
}
```

Ouvrez le nouveau fichier Function.java à partir du chemin *src/main/java* dans un éditeur de texte et passez en revue le code généré. Ce code est une fonction [déclenchée par HTTP](functions-bindings-http-webhook.md) qui renvoie le corps de la requête. 

> [!div class="nextstepaction"]
> [J’ai rencontré un problème](https://www.research.net/r/javae2e?tutorial=functions-create-first-java-gradle&step=generate-project)

## <a name="run-the-function-locally"></a>Exécuter la fonction localement

Exécutez la commande suivante pour générer et exécuter le projet de fonction :

```bash
gradle jar --info
gradle azureFunctionsRun
```
Une sortie semblable à celle-ci est retournée par Azure Functions Core Tools quand vous exécutez le projet localement :

<pre>
...

Now listening on: http://0.0.0.0:7071
Application started. Press Ctrl+C to shut down.

Http Functions:

    HttpExample: [GET,POST] http://localhost:7071/api/HttpExample
...
</pre>

Déclenchez la fonction à partir de la ligne de commande en utilisant la commande cURL suivante dans une nouvelle fenêtre de terminal :

```bash
curl -w "\n" http://localhost:7071/api/HttpExample --data AzureFunctions
```

La sortie attendue est la suivante :

<pre>
Hello, AzureFunctions
</pre>

> [!NOTE]
> Si vous définissez authLevel sur `FUNCTION` ou `ADMIN`, la [clé de fonction](functions-bindings-http-webhook-trigger.md#authorization-keys) n'est pas nécessaire lors de l'exécution locale.  

Utilisez `Ctrl+C` dans le terminal pour arrêter le code de la fonction.

> [!div class="nextstepaction"]
> [J’ai rencontré un problème](https://www.research.net/r/javae2e?tutorial=functions-create-first-java-gradle&step=local-run)

## <a name="deploy-the-function-to-azure"></a>Déployer la fonction sur Azure

Une application de fonction et les ressources associées sont créées dans Azure lors du déploiement initial de votre application de fonction. Avant de procéder au déploiement, utilisez la commande Azure CLI [az login](/cli/azure/authenticate-azure-cli) pour vous connecter à votre abonnement Azure. 

```azurecli
az login
```

> [!TIP]
> Si votre compte peut accéder à plusieurs abonnements, utilisez [az account set](/cli/azure/account#az-account-set) pour définir l’abonnement par défaut pour cette session. 

Utilisez la commande suivante pour déployer votre projet vers une nouvelle application de fonction. 

```bash
gradle azureFunctionsDeploy
```

Cela crée les ressources suivantes dans Azure, en fonction des valeurs du fichier build.gradle :

+ Groupe de ressources. Nommé avec le _resourceGroup_ que vous avez fourni.
+ Compte de stockage. Requis par les fonctions. Le nom est généré de façon aléatoire en fonction des exigences du nom de compte de stockage.
+ Plan App Service. Hébergement de Plan de consommation serverless pour votre application de fonction dans l’_appRegion_ spécifiée. Le nom est généré de façon aléatoire.
+ Application de fonction. Une application de fonction est l’unité de déploiement et d’exécution de vos fonctions. Le nom est l’_appName_, auquel est ajouté un numéro généré de façon aléatoire. 

Le déploiement empaquette également les fichiers projet et les déploie dans la nouvelle application de fonction à l’aide de [zip deployment](functions-deployment-technologies.md#zip-deploy), en activant le mode d’exécution à partir du package.

Le paramètre authLevel du déclencheur HTTP de l'exemple de projet est `ANONYMOUS`, ce qui permet d'ignorer l'authentification. Cependant, si vous utilisez un autre niveau d'authentification, comme `FUNCTION` ou `ADMIN`, vous devez obtenir la clé de fonction pour appeler le point de terminaison de fonction sur HTTP. Le moyen le plus simple d’obtenir la clé de fonction consiste à passer par le [Azure portal].

> [!div class="nextstepaction"]
> [J’ai rencontré un problème](https://www.research.net/r/javae2e?tutorial=functions-create-first-java-gradle&step=deploy)

## <a name="get-the-http-trigger-url"></a>Obtient l’URL du déclencheur HTTP

Vous pouvez obtenir l’URL requise pour déclencher votre fonction, avec la clé de fonction, à partir du portail Azure. 

1. Accédez au [Azure portal], connectez-vous, tapez l’_appName_ de votre application de fonction dans **Rechercher** en haut de la page, puis appuyez sur Entrée.
 
1. Dans votre application de fonction, sélectionnez **Fonctions**, choisissez votre fonction, puis cliquez sur **</> Obtenir l'URL de fonction** en haut à droite. 

    :::image type="content" source="./media/functions-create-first-java-gradle/get-function-url-portal.png" alt-text="Copier l'URL de fonction à partir du portail Azure":::

1. Choisissez **par défaut (Clé de fonction)** et sélectionnez **Copier**. 

Vous pouvez maintenant utiliser l’URL copiée pour accéder à votre fonction.

## <a name="verify-the-function-in-azure"></a>Vérifier la fonction dans Azure

Pour vérifier que l’application de fonction s’exécute sur Azure à l’aide de `cURL`, remplacez l’URL de l’exemple ci-dessous par celle que vous avez copiée à partir du portail.

```console
curl -w "\n" http://azure-functions-sample-demo.azurewebsites.net/api/HttpExample --data AzureFunctions
```

Cela envoie une requête POST au point de terminaison de fonction avec `AzureFunctions` dans le corps de la requête. Vous obtenez la réponse suivante.

<pre>
Hello, AzureFunctions
</pre>

> [!div class="nextstepaction"]
> [J’ai rencontré un problème](https://www.research.net/r/javae2e?tutorial=functions-create-first-java-gradle&step=verify-deployment)

## <a name="next-steps"></a>Étapes suivantes

Vous avez créé un projet de fonctions Java avec une fonction déclenchée par HTTP, vous l’avez exécuté sur votre ordinateur local et vous l’avez déployée sur Azure. À présent, étendez votre fonction en effectuant un...

> [!div class="nextstepaction"]
> [Ajout de liaison de sortie de file d’attente Stockage Azure](functions-add-output-binding-storage-queue-java.md)


[Azure CLI]: /cli/azure
[Azure portal]: https://portal.azure.com