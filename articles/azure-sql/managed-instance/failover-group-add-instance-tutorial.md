---
title: 'Tutoriel : Ajouter une instance SQL Managed Instance à un groupe de basculement'
titleSuffix: Azure SQL Managed Instance
description: Dans ce tutoriel, vous allez apprendre à créer un groupe de basculement entre une instance managée Azure SQL Managed Instance principale et une instance managée Azure SQL Managed Instance secondaire.
services: sql-database
ms.service: sql-managed-instance
ms.subservice: high-availability
ms.custom: sqldbrb=1, devx-track-azurepowershell
ms.devlang: ''
ms.topic: tutorial
author: stevestein
ms.author: sstein
ms.reviewer: sashan
ms.date: 08/27/2019
ms.openlocfilehash: 92d1ce51306e846e2d842bef33bb9782da14019a
ms.sourcegitcommit: 1cf157f9a57850739adef72219e79d76ed89e264
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2020
ms.locfileid: "94593992"
---
# <a name="tutorial-add-sql-managed-instance-to-a-failover-group"></a>Tutoriel : Ajouter une instance managée SQL Managed Instance à un groupe de basculement
[!INCLUDE[appliesto-sqlmi](../includes/appliesto-sqlmi.md)]

Ajoutez des instances managées d’Azure SQL Managed Instance à un groupe de basculement. Dans cet article, vous allez apprendre à :

