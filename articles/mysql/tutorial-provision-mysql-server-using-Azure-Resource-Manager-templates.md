---
title: 'Tutoriel : Créer une base de données Azure Database pour MySQL - Modèle Azure Resource Manager'
description: Ce tutoriel explique comment provisionner et automatiser des déploiements de serveur Azure Database pour MySQL à l’aide d’un modèle Azure Resource Manager.
author: savjani
ms.author: pariks
ms.service: mysql
ms.devlang: json
ms.topic: tutorial
ms.date: 12/02/2019
ms.custom: mvc, devx-track-azurecli
ms.openlocfilehash: fd5fa4e09dd3f73aa7a8f044ded04d504bd2f3dd
ms.sourcegitcommit: e7152996ee917505c7aba707d214b2b520348302
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/20/2020
ms.locfileid: "97705470"
---
# <a name="tutorial-provision-an-azure-database-for-mysql-server-using-azure-resource-manager-template"></a>Tutoriel : Provisionner un serveur Azure Database pour MySQL à l’aide d’un modèle Azure Resource Manager

[L’API REST Azure Database pour MySQL](/rest/api/mysql/) permet aux ingénieurs DevOps d’automatiser et d’intégrer l’approvisionnement, la configuration et les opérations des bases de données et des serveurs MySQL gérés dans Azure.  L’API permet la création, l’énumération, la gestion et la suppression des bases de données et serveurs MySQL sur le service Azure Database pour MySQL.

