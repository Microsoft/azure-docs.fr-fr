---
title: Script CLI - Mettre à l’échelle un serveur - Azure Database for MariaDB
description: Cet exemple de script CLI met à l’échelle un serveur Azure Database for MariaDB pour obtenir un autre niveau de performances, après analyse des métriques.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.devlang: azurecli
ms.topic: sample
ms.custom: mvc, devx-track-azurecli
ms.date: 12/02/2019
ms.openlocfilehash: 3a099f1a3ab16c33baa8037e727d26f107a593a2
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/20/2021
ms.locfileid: "107785538"
---
# <a name="monitor-and-scale-an-azure-database-for-mariadb-server-using-azure-cli"></a>Superviser et mettre à l’échelle une base de données Azure Database for MariaDB à l’aide d’Azure CLI
Cet exemple de script CLI met à l’échelle le calcul et le stockage d’un serveur Azure Database for MariaDB après interrogation des métriques. Le calcul peut effectuer une opération d’augmentation ou de réduction d'échelle. Le stockage peut uniquement effectuer une opération d’augmentation d'échelle.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

- Cet article nécessite la version 2.0 ou ultérieure d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée. 

## <a name="sample-script"></a>Exemple de script
Mettez à jour le script avec votre ID d’abonnement.
[!code-azurecli-interactive[main](../../../cli_scripts/mariadb/scale-mariadb-server/scale-mariadb-server.sh "Create and scale Azure Database for MariaDB.")]

## <a name="clean-up-deployment"></a>Nettoyer le déploiement
Utilisez la commande suivante pour supprimer le groupe de ressources et toutes les ressources associées après l’exécution du script. 
[!code-azurecli-interactive[main](../../../cli_scripts/mariadb/scale-mariadb-server/delete-mariadb.sh  "Delete the resource group.")]

## <a name="script-explanation"></a>Explication du script
Ce script utilise les commandes décrites dans le tableau suivant :

| **Commande** | **Remarques** |
|---|---|
| [az group create](/cli/azure/group#az_group_create) | Crée un groupe de ressources dans lequel toutes les ressources sont stockées. |
| [az mariadb server create](/cli/azure/mariadb/server#az_mariadb_server_create) | Crée un serveur MariaDB qui héberge les bases de données. |
| [az mariadb server update](/cli/azure/mariadb/server#az_mariadb_server_update) | Met à jour les propriétés du serveur MariaDB. |
| [az monitor metrics list](/cli/azure/monitor/metrics#az_monitor_metrics_list) | Affiche la valeur de métrique pour les ressources. |
| [az group delete](/cli/azure/group#az_group_delete) | Supprime un groupe de ressources, y compris toutes les ressources imbriquées. |

## <a name="next-steps"></a>Étapes suivantes
- En savoir plus sur [le calcul et le stockage Azure Database for MariaDB](../concepts-pricing-tiers.md)
- Essayez d’autres scripts : [Exemples Azure CLI pour Azure Database for MariaDB](../sample-scripts-azure-cli.md)
- En savoir plus sur [Azure CLI](/cli/azure)
