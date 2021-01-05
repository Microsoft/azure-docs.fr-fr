---
title: Configurer un groupe de basculement
titleSuffix: Azure SQL Database & SQL Managed Instance
description: Découvrez comment configurer un groupe de basculement automatique pour une base de données SQL Azure (unique et mise en pool) et une instance gérée SQL à l’aide du portail Azure, de l’interface de ligne de commande Azure et de PowerShell.
services: sql-database
ms.service: sql-db-mi
ms.subservice: high-availability
ms.custom: sqldbrb=2
ms.devlang: ''
ms.topic: how-to
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 08/14/2019
ms.openlocfilehash: 09bb7cb2344e3e708a64842916e6e483136da3bb
ms.sourcegitcommit: 1cf157f9a57850739adef72219e79d76ed89e264
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2020
ms.locfileid: "94594281"
---
# <a name="configure-a-failover-group-for-azure-sql-database"></a>Configurer un groupe de basculement pour Azure SQL Database
[!INCLUDE[appliesto-sqldb-sqlmi](../includes/appliesto-sqldb-sqlmi.md)]

Cette rubrique explique comment configurer un [groupe de basculement automatique](auto-failover-group-overview.md) pour Azure SQL Database et Azure SQL Managed Instance.

## <a name="single-database"></a>Base de données unique

Créez le groupe de basculement et ajoutez-y une base de données unique à l’aide du portail Azure ou de PowerShell.

### <a name="prerequisites"></a>Prérequis

Prenez en compte les prérequis suivants :

- La connexion au serveur et les paramètres de pare-feu pour le serveur secondaire doivent correspondre à ceux de votre serveur principal.

### <a name="create-failover-group"></a>Créer un groupe de basculement

# <a name="portal"></a>[Portail](#tab/azure-portal)

Créez votre groupe de basculement et ajoutez-y votre base de données en utilisant le portail Azure.

