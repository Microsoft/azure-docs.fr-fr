---
title: Modèles de requête courants dans Azure Stream Analytics
description: Cet article décrit plusieurs modèles et conceptions de requête courants pratiques dans les travaux Azure Stream Analytics.
services: stream-analytics
author: rodrigoaatmicrosoft
ms.author: rodrigoa
ms.reviewer: mamccrea
ms.service: stream-analytics
ms.topic: how-to
ms.date: 12/18/2019
ms.custom: devx-track-js
ms.openlocfilehash: f0c5363cfec42ba78ee6c41a1970211518b74a71
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93127533"
---
# <a name="common-query-patterns-in-azure-stream-analytics"></a>Modèles de requête courants dans Azure Stream Analytics

Les requêtes dans Azure Stream Analytics sont exprimées dans un langage de requête de type SQL. Les constructions de langage sont documentées dans le guide [Stream Analytics query language reference](/stream-analytics-query/stream-analytics-query-language-reference) (Informations de référence sur le langage de requête Stream Analytics). 

La conception de requêtes peut exprimer une logique de transmission simple pour déplacer des données d’événement d’un flux d’entrée à un magasin de données de sortie. Elle peut également effectuer une analyse enrichie temporelle et de critères spéciaux, afin de calculer les agrégats dans différentes fenêtres de temps, comme dans le guide [Créer une solution IoT à l’aide de Stream Analytics](stream-analytics-build-an-iot-solution-using-stream-analytics.md). Vous pouvez joindre des données provenant de plusieurs entrées pour combiner des événements de streaming, et effectuer également des recherches sur les données de référence statiques pour enrichir les valeurs des événements. Vous pouvez aussi écrire des données vers plusieurs sorties.

Cet article décrit des solutions pour plusieurs modèles de requête courants, inspirés de scénarios réels.

## <a name="supported-data-formats"></a>Formats de données pris en charge

Azure Stream Analytics prend en charge le traitement des événements dans les formats de données CSV, JSON et Avro.

JSON et Avro peuvent contenir des types complexes, comme des objets imbriqués (enregistrements) ou des tableaux. Pour plus d’informations sur le fonctionnement de ces types de données complexes, reportez-vous à l’article [Analyse des données JSON et AVRO](stream-analytics-parsing-json.md).

## <a name="send-data-to-multiple-outputs"></a>Envoyer des données à plusieurs sorties

Vous pouvez utiliser plusieurs instructions **SELECT** pour sortir des données vers différents récepteurs de sortie. Par exemple, une instruction **SELECT** peut produire une alerte basée sur un seuil, tandis qu’une autre peut produire des événements dans Stockage Blob.

**Entrée** :

| Marque | Temps |
| --- | --- |
| Make1 |2015-01-01T00:00:01.0000000Z |
| Make1 |2015-01-01T00:00:02.0000000Z |
| Make2 |2015-01-01T00:00:01.0000000Z |
| Make2 |2015-01-01T00:00:02.0000000Z |
| Make2 |2015-01-01T00:00:03.0000000Z |

**Output ArchiveOutput**  :

| Marque | Temps |
| --- | --- |
| Make1 |2015-01-01T00:00:01.0000000Z |
| Make1 |2015-01-01T00:00:02.0000000Z |
| Make2 |2015-01-01T00:00:01.0000000Z |
| Make2 |2015-01-01T00:00:02.0000000Z |
| Make2 |2015-01-01T00:00:03.0000000Z |

**Output AlertOutput**  :

| Marque | Temps | Count |
| --- | --- | --- |
| Make2 |2015-01-01T00:00:10.0000000Z |3 |

**Requête** :

```SQL
SELECT
    *
INTO
    ArchiveOutput
FROM
    Input TIMESTAMP BY Time

SELECT
    Make,
    System.TimeStamp() AS Time,
    COUNT(*) AS [Count]
INTO
    AlertOutput
FROM
    Input TIMESTAMP BY Time
GROUP BY
    Make,
    TumblingWindow(second, 10)
HAVING
    [Count] >= 3
```

La clause **INTO** indique à Stream Analytics la sortie sur laquelle écrire les données. La première instruction **SELECT** définit une requête directe qui reçoit des données de l’entrée et les envoie à la sortie nommée **ArchiveOutput**. La deuxième requête effectue une agrégation et un filtrage simples, puis envoie les résultats vers un système d’alerte en aval appelé **AlertOutput**.

Notez que la clause **WITH** peut être utilisée pour définir plusieurs blocs de sous-requête. Cette option a l’avantage d’ouvrir moins de lecteurs sur la source d’entrée.

**Requête** :

```SQL
WITH ReaderQuery AS (
    SELECT
        *
    FROM
        Input TIMESTAMP BY Time
)

SELECT * INTO ArchiveOutput FROM ReaderQuery

SELECT 
    Make,
    System.TimeStamp() AS Time,
    COUNT(*) AS [Count] 
INTO AlertOutput 
FROM ReaderQuery
GROUP BY
    Make,
    TumblingWindow(second, 10)
HAVING [Count] >= 3
```

Pour plus d’informations, reportez-vous à [Clause **WITH**](/stream-analytics-query/with-azure-stream-analytics).

## <a name="simple-pass-through-query"></a>Requête directe simple

Une requête directe simple peut être utilisée pour copier les données d’un flux d’entrée dans la sortie. Par exemple, si un flux de données contenant des informations en temps réel sur un véhicule doit être enregistré dans une base de données SQL pour une analyse plus poussée, une requête directe simple effectue ce travail.

**Entrée** :

