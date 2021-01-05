---
title: 'Tutoriel : Utiliser un pool SQL serverless pour analyser des jeux de données Azure Open Datasets dans Azure Synapse Studio'
description: Ce tutoriel explique comment effectuer facilement des analyses exploratoires de données en combinant différents jeux de données Azure Open Datasets à l’aide d’un pool SQL serverless, et comment visualiser les résultats dans Azure Synapse Studio.
services: synapse-analytics
author: azaricstefan
ms.service: synapse-analytics
ms.topic: tutorial
ms.subservice: sql
ms.date: 11/20/2020
ms.author: stefanazaric
ms.reviewer: jrasnick
ms.openlocfilehash: d37597f8667c461e8d61f8214483f57eb702c2a0
ms.sourcegitcommit: 273c04022b0145aeab68eb6695b99944ac923465
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97007549"
---
# <a name="tutorial-explore-and-analyze-data-lakes-with-serverless-sql-pool"></a>Tutoriel : Explorer et analyser des lacs de données avec un pool SQL serverless

Dans ce tutoriel, vous allez apprendre à effectuer des analyses de données exploratoires. Vous combinerez différents jeux de données Azure Open Datasets à l’aide d’un pool SQL serverless. Vous visualiserez ensuite les résultats dans Synapse Studio pour Azure Synapse Analytics.

La fonction OPENROWSET (BULK...) vous permet d’accéder aux fichiers dans le stockage Azure. La fonction [OPENROWSET](develop-openrowset.md) lit le contenu d’une source de données distante (par exemple, un fichier) et retourne le contenu sous la forme d’un ensemble de lignes.

## <a name="automatic-schema-inference"></a>Inférence de schéma automatique

Les données étant stockées au format de fichier Parquet, l’inférence de schéma automatique est disponible. Il est donc possible d’interroger facilement les données sans lister les types de données de toutes les colonnes dans les fichiers. Vous pouvez aussi utiliser le mécanisme de colonne virtuelle et la fonction filepath pour filtrer un certain sous-ensemble de fichiers.

Commençons par nous familiariser avec les données sur les taxis de la ville de New York en exécutant la requête suivante :

```sql
SELECT TOP 100 * FROM
    OPENROWSET(
        BULK 'https://azureopendatastorage.blob.core.windows.net/nyctlc/yellow/puYear=*/puMonth=*/*.parquet',
        FORMAT='PARQUET'
    ) AS [nyc]
```

