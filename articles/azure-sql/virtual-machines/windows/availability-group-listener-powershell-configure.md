---
title: Configurer des écouteurs de groupe de disponibilité et l’équilibreur de charge (PowerShell)
description: Configurez les écouteurs de groupe de disponibilité sur le modèle Azure Resource Manager, à l’aide d’un équilibreur de charge interne avec une ou plusieurs adresses IP.
services: virtual-machines
documentationcenter: na
author: MashaMSFT
editor: monicar
ms.assetid: 14b39cde-311c-4ddf-98f3-8694e01a7d3b
ms.service: virtual-machines-sql
ms.subservice: hadr
ms.topic: how-to
ms.tgt_pltfrm: vm-windows-sql-server
ms.workload: iaas-sql-server
ms.date: 02/06/2019
ms.author: mathoma
ms.custom: seo-lt-2019
ms.openlocfilehash: 9337d1c2767923e6dc7c6b267e0c180b460a116e
ms.sourcegitcommit: dfc4e6b57b2cb87dbcce5562945678e76d3ac7b6
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/12/2020
ms.locfileid: "97359419"
---
# <a name="configure-one-or-more-always-on-availability-group-listeners---resource-manager"></a>Configurer un ou plusieurs écouteurs de groupe de disponibilité AlwaysOn - Resource Manager

[!INCLUDE[appliesto-sqlvm](../../includes/appliesto-sqlvm.md)]

Ce document vous montre comment utiliser PowerShell pour effectuer l’une des tâches suivantes :
- Créer un équilibreur de charge
- Ajouter des adresses IP à un équilibreur de charge existant pour les groupes de disponibilité SQL Server

Un écouteur de groupe de disponibilité est un nom de réseau virtuel auquel les clients se connectent pour accéder aux bases de données. Sur les machines virtuelles Azure, un équilibreur de charge comporte l’adresse IP de l’écouteur. L’équilibreur de charge achemine le trafic vers l’instance de SQL Server qui est à l’écoute sur le port de la sonde. Dans la plupart des cas, un groupe de disponibilité utilise un équilibreur de charge interne. Un équilibreur de charge interne Azure peut héberger une ou plusieurs adresses IP. Chaque adresse IP utilise un port de sonde spécifique. 

L’affectation de plusieurs adresses IP à un équilibreur de charge interne est une nouveauté d’Azure qui n’est disponible que dans le modèle Resource Manager. Pour effectuer cette tâche, vous devez disposer d’un groupe de disponibilité SQL Server déployé sur des machines virtuelles Azure dans le modèle Resource Manager. Les deux machines virtuelles SQL Server doivent appartenir au même groupe à haute disponibilité. Vous pouvez utiliser le [modèle Microsoft](./availability-group-quickstart-template-configure.md) pour créer automatiquement le groupe de disponibilité dans Azure Resource Manager. Ce modèle crée automatiquement le groupe de disponibilité, y compris l’équilibreur de charge interne. Si vous préférez, vous pouvez [configurer manuellement un groupe de disponibilité AlwaysOn](availability-group-manually-configure-tutorial.md).

Pour effectuer les étapes décrites dans cet article, vous devez avoir déjà configuré vos groupes de disponibilité.  

Rubriques connexes :

* [Configuration de groupes de disponibilité AlwaysOn dans Azure VM (GUI)](availability-group-manually-configure-tutorial.md)   
* [Configurer une connexion de réseau virtuel à réseau virtuel à l’aide d’Azure Resource Manager et de PowerShell](../../../vpn-gateway/vpn-gateway-vnet-vnet-rm-ps.md)

[!INCLUDE [updated-for-az.md](../../../../includes/updated-for-az.md)]

[!INCLUDE [Start your PowerShell session](../../../../includes/sql-vm-powershell.md)]

## <a name="verify-powershell-version"></a>Vérifier la version de PowerShell

Les exemples présentés dans cet article sont testés à l'aide du module Azure PowerShell version 5.4.1.

Vérifiez que votre module PowerShell correspond bien à la version 5.4.1 ou ultérieure.

Consultez [Installer le module Azure PowerShell](/powershell/azure/install-az-ps).

## <a name="configure-the-windows-firewall"></a>Configurer le pare-feu Windows

