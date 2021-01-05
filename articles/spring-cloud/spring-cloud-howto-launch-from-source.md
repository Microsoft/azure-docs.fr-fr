---
title: Guide pratique pour lancer votre application Spring Cloud à partir du code source
description: Dans ce guide de démarrage rapide, vous découvrirez comment lancer votre application Azure Spring Cloud directement à partir de votre code source.
author: MikeDodaro
ms.service: spring-cloud
ms.topic: quickstart
ms.date: 09/03/2020
ms.author: brendm
ms.custom: devx-track-java, devx-track-azurecli
ms.openlocfilehash: d6c9bea3d0ff95a3cb9d8fcce41eaa6f8bbfed21
ms.sourcegitcommit: 30505c01d43ef71dac08138a960903c2b53f2499
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/15/2020
ms.locfileid: "92093937"
---
# <a name="how-to-launch-your-spring-cloud-application-from-source-code"></a>Guide pratique pour lancer votre application Spring Cloud à partir du code source

**Cet article s’applique à :** ✔️ Java

Azure Spring Cloud permet d’exécuter dans Azure des applications de microservices basées sur Spring Cloud.

Vous pouvez lancer des applications directement à partir du code source Java ou d’un fichier JAR prédéfini. Cet article explique les procédures de déploiement.

Ce guide de démarrage rapide explique comment :

> [!div class="checklist"]
> * Provisionner une instance de service
> * Définir un serveur de configuration pour une instance
> * Générer localement une application de microservices
> * Déployer chaque microservice
> * Affecter un point de terminaison public pour votre application

## <a name="prerequisites"></a>Prérequis
Avant de commencer, vérifiez que votre abonnement Azure a les dépendances requises :