Le [jeu de données Taxis de la ville de New York](https://azure.microsoft.com/services/open-datasets/catalog/nyc-taxi-limousine-commission-yellow-taxi-trip-records/) comprend les éléments suivants :

- Dates et heures de début et de fin de trajet.
- Emplacements de début et de fin de trajet. 
- Distance des trajets.
- Tarifs détaillés.
- Types de tarifs.
- Types de paiement. 
- Nombre de passagers indiqué par le chauffeur.

De même, vous pouvez interroger le jeu de données sur les jours fériés à l’aide de la requête suivante :

```sql
SELECT TOP 100 * FROM
    OPENROWSET(
        BULK 'https://azureopendatastorage.blob.core.windows.net/holidaydatacontainer/Processed/*.parquet',
        FORMAT='PARQUET'
    ) AS [holidays]
```

Enfin, vous pouvez également interroger le jeu de données sur les données météorologiques à l’aide de la requête suivante :

```sql
SELECT
    TOP 100 *
FROM  
    OPENROWSET(
        BULK 'https://azureopendatastorage.blob.core.windows.net/isdweatherdatacontainer/ISDWeather/year=*/month=*/*.parquet',
        FORMAT='PARQUET'
    ) AS [weather]
```

Vous pouvez en savoir plus sur la signification des différentes colonnes dans les descriptions des jeux de données : 
- [Taxis de la ville de New York](https://azure.microsoft.com/services/open-datasets/catalog/nyc-taxi-limousine-commission-yellow-taxi-trip-records/)
- [Jours fériés](https://azure.microsoft.com/services/open-datasets/catalog/public-holidays/)
- [Données météorologiques](https://azure.microsoft.com/services/open-datasets/catalog/noaa-integrated-surface-data/)

## <a name="time-series-seasonality-and-outlier-analysis"></a>Analyse des séries chronologiques, de la saisonnalité et des valeurs hors norme

Vous pouvez facilement totaliser le nombre annuel de courses de taxi à l’aide de la requête suivante :

```sql
SELECT
    YEAR(tpepPickupDateTime) AS current_year,
    COUNT(*) AS rides_per_year
FROM
    OPENROWSET(
        BULK 'https://azureopendatastorage.blob.core.windows.net/nyctlc/yellow/puYear=*/puMonth=*/*.parquet',
        FORMAT='PARQUET'
    ) AS [nyc]
WHERE nyc.filepath(1) >= '2009' AND nyc.filepath(1) <= '2019'
GROUP BY YEAR(tpepPickupDateTime)
ORDER BY 1 ASC
```

L’extrait de code suivant présente le résultat pour le nombre annuel de courses de taxi :

![Extrait du résultat du nombre annuel de courses de taxi](./media/tutorial-data-analyst/yearly-taxi-rides.png)

Vous pouvez visualiser les données dans Synapse Studio en passant de la vue **Table** à la vue **Graphique**. Vous pouvez choisir parmi différents types de graphiques, par exemple **Aires**, **Barres**, **Histogramme**, **Courbes**, **Secteurs** et **Nuages de points**. Dans le cas présent, tracez l’**histogramme** avec la colonne **Category** définie sur **current_year** :

![Histogramme présentant les courses effectuées par an](./media/tutorial-data-analyst/column-chart-rides-year.png)

À partir de cette visualisation, vous pouvez voir une tendance à la baisse du nombre de courses au fil des ans. Cette baisse est vraisemblablement due à une augmentation récente du nombre d’entreprises de covoiturage.

> [!NOTE]
> Au moment de la rédaction de ce tutoriel, les données pour 2019 sont incomplètes. En conséquence, il y a une forte baisse du nombre de courses pour cette année.

À présent, concentrons l’analyse sur une seule année, par exemple, 2016. La requête suivante retourne le nombre quotidien de courses au cours de cette année :

```sql
SELECT
    CAST([tpepPickupDateTime] AS DATE) AS [current_day],
    COUNT(*) as rides_per_day
FROM
    OPENROWSET(
        BULK 'https://azureopendatastorage.blob.core.windows.net/nyctlc/yellow/puYear=*/puMonth=*/*.parquet',
        FORMAT='PARQUET'
    ) AS [nyc]
WHERE nyc.filepath(1) = '2016'
GROUP BY CAST([tpepPickupDateTime] AS DATE)
ORDER BY 1 ASC
```

L’extrait de code suivant présente le résultat de cette requête :

![Extrait du résultat du nombre quotidien de courses pour 2016](./media/tutorial-data-analyst/daily-rides.png)

Là encore, vous pouvez facilement visualiser les données en traçant l’**histogramme** avec la colonne **Category** définie sur **current_day** et la colonne **Legend (series)** définie sur **rides_per_day**.

![Histogramme présentant le nombre quotidien de courses pour 2016](./media/tutorial-data-analyst/column-chart-daily-rides.png)

Dans le graphique en nuage de points, vous pouvez voir qu’il existe un modèle hebdomadaire, avec les samedis comme jour de pointe. Pendant les mois d’été, il y a moins de courses de taxi en raison des vacances. Notez également des baisses importantes du nombre de courses de taxi, sans schéma clair de quand et pourquoi elles se produisent.

Ensuite, voyons s’il existe une corrélation entre la baisse du nombre de courses et les jours fériés. Nous pouvons voir s’il existe une corrélation en joignant le jeu de données sur les courses de taxi de la ville de New York au jeu de données sur les jours fériés :

```sql
WITH taxi_rides AS
(
    SELECT
        CAST([tpepPickupDateTime] AS DATE) AS [current_day],
        COUNT(*) as rides_per_day
    FROM  
        OPENROWSET(
            BULK 'https://azureopendatastorage.blob.core.windows.net/nyctlc/yellow/puYear=*/puMonth=*/*.parquet',
            FORMAT='PARQUET'
        ) AS [nyc]
    WHERE nyc.filepath(1) = '2016'
    GROUP BY CAST([tpepPickupDateTime] AS DATE)
),
public_holidays AS
(
    SELECT
        holidayname as holiday,
        date
    FROM
        OPENROWSET(
            BULK 'https://azureopendatastorage.blob.core.windows.net/holidaydatacontainer/Processed/*.parquet',
            FORMAT='PARQUET'
        ) AS [holidays]
    WHERE countryorregion = 'United States' AND YEAR(date) = 2016
)
SELECT
*
FROM taxi_rides t
LEFT OUTER JOIN public_holidays p on t.current_day = p.date
ORDER BY current_day ASC
```

![Visualisation des résultats des jeux de données sur les courses de taxi de la ville de New York et sur les jours fériés](./media/tutorial-data-analyst/rides-public-holidays.png)

Cette fois, nous voulons mettre en évidence le nombre de courses de taxi pendant les jours fériés. À cet effet, nous choisissons **none** pour la colonne **Category**, et **rides_per_day** et **holiday** comme colonnes **Legend (series)** .

![Graphique en nuage de points présentant le nombre de courses de taxi pendant les jours fériés](./media/tutorial-data-analyst/plot-chart-public-holidays.png)

À partir du graphique en nuage de points, vous pouvez voir que pendant les jours fériés, le nombre de courses de taxi est plus faible. Il existe toujours une forte chute inexpliquée le 23 janvier. Vérifions la météo de ce jour-là à New York en interrogeant le jeu de données sur les données météorologiques :

```sql
SELECT
    AVG(windspeed) AS avg_windspeed,
    MIN(windspeed) AS min_windspeed,
    MAX(windspeed) AS max_windspeed,
    AVG(temperature) AS avg_temperature,
    MIN(temperature) AS min_temperature,
    MAX(temperature) AS max_temperature,
    AVG(sealvlpressure) AS avg_sealvlpressure,
    MIN(sealvlpressure) AS min_sealvlpressure,
    MAX(sealvlpressure) AS max_sealvlpressure,
    AVG(precipdepth) AS avg_precipdepth,
    MIN(precipdepth) AS min_precipdepth,
    MAX(precipdepth) AS max_precipdepth,
    AVG(snowdepth) AS avg_snowdepth,
    MIN(snowdepth) AS min_snowdepth,
    MAX(snowdepth) AS max_snowdepth
FROM
    OPENROWSET(
        BULK 'https://azureopendatastorage.blob.core.windows.net/isdweatherdatacontainer/ISDWeather/year=*/month=*/*.parquet',
        FORMAT='PARQUET'
    ) AS [weather]
WHERE countryorregion = 'US' AND CAST([datetime] AS DATE) = '2016-01-23' AND stationname = 'JOHN F KENNEDY INTERNATIONAL AIRPORT'
```

![Visualisation des résultats du jeu de données sur les données météorologiques](./media/tutorial-data-analyst/weather-data-set-visualization.png)

Les résultats de la requête indiquent que la chute du nombre de courses de taxi était liée aux raisons suivantes :

- Il y avait du blizzard ce jour-là à New-York, avec de fortes chutes de neige (environ 30 cm).
- Il faisait froid (la température était inférieure à zéro degré Celsius).
- Il y avait du vent (environ 10 m/s).

Ce tutoriel vous a montré comment un analyste de données peut rapidement effectuer une analyse exploratoire des données, combiner facilement différents jeux de données à l’aide d’un pool SQL serverless et visualiser les résultats à l’aide d’Azure Synapse Studio.

## <a name="next-steps"></a>Étapes suivantes

Pour savoir comment connecter un pool SQL serverless à Power BI Desktop et créer des rapports, consultez [Connecter un pool SQL serverless à Power BI Desktop et créer des rapports](tutorial-connect-power-bi-desktop.md).

Pour découvrir comment utiliser des tables externes dans un pool SQL serverless, consultez [Utiliser des tables externes avec Synapse SQL](develop-tables-external-tables.md?tabs=sql-pool).
 
