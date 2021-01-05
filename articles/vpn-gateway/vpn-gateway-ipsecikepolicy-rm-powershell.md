---
title: Stratégie IPsec/IKE pour des connexions VPN S2S ou de réseau virtuel à réseau virtuel
titleSuffix: Azure VPN Gateway
description: Configurez la stratégie IPSec/IKE pour des connexions VPN S2S ou de réseau virtuel à réseau virtuel avec des passerelles VPN Azure à l’aide d’Azure Resource Manager et de PowerShell.
services: vpn-gateway
author: yushwang
ms.service: vpn-gateway
ms.topic: how-to
ms.date: 09/02/2020
ms.author: yushwang
ms.openlocfilehash: 96931d2dd94a8a31021ebe62caaefc54f643b007
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94649260"
---
# <a name="configure-ipsecike-policy-for-s2s-vpn-or-vnet-to-vnet-connections"></a>Configurer la stratégie IPsec/IKE pour des connexions VPN S2S ou de réseau virtuel à réseau virtuel

Cet article vous guide dans les étapes de configuration de la stratégie IPsec/IKE pour la connexion VPN de site à site ou la connexion de réseau virtuel à réseau virtuel à l’aide du modèle de déploiement Resource Manager et de PowerShell.



## <a name="about-ipsec-and-ike-policy-parameters-for-azure-vpn-gateways"></a><a name="about"></a>À propos des paramètres de stratégie IPsec et IKE pour les passerelles VPN Azure
La norme de protocole IPsec et IKE standard prend en charge un vaste éventail d’algorithmes de chiffrement dans différentes combinaisons. Pour découvrir comment cela peut vous aider à vous assurer que la connectivité entre sites locaux et de réseau virtuel à réseau virtuel répond à vos besoins de conformité ou de sécurité, voir [À propos des exigences de chiffrement et des passerelles VPN Azure](vpn-gateway-about-compliance-crypto.md).

Cet article fournit des instructions pour créer et configurer une stratégie IPsec/IKE et appliquer celle-ci à une connexion nouvelle ou existante :