| Marque | Temps | Poids |
| --- | --- | --- |
| Make1 |2015-01-01T00:00:01.0000000Z |"1000" |
| Make1 |2015-01-01T00:00:02.0000000Z |"2000" |

**Sortie** :

| Marque | Temps | Poids |
| --- | --- | --- |
| Make1 |2015-01-01T00:00:01.0000000Z |"1000" |
| Make1 |2015-01-01T00:00:02.0000000Z |"2000" |

**Requête** :

```SQL
SELECT
    *
INTO Output
FROM Input
```

Une requête **SELECT** * projette tous les champs d’un événement entrant et les envoie à la sortie. De la même façon, **SELECT** peut également être utilisé pour projeter seulement les champs de l’entrée nécessaires. Dans cet exemple, si les champs *Make* et *Time* du véhicule sont les seuls champs nécessaires à enregistrer, ces champs peuvent être spécifiés dans l’instruction **SELECT**.

**Entrée** :

| Marque | Temps | Poids |
| --- | --- | --- |
| Make1 |2015-01-01T00:00:01.0000000Z |1 000 |
| Make1 |2015-01-01T00:00:02.0000000Z |2000 |
| Make2 |2015-01-01T00:00:04.0000000Z |1500 |

**Sortie** :

| Marque | Temps |
| --- | --- |
| Make1 |2015-01-01T00:00:01.0000000Z |
| Make1 |2015-01-01T00:00:02.0000000Z |
| Make2 |2015-01-01T00:00:04.0000000Z |

**Requête** :

```SQL
SELECT
    Make, Time
INTO Output
FROM Input
```

## <a name="string-matching-with-like-and-not-like"></a>Correspondance de chaînes avec LIKE et NOT LIKE

**LIKE** et **NOT LIKE** peuvent être utilisés pour vérifier si un champ correspond à un certain modèle. Par exemple, vous pouvez créer un filtre pour retourner seulement les plaques d’immatriculation qui commencent par la lettre « A » et se terminent par le chiffre 9.

**Entrée** :

| Marque | License_plate | Temps |
| --- | --- | --- |
| Make1 |ABC-123 |2015-01-01T00:00:01.0000000Z |
| Make2 |AAA-999 |2015-01-01T00:00:02.0000000Z |
| Make3 |ABC-369 |2015-01-01T00:00:03.0000000Z |

**Sortie** :

| Marque | License_plate | Temps |
| --- | --- | --- |
| Make2 |AAA-999 |2015-01-01T00:00:02.0000000Z |
| Make3 |ABC-369 |2015-01-01T00:00:03.0000000Z |

**Requête** :

```SQL
SELECT
    *
FROM
    Input TIMESTAMP BY Time
WHERE
    License_plate LIKE 'A%9'
```

Utilisez l’instruction **LIKE** pour vérifier la valeur du champ **License_plate**. Il doit commencer par la lettre « A », puis avoir une chaîne de zéro, un ou plusieurs caractères, et se terminer par le chiffre 9.

## <a name="calculation-over-past-events"></a>Calcul sur des événements antérieurs

La fonction **LAG** peut être utilisée pour examiner des événements antérieurs dans une fenêtre de temps et les comparer à l’événement actuel. Par exemple, la marque de voiture actuelle peut être placée dans la sortie si elle est différente de la dernière voiture qui a passé le péage.

**Entrée** :

| Marque | Temps |
| --- | --- |
| Make1 |2015-01-01T00:00:01.0000000Z |
| Make2 |2015-01-01T00:00:02.0000000Z |

**Sortie** :

| Marque | Temps |
| --- | --- |
| Make2 |2015-01-01T00:00:02.0000000Z |

**Requête** :

```SQL
SELECT
    Make,
    Time
FROM
    Input TIMESTAMP BY Time
WHERE
    LAG(Make, 1) OVER (LIMIT DURATION(minute, 1)) <> Make
```

Utilisez **LAG** pour regarder un événement précédent dans le flux d’entrée, en récupérant la valeur de *Make* et en la comparant à la valeur de *Make* de l’événement actuel, et pour placer l’événement dans la sortie.

Pour plus d’informations, reportez-vous à [**LAG**](/stream-analytics-query/lag-azure-stream-analytics).

## <a name="return-the-last-event-in-a-window"></a>Retourner le dernier événement dans une fenêtre

Les événements étant consommés par le système en temps réel, aucune fonction ne peut déterminer si un événement sera le dernier à arriver pour cette fenêtre de temps. Pour faire cela, le flux d’entrée doit être joint avec un autre quand l’heure d’un événement est l’heure maximale pour tous les événements de cette fenêtre.

**Entrée** :

| License_plate | Marque | Temps |
| --- | --- | --- |
| DXE 5291 |Make1 |2015-07-27T00:00:05.0000000Z |
| YZK 5704 |Make3 |2015-07-27T00:02:17.0000000Z |
| RMV 8282 |Make1 |2015-07-27T00:05:01.0000000Z |
| YHN 6970 |Make2 |2015-07-27T00:06:00.0000000Z |
| VFE 1616 |Make2 |2015-07-27T00:09:31.0000000Z |
| QYF 9358 |Make1 |2015-07-27T00:12:02.0000000Z |
| MDR 6128 |Make4 |2015-07-27T00:13:45.0000000Z |

**Sortie** :

| License_plate | Marque | Temps |
| --- | --- | --- |
| VFE 1616 |Make2 |2015-07-27T00:09:31.0000000Z |
| MDR 6128 |Make4 |2015-07-27T00:13:45.0000000Z |

**Requête** :

