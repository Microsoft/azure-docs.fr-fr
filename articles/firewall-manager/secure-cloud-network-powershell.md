---
title: 'Tutoriel : Sécuriser votre hub virtuel avec Azure PowerShell'
description: Cet article explique comment créer un WAN virtuel Azure dans une région où le Pare-feu Azure est activé dans le hub.
services: firewall-manager
author: jomore
ms.topic: tutorial
ms.service: firewall-manager
ms.date: 10/22/2020
ms.author: victorh
ms.openlocfilehash: e2bcdf947db60fb50b44abba9d6a224768840b28
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94656137"
---
# <a name="tutorial-secure-your-virtual-hub-using-azure-powershell"></a>Tutoriel : Sécuriser votre hub virtuel avec Azure PowerShell

Dans ce tutoriel, vous allez créer une instance de WAN virtuel avec un hub virtuel dans une région, et vous allez déployer un pare-feu Azure dans le hub virtuel pour sécuriser la connectivité. Dans cet exemple, vous allez illustrer la connectivité sécurisée entre des réseaux virtuels. Le trafic entre les réseaux virtuels et les branches de site à site, de point à site ou ExpressRoute est également pris en charge par le Hub virtuel sécurisé.

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> * Déployer le WAN virtuel.
> * Déployer le Pare-feu Azure et configurer le routage personnalisé.
> * Tester la connectivité

## <a name="prerequisites"></a>Prérequis

- Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

- PowerShell 7

   Pour ce tutoriel, vous devez exécuter Azure PowerShell localement sur PowerShell 7. Pour installer PowerShell 7, consultez [Migration de Windows PowerShell 5.1 vers PowerShell 7](/powershell/scripting/install/migrating-from-windows-powershell-51-to-powershell-7?view=powershell-7).
- Az.Network version 3.2.0

    Si vous avez Az.Network version 3.4.0 ou ultérieure, vous devez passer à une version antérieure pour utiliser certaines des commandes de ce tutoriel. Vous pouvez vérifier la version de votre module Az.Network à l’aide de la commande `Get-InstalledModule -Name Az.Network`. Pour désinstaller le module Az.Network, exécutez `Uninstall-Module -name az.network`. Pour installer le module Az.Network 3.2.0, exécutez `Install-Module az.network -RequiredVersion 3.2.0 -force`.

## <a name="sign-in-to-azure"></a>Connexion à Azure

```azurepowershell
Connect-AzAccount
Select-AzSubscription -Subscription "<sub name>"
```

## <a name="initial-virtual-wan-deployment"></a>Déploiement du WAN virtuel initial

Dans un premier temps, vous devez définir certaines variables et créer le groupe de ressources, l’instance du WAN virtuel et le hub virtuel :

```azurepowershell
# Variable definition
$RG = "vwan-rg"
$Location = "westeurope"
$VwanName = "vwan"
$HubName =  "hub1"
# Create Resource Group, Virtual WAN and Virtual Hub
New-AzResourceGroup -Name $RG -Location $Location
$Vwan = New-AzVirtualWan -Name $VwanName -ResourceGroupName $RG -Location $Location -AllowVnetToVnetTraffic -AllowBranchToBranchTraffic -VirtualWANType "Standard"
$Hub = New-AzVirtualHub -Name $HubName -ResourceGroupName $RG -VirtualWan $Vwan -Location $Location -AddressPrefix "192.168.1.0/24" -Sku "Standard"
```

Créez deux réseaux virtuels et connectez-les au hub en tant que spokes :

