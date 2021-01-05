---
title: Déployer un pare-feu Azure avec Zones de disponibilité à l’aide de PowerShell
description: Dans cet article, vous allez apprendre à déployer un pare-feu Azure avec Zones de disponibilité à l’aide d’Azure PowerShell.
services: firewall
author: vhorne
ms.service: firewall
ms.topic: how-to
ms.date: 11/19/2019
ms.author: victorh
ms.openlocfilehash: 46ecc4754a064e26e61365ed6ca167606bef9d81
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94656103"
---
# <a name="deploy-an-azure-firewall-with-availability-zones-using-azure-powershell"></a>Déployer un pare-feu Azure avec des zones de disponibilité à l’aide d’Azure PowerShell

Le Pare-feu Azure peut être configuré pendant le déploiement pour couvrir plusieurs zones de disponibilité afin de fournir une disponibilité supérieure.

Cette fonctionnalité donne accès aux scénarios suivants :

- Vous pouvez augmenter la disponibilité à 99,99 %. Pour plus d’informations, consultez [Contrat de niveau de service (SLA)](https://azure.microsoft.com/support/legal/sla/azure-firewall/v1_0/) du Pare-feu Azure. Un contrat de niveau de service de 99,99 % est offert quand deux Zones de disponibilité ou plus sont sélectionnées.
- Vous pouvez également associer le Pare-feu Azure à une zone spécifique uniquement pour des raisons de proximité, en utilisant le contrat de niveau de service standard garantissant un taux de disponibilité de 99,95 %.

Pour plus d’informations sur les Zones de disponibilité du Pare-feu Azure, consultez [Qu’est-ce que le Pare-feu Azure ?](overview.md).

L’exemple Azure PowerShell suivant montre comment vous pouvez déployer un pare-feu Azure avec Zones de disponibilité.

## <a name="create-a-firewall-with-availability-zones"></a>Créer un pare-feu avec Zones de disponibilité

Cet exemple crée un pare-feu dans les zones 1, 2 et 3.

Quand l’adresse IP publique standard est créée, aucune zone n’est spécifiée. Cette opération crée une adresse IP redondante interzone par défaut. Les adresses IP publiques standard peuvent être configurées dans toutes les zones ou dans une seule zone.

Il est important de le savoir, car vous ne pouvez pas avoir un pare-feu dans la zone 1 et une adresse IP dans la zone 2. Mais vous pouvez avoir un pare-feu dans la zone 1 et l’adresse IP dans toutes les zones, ou bien un pare-feu et une adresse IP dans la même zone à des fins de proximité.

```azurepowershell
$rgName = "resourceGroupName"

$vnet = Get-AzVirtualNetwork `
  -Name "vnet" `
  -ResourceGroupName $rgName

$pip1 = New-AzPublicIpAddress `
  -Name "AzFwPublicIp1" `
  -ResourceGroupName "rg" `
  -Sku "Standard" `
  -Location "centralus" `
  -AllocationMethod Static

New-AzFirewall `
  -Name "azFw" `
  -ResourceGroupName $rgName `
  -Location centralus `
  -VirtualNetwork $vnet `
  -PublicIpAddress @($pip1) `
  -Zone 1,2,3
```

## <a name="next-steps"></a>Étapes suivantes

- [Tutoriel : Surveiller les journaux d’activité de Pare-feu Azure](./firewall-diagnostics.md)