Configurez le Pare-feu Windows pour autoriser l’accès à SQL Server. Les règles de pare-feu autorisent les connexions TCP aux ports utilisés par l’instance de SQL Server et à la sonde de l’écouteur. Pour obtenir des instructions détaillées, consultez [Configurer un pare-feu Windows pour accéder au moteur de base de données](/sql/database-engine/configure-windows/configure-a-windows-firewall-for-database-engine-access#Anchor_1). Créez une règle entrante pour le port SQL Server et pour le port de la sonde.

Si vous restreignez l’accès avec un groupe de sécurité réseau Azure, vérifiez que les règles d’autorisation comprennent l’adresse IP de la machine virtuelle du serveur backend SQL, les adresses IP flottantes de l’équilibreur de charge pour l’écouteur de groupe de disponibilité, et l’adresse IP du cluster principal, le cas échéant.

## <a name="determine-the-load-balancer-sku-required"></a>Déterminer la référence SKU requise pour l'équilibreur de charge

L’[équilibreur de charge Azure](../../../load-balancer/load-balancer-overview.md) se décline en deux références SKU : De base et Standard. L'équilibreur de charge standard est recommandé. Si les machines virtuelles se trouvent dans un groupe à haute disponibilité, l'équilibreur de charge de base est autorisé. Si les machines virtuelles se trouvent dans une zone de disponibilité, un équilibreur de charge standard est requis. L'équilibreur de charge standard exige que toutes les machines virtuelles utilisent des adresses IP standard.

Le [modèle Microsoft](./availability-group-quickstart-template-configure.md) actuel de groupe de disponibilité utilise un équilibreur de charge de base avec des adresses IP de base.

   > [!NOTE]
   > Si vous utilisez un équilibreur de charge standard et Stockage Azure pour le témoin cloud, vous devez configurer un [point de terminaison de service](../../../storage/common/storage-network-security.md?toc=%2fazure%2fvirtual-network%2ftoc.json#grant-access-from-a-virtual-network). 
   > 

Les exemples présentés dans cet article spécifient un équilibreur de charge standard. Dans les exemples, le script inclut `-sku Standard`.

```powershell
$ILB= New-AzLoadBalancer -Location $Location -Name $ILBName -ResourceGroupName $ResourceGroupName -FrontendIpConfiguration $FEConfig -BackendAddressPool $BEConfig -LoadBalancingRule $ILBRule -Probe $SQLHealthProbe -sku Standard
```

Pour créer un équilibreur de charge de base, supprimez `-sku Standard` de la ligne qui crée l'équilibreur de charge. Par exemple :

```powershell
$ILB= New-AzLoadBalancer -Location $Location -Name $ILBName -ResourceGroupName $ResourceGroupName -FrontendIpConfiguration $FEConfig -BackendAddressPool $BEConfig -LoadBalancingRule $ILBRule -Probe $SQLHealthProbe
```

## <a name="example-script-create-an-internal-load-balancer-with-powershell"></a>Exemple de script : Créer un équilibreur de charge interne avec PowerShell

> [!NOTE]
> Si vous avez créé un groupe de disponibilité à l’aide du [modèle Microsoft](./availability-group-quickstart-template-configure.md), l’équilibreur de charge interne a déjà été créé.

Le script PowerShell suivant crée un équilibreur de charge interne, configure les règles d’équilibrage de charge et affecte une adresse IP pour l’équilibrage de charge. Pour exécuter le script, ouvrez Windows PowerShell ISE, puis collez le script dans le volet Script. Utilisez `Connect-AzAccount` pour vous connecter à PowerShell. Si vous possédez plusieurs abonnements Azure, utilisez `Select-AzSubscription` pour définir l’abonnement. 

```powershell
# Connect-AzAccount
# Select-AzSubscription -SubscriptionId <xxxxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx>

$ResourceGroupName = "<Resource Group Name>" # Resource group name
$VNetName = "<Virtual Network Name>"         # Virtual network name
$SubnetName = "<Subnet Name>"                # Subnet name
$ILBName = "<Load Balancer Name>"            # ILB name
$Location = "<Azure Region>"                 # Azure location
$VMNames = "<VM1>","<VM2>"                   # Virtual machine names

$ILBIP = "<n.n.n.n>"                         # IP address
[int]$ListenerPort = "<nnnn>"                # AG listener port
[int]$ProbePort = "<nnnn>"                   # Probe port

$LBProbeName ="ILBPROBE_$ListenerPort"       # The Load balancer Probe Object Name              
$LBConfigRuleName = "ILBCR_$ListenerPort"    # The Load Balancer Rule Object Name

$FrontEndConfigurationName = "FE_SQLAGILB_1" # Object name for the front-end configuration 
$BackEndConfigurationName ="BE_SQLAGILB_1"   # Object name for the back-end configuration

$VNet = Get-AzVirtualNetwork -Name $VNetName -ResourceGroupName $ResourceGroupName 

$Subnet = Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $VNet -Name $SubnetName 

$FEConfig = New-AzLoadBalancerFrontendIpConfig -Name $FrontEndConfigurationName -PrivateIpAddress $ILBIP -SubnetId $Subnet.id

$BEConfig = New-AzLoadBalancerBackendAddressPoolConfig -Name $BackEndConfigurationName 

$SQLHealthProbe = New-AzLoadBalancerProbeConfig -Name $LBProbeName -Protocol tcp -Port $ProbePort -IntervalInSeconds 15 -ProbeCount 2

$ILBRule = New-AzLoadBalancerRuleConfig -Name $LBConfigRuleName -FrontendIpConfiguration $FEConfig -BackendAddressPool $BEConfig -Probe $SQLHealthProbe -Protocol tcp -FrontendPort $ListenerPort -BackendPort $ListenerPort -LoadDistribution Default -EnableFloatingIP 

$ILB= New-AzLoadBalancer -Location $Location -Name $ILBName -ResourceGroupName $ResourceGroupName -FrontendIpConfiguration $FEConfig -BackendAddressPool $BEConfig -LoadBalancingRule $ILBRule -Probe $SQLHealthProbe 

$bepool = Get-AzLoadBalancerBackendAddressPoolConfig -Name $BackEndConfigurationName -LoadBalancer $ILB 

foreach($VMName in $VMNames)
    {
        $VM = Get-AzVM -ResourceGroupName $ResourceGroupName -Name $VMName 
        $NICName = ($vm.NetworkProfile.NetworkInterfaces.Id.split('/') | select -last 1)
        $NIC = Get-AzNetworkInterface -name $NICName -ResourceGroupName $ResourceGroupName
        $NIC.IpConfigurations[0].LoadBalancerBackendAddressPools = $BEPool
        Set-AzNetworkInterface -NetworkInterface $NIC
        start-AzVM -ResourceGroupName $ResourceGroupName -Name $VM.Name 
    }
```

## <a name="example-script-add-an-ip-address-to-an-existing-load-balancer-with-powershell"></a><a name="Add-IP"></a> Exemple de script : Ajouter une adresse IP à un équilibreur de charge existant à l’aide de PowerShell

Pour utiliser plusieurs groupes de disponibilité, ajoutez une adresse IP supplémentaire à l’équilibreur de charge. Chaque adresse IP nécessite une règle d’équilibrage de charge, un port de sonde et un port frontal propres.

Le port frontal est le port que les applications utilisent pour se connecter à l’instance SQL Server. Les adresses IP de groupes de disponibilité différents peuvent utiliser le même port frontal.

> [!NOTE]
> Pour les groupes de disponibilité SQL Server, chaque adresse IP nécessite un port de sonde spécifique. Par exemple, si une adresse IP sur un équilibreur de charge utilise le port de sonde 59999, aucune autre adresse IP de cet équilibreur de charge ne peut utiliser le port 59999 de la sonde.

* Pour plus d’informations sur les limites de l’équilibreur de charge, consultez **Adresse IP frontale privée par équilibreur de charge** sous [Limites de mise en réseau – Azure Resource Manager](../../../azure-resource-manager/management/azure-subscription-service-limits.md#azure-resource-manager-virtual-networking-limits).
* Pour plus d’informations sur les limites de groupe de disponibilité, consultez [Restrictions (groupes de disponibilité)](/sql/database-engine/availability-groups/windows/prereqs-restrictions-recommendations-always-on-availability#RestrictionsAG).

Le script suivant ajoute une nouvelle adresse IP à un équilibreur de charge existant. L’équilibreur de charge interne utilise le port d’écoute pour le port frontal de l’équilibrage de charge. Ce port peut être le port sur lequel SQL Server écoute le trafic. Pour les instances par défaut de SQL Server, il s’agit du port 1433. La règle d’équilibrage de charge d’un groupe de disponibilité nécessite une adresse IP flottante (retour direct du serveur) de sorte que le port de serveur principal est le même que le port frontal. Mettez à jour les variables de votre environnement. 

```powershell
# Connect-AzAccount
# Select-AzSubscription -SubscriptionId <xxxxxxxxxxx-xxxx-xxxx-xxxx-xxxxxxxxxxxx>

$ResourceGroupName = "<ResourceGroup>"          # Resource group name
$VNetName = "<VirtualNetwork>"                  # Virtual network name
$SubnetName = "<Subnet>"                        # Subnet name
$ILBName = "<ILBName>"                          # ILB name                      

$ILBIP = "<n.n.n.n>"                            # IP address
[int]$ListenerPort = "<nnnn>"                   # AG listener port
[int]$ProbePort = "<nnnnn>"                     # Probe port 

$ILB = Get-AzLoadBalancer -Name $ILBName -ResourceGroupName $ResourceGroupName 

$count = $ILB.FrontendIpConfigurations.Count+1
$FrontEndConfigurationName ="FE_SQLAGILB_$count"  

$LBProbeName = "ILBPROBE_$count"
$LBConfigrulename = "ILBCR_$count"

$VNet = Get-AzVirtualNetwork -Name $VNetName -ResourceGroupName $ResourceGroupName 
$Subnet = Get-AzVirtualNetworkSubnetConfig -VirtualNetwork $VNet -Name $SubnetName

$ILB | Add-AzLoadBalancerFrontendIpConfig -Name $FrontEndConfigurationName -PrivateIpAddress $ILBIP -SubnetId $Subnet.Id 

$ILB | Add-AzLoadBalancerProbeConfig -Name $LBProbeName  -Protocol Tcp -Port $Probeport -ProbeCount 2 -IntervalInSeconds 15  | Set-AzLoadBalancer 

$ILB = Get-AzLoadBalancer -Name $ILBname -ResourceGroupName $ResourceGroupName

$FEConfig = get-AzLoadBalancerFrontendIpConfig -Name $FrontEndConfigurationName -LoadBalancer $ILB

$SQLHealthProbe  = Get-AzLoadBalancerProbeConfig -Name $LBProbeName -LoadBalancer $ILB

$BEConfig = Get-AzLoadBalancerBackendAddressPoolConfig -Name $ILB.BackendAddressPools[0].Name -LoadBalancer $ILB 

$ILB | Add-AzLoadBalancerRuleConfig -Name $LBConfigRuleName -FrontendIpConfiguration $FEConfig  -BackendAddressPool $BEConfig -Probe $SQLHealthProbe -Protocol tcp -FrontendPort  $ListenerPort -BackendPort $ListenerPort -LoadDistribution Default -EnableFloatingIP | Set-AzLoadBalancer   
```

## <a name="configure-the-listener"></a>Configurer l’écouteur

[!INCLUDE [ag-listener-configure](../../../../includes/virtual-machines-ag-listener-configure.md)]

## <a name="set-the-listener-port-in-sql-server-management-studio"></a>Définissez le port d’écoute dans SQL Server Management Studio

1. Lancez SQL Server Management Studio et connectez-vous au réplica principal.

1. Accédez à **Haute disponibilité AlwaysOn** > **Groupes de disponibilité** > **Écouteurs de groupe de disponibilité**. 

1. Vous devez maintenant voir le nom de l'écouteur que vous avez créé dans le Gestionnaire du cluster de basculement. Cliquez avec le bouton droit sur le nom de l’écouteur, puis sélectionnez **Propriétés**.

1. Dans le champ **Port**, indiquez le numéro de port de l’écouteur du groupe de disponibilité à l’aide du paramètre $EndpointPort utilisé précédemment (valeur par défaut : 1433), puis cliquez sur **OK**.

## <a name="test-the-connection-to-the-listener"></a>Tester la connexion à l’écouteur

Pour tester la connexion :

1. Utilisez le protocole RDP (Remote Desktop Protocol) pour vous connecter à un serveur SQL qui est sur le même réseau virtuel mais qui n’a pas de réplica. Il peut s’agir de l’autre serveur SQL du cluster.

1. Utilisez l’utilitaire **sqlcmd** pour tester la connexion. Par exemple, le script suivant établit une connexion **sqlcmd** avec le réplica principal au moyen de l’écouteur avec une authentification Windows :
   
    ```
    sqlcmd -S <listenerName> -E
    ```
   
    Si l’écouteur utilise un port autre que le port par défaut (1433), spécifiez le port dans la chaîne de connexion. Par exemple, la commande sqlcmd suivante se connecte à un écouteur sur le port 1435 : 
   
    ```
    sqlcmd -S <listenerName>,1435 -E
    ```

La connexion SQLCMD se connecte automatiquement à l’instance SQL Server hébergeant le réplica principal. 

> [!NOTE]
> Vérifiez que le port spécifié est ouvert sur le pare-feu des deux serveurs SQL. Les deux serveurs requièrent une règle de trafic entrant sur le port TCP utilisé. Pour plus d’informations, consultez [Ajouter ou modifier une règle de pare-feu](/previous-versions/orphan-topics/ws.11/cc753558(v=ws.11)). 
> 

## <a name="guidelines-and-limitations"></a>Instructions et limitations

Notez les instructions suivantes concernant l’écouteur de groupe de disponibilité dans Azure utilisant l’équilibrage de charge interne :

* Avec un équilibreur de charge interne, vous n’accédez à l’écouteur qu’à partir du même réseau virtuel.

* Si vous restreignez l’accès avec un groupe de sécurité réseau Azure, assurez-vous que les règles d’autorisation sont les suivantes :
  - Adresses IP principales des machines virtuelles SQL Server
  - Adresses IP flottantes de l’équilibreur de charge pour l’écouteur GA
  - Adresse IP principale du cluster, le cas échéant.

* Créez un point de terminaison de service lorsque vous utilisez un équilibreur de charge standard avec Stockage Azure pour le témoin cloud. Pour plus d'informations, consultez [Accorder l'accès à partir d'un réseau virtuel](../../../storage/common/storage-network-security.md?toc=%2fazure%2fvirtual-network%2ftoc.json#grant-access-from-a-virtual-network).

## <a name="for-more-information"></a>Informations supplémentaires

Pour plus d’informations, consultez [Configure Always On availability group in Azure VM manually (Configuration manuelle d’un groupe de disponibilité Always On dans une machine virtuelle Azure)](availability-group-manually-configure-tutorial.md).

## <a name="powershell-cmdlets"></a>Applets de commande PowerShell

Utilisez les applets de commande PowerShell suivantes pour créer un équilibreur de charge interne pour les machines virtuelles Azure.

* [New-AzLoadBalancer](/powershell/module/Azurerm.Network/New-AzureRmLoadBalancer) permet de créer un équilibreur de charge. 
* [New-AzLoadBalancerFrontendIpConfig](/powershell/module/Azurerm.Network/New-AzureRmLoadBalancerFrontendIpConfig) permet de créer une configuration d'adresse IP frontale pour un équilibreur de charge. 
* [New-AzLoadBalancerRuleConfig](/powershell/module/Azurerm.Network/New-AzureRmLoadBalancerRuleConfig) permet de créer une configuration de règle pour un équilibreur de charge. 
* [New-AzLoadBalancerBackendAddressPoolConfig](/powershell/module/Azurerm.Network/New-AzureRmLoadBalancerBackendAddressPoolConfig) permet de créer une configuration de pool d'adresses principales pour un équilibreur de charge. 
* [New-AzLoadBalancerProbeConfig](/powershell/module/Azurerm.Network/New-AzureRmLoadBalancerProbeConfig) permet de créer une configuration de probe pour un équilibreur de charge.
* [Remove-AzLoadBalancer](/powershell/module/Azurerm.Network/Remove-AzureRmLoadBalancer) permet de supprimer un équilibreur de charge dans un groupe de ressources Azure.