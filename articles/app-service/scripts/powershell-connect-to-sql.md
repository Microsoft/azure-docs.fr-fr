---
title: 'PowerShell : Connexion à la base de données SQL'
description: Découvrez comment utiliser Azure PowerShell pour automatiser le déploiement et la gestion d’App Service. Cet exemple montre comment connecter une application à une instance SQL Database.
tags: azure-service-management
ms.assetid: 055440a9-fff1-49b2-b964-9c95b364e533
ms.topic: sample
ms.date: 03/20/2017
ms.custom: mvc, devx-track-azurepowershell
ms.openlocfilehash: 3b355a830703b51264d2f4b819d5be7f2627b4e2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89077787"
---
# <a name="connect-an-app-service-app-to-sql-database"></a>Connecter une application App Service à SQL Database

Dans ce scénario, vous allez apprendre à créer une base de données dans Azure SQL Database et une application App Service. Ensuite, vous allez lier la base de données à l’application à l’aide de paramètres d’application.

Si nécessaire, installez Azure PowerShell à l’aide des instructions figurant dans le [Guide Azure PowerShell](/powershell/azure/), puis exécutez `Connect-AzAccount` pour créer une connexion avec Azure.

## <a name="sample-script"></a>Exemple de script

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

[!code-azurepowershell-interactive[main](../../../powershell_scripts/app-service/connect-to-sql/connect-to-sql.ps1?highlight=13 "Connect an app to SQL Database")]

## <a name="clean-up-deployment"></a>Nettoyer le déploiement 

Une fois l’exemple de script exécuté, vous pouvez utiliser la commande suivante pour supprimer le groupe de ressources, l’application App Service et toutes les ressources associées.

```powershell
Remove-AzResourceGroup -Name myResourceGroup -Force
```

## <a name="script-explanation"></a>Explication du script

Ce script utilise les commandes suivantes. Chaque commande du tableau renvoie à une documentation spécifique.

| Commande | Notes |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Crée un groupe de ressources dans lequel toutes les ressources sont stockées. |
| [New-AzAppServicePlan](/powershell/module/az.websites/new-azappserviceplan) | Crée un plan App Service. |
| [New-AzWebApp](/powershell/module/az.websites/new-azwebapp) | Crée une application App Service. |
| [New-AzSQLServer](/powershell/module/az.sql/new-azsqlserver) | Crée un serveur. |
| [New-AzSqlServerFirewallRule](/powershell/module/az.sql/new-azsqlserverfirewallrule) | Crée une règle de pare-feu au niveau du serveur. |
| [New-AzSQLDatabase](/powershell/module/az.sql/new-azsqldatabase) | Crée une base de données ou une base de données élastique. |
| [Set-AzWebApp](/powershell/module/az.websites/set-azwebapp) | Modifie la configuration d’une application App Service. |

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur le module Azure PowerShell, consultez [Documentation Azure PowerShell](/powershell/azure/).

Vous trouverez des exemples supplémentaires de scripts Azure PowerShell pour Azure App Service sur la page [Exemples Azure PowerShell](../samples-powershell.md).
