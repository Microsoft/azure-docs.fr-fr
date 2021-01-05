---
title: 'PowerShell : Créer une instance gérée'
titleSuffix: Azure SQL Managed Instance
description: Cet article fournit un exemple de script Azure PowerShell permettant de créer une instance managée.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: operations
ms.custom: ''
ms.devlang: PowerShell
ms.topic: sample
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 03/25/2019
ms.openlocfilehash: fd091cedb67c07b7de2c7e8540e99c3e8daf7dcf
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91323717"
---
# <a name="use-powershell-to-create-a-managed-instance"></a>Utiliser PowerShell pour créer une instance managée

[!INCLUDE[appliesto-sqldb](../../includes/appliesto-sqlmi.md)]

Cet exemple de script PowerShell crée une instance managée dans un sous-réseau dédié au sein d’un réseau virtuel. Il configure également une table de routage et un groupe de sécurité réseau pour le réseau virtuel. Une fois le script correctement exécuté, l’instance managée est accessible à partir du réseau virtuel ou d’un environnement local. Consultez [Configurer la machine virtuelle Azure pour qu’elle se connecte à Azure SQL Database Managed Instance](../connect-vm-instance-configure.md) et [Configurer une connexion point à site à Azure SQL Managed Instance à partir d’un emplacement local](../point-to-site-p2s-configure.md).

> [!IMPORTANT]
> Pour connaître les limitations, consultez les [régions prises en charge](../resource-limits.md#supported-regions) et les [types d’abonnement pris en charge](../resource-limits.md#supported-subscription-types).

[!INCLUDE [cloud-shell-try-it.md](../../../../includes/cloud-shell-try-it.md)]

Si vous choisissez d’installer et d’utiliser PowerShell localement, ce tutoriel nécessite Azure PowerShell 1.4.0 ou ultérieur. Si vous devez effectuer une mise à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps). Si vous exécutez PowerShell en local, vous devez également lancer `Connect-AzAccount` pour créer une connexion avec Azure.

## <a name="sample-script"></a>Exemple de script

[!code-powershell-interactive[main](../../../../powershell_scripts/sql-database/managed-instance/create-and-configure-managed-instance.ps1 "Create managed instance")]

## <a name="clean-up-deployment"></a>Nettoyer le déploiement

Utilisez la commande suivante pour supprimer le groupe de ressources et toutes les ressources associées.

```powershell
Remove-AzResourceGroup -ResourceGroupName $resourcegroupname
```

## <a name="script-explanation"></a>Explication du script

Ce script utilise certaines des commandes suivantes. Pour plus d’informations sur les commandes utilisées et les autres commandes indiquées dans le tableau ci-dessous, cliquez sur les liens afin d’accéder à la documentation correspondante.

| Commande | Notes |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Crée un groupe de ressources dans lequel toutes les ressources sont stockées.
| [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) | Créer un réseau virtuel. |
| [Add-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/Add-AzVirtualNetworkSubnetConfig) | Ajoute une configuration de sous-réseau à un réseau virtuel. |
| [Get-AzVirtualNetwork](/powershell/module/az.network/Get-AzVirtualNetwork) | Obtient un réseau virtuel dans un groupe de ressources. |
| [Set-AzVirtualNetwork](/powershell/module/az.network/Set-AzVirtualNetwork) | Définit l’état visé d’un réseau virtuel. |
| [Get-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/Get-AzVirtualNetworkSubnetConfig) | Obtient un sous-réseau dans un réseau virtuel. |
| [Set-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/Set-AzVirtualNetworkSubnetConfig) | Configure l’état visé d’une configuration de sous-réseau dans un réseau virtuel. |
| [New-AzRouteTable](/powershell/module/az.network/New-AzRouteTable) | Crée une table de routage. |
| [Get-AzRouteTable](/powershell/module/az.network/Get-AzRouteTable) | Obtient les tables de routage. |
| [Set-AzRouteTable](/powershell/module/az.network/Set-AzRouteTable) | Définit l’état visé d’une table de routage. |
| [New-AzSqlInstance](/powershell/module/az.sql/New-AzSqlInstance) | Crée une instance managée. |
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Supprime un groupe de ressources, y compris toutes les ressources imbriquées. |
|||

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur Azure PowerShell, consultez la [documentation Azure PowerShell](/powershell/azure/).

Vous trouverez d’autres exemples de scripts PowerShell pour Azure SQL Managed Instance dans [Scripts PowerShell Azure SQL Managed Instance](../../database/powershell-script-content-guide.md).
