---
title: Créer et gérer le peering public Azure ExpressRoute
description: En savoir plus sur la gestion du peering public Azure
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: conceptual
ms.date: 12/16/2019
ms.author: duau
ms.openlocfilehash: 477145619e1b4d8b41c422389b57a46615597478
ms.sourcegitcommit: 957c916118f87ea3d67a60e1d72a30f48bad0db6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/19/2020
ms.locfileid: "92202546"
---
# <a name="create-and-manage-expressroute-public-peering"></a>Créer et gérer le peering public ExpressRoute

> [!div class="op_single_selector"]
> * [Article - Peering public](about-public-peering.md)
> * [Vidéo - Peering public](https://azure.microsoft.com/documentation/videos/azure-expressroute-how-to-set-up-azure-public-peering-for-your-expressroute-circuit)
> * [Article - Peering Microsoft](expressroute-circuit-peerings.md#microsoftpeering)
>

Cet article est conçu pour vous aider à créer et à gérer la configuration du routage de peering public d’un circuit ExpressRoute. Vous pouvez également mettre à jour, supprimer et déprovisionner des peerings, ainsi qu’en vérifier l’état. Cet article s’applique aux circuits Resource Manager qui ont été créés avant que le peering public soit déprécié. Si vous disposez déjà d’un circuit (créé avant la dépréciation du peering public), vous pouvez gérer/configurer le peering public à l’aide d’[Azure PowerShell](#powershell), d’[Azure CLI](#cli) et du [portail Azure](#portal).

>[!NOTE]
>Le peering public Azure est déprécié. Vous ne pouvez pas créer de peering public sur les nouveaux circuits ExpressRoute. Si vous avez un nouveau circuit ExpressRoute, utilisez à la place le [peering Microsoft](expressroute-circuit-peerings.md#microsoftpeering) pour vos services Azure.
>

## <a name="connectivity"></a>Connectivité

La connectivité est toujours initiée de votre réseau étendu vers les services Microsoft Azure. Ces derniers ne sont pas en mesure d’initier des connexions à votre réseau via ce domaine de routage. Si votre circuit ExpressRoute est activé pour le peering public Azure, vous pouvez accéder aux [plages d’adresses IP publiques dans Azure](../virtual-network/public-ip-addresses.md#public-ip-addresses) sur le circuit.

Une fois le peering public activé, vous êtes en mesure de vous connecter à presque tous les services Azure. Nous ne vous permettons pas de sélectionner les services pour lesquels nous publions les itinéraires.

* Les services tels que Stockage Azure, SQL Database et Sites web sont proposés sur des adresses IP publiques.
* Via le domaine de routage de peering publié, en privé, vous pouvez vous connecter à des services hébergés sur des adresses IP publiques (notamment les adresses IP virtuelles de vos services cloud).
* Vous pouvez connecter le domaine de peering public à votre zone DMZ et vous connecter à tous les services Azure sur leurs adresses IP publiques à partir de votre réseau étendu, sans avoir à vous connecter via Internet.

## <a name="services"></a><a name="services"></a>Services

Cette section présente les services disponibles via le peering public. Étant donné que le peering public est déprécié, il n’est pas prévu d’ajouter de nouveaux services ou des services supplémentaires au peering public. Si vous utilisez le peering public et que le service que vous souhaitez utiliser est pris en charge uniquement sur le peering Microsoft, vous devez passer au peering Microsoft. Pour obtenir la liste des services pris en charge, consultez [Peering Microsoft](expressroute-faqs.md#microsoft-peering).

**Pris en charge :**

* Power BI
* La plupart des services Azure sont pris en charge. Vérifiez directement auprès du service que vous souhaitez utiliser s’il est pris en charge.

**Non pris en charge :**
  * CDN
  * Azure Front Door
  * Serveur Multi-Factor Authentication (hérité)
  * Traffic Manager

Pour valider la disponibilité d’un service spécifique, vous pouvez consulter la documentation de ce service pour voir si une plage réservée est publiée pour ce service. Recherchez ensuite les plages d’adresses IP du service cible et comparez-les aux plages listées dans [Plages d’adresses IP Azure et balises de service - Fichier XML cloud public](https://www.microsoft.com/download/details.aspx?id=56519). Vous pouvez également ouvrir un ticket de support pour le service en question afin de clarifier le problème.

## <a name="peering-comparison"></a><a name="compare"></a>Comparaison de peerings

[!INCLUDE [peering comparison](../../includes/expressroute-peering-comparison.md)]

> [!NOTE]
> Le peering public Azure possède une adresse IP NAT associée à chaque session BGP. Pour plus de deux adresses IP NAT, optez pour le peering Microsoft. Le peering Microsoft vous permet de configurer vos propres répartitions NAT, ainsi que d'utiliser des filtres de routage pour des publications de préfixe sélectif. Pour plus d’informations, consultez [Passer au peering Microsoft](./how-to-move-peering.md).
>

## <a name="custom-route-filters"></a>Filtres d’itinéraires personnalisés

Vous pouvez définir des filtres d’itinéraires personnalisés au sein de votre réseau pour utiliser uniquement les itinéraires dont vous avez besoin. Reportez-vous à la page [Routage](expressroute-routing.md) pour plus d'informations sur la configuration du routage.

## <a name="azure-powershell-steps"></a><a name="powershell"></a>Étapes dans Azure PowerShell


[!INCLUDE [CloudShell](../../includes/expressroute-cloudshell-powershell-about.md)]

Étant donné que le peering public est déprécié, vous ne pouvez pas configurer le peering public sur un nouveau circuit ExpressRoute.

1. Vérifiez que vous disposez d’un circuit ExpressRoute qui est provisionné et activé. Consultez l’exemple qui suit :

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
2. Configurez le peering public Azure pour le circuit. Assurez-vous que vous disposez des informations suivantes avant de continuer.

   * Un sous-réseau /30 pour le lien principal. Ce doit être un préfixe IPv4 public valide.
   * Un sous-réseau /30 pour le lien secondaire. Ce doit être un préfixe IPv4 public valide.
   * Un ID VLAN valide pour établir ce peering. Assurez-vous qu'aucun autre peering sur le circuit n'utilise le même ID VLAN.
   * Un numéro AS pour le peering. Vous pouvez utiliser des numéros à 2 et 4 octets.
   * Facultatif :
   * Un hachage MD5 si vous choisissez d’en utiliser un.

   Exécutez l’exemple suivant pour configurer le peering public Azure pour votre circuit :

   ```azurepowershell-interactive
   Add-AzExpressRouteCircuitPeeringConfig -Name "AzurePublicPeering" -ExpressRouteCircuit $ckt -PeeringType AzurePublicPeering -PeerASN 100 -PrimaryPeerAddressPrefix "12.0.0.0/30" -SecondaryPeerAddressPrefix "12.0.0.4/30" -VlanId 100

   Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
   ```

   Si vous choisissez d’utiliser un hachage MD5, utilisez l’exemple suivant :

   ```azurepowershell-interactive
   Add-AzExpressRouteCircuitPeeringConfig -Name "AzurePublicPeering" -ExpressRouteCircuit $ckt -PeeringType AzurePublicPeering -PeerASN 100 -PrimaryPeerAddressPrefix "12.0.0.0/30" -SecondaryPeerAddressPrefix "12.0.0.4/30" -VlanId 100  -SharedKey "A1B2C3D4"

   Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
   ```

   > [!IMPORTANT]
   > Veillez à spécifier votre numéro AS comme ASN de peering et non pas comme ASN client.
   > 
   >

### <a name="to-get-azure-public-peering-details"></a><a name="getpublic"></a>Pour obtenir les détails d’un peering public Azure

Vous pouvez obtenir des détails sur la configuration à l’aide de l’applet de commande suivante :

```azurepowershell-interactive
  $ckt = Get-AzExpressRouteCircuit -Name "ExpressRouteARMCircuit" -ResourceGroupName "ExpressRouteResourceGroup"

  Get-AzExpressRouteCircuitPeeringConfig -Name "AzurePublicPeering" -Circuit $ckt
  ```

### <a name="to-update-azure-public-peering-configuration"></a><a name="updatepublic"></a>Pour mettre à jour la configuration de peering public Azure

Vous pouvez mettre à jour toute partie de la configuration à l’aide de l’exemple suivant : Dans cet exemple, l’ID VLAN du circuit est mis à jour de 200 à 600.

```azurepowershell-interactive
Set-AzExpressRouteCircuitPeeringConfig  -Name "AzurePublicPeering" -ExpressRouteCircuit $ckt -PeeringType AzurePublicPeering -PeerASN 100 -PrimaryPeerAddressPrefix "123.0.0.0/30" -SecondaryPeerAddressPrefix "123.0.0.4/30" -VlanId 600

Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
```

### <a name="to-delete-azure-public-peering"></a><a name="deletepublic"></a>Pour supprimer un peering public Azure

Vous pouvez supprimer votre configuration de peering en exécutant l’exemple suivant :

```azurepowershell-interactive
Remove-AzExpressRouteCircuitPeeringConfig -Name "AzurePublicPeering" -ExpressRouteCircuit $ckt
Set-AzExpressRouteCircuit -ExpressRouteCircuit $ckt
```

## <a name="azure-cli-steps"></a><a name="cli"></a>Étapes dans Azure CLI


[!INCLUDE [CloudShell](../../includes/expressroute-cloudshell-powershell-about.md)]

1. Vérifiez que le circuit ExpressRoute est approvisionné et activé. Consultez l’exemple qui suit :

   ```azurecli-interactive
   az network express-route list
   ```

   La réponse ressemble à ce qui suit :

   ```output
   "allowClassicOperations": false,
   "authorizations": [],
   "circuitProvisioningState": "Enabled",
   "etag": "W/\"1262c492-ffef-4a63-95a8-a6002736b8c4\"",
   "gatewayManagerEtag": null,
   "id": "/subscriptions/81ab786c-56eb-4a4d-bb5f-f60329772466/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/MyCircuit",
   "location": "westus",
   "name": "MyCircuit",
   "peerings": [],
   "provisioningState": "Succeeded",
   "resourceGroup": "ExpressRouteResourceGroup",
   "serviceKey": "1d05cf70-1db5-419f-ad86-1ca62c3c125b",
   "serviceProviderNotes": null,
   "serviceProviderProperties": {
    "bandwidthInMbps": 200,
    "peeringLocation": "Silicon Valley",
    "serviceProviderName": "Equinix"
   },
   "serviceProviderProvisioningState": "Provisioned",
   "sku": {
    "family": "UnlimitedData",
    "name": "Standard_MeteredData",
    "tier": "Standard"
   },
   "tags": null,
   "type": "Microsoft.Network/expressRouteCircuits]
   ```

2. Configurez le peering public Azure pour le circuit. Assurez-vous que vous disposez des informations suivantes avant de continuer.

   * Un sous-réseau /30 pour le lien principal. Ce doit être un préfixe IPv4 public valide.
   * Un sous-réseau /30 pour le lien secondaire. Ce doit être un préfixe IPv4 public valide.
   * Un ID VLAN valide pour établir ce peering. Assurez-vous qu'aucun autre peering sur le circuit n'utilise le même ID VLAN.
   * Un numéro AS pour le peering. Vous pouvez utiliser des numéros à 2 et 4 octets.
   * **Facultatif :** un hachage MD5 si vous choisissez d’en utiliser un.

   Exécutez l’exemple suivant pour configurer le peering public Azure pour votre circuit :

   ```azurecli-interactive
   az network express-route peering create --circuit-name MyCircuit --peer-asn 100 --primary-peer-subnet 12.0.0.0/30 -g ExpressRouteResourceGroup --secondary-peer-subnet 12.0.0.4/30 --vlan-id 200 --peering-type AzurePublicPeering
   ```

   Si vous choisissez d’utiliser un hachage MD5, utilisez l’exemple suivant :

   ```azurecli-interactive
   az network express-route peering create --circuit-name MyCircuit --peer-asn 100 --primary-peer-subnet 12.0.0.0/30 -g ExpressRouteResourceGroup --secondary-peer-subnet 12.0.0.4/30 --vlan-id 200 --peering-type AzurePublicPeering --SharedKey "A1B2C3D4"
   ```

   > [!IMPORTANT]
   > Veillez à spécifier votre numéro AS comme ASN de peering et non pas comme ASN client.

### <a name="to-view-azure-public-peering-details"></a><a name="getpublic"></a>Pour afficher les détails d’un peering public Azure

Vous pouvez obtenir les détails de la configuration à l’aide de l’exemple suivant :

```azurecli
az network express-route peering show -g ExpressRouteResourceGroup --circuit-name MyCircuit --name AzurePublicPeering
```

Le résultat ressemble à l’exemple suivant :

```output
{
  "azureAsn": 12076,
  "etag": "W/\"2e97be83-a684-4f29-bf3c-96191e270666\"",
  "gatewayManagerEtag": "18",
  "id": "/subscriptions/9a0c2943-e0c2-4608-876c-e0ddffd1211b/resourceGroups/ExpressRouteResourceGroup/providers/Microsoft.Network/expressRouteCircuits/MyCircuit/peerings/AzurePublicPeering",
  "lastModifiedBy": "Customer",
  "microsoftPeeringConfig": null,
  "name": "AzurePublicPeering",
  "peerAsn": 7671,
  "peeringType": "AzurePublicPeering",
  "primaryAzurePort": "",
  "primaryPeerAddressPrefix": "",
  "provisioningState": "Succeeded",
  "resourceGroup": "ExpressRouteResourceGroup",
  "routeFilter": null,
  "secondaryAzurePort": "",
  "secondaryPeerAddressPrefix": "",
  "sharedKey": null,
  "state": "Enabled",
  "stats": null,
  "vlanId": 100
}
```

### <a name="to-update-azure-public-peering-configuration"></a><a name="updatepublic"></a>Pour mettre à jour la configuration de peering public Azure

Vous pouvez mettre à jour toute partie de la configuration à l’aide de l’exemple suivant : Dans cet exemple, l’ID VLAN du circuit est mis à jour de 200 à 600.

```azurecli-interactive
az network express-route peering update --vlan-id 600 -g ExpressRouteResourceGroup --circuit-name MyCircuit --name AzurePublicPeering
```

### <a name="to-delete-azure-public-peering"></a><a name="deletepublic"></a>Pour supprimer un peering public Azure

Vous pouvez supprimer votre configuration de peering en exécutant l’exemple suivant :

```azurecli-interactive
az network express-route peering delete -g ExpressRouteResourceGroup --circuit-name MyCircuit --name AzurePublicPeering
```

## <a name="azure-portal-steps"></a><a name="portal"></a>Étapes dans le portail Azure

Pour configurer le peering, utilisez les étapes PowerShell ou CLI contenues dans cet article. Pour gérer un peering, vous pouvez vous référer aux sections ci-dessous. Pour référence, ces étapes ressemblent à la gestion d’un [peering Microsoft dans le portail](expressroute-howto-routing-portal-resource-manager.md#msft).

### <a name="to-view-azure-public-peering-details"></a><a name="get"></a>Pour afficher les détails d’un peering public Azure

Affichez les propriétés du peering public Azure en sélectionnant le peering dans le portail.

### <a name="to-update-azure-public-peering-configuration"></a><a name="update"></a>Pour mettre à jour la configuration de peering public Azure

Sélectionnez la ligne du peering, puis modifiez les propriétés de peering.

### <a name="to-delete-azure-public-peering"></a><a name="delete"></a>Pour supprimer un peering public Azure

Supprimez votre configuration de peering en sélectionnant l’icône Supprimer.

## <a name="next-steps"></a>Étapes suivantes

Ensuite, [liez un réseau virtuel à un circuit ExpressRoute](expressroute-howto-linkvnet-arm.md).

* Pour plus d'informations sur les workflows ExpressRoute, consultez [Workflows ExpressRoute](expressroute-workflows.md).
* Pour plus d’informations sur le peering du circuit, consultez [Circuits ExpressRoute et domaines de routage](expressroute-circuit-peerings.md).
* Pour plus d’informations sur l’utilisation des réseaux virtuels, consultez la page [Présentation du réseau virtuel](../virtual-network/virtual-networks-overview.md).