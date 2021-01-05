---
title: Empêcher la suppression ou la modification des ressources Azure Cosmos DB
description: Utilisez les verrous de ressources Azure pour empêcher la suppression ou la modification des ressources Azure Cosmos DB.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 10/06/2020
ms.author: mjbrown
ms.openlocfilehash: 1c8c766208132aec115e1fbeb15af3a057c3de3e
ms.sourcegitcommit: 295db318df10f20ae4aa71b5b03f7fb6cba15fc3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/15/2020
ms.locfileid: "94636688"
---
# <a name="prevent-azure-cosmos-db-resources-from-being-deleted-or-changed"></a>Empêcher la suppression ou la modification des ressources Azure Cosmos DB
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

En tant qu’administrateur, vous pouvez avoir besoin de verrouiller un compte, une base de données ou un conteneur Azure Cosmos afin d’empêcher les utilisateurs de votre organisation de supprimer ou de modifier de manière accidentelle des ressources critiques. Vous pouvez définir le niveau de verrouillage sur CanNotDelete ou ReadOnly.

- **CanNotDelete** signifie que les utilisateurs autorisés peuvent lire et modifier une ressource, mais pas la supprimer.
- **ReadOnly** signifie que les utilisateurs autorisés peuvent lire une ressource, mais pas la supprimer ni la mettre à jour. Appliquer ce verrou revient à limiter à tous les utilisateurs autorisés les autorisations accordées par le rôle Lecteur.

## <a name="how-locks-are-applied"></a>Application des verrous

Lorsque vous appliquez un verrou à une étendue parente, toutes les ressources de cette étendue héritent du même verrou. Même les ressources que vous ajoutez par la suite héritent du verrou du parent. Le verrou le plus restrictif de l’héritage est prioritaire.

Contrairement au contrôle d'accès basé sur les rôles Azure, vous utilisez des verrous de gestion pour appliquer une restriction à tous les utilisateurs et rôles. Pour plus d’informations sur les rôles Azure RBAC pour Azure Cosmos DB, consultez [Contrôle d’accès en fonction du rôle Azure dans Azure Cosmos DB](role-based-access-control.md).

Les verrous Resource Manager s'appliquent uniquement aux opérations qui se produisent dans le plan de gestion, c'est-à-dire les opérations envoyées à https://management.azure.com. Les verrous ne limitent pas la manière dont les ressources exécutent leurs propres fonctions. Les modifications des ressources sont limitées, mais pas les opérations sur les ressources. Par exemple, un verrou ReadOnly appliqué à un conteneur Azure Cosmos vous empêchera de supprimer ou de modifier le conteneur. Il ne vous empêche pas de créer, de mettre à jour ou de supprimer des données dans le conteneur. Les transactions de données sont autorisées, car ces opérations ne sont pas envoyées à https://management.azure.com.

## <a name="manage-locks"></a>Gérer les verrous

> [!WARNING]
> Les verrous de ressources ne fonctionnent pas pour les modifications qui sont apportées par les utilisateurs qui accèdent à Azure Cosmos DB à l’aide de clés de compte, sauf si le compte Azure Cosmos est d’abord verrouillé par l’activation de la propriété disableKeyBasedMetadataWriteAccess. Avant d’activer cette propriété, vérifiez bien qu’elle n’empêche pas l’exécution des applications existantes qui modifient les ressources à l’aide d’un SDK, du portail Azure ou d’outils tiers en se connectant par le biais de clés de compte, et qui modifient les ressources en modifiant le débit, en mettant à jour les stratégies d’index, etc. Pour obtenir plus d’informations et voir la liste de vérification visant à garantir la bonne exécution de vos applications, consultez [Prévention des modifications des Kits de développement logiciel (SDK) Azure Cosmos DB](role-based-access-control.md#prevent-sdk-changes).

### <a name="powershell"></a>PowerShell

```powershell
$resourceGroupName = "myResourceGroup"
$accountName = "my-cosmos-account"
$lockName = "$accountName-Lock"

# First, update the account to prevent changes by anything that connects via account keys
Update-AzCosmosDBAccount -ResourceGroupName $resourceGroupName -Name $accountName -DisableKeyBasedMetadataWriteAccess true

# Create a Delete Lock on an Azure Cosmos account resource and all child resources
New-AzResourceLock `
    -ApiVersion "2020-04-01" `
    -ResourceType "Microsoft.DocumentDB/databaseAccounts" `
    -ResourceGroupName $resourceGroupName `
    -ResourceName $accountName `
    -LockName $lockName `
    -LockLevel "CanNotDelete" # CanNotDelete or ReadOnly
```

### <a name="azure-cli"></a>Azure CLI

```bash
resourceGroupName='myResourceGroup'
accountName='my-cosmos-account'
$lockName="$accountName-Lock"

# First, update the account to prevent changes by anything that connects via account keys
az cosmosdb update  --name $accountName --resource-group $resourceGroupName  --disable-key-based-metadata-write-access true

# Create a Delete Lock on an Azure Cosmos account resource
az lock create --name $lockName \
    --resource-group $resourceGroupName \
    --resource-type Microsoft.DocumentDB/databaseAccount \
    --lock-type 'CanNotDelete' # CanNotDelete or ReadOnly \
    --resource $accountName
```

### <a name="template"></a>Modèle

Pour appliquer un verrou à une ressource Azure Cosmos DB, utilisez les formats suivants :

- nom : `{resourceName}/Microsoft.Authorization/{lockName}`
- type : `{resourceProviderNamespace}/{resourceType}/providers/locks`

> [!IMPORTANT]
> Si vous modifiez un compte Azure Cosmos existant, veillez à inclure les autres propriétés de votre compte et de vos ressources enfants lorsque vous effectuez le redéploiement avec cette propriété. Ne déployez pas ce modèle tel quel, car toutes les propriétés de votre compte seraient réinitialisées.

```json
"resources": [
    {
        "type": "Microsoft.DocumentDB/databaseAccounts",
        "name": "[variables('accountName')]",
        "apiVersion": "2020-04-01",
        "kind": "GlobalDocumentDB",
        "location": "[parameters('location')]",
        "properties": {
            "consistencyPolicy": "[variables('consistencyPolicy')[parameters('defaultConsistencyLevel')]]",
            "locations": "[variables('locations')]",
            "databaseAccountOfferType": "Standard",
            "enableAutomaticFailover": "[parameters('automaticFailover')]",
            "disableKeyBasedMetadataWriteAccess": true
        }
    },
    {
        "type": "Microsoft.DocumentDB/databaseAccounts/providers/locks",
        "apiVersion": "2020-04-01",
        "name": "[concat(variables('accountName'), '/Microsoft.Authorization/siteLock')]",
        "dependsOn": [
        "[resourceId('Microsoft.DocumentDB/databaseAccounts', variables('accountName'))]"
        ],
        "properties": {
        "level": "CanNotDelete",
        "notes": "Cosmos account should not be deleted."
        }
    }
]
```

## <a name="next-steps"></a>Étapes suivantes

- [Vue d’ensemble des verrous Azure Resource Manager](../azure-resource-manager/management/lock-resources.md)
