---
title: Créer une fonction Azure Functions sur Linux avec une image personnalisée
description: Découvrez comment créer une exécution d’Azure Functions sur une image Linux personnalisée.
ms.date: 12/2/2020
ms.topic: tutorial
ms.custom: devx-track-csharp, mvc, devx-track-python, devx-track-azurepowershell, devx-track-azurecli
zone_pivot_groups: programming-languages-set-functions-full
ms.openlocfilehash: f270f74f97a9b9306d7b23dacec12c38f418dbd1
ms.sourcegitcommit: fec60094b829270387c104cc6c21257826fccc54
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96921821"
---
# <a name="create-a-function-on-linux-using-a-custom-container"></a>Créer une fonction sur Linux avec un conteneur personnalisé

Dans ce tutoriel, vous allez créer et déployer votre code sur Azure Functions en tant que conteneur Docker personnalisé en utilisant une image de base Linux. Vous utilisez une image personnalisée généralement quand vos fonctions nécessitent une version d’un langage spécifique, ou quand elles ont une dépendance ou une configuration spécifique qui n’est pas fournie par l’image intégrée.

::: zone pivot="programming-language-other"
Azure Functions prend en charge n’importe quel langage ou runtime à l’aide de [gestionnaires personnalisés](functions-custom-handlers.md). Pour certains langages, tels que le langage de programmation R utilisé dans ce tutoriel, vous devez installer le runtime ou des bibliothèques supplémentaires en tant que dépendances qui demandent d’utiliser un conteneur personnalisé.
::: zone-end

