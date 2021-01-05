---
title: 'Démarrage rapide : Créer un serveur – Azure CLI – Azure Database for MariaDB'
description: Ce guide de démarrage rapide explique comment utiliser l’interface CLI Azure pour créer un serveur Azure Database for MariaDB dans un groupe de ressources Azure.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.devlang: azurecli
ms.topic: quickstart
ms.date: 3/18/2020
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 3279150d0cb7b287f0a78581094a51356033596c
ms.sourcegitcommit: 5e5a0abe60803704cf8afd407784a1c9469e545f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96435737"
---
# <a name="quickstart-create-an-azure-database-for-mariadb-server-by-using-the-azure-cli"></a>Démarrage rapide : Créer un serveur Azure Database for MariaDB à l’aide d’Azure CLI

Vous pouvez utiliser Azure CLI pour créer et gérer des ressources Azure à partir de la ligne de commande ou dans des scripts. Ce guide de démarrage rapide explique comment utiliser l’interface Azure CLI pour créer en quelque cinq minutes un serveur Azure Database for MariaDB dans un groupe de ressources Azure.

Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

[!INCLUDE [azure-cli-prepare-your-environment.md](../../includes/azure-cli-prepare-your-environment.md)]

- Cet article nécessite la version 2.0 ou ultérieure d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

