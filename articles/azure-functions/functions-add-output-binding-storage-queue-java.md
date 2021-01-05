---
title: Connecter votre fonction Java à Stockage Azure
description: Découvrez comment connecter une fonction Java déclenchée par HTTP à Stockage Azure à l’aide d’une liaison de sortie de stockage de file d’attente.
author: KarlErickson
ms.custom: devx-track-java
ms.author: karler
ms.date: 10/14/2019
ms.topic: quickstart
zone_pivot_groups: java-build-tools-set
ms.openlocfilehash: b4381c4acbea8a3b7d86d9c32aaf9cea24cf15fa
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93422652"
---
# <a name="connect-your-java-function-to-azure-storage"></a>Connecter votre fonction Java à Stockage Azure

[!INCLUDE [functions-add-storage-binding-intro](../../includes/functions-add-storage-binding-intro.md)]

Cet article vous montre comment intégrer la fonction que vous avez créée dans l’[article du guide de démarrage rapide précédent](./create-first-function-cli-java.md?tabs=bash,browser) à une file d’attente Stockage Azure. La liaison de sortie que vous ajoutez à cette fonction écrit des données d’une requête HTTP dans un message en file d’attente.

La plupart des liaisons requièrent une chaîne de connexion stockée que Functions utilise pour accéder au service lié. Pour faciliter cette connexion, vous utilisez le compte de stockage que vous avez créé avec votre application de fonction. La connexion à ce compte est déjà stockée dans un paramètre d’application nommé `AzureWebJobsStorage`.  

## <a name="prerequisites"></a>Prérequis

Avant de commencer cet article, suivez les étapes de la [partie 1 du guide de démarrage rapide Java](./create-first-function-cli-java.md?tabs=bash,browser).

## <a name="download-the-function-app-settings"></a>Télécharger les paramètres de l’application de fonction

[!INCLUDE [functions-app-settings-download-cli](../../includes/functions-app-settings-download-local-cli.md)]

## <a name="enable-extension-bundles"></a>Activer les offres groupées d’extension

[!INCLUDE [functions-extension-bundles](../../includes/functions-extension-bundles.md)]

Vous pouvez maintenant ajouter la liaison de sortie de stockage à votre projet.

## <a name="add-an-output-binding"></a>Ajouter une liaison de sortie

[!INCLUDE [functions-add-output-binding-java-cli](../../includes/functions-add-output-binding-java-cli.md)]

## <a name="add-code-that-uses-the-output-binding"></a>Ajouter le code qui utilise la liaison de sortie

[!INCLUDE [functions-add-output-binding-java-code](../../includes/functions-add-output-binding-java-code.md)]

[!INCLUDE [functions-add-output-binding-java-test-cli](../../includes/functions-add-output-binding-java-test-cli.md)]

Vous êtes maintenant prêt à essayer la nouvelle liaison de sortie localement.

## <a name="run-the-function-locally"></a>Exécuter la fonction localement

Comme auparavant, utilisez la commande suivante pour générer le projet et démarrer le runtime Functions localement :

# <a name="maven"></a>[Maven](#tab/maven)
```bash
mvn clean package 
mvn azure-functions:run
```
# <a name="gradle"></a>[Gradle](#tab/gradle) 
```bash
gradle jar --info
gradle azureFunctionsRun
```
---

> [!NOTE]  
> Comme vous avez activé des bundles d’extension dans le fichier host.json, l’[extension de liaison de stockage](functions-bindings-storage-blob.md#add-to-your-functions-app) a été téléchargée et installée lors du démarrage, avec les autres extensions de liaison Microsoft.

Comme précédemment, déclenchez la fonction à partir de la ligne de commande en utilisant cURL dans une nouvelle fenêtre de terminal :

```CMD
curl -w "\n" http://localhost:7071/api/HttpTrigger-Java --data AzureFunctions
```

Cette fois, la liaison de sortie crée également une file d’attente nommée `outqueue` dans votre compte de stockage et ajoute un message avec cette même chaîne.

Ensuite, vous utilisez l’interface de ligne de commande Azure pour afficher la nouvelle file d’attente et vérifier qu’un message a été ajouté. Vous pouvez également afficher votre file d’attente en utilisant l’[Explorateur Stockage Azure][Azure Storage Explorer] ou sur le [portail Azure](https://portal.azure.com).

[!INCLUDE [functions-storage-account-set-cli](../../includes/functions-storage-account-set-cli.md)]

[!INCLUDE [functions-query-storage-cli](../../includes/functions-query-storage-cli.md)]

### <a name="redeploy-the-project"></a>Redéployer le projet 

Pour mettre à jour votre application publiée, réexécutez la commande suivante :  

# <a name="maven"></a>[Maven](#tab/maven)  
```bash
mvn azure-functions:deploy
```
# <a name="gradle"></a>[Gradle](#tab/gradle)  
```bash
gradle azureFunctionsDeploy
```
---

Là encore, vous pouvez utiliser cURL pour tester la fonction déployée. Comme précédemment, transmettez la valeur `AzureFunctions` dans le corps de la requête POST à l’URL, comme dans l’exemple suivant :

```bash
curl -w "\n" https://fabrikam-functions-20190929094703749.azurewebsites.net/api/HttpTrigger-Java?code=zYRohsTwBlZ68YF.... --data AzureFunctions
```

Vous pouvez [réexaminer le message de file d’attente de stockage](#query-the-storage-queue) pour vérifier que la liaison de sortie génère un nouveau message dans la file d’attente, comme prévu.

[!INCLUDE [functions-cleanup-resources](../../includes/functions-cleanup-resources.md)]

## <a name="next-steps"></a>Étapes suivantes

Vous avez mis à jour votre fonction déclenchée via HTTP pour écrire des données dans une file d’attente de stockage. Pour en savoir plus sur le développement de fonctions Azure Functions avec Java, consultez le [Guide des développeurs Java sur Azure Functions](functions-reference-java.md) et [Déclencheurs et liaisons Azure Functions](functions-triggers-bindings.md). Pour obtenir des exemples de projets Functions complets en Java, consultez les [exemples Functions en Java](/samples/browse/?products=azure-functions&languages=Java). 

Ensuite, vous devez activer la supervision Application Insights pour votre application de fonction :

> [!div class="nextstepaction"]
> [Activer l’intégration à Application Insights](configure-monitoring.md#add-to-an-existing-function-app)


[Azure Storage Explorer]: https://storageexplorer.com/
