---
title: 'Démarrage rapide : Votre première requête Azure CLI'
description: Dans ce guide de démarrage rapide, vous suivez les étapes pour activer l’extension Resource Graph pour Azure CLI et vous exécutez votre première requête.
ms.date: 10/14/2020
ms.topic: quickstart
ms.custom: devx-track-azurecli
ms.openlocfilehash: 216972fb3c64efa22dc4d344e5021a3c2da74266
ms.sourcegitcommit: 1b47921ae4298e7992c856b82cb8263470e9e6f9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92056768"
---
# <a name="quickstart-run-your-first-resource-graph-query-using-azure-cli"></a>Démarrage rapide : Exécuter votre première requête Resource Graph à l’aide d’Azure CLI

La première étape consiste à vérifier que l’extension Azure Resource Graph pour [Azure CLI](/cli/azure/) est installée. Ce guide de démarrage rapide décrit le processus d’ajout de l’extension à votre installation Azure CLI. Vous pouvez utiliser l’extension avec Azure CLI dans le cadre d’une installation locale ou par le biais [d’Azure Cloud Shell](https://shell.azure.com).

Au terme de ce processus, vous aurez ajouté l’extension à l’installation Azure CLI de votre choix et vous pourrez exécuter votre première requête Resource Graph.

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="add-the-resource-graph-extension"></a>Ajouter l’extension Resource Graph

Pour permettre à Azure CLI d’interroger Azure Resource Graph, vous devez ajouter l’extension. Cette extension fonctionne avec Azure CLI quel que soit l’endroit où vous l’utilisez : [bash sur Windows 10](/windows/wsl/install-win10), [Cloud Shell](https://shell.azure.com) (en version autonome et à l’intérieur du portail), [image Docker Azure CLI](https://hub.docker.com/_/microsoft-azure-cli) ou installation locale.

1. Vérifiez que la version la plus récente d’Azure CLI est installée (**2.0.76** au minimum). S’il n’est pas installé, suivez [ces instructions](/cli/azure/install-azure-cli-windows).

1. Dans l’environnement Azure CLI de choix, vous devez l’importer avec la commande suivante :

   ```azurecli-interactive
   # Add the Resource Graph extension to the Azure CLI environment
   az extension add --name resource-graph
   ```

1. Vérifiez que l’extension a été installée et qu’il s’agit de la version attendue (au moins **1.0.0**) :

   ```azurecli-interactive
   # Check the extension list (note that you may have other extensions installed)
   az extension list

   # Run help for graph query options
   az graph query -h
   ```

## <a name="run-your-first-resource-graph-query"></a>Exécuter votre première requête Resource Graph

Une fois l’extension Azure CLI ajoutée à l’environnement de votre choix, vous pouvez exécuter une requête Resource Graph simple. La requête retourne les cinq premières ressources Azure avec le nom (**name**) et le **type** de chaque ressource.

1. Exécutez votre première requête Azure Resource Graph à l’aide de l’extension `graph` et de la commande `query` :

   ```azurecli-interactive
   # Login first with az login if not using Cloud Shell

   # Run Azure Resource Graph query
   az graph query -q 'Resources | project name, type | limit 5'
   ```

   > [!NOTE]
   > Comme cet exemple de requête ne fournit pas un modificateur de tri tel que `order by`, l’exécution répétée de cette requête peut produire un ensemble différent de ressources.

1. Mettez à jour la requête pour la trier (`order by`) en fonction du nom (**name**) de la propriété :

   ```azurecli-interactive
   # Run Azure Resource Graph query with 'order by'
   az graph query -q 'Resources | project name, type | limit 5 | order by name asc'
   ```

   > [!NOTE]
   > Comme précédemment, l’exécution répétée de cette requête peut produire un ensemble différent de ressources. L’ordre des commandes de requête est important. Dans cet exemple, `order by` vient après `limit`. Cet ordre de commande limite d’abord les résultats de la requête, puis les classe.

1. Mettez à jour la requête pour d’abord trier (`order by`) les résultats en fonction de la propriété **name**, puis les limiter (`limit`) aux cinq premiers :

   ```azurecli-interactive
   # Run Azure Resource Graph query with `order by` first, then with `limit`
   az graph query -q 'Resources | project name, type | order by name asc | limit 5'
   ```

Si votre environnement ne change pas et si vous exécutez plusieurs fois la requête finale, les résultats retournés sont cohérents et classés en fonction de la propriété **Name**, mais toujours limités aux cinq premiers.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous souhaitez supprimer l’extension Resource Graph de votre environnement Azure CLI, vous pouvez le faire à l’aide de la commande suivante :

```azurecli-interactive
# Remove the Resource Graph extension from the Azure CLI environment
az extension remove -n resource-graph
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez ajouté l’extension Resource Graph à votre environnement Azure CLI et vous avez exécuté votre première requête. Pour plus d’informations sur le langage Resource Graph, passez à la page des détails du langage de requête.

> [!div class="nextstepaction"]
> [Obtenir plus d’informations sur le langage de requête](./concepts/query-language.md)
