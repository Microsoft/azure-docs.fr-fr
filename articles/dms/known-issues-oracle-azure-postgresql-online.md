---
title: 'Problèmes connus : Migrer depuis Oracle vers Azure Database pour PostgreSQL'
titleSuffix: Azure Database Migration Service
description: Découvrez les problèmes connus et les limitations de migration avec les migrations en ligne d’Oracle vers Azure Database pour PostgreSQL - Serveur unique, à l’aide d’Azure Database Migration Service.
services: database-migration
author: arunkumarthiags
ms.author: arthiaga
manager: craigg
ms.reviewer: craigg
ms.service: dms
ms.workload: data-services
ms.custom: seo-lt-2019
ms.topic: troubleshooting
ms.date: 05/20/2020
ms.openlocfilehash: 1b331f8e0af452937028c63fba123cb92f57a6b0
ms.sourcegitcommit: cd9754373576d6767c06baccfd500ae88ea733e4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/20/2020
ms.locfileid: "94962414"
---
# <a name="known-issuesmigration-limitations-with-online-migrations-from-oracle-to-azure-db-for-postgresql-single-server"></a>Problèmes connus/limitations de migration dans le cadre des migrations en ligne d'Oracle vers Azure DB pour PostgreSQL - Serveur unique.

Les sections suivantes décrivent les problèmes connus et limitations associés aux migrations en ligne d'Oracle vers Azure Database pour PostgreSQL - Serveur unique.

## <a name="oracle-versions-supported-as-a-source-database"></a>Versions d'Oracle prises en charge en tant que base de données source

Azure Database Migration Service prend en charge la connexion à :

- Oracle versions 10g, 11g et 12c.
- Oracle Enterprise, Standard, Express et Personal Edition.

Azure Database Migration Service ne prend pas en charge la connexion aux bases de données de conteneur mutualisées.

## <a name="postgresql-versions-supported-as-a-target-database"></a>Versions de PostgreSQL prises en charge en tant que base de données cible

Azure Database Migration Service prend en charge les migrations vers Azure Database pour PostgreSQL - Serveur unique versions 9.5, 9.6, 10 et 11. Pour plus d’informations sur la prise en charge des versions dans Azure Database pour PostgreSQL - Serveur unique, consultez l’article [Versions prises en charge de la base de données PostgreSQL](../postgresql/concepts-supported-versions.md).

## <a name="datatype-limitations"></a>Limitations relatives au type de données

Les types de données suivants **ne seront pas** migrés :

- BFILE
- ROWID
- REF
- UROWID
- ANYDATA
- SDO_GEOMETRY
- Tables imbriquées
- Types de données définis par l'utilisateur
- Notes
- Colonnes virtuelles
- Vues matérialisées basées sur la colonne ROWID

De plus, les colonnes BLOB/CLOB vides sont mappées sur NULL dans la cible.

## <a name="lob-limitations"></a>Limitations relatives aux objets LOB

- Lorsque le mode LOB à taille limitée est activé, les LOB vides de la source Oracle sont répliqués en tant que valeurs NULL.
- Les noms d’objets longs (plus de 30 octets) ne sont pas pris en charge.
- Les données des colonnes LONG et LONG RAW ne peuvent pas dépasser 64 Ko. Au-delà de 64 Ko, les données sont tronquées.
- Dans Oracle 12 uniquement, les modifications apportées aux colonnes LOB ne sont pas prises en charge (migrées).
- Les mises à jour des colonnes XMLTYPE et LOB ne sont pas prises en charge (migrées).

## <a name="known-issues-and-limitations"></a>Problèmes connus et limitations

- L’utilisateur doit disposer du privilège DBA sur le serveur Oracle.
- Les modifications de données résultant d’opérations de partition/sous-partition (ADD, DROP, EXCHANGE et TRUNCATE) ne sont pas migrées et peuvent entraîner les erreurs suivantes :
  - Pour les opérations ADD, les mises à jour et les suppressions ayant trait aux données ajoutées peuvent renvoyer un avertissement « 0 lignes affectées ».
  - Pour les opérations DROP et TRUNCATE, les nouvelles insertions peuvent entraîner des erreurs de « doublons ».
  - Pour les opérations EXCHANGE, des erreurs « 0 lignes affectées » et « doublons » peuvent se produire.
- Les tables dont les noms contiennent des apostrophes ne peuvent pas être répliquées.