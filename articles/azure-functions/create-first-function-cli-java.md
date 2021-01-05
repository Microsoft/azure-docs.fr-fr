---
title: Créer une fonction Java à partir de la ligne de commande – Azure Functions
description: Apprenez à créer une fonction Java à partir de la ligne de commande, puis à publier le projet local sur un hébergement serverless dans Azure Functions.
ms.date: 11/03/2020
ms.topic: quickstart
ms.custom:
- devx-track-java
- devx-track-azurecli
ms.openlocfilehash: 504d3a834760605149c93c0beea84bbe1b59e263
ms.sourcegitcommit: 17e9cb8d05edaac9addcd6e0f2c230f71573422c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/21/2020
ms.locfileid: "97707552"
---
# <a name="quickstart-create-a-java-function-in-azure-from-the-command-line"></a>Démarrage rapide : Créer une fonction Java dans Azure à partir de la ligne de commande

[!INCLUDE [functions-language-selector-quickstart-cli](../../includes/functions-language-selector-quickstart-cli.md)]

Dans cet article, vous allez utiliser des outils en ligne de commande pour créer une fonction Java qui répond aux requêtes HTTP. Après avoir testé le code localement, vous le déployez dans l’environnement serverless d’Azure Functions.

Le fait de suivre ce guide de démarrage rapide entraîne une petite dépense de quelques cents USD tout au plus dans votre compte Azure.

> [!NOTE]
> Si Maven n’est pas votre outil de développement préféré, consultez nos tutoriels similaires destinés aux développeurs Java utilisant [Gradle](./functions-create-first-java-gradle.md), [IntelliJ IDEA](/azure/developer/java/toolkit-for-intellij/quickstart-functions) et [Visual Studio Code](create-first-function-vs-code-java.md).

## <a name="configure-your-local-environment"></a>Configurer votre environnement local

Avant de commencer la lecture cet article, vous devez disposer des éléments suivants :

+ Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio).