```SQL
WITH LastInWindow AS
(
    SELECT 
        MAX(Time) AS LastEventTime
    FROM 
        Input TIMESTAMP BY Time
    GROUP BY 
        TumblingWindow(minute, 10)
)

SELECT 
    Input.License_plate,
    Input.Make,
    Input.Time
FROM
    Input TIMESTAMP BY Time 
    INNER JOIN LastInWindow
    ON DATEDIFF(minute, Input, LastInWindow) BETWEEN 0 AND 10
    AND Input.Time = LastInWindow.LastEventTime
```

La première étape de la requête recherche l’horodatage maximal dans les fenêtres de 10 minutes, c’est-à-dire l’horodatage du dernier événement de cette fenêtre. La deuxième joint les résultats de la première requête avec le flux d’origine pour rechercher l’événement qui correspond aux derniers horodatages dans chaque fenêtre. 

**DATEDIFF** est une fonction spécifique aux dates qui compare et retourne la différence de temps entre deux champs DateTime. Pour plus d’informations, reportez-vous à [Fonctions de date](/stream-analytics-query/date-and-time-functions-azure-stream-analytics).

Pour plus d’informations sur la jointure de flux, reportez-vous à [**JOIN**](/stream-analytics-query/join-azure-stream-analytics).

## <a name="data-aggregation-over-time"></a>Agrégation de données dans le temps

Pour calculer des informations sur une fenêtre de temps, les données peuvent être agrégées ensemble. Dans cet exemple, un total est calculé sur les 10 dernières secondes pour chaque marque de voiture.

**Entrée** :

| Marque | Temps | Poids |
| --- | --- | --- |
| Make1 |2015-01-01T00:00:01.0000000Z |1 000 |
| Make1 |2015-01-01T00:00:02.0000000Z |2000 |
| Make2 |2015-01-01T00:00:04.0000000Z |1500 |

**Sortie** :

| Marque | Count |
| --- | --- |
| Make1 | 2 |
| Make2 | 1 |

**Requête** :

```SQL
SELECT
    Make,
    COUNT(*) AS Count
FROM
    Input TIMESTAMP BY Time
GROUP BY
    Make,
    TumblingWindow(second, 10)
```

Cette agrégation regroupe les voitures par *Make* (Marque) et les totalise toutes les 10 secondes. La sortie contient les champs *Make* (Marque) et *Count* (Total) des voitures qui sont passées au péage.

TumblingWindow est une fonction de fenêtrage utilisée pour regrouper des événements. Une agrégation peut être appliquée à tous les événements regroupés. Pour plus d’informations, consultez [Fonctions de fenêtrage](stream-analytics-window-functions.md).

Pour plus d’informations sur l’agrégation, reportez-vous aux[fonctions d’agrégation](/stream-analytics-query/aggregate-functions-azure-stream-analytics).

## <a name="periodically-output-values"></a>Valeurs périodiquement produites en sortie

En cas d’événements irréguliers ou manquants, une sortie à intervalle régulier peut être générée à partir d’une entrée de données plus éparses. Par exemple, générer toutes les cinq secondes un événement qui indique le point de données le plus récemment observé.

**Entrée** :

| Temps | Valeur |
| --- | --- |
| "2014-01-01T06:01:00" |1 |
| "2014-01-01T06:01:05" |2 |
| "2014-01-01T06:01:10" |3 |
| "2014-01-01T06:01:15" |4 |
| "2014-01-01T06:01:30" |5 |
| "2014-01-01T06:01:35" |6 |

**Sortie (10 premières lignes)** :

| Window_end | Last_event.Time | Last_event.Value |
| --- | --- | --- |
| 2014-01-01T14:01:00.000Z |2014-01-01T14:01:00.000Z |1 |
| 2014-01-01T14:01:05.000Z |2014-01-01T14:01:05.000Z |2 |
| 2014-01-01T14:01:10.000Z |2014-01-01T14:01:10.000Z |3 |
| 2014-01-01T14:01:15.000Z |2014-01-01T14:01:15.000Z |4 |
| 2014-01-01T14:01:20.000Z |2014-01-01T14:01:15.000Z |4 |
| 2014-01-01T14:01:25.000Z |2014-01-01T14:01:15.000Z |4 |
| 2014-01-01T14:01:30.000Z |2014-01-01T14:01:30.000Z |5 |
| 2014-01-01T14:01:35.000Z |2014-01-01T14:01:35.000Z |6 |
| 2014-01-01T14:01:40.000Z |2014-01-01T14:01:35.000Z |6 |
| 2014-01-01T14:01:45.000Z |2014-01-01T14:01:35.000Z |6 |

**Requête** :

```SQL
SELECT
    System.Timestamp() AS Window_end,
    TopOne() OVER (ORDER BY Time DESC) AS Last_event
FROM
    Input TIMESTAMP BY Time
GROUP BY
    HOPPINGWINDOW(second, 300, 5)
```

Cette requête génère des événements toutes les 5 secondes et produit le dernier événement précédemment reçu. La durée **HOPPINGWINDOW** détermine la période d’antériorité où la requête recherche l’événement le plus récent.

Pour plus d’informations, reportez-vous à [Fenêtre récurrente](/stream-analytics-query/hopping-window-azure-stream-analytics).

## <a name="correlate-events-in-a-stream"></a>Corréler des événements dans un flux

La corrélation d’événements dans le même flux peut être effectuée en examinant des événements antérieurs avec la fonction **LAG**. Par exemple, une sortie peut être générée chaque fois que deux voitures consécutives de la même *Make* (Marque) passent par le péage au cours des 90 dernières secondes.

**Entrée** :

