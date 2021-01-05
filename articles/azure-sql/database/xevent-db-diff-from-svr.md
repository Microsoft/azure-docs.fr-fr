---
title: Événements étendus
description: Décrit les événements étendus (XEvents) dans Azure SQL Database et les légères différences entre les sessions d’événements dans Microsoft SQL Server.
services: sql-database
ms.service: sql-database
ms.subservice: performance
ms.custom: sqldbrb=1
ms.devlang: ''
ms.topic: reference
author: WilliamDAssafMSFT
ms.author: wiassaf
ms.reviewer: sstein
ms.date: 12/19/2018
ms.openlocfilehash: 139673e46421aa0dc19298697872fbff5fe587af
ms.sourcegitcommit: d60976768dec91724d94430fb6fc9498fdc1db37
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96501207"
---
# <a name="extended-events-in-azure-sql-database"></a>Événements étendus dans une base de données SQL Azure 
[!INCLUDE[appliesto-sqldb](../includes/appliesto-sqldb.md)]

[!INCLUDE [sql-database-xevents-selectors-1-include](../../../includes/sql-database-xevents-selectors-1-include.md)]

L’ensemble de fonctionnalités des événements étendus dans Azure SQL Database est une bonne partie des fonctionnalités sur SQL Server et Azure SQL Managed Instance.

*XEvents* est un surnom informel parfois utilisé pour désigne les « événements étendus » dans les blogs et autres sites informels.

Vous trouverez des informations supplémentaires sur les événements étendus dans les articles suivants :

- [Démarrage rapide : Événements étendus dans SQL Server](/sql/relational-databases/extended-events/quick-start-extended-events-in-sql-server)
- [Événements étendus](/sql/relational-databases/extended-events/extended-events)

## <a name="prerequisites"></a>Prérequis

Cette rubrique part du principe que vous connaissez déjà les éléments suivants :

