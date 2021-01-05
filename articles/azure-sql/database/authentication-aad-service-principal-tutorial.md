---
title: Créer des utilisateurs Azure AD avec des principaux de service
description: Ce tutoriel vous guide dans la création d’un utilisateur Azure AD avec des applications Azure AD (principaux de service) dans Azure SQL Database et Azure Synapse Analytics.
ms.service: sql-database
ms.subservice: security
ms.custom: azure-synapse
ms.topic: tutorial
author: GithubMirek
ms.author: mireks
ms.reviewer: vanto
ms.date: 10/21/2020
ms.openlocfilehash: e068ad01c07af4e5833399c0053da3362cd6aaa6
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96185638"
---
# <a name="tutorial-create-azure-ad-users-using-azure-ad-applications"></a>Tutoriel : Créer des utilisateurs Azure AD avec des applications Azure AD

[!INCLUDE[appliesto-sqldb-asa](../includes/appliesto-sqldb-asa.md)]

> [!NOTE]
> Cet article est en **préversion publique**. Pour plus d’informations, consultez [Principal de service Azure Active Directory avec Azure SQL](authentication-aad-service-principal.md). Cet article utilise Azure SQL Database pour illustrer les étapes essentielles du tutoriel. Il s’applique cependant de manière similaire à [Azure Synapse Analytics](../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md).

Cet article vous guide dans le processus de création d’utilisateurs Azure AD dans Azure SQL Database avec des principaux de service Azure (applications Azure AD). Cette fonctionnalité existe déjà dans Azure SQL Managed Instance, mais elle est à présent introduite dans Azure SQL Database et Azure Synapse Analytics. Dans ce scénario, vous devez générer une identité Azure AD et l’attribuer au serveur logique Azure SQL.

Pour plus d’informations sur l’authentification Azure AD pour Azure SQL, consultez l’article [Utiliser l’authentification Azure Active Directory](authentication-aad-overview.md).

Dans ce tutoriel, vous allez apprendre à :

> [!div class="checklist"]
> - Attribuer une identité au serveur logique Azure SQL
> - Attribuer une autorisation Lecteurs de répertoire à l’identité du serveur logique SQL
> - Créer un principal de service (une application Azure AD) dans Azure AD
> - Créer un utilisateur de principal de service dans Azure SQL Database
> - Créer un utilisateur Azure AD différent dans SQL Database à l’aide d’un utilisateur de principal de service Azure AD

## <a name="prerequisites"></a>Prérequis

