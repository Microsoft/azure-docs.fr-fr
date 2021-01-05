---
title: Exemples Azure PowerShell - Activer la mise à l’échelle basée sur l’hôte
description: Ce script crée un groupe de machines virtuelles identiques exécutant Windows Server 2016 et utilise des métriques basées sur l’hôte pour mettre à l’échelle automatiquement tandis que la charge du processeur change.
author: ju-shim
ms.author: jushiman
ms.topic: sample
ms.service: virtual-machine-scale-sets
ms.subservice: autoscale
ms.date: 06/25/2020
ms.custom: avverma, devx-track-azurepowershell
ms.openlocfilehash: 9530f34ee919547049df06fb0974971c3ba4b2a9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89079623"
---
# <a name="automatically-scale-a-virtual-machine-scale-set-with-powershell"></a>Mettre à l’échelle automatiquement un groupe de machines virtuelles identiques avec PowerShell
Ce script crée un groupe de machines virtuelles identiques exécutant Windows Server 2016 et utilise des métriques basées sur l’hôte pour mettre à l’échelle automatiquement tandis que la charge du processeur change.

[!INCLUDE [quickstarts-free-trial-note](../../../includes/quickstarts-free-trial-note.md)]

[!INCLUDE [updated-for-az.md](../../../includes/updated-for-az.md)]

## <a name="sample-script"></a>Exemple de script


[!code-powershell[main](../../../powershell_scripts/virtual-machine-scale-sets/auto-scale-host-metrics/auto-scale-host-metrics.ps1 "Automatically scale a virtual machine scale set")]

## <a name="clean-up-deployment"></a>Nettoyer le déploiement
Exécutez la commande suivante pour supprimer le groupe de ressources, le groupe identique et toutes les ressources associées.

```powershell
Remove-AzResourceGroup -Name myResourceGroup
```

## <a name="script-explanation"></a>Explication du script
Ce script a recours aux commandes suivantes pour créer le déploiement. Chaque élément du tableau renvoie à une documentation spécifique.

| Commande | Notes |
|---|---|
| [New-AzVmss](/powershell/module/az.compute/new-azvmss) | Crée le groupe de machines virtuelles identiques et toutes les ressources de support, y compris le réseau virtuel, l’équilibreur de charge et les règles NAT. |
| [Get-AzVmss](/powershell/module/az.compute/get-azvmss) | Obtient des informations relatives à un groupe de machines virtuelles identiques. |
| [Add-AzVmssExtension](/powershell/module/az.compute/add-azvmssextension) | Ajoute une extension de machine virtuelle pour le script personnalisé afin d’installer une application web de base. |
| [Update-AzVmss](/powershell/module/az.compute/update-azvmss) | Met à jour le modèle de groupe de machines virtuelles identiques pour appliquer l’extension de machine virtuelle. |
| [Get-AzPublicIpAddress](/powershell/module/az.network/get-azpublicipaddress) | Obtient des informations sur l’adresse IP publique assignée qui est utilisée par l’équilibreur de charge. |
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Supprime un groupe de ressources et toutes les ressources contenues. |

## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur le module Azure PowerShell, consultez [Documentation Azure PowerShell](/powershell/azure/).

