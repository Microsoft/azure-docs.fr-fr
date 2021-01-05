---
title: Sauvegarde et restauration - Azure CLI - Azure Database for MariaDB
description: Découvrez comment sauvegarder et restaurer un serveur dans Azure Database pour MySQL à l’aide d’Azure CLI.
author: savjani
ms.author: pariks
ms.service: mariadb
ms.devlang: azurecli
ms.topic: how-to
ms.date: 3/27/2020
ms.custom: devx-track-azurecli
ms.openlocfilehash: a6e46efd7f998437c3998df9a989ef9e1500e888
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94539581"
---
# <a name="how-to-back-up-and-restore-a-server-in-azure-database-for-mariadb-using-the-azure-cli"></a>Découvrez comment sauvegarder et restaurer un serveur dans Azure Database pour MariaDB à l’aide d’Azure CLI.

Les serveurs Azure Database for MariaDB sont sauvegardés régulièrement pour activer les fonctionnalités de restauration. À l’aide de cette fonctionnalité, vous pouvez restaurer le serveur et toutes ses bases de données à un point dans le temps antérieur, sur un nouveau serveur.

[!INCLUDE [quickstarts-free-trial-note](../../includes/quickstarts-free-trial-note.md)]

## <a name="prerequisites"></a>Prérequis

- Vous avez besoin d’un [serveur et d’une base de données Azure Database for MariaDB](quickstart-create-mariadb-server-database-using-azure-cli.md).

[!INCLUDE [azure-cli-prepare-your-environment-no-header.md](../../includes/azure-cli-prepare-your-environment-no-header.md)]

- Ce guide pratique nécessite la version 2.0 ou ultérieure d’Azure CLI. Si vous utilisez Azure Cloud Shell, la version la plus récente est déjà installée.

## <a name="set-backup-configuration"></a>Définir la configuration de sauvegarde

Vous choisissez entre la configuration de votre serveur pour des sauvegardes redondantes localement ou géographiquement redondantes lors de la création du serveur.

> [!NOTE]
> Après la création d’un serveur, son type de redondance (géographique ou locale) ne peut pas être modifié.
>

Lors de la création d’un serveur via la commande `az mariadb server create`, le paramètre `--geo-redundant-backup` décide de votre Option de redondance de sauvegarde. Si `Enabled`, des sauvegardes géo-redondantes sont effectuées. Ou si `Disabled`, des sauvegardes localement redondantes sont effectuées.

La période de rétention des sauvegardes est définie par le paramètre `--backup-retention`.

Pour plus d’informations sur la définition de ces valeurs lors de la création, consultez [Démarrage rapide du serveur Azure Database for MariaDB à partir de l’interface CLI](quickstart-create-mariadb-server-database-using-azure-cli.md).

La période de rétention des sauvegardes d’un serveur peut être modifiée comme suit :

```azurecli-interactive
az mariadb server update --name mydemoserver --resource-group myresourcegroup --backup-retention 10
```

L’exemple précédent modifie la période de période de rétention des sauvegardes de mydemoserver à 10 jours.

La période de rétention de sauvegarde détermine jusqu’à quelle date une restauration à un point dans le temps peut être récupérée, dans la mesure où elle est basée sur les sauvegardes disponibles. La limite de restauration dans le temps est décrite dans la section suivante.

## <a name="server-point-in-time-restore"></a>Limite de restauration dans le temps

Vous pouvez restaurer le serveur à une version antérieure. Les données restaurées sont copiées dans un nouveau serveur et le serveur existant est conservé tel quel. Par exemple, si une table est accidentellement supprimée à midi aujourd’hui, vous pouvez restaurer le serveur à l’état qu’il présentait juste avant midi. Vous pouvez ensuite récupérer la table et les données manquantes à partir de la copie restaurée du serveur.

