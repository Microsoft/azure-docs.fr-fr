---
title: Configurer les paramètres de serveur – Azure CLI – Azure Database pour MySQL – Serveur flexible
description: Cet article décrit comment configurer les paramètres de service dans Azure Database pour MySQL – Serveur flexible à l’aide de l’utilitaire en ligne de commande Azure CLI.
author: ambhatna
ms.author: ambhatna
ms.service: mysql
ms.devlang: azurecli
ms.topic: how-to
ms.date: 11/10/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 58e7c024d6494aee745884997e42b527c51ab237
ms.sourcegitcommit: b4880683d23f5c91e9901eac22ea31f50a0f116f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94489537"
---
# <a name="configure-server-parameters-in-azure-database-for-mysql-flexible-server-using-the-azure-cli"></a>Configurer les paramètres de serveur dans Azure Database pour MySQL – Serveur flexible à l’aide d’Azure CLI

> [!IMPORTANT] 
> Azure Database pour MySQL - Serveur flexible est actuellement en préversion publique.

Vous pouvez répertorier, afficher et mettre à jour les paramètres d’un serveur flexible Azure Database pour MySQL à l’aide d’Azure CLI, l’utilitaire en ligne de commande Azure. Les paramètres de serveur sont configurés avec la valeur par défaut et la valeur recommandée lors de la création du serveur.  

Cet article explique comment répertorier, afficher et mettre à jour les paramètres du serveur à l’aide d’Azure CLI.

>[!Note]
> Les paramètres de serveur peuvent être mis à jour globalement au niveau du serveur à partir d’[Azure CLI](./how-to-configure-server-parameters-cli.md) ou du [portail Azure](./how-to-configure-server-parameters-portal.md).

## <a name="prerequisites"></a>Prérequis
Pour parcourir ce guide pratique, vous avez besoin des éléments suivants :
- Un [serveur flexible Azure Database pour MySQL](quickstart-create-server-cli.md).
- L’utilitaire en ligne de commande [Azure CLI](/cli/azure/install-azure-cli) ou Azure Cloud Shell dans le navigateur.

## <a name="list-server-parameters-for-azure-database-for-mysql-flexible-server"></a>Répertorier les paramètres de serveur pour Azure Database pour MySQL – Serveur flexible
Pour répertorier tous les paramètres dans un serveur, ainsi que leurs valeurs, exécutez la commande [az mysql flexible-server parameter list](/cli/azure/mysql/flexible-server/parameter).

