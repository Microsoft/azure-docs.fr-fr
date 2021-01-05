---
title: 'Tutoriel : Ajouter un pool élastique à un groupe de basculement'
description: Ajoutez un pool élastique Azure SQL Database à un groupe de basculement à l’aide du portail Azure, de PowerShell ou d’Azure CLI.
services: sql-database
ms.service: sql-database
ms.subservice: high-availability
ms.custom: seo-lt-2019 sqldbrb=1
ms.devlang: ''
ms.topic: tutorial
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 08/27/2019
ms.openlocfilehash: cdbc44158de2f24d7d33d68311979c3b8bdda85d
ms.sourcegitcommit: 1cf157f9a57850739adef72219e79d76ed89e264
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/13/2020
ms.locfileid: "94593975"
---
# <a name="tutorial-add-an-azure-sql-database-elastic-pool-to-a-failover-group"></a>Tutoriel : Ajouter un pool élastique Azure SQL Database à un groupe de basculement
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

Configurez un groupe de basculement pour un pool élastique Azure SQL Database et tester le basculement à l’aide du portail Azure.  Dans ce didacticiel, vous apprendrez à :

> [!div class="checklist"]
>
> - Créer une base de données unique.
> - Ajouter la base de données à un pool élastique.
> - Créer un [groupe de basculement](auto-failover-group-overview.md) pour deux pools élastiques entre deux serveurs.
> - Tester le basculement.

## <a name="prerequisites"></a>Prérequis

Pour suivre ce tutoriel, veillez à disposer des éléments suivants :

