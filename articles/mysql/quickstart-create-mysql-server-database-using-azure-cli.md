---
title: 'Démarrage rapide : Créer un serveur – Azure CLI – Azure Database pour MySQL'
description: Ce guide de démarrage rapide explique comment utiliser l’interface CLI Azure pour créer un serveur Azure Database pour MySQL dans un groupe de ressources Azure.
author: savjani
ms.author: pariks
ms.service: mysql
ms.devlang: azurecli
ms.topic: quickstart
ms.date: 07/15/2020
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 4f774351fc0eaaf32069687a1943c72b74b6308a
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96494322"
---
# <a name="quickstart-create-an-azure-database-for-mysql-server-using-azure-cli"></a>Démarrage rapide : Création d’un serveur Azure Database pour MySQL à l’aide de la CLI Azure

> [!TIP]
> Envisagez d’utiliser la commande Azure CLI plus simple [az mysql up](/cli/azure/ext/db-up/mysql#ext-db-up-az-mysql-up) (actuellement en préversion). Essayez le guide de [démarrage rapide](./quickstart-create-server-up-azure-cli.md).

Ce guide de démarrage rapide montre comment utiliser les commandes [Azure CLI](/cli/azure/get-started-with-azure-cli) dans [Azure Cloud Shell](https://shell.azure.com) afin de créer un serveur Azure Database pour MySQL en cinq minutes. 

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

 - Ce guide de démarrage rapide nécessite la version 2.0 ou ultérieure de l’interface Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

 - Sélectionnez l’abonnement spécifique sous votre compte à l’aide de la commande [az account set](/cli/azure/account). Notez la valeur **id** issue de la commande **az login** ; vous devez vous en servir comme valeur de l’argument **subscription** dans la commande. Si vous avez plusieurs abonnements, sélectionnez l’abonnement approprié dans lequel la ressource doit être facturée. Pour accéder à l’ensemble de votre abonnement, utilisez [az account list](/cli/azure/account#az-account-list).

   ```azurecli
   az account set --subscription <subscription id>
   ```

## <a name="create-an-azure-database-for-mysql-server"></a>Création d’un serveur Azure Database pour MySQL
Créez un [groupe de ressources Azure](../azure-resource-manager/management/overview.md) à l’aide de la commande [az group create](/cli/azure/group), puis créez votre serveur MySQL à l’intérieur de ce groupe de ressources. Vous devez fournir un nom unique. L’exemple suivant crée un groupe de ressources nommé `myresourcegroup` à l’emplacement `westus`.

```azurecli-interactive
az group create --name myresourcegroup --location westus
```

Créez un serveur Azure Database pour MySQL avec la commande [az sql server create](/cli/azure/mysql/server#az-mysql-server-create). Un serveur peut contenir plusieurs bases de données.

```azurecli
az mysql server create --resource-group myresourcegroup --name mydemoserver --location westus --admin-user myadmin --admin-password <server_admin_password> --sku-name GP_Gen5_2 
```

Voici les détails des arguments ci-dessus : 

**Paramètre** | **Exemple de valeur** | **Description**
---|---|---
name | mydemoserver | Entrez un nom unique pour votre serveur Azure Database pour MySQL. Le nom de serveur ne peut contenir que des lettres minuscules, des chiffres et le caractère de trait d’union (-). Il doit inclure entre 3 et 63 caractères.
resource-group | myResourceGroup | Indiquez le nom du groupe de ressources Azure.
location | westus | Emplacement Azure du serveur.
admin-user | myadmin | Nom d’utilisateur du compte administrateur. Il ne peut pas être **azure_superuser** (superutilisateur), **admin**, **administrator** (administrateur), **root** (racine), **guest** (invité) ou **public**.
admin-password | *mot de passe sécurisé* | Mot de passe de l’utilisateur Administrateur. Il doit contenir entre 8 et 128 caractères. Votre mot de passe doit contenir des caractères de trois des catégories suivantes : Lettres majuscules, lettres minuscules, chiffres et caractères non alphanumériques.
sku-name|GP_Gen5_2|Entrez le nom du niveau tarifaire et de la configuration de calcul. Suit la convention {niveau tarifaire} _{génération de calcul}_ {vCores} dans le raccourci. Pour plus d’informations, consultez les [niveaux tarifaires](./concepts-pricing-tiers.md).

>[!IMPORTANT] 
>- La version MySQL par défaut sur votre serveur est 5.7. Nous disposons également des versions 5.6 et 8.0.
>- Pour voir tous les arguments de la commande **az mysql server create**, consultez ce [document de référence](/cli/azure/mysql/server#az-mysql-server-create).
>- Le protocole SSL est activé par défaut sur votre serveur. Pour plus d’information sur le protocole SSL, consultez [Configurer la connectivité SSL](howto-configure-ssl.md).

## <a name="configure-a-server-level-firewall-rule"></a>Configurer une règle de pare-feu au niveau du serveur 
Par défaut, le serveur créé est protégé par des règles de pare-feu et n’est pas accessible publiquement. Vous pouvez configurer la règle de pare-feu sur votre serveur à l’aide de la commande [az mysql server firewall-rule create](/cli/azure/mysql/server/firewall-rule). Cela vous permettra de vous connecter au serveur localement.

L’exemple suivant crée une règle de pare-feu appelée `AllowMyIP` qui autorise les connexions d’une adresse IP spécifique, 192.168.0.1. Remplacez l’adresse IP à partir de laquelle vous allez vous connecter. Vous pouvez utiliser une plage d’adresses IP si nécessaire. En cas de doute, accédez à [https://whatismyipaddress.com/](https://whatismyipaddress.com/) pour obtenir votre adresse IP.

```azurecli-interactive
az mysql server firewall-rule create --resource-group myresourcegroup --server mydemoserver --name AllowMyIP --start-ip-address 192.168.0.1 --end-ip-address 192.168.0.1
```

> [!NOTE]
> Les connexions à la base de données Azure pour MySQL communiquent via le port 3306. Si vous essayez de vous connecter à partir d’un réseau d’entreprise, le trafic sortant sur le port 3306 peut être bloqué. Si c’est le cas, vous ne pouvez pas vous connecter à votre serveur, sauf si votre service informatique ouvre le port 3306.

## <a name="get-the-connection-information"></a>Obtenir les informations de connexion

Pour vous connecter à votre serveur, vous devez fournir des informations sur l’hôte et des informations d’identification pour l’accès.

```azurecli-interactive
az mysql server show --resource-group myresourcegroup --name mydemoserver
```

Le résultat est au format JSON. Notez les valeurs **fullyQualifiedDomainName** et **administratorLogin**.
```json
{
  "administratorLogin": "myadmin",
  "earliestRestoreDate": null,
  "fullyQualifiedDomainName": "mydemoserver.mysql.database.azure.com",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.DBforMySQL/servers/mydemoserver",
  "location": "westus",
  "name": "mydemoserver",
  "resourceGroup": "myresourcegroup",
  "sku": {
    "capacity": 2,
    "family": "Gen5",
    "name": "GP_Gen5_2",
    "size": null,
    "tier": "GeneralPurpose"
  },
  "sslEnforcement": "Enabled",
  "storageProfile": {
    "backupRetentionDays": 7,
    "geoRedundantBackup": "Disabled",
    "storageMb": 5120
  },
  "tags": null,
  "type": "Microsoft.DBforMySQL/servers",
  "userVisibleState": "Ready",
  "version": "5.7"
}
```

## <a name="connect-to-azure-database-for-mysql-server-using-mysql-command-line-client"></a>Se connecter au serveur Azure Database pour MySQL à l’aide du client de ligne de commande mysql
Vous pouvez vous connecter à votre serveur à l’aide d’un outil en ligne de commande populaire, **[mysql.exe](https://dev.mysql.com/downloads/)** , avec [Azure Cloud Shell](../cloud-shell/overview.md). Vous pouvez également utiliser la ligne de commande mysql dans votre environnement local.
```bash
 mysql -h mydemoserver.mysql.database.azure.com -u myadmin@mydemoserver -p
```

## <a name="clean-up-resources"></a>Nettoyer les ressources
Si vous n’avez pas besoin de ces ressources pour un autre guide de démarrage rapide ou didacticiel, vous pouvez les supprimer en exécutant la commande suivante : 

```azurecli-interactive
az group delete --name myresourcegroup
```

Si vous souhaitez simplement supprimer le serveur nouvellement créé, vous pouvez exécuter la commande [az mysql server delete](/cli/azure/mysql/server#az-mysql-server-delete).

```azurecli-interactive
az mysql server delete --resource-group myresourcegroup --name mydemoserver
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
>[Créer une application PHP sur Windows avec MySQL](../app-service/tutorial-php-mysql-app.md)