> [!div class="checklist"]
> - Créer une instance managée principale.
> - Créez une instance managée secondaire dans le cadre d’un [groupe de basculement](../database/auto-failover-group-overview.md). 
> - Tester le basculement.

  > [!NOTE]
  > - En parcourant ce tutoriel, assurez-vous de configurer vos ressources dans le respect des [conditions préalables à la configuration de groupes de basculement pour SQL Managed Instance](../database/auto-failover-group-overview.md#enabling-geo-replication-between-managed-instances-and-their-vnets). 
  > - La création d’une instance managée peut prendre beaucoup de temps. En conséquence, ce didacticiel peut prendre plusieurs heures. Pour plus d’informations sur les délais de provisionnement, consultez [Opérations de gestion SQL Managed Instance](sql-managed-instance-paas-overview.md#management-operations). 
  > - Les instances managées participant à un groupe de basculement nécessitent soit [Azure ExpressRoute](../../expressroute/expressroute-howto-circuit-portal-resource-manager.md) soit deux passerelles VPN connectées. L’homologation Global VNet Peering n’est pas prise en charge. Ce didacticiel décrit les étapes de création et de connexion des passerelles VPN. Ignorez ces étapes si vous avez déjà configuré ExpressRoute. 


## <a name="prerequisites"></a>Prérequis

# <a name="portal"></a>[Portail](#tab/azure-portal)
Pour suivre ce tutoriel, veillez à disposer des éléments suivants : 

- Un abonnement Azure. [Créez un compte gratuit](https://azure.microsoft.com/free/) si vous n’en avez pas déjà un.


# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Pour suivre le tutoriel, vérifiez que les prérequis ci-dessous sont remplis :

- Un abonnement Azure. [Créez un compte gratuit](https://azure.microsoft.com/free/) si vous n’en avez pas déjà un.
- [Azure PowerShell](/powershell/azure/)

---


## <a name="create-a-resource-group-and-primary-managed-instance"></a>Créer un groupe de ressources et une instance managée principale

Au cours de cette étape, vous allez créer le groupe de ressources et l’instance managée principale pour votre groupe de basculement à l’aide du portail Azure ou de PowerShell. 

Déployez les deux instances gérées dans des [régions appairées](../../best-practices-availability-paired-regions.md) pour des raisons de performances. Les instances managées résidant dans des régions appairées géographiquement offrent des performances sensiblement meilleures que celles résidant dans des régions non appairées. 


# <a name="portal"></a>[Portail](#tab/azure-portal) 

Créez le groupe de ressources et votre instance managée principale à l’aide du portail Azure. 

1. Dans le menu de gauche du portail Azure, sélectionnez **Azure SQL**. Si **Azure SQL** ne figure pas dans la liste, sélectionnez **Tous les services**, puis tapez `Azure SQL` dans la zone de recherche. (Facultatif) Sélectionnez l’étoile en regard d’**Azure SQL** pour l’ajouter aux favoris et l’ajouter en tant qu’élément dans le volet de navigation de gauche. 
1. Sélectionnez **+Ajouter** pour ouvrir la page **Sélectionner l’option de déploiement SQL**. Vous pouvez afficher des informations supplémentaires sur les différentes bases de données en sélectionnant **Afficher les détails** sur la vignette **Bases de données**.
1. Sélectionnez **Créer** dans la mosaïque **Instances gérées SQL**. 

    ![Sélectionner SQL Managed Instance](./media/failover-group-add-instance-tutorial/select-managed-instance.png)

1. Dans la page **Créer une instance managée Azure SQL** sous l’onglet **De base** :
    1. Sous **Détails du projet**, sélectionnez votre **Abonnement** dans la liste déroulante, puis choisissez de **Créer** un groupe de ressources. Saisissez un nom pour votre groupe de ressources, par exemple `myResourceGroup`. 
    1. Sous **Détails de l’instance managée SQL**, indiquez le nom de votre instance managée et la région où vous souhaitez déployer votre instance managée. Laissez **Calcul + Stockage** aux valeurs par défaut. 
    1. Sous **Compte administrateur**, fournissez un identifiant d’administrateur, comme `azureuser`, et un mot de passe d’administrateur complexe. 

    ![Créer une instance managée principale](./media/failover-group-add-instance-tutorial/primary-sql-mi-values.png)

1. Laissez le reste des paramètres aux valeurs par défaut, puis sélectionnez **Évaluer + créer** pour évaluer les paramètres de votre instance managée SQL. 
1. Sélectionnez **Créer** pour créer votre instance managée principale. 

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Créez votre groupe de ressources et l’instance managée principale à l’aide de PowerShell. 

   ```powershell-interactive
   # Connect-AzAccount
   # The SubscriptionId in which to create these objects
   $SubscriptionId = '<Subscription-ID>'
   # Create a random identifier to use as subscript for the different resource names
   $randomIdentifier = $(Get-Random)
   # Set the resource group name and location for SQL Managed Instance
   $resourceGroupName = "myResourceGroup-$randomIdentifier"
   $location = "eastus"
   $drLocation = "eastus2"
   
   # Set the networking values for your primary managed instance
   $primaryVNet = "primaryVNet-$randomIdentifier"
   $primaryAddressPrefix = "10.0.0.0/16"
   $primaryDefaultSubnet = "primaryDefaultSubnet-$randomIdentifier"
   $primaryDefaultSubnetAddress = "10.0.0.0/24"
   $primaryMiSubnetName = "primaryMISubnet-$randomIdentifier"
   $primaryMiSubnetAddress = "10.0.0.0/24"
   $primaryMiGwSubnetAddress = "10.0.255.0/27"
   $primaryGWName = "primaryGateway-$randomIdentifier"
   $primaryGWPublicIPAddress = $primaryGWName + "-ip"
   $primaryGWIPConfig = $primaryGWName + "-ipc"
   $primaryGWAsn = 61000
   $primaryGWConnection = $primaryGWName + "-connection"
   
   
   # Set the networking values for your secondary managed instance
   $secondaryVNet = "secondaryVNet-$randomIdentifier"
   $secondaryAddressPrefix = "10.128.0.0/16"
   $secondaryDefaultSubnet = "secondaryDefaultSubnet-$randomIdentifier"
   $secondaryDefaultSubnetAddress = "10.128.0.0/24"
   $secondaryMiSubnetName = "secondaryMISubnet-$randomIdentifier"
   $secondaryMiSubnetAddress = "10.128.0.0/24"
   $secondaryMiGwSubnetAddress = "10.128.255.0/27"
   $secondaryGWName = "secondaryGateway-$randomIdentifier"
   $secondaryGWPublicIPAddress = $secondaryGWName + "-IP"
   $secondaryGWIPConfig = $secondaryGWName + "-ipc"
   $secondaryGWAsn = 62000
   $secondaryGWConnection = $secondaryGWName + "-connection"
   
   
   
   # Set the SQL Managed Instance name for the new managed instances
   $primaryInstance = "primary-mi-$randomIdentifier"
   $secondaryInstance = "secondary-mi-$randomIdentifier"
   
   # Set the admin login and password for SQL Managed Instance
   $secpasswd = "PWD27!"+(New-Guid).Guid | ConvertTo-SecureString -AsPlainText -Force
   $mycreds = New-Object System.Management.Automation.PSCredential ("azureuser", $secpasswd)
   
   
   # Set the SQL Managed Instance service tier, compute level, and license mode
   $edition = "General Purpose"
   $vCores = 8
   $maxStorage = 256
   $computeGeneration = "Gen5"
   $license = "LicenseIncluded" #"BasePrice" or LicenseIncluded if you have don't have SQL Server license that can be used for AHB discount
   
   # Set failover group details
   $vpnSharedKey = "mi1mi2psk"
   $failoverGroupName = "failovergroup-$randomIdentifier"
   
   # Show randomized variables
   Write-host "Resource group name is" $resourceGroupName
   Write-host "Password is" $secpasswd
   Write-host "Primary Virtual Network name is" $primaryVNet
   Write-host "Primary default subnet name is" $primaryDefaultSubnet
   Write-host "Primary SQL Managed Instance subnet name is" $primaryMiSubnetName
   Write-host "Secondary Virtual Network name is" $secondaryVNet
   Write-host "Secondary default subnet name is" $secondaryDefaultSubnet
   Write-host "Secondary SQL Managed Instance subnet name is" $secondaryMiSubnetName
   Write-host "Primary SQL Managed Instance name is" $primaryInstance
   Write-host "Secondary SQL Managed Instance name is" $secondaryInstance
   Write-host "Failover group name is" $failoverGroupName
   
   # Suppress networking breaking changes warning (https://aka.ms/azps-changewarnings
   Set-Item Env:\SuppressAzurePowerShellBreakingChangeWarnings "true"
   
   # Set the subscription context
   Set-AzContext -SubscriptionId $subscriptionId 
   
   # Create the resource group
   Write-host "Creating resource group..."
   $resourceGroup = New-AzResourceGroup -Name $resourceGroupName -Location $location -Tag @{Owner="SQLDB-Samples"}
   $resourceGroup
   
   # Configure the primary virtual network
   Write-host "Creating primary virtual network..."
   $primaryVirtualNetwork = New-AzVirtualNetwork `
                         -ResourceGroupName $resourceGroupName `
                         -Location $location `
                         -Name $primaryVNet `
                         -AddressPrefix $primaryAddressPrefix
   
                     Add-AzVirtualNetworkSubnetConfig `
                         -Name $primaryMiSubnetName `
                         -VirtualNetwork $primaryVirtualNetwork `
                         -AddressPrefix $PrimaryMiSubnetAddress `
                     | Set-AzVirtualNetwork
   $primaryVirtualNetwork
   
   
   # Configure the primary managed instance subnet
   Write-host "Configuring primary MI subnet..."
   $primaryVirtualNetwork = Get-AzVirtualNetwork -Name $primaryVNet -ResourceGroupName $resourceGroupName
   
   
   $primaryMiSubnetConfig = Get-AzVirtualNetworkSubnetConfig `
                           -Name $primaryMiSubnetName `
                           -VirtualNetwork $primaryVirtualNetwork
   $primaryMiSubnetConfig
   
   # Configure the network security group management service
   Write-host "Configuring primary MI subnet..."
   
   $primaryMiSubnetConfigId = $primaryMiSubnetConfig.Id
   
   $primaryNSGMiManagementService = New-AzNetworkSecurityGroup `
                         -Name 'primaryNSGMiManagementService' `
                         -ResourceGroupName $resourceGroupName `
                         -location $location
   $primaryNSGMiManagementService
   
   # Configure the route table management service
   Write-host "Configuring primary MI route table management service..."
   
   $primaryRouteTableMiManagementService = New-AzRouteTable `
                         -Name 'primaryRouteTableMiManagementService' `
                         -ResourceGroupName $resourceGroupName `
                         -location $location
   $primaryRouteTableMiManagementService
   
   # Configure the primary network security group
   Write-host "Configuring primary network security group..."
   Set-AzVirtualNetworkSubnetConfig `
                         -VirtualNetwork $primaryVirtualNetwork `
                         -Name $primaryMiSubnetName `
                         -AddressPrefix $PrimaryMiSubnetAddress `
                         -NetworkSecurityGroup $primaryNSGMiManagementService `
                         -RouteTable $primaryRouteTableMiManagementService | `
                       Set-AzVirtualNetwork
   
   Get-AzNetworkSecurityGroup `
                         -ResourceGroupName $resourceGroupName `
                         -Name "primaryNSGMiManagementService" `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 100 `
                         -Name "allow_management_inbound" `
                         -Access Allow `
                         -Protocol Tcp `
                         -Direction Inbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix * `
                         -DestinationPortRange 9000,9003,1438,1440,1452 `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 200 `
                         -Name "allow_misubnet_inbound" `
                         -Access Allow `
                         -Protocol * `
                         -Direction Inbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix $PrimaryMiSubnetAddress `
                         -DestinationPortRange * `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 300 `
                         -Name "allow_health_probe_inbound" `
                         -Access Allow `
                         -Protocol * `
                         -Direction Inbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix AzureLoadBalancer `
                         -DestinationPortRange * `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 1000 `
                         -Name "allow_tds_inbound" `
                         -Access Allow `
                         -Protocol Tcp `
                         -Direction Inbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix VirtualNetwork `
                         -DestinationPortRange 1433 `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 1100 `
                         -Name "allow_redirect_inbound" `
                         -Access Allow `
                         -Protocol Tcp `
                         -Direction Inbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix VirtualNetwork `
                         -DestinationPortRange 11000-11999 `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 1200 `
                         -Name "allow_geodr_inbound" `
                         -Access Allow `
                         -Protocol Tcp `
                         -Direction Inbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix VirtualNetwork `
                         -DestinationPortRange 5022 `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 4096 `
                         -Name "deny_all_inbound" `
                         -Access Deny `
                         -Protocol * `
                         -Direction Inbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix * `
                         -DestinationPortRange * `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 100 `
                         -Name "allow_management_outbound" `
                         -Access Allow `
                         -Protocol Tcp `
                         -Direction Outbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix * `
                         -DestinationPortRange 80,443,12000 `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 200 `
                         -Name "allow_misubnet_outbound" `
                         -Access Allow `
                         -Protocol * `
                         -Direction Outbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix * `
                         -DestinationPortRange * `
                         -DestinationAddressPrefix $PrimaryMiSubnetAddress `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 1100 `
                         -Name "allow_redirect_outbound" `
                         -Access Allow `
                         -Protocol Tcp `
                         -Direction Outbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix VirtualNetwork `
                         -DestinationPortRange 11000-11999 `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 1200 `
                         -Name "allow_geodr_outbound" `
                         -Access Allow `
                         -Protocol Tcp `
                         -Direction Outbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix VirtualNetwork `
                         -DestinationPortRange 5022 `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 4096 `
                         -Name "deny_all_outbound" `
                         -Access Deny `
                         -Protocol * `
                         -Direction Outbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix * `
                         -DestinationPortRange * `
                         -DestinationAddressPrefix * `
                       | Set-AzNetworkSecurityGroup
   Write-host "Primary network security group configured successfully."
   
   
   Get-AzRouteTable `
                         -ResourceGroupName $resourceGroupName `
                         -Name "primaryRouteTableMiManagementService" `
                       | Add-AzRouteConfig `
                         -Name "primaryToMIManagementService" `
                         -AddressPrefix 0.0.0.0/0 `
                         -NextHopType Internet `
                       | Add-AzRouteConfig `
                         -Name "ToLocalClusterNode" `
                         -AddressPrefix $PrimaryMiSubnetAddress `
                         -NextHopType VnetLocal `
                       | Set-AzRouteTable
   Write-host "Primary network route table configured successfully."
   
   
   # Create the primary managed instance
   
   Write-host "Creating primary SQL Managed Instance..."
   Write-host "This will take some time, see https://docs.microsoft.com/azure/sql-database/sql-database-managed-instance#managed-instance-management-operations or more information."
   New-AzSqlInstance -Name $primaryInstance `
                         -ResourceGroupName $resourceGroupName `
                         -Location $location `
                         -SubnetId $primaryMiSubnetConfigId `
                         -AdministratorCredential $mycreds `
                         -StorageSizeInGB $maxStorage `
                         -VCore $vCores `
                         -Edition $edition `
                         -ComputeGeneration $computeGeneration `
                         -LicenseType $license
   Write-host "Primary SQL Managed Instance created successfully."
   ```

Cette partie du tutoriel utilise les cmdlets PowerShell suivantes :

| Commande | Notes |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Crée un groupe de ressources Azure.  |
| [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) | Créer un réseau virtuel.  |
| [Add-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/add-azvirtualnetworksubnetconfig) | Ajoute une configuration de sous-réseau à un réseau virtuel. | 
| [Get-AzVirtualNetwork](/powershell/module/az.network/get-azvirtualnetwork) | Obtient un réseau virtuel dans un groupe de ressources. | 
| [Get-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/get-azvirtualnetworksubnetconfig) | Obtient un sous-réseau dans un réseau virtuel. | 
| [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup) | Crée un groupe de sécurité réseau. | 
| [New-AzRouteTable](/powershell/module/az.network/new-azroutetable) | Crée une table de routage. |
| [Set-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/set-azvirtualnetworksubnetconfig) | Met à jour une configuration de sous-réseau pour un réseau virtuel.  |
| [Set-AzVirtualNetwork](/powershell/module/az.network/set-azvirtualnetwork) | Met à jour un réseau virtuel.  |
| [Get-AzNetworkSecurityGroup](/powershell/module/az.network/get-aznetworksecuritygroup) | Obtient un groupe de sécurité réseau. |
| [Add-AzNetworkSecurityRuleConfig](/powershell/module/az.network/add-aznetworksecurityruleconfig)| Ajoute une configuration de règle de sécurité réseau à un groupe de sécurité réseau. |
| [Set-AzNetworkSecurityGroup](/powershell/module/az.network/set-aznetworksecuritygroup) | Met à jour un groupe de sécurité réseau.  | 
| [Add-AzRouteConfig](/powershell/module/az.network/add-azrouteconfig) | Ajoute un itinéraire à une table de routage. |
| [Set-AzRouteTable](/powershell/module/az.network/set-azroutetable) | Met à jour une table de routage.  |
| [New-AzSqlInstance](/powershell/module/az.sql/new-azsqlinstance) | Crée une instance managée.  |

---

## <a name="create-secondary-virtual-network"></a>Créer le réseau virtuel secondaire

Si vous utilisez le portail Azure pour créer votre instance managée, vous devez créer le réseau virtuel séparément, car les plages du sous-réseau de l’instance managée principale et secondaire ne doivent pas se chevaucher. Si vous utilisez PowerShell pour configurer votre instance managée, passez directement à l’étape 3. 

# <a name="portal"></a>[Portail](#tab/azure-portal) 

Pour vérifier la plage du sous-réseau de votre réseau virtuel principal, procédez comme suit :

1. Dans le [portail Azure](https://portal.azure.com), accédez à votre groupe de ressources et sélectionnez le réseau virtuel pour votre instance principale.  
2. Sélectionnez **Sous-réseaux** sous **Paramètres** et notez la **Plage d’adresses**. La plage d’adresses du sous-réseau du réseau virtuel pour l’instance managée secondaire ne peut pas la chevaucher. 


   ![Sous-réseau principal](./media/failover-group-add-instance-tutorial/verify-primary-subnet-range.png)

Pour créer un réseau virtuel, procédez comme suit :

1. Dans le [portail Azure](https://portal.azure.com), sélectionnez **Créer une ressource** et recherchez *réseau virtuel*. 
1. Sélectionnez l'option **Réseau virtuel** publiée par Microsoft, puis sélectionnez **Créer** sur la page suivante. 
1. Renseignez les champs requis pour configurer le réseau virtuel de votre instance managée secondaire, puis sélectionnez **Créer**. 

   Le tableau suivant montre les valeurs nécessaires pour le réseau virtuel secondaire :

    | **Champ** | Valeur |
    | --- | --- |
    | **Nom** |  Le nom du réseau virtuel qui sera utilisé par l’instance gérée secondaire, par exemple `vnet-sql-mi-secondary`. |
    | **Espace d’adressage** | L’espace d’adressage pour votre réseau virtuel, par exemple `10.128.0.0/16`. | 
    | **Abonnement** | L’abonnement dans lequel votre instance managée principale et le groupe de ressources se trouvent. |
    | **Région** | Emplacement dans lequel vous allez déployer votre instance managée secondaire. |
    | **Sous-réseau** | Le nom de votre sous-réseau. `default` est fourni pour vous par défaut. |
    | **Plage d’adresses**| La plage d’adresses de votre sous-réseau. Celle-ci doit être différente de la plage d’adresses de sous-réseau utilisée par le réseau virtuel de votre instance managée principale, comme `10.128.0.0/24`.  |
    | &nbsp; | &nbsp; |

    ![Valeurs de réseau virtuel secondaire](./media/failover-group-add-instance-tutorial/secondary-virtual-network.png)

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Cette étape n’est utile que si vous utilisez le portail Azure pour déployer l’instance managée SQL. Passez à l’étape 3 si vous utilisez PowerShell. 

---

## <a name="create-a-secondary-managed-instance"></a>Créer une instance managée secondaire
Lors de cette étape, vous allez créer une instance gérée secondaire dans le portail Azure, ce qui configurera également la mise en réseau entre les deux instances managées. 

Votre deuxième instance gérée doit :
- Être vide. 
- Avoir un sous-réseau et une plage d’adresses IP différents de ceux de l’instance managée principale. 

# <a name="portal"></a>[Portail](#tab/azure-portal) 

Créez l’instance managée secondaire à l’aide du portail Azure. 

1. Dans le menu de gauche du portail Azure, sélectionnez **Azure SQL**. Si **Azure SQL** ne figure pas dans la liste, sélectionnez **Tous les services**, puis tapez `Azure SQL` dans la zone de recherche. (Facultatif) Sélectionnez l’étoile en regard d’**Azure SQL** pour l’ajouter aux favoris et l’ajouter en tant qu’élément dans le volet de navigation de gauche. 
1. Sélectionnez **+Ajouter** pour ouvrir la page **Sélectionner l’option de déploiement SQL**. Vous pouvez afficher des informations supplémentaires sur les différentes bases de données en sélectionnant **Afficher les détails** sur la vignette **Bases de données**.
1. Sélectionnez **Créer** dans la vignette **Instances managées SQL**. 

    ![Sélectionner SQL Managed Instance](./media/failover-group-add-instance-tutorial/select-managed-instance.png)

1. Sous l’onglet **De base** de la page **Créer une instance managée Azure SQL**, renseignez les champs requis pour configurer votre instance managée secondaire. 

   Le tableau suivant montre les valeurs nécessaires pour l’instance managée secondaire :
 
    | **Champ** | Valeur |
    | --- | --- |
    | **Abonnement** |  L’abonnement dans lequel votre instance managée principale se trouve. |
    | **Groupe de ressources**| Le groupe de ressources où se trouve votre instance managée principale. |
    | **Nom de l’instance gérée SQL** | Le nom de votre nouvelle instance managée secondaire, par exemple `sql-mi-secondary`.  | 
    | **Région**| Emplacement de votre instance managée secondaire.  |
    | **Connexion administrateur de l’instance gérée SQL** | L’identifiant que vous souhaitez utiliser pour votre nouvelle instance managée secondaire, par exemple `azureuser`. |
    | **Mot de passe** | Un mot de passe complexe qui sera utilisé par la connexion de l’administrateur à la nouvelle instance gérée secondaire.  |
    | &nbsp; | &nbsp; |

1. Sous l'onglet **Mise en réseau**, pour le **Réseau virtuel**, sélectionnez le réseau virtuel que vous avez créé pour l’instance managée secondaire dans la liste déroulante.

   ![Réseau MI secondaire](./media/failover-group-add-instance-tutorial/networking-settings-for-secondary-mi.png)

1. Sous l’onglet **Paramètres supplémentaires**, pour **Géoréplication**, choisissez **Oui** pour _Utiliser pour le basculement secondaire_. Sélectionnez l’instance managée principale dans la liste déroulante. 
    
   Assurez-vous que le classement et le fuseau horaire correspondent à ceux de l’instance managée principale. L’instance managée principale créée dans ce tutoriel utilisait le classement `SQL_Latin1_General_CP1_CI_AS` par défaut et le fuseau horaire `(UTC) Coordinated Universal Time`. 

   ![Mise en réseau de l’instance managée secondaire](./media/failover-group-add-instance-tutorial/secondary-mi-failover.png)

1. Sélectionnez **Vérifier + créer** pour passer en revue les paramètres de votre instance managée secondaire. 
1. Sélectionnez **Créer** pour créer votre instance managée secondaire. 

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Créez l’instance managée secondaire à l’aide de PowerShell. 

   ```powershell-interactive
   # Configure the secondary virtual network
   Write-host "Configuring secondary virtual network..."
   
   $SecondaryVirtualNetwork = New-AzVirtualNetwork `
                         -ResourceGroupName $resourceGroupName `
                         -Location $drlocation `
                         -Name $secondaryVNet `
                         -AddressPrefix $secondaryAddressPrefix
   
   Add-AzVirtualNetworkSubnetConfig `
                         -Name $secondaryMiSubnetName `
                         -VirtualNetwork $SecondaryVirtualNetwork `
                         -AddressPrefix $secondaryMiSubnetAddress `
                       | Set-AzVirtualNetwork
   $SecondaryVirtualNetwork
   
   # Configure the secondary managed instance subnet
   Write-host "Configuring secondary MI subnet..."
   
   $SecondaryVirtualNetwork = Get-AzVirtualNetwork -Name $secondaryVNet `
                                   -ResourceGroupName $resourceGroupName
   
   $secondaryMiSubnetConfig = Get-AzVirtualNetworkSubnetConfig `
                           -Name $secondaryMiSubnetName `
                           -VirtualNetwork $SecondaryVirtualNetwork
   $secondaryMiSubnetConfig
   
   # Configure the secondary network security group management service
   Write-host "Configuring secondary network security group management service..."
   
   $secondaryMiSubnetConfigId = $secondaryMiSubnetConfig.Id
   
   $secondaryNSGMiManagementService = New-AzNetworkSecurityGroup `
                         -Name 'secondaryToMIManagementService' `
                         -ResourceGroupName $resourceGroupName `
                         -location $drlocation
   $secondaryNSGMiManagementService
   
   # Configure the secondary route table MI management service
   Write-host "Configuring secondary route table MI management service..."
   
   $secondaryRouteTableMiManagementService = New-AzRouteTable `
                         -Name 'secondaryRouteTableMiManagementService' `
                         -ResourceGroupName $resourceGroupName `
                         -location $drlocation
   $secondaryRouteTableMiManagementService
   
   # Configure the secondary network security group
   Write-host "Configuring secondary network security group..."
   
   Set-AzVirtualNetworkSubnetConfig `
                         -VirtualNetwork $SecondaryVirtualNetwork `
                         -Name $secondaryMiSubnetName `
                         -AddressPrefix $secondaryMiSubnetAddress `
                         -NetworkSecurityGroup $secondaryNSGMiManagementService `
                         -RouteTable $secondaryRouteTableMiManagementService `
                       | Set-AzVirtualNetwork
   
   Get-AzNetworkSecurityGroup `
                         -ResourceGroupName $resourceGroupName `
                         -Name "secondaryToMIManagementService" `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 100 `
                         -Name "allow_management_inbound" `
                         -Access Allow `
                         -Protocol Tcp `
                         -Direction Inbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix * `
                         -DestinationPortRange 9000,9003,1438,1440,1452 `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 200 `
                         -Name "allow_misubnet_inbound" `
                         -Access Allow `
                         -Protocol * `
                         -Direction Inbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix $secondaryMiSubnetAddress `
                         -DestinationPortRange * `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 300 `
                         -Name "allow_health_probe_inbound" `
                         -Access Allow `
                         -Protocol * `
                         -Direction Inbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix AzureLoadBalancer `
                         -DestinationPortRange * `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 1000 `
                         -Name "allow_tds_inbound" `
                         -Access Allow `
                         -Protocol Tcp `
                         -Direction Inbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix VirtualNetwork `
                         -DestinationPortRange 1433 `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 1100 `
                         -Name "allow_redirect_inbound" `
                         -Access Allow `
                         -Protocol Tcp `
                         -Direction Inbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix VirtualNetwork `
                         -DestinationPortRange 11000-11999 `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 1200 `
                         -Name "allow_geodr_inbound" `
                         -Access Allow `
                         -Protocol Tcp `
                         -Direction Inbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix VirtualNetwork `
                         -DestinationPortRange 5022 `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 4096 `
                         -Name "deny_all_inbound" `
                         -Access Deny `
                         -Protocol * `
                         -Direction Inbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix * `
                         -DestinationPortRange * `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 100 `
                         -Name "allow_management_outbound" `
                         -Access Allow `
                         -Protocol Tcp `
                         -Direction Outbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix * `
                         -DestinationPortRange 80,443,12000 `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 200 `
                         -Name "allow_misubnet_outbound" `
                         -Access Allow `
                         -Protocol * `
                         -Direction Outbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix * `
                         -DestinationPortRange * `
                         -DestinationAddressPrefix $secondaryMiSubnetAddress `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 1100 `
                         -Name "allow_redirect_outbound" `
                         -Access Allow `
                         -Protocol Tcp `
                         -Direction Outbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix VirtualNetwork `
                         -DestinationPortRange 11000-11999 `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 1200 `
                         -Name "allow_geodr_outbound" `
                         -Access Allow `
                         -Protocol Tcp `
                         -Direction Outbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix VirtualNetwork `
                         -DestinationPortRange 5022 `
                         -DestinationAddressPrefix * `
                       | Add-AzNetworkSecurityRuleConfig `
                         -Priority 4096 `
                         -Name "deny_all_outbound" `
                         -Access Deny `
                         -Protocol * `
                         -Direction Outbound `
                         -SourcePortRange * `
                         -SourceAddressPrefix * `
                         -DestinationPortRange * `
                         -DestinationAddressPrefix * `
                       | Set-AzNetworkSecurityGroup
   
   
   Get-AzRouteTable `
                         -ResourceGroupName $resourceGroupName `
                         -Name "secondaryRouteTableMiManagementService" `
                       | Add-AzRouteConfig `
                         -Name "secondaryToMIManagementService" `
                         -AddressPrefix 0.0.0.0/0 `
                         -NextHopType Internet `
                       | Add-AzRouteConfig `
                         -Name "ToLocalClusterNode" `
                         -AddressPrefix $secondaryMiSubnetAddress `
                         -NextHopType VnetLocal `
                       | Set-AzRouteTable
   Write-host "Secondary network security group configured successfully."
   
   # Create the secondary managed instance
   
   $primaryManagedInstanceId = Get-AzSqlInstance -Name $primaryInstance -ResourceGroupName $resourceGroupName | Select-Object Id
   
   
   Write-host "Creating secondary SQL Managed Instance..."
   Write-host "This will take some time, see https://docs.microsoft.com/azure/sql-database/sql-database-managed-instance#managed-instance-management-operations or more information."
   New-AzSqlInstance -Name $secondaryInstance `
                     -ResourceGroupName $resourceGroupName `
                     -Location $drLocation `
                     -SubnetId $secondaryMiSubnetConfigId `
                     -AdministratorCredential $mycreds `
                     -StorageSizeInGB $maxStorage `
                     -VCore $vCores `
                     -Edition $edition `
                     -ComputeGeneration $computeGeneration `
                     -LicenseType $license `
                     -DnsZonePartner $primaryManagedInstanceId.Id
   Write-host "Secondary SQL Managed Instance created successfully."
   ```

Cette partie du tutoriel utilise les cmdlets PowerShell suivantes :

| Commande | Notes |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Crée un groupe de ressources Azure.  |
| [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) | Créer un réseau virtuel.  |
| [Add-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/add-azvirtualnetworksubnetconfig) | Ajoute une configuration de sous-réseau à un réseau virtuel. | 
| [Get-AzVirtualNetwork](/powershell/module/az.network/get-azvirtualnetwork) | Obtient un réseau virtuel dans un groupe de ressources. | 
| [Get-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/get-azvirtualnetworksubnetconfig) | Obtient un sous-réseau dans un réseau virtuel. | 
| [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup) | Crée un groupe de sécurité réseau. | 
| [New-AzRouteTable](/powershell/module/az.network/new-azroutetable) | Crée une table de routage. |
| [Set-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/set-azvirtualnetworksubnetconfig) | Met à jour une configuration de sous-réseau pour un réseau virtuel.  |
| [Set-AzVirtualNetwork](/powershell/module/az.network/set-azvirtualnetwork) | Met à jour un réseau virtuel.  |
| [Get-AzNetworkSecurityGroup](/powershell/module/az.network/get-aznetworksecuritygroup) | Obtient un groupe de sécurité réseau. |
| [Add-AzNetworkSecurityRuleConfig](/powershell/module/az.network/add-aznetworksecurityruleconfig)| Ajoute une configuration de règle de sécurité réseau à un groupe de sécurité réseau. |
| [Set-AzNetworkSecurityGroup](/powershell/module/az.network/set-aznetworksecuritygroup) | Met à jour un groupe de sécurité réseau.  | 
| [Add-AzRouteConfig](/powershell/module/az.network/add-azrouteconfig) | Ajoute un itinéraire à une table de routage. |
| [Set-AzRouteTable](/powershell/module/az.network/set-azroutetable) | Met à jour une table de routage.  |
| [New-AzSqlInstance](/powershell/module/az.sql/new-azsqlinstance) | Crée une instance managée.  |

---

## <a name="create-a-primary-gateway"></a>Créer une passerelle principale 

Pour que deux instances gérées participent à un groupe de basculement, il doit y avoir ExpressRoute ou une passerelle configurée entre les réseaux virtuels des deux instances gérées pour permettre la communication réseau. Si vous choisissez de configurer [ExpressRoute](../../expressroute/expressroute-howto-circuit-portal-resource-manager.md) au lieu de connecter deux passerelles VPN, passez directement à l’[étape 7](#create-a-failover-group).  

Cet article explique comment créer les deux passerelles VPN et les connecter, mais vous pouvez passer directement à la création du groupe de basculement si vous avez configuré ExpressRoute à la place. 

> [!NOTE]
> La référence SKU de la passerelle affecte les performances de débit. Ce didacticiel déploie une passerelle avec la référence (`HwGw1`) la plus basique. Pour obtenir un débit plus élevé, déployez une référence (SKU) supérieure (par exemple, `VpnGw3`). Pour toutes les options disponibles, consultez [Références (SKU) de passerelle](../../vpn-gateway/vpn-gateway-about-vpngateways.md#benchmark)

# <a name="portal"></a>[Portail](#tab/azure-portal)

Créez la passerelle pour le réseau virtuel de votre instance managée principale à l’aide du portail Azure. 


1. Dans le [portail Azure](https://portal.azure.com), accédez à votre groupe de ressources et sélectionnez la ressource **Réseau virtuel** pour votre instance managée principale. 
1. Sélectionnez **Sous-réseaux** sous **Paramètres**, puis choisissez d’ajouter un nouveau **Sous-réseau de passerelle**. Laissez les valeurs par défaut. 

   ![Ajouter une passerelle pour l’instance managée principale](./media/failover-group-add-instance-tutorial/add-subnet-gateway-primary-vnet.png)

1. Une fois la passerelle de sous-réseau créée, sélectionnez **Créer une ressource** dans le volet de navigation gauche, puis saisissez `Virtual network gateway` dans la zone de recherche. Sélectionnez la ressource de **Passerelle de réseau virtuel** publiée par **Microsoft**. 

   ![Créer une passerelle de réseau virtuel](./media/failover-group-add-instance-tutorial/create-virtual-network-gateway.png)

1. Renseignez les champs requis pour configurer la passerelle de votre instance managée principale. 

   Le tableau suivant montre les valeurs nécessaires pour la passerelle de l’instance managée principale :
 
    | **Champ** | Valeur |
    | --- | --- |
    | **Abonnement** |  L’abonnement dans lequel votre instance managée principale se trouve. |
    | **Nom** | Le nom de votre passerelle de réseau virtuel, par exemple `primary-mi-gateway`. | 
    | **Région** | La région dans laquelle se trouve votre instance gérée principale. |
    | **Type de passerelle** | Sélectionnez **VPN**. |
    | **Type de VPN** | Sélectionnez **Route-based**. |
    | **Référence (SKU)**| Laissez la valeur `VpnGw1` par défaut. |
    | **Réseau virtuel**| Sélectionnez le réseau virtuel créé dans la section 2, par exemple `vnet-sql-mi-primary`. |
    | **Adresse IP publique**| Sélectionnez **Créer nouveau**. |
    | **Nom de l’adresse IP publique**| Entrez un nom pour votre adresse IP, par exemple `primary-gateway-IP`. |
    | &nbsp; | &nbsp; |

1. Laissez les autres valeurs par défaut, puis sélectionnez **Vérifier + créer** pour passer en revue les paramètres de votre passerelle de réseau virtuel.

   ![Paramètres de la passerelle principale](./media/failover-group-add-instance-tutorial/settings-for-primary-gateway.png)

1. Sélectionnez **Créer** pour créer votre passerelle de réseau virtuel. 


# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Créez la passerelle pour le réseau virtuel de votre instance managée principale à l’aide de PowerShell. 

   ```powershell-interactive
   # Create the primary gateway
   Write-host "Adding GatewaySubnet to primary VNet..."
   Get-AzVirtualNetwork `
                     -Name $primaryVNet `
                     -ResourceGroupName $resourceGroupName `
                   | Add-AzVirtualNetworkSubnetConfig `
                     -Name "GatewaySubnet" `
                     -AddressPrefix $primaryMiGwSubnetAddress `
                   | Set-AzVirtualNetwork
   
   $primaryVirtualNetwork  = Get-AzVirtualNetwork `
                     -Name $primaryVNet `
                     -ResourceGroupName $resourceGroupName
   $primaryGatewaySubnet = Get-AzVirtualNetworkSubnetConfig `
                     -Name "GatewaySubnet" `
                     -VirtualNetwork $primaryVirtualNetwork
   
   Write-host "Creating primary gateway..."
   Write-host "This will take some time."
   $primaryGWPublicIP = New-AzPublicIpAddress -Name $primaryGWPublicIPAddress -ResourceGroupName $resourceGroupName `
            -Location $location -AllocationMethod Dynamic
   $primaryGatewayIPConfig = New-AzVirtualNetworkGatewayIpConfig -Name $primaryGWIPConfig `
            -Subnet $primaryGatewaySubnet -PublicIpAddress $primaryGWPublicIP
   
   $primaryGateway = New-AzVirtualNetworkGateway -Name $primaryGWName -ResourceGroupName $resourceGroupName `
       -Location $location -IpConfigurations $primaryGatewayIPConfig -GatewayType Vpn `
       -VpnType RouteBased -GatewaySku VpnGw1 -EnableBgp $true -Asn $primaryGWAsn
   $primaryGateway
   ```

Cette partie du tutoriel utilise les cmdlets PowerShell suivantes :

| Commande | Notes |
|---|---|
| [Get-AzVirtualNetwork](/powershell/module/az.network/get-azvirtualnetwork) | Obtient un réseau virtuel dans un groupe de ressources. |
| [Add-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/add-azvirtualnetworksubnetconfig) | Ajoute une configuration de sous-réseau à un réseau virtuel. | 
| [Set-AzVirtualNetwork](/powershell/module/az.network/set-azvirtualnetwork) | Met à jour un réseau virtuel.  |
| [Get-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/get-azvirtualnetworksubnetconfig) | Obtient un sous-réseau dans un réseau virtuel. |
| [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) | Crée une adresse IP publique.  | 
| [New-AzVirtualNetworkGatewayIpConfig](/powershell/module/az.network/new-azvirtualnetworkgatewayipconfig) | Crée une configuration IP pour une passerelle de réseau virtuel. |
| [New-AzVirtualNetworkGateway](/powershell/module/az.network/new-azvirtualnetworkgateway) | Crée une passerelle de réseau virtuel. |


---


## <a name="create-secondary-gateway"></a>Créer une passerelle secondaire 
Dans cette étape, créez la passerelle pour le réseau virtuel de votre instance managée secondaire à l’aide du portail Azure. 


# <a name="portal"></a>[Portail](#tab/azure-portal)

Via le portail Azure, répétez les étapes de la section précédente pour créer le sous-réseau et la passerelle du réseau virtuel pour l’instance managée secondaire. Renseignez les champs requis pour configurer la passerelle de votre instance managée secondaire. 

   Le tableau suivant montre les valeurs nécessaires pour la passerelle de l’instance managée secondaire :

   | **Champ** | Valeur |
   | --- | --- |
   | **Abonnement** |  L’abonnement dans lequel votre instance managée secondaire se trouve. |
   | **Nom** | Le nom de votre passerelle de réseau virtuel, par exemple `secondary-mi-gateway`. | 
   | **Région** | La région dans laquelle votre instance managée secondaire se trouve. |
   | **Type de passerelle** | Sélectionnez **VPN**. |
   | **Type de VPN** | Sélectionnez **Route-based**. |
   | **Référence (SKU)**| Laissez la valeur `VpnGw1` par défaut. |
   | **Réseau virtuel**| Sélectionnez le réseau virtuel pour l’instance managée secondaire, par exemple `vnet-sql-mi-secondary`. |
   | **Adresse IP publique**| Sélectionnez **Créer nouveau**. |
   | **Nom de l’adresse IP publique**| Entrez un nom pour votre adresse IP, par exemple `secondary-gateway-IP`. |
   | &nbsp; | &nbsp; |

   ![Paramètres de la passerelle secondaire](./media/failover-group-add-instance-tutorial/settings-for-secondary-gateway.png)


# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Créez la passerelle pour le réseau virtuel de l’instance managée secondaire à l’aide de PowerShell. 

   ```powershell-interactive
   # Create the secondary gateway
   Write-host "Creating secondary gateway..."
   
   Write-host "Adding GatewaySubnet to secondary VNet..."
   Get-AzVirtualNetwork `
                     -Name $secondaryVNet `
                     -ResourceGroupName $resourceGroupName `
                   | Add-AzVirtualNetworkSubnetConfig `
                     -Name "GatewaySubnet" `
                     -AddressPrefix $secondaryMiGwSubnetAddress `
                   | Set-AzVirtualNetwork
   
   $secondaryVirtualNetwork  = Get-AzVirtualNetwork `
                     -Name $secondaryVNet `
                     -ResourceGroupName $resourceGroupName
   $secondaryGatewaySubnet = Get-AzVirtualNetworkSubnetConfig `
                     -Name "GatewaySubnet" `
                     -VirtualNetwork $secondaryVirtualNetwork
   $drLocation = $secondaryVirtualNetwork.Location
   
   Write-host "Creating secondary gateway..."
   Write-host "This will take some time."
   $secondaryGWPublicIP = New-AzPublicIpAddress -Name $secondaryGWPublicIPAddress -ResourceGroupName $resourceGroupName `
            -Location $drLocation -AllocationMethod Dynamic
   $secondaryGatewayIPConfig = New-AzVirtualNetworkGatewayIpConfig -Name $secondaryGWIPConfig `
            -Subnet $secondaryGatewaySubnet -PublicIpAddress $secondaryGWPublicIP
   
   $secondaryGateway = New-AzVirtualNetworkGateway -Name $secondaryGWName -ResourceGroupName $resourceGroupName `
       -Location $drLocation -IpConfigurations $secondaryGatewayIPConfig -GatewayType Vpn `
       -VpnType RouteBased -GatewaySku VpnGw1 -EnableBgp $true -Asn $secondaryGWAsn
   $secondaryGateway
   ```

Cette partie du tutoriel utilise les cmdlets PowerShell suivantes :

| Commande | Notes |
|---|---|
| [Get-AzVirtualNetwork](/powershell/module/az.network/get-azvirtualnetwork) | Obtient un réseau virtuel dans un groupe de ressources. |
| [Add-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/add-azvirtualnetworksubnetconfig) | Ajoute une configuration de sous-réseau à un réseau virtuel. | 
| [Set-AzVirtualNetwork](/powershell/module/az.network/set-azvirtualnetwork) | Met à jour un réseau virtuel.  |
| [Get-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/get-azvirtualnetworksubnetconfig) | Obtient un sous-réseau dans un réseau virtuel. |
| [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) | Crée une adresse IP publique.  | 
| [New-AzVirtualNetworkGatewayIpConfig](/powershell/module/az.network/new-azvirtualnetworkgatewayipconfig) | Crée une configuration IP pour une passerelle de réseau virtuel. |
| [New-AzVirtualNetworkGateway](/powershell/module/az.network/new-azvirtualnetworkgateway) | Crée une passerelle de réseau virtuel. |

---


## <a name="connect-the-gateways"></a>Connecter les passerelles
Dans cette étape, créez une connexion bidirectionnelle entre les deux passerelles des deux réseaux virtuels. 


# <a name="portal"></a>[Portail](#tab/azure-portal)

Connectez les deux passerelles à l’aide du portail Azure. 


1. Dans le [portail Azure](https://portal.azure.com), sélectionnez **Créer une ressource**.
1. Saisissez `connection` dans la zone de recherche et appuyez sur Entrée pour lancer la recherche. Ceci vous amène à la ressource **Connexion**, publiée par Microsoft.
1. Sélectionnez **Créer** pour créer votre connexion. 
1. Dans la page **De base**, sélectionnez les valeurs suivantes, puis sélectionnez **OK**. 
    1. Sélectionnez `VNet-to-VNet` pour le **Type de connexion**. 
    1. Sélectionnez votre abonnement dans la liste déroulante. 
    1. Sélectionnez le groupe de ressources de l’instance managée SQL dans le menu déroulant. 
    1. Sélectionnez l’emplacement de votre instance gérée principale dans la liste déroulante. 
1. Dans la page **Paramètres**, sélectionnez ou saisissez les valeurs suivantes, puis sélectionnez **OK** :
    1. Choisissez la passerelle de réseau principal de la **Passerelle du premier réseau virtuel**, par exemple `primaryGateway`.  
    1. Choisissez la passerelle de réseau principal de la **Passerelle du deuxième réseau virtuel**, par exemple `secondaryGateway`. 
    1. Cochez la case en regard de **Établir une connectivité bidirectionnelle**. 
    1. Laissez le nom de la connexion principale par défaut ou renommez-la en choisissant une valeur de votre choix. 
    1. Fournissez une **Clé partagée (PSK)** à la connexion, par exemple `mi1m2psk`. 
    1. Sélectionnez **OK** pour enregistrer vos paramètres. 

    ![Créer une connexion de passerelle](./media/failover-group-add-instance-tutorial/create-gateway-connection.png)

    

1. Sous l’onglet **Vérifier + créer**, passez en revue les paramètres de votre connexion bidirectionnelle, puis sélectionnez **OK** pour créer votre connexion. 


# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Connectez les deux passerelles à l’aide de PowerShell. 

   ```powershell-interactive
   # Connect the primary to secondary gateway
   Write-host "Connecting the primary gateway to secondary gateway..."
   New-AzVirtualNetworkGatewayConnection -Name $primaryGWConnection -ResourceGroupName $resourceGroupName `
       -VirtualNetworkGateway1 $primaryGateway -VirtualNetworkGateway2 $secondaryGateway -Location $location `
       -ConnectionType Vnet2Vnet -SharedKey $vpnSharedKey -EnableBgp $true
   $primaryGWConnection
   
   # Connect the secondary to primary gateway
   Write-host "Connecting the secondary gateway to primary gateway..."
   
   New-AzVirtualNetworkGatewayConnection -Name $secondaryGWConnection -ResourceGroupName $resourceGroupName `
       -VirtualNetworkGateway1 $secondaryGateway -VirtualNetworkGateway2 $primaryGateway -Location $drLocation `
       -ConnectionType Vnet2Vnet -SharedKey $vpnSharedKey -EnableBgp $true
   $secondaryGWConnection
   ```

Cette partie du tutoriel utilise la cmdlet PowerShell suivante :

| Commande | Notes |
|---|---|
| [New-AzVirtualNetworkGatewayConnection](/powershell/module/az.network/new-azvirtualnetworkgatewayconnection) | Crée une connexion entre les deux passerelles de réseau virtuel.   |

---


## <a name="create-a-failover-group"></a>Créer un groupe de basculement
Au cours de cette étape, vous allez créer le groupe de basculement et y ajouter des instances managées. 


# <a name="portal"></a>[Portail](#tab/azure-portal)
Créez le groupe de basculement à l’aide du portail Azure. 


1. Dans le menu de gauche du **Portail Azure**, sélectionnez [Azure SQL](https://portal.azure.com). Si **Azure SQL** ne figure pas dans la liste, sélectionnez **Tous les services**, puis tapez `Azure SQL` dans la zone de recherche. (Facultatif) Sélectionnez l’étoile en regard d’**Azure SQL** pour l’ajouter aux favoris et l’ajouter en tant qu’élément dans le volet de navigation de gauche. 
1. Sélectionnez l’instance managée principale que vous avez créée dans le cadre de la première section, par exemple `sql-mi-primary`. 
1. Sous **Paramètres**, accédez à **Groupes de basculement d’instance**, puis choisissez **Ajouter un groupe** pour ouvrir la page **Groupe de basculement d’instance**. 

   ![Ajouter un groupe de basculement](./media/failover-group-add-instance-tutorial/add-failover-group.png)

1. Dans la page **Groupe de basculement d’instances**, tapez le nom de votre groupe de basculement, par exemple `failovergrouptutorial`. Choisissez ensuite l’instance managée secondaire, par exemple `sql-mi-secondary`, dans la liste déroulante. Sélectionnez **Créer** pour créer votre groupe de basculement. 

   ![Créer un groupe de basculement](./media/failover-group-add-instance-tutorial/create-failover-group.png)

1. Une fois le déploiement du groupe de basculement terminé, vous serez redirigé vers la page **Groupe de basculement**. 


# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Créez le groupe de basculement à l’aide de PowerShell. 

   ```powershell-interactive
   Write-host "Creating the failover group..."
   $failoverGroup = New-AzSqlDatabaseInstanceFailoverGroup -Name $failoverGroupName `
        -Location $location -ResourceGroupName $resourceGroupName -PrimaryManagedInstanceName $primaryInstance `
        -PartnerRegion $drLocation -PartnerManagedInstanceName $secondaryInstance `
        -FailoverPolicy Automatic -GracePeriodWithDataLossHours 1
   $failoverGroup
   ```

Cette partie du tutoriel utilise la cmdlet PowerShell suivante :

| Commande | Notes |
|---|---|
| [New-AzSqlDatabaseInstanceFailoverGroup](/powershell/module/az.sql/new-azsqldatabaseinstancefailovergroup)| Crée un nouveau groupe de basculement Azure SQL Managed Instance.  |


---


## <a name="test-failover"></a>Test de basculement
Dans cette étape, vous allez faire basculer votre groupe de basculement sur le serveur secondaire, puis effectuer une restauration automatique en utilisant le portail Azure. 


# <a name="portal"></a>[Portail](#tab/azure-portal)
Testez le basculement en utilisant le portail Azure. 


1. Accédez à votre instance managée _secondaire_ dans le [portail Azure](https://portal.azure.com) et sélectionnez **Groupes de basculement d’instance** sous les paramètres. 
1. Vérifiez quelle instance managée est la principale et laquelle est la secondaire. 
1. Sélectionnez **Basculement**, puis cliquez sur **Oui** dans l’avertissement concernant les sessions TDS sur le point d’être déconnectées. 

   ![Basculer le groupe de basculement](./media/failover-group-add-instance-tutorial/failover-mi-failover-group.png)

1. Vérifiez quelle instance managée est la principale et laquelle est la secondaire. Si le basculement a réussi, les deux instances doivent avoir échangé leur rôle. 

   ![Les instances managées ont échangé leurs rôles après le basculement](./media/failover-group-add-instance-tutorial/mi-switched-after-failover.png)

1. Accédez à la nouvelle instance managée _secondaire_ et sélectionnez de nouveau **Basculement** pour rebasculer l’instance principale vers le rôle principal. 


# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
Testez le basculement en utilisant PowerShell. 

   ```powershell-interactive
    
   # Verify the current primary role
   Get-AzSqlDatabaseInstanceFailoverGroup -ResourceGroupName $resourceGroupName `
       -Location $location -Name $failoverGroupName
   
   # Fail over the primary managed instance to the secondary role
   Write-host "Failing primary over to the secondary location"
   Get-AzSqlDatabaseInstanceFailoverGroup -ResourceGroupName $resourceGroupName `
       -Location $drLocation -Name $failoverGroupName | Switch-AzSqlDatabaseInstanceFailoverGroup
   Write-host "Successfully failed failover group to secondary location"
   ```


Rétablir le groupe de basculement sur le serveur principal :

   ```powershell-interactive
   # Verify the current primary role
   Get-AzSqlDatabaseInstanceFailoverGroup -ResourceGroupName $resourceGroupName `
       -Location $drLocation -Name $failoverGroupName
   
   # Fail the primary managed instance back to the primary role
   Write-host "Failing primary back to primary role"
   Get-AzSqlDatabaseInstanceFailoverGroup -ResourceGroupName $resourceGroupName `
       -Location $location -Name $failoverGroupName | Switch-AzSqlDatabaseInstanceFailoverGroup
   Write-host "Successfully failed failover group to primary location"
   
   # Verify the current primary role
   Get-AzSqlDatabaseInstanceFailoverGroup -ResourceGroupName $resourceGroupName `
       -Location $location -Name $failoverGroupName
   ```

Cette partie du tutoriel utilise les cmdlets PowerShell suivantes :

| Commande | Notes |
|---|---|
| [Get-AzSqlDatabaseInstanceFailoverGroup](/powershell/module/az.sql/get-azsqldatabaseinstancefailovergroup) | Obtient ou répertorie des groupes de basculement Managed Instance SQL.| 
| [Switch-AzSqlDatabaseInstanceFailoverGroup](/powershell/module/az.sql/switch-azsqldatabaseinstancefailovergroup) | Exécute un basculement d’un groupe de basculement Managed Instance SQL. | 

---



## <a name="clean-up-resources"></a>Nettoyer les ressources
Nettoyez les ressources en supprimant d’abord les instances managées, puis le cluster virtuel, puis les ressources restantes et enfin le groupe de ressources. 

# <a name="portal"></a>[Portail](#tab/azure-portal)
1. Accédez à votre groupe de ressources sur le [portail Azure](https://portal.azure.com). 
1. Sélectionnez la ou les instances gérées, puis choisissez **Supprimer**. Saisissez `yes` dans la zone de texte pour confirmer que vous souhaitez supprimer la ressource, puis sélectionnez **Supprimer**. Ce processus peut prendre un certain temps en arrière-plan et tant qu’il n’est pas terminé, vous ne pourrez pas supprimer le *cluster virtuel* ou d’autres ressources dépendantes. Supervisez la suppression dans l’onglet **Activité** pour confirmer que votre instance managée a été supprimée. 
1. Une fois l’instance managée supprimée, supprimez le *cluster virtuel* en le sélectionnant dans votre groupe de ressources, puis en choisissant **Supprimer**. Saisissez `yes` dans la zone de texte pour confirmer que vous souhaitez supprimer la ressource, puis sélectionnez **Supprimer**. 
1. Supprimez toutes les ressources restantes. Saisissez `yes` dans la zone de texte pour confirmer que vous souhaitez supprimer la ressource, puis sélectionnez **Supprimer**. 
1. Supprimez le groupe de ressources en sélectionnant **Supprimer le groupe de ressources**, saisissez le nom du groupe de ressources, `myResourceGroup`, puis sélectionnez **Supprimer**. 

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Vous devez supprimer le groupe de ressources à deux reprises. La première suppression du groupe de ressources entraîne la suppression des instances managées et des clusters virtuels, mais échouera avec le message d’erreur `Remove-AzResourceGroup : Long running operation failed with status 'Conflict'`. Exécutez la commande Remove-AzResourceGroup une deuxième fois pour supprimer toutes les ressources résiduelles, ainsi que le groupe de ressources.

```powershell-interactive
Remove-AzResourceGroup -ResourceGroupName $resourceGroupName
Write-host "Removing SQL Managed Instance and virtual cluster..."
Remove-AzResourceGroup -ResourceGroupName $resourceGroupName
Write-host "Removing residual resources and resource group..."
```

Cette partie du tutoriel utilise la cmdlet PowerShell suivante :

| Commande | Notes |
|---|---|
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Supprime un groupe de ressources. |

---

## <a name="full-script"></a>Script complet

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)
[!code-powershell-interactive[main](../../../powershell_scripts/sql-database/failover-groups/add-managed-instance-to-failover-group-az-ps.ps1 "Add SQL Managed Instance to a failover group")]

Ce script utilise les commandes suivantes. Chaque commande du tableau renvoie à une documentation spécifique.

| Commande | Notes |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Crée un groupe de ressources Azure.  |
| [New-AzVirtualNetwork](/powershell/module/az.network/new-azvirtualnetwork) | Créer un réseau virtuel.  |
| [Add-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/add-azvirtualnetworksubnetconfig) | Ajoute une configuration de sous-réseau à un réseau virtuel. | 
| [Get-AzVirtualNetwork](/powershell/module/az.network/get-azvirtualnetwork) | Obtient un réseau virtuel dans un groupe de ressources. | 
| [Get-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/get-azvirtualnetworksubnetconfig) | Obtient un sous-réseau dans un réseau virtuel. | 
| [New-AzNetworkSecurityGroup](/powershell/module/az.network/new-aznetworksecuritygroup) | Crée un groupe de sécurité réseau. | 
| [New-AzRouteTable](/powershell/module/az.network/new-azroutetable) | Crée une table de routage. |
| [Set-AzVirtualNetworkSubnetConfig](/powershell/module/az.network/set-azvirtualnetworksubnetconfig) | Met à jour une configuration de sous-réseau pour un réseau virtuel.  |
| [Set-AzVirtualNetwork](/powershell/module/az.network/set-azvirtualnetwork) | Met à jour un réseau virtuel.  |
| [Get-AzNetworkSecurityGroup](/powershell/module/az.network/get-aznetworksecuritygroup) | Obtient un groupe de sécurité réseau. |
| [Add-AzNetworkSecurityRuleConfig](/powershell/module/az.network/add-aznetworksecurityruleconfig)| Ajoute une configuration de règle de sécurité réseau à un groupe de sécurité réseau. |
| [Set-AzNetworkSecurityGroup](/powershell/module/az.network/set-aznetworksecuritygroup) | Met à jour un groupe de sécurité réseau.  | 
| [Add-AzRouteConfig](/powershell/module/az.network/add-azrouteconfig) | Ajoute un itinéraire à une table de routage. |
| [Set-AzRouteTable](/powershell/module/az.network/set-azroutetable) | Met à jour une table de routage.  |
| [New-AzSqlInstance](/powershell/module/az.sql/new-azsqlinstance) | Crée une instance managée.  |
| [Get-AzSqlInstance](/powershell/module/az.sql/get-azsqlinstance)| Retourne des informations sur Azure SQL Managed Instance. |
| [New-AzPublicIpAddress](/powershell/module/az.network/new-azpublicipaddress) | Crée une adresse IP publique.  | 
| [New-AzVirtualNetworkGatewayIpConfig](/powershell/module/az.network/new-azvirtualnetworkgatewayipconfig) | Crée une configuration IP pour une passerelle de réseau virtuel. |
| [New-AzVirtualNetworkGateway](/powershell/module/az.network/new-azvirtualnetworkgateway) | Crée une passerelle de réseau virtuel. |
| [New-AzVirtualNetworkGatewayConnection](/powershell/module/az.network/new-azvirtualnetworkgatewayconnection) | Crée une connexion entre les deux passerelles de réseau virtuel.   |
| [New-AzSqlDatabaseInstanceFailoverGroup](/powershell/module/az.sql/new-azsqldatabaseinstancefailovergroup)| Crée un groupe de basculement SQL Managed Instance.  |
| [Get-AzSqlDatabaseInstanceFailoverGroup](/powershell/module/az.sql/get-azsqldatabaseinstancefailovergroup) | Obtient ou répertorie des groupes de basculement Managed Instance SQL.| 
| [Switch-AzSqlDatabaseInstanceFailoverGroup](/powershell/module/az.sql/switch-azsqldatabaseinstancefailovergroup) | Exécute un basculement d’un groupe de basculement Managed Instance SQL. | 
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Supprime un groupe de ressources. | 

# <a name="portal"></a>[Portail](#tab/azure-portal) 

Aucun script n’est disponible pour le portail Azure.

---

## <a name="next-steps"></a>Étapes suivantes

Dans ce didacticiel, vous avez configuré un groupe de basculement entre deux instances managées. Vous avez appris à :

> [!div class="checklist"]
> - Créer une instance managée principale.
> - Créez une instance managée secondaire dans le cadre d’un [groupe de basculement](../database/auto-failover-group-overview.md). 
> - Tester le basculement.

Passez au démarrage rapide suivant pour savoir comment vous connecter à l’instance managée SQL et comment restaurer une base de données sur l’instance managée SQL : 

> [!div class="nextstepaction"]
> [Se connecter à l’instance managée SQL](connect-vm-instance-configure.md)
> [Restaurer une base de données sur l’instance managée SQL](restore-sample-database-quickstart.md)


