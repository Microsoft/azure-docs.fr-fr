---
title: 'Démarrage rapide : Créer un serveur – az postgres up – Azure Database pour PostgreSQL – Serveur unique'
description: Guide de démarrage rapide pour créer une base de données Azure pour PostgreSQL(serveur unique) à l’aide de la commande up d’Azure CLI (interface de ligne de commande).
author: lfittl-msft
ms.author: lufittl
ms.service: postgresql
ms.devlang: azurecli
ms.topic: quickstart
ms.date: 05/06/2019
ms.custom: devx-track-azurecli
ms.openlocfilehash: c11a98c04f81e7c111faf4aa449ab2bf9a4890bb
ms.sourcegitcommit: 3bcce2e26935f523226ea269f034e0d75aa6693a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92485220"
---
# <a name="quickstart-use-an-azure-cli-command-az-postgres-up-preview-to-create-an-azure-database-for-postgresql---single-server"></a>Démarrage rapide : Utiliser une commande Azure CLI, az postgres up (préversion), pour créer une base de données Azure pour PostgreSQL (serveur unique)

> [!IMPORTANT]
> La commande Azure CLI [az postgres up](/cli/azure/ext/db-up/postgres#ext-db-up-az-postgres-up) est disponible en préversion.

Base de données Azure pour PostgreSQL est un service géré qui vous permet d’exécuter, de gérer et de mettre à l’échelle des bases de données PostgreSQL hautement disponibles dans le cloud. L’interface de ligne de commande (CLI) Azure permet de créer et gérer des ressources Azure à partir de la ligne de commande ou dans les scripts. Ce démarrage rapide vous montre comment utiliser la commande [az postgres up](/cli/azure/ext/db-up/postgres#ext-db-up-az-postgres-up) pour créer un serveur Azure Database pour PostgreSQL à partir d’Azure CLI. En plus de créer le serveur, la commande `az postgres up` crée un exemple de base de données, un utilisateur racine dans la base de données, ouvre le pare-feu pour les services Azure et crée des règles de pare-feu par défaut pour l’ordinateur client. Ces valeurs par défaut aident à accélérer le processus de développement.

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

Cet article nécessite que vous exécutiez localement Azure CLI version 2.0 ou ultérieure. Pour afficher la version installée, exécutez la commande `az --version`. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI](/cli/azure/install-azure-cli).

Vous devrez vous connecter à votre compte avec la commande [az login](/cli/azure/authenticate-azure-cli). Notez la propriété **ID** de la sortie de commande pour le nom d’abonnement correspondant.

```azurecli
az login
```

Si vous avez plusieurs abonnements, sélectionnez l’abonnement approprié dans lequel la ressource doit être facturée. Sélectionnez l’ID d’abonnement spécifique sous votre compte à l’aide de la commande [az account set](/cli/azure/account). Remplacez la propriété **ID d’abonnement** de la sortie **az login** pour votre abonnement dans l’espace réservé de l’ID d’abonnement.

```azurecli
az account set --subscription <subscription id>
```

## <a name="create-an-azure-database-for-postgresql-server"></a>Créer un serveur Azure Database pour PostgreSQL

Pour utiliser les commandes, installez l’extension [db-up](/cli/azure/ext/db-up). Si une erreur est retournée, vérifiez que vous avez installé la dernière version d’Azure CLI. Consultez [Installer Azure CLI](/cli/azure/install-azure-cli).

```azurecli
az extension add --name db-up
```

Créez un serveur Azure Database pour PostgreSQL à l’aide de la commande suivante :

```azurecli
az postgres up
```

Le serveur est créé avec les valeurs par défaut suivantes (sauf si vous les remplacez manuellement) :

**Paramètre** | **Valeur par défaut** | **Description**
---|---|---
server-name | Générée par le système | Un nom unique qui identifie votre serveur Azure Database pour PostgreSQL.
resource-group | Générée par le système | Nouveau groupe de ressources Azure.
sku-name | GP_Gen5_2 | Nom du la référence SKU. Suit la convention {niveau tarifaire}\_{génération de calcul}\_{vCores} dans le raccourci. Par défaut, il s’agit d’un serveur à usage général de 5e génération à 2 vCores. Consultez notre [page de tarifs](https://azure.microsoft.com/pricing/details/postgresql/) pour plus d’informations sur les niveaux de service.
backup-retention | 7 | Durée de conservation d’une sauvegarde. Exprimée en jours.
geo-redundant-backup | Désactivé | Indique si les sauvegardes géoredondantes doivent être activées ou non pour ce serveur.
location | westus2 | Emplacement Azure du serveur.
ssl-enforcement | Désactivé | Indique si le protocole TLS/SSL doit être activé ou non pour ce serveur.
storage-size | 5120 | Capacité de stockage du serveur (en mégaoctets).
version | 10 | La version principale de PostgreSQL.
admin-user | Générée par le système | Le nom d’utilisateur pour l’administrateur.
admin-password | Générée par le système | Mot de passe de l’utilisateur Administrateur.

> [!NOTE]
> Pour plus d’informations sur la commande `az postgres up` et ses paramètres supplémentaires, consultez la [documentation Azure CLI](/cli/azure/ext/db-up/postgres#ext-db-up-az-postgres-up).

Une fois que votre serveur est créé, il présente les paramètres suivants :

- Une règle de pare-feu appelée « devbox » est créée. Azure CLI tente de détecter l’adresse IP de la machine à partir de laquelle la commande `az postgres up` est exécutée et autorise cette adresse IP.
- « Autoriser l’accès aux services Azure » est défini sur ACTIVÉ. Ce paramètre configure le pare-feu du serveur pour qu’il accepte les connexions de toutes les ressources Azure, y compris celles qui ne font pas partie de votre abonnement.
- Une base de données nommée « sampledb » est créée
- Un utilisateur nommé « root » est créé avec des privilèges « sampledb »

> [!NOTE]
> Azure Database pour PostgreSQL communique sur le port 5432. Lorsque vous vous connectez à partir d’un réseau d’entreprise, le trafic sortant sur le port 5432 peut être bloqué par le pare-feu de votre réseau. Pour vous connecter à votre serveur, demandez à votre service informatique d’ouvrir le port 5432.

## <a name="get-the-connection-information"></a>Obtenir les informations de connexion

Une fois que la commande `az postgres up` a abouti, une liste de chaînes de connexion pour les langages de programmation les plus courants vous est retournée. Ces chaînes de connexion sont préconfigurées avec les attributs spécifiques de votre nouveau serveur Azure Database pour PostgreSQL.

Vous pouvez utiliser la commande [az postgres show-connection-string](/cli/azure/ext/db-up/postgres#ext-db-up-az-postgres-show-connection-string) pour lister à nouveau ces chaînes de connexion.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Supprimez toutes les ressources que vous avez créées au cours de ce démarrage rapide à l’aide de la commande ci-dessous. Cette commande supprime le serveur Azure Database pour PostgreSQL et le groupe de ressources.

```azurecli
az postgres down --delete-group
```

Si vous souhaitez simplement supprimer le serveur nouvellement créé, vous pouvez exécuter la commande [az postgres down](/cli/azure/ext/db-up/postgres#ext-db-up-az-postgres-down).

```azurecli
az postgres down
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [Migration de votre base de données PostgreSQL par exportation et importation](./howto-migrate-using-export-and-import.md)
