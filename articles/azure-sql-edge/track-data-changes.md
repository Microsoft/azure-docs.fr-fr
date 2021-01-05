---
title: Suivre les modifications de données dans Azure SQL Edge
description: Découvrez le suivi des modifications et la capture des changements de données dans Azure SQL Edge.
keywords: ''
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 05/19/2020
ms.openlocfilehash: 86e84c60aec99246f58b5dc9d67584b23a3969f3
ms.sourcegitcommit: 0ce1ccdb34ad60321a647c691b0cff3b9d7a39c8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/05/2020
ms.locfileid: "93394921"
---
# <a name="track-data-changes-in-azure-sql-edge"></a>Suivre les modifications de données dans Azure SQL Edge

Azure SQL Edge prend en charge les deux fonctionnalités SQL Server qui suivent les modifications apportées aux données d’une base de données : [suivi des modifications](/sql/relational-databases/track-changes/track-data-changes-sql-server#Tracking) et [capture des changements de données](/sql/relational-databases/track-changes/track-data-changes-sql-server#Capture). Ces fonctionnalités permettent aux applications de déterminer les changements de langage de modification de données (opérations d’insertion, de mise à jour et de suppression) apportées aux tables utilisateur dans une base de données. Vous pouvez activer la capture de données modifiées et le suivi des modifications sur la même base de données. Aucune considération particulière ne s'applique.

Certaines applications doivent, pour être efficaces, être en mesure de rechercher les données qui ont été modifiées dans une base de données. En règle générale, les développeurs d'applications doivent, pour identifier les données modifiées, implémenter une méthode de suivi personnalisée dans leurs applications en utilisant une combinaison de déclencheurs, de colonnes d'horodateur et de tables supplémentaires. La création de ces applications demande généralement un effort considérable, aboutit à des mises à jour de schéma et se traduit souvent par une importante diminution des performances.

Dans le cas d’une solution IoT, où vous devez déplacer régulièrement des données de la périphérie vers un cloud ou centre de données, le suivi des modifications peut s’avérer particulièrement utile. Les utilisateurs peuvent rapidement et efficacement interroger uniquement les modifications à partir de la dernière synchronisation et charger ces modifications sur le cloud ou centre de données cible. Pour plus d’informations, consultez [Avantages liés à l’utilisation de la capture de données modifiées ou du suivi des modifications](/sql/relational-databases/track-changes/track-data-changes-sql-server#benefits-of-using-change-data-capture-or-change-tracking). 

Ces deux fonctionnalités ne sont pas les mêmes. Pour plus d’informations, consultez [Différences de fonctionnalités entre la capture de données modifiées et le suivi des modifications](/sql/relational-databases/track-changes/track-data-changes-sql-server#feature-differences-between-change-data-capture-and-change-tracking).

## <a name="change-data-capture"></a>Capture des données modifiées

Pour comprendre le fonctionnement de cette fonctionnalité, consultez [À propos de la capture des changements de données](/sql/relational-databases/track-changes/about-change-data-capture-sql-server).

Pour savoir comment activer ou désactiver cette fonctionnalité, consultez [Activer et désactiver la capture des changements de données](/sql/relational-databases/track-changes/enable-and-disable-change-data-capture-sql-server).

Pour administrer et superviser cette fonctionnalité, consultez [Administrer et surveiller la capture de données modifiées (SQL Server)](/sql/relational-databases/track-changes/administer-and-monitor-change-data-capture-sql-server).

Pour savoir comment interroger et utiliser les données modifiées, consultez [Utiliser des données modifiées](/sql/relational-databases/track-changes/work-with-change-data-sql-server).

## <a name="change-tracking"></a>Suivi des modifications

Pour comprendre le fonctionnement de cette fonctionnalité, consultez [À propos du suivi des modifications](/sql/relational-databases/track-changes/about-change-tracking-sql-server).

Pour savoir comment activer ou désactiver cette fonctionnalité, consultez [Activer et désactiver le suivi des modifications](/sql/relational-databases/track-changes/enable-and-disable-change-tracking-sql-server).

Pour administrer, superviser et gérer cette fonctionnalité, consultez [Administrer et superviser le suivi des modifications](/sql/relational-databases/track-changes/manage-change-tracking-sql-server).

Pour savoir comment interroger et utiliser les données modifiées, consultez [Utiliser des données modifiées](/sql/relational-databases/track-changes/work-with-change-tracking-sql-server).

## <a name="temporal-tables"></a>Tables temporelles

Azure SQL Edge prend également en charge la fonctionnalité des tables temporelles de SQL Server. Cette fonctionnalité (également appelée *tables temporelles versionnées par le système* ) offre une prise en charge intégrée de la remise d’informations sur les données stockées dans la table à tout moment. La fonctionnalité ne fournit pas simplement des informations sur les seules données qui sont actuellement correctes.

Une table temporelle versionnée par le système est un type de table utilisateur conçu pour conserver un historique complet des modifications apportées aux données et pour permettre l’analyse à un point dans le temps. Ce type de table temporelle est appelée table temporelle versionnée par le système, car la période de validité de chaque ligne est gérée par le système (à avoir, le moteur de base de données).

Chaque table temporelle contient deux colonnes définies explicitement, chacune d’elles contenant un type de données `datetime2`. Ces colonnes sont appelées colonnes de *période*. Le système utilise ces colonnes de période de manière exclusive pour enregistrer la période de validité de chaque ligne quand une ligne est modifiée.

En plus de ces colonnes de période, une table temporelle contient également une référence à une autre table avec un schéma en miroir. Le système utilise cette table pour stocker automatiquement la version précédente de la ligne chaque fois qu’une ligne de la table temporelle est mise à jour ou supprimée. Cette table supplémentaire est appelée table d’ *historique*. La table principale qui stocke les versions de ligne actuelles (réelles) est appelée table *actuelle* ou simplement table temporelle. Lors de la création d’une table temporelle, les utilisateurs peuvent spécifier une table d’historique existante (qui doit être compatible avec le schéma) ou laisser le système créer la table d’historique par défaut.

Pour plus d’informations, voir [Tables temporelles](/sql/relational-databases/tables/temporal-tables).

## <a name="next-steps"></a>Étapes suivantes

- [Streaming de données dans Azure SQL Edge](stream-data.md)
- [Machine Learning et IA avec ONNX dans Azure SQL Edge](onnx-overview.md)
- [Configurer la réplication sur Azure SQL Edge](configure-replication.md)
- [Sauvegarder et restaurer des bases de données dans Azure SQL Edge](backup-restore.md)