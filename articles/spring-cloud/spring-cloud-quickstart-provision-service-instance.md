---
title: Démarrage rapide - Provisionner le service Azure Spring Cloud
description: Décrit la création d’une instance d’Azure Spring Cloud pour le déploiement d’applications.
author: MikeDodaro
ms.author: brendm
ms.service: spring-cloud
ms.topic: quickstart
ms.date: 09/08/2020
ms.custom: devx-track-java, devx-track-azurecli
zone_pivot_groups: programming-languages-spring-cloud
ms.openlocfilehash: 22abdb64a2349045923e148db57f683b4be87636
ms.sourcegitcommit: 8c7f47cc301ca07e7901d95b5fb81f08e6577550
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92742694"
---
# <a name="quickstart-provision-azure-spring-cloud-service"></a>Démarrage rapide : Provisionner le service Azure Spring Cloud

::: zone pivot="programming-language-csharp"
Dans ce guide de démarrage rapide, vous allez utiliser Azure CLI pour provisionner une instance du service Azure Spring Cloud.

## <a name="prerequisites"></a>Prérequis

* Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
* [Kit SDK .NET Core 3.1](https://dotnet.microsoft.com/download/dotnet-core/3.1). Le service Azure Spring Cloud prend en charge .NET Core 3.1 et versions ultérieures.
* [Azure CLI version 2.0.67 ou ultérieure](/cli/azure/install-azure-cli?preserve-view=true&view=azure-cli-latest).
* [Git](https://git-scm.com/).

## <a name="install-azure-cli-extension"></a>Installer l’extension Azure CLI

Vérifiez également que votre version d’Azure CLI est égale ou supérieure à la version 2.0.67 :

```azurecli
az --version
```

Installez l’extension Azure Spring Cloud pour Azure CLI avec la commande suivante :

```azurecli
az extension add --name spring-cloud
```

## <a name="log-in-to-azure"></a>Connexion à Azure

1. Connectez-vous à Azure CLI.

    ```azurecli
    az login
    ```

1. Si vous avez plusieurs abonnements, choisissez celui que vous souhaitez utiliser pour ce guide de démarrage rapide.

   ```azurecli
   az account list -o table
   ```

   ```azurecli
   az account set --subscription <Name or ID of a subscription from the last step>
   ```

## <a name="provision-an-instance-of-azure-spring-cloud"></a>Provisionner une instance d’Azure Spring Cloud

1. Créez un [groupe de ressources](../azure-resource-manager/management/overview.md) destiné à contenir votre service Azure Spring Cloud. Le nom du groupe de ressources peut comprendre des caractères alphanumériques, des traits de soulignement, des parenthèses, des traits d’union, des points (sauf à la fin) et des caractères Unicode.

   ```azurecli
   az group create --location eastus --name <resource group name>
   ```

1. Provisionnez une instance du service Azure Spring Cloud. Le nom de l’instance de service doit être unique, comporter entre 4 et 32 caractères, et contenir uniquement des lettres minuscules, des chiffres et des traits d’union. Le premier caractère du nom du service doit être une lettre, et le dernier doit être une lettre ou un chiffre.

    ```azurecli
    az spring-cloud create -n <service instance name> -g <resource group name>
    ```

    L’exécution de cette commande peut prendre plusieurs minutes.

1. Définissez le nom du groupe de ressources et le nom de l’instance de service par défaut afin de ne pas avoir à spécifier ces valeurs à chaque fois dans les commandes suivantes.

   ```azurecli
   az configure --defaults group=<resource group name>
   ```

   ```azurecli
   az configure --defaults spring-cloud=<service instance name>
   ```
::: zone-end

::: zone pivot="programming-language-java"
Vous pouvez instancier Azure Spring Cloud en utilisant le portail Azure ou Azure CLI.  Les deux méthodes sont expliquées dans les procédures suivantes.
## <a name="prerequisites"></a>Prérequis

* [Installez JDK 8](/java/azure/jdk/?preserve-view=true&view=azure-java-stable)
* [Souscrire à un abonnement Azure](https://azure.microsoft.com/free/)
* (Facultatif) [Installez Azure CLI version 2.0.67 ou ultérieure](/cli/azure/install-azure-cli?preserve-view=true&view=azure-cli-latest) et installez l’extension Azure Spring Cloud avec la commande : `az extension add --name spring-cloud`
* (Facultatif) [Installez Azure Toolkit for IntelliJ](https://plugins.jetbrains.com/plugin/8053-azure-toolkit-for-intellij/) et [connectez-vous](/azure/developer/java/toolkit-for-intellij/create-hello-world-web-app#installation-and-sign-in).

## <a name="provision-an-instance-of-azure-spring-cloud"></a>Provisionner une instance d’Azure Spring Cloud

#### <a name="portal"></a>[Portail](#tab/Azure-portal)

La procédure suivante crée une instance d’Azure Spring Cloud en utilisant le portail Azure.

1. Sous un nouvel onglet, ouvrez le [portail Azure](https://ms.portal.azure.com/). 

2. Dans la zone de recherche supérieure, recherchez **Azure Spring Cloud** .

3. Sélectionnez **Azure Spring Cloud** dans les résultats.

    ![ASC - Icône Démarrer](media/spring-cloud-quickstart-launch-app-portal/find-spring-cloud-start.png)

4. Dans la page Azure Spring Cloud, cliquez sur **+ Ajouter** .

    ![ASC - Icône Ajouter](media/spring-cloud-quickstart-launch-app-portal/spring-cloud-add.png)

5. Remplissez le formulaire sur la page **Créer** Azure Spring Cloud.  Tenez compte des recommandations suivantes :
    - **Abonnement** : Sélectionnez l’abonnement à facturer pour cette ressource.
    - **Groupe de ressources**  : Une bonne pratique consiste à créer des groupes de ressources pour les nouvelles ressources. Notez que ceci sera utilisé dans des étapes ultérieures en tant que **\<resource group name\>** .
    - **Détails/Nom du service**  : Spécifiez **\<service instance name\>** .  Le nom doit comporter entre 4 et 32 caractères, et contenir uniquement des lettres minuscules, des chiffres et des traits d’union.  Le premier caractère du nom du service doit être une lettre, et le dernier doit être une lettre ou un chiffre.
    - **Emplacement** : Sélectionnez l’emplacement de votre instance de service.

    ![Démarrage du portail ASC](media/spring-cloud-quickstart-launch-app-portal/portal-start.png)

6. Cliquez sur **Vérifier et créer** .

> [!div class="nextstepaction"]
> [J’ai rencontré un problème](https://www.research.net/r/javae2e?tutorial=asc-cli-quickstart&step=public-endpoint)

#### <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/Azure-CLI)

La procédure suivante utilise l’extension Azure CLI pour provisionner une instance d’Azure Spring Cloud.

1. Connectez-vous à Azure CLI et choisissez votre abonnement actif.

    ```azurecli
    az login
    az account list -o table
    az account set --subscription <Name or ID of subscription, skip if you only have 1 subscription>
    ```

1. Préparez un nom pour votre service Azure Spring Cloud.  Le nom doit comporter entre 4 et 32 caractères, et contenir uniquement des lettres minuscules, des chiffres et des traits d’union.  Le premier caractère du nom du service doit être une lettre, et le dernier doit être une lettre ou un chiffre.

1. Créez un groupe de ressources destiné à contenir votre service Azure Spring Cloud.

    ```azurecli
    az group create --location eastus --name <resource group name>
    ```

    Obtenez plus d’informations sur les [groupes de ressources Azure](../azure-resource-manager/management/overview.md).

1. Ouvrez une fenêtre Azure CLI et exécutez les commandes suivantes pour provisionner une instance d’Azure Spring Cloud.

    ```azurecli
    az spring-cloud create -n <service instance name> -g <resource group name>
    ```

    Le déploiement de l’instance de service prend environ cinq minutes.
---
::: zone-end

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez créé des ressources Azure qui continueront de générer des frais tant qu’elles resteront dans votre abonnement. Si vous n’avez pas prévu de passer au guide de démarrage rapide suivant, consultez [Nettoyer les ressources](spring-cloud-quickstart-logs-metrics-tracing.md#clean-up-resources). Dans le cas contraire, passez au guide de démarrage rapide suivant :

> [!div class="nextstepaction"]
> [Configurer un serveur de configuration](spring-cloud-quickstart-setup-config-server.md)
