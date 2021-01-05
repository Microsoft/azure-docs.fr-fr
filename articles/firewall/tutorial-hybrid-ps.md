---
title: Déployer et configurer un Pare-feu Azure dans un réseau hybride à l’aide de PowerShell
description: Dans cet article, vous apprendrez à déployer et configurer un pare-feu Azure à l'aide d'Azure PowerShell.
services: firewall
author: vhorne
ms.service: firewall
ms.topic: how-to
ms.date: 08/28/2020
ms.author: victorh
customer intent: As an administrator, I want to control network access from an on-premises network to an Azure virtual network.
ms.openlocfilehash: e60c829831bde3b454ab180d1a39ec46cb346963
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94658636"
---
# <a name="deploy-and-configure-azure-firewall-in-a-hybrid-network-using-azure-powershell"></a>Déployer et configurer un pare-feu Azure dans un réseau hybride à l’aide d’Azure PowerShell

Lorsque vous connectez votre réseau local à un réseau virtuel Azure pour créer un réseau hybride, la possibilité de contrôler l’accès à vos ressources réseau Azure représente une part importante dans un plan de sécurité générale.

Vous pouvez utiliser le Pare-feu Azure pour contrôler l’accès réseau d’un réseau hybride à l’aide de règles définissant le trafic réseau autorisé et refusé.

Pour cet article, vous créerez trois réseaux virtuels :

- **VNet-Hub** : Le pare-feu se trouve dans ce réseau virtuel.
- **VNet-Spoke** : Le réseau virtuel spoke correspond à la charge de travail sur Azure.
- **VNet-Onprem** : Le réseau virtuel local représente un réseau local. Dans un déploiement réel, il peut être connecté via un VPN ou une connexion ExpressRoute. Par souci de simplicité, cet article utilise une connexion de passerelle VPN, et un réseau virtuel Azure est utilisé pour représenter un réseau local.

![Pare-feu dans un réseau hybride](media/tutorial-hybrid-ps/hybrid-network-firewall.png)

Dans cet article, vous apprendrez comment :

* Déclarer les variables
* Créer le réseau virtuel du hub de pare-feu
* Créer le réseau virtuel spoke
* Créer le réseau virtuel local
* Configurer et déployer le pare-feu
* Créer et connecter les passerelles VPN
* Appairer les réseaux virtuels hub et spoke
* Créer les itinéraires
* Créer les machines virtuelles
* Tester le pare-feu

