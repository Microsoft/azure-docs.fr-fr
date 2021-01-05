---
title: 'Démarrage rapide : Créer un serveur - Azure CLI - Azure Database pour MySQL - Serveur flexible'
description: Ce guide de démarrage rapide explique comment utiliser l'interface Azure CLI pour créer un serveur flexible Azure Database pour MySQL dans un groupe de ressources Azure.
author: savjani
ms.author: pariks
ms.service: mysql
ms.devlang: azurecli
ms.topic: quickstart
ms.date: 9/21/2020
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: 65cc3d2fdcbdea934e80a5f0012ca4f3da157ca3
ms.sourcegitcommit: 0a9df8ec14ab332d939b49f7b72dea217c8b3e1e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/18/2020
ms.locfileid: "94843432"
---
# <a name="quickstart-create-an-azure-database-for-mysql-flexible-server-using-azure-cli"></a>Démarrage rapide : Créer un serveur flexible Azure Database pour MySQL à l'aide d'Azure CLI

Ce guide de démarrage rapide montre comment utiliser les commandes [Azure CLI](/cli/azure/get-started-with-azure-cli) dans [Azure Cloud Shell](https://shell.azure.com) afin de créer un serveur flexible Azure Database pour MySQL en cinq minutes. Si vous n’avez pas d’abonnement Azure, créez un compte [gratuit](https://azure.microsoft.com/free/) avant de commencer.

> [!IMPORTANT] 
> La fonctionnalité Serveur flexible Azure Database pour MySQL est actuellement disponible en préversion publique

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

Créez un [groupe de ressources Azure](../../azure-resource-manager/management/overview.md) à l'aide de la commande `az group create`, puis créez votre serveur flexible MySQL à l'intérieur de ce groupe de ressources. Vous devez fournir un nom unique. L’exemple suivant crée un groupe de ressources nommé `myresourcegroup` à l’emplacement `eastus2`.

```azurecli-interactive
az group create --name myresourcegroup --location eastus2
```

Créez un serveur flexible à l'aide de la commande `az mysql flexible-server create`. Un serveur peut contenir plusieurs bases de données. La commande suivante crée un serveur en utilisant les valeurs par défaut du service et les valeurs issues du [contexte local](/cli/azure/local-context) de votre interface Azure CLI : 

```azurecli
az mysql flexible-server create
```

Le serveur créé possède les attributs suivants : 
- Nom du serveur généré automatiquement, nom d'utilisateur administrateur, mot de passe administrateur, nom du groupe de ressources (s'il n'est pas déjà spécifié dans le contexte local) et même emplacement que votre groupe de ressources 
- Valeurs par défaut du service pour les configurations restantes : Niveau de calcul (Modulable), Taille de calcul/SKU (B1MS), Période de rétention des sauvegardes (7 jours) et Version de MySQL (5.7)
- La méthode de connectivité par défaut est Accès privé (intégration au réseau virtuel), avec un réseau virtuel et un sous-réseau générés automatiquement

> [!NOTE] 
> Une fois le serveur créé, la méthode de connectivité ne peut pas être modifiée. Par exemple, si vous avez sélectionné *Accès privé (intégration au réseau virtuel)* lors de la création du serveur, vous ne pouvez pas passer à *Accès public (adresses IP autorisées)* à l'issue de cette création. Nous vous recommandons vivement de sélectionner l'Accès privé lors de la création d'un serveur afin de pouvoir y accéder en toute sécurité à l'aide de l'intégration au réseau virtuel. Pour en savoir plus sur l'accès privé, consultez l'[article consacré aux concepts](./concepts-networking.md).

Si vous souhaitez changer des valeurs par défaut, reportez-vous à la [documentation de référence](/cli/azure/mysql/flexible-server) d’Azure CLI pour obtenir la liste complète des paramètres CLI configurables. 

Voici un exemple de sortie : 

```json
Command group 'mysql flexible-server' is in preview. It may be changed/removed in a future release.
Creating Resource Group 'groupXXXXXXXXXX'...
Creating new vnet "serverXXXXXXXXXVNET" in resource group "groupXXXXXXXXXX"...
Creating new subnet "serverXXXXXXXXXSubnet" in resource group "groupXXXXXXXXXX" and delegating it to "Microsoft.DBforMySQL/flexibleServers"...
Creating MySQL Server 'serverXXXXXXXXX' in group 'groupXXXXXXXXXX'...
Your server 'serverXXXXXXXXX' is using sku 'Standard_B1ms' (Paid Tier). Please refer to https://aka.ms/mysql-pricing for pricing details
Creating MySQL database 'flexibleserverdb'...
Make a note of your password. If you forget, you would have to reset your password with 'az mysql flexible-server update -n serverXXXXXXXXX -g groupXXXXXXXXXX -p <new-password>'.
{
  "connectionString": "server=serverXXXXXXXXX.mysql.database.azure.com;database=flexibleserverdb;uid=secureusername;pwd=securepasswordstring",
  "databaseName": "flexibleserverdb",
  "host": "serverXXXXXXXXX.mysql.database.azure.com",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/groupXXXXXXXXXX/providers/Microsoft.DBforMySQL/flexibleServers/serverXXXXXXXXX",
  "location": "East US 2",
  "password": "securepasswordstring",
  "resourceGroup": "groupXXXXXXXXXX",
  "skuname": "Standard_B1ms",
  "subnetId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/groupXXXXXXXXXX/providers/Microsoft.Network/virtualNetworks/serverXXXXXXXXXVNET/subnets/serverXXXXXXXXXSubnet",
  "username": "secureusername",
  "version": "5.7"
}
```

Si vous souhaitez changer des valeurs par défaut, reportez-vous à la [documentation de référence](/cli/azure/mysql/flexible-server) d’Azure CLI pour obtenir la liste complète des paramètres CLI configurables. 

> [!NOTE]
> Les connexions à la base de données Azure pour MySQL communiquent via le port 3306. Si vous essayez de vous connecter à partir d’un réseau d’entreprise, le trafic sortant sur le port 3306 peut être bloqué. Si c’est le cas, vous ne pouvez pas vous connecter à votre serveur, sauf si votre service informatique ouvre le port 3306.

## <a name="get-the-connection-information"></a>Obtenir les informations de connexion

Pour vous connecter à votre serveur, vous devez fournir des informations sur l’hôte et des informations d’identification pour l’accès.

```azurecli-interactive
az mysql flexible-server show --resource-group myresourcegroup --name mydemoserver
```

Le résultat est au format JSON. Notez les valeurs **fullyQualifiedDomainName** et **administratorLogin**. Voici un exemple de sortie JSON : 

```json
{
  "administratorLogin": "myadminusername",
  "administratorLoginPassword": null,
  "delegatedSubnetArguments": {
    "subnetArmResourceId": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.Network/virtualNetworks/mydemoserverVNET/subnets/mydemoserverSubnet"
  },
  "fullyQualifiedDomainName": "mydemoserver.mysql.database.azure.com",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.DBforMySQL/flexibleServers/mydemoserver",
  "location": "East US 2",
  "name": "mydemoserver",
  "publicNetworkAccess": "Disabled",
  "resourceGroup": "myresourcegroup",
  "sku": {
    "capacity": 0,
    "name": "Standard_B1ms",
    "tier": "Burstable"
  },
  "storageProfile": {
    "backupRetentionDays": 7,
    "fileStorageSkuName": "Premium_LRS",
    "storageAutogrow": "Disabled",
    "storageIops": 0,
    "storageMb": 10240
  },
  "tags": null,
  "type": "Microsoft.DBforMySQL/flexibleServers",
  "version": "5.7"
}
```

## <a name="connect-using-mysql-command-line-client"></a>Se connecter à l'aide du client de ligne de commande mysql

Dans la mesure où le serveur flexible a été créé avec l'option *Accès privé (intégration au réseau virtuel)* , vous devez vous connecter à votre serveur à partir d'une ressource qui se trouve au sein du même réseau virtuel que votre serveur. Vous pouvez créer une machine virtuelle et l'ajouter au réseau virtuel créé. 

Une fois votre machine virtuelle créée, vous pouvez exécuter SSH sur celle-ci et installer l'outil de ligne de commande **[mysql.exe](https://dev.mysql.com/downloads/)** .

Avec mysql.exe, connectez-vous à l'aide de la commande ci-dessous. Remplacez les valeurs par le nom et le mot de passe réels de votre serveur. 

```bash
 mysql -h mydemoserver.mysql.database.azure.com -u mydemouser -p
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Si vous n’avez pas besoin de ces ressources pour un autre guide de démarrage rapide ou didacticiel, vous pouvez les supprimer en exécutant la commande suivante :

```azurecli-interactive
az group delete --name myresourcegroup
```

Si vous souhaitez simplement supprimer le serveur nouvellement créé, vous pouvez exécuter la commande `az mysql server delete`.

```azurecli-interactive
az mysql flexible-server delete --resource-group myresourcegroup --name mydemoserver
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
>[Créer une application web PHP (Laravel) avec MySQL](tutorial-php-database-app.md)
