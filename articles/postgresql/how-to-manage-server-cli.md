---
title: Gérer un serveur – Azure CLI – Azure Database pour PostgreSQL
description: Découvrez comment gérer un serveur Azure Database pour PostgreSQL à partir d’Azure CLI.
author: ajlam
ms.author: andrela
ms.service: mysql
ms.topic: how-to
ms.date: 9/22/2020
ms.openlocfilehash: d99634388b9c4db99c996cfccb9bb5f12682f217
ms.sourcegitcommit: 3bcce2e26935f523226ea269f034e0d75aa6693a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92490116"
---
# <a name="manage-an-azure-database-for-postgresql-single-server-using-the-azure-cli"></a>Gérer un serveur unique Azure Database pour PostgreSQL à l’aide d’Azure CLI

Cet article vous explique comment gérer vos serveurs uniques déployés sur Azure. Les tâches de gestion incluent notamment la mise à l’échelle du calcul et du stockage, la réinitialisation de mot de passe et l’affichage des informations relatives au serveur.

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer. Cet article nécessite que vous exécutiez localement Azure CLI version 2.0 ou ultérieure. Pour afficher la version installée, exécutez la commande `az --version`. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI](/cli/azure/install-azure-cli).

Vous devrez vous connecter à votre compte à l’aide de la commande [az login](/cli/azure/reference-index#az-login). Notez la propriété **id** , qui fait référence à l’ **ID d’abonnement** pour votre compte Azure.

```azurecli-interactive
az login
```

Sélectionnez l’abonnement spécifique sous votre compte à l’aide de la commande [az account set](/cli/azure/account). Notez la valeur **id** issue de la commande **az login**  ; vous devez vous en servir comme valeur de l’argument **subscription** dans la commande. Si vous avez plusieurs abonnements, sélectionnez l’abonnement approprié dans lequel la ressource doit être facturée. Pour accéder à l’ensemble de votre abonnement, utilisez [az account list](/cli/azure/account#az-account-list).

```azurecli
az account set --subscription <subscription id>
```

Si vous n’avez pas encore créé de serveur, reportez-vous à ce [démarrage rapide](quickstart-create-server-database-azure-cli.md) pour en créer un.

[!INCLUDE [cloud-shell-try-it](../../includes/cloud-shell-try-it.md)]

## <a name="scale-compute-and-storage"></a>Mettre à l’échelle le calcul et le stockage

Vous pouvez facilement faire évoluer votre niveau tarifaire, votre calcul et votre stockage à l’aide de la commande suivante. Vous pouvez voir toutes les opérations de serveur que vous pouvez effectuer [az postgres server overview](/cli/azure/mysql/server)

```azurecli-interactive
az postgres server update --resource-group myresourcegroup --name mydemoserver --sku-name GP_Gen5_4 --storage-size 6144
```

Voici les détails des arguments ci-dessus :

**Paramètre** | **Exemple de valeur** | **Description**
---|---|---
name | mydemoserver | Entrez un nom unique pour votre serveur Azure Database pour PostgreSQL. Le nom de serveur ne peut contenir que des lettres minuscules, des chiffres et le caractère de trait d’union (-). Il doit inclure entre 3 et 63 caractères.
resource-group | myResourceGroup | Indiquez le nom du groupe de ressources Azure.
sku-name|GP_Gen5_2|Entrez le nom du niveau tarifaire et de la configuration de calcul. Suit la convention {niveau tarifaire} _{génération de calcul}_ {vCores} dans le raccourci. Pour plus d’informations, consultez les [niveaux tarifaires](./concepts-pricing-tiers.md).
storage-size | 6144 | Capacité de stockage du serveur (en mégaoctets). 5120 minimum et augmente par incrément de 1024.

> [!Important]
> - Vous pouvez effectuer un scale-up du stockage (mais pas un scale-down)
> - Le passage du niveau tarifaire De base au niveau à Usage général ou à Mémoire optimisée n’est pas pris en charge. Vous pouvez procéder à une modification manuelle soit [en utilisant un script Bash](https://techcommunity.microsoft.com/t5/azure-database-for-mysql/upgrade-from-basic-to-general-purpose-or-memory-optimized-tiers/ba-p/830404) soit [en utilisant PostgreSQL Workbench](https://techcommunity.microsoft.com/t5/azure-database-support-blog/how-to-scale-up-azure-database-for-mysql-from-basic-tier-to/ba-p/369134)


## <a name="manage-postgresql-databases-on-a-server"></a>Gérer des bases de données PostgreSQL sur un serveur.
Vous pouvez utiliser l’une de ces commandes pour créer, supprimer, répertorier et afficher les propriétés d’une base de données sur votre serveur

| Applet de commande | Usage| Description |
| --- | ---| --- |
|[az postgres db create](/cli/azure/sql/db#az-mysql-db-create)|```az postgres db create -g myresourcegroup -s mydemoserver -n mydatabasename``` |Crée une base de données|
|[az postgres db delete](/cli/azure/sql/db#az-mysql-db-delete)|```az postgres db delete -g myresourcegroup -s mydemoserver -n mydatabasename```|Supprime votre base de données de votre serveur. Cette commande ne supprime pas votre serveur. |
|[az postgres db list](/cli/azure/sql/db#az-mysql-db-list)|```az postgres db list -g myresourcegroup -s mydemoserver```|Répertorie toutes les bases de données sur le serveur|
|[az postgres db show](/cli/azure/sql/db#az-mysql-db-show)|```az postgres db show -g myresourcegroup -s mydemoserver -n mydatabasename```|Affiche plus de détails sur la base de données|

## <a name="update-admin-password"></a>Mettre à jour le mot de passe administrateur
Vous pouvez modifier le mot de passe du rôle d’administrateur avec cette commande
```azurecli-interactive
az postgres server update --resource-group myresourcegroup --name mydemoserver --admin-password <new-password>
```

> [!Important]
>  Assurez-vous que le mot de passe compte huit caractères minimum et 128 caractères maximum.
> Le mot de passe doit contenir des caractères de trois des catégories suivantes : Lettres majuscules, lettres minuscules, chiffres et caractères non alphanumériques.

## <a name="delete-a-server"></a>Supprimer un serveur
Si vous souhaitez simplement supprimer le serveur unique PostgreSQL, vous pouvez exécuter la commande [az postgres server delete](/cli/azure/mysql/server#az-mysql-server-delete).

```azurecli-interactive
az postgres server delete --resource-group myresourcegroup --name mydemoserver
```

## <a name="next-steps"></a>Étapes suivantes
- [Redémarrer un serveur](howto-restart-server-cli.md)
- [Restaurer un serveur dans un état incorrect](howto-restore-server-cli.md)
- [Superviser et optimiser le serveur](concepts-monitoring.md)