---
title: Résoudre les problèmes liés aux connecteurs dans Azure Data Factory
description: Découvrez comment résoudre les problèmes liés aux connecteurs dans Azure Data Factory.
services: data-factory
author: linda33wj
ms.service: data-factory
ms.topic: troubleshooting
ms.date: 12/09/2020
ms.author: jingwang
ms.reviewer: craigg
ms.custom: has-adal-ref
ms.openlocfilehash: a7a81a742922d45be965c7f73e3cb910d0ef989a
ms.sourcegitcommit: 6172a6ae13d7062a0a5e00ff411fd363b5c38597
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/11/2020
ms.locfileid: "97109286"
---
# <a name="troubleshoot-azure-data-factory-connectors"></a>Résoudre les problèmes liés aux connecteurs dans Azure Data Factory

[!INCLUDE[appliesto-adf-asa-md](includes/appliesto-adf-asa-md.md)]

Cet article présente des méthodes couramment employées pour résoudre les problèmes liés aux connecteurs dans Azure Data Factory.
  
## <a name="azure-blob-storage"></a>Stockage Blob Azure

### <a name="error-code--azurebloboperationfailed"></a>Code d’erreur :  AzureBlobOperationFailed

- **Message** : `Blob operation Failed. ContainerName: %containerName;, path: %path;.`

- **Cause** : Problème d’accès à l’opération de stockage d’objets Blob.

- **Recommandation** :  Pour plus d’informations, consultez l’erreur. Reportez-vous au document d’aide sur les blobs : https://docs.microsoft.com/rest/api/storageservices/blob-service-error-codes. Contactez l’équipe de stockage si vous avez besoin d’aide.


### <a name="error-code--azureblobservicenotreturnexpecteddatalength"></a>Code d’erreur :  AzureBlobServiceNotReturnExpectedDataLength

- **Message** : `Error occurred when trying to fetch the blob '%name;'. This could be a transient issue and you may rerun the job. If it fails again continuously, contact customer support.`


### <a name="error-code--azureblobnotsupportmultiplefilesintosingleblob"></a>Code d’erreur :  AzureBlobNotSupportMultipleFilesIntoSingleBlob

- **Message** : `Transferring multiple files into a single Blob is not supported. Currently only single file source is supported.`


### <a name="error-code--azurestorageoperationfailedconcurrentwrite"></a>Code d’erreur :  AzureStorageOperationFailedConcurrentWrite

- **Message** : `Error occurred when trying to upload a file. It's possible because you have multiple concurrent copy activities runs writing to the same file '%name;'. Check your ADF configuration.`


### <a name="invalid-property-during-copy-activity"></a>Propriété non valide durant l’activité de copie

- **Message** : `Copy activity <Activity Name> has an invalid "source" property. The source type is not compatible with the dataset <Dataset Name> and its linked service <Linked Service Name>. Please verify your input against.`

- **Cause** : Le type défini dans le jeu de données n’est pas cohérent avec le type de source/récepteur défini dans l’activité de copie.

- **Résolution** : Modifiez la définition JSON du jeu de données ou du pipeline pour rendre les types cohérents et réexécutez le déploiement.


## <a name="azure-cosmos-db"></a>Azure Cosmos DB

### <a name="error-message-request-size-is-too-large"></a>Message d’erreur : La taille de la demande est trop grande

- **Symptômes** : Vous copiez des données dans Azure Cosmos DB avec la taille du lot d’écriture par défaut et vous rencontrez l’erreur *« **La taille de la demande est trop grande** »* .

- **Cause** : Cosmos DB limite la taille d’une demande unique à 2 Mo. La formule est la suivante : Taille de la demande = Taille de document unique * Taille du lot d’écriture. Si la taille de votre document est importante, le comportement par défaut entraîne une trop grande taille de demande. Vous pouvez ajuster la taille du lot d’écriture.

- **Résolution** : Dans le récepteur de l’activité de copie, réduisez la taille du lot d’écriture « Write batch size » (la valeur par défaut est 10000).

### <a name="error-message-unique-index-constraint-violation"></a>Message d’erreur : Violation de contrainte d’index unique

- **Symptômes** : Lorsque vous copiez des données dans Cosmos DB, vous rencontrez l’erreur suivante :

    ```
    Message=Partition range id 0 | Failed to import mini-batch. 
    Exception was Message: {"Errors":["Encountered exception while executing function. Exception = Error: {\"Errors\":[\"Unique index constraint violation.\"]}... 
    ```

- **Cause** : Il existe deux causes possibles :

    - Si vous utilisez **Insérer** comme comportement d’écriture, cette erreur signifie que vos données sources ont des lignes/objets avec le même ID.

    - Si vous utilisez **Upsert** comme comportement d’écriture et que vous définissez une autre clé unique sur le conteneur, cette erreur signifie que vos données sources ont des lignes/objets avec des ID différents, mais une même valeur pour la clé unique définie.

- **Résolution** : 

    - Pour cause1, définissez **Upsert** comme comportement d’écriture.
    - Pour cause2, assurez-vous que chaque document a une valeur différente pour la clé unique définie.

### <a name="error-message-request-rate-is-large"></a>Message d’erreur : Le taux de demandes est élevé

- **Symptômes** : Lorsque vous copiez des données dans Cosmos DB, vous rencontrez l’erreur suivante :

    ```
    Type=Microsoft.Azure.Documents.DocumentClientException,
    Message=Message: {"Errors":["Request rate is large"]}
    ```

