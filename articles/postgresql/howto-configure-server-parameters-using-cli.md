---
title: Configurer les paramètres - Azure Database pour PostgreSQL - Serveur unique
description: Cet article explique comment configurer des paramètres Postgres dans Azure Database pour PostgreSQL - Serveur unique à l’aide d’Azure CLI.
author: lfittl-msft
ms.author: lufittl
ms.service: postgresql
ms.devlang: azurecli
ms.topic: how-to
ms.date: 06/19/2019
ms.custom: devx-track-azurecli
ms.openlocfilehash: 4231f348f99073406fcb6a5bef9bf0f84cacf2eb
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96005569"
---
# <a name="customize-server-configuration-parameters-for-azure-database-for-postgresql---single-server-using-azure-cli"></a>Personnaliser les paramètres de configuration du serveur d’Azure Database pour PostgreSQL (serveur unique) à l’aide d’Azure CLI
Il est possible de lister, d’afficher et de mettre à jour les paramètres de configuration d’un serveur Azure PostgreSQL à l’aide de l’interface de ligne de commande (Azure CLI). Un sous-ensemble de configurations de moteur est exposé au niveau du serveur et peut être modifié. 

## <a name="prerequisites"></a>Prérequis
Pour parcourir ce guide pratique, vous avez besoin des éléments suivants :
- Créez un serveur et une base de données Azure Database pour PostgreSQL en suivant les instructions de la page [Créer une base de données Azure Database pour PostgreSQL](quickstart-create-server-database-azure-cli.md).
- Installez l’interface de ligne de commande [Azure CLI](/cli/azure/install-azure-cli) sur votre machine ou utilisez [Azure Cloud Shell](../cloud-shell/overview.md) dans le portail Azure avec votre navigateur.

## <a name="list-server-configuration-parameters-for-azure-database-for-postgresql-server"></a>Répertorier les paramètres de configuration de Base de données Azure pour le serveur PostgreSQL
Pour répertorier tous les paramètres modifiables dans un serveur, ainsi que leurs valeurs, exécutez la commande [az postgres server configuration list](/cli/azure/postgres/server/configuration).

Vous pouvez répertorier les paramètres de configuration du serveur **mydemoserver.postgres.database.azure.com** du groupe de ressources **myresourcegroup**.
```azurecli-interactive
az postgres server configuration list --resource-group myresourcegroup --server mydemoserver
```
## <a name="show-server-configuration-parameter-details"></a>Affichage des détails des paramètres de configuration du serveur
Pour afficher les détails d’un paramètre de configuration particulier pour un serveur, exécutez la commande [az postgres server configuration show](/cli/azure/postgres/server/configuration).

Cet exemple affiche les détails du paramètre de configuration de serveur **log\_min\_messages** pour le serveur **mydemoserver.postgres.database.azure.com** du groupe de ressources **myresourcegroup.**
```azurecli-interactive
az postgres server configuration show --name log_min_messages --resource-group myresourcegroup --server mydemoserver
```
## <a name="modify-server-configuration-parameter-value"></a>Modification de la valeur de paramètre de configuration du serveur
Vous pouvez également modifier la valeur d’un paramètre de configuration du serveur, ce qui a pour effet de mettre à jour la valeur de configuration sous-jacente du moteur du serveur PostgreSQL. Pour mettre à jour la configuration, exécutez la commande [az postgres server configuration set](/cli/azure/postgres/server/configuration). 

Pour mettre à jour le paramètre de configuration de serveur **log\_min\_messages** pour le serveur **mydemoserver.postgres.database.azure.com** du groupe de ressources **myresourcegroup**.
```azurecli-interactive
az postgres server configuration set --name log_min_messages --resource-group myresourcegroup --server mydemoserver --value INFO
```
Si vous souhaitez réinitialiser la valeur d’un paramètre de configuration, il vous suffit de ne pas définir le paramètre `--value` facultatif. Le service applique alors la valeur par défaut. Dans l’exemple ci-dessus, elle peut se présenter comme suit :
```azurecli-interactive
az postgres server configuration set --name log_min_messages --resource-group myresourcegroup --server mydemoserver
```
Cette commande réinitialise la configuration **log\_min\_messages** à la valeur par défaut **WARNING**. Pour plus d’informations sur la configuration du serveur et les valeurs autorisées, consultez la documentation PostgreSQL à la rubrique [Configuration du serveur](https://www.postgresql.org/docs/9.6/static/runtime-config.html).

## <a name="next-steps"></a>Étapes suivantes
- [Découvrez comment redémarrer un serveur](howto-restart-server-cli.md)
- Pour configurer et accéder aux journaux d’activité du serveur, consultez la rubrique [Journaux d’activité de serveur dans Azure Database pour PostgreSQL](concepts-server-logs.md)
