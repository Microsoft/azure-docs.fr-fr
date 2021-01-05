---
title: Découvrez comment gérer les comptes de base de données dans Azure Cosmos DB
description: Apprenez à gérer les ressources Azure Cosmos DB à l’aide du portail Azure, de PowerShell, de l’interface CLI et des modèles Azure Resource Manager.
author: markjbrown
ms.service: cosmos-db
ms.subservice: cosmosdb-sql
ms.topic: how-to
ms.date: 09/18/2020
ms.author: mjbrown
ms.openlocfilehash: d8763a794d2fb96d0c464fb1249b9eb400fd23e7
ms.sourcegitcommit: fa90cd55e341c8201e3789df4cd8bd6fe7c809a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93339867"
---
# <a name="manage-an-azure-cosmos-account"></a>Gérer un compte Azure Cosmos
[!INCLUDE[appliesto-sql-api](includes/appliesto-sql-api.md)]

Cet article décrit comment gérer différentes tâches sur un compte Azure Cosmos à l’aide du portail Azure, d’Azure PowerShell, d’Azure CLI et de modèles Azure Resource Manager.

## <a name="create-an-account"></a>Créer un compte

### <a name="azure-portal"></a><a id="create-database-account-via-portal"></a>Portail Azure

[!INCLUDE [cosmos-db-create-dbaccount](../../includes/cosmos-db-create-dbaccount.md)]

### <a name="azure-cli"></a><a id="create-database-account-via-cli"></a>Interface CLI Azure