Si vous avez plusieurs abonnements, choisissez celui dans lequel se trouve la ressource ou celui pour lequel vous êtes facturé. Pour sélectionner un ID d’abonnement dans votre compte, utilisez la commande [az account set](/cli/azure/account#az-account-set) :

```azurecli-interactive
az account set --subscription 00000000-0000-0000-0000-000000000000
```

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Créez un [groupe de ressources Azure](../azure-resource-manager/management/overview.md) avec la commande [az group create](/cli/azure/group#az-group-create). Un groupe de ressources est un conteneur logique dans lequel les ressources Azure sont déployées et gérées en tant que groupe.

L’exemple suivant crée un groupe de ressources nommé `myresourcegroup` à l’emplacement `westus` :

```azurecli-interactive
az group create --name myresourcegroup --location westus
```

## <a name="create-an-azure-database-for-mariadb-server"></a>Créer un serveur Azure Database for MariaDB

Créez un serveur Azure Database for MariaDB avec la commande [az mariadb server create](/cli/azure/mariadb/server#az-mariadb-server-create). Un serveur peut gérer plusieurs bases de données. En règle générale, une base de données distincte est utilisée pour chaque projet ou pour chaque utilisateur.

Paramètre | Exemple de valeur | Description
---|---|---
name | **mydemoserver** | Entrez un nom unique qui identifie votre serveur Azure Database for MariaDB. Le nom de serveur ne peut contenir que des lettres minuscules, des chiffres et le caractère de trait d’union (-). Il doit contenir entre 3 et 63 caractères.
resource-group | **myresourcegroup** | Entez le nom du groupe de ressources Azure.
sku-name | **GP_Gen5_2** | Nom de la référence SKU. Suit la convention *niveau tarifaire*\_*génération de calcul*\_*vCores* dans le raccourci. Pour plus d’informations sur le paramètre **sku-name**, consultez la section après ce tableau.
backup-retention | **7** | Durée pendant laquelle la sauvegarde doit être conservée. Exprimée en jours. Plage : 7 à 35. 
geo-redundant-backup | **Désactivé** | Indique si les sauvegardes géoredondantes doivent être activées ou non pour ce serveur. Valeurs autorisées : **Activé**, **Désactivé**.
location | **westus** | Emplacement Azure du serveur.
ssl-enforcement | **Activé** | Indique si le protocole SSL doit être activé ou non pour ce serveur. Valeurs autorisées : **Activé**, **Désactivé**.
storage-size | **51200** | Capacité de stockage du serveur (exprimée en mégaoctets). Les tailles de stockage valides sont 5 120 Mo (minimum) avec des augmentations par incréments de 1 024 Mo. Consultez le document [Niveaux tarifaires pour Azure Database for MariaDB](./concepts-pricing-tiers.md) pour plus d’informations sur les limites de taille de stockage. 
version | **10.2** | Version majeure du moteur MariaDB.
admin-user | **myadmin** | Nom d’utilisateur du compte administrateur. Le paramètre **admin-user** ne peut pas être **azure_superuser**, **admin**, **administrator**, **root**, **guest** ou **public**.
admin-password | *votre mot de passe* | Mot de passe de l’utilisateur Administrateur. Votre mot de passe doit contenir entre 8 et 128 caractères. Il doit contenir des caractères de trois des catégories suivantes : Lettres majuscules, lettres minuscules, chiffres et caractères non alphanumériques.

La valeur du paramètre sku-name suit la convention {tarification}\_{génération de calcul}\_{vCores} comme dans les exemples ci-dessous :
+ `--sku-name B_Gen5_1` correspond à De base, Gen 5 et 1 vCore. Cette option correspond à la plus petite référence disponible.
+ `--sku-name GP_Gen5_32` correspond à Usage général, Gen 5 et 32 vCores.
+ `--sku-name MO_Gen5_2` correspond à Mémoire optimisée, Gen 5 et 2 vCores.

Pour plus d’informations sur les valeurs valides par région et pour les niveaux, consultez [Niveaux tarifaires pour Azure Database for MariaDB](./concepts-pricing-tiers.md).

L’exemple suivant crée un serveur nommé **mydemoserver** dans la région USA Ouest. Le serveur se trouve dans le groupe de ressources **myresourcegroup** et a la connexion d’administrateur du serveur **myadmin**. Le serveur est de type Gen 5, appartient au niveau tarifaire Usage général et possède 2 vCores. Un nom de serveur est mappé à un nom DNS et doit être globalement unique dans Azure. Remplacez `<server_admin_password>` par votre propre mot de passe d’administrateur du serveur.

```azurecli-interactive
az mariadb server create --resource-group myresourcegroup --name mydemoserver  --location westus --admin-user myadmin --admin-password <server_admin_password> --sku-name GP_Gen5_2 --version 10.2
```

> [!NOTE]
> Choisissez le niveau tarifaire De base si votre charge de travail n’a pas besoin d’une grande capacité de calcul et d’E/S. Notez que les serveurs créés avec le niveau tarifaire De base ne peuvent plus ensuite être mis à l’échelle vers le niveau Usage général ou Mémoire optimisée. Pour plus d’informations, consultez la [page des tarifs](https://azure.microsoft.com/pricing/details/mariadb/).

## <a name="configure-a-firewall-rule"></a>Configurer une règle de pare-feu

Créez une règle de pare-feu au niveau du serveur Azure Database for MariaDB avec la commande [az mariadb server firewall-rule create](/cli/azure/mariadb/server/firewall-rule#az-mariadb-server-firewall-rule-create). Une règle de pare-feu au niveau du serveur permet à une application externe, comme l’outil en ligne de commande mysql ou MySQL Workbench de se connecter à votre serveur via le pare-feu du service Azure Database for MariaDB.

L’exemple suivant crée une règle de pare-feu appelée `AllowMyIP` qui autorise les connexions d’une adresse IP spécifique, 192.168.0.1. Remplacez une adresse IP ou une plage d’adresses IP qui correspond à l’emplacement à partir duquel vous vous connectez.

```azurecli-interactive
az mariadb server firewall-rule create --resource-group myresourcegroup --server mydemoserver --name AllowMyIP --start-ip-address 192.168.0.1 --end-ip-address 192.168.0.1
```

> [!NOTE]
> Les connexions à Azure Database for MariaDB communiquent sur le port 3306. Si vous essayez de vous connecter depuis un réseau d’entreprise, le trafic sortant sur le port 3306 peut être bloqué. Dans ce cas, vous pouvez vous connecter à votre serveur uniquement si votre service informatique ouvre le port 3306.

## <a name="configure-ssl-settings"></a>Configurer les paramètres SSL

Par défaut, des connexions SSL entre votre serveur et les applications clientes sont appliquées. Ce paramètre par défaut garantit la sécurité des données « en mouvement » en chiffrant le flux de données sur Internet. Pour ce démarrage rapide, désactivez les connexions SSL de votre serveur. La désactivation des connexions SSL n’est pas recommandée pour les serveurs de production. Pour plus d’informations, voir [Configuration de la connectivité SSL dans votre application pour se connecter de manière sécurisée à Azure Database for MariaDB](./howto-configure-ssl.md).

L’exemple suivant désactive l’application du protocole SSL sur votre serveur Azure Database for MariaDB :

```azurecli-interactive
az mariadb server update --resource-group myresourcegroup --name mydemoserver --ssl-enforcement Disabled
```

## <a name="get-connection-information"></a>Obtenir des informations de connexion

Pour vous connecter à votre serveur, vous devez fournir des informations sur l’hôte et des informations d’identification pour l’accès. Pour obtenir les informations de connexion, exécutez la commande suivante :

```azurecli-interactive
az mariadb server show --resource-group myresourcegroup --name mydemoserver
```

Le résultat est au format JSON. Notez les valeurs des champs **fullyQualifiedDomainName** et **administratorLogin**.

```json
{
  "administratorLogin": "myadmin",
  "earliestRestoreDate": null,
  "fullyQualifiedDomainName": "mydemoserver.mariadb.database.azure.com",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.DBforMariaDB/servers/mydemoserver",
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
  "type": "Microsoft.DBforMariaDB/servers",
  "userVisibleState": "Ready",
  "version": "10.2"
}
```

## <a name="connect-to-the-server-by-using-the-mysql-command-line-tool"></a>Se connecter au serveur avec l’outil en ligne de commande mysql

Connectez-vous au serveur avec l’outil en ligne de commande mysql. Vous pouvez [télécharger](https://dev.mysql.com/downloads/) cet outil et l’installer sur votre ordinateur. Vous pouvez également accéder à l’outil en ligne de commande en sélectionnant le bouton **Try It** (Essayer) sur un exemple de code dans cet article. Une autre façon d’accéder à l’outil en ligne de commande consiste à sélectionner le bouton **>_** dans la barre d’outils en haut à droite dans le Portail Azure pour ouvrir **Azure Cloud Shell**.

Connectez-vous au serveur avec l’outil en ligne de commande mysql :

1. Connectez-vous au serveur :

   ```azurecli-interactive
   mysql -h mydemoserver.mariadb.database.azure.com -u myadmin@mydemoserver -p
   ```

2. Affichez l’état du serveur à l’invite `mysql>` :

   ```sql
   status
   ```

   Vous devez voir un texte similaire à ce qui suit :

   ```cmd
   C:\Users\>mysql -h mydemoserver.mariadb.database.azure.com -u myadmin@mydemoserver -p
   Enter password: ***********
   Welcome to the MySQL monitor.  Commands end with ; or \g.
   Your MySQL connection id is 65512
   Server version: 5.6.39.0 MariaDB Server

   Copyright (c) 2000, 2016, Oracle and/or its affiliates. All rights reserved.

   Oracle is a registered trademark of Oracle Corporation and/or its
   affiliates. Other names may be trademarks of their respective
   owners.

   Type 'help;' or '\h' for help. Type '\c' to clear the current input statement.

   mysql> status
   --------------
   mysql  Ver 14.14 Distrib 5.7.23, for Linux (x86_64)

   Connection id:          64681
   Current database:
   Current user:           myadmin@40.118.201.21
   SSL:                    Cipher in use is AES256-SHA
   Current pager:          stdout
   Using outfile:          ''
   Using delimiter:        ;
   Server version:         5.6.39.0 MariaDB Server
   Protocol version:       10
   Connection:             mydemoserver.mariadb.database.azure.com via TCP/IP
   Server characterset:    latin1
   Db     characterset:    latin1
   Client characterset:    utf8
   Conn.  characterset:    utf8
   TCP port:               3306
   Uptime:                 1 day 3 hours 28 min 50 sec

   Threads: 10  Questions: 29002  Slow queries: 0  Opens: 33  Flush tables: 3  Open tables: 1  Queries per second avg: 0.293
   --------------

   mysql>
   ```

> [!TIP]
> Pour les autres commandes, consultez le [Manuel de référence de MySQL 5.7 - Chapitre 4.5.1](https://dev.mysql.com/doc/refman/5.7/en/mysql.html).

## <a name="connect-to-the-server-by-using-mysql-workbench"></a>Se connecter au serveur à l’aide de MySQL Workbench

1. Ouvrez MySQL Workbench sur votre ordinateur client. S’il n’est pas déjà installé, [téléchargez](https://dev.mysql.com/downloads/workbench/) et installez l’application.

2. Dans la boîte de dialogue **Configurer une nouvelle connexion**, entrez les informations suivantes dans l’onglet **Paramètres** :

   ![Configurer une nouvelle connexion](./media/quickstart-create-mariadb-server-database-using-azure-cli/setup-new-connection.png)

   | Paramètre | Valeur suggérée | Description |
   |---|---|---|
   | Nom de connexion | **Connexion démo** | Entrez une étiquette pour cette connexion (il peut s’agir de n’importe quoi) |
   | Méthode de connexion | **Standard (TCP/IP)** | Utilisez le protocole TCP/IP pour vous connecter à Azure Database for MariaDB |
   | HostName | **mydemoserver.mariadb.database.azure.com** | Le nom du serveur que vous avez noté précédemment. |
   | Port | **3306** | Le port par défaut pour Azure Database for MariaDB. |
   | Nom d’utilisateur | **myadmin\@mydemoserver** | La connexion d’administrateur du serveur que vous avez notée précédemment. |
   | Mot de passe | *votre mot de passe* | Utilisez le mot de passe du compte administrateur que vous avez configuré précédemment. |

3. Sélectionnez **Tester la connexion** pour vérifier que tous les paramètres sont correctement configurés.

4. Sélectionnez la connexion pour vous connecter au serveur.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous n’avez pas besoin des ressources que vous avez utilisées dans ce démarrage rapide pour un autre démarrage rapide ou didacticiel, vous pouvez les supprimer en exécutant la commande suivante : 

```azurecli-interactive
az group delete --name myresourcegroup
```

Si vous souhaitez supprimer uniquement le serveur que vous avez créé dans ce démarrage rapide, exécutez la commande [az mariadb server delete](/cli/azure/mariadb/server#az-mariadb-server-delete) :

```azurecli-interactive
az mariadb server delete --resource-group myresourcegroup --name mydemoserver
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Concevoir une base de données MariaDB avec Azure CLI](tutorial-design-database-cli.md)
