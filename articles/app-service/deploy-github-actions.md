---
title: Configurer CI/CD avec GitHub Actions
description: Découvrez comment déployer votre code dans Azure App Service à partir d’un pipeline CI/CD avec GitHub Actions. Personnalisez les tâches de génération et exécutez des déploiements complexes.
ms.devlang: na
ms.topic: article
ms.date: 09/14/2020
ms.author: jafreebe
ms.reviewer: ushan
ms.custom: devx-track-python, github-actions-azure, devx-track-azurecli
ms.openlocfilehash: 0c10cc683d8c8c2496ca8fdbd00f0e5065e2db35
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97604921"
---
# <a name="deploy-to-app-service-using-github-actions"></a>Déployer sur App Service à l’aide de GitHub Actions

Prenez en main [GitHub Actions](https://docs.github.com/en/free-pro-team@latest/actions/learn-github-actions) pour automatiser votre workflow et le déployer sur [Azure App Service](overview.md) à partir de GitHub. 

## <a name="prerequisites"></a>Prérequis 

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Un compte GitHub. Si vous n’en avez pas, inscrivez-vous [gratuitement](https://github.com/join).  
- Une application Azure App Service fonctionnelle. 
    - .NET : [Créer une application web ASP.NET Core dans Azure](quickstart-dotnetcore.md)
    - ASP.NET : [Créer une application web ASP.NET Framework dans Azure](quickstart-dotnet-framework.md)
    - JavaScript : [Créer une application web Node.js dans Azure App Service](quickstart-nodejs.md)  
    - Java : [Créer une application Java dans Azure App Service](quickstart-java.md)
    - Python : [Créer une application Python dans Azure App Service](quickstart-python.md)

## <a name="workflow-file-overview"></a>Vue d’ensemble du fichier de workflow

Un workflow est défini par un fichier YAML (.yml) situé dans le chemin `/.github/workflows/` de votre dépôt. Cette définition contient les étapes et les paramètres qui composent le workflow.

Le fichier comporte trois sections :

|Section  |Tâches  |
|---------|---------|
|**Authentification** | 1. Définissez un principal de service ou un profil de publication. <br /> 2. Créez un secret GitHub. |
|**Créer** | 1. Configurez l’environnement. <br /> 2. Créez l’application web. |
|**Déployer** | 1. Déployez l’application web. |

## <a name="use-the-deployment-center"></a>Utiliser le centre de déploiement

Vous pouvez rapidement commencer à utiliser GitHub Actions à l’aide du centre de déploiement App Service. Cela génère automatiquement un fichier de workflow basé sur votre pile d’applications et le valide dans votre référentiel GitHub dans le répertoire approprié.

1. Accédez à votre application web dans le portail Azure.
1. Sur le côté gauche, cliquez sur **Centre de déploiement**.
1. Sous **Déploiement continu (CI/CD)** , sélectionnez **GitHub**.
1. Ensuite, sélectionnez **GitHub Actions**.
1. Utilisez les listes déroulantes pour sélectionner votre référentiel GitHub, votre branche et votre pile d’applications.
    - Si la branche sélectionnée est protégée, vous pouvez toujours continuer à ajouter le fichier de workflow. Assurez-vous de vérifier les protections de votre branche avant de continuer.
1. Sur le dernier écran, vous pouvez passer en revue vos sélections et afficher un aperçu du fichier de workflow qui sera validé dans le référentiel. Si les sélections sont correctes, cliquez sur **Terminer**.

Cette action validera le fichier de workflow dans le référentiel. Le workflow de génération et de déploiement de votre application démarrera immédiatement.

## <a name="set-up-a-workflow-manually"></a>Configurer un workflow manuellement

Vous pouvez également déployer un workflow sans utiliser le centre de déploiement. Pour ce faire, vous devez d’abord générer des informations d’identification de déploiement. 

## <a name="generate-deployment-credentials"></a>Générer les informations d’identification du déploiement

La méthode recommandée pour l’authentification auprès d’Azure App Service pour GitHub Actions consiste à utiliser un profil de publication. Vous pouvez également vous authentifier avec un principal de service, mais le processus requiert des étapes supplémentaires. 

Enregistrez les informations d’identification de votre profil de publication ou le principal du service en tant que [secret GitHub](https://docs.github.com/en/free-pro-team@latest/actions/reference/encrypted-secrets) pour vous authentifier auprès d’Azure. Vous allez accéder au secret dans votre workflow. 

# <a name="publish-profile"></a>[Profil de publication](#tab/applevel)

Un profil de publication est une information d’identification au niveau de l’application. Configurez votre profil de publication en tant que secret GitHub. 

1. Accédez à votre service d’application dans le portail Azure. 

1. Dans la page **Vue d’ensemble**, sélectionnez **Obtenir le profil de publication**.

1. Enregistrez le fichier téléchargé. Vous utiliserez le contenu du fichier pour créer un secret GitHub.

> [!NOTE]
> À compter d’octobre 2020, les applications web Linux ont besoin que le paramètre d’application `WEBSITE_WEBDEPLOY_USE_SCM` soit défini sur `true` **avant de télécharger le profil de publication**. Cette condition sera supprimée ultérieurement.

# <a name="service-principal"></a>[Principal du service](#tab/userlevel)

Vous pouvez créer un [principal de service](../active-directory/develop/app-objects-and-service-principals.md#service-principal-object) à l’aide de la commande [az ad sp create-for-rbac](/cli/azure/ad/sp#az-ad-sp-create-for-rbac) dans [Azure CLI](/cli/azure/). Exécutez cette commande en utilisant [Azure Cloud Shell](https://shell.azure.com/) dans le portail Azure ou en sélectionnant le bouton **Essayer**.

```azurecli-interactive
az ad sp create-for-rbac --name "myApp" --role contributor \
                            --scopes /subscriptions/<subscription-id>/resourceGroups/<group-name>/providers/Microsoft.Web/sites/<app-name> \
                            --sdk-auth
```

Dans l’exemple ci-dessus, remplacez les espaces réservés par votre ID d’abonnement, le nom de votre groupe de ressources et le nom de votre application. La sortie correspond à un objet JSON avec les informations d’identification de l’attribution de rôle qui fournit l’accès à votre application App Service, similaire à ce qui suit. Copiez cet objet JSON pour une version ultérieure.

```output 
  {
    "clientId": "<GUID>",
    "clientSecret": "<GUID>",
    "subscriptionId": "<GUID>",
    "tenantId": "<GUID>",
    (...)
  }
```

> [!IMPORTANT]
> Il est toujours conseillé d’accorder un accès minimal. L’étendue dans l’exemple précédent est limitée à l’application App Service spécifique, et non à l’ensemble du groupe de ressources.

---

## <a name="configure-the-github-secret"></a>Configurer le secret GitHub


# <a name="publish-profile"></a>[Profil de publication](#tab/applevel)

Dans [GitHub](https://github.com/), parcourez votre référentiel, sélectionnez **Paramètres > Secrets > Ajouter un nouveau secret**.

Pour utiliser les [informations d’identification au niveau de l’application](#generate-deployment-credentials), collez le contenu du fichier de profil de publication téléchargé dans le champ de valeur du secret. Nommez le secret `AZURE_WEBAPP_PUBLISH_PROFILE`.

Quand vous configurez votre workflow GitHub, vous utilisez `AZURE_WEBAPP_PUBLISH_PROFILE` dans l’action Déployer l’application web Azure. Par exemple :
    
```yaml
- uses: azure/webapps-deploy@v2
  with:
    publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
```

# <a name="service-principal"></a>[Principal du service](#tab/userlevel)

Dans [GitHub](https://github.com/), parcourez votre référentiel, sélectionnez **Paramètres > Secrets > Ajouter un nouveau secret**.

Pour utiliser les [informations d’identification au niveau de l’utilisateur](#generate-deployment-credentials), collez l’intégralité de la sortie JSON à partir de la commande Azure CLI dans le champ de valeur du secret. Nommez le secret `AZURE_CREDENTIALS`.

Quand vous configurez le fichier de flux de travail ultérieurement, vous utilisez le secret pour l’entrée `creds` de l’action de connexion Azure. Par exemple :

```yaml
- uses: azure/login@v1
  with:
    creds: ${{ secrets.AZURE_CREDENTIALS }}
```

---

## <a name="set-up-the-environment"></a>Configurer l’environnement

La configuration de l’environnement peut être effectuée à l’aide de l’une des actions de configuration.

|**Langage**  |**Action de configuration**  |
|---------|---------|
|**.NET**     | `actions/setup-dotnet` |
|**ASP.NET**     | `actions/setup-dotnet` |
|**Java**     | `actions/setup-java` |
|**JavaScript** | `actions/setup-node` |
|**Python**     | `actions/setup-python` |

Les exemples suivants montrent comment configurer l’environnement pour les différentes langages pris en charge :

**.NET**

```yaml
    - name: Setup Dotnet 3.3.x
      uses: actions/setup-dotnet@v1
      with:
        dotnet-version: '3.3.x'
```

**ASP.NET**

```yaml
    - name: Install Nuget
      uses: nuget/setup-nuget@v1
      with:
        nuget-version: ${{ env.NUGET_VERSION}}
```

**Java**

```yaml
    - name: Setup Java 1.8.x
      uses: actions/setup-java@v1
      with:
        # If your pom.xml <maven.compiler.source> version is not in 1.8.x,
        # change the Java version to match the version in pom.xml <maven.compiler.source>
        java-version: '1.8.x'
```

**JavaScript**

```yaml
env:
  NODE_VERSION: '14.x'                # set this to the node version to use

jobs:
  build-and-deploy:
    name: Build and Deploy
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@main
    - name: Use Node.js ${{ env.NODE_VERSION }}
      uses: actions/setup-node@v1
      with:
        node-version: ${{ env.NODE_VERSION }}
```
**Python**

```yaml
    - name: Setup Python 3.x 
      uses: actions/setup-python@v1
      with:
        python-version: 3.x
```

## <a name="build-the-web-app"></a>Créer l’application Web

Le processus de création d’une application web et de déploiement vers Azure App Service change en fonction du langage. 

Les exemples suivants illustrent la partie du workflow qui génère l’application web, dans différents langages pris en charge.

Pour tous les langages, vous pouvez définir le répertoire racine de l’application web avec `working-directory`. 

**.NET**

La variable d’environnement `AZURE_WEBAPP_PACKAGE_PATH` définit le chemin d’accès de votre projet d’application web. 

```yaml
- name: dotnet build and publish
  run: |
    dotnet restore
    dotnet build --configuration Release
    dotnet publish -c Release -o '${{ env.AZURE_WEBAPP_PACKAGE_PATH }}/myapp' 
```
**ASP.NET**

Vous pouvez restaurer les dépendances NuGet et exécuter MSBuild avec `run`. 

```yaml
- name: NuGet to restore dependencies as well as project-specific tools that are specified in the project file
  run: nuget restore

- name: Add msbuild to PATH
  uses: microsoft/setup-msbuild@v1.0.0

- name: Run msbuild
  run: msbuild .\SampleWebApplication.sln
```

**Java**

```yaml
- name: Build with Maven
  run: mvn package --file pom.xml
```

**JavaScript**

Pour Node.js, vous pouvez définir `working-directory` ou changer pour le répertoire npm dans `pushd`. 

```yaml
- name: npm install, build, and test
  run: |
    npm install
    npm run build --if-present
    npm run test --if-present
  working-directory: my-app-folder # set to the folder with your app if it is not the root directory
```

**Python**

```yaml
- name: Install dependencies
  run: |
    python -m pip install --upgrade pip
    pip install -r requirements.txt
```


## <a name="deploy-to-app-service"></a>Déployer dans App Service

Pour déployer votre code dans une application App Service, utilisez l’action `azure/webapps-deploy@v2`. Cette action a quatre paramètres :

| **Paramètre**  | **Explication**  |
|---------|---------|
| **app-name** | (Requis) Nom de l’application App Service | 
| **publish-profile** | (Facultatif) Publier le contenu du fichier de profil avec les secrets Web Deploy |
| **package** | (Facultatif) Chemin d’accès au package ou dossier. Le chemin d’accès peut inclure *.zip, *.war, *.jar ou un dossier à déployer |
| **slot-name** | (Facultatif) Entrer un emplacement existant autre que l’[emplacement](deploy-staging-slots.md) de production |


# <a name="publish-profile"></a>[Profil de publication](#tab/applevel)

### <a name="net-core"></a>.NET Core

Créez et déployez une application .NET Core sur Azure à l’aide d’un profil de publication Azure. L’entrée `publish-profile` fait référence au secret `AZURE_WEBAPP_PUBLISH_PROFILE` que vous avez créé précédemment.

```yaml
name: .NET Core CI

on: [push]

env:
  AZURE_WEBAPP_NAME: my-app-name    # set this to your application's name
  AZURE_WEBAPP_PACKAGE_PATH: '.'      # set this to the path to your web app project, defaults to the repository root
  DOTNET_VERSION: '3.1.x'           # set this to the dot net version to use

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      # Checkout the repo
      - uses: actions/checkout@main
      
      # Setup .NET Core SDK
      - name: Setup .NET Core
        uses: actions/setup-dotnet@v1
        with:
          dotnet-version: ${{ env.DOTNET_VERSION }} 
      
      # Run dotnet build and publish
      - name: dotnet build and publish
        run: |
          dotnet restore
          dotnet build --configuration Release
          dotnet publish -c Release -o '${{ env.AZURE_WEBAPP_PACKAGE_PATH }}/myapp' 
          
      # Deploy to Azure Web apps
      - name: 'Run Azure webapp deploy action using publish profile credentials'
        uses: azure/webapps-deploy@v2
        with: 
          app-name: ${{ env.AZURE_WEBAPP_NAME }} # Replace with your app name
          publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE  }} # Define secret variable in repository settings as per action documentation
          package: '${{ env.AZURE_WEBAPP_PACKAGE_PATH }}/myapp'
```

### <a name="aspnet"></a>ASP.NET

Générez et déployez une application ASP.NET MVC qui utilise NuGet et `publish-profile` pour l’authentification. 


```yaml
name: Deploy ASP.NET MVC App deploy to Azure Web App

on: [push]

env:
  AZURE_WEBAPP_NAME: my-app    # set this to your application's name
  AZURE_WEBAPP_PACKAGE_PATH: '.'      # set this to the path to your web app project, defaults to the repository root
  NUGET_VERSION: '5.3.x'           # set this to the dot net version to use

jobs:
  build-and-deploy:
    runs-on: windows-latest
    steps:

    - uses: actions/checkout@main  
    
    - name: Install Nuget
      uses: nuget/setup-nuget@v1
      with:
        nuget-version: ${{ env.NUGET_VERSION}}
    - name: NuGet to restore dependencies as well as project-specific tools that are specified in the project file
      run: nuget restore
  
    - name: Add msbuild to PATH
      uses: microsoft/setup-msbuild@v1.0.0

    - name: Run MSBuild
      run: msbuild .\SampleWebApplication.sln
       
    - name: 'Run Azure webapp deploy action using publish profile credentials'
      uses: azure/webapps-deploy@v2
      with: 
        app-name: ${{ env.AZURE_WEBAPP_NAME }} # Replace with your app name
        publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE  }} # Define secret variable in repository settings as per action documentation
        package: '${{ env.AZURE_WEBAPP_PACKAGE_PATH }}/SampleWebApplication/'
```

### <a name="java"></a>Java

Créez et déployez une application Java Spring sur Azure à l’aide d’un profil de publication Azure. L’entrée `publish-profile` fait référence au secret `AZURE_WEBAPP_PUBLISH_PROFILE` que vous avez créé précédemment.

```yaml
name: Java CI with Maven

on: [push]

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - name: Set up JDK 1.8
      uses: actions/setup-java@v1
      with:
        java-version: 1.8
    - name: Build with Maven
      run: mvn -B package --file pom.xml
      working-directory: my-app-path
    - name: Azure WebApp
      uses: Azure/webapps-deploy@v2
      with:
        app-name: my-app-name
        publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
        package: my/target/*.jar
```

Pour déployer un `war` au lieu d’un `jar`, modifiez la valeur `package`. 


```yaml
    - name: Azure WebApp
      uses: Azure/webapps-deploy@v2
      with:
        app-name: my-app-name
        publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
        package: my/target/*.war
```

### <a name="javascript"></a>JavaScript 

Créez et déployez une application Node.js sur Azure à l’aide du profil de publication de l’application. L’entrée `publish-profile` fait référence au secret `AZURE_WEBAPP_PUBLISH_PROFILE` que vous avez créé précédemment.

```yaml
# File: .github/workflows/workflow.yml
name: JavaScript CI

on: [push]

env:
  AZURE_WEBAPP_NAME: my-app-name   # set this to your application's name
  AZURE_WEBAPP_PACKAGE_PATH: 'my-app-path'      # set this to the path to your web app project, defaults to the repository root
  NODE_VERSION: '14.x'                # set this to the node version to use

jobs:
  build-and-deploy:
    name: Build and Deploy
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@main
    - name: Use Node.js ${{ env.NODE_VERSION }}
      uses: actions/setup-node@v1
      with:
        node-version: ${{ env.NODE_VERSION }}
    - name: npm install, build, and test
      run: |
        # Build and test the project, then
        # deploy to Azure Web App.
        npm install
        npm run build --if-present
        npm run test --if-present
      working-directory: my-app-path
    - name: 'Deploy to Azure WebApp'
      uses: azure/webapps-deploy@v2
      with: 
        app-name: ${{ env.AZURE_WEBAPP_NAME }}
        publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
        package: ${{ env.AZURE_WEBAPP_PACKAGE_PATH }}
```

### <a name="python"></a>Python 

Créez et déployez une application Python sur Azure à l’aide du profil de publication de l’application. Notez la manière dont l’entrée `publish-profile` fait référence au secret `AZURE_WEBAPP_PUBLISH_PROFILE` que vous avez créé précédemment.

```yaml
name: Python CI

on:
  [push]

env:
  AZURE_WEBAPP_NAME: my-web-app # set this to your application's name
  AZURE_WEBAPP_PACKAGE_PATH: '.' # set this to the path to your web app project, defaults to the repository root

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    - name: Set up Python 3.x
      uses: actions/setup-python@v2
      with:
        python-version: 3.x
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
    - name: Building web app
      uses: azure/appservice-build@v2
    - name: Deploy web App using GH Action azure/webapps-deploy
      uses: azure/webapps-deploy@v2
      with:
        app-name: ${{ env.AZURE_WEBAPP_NAME }}
        publish-profile: ${{ secrets.AZURE_WEBAPP_PUBLISH_PROFILE }}
        package: ${{ env.AZURE_WEBAPP_PACKAGE_PATH }}
```

# <a name="service-principal"></a>[Principal du service](#tab/userlevel)

### <a name="net-core"></a>.NET Core 

Créez et déployez une application .NET Core sur Azure à l’aide d’un principal de service Azure. Notez la manière dont l’entrée `creds` fait référence au secret `AZURE_CREDENTIALS` que vous avez créé précédemment.


```yaml
name: .NET Core

on: [push]

env:
  AZURE_WEBAPP_NAME: my-app    # set this to your application's name
  AZURE_WEBAPP_PACKAGE_PATH: '.'      # set this to the path to your web app project, defaults to the repository root
  DOTNET_VERSION: '3.1.x'           # set this to the dot net version to use

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      # Checkout the repo
      - uses: actions/checkout@main
      - uses: azure/login@v1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      
      # Setup .NET Core SDK
      - name: Setup .NET Core
        uses: actions/setup-dotnet@v1
        with:
          dotnet-version: ${{ env.DOTNET_VERSION }} 
      
      # Run dotnet build and publish
      - name: dotnet build and publish
        run: |
          dotnet restore
          dotnet build --configuration Release
          dotnet publish -c Release -o '${{ env.AZURE_WEBAPP_PACKAGE_PATH }}/myapp' 
          
      # Deploy to Azure Web apps
      - name: 'Run Azure webapp deploy action using publish profile credentials'
        uses: azure/webapps-deploy@v2
        with: 
          app-name: ${{ env.AZURE_WEBAPP_NAME }} # Replace with your app name
          package: '${{ env.AZURE_WEBAPP_PACKAGE_PATH }}/myapp'
      
      - name: logout
        run: |
          az logout
```

### <a name="aspnet"></a>ASP.NET

Créez et déployez une application ASP.NET MVC sur Azure à l’aide d’un principal de service Azure. Notez la manière dont l’entrée `creds` fait référence au secret `AZURE_CREDENTIALS` que vous avez créé précédemment.

```yaml
name: Deploy ASP.NET MVC App deploy to Azure Web App

on: [push]

env:
  AZURE_WEBAPP_NAME: my-app    # set this to your application's name
  AZURE_WEBAPP_PACKAGE_PATH: '.'      # set this to the path to your web app project, defaults to the repository root
  NUGET_VERSION: '5.3.x'           # set this to the dot net version to use

jobs:
  build-and-deploy:
    runs-on: windows-latest
    steps:

    # checkout the repo
    - uses: actions/checkout@main
    
    - uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}

    - name: Install Nuget
      uses: nuget/setup-nuget@v1
      with:
        nuget-version: ${{ env.NUGET_VERSION}}
    - name: NuGet to restore dependencies as well as project-specific tools that are specified in the project file
      run: nuget restore
  
    - name: Add msbuild to PATH
      uses: microsoft/setup-msbuild@v1.0.0

    - name: Run MSBuild
      run: msbuild .\SampleWebApplication.sln
       
    - name: 'Run Azure webapp deploy action using publish profile credentials'
      uses: azure/webapps-deploy@v2
      with: 
        app-name: ${{ env.AZURE_WEBAPP_NAME }} # Replace with your app name
        package: '${{ env.AZURE_WEBAPP_PACKAGE_PATH }}/SampleWebApplication/'
  
    # Azure logout 
    - name: logout
      run: |
        az logout
```

### <a name="java"></a>Java 

Créez et déployez une application Java Spring sur Azure à l’aide d’un principal de service Azure. Notez la manière dont l’entrée `creds` fait référence au secret `AZURE_CREDENTIALS` que vous avez créé précédemment.

```yaml
name: Java CI with Maven

on: [push]

jobs:
  build:

    runs-on: ubuntu-latest

    steps:
    - uses: actions/checkout@v2
    - uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
    - name: Set up JDK 1.8
      uses: actions/setup-java@v1
      with:
        java-version: 1.8
    - name: Build with Maven
      run: mvn -B package --file pom.xml
      working-directory: complete
    - name: Azure WebApp
      uses: Azure/webapps-deploy@v2
      with:
        app-name: my-app-name
        package: my/target/*.jar

    # Azure logout 
    - name: logout
      run: |
        az logout
```

### <a name="javascript"></a>JavaScript 

Créez et déployez une application Node.js sur Azure à l’aide d’un principal de service Azure. Notez la manière dont l’entrée `creds` fait référence au secret `AZURE_CREDENTIALS` que vous avez créé précédemment.

```yaml
name: JavaScript CI

on: [push]

name: Node.js

env:
  AZURE_WEBAPP_NAME: my-app   # set this to your application's name
  NODE_VERSION: '14.x'                # set this to the node version to use

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
    # checkout the repo
    - name: 'Checkout GitHub Action' 
      uses: actions/checkout@main
   
    - uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
        
    - name: Setup Node ${{ env.NODE_VERSION }}
      uses: actions/setup-node@v1
      with:
        node-version: ${{ env.NODE_VERSION }}
    
    - name: 'npm install, build, and test'
      run: |
        npm install
        npm run build --if-present
        npm run test --if-present
      working-directory:  my-app-path
               
    # deploy web app using Azure credentials
    - uses: azure/webapps-deploy@v2
      with:
        app-name: ${{ env.AZURE_WEBAPP_NAME }}
        package: ${{ env.AZURE_WEBAPP_PACKAGE_PATH }}

    # Azure logout 
    - name: logout
      run: |
        az logout
```

### <a name="python"></a>Python 

Créez et déployez une application Python sur Azure à l’aide d’un principal de service Azure. Notez la manière dont l’entrée `creds` fait référence au secret `AZURE_CREDENTIALS` que vous avez créé précédemment.

```yaml
name: Python application

on:
  [push]

env:
  AZURE_WEBAPP_NAME: my-app # set this to your application's name
  AZURE_WEBAPP_PACKAGE_PATH: '.' # set this to the path to your web app project, defaults to the repository root

jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@v2
    
    - uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}

    - name: Set up Python 3.x
      uses: actions/setup-python@v2
      with:
        python-version: 3.x
    - name: Install dependencies
      run: |
        python -m pip install --upgrade pip
        pip install -r requirements.txt
    - name: Deploy web App using GH Action azure/webapps-deploy
      uses: azure/webapps-deploy@v2
      with:
        app-name: ${{ env.AZURE_WEBAPP_NAME }}
        package: ${{ env.AZURE_WEBAPP_PACKAGE_PATH }}
    - name: logout
      run: |
        az logout
```


---

## <a name="next-steps"></a>Étapes suivantes

Vous pouvez trouver notre ensemble d’Actions regroupées dans différents référentiels GitHub, chacun contenant de la documentation et des exemples pour vous aider à utiliser GitHub pour les opérations de CI/CD et à déployer vos applications sur Azure.

- [Flux de travail d’actions à déployer sur Azure](https://github.com/Azure/actions-workflow-samples)

- [Connexion à Azure](https://github.com/Azure/login)

- [Azure Web App](https://github.com/Azure/webapps-deploy)

- [Azure Web App pour conteneurs](https://github.com/Azure/webapps-container-deploy)

- [Connexion/déconnexion de Docker](https://github.com/Azure/docker-login)

- [Événements qui déclenchent des flux de travail](https://docs.github.com/en/free-pro-team@latest/actions/reference/events-that-trigger-workflows)

- [Déployer K8](https://github.com/Azure/k8s-deploy)

- [Flux de travail de démarrage](https://github.com/actions/starter-workflows)
