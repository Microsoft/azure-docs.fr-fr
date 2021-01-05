---
title: Utiliser les extensions PostgreSQL
description: Utiliser les extensions PostgreSQL
titleSuffix: Azure Arc enabled data services
services: azure-arc
ms.service: azure-arc
ms.subservice: azure-arc-data
author: TheJY
ms.author: jeanyd
ms.reviewer: mikeray
ms.date: 09/22/2020
ms.topic: how-to
ms.openlocfilehash: 3b9c3c66e58ae51773a959aba0b2c76d97b44445
ms.sourcegitcommit: ce8eecb3e966c08ae368fafb69eaeb00e76da57e
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/21/2020
ms.locfileid: "92309490"
---
# <a name="use-postgresql-extensions-in-your-azure-arc-enabled-postgresql-hyperscale-server-group"></a>Utiliser les extensions PostgreSQL dans votre groupe de serveurs Azure Arc enabled PostgreSQL Hyperscale

PostgreSQL est optimal quand vous l’utilisez avec des extensions. En fait, un élément clé de notre propre fonctionnalité Hyperscale est l’extension `citus` fournie par Microsoft qui est installée par défaut et permet à Postgres de partitionner les données sur plusieurs nœuds de façon transparente.


[!INCLUDE [azure-arc-data-preview](../../../includes/azure-arc-data-preview.md)]