```azurepowershell
# Create Virtual Network
$Spoke1 = New-AzVirtualNetwork -Name "spoke1" -ResourceGroupName $RG -Location $Location -AddressPrefix "10.1.1.0/24"
$Spoke2 = New-AzVirtualNetwork -Name "spoke2" -ResourceGroupName $RG -Location $Location -AddressPrefix "10.1.2.0/24"
# Connect Virtual Network to Virtual WAN
$Spoke1Connection = New-AzVirtualHubVnetConnection -ResourceGroupName $RG -ParentResourceName  $HubName -Name "spoke1" -RemoteVirtualNetwork $Spoke1
$Spoke2Connection = New-AzVirtualHubVnetConnection -ResourceGroupName $RG -ParentResourceName  $HubName -Name "spoke2" -RemoteVirtualNetwork $Spoke2
```

À ce stade, vous disposez d’un WAN virtuel entièrement fonctionnel fournissant une connectivité Any-To-Any. Pour améliorer la sécurité, vous devez déployer un pare-feu Azure sur chaque hub virtuel. Des stratégies de pare-feu peuvent être utilisées pour gérer efficacement l’instance de Pare-feu Azure du WAN virtuel. Par conséquent, une stratégie de pare-feu est également créée dans cet exemple :

```azurepowershell
# New Firewall Policy
$FWPolicy = New-AzFirewallPolicy -Name "VwanFwPolicy" -ResourceGroupName $RG -Location $Location
# New Firewall Public IP
$AzFWPIPs = New-AzFirewallHubPublicIpAddress -Count 1
$AzFWHubIPs = New-AzFirewallHubIpAddress -PublicIP $AzFWPIPs
# New Firewall
$AzFW = New-AzFirewall -Name "azfw1" -ResourceGroupName $RG -Location $Location `
            -VirtualHubId $Hub.Id -FirewallPolicyId $FWPolicy.Id `
            -Sku AZFW_Hub -HubIPAddress $AzFWHubIPs
```

L’activation de la journalisation du pare-feu Azure vers Azure Monitor est facultative, mais dans cet exemple vous utilisez les journaux de pare-feu pour prouver que le trafic traverse le pare-feu :

```azurepowershell
# Optionally, enable looging of Azure Firewall to Azure Monitor
$LogWSName = "vwan-" + (Get-Random -Maximum 99999) + "-" + $RG
$LogWS = New-AzOperationalInsightsWorkspace -Location $Location -Name $LogWSName -Sku Standard -ResourceGroupName $RG
Set-AzDiagnosticSetting -ResourceId $AzFW.Id -Enabled $True -Category AzureFirewallApplicationRule, AzureFirewallNetworkRule -WorkspaceId $LogWS.ResourceId
```

## <a name="deploy-azure-firewall-and-configure-custom-routing"></a>Déployer le Pare-feu Azure et configurer le routage personnalisé.

Vous disposez maintenant d’un pare-feu Azure dans le hub, mais vous devez encore modifier le routage de sorte que le WAN virtuel envoie le trafic en provenance des réseaux virtuels et des branches à travers le pare-feu. Cette opération s’effectue en deux étapes :

1. Configurez toutes les connexions de réseau virtuel (et les connexions de branche, le cas échéant) pour une propagation vers la table de routage `None`. L’effet de cette configuration est que les autres réseaux virtuels et branches n’apprendront pas leurs préfixes, et n’auront donc pas de routage pour les atteindre.
1. Vous pouvez maintenant insérer des routes statiques dans la table de routage `Default` (où tous les réseaux virtuels et branches sont associés par défaut), afin que tout le trafic soit envoyé au pare-feu Azure.

> [!NOTE]
> Il s’agit de la configuration déployée lors de la sécurisation de la connectivité à partir du portail Azure avec Azure Firewall Manager.

Commencez par la première étape afin de configurer vos connexions de réseau virtuel pour qu’elles se propagent à la table de routage `None` :