- **Cause** : Les unités de demande utilisées sont plus volumineuses que l’unité de requête disponible configurée dans Cosmos DB. Découvrez [ici](../cosmos-db/request-units.md#request-unit-considerations) comment Cosmos DB calcule l’unité de requête.

- **Résolution** : Il existe deux solutions :

    1. **Augmentez l’unité de requête du conteneur** dans Cosmos DB, ce qui améliorera les performances de l’activité de copie, mais engendrera des coûts plus élevés dans Cosmos DB. 

    2. Réduisez la valeur **writeBatchSize** (à 1000, par exemple) et spécifiez une valeur plus petite pour **parallelCopies**, telle que 1, ce qui réduira les performances d’exécution de la copie mais n’entraînera pas de frais supplémentaires dans Cosmos DB.

### <a name="column-missing-in-column-mapping"></a>Colonne manquante dans le mappage de colonnes

- **Symptômes** : Lorsque vous importez un schéma pour Cosmos DB pour le mappage de colonnes, certaines colonnes sont manquantes. 

- **Cause** : ADF déduit le schéma des 10 premiers documents Cosmos DB. Si certaines colonnes/propriétés n’ont pas de valeur dans ces documents, elles ne sont pas détectées par ADF et ne s’affichent donc pas.

- **Résolution** : Vous pouvez ajuster la requête comme ci-dessous pour forcer l’affichage de la colonne dans le jeu de résultats avec une valeur vide : (supposons que la colonne « impossible » est manquante dans les 10 premiers documents). Vous pouvez également ajouter manuellement la colonne pour le mappage.

    ```sql
    select c.company, c.category, c.comments, (c.impossible??'') as impossible from c
    ```

### <a name="error-message-the-guidrepresentation-for-the-reader-is-csharplegacy"></a>Message d’erreur : Le GuidRepresentation pour le lecteur est CSharpLegacy

- **Symptômes** : En copiant des données à partir de Cosmos DB MongoAPI/MongoDB avec le champ UUID, vous rencontrez l’erreur suivante :

    ```
    Failed to read data via MongoDB client.,
    Source=Microsoft.DataTransfer.Runtime.MongoDbV2Connector,Type=System.FormatException,
    Message=The GuidRepresentation for the reader is CSharpLegacy which requires the binary sub type to be UuidLegacy not UuidStandard.,Source=MongoDB.Bson,’“,
    ```

- **Cause** : Il existe deux façons de représenter UUID dans BSON : UuidStandard et UuidLegacy. Par défaut, UuidLegacy est utilisé pour lire les données. Vous rencontrerez cette erreur si vos données UUID dans MongoDB sont UuidStandard.

- **Résolution** : Dans la chaîne de connexion MongoDB, ajoutez l’option « **uuidRepresentation=standard** ». Pour plus d’informations, consultez [Chaîne de connexion MongoDB](connector-mongodb.md#linked-service-properties).
            

## <a name="azure-data-lake-storage-gen2"></a>Azure Data Lake Storage Gen2

### <a name="error-code--adlsgen2operationfailed"></a>Code d’erreur :  AdlsGen2OperationFailed

- **Message** : `ADLS Gen2 operation failed for: %adlsGen2Message;.%exceptionData;.`

- **Cause** : ADLS Gen2 génère l’erreur indiquant que l’opération a échoué.

- **Recommandation** :  Consultez le message d’erreur détaillé généré par ADLS Gen2. Si l’erreur est causée par une défaillance temporaire, réessayez. Si vous avez besoin d’aide supplémentaire, contactez le support du stockage Azure et fournissez l’ID de la requête dans le message d’erreur.

- **Cause** : Lorsque le message d’erreur contient « Forbidden », le principal du service ou l’identité gérée que vous utilisez ne dispose peut-être pas des autorisations suffisantes pour accéder à ADLS Gen2.

- **Recommandation** :  Reportez-vous au document d’aide : https://docs.microsoft.com/azure/data-factory/connector-azure-data-lake-storage#service-principal-authentication.

- **Cause** : Lorsque le message d’erreur contient « InternalServerError », l’erreur est retournée par ADLS Gen2.

- **Recommandation** :  Cela peut être dû à un échec temporaire, réessayez. Si le problème persiste, contactez le support du stockage Azure et fournissez l’ID de la requête dans le message d’erreur.


### <a name="error-code--adlsgen2invalidurl"></a>Code d’erreur :  AdlsGen2InvalidUrl

- **Message** : `Invalid url '%url;' provided, expecting http[s]://<accountname>.dfs.core.windows.net.`


### <a name="error-code--adlsgen2invalidfolderpath"></a>Code d’erreur :  AdlsGen2InvalidFolderPath

- **Message** : `The folder path is not specified. Cannot locate the file '%name;' under the ADLS Gen2 account directly. Please specify the folder path instead.`


### <a name="error-code--adlsgen2operationfailedconcurrentwrite"></a>Code d’erreur :  AdlsGen2OperationFailedConcurrentWrite

- **Message** : `Error occurred when trying to upload a file. It's possible because you have multiple concurrent copy activities runs writing to the same file '%name;'. Check your ADF configuration.`


### <a name="error-code-adlsgen2timeouterror"></a>Code d’erreur : AdlsGen2TimeoutError

- **Message** : `Request to ADLS Gen2 account '%account;' met timeout error. It is mostly caused by the poor network between the Self-hosted IR machine and the ADLS Gen2 account. Check the network to resolve such error.`


### <a name="request-to-adls-gen2-account-met-timeout-error"></a>La demande au compte ADLS Gen2 a rencontré une erreur d’expiration de délai

- **Message** : Code d’erreur = `UserErrorFailedBlobFSOperation`, Message d’erreur = `BlobFS operation failed for: A task was canceled`.

- **Cause** : Le problème est dû à une erreur d’expiration du délai du récepteur ADLS Gen2, ce qui se produit essentiellement sur la machine de l’IR auto-hébergé.

- **Recommandation** : 

    1. Placez votre machine de l’IR auto-hébergé et le compte ADLS Gen2 cible dans la même région, si possible. Cela peut éviter une erreur d’expiration de délai aléatoire et améliorer les performances.

    1. Vérifiez s’il existe un paramètre de réseau spécial comme ExpressRoute et assurez-vous que le réseau dispose d’une bande passante suffisante. Il est recommandé de réduire le paramètre des tâches simultanées de l’IR auto-hébergé quand la bande passante globale est faible, ce qui peut éviter la concurrence des ressources réseau entre les différentes tâches simultanées.

    1. Utilisez une taille de bloc plus petite pour la copie non binaire afin d’atténuer les effets d’une telle erreur d’expiration de délai si la taille de fichier est modérée ou petite. Consultez [Stockage Blob – Put Block](https://docs.microsoft.com/rest/api/storageservices/put-block).

       Pour spécifier la taille de bloc personnalisée, vous pouvez modifier la propriété dans l’éditeur .json :
    ```
    "sink": {
        "type": "DelimitedTextSink",
        "storeSettings": {
            "type": "AzureBlobFSWriteSettings",
            "blockSizeInMB": 8
        }
    }
    ```


## <a name="azure-data-lake-storage-gen1"></a>Azure Data Lake Storage Gen1

### <a name="error-message-the-underlying-connection-was-closed-could-not-establish-trust-relationship-for-the-ssltls-secure-channel"></a>Message d’erreur : Le serveur a clos la connexion sous-jacente : Impossible d’établir une relation de confiance pour le canal sécurisé SSL/TLS.

- **Symptômes** : L’activité de copie échoue avec l’erreur suivante : 

    ```
    Message: Failure happened on 'Sink' side. ErrorCode=UserErrorFailedFileOperation,'Type=Microsoft.DataTransfer.Common.Shared.HybridDeliveryException,Message=Upload file failed at path STAGING/PLANT/INDIARENEWABLE/LiveData/2020/01/14\\20200114-0701-oem_gibtvl_mannur_data_10min.csv.,Source=Microsoft.DataTransfer.ClientLibrary,''Type=System.Net.WebException,Message=The underlying connection was closed: Could not establish trust relationship for the SSL/TLS secure channel.,Source=System,''Type=System.Security.Authentication.AuthenticationException,Message=The remote certificate is invalid according to the validation procedure.,Source=System,'.
    ```

- **Cause** : La validation du certificat a échoué lors de la négociation TLS.

- **Résolution** : Solution de contournement : Utilisez la copie intermédiaire afin d’ignorer la validation TLS pour ADLS Gen1. Vous devez reproduire ce problème et collecter la trace netmon, puis demander à votre équipe réseau de vérifier la configuration du réseau local.

    ![Résoudre les problèmes liés à ADLS Gen1](./media/connector-troubleshoot-guide/adls-troubleshoot.png)


### <a name="error-message-the-remote-server-returned-an-error-403-forbidden"></a>Message d’erreur : Le serveur distant a retourné une erreur : (403) Interdit

- **Symptômes** : L’activité de copie échoue avec l’erreur suivante : 

    ```
    Message: The remote server returned an error: (403) Forbidden.. 
    Response details: {"RemoteException":{"exception":"AccessControlException""message":"CREATE failed with error 0x83090aa2 (Forbidden. ACL verification failed. Either the resource does not exist or the user is not authorized to perform the requested operation.)....
    ```

- **Cause** : L’une des causes possibles est que le principal du service ou l’identité managée que vous utilisez n’a pas l’autorisation d’accéder au dossier/fichier donné.

- **Résolution** : Accordez des autorisations appropriées sur tous les dossiers et sous-dossiers que vous devez copier. Consultez [ce document](connector-azure-data-lake-store.md#linked-service-properties).

### <a name="error-message-failed-to-get-access-token-by-using-service-principal-adal-error-service_unavailable"></a>Message d’erreur : Échec d’obtention du jeton d’accès à l’aide du principal du service. Erreur ADAL : service_unavailable

- **Symptômes** : L’activité de copie échoue avec l’erreur suivante :

    ```
    Failed to get access token by using service principal. 
    ADAL Error: service_unavailable, The remote server returned an error: (503) Server Unavailable.
    ```

- **Cause** : Lorsque le service d’émission de jeton de sécurité (STS) détenu par Azure Active Directory n’est pas disponible, c.-à-d. qu’il est trop occupé pour gérer les demandes, il renvoie une erreur HTTP 503. 

- **Résolution** : Réexécutez l’activité de copie au bout de quelques minutes.
                  

## <a name="azure-synapse-analyticsazure-sql-databasesql-server"></a>Azure Synapse Analytics/Azure SQL Database/SQL Server

### <a name="error-code--sqlfailedtoconnect"></a>Code d’erreur :  SqlFailedToConnect

- **Message** : `Cannot connect to SQL Database: '%server;', Database: '%database;', User: '%user;'. Check the linked service configuration is correct, and make sure the SQL Database firewall allows the integration runtime to access.`

- **Cause** : Si le message d’erreur contient « SqlException »,SQL Database génère l’erreur indiquant qu’une opération spécifique a échoué.

- **Recommandation** :  Pour plus d’informations, effectuez une recherche par code d’erreur SQL dans ce document de référence : https://docs.microsoft.com/sql/relational-databases/errors-events/database-engine-events-and-errors. Si vous avez besoin d’aide supplémentaire, contactez le support Azure SQL.

- **Cause** : Si le message d’erreur contient « Le client avec l’adresse IP "…" n’est pas autorisé à accéder au serveur » et que vous essayez de vous connecter à Azure SQL Database, cela est généralement causé par un problème de pare-feu Azure SQL Database.

- **Recommandation** :  Dans la configuration du pare-feu du serveur SQL logique, activez l’option « Autoriser les services et les ressources Azure à accéder à ce serveur ». Doc de référence : https://docs.microsoft.com/azure/sql-database/sql-database-firewall-configure.


### <a name="error-code--sqloperationfailed"></a>Code d’erreur :  SqlOperationFailed

- **Message** : `A database operation failed. Please search error to get more details.`

- **Cause** : Si le message d’erreur contient « SqlException »,SQL Database génère l’erreur indiquant qu’une opération spécifique a échoué.

- **Recommandation** :  Si l’erreur SQL n’est pas claire, essayez de modifier la base de données au niveau de compatibilité « 150 » le plus récent. Cela peut lever des erreurs SQL de dernière version. Reportez-vous au [document détaillé](/sql/t-sql/statements/alter-database-transact-sql-compatibility-level#backwardCompat).

    Pour détecter des problèmes SQL, effectuez une recherche par code d’erreur SQL dans ce document de référence pour plus d’informations : https://docs.microsoft.com/sql/relational-databases/errors-events/database-engine-events-and-errors. Si vous avez besoin d’aide supplémentaire, contactez le support Azure SQL.

- **Cause** : Si le message d’erreur contient « PdwManagedToNativeInteropException », cela est généralement dû à une incompatibilité entre les tailles de colonne source et récepteur.

- **Recommandation** :  Vérifiez la taille des colonnes source et récepteur. Si vous avez besoin d’aide supplémentaire, contactez le support Azure SQL.

- **Cause** : Si le message d’erreur contient « InvalidOperationException », cela est généralement dû à des données d’entrée non valides.

- **Recommandation** :  Pour identifier la ligne qui rencontre le problème, activez la fonctionnalité de tolérance de panne sur l’activité de copie, qui peut rediriger la ou les lignes problématiques vers le stockage pour une investigation plus poussée. Doc de référence : https://docs.microsoft.com/azure/data-factory/copy-activity-fault-tolerance.



### <a name="error-code--sqlunauthorizedaccess"></a>Code d’erreur :  SqlUnauthorizedAccess

- **Message** : `Cannot connect to '%connectorName;'. Detail Message: '%message;'`

- **Cause** : Les informations d’identification sont incorrectes ou le compte de connexion ne peut pas accéder à SQL Database.

- **Recommandation** :  Vérifiez que le compte de connexion dispose des autorisations suffisantes pour accéder à SQL Database.


### <a name="error-code--sqlopenconnectiontimeout"></a>Code d’erreur :  SqlOpenConnectionTimeout

- **Message** : `Open connection to database timeout after '%timeoutValue;' seconds.`

- **Cause** : Il peut s’agir d’une défaillance temporaire de SQL Database.

- **Recommandation** :  Réessayez de mettre à jour la chaîne de connexion de service lié avec une valeur de délai d’attente de connexion plus importante.


### <a name="error-code--sqlautocreatetabletypemapfailed"></a>Code d’erreur :  SqlAutoCreateTableTypeMapFailed

- **Message** : `Type '%dataType;' in source side cannot be mapped to a type that supported by sink side(column name:'%columnName;') in autocreate table.`

- **Cause** : La création automatique de la table ne peut pas répondre aux exigences de la source.

- **Recommandation** :  Mettez à jour le type de colonne en « mappings », ou créez manuellement la table du récepteur dans le serveur cible.


### <a name="error-code--sqldatatypenotsupported"></a>Code d’erreur :  SqlDataTypeNotSupported

- **Message** : `A database operation failed. Check the SQL errors.`

- **Cause** : Si le problème se produit sur la source SQL et que l’erreur est liée au dépassement de SqlDateTime, la valeur des données est supérieure à la plage du type logique (de 1/1/1753 12:00:00 à 12/31/9999 11:59:59 PM).

- **Recommandation** :  Effectuez un forçage de type en chaîne dans la requête SQL source ou, dans le mappage de colonne d’activité de copie, modifiez le type de colonne en « Chaîne ».

- **Cause** : Si le problème se produit sur un récepteur SQL et que l’erreur est liée au dépassement de SqlDateTime, la valeur des données est supérieure à la plage autorisée dans la table sink.

- **Recommandation** :  Mettez à jour le type de colonne correspondant en type « datetime2 » dans la table du récepteur.


### <a name="error-code--sqlinvaliddbstoredprocedure"></a>Code d’erreur :  SqlInvalidDbStoredProcedure

- **Message** : `The specified Stored Procedure is not valid. It could be caused by that the stored procedure doesn't return any data. Invalid Stored Procedure script: '%scriptName;'.`

- **Cause** : La procédure stockée spécifiée n’est pas valide. Cela peut être dû au fait que la procédure stockée ne retourne pas de données.

- **Recommandation** :  Validez la procédure stockée à l’aide des outils SQL. Assurez-vous que la procédure stockée peut renvoyer des données.


### <a name="error-code--sqlinvaliddbquerystring"></a>Code d’erreur :  SqlInvalidDbQueryString

- **Message** : `The specified SQL Query is not valid. It could be caused by that the query doesn't return any data. Invalid query: '%query;'`

- **Cause** : La requête SQL spécifiée n’est pas valide. Cela peut être dû au fait que la requête ne renvoie aucune donnée

- **Recommandation** :  Validez la requête SQL à l’aide des outils SQL. Assurez-vous que la requête peut renvoyer des données.


### <a name="error-code--sqlinvalidcolumnname"></a>Code d’erreur :  SqlInvalidColumnName

- **Message** : `Column '%column;' does not exist in the table '%tableName;', ServerName: '%serverName;', DatabaseName: '%dbName;'.`

- **Cause** : Impossible de trouver la colonne. La configuration est peut-être incorrecte.

- **Recommandation** :  Vérifiez la colonne dans la requête, « structure » dans le jeu de données et « mappings » dans l’activité.


### <a name="error-code--sqlcolumnnamemismatchbycasesensitive"></a>Code d’erreur :  SqlColumnNameMismatchByCaseSensitive

- **Message** : `Column '%column;' in DataSet '%dataSetName;' cannot be found in physical SQL Database. Column matching is case-sensitive. Column '%columnInTable;' appears similar. Check the DataSet(s) configuration to proceed further.`


### <a name="error-code--sqlbatchwritetimeout"></a>Code d’erreur :  SqlBatchWriteTimeout

- **Message** : `Timeouts in SQL write operation.`

- **Cause** : Il peut s’agir d’une défaillance temporaire de SQL Database.

- **Recommandation** :  Réessayez. Si le problème persiste, contactez le support Azure SQL.


### <a name="error-code--sqlbatchwritetransactionfailed"></a>Code d’erreur :  SqlBatchWriteTransactionFailed

- **Message** : `SQL transaction commits failed`

- **Cause** : Si les détails de l’exception indiquent constamment une expiration du délai de la transaction, la latence du réseau entre le runtime d’intégration et la base de données est supérieure au seuil par défaut de 30 secondes.

- **Recommandation** :  Mettez à jour la chaîne de connexion du service lié à SQL avec la valeur « connection timeout » égale à 120 ou supérieure, puis réexécutez l’activité.

- **Cause** : Si les détails de l’exception indiquent par intermittence que sqlconnection est rompu, il peut s’agir d’une défaillance temporaire du réseau ou d’un problème du côté de SQL Database.

- **Recommandation** :  Réessayez l’activité et passez en revue les métriques du côté de SQL Database.


### <a name="error-code--sqlbulkcopyinvalidcolumnlength"></a>Code d’erreur :  SqlBulkCopyInvalidColumnLength

- **Message** : `SQL Bulk Copy failed due to receive an invalid column length from the bcp client.`

- **Cause** : La copie en bloc SQL a échoué à cause de la réception d’une longueur de colonne non valide du client bcp.

- **Recommandation** :  Pour identifier la ligne qui rencontre le problème, activez la fonctionnalité de tolérance de panne sur l’activité de copie, qui peut rediriger la ou les lignes problématiques vers le stockage pour une investigation plus poussée. Doc de référence : https://docs.microsoft.com/azure/data-factory/copy-activity-fault-tolerance.


### <a name="error-code--sqlconnectionisclosed"></a>Code d’erreur :  SqlConnectionIsClosed

- **Message** : `The connection is closed by SQL Database.`

- **Cause** : La connexion SQL est fermée par SQL Database lorsque le nombre d’exécutions simultanées est élevé et que le serveur termine la connexion.

- **Recommandation** :  Le serveur distant a fermé la connexion SQL. Reprise Si le problème persiste, contactez le support Azure SQL.


### <a name="error-code--sqlcreatetablefailedunsupportedtype"></a>Code d’erreur :  SqlCreateTableFailedUnsupportedType

- **Message** : `Type '%type;' in source side cannot be mapped to a type that supported by sink side(column name:'%name;') in autocreate table.`


### <a name="error-message-conversion-failed-when-converting-from-a-character-string-to-uniqueidentifier"></a>Message d’erreur : Échec lors de la conversion d’une chaîne de caractères en valeur de type uniqueidentifier

- **Symptômes** : Lorsque vous copiez des données d’une source de données tabulaire (telle que SQL Server) vers Azure Synapse Analytics à l’aide de la copie intermédiaire et de PolyBase, vous rencontrez l’erreur suivante :

    ```
    ErrorCode=FailedDbOperation,Type=Microsoft.DataTransfer.Common.Shared.HybridDeliveryException,
    Message=Error happened when loading data into Azure Synapse Analytics.,
    Source=Microsoft.DataTransfer.ClientLibrary,Type=System.Data.SqlClient.SqlException,
    Message=Conversion failed when converting from a character string to uniqueidentifier...
    ```

- **Cause** : Azure Synapse Analytics PolyBase ne peut pas convertir une chaîne vide en GUID.

- **Résolution** : Dans le récepteur de l’activité de copie, sous les paramètres de Polybase, affectez la valeur false à « **Utiliser l’option de type par défaut** ».


### <a name="error-message-expected-data-type-decimalxx-offending-value"></a>Message d’erreur : Type de données attendu : DECIMAL(x,x), valeur incriminée

- **Symptômes** : Lorsque vous copiez des données d’une source de données tabulaire (telle que SQL Server) vers Azure Synapse Analytics à l’aide de la copie intermédiaire et de PolyBase, vous rencontrez l’erreur suivante :

    ```
    ErrorCode=FailedDbOperation,Type=Microsoft.DataTransfer.Common.Shared.HybridDeliveryException,
    Message=Error happened when loading data into Azure Synapse Analytics.,
    Source=Microsoft.DataTransfer.ClientLibrary,Type=System.Data.SqlClient.SqlException,
    Message=Query aborted-- the maximum reject threshold (0 rows) was reached while reading from an external source: 1 rows rejected out of total 415 rows processed. (/file_name.txt) 
    Column ordinal: 18, Expected data type: DECIMAL(x,x), Offending value:..
    ```

- **Cause** : Azure Synapse Analytics Polybase ne peut pas insérer une chaîne vide (valeur Null) dans une colonne décimale.

- **Résolution** : Dans le récepteur de l’activité de copie, sous les paramètres de Polybase, affectez la valeur false à « **Utiliser l’option de type par défaut** ».


### <a name="error-message-java-exception-message-hdfsbridgecreaterecordreader"></a>Message d’erreur : Message d’exception Java : HdfsBridge::CreateRecordReader

- **Symptômes** : Vous copiez des données dans Azure Synapse Analytics à l’aide de PolyBase et vous rencontrez l’erreur suivante :

    ```
    Message=110802;An internal DMS error occurred that caused this operation to fail. 
    Details: Exception: Microsoft.SqlServer.DataWarehouse.DataMovement.Common.ExternalAccess.HdfsAccessException, 
    Message: Java exception raised on call to HdfsBridge_CreateRecordReader. 
    Java exception message:HdfsBridge::CreateRecordReader - Unexpected error encountered creating the record reader.: Error [HdfsBridge::CreateRecordReader - Unexpected error encountered creating the record reader.] occurred while accessing external file.....
    ```

- **Cause** : La cause possible est que le schéma (largeur totale de colonne) est trop grand (plus de 1 Mo). Vérifiez le schéma de la table Azure Synapse Analytics cible en ajoutant la taille de toutes les colonnes :

    - Int -> 4 octets
    - Bigint -> 8 octets
    - Varchar(n), char(n), binary(n), varbinary(n) -> n octets
    - Nvarchar(n), nchar(n) -> n*2 octets
    - Date -> 6 octets
    - Datetime/(2), smalldatetime -> 16 octets
    - Datetimeoffset -> 20 octets
    - Decimal -> 19 octets
    - Float -> 8 octets
    - Money -> 8 octets
    - Smallmoney -> 4 octets
    - Real -> 4 octets
    - Smallint -> 2 octets
    - Time -> 12 octets
    - Tinyint -> 1 octet

- **Résolution** : Réduisez la largeur de colonne à moins de 1 Mo

- Ou utilisez l’approche d’insertion en bloc en désactivant Polybase


### <a name="error-message-the-condition-specified-using-http-conditional-headers-is-not-met"></a>Message d’erreur : La condition spécifiée avec un ou plusieurs en-têtes conditionnels HTTP n’est pas remplie

- **Symptômes** : Vous utilisez la requête SQL pour extraire des données d’Azure Synapse Analytics et vous pouvez rencontrez l’erreur suivante :

    ```
    ...StorageException: The condition specified using HTTP conditional header(s) is not met...
    ```

- **Cause** : Azure Synapse Analytics a rencontré un problème en interrogeant la table externe dans Stockage Azure.

- **Résolution** : Exécutez la même requête dans SSMS et vérifiez si vous voyez le même résultat. Si c’est le cas, ouvrez un ticket de support pour Azure Synapse Analytics et indiquez le nom de votre base de données et de votre serveur Azure Synapse Analytics à des fins de dépannage.
            

### <a name="low-performance-when-load-data-into-azure-sql"></a>Faible niveau de performance pendant le chargement de données dans Azure SQL

- **Symptômes** : La copie de données dans Azure SQL devient lente.

- **Cause** : La cause racine du problème est principalement provoquée par le goulot d’étranglement du côté d’Azure SQL. Voici quelques causes possibles :

    1. Le niveau Azure DB n’est pas suffisamment élevé.

    1. L’utilisation de DTU Azure DB est proche de 100 %. Vous pouvez [superviser le niveau de performance](https://docs.microsoft.com/azure/azure-sql/database/monitor-tune-overview) et envisager une mise à niveau pour le niveau de base de données.

    1. Les index ne sont pas bien définis. Supprimez tous les index avant le chargement des données, puis recréez-les une fois le chargement terminé.

    1. La valeur de WriteBatchSize n’est pas suffisante pour la taille de ligne de schéma. Essayez d’attribuer une valeur supérieure à la propriété pour résoudre le problème.

    1. Une procédure stockée est utilisée à la place de l’insertion en bloc, ce qui est censé amoindrir le niveau de performance. 

- **Résolution** : Reportez-vous au guide de résolution des problèmes (TSG) relatif aux [performances de l’activité de copie](https://docs.microsoft.com/azure/data-factory/copy-activity-performance-troubleshooting)


### <a name="performance-tier-is-low-and-leads-to-copy-failure"></a>Le niveau de performance est faible et entraîne un échec de la copie

- **Symptômes** : Le message d’erreur ci-dessous s’est affiché pendant la copie des données dans Azure SQL : `Database operation failed. Error message from database execution : ExecuteNonQuery requires an open and available Connection. The connection's current state is closed.`

- **Cause** : Azure SQL s1 étant utilisé, les limites d’E/S sont atteintes en pareil cas.

- **Résolution** : Mettez à niveau le niveau de performance d’Azure SQL pour corriger le problème. 


### <a name="sql-table-cannot-be-found"></a>Table SQL introuvable 

- **Symptômes** : Une erreur s’est produite pendant la copie des données de Hybrid vers la table SQL Server locale : `Cannot find the object "dbo.Contoso" because it does not exist or you do not have permissions.`

- **Cause** : Le compte SQL actuel ne dispose pas d’autorisations suffisantes pour exécuter les demandes émises par SqlBulkCopy.WriteToServer (.NET).

- **Résolution** : Passez à un compte SQL dotée de privilèges supérieurs.


### <a name="string-or-binary-data-would-be-truncated"></a>Les données de type chaîne ou binaire sont tronquées

- **Symptômes** : Une erreur s’est produite pendant la copie de données dans une table SQL Server Azure/locale : 

- **Cause** : La définition du schéma de table Cx Sql présente une ou plusieurs colonnes dont la longueur est inférieure à celle attendue.

- **Résolution** : Essayez les étapes suivantes pour corriger le problème :

    1. Appliquez la [tolérance de panne](https://docs.microsoft.com/azure/data-factory/copy-activity-fault-tolerance), en particulier « redirectIncompatibleRowSettings », pour identifier les lignes qui présentent le problème.

    1. Examinez bien les données redirigées par rapport à la longueur de colonne du schéma de table SQL pour déterminer la ou les colonnes qui doivent être mises à jour.

    1. Mettez à jour le schéma de table en conséquence.


## <a name="delimited-text-format"></a>Format de texte délimité

### <a name="error-code--delimitedtextcolumnnamenotallownull"></a>Code d’erreur :  DelimitedTextColumnNameNotAllowNull

- **Message** : `The name of column index %index; is empty. Make sure column name is properly specified in the header row.`

- **Cause** : Quand vous définissez « firstRowAsHeader » dans l’activité, la première ligne est utilisée comme nom de colonne. Cette erreur signifie que la première ligne contient une valeur vide. Exemple : « ColonneA, ColonneB ».

- **Recommandation** :  Vérifiez la première ligne et corrigez la valeur s’il s’agit d’une valeur vide.


### <a name="error-code--delimitedtextmorecolumnsthandefined"></a>Code d’erreur :  DelimitedTextMoreColumnsThanDefined

- **Message** : `Error found when processing '%function;' source '%name;' with row number %rowCount;: found more columns than expected column count: %columnCount;.`

- **Cause** : Le nombre de colonnes de la ligne problématique est plus important que le nombre de colonnes de la première ligne. Cela peut être dû à un problème de données ou à un délimiteur de colonne ou des paramètres de caractère de citation incorrects.

- **Recommandation** :  Récupérez le nombre de lignes dans le message d’erreur, vérifiez la colonne de la ligne et corrigez les données.

- **Cause** : Si le nombre de colonnes attendu est « 1 » dans le message d’erreur, vous avez peut-être spécifié des paramètres de compression ou de format incorrects. Par conséquent, ADF a mal analysé vos fichiers.

- **Recommandation** :  Vérifiez les paramètres de mise en forme pour vous assurer qu’ils correspondent à vos fichiers sources.

- **Cause** : Si votre source est un dossier, il est possible que les fichiers du dossier spécifié aient un schéma différent.

- **Recommandation** :  Assurez-vous que les fichiers du dossier donné ont un schéma identique.


### <a name="error-code--delimitedtextincorrectrowdelimiter"></a>Code d’erreur :  DelimitedTextIncorrectRowDelimiter

- **Message** : `The specified row delimiter %rowDelimiter; is incorrect. Cannot detect a row after parse %size; MB data.`


### <a name="error-code--delimitedtexttoolargecolumncount"></a>Code d’erreur :  DelimitedTextTooLargeColumnCount

- **Message** : `Column count reaches limitation when deserializing csv file. Maximum size is '%size;'. Check the column delimiter and row delimiter provided. (Column delimiter: '%columnDelimiter;', Row delimiter: '%rowDelimiter;')`


### <a name="error-code--delimitedtextinvalidsettings"></a>Code d’erreur :  DelimitedTextInvalidSettings

- **Message** : `%settingIssues;`



## <a name="dynamics-365common-data-servicedynamics-crm"></a>Dynamics 365/Common Data Service/Dynamics CRM

### <a name="error-code--dynamicscreateserviceclienterror"></a>Code d’erreur :  DynamicsCreateServiceClientError

- **Message** : `This is a transient issue on dynamics server side. Try to rerun the pipeline.`

- **Cause** : Il s’agit d’un problème temporaire du côté du serveur Dynamics.

- **Recommandation** :  Exécutez à nouveau le pipeline. Si l’échec persiste, essayez de réduire le parallélisme. Si le problème n’est toujours pas résolu, contactez le support Dynamics.


### <a name="columns-are-missing-when-previewingimporting-schema"></a>Il manque des colonnes au moment de prévisualiser/importer le schéma

- **Symptômes** : Certaines colonnes sont manquantes au moment d’importer le schéma ou de prévisualiser les données. Message d’erreur : `The valid structure information (column name and type) are required for Dynamics source.`

- **Cause** : Ce problème est avant tout lié à la conception. En effet, ADF ne peut pas afficher les colonnes dont les 10 premiers enregistrements n’ont pas de valeur. Vérifiez que les colonnes que vous avez ajoutées présentent le bon format. 

- **Recommandation** : Ajoutez manuellement les colonnes sous l’onglet de mappage.


## <a name="excel-format"></a>Format Excel

### <a name="timeout-or-slow-performance-when-parsing-large-excel-file"></a>Expiration du délai ou baisse du niveau de performance lors de l’analyse d’un fichier Excel volumineux

- **Symptômes** :

    1. Lorsque vous créez un jeu de données Excel et importez un schéma à partir de feuilles de calcul de connexion/du magasin, d’aperçu de données, de liste ou d’actualisation, le délai d’attente peut expirer si la taille du fichier Excel est importante.

    1. Lorsque vous utilisez l’activité de copie pour copier des données à partir d’un fichier Excel volumineux (> = 100 Mo) dans un autre magasin de données, vous pouvez rencontrer une baisse du niveau de performance ou une erreur de type OOM (mémoire insuffisante).

- **Cause** : 

    1. Pour les opérations comme l’importation de feuilles de calcul de schéma, d’aperçu de données et de liste dans un jeu de données Excel, le délai d’expiration est statique et fixé à 100 secondes. Pour un fichier Excel volumineux, ces opérations peuvent ne pas se terminer dans ce délai.

    2. L’activité de copie ADF lit l’intégralité du fichier Excel en mémoire, puis localise la feuille de calcul et les cellules spécifiées pour lire les données. Ce comportement est dû à l’utilisation du Kit de développement logiciel (SDK) sous-jacent.

- **Résolution** : 

    1. Pour importer un schéma, vous pouvez générer un exemple de fichier plus petit représentant un sous-ensemble du fichier d’origine, puis choisir « import schema from sample file » (importer le schéma à partir d’un exemple de fichier) au lieu de « import schema from connection/store » (importer le schéma à partir d’une connexion/d’un magasin).

    2. Pour lister une feuille de calcul, dans la liste déroulante de la feuille de calcul, vous pouvez cliquer sur « Modifier » et entrer à la place le nom/index de la feuille.

    3. Pour copier un fichier Excel volumineux (> 100 Mo) dans un autre magasin, vous pouvez utiliser le flux de données source Excel qui prend en charge diffusion en continu et offre de meilleures performances.


## <a name="hdinsight"></a>HDInsight

### <a name="ssl-error-when-adf-linked-service-using-hdinsight-esp-cluster"></a>Erreur SSL pendant que le service lié ADF utilisait le cluster ESP HDInsight

- **Message** : `Failed to connect to HDInsight cluster: 'ERROR [HY000] [Microsoft][DriverSupport] (1100) SSL certificate verification failed because the certificate is missing or incorrect.`

- **Cause** : Le problème est très probablement lié au magasin de confiance système.

- **Résolution** : Vous pouvez accéder au chemin **Microsoft Integration Runtime\4.0\Shared\ODBC Drivers\Microsoft Hive ODBC Driver\lib** et ouvrir DriverConfiguration64.exe pour modifier le paramètre.

    ![Décocher Utiliser le magasin de confiance système](./media/connector-troubleshoot-guide/system-trust-store-setting.png)


## <a name="json-format"></a>Format JSON

### <a name="error-code--jsoninvalidarraypathdefinition"></a>Code d’erreur :  JsonInvalidArrayPathDefinition

- **Message** : `Error occurred when deserializing source JSON data. Check whether the JsonPath in JsonNodeReference and JsonPathDefintion is valid.`


### <a name="error-code--jsonemptyjobjectdata"></a>Code d’erreur :  JsonEmptyJObjectData

- **Message** : `The specified row delimiter %rowDelimiter; is incorrect. Cannot detect a row after parse %size; MB data.`


### <a name="error-code--jsonnullvalueinpathdefinition"></a>Code d’erreur :  JsonNullValueInPathDefinition

- **Message** : `Null JSONPath detected in JsonPathDefinition.`


### <a name="error-code--jsonunsupportedhierarchicalcomplexvalue"></a>Code d’erreur :  JsonUnsupportedHierarchicalComplexValue

- **Message** : `The retrieved type of data %data; with value %value; is not supported yet. Please either remove the targeted column '%name;' or enable skip incompatible row to skip the issue rows.`


### <a name="error-code--jsonconflictpartitiondiscoveryschema"></a>Code d’erreur :  JsonConflictPartitionDiscoverySchema

- **Message** : `Conflicting partition column names detected.'%schema;', '%partitionDiscoverySchema;'`


### <a name="error-code--jsoninvaliddataformat"></a>Code d’erreur :  JsonInvalidDataFormat

- **Message** : `Error occurred when deserializing source JSON file '%fileName;'. Check if the data is in valid JSON object format.`


### <a name="error-code--jsoninvaliddatamixedarrayandobject"></a>Code d’erreur :  JsonInvalidDataMixedArrayAndObject

- **Message** : `Error occurred when deserializing source JSON file '%fileName;'. The JSON format doesn't allow mixed arrays and objects.`


## <a name="oracle"></a>Oracle

### <a name="error-code-argumentoutofrangeexception"></a>Code d’erreur : ArgumentOutOfRangeException

- **Message** : `Hour, Minute, and Second parameters describe an un-representable DateTime.`

- **Cause** : Dans ADF, les valeurs DateTime sont prises en charge dans la plage comprise entre 0001-01-01 00:00:00 et 9999-12-31 23:59:59. Cependant, Oracle prend en charge une plage plus étendue de valeurs DateTime (comme les siècles av. J-C ou mn/s > 59), ce qui entraîne une défaillance dans ADF.

- **Recommandation** : 

    Exécutez `select dump(<column name>)` pour vérifier si la valeur dans Oracle est comprise dans la plage d’ADF. 

    Si vous souhaitez connaître la séquence d’octets dans le résultat, consultez https://stackoverflow.com/questions/13568193/how-are-dates-stored-in-oracle.


## <a name="parquet-format"></a>Format Parquet

### <a name="error-code--parquetjavainvocationexception"></a>Code d’erreur :  ParquetJavaInvocationException

- **Message** : `An error occurred when invoking java, message: %javaException;.`

- **Cause** : Lorsque le message d’erreur contient « java.lang.OutOfMemory », « Java heap space » et « doubleCapacity », il s’agit généralement d’un problème de gestion de la mémoire dans l’ancienne version du runtime d’intégration.

- **Recommandation** :  Si vous utilisez le runtime d’intégration auto-hébergé et que la version est antérieure à 3.20.7159.1, il est recommandé de procéder à une mise à niveau vers la version la plus récente.

- **Cause** : Lorsque le message d’erreur contient « java.lang.OutOfMemory », le runtime d’intégration ne dispose pas de suffisamment de ressources pour traiter le ou les fichiers.

- **Recommandation** :  Limitez les exécutions simultanées sur le runtime d’intégration. Pour les runtimes d’intégration auto-hébergés, montez en puissance vers un ordinateur puissant doté d’une mémoire égale ou supérieure à 8 Go.

- **Cause** : Quand le message d’erreur contient « NullPointerReference », il peut s’agir d’une erreur temporaire.

- **Recommandation** :  Réessayez. Si le problème persiste, contactez le support.


### <a name="error-code--parquetinvalidfile"></a>Code d’erreur :  ParquetInvalidFile

- **Message** : `File is not a valid Parquet file.`

- **Cause** : Problème de fichier Parquet.

- **Recommandation** :  Vérifiez que l’entrée est un fichier Parquet valide.


### <a name="error-code--parquetnotsupportedtype"></a>Code d’erreur :  ParquetNotSupportedType

- **Message** : `Unsupported Parquet type. PrimitiveType: %primitiveType; OriginalType: %originalType;.`

- **Cause** : Le format Parquet n’est pas pris en charge dans Azure Data Factory.

- **Recommandation** :  Revérifiez les données sources. Reportez-vous au document : https://docs.microsoft.com/azure/data-factory/supported-file-formats-and-compression-codecs.


### <a name="error-code--parquetmisseddecimalprecisionscale"></a>Code d’erreur :  ParquetMissedDecimalPrecisionScale

- **Message** : `Decimal Precision or Scale information is not found in schema for column: %column;.`

- **Cause** : Essaie d’analyser la précision et l’échelle des nombres, mais aucune information de ce type n’est fournie.

- **Recommandation** :  « Source » ne renvoie pas une précision et une échelle correctes. Vérifiez la précision et l’échelle de la colonne posant problème.


### <a name="error-code--parquetinvaliddecimalprecisionscale"></a>Code d’erreur :  ParquetInvalidDecimalPrecisionScale

- **Message** : `Invalid Decimal Precision or Scale. Precision: %precision; Scale: %scale;.`

- **Cause** : Le schéma n’est pas valide.

- **Recommandation** :  Vérifiez la précision et l’échelle de la colonne posant problème.


### <a name="error-code--parquetcolumnnotfound"></a>Code d’erreur :  ParquetColumnNotFound

- **Message** : `Column %column; does not exist in Parquet file.`

- **Cause** : Le schéma source est incompatible avec le schéma du récepteur.

- **Recommandation** :  Vérifiez la colonne « mappings » dans « activity ». Assurez-vous que la colonne source peut être mappée à la colonne du récepteur appropriée.


### <a name="error-code--parquetinvaliddataformat"></a>Code d’erreur :  ParquetInvalidDataFormat

- **Message** : `Incorrect format of %srcValue; for converting to %dstType;.`

- **Cause** : Les données ne peuvent pas être converties dans le type spécifié dans mappings.source.

- **Recommandation** :  Revérifiez les données sources ou spécifiez le type de données approprié pour cette colonne dans le mappage des colonnes de l’activité de copie. Reportez-vous au document : https://docs.microsoft.com/azure/data-factory/supported-file-formats-and-compression-codecs.


### <a name="error-code--parquetdatacountnotmatchcolumncount"></a>Code d’erreur :  ParquetDataCountNotMatchColumnCount

- **Message** : `The data count in a row '%sourceColumnCount;' does not match the column count '%sinkColumnCount;' in given schema.`

- **Cause** : Non-concordance du nombre de colonnes sources et du nombre de colonnes du récepteur

- **Recommandation** :  Revérifiez que le nombre de colonnes sources est identique au nombre de colonnes du récepteur dans « mapping ».


### <a name="error-code--parquetdatatypenotmatchcolumntype"></a>Code d’erreur :  ParquetDataTypeNotMatchColumnType

- **Message** : Le type de données%srcType; ne correspond pas au type de colonne %dstType; donné dans la colonne « %columnIndex; ».

- **Cause** : Les données de la source ne peuvent pas être converties dans le type défini du récepteur

- **Recommandation** :  Spécifiez un type correct dans mappage.sink.


### <a name="error-code--parquetbridgeinvaliddata"></a>Code d’erreur :  ParquetBridgeInvalidData

- **Message** : `%message;`

- **Cause** : Valeur de données au-delà de la limitation

- **Recommandation** :  Réessayez. Si le problème persiste, contactez-nous.


### <a name="error-code--parquetunsupportedinterpretation"></a>Code d’erreur :  ParquetUnsupportedInterpretation

- **Message** : `The given interpretation '%interpretation;' of Parquet format is not supported.`

- **Cause** : Scénario non pris en charge

- **Recommandation** :  « ParquetInterpretFor » ne doit pas être « sparkSql ».


### <a name="error-code--parquetunsupportfilelevelcompressionoption"></a>Code d’erreur :  ParquetUnsupportFileLevelCompressionOption

- **Message** : `File level compression is not supported for Parquet.`

- **Cause** : Scénario non pris en charge

- **Recommandation** :  Supprimez « CompressionType » dans la charge utile.


### <a name="error-code--usererrorjniexception"></a>Code d’erreur :  UserErrorJniException

- **Message** : `Cannot create JVM: JNI return code [-6][JNI call failed: Invalid arguments.]`

- **Cause** : La machine virtuelle Java ne peut pas être créée, car des arguments (globaux) non conformes sont définis.

- **Recommandation** :  Connectez-vous à la machine qui héberge **chaque nœud** de votre IR auto-hébergé. Vérifiez si la variable système est correctement définie comme suit : `_JAVA_OPTIONS "-Xms256m -Xmx16g" with memory bigger than 8 G`. Redémarrez tous les nœuds IR, puis réexécutez le pipeline.


### <a name="arithmetic-overflow"></a>Dépassement arithmétique

- **Symptômes** : Un message d’erreur s’est produit pendant la copie de fichiers Parquet : `Message = Arithmetic Overflow., Source = Microsoft.DataTransfer.Common`

- **Cause** : Pour l’heure, seules les décimales d’une précision < = 38 et une longueur de la partie entière < = 20 sont prises en charge lors de la copie de fichiers d’Oracle vers Parquet. 

- **Résolution** : Pour contourner ce problème, vous pouvez convertir les colonnes en VARCHAR2.


### <a name="no-enum-constant"></a>Absence de constante enum

- **Symptômes** : Un message d’erreur s’est affiché pendant la copie des données au format Paquet : `java.lang.IllegalArgumentException:field ended by &apos;;&apos;` ou `java.lang.IllegalArgumentException:No enum constant org.apache.parquet.schema.OriginalType.test`.

- **Cause** : 

    Le problème peut être dû à la présence d’espaces blancs ou de caractères non pris en charge (comme ,;{}()\n\t=) dans le nom de colonne, car Parquet ne prend pas en charge un tel format. 

    Par exemple, le nom de colonne *contoso(test)* analyse le type entre crochets dans le [code](https://github.com/apache/parquet-mr/blob/master/parquet-column/src/main/java/org/apache/parquet/schema/MessageTypeParser.java) `Tokenizer st = new Tokenizer(schemaString, " ;{}()\n\t");`. L’erreur est déclenchée, car il n’existe pas de type « test ».

    Vous pouvez vérifier les types pris en charge [ici](https://github.com/apache/parquet-mr/blob/master/parquet-column/src/main/java/org/apache/parquet/schema/OriginalType.java).

- **Résolution** : 

    1. Vérifiez si le nom de colonne récepteur contient des espaces blancs.

    1. Vérifiez si la première ligne contenant des espaces blancs sert de nom de colonne.

    1. Vérifiez si le type OriginalType est pris en charge ou non. Essayez d’éviter ces caractères spéciaux `,;{}()\n\t=`. 


## <a name="rest"></a>REST

### <a name="unexpected-network-response-from-rest-connector"></a>Réponse réseau inattendue du connecteur REST

- **Symptômes** : Le point de terminaison reçoit parfois une réponse inattendue (400/401/403/500) du connecteur REST.

- **Cause** : Le connecteur source REST utilise l’URL et la méthode/l’en-tête/le corps HTTP du service/du jeu de données/de la copie source comme paramètres lors de la construction d’une requête HTTP. Le problème est probablement dû à des erreurs dans un ou plusieurs paramètres spécifiés.

- **Résolution** : 
    - Utilisez 'curl' dans la fenêtre cmd pour vérifier si le paramètre est la cause ou non (les en-têtes **Accept** et **User-Agent** doivent toujours être inclus) :
        ```
        curl -i -X <HTTP method> -H <HTTP header1> -H <HTTP header2> -H "Accept: application/json" -H "User-Agent: azure-data-factory/2.0" -d '<HTTP body>' <URL>
        ```
      Si la commande retourne la même réponse inattendue, corrigez les paramètres ci-dessus en utilisant 'curl' jusqu’à obtenir la réponse attendue. 

      Vous pouvez également utiliser 'curl --help' pour exécuter des options plus avancées de la commande.

    - Si seul le connecteur REST ADF renvoie une réponse inattendue, contactez le support technique de Microsoft pour poursuivre la résolution des problèmes.
    
    - Notez que 'curl' peut ne pas convenir pour reproduire le problème de validation du certificat SSL. Dans certains scénarios, la commande 'curl' a été exécutée avec succès sans rencontrer de problème de validation de certificat SSL. Toutefois, lorsque la même URL est exécutée dans le navigateur, aucun certificat SSL n’est réellement renvoyé dans le premier emplacement pour permettre au client d’établir une relation de confiance avec le serveur.

      Les outils comme **Postman** et **Fiddler** sont recommandés dans le cas ci-dessus.


## <a name="sftp"></a>SFTP

### <a name="invalid-sftp-credential-provided-for-sshpublickey-authentication-type"></a>Informations d’identification SFTP non valides fournies pour le type d’authentification « SSHPublicKey »

- **Symptômes** : L’authentification par clé publique SSH est utilisée alors que des informations d’identification SFTP non valides sont fournies pour le type d’authentification « SshPublicKey ».

- **Cause** : Les raisons possibles à l’origine de cette erreur sont au nombre de trois :

    1. Le contenu de la clé privée est extrait d’AKV ou du SDK, mais il n’est pas correctement encodé.

    1. Le format de contenu de clé choisi est incorrect.

    1. Contenu de clé privée ou informations d’identification non valides.

- **Résolution** : 

    1. Pour la **cause 1** :

       Si le contenu de la clé privée provient d’AKV et que le fichier de clé d’origine peut fonctionner si le client le charge directement dans le service lié SFTP

       Consultez https://docs.microsoft.com/azure/data-factory/connector-sftp#using-ssh-public-key-authentication, le contenu de privateKey est un contenu de clé privée SSH encodé en base64.

       Encodez **l’intégralité du contenu du fichier de clé privée d’origine** avec un encodage en base64 et stockez la chaîne encodée dans AKV. Le fichier de clé privée d’origine est celui qui peut fonctionner dans le service lié SFTP si vous cliquez sur Charger à partir d’un fichier.

       Voici quelques exemples utilisés pour générer la chaîne :

       - Utilisation de code C# :
       ```
       byte[] keyContentBytes = File.ReadAllBytes(Private Key Path);
       string keyContent = Convert.ToBase64String(keyContentBytes, Base64FormattingOptions.None);
       ```

       - Utilisation de code Python :
       ```
       import base64
       rfd = open(r'{Private Key Path}', 'rb')
       keyContent = rfd.read()
       rfd.close()
       print base64.b64encode(Key Content)
       ```

       - Utilisation d’un outil de conversion en base64 tiers

         Des outils tels que https://www.base64encode.org/ sont recommandés.

    1. Pour la **cause 2** :

       Si la clé privée SSH au format PKCS#8 est utilisée

       La clé privée SSH au format PKCS#8 (commençant par « -----BEGIN ENCRYPTED PRIVATE KEY----- ») n’est actuellement pas prise en charge pour l’accès au serveur SFTP dans ADF. 

       Exécutez les commandes ci-dessous pour convertir la clé au format de clé SSH classique (commençant par « ------BEGIN RSA PRIVATE KEY----- ») :

       ```
       openssl pkcs8 -in pkcs8_format_key_file -out traditional_format_key_file
       chmod 600 traditional_format_key_file
       ssh-keygen -f traditional_format_key_file -p
       ```
    1. Pour la **cause 3** :

       Servez-vous d’un outil comme WinSCP pour vérifier si votre fichier de clé ou mot de passe est correct.


### <a name="incorrect-linked-service-type-is-used"></a>Le type de service lié utilisé est incorrect

- **Symptômes** : Le serveur FTP/SFTP n’est pas accessible.

- **Cause** : Le type de service lié utilisé est incorrect pour le serveur FTP ou SFTP. Tel est le cas si le service lié FTP est utilisé pour se connecter à un serveur SFTP ou inversement.

- **Résolution** : Vérifiez le port du serveur cible. Par défaut, FTP utilise le port 21 et SFTP le port 22.


### <a name="sftp-copy-activity-failed"></a>Échec de l’activité de copie SFTP

- **Symptômes** : Code d’erreur : UserErrorInvalidColumnMappingColumnNotFound. Message d’erreur : `Column &apos;AccMngr&apos; specified in column mapping cannot be found in source data.`

- **Cause** : La source ne comporte pas de colonne nommée « AccMngr ».

- **Résolution** : Vérifiez la façon dont votre jeu de données est configuré en mappant la colonne du jeu de données de destination afin de confirmer la présence d’une colonne « AccMngr ».


### <a name="sftp-server-connection-throttling"></a>Limitation de la connexion du serveur SFTP

- **Symptômes** : La réponse du serveur ne contient pas d’identification de protocole SSH et la copie a échoué.

- **Cause** : ADF crée plusieurs connexions pour opérer des téléchargements parallèles à partir du serveur SFTP et atteint parfois la limitation du serveur SFTP. Dans la pratique, l’erreur retournée une fois la limitation atteinte varie d’un serveur à un autre.

- **Résolution** : 

    Définissez le nombre maximal de connexions simultanées du jeu de données SFTP sur 1, puis réexécutez la copie. Si elle aboutit, cela est le signe que la limitation est en cause.

    Si vous souhaitez promouvoir le faible débit, contactez l’administrateur SFTP pour augmenter le nombre limite de connexions simultanées ou ajoutez les adresses IP ci-dessous à la liste verte :

    - Si vous utilisez l’IR managé, ajoutez les [plages d’adresses IP du centre de données Azure](https://www.microsoft.com/download/details.aspx?id=41653).
      Vous pouvez aussi installer l’IR auto-hébergé si vous ne voulez pas ajouter une longue plage d’adresses IP dans la liste verte du serveur SFTP.

    - Si vous utilisez l’IR auto-hébergé, ajoutez l’adresse IP de la machine qui a installé SHIR à la liste verte.


### <a name="error-code-sftprenameoperationfail"></a>Code d’erreur : SftpRenameOperationFail

- **Symptômes** : Le pipeline n’a pas pu copier les données de l’objet blob vers SFTP avec l’erreur suivante : `Operation on target Copy_5xe failed: Failure happened on 'Sink' side. ErrorCode=SftpRenameOperationFail,Type=Microsoft.DataTransfer.Common.Shared.HybridDeliveryException`.

- **Cause** : L’option useTempFileRename était définie sur True au moment où la copie des données s’est produite. Cela permet au processus d’utiliser des fichiers temporaires. L’erreur se déclenche si un ou plusieurs fichiers temporaires ont été supprimés avant la copie de l’ensemble des données.

- **Résolution** : Définissez l’option useTempFileName sur False.


## <a name="general-copy-activity-error"></a>Erreur générale de l’activité de copie

### <a name="error-code--jrenotfound"></a>Code d’erreur :  JreNotFound

- **Message** : `Java Runtime Environment cannot be found on the Self-hosted Integration Runtime machine. It is required for parsing or writing to Parquet/ORC files. Make sure Java Runtime Environment has been installed on the Self-hosted Integration Runtime machine.`

- **Cause** : Le runtime d’intégration auto-hébergé ne parvient pas à trouver le runtime Java. Le runtime Java est requis pour la lecture d’une source particulière.

- **Recommandation** :  Vérifiez votre environnement de runtime d’intégration et reportez-vous au document de référence : https://docs.microsoft.com/azure/data-factory/format-parquet#using-self-hosted-integration-runtime


### <a name="error-code--wildcardpathsinknotsupported"></a>Code d’erreur :  WildcardPathSinkNotSupported

- **Message** : `Wildcard in path is not supported in sink dataset. Fix the path: '%setting;'.`

- **Cause** : Le jeu de données du récepteur ne prend pas en charge les caractères génériques.

- **Recommandation** :  Vérifiez le jeu de données du récepteur et corrigez le chemin d’accès sans la valeur de caractère générique.


### <a name="error-code--mappinginvalidpropertywithemptyvalue"></a>Code d’erreur :  MappingInvalidPropertyWithEmptyValue

- **Message** : `One or more '%sourceOrSink;' in copy activity mapping doesn't point to any data. Choose one of the three properties 'name', 'path' and 'ordinal' to reference columns/fields.`


### <a name="error-code--mappinginvalidpropertywithnamepathandordinal"></a>Code d’erreur :  MappingInvalidPropertyWithNamePathAndOrdinal

- **Message** : `Mixed properties are used to reference '%sourceOrSink;' columns/fields in copy activity mapping. Please only choose one of the three properties 'name', 'path' and 'ordinal'. The problematic mapping setting is 'name': '%name;', 'path': '%path;','ordinal': '%ordinal;'.`


### <a name="error-code--mappingduplicatedordinal"></a>Code d’erreur :  MappingDuplicatedOrdinal

- **Message** : `Copy activity 'mappings' has duplicated ordinal value "%Ordinal;". Fix the setting in 'mappings'.`


### <a name="error-code--mappinginvalidordinalforsinkcolumn"></a>Code d’erreur :  MappingInvalidOrdinalForSinkColumn

- **Message** : `Invalid 'ordinal' property for sink column under 'mappings' property. Ordinal: %Ordinal;.`


## <a name="next-steps"></a>Étapes suivantes

Si vous avez besoin d’une aide supplémentaire, essayez les ressources suivantes :

*  [Blog Data Factory](https://azure.microsoft.com/blog/tag/azure-data-factory/)
*  [Demandes de fonctionnalités Data Factory](https://feedback.azure.com/forums/270578-data-factory)
*  [Vidéos Azure](https://azure.microsoft.com/resources/videos/index/?sort=newest&services=data-factory)
*  [Page de questions Microsoft Q&R](/answers/topics/azure-data-factory.html)
*  [Forum Stack Overflow pour Data Factory](https://stackoverflow.com/questions/tagged/azure-data-factory)
*  [Informations Twitter sur Data Factory](https://twitter.com/hashtag/DataFactory)
