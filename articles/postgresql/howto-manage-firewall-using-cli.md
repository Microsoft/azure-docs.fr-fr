---
title: Gérer des règles de pare-feu - Azure CLI - Azure Database pour PostgreSQL - Serveur unique
description: Cet article décrit comment créer et gérer des règles de pare-feu dans Azure Database pour PostgreSQL – Serveur unique en utilisant l’interface de ligne de commande Azure.
author: niklarin
ms.author: nlarin
ms.service: postgresql
ms.devlang: azurecli
ms.topic: how-to
ms.date: 5/6/2019
ms.custom: devx-track-azurecli
ms.openlocfilehash: 3a559b8c65ab57b0144b807a3b4cc1faa912d430
ms.sourcegitcommit: 7cc10b9c3c12c97a2903d01293e42e442f8ac751
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/06/2020
ms.locfileid: "93422737"
---
# <a name="create-and-manage-firewall-rules-in-azure-database-for-postgresql---single-server-using-azure-cli"></a>Créer et gérer des règles de pare-feu dans Azure Database pour PostgreSQL – Serveur unique en utilisant Azure CLI
Les règles de pare-feu au niveau du serveur peuvent être utilisées pour gérer l’accès à un serveur Azure Database pour PostgreSQL à partir d’une adresse IP ou d’une plage d’adresses IP spécifiques. À l’aide de commandes d’interface de ligne de commande Azure pratiques, vous pouvez créer, mettre à jour, supprimer, répertorier et afficher les règles de pare-feu pour gérer votre serveur. Pour une vue d’ensemble des règles de pare-feu Azure Database pour PostgreSQL, voir [Règles de pare-feu d’un serveur Azure Database pour PostgreSQL](concepts-firewall-rules.md).

Des règles de réseau virtuel (VNet) peuvent également être utilisées pour sécuriser l'accès à votre serveur. Découvrez-en plus sur [la création et la gestion des règles et des points de terminaison de service de réseau virtuel à l'aide d'Azure CLI](howto-manage-vnet-using-cli.md).

## <a name="prerequisites"></a>Prérequis
Pour parcourir ce guide pratique, vous avez besoin des éléments suivants :
- Installer l’utilitaire de ligne de commande [Azure CLI](/cli/azure/install-azure-cli) ou utiliser Azure Cloud Shell dans le navigateur.
- Un [serveur Azure Database pour PostgreSQL et une base de données](quickstart-create-server-database-azure-cli.md).

## <a name="configure-firewall-rules-for-azure-database-for-postgresql"></a>Configuration des règles de pare-feu pour Azure Database pour PostgreSQL
Les commandes [az postgres server firewall-rule](/cli/azure/postgres/server/firewall-rule) permettent de configurer des règles de pare-feu.

## <a name="list-firewall-rules"></a>Répertorier les règles de pare-feu 
Pour répertorier les règles de pare-feu de serveur existant, exécutez la commande [az postgres server firewall-rule list](/cli/azure/postgres/server/firewall-rule).
```azurecli-interactive
az postgres server firewall-rule list --resource-group myresourcegroup --server-name mydemoserver
```
La sortie répertorie les règles de pare-feu éventuelles, par défaut au format JSON. Vous pouvez utiliser le commutateur `--output table` pour obtenir une sortie dans un format de tableau plus lisible.
```azurecli-interactive
az postgres server firewall-rule list --resource-group myresourcegroup --server-name mydemoserver --output table
```
## <a name="create-firewall-rule"></a>Créer une règle de pare-feu
Pour créer une règle de pare-feu sur le serveur, exécutez la commande [az postgres server firewall-rule create](/cli/azure/postgres/server/firewall-rule). 


