---
title: Colocation de tables – Hyperscale (Citus) – Azure Database pour PostgreSQL
description: Guide pratique pour stocker ensemble des informations associées afin d’accélérer les requêtes
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.topic: conceptual
ms.date: 05/06/2019
ms.openlocfilehash: 842563319e09a001fd6e85403d8aee6fb14690ee
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90884427"
---
# <a name="table-colocation-in-azure-database-for-postgresql--hyperscale-citus"></a>Colocation de tables dans Azure Database pour PostgreSQL - Hyperscale (Citus)

La colocation signifie que vous stockez ensemble des informations connexes sur les mêmes nœuds. Les requêtes peuvent s'exécuter rapidement lorsque toutes les données nécessaires sont disponibles sans aucun trafic réseau. La colocation de données associées sur différents nœuds permet aux requêtes de s’exécuter efficacement en parallèle sur chaque nœud.

## <a name="data-colocation-for-hash-distributed-tables"></a>Colocation de données pour les tables distribuées par hachage

Dans Azure Database pour PostgreSQL – Hyperscale (Citus), une ligne est stockée au sein d'une partition si le hachage de la valeur de la colonne de distribution se situe dans la plage de hachage de la partition. Les partitions avec la même plage de hachage sont toujours placées sur le même nœud. Les lignes avec des valeurs de colonne de distribution égales sont toujours placées sur le même nœud dans toutes les tables.

:::image type="content" source="media/concepts-hyperscale-colocation/colocation-shards.png" alt-text="Partitions":::

## <a name="a-practical-example-of-colocation"></a>Exemple pratique de colocation

Imaginez les tableaux suivants, qui peuvent faire partie d’une solution SaaS d’analyse web mutualisée :

```sql
CREATE TABLE event (
  tenant_id int,
  event_id bigint,
  page_id int,
  payload jsonb,
  primary key (tenant_id, event_id)
);

CREATE TABLE page (
  tenant_id int,
  page_id int,
  path text,
  primary key (tenant_id, page_id)
);
```

Maintenant, nous voulons répondre aux requêtes qui peuvent être émises par un tableau de bord orienté client. Exemple de requête : « Renvoyer le nombre de visites la semaine dernière pour toutes les pages commençant par « /blog » dans le locataire six. »

Si nos données se trouvaient dans l'option de déploiement Serveur unique, nous pourrions facilement exprimer notre requête à l'aide de l'ensemble complet d'opérations relationnelles offertes par SQL :

```sql
SELECT page_id, count(event_id)
FROM
  page
LEFT JOIN  (
  SELECT * FROM event
  WHERE (payload->>'time')::timestamptz >= now() - interval '1 week'
) recent
USING (tenant_id, page_id)
WHERE tenant_id = 6 AND path LIKE '/blog%'
GROUP BY page_id;
```

Tant que la [plage de travail](https://en.wikipedia.org/wiki/Working_set) pour cette requête tient en mémoire, une table de serveur unique est une solution appropriée. Prenons l'exemple d'opportunités de mise à l'échelle du modèle de données avec l'option de déploiement Hyperscale (Citus).

### <a name="distribute-tables-by-id"></a>Distribuer les tables par ID

Les requêtes de serveur unique commencent à ralentir lorsque le nombre de locataires et les données stockées pour chaque locataire augmentent. La plage de travail ne tient plus en mémoire, et l’UC devient un goulot d’étranglement.

Dans ce cas, nous pouvons partitionner les données sur de nombreux nœuds à l'aide d'Hyperscale (Citus). Tout d'abord, en matière de partitionnement, nous devons choisir la colonne de distribution. Commençons naïvement par choisir `event_id` pour la table d’événements et `page_id` pour la table `page` :

```sql
-- naively use event_id and page_id as distribution columns

SELECT create_distributed_table('event', 'event_id');
SELECT create_distributed_table('page', 'page_id');
```

Lorsque les données sont dispersées sur différents workers, nous ne pouvons pas effectuer de jointure, comme nous le ferions sur un seul nœud PostgreSQL. Au lieu de cela, nous devons émettre deux requêtes :

```sql
-- (Q1) get the relevant page_ids
SELECT page_id FROM page WHERE path LIKE '/blog%' AND tenant_id = 6;

-- (Q2) get the counts
SELECT page_id, count(*) AS count
FROM event
WHERE page_id IN (/*…page IDs from first query…*/)
  AND tenant_id = 6
  AND (payload->>'time')::date >= now() - interval '1 week'
GROUP BY page_id ORDER BY count DESC LIMIT 10;
```

Ensuite, l’application doit combiner les résultats des deux étapes.

Les requêtes exécutées doivent consulter les données dans les partitions réparties entre les nœuds.

:::image type="content" source="media/concepts-hyperscale-colocation/colocation-inefficient-queries.png" alt-text="Partitions":::

Dans ce cas, la distribution des données présente de sérieux inconvénients :

-   Surcharge lors de l'interrogation de chaque partition et exécution de plusieurs requêtes.
-   Surcharge de Q1, renvoyant beaucoup de lignes au client.
-   Q2 devient volumineux.
-   Le besoin d'écrire des requêtes en plusieurs étapes nécessite des modifications dans l'application.

Comme les données sont dispersées, les requêtes peuvent être parallélisées. Ce n'est utile que si la quantité de travail effectuée par la requête est nettement supérieure à la surcharge que représente l'interrogation de plusieurs partitions.

### <a name="distribute-tables-by-tenant"></a>Distribuer les tables par locataire

Dans Hyperscale (Citus), les lignes dotées de la même valeur de colonne de distribution sont obligatoirement situées sur le même nœud. Si nous recommençons, nous pouvons créer nos tables avec `tenant_id` comme colonne de distribution.

```sql
-- co-locate tables by using a common distribution column
SELECT create_distributed_table('event', 'tenant_id');
SELECT create_distributed_table('page', 'tenant_id', colocate_with => 'event');
```

Désormais, Hyperscale (Citus) peut répondre à la requête de serveur unique d'origine sans modification (Q1) :

```sql
SELECT page_id, count(event_id)
FROM
  page
LEFT JOIN  (
  SELECT * FROM event
  WHERE (payload->>'time')::timestamptz >= now() - interval '1 week'
) recent
USING (tenant_id, page_id)
WHERE tenant_id = 6 AND path LIKE '/blog%'
GROUP BY page_id;
```

En raison du filtre et de la jointure sur tenant_id, Hyperscale (Citus) sait que l'intégralité de la requête peut être traitée à l'aide de l'ensemble de partitions colocalisées qui contiennent les données de ce locataire particulier. Un seul nœud PostgreSQL peut répondre à la requête en une seule étape.

:::image type="content" source="media/concepts-hyperscale-colocation/colocation-better-query.png" alt-text="Partitions":::

Dans certains cas, les requêtes et les schémas de table doivent être modifiés pour inclure l’ID de locataire dans des conditions de jointure et des contraintes uniques. Cette modification est généralement simple.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment les données des locataires sont colocalisées dans le [didacticiel mutualisé](tutorial-design-database-hyperscale-multi-tenant.md).
