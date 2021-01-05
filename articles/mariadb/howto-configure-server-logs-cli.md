---
title: Accéder aux journaux des requêtes lentes - Azure CLI - Azure Database for MariaDB
description: Cet article décrit comment accéder aux journaux des requêtes lentes dans Azure Database for MariaDB à l’aide de l’utilitaire de ligne de commande Azure CLI.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.devlang: azurecli
ms.topic: how-to
ms.date: 4/13/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: cea1571a48afe00350dde247c3c10e222a446247
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94539870"
---
# <a name="configure-and-access-azure-database-for-maria-db-slow-query-logs-by-using-azure-cli"></a>Configurer et accéder au journaux des requêtes lentes dans Azure Database pour Maria DB en utilisant Azure CLI

Vous pouvez télécharger les journaux des requêtes lentes Azure Database for MariaDB à l’aide d’Azure CLI, l’utilitaire en ligne de commande Azure.

## <a name="prerequisites"></a>Prérequis
Pour parcourir ce guide pratique, vous avez besoin des éléments suivants :
- [Serveur Azure Database for MariaDB](quickstart-create-mariadb-server-database-using-azure-cli.md)
- [Azure CLI](/cli/azure/install-azure-cli) ou Azure Cloud Shell dans le navigateur

## <a name="configure-logging"></a>Configuration de la journalisation
Vous pouvez configurer le serveur afin d’accéder au journal des requêtes lentes MySQL, comme suit :
1. Activez la journalisation des requêtes lentes en définissant le paramètre **slow\_query\_log** sur ON.
2. Sélectionnez l’emplacement de sortie des journaux avec **journal\_sortie**. Pour envoyer des journaux vers le stockage local et vers les journaux de diagnostic Azure Monitor, sélectionnez **Fichier**. Pour envoyer des journaux uniquement aux journaux de Azure Monitor, sélectionnez **Aucun**
3. Réglez les autres paramètres tels que **durée\_longue\_d’une requête** et **instructions\_admin\_lentes\_du journal**.

Pour savoir comment définir la valeur de ces paramètres via Azure CLI, consultez [Comment configurer les paramètres du serveur](howto-configure-server-parameters-cli.md).

Par exemple, la commande CLI suivante active le journal des requêtes lentes, définit la durée de requête longue sur 10 secondes et désactive la journalisation de l’instruction admin lente. Enfin, elle répertorie les options de configuration à vérifier.
```azurecli-interactive
az mariadb server configuration set --name slow_query_log --resource-group myresourcegroup --server mydemoserver --value ON
az mariadb server configuration set --name log_output --resource-group myresourcegroup --server mydemoserver --value FILE
az mariadb server configuration set --name long_query_time --resource-group myresourcegroup --server mydemoserver --value 10
az mariadb server configuration set --name log_slow_admin_statements --resource-group myresourcegroup --server mydemoserver --value OFF
az mariadb server configuration list --resource-group myresourcegroup --server mydemoserver
```

## <a name="list-logs-for-azure-database-for-mariadb-server"></a>Répertorier les journaux d’activité de serveur pour Azure Database for MariaDB
Si **log_output** est configurée sur « fichier », vous pouvez accéder aux journaux directement à partir du stockage local du serveur. Pour lister les fichiers journaux des requêtes lentes disponibles pour votre serveur, exécutez la commande [az mariadb server-logs list](/cli/azure/mariadb/server-logs#az-mariadb-server-logs-list).

Vous pouvez répertorier les fichiers journaux pour le serveur **mydemoserver.mariadb.database.azure.com** sous le groupe de ressources **myresourcegroup**. Puis, dirigez la liste des fichiers journaux vers un fichier texte appelé **log\_files\_list.txt**.
```azurecli-interactive
az mariadb server-logs list --resource-group myresourcegroup --server mydemoserver > log_files_list.txt
```
## <a name="download-logs-from-the-server"></a>Télécharger des journaux d’activité à partir du serveur
Si **log_output** est configurée sur « Fichier », vous pouvez télécharger des fichiers journaux individuels à partir de votre serveur avec la commande [az mariadb server-logs download](/cli/azure/mariadb/server-logs#az-mariadb-server-logs-download).

Utilisez l’exemple suivant pour télécharger le fichier journal spécifique pour le serveur **mydemoserver.mariadb.database.azure.com** sous le groupe de ressources **myresourcegroup** dans votre environnement local.
```azurecli-interactive
az mariadb server-logs download --name mysql-slow-mydemoserver-2018110800.log --resource-group myresourcegroup --server mydemoserver
```

## <a name="next-steps"></a>Étapes suivantes
- Découvrez les [journaux des requêtes lentes dans Azure Database for MariaDB](concepts-server-logs.md).
