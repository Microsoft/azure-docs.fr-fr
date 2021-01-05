---
title: Comblement des écarts temporels et imputation des valeurs manquantes - Azure SQL Edge
description: Découvrez le comblement des écarts temporels et l’imputation des valeurs manquantes dans Azure SQL Edge
keywords: SQL Edge, séries chronologiques
services: sql-edge
ms.service: sql-edge
ms.topic: conceptual
author: SQLSourabh
ms.author: sourabha
ms.reviewer: sstein
ms.date: 09/22/2020
ms.openlocfilehash: c444732a497d595235ac337d7f5c71fb84f17cca
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96185587"
---
# <a name="filling-time-gaps-and-imputing-missing-values"></a>Comblement des écarts temporels et imputation des valeurs manquantes 

Lors du traitement de données de série chronologique, il arrive souvent de constater des valeurs d’attributs manquantes dans les données. Il est également possible d’observer des *écarts* temporels dans le jeu de données, qui sont dus à la nature même des données ou à des interruptions dans la collecte des données.

Par exemple, dans le cadre d’une collecte de statistiques de consommation d’énergie sur un smart device, des écarts dans ces statistiques sont observés chaque fois que l’appareil n’est pas opérationnel. De la même façon, dans un scénario de collecte de données de télémétrie sur une machine, il est possible que les différents capteurs soient configurés pour émettre des données à des fréquences différentes, ce qui entraîne des valeurs de capteurs manquantes. Prenons l’exemple de deux capteurs, mesurant la tension et la pression, qui sont configurés respectivement à une fréquence de 100 Hz et 10 Hz. Le capteur de tension émet des données tous les centièmes de seconde, tandis que le capteur de pression en émet tous les dixièmes de seconde seulement.

Le tableau suivant montre un jeu de données de télémétrie d’une machine, qui ont été collectées à des intervalles d’une seconde. 

```
timestamp               VoltageReading  PressureReading
----------------------- --------------- ----------------
2020-09-07 06:14:41.000 164.990400      97.223600
2020-09-07 06:14:42.000 162.241300      93.992800
2020-09-07 06:14:43.000 163.271200      NULL
2020-09-07 06:14:44.000 161.368100      93.403700
2020-09-07 06:14:45.000 NULL            NULL
2020-09-07 06:14:46.000 NULL            98.364800
2020-09-07 06:14:49.000 NULL            94.098300
2020-09-07 06:14:51.000 157.695700      103.359100
2020-09-07 06:14:52.000 157.019200      NULL
2020-09-07 06:14:54.000 NULL            95.352000
2020-09-07 06:14:56.000 159.183500      100.748200

```

Le jeu de données précédent présente deux caractéristiques importantes. 

- Le jeu de données ne contient pas de points de données pour plusieurs horodatages (`2020-09-07 06:14:47.000`, `2020-09-07 06:14:48.000`, `2020-09-07 06:14:50.000`, `2020-09-07 06:14:53.000` et `2020-09-07 06:14:55.000`). Ces horodatages constituent des *écarts* dans le jeu de données.  
- Il y a des valeurs manquantes, représentées par la valeur `null`, pour les lectures de tension et de pression. 

## <a name="gap-filling"></a>Comblement des écarts 

Le comblement des écarts est une technique qui permet de créer un ensemble d’horodatages contigus et ordonnés pour faciliter l’analyse des données de série chronologique. Dans Azure SQL Edge, le moyen le plus simple de combler les écarts dans le jeu de données de série chronologique consiste à définir une table temporaire et la distribution de temps souhaitée, puis à effectuer une opération `LEFT OUTER JOIN` ou `RIGHT OUTER JOIN` sur la table du jeu de données. 

En reprenant les données `MachineTelemetry` représentées ci-dessus comme exemple, la requête suivante peut être utilisée pour générer un ensemble d’horodatages contigus et ordonnés à des fins d’analyse. 

> [!NOTE]
> La requête ci-dessous génère les lignes manquantes, avec les valeurs d’horodatage et les valeurs `null` pour les attributs. 

