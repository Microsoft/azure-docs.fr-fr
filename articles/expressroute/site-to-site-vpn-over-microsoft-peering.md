---
title: 'Azure ExpressRoute : Configurer un réseau VPN site à site via le peering Microsoft'
description: Configurez une connectivité IPsec/IKE vers Azure via un circuit de peering Microsoft ExpressRoute à l’aide d’une passerelle VPN de site à site.
services: expressroute
author: duongau
ms.service: expressroute
ms.topic: how-to
ms.date: 02/25/2019
ms.author: duau
ms.custom: seodec18
ms.openlocfilehash: 366f27a0e2a22e9aa10dda20e105bf644255bdd4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89393137"
---
# <a name="configure-a-site-to-site-vpn-over-expressroute-microsoft-peering"></a>Configurer un réseau VPN de site à site via le peering Microsoft ExpressRoute

Cet article a été conçu pour vous aider à configurer une connectivité chiffrée et sécurisée entre votre réseau local et vos réseaux virtuels Azure via une connexion privée ExpressRoute. Vous pouvez utiliser le peering Microsoft pour établir un tunnel VPN IPsec/IKE de site à site entre les réseaux locaux sélectionnés et les réseaux virtuels Azure. La configuration d’un tunnel sécurisé via ExpressRoute permet un échange de données garantissant confidentialité, authenticité et intégrité, notamment grâce à un système anti-relecture.

