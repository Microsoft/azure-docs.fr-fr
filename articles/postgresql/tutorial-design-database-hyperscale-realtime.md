---
title: 'Tutoriel : Concevoir un tableau de bord en temps réel - Hyperscale (Citus) - Azure Database pour PostgreSQL'
description: Ce tutoriel montre comment paralléliser un tableau de bord en temps avec Azure Database pour PostgreSQL - Hyperscale (Citus).
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.custom: mvc
ms.topic: tutorial
ms.date: 05/14/2019
ms.openlocfilehash: 480af87519f8e11c14f009058b0f518bdfc34a1b
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95026265"
---
# <a name="tutorial-design-a-real-time-analytics-dashboard-by-using-azure-database-for-postgresql--hyperscale-citus"></a>Tutoriel : Concevoir un tableau de bord analytique en temps réel avec Azure Database pour PostgreSQL – Hyperscale (Citus)

Dans ce tutoriel, vous utilisez Azure Database pour PostgreSQL - Hyperscale (Citus) pour apprendre à :

> [!div class="checklist"]
> * Créer un groupe de serveurs Hyperscale (Citus)
> * Utiliser l’utilitaire psql pour créer un schéma
> * Partitionner des tables entre des nœuds
> * Générer un exemple de données
> * Effectuer des cumuls
> * Interroger des données brutes et agrégées
> * Faire expirer des données

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [azure-postgresql-hyperscale-create-db](../../includes/azure-postgresql-hyperscale-create-db.md)]

## <a name="use-psql-utility-to-create-a-schema"></a>Utiliser l’utilitaire psql pour créer un schéma

Une fois que vous êtes connecté à Azure Database pour PostgreSQL - Hyperscale (Citus) à l’aide de psql, vous pouvez effectuer certaines tâches de base. Ce tutoriel vous guide lors de l’ingestion de données de trafic à partir d’analytiques web, puis du cumul des données pour fournir des tableaux de bord en temps réel basés sur ces données.

Vous allez créer une table qui consomme toutes vos données de trafic web brutes. Exécutez les commandes suivantes dans le terminal psql :

```sql
CREATE TABLE http_request (
  site_id INT,
  ingest_time TIMESTAMPTZ DEFAULT now(),

  url TEXT,
  request_country TEXT,
  ip_address TEXT,

  status_code INT,
  response_time_msec INT
);
```

Vous allez également créer une table qui conserve vos agrégats par minute et une table qui conserve la position de votre dernier cumul. Exécutez également les commandes suivantes dans psql :

```sql
CREATE TABLE http_request_1min (
  site_id INT,
  ingest_time TIMESTAMPTZ, -- which minute this row represents

  error_count INT,
  success_count INT,
  request_count INT,
  average_response_time_msec INT,
  CHECK (request_count = error_count + success_count),
  CHECK (ingest_time = date_trunc('minute', ingest_time))
);

CREATE INDEX http_request_1min_idx ON http_request_1min (site_id, ingest_time);

CREATE TABLE latest_rollup (
  minute timestamptz PRIMARY KEY,

  CHECK (minute = date_trunc('minute', minute))
);
```

Vous pouvez localiser les tables nouvellement créées dans la liste des tables avec cette commande psql :

```postgres
\dt
```

## <a name="shard-tables-across-nodes"></a>Partitionner des tables entre des nœuds

Un déploiement hyperscale stocke les lignes des tables sur des nœuds différents selon la valeur d’une colonne désignée par l’utilisateur. Cette « colonne de distribution » marque la façon dont les données sont partitionnées entre les nœuds.

Nous allons définir la colonne de distribution sur site\_id, la clé de partitionnement. Dans psql, exécutez les fonctions suivantes :

  ```sql
SELECT create_distributed_table('http_request',      'site_id');
SELECT create_distributed_table('http_request_1min', 'site_id');
```

[!INCLUDE [azure-postgresql-hyperscale-dist-alert](../../includes/azure-postgresql-hyperscale-dist-alert.md)]

## <a name="generate-sample-data"></a>Générer un exemple de données

Votre groupe de serveurs doit à présent être prêt à recevoir des données. Vous pouvez exécuter ce qui suit localement à partir de votre connexion `psql` pour insérer des données en continu.

```sql
DO $$
  BEGIN LOOP
    INSERT INTO http_request (
      site_id, ingest_time, url, request_country,
      ip_address, status_code, response_time_msec
    ) VALUES (
      trunc(random()*32), clock_timestamp(),
      concat('http://example.com/', md5(random()::text)),
      ('{China,India,USA,Indonesia}'::text[])[ceil(random()*4)],
      concat(
        trunc(random()*250 + 2), '.',
        trunc(random()*250 + 2), '.',
        trunc(random()*250 + 2), '.',
        trunc(random()*250 + 2)
      )::inet,
      ('{200,404}'::int[])[ceil(random()*2)],
      5+trunc(random()*150)
    );
    COMMIT;
    PERFORM pg_sleep(random() * 0.25);
  END LOOP;
END $$;
```

La requête insère environ huit lignes par seconde. Les lignes sont stockées sur différents nœuds Worker, comme indiqué par la colonne de distribution, `site_id`.

   > [!NOTE]
   > Continuez à exécuter la requête de génération de données, puis ouvrez une deuxième connexion psql pour les commandes restantes dans ce tutoriel.
   >

## <a name="query"></a>Requête