| Marque | License_plate | Temps |
| --- | --- | --- |
| Make1 |ABC-123 |2015-01-01T00:00:01.0000000Z |
| Make1 |AAA-999 |2015-01-01T00:00:02.0000000Z |
| Make2 |DEF-987 |2015-01-01T00:00:03.0000000Z |
| Make1 |GHI-345 |2015-01-01T00:00:04.0000000Z |

**Sortie** :

| Marque | Temps | Current_car_license_plate | First_car_license_plate | First_car_time |
| --- | --- | --- | --- | --- |
| Make1 |2015-01-01T00:00:02.0000000Z |AAA-999 |ABC-123 |2015-01-01T00:00:01.0000000Z |

**Requête** :

```SQL
SELECT
    Make,
    Time,
    License_plate AS Current_car_license_plate,
    LAG(License_plate, 1) OVER (LIMIT DURATION(second, 90)) AS First_car_license_plate,
    LAG(Time, 1) OVER (LIMIT DURATION(second, 90)) AS First_car_time
FROM
    Input TIMESTAMP BY Time
WHERE
    LAG(Make, 1) OVER (LIMIT DURATION(second, 90)) = Make
```

La fonction **LAG** peut rechercher dans le flux d’entrée un événement antérieur et récupérer la valeur de *Make* , en comparant celle-ci à la valeur de *Make* de l’événement actuel.  Si la condition est remplie, les données de l’événement antérieur peuvent être projetées en utilisant **LAG** dans l’instruction **SELECT**.

Pour plus d’informations, reportez-vous à [LAG](/stream-analytics-query/lag-azure-stream-analytics).

## <a name="detect-the-duration-between-events"></a>Détecter la durée entre événements

La durée d’un événement peut être calculée en examinant le dernier événement Start (Début) après la réception d’un événement End (Fin). Cette requête peut être pratique pour déterminer le temps passé par un utilisateur sur une page ou une fonctionnalité.

**Entrée** :  

| Utilisateur | Fonctionnalité | Événement | Temps |
| --- | --- | --- | --- |
| user@location.com |RightMenu |Démarrer |2015-01-01T00:00:01.0000000Z |
| user@location.com |RightMenu |End |2015-01-01T00:00:08.0000000Z |

**Sortie** :  

| Utilisateur | Fonctionnalité | Duration |
| --- | --- | --- |
| user@location.com |RightMenu |7 |

**Requête** :

```SQL
SELECT
    [user],
    feature,
    DATEDIFF(
        second,
        LAST(Time) OVER (PARTITION BY [user], feature LIMIT DURATION(hour, 1) WHEN Event = 'start'),
        Time) as duration
FROM input TIMESTAMP BY Time
WHERE
    Event = 'end'
```

La fonction **LAST** peut être utilisée pour récupérer le dernier événement avec une condition spécifique. Dans cet exemple, la condition est un événement de type Start (Début), qui partitionne la recherche par utilisateur et par fonctionnalité avec **PARTITION BY**. De cette façon, chaque utilisateur et chaque fonctionnalité est traitée indépendamment lors de la recherche de l’événement Start (Début). **LIMIT DURATION** limite la recherche dans l’antériorité à 1 heure entre les événements Start (Début) et End (Fin).

## <a name="count-unique-values"></a>Compter des valeurs uniques

**COUNT** et **DISTINCT** peuvent être utilisés pour compter le nombre de valeurs de champ uniques qui apparaissent dans le flux au cours d’une fenêtre de temps. Vous pouvez créer une requête pour calculer le nombre de voitures d’une même marque *Make* ont franchi le péage dans une fenêtre de temps de 2 secondes.

**Entrée** :

| Marque | Temps |
| --- | --- |
| Make1 |2015-01-01T00:00:01.0000000Z |
| Make1 |2015-01-01T00:00:02.0000000Z |
| Make2 |2015-01-01T00:00:01.0000000Z |
| Make2 |2015-01-01T00:00:02.0000000Z |
| Make2 |2015-01-01T00:00:03.0000000Z |

**Output:**

| Count_make | Temps |
| --- | --- |
| 2 |2015-01-01T00:00:02.000Z |
| 1 |2015-01-01T00:00:04.000Z |

**Requête :**

```SQL
SELECT
     COUNT(DISTINCT Make) AS Count_make,
     System.TIMESTAMP() AS Time
FROM Input TIMESTAMP BY TIME
GROUP BY 
     TumblingWindow(second, 2)
```

**COUNT(DISTINCT Make)** retourne le nombre de valeurs distinctes de la colonne **Make** dans une fenêtre de temps.
Pour plus d’informations, reportez-vous à [Fonction d’agrégation **COUNT**](/stream-analytics-query/count-azure-stream-analytics).

## <a name="retrieve-the-first-event-in-a-window"></a>Récupérer le premier événement dans une fenêtre

**IsFirst** peut être utilisé pour récupérer le premier événement dans une fenêtre de temps. Par exemple, produire en sortie les informations de la première voiture à chaque intervalle de 10 minutes.

**Entrée** :

| License_plate | Marque | Temps |
| --- | --- | --- |
| DXE 5291 |Make1 |2015-07-27T00:00:05.0000000Z |
| YZK 5704 |Make3 |2015-07-27T00:02:17.0000000Z |
| RMV 8282 |Make1 |2015-07-27T00:05:01.0000000Z |
| YHN 6970 |Make2 |2015-07-27T00:06:00.0000000Z |
| VFE 1616 |Make2 |2015-07-27T00:09:31.0000000Z |
| QYF 9358 |Make1 |2015-07-27T00:12:02.0000000Z |
| MDR 6128 |Make4 |2015-07-27T00:13:45.0000000Z |

