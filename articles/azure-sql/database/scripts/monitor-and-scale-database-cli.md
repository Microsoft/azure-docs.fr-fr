---
title: 'Azure CLI : Superviser et mettre à l’échelle une base de données unique dans Azure SQL Database'
description: Utilisez un exemple de script Azure CLI pour superviser et mettre à l’échelle une base de données unique dans Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: performance
ms.custom: sqldbrb=1, devx-track-azurecli
ms.devlang: azurecli
ms.topic: sample
author: WilliamDAssafMSFT
ms.author: wiassaf
ms.reviewer: sstein
ms.date: 06/25/2019
ms.openlocfilehash: 0a72bea08052e5cd17502a21a98b41cb5582a3b9
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96493625"
---
# <a name="use-the-azure-cli-to-monitor-and-scale-a-single-database-in-azure-sql-database"></a>Utiliser Azure CLI pour superviser et mettre à l’échelle une base de données unique dans Azure SQL Database

[!INCLUDE[appliesto-sqldb](../../includes/appliesto-sqldb.md)]

Cet exemple de script Azure CLI met à l’échelle une base de données unique dans Azure SQL Database vers une autre taille de calcul après avoir déterminé la taille de la base de données.

Si vous choisissez d’installer et d’utiliser l’interface de ligne de commande Azure localement, vous devez exécuter Azure CLI version 2.0 ou ultérieure pour poursuivre la procédure décrite dans cet article. Exécutez `az --version` pour trouver la version. Si vous devez effectuer une installation ou une mise à niveau, consultez [Installer Azure CLI](/cli/azure/install-azure-cli).

## <a name="sample-script"></a>Exemple de script

### <a name="sign-in-to-azure"></a>Connexion à Azure

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]

```azurecli-interactive
$subscription = "<subscriptionId>" # add subscription here

az account set -s $subscription # ...or use 'az login'
```

### <a name="run-the-script"></a>Exécuter le script

[!code-azurecli-interactive[main](../../../../cli_scripts/sql-database/monitor-and-scale-database/monitor-and-scale-database.sh "Monitor and scale a database in Azure SQL Database")]

> [!TIP]
> Utilisez [az sql db op list](/cli/azure/sql/db/op?#az-sql-db-op-list) pour obtenir la liste des opérations effectuées sur la base de données et [az sql db op cancel](/cli/azure/sql/db/op#az-sql-db-op-cancel) pour annuler une opération de mise à jour sur la base de données.

### <a name="clean-up-deployment"></a>Nettoyer le déploiement

Utilisez la commande suivante pour supprimer le groupe de ressources et toutes les ressources associées.

```azurecli-interactive
az group delete --name $resource
```

## <a name="sample-reference"></a>Informations de référence sur l’exemple

Ce script utilise les commandes suivantes. Chaque commande du tableau renvoie à une documentation spécifique.

| Script | Description |
|---|---|
| [az sql server](/cli/azure/sql/server) | Commandes de serveur. |
| [az sql db show-usage](/cli/azure/sql#az-sql-show-usage) | Affiche les données de taille d’une base de données. |

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’interface Azure CLI, consultez la [documentation relative à l’interface Azure CLI](/cli/azure).

Vous trouverez des exemples de script d’interface CLI supplémentaires dans [Exemples de scripts Azure CLI](../az-cli-script-samples-content-guide.md).
