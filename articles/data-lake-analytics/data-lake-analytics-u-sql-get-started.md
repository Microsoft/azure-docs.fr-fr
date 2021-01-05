---
title: Bien démarrer avec le langage U-SQL dans Azure Data Lake Analytics
description: Découvrez les principes de base du langage U-SQL dans Azure Data Lake Analytics. Écrivez votre première requête à l’aide de variables sur des données supplémentaires provenant de fichiers, transformez l’ensemble de lignes et agrégez les données.
ms.reviewer: jasonh
ms.service: data-lake-analytics
ms.topic: how-to
ms.date: 06/23/2017
ms.openlocfilehash: 5920f2b5823e568d56d13107200eb0025a654db0
ms.sourcegitcommit: 8d8deb9a406165de5050522681b782fb2917762d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/20/2020
ms.locfileid: "92219919"
---
# <a name="get-started-with-u-sql-in-azure-data-lake-analytics"></a>Bien démarrer avec U-SQL dans Azure Data Lake Analytics

U-SQL est un langage qui combine le langage SQL déclaratif avec le langage C# impératif pour vous permettre de traiter des données quelle que soit l’échelle. La fonctionnalité évolutive de requête distribuée d’U-SQL vous permet d’analyser efficacement les données entre magasins relationnels comme Azure SQL Database. Avec U-SQL, vous pouvez traiter des données non structurées en appliquant des schémas de lecture et en insérant une logique personnalisée et des fonctions définies par l'utilisateur. En outre, U-SQL comprend l’extensibilité qui vous donne un contrôle précis sur l’exécution à l’échelle.

## <a name="learning-resources"></a>Ressources d’apprentissage

