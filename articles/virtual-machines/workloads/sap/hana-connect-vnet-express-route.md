---
title: Configuration de la connectivité d’un réseau virtuel à SAP HANA sur Azure (grandes instances) | Microsoft Docs
description: Configuration de la connectivité d’un réseau virtuel à SAP HANA sur Azure (grandes instances).
services: virtual-machines-linux
documentationcenter: ''
author: msjuergent
manager: bburns
editor: ''
ms.service: virtual-machines-linux
ms.subservice: workloads
ms.topic: article
ms.tgt_pltfrm: vm-linux
ms.workload: infrastructure
ms.date: 05/25/2019
ms.author: juergent
ms.custom: H1Hack27Feb2017
ms.openlocfilehash: c5a8a8157721f34abf7761a85febc7bcea3abb88
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94967837"
---
# <a name="connect-a-virtual-network-to-hana-large-instances"></a>Connecter un réseau virtuel à de grandes instances HANA

Après avoir créé un réseau virtuel Azure, vous pouvez connecter ce réseau à SAP HANA sur de grandes instances Azure. Créez une passerelle Azure ExpressRoute sur le réseau virtuel. Cette passerelle vous permet de lier le réseau virtuel au circuit ExpressRoute qui se connecte au locataire du client sur le tampon de Grande instance HANA.

> [!NOTE] 
> Cette étape peut prendre jusqu’à 30 minutes. La nouvelle passerelle est créée dans l’abonnement Azure désigné, puis connectée au réseau virtuel Azure spécifié.

[!INCLUDE [updated-for-az](../../../../includes/updated-for-az.md)]

S’il existe déjà une passerelle, vérifiez qu’il s’agit d’une passerelle ExpressRoute. S’il ne s’agit pas d’une passerelle ExpressRoute, supprimez la passerelle et recréez-la comme passerelle ExpressRoute. Si une passerelle ExpressRoute est déjà établie, consultez la section « Lier des réseaux virtuels » de cet article. 

