---
title: Extensions - Azure Database pour PostgreSQL - Serveur unique
description: Découvrez les extensions Postgres disponibles dans Azure Database pour PostgreSQL - Serveur unique
author: lfittl-msft
ms.author: lufittl
ms.service: postgresql
ms.topic: conceptual
ms.date: 09/14/2020
ms.openlocfilehash: 78395873457f9fe53d45dfbfd94aa9ccdccd614d
ms.sourcegitcommit: 3bcce2e26935f523226ea269f034e0d75aa6693a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/23/2020
ms.locfileid: "92485458"
---
# <a name="postgresql-extensions-in-azure-database-for-postgresql---single-server"></a>Extensions PostgreSQL dans Azure Database pour PostgreSQL - Serveur unique
PostgreSQL offre la possibilité d’étendre les fonctionnalités d’une base de données à l’aide des extensions. Les extensions regroupent plusieurs objets SQL associés au sein d’un package qui peut être chargé ou supprimé de votre base de données à l’aide d’une seule commande. Une fois chargées dans la base de données, les extensions fonctionnent comme des fonctionnalités intégrées.

## <a name="how-to-use-postgresql-extensions"></a>Guide pratique pour utiliser les extensions PostgreSQL
Les extensions PostgreSQL doivent être installées dans votre base de données pour être utilisables. Pour installer une extension donnée, exécutez la commande [CRÉER UNE EXTENSION](https://www.postgresql.org/docs/current/sql-createextension.html) dans l’outil psql pour charger les objets empaquetés dans votre base de données.

Azure Database pour PostgreSQL prend en charge un sous-ensemble d’extensions clés, comme indiqué ci-dessous. Ces informations sont également disponibles en exécutant `SELECT * FROM pg_available_extensions;`. Les extensions qui ne figurent pas dans la liste ne sont pas prises en charge. Vous ne pouvez pas créer votre propre extension dans Azure Database pour PostgreSQL.

## <a name="postgres-11-extensions"></a>Extensions Postgres 11

Les extensions suivantes sont disponibles dans les serveurs Azure Database pour PostgreSQL qui disposent de la version 11 de Postgres. 

> [!div class="mx-tableFixed"]
> | **Extension**| **Version d’extension** | **Description** |
> |---|---|---|
> |[address_standardizer](http://postgis.net/docs/Address_Standardizer.html)         | 2.5.1           | Utilisé pour analyser une adresse et la décomposer en éléments constitutifs, |
> |[address_standardizer_data_us](http://postgis.net/docs/Address_Standardizer.html) | 2.5.1           | Exemple de jeu de données de normalisation des adresses aux États-Unis|
> |[btree_gin](https://www.postgresql.org/docs/11/btree-gin.html)                    | 1.3             | Prise en charge de l’indexation des types de données communs dans GIN|
> |[btree_gist](https://www.postgresql.org/docs/11/btree-gist.html)                   | 1.5             | Prise en charge de l’indexation des types de données communs dans GiST|
> |[citext](https://www.postgresql.org/docs/11/citext.html)                       | 1.5             | Type de données des chaînes de caractères sans respect de la casse|
> |[cube](https://www.postgresql.org/docs/11/cube.html)                         | 1.4             | Type de données pour les cubes multidimensionnels|
> |[dblink](https://www.postgresql.org/docs/11/dblink.html)                       | 1.2             | Se connecte à d’autres bases de données PostgreSQL à partir d’une base de données|
> |[dict_int](https://www.postgresql.org/docs/11/dict-int.html)                     | 1.0             | Modèle de dictionnaire de recherche de texte pour les entiers|
> |[earthdistance](https://www.postgresql.org/docs/11/earthdistance.html)                | 1.1             | Calcule les distances orthodromiques à la surface de la Terre|
> |[fuzzystrmatch](https://www.postgresql.org/docs/11/fuzzystrmatch.html)                | 1.1             | Détermine les ressemblances et la distance entre les chaînes|
> |[hstore](https://www.postgresql.org/docs/11/hstore.html)                       | 1.5             | Type de données permettant de stocker des paires clé/valeur|
> |[hypopg](https://hypopg.readthedocs.io/en/latest/)                       | 1.1.2           | Index hypothétiques pour PostgreSQL|
> |[intarray](https://www.postgresql.org/docs/11/intarray.html)                     | 1.2             | Fonctions, opérateurs et prise en charge d’index pour les tableaux d’entiers 1D|
> |[isn](https://www.postgresql.org/docs/11/isn.html)                          | 1.2             | Types de données pour les standards internationaux de numérotation de produits|
> |[ltree](https://www.postgresql.org/docs/11/ltree.html)                        | 1.1             | Type de données pour les structures hiérarchiques de type arborescence|
> |[orafce](https://github.com/orafce/orafce)                       | 3.7             | Fonctions et opérateurs qui émulent un sous-ensemble de fonctions et de packages à partir de systèmes SGBDR commerciaux|
> |[pgaudit](https://www.pgaudit.org/)                     | 1.3.1             | fournit des fonctionnalités d’audit|
> |[pgcrypto](https://www.postgresql.org/docs/11/pgcrypto.html)                     | 1.3             | Fonctions de chiffrement|
> |[pgrouting](https://pgrouting.org/)                    | 2.6.2           | Extension pgRouting|
> |[pgrowlocks](https://www.postgresql.org/docs/11/pgrowlocks.html)                   | 1.2             | Affiche les informations de verrouillage au niveau des lignes|
> |[pgstattuple](https://www.postgresql.org/docs/11/pgstattuple.html)                  | 1.5             | Affiche les statistiques au niveau du tuple|
> |[pg_buffercache](https://www.postgresql.org/docs/11/pgbuffercache.html)               | 1.3             | Examine le cache des tampons partagé|
> |[pg_partman](https://github.com/pgpartman/pg_partman)                   | 4.0.0           | Extension pour gérer les tables partitionnées par date et heure ou par ID|
> |[pg_prewarm](https://www.postgresql.org/docs/11/pgprewarm.html)                   | 1.2             | Préchauffe les données de relation|
> |[pg_stat_statements](https://www.postgresql.org/docs/11/pgstatstatements.html)           | 1.6             | Suit les statistiques d’exécution de toutes les instructions SQL exécutées|
> |[pg_trgm](https://www.postgresql.org/docs/11/pgtrgm.html)                      | 1.4             | Mesure de la similarité du texte et recherche d’index sur la base de trigrammes|
> |[plpgsql](https://www.postgresql.org/docs/11/plpgsql.html)                      | 1.0             | Langage procédural PL/pgSQL|
> |[plv8](https://plv8.github.io/)                         | 2.3.11          | Langage procédural PL/JavaScript (v8) approuvé|
> |[postgis](https://www.postgis.net/)                      | 2.5.1           | Géométrie, géographie et types et fonctions spatiaux raster PostGIS|
> |[postgis_sfcgal](https://www.postgis.net/)               | 2.5.1           | Fonctions SFCGAL PostGIS|
> |[postgis_tiger_geocoder](https://www.postgis.net/)       | 2.5.1           | PostGIS tiger geocoder et geocoder inverse|
> |[postgis_topology](https://postgis.net/docs/Topology.html)             | 2.5.1           | Types et fonctions spatiaux de topologie PostGIS|
> |[postgres_fdw](https://www.postgresql.org/docs/11/postgres-fdw.html)                 | 1.0             | Wrapper de données externes pour les serveurs PostgreSQL distants|
> |[tablefunc](https://www.postgresql.org/docs/11/tablefunc.html)                    | 1.0             | Fonctions qui manipulent des tables entières, y compris les tables d’analyse croisée|
> |[timescaledb](https://docs.timescale.com/latest)                    | 1.3.2             | Permet des insertions scalables et des requêtes complexes pour les données de séries chronologiques|
> |[unaccent](https://www.postgresql.org/docs/11/unaccent.html)                     | 1.1             | Dictionnaire de recherche de texte qui supprime les accents|
> |[uuid-ossp](https://www.postgresql.org/docs/11/uuid-ossp.html)                    | 1.1             | Génère des identificateurs uniques universels (UUID)|

## <a name="postgres-10-extensions"></a>Extensions Postgres 10 

Les extensions suivantes sont disponibles dans les serveurs Azure Database pour PostgreSQL qui disposent de la version 10 de Postgres.

> [!div class="mx-tableFixed"]
> | **Extension**| **Version d’extension** | **Description** |
> |---|---|---|
> |[address_standardizer](http://postgis.net/docs/Address_Standardizer.html)         | 2.5.1           | Utilisé pour analyser une adresse et la décomposer en éléments constitutifs, |
> |[address_standardizer_data_us](http://postgis.net/docs/Address_Standardizer.html) | 2.5.1           | Exemple de jeu de données de normalisation des adresses aux États-Unis|
> |[btree_gin](https://www.postgresql.org/docs/10/btree-gin.html)                    | 1.3             | Prise en charge de l’indexation des types de données communs dans GIN|
> |[btree_gist](https://www.postgresql.org/docs/10/btree-gist.html)                   | 1.5             | Prise en charge de l’indexation des types de données communs dans GiST|
> |[chkpass](https://www.postgresql.org/docs/10/chkpass.html)                       | 1.0             | Type de données pour les mots de passe chiffrés automatiquement|
> |[citext](https://www.postgresql.org/docs/10/citext.html)                       | 1.4             | Type de données des chaînes de caractères sans respect de la casse|
> |[cube](https://www.postgresql.org/docs/10/cube.html)                         | 1.2             | Type de données pour les cubes multidimensionnels|
> |[dblink](https://www.postgresql.org/docs/10/dblink.html)                       | 1.2             | Se connecte à d’autres bases de données PostgreSQL à partir d’une base de données|
> |[dict_int](https://www.postgresql.org/docs/10/dict-int.html)                     | 1.0             | Modèle de dictionnaire de recherche de texte pour les entiers|
> |[earthdistance](https://www.postgresql.org/docs/10/earthdistance.html)                | 1.1             | Calcule les distances orthodromiques à la surface de la Terre|
> |[fuzzystrmatch](https://www.postgresql.org/docs/10/fuzzystrmatch.html)                | 1.1             | Détermine les ressemblances et la distance entre les chaînes|
> |[hstore](https://www.postgresql.org/docs/10/hstore.html)                       | 1.4             | Type de données permettant de stocker des paires clé/valeur|
> |[hypopg](https://hypopg.readthedocs.io/en/latest/)                       | 1.1.1           | Index hypothétiques pour PostgreSQL|
> |[intarray](https://www.postgresql.org/docs/10/intarray.html)                     | 1.2             | Fonctions, opérateurs et prise en charge d’index pour les tableaux d’entiers 1D|
> |[isn](https://www.postgresql.org/docs/10/isn.html)                          | 1.1             | Types de données pour les standards internationaux de numérotation de produits|
> |[ltree](https://www.postgresql.org/docs/10/ltree.html)                        | 1.1             | Type de données pour les structures hiérarchiques de type arborescence|
> |[orafce](https://github.com/orafce/orafce)                       | 3.7             | Fonctions et opérateurs qui émulent un sous-ensemble de fonctions et de packages à partir de systèmes SGBDR commerciaux|
> |[pgaudit](https://www.pgaudit.org/)                     | 1.2             | fournit des fonctionnalités d’audit|
> |[pgcrypto](https://www.postgresql.org/docs/10/pgcrypto.html)                     | 1.3             | Fonctions de chiffrement|
> |[pgrouting](https://pgrouting.org/)                    | 2.5.2           | Extension pgRouting|
> |[pgrowlocks](https://www.postgresql.org/docs/10/pgrowlocks.html)                   | 1.2             | Affiche les informations de verrouillage au niveau des lignes|
> |[pgstattuple](https://www.postgresql.org/docs/10/pgstattuple.html)                  | 1.5             | Affiche les statistiques au niveau du tuple|
> |[pg_buffercache](https://www.postgresql.org/docs/10/pgbuffercache.html)               | 1.3             | Examine le cache des tampons partagé|
> |[pg_partman](https://github.com/pgpartman/pg_partman)                   | 2.6.3           | Extension pour gérer les tables partitionnées par date et heure ou par ID|
> |[pg_prewarm](https://www.postgresql.org/docs/10/pgprewarm.html)                   | 1.1             | Préchauffe les données de relation|
> |[pg_stat_statements](https://www.postgresql.org/docs/10/pgstatstatements.html)           | 1.6             | Suit les statistiques d’exécution de toutes les instructions SQL exécutées|
> |[pg_trgm](https://www.postgresql.org/docs/10/pgtrgm.html)                      | 1.3             | Mesure de la similarité du texte et recherche d’index sur la base de trigrammes|
> |[plpgsql](https://www.postgresql.org/docs/10/plpgsql.html)                      | 1.0             | Langage procédural PL/pgSQL|
> |[plv8](https://plv8.github.io/)                         | 2.1.0          | Langage procédural PL/JavaScript (v8) approuvé|
> |[postgis](https://www.postgis.net/)                      | 2.4.3           | Géométrie, géographie et types et fonctions spatiaux raster PostGIS|
> |[postgis_sfcgal](https://www.postgis.net/)               | 2.4.3           | Fonctions SFCGAL PostGIS|
> |[postgis_tiger_geocoder](https://www.postgis.net/)       | 2.4.3           | PostGIS tiger geocoder et geocoder inverse|
> |[postgis_topology](https://postgis.net/docs/Topology.html)             | 2.4.3           | Types et fonctions spatiaux de topologie PostGIS|
> |[postgres_fdw](https://www.postgresql.org/docs/10/postgres-fdw.html)                 | 1.0             | Wrapper de données externes pour les serveurs PostgreSQL distants|
> |[tablefunc](https://www.postgresql.org/docs/10/tablefunc.html)                    | 1.0             | Fonctions qui manipulent des tables entières, y compris les tables d’analyse croisée|
> |[timescaledb](https://docs.timescale.com/latest)                    | 1.1.1             | Permet des insertions scalables et des requêtes complexes pour les données de séries chronologiques|
> |[unaccent](https://www.postgresql.org/docs/10/unaccent.html)                     | 1.1             | Dictionnaire de recherche de texte qui supprime les accents|
> |[uuid-ossp](https://www.postgresql.org/docs/10/uuid-ossp.html)                    | 1.1             | Génère des identificateurs uniques universels (UUID)|

## <a name="postgres-96-extensions"></a>Extensions Postgres 9.6 

Les extensions suivantes sont disponibles dans les serveurs Azure Database pour PostgreSQL qui disposent de la version 9.6 de Postgres.

> [!div class="mx-tableFixed"]
> | **Extension**| **Version d’extension** | **Description** |
> |---|---|---|
> |[address_standardizer](http://postgis.net/docs/Address_Standardizer.html)         | 2.3.2           | Utilisé pour analyser une adresse et la décomposer en éléments constitutifs, |
> |[address_standardizer_data_us](http://postgis.net/docs/Address_Standardizer.html) | 2.3.2           | Exemple de jeu de données de normalisation des adresses aux États-Unis|
> |[btree_gin](https://www.postgresql.org/docs/9.6/btree-gin.html)                    | 1.0             | Prise en charge de l’indexation des types de données communs dans GIN|
> |[btree_gist](https://www.postgresql.org/docs/9.6/btree-gist.html)                   | 1.2             | Prise en charge de l’indexation des types de données communs dans GiST|
> |[chkpass](https://www.postgresql.org/docs/9.6/chkpass.html)                       | 1.0             | Type de données pour les mots de passe chiffrés automatiquement|
> |[citext](https://www.postgresql.org/docs/9.6/citext.html)                       | 1.3             | Type de données des chaînes de caractères sans respect de la casse|
> |[cube](https://www.postgresql.org/docs/9.6/cube.html)                         | 1.2             | Type de données pour les cubes multidimensionnels|
> |[dblink](https://www.postgresql.org/docs/9.6/dblink.html)                       | 1.2             | Se connecte à d’autres bases de données PostgreSQL à partir d’une base de données|
> |[dict_int](https://www.postgresql.org/docs/9.6/dict-int.html)                     | 1.0             | Modèle de dictionnaire de recherche de texte pour les entiers|
> |[earthdistance](https://www.postgresql.org/docs/9.6/earthdistance.html)                | 1.1             | Calcule les distances orthodromiques à la surface de la Terre|
> |[fuzzystrmatch](https://www.postgresql.org/docs/9.6/fuzzystrmatch.html)                | 1.1             | Détermine les ressemblances et la distance entre les chaînes|
> |[hstore](https://www.postgresql.org/docs/9.6/hstore.html)                       | 1.4             | Type de données permettant de stocker des paires clé/valeur|
> |[hypopg](https://hypopg.readthedocs.io/en/latest/)                       | 1.1.1           | Index hypothétiques pour PostgreSQL|
> |[intarray](https://www.postgresql.org/docs/9.6/intarray.html)                     | 1.2             | Fonctions, opérateurs et prise en charge d’index pour les tableaux d’entiers 1D|
> |[isn](https://www.postgresql.org/docs/9.6/isn.html)                          | 1.1             | Types de données pour les standards internationaux de numérotation de produits|
> |[ltree](https://www.postgresql.org/docs/9.6/ltree.html)                        | 1.1             | Type de données pour les structures hiérarchiques de type arborescence|
> |[orafce](https://github.com/orafce/orafce)                       | 3.7             | Fonctions et opérateurs qui émulent un sous-ensemble de fonctions et de packages à partir de systèmes SGBDR commerciaux|
> |[pgaudit](https://www.pgaudit.org/)                     | 1.1.2             | fournit des fonctionnalités d’audit|
> |[pgcrypto](https://www.postgresql.org/docs/9.6/pgcrypto.html)                     | 1.3             | Fonctions de chiffrement|
> |[pgrouting](https://pgrouting.org/)                    | 2.3.2           | Extension pgRouting|
> |[pgrowlocks](https://www.postgresql.org/docs/9.6/pgrowlocks.html)                   | 1.2             | Affiche les informations de verrouillage au niveau des lignes|
> |[pgstattuple](https://www.postgresql.org/docs/9.6/pgstattuple.html)                  | 1.4             | Affiche les statistiques au niveau du tuple|
> |[pg_buffercache](https://www.postgresql.org/docs/9.6/pgbuffercache.html)               | 1.2             | Examine le cache des tampons partagé|
> |[pg_partman](https://github.com/pgpartman/pg_partman)                   | 2.6.3           | Extension pour gérer les tables partitionnées par date et heure ou par ID|
> |[pg_prewarm](https://www.postgresql.org/docs/9.6/pgprewarm.html)                   | 1.1             | Préchauffe les données de relation|
> |[pg_stat_statements](https://www.postgresql.org/docs/9.6/pgstatstatements.html)           | 1.4             | Suit les statistiques d’exécution de toutes les instructions SQL exécutées|
> |[pg_trgm](https://www.postgresql.org/docs/9.6/pgtrgm.html)                      | 1.3             | Mesure de la similarité du texte et recherche d’index sur la base de trigrammes|
> |[plpgsql](https://www.postgresql.org/docs/9.6/plpgsql.html)                      | 1.0             | Langage procédural PL/pgSQL|
> |[plv8](https://plv8.github.io/)                         | 2.1.0          | Langage procédural PL/JavaScript (v8) approuvé|
> |[postgis](https://www.postgis.net/)                      | 2.3.2           | Géométrie, géographie et types et fonctions spatiaux raster PostGIS|
> |[postgis_sfcgal](https://www.postgis.net/)               | 2.3.2           | Fonctions SFCGAL PostGIS|
> |[postgis_tiger_geocoder](https://www.postgis.net/)       | 2.3.2           | PostGIS tiger geocoder et geocoder inverse|
> |[postgis_topology](https://postgis.net/docs/Topology.html)             | 2.3.2           | Types et fonctions spatiaux de topologie PostGIS|
> |[postgres_fdw](https://www.postgresql.org/docs/9.6/postgres-fdw.html)                 | 1.0             | Wrapper de données externes pour les serveurs PostgreSQL distants|
> |[tablefunc](https://www.postgresql.org/docs/9.6/tablefunc.html)                    | 1.0             | Fonctions qui manipulent des tables entières, y compris les tables d’analyse croisée|
> |[timescaledb](https://docs.timescale.com/latest)                    | 1.1.1             | Permet des insertions scalables et des requêtes complexes pour les données de séries chronologiques|
> |[unaccent](https://www.postgresql.org/docs/9.6/unaccent.html)                     | 1.1             | Dictionnaire de recherche de texte qui supprime les accents|
> |[uuid-ossp](https://www.postgresql.org/docs/9.6/uuid-ossp.html)                    | 1.1             | Génère des identificateurs uniques universels (UUID)|

## <a name="postgres-95-extensions"></a>Extensions Postgres 9.5 

Les extensions suivantes sont disponibles dans les serveurs Azure Database pour PostgreSQL qui disposent de la version 9.5 de Postgres.

> [!div class="mx-tableFixed"]
> | **Extension**| **Version d’extension** | **Description** |
> |---|---|---|
> |[address_standardizer](http://postgis.net/docs/Address_Standardizer.html)         | 2.3.0           | Utilisé pour analyser une adresse et la décomposer en éléments constitutifs, |
> |[address_standardizer_data_us](http://postgis.net/docs/Address_Standardizer.html) | 2.3.0           | Exemple de jeu de données de normalisation des adresses aux États-Unis|
> |[btree_gin](https://www.postgresql.org/docs/9.5/btree-gin.html)                    | 1.0             | Prise en charge de l’indexation des types de données communs dans GIN|
> |[btree_gist](https://www.postgresql.org/docs/9.5/btree-gist.html)                   | 1.1             | Prise en charge de l’indexation des types de données communs dans GiST|
> |[chkpass](https://www.postgresql.org/docs/9.5/chkpass.html)                       | 1.0             | Type de données pour les mots de passe chiffrés automatiquement|
> |[citext](https://www.postgresql.org/docs/9.5/citext.html)                       | 1.1             | Type de données des chaînes de caractères sans respect de la casse|
> |[cube](https://www.postgresql.org/docs/9.5/cube.html)                         | 1.0             | Type de données pour les cubes multidimensionnels|
> |[dblink](https://www.postgresql.org/docs/9.5/dblink.html)                       | 1.1             | Se connecte à d’autres bases de données PostgreSQL à partir d’une base de données|
> |[dict_int](https://www.postgresql.org/docs/9.5/dict-int.html)                     | 1.0             | Modèle de dictionnaire de recherche de texte pour les entiers|
> |[earthdistance](https://www.postgresql.org/docs/9.5/earthdistance.html)                | 1.0             | Calcule les distances orthodromiques à la surface de la Terre|
> |[fuzzystrmatch](https://www.postgresql.org/docs/9.5/fuzzystrmatch.html)                | 1.0             | Détermine les ressemblances et la distance entre les chaînes|
> |[hstore](https://www.postgresql.org/docs/9.5/hstore.html)                       | 1.3             | Type de données permettant de stocker des paires clé/valeur|
> |[hypopg](https://hypopg.readthedocs.io/en/latest/)                       | 1.1.1           | Index hypothétiques pour PostgreSQL|
> |[intarray](https://www.postgresql.org/docs/9.5/intarray.html)                     | 1.0             | Fonctions, opérateurs et prise en charge d’index pour les tableaux d’entiers 1D|
> |[isn](https://www.postgresql.org/docs/9.5/isn.html)                          | 1.0             | Types de données pour les standards internationaux de numérotation de produits|
> |[ltree](https://www.postgresql.org/docs/9.5/ltree.html)                        | 1.0             | Type de données pour les structures hiérarchiques de type arborescence|
> |[orafce](https://github.com/orafce/orafce)                       | 3.7             | Fonctions et opérateurs qui émulent un sous-ensemble de fonctions et de packages à partir de systèmes SGBDR commerciaux|
> |[pgaudit](https://www.pgaudit.org/)                     | 1.0.7             | fournit des fonctionnalités d’audit|
> |[pgcrypto](https://www.postgresql.org/docs/9.5/pgcrypto.html)                     | 1.2             | Fonctions de chiffrement|
> |[pgrouting](https://pgrouting.org/)                    | 2.3.0           | Extension pgRouting|
> |[pgrowlocks](https://www.postgresql.org/docs/9.5/pgrowlocks.html)                   | 1.1             | Affiche les informations de verrouillage au niveau des lignes|
> |[pgstattuple](https://www.postgresql.org/docs/9.5/pgstattuple.html)                  | 1.3             | Affiche les statistiques au niveau du tuple|
> |[pg_buffercache](https://www.postgresql.org/docs/9.5/pgbuffercache.html)               | 1.1             | Examine le cache des tampons partagé|
> |[pg_partman](https://github.com/pgpartman/pg_partman)                   | 2.6.3           | Extension pour gérer les tables partitionnées par date et heure ou par ID|
> |[pg_prewarm](https://www.postgresql.org/docs/9.5/pgprewarm.html)                   | 1.0             | Préchauffe les données de relation|
> |[pg_stat_statements](https://www.postgresql.org/docs/9.5/pgstatstatements.html)           | 1.3             | Suit les statistiques d’exécution de toutes les instructions SQL exécutées|
> |[pg_trgm](https://www.postgresql.org/docs/9.5/pgtrgm.html)                      | 1.1             | Mesure de la similarité du texte et recherche d’index sur la base de trigrammes|
> |[plpgsql](https://www.postgresql.org/docs/9.5/plpgsql.html)                      | 1.0             | Langage procédural PL/pgSQL|
> |[postgis](https://www.postgis.net/)                      | 2.3.0           | Géométrie, géographie et types et fonctions spatiaux raster PostGIS|
> |[postgis_sfcgal](https://www.postgis.net/)               | 2.3.0           | Fonctions SFCGAL PostGIS|
> |[postgis_tiger_geocoder](https://www.postgis.net/)       | 2.3.0           | PostGIS tiger geocoder et geocoder inverse|
> |[postgis_topology](https://postgis.net/docs/Topology.html)             | 2.3.0           | Types et fonctions spatiaux de topologie PostGIS|
> |[postgres_fdw](https://www.postgresql.org/docs/9.5/postgres-fdw.html)                 | 1.0             | Wrapper de données externes pour les serveurs PostgreSQL distants|
> |[tablefunc](https://www.postgresql.org/docs/9.5/tablefunc.html)                    | 1.0             | Fonctions qui manipulent des tables entières, y compris les tables d’analyse croisée|
> |[unaccent](https://www.postgresql.org/docs/9.5/unaccent.html)                     | 1.0             | Dictionnaire de recherche de texte qui supprime les accents|
> |[uuid-ossp](https://www.postgresql.org/docs/9.5/uuid-ossp.html)                    | 1.0             | Génère des identificateurs uniques universels (UUID)|


## <a name="pg_stat_statements"></a>pg_stat_statements
L’[extension pg_stat_statements](https://www.postgresql.org/docs/current/pgstatstatements.html) est préchargée sur chaque serveur Azure Database pour PostgreSQL afin de vous fournir un moyen de suivre les statistiques d’exécution des instructions SQL.
Le paramètre `pg_stat_statements.track`, qui contrôle quelles instructions sont comptées par l’extension, a la valeur par défaut `top`, ce qui signifie que toutes les instructions exécutées directement par les clients sont suivies. Les deux autres niveaux de suivi sont `none` et `all`. Ce paramètre peut être configuré comme paramètre de serveur via le [portail Azure](./howto-configure-server-parameters-using-portal.md) ou [Azure CLI](./howto-configure-server-parameters-using-cli.md).

Un compromis existe entre les informations d’exécution des requêtes fournies par pg_stat_statements et l’impact sur les performances du serveur en raison de la journalisation de chaque instruction SQL. Si vous n’utilisez pas l’extension pg_stat_statements de façon active, nous vous recommandons de définir `pg_stat_statements.track` sur `none`. Notez que certains services de surveillance tiers peuvent s’appuyer sur pg_stat_statements pour fournir des insights sur les performances des requêtes : vérifiez donc si c’est ou non le cas pour vous.

## <a name="dblink-and-postgres_fdw"></a>dblink et postgres_fdw
[dblink](https://www.postgresql.org/docs/current/contrib-dblink-function.html) et [postgres_fdw](https://www.postgresql.org/docs/current/postgres-fdw.html) vous permettent de vous connecter d’un serveur PostgreSQL à un autre ou à une autre base de données dans le même serveur. Le serveur de réception doit autoriser les connexions à partir du serveur d’envoi via son pare-feu. Quand vous utilisez ces extensions pour établir une connexion entre des serveurs Azure Database pour PostgreSQL, vous pouvez effectuer cette opération en définissant « Autoriser l’accès aux services Azure » sur Activé. Vous devez faire de même si vous souhaitez utiliser les extensions pour établir la connexion vers le même serveur. Le paramètre « Autoriser l’accès aux services Azure » se trouve dans la page du portail Azure dédiée au serveur Postgres, sous Sécurité de la connexion. L’activation du paramètre « Autoriser l’accès aux services Azure » place toutes les adresses IP Azure sur liste verte.

Actuellement, les connexions sortantes d’Azure Database pour PostgreSQL ne sont pas prises en charge, à l’exception des connexions vers d’autres serveurs Azure Database pour PostgreSQL au sein de la même région.

## <a name="uuid"></a>uuid
Si vous pensez utiliser `uuid_generate_v4()` à partir de l’[extension uuid-ossp](https://www.postgresql.org/docs/current/uuid-ossp.html), envisagez la comparaison avec `gen_random_uuid()` à partir de l’[extension pgcrypto](https://www.postgresql.org/docs/current/pgcrypto.html) pour optimiser les performances.

## <a name="pgaudit"></a>pgAudit
L’[extension pgAudit](https://github.com/pgaudit/pgaudit/blob/master/README.md) permet la journalisation d’audit des sessions et des objets. Pour savoir comment utiliser cette extension dans Azure Database pour PostgreSQL, consultez l’[article sur les concepts d’audit](concepts-audit.md). 

## <a name="pg_prewarm"></a>pg_prewarm
L’extension pg_prewarm charge des données relationnelles dans le cache. Le préchauffage de vos caches signifie que vos requêtes ont de meilleurs temps de réponse lors de leur première exécution après un redémarrage. Dans Postgres 10 et versions antérieures, le préchauffage est effectué manuellement à l’aide de la [fonction de préchauffage](https://www.postgresql.org/docs/10/pgprewarm.html).

Dans Postgres 11 et versions ultérieures, vous pouvez configurer le préchauffage pour qu’il se produise [automatiquement](https://www.postgresql.org/docs/current/pgprewarm.html). Vous devez inclure pg_prewarm dans la liste de votre paramètre `shared_preload_libraries` et redémarrer le serveur pour appliquer la modification. Les paramètres peuvent être définis à partir du [portail Azure](howto-configure-server-parameters-using-portal.md), de [CLI](howto-configure-server-parameters-using-cli.md), de l’API REST ou du modèle ARM. 

## <a name="timescaledb"></a>TimescaleDB
TimescaleDB est une base de données de séries chronologiques empaquetée en tant qu’extension pour PostgreSQL. TimescaleDB fournit des fonctions analytiques axées sur le temps et des optimisations et met à l’échelle PostgreSQL pour les charges de travail de série chronologique.

[Apprenez-en davantage sur TimescaleDB](https://docs.timescale.com/latest), marque déposée de [Timescale, Inc.](https://www.timescale.com/). Azure Database pour PostgreSQL fournit la TimescaleDB [Apache-2](https://www.timescale.com/legal/licenses).

### <a name="installing-timescaledb"></a>Installation de TimescaleDB
Pour installer TimescaleDB, vous devez l’inclure dans les bibliothèques de préchargement partagées du serveur. Une modification apportée au paramètre `shared_preload_libraries` de Postgres nécessite un **redémarrage du serveur** pour prendre effet. Vous pouvez modifier les paramètres à l’aide du [portail Azure](howto-configure-server-parameters-using-portal.md) ou d’[Azure CLI](howto-configure-server-parameters-using-cli.md).

À l’aide du [Portail Azure](https://portal.azure.com/) :

1. Sélectionnez votre serveur Azure Database pour PostgreSQL.

2. Dans la barre latérale, sélectionnez **Paramètres du serveur** .

3. Recherchez le paramètre `shared_preload_libraries`.

4. Sélectionnez **TimescaleDB** .

5. Sélectionnez **Enregistrer** pour conserver vos modifications. Vous recevez une notification une fois que la modification est enregistrée. 

6. Après avoir reçu la notification, **redémarrez** le serveur pour appliquer ces modifications. Pour apprendre à redémarrer un serveur, consultez [Redémarrer un serveur Azure Database pour PostgreSQL](howto-restart-server-portal.md).


Vous pouvez maintenant activer TimescaleDB dans votre base de données Postgres. Connectez-vous à la base de données et exécutez la commande suivante :
```sql
CREATE EXTENSION IF NOT EXISTS timescaledb CASCADE;
```
> [!TIP]
> Si vous voyez une erreur, vérifiez que vous avez [redémarré le serveur](howto-restart-server-portal.md) après l’enregistrement de shared_preload_libraries. 

Vous pouvez maintenant créer une hypertable TimescaleDB [à partir de zéro](https://docs.timescale.com/getting-started/creating-hypertables) ou migrer des [données de série chronologiques existantes dans PostgreSQL](https://docs.timescale.com/getting-started/migrating-data).

### <a name="restoring-a-timescale-database"></a>Restauration d’une base de données d’échelle de temps
Pour restaurer une base de données d’échelle de temps à l’aide de pg_dump et pg_restore, vous devez exécuter deux procédures d’assistance dans la base de données de destination : `timescaledb_pre_restore()` et `timescaledb_post restore()`.

Préparez d’abord la base de données de destination :

```SQL
--create the new database where you'll perform the restore
CREATE DATABASE tutorial;
\c tutorial --connect to the database 
CREATE EXTENSION timescaledb;

SELECT timescaledb_pre_restore();
```

Vous pouvez maintenant exécuter pg_dump sur la base de données d’origine, puis effectuer pg_restore. Après la restauration, veillez à exécuter la commande suivante dans la base de données restaurée :

```SQL
SELECT timescaledb_post_restore();
```


## <a name="next-steps"></a>Étapes suivantes
Si vous ne voyez pas une extension que vous souhaitez utiliser, faites-le-nous savoir. Votez pour les demandes existantes ou créez de nouvelles demandes dans notre [Forum de commentaires](https://feedback.azure.com/forums/597976-azure-database-for-postgresql).