Pour restaurer le serveur, utilisez la commande [az mariadb server restore](/cli/azure/mariadb/server#az-mariadb-server-restore) de l’interface Azure CLI.

### <a name="run-the-restore-command"></a>Exécuter la commande de restauration

Pour restaurer le serveur, à l’invite de commande de l’interface Azure CLI, entrez la commande suivante :

```azurecli-interactive
az mariadb server restore --resource-group myresourcegroup --name mydemoserver-restored --restore-point-in-time 2018-03-13T13:59:00Z --source-server mydemoserver
```

La commande `az mariadb server restore` requiert les paramètres suivants :

| Paramètre | Valeur suggérée | Description  |
| --- | --- | --- |
| resource-group |  myResourceGroup |  Groupe de ressources où se trouve le serveur source.  |
| name | mydemoserver-restored | Nom du serveur créé par la commande de restauration. |
| restore-point-in-time | 2018-03-13T13:59:00Z | Sélectionnez un état antérieur auquel effectuer la restauration. Elles doivent être comprises dans la période de rétention de la sauvegarde du serveur source. Utilisez le format de date et d’heure ISO8601. Par exemple, vous pouvez utiliser votre fuseau horaire local, comme `2018-03-13T05:59:00-08:00`. Vous pouvez également utiliser le format UTC Zulu, par exemple, `2018-03-13T13:59:00Z`. |
| source-server | mydemoserver | Nom ou identifiant du serveur source à partir duquel la restauration s’effectuera. |

Lorsque vous restaurez un serveur à un état antérieur, un nouveau serveur est créé. Le serveur d’origine et ses bases de données à l’état spécifié sont copiés sur le nouveau serveur.

Les valeurs d’emplacement et de niveau tarifaire du serveur restauré restent les mêmes que celles du serveur d’origine. 

Une fois la restauration terminée, recherchez le nouveau serveur et vérifiez que les données ont été restaurées correctement. Le nouveau serveur a le même nom de connexion d’administrateur et le même mot de passe que le serveur qui existait au moment du lancement de la restauration. Le mot de passe peut être modifié sur la page **Vue d’ensemble** du nouveau serveur.

Le nouveau serveur créé lors d’une restauration ne dispose pas des points de terminaison de service de réseau virtuel qui se trouvaient sur le serveur d’origine. Ces règles doivent être définies séparément pour ce nouveau serveur. Les règles de pare-feu du serveur d’origine sont restaurées.

## <a name="geo-restore"></a>Restauration géographique

Si vous avez configuré votre serveur pour les sauvegardes redondantes géographiquement, un serveur peut être créé à partir de la sauvegarde de ce serveur existant. Ce nouveau serveur peut être créé dans toutes les régions dans lesquelles Azure Database for MariaDB est disponible.  

Pour créer un serveur à l’aide d’une sauvegarde géo-redondante, utilisez la commande `az mariadb server georestore` Azure CLI.

> [!NOTE]
> Lorsqu’un serveur est créé, il est possible qu’il ne soit pas immédiatement disponible pour la restauration géographique. Le remplissage des métadonnées nécessaires peut prendre quelques heures.
>

Pour géo-restaurer le serveur, à l’invite de commande de l’interface de ligne de commande Azure, entrez la commande suivante :

```azurecli-interactive
az mariadb server georestore --resource-group myresourcegroup --name mydemoserver-georestored --source-server mydemoserver --location eastus --sku-name GP_Gen5_8
```

Cette commande crée un nouveau serveur appelé *mydemoserver-georestored* dans la région USA Est appartenant à *myresourcegroup*. Il s’agit d’un serveur à usage général, de 5e génération avec 8 vCores. Le serveur est créé à partir de la sauvegarde géo-redondante *mydemoserver*, qui est également dans le groupe de ressources *myresourcegroup*

Si vous souhaitez créer le nouveau serveur dans un autre groupe de ressources du serveur existant, dans le paramètre `--source-server`, vous devez qualifier le nom du serveur comme dans l’exemple suivant :

```azurecli-interactive
az mariadb server georestore --resource-group newresourcegroup --name mydemoserver-georestored --source-server "/subscriptions/$<subscription ID>/resourceGroups/$<resource group ID>/providers/Microsoft.DBforMariaDB/servers/mydemoserver" --location eastus --sku-name GP_Gen5_8

```

La commande `az mariadb server georestore` requiert les paramètres suivants :

| Paramètre | Valeur suggérée | Description  |
| --- | --- | --- |
|resource-group| myResourceGroup | Le nom du groupe de ressources auquel appartient le nouveau serveur.|
|name | mydemoserver-georestored | Le nom du nouveau serveur. |
|source-server | mydemoserver | Le nom du serveur existant dont les sauvegardes géo-redondantes sont utilisées. |
|location | eastus | L’emplacement du nouveau serveur. |
|sku-name| GP_Gen5_8 | Ce paramètre définit le niveau tarifaire, la génération de calculs et le nombre de vCores du nouveau serveur. GP_Gen5_8 mappe sur un serveur d’usage général, 5e génération avec 8 vCores.|

Lorsque vous créez un nouveau serveur par une restauration géographique, il hérite de la même taille de stockage et du même niveau tarifaire que le serveur source. Ces valeurs ne peuvent pas être modifiées lors de la création. Une fois le nouveau serveur créé, sa taille de stockage peut être montée en puissance.

Une fois la restauration terminée, recherchez le nouveau serveur et vérifiez que les données ont été restaurées correctement. Le nouveau serveur a le même nom de connexion d’administrateur et le même mot de passe que le serveur qui existait au moment du lancement de la restauration. Le mot de passe peut être modifié sur la page **Vue d’ensemble** du nouveau serveur.

Le nouveau serveur créé lors d’une restauration ne dispose pas des points de terminaison de service de réseau virtuel qui se trouvaient sur le serveur d’origine. Ces règles doivent être définies séparément pour ce nouveau serveur. Les règles de pare-feu du serveur d’origine sont restaurées.

## <a name="next-steps"></a>Étapes suivantes

- En savoir plus sur les [sauvegardes](concepts-backup.md) du service.
- En savoir plus sur les [réplicas](concepts-read-replicas.md).
- En savoir plus sur les options de [continuité d’activité](concepts-business-continuity.md).