>[!NOTE]
>Lorsque vous configurez un VPN de site à site via le peering Microsoft, la passerelle VPN et la sortie VPN vous sont facturées. Pour plus d’informations, consultez [Tarification Passerelle VPN](https://azure.microsoft.com/pricing/details/vpn-gateway).
>
>

[!INCLUDE [updated-for-az](../../includes/hybrid-az-ps.md)]

## <a name="architecture"></a><a name="architecture"></a>Architecture


  ![présentation de la connectivité](./media/site-to-site-vpn-over-microsoft-peering/IPsecER_Overview.png)


Pour la haute disponibilité et la redondance, vous pouvez configurer plusieurs tunnels sur les deux paires MSEE-PE d’un circuit ExpressRoute, puis activer l’équilibrage de charge entre ces tunnels.

  ![options de haute disponibilité](./media/site-to-site-vpn-over-microsoft-peering/HighAvailability.png)

Les tunnels VPN qui utilisent le peering Microsoft peuvent être terminés à l’aide de la passerelle VPN ou de l’une des appliances virtuelles réseau disponibles sur la Place de marché Azure. Vous pouvez échanger les itinéraires statiquement ou dynamiquement via les tunnels chiffrés, sans exposer l’échange d’itinéraires au peering Microsoft sous-jacent. Dans les exemples de cet article, le protocole BGP (qui est différent de la session BGP utilisée pour créer le peering Microsoft) est utilisé pour échanger dynamiquement des préfixes via les tunnels chiffrés.

>[!IMPORTANT]
>Au niveau local, le peering Microsoft est généralement terminé sur le réseau DMZ, et le peering privé dans la zone réseau principale. Ces deux zones sont généralement séparées par un pare-feu. Si vous configurez le peering Microsoft exclusivement pour l’établissement de tunnels sécurisés via ExpressRoute, pensez à filtrer uniquement les adresses IP publiques d’intérêt qui sont publiées via le peering Microsoft.
>
>

## <a name="workflow"></a><a name="workflow"></a>Flux de travail

1. Configurez le peering Microsoft pour votre circuit ExpressRoute.
2. Publiez les préfixes publics régionaux Azure sélectionnés sur votre réseau local via le peering Microsoft.
3. Configurer une passerelle VPN et établir des tunnels IPsec
4. Configurez des appareils VPN locaux.
5. Créez la connexion IPsec/IKE de site à site.
6. (Facultatif) Configurez les pare-feu ou le filtrage sur des appareils VPN locaux.
7. Testez et validez la communication IPsec via le circuit ExpressRoute.

## <a name="1-configure-microsoft-peering"></a><a name="peering"></a>1. Configurer le peering Microsoft

Pour configurer une connexion VPN de site à site via ExpressRoute, vous devez utiliser le peering Microsoft ExpressRoute.

* Pour configurer un nouveau circuit ExpressRoute, lisez les articles [Configuration requise pour ExpressRoute](expressroute-prerequisites.md) et [Création et modification d’un circuit ExpressRoute](expressroute-howto-circuit-arm.md).

* Si vous disposez déjà d’un circuit ExpressRoute, mais n’avez pas configuré le peering Microsoft, configurez celui-ci en vous aidant de l’article [Créer et modifier le peering pour un circuit ExpressRoute](expressroute-howto-routing-arm.md#msft).

Une fois que vous avez configuré votre circuit et le peering Microsoft, vous pouvez facilement l’afficher via la page **Vue d’ensemble** du portail Azure.

![circuit](./media/site-to-site-vpn-over-microsoft-peering/ExpressRouteCkt.png)

## <a name="2-configure-route-filters"></a><a name="routefilter"></a>2. Configurer des filtres de routage

Un filtre de routage vous permet d’identifier les services que vous souhaitez utiliser via le peering Microsoft de votre circuit ExpressRoute. Il s’agit essentiellement de la liste verte de toutes les valeurs de la communauté BGP. 

![Filtre de routage](./media/site-to-site-vpn-over-microsoft-peering/route-filter.png)

Dans cet exemple, le déploiement a lieu uniquement dans la région *Azure USA Ouest 2*. Une règle de filtre de routage est ajoutée pour permettre uniquement la publication des préfixes régionaux Azure USA Ouest 2, dont la valeur de communauté BGP est *12076:51026*. Vous pouvez spécifier les préfixes régionaux que vous souhaitez autoriser en sélectionnant **Gérer la règle**.

Dans le filtre de routage, vous devez également choisir les circuits ExpressRoute auxquels s’applique le filtre. Vous pouvez choisir les circuits ExpressRoute en sélectionnant **Ajouter un circuit**. Dans la figure précédente, le filtre de routage est associé à l’exemple de circuit ExpressRoute.

### <a name="21-configure-the-route-filter"></a><a name="configfilter"></a>2.1 Configurer le filtre de routage

Configurez le filtre de routage. Pour connaître les étapes à suivre, consultez [Configurer des filtres de routage pour le peering Microsoft](how-to-routefilter-portal.md).

### <a name="22-verify-bgp-routes"></a><a name="verifybgp"></a>2.2 Vérifier les itinéraires BGP

Une fois que vous avez créé le peering Microsoft sur votre circuit ExpressRoute et associé un filtre de routage au circuit, vous pouvez vérifier les itinéraires BGP envoyés par les MSEE sur les appareils PE qui sont appairés aux MSEE. La commande de vérification varie selon le système d’exploitation de vos appareils PE.

#### <a name="cisco-examples"></a>Exemples Cisco

Cet exemple utilise une commande IOS-XE Cisco. Dans cet exemple, une instance virtuelle de routage et de transfert est utilisée pour isoler le trafic de peering.

```
show ip bgp vpnv4 vrf 10 summary
```

La sortie partielle ci-dessous montre que 68 préfixes ont été reçus du voisin \*.243.229.34 avec l’ASN 12076 (MSEE) :

```
...

Neighbor        V           AS MsgRcvd MsgSent   TblVer  InQ OutQ Up/Down  State/PfxRcd
X.243.229.34    4        12076   17671   17650    25228    0    0 1w4d           68
```

Pour afficher la liste des préfixes reçus du voisin, utilisez l’exemple suivant :

```
sh ip bgp vpnv4 vrf 10 neighbors X.243.229.34 received-routes
```

Pour vérifier que vous recevez le bon jeu de préfixes, vous pouvez effectuer une vérification croisée. La sortie de commande Azure PowerShell suivante répertorie les préfixes publiés via le peering Microsoft pour chacun des services et chacune des régions Azure :

```azurepowershell-interactive
Get-AzBgpServiceCommunity
```

## <a name="3-configure-the-vpn-gateway-and-ipsec-tunnels"></a><a name="vpngateway"></a>3. Configurer la passerelle VPN et les tunnels IPsec

Dans cette section, les tunnels VPN IPsec sont créés entre la passerelle VPN Azure et le périphérique VPN local. Les exemples utilisent des périphériques VPN Cisco Cloud Service Router (CSR1000).

Le diagramme suivant illustre les tunnels VPN IPsec établis entre le périphérique VPN local 1 et la paire d’instances de passerelles VPN Azure. Les deux tunnels VPN IPsec établis entre le périphérique VPN local 2 et la paire d’instances de passerelles VPN Azure ne sont pas illustrés dans ce diagramme et les détails de leur configuration ne sont pas fournis. Quoi qu’il en soit, les tunnels VPN supplémentaires améliorent la haute disponibilité.

  ![Tunnels VPN](./media/site-to-site-vpn-over-microsoft-peering/EstablishTunnels.png)

Sur la paire de tunnels IPsec, une session eBGP est établie pour échanger des itinéraires de réseau privé. Le diagramme suivant illustre la session eBGP établie sur la paire de tunnels IPsec :

  ![Sessions eBGP sur la paire de tunnels](./media/site-to-site-vpn-over-microsoft-peering/TunnelBGP.png)

Le diagramme suivant montre un extrait de la vue d’ensemble de l’exemple de réseau :

  ![Exemple de réseau](./media/site-to-site-vpn-over-microsoft-peering/OverviewRef.png)

### <a name="about-the-azure-resource-manager-template-examples"></a>À propos des exemples de modèles ARM

Dans les exemples, la fin de la passerelle VPN et la fin du tunnel IPsec sont configurées à l’aide d’un modèle ARM (Azure Resource Manager). Si vous utilisez les modèles Resource Manager pour la première fois et ne connaissez pas ses principes fondamentaux, consultez [Comprendre la structure et la syntaxe des modèles Azure Resource Manager](../azure-resource-manager/templates/template-syntax.md). Le modèle de cette section crée un environnement Azure Greenfield (réseau virtuel). Toutefois, si vous disposez déjà d’un réseau virtuel existant, vous pouvez le référencer dans le modèle. Si vous n’êtes pas familiarisé avec les configurations de site à site IPsec/IKE des passerelles VPN, consultez [Créer une connexion de site à site](../vpn-gateway/vpn-gateway-create-site-to-site-rm-powershell.md).

>[!NOTE]
>Vous n’avez pas besoin d’utiliser les modèles ARM (Azure Resource Manager) pour cette configuration. Vous pouvez créer cette configuration à l’aide du portail Azure ou de PowerShell.
>
>

### <a name="31-declare-the-variables"></a><a name="variables3"></a>3.1 Déclarer les variables

Dans cet exemple, les déclarations de variable sont relatives à l’exemple de réseau. Lorsque vous déclarez des variables, modifiez cette section en fonction de votre environnement.

* La variable **localAddressPrefix** est un tableau d’adresses IP locales qui permet de terminer les tunnels IPsec.
* **gatewaySku** détermine le débit VPN. Pour plus d’informations sur gatewaySku et vpnType, consultez [À propos des paramètres de configuration de la passerelle VPN](../vpn-gateway/vpn-gateway-about-vpn-gateway-settings.md#gwsku). Pour connaître la tarification, consultez [Tarification Passerelle VPN](https://azure.microsoft.com/pricing/details/vpn-gateway).
* Définissez **vpnType** sur **RouteBased**.

```json
"variables": {
  "virtualNetworkName": "SecureVNet",       // Name of the Azure VNet
  "azureVNetAddressPrefix": "10.2.0.0/24",  // Address space assigned to the VNet
  "subnetName": "Tenant",                   // subnet name in which tenants exists
  "subnetPrefix": "10.2.0.0/25",            // address space of the tenant subnet
  "gatewaySubnetPrefix": "10.2.0.224/27",   // address space of the gateway subnet
  "localGatewayName": "localGW1",           // name of remote gateway (on-premises)
  "localGatewayIpAddress": "X.243.229.110", // public IP address of the on-premises VPN device
  "localAddressPrefix": [
    "172.16.0.1/32",                        // termination of IPsec tunnel-1 on-premises 
    "172.16.0.2/32"                         // termination of IPsec tunnel-2 on-premises 
  ],
  "gatewayPublicIPName1": "vpnGwVIP1",    // Public address name of the first VPN gateway instance
  "gatewayPublicIPName2": "vpnGwVIP2",    // Public address name of the second VPN gateway instance 
  "gatewayName": "vpnGw",                 // Name of the Azure VPN gateway
  "gatewaySku": "VpnGw1",                 // Azure VPN gateway SKU
  "vpnType": "RouteBased",                // type of VPN gateway
  "sharedKey": "string",                  // shared secret needs to match with on-premises configuration
  "asnVpnGateway": 65000,                 // BGP Autonomous System number assigned to the VPN Gateway 
  "asnRemote": 65010,                     // BGP Autonmous Syste number assigned to the on-premises device
  "bgpPeeringAddress": "172.16.0.3",      // IP address of the remote BGP peer on-premises
  "connectionName": "vpn2local1",
  "vnetID": "[resourceId('Microsoft.Network/virtualNetworks', variables('virtualNetworkName'))]",
  "gatewaySubnetRef": "[concat(variables('vnetID'),'/subnets/','GatewaySubnet')]",
  "subnetRef": "[concat(variables('vnetID'),'/subnets/',variables('subnetName'))]",
  "api-version": "2017-06-01"
},
```

### <a name="32-create-virtual-network-vnet"></a><a name="vnet"></a>3.2 Créer un réseau virtuel

Si vous associez un réseau virtuel existant à des tunnels VPN, vous pouvez ignorer cette étape.

```json
{
  "apiVersion": "[variables('api-version')]",
  "type": "Microsoft.Network/virtualNetworks",
  "name": "[variables('virtualNetworkName')]",
  "location": "[resourceGroup().location]",
  "properties": {
    "addressSpace": {
      "addressPrefixes": [
        "[variables('azureVNetAddressPrefix')]"
      ]
    },
    "subnets": [
      {
        "name": "[variables('subnetName')]",
        "properties": {
          "addressPrefix": "[variables('subnetPrefix')]"
        }
      },
      {
        "name": "GatewaySubnet",
        "properties": {
          "addressPrefix": "[variables('gatewaySubnetPrefix')]"
        }
      }
    ]
  },
  "comments": "Create a Virtual Network with Subnet1 and Gatewaysubnet"
},
```

### <a name="33-assign-public-ip-addresses-to-vpn-gateway-instances"></a><a name="ip"></a>3.3 Affecter des adresses IP publiques aux instances de passerelles VPN
 
Affectez une adresse IP publique à chaque instance de passerelle VPN.

```json
{
  "apiVersion": "[variables('api-version')]",
  "type": "Microsoft.Network/publicIPAddresses",
    "name": "[variables('gatewayPublicIPName1')]",
    "location": "[resourceGroup().location]",
    "properties": {
      "publicIPAllocationMethod": "Dynamic"
    },
    "comments": "Public IP for the first instance of the VPN gateway"
  },
  {
    "apiVersion": "[variables('api-version')]",
    "type": "Microsoft.Network/publicIPAddresses",
    "name": "[variables('gatewayPublicIPName2')]",
    "location": "[resourceGroup().location]",
    "properties": {
      "publicIPAllocationMethod": "Dynamic"
    },
    "comments": "Public IP for the second instance of the VPN gateway"
  },
```

### <a name="34-specify-the-on-premises-vpn-tunnel-termination-local-network-gateway"></a><a name="termination"></a>3.4 Spécifier la fin du tunnel VPN local (passerelle de réseau local)

Les appareils VPN locaux sont désignés comme des **passerelles de réseau local**. L’extrait de code json suivant spécifie également les informations du pair BGP distant :

```json
{
  "apiVersion": "[variables('api-version')]",
  "type": "Microsoft.Network/localNetworkGateways",
  "name": "[variables('localGatewayName')]",
  "location": "[resourceGroup().location]",
  "properties": {
    "localNetworkAddressSpace": {
      "addressPrefixes": "[variables('localAddressPrefix')]"
    },
    "gatewayIpAddress": "[variables('localGatewayIpAddress')]",
    "bgpSettings": {
      "asn": "[variables('asnRemote')]",
      "bgpPeeringAddress": "[variables('bgpPeeringAddress')]",
      "peerWeight": 0
    }
  },
  "comments": "Local Network Gateway (referred to your on-premises location) with IP address of remote tunnel peering and IP address of remote BGP peer"
},
```

### <a name="35-create-the-vpn-gateway"></a><a name="creategw"></a>3.5 Créer la passerelle VPN

Cette section du modèle configure la passerelle VPN à l’aide des paramètres nécessaires à une configuration en mode actif/passif. Gardez à l’esprit les exigences suivantes :

* Créez la passerelle VPN avec VpnType défini sur **RouteBased**. Ce paramètre est obligatoire si vous souhaitez activer le routage BGP entre la passerelle VPN et le VPN local.
* Pour établir des tunnels VPN entre les deux instances de la passerelle VPN et un appareil local donné en mode actif/passif, le paramètre **activeActive** doit être défini sur **true** dans le modèle Resource Manager. Pour en savoir plus sur les passerelles VPN hautement disponibles, consultez [Connectivité des passerelles VPN hautement disponibles](../vpn-gateway/vpn-gateway-highlyavailable.md).
* Pour configurer des sessions eBGP entre les tunnels VPN, vous devez spécifier deux ASN différents de chaque côté. Il est préférable de spécifier des numéros ASN privés. Pour plus d’informations, consultez [Vue d’ensemble du protocole BGP avec les passerelles VPN Azure](../vpn-gateway/vpn-gateway-bgp-overview.md).

```json
{
"apiVersion": "[variables('api-version')]",
"type": "Microsoft.Network/virtualNetworkGateways",
"name": "[variables('gatewayName')]",
"location": "[resourceGroup().location]",
"dependsOn": [
  "[concat('Microsoft.Network/publicIPAddresses/', variables('gatewayPublicIPName1'))]",
  "[concat('Microsoft.Network/publicIPAddresses/', variables('gatewayPublicIPName2'))]",
  "[concat('Microsoft.Network/virtualNetworks/', variables('virtualNetworkName'))]"
],
"properties": {
  "ipConfigurations": [
    {
      "properties": {
        "privateIPAllocationMethod": "Dynamic",
        "subnet": {
          "id": "[variables('gatewaySubnetRef')]"
        },
        "publicIPAddress": {
          "id": "[resourceId('Microsoft.Network/publicIPAddresses',variables('gatewayPublicIPName1'))]"
        }
      },
      "name": "vnetGtwConfig1"
    },
    {
      "properties": {
        "privateIPAllocationMethod": "Dynamic",
        "subnet": {
          "id": "[variables('gatewaySubnetRef')]"
        },
        "publicIPAddress": {
          "id": "[resourceId('Microsoft.Network/publicIPAddresses',variables('gatewayPublicIPName2'))]"
        }
      },
          "name": "vnetGtwConfig2"
        }
      ],
      "sku": {
        "name": "[variables('gatewaySku')]",
        "tier": "[variables('gatewaySku')]"
      },
      "gatewayType": "Vpn",
      "vpnType": "[variables('vpnType')]",
      "enableBgp": true,
      "activeActive": true,
      "bgpSettings": {
        "asn": "[variables('asnVpnGateway')]"
      }
    },
    "comments": "VPN Gateway in active-active configuration with BGP support"
  },
```

### <a name="36-establish-the-ipsec-tunnels"></a><a name="ipsectunnel"></a>3.6 Établir les tunnels IPsec

La dernière action du script crée les tunnels IPsec entre la passerelle VPN Azure et l’appareil VPN local.

```json
{
  "apiVersion": "[variables('api-version')]",
  "name": "[variables('connectionName')]",
  "type": "Microsoft.Network/connections",
  "location": "[resourceGroup().location]",
  "dependsOn": [
    "[concat('Microsoft.Network/virtualNetworkGateways/', variables('gatewayName'))]",
    "[concat('Microsoft.Network/localNetworkGateways/', variables('localGatewayName'))]"
  ],
  "properties": {
    "virtualNetworkGateway1": {
      "id": "[resourceId('Microsoft.Network/virtualNetworkGateways', variables('gatewayName'))]"
    },
    "localNetworkGateway2": {
      "id": "[resourceId('Microsoft.Network/localNetworkGateways', variables('localGatewayName'))]"
    },
    "connectionType": "IPsec",
    "routingWeight": 0,
    "sharedKey": "[variables('sharedKey')]",
    "enableBGP": "true"
  },
  "comments": "Create a Connection type site-to-site (IPsec) between the Azure VPN Gateway and the VPN device on-premises"
  }
```

## <a name="4-configure-the-on-premises-vpn-device"></a><a name="device"></a>4. Configurer l’appareil VPN local

La passerelle VPN Azure est compatible avec la plupart des appareils VPN des différents fournisseurs. Pour plus d’informations sur la configuration et les appareils qui ont été validés pour fonctionner avec la passerelle VPN, consultez [À propos des appareils VPN](../vpn-gateway/vpn-gateway-about-vpn-devices.md).

Pour configurer votre appareil VPN, vous avez besoin des éléments suivants :

* Une clé partagée. Il s’agit de la clé partagée spécifiée lors de la création de la connexion VPN de site à site. Les exemples utilisent une clé partagée basique. Nous vous conseillons de générer une clé plus complexe.
* Il s’agit de l’adresse IP publique de votre passerelle VPN. Vous pouvez afficher l’adresse IP publique à l’aide du portail Azure, de PowerShell ou de l’interface de ligne de commande. Pour rechercher l’adresse IP publique de votre passerelle VPN à l’aide du portail Azure, accédez à Passerelles de réseau virtuel, puis cliquez sur le nom de votre passerelle.

En règle générale, les pairs eBGP sont directement connectés (souvent via une connexion WAN). Toutefois, lorsque vous configurez eBGP sur des tunnels VPN IPsec via le peering ExpressRoute Microsoft, il existe plusieurs domaines de routage entre les pairs eBGP. Utilisez la commande **ebgp-multihop** pour établir la relation de voisin eBGP entre les deux pairs connectés indirectement. L’entier qui suit la commande ebgp-multihop spécifie la valeur de durée de vie dans les paquets BGP. La commande **maximum-paths eibgp 2** permet l’équilibrage de charge du trafic entre les deux chemins BGP.

### <a name="cisco-csr1000-example"></a><a name="cisco1"></a>Exemple Cisco CSR1000

L’exemple suivant montre une configuration Cisco CSR1000 sur une machine virtuelle Hyper-V en tant que appareil VPN local :

```
!
crypto ikev2 proposal az-PROPOSAL
 encryption aes-cbc-256 aes-cbc-128 3des
 integrity sha1
 group 2
!
crypto ikev2 policy az-POLICY
 proposal az-PROPOSAL
!
crypto ikev2 keyring key-peer1
 peer azvpn1
  address 52.175.253.112
  pre-shared-key secret*1234
 !
!
crypto ikev2 keyring key-peer2
 peer azvpn2
  address 52.175.250.191
  pre-shared-key secret*1234
 !
!
!
crypto ikev2 profile az-PROFILE1
 match address local interface GigabitEthernet1
 match identity remote address 52.175.253.112 255.255.255.255
 authentication remote pre-share
 authentication local pre-share
 keyring local key-peer1
!
crypto ikev2 profile az-PROFILE2
 match address local interface GigabitEthernet1
 match identity remote address 52.175.250.191 255.255.255.255
 authentication remote pre-share
 authentication local pre-share
 keyring local key-peer2
!
crypto ikev2 dpd 10 2 on-demand
!
!
crypto ipsec transform-set az-IPSEC-PROPOSAL-SET esp-aes 256 esp-sha-hmac
 mode tunnel
!
crypto ipsec profile az-VTI1
 set transform-set az-IPSEC-PROPOSAL-SET
 set ikev2-profile az-PROFILE1
!
crypto ipsec profile az-VTI2
 set transform-set az-IPSEC-PROPOSAL-SET
 set ikev2-profile az-PROFILE2
!
!
interface Loopback0
 ip address 172.16.0.3 255.255.255.255
!
interface Tunnel0
 ip address 172.16.0.1 255.255.255.255
 ip tcp adjust-mss 1350
 tunnel source GigabitEthernet1
 tunnel mode ipsec ipv4
 tunnel destination 52.175.253.112
 tunnel protection ipsec profile az-VTI1
!
interface Tunnel1
 ip address 172.16.0.2 255.255.255.255
 ip tcp adjust-mss 1350
 tunnel source GigabitEthernet1
 tunnel mode ipsec ipv4
 tunnel destination 52.175.250.191
 tunnel protection ipsec profile az-VTI2
!
interface GigabitEthernet1
 description External interface
 ip address x.243.229.110 255.255.255.252
 negotiation auto
 no mop enabled
 no mop sysid
!
interface GigabitEthernet2
 ip address 10.0.0.1 255.255.255.0
 negotiation auto
 no mop enabled
 no mop sysid
!
router bgp 65010
 bgp router-id interface Loopback0
 bgp log-neighbor-changes
 network 10.0.0.0 mask 255.255.255.0
 network 10.1.10.0 mask 255.255.255.128
 neighbor 10.2.0.228 remote-as 65000
 neighbor 10.2.0.228 ebgp-multihop 5
 neighbor 10.2.0.228 update-source Loopback0
 neighbor 10.2.0.228 soft-reconfiguration inbound
 neighbor 10.2.0.228 filter-list 10 out
 neighbor 10.2.0.229 remote-as 65000    
 neighbor 10.2.0.229 ebgp-multihop 5
 neighbor 10.2.0.229 update-source Loopback0
 neighbor 10.2.0.229 soft-reconfiguration inbound
 maximum-paths eibgp 2
!
ip route 0.0.0.0 0.0.0.0 10.1.10.1
ip route 10.2.0.228 255.255.255.255 Tunnel0
ip route 10.2.0.229 255.255.255.255 Tunnel1
!
```

## <a name="5-configure-vpn-device-filtering-and-firewalls-optional"></a><a name="firewalls"></a>5. Configurer le filtrage des appareils VPN et les pare-feu (facultatif)

Configurez votre pare-feu et votre filtrage selon vos besoins.

## <a name="6-test-and-validate-the-ipsec-tunnel"></a><a name="testipsec"></a>6. Tester et valider le tunnel IPsec

L’état des tunnels IPsec peut être vérifié sur la passerelle VPN Azure à l’aide de commandes Powershell :

```azurepowershell-interactive
Get-AzVirtualNetworkGatewayConnection -Name vpn2local1 -ResourceGroupName myRG | Select-Object  ConnectionStatus,EgressBytesTransferred,IngressBytesTransferred | fl
```

Exemple de sortie :

```azurepowershell
ConnectionStatus        : Connected
EgressBytesTransferred  : 17734660
IngressBytesTransferred : 10538211
```

Pour vérifier l’état des tunnels de chacune des instances de passerelles VPN Azure, utilisez l’exemple suivant :

```azurepowershell-interactive
Get-AzVirtualNetworkGatewayConnection -Name vpn2local1 -ResourceGroupName myRG | Select-Object -ExpandProperty TunnelConnectionStatus
```

Exemple de sortie :

```azurepowershell
Tunnel                           : vpn2local1_52.175.250.191
ConnectionStatus                 : Connected
IngressBytesTransferred          : 4877438
EgressBytesTransferred           : 8754071
LastConnectionEstablishedUtcTime : 11/04/2017 17:03:30

Tunnel                           : vpn2local1_52.175.253.112
ConnectionStatus                 : Connected
IngressBytesTransferred          : 5660773
EgressBytesTransferred           : 8980589
LastConnectionEstablishedUtcTime : 11/04/2017 17:03:13
```

Vous pouvez également vérifier l’état du tunnel sur votre appareil VPN local.

Exemple Cisco CSR1000 :

```
show crypto session detail
show crypto ikev2 sa
show crypto ikev2 session detail
show crypto ipsec sa
```

Exemple de sortie :

```
csr1#show crypto session detail

Crypto session current status

Code: C - IKE Configuration mode, D - Dead Peer Detection
K - Keepalives, N - NAT-traversal, T - cTCP encapsulation
X - IKE Extended Authentication, F - IKE Fragmentation
R - IKE Auto Reconnect

Interface: Tunnel1
Profile: az-PROFILE2
Uptime: 00:52:46
Session status: UP-ACTIVE
Peer: 52.175.250.191 port 4500 fvrf: (none) ivrf: (none)
      Phase1_id: 52.175.250.191
      Desc: (none)
  Session ID: 3
  IKEv2 SA: local 10.1.10.50/4500 remote 52.175.250.191/4500 Active
          Capabilities:DN connid:3 lifetime:23:07:14
  IPSEC FLOW: permit ip 0.0.0.0/0.0.0.0 0.0.0.0/0.0.0.0
        Active SAs: 2, origin: crypto map
        Inbound:  #pkts dec'ed 279 drop 0 life (KB/Sec) 4607976/433
        Outbound: #pkts enc'ed 164 drop 0 life (KB/Sec) 4607992/433

Interface: Tunnel0
Profile: az-PROFILE1
Uptime: 00:52:43
Session status: UP-ACTIVE
Peer: 52.175.253.112 port 4500 fvrf: (none) ivrf: (none)
      Phase1_id: 52.175.253.112
      Desc: (none)
  Session ID: 2
  IKEv2 SA: local 10.1.10.50/4500 remote 52.175.253.112/4500 Active
          Capabilities:DN connid:2 lifetime:23:07:17
  IPSEC FLOW: permit ip 0.0.0.0/0.0.0.0 0.0.0.0/0.0.0.0
        Active SAs: 2, origin: crypto map
        Inbound:  #pkts dec'ed 668 drop 0 life (KB/Sec) 4607926/437
        Outbound: #pkts enc'ed 477 drop 0 life (KB/Sec) 4607953/437
```

Dans l’interface VTI, le protocole de ligne ne passe pas à l’état actif tant que la phase 2 IKE n’est pas terminée. La commande suivante vérifie l’association de sécurité :

```
csr1#show crypto ikev2 sa

IPv4 Crypto IKEv2  SA

Tunnel-id Local                 Remote                fvrf/ivrf            Status
2         10.1.10.50/4500       52.175.253.112/4500   none/none            READY
      Encr: AES-CBC, keysize: 256, PRF: SHA1, Hash: SHA96, DH Grp:2, Auth sign: PSK, Auth verify: PSK
      Life/Active Time: 86400/3277 sec

Tunnel-id Local                 Remote                fvrf/ivrf            Status
3         10.1.10.50/4500       52.175.250.191/4500   none/none            READY
      Encr: AES-CBC, keysize: 256, PRF: SHA1, Hash: SHA96, DH Grp:2, Auth sign: PSK, Auth verify: PSK
      Life/Active Time: 86400/3280 sec

IPv6 Crypto IKEv2  SA

csr1#show crypto ipsec sa | inc encaps|decaps
    #pkts encaps: 177, #pkts encrypt: 177, #pkts digest: 177
    #pkts decaps: 296, #pkts decrypt: 296, #pkts verify: 296
    #pkts encaps: 554, #pkts encrypt: 554, #pkts digest: 554
    #pkts decaps: 746, #pkts decrypt: 746, #pkts verify: 746
```

### <a name="verify-end-to-end-connectivity-between-the-inside-network-on-premises-and-the-azure-vnet"></a><a name="verifye2e"></a>Vérifier la connectivité de bout en bout entre l’intérieur du réseau local et le réseau virtuel Azure

Si les tunnels IPsec sont activés et les itinéraires statiques correctement définis, vous devez pouvoir effectuer un test ping sur l’adresse IP du pair BGP distant :

```
csr1#ping 10.2.0.228
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.2.0.228, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 5/5/5 ms

#ping 10.2.0.229
Type escape sequence to abort.
Sending 5, 100-byte ICMP Echos to 10.2.0.229, timeout is 2 seconds:
!!!!!
Success rate is 100 percent (5/5), round-trip min/avg/max = 4/5/6 ms
```

### <a name="verify-the-bgp-sessions-over-ipsec"></a><a name="verifybgp"></a>Vérifier les sessions BGP sur IPsec

Sur la passerelle VPN Azure, vérifiez l’état du pair BGP :

```azurepowershell-interactive
Get-AzVirtualNetworkGatewayBGPPeerStatus -VirtualNetworkGatewayName vpnGtw -ResourceGroupName SEA-C1-VPN-ER | ft
```

Exemple de sortie :

```azurepowershell
  Asn ConnectedDuration LocalAddress MessagesReceived MessagesSent Neighbor    RoutesReceived State    
  --- ----------------- ------------ ---------------- ------------ --------    -------------- -----    
65010 00:57:19.9003584  10.2.0.228               68           72   172.16.0.10              2 Connected
65000                   10.2.0.228                0            0   10.2.0.228               0 Unknown  
65000 07:13:51.0109601  10.2.0.228              507          500   10.2.0.229               6 Connected
```

Pour vérifier la liste des préfixes réseau envoyés via eBGP par le concentrateur VPN local, vous pouvez filtrer selon l’attribut « Origin » :

```azurepowershell-interactive
Get-AzVirtualNetworkGatewayLearnedRoute -VirtualNetworkGatewayName vpnGtw -ResourceGroupName myRG  | Where-Object Origin -eq "EBgp" |ft
```

Dans l’exemple de sortie, l’ASN 65010 correspond au numéro de système autonome BGP dans le VPN local.

```azurepowershell
AsPath LocalAddress Network      NextHop     Origin SourcePeer  Weight
------ ------------ -------      -------     ------ ----------  ------
65010  10.2.0.228   10.1.10.0/25 172.16.0.10 EBgp   172.16.0.10  32768
65010  10.2.0.228   10.0.0.0/24  172.16.0.10 EBgp   172.16.0.10  32768
```

Pour afficher la liste des itinéraires publiés :

```azurepowershell-interactive
Get-AzVirtualNetworkGatewayAdvertisedRoute -VirtualNetworkGatewayName vpnGtw -ResourceGroupName myRG -Peer 10.2.0.228 | ft
```

Exemple de sortie :

```azurepowershell
AsPath LocalAddress Network        NextHop    Origin SourcePeer Weight
------ ------------ -------        -------    ------ ---------- ------
       10.2.0.229   10.2.0.0/24    10.2.0.229 Igp                  0
       10.2.0.229   172.16.0.10/32 10.2.0.229 Igp                  0
       10.2.0.229   172.16.0.5/32  10.2.0.229 Igp                  0
       10.2.0.229   172.16.0.1/32  10.2.0.229 Igp                  0
65010  10.2.0.229   10.1.10.0/25   10.2.0.229 Igp                  0
65010  10.2.0.229   10.0.0.0/24    10.2.0.229 Igp                  0
```

Exemple pour le routeur local Cisco CSR1000 :

```
csr1#show ip bgp neighbors 10.2.0.228 routes
BGP table version is 7, local router ID is 172.16.0.10
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
              t secondary path,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>   10.2.0.0/24      10.2.0.228                             0 65000 i
 r>   172.16.0.1/32    10.2.0.228                             0 65000 i
 r>   172.16.0.2/32    10.2.0.228                             0 65000 i
 r>   172.16.0.3/32   10.2.0.228                             0 65000 i

Total number of prefixes 4
```

La liste des réseaux publiés à partir du routeur local Cisco CSR1000 sur la passerelle VPN Azure peut être obtenue à l’aide de la commande suivante :

```
csr1#show ip bgp neighbors 10.2.0.228 advertised-routes
BGP table version is 7, local router ID is 172.16.0.10
Status codes: s suppressed, d damped, h history, * valid, > best, i - internal,
              r RIB-failure, S Stale, m multipath, b backup-path, f RT-Filter,
              x best-external, a additional-path, c RIB-compressed,
              t secondary path,
Origin codes: i - IGP, e - EGP, ? - incomplete
RPKI validation codes: V valid, I invalid, N Not found

     Network          Next Hop            Metric LocPrf Weight Path
 *>   10.0.0.0/24      0.0.0.0                  0         32768 i
 *>   10.1.10.0/25     0.0.0.0                  0         32768 i

Total number of prefixes 2
```

## <a name="next-steps"></a>Étapes suivantes

* [Configurer Network Performance Monitor pour ExpressRoute](how-to-npm.md)

* [Ajouter une connexion de site à site à un réseau virtuel avec une connexion de passerelle VPN existante](../vpn-gateway/vpn-gateway-howto-multi-site-to-site-resource-manager-portal.md)
