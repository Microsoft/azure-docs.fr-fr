---
title: 'Démarrage rapide : Créer une base de données Azure Database pour MySQL en utilisant az mysql up'
description: Guide de démarrage rapide pour créer un serveur Azure Database pour MySQL à l’aide de la commande up dans Azure CLI (interface de ligne de commande).
author: savjani
ms.author: pariks
ms.service: mysql
ms.devlang: azurecli
ms.topic: quickstart
ms.date: 3/18/2020
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: d89cc41ed26124ae4ad2e6689be6d59278c3d9da
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94542165"
---
# <a name="quickstart-create-an-azure-database-for-mysql-using-a-simple-azure-cli-command---az-mysql-up-preview"></a>Démarrage rapide : Créer un serveur Azure Database pour MySQL à l’aide d’une simple commande Azure CLI - az mysql up (préversion)

> [!IMPORTANT]
> La commande Azure CLI [az mysql up](/cli/azure/ext/db-up/mysql#ext-db-up-az-mysql-up) est disponible en préversion.

Base de données Azure pour MySQL est un service géré qui vous permet d’exécuter, de gérer et de mettre à l’échelle des bases de données MySQL hautement disponibles dans le cloud. L’interface Azure CLI permet de créer et gérer des ressources Azure à partir de la ligne de commande ou à l’aide de scripts. Ce guide de démarrage rapide vous montre comment utiliser la commande [az mysql up](/cli/azure/ext/db-up/mysql#ext-db-up-az-mysql-up) pour créer un serveur Azure Database pour MySQL à partir d’Azure CLI. En plus de créer le serveur, la commande `az mysql up` crée un exemple de base de données, un utilisateur racine dans la base de données, ouvre le pare-feu pour les services Azure et crée des règles de pare-feu par défaut pour l’ordinateur client. Cela permet d’accélérer le processus de développement.

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

Cet article nécessite que vous exécutiez localement Azure CLI version 2.0 ou ultérieure. Pour afficher la version installée, exécutez la commande `az --version`. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI](/cli/azure/install-azure-cli).

Vous devrez vous connecter à votre compte à l’aide de la commande [az login](/cli/azure/authenticate-azure-cli). Notez la propriété **id** depuis la sortie de commande pour le nom d’abonnement correspondant.

```azurecli
az login
```

Si vous avez plusieurs abonnements, sélectionnez l’abonnement approprié dans lequel la ressource doit être facturée. Sélectionnez l’ID d’abonnement spécifique sous votre compte à l’aide de la commande [az account set](/cli/azure/account). Remplacez la propriété **ID d’abonnement** de la sortie **az login** pour votre abonnement dans l’espace réservé de l’ID d’abonnement.

```azurecli
az account set --subscription <subscription id>
```

## <a name="create-an-azure-database-for-mysql-server"></a>Création d’un serveur Azure Database pour MySQL

Pour utiliser les commandes, installez l’extension [db-up](/cli/azure/ext/db-up). Si une erreur est retournée, vérifiez que vous avez installé la dernière version d’Azure CLI. Consultez [Installer Azure CLI](/cli/azure/install-azure-cli).

```azurecli
az extension add --name db-up
```

Créez un serveur Azure Database pour MySQL à l’aide de la commande suivante :

```azurecli
az mysql up
```

Le serveur est créé avec les valeurs par défaut suivantes (sauf si vous les remplacez manuellement) :

**Paramètre** | **Valeur par défaut** | **Description**
---|---|---
server-name | Générée par le système | Un nom unique qui identifie votre serveur Azure Database pour MySQL.
resource-group | Générée par le système | Nouveau groupe de ressources Azure.
sku-name | GP_Gen5_2 | Nom du la référence SKU. Suit la convention {niveau tarifaire}\_{génération de calcul}\_{vCores} dans le raccourci. Par défaut, il s’agit d’un serveur à usage général de 5e génération à 2 vCores. Consultez notre [page de tarifs](https://azure.microsoft.com/pricing/details/mysql/) pour plus d’informations sur les niveaux de service.
backup-retention | 7 | Durée pendant laquelle la sauvegarde doit être conservée. Exprimée en jours.
geo-redundant-backup | Désactivé | Indique si les sauvegardes géoredondantes doivent être activées ou non pour ce serveur.
location | westus2 | Emplacement Azure du serveur.
ssl-enforcement | activé | Indique si le protocole SSL doit être activé ou non pour ce serveur.
storage-size | 5120 | Capacité de stockage du serveur (en mégaoctets).
version | 5.7 | Version principale de MySQL.
admin-user | Générée par le système | Nom d’utilisateur du compte administrateur.
admin-password | Générée par le système | Mot de passe de l’utilisateur Administrateur.

> [!NOTE]
> Pour plus d’informations sur la commande `az mysql up` et ses paramètres supplémentaires, consultez la [documentation Azure CLI](/cli/azure/ext/db-up/mysql#ext-db-up-az-mysql-up).

Une fois que votre serveur est créé, il présente les paramètres suivants :

- Une règle de pare-feu appelée « devbox » est créée. Azure CLI tente de détecter l’adresse IP de la machine à partir de laquelle la commande `az mysql up` est exécutée et autorise cette adresse IP.
- « Autoriser l’accès aux services Azure » est défini sur ACTIVÉ. Ce paramètre configure le pare-feu du serveur pour qu’il accepte les connexions de toutes les ressources Azure, y compris celles qui ne font pas partie de votre abonnement.
- Le paramètre `wait_timeout` est défini sur 8 heures
- Une base de données nommée « sampledb » est créée
- Un utilisateur nommé « root » est créé avec des privilèges « sampledb »

> [!NOTE]
> Azure Database pour MySQL communique sur le port 3306. Quand vous vous connectez à partir d’un réseau d’entreprise, il se peut que le trafic sortant via le port 3306 ne soit pas autorisé par le pare-feu de votre réseau. Pour vous connecter à votre serveur, demandez à votre service informatique d’ouvrir le port 3306.

## <a name="get-the-connection-information"></a>Obtenir les informations de connexion

Une fois que la commande `az mysql up` a abouti, une liste de chaînes de connexion pour les langages de programmation les plus courants vous est retournée. Ces chaînes de connexion sont préconfigurées avec les attributs spécifiques de votre nouveau serveur Azure Database pour MySQL.

Vous pouvez utiliser la commande [az mysql show-connection-string](/cli/azure/ext/db-up/mysql#ext-db-up-az-mysql-show-connection-string) pour lister à nouveau ces chaînes de connexion.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Supprimez toutes les ressources que vous avez créées au cours de ce démarrage rapide à l’aide de la commande ci-dessous. Cette commande supprime le serveur Azure Database pour MySQL et le groupe de ressources.

```azurecli
az mysql down --delete-group
```

Si vous souhaitez simplement supprimer le serveur nouvellement créé, vous pouvez exécuter la commande [az mysql down](/cli/azure/ext/db-up/mysql#ext-db-up-az-mysql-down).

```azurecli
az mysql down
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Conception d’une base de données MySQL avec Azure CLI](./tutorial-design-database-using-cli.md)
