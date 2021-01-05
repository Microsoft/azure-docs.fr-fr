---
title: CI/CD Azure Spring Cloud avec GitHub Actions
description: Guide pratique pour générer le workflow CI/CD pour Azure Spring Cloud avec GitHub Actions
author: MikeDodaro
ms.author: barbkess
ms.service: spring-cloud
ms.topic: how-to
ms.date: 09/08/2020
ms.custom: devx-track-java, devx-track-azurecli
zone_pivot_groups: programming-languages-spring-cloud
ms.openlocfilehash: 95a97d3f5104891ef5e23a03ffe1477f9b6ad036
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97561611"
---
# <a name="azure-spring-cloud-cicd-with-github-actions"></a>CI/CD Azure Spring Cloud avec GitHub Actions

GitHub Actions prend en charge un workflow automatisé de cycle de vie de développement logiciel. Grâce à GitHub Actions pour Azure Spring Cloud, vous pouvez créer des workflows dans votre référentiel pour générer, tester, créer un package, publier et déployer sur Azure. 

## <a name="prerequisites"></a>Prérequis
Cet exemple nécessite [Azure CLI](/cli/azure/install-azure-cli?preserve-view=true&view=azure-cli-latest).

::: zone pivot="programming-language-csharp"
## <a name="set-up-github-repository-and-authenticate"></a>Configurer le référentiel GitHub et s’authentifier
Vous avez besoin des informations d’identification du principal de service Azure pour autoriser une action de connexion Azure. Pour récupérer des informations d’identification Azure, exécutez les commandes suivantes sur votre ordinateur local :

```
az login
az ad sp create-for-rbac --role contributor --scopes /subscriptions/<SUBSCRIPTION_ID> --sdk-auth 
```

Pour accéder à un groupe de ressources spécifique, vous pouvez réduire l’étendue :

```
az ad sp create-for-rbac --role contributor --scopes /subscriptions/<SUBSCRIPTION_ID>/resourceGroups/<RESOURCE_GROUP> --sdk-auth
```

La commande doit générer un objet JSON :

```JSON
{
    "clientId": "<GUID>",
    "clientSecret": "<GUID>",
    "subscriptionId": "<GUID>",
    "tenantId": "<GUID>",
    ...
}
```

Cet exemple utilise l’[échantillon steeltoe sur GitHub](https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples/tree/master/steeltoe-sample).  Dupliquez le référentiel, ouvrez la page de référentiel GitHub pour la fourche, puis sélectionnez l’onglet **Paramètres**. Ouvrez le menu **Secrets**, puis sélectionnez **Nouveau secret** :

 ![Ajouter un nouveau secret](./media/github-actions/actions1.png)

Définissez le nom du secret sur `AZURE_CREDENTIALS` et sa valeur sur la chaîne JSON que vous avez trouvée sous le titre *Configurer le référentiel GitHub et s’authentifier*.

 ![Définir les données du secret](./media/github-actions/actions2.png)

Vous pouvez également récupérer les informations d’identification de connexion Azure à partir de Key Vault dans GitHub Actions, comme expliqué dans [Authentifier Azure Spring Cloud auprès de Key Vault dans GitHub Actions](./spring-cloud-github-actions-key-vault.md).

## <a name="provision-service-instance"></a>Approvisionner une instance de service
Pour approvisionner votre instance de service Azure Spring Cloud, exécutez les commandes suivantes à l’aide d’Azure CLI.

```azurecli
az extension add --name spring-cloud
az group create --location eastus --name <resource group name>
az spring-cloud create -n <service instance name> -g <resource group name>
az spring-cloud config-server git set -n <service instance name> --uri https://github.com/xxx/Azure-Spring-Cloud-Samples --label main --search-paths steeltoe-sample/config
```

## <a name="build-the-workflow"></a>Générer le workflow
Le workflow est défini à l’aide des options suivantes.