```azurepowershell
# Configure Virtual Network connections in hub to propagate to None
$VnetRoutingConfig = $Spoke1Connection.RoutingConfiguration    # We take $Spoke1Connection as baseline for the future vnet config, all vnets will have an identical config
$NoneRT = Get-AzVhubRouteTable -ResourceGroupName $RG -HubName $HubName -Name "noneRouteTable"
$NewPropRT = @{}
$NewPropRT.Add('Id', $NoneRT.id)
$PropRTList = @()
$PropRTList += $NewPropRT
$VnetRoutingConfig.PropagatedRouteTables.Ids = $PropRTList
$VnetRoutingConfig.PropagatedRouteTables.Labels = @()
$Spoke1Connection = Update-AzVirtualHubVnetConnection -ResourceGroupName $RG -ParentResourceName  $HubName -Name "spoke1" -RoutingConfiguration $VnetRoutingConfig
$Spoke2Connection = Update-AzVirtualHubVnetConnection -ResourceGroupName $RG -ParentResourceName  $HubName -Name "spoke2" -RoutingConfiguration $VnetRoutingConfig
```

Vous pouvez maintenant passer à la deuxième étape pour ajouter les routes statiques à la table de routage `Default`. Dans cet exemple, vous appliquez la configuration par défaut qui serait générée par Azure Firewall Manager lors de la sécurisation de la connectivité dans un WAN virtuel, mais vous pouvez modifier la liste des préfixes dans la route statique en fonction de vos besoins spécifiques :

```azurepowershell
# Create static routes in default Route table
$AzFWId = $(Get-AzVirtualHub -ResourceGroupName $RG -name  $HubName).AzureFirewall.Id
$AzFWRoute = New-AzVHubRoute -Name "private-traffic" -Destination @("0.0.0.0/0", "10.0.0.0/8", "172.16.0.0/12", "192.168.0.0/16") -DestinationType "CIDR" -NextHop $AzFWId -NextHopType "ResourceId"
$DefaultRT = Update-AzVHubRouteTable -Name "defaultRouteTable" -ResourceGroupName $RG -VirtualHubName  $HubName -Route @($AzFWRoute)
```

## <a name="test-connectivity"></a>Tester la connectivité

Vous disposez maintenant d’un hub sécurisé entièrement opérationnel. Pour tester la connectivité, vous avez besoin d’une machine virtuelle dans chaque réseau virtuel spoke connecté au hub :

```azurepowershell
# Create VMs in spokes for testing
$VMLocalAdminUser = "lab-user"
$VMLocalAdminSecurePassword = ConvertTo-SecureString -AsPlainText -Force
$VMCredential = New-Object System.Management.Automation.PSCredential ($VMLocalAdminUser, $VMLocalAdminSecurePassword);
$VMSize = "Standard_B2ms"
# Spoke1
$Spoke1 = Get-AzVirtualNetwork -ResourceGroupName $RG -Name "spoke1"
Add-AzVirtualNetworkSubnetConfig -Name "vm" -VirtualNetwork $Spoke1 -AddressPrefix "10.1.1.0/26"
$Spoke1 | Set-AzVirtualNetwork
$VM1 = New-AzVM -Name "spoke1-vm" -ResourceGroupName $RG -Location $Location `
            -Image "UbuntuLTS" -credential $VMCredential `
            -VirtualNetworkName "spoke1" -SubnetName "vm" -PublicIpAddressName "spoke1-pip"
$NIC1 = Get-AzNetworkInterface -ResourceId $($VM1.NetworkProfile.NetworkInterfaces[0].Id)
$Spoke1VMPrivateIP = $NIC1.IpConfigurations[0].PrivateIpAddress
$Spoke1VMPIP = $(Get-AzPublicIpAddress -ResourceGroupName $RG -Name "spoke1-pip")
# Spoke2
$Spoke2 = Get-AzVirtualNetwork -ResourceGroupName $RG -Name "spoke2"
Add-AzVirtualNetworkSubnetConfig -Name "vm" -VirtualNetwork $Spoke2 -AddressPrefix "10.1.2.0/26"
$Spoke2 | Set-AzVirtualNetwork
$VM2 = New-AzVM -Name "spoke2-vm" -ResourceGroupName $RG -Location $Location `
            -Image "UbuntuLTS" -credential $VMCredential `
            -VirtualNetworkName "spoke2" -SubnetName "vm" -PublicIpAddressName "spoke2-pip"
$NIC2 = Get-AzNetworkInterface -ResourceId $($VM2.NetworkProfile.NetworkInterfaces[0].Id)
$Spoke2VMPrivateIP = $NIC2.IpConfigurations[0].PrivateIpAddress
$Spoke2VMPIP = $(Get-AzPublicIpAddress -ResourceGroupName $RG -Name "spoke2-pip")
```