* [Partie 1 : flux de travail de la création et de la définition d’une stratégie IPsec/IKE](#workflow)
* [Partie 2 : algorithmes de chiffrement pris en charge et avantages clés](#params)
* [Partie 3 : création d’une connexion VPN S2S avec une stratégie IPsec/IKE](#crossprem)
* [Partie 4 : création d’une connexion de réseau virtuel à réseau virtuel avec une stratégie IPsec/IKE](#vnet2vnet)
* [Partie 5 : gestion (créer, ajouter, supprimer) de la stratégie IPsec/IKE pour une connexion](#managepolicy)

> [!IMPORTANT]
> 1. Notez que la stratégie IPsec/IKE fonctionne uniquement sur les références (SKU) de passerelle suivantes :
>    * ***VpnGw1, VpnGw2, VpnGw3** _ (basé sur un itinéraire) _ ***Standard** _ et _*_HighPerformance_*_ (basé sur un itinéraire)
> 2. Vous ne pouvez spécifier qu’_*_une_*_ seule combinaison de stratégies pour une connexion donnée.
> 3. Vous devez spécifier tous les algorithmes et paramètres pour IKE (mode principal) et IPsec (mode rapide). Vous n’êtes pas en droit de spécifier de stratégie partielle.
> 4. Consulter les spécifications de votre fournisseur de périphérique VPN pour vous assurer que la stratégie est prise en charge sur vos périphériques VPN locaux. Des connexions S2S ou de réseau virtuel à réseau virtuel ne peuvent pas être établies si les stratégies ne sont pas compatibles.

## <a name="part-1---workflow-to-create-and-set-ipsecike-policy"></a><a name ="workflow"></a>Partie 1 - Workflow de création et de définition d’une stratégie IPsec/IKE
Cette section décrit le flux de travail de la création et de la mise à jour d’une stratégie IPsec/IKE sur une connexion VPN S2S ou de réseau virtuel à réseau virtuel :
1. créer un réseau virtuel et une passerelle VPN ;
2. créer une passerelle de réseau local pour une connexion entre sites locaux, ou un autre réseau virtuel et une passerelle pour une connexion de réseau virtuel à réseau virtuel ;
3. créer une stratégie IPsec/IKE avec des algorithmes et paramètres sélectionnés ;
4. créer une connexion (IPsec ou VNet2VNet) avec la stratégie IPsec/IKE ;
5. ajouter/mettre à jour/supprimer une stratégie IPsec/IKE pour une connexion existante.

Les instructions fournies dans cet article expliquent comment établir et configurer des stratégies IPsec/IKE comme indiqué dans le diagramme :

![stratégie IPSec/IKE](./media/vpn-gateway-ipsecikepolicy-rm-powershell/ipsecikepolicy.png)

## <a name="part-2---supported-cryptographic-algorithms--key-strengths"></a><a name ="params"></a>Partie 2 : algorithmes de chiffrement pris en charge et avantages clés

Le tableau suivant répertorie les algorithmes de chiffrement et les forces de clé pris en charge et configurables par les clients :

| _ *IPsec/IKEv2**  | **Options**    |
| ---  | --- 
| Chiffrement IKEv2 | AES256, AES192, AES128, DES3, DES  
| Intégrité IKEv2  | SHA384, SHA256, SHA1, MD5  |
| Groupe DH         | DHGroup24, ECP384, ECP256, DHGroup14, DHGroup2048, DHGroup2, DHGroup1, Aucun |
| Chiffrement IPsec | GCMAES256, GCMAES192, GCMAES128, AES256, AES192, AES128, DES3, DES, Aucun    |
| Intégrité IPsec  | GCMASE256, GCMAES192, GCMAES128, SHA256, SHA1, MD5 |
| Groupe PFS        | PFS24, ECP384, ECP256, PFS2048, PFS2, PFS1, Aucun 
| Durée de vie de l’AS en mode rapide   | (**Facultatif** : les valeurs par défaut sont utilisées si aucun valeur n’est indiquée)<br>Secondes (entier ; **min 300** /par défaut 27 000 secondes)<br>Kilo-octets (entier ; **min 1024** /par défaut 102 400 000 Ko)   |
| Sélecteur de trafic | UsePolicyBasedTrafficSelectors** ($True/$False ; **facultatif**, $False par défaut si aucune valeur n’est indiquée)    |
|  |  |

> [!IMPORTANT]
> 1. **La configuration de votre périphérique VPN local doit correspondre aux algorithmes et paramètres suivants, spécifiés dans la stratégie IPsec/IKE Azure ou les contenir :**
>    * Algorithme de chiffrement IKE (Mode principal / Phase 1)
>    * Algorithme d’intégrité IKE (Mode principal / Phase 1)
>    * Groupe DH (Mode principal / Phase 1)
>    * Algorithme de chiffrement IPsec (Mode rapide / Phase 2)
>    * Algorithme d’intégrité IPsec (Mode rapide / Phase 2)
>    * Groupe PFS (Mode rapide / Phase 2)
>    * Sélecteur de trafic (si UsePolicyBasedTrafficSelectors est utilisé)
>    * Les durées de vie de l’AS sont uniquement des spécifications locales, elles n’ont pas besoin de correspondre.
>
> 2. **Si GCMAES est utilisé pour l’algorithme de chiffrement IPsec, vous devez sélectionner le même algorithme GCMAES et la même longueur de clé pour l’intégrité IPsec ; par exemple, GCMAES128 pour les deux**
> 3. Dans le tableau ci-dessus :
>    * IKEv2 correspond au Mode principal ou à la Phase 1
>    * IPsec correspond au Mode rapide ou à la Phase 2
>    * Groupe DH spécifie le groupe Diffie-Hellmen utilisé dans le Mode principal ou à la Phase 1
>    * Groupe PFS spécifie le groupe Diffie-Hellmen utilisé dans le Mode rapide ou à la Phase 2
> 4. La durée de vie de l’AS en mode principal IKEv2 est fixée à 28 800 secondes pour les passerelles VPN Azure
> 5. La définition du paramètre « UsePolicyBasedTrafficSelectors » sur $True sur une connexion a pour effet de configurer la passerelle VPN Azure pour se connecter à un pare-feu VPN basé sur une stratégie en local. Si vous activez UsePolicyBasedTrafficSelectors, vous devez vous assurer que votre périphérique VPN dispose des sélecteurs de trafic correspondant définis avec toutes les combinaisons de préfixes de réseau local (passerelle réseau locale) à destination et à partir des préfixes du réseau virtuel Azure, plutôt que de manière indifférenciée. Par exemple, si les préfixes de votre réseau local sont 10.1.0.0/16 et 10.2.0.0/16 et si les préfixes de votre réseau virtuel sont 192.168.0.0/16 et 172.16.0.0/16, vous devez spécifier les sélecteurs de trafic suivants :
>    * 10.1.0.0/16 <====> 192.168.0.0/16
>    * 10.1.0.0/16 <====> 172.16.0.0/16
>    * 10.2.0.0/16 <====> 192.168.0.0/16
>    * 10.2.0.0/16 <====> 172.16.0.0/16

Pour plus d’informations sur les sélecteurs de trafic basés sur une stratégie, voir [Connecter plusieurs périphériques VPN basés sur une stratégie locale](vpn-gateway-connect-multiple-policybased-rm-ps.md).

Le tableau suivant répertorie les groupes Diffie-Hellman correspondants pris en charge par la stratégie personnalisée :

| **Groupe Diffie-Hellman**  | **DHGroup**              | **PFSGroup** | **Longueur de clé** |
| --- | --- | --- | --- |
| 1                         | DHGroup1                 | PFS1         | MODP 768 bits   |
| 2                         | DHGroup2                 | PFS2         | MODP 1 024 bits  |
| 14                        | DHGroup14<br>DHGroup2048 | PFS2048      | MODP 2 048 bits  |
| 19                        | ECP256                   | ECP256       | ECP 256 bits    |
| 20                        | ECP384                   | ECP384       | ECP 384 bits    |
| 24                        | DHGroup24                | PFS24        | MODP 2 048 bits  |

Reportez-vous à [RFC3526](https://tools.ietf.org/html/rfc3526) et [RFC5114](https://tools.ietf.org/html/rfc5114) pour plus d’informations.

## <a name="part-3---create-a-new-s2s-vpn-connection-with-ipsecike-policy"></a><a name ="crossprem"></a>Partie 3 : création d’une connexion VPN S2S avec une stratégie IPsec/IKE

Cette section vous guide tout au long des étapes de création d’une connexion VPN S2S avec une stratégie IPsec/IKE. Les étapes suivantes créent la connexion comme indiqué dans le diagramme :

![stratégie S2S](./media/vpn-gateway-ipsecikepolicy-rm-powershell/s2spolicy.png)

Pour des instructions détaillées concernant la création d’une connexion VPN S2S, voir [Créer une connexion VPN S2S](vpn-gateway-create-site-to-site-rm-powershell.md).

### <a name="before-you-begin"></a><a name="before"></a>Avant de commencer

* Assurez-vous de disposer d’un abonnement Azure. Si vous ne disposez pas déjà d’un abonnement Azure, vous pouvez activer vos [avantages abonnés MSDN](https://azure.microsoft.com/pricing/member-offers/msdn-benefits-details/) ou créer un [compte gratuit](https://azure.microsoft.com/pricing/free-trial/).
* Installez les applets de commande PowerShell Azure Resource Manager. Consultez [Présentation d’Azure PowerShell](/powershell/azure/) pour plus d’informations sur l’installation des applets de commande PowerShell.

### <a name="step-1---create-the-virtual-network-vpn-gateway-and-local-network-gateway"></a><a name="createvnet1"></a>Étape 1 : création du réseau virtuel, de la passerelle VPN et de la passerelle de réseau local

#### <a name="1-declare-your-variables"></a>1. Déclarer vos variables

Dans cet exercice, nous allons commencer par déclarer les variables. Veillez à les remplacer par vos valeurs lors de la configuration dans un contexte de production.

```powershell
$Sub1          = "<YourSubscriptionName>"
$RG1           = "TestPolicyRG1"
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
$DNS1          = "8.8.8.8"
$GWName1       = "VNet1GW"
$GW1IPName1    = "VNet1GWIP1"
$GW1IPconf1    = "gw1ipconf1"
$Connection16  = "VNet1toSite6"

$LNGName6      = "Site6"
$LNGPrefix61   = "10.61.0.0/16"
$LNGPrefix62   = "10.62.0.0/16"
$LNGIP6        = "131.107.72.22"
```

#### <a name="2-connect-to-your-subscription-and-create-a-new-resource-group"></a>2. Se connecter à votre abonnement et créer un groupe de ressources

Pour utiliser les applets de commande Resource Manager, passez au mode PowerShell. Pour plus d’informations, consultez la page [Utilisation de Windows PowerShell avec Resource Manager](../azure-resource-manager/management/manage-resources-powershell.md).

Ouvrez la console PowerShell et connectez-vous à votre compte. Utilisez l’exemple suivant pour faciliter votre connexion :

```powershell
Connect-AzAccount
Select-AzSubscription -SubscriptionName $Sub1
New-AzResourceGroup -Name $RG1 -Location $Location1
```

#### <a name="3-create-the-virtual-network-vpn-gateway-and-local-network-gateway"></a>3. Créer le réseau virtuel, la passerelle VPN et la passerelle de réseau local

L’exemple suivant crée le réseau virtuel, TestVNet1, avec trois sous-réseaux et la passerelle VPN. Lorsque vous remplacez les valeurs, pensez à toujours nommer votre sous-réseau de passerelle « GatewaySubnet ». Si vous le nommez autrement, la création de votre passerelle échoue.

```powershell
$fesub1 = New-AzVirtualNetworkSubnetConfig -Name $FESubName1 -AddressPrefix $FESubPrefix1
$besub1 = New-AzVirtualNetworkSubnetConfig -Name $BESubName1 -AddressPrefix $BESubPrefix1
$gwsub1 = New-AzVirtualNetworkSubnetConfig -Name $GWSubName1 -AddressPrefix $GWSubPrefix1

New-AzVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1 -Location $Location1 -AddressPrefix $VNetPrefix11,$VNetPrefix12 -Subnet $fesub1,$besub1,$gwsub1

$gw1pip1    = New-AzPublicIpAddress -Name $GW1IPName1 -ResourceGroupName $RG1 -Location $Location1 -AllocationMethod Dynamic
$vnet1      = Get-AzVirtualNetwork -Name $VNetName1 -ResourceGroupName $RG1
$subnet1    = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet1
$gw1ipconf1 = New-AzVirtualNetworkGatewayIpConfig -Name $GW1IPconf1 -Subnet $subnet1 -PublicIpAddress $gw1pip1

New-AzVirtualNetworkGateway -Name $GWName1 -ResourceGroupName $RG1 -Location $Location1 -IpConfigurations $gw1ipconf1 -GatewayType Vpn -VpnType RouteBased -GatewaySku VpnGw1

New-AzLocalNetworkGateway -Name $LNGName6 -ResourceGroupName $RG1 -Location $Location1 -GatewayIpAddress $LNGIP6 -AddressPrefix $LNGPrefix61,$LNGPrefix62
```

### <a name="step-2---create-a-s2s-vpn-connection-with-an-ipsecike-policy"></a><a name="s2sconnection"></a>Étape 2 : création d’une connexion VPN S2S avec une stratégie IPsec/IKE

#### <a name="1-create-an-ipsecike-policy"></a>1. Créez une stratégie IPsec/IKE.

L’exemple de script ci-dessous crée une stratégie IPsec/IKE avec les paramètres et les algorithmes suivants :

* IKEv2 : AES256, SHA384, DHGroup24
* IPsec : AES256, SHA256, PFS None, SA Lifetime 14400 seconds & 102400000KB

```powershell
$ipsecpolicy6 = New-AzIpsecPolicy -IkeEncryption AES256 -IkeIntegrity SHA384 -DhGroup DHGroup24 -IpsecEncryption AES256 -IpsecIntegrity SHA256 -PfsGroup None -SALifeTimeSeconds 14400 -SADataSizeKilobytes 102400000
```

Si vous utilisez des algorithmes GCMAES pour IPsec, vous devez utiliser les mêmes algorithme GCMAES et longueur de clé pour le chiffrement IPsec et l’intégrité IPsec. Ainsi, dans l’exemple ci-dessus, les paramètres correspondants seront « -IpsecEncryption GCMAES256 - IpsecIntegrity GCMAES256 » en cas d’utilisation de GCMAES256.

#### <a name="2-create-the-s2s-vpn-connection-with-the-ipsecike-policy"></a>2. Créer la connexion VPN S2S avec la stratégie IPsec/IKE

Créer une connexion VPN S2S et appliquer la stratégie IPsec/IKE créée précédemment.

```powershell
$vnet1gw = Get-AzVirtualNetworkGateway -Name $GWName1  -ResourceGroupName $RG1
$lng6 = Get-AzLocalNetworkGateway  -Name $LNGName6 -ResourceGroupName $RG1

New-AzVirtualNetworkGatewayConnection -Name $Connection16 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -LocalNetworkGateway2 $lng6 -Location $Location1 -ConnectionType IPsec -IpsecPolicies $ipsecpolicy6 -SharedKey 'AzureA1b2C3'
```

Vous pouvez éventuellement ajouter « -UsePolicyBasedTrafficSelectors $True » à l’applet de commande créant la connexion afin de permettre à la passerelle VPN Azure de se connecter à des périphériques VPN basés sur une stratégie en local, comme décrit ci-dessus.

> [!IMPORTANT]
> Une fois qu’une stratégie IPsec/IKE est spécifiée sur une connexion, la passerelle VPN Azure ne fait qu’envoyer ou accepter la proposition IPsec/IKE avec les algorithmes de chiffrement et puissances de clé spécifiés sur cette connexion particulière. Assurez-vous que votre périphérique VPN local pour la connexion utilise ou accepte la combinaison de stratégies exacte. À défaut, le tunnel VPN S2S ne peut pas être établi.


## <a name="part-4---create-a-new-vnet-to-vnet-connection-with-ipsecike-policy"></a><a name ="vnet2vnet"></a>Partie 4 : création d’une connexion de réseau virtuel à réseau virtuel avec une stratégie IPsec/IKE

Les étapes de création d’une connexion de réseau virtuel à réseau virtuel avec une stratégie IPsec/IKE sont similaires à celles d’une connexion VPN S2S. Les exemples de scripts ci-dessous créent la connexion comme illustré dans le diagramme :

![Stratégie de réseau virtuel à réseau virtuel](./media/vpn-gateway-ipsecikepolicy-rm-powershell/v2vpolicy.png)

Pour plus d’informations sur les étapes de création d’une connexion de réseau virtuel à réseau virtuel, voir [Créer une connexion de réseau virtuel à réseau virtuel](vpn-gateway-vnet-vnet-rm-ps.md). Pour créer et configurer TestVNet1 et la passerelle VPN, vous devez procéder de la manière décrite dans la [Partie 3](#crossprem).

### <a name="step-1---create-the-second-virtual-network-and-vpn-gateway"></a><a name="createvnet2"></a>Étape 1 : création du deuxième réseau virtuel et de la passerelle VPN

#### <a name="1-declare-your-variables"></a>1. Déclarer vos variables

Veillez à remplacer les valeurs par celles que vous souhaitez utiliser pour votre configuration.

```powershell
$RG2          = "TestPolicyRG2"
$Location2    = "East US 2"
$VNetName2    = "TestVNet2"
$FESubName2   = "FrontEnd"
$BESubName2   = "Backend"
$GWSubName2   = "GatewaySubnet"
$VNetPrefix21 = "10.21.0.0/16"
$VNetPrefix22 = "10.22.0.0/16"
$FESubPrefix2 = "10.21.0.0/24"
$BESubPrefix2 = "10.22.0.0/24"
$GWSubPrefix2 = "10.22.255.0/27"
$DNS2         = "8.8.8.8"
$GWName2      = "VNet2GW"
$GW2IPName1   = "VNet2GWIP1"
$GW2IPconf1   = "gw2ipconf1"
$Connection21 = "VNet2toVNet1"
$Connection12 = "VNet1toVNet2"
```

#### <a name="2-create-the-second-virtual-network-and-vpn-gateway-in-the-new-resource-group"></a>2. Créer le deuxième réseau virtuel et la passerelle VPN dans le nouveau groupe de ressources

```powershell
New-AzResourceGroup -Name $RG2 -Location $Location2

$fesub2 = New-AzVirtualNetworkSubnetConfig -Name $FESubName2 -AddressPrefix $FESubPrefix2
$besub2 = New-AzVirtualNetworkSubnetConfig -Name $BESubName2 -AddressPrefix $BESubPrefix2
$gwsub2 = New-AzVirtualNetworkSubnetConfig -Name $GWSubName2 -AddressPrefix $GWSubPrefix2

New-AzVirtualNetwork -Name $VNetName2 -ResourceGroupName $RG2 -Location $Location2 -AddressPrefix $VNetPrefix21,$VNetPrefix22 -Subnet $fesub2,$besub2,$gwsub2

$gw2pip1    = New-AzPublicIpAddress -Name $GW2IPName1 -ResourceGroupName $RG2 -Location $Location2 -AllocationMethod Dynamic
$vnet2      = Get-AzVirtualNetwork -Name $VNetName2 -ResourceGroupName $RG2
$subnet2    = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet2
$gw2ipconf1 = New-AzVirtualNetworkGatewayIpConfig -Name $GW2IPconf1 -Subnet $subnet2 -PublicIpAddress $gw2pip1

New-AzVirtualNetworkGateway -Name $GWName2 -ResourceGroupName $RG2 -Location $Location2 -IpConfigurations $gw2ipconf1 -GatewayType Vpn -VpnType RouteBased -GatewaySku HighPerformance
```

### <a name="step-2---create-a-vnet-tovnet-connection-with-the-ipsecike-policy"></a>Étape 2 : création d’une connexion de réseau virtuel à réseau virtuel avec la stratégie IPsec/IKE

Comme pour la connexion VPN S2S, créez une stratégie IPsec/IKE, puis appliquez-la à la nouvelle connexion.

#### <a name="1-create-an-ipsecike-policy"></a>1. Créez une stratégie IPsec/IKE.

L’exemple de script ci-dessous crée une stratégie IPsec/IKE différente avec les algorithmes et paramètres suivants :
* IKEv2 : AES128, SHA1, DHGroup14
* IPsec : GCMAES128, GCMAES128, PFS14, SA Lifetime 14400 seconds & 102400000KB

```powershell
$ipsecpolicy2 = New-AzIpsecPolicy -IkeEncryption AES128 -IkeIntegrity SHA1 -DhGroup DHGroup14 -IpsecEncryption GCMAES128 -IpsecIntegrity GCMAES128 -PfsGroup PFS14 -SALifeTimeSeconds 14400 -SADataSizeKilobytes 102400000
```

#### <a name="2-create-vnet-to-vnet-connections-with-the-ipsecike-policy"></a>2. Créer des connexions de réseau virtuel à réseau virtuel avec la stratégie IPsec/IKE

Créez une connexion de réseau virtuel à réseau virtuel, et appliquez la stratégie IPsec/IKE créée. Dans cet exemple, les deux passerelles sont dans le même abonnement. Il est donc possible de créer et configurer les deux connexions avec la même stratégie IPsec/IKE dans la même session PowerShell.

```powershell
$vnet1gw = Get-AzVirtualNetworkGateway -Name $GWName1  -ResourceGroupName $RG1
$vnet2gw = Get-AzVirtualNetworkGateway -Name $GWName2  -ResourceGroupName $RG2

New-AzVirtualNetworkGatewayConnection -Name $Connection12 -ResourceGroupName $RG1 -VirtualNetworkGateway1 $vnet1gw -VirtualNetworkGateway2 $vnet2gw -Location $Location1 -ConnectionType Vnet2Vnet -IpsecPolicies $ipsecpolicy2 -SharedKey 'AzureA1b2C3'

New-AzVirtualNetworkGatewayConnection -Name $Connection21 -ResourceGroupName $RG2 -VirtualNetworkGateway1 $vnet2gw -VirtualNetworkGateway2 $vnet1gw -Location $Location2 -ConnectionType Vnet2Vnet -IpsecPolicies $ipsecpolicy2 -SharedKey 'AzureA1b2C3'
```

> [!IMPORTANT]
> Une fois qu’une stratégie IPsec/IKE est spécifiée sur une connexion, la passerelle VPN Azure ne fait qu’envoyer ou accepter la proposition IPsec/IKE avec les algorithmes de chiffrement et puissances de clé spécifiés sur cette connexion particulière. Assurez-vous que les stratégies IPsec pour les deux connexions sont identiques. À défaut, la connexion de réseau virtuel à réseau virtuel ne peut pas être établie.

Une fois ces étapes accomplies, la connexion est établie en quelques minutes et vous disposez de la topologie de réseau suivante, comme indiqué au début :

![stratégie IPSec/IKE](./media/vpn-gateway-ipsecikepolicy-rm-powershell/ipsecikepolicy.png)


## <a name="part-5---update-ipsecike-policy-for-a-connection"></a><a name ="managepolicy"></a>Partie 5 : mise à jour de la stratégie IPsec/IKE pour une connexion

La dernière section présente la gestion de la stratégie IPsec/IKE pour une connexion S2S ou de réseau virtuel à réseau virtuel existante. L’exercice ci-dessous vous guide dans les opérations suivantes sur une connexion :

1. Afficher la stratégie IPsec/IKE d’une connexion
2. Ajouter la stratégie IPsec/IKE à une connexion ou la mettre à jour
3. Supprimer la stratégie IPsec/IKE d’une connexion

Les mêmes étapes s’appliquent aux connexions S2S et de réseau virtuel à réseau virtuel.

> [!IMPORTANT]
> Une stratégie IPsec/IKE est prise en charge uniquement sur des passerelles VPN *Standard* et *HighPerformance* basées sur itinéraires. Elle ne fonctionne pas sur la référence (SKU) de passerelle de base ou la passerelle VPN basée sur une stratégie.

#### <a name="1-show-the-ipsecike-policy-of-a-connection"></a>1. Afficher la stratégie IPsec/IKE d’une connexion

L’exemple suivant montre comment configurer la stratégie IPsec/IKE sur une connexion. Les scripts des exercices précédents continuent également.

```powershell
$RG1          = "TestPolicyRG1"
$Connection16 = "VNet1toSite6"
$connection6  = Get-AzVirtualNetworkGatewayConnection -Name $Connection16 -ResourceGroupName $RG1
$connection6.IpsecPolicies
```

La dernière commande affiche la stratégie IPsec/IKE actuelle éventuelle configurée sur la connexion. Voici un exemple de sortie pour la connexion :

```powershell
SALifeTimeSeconds   : 14400
SADataSizeKilobytes : 102400000
IpsecEncryption     : AES256
IpsecIntegrity      : SHA256
IkeEncryption       : AES256
IkeIntegrity        : SHA384
DhGroup             : DHGroup24
PfsGroup            : PFS24
```

Si aucune stratégie IPsec/IKE n’est configurée, la commande (PS>$connection6.IpsecPolicies) obtient un retour vide. Cela ne signifie pas qu’une stratégie IPsec/IKE n’est pas configurée sur la connexion, mais qu’il n’existe pas de stratégie IPsec/IKE personnalisée. La connexion réelle utilise la stratégie par défaut négociée entre votre périphérique VPN local et la passerelle VPN Azure.

#### <a name="2-add-or-update-an-ipsecike-policy-for-a-connection"></a>2. Ajouter ou mettre à jour une stratégie IPsec/IKE pour une connexion

Les étapes d’ajout ou de mise à jour d’une stratégie sur une connexion sont identiques : créer une stratégie, puis l’appliquer à la connexion.

```powershell
$RG1          = "TestPolicyRG1"
$Connection16 = "VNet1toSite6"
$connection6  = Get-AzVirtualNetworkGatewayConnection -Name $Connection16 -ResourceGroupName $RG1

$newpolicy6   = New-AzIpsecPolicy -IkeEncryption AES128 -IkeIntegrity SHA1 -DhGroup DHGroup14 -IpsecEncryption AES256 -IpsecIntegrity SHA256 -PfsGroup None -SALifeTimeSeconds 14400 -SADataSizeKilobytes 102400000

Set-AzVirtualNetworkGatewayConnection -VirtualNetworkGatewayConnection $connection6 -IpsecPolicies $newpolicy6
```

Pour activer « UsePolicyBasedTrafficSelectors » lors de la connexion à un périphérique VPN local basé sur une stratégie, ajoutez le paramètre « -UsePolicyBaseTrafficSelectors » à l’applet de commande, ou définissez-le sur $False pour désactiver l’option :

```powershell
Set-AzVirtualNetworkGatewayConnection -VirtualNetworkGatewayConnection $connection6 -IpsecPolicies $newpolicy6 -UsePolicyBasedTrafficSelectors $True
```

Vous pouvez obtenir de nouveau la connexion pour vérifier si la stratégie est mise à jour.

```powershell
$connection6  = Get-AzVirtualNetworkGatewayConnection -Name $Connection16 -ResourceGroupName $RG1
$connection6.IpsecPolicies
```

La sortie de la dernière ligne devrait être celle illustrée dans l’exemple suivant :

```powershell
SALifeTimeSeconds   : 14400
SADataSizeKilobytes : 102400000
IpsecEncryption     : AES256
IpsecIntegrity      : SHA256
IkeEncryption       : AES128
IkeIntegrity        : SHA1
DhGroup             : DHGroup14
PfsGroup            : None
```

#### <a name="3-remove-an-ipsecike-policy-from-a-connection"></a>3. Supprimer une stratégie IPsec/IKE d’une connexion

Après que vous avez supprimé la stratégie personnalisée d’une connexion, la passerelle VPN Azure revient à la [liste par défaut des propositions IPsec/IKE](vpn-gateway-about-vpn-devices.md) et relance la négociation avec votre périphérique VPN local.

```powershell
$RG1           = "TestPolicyRG1"
$Connection16  = "VNet1toSite6"
$connection6   = Get-AzVirtualNetworkGatewayConnection -Name $Connection16 -ResourceGroupName $RG1

$currentpolicy = $connection6.IpsecPolicies[0]
$connection6.IpsecPolicies.Remove($currentpolicy)

Set-AzVirtualNetworkGatewayConnection -VirtualNetworkGatewayConnection $connection6
```

Vous pouvez utiliser ce script pour vérifier si la stratégie a été supprimée de la connexion.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur les sélecteurs de trafic basés sur une stratégie, voir [Connecter plusieurs périphériques VPN basés sur un stratégie locale](vpn-gateway-connect-multiple-policybased-rm-ps.md).

Une fois la connexion achevée, vous pouvez ajouter des machines virtuelles à vos réseaux virtuels. Consultez [Création d’une machine virtuelle](../virtual-machines/windows/quick-create-portal.md) pour connaître les différentes étapes.