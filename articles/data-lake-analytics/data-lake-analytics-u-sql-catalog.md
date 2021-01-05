---
title: Utiliser le catalogue U-SQL dans Azure Data Lake Analytics
description: Découvrez comment utiliser le catalogue U-SQL pour partager du code et des données. Créez des fonctions table, créez des vues, créez des tables, et interrogez-les.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 05/09/2017
ms.openlocfilehash: f92aadc8ccf18dd91b5dd4b35285f60b174e4cf7
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92220072"
---
# <a name="get-started-with-the-u-sql-catalog-in-azure-data-lake-analytics"></a>Bien démarrer avec le catalogue U-SQL dans Azure Data Lake Analytics

## <a name="create-a-tvf"></a>Création d'une TVF

Dans le script U-SQL précédent, vous avez réutilisé EXTRACT pour lire depuis le même fichier source. Avec la fonction table U-SQL, vous pouvez encapsuler les données pour une réutilisation ultérieure.  

Le script suivant crée une TVF appelée `Searchlog()` dans la base de données et le schéma par défaut :

```usql
DROP FUNCTION IF EXISTS Searchlog;

CREATE FUNCTION Searchlog()
RETURNS @searchlog TABLE
(
            UserId          int,
            Start           DateTime,
            Region          string,
            Query           string,
            Duration        int?,
            Urls            string,
            ClickedUrls     string
)
AS BEGIN
@searchlog =
    EXTRACT UserId          int,
            Start           DateTime,
            Region          string,
            Query           string,
            Duration        int?,
            Urls            string,
            ClickedUrls     string
    FROM "/Samples/Data/SearchLog.tsv"
USING Extractors.Tsv();
RETURN;
END;
```

Le script suivant montre comment utiliser la fonction table définie dans le script précédent :

```usql
@res =
    SELECT
        Region,
        SUM(Duration) AS TotalDuration
    FROM Searchlog() AS S
GROUP BY Region
HAVING SUM(Duration) > 200;

OUTPUT @res
    TO "/output/SearchLog-use-tvf.csv"
    ORDER BY TotalDuration DESC
    USING Outputters.Csv();
```

## <a name="create-views"></a>Créer des vues

Si vous avez une expression de requête unique, vous pouvez utiliser une vue U-SQL pour encapsuler cette expression, plutôt qu’une TVF.

Le script suivant crée une vue appelée `SearchlogView` dans la base de données et le schéma par défaut :

```usql
DROP VIEW IF EXISTS SearchlogView;

CREATE VIEW SearchlogView AS  
    EXTRACT UserId          int,
            Start           DateTime,
            Region          string,
            Query           string,
            Duration        int?,
            Urls            string,
            ClickedUrls     string
    FROM "/Samples/Data/SearchLog.tsv"
USING Extractors.Tsv();
```

Le script suivant montre l'utilisation de la vue définie :

```usql
@res =
    SELECT
        Region,
        SUM(Duration) AS TotalDuration
    FROM SearchlogView
GROUP BY Region
HAVING SUM(Duration) > 200;

OUTPUT @res
    TO "/output/Searchlog-use-view.csv"
    ORDER BY TotalDuration DESC
    USING Outputters.Csv();
```

## <a name="create-tables"></a>Créer des tables
De façon similaire aux tables de base de données relationnelle, U-SQL vous permet de créer une table avec un schéma prédéfini ou de créer une table et de déduire le schéma à partir de la requête de remplissage de la table (également appelée CREATE TABLE AS SELECT ou CTAS).

Créez une base de données et deux tables avec le script suivant :

```usql
DROP DATABASE IF EXISTS SearchLogDb;
CREATE DATABASE SearchLogDb;
USE DATABASE SearchLogDb;

DROP TABLE IF EXISTS SearchLog1;
DROP TABLE IF EXISTS SearchLog2;

CREATE TABLE SearchLog1 (
            UserId          int,
            Start           DateTime,
            Region          string,
            Query           string,
            Duration        int?,
            Urls            string,
            ClickedUrls     string,

            INDEX sl_idx CLUSTERED (UserId ASC)
                DISTRIBUTED BY HASH (UserId)
);

INSERT INTO SearchLog1 SELECT * FROM master.dbo.Searchlog() AS s;

CREATE TABLE SearchLog2(
    INDEX sl_idx CLUSTERED (UserId ASC)
            DISTRIBUTED BY HASH (UserId)
) AS SELECT * FROM master.dbo.Searchlog() AS S; // You can use EXTRACT or SELECT here
```

## <a name="query-tables"></a>Tables de requête
Vous pouvez interroger des tables, par exemple celles que vous avez créées dans le script précédent, de la même manière que vous interrogez les fichiers de données. Au lieu de créer un ensemble de lignes avec EXTRACT, vous pouvez maintenant faire référence au nom de table.

Pour lire à partir des tables, modifiez le script de transformation que vous avez utilisé précédemment :

```usql
@rs1 =
    SELECT
        Region,
        SUM(Duration) AS TotalDuration
    FROM SearchLogDb.dbo.SearchLog2
GROUP BY Region;

@res =
    SELECT *
    FROM @rs1
    ORDER BY TotalDuration DESC
    FETCH 5 ROWS;

OUTPUT @res
    TO "/output/Searchlog-query-table.csv"
    ORDER BY TotalDuration DESC
    USING Outputters.Csv();
```

 >[!NOTE]
 >Actuellement, vous ne pouvez pas exécuter une instruction SELECT sur une table dans le script dans lequel vous avez créé cette table.

## <a name="next-steps"></a>Étapes suivantes
* [Vue d'ensemble de Microsoft Azure Data Lake Analytics](data-lake-analytics-overview.md)
* [Développer des scripts de U-SQL à l’aide d’outils Data Lake Tools pour Visual Studio](data-lake-analytics-data-lake-tools-get-started.md)
* [Surveiller et résoudre les problèmes des tâches Azure Data Lake Analytics à l’aide du portail Azure](data-lake-analytics-monitor-and-troubleshoot-jobs-tutorial.md)