La configuration par défaut dans la stratégie de pare-feu consiste à tout supprimer. Vous devez donc configurer certaines règles. Commencez par des règles DNAT, afin que les machines virtuelles de test soient accessibles par le biais de l’adresse IP publique du pare-feu :

```azurepowershell
# Adding DNAT rules for virtual machines in the spokes
$AzFWPublicAddress = $AzFW.HubIPAddresses.PublicIPs.Addresses[0].Address
$NATRuleSpoke1 = New-AzFirewallPolicyNatRule -Name "Spoke1SSH" -Protocol "TCP" `
        -SourceAddress "*" -DestinationAddress $AzFWPublicAddress -DestinationPort 10001 `
        -TranslatedAddress $Spoke1VMPrivateIP -TranslatedPort 22
$NATRuleSpoke2 = New-AzFirewallPolicyNatRule -Name "Spoke2SSH" -Protocol "TCP" `
        -SourceAddress "*" -DestinationAddress $AzFWPublicAddress -DestinationPort 10002 `
        -TranslatedAddress $Spoke2VMPrivateIP -TranslatedPort 22
$NATCollection = New-AzFirewallPolicyNatRuleCollection -Name "SSH" -Priority 100 `
        -Rule @($NATRuleSpoke1, $NATRuleSpoke2) -ActionType "Dnat"
$NATGroup = New-AzFirewallPolicyRuleCollectionGroup -Name "NAT" -Priority 100 -RuleCollection $NATCollection -FirewallPolicyObject $FWPolicy
```

Vous pouvez maintenant configurer des exemples de règles. Définissez une règle de réseau qui autorise le trafic SSH, ainsi qu’une règle d’application qui autorise l’accès Internet au nom de domaine complet `ifconfig.co`. Cette URL retourne l’adresse IP source qu’elle voit dans la requête HTTP :

```azurepowershell
# Add Network Rule
$SSHRule = New-AzFirewallPolicyNetworkRule -Name PermitSSH -Protocol TCP `
        -SourceAddress "10.0.0.0/8" -DestinationAddress "10.0.0.0/8" -DestinationPort 22
$NetCollection = New-AzFirewallPolicyFilterRuleCollection -Name "Management" -Priority 100 -ActionType Allow -Rule $SSHRule
$NetGroup = New-AzFirewallPolicyRuleCollectionGroup -Name "Management" -Priority 200 -RuleCollection $NetCollection -FirewallPolicyObject $FWPolicy
# Add Application Rul
$ifconfigRule = New-AzFirewallPolicyApplicationRule -Name PermitIfconfig -SourceAddress "10.0.0.0/8" -TargetFqdn "ifconfig.co" -Protocol "http:80","https:443"
$AppCollection = New-AzFirewallPolicyFilterRuleCollection -Name "TargetURLs" -Priority 300 -ActionType Allow -Rule $ifconfigRule
$NetGroup = New-AzFirewallPolicyRuleCollectionGroup -Name "TargetURLs" -Priority 300 -RuleCollection $AppCollection -FirewallPolicyObject $FWPolicy
```

Avant d’envoyer réellement le trafic, vous pouvez inspecter les routes effectives des machines virtuelles. Elles doivent contenir les préfixes appris du WAN virtuel (`0.0.0.0/0` plus RFC1918), mais pas le préfixe de l’autre spoke :

