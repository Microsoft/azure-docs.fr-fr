---
title: Gestion des coûts pour un pool SQL serverless
description: Ce document explique comment gérer le coût d’un pool SQL serverless et comment les données traitées sont calculées lorsqu’elles sont interrogées dans le Stockage Azure.
services: synapse analytics
author: filippopovic
ms.service: synapse-analytics
ms.topic: conceptual
ms.subservice: sql
ms.date: 11/05/2020
ms.author: fipopovi
ms.reviewer: jrasnick
ms.openlocfilehash: 8a26f8ced5e91810f8cadff0a27796dc817e6517
ms.sourcegitcommit: b4880683d23f5c91e9901eac22ea31f50a0f116f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94491567"
---
# <a name="cost-management-for-serverless-sql-pool-in-azure-synapse-analytics"></a>Gestion des coûts d’un pool SQL serverless dans Azure Synapse Analytics

Cet article explique comment estimer et gérer les coûts d’un pool SQL serverless dans Azure Synapse Analytics :
- Estimer la quantité de données traitées avant d’émettre une requête
- Utiliser la fonctionnalité de contrôle des coûts pour définir le budget

Il est important de comprendre que les coûts d’un pool SQL serverless dans Azure Synapse Analytics ne représentent qu’une partie des coûts mensuels qui apparaissent sur votre facture Azure. Si vous utilisez d’autres services Azure, vous êtes facturé pour tous les services et ressources Azure utilisés dans votre abonnement Azure, y compris les services tiers. Cet article explique comment planifier et gérer les coûts d’un pool SQL serverless dans Azure Synapse Analytics.

## <a name="data-processed"></a>Données traitées

Les *données traitées* correspondent à la quantité de données stockées temporairement par le système pendant l’exécution d’une requête. Elles se composent des quantités suivantes :

- Quantité de données lues à partir du stockage, soit :
  - Données lues pendant la lecture des données
  - Données lues pendant la lecture des métadonnées (pour les formats de fichiers qui contiennent des métadonnées, comme Parquet)
- Quantité de données présentes dans les résultats intermédiaires, transférées entre les nœuds pendant l’exécution de la requête, y compris le transfert de données vers votre point de terminaison, dans un format non compressé. 
- Quantité de données écrites dans le stockage. Si vous utilisez CETAS pour exporter votre jeu de résultats dans le stockage, la quantité de données écrites s’ajoute à la quantité de données traitées pour la partie SELECT de CETAS.

La lecture des fichiers à partir du stockage est hautement optimisée. Plusieurs techniques sont utilisées dans le processus :

- La prérécupération (prefetching), susceptible d’ajouter à la quantité de données lues. Si la requête lit un fichier entier, il n’y a aucune surcharge. Si le fichier est lu partiellement (par exemple, dans les requêtes TOP N), les quantités de données lues augmentent légèrement avec la prérécupération.
- Un analyseur CSV (valeurs séparées par des virgules) optimisé. Si vous utilisez PARSER_VERSION=’2.0’ pour lire des fichiers CSV, les quantités de données lues à partir du stockage augmentent légèrement. Un analyseur CSV optimisé lit les fichiers en parallèle, dans des blocs de taille égale. Ces blocs ne contiennent pas nécessairement des lignes entières. L’analyseur CSV optimisé lit également de petits fragments des blocs adjacents pour que toutes les lignes soient bien analysées. Ce processus ajoute un peu de surcharge.

## <a name="statistics"></a>Statistiques

L’optimiseur de requête du pool SQL serverless s’appuie sur des statistiques pour générer des plans optimaux d’exécution des requêtes. Il est possible de créer manuellement ces statistiques. Dans le cas contraire, le pool SQL serverless s’en charge automatiquement. Dans les deux cas, les statistiques sont créées en exécutant une requête distincte qui retourne une colonne spécifique suivant l’échantillonnage fourni. La quantité de données associée à cette requête est traitée.

Si vous exécutez la même requête ou une autre à laquelle seraient utiles les statistiques créées, ces dernières sont réutilisées dans la mesure du possible. Aucune donnée supplémentaire n’est traitée pour la création de statistiques.

Lorsque des statistiques sont créées pour une colonne Parquet, seule la colonne appropriée est lue dans les fichiers. Lorsque des statistiques sont créées pour une colonne CSV, les fichiers sont lus et analysés en totalité.

## <a name="rounding"></a>Arrondi

La quantité de données traitées est arrondie au volume de Mo le plus proche par requête. Chaque requête comporte au minimum 10 Mo de données traités.

## <a name="what-data-processed-doesnt-include"></a>Données non traitées

- Métadonnées au niveau du serveur (par exemple, connexions, rôles et informations d’identification au niveau du serveur)
- Bases de données créées dans votre point de terminaison, qui contiennent uniquement des métadonnées (par exemple, utilisateurs, rôles, schémas, vues, Fonctions table [TVF] inline, procédures stockées, informations d’identification de portée base de données, sources de données externes, formats de fichiers externes et tables externes)
  - Si vous utilisez l’inférence de schéma, les fragments de fichiers sont lus pour déduire les noms de colonnes et les types de données, et la quantité de données lues est ajoutée à la quantité de données traitées.
- Instructions du langage de définition de données (DDL), à l’exception de l’instruction CREATE STATISTICS, qui traite les données du stockage en fonction du pourcentage d’échantillonnage spécifié
- Requêtes portant uniquement sur les métadonnées

