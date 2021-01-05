---
title: 'Azure ExpressRoute : Configurer le Peering : classique'
description: Cet article vous guide tout au long des étapes de création et d’approvisionnement du peering privé, public et Microsoft d’un circuit ExpressRoute. Cet article vous montre également comment vérifier l'état, mettre à jour ou supprimer des peerings pour votre circuit.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 12/06/2019
ms.author: duau
ms.openlocfilehash: a4a3bad1e868fa0e75611630ffb5db5ba13126b6
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89395551"
---
# <a name="create-and-modify-peering-for-an-expressroute-circuit-classic"></a>Créer et modifier le peering pour un circuit ExpressRoute (Classic)
> [!div class="op_single_selector"]
> * [Azure portal](expressroute-howto-routing-portal-resource-manager.md)
> * [PowerShell](expressroute-howto-routing-arm.md)
> * [Azure CLI](howto-routing-cli.md)
> * [Vidéo - Peering privé](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-set-up-azure-private-peering-for-your-expressroute-circuit)
> * [Vidéo - Peering public](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-set-up-azure-public-peering-for-your-expressroute-circuit)
> * [Vidéo - Peering Microsoft](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-set-up-microsoft-peering-for-your-expressroute-circuit)
> * [PowerShell (classique)](expressroute-howto-routing-classic.md)
> 

Cet article vous guide tout au long des étapes de création et de gestion de la configuration du peering/routage d’un circuit ExpressRoute à l’aide de PowerShell et du modèle de déploiement classique. Les étapes ci-dessous vous montreront également comment vérifier l'état, mettre à jour ou supprimer et annuler le provisionnement des peerings d'un circuit ExpressRoute. Vous pouvez configurer un, deux ou les trois peerings (privé Azure, public Azure et Microsoft) pour un circuit ExpressRoute. Vous pouvez configurer les peerings dans l’ordre de votre choix. Toutefois, vous devez veiller à finaliser une par une la configuration de chaque peering. 

Ces instructions s’appliquent seulement aux circuits créés avec des fournisseurs de services proposant des services de connectivité de couche 2. Si vous utilisez un fournisseur de services proposant des services gérés de couche 3 (généralement un VPN IP, comme MPLS), votre fournisseur de connectivité configure et gère le routage pour vous.

[!INCLUDE [expressroute-classic-end-include](../../includes/expressroute-classic-end-include.md)]

**À propos des modèles de déploiement Azure**

[!INCLUDE [vpn-gateway-classic-rm](../../includes/vpn-gateway-classic-rm-include.md)]

## <a name="configuration-prerequisites"></a>Prérequis de configuration

* Veillez à consulter les pages relatives aux [conditions préalables](expressroute-prerequisites.md), à la [configuration requise pour le routage](expressroute-routing.md) et aux [flux de travail](expressroute-workflows.md) avant de commencer la configuration.
* Vous devez disposer d’un circuit ExpressRoute actif. Suivez les instructions permettant de [créer un circuit ExpressRoute](expressroute-howto-circuit-classic.md) et faites-le activer par votre fournisseur de connectivité avant de poursuivre. Le circuit ExpressRoute doit être dans un état approvisionné et activé pour être en mesure d'exécuter les applets de commande décrites ci-dessous.

### <a name="download-the-latest-powershell-cmdlets"></a>Télécharger les dernières applets de commande PowerShell

[!INCLUDE [classic powershell install instructions](../../includes/expressroute-poweshell-classic-install-include.md)]

## <a name="azure-private-peering"></a>Peering privé Azure

Cette section fournit des instructions sur la façon de créer, obtenir, mettre à jour et supprimer la configuration de peering privé Azure pour un circuit ExpressRoute. 

### <a name="to-create-azure-private-peering"></a>Pour créer un peering privé Azure

1. **Créez un circuit ExpressRoute.**

   Suivez les instructions permettant de [créer un circuit ExpressRoute](expressroute-howto-circuit-classic.md) et faites-le approvisionner par votre fournisseur de connectivité. Si votre fournisseur de connectivité propose des services gérés de couche 3, vous pouvez lui demander d’activer le peering privé Azure pour vous. Dans ce cas, vous n'aurez pas besoin de suivre les instructions indiquées dans les sections suivantes. Toutefois, si votre fournisseur de connectivité ne gère pas le routage pour vous, après avoir créé votre circuit, suivez les instructions ci-dessous.
