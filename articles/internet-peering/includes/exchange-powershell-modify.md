---
title: Fichier include
titleSuffix: Azure
description: Fichier include
services: internet-peering
author: prmitiki
ms.service: internet-peering
ms.topic: include
ms.date: 11/27/2019
ms.author: prmitiki
ms.openlocfilehash: 12d169697a35af446392843eb57e6ec3a5508e45
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "81678653"
---
Les opérations de modification suivantes sont prises en charge pour le Peering Exchange :
* Ajouter des connexions de Peering Exchange.
* Supprimer des connexions de Peering Exchange.
* Ajouter une session IPv4 ou IPv6 sur des connexions actives.
* Supprimer une session IPv4 ou IPv6 sur des connexions actives.


### <a name="add-exchange-peering-connections"></a>Ajouter des connexions de Peering Exchange

Cet exemple décrit comment ajouter des connexions à un Peering Exchange existant.

```powershell

$exchangePeering = Get-AzPeering -Name "SeattleExchangePeering" -ResourceGroupName "PeeringResourceGroup"

$connection = New-AzPeeringExchangeConnectionObject `
    -PeeringDBFacilityId $exchangeLocation[1].PeeringDBFacilityId `
    -PeerSessionIPv4Address 198.32.134.22 `
    -PeerSessionIPv6Address  2001:504:12::22 `
    -MaxPrefixesAdvertisedIPv4 2000 `
    -MaxPrefixesAdvertisedIPv6 2000 `

$exchangePeering.Connections.Add($connection)

$exchangePeering | Update-AzPeering

```

### <a name="remove-exchange-peering-connections"></a>Supprimer des connexions de peering Exchange

Cet exemple décrit comment supprimer des connexions sur un Peering Exchange existant.

```powershell

$exchangePeering = Get-AzPeering -Name "SeattleExchangePeering" -ResourceGroupName "PeeringResourceGroup"

```

Affichez toutes les connexions, puis sélectionnez la connexion que vous souhaitez supprimer. 

```powershell

$exchangePeering

Name              : SeattleExchangePeering
Sku.Name          : Basic_Exchange_Free
Kind              : Exchange
Connections       : {11}
PeerAsn.Id        : /subscriptions/{subscriptionId}/providers/Microsoft.Peering/peerAsns/{peerAsnName}
PeeringLocation   : Seattle
ProvisioningState : Succeeded
Location          : West US 2
Id                : /subscriptions/{subscriptionId}/resourceGroups/PeeringResourceGroup/providers/Microsoft.Peering/peerings/SeattleExchangePeering
Type              : Microsoft.Peering/peerings
Tags              : {}

```

Dans la commande suivante, au lieu de 0, entrez le numéro d’index de la connexion que vous souhaitez supprimer.

```powershell

$connection = $exchangePeering.Connections[0]

$exchangePeering.Connections.Remove($connection)

$exchangePeering | Update-AzPeering

```

### <a name="add-an-ipv4-or-ipv6-session-on-active-connections"></a>Ajouter une session IPv4 ou IPv6 sur des connexions actives

Cet exemple décrit comment ajouter une session IPv6 à une connexion Exchange existante.

```powershell

$exchangePeering = Get-AzPeering -Name "SeattleExchangePeering" -ResourceGroupName "PeeringResourceGroup"

$exchangePeering.Connections[0].BgpSession.PeerSessionIPv6Address = "2001:504:12::34"

$exchangePeering | Update-AzPeering

```

### <a name="remove-an-ipv4-or-ipv6-session-on-active-connections"></a>Supprimer une session IPv4 ou IPv6 sur des connexions actives

La suppression d’une session IPv4 ou IPv6 sur une connexion existante n’est actuellement pas prise en charge sur PowerShell. Pour plus d’informations, contactez l’équipe [Peering Microsoft](mailto:peeringexperience@microsoft.com).