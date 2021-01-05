---
title: Gérer un serveur – Azure CLI – Azure Database pour PostgreSQL – Serveur flexible
description: Découvrez comment gérer un serveur Azure Database pour PostgreSQL – Serveur flexible à partir d’Azure CLI.
author: mksuni
ms.author: sumuth
ms.service: postgresql
ms.topic: how-to
ms.date: 09/22/2020
ms.openlocfilehash: 278f8f816909a7e365d7e45d04c5169950e79a65
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96493676"
---
# <a name="manage-an-azure-database-for-postgresql---flexible-server-by-using-the-azure-cli"></a>Gérer une instance Azure Database pour PostgreSQL – Serveur flexible à l’aide d’Azure CLI

> [!IMPORTANT]
> Azure Database pour PostgreSQL – Serveur flexible est disponible en préversion.

Cet article explique comment gérer un serveur flexible déployé sur Azure. Les tâches de gestion incluent notamment la mise à l’échelle du calcul et du stockage, la réinitialisation de mot de passe et l’affichage des informations relatives au serveur.

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer. 

Vous devez exécuter Azure CLI version 2.0 ou ultérieure localement. Pour afficher la version installée, exécutez la commande `az --version`. Si vous devez effectuer une installation ou une mise à niveau, consultez [Installer Azure CLI](/cli/azure/install-azure-cli).