2. **Vérifiez que le circuit ExpressRoute est approvisionné.**
   
   Vérifiez que le circuit ExpressRoute est approvisionné et activé.

   ```powershell
   Get-AzureDedicatedCircuit -ServiceKey "*********************************"
   ```

   Renvoie :

   ```powershell
   Bandwidth                        : 200
   CircuitName                      : MyTestCircuit
   Location                         : Silicon Valley
   ServiceKey                       : *********************************
   ServiceProviderName              : equinix
   ServiceProviderProvisioningState : Provisioned
   Sku                              : Standard
   Status                           : Enabled
   ```
   
   Assurez-vous que le circuit affiche l’état Provisioned (approvisionné) et Enabled (activé). Si ce n'est pas le cas, contactez votre fournisseur de connectivité afin que votre circuit affiche l’état requis.

   ```powershell
   ServiceProviderProvisioningState : Provisioned
   Status                           : Enabled
   ```
3. **Configurez le peering privé Azure pour le circuit.**

   Assurez-vous de disposer des éléments suivants avant de procéder aux étapes suivantes :
   
   * Un sous-réseau /30 pour le lien principal. Ce sous-réseau ne doit faire partie d’aucun espace d'adressage réservé aux réseaux virtuels.
   * Un sous-réseau /30 pour le lien secondaire. Ce sous-réseau ne doit faire partie d’aucun espace d'adressage réservé aux réseaux virtuels.
   * Un ID VLAN valide pour établir ce peering. Vérifiez qu'aucun autre peering sur le circuit n'utilise le même ID VLAN.
   * Un numéro AS pour le peering. Vous pouvez utiliser des numéros à 2 et 4 octets. Vous pouvez utiliser un numéro AS privé pour ce peering. Veillez à ne pas utiliser le numéro 65515.
   * Un hachage MD5 si vous choisissez d’en utiliser un. **Facultatif**.
     
   Vous pouvez utiliser l’exemple suivant pour configurer le peering privé Azure pour votre circuit :

   ```powershell
   New-AzureBGPPeering -AccessType Private -ServiceKey "*********************************" -PrimaryPeerSubnet "10.0.0.0/30" -SecondaryPeerSubnet "10.0.0.4/30" -PeerAsn 1234 -VlanId 100
   ```    

   Si vous souhaitez utiliser un code de hachage MD5, utilisez l’exemple suivant pour configurer le peering privé Azure pour votre circuit :

   ```powershell
   New-AzureBGPPeering -AccessType Private -ServiceKey "*********************************" -PrimaryPeerSubnet "10.0.0.0/30" -SecondaryPeerSubnet "10.0.0.4/30" -PeerAsn 1234 -VlanId 100 -SharedKey "A1B2C3D4"
   ```
     
   > [!IMPORTANT]
   > Veillez à spécifier votre numéro AS comme ASN de peering et non pas comme ASN client.
   > 

### <a name="to-view-azure-private-peering-details"></a>Pour afficher les détails d’un peering privé Azure

Vous pouvez obtenir des détails sur la configuration à l’aide de l’applet de commande suivante :

```powershell
Get-AzureBGPPeering -AccessType Private -ServiceKey "*********************************"
```

Renvoie :

```
AdvertisedPublicPrefixes       : 
AdvertisedPublicPrefixesState  : Configured
AzureAsn                       : 12076
CustomerAutonomousSystemNumber : 
PeerAsn                        : 1234
PrimaryAzurePort               : 
PrimaryPeerSubnet              : 10.0.0.0/30
RoutingRegistryName            : 
SecondaryAzurePort             : 
SecondaryPeerSubnet            : 10.0.0.4/30
State                          : Enabled
VlanId                         : 100
```

### <a name="to-update-azure-private-peering-configuration"></a>Pour mettre à jour la configuration de peering privé Azure

Vous pouvez mettre à jour toute partie de la configuration à l'aide de l’applet de commande suivante. Dans l'exemple ci-dessous, l'ID VLAN du circuit est mis à jour de 100 à 500.

