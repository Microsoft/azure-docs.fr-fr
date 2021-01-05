---
title: 'Tutoriel : Configurer une homologation pour un circuit ExpressRoute – Azure PowerShell'
description: Ce tutoriel explique comment créer et gérer une configuration de routage pour un circuit ExpressRoute dans le modèle de déploiement Resource Manager à l’aide de PowerShell.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: tutorial
ms.date: 10/08/2020
ms.author: duau
ms.openlocfilehash: 641d7eeef96af84f0f058aebd19d795083e3567f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91855342"
---
# <a name="tutorial-create-and-modify-peering-for-an-expressroute-circuit-using-powershell"></a>Tutoriel : Créer et modifier le peering d’un circuit ExpressRoute à l’aide de PowerShell

Ce tutoriel vous aide à créer et gérer une configuration de routage pour un circuit ExpressRoute dans le modèle de déploiement Resource Manager à l’aide de PowerShell. Vous pouvez également mettre à jour, supprimer et déprovisionner des peerings d’un circuit ExpressRoute, ainsi qu’en vérifier l’état. Si vous souhaitez utiliser une autre méthode pour votre circuit, sélectionnez un article dans la liste suivante :

> [!div class="op_single_selector"]
> * [Azure portal](expressroute-howto-routing-portal-resource-manager.md)
> * [PowerShell](expressroute-howto-routing-arm.md)
> * [Azure CLI](howto-routing-cli.md)
> * [Peering public](about-public-peering.md)
> * [Vidéo - Peering privé](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-set-up-azure-private-peering-for-your-expressroute-circuit)
> * [Vidéo - Peering Microsoft](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-set-up-microsoft-peering-for-your-expressroute-circuit)
> * [PowerShell (classique)](expressroute-howto-routing-classic.md)
> 

Ces instructions s'appliquent uniquement aux circuits créés avec des fournisseurs de services proposant des services de connectivité de couche 2. Si vous utilisez un fournisseur de services proposant des services gérés de couche 3 (généralement un IPVPN, comme MPLS), votre fournisseur de connectivité configure et gère le routage pour vous.

> [!IMPORTANT]
> Nous n'annonçons pas de peerings configurés par les fournisseurs de services via le portail de gestion des services. Nous travaillons sur la prochaine activation de cette fonctionnalité. Contactez votre fournisseur de services avant de configurer des peerings BGP.
> 

Vous pouvez configurer le peering privé et le peering Microsoft pour un circuit ExpressRoute (le peering public Azure est déprécié pour les nouveaux circuits). Les peerings peuvent être configurés dans l’ordre de votre choix. Toutefois, vous devez veiller à finaliser une par une la configuration de chaque peering. Pour plus d’informations sur les domaines de routage et les peerings, consultez [Domaines de routage ExpressRoute](expressroute-circuit-peerings.md). Pour obtenir des informations sur le peering public, consultez [Peering public ExpressRoute](about-public-peering.md).

Dans ce tutoriel, vous allez apprendre à :
> [!div class="checklist"]
> - configurer, mettre à jour et supprimer une homologation Microsoft pour un circuit ;
> - configurer, mettre à jour et supprimer une homologation privée Azure pour un circuit.

## <a name="prerequisites"></a>Prérequis

* Avant de commencer la configuration, veillez à consulter les pages suivantes :
    * [Composants requis](expressroute-prerequisites.md) 
    * [Configuration requise du routage](expressroute-routing.md)
    * [Flux de travail](expressroute-workflows.md)
* Vous devez disposer d’un circuit ExpressRoute actif. Suivez les instructions fournies pour [Créer un circuit ExpressRoute](expressroute-howto-circuit-arm.md) puis, avant de poursuivre, demandez à votre fournisseur de connectivité de l’activer. Pour que vous puissiez exécuter les cmdlets décrites dans cet article, le circuit ExpressRoute doit être dans un état approvisionné et activé.

[!INCLUDE [cloud-shell-try-it.md](../../includes/cloud-shell-try-it.md)]

## <a name="microsoft-peering"></a><a name="msft"></a>Peering Microsoft

Cette section explique comment créer, obtenir, mettre à jour et supprimer la configuration de peering Microsoft pour un circuit ExpressRoute.

