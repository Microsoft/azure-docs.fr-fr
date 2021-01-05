---
title: 'Passerelle VPN Azure : Configurer BGP : PowerShell'
description: Cet article vous guide dans la configuration de BGP avec des passerelles VPN Azure à l’aide d’Azure Resource Manager et de PowerShell.
services: vpn-gateway
author: yushwang
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/02/2020
ms.author: yushwang
ms.custom: devx-track-azurepowershell
ms.openlocfilehash: 8573d9e55299382392927b532966a6e6fdd8c439
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94659758"
---
# <a name="how-to-configure-bgp-on-azure-vpn-gateways-using-powershell"></a>Configurer BGP sur des passerelles VPN Azure à l’aide de PowerShell
Cet article vous guide pas à pas dans l’activation de BGP sur une connexion VPN de site à site (S2S) et une connexion de réseau virtuel à réseau virtuel, à l’aide du modèle de déploiement de Resource Manager et de PowerShell.



## <a name="about-bgp"></a>À propos du protocole BGP
BGP est le protocole de routage standard couramment utilisé sur Internet pour échanger des informations de routage et d’accessibilité entre plusieurs réseaux. Il permet aux passerelles VPN Azure et à vos périphériques VPN locaux (appelés voisins ou homologues BGP) d’échanger des « itinéraires » informant les deux passerelles sur la disponibilité et l’accessibilité de ces préfixes à travers les passerelles ou routeurs impliqués. Le protocole BGP assure également le routage de transit entre plusieurs réseaux en propageant les itinéraires qu’une passerelle BGP obtient d’un homologue BGP à tous les autres homologues BGP.

Pour plus d’informations sur les avantages de BGP et les exigences techniques et considérations d’utilisation de BGP, consultez [Vue d’ensemble du protocole BGP avec les passerelles VPN Azure](vpn-gateway-bgp-overview.md).

## <a name="getting-started-with-bgp-on-azure-vpn-gateways"></a>Prise en main de BGP sur les passerelles VPN Azure

Cet article détaille les étapes permettant d’effectuer les tâches suivantes :