```powershell
Set-AzureBGPPeering -AccessType Private -ServiceKey "*********************************" -PrimaryPeerSubnet "10.0.0.0/30" -SecondaryPeerSubnet "10.0.0.4/30" -PeerAsn 1234 -VlanId 500 -SharedKey "A1B2C3D4"
```

### <a name="to-delete-azure-private-peering"></a>Pour supprimer un peering privé Azure

Vous pouvez supprimer votre configuration de peering en exécutant l’applet de commande suivante. Vous devez vous assurer que tous les réseaux virtuels sont dissociés du circuit ExpressRoute avant d'exécuter cette applet de commande.

```powershell
Remove-AzureBGPPeering -AccessType Private -ServiceKey "*********************************"
```

## <a name="azure-public-peering"></a>Peering public Azure

Cette section fournit des instructions sur la façon de créer, d’obtenir, de mettre à jour et de supprimer la configuration de peering public Azure pour un circuit ExpressRoute.

> [!NOTE]
> Le peering public Azure est déprécié pour les nouveaux circuits.
>

### <a name="to-create-azure-public-peering"></a>Pour créer un peering public Azure

1. **Création d’un circuit ExpressRoute**

   Suivez les instructions permettant de [créer un circuit ExpressRoute](expressroute-howto-circuit-classic.md) et faites-le approvisionner par votre fournisseur de connectivité. Si votre fournisseur de connectivité propose des services gérés de couche 3, vous pouvez lui demander d’activer le peering privé Azure pour vous. Dans ce cas, vous n'aurez pas besoin de suivre les instructions indiquées dans les sections suivantes. Toutefois, si votre fournisseur de connectivité ne gère pas le routage pour vous, après avoir créé votre circuit, suivez les instructions ci-dessous.
2. **Vérification de l’approvisionnement du circuit ExpressRoute**

   Vous devez tout d'abord vérifier que le circuit ExpressRoute est approvisionné et activé.

   ```powershell
   Get-AzureDedicatedCircuit -ServiceKey "*********************************"
   ```

   Renvoie :

   ```powershell
   Bandwidth                        : 200
   CircuitName                      : MyTestCircuit
   Location                         : Silicon Valley
   ServiceKey                       : *********************************
   ServiceProviderName              : equinix
   ServiceProviderProvisioningState : Provisioned
   Sku                              : Standard
   Status                           : Enabled
   ```
   
   Assurez-vous que le circuit affiche l’état Provisioned (approvisionné) et Enabled (activé). Si ce n'est pas le cas, contactez votre fournisseur de connectivité afin que votre circuit affiche l’état requis.

   ```powershell
   ServiceProviderProvisioningState : Provisioned
   Status                           : Enabled
   ```
4. **Configuration du peering public Azure pour le circuit**
   
   Assurez-vous de disposer des informations suivantes avant de poursuivre :
   
   * Un sous-réseau /30 pour le lien principal. Ce doit être un préfixe IPv4 public valide.
   * Un sous-réseau /30 pour le lien secondaire. Ce doit être un préfixe IPv4 public valide.
   * Un ID VLAN valide pour établir ce peering. Vérifiez qu'aucun autre peering sur le circuit n'utilise le même ID VLAN.
   * Un numéro AS pour le peering. Vous pouvez utiliser des numéros à 2 et 4 octets.
   * Un hachage MD5 si vous choisissez d’en utiliser un. **Facultatif**.

   > [!IMPORTANT]
   > Veillez à spécifier votre numéro AS comme ASN de peering et non pas comme ASN client.
   >  
     
   Vous pouvez utiliser l’exemple suivant pour configurer le peering public Azure pour votre circuit :

   ```powershell
   New-AzureBGPPeering -AccessType Public -ServiceKey "*********************************" -PrimaryPeerSubnet "131.107.0.0/30" -SecondaryPeerSubnet "131.107.0.4/30" -PeerAsn 1234 -VlanId 200
   ```
     
   Si vous souhaitez utiliser un code de hachage MD5, utilisez l’exemple suivant pour configurer votre circuit :
     
   ```powershell
   New-AzureBGPPeering -AccessType Public -ServiceKey "*********************************" -PrimaryPeerSubnet "131.107.0.0/30" -SecondaryPeerSubnet "131.107.0.4/30" -PeerAsn 1234 -VlanId 200 -SharedKey "A1B2C3D4"
   ```
     
