---
title: Code du fichier d’événements XEvent
description: Fournit PowerShell et Transact-SQL pour un exemple de code en deux phases qui montre l’utilisation de la cible du fichier d’événements dans un événement étendu sur Azure SQL Database. Le service Azure Storage est nécessaire pour ce scénario.
services: sql-database
ms.service: sql-database
ms.subservice: performance
ms.custom: sqldbrb=1
ms.devlang: PowerShell
ms.topic: sample
author: WilliamDAssafMSFT
ms.author: wiassaf
ms.reviewer: sstein
ms.date: 06/06/2020
ms.openlocfilehash: d7a57f98551cf91ed87858caba0907471bcf6b12
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96501224"
---
# <a name="event-file-target-code-for-extended-events-in-azure-sql-database"></a>Code cible du fichier d’événements pour les événements étendus dans Azure SQL Database
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

[!INCLUDE [sql-database-xevents-selectors-1-include](../../../includes/sql-database-xevents-selectors-1-include.md)]

Vous pouvez utiliser un exemple de code complet pour capturer et signaler les informations liées à un événement étendu.

Dans Microsoft SQL Server, la [cible du fichier d’événements](/previous-versions/sql/sql-server-2016/ff878115(v=sql.130)) est utilisée pour stocker les sorties d’événement dans un fichier sur un disque dur local. Mais ce type de fichier n’est pas disponible dans Azure SQL Database. À la place, nous utilisons le service Azure Storage pour prendre en charge la cible du fichier d’événements.

Cette rubrique présente un exemple de code en deux phases :

- PowerShell, pour créer un conteneur Azure Storage dans le cloud.
- Transact-SQL :
  - Pour affecter le conteneur Azure Storage à une cible du fichier d’événements.
  - Pour créer et démarrer la session d’événement, etc.

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [updated-for-az](../../../includes/updated-for-az.md)]

> [!IMPORTANT]
> Le module PowerShell Azure Resource Manager est toujours pris en charge par Azure SQL Database, mais tous les développements futurs sont destinés au module Az.Sql. Pour ces cmdlets, voir [AzureRM.Sql](/powershell/module/AzureRM.Sql/). Les arguments des commandes dans le module Az sont sensiblement identiques à ceux des modules AzureRm.