> [!IMPORTANT]
> Le peering Microsoft des circuits ExpressRoute qui ont été configurés avant le 1er août 2017 entraînera la publication de tous les préfixes de service via le peering Microsoft, même si aucun filtre d’itinéraire n’est défini. Le peering Microsoft des circuits ExpressRoute qui sont configurés le 1er août 2017 ou après n’entraînera la publication d’aucun préfixe tant qu’un filtre de routage n’aura pas été attaché au circuit. Pour plus d’informations, consultez [Configure a route filter for Microsoft peering](how-to-routefilter-powershell.md) (Configurer un filtre d’itinéraire pour l’homologation Microsoft).
> 

### <a name="to-create-microsoft-peering"></a>Pour créer un peering Microsoft

1. Connectez-vous et sélectionnez votre abonnement.

   Si vous avez installé PowerShell localement, connectez-vous. Si vous utilisez Azure Cloud Shell, vous pouvez ignorer cette étape.

   ```azurepowershell
   Connect-AzAccount
   ```

   Sélectionnez l’abonnement pour lequel vous souhaitez créer le circuit ExpressRoute.

   ```azurepowershell-interactive
   Select-AzSubscription -SubscriptionId "<subscription ID>"
   ```
2. Créez un circuit ExpressRoute.

   Suivez les instructions permettant de [créer un circuit ExpressRoute](expressroute-howto-circuit-arm.md) et faites-le approvisionner par votre fournisseur de connectivité. Si votre fournisseur de connectivité propose des services gérés de couche 3, vous pouvez lui demander d’activer le peering Microsoft pour vous. Vous n’avez pas besoin de suivre les instructions des sections suivantes. En revanche, si votre fournisseur de connectivité ne gère pas le routage pour vous, après avoir créé votre circuit, poursuivez la configuration en procédant comme suit. 

3. Vérifiez que le circuit ExpressRoute est approvisionné et activé. Consultez l’exemple qui suit :

   ```azurepowershell-interactive
   Get-AzExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"
   ```

   La réponse ressemble à ce qui suit :

   ```
   Name                             : ExpressRouteARMCircuit
   ResourceGroupName                : ExpressRouteResourceGroup
   Location                         : westus
   Id                               : /subscriptions/***************************/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit
   Etag                             : W/"################################"
   ProvisioningState                : Succeeded
   Sku                              : {
                                       "Name": "Standard_MeteredData",
                                       "Tier": "Standard",
                                       "Family": "MeteredData"
                                     }
   CircuitProvisioningState         : Enabled
   ServiceProviderProvisioningState : Provisioned
   ServiceProviderNotes             : 
   ServiceProviderProperties        : {
                                       "ServiceProviderName": "Equinix",
                                       "PeeringLocation": "Silicon Valley",
                                       "BandwidthInMbps": 200
                                     }
   ServiceKey                       : **************************************
   Peerings                         : []
   ```
4. Configurez le peering Microsoft pour le circuit. Avant de poursuivre, assurez-vous de disposer des informations suivantes.

   * Un sous-réseau /30 ou /126 pour le lien principal. Le bloc d’adresse doit être un préfixe IPv4 ou IPv6 public valide vous appartenant et inscrit dans un registre RIR / IRR.
   * Un sous-réseau /30 ou /126 pour le lien secondaire. Le bloc d’adresse doit être un préfixe IPv4 ou IPv6 public valide vous appartenant et inscrit dans un registre RIR / IRR.
   * Un ID VLAN valide pour établir ce peering. Assurez-vous qu'aucun autre peering sur le circuit n'utilise le même ID VLAN.
   * Un numéro AS pour le peering. Vous pouvez utiliser des numéros à 2 et 4 octets.
   * Préfixes publiés : vous fournissez la liste de tous les préfixes que vous prévoyez de publier sur la session BGP. Seuls les préfixes d'adresses IP publiques sont acceptés. Si vous prévoyez d’envoyer un jeu de préfixes, vous pouvez envoyer une liste séparée par des virgules. Ces préfixes doivent être enregistrés en votre nom dans un registre RIR / IRR. Les sessions IPv4 BGP nécessitent des préfixes publiés IPv4 et les sessions IPv6 BGP nécessitent des préfixes publiés IPv6. 
   * Nom du registre de routage : Vous pouvez spécifier les registres RIR/IRR sur lesquels le numéro AS et les préfixes sont inscrits.
   * Facultatif :
     * ASN client : si vous publiez des préfixes non inscrits avec le numéro AS d’homologation, vous pouvez spécifier le numéro AS avec lequel ils sont inscrits.
     * Un hachage MD5 si vous choisissez d’en utiliser un.

