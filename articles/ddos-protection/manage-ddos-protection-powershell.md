---
title: Créer et configurer un plan Azure DDoS Protection à l’aide d’Azure PowerShell
description: Découvrez comment créer un plan de protection DDoS à l’aide d’Azure PowerShell
services: ddos-protection
documentationcenter: na
author: yitoh
ms.service: ddos-protection
ms.devlang: na
ms.topic: article
ms.tgt_pltfrm: na
ms.workload: infrastructure-services
ms.date: 09/28/2020
ms.author: yitoh
ms.openlocfilehash: 69f9b5a74566879ecf8f15f23e689ebb731da45a
ms.sourcegitcommit: 1140ff2b0424633e6e10797f6654359947038b8d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/30/2020
ms.locfileid: "97814140"
---
# <a name="quickstart-create-and-configure-azure-ddos-protection-standard-using-azure-powershell"></a>Démarrage rapide : Créer et configurer Azure DDoS Protection Standard à l’aide d’Azure PowerShell

Prise en main d’Azure DDoS Protection Standard à l’aide d’Azure PowerShell. 

Un plan de protection DDoS définit un ensemble de réseaux virtuels pour lesquels la protection DDoS standard est activée par le biais des abonnements. Vous pouvez configurer un plan de protection DDoS pour votre organisation et lier des réseaux virtuels à partir de plusieurs abonnements au même plan. 

Dans ce démarrage rapide, vous allez créer un plan de protection DDoS et le lier à un réseau virtuel. 

## <a name="prerequisites"></a>Prérequis

- Compte Azure avec un abonnement actif. [Créez un compte gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F).
- Azure PowerShell installé localement ou Azure Cloud Shell

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="create-a-ddos-protection-plan"></a>Créer un plan de protection DDoS

Dans Azure, vous allouez les ressources associées à un groupe de ressources. Vous pouvez utiliser un groupe de ressources existant ou en créer un.

Pour créer un groupe de ressources, utilisez [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Dans cet exemple, nous allons nommer notre groupe de ressources _MyResourceGroup_ et utiliser la région _USA Est_ :

```azurepowershell-interactive
New-AzResourceGroup -Name MyResourceGroup -Location "East US"
```

Créez maintenant un plan de protection DDoS nommé _MyDdosProtectionPlan_ :

```azurepowershell-interactive
New-AzDdosProtectionPlan -ResourceGroupName MyResourceGroup -Name MyDdosProtectionPlan -Location "East US"
```

## <a name="enable-ddos-for-a-virtual-network"></a>Activer la protection DDoS pour un réseau virtuel

### <a name="enable-ddos-for-a-new-virtual-network"></a>Activer la protection DDoS pour un nouveau réseau virtuel

Vous pouvez activer la protection DDoS lors de la création d’un réseau virtuel. Dans cet exemple, nous allons nommer notre réseau virtuel _MyVnet_ : 

```azurepowershell-interactive
New-AzVirtualNetwork -Name MyVnet -ResourceGroupName MyResourceGroup -Location "East US" -AddressPrefix 10.0.0.0/16
```

### <a name="enable-ddos-for-an-existing-virtual-network"></a>Activer la protection DDoS pour un réseau virtuel existant

Vous pouvez associer un réseau virtuel existant lors de la création d’un plan de protection DDoS :

```azurepowershell-interactive
# Creates the DDoS protection plan
$ddosProtectionPlan = New-AzDdosProtectionPlan -ResourceGroupName MyResourceGroup -Name MyDdosProtectionPlan -Location "East US"

# Gets the most updated version of the virtual network
$vnet = Get-AzVirtualNetwork -Name MyVnet -ResourceGroupName MyResourceGroup
$vnet.DdosProtectionPlan = New-Object Microsoft.Azure.Commands.Network.Models.PSResourceId

# Update the properties and enable DDoS protection
$vnet.DdosProtectionPlan.Id = $ddosProtectionPlan.Id
$vnet.EnableDdosProtection = $true
$vnet | Set-AzVirtualNetwork
``` 

## <a name="validate-and-test"></a>Valider et tester

Tout d’abord, vérifiez les détails de votre plan de protection DDoS :

```azurepowershell-interactive
Get-AzDdosProtectionPlan -ResourceGroupName MyResourceGroup -Name MyDdosProtectionPlan
```

Vérifiez que la commande retourne les détails corrects de votre plan de protection DDoS.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Vous pouvez conserver vos ressources pour le tutoriel suivant. Si vous n’en avez plus besoin, supprimez le groupe de ressources _MyResourceGroup_. Lorsque vous supprimez le groupe de ressources, vous supprimez aussi le plan de protection DDoS et toutes ses ressources associées. 

```azurepowershell-interactive
Remove-AzResourceGroup -Name MyResourceGroup
```

Pour désactiver la protection DDoS pour un réseau virtuel : 

```azurepowershell-interactive
# Gets the most updated version of the virtual network
$vnet = Get-AzVirtualNetwork -Name MyVnet -ResourceGroupName MyResourceGroup
$vnet.DdosProtectionPlan = $null
$vnet.EnableDdosProtection = $false
$vnet | Set-AzVirtualNetwork
```

Si vous souhaitez supprimer un plan de protection DDoS, vous devez d’abord dissocier tous les réseaux virtuels de celui-ci.

## <a name="next-steps"></a>Étapes suivantes

Pour savoir comment consulter et configurer la télémétrie pour votre plan de protection DDoS, passez aux tutoriels.

> [!div class="nextstepaction"]
> [Consulter et configurer la télémétrie de la protection DDoS](telemetry.md)