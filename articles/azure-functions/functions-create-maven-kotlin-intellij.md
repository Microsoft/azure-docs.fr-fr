---
title: Créer une fonction Azure avec Kotlin et IntelliJ
description: Découvrez comment créer et publier une application serverless simple déclenchée par HTTP sur Azure en utilisant Kotlin et IntelliJ.
author: dglover
ms.service: azure-functions
ms.topic: quickstart
ms.date: 03/25/2020
ms.author: dglover
ms.openlocfilehash: 09dd868dc9e05241943899654d7c8bb427a8f268
ms.sourcegitcommit: ae6e7057a00d95ed7b828fc8846e3a6281859d40
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/16/2020
ms.locfileid: "92104833"
---
# <a name="quickstart-create-your-first-http-triggered-function-with-kotlin-and-intellij"></a>Démarrage rapide : Créer votre première fonction déclenchée par HTTP avec Kotlin et IntelliJ

Cet article vous montre comment créer un projet de fonction [serverless](https://azure.microsoft.com/overview/serverless-computing/) avec IntelliJ IDEA et Apache Maven. Il montre également comment déboguer localement votre code de fonction dans l’environnement de développement intégré (IDE), puis déployer le projet de fonction dans Azure.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="set-up-your-development-environment"></a>Configurer l''environnement de développement

Pour développer une fonction avec Kotlin et IntelliJ, installez les logiciels suivants :

- [Kit de développeur Java](/azure/developer/java/fundamentals/java-jdk-long-term-support) (JDK), version 8
- [Apache Maven](https://maven.apache.org), version 3.0 ou ultérieure
- [IntelliJ IDEA](https://www.jetbrains.com/idea/download), versions Community ou Ultimate avec Maven
- [Azure CLI](/cli/azure)
- [Version 2.x](functions-run-local.md#v2) d’Azure Functions Core Tools. Il fournit un environnement de développement local pour l’écriture, l’exécution et le débogage des fonctions Azure Functions.

> [!IMPORTANT]
> Pour pouvoir suivre les différentes étapes de cet article, vous devez définir la variable d’environnement JAVA_HOME sur l’emplacement d’installation du JDK.

## <a name="create-a-functions-project"></a>Créer un projet Functions

1. Dans IntelliJ IDEA, sélectionnez **Create New Project** (Créer un projet).  
1. Dans la fenêtre **New Project** (Nouveau projet), sélectionnez **Maven** dans le volet gauche.
1. Cochez la case **Create from archetype** (Créer à partir d’un archétype) et sélectionnez **Add Archetype** (Ajouter un archétype) pour [azure-functions-kotlin-archetype](https://mvnrepository.com/artifact/com.microsoft.azure/azure-functions-kotlin-archetype).
1. Dans la fenêtre **Add Archetype** (Ajouter un archétype), renseignez les champs comme suit :
    - _GroupId_ : com.microsoft.azure
    - _ArtifactId_ : azure-functions-kotlin-archetype
    - _Version_ : utilisez la version la plus récente du [dépôt central](https://mvnrepository.com/artifact/com.microsoft.azure/azure-functions-kotlin-archetype)
    ![Créer un projet Maven à partir d’un archétype dans IntelliJ IDEA](media/functions-create-first-kotlin-intellij/functions-create-intellij.png)  
1. Sélectionnez **OK**, puis **Suivant**.
1. Entrez les détails de votre projet en cours et sélectionnez **Finish** (Terminer).

Maven crée les fichiers projet dans un nouveau dossier avec le même nom que la valeur _ArtifactId_. Le code généré du projet est une fonction simple, [déclenchée par HTTP](./functions-bindings-http-webhook.md), qui retourne le corps de la requête HTTP à l’origine du déclenchement.

## <a name="run-functions-locally-in-the-ide"></a>Exécuter des fonctions localement dans l’IDE

> [!NOTE]
> Pour exécuter et déboguer des fonctions localement, assurez-vous qu’[Azure Functions Core Tools, version 2](functions-run-local.md#v2) est installé.

1. Importez les modifications manuellement ou activez l’[importation automatique](https://www.jetbrains.com/help/idea/creating-and-optimizing-imports.html).
1. Ouvrez la barre d’outils **Projets Maven**.
1. Développez **Cycle de vie**, puis ouvrez **package**. La solution est générée et empaquetée dans un répertoire cible nouvellement créé.
1. Développez **Plug-ins** >  **azure-functions** et ouvrez **azure-functions:run** pour démarrer le runtime local d’Azure Functions.  
  ![Barre d’outils Maven pour Azure Functions](media/functions-create-first-kotlin-intellij/functions-intellij-kotlin-maven-toolbar.png)  

1. Fermez la boîte de dialogue d’exécution quand vous avez fini de tester votre fonction. Un seul hôte de fonction à la fois peut être actif et en cours d’exécution localement.

## <a name="debug-the-function-in-intellij"></a>Déboguer la fonction dans IntelliJ

1. Pour démarrer l’hôte de fonction en mode débogage, ajoutez **-DenableDebug** comme argument quand vous exécutez votre fonction. Vous pouvez modifier la configuration dans les [objectifs maven](https://www.jetbrains.com/help/idea/maven-support.html#run_goal) ou exécuter la commande suivante dans une fenêtre de terminal :  

   ```
   mvn azure-functions:run -DenableDebug
   ```

   Cette commande force l’hôte de la fonction à ouvrir un port de débogage sur 5005.

1. Dans le menu **Run** (Exécuter), sélectionnez **Edit Configurations** (Modifier les configurations).
1. Sélectionnez **+** pour ajouter une configuration **Remote** (Distante).
1. Renseignez les champs _Name_ (Nom) et _Settings_ (Paramètres), puis sélectionnez **OK** pour enregistrer la configuration.
1. Après l’installation, sélectionnez **Debug (Déboguer) < Nom de la configuration distante >** ou appuyez sur les touches MAJ+F9 de votre clavier pour démarrer le débogage.

   ![Déboguer des fonctions dans IntelliJ](media/functions-create-first-kotlin-intellij/debug-configuration-intellij.PNG)

1. Quand vous avez terminé, arrêtez le débogueur et le processus en cours d’exécution. Un seul hôte de fonction à la fois peut être actif et en cours d’exécution localement.

## <a name="deploy-the-function-to-azure"></a>Déployer la fonction sur Azure

1. Avant de pouvoir déployer votre fonction sur Azure, vous devez vous [connecter par l’intermédiaire de l’interface Azure CLI](/cli/azure/authenticate-azure-cli?view=azure-cli-latest).

   ``` azurecli
   az login
   ```

1. Déployez votre code dans une nouvelle fonction en utilisant la cible Maven `azure-functions:deploy`. Vous pouvez également sélectionner l’option **azure-fonctions:deploy** dans la fenêtre des projets Maven.

   ```
   mvn azure-functions:deploy
   ```

1. Trouvez l’URL de votre fonction dans la sortie de l’interface Azure CLI une fois la fonction déployée avec succès.

   ``` output
   [INFO] Successfully deployed Function App with package.
   [INFO] Deleting deployment package from Azure Storage...
   [INFO] Successfully deleted deployment package fabrikam-function-20170920120101928.20170920143621915.zip
   [INFO] Successfully deployed Function App at https://fabrikam-function-20170920120101928.azurewebsites.net
   [INFO] ------------------------------------------------------------------------
   ```

## <a name="next-steps"></a>Étapes suivantes

À présent que vous avez déployé votre première fonction Kotlin sur Azure, consultez le [Guide du développeur de fonctions Java](functions-reference-java.md) pour plus d’informations sur le développement de fonctions Java et Kotlin.
- Ajoutez à votre projet des fonctions supplémentaires avec différents déclencheurs en utilisant la cible Maven `azure-functions:add`.