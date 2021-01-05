---
title: Conversion d’un peering Direct hérité en ressource Azure avec PowerShell
titleSuffix: Azure
description: Convertissez un peering Direct hérité en ressource Azure avec PowerShell.
services: internet-peering
author: prmitiki
ms.service: internet-peering
ms.topic: how-to
ms.date: 11/27/2019
ms.author: prmitiki
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: d3e7cdf11e1e1e033b4e72b9579d8c63b28e6c88
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89071684"
---
# <a name="convert-a-legacy-direct-peering-to-an-azure-resource-by-using-powershell"></a>Conversion d’un peering Direct hérité en ressource Azure avec PowerShell

Cet article explique comment convertir un peering Direct hérité en ressource Azure avec les cmdlets PowerShell.

Si vous préférez, vous pouvez suivre ce guide en utilisant le [Portail](howto-legacy-direct-portal.md) Azure.

## <a name="before-you-begin"></a>Avant de commencer
* Lisez les [prérequis](prerequisites.md) et la [procédure pas à pas du peering Direct](walkthrough-direct-all.md) avant de commencer la configuration.

### <a name="work-with-azure-powershell"></a>Utilisation d’Azure PowerShell
[!INCLUDE [CloudShell](./includes/cloudshell-powershell-about.md)]

## <a name="convert-a-legacy-direct-peering-to-an-azure-resource"></a>Conversion d’un peering Direct hérité en ressource Azure

### <a name="sign-in-to-your-azure-account-and-select-your-subscription"></a>Vous connecter à votre compte Azure et sélectionner votre abonnement
[!INCLUDE [Account](./includes/account-powershell.md)]

### <a name="get-a-legacy-direct-peering-for-conversion"></a><a name= get></a>Récupération d’un peering Direct hérité pour la conversion
Cet exemple montre comment obtenir un peering Direct hérité à l’emplacement de peering Seattle.

```powershell
$legacyPeering = Get-AzLegacyPeering `
    -Kind Direct -PeeringLocation "Seattle"
$legacyPeering
```

Voici un exemple de réponse :
```powershell
Name                       :
Sku                        : Basic_Direct_Free
Kind                       : Direct
PeeringLocation            : Seattle
UseForPeeringService       : False
PeerAsn.Id                 :
Connection                 : ------------------------
PeeringDBFacilityId        : 71
SessionPrefixIPv4          : 4.71.156.72/30
PeerSessionIPv4Address     : 4.71.156.73
MicrosoftIPv4Address       : 4.71.156.74
SessionStateV4             : Established
MaxPrefixesAdvertisedV4    : 20000
SessionPrefixIPv6          : 2001:1900:2100::1e10/126
MaxPrefixesAdvertisedV6    : 2000
ConnectionState            : Active
BandwidthInMbps            : 0
ProvisionedBandwidthInMbps : 20000
Connection                 : ------------------------
PeeringDBFacilityId        : 71
SessionPrefixIPv4          : 4.68.70.140/30
PeerSessionIPv4Address     : 4.68.70.141
MicrosoftIPv4Address       : 4.68.70.142
SessionStateV4             : Established
MaxPrefixesAdvertisedV4    : 20000
SessionPrefixIPv6          : 2001:1900:4:3::cc/126
PeerSessionIPv6Address     : 2001:1900:4:3::cd
MicrosoftIPv6Address       : 2001:1900:4:3::ce
SessionStateV6             : Established
MaxPrefixesAdvertisedV6    : 2000
ConnectionState            : Active
BandwidthInMbps            : 0
ProvisionedBandwidthInMbps : 20000
ProvisioningState          : Succeeded
```

### <a name="convert-a-legacy-direct-peering"></a>Conversion d’un peering Direct hérité

&nbsp;
> [!IMPORTANT]
> Lors de la conversion d’un peering hérité en ressource Azure, les modifications ne sont pas prises en charge. &nbsp;

Utilisez cette commande pour convertir un peering Direct hérité en ressource Azure :

```powershell
$legacyPeering[0] | New-AzPeering `
    -Name "SeattleDirectPeering" `
    -ResourceGroupName "PeeringResourceGroup" `

```

Voici un exemple de réponse :

```powershell
Name                 : SeattleDirectPeering
Sku.Name             : Basic_Direct_Free
Kind                 : Direct
Connections          : {11, 11}
PeerAsn.Id           : /subscriptions/{subscriptionId}/providers/Microsoft.Peering/peerAsns/{asnNumber}
UseForPeeringService : False
PeeringLocation      : Seattle
ProvisioningState    : Succeeded
Location             : centralus
Id                   : /subscriptions/{subscriptionId}/resourceGroups/PeeringResourceGroup/providers/Microsoft.Peering/peerings/SeattleDirectPeering
Type                 : Microsoft.Peering/peerings
Tags                 : {}
```

## <a name="additional-resources"></a>Ressources supplémentaires
Pour obtenir une description détaillée de tous les paramètres, exécutez cette commande :

```powershell
Get-Help Get-AzPeering -detailed
```

Pour plus d’informations, consultez [FAQ sur le peering Internet](faqs.md).

## <a name="next-steps"></a>Étapes suivantes

* [Création ou modification d’un peering Direct avec PowerShell](howto-direct-powershell.md)