- [Azure SQL Database](https://azure.microsoft.com/services/sql-database/)
- [Événements étendus](/sql/relational-databases/extended-events/extended-events)

- La majeure partie de notre documentation sur les événements étendus s’applique à SQL Server, Azure SQL Database et Azure SQL Managed Instance.

Il est utile d’avoir une connaissance préalable des éléments suivants lorsque vous choisissez le fichier d’événements comme [cible](#AzureXEventsTargets):

- [Service Azure Storage](https://azure.microsoft.com/services/storage/)

- [Azure PowerShell avec Stockage Azure](/powershell/module/az.storage/)

## <a name="code-samples"></a>Exemples de code

Des rubriques connexes fournissent deux exemples de code :

- [Code cible de la mémoire tampon en anneau pour les événements étendus dans Azure SQL Database](xevent-code-ring-buffer.md)

  - Script Transact-SQL simple court.
  - Dans la rubrique concernant l’exemple de code, nous insistons sur le fait que, lorsque vous avez fini de traiter une cible de la mémoire tampon en anneau, vous devez en libérer les ressources en exécutant une instruction `ALTER EVENT SESSION ... ON DATABASE DROP TARGET ...;` alter-drop. Vous pouvez ensuite ajouter une autre instance de mémoire tampon en anneau avec l’instruction `ALTER EVENT SESSION ... ON DATABASE ADD TARGET ...`.

- [Code cible du fichier d’événements pour les événements étendus dans Azure SQL Database](xevent-code-event-file.md)

  - La Phase 1 est PowerShell pour créer un conteneur Azure Storage.
  - La Phase 2 est Transact-SQL qui utilise le conteneur Azure Storage.

## <a name="transact-sql-differences"></a>Différences Transact-SQL

- Lorsque vous exécutez la commande [CREATE EVENT SESSION](/sql/t-sql/statements/create-event-session-transact-sql) sur SQL Server, vous pouvez utiliser la clause **ON SERVER** . Mais sur Azure SQL Database, vous utilisez la clause **ON DATABASE** à la place.
- La clause **ON DATABASE** s’applique également aux commandes Transact-SQL [ALTER EVENT SESSION](/sql/t-sql/statements/alter-event-session-transact-sql) et [DROP EVENT SESSION](/sql/t-sql/statements/drop-event-session-transact-sql).

- Il est recommandé d’inclure l’option de session d’événement de **STARTUP_STATE = ON** dans vos instructions **CREATE EVENT SESSION** ou **ALTER EVENT SESSION**.
  - La valeur **= ON** prend en charge le redémarrage automatique après la reconfiguration de la base de données logique en raison d’un basculement.

## <a name="new-catalog-views"></a>Nouveaux affichages catalogue

La fonctionnalité des événements étendus est prise en charge par plusieurs [vues catalogue](/sql/relational-databases/system-catalog-views/catalog-views-transact-sql). Les vues catalogue vous indiquent les *métadonnées ou définitions* des sessions d’événements créées par l’utilisateur dans la base de données actuelle. Les affichages ne renvoient pas d’informations sur les instances des sessions d’événements actives.

| Nom de<br/>la vue catalogue | Description |
|:--- |:--- |
| **sys.database_event_session_actions** |Retourne une ligne pour chaque action d'un événement d'une session d'événements. |
| **sys.database_event_session_events** |Renvoie une ligne pour chaque événement d’une session d’événements. |
| **sys.database_event_session_fields** |Renvoie une ligne pour chaque colonne pouvant être personnalisée définie explicitement sur les événements et les cibles. |
| **sys.database_event_session_targets** |Retourne une ligne pour chaque cible d'événement d'une session d'événements. |
| **sys.database_event_sessions** |Retourne une ligne pour chaque session d’événements dans la base de données. |

Dans Microsoft SQL Server, les noms des vues catalogue similaires contiennent *.server\_* au lieu de *.database\_* . Les noms suivent le modèle **sys.server_event_%** .

## <a name="new-dynamic-management-views-dmvs"></a>Nouvelles vues de gestion dynamique [(DMV)](/sql/relational-databases/system-dynamic-management-views/system-dynamic-management-views)

Azure SQL Database a des [vues de gestion dynamique (DMV)](/sql/relational-databases/system-dynamic-management-views/extended-events-dynamic-management-views) qui prennent en charge les événements étendus. Les vues de gestion dynamique vous renseignent sur les sessions d’événements *actives* .

| Nom de la vue de gestion dynamique | Description |
|:--- |:--- |
| **sys.dm_xe_database_session_event_actions** |Retourne des informations sur les actions de la session d'événements. |
| **sys.dm_xe_database_session_events** |Retourne des informations sur les événements de la session. |
| **sys.dm_xe_database_session_object_columns** |Indique les valeurs de configuration d'objets liés à une session. |
| **sys.dm_xe_database_session_targets** |Renvoie des informations sur les cibles d’une session d’événements. |
| **sys.dm_xe_database_sessions** |Renvoie une ligne pour chaque session d’événements incluse dans la base de données actuelle. |

Dans Microsoft SQL Server, les noms des vues catalogue similaires ne contiennent pas la partie *\_database*, par exemple :

- **sys.dm_xe_sessions**, au lieu de name<br/>**sys.dm_xe_database_sessions**.

### <a name="dmvs-common-to-both"></a>Vues de gestion dynamique communes aux deux produits

Pour les événements étendus, des vues de gestion dynamique supplémentaires sont communes à Azure SQL Database, Azure SQL Managed Instance et Microsoft SQL Server :

- **sys.dm_xe_map_values**
- **sys.dm_xe_object_columns**
- **sys.dm_xe_objects**
- **sys.dm_xe_packages**

<a name="sqlfindseventsactionstargets" id="sqlfindseventsactionstargets"></a>

## <a name="find-the-available-extended-events-actions-and-targets"></a>Rechercher les actions, cibles et événements étendus disponibles

Vous pouvez exécuter une simple instruction SQL **SELECT** pour obtenir une liste des événements, actions et cibles disponibles.

```sql
SELECT
        o.object_type,
        p.name         AS [package_name],
        o.name         AS [db_object_name],
        o.description  AS [db_obj_description]
    FROM
                   sys.dm_xe_objects  AS o
        INNER JOIN sys.dm_xe_packages AS p  ON p.guid = o.package_guid
    WHERE
        o.object_type in
            (
            'action',  'event',  'target'
            )
    ORDER BY
        o.object_type,
        p.name,
        o.name;
```

<a name="AzureXEventsTargets" id="AzureXEventsTargets"></a> &nbsp;

## <a name="targets-for-your-azure-sql-database-event-sessions"></a>Cibles de vos sessions d’événements Azure SQL Database

Voici les cibles capables de capturer des résultats à partir de vos sessions d’événements sur Azure SQL Database :

- [Cible de mémoire tampon en anneau](/previous-versions/sql/sql-server-2016/bb630339(v=sql.130)) -Maintient brièvement les données d’événements en mémoire.
- [Cible de compteur d’événements](/previous-versions/sql/sql-server-2016/ff878025(v=sql.130)) - Compte tous les événements qui se produisent pendant une session d’événements étendus.
- [Cible du fichier d’événements](/previous-versions/sql/sql-server-2016/ff878115(v=sql.130)) - Écrit les mémoires tampons complètes dans un conteneur Azure Storage.

L’API [ETW (suivi d’événements pour Windows)](/dotnet/framework/wcf/samples/etw-tracing) n’est pas disponible pour les événements étendus sur Azure SQL Database.

## <a name="restrictions"></a>Restrictions

Il existe certaines différences liées à la sécurité qui conviennent à l’environnement cloud d’Azure SQL Database :

- Les événements étendus sont fondés sur le modèle d’isolement à locataire unique. Une session d’événements dans une base de données ne peut pas accéder aux données ou événements d’une autre base de données.
- Vous ne pouvez pas émettre une instruction **CREATE EVENT SESSION** dans le contexte de la base de données **master**.

## <a name="permission-model"></a>Modèle d’autorisation

Vous devez disposer de l’autorisation **Contrôle** sur la base de données pour émettre une instruction **CREATE EVENT SESSION**. Le propriétaire de la base de données (dbo) possède l’autorisation **Contrôle** .

### <a name="storage-container-authorizations"></a>Autorisations de conteneur de stockage

Le jeton SAP que vous générez pour votre conteneur Azure Storage doit spécifier **rwl** pour les autorisations. La valeur **rwl** fournit les autorisations suivantes :

- Lire
- Write
- List

## <a name="performance-considerations"></a>Considérations relatives aux performances

Il existe des scénarios où l’utilisation intensive des événements étendus peut accumuler plus de mémoire active qu’il n’en faut pour l’intégrité de l’ensemble du système. Par conséquent, Azure SQL Database définit et ajuste dynamiquement les limites de quantité de mémoire active pouvant être accumulée par une session d’événements. De nombreux facteurs entrent dans le calcul dynamique.

Si vous recevez un message d’erreur indiquant qu’une quantité maximale de mémoire a été appliquée, vous pouvez envisager certaines actions correctives, notamment :

- Exécuter moins de sessions d’événement simultanées.
- À l’aide des instructions **CREATE** et **ALTER** pour les sessions d’événements, réduisez la quantité de mémoire spécifiée dans la clause **MAX\_MEMORY**.

### <a name="network-latency"></a>Latence du réseau

La cible **Fichier d’événement** peut rencontrer une latence ou des problèmes de réseau lors de la persistance de données sur des objets blob Azure Storage. D’autres événements dans Azure SQL Database peuvent être retardés en attendant la fin de la communication réseau. Ce délai peut ralentir votre charge de travail.

- Pour atténuer ce risque de baisse des performances, évitez de définir l’option **EVENT_RETENTION_MODE** sur **NO_EVENT_LOSS** dans vos définitions de session d’événements.

## <a name="related-links"></a>Liens connexes

- [Utilisation d’Azure PowerShell avec Azure Storage](/powershell/module/az.storage/)
- [Applets de commande Azure Storage](/powershell/module/Azure.Storage)
- [Utilisation d'Azure PowerShell avec Azure Storage](/powershell/module/az.storage/)
- [Utilisation du stockage d’objets blob à partir de .NET](../../storage/blobs/storage-quickstart-blobs-dotnet.md)
- [CREATE CREDENTIAL (Transact-SQL)](/sql/t-sql/statements/create-credential-transact-sql)
- [CREATE EVENT SESSION (Transact-SQL)](/sql/t-sql/statements/create-event-session-transact-sql)
- [Billets de blog de Jonathan Kehayias sur les événements étendus dans Microsoft SQL Server](https://www.sqlskills.com/blogs/jonathan/category/extended-events/)
- La page web *Mises à jour de service* Azure, restreinte par paramètre à Azure SQL Database :
  - [https://azure.microsoft.com/updates/?service=sql-database](https://azure.microsoft.com/updates/?service=sql-database)

<!--
('lock_acquired' event.)

- Code sample for SQL Server: [Determine Which Queries Are Holding Locks](/sql/relational-databases/extended-events/determine-which-queries-are-holding-locks)
- Code sample for SQL Server: [Find the Objects That Have the Most Locks Taken on Them](/sql/relational-databases/extended-events/find-the-objects-that-have-the-most-locks-taken-on-them)
-->