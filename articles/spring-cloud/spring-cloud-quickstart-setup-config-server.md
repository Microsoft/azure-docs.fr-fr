---
title: Démarrage rapide - Configurer un serveur de configuration Azure Spring Cloud
description: Décrit la configuration du serveur de configuration Azure Spring Cloud pour le déploiement d’applications.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: quickstart
ms.date: 09/08/2020
ms.custom: devx-track-java
zone_pivot_groups: programming-languages-spring-cloud
ms.openlocfilehash: b23b5d516479cb28948a601c6bd71309500e73f1
ms.sourcegitcommit: e7179fa4708c3af01f9246b5c99ab87a6f0df11c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/30/2020
ms.locfileid: "97825781"
---
# <a name="quickstart-set-up-azure-spring-cloud-configuration-server"></a>Démarrage rapide : Configurer un serveur de configuration Azure Spring Cloud

Le serveur de configuration Azure Spring Cloud est un service de configuration centralisé pour les systèmes distribués. Il utilise une couche de dépôt enfichable qui prend actuellement en charge le stockage local, Git et Subversion. Dans ce guide de démarrage rapide, vous allez configurer le serveur de configuration pour obtenir des données à partir d’un dépôt Git.

::: zone pivot="programming-language-csharp"

## <a name="prerequisites"></a>Prérequis

* Suivez ce guide de démarrage rapide précédent de la série : [Provisionner le service Azure Spring Cloud](spring-cloud-quickstart-provision-service-instance.md).

## <a name="azure-spring-cloud-config-server-procedures"></a>Procédures pour le serveur de configuration Azure Spring Cloud

Configurez votre serveur de configuration avec l’emplacement du dépôt Git du projet en exécutant la commande suivante. Remplacez `<service instance name>` par le nom du service que vous avez créé. La valeur par défaut du nom de l’instance de service que vous avez définie dans le guide de démarrage rapide précédent ne fonctionne pas avec cette commande.

```azurecli
az spring-cloud config-server git set -n <service instance name> --uri https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples --search-paths steeltoe-sample/config
```

