---
title: Créer une application App Service avec un modèle Azure Resource Manager
description: Créez votre première application pour Azure App Service en quelques secondes en utilisant un modèle Azure Resource Manager (modèle ARM), qui constitue l’un des nombreux moyens d’effectuer un déploiement sur App Service.
author: msangapu-msft
ms.author: msangapu
ms.assetid: 582bb3c2-164b-42f5-b081-95bfcb7a502a
ms.topic: quickstart
ms.date: 10/16/2020
ms.custom: subject-armqs, devx-track-azurecli
zone_pivot_groups: app-service-platform-windows-linux
ms.openlocfilehash: 5be4cea7d8ddb00a4c8a22a696dbdce1c062765c
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96183020"
---
# <a name="quickstart-create-app-service-app-using-an-arm-template"></a>Démarrage rapide : Créer une application App Service à l’aide d’un modèle ARM

Familiarisez-vous avec [Azure App Service](overview.md) en déployant une application sur le cloud à l’aide d’un modèle Azure Resource Manager (modèle ARM) et d’[Azure CLI](/cli/azure/get-started-with-azure-cli) dans Cloud Shell. Comme vous utilisez un niveau App Service gratuit, vous pouvez suivre ce guide de démarrage rapide gratuitement.

 [!INCLUDE [About Azure Resource Manager](../../includes/resource-manager-quickstart-introduction.md)]

Si votre environnement remplit les prérequis et que vous êtes déjà familiarisé avec l’utilisation des modèles ARM, sélectionnez le bouton **Déployer sur Azure**. Le modèle s’ouvre dans le portail Azure.

Utilisez le bouton suivant pour effectuer un déploiement sur **Linux** :

