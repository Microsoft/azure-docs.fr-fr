---
title: Configurations d’appareils VPN partenaires pour une connexion à des passerelles VPN Azure
description: Cet article fournit une vue d’ensemble des configurations d’appareils VPN partenaires pour une connexion à des passerelles VPN Azure.
services: vpn-gateway
author: yushwang
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/02/2020
ms.author: yushwang
ms.openlocfilehash: 01ceb067f589ef04b04ff0bf05e2971ee30f407c
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89400158"
---
# <a name="overview-of-partner-vpn-device-configurations"></a>Vue d’ensemble des configurations d’appareils VPN partenaires
Cet article fournit une vue d’ensemble de la configuration des appareils VPN locaux pour une connexion à des passerelles VPN Azure. Un exemple de configuration de réseau virtuel Azure et de passerelle VPN est utilisé pour vous montrer comment établir une connexion à différentes configurations d’appareils VPN locaux en utilisant les mêmes paramètres.



## <a name="device-requirements"></a>Configuration requise du périphérique
Les passerelles VPN Azure utilisent des suites de protocoles IPsec/IKE standard pour les tunnels VPN de site à site (S2S). Pour obtenir la liste des paramètres et algorithmes de chiffrement IPsec/IKE pour les passerelles VPN Azure, consultez [À propos des appareils VPN](vpn-gateway-about-vpn-devices.md). Vous pouvez aussi spécifier les algorithmes et les forces de clé exacts d’une connexion spécifique, comme décrit dans [À propos des exigences de chiffrement](vpn-gateway-about-compliance-crypto.md).

