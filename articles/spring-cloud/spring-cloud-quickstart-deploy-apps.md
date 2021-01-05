---
title: Démarrage rapide - Créer et déployer des applications sur Azure Spring Cloud
description: Décrit le déploiement d’applications sur Azure Spring Cloud.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: quickstart
ms.date: 08/03/2020
ms.custom: devx-track-java, devx-track-azurecli
zone_pivot_groups: programming-languages-spring-cloud
ms.openlocfilehash: 173e6541b4113a5d2e71d76b3b939a69d5224b5a
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92735590"
---
# <a name="quickstart-build-and-deploy-apps-to-azure-spring-cloud"></a>Démarrage rapide : Créer et déployer des applications sur Azure Spring Cloud

::: zone pivot="programming-language-csharp"
Dans ce guide de démarrage rapide, vous allez créer et déployer des applications de microservices sur Azure Spring Cloud à l’aide d’Azure CLI.

## <a name="prerequisites"></a>Prérequis

* Suivez les guides de démarrage rapide précédents de cette série :

  * [Provisionner le service Azure Spring Cloud](spring-cloud-quickstart-provision-service-instance.md).
  * [Configurer un serveur de configuration Azure Spring Cloud](spring-cloud-quickstart-setup-config-server.md).

## <a name="download-the-sample-app"></a>Télécharger l’exemple d’application

Si vous avez utilisé Azure Cloud Shell jusqu’à présent, passez à une invite de commandes locale pour effectuer les étapes ci-dessous.

1. Créez un dossier et clonez le dépôt de l’exemple d’application.

   ```console
   mkdir source-code
   ```

   ```console
   cd source-code
   ```

   ```console
   git clone https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples
   ```

1. Accédez au répertoire du dépôt.

   ```console
   cd Azure-Spring-Cloud-Samples
   ```

## <a name="deploy-planetweatherprovider"></a>Déployer PlanetWeatherProvider

1. Créez une application pour le projet PlanetWeatherProvider dans votre instance Azure Spring Cloud.

   ```azurecli
   az spring-cloud app create --name planet-weather-provider --runtime-version NetCore_31
   ```

   Pour permettre l’inscription automatique du service, vous avez donné à l’application le nom `spring.application.name` dans le fichier *appsettings.json* du projet :

   ```json
   "spring": {
     "application": {
       "name": "planet-weather-provider"
     }
   }
   ```

   L’exécution de cette commande peut durer plusieurs minutes.

1. Accédez au répertoire du dossier du projet `PlanetWeatherProvider`.

   ```console
   cd steeltoe-sample/src/planet-weather-provider
   ```

1. Créez les fichiers binaires et le fichier *.zip* à déployer.

   ```console
   dotnet publish -c release -o ./publish
   ```

   > [!TIP]
   > Le fichier projet contient le code XML suivant permettant d’empaqueter les fichiers binaires dans un fichier *.zip* après les avoir écrits dans le dossier *./publish*  :
   >
   > ```xml
   > <Target Name="Publish-Zip" AfterTargets="Publish">
   >   <ZipDirectory SourceDirectory="$(PublishDir)" DestinationFile="$(MSBuildProjectDirectory)/publish-deploy-planet.zip" Overwrite="true" />
   > </Target>
   > ```

1. Effectuez le déploiement dans Azure.

   Vérifiez que l’invite de commandes se trouve bien dans le dossier du projet avant d’exécuter la commande suivante.

   ```console
   az spring-cloud app deploy -n planet-weather-provider --runtime-version NetCore_31 --main-entry Microsoft.Azure.SpringCloud.Sample.PlanetWeatherProvider.dll --artifact-path ./publish-deploy-planet.zip
   ```

   L’option `--main-entry` spécifie le chemin relatif qui va du dossier racine où se trouve le fichier *.zip* au fichier *.dll* qui contient le point d’entrée de l’application. Une fois que le service a chargé le fichier *.zip* , il extrait tous les fichiers et dossiers, puis tente d’exécuter le point d’entrée dans le fichier *.dll* spécifié.

   L’exécution de cette commande peut durer plusieurs minutes.

## <a name="deploy-solarsystemweather"></a>Déployer SolarSystemWeather

1. Créez une autre application dans votre instance Azure Spring Cloud, cette fois pour le projet SolarSystemWeather :

   ```azurecli
   az spring-cloud app create --name solar-system-weather --runtime-version NetCore_31
   ```

   `solar-system-weather` est le nom spécifié dans le fichier *appsettings.json* du projet `SolarSystemWeather`.

   L’exécution de cette commande peut durer plusieurs minutes.

