---
title: 'Tutoriel : Concevoir une base de données multilocataire - Hyperscale (Citus) - Azure Database pour PostgreSQL'
description: Ce tutoriel explique comment alimenter une application multilocataire évolutive avec Azure Database pour PostgreSQL Hyperscale (Citus).
author: jonels-msft
ms.author: jonels
ms.service: postgresql
ms.subservice: hyperscale-citus
ms.custom: mvc
ms.devlang: azurecli
ms.topic: tutorial
ms.date: 05/14/2019
ms.openlocfilehash: d8dee086f55c5596af50245b5271d98536eb7e57
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95026197"
---
# <a name="tutorial-design-a-multi-tenant-database-by-using-azure-database-for-postgresql--hyperscale-citus"></a>Tutoriel : Concevoir une base de données multilocataire avec Azure Database pour PostgreSQL – Hyperscale (Citus)

Dans ce tutoriel, vous utilisez Azure Database pour PostgreSQL - Hyperscale (Citus) pour apprendre à :

> [!div class="checklist"]
> * Créer un groupe de serveurs Hyperscale (Citus)
> * Utiliser l’utilitaire psql pour créer un schéma
> * Partitionner des tables entre des nœuds
> * Ingérer des exemples de données
> * Interroger les données de locataire
> * Partager des données entre les locataires
> * Personnaliser le schéma par locataire

## <a name="prerequisites"></a>Prérequis

[!INCLUDE [azure-postgresql-hyperscale-create-db](../../includes/azure-postgresql-hyperscale-create-db.md)]

## <a name="use-psql-utility-to-create-a-schema"></a>Utiliser l’utilitaire psql pour créer un schéma

Une fois que vous êtes connecté à Azure Database pour PostgreSQL - Hyperscale (Citus) à l’aide de psql, vous pouvez effectuer certaines tâches de base. Ce tutoriel vous explique comment créer une application web qui autorise les annonceurs à effectuer le suivi de leurs campagnes.

Plusieurs entreprises peuvent utiliser l’application. Nous allons donc créer une table pour stocker les entreprises et une autre pour leurs campagnes. Dans la console psql, exécutez les commandes suivantes :

```sql
CREATE TABLE companies (
  id bigserial PRIMARY KEY,
  name text NOT NULL,
  image_url text,
  created_at timestamp without time zone NOT NULL,
  updated_at timestamp without time zone NOT NULL
);

CREATE TABLE campaigns (
  id bigserial,
  company_id bigint REFERENCES companies (id),
  name text NOT NULL,
  cost_model text NOT NULL,
  state text NOT NULL,
  monthly_budget bigint,
  blacklisted_site_urls text[],
  created_at timestamp without time zone NOT NULL,
  updated_at timestamp without time zone NOT NULL,

  PRIMARY KEY (company_id, id)
);
```

Chaque campagne paiera pour afficher des publicités. Ajoutez également une table pour les publicités, en exécutant le code suivant dans psql après le code ci-dessus :

```sql
CREATE TABLE ads (
  id bigserial,
  company_id bigint,
  campaign_id bigint,
  name text NOT NULL,
  image_url text,
  target_url text,
  impressions_count bigint DEFAULT 0,
  clicks_count bigint DEFAULT 0,
  created_at timestamp without time zone NOT NULL,
  updated_at timestamp without time zone NOT NULL,

  PRIMARY KEY (company_id, id),
  FOREIGN KEY (company_id, campaign_id)
    REFERENCES campaigns (company_id, id)
);
```

Enfin, nous effectuerons le suivi des statistiques relatives aux clics et aux impressions pour chaque publicité :

