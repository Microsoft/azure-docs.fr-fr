---
title: Créer une application de fonction Azure avec Java et Eclipse
description: Guide pratique pour créer et publier sur Azure Functions une application serverless simple déclenchée par HTTP avec Java et Eclipse.
author: jeffhollan
ms.topic: how-to
ms.date: 07/01/2018
ms.author: jehollan
ms.custom: mvc, devcenter, devx-track-java
ms.openlocfilehash: a4e240837dd3c95efc4dc83a7491f2ad9163435a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87373315"
---
# <a name="create-your-first-function-with-java-and-eclipse"></a>Créer votre première fonction avec Java et Eclipse 

Cet article vous explique comment créer un projet de fonction [serverless](https://azure.microsoft.com/solutions/serverless/) avec l’environnement de développement intégré (IDE) Eclipse et Apache Maven, comment le tester et le déboguer, puis comment le déployer sur Azure Functions. 

<!-- TODO ![Access a Hello World function from the command line with cURL](media/functions-create-java-maven/hello-azure.png) -->

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="set-up-your-development-environment"></a>Configurer l''environnement de développement

Pour développer une application de fonction avec Java et Eclipse, vous devez avoir installé les éléments suivants :

-  [Java Developer Kit (JDK)](https://www.azul.com/downloads/zulu/) version 8.
-  [Apache Maven](https://maven.apache.org) version 3.0 ou ultérieure.
-  [Eclipse](https://www.eclipse.org/downloads/packages/), avec prise en charge de Java et Maven.
-  [Azure CLI](/cli/azure)

> [!IMPORTANT] 
> Pour pouvoir effectuer ce démarrage rapide, vous devez définir la variable d’environnement JAVA_HOME sur l’emplacement d’installation du JDK.

Il est vivement recommandé d’installer également [Azure Functions Core Tools, version 2](functions-run-local.md#v2), qui offre un environnement local pour l’exécution et le débogage d’Azure Functions. 

## <a name="create-a-functions-project"></a>Créer un projet Functions

1. Dans Eclipse, sélectionnez le menu **Fichier**, puis **Nouveau -&gt; Projet Maven**. 
1. Acceptez les valeurs par défaut de la boîte de dialogue **Nouveau projet Maven**, puis sélectionnez **Suivant**.
1. Recherchez et sélectionnez [azure-functions-archetype](https://mvnrepository.com/artifact/com.microsoft.azure/azure-functions-archetype) et cliquez sur **Suivant**.
1. Veillez à renseigner les valeurs de tous les champs, y compris `resourceGroup`, `appName`et `appRegion` (utilisez un appName différent de **Fabrikam-Function-20170920120101928**), puis **Terminer**.
    ![Création de projet Maven dans Eclipse2](media/functions-create-first-java-eclipse/functions-create-eclipse2.png)  

Maven crée les fichiers projet dans un nouveau dossier avec le nom de _artifactId_. Le code généré dans le projet est une fonction simple [déclenchée par HTTP](./functions-bindings-http-webhook.md) qui retourne le corps de la demande HTTP à l’origine du déclenchement.

## <a name="run-functions-locally-in-the-ide"></a>Exécuter des fonctions localement dans l’IDE

> [!NOTE]
> [Azure Functions Core Tools, version 2](functions-run-local.md#v2) doit être installé pour pouvoir exécuter et déboguer des fonctions localement.

1. Cliquez avec le bouton droit sur le projet généré, puis choisissez **Exécuter en tant que** et **Build Maven**.
1. Dans la boîte de dialogue **Edit Configuration** (Modifier la configuration), entrez `package` dans les champs **Goals** (Objectifs) et **Nom**, puis sélectionnez **Exécuter**. Cette opération entraîne la génération et l’empaquetage du code de fonction.
1. Une fois la génération terminée, créez une autre configuration Exécuter comme ci-dessus, en utilisant `azure-functions:run` en tant qu’objectif et que nom. Sélectionnez **Exécuter** pour exécuter la fonction dans l’IDE.

Lorsque vous avez fini de tester votre fonction, terminez le runtime dans la fenêtre de console. Un seul hôte de fonction à la fois peut être actif et en cours d’exécution localement.

### <a name="debug-the-function-in-eclipse"></a>Déboguer la fonction dans Eclipse

Dans votre configuration **Exécuter en tant que** définie à l’étape précédente, remplacez `azure-functions:run` par `azure-functions:run -DenableDebug`, puis exécutez la configuration mise à jour pour démarrer l’application de fonction en mode débogage.

Sélectionnez le menu **Exécuter**, puis ouvrez **Déboguer les configurations**. Choisissez **Application Java distante** et créez une configuration. Donnez un nom à votre configuration et renseignez les paramètres. Le port doit être cohérent avec le port de débogage ouvert par l’hôte de fonction, qui correspond par défaut à `5005`. Après la configuration, cliquez sur `Debug` pour démarrer le débogage.

![Débogage de fonctions dans Eclipse](media/functions-create-first-java-eclipse/debug-configuration-eclipse.PNG)

Définissez des points d’arrêt et inspectez les objets de votre fonction à l’aide de l’IDE. Quand vous avez terminé, arrêtez le débogueur et l’hôte de fonction en cours d’exécution. Un seul hôte de fonction à la fois peut être actif et en cours d’exécution localement.

## <a name="deploy-the-function-to-azure"></a>Déployer la fonction sur Azure

Le processus de déploiement sur Azure Functions utilise les informations d’identification du compte provenant d’Azure CLI. [Connectez-vous avec Azure CLI](/cli/azure/authenticate-azure-cli?view=azure-cli-latest) avant de continuer en utilisant l’invite de commandes de votre ordinateur.

```azurecli
az login
```

Déployez votre code dans une nouvelle application de fonction en utilisant l’objectif Maven `azure-functions:deploy` dans une nouvelle configuration **Exécuter en tant que**.

Quand le déploiement est terminé, vous voyez l’URL que vous pouvez utiliser pour accéder à votre application de fonction Azure :

```output
[INFO] Successfully deployed Function App with package.
[INFO] Deleting deployment package from Azure Storage...
[INFO] Successfully deleted deployment package fabrikam-function-20170920120101928.20170920143621915.zip
[INFO] Successfully deployed Function App at https://fabrikam-function-20170920120101928.azurewebsites.net
[INFO] ------------------------------------------------------------------------
```

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur le développement de fonctions Java, consultez le [Guide du développeur de fonctions Java](functions-reference-java.md).
- Ajoutez des fonctions supplémentaires avec différents déclencheurs à votre projet en utilisant la cible Maven `azure-functions:add`.
