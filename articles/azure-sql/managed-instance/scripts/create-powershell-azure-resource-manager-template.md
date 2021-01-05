---
title: Créer une instance managée (modèle Resource Manager et PowerShell)
titleSuffix: Azure SQL Managed Instance
description: Utilisez cet exemple de script Azure PowerShell pour créer une instance managée.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: seo-dt-2019, devx-track-azurepowershell
ms.devlang: PowerShell
ms.topic: sample
author: jovanpop-msft
ms.author: jovanpop
ms.reviewer: sstein
ms.date: 03/12/2019
ms.openlocfilehash: 7c364fe7fa0ac4dd70a01a75478289ea861dee7f
ms.sourcegitcommit: 400f473e8aa6301539179d4b320ffbe7dfae42fe
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92790829"
---
# <a name="use-powershell-with-an-azure-resource-manager-template-to-create-a-managed-instance"></a>Utiliser PowerShell avec un modèle Azure Resource Manager pour créer une instance managée

[!INCLUDE[appliesto-sqldb](../../includes/appliesto-sqlmi.md)]

Vous pouvez créer une instance managée à l’aide de la bibliothèque Azure PowerShell et des modèles Azure Resource Manager.

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]
[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]
[!INCLUDE [cloud-shell-try-it.md](../../../../includes/cloud-shell-try-it.md)]

Si vous choisissez d’installer et d’utiliser PowerShell localement, ce tutoriel nécessite Azure PowerShell 1.4.0 ou ultérieur. Si vous devez effectuer une mise à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps). Si vous exécutez PowerShell en local, exécutez `Connect-AzAccount` pour créer une connexion à Azure.

Les commandes Azure PowerShell peuvent démarrer le déploiement à l’aide d’un modèle Azure Resource Manager prédéfini. Vous pouvez spécifier les propriétés suivantes dans le modèle :

- Nom de l’instance managée
- Le nom d’utilisateur et le mot de passe administrateur SQL
- La taille de l’instance (nombre de cœurs et taille de stockage maximale)
- Le réseau virtuel et le sous-réseau où l’instance doit être placée
- Le classement de l’instance au niveau du serveur (en préversion).

Les éléments que vous ne pouvez pas changer ultérieurement sont le nom de l’instance, le nom d’utilisateur de l’administrateur SQL, le réseau virtuel et le sous-réseau. Les autres propriétés d’instance peuvent être modifiées.

## <a name="prerequisites"></a>Prérequis

Cet exemple suppose que vous avez [créé un environnement réseau valide](../virtual-network-subnet-create-arm-template.md) ou [modifié un réseau virtuel existant](../vnet-existing-add-subnet.md) pour votre instance managée. Vous pouvez préparer l’environnement réseau à l’aide d’un [modèle Azure Resource Manager](https://github.com/Azure/azure-quickstart-templates/tree/master/101-sql-managed-instance-azure-environment) distinct, si nécessaire. 


L’exemple utilise les applets de commande [New-AzResourceGroupDeployment](/powershell/module/az.resources/new-azresourcegroupdeployment) et [Get-AzVirtualNetwork](/powershell/module/az.network/get-azvirtualnetwork). Vous devez donc vérifier que vous avez installé les modules PowerShell suivants :

```powershell
Install-Module Az.Network
Install-Module Az.Resources
```

## <a name="azure-resource-manager-template"></a>Modèle Azure Resource Manager


Enregistrez le script suivant dans un fichier .json et notez l’emplacement du fichier : 

```json
{
    "$schema": "https://schema.management.azure.com/schemas/2014-04-01-preview/deploymentTemplate.json#",
    "contentVersion": "1.0.0.1",
    "parameters": {
        "instance": {
            "type": "string"
        },
        "user": {
            "type": "string"
        },
        "pwd": {
            "type": "securestring"
        },
        "subnetId": {
            "type": "string"
        }
    },
    "resources": [
        {
            "name": "[parameters('instance')]",
            "location": "West Central US",
            "tags": {
                "Description":"GP Instance with custom instance collation - Serbian_Cyrillic_100_CS_AS"
            },
            "sku": {
                "name": "GP_Gen5",
                "tier": "GeneralPurpose"
            },
            "properties": {
                "administratorLogin": "[parameters('user')]",
                "administratorLoginPassword": "[parameters('pwd')]",
                "subnetId": "[parameters('subnetId')]",
                "storageSizeInGB": 256,
                "vCores": 8,
                "licenseType": "LicenseIncluded",
                "hardwareFamily": "Gen5",
                "collation": "Serbian_Cyrillic_100_CS_AS"
            },
            "type": "Microsoft.Sql/managedInstances",
            "identity": {
                "type": "SystemAssigned"
            },
            "apiVersion": "2015-05-01-preview"
        }
    ]
}
```

Mettez à jour le script PowerShell suivant avec le chemin du fichier .json que vous avez enregistré, puis changez les noms des objets dans le script :

```powershell
$subscriptionId = "ed827499-xxxx-xxxx-xxxx-xxxxxxxxxx"
Select-AzSubscription -SubscriptionId $subscriptionId

# Managed instance properties
$resourceGroup = "rg_mi"
$location = "West Central US"
$name = "managed-instance-name"
$user = "miSqlAdmin"
$secpasswd = ConvertTo-SecureString "<Put some strong password here>" -AsPlainText -Force

# Network configuration
$vNetName = "my_vnet"
$vNetResourceGroup = "rg_mi_vnet"
$subnetName = "ManagedInstances"
$vNet = Get-AzVirtualNetwork -Name $vNetName -ResourceGroupName $vNetResourceGroup
$subnet = Get-AzVirtualNetworkSubnetConfig -Name $SubnetName -VirtualNetwork $vNet
$subnetId = $subnet.Id

# Deploy instance using Azure Resource Manager template:
New-AzResourceGroupDeployment  -Name MyDeployment -ResourceGroupName $resourceGroup  `
                                    -TemplateFile 'C:\...\create-managed-instance.json' `
                                    -instance $name -user $user -pwd $secpasswd -subnetId $subnetId
```

Une fois le script terminé, l’instance managée est accessible à partir de tous les services Azure et de l’adresse IP configurée.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur Azure PowerShell, consultez la [documentation Azure PowerShell](/powershell/azure/).

Vous trouverez d’autres exemples de scripts PowerShell pour Azure SQL Managed Instance dans [Scripts PowerShell Azure SQL Managed Instance](../../database/powershell-script-content-guide.md).