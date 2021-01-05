---
title: Accéder aux journaux d'audit - Azure CLI - Azure Database for MariaDB
description: Cet article explique comment configurer et consulter les journaux d'audit dans Azure Database for MariaDB à partir d'Azure CLI.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.topic: how-to
ms.date: 6/24/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: c1d446d8ee2863077ad84c361876758336f5a3cb
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94540958"
---
# <a name="configure-and-access-azure-database-for-maria-db-audit-logs-in-the-azure-cli"></a>Configurer et consulter les journaux d’audit Azure Database pour Maria DB dans l’interface de ligne de commande Azure

Vous pouvez configurer les [journaux d'audit Azure Database for MariaDB](concepts-audit-logs.md) à partir d'Azure CLI.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

Pour suivre ce guide :

- Vous avez besoin d’un [serveur Azure Database for MariaDB](quickstart-create-mariadb-server-database-using-azure-portal.md).

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

- Cet article nécessite la version 2.0 ou ultérieure d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="configure-audit-logging"></a>Configurer l’enregistrement d’audit

>[!IMPORTANT]
> Il est recommandé de ne consigner que les types d’événements et les utilisateurs requis à des fins d’audit pour garantir que les performances de votre serveur ne sont pas fortement affectées.

Activez et configurez l'enregistrement d'audit en procédant comme suit : 

1. Activez les journaux d’audit en définissant le paramètre **audit_logs_enabled** sur « ON ». 
    ```azurecli-interactive
    az mariadb server configuration set --name audit_log_enabled --resource-group myresourcegroup --server mydemoserver --value ON
    ```

1. Sélectionnez les [types d’événements](concepts-audit-logs.md#configure-audit-logging) à enregistrer en mettant à jour le paramètre **audit_log_events**.
    ```azurecli-interactive
    az mariadb server configuration set --name audit_log_events --resource-group myresourcegroup --server mydemoserver --value "ADMIN,CONNECTION"
    ```

1. Ajoutez des utilisateurs MariaDB à exclure de la journalisation en mettant à jour le paramètre **audit_log_exclude_users**. Spécifiez les utilisateurs en fournissant leur nom d’utilisateur MariaDB.
    ```azurecli-interactive
    az mariadb server configuration set --name audit_log_exclude_users --resource-group myresourcegroup --server mydemoserver --value "azure_superuser"
    ```

1. Ajoutez les utilisateurs MariaDB à exclure de l'enregistrement en mettant à jour le paramètre **audit_log_include_users**. Spécifiez les utilisateurs en fournissant leur nom d’utilisateur MariaDB.
    ```azurecli-interactive
    az mariadb server configuration set --name audit_log_include_users --resource-group myresourcegroup --server mydemoserver --value "sampleuser"
    ```

## <a name="next-steps"></a>Étapes suivantes

- Apprenez-en davantage sur les [journaux d'audit](concepts-audit-logs.md) dans Azure Database for MariaDB
- Apprenez à configurer les journaux d'audit à partir du [portail Azure](howto-configure-audit-logs-portal.md)
