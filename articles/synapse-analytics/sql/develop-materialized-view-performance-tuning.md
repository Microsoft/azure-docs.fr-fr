---
title: Réglage des performances avec des vues matérialisées
description: Recommandations et points à prendre en compte concernant les vues matérialisées pour améliorer les performances de vos requêtes.
services: synapse-analytics
author: XiaoyuMSFT
manager: craigg
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql
ms.date: 04/15/2020
ms.author: xiaoyul
ms.reviewer: nibruno; jrasnick
ms.openlocfilehash: d10b7084cfc49d60e9d14c3c857d1ade839398ac
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93305104"
---
# <a name="performance-tuning-with-materialized-views-using-dedicated-sql-pool-in-azure-synapse-analytics"></a>Réglage des performances avec des vues matérialisées à l’aide d’un pool SQL dédié dans Azure Synapse Analytics

Dans un pool SQL dédié, les vues matérialisées fournissent une méthode à faible maintenance pour les requêtes analytiques complexes en vue d’obtenir des performances rapides sans aucune modification des requêtes. Cet article dispense des conseils d’ordre général sur l’utilisation des vues matérialisées.

## <a name="materialized-views-vs-standard-views"></a>Vues matérialisées et vues standard

Le pool SQL prend en charge les vues standard et matérialisées.  Les deux sont des tables virtuelles créées avec des expressions SELECT et présentées aux requêtes sous forme de tables logiques.  Les vues révèlent la complexité du calcul de données courant et ajoutent une couche d’abstraction aux modifications de calcul si bien qu’il n’est pas nécessaire de réécrire les requêtes.  

Une vue standard calcule ses données chaque fois que la vue est utilisée.  Aucune donnée n’est stockée sur le disque. L’usage veut que les vues standard soient employés en tant qu’outil permettant d’organiser les objets logiques et les requêtes dans une base de données.  Pour utiliser une vue standard, une requête a besoin d’y faire référence directement.

Une vue matérialisée précalcule, stocke et conserve ses données dans le pool SQL dédié à l'instar d'une table.  Un recalcul n’est pas nécessaire à chaque utilisation d’une vue matérialisée.  C’est pourquoi des requêtes qui utilisent la totalité ou un sous-ensemble des données dans des vues matérialisées peuvent être plus rapides.  Mieux encore, les requêtes peuvent utiliser une vue matérialisée sans y faire référence directement. Il n’est donc pas nécessaire de modifier le code de l’application.  

La plupart des exigences applicables à une vue standard s’appliquent aussi à une vue matérialisée. Pour plus d’informations sur la syntaxe d’une vue matérialisée et d’autres exigences, consultez [CREATE MATERIALIZED VIEW AS SELECT](/sql/t-sql/statements/create-materialized-view-as-select-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).

| Comparaison                     | Affichage                                         | Vue matérialisée
|:-------------------------------|:---------------------------------------------|:--------------------------------------------------------------|
|Afficher la définition                 | Stockée dans Azure Data Warehouse.              | Stockée dans Azure Data Warehouse.
|Contenu de la vue                    | Générée chaque fois que la vue est utilisée.   | Prétraitée et stockée dans Azure Data Warehouse pendant sa création. Mise à jour à mesure que des données sont ajoutées aux tables sous-jacentes.
|Actualisation des données                    | Toujours mise à jour                               | Toujours mise à jour
|Vitesse pour récupérer des données de vue à partir de requêtes complexes     | Lente                                         | Rapide  
|Stockage supplémentaire                   | Non                                           | Oui
|Syntaxe                          | CREATE VIEW                                  | CREATE MATERIALIZED VIEW AS SELECT

## <a name="benefits-of-materialized-views"></a>Avantages des vues matérialisées

Une vue matérialisée correctement conçue offre les avantages suivants :

- Temps d’exécution réduit des requêtes complexes avec des jointures et des fonctions d’agrégation. Plus la requête est complexe, plus le potentiel d’enregistrement au moment de l’exécution est élevé. Le plus grand bénéfice est obtenu quand le coût de calcul d’une requête est élevé et que le jeu de données résultant est petit.  