### <a name="prepare-for-deployment-with-azure-cli"></a>Préparer le déploiement avec Azure CLI

La commande `az spring-cloud app create` n’est pas idempotente actuellement. Après l’avoir exécutée une fois, vous obtiendrez une erreur si vous réexécutez la même commande. Nous vous recommandons d’utiliser ce workflow sur les applications et les instances Azure Spring Cloud existantes.

Utilisez les commandes suivantes dans l’interface de ligne de commande Azure pour la préparation :
```
az configure --defaults group=<service group name>
az configure --defaults spring-cloud=<service instance name>
az spring-cloud app create --name planet-weather-provider
az spring-cloud app create --name solar-system-weather
```

### <a name="deploy-with-azure-cli-directly"></a>Déployer avec Azure CLI directement

Créez le fichier `.github/workflows/main.yml` dans le référentiel avec le contenu suivant. Remplacez `<your resource group name>` et `<your service name>` par les valeurs correctes.

```yaml
name: Steeltoe-CD

# Controls when the action will run. Triggers the workflow on push or pull request
# events but only for the main branch
on:
  push:
    branches: [ main]

# A workflow run is made up of one or more jobs that can run sequentially or in parallel
jobs:
  # This workflow contains a single job called "build"
  build:
    # The type of runner that the job will run on
    runs-on: ubuntu-latest
    env:
      working-directory: ./steeltoe-sample
      resource-group-name: <your resource group name>
      service-name: <your service name>
    
    # Supported .NET Core version matrix.
    strategy:
      matrix:
        dotnet: [ '3.1.x' ]

    # Steps represent a sequence of tasks that will be executed as part of the job
    steps:
      # Checks-out your repository under $GITHUB_WORKSPACE, so your job can access it
      - uses: actions/checkout@v2

      # Set up .NET Core 3.1 SDK
      - uses: actions/setup-dotnet@v1
        with:
          dotnet-version: ${{ matrix.dotnet }}
          
      # Set credential for az login
      - uses: azure/login@v1.1
        with:
          creds: ${{ secrets.AZURE_CREDENTIALS }}

      - name: install Azure CLI extension
        run: |
          az extension add --name spring-cloud --yes
      
      - name: Build and package planet-weather-provider app
        working-directory: ${{env.working-directory}}/src/planet-weather-provider
        run: |
          dotnet publish
          az spring-cloud app deploy -n planet-weather-provider --runtime-version NetCore_31 --main-entry Microsoft.Azure.SpringCloud.Sample.PlanetWeatherProvider.dll --artifact-path ./publish-deploy-planet.zip -s ${{ env.service-name }} -g ${{ env.resource-group-name }}
      - name: Build solar-system-weather app
        working-directory: ${{env.working-directory}}/src/solar-system-weather
        run: |
          dotnet publish
          az spring-cloud app deploy -n solar-system-weather --runtime-version NetCore_31 --main-entry Microsoft.Azure.SpringCloud.Sample.SolarSystemWeather.dll --artifact-path ./publish-deploy-solar.zip -s ${{ env.service-name }} -g ${{ env.resource-group-name }}
```
::: zone-end

::: zone pivot="programming-language-java"
## <a name="set-up-github-repository-and-authenticate"></a>Configurer le référentiel GitHub et s’authentifier
Vous avez besoin des informations d’identification du principal de service Azure pour autoriser une action de connexion Azure. Pour récupérer des informations d’identification Azure, exécutez les commandes suivantes sur votre ordinateur local :
```
az login
az ad sp create-for-rbac --role contributor --scopes /subscriptions/<SUBSCRIPTION_ID> --sdk-auth 
```
Pour accéder à un groupe de ressources spécifique, vous pouvez réduire l’étendue :
```
az ad sp create-for-rbac --role contributor --scopes /subscriptions/<SUBSCRIPTION_ID>/resourceGroups/<RESOURCE_GROUP> --sdk-auth
```
La commande doit générer un objet JSON :
```JSON
{
    "clientId": "<GUID>",
    "clientSecret": "<GUID>",
    "subscriptionId": "<GUID>",
    "tenantId": "<GUID>",
    ...
}
```

