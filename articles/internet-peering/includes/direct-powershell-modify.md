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
ms.openlocfilehash: b2609a069872ec55ac9068fadcbb3f312d68a630
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "81680789"
---
Cette section décrit comment effectuer les opérations de modification suivantes pour le peering direct :

* Ajouter des connexions de Peering direct.
* Supprimer des connexions de Peering direct.
* Mettez à niveau ou rétrogradez la bande passante sur les connexions actives.
* Ajouter des sessions IPv4 ou IPv6 sur des connexions actives.
* Supprimer des sessions IPv4 ou IPv6 sur des connexions actives.

### <a name="add-direct-peering-connections"></a>Ajouter les connexions de Peering direct

Cet exemple décrit comment ajouter des connexions au Peering direct existant.

```powershell

$directPeering = Get-AzPeering -Name "SeattleDirectPeering" -ResourceGroupName "PeeringResourceGroup"

$connection = New-AzPeeringDirectConnection `
    -PeeringDBFacilityId $peeringLocation.PeeringDBFacilityId `
    -SessionPrefixV4 "10.22.31.0/31" `
    -SessionPrefixV6 "fe02::3e:0/127" `
    -MaxPrefixesAdvertisedIPv4 1000 `
    -MaxPrefixesAdvertisedIPv6 100 `
    -BandwidthInMbps 10000

$directPeering.Connections.Add($connection)

$directPeering | Update-AzPeering
```

### <a name="remove-direct-peering-connections"></a>Supprimer les connexions de Peering direct

Pour le moment, PowerShell ne prend pas en charge la suppression d’une connexion. Pour plus d’informations, contactez l’équipe [Peering Microsoft](mailto:peeringexperience@microsoft.com).

<!--
```powershell
$directPeering.Connections.Remove($directPeering.Connections[0])

$directPeering | Update-AzPeering
```
-->

### <a name="upgrade-or-downgrade-bandwidth-on-active-connections"></a>Mettre à niveau ou rétrograder la bande passante sur les connexions actives

Cet exemple décrit comment ajouter 10 Gbits/s à une connexion directe existante.

```powershell

$directPeering = Get-AzPeering -Name "SeattleDirectPeering" -ResourceGroupName "PeeringResourceGroup"
$directPeering.Connections[0].BandwidthInMbps  = 20000
$directPeering | Update-AzPeering

```

### <a name="add-ipv4-or-ipv6-sessions-on-active-connections"></a>Ajouter des sessions IPv4 ou IPv6 sur des connexions actives

Cet exemple décrit comment ajouter une session IPv6 sur une connexion directe existante avec une seule session IPv4. 

```powershell

$directPeering = Get-AzPeering -Name "SeattleDirectPeering" -ResourceGroupName "PeeringResourceGroup"
$directPeering.Connections[0].BGPSession.SessionPrefixv6 = "fe01::3e:0/127"
$directPeering | Update-AzPeering

```

### <a name="remove-ipv4-or-ipv6-sessions-on-active-connections"></a>Supprimer des sessions IPv4 ou IPv6 sur des connexions actives

La suppression d’une session IPv4 ou IPv6 sur une connexion existante n’est actuellement pas prise en charge sur PowerShell. Pour plus d’informations, contactez l’équipe [Peering Microsoft](mailto:peeringexperience@microsoft.com).