```sql
Create Table #SeriesGenerate(dt datetime Primary key Clustered)
GO

Declare @startdate datetime = '2020-09-07 06:14:41.000', @endtime datetime = '2020-09-07 06:14:56.000'
While (@startdate <= @endtime)
BEGIN
Insert into #SeriesGenerate values (@startdate)
set @startdate = DATEADD(SECOND, 1, @startdate)
END

Select a.dt as timestamp, b.VoltageReading, b.PressureReading 
From 
#SeriesGenerate a LEFT OUTER JOIN MachineTelemetry b 
    on a.dt = b.[timestamp]
```
La requête ci-dessus retourne la sortie suivante, qui contient tous les horodatages d’*une seconde* dans la plage spécifiée.

Voici le jeu de résultats obtenu

```

timestamp               VoltageReading    PressureReading
----------------------- ----------------- ----------------
2020-09-07 06:14:41.000 164.990400        97.223600
2020-09-07 06:14:42.000 162.241300        93.992800
2020-09-07 06:14:43.000 163.271200        NULL
2020-09-07 06:14:44.000 161.368100        93.403700
2020-09-07 06:14:45.000 NULL              NULL
2020-09-07 06:14:46.000 NULL              98.364800
2020-09-07 06:14:47.000 NULL              NULL
2020-09-07 06:14:48.000 NULL              NULL
2020-09-07 06:14:49.000 NULL              94.098300
2020-09-07 06:14:50.000 NULL              NULL
2020-09-07 06:14:51.000 157.695700        103.359100
2020-09-07 06:14:52.000 157.019200        NULL
2020-09-07 06:14:53.000 NULL              NULL
2020-09-07 06:14:54.000 NULL              95.352000
2020-09-07 06:14:55.000 NULL              NULL
2020-09-07 06:14:56.000 159.183500        100.748200
```

## <a name="imputing-missing-values"></a>Imputer les valeurs manquantes

La requête précédente a généré les horodatages manquants pour l’analyse des données, mais elle n’a pas remplacé les valeurs manquantes (représentées par la valeur null) pour les mesures `voltage` et `pressure`. Dans Azure SQL Edge, une nouvelle syntaxe a été ajoutée aux fonctions T-SQL `LAST_VALUE()` et `FIRST_VALUE()`, qui fournissent des mécanismes d’imputation des valeurs manquantes sur la base des valeurs précédentes ou suivantes dans le jeu de données. 

La nouvelle syntaxe ajoute la clause `IGNORE NULLS` et `RESPECT NULLS` aux fonctions `LAST_VALUE()` et `FIRST_VALUE()`. Une requête suivante sur le jeu de données `MachineTelemetry` calcule les valeurs manquantes à l’aide de la fonction last_value, où les valeurs manquantes sont remplacées par la valeur observée précédente dans le jeu de données.

```sql
Select 
    timestamp,
    VoltageReading As OriginalVoltageValues,
    LAST_VALUE(VoltageReading) IGNORE NULLS OVER (ORDER BY timestamp) As ImputedUsingLastValue, 
    PressureReading As OriginalPressureValues,
    LAST_VALUE(PressureReading) IGNORE NULLS OVER (ORDER BY timestamp) As ImputedUsingLastValue
From 
MachineTelemetry 
order by timestamp 
```
Voici le jeu de résultats obtenu

```

timestamp               OrigVoltageVals  ImputedVoltage OrigPressureVals  ImputedPressure
----------------------- ---------------- -------------- ----------------- ----------------
2020-09-07 06:14:41.000 164.990400       164.990400     97.223600         97.223600
2020-09-07 06:14:42.000 162.241300       162.241300     93.992800         93.992800
2020-09-07 06:14:43.000 163.271200       163.271200     NULL              93.992800
2020-09-07 06:14:44.000 161.368100       161.368100     93.403700         93.403700
2020-09-07 06:14:45.000 NULL             161.368100     NULL              93.403700
2020-09-07 06:14:46.000 NULL             161.368100     98.364800         98.364800
2020-09-07 06:14:49.000 NULL             161.368100     94.098300         94.098300
2020-09-07 06:14:51.000 157.695700       157.695700     103.359100        103.359100
2020-09-07 06:14:52.000 157.019200       157.019200     NULL              103.359100
2020-09-07 06:14:54.000 NULL             157.019200     95.352000         95.352000
2020-09-07 06:14:56.000 159.183500       159.183500     100.748200        100.748200

```