- L'optimiseur de requête inclus dans le pool SQL dédié peut automatiquement utiliser les vues matérialisées déployées pour améliorer les plans d'exécution des requêtes.  Ce processus est transparent pour les utilisateurs, offrant des performances plus rapides aux requêtes. De plus, il ne nécessite pas que les requêtes fassent référence directement aux vues matérialisées.

- Peu de maintenance nécessaire sur les vues.  Une vue matérialisée stocke les données à deux emplacements : un index columnstore en cluster pour les données initiales au moment de la création de la vue et un deltastore pour les modifications de données incrémentielles.  Toutes les modifications apportées aux données à partir des tables de base sont automatiquement ajoutées au deltastore de manière synchrone.  Un processus en arrière-plan (moteur de tuple) déplace régulièrement les données du deltastore vers l’index columnstore de la vue.  Cette conception permet l’interrogation des vues matérialisées pour retourner les mêmes données qu’avec une interrogation directe des tables de base.
- Les données comprises dans une vue matérialisée peuvent être distribuées différemment dans les tables de base.  
- Dans les vues matérialisées, les données présentent les mêmes avantages en matière de haute disponibilité et de résilience que les données comprises dans des tables normales.  

En comparaison avec d’autres fournisseurs d’entrepôts de données, les vues matérialisées implémentées dans un pool SQL dédié offrent également les avantages supplémentaires suivants :

- Actualisation automatique et synchrone des données avec les modifications apportées aux données dans les tables de base. Aucune action de l'utilisateur n'est requise.
- Prise en charge étendue des fonctions d’agrégation. Consultez [CREATE MATERIALIZED VIEW AS SELECT (Transact-SQL)](/sql/t-sql/statements/create-materialized-view-as-select-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).
- Prise en charge de la recommandation de vue matérialisée propre à la requête.  Consultez [EXPLAIN (Transact-SQL)](/sql/t-sql/queries/explain-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true).

## <a name="common-scenarios"></a>Scénarios courants  

Les vues matérialisées sont généralement utilisées dans les scénarios suivants :

**Besoin d’améliorer les performances des requêtes analytiques complexes sur des données volumineuses en taille**

Les requêtes analytiques complexes utilisent généralement plus de fonctions d’agrégation et de jointures de table, ce qui entraîne plus d’opérations de calcul lourdes, comme des lectures aléatoires et des jointures dans l’exécution des requêtes.  C’est pourquoi ces requêtes sont plus longues à exécuter, en particulier sur des tables volumineuses.  

Les utilisateurs peuvent créer des vues matérialisées pour les données retournées par les calculs courants des requêtes. Il n’est donc pas nécessaire de recalculer ces données dont les requêtes ont besoin, ce qui permet de réduire le coût de calcul et d’accélérer la réponse aux requêtes.

**Besoin de performances plus rapides sans aucune modification des requêtes ou quelques modifications minimes**

Les modifications de schéma et de requête dans les entrepôts de données sont généralement maintenues à leur minimum pour prendre en charge les opérations ETL standard et la création de rapports.  Les utilisateurs peuvent utiliser des vues matérialisées pour le réglage des performances des requêtes, si le coût engendré par les vues peut être compensé par le gain de performance des requêtes.

En comparaison avec d’autres options de réglage, comme la mise à l’échelle et la gestion des statistiques, la création et la gestion d’une vue matérialisée est une modification de production à l’impact bien moindre et son gain de performance potentiel est également plus élevé.

- La création ou la gestion de vues matérialisées n’affecte pas les requêtes exécutées sur les tables de base.
- L’optimiseur de requête peut utiliser automatiquement les vues matérialisées déployées sans y faire référence directement dans une requête. Cette capacité réduit le besoin de modifier les requêtes dans le réglage des performances.

**Besoin d’une stratégie de distribution de données différente pour des performances de requête plus rapides**

Azure Data Warehouse est un système de traitement massivement parallèle distribué.  

