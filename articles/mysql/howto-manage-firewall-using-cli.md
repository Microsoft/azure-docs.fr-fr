---
title: Gérer les règles de pare-feu - Azure CLI - Azure Database pour MySQL
description: Cet article décrit comment créer et gérer des règles de pare-feu Azure Database pour MySQL à l’aide de la ligne de commande Azure CLI.
author: savjani
ms.author: pariks
ms.service: mysql
ms.devlang: azurecli
ms.topic: how-to
ms.date: 3/18/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: eaddd8b2979b30251301ad041ea4b872c23d680b
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94541332"
---
# <a name="create-and-manage-azure-database-for-mysql-firewall-rules-by-using-the-azure-cli"></a>Créer et gérer des règles de pare-feu Azure Database pour MySQL à l’aide de l’interface de ligne de commande Azure
Les règles de pare-feu au niveau du serveur peuvent être utilisées pour gérer l'accès à un serveur Azure Database pour MySQL à partir d'une adresse IP ou d'une plage d'adresses IP spécifique. À l’aide de commandes d’interface de ligne de commande Azure pratiques, vous pouvez créer, mettre à jour, supprimer, répertorier et afficher les règles de pare-feu pour gérer votre serveur. Pour une vue d'ensemble des pare-feu Azure Database pour MySQL, consultez [Règles de pare-feu d'Azure Database pour MySQL](./concepts-firewall-rules.md).

Des règles de réseau virtuel (VNet) peuvent également être utilisées pour sécuriser l'accès à votre serveur. Découvrez-en plus sur [la création et la gestion des règles et des points de terminaison de service de réseau virtuel à l'aide d'Azure CLI](howto-manage-vnet-using-cli.md).

## <a name="prerequisites"></a>Prérequis
* [Installez l’interface de ligne de commande Azure](/cli/azure/install-azure-cli).
* Un [serveur et une base de données Azure Database pour MySQL](quickstart-create-mysql-server-database-using-azure-cli.md).

## <a name="firewall-rule-commands"></a>Commandes de règle de pare-feu :
La commande **az mysql server firewall-rule** est utilisée à partir de l’interface de ligne de commande Azure pour créer, supprimer, répertorier, afficher et mettre à jour des règles de pare-feu.

Commandes :
- **create** : créez une règle de pare-feu du serveur Azure MySQL.
- **delete** : supprimez une règle de pare-feu du serveur Azure MySQL.
- **list** : répertoriez les règles de pare-feu du serveur Azure MySQL.
- **show** : affichez les détails d’une règle de pare-feu du serveur Azure MySQL.
- **update** : mettez à jour une règle de pare-feu du serveur Azure MySQL.

## <a name="sign-in-to-azure-and-list-your-azure-database-for-mysql-servers"></a>Se connecter à Azure et répertorier les serveurs Azure Database pour MySQL
Connectez-vous en toute sécurité à Azure CLI à l’aide de votre compte Azure en exécutant la commande **az login**.

1. Exécutez la commande suivante à partir de la ligne de commande :
    ```azurecli
    az login
    ```
   Cette commande génère un code à utiliser lors de l’étape suivante.

