---
title: 'Tutoriel : Créer une application web statique avec Blazor dans Azure Static Web Apps'
description: Apprenez à créer un site web Azure Static Web Apps avec Blazor.
services: static-web-apps
author: craigshoemaker
ms.service: static-web-apps
ms.topic: tutorial
ms.date: 09/10/2020
ms.author: cshoe
ms.openlocfilehash: 0086f7f68fd05d6925d19c7ab457fbc125e36be4
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96350226"
---
# <a name="tutorial-building-a-static-web-app-with-blazor-in-azure-static-web-apps"></a>Tutoriel : Créer une application web statique avec Blazor dans Azure Static Web Apps

Azure Static Web Apps publie un site web dans un environnement de production en compilant les applications à partir d’un dépôt GitHub. Dans ce tutoriel, vous allez déployer une application web dans Azure Static Web Apps à partir du portail Azure.

Si vous n’avez pas d’abonnement Azure, [créez un compte d’essai gratuit](https://azure.microsoft.com/free).

## <a name="prerequisites"></a>Prérequis

- [GitHub](https://github.com)
- Compte [Azure](https://portal.azure.com)

## <a name="application-overview"></a>Vue d’ensemble de l’application

Azure Static Web Apps vous permet de créer des applications web statiques prises en charge par un back-end serverless. Le tutoriel suivant explique comment déployer une application web C# Blazor qui renvoie des données météorologiques.

:::image type="content" source="./media/deploy-blazor/blazor-app-complete.png" alt-text="Application Blazor complète":::

L'application présentée dans ce tutoriel est constituée de trois projets Visual Studio différents :

- **Api** : application Azure Functions C# implémentant le point de terminaison d'API qui fournit les informations météorologiques à l'application statique. **WeatherForecastFunction** retourne un tableau d’objets `WeatherForecast`.

- **Client** : projet d'assembly web Blazor frontal. Un [itinéraire de secours](#fallback-route) est implémenté pour veiller à ce que tous les itinéraires reçoivent le fichier _index.html_.

- **Partagé** : contient les classes communes référencées par les projets API et Client, ce qui permet aux données de circuler du point de terminaison d'API vers l'application web frontale. La classe [`WeatherForecast`](https://github.com/staticwebdev/blazor-starter/blob/main/Shared/WeatherForecast.cs) est partagée entre les deux applications.

Ces projets sont tous nécessaires à la création d'une application d'assembly web Blazor exécutée dans le navigateur pris en charge par un back-end d'API.

## <a name="fallback-route"></a>Itinéraire de secours

L'application expose des URL telles que _/counter_ et _/fetchdata_ qui correspondent à des itinéraires spécifiques de l'application. Dans la mesure où cette application est implémentée sous forme d'application monopage, chaque itinéraire reçoit le fichier _index.html_. Pour veiller à ce que toutes les requêtes de chemin renvoient _index.html_, un [itinéraire de secours](./routes.md#fallback-routes) est implémenté dans le fichier _routes.json_ situé dans le dossier _wwwroot_ du projet Client.

```json
{
  "routes": [
    {
      "route": "/*",
      "serve": "/index.html",
      "statusCode": 200
    }
  ]
}
```

Grâce à la configuration ci-dessus, les requêtes adressées à tout itinéraire associé à l'application renvoient la page _index.html_.

## <a name="create-a-repository"></a>Créer un référentiel

Cet article utilise un dépôt de modèles GitHub pour vous permettre de démarrer facilement. Le modèle comprend une application de démarrage déployée sur Azure Static Web Apps.

1. Veillez à être connecté à GitHub et accédez à l’emplacement suivant pour créer un dépôt :
    - [https://github.com/staticwebdev/blazor-starter/generate](https://github.com/login?return_to=/staticwebdev/blazor-starter/generate)
1. Nommez votre référentiel **my-first-static-blazor-app**

## <a name="create-a-static-web-app"></a>Créer une application web statique

Maintenant que le référentiel est créé, créez une application web statique à partir du portail Azure.

1. Accéder au [portail Azure](https://portal.azure.com)
1. Sélectionnez **Créer une ressource**
1. Recherchez **Applications web statiques**
1. Sélectionnez **Static Web Apps (Préversion)**
1. Sélectionnez **Créer**

Dans la section _De base_, commencez par configurer votre nouvelle application et liez-la à un référentiel GitHub.

:::image type="content" source="media/deploy-blazor/basics.png" alt-text="Onglet Informations de base":::

1. Sélectionnez votre _abonnement Azure_.
1. Sélectionnez ou créez un _groupe de ressources_.
1. Nommez l'application **my-first-static-blazor-app**.
    - Les caractères valides sont `a-z` (insensible à la casse), `0-9`et `-`.
1. Sélectionnez la _région_ la plus proche de vous.
1. Sélectionnez **SKU** _gratuite_.
1. Sélectionnez le bouton **Se connecter à GitHub** et authentifiez-vous auprès de GitHub.

Après vous être connecté à GitHub, entrez les informations relatives au dépôt.

:::image type="content" source="media/deploy-blazor/repository-details.png" alt-text="Détails du référentiel":::

1. Sélectionnez votre _organisation_ préférée.
1. Sélectionnez **my-first-static-blazor-app** dans la liste déroulante _Référentiel_.
1. Sélectionnez **principale** dans la liste déroulante _Branche_.

    En l’absence de dépôt, il peut être nécessaire d’autoriser Azure Static Web Apps dans GitHub. Accédez à votre dépôt GitHub, puis à **Paramètres > Applications > Applications OAuth autorisées**, sélectionnez **Azure Static Web Apps**, puis sélectionnez **Octroyer**. Pour les dépôts de l’organisation, vous devez être propriétaire de l’organisation pour accorder des autorisations.

1. Dans la section _Détails de la build_, ajoutez les détails de configuration spécifiques à Blazor.

    - Sélectionnez **Blazor** dans la liste déroulante _Présélections de build_ et conservez toutes les valeurs par défaut.

1. Sélectionnez **Revoir + créer**.

    :::image type="content" source="media/deploy-blazor/review-create.png" alt-text="Bouton Vérifier + créer":::

1. Sélectionnez **Create** (Créer).

    :::image type="content" source="media/deploy-blazor/create-button.png" alt-text="Bouton Créer":::

1. Sélectionnez **Accéder à la ressource**.

    :::image type="content" source="media/deploy-blazor/resource-button.png" alt-text="Accédez au bouton de ressources":::

## <a name="view-the-website"></a>Voir le site web

Le déploiement d’une application statique comporte deux aspects. Le premier provisionne les ressources Azure sous-jacentes qui composent votre application. Le second est un flux de travail GitHub Actions qui génère et publie votre application.

Avant de pouvoir accéder à votre nouveau site statique, la build de déploiement doit d’abord finir de s’exécuter.

La fenêtre de vue d’ensemble de Static Web Apps présente des liens qui vous permettent d’interagir avec votre application web.

:::image type="content" source="./media/deploy-blazor/overview-window.png" alt-text="Fenêtre Vue d’ensemble":::

1. Cliquez sur la bannière intitulée _Cliquez ici pour vérifier l’état de vos exécutions GitHub Actions_ afin d’accéder aux actions GitHub en cours d’exécution sur votre dépôt. Une fois que vous avez vérifié que le travail de déploiement est terminé, vous pouvez accéder à votre site web à l’aide de l’URL générée.

2. Une fois le flux de travail GitHub Actions terminé, vous pouvez sélectionner l'_URL_ pour ouvrir le site web dans un nouvel onglet.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous ne prévoyez pas de continuer à utiliser cette application, vous pouvez supprimer l’instance Azure Static Web Apps en effectuant les étapes suivantes :

1. Ouvrez le [portail Azure](https://portal.azure.com)
1. Recherchez **my-first-static-blazor-app** sur la barre de recherche supérieure.
1. Sélectionnez le nom de l’application.
1. Sélectionnez le bouton **Supprimer**.
1. Sélectionnez **Oui** pour confirmer la suppression.

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Authentification et autorisation](./authentication-authorization.md)
