---
title: Code de mémoire tampon en anneau XEvent
description: Fournit un exemple de code Transact-SQL simple et rapide en utilisant la cible de la mémoire tampon en anneau, dans Azure SQL Database.
services: sql-database
ms.service: sql-database
ms.subservice: performance
ms.custom: sqldbrb=1
ms.devlang: PowerShell
ms.topic: sample
author: WilliamDAssafMSFT
ms.author: wiassaf
ms.reviewer: sstein
ms.date: 12/19/2018
ms.openlocfilehash: a646588616b874e40b1ed2a5a0b5e691b075075d
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96487301"
---
# <a name="ring-buffer-target-code-for-extended-events-in-azure-sql-database"></a>Code cible de la mémoire tampon en anneau pour les événements étendus dans Azure SQL Database
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

[!INCLUDE [sql-database-xevents-selectors-1-include](../../../includes/sql-database-xevents-selectors-1-include.md)]

Vous pouvez utiliser un exemple de code complet pour capturer et signaler le plus simplement et rapidement possible les informations liées à un événement étendu pendant un test. La cible la plus simple pour les données d’événement étendu est la [cible de la mémoire tampon en anneau](/previous-versions/sql/sql-server-2016/bb630339(v=sql.130)).

Cette rubrique présente un exemple de code Transact-SQL qui :

1. Crée une table contenant des données pour la démonstration.
2. Crée une session pour un événement étendu existant, à savoir **sqlserver.sql_statement_starting**.

   * L’événement est limité aux instructions SQL qui contiennent une chaîne Update particulière : **statement LIKE '%UPDATE tabEmployee%'** .
   * Choisit d’envoyer la sortie de l’événement vers une cible de type mémoire tampon en anneau, à savoir **package0.ring_buffer**.
3. Démarre la session d’événement.
4. Émet un ensemble d’instructions SQL UPDATE simples.
5. Émet une instruction SQL SELECT pour récupérer la sortie d’événement de la mémoire tampon en anneau.

   * **sys.dm_xe_database_session_targets** et d’autres vues de gestion dynamique (DMV) sont incluses.
6. Arrête la session d’événement.
7. Supprime la cible de la mémoire tampon en anneau pour libérer ses ressources.
8. Supprime la session d’événement et la table de démonstration.

## <a name="prerequisites"></a>Prérequis