## <a name="list-of-extensions"></a>Liste des extensions
En plus des extensions de [`contrib`](https://www.postgresql.org/docs/12/contrib.html), la liste des extensions présentes dans les conteneurs du groupe de serveurs Azure Arc enabled PostgreSQL Hyperscale est la suivante :
- `citus`, v : 9,4
- `pg_cron`, v : 1.2
- `plpgsql`, v : 1.0
- `postgis`, v : 3.0.2
- `plv8`, v : 2.3.14

Cette liste évolue au fil du temps et les mises à jour seront publiées dans ce document. Vous n’avez pas encore la possibilité d’ajouter des extensions en dehors de celles listées ci-dessus.

Ce guide se base sur un scénario d’utilisation de deux de ces extensions :
- [PostGIS](https://postgis.net/)
- [`pg_cron`](https://github.com/citusdata/pg_cron)


## <a name="manage-extensions"></a>Gérer les extensions

### <a name="enable-extensions"></a>Activer les extensions
Cette étape n’est pas nécessaire pour les extensions qui font partie de `contrib`.
Le format général de la commande pour activer les extensions est :

#### <a name="enable-an-extension-at-the-creation-time-of-a-server-group"></a>Activer une extension au moment de la création d’un groupe de serveurs :
```console
azdata arc postgres server create -n <name of your postgresql server group> --extensions <extension names>
```
#### <a name="enable-an-extension-on-an-instance-that-already-exists"></a>Activer une extension sur une instance qui existe déjà :
```console
azdata arc postgres server edit -n <name of your postgresql server group> --extensions <extension names>
```

#### <a name="get-the-list-of-extensions-enabled"></a>Obtenir la liste des extensions activées :
Exécutez une des commandes suivantes.

##### <a name="with-azure-data-cli-azdata"></a>Avec [!INCLUDE [azure-data-cli-azdata](../../../includes/azure-data-cli-azdata.md)]
```console
azdata arc postgres server show -n <server group name>
```
Faites défiler la sortie et notez les sections moteur\extensions dans les spécifications de votre groupe de serveurs. Exemple :
```console
"engine": {
      "extensions": [
        {
          "name": "citus"
        },
        {
          "name": "pg_cron"
        }
      ]
    },
```
##### <a name="with-kubectl"></a>Avec kubectl
```console
kubectl describe postgresql-12s/postgres02
```
Faites défiler la sortie et notez les sections moteur\extensions dans les spécifications de votre groupe de serveurs. Exemple :
```console
Engine:
    Extensions:
      Name:  citus
      Name:  pg_cron
```


### <a name="create-extensions"></a>Créer des extensions :
Connectez-vous à votre groupe de serveurs avec l’outil client de votre choix et exécutez la requête PostgreSQL standard :
```console
CREATE EXTENSION <extension name>;
```

### <a name="get-the-list-of-extension-created-in-your-server-group"></a>Obtenir la liste des extensions créées dans votre groupe de serveurs :
Connectez-vous à votre groupe de serveurs avec l’outil client de votre choix et exécutez la requête PostgreSQL standard :
```console
select * from pg_extension;
```

### <a name="drop-an-extension-from-your-server-group"></a>Supprimer une extension de votre groupe de serveurs :
Connectez-vous à votre groupe de serveurs avec l’outil client de votre choix et exécutez la requête PostgreSQL standard :
```console
drop extension <extension name>;
```

## <a name="use-the-postgis-and-the-pg_cron-extensions"></a>Utiliser les extensions PostGIS et Pg_cron

### <a name="the-postgis-extension"></a>L’extension PostGIS

Nous pouvons activer l’extension PostGIS sur un groupe de serveurs existant ou créer un groupe de serveurs avec l’extension déjà activée :

**Activation d’une extension au moment de la création d’un groupe de serveurs :**
```console
azdata arc postgres server create -n <name of your postgresql server group> --extensions <extension names>

#Example:
azdata arc postgres server create -n pg2 -w 2 --extensions postgis
```

**Activation d’une extension sur une instance déjà existante :**
```console
azdata arc postgres server edit -n <name of your postgresql server group> --extensions <extension names>

#Example:
azdata arc postgres server edit --extensions postgis -n pg2
```

Pour vérifier quelles extensions sont installées, utilisez la commande PostgreSQL standard ci-dessous après vous être connecté à l’instance avec votre outil client PostgreSQL préféré, comme Azure Data Studio :
```console
select * from pg_extension;
```

Pour obtenir un exemple PostGIS, commencez par obtenir l’[exemple de données](http://duspviz.mit.edu/tutorials/intro-postgis/) du Department of Urban Studies & Planning du MIT. Il peut être nécessaire d’exécuter `apt-get install unzip` pour installer unzip lors de l’utilisation de la machine virtuelle à des fins de test.

```console
wget http://duspviz.mit.edu/_assets/data/intro-postgis-datasets.zip
unzip intro-postgis-datasets.zip
```

Nous allons nous connecter à notre base de données, puis créer l’extension PostGIS :

```console
CREATE EXTENSION postgis;
```

> [!NOTE]
> Si vous voulez utiliser une des extensions du package `postgis` (par exemple `postgis_raster`, `postgis_topology`, `postgis_sfcgal`, `fuzzystrmatch`...), vous devez d’abord créer l’extension postgis, puis créer l’autre extension. Par exemple : `CREATE EXTENSION postgis`; `CREATE EXTENSION postgis_raster`;

Vous créez ensuite le schéma :

```sql
CREATE TABLE coffee_shops (
  id serial NOT NULL,
  name character varying(50),
  address character varying(50),
  city character varying(50),
  state character varying(50),
  zip character varying(10),
  lat numeric,
  lon numeric,
  geom geometry(POINT,4326)
);
CREATE INDEX coffee_shops_gist ON coffee_shops USING gist (geom);
```

Maintenant, nous pouvons combiner PostGIS avec la fonctionnalité de scale-out, en faisant de coffee_shops une table distribuée :

```sql
SELECT create_distributed_table('coffee_shops', 'id');
```

Nous allons charger des données :

```console
\copy coffee_shops(id,name,address,city,state,zip,lat,lon) from cambridge_coffee_shops.csv CSV HEADER;
```

Ensuite, nous renseignons le champ `geom` avec la latitude et la longitude encodées correctement dans le type de données `geometry` de PostGIS :

```sql
UPDATE coffee_shops SET geom = ST_SetSRID(ST_MakePoint(lon,lat),4326);
```

Nous pouvons maintenant lister les cafés les plus proches du MIT (77 Massachusetts Ave aux coordonnées 42,359055, -71,093500) :

```sql
SELECT name, address FROM coffee_shops ORDER BY geom <-> ST_SetSRID(ST_MakePoint(-71.093500,42.359055),4326);
```


### <a name="the-pg_cron-extension"></a>L’extension pg_cron

Activons `pg_cron` sur notre groupe de serveurs PostgreSQL, en plus de PostGIS :

```console
azdata postgres server update -n pg2 -ns arc --extensions postgis,pg_cron
```

Notez que cette opération redémarre les nœuds et installe les extensions supplémentaires, ce qui peut prendre 2 à 3 minutes.

Nous pouvons maintenant nous reconnecter et créer l’extension `pg_cron` :

```sql
CREATE EXTENSION pg_cron;
```

À des fins de test, créons une table `the_best_coffee_shop` qui prend un nom aléatoire à partir de notre table `coffee_shops` précédente et qui définit le contenu de la table :

```sql
CREATE TABLE the_best_coffee_shop(name text);
```

Nous pouvons utiliser `cron.schedule` plus quelques instructions SQL pour obtenir un nom de table aléatoire (remarquez l’utilisation d’une table temporaire pour stocker le résultat d’une requête distribuée) et nous le stockons dans `the_best_coffee_shop` :

```sql
SELECT cron.schedule('* * * * *', $$
  TRUNCATE the_best_coffee_shop;
  CREATE TEMPORARY TABLE tmp AS SELECT name FROM coffee_shops ORDER BY random() LIMIT 1;
  INSERT INTO the_best_coffee_shop SELECT * FROM tmp;
  DROP TABLE tmp;
$$);
```

Et maintenant, une fois par minute, nous obtenons un nom différent :

```sql
SELECT * FROM the_best_coffee_shop;
```

```console
      name
-----------------
 B & B Snack Bar
(1 row)
```

Pour plus de détails sur la syntaxe, consultez le fichier [README de pg_cron](https://github.com/citusdata/pg_cron).

>[!NOTE]
>La suppression de l’extension `citus` n’est pas prise en charge. L’extension `citus` est nécessaire pour fournir l’expérience Hyperscale.

## <a name="next-steps"></a>Étapes suivantes :
- Lire la documentation relative à [plv8](https://plv8.github.io/)
- Lire la documentation relative à [PostGIS](https://postgis.net/)
- Lire la documentation relative à [`pg_cron`](https://github.com/citusdata/pg_cron)