+ [Azure Functions Core Tools](functions-run-local.md#v2) version 3.x.

+ [Azure CLI](/cli/azure/install-azure-cli) version 2.4 ou ultérieure.

+ [Java Developer Kit (JDK)](/azure/developer/java/fundamentals/java-jdk-long-term-support) version 8 ou 11. La variable d’environnement `JAVA_HOME` doit être définie sur l’emplacement d’installation de la version appropriée du kit JDK.     

+ [Apache Maven](https://maven.apache.org) version 3.0 ou ultérieure.

### <a name="prerequisite-check"></a>Vérification du prérequis

+ Dans une fenêtre de terminal ou de commande, exécutez `func --version` pour vérifier que vous disposez d’Azure Functions Core Tools version 3.x.

+ Exécutez `az --version` pour vérifier que vous disposez d’Azure CLI version 2.4 ou ultérieure.

+ Exécutez `az login` pour vous connecter à Azure et vérifier que l’abonnement est actif.

## <a name="create-a-local-function-project"></a>Créer un projet de fonction local

Dans Azure Functions, un projet de fonction est un conteneur pour une ou plusieurs fonctions qui répondent chacune à un déclencheur spécifique. Toutes les fonctions d’un projet partagent les mêmes configurations locale et d’hébergement. Dans cette section, vous créez un projet de fonction qui contient une seule fonction.

1. Dans un dossier vide, exécutez la commande suivante pour générer le projet Functions à partir d’un [archétype Maven](https://maven.apache.org/guides/introduction/introduction-to-archetypes.html). 

    # <a name="bash"></a>[Bash](#tab/bash)
    
    ```bash
    mvn archetype:generate -DarchetypeGroupId=com.microsoft.azure -DarchetypeArtifactId=azure-functions-archetype -DjavaVersion=8
    ```
    
    # <a name="powershell"></a>[PowerShell](#tab/powershell)
    
    ```powershell
    mvn archetype:generate "-DarchetypeGroupId=com.microsoft.azure" "-DarchetypeArtifactId=azure-functions-archetype" "-DjavaVersion=8" 
    ```
    
    # <a name="cmd"></a>[Cmd](#tab/cmd)
    
    ```cmd
    mvn archetype:generate "-DarchetypeGroupId=com.microsoft.azure" "-DarchetypeArtifactId=azure-functions-archetype" "-DjavaVersion=8"
    ```
    
    ---

    > [!IMPORTANT]
    > + Utilisez `-DjavaVersion=11` si vous voulez que vos fonctions s’exécutent sur Java 11. Pour plus d’informations, consultez les [versions Java](functions-reference-java.md#java-versions). 
    > + La variable d’environnement `JAVA_HOME` doit être définie sur l’emplacement d’installation de la version appropriée du JDK pour que vous puissiez suivre cet article.

1. Maven vous invite à entrer les valeurs nécessaires pour terminer la génération du projet lors du déploiement.   
    Fournissez les valeurs suivantes à l’invite :

    | Prompt | Valeur | Description |
    | ------ | ----- | ----------- |
    | **groupId** | `com.fabrikam` | Valeur qui identifie de façon unique votre projet parmi tous les projets, avec respect des [règles de nommage de package](https://docs.oracle.com/javase/specs/jls/se6/html/packages.html#7.7) pour Java. |
    | **artifactId** | `fabrikam-functions` | Valeur qui correspond au nom du fichier jar, sans numéro de version. |
    | **version** | `1.0-SNAPSHOT` | Choisissez la valeur par défaut. |
    | **package** | `com.fabrikam` | Valeur qui correspond au package Java pour le code de fonction généré. Utilisez la valeur par défaut. |

1. Tapez `Y` ou appuyez sur Entrée pour confirmer.

    Maven crée les fichiers projet dans un nouveau dossier avec le nom d’_artifactId_, qui est `fabrikam-functions` dans cet exemple. 

1. Accédez au dossier du projet :

    ```console
    cd fabrikam-functions
    ```

    Ce dossier contient divers fichiers pour le projet, notamment des fichiers config nommés [local.settings.json](functions-run-local.md#local-settings-file) et [host.json](functions-host-json.md). Dans la mesure où *local.settings.json* peut contenir des secrets téléchargés à partir d’Azure, le fichier est exclu du contrôle de code source par défaut dans le fichier *.gitignore*.

### <a name="optional-examine-the-file-contents"></a>(Facultatif) Examiner le contenu du fichier

Si vous le souhaitez, vous pouvez passer à [Exécuter la fonction localement](#run-the-function-locally) et examiner le contenu du fichier plus tard.

#### <a name="functionjava"></a>Function.java
*Function.java* contient une méthode `run` qui reçoit des données de requête dans la variable `request` d’un objet [HttpRequestMessage](/java/api/com.microsoft.azure.functions.httprequestmessage), doté de l’annotation [HttpTrigger](/java/api/com.microsoft.azure.functions.annotation.httptrigger) qui définit le comportement du déclencheur. 

:::code language="java" source="~/azure-functions-samples-java/src/main/java/com/functions/Function.java":::

Le message de réponse est généré par l’API [HttpResponseMessage. Builder](/java/api/com.microsoft.azure.functions.httpresponsemessage.builder).

#### <a name="pomxml"></a>pom.xml

Les paramètres des ressources Azure créées pour héberger votre application sont définis dans l’élément de **configuration** du plug-in avec un **groupId** de `com.microsoft.azure` dans le fichier pom.xml généré. Par exemple, l’élément de configuration ci-dessous indique à un déploiement basé sur Maven de créer une application de fonction dans le groupe de ressources `java-functions-group` de la région `westus`. L’application de fonction elle-même s’exécute sur Windows hébergé dans le plan `java-functions-app-service-plan` qui, par défaut, est un plan de consommation serverless.

:::code language="java" source="~/azure-functions-samples-java/pom.xml" range="62-102":::

Vous pouvez modifier ces paramètres pour contrôler la façon dont les ressources sont créées dans Azure, par exemple en modifiant `runtime.os` pour passer de `windows` à `linux` avant le déploiement initial. Pour obtenir la liste complète des paramètres pris en charge par le plug-in Maven, consultez les [détails de la configuration](https://github.com/microsoft/azure-maven-plugins/wiki/Azure-Functions:-Configuration-Details).

#### <a name="functiontestjava"></a>FunctionTest.java

L’archétype génère également un test unitaire pour votre fonction. Lorsque vous modifiez votre fonction pour ajouter des liaisons ou de nouvelles fonctions au projet, vous devez également modifier les tests dans le fichier *FunctionTest.java*.

## <a name="run-the-function-locally"></a>Exécuter la fonction localement

1. Exécutez votre fonction en démarrant l’hôte du runtime d’Azure Functions local à partir du dossier *LocalFunctionProj* :

    ```console
    mvn clean package
    mvn azure-functions:run
    ```
    
    Vers la fin de la sortie, les lignes suivantes doivent s’afficher :
    
    <pre>
    ...
    
    Now listening on: http://0.0.0.0:7071
    Application started. Press Ctrl+C to shut down.
    
    Http Functions:
    
            HttpExample: [GET,POST] http://localhost:7071/api/HttpExample
    ...
    
    </pre>
    
    > [!NOTE]  
    > Si HttpExample n’apparaît pas comme indiqué ci-dessus, cela signifie probablement que vous avez démarré l’hôte à partir d’un emplacement autre que le dossier racine du projet. Dans ce cas, utilisez **Ctrl**+**C** pour arrêter l’hôte, accédez au dossier racine du projet, puis réexécutez la commande précédente.

1. Copiez dans un navigateur l’URL de votre fonction `HttpExample` à partir de cette sortie, puis ajoutez la chaîne de requête `?name=<YOUR_NAME>` pour obtenir une URL complète semblable à `http://localhost:7071/api/HttpExample?name=Functions`. Le navigateur doit afficher un message similaire à `Hello Functions` :

    ![Résultat de la fonction exécutée localement dans le navigateur](./media/functions-create-first-azure-function-azure-cli/function-test-local-browser.png)
    
    Le terminal dans lequel vous avez démarré votre projet affiche également une sortie de journal quand vous effectuez des requêtes.

1. Quand vous avez terminé, utilisez **Ctrl**+**C**, puis choisissez `y` pour arrêter l’hôte Functions.

## <a name="deploy-the-function-project-to-azure"></a>Déployer le projet de fonction sur Azure

Une application de fonction et les ressources associées sont créées dans Azure lors du déploiement initial de votre projet Functions. Les paramètres des ressources Azure créées pour héberger votre application sont définis dans le [fichier pom.xml](#pomxml). Dans cet article, vous allez accepter les valeurs par défaut.

> [!TIP]
> Pour créer une application de fonction s’exécutant sur Linux au lieu de Windows, changez l’élément `runtime.os` dans le fichier pom.xml en remplaçant `windows` par `linux`. L’exécution de Linux dans un plan de consommation est prise en charge dans [ces régions](https://github.com/Azure/azure-functions-host/wiki/Linux-Consumption-Regions). Des applications s’exécutant sur Linux et des applications s’exécutant sur Windows ne peuvent pas se côtoyer dans le même groupe de ressources.

1. Avant d’effectuer le déploiement, connectez-vous à votre abonnement Azure en utilisant Azure CLI ou Azure PowerShell. 

    # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)
    ```azurecli
    az login
    ```

    La commande [az login](/cli/azure/reference-index#az-login) vous connecte à votre compte Azure.

    # <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell) 
    ```azurepowershell
    Connect-AzAccount
    ```

    L’applet de commande [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount) vous connecte à votre compte Azure.

    ---

1. Utilisez la commande suivante pour déployer votre projet vers une nouvelle application de fonction.

    ```console
    mvn azure-functions:deploy
    ```
    
    Cette commande crée les ressources suivantes dans Azure :
    
    + Groupe de ressources. Nommé en tant que _java-functions-group_.
    + Compte de stockage. Requis par les fonctions. Le nom est généré de façon aléatoire en fonction des exigences du nom de compte de stockage.
    + Plan d’hébergement. Hébergement serverless pour votre application de fonction dans la région _westus_. Le nom est _java-functions-app-service-plan_.
    + Application de fonction. Une application de fonction est l’unité de déploiement et d’exécution de vos fonctions. Le nom est généré de façon aléatoire, en fonction de votre _artifactId_ auquel est ajouté un numéro généré de façon aléatoire.
    
    Le déploiement empaquette les fichiers du projet et les déploie dans la nouvelle application de fonction à l’aide du [déploiement zip](functions-deployment-technologies.md#zip-deploy). Le code s’exécute à partir du package de déploiement dans Azure.

[!INCLUDE [functions-run-remote-azure-cli](../../includes/functions-run-remote-azure-cli.md)]

[!INCLUDE [functions-streaming-logs-cli-qs](../../includes/functions-streaming-logs-cli-qs.md)]

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous passez à l’[étape suivante](#next-steps), qui consiste à ajouter une liaison de sortie de file d’attente Stockage Azure, gardez toutes vos ressources en place, car vous allez tirer parti de ce que vous avez déjà fait.

Sinon, utilisez la commande suivante pour supprimer le groupe de ressources et toutes les ressources qu’il contient afin d’éviter les coûts supplémentaires.

 # <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

```azurecli
az group delete --name java-functions-group
```

# <a name="azure-powershell"></a>[Azure PowerShell](#tab/azure-powershell)

```azurepowershell
Remove-AzResourceGroup -Name java-functions-group
```

---

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Se connecter à une file d’attente Stockage Azure](functions-add-output-binding-storage-queue-cli.md?pivots=programming-language-java)
