---
title: Guide de programmabilité U-SQL pour Azure Data Lake
description: En savoir plus sur la vue d’ensemble U-SQL et la programmabilité UDF pour Azure Data Lake Analytics afin de créer un script USQL efficace.
ms.service: data-lake-analytics
ms.reviewer: jasonh
ms.topic: how-to
ms.date: 06/30/2017
ms.openlocfilehash: d263f2616607a96a8aa14f1ad1c06330b1b44644
ms.sourcegitcommit: df66dff4e34a0b7780cba503bb141d6b72335a96
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/02/2020
ms.locfileid: "96510860"
---
# <a name="u-sql-programmability-guide-overview"></a>Vue d’ensemble du guide de programmabilité U-SQL

U-SQL est un langage de requête spécifiquement conçu pour les charges de travail de type Big Data. L’une des fonctionnalités uniques du langage U-SQL est la combinaison d’un langage déclaratif de type SQL avec l’extensibilité et la programmabilité offertes par C#. Ce guide se concentre sur l’extensibilité et la programmabilité du langage U-SQL que permet C#.

## <a name="requirements"></a>Spécifications

Téléchargez et installez [Azure Data Lake Tools pour Visual Studio](https://www.microsoft.com/download/details.aspx?id=49504).

## <a name="get-started-with-u-sql"></a>Bien démarrer avec U-SQL  

Examinez le script U-SQL suivant :

```usql
@a  = 
  SELECT * FROM 
    (VALUES
       ("Contoso",   1500.0, "2017-03-39"),
       ("Woodgrove", 2700.0, "2017-04-10")
    ) AS D( customer, amount, date );

@results =
  SELECT
    customer,
    amount,
    date
  FROM @a;    
```

Ce script définit deux ensembles de lignes : `@a` et `@results`. L’ensemble de lignes `@results` est défini à partir de `@a`.

## <a name="c-types-and-expressions-in-u-sql-script"></a>Types et expressions C# dans les scripts U-SQL

Une Expression U-SQL est une expression C# combinée à des opérations logiques U-SQL comme `AND`, `OR` et `NOT`. Les expressions U-SQL peuvent être utilisées avec SELECT, EXTRACT, WHERE, HAVING, GROUP BY et DECLARE. Par exemple, le script suivant analyse une chaîne en tant que valeur DateTime.

```usql
@results =
  SELECT
    customer,
    amount,
    DateTime.Parse(date) AS date
  FROM @a;    
```

L’extrait de code suivant analyse une chaîne en tant que valeur DateTime dans une instruction DECLARE.

```usql
DECLARE @d = DateTime.Parse("2016/01/01");
```

### <a name="use-c-expressions-for-data-type-conversions"></a>Utiliser des expressions C# pour les conversions de types de données

L’exemple suivant montre comment faire une conversion de données datetime en utilisant des expressions C#. Dans ce scénario, les données datetime de chaîne sont converties en données datetime standard avec la notation d’heure 00:00:00 (minuit).

```usql
DECLARE @dt = "2016-07-06 10:23:15";

@rs1 =
  SELECT 
    Convert.ToDateTime(Convert.ToDateTime(@dt).ToString("yyyy-MM-dd")) AS dt,
    dt AS olddt
  FROM @rs0;

OUTPUT @rs1 
  TO @output_file 
  USING Outputters.Text();
```

### <a name="use-c-expressions-for-todays-date"></a>Utiliser des expressions C# pour la date du jour

Pour extraire la date du jour, nous pouvons utiliser l’expression C# suivante : `DateTime.Now.ToString("M/d/yyyy")`

Voici un exemple montrant comment utiliser cette expression dans un script :

```usql
@rs1 =
  SELECT
    MAX(guid) AS start_id,
    MIN(dt) AS start_time,
    MIN(Convert.ToDateTime(Convert.ToDateTime(dt<@default_dt?@default_dt:dt).ToString("yyyy-MM-dd"))) AS start_zero_time,
    MIN(USQL_Programmability.CustomFunctions.GetFiscalPeriod(dt)) AS start_fiscalperiod,
    DateTime.Now.ToString("M/d/yyyy") AS Nowdate,
    user,
    des
  FROM @rs0
  GROUP BY user, des;
```
## <a name="using-net-assemblies"></a>Avec des assemblys .NET

Le modèle d’extensibilité de U-SQL dépend beaucoup de la possibilité d’ajouter du code personnalisé à partir des assemblys .NET. 

### <a name="register-a-net-assembly"></a>Enregistrer un assembly .NET

Utilisez l’instruction `CREATE ASSEMBLY` pour placer un assembly .NET dans une base de données U-SQL. Par la suite, les scripts U-SQL peuvent utiliser ces assemblys au moyen de l’instruction `REFERENCE ASSEMBLY`. 

Le code suivant illustre comment enregistrer un assembly :

```usql
CREATE ASSEMBLY MyDB.[MyAssembly]
   FROM "/myassembly.dll";
```

Le code suivant illustre comment référencer un assembly :

```usql
REFERENCE ASSEMBLY MyDB.[MyAssembly];
```

Consultez les [instructions d’enregistrement des assemblys](/archive/blogs/azuredatalake/how-to-register-u-sql-assemblies-in-your-u-sql-catalog) qui traitent de ce sujet plus en détail.


### <a name="use-assembly-versioning"></a>Utilisez le contrôle de version des assemblys
Actuellement, U-SQL utilise .NET Framework version 4.7.2. Par conséquent, vérifiez que vos propres assemblys sont compatibles avec cette version du runtime.

Comme mentionné précédemment, U-SQL exécute le code dans un format 64 bits (x64). Par conséquent, assurez-vous que votre code est compilé pour s’exécuter sur des systèmes x64. Dans le cas contraire, vous recevez l’erreur de format incorrect vue précédemment.

Chaque DLL d’assembly ou fichier de ressources chargé, comme un runtime différent, un assembly natif ou un fichier de configuration peut être de 400 Mo maximum. La taille totale des ressources déployées, via l’instruction DEPLOY RESOURCE ou via des références aux assemblys et leurs fichiers supplémentaires, ne peut pas dépasser 3 Go.

Enfin, notez que chaque base de données U-SQL ne peut contenir qu’une seule version d’un assembly donné. Par exemple, si vous avez besoin à la fois des versions 7 et 8 de la bibliothèque NewtonSoft Json.NET, vous devez les enregistrer dans deux bases de données distinctes. En outre, chaque script ne peut faire référence qu’à une seule version d’une DLL d’assembly donnée. À cet égard, U-SQL suit la sémantique de gestion et de contrôle de version d’assembly C#.

## <a name="use-user-defined-functions-udf"></a>Utiliser les fonctions définies par l’utilisateur : Fonctions définies par l'utilisateur
Les fonctions U-SQL définies par l’utilisateur (ou UDF) sont des routines de programmation qui acceptent des paramètres, effectuent une action (telle qu’un calcul complexe) et retournent le résultat de celle-ci en tant que valeur. La valeur de retour d’une fonction définie par l’utilisateur ne peut être qu’une valeur scalaire unique. Une fonction définie par l’utilisateur U-SQL peut être appelée dans un script U-SQL de base comme toute autre fonction scalaire C#.

Nous vous recommandons d’initialiser les fonctions définies par l’utilisateur U-SQL en tant que **publiques** et **statiques**.

```usql
public static string MyFunction(string param1)
{
    return "my result";
}
```

Examinons d’abord l’exemple simple de création d’une fonction définie par l’utilisateur.

Dans ce scénario d’utilisation, nous devons déterminer la période fiscale, le trimestre et le mois fiscal de la première connexion de l’utilisateur spécifique inclus. Le premier mois fiscal de l’année dans notre scénario est juin.

Pour calculer la période fiscale, nous introduisons la fonction C# suivante :

```usql
public static string GetFiscalPeriod(DateTime dt)
{
    int FiscalMonth=0;
    if (dt.Month < 7)
    {
    FiscalMonth = dt.Month + 6;
    }
    else
    {
    FiscalMonth = dt.Month - 6;
    }

    int FiscalQuarter=0;
    if (FiscalMonth >=1 && FiscalMonth<=3)
    {
    FiscalQuarter = 1;
    }
    if (FiscalMonth >= 4 && FiscalMonth <= 6)
    {
    FiscalQuarter = 2;
    }
    if (FiscalMonth >= 7 && FiscalMonth <= 9)
    {
    FiscalQuarter = 3;
    }
    if (FiscalMonth >= 10 && FiscalMonth <= 12)
    {
    FiscalQuarter = 4;
    }

    return "Q" + FiscalQuarter.ToString() + ":P" + FiscalMonth.ToString();
}
```

La fonction calcule simplement le trimestre et le mois fiscaux, puis retourne une valeur de chaîne. Pour le mois de juin, premier mois du premier trimestre fiscal, nous utilisons « Q1:P1 ». Pour le mois de juillet, nous utilisons « Q1:P2 », etc.

Il s’agit d’une fonction C# standard que nous allons utiliser dans notre projet U-SQL.

La section code-behind dans ce scénario se présente comme suit :

```usql
using Microsoft.Analytics.Interfaces;
using Microsoft.Analytics.Types.Sql;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

namespace USQL_Programmability
{
    public class CustomFunctions
    {
        public static string GetFiscalPeriod(DateTime dt)
        {
            int FiscalMonth=0;
            if (dt.Month < 7)
            {
                FiscalMonth = dt.Month + 6;
            }
            else
            {
                FiscalMonth = dt.Month - 6;
            }

            int FiscalQuarter=0;
            if (FiscalMonth >=1 && FiscalMonth<=3)
            {
                FiscalQuarter = 1;
            }
            if (FiscalMonth >= 4 && FiscalMonth <= 6)
            {
                FiscalQuarter = 2;
            }
            if (FiscalMonth >= 7 && FiscalMonth <= 9)
            {
                FiscalQuarter = 3;
            }
            if (FiscalMonth >= 10 && FiscalMonth <= 12)
            {
                FiscalQuarter = 4;
            }

            return "Q" + FiscalQuarter.ToString() + ":" + FiscalMonth.ToString();
        }
    }
}
```

Nous allons maintenant appeler cette fonction à partir du script U-SQL de base. Pour ce faire, nous devons fournir un nom complet pour la fonction, qui inclut l’espace de noms, dans ce cas : NameSpace.Class.Function(paramètre).
```usql
USQL_Programmability.CustomFunctions.GetFiscalPeriod(dt)
```

Le script de base U-SQL réel est le suivant :
```usql
DECLARE @input_file string = @"\usql-programmability\input_file.tsv";
DECLARE @output_file string = @"\usql-programmability\output_file.tsv";

@rs0 =
    EXTRACT
            guid Guid,
        dt DateTime,
            user String,
            des String
    FROM @input_file USING Extractors.Tsv();

DECLARE @default_dt DateTime = Convert.ToDateTime("06/01/2016");

@rs1 =
    SELECT
        MAX(guid) AS start_id,
    MIN(dt) AS start_time,
        MIN(Convert.ToDateTime(Convert.ToDateTime(dt<@default_dt?@default_dt:dt).ToString("yyyy-MM-dd"))) AS start_zero_time,
        MIN(USQL_Programmability.CustomFunctions.GetFiscalPeriod(dt)) AS start_fiscalperiod,
        user,
        des
    FROM @rs0
    GROUP BY user, des;

OUTPUT @rs1 
    TO @output_file 
    USING Outputters.Text();
```

Le fichier de sortie résultant de l’exécution du script est le suivant :

```output
0d8b9630-d5ca-11e5-8329-251efa3a2941,2016-02-11T07:04:17.2630000-08:00,2016-06-01T00:00:00.0000000,"Q3:8","User1",""

20843640-d771-11e5-b87b-8b7265c75a44,2016-02-11T07:04:17.2630000-08:00,2016-06-01T00:00:00.0000000,"Q3:8","User2",""

301f23d2-d690-11e5-9a98-4b4f60a1836f,2016-02-11T09:01:33.9720000-08:00,2016-06-01T00:00:00.0000000,"Q3:8","User3",""
```

Cet exemple illustre une utilisation simple d’une fonction définie par l’utilisateur inline dans U-SQL.

### <a name="keep-state-between-udf-invocations"></a>Conserver l’état entre des appels de fonction définie par l’utilisateur
Des objets de programmabilité C# U-SQL peuvent être plus sophistiqués en utilisant une interactivité via des variables globales code-behind. Examinons le scénario d’utilisation professionnelle suivant :

Dans les grandes organisations, les utilisateurs peuvent basculer entre divers types d’applications internes. Celles-ci peuvent inclure Microsoft Dynamics CRM, Power BI, etc. Les clients pourraient souhaiter appliquer une analyse de télémétrie en relation avec la manière dont les utilisateurs basculent entre les différentes applications, la tendance d’utilisation, etc. L’objectif de l’entreprise est d’optimiser l’utilisation des applications. Ils pourraient également avoir envie de combiner différentes applications ou routines de connexion spécifiques.

Pour atteindre cet objectif, nous devons déterminer des ID de session et un décalage avec la dernière session ayant eu lieu.

Nous devons trouver une connexion précédente, puis affecter cette connexion à toutes les sessions générées pour la même application. La première difficulté est que le script U-SQL de base ne nous permet pas d’appliquer des calculs à des colonnes déjà calculées avec la fonction LAG. La deuxième difficulté est que nous devons garder la session spécifique pour toutes les sessions au cours de la même période.

Pour résoudre ce problème, nous utilisons une variable globale à l’intérieur de la section code-behind : `static public string globalSession;`.

Cette variable globale est appliquée à l’ensemble de lignes lors de l’exécution de notre script.

Voici la section code-behind de notre programme U-SQL :

```csharp
using Microsoft.Analytics.Interfaces;
using Microsoft.Analytics.Types.Sql;
using System;
using System.Collections.Generic;
using System.Linq;
using System.Text;

namespace USQLApplication21
{
    public class UserSession
    {
        static public string globalSession;
        static public string StampUserSession(string eventTime, string PreviousRow, string Session)
        {

            if (!string.IsNullOrEmpty(PreviousRow))
            {
                double timeGap = Convert.ToDateTime(eventTime).Subtract(Convert.ToDateTime(PreviousRow)).TotalMinutes;
                if (timeGap <= 60) {return Session;}
                else {return Guid.NewGuid().ToString();}
            }
            else {return Guid.NewGuid().ToString();}

        }

        static public string getStampUserSession(string Session)
        {
            if (Session != globalSession && !string.IsNullOrEmpty(Session)) { globalSession = Session; }
            return globalSession;
        }

    }
}
```

Cet exemple montre la variable globale `static public string globalSession;` utilisée dans la fonction `getStampUserSession` et réinitialisée à chaque modification du paramètre Session.

Le script de base U-SQL est le suivant :

```usql
DECLARE @in string = @"\UserSession\test1.tsv";
DECLARE @out1 string = @"\UserSession\Out1.csv";
DECLARE @out2 string = @"\UserSession\Out2.csv";
DECLARE @out3 string = @"\UserSession\Out3.csv";

@records =
    EXTRACT DataId string,
            EventDateTime string,           
            UserName string,
            UserSessionTimestamp string

    FROM @in
    USING Extractors.Tsv();

@rs1 =
    SELECT 
        EventDateTime,
        UserName,
    LAG(EventDateTime, 1) 
        OVER(PARTITION BY UserName ORDER BY EventDateTime ASC) AS prevDateTime,          
        string.IsNullOrEmpty(LAG(EventDateTime, 1) 
        OVER(PARTITION BY UserName ORDER BY EventDateTime ASC)) AS Flag,           
        USQLApplication21.UserSession.StampUserSession
           (
            EventDateTime,
            LAG(EventDateTime, 1) OVER(PARTITION BY UserName ORDER BY EventDateTime ASC),
            LAG(UserSessionTimestamp, 1) OVER(PARTITION BY UserName ORDER BY EventDateTime ASC)
           ) AS UserSessionTimestamp
    FROM @records;

@rs2 =
    SELECT 
        EventDateTime,
        UserName,
        LAG(EventDateTime, 1) 
        OVER(PARTITION BY UserName ORDER BY EventDateTime ASC) AS prevDateTime,
        string.IsNullOrEmpty( LAG(EventDateTime, 1) OVER(PARTITION BY UserName ORDER BY EventDateTime ASC)) AS Flag,
        USQLApplication21.UserSession.getStampUserSession(UserSessionTimestamp) AS UserSessionTimestamp
    FROM @rs1
    WHERE UserName != "UserName";

OUTPUT @rs2
    TO @out2
    ORDER BY UserName, EventDateTime ASC
    USING Outputters.Csv();
```

La fonction `USQLApplication21.UserSession.getStampUserSession(UserSessionTimestamp)` est appelée ici pendant le deuxième calcul d’ensemble de lignes de mémoire. Elle transmet la colonne `UserSessionTimestamp` et retourne la valeur jusqu’à ce que `UserSessionTimestamp` soit modifié.

Le fichier de sortie se présente comme suit :

```output
"2016-02-19T07:32:36.8420000-08:00","User1",,True,"72a0660e-22df-428e-b672-e0977007177f"
"2016-02-17T11:52:43.6350000-08:00","User2",,True,"4a0cd19a-6e67-4d95-a119-4eda590226ba"
"2016-02-17T11:59:08.8320000-08:00","User2","2016-02-17T11:52:43.6350000-08:00",False,"4a0cd19a-6e67-4d95-a119-4eda590226ba"
"2016-02-11T07:04:17.2630000-08:00","User3",,True,"51860a7a-1610-4f74-a9ea-69d5eef7cd9c"
"2016-02-11T07:10:33.9720000-08:00","User3","2016-02-11T07:04:17.2630000-08:00",False,"51860a7a-1610-4f74-a9ea-69d5eef7cd9c"
"2016-02-15T21:27:41.8210000-08:00","User3","2016-02-11T07:10:33.9720000-08:00",False,"4d2bc48d-bdf3-4591-a9c1-7b15ceb8e074"
"2016-02-16T05:48:49.6360000-08:00","User3","2016-02-15T21:27:41.8210000-08:00",False,"dd3006d0-2dcd-42d0-b3a2-bc03dd77c8b9"
"2016-02-16T06:22:43.6390000-08:00","User3","2016-02-16T05:48:49.6360000-08:00",False,"dd3006d0-2dcd-42d0-b3a2-bc03dd77c8b9"
"2016-02-17T16:29:53.2280000-08:00","User3","2016-02-16T06:22:43.6390000-08:00",False,"2fa899c7-eecf-4b1b-a8cd-30c5357b4f3a"
"2016-02-17T16:39:07.2430000-08:00","User3","2016-02-17T16:29:53.2280000-08:00",False,"2fa899c7-eecf-4b1b-a8cd-30c5357b4f3a"
"2016-02-17T17:20:39.3220000-08:00","User3","2016-02-17T16:39:07.2430000-08:00",False,"2fa899c7-eecf-4b1b-a8cd-30c5357b4f3a"
"2016-02-19T05:23:54.5710000-08:00","User3","2016-02-17T17:20:39.3220000-08:00",False,"6ca7ed80-c149-4c22-b24b-94ff5b0d824d"
"2016-02-19T05:48:37.7510000-08:00","User3","2016-02-19T05:23:54.5710000-08:00",False,"6ca7ed80-c149-4c22-b24b-94ff5b0d824d"
"2016-02-19T06:40:27.4830000-08:00","User3","2016-02-19T05:48:37.7510000-08:00",False,"6ca7ed80-c149-4c22-b24b-94ff5b0d824d"
"2016-02-19T07:27:37.7550000-08:00","User3","2016-02-19T06:40:27.4830000-08:00",False,"6ca7ed80-c149-4c22-b24b-94ff5b0d824d"
"2016-02-19T19:35:40.9450000-08:00","User3","2016-02-19T07:27:37.7550000-08:00",False,"3f385f0b-3e68-4456-ac74-ff6cef093674"
"2016-02-20T00:07:37.8250000-08:00","User3","2016-02-19T19:35:40.9450000-08:00",False,"685f76d5-ca48-4c58-b77d-bd3a9ddb33da"
"2016-02-11T09:01:33.9720000-08:00","User4",,True,"9f0cf696-c8ba-449a-8d5f-1ca6ed8f2ee8"
"2016-02-17T06:30:38.6210000-08:00","User4","2016-02-11T09:01:33.9720000-08:00",False,"8b11fd2a-01bf-4a5e-a9af-3c92c4e4382a"
"2016-02-17T22:15:26.4020000-08:00","User4","2016-02-17T06:30:38.6210000-08:00",False,"4e1cb707-3b5f-49c1-90c7-9b33b86ca1f4"
"2016-02-18T14:37:27.6560000-08:00","User4","2016-02-17T22:15:26.4020000-08:00",False,"f4e44400-e837-40ed-8dfd-2ea264d4e338"
"2016-02-19T01:20:31.4800000-08:00","User4","2016-02-18T14:37:27.6560000-08:00",False,"2136f4cf-7c7d-43c1-8ae2-08f4ad6a6e08"
```

Cet exemple illustre un scénario d’utilisation plus complexe dans lequel nous utilisons une variable globale à l’intérieur de la section code-behind, qui est appliquée à l’ensemble de lignes de mémoire.

## <a name="next-steps"></a>Étapes suivantes
* [Guide de programmabilité U-SQL - UDT et UDAGG](data-lake-analytics-u-sql-programmability-guide-UDT-AGG.md)
* [Guide de programmabilité U-SQL - UDO](data-lake-analytics-u-sql-programmability-guide-UDO.md)