Si vous préférez utiliser le portail Azure pour suivre ce didacticiel, reportez-vous à [Didacticiel : Déployer et configurer un Pare-feu Azure dans un réseau hybride à l'aide du portail Azure](tutorial-hybrid-portal.md).

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="prerequisites"></a>Prérequis

Pour cet article, vous devez exécuter PowerShell localement. Les modules Azure PowerShell doivent être installés. Exécutez `Get-Module -ListAvailable Az` pour trouver la version. Si vous devez effectuer une mise à niveau, consultez [Installer le module Azure PowerShell](/powershell/azure/install-Az-ps). Après avoir vérifié la version PowerShell, exécutez `Login-AzAccount` pour créer une connexion avec Azure.

Il existe trois conditions clés pour que ce scénario fonctionne correctement :

- Un itinéraire défini par l’utilisateur (UDR) sur le sous-réseau spoke qui pointe vers l’adresse IP du Pare-feu Azure en tant que passerelle par défaut. La propagation de l’itinéraire de la passerelle de réseau virtuel doit être **désactivée** sur cette table de routage.
- Un UDR sur le sous-réseau de passerelle hub doit pointer vers l’adresse IP du pare-feu comme prochain tronçon pour les réseaux spoke.

   Aucun UDR n’est requis sur le sous-réseau du Pare-feu Azure, puisqu’il apprend les itinéraires à partir de BGP.
- Assurez-vous de définir **AllowGatewayTransit** lors du peering de VNet-Hub avec VNet-Spoke et **UseRemoteGateways** lors du peering de VNet-Spoke avec VNet-Hub.

Pour plus d'informations sur la création de ces itinéraires, consultez la section [Créer des itinéraires](#create-the-routes) de cet article.

>[!NOTE]
>Le Pare-feu Azure doit avoir une connectivité Internet directe. Si votre AzureFirewallSubnet prend connaissance d’un itinéraire par défaut pour votre réseau local via le protocole BGP, vous devez le remplacer par un UDR 0.0.0.0/0 avec la valeur **NextHopType** définie sur **Internet** pour garantir une connectivité Internet directe.
>
>Le Pare-feu Azure peut être configuré pour prendre en charge le tunneling forcé. Pour plus d’informations, consultez la page [Tunneling forcé du Pare-feu Azure](forced-tunneling.md).

>[!NOTE]
>Le trafic entre les réseaux virtuels directement appairés est acheminé directement même si l’UDR pointe vers le Pare-feu Azure en tant que passerelle par défaut. Pour envoyer un trafic de sous-réseau à sous-réseau au pare-feu dans ce scénario, un UDR doit contenir explicitement le préfixe du réseau cible dans les deux sous-réseaux.

Pour consulter la documentation de référence d’Azure PowerShell concernée, consultez [Informations de référence sur Azure PowerShell](/powershell/module/az.network/new-azfirewall).

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

## <a name="declare-the-variables"></a>Déclarer les variables

L'exemple suivant déclare les variables à l'aide des valeurs de cet article. Dans certains cas, vous devrez peut-être remplacer certaines valeurs par les vôtres pour travailler dans votre abonnement. Si besoin, modifiez les variables, puis copiez et collez-les dans la console PowerShell.

```azurepowershell
$RG1 = "FW-Hybrid-Test"
$Location1 = "East US"

# Variables for the firewall hub VNet

$VNetnameHub = "VNet-hub"
$SNnameHub = "AzureFirewallSubnet"
$VNetHubPrefix = "10.5.0.0/16"
$SNHubPrefix = "10.5.0.0/24"
$SNGWHubPrefix = "10.5.1.0/24"
$GWHubName = "GW-hub"
$GWHubpipName = "VNet-hub-GW-pip"
$GWIPconfNameHub = "GW-ipconf-hub"
$ConnectionNameHub = "hub-to-Onprem"

# Variables for the spoke virtual network

$VnetNameSpoke = "VNet-Spoke"
$SNnameSpoke = "SN-Workload"
$VNetSpokePrefix = "10.6.0.0/16"
$SNSpokePrefix = "10.6.0.0/24"
$SNSpokeGWPrefix = "10.6.1.0/24"

# Variables for the on-premises virtual network

$VNetnameOnprem = "Vnet-Onprem"
$SNNameOnprem = "SN-Corp"
$VNetOnpremPrefix = "192.168.0.0/16"
$SNOnpremPrefix = "192.168.1.0/24"
$SNGWOnpremPrefix = "192.168.2.0/24"
$GWOnpremName = "GW-Onprem"
$GWIPconfNameOnprem = "GW-ipconf-Onprem"
$ConnectionNameOnprem = "Onprem-to-hub"
$GWOnprempipName = "VNet-Onprem-GW-pip"

$SNnameGW = "GatewaySubnet"
```


## <a name="create-the-firewall-hub-virtual-network"></a>Créer le réseau virtuel du hub de pare-feu

Commencez par créer le groupe de ressources qui doit contenir les ressources de cet article :

```azurepowershell
  New-AzResourceGroup -Name $RG1 -Location $Location1
  ```

Définissez les sous-réseaux à inclure dans le réseau virtuel :

```azurepowershell
$FWsub = New-AzVirtualNetworkSubnetConfig -Name $SNnameHub -AddressPrefix $SNHubPrefix
$GWsub = New-AzVirtualNetworkSubnetConfig -Name $SNnameGW -AddressPrefix $SNGWHubPrefix
```

À présent, créez le réseau virtuel du hub de pare-feu :

```azurepowershell
$VNetHub = New-AzVirtualNetwork -Name $VNetnameHub -ResourceGroupName $RG1 `
-Location $Location1 -AddressPrefix $VNetHubPrefix -Subnet $FWsub,$GWsub
```

Demandez l’allocation d’une adresse IP publique destinée à la passerelle VPN que vous allez créer pour votre réseau virtuel. Notez que la valeur *AllocationMethod* est **dynamique**. Vous ne pouvez pas spécifier l’adresse IP que vous souhaitez utiliser. Elle est allouée à votre passerelle VPN de façon dynamique.

  ```azurepowershell
  $gwpip1 = New-AzPublicIpAddress -Name $GWHubpipName -ResourceGroupName $RG1 `
  -Location $Location1 -AllocationMethod Dynamic
```

## <a name="create-the-spoke-virtual-network"></a>Créer le réseau virtuel spoke

Définissez les sous-réseaux à inclure dans le réseau virtuel spoke :

```azurepowershell
$Spokesub = New-AzVirtualNetworkSubnetConfig -Name $SNnameSpoke -AddressPrefix $SNSpokePrefix
$GWsubSpoke = New-AzVirtualNetworkSubnetConfig -Name $SNnameGW -AddressPrefix $SNSpokeGWPrefix
```

Créez le réseau virtuel spoke :

```azurepowershell
$VNetSpoke = New-AzVirtualNetwork -Name $VnetNameSpoke -ResourceGroupName $RG1 `
-Location $Location1 -AddressPrefix $VNetSpokePrefix -Subnet $Spokesub,$GWsubSpoke
```

## <a name="create-the-on-premises-virtual-network"></a>Créer le réseau virtuel local

Définissez les sous-réseaux à inclure dans le réseau virtuel :

```azurepowershell
$Onpremsub = New-AzVirtualNetworkSubnetConfig -Name $SNNameOnprem -AddressPrefix $SNOnpremPrefix
$GWOnpremsub = New-AzVirtualNetworkSubnetConfig -Name $SNnameGW -AddressPrefix $SNGWOnpremPrefix
```

À présent, créez le réseau virtuel local :

```azurepowershell
$VNetOnprem = New-AzVirtualNetwork -Name $VNetnameOnprem -ResourceGroupName $RG1 `
-Location $Location1 -AddressPrefix $VNetOnpremPrefix -Subnet $Onpremsub,$GWOnpremsub
```

Demandez l’allocation d’une adresse IP publique destinée à la passerelle que vous allez créer pour le réseau virtuel. Notez que la valeur *AllocationMethod* est **dynamique**. Vous ne pouvez pas spécifier l’adresse IP que vous souhaitez utiliser. Elle est allouée à votre passerelle de façon dynamique.

  ```azurepowershell
  $gwOnprempip = New-AzPublicIpAddress -Name $GWOnprempipName -ResourceGroupName $RG1 `
  -Location $Location1 -AllocationMethod Dynamic
```

## <a name="configure-and-deploy-the-firewall"></a>Configurer et déployer le pare-feu

Déployez maintenant le pare-feu dans le réseau virtuel hub.

```azurepowershell
# Get a Public IP for the firewall
$FWpip = New-AzPublicIpAddress -Name "fw-pip" -ResourceGroupName $RG1 `
  -Location $Location1 -AllocationMethod Static -Sku Standard
# Create the firewall
$Azfw = New-AzFirewall -Name AzFW01 -ResourceGroupName $RG1 -Location $Location1 -VirtualNetworkName $VNetnameHub -PublicIpName fw-pip

#Save the firewall private IP address for future use

$AzfwPrivateIP = $Azfw.IpConfigurations.privateipaddress
$AzfwPrivateIP

```

### <a name="configure-network-rules"></a>Configurer des règles de réseau

<!--- $Rule3 = New-AzFirewallNetworkRule -Name "AllowPing" -Protocol ICMP -SourceAddress $SNOnpremPrefix `
   -DestinationAddress $VNetSpokePrefix -DestinationPort *--->

```azurepowershell
$Rule1 = New-AzFirewallNetworkRule -Name "AllowWeb" -Protocol TCP -SourceAddress $SNOnpremPrefix `
   -DestinationAddress $VNetSpokePrefix -DestinationPort 80

$Rule2 = New-AzFirewallNetworkRule -Name "AllowRDP" -Protocol TCP -SourceAddress $SNOnpremPrefix `
   -DestinationAddress $VNetSpokePrefix -DestinationPort 3389

$NetRuleCollection = New-AzFirewallNetworkRuleCollection -Name RCNet01 -Priority 100 `
   -Rule $Rule1,$Rule2 -ActionType "Allow"
$Azfw.NetworkRuleCollections = $NetRuleCollection
Set-AzFirewall -AzureFirewall $Azfw
```

## <a name="create-and-connect-the-vpn-gateways"></a>Créer et connecter les passerelles VPN

Les réseaux virtuels hub et local sont connectés via des passerelles VPN.

### <a name="create-a-vpn-gateway-for-the-hub-virtual-network"></a>Créer une passerelle VPN pour le réseau virtuel hub

Créez la configuration de la passerelle VPN. La configuration de la passerelle VPN définit le sous-réseau et l’adresse IP publique à utiliser.

  ```azurepowershell
  $vnet1 = Get-AzVirtualNetwork -Name $VNetnameHub -ResourceGroupName $RG1
  $subnet1 = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet1
  $gwipconf1 = New-AzVirtualNetworkGatewayIpConfig -Name $GWIPconfNameHub `
  -Subnet $subnet1 -PublicIpAddress $gwpip1
  ```

Maintenant, créez la passerelle VPN pour le réseau virtuel hub. Les configurations de réseau virtuel à réseau virtuel nécessitent un VPN de type RouteBased. La création d’une passerelle VPN nécessite généralement au moins 45 minutes, selon la référence SKU de passerelle VPN sélectionnée.

```azurepowershell
New-AzVirtualNetworkGateway -Name $GWHubName -ResourceGroupName $RG1 `
-Location $Location1 -IpConfigurations $gwipconf1 -GatewayType Vpn `
-VpnType RouteBased -GatewaySku basic
```

### <a name="create-a-vpn-gateway-for-the-on-premises-virtual-network"></a>Créer une passerelle VPN pour le réseau virtuel local

Créez la configuration de la passerelle VPN. La configuration de la passerelle VPN définit le sous-réseau et l’adresse IP publique à utiliser.

  ```azurepowershell
$vnet2 = Get-AzVirtualNetwork -Name $VNetnameOnprem -ResourceGroupName $RG1
$subnet2 = Get-AzVirtualNetworkSubnetConfig -Name "GatewaySubnet" -VirtualNetwork $vnet2
$gwipconf2 = New-AzVirtualNetworkGatewayIpConfig -Name $GWIPconfNameOnprem `
  -Subnet $subnet2 -PublicIpAddress $gwOnprempip
  ```

À présent, créez la passerelle VPN pour le réseau virtuel local. Les configurations de réseau virtuel à réseau virtuel nécessitent un VPN de type RouteBased. La création d’une passerelle VPN nécessite généralement au moins 45 minutes, selon la référence SKU de passerelle VPN sélectionnée.

```azurepowershell
New-AzVirtualNetworkGateway -Name $GWOnpremName -ResourceGroupName $RG1 `
-Location $Location1 -IpConfigurations $gwipconf2 -GatewayType Vpn `
-VpnType RouteBased -GatewaySku basic
```

### <a name="create-the-vpn-connections"></a>Créer les connexions VPN

Vous pouvez maintenant créer les connexions VPN entre les passerelles hub et local.

#### <a name="get-the-vpn-gateways"></a>Obtenir les passerelles VPN

```azurepowershell
$vnetHubgw = Get-AzVirtualNetworkGateway -Name $GWHubName -ResourceGroupName $RG1
$vnetOnpremgw = Get-AzVirtualNetworkGateway -Name $GWOnpremName -ResourceGroupName $RG1
```

#### <a name="create-the-connections"></a>Créer les connexions

Dans cette étape, vous créez la connexion entre le réseau virtuel hub et le réseau virtuel local. Une clé partagée est référencée dans les exemples. Vous pouvez utiliser vos propres valeurs pour cette clé partagée. Il est important que la clé partagée corresponde aux deux connexions. La création d’une connexion peut prendre quelques instants.

```azurepowershell
New-AzVirtualNetworkGatewayConnection -Name $ConnectionNameHub -ResourceGroupName $RG1 `
-VirtualNetworkGateway1 $vnetHubgw -VirtualNetworkGateway2 $vnetOnpremgw -Location $Location1 `
-ConnectionType Vnet2Vnet -SharedKey 'AzureA1b2C3'
```
Créez la connexion entre les réseaux virtuels hub et local. Cette étape est similaire à la précédente, sauf que vous créez la connexion du réseau virtuel OnPrem vers le réseau virtuel hub. Vérifiez que les clés partagées correspondent. Après quelques minutes, la connexion est établie.

  ```azurepowershell
  New-AzVirtualNetworkGatewayConnection -Name $ConnectionNameOnprem -ResourceGroupName $RG1 `
  -VirtualNetworkGateway1 $vnetOnpremgw -VirtualNetworkGateway2 $vnetHubgw -Location $Location1 `
  -ConnectionType Vnet2Vnet -SharedKey 'AzureA1b2C3'
  ```

#### <a name="verify-the-connection"></a>Vérifier la connexion

Vous pouvez vérifier la réussite de la connexion via l’applet de commande *Get-AzVirtualNetworkGatewayConnection*, avec ou sans *-Debug*. Utilisez l’exemple d’applet de commande suivant, en configurant les valeurs sur les vôtres. Si vous y êtes invité, sélectionnez **A** pour exécuter **tout**. Dans l’exemple, *-Name* fait référence au nom de la connexion que vous voulez tester.

```azurepowershell
Get-AzVirtualNetworkGatewayConnection -Name $ConnectionNameHub -ResourceGroupName $RG1
```

Une fois la cmdlet exécutée, affichez les valeurs. Dans l’exemple ci-dessous, l’état de la connexion indique *Connecté* et vous pouvez voir les octets d’entrée et de sortie.

```
"connectionStatus": "Connected",
"ingressBytesTransferred": 33509044,
"egressBytesTransferred": 4142431
```

## <a name="peer-the-hub-and-spoke-virtual-networks"></a>Appairer les réseaux virtuels hub et spoke

À présent, appairez les réseaux virtuels hub et spoke.

```azurepowershell
# Peer hub to spoke
Add-AzVirtualNetworkPeering -Name HubtoSpoke -VirtualNetwork $VNetHub -RemoteVirtualNetworkId $VNetSpoke.Id -AllowGatewayTransit

# Peer spoke to hub
Add-AzVirtualNetworkPeering -Name SpoketoHub -VirtualNetwork $VNetSpoke -RemoteVirtualNetworkId $VNetHub.Id -AllowForwardedTraffic -UseRemoteGateways
```

## <a name="create-the-routes"></a>Créer les itinéraires

Ensuite, créez deux itinéraires :

- Un itinéraire à partir du sous-réseau de passerelle hub vers le sous-réseau spoke via l’adresse IP du pare-feu
- Un itinéraire par défaut à partir du sous-réseau spoke via l’adresse IP du pare-feu

```azurepowershell
#Create a route table
$routeTableHubSpoke = New-AzRouteTable `
  -Name 'UDR-Hub-Spoke' `
  -ResourceGroupName $RG1 `
  -location $Location1

#Create a route
Get-AzRouteTable `
  -ResourceGroupName $RG1 `
  -Name UDR-Hub-Spoke `
  | Add-AzRouteConfig `
  -Name "ToSpoke" `
  -AddressPrefix $VNetSpokePrefix `
  -NextHopType "VirtualAppliance" `
  -NextHopIpAddress $AzfwPrivateIP `
 | Set-AzRouteTable

#Associate the route table to the subnet

Set-AzVirtualNetworkSubnetConfig `
  -VirtualNetwork $VNetHub `
  -Name $SNnameGW `
  -AddressPrefix $SNGWHubPrefix `
  -RouteTable $routeTableHubSpoke | `
Set-AzVirtualNetwork

#Now create the default route

#Create a table, with BGP route propagation disabled. The property is now called "Virtual network gateway route propagation," but the API still refers to the parameter as "DisableBgpRoutePropagation."
$routeTableSpokeDG = New-AzRouteTable `
  -Name 'UDR-DG' `
  -ResourceGroupName $RG1 `
  -location $Location1 `
  -DisableBgpRoutePropagation

#Create a route
Get-AzRouteTable `
  -ResourceGroupName $RG1 `
  -Name UDR-DG `
  | Add-AzRouteConfig `
  -Name "ToFirewall" `
  -AddressPrefix 0.0.0.0/0 `
  -NextHopType "VirtualAppliance" `
  -NextHopIpAddress $AzfwPrivateIP `
 | Set-AzRouteTable

#Associate the route table to the subnet

Set-AzVirtualNetworkSubnetConfig `
  -VirtualNetwork $VNetSpoke `
  -Name $SNnameSpoke `
  -AddressPrefix $SNSpokePrefix `
  -RouteTable $routeTableSpokeDG | `
Set-AzVirtualNetwork
```

## <a name="create-virtual-machines"></a>Créer des machines virtuelles

Maintenant, créez les machines virtuelles de charge de travail spoke et locale, et placez-les dans les sous-réseaux appropriés.

### <a name="create-the-workload-virtual-machine"></a>Créer la machine virtuelle de charge de travail

Créez une machine virtuelle dans le réseau virtuel spoke, exécutant IIS, sans adresse IP publique, et autorisez les tests Ping entrants.
Lorsque vous y êtes invité, saisissez le nom d’utilisateur et le mot de passe pour la machine virtuelle.

```azurepowershell
# Create an inbound network security group rule for ports 3389 and 80
$nsgRuleRDP = New-AzNetworkSecurityRuleConfig -Name Allow-RDP  -Protocol Tcp `
  -Direction Inbound -Priority 200 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix $SNSpokePrefix -DestinationPortRange 3389 -Access Allow
$nsgRuleWeb = New-AzNetworkSecurityRuleConfig -Name Allow-web  -Protocol Tcp `
  -Direction Inbound -Priority 202 -SourceAddressPrefix * -SourcePortRange * -DestinationAddressPrefix $SNSpokePrefix -DestinationPortRange 80 -Access Allow

# Create a network security group
$nsg = New-AzNetworkSecurityGroup -ResourceGroupName $RG1 -Location $Location1 -Name NSG-Spoke02 -SecurityRules $nsgRuleRDP,$nsgRuleWeb

#Create the NIC
$NIC = New-AzNetworkInterface -Name spoke-01 -ResourceGroupName $RG1 -Location $Location1 -SubnetId $VnetSpoke.Subnets[0].Id -NetworkSecurityGroupId $nsg.Id

#Define the virtual machine
$VirtualMachine = New-AzVMConfig -VMName VM-Spoke-01 -VMSize "Standard_DS2"
$VirtualMachine = Set-AzVMOperatingSystem -VM $VirtualMachine -Windows -ComputerName Spoke-01 -ProvisionVMAgent -EnableAutoUpdate
$VirtualMachine = Add-AzVMNetworkInterface -VM $VirtualMachine -Id $NIC.Id
$VirtualMachine = Set-AzVMSourceImage -VM $VirtualMachine -PublisherName 'MicrosoftWindowsServer' -Offer 'WindowsServer' -Skus '2016-Datacenter' -Version latest

#Create the virtual machine
New-AzVM -ResourceGroupName $RG1 -Location $Location1 -VM $VirtualMachine -Verbose

#Install IIS on the VM
Set-AzVMExtension `
    -ResourceGroupName $RG1 `
    -ExtensionName IIS `
    -VMName VM-Spoke-01 `
    -Publisher Microsoft.Compute `
    -ExtensionType CustomScriptExtension `
    -TypeHandlerVersion 1.4 `
    -SettingString '{"commandToExecute":"powershell Add-WindowsFeature Web-Server"}' `
    -Location $Location1
```

<!---#Create a host firewall rule to allow ping in
Set-AzVMExtension `
    -ResourceGroupName $RG1 `
    -ExtensionName IIS `
    -VMName VM-Spoke-01 `
    -Publisher Microsoft.Compute `
    -ExtensionType CustomScriptExtension `
    -TypeHandlerVersion 1.4 `
    -SettingString '{"commandToExecute":"powershell New-NetFirewallRule –DisplayName "Allow ICMPv4-In" –Protocol ICMPv4"}' `
    -Location $Location1--->

### <a name="create-the-on-premises-virtual-machine"></a>Créer la machine virtuelle locale

Il s’agit d’une machine virtuelle simple que vous utilisez pour vous connecter au moyen du Bureau à distance et de l’adresse IP publique. À partir de là, vous vous connectez au serveur local via le pare-feu. Lorsque vous y êtes invité, saisissez le nom d’utilisateur et le mot de passe pour la machine virtuelle.

```azurepowershell
New-AzVm `
    -ResourceGroupName $RG1 `
    -Name "VM-Onprem" `
    -Location $Location1 `
    -VirtualNetworkName $VNetnameOnprem `
    -SubnetName $SNNameOnprem `
    -OpenPorts 3389 `
    -Size "Standard_DS2"
```

## <a name="test-the-firewall"></a>Tester le pare-feu

Tout d’abord, obtenez puis notez l’adresse IP privée pour la machine virtuelle **VM-spoke-01**.

```azurepowershell
$NIC.IpConfigurations.privateipaddress
```

À partir du portail Azure, connectez-vous à la machine virtuelle **VM-Onprem**.
<!---2. Open a Windows PowerShell command prompt on **VM-Onprem**, and ping the private IP for **VM-spoke-01**.

   You should get a reply.--->
Ouvrez un navigateur web sur **VM-Onprem** et accédez à http://\<VM-spoke-01 private IP\>.

Vous devez voir la page Internet Information Services par défaut.

À partir de **VM-Onprem**, ouvrez une session de Bureau à distance sur **VM-spoke-01** à l’adresse IP privée.

Votre connexion doit réussir et vous devriez pouvoir vous connecter à l’aide de votre nom d’utilisateur et de votre mot de passe.

Maintenant que vous avez vérifié que les règles de pare-feu fonctionnent :

<!---- You can ping the server on the spoke VNet.--->
- Vous pouvez parcourir le serveur web sur le réseau virtuel spoke.
- Vous pouvez vous connecter au serveur sur le réseau virtuel spoke à l’aide de RDP.

Modifiez ensuite l’action de collecte des règles du réseau de pare-feu en **Refuser** pour vérifier que les règles de pare-feu fonctionnent comme prévu. Exécutez le script suivant pour modifier l’action de collection des règles en **Refuser**.

```azurepowershell
$rcNet = $azfw.GetNetworkRuleCollectionByName("RCNet01")
$rcNet.action.type = "Deny"

Set-AzFirewall -AzureFirewall $azfw
```

Maintenant, réexécutez les tests. Cette fois, ils doivent échouer. Fermez les bureaux à distance existants avant de tester les règles modifiées.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Vous pouvez garder vos ressources de pare-feu pour le prochain didacticiel, ou, si vous n’en avez plus besoin, vous pouvez supprimer le groupe de ressources **FW-Hybrid-Test** pour supprimer toutes les ressources associées au pare-feu.

## <a name="next-steps"></a>Étapes suivantes

Ensuite, vous pouvez surveiller les journaux d’activité de Pare-feu Azure.

[Tutoriel : Superviser les journaux d’activité de Pare-feu Azure](./firewall-diagnostics.md)