SQL Synapse est un système de requêtes distribuées qui permet aux entreprises d’implémenter des scénarios d’entreposage et de virtualisation de données à l’aide d’expériences T-SQL standard bien connues des ingénieurs de données. Il étend également les capacités de SQL pour prendre en charge les scénarios de streaming et de Machine Learning. Les données d’une table d’entrepôt de données sont réparties sur 60 nœuds à l’aide de l’une des trois [stratégies de distribution](../sql-data-warehouse/sql-data-warehouse-tables-distribute.md?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json) (hachage, tourniquet ou réplication).  

La distribution des données est spécifiée au moment de la création de la table et reste inchangée jusqu’à ce que la table soit supprimée. Une vue matérialisée étant une table virtuelle sur un disque prend en charge les distributions de données en hachage et tourniquet.  Les utilisateurs peuvent choisir une distribution de données différente des tables de base, mais optimale pour les performances des requêtes qui utilisent fréquemment les vues.  

## <a name="design-guidance"></a>Guide de conception

Voici des conseils d’ordre général sur l’utilisation des vues matérialisées pour améliorer les performances des requêtes :

**Conception pour votre charge de travail**

Avant de commencer à créer des vues matérialisées, il est important de bien comprendre votre charge de travail en termes de modèles de requête, d’importance, de fréquence et de taille des données obtenues.  

Les utilisateurs peuvent exécuter EXPLAIN WITH_RECOMMENDATIONS <Instruction_SQL> pour les vues matérialisées recommandées par l’optimiseur de requête.  Étant donné que ces recommandations sont propres aux requêtes, une vue matérialisée qui tire parti d’une seule requête risque de ne pas être optimale pour d’autres requêtes dans la même charge de travail.  

Évaluez ces recommandations en tenant compte des besoins de votre charge de travail.  Les vues matérialisées idéales sont celles qui tirent parti des performances de la charge de travail.  

**Compromis entre des requêtes plus rapides et le coût**

Pour chaque vue matérialisée, il existe un coût de stockage des données et un coût de maintenance de la vue.  À mesure que les données sont modifiées dans les tables de base, la taille de la vue matérialisée augmente et sa structure physique change également.  

Pour éviter la dégradation des performances des requêtes, chaque vue matérialisée est tenue à jour séparément par le moteur de l’entrepôt de données, y compris le déplacement de lignes à partir du magasin Delta vers les segments d’index columnstore et la consolidation des modifications de données.  

La charge de travail de maintenance s’accroît quand le nombre de vues matérialisées et de modifications des tables de base augmente.   Les utilisateurs doivent vérifier si les coûts engendrés par toutes les vues matérialisées peuvent être compensés par le gain de performance des requêtes.  

Vous pouvez exécuter cette requête pour obtenir la liste des vues matérialisées dans une base de données :

```sql
SELECT V.name as materialized_view, V.object_id
FROM sys.views V
JOIN sys.indexes I ON V.object_id= I.object_id AND I.index_id < 2;
```

Options pour réduire le nombre de vues matérialisées :

- Identifiez les jeux de données courants fréquemment utilisés par les requêtes complexes dans votre charge de travail.  Créez des vues matérialisées pour stocker ces jeux de données afin que l’optimiseur puisse les utiliser en tant que composants lors de la création de plans d’exécution.  

- Supprimez les vues matérialisées peu utilisées ou devenues inutiles.  Une vue matérialisée désactivée ne fait pas l’objet d’une maintenance, mais elle induit toujours un coût de stockage.  

- Combinez des vues matérialisées créées sur des tables de base identiques ou similaires, même si leurs données ne se chevauchent pas.  La combinaison de vues matérialisées peut créer une taille plus grande que la somme des tailles de chaque vue, mais le coût de leur maintenance s’en retrouve réduit.  Par exemple :

```sql
-- Query 1 would benefit from having a materialized view created with this SELECT statement
SELECT A, SUM(B)
FROM T
GROUP BY A

-- Query 2 would benefit from having a materialized view created with this SELECT statement
SELECT C, SUM(D)
FROM T
GROUP BY C

-- You could create a single mateiralized view of this form
SELECT A, C, SUM(B), SUM(D)
FROM T
GROUP BY A, C

```