Le déploiement du code de votre fonction dans un conteneur Linux personnalisé nécessite un hébergement de [plan Premium](functions-premium-plan.md#features) ou de [Plan dédié (App Service)](functions-scale.md#app-service-plan). Effectuer ce tutoriel entraîne des coûts de quelques dollars US dans votre compte Azure, que vous pouvez réduire en [nettoyant les ressources](#clean-up-resources) quand vous avez terminé.

Vous pouvez également utiliser un conteneur Azure App Service par défaut, comme décrit dans [Créer votre première fonction hébergée sur Linux](./create-first-function-cli-csharp.md?pivots=programming-language-python). Les images de base prises en charge pour Azure Functions se trouvent dans le [référentiel d’images de base Azure Functions](https://hub.docker.com/_/microsoft-azure-functions-base).

Dans ce tutoriel, vous allez apprendre à :

::: zone pivot="programming-language-csharp,programming-language-javascript,programming-language-typescript,programming-language-powershell,programming-language-python,programming-language-java"
> [!div class="checklist"]
> * Créer une application de fonction et un Dockerfile avec Azure Functions Core Tools.
> * Créer une image personnalisée à l’aide de Docker
> * Publier une image personnalisée dans un registre de conteneurs
> * Créer des ressources de prise en charge dans Azure pour l’application de fonction
> * Déployer une application de fonction à partir de Docker Hub
> * Ajouter des paramètres d’application à l’application de fonction
> * Activer le déploiement continu.
> * Activer les connexions SSH au conteneur.
> * Ajouter une liaison de sortie de Stockage File d’attente. 
::: zone-end
::: zone pivot="programming-language-other"
> [!div class="checklist"]
> * Créer une application de fonction et un Dockerfile avec Azure Functions Core Tools.
> * Créer une image personnalisée à l’aide de Docker
> * Publier une image personnalisée dans un registre de conteneurs
> * Créer des ressources de prise en charge dans Azure pour l’application de fonction
> * Déployer une application de fonction à partir de Docker Hub
> * Ajouter des paramètres d’application à l’application de fonction
> * Activer le déploiement continu.
> * Activer les connexions SSH au conteneur.
::: zone-end

Vous pouvez suivre ce tutoriel sur n’importe quel ordinateur exécutant Windows, macOS ou Linux. 

[!INCLUDE [functions-requirements-cli](../../includes/functions-requirements-cli.md)]

<!---Requirements specific to Docker --->
+ [Docker](https://docs.docker.com/install/)  

+ Un [ID Docker](https://hub.docker.com/signup)

[!INCLUDE [functions-cli-verify-prereqs](../../includes/functions-cli-verify-prereqs.md)]

+ Exécutez `docker login` pour vous connecter à Docker. Cette commande échoue si Docker n’est pas en cours d’exécution, auquel cas démarrez Docker et réessayez la commande.

[!INCLUDE [functions-cli-create-venv](../../includes/functions-cli-create-venv.md)]

## <a name="create-and-test-the-local-functions-project"></a>Créer et tester un projet de fonction local

::: zone pivot="programming-language-csharp,programming-language-javascript,programming-language-typescript,programming-language-powershell,programming-language-python"  
Dans un terminal ou une invite de commandes, exécutez la commande suivante dans le langage de votre choix pour créer un projet d’application de fonction dans un dossier nommé `LocalFunctionsProject`.  
::: zone-end  
::: zone pivot="programming-language-csharp"  
```console
func init LocalFunctionsProject --worker-runtime dotnet --docker
```
::: zone-end  
::: zone pivot="programming-language-javascript"  
```console
func init LocalFunctionsProject --worker-runtime node --language javascript --docker
```
::: zone-end  
::: zone pivot="programming-language-powershell"  
```console
func init LocalFunctionsProject --worker-runtime powershell --docker
```
::: zone-end  
::: zone pivot="programming-language-python"  
```console
func init LocalFunctionsProject --worker-runtime python --docker
```
::: zone-end  
::: zone pivot="programming-language-typescript"  
```console
func init LocalFunctionsProject --worker-runtime node --language typescript --docker
```
::: zone-end
::: zone pivot="programming-language-java"  
Dans un dossier vide, exécutez la commande suivante pour générer le projet Functions à partir d’un [archétype Maven](https://maven.apache.org/guides/introduction/introduction-to-archetypes.html).

# <a name="bash"></a>[Bash](#tab/bash)
```bash
mvn archetype:generate -DarchetypeGroupId=com.microsoft.azure -DarchetypeArtifactId=azure-functions-archetype -DjavaVersion=8 -Ddocker
```
# <a name="powershell"></a>[PowerShell](#tab/powershell)
```powershell
mvn archetype:generate "-DarchetypeGroupId=com.microsoft.azure" "-DarchetypeArtifactId=azure-functions-archetype" "-DjavaVersion=8" "-Ddocker"
```
# <a name="cmd"></a>[Cmd](#tab/cmd)
```cmd
mvn archetype:generate "-DarchetypeGroupId=com.microsoft.azure" "-DarchetypeArtifactId=azure-functions-archetype" "-DjavaVersion=8" "-Ddocker"
```
---

Le paramètre `-DjavaVersion` dit au runtime Functions quelle version de Java utiliser. Utilisez `-DjavaVersion=11` si vous voulez que vos fonctions s’exécutent sur Java 11. Si vous ne spécifiez pas `-DjavaVersion`, par défaut Maven utilise Java 8. Pour plus d’informations, consultez [Versions de Java](functions-reference-java.md#java-versions).

> [!IMPORTANT]
> La variable d’environnement `JAVA_HOME` doit être définie sur l’emplacement d’installation de la version appropriée du JDK pour suivre cet article.

Maven vous invite à entrer les valeurs nécessaires pour terminer la génération du projet lors du déploiement.   
Fournissez les valeurs suivantes à l’invite :

| Prompt | Valeur | Description |
| ------ | ----- | ----------- |
| **groupId** | `com.fabrikam` | Valeur qui identifie de façon unique votre projet parmi tous les projets, avec respect des [règles de nommage de package](https://docs.oracle.com/javase/specs/jls/se6/html/packages.html#7.7) pour Java. |
| **artifactId** | `fabrikam-functions` | Valeur qui correspond au nom du fichier jar, sans numéro de version. |
| **version** | `1.0-SNAPSHOT` | Choisissez la valeur par défaut. |
| **package** | `com.fabrikam.functions` | Valeur qui correspond au package Java pour le code de fonction généré. Utilisez la valeur par défaut. |

Tapez `Y` ou appuyez sur Entrée pour confirmer.

Maven crée les fichiers projet dans un nouveau dossier avec le nom d’_artifactId_, qui est `fabrikam-functions` dans cet exemple. 
::: zone-end

::: zone pivot="programming-language-other"  
```console
func init LocalFunctionsProject --worker-runtime custom --docker
```
::: zone-end

L’option `--docker` génère un `Dockerfile` pour le projet, qui définit un conteneur personnalisé approprié pour une utilisation avec Azure Functions et le runtime sélectionné.

Accédez au dossier du projet :
::: zone pivot="programming-language-csharp,programming-language-javascript,programming-language-typescript,programming-language-powershell,programming-language-python,programming-language-other"  
```console
cd LocalFunctionsProject
```
::: zone-end  
::: zone pivot="programming-language-java"  
```console
cd fabrikam-functions
```
::: zone-end  
::: zone pivot="programming-language-csharp,programming-language-javascript,programming-language-typescript,programming-language-powershell,programming-language-python" 
Ajoutez une fonction à votre projet à l’aide de la commande suivante, où l’argument `--name` est le nom unique de votre fonction, et l’argument `--template` spécifie le déclencheur de la fonction. `func new` crée un sous-dossier correspondant au nom de la fonction qui contient le fichier de code approprié au langage choisi pour le projet, et un fichier config nommé *function.json*.

```console
func new --name HttpExample --template "HTTP trigger"
```
::: zone-end

::: zone pivot="programming-language-other" 
Ajoutez une fonction à votre projet à l’aide de la commande suivante, où l’argument `--name` est le nom unique de votre fonction, et l’argument `--template` spécifie le déclencheur de la fonction. `func new` crée un sous-dossier correspondant au nom de la fonction qui contient un fichier config nommé *function.json*.

```console
func new --name HttpExample --template "HTTP trigger"
```

Au moyen d’un éditeur de texte, dans le dossier du projet, créez un fichier nommé *handler.R*. Ajoutez le code suivant en tant que contenu.

```r
library(httpuv)

PORTEnv <- Sys.getenv("FUNCTIONS_CUSTOMHANDLER_PORT")
PORT <- strtoi(PORTEnv , base = 0L)

http_not_found <- list(
  status=404,
  body='404 Not Found'
)

http_method_not_allowed <- list(
  status=405,
  body='405 Method Not Allowed'
)

hello_handler <- list(
  GET = function (request) {
    list(body=paste(
      "Hello,",
      if(substr(request$QUERY_STRING,1,6)=="?name=") 
        substr(request$QUERY_STRING,7,40) else "World",
      sep=" "))
  }
)

routes <- list(
  '/api/HttpExample' = hello_handler
)

router <- function (routes, request) {
  if (!request$PATH_INFO %in% names(routes)) {
    return(http_not_found)
  }
  path_handler <- routes[[request$PATH_INFO]]

  if (!request$REQUEST_METHOD %in% names(path_handler)) {
    return(http_method_not_allowed)
  }
  method_handler <- path_handler[[request$REQUEST_METHOD]]

  return(method_handler(request))
}

app <- list(
  call = function (request) {
    response <- router(routes, request)
    if (!'status' %in% names(response)) {
      response$status <- 200
    }
    if (!'headers' %in% names(response)) {
      response$headers <- list()
    }
    if (!'Content-Type' %in% names(response$headers)) {
      response$headers[['Content-Type']] <- 'text/plain'
    }

    return(response)
  }
)

cat(paste0("Server listening on :", PORT, "...\n"))
runServer("0.0.0.0", PORT, app)
```

Dans *host.json*, modifiez la section `customHandler` pour configurer la commande de démarrage du gestionnaire personnalisé.

```json
"customHandler": {
  "description": {
      "defaultExecutablePath": "Rscript",
      "arguments": [
      "handler.R"
    ]
  },
  "enableForwardingHttpRequest": true
}
```
::: zone-end

Pour tester la fonction localement, démarrez l’hôte du runtime Azure Functions local à la racine du dossier du projet : 
::: zone pivot="programming-language-csharp"  
```console
func start --build  
```
::: zone-end  
::: zone pivot="programming-language-javascript,programming-language-powershell,programming-language-python"   
```console
func start  
```
::: zone-end  
::: zone pivot="programming-language-typescript"  
```console
npm install
npm start
```
::: zone-end  
::: zone pivot="programming-language-java"  
```console
mvn clean package  
mvn azure-functions:run
```
::: zone-end
::: zone pivot="programming-language-other"
```console
R -e "install.packages('httpuv', repos='http://cran.rstudio.com/')"
func start
```
::: zone-end 

Une fois que le point de terminaison `HttpExample` apparaît dans la sortie, accédez à `http://localhost:7071/api/HttpExample?name=Functions`. Le navigateur doit afficher un message « hello » qui renvoie `Functions` par écho, c’est-à-dire la valeur fournie au paramètre de requête `name`.

Utilisez **Ctrl**-**C** pour arrêter l’hôte.

## <a name="build-the-container-image-and-test-locally"></a>Générer l’image conteneur et tester localement

::: zone pivot="programming-language-csharp,programming-language-javascript,programming-language-powershell,programming-language-python,programming-language-java,programming-language-typescript"
(Facultatif) Examinez le *Dockerfile* à la racine du dossier du projet. Le Dockerfile décrit l’environnement requis pour exécuter l’application de fonction sur Linux.  La liste complète des images de base prises en charge pour Azure Functions se trouve sur la [pages des images de base Azure Functions](https://hub.docker.com/_/microsoft-azure-functions-base).
::: zone-end

::: zone pivot="programming-language-other"
Examinez le *Dockerfile* à la racine du dossier du projet. Le Dockerfile décrit l’environnement requis pour exécuter l’application de fonction sur Linux. Les applications de gestionnaire personnalisé utilisent l’image `mcr.microsoft.com/azure-functions/dotnet:3.0-appservice` comme base.

Modifiez le *Dockerfile* pour installer R. Remplacez le contenu de *Dockerfile* par le code suivant.

```dockerfile
FROM mcr.microsoft.com/azure-functions/dotnet:3.0-appservice 
ENV AzureWebJobsScriptRoot=/home/site/wwwroot \
    AzureFunctionsJobHost__Logging__Console__IsEnabled=true

RUN apt update && \
    apt install -y r-base && \
    R -e "install.packages('httpuv', repos='http://cran.rstudio.com/')"

COPY . /home/site/wwwroot
```
::: zone-end

À la racine du dossier du projet, exécutez la commande [docker build](https://docs.docker.com/engine/reference/commandline/build/), puis indiquez un nom (`azurefunctionsimage`) et une balise (`v1.0.0`). Remplacez `<DOCKER_ID>` par votre ID de compte Docker Hub. Cette commande génère l’image Docker pour le conteneur.

```console
docker build --tag <DOCKER_ID>/azurefunctionsimage:v1.0.0 .
```

Une fois la commande terminée, vous pouvez exécuter le nouveau conteneur localement.
    
Pour tester la génération, exécutez l’image dans un conteneur local avec la commande [Docker run](https://docs.docker.com/engine/reference/commandline/run/), en remplaçant à nouveau `<DOCKER_ID` par votre ID Docker et en ajoutant l’argument pour les ports, `-p 8080:80` :

```console
docker run -p 8080:80 -it <docker_id>/azurefunctionsimage:v1.0.0
```

::: zone pivot="programming-language-csharp,programming-language-javascript,programming-language-typescript,programming-language-powershell,programming-language-python,programming-language-other"  
Une fois que l’image s’exécute dans un conteneur local, ouvrez un navigateur sur `http://localhost:8080`, qui doit afficher l’image de l’espace réservé montrée ci-dessous. L’image apparaît à ce stade, car votre fonction est exécutée dans le conteneur local, comme ce serait le cas dans Azure, ce qui signifie qu’elle est protégée par une clé d’accès telle que définie dans *function.json* avec la propriété `"authLevel": "function"`. Le conteneur n’a cependant pas encore été publié sur une application de fonction : la clé n’est donc pas encore disponible. Si vous voulez effectuer un test avec le conteneur local, arrêtez Docker, changez la propriété d’autorisation en `"authLevel": "anonymous"`, regénérez l’image, puis redémarrez Docker. Ensuite, réinitialisez `"authLevel": "function"` dans *function.json*. Pour plus d’informations, consultez [Clés d’autorisation](functions-bindings-http-webhook-trigger.md#authorization-keys).

![Image d’espace réservé indiquant que le conteneur s’exécute localement](./media/functions-create-function-linux-custom-image/run-image-local-success.png)

::: zone-end
::: zone pivot="programming-language-java"  
Une fois que l’image s’exécute dans un conteneur local, accédez à `http://localhost:8080/api/HttpExample?name=Functions`, qui doit afficher le même message « hello » que précédemment. Étant donné que l’archétype Maven génère une fonction HTTP déclenchée qui utilise une autorisation anonyme, vous pouvez toujours appeler la fonction même si elle est déjà exécutée dans le conteneur. 
::: zone-end  

Après avoir vérifié l’application de fonction dans le conteneur, arrêtez Docker avec **Ctrl**+**C**.

## <a name="push-the-image-to-docker-hub"></a>Envoyer (push) l’image sur Docker Hub

Docker Hub est un registre de conteneurs qui héberge des images, et qui fournit des services d’image et de conteneur. Pour partager votre image, ce qui inclut un déploiement sur Azure, vous devez l’envoyer dans un registre.

1. Si vous ne vous êtes pas déjà connecté à Docker, utilisez la commande [docker login](https://docs.docker.com/engine/reference/commandline/login/), en remplaçant `<docker_id>` par votre ID Docker. Cette commande vous invite à entrer votre nom d’utilisateur et votre mot de passe. Un message « Connexion réussie » confirme que vous êtes connecté.

    ```console
    docker login
    ```
    
1. Une fois connecté, envoyez l’image à Docker Hub en utilisant la commande [docker push](https://docs.docker.com/engine/reference/commandline/push/), en remplaçant à nouveau `<docker_id>` par votre ID Docker.

    ```console
    docker push <docker_id>/azurefunctionsimage:v1.0.0
    ```

1. Selon la rapidité de votre réseau, le premier initial de l’image peut prendre quelques minutes (l’envoi des modifications suivantes est beaucoup plus rapide). Pendant que vous attendez, vous pouvez passer à la section suivante et créer des ressources Azure dans un autre terminal.

## <a name="create-supporting-azure-resources-for-your-function"></a>Créer des ressources Azure de support pour votre fonction

Pour déployer votre code de fonction sur Azure, vous devez créer trois ressources :

- Un groupe de ressources, qui est un conteneur logique pour les ressources associées.
- Un compte de stockage Azure, qui conserve l’état et d’autres informations spécifiques à vos projets.
- Une application Azure Functions, qui fournit l’environnement d’exécution de votre code de fonction. Une application de fonction est mappée à votre projet de fonction local. Elle vous permet de regrouper les fonctions en tant qu’unité logique pour faciliter la gestion, le déploiement et le partage des ressources.

Vous utilisez les commandes Azure CLI pour créer ces éléments. Chaque commande fournit une sortie JSON une fois l’opération terminée.

1. Connectez-vous à Azure avec la commande [az login](/cli/azure/reference-index#az-login) :

    ```azurecli
    az login
    ```
    
1. Créez un groupe de ressources avec la commande [az group create](/cli/azure/group#az-group-create). L’exemple suivant crée un groupe de ressources nommé `AzureFunctionsContainers-rg` dans la région `westeurope`. (Vous créez généralement votre groupe de ressources et vos ressources dans une région près de chez vous, en utilisant une région disponible à partir de la commande `az account list-locations`.)

    ```azurecli
    az group create --name AzureFunctionsContainers-rg --location westeurope
    ```
    
    > [!NOTE]
    > Vous ne pouvez pas héberger des applications Linux et Windows dans le même groupe de ressources. Si vous disposez d’un groupe de ressources existant nommé `AzureFunctionsContainers-rg` avec une application de fonction Windows ou une application web, vous devez utiliser un autre groupe de ressources.
    
1. Créez un compte de stockage universel dans votre groupe de ressources et votre région à l’aide de la commande [az storage account create](/cli/azure/storage/account#az-storage-account-create). Dans l’exemple suivant, remplacez `<storage_name>` par un nom globalement unique qui vous convient. Les noms doivent contenir entre 3 et 24 caractères, et comporter uniquement des lettres minuscules. `Standard_LRS` spécifie un compte universel standard.

    ```azurecli
    az storage account create --name <storage_name> --location westeurope --resource-group AzureFunctionsContainers-rg --sku Standard_LRS
    ```
    
    Le compte de stockage génère un coût de quelques cents USD seulement pour ce tutoriel.
    
1. Utilisez la commande pour créer un plan Premium pour Azure Functions nommé `myPremiumPlan` dans le niveau tarifaire **Elastic Premium 1** (`--sku EP1`), dans la région Europe Ouest (`-location westeurope` ou utilisez une région appropriée proche de vous) et dans un conteneur Linux (`--is-linux`).

    ```azurecli
    az functionapp plan create --resource-group AzureFunctionsContainers-rg --name myPremiumPlan --location westeurope --number-of-workers 1 --sku EP1 --is-linux
    ```   

    Nous utilisons ici le plan Premium, qui peut être mis à l’échelle en fonction des besoins. Pour en savoir plus sur l’hébergement, consultez [Comparaison des plans d’hébergement Azure Functions](functions-scale.md). Pour calculer les coûts, consultez la [page Tarification d’Azure Functions](https://azure.microsoft.com/pricing/details/functions/).

    La commande provisionne également une instance Azure Application Insights associée dans le même groupe de ressources, avec laquelle vous pouvez superviser votre application de fonction et visualiser les journaux. Pour plus d’informations, consultez [Surveiller l’exécution des fonctions Azure](functions-monitoring.md). L’instance n’entraîne aucun coût tant que vous ne l’activez pas.

## <a name="create-and-configure-a-function-app-on-azure-with-the-image"></a>Créer et configurer une application de fonction sur Azure avec l’image

Une application de fonction sur Azure gère l’exécution de vos fonctions dans votre plan d’hébergement. Dans cette section, vous utilisez les ressources Azure de la section précédente pour créer une application de fonction à partir d’une image sur Docker Hub et pour la configurer avec une chaîne de connexion à Stockage Azure.

1. Créez l’application de fonction avec la commande [az functionapp create](/cli/azure/functionapp#az-functionapp-create). Dans l’exemple suivant, remplacez `<storage_name>` par le nom que vous avez utilisé dans la section précédente pour le compte de stockage. Remplacez aussi `<app_name>` par un nom globalement unique qui vous convient et `<docker_id>` par votre ID Docker.

    ::: zone pivot="programming-language-csharp,programming-language-javascript,programming-language-typescript,programming-language-powershell,programming-language-python,programming-language-java"
    ```azurecli
    az functionapp create --name <app_name> --storage-account <storage_name> --resource-group AzureFunctionsContainers-rg --plan myPremiumPlan --runtime <functions runtime stack> --deployment-container-image-name <docker_id>/azurefunctionsimage:v1.0.0
    ```
    ::: zone-end
    ::: zone pivot="programming-language-other"
    ```azurecli
    az functionapp create --name <app_name> --storage-account <storage_name> --resource-group AzureFunctionsContainers-rg --plan myPremiumPlan --runtime custom --deployment-container-image-name <docker_id>/azurefunctionsimage:v1.0.0
    ```
    ::: zone-end
    
    Le paramètre *deployment-container-image-name* spécifie l’image à utiliser pour l’application de fonction. Vous pouvez utiliser la commande [az functionapp config container show](/cli/azure/functionapp/config/container#az-functionapp-config-container-show) pour voir des informations sur l’image utilisée pour le déploiement. Vous pouvez aussi utiliser la commande [az functionapp config container set](/cli/azure/functionapp/config/container#az-functionapp-config-container-set) pour déployer à partir d’une autre image.

1. Affichez la chaîne de connexion pour le compte de stockage que vous avez créé avec la commande [az storage account show-connection-string](/cli/azure/storage/account). Remplacez `<storage-name>` par le nom du compte de stockage créé ci-dessus :

    ```azurecli
    az storage account show-connection-string --resource-group AzureFunctionsContainers-rg --name <storage_name> --query connectionString --output tsv
    ```
    
1. Ajoutez ce paramètre à l’application de fonction à l’aide de la commande [az functionapp config appsettings set](/cli/azure/functionapp/config/appsettings#az-functionapp-config-appsettings-set). Dans la commande suivante, remplacez `<app_name>` par le nom de votre application de fonction et `<connection_string>` par la chaîne de connexion récupérée à l’étape précédente (une longue chaîne codée qui commence par « DefaultEndpointProtocol= ») :
 
    ```azurecli
    az functionapp config appsettings set --name <app_name> --resource-group AzureFunctionsContainers-rg --settings AzureWebJobsStorage=<connection_string>
    ```

    > [!TIP]
    > Dans Bash, vous pouvez utiliser une variable de shell pour capturer la chaîne de connexion, au lieu d’utiliser le Presse-papiers. Pour commencer, utilisez la commande suivante pour créer une variable avec la chaîne de connexion :
    > 
    > ```bash
    > storageConnectionString=$(az storage account show-connection-string --resource-group AzureFunctionsContainers-rg --name <storage_name> --query connectionString --output tsv)
    > ```
    > 
    > Ensuite, référencez la variable dans la deuxième commande :
    > 
    > ```azurecli
    > az functionapp config appsettings set --name <app_name> --resource-group AzureFunctionsContainers-rg --settings AzureWebJobsStorage=$storageConnectionString
    > ```

1. La fonction peut maintenant utiliser cette chaîne de connexion pour accéder au compte de stockage.

> [!NOTE]    
> Si vous publiez votre image personnalisée sur un compte de conteneur privé, vous devez utiliser à la place des variables d’environnement dans le Dockerfile pour la chaîne de connexion. Pour plus d’informations, consultez l’[instruction ENV](https://docs.docker.com/engine/reference/builder/#env). Vous devez également définir les variables `DOCKER_REGISTRY_SERVER_USERNAME` et `DOCKER_REGISTRY_SERVER_PASSWORD`. Pour utiliser les valeurs, vous devez regénérer l’image, envoyer l’image vers le registre, puis redémarrer l’application de fonction sur Azure.

## <a name="verify-your-functions-on-azure"></a>Vérifier vos fonctions sur Azure

Avec l’image déployée sur l’application de fonction sur Azure, vous pouvez désormais appeler la fonction via des requêtes HTTP. Comme la définition de *function.json* inclut la propriété `"authLevel": "function"`, vous devez d’abord obtenir la clé d’accès (également appelée « clé de fonction ») et l’inclure en tant que paramètre d’URL dans toutes les requêtes adressées au point de terminaison.

1. Récupérez l’URL de la fonction avec la clé (de fonction) d’accès en utilisant le portail Azure ou Azure CLI avec la commande `az rest`.

    # <a name="portal"></a>[Portail](#tab/portal)

    1. Connectez-vous au portail Azure, puis recherchez et sélectionnez **Application de fonction**.

    1. Sélectionnez la fonction que vous souhaitez vérifier.

    1. Dans le volet de navigation gauche, sélectionnez **Fonctions**, puis sélectionnez la fonction que vous souhaitez vérifier.

        ![Sélectionnez votre fonction dans le portail Azure](./media/functions-create-function-linux-custom-image/functions-portal-select-function.png)   

    
    1. Sélectionnez **Obtenir l’URL de la fonction**.

        ![Obtenir l'URL de fonction à partir du portail Azure](./media/functions-create-function-linux-custom-image/functions-portal-get-function-url.png)   

    
    1. Dans la fenêtre contextuelle, sélectionnez **Par défaut (clé de fonction)** , puis copiez l’URL dans le presse-papiers. La clé est la chaîne de caractères qui suit `?code=`.

        ![Choisir la clé d’accès par défaut de la fonction](./media/functions-create-function-linux-custom-image/functions-portal-copy-url.png)   


    > [!NOTE]  
    > Dans la mesure où votre application de fonction est déployée en tant que conteneur, vous ne pouvez pas apporter de modifications à votre code de fonction dans le portail. Vous devez à la place mettre à jour le projet dans l’image locale, envoyer à nouveau l’image vers le registre, puis redéployer sur Azure. Vous pouvez configurer le déploiement continu dans une section ultérieure.
    
    # <a name="azure-cli"></a>[Azure CLI](#tab/azurecli)

    1. Construisez une chaîne d’URL au format suivant, en remplaçant `<subscription_id>`, `<resource_group>`et `<app_name>` par votre ID d’abonnement Azure, par le groupe de ressources de votre application de fonction et par le nom de votre application de fonction, respectivement :

        ```
        "/subscriptions/<subscription_id>/resourceGroups/<resource_group>/providers/Microsoft.Web/sites/<app_name>/host/default/listKeys?api-version=2018-11-01"
        ```

        Par exemple, l’URL peut se présenter comme l’adresse suivante :

        ```
        "/subscriptions/1234aaf4-1234-abcd-a79a-245ed34eabcd/resourceGroups/AzureFunctionsContainers-rg/providers/Microsoft.Web/sites/msdocsfunctionscontainer/host/default/listKeys?api-version=2018-11-01"
        ```

        > [!TIP]
        > Par souci pratique, vous pouvez à la place affecter l’URL à une variable d’environnement et l’utiliser dans la commande `az rest`.
    
    1. Exécutez la commande `az rest` suivante (disponible dans Azure CLI version 2.0.77 et ultérieure), en remplaçant `<uri>` par la chaîne d’URI de la dernière étape, en y incluant les guillemets :

        ```azurecli
        az rest --method post --uri <uri> --query functionKeys.default --output tsv
        ```

    1. La sortie de la commande est la clé de la fonction. L’URL complète de la fonction est alors `https://<app_name>.azurewebsites.net/api/<function_name>?code=<key>` (remplacez `<app_name>`, `<function_name>` et `<key>` par vos valeurs spécifiques).
    
        > [!NOTE]
        > La clé récupérée ici est la clé d’*hôte* qui marche pour toutes les fonctions de l’application de fonction ; la méthode montrée pour le portail récupère la clé pour cette seule fonction.

    ---

1. Collez l’URL de la fonction dans la barre d’adresse de votre navigateur, en ajoutant le paramètre `&name=Azure` à la fin de cette URL. Un texte similaire à « Hello, Azure » doit apparaître dans le navigateur.

    ![Réponse de la fonction dans le navigateur.](./media/functions-create-function-linux-custom-image/function-app-browser-testing.png)

1. Pour tester l’autorisation, supprimez le paramètre `code=` de l’URL et vérifiez que vous ne recevez pas de réponse de la fonction.


## <a name="enable-continuous-deployment-to-azure"></a>Activer le déploiement continu sur Azure

Vous pouvez activer Azure Functions pour mettre à jour automatiquement votre déploiement d’une image chaque fois que vous mettez à jour l’image dans le registre.

1. Activez le déploiement continu avec la commande [az functionapp deployment container config](/cli/azure/functionapp/deployment/container#az-functionapp-deployment-container-config), en remplaçant `<app_name>` par le nom de votre application de fonction :

    ```azurecli
    az functionapp deployment container config --enable-cd --query CI_CD_URL --output tsv --name <app_name> --resource-group AzureFunctionsContainers-rg
    ```
    
    Cette commande active le déploiement continu et retourne l’URL du webhook de déploiement. (Vous pouvez récupérer cette URL ultérieurement avec la commande [az functionapp deployment container show-cd-url](/cli/azure/functionapp/deployment/container#az-functionapp-deployment-container-show-cd-url).)

1. Copiez l’URL du webhook de déploiement dans le Presse-papiers.

1. Ouvrez [Docker Hub](https://hub.docker.com/), connectez-vous, puis sélectionnez **Référentiels** dans la barre de navigation. Recherchez et sélectionnez l’image, sélectionnez l’onglet **Webhooks**, spécifiez un **Nom de webhook**, collez votre URL dans **URL du webhook**, puis sélectionnez **Créer** :

    ![Ajoutez le webhook à votre référentiel DockerHub](./media/functions-create-function-linux-custom-image/dockerhub-set-continuous-webhook.png)  

1. Avec le webhook défini, Azure Functions redéploie votre image chaque fois que vous la mettez à jour dans Docker Hub.

## <a name="enable-ssh-connections"></a>Activer les connexions SSH

SSH permet d’établir une communication sécurisée entre un conteneur et un client. Avec le protocole SSH activé, vous pouvez vous connecter à votre conteneur en utilisant App Service Advanced Tools (Kudu). Pour faciliter la connexion à votre conteneur via SSH, Azure Functions fournit une image de base pour laquelle SSH est déjà activé. Vous devez seulement modifier votre Dockerfile, puis regénérer et redéployer l’image. Vous pouvez ensuite vous connecter au conteneur via les outils avancés (Kudu).

1. Dans votre Dockerfile, ajoutez la chaîne `-appservice` à l’image de base dans votre instruction `FROM` :

    ::: zone pivot="programming-language-csharp"
    ```Dockerfile
    FROM mcr.microsoft.com/azure-functions/dotnet:3.0-appservice
    ```
    ::: zone-end

    ::: zone pivot="programming-language-javascript"
    ```Dockerfile
    FROM mcr.microsoft.com/azure-functions/node:2.0-appservice
    ```
    ::: zone-end

    ::: zone pivot="programming-language-powershell"
    ```Dockerfile
    FROM mcr.microsoft.com/azure-functions/powershell:2.0-appservice
    ```
    ::: zone-end

    ::: zone pivot="programming-language-python"
    ```Dockerfile
    FROM mcr.microsoft.com/azure-functions/python:2.0-python3.7-appservice
    ```
    ::: zone-end

    ::: zone pivot="programming-language-typescript"
    ```Dockerfile
    FROM mcr.microsoft.com/azure-functions/node:2.0-appservice
    ```
    ::: zone-end
    
1. Regénérez l’image avec la commande `docker build`, en remplaçant `<docker_id>` par votre ID Docker :

    ```console
    docker build --tag <docker_id>/azurefunctionsimage:v1.0.0 .
    ```
    
1. Envoyez l’image mise à jour sur Docker Hub, ce qui doit prendre beaucoup moins de temps que lors du premier envoi : seuls les segments mis à jour de l’image doivent être chargés.

    ```console
    docker push <docker_id>/azurefunctionsimage:v1.0.0
    ```
    
1. Azure Functions redéploie automatiquement l’image sur votre application de fonctions ; le processus prend moins d’une minute.

1. Dans un navigateur, ouvrez `https://<app_name>.scm.azurewebsites.net/` (remplacez `<app_name>` par votre nom unique). Cette URL est le point de terminaison des outils avancés (Kudu) pour votre conteneur d’application de fonction.

1. Connectez-vous à votre compte Azure, puis sélectionnez **SSH** pour établir une connexion avec votre conteneur. La connexion peut prendre quelques instants si Azure est toujours en train de mettre à jour l’image conteneur.

1. Une fois la connexion établie avec votre conteneur, exécutez la commande `top` pour voir les processus en cours d’exécution. 

    ![Commande top Linux s’exécutant dans une session SSH](media/functions-create-function-linux-custom-image/linux-custom-kudu-ssh-top.png)

::: zone pivot="programming-language-csharp,programming-language-javascript,programming-language-typescript,programming-language-powershell,programming-language-python,programming-language-java"

## <a name="write-to-an-azure-storage-queue"></a>Écrire dans une file d’attente Stockage Azure

Azure Functions vous permet de connecter vos fonctions à d’autres services et ressources Azure sans devoir écrire votre propre code d’intégration. Ces *liaisons*, qui représentent l’entrée et la sortie, sont déclarées dans la définition de la fonction. Les données issues des liaisons sont fournies à la fonction en tant que paramètres. Un *déclencheur* est un type spécial de liaison d’entrée. Si une fonction ne peut avoir qu’un seul déclencheur, elle peut avoir plusieurs liaisons d’entrée et de sortie. Pour en savoir plus, consultez [Concepts des déclencheurs et liaisons Azure Functions](functions-triggers-bindings.md).

Cette section vous montre comment intégrer votre fonction à une file d’attente de stockage Azure. La liaison de sortie que vous ajoutez à cette fonction écrit des données d’une requête HTTP dans un message en file d’attente.

[!INCLUDE [functions-cli-get-storage-connection](../../includes/functions-cli-get-storage-connection.md)]
::: zone-end

[!INCLUDE [functions-register-storage-binding-extension-csharp](../../includes/functions-register-storage-binding-extension-csharp.md)]

[!INCLUDE [functions-add-output-binding-cli](../../includes/functions-add-output-binding-cli.md)]

::: zone pivot="programming-language-csharp"  
[!INCLUDE [functions-add-storage-binding-csharp-library](../../includes/functions-add-storage-binding-csharp-library.md)]  
::: zone-end  
::: zone pivot="programming-language-java" 
[!INCLUDE [functions-add-output-binding-java-cli](../../includes/functions-add-output-binding-java-cli.md)]
::: zone-end  

::: zone pivot="programming-language-csharp,programming-language-javascript,programming-language-typescript,programming-language-powershell,programming-language-python,programming-language-java"

## <a name="add-code-to-use-the-output-binding"></a>Ajouter du code pour utiliser la liaison de sortie

Avec la liaison de file d’attente définie, vous pouvez à présent mettre à jour votre fonction pour qu’elle reçoive le paramètre de sortie `msg` et écrive des messages dans la file d’attente.
::: zone-end

::: zone pivot="programming-language-python"     
[!INCLUDE [functions-add-output-binding-python](../../includes/functions-add-output-binding-python.md)]
::: zone-end  

::: zone pivot="programming-language-javascript"  
[!INCLUDE [functions-add-output-binding-js](../../includes/functions-add-output-binding-js.md)]
::: zone-end  

::: zone pivot="programming-language-typescript"  
[!INCLUDE [functions-add-output-binding-ts](../../includes/functions-add-output-binding-ts.md)]
::: zone-end  

::: zone pivot="programming-language-powershell"  
[!INCLUDE [functions-add-output-binding-powershell](../../includes/functions-add-output-binding-powershell.md)]  
::: zone-end

::: zone pivot="programming-language-csharp"  
[!INCLUDE [functions-add-storage-binding-csharp-library-code](../../includes/functions-add-storage-binding-csharp-library-code.md)]
::: zone-end 

::: zone pivot="programming-language-java"
[!INCLUDE [functions-add-output-binding-java-code](../../includes/functions-add-output-binding-java-code.md)]

[!INCLUDE [functions-add-output-binding-java-test-cli](../../includes/functions-add-output-binding-java-test-cli.md)]
::: zone-end

::: zone pivot="programming-language-csharp,programming-language-javascript,programming-language-typescript,programming-language-powershell,programming-language-python,programming-language-java"
### <a name="update-the-image-in-the-registry"></a>Mettre à jour l’image dans le registre

1. Dans le dossier racine, réexécutez la commande `docker build` et mettez cette fois à jour la version dans l’étiquette vers `v1.0.1`. Comme précédemment, remplacez `<docker_id>` par votre ID de compte Docker Hub :

    ```console
    docker build --tag <docker_id>/azurefunctionsimage:v1.0.1 .
    ```
    
1. Envoyez l’image mise à jour dans le référentiel avec `docker push` :

    ```console
    docker push <docker_id>/azurefunctionsimage:v1.0.1
    ```

1. Comme vous avez configuré la livraison continue, la mise à jour de l’image dans le registre met à jour automatiquement votre application de fonction dans Azure.

## <a name="view-the-message-in-the-azure-storage-queue"></a>Afficher le message dans la file d’attente Stockage Azure

Dans un navigateur, utilisez la même URL qu’auparavant pour appeler votre fonction. Le navigateur doit afficher la même réponse qu’auparavant, car vous n’avez pas modifié cette partie du code de la fonction. Le code ajouté a cependant écrit un message en utilisant le paramètre d’URL `name` dans la file d’attente de stockage `outqueue`.

[!INCLUDE [functions-add-output-binding-view-queue-cli](../../includes/functions-add-output-binding-view-queue-cli.md)]

::: zone-end

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous voulez continuer à utiliser Azure Functions en utilisant les ressources que vous avez créées dans ce tutoriel, vous pouvez conserver toutes ces ressources en place. Comme vous avez créé un plan Premium pour Azure Functions, un ou deux dollars US vous seront facturés quotidiennement.

Pour éviter ces frais récurrents, supprimez le groupe de ressources `AzureFunctionsContainer-rg` pour supprimer toutes les ressources de ce groupe : 

```azurecli
az group delete --name AzureFunctionsContainer-rg
```

## <a name="next-steps"></a>Étapes suivantes

+ [Supervision des fonctions](functions-monitoring.md)
+ [Mise à l’échelle et options d’hébergement](functions-scale.md)
+ [Hébergement serverless basé sur Kubernetes](functions-kubernetes-keda.md)