- Un compte et un abonnement Azure. Vous pouvez vous inscrire à un [essai gratuit](https://azure.microsoft.com/pricing/free-trial/).
- Une base de données dans laquelle vous pouvez créer une table.
  
  - Vous pouvez aussi [créer une base de données de démonstration **AdventureWorksLT**](single-database-create-quickstart.md) en quelques minutes.

- SQL Server Management Studio (ssms.exe), dans l’idéal, la version de sa dernière mise à jour mensuelle.
  Vous pouvez télécharger la dernière version de ssms.exe :
  
  - À partir de la rubrique [Télécharger SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms).
  - [En utilisant un lien direct vers le téléchargement.](https://go.microsoft.com/fwlink/?linkid=616025)

- Les [modules Azure PowerShell](https://go.microsoft.com/?linkid=9811175) doivent être installés.

  - Ces modules fournissent des commandes, telles que **New-AzStorageAccount**.

## <a name="phase-1-powershell-code-for-azure-storage-container"></a>Phase 1 : code PowerShell pour conteneur Stockage Azure

Il s’agit de la première phase de l’exemple de code en deux phases.

Le script commence par des commandes à supprimer s’il a déjà été exécuté, et est réexécutable.

1. Collez le script PowerShell dans un éditeur de texte simple, tel que Notepad.exe, puis enregistrez-le dans un fichier avec l’extension **.ps1**.
2. Démarrez PowerShell ISE en tant qu’administrateur.
3. À l’invite, tapez<br/>`Set-ExecutionPolicy -ExecutionPolicy Unrestricted -Scope CurrentUser`<br/>et appuyez sur Entrée.
4. Dans PowerShell ISE, ouvrez votre fichier **.ps1** . Exécutez le script.
5. Tout d’abord, le script ouvre une nouvelle fenêtre pour vous permettre de vous connecter à Azure.

   - Si vous réexécutez le script sans interrompre votre session, vous avez la possibilité de commenter la commande **Add-AzureAccount** .

![PowerShell ISE, avec le module Azure installé, prêt à exécuter le script.](./media/xevent-code-event-file/event-file-powershell-ise-b30.png)

### <a name="powershell-code"></a>Code PowerShell

Ce script PowerShell part du principe que vous avez déjà installé le module Az. Pour plus d’informations, consultez [Installer le module Azure PowerShell](/powershell/azure/install-Az-ps).

```powershell
## TODO: Before running, find all 'TODO' and make each edit!!

cls;

#--------------- 1 -----------------------

'Script assumes you have already logged your PowerShell session into Azure.
But if not, run  Connect-AzAccount (or  Connect-AzAccount), just one time.';
#Connect-AzAccount;   # Same as  Connect-AzAccount.

#-------------- 2 ------------------------

'
TODO: Edit the values assigned to these variables, especially the first few!
';

# Ensure the current date is between
# the Expiry and Start time values that you edit here.

$subscriptionName    = 'YOUR_SUBSCRIPTION_NAME';
$resourceGroupName   = 'YOUR_RESOURCE-GROUP-NAME';

$policySasExpiryTime = '2018-08-28T23:44:56Z';
$policySasStartTime  = '2017-10-01';

$storageAccountLocation = 'YOUR_STORAGE_ACCOUNT_LOCATION';
$storageAccountName     = 'YOUR_STORAGE_ACCOUNT_NAME';
$contextName            = 'YOUR_CONTEXT_NAME';
$containerName          = 'YOUR_CONTAINER_NAME';
$policySasToken         = ' ? ';

$policySasPermission = 'rwl';  # Leave this value alone, as 'rwl'.

#--------------- 3 -----------------------

# The ending display lists your Azure subscriptions.
# One should match the $subscriptionName value you assigned
#   earlier in this PowerShell script.

'Choose an existing subscription for the current PowerShell environment.';

Select-AzSubscription -Subscription $subscriptionName;

#-------------- 4 ------------------------

'
Clean up the old Azure Storage Account after any previous run,
before continuing this new run.';

if ($storageAccountName) {
    Remove-AzStorageAccount `
        -Name              $storageAccountName `
        -ResourceGroupName $resourceGroupName;
}

#--------------- 5 -----------------------

[System.DateTime]::Now.ToString();

'
Create a storage account.
This might take several minutes, will beep when ready.
  ...PLEASE WAIT...';

New-AzStorageAccount `
    -Name              $storageAccountName `
    -Location          $storageAccountLocation `
    -ResourceGroupName $resourceGroupName `
    -SkuName           'Standard_LRS';

[System.DateTime]::Now.ToString();
[System.Media.SystemSounds]::Beep.Play();

'
Get the access key for your storage account.
';

$accessKey_ForStorageAccount = `
    (Get-AzStorageAccountKey `
        -Name              $storageAccountName `
        -ResourceGroupName $resourceGroupName
        ).Value[0];

"`$accessKey_ForStorageAccount = $accessKey_ForStorageAccount";

'Azure Storage Account cmdlet completed.
Remainder of PowerShell .ps1 script continues.
';

#--------------- 6 -----------------------

# The context will be needed to create a container within the storage account.

'Create a context object from the storage account and its primary access key.
';

$context = New-AzStorageContext `
    -StorageAccountName $storageAccountName `
    -StorageAccountKey  $accessKey_ForStorageAccount;

'Create a container within the storage account.
';

$containerObjectInStorageAccount = New-AzStorageContainer `
    -Name    $containerName `
    -Context $context;

'Create a security policy to be applied to the SAS token.
';

New-AzStorageContainerStoredAccessPolicy `
    -Container  $containerName `
    -Context    $context `
    -Policy     $policySasToken `
    -Permission $policySasPermission `
    -ExpiryTime $policySasExpiryTime `
    -StartTime  $policySasStartTime;

'
Generate a SAS token for the container.
';
try {
    $sasTokenWithPolicy = New-AzStorageContainerSASToken `
        -Name    $containerName `
        -Context $context `
        -Policy  $policySasToken;
}
catch {
    $Error[0].Exception.ToString();
}

#-------------- 7 ------------------------

'Display the values that YOU must edit into the Transact-SQL script next!:
';

"storageAccountName: $storageAccountName";
"containerName:      $containerName";
"sasTokenWithPolicy: $sasTokenWithPolicy";

'
REMINDER: sasTokenWithPolicy here might start with "?" character, which you must exclude from Transact-SQL.
';

'
(Later, return here to delete your Azure Storage account. See the preceding  Remove-AzStorageAccount -Name $storageAccountName)';

'
Now shift to the Transact-SQL portion of the two-part code sample!';

# EOFile
```

Prenez note des quelques valeurs nommées que le script PowerShell affiche à la fin de son exécution. Vous devrez modifier ces valeurs dans le script Transact-SQL lors de la phase 2 ci-après.

<!--
TODO:   Consider whether the preceding PowerShell code example deserves to be updated to the latest package (AzureRM.SQL?).
2020/June/06   Adding the !NOTE below about "ADLS Gen2 storage accounts".
Related to   https://github.com/MicrosoftDocs/azure-docs/issues/56520
-->

> [!NOTE]
> Dans l’exemple de code PowerShell précédent, les événements étendus SQL ne sont pas compatibles avec les comptes de stockage ADLS Gen2.

## <a name="phase-2-transact-sql-code-that-uses-azure-storage-container"></a>Phase 2 : code Transact-SQL utilisant un conteneur Stockage Azure

- Lors de la phase 1 de cet exemple de code, vous avez exécuté un script PowerShell pour créer un conteneur Azure Storage.
- Dans la phase 2, le script Transact-SQL suivant doit utiliser ce conteneur.

Le script commence par des commandes à supprimer s’il a déjà été exécuté, et est réexécutable.

Le script PowerShell a affiché quelques valeurs nommées à la fin de son exécution. Vous devez modifier le script Transact-SQL pour utiliser ces valeurs. Recherchez les occurrences de **TODO** dans le script Transact-SQL pour trouver les parties du code à modifier.

1. Ouvrez SQL Server Management Studio (ssms.exe).
2. Connectez-vous à votre base de données dans Azure SQL Database.
3. Cliquez pour ouvrir un nouveau volet de requête.
4. Collez le script Transact-SQL suivant dans le volet de requête.
5. Recherchez chaque occurrence de **TODO** dans le script et effectuez les modifications appropriées.
6. Enregistrez le script, puis exécutez-le.

> [!WARNING]
> La valeur de clé SAS générée par le script PowerShell précédent pourrait commencer par un « ? » (point d’interrogation). Lorsque vous utilisez la clé SAP dans le script T-SQL suivant, vous devez *supprimer le point d’interrogation « ? » au début*. Dans le cas contraire, vos efforts peuvent être bloqués par la sécurité.

### <a name="transact-sql-code"></a>Code Transact-SQL

```sql
---- TODO: First, run the earlier PowerShell portion of this two-part code sample.
---- TODO: Second, find every 'TODO' in this Transact-SQL file, and edit each.

---- Transact-SQL code for Event File target on Azure SQL Database.

SET NOCOUNT ON;
GO

----  Step 1.  Establish one little table, and  ---------
----  insert one row of data.

IF EXISTS
    (SELECT * FROM sys.objects
        WHERE type = 'U' and name = 'gmTabEmployee')
BEGIN
    DROP TABLE gmTabEmployee;
END
GO

CREATE TABLE gmTabEmployee
(
    EmployeeGuid         uniqueIdentifier   not null  default newid()  primary key,
    EmployeeId           int                not null  identity(1,1),
    EmployeeKudosCount   int                not null  default 0,
    EmployeeDescr        nvarchar(256)          null
);
GO

INSERT INTO gmTabEmployee ( EmployeeDescr )
    VALUES ( 'Jane Doe' );
GO

------  Step 2.  Create key, and  ------------
------  Create credential (your Azure Storage container must already exist).

IF NOT EXISTS
    (SELECT * FROM sys.symmetric_keys
        WHERE symmetric_key_id = 101)
BEGIN
    CREATE MASTER KEY ENCRYPTION BY PASSWORD = '0C34C960-6621-4682-A123-C7EA08E3FC46' -- Or any newid().
END
GO

IF EXISTS
    (SELECT * FROM sys.database_scoped_credentials
        -- TODO: Assign AzureStorageAccount name, and the associated Container name.
        WHERE name = 'https://gmstorageaccountxevent.blob.core.windows.net/gmcontainerxevent')
BEGIN
    DROP DATABASE SCOPED CREDENTIAL
        -- TODO: Assign AzureStorageAccount name, and the associated Container name.
        [https://gmstorageaccountxevent.blob.core.windows.net/gmcontainerxevent] ;
END
GO

CREATE
    DATABASE SCOPED
    CREDENTIAL
        -- use '.blob.',   and not '.queue.' or '.table.' etc.
        -- TODO: Assign AzureStorageAccount name, and the associated Container name.
        [https://gmstorageaccountxevent.blob.core.windows.net/gmcontainerxevent]
    WITH
        IDENTITY = 'SHARED ACCESS SIGNATURE',  -- "SAS" token.
        -- TODO: Paste in the long SasToken string here for Secret, but exclude any leading '?'.
        SECRET = 'sv=2014-02-14&sr=c&si=gmpolicysastoken&sig=EjAqjo6Nu5xMLEZEkMkLbeF7TD9v1J8DNB2t8gOKTts%3D'
    ;
GO

------  Step 3.  Create (define) an event session.  --------
------  The event session has an event with an action,
------  and a has a target.

IF EXISTS
    (SELECT * from sys.database_event_sessions
        WHERE name = 'gmeventsessionname240b')
BEGIN
    DROP
        EVENT SESSION
            gmeventsessionname240b
        ON DATABASE;
END
GO

CREATE
    EVENT SESSION
        gmeventsessionname240b
    ON DATABASE

    ADD EVENT
        sqlserver.sql_statement_starting
            (
            ACTION (sqlserver.sql_text)
            WHERE statement LIKE 'UPDATE gmTabEmployee%'
            )
    ADD TARGET
        package0.event_file
            (
            -- TODO: Assign AzureStorageAccount name, and the associated Container name.
            -- Also, tweak the .xel file name at end, if you like.
            SET filename =
                'https://gmstorageaccountxevent.blob.core.windows.net/gmcontainerxevent/anyfilenamexel242b.xel'
            )
    WITH
        (MAX_MEMORY = 10 MB,
        MAX_DISPATCH_LATENCY = 3 SECONDS)
    ;
GO

------  Step 4.  Start the event session.  ----------------
------  Issue the SQL Update statements that will be traced.
------  Then stop the session.

------  Note: If the target fails to attach,
------  the session must be stopped and restarted.

ALTER
    EVENT SESSION
        gmeventsessionname240b
    ON DATABASE
    STATE = START;
GO

SELECT 'BEFORE_Updates', EmployeeKudosCount, * FROM gmTabEmployee;

UPDATE gmTabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 2
    WHERE EmployeeDescr = 'Jane Doe';

UPDATE gmTabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 13
    WHERE EmployeeDescr = 'Jane Doe';

SELECT 'AFTER__Updates', EmployeeKudosCount, * FROM gmTabEmployee;
GO

ALTER
    EVENT SESSION
        gmeventsessionname240b
    ON DATABASE
    STATE = STOP;
GO

-------------- Step 5.  Select the results. ----------

SELECT
        *, 'CLICK_NEXT_CELL_TO_BROWSE_ITS_RESULTS!' as [CLICK_NEXT_CELL_TO_BROWSE_ITS_RESULTS],
        CAST(event_data AS XML) AS [event_data_XML]  -- TODO: In ssms.exe results grid, double-click this cell!
    FROM
        sys.fn_xe_file_target_read_file
            (
                -- TODO: Fill in Storage Account name, and the associated Container name.
                -- TODO: The name of the .xel file needs to be an exact match to the files in the storage account Container (You can use Storage Account explorer from the portal to find out the exact file names or you can retrieve the name using the following DMV-query: select target_data from sys.dm_xe_database_session_targets. The 3rd xml-node, "File name", contains the name of the file currently written to.)
                'https://gmstorageaccountxevent.blob.core.windows.net/gmcontainerxevent/anyfilenamexel242b',
                null, null, null
            );
GO

-------------- Step 6.  Clean up. ----------

DROP
    EVENT SESSION
        gmeventsessionname240b
    ON DATABASE;
GO

DROP DATABASE SCOPED CREDENTIAL
    -- TODO: Assign AzureStorageAccount name, and the associated Container name.
    [https://gmstorageaccountxevent.blob.core.windows.net/gmcontainerxevent]
    ;
GO

DROP TABLE gmTabEmployee;
GO

PRINT 'Use PowerShell Remove-AzStorageAccount to delete your Azure Storage account!';
GO
```

Si la liaison avec la cible n’a pas pu se faire au moment de l’exécution, vous devez arrêter et redémarrer la session d’événement :

```sql
ALTER EVENT SESSION ... STATE = STOP;
GO
ALTER EVENT SESSION ... STATE = START;
GO
```

## <a name="output"></a>Output

Quand le script Transact-SQL a fini de s’exécuter, cliquez sur une cellule sous l’en-tête de colonne **event_data_XML**. Un élément **\<event>** s’affiche, avec une instruction UPDATE.

Voici un élément **\<event>** généré pendant le test :

```xml
<event name="sql_statement_starting" package="sqlserver" timestamp="2015-09-22T19:18:45.420Z">
  <data name="state">
    <value>0</value>
    <text>Normal</text>
  </data>
  <data name="line_number">
    <value>5</value>
  </data>
  <data name="offset">
    <value>148</value>
  </data>
  <data name="offset_end">
    <value>368</value>
  </data>
  <data name="statement">
    <value>UPDATE gmTabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 2
    WHERE EmployeeDescr = 'Jane Doe'</value>
  </data>
  <action name="sql_text" package="sqlserver">
    <value>

SELECT 'BEFORE_Updates', EmployeeKudosCount, * FROM gmTabEmployee;

UPDATE gmTabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 2
    WHERE EmployeeDescr = 'Jane Doe';

UPDATE gmTabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 13
    WHERE EmployeeDescr = 'Jane Doe';

SELECT 'AFTER__Updates', EmployeeKudosCount, * FROM gmTabEmployee;
</value>
  </action>
</event>
```

Le script Transact-SQL précédent a utilisé la fonction système suivante pour lire le fichier d’événement :

- [sys.fn_xe_file_target_read_file (Transact-SQL)](/sql/relational-databases/system-functions/sys-fn-xe-file-target-read-file-transact-sql)

Une explication des options avancées de l’affichage des données d’événements étendus est disponible dans :

- [Affichage avancée des données cibles à partir d’événements étendus](/sql/relational-databases/extended-events/advanced-viewing-of-target-data-from-extended-events-in-sql-server)

## <a name="converting-the-code-sample-to-run-on-sql-server"></a>Conversion de l’exemple de code pour l’exécuter sur SQL Server

Vous voulez maintenant exécuter l’exemple de code Transact-SQL précédent sur Microsoft SQL Server.

- Par souci de simplicité, vous allez remplacer entièrement le conteneur Stockage Azure par un simple fichier tel que *C:\myeventdata.xel*. Le fichier doit être stocké sur le disque dur local de l’ordinateur qui héberge SQL Server.
- Vous n’avez pas besoin d’instructions Transact-SQL de type **CREATE MASTER KEY** et **CREATE CREDENTIAL**.
- Dans l’instruction **CREATE EVENT SESSION**, dans sa clause **ADD TARGET**, vous devez remplacer la valeur Http affectée à **filename=** par une chaîne de chemin d’accès complet comme *C:\myfile.xel*.
  
  - Vous n’avez pas besoin de compte Azure Storage.

## <a name="more-information"></a>Informations complémentaires

Pour plus d’informations sur les comptes et les conteneurs du service Azure Storage, consultez :

- [Utilisation du stockage d’objets blob à partir de .NET](../../storage/blobs/storage-quickstart-blobs-dotnet.md)
- [Désignation et référencement des conteneurs, des objets BLOB et des métadonnées](/rest/api/storageservices/Naming-and-Referencing-Containers--Blobs--and-Metadata)
- [Utilisation du conteneur racine](/rest/api/storageservices/Working-with-the-Root-Container)
- [Leçon 1 : Créer une stratégie d’accès stockée et une signature d’accès partagé sur un conteneur Azure](/sql/relational-databases/tutorial-use-azure-blob-storage-service-with-sql-server-2016#1---create-stored-access-policy-and-shared-access-storage)
  - [Leçon 2 : Créer des informations d’identification SQL Server à l’aide d’une signature d’accès partagé](/sql/relational-databases/tutorial-use-azure-blob-storage-service-with-sql-server-2016#2---create-a-sql-server-credential-using-a-shared-access-signature)
- [Événements étendus pour Microsoft SQL Server](/sql/relational-databases/extended-events/extended-events)