Cet exemple utilise l’échantillon [PiggyMetrics](https://github.com/Azure-Samples/piggymetrics) sur GitHub.  Dupliquez l’échantillon, ouvrez la page du référentiel GitHub, puis cliquez sur l’onglet **Paramètres**. Ouvrez le menu **Secrets**, puis cliquez sur **Ajouter un nouveau secret** :

 ![Ajouter un nouveau secret](./media/github-actions/actions1.png)

Définissez le nom du secret sur `AZURE_CREDENTIALS` et sa valeur sur la chaîne JSON que vous avez trouvée sous le titre *Configurer le référentiel GitHub et s’authentifier*.

 ![Définir les données du secret](./media/github-actions/actions2.png)

Vous pouvez également récupérer les informations d’identification de connexion Azure à partir de Key Vault dans GitHub Actions, comme expliqué dans [Authentifier Azure Spring Cloud auprès de Key Vault dans GitHub Actions](./spring-cloud-github-actions-key-vault.md).

## <a name="provision-service-instance"></a>Approvisionner une instance de service
Pour approvisionner votre instance de service Azure Spring Cloud, exécutez les commandes suivantes à l’aide d’Azure CLI.
```
az extension add --name spring-cloud
az group create --location eastus --name <resource group name>
az spring-cloud create -n <service instance name> -g <resource group name>
az spring-cloud config-server git set -n <service instance name> --uri https://github.com/xxx/piggymetrics --label config
```
## <a name="build-the-workflow"></a>Générer le workflow
Le workflow est défini à l’aide des options suivantes.

### <a name="prepare-for-deployment-with-azure-cli"></a>Préparer le déploiement avec Azure CLI
La commande `az spring-cloud app create` n’est pas idempotente actuellement.  Nous vous recommandons d’utiliser ce workflow sur les applications et les instances Azure Spring Cloud existantes.

Utilisez les commandes suivantes dans l’interface de ligne de commande Azure pour la préparation :
```
az configure --defaults group=<service group name>
az configure --defaults spring-cloud=<service instance name>
az spring-cloud app create --name gateway
az spring-cloud app create --name auth-service
az spring-cloud app create --name account-service
```

### <a name="deploy-with-azure-cli-directly"></a>Déployer avec Azure CLI directement
Créez le fichier `.github/workflow/main.yml` dans le référentiel :

```
name: AzureSpringCloud
on: push

env:
  GROUP: <resource group name>
  SERVICE_NAME: <service instance name>

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
    
    - uses: actions/checkout@main
    
    - name: Set up JDK 1.8
      uses: actions/setup-java@v1
      with:
        java-version: 1.8
    
    - name: maven build, clean
      run: |
        mvn clean package -DskipTests
    
    - name: Azure Login
      uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
      
    - name: Install ASC AZ extension
      run: az extension add --name spring-cloud
   
    - name: Deploy with AZ CLI commands
      run: |
        az configure --defaults group=$GROUP
        az configure --defaults spring-cloud=$SERVICE_NAME
        az spring-cloud app deploy -n gateway --jar-path ${{ github.workspace }}/gateway/target/gateway.jar
        az spring-cloud app deploy -n account-service --jar-path ${{ github.workspace }}/account-service/target/account-service.jar
        az spring-cloud app deploy -n auth-service --jar-path ${{ github.workspace }}/auth-service/target/auth-service.jar
```
### <a name="deploy-with-azure-cli-action"></a>Déployer avec une action Azure CLI
La commande az `run` utilise la version la plus récente d’Azure CLI. En cas de changements cassants, vous pouvez également utiliser une version spécifique d’Azure CLI à l’aide de la commande azure/CLI `action`. 

> [!Note] 
> Cette commande s’exécute dans un nouveau conteneur, de sorte que `env` ne fonctionne pas, et l’accès aux fichiers d’action croisée peut présenter des restrictions supplémentaires.

Créez le fichier .github/workflow/main.yml dans le référentiel :
```
name: AzureSpringCloud
on: push

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
    
    - uses: actions/checkout@main
    
    - name: Set up JDK 1.8
      uses: actions/setup-java@v1
      with:
        java-version: 1.8
    
    - name: maven build, clean
      run: |
        mvn clean package -DskipTests
        
    - name: Azure Login
      uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
              
    - name: Azure CLI script
      uses: azure/CLI@v1
      with:
        azcliversion: 2.0.75
        inlineScript: |
          az extension add --name spring-cloud
          az configure --defaults group=<service group name>
          az configure --defaults spring-cloud=<service instance name>
          az spring-cloud app deploy -n gateway --jar-path $GITHUB_WORKSPACE/gateway/target/gateway.jar
          az spring-cloud app deploy -n account-service --jar-path $GITHUB_WORKSPACE/account-service/target/account-service.jar
          az spring-cloud app deploy -n auth-service --jar-path $GITHUB_WORKSPACE/auth-service/target/auth-service.jar
```

## <a name="deploy-with-maven-plugin"></a>Déployer avec le plug-in Maven
Une autre option consiste à utiliser le [plug-in Maven](./spring-cloud-quickstart.md) pour déployer le fichier jar et mettre à jour les paramètres de l’application. La commande `mvn azure-spring-cloud:deploy` est idempotente et crée automatiquement des applications si nécessaire. Vous n’avez pas besoin de créer des applications correspondantes à l’avance.

```
name: AzureSpringCloud
on: push

jobs:
  build-and-deploy:
    runs-on: ubuntu-latest
    steps:
    
    - uses: actions/checkout@main
    
    - name: Set up JDK 1.8
      uses: actions/setup-java@v1
      with:
        java-version: 1.8
    
    - name: maven build, clean
      run: |
        mvn clean package -DskipTests
        
    # Maven plugin can cosume this authentication method automatically
    - name: Azure Login
      uses: azure/login@v1
      with:
        creds: ${{ secrets.AZURE_CREDENTIALS }}
    
    # Maven deploy, make sure you have correct configurations in your pom.xml
    - name: deploy to Azure Spring Cloud using Maven
      run: |
        mvn azure-spring-cloud:deploy
```

::: zone-end

## <a name="run-the-workflow"></a>Exécuter le workflow

**GitHub Actions** doit être activé automatiquement une fois que vous avez envoyé (push) `.github/workflow/main.yml` à GitHub. L’action sera déclenchée lorsque vous enverrez (push) une nouvelle validation. Si vous créez ce fichier dans le navigateur, votre action doit déjà avoir été exécutée.

Pour vérifier que l’action a été activée, cliquez sur l’onglet **Actions** de la page du référentiel GitHub :

![Vérifier l’activation de l’action](./media/github-actions/actions3.png)

Si votre action s’exécute par erreur, par exemple si vous n’avez pas défini les informations d’identification Azure, vous pouvez réexécuter les vérifications après avoir corrigé l’erreur. Dans la page du référentiel GitHub, cliquez sur **Actions**, sélectionnez la tâche de workflow spécifique, puis cliquez sur le bouton **Réexécuter les vérifications** pour réexécuter les vérifications :

![Réexécuter les vérifications](./media/github-actions/actions4.png)

## <a name="next-steps"></a>Étapes suivantes

* [Key Vault pour GitHub Actions avec Spring Cloud](./spring-cloud-github-actions-key-vault.md)
* [Principaux du service Azure Active Directory](/cli/azure/ad/sp?preserve-view=true&view=azure-cli-latest#az-ad-sp-create-for-rbac)
* [GitHub Actions pour Azure](https://github.com/Azure/actions/)
