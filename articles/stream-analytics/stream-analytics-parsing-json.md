---
title: Analyse de données JSON et d’AVRO dans Stream Analytics
description: Cet article explique comment utiliser des types de données complexes tels que des tableaux, et des données au format JSON ou CSV.
ms.service: stream-analytics
author: mamccrea
ms.author: mamccrea
ms.topic: conceptual
ms.date: 01/29/2020
ms.custom: devx-track-js
ms.openlocfilehash: 6c2eb4225cb014b3251d12470e4e9827150a5cf2
ms.sourcegitcommit: 857859267e0820d0c555f5438dc415fc861d9a6b
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93123351"
---
# <a name="parse-json-and-avro-data-in-azure-stream-analytics"></a>Analyser des données JSON et Avro dans Azure Stream Analytics

Azure Stream Analytics prend en charge le traitement d’événements dans les formats de données CSV, JSON et Avro. Des données JSON et Avro peuvent être structurées et contenir certains types complexes, tels que des tableaux et objets (enregistrements) imbriqués. 

>[!NOTE]
>Les fichiers AVRO créés par la capture Event Hub répondent à un format spécifique qui implique que vous utilisiez la fonctionnalité de *désérialiseur personnalisé*. Pour plus d'informations, consultez [Lire les entrées dans n’importe quel format avec des désérialiseurs .NET personnalisés](./custom-deserializer-examples.md).
>
>La désérialisation AVRO de Stream Analytics ne prend pas en charge le type Mappage. Stream Analytics ne peut pas lire les objets blob de capture EventHub, car celle-ci utilise un mappage.


## <a name="record-data-types"></a>Données de type enregistrement
Des données de type enregistrement sont utilisées pour représenter des tableaux JSON et Avro quand des formats correspondants sont utilisés dans les flux de données en entrée. Ces exemples montrent un capteur qui lit les événements en entrée au format JSON. Voici un exemple d’événement unique :

```json
{
    "DeviceId" : "12345",
    "Location" :
    {
        "Lat": 47,
        "Long": 122
    },
    "SensorReadings" :
    {
        "Temperature" : 80,
        "Humidity" : 70,
        "CustomSensor01" : 5,
        "CustomSensor02" : 99,
        "SensorMetadata" : 
        {
        "Manufacturer":"ABC",
        "Version":"1.2.45"
        }
    }
}
```

### <a name="access-nested-fields-in-known-schema"></a>Accéder aux champs imbriqués dans un schéma connu
Utilisez la notation par points (.) pour accéder facilement aux champs imbriqués, directement à partir de votre requête. Par exemple, cette requête sélectionne les coordonnées de latitude et de longitude sous la propriété Location dans les données de l’extrait JSON précédent. La notation par points peut servir à parcourir plusieurs niveaux, comme indiqué ci-dessous.

```SQL
SELECT
    DeviceID,
    Location.Lat,
    Location.Long,
    SensorReadings.Temperature,
    SensorReadings.SensorMetadata.Version
FROM input
```

Le résultat est le suivant :

|DeviceID|Lat|Long|Température|Version|
|-|-|-|-|-|
|12345|47|122|80|1.2.45|


### <a name="select-all-properties"></a>Sélectionner toutes les propriétés
Vous pouvez sélectionner toutes les propriétés d’un enregistrement imbriqué à l’aide du caractère générique «  * ». Prenons l’exemple suivant :

```SQL
SELECT
    DeviceID,
    Location.*
FROM input
```

Le résultat est le suivant :

|DeviceID|Lat|Long|
|-|-|-|
|12345|47|122|


### <a name="access-nested-fields-when-property-name-is-a-variable"></a>Accéder aux champs imbriqués lorsque le nom de la propriété est une variable

Utilisez la fonction [GetRecordPropertyValue](/stream-analytics-query/getrecordpropertyvalue-azure-stream-analytics) si le nom de la propriété est une variable. Elle permet de créer des requêtes dynamiques sans codage en dur des noms de propriété.

Par exemple, imaginez que l'exemple de flux de données doive **être joint à des données de référence** contenant des seuils pour chaque capteur d’appareil. Un extrait de ce type de données de référence est reproduit ci-dessous.

```json
{
    "DeviceId" : "12345",
    "SensorName" : "Temperature",
    "Value" : 85
},
{
    "DeviceId" : "12345",
    "SensorName" : "Humidity",
    "Value" : 65
}
```

