---
title: 'Démarrage rapide : Créer une requête partagée avec Azure CLI'
description: Dans ce guide de démarrage rapide, vous suivez les étapes permettant d’activer l’extension Resource Graph pour Azure CLI et vous créez une requête partagée.
ms.date: 10/14/2020
ms.topic: quickstart
ms.openlocfilehash: 93df1c858ac6238a0192bcdedac8286f2cf75007
ms.sourcegitcommit: 03c0a713f602e671b278f5a6101c54c75d87658d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94919707"
---
# <a name="quickstart-create-a-resource-graph-shared-query-using-azure-cli"></a>Démarrage rapide : Créer une requête partagée Resource Graph avec Azure CLI

La première étape pour utiliser Azure Resource Graph avec [Azure CLI](/cli/azure/) consiste à vérifier que l’extension est installée. Ce guide de démarrage rapide décrit le processus d’ajout de l’extension à votre installation Azure CLI. Vous pouvez utiliser l’extension avec Azure CLI dans le cadre d’une installation locale ou par le biais [d’Azure Cloud Shell](https://shell.azure.com).

Au terme de ce processus, vous aurez ajouté l’extension à l’installation Azure CLI de votre choix et créé une requête partagée Resource Graph.

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="add-the-resource-graph-extension"></a>Ajouter l’extension Resource Graph

Pour permettre à Azure CLI d’utiliser Azure Resource Graph, vous devez ajouter l’extension. Cette extension fonctionne avec Azure CLI quel que soit l’endroit où vous l’utilisez : [bash sur Windows 10](/windows/wsl/install-win10), [Cloud Shell](https://shell.azure.com) (en version autonome et à l’intérieur du portail), [image Docker Azure CLI](https://hub.docker.com/_/microsoft-azure-cli) ou installation locale.

1. Vérifiez que la version la plus récente d’Azure CLI est installée (**2.8.0** au minimum). S’il n’est pas installé, suivez [ces instructions](/cli/azure/install-azure-cli-windows).

1. Dans l’environnement Azure CLI de votre choix, utilisez [az extension add](/cli/azure/extension#az_extension_add) pour importer l’extension Resource Graph avec la commande suivante :

   ```azurecli-interactive
   # Add the Resource Graph extension to the Azure CLI environment
   az extension add --name resource-graph
   ```

1. Vérifiez que l’extension a été installée et qu’il s’agit de la version attendue (au moins **1.1.0**) avec [az extension list](/cli/azure/extension#az_extension_list) :

   ```azurecli-interactive
   # Check the extension list (note that you may have other extensions installed)
   az extension list

   # Run help for graph query options
   az graph query -h
   ```

## <a name="create-a-resource-graph-shared-query"></a>Créer une requête partagée Resource Graph

Une fois l’extension Azure CLI ajoutée à l’environnement de votre choix, vous pouvez créer une requête partagée Resource Graph. La requête partagée est un objet Azure Resource Manager auquel vous pouvez accorder des autorisations ou que vous pouvez exécuter dans l’Explorateur Azure Resource Graph. La requête récapitule le nombre total de ressources regroupées par _emplacement_.

1. Créez un groupe de ressources avec [az group create](/cli/azure/group#az_group_create) pour stocker la requête partagée Azure Resource Graph. Ce groupe de ressources est nommé `resource-graph-queries` et son emplacement est `westus2`.

   ```azurecli-interactive
   # Login first with az login if not using Cloud Shell

   # Create the resource group
   az group create --name 'resource-graph-queries' --location 'westus2'
   ```

1. Créez la requête partagée Azure Resource Graph à l’aide de l’extension `graph` et de la commande [az graph shared-query create](/cli/azure/ext/resource-graph/graph/shared-query#ext_resource_graph_az_graph_shared_query_create) :

   ```azurecli-interactive
   # Create the Azure Resource Graph shared query
   az graph shared-query create --name 'Summarize resources by location' \
      --description 'This shared query summarizes resources by location for a pinnable map graphic.' \
      --graph-query 'Resources | summarize count() by location' \
      --resource-group 'resource-graph-queries'
   ```

1. Listez les requêtes partagées dans le nouveau groupe de ressources. La commande [az graph shared-query list](/cli/azure/ext/resource-graph/graph/shared-query#ext_resource_graph_az_graph_shared_query_list) retourne un tableau de valeurs.

   ```azurecli-interactive
   # List all the Azure Resource Graph shared queries in a resource group
   az graph shared-query list --resource-group 'resource-graph-queries'
   ```

1. Pour obtenir un seul résultat de requête partagée, utilisez la commande [az graph shared-query show](/cli/azure/ext/resource-graph/graph/shared-query#ext_resource_graph_az_graph_shared_query_show).

   ```azurecli-interactive
   # Show a specific Azure Resource Graph shared query
   az graph shared-query show --resource-group 'resource-graph-queries' \
      --name 'Summarize resources by location'
   ```

1. Exécutez la requête partagée dans Azure CLI à l’aide de la syntaxe `{{shared-query-uri}}` dans une commande [az graph query](/cli/azure/ext/resource-graph/graph#ext_resource_graph_az_graph_query).
   Tout d’abord, copiez le champ `id` à partir du résultat de la commande `show` précédente. Remplacez le texte `shared-query-uri` de l’exemple par la valeur provenant du champ `id`, mais laissez les caractères `{{` et `}}` autour.

   ```azurecli-interactive
   # Run a Azure Resource Graph shared query
   az graph query --graph-query "{{shared-query-uri}}"
   ```

   > [!NOTE]
   > La syntaxe `{{shared-query-uri}}` est une fonctionnalité en **préversion**.

Une autre façon de rechercher des requêtes partagées Resource Graph consiste à utiliser le portail Azure. Dans le portail, utilisez la barre de recherche pour rechercher « requêtes Resource Graph ». Sélectionnez la requête partagée. Dans la page **Vue d’ensemble** , l’onglet **Requête** affiche la requête enregistrée. Le bouton **Modifier** l’ouvre dans l’[Explorateur Resource Graph](./first-query-portal.md).

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous souhaitez supprimer la requête partagée, le groupe de ressources et l’extension Resource Graph de votre environnement Azure CLI, vous pouvez le faire à l’aide des commandes suivantes :

- [az graph shared-query delete](/cli/azure/ext/resource-graph/graph/shared-query#ext_resource_graph_az_graph_shared_query_delete)
- [az group delete](/cli/azure/group#az_group_delete)
- [az extension remove](/cli/azure/extension#az_extension_remove)

```azurecli-interactive
# Delete the Azure Resource Graph shared query
az graph shared-query delete --resource-group 'resource-graph-queries' \
   --name 'Summarize resources by location'

# Remove the resource group
# WARNING: This command deletes ALL resources you've added to this resource group without prompting for confirmation
az group delete --resource-group 'resource-graph-queries' --yes

# Remove the Azure Resource Graph extension from the Azure CLI environment
az extension remove -n resource-graph
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez ajouté l’extension Resource Graph à votre environnement Azure CLI et vous avez créé une requête partagée. Pour plus d’informations sur le langage Resource Graph, passez à la page des détails du langage de requête.

> [!div class="nextstepaction"]
> [Obtenir plus d’informations sur le langage de requête](./concepts/query-language.md)