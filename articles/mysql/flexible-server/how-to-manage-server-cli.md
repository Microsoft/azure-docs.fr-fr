---
title: Gérer un serveur - Azure CLI - Azure Database pour MySQL - Serveur flexible
description: Découvrez comment gérer un serveur flexible Azure Database pour MySQL à partir de l’interface de ligne de commande Azure.
author: mksuni
ms.author: sumuth
ms.service: mysql
ms.topic: how-to
ms.date: 9/21/2020
ms.openlocfilehash: b58a9dd7901f85c59b09bc4ccb197d012bce2200
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92545053"
---
# <a name="manage-an-azure-database-for-mysql---flexible-server-preview-using-the-azure-cli"></a>Gérer un serveur flexible Azure Database pour MySQL (préversion) à l’aide de l’interface de ligne de commande Azure

> [!IMPORTANT]
> Azure Database pour MySQL – Serveur flexible est actuellement en préversion publique.

Cet article vous explique comment gérer votre serveur flexible (préversion) déployé sur Azure. Les tâches de gestion incluent notamment la mise à l’échelle du calcul et du stockage, la réinitialisation de mot de passe et l’affichage des informations relatives au serveur.

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

> [!Important]
> Si vous n’avez pas encore créé de serveur flexible, créez-en un pour vous familiariser avec ce guide.

## <a name="scale-compute-and-storage"></a>Mettre à l’échelle le calcul et le stockage

Vous pouvez facilement effectuer un scale-up de votre niveau de calcul, de vos vCores et de votre stockage à l’aide de la commande suivante. Vous pouvez voir toutes les opérations de serveur que vous pouvez effectuer : [az mysql flexible-server update](/cli/azure/mysql/flexible-server#az_mysql_flexible_server_update).

```azurecli-interactive
az mysql flexible-server update --resource-group myresourcegroup --name mydemoserver --sku-name Standard_D4ds_v4 --storage-size 6144
```

Voici les détails des arguments ci-dessus :

**Paramètre** | **Exemple de valeur** | **Description**
---|---|---
name | mydemoserver | Entrez un nom unique pour votre serveur Azure Database pour MySQL. Le nom de serveur ne peut contenir que des lettres minuscules, des chiffres et le caractère de trait d’union (-). Il doit inclure entre 3 et 63 caractères.
resource-group | myResourceGroup | Indiquez le nom du groupe de ressources Azure.
sku-name|Standard_D4ds_v4|Entrez le nom du niveau de calcul et de la taille. Respecte la convention Standard_ {taille de machine virtuelle} en abrégé. Pour plus d’informations, consultez les [niveaux tarifaires](../concepts-pricing-tiers.md).
storage-size | 6144 | Capacité de stockage du serveur (en mégaoctets). 5120 minimum et augmente par incrément de 1024.

> [!Important]
> - Le stockage peut être monté en puissance (mais pas descendu en puissance)


## <a name="manage-mysql-databases-on-a-server"></a>Gérez des bases de données MySQL sur un serveur.
Vous pouvez utiliser l’une de ces commandes pour créer, supprimer, répertorier et afficher les propriétés d’une base de données sur votre serveur.

| Applet de commande | Usage| Description |
| --- | ---| --- |
|[az mysql flexible-server db create](/cli/azure/mysql/flexible-server/db#az_mysql_flexible_server_db_create)|```az mysql flexible-server db create -g myresourcegroup -s mydemoserver -n mydatabasename``` |Crée une base de données|
|[az mysql flexible-server db delete](/cli/azure/mysql/flexible-server/db#az_mysql_flexible_server_db_delete)|```az mysql flexible-server db delete -g myresourcegroup -s mydemoserver -n mydatabasename```|Supprime votre base de données de votre serveur. Cette commande ne supprime pas votre serveur. |
|[az mysql flexible-server db list](/cli/azure/mysql/flexible-server/db#az_mysql_flexible_server_db_list)|```az mysql flexible-server db list -g myresourcegroup -s mydemoserver```|Répertorie toutes les bases de données sur le serveur|
|[az mysql flexible-server db show](/cli/azure/mysql/flexible-server/db#az_mysql_flexible_server_db_show)|```az mysql flexible-server db show -g myresourcegroup -s mydemoserver -n mydatabasename```|Affiche plus de détails sur la base de données|

## <a name="update-admin-password"></a>Mettre à jour le mot de passe administrateur
Vous pouvez modifier le mot de passe du rôle d’administrateur avec cette commande
```azurecli-interactive
az mysql flexible-server update --resource-group myresourcegroup --name mydemoserver --admin-password <new-password>
```

> [!Important]
>  Assurez-vous que le mot de passe compte huit caractères minimum et 128 caractères maximum.
> Le mot de passe doit contenir des caractères de trois des catégories suivantes : Lettres majuscules, lettres minuscules, chiffres et caractères non alphanumériques.

## <a name="delete-a-server"></a>Supprimer un serveur
Si vous souhaitez simplement supprimer le serveur flexible MySQL, vous pouvez exécuter la commande [az mysql flexible-server server delete](/cli/azure/mysql/flexible-server#az_mysql_flexible_server_delete).

```azurecli-interactive
az mysql flexible-server delete --resource-group myresourcegroup --name mydemoserver
```

## <a name="next-steps"></a>Étapes suivantes
- [Découvrir comment démarrer ou arrêter un serveur](how-to-stop-start-server-portal.md)
- [Découvrir comment gérer un réseau virtuel](how-to-manage-virtual-network-cli.md)
- [Résoudre les problèmes de connexion](how-to-troubleshoot-common-connection-issues.md)
- [Créer et gérer un pare-feu](how-to-manage-firewall-cli.md)