## <a name="reducing-the-amount-of-data-processed"></a>Réduction de la quantité de données traitées

Il est possible d’optimiser la quantité de données traitées par requête et d’améliorer le niveau de performance en partitionnant et convertissant les données dans un format compressé en colonnes comme Parquet.

## <a name="examples"></a>Exemples

Prenons trois tables.

- La table population_csv est sauvegardée par 5 To de fichiers CSV. Les fichiers sont organisés en cinq colonnes de taille égale.
- La table population_parquet comprend les mêmes données que la table population_csv. Elle est sauvegardée par 1 To de fichiers Parquet. Elle est plus petite que la précédente, car les données sont compressées au format Parquet.
- La table very_small_csv est sauvegardée par 100 Ko de fichiers CSV.

**Requête 1** : SELECT SUM(population) FROM population_csv

Cette requête lit et analyse les fichiers entiers pour obtenir les valeurs de la colonne population. Les nœuds traitent les fragments de cette table, et la somme de population pour chaque fragment est transférée entre les nœuds. La somme finale est transmise à votre point de terminaison. 

Cette requête traite 5 To de données, plus une petite quantité supplémentaire pour transférer les sommes des fragments.

**Requête 2** : SELECT SUM(population) FROM population_parquet

Lorsque vous interrogez des formats compressés en colonnes comme Parquet, la quantité de données lues est inférieure par rapport à la requête 1. En effet, le pool SQL serverless lit une seule colonne compressée, et non l’intégralité du fichier. Dans ce cas, 0,2 To de données sont lues. (Les cinq colonnes de taille identique font 0,2 To chacune.) Les nœuds traitent les fragments de cette table, et la somme de population pour chaque fragment est transférée entre les nœuds. La somme finale est transmise à votre point de terminaison. 

Cette requête traite 0,2 To, plus une petite quantité supplémentaire pour transférer les sommes des fragments.

**Requête 3** : SELECT * FROM population_parquet

Cette requête lit toutes les colonnes et transfère toutes les données dans un format non compressé. Si le format de compression est 5:1, la requête traite 6 To, car elle lit 1 To et transfère 5 To de données non compressées.

**Requête 4** : SELECT COUNT(*) FROM very_small_csv

Cette requête lit des fichiers entiers. La taille totale des fichiers dans le stockage de cette table est de 100 Ko. Les nœuds traitent les fragments de cette table, et la somme pour chaque fragment est transférée entre les nœuds. La somme finale est transmise à votre point de terminaison. 

Cette requête traite légèrement plus de 100 Ko de données. La quantité de données traitées pour cette requête est arrondie à 10 Mo (cf. section [Arrondi](#rounding) de cet article).

## <a name="cost-control"></a>Contrôle des coûts

La fonctionnalité de contrôle des coûts dans un pool SQL serverless permet de définir le budget adapté à la quantité de données traitées. Ce budget peut être défini en To de données traitées pour une journée, une semaine et un mois. Il est possible de disposer de plusieurs budgets simultanément. Pour configurer le contrôle des coûts d’un pool SQL serverless, vous pouvez utiliser Synapse Studio ou T-SQL.

## <a name="configure-cost-control-for-serverless-sql-pool-in-synapse-studio"></a>Configuration du contrôle des coûts d’un pool SQL serverless dans Synapse Studio
 
Pour configurer le contrôle des coûts d’un pool SQL serverless dans Synapse Studio, accédez à l’élément Gérer dans le menu de gauche, puis sélectionnez l’élément Pools SQL sous Pools d’analytique. Lorsque vous pointez sur le pool SQL serverless, une icône de contrôle des coûts apparaît. Cliquez dessus.

![Navigation dans le contrôle des coûts](./media/data-processed/cost-control-menu.png)

Lorsque vous cliquez sur l’icône de contrôle des coûts, un encadré s’affiche :

![Configuration du contrôle des coûts](./media/data-processed/cost-control-sidebar.png)

Pour définir un ou plusieurs budgets, commencez par cliquer sur la case d’option Activer de l’un deux, puis entrez la valeur entière dans la zone de texte. Cette valeur est exprimée en To. Une fois que vous avez configuré les budgets souhaités, cliquez sur le bouton Appliquer en bas de l’encadré. Votre budget est maintenant défini.

## <a name="configure-cost-control-for-serverless-sql-pool-in-t-sql"></a>Configuration du contrôle des coûts pour un pool SQL serverless dans T-SQL

Pour configurer le contrôle des coûts d’un pool SQL serverless dans T-SQL, vous devez exécuter une ou plusieurs des procédures stockées suivantes.

```sql
sp_set_data_processed_limit
    @type = N'daily',
    @limit_tb = 1

sp_set_data_processed_limit
    @type= N'weekly',
    @limit_tb = 2

sp_set_data_processed_limit
    @type= N'monthly',
    @limit_tb = 3334
```

Pour afficher la configuration actuelle, exécutez l’instruction T-SQL suivante :

```sql
SELECT * FROM sys.configurations
WHERE name like 'Data processed %';
```

Pour voir quelle quantité de données a été traitée au cours de la journée, de la semaine ou du mois en cours, exécutez l’instruction T-SQL suivante :

```sql
SELECT * FROM sys.dm_external_data_processed
```

## <a name="next-steps"></a>Étapes suivantes

Pour savoir comment optimiser le niveau de performance de vos requêtes, consultez [Meilleures pratiques pour le pool SQL serverless](best-practices-sql-on-demand.md).
