---
title: Utiliser sys_schema - Azure Database pour MySQL
description: Découvrez comment utiliser sys_schema pour rechercher des problèmes de performances et assurer la maintenance d’une base de données dans Azure Database pour MySQL.
author: savjani
ms.author: pariks
ms.service: mysql
ms.topic: troubleshooting
ms.date: 3/30/2020
ms.openlocfilehash: a20510ee2800a54f9a51a2f498ee8ae8a3e51d55
ms.sourcegitcommit: 6ab718e1be2767db2605eeebe974ee9e2c07022b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/12/2020
ms.locfileid: "94543147"
---
# <a name="how-to-use-sys_schema-for-performance-tuning-and-database-maintenance-in-azure-database-for-mysql"></a>Guide pratique pour utiliser sys_schema à des fins de réglage des performances et de maintenance de base de données dans Azure Database pour MySQL

MySQL performance_schema, disponible initialement dans MySQL 5.5, fournit l’instrumentation pour bon nombre de ressources serveur vitales, telles que l’allocation de mémoire, les programmes stockés, le verrouillage des métadonnées, etc. Or, performance_schema contient plus de 80 tables, et pour obtenir les informations nécessaires, il est souvent utile de joindre les tables de performance_schema, ainsi que les tables d’information_schema. En s’appuyant sur both performance_schema et information_schema, sys_schema propose une collection considérable de [vues conviviales](https://dev.mysql.com/doc/refman/5.7/en/sys-schema-views.html) dans une base de données en lecture seule que vous pouvez tout à fait utiliser dans Azure Database pour MySQL version 5.7.

:::image type="content" source="./media/howto-troubleshoot-sys-schema/sys-schema-views.png" alt-text="vues de sys_schema":::

Il existe 52 vues dans le sys_schema, et chaque vue présente l’un des préfixes suivants :

- Host_summary ou IO : latences liées aux E/S.
- InnoDB : état de la mémoire tampon et verrous InnoDB.
- Mémoire : utilisation de la mémoire par l’hôte et les utilisateurs.
- Schéma : informations relatives au schéma, telles que l’incrémentation automatique, les index, etc.
- Statement : informations sur les instructions SQL ; il peut s’agir d’une instruction qui a entraîné une analyse de table complète ou une durée de requête longue.
- Utilisateur : ressources consommées et regroupées par utilisateur. Il peut s’agir par exemple d’E/S de fichiers, de connexions ou de mémoire.
- Wait : événements d’attente regroupés par hôte ou utilisateur.

À présent, intéressons-nous à quelques modèles d’utilisation courants de sys_schema. Pour commencer, nous allons regrouper les modèles d’utilisation dans deux catégories : **réglage des performances** et **maintenance de base de données**.

## <a name="performance-tuning"></a>Réglage des performances

### <a name="sysuser_summary_by_file_io"></a>*sys.user_summary_by_file_io*

Les E/S représentent l’opération la plus coûteuse de la base de données. Nous pouvons déterminer la latence moyenne des E/S en interrogeant la vue *sys.user_summary_by_file_io*. Avec par défaut 125 Go de stockage provisionné, la latence des E/S est d’environ 15 secondes.

:::image type="content" source="./media/howto-troubleshoot-sys-schema/io-latency-125GB.png" alt-text="Latence des E/S : 125 Go":::

Dans la mesure où Azure Database pour MySQL adapte les E/S en fonction du stockage, après augmentation de mon stockage provisionné à 1 To, la latence des E/S se réduit à 571 ms.

:::image type="content" source="./media/howto-troubleshoot-sys-schema/io-latency-1TB.png" alt-text="Latence des E/S : 1 To":::

### <a name="sysschema_tables_with_full_table_scans"></a>*sys.schema_tables_with_full_table_scans*

En dépit d’une planification minutieuse, de nombreuses requêtes peuvent donner lieu à des analyses de table complète. Pour obtenir des informations supplémentaires sur les types d’index et sur la façon de les optimiser, vous pouvez consulter cet article : [Guide pratique pour résoudre les problèmes de performances des requêtes](./howto-troubleshoot-query-performance.md). Les analyses de table complète sont gourmandes en ressources et dégradent les performances de votre base de données. Le moyen le plus rapide de rechercher des tables avec une analyse de table complète est d’interroger la vue *sys.schema_tables_with_full_table_scans*.

:::image type="content" source="./media/howto-troubleshoot-sys-schema/full-table-scans.png" alt-text="analyses de table complète":::

### <a name="sysuser_summary_by_statement_type"></a>*sys.user_summary_by_statement_type*

Pour résoudre les problèmes de performances de base de données, il peut être utile d’identifier les événements qui se produisent à l’intérieur de votre base de données, ce que permet la vue *sys.user_summary_by_statement_type*.

:::image type="content" source="./media/howto-troubleshoot-sys-schema/summary-by-statement.png" alt-text="résumé par instruction":::

Dans cet exemple, Azure Database pour MySQL a passé 53 minutes à vider le journal des requêtes lentes à 44 579 reprises. Cela représente beaucoup de temps et un grand nombre d’E/S. Vous pouvez réduire cette activité en désactivant votre journal des requêtes lentes ou en diminuant la fréquence du journal des requêtes lentes dans le portail Azure.

## <a name="database-maintenance"></a>Maintenance de base de données

### <a name="sysinnodb_buffer_stats_by_table"></a>*sys.innodb_buffer_stats_by_table*

[!IMPORTANT]
> L’interrogation de cette vue peut avoir un impact sur les performances. Il est recommandé d’effectuer cette résolution des problèmes pendant les heures creuses.

Le pool de mémoires tampons InnoDB réside en mémoire et constitue le principal mécanisme de cache entre le SGBD et le stockage. La taille du pool de mémoires tampons InnoDB est liée au niveau de performances et ne peut pas être changée, à moins de choisir une autre référence SKU de produit. Comme pour la mémoire de votre système d’exploitation, les pages anciennes sont écartées pour faire place à des données plus récentes. Pour identifier les tables qui consomment la majeure partie de la mémoire du pool de mémoires tampons InnoDB, vous pouvez interroger la vue *sys.innodb_buffer_stats_by_table*.

:::image type="content" source="./media/howto-troubleshoot-sys-schema/innodb-buffer-status.png" alt-text="état de la mémoire tampon InnoDB":::

Dans le schéma ci-dessus, il apparaît qu’en dehors des tables et vues système, chaque table de la base de données mysqldatabase033, qui héberge l’un de mes sites WordPress, occupe 16 Ko, soit 1 page, de données en mémoire.

### <a name="sysschema_unused_indexes--sysschema_redundant_indexes"></a>*Sys.schema_unused_indexes* & *sys.schema_redundant_indexes*

Les index sont des outils efficaces pour améliorer les performances de lecture, mais ils induisent des coûts supplémentaires en termes d’insertions et de stockage. *Sys.schema_unused_indexes* et *sys.schema_redundant_indexes* donnent des indications sur les index non utilisés ou en double.

:::image type="content" source="./media/howto-troubleshoot-sys-schema/unused-indexes.png" alt-text="index non utilisés":::

:::image type="content" source="./media/howto-troubleshoot-sys-schema/redundant-indexes.png" alt-text="index redondants":::

## <a name="conclusion"></a>Conclusion

En résumé, sys_schema est un outil efficace à la fois pour le réglage des performances et la maintenance de base de données. Veillez à tirer parti de cette fonctionnalité dans Azure Database pour MySQL. 

## <a name="next-steps"></a>Étapes suivantes
- Pour consulter les réponses d’homologues aux questions qui vous préoccupent le plus ou pour poster une nouvelle question/réponse, visitez la [page de questions Microsoft Q&A](/answers/topics/azure-database-mysql.html) ou [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-database-mysql).