### <a name="to-view-azure-public-peering-details"></a>Pour afficher les détails d’un peering public Azure

Utilisez l’applet de commande suivante pour obtenir des détails sur la configuration :

```powershell
Get-AzureBGPPeering -AccessType Public -ServiceKey "*********************************"
```

Renvoie :

```powershell
AdvertisedPublicPrefixes       : 
AdvertisedPublicPrefixesState  : Configured
AzureAsn                       : 12076
CustomerAutonomousSystemNumber : 
PeerAsn                        : 1234
PrimaryAzurePort               : 
PrimaryPeerSubnet              : 131.107.0.0/30
RoutingRegistryName            : 
SecondaryAzurePort             : 
SecondaryPeerSubnet            : 131.107.0.4/30
State                          : Enabled
VlanId                         : 200
```

### <a name="to-update-azure-public-peering-configuration"></a>Pour mettre à jour la configuration de peering public Azure

Vous pouvez mettre à jour toute partie de la configuration à l'aide de l’applet de commande suivante. Dans cet exemple, l’ID VLAN du circuit est mis à jour de 200 à 600.

```powershell
Set-AzureBGPPeering -AccessType Public -ServiceKey "*********************************" -PrimaryPeerSubnet "131.107.0.0/30" -SecondaryPeerSubnet "131.107.0.4/30" -PeerAsn 1234 -VlanId 600 -SharedKey "A1B2C3D4"
```

Assurez-vous que le circuit affiche l’état Provisioned (approvisionné) et Enabled (activé). 
### <a name="to-delete-azure-public-peering"></a>Pour supprimer un peering public Azure

Vous pouvez supprimer votre configuration de peering en exécutant l’applet de commande suivante :

```powershell
Remove-AzureBGPPeering -AccessType Public -ServiceKey "*********************************"
```

## <a name="microsoft-peering"></a>Peering Microsoft

Cette section fournit des instructions sur la façon de créer, d’obtenir, de mettre à jour et de supprimer la configuration de peering Microsoft pour un circuit ExpressRoute. 

### <a name="to-create-microsoft-peering"></a>Pour créer un peering Microsoft

1. **Création d’un circuit ExpressRoute**
  
   Suivez les instructions permettant de [créer un circuit ExpressRoute](expressroute-howto-circuit-classic.md) et faites-le approvisionner par votre fournisseur de connectivité. Si votre fournisseur de connectivité propose des services gérés de couche 3, vous pouvez lui demander d’activer le peering privé Azure pour vous. Dans ce cas, vous n'aurez pas besoin de suivre les instructions indiquées dans les sections suivantes. Toutefois, si votre fournisseur de connectivité ne gère pas le routage pour vous, après avoir créé votre circuit, suivez les instructions ci-dessous.
2. **Vérification de l’approvisionnement du circuit ExpressRoute**

   Assurez-vous que le circuit affiche l’état Provisioned (approvisionné) et Enabled (activé). 
   
   ```powershell
   Get-AzureDedicatedCircuit -ServiceKey "*********************************"
   ```

   Renvoie :
   
   ```powershell
   Bandwidth                        : 200
   CircuitName                      : MyTestCircuit
   Location                         : Silicon Valley
   ServiceKey                       : *********************************
   ServiceProviderName              : equinix
   ServiceProviderProvisioningState : Provisioned
   Sku                              : Standard
   Status                           : Enabled
   ```
   
   Assurez-vous que le circuit affiche l’état Provisioned (approvisionné) et Enabled (activé). Si ce n'est pas le cas, contactez votre fournisseur de connectivité afin que votre circuit affiche l’état requis.

   ```powershell
   ServiceProviderProvisioningState : Provisioned
   Status                           : Enabled
   ```