**Sortie** :

| License_plate | Marque | Temps |
| --- | --- | --- |
| DXE 5291 |Make1 |2015-07-27T00:00:05.0000000Z |
| QYF 9358 |Make1 |2015-07-27T00:12:02.0000000Z |

**Requête** :

```SQL
SELECT 
    License_plate,
    Make,
    Time
FROM 
    Input TIMESTAMP BY Time
WHERE 
    IsFirst(minute, 10) = 1
```

**IsFirst** peut également partitionner les données et calculer le premier événement sur chaque *Make* (Marque) de voiture spécifique trouvée à chaque intervalle de 10 minutes.

**Sortie** :

| License_plate | Marque | Temps |
| --- | --- | --- |
| DXE 5291 |Make1 |2015-07-27T00:00:05.0000000Z |
| YZK 5704 |Make3 |2015-07-27T00:02:17.0000000Z |
| YHN 6970 |Make2 |2015-07-27T00:06:00.0000000Z |
| QYF 9358 |Make1 |2015-07-27T00:12:02.0000000Z |
| MDR 6128 |Make4 |2015-07-27T00:13:45.0000000Z |

**Requête** :

```SQL
SELECT 
    License_plate,
    Make,
    Time
FROM 
    Input TIMESTAMP BY Time
WHERE 
    IsFirst(minute, 10) OVER (PARTITION BY Make) = 1
```

Pour plus d’informations, reportez-vous à [**IsFirst**](/stream-analytics-query/isfirst-azure-stream-analytics).

## <a name="remove-duplicate-events-in-a-window"></a>Supprimer des événements en double dans une fenêtre

Quand vous effectuez une opération comme calculer des moyennes de plusieurs événements dans une fenêtre de temps donnée, les événements en double doivent être filtrés. Dans l’exemple suivant, le deuxième événement est un doublon du premier.

**Entrée** :  

| deviceId | Temps | Attribut | Valeur |
| --- | --- | --- | --- |
| 1 |2018-07-27T00:00:01.0000000Z |Température |50 |
| 1 |2018-07-27T00:00:01.0000000Z |Température |50 |
| 2 |2018-07-27T00:00:01.0000000Z |Température |40 |
| 1 |2018-07-27T00:00:05.0000000Z |Température |60 |
| 2 |2018-07-27T00:00:05.0000000Z |Température |50 |
| 1 |2018-07-27T00:00:10.0000000Z |Température |100 |

**Sortie** :  

| AverageValue | deviceId |
| --- | --- |
| 70 | 1 |
|45 | 2 |

**Requête** :

```SQL
With Temp AS (
SELECT
    COUNT(DISTINCT Time) AS CountTime,
    Value,
    DeviceId
FROM
    Input TIMESTAMP BY Time
GROUP BY
    Value,
    DeviceId,
    SYSTEM.TIMESTAMP()
)

SELECT
    AVG(Value) AS AverageValue, DeviceId
INTO Output
FROM Temp
GROUP BY DeviceId,TumblingWindow(minute, 5)
```

**COUNT(DISTINCT Time)** retourne le nombre de valeurs distinctes de la colonne Time dans une fenêtre de temps. La sortie de la première étape peut ensuite être utilisée pour calculer la moyenne par appareil, en ignorant les doublons.

Pour plus d’informations, reportez-vous à [COUNT(DISTINCT Time)](/stream-analytics-query/count-azure-stream-analytics).

## <a name="specify-logic-for-different-casesvalues-case-statements"></a>Spécifier la logique de différentes casses/valeurs (instructions CASE)

Les instructions **CASE** peuvent fournir des calculs différents pour différents champs, en fonction d’un critère particulier. Par exemple, affectez la voie « A » aux voitures *Make1* et la voie « B » aux autres marques.

**Entrée** :

| Marque | Temps |
| --- | --- |
| Make1 |2015-01-01T00:00:01.0000000Z |
| Make2 |2015-01-01T00:00:02.0000000Z |
| Make2 |2015-01-01T00:00:03.0000000Z |

**Sortie** :

| Marque |Dispatch_to_lane | Temps |
| --- | --- | --- |
| Make1 |« A » |2015-01-01T00:00:01.0000000Z |
| Make2 |« B » |2015-01-01T00:00:02.0000000Z |

**Solution** :

```SQL
SELECT
    Make
    CASE
        WHEN Make = "Make1" THEN "A"
        ELSE "B"
    END AS Dispatch_to_lane,
    System.TimeStamp() AS Time
FROM
    Input TIMESTAMP BY Time
```

L’expression **CASE** compare une expression à un ensemble d’expressions simples pour déterminer son résultat. Dans cet exemple, les véhicules *Make1* sont affectés à la voie « A », tandis que les véhicules des autres marques sont affectés à la voie « B ».

Pour plus d’informations, reportez-vous à [Expression case](/stream-analytics-query/case-azure-stream-analytics).

## <a name="data-conversion"></a>Conversion de données

Les données peuvent être converties en temps réel avec la méthode **CAST**. Par exemple, le poids d’une voiture peut être converti du type **nvarchar(max)** en type **bigint** et être utilisé sur un calcul numérique.

**Entrée** :

| Marque | Temps | Poids |
| --- | --- | --- |
| Make1 |2015-01-01T00:00:01.0000000Z |"1000" |
| Make1 |2015-01-01T00:00:02.0000000Z |"2000" |

**Sortie** :

| Marque | Poids |
| --- | --- |
| Make1 |3000 |

**Requête** :

