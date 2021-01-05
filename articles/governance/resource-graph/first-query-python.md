---
title: 'Démarrage rapide : Votre première requête Python'
description: Dans ce démarrage rapide, vous suivez les étapes pour activer la bibliothèque Resource Graph pour Python et vous exécutez votre première requête.
ms.date: 10/14/2020
ms.topic: quickstart
ms.custom: devx-track-python, devx-track-azurecli
ms.openlocfilehash: 01e0fd5bfbc5f353474c8305eefc044390c3586a
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94845105"
---
# <a name="quickstart-run-your-first-resource-graph-query-using-python"></a>Démarrage rapide : Exécuter votre première requête Resource Graph à l’aide de Python

La première étape pour utiliser Azure Resource Graph est de vérifier que les bibliothèques requises pour Python sont installées. Ce démarrage rapide décrit le processus d’ajout des bibliothèques à votre installation Python.

Au terme de ce processus, vous aurez ajouté les bibliothèques à votre installation Python et vous pourrez exécuter votre première requête Resource Graph.

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="add-the-resource-graph-library"></a>Ajouter la bibliothèque Resource Graph

Pour permettre à Python d’interroger Azure Resource Graph, vous devez ajouter la bibliothèque. Cette bibliothèque fonctionne partout où Python peut être utilisé, notamment avec [bash sur Windows 10](/windows/wsl/install-win10) ou via une installation locale.

1. Vérifiez que la version la plus récente de Python est installée (**3.8** au minimum). Si ce n’est pas le cas, téléchargez-la à l’adresse [Python.org](https://www.python.org/downloads/).

1. Vérifiez que la version la plus récente d’Azure CLI est installée (**2.5.1** au minimum). Si ce n’est pas le cas, consultez [Installer Azure CLI](/cli/azure/install-azure-cli).

   > [!NOTE]
   > Azure CLI est requis pour permettre à Python d’utiliser **l’authentification basée sur l’interface CLI** dans les exemples suivants. Pour plus d’informations sur les autres options, consultez [S’authentifier à l’aide des bibliothèques de gestion Azure pour Python](/azure/developer/python/azure-sdk-authenticate).

1. S’authentifier par le biais d’Azure CLI.

   ```azurecli
   az login
   ```

1. Dans votre environnement Python, installez les bibliothèques requises pour Azure Resource Graph :

   ```bash
   # Add the Resource Graph library for Python
   pip install azure-mgmt-resourcegraph

   # Add the Resources library for Python
   pip install azure-mgmt-resource

   # Add the CLI Core library for Python for authentication (development only!)
   pip install azure-cli-core
   ```

   > [!NOTE]
   > Si Python est installé pour tous les utilisateurs, cette commande doit être exécutée à partir d’une console avec élévation de privilèges.

1. Vérifiez que les bibliothèques ont été installées. `azure-mgmt-resourcegraph` doit avoir la version **2.0.0** ou supérieure, `azure-mgmt-resource` doit avoir la version **9.0.0** ou supérieure et `azure-cli-core` doit avoir la version **2.5.0** ou supérieure.

   ```bash
   # Check each installed library
   pip show azure-mgmt-resourcegraph azure-mgmt-resource azure-cli-core
   ```

## <a name="run-your-first-resource-graph-query"></a>Exécuter votre première requête Resource Graph

Une fois les bibliothèques Python ajoutées à l’environnement de votre choix, vous pouvez exécuter une requête Resource Graph simple. La requête retourne les cinq premières ressources Azure avec le nom (**Name**) et le **Type** de chaque ressource.

1. Exécutez votre première requête Azure Resource Graph à l’aide des bibliothèques installées et de la méthode `resources` :

   ```python
   # Import Azure Resource Graph library
   import azure.mgmt.resourcegraph as arg
   
   # Import specific methods and models from other libraries
   from azure.common.credentials import get_azure_cli_credentials
   from azure.common.client_factory import get_client_from_cli_profile
   from azure.mgmt.resource import SubscriptionClient
   
   # Wrap all the work in a function
   def getresources( strQuery ):
       # Get your credentials from Azure CLI (development only!) and get your subscription list
       subsClient = get_client_from_cli_profile(SubscriptionClient)
       subsRaw = []
       for sub in subsClient.subscriptions.list():
           subsRaw.append(sub.as_dict())
       subsList = []
       for sub in subsRaw:
           subsList.append(sub.get('subscription_id'))
       
       # Create Azure Resource Graph client and set options
       argClient = get_client_from_cli_profile(arg.ResourceGraphClient)
       argQueryOptions = arg.models.QueryRequestOptions(result_format="objectArray")
       
       # Create query
       argQuery = arg.models.QueryRequest(subscriptions=subsList, query=strQuery, options=argQueryOptions)
       
       # Run query
       argResults = argClient.resources(argQuery)
   
       # Show Python object
       print(argResults)
   
   getresources("Resources | project name, type | limit 5")
   ```

   > [!NOTE]
   > Comme cet exemple de requête ne fournit pas un modificateur de tri tel que `order by`, l’exécution répétée de cette requête peut produire un ensemble différent de ressources.

1. Mettez à jour l’appel à `getresources` et changez la requête pour `order by` la propriété **Name** :

   ```python
   getresources("Resources | project name, type | limit 5 | order by name asc")
   ```

   > [!NOTE]
   > Comme précédemment, l’exécution répétée de cette requête peut produire un ensemble différent de ressources. L’ordre des commandes de requête est important. Dans cet exemple, `order by` vient après `limit`. Cet ordre de commande limite d’abord les résultats de la requête, puis les classe.

1. Mettez à jour l’appel à `getresources` et changez la requête pour `order by` d’abord la propriété **Name**, puis `limit` les cinq premiers résultats :

   ```python
   getresources("Resources | project name, type | order by name asc | limit 5")
   ```

Si votre environnement ne change pas et si vous exécutez plusieurs fois la requête finale, les résultats retournés sont cohérents et classés en fonction de la propriété **Name**, mais toujours limités aux cinq premiers.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous souhaitez supprimer les bibliothèques installées de votre environnement Python, vous pouvez utiliser la commande suivante :

```bash
# Remove the installed libraries from the Python environment
pip uninstall azure-mgmt-resourcegraph azure-mgmt-resource azure-cli-core
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce démarrage rapide, vous avez ajouté les bibliothèques Resource Graph à votre environnement Python et vous avez exécuté votre première requête. Pour plus d’informations sur le langage Resource Graph, passez à la page des détails du langage de requête.

> [!div class="nextstepaction"]
> [Obtenir plus d’informations sur le langage de requête](./concepts/query-language.md)
