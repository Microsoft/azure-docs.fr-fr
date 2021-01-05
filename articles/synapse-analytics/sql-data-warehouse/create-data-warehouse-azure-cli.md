---
title: 'Démarrage rapide : Créer un pool Synapse SQL avec Azure CLI'
description: Créez rapidement un pool Synapse SQL avec une règle de pare-feu au niveau du serveur à l’aide d’Azure CLI.
services: synapse-analytics
author: kevinvngo
ms.service: synapse-analytics
ms.topic: quickstart
ms.subservice: sql-dw
ms.date: 11/20/2020
ms.author: Kevin
ms.custom: azure-synapse
ms.openlocfilehash: 59195bba69bb343e55cfcb7342400d93dcce60c0
ms.sourcegitcommit: 03c0a713f602e671b278f5a6101c54c75d87658d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/19/2020
ms.locfileid: "94932956"
---
# <a name="quickstart-create-a-synapse-sql-pool-with-azure-cli"></a>Démarrage rapide : Créer un pool Synapse SQL avec Azure CLI

Créez un pool Synapse SQL (entrepôt de données) dans Azure Synapse Analytics à l’aide d’Azure CLI.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

## <a name="getting-started"></a>Prise en main

Utilisez ces commandes pour vous connecter à Azure et configurer un groupe de ressources.

1. Si vous utilisez une installation locale, exécutez la commande [az login](/cli/azure/reference-index#az_login) pour vous connecter à Azure :

   ```azurecli
   az login
   ```

1. Si nécessaire, utilisez la commande [az account set](/cli/azure/account#az_account_set) pour sélectionner votre abonnement :

   ```azurecli
   az account set --subscription 00000000-0000-0000-0000-000000000000
   ```

1. Exécutez la commande [az group create](/cli/azure/group#az_group_create) pour créer un groupe de ressources :

   ```azurecli
   az group create --name myResourceGroup --location WestEurope
   ```

1. Créez un [serveur SQL logique](../../azure-sql/database/logical-servers.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json) à l’aide de la commande [az sql server create](/cli/azure/sql/server#az_sql_server_create) :

   ```azurecli
   az sql server create --resource-group myResourceGroup --name mysqlserver \
      --admin-user ServerAdmin --admin-password ChangeYourAdminPassword1
   ```

   Un serveur contient un groupe de bases de données gérées en tant que groupe.

## <a name="configure-a-server-level-firewall-rule"></a>Configurer une règle de pare-feu au niveau du serveur

Créez une [règle de pare-feu au niveau du serveur](../../azure-sql/database/firewall-configure.md?toc=/azure/synapse-analytics/sql-data-warehouse/toc.json&bc=/azure/synapse-analytics/sql-data-warehouse/breadcrumb/toc.json). Une règle de pare-feu au niveau du serveur permet à une application externe, telle que SQL Server Management Studio ou l’utilitaire SQLCMD, de se connecter à un pool SQL à travers le pare-feu du service de pool SQL.

Exécutez la commande [az sql server firewall-rule create](/cli/azure/sql/server/firewall-rule#az_sql_server_firewall_rule_create) pour créer une règle de pare-feu :

```azurecli
az sql server firewall-rule create --resource-group myResourceGroup --name AllowSome \
   --server mysqlserver --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0
```

Dans cet exemple, le pare-feu n’est ouvert qu’à d’autres ressources Azure. Pour activer la connectivité externe, remplacez l’adresse IP par une adresse correspondant à votre environnement. Pour ouvrir toutes les adresses IP, utilisez 0.0.0.0 comme adresse IP de début et 255.255.255.255 comme adresse de fin.

> [!NOTE]
> Les points de terminaison SQL communiquent via le port 1433. Si vous essayez de vous connecter à partir d’un réseau d’entreprise, le trafic sortant sur le port 1433 peut ne pas être autorisé par le pare-feu de votre réseau. Dans ce cas, vous ne pourrez pas vous connecter à votre serveur, sauf si votre service informatique ouvre le port 1433.
>

## <a name="create-and-manage-your-sql-pool"></a>Créer et gérer votre pool SQL

Créez le pool SQL. Cet exemple utilise DW100c comme objectif de service, ce qui correspond à un point de départ peu coûteux pour votre pool SQL.

> [!NOTE]
> Vous avez besoin d’un espace de travail existant. Pour plus d’informations, consultez [Démarrage rapide : Créez un espace de travail Azure Synapse avec Azure CLI](../quickstart-create-workspace-cli.md).

Utilisez la commande [az synapse sql pool create](/cli/azure/ext/synapse/synapse/sql/pool#ext_synapse_az_synapse_sql_pool_create) pour créer le pool SQL :

```azurecli
az synapse sql pool create --resource-group myResourceGroup --name mySampleDataWarehouse \
   --performance-level "DW1000c" --workspace-name testsynapseworkspace
```

Pour plus d’informations sur les options de paramètre, consultez [az synapse sql pool](/cli/azure/ext/synapse/synapse/sql/pool).

Vous pouvez voir vos pools SQL à l’aide de la commande [az synapse sql pool list](/cli/azure/ext/synapse/synapse/sql/pool#ext_synapse_az_synapse_sql_pool_list) :

```azurecli
az synapse sql pool list --resource-group myResourceGroup --workspace-name testsynapseworkspace
```

Utilisez la commande [az synapse sql pool update](/cli/azure/ext/synapse/synapse/sql/pool#ext_synapse_az_synapse_sql_pool_update) pour mettre à jour un pool existant :

```azurecli
az synapse sql pool update --resource-group myResourceGroup --name mySampleDataWarehouse \
   --workspace-name testsynapseworkspace
```

Utilisez la commande [az synapse sql pool pause](/cli/azure/ext/synapse/synapse/sql/pool#ext_synapse_az_synapse_sql_pool_pause) pour suspendre votre pool :

```azurecli
az synapse sql pool pause --resource-group myResourceGroup --name mySampleDataWarehouse \
   --workspace-name testsynapseworkspace
```

Utilisez la commande [az synapse sql pool resume](/cli/azure/ext/synapse/synapse/sql/pool#ext_synapse_az_synapse_sql_pool_resume) pour démarrer un pool suspendu :

```azurecli
az synapse sql pool resume --resource-group myResourceGroup --name mySampleDataWarehouse \
   --workspace-name testsynapseworkspace
```

Pour supprimer un pool SQL existant, utilisez la commande [az synapse sql pool delete](/cli/azure/ext/synapse/synapse/sql/pool#ext_synapse_az_synapse_sql_pool_delete) :

```azurecli
az synapse sql pool delete --resource-group myResourceGroup --name mySampleDataWarehouse \
   --workspace-name testsynapseworkspace
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

D’autres tutoriels de démarrage rapide de cette collection reposent sur ce démarrage rapide.

> [!TIP]
> Si vous prévoyez de continuer avec d’autres tutoriels de démarrage rapide, ne nettoyez pas les ressources créées dans ce guide de démarrage rapide. Sinon, utilisez la commande [az group delete](/cli/azure/group#az_group_delete) pour supprimer toutes les ressources créées dans le cadre de ce guide de démarrage rapide.
>

```azurecli
az group delete --ResourceGroupName MyResourceGroup
```

## <a name="next-steps"></a>Étapes suivantes

Vous avez créé un pool SQL, une règle de pare-feu et vous êtes connecté à votre pool SQL. Pour en savoir plus, consultez l’article [Charger des données dans un pool SQL](load-data-from-azure-blob-storage-using-polybase.md).