L’objectif consiste ici à joindre l'exemple de jeu de données du début de l'article à ces données de référence, et à générer un événement pour chaque mesure de capteur au-dessus de son seuil. Ainsi, l'événement unique ci-dessus peut générer plusieurs événements de sortie si différents capteurs se trouvent au-dessus de leurs seuils respectifs et ce, grâce à la jointure. Pour obtenir des résultats similaires sans jointure, consultez la section ci-dessous.

```SQL
SELECT
    input.DeviceID,
    thresholds.SensorName,
    "Alert : Sensor above threshold" AS AlertMessage
FROM input      -- stream input
JOIN thresholds -- reference data input
ON
    input.DeviceId = thresholds.DeviceId
WHERE
    GetRecordPropertyValue(input.SensorReadings, thresholds.SensorName) > thresholds.Value
```

**GetRecordPropertyValue** sélectionne la propriété dans *SensorReadings* , et dont le nom correspond au nom de propriété provenant des données de référence. La valeur associée de *SensorReadings* est ensuite extraite.

Le résultat est le suivant :

|DeviceID|SensorName|AlertMessage|
|-|-|-|
|12345|Humidité|Alerte : capteur au-dessus du seuil|

### <a name="convert-record-fields-into-separate-events"></a>Convertir les champs d’enregistrement en événements distincts

Pour convertir les champs d’enregistrement en événements distincts, utilisez l’opérateur [APPLIQUER](/stream-analytics-query/apply-azure-stream-analytics) avec la fonction [GetRecordProperties](/stream-analytics-query/getrecordproperties-azure-stream-analytics).

Avec les exemples de données d’origine, la requête suivante peut être utilisée pour extraire des propriétés dans différents événements.

```SQL
SELECT
    event.DeviceID,
    sensorReading.PropertyName,
    sensorReading.PropertyValue
FROM input as event
CROSS APPLY GetRecordProperties(event.SensorReadings) AS sensorReading
```

Le résultat est le suivant :

|DeviceID|SensorName|AlertMessage|
|-|-|-|
|12345|Température|80|
|12345|Humidité|70|
|12345|CustomSensor01|5|
|12345|CustomSensor02|99|
|12345|SensorMetadata|[object Object]|

[WITH](/stream-analytics-query/with-azure-stream-analytics) permet de router ces événements vers différentes destinations :

```SQL
WITH Stage0 AS
(
    SELECT
        event.DeviceID,
        sensorReading.PropertyName,
        sensorReading.PropertyValue
    FROM input as event
    CROSS APPLY GetRecordProperties(event.SensorReadings) AS sensorReading
)

SELECT DeviceID, PropertyValue AS Temperature INTO TemperatureOutput FROM Stage0 WHERE PropertyName = 'Temperature'
SELECT DeviceID, PropertyValue AS Humidity INTO HumidityOutput FROM Stage0 WHERE PropertyName = 'Humidity'
```

### <a name="parse-json-record-in-sql-reference-data"></a>Analyser un enregistrement JSON dans des données de référence SQL
Lorsque vous utilisez Azure SQL Database comme données de référence dans votre travail, il est possible d’avoir une colonne qui contient des données au format JSON. Voici un exemple.

|DeviceID|Données|
|-|-|
|12345|{"key" : "value1"}|
|54321|{"key" : "value2"}|

Vous pouvez analyser l’enregistrement JSON dans la colonne *Données* en écrivant une simple fonction JavaScript définie par l’utilisateur.

```javascript
function parseJson(string) {
return JSON.parse(string);
}
```

Vous pouvez ensuite créer une étape dans votre requête Stream Analytics comme indiqué ci-dessous pour accéder aux champs de vos enregistrements JSON.

 ```SQL
 WITH parseJson as
 (
 SELECT DeviceID, udf.parseJson(sqlRefInput.Data) as metadata,
 FROM sqlRefInput
 )
 
 SELECT metadata.key
 INTO output
 FROM streamInput
 JOIN parseJson 
 ON streamInput.DeviceID = parseJson.DeviceID
```

## <a name="array-data-types"></a>Données de type tableau

Les données de type tableau sont des collections ordonnées de valeurs. Certaines opérations courantes sur des valeurs de tableau sont décrites ci-dessous. Ces exemples utilisent les fonctions [GetArrayElement](/stream-analytics-query/getarrayelement-azure-stream-analytics), [GetArrayElements](/stream-analytics-query/getarrayelements-azure-stream-analytics) et [GetArrayLength](/stream-analytics-query/getarraylength-azure-stream-analytics), ainsi que l’opérateur [APPLY](/stream-analytics-query/apply-azure-stream-analytics).

