---
title: Activer AAD pour Azure-SSIS Integration Runtime
description: Cet article décrit comment activer l’authentification Azure Active Directory avec l’identité managée pour Azure Data Factory afin de créer un Azure-SSIS Integration Runtime.
services: data-factory
ms.service: data-factory
ms.workload: data-services
ms.devlang: powershell
ms.topic: conceptual
author: swinarko
ms.author: sawinark
manager: mflasko
ms.custom: seo-lt-2019
ms.date: 07/09/2020
ms.openlocfilehash: 30f5b5990e189cb6942c15b65b6a417ce49f0c2b
ms.sourcegitcommit: fb3c846de147cc2e3515cd8219d8c84790e3a442
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92637800"
---
# <a name="enable-azure-active-directory-authentication-for-azure-ssis-integration-runtime"></a>Activer l’authentification Azure Active Directory pour Azure-SSIS Integration Runtime

[!INCLUDE[appliesto-adf-xxx-md](includes/appliesto-adf-xxx-md.md)]

Cet article vous explique comment activer l’authentification Azure Active Directory (Azure AD) avec l’identité managée pour votre Azure Data Factory (ADF) et l’utiliser à la place de méthodes d’authentification conventionnelles (telles que l’authentification SQL) pour :

- Créez un runtime d’intégration Azure-SSIS qui, à son tour, provisionne une base de données de catalogue SSIS (SSISDB) dans une base de données SQL ou une instance managée SQL en votre nom.

- Connectez-vous à différentes ressources Azure lors de l’exécution de packages SSIS sur Azure-SSIS IR.

Pour plus d’informations sur l’identité managée d’une fabrique ADF, consultez [Identité managée pour Data Factory](./data-factory-service-identity.md).

> [!NOTE]
>
> - Dans ce scénario, l’authentification Azure AD avec l’identité managée pour votre ADF est utilisée uniquement pour la création et les opérations de démarrage suivantes de votre SSIS IR qui, à son tour, approvisionnera et se connectera à la base de données SSISDB. Pour les exécutions de package SSIS, votre SSIS IR se connectera toujours à la base de données SSISDB à l’aide de l’authentification SQL avec des comptes entièrement gérés, créés lors de l’approvisionnement de la base de données SSISDB.
> - Si vous avez déjà créé un SSIS IR à l’aide de l’authentification SQL, vous ne pouvez pas pour l’instant le reconfigurer pour utiliser l’authentification Azure AD avec PowerShell, mais vous pouvez le faire dans l’application ADF/le Portail Azure. 

[!INCLUDE [updated-for-az](../../includes/updated-for-az.md)]

## <a name="enable-azure-ad-on-azure-sql-database"></a>Activer Azure AD sur Azure SQL Database

SQL Database prend en charge la création d’une base de données avec un utilisateur Azure AD. Tout d’abord, vous devez créer un groupe Azure AD avec l’identité managée pour votre ADF en tant que membre. Ensuite, vous devez définir un utilisateur Azure AD comme administrateur Active Directory pour votre base de données SQL, puis vous y connecter sur SSMS (SQL Server Management Studio) avec cet utilisateur. Pour finir, vous devez créer un utilisateur de base de données autonome représentant le groupe Azure AD, afin que l’identité managée pour votre ADF soit utilisable par Azure-SSIS IR pour créer SSISDB à votre place.

### <a name="create-an-azure-ad-group-with-the-managed-identity-for-your-adf-as-a-member"></a>Créer un groupe Azure AD avec l’identité managée pour votre ADF en tant que membre

Vous pouvez utiliser un groupe Azure AD existant ou en créer un à l’aide d’Azure AD PowerShell.

1.  Installez le module [Azure AD PowerShell](/powershell/azure/active-directory/install-adv2).