```SQL
SELECT
    Make,
    SUM(CAST(Weight AS BIGINT)) AS Weight
FROM
    Input TIMESTAMP BY Time
GROUP BY
    Make,
    TumblingWindow(second, 10)
```

Utilisez une instruction **CAST** pour spécifier son type de données. Consultez la liste des types de données pris en charge dans [Types de données (Azure Stream Analytics)](/stream-analytics-query/data-types-azure-stream-analytics).

Pour plus d’informations, consultez [Fonctions de conversion des données](/stream-analytics-query/conversion-functions-azure-stream-analytics).

## <a name="detect-the-duration-of-a-condition"></a>Détecter la durée d’une condition

Pour les conditions qui s’étendent sur plusieurs événements, la fonction **LAG** peut être utilisée pour identifier la durée de cette condition. Par exemple, supposons qu’à la suite d’un bogue, le poids de toutes les voitures soit incorrect (supérieur 20 000 livres) et que la durée de ce bogue doive être calculée.

**Entrée** :

| Marque | Temps | Poids |
| --- | --- | --- |
| Make1 |2015-01-01T00:00:01.0000000Z |2000 |
| Make2 |2015-01-01T00:00:02.0000000Z |25000 |
| Make1 |2015-01-01T00:00:03.0000000Z |26000 |
| Make2 |2015-01-01T00:00:04.0000000Z |25000 |
| Make1 |2015-01-01T00:00:05.0000000Z |26000 |
| Make2 |2015-01-01T00:00:06.0000000Z |25000 |
| Make1 |2015-01-01T00:00:07.0000000Z |26000 |
| Make2 |2015-01-01T00:00:08.0000000Z |2000 |

**Sortie** :

| Start_fault | End_fault |
| --- | --- |
| 2015-01-01T00:00:02.000Z |2015-01-01T00:00:07.000Z |

**Requête** :

```SQL
WITH SelectPreviousEvent AS
(
SELECT
    *,
    LAG([time]) OVER (LIMIT DURATION(hour, 24)) as previous_time,
    LAG([weight]) OVER (LIMIT DURATION(hour, 24)) as previous_weight
FROM input TIMESTAMP BY [time]
)

SELECT 
    LAG(time) OVER (LIMIT DURATION(hour, 24) WHEN previous_weight < 20000 ) [Start_fault],
    previous_time [End_fault]
FROM SelectPreviousEvent
WHERE
    [weight] < 20000
    AND previous_weight > 20000
```
La première instruction **SELECT** met en corrélation la mesure de poids actuelle avec la mesure précédente, en la projetant avec la mesure actuelle. Le deuxième **SELECT** regarde le dernier événement où le *previous_weight* est inférieur à 20 000, où le poids actuel est inférieur à 20 000 et où le *previous_weight* de l’événement actuel était supérieur à 20 000.

End_fault est l’événement de non-défaillance actuel où l’événement précédent était une défaillance, et Start_fault est le dernier événement de non-défaillance avant cela.

## <a name="process-events-with-independent-time-substreams"></a>Traiter les événements avec une heure indépendante (sous-flux)

Les événements peuvent arriver en retard ou dans le désordre en raison des décalages des horloges entre les producteurs d’événements, des décalages des horloges entre les partitions ou de la latence du réseau.
Dans l’exemple suivant, l’horloge de l’appareil pour *TollID*  2 a cinq secondes de retard par rapport à *TollID*  1, et l’horloge de l’appareil pour *TollID*  3 a dix secondes de retard par rapport à *TollID*  1. Un calcul peut être effectué de façon indépendante pour chaque péage, en prenant seulement en compte les données de sa propre horloge comme horodatage.

**Entrée** :

| LicensePlate | Marque | Temps | TollID |
| --- | --- | --- | --- |
| DXE 5291 |Make1 |2015-07-27T00:00:01.0000000Z | 1 |
| YHN 6970 |Make2 |2015-07-27T00:00:05.0000000Z | 1 |
| QYF 9358 |Make1 |2015-07-27T00:00:01.0000000Z | 2 |
| GXF 9462 |Make3 |2015-07-27T00:00:04.0000000Z | 2 |
| VFE 1616 |Make2 |2015-07-27T00:00:10.0000000Z | 1 |
| RMV 8282 |Make1 |2015-07-27T00:00:03.0000000Z | 3 |
| MDR 6128 |Make3 |2015-07-27T00:00:11.0000000Z | 2 |
| YZK 5704 |Make4 |2015-07-27T00:00:07.0000000Z | 3 |

**Sortie** :

| TollID | Count |
| --- | --- |
| 1 | 2 |
| 2 | 2 |
| 1 | 1 |
| 3 | 1 |
| 2 | 1 |
| 3 | 1 |

**Requête** :

```SQL
SELECT
      TollId,
      COUNT(*) AS Count
FROM input
      TIMESTAMP BY Time OVER TollId
GROUP BY TUMBLINGWINDOW(second, 5), TollId
```

La clause **TIMESTAMP OVER BY** examine séparément la chronologie de chaque appareil en utilisant des sous-flux. L’événement de sortie pour chaque *TollID* est généré tels qu’il est calculé, ce qui signifie que les événements sont dans l’ordre relativement à chaque *TollID* , au lieu d’être réorganisés comme si tous les appareils avaient la même horloge.

