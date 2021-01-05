---
title: Déployer une application web avec un modèle – Azure Cosmos DB
description: Découvrez comment déployer un compte Azure Cosmos, Azure App Service Web Apps et un échantillon d’application web avec un modèle Azure Resource Manager.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 06/19/2020
ms.author: mjbrown
ms.openlocfilehash: 55d58a6c4724bd01325db029ed75d77ccc96d0f8
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93333577"
---
# <a name="deploy-azure-cosmos-db-and-azure-app-service-with-a-web-app-from-github-using-an-azure-resource-manager-template"></a>Déployer Azure Cosmos DB et Azure App Service avec une application web à partir de GitHub à l’aide d’un modèle Azure Resource Manager
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Ce didacticiel vous montre comment effectuer un déploiement « sans toucher » d’une application web qui se connecte à Azure Cosmos DB lors de la première exécution sans avoir à couper et coller les informations de connexion à partir d’Azure Cosmos DB vers `appsettings.json` ou vers les paramètres de l’application Azure App Services dans le Portail Azure. Toutes ces actions sont accomplies à l’aide d’un modèle Azure Resource Manager en une seule opération. Dans cet exemple, nous allons déployer l’[échantillon de tâche ToDo Azure Cosmos DB](https://github.com/Azure-Samples/cosmos-dotnet-core-todo-app) à partir d’un [didacticiel d’application web](sql-api-dotnet-application.md).

Les modèles Resource Manager sont très flexibles et vous permettent de composer des déploiements complexes sur n’importe quel service dans Azure. Cela comprend des tâches avancées telles que le déploiement d’applications à partir de GitHub et l’injection d’informations de connexion dans les paramètres d’application d’Azure App Service dans le Portail Azure. Ce didacticiel vous montre comment effectuer les opérations suivantes à l’aide d’un seul modèle Resource Manager.

* Déployez un compte Azure Cosmos.
* Déployez un plan d’hébergement Azure App Service.
* Déployez un  Azure App Service.
* Injectez le point de terminaison et les clés du compte Azure Cosmos dans les paramètres d’application App Service dans le Portail Azure.
* Déployez une application web à partir d’un référentiel GitHub vers l’App Service.

Le déploiement obtenu dispose d’une application web entièrement fonctionnelle qui peut se connecter à Azure Cosmos DB sans avoir à couper et coller l’URL de point de terminaison ou les clés d’authentification d’Azure Cosmos DB à partir du Portail Azure.

## <a name="prerequisites"></a>Prérequis

> [!TIP]
> Ce didacticiel ne suppose pas d’être familiarisé avec les modèles Azure Resource Manager ou JSON, sauf si vous envisagez de modifier les modèles référencés ou les options de déploiement.

## <a name="step-1-deploy-the-template"></a>Étape 1 : Déployer le modèle

Tout d’abord, sélectionnez le bouton **Déployer sur Azure** ci-dessous pour ouvrir le Portail Azure afin de créer un déploiement personnalisé. Vous pouvez également afficher le modèle Azure Resource Management depuis la [galerie de modèles de démarrage rapide Azure](https://github.com/Azure/azure-quickstart-templates/tree/master/101-cosmosdb-webapp)

[:::image type="content" source="../media/template-deployments/deploy-to-azure.svg" alt-text="Déployer sur Azure":::](https://portal.azure.com/#create/Microsoft.Template/uri/https%3A%2F%2Fraw.githubusercontent.com%2FAzure%2Fazure-quickstart-templates%2Fmaster%2F101-cosmosdb-webapp%2Fazuredeploy.json)

Une fois dans le Portail Azure, sélectionnez l’abonnement dans lequel effectuer le déploiement, puis sélectionnez ou créez un nouveau groupe de ressources. Renseignez ensuite les valeurs suivantes.

:::image type="content" source="./media/create-website/template-deployment.png" alt-text="Capture d’écran de l’interface utilisateur du déploiement de modèle":::

* **Région**  : cela est requis par le Resource Manager. Entrez la même région que celle utilisée par le paramètre d’emplacement où se trouvent vos ressources.
* **Nom de l’application**  : ce nom est utilisé par toutes les ressources pour ce déploiement. Veillez à choisir un nom unique pour éviter les conflits avec les comptes Azure Cosmos DB et App Service existants.
* **Emplacement**  : la région où les ressources sont déployées.
* **Niveau du plan App Service**  : niveau tarifaire du plan App Service.
* **Instances du plan App Service**  : le nombre de Workers du plan App Service.
* **URL du référentiel**  : le référentiel de l’application web sur GitHub.
* **Branche**  : la branche du référentiel GitHub.
* **Nom de la base de données**  : le nom de la base de données Azure Cosmos.
* **Nom du conteneur**  : le nom du conteneur Azure Cosmos.

Après avoir renseigné les valeurs, sélectionnez le bouton **Créer** pour démarrer le déploiement. Cette étape devrait prendre 5 à 10 minutes.

> [!TIP]
> Ce modèle ne valide pas le fait que le nom d’Azure App Service et le nom du compte Azure Cosmos DB entrés dans le modèle soient corrects et disponibles. Il est vivement recommandé de vérifier la disponibilité des noms que vous souhaitez fournir avant de soumettre le déploiement.


## <a name="step-2-explore-the-resources"></a>Étape 2 : Explorer les ressources

### <a name="view-the-deployed-resources"></a>Afficher les ressources déployées

Une fois que le modèle a déployé les ressources, vous pouvez voir chacune d’elles dans votre groupe de ressources.

:::image type="content" source="./media/create-website/resource-group.png" alt-text="Groupe de ressources":::

### <a name="view-cosmos-db-endpoint-and-keys"></a>Afficher le point de terminaison et les clés Cosmos DB

Ensuite, ouvrez le compte Azure Cosmos DB dans le portail. La capture d’écran suivante montre le point de terminaison et les clés d’un compte Azure Cosmos.

:::image type="content" source="./media/create-website/cosmos-keys.png" alt-text="Clés Cosmos":::

### <a name="view-the-azure-cosmos-db-keys-in-application-settings"></a>Afficher les clés Azure Cosmos DB dans les paramètres de l’application

Ensuite, naviguez dans Azure App Service, dans le groupe de ressources. Cliquez sur l’onglet Configuration pour afficher les paramètres de configuration de l’App Service. Les paramètres d’application contiennent le compte Cosmos DB et les valeurs de clé primaire nécessaires pour se connecter à Cosmos DB, ainsi que les noms de la base de données et du conteneur qui ont été transmis à partir du déploiement du modèle.

:::image type="content" source="./media/create-website/application-settings.png" alt-text="Paramètres de l’application":::

### <a name="view-web-app-in-deployment-center"></a>Afficher l’application web dans le Centre de déploiement

Ensuite, accédez au Centre de déploiement pour l’App Service. Vous verrez ici des points de référentiel vers le référentiel GitHub passés au modèle. En outre, l’état ci-dessous indique Réussi (active), ce qui signifie que l’application a été déployée et démarrée.

:::image type="content" source="./media/create-website/deployment-center.png" alt-text="Centre de déploiement":::

### <a name="run-the-web-application"></a>Exécuter l’application web

Cliquez sur **parcourir** en haut du Centre de déploiement pour ouvrir l’application web. L’application web s’ouvre sur l’écran d’accueil. Cliquez sur **Créer une nouvelle** et entrez des données dans les champs, puis cliquez sur Enregistrer. L’écran qui s’affiche montre les données enregistrées dans Cosmos DB.

:::image type="content" source="./media/create-website/app-home-screen.png" alt-text="Écran d'accueil":::

## <a name="step-3-how-does-it-work"></a>Étape 3 : Comment cela fonctionne-t-il

Trois éléments sont nécessaires pour que cela fonctionne.

### <a name="reading-app-settings-at-runtime"></a>Lecture des paramètres de l’application au moment de l’exécution

Tout d’abord, l’application doit demander le point de terminaison et la clé Cosmos DB dans la classe `Startup` de l’application web ASP.NET MVC. L’[échantillon To Do Cosmos DB](https://github.com/Azure-Samples/cosmos-dotnet-core-todo-app) peut s’exécuter localement, où vous pouvez entrer les informations de connexion dans appsettings.json. Toutefois, lors du déploiement, ce fichier est déployé avec l’application. Si ces lignes en rouge ne peuvent pas accéder aux paramètres depuis appsettings.json, elles essaieront à partir des paramètres d’application dans Azure App Service.

:::image type="content" source="./media/create-website/startup.png" alt-text="La capture d’écran présente une méthode avec plusieurs variables de chaîne marquées en rouge, notamment databaseName, containerName, account et key.":::

### <a name="using-special-azure-resource-management-functions"></a>Utilisation de fonctions spéciales Azure Resource Management

Pour que ces valeurs soient disponibles pour l’application lors de son déploiement, le modèle Azure Resource Manager peut demander ces valeurs à partir du compte Cosmos DB à l’aide de fonctions spéciales Azure Resource Management, y compris la [référence](../azure-resource-manager/templates/template-functions-resource.md#reference) et les [listKeys](../azure-resource-manager/templates/template-functions-resource.md#listkeys) qui récupèrent les valeurs du compte Cosmos DB et les insèrent dans les valeurs des paramètres de l’application avec des noms de clé qui correspondent à ce qui est utilisé dans l’application ci-dessus dans un format '{section:key}'. Par exemple : `CosmosDb:Account`.

:::image type="content" source="./media/create-website/template-keys.png" alt-text="Clés de modèle":::

### <a name="deploying-web-apps-from-github"></a>Déploiement d’applications web à partir de GitHub

Enfin, nous devons déployer l’application web à partir de GitHub dans l’App Service. Cette opération est effectuée avec la JSON ci-dessous. Les deux éléments à prendre en considération sont le type et le nom de cette ressource. Les valeurs de propriété `"type": "sourcecontrols"` et `"name": "web"` sont codées en dur et ne doivent pas être modifiées.

:::image type="content" source="./media/create-website/deploy-from-github.png" alt-text="Déployer à partir de GitHub":::

## <a name="next-steps"></a>Étapes suivantes

Félicitations ! Vous avez déployé Azure Cosmos DB, Azure App Service et un échantillon d’application web qui dispose automatiquement des informations de connexion nécessaires pour se connecter à Cosmos DB, le tout en une seule opération et sans avoir à couper et coller des informations sensibles. En utilisant ce modèle comme point de départ, vous pouvez le modifier pour déployer vos propres applications web de la même façon.

* Pour obtenir l’échantillon du modèle Azure Resource Manager, accédez à la [galerie de modèles de démarrage rapide Azure](https://github.com/Azure/azure-quickstart-templates/tree/master/101-cosmosdb-webapp)
* Pour obtenir le code source de l’échantillon d’application, accédez à [Cosmos DB To Do App sur GitHub](https://github.com/Azure-Samples/cosmos-dotnet-core-todo-app).