Vous pouvez répertorier les paramètres de serveur pour le serveur **mydemoserver.mysql.database.azure.com** du groupe de ressources **myresourcegroup**.
```azurecli-interactive
az mysql flexible-server parameter list --resource-group myresourcegroup --server-name mydemoserver
```
Pour obtenir la définition de chacun des paramètres répertoriés, consultez la section de référence MySQL dans [Server System Variables (Variables système du serveur)](https://dev.mysql.com/doc/refman/5.7/en/server-system-variables.html).

## <a name="show-server-parameter-details"></a>Afficher les détails des paramètres du serveur
Pour afficher les détails d’un paramètre particulier pour un serveur, exécutez la commande [az mysql flexible-server parameter show](/cli/azure/mysql/flexible-server/parameter).

Cet exemple affiche les détails du paramètre de serveur **slow\_query\_log** pour le serveur **mydemoserver.mysql.database.azure.com** du groupe de ressources **myresourcegroup**.
```azurecli-interactive
az mysql flexible-server parameter show --name slow_query_log --resource-group myresourcegroup --server-name mydemoserver
```
## <a name="modify-a-server-parameter-value"></a>Modifier une valeur de paramètre de serveur
Vous pouvez également modifier la valeur d’un paramètre de serveur, ce qui a pour effet de mettre à jour la valeur de configuration sous-jacente du moteur du serveur MySQL. Pour mettre à jour le paramètre de serveur, utilisez la commande [az mysql flexible-server parameter set](/cli/azure/mysql/flexible-server/parameter). 

Pour mettre à jour le paramètre de serveur **slow\_query\_log** du serveur **mydemoserver.mysql.database.azure.com** du groupe de ressources **myresourcegroup**.
```azurecli-interactive
az mysql flexible-server parameter set --name slow_query_log --resource-group myresourcegroup --server-name mydemoserver --value ON
```
Si vous souhaitez réinitialiser la valeur d’un paramètre, omettez le paramètre `--value` facultatif. Le service applique alors la valeur par défaut. Pour l’exemple ci-dessus, on aurait ce qui suit :
```azurecli-interactive
az mysql flexible-server parameter set --name slow_query_log --resource-group myresourcegroup --server-name mydemoserver
```
Ce code réinitialise le paramètre **slow\_query\_log** à la valeur par défaut **OFF**. 

## <a name="setting-non-modifiable-server-parameters"></a>Définition des paramètres de serveur non modifiables

Si le paramètre de serveur que vous souhaitez mettre à jour n’est pas modifiable, vous pouvez éventuellement définir le paramètre au niveau de la connexion avec `init_connect`. Cela définit les paramètres de serveur pour chaque client qui se connecte. 

Mettez à jour le paramètre de serveur **init\_connect** du serveur **mydemoserver.mysql.database.azure.com** dans le groupe de ressources **myresourcegroup** pour définir les valeurs comme jeu de caractères.
```azurecli-interactive
az mysql flexible-server parameter set --name init_connect --resource-group myresourcegroup --server-name mydemoserver --value "SET character_set_client=utf8;SET character_set_database=utf8mb4;SET character_set_connection=latin1;SET character_set_results=latin1;"
```
>[!Note]
> `init_connect` peut être utilisé pour modifier des paramètres qui ne nécessitent pas de privilèges de super utilisateur (SUPER) au niveau de la session. Pour vérifier si vous pouvez définir le paramètre avec `init_connect`, exécutez la commande `set session parameter_name=YOUR_DESIRED_VALUE;` et, si vous obtenez une erreur **Accès refusé. Vous devez avoir des privilèges SUPER** , cela indique que vous n’êtes pas autorisé à définir le paramètre avec « init_connect ».

## <a name="working-with-the-time-zone-parameter"></a>Utilisation avec le paramètre de fuseau horaire

### <a name="populating-the-time-zone-tables"></a>Remplissage des tables de fuseaux horaires

Les tables de fuseaux horaires sur votre serveur peuvent être remplies en appelant la procédure stockée `mysql.az_load_timezone` à partir d’un outil tel que la ligne de commande MySQL ou MySQL Workbench.

> [!NOTE]
> Si vous exécutez la commande `mysql.az_load_timezone` à partir de MySQL Workbench, vous devrez peut-être d’abord désactiver le mode de mise à jour sécurisée à l’aide de `SET SQL_SAFE_UPDATES=0;`.

```sql
CALL mysql.az_load_timezone();
```

> [!IMPORTANT]
> Vous devez redémarrer le serveur pour vous assurer que les tables des fuseaux horaires sont correctement remplies.<!-- fIX me To restart the server, use the [Azure portal](howto-restart-server-portal.md) or [CLI](howto-restart-server-cli.md). -->

Pour afficher les valeurs de fuseau horaire disponibles, exécutez la commande suivante :

```sql
SELECT name FROM mysql.time_zone_name;
```

### <a name="setting-the-global-level-time-zone"></a>Définition du fuseau horaire global

Le fuseau horaire global peut être défini à l’aide de la commande [az mysql flexible-server parameter set](/cli/azure/mysql/flexible-server/parameter).

La commande suivante met à jour le paramètre de serveur **time\_zone** du serveur **mydemoserver.mysql.database.azure.com** dans le groupe de ressources **myresourcegroup** sur **US/Pacific**.

```azurecli-interactive
az mysql flexible-server parameter set --name time_zone --resource-group myresourcegroup --server-name mydemoserver --value "US/Pacific"
```

### <a name="setting-the-session-level-time-zone"></a>Définition du fuseau horaire de la session

Le fuseau horaire de la session peut être défini en exécutant la commande `SET time_zone` à partir d’un outil tel que la ligne de commande MySQL ou MySQL Workbench. L’exemple ci-dessous définit le fuseau horaire **US/Pacific**.  

```sql
SET time_zone = 'US/Pacific';
```

Reportez-vous à la documentation de MySQL sur les [fonction de date et heure](https://dev.mysql.com/doc/refman/5.7/en/date-and-time-functions.html#function_convert-tz).


## <a name="next-steps"></a>Étapes suivantes

- Guide pratique pour configurer des [paramètres de serveur dans le portail Azure](./how-to-configure-server-parameters-portal.md)
