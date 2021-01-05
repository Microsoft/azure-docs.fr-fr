---
title: 'Démarrage rapide : Générer des images conteneurs Java et les envoyer (push) à Azure Container Registry à l’aide de Maven et Jib'
description: Générez une application Java conteneurisée et envoyez-la à Azure Container Registry à l’aide du plug-in Maven Jib.
author: KarlErickson
ms.custom: devx-track-java, devx-track-azurecli
ms.author: karler
ms.topic: quickstart
ms.date: 02/26/2020
ms.openlocfilehash: 9e400ee0bae2690a84f9cfd3f6a76359e08eabc2
ms.sourcegitcommit: 2c586a0fbec6968205f3dc2af20e89e01f1b74b5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92018336"
---
# <a name="quickstart-build-and-push-java-container-images-to-azure-container-registry"></a>Démarrage rapide : Générer des images conteneurs Java et les envoyer (push) à Azure Container Registry

Ce guide de démarrage rapide vous montre comment générer une application Java conteneurisée et comment l’envoyer (push) à Azure Container Registry à l’aide du plug-in Maven Jib. L’emploi de Maven et de Jib est un exemple d’utilisation des outils de développement pour interagir avec un registre de conteneurs Azure.

## <a name="prerequisites"></a>Prérequis

* Un abonnement Azure. Si vous n’avez pas déjà un abonnement Azure, vous pouvez activer vos [avantages d’abonné MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details) ou vous inscrire pour un [compte Azure gratuit](https://azure.microsoft.com/pricing/free-trial).
* [Azure CLI](/cli/azure/overview).
* Un kit de développement Java (JDK) pris en charge. Pour en savoir plus sur les kits de développement disponibles pour le développement sur Azure, consultez <https://aka.ms/azure-jdks>.
* L’outil de génération [Maven](http://maven.apache.org) (version 3 ou ultérieure) d’Apache.
* Un client [Git](https://git-scm.com).
* Un client [Docker](https://www.docker.com).
* L’[assistance des informations d’identification ACR Docker](https://github.com/Azure/acr-docker-credential-helper).

## <a name="create-the-spring-boot-on-docker-getting-started-web-app"></a>Créer l’application web Spring Boot on Docker Getting Started

Les étapes suivantes vous guident à travers la création d’une application web Spring Boot et vous aident à effectuer le test en local.

1. À partir de l’invite de commandes, utilisez la commande suivante pour cloner l’exemple de projet [Spring Boot du Guide de prise en main Docker](https://github.com/spring-guides/gs-spring-boot-docker).

   ```bash
   git clone https://github.com/spring-guides/gs-spring-boot-docker.git
   ```

1. Accédez au répertoire du projet terminé.

   ```bash
   cd gs-spring-boot-docker/complete
   ```

1. Utilisez Maven pour créer et exécuter l’exemple d’application.

   ```bash
   mvn package spring-boot:run
   ```

1. Testez l’application web en accédant à l’URL `http://localhost:8080`, ou avec la commande `curl` suivante :

   ```bash
   curl http://localhost:8080
   ```

Vous devriez voir le message suivant : **Hello Docker World**

## <a name="create-an-azure-container-registry-using-the-azure-cli"></a>Créer un registre de conteneurs Azure à l’aide de l’interface de ligne de commande Azure

Ensuite, vous créez un groupe de ressources Azure et votre ACR en procédant comme suit :

1. Connectez-vous à votre compte Azure à l’aide de la commande suivante :

   ```azurecli
   az login
   ```

1. Spécifiez l’abonnement Azure à utiliser :

   ```azurecli
   az account set -s <subscription ID>
   ```

1. Créer un groupe de ressources pour les ressources Azure utilisées dans ce didacticiel. Dans la commande suivante, remplacez les espaces réservés par le nom de votre ressource et par un emplacement tel que `eastus`.

   ```azurecli
   az group create \
       --name=<your resource group name> \
       --location=<location>
   ```

1. Créez un registre de conteneurs Azure privé dans le groupe de ressources, à l’aide de la commande suivante. N’oubliez pas de remplacer les espaces réservés par les valeurs réelles. Au cours des dernières étapes, le didacticiel envoie au registre l’exemple d’application en tant qu’image Docker.

   ```azurecli
   az acr create \
       --resource-group <your resource group name> \
       --location <location> \
       --name <your registry name> \
       --sku Basic
   ```

## <a name="push-your-app-to-the-container-registry-via-jib"></a>Envoyer l’application dans le registre de conteneurs via Jib

Enfin, vous mettez à jour la configuration de votre projet et vous utilisez l’invite de commandes pour générer et déployer votre image.

> [!NOTE]
> Pour vous connecter au registre de conteneurs Azure que vous venez de créer, vous devez exécuter le démon Docker. Pour installer Docker sur votre ordinateur, consultez la [documentation officielle de Docker](https://docs.docker.com/install/).

1. Connectez-vous à votre instance Azure Container Registry depuis Azure CLI à l’aide de la commande suivante. N’oubliez pas de remplacer l’espace réservé par le nom de votre registre.

   ```azurecli
   az configure --defaults acr=<your registry name>
   az acr login
   ```

   La commande `az configure` définit le nom de registre par défaut à utiliser avec les commandes `az acr`.

1. Accédez au répertoire de projet terminé de votre application Spring Boot (par exemple, « *C:\SpringBoot\gs-spring-boot-docker\complete* » ou «  */users/robert/SpringBoot/gs-spring-boot-docker/complete* ») et ouvrez le fichier *pom.xml* avec un éditeur de texte.

1. Mettez à jour la collection `<properties>` dans le fichier *pom.xml* avec le code XML suivant. Remplacez l’espace réservé par le nom de votre registre et ajoutez une propriété `<jib-maven-plugin.version>` avec la valeur `2.2.0` ou une version plus récente du [plug-in Maven Jib](https://github.com/GoogleContainerTools/jib/tree/master/jib-maven-plugin).

   ```xml
   <properties>
      <docker.image.prefix><your registry name>.azurecr.io</docker.image.prefix>
      <java.version>1.8</java.version>
      <jib-maven-plugin.version>2.2.0</jib-maven-plugin.version>
   </properties>
   ```

1. Mettez à jour la collection `<plugins>` dans le fichier *pom.xml* afin que l’élément `<plugin>` contienne une entrée pour `jib-maven-plugin`, comme indiqué dans l’exemple ci-dessous. Notez que nous utilisons une image de base provenant du Registre de conteneurs Microsoft, `mcr.microsoft.com/java/jdk:8-zulu-alpine`, qui contient un JDK officiellement pris en charge pour Azure. Pour obtenir d’autres images de base de ce Registre avec les JDK officiellement pris en charge, consultez [Java SE JDK](https://hub.docker.com/_/microsoft-java-jdk), [Java SE JRE](https://hub.docker.com/_/microsoft-java-jre), [Java SE Headless JRE](https://hub.docker.com/_/microsoft-java-jre-headless) et [Java SE JDK and Maven](https://hub.docker.com/_/microsoft-java-maven).

   ```xml
   <plugin>
     <artifactId>jib-maven-plugin</artifactId>
     <groupId>com.google.cloud.tools</groupId>
     <version>${jib-maven-plugin.version}</version>
     <configuration>
        <from>
            <image>mcr.microsoft.com/java/jdk:8-zulu-alpine</image>
        </from>
        <to>
            <image>${docker.image.prefix}/${project.artifactId}</image>
        </to>
     </configuration>
   </plugin>
   ```

1. Pour créer l’image Docker et la pousser dans le registre, accédez au répertoire de projet terminé de votre application Spring Boot et exécutez la commande suivante :

   ```bash
   az acr login && mvn compile jib:build
   ```

> [!NOTE]
>
> Pour des raisons de sécurité, les informations d’identification créées par `az acr login` sont valides pendant une heure seulement. Si vous recevez une erreur *401 Non autorisé*, vous pouvez réexécuter la commande `az acr login -n <your registry name>` pour effectuer une nouvelle authentification.

## <a name="verify-your-container-image"></a>Vérifier votre image conteneur

Félicitations ! Vous avez généré votre application Java conteneurisée dans le JDK pris en charge pour Azure et l’avez envoyée à votre instance Azure Container Registry. Vous pouvez maintenant tester l’image en la déployant sur Azure App Service ou en la tirant (pull) localement avec la commande (en remplaçant l’espace réservé par la valeur réelle) :

```bash
docker pull <your registry name>.azurecr.io/gs-spring-boot-docker
```

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir d’autres versions des images de base Java officiellement prises en charge par Microsoft, consultez :

* [Java SE JDK](https://hub.docker.com/_/microsoft-java-jdk)
* [Java SE JRE](https://hub.docker.com/_/microsoft-java-jre)
* [Java SE Headless JRE](https://hub.docker.com/_/microsoft-java-jre-headless)
* [Java SE JDK and Maven](https://hub.docker.com/_/microsoft-java-maven)

Pour en savoir plus sur Spring et Azure, poursuivez vers le centre de documentation Spring sur Azure.

> [!div class="nextstepaction"]
> [Spring sur Azure](/azure/developer/java/spring-framework)

### <a name="additional-resources"></a>Ressources supplémentaires

Pour plus d’informations, consultez les ressources suivantes :

* [Azure pour les développeurs Java](/azure/java)
* [Utilisation d’Azure DevOps et Java](/azure/devops/java)
* [Spring Boot on Docker Getting Started](https://spring.io/guides/gs/spring-boot-docker)
* [Spring Initializr](https://start.spring.io)
* [Déployer une application Spring Boot sur Azure App Service](/azure/developer/java/spring-framework/deploy-spring-boot-java-app-on-linux#configure-maven-to-build-image-to-your-azure-container-registry)
* [Comment utiliser une image Docker personnalisée pour Azure Web App sur Linux](../app-service/tutorial-custom-container.md)