> [!IMPORTANT]
> Microsoft vérifie si les « préfixes publics publiés » et « ASN pairs » (ou « ASN client ») spécifiés vous sont attribués dans le registre de routage Internet. Si vous obtenez les préfixes publics d’une autre entité et si l’affectation n’est pas enregistrée avec le registre de routage, la validation automatique ne se termine pas et nécessite une validation manuelle. Si la validation automatique échoue, vous verrez « AdvertisedPublicPrefixesState » comme « Validation requise » sur la sortie de « Get-AzExpressRouteCircuitPeeringConfig » (voir la commande « Pour obtenir des détails sur le Peering Microsoft » dans la section suivante). 
> 
> Si vous voyez le message « Validation nécessaire », collectez le ou les documents contenant les préfixes publics attribués à votre organisation par l’entité répertoriée comme propriétaire des préfixes dans le registre de routage, et soumettez ces documents pour validation manuelle en ouvrant un ticket de support. 
> 

   Utilisez l’exemple suivant afin de configurer le peering Microsoft pour votre circuit :

   ```azurepowershell-interactive
   Add-AzExpressRouteCircuitPeeringConfig -Name "MicrosoftPeering" -ExpressRouteCircuit $ckt -PeeringType MicrosoftPeering -PeerASN 100 -PeerAddressType IPv4 -PrimaryPeerAddressPrefix "123.0.0.0/30" -SecondaryPeerAddressPrefix "123.0.0.4/30" -VlanId 300 -MicrosoftConfigAdvertisedPublicPrefixes "123.1.0.0/24" -MicrosoftConfigCustomerAsn 23 -MicrosoftConfigRoutingRegistryName "ARIN"

   Add-AzExpressRouteCircuitPeeringConfig -Name "MicrosoftPeering" -ExpressRouteCircuit $ckt -PeeringType MicrosoftPeering -PeerASN 100 -PeerAddressType IPv6 -PrimaryPeerAddressPrefix "3FFE:FFFF:0:CD30::/126" -SecondaryPeerAddressPrefix "3FFE:FFFF:0:CD30::4/126" -VlanId 300 -MicrosoftConfigAdvertisedPublicPrefixes "3FFE:FFFF:0:CD31::/120" -MicrosoftConfigCustomerAsn 23 -MicrosoftConfigRoutingRegistryName "ARIN"

   Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
   ```

### <a name="to-get-microsoft-peering-details"></a><a name="getmsft"></a>Pour obtenir des détails sur le peering Microsoft

Vous pouvez obtenir les détails de la configuration à l’aide de l’exemple suivant :

```azurepowershell-interactive
$ckt = Get-AzExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

Get-AzExpressRouteCircuitPeeringConfig -Name "MicrosoftPeering" -ExpressRouteCircuit $ckt
```

### <a name="to-update-microsoft-peering-configuration"></a><a name="updatemsft"></a>Pour mettre à jour la configuration de peering Microsoft

Vous pouvez mettre à jour toute partie de la configuration à l’aide de l’exemple suivant :