Cette commande demande au serveur de configuration de trouver les données de configuration dans le dossier [steeltoe-sample/config](https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples/tree/master/steeltoe-sample/config) du dépôt de l’exemple d’application. Étant donné que le nom de l’application qui obtiendra les données de configuration est `planet-weather-provider`, le fichier qui sera utilisé est [planet-weather-provider.yml](https://github.com/Azure-Samples/Azure-Spring-Cloud-Samples/blob/master/steeltoe-sample/config/planet-weather-provider.yml).

::: zone-end

::: zone pivot="programming-language-java"
Le serveur de configuration Azure Spring Cloud est un service de configuration centralisé pour les systèmes distribués. Il utilise une couche de dépôt enfichable qui prend actuellement en charge le stockage local, Git et Subversion.  Configurez le serveur de configuration pour déployer des applications de microservice sur Azure Spring Cloud.

## <a name="prerequisites"></a>Prérequis

* [Installez JDK 8](/java/azure/jdk/?preserve-view=true&view=azure-java-stable)
* [Souscrire à un abonnement Azure](https://azure.microsoft.com/free/)
* (Facultatif) [Installez Azure CLI version 2.0.67 ou ultérieure](/cli/azure/install-azure-cli?preserve-view=true&view=azure-cli-latest) et installez l’extension Azure Spring Cloud avec la commande : `az extension add --name spring-cloud`
* (Facultatif) [Installez Azure Toolkit for IntelliJ](https://plugins.jetbrains.com/plugin/8053-azure-toolkit-for-intellij/) et [connectez-vous](/azure/developer/java/toolkit-for-intellij/create-hello-world-web-app#installation-and-sign-in).

## <a name="azure-spring-cloud-config-server-procedures"></a>Procédures pour le serveur de configuration Azure Spring Cloud

#### <a name="portal"></a>[Portail](#tab/Azure-portal)

La procédure suivante permet de configurer le serveur de configuration en utilisant le portail Azure pour déployer l’[exemple Piggymetrics](spring-cloud-quickstart-sample-app-introduction.md).

1. Accédez à la page **Vue d’ensemble** du service, puis sélectionnez **Serveur de configuration**.

2. Dans la section **Dépôt par défaut**, définissez **URI** sur « https://github.com/Azure-Samples/piggymetrics-config  ».

3. Cliquez sur **Valider**.

    ![Accéder au serveur de configuration](media/spring-cloud-quickstart-launch-app-portal/portal-config.png)

4. Une fois la validation terminée, cliquez sur **Appliquer** pour enregistrer vos modifications.

    ![Validation du serveur de configuration](media/spring-cloud-quickstart-launch-app-portal/validate-complete.png)

5. La mise à jour de la configuration peut prendre quelques minutes.
 
    ![Mise à jour du serveur de configuration](media/spring-cloud-quickstart-launch-app-portal/updating-config.png) 

6. Une fois la configuration terminée, vous devriez recevoir une notification.

#### <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/Azure-CLI)

La procédure suivante utilise Azure CLI pour configurer le serveur de configuration afin de déployer l’[exemple Piggymetrics](spring-cloud-quickstart-sample-app-introduction.md).

Configurez votre serveur de configuration avec l’emplacement du dépôt Git pour le projet :

```azurecli
az spring-cloud config-server git set -n <service instance name> --uri https://github.com/Azure-Samples/piggymetrics-config
```
---
::: zone-end

> [!TIP]
> Si vous utilisez un dépôt privé pour le serveur de configuration, reportez-vous à notre [tutoriel sur la configuration de l’authentification](https://docs.microsoft.com/azure/spring-cloud/spring-cloud-tutorial-config-server).

## <a name="troubleshooting-of-azure-spring-cloud-config-server"></a>Résolution des problèmes de Config Server d’Azure Spring Cloud

La procédure suivante explique comment résoudre les problèmes liés aux paramètres de Config Server.

1. Dans le portail Azure, accédez à la page **Vue d’ensemble** du service, puis sélectionnez **Journaux**. 
1. Sélectionnez **Requêtes** et **Afficher les journaux des applications qui contiennent les termes « erreur » ou « exceptions »** . 
1. Cliquez sur **Exécuter**. 
1. Si vous trouvez l’erreur **java.lang.illegalStateException** dans les journaux, cela indique que le service Spring Cloud ne peut pas localiser les propriétés provenant de Config Server.

    [ ![Portail ASC, exécuter une requête](media/spring-cloud-quickstart-setup-config-server/setup-config-server-query.png) ](media/spring-cloud-quickstart-setup-config-server/setup-config-server-query.png)

1. Allez à la page **Vue d’ensemble** du service.
1. Sélectionnez **Diagnostiquer et résoudre les problèmes**. 
1. Sélectionnez le détecteur **Config Server**.

    [ ![Portail ASC, diagnostiquer les problèmes](media/spring-cloud-quickstart-setup-config-server/setup-config-server-diagnose.png) ](media/spring-cloud-quickstart-setup-config-server/setup-config-server-diagnose.png)

3. Cliquez sur **Contrôle d’intégrité de Config Server**.

    [ ![Portail ASC, Genie](media/spring-cloud-quickstart-setup-config-server/setup-config-server-genie.png) ](media/spring-cloud-quickstart-setup-config-server/setup-config-server-genie.png)

4. Cliquez sur **État de Config Server** pour afficher plus de détails à partir du détecteur.

    [ ![Portail ASC, état d’intégrité](media/spring-cloud-quickstart-setup-config-server/setup-config-server-health-status.png) ](media/spring-cloud-quickstart-setup-config-server/setup-config-server-health-status.png)

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé des ressources Azure qui continueront de générer des frais tant qu’elles resteront dans votre abonnement. Si vous n’avez pas prévu de passer au guide de démarrage rapide suivant, consultez [Nettoyer les ressources](spring-cloud-quickstart-logs-metrics-tracing.md#clean-up-resources). Dans le cas contraire, passez au guide de démarrage rapide suivant :

> [!div class="nextstepaction"]
> [Créer et déployer des applications](spring-cloud-quickstart-deploy-apps.md)
