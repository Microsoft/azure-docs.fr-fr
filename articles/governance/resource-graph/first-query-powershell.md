---
title: 'Démarrage rapide : Votre première requête PowerShell'
description: Dans ce guide de démarrage rapide, vous suivez les étapes pour activer le module Resource Graph pour Azure PowerShell et vous exécutez votre première requête.
ms.date: 10/14/2020
ms.topic: quickstart
ms.openlocfilehash: b810f4af319d3a5212d35e3387ab35b9fa40aafa
ms.sourcegitcommit: 1b47921ae4298e7992c856b82cb8263470e9e6f9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/14/2020
ms.locfileid: "92057431"
---
# <a name="quickstart-run-your-first-resource-graph-query-using-azure-powershell"></a>Démarrage rapide : Exécuter votre première requête Resource Graph à l’aide d’Azure PowerShell

La première étape est de vérifier que le module Azure Resource Graph pour Azure PowerShell est installé. Ce guide de démarrage rapide décrit le processus d’ajout du module à votre installation Azure PowerShell.

Au terme de ce processus, vous aurez ajouté le module à l’installation Azure PowerShell de votre choix et vous pourrez exécuter votre première requête Resource Graph.

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

[!INCLUDE [cloud-shell-try-it.md](../../../includes/cloud-shell-try-it.md)]

## <a name="add-the-resource-graph-module"></a>Ajouter le module Resource Graph

Pour permettre à Azure PowerShell d’interroger Azure Resource Graph, vous devez ajouter le module. Vous pouvez utiliser ce module avec PowerShell installé localement, avec [Azure Cloud Shell](https://shell.azure.com) ou avec l’[image Docker PowerShell](https://hub.docker.com/_/microsoft-powershell).

### <a name="base-requirements"></a>Configuration de base requise

Le module Azure Resource Graph nécessite les logiciels suivants :

- Azure PowerShell 1.0.0 ou version ultérieure. S’il n’est pas installé, suivez [ces instructions](/powershell/azure/install-az-ps).

- PowerShellGet 2.0.1 ou une version ultérieure. S’il n’est pas installé ou mis à jour, suivez [ces instructions](/powershell/scripting/gallery/installing-psget).

### <a name="install-the-module"></a>Installer le module

Le module Resource Graph pour PowerShell est **Az.ResourceGraph**.

1. À partir d’une invite PowerShell **d’administration**, exécutez la commande suivante :

   ```azurepowershell-interactive
   # Install the Resource Graph module from PowerShell Gallery
   Install-Module -Name Az.ResourceGraph
   ```

1. Vérifiez que le module a été importé et qu’il s’agit de la dernière version (0.7.5) :

   ```azurepowershell-interactive
   # Get a list of commands for the imported Az.ResourceGraph module
   Get-Command -Module 'Az.ResourceGraph' -CommandType 'Cmdlet'
   ```

## <a name="run-your-first-resource-graph-query"></a>Exécuter votre première requête Resource Graph

Une fois le module Azure PowerShell ajouté à l’environnement de votre choix, vous pouvez exécuter une requête Resource Graph simple. La requête retourne les cinq premières ressources Azure avec le nom (**Name**) et le **Type** de chaque ressource.

1. Exécutez votre première requête Azure Resource Graph à l’aide de l’applet de commande `Search-AzGraph` :

   ```azurepowershell-interactive
   # Login first with Connect-AzAccount if not using Cloud Shell

   # Run Azure Resource Graph query
   Search-AzGraph -Query 'Resources | project name, type | limit 5'
   ```

   > [!NOTE]
   > Comme cet exemple de requête ne fournit pas un modificateur de tri tel que `order by`, l’exécution répétée de cette requête peut produire un ensemble différent de ressources.

1. Mettez à jour la requête pour la trier (`order by`) en fonction du nom (**name**) de la propriété :

   ```azurepowershell-interactive
   # Run Azure Resource Graph query with 'order by'
   Search-AzGraph -Query 'Resources | project name, type | limit 5 | order by name asc'
   ```

   > [!NOTE]
   > Comme précédemment, l’exécution répétée de cette requête peut produire un ensemble différent de ressources. L’ordre des commandes de requête est important. Dans cet exemple, `order by` vient après `limit`. Cet ordre de commande limite d’abord les résultats de la requête, puis les classe.

1. Mettez à jour la requête pour d’abord trier (`order by`) les résultats en fonction de la propriété **name**, puis les limiter (`limit`) aux cinq premiers :

   ```azurepowershell-interactive
   # Run Azure Resource Graph query with `order by` first, then with `limit`
   Search-AzGraph -Query 'Resources | project name, type | order by name asc | limit 5'
   ```

Si votre environnement ne change pas et si vous exécutez plusieurs fois la requête finale, les résultats retournés sont cohérents et classés en fonction de la propriété **Name**, mais toujours limités aux cinq premiers.

> [!NOTE]
> Si la requête ne retourne aucun résultat à partir d’un abonnement auquel vous avez déjà accès, notez que l’applet de commande `Search-AzGraph` porte par défaut sur les abonnements du contexte par défaut. Pour voir la liste des ID d’abonnement qui font partie du contexte par défaut, exécutez `(Get-AzContext).Account.ExtendedProperties.Subscriptions`. Si vous souhaitez effectuer une recherche sur tous les abonnements auxquels vous avez accès, vous pouvez définir PSDefaultParameterValues pour l’applet de commande `Search-AzGraph` en exécutant `$PSDefaultParameterValues=@{"Search-AzGraph:Subscription"= $(Get-AzSubscription).ID}`.
   
## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous souhaitez supprimer le module Resource Graph de votre environnement Azure PowerShell, vous pouvez le faire à l’aide de la commande suivante :

```azurepowershell-interactive
# Remove the Resource Graph module from the current session
Remove-Module -Name 'Az.ResourceGraph'

# Uninstall the Resource Graph module from the environment
Uninstall-Module -Name 'Az.ResourceGraph'
```

> [!NOTE]
> Cette opération ne supprime pas le fichier du module téléchargé précédemment. Elle le supprime uniquement de la session PowerShell en cours d’exécution.

## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez ajouté le module Resource Graph à votre environnement Azure PowerShell et vous avez exécuté votre première requête. Pour plus d’informations sur le langage Resource Graph, passez à la page des détails du langage de requête.

> [!div class="nextstepaction"]
> [Obtenir plus d’informations sur le langage de requête](./concepts/query-language.md)