```azurepowershell
# Check effective routes in the VM NIC in spoke 1
# Note that 10.1.2.0/24 (the prefix for spoke2) should not appear
Get-AzEffectiveRouteTable -ResourceGroupName $RG -NetworkInterfaceName $NIC1.Name | ft
# Check effective routes in the VM NIC in spoke 2
# Note that 10.1.1.0/24 (the prefix for spoke1) should not appear
Get-AzEffectiveRouteTable -ResourceGroupName $RG -NetworkInterfaceName $NIC2.Name | ft
```

Maintenant, générez du trafic d’une machine virtuelle à l’autre et vérifiez qu’il est supprimé dans le Pare-feu Azure. Dans les commandes SSH suivantes, vous devez accepter les empreintes digitales des machines virtuelles et fournir le mot de passe que vous avez défini lors de la création des machines virtuelles. Dans cet exemple, vous allez envoyer cinq paquets de demande d’écho ICMP à partir de la machine virtuelle dans spoke1 vers spoke2, ainsi qu’une tentative de connexion TCP sur le port 22 à l’aide de l’utilitaire Linux `nc` (avec les indicateurs `-vz`, il envoie simplement une demande de connexion et affiche le résultat). Le test ping doit échouer et la tentative de connexion TCP sur le port 22 réussir, puisqu’elle est autorisée par la règle réseau que vous avez configurée :

```azurepowershell
# Connect to one VM and ping the other. It shouldnt work, because the firewall should drop the traffic, since no rule for ICMP is configured
ssh $AzFWPublicAddress -p 10001 -l $VMLocalAdminUser "ping $Spoke2VMPrivateIP -c 5"
# Connect to one VM and send a TCP request on port 22 to the other. It should work, because the firewall is configured to allow SSH traffic (port 22)
ssh $AzFWPublicAddress -p 10001 -l $VMLocalAdminUser "nc -vz $Spoke2VMPrivateIP 22"
```

Vous pouvez également vérifier le trafic Internet. Les requêtes HTTP effectuées par le biais de l’utilitaire `curl` au nom de domaine complet que vous avez autorisé dans la stratégie de pare-feu (`ifconfig.co`) doivent fonctionner, mais les requêtes HTTP adressées à toute autre destination doivent échouer (dans cet exemple, vous effectuez un test avec `bing.com`) :

```azurepowershell
# This HTTP request should succeed, since it is allowed in an app rule in the AzFW, and return the public IP of the FW
ssh $AzFWPublicAddress -p 10001 -l $VMLocalAdminUser "curl -s4 ifconfig.co"
# This HTTP request should fail, since the FQDN bing.com is not in any app rule in the firewall policy
ssh $AzFWPublicAddress -p 10001 -l $VMLocalAdminUser "curl -s4 bing.com"
```

Le moyen le plus simple de vérifier que les paquets sont supprimés par le pare-feu consiste à vérifier les journaux. Étant donné que vous avez configuré le Pare-feu Azure pour envoyer des journaux à Azure Monitor, vous pouvez utiliser le langage de requête Kusto pour récupérer les journaux appropriés à partir d’Azure Monitor :

> [!NOTE]
> L’envoi des journaux à Azure Monitor peut prendre environ une minute.