Azure Resource Manager tire parti de l’API REST sous-jacente pour déclarer et programmer les ressources Azure nécessaires aux déploiements à grande échelle, en s’alignant sur l’infrastructure sous la forme d’un concept de code. Le modèle paramètre le nom de ressource Azure, la référence, le réseau, la configuration du pare-feu et les réglages. Vous pouvez ainsi le créer une fois et l’utiliser plusieurs fois.  Les modèles Azure Resource Manager peuvent être facilement créés à l’aide du [portail Azure](../azure-resource-manager/templates/quickstart-create-templates-use-the-portal.md) ou de [Visual Studio Code](../azure-resource-manager/templates/quickstart-create-templates-use-visual-studio-code.md?tabs=CLI). Ils permettent la création d’applications, la normalisation et l’automatisation du déploiement, qui peuvent être intégrées dans le pipeline CI/CD DevOps.  Par exemple, si vous cherchez à déployer rapidement une application web avec le serveur principal Azure Database pour MySQL, vous pouvez effectuer le déploiement de bout en bout à l’aide de ce [modèle de démarrage rapide](https://azure.microsoft.com/resources/templates/101-webapp-managed-mysql/) depuis la galerie GitHub.

Dans ce tutoriel, vous allez utiliser un modèle Azure Resource Manager et d’autres utilitaires pour apprendre à :

> [!div class="checklist"]
> * Créer un serveur Azure Database pour MySQL avec le point de terminaison de service de réseau virtuel à l’aide du modèle Azure Resource Manager
> * Utiliser [l’outil de ligne de commande mysql](https://dev.mysql.com/doc/refman/5.6/en/mysql.html) pour créer une base de données
> * Charger un exemple de données
> * Interroger des données
> * Mettre à jour des données

## <a name="prerequisites"></a>Prérequis

Si vous n’avez pas d’abonnement Azure, créez un [compte Azure gratuit](https://azure.microsoft.com/free/) avant de commencer.

## <a name="create-an-azure-database-for-mysql-server-with-vnet-service-endpoint-using-azure-resource-manager-template"></a>Créer un serveur Azure Database pour MySQL avec le point de terminaison de service de réseau virtuel à l’aide du modèle Azure Resource Manager

Pour connaître la référence de modèle JSON d’un serveur Azure Database pour MySQL, consultez les informations de référence sur les modèles des [serveurs Microsoft.DBforMySQL](/azure/templates/microsoft.dbformysql/servers). Vous pouvez utiliser l’exemple de modèle JSON ci-dessous pour créer un nouveau serveur exécutant Azure Database pour MySQL avec le point de terminaison de service de réseau virtuel.
```json
{
  "apiVersion": "2017-12-01",
  "type": "Microsoft.DBforMySQL/servers",
  "name": "string",
  "location": "string",
  "tags": "string",
  "properties": {
    "version": "string",
    "sslEnforcement": "string",
    "administratorLogin": "string",
    "administratorLoginPassword": "string",
    "storageProfile": {
      "storageMB": "string",
      "backupRetentionDays": "string",
      "geoRedundantBackup": "string"
    }
  },
  "sku": {
    "name": "string",
    "tier": "string",
    "capacity": "string",
    "family": "string"
  },
  "resources": [
    {
      "name": "AllowSubnet",
      "type": "virtualNetworkRules",
      "apiVersion": "2017-12-01",
      "properties": {
        "virtualNetworkSubnetId": "[resourceId('Microsoft.Network/virtualNetworks/subnets', parameters('virtualNetworkName'), parameters('subnetName'))]",
        "ignoreMissingVnetServiceEndpoint": true
      },
      "dependsOn": [
        "[concat('Microsoft.DBforMySQL/servers/', parameters('serverName'))]"
      ]
    }
  ]
}
```
Dans cette requête, vous devez personnaliser les valeurs suivantes :
+   `name` -Spécifiez le nom de votre serveur MySQL (sans nom de domaine).
+   `location` -Spécifiez une région de centre de données Azure valide pour votre serveur MySQL. Par exemple, westus2.
+   `properties/version` -Spécifiez la version de serveur MySQL à déployer. Par exemple, 5.6 ou 5.7.
+   `properties/administratorLogin` -Spécifiez la connexion d’administrateur MySQL pour le serveur. Le nom de connexion d’administrateur ne doit pas être azure_superuser, admin, administrator, root, guest ou public.
+   `properties/administratorLoginPassword` -Spécifiez le mot de passe pour l’utilisateur administrateur de MySQL spécifié ci-dessus.
+   `properties/sslEnforcement` -Spécifiez Activé/Désactivé pour activer/désactiver sslEnforcement.
+   `storageProfile/storageMB` -Spécifiez la taille maximale de stockage approvisionné requise pour le serveur en mégaoctets. Par exemple, 5120.
+   `storageProfile/backupRetentionDays` - Spécifiez la période de rétention de sauvegarde souhaitée en jours. Par exemple, 7. 
+   `storageProfile/geoRedundantBackup` -Spécifiez Activé/Désactivé selon les exigences de géo-reprise d’activité.
+   `sku/tier` -Spécifiez le niveau Basique, Usage général ou Mémoire optimisée pour le déploiement.
+   `sku/capacity` -Spécifiez la capacité de vCore. Les valeurs possibles sont 2, 4, 8, 16, 32 ou 64.
+   `sku/family` - Spécifiez Gen5 pour choisir la génération du matériel pour le déploiement du serveur.
+   `sku/name` -Spécifiez la valeur TierPrefix_family_capacity. Par exemple B_Gen5_1, GP_Gen5_16, MO_Gen5_32. Pour comprendre les valeurs valides par région et par niveau, consultez la documentation sur les [niveaux tarifaires](./concepts-pricing-tiers.md).
+   `resources/properties/virtualNetworkSubnetId` -Spécifiez l’identificateur Azure du sous-réseau de réseau virtuel dans lequel placer le serveur Azure MySQL. 
+   `tags(optional)` -Spécifiez les balises facultatives comme paires clé/valeur que vous utiliseriez pour catégoriser les ressources pour la facturation, etc.

Si vous souhaitez créer un modèle Azure Resource Manager pour automatiser les déploiements Azure Database pour MySQL de votre organisation, nous vous recommandons de commencer par l’exemple de [modèle Azure Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/101-managed-mysql-with-vnet) dans la galerie GitHub Démarrage rapide Azure et de poursuivre à partir de là. 

Si vous ne connaissez pas encore les modèles Azure Resource Manager et que vous souhaitez les essayer, commencez en procédant comme suit :
+   Clonez ou téléchargez l’exemple de [modèle Azure Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/101-managed-mysql-with-vnet) à partir de la galerie Démarrage rapide Azure.  
+   Modifiez azuredeploy.parameters.json pour mettre à jour les valeurs de paramètre en fonction de vos préférences, puis enregistrez le fichier. 
+   Utilisez Azure CLI pour créer le serveur Azure MySQL à l’aide des commandes suivantes

Vous pouvez utiliser Azure Cloud Shell dans le navigateur ou installer Azure CLI sur votre ordinateur pour exécuter les blocs de code de ce tutoriel.

[!INCLUDE [cloud-shell-try-it](../../includes/cloud-shell-try-it.md)]

```azurecli-interactive
az login
az group create -n ExampleResourceGroup  -l "West US2"
az deployment group create -g $ ExampleResourceGroup   --template-file $ {templateloc} --parameters $ {parametersloc}
```

## <a name="get-the-connection-information"></a>Obtenir les informations de connexion
Pour vous connecter à votre serveur, vous devez fournir des informations sur l’hôte et des informations d’identification pour l’accès.
```azurecli-interactive
az mysql server show --resource-group myresourcegroup --name mydemoserver
```

Le résultat est au format JSON. Notez les valeurs **fullyQualifiedDomainName** et **administratorLogin**.
```json
{
  "administratorLogin": "myadmin",
  "administratorLoginPassword": null,
  "fullyQualifiedDomainName": "mydemoserver.mysql.database.azure.com",
  "id": "/subscriptions/00000000-0000-0000-0000-000000000000/resourceGroups/myresourcegroup/providers/Microsoft.DBforMySQL/servers/mydemoserver",
  "location": "westus2",
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

## <a name="connect-to-the-server-using-mysql"></a>Se connecter au serveur à l’aide de mysql
Utilisez [l’outil de ligne de commande mysql](https://dev.mysql.com/doc/refman/5.6/en/mysql.html) pour établir une connexion à votre serveur Azure Database pour MySQL. Dans cet exemple, la commande est :
```cmd
mysql -h mydemoserver.database.windows.net -u myadmin@mydemoserver -p
```

## <a name="create-a-blank-database"></a>Créer une base de données vide
Une fois que vous êtes connecté au serveur, créez une base de données vide.
```sql
mysql> CREATE DATABASE mysampledb;
```

À l’invite, exécutez la commande suivante pour basculer la connexion sur la base de données nouvellement créée :
```sql
mysql> USE mysampledb;
```

## <a name="create-tables-in-the-database"></a>Créer des tables dans la base de données
Maintenant que vous savez vous connecter à la base de données Azure Database pour MySQL, effectuez certaines tâches de base.

Tout d’abord, créez une table et chargez-y des données. Nous allons créer une table qui stocke des données d’inventaire.
```sql
CREATE TABLE inventory (
    id serial PRIMARY KEY, 
    name VARCHAR(50), 
    quantity INTEGER
);
```

## <a name="load-data-into-the-tables"></a>Charger des données dans les tables
Maintenant que vous disposez d’une table, insérez-y des données. Dans la fenêtre d’invite de commandes ouverte, exécutez la requête suivante pour insérer des lignes de données.
```sql
INSERT INTO inventory (id, name, quantity) VALUES (1, 'banana', 150); 
INSERT INTO inventory (id, name, quantity) VALUES (2, 'orange', 154);
```

Vous avez maintenant chargé deux lignes de données dans la table que vous avez créée précédemment.

## <a name="query-and-update-the-data-in-the-tables"></a>Interroger et mettre à jour les données des tables
Exécutez la requête suivante pour récupérer des informations à partir de la table de base de données.
```sql
SELECT * FROM inventory;
```

Vous pouvez également mettre à jour les données des tables.
```sql
UPDATE inventory SET quantity = 200 WHERE name = 'banana';
```

La ligne est mise à jour en conséquence lorsque vous récupérez les données.
```sql
SELECT * FROM inventory;
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Quand vous n’en avez plus besoin, supprimez le groupe de ressources, ce qui supprime également les ressources qu’il contient.

# <a name="portal"></a>[Portail](#tab/azure-portal)

1. Dans le [portail Azure](https://portal.azure.com), recherchez et sélectionnez **Groupes de ressources**.

2. Dans la liste des groupes de ressources, choisissez le nom de votre groupe de ressources.

3. Dans la page **Vue d’ensemble** de votre groupe de ressources, sélectionnez **Supprimer le groupe de ressources**.

4. Dans la boîte de dialogue de confirmation, entrez le nom de votre groupe de ressources, puis sélectionnez **Supprimer**.

# <a name="powershell"></a>[PowerShell](#tab/PowerShell)

```azurepowershell-interactive
$resourceGroupName = Read-Host -Prompt "Enter the Resource Group name"
Remove-AzResourceGroup -Name $resourceGroupName
Write-Host "Press [ENTER] to continue..."
```

# <a name="cli"></a>[INTERFACE DE LIGNE DE COMMANDE](#tab/CLI)

```azurecli-interactive
echo "Enter the Resource Group name:" &&
read resourceGroupName &&
az group delete --name $resourceGroupName &&
echo "Press [ENTER] to continue ..."
```

---

## <a name="next-steps"></a>Étapes suivantes
Dans ce didacticiel, vous avez appris à effectuer les opérations suivantes :
> [!div class="checklist"]
> * Créer un serveur Azure Database pour MySQL avec le point de terminaison de service de réseau virtuel à l’aide du modèle Azure Resource Manager
> * Utiliser l’outil en ligne de commande mysql pour créer une base de données
> * Charger un exemple de données
> * Interroger des données
> * Mettre à jour des données

> [!div class="nextstepaction"]
> [Connexion d’applications à la base de données Azure pour MySQL](./howto-connection-string.md)