2. Utilisez un navigateur web pour ouvrir la page [https://aka.ms/devicelogin](https://aka.ms/devicelogin), puis entrez le code.

3. À l'invite, connectez-vous à l'aide de vos informations d'identification Azure.

4. Une fois votre connexion autorisée, une liste des abonnements s’affiche dans la console. Copiez l’ID de l’abonnement souhaité pour définir l’abonnement actif à utiliser. Utilisez la commande [az account set](/cli/azure/account#az-account-set).
    ```azurecli-interactive
    az account set --subscription <your subscription id>
    ```

5. Si vous ne connaissez pas le nom de votre abonnement et du groupe de ressources, affichez la liste des serveurs Azure Database for MySQL. Utilisez la commande [az mysql server list](/cli/azure/mysql/server#az-mysql-server-list).

    ```azurecli-interactive
    az mysql server list --resource-group myresourcegroup
    ```

   Notez l’attribut de nom dans la liste ; il servira à spécifier le serveur MySQL sur lequel vous allez travailler. Si nécessaire, vérifiez les informations permettant d’utiliser l’attribut de nom et ce serveur pour vérifier que le nom est correct. Utilisez la commande [az mysql server show](/cli/azure/mysql/server#az-mysql-server-show).

    ```azurecli-interactive
    az mysql server show --resource-group myresourcegroup --name mydemoserver
    ```

## <a name="list-firewall-rules-on-azure-database-for-mysql-server"></a>Répertorier les règles de pare-feu d’un serveur Azure Database pour MySQL 
À l’aide du nom du serveur et du nom de groupe de ressources, répertoriez les règles de pare-feu existantes sur le serveur. Utilisez la commande [az mysql server firewall list](/cli/azure/mysql/server/firewall-rule#az-mysql-server-firewall-rule-list).  Notez que l’attribut de nom de serveur est spécifié dans le commutateur **-server** et non dans le commutateur **--name**. 
```azurecli-interactive
az mysql server firewall-rule list --resource-group myresourcegroup --server-name mydemoserver
```
La sortie répertorie les règles éventuelles au format JSON (par défaut). Vous pouvez utiliser le commutateur **--output table** pour générer les résultats dans un format tabulaire plus lisible.
```azurecli-interactive
az mysql server firewall-rule list --resource-group myresourcegroup --server-name mydemoserver --output table
```
## <a name="create-a-firewall-rule-on-azure-database-for-mysql-server"></a>Créer une règle de pare-feu sur un serveur Azure Database pour MySQL
À l’aide du nom du serveur Azure pour MyQL et du nom du groupe de ressources, créez une nouvelle règle de pare-feu sur le serveur. Utilisez la commande [az mysql server firewall create](/cli/azure/mysql/server/firewall-rule#az-mysql-server-firewall-rule-create). Indiquez le nom de la règle, ainsi que les adresses IP de début et de fin de la règle (pour fournir un accès à une plage d’adresses IP).
```azurecli-interactive
az mysql server firewall-rule create --resource-group myresourcegroup --server-name mydemoserver --name FirewallRule1 --start-ip-address 13.83.152.0 --end-ip-address 13.83.152.15
```

Pour autoriser une seule adresse IP à accéder au serveur, fournissez des adresses IP de début et de fin identiques, comme dans cet exemple.
```azurecli-interactive
az mysql server firewall-rule create --resource-group myresourcegroup --server-name mydemoserver --name FirewallRule1 --start-ip-address 1.1.1.1 --end-ip-address 1.1.1.1
```

Pour autoriser les applications à partir d’adresses IP Azure à se connecter à votre Azure Database pour MySQL, fournissez l’adresse IP 0.0.0.0 comme adresse IP de début et adresse IP de fin, comme dans cet exemple.
```azurecli-interactive
az mysql server firewall-rule create --resource-group myresourcegroup --server mysql --name "AllowAllWindowsAzureIps" --start-ip-address 0.0.0.0 --end-ip-address 0.0.0.0
```

> [!IMPORTANT]
> Cette option configure le pare-feu pour autoriser toutes les connexions à partir d’Azure, notamment les connexions issues des abonnements d’autres clients. Lorsque vous sélectionnez cette option, vérifiez que votre connexion et vos autorisations utilisateur limitent l’accès aux seuls utilisateurs autorisés.
> 

En cas de réussite, la sortie de chaque commande de création affiche les détails de la règle de pare-feu que vous avez créée au format JSON (par défaut). En cas d’échec, la sortie affiche un texte de message d’erreur.

## <a name="update-a-firewall-rule-on-azure-database-for-mysql-server"></a>Mettre à jour une règle de pare-feu sur un serveur Azure Database pour MySQL 
À l’aide du nom du serveur Azure pour MySQL et du nom du groupe de ressources, mettez à jour une règle de pare-feu existante sur le serveur. Utilisez la commande [az mysql server firewall update](/cli/azure/mysql/server/firewall-rule#az-mysql-server-firewall-rule-update). Indiquez le nom de la règle de pare-feu existante comme entrée, ainsi que les adresses IP de début et de fin à mettre à jour.
```azurecli-interactive
az mysql server firewall-rule update --resource-group myresourcegroup --server-name mydemoserver --name FirewallRule1 --start-ip-address 13.83.152.0 --end-ip-address 13.83.152.1
```
En cas de réussite, la sortie de la commande affiche les détails de la règle de pare-feu que vous avez mise à jour au format JSON (par défaut). En cas d’échec, la sortie affiche un texte de message d’erreur.

> [!NOTE]
> Si la règle de pare-feu n’existe pas, elle est créée par la commande de mise à jour.

## <a name="show-firewall-rule-details-on-azure-database-for-mysql-server"></a>Afficher les détails d’une règle de pare-feu sur un serveur Azure Database pour MySQL
À l’aide du nom du serveur Azure pour MySQL et du nom du groupe de ressources, affichez les détails d’une règle de pare-feu existante sur le serveur. Utilisez la commande [az mysql server firewall show](/cli/azure/mysql/server/firewall-rule#az-mysql-server-firewall-rule-show). Indiquez le nom de la règle de pare-feu existante comme entrée.
```azurecli-interactive
az mysql server firewall-rule show --resource-group myresourcegroup --server-name mydemoserver --name FirewallRule1
```
En cas de réussite, la sortie de la commande affiche les détails de la règle de pare-feu que vous avez spécifiée au format JSON (par défaut). En cas d’échec, la sortie affiche un texte de message d’erreur.

## <a name="delete-a-firewall-rule-on-azure-database-for-mysql-server"></a>Supprimer une règle de pare-feu sur un serveur Azure Database pour MySQL
À l’aide du nom du serveur Azure pour MySQL et du nom du groupe de ressources, supprimez une règle de pare-feu existante du serveur. Utilisez la commande [az mysql server firewall delete](/cli/azure/mysql/server/firewall-rule#az-mysql-server-firewall-rule-delete). Indiquez le nom de la règle de pare-feu existante.
```azurecli-interactive
az mysql server firewall-rule delete --resource-group myresourcegroup --server-name mydemoserver --name FirewallRule1
```
En cas de réussite, aucun résultat ne s’affiche. En cas d’échec, le texte du message d’erreur s’affiche.

## <a name="next-steps"></a>Étapes suivantes
- En savoir plus sur les [règles de pare-feu du serveur Azure Database pour MySQL](./concepts-firewall-rules.md).
- [Créer et gérer des règles de pare-feu Azure Database pour MySQL à l’aide du portail Azure](./howto-manage-firewall-using-portal.md).
- Pour sécuriser davantage l’accès à votre serveur, [créez et gérez des règles et des points de terminaison de service de réseau virtuel à l’aide d’Azure CLI](howto-manage-vnet-using-cli.md).