L’option d’hébergement hyperscale permet à plusieurs nœuds de traiter des requêtes en parallèle, ce qui accélère la procédure. Par exemple, la base de données calcule des agrégats tels que SUM (SOMME) et COUNT (NOMBRE) sur les nœuds Worker et combine les résultats dans une réponse finale.

Voici une requête qui compte les requêtes web par minute ainsi que quelques statistiques.
Essayez de l’exécuter dans psql pour voir les résultats.

```sql
SELECT
  site_id,
  date_trunc('minute', ingest_time) as minute,
  COUNT(1) AS request_count,
  SUM(CASE WHEN (status_code between 200 and 299) THEN 1 ELSE 0 END) as success_count,
  SUM(CASE WHEN (status_code between 200 and 299) THEN 0 ELSE 1 END) as error_count,
  SUM(response_time_msec) / COUNT(1) AS average_response_time_msec
FROM http_request
WHERE date_trunc('minute', ingest_time) > now() - '5 minutes'::interval
GROUP BY site_id, minute
ORDER BY minute ASC;
```

## <a name="rolling-up-data"></a>Cumuler les données

La requête précédente fonctionne bien dans les premières étapes, mais ses performances diminuent à mesure que la quantité de données augmente. Même avec un traitement distribué, il est plus rapide de précalculer les données que de les recalculer de façon répétée.

Vous pouvez vérifier que votre tableau de bord a conservé sa rapidité en cumulant régulièrement les données brutes dans une table d’agrégation. Vous pouvez tester la durée d’agrégation. Nous avons utilisé une table d’agrégation par minute, mais vous pouvez diviser les données par 5, 15 ou 60 minutes.

Pour exécuter ce cumul plus facilement, nous allons le placer dans une fonction plpgsql. Exécutez les commandes suivantes dans psql pour créer la fonction `rollup_http_request`.

```sql
-- initialize to a time long ago
INSERT INTO latest_rollup VALUES ('10-10-1901');

-- function to do the rollup
CREATE OR REPLACE FUNCTION rollup_http_request() RETURNS void AS $$
DECLARE
  curr_rollup_time timestamptz := date_trunc('minute', now());
  last_rollup_time timestamptz := minute from latest_rollup;
BEGIN
  INSERT INTO http_request_1min (
    site_id, ingest_time, request_count,
    success_count, error_count, average_response_time_msec
  ) SELECT
    site_id,
    date_trunc('minute', ingest_time),
    COUNT(1) as request_count,
    SUM(CASE WHEN (status_code between 200 and 299) THEN 1 ELSE 0 END) as success_count,
    SUM(CASE WHEN (status_code between 200 and 299) THEN 0 ELSE 1 END) as error_count,
    SUM(response_time_msec) / COUNT(1) AS average_response_time_msec
  FROM http_request
  -- roll up only data new since last_rollup_time
  WHERE date_trunc('minute', ingest_time) <@
          tstzrange(last_rollup_time, curr_rollup_time, '(]')
  GROUP BY 1, 2;

  -- update the value in latest_rollup so that next time we run the
  -- rollup it will operate on data newer than curr_rollup_time
  UPDATE latest_rollup SET minute = curr_rollup_time;
END;
$$ LANGUAGE plpgsql;
```

Une fois votre fonction en place, exécutez-la pour cumuler les données :

```sql
SELECT rollup_http_request();
```

Avec vos données au format préagrégé, vous pouvez interroger la table de cumul pour obtenir le même rapport que précédemment. Exécutez la requête suivante :

```sql
SELECT site_id, ingest_time as minute, request_count,
       success_count, error_count, average_response_time_msec
  FROM http_request_1min
 WHERE ingest_time > date_trunc('minute', now()) - '5 minutes'::interval;
 ```

## <a name="expiring-old-data"></a>Expiration des données anciennes

Les cumuls accélèrent les requêtes, mais vous devez tout de même faire expirer les données anciennes pour éviter les coûts de stockage démesurés. Déterminez la durée pendant laquelle vous souhaitez conserver les données pour chaque granularité et utilisez des requêtes standard pour supprimer les données expirées. Dans l’exemple suivant, vous allez conserver les données brutes pendant un jour et les agrégations par minute pendant un mois :

```sql
DELETE FROM http_request WHERE ingest_time < now() - interval '1 day';
DELETE FROM http_request_1min WHERE ingest_time < now() - interval '1 month';
```

En production, vous pouvez encapsuler ces requêtes dans une fonction et appeler celle-ci toutes les minutes dans un travail cron.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Au cours des étapes précédentes, vous avez créé des ressources Azure dans un groupe de serveurs. Si vous pensez ne pas avoir besoin de ces ressources à l’avenir, supprimez le groupe de serveurs. Appuyez sur le bouton *Supprimer* dans la page *Vue d’ensemble* de votre groupe de serveurs. Quand vous y êtes invité dans une page contextuelle, confirmez le nom du groupe de serveurs, puis cliquez sur le bouton *Supprimer*.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris à configurer un groupe de serveurs Hyperscale (Citus). Vous vous y êtes connecté avec psql, vous avez créé un schéma et vous avez distribué les données. Vous avez vu comment interroger des données sous leur forme brute, à agréger régulièrement ces données, à interroger les tables agrégées et à faire expirer les données anciennes.

- En savoir plus sur les [types de nœuds](./concepts-hyperscale-nodes.md) des groupes de serveurs
- Déterminer la [taille initiale](howto-hyperscale-scale-initial.md) optimale pour votre groupe de serveurs