3. **Configuration du peering Microsoft pour le circuit**
   
    Assurez-vous de disposer des informations suivantes avant de poursuivre.
   
   * Un sous-réseau /30 pour le lien principal. Il doit s’agir d’un préfixe IPv4 public valide vous appartenant et enregistré dans un registre RIR / IRR.
   * Un sous-réseau /30 pour le lien secondaire. Il doit s’agir d’un préfixe IPv4 public valide vous appartenant et enregistré dans un registre RIR / IRR.
   * Un ID VLAN valide pour établir ce peering. Vérifiez qu'aucun autre peering sur le circuit n'utilise le même ID VLAN.
   * Un numéro AS pour le peering. Vous pouvez utiliser des numéros à 2 et 4 octets.
   * Préfixes publiés : Vous devez fournir la liste de tous les préfixes que vous prévoyez de publier sur la session BGP. Seuls les préfixes d'adresses IP publiques sont acceptés. Vous pouvez envoyer une liste séparée par des virgules si vous prévoyez d'envoyer un jeu de préfixes. Ces préfixes doivent être enregistrés en votre nom dans un registre RIR / IRR.
   * ASN client : si vous publiez des préfixes non enregistrés dans le numéro AS de peering, vous pouvez spécifier le numéro AS avec lequel ils sont enregistrés. **Facultatif**.
   * Nom du registre de routage : Vous pouvez spécifier les registres RIR/IRR sur lesquels le numéro AS et les préfixes sont inscrits.
   * Un hachage MD5 si vous choisissez d’en utiliser un. **Facultatif.**
     
   Exécutez l'applet de commande suivante afin de configurer le peering Microsoft pour votre circuit :
 
   ```powershell
   New-AzureBGPPeering -AccessType Microsoft -ServiceKey "*********************************" -PrimaryPeerSubnet "131.107.0.0/30" -SecondaryPeerSubnet "131.107.0.4/30" -VlanId 300 -PeerAsn 1234 -CustomerAsn 2245 -AdvertisedPublicPrefixes "123.0.0.0/30" -RoutingRegistryName "ARIN" -SharedKey "A1B2C3D4"
   ```

### <a name="to-view-microsoft-peering-details"></a>Pour afficher les détails du peering Microsoft

Vous pouvez obtenir des détails sur la configuration à l’aide de l’applet de commande suivante :

```powershell
Get-AzureBGPPeering -AccessType Microsoft -ServiceKey "*********************************"
```
Renvoie :

```powershell
AdvertisedPublicPrefixes       : 123.0.0.0/30
AdvertisedPublicPrefixesState  : Configured
AzureAsn                       : 12076
CustomerAutonomousSystemNumber : 2245
PeerAsn                        : 1234
PrimaryAzurePort               : 
PrimaryPeerSubnet              : 10.0.0.0/30
RoutingRegistryName            : ARIN
SecondaryAzurePort             : 
SecondaryPeerSubnet            : 10.0.0.4/30
State                          : Enabled
VlanId                         : 300
```

### <a name="to-update-microsoft-peering-configuration"></a>Pour mettre à jour la configuration de peering Microsoft

Vous pouvez mettre à jour n’importe quelle partie de la configuration à l’aide de l’applet de commande suivante :

```powershell
Set-AzureBGPPeering -AccessType Microsoft -ServiceKey "*********************************" -PrimaryPeerSubnet "131.107.0.0/30" -SecondaryPeerSubnet "131.107.0.4/30" -VlanId 300 -PeerAsn 1234 -CustomerAsn 2245 -AdvertisedPublicPrefixes "123.0.0.0/30" -RoutingRegistryName "ARIN" -SharedKey "A1B2C3D4"
```

### <a name="to-delete-microsoft-peering"></a>Pour supprimer un peering Microsoft

Vous pouvez supprimer votre configuration de peering en exécutant l’applet de commande suivante :

```powershell
Remove-AzureBGPPeering -AccessType Microsoft -ServiceKey "*********************************"
```

## <a name="next-steps"></a>Étapes suivantes

Ensuite, [liez un réseau virtuel à un circuit ExpressRoute](expressroute-howto-linkvnet-classic.md).

* Pour plus d'informations sur les workflows, consultez [Workflows ExpressRoute](expressroute-workflows.md).
* Pour plus d’informations sur le peering du circuit, consultez [Circuits ExpressRoute et domaines de routage](expressroute-circuit-peerings.md).