```sql
CREATE TABLE clicks (
  id bigserial,
  company_id bigint,
  ad_id bigint,
  clicked_at timestamp without time zone NOT NULL,
  site_url text NOT NULL,
  cost_per_click_usd numeric(20,10),
  user_ip inet NOT NULL,
  user_data jsonb NOT NULL,

  PRIMARY KEY (company_id, id),
  FOREIGN KEY (company_id, ad_id)
    REFERENCES ads (company_id, id)
);

CREATE TABLE impressions (
  id bigserial,
  company_id bigint,
  ad_id bigint,
  seen_at timestamp without time zone NOT NULL,
  site_url text NOT NULL,
  cost_per_impression_usd numeric(20,10),
  user_ip inet NOT NULL,
  user_data jsonb NOT NULL,

  PRIMARY KEY (company_id, id),
  FOREIGN KEY (company_id, ad_id)
    REFERENCES ads (company_id, id)
);
```

Vous pouvez localiser les tables nouvellement créées dans la liste des tables dans psql en exécutant :

```postgres
\dt
```

Les applications multilocataires peuvent imposer l’unicité seulement par client. C’est pourquoi toutes les clés primaires et étrangères incluent l’ID de l’entreprise.

## <a name="shard-tables-across-nodes"></a>Partitionner des tables entre des nœuds

Un déploiement hyperscale stocke les lignes des tables sur des nœuds différents selon la valeur d’une colonne désignée par l’utilisateur. Cette « colonne de distribution » désigne quel locataire possède quelles lignes.

Nous allons définir la colonne de distribution en tant que company\_id, l’identificateur du locataire. Dans psql, exécutez les fonctions suivantes :

```sql
SELECT create_distributed_table('companies',   'id');
SELECT create_distributed_table('campaigns',   'company_id');
SELECT create_distributed_table('ads',         'company_id');
SELECT create_distributed_table('clicks',      'company_id');
SELECT create_distributed_table('impressions', 'company_id');
```

[!INCLUDE [azure-postgresql-hyperscale-dist-alert](../../includes/azure-postgresql-hyperscale-dist-alert.md)]

## <a name="ingest-sample-data"></a>Ingérer des exemples de données

En dehors de psql à présent, dans la ligne de commande normale, téléchargez des exemples de jeux de données :

```bash
for dataset in companies campaigns ads clicks impressions geo_ips; do
  curl -O https://examples.citusdata.com/mt_ref_arch/${dataset}.csv
done
```

De nouveau dans psql, effectuez le chargement en masse des données. Veillez à exécuter psql dans le répertoire où vous avez téléchargé les fichiers de données.

```sql
SET CLIENT_ENCODING TO 'utf8';

\copy companies from 'companies.csv' with csv
\copy campaigns from 'campaigns.csv' with csv
\copy ads from 'ads.csv' with csv
\copy clicks from 'clicks.csv' with csv
\copy impressions from 'impressions.csv' with csv
```

Ces données sont désormais réparties entre les nœuds worker.

## <a name="query-tenant-data"></a>Interroger les données de locataire

Lorsque l’application demande des données pour un locataire individuel, la base de données peut exécuter la requête sur un nœud worker unique. Les requêtes de locataire unique filtrent par un ID de locataire unique. Par exemple, la requête suivante filtre `company_id = 5` pour les publicités et les impressions. Essayez de l’exécuter dans psql pour voir les résultats.

```sql
SELECT a.campaign_id,
       RANK() OVER (
         PARTITION BY a.campaign_id
         ORDER BY a.campaign_id, count(*) desc
       ), count(*) as n_impressions, a.id
  FROM ads as a
  JOIN impressions as i
    ON i.company_id = a.company_id
   AND i.ad_id      = a.id
 WHERE a.company_id = 5
GROUP BY a.campaign_id, a.id
ORDER BY a.campaign_id, n_impressions desc;
```

## <a name="share-data-between-tenants"></a>Partager des données entre les locataires

Jusqu'à présent, toutes les tables ont été distribuées par `company_id`, mais certaines données « n’appartiennent » par nature à aucun locataire en particulier et peuvent être partagées. Par exemple, toutes les entreprises figurant dans l’exemple de plateforme de publicité peuvent souhaiter obtenir des informations géographiques sur leur public ciblé en fonction des adresses IP.

Créez une table pour stocker les informations géographiques partagées. Exécutez les commandes suivantes dans psql :