- Utilisez soit le [portail Azure](https://portal.azure.com/), soit PowerShell pour créer une passerelle VPN ExpressRoute connectée à votre réseau virtuel.
  - Si vous utilisez le portail Azure, ajoutez une nouvelle **passerelle de réseau virtuel**, puis sélectionnez **ExpressRoute** comme type de passerelle.
  - Si vous utilisez PowerShell, commencez par télécharger et par utiliser le dernier [SDK Azure PowerShell](https://azure.microsoft.com/downloads/). 
 
Les commandes suivantes créent une passerelle ExpressRoute. Le texte précédé par un _$_ correspond à des variables définies par l’utilisateur qui doivent être mises à jour avec vos propres informations.

```powershell
# These Values should already exist, update to match your environment
$myAzureRegion = "eastus"
$myGroupName = "SAP-East-Coast"
$myVNetName = "VNet01"

# These values are used to create the gateway, update for how you wish the GW components to be named
$myGWName = "VNet01GW"
$myGWConfig = "VNet01GWConfig"
$myGWPIPName = "VNet01GWPIP"
$myGWSku = "HighPerformance" # Supported values for HANA large instances are: HighPerformance or UltraPerformance

# These Commands create the Public IP and ExpressRoute Gateway
$vnet = Get-AzVirtualNetwork -Name $myVNetName -ResourceGroupName $myGroupName
$subnet = Get-AzVirtualNetworkSubnetConfig -Name 'GatewaySubnet' -VirtualNetwork $vnet
New-AzPublicIpAddress -Name $myGWPIPName -ResourceGroupName $myGroupName `
-Location $myAzureRegion -AllocationMethod Dynamic
$gwpip = Get-AzPublicIpAddress -Name $myGWPIPName -ResourceGroupName $myGroupName
$gwipconfig = New-AzVirtualNetworkGatewayIpConfig -Name $myGWConfig -SubnetId $subnet.Id `
-PublicIpAddressId $gwpip.Id

New-AzVirtualNetworkGateway -Name $myGWName -ResourceGroupName $myGroupName -Location $myAzureRegion `
-IpConfigurations $gwipconfig -GatewayType ExpressRoute `
-GatewaySku $myGWSku -VpnType PolicyBased -EnableBgp $true
```

Dans cet exemple, la référence SKU de passerelle HighPerformance a été utilisée. Vous pouvez opter pour HighPerformance ou pour UltraPerformance, les seules références SKU de passerelle prises en charge pour SAP HANA sur Azure (grandes instances).

> [!IMPORTANT]
> Pour les grandes instances HANA de classe Type II, vous devez utiliser la référence SKU de la passerelle UltraPerformance.

## <a name="link-virtual-networks"></a>Lier des réseaux virtuels

Le réseau virtuel Azure a maintenant une passerelle ExpressRoute. Utilisez les informations d’autorisation fournies par Microsoft pour connecter la passerelle ExpressRoute au circuit de grandes instances ExpressRoute SAP HANA. Vous pouvez vous connecter à l’aide du portail Azure ou de PowerShell. Les instructions PowerShell sont les suivantes. 

Exécutez les commandes suivantes pour chaque passerelle ExpressRoute, en utilisant un AuthGUID différent pour chaque connexion. Les deux premières entrées dans le script suivant proviennent des informations fournies par Microsoft. En outre, AuthGUID est spécifique à chaque réseau virtuel et à sa passerelle. Si vous souhaitez ajouter un autre réseau virtuel Azure, vous devez obtenir un autre AuthID pour votre circuit ExpressRoute, qui connecte les Grandes instances HANA dans Azure à partir de Microsoft. 

```powershell
# Populate with information provided by Microsoft Onboarding team
$PeerID = "/subscriptions/9cb43037-9195-4420-a798-f87681a0e380/resourceGroups/Customer-USE-Circuits/providers/Microsoft.Network/expressRouteCircuits/Customer-USE01"
$AuthGUID = "76d40466-c458-4d14-adcf-3d1b56d1cd61"

# Your ExpressRoute Gateway information
$myGroupName = "SAP-East-Coast"
$myGWName = "VNet01GW"
$myGWLocation = "East US"

# Define the name for your connection
$myConnectionName = "VNet01GWConnection"

# Create a new connection between the ER Circuit and your Gateway using the Authorization
$gw = Get-AzVirtualNetworkGateway -Name $myGWName -ResourceGroupName $myGroupName
    
New-AzVirtualNetworkGatewayConnection -Name $myConnectionName `
-ResourceGroupName $myGroupName -Location $myGWLocation -VirtualNetworkGateway1 $gw `
-PeerId $PeerID -ConnectionType ExpressRoute -AuthorizationKey $AuthGUID -ExpressRouteGatewayBypass
```

> [!NOTE]
> Le dernier paramètre de la commande New-AzVirtualNetworkGatewayConnection, **ExpressRouteGatewayBypass** est un nouveau paramètre qui active ExpressRoute Fast Path. Il s’agit d’une fonctionnalité qui réduit la latence réseau entre vos unités de Grande instance HANA et les machines virtuelles Azure. La fonctionnalité a été ajoutée en mai 2019. Pour plus d’informations, voir l’article [Architecture réseau de SAP HANA (grandes instances)](./hana-network-architecture.md). Avant d’exécuter les commandes, assurez-vous que vous exécutez la dernière version des cmdlets PowerShell.

Pour connecter la passerelle à plusieurs circuits ExpressRoute associés à votre abonnement, vous devrez peut-être exécuter plusieurs fois cette étape. Par exemple, vous allez probablement connecter la même passerelle de réseau virtuel au circuit ExpressRoute qui relie le réseau virtuel à votre réseau local.

## <a name="applying-expressroute-fast-path-to-existing-hana-large-instance-expressroute-circuits"></a>Application d’ExpressRoute Fast Path à des circuits ExpressRoute de Grande instance HANA existants
Jusqu’à présent, la documentation expliquait comment connecter un nouveau circuit ExpressRoute créé avec un déploiement de Grande instance HANA à une passerelle Azure ExpressRoute de l’un de vos réseaux virtuels Azure. Mais les circuits ExpressRoute de nombreux clients sont déjà configurés, et leurs réseaux virtuels déjà connectés à de Grandes instances HANA. Le nouveau ExpressRoute Fast Path réduisant la latence du réseau, il est recommandé d’appliquer la modification pour utiliser cette fonctionnalité. Les commandes pour se connecter à un nouveau circuit ExpressRoute et modifier un circuit ExpressRoute existant sont les mêmes. Par conséquent, vous devez exécuter cette séquence de commandes PowerShell pour modifier un circuit existant afin de l’utiliser 

```powershell
# Populate with information provided by Microsoft Onboarding team
$PeerID = "/subscriptions/9cb43037-9195-4420-a798-f87681a0e380/resourceGroups/Customer-USE-Circuits/providers/Microsoft.Network/expressRouteCircuits/Customer-USE01"
$AuthGUID = "76d40466-c458-4d14-adcf-3d1b56d1cd61"

# Your ExpressRoute Gateway information
$myGroupName = "SAP-East-Coast"
$myGWName = "VNet01GW"
$myGWLocation = "East US"

# Define the name for your connection
$myConnectionName = "VNet01GWConnection"

# Create a new connection between the ER Circuit and your Gateway using the Authorization
$gw = Get-AzVirtualNetworkGateway -Name $myGWName -ResourceGroupName $myGroupName
    
New-AzVirtualNetworkGatewayConnection -Name $myConnectionName `
-ResourceGroupName $myGroupName -Location $myGWLocation -VirtualNetworkGateway1 $gw `
-PeerId $PeerID -ConnectionType ExpressRoute -AuthorizationKey $AuthGUID -ExpressRouteGatewayBypass
```

Il est important que vous ajoutiez le dernier paramètre tel qu’affiché ci-dessus pour activer la fonctionnalité ExpressRoute Fast Path


## <a name="expressroute-global-reach"></a>Service Global Reach d’ExpressRoute
Vous pouvez activer le service Global Reach pour deux scénarios :

 - Réplication de système HANA sans proxy ou pare-feu supplémentaires
 - Copie de sauvegardes entre des unités de Grande instance HANA situées dans deux régions différentes pour effectuer des copies ou actualisations du système

Vous devez prendre en compte les aspects suivants :

- Vous devez fournir une plage d’adresses d’un espace d’adressage /29. Cette plage d’adresses ne peut pas couvrir d’autres plages d’espace d’adressage utilisées précédemment en connectant de grandes instances HANA à Azure, ou des plages d’adresses IP utilisées ailleurs dans Azure ou localement.
- Il existe une limitation concernant les ASN (numéros de système autonome) utilisables pour annoncer vos itinéraires locaux vers des Grandes instances HANA. Votre système local ne doit pas publier d’itinéraires avec des ASN privés dans la plage de 65000 à 65020 ou 65515. 
- Pour le scénario de connexion d’accès direct local à de grandes instances HANA, vous devez calculer un tarif pour le circuit qui vous connecte à Azure. Pour connaître les prix, voir [Composant additionnel Global Reach](https://azure.microsoft.com/pricing/details/expressroute/).

Pour que l’un des scénarios ou les deux soient appliqués à votre déploiement, adresses une demande de support à Azure en procédant de la manière décrite dans [Ouvrir une demande de support pour les Grandes instances HANA](./hana-li-portal.md#open-a-support-request-for-hana-large-instances).

Les données nécessaires et les mots-clés que vous devez utiliser pour que Microsoft puisse router et exécuter votre demande ressemblent à ceci :

- Service : Grande instance SAP HANA
- Type de problème : Configuration et installation
- Sous-type de problème : Mon problème n’est pas répertorié ci-dessus
- Objet « Modifier mon réseau – ajouter Global Reach »
- Détails : Ajouter Global Reach à une grande instance HANA pour un locataire de grande instance HANA ou Ajouter Global Reach au système local pour un locataire de grande instance HANA.
- Détails supplémentaires sur le cas de Grande instance HANA pour un locataire de Grande instance HANA : Vous devez définir les **deux régions Azure** où se trouvent les deux locataires à connecter **ET** envoyer la **plage d’adresses IP /29**
- Détails supplémentaires sur le cas de système local pour un locataire de grande instance HANA : Vous devez définir la **Région Azure** sur laquelle est déployé le locataire de Grande instance HANA auquel vous souhaitez vous connecter. De plus, vous devez fournir les paramètres **Auth GUID** (GUID Auth) et **Circuit Peer ID** (ID homologue de circuit) que vous avez reçus lors de l’établissement de votre circuit ExpressRoute entre le système local et Azure. Vous devez également nommer votre **ASN**. La dernier élément à fournir est une **plage d’adresses IP /29** pour le service Global Reach d’ExpressRoute.

> [!NOTE]
> Si vous souhaitez que les deux cas soient gérés, vous devez fournir deux plages d’adresses IP /29 différentes qui ne chevauchent pas d’autre plage d’adresses IP utilisée jusqu’à présent. 




## <a name="next-steps"></a>Étapes suivantes

- [Configuration réseau requise supplémentaire pour HLI](hana-additional-network-requirements.md)