1. Dans le menu de gauche du **Portail Azure**, sélectionnez [Azure SQL](https://portal.azure.com). Si **Azure SQL** ne figure pas dans la liste, sélectionnez **Tous les services**, puis tapez Azure SQL dans la zone de recherche. (Facultatif) Sélectionnez l’étoile en regard d’**Azure SQL** pour l’ajouter aux favoris et l’ajouter en tant qu’élément dans le volet de navigation de gauche.
1. Sélectionnez la base de données que vous souhaitez ajouter au groupe de basculement.
1. Sélectionnez le nom du serveur sous **Nom du serveur** pour ouvrir les paramètres du serveur.

   ![Ouvrir un serveur pour une base de donnée unique](./media/auto-failover-group-configure/open-sql-db-server.png)

1. Sélectionnez **Groupes de basculement** dans le volet **Paramètres**, puis sélectionnez **Ajouter un groupe** pour créer un groupe de basculement.

   ![Ajouter un nouveau groupe de basculement](./media/auto-failover-group-configure/sqldb-add-new-failover-group.png)

1. Dans la page **Groupe de basculement**, entrez ou sélectionnez les valeurs requises, puis sélectionnez **Créer**.

   - **Base de données dans le groupe** : Choisissez la base de données que vous souhaitez ajouter à votre groupe de basculement. L’ajout de la base de données au groupe de basculement démarre automatiquement le processus de géoréplication.

   ![Ajouter SQL Database à un groupe de basculement](./media/auto-failover-group-configure/add-sqldb-to-failover-group.png)

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Créez votre groupe de basculement et ajoutez-y votre base de données à l’aide de PowerShell.

   ```powershell-interactive
   $subscriptionId = "<SubscriptionID>"
   $resourceGroupName = "<Resource-Group-Name>"
   $location = "<Region>"
   $adminLogin = "<Admin-Login>"
   $password = "<Complex-Password>"
   $serverName = "<Primary-Server-Name>"
   $databaseName = "<Database-Name>"
   $drLocation = "<DR-Region>"
   $drServerName = "<Secondary-Server-Name>"
   $failoverGroupName = "<Failover-Group-Name>"

   # Create a secondary server in the failover region
   Write-host "Creating a secondary server in the failover region..."
   $drServer = New-AzSqlServer -ResourceGroupName $resourceGroupName `
      -ServerName $drServerName `
      -Location $drLocation `
      -SqlAdministratorCredentials $(New-Object -TypeName System.Management.Automation.PSCredential `
         -ArgumentList $adminlogin, $(ConvertTo-SecureString -String $password -AsPlainText -Force))
   $drServer

   # Create a failover group between the servers
   $failovergroup = Write-host "Creating a failover group between the primary and secondary server..."
   New-AzSqlDatabaseFailoverGroup `
      –ResourceGroupName $resourceGroupName `
      -ServerName $serverName `
      -PartnerServerName $drServerName  `
      –FailoverGroupName $failoverGroupName `
      –FailoverPolicy Automatic `
      -GracePeriodWithDataLossHours 2
   $failovergroup

   # Add the database to the failover group
   Write-host "Adding the database to the failover group..."
   Get-AzSqlDatabase `
      -ResourceGroupName $resourceGroupName `
      -ServerName $serverName `
      -DatabaseName $databaseName | `
   Add-AzSqlDatabaseToFailoverGroup `
      -ResourceGroupName $resourceGroupName `
      -ServerName $serverName `
      -FailoverGroupName $failoverGroupName
   Write-host "Successfully added the database to the failover group..."
   ```

---

### <a name="test-failover"></a>Test de basculement

Testez le basculement de votre groupe de basculement à l’aide du portail Azure ou de PowerShell.

# <a name="portal"></a>[Portail](#tab/azure-portal)

Testez le basculement de votre groupe de basculement à l’aide du portail Azure.

1. Dans le menu de gauche du **Portail Azure**, sélectionnez [Azure SQL](https://portal.azure.com). Si **Azure SQL** ne figure pas dans la liste, sélectionnez **Tous les services**, puis saisissez « Azure SQL » dans la zone de recherche. (Facultatif) Sélectionnez l’étoile en regard d’**Azure SQL** pour l’ajouter aux favoris et l’ajouter en tant qu’élément dans le volet de navigation de gauche.
1. Sélectionnez la base de données que vous souhaitez ajouter au groupe de basculement.

   ![Ouvrir un serveur pour une base de donnée unique](./media/auto-failover-group-configure/open-sql-db-server.png)

1. Sélectionnez **Groupes de basculement** dans le volet **Paramètres**, puis choisissez le groupe de basculement que vous venez de créer.
  
   ![Sélectionner le groupe de basculement à partir du portail](./media/auto-failover-group-configure/select-failover-group.png)

1. Vérifiez quel est le serveur principal et quel est le serveur secondaire.
1. Sélectionnez **Basculement** dans le volet des tâches pour faire basculer le groupe de basculement contenant votre base de données.
1. Sélectionnez **Oui** en réponse à l’avertissement qui signale que les sessions TDS seront déconnectées.

   ![Basculer le groupe de basculement contenant votre base de données](./media/auto-failover-group-configure/failover-sql-db.png)

1. Vérifiez quel est maintenant le serveur principal et quel est le serveur secondaire. Si le basculement a réussi, les deux serveurs doivent avoir échangé leur rôle.
1. Resélectionnez **Basculer** pour rétablir les serveurs dans leur rôle d’origine.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Testez le basculement de votre groupe de basculement à l’aide de PowerShell.  

Vérifiez le rôle du réplica secondaire :

   ```powershell-interactive
   # Set variables
   $resourceGroupName = "<Resource-Group-Name>"
   $serverName = "<Primary-Server-Name>"
   $failoverGroupName = "<Failover-Group-Name>"

   # Check role of secondary replica
   Write-host "Confirming the secondary replica is secondary...."
   (Get-AzSqlDatabaseFailoverGroup `
      -FailoverGroupName $failoverGroupName `
      -ResourceGroupName $resourceGroupName `
      -ServerName $drServerName).ReplicationRole
   ```

Basculez vers le serveur secondaire :

   ```powershell-interactive
   # Set variables
   $resourceGroupName = "<Resource-Group-Name>"
   $serverName = "<Primary-Server-Name>"
   $failoverGroupName = "<Failover-Group-Name>"

   # Failover to secondary server
   Write-host "Failing over failover group to the secondary..."
   Switch-AzSqlDatabaseFailoverGroup `
      -ResourceGroupName $resourceGroupName `
      -ServerName $drServerName `
      -FailoverGroupName $failoverGroupName
   Write-host "Failed failover group to successfully to" $drServerName
   ```

Rétablir le groupe de basculement sur le serveur principal :

   ```powershell-interactive
   # Set variables
   $resourceGroupName = "<Resource-Group-Name>"
   $serverName = "<Primary-Server-Name>"
   $failoverGroupName = "<Failover-Group-Name>"

   # Revert failover to primary server
   Write-host "Failing over failover group to the primary...."
   Switch-AzSqlDatabaseFailoverGroup `
      -ResourceGroupName $resourceGroupName `
      -ServerName $serverName `
      -FailoverGroupName $failoverGroupName
   Write-host "Failed failover group successfully to back to" $serverName
   ```

---

> [!IMPORTANT]
> Si vous devez supprimer la base de données secondaire, retirez-la du groupe de basculement avant de la supprimer. La suppression d’une base de données secondaire avant son retrait du groupe de basculement peut entraîner un comportement imprévisible.

## <a name="elastic-pool"></a>Pool élastique

Créez le groupe de basculement et ajoutez-y un pool élastique en utilisant le portail Azure ou PowerShell.  

### <a name="prerequisites"></a>Prérequis

Prenez en compte les prérequis suivants :

- La connexion au serveur et les paramètres de pare-feu pour le serveur secondaire doivent correspondre à ceux de votre serveur principal.

### <a name="create-the-failover-group"></a>Créer le groupe de basculement

Créez le groupe de basculement pour votre pool élastique en utilisant le portail Azure ou PowerShell.

# <a name="portal"></a>[Portail](#tab/azure-portal)

Créez votre groupe de basculement et ajoutez-y votre pool élastique en utilisant le portail Azure.

1. Dans le menu de gauche du **Portail Azure**, sélectionnez [Azure SQL](https://portal.azure.com). Si **Azure SQL** ne figure pas dans la liste, sélectionnez **Tous les services**, puis saisissez « Azure SQL » dans la zone de recherche. (Facultatif) Sélectionnez l’étoile en regard d’**Azure SQL** pour l’ajouter aux favoris et l’ajouter en tant qu’élément dans le volet de navigation de gauche.
1. Sélectionnez le pool élastique que vous souhaitez ajouter au groupe de basculement.
1. Dans le volet **Vue d’ensemble**, sélectionnez le nom du serveur sous **Nom du serveur** pour ouvrir les paramètres du serveur.
  
   ![Ouvrir le serveur pour le pool élastique](./media/auto-failover-group-configure/server-for-elastic-pool.png)

1. Sélectionnez **Groupes de basculement** dans le volet **Paramètres**, puis sélectionnez **Ajouter un groupe** pour créer un groupe de basculement.

   ![Ajouter un nouveau groupe de basculement](./media/auto-failover-group-configure/sqldb-add-new-failover-group.png)

1. Dans la page **Groupe de basculement**, entrez ou sélectionnez les valeurs requises, puis sélectionnez **Créer**. Vous pouvez soit créer un serveur secondaire, soit sélectionner un serveur secondaire existant.

1. Sélectionnez **Base de données dans le groupe**, puis choisissez le pool élastique que vous souhaitez ajouter au groupe de basculement. Si aucun pool élastique n’existe sur le serveur secondaire, un avertissement s’affiche pour vous inviter à y en créer un. Sélectionnez l’avertissement, puis cliquez sur **OK** pour créer le pool élastique sur le serveur secondaire.

   ![Ajouter un pool élastique à un groupe de basculement](./media/auto-failover-group-configure/add-elastic-pool-to-failover-group.png)

1. Sélectionnez **Sélectionner** pour appliquer les paramètres de votre pool élastique au groupe de basculement, puis sélectionnez **Créer** pour créer votre groupe de basculement. L’ajout du pool élastique au groupe de basculement démarre automatiquement le processus de géoréplication.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Créez votre groupe de basculement et ajoutez-y votre pool élastique en utilisant PowerShell.

   ```powershell-interactive
   $subscriptionId = "<SubscriptionID>"
   $resourceGroupName = "<Resource-Group-Name>"
   $location = "<Region>"
   $adminLogin = "<Admin-Login>"
   $password = "<Complex-Password>"
   $serverName = "<Primary-Server-Name>"
   $databaseName = "<Database-Name>"
   $poolName = "myElasticPool"
   $drLocation = "<DR-Region>"
   $drServerName = "<Secondary-Server-Name>"
   $failoverGroupName = "<Failover-Group-Name>"

   # Create a failover group between the servers
   Write-host "Creating failover group..."
   New-AzSqlDatabaseFailoverGroup `
       –ResourceGroupName $resourceGroupName `
       -ServerName $serverName `
       -PartnerServerName $drServerName  `
       –FailoverGroupName $failoverGroupName `
       –FailoverPolicy Automatic `
       -GracePeriodWithDataLossHours 2
   Write-host "Failover group created successfully."

   # Add elastic pool to the failover group
   Write-host "Enumerating databases in elastic pool...."
   $FailoverGroup = Get-AzSqlDatabaseFailoverGroup `
                    -ResourceGroupName $resourceGroupName `
                    -ServerName $serverName `
                    -FailoverGroupName $failoverGroupName
   $databases = Get-AzSqlElasticPoolDatabase `
               -ResourceGroupName $resourceGroupName `
               -ServerName $serverName `
               -ElasticPoolName $poolName
   Write-host "Adding databases to failover group..."
   $failoverGroup = $failoverGroup | Add-AzSqlDatabaseToFailoverGroup `
                                     -Database $databases
   Write-host "Databases added to failover group successfully."
  ```

---

### <a name="test-failover"></a>Test de basculement

Testez le basculement de votre pool élastique à l’aide du portail Azure ou de PowerShell.

# <a name="portal"></a>[Portail](#tab/azure-portal)

Faites basculer votre groupe de basculement sur le serveur secondaire, puis effectuez une restauration automatique en utilisant le portail Azure.

1. Dans le menu de gauche du **Portail Azure**, sélectionnez [Azure SQL](https://portal.azure.com). Si **Azure SQL** ne figure pas dans la liste, sélectionnez **Tous les services**, puis saisissez « Azure SQL » dans la zone de recherche. (Facultatif) Sélectionnez l’étoile en regard d’**Azure SQL** pour l’ajouter aux favoris et l’ajouter en tant qu’élément dans le volet de navigation de gauche.
1. Sélectionnez le pool élastique que vous souhaitez ajouter au groupe de basculement.
1. Dans le volet **Vue d’ensemble**, sélectionnez le nom du serveur sous **Nom du serveur** pour ouvrir les paramètres du serveur.

   ![Ouvrir le serveur pour le pool élastique](./media/auto-failover-group-configure/server-for-elastic-pool.png)
1. Sélectionnez **Groupes de basculement** dans le volet **Paramètres**, puis choisissez le groupe de basculement créé dans la section 2.
  
   ![Sélectionner le groupe de basculement à partir du portail](./media/auto-failover-group-configure/select-failover-group.png)

1. Vérifiez quel est le serveur principal et quel est le serveur secondaire.
1. Sélectionnez **Basculement** dans le volet des tâches pour faire basculer votre groupe de basculement contenant votre pool élastique.
1. Sélectionnez **Oui** en réponse à l’avertissement qui signale que les sessions TDS seront déconnectées.

   ![Basculer le groupe de basculement contenant votre base de données](./media/auto-failover-group-configure/failover-sql-db.png)

1. Vérifiez quel est le serveur principal et quel est le serveur secondaire. Si le basculement a réussi, les deux serveurs doivent avoir échangé leur rôle.
1. Sélectionnez **Basculement** à nouveau pour rétablir les paramètres d’origine du groupe de basculement.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Testez le basculement de votre groupe de basculement à l’aide de PowerShell.

Vérifiez le rôle du réplica secondaire :

   ```powershell-interactive
   # Set variables
   $resourceGroupName = "<Resource-Group-Name>"
   $serverName = "<Primary-Server-Name>"
   $failoverGroupName = "<Failover-Group-Name>"

   # Check role of secondary replica
   Write-host "Confirming the secondary replica is secondary...."
   (Get-AzSqlDatabaseFailoverGroup `
      -FailoverGroupName $failoverGroupName `
      -ResourceGroupName $resourceGroupName `
      -ServerName $drServerName).ReplicationRole
   ```

Basculez vers le serveur secondaire :

   ```powershell-interactive
   # Set variables
   $resourceGroupName = "<Resource-Group-Name>"
   $serverName = "<Primary-Server-Name>"
   $failoverGroupName = "<Failover-Group-Name>"

   # Failover to secondary server
   Write-host "Failing over failover group to the secondary..."
   Switch-AzSqlDatabaseFailoverGroup `
      -ResourceGroupName $resourceGroupName `
      -ServerName $drServerName `
      -FailoverGroupName $failoverGroupName
   Write-host "Failed failover group to successfully to" $drServerName
   ```

---

> [!IMPORTANT]
> Si vous devez supprimer la base de données secondaire, retirez-la du groupe de basculement avant de la supprimer. La suppression d’une base de données secondaire avant son retrait du groupe de basculement peut entraîner un comportement imprévisible.

## <a name="sql-managed-instance"></a>Instance managée SQL

Créez un groupe de basculement entre deux instances gérées dans Azure SQL Managed Instance à l’aide du portail Azure ou de PowerShell.

Vous devrez soit configurer [ExpressRoute](../../expressroute/expressroute-howto-circuit-portal-resource-manager.md), soit créer une passerelle pour le réseau virtuel de chaque instance gérée SQL, connecter les deux passerelles, puis créer le groupe de basculement. 

Déployez les deux instances gérées dans des [régions appairées](../../best-practices-availability-paired-regions.md) pour des raisons de performances. Les instances managées résidant dans des régions appairées géographiquement offrent des performances sensiblement meilleures que celles résidant dans des régions non appairées. 

### <a name="prerequisites"></a>Prérequis

Prenez en compte les prérequis suivants :

- L’instance managée secondaire doit être vide.
- La plage de sous-réseau du réseau virtuel secondaire ne doit pas chevaucher la plage de sous-réseau du réseau virtuel principal.
- Le classement et le fuseau horaire de l’instance gérée secondaire doivent correspondre à ceux de l’instance gérée principale.
- Lors de la connexion des deux passerelles, la **clé partagée** doit être la même pour les deux connexions.

### <a name="create-primary-virtual-network-gateway"></a>Créer une passerelle de réseau virtuel principal

Si vous n’avez pas configuré [ExpressRoute](../../expressroute/expressroute-howto-circuit-portal-resource-manager.md), vous pouvez créer la passerelle de réseau virtuel principal avec le portail Azure ou PowerShell.

> [!NOTE]
> La référence SKU de la passerelle affecte les performances de débit. Cet article déploie une passerelle avec la référence SKU la plus basique (`HwGw1`). Pour obtenir un débit plus élevé, déployez une référence (SKU) supérieure (par exemple, `VpnGw3`). Pour toutes les options disponibles, consultez [Références (SKU) de passerelle](../../vpn-gateway/vpn-gateway-about-vpngateways.md#benchmark) 

# <a name="portal"></a>[Portail](#tab/azure-portal)

Créez la passerelle de réseau virtuel principal avec le portail Azure.

1. Dans le [portail Azure](https://portal.azure.com), accédez à votre groupe de ressources et sélectionnez la ressource **Réseau virtuel** pour votre instance managée principale.
1. Sélectionnez **Sous-réseaux** sous **Paramètres**, puis choisissez d’ajouter un nouveau **Sous-réseau de passerelle**. Laissez les valeurs par défaut.

   ![Ajouter une passerelle pour l’instance managée principale](./media/auto-failover-group-configure/add-subnet-gateway-primary-vnet.png)

1. Une fois la passerelle de sous-réseau créée, sélectionnez **Créer une ressource** dans le volet de navigation gauche, puis saisissez `Virtual network gateway` dans la zone de recherche. Sélectionnez la ressource de **Passerelle de réseau virtuel** publiée par **Microsoft**.

   ![Créer une passerelle de réseau virtuel](./media/auto-failover-group-configure/create-virtual-network-gateway.png)

1. Renseignez les champs requis pour configurer la passerelle de votre instance managée principale.

   Le tableau suivant montre les valeurs nécessaires pour la passerelle de l’instance managée principale :

    | **Champ** | Valeur |
    | --- | --- |
    | **Abonnement** |  L’abonnement dans lequel votre instance managée principale se trouve. |
    | **Nom** | Le nom de votre passerelle de réseau virtuel. |
    | **Région** | La région dans laquelle se trouve votre instance gérée principale. |
    | **Type de passerelle** | Sélectionnez **VPN**. |
    | **Type de VPN** | Sélectionnez **Route-based** |
    | **Référence (SKU)**| Laissez la valeur `VpnGw1` par défaut. |
    | **Lieu**| L’emplacement où se trouve votre instance gérée secondaire et votre réseau virtuel secondaire.   |
    | **Réseau virtuel**| Sélectionnez le réseau virtuel pour votre instance managée secondaire. |
    | **Adresse IP publique**| Sélectionnez **Créer nouveau**. |
    | **Nom de l’adresse IP publique**| Entrez un nom pour votre adresse IP. |
    | &nbsp; | &nbsp; |

1. Laissez les autres valeurs par défaut, puis sélectionnez **Vérifier + créer** pour passer en revue les paramètres de votre passerelle de réseau virtuel.

   ![Paramètres de la passerelle principale](./media/auto-failover-group-configure/settings-for-primary-gateway.png)

1. Sélectionnez **Créer** pour créer votre passerelle de réseau virtuel.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Créez la passerelle de réseau virtuel principal à l’aide de PowerShell.

   ```powershell-interactive
   $primaryResourceGroupName = "<Primary-Resource-Group>"
   $primaryVnetName = "<Primary-Virtual-Network-Name>"
   $primaryGWName = "<Primary-Gateway-Name>"
   $primaryGWPublicIPAddress = $primaryGWName + "-ip"
   $primaryGWIPConfig = $primaryGWName + "-ipc"
   $primaryGWAsn = 61000

   # Get the primary virtual network
   $vnet1 = Get-AzVirtualNetwork -Name $primaryVnetName -ResourceGroupName $primaryResourceGroupName
   $primaryLocation = $vnet1.Location

   # Create primary gateway
   Write-host "Creating primary gateway..."
   $subnet1 = Get-AzVirtualNetworkSubnetConfig -Name GatewaySubnet -VirtualNetwork $vnet1
   $gwpip1= New-AzPublicIpAddress -Name $primaryGWPublicIPAddress -ResourceGroupName $primaryResourceGroupName `
            -Location $primaryLocation -AllocationMethod Dynamic
   $gwipconfig1 = New-AzVirtualNetworkGatewayIpConfig -Name $primaryGWIPConfig `
            -SubnetId $subnet1.Id -PublicIpAddressId $gwpip1.Id

   $gw1 = New-AzVirtualNetworkGateway -Name $primaryGWName -ResourceGroupName $primaryResourceGroupName `
       -Location $primaryLocation -IpConfigurations $gwipconfig1 -GatewayType Vpn `
       -VpnType RouteBased -GatewaySku VpnGw1 -EnableBgp $true -Asn $primaryGWAsn
   $gw1
   ```

---

### <a name="create-secondary-virtual-network-gateway"></a>Créer la passerelle de réseau virtuel secondaire

Créez la passerelle de réseau virtuel secondaire à l’aide du portail Azure ou de PowerShell.

# <a name="portal"></a>[Portail](#tab/azure-portal)

Répétez les étapes de la section précédente pour créer le sous-réseau et la passerelle du réseau virtuel pour l’instance managée secondaire. Renseignez les champs requis pour configurer la passerelle de votre instance managée secondaire.

Le tableau suivant montre les valeurs nécessaires pour la passerelle de l’instance managée secondaire :

   | **Champ** | Valeur |
   | --- | --- |
   | **Abonnement** |  L’abonnement dans lequel votre instance managée secondaire se trouve. |
   | **Nom** | Le nom de votre passerelle de réseau virtuel, par exemple `secondary-mi-gateway`. |
   | **Région** | La région dans laquelle votre instance managée secondaire se trouve. |
   | **Type de passerelle** | Sélectionnez **VPN**. |
   | **Type de VPN** | Sélectionnez **Route-based** |
   | **Référence (SKU)**| Laissez la valeur `VpnGw1` par défaut. |
   | **Lieu**| L’emplacement où se trouve votre instance gérée secondaire et votre réseau virtuel secondaire.   |
   | **Réseau virtuel**| Sélectionnez le réseau virtuel créé dans la section 2, par exemple `vnet-sql-mi-secondary`. |
   | **Adresse IP publique**| Sélectionnez **Créer nouveau**. |
   | **Nom de l’adresse IP publique**| Entrez un nom pour votre adresse IP, par exemple `secondary-gateway-IP`. |
   | &nbsp; | &nbsp; |

   ![Paramètres de la passerelle secondaire](./media/auto-failover-group-configure/settings-for-secondary-gateway.png)

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Créez la passerelle de réseau virtuel secondaire à l’aide de PowerShell.

   ```powershell-interactive
   $secondaryResourceGroupName = "<Secondary-Resource-Group>"
   $secondaryVnetName = "<Secondary-Virtual-Network-Name>"
   $secondaryGWName = "<Secondary-Gateway-Name>"
   $secondaryGWPublicIPAddress = $secondaryGWName + "-IP"
   $secondaryGWIPConfig = $secondaryGWName + "-ipc"
   $secondaryGWAsn = 62000

   # Get the secondary virtual network
   $vnet2 = Get-AzVirtualNetwork -Name $secondaryVnetName -ResourceGroupName $secondaryResourceGroupName
   $secondaryLocation = $vnet2.Location

   # Create the secondary gateway
   Write-host "Creating secondary gateway..."
   $subnet2 = Get-AzVirtualNetworkSubnetConfig -Name GatewaySubnet -VirtualNetwork $vnet2
   $gwpip2= New-AzPublicIpAddress -Name $secondaryGWPublicIPAddress -ResourceGroupName $secondaryResourceGroupName `
            -Location $secondaryLocation -AllocationMethod Dynamic
   $gwipconfig2 = New-AzVirtualNetworkGatewayIpConfig -Name $secondaryGWIPConfig `
            -SubnetId $subnet2.Id -PublicIpAddressId $gwpip2.Id

   $gw2 = New-AzVirtualNetworkGateway -Name $secondaryGWName -ResourceGroupName $secondaryResourceGroupName `
       -Location $secondaryLocation -IpConfigurations $gwipconfig2 -GatewayType Vpn `
       -VpnType RouteBased -GatewaySku VpnGw1 -EnableBgp $true -Asn $secondaryGWAsn

   $gw2
   ```

---

### <a name="connect-the-gateways"></a>Connecter les passerelles

Créez les connexions entre les deux passerelles à l’aide du portail Azure ou de PowerShell.

Deux connexions doivent être créées : la connexion entre la passerelle principale et la passerelle secondaire, puis la connexion entre la passerelle secondaire et la passerelle principale.

La clé partagée utilisée pour les deux connexions doit être la même pour chaque connexion.

# <a name="portal"></a>[Portail](#tab/azure-portal)

Créez les connexions entre les deux passerelles à l’aide du portail Azure.

1. Dans le [portail Azure](https://portal.azure.com), sélectionnez **Créer une ressource**.
1. Saisissez `connection` dans la zone de recherche et appuyez sur Entrée pour lancer la recherche. Ceci vous amène à la ressource **Connexion**, publiée par Microsoft.
1. Sélectionnez **Créer** pour créer votre connexion.
1. Sous l’onglet **De base**, sélectionnez les valeurs suivantes, puis sélectionnez **OK**.
    1. Sélectionnez `VNet-to-VNet` pour le **Type de connexion**.
    1. Sélectionnez votre abonnement dans la liste déroulante.
    1. Sélectionnez le groupe de ressources de votre instance managée dans le menu déroulant.
    1. Sélectionnez l’emplacement de votre instance gérée principale dans la liste déroulante.
1. Sous l’onglet **Paramètres**, sélectionnez ou saisissez les valeurs suivantes, puis sélectionnez **OK** :
    1. Choisissez la passerelle de réseau principal de la **Passerelle du premier réseau virtuel**, par exemple `Primary-Gateway`.  
    1. Choisissez la passerelle de réseau principal de la **Passerelle du deuxième réseau virtuel**, par exemple `Secondary-Gateway`.
    1. Cochez la case en regard de **Établir une connectivité bidirectionnelle**.
    1. Laissez le nom de la connexion principale par défaut ou renommez-la en choisissant une valeur de votre choix.
    1. Fournissez une **Clé partagée (PSK)** à la connexion, par exemple `mi1m2psk`.

   ![Créer une connexion de passerelle](./media/auto-failover-group-configure/create-gateway-connection.png)

1. Sous l’onglet **Résumé**, passez en revue les paramètres de votre connexion bidirectionnelle, puis sélectionnez **OK** pour créer votre connexion.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Créez les connexions entre les deux passerelles à l’aide de PowerShell.

   ```powershell-interactive
   $vpnSharedKey = "mi1mi2psk"
   $primaryResourceGroupName = "<Primary-Resource-Group>"
   $primaryGWConnection = "<Primary-connection-name>"
   $primaryLocation = "<Primary-Region>"
   $secondaryResourceGroupName = "<Secondary-Resource-Group>"
   $secondaryGWConnection = "<Secondary-connection-name>"
   $secondaryLocation = "<Secondary-Region>"
  
   # Connect the primary to secondary gateway
   Write-host "Connecting the primary gateway"
   New-AzVirtualNetworkGatewayConnection -Name $primaryGWConnection -ResourceGroupName $primaryResourceGroupName `
       -VirtualNetworkGateway1 $gw1 -VirtualNetworkGateway2 $gw2 -Location $primaryLocation `
       -ConnectionType Vnet2Vnet -SharedKey $vpnSharedKey -EnableBgp $true
   $primaryGWConnection

   # Connect the secondary to primary gateway
   Write-host "Connecting the secondary gateway"

   New-AzVirtualNetworkGatewayConnection -Name $secondaryGWConnection -ResourceGroupName $secondaryResourceGroupName `
       -VirtualNetworkGateway1 $gw2 -VirtualNetworkGateway2 $gw1 -Location $secondaryLocation `
       -ConnectionType Vnet2Vnet -SharedKey $vpnSharedKey -EnableBgp $true
   $secondaryGWConnection
   ```

---

### <a name="create-the-failover-group"></a>Créer le groupe de basculement

Créez le groupe de basculement pour vos instances gérées à l’aide du portail Azure ou de PowerShell.

# <a name="portal"></a>[Portail](#tab/azure-portal)

Créez le groupe de basculement pour vos instances gérées SQL à l’aide du portail Azure ou de PowerShell.

1. Dans le menu de gauche du **Portail Azure**, sélectionnez [Azure SQL](https://portal.azure.com). Si **Azure SQL** ne figure pas dans la liste, sélectionnez **Tous les services**, puis tapez Azure SQL dans la zone de recherche. (Facultatif) Sélectionnez l’étoile en regard d’**Azure SQL** pour l’ajouter aux favoris et l’ajouter en tant qu’élément dans le volet de navigation de gauche.
1. Sélectionnez l’instance managée principale que vous souhaitez ajouter au groupe de basculement.  
1. Sous **Paramètres**, accédez à **Groupes de basculement d’instance**, puis choisissez **Ajouter un groupe** pour ouvrir la page **Groupe de basculement d’instance**.

   ![Ajouter un groupe de basculement](./media/auto-failover-group-configure/add-failover-group.png)

1. Dans la page **Groupe de basculement d’instance**, tapez le nom de votre groupe de basculement, puis choisissez l’instance managée secondaire dans la liste déroulante. Sélectionnez **Créer** pour créer votre groupe de basculement.

   ![Créer un groupe de basculement](./media/auto-failover-group-configure/create-failover-group.png)

1. Une fois le déploiement du groupe de basculement terminé, vous serez redirigé vers la page **Groupe de basculement**.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Créez le groupe de basculement pour vos instances managées à l’aide de PowerShell.

   ```powershell-interactive
   $primaryResourceGroupName = "<Primary-Resource-Group>"
   $failoverGroupName = "<Failover-Group-Name>"
   $primaryLocation = "<Primary-Region>"
   $secondaryLocation = "<Secondary-Region>"
   $primaryManagedInstance = "<Primary-Managed-Instance-Name>"
   $secondaryManagedInstance = "<Secondary-Managed-Instance-Name>"

   # Create failover group
   Write-host "Creating the failover group..."
   $failoverGroup = New-AzSqlDatabaseInstanceFailoverGroup -Name $failoverGroupName `
        -Location $primaryLocation -ResourceGroupName $primaryResourceGroupName -PrimaryManagedInstanceName $primaryManagedInstance `
        -PartnerRegion $secondaryLocation -PartnerManagedInstanceName $secondaryManagedInstance `
        -FailoverPolicy Automatic -GracePeriodWithDataLossHours 1
   $failoverGroup
   ```

---

### <a name="test-failover"></a>Test de basculement

Testez le basculement de votre groupe de basculement à l’aide du portail Azure ou de PowerShell.

# <a name="portal"></a>[Portail](#tab/azure-portal)

Testez le basculement de votre groupe de basculement à l’aide du portail Azure.

1. Accédez à votre instance gérée _secondaire_ dans le [Portail Azure](https://portal.azure.com) et sélectionnez **Groupes de basculement d’instance** sous les paramètres.
1. Vérifiez quelle instance managée est la principale et laquelle est la secondaire.
1. Sélectionnez **Basculement**, puis cliquez sur **Oui** dans l’avertissement concernant les sessions TDS sur le point d’être déconnectées.

   ![Basculer le groupe de basculement](./media/auto-failover-group-configure/failover-mi-failover-group.png)

1. Vérifiez quelle instance est la principale et laquelle est la secondaire. Si le basculement a réussi, les deux instances doivent avoir échangé leur rôle.

   ![Les instances managées ont échangé leurs rôles après le basculement](./media/auto-failover-group-configure/mi-switched-after-failover.png)

1. Accédez à la nouvelle instance managée _secondaire_ et sélectionnez de nouveau **Basculement** pour rebasculer l’instance principale vers le rôle principal.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Testez le basculement de votre groupe de basculement à l’aide de PowerShell.

   ```powershell-interactive
   $primaryResourceGroupName = "<Primary-Resource-Group>"
   $secondaryResourceGroupName = "<Secondary-Resource-Group>"
   $failoverGroupName = "<Failover-Group-Name>"
   $primaryLocation = "<Primary-Region>"
   $secondaryLocation = "<Secondary-Region>"
   $primaryManagedInstance = "<Primary-Managed-Instance-Name>"
   $secondaryManagedInstance = "<Secondary-Managed-Instance-Name>"

   # Verify the current primary role
   Get-AzSqlDatabaseInstanceFailoverGroup -ResourceGroupName $primaryResourceGroupName `
       -Location $secondaryLocation -Name $failoverGroupName

   # Failover the primary managed instance to the secondary role
   Write-host "Failing primary over to the secondary location"
   Get-AzSqlDatabaseInstanceFailoverGroup -ResourceGroupName $secondaryResourceGroupName `
       -Location $secondaryLocation -Name $failoverGroupName | Switch-AzSqlDatabaseInstanceFailoverGroup
   Write-host "Successfully failed failover group to secondary location"

   # Verify the current primary role
   Get-AzSqlDatabaseInstanceFailoverGroup -ResourceGroupName $primaryResourceGroupName `
       -Location $secondaryLocation -Name $failoverGroupName

   # Fail primary managed instance back to primary role
   Write-host "Failing primary back to primary role"
   Get-AzSqlDatabaseInstanceFailoverGroup -ResourceGroupName $primaryResourceGroupName `
       -Location $primaryLocation -Name $failoverGroupName | Switch-AzSqlDatabaseInstanceFailoverGroup
   Write-host "Successfully failed failover group to primary location"

   # Verify the current primary role
   Get-AzSqlDatabaseInstanceFailoverGroup -ResourceGroupName $primaryResourceGroupName `
       -Location $secondaryLocation -Name $failoverGroupName
   ```

---

## <a name="use-private-link"></a>Utiliser Private Link

L’utilisation d’un lien privé vous permet d’associer un serveur logique à une adresse IP privée spécifique au sein du réseau virtuel et du sous-réseau. 

Pour utiliser un lien privé avec votre groupe de basculement, procédez comme suit :

1. Assurez-vous que vos serveurs principaux et secondaires se trouvent dans une [région couplée](../../best-practices-availability-paired-regions.md). 
1. Créez le réseau virtuel et le sous-réseau dans chaque région pour héberger des points de terminaison privés pour les serveurs principaux et secondaires de sorte qu’ils aient des espaces d’adressage IP qui ne se chevauchent pas. Par exemple, la plage d’adresses de réseau virtuel principale 10.0.0.0/16 et la plage d’adresses de réseau virtuel secondaire de 10.0.0.1/16 se chevauchent. Pour plus d’informations sur les plages d’adresses de réseaux virtuels, consultez le blog [Conception de réseaux virtuels Azure](https://devblogs.microsoft.com/premier-developer/understanding-cidr-notation-when-designing-azure-virtual-networks-and-subnets/).
1. Créez un [point de terminaison privé et une zone de DNS privé Azure pour le serveur principal](../../private-link/create-private-endpoint-portal.md#create-a-private-endpoint). 
1. Créez également un point de terminaison privé pour le serveur secondaire, mais cette fois, choisissez de réutiliser la même zone DNS privée que celle qui a été créée pour le serveur principal. 
1. Une fois le lien privé établi, vous pouvez créer le groupe de basculement en suivant les étapes décrites précédemment dans cet article. 


## <a name="locate-listener-endpoint"></a>Localiser le point de terminaison de l’écouteur

Une fois votre groupe de basculement configuré, mettez à jour la chaîne de connexion de votre application sur le point de terminaison de l’écouteur. Ainsi, votre application restera connectée à l’écouteur du groupe de basculement, plutôt qu’à la base de données primaire, au pool élastique ou à la base de données d’instance. De cette façon, vous n’avez pas besoin de mettre à jour manuellement la chaîne de connexion chaque fois que votre entité de base de données fait l’objet d’un basculement et le trafic est acheminé vers l’entité qui fait office d’entité principale.

Le point de terminaison de l’écouteur se présente sous la forme, `fog-name.database.windows.net` et est visible dans le Portail Azure, quand vous affichez le groupe de basculement :

![Chaîne de connexion du groupe de basculement](./media/auto-failover-group-configure/find-failover-group-connection-string.png)

## <a name="remarks"></a>Notes

- La suppression d’un groupe de basculement pour une base de données unique ou mise en pool n’interrompt pas la réplication et n’entraîne pas la suppression de la base de données répliquée. Vous devrez arrêter manuellement la géoréplication et supprimer la base de données du serveur secondaire si vous souhaitez rajouter une base de données unique ou mise en pool à un groupe de basculement après sa suppression. Si vous n’effectuez pas l’une ou l’autre de ces opérations, vous risquez d’obtenir une erreur telle que `The operation cannot be performed due to multiple errors` lorsque vous tentez d’ajouter la base de données au groupe de basculement.

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir des instructions détaillées sur la configuration d’un groupe de basculement, consultez les tutoriels suivants :

- [Ajouter une base de données unique à un groupe de basculement](failover-group-add-single-database-tutorial.md)
- [Ajouter un pool élastique à un groupe de basculement](failover-group-add-elastic-pool-tutorial.md)
- [Ajouter une instance gérée à un groupe de basculement](../managed-instance/failover-group-add-instance-tutorial.md)

Pour obtenir une vue d’ensemble des options de haute disponibilité pour Azure SQL Database, consultez [Géoréplication](active-geo-replication-overview.md) et [Groupes de basculement automatique](auto-failover-group-overview.md).