Pour plus d’informations, reportez-vous à [TIMESTAMP BY OVER](/stream-analytics-query/timestamp-by-azure-stream-analytics#over-clause-interacts-with-event-ordering).

## <a name="session-windows"></a>Fenêtres de session

Une fenêtre de session est une fenêtre qui s’étend au fur et à mesure que des événements se produisent, et se ferme pour le calcul si aucun événement n’est reçu après un laps de temps spécifique ou si la fenêtre atteint sa durée maximale.
Cette fenêtre est particulièrement pratique lors du calcul de données d’interaction de l’utilisateur. Une fenêtre commence quand un utilisateur commence à interagir avec le système et se ferme quand aucun autre événement n’est observé, ce qui signifie que l’utilisateur a cessé d’interagir.
Par exemple, un utilisateur interagit avec une page web dans laquelle le nombre de clics est journalisé : une fenêtre de session peut être utilisée pour déterminer la durée de l’interaction de l’utilisateur avec le site.

**Entrée** :

| User_id | Temps | URL |
| --- | --- | --- |
| 0 | 2017-01-26T00:00:00.0000000Z | "www.example.com/a.html" |
| 0 | 2017-01-26T00:00:20.0000000Z | "www.example.com/b.html" |
| 1 | 2017-01-26T00:00:55.0000000Z | "www.example.com/c.html" |
| 0 | 2017-01-26T00:01:10.0000000Z | "www.example.com/d.html" |
| 1 | 2017-01-26T00:01:15.0000000Z | "www.example.com/e.html" |

**Sortie** :

| User_id | StartTime | EndTime | Duration_in_seconds |
| --- | --- | --- | --- |
| 0 | 2017-01-26T00:00:00.0000000Z | 2017-01-26T00:01:10.0000000Z | 70 |
| 1 | 2017-01-26T00:00:55.0000000Z | 2017-01-26T00:01:15.0000000Z | 20 |

**Requête** :

``` SQL
SELECT
    user_id,
    MIN(time) as StartTime,
    MAX(time) as EndTime,
    DATEDIFF(second, MIN(time), MAX(time)) AS duration_in_seconds
FROM input TIMESTAMP BY time
GROUP BY
    user_id,
    SessionWindow(minute, 1, 60) OVER (PARTITION BY user_id)
```

Le **SELECT** projette les données relatives à l’interaction de l’utilisateur, ainsi que la durée de l’interaction. Regroupement des données par utilisateur et une **SessionWindow** qui se ferme si aucune interaction ne se produit pendant 1 minute, avec une taille de fenêtre maximale de 60 minutes.

Pour plus d’informations sur **SessionWindow** , reportez-vous à [Fenêtre de session](/stream-analytics-query/session-window-azure-stream-analytics).

## <a name="language-extensibility-with-user-defined-function-in-javascript-and-c"></a>Extensibilité du langage avec une fonction définie par l’utilisateur en JavaScript et en C#

Le langage de requête Azure Stream Analytics peut être étendu avec des fonctions personnalisées écrites en langages JavaScript ou C#. Les fonctions définies par l’utilisateur (UDF) sont des calculs personnalisés/complexes qui ne peuvent pas être exprimés facilement avec le langage **SQL**. Ces fonctions définies par l’utilisateur peuvent être définies une seule fois et utilisées plusieurs fois dans une requête. Par exemple, une fonction définie par l’utilisateur peut être utilisée pour convertir une valeur hexadécimale *nvarchar(max)* en une valeur *bigint*.

**Entrée** :

| Device_id | HexValue |
| --- | --- |
| 1 | "B4" |
| 2 | "11B" |
| 3 | "121" |

**Sortie** :

| Device_id | Decimal |
| --- | --- |
| 1 | 180 |
| 2 | 283 |
| 3 | 289 |

```JavaScript
function hex2Int(hexValue){
    return parseInt(hexValue, 16);
}
```

```C#
public static class MyUdfClass {
    public static long Hex2Int(string hexValue){
        return int.Parse(hexValue, System.Globalization.NumberStyles.HexNumber);
    }
}
```

```SQL
SELECT
    Device_id,
    udf.Hex2Int(HexValue) AS Decimal
From
    Input
```

La fonction définie par l’utilisateur calcule la valeur *bigint* à partir de la valeur en hexadécimal (HexValue) sur chaque événement consommé.

Pour plus d’informations, reportez-vous à [JavaScript](./stream-analytics-javascript-user-defined-functions.md) et [C#](./stream-analytics-edge-csharp-udf.md).

## <a name="advanced-pattern-matching-with-match_recognize"></a>Correspondance de modèle avancé avec MATCH_RECOGNIZE

**MATCH_RECOGNIZE** est un mécanisme de correspondance de modèle avancé qui peut être utilisé pour faire correspondre une séquence d’événements à un modèle d’expression régulière bien défini.
Par exemple, la défaillance éventuelle d’un ATM est surveillée en temps réel : pendant le fonctionnement de l’ATM, s’il y a deux messages d’avertissement consécutifs, l’administrateur doit être averti.

**Entrée** :

| ATM_id | Operation_id | Return_Code | Temps |
| --- | --- | --- | --- |
| 1 | « Entering Pin » (Entrée du code PIN) | « Success » | 2017-01-26T00:10:00.0000000Z |
| 2 | « Opening Money Slot » (Ouverture du dispositif de distribution des billets) | « Success » | 2017-01-26T00:10:07.0000000Z |
| 2 | « Closing Money Slot » (Fermeture du dispositif de distribution des billets) | « Success » | 2017-01-26T00:10:11.0000000Z |
| 1 | « Entering Withdraw Quantity » (Entrée de la quantité à retirer) | « Success » | 2017-01-26T00:10:08.0000000Z |
| 1 | « Opening Money Slot » (Ouverture du dispositif de distribution des billets) | « Warning » (Avertissement) | 2017-01-26T00:10:14.0000000Z |
| 1 | « Printing Bank Balance » (Impression du solde du compte) | « Warning » (Avertissement) | 2017-01-26T00:10:19.0000000Z |

**Sortie** :

| ATM_id | First_Warning_Operation_id | Warning_Time |
| --- | --- | --- |
| 1 | « Opening Money Slot » (Ouverture du dispositif de distribution des billets) | 2017-01-26T00:10:14.0000000Z |

```SQL
SELECT *
FROM input TIMESTAMP BY time OVER ATM_id
MATCH_RECOGNIZE (
    LIMIT DURATION(minute, 1)
    PARTITON BY ATM_id
    MEASURES
        First(Warning.ATM_id) AS ATM_id,
        First(Warning.Operation_Id) AS First_Warning_Operation_id,
        First(Warning.Time) AS Warning_Time
    AFTER MATCH SKIP TO NEXT ROW
    PATTERN (Success* Warning{2,})
    DEFINE
        Success AS Succes.Return_Code = 'Success',
        Failure AS Warning.Return_Code <> 'Success'
) AS patternMatch
```

Cette requête correspond à au moins deux événements de défaillance consécutifs et génère une alarme quand les conditions sont remplies.
**PATTERN** définit l’expression régulière à utiliser sur la correspondance ; dans le cas présent, un nombre quelconque d’opérations réussies suivies d’au moins deux défaillances consécutives.
La réussite et l’échec sont définis avec la valeur de Return_Code et, une fois que la condition est remplie, les **MEASURES** (Mesures) sont projetées avec *ATM_id* , la première opération avec avertissement et l’heure du premier avertissement.

Pour plus d’informations, reportez-vous à [MATCH_RECOGNIZE](/stream-analytics-query/match-recognize-stream-analytics).

## <a name="geofencing-and-geospatial-queries"></a>Requêtes géospatiales et de geofencing

Azure Stream Analytics fournit des fonctions géospatiales intégrées qui peuvent être utilisées pour implémenter des scénarios, tels que la gestion de flottes, le covoiturage, les voitures connectées et le suivi de ressources.
Les données géospatiales peuvent être ingérées au format GeoJSON ou WKT en tant que données de référence ou de flux d’événements.
Par exemple, une entreprise spécialisée dans la fabrication de machines pour l’impression de passeports loue ses machines aux gouvernements et aux consulats. L’emplacement de ces machines est étroitement contrôlé de façon à en éviter un placement erroné et une possible utilisation pour la contrefaçon de passeports. Chaque machine est équipée d’un dispositif de suivi GPS. Ces informations sont relayées vers un travail Azure Stream Analytics.
La manufacture souhaite effectuer le suivi de l’emplacement de ces machines et être alertée si une d’entre elles quitte une zone autorisée : de cette façon, elle peut la désactiver à distance, alerter les autorités et récupérer l’équipement.

**Entrée** :

| Equipment_id | Equipment_current_location | Temps |
| --- | --- | --- |
| 1 | « POINT(-122.13288797982818 47.64082002051315) » | 2017-01-26T00:10:00.0000000Z |
| 1 | « POINT(-122.13307252987875 47.64081350934929) » | 2017-01-26T00:11:00.0000000Z |
| 1 | « POINT(-122.13308862313283 47.6406508603241) » | 2017-01-26T00:12:00.0000000Z |
| 1 | « POINT(-122.13341048821462 47.64043760861279) » | 2017-01-26T00:13:00.0000000Z |

**Entrée de données de référence**  :

| Equipment_id | Equipment_lease_location |
| --- | --- |
| 1 | « POLYGON((-122.13326028450979 47.6409833866794,-122.13261655434621 47.6409833866794,-122.13261655434621 47.64061471602751,-122.13326028450979 47.64061471602751,-122.13326028450979 47.6409833866794)) » |

**Sortie** :

| Equipment_id | Equipment_alert_location | Temps |
| --- | --- | --- |
| 1 | « POINT(-122.13341048821462 47.64043760861279) » | 2017-01-26T00:13:00.0000000Z |

```SQL
SELECT
    input.Equipment_id AS Equipment_id,
    input.Equipment_current_location AS Equipment_current_location,
    input.Time AS Time
FROM input TIMESTAMP BY time
JOIN
    referenceInput 
    ON input.Equipment_id = referenceInput.Equipment_id
    WHERE 
        ST_WITHIN(input.Equipment_currenct_location, referenceInput.Equipment_lease_location) = 1
```

La requête permet au fabricant de surveiller automatiquement l’emplacement des machines, en recevant des alertes quand une machine quitte la limite géographique autorisée. La fonction géospatiale intégrée permet aux utilisateurs d’utiliser des données GPS dans la requête sans recourir à des bibliothèques tierces.

Pour plus d’informations, reportez-vous à l’article [Scénarios de geofencing et d’agrégation géospatiale avec Azure Stream Analytics](geospatial-scenarios.md).

## <a name="get-help"></a>Obtenir de l’aide

Pour obtenir de l’aide supplémentaire, essayez notre [page de questions Microsoft Q&A pour Azure Stream Analytics](/answers/topics/azure-stream-analytics.html).

## <a name="next-steps"></a>Étapes suivantes
* [Présentation d’Azure Stream Analytics](stream-analytics-introduction.md)
* [Prise en main d'Azure Stream Analytics](stream-analytics-real-time-fraud-detection.md)
* [Mise à l’échelle des travaux Azure Stream Analytics](stream-analytics-scale-jobs.md)
* [Références sur le langage des requêtes d'Azure Stream Analytics](/stream-analytics-query/stream-analytics-query-language-reference)
* [Références sur l’API REST de gestion d’Azure Stream Analytics](/rest/api/streamanalytics/)