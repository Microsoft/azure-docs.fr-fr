---
title: 'PowerShell : Ajouter une instance managée à un groupe de basculement automatique'
titleSuffix: Azure SQL Managed Instance
description: Exemple de script Azure PowerShell pour créer une instance managée, l’ajouter à un groupe de basculement automatique et tester le basculement.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: high-availability
ms.custom: sqldbrb=1
ms.devlang: PowerShell
ms.topic: sample
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 07/16/2019
ms.openlocfilehash: af1d7110cccfd8b0617d6c79eb5373cc7c087159
ms.sourcegitcommit: 1cf157f9a57850739adef72219e79d76ed89e264
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2020
ms.locfileid: "94594009"
---
# <a name="use-powershell-to-add-a-managed-instance-to-a-failover-group"></a>Utiliser PowerShell pour ajouter une instance managée à un groupe de basculement 

[!INCLUDE[appliesto-sqldb](../../includes/appliesto-sqlmi.md)]

Cet exemple de script PowerShell crée deux instances gérées, les ajoute à un groupe de basculement, puis teste le basculement de l’instance gérée principale vers l’instance gérée secondaire. 

[!INCLUDE [quickstarts-free-trial-note](../../../../includes/quickstarts-free-trial-note.md)]
[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]
[!INCLUDE [cloud-shell-try-it.md](../../../../includes/cloud-shell-try-it.md)]

Si vous choisissez d’installer et d’utiliser PowerShell localement, ce tutoriel nécessite Azure PowerShell 1.4.0 ou ultérieur. Si vous devez effectuer une mise à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps). Si vous exécutez PowerShell en local, vous devez également lancer `Connect-AzAccount` pour créer une connexion avec Azure.

## <a name="sample-scripts"></a>Exemples de scripts

[!code-powershell-interactive[main](../../../../powershell_scripts/sql-database/failover-groups/add-managed-instance-to-failover-group-az-ps.ps1 "Add managed instance to a failover group")]

## <a name="clean-up-deployment"></a>Nettoyer le déploiement

Utilisez la commande suivante pour supprimer le groupe de ressources et toutes les ressources associées. Vous devez supprimer le groupe de ressources à deux reprises. La première suppression du groupe de ressources entraîne la suppression de l’instance gérée et des clusters virtuels, mais échouera avec le message d’erreur `Remove-AzResourceGroup : Long running operation failed with status 'Conflict'`. Exécutez la commande Remove-AzResourceGroup une deuxième fois pour supprimer toutes les ressources résiduelles, ainsi que le groupe de ressources.

```powershell
Remove-AzResourceGroup -ResourceGroupName $resourceGroupName
```

## <a name="script-explanation"></a>Explication du script

Ce script utilise les commandes suivantes. Chaque commande du tableau renvoie à une documentation spécifique.

| Commande | Notes |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Crée un groupe de ressources Azure.  |
| [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) | Créer un réseau virtuel.  |
| [Add-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/add-azvirtualnetworksubnetconfig) | Ajoute une configuration de sous-réseau à un réseau virtuel. | 
| [Get-AzVirtualNetwork](/powershell/module/az.network/get-azvirtualnetwork) | Obtient un réseau virtuel dans un groupe de ressources. | 
| [Get-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/get-azvirtualnetworksubnetconfig) | Obtient un sous-réseau dans un réseau virtuel. | 
| [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup) | Crée un groupe de sécurité réseau. | 
| [New-AzRouteTable](/powershell/module/az.network/new-azroutetable) | Crée une table de routage. |
| [Set-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/set-azvirtualnetworksubnetconfig) | Met à jour une configuration de sous-réseau pour un réseau virtuel.  |
| [Set-AzVirtualNetwork](/powershell/module/az.network/set-azvirtualnetwork) | Met à jour un réseau virtuel.  |
| [Get-AzNetworkSecurityGroup](/powershell/module/az.network/get-aznetworksecuritygroup) | Obtient un groupe de sécurité réseau. |
| [Add-AzNetworkSecurityRuleConfig](/powershell/module/az.network/add-aznetworksecurityruleconfig)| Ajoute une configuration de règle de sécurité réseau à un groupe de sécurité réseau. |
| [Set-AzNetworkSecurityGroup](/powershell/module/az.network/set-aznetworksecuritygroup) | Met à jour un groupe de sécurité réseau.  | 
| [Add-AzRouteConfig](/powershell/module/az.network/add-azrouteconfig) | Ajoute un itinéraire à une table de routage. |
| [Set-AzRouteTable](/powershell/module/az.network/set-azroutetable) | Met à jour une table de routage.  |
| [New-AzSqlInstance](/powershell/module/az.sql/new-azsqlinstance) | Crée une instance managée.  |
| [Get-AzSqlInstance](/powershell/module/az.sql/get-azsqlinstance)| Retourne des informations sur Azure SQL Managed Instance. |
| [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) | Crée une adresse IP publique.  | 
| [New-AzVirtualNetworkGatewayIpConfig](/powershell/module/az.network/new-azvirtualnetworkgatewayipconfig) | Crée une configuration IP pour une passerelle de réseau virtuel |
| [New-AzVirtualNetworkGateway](/powershell/module/az.network/new-azvirtualnetworkgateway) | Crée une passerelle de réseau virtuel |
| [New-AzVirtualNetworkGatewayConnection](/powershell/module/az.network/new-azvirtualnetworkgatewayconnection) | Crée une connexion entre les deux passerelles de réseau virtuel.   |
| [New-AzSqlDatabaseInstanceFailoverGroup](/powershell/module/az.sql/new-azsqldatabaseinstancefailovergroup)| Crée un groupe de basculement Azure SQL Managed Instance.  |
| [Get-AzSqlDatabaseInstanceFailoverGroup](/powershell/module/az.sql/get-azsqldatabaseinstancefailovergroup) | Obtient ou liste des groupes de basculement SQL Managed Instance.| 
| [Switch-AzSqlDatabaseInstanceFailoverGroup](/powershell/module/az.sql/switch-azsqldatabaseinstancefailovergroup) | Exécute un basculement d’un groupe de basculement SQL Managed Instance. | 
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Supprime un groupe de ressources. | 

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur Azure PowerShell, consultez la [documentation Azure PowerShell](/powershell/azure/).

Vous trouverez d’autres exemples de scripts PowerShell pour SQL Managed Instance dans [Scripts PowerShell Azure SQL Managed Instance](../../database/powershell-script-content-guide.md).