[![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-app-service-docs-linux%2Fazuredeploy.json)

Utilisez le bouton suivant pour effectuer un déploiement sur **Windows** :

[![Déployer sur Azure](../media/template-deployments/deploy-to-azure.svg)](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-app-service-docs-windows%2Fazuredeploy.json)

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="review-the-template"></a>Vérifier le modèle

::: zone pivot="platform-windows"
Le modèle utilisé dans ce démarrage rapide est tiré des [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/101-app-service-docs-windows). Il déploie un plan App Service et une application App Service sur Windows. Il est compatible avec .NET Core, .NET Framework, PHP, Node.js et les applications HTML statiques. Pour Java, consultez [Créer une application Java](./quickstart-java.md).

:::code language="json" source="~/quickstart-templates/101-app-service-docs-windows/azuredeploy.json":::

Deux ressources Azure sont définies dans le modèle :

* [**Microsoft.Web/serverfarms**](/azure/templates/microsoft.web/serverfarms) : créer un plan App Service.
* [**Microsoft.Web/sites**](/azure/templates/microsoft.web/sites) : créer une application App Service.

Ce modèle contient plusieurs paramètres prédéfinis par souci pratique. Consultez le tableau ci-dessous pour connaître les valeurs par défaut des paramètres et leurs descriptions :

| Paramètres | Type    | Valeur par défaut                | Description |
|------------|---------|------------------------------|-------------|
| webAppName | string  | "webApp- **[`<uniqueString>`](../azure-resource-manager/templates/template-functions-string.md#uniquestring)** " | Nom de l’application |
| location   | string  | "[[resourceGroup().location](../azure-resource-manager/templates/template-functions-resource.md#resourcegroup)]" | Région de l’application |
| sku        | string  | "F1"                         | Taille de l’instance (F1 = niveau gratuit) |
| langage   | string  | ".net"                       | Pile du langage de programmation (.net, php, node, html) |
| helloWorld | boolean | False                        | True = déployer l’application « Hello World » |
| repoUrl    | string  | " "                          | Dépôt Git externe (facultatif) |
::: zone-end
::: zone pivot="platform-linux"
Le modèle utilisé dans ce démarrage rapide est tiré des [modèles de démarrage rapide Azure](https://azure.microsoft.com/resources/templates/101-app-service-docs-linux). Il déploie un plan App Service et une application App Service sur Linux. Il est compatible avec tous les langages de programmation pris en charge sur App Service.

:::code language="json" source="~/quickstart-templates/101-app-service-docs-linux/azuredeploy.json":::

Deux ressources Azure sont définies dans le modèle :

* [**Microsoft.Web/serverfarms**](/azure/templates/microsoft.web/serverfarms) : créer un plan App Service.
* [**Microsoft.Web/sites**](/azure/templates/microsoft.web/sites) : créer une application App Service.

Ce modèle contient plusieurs paramètres prédéfinis par souci pratique. Consultez le tableau ci-dessous pour connaître les valeurs par défaut des paramètres et leurs descriptions :

| Paramètres | Type    | Valeur par défaut                | Description |
|------------|---------|------------------------------|-------------|
| webAppName | string  | "webApp- **[`<uniqueString>`](../azure-resource-manager/templates/template-functions-string.md#uniquestring)** " | Nom de l’application |
| location   | string  | "[[resourceGroup().location](../azure-resource-manager/templates/template-functions-resource.md#resourcegroup)]" | Région de l’application |
| sku        | string  | "F1"                         | Taille de l’instance (F1 = niveau gratuit) |
| linuxFxVersion   | string  | "DOTNETCORE&#124;3.0        | Pile du langage de programmation &#124; Version |
| repoUrl    | string  | " "                          | Dépôt Git externe (facultatif) |

---
::: zone-end

## <a name="deploy-the-template"></a>Déployer le modèle

Azure CLI est utilisé ici pour déployer le modèle. Vous pouvez également utiliser le portail Azure, Azure PowerShell ou l’API REST. Pour découvrir d’autres méthodes de déploiement, consultez [Déployer des modèles](../azure-resource-manager/templates/deploy-powershell.md).

Le code suivant crée un groupe de ressources, un plan App Service et une application web. Un groupe de ressources, un plan App Service et une localisation par défaut ont été définis pour vous. Remplacez `<app-name>` par un nom d’application global unique (les caractères valides sont `a-z`, `0-9` et `-`).

::: zone pivot="platform-windows"
Exécutez le code ci-dessous pour déployer une application .NET Framework sur Windows.

```azurecli-interactive
az group create --name myResourceGroup --location "southcentralus" &&
az deployment group create --resource-group myResourceGroup \
--parameters language=".net" helloWorld="true" webAppName="<app-name>" \
--template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-app-service-docs-windows/azuredeploy.json"
::: zone-end
::: zone pivot="platform-linux"
Run the code below to create a Python app on Linux.

```azurecli-interactive
az group create --name myResourceGroup --location "southcentralus" &&
az deployment group create --resource-group myResourceGroup --parameters webAppName="<app-name>" linuxFxVersion="PYTHON|3.7" \
--template-uri "https://raw.githubusercontent.com/Azure/azure-quickstart-templates/master/101-app-service-docs-linux/azuredeploy.json"
```

Pour déployer une autre pile de langage, mettez à jour `linuxFxVersion` avec les valeurs appropriées. Des exemples sont présentés ci-dessous. Pour afficher les versions actuelles, exécutez la commande suivante dans Cloud Shell : `az webapp config show --resource-group myResourceGroup --name <app-name> --query linuxFxVersion`

| Langage    | Exemple :                                               |
|-------------|------------------------------------------------------|
| **.NET**    | linuxFxVersion="DOTNETCORE&#124;3.0"                 |
| **PHP**     | linuxFxVersion="PHP&#124;7.4"                        |
| **Node.JS** | linuxFxVersion="NODE&#124;10.15"                     |
| **Java**    | linuxFxVersion="JAVA&#124;1.8 &#124;TOMCAT&#124;9.0" |
| **Python**  | linuxFxVersion="PYTHON&#124;3.7"                     |
| **Ruby**    | linuxFxVersion="RUBY&#124;2.6"                       |

---
::: zone-end

> [!NOTE]
> Vous trouverez d’autres exemples de [modèles Azure App Service ici](https://azure.microsoft.com/resources/templates/?resourceType=Microsoft.Sites).

## <a name="validate-the-deployment"></a>Valider le déploiement

Accédez à `http://<app_name>.azurewebsites.net/` et vérifiez qu’il a été créé.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand il n’est plus nécessaire, [supprimez le groupe de ressources](../azure-resource-manager/management/delete-resource-group.md?tabs=azure-portal#delete-resource-group).

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Déployer à partir de Git local](deploy-local-git.md)

> [!div class="nextstepaction"]
> [Build a .NET Core and SQL Database web app in Azure App Service](tutorial-dotnetcore-sqldb-app.md) (Créer une application web .NET Core et SQL Database dans Azure App Service)

> [!div class="nextstepaction"]
> Python avec Postgres

> [!div class="nextstepaction"]
> [PHP avec MySQL](tutorial-php-mysql-app.md)

> [!div class="nextstepaction"]
> [Se connecter à Azure SQL Database à l’aide de Java](../azure-sql/database/connect-query-java.md?toc=%2fazure%2fjava%2ftoc.json)

> [!div class="nextstepaction"]
> [Mapper un domaine personnalisé](app-service-web-tutorial-custom-domain.md)