La requête suivante impute les valeurs manquantes à l’aide des deux fonctions `LAST_VALUE()` et `FIRST_VALUE`. Dans la colonne de sortie `ImputedVoltage`, les valeurs manquantes sont remplacées par la valeur observée précédente, alors que dans la colonne de sortie `ImputedPressure`, les valeurs manquantes sont remplacées par la valeur observée suivante dans le jeu de données. 

```sql
Select 
    dt as timestamp, 
    VoltageReading As OrigVoltageVals,
    LAST_VALUE(VoltageReading) IGNORE NULLS OVER (ORDER BY dt) As ImputedVoltage, 
    PressureReading As OrigPressureVals,
    First_VALUE(PressureReading) IGNORE NULLS OVER (ORDER BY dt ROWS 
                    BETWEEN CURRENT ROW AND UNBOUNDED FOLLOWING) As ImputedPressure
From 
(Select a.dt, b.VoltageReading,b.PressureReading  from 
    #SeriesGenerate a 
        LEFT OUTER JOIN 
    MachineTelemetry b 
        on a.dt = b.[timestamp]) A
order by timestamp
```
Voici le jeu de résultats obtenu

```

timestamp               OrigVoltageVals  ImputedVoltage  OrigPressureVals  ImputedPressure
----------------------- ---------------- --------------- ----------------- ---------------
2020-09-07 06:14:41.000 164.990400       164.990400      97.223600         97.223600
2020-09-07 06:14:42.000 162.241300       162.241300      93.992800         93.992800
2020-09-07 06:14:43.000 163.271200       163.271200      NULL              93.403700
2020-09-07 06:14:44.000 161.368100       161.368100      93.403700         93.403700
2020-09-07 06:14:45.000 NULL             161.368100      NULL              98.364800
2020-09-07 06:14:46.000 NULL             161.368100      98.364800         98.364800
2020-09-07 06:14:47.000 NULL             161.368100      NULL              94.098300
2020-09-07 06:14:48.000 NULL             161.368100      NULL              94.098300
2020-09-07 06:14:49.000 NULL             161.368100      94.098300         94.098300
2020-09-07 06:14:50.000 NULL             161.368100      NULL              103.359100
2020-09-07 06:14:51.000 157.695700       157.695700      103.359100        103.359100
2020-09-07 06:14:52.000 157.019200       157.019200      NULL              95.352000
2020-09-07 06:14:53.000 NULL             157.019200      NULL              95.352000
2020-09-07 06:14:54.000 NULL             157.019200      95.352000         95.352000
2020-09-07 06:14:55.000 NULL             157.019200      NULL              100.748200
2020-09-07 06:14:56.000 159.183500       159.183500      100.748200        100.748200
```

> [!NOTE]
> La requête ci-dessus utilise la fonction `FIRST_VALUE()` pour remplacer les valeurs manquantes par la valeur observée suivante. Le même résultat est possible en utilisant une fonction `LAST_VALUE()` avec une clause `ORDER BY <ordering_column> DESC`.

## <a name="next-steps"></a>Étapes suivantes 

- [FIRST_VALUE (Transact-SQL)](/sql/t-sql/functions/first-value-transact-sql?toc=%2fazure%2fazure-sql-edge%2ftoc.json)
- [LAST_VALUE (Transact-SQL)](/sql/t-sql/functions/last-value-transact-sql?toc=%2fazure%2fazure-sql-edge%2ftoc.json)
- [DATE_BUCKET (Transact-SQL)](date-bucket-tsql.md)
- [Fonctions d'agrégation (Transact-SQL)](/sql/t-sql/functions/aggregate-functions-transact-sql)