1. [Installez Git](https://git-scm.com/)
2. [Installez JDK 8](https://www.oracle.com/technetwork/java/javase/downloads/jdk8-downloads-2133151.html)
3. [Installez Maven 3.0 ou ultérieur](https://maven.apache.org/download.cgi)
4. [Installer l’interface de ligne de commande Microsoft Azure](/cli/azure/install-azure-cli?view=azure-cli-latest)
5. [Souscrire à un abonnement Azure](https://azure.microsoft.com/free/)

> [!TIP]
> Azure Cloud Shell est un interpréteur de commandes interactif et gratuit que vous pouvez utiliser pour exécuter les étapes de cet article.  Les outils Azure les plus courants sont préinstallés, notamment les dernières versions de Git, JDK, Maven et Azure CLI. Si vous êtes connecté à votre abonnement Azure, lancez [Azure Cloud Shell](https://shell.azure.com) à partir de shell.azure.com.  Pour en savoir plus sur Azure Cloud Shell, consultez [notre documentation](../cloud-shell/overview.md).

## <a name="install-the-azure-cli-extension"></a>Installer l’extension Azure CLI

Installez l’extension Azure Spring Cloud pour Azure CLI avec la commande suivante.

```azurecli
az extension add --name spring-cloud
```

## <a name="provision-a-service-instance-using-the-azure-cli"></a>Provisionner une instance de service à l’aide d’Azure CLI

Connectez-vous à Azure CLI et choisissez votre abonnement actif. 

```azurecli
az login
az account list -o table
az account set --subscription
```

Créez un groupe de ressources destiné à contenir votre service Azure Spring Cloud. Vous pouvez en apprendre davantage sur les [groupes de ressources Azure](../azure-resource-manager/management/overview.md).

```azurecli
az group create --location eastus --name <resource group name>
```

Exécutez les commandes suivantes pour provisionner une instance d’Azure Spring Cloud. Préparez un nom pour votre service Azure Spring Cloud. Le nom doit comporter entre 4 et 32 caractères, et contenir uniquement des lettres minuscules, des chiffres et des traits d’union. Le premier caractère du nom du service doit être une lettre, et le dernier doit être une lettre ou un chiffre.

```azurecli
az spring-cloud create -n <resource name> -g <resource group name>
```

Le déploiement de l’instance de service prend environ cinq minutes.

Définissez le nom du groupe de ressources et le nom de l’instance Azure Spring Cloud par défaut en exécutant les commandes suivantes :

```azurecli
az configure --defaults group=<service group name>
az configure --defaults spring-cloud=<service instance name>
```

## <a name="create-the-azure-spring-cloud-application"></a>Créer l’application Azure Spring Cloud

La commande suivante crée une application Azure Spring Cloud dans votre abonnement.  Elle crée un service vide dans lequel nous pouvons charger notre application.

```azurecli
az spring-cloud app create -n <app-name>
```

## <a name="deploy-your-spring-cloud-application"></a>Déployer votre application Spring Cloud

Vous pouvez déployer votre application à partir d’un fichier JAR prédéfini ou d’un référentiel Gradle ou Maven.  Vous trouverez ci-dessous des instructions pour chaque cas.

### <a name="deploy-a-built-jar"></a>Déployer un fichier JAR généré

Pour effectuer un déploiement à partir d’un fichier JAR généré sur votre ordinateur local, vérifiez que votre build produit un fichier [fat-JAR](https://docs.spring.io/spring-boot/docs/current/reference/html/howto-build.html#howto-create-an-executable-jar-with-maven).

Pour déployer le fichier fat-JAR dans un déploiement actif :

```azurecli
az spring-cloud app deploy -n <app-name> --jar-path <path-to-fat-JAR e.g. "target\hellospring-0.0.1-SNAPSHOT.jar">
```

Pour déployer le fichier fat-JAR dans un déploiement spécifique :

```azurecli
az spring-cloud app deployment create --app <app-name> -n <deployment-name> --jar-path <path-to-fat-JAR e.g. "target\hellospring-0.0.1-SNAPSHOT.jar">
```

### <a name="deploy-from-source-code"></a>Déployer à partir de code source

Azure Spring Cloud utilise [kpack](https://github.com/pivotal/kpack) pour générer votre projet.  Vous pouvez utiliser Azure CLI pour charger votre code source, générer votre projet à l’aide de kpack, et le déployer dans l’application cible.

> [!WARNING]
> Le projet ne doit produire qu’un seul fichier JAR avec une entrée `main-class` dans le `MANIFEST.MF` de `target` (pour les déploiements Maven) ou `build/libs` (pour les déploiements Gradle).  La présence de plusieurs fichiers JAR avec des entrées `main-class` entraînent l’échec du déploiement.

Pour les projets Maven/Gradle à un seul module :

```azurecli
cd <path-to-maven-or-gradle-source-root>
az spring-cloud app deploy -n <app-name>
```

Pour les projets Maven/Gradle avec plusieurs modules, répétez cette opération pour chaque module :

```azurecli
cd <path-to-maven-or-gradle-source-root>
az spring-cloud app deploy -n <app-name> --target-module <relative-path-to-module>
```

### <a name="show-deployment-logs"></a>Afficher les journaux de déploiement

Passez en revue les journaux de génération kpack à l’aide de la commande suivante :

```azurecli
az spring-cloud app show-deploy-log -n <app-name> [-d <deployment-name>]
```

> [!NOTE]
> Les journaux kpack affichent uniquement le dernier déploiement si celui-ci a été généré à partir de la source à l’aide de kpack.

## <a name="assign-a-public-endpoint-to-gateway"></a>Affecter un point de terminaison public à la passerelle

1. Ouvrez la page **Tableau de bord de l’application**.
2. Sélectionnez l’application `gateway` pour afficher la page **Détails de l’application**.
3. Sélectionnez **Affecter un point de terminaison** pour affecter un point de terminaison public à la passerelle. Cette opération peut prendre quelques minutes. 
4. Entrez l’adresse IP publique affectée dans votre navigateur pour voir votre application en cours d’exécution.

> [!div class="nextstepaction"]
> [J’ai rencontré un problème](https://www.research.net/r/javae2e?tutorial=asc-source-quickstart&step=public-endpoint)

## <a name="next-steps"></a>Étapes suivantes

Dans ce démarrage rapide, vous avez appris comment :

> [!div class="checklist"]
> * Provisionner une instance de service
> * Définir un serveur de configuration pour une instance
> * Générer localement une application de microservices
> * Déployer chaque microservice
> * Modifier les variables d’environnement pour les applications
> * Affecter une adresse IP publique à votre passerelle d’application

> [!div class="nextstepaction"]
> [Journaux, métriques et suivi Spring Cloud](spring-cloud-quickstart-logs-metrics-tracing.md)

D’autres exemples sont disponibles sur GitHub : [Exemples Azure Spring Cloud](https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples/tree/master/service-binding-cosmosdb-sql).