* [Partie 1 - Activer BGP sur votre passerelle VPN Azure](#enablebgp)
* Partie 2 - Établir une connexion intersite avec BGP
* [Partie 3 - Établir une connexion de réseau virtuel à réseau virtuel avec BGP](#v2vbgp)

Chaque partie des instructions constitue un bloc de base pour activer BGP dans votre connectivité réseau. Si vous terminez ces trois parties, vous générez la topologie comme sur le diagramme suivant :

![Topologie BGP](./media/vpn-gateway-bgp-resource-manager-ps/bgp-crosspremv2v.png)

Vous pouvez les combiner pour créer un réseau de transit plus complexe, à plusieurs tronçons, qui répond à vos besoins.

## <a name="part-1---configure-bgp-on-the-azure-vpn-gateway"></a><a name ="enablebgp"></a>Partie 1 – Configurer BGP sur la passerelle VPN Azure
Les étapes suivantes configurent les paramètres BGP de la passerelle VPN Azure comme indiqué dans le diagramme ci-dessous :

![Passerelle BGP](./media/vpn-gateway-bgp-resource-manager-ps/bgp-gateway.png)

### <a name="before-you-begin"></a>Avant de commencer
* Assurez-vous de disposer d’un abonnement Azure. Si vous ne disposez pas déjà d’un abonnement Azure, vous pouvez activer vos [avantages abonnés MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) ou créer un [compte gratuit](https://azure.microsoft.com/pricing/free-trial/).
* Installez les applets de commande PowerShell Azure Resource Manager. Pour plus d’informations sur l’installation des applets de commande PowerShell, consultez [Installation et configuration d’Azure PowerShell](/powershell/azure/). 

### <a name="step-1---create-and-configure-vnet1"></a>Étape 1 – Créer et configurer le réseau virtuel VNet1
#### <a name="1-declare-your-variables"></a>1. Déclarer vos variables
Dans cet exercice, nous allons commencer par déclarer les variables. L’exemple suivant déclare les variables avec les valeurs de cet exercice. Veillez à les remplacer par vos valeurs lors de la configuration dans un contexte de production. Vous pouvez utiliser ces variables si vous exécutez la procédure pour vous familiariser avec ce type de configuration. Modifiez les variables, puis copiez et collez-les dans la console PowerShell.

```powershell
$Sub1 = "Replace_With_Your_Subscription_Name"
$RG1 = "TestBGPRG1"
$Location1 = "East US"
$VNetName1 = "TestVNet1"
$FESubName1 = "FrontEnd"
$BESubName1 = "Backend"
$GWSubName1 = "GatewaySubnet"
$VNetPrefix11 = "10.11.0.0/16"
$VNetPrefix12 = "10.12.0.0/16"
$FESubPrefix1 = "10.11.0.0/24"
$BESubPrefix1 = "10.12.0.0/24"
$GWSubPrefix1 = "10.12.255.0/27"
$VNet1ASN = 65010
$DNS1 = "8.8.8.8"
$GWName1 = "VNet1GW"
$GWIPName1 = "VNet1GWIP"
$GWIPconfName1 = "gwipconf1"
$Connection12 = "VNet1toVNet2"
$Connection15 = "VNet1toSite5"
```

#### <a name="2-connect-to-your-subscription-and-create-a-new-resource-group"></a>2. Se connecter à votre abonnement et créer un groupe de ressources
Pour utiliser les applets de commande Resource Manager, passez au mode PowerShell. Pour plus d’informations, consultez la page [Utilisation de Windows PowerShell avec Resource Manager](../azure-resource-manager/management/manage-resources-powershell.md).

Ouvrez la console PowerShell et connectez-vous à votre compte. Utilisez l’exemple suivant pour faciliter votre connexion :

```powershell
Connect-AzAccount
Select-AzSubscription -SubscriptionName $Sub1
New-AzResourceGroup -Name $RG1 -Location $Location1
```

#### <a name="3-create-testvnet1"></a>3. Créer TestVNet1
L’exemple suivant crée un réseau virtuel nommé TestVNet1 et trois sous-réseaux nommés GatewaySubnet, FrontEnd et Backend. Lorsque vous remplacez les valeurs, pensez à toujours nommer votre sous-réseau de passerelle « GatewaySubnet ». Si vous le nommez autrement, la création de votre passerelle échoue.

```powershell
$fesub1 = New-AzVirtualNetworkSubnetConfig -Name $FESubName1 -AddressPrefix $FESubPrefix1
$besub1 = New-AzVirtualNetworkSubnetConfig -Name $BESubName1 -AddressPrefix $BESubPrefix1
$gwsub1 = New-AzVirtualNetworkSubnetConfig -Name $GWSubName1 -AddressPrefix $GWSubPrefix1

New-AzVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1 -Location $Location1 -AddressPrefix $VNetPrefix11,$VNetPrefix12 -Subnet $fesub1,$besub1,$gwsub1
```

### <a name="step-2---create-the-vpn-gateway-for-testvnet1-with-bgp-parameters"></a>Étape 2 – Créer la passerelle VPN de TestVNet1 avec les paramètres BGP
#### <a name="1-create-the-ip-and-subnet-configurations"></a>1. Créer les configurations IP et de sous-réseau
Demandez l’allocation d’une adresse IP publique à la passerelle que vous allez créer pour votre réseau virtuel. Vous allez également définir les configurations IP et sous-réseau requises.

```powershell
$gwpip1 = New-AzPublicIpAddress -Name $GWIPName1 -ResourceGroupName $RG1 -Location $Location1 -AllocationMethod Dynamic

$vnet1 = Get-AzVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1
$subnet1 = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet1
$gwipconf1 = New-AzVirtualNetworkGatewayIpConfig -Name $GWIPconfName1 -Subnet $subnet1 -PublicIpAddress $gwpip1
```

#### <a name="2-create-the-vpn-gateway-with-the-as-number"></a>2. Créer la passerelle VPN avec le numéro AS
Créez la passerelle de réseau virtuel pour TestVNet1. BGP requiert une passerelle VPN basée sur l’itinéraire ainsi que le paramètre d’ajout, - Asn, pour définir l’ASN (numéro AS) de TestVNet1. Si vous ne définissez pas le paramètre ASN, le numéro ASN 65515 est attribué. La création d’une passerelle peut prendre un certain temps (30 minutes ou plus).

```powershell
New-AzVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1 -Location $Location1 -IpConfigurations $gwipconf1 -GatewayType Vpn -VpnType RouteBased -GatewaySku VpnGw1 -Asn $VNet1ASN
```

#### <a name="3-obtain-the-azure-bgp-peer-ip-address"></a>3. Obtenir l’adresse IP de l’homologue BGP Azure
Une fois la passerelle créée, vous devez obtenir l’adresse IP de l’homologue BGP sur la passerelle VPN Azure. Cette adresse est nécessaire pour configurer la passerelle VPN Azure comme un homologue BGP pour vos périphériques VPN locaux.

```powershell
$vnet1gw = Get-AzVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1
$vnet1gw.BgpSettingsText
```

La dernière commande affiche les configurations BGP correspondantes sur la passerelle VPN Azure ; par exemple :

```powershell
$vnet1gw.BgpSettingsText
{
    "Asn": 65010,
    "BgpPeeringAddress": "10.12.255.30",
    "PeerWeight": 0
}
```

Une fois la passerelle créée, vous pouvez l’utiliser pour établir une connexion intersite ou de réseau virtuel à réseau virtuel avec BGP. Les sections suivantes détaillent les étapes à effectuer pour terminer l’exercice.

## <a name="part-2---establish-a-cross-premises-connection-with-bgp"></a><a name ="crossprembbgp"></a>Partie 2 - Établir une connexion intersite avec BGP

Pour établir une connexion intersite, vous devez créer une passerelle de réseau local pour représenter votre périphérique VPN local, et une connexion entre la passerelle VPN et la passerelle du réseau local. Bien qu’il existe des articles qui vous guident tout au long de ces étapes, cet article contient les propriétés supplémentaires nécessaires pour spécifier les paramètres de configuration BGP.

![BGP pour une connexion intersite](./media/vpn-gateway-bgp-resource-manager-ps/bgp-crossprem.png)

Avant de poursuivre, vérifiez que vous avez terminé la [Partie 1](#enablebgp) de cet exercice.

### <a name="step-1---create-and-configure-the-local-network-gateway"></a>Étape 1 - Créer et configurer la passerelle de réseau local

#### <a name="1-declare-your-variables"></a>1. Déclarer vos variables

Cet exercice continue à générer la configuration représentée dans le diagramme. Veillez à remplacer les valeurs par celles que vous souhaitez utiliser pour votre configuration.

```powershell
$RG5 = "TestBGPRG5"
$Location5 = "East US 2"
$LNGName5 = "Site5"
$LNGPrefix50 = "10.52.255.254/32"
$LNGIP5 = "Your_VPN_Device_IP"
$LNGASN5 = 65050
$BGPPeerIP5 = "10.52.255.254"
```

Quelques points à noter concernant les paramètres de la passerelle de réseau local :

* La passerelle de réseau local peut se trouver dans les mêmes emplacement et groupe de ressources que la passerelle VPN ou dans un emplacement et un groupe de ressources différents. Cet exemple les montre dans différents groupes de ressources situés dans différents emplacements.
* Le préfixe à déclarer pour la passerelle de réseau local est l’adresse IP hôte de votre homologue BGP sur votre appareil VPN. Dans ce cas, c’est un préfixe /32 de « 10.52.255.254/32 ».
* À titre de rappel, vous devez utiliser différents ASN BGP entre vos réseaux locaux et le réseau virtuel Azure. S’ils sont identiques, vous devez modifier l’ASN de votre réseau si votre périphérique VPN local utilise déjà cet ASN pour s’homologuer avec d’autres voisins BGP.

Avant de continuer, assurez-vous que vous êtes toujours connecté à l’abonnement 1.

#### <a name="2-create-the-local-network-gateway-for-site5"></a>2. Créer la passerelle de réseau local pour le site 5

Veillez à créer le groupe de ressources (si ce n’est déjà fait) avant la passerelle de réseau local. Remarquez les deux paramètres supplémentaires pour la passerelle de réseau local : ASN et BgpPeerAddress.

```powershell
New-AzResourceGroup -Name $RG5 -Location $Location5

New-AzLocalNetworkGateway -Name $LNGName5 -ResourceGroupName $RG5 -Location $Location5 -GatewayIpAddress $LNGIP5 -AddressPrefix $LNGPrefix50 -Asn $LNGASN5 -BgpPeeringAddress $BGPPeerIP5
```

### <a name="step-2---connect-the-vnet-gateway-and-local-network-gateway"></a>Étape 2 - Connecter la passerelle de réseau virtuel et la passerelle de réseau local

#### <a name="1-get-the-two-gateways"></a>1. Obtenir les deux passerelles

```powershell
$vnet1gw = Get-AzVirtualNetworkGateway -Name $GWName1  -ResourceGroupName $RG1
$lng5gw  = Get-AzLocalNetworkGateway -Name $LNGName5 -ResourceGroupName $RG5
```

#### <a name="2-create-the-testvnet1-to-site5-connection"></a>2. Créer la connexion entre TestVNet1 et Site5

Dans cette étape, vous allez créer la connexion entre TestVNet1 et Site5. Vous devez spécifier « -EnableBGP $True » pour activer le BGP sur cette connexion. Comme nous l’avons vu, il est possible d’avoir des connexions BGP et non BGP sur la même passerelle VPN Azure. À moins que BGP ne soit activé dans la propriété de connexion, Azure n’active pas BGP sur cette connexion même si les paramètres BGP sont déjà configurés sur les deux passerelles.

```powershell
New-AzVirtualNetworkGatewayConnection -Name $Connection15 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -LocalNetworkGateway2 $lng5gw -Location $Location1 -ConnectionType IPsec -SharedKey 'AzureA1b2C3' -EnableBGP $True
```

L’exemple ci-dessous répertorie les paramètres que vous devez saisir dans la section de configuration de BGP de votre périphérique VPN local pour cet exercice :

```

- Site5 ASN            : 65050
- Site5 BGP IP         : 10.52.255.254
- Prefixes to announce : (for example) 10.51.0.0/16 and 10.52.0.0/16
- Azure VNet ASN       : 65010
- Azure VNet BGP IP    : 10.12.255.30
- Static route         : Add a route for 10.12.255.30/32, with nexthop being the VPN tunnel interface on your device
- eBGP Multihop        : Ensure the "multihop" option for eBGP is enabled on your device if needed
```

La connexion est établie après quelques minutes, et la session de peering BGP débute une fois la connexion IPsec établie.

## <a name="part-3---establish-a-vnet-to-vnet-connection-with-bgp"></a><a name ="v2vbgp"></a>Partie 3 - Établir une connexion de réseau virtuel à réseau virtuel avec BGP

Cette section ajoute une connexion de réseau virtuel à réseau virtuel avec le protocole BGP, comme illustré dans le diagramme ci-dessous :

![Diagramme montrant une connexion de réseau virtuel à réseau virtuel.](./media/vpn-gateway-bgp-resource-manager-ps/bgp-vnet2vnet.png)

Ces instructions ci-dessous sont la suite des étapes précédentes. Vous devez terminer la [Partie 1](#enablebgp) pour créer et configurer TestVNet1 et la passerelle VPN avec le protocole BGP. 

### <a name="step-1---create-testvnet2-and-the-vpn-gateway"></a>Étape 1 - Créer TestVNet2 et la passerelle VPN

Il est important de s’assurer que l’espace d’adresse IP du nouveau réseau virtuel, TestVNet2, n’empiète sur aucune de vos plages de réseau virtuel.

Dans cet exemple, les réseaux virtuels appartiennent au même abonnement. Vous pouvez configurer les connexions de réseau virtuel à réseau virtuel entre différents abonnements. Pour plus d'informations, voir [Configurer une connexion de réseau virtuel à réseau virtuel](vpn-gateway-vnet-vnet-rm-ps.md). Veillez à ajouter l’argument « -EnableBgp $True » lors de la création de connexions pour activer BGP.

#### <a name="1-declare-your-variables"></a>1. Déclarer vos variables

Veillez à remplacer les valeurs par celles que vous souhaitez utiliser pour votre configuration.

```powershell
$RG2 = "TestBGPRG2"
$Location2 = "West US"
$VNetName2 = "TestVNet2"
$FESubName2 = "FrontEnd"
$BESubName2 = "Backend"
$GWSubName2 = "GatewaySubnet"
$VNetPrefix21 = "10.21.0.0/16"
$VNetPrefix22 = "10.22.0.0/16"
$FESubPrefix2 = "10.21.0.0/24"
$BESubPrefix2 = "10.22.0.0/24"
$GWSubPrefix2 = "10.22.255.0/27"
$VNet2ASN = 65020
$DNS2 = "8.8.8.8"
$GWName2 = "VNet2GW"
$GWIPName2 = "VNet2GWIP"
$GWIPconfName2 = "gwipconf2"
$Connection21 = "VNet2toVNet1"
$Connection12 = "VNet1toVNet2"
```

#### <a name="2-create-testvnet2-in-the-new-resource-group"></a>2. Créer TestVNet2 dans le nouveau groupe de ressources

```powershell
New-AzResourceGroup -Name $RG2 -Location $Location2

$fesub2 = New-AzVirtualNetworkSubnetConfig -Name $FESubName2 -AddressPrefix $FESubPrefix2
$besub2 = New-AzVirtualNetworkSubnetConfig -Name $BESubName2 -AddressPrefix $BESubPrefix2
$gwsub2 = New-AzVirtualNetworkSubnetConfig -Name $GWSubName2 -AddressPrefix $GWSubPrefix2

New-AzVirtualNetwork -Name $VNetName2 -ResourceGroupName $RG2 -Location $Location2 -AddressPrefix $VNetPrefix21,$VNetPrefix22 -Subnet $fesub2,$besub2,$gwsub2
```

#### <a name="3-create-the-vpn-gateway-for-testvnet2-with-bgp-parameters"></a>3. Créer la passerelle VPN de TestVNet2 avec les paramètres BGP

Demandez l’allocation d’une adresse IP publique à la passerelle que vous allez créer pour votre réseau virtuel et définissez les configurations de sous-réseau et IP requises.

```powershell
$gwpip2    = New-AzPublicIpAddress -Name $GWIPName2 -ResourceGroupName $RG2 -Location $Location2 -AllocationMethod Dynamic

$vnet2     = Get-AzVirtualNetwork -Name $VNetName2 -ResourceGroupName $RG2
$subnet2   = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet2
$gwipconf2 = New-AzVirtualNetworkGatewayIpConfig -Name $GWIPconfName2 -Subnet $subnet2 -PublicIpAddress $gwpip2
```

Créez la passerelle VPN avec le numéro AS. Vous devez substituer la valeur par défaut de l’ASN sur vos passerelles VPN Azure. Les ASN des réseaux virtuels connectés doivent être différents pour activer BGP et le routage de transit.

```powershell
New-AzVirtualNetworkGateway -Name $GWName2 -ResourceGroupName $RG2 -Location $Location2 -IpConfigurations $gwipconf2 -GatewayType Vpn -VpnType RouteBased -GatewaySku VpnGw1 -Asn $VNet2ASN
```

### <a name="step-2---connect-the-testvnet1-and-testvnet2-gateways"></a>Étape 2 - Connecter les passerelles TestVNet1 et TestVNet2

Dans cet exemple, les deux passerelles sont dans le même abonnement. Vous pouvez effectuer cette opération dans la même session PowerShell.

#### <a name="1-get-both-gateways"></a>1. Accéder aux deux passerelles

Veillez à ouvrir une session et à vous connecter à Abonnement 1.

```powershell
$vnet1gw = Get-AzVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1
$vnet2gw = Get-AzVirtualNetworkGateway -Name $GWName2 -ResourceGroupName $RG2
```

#### <a name="2-create-both-connections"></a>2. Créer les deux connexions

Cette étape, vous permet créer la connexion de TestVNet1 à TestVNet2 et de TestVNet2 à TestVNet1.

```powershell
New-AzVirtualNetworkGatewayConnection -Name $Connection12 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -VirtualNetworkGateway2 $vnet2gw -Location $Location1 -ConnectionType Vnet2Vnet -SharedKey 'AzureA1b2C3' -EnableBgp $True

New-AzVirtualNetworkGatewayConnection -Name $Connection21 -ResourceGroupName $RG2 -VirtualNetworkGateway1 $vnet2gw -VirtualNetworkGateway2 $vnet1gw -Location $Location2 -ConnectionType Vnet2Vnet -SharedKey 'AzureA1b2C3' -EnableBgp $True
```

> [!IMPORTANT]
> Veillez à activer BGP pour les deux connexions.
> 
> 

Une fois ces étapes effectuées, la connexion est établie après quelques minutes. La session de peering BGP démarre une fois la connexion de réseau virtuel à réseau virtuel établie.

Si vous avez effectué les trois parties de cet exercice, vous avez obtenu une topologie de réseau similaire à la suivante :

![BGP pour une connexion de réseau virtuel à réseau virtuel](./media/vpn-gateway-bgp-resource-manager-ps/bgp-crosspremv2v.png)

## <a name="next-steps"></a>Étapes suivantes

Une fois la connexion achevée, vous pouvez ajouter des machines virtuelles à vos réseaux virtuels. Consultez [Création d’une machine virtuelle](../virtual-machines/windows/quick-create-portal.md) pour connaître les différentes étapes.