**Tous les réglages des performances n’ont pas besoin de modification des requêtes**

L’optimiseur de l’entrepôt de données peut utiliser automatiquement des vues matérialisées déployées pour améliorer les performances des requêtes.  Cette prise en charge s’applique en toute transparence aux requêtes qui ne font pas référence aux vues et aux requêtes qui utilisent des agrégats non pris en charge dans la création de vues matérialisées.  Aucune modification de requête n’est nécessaire. Vous pouvez vérifier le plan d’exécution estimé d’une requête pour confirmer si une vue matérialisée est utilisée.  

**Superviser les vues matérialisées**

Une vue matérialisée est stockée dans l’entrepôt de données de la même manière qu’une table avec un index columnstore cluster.  La lecture de données à partir d’une vue matérialisée comprend l’analyse de l’index et l’application des modifications à partir du deltastore.  Quand le nombre de lignes dans le deltastore est trop élevé, la résolution d’une requête à partir d’une vue matérialisée peut prendre plus de temps que l’interrogation directe des tables de base.  

Pour éviter cette dégradation des performances des requêtes, il est recommandé d’exécuter [DBCC PDW_SHOWMATERIALIZEDVIEWOVERHEAD](/sql/t-sql/database-console-commands/dbcc-pdw-showmaterializedviewoverhead-transact-sql?toc=/azure/synapse-analytics/toc.json&bc=/azure/synapse-analytics/breadcrumb/toc.json&view=azure-sqldw-latest&preserve-view=true) pour superviser la valeur overhead_ratio de la vue (total_rows/base_view_row).  Si la valeur overhead_ratio est trop élevée, envisagez de regénérer la vue matérialisée afin que toutes les lignes du deltastore soient déplacées vers l’index columnstore.  

**Vue matérialisée et mise en cache du jeu de résultats**

Ces deux fonctionnalités sont introduites dans le pool SQL dédié à peu près en même temps à des fins de réglage des performances des requêtes. La mise en cache du jeu de résultats est utilisée pour obtenir une concurrence élevée et des temps de réponse rapides aux requêtes répétitives sur des données statiques.  

Pour utiliser le résultat mis en cache, la forme de la requête de cache doit correspondre à la requête qui a produit le cache.  De plus, le résultat mis en cache doit s’appliquer à la requête entière.  

Les vues matérialisées permettent de modifier les données dans les tables de base.  Les données des vues matérialisées peuvent être appliquées à une partie d’une requête.  Cette prise en charge permet à des requêtes différentes, qui partagent des calculs à des fins d’accélération des performances, d’utiliser les mêmes vues matérialisées.

## <a name="example"></a>Exemple

Cet exemple utilise une requête de type TPCDS qui recherche les clients qui dépensent plus d’argent via catalogue que dans les magasins. Il identifie également les clients préférés et leur pays/région d’origine.   La requête consiste à sélectionner les 100 premiers enregistrements issus de l’instruction UNION de trois sous-instructions SELECT impliquant les fonctions SUM() et GROUP BY.

