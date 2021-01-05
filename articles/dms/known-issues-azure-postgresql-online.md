---
title: 'Problèmes connus : Migrations en ligne de PostgreSQL vers Azure Database pour PostgreSQL'
titleSuffix: Azure Database Migration Service
description: Découvrez les problèmes connus et les limitations relatifs aux migrations en ligne de PostgreSQL vers Azure Database pour PostgreSQL à l’aide d’Azure Database Migration Service.
services: database-migration
author: arunkumarthiags
ms.author: arthiaga
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom:
- seo-lt-2019
- seo-dt-2019
ms.topic: troubleshooting
ms.date: 02/20/2020
ms.openlocfilehash: 8c3c1d28a7fbb3e3c9c449feb03a75d48178b718
ms.sourcegitcommit: e15c0bc8c63ab3b696e9e32999ef0abc694c7c41
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97609035"
---
# <a name="known-issuesmigration-limitations-with-online-migrations-from-postgresql-to-azure-db-for-postgresql"></a>Problèmes connus et limitations concernant les migrations en ligne de PostgreSQL vers Azure DB pour PostgreSQL

Les sections suivantes décrivent les problèmes connus et limitations associés aux migrations en ligne de PostgreSQL vers Azure Database pour PostgreSQL.

## <a name="online-migration-configuration"></a>Configuration d’une migration en ligne

- Le serveur PostgreSQL source doit exécuter la version 9.4, 9.5, 9.6, 10 ou 11. Pour plus d’informations, consultez l’article [Versions de bases de données PostgreSQL prises en charge](../postgresql/concepts-supported-versions.md).
- Seules les migrations vers une version identique ou supérieure sont prises en charge. Par exemple, la migration de PostgreSQL 9.5 vers Azure Database pour PostgreSQL 9.6 ou 10 est prise en charge, mais la migration de PostgreSQL 11 vers PostgreSQL 9.6 ne l’est pas.
- Pour activer la réplication logique dans le fichier **postgresql.config PostgreSQL source**, définissez les paramètres suivants :
  - **wal_level** = logique
  - **max_replication_slots** = [at least max number of databases for migration] ; si vous voulez migrer quatre bases de données, définissez la valeur sur 4 au minimum.
  - **max_wal_senders** = [nombre de bases de données exécutées simultanément] ; la valeur recommandée est 10
- Ajouter une adresse IP d’agent DMS au fichier pg_hba.conf PostgreSQL source
  1. Notez l’adresse IP DMS après avoir terminé le provisionnement d’une instance Azure Database Migration Service.
  2. Ajoutez l’adresse IP dans le fichier pg_hba.conf comme indiqué :

      ```
          host    all    172.16.136.18/10    md5
          host    replication postgres    172.16.136.18/10     md5
      ```

- L’utilisateur doit disposer du rôle REPLICATION sur le serveur qui héberge la base de données source.
- Les schémas des bases de données source et cible doivent correspondre.
- Le schéma de la base de données cible dans Azure Database pour PostgreSQL - Serveur unique ne doit pas inclure de clés étrangères. Pour supprimer les clés étrangères, utilisez la requête suivante :

    ```
                  SELECT Queries.tablename
           ,concat('alter table ', Queries.tablename, ' ', STRING_AGG(concat('DROP CONSTRAINT ', Queries.foreignkey), ',')) as DropQuery
                ,concat('alter table ', Queries.tablename, ' ', 
                                                STRING_AGG(concat('ADD CONSTRAINT ', Queries.foreignkey, ' FOREIGN KEY (', column_name, ')', 'REFERENCES ', foreign_table_name, '(', foreign_column_name, ')' ), ',')) as AddQuery
        FROM
        (SELECT
        tc.table_schema, 
        tc.constraint_name as foreignkey, 
        tc.table_name as tableName, 
        kcu.column_name, 
        ccu.table_schema AS foreign_table_schema,
        ccu.table_name AS foreign_table_name,
        ccu.column_name AS foreign_column_name 
    FROM 
        information_schema.table_constraints AS tc 
        JOIN information_schema.key_column_usage AS kcu
          ON tc.constraint_name = kcu.constraint_name
          AND tc.table_schema = kcu.table_schema
        JOIN information_schema.constraint_column_usage AS ccu
          ON ccu.constraint_name = tc.constraint_name
          AND ccu.table_schema = tc.table_schema
    WHERE constraint_type = 'FOREIGN KEY') Queries
      GROUP BY Queries.tablename;
    
    ```

    Exécutez la clé étrangère Drop (deuxième colonne) dans le résultat de la requête.

