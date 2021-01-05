---
title: Gérer un serveur - Azure CLI - Azure Database pour MySQL
description: Découvrez comment gérer un serveur Azure Database pour MySQL à partir d’Azure CLI.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: how-to
ms.date: 9/22/2020
ms.openlocfilehash: bd068f0cf76a8edefca854d72d5240c0becaf8fc
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94542063"
---
# <a name="manage-an-azure-database-for-mysql-single-server-using-the-azure-cli"></a>Gérer un serveur unique Azure Database pour MySQL avec Azure CLI

Cet article vous explique comment gérer vos serveurs uniques déployés sur Azure. Les tâches de gestion incluent notamment la mise à l’échelle du calcul et du stockage, la réinitialisation de mot de passe et l’affichage des informations relatives au serveur.

## <a name="prerequisites"></a>Prérequis
Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer. Cet article nécessite que vous exécutiez localement Azure CLI version 2.0 ou ultérieure. Pour afficher la version installée, exécutez la commande `az --version`. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI](/cli/azure/install-azure-cli).

Vous devrez vous connecter à votre compte à l’aide de la commande [az login](/cli/azure/reference-index#az-login). Notez la propriété **id**, qui fait référence à l’**ID d’abonnement** pour votre compte Azure.

```azurecli-interactive
az login
```

Sélectionnez l’abonnement spécifique sous votre compte à l’aide de la commande [az account set](/cli/azure/account). Notez la valeur **id** issue de la commande **az login** ; vous devez vous en servir comme valeur de l’argument **subscription** dans la commande. Si vous avez plusieurs abonnements, sélectionnez l’abonnement approprié dans lequel la ressource doit être facturée. Pour accéder à l’ensemble de votre abonnement, utilisez [az account list](/cli/azure/account#az-account-list).

```azurecli
az account set --subscription <subscription id>
```

Si vous n’avez pas encore créé de serveur, reportez-vous à ce [démarrage rapide](quickstart-create-mysql-server-database-using-azure-cli.md) pour en créer un.

## <a name="scale-compute-and-storage"></a>Mettre à l’échelle le calcul et le stockage
Vous pouvez facilement faire évoluer votre niveau tarifaire, votre calcul et votre stockage à l’aide de la commande suivante. Vous pouvez voir toutes les opérations de serveur que vous pouvez effectuer [az mysql server overview](/cli/azure/mysql/server)

```azurecli-interactive
az mysql server update --resource-group myresourcegroup --name mydemoserver --sku-name GP_Gen5_4 --storage-size 6144
```

Voici les détails des arguments ci-dessus :

**Paramètre** | **Exemple de valeur** | **Description**
---|---|---
name | mydemoserver | Entrez un nom unique pour votre serveur Azure Database pour MySQL. Le nom de serveur ne peut contenir que des lettres minuscules, des chiffres et le caractère de trait d’union (-). Il doit inclure entre 3 et 63 caractères.
resource-group | myResourceGroup | Indiquez le nom du groupe de ressources Azure.
sku-name|GP_Gen5_2|Entrez le nom du niveau tarifaire et de la configuration de calcul. Suit la convention {niveau tarifaire} _{génération de calcul}_ {vCores} dans le raccourci. Pour plus d’informations, consultez les [niveaux tarifaires](./concepts-pricing-tiers.md).
storage-size | 6144 | Capacité de stockage du serveur (en mégaoctets). 5120 minimum et augmente par incrément de 1024.

> [!Important]
> - Vous pouvez effectuer un scale-up du stockage (mais pas un scale-down)
> - Le passage du niveau tarifaire de Usage de base à Usage général ou à Mémoire optimisée n’est pas prise en charge. Vous pouvez procéder à une modification manuelle soit [en utilisant un script Bash](https://techcommunity.microsoft.com/t5/azure-database-for-mysql/upgrade-from-basic-to-general-purpose-or-memory-optimized-tiers/ba-p/830404) soit [en utilisant MySQL Workbench](https://techcommunity.microsoft.com/t5/azure-database-support-blog/how-to-scale-up-azure-database-for-mysql-from-basic-tier-to/ba-p/369134)


## <a name="manage-mysql-databases-on-a-server"></a>Gérer des bases de données MySQL sur un serveur
Vous pouvez utiliser l’une de ces commandes pour créer, supprimer, répertorier et afficher les propriétés d’une base de données sur votre serveur

| Applet de commande | Usage| Description |
| --- | ---| --- |
|[az mysql db create](/cli/azure/sql/db#az-mysql-db-create)|```az mysql db create -g myresourcegroup -s mydemoserver -n mydatabasename``` |Crée une base de données|
|[az mysql db delete](/cli/azure/sql/db#az-mysql-db-delete)|```az mysql db delete -g myresourcegroup -s mydemoserver -n mydatabasename```|Supprime votre base de données de votre serveur. Cette commande ne supprime pas votre serveur. |
|[az mysql db list](/cli/azure/sql/db#az-mysql-db-list)|```az mysql db list -g myresourcegroup -s mydemoserver```|Répertorie toutes les bases de données sur le serveur|
|[az mysql db show](/cli/azure/sql/db#az-mysql-db-show)|```az mysql db show -g myresourcegroup -s mydemoserver -n mydatabasename```|Affiche plus de détails sur la base de données|

## <a name="update-admin-password"></a>Mettre à jour le mot de passe administrateur
Vous pouvez modifier le mot de passe du rôle d’administrateur avec cette commande
```azurecli-interactive
az mysql server update --resource-group myresourcegroup --name mydemoserver --admin-password <new-password>
```

> [!Important]
>  Assurez-vous que le mot de passe dispose de 8 caractères minimum et 128 caractères maximum.
> Le mot de passe doit contenir des caractères de trois des catégories suivantes : Lettres majuscules, lettres minuscules, chiffres et caractères non alphanumériques.

## <a name="delete-a-server"></a>Supprimer un serveur
Si vous souhaitez simplement supprimer le serveur unique MySQL, vous pouvez exécuter la commande [az mysql server delete](/cli/azure/mysql/server#az-mysql-server-delete).

```azurecli-interactive
az mysql server delete --resource-group myresourcegroup --name mydemoserver
```

## <a name="next-steps"></a>Étapes suivantes
- [Redémarrer un serveur](howto-restart-server-cli.md)
- [Restaurer un serveur dans un état incorrect](howto-restore-server-cli.md)
- [Superviser et optimiser le serveur](concepts-monitoring.md)