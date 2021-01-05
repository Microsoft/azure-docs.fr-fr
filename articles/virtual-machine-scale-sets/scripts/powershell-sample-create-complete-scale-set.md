---
title: Exemples Azure PowerShell - Créer un groupe de machines virtuelles identiques complet
description: Ce script crée un groupe de machines virtuelles identiques exécutant Windows Server 2016, dans lequel des ressources sont configurées et créées.
author: mimckitt
ms.author: mimckitt
ms.topic: sample
ms.service: virtual-machine-scale-sets
ms.subservice: powershell
ms.date: 05/29/2018
ms.reviewer: jushiman
ms.custom: mimckitt, devx-track-azurepowershell
ms.openlocfilehash: d37f5f624459db6bc336884987a16c60503492a8
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89078501"
---
# <a name="create-a-complete-virtual-machine-scale-set-with-powershell"></a>Créer un groupe de machines virtuelles identiques complet à l’aide de PowerShell

Ce script crée un groupe de machines virtuelles identiques exécutant Windows Server 2016. Les ressources individuelles sont configurées et créées, plutôt que d’utiliser les [options de création de ressources intégrées disponibles ici dans New-AzVmss](powershell-sample-create-simple-scale-set.md). Une fois que vous avez exécuté le script, vous pouvez accéder aux instances de la machine virtuelle via RDP.


[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [updated-for-az.md](../../../includes/updated-for-az.md)]

## <a name="sample-script"></a>Exemple de script

[!code-powershell[main](../../../powershell_scripts/virtual-machine-scale-sets/complete-scale-set/complete-scale-set.ps1 "Create a complete virtual machine scale set")]

## <a name="clean-up-deployment"></a>Nettoyer le déploiement
Exécutez la commande suivante pour supprimer le groupe de ressources, le groupe identique et toutes les ressources associées.

```powershell
Remove-AzResourceGroup -Name $resourceGroupName
```

## <a name="script-explanation"></a>Explication du script
Ce script a recours aux commandes suivantes pour créer le déploiement. Chaque élément du tableau renvoie à une documentation spécifique.

| Commande | Notes |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Crée un groupe de ressources dans lequel toutes les ressources sont stockées. |
| [New-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/new-azvirtualnetworksubnetconfig) | Crée une configuration de sous-réseau. Cette configuration est utilisée avec le processus de création du réseau virtuel. |
| [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) | Créer un réseau virtuel. |
| [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) | Crée une adresse IP publique. |
| [New-AzLoadBalancerFrontendIpConfig](/powershell/module/az.network/new-azloadbalancerfrontendipconfig) | Crée une configuration IP frontale pour un équilibreur de charge. |
| [New-AzLoadBalancerBackendAddressPoolConfig](/powershell/module/az.network/new-azloadbalancerbackendaddresspoolconfig) | Crée une configuration de pool d’adresses principales pour un équilibreur de charge. |
| [New-AzLoadBalancerInboundNatRuleConfig](/powershell/module/az.network/new-azloadbalancerinboundnatruleconfig) | Crée une configuration de règle NAT entrante pour un équilibreur de charge. |
| [New-AzLoadBalancer](/powershell/module/az.network/new-azloadbalancer) | Crée un équilibreur de charge. |
| [Add-AzLoadBalancerProbeConfig](/powershell/module/az.network/new-azloadbalancerprobeconfig) | Crée une configuration de sonde pour un équilibreur de charge. |
| [Add-AzLoadBalancerRuleConfig](/powershell/module/az.network/new-azloadbalancerruleconfig) | Crée une configuration de règle pour un équilibreur de charge. |
| [Set-AzLoadBalancer](/powershell/module/az.Network/Set-azLoadBalancer) | Mettez à jour l’équillibreur de charge avec les informations fournies. |
| [New-AzVmssIpConfig](/powershell/module/az.Compute/New-azVmssIpConfig) | Créez une configuration d’adresse IP pour les instances de machine virtuelle du groupe identique. Les instances de machines virtuelles sont connectées pour le pool principal d’équilibreur de charge, au pool NAT et au sous-réseau de réseau virtuel. |
| [New-AzVmssConfig](/powershell/module/az.Compute/New-azVmssConfig) | Créez une configuration de groupe identique. Cette configuration inclut des informations telles que le nombre d’instances de machine virtuelle à créer, la référence SKU de la machine virtuelle (taille) et le mode de stratégie de mise à niveau. La configuration est ajoutée par d’autres applets de commande et utilisée lors de la création de groupe identique. |
| [Set-AzVmssStorageProfile](/powershell/module/az.Compute/Set-azVmssStorageProfile) | Définissez l’image à utiliser pour les instances de machine virtuelle et ajouter-la à la configuration de groupe identique. |
| [Set-AzVmssOsProfile](/powershell/module/az.Compute/Set-azVmssStorageProfile) | Définissez les informations d’identification d’administration (nom d’utilisateur et de mot de passe) et préfixe d’affectation de nom de machine virtuelle. Ajoutez ces valeurs à la configuration de groupe identique. |
| [Add-AzVmssNetworkInterfaceConfiguration](/powershell/module/az.Compute/Add-azVmssNetworkInterfaceConfiguration) | Ajoutez une interface de réseau virtuel aux instances de machine virtuelle, en fonction de la configuration d’adresse IP. Ajoutez ces valeurs à la configuration de groupe identique. |
| [New-AzVmss](/powershell/module/az.Compute/New-azVmss) | Créez le groupe identique selon les informations fournies dans la configuration correspondante. |
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Supprime un groupe de ressources et toutes les ressources contenues. |

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur le module Azure PowerShell, consultez [Documentation Azure PowerShell](/powershell/azure/).