Pour autoriser l’accès à une seule adresse IP, fournissez la même adresse en `--start-ip-address` et en `--end-ip-address`, comme dans cet exemple, en remplaçant l’adresse IP indiquée ici par votre adresse IP spécifique.
```azurecli-interactive
az postgres server firewall-rule create --resource-group myresourcegroup --server-name mydemoserver --name AllowSingleIpAddress --start-ip-address 13.83.152.1 --end-ip-address 13.83.152.1
```
Pour autoriser les applications à se connecter à votre serveur Azure Database pour PostgreSQL à partir d’adresses IP Azure, indiquez l’adresse IP 0.0.0.0 comme adresse IP de début et adresse IP de fin, comme dans cet exemple.
```azurecli-interactive
az postgres server firewall-rule create --resource-group myresourcegroup --server-name mydemoserver --name AllowAllAzureIps --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0
```

> [!IMPORTANT]
> Cette option configure le pare-feu pour autoriser toutes les connexions à partir d’Azure, notamment les connexions issues des abonnements d’autres clients. Lorsque vous sélectionnez cette option, vérifiez que votre connexion et vos autorisations utilisateur limitent l’accès aux seuls utilisateurs autorisés.
> 

En cas de réussite, la sortie de la commande affiche les détails de la règle de pare-feu que vous avez créée, par défaut au format JSON. En cas d’échec, la sortie affiche un message d’erreur à la place.

## <a name="update-firewall-rule"></a>Mettre à jour une règle de pare-feu 
Mettez à jour une règle de pare-feu existante sur le serveur à l’aide de la commande [az postgres server firewall-rule update](/cli/azure/postgres/server/firewall-rule). Indiquez le nom de la règle de pare-feu existante comme entrée puis les adresses IP de début et de fin à mettre à jour.
```azurecli-interactive
az postgres server firewall-rule update --resource-group myresourcegroup --server-name mydemoserver --name AllowIpRange --start-ip-address 13.83.152.0 --end-ip-address 13.83.152.0
```
En cas de réussite, la sortie de la commande affiche les détails de la règle de pare-feu que vous avez mise à jour, par défaut au format JSON. En cas d’échec, la sortie affiche un message d’erreur à la place.
> [!NOTE]
> Si la règle de pare-feu n’existe pas, elle est créée par la commande de mise à jour.

## <a name="show-firewall-rule-details"></a>Afficher les détails d’une règle de pare-feu
Vous pouvez également afficher les détails d’une règle de pare-feu existante au niveau du serveur en exécutant la commande [az postgres server firewall-rule show](/cli/azure/postgres/server/firewall-rule).
```azurecli-interactive
az postgres server firewall-rule show --resource-group myresourcegroup --server-name mydemoserver --name AllowIpRange
```
En cas de réussite, la sortie de la commande affiche les détails de la règle de pare-feu que vous avez spécifiée, par défaut au format JSON. En cas d’échec, la sortie affiche un message d’erreur à la place.

## <a name="delete-firewall-rule"></a>Supprimer une règle de pare-feu
Pour révoquer l’accès pour une plage d’adresses IP vers le serveur, supprimez une règle de pare-feu existante en exécutant la commande [az postgres server firewall-rule delete](/cli/azure/postgres/server/firewall-rule). Indiquez le nom de la règle de pare-feu existante.
```azurecli-interactive
az postgres server firewall-rule delete --resource-group myresourcegroup --server-name mydemoserver --name AllowIpRange
```
En cas de réussite, aucun résultat ne s’affiche. En cas d’échec, le texte du message d’erreur est retourné.

## <a name="next-steps"></a>Étapes suivantes
- De même, vous pouvez utiliser un navigateur web pour [créer et gérer les règles de pare-feu Azure Database pour PostgreSQL feu à l’aide du portail Azure](howto-manage-firewall-using-portal.md).
- En savoir plus sur les [règles de pare-feu du serveur Azure Database pour PostgreSQL](concepts-firewall-rules.md).
- Pour sécuriser davantage l’accès à votre serveur, [créez et gérez des règles et des points de terminaison de service de réseau virtuel à l’aide d’Azure CLI](howto-manage-vnet-using-cli.md).
- Pour vous aider à vous connecter à un serveur Azure Database pour PostgreSQL, consultez la rubrique [Bibliothèques de connexions pour Azure Database pour PostgreSQL](concepts-connection-libraries.md).