```azurepowershell
# Getting Azure Firewall network rule Logs
$LogWS = Get-AzOperationalInsightsWorkspace -ResourceGroupName $RG
$LogQuery = 'AzureDiagnostics 
| where Category == "AzureFirewallNetworkRule" 
| where TimeGenerated >= ago(5m) 
| parse msg_s with Protocol " request from " SourceIP ":" SourcePortInt:int " to " TargetIP ":" TargetPortInt:int * 
| parse msg_s with * ". Action: " Action1a 
| parse msg_s with * " was " Action1b " to " NatDestination 
| parse msg_s with Protocol2 " request from " SourceIP2 " to " TargetIP2 ". Action: " Action2 
| extend SourcePort = tostring(SourcePortInt),TargetPort = tostring(TargetPortInt) 
| extend Action = case(Action1a == "", case(Action1b == "",Action2,Action1b), Action1a),Protocol = case(Protocol == "", Protocol2, Protocol),SourceIP = case(SourceIP == "", SourceIP2, SourceIP),TargetIP = case(TargetIP == "", TargetIP2, TargetIP),SourcePort = case(SourcePort == "", "N/A", SourcePort),TargetPort = case(TargetPort == "", "N/A", TargetPort),NatDestination = case(NatDestination == "", "N/A", NatDestination) 
| project TimeGenerated, Protocol, SourceIP,SourcePort,TargetIP,TargetPort,Action, NatDestination, Resource 
| take 25 '
$(Invoke-AzOperationalInsightsQuery -Workspace $LogWS -Query $LogQuery).Results | ft
```

Dans la commande précédente, vous devriez voir différentes entrées :

* Traduction DNAT appliquée à votre connexion SSH.
* Suppression des paquets ICMP entre les machines virtuelles dans les spokes (10.1.1.4 et 10.1.2.4).
* Connexions SSH autorisées entre les machines virtuelles dans les spokes.

Voici un exemple de sortie produit par la commande ci-dessus :

```
TimeGenerated            Protocol    SourceIP       SourcePort TargetIP      TargetPort Action  NatDestination Resource
-------------            --------    --------       ---------- --------      ---------- ------  -------------- --------
2020-10-04T20:53:02.41Z  TCP         109.125.122.99 62281      51.105.224.44 10001      DNAT'ed 10.1.1.4:22    AZFW1
2020-10-04T20:53:07.045Z TCP         10.1.1.4       35932      10.1.2.4      22         Allow   N/A            AZFW1
2020-10-04T20:53:50.119Z TCP         109.125.122.99 62293      51.105.224.44 10001      DNAT'ed 10.1.2.4:22    AZFW1
2020-10-04T20:52:47.475Z TCP         109.125.122.99 62273      51.105.224.44 10001      DNAT'ed 10.1.2.4:22    AZFW1
2020-10-04T20:51:04.682Z TCP         109.125.122.99 62200      51.105.224.44 10001      DNAT'ed 10.1.2.4:22    AZFW1
2020-10-04T20:51:17.031Z ICMP Type=8 10.1.1.4       N/A        10.1.2.4      N/A        Deny    N/A            AZFW1
2020-10-04T20:51:18.049Z ICMP Type=8 10.1.1.4       N/A        10.1.2.4      N/A        Deny    N/A            AZFW1
2020-10-04T20:51:19.075Z ICMP Type=8 10.1.1.4       N/A        10.1.2.4      N/A        Deny    N/A            AZFW1
2020-10-04T20:51:20.097Z ICMP Type=8 10.1.1.4       N/A        10.1.2.4      N/A        Deny    N/A            AZFW1
2020-10-04T20:51:21.121Z ICMP Type=8 10.1.1.4       N/A        10.1.2.4      N/A        Deny    N/A            AZFW1
2020-10-04T20:52:52.356Z TCP         10.1.1.4       53748      10.1.2.4      22         Allow   N/A            AZFW1
```

Si vous souhaitez afficher les journaux des règles d’application (décrivant les connexions HTTP autorisées et refusées) ou modifier la façon dont les journaux sont affichés, vous pouvez essayer d’autres requêtes KQL. Vous trouverez des exemples dans [Journaux Azure Monitor pour Pare-feu Azure](../firewall/log-analytics-samples.md).


## <a name="clean-up-resources"></a>Nettoyer les ressources

Pour supprimer l’environnement de test, vous pouvez supprimer le groupe de ressources avec tous les objets qu’il contient :

```azurepowershell
# Delete resource group and all contained resources
Remove-AzResourceGroup -Name $RG
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
> [En savoir plus sur les partenaires de sécurité de confiance](trusted-security-partners.md)