---
title: 'Démarrage rapide : Créer un serveur – Azure CLI – Azure Database pour PostgreSQL – serveur unique'
description: Dans ce guide de démarrage rapide, vous allez créer un serveur Azure Database pour PostgreSQL à l’aide de l’interface Azure CLI.
author: lfittl-msft
ms.author: lufittl
ms.service: postgresql
ms.devlang: azurecli
ms.topic: quickstart
ms.date: 06/25/2020
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: d174e410aaef876dfe97af62750322641de95fd3
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94659452"
---
# <a name="quickstart-create-an-azure-database-for-postgresql-server-by-using-the-azure-cli"></a>Démarrage rapide : Créer un serveur Azure Database pour PostgreSQL à l’aide de l’interface Azure CLI

Ce guide de démarrage rapide montre comment utiliser les commandes [Azure CLI](/cli/azure/get-started-with-azure-cli) dans [Azure Cloud Shell](https://shell.azure.com) afin de créer un serveur unique Azure Database pour PostgreSQL en cinq minutes. Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Cet article nécessite la version 2.0 ou ultérieure d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

    > [!TIP]
    >  Songez à utiliser la commande Azure CLI simplifiée [az postgres up](/cli/azure/ext/db-up/postgres#ext-db-up-az-postgres-up), disponible actuellement en préversion. Essayez le guide de [démarrage rapide](./quickstart-create-server-up-azure-cli.md).

- Sélectionnez l’ID d’abonnement spécifique sous votre compte au moyen de la commande [az account set](/cli/azure/account).

    - Notez la valeur **id** issue de la sortie **az login** ; vous devez vous en servir comme valeur de l’argument **subscription** dans la commande. 

        ```azurecli
        az account set --subscription <subscription id>
        ```

    - Si vous avez plusieurs abonnements, sélectionnez l’abonnement approprié dans lequel la ressource doit être facturée. Pour obtenir l’ensemble de vos abonnements, utilisez [az account list](/cli/azure/account#az-account-list).

## <a name="create-an-azure-database-for-postgresql-server"></a>Créer un serveur Azure Database pour PostgreSQL

Créez un [groupe de ressources Azure](../azure-resource-manager/management/overview.md) à l’aide de la commande [az group create](/cli/azure/group#az-group-create), puis créez votre serveur PostgreSQL à l’intérieur de ce groupe de ressources. Vous devez fournir un nom unique. L’exemple suivant crée un groupe de ressources nommé `myresourcegroup` à l’emplacement `westus`.

```azurecli-interactive
az group create --name myresourcegroup --location westus
```

Créez un [serveur Azure Database pour PostgreSQL](overview.md) au moyen de la commande [az postgres server create](/cli/azure/postgres/server). Un serveur peut contenir plusieurs bases de données.

```azurecli-interactive
az postgres server create --resource-group myresourcegroup --name mydemoserver  --location westus --admin-user myadmin --admin-password <server_admin_password> --sku-name GP_Gen5_2 
```
Voici les détails des arguments précédents : 

**Paramètre** | **Exemple de valeur** | **Description**
---|---|---
name | mydemoserver | Nom unique qui identifie votre serveur Azure Database pour PostgreSQL. Le nom de serveur ne peut contenir que des lettres minuscules, des chiffres et le caractère de trait d’union (-). Il doit contenir entre 3 et 63 caractères. Pour plus d’informations, consultez les [règles de nommage d’Azure Database pour PostgreSQL](../azure-resource-manager/management/resource-name-rules.md#microsoftdbforpostgresql).
resource-group | myResourceGroup | Nom du groupe de ressources Azure.
location | westus | Emplacement Azure du serveur.
admin-user | myadmin | Nom d’utilisateur du compte administrateur. Il ne peut pas être **azure_superuser** (superutilisateur_azure), **admin**, **administrator** (administrateur), **root** (racine), **guest** (invité) ni **public**.
admin-password | *mot de passe sécurisé* | Mot de passe de l’utilisateur Administrateur. Il doit contenir entre 8 et 128 caractères de trois des catégories suivantes : Lettres majuscules, lettres minuscules, chiffres et caractères non alphanumériques.
sku-name|GP_Gen5_2| Nom du niveau tarifaire et de la configuration de calcul. Suivez la convention {niveau tarifaire} _{génération de calcul}_ {vCores} sous forme abrégée. Pour plus d’informations, consultez [Niveaux tarifaires d’Azure Database pour PostgreSQL](https://azure.microsoft.com/pricing/details/postgresql/server/).

>[!IMPORTANT] 
>- La version PostgreSQL par défaut sur votre serveur est 9.6. Pour voir toutes les versions prises en charge, consultez [Versions majeures de PostgreSQL prises en charge](./concepts-supported-versions.md).
>- Pour voir tous les arguments de la commande **az postgres server create**, consultez [ce document de référence](/cli/azure/postgres/server#az-postgres-server-create).
>- Le protocole SSL est activé par défaut sur votre serveur. Pour plus d’informations sur le protocole SSL, consultez [Configurer la connectivité SSL](./concepts-ssl-connection-security.md).

## <a name="configure-a-server-level-firewall-rule"></a>Configurer une règle de pare-feu au niveau du serveur 
Par défaut, le serveur que vous avez créé n’est pas accessible publiquement et est protégé par des règles de pare-feu. Vous pouvez configurer la règle de pare-feu sur votre serveur au moyen de la commande [az postgres server firewall-rule create](/cli/azure/postgres/server/firewall-rule) afin d’accorder à votre environnement local un accès pour se connecter au serveur. 

L’exemple suivant crée une règle de pare-feu appelée `AllowMyIP` qui autorise les connexions d’une adresse IP spécifique, 192.168.0.1. Remplacez l’adresse IP ou la plage d’adresses IP qui correspond à l’emplacement de votre connexion. Si vous ne connaissez pas votre adresse IP, accédez à [WhatIsMyIPAddress.com](https://whatismyipaddress.com/) pour l’obtenir.


```azurecli-interactive
az postgres server firewall-rule create --resource-group myresourcegroup --server mydemoserver --name AllowMyIP --start-ip-address 192.168.0.1 --end-ip-address 192.168.0.1
```

> [!NOTE]
> Pour éviter les problèmes de connectivité, assurez-vous que le pare-feu de votre réseau autorise le port 5432. Les serveurs Azure Database pour PostgreSQL utilisent ce port. 

## <a name="get-the-connection-information"></a>Obtenir les informations de connexion

Pour vous connecter à votre serveur, fournissez des informations sur l’hôte et des informations d’identification pour l’accès.

```azurecli-interactive
az postgres server show --resource-group myresourcegroup --name mydemoserver
```

Le résultat est au format JSON. Prenez note des valeurs **administratorLogin** et **fullyQualifiedDomainName**.

```json
{
  "administratorLogin": "myadmin",
  "earliestRestoreDate": null,
  "fullyQualifiedDomainName": "mydemoserver.postgres.database.azure.com",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.DBforPostgreSQL/servers/mydemoserver",
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
  "type": "Microsoft.DBforPostgreSQL/servers",
  "userVisibleState": "Ready",
  "version": "9.6"
}
```

## <a name="connect-to-the-azure-database-for-postgresql-server-by-using-psql"></a>Se connecter au serveur Azure Database pour PostgreSQL à l’aide de psql
Le client [psql](https://www.postgresql.org/docs/current/static/app-psql.html) est un choix répandu pour la connexion aux serveurs PostgreSQL. Vous pouvez vous connecter à votre serveur par l’intermédiaire de psql avec [Azure Cloud Shell](../cloud-shell/overview.md). Vous pouvez également utiliser psql sur votre environnement local s’il est disponible. Une base de données vide, **postgres**, est automatiquement créée avec un nouveau serveur PostgreSQL. Vous pouvez utiliser cette base de données pour vous connecter à psql, comme illustré dans le code suivant. 

   ```bash
 psql --host=mydemoserver.postgres.database.azure.com --port=5432 --username=myadmin@mydemoserver --dbname=postgres
   ```

> [!TIP]
> Si vous préférez utiliser un chemin d’URL pour vous connecter à Postgres, encodez par URL le signe @ dans le nom d’utilisateur avec `%40`. La chaîne de connexion pour psql serait, par exemple :
>
> ```
> psql postgresql://myadmin%40mydemoserver@mydemoserver.postgres.database.azure.com:5432/postgres
> ```


## <a name="clean-up-resources"></a>Nettoyer les ressources
Si vous n’avez pas besoin de ces ressources pour un autre guide de démarrage rapide ou tutoriel, vous pouvez les supprimer en exécutant la commande suivante. 

```azurecli-interactive
az group delete --name myresourcegroup
```

Si vous voulez simplement supprimer le serveur nouvellement créé, vous pouvez exécuter la commande [az postgres server delete](/cli/azure/postgres/server).

```azurecli-interactive
az postgres server delete --resource-group myresourcegroup --name mydemoserver
```

## <a name="next-steps"></a>Étapes suivantes
> [!div class="nextstepaction"]
> [Effectuer la migration de votre base de données par exportation et importation](./howto-migrate-using-export-and-import.md)
