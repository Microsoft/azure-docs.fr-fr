---
title: Configurer les paramètres d’application pour Azure Static Web Apps
description: Apprendre à configurer les paramètres d’application pour Azure Static Web Apps
services: static-web-apps
author: burkeholland
ms.service: static-web-apps
ms.topic: how-to
ms.date: 05/08/2020
ms.author: buhollan
ms.custom: devx-track-js, devx-track-azurecli
ms.openlocfilehash: 00502c97e3fc06972c27cedf565d77b52f558795
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94844810"
---
# <a name="configure-application-settings-for-azure-static-web-apps-preview"></a>Configurer les paramètres d’application pour Azure Static Web Apps - Préversion

Les paramètres d’application contiennent les paramètres de configuration de valeurs susceptibles d’être modifiées, comme les chaînes de connexion de base de données. L’ajout de paramètres d’application vous permet de modifier l’entrée de configuration de votre application, sans avoir à modifier le code de l’application.

Les paramètres de l’application sont :

- Chiffrés au repos
- Copiés dans les environnements [intermédiaire](review-publish-pull-requests.md) et de production

Les paramètres d’application sont également parfois appelés variables d’environnement.

> [!IMPORTANT]
> Les paramètres d’application décrits dans cet article s’appliquent uniquement à l’API back-end d’une application Azure Static Web Apps.
>
> Pour plus d’informations sur l’utilisation des variables d’environnement avec votre application web, consultez la documentation de votre [framework JavaScript](#javascript-frameworks-and-libraries) ou du [générateur de site statique](#static-site-generators).

## <a name="prerequisites"></a>Prérequis

- Une application Azure Static Web Apps
- [Azure CLI](/cli/azure/install-azure-cli?view=azure-cli-latest)

## <a name="types-of-application-settings"></a>Types de paramètres d’application

Il existe généralement deux aspects dans une application Azure Static Web Apps. La première est l’application web, ou contenu statique, qui est représentée par du code HTML, CSS, JavaScript et des images. La seconde est l’API back-end, qui utilise une application Azure Functions.

Cet article explique comment gérer les paramètres d’application pour l’API back-end dans Azure Functions.

Les paramètres d’application décrits dans cet article ne peuvent pas être utilisés ou référencés dans des applications web statiques. Toutefois, de nombreux frameworks et générateurs de site statiques permettent l’utilisation de variables d’environnement pendant la phase de développement. Au moment de la compilation, ces variables sont remplacées par leurs valeurs dans le code HTML ou JavaScript généré. Étant donné que les données au format HTML et JavaScript sont facilement détectables par les visiteurs du site, vous devez éviter de laisser des informations sensibles dans l’application frontale. Les paramètres qui contiennent des données sensibles doivent se trouver dans la partie API de votre application.

Pour plus d’informations sur l’utilisation des variables d’environnement avec votre framework ou bibliothèque JavaScript, reportez-vous aux articles suivants pour plus de détails.

### <a name="javascript-frameworks-and-libraries"></a>Bibliothèques et frameworks JavaScript

- [Angular](https://angular.io/guide/build#configuring-application-environments)
- [React](https://create-react-app.dev/docs/adding-custom-environment-variables/)
- [Svelte](https://linguinecode.com/post/how-to-add-environment-variables-to-your-svelte-js-app)
- [Vue](https://cli.vuejs.org/guide/mode-and-env.html)

### <a name="static-site-generators"></a>Générateurs de sites statiques

- [Gatsby](https://www.gatsbyjs.org/docs/environment-variables/)
- [Hugo](https://gohugo.io/getting-started/configuration/)
- [Jekyll](https://jekyllrb.com/docs/configuration/environments/)

## <a name="about-api-app-settings"></a>À propos des paramètres d’application API

Les API dans Azure Static Web Apps sont fournies par Azure Functions, ce qui vous permet de définir les paramètres d’application dans le fichier _local.settings.json_. Ce fichier définit les paramètres d’application dans la propriété `Values` de la configuration.

L’exemple suivant _local.settings.json_ montre comment ajouter une valeur pour `DATABASE_CONNECTION_STRING`.

```json
{
  "IsEncrypted": false,
  "Values": {
    "AzureWebJobsStorage": "",
    "FUNCTIONS_WORKER_RUNTIME": "node",
    "DATABASE_CONNECTION_STRING": "<YOUR_DATABASE_CONNECTION_STRING>"
  }
}
```

Les paramètres définis dans la propriété `Values` peuvent être référencés à partir du code en tant que variables d’environnement, disponibles dans l’objet `process.env`.

```js
const connectionString = process.env.DATABASE_CONNECTION_STRING;
```

Le fichier `local.settings.json` n’est pas suivi par le référentiel GitHub, car les informations sensibles, comme les chaînes de connexion de base de données, sont souvent incluses dans le fichier. Étant donné que les paramètres locaux restent sur votre ordinateur, vous devez charger manuellement vos paramètres sur Azure.

Le chargement des paramètres est rarement effectué de façon régulière et il n’est pas nécessaire pour chaque compilation.

## <a name="uploading-application-settings"></a>Chargement des paramètres d’application

Vous pouvez configurer les paramètres d’application via le Portail Azure ou avec l’interface de ligne de commande Azure.

### <a name="using-the-azure-portal"></a>Utilisation du portail Azure

Le portail Azure fournit une interface pour la création, la mise à jour et la suppression des paramètres d’application.

1. Accédez au [portail Azure](https://portal.azure.com).

1. Dans la barre de recherche, recherchez et sélectionnez **Static Web Apps**.

1. Cliquez sur l’option de **configuration** dans la barre latérale.

1. Sélectionnez l’environnement auquel vous souhaitez appliquer les paramètres d’application. Les environnements intermédiaires sont automatiquement créés quand une demande de tirage (pull request) est générée et sont promus en production une fois que la demande de tirage est fusionnée. Vous pouvez définir les paramètres d’application par environnement.

1. Cliquez sur le bouton **Ajouter** pour ajouter un nouveau paramètre d’application.

    :::image type="content" source="media/application-settings/configuration.png" alt-text="Affichage de la configuration Azure Static Web Apps":::

1. Entrez un **nom** et une **valeur**.

1. Cliquez sur **OK**.

1. Cliquez sur **Enregistrer**.

### <a name="using-the-azure-cli"></a>Utilisation de l’interface de ligne de commande Azure (CLI)

Vous pouvez utiliser la commande `az rest` pour effectuer des chargements par lots de vos paramètres sur Azure. La commande accepte les paramètres d’application en tant qu’objets JSON dans une propriété parent appelée `properties`.

Le moyen le plus simple de créer un fichier JSON avec les valeurs appropriées consiste à créer une version modifiée de votre fichier _local.settings.json_.

1. Pour vous assurer que votre nouveau fichier contenant des données sensibles n’est pas exposé publiquement, ajoutez l’entrée suivante dans votre fichier _.gitignore_.

   ```bash
   local.settings*.json
   ```

2. Ensuite, faites une copie de votre fichier _local.settings.json_ et nommez-le _local.settings.properties.json_.

3. Dans ce nouveau fichier, supprimez toutes les autres données, à l’exception des paramètres d’application et renommez `Values` en `properties`.

   Votre fichier doit maintenant ressembler à l’exemple suivant :

   ```json
   {
     "properties": {
       "DATABASE_CONNECTION_STRING": "<YOUR_DATABASE_CONNECTION_STRING>"
     }
   }
   ```

L’interface de ligne de commande Azure exige un certain nombre de valeurs spécifiques à votre compte pour exécuter le chargement. Dans la fenêtre _Vue d’ensemble_ de votre ressource Static Web Apps, vous avez accès aux informations suivantes :

1. Nom du site statique
2. Nom de groupe ressources
3. Identifiant d’abonnement

:::image type="content" source="media/application-settings/overview.png" alt-text="Vue d’ensemble d’Azure Static Web Apps":::

4. Dans un terminal ou une ligne de commande, exécutez la commande suivante. Veillez à remplacer les espaces réservés de `<YOUR_STATIC_SITE_NAME>`, `<YOUR_RESOURCE_GROUP_NAME>`et `<YOUR_SUBSCRIPTION_ID>` par les valeurs de la fenêtre _Vue d’ensemble_.

   ```bash
   az rest --method put --headers "Content-Type=application/json" --uri "/subscriptions/<YOUR_SUBSCRIPTION_ID>/resourceGroups/<YOUR_RESOURCE_GROUP_NAME>/providers/Microsoft.Web/staticSites/<YOUR_STATIC_SITE_NAME>/config/functionappsettings?api-version=2019-12-01-preview" --body @local.settings.properties.json
   ```

> [!IMPORTANT]
> Le fichier « local.settings.properties.json » doit se trouver dans le même répertoire que celui où cette commande est exécutée. Ce fichier peut porter le nom que vous souhaitez. Ce nom n’a pas d’importance.

### <a name="view-application-settings-with-the-azure-cli"></a>Voir les paramètres d’application avec Azure CLI

Les paramètres d’application peuvent être consultés via l’interface de ligne de commande Azure.

- Dans un terminal ou une ligne de commande, exécutez la commande suivante. Veillez à remplacer les espaces réservés `<YOUR_SUBSCRIPTION_ID>`, `<YOUR_RESOURCE_GROUP_NAME>` et `<YOUR_STATIC_SITE_NAME>` par vos valeurs.

   ```bash
   az rest --method post --uri "/subscriptions/<YOUR_SUBSCRIPTION_ID>/resourceGroups/<YOUR_RESOURCE_GROUP_NAME>/providers/Microsoft.Web/staticSites/<YOUR_STATIC_SITE_NAME>/listFunctionAppSettings?api-version=2019-12-01-preview"
   ```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Configurer le développement local](local-development.md)