```azurepowershell-interactive
Set-AzExpressRouteCircuitPeeringConfig  -Name "MicrosoftPeering" -ExpressRouteCircuit $ckt -PeeringType MicrosoftPeering -PeerASN 100 -PeerAddressType IPv4 -PrimaryPeerAddressPrefix "123.0.0.0/30" -SecondaryPeerAddressPrefix "123.0.0.4/30" -VlanId 300 -MicrosoftConfigAdvertisedPublicPrefixes "124.1.0.0/24" -MicrosoftConfigCustomerAsn 23 -MicrosoftConfigRoutingRegistryName "ARIN"

Set-AzExpressRouteCircuitPeeringConfig  -Name "MicrosoftPeering" -ExpressRouteCircuit $ckt -PeeringType MicrosoftPeering -PeerASN 100 -PeerAddressType IPv6 -PrimaryPeerAddressPrefix "3FFE:FFFF:0:CD30::/126" -SecondaryPeerAddressPrefix "3FFE:FFFF:0:CD30::4/126" -VlanId 300 -MicrosoftConfigAdvertisedPublicPrefixes "3FFE:FFFF:0:CD31::/120" -MicrosoftConfigCustomerAsn 23 -MicrosoftConfigRoutingRegistryName "ARIN"

Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
```

## <a name="azure-private-peering"></a><a name="private"></a>Peering privé Azure

Cette section explique comment créer, obtenir, mettre à jour et supprimer la configuration de peering privé Azure pour un circuit ExpressRoute.

### <a name="to-create-azure-private-peering"></a>Pour créer un peering privé Azure