```sql
CREATE TABLE geo_ips (
  addrs cidr NOT NULL PRIMARY KEY,
  latlon point NOT NULL
    CHECK (-90  <= latlon[0] AND latlon[0] <= 90 AND
           -180 <= latlon[1] AND latlon[1] <= 180)
);
CREATE INDEX ON geo_ips USING gist (addrs inet_ops);
```

Définissez ensuite `geo_ips` comme « table de référence » pour stocker une copie de la table sur chaque nœud worker.

```sql
SELECT create_reference_table('geo_ips');
```

Chargez-la avec des exemples de données. Pensez à exécuter cette commande dans psql à partir du répertoire où vous avez téléchargé le jeu de données.

```sql
\copy geo_ips from 'geo_ips.csv' with csv
```

La jointure de la table de clics avec geo\_ips est efficace sur tous les nœuds.
Voici une jointure fournissant les emplacements de toutes les personnes qui ont cliqué sur la publicité
290. Essayez d’exécuter cette requête dans psql.

```sql
SELECT c.id, clicked_at, latlon
  FROM geo_ips, clicks c
 WHERE addrs >> c.user_ip
   AND c.company_id = 5
   AND c.ad_id = 290;
```

## <a name="customize-the-schema-per-tenant"></a>Personnaliser le schéma par locataire

Chaque locataire peut avoir besoin de stocker des informations spéciales dont les autres utilisateurs n’ont pas besoin. Toutefois, tous les locataires partagent une infrastructure commune avec un schéma de base de données identique. Où peuvent aller les données supplémentaires ?

Une astuce consiste à utiliser un type de colonne de durée indéterminée comme le type JSONB de PostgreSQL.  Notre schéma a un champ JSONB dans `clicks` appelé `user_data`.
Une entreprise (par exemple l’entreprise 5) peut utiliser cette colonne pour déterminer si l’utilisateur est sur un appareil mobile.

Voici une requête permettant de déterminer qui effectue le plus de clics : les visiteurs mobiles ou traditionnels.

```sql
SELECT
  user_data->>'is_mobile' AS is_mobile,
  count(*) AS count
FROM clicks
WHERE company_id = 5
GROUP BY user_data->>'is_mobile'
ORDER BY count DESC;
```

Nous pouvons optimiser cette requête pour une entreprise individuelle en créant un [index partiel](https://www.postgresql.org/docs/current/static/indexes-partial.html).

```sql
CREATE INDEX click_user_data_is_mobile
ON clicks ((user_data->>'is_mobile'))
WHERE company_id = 5;
```

Plus généralement, nous pouvons créer des [index GIN](https://www.postgresql.org/docs/current/static/gin-intro.html) sur chaque clé et chaque valeur de la colonne.

```sql
CREATE INDEX click_user_data
ON clicks USING gin (user_data);

-- this speeds up queries like, "which clicks have
-- the is_mobile key present in user_data?"

SELECT id
  FROM clicks
 WHERE user_data ? 'is_mobile'
   AND company_id = 5;
```

## <a name="clean-up-resources"></a>Nettoyer les ressources

Au cours des étapes précédentes, vous avez créé des ressources Azure dans un groupe de serveurs. Si vous pensez ne pas avoir besoin de ces ressources à l’avenir, supprimez le groupe de serveurs. Appuyez sur le bouton *Supprimer* dans la page *Vue d’ensemble* de votre groupe de serveurs. Quand vous y êtes invité dans une page contextuelle, confirmez le nom du groupe de serveurs, puis cliquez sur le bouton *Supprimer*.

## <a name="next-steps"></a>Étapes suivantes

Dans ce tutoriel, vous avez appris à configurer un groupe de serveurs Hyperscale (Citus). Vous vous y êtes connecté avec psql, vous avez créé un schéma et vous avez distribué les données. Vous avez appris à exécuter des requêtes de données au sein et entre les locataires, ainsi qu’à personnaliser le schéma par locataire.

- En savoir plus sur les [types de nœuds](./concepts-hyperscale-nodes.md) des groupes de serveurs
- Déterminer la [taille initiale](howto-hyperscale-scale-initial.md) optimale pour votre groupe de serveurs