- Un déploiement [Azure SQL Database](single-database-create-quickstart.md) ou [Azure Synapse Analytics](../../synapse-analytics/sql-data-warehouse/sql-data-warehouse-overview-what-is.md) existant. Nous supposons que vous disposez d’une base de données SQL opérationnelle pour ce tutoriel.
- L’accès à une instance Azure Active Directory existante.
- Le module [Az.Sql 2.9.0](https://www.powershellgallery.com/packages/Az.Sql/2.9.0) (ou version supérieure) pour utiliser PowerShell afin de configurer une application Azure AD individuelle en tant qu’administrateur Azure AD pour Azure SQL. Assurez-vous que vous avez mis à niveau le module vers la version la plus récente.

## <a name="assign-an-identity-to-the-azure-sql-logical-server"></a>Attribuer une identité au serveur logique Azure SQL

1. Connectez-vous à votre instance Azure Active Directory. Vous devez trouver votre ID de locataire. Pour cela, accédez à votre ressource **Azure Active Directory**  dans le [portail Azure](https://portal.azure.com). Dans le volet **Vue d’ensemble**, vous devez voir votre **ID de locataire**. Exécutez la commande PowerShell suivante :

    - Remplacez `<TenantId>` par votre **ID de locataire**.

    ```powershell
    Connect-AzAccount -Tenant <TenantId>
    ```

    Enregistrez le `TenantId`. Vous l’utiliserez plus tard dans ce tutoriel.

1. Générez une identité Azure AD et attribuez-la au serveur logique Azure SQL. Exécutez la commande PowerShell suivante :

    - Remplacez `<resource group>` et `<server name>` par vos ressources. Si le nom de votre serveur est `myserver.database.windows.net`, remplacez `<server name>` par `myserver`.

    ```powershell
    Set-AzSqlServer -ResourceGroupName <resource group> -ServerName <server name> -AssignIdentity
    ```

    Pour plus d’informations, consultez la page sur la commande [Set-AzSqlServer](/powershell/module/az.sql/set-azsqlserver).

    > [!IMPORTANT]
    > Si une identité Azure AD est configurée pour le serveur logique Azure SQL, l’autorisation [**Lecteurs de répertoire**](../../active-directory/roles/permissions-reference.md#directory-readers) doit lui être accordée. Nous expliquerons cette étape dans la section suivante. Vous **ne devez pas** ignorer cette étape, car l’authentification Azure AD cesserait de fonctionner.

    - Si vous avez déjà utilisé la commande [New-AzSqlServer](/powershell/module/az.sql/new-azsqlserver) avec le paramètre `AssignIdentity` pour créer un serveur SQL, vous devez par la suite exécuter la commande [Set-AzSqlServer](/powershell/module/az.sql/set-azsqlserver) séparément pour activer cette propriété dans la structure Azure.

1. Vérifiez que l’identité du serveur a été correctement attribuée. Exécutez la commande PowerShell suivante :

    - Remplacez `<resource group>` et `<server name>` par vos ressources. Si le nom de votre serveur est `myserver.database.windows.net`, remplacez `<server name>` par `myserver`.
    
    ```powershell
    $xyz = Get-AzSqlServer  -ResourceGroupName <resource group> -ServerName <server name>
    $xyz.identity
    ```

    La sortie doit indiquer `PrincipalId`, `Type` et `TenantId`. L’identité attribuée est le `PrincipalId`.

1. Vous pouvez également vérifier l’identité en accédant au [portail Azure](https://portal.azure.com).

    - Sous la ressource **Azure Active Directory**, accédez à **Applications d’entreprise**. Tapez le nom de votre serveur logique SQL. Vous voyez qu’il a un **ID d’objet** associé à la ressource.
    
    :::image type="content" source="media/authentication-aad-service-principals-tutorial/enterprise-applications-object-id.png" alt-text="ID d’objet":::

## <a name="assign-directory-readers-permission-to-the-sql-logical-server-identity"></a>Attribuer une autorisation Lecteurs de répertoire à l’identité du serveur logique SQL

Pour que l’identité Azure AD attribuée fonctionne correctement avec Azure SQL, l’autorisation Azure AD `Directory Readers` doit être accordée à l’identité du serveur.

Pour accorder cette autorisation nécessaire, exécutez le script suivant.

> [!NOTE] 
> Ce script doit être exécuté par un `Privileged Roles Administrator` ou un `Global Administrator` Azure AD.
>
> Dans la **préversion publique**, vous pouvez affecter le rôle `Directory Readers` à un groupe dans Azure AD. Les propriétaires du groupe peuvent ensuite ajouter l’identité managée en tant que membre de ce groupe, évitant ainsi qu’un `Global Administrator` ou `Privileged Roles Administrator` accorde le rôle `Directory Readers`. Pour plus d’informations sur cette fonctionnalité, consultez [Rôle Lecteurs d’annuaires dans Azure Active Directory pour Azure SQL](authentication-aad-directory-readers-role.md).

- Remplacez `<TenantId>` par le `TenantId` récupéré précédemment.
- Remplacez `<server name>` par le nom de votre serveur logique SQL. Si le nom de votre serveur est `myserver.database.windows.net`, remplacez `<server name>` par `myserver`.

```powershell
# This script grants Azure "Directory Readers" permission to a Service Principal representing the Azure SQL logical server
# It can be executed only by a "Global Administrator" or "Privileged Roles Administrator" type of user.
# To check if the "Directory Readers" permission was granted, execute this script again

Import-Module AzureAD
Connect-AzureAD -TenantId "<TenantId>"    #Enter your actual TenantId
$AssignIdentityName = "<server name>"     #Enter Azure SQL logical server name
 
# Get Azure AD role "Directory Users" and create if it doesn't exist
$roleName = "Directory Readers"
$role = Get-AzureADDirectoryRole | Where-Object {$_.displayName -eq $roleName}
if ($role -eq $null) {
    # Instantiate an instance of the role template
    $roleTemplate = Get-AzureADDirectoryRoleTemplate | Where-Object {$_.displayName -eq $roleName}
    Enable-AzureADDirectoryRole -RoleTemplateId $roleTemplate.ObjectId
    $role = Get-AzureADDirectoryRole | Where-Object {$_.displayName -eq $roleName}
}
 
# Get service principal for managed instance
$roleMember = Get-AzureADServicePrincipal -SearchString $AssignIdentityName
$roleMember.Count
if ($roleMember -eq $null) {
    Write-Output "Error: No Service Principals with name '$($AssignIdentityName)', make sure that AssignIdentityName parameter was entered correctly."
    exit
}

if (-not ($roleMember.Count -eq 1)) {
    Write-Output "Error: More than one service principal with name pattern '$($AssignIdentityName)'"
    Write-Output "Dumping selected service principals...."
    $roleMember
    exit
}
 
# Check if service principal is already member of readers role
$allDirReaders = Get-AzureADDirectoryRoleMember -ObjectId $role.ObjectId
$selDirReader = $allDirReaders | where{$_.ObjectId -match $roleMember.ObjectId}
 
if ($selDirReader -eq $null) {
    # Add principal to readers role
    Write-Output "Adding service principal '$($AssignIdentityName)' to 'Directory Readers' role'..."
    Add-AzureADDirectoryRoleMember -ObjectId $role.ObjectId -RefObjectId $roleMember.ObjectId
    Write-Output "'$($AssignIdentityName)' service principal added to 'Directory Readers' role'..."
 
    #Write-Output "Dumping service principal '$($AssignIdentityName)':"
    #$allDirReaders = Get-AzureADDirectoryRoleMember -ObjectId $role.ObjectId
    #$allDirReaders | where{$_.ObjectId -match $roleMember.ObjectId}
} else {
    Write-Output "Service principal '$($AssignIdentityName)' is already member of 'Directory Readers' role'."
}
```

> [!NOTE]
> La sortie de ce script indique si l’autorisation Lecteurs de répertoire a été accordée à l’identité. Vous pouvez réexécuter le script si vous n’êtes pas certain que l’autorisation a été accordée.

Vous trouverez dans l’article [Provisionner un administrateur Azure AD (SQL Managed Instance)](authentication-aad-configure.md#powershell) une approche similaire de la définition de l’autorisation **Lecteurs de répertoire** pour SQL Managed Instance.

## <a name="create-a-service-principal-an-azure-ad-application-in-azure-ad"></a>Créer un principal de service (une application Azure AD) dans Azure AD

1. Suivez les instructions de la section [Inscrire votre application et définir des autorisations](active-directory-interactive-connect-azure-sql-db.md#register-your-app-and-set-permissions).

    Veillez à ajouter les **permissions d’application** et les **autorisations déléguées**.

    :::image type="content" source="media/authentication-aad-service-principals-tutorial/aad-apps.png" alt-text="Capture d’écran montrant la page des inscriptions d’applications pour Azure Active Directory. Une application portant le nom d’affichage AppSP est mise en surbrillance.":::

    :::image type="content" source="media/authentication-aad-service-principals-tutorial/aad-app-registration-api-permissions.png" alt-text="API - autorisations":::

2. Vous devez aussi créer un secret client pour la connexion. Suivez les instructions ici pour [charger un certificat ou créer un secret pour la connexion](../../active-directory/develop/howto-create-service-principal-portal.md#authentication-two-options).

3. Enregistrez les informations suivantes de votre inscription d’application. Elles doivent être disponibles dans le volet **vue d’ensemble** :
    - **ID d’application**
    - **ID de locataire**, qui doit être le même que celui utilisé précédemment

Dans ce tutoriel, nous allons utiliser *AppSP* comme premier utilisateur du principal de service et *myapp* comme deuxième utilisateur du principal de service. Celui-ci sera créé dans Azure SQL par *AppSP*. Vous devrez créer les deux applications *AppSP* et *myapp*.

Pour plus d’informations sur la création d’une application Azure AD, consultez l’article [Procédure : Utilisez le portail pour créer une application Azure AD et un principal du service pouvant accéder aux ressources](../../active-directory/develop/howto-create-service-principal-portal.md).

### <a name="permissions-required-to-set-or-unset-the-azure-ad-admin"></a>Autorisations nécessaires pour définir ou annuler l’administrateur Azure AD

Pour que le principal de service puisse définir ou annuler un administrateur Azure AD pour Azure SQL, une autorisation d’API supplémentaire est nécessaire. L’autorisation de l’API d’application [Directory.Read.All](/graph/permissions-reference#application-permissions-18) devra être ajoutée à votre application dans Azure AD.

:::image type="content" source="media/authentication-aad-service-principals-tutorial/aad-directory-reader-all-permissions.png" alt-text="Autorisations Directory.Reader.All dans Azure AD":::

Le principal de service a également besoin du rôle [ **Contributeur SQL Server**](../../role-based-access-control/built-in-roles.md#sql-server-contributor) pour SQL Database ou du rôle [**Contributeur SQL Managed Instance**](../../role-based-access-control/built-in-roles.md#sql-managed-instance-contributor) pour SQL Managed Instance.

> [!NOTE]
> Même si l’API Graph Azure AD est dépréciée, l’autorisation **Directory.Reader.All** s’applique toujours à ce tutoriel. En revanche, l’API Microsoft Graph ne s’applique pas à ce tutoriel.

## <a name="create-the-service-principal-user-in-azure-sql-database"></a>Créer l’utilisateur de principal de service dans Azure SQL Database

Après avoir créé un principal de service dans Azure AD, créez l’utilisateur dans SQL Database. Vous devez vous connecter à votre base de données SQL à l’aide d’un ID de connexion valide ayant les autorisations nécessaires pour créer des utilisateurs dans la base de données.

1. Créez l’utilisateur *AppSP* dans la base de données SQL à l’aide de la commande T-SQL suivante :

    ```sql
    CREATE USER [AppSP] FROM EXTERNAL PROVIDER
    GO
    ```

2. Accordez l’autorisation `db_owner` à *AppSP*. Elle permet à l’utilisateur de créer d’autres utilisateurs Azure AD dans la base de données.

    ```sql
    EXEC sp_addrolemember 'db_owner', [AppSP]
    GO
    ```

    Pour plus d’informations, consultez [sp_addrolemember](/sql/relational-databases/system-stored-procedures/sp-addrolemember-transact-sql).

    Vous pouvez également accorder l’autorisation `ALTER ANY USER` au lieu de donner le rôle `db_owner`. Ceci permettra au principal de service d’ajouter d’autres utilisateurs Azure AD.

    ```sql
    GRANT ALTER ANY USER TO [AppSp]
    GO
    ```

    > [!NOTE]
    > Le paramètre ci-dessus n’est pas nécessaire quand *AppSP* est défini comme administrateur Azure AD pour le serveur. Pour définir le principal de service comme administrateur AD pour le serveur logique SQL, vous pouvez utiliser le portail Azure ou des commandes PowerShell ou Azure CLI. Pour plus d’informations, consultez [Provisionner un administrateur Azure AD (SQL Database)](authentication-aad-configure.md?tabs=azure-powershell#powershell-for-sql-database-and-azure-synapse).

## <a name="create-an-azure-ad-user-in-sql-database-using-an-azure-ad-service-principal"></a>Créer un utilisateur Azure AD dans SQL Database à l’aide d’un principal de service Azure AD

> [!IMPORTANT]
> Le principal de service utilisé pour la connexion à SQL Database doit avoir un secret client. Si ce n’est pas le cas, suivez l’étape 2 de la section [Créer un principal de service (une application Azure AD) dans Azure AD](#create-a-service-principal-an-azure-ad-application-in-azure-ad). Ce secret client doit être ajouté comme paramètre d’entrée dans le script ci-dessous.

1. Utilisez le script suivant pour créer un utilisateur de principal de service Azure AD *myapp* avec le principal de service *AppSP*.

    - Remplacez `<TenantId>` par le `TenantId` récupéré précédemment.
    - Remplacez `<ClientId>` par le `ClientId` récupéré précédemment.
    - Remplacez `<ClientSecret>` par le secret client créé précédemment.
    - Remplacez `<server name>` par le nom de votre serveur logique SQL. Si le nom de votre serveur est `myserver.database.windows.net`, remplacez `<server name>` par `myserver`.
    - Remplacez `<database name>` par le nom de votre base de données SQL.

    ```powershell
    # PowerShell script for creating a new SQL user called myapp using application AppSP with secret

    $tenantId = "<TenantId>"   #  tenantID (Azure Directory ID) were AppSP resides
    $clientId = "<ClientId>"   #  AppID also ClientID for AppSP     
    $clientSecret = "<ClientSecret>"   #  client secret for AppSP 
    $Resource = "https://database.windows.net/"
    
    $adalPath  = "${env:ProgramFiles}\WindowsPowerShell\Modules\AzureRM.profile\5.8.3"
    # To install the latest AzureRM.profile version execute  -Install-Module -Name AzureRM.profile
    $adal      = "$adalPath\Microsoft.IdentityModel.Clients.ActiveDirectory.dll"
    $adalforms = "$adalPath\Microsoft.IdentityModel.Clients.ActiveDirectory.WindowsForms.dll"
    [System.Reflection.Assembly]::LoadFrom($adal) | Out-Null
      $resourceAppIdURI = 'https://database.windows.net/'

      # Set Authority to Azure AD Tenant
      $authority = 'https://login.windows.net/' + $tenantId

      $ClientCred = [Microsoft.IdentityModel.Clients.ActiveDirectory.ClientCredential]::new($clientId, $clientSecret)
      $authContext = [Microsoft.IdentityModel.Clients.ActiveDirectory.AuthenticationContext]::new($authority)
      $authResult = $authContext.AcquireTokenAsync($resourceAppIdURI,$ClientCred)
      $Tok = $authResult.Result.CreateAuthorizationHeader()
      $Tok=$Tok.Replace("Bearer ","")
      Write-host "token"
      $Tok
      Write-host  " "

    $SQLServerName = "<server name>"    # Azure SQL logical server name 
    Write-Host "Create SQL connectionstring"
    $conn = New-Object System.Data.SqlClient.SQLConnection 
    $DatabaseName = "<database name>"     # Azure SQL database name
    $conn.ConnectionString = "Data Source=$SQLServerName.database.windows.net;Initial Catalog=$DatabaseName;Connect Timeout=30"
    $conn.AccessToken = $Tok
    
    Write-host "Connect to database and execute SQL script"
    $conn.Open() 
    $ddlstmt = 'CREATE USER [myapp] FROM EXTERNAL PROVIDER;'
    Write-host " "
    Write-host "SQL DDL command"
    $ddlstmt
    $command = New-Object -TypeName System.Data.SqlClient.SqlCommand($ddlstmt, $conn)       
    
    Write-host "results"
    $command.ExecuteNonQuery()
    $conn.Close()  
    ``` 

    Vous pouvez également utiliser l’exemple de code fourni dans le blog [Azure AD Service Principal authentication to SQL DB - Code Sample](https://techcommunity.microsoft.com/t5/azure-sql-database/azure-ad-service-principal-authentication-to-sql-db-code-sample/ba-p/481467) (Authentification auprès d’une base de données SQL avec le principal de service Azure AD - exemple de code). Modifiez le script pour exécuter l’instruction DDL `CREATE USER [myapp] FROM EXTERNAL PROVIDER`. Vous pouvez utiliser le même script pour créer un utilisateur Azure AD standard ou un groupe dans SQL Database.

    > [!NOTE]
    > Si vous devez installer le module AzureRM.profile, vous devez ouvrir PowerShell en tant qu’administrateur. Vous pouvez utiliser les commandes suivantes pour installer automatiquement la dernière version d’AzureRM.profile et définir `$adalpath` pour le script ci-dessus :
    > 
    > ```powershell
    > Install-Module AzureRM.profile -force
    > Import-Module AzureRM.profile
    > $version = (Get-Module -Name AzureRM.profile).Version.toString()
    > $adalPath = "${env:ProgramFiles}\WindowsPowerShell\Modules\AzureRM.profile\${version}"
    > ```
    
2. Vérifiez si l’utilisateur *myapp* existe dans la base de données en exécutant la commande suivante :

    ```sql
    SELECT name, type, type_desc, CAST(CAST(sid as varbinary(16)) as uniqueidentifier) as appId from sys.database_principals WHERE name = 'myapp'
    GO
    ```

    La sortie doit ressembler à celle-ci :

    ```output
    name    type    type_desc   appId
    myapp   E   EXTERNAL_USER   6d228f48-xxxx-xxxx-xxxx-xxxxxxxxxxxx
    ```

## <a name="next-steps"></a>Étapes suivantes

- [Principal de service Azure Active Directory avec Azure SQL](authentication-aad-service-principal.md)
- [Que sont les identités managées pour les ressources Azure ?](../../active-directory/managed-identities-azure-resources/overview.md)
- [Guide pratique pour utiliser des identités managées avec App Service et Azure Functions](../../app-service/overview-managed-identity.md)
- [Azure AD Service Principal authentication to SQL DB - Code Sample](https://techcommunity.microsoft.com/t5/azure-sql-database/azure-ad-service-principal-authentication-to-sql-db-code-sample/ba-p/481467) (Authentification auprès d’une base de données SQL avec le principal de service Azure AD - exemple de code)
- [Objets application et principal du service dans Azure Active Directory](../../active-directory/develop/app-objects-and-service-principals.md)
- [Créer un principal de service Azure avec Azure PowerShell](/powershell/azure/create-azure-service-principal-azureps)
- [Rôle Lecteurs d’annuaires dans Azure Active Directory pour Azure SQL](authentication-aad-directory-readers-role.md)