1. Importez le module PowerShell pour ExpressRoute.

   Installez le dernier programme d’installation de PowerShell à partir de [PowerShell Gallery](https://www.powershellgallery.com/). Importez ensuite les modules Azure Resource Manager dans la session PowerShell pour commencer à utiliser les cmdlets ExpressRoute. Vous devez exécuter PowerShell en tant qu’administrateur.

   ```azurepowershell-interactive
   Install-Module Az
   ```

   Importez tous les modules Az.\* dans la plage de version sémantique connue.

   ```azurepowershell-interactive
   Import-Module Az
   ```

   Vous pouvez également importer un seul module sélectionné dans la plage de version sémantique connue.

   ```azurepowershell-interactive
   Import-Module Az.Network 
   ```

   Connectez-vous à votre compte.

   ```azurepowershell-interactive
   Connect-AzAccount
   ```

   Sélectionnez l’abonnement pour lequel vous souhaitez créer le circuit ExpressRoute.

   ```azurepowershell-interactive
   Select-AzSubscription -SubscriptionId "<subscription ID>"
   ```
2. Créez un circuit ExpressRoute.

   Suivez les instructions permettant de [créer un circuit ExpressRoute](expressroute-howto-circuit-arm.md) et faites-le approvisionner par votre fournisseur de connectivité. Si votre fournisseur de connectivité propose des services gérés de couche 3, vous pouvez lui demander d’activer le peering privé Azure pour vous. Vous n’avez pas besoin de suivre les instructions des sections suivantes. En revanche, si votre fournisseur de connectivité ne gère pas le routage pour vous, après avoir créé votre circuit, poursuivez la configuration en procédant comme suit.

3. Vérifiez que le circuit ExpressRoute est approvisionné et activé. Consultez l’exemple qui suit :

   ```azurepowershell-interactive
   Get-AzExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"
   ```

   La réponse ressemble à ce qui suit :

   ```
   Name                             : ExpressRouteARMCircuit
   ResourceGroupName                : ExpressRouteResourceGroup
   Location                         : westus
   Id                               : /subscriptions/***************************/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/ExpressRouteARMCircuit
   Etag                             : W/"################################"
   ProvisioningState                : Succeeded
   Sku                              : {
                                       "Name": "Standard_MeteredData",
                                       "Tier": "Standard",
                                       "Family": "MeteredData"
                                     }
   CircuitProvisioningState         : Enabled
   ServiceProviderProvisioningState : Provisioned
   ServiceProviderNotes             : 
   ServiceProviderProperties        : {
                                       "ServiceProviderName": "Equinix",
                                       "PeeringLocation": "Silicon Valley",
                                       "BandwidthInMbps": 200
                                     }
   ServiceKey                       : **************************************
   Peerings                         : []
   ```
4. Configurez le peering privé Azure pour le circuit. Avant de passer aux étapes suivantes, assurez-vous de disposer des éléments suivants :

   * Un sous-réseau /30 pour le lien principal. Le sous-réseau ne doit faire partie d’aucun espace d’adressage réservé aux réseaux virtuels.
   * Un sous-réseau /30 pour le lien secondaire. Le sous-réseau ne doit faire partie d’aucun espace d’adressage réservé aux réseaux virtuels.
   * Un ID VLAN valide pour établir ce peering. Assurez-vous qu'aucun autre peering sur le circuit n'utilise le même ID VLAN.
   * Un numéro AS pour le peering. Vous pouvez utiliser des numéros à 2 et 4 octets. Vous pouvez utiliser un numéro AS privé pour ce peering. Assurez-vous que vous n’utilisez pas 65515.
   * Facultatif :
     * Un hachage MD5 si vous choisissez d’en utiliser un.

   Utilisez l’exemple suivant pour configurer le peering privé Azure pour votre circuit :

   ```azurepowershell-interactive
   Add-AzExpressRouteCircuitPeeringConfig -Name "AzurePrivatePeering" -ExpressRouteCircuit $ckt -PeeringType AzurePrivatePeering -PeerASN 100 -PrimaryPeerAddressPrefix "10.0.0.0/30" -SecondaryPeerAddressPrefix "10.0.0.4/30" -VlanId 200

   Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
   ```

   Si vous choisissez d’utiliser un hachage MD5, utilisez l’exemple suivant :

   ```azurepowershell-interactive
   Add-AzExpressRouteCircuitPeeringConfig -Name "AzurePrivatePeering" -ExpressRouteCircuit $ckt -PeeringType AzurePrivatePeering -PeerASN 100 -PrimaryPeerAddressPrefix "10.0.0.0/30" -SecondaryPeerAddressPrefix "10.0.0.4/30" -VlanId 200  -SharedKey "A1B2C3D4"
   ```

   > [!IMPORTANT]
   > Veillez à spécifier votre numéro AS comme ASN de peering et non pas comme ASN client.
   > 
   >

### <a name="to-get-azure-private-peering-details"></a><a name="getprivate"></a>Pour obtenir les détails d’un peering privé Azure

Vous pouvez obtenir les détails de la configuration à l’aide de l’exemple suivant :

```azurepowershell-interactive
$ckt = Get-AzExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

Get-AzExpressRouteCircuitPeeringConfig -Name "AzurePrivatePeering" -ExpressRouteCircuit $ckt
```

### <a name="to-update-azure-private-peering-configuration"></a><a name="updateprivate"></a>Pour mettre à jour la configuration de peering privé Azure

Vous pouvez mettre à jour toute partie de la configuration à l’aide de l’exemple suivant : Dans cet exemple, l’ID VLAN du circuit est mis à jour de 100 à 500.

```azurepowershell-interactive
Set-AzExpressRouteCircuitPeeringConfig -Name "AzurePrivatePeering" -ExpressRouteCircuit $ckt -PeeringType AzurePrivatePeering -PeerASN 100 -PrimaryPeerAddressPrefix "10.0.0.0/30" -SecondaryPeerAddressPrefix "10.0.0.4/30" -VlanId 200

Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

### <a name="to-delete-microsoft-peering"></a><a name="deletemsft"></a>Pour supprimer le peering Microsoft

Vous pouvez supprimer votre configuration de peering en exécutant l’applet de commande suivante :

```azurepowershell-interactive
Remove-AzExpressRouteCircuitPeeringConfig -Name "MicrosoftPeering" -ExpressRouteCircuit $ckt

Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
```

### <a name="to-delete-azure-private-peering"></a><a name="deleteprivate"></a>Pour supprimer un peering privé Azure

Vous pouvez supprimer votre configuration de peering en exécutant l’exemple suivant :

> [!WARNING]
> Vous devez vous assurer que la totalité des réseaux virtuels et des connexions ExpressRoute Global Reach sont supprimés avant d’exécuter cet exemple. 
> 

```azurepowershell-interactive
Remove-AzExpressRouteCircuitPeeringConfig -Name "AzurePrivatePeering" -ExpressRouteCircuit $ckt

Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
```

## <a name="next-steps"></a>Étapes suivantes

Une fois que vous avez configuré l’homologation privée Azure, vous pouvez créer une passerelle ExpressRoute pour lier un réseau virtuel au circuit. 

> [!div class="nextstepaction"]
> [Configurer une passerelle de réseau virtuel pour ExpressRoute](expressroute-howto-add-gateway-resource-manager.md)