## <a name="single-vpn-tunnel"></a><a name ="singletunnel"></a>Tunnel VPN unique
La première configuration de l’exemple présente un tunnel VPN S2S unique entre une passerelle VPN Azure et un appareil VPN local. Vous pouvez éventuellement configurer le [protocole de passerelle frontière (BGP) dans le tunnel VPN](#bgp).

![Diagramme d’un tunnel VPN S2S unique](./media/vpn-gateway-3rdparty-device-config-overview/singletunnel.png)

Pour obtenir des instructions pas à pas pour configurer un tunnel VPN unique, consultez [Configurer une connexion de site à site](vpn-gateway-howto-site-to-site-resource-manager-portal.md). Les sections suivantes spécifient les paramètres de connexion de l’exemple de configuration et fournissent un script PowerShell pour vous aider à démarrer.

### <a name="connection-parameters"></a>Paramètres de connexion
Cette section répertorie les paramètres des exemples décrits dans les sections précédentes.

| **Paramètre**                | **Valeur**                    |
| ---                          | ---                          |
| Préfixes d’adresse de réseau virtuel        | 10.11.0.0/16<br>10.12.0.0/16 |
| IP de la passerelle VPN Azure         | IP de la passerelle VPN Azure         |
| Préfixes d’adresse locale | 10.51.0.0/16<br>10.52.0.0/16 |
| IP du périphérique VPN local    | IP du périphérique VPN local    |
| * ASN BGP du réseau virtuel                | 65010                        |
| * IP d’homologue BGP Azure           | 10.12.255.30                 |
| * ASN BGP local         | 65050                        |
| * IP d’homologue BGP local     | 10.52.255.254                |

\* Paramètre facultatif pour BGP uniquement.

### <a name="sample-powershell-script"></a>Exemple de script PowerShell
Cette section fournit un exemple de script pour vous aider à démarrer. Pour obtenir des instructions détaillées, consultez [Créer une connexion VPN S2S à l’aide de PowerShell](vpn-gateway-create-site-to-site-rm-powershell.md).

```powershell
# Declare your variables

$Sub1          = "Replace_With_Your_Subscription_Name"
$RG1           = "TestRG1"
$Location1     = "East US 2"
$VNetName1     = "TestVNet1"
$FESubName1    = "FrontEnd"
$BESubName1    = "Backend"
$GWSubName1    = "GatewaySubnet"
$VNetPrefix11  = "10.11.0.0/16"
$VNetPrefix12  = "10.12.0.0/16"
$FESubPrefix1  = "10.11.0.0/24"
$BESubPrefix1  = "10.12.0.0/24"
$GWSubPrefix1  = "10.12.255.0/27"
$VNet1ASN      = 65010
$DNS1          = "8.8.8.8"
$GWName1       = "VNet1GW"
$GWIPName1     = "VNet1GWIP"
$GWIPconfName1 = "gwipconf1"
$Connection15  = "VNet1toSite5"
$LNGName5      = "Site5"
$LNGPrefix50   = "10.52.255.254/32"
$LNGPrefix51   = "10.51.0.0/16"
$LNGPrefix52   = "10.52.0.0/16"
$LNGIP5        = "Your_VPN_Device_IP"
$LNGASN5       = 65050
$BGPPeerIP5    = "10.52.255.254"

# Connect to your subscription and create a new resource group

Connect-AzAccount
Select-AzSubscription -SubscriptionName $Sub1
New-AzResourceGroup -Name $RG1 -Location $Location1

# Create virtual network

$fesub1 = New-AzVirtualNetworkSubnetConfig -Name $FESubName1 -AddressPrefix $FESubPrefix1 $besub1 = New-AzVirtualNetworkSubnetConfig -Name $BESubName1 -AddressPrefix $BESubPrefix1
$gwsub1 = New-AzVirtualNetworkSubnetConfig -Name $GWSubName1 -AddressPrefix $GWSubPrefix1

New-AzVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1 -Location $Location1 -AddressPrefix $VNetPrefix11,$VNetPrefix12 -Subnet $fesub1,$besub1,$gwsub1

# Create VPN gateway

$gwpip1    = New-AzPublicIpAddress -Name $GWIPName1 -ResourceGroupName $RG1 -Location $Location1 -AllocationMethod Dynamic
$vnet1     = Get-AzVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1
$subnet1   = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet1
$gwipconf1 = New-AzVirtualNetworkGatewayIpConfig -Name $GWIPconfName1 -Subnet $subnet1 -PublicIpAddress $gwpip1

New-AzVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1 -Location $Location1 -IpConfigurations $gwipconf1 -GatewayType Vpn -VpnType RouteBased -GatewaySku VpnGw1 -Asn $VNet1ASN

# Create local network gateway

New-AzLocalNetworkGateway -Name $LNGName5 -ResourceGroupName $RG1 -Location $Location1 -GatewayIpAddress $LNGIP5 -AddressPrefix $LNGPrefix51,$LNGPrefix52 -Asn $LNGASN5 -BgpPeeringAddress $BGPPeerIP5

# Create the S2S VPN connection

$vnet1gw = Get-AzVirtualNetworkGateway -Name $GWName1  -ResourceGroupName $RG1
$lng5gw  = Get-AzLocalNetworkGateway -Name $LNGName5 -ResourceGroupName $RG1

New-AzVirtualNetworkGatewayConnection -Name $Connection15 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -LocalNetworkGateway2 $lng5gw -Location $Location1 -ConnectionType IPsec -SharedKey 'AzureA1b2C3' -EnableBGP $False
```

### <a name="optional-use-custom-ipsecike-policy-with-usepolicybasedtrafficselectors"></a><a name ="policybased"></a>(Facultatif) Utiliser une stratégie IPsec/IKE personnalisée avec l’option UsePolicyBasedTrafficSelectors
Si vos appareils VPN ne prennent pas en charge les sélecteurs de trafic universels, comme les configurations basées sur le routage ou sur une interface virtuelle de tunnel (VTI), créez une stratégie IPsec/IKE personnalisée avec l’option [UsePolicyBasedTrafficSelectors](vpn-gateway-connect-multiple-policybased-rm-ps.md).

> [!IMPORTANT]
> Vous devez créer une stratégie IPsec/IKE pour activer l’option **UsePolicyBasedTrafficSelectors** sur la connexion.


L’exemple de script crée une stratégie IPsec/IKE avec les paramètres et algorithmes suivants :
* IKEv2: AES256, SHA384, DHGroup24
* IPsec : AES256, SHA1, PFS24, SA Lifetime 7 200 secondes et 20 480 000 Ko(20 Go)

Le script applique la stratégie IPsec/IKE et active l’option **UsePolicyBasedTrafficSelectors** sur la connexion.

```powershell
$ipsecpolicy5 = New-AzIpsecPolicy -IkeEncryption AES256 -IkeIntegrity SHA384 -DhGroup DHGroup24 -IpsecEncryption AES256 -IpsecIntegrity SHA1 -PfsGroup PFS24 -SALifeTimeSeconds 7200 -SADataSizeKilobytes 20480000

$vnet1gw = Get-AzVirtualNetworkGateway -Name $GWName1  -ResourceGroupName $RG1
$lng5gw  = Get-AzLocalNetworkGateway -Name $LNGName5 -ResourceGroupName $RG1

New-AzVirtualNetworkGatewayConnection -Name $Connection15 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -LocalNetworkGateway2 $lng5gw -Location $Location1 -ConnectionType IPsec -SharedKey 'AzureA1b2C3' -EnableBGP $False -IpsecPolicies $ipsecpolicy5 -UsePolicyBasedTrafficSelectors $True
```

### <a name="optional-use-bgp-on-s2s-vpn-connection"></a><a name ="bgp"></a>(Facultatif) Utiliser le protocole de passerelle frontière (BGP) sur la connexion VPN S2S
Quand vous créez la connexion VPN S2S, vous pouvez éventuellement utiliser le protocole [BGP pour la passerelle VPN](vpn-gateway-bgp-resource-manager-ps.md). Cette approche présente deux différences :

* Les préfixes d’adresse locale peuvent correspondre à une adresse d’hôte unique. L’adresse IP de l’homologue BGP local est spécifiée comme suit :

    ```powershell
    New-AzLocalNetworkGateway -Name $LNGName5 -ResourceGroupName $RG1 -Location $Location1 -GatewayIpAddress $LNGIP5 -AddressPrefix $LNGPrefix50 -Asn $LNGASN5 -BgpPeeringAddress $BGPPeerIP5
    ```

* Quand vous créez la connexion, vous devez définir l’option **-EnableBGP** sur $true :

    ```powershell
    New-AzVirtualNetworkGatewayConnection -Name $Connection15 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -LocalNetworkGateway2 $lng5gw -Location $Location1 -ConnectionType IPsec -SharedKey 'AzureA1b2C3' -EnableBGP $True
    ```

## <a name="next-steps"></a>Étapes suivantes
Pour obtenir des instructions pas à pas pour configurer des passerelles VPN en mode actif/actif, consultez [Configuration de passerelles VPN en mode actif/actif pour des connexions entre différents locaux et entre deux réseaux virtuels](vpn-gateway-activeactive-rm-powershell.md).