Voici un exemple d’événement unique. `CustomSensor03` et `SensorMetadata` correspondent à un tableau  **de type**  :

```json
{
    "DeviceId" : "12345",
    "SensorReadings" :
    {
        "Temperature" : 80,
        "Humidity" : 70,
        "CustomSensor01" : 5,
        "CustomSensor02" : 99,
        "CustomSensor03": [12,-5,0]
     },
    "SensorMetadata":[
        {          
            "smKey":"Manufacturer",
            "smValue":"ABC"                
        },
        {
            "smKey":"Version",
            "smValue":"1.2.45"
        }
    ]
}
```

### <a name="working-with-a-specific-array-element"></a>Utilisation d’un élément de tableau spécifique

Sélectionnez un élément de tableau à un index spécifié (en sélectionnant le premier élément du tableau) :

```SQL
SELECT
    GetArrayElement(SensorReadings.CustomSensor03, 0) AS firstElement
FROM input
```

Le résultat est le suivant :

|firstElement|
|-|
|12|

### <a name="select-array-length"></a>Sélection d’une longueur de tableau

```SQL
SELECT
    GetArrayLength(SensorReadings.CustomSensor03) AS arrayLength
FROM input
```

Le résultat est le suivant :

|arrayLength|
|-|
|3|

### <a name="convert-array-elements-into-separate-events"></a>Conversion des éléments de tableau en événements distincts

Sélectionnez tous les éléments du tableau en tant qu’événements individuels. L’opérateur [APPLY](/stream-analytics-query/apply-azure-stream-analytics) avec la fonction intégrée [GetArrayElements](/stream-analytics-query/getarrayelements-azure-stream-analytics) extrait tous les éléments du tableau en tant qu’événements individuels :

```SQL
SELECT
    DeviceId,
    CustomSensor03Record.ArrayIndex,
    CustomSensor03Record.ArrayValue
FROM input
CROSS APPLY GetArrayElements(SensorReadings.CustomSensor03) AS CustomSensor03Record

```

Le résultat est le suivant :

|deviceId|ArrayIndex|ArrayValue|
|-|-|-|
|12345|0|12|
|12345|1|-5|
|12345|2|0|

```SQL
SELECT   
    i.DeviceId, 
    SensorMetadataRecords.ArrayValue.smKey as smKey,
    SensorMetadataRecords.ArrayValue.smValue as smValue
FROM input i
CROSS APPLY GetArrayElements(SensorMetadata) AS SensorMetadataRecords
 ```
 
Le résultat est le suivant :

|deviceId|smKey|smValue|
|-|-|-|
|12345|Fabricant|ABC|
|12345|Version|1.2.45|

Si les champs extraits doivent apparaître dans des colonnes, il est possible de faire pivoter le jeu de données à l’aide de la syntaxe [WITH](/stream-analytics-query/with-azure-stream-analytics), en plus de l’opération [JOIN](/stream-analytics-query/join-azure-stream-analytics). Cette jointure requiert une condition de [limite d'heure](/stream-analytics-query/join-azure-stream-analytics#BKMK_DateDiff) qui évite la duplication :

```SQL
WITH DynamicCTE AS (
    SELECT   
        i.DeviceId,
        SensorMetadataRecords.ArrayValue.smKey as smKey,
        SensorMetadataRecords.ArrayValue.smValue as smValue
    FROM input i
    CROSS APPLY GetArrayElements(SensorMetadata) AS SensorMetadataRecords 
)

SELECT
    i.DeviceId,
    i.Location.*,
    V.smValue AS 'smVersion',
    M.smValue AS 'smManufacturer'
FROM input i
LEFT JOIN DynamicCTE V ON V.smKey = 'Version' and V.DeviceId = i.DeviceId AND DATEDIFF(minute,i,V) BETWEEN 0 AND 0 
LEFT JOIN DynamicCTE M ON M.smKey = 'Manufacturer' and M.DeviceId = i.DeviceId AND DATEDIFF(minute,i,M) BETWEEN 0 AND 0
```

Le résultat est le suivant :

|deviceId|Lat|Long|smVersion|smManufacturer|
|-|-|-|-|-|
|12345|47|122|1.2.45|ABC|

## <a name="see-also"></a>Voir aussi
[Types de données dans Azure Stream Analytics](/stream-analytics-query/data-types-azure-stream-analytics)