- Le schéma de la base de données cible dans Azure Database pour PostgreSQL - Serveur unique ne doit pas inclure de déclencheurs. Utilisez la commande suivante pour désactiver les déclencheurs dans la base de données cible :

     ```
    SELECT Concat('DROP TRIGGER ', Trigger_Name, ';') FROM  information_schema.TRIGGERS WHERE TRIGGER_SCHEMA = 'your_schema';
     ```

## <a name="size-limitations"></a>Limitations de taille
- Vous pouvez migrer jusqu’à 2 To de données de PostgreSQL vers Azure DB pour PostgreSQL à l’aide d’un seul service DMS.
## <a name="datatype-limitations"></a>Limitations relatives au type de données

  **Limitation** : S’il n’existe aucune clé primaire pour les tables, les modifications risquent de ne pas être synchronisées avec la base de données cible.

  **Solution de contournement** : définissez temporairement une clé primaire pour la table afin de poursuivre la migration. Vous pouvez supprimer la clé primaire à l’issue de la migration de données.

## <a name="limitations-when-migrating-online-from-aws-rds-postgresql"></a>Limitations lors de la migration en ligne depuis AWS RDS PostgreSQL

Lorsque vous essayez d’effectuer une migration en ligne depuis AWS RDS PostgreSQL vers Azure Database pour PostgreSQL, vous pouvez rencontrer les erreurs suivantes.

- **Erreur** : La valeur par défaut de la colonne '{colonne}' dans la table '{table}' de la base de données '{base de données}' est différente sur les serveurs source et cible. Elle est '{valeur sur la source}' sur la source et '{valeur sur la cible}' sur la cible.

  **Limitation** : Cette erreur se produit quand la valeur par défaut sur un schéma de colonne est différente entre les bases de données source et cible.
  **Solution de contournement** : Vérifiez que le schéma sur la cible correspond au schéma sur la source. Pour en savoir plus sur la migration de schéma, reportez-vous à la [documentation sur la migration en ligne Azure PostgreSQL](./tutorial-postgresql-azure-postgresql-online.md#migrate-the-sample-schema).

- **Erreur** : La base de données cible '{base de données}' a '{nombre de tables}' tables alors que la base de données source '{base de données}' en a '{nombre de tables}'. Les bases de données source et cible doivent avoir le même nombre de tables.

  **Limitation** : Cette erreur se produit quand le nombre de tables est différent entre les bases de données source et cible.

  **Solution de contournement** : Vérifiez que le schéma sur la cible correspond au schéma sur la source. Pour en savoir plus sur la migration de schéma, reportez-vous à la [documentation sur la migration en ligne Azure PostgreSQL](./tutorial-postgresql-azure-postgresql-online.md#migrate-the-sample-schema).

- **Erreur :** La base de données source {database} est vide.

  **Limitation** : Cette erreur se produit quand la base de données source est vide. Il est fort probable que vous ayez sélectionné la mauvaise base de données pour la source.

  **Solution de contournement** : Revérifiez la base de données source que vous avez sélectionnée pour la migration, puis réessayez.

- **Erreur :** La base de données cible {database} est vide. Migrez le schéma.

  **Limitation** : Cette erreur se produit quand il n’y a pas de schéma dans la base de données cible. Vérifiez que le schéma sur la cible correspond au schéma sur la source.
  **Solution de contournement** : Vérifiez que le schéma sur la cible correspond au schéma sur la source. Pour en savoir plus sur la migration de schéma, reportez-vous à la [documentation sur la migration en ligne Azure PostgreSQL](./tutorial-postgresql-azure-postgresql-online.md#migrate-the-sample-schema).

## <a name="other-limitations"></a>Autres limitations

- Le nom de la base de données ne peut pas contenir un point-virgule (;).
- Une table capturée doit avoir une clé primaire. Si une table n’a pas de clé primaire, le résultat des opérations d’enregistrement DELETE et UPDATE est imprévisible.
- La mise à jour d’un segment de clé primaire est ignorée. Dans ce cas, l’application d’une telle mise à jour sera identifiée par la cible comme une mise à jour n’ayant mis à jour aucune ligne et un enregistrement sera écran dans la table des exceptions.
- Le comportement d’une migration de plusieurs tables portant le même nom mais selon une casse différente (par exemple, table1, TABLE1 et Table1) peut-être imprévisible, elle n’est donc pas prise en charge.
- Le traitement des modifications des DDL de table [CREATE | ALTER | DROP | TRUNCATE] n’est pas pris en charge.
- Dans Azure Database Migration Service, une même activité de migration peut prendre en charge jusqu’à quatre bases de données.