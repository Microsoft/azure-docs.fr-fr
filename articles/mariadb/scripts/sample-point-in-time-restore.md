---
title: Script CLI - Restaurer un serveur - Azure Database for MariaDB
description: Cet exemple de script Azure CLI montre comment restaurer un serveur Azure Database for MariaDB et ses bases de données à un point antérieur dans le temps.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.devlang: azurecli
ms.topic: sample
ms.custom: mvc, devx-track-azurecli
ms.date: 12/02/2019
ms.openlocfilehash: 3c56c7d933f840e4418bd481cce0db1bc2216e3f
ms.sourcegitcommit: 4b0e424f5aa8a11daf0eec32456854542a2f5df0
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 04/20/2021
ms.locfileid: "107785610"
---
# <a name="restore-an-azure-database-for-mariadb-server-using-azure-cli"></a>Restaurer un serveur Azure Database for MariaDB à l’aide d’Azure CLI
Cet exemple de script Azure CLI permet de restaurer un serveur Azure Database for MariaDB à un point antérieur dans le temps.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../../includes/azure-cli-prepare-your-environment.md)]

- Cet article nécessite la version 2.0 ou ultérieure d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée. 

## <a name="sample-script"></a>Exemple de script
Dans cet exemple de script, modifiez les lignes en surbrillance pour mettre à jour le nom d’utilisateur et le mot de passe d’administrateur et utiliser les vôtres. Remplacez l’ID d’abonnement utilisé dans les commandes `az monitor` par votre propre ID d’abonnement.
[!code-azurecli-interactive[main](../../../cli_scripts/mariadb/backup-restore-pitr/backup-restore.sh?highlight=15-16 "Restore Azure Database for MariaDB.")]

## <a name="clean-up-deployment"></a>Nettoyer le déploiement
Utilisez la commande suivante pour supprimer le groupe de ressources et toutes les ressources associées après l’exécution du script. 
[!code-azurecli-interactive[main](../../../cli_scripts/mariadb/backup-restore-pitr/delete-mariadb.sh  "Delete the resource group.")]

## <a name="script-explanation"></a>Explication du script
Ce script utilise les commandes décrites dans le tableau suivant :

| **Commande** | **Remarques** |
|---|---|
| [az group create](/cli/azure/group#az_group_create) | Crée un groupe de ressources dans lequel toutes les ressources sont stockées. |
| [az mariadb server create](/cli/azure/mariadb/server#az_mariadb_server_create) | Crée un serveur MariaDB qui héberge les bases de données. |
| [az mariadb server restore](/cli/azure/mariadb/server#az_mariadb_server_restore) | Restaure un serveur à partir de la sauvegarde. |
| [az group delete](/cli/azure/group#az_group_delete) | Supprime un groupe de ressources, y compris toutes les ressources imbriquées. |

## <a name="next-steps"></a>Étapes suivantes
- En savoir plus sur Azure CLI : [Documentation d’Azure CLI](/cli/azure)
- Essayez d’autres scripts : [Exemples Azure CLI pour Azure Database for MariaDB](../sample-scripts-azure-cli.md)