Connectez-vous à votre compte à l’aide de la commande [az login](/cli/azure/reference-index#az-login). 

```azurecli-interactive
az login
```

Sélectionnez votre abonnement à l’aide de la commande [az account set](/cli/azure/account). Notez la valeur **id** issue de la sortie **az login**. Vous devez vous en servir comme valeur pour l’argument **subscription** dans la commande suivante. Si vous avez plusieurs abonnements, sélectionnez celui sur lequel la ressource doit être facturée. Pour identifier tous vos abonnements, utilisez la commande [az account list](/cli/azure/account#az-account-list).

```azurecli
az account set --subscription <subscription id>
```

> [!Important]
> Si vous n’avez pas encore créé de serveur flexible, vous devez le faire pour suivre ce guide pratique.

## <a name="scale-compute-and-storage"></a>Mettre à l’échelle le calcul et le stockage

Vous pouvez facilement effectuer un scale-up de votre niveau de calcul, de vos vCores et de votre stockage à l’aide de la commande suivante. Pour obtenir la liste de toutes les opérations de serveur que vous pouvez effectuer, consultez la vue d’ensemble de la commande [az postgres flexible-server](/cli/azure/postgres/flexible-server).

```azurecli-interactive
az postgres flexible-server update --resource-group myresourcegroup --name mydemoserver --sku-name Standard_D4ds_v3 --storage-size 6144
```

Voici les détails des arguments dans le code précédent :

**Paramètre** | **Exemple de valeur** | **Description**
---|---|---
name | mydemoserver | Entrez un nom unique pour votre serveur. Le nom de serveur ne peut contenir que des lettres minuscules, des chiffres et le caractère de trait d’union (-). Il doit contenir entre 3 et 63 caractères.
resource-group | myResourceGroup | Indiquez le nom du groupe de ressources Azure.
sku-name|Standard_D4ds_v3|Entrez le nom du niveau de calcul et de la taille. La valeur respecte la convention *Standard_{taille de machine virtuelle}* en abrégé. Pour plus d’informations, consultez les [niveaux tarifaires](../concepts-pricing-tiers.md).
storage-size | 6144 | Entrez la capacité de stockage du serveur en mégaoctets. La valeur minimale est 5120. Elle peut être augmentée par incréments de 1 024.

> [!IMPORTANT]
> Vous ne pouvez pas effectuer un scale-down du stockage. 

## <a name="manage-postgresql-databases-on-a-server"></a>Gérer des bases de données PostgreSQL sur un serveur

Vous pouvez utiliser différentes applications pour vous connecter à votre serveur de base de données Azure pour PostgreSQL. Si PostgreSQL est installé sur votre ordinateur client, vous pouvez utiliser une instance locale de [psql](https://www.postgresql.org/docs/current/static/app-psql.html). Nous allons maintenant utiliser l’outil de ligne de commande psql pour nous connecter au serveur Azure Database pour PostgreSQL.

1. Exécutez la commande **psql** suivante :

   ```bash
   psql --host=<servername> --port=<port> --username=<user> --dbname=<dbname>
   ```

   Par exemple, la commande suivante se connecte à la base de données par défaut appelée **postgres** sur votre serveur PostgreSQL **mydemoserver.postgres.database.azure.com** en utilisant vos informations d’identification d’accès. Lorsque vous y êtes invité, entrez le `<server_admin_password>` que vous avez choisi.
  
   ```bash
   psql --host=mydemoserver.postgres.database.azure.com --port=5432 --username=myadmin --dbname=postgres
   ```

   Une fois la connexion établie, l’outil psql affiche une invite **postgres** dans laquelle vous pouvez taper des commandes SQL. Un avertissement s’affiche dans la sortie de la connexion initiale si la version de psql que vous utilisez est différente de la version installée sur le serveur Azure Database pour PostgreSQL.

   Exemple de sortie psql :

   ```bash
   psql (11.3, server 12.1)
   WARNING: psql major version 11, server major version 12.
            Some psql features might not work.
   SSL connection (protocol: TLSv1.2, cipher: ECDHE-RSA-AES256-GCM-SHA384, bits: 256, compression: off)
   Type "help" for help.

   postgres=>
   ```

   > [!TIP]
   > Si le pare-feu n’est pas configuré pour autoriser l’adresse IP de votre client, l’erreur suivante se produit :
   >
   > « psql: FATAL: aucune entrée pg_hba.conf pour l’hôte `<IP address>`, utilisateur « myadmin », base de données « postgres », SSL sur FATAL : Connexion SSL obligatoire. Spécifiez les options SSL puis réessayez. »
   >
   > Confirmez que l’adresse IP de votre client est autorisée dans les règles de pare-feu.

2. Créez une base de données vide nommée **postgresdb** à l’invite en tapant la commande suivante :

    ```bash
    CREATE DATABASE postgresdb;
    ```

3. À l’invite, exécutez la commande suivante pour basculer les connexions sur la nouvelle base de données **postgresdb** :

    ```bash
    \c postgresdb
    ```

4. Tapez `\q`, puis sélectionnez Entrée pour quitter psql.

Dans cette section, vous vous êtes connecté au serveur Azure Database pour PostgreSQL via psql, et avez créé une base de données utilisateur vide.

## <a name="reset-the-admin-password"></a>Réinitialiser le mot de passe d’administrateur

Vous pouvez modifier le mot de passe du rôle Administrateur avec la commande suivante :

```azurecli-interactive
az postgres flexible-server update --resource-group myresourcegroup --name mydemoserver --admin-password <new-password>
```

> [!IMPORTANT]
> Choisissez un mot de passe d’une longueur comprise entre 8 et 128 caractères. Le mot de passe doit contenir des caractères de trois des catégories suivantes : 
> - Lettres majuscules de l’alphabet anglais
> - Lettres minuscules de l’alphabet
> - Nombres
> - Caractères non alphanumériques

## <a name="delete-a-server"></a>Supprimer un serveur

Pour supprimer le serveur flexible Azure Database pour PostgreSQL, exécutez la commande [az postgres flexible-server delete](/cli/azure/postgres/flexible-server#az-PostgreSQL-flexible-server-delete).

```azurecli-interactive
az postgres flexible-server delete --resource-group myresourcegroup --name mydemoserver
```

## <a name="next-steps"></a>Étapes suivantes

- [Présentation des concepts de sauvegarde et de restauration](concepts-backup-restore.md)
- [Régler et superviser le serveur](concepts-monitoring.md)