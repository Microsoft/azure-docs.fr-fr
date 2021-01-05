---
title: 'Démarrage rapide : Créer un serveur - Azure CLI - Azure Database pour PostgreSQL – Serveur flexible'
description: Ce guide de démarrage rapide explique comment utiliser l'interface Azure CLI pour créer un serveur flexible Azure Database pour PostgreSQL dans un groupe de ressources Azure.
author: lfittl-msft
ms.author: lufittl
ms.service: postgresql
ms.devlang: azurecli
ms.topic: quickstart
ms.date: 9/22/2020
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: d43fab43b05ccfd88308390f58ba48ff9111b295
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94842819"
---
# <a name="quickstart-create-an-azure-database-for-postgresql-flexible-server-using-azure-cli"></a>Démarrage rapide : Créer un serveur flexible Azure Database pour PostgreSQL à l'aide d'Azure CLI

Ce guide de démarrage rapide montre comment utiliser les commandes [Azure CLI](/cli/azure/get-started-with-azure-cli) dans [Azure Cloud Shell](https://shell.azure.com) afin de créer un serveur flexible Azure Database pour PostgreSQL en cinq minutes. Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

> [!IMPORTANT] 
> Azure Database pour PostgreSQL - Serveur flexible est en actuellement en préversion.

## <a name="launch-azure-cloud-shell"></a>Lancement d’Azure Cloud Shell

Le service [Azure Cloud Shell](../../cloud-shell/overview.md) est un interpréteur de commandes interactif et gratuit que vous pouvez utiliser pour exécuter les étapes décrites dans cet article. Il contient des outils Azure courants préinstallés et configurés pour être utilisés avec votre compte.

Pour ouvrir Cloud Shell, sélectionnez simplement **Essayer** en haut à droite d’un bloc de code. Vous pouvez également ouvrir Cloud Shell dans un onglet distinct du navigateur en accédant à [https://shell.azure.com/bash](https://shell.azure.com/bash). Sélectionnez **Copier** pour copier les blocs de code, collez-les dans Cloud Shell et sélectionnez **Entrée** pour les exécuter.

Si vous préférez installer et utiliser l'interface de ligne de commande localement, ce guide de démarrage rapide nécessite au minimum la version 2.0 d'Azure CLI. Exécutez `az --version` pour trouver la version. Si vous devez installer ou mettre à niveau, voir [Installer Azure CLI](/cli/azure/install-azure-cli).

## <a name="prerequisites"></a>Prérequis

Vous devrez vous connecter à votre compte à l’aide de la commande [az login](/cli/azure/reference-index#az-login). Notez la propriété **id**, qui fait référence à l’**ID d’abonnement** pour votre compte Azure.

```azurecli-interactive
az login
```

Sélectionnez l’abonnement spécifique sous votre compte à l’aide de la commande [az account set](/cli/azure/account#az-account-set). Notez la valeur **id** issue de la commande **az login** ; vous devez vous en servir comme valeur de l’argument **subscription** dans la commande. Si vous avez plusieurs abonnements, sélectionnez l’abonnement approprié dans lequel la ressource doit être facturée. Pour accéder à l’ensemble de votre abonnement, utilisez [az account list](/cli/azure/account#az-account-list).

```azurecli
az account set --subscription <subscription id>
```

## <a name="create-a-flexible-server"></a>Créer un serveur flexible

Créez un [groupe de ressources Azure](../../azure-resource-manager/management/overview.md) à l'aide de la commande `az group create`, puis créez votre serveur flexible PostgreSQL à l'intérieur de ce groupe de ressources. Vous devez fournir un nom unique. L’exemple suivant crée un groupe de ressources nommé `myresourcegroup` à l’emplacement `westus`.

```azurecli-interactive
az group create --name myresourcegroup --location westus
```

Créez un serveur flexible à l'aide de la commande `az postgres flexible-server create`. Un serveur peut contenir plusieurs bases de données. La commande suivante crée un serveur en utilisant les valeurs par défaut du service et les valeurs issues du [contexte local](/cli/azure/local-context) de votre interface Azure CLI : 

```azurecli
az postgres flexible-server create
```

Le serveur créé possède les attributs suivants : 
- Nom du serveur généré automatiquement, nom d'utilisateur administrateur, mot de passe administrateur, nom du groupe de ressources (s'il n'est pas déjà spécifié dans le contexte local) et même emplacement que votre groupe de ressources 
- Valeurs par défaut du service pour les configurations restantes : Niveau de calcul (Usage général), Taille de calcul/SKU (D2s_v3 - 2 vCore, 8 Go de RAM), Période de rétention des sauvegardes (7 jours) et Version de PostgreSQL (12)
- La méthode de connectivité par défaut est Accès privé (intégration au réseau virtuel), avec un réseau virtuel et un sous-réseau générés automatiquement

> [!NOTE] 
> Une fois le serveur créé, la méthode de connectivité ne peut pas être modifiée. Par exemple, si vous avez sélectionné *Accès privé (intégration au réseau virtuel)* lors de la création du serveur, vous ne pouvez pas passer à *Accès public (adresses IP autorisées)* à l'issue de cette création. Nous vous recommandons vivement de sélectionner l'Accès privé lors de la création d'un serveur afin de pouvoir y accéder en toute sécurité à l'aide de l'intégration au réseau virtuel. Pour en savoir plus sur l'accès privé, consultez l'[article consacré aux concepts](./concepts-networking.md).

Si vous souhaitez modifier les valeurs par défaut, reportez-vous à la documentation de référence de l'interface de ligne de commande Azure <!--FIXME --> pour obtenir la liste complète des paramètres configurables de celle-ci. 

> [!NOTE]
> Les connexions à Azure Database pour PostgreSQL communiquent sur le port 5432. Si vous essayez de vous connecter à partir d’un réseau d’entreprise, le trafic sortant sur le port 5432 peut être bloqué. Dans ce cas, vous ne pouvez pas vous connecter à votre serveur, sauf si votre service informatique ouvre le port 5432.

## <a name="get-the-connection-information"></a>Obtenir les informations de connexion

Pour vous connecter à votre serveur, vous devez fournir des informations sur l’hôte et des informations d’identification pour l’accès.

```azurecli-interactive
az postgres flexible-server show --resource-group myresourcegroup --name mydemoserver
```

Le résultat est au format JSON. Notez les valeurs **fullyQualifiedDomainName** et **administratorLogin**.

<!--FIXME-->
```json
{
  "administratorLogin": "myadmin",
  "earliestRestoreDate": null,
  "fullyQualifiedDomainName": "mydemoserver.postgres.database.azure.com",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.DBforPostgreSQL/flexibleServers/mydemoserver",
  "location": "westus",
  "name": "mydemoserver",
  "resourceGroup": "myresourcegroup",
  "sku": {
    "capacity": 2,
    "name": "Standard_D2s_v3",
    "size": null,
    "tier": "GeneralPurpose"
  },
  "publicAccess": "Enabled",
  "storageProfile": {
    "backupRetentionDays": 7,
    "geoRedundantBackup": "Disabled",
    "storageMb": 131072
  },
  "tags": null,
  "type": "Microsoft.DBforPostgreSQL/flexibleServers",
  "userVisibleState": "Ready",
  "version": "12"
}
```

## <a name="connect-using-postgresql-command-line-client"></a>Se connecter à l'aide du client de ligne de commande PostgreSQL

Dans la mesure où le serveur flexible a été créé avec l'option *Accès privé (intégration au réseau virtuel)* , vous devez vous connecter à votre serveur à partir d'une ressource qui se trouve au sein du même réseau virtuel que votre serveur. Vous pouvez créer une machine virtuelle et l'ajouter au réseau virtuel créé. 

Une fois votre machine virtuelle créée, vous pouvez exécuter SSH sur celle-ci et installer l'outil de ligne de commande **[psql](https://www.postgresql.org/download/)** .

Avec psql, connectez-vous à l'aide de la commande ci-dessous. Remplacez les valeurs par le nom et le mot de passe réels de votre serveur. 

```bash
psql -h mydemoserver.postgres.database.azure.com -u mydemouser -p
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous n’avez pas besoin de ces ressources pour un autre guide de démarrage rapide ou didacticiel, vous pouvez les supprimer en exécutant la commande suivante :

```azurecli-interactive
az group delete --name myresourcegroup
```

Si vous souhaitez simplement supprimer le serveur nouvellement créé, vous pouvez exécuter la commande `az postgres flexible-server delete`.

```azurecli-interactive
az postgres flexible-server delete --resource-group myresourcegroup --name mydemoserver
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
>[Déployer une application Django avec App Service et PostgreSQL](tutorial-django-app-service-postgres.md)
