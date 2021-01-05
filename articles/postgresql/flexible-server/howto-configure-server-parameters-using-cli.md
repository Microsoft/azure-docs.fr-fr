---
title: Configurer les paramètres - Azure Database pour PostgreSQL - Serveur flexible
description: Cet article explique comment configurer les paramètres dans le serveur flexible Azure Database pour PostgreSQL à l’aide d’Azure CLI.
author: lfittl-msft
ms.author: lufittl
ms.service: postgresql
ms.devlang: azurecli
ms.topic: how-to
ms.date: 9/22/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: 43b14858cc99cac41e277b03171fd4cac4d6eafa
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90930766"
---
# <a name="customize-server-parameters-for-azure-database-for-postgresql---flexible-server-using-azure-cli"></a>Personnaliser les paramètres de serveur pour le serveur flexible Azure Database pour PostgreSQL à l’aide d’Azure CLI

Il est possible de lister, d’afficher et de mettre à jour les paramètres de configuration d’un serveur Azure PostgreSQL à l’aide de l’interface de ligne de commande (Azure CLI). Un sous-ensemble de paramètres de moteur est exposé au niveau du serveur et peut être modifié. 

## <a name="prerequisites"></a>Prérequis

Pour parcourir ce guide pratique, vous avez besoin des éléments suivants :
- Créez un serveur et une base de données Azure Database pour PostgreSQL en suivant les instructions de la page [Créer une base de données Azure Database pour PostgreSQL](quickstart-create-server-cli.md).
- Installez l’interface de ligne de commande [Azure CLI](/cli/azure/install-azure-cli) sur votre machine ou utilisez [Azure Cloud Shell](../../cloud-shell/overview.md) dans le portail Azure avec votre navigateur.

## <a name="list-server-parameters-for-a-flexible-server"></a>Répertorier les paramètres de serveur pour un serveur flexible

Pour répertorier tous les paramètres modifiables dans un serveur, ainsi que leurs valeurs, exécutez la commande [az postgres server configuration list](/cli/azure/postgres/flexible-server/parameter).

Vous pouvez répertorier les paramètres de serveur pour le serveur **mydemoserver.postgres.database.azure.com** du groupe de ressources **myresourcegroup**.

```azurecli-interactive
az postgres flexible-server parameter list --resource-group myresourcegroup --server-name mydemoserver
```

## <a name="show-server-parameter-details"></a>Afficher les détails des paramètres du serveur

Pour afficher les détails d’un paramètre de configuration particulier pour un serveur, exécutez la commande [az postgres server configuration show](/cli/azure/postgres/flexible-server/parameter).

Cet exemple affiche les détails du paramètre de serveur **log\_min\_messages** pour le serveur **mydemoserver.postgres.database.azure.com** du groupe de ressources **myresourcegroup.**

```azurecli-interactive
az postgres flexible-server parameter show --name log_min_messages --resource-group myresourcegroup --server-name mydemoserver
```

## <a name="modify-server-parameter-value"></a>Modifier la valeur du paramètre de serveur

Vous pouvez également modifier la valeur d’un paramètre de serveur, ce qui a pour effet de mettre à jour la valeur de configuration sous-jacente du moteur du serveur PostgreSQL. Pour mettre à jour le paramètre, utilisez la commande [az postgres flexible-server parameter set](/cli/azure/postgres/flexible-server/parameter). 

Pour mettre à jour le paramètre de serveur **log\_min\_messages** pour le serveur **mydemoserver.postgres.database.azure.com** du groupe de ressources **myresourcegroup.**

```azurecli-interactive
az postgres flexible-server parameter set --name log_min_messages --value INFO --resource-group myresourcegroup --server-name mydemoserver
```

Si vous souhaitez réinitialiser la valeur d’un paramètre, il vous suffit de ne pas définir le paramètre `--value` facultatif. Le service applique alors la valeur par défaut. Dans l’exemple ci-dessus, elle peut se présenter comme suit :

```azurecli-interactive
az postgres flexible-server parameter set --name log_min_messages --resource-group myresourcegroup --server-name mydemoserver
```

Cette commande réinitialise le paramètre **log\_min\_messages** à la valeur par défaut **WARNING**. Pour plus d’informations sur les paramètres de serveur et les valeurs autorisées, consultez la documentation PostgreSQL à la rubrique [Définition des paramètres](https://www.postgresql.org/docs/12/config-setting.html).

## <a name="next-steps"></a>Étapes suivantes

- Pour configurer et accéder aux journaux d’activité du serveur, consultez la rubrique [Journaux d’activité de serveur dans Azure Database pour PostgreSQL](concepts-logging.md)
