---
title: 'Démarrage rapide : Charger en masse des données à l’aide d’une seule instruction T-SQL'
description: Charger en masse des données à l’aide de l’instruction COPY
services: synapse-analytics
author: kevinvngo
manager: craigg
ms.service: synapse-analytics
ms.topic: quickstart
ms.subservice: sql-dw
ms.date: 11/20/2020
ms.author: kevin
ms.reviewer: jrasnick
ms.custom: azure-synapse
ms.openlocfilehash: 83b5804888379316b855c36f803f646cec102d9e
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95024583"
---
# <a name="quickstart-bulk-load-data-using-the-copy-statement"></a>Démarrage rapide : Charger en masse des données à l’aide de l’instruction COPY

Dans ce guide de démarrage rapide, vous allez charger en masse des données dans votre pool SQL dédié au moyen de l’[instruction COPY](https://docs.microsoft.com/sql/t-sql/statements/copy-into-transact-sql?view=azure-sqldw-latest) simple et flexible pour l’ingestion de données à débit élevé. L’instruction COPY est l’utilitaire de chargement recommandé, car il vous permet de charger des données de manière fluide et flexible en fournissant des fonctionnalités pour :

- Autoriser le chargement aux utilisateurs avec privilèges plus restreints, sans avoir besoin d’autorisations CONTROL strictes sur l’entrepôt de données
- Tirer parti uniquement d’une instruction T-SQL sans avoir à créer d’objets de base de données supplémentaires
- Bénéficier d’un meilleur modèle d’autorisation sans exposer les clés de compte de stockage à l’aide de signatures d’accès partagé (SAS)
- Spécifier un autre compte de stockage pour l’emplacement ERRORFILE (REJECTED_ROW_LOCATION)
- Personnaliser les valeurs par défaut pour chaque colonne cible et spécifier les champs de données sources à charger dans des colonnes cibles spécifiques
- Spécifier une marque de fin de ligne personnalisée pour les fichiers CSV
- Séparateurs de ligne, de champ et de chaîne d’échappement pour les fichiers CSV
- Tirer parti des formats de date SQL Server pour les fichiers CSV
- Spécifier des caractères génériques et plusieurs fichiers dans le chemin de l’emplacement du stockage

## <a name="prerequisites"></a>Prérequis

Ce guide de démarrage rapide part du principe que vous disposez déjà d’un pool SQL dédié. Si aucun pool SQL dédié n’a été créé, utilisez le guide démarrage rapide [Créer et se connecter - Portail](create-data-warehouse-portal.md).

## <a name="set-up-the-required-permissions"></a>Configurer les autorisations nécessaires

```sql
-- List the permissions for your user
select  princ.name
,       princ.type_desc
,       perm.permission_name
,       perm.state_desc
,       perm.class_desc
,       object_name(perm.major_id)
from    sys.database_principals princ
left join
        sys.database_permissions perm
on      perm.grantee_principal_id = princ.principal_id
where name = '<yourusername>';

--Make sure your user has the permissions to CREATE tables in the [dbo] schema
GRANT CREATE TABLE TO <yourusername>;
GRANT ALTER ON SCHEMA::dbo TO <yourusername>;

--Make sure your user has ADMINISTER DATABASE BULK OPERATIONS permissions
GRANT ADMINISTER DATABASE BULK OPERATIONS TO <yourusername>

--Make sure your user has INSERT permissions on the target table
GRANT INSERT ON <yourtable> TO <yourusername>

```

## <a name="create-the-target-table"></a>Créer la table cible

Dans cet exemple, nous allons charger des données à partir du jeu de données des taxis de New York. Nous allons charger une table appelée Trip qui représente les trajets de taxi effectués en une seule année. Exécutez le code suivant pour créer la table :

```sql
CREATE TABLE [dbo].[Trip]
(
    [DateID] int NOT NULL,
    [MedallionID] int NOT NULL,
    [HackneyLicenseID] int NOT NULL,
    [PickupTimeID] int NOT NULL,
    [DropoffTimeID] int NOT NULL,
    [PickupGeographyID] int NULL,
    [DropoffGeographyID] int NULL,
    [PickupLatitude] float NULL,
    [PickupLongitude] float NULL,
    [PickupLatLong] varchar(50) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
    [DropoffLatitude] float NULL,
    [DropoffLongitude] float NULL,
    [DropoffLatLong] varchar(50) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
    [PassengerCount] int NULL,
    [TripDurationSeconds] int NULL,
    [TripDistanceMiles] float NULL,
    [PaymentType] varchar(50) COLLATE SQL_Latin1_General_CP1_CI_AS NULL,
    [FareAmount] money NULL,
    [SurchargeAmount] money NULL,
    [TaxAmount] money NULL,
    [TipAmount] money NULL,
    [TollsAmount] money NULL,
    [TotalAmount] money NULL
)
WITH
(
    DISTRIBUTION = ROUND_ROBIN,
    CLUSTERED COLUMNSTORE INDEX
);
```

## <a name="run-the-copy-statement"></a>Exécuter l’instruction COPY

Exécutez l’instruction COPY suivante qui chargera les données du compte de stockage Blob Azure dans la table Trip.

```sql
COPY INTO [dbo].[Trip] FROM 'https://nytaxiblob.blob.core.windows.net/2013/Trip2013/'
WITH (
   FIELDTERMINATOR='|',
   ROWTERMINATOR='0x0A'
) OPTION (LABEL = 'COPY: dbo.trip');
```

## <a name="monitor-the-load"></a>Superviser la charge

Vérifiez la progression de votre charge en exécutant régulièrement la requête suivante :

```sql
SELECT  r.[request_id]                           
,       r.[status]                               
,       r.resource_class                         
,       r.command
,       sum(bytes_processed) AS bytes_processed
,       sum(rows_processed) AS rows_processed
FROM    sys.dm_pdw_exec_requests r
              JOIN sys.dm_pdw_dms_workers w
                     ON r.[request_id] = w.request_id
WHERE [label] = 'COPY: dbo.trip' and session_id <> session_id() and type = 'WRITER'
GROUP BY r.[request_id]                           
,       r.[status]                               
,       r.resource_class                         
,       r.command;

```

## <a name="next-steps"></a>Étapes suivantes

- Pour connaître les bonnes pratiques en matière de chargement de données, consultez [Bonnes pratiques de chargement de données](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/guidance-for-loading-data).
- Pour plus d’informations sur la gestion des ressources de vos charges de données, consultez [Isolation des charges de travail](https://docs.microsoft.com/azure/synapse-analytics/sql-data-warehouse/quickstart-configure-workload-isolation-tsql). 