```sql
WITH year_total AS (
SELECT c_customer_id customer_id
       ,c_first_name customer_first_name
       ,c_last_name customer_last_name
       ,c_preferred_cust_flag customer_preferred_cust_flag
       ,c_birth_country customer_birth_country
       ,c_login customer_login
       ,c_email_address customer_email_address
       ,d_year dyear
       ,sum(isnull(ss_ext_list_price-ss_ext_wholesale_cost-ss_ext_discount_amt+ss_ext_sales_price, 0)/2) year_total
       ,'s' sale_type
FROM customer
     ,store_sales
     ,date_dim
WHERE c_customer_sk = ss_customer_sk
   AND ss_sold_date_sk = d_date_sk
GROUP BY c_customer_id
         ,c_first_name
         ,c_last_name
         ,c_preferred_cust_flag
         ,c_birth_country
         ,c_login
         ,c_email_address
         ,d_year
UNION ALL
SELECT c_customer_id customer_id
       ,c_first_name customer_first_name
       ,c_last_name customer_last_name
       ,c_preferred_cust_flag customer_preferred_cust_flag
       ,c_birth_country customer_birth_country
       ,c_login customer_login
       ,c_email_address customer_email_address
       ,d_year dyear
       ,sum(isnull(cs_ext_list_price-cs_ext_wholesale_cost-cs_ext_discount_amt+cs_ext_sales_price, 0)/2) year_total
       ,'c' sale_type
FROM customer
     ,catalog_sales
     ,date_dim
WHERE c_customer_sk = cs_bill_customer_sk
   AND cs_sold_date_sk = d_date_sk
GROUP BY c_customer_id
         ,c_first_name
         ,c_last_name
         ,c_preferred_cust_flag
         ,c_birth_country
         ,c_login
         ,c_email_address
         ,d_year
UNION ALL
SELECT c_customer_id customer_id
       ,c_first_name customer_first_name
       ,c_last_name customer_last_name
       ,c_preferred_cust_flag customer_preferred_cust_flag
       ,c_birth_country customer_birth_country
       ,c_login customer_login
       ,c_email_address customer_email_address
       ,d_year dyear
       ,sum(isnull(ws_ext_list_price-ws_ext_wholesale_cost-ws_ext_discount_amt+ws_ext_sales_price, 0)/2) year_total
       ,'w' sale_type
FROM customer
     ,web_sales
     ,date_dim
WHERE c_customer_sk = ws_bill_customer_sk
   AND ws_sold_date_sk = d_date_sk
GROUP BY c_customer_id
         ,c_first_name
         ,c_last_name
         ,c_preferred_cust_flag
         ,c_birth_country
         ,c_login
         ,c_email_address
         ,d_year
         )
  SELECT TOP 100
                  t_s_secyear.customer_id
                 ,t_s_secyear.customer_first_name
                 ,t_s_secyear.customer_last_name
                 ,t_s_secyear.customer_birth_country
FROM year_total t_s_firstyear
     ,year_total t_s_secyear
     ,year_total t_c_firstyear
     ,year_total t_c_secyear
     ,year_total t_w_firstyear
     ,year_total t_w_secyear
WHERE t_s_secyear.customer_id = t_s_firstyear.customer_id
   AND t_s_firstyear.customer_id = t_c_secyear.customer_id
   AND t_s_firstyear.customer_id = t_c_firstyear.customer_id
   AND t_s_firstyear.customer_id = t_w_firstyear.customer_id
   AND t_s_firstyear.customer_id = t_w_secyear.customer_id
   AND t_s_firstyear.sale_type = 's'
   AND t_c_firstyear.sale_type = 'c'
   AND t_w_firstyear.sale_type = 'w'
   AND t_s_secyear.sale_type = 's'
   AND t_c_secyear.sale_type = 'c'
   AND t_w_secyear.sale_type = 'w'
   AND t_s_firstyear.dyear+0 =  1999
   AND t_s_secyear.dyear+0 = 1999+1
   AND t_c_firstyear.dyear+0 =  1999
   AND t_c_secyear.dyear+0 =  1999+1
   AND t_w_firstyear.dyear+0 = 1999
   AND t_w_secyear.dyear+0 = 1999+1
   AND t_s_firstyear.year_total > 0
   AND t_c_firstyear.year_total > 0
   AND t_w_firstyear.year_total > 0
   AND CASE WHEN t_c_firstyear.year_total > 0 THEN t_c_secyear.year_total / t_c_firstyear.year_total ELSE NULL END
           > CASE WHEN t_s_firstyear.year_total > 0 THEN t_s_secyear.year_total / t_s_firstyear.year_total ELSE NULL END
   AND CASE WHEN t_c_firstyear.year_total > 0 THEN t_c_secyear.year_total / t_c_firstyear.year_total ELSE NULL END
           > CASE WHEN t_w_firstyear.year_total > 0 THEN t_w_secyear.year_total / t_w_firstyear.year_total ELSE NULL END
ORDER BY t_s_secyear.customer_id
         ,t_s_secyear.customer_first_name
         ,t_s_secyear.customer_last_name
         ,t_s_secyear.customer_birth_country
OPTION ( LABEL = 'Query04-af359846-253-3');
```