* Le [Didacticiel U-SQL](https://aka.ms/usqltutorial) fournit une procédure pas à pas pour la plupart du langage U-SQL. La lecture de ce document est recommandée pour tous les développeurs qui veulent apprendre le langage U-SQL.
* Pour plus d’informations sur la **syntaxe du langage U-SQL**, consultez la [Référence du langage U-SQL](/u-sql/).
* Pour comprendre la [philosophie de conception d’U-SQL](https://blogs.msdn.microsoft.com/visualstudio/2015/09/28/introducing-u-sql-a-language-that-makes-big-data-processing-easy/), consultez l’article de blog Visual Studio **Présentation d’U-SQL – Un langage qui facilite le traitement du Big Data**.

## <a name="prerequisites"></a>Conditions préalables requises

Avant de parcourir les exemples U-SQL de ce document, lisez et suivez le [Didacticiel : Développer des scripts U-SQL avec Data Lake Tools pour Visual Studio](data-lake-analytics-data-lake-tools-get-started.md). Ce didacticiel explique les mécanismes de l’utilisation d’U-SQL avec Azure Data Lake Tools pour Visual Studio.

## <a name="your-first-u-sql-script"></a>Votre premier script U-SQL

Le script U-SQL suivant est simple et nous permet d’explorer de nombreux aspects du langage U-SQL.

```usql
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

OUTPUT @searchlog
    TO "/output/SearchLog-first-u-sql.csv"
    USING Outputters.Csv();
```

Ce script n'a aucune étape de transformation. Il lit le fichier source nommé `SearchLog.tsv`, le schématise et écrit l’ensemble de lignes dans un fichier nommé SearchLog-first-u-sql.csv.

Notez le point d'interrogation en regard du type de données dans le champ `Duration`. Il signifie que le champ `Duration` pourrait avoir la valeur Null.

### <a name="key-concepts"></a>Concepts clés

* **Variables de l'ensemble de lignes**: toute expression de requête qui produit un ensemble de lignes peut être affectée à une variable. U-SQL suit le modèle d’affectation de noms variable T-SQL (`@searchlog`, par exemple) dans le script.
* Le mot-clé **EXTRACT** lit les données d’un fichier et définit le schéma à la lecture. `Extractors.Tsv` est un extracteur U-SQL intégré pour les fichiers de valeurs séparées par des tabulations. Vous pouvez développer des extracteurs personnalisés.
* **OUTPUT** écrit les données dans un fichier à partir d’un ensemble de lignes. `Outputters.Csv()` est un générateur de sortie U-SQL intégré pour créer un fichier de valeurs séparées par des virgules. Vous pouvez développer des générateurs de sortie personnalisés.

### <a name="file-paths"></a>Chemins d’accès

Les instructions EXTRACT et OUTPUT utilisent des chemins d’accès. Les chemins d’accès peuvent être absolus ou relatifs :

Le chemin d’accès absolu suivant fait référence à un fichier dans un Data Lake Store nommé `mystore` :

```usql
adl://mystore.azuredatalakestore.net/Samples/Data/SearchLog.tsv
```

Le chemin d’accès suivant commence par `"/"`. Il fait référence à un fichier dans le compte Data Lake Store par défaut :

```usql
/output/SearchLog-first-u-sql.csv
```

## <a name="use-scalar-variables"></a>Utiliser des variables scalaires

Vous pouvez également utiliser des variables scalaires pour faciliter la maintenance de votre script. Le script U-SQL précédent peut également s'écrire comme suit :

```usql
DECLARE @in  string = "/Samples/Data/SearchLog.tsv";
DECLARE @out string = "/output/SearchLog-scalar-variables.csv";
@searchlog =
    EXTRACT UserId          int,
            Start           DateTime,
            Region          string,
            Query           string,
            Duration        int?,
            Urls            string,
            ClickedUrls     string
    FROM @in
    USING Extractors.Tsv();
OUTPUT @searchlog
    TO @out
    USING Outputters.Csv();
```

## <a name="transform-rowsets"></a>Transformer des ensembles de lignes

Utilisez **SELECT** pour transformer des ensembles de lignes :

```usql
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
@rs1 =
    SELECT Start, Region, Duration
    FROM @searchlog
WHERE Region == "en-gb";
OUTPUT @rs1
    TO "/output/SearchLog-transform-rowsets.csv"
    USING Outputters.Csv();
```

La clause WHERE utilise une [expression booléenne C#](/dotnet/csharp/language-reference/operators/index). Vous pouvez utiliser le langage d'expressions C# pour faire vos propres expressions et fonctions. Vous pouvez même effectuer un filtrage plus complexe en les combinant avec des conjonctions logiques (AND) et des disjonctions (OR).

Le script suivant utilise la méthode DateTime.Parse() et une conjonction.

```usql
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
@rs1 =
    SELECT Start, Region, Duration
    FROM @searchlog
WHERE Region == "en-gb";
@rs1 =
    SELECT Start, Region, Duration
    FROM @rs1
    WHERE Start >= DateTime.Parse("2012/02/16") AND Start <= DateTime.Parse("2012/02/17");
OUTPUT @rs1
    TO "/output/SearchLog-transform-datetime.csv"
    USING Outputters.Csv();
```

 >[!NOTE]
 >La deuxième requête fonctionne sur le résultat du premier ensemble de lignes, ce qui crée une combinaison des deux filtres. Vous pouvez également réutiliser un nom de variable, et les noms ont une portée lexicale.

## <a name="aggregate-rowsets"></a>Ensembles de lignes agrégés

U-SQL vous fournit les ORDER BY, GROUP BY et les agrégations que vous connaissez déjà.

La requête suivante recherche la durée totale par région, puis affiche les cinq premières durées dans l’ordre.

Les ensembles de lignes U-SQL ne conservent pas leur ordre pour la requête suivante. Par conséquent, pour ordonner un résultat, vous devez ajouter ORDER BY à l'instruction OUTPUT :

```usql
DECLARE @outpref string = "/output/Searchlog-aggregation";
DECLARE @out1    string = @outpref+"_agg.csv";
DECLARE @out2    string = @outpref+"_top5agg.csv";
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
@rs1 =
    SELECT
        Region,
        SUM(Duration) AS TotalDuration
    FROM @searchlog
GROUP BY Region;
@res =
    SELECT *
    FROM @rs1
    ORDER BY TotalDuration DESC
    FETCH 5 ROWS;
OUTPUT @rs1
    TO @out1
    ORDER BY TotalDuration DESC
    USING Outputters.Csv();
OUTPUT @res
    TO @out2
    ORDER BY TotalDuration DESC
    USING Outputters.Csv();
```

La clause U-SQL ORDER BY exige l’utilisation de la clause FETCH dans une expression SELECT.

La clause U-SQL HAVING peut être utilisée pour restreindre le résultat aux groupes qui remplissent la condition HAVING :

```usql
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
@res =
    SELECT
        Region,
        SUM(Duration) AS TotalDuration
    FROM @searchlog
    GROUP BY Region
    HAVING SUM(Duration) > 200;
OUTPUT @res
    TO "/output/Searchlog-having.csv"
    ORDER BY TotalDuration DESC
    USING Outputters.Csv();
```

Pour les scénarios d’agrégation avancés, consultez la documentation de référence U-SQL sur les [fonctions d’agrégation, d’analytique et de référence](/u-sql/built-in-functions)

## <a name="next-steps"></a>Étapes suivantes

* [Vue d'ensemble de Microsoft Azure Data Lake Analytics](data-lake-analytics-overview.md)
* [Développer des scripts U-SQL à l’aide de Data Lake Tools pour Visual Studio](data-lake-analytics-data-lake-tools-get-started.md)