- Un abonnement Azure. [Créez un compte gratuit](https://azure.microsoft.com/free/) si vous n’en avez pas déjà un.

## <a name="1---create-a-single-database"></a>1 - Créer une base de données unique

[!INCLUDE [sql-database-create-single-database](../includes/sql-database-create-single-database.md)]

## <a name="2---add-the-database-to-an-elastic-pool"></a>2 - Ajouter la base de données à un pool élastique

Dans cette étape, vous allez créer un pool élastique et y ajouter votre base de données.

# <a name="portal"></a>[Portail](#tab/azure-portal)

Créez votre pool élastique avec le Portail Azure.

1. Dans le menu de gauche du portail Azure, sélectionnez **Azure SQL**. Si **Azure SQL** ne figure pas dans la liste, sélectionnez **Tous les services**, puis entrez « Azure SQL » dans la zone de recherche. (Facultatif) Sélectionnez l’étoile en regard d’**Azure SQL** pour l’ajouter aux favoris et l’ajouter en tant qu’élément dans le volet de navigation de gauche.
1. Sélectionnez **+Ajouter** pour ouvrir la page **Sélectionner l’option de déploiement SQL**. Vous pouvez afficher des informations supplémentaires sur les différentes bases de données en sélectionnant Afficher les détails sur la vignette Bases de données.
1. Sélectionnez **Pool élastique** dans la liste déroulante **Type de ressource** de la vignette **Bases de données SQL**. Sélectionnez **Créer** pour créer votre pool élastique.

    ![Sélectionner un pool élastique](./media/failover-group-add-elastic-pool-tutorial/select-azure-sql-elastic-pool.png)

1. Configurez votre pool élastique avec les valeurs suivantes :
   - **Name** : spécifiez un nom unique pour votre pool élastique, par exemple `myElasticPool`.
   - **Abonnement**: Sélectionnez votre abonnement dans la liste déroulante.
   - **Groupe de ressources** : sélectionnez `myResourceGroup` dans la liste déroulante, le groupe de ressources que vous avez créé dans la section 1.
   - **Serveur** : sélectionnez dans la liste déroulante le serveur créé dans la section 1.  

       ![Créer un serveur pour le pool élastique](./media/failover-group-add-elastic-pool-tutorial/use-existing-server-for-elastic-pool.png)

   - **Calcul + stockage** : sélectionnez **Configurer un pool élastique** pour configurer le calcul, le stockage, et ajouter votre base de données unique à votre pool élastique. Sous l’onglet **Paramètres du pool**, conservez la valeur par défaut Gen5, avec deux vCores et 32 Go.

1. Dans la page **Configurer**, sélectionnez l’onglet **Bases de données **, puis choisissez d’** Ajouter la base de données**. Choisissez la base de données créée dans la section 1, puis sélectionnez **Appliquer** pour l’ajouter à votre pool élastique. Sélectionnez **Appliquer** à nouveau pour appliquer vos paramètres de pool élastique et fermez la page **Configurer**.

    ![Ajouter une base de données à un pool élastique](./media/failover-group-add-elastic-pool-tutorial/add-database-to-elastic-pool.png)

1. Sélectionnez **Vérifier + créer** pour passer en revue les paramètres de votre pool élastique, puis sélectionnez **Créer** pour créer votre pool élastique.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Créez vos pools élastiques et votre serveur secondaire avec PowerShell.

   ```powershell-interactive
   # Set variables for your server and database
   # $subscriptionId = '<SubscriptionID>'
   # $resourceGroupName = "myResourceGroup-$(Get-Random)"
   # $location = "East US"
   # $adminLogin = "azureuser"
   # $password = "PWD27!"+(New-Guid).Guid
   # $serverName = "mysqlserver-$(Get-Random)"
   $poolName = "myElasticPool"
   $databaseName = "mySampleDatabase"
   $drLocation = "West US"
   $drServerName = "mysqlsecondary-$(Get-Random)"
   $failoverGroupName = "failovergrouptutorial-$(Get-Random)"

   # The ip address range that you want to allow to access your server
   # Leaving at 0.0.0.0 will prevent outside-of-azure connections
   # $startIp = "0.0.0.0"
   # $endIp = "0.0.0.0"

   # Show randomized variables
   Write-host "DR Server name is" $drServerName
   Write-host "Failover group name is" $failoverGroupName

   # Create primary Gen5 elastic 2 vCore pool
   Write-host "Creating elastic pool..."
   $elasticPool = New-AzSqlElasticPool -ResourceGroupName $resourceGroupName `
       -ServerName $serverName `
       -ElasticPoolName $poolName `
       -Edition "GeneralPurpose" `
       -vCore 2 `
       -ComputeGeneration Gen5
   $elasticPool

   # Add single db into elastic pool
   Write-host "Creating elastic pool..."
   $addDatabase = Set-AzSqlDatabase -ResourceGroupName $resourceGroupName `
       -ServerName $serverName `
       -DatabaseName $databaseName `
       -ElasticPoolName $poolName
   $addDatabase
   ```

Cette partie du tutoriel utilise les cmdlets PowerShell suivantes :

| Commande | Notes |
|---|---|
| [New-AzSqlElasticPool](/powershell/module/az.sql/new-azsqlelasticpool) | Crée un pool de bases de données élastique pour une base de données Azure SQL Database.|
| [Set-AzSqlDatabase](/powershell/module/az.sql/set-azsqldatabase) | Définit les propriétés d’une base de données, ou déplace une base de données existante dans un pool élastique. |

---

## <a name="3---create-the-failover-group"></a>3 - Créer le groupe de basculement

Dans de cette étape, vous allez créer un [groupe de basculement](auto-failover-group-overview.md) entre un serveur existant et un nouveau serveur dans une autre région. Ensuite, vous ajouterez le pool élastique au groupe de basculement.

# <a name="portal"></a>[Portail](#tab/azure-portal)

Créez votre groupe de basculement avec le Portail Azure.

1. Dans le menu de gauche du **Portail Azure**, sélectionnez [Azure SQL](https://portal.azure.com). Si **Azure SQL** ne figure pas dans la liste, sélectionnez **Tous les services**, puis tapez Azure SQL dans la zone de recherche. (Facultatif) Sélectionnez l’étoile en regard d’**Azure SQL** pour l’ajouter aux favoris et l’ajouter en tant qu’élément dans le volet de navigation de gauche.
1. Sélectionnez le pool élastique créé dans la section précédente, par exemple `myElasticPool`.
1. Dans le volet **Vue d’ensemble**, sélectionnez le nom du serveur sous **Nom du serveur** pour ouvrir les paramètres du serveur.
  
    ![Ouvrir le serveur pour le pool élastique](./media/failover-group-add-elastic-pool-tutorial/server-for-elastic-pool.png)

1. Sélectionnez **Groupes de basculement** dans le volet **Paramètres**, puis sélectionnez **Ajouter un groupe** pour créer un groupe de basculement.

    ![Ajouter un nouveau groupe de basculement](./media/failover-group-add-elastic-pool-tutorial/elastic-pool-failover-group.png)

1. Dans la page **Groupe de basculement**, entrez ou sélectionnez les valeurs suivantes, puis sélectionnez **Créer** :
    - **Nom du groupe de basculement** : Tapez un nom de groupe de basculement unique, par exemple `failovergrouptutorial`.
    - **Serveur secondaire** : Sélectionnez l’option permettant de *configurer les paramètres requis*, puis choisissez de **Créer un serveur**. Vous pouvez également choisir un serveur existant en tant que serveur secondaire. Après avoir entré les valeurs suivantes pour votre nouveau serveur secondaire, sélectionnez **Sélectionner**.
        - **Nom du serveur** : Tapez un nom unique pour le serveur secondaire, par exemple `mysqlsecondary`.
        - **Connexion administrateur au serveur** : Tapez `azureuser`.
        - **Mot de passe** : tapez un mot de passe complexe qui répond aux exigences de mot de passe.
        - **Emplacement** : choisissez un emplacement dans la liste déroulante, tel que `East US`. Cet emplacement ne peut pas être le même que celui de votre serveur principal.

       > [!NOTE]
       > La connexion au serveur et les paramètres de pare-feu doivent correspondre à ceux de votre serveur principal.

       ![Créer un serveur secondaire pour le groupe de basculement](./media/failover-group-add-elastic-pool-tutorial/create-secondary-failover-server.png)

1. Sélectionnez **Bases de données au sein du groupe**, puis sélectionnez le pool élastique créé dans la section 2. Un avertissement doit s’afficher, vous invitant à créer un pool élastique sur le serveur secondaire. Sélectionnez l’avertissement, puis cliquez sur **OK** pour créer le pool élastique sur le serveur secondaire. 

   ![Ajouter un pool élastique à un groupe de basculement](./media/failover-group-add-elastic-pool-tutorial/add-elastic-pool-to-failover-group.png)

1. Sélectionnez **Sélectionner** pour appliquer les paramètres de votre pool élastique au groupe de basculement, puis sélectionnez **Créer** pour créer votre groupe de basculement. L’ajout du pool élastique au groupe de basculement démarre automatiquement le processus de géoréplication.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Créez votre groupe de basculement avec PowerShell.

   ```powershell-interactive
   # Set variables for your server and database
   # $subscriptionId = '<SubscriptionID>'
   # $resourceGroupName = "myResourceGroup-$(Get-Random)"
   # $location = "East US"
   # $adminLogin = "azureuser"
   # $password = "PWD27!"+(New-Guid).Guid
   # $serverName = "mysqlserver-$(Get-Random)"
   # $poolName = "myElasticPool"
   # $databaseName = "mySampleDatabase"
   # $drLocation = "West US"
   # $drServerName = "mysqlsecondary-$(Get-Random)"
   $failoverGroupName = "failovergrouptutorial-$(Get-Random)"

   # Create a secondary server in the failover region
   Write-host "Creating a secondary server in the failover region..."
   New-AzSqlServer -ResourceGroupName $resourceGroupName `
      -ServerName $drServerName `
      -Location $drLocation `
      -SqlAdministratorCredentials $(New-Object -TypeName System.Management.Automation.PSCredential `
         -ArgumentList $adminlogin, $(ConvertTo-SecureString -String $password -AsPlainText -Force))
   Write-host "Secondary server =" $drServerName

   # Create a server firewall rule that allows access from the specified IP range
   Write-host "Configuring firewall for secondary server..."
   New-AzSqlServerFirewallRule -ResourceGroupName $resourceGroupName `
      -ServerName $drServerName `
      -FirewallRuleName "AllowedIPs" -StartIpAddress $startIp -EndIpAddress $endIp
   Write-host "Firewall configured"

   # Create secondary Gen5 elastic 2 vCore pool
   Write-host "Creating secondary elastic pool..."
   $elasticPool = New-AzSqlElasticPool -ResourceGroupName $resourceGroupName `
       -ServerName $drServerName `
       -ElasticPoolName $poolName `
       -Edition "GeneralPurpose" `
       -vCore 2 `
       -ComputeGeneration Gen5
   $elasticPool

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
   $failoverGroup
   ```

Cette partie du tutoriel utilise les cmdlets PowerShell suivantes :

| Commande | Notes |
|---|---|
| [New-AzSqlServer](/powershell/module/az.sql/new-azsqlserver) | Crée un serveur qui héberge des bases de données et des pools élastiques. |
| [New-AzSqlServerFirewallRule](/powershell/module/az.sql/new-azsqlserverfirewallrule) | Crée une règle de pare-feu pour un serveur. |
| [New-AzSqlElasticPool](/powershell/module/az.sql/new-azsqlelasticpool) | Crée un pool élastique pour une instance Azure SQL Database.|
| [New-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/new-azsqldatabasefailovergroup) | Crée un groupe de basculement. |
| [Add-AzSqlDatabaseToFailoverGroup](/powershell/module/az.sql/add-azsqldatabasetofailovergroup) | Ajoute une ou plusieurs bases de données Azure SQL à un groupe de basculement. |
| [Get-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/get-azsqldatabasefailovergroup) | Obtient ou dresse la liste des groupes de basculement Azure SQL Database. |

---

## <a name="4---test-failover"></a>4 - Tester le basculement

Dans cette étape, vous allez faire basculer votre groupe de basculement sur le serveur secondaire, puis effectuer une restauration automatique en utilisant le portail Azure.

# <a name="portal"></a>[Portail](#tab/azure-portal)

Testez le basculement de votre groupe de basculement à l’aide du portail Azure.

1. Dans le menu de gauche du **Portail Azure**, sélectionnez [Azure SQL](https://portal.azure.com). Si **Azure SQL** ne figure pas dans la liste, sélectionnez **Tous les services**, puis tapez Azure SQL dans la zone de recherche. (Facultatif) Sélectionnez l’étoile en regard d’**Azure SQL** pour l’ajouter aux favoris et l’ajouter en tant qu’élément dans le volet de navigation de gauche.
1. Sélectionnez le pool élastique créé dans la section précédente, par exemple `myElasticPool`.
1. Sélectionnez le nom du serveur sous **Nom du serveur** pour ouvrir les paramètres du serveur.

    ![Ouvrir le serveur pour le pool élastique](./media/failover-group-add-elastic-pool-tutorial/server-for-elastic-pool.png)

1. Sélectionnez **Groupes de basculement** dans le volet **Paramètres**, puis choisissez le groupe de basculement créé dans la section 2.
  
   ![Sélectionner le groupe de basculement à partir du portail](./media/failover-group-add-elastic-pool-tutorial/select-failover-group.png)

1. Vérifiez quel est le serveur principal et quel est le serveur secondaire.
1. Sélectionnez **Basculement** dans le volet des tâches pour faire basculer votre groupe de basculement contenant votre pool élastique.
1. Sélectionnez **Oui** en réponse à l’avertissement qui signale que les sessions TDS seront déconnectées.

   ![Basculer le groupe de basculement contenant votre base de données](./media/failover-group-add-elastic-pool-tutorial/failover-sql-db.png)

1. Vérifiez quel est le serveur principal et quel est le serveur secondaire. Si le basculement a réussi, les deux serveurs doivent avoir échangé leur rôle.
1. Sélectionnez **Basculement** à nouveau pour rétablir les paramètres d’origine du groupe de basculement.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Testez le basculement de votre groupe de basculement à l’aide de PowerShell.

   ```powershell-interactive
   # Set variables for your server and database
   # $subscriptionId = '<SubscriptionID>'
   # $resourceGroupName = "myResourceGroup-$(Get-Random)"
   # $location = "East US"
   # $adminLogin = "azureuser"
   # $password = "PWD27!"+(New-Guid).Guid
   # $serverName = "mysqlserver-$(Get-Random)"
   # $poolName = "myElasticPool"
   # $databaseName = "mySampleDatabase"
   # $drLocation = "West US"
   # $drServerName = "mysqlsecondary-$(Get-Random)"
   # $failoverGroupName = "failovergrouptutorial-$(Get-Random)"

   # Check role of secondary replica
   Write-host "Confirming the secondary server is secondary...."
   (Get-AzSqlDatabaseFailoverGroup `
      -FailoverGroupName $failoverGroupName `
      -ResourceGroupName $resourceGroupName `
      -ServerName $drServerName).ReplicationRole

   # Failover to secondary server
   Write-host "Failing over failover group to the secondary..."
   Switch-AzSqlDatabaseFailoverGroup `
      -ResourceGroupName $resourceGroupName `
      -ServerName $drServerName `
      -FailoverGroupName $failoverGroupName
   Write-host "Failover group failed over to" $drServerName
   ```

Faites basculer votre groupe de basculement sur le serveur secondaire, puis effectuez une restauration automatique avec PowerShell.

   ```powershell-interactive
   # Set variables for your server and database
   # $subscriptionId = '<SubscriptionID>'
   # $resourceGroupName = "myResourceGroup-$(Get-Random)"
   # $location = "East US"
   # $adminLogin = "azureuser"
   # $password = "PWD27!"+(New-Guid).Guid
   # $serverName = "mysqlserver-$(Get-Random)"
   # $poolName = "myElasticPool"
   # $databaseName = "mySampleDatabase"
   # $drLocation = "West US"
   # $drServerName = "mysqlsecondary-$(Get-Random)"
   # $failoverGroupName = "failovergrouptutorial-$(Get-Random)"

   # Check role of secondary replica
   Write-host "Confirming the secondary server is now primary"
   (Get-AzSqlDatabaseFailoverGroup `
      -FailoverGroupName $failoverGroupName `
      -ResourceGroupName $resourceGroupName `
      -ServerName $drServerName).ReplicationRole

   # Revert failover to primary server
   Write-host "Failing over failover group to the primary...."
   Switch-AzSqlDatabaseFailoverGroup `
      -ResourceGroupName $resourceGroupName `
      -ServerName $serverName `
      -FailoverGroupName $failoverGroupName
   Write-host "Failover group failed over to" $serverName
   ```

Cette partie du tutoriel utilise les cmdlets PowerShell suivantes :

| Commande | Notes |
|---|---|
| [Get-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/get-azsqldatabasefailovergroup) | Obtient ou dresse la liste des groupes de basculement Azure SQL Database. |
| [Switch-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/switch-azsqldatabasefailovergroup)| Exécute le basculement d’un groupe de basculement Azure SQL Database. |

---

## <a name="clean-up-resources"></a>Nettoyer les ressources

Nettoyez les ressources en supprimant le groupe de ressources.

# <a name="portal"></a>[Portail](#tab/azure-portal)

1. Accédez à votre groupe de ressources sur le [portail Azure](https://portal.azure.com).
1. Sélectionnez **Supprimer le groupe de ressources** pour supprimer toutes les ressources du groupe ainsi que le groupe de ressources lui-même.
1. Tapez le nom du groupe de ressources, `myResourceGroup`, dans la zone de texte, puis sélectionnez **Supprimer** pour supprimer le groupe de ressources.

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Nettoyez vos ressources avec PowerShell.

   ```powershell-interactive
   # Set variables for your server and database
   # $resourceGroupName = "myResourceGroup-$(Get-Random)"

   # Clean up resources by removing the resource group
   Write-host "Removing resource group..."
   Remove-AzResourceGroup -ResourceGroupName $resourceGroupName
   Write-host "Resource group removed =" $resourceGroupName
   ```

Cette partie du tutoriel utilise la cmdlet PowerShell suivante :

| Commande | Notes |
|---|---|
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Supprime un groupe de ressources. |

---

> [!IMPORTANT]
> Si vous souhaitez conserver le groupe de ressources tout en supprimant la base de données secondaire, retirez-la du groupe de basculement avant de la supprimer. La suppression d’une base de données secondaire avant son retrait du groupe de basculement peut entraîner un comportement imprévisible.

## <a name="full-script"></a>Script complet

# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

[!code-powershell-interactive[main](../../../powershell_scripts/sql-database/failover-groups/add-elastic-pool-to-failover-group-az-ps.ps1 "Add elastic pool to a failover group")]

Ce script utilise les commandes suivantes. Chaque commande du tableau renvoie à une documentation spécifique.

| Commande | Notes |
|---|---|
| [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup) | Crée un groupe de ressources dans lequel toutes les ressources sont stockées. |
| [New-AzSqlServer](/powershell/module/az.sql/new-azsqlserver) | Crée un serveur qui héberge des bases de données et des pools élastiques. |
| [New-AzSqlServerFirewallRule](/powershell/module/az.sql/new-azsqlserverfirewallrule) | Crée une règle de pare-feu pour un serveur. |
| [New-AzSqlDatabase](/powershell/module/az.sql/new-azsqldatabase) | Crée une base de données. |
| [New-AzSqlElasticPool](/powershell/module/az.sql/new-azsqlelasticpool) | Crée un pool de bases de données élastique pour une base de données Azure SQL Database.|
| [Set-AzSqlDatabase](/powershell/module/az.sql/set-azsqldatabase) | Définit les propriétés d’une base de données, ou déplace une base de données existante dans un pool élastique. |
| [New-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/new-azsqldatabasefailovergroup) | Crée un groupe de basculement. |
| [Get-AzSqlDatabase](/powershell/module/az.sql/get-azsqldatabase) | Obtient une ou plusieurs bases de données dans SQL Database. |
| [Add-AzSqlDatabaseToFailoverGroup](/powershell/module/az.sql/add-azsqldatabasetofailovergroup) | Ajoute une ou plusieurs bases de données Azure SQL à un groupe de basculement. |
| [Get-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/get-azsqldatabasefailovergroup) | Obtient ou dresse la liste des groupes de basculement Azure SQL Database. |
| [Switch-AzSqlDatabaseFailoverGroup](/powershell/module/az.sql/switch-azsqldatabasefailovergroup)| Exécute le basculement d’un groupe de basculement Azure SQL Database. |
| [Remove-AzResourceGroup](/powershell/module/az.resources/remove-azresourcegroup) | Supprime un groupe de ressources. |

# <a name="portal"></a>[Portail](#tab/azure-portal)

Aucun script n’est disponible pour le portail Azure.

---

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez ajouté un pool élastique Azure SQL Database à un groupe de basculement et testé le basculement. Vous avez appris à :

> [!div class="checklist"]
>
> - Créer une base de données unique.
> - Ajouter une base de données à un pool élastique.
> - Créer un [groupe de basculement](auto-failover-group-overview.md) pour deux pools élastiques entre deux serveurs.
> - Tester le basculement.

Passez au tutoriel suivant sur la migration à l’aide de DMS.

> [!div class="nextstepaction"]
> [Tutoriel : Migrer SQL Server vers une base de données mise en pool à l’aide de DMS](../../dms/tutorial-sql-server-to-azure-sql.md?toc=/azure/sql-database/toc.json)