2.  Connectez-vous avec `Connect-AzureAD`, exécutez la cmdlet suivante pour créer un groupe, puis enregistrez-le dans une variable :

    ```powershell
    $Group = New-AzureADGroup -DisplayName "SSISIrGroup" `
                              -MailEnabled $false `
                              -SecurityEnabled $true `
                              -MailNickName "NotSet"
    ```

    Le résultat ressemble à l’exemple suivant, qui affiche également la valeur de la variable :

    ```powershell
    $Group

    ObjectId DisplayName Description
    -------- ----------- -----------
    6de75f3c-8b2f-4bf4-b9f8-78cc60a18050 SSISIrGroup
    ```

3.  Ajoutez au groupe l’identité managée pour votre service Azure Data Factory. Vous pouvez suivre l’article [Identité managée pour Data Factory](./data-factory-service-identity.md) pour obtenir l’ID objet principal de l’identité managée (p. ex., 765ad4ab-XXXX-XXXX-XXXX-51ed985819dc), mais n’utilisez pas l’ID d’application de l’identité managée à cette fin.

    ```powershell
    Add-AzureAdGroupMember -ObjectId $Group.ObjectId -RefObjectId 765ad4ab-XXXX-XXXX-XXXX-51ed985819dc
    ```

    Vous pouvez également vérifier l’appartenance au groupe par la suite.

    ```powershell
    Get-AzureAdGroupMember -ObjectId $Group.ObjectId
    ```

### <a name="configure-azure-ad-authentication-for-sql-database"></a>Configurer l’authentification Azure AD pour SQL Database

Pour [Configurer et gérer l'authentification Azure AD avec SQL](../azure-sql/database/authentication-aad-configure.md), procédez comme suit :

1.  Dans le portail Azure, dans le volet de navigation gauche, sélectionnez **Tous les services** -> **Serveurs SQL**.

2.  Sélectionnez votre serveur dans SQL Database à configurer avec l’authentification Azure AD.

3.  Dans la section **Paramètres** du panneau, sélectionnez **Administrateur Active Directory**.

4.  Dans la barre de commandes, sélectionnez **Définir l’administrateur**.

5.  Sélectionnez un compte d’utilisateur Azure AD à désigner comme administrateur du serveur, puis choisissez **Sélectionner**.

6.  Dans la barre de commandes, sélectionnez **Enregistrer**.

### <a name="create-a-contained-user-in-sql-database-representing-the-azure-ad-group"></a>Créer un utilisateur de base de données autonome dans SQL Database représentant le groupe Azure AD

Pour cette nouvelle étape, vous avez besoin de [Microsoft SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) (SSMS).

1. Démarrer SSMS.

2. Dans la boîte de dialogue **Se connecter au serveur** , entrez le nom de votre serveur dans le champ **Nom du serveur**.

3. Dans le champ **Authentification** , sélectionnez **Active Directory - Authentification universelle avec prise en charge de MFA** (vous pouvez également utiliser les deux autres types d’authentification Active Directory ; consultez [Configurer et gérer l’authentification Azure AD avec SQL](../azure-sql/database/authentication-aad-configure.md)).

4. Dans le champ **Nom d’utilisateur** , entrez le nom du compte Azure AD que vous avez défini comme administrateur du serveur, par exemple testuser@xxxonline.com.

5. Sélectionnez **Se connecter** et terminez le processus de connexion.

6. Dans l’ **Explorateur d’objets** , développez le dossier **Bases de données** -> **Bases de données système**.

7. Cliquez avec le bouton droit sur la base de données **master** et sélectionnez **Nouvelle requête**.

8. Dans la fenêtre de requête, entrez la commande T-SQL suivante, puis sélectionnez **Exécuter** dans la barre d’outils.

   ```sql
   CREATE USER [SSISIrGroup] FROM EXTERNAL PROVIDER
   ```

   La commande doit s’exécuter correctement, en créant l’utilisateur de base de données autonome pour représenter le groupe.

9. Effacez la fenêtre de requête, entrez la commande T-SQL suivante, puis sélectionnez **Exécuter** dans la barre d’outils.

   ```sql
   ALTER ROLE dbmanager ADD MEMBER [SSISIrGroup]
   ```

   La commande doit s’exécuter correctement, et octroyer à l’utilisateur contenu la capacité des créer une base de données (SSISDB).

10. Si votre SSISDB a été créé à l’aide de l’authentification SQL et que vous souhaitez utiliser l’authentification Azure AD pour votre que votre runtime d’intégration Azure-SSIS y accède, commencez par vérifier que les étapes permettant d’accorder l’autorisation à la base de données **master** ont été accomplies avec succès. Ensuite, cliquez avec le bouton droit sur la base de données **SSISDB** et sélectionnez **Nouvelle requête**.

11. Dans la fenêtre de requête, entrez la commande T-SQL suivante, puis sélectionnez **Exécuter** dans la barre d’outils.

    ```sql
    CREATE USER [SSISIrGroup] FROM EXTERNAL PROVIDER
    ```

    La commande doit s’exécuter correctement, en créant l’utilisateur de base de données autonome pour représenter le groupe.

12. Effacez la fenêtre de requête, entrez la commande T-SQL suivante, puis sélectionnez **Exécuter** dans la barre d’outils.

    ```sql
    ALTER ROLE db_owner ADD MEMBER [SSISIrGroup]
    ```

    La commande doit s’exécuter correctement et octroyer à l’utilisateur contenu la capacité d’accéder à la base de données SSISDB.

## <a name="enable-azure-ad-on-sql-managed-instance"></a>Activer Azure AD sur SQL Managed Instance

SQL Managed Instance prend en charge la création directe d’une base de données avec l’identité managée pour votre ADF. Vous n’avez pas besoin de joindre l’identité managée pour votre ADF à un groupe Azure AD ni de créer un utilisateur autonome représentant ce groupe dans SQL Managed Instance.

### <a name="configure-azure-ad-authentication-for-azure-sql-managed-instance"></a>Configurer l’authentification Azure AD pour Azure SQL Managed Instance

Suivez les étapes décrites dans [Provisionner un administrateur Azure Active Directory pour SQL Managed Instance](../azure-sql/database/authentication-aad-configure.md#provision-azure-ad-admin-sql-managed-instance).

### <a name="add-the-managed-identity-for-your-adf-as-a-user-in-sql-managed-instance"></a>Ajouter l’identité managée pour votre ADF comme utilisateur dans SQL Managed Instance

Pour cette nouvelle étape, vous avez besoin de [Microsoft SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms) (SSMS).

1.  Démarrer SSMS.

2.  Connectez-vous à votre instance managée SQL à l’aide d’un compte SQL Server **administrateur système**. Il s’agit d’une limitation temporaire qui sera levée quand les principaux de serveur (connexions) Azure AD d’Azure SQL Managed Instance seront en disponibilité générale. L’erreur suivante s’affiche si vous essayez d’utiliser un compte d’administrateur Azure AD pour créer la connexion : Msg 15247, Niveau 16, État 1, Ligne 1 L’utilisateur n’est pas autorisé à effectuer cette action.

3.  Dans l’ **Explorateur d’objets** , développez le dossier **Bases de données** -> **Bases de données système**.

4.  Cliquez avec le bouton droit sur la base de données **master** et sélectionnez **Nouvelle requête**.

5.  Dans la fenêtre de requête, exécutez le script T-SQL suivant pour ajouter l’identité managée pour votre ADF comme utilisateur.

    ```sql
    CREATE LOGIN [{your ADF name}] FROM EXTERNAL PROVIDER
    ALTER SERVER ROLE [dbcreator] ADD MEMBER [{your ADF name}]
    ALTER SERVER ROLE [securityadmin] ADD MEMBER [{your ADF name}]
    ```
    
    La commande doit s’exécuter correctement et octroyer à l’identité managée pour votre ADF la capacité de créer une base de données.

6.  Si votre SSISDB a été créé à l’aide de l’authentification SQL et que vous souhaitez utiliser l’authentification Azure AD pour votre que votre runtime d’intégration Azure-SSIS y accède, commencez par vérifier que les étapes permettant d’accorder l’autorisation à la base de données **master** ont été accomplies avec succès. Ensuite, cliquez avec le bouton droit sur la base de données **SSISDB** et sélectionnez **Nouvelle requête**.

7.  Dans la fenêtre de requête, entrez la commande T-SQL suivante, puis sélectionnez **Exécuter** dans la barre d’outils.

    ```sql
    CREATE USER [{your ADF name}] FOR LOGIN [{your ADF name}] WITH DEFAULT_SCHEMA = dbo
    ALTER ROLE db_owner ADD MEMBER [{your ADF name}]
    ```

    La commande doit s’exécuter correctement et octroyer à l’identité managée pour votre ADF la capacité d’accéder à la base de données SSISDB.

## <a name="provision-azure-ssis-ir-in-azure-portaladf-app"></a>Provisionner Azure-SSIS IR dans le portail Azure/l’application ADF

Quand vous provisionnez votre Azure-SSIS IR dans le portail Azure/l’application ADF, dans la page **Paramètres SQL** , sélectionnez l’option **Utiliser l’authentification AAD avec l’identité managée pour votre ADF**. La capture d’écran suivante montre les paramètres pour IR avec SQL Database hébergeant SSISDB. Pour IR avec SQL Managed Instance hébergeant SSISDB, les paramètres **Niveau de service de base de données de catalogue** et **Autoriser les services Azure à accéder au serveur** ne sont pas applicables, tandis que les autres paramètres sont identiques.

Pour plus d’informations sur la création d’un Azure-SSIS IR, consultez [Créer un runtime d’intégration Azure-SSIS dans Azure Data Factory](./create-azure-ssis-integration-runtime.md).

![Paramètres pour le runtime d’intégration Azure-SSIS](media/enable-aad-authentication-azure-ssis-ir/enable-aad-authentication.png)

## <a name="provision-azure-ssis-ir-with-powershell"></a>Provisionner Azure-SSIS IR avec PowerShell

Pour provisionner votre runtime d’intégration Azure-SSIS avec PowerShell, procédez comme suit :

1.  Installez le module [Azure PowerShell](https://github.com/Azure/azure-powershell/releases/tag/v5.5.0-March2018).

2.  Dans votre script, ne définissez pas le paramètre `CatalogAdminCredential`. Par exemple :

    ```powershell
    Set-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
                                               -DataFactoryName $DataFactoryName `
                                               -Name $AzureSSISName `
                                               -Description $AzureSSISDescription `
                                               -Type Managed `
                                               -Location $AzureSSISLocation `
                                               -NodeSize $AzureSSISNodeSize `
                                               -NodeCount $AzureSSISNodeNumber `
                                               -Edition $AzureSSISEdition `
                                               -MaxParallelExecutionsPerNode $AzureSSISMaxParallelExecutionsPerNode `
                                               -CatalogServerEndpoint $SSISDBServerEndpoint `
                                               -CatalogPricingTier $SSISDBPricingTier

    Start-AzDataFactoryV2IntegrationRuntime -ResourceGroupName $ResourceGroupName `
                                                 -DataFactoryName $DataFactoryName `
                                                 -Name $AzureSSISName
    ```

## <a name="run-ssis-packages-with-managed-identity-authentication"></a>Exécuter des packages SSIS avec l’authentification d’identité managée

Lorsque vous exécutez des packages SSIS sur Azure-SSIS Integration Runtime (IR), vous pouvez utiliser l’authentification d’identité managée pour vous connecter à différentes ressources Azure. Actuellement, nous prenons déjà en charge l’authentification d’identité managée dans les gestionnaires de connexions suivants.

- [Gestionnaire de connexions OLE DB](/sql/integration-services/connection-manager/ole-db-connection-manager#managed-identities-for-azure-resources-authentication)

- [Gestionnaire de connexions ADO.NET](/sql/integration-services/connection-manager/ado-net-connection-manager#managed-identities-for-azure-resources-authentication)

- [Gestionnaire de connexions Stockage Azure](/sql/integration-services/connection-manager/azure-storage-connection-manager#managed-identities-for-azure-resources-authentication)
