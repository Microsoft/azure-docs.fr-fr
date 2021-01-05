---
title: Créer une base de données unique
description: Créez une base de données unique dans Azure SQL Database à l’aide du portail Azure, de PowerShell ou de l’interface Azure CLI.
services: sql-database
ms.service: sql-database
ms.subservice: single-database
ms.custom: contperf-fy21q1, devx-track-azurecli
ms.devlang: ''
ms.topic: quickstart
author: stevestein
ms.author: sstein
ms.reviewer: ''
ms.date: 09/03/2020
ms.openlocfilehash: 21ccbc83c80e1b659dd240973a609d624f270835
ms.sourcegitcommit: 3ea45bbda81be0a869274353e7f6a99e4b83afe2
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97028510"
---
# <a name="quickstart-create-an-azure-sql-database-single-database"></a>Démarrage rapide : Créer une base de données Azure SQL

Dans ce guide de démarrage rapide, vous créez une [base de données unique](single-database-overview.md) dans Azure SQL Database en utilisant le portail Azure, un script PowerShell ou un script d’interface de ligne de commande Azure. Vous allez ensuite interroger la base de données à l’aide de l’**éditeur de requête** dans le portail Azure.



## <a name="prerequisite"></a>Configuration requise

- Un abonnement Azure actif. Si vous n’en avez pas, [créez un compte gratuit](https://azure.microsoft.com/free/).

## <a name="create-a-single-database"></a>Créer une base de données unique

Ce guide de démarrage rapide crée une base de données unique dans la [couche de calcul serverless](serverless-tier-overview.md).

# <a name="portal"></a>[Portail](#tab/azure-portal)

Pour créer une base de données unique dans le portail Azure, ce guide de démarrage rapide démarre dans la page Azure SQL.

1. Accédez à la page [Sélectionner l’option de déploiement SQL](https://portal.azure.com/#create/Microsoft.AzureSQL).
1. Sous **Bases de données SQL**, laissez **Type de ressource** défini sur **Base de données unique**, puis sélectionnez **Créer**.

   ![Ajouter à Azure SQL](./media/single-database-create-quickstart/select-deployment.png)

1. Sous l’onglet **De base** du formulaire **Créer une base de données SQL**, sous **Détails du projet**, sélectionnez l’**Abonnement** Azure souhaité.
1. Pour **Groupe de ressources**, sélectionnez **Créer**, entrez *myResourceGroup*, puis sélectionnez **OK**.
1. Pour **Nom de la base de données**, entrez *mySampleDatabase*.
1. Pour **Serveur**, sélectionnez **Créer**, puis remplissez le formulaire **Nouveau serveur** avec les valeurs suivantes :
   - **Nom du serveur** : entrez *mysqlserver* et ajoutez quelques caractères pour l’unicité. Nous ne pouvons pas fournir un nom de serveur exact à utiliser, car les noms de serveur doivent être globalement uniques pour tous les serveurs dans Azure, et non pas seulement dans un abonnement. Par conséquent, entrez un nom tel que mysqlserver12345 et le portail vous indique s’il est disponible ou non.
   - **Connexion administrateur au serveur** : entrez *azureuser*.
   - **Mot de passe** : entrez un mot de passe qui répond aux exigences, puis réentrez-le dans le champ **Confirmer le mot de passe**.
   - **Emplacement** : sélectionnez un emplacement dans la liste déroulante.

   Sélectionnez **OK**.

1. Laissez **Vous souhaitez utiliser un pool élastique SQL ?** avec la valeur **Non**.
1. Sous **Calcul + stockage**, sélectionnez **Configurer la base de données**.
1. Comme ce guide de démarrage rapide utilise une base de données serverless, sélectionnez **Serverless**, puis **Appliquer**. 

      ![configurer une base de données serverless](./media/single-database-create-quickstart/configure-database.png)

1. Sélectionnez **Suivant : Réseau** en bas de la page.

   ![Nouvelle base de données SQL - Onglet des informations de base](./media/single-database-create-quickstart/new-sql-database-basics.png)

1. Sous l’onglet **Réseau**, pour **Méthode de connectivité**, sélectionnez **Point de terminaison public**.
1. Pour **Règles de pare-feu**, affectez la valeur **Oui** à **Ajouter l’adresse IP actuelle du client**. Laissez **Autoriser les services et les ressources Azure à accéder à ce serveur** avec la valeur **Non**.
1. Sélectionnez **Suivant : Paramètres supplémentaires** en bas de la page.

   ![Onglet Réseau](./media/single-database-create-quickstart/networking.png)
  

1. Sous l’onglet **Paramètres supplémentaires**, dans la section **Source de données**, pour **Utiliser des données existantes**, sélectionnez **Exemple**. Vous créez ainsi un exemple de base de données AdventureWorksLT, de sorte qu’il existe des tables et des données à interroger et expérimenter, par opposition à une base de données vide.
1. Au bas de la page, sélectionnez **Vérifier + créer** :

   ![Onglet Paramètres supplémentaires](./media/single-database-create-quickstart/additional-settings.png)

1. Dans la page **Vérifier + créer**, après vérification, sélectionnez **Créer**.

# <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

## <a name="launch-azure-cloud-shell"></a>Lancement d’Azure Cloud Shell

Azure Cloud Shell est un interpréteur de commandes interactif et gratuit que vous pouvez utiliser pour exécuter les étapes de cet article. Il contient des outils Azure courants préinstallés et configurés pour être utilisés avec votre compte. 

Pour ouvrir Cloud Shell, sélectionnez simplement **Essayer** en haut à droite d’un bloc de code. Vous pouvez également lancer Cloud Shell dans un onglet distinct du navigateur en accédant à [https://shell.azure.com](https://shell.azure.com). Sélectionnez **Copier** pour copier les blocs de code, collez-les dans Cloud Shell, puis appuyez sur **Entrée** pour les exécuter.

## <a name="set-parameter-values"></a>Définir les valeurs de paramètres

Les valeurs ci-dessous sont utilisées dans les commandes suivantes pour créer la base de données et les ressources requises. Les noms de serveur doivent être globalement uniques dans l’ensemble d’Azure, de sorte que la fonction $RANDOM est utilisée pour créer le nom du serveur. Remplacez les valeurs 0.0.0.0 de la plage d’adresses IP pour les faire correspondre à votre environnement spécifique.

```azurecli-interactive
# Set the resource group name and location for your server
resourceGroupName=myResourceGroup
location=eastus

# Set an admin login and password for your database
adminlogin=azureuser
password=Azure1234567!

# Set a server name that is unique to Azure DNS (<server_name>.database.windows.net)
serverName=server-$RANDOM

# Set the ip address range that can access your database
startip=0.0.0.0
endip=0.0.0.0
```

## <a name="create-a-resource-group"></a>Créer un groupe de ressources

Créez un groupe de ressources avec la commande [az group create](/cli/azure/group). Un groupe de ressources Azure est un conteneur logique dans lequel les ressources Azure sont déployées et gérées. L’exemple suivant crée un groupe de ressources nommé *myResourceGroup* à l’emplacement *eastus* :

```azurecli-interactive
az group create --name $resourceGroupName --location $location
```

## <a name="create-a-server"></a>Créer un serveur

Créez un serveur avec la commande [az sql server create](/cli/azure/sql/server).

```azurecli-interactive
az sql server create \
    --name $serverName \
    --resource-group $resourceGroupName \
    --location $location  \
    --admin-user $adminlogin \
    --admin-password $password
```


## <a name="configure-a-firewall-rule-for-the-server"></a>Configurer une règle de pare-feu pour le serveur

Créez une règle de pare-feu avec la commande [az sql server firewall-rule create](/cli/azure/sql/server/firewall-rule).

```azurecli-interactive
az sql server firewall-rule create \
    --resource-group $resourceGroupName \
    --server $serverName \
    -n AllowYourIp \
    --start-ip-address $startip \
    --end-ip-address $endip
```


## <a name="create-a-single-database"></a>Créer une base de données unique

Créez une base de données avec la commande [az sql db create](/cli/azure/sql/db). Le code suivant permet la création


```azurecli-interactive
az sql db create \
    --resource-group $resourceGroupName \
    --server $serverName \
    --name mySampleDatabase \
    --sample-name AdventureWorksLT \
    --edition GeneralPurpose \
    --compute-model Serverless \
    --family Gen5 \
    --capacity 2
```


# <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Vous pouvez créer un groupe de ressources, un serveur et une base de données unique à l’aide de Windows PowerShell.

## <a name="launch-azure-cloud-shell"></a>Lancement d’Azure Cloud Shell

Azure Cloud Shell est un interpréteur de commandes interactif et gratuit que vous pouvez utiliser pour exécuter les étapes de cet article. Il contient des outils Azure courants préinstallés et configurés pour être utilisés avec votre compte. 

Pour ouvrir Cloud Shell, sélectionnez simplement **Essayer** en haut à droite d’un bloc de code. Vous pouvez également lancer Cloud Shell dans un onglet distinct du navigateur en accédant à [https://shell.azure.com](https://shell.azure.com). Sélectionnez **Copier** pour copier les blocs de code, collez-les dans Cloud Shell, puis appuyez sur **Entrée** pour les exécuter.

## <a name="set-parameter-values"></a>Définir les valeurs de paramètres

Les valeurs ci-dessous sont utilisées dans les commandes suivantes pour créer la base de données et les ressources requises. Les noms de serveur doivent être globalement uniques dans l’ensemble d’Azure, de sorte que l’applet de commande Get-Random est utilisée pour créer le nom du serveur. Remplacez les valeurs 0.0.0.0 de la plage d’adresses IP pour les faire correspondre à votre environnement spécifique.

```azurepowershell-interactive
   # Set variables for your server and database
   $resourceGroupName = "myResourceGroup"
   $location = "eastus"
   $adminLogin = "azureuser"
   $password = "Azure1234567!"
   $serverName = "mysqlserver-$(Get-Random)"
   $databaseName = "mySampleDatabase"

   # The ip address range that you want to allow to access your server
   $startIp = "0.0.0.0"
   $endIp = "0.0.0.0"

   # Show randomized variables
   Write-host "Resource group name is" $resourceGroupName
   Write-host "Server name is" $serverName
```


## <a name="create-resource-group"></a>Créer un groupe de ressources

Créez un groupe de ressources Azure avec [New-AzResourceGroup](/powershell/module/az.resources/new-azresourcegroup). Un groupe de ressources est un conteneur logique dans lequel les ressources Azure sont déployées et gérées.

```azurepowershell-interactive
   Write-host "Creating resource group..."
   $resourceGroup = New-AzResourceGroup -Name $resourceGroupName -Location $location -Tag @{Owner="SQLDB-Samples"}
   $resourceGroup
```


## <a name="create-a-server"></a>Créer un serveur

Créez un serveur avec l’applet de commande [New-AzSqlServer](/powershell/module/az.sql/new-azsqlserver).

```azurepowershell-interactive
  Write-host "Creating primary server..."
   $server = New-AzSqlServer -ResourceGroupName $resourceGroupName `
      -ServerName $serverName `
      -Location $location `
      -SqlAdministratorCredentials $(New-Object -TypeName System.Management.Automation.PSCredential `
      -ArgumentList $adminLogin, $(ConvertTo-SecureString -String $password -AsPlainText -Force))
   $server
```

## <a name="create-a-firewall-rule"></a>Créer une règle de pare-feu

Créez une règle de pare-feu de serveur avec l’applet de commande [New-AzSqlServerFirewallRule](/powershell/module/az.sql/new-azsqlserverfirewallrule).

```azurepowershell-interactive
   Write-host "Configuring server firewall rule..."
   $serverFirewallRule = New-AzSqlServerFirewallRule -ResourceGroupName $resourceGroupName `
      -ServerName $serverName `
      -FirewallRuleName "AllowedIPs" -StartIpAddress $startIp -EndIpAddress $endIp
   $serverFirewallRule
```


## <a name="create-a-single-database"></a>Créer une base de données unique

Créez une base de données unique avec l’applet de commande [New-AzSqlDatabase](/powershell/module/az.sql/new-azsqldatabase).

```azurepowershell-interactive
   Write-host "Creating a gen5 2 vCore serverless database..."
   $database = New-AzSqlDatabase  -ResourceGroupName $resourceGroupName `
      -ServerName $serverName `
      -DatabaseName $databaseName `
      -Edition GeneralPurpose `
      -ComputeModel Serverless `
      -ComputeGeneration Gen5 `
      -VCore 2 `
      -MinimumCapacity 2 `
      -SampleName "AdventureWorksLT"
   $database
```

---



## <a name="query-the-database"></a>Interroger la base de données

Une fois la base de données créée, vous pouvez utiliser l’**éditeur de requête (préversion)** dans le portail Azure pour vous connecter à la base de données et interroger ses données.

1. Dans le portail, recherchez et sélectionnez **Bases de données SQL**, puis sélectionnez votre base de données dans la liste.
1. Dans la page de votre base de données, sélectionnez **Éditeur de requête (préversion)** dans le menu de gauche.
1. Entrez vos informations de connexion d’administrateur de serveur, puis sélectionnez **OK**.

   ![Se connecter à l’Éditeur de requêtes](./media/single-database-create-quickstart/query-editor-login.png)

1. Entrez la requête suivante dans le volet **Éditeur de requêtes**.

   ```sql
   SELECT TOP 20 pc.Name as CategoryName, p.name as ProductName
   FROM SalesLT.ProductCategory pc
   JOIN SalesLT.Product p
   ON pc.productcategoryid = p.productcategoryid;
   ```

1. Sélectionnez **Exécuter**, puis passez en revue les résultats de la requête dans le volet **Résultats**.

   ![Interroger les résultats de l’Éditeur](./media/single-database-create-quickstart/query-editor-results.png)

1. Fermez la page **Éditeur de requêtes**, puis sélectionnez **OK** à l’invite pour ignorer les modifications que vous n’avez pas enregistrées.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Conservez le groupe de ressources, le serveur et la base de données unique pour passer aux étapes suivantes et savoir comment vous connecter à votre base de données et interroger ses données à l’aide de différentes méthodes.

Lorsque vous avez terminé d’utiliser ces ressources, vous pouvez supprimer le groupe de ressources que vous avez créé, ce qui supprimera également le serveur et la base de données unique qu’il contient.

### <a name="portal"></a>[Portail](#tab/azure-portal)

Pour supprimer **myResourceGroup** et toutes ses ressources à l’aide du portail Azure :

1. Dans le portail Azure, recherchez puis sélectionnez **Groupes de ressources**, puis sélectionnez **myResourceGroup** dans la liste.
1. Sur la page Groupe de ressources, sélectionnez **Supprimer un groupe de ressources**.
1. Sous **Tapez le nom du groupe de ressources**, entrez *myResourceGroup*, puis sélectionnez **Supprimer**.

### <a name="azure-cli"></a>[Azure CLI](#tab/azure-cli)

Pour supprimer le groupe de ressources et toutes les ressources qu’il contient, exécutez la commande Azure CLI suivante, en utilisant le nom de votre groupe de ressources :

```azurecli-interactive
az group delete --name $resourceGroupName
```

### <a name="powershell"></a>[PowerShell](#tab/azure-powershell)

Pour supprimer le groupe de ressources et toutes les ressources qu’il contient, exécutez l’applet de commande PowerShell suivante, en utilisant le nom de votre groupe de ressources :

```azurepowershell-interactive
Remove-AzResourceGroup -Name $resourceGroupName
```

---

## <a name="next-steps"></a>Étapes suivantes

[Connectez-vous à votre base de données et interrogez ses données](connect-query-content-reference-guide.md) à l’aide de différents outils et langages :
> [!div class="nextstepaction"]
> [Se connecter et effectuer des requêtes à l’aide de SQL Server Management Studio](connect-query-ssms.md)
>
> [Se connecter et effectuer des requêtes à l’aide d’Azure Data Studio](/sql/azure-data-studio/quickstart-sql-database?toc=/azure/sql-database/toc.json)

Vous souhaitez optimiser et réduire vos coûts de cloud ?

> [!div class="nextstepaction"]
> [Démarrer l’analyse des coûts avec Cost Management](../../cost-management-billing/costs/quick-acm-cost-analysis.md?WT.mc_id=costmanagementcontent_docsacmhorizontal_-inproduct-learn)