1. Accédez au répertoire du projet `SolarSystemWeather`.

   ```console
   cd ../solar-system-weather
   ```

1. Créez les fichiers binaires et le fichier *.zip* à déployer.

   ```console
   dotnet publish -c release -o ./publish
   ```

1. Effectuez le déploiement dans Azure.

   ```console
   az spring-cloud app deploy -n solar-system-weather --runtime-version NetCore_31 --main-entry Microsoft.Azure.SpringCloud.Sample.SolarSystemWeather.dll --artifact-path ./publish-deploy-solar.zip
   ```
   
   L’exécution de cette commande peut durer plusieurs minutes.

## <a name="assign-public-endpoint"></a>Affecter le point de terminaison public

Pour tester l’application, envoyez une requête HTTP GET à l’application `solar-system-weather` à partir d’un navigateur.  Pour cela, vous avez besoin d’un point de terminaison public pour la requête.

1. Pour affecter le point de terminaison, exécutez la commande suivante.

   ```azurecli
   az spring-cloud app update -n solar-system-weather --is-public true
   ```

1. Pour obtenir l’URL du point de terminaison, exécutez la commande suivante.

   Windows :

   ```azurecli
   az spring-cloud app show -n solar-system-weather -o table
   ```

   Linux :

   ```azurecli
   az spring-cloud app show --name solar-system-weather | grep url
   ```

## <a name="test-the-application"></a>Test de l’application

Envoyez une requête GET à l’application `solar-system-weather`. Dans un navigateur, accédez à l’URL publique en ajoutant `/weatherforecast` à la fin de celle-ci. Par exemple :

```
https://servicename-solar-system-weather.azuremicroservices.io/weatherforecast
```

La sortie est du code JSON :

```json
[{"Key":"Mercury","Value":"very warm"},{"Key":"Venus","Value":"quite unpleasant"},{"Key":"Mars","Value":"very cool"},{"Key":"Saturn","Value":"a little bit sandy"}]
```

Cette réponse indique que les deux applications de microservices fonctionnent. L’application `SolarSystemWeather` retourne les données qu’elle a récupérées à partir de l’application `PlanetWeatherProvider`.
::: zone-end

::: zone pivot="programming-language-java"
Ce document explique comment créer et déployer des applications de microservices sur Azure Spring Cloud en utilisant :
* Azure CLI
* Plug-in Maven
* Intellij

Avant de procéder au déploiement avec Azure CLI ou Maven, effectuez les exemples qui [provisionnent une instance Azure Spring Cloud](spring-cloud-quickstart-provision-service-instance.md) et qui [configurent le serveur de configuration](spring-cloud-quickstart-setup-config-server.md).

## <a name="prerequisites"></a>Prérequis