Vérifiez le plan d’exécution estimé de la requête.  Il y a 18 lectures aléatoires et 17 opérations de jointure, dont l’exécution prend plus de temps. Créons maintenant une vue matérialisée pour chacune des trois sous-instructions SELECT.

```sql
CREATE materialized view nbViewSS WITH (DISTRIBUTION=HASH(customer_id)) AS
SELECT c_customer_id customer_id
       ,c_first_name customer_first_name
       ,c_last_name customer_last_name
       ,c_preferred_cust_flag customer_preferred_cust_flag
       ,c_birth_country customer_birth_country
       ,c_login customer_login
       ,c_email_address customer_email_address
       ,d_year dyear
       ,sum(isnull(ss_ext_list_price-ss_ext_wholesale_cost-ss_ext_discount_amt+ss_ext_sales_price, 0)/2) year_total
          , count_big(*) AS cb
FROM dbo.customer
     ,dbo.store_sales
     ,dbo.date_dim
WHERE c_customer_sk = ss_customer_sk
   AND ss_sold_date_sk = d_date_sk
GROUP BY c_customer_id
         ,c_first_name
         ,c_last_name
         ,c_preferred_cust_flag
         ,c_birth_country
         ,c_login
         ,c_email_address
         ,d_year
GO
CREATE materialized view nbViewCS WITH (DISTRIBUTION=HASH(customer_id)) AS
SELECT c_customer_id customer_id
       ,c_first_name customer_first_name
       ,c_last_name customer_last_name
       ,c_preferred_cust_flag customer_preferred_cust_flag
       ,c_birth_country customer_birth_country
       ,c_login customer_login
       ,c_email_address customer_email_address
       ,d_year dyear
       ,sum(isnull(cs_ext_list_price-cs_ext_wholesale_cost-cs_ext_discount_amt+cs_ext_sales_price, 0)/2) year_total
          , count_big(*) as cb
FROM dbo.customer
     ,dbo.catalog_sales
     ,dbo.date_dim
WHERE c_customer_sk = cs_bill_customer_sk
   AND cs_sold_date_sk = d_date_sk
GROUP BY c_customer_id
         ,c_first_name
         ,c_last_name
         ,c_preferred_cust_flag
         ,c_birth_country
         ,c_login
         ,c_email_address
         ,d_year

GO
CREATE materialized view nbViewWS WITH (DISTRIBUTION=HASH(customer_id)) AS
SELECT c_customer_id customer_id
       ,c_first_name customer_first_name
       ,c_last_name customer_last_name
       ,c_preferred_cust_flag customer_preferred_cust_flag
       ,c_birth_country customer_birth_country
       ,c_login customer_login
       ,c_email_address customer_email_address
       ,d_year dyear
       ,sum(isnull(ws_ext_list_price-ws_ext_wholesale_cost-ws_ext_discount_amt+ws_ext_sales_price, 0)/2) year_total
          , count_big(*) AS cb
FROM dbo.customer
     ,dbo.web_sales
     ,dbo.date_dim
WHERE c_customer_sk = ws_bill_customer_sk
   AND ws_sold_date_sk = d_date_sk
GROUP BY c_customer_id
         ,c_first_name
         ,c_last_name
         ,c_preferred_cust_flag
         ,c_birth_country
         ,c_login
         ,c_email_address
         ,d_year

```

Revérifiez le plan d’exécution de la requête d’origine.  À présent, le nombre de jointures passe de 17 à 5 et il n’y a plus de lecture aléatoire.  Sélectionnez l’icône de l’opération Filtre dans le plan. Sa liste de résultat indique que les données sont lues à partir des vues matérialisées et non des tables de base.  

 ![Plan_Output_List_with_Materialized_Views](./media/develop-materialized-view-performance-tuning/output-list.png)

Avec les vues matérialisées, la même requête s’exécute beaucoup plus rapidement sans aucune modification de code.  

## <a name="next-steps"></a>Étapes suivantes

Pour obtenir des conseils supplémentaires en matière de développement, consultez [Présentation du développement SQL Synapse](develop-overview.md).
 