Voir [Créer un compte Azure Cosmos DB avec Azure CLI](manage-with-cli.md#create-an-azure-cosmos-db-account)

### <a name="azure-powershell"></a><a id="create-database-account-via-ps"></a>Azure PowerShell

Veuillez consulter [Créer un compte Azure Cosmos DB avec PowerShell](manage-with-powershell.md#create-account)

### <a name="azure-resource-manager-template"></a><a id="create-database-account-via-arm-template"></a>Modèle Azure Resource Manager

Veuillez consulter [Créer un compte Azure Cosmos DB avec des modèles Azure Resource Manager](./manage-with-templates.md)

## <a name="addremove-regions-from-your-database-account"></a>Ajouter/supprimer des régions à partir de votre compte de base de données

### <a name="azure-portal"></a><a id="add-remove-regions-via-portal"></a>Portail Azure

1. Connectez-vous au [portail Azure](https://portal.azure.com).

1. Accédez à votre compte Azure Cosmos, puis ouvrez le menu **Répliquer les données globalement**.

1. Pour ajouter des régions, sélectionnez les hexagones sur la carte avec l’étiquette **+** correspondante. Sinon, pour ajouter une région, sélectionnez l’option **+ Ajouter une région** , puis choisissez une région dans le menu déroulant.

1. Pour supprimer des régions, désactivez une ou plusieurs régions à partir de la carte en sélectionnant les hexagones bleus avec des coches. Vous pouvez aussi sélectionner l’icône de la « Corbeille » (🗑) en regard de la région, sur le côté droit.

1. Pour enregistrer vos modifications, sélectionnez **OK**.

   :::image type="content" source="./media/how-to-manage-database-account/add-region.png" alt-text="Ajouter ou supprimer le menu des régions":::

Dans un mode écriture dans une seule région, vous ne pouvez pas supprimer la région d’écriture. Vous devez basculer vers une autre région avant de pouvoir supprimer la région d’écriture actuelle.

Dans un mode d’écriture dans plusieurs régions, vous pouvez ajouter ou supprimer n’importe quelle région si vous en avez au moins une.

### <a name="azure-cli"></a><a id="add-remove-regions-via-cli"></a>Interface CLI Azure

Voir [Ajouter ou supprimer des régions avec Azure CLI](manage-with-cli.md#add-or-remove-regions)

### <a name="azure-powershell"></a><a id="add-remove-regions-via-ps"></a>Azure PowerShell

Veuillez consulter [Ajouter ou supprimer des régions avec PowerShell](manage-with-powershell.md#update-account)

## <a name="configure-multiple-write-regions"></a><a id="configure-multiple-write-regions"></a>Configurer plusieurs régions d’écriture

### <a name="azure-portal"></a><a id="configure-multiple-write-regions-portal"></a>Portail Azure

Ouvrez l’onglet **Répliquer les données globalement** et sélectionnez **Activer** pour activer l’écriture dans plusieurs régions. Une fois que vous avez activé l’écriture dans plusieurs régions, toutes les régions de lecture indiquées sur votre compte deviennent des régions de lecture et d’écriture.

:::image type="content" source="./media/how-to-manage-database-account/single-to-multi-master.png" alt-text="Capture d’écran de la configuration des écritures multirégions dans un compte Azure Cosmos":::

### <a name="azure-cli"></a><a id="configure-multiple-write-regions-cli"></a>Interface CLI Azure

Voir [Activer plusieurs régions d’écriture avec Azure CLI](manage-with-cli.md#enable-multiple-write-regions)

### <a name="azure-powershell"></a><a id="configure-multiple-write-regions-ps"></a>Azure PowerShell

Veuillez consulter [Activer plusieurs régions d’écriture avec PowerShell](manage-with-powershell.md#multi-region-writes)

### <a name="resource-manager-template"></a><a id="configure-multiple-write-regions-arm"></a>modèle Azure Resource Manager

Un compte peut passer de région d’écriture unique à plusieurs régions d’écriture en déployant le modèle Resource Manager utilisé pour créer le compte et en définissant `enableMultipleWriteLocations: true`. Le modèle Azure Resource Manager suivant est un modèle minimal qui va déployer un compte Azure Cosmos pour l’API SQL avec deux régions et plusieurs emplacements d’écriture activés.

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2015-01-01/deploymentTemplate.json#",
    "contentVersion": "1.0.0.0",
    "parameters": {
        "name": {
            "type": "String"
        },
        "location": {
            "type": "String",
            "defaultValue": "[resourceGroup().location]"
        },
        "primaryRegion":{
            "type":"string",
            "metadata": {
                "description": "The primary replica region for the Cosmos DB account."
            }
        },
        "secondaryRegion":{
            "type":"string",
            "metadata": {
              "description": "The secondary replica region for the Cosmos DB account."
          }
        }
    },
    "resources": [
        {
            "type": "Microsoft.DocumentDb/databaseAccounts",
            "kind": "GlobalDocumentDB",
            "name": "[parameters('name')]",
            "apiVersion": "2019-08-01",
            "location": "[parameters('location')]",
            "tags": {},
            "properties": {
                "databaseAccountOfferType": "Standard",
                "consistencyPolicy": { "defaultConsistencyLevel": "Session" },
                "locations":
                [
                    {
                        "locationName": "[parameters('primaryRegion')]",
                        "failoverPriority": 0,
                        "isZoneRedundant": false
                    },
                    {
                        "locationName": "[parameters('secondaryRegion')]",
                        "failoverPriority": 1,
                        "isZoneRedundant": false
                    }
                ],
                "enableMultipleWriteLocations": true
            }
        }
    ]
}
```

## <a name="enable-automatic-failover-for-your-azure-cosmos-account"></a><a id="automatic-failover"></a>Activer le basculement automatique pour votre compte Azure Cosmos

L’option de basculement automatique permet à Azure Cosmos DB de basculer vers la région ayant la priorité de basculement la plus élevée sans action de l’utilisateur au cas où une région devienne indisponible. Lorsque le basculement automatique est activé, la priorité des régions peut être modifiée. Le compte doit avoir au moins deux régions pour activer le basculement automatique.

### <a name="azure-portal"></a><a id="enable-automatic-failover-via-portal"></a>Portail Azure

1. Dans votre compte Azure Cosmos, ouvrez le volet **Répliquer les données globalement**.

2. En haut du volet, sélectionnez **Basculement automatique**.

   :::image type="content" source="./media/how-to-manage-database-account/replicate-data-globally.png" alt-text="Menu pour répliquer les données globalement":::

3. Dans le volet **Basculement automatique** , vérifiez que l’option **Activer le basculement automatique** est définie sur **ACTIVÉ**. 

4. Sélectionnez **Enregistrer**.

   :::image type="content" source="./media/how-to-manage-database-account/automatic-failover.png" alt-text="Menu du portail pour le basculement automatique":::

### <a name="azure-cli"></a><a id="enable-automatic-failover-via-cli"></a>Interface CLI Azure

Voir [Activer le basculement automatique avec Azure CLI](manage-with-cli.md#enable-automatic-failover)

### <a name="azure-powershell"></a><a id="enable-automatic-failover-via-ps"></a>Azure PowerShell

Veuillez consulter [Activer le basculement automatique avec PowerShell](manage-with-powershell.md#enable-automatic-failover)

## <a name="set-failover-priorities-for-your-azure-cosmos-account"></a>Définir les priorités de basculement pour votre compte Azure Cosmos

Une fois qu’un compte Cosmos est configuré pour le basculement automatique, la priorité de basculement des régions peut être modifiée.

> [!IMPORTANT]
> Vous ne pouvez pas modifier la région d’écriture (priorité de basculement de zéro) lorsque le compte est configuré pour le basculement automatique. Pour changer la région d’écriture, vous devez désactiver le basculement automatique et effectuer un basculement manuel.

### <a name="azure-portal"></a><a id="set-failover-priorities-via-portal"></a>Portail Azure

1. Dans votre compte Azure Cosmos, ouvrez le volet **Répliquer les données globalement**.

2. En haut du volet, sélectionnez **Basculement automatique**.

   :::image type="content" source="./media/how-to-manage-database-account/replicate-data-globally.png" alt-text="Menu pour répliquer les données globalement":::

3. Dans le volet **Basculement automatique** , vérifiez que l’option **Activer le basculement automatique** est définie sur **ACTIVÉ**.

4. Pour modifier la priorité de basculement, faites glisser les régions de lecture via les trois points à gauche de la ligne qui s’affichent quand vous placez le curseur dessus.

5. Sélectionnez **Enregistrer**.

   :::image type="content" source="./media/how-to-manage-database-account/automatic-failover.png" alt-text="Menu du portail pour le basculement automatique":::

### <a name="azure-cli"></a><a id="set-failover-priorities-via-cli"></a>Interface CLI Azure

Voir [Définir la priorité de basculement avec Azure CLI](manage-with-cli.md#set-failover-priority)

### <a name="azure-powershell"></a><a id="set-failover-priorities-via-ps"></a>Azure PowerShell

Veuillez consulter [Définir la priorité de basculement avec PowerShell](manage-with-powershell.md#modify-failover-priority)

## <a name="perform-manual-failover-on-an-azure-cosmos-account"></a><a id="manual-failover"></a>Effectuer un basculement manuel sur un compte Azure Cosmos

> [!IMPORTANT]
> Le compte Azure Cosmos doit être configuré pour le basculement manuel pour que l’opération fonctionne.

Le processus de basculement manuel implique de changer la région d’écriture du compte (priorité de basculement = 0) pour une autre région configurée pour le compte.

> [!NOTE]
> Les comptes dotés de plusieurs régions d’écriture ne peuvent pas être basculés manuellement. Pour les applications qui utilisent le SDK Azure Cosmos, celui-ci détecte le moment où une région devient indisponible, puis redirige automatiquement vers la région la plus proche si vous utilisez une API d’hébergement multiple dans le SDK.

### <a name="azure-portal"></a><a id="enable-manual-failover-via-portal"></a>Portail Azure

1. Accédez à votre compte Azure Cosmos, puis ouvrez le menu **Répliquer les données globalement**.

2. En haut du menu, sélectionnez **Basculement manuel**.

   :::image type="content" source="./media/how-to-manage-database-account/replicate-data-globally.png" alt-text="Menu pour répliquer les données globalement":::

3. Dans le menu **Basculement manuel** , sélectionnez votre nouvelle région d’écriture. Sélectionnez la case à cocher pour indiquer que vous comprenez que cette option modifie votre région d’écriture.

4. Pour déclencher le basculement, sélectionnez **OK**.

   :::image type="content" source="./media/how-to-manage-database-account/manual-failover.png" alt-text="Menu du portail pour le basculement manuel":::

### <a name="azure-cli"></a><a id="enable-manual-failover-via-cli"></a>Interface CLI Azure

Voir [Déclencher un basculement manuel avec Azure CLI](manage-with-cli.md#trigger-manual-failover)

### <a name="azure-powershell"></a><a id="enable-manual-failover-via-ps"></a>Azure PowerShell

Veuillez consulter [Déclencher un basculement manuel avec PowerShell](manage-with-powershell.md#trigger-manual-failover)

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations et d’exemples de gestion du compte Azure Cosmos ainsi que des bases de données et conteneurs, consultez les articles suivants :

* [Gérer Azure Cosmos DB à l’aide d’Azure PowerShell](manage-with-powershell.md)
* [Gérer Azure Cosmos DB à l’aide d’Azure CLI](manage-with-cli.md)