* Un compte et un abonnement Azure. Vous pouvez vous inscrire à un [essai gratuit](https://azure.microsoft.com/pricing/free-trial/).
* Une base de données dans laquelle vous pouvez créer une table.
  
  * Vous pouvez aussi [créer une base de données de démonstration **AdventureWorksLT**](single-database-create-quickstart.md) en quelques minutes.
* SQL Server Management Studio (ssms.exe), dans l’idéal, la version de sa dernière mise à jour mensuelle.
  Vous pouvez télécharger la dernière version de ssms.exe :
  
  * À partir de la rubrique [Télécharger SQL Server Management Studio](/sql/ssms/download-sql-server-management-studio-ssms).
  * [En utilisant un lien direct vers le téléchargement.](https://go.microsoft.com/fwlink/?linkid=616025)

## <a name="code-sample"></a>Exemple de code

Après quelques modifications mineures, vous pouvez exécuter l’exemple suivant de code de mémoire tampon en anneau sur Azure SQL Database ou Microsoft SQL Server. La différence se limite à la présence du nœud « _database » dans le nom de certaines vues de gestion dynamique (DMV) utilisées dans la clause FROM à l’étape 5. Par exemple :

* sys.dm_xe<strong>_database</strong>_session_targets
* sys.dm_xe_session_targets

&nbsp;

```sql
GO
----  Transact-SQL.
---- Step set 1.

SET NOCOUNT ON;
GO


IF EXISTS
    (SELECT * FROM sys.objects
        WHERE type = 'U' and name = 'tabEmployee')
BEGIN
    DROP TABLE tabEmployee;
END
GO


CREATE TABLE tabEmployee
(
    EmployeeGuid         uniqueIdentifier   not null  default newid()  primary key,
    EmployeeId           int                not null  identity(1,1),
    EmployeeKudosCount   int                not null  default 0,
    EmployeeDescr        nvarchar(256)          null
);
GO


INSERT INTO tabEmployee ( EmployeeDescr )
    VALUES ( 'Jane Doe' );
GO

---- Step set 2.


IF EXISTS
    (SELECT * from sys.database_event_sessions
        WHERE name = 'eventsession_gm_azuresqldb51')
BEGIN
    DROP EVENT SESSION eventsession_gm_azuresqldb51
        ON DATABASE;
END
GO


CREATE
    EVENT SESSION eventsession_gm_azuresqldb51
    ON DATABASE
    ADD EVENT
        sqlserver.sql_statement_starting
            (
            ACTION (sqlserver.sql_text)
            WHERE statement LIKE '%UPDATE tabEmployee%'
            )
    ADD TARGET
        package0.ring_buffer
            (SET
                max_memory = 500   -- Units of KB.
            );
GO

---- Step set 3.


ALTER EVENT SESSION eventsession_gm_azuresqldb51
    ON DATABASE
    STATE = START;
GO

---- Step set 4.


SELECT 'BEFORE_Updates', EmployeeKudosCount, * FROM tabEmployee;

UPDATE tabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 102;

UPDATE tabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 1015;

SELECT 'AFTER__Updates', EmployeeKudosCount, * FROM tabEmployee;
GO

---- Step set 5.


SELECT
    se.name                      AS [session-name],
    ev.event_name,
    ac.action_name,
    st.target_name,
    se.session_source,
    st.target_data,
    CAST(st.target_data AS XML)  AS [target_data_XML]
FROM
               sys.dm_xe_database_session_event_actions  AS ac

    INNER JOIN sys.dm_xe_database_session_events         AS ev  ON ev.event_name = ac.event_name
        AND CAST(ev.event_session_address AS BINARY(8)) = CAST(ac.event_session_address AS BINARY(8))

    INNER JOIN sys.dm_xe_database_session_object_columns AS oc
         ON CAST(oc.event_session_address AS BINARY(8)) = CAST(ac.event_session_address AS BINARY(8))

    INNER JOIN sys.dm_xe_database_session_targets        AS st
         ON CAST(st.event_session_address AS BINARY(8)) = CAST(ac.event_session_address AS BINARY(8))

    INNER JOIN sys.dm_xe_database_sessions               AS se
         ON CAST(ac.event_session_address AS BINARY(8)) = CAST(se.address AS BINARY(8))
WHERE
        oc.column_name = 'occurrence_number'
    AND
        se.name        = 'eventsession_gm_azuresqldb51'
    AND
        ac.action_name = 'sql_text'
ORDER BY
    se.name,
    ev.event_name,
    ac.action_name,
    st.target_name,
    se.session_source
;
GO

---- Step set 6.


ALTER EVENT SESSION eventsession_gm_azuresqldb51
    ON DATABASE
    STATE = STOP;
GO

---- Step set 7.


ALTER EVENT SESSION eventsession_gm_azuresqldb51
    ON DATABASE
    DROP TARGET package0.ring_buffer;
GO

---- Step set 8.


DROP EVENT SESSION eventsession_gm_azuresqldb51
    ON DATABASE;
GO

DROP TABLE tabEmployee;
GO
```

&nbsp;

## <a name="ring-buffer-contents"></a>Contenu de la mémoire tampon en anneau

Nous avons utilisé `ssms.exe` pour exécuter l'exemple de code.

Pour afficher les résultats, nous avons cliqué sur la cellule sous l’en-tête de colonne **target_data_XML**.

Puis, dans le volet de résultats, nous avons cliqué sur la cellule sous l’en-tête de colonne **target_data_XML**. Ce clic a créé un autre onglet Fichier dans ssms.exe pour afficher (au format XML) le contenu de la cellule de résultat.

La sortie est présentée dans le bloc suivant. Elle semble longue, mais ne comprend que deux éléments **\<event>** .

&nbsp;

```
<RingBufferTarget truncated="0" processingTime="0" totalEventsProcessed="2" eventCount="2" droppedCount="0" memoryUsed="1728">
  <event name="sql_statement_starting" package="sqlserver" timestamp="2015-09-22T15:29:31.317Z">
    <data name="state">
      <type name="statement_starting_state" package="sqlserver" />
      <value>0</value>
      <text>Normal</text>
    </data>
    <data name="line_number">
      <type name="int32" package="package0" />
      <value>7</value>
    </data>
    <data name="offset">
      <type name="int32" package="package0" />
      <value>184</value>
    </data>
    <data name="offset_end">
      <type name="int32" package="package0" />
      <value>328</value>
    </data>
    <data name="statement">
      <type name="unicode_string" package="package0" />
      <value>UPDATE tabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 102</value>
    </data>
    <action name="sql_text" package="sqlserver">
      <type name="unicode_string" package="package0" />
      <value>
---- Step set 4.


SELECT 'BEFORE_Updates', EmployeeKudosCount, * FROM tabEmployee;

UPDATE tabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 102;

UPDATE tabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 1015;

SELECT 'AFTER__Updates', EmployeeKudosCount, * FROM tabEmployee;
</value>
    </action>
  </event>
  <event name="sql_statement_starting" package="sqlserver" timestamp="2015-09-22T15:29:31.327Z">
    <data name="state">
      <type name="statement_starting_state" package="sqlserver" />
      <value>0</value>
      <text>Normal</text>
    </data>
    <data name="line_number">
      <type name="int32" package="package0" />
      <value>10</value>
    </data>
    <data name="offset">
      <type name="int32" package="package0" />
      <value>340</value>
    </data>
    <data name="offset_end">
      <type name="int32" package="package0" />
      <value>486</value>
    </data>
    <data name="statement">
      <type name="unicode_string" package="package0" />
      <value>UPDATE tabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 1015</value>
    </data>
    <action name="sql_text" package="sqlserver">
      <type name="unicode_string" package="package0" />
      <value>
---- Step set 4.


SELECT 'BEFORE_Updates', EmployeeKudosCount, * FROM tabEmployee;

UPDATE tabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 102;

UPDATE tabEmployee
    SET EmployeeKudosCount = EmployeeKudosCount + 1015;

SELECT 'AFTER__Updates', EmployeeKudosCount, * FROM tabEmployee;
</value>
    </action>
  </event>
</RingBufferTarget>
```

### <a name="release-resources-held-by-your-ring-buffer"></a>Libérer les ressources détenues par votre mémoire tampon en anneau

Quand vous n’avez plus besoin de votre mémoire tampon en anneau, vous pouvez la supprimer et libérer ses ressources à l’aide d’une instruction **ALTER** , comme suit :

```sql
ALTER EVENT SESSION eventsession_gm_azuresqldb51
    ON DATABASE
    DROP TARGET package0.ring_buffer;
GO
```

La définition de votre session d’événement est mise à jour, mais pas supprimée. Vous pouvez ajouter ultérieurement une autre instance de la mémoire tampon en anneau à votre session d’événement :

```sql
ALTER EVENT SESSION eventsession_gm_azuresqldb51
    ON DATABASE
    ADD TARGET
        package0.ring_buffer
            (SET
                max_memory = 500   -- Units of KB.
            );
```

## <a name="more-information"></a>Informations complémentaires

La rubrique principale sur les événements étendus dans Azure SQL Database est :

* La rubrique [Considérations relatives aux événements étendus dans Azure SQL Database](xevent-db-diff-from-svr.md) décrit les différences à prendre en compte entre les événements étendus d'Azure SQL Database et de Microsoft SQL Server.

Vous trouverez d’autres rubriques d’exemples de code pour les événements étendus en suivant le lien ci-dessous. Toutefois, vous devez vérifier régulièrement les exemples pour voir s’ils ciblent Microsoft SQL Server ou Azure SQL Database. Vous pouvez ensuite déterminer si vous devez apporter quelques modifications mineures avant d’exécuter un exemple.

* Exemple de code pour Azure SQL Database : [Code cible du fichier d'événements pour les événements étendus dans Azure SQL Database](xevent-code-event-file.md)

<!--
('lock_acquired' event.)

- Code sample for SQL Server: [Determine Which Queries Are Holding Locks](/sql/relational-databases/extended-events/determine-which-queries-are-holding-locks)
- Code sample for SQL Server: [Find the Objects That Have the Most Locks Taken on Them](/sql/relational-databases/extended-events/find-the-objects-that-have-the-most-locks-taken-on-them)
-->