* [Installez JDK 8](/java/azure/jdk/?preserve-view=true&view=azure-java-stable)
* [Souscrire à un abonnement Azure](https://azure.microsoft.com/free/)
* (Facultatif) [Installez Azure CLI version 2.0.67 ou ultérieure](/cli/azure/install-azure-cli?preserve-view=true&view=azure-cli-latest) et installez l’extension Azure Spring Cloud avec la commande : `az extension add --name spring-cloud`
* (Facultatif) [Installez Azure Toolkit for IntelliJ](https://plugins.jetbrains.com/plugin/8053-azure-toolkit-for-intellij/) et [connectez-vous](/azure/developer/java/toolkit-for-intellij/create-hello-world-web-app#installation-and-sign-in).

## <a name="deployment-procedures"></a>Procédures de déploiement

#### <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/Azure-CLI)

### <a name="build-the-microservices-applications-locally"></a>Générer localement les applications de microservices

1. Clonez le dépôt des exemples d’application sur votre compte cloud Azure.  

    ```azurecli
    git clone https://github.com/Azure-Samples/piggymetrics
    ```

2. Changez de répertoire et générez le projet.

    ```azurecli
    cd piggymetrics
    mvn clean package -DskipTests
    ```

La compilation du projet prend environ 5 minutes. Une fois l’opération terminée, vous devez avoir des fichiers JAR individuels pour chaque service dans leurs dossiers respectifs.

### <a name="create-and-deploy-the-apps"></a>Créer et déployer les applications

1. Définissez le nom du groupe de ressources et le nom du cluster par défaut en exécutant les commandes suivantes :

    ```azurecli
    az configure --defaults group=<resource group name>
    az configure --defaults spring-cloud=<service instance name>
    ```

1. Créez des microservices Azure Spring Cloud en utilisant les fichiers JAR générés à l’étape précédente. Vous allez créer trois applications : **gateway** , **auth-service** et **account-service** .

    ```azurecli
    az spring-cloud app create --name gateway
    az spring-cloud app create --name auth-service
    az spring-cloud app create --name account-service
    ```

1. Nous devons déployer les applications créées à l’étape précédente sur Azure. Utilisez les commandes suivantes pour déployer les trois applications :

    ```azurecli
    az spring-cloud app deploy -n gateway --jar-path ./gateway/target/gateway.jar
    az spring-cloud app deploy -n account-service --jar-path ./account-service/target/account-service.jar
    az spring-cloud app deploy -n auth-service --jar-path ./auth-service/target/auth-service.jar
    ```

### <a name="assign-public-endpoint-to-gateway"></a>Affecter un point de terminaison public à la passerelle

Nous avons besoin d’un moyen d’accéder à l’application via un navigateur web. Notre application de passerelle a besoin d’un point de terminaison public.

1. Affectez le point de terminaison à l’aide de la commande suivante :

    ```azurecli
    az spring-cloud app update -n gateway --is-public true
    ```

2. Interrogez l’application de **passerelle** pour obtenir son adresse IP publique, afin de pouvoir vérifier que l’application est en cours d’exécution :

    ```azurecli
    az spring-cloud app show --name gateway --query properties.url
    ```

#### <a name="maven"></a>[Maven](#tab/Maven)

### <a name="clone-and-build-the-sample-application-repository"></a>Cloner et générer le dépôt d’exemples d’applications

1. Clonez le dépôt Git en exécutant la commande suivante :

    ```
    git clone https://github.com/Azure-Samples/piggymetrics
    ```
  
1. Changez d’annuaire, puis générez le projet en exécutant la commande suivante :

    ```
    cd piggymetrics
    mvn clean package -DskipTests
    ```

### <a name="generate-configurations-and-deploy-to-the-azure-spring-cloud"></a>Générer des configurations et déployer vers Azure Spring Cloud

1. Générez des configurations en exécutant la commande suivante dans le dossier racine de PiggyMetrics contenant le fichier POM parent. Si vous vous êtes déjà connecté avec Azure CLI, la commande sélectionne automatiquement les informations d’identification. Sinon, elle vous connecte avec des instructions d’invite. Pour plus d’informations, consultez notre [page wiki](https://github.com/microsoft/azure-maven-plugins/wiki/Authentication).

    ```
    mvn com.microsoft.azure:azure-spring-cloud-maven-plugin:1.3.0:config
    ```
    
    Il vous sera demandé de sélectionner les éléments suivants :
    * **Modules :** Sélectionnez `gateway`,`auth-service` et `account-service`.
    * **Abonnement :** Il s’agit de votre abonnement utilisé pour créer une instance Azure Spring Cloud.
    * **Instance de service :** Il s’agit du nom de votre instance Azure Spring Cloud.
    * **Point de terminaison public :** Dans la liste des projets fournis, entrez le numéro qui correspond à `gateway`.  Ceci lui donne un accès public.

1. Le fichier POM contient désormais les dépendances et les configurations du plug-in. Déployez les applications en utilisant la commande suivante. 

    ```
    mvn azure-spring-cloud:deploy
    ```

#### <a name="intellij"></a>[IntelliJ](#tab/IntelliJ)

### <a name="import-sample-project-in-intellij"></a>Importer l’exemple de projet dans IntelliJ

1. Téléchargez et décompressez le dépôt source pour ce tutoriel, ou clonez-le avec Git : `git clone https://github.com/Azure-Samples/piggymetrics` 

1. Ouvrez la boîte de dialogue **Welcome** d’IntelliJ et sélectionnez **Import Project** pour ouvrir l’Assistant Importation.

1. Sélectionnez le dossier `piggymetric`.

    ![Importer un projet](media/spring-cloud-intellij-howto/revision-import-project-1.png)

### <a name="deploy-gateway-app-to-azure-spring-cloud"></a>Déployer une application de passerelle sur Azure Spring Cloud
Pour effectuer un déploiement dans Azure, vous devez vous connecter avec votre compte Azure auprès d’Azure Toolkit for IntelliJ et choisir votre abonnement. Pour plus d’informations sur la connexion, consultez [Installation et connexion](/azure/developer/java/toolkit-for-intellij/create-hello-world-web-app#installation-and-sign-in).

1. Cliquez avec le bouton droit sur votre projet dans l’explorateur de projets IntelliJ, puis sélectionnez **Azure** -> **Deploy to Azure Spring Cloud** .

    ![Déployer sur Azure 1](media/spring-cloud-intellij-howto/revision-deploy-to-azure-1.png)

1. Dans le champ **Name** (Nom), ajoutez *:gateway* au **nom** existant.
1. Dans la zone de texte **Artifact** , sélectionnez *com.piggymetrics:gateway:1.0-SNAPSHOT* .
1. Dans la zone de texte **Subscription** (Abonnement), vérifiez votre abonnement.
1. Dans la zone de texte **Spring Cloud** , sélectionnez l’instance Azure Spring Cloud que vous avez créée dans [Provisionner une instance Azure Spring Cloud](./spring-cloud-quickstart-provision-service-instance.md).
1. Définissez **Public Endpoint** (Point de terminaison public) sur *Enable* (Activer).
1. Dans la zone de texte **App:** , sélectionnez **Create app...** .
1. Entrez *gateway* , puis cliquez sur **OK** .

    ![Déployer sur Azure OK](media/spring-cloud-intellij-howto/revision-deploy-to-azure-2.png)

1. Dans la section **Before launch** (Avant le lancement) de la boîte de dialogue, double-cliquez sur *Run Maven Goal* (Exécuter Maven Goal).
1. Dans la zone de texte **Working directory** (Répertoire de travail), accédez au dossier *piggymetrics/gateway* .
1. Dans zone de texte **Command line** (Ligne de commande), entrez *package -DskipTests* . Cliquez sur **OK** .
1. Démarrez le déploiement en cliquant sur le bouton **Run** (Exécuter) en bas de la boîte de dialogue **Deploy Azure Spring Cloud app** (Déployer l’application Azure Spring Cloud). Le plug-in va exécuter la commande `mvn package` sur l’application `gateway`, puis va déployer le fichier jar généré par la commande `package`.

### <a name="deploy-auth-service-and-account-service-apps-to-azure-spring-cloud"></a>Déployer les applications auth-service et account-service sur Azure Spring Cloud
Vous pouvez répéter les étapes ci-dessus pour déployer les applications `auth-service` et `account-service` sur Azure Spring Cloud :

1. Modifiez **Name** et **Artifact** de façon à identifier l’application `auth-service`.
1. Dans la zone de texte **App:** , sélectionnez **Create app...** pour créer l’application `auth-service`.
1. Vérifiez que l’option **Public Endpoint** est définie sur *Disabled* (Désactivé).
1. Dans la section **Before launch** de la boîte de dialogue, faites passer **Working directory** sur le dossier *piggymetrics/auth-service* .
1. Démarrez le déploiement en cliquant sur le bouton **Run** (Exécuter) en bas de la boîte de dialogue **Deploy Azure Spring Cloud app** (Déployer l’application Azure Spring Cloud). 
1. Répétez ces procédures pour configurer et déployer `account-service`.
---

Accédez à l’URL fournie dans la sortie des étapes précédentes pour accéder à l’application PiggyMetrics. Par exemple : `https://<service instance name>-gateway.azuremicroservices.io`

![Accéder à PiggyMetrics](media/spring-cloud-quickstart-launch-app-cli/launch-app.png)

Vous pouvez également accéder au Portail Azure pour trouver l’URL. 
1. Accéder au service
2. Sélectionner **Applications**
3. Sélectionner **passerelle**

    ![Naviguer dans l’application](media/spring-cloud-quickstart-launch-app-cli/navigate-app1.png)
    
4. Recherchez l’URL dans la page **gateway | Vue d’ensemble**

    ![Naviguer dans la deuxième application](media/spring-cloud-quickstart-launch-app-cli/navigate-app2-url.png)

::: zone-end

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé des ressources Azure qui continueront de générer des frais tant qu’elles resteront dans votre abonnement. Si vous n’avez pas prévu de passer au guide de démarrage rapide suivant, consultez [Nettoyer les ressources](spring-cloud-quickstart-logs-metrics-tracing.md#clean-up-resources). Dans le cas contraire, passez au guide de démarrage rapide suivant :

> [!div class="nextstepaction"]
> [Journaux, métriques et suivi](spring-cloud-quickstart-logs-metrics-tracing.md)
