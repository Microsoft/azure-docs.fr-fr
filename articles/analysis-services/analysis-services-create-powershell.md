---
title: 'Démarrage rapide : Créer des services Azure Analysis Services avec PowerShell Azure Analysis Services | Microsoft Docs'
description: Ce guide de démarrage rapide explique comment créer un serveur Azure Analysis Services à l’aide de PowerShell.
author: minewiskan
ms.service: azure-analysis-services
ms.topic: quickstart
ms.date: 08/31/2020
ms.author: owend
ms.reviewer: minewiskan
ms.custom: references_regions , devx-track-azurepowershell
ms.openlocfilehash: 737649538aaf82352e27aec6220b13ba355a7a82
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "89229325"
---
# <a name="quickstart-create-a-server---powershell"></a>Démarrage rapide : créer un serveur - PowerShell

Ce démarrage rapide décrit comment utiliser PowerShell à partir d’une ligne de commande pour créer un serveur Azure Analysis Services de votre abonnement Azure.

## <a name="prerequisites"></a>Conditions préalables requises

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

- **Abonnement Azure** : visitez [version d’évaluation gratuite d’Azure](https://azure.microsoft.com/offers/ms-azr-0044p/) pour créer un compte.
- **Azure Active Directory** : votre abonnement doit être associé à un client Azure Active Directory et vous devez disposer d’un compte dans ce répertoire. Pour en savoir plus, consultez [Authentification et autorisations utilisateur](analysis-services-manage-users.md).
- **Azure PowerShell**. Pour trouver la version installée, exécutez `Get-Module -ListAvailable Az`. Pour installer ou mettre à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-Az-ps).

## <a name="import-azanalysisservices-module"></a>Importer le module Az.AnalysisServices

Pour créer un serveur dans votre abonnement, vous utilisez le module [Az.AnalysisServices](/powershell/module/az.analysisservices). Chargez le module Az.AnalysisServices dans votre session PowerShell.

```powershell
Import-Module Az.AnalysisServices
```

## <a name="sign-in-to-azure"></a>Connexion à Azure

Connectez-vous à votre abonnement Azure à l’aide de la commande [Connect-AzAccount](/powershell/module/az.accounts/connect-azaccount). Suivez les instructions à l’écran.

```powershell
Connect-AzAccount
```

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Un [groupe de ressources Azure](../azure-resource-manager/management/overview.md) est un conteneur logique dans lequel les ressources Azure sont déployées et gérées en tant que groupe. Lorsque vous créez votre serveur, vous devez spécifier un groupe de ressources dans votre abonnement. Si vous ne disposez pas d’un groupe de ressources, créez-en un à l’aide de la commande [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). L’exemple suivant crée un groupe de ressources nommé `myResourceGroup` dans la région USA Ouest.

```powershell
New-AzResourceGroup -Name "myResourceGroup" -Location "WestUS"
```

## <a name="create-a-server"></a>Créer un serveur

Créez un serveur à l’aide de la commande [New-AzAnalysisServicesServer](/powershell/module/az.analysisservices/new-azanalysisservicesserver). L’exemple suivant crée un serveur nommé myServer dans myResourceGroup, dans la région États-Unis de l’Ouest, au niveau D1, et spécifie philipc@adventureworks.com en tant qu’administrateur de serveur.

```powershell
New-AzAnalysisServicesServer -ResourceGroupName "myResourceGroup" -Name "myserver" -Location WestUS -Sku D1 -Administrator "philipc@adventure-works.com"
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Vous pouvez supprimer le serveur de votre abonnement à l’aide de la commande [Remove-AzAnalysisServicesServer](/powershell/module/az.analysisservices/new-azanalysisservicesserver). Si vous continuez avec d’autres démarrages rapides et tutoriels de cette collection, ne supprimez pas votre serveur. L’exemple suivant supprime le serveur créé à l’étape précédente.


```powershell
Remove-AzAnalysisServicesServer -Name "myserver" -ResourceGroupName "myResourceGroup"
```

## <a name="next-steps"></a>Étapes suivantes

Dans ce démarrage rapide, vous avez appris à créer un serveur dans votre abonnement Azure à l’aide de PowerShell. Maintenant que vous avez serveur, vous pouvez le sécuriser en configurant un pare-feu de serveur (facultatif). Vous pouvez également ajouter un modèle de données d’exemple de base à votre serveur directement à partir du portail. Un exemple de modèle permet d’en savoir plus sur la configuration des rôles de base de données de modèle et le test des connexions client. Pour en savoir plus, passez au tutoriel sur l’ajout d’un exemple de modèle.

> [!div class="nextstepaction"]
> [Démarrage rapide : Configurer un pare-feu de serveur - Portail](analysis-services-qs-firewall.md)      