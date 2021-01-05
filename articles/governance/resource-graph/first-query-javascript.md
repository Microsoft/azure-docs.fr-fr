---
title: 'Démarrage rapide : Votre première requête JavaScript'
description: Dans ce guide de démarrage rapide, vous suivez les étapes permettant d’activer la bibliothèque Resource Graph pour JavaScript et vous exécutez votre première requête.
ms.date: 10/14/2020
ms.topic: quickstart
ms.custom: devx-track-js
ms.openlocfilehash: 194097594340e2f34433c07185c6132f8a13c845
ms.sourcegitcommit: 1b47921ae4298e7992c856b82cb8263470e9e6f9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92057414"
---
# <a name="quickstart-run-your-first-resource-graph-query-using-javascript"></a>Démarrage rapide : Exécuter votre première requête Resource Graph à l’aide de JavaScript

Ce guide de démarrage rapide décrit le processus d’ajout des bibliothèques à votre installation JavaScript. La première étape pour utiliser Azure Resource Graph consiste à initialiser une application JavaScript avec les bibliothèques nécessaires.

Au terme de ce processus, vous aurez ajouté les bibliothèques à votre installation JavaScript et vous pourrez exécuter votre première requête Resource Graph.

## <a name="prerequisites"></a>Prérequis

- **Abonnement Azure** : Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

- **Node.JS** : La version 12 ou supérieure de [Node.js](https://nodejs.org/) est nécessaire.

## <a name="application-initialization"></a>Initialisation d’applications

Pour permettre à JavaScript d’interroger Azure Resource Graph, l’environnement doit être configuré. Cette configuration fonctionne partout où vous pouvez utiliser JavaScript, y compris [bash sur Windows 10](/windows/wsl/install-win10).

1. Initialisez un nouveau projet Node.js en exécutant la commande suivante.

   ```bash
   npm init -y
   ```

1. Ajoutez une référence au module yargs.

   ```bash
   npm install yargs
   ```

1. Ajoutez une référence au module Azure Resource Graph.

   ```bash
   npm install @azure/arm-resourcegraph
   ```

1. Ajoutez une référence à la bibliothèque d’authentification Azure.

   ```bash
   npm install @azure/ms-rest-nodeauth
   ```

   > [!NOTE]
   > Vérifiez dans _package.json_ que la version de `@azure/arm-resourcegraph` est **2.0.0** ou ultérieure et que la version de `@azure/ms-rest-nodeauth` est **3.0.3** ou ultérieure.

## <a name="query-the-resource-graph"></a>Interroger Resource Graph

1. Créez un fichier nommé _index.js_ et entrez le code suivant.

   ```javascript
   const argv = require("yargs").argv;
   const authenticator = require("@azure/ms-rest-nodeauth");
   const resourceGraph = require("@azure/arm-resourcegraph");

   if (argv.query && argv.subs) {
       const subscriptionList = argv.subs.split(",");

       const query = async () => {
          const credentials = await authenticator.interactiveLogin();
          const client = new resourceGraph.ResourceGraphClient(credentials);
          const result = await client.resources(
             {
                 query: argv.query,
                 subscriptions: subscriptionList,
             },
             { resultFormat: "table" }
          );
          console.log("Records: " + result.totalRecords);
          console.log(result.data);
       };

       query();
   }
   ```

1. Entrez la commande suivante dans le terminal :

   ```bash
   node index.js --query "Resources | project name, type | limit 5" --subs <YOUR_SUBSCRIPTION_ID_LIST>
   ```

   Veillez à remplacer l’espace réservé `<YOUR_SUBSCRIPTION_ID_LIST>` par la liste des ID d’abonnement Azure séparés par des virgules.

   > [!NOTE]
   > Comme cet exemple de requête ne fournit pas de modificateur de tri tel que `order by`, l’exécution répétée de cette requête peut produire un ensemble distinct de ressources par requête.

1. Remplacez le premier paramètre par `index.js` et la requête par `order by` suivi de la propriété **Name**. Remplacez `<YOUR_SUBSCRIPTION_ID_LIST>` par votre ID d’abonnement :

   ```bash
   node index.js --query "Resources | project name, type | limit 5 | order by name asc" --subs "<YOUR_SUBSCRIPTION_ID_LIST>"
   ```

   Quand le script tente de s’authentifier, un message similaire au message suivant s’affiche dans le terminal :

   > Pour vous connecter, utilisez un navigateur web afin d’ouvrir la page https://microsoft.com/devicelogin et entrez le code FGB56WJUGK pour vous authentifier.

   Une fois que vous vous authentifiez dans le navigateur, le script continue à s’exécuter.

   > [!NOTE]
   > Comme précédemment, l’exécution répétée de cette requête peut produire un ensemble différent de ressources. L’ordre des commandes de requête est important. Dans cet exemple, `order by` vient après `limit`. Cet ordre de commande limite d’abord les résultats de la requête, puis les classe.

1. Remplacez le premier paramètre par `index.js` et changez la requête en `order by` pour classer d’abord les résultats selon la propriété **Name**, puis limitez (`limit`) les résultats aux cinq premiers. Remplacez `<YOUR_SUBSCRIPTION_ID_LIST>` par votre ID d’abonnement :

   ```bash
   node index.js --query "Resources | project name, type | order by name asc | limit 5" --subs "<YOUR_SUBSCRIPTION_ID_LIST>"
   ```

Si votre environnement ne change pas et si vous exécutez plusieurs fois la requête finale, les résultats retournés sont cohérents et classés en fonction de la propriété **Name**, mais toujours limités aux cinq premiers.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous souhaitez supprimer les bibliothèques installées de votre application, exécutez la commande suivante.

```bash
npm uninstall @azure/arm-resourcegraph @azure/ms-rest-nodeauth yargs
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez ajouté les bibliothèques Resource Graph à votre environnement JavaScript et vous avez exécuté votre première requête. Pour plus d’informations sur le langage Resource Graph, passez à la page des détails du langage de requête.

> [!div class="nextstepaction"]
> [Obtenir plus d’informations sur le langage de requête](./concepts/query-language.md)