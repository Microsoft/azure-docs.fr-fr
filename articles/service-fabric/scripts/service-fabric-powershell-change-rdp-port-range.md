---
title: Exemple de script Azure PowerShell - Changer la plage de ports RDP | Microsoft Docs
description: Exemple de script Azure PowerShell - Changer la plage de ports RDP d’un cluster déployé
services: service-fabric
tags: azure-service-management
author: athinanthny
ms.author: atsenthi
ms.service: service-fabric
ms.workload: multiple
ms.topic: sample
ms.date: 03/19/2018
ms.openlocfilehash: 750a2100d34e02cac7c613cc6b95160fc348b535
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96576364"
---
# <a name="update-the-rdp-port-range-values"></a>Mettre à jour les valeurs de la plage de ports RDP

Cet exemple de script change les valeurs de la plage de ports RDP sur les machines virtuelles de nœud de cluster une fois que le cluster a été déployé.  Azure PowerShell est utilisé afin que les machines virtuelles sous-jacentes ne s’interrompent pas.  Le script obtient la ressource `Microsoft.Network/loadBalancers` dans le groupe de ressources du cluster et met à jour les valeurs `inboundNatPools.frontendPortRangeStart` et `inboundNatPools.frontendPortRangeEnd`. Personnalisez les paramètres selon vos besoins.

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

Si nécessaire, installez Azure PowerShell à l’aide des instructions figurant dans le [Guide Azure PowerShell](/powershell/azure/).

## <a name="sample-script"></a>Exemple de script

[!code-powershell[main](../../../powershell_scripts/service-fabric/change-rdp-port-range/change-rdp-port-range.ps1 "Update the RDP port range values")]

## <a name="script-explanation"></a>Explication du script

Ce script utilise les commandes suivantes. Chaque commande du tableau renvoie à une documentation spécifique.

| Commande | Notes |
|---|---|
| [Get-AzResource](/powershell/module/az.resources/get-azresource) | Obtient la ressource `Microsoft.Network/loadBalancers`. |
|[Set-AzResource](/powershell/module/az.resources/set-azresource)|Met à jour la ressource `Microsoft.Network/loadBalancers`.|

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur le module Azure PowerShell, consultez [Documentation Azure PowerShell](/powershell/azure/).

Vous trouverez des exemples supplémentaires de scripts Azure PowerShell pour Azure Service Fabric sur la page [Azure PowerShell Samples](../service-fabric-powershell-samples.md) (Exemples Azure PowerShell).
