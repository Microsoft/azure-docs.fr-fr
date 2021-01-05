---
title: 'Concepts : Modèles de mappage dans le connecteur Azure IoT pour la fonctionnalité FHIR (préversion) de l’API Azure pour FHIR'
description: Apprenez à créer deux types de modèles de mappage dans le connecteur Azure IoT pour FHIR (préversion). Le modèle de mappage d’appareil transforme les données de l’appareil en schéma normalisé. Le modèle de mappage FHIR transforme un message normalisé en ressource d’observation basée sur FHIR.
services: healthcare-apis
author: ms-puneet-nagpal
ms.service: healthcare-apis
ms.subservice: iomt
ms.topic: conceptual
ms.date: 08/03/2020
ms.author: punagpal
ms.openlocfilehash: f348a8d8755402d6426f19eabc432f54e3fb8e42
ms.sourcegitcommit: 8e7316bd4c4991de62ea485adca30065e5b86c67
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/17/2020
ms.locfileid: "94659656"
---
# <a name="azure-iot-connector-for-fhir-preview-mapping-templates"></a>Modèles de mappage du connecteur Azure IoT pour FHIR (préversion)
Cet article explique comment configurer le Connecteur Azure IoT pour Fast Healthcare Interoperability Resources (FHIR&#174;)* à l’aide de modèles de mappage.

Le connecteur Azure IoT pour FHIR requiert deux types de modèles de mappage basés sur JSON. Le premier type, le **mappage d’appareils**, est chargé de mapper les charges utiles d’appareils envoyées au point de terminaison `devicedata` Azure Event Hub. Il extrait les types, les identificateurs d’appareil, les dates et heures de mesure, ainsi que la ou les valeur(s) de mesure. Le second type, le **mappage FHIR**, contrôle le mappage de la ressource FHIR. Il permet de configurer la durée de la période d’observation, le type de données FHIR utilisé pour stocker les valeurs et le ou les code(s) de terminologie. 

Les modèles de mappage sont composés dans un document JSON en fonction de leur type. Ces documents JSON sont ensuite ajoutés à votre connecteur Azure IoT pour FHIR via le Portail Azure. Le document de mappage d’appareils est ajouté via la page **Configurer le mappage d’appareils** et le document de mappage FHIR via la page **Configurer le mappage de FHIR**.

> [!NOTE]
> Les modèles de mappage sont stockés dans un stockage d’objets BLOB sous-jacent et chargés à partir d’un objet BLOB par exécution de calcul. Une fois mis à jour, ils doivent prendre effet immédiatement. 

## <a name="device-mapping"></a>Mappage d’appareil
Le mappage d’appareils fournit des fonctionnalités de mappage pour extraire le contenu de l’appareil dans un format commun pour une évaluation supplémentaire. Chaque message reçu est évalué par rapport à tous les modèles. Cette approche permet de projeter un message entrant unique pour plusieurs messages sortants, qui sont ensuite mappés à différentes observations dans FHIR. Le résultat est un objet de données normalisé représentant la valeur ou les valeurs analysées par les modèles. Le modèle de données normalisé contient quelques propriétés obligatoires qui doivent être recherchées et extraites :

| Propriété | Description |
| - | - |
|**Type**|Nom/type pour classifier la mesure. Cette valeur est utilisée pour établir une liaison avec le modèle de mappage FHIR requis.  Plusieurs modèles peuvent être générés dans le même type, ce qui vous permet de mapper différentes représentations sur plusieurs appareils à une sortie commune unique.|
|**OccurenceTimeUtc**|Heure à laquelle la mesure a été effectuée.|
|**DeviceId**|Identificateur de l’appareil. Cette valeur doit correspondre à un identificateur sur la ressource de l’appareil qui existe sur le serveur FHIR de destination.|
 |**Propriétés**|Extrayez au moins une propriété afin que la valeur puisse être enregistrée dans la ressource d’observation créée.  Les propriétés sont un regroupement de paires de valeurs clés extraites pendant la normalisation.|

Vous trouverez ci-dessous un exemple conceptuel de ce qui se passe lors de la normalisation.

![Exemple de normalisation](media/concepts-iot-mapping-templates/normalization-example.png)

La charge utile de contenu est un message Azure Event Hub, qui est composé de trois parties : Corps, Propriétés et SystemProperties. Le `Body` est un tableau d’octets qui représente une chaîne encodée en UTF-8. Pendant l’évaluation du modèle, le tableau d’octets est automatiquement converti en valeur de chaîne. `Properties` est un regroupement de valeurs clés à utiliser par le créateur du message. `SystemProperties` est également un regroupement de valeurs clés réservé par l’infrastructure Azure Event Hub avec des entrées automatiquement remplies par celles-ci.

```json
{
    "Body": {
        "content": "value"
    },
    "Properties": {
        "key1": "value1",
        "key2": "value2"
    },
    "SystemProperties": {
        "x-opt-sequence-number": 1,
        "x-opt-enqueued-time": "2019-02-01T22:46:01.8750000Z",
        "x-opt-offset": 1,
        "x-opt-partition-key": "1"
    }
}
```

### <a name="mapping-with-json-path"></a>Le mappage avec le chemin d’accès JSON
Les trois types de modèles de contenu d’appareil pris en charge aujourd’hui s’appuient sur le chemin d’accès JSON pour correspondre au modèle et aux valeurs extraites requis. Des informations supplémentaires sur le chemin d’accès JSON sont disponibles [ici](https://goessner.net/articles/JsonPath/). Les trois types de modèles utilisent l’[implémentation .NET JSON](https://www.newtonsoft.com/json/help/html/QueryJsonSelectTokenJsonPath.htm) pour la résolution des expressions de chemin d’accès JSON.

#### <a name="jsonpathcontenttemplate"></a>JsonPathContentTemplate
JsonPathContentTemplate permet la correspondance et l’extraction des valeurs d’un message Event Hub à l’aide du chemin d’accès JSON.

| Propriété | Description |<div style="width:150px">Exemple</div>
| --- | --- | --- 
|**TypeName**|Type à associer aux mesures correspondant au modèle.|`heartrate`
|**TypeMatchExpression**|Expression de chemin d’accès JSON qui est évaluée par rapport à la charge utile Event Hub. Si un JToken correspondant est trouvé, le modèle est considéré comme une correspondance. Toutes les expressions suivantes sont évaluées par rapport aux JToken extraits correspondants.|`$..[?(@heartRate)]`
|**TimestampExpression**|Expression de chemin d’accès JSON pour extraire la valeur d’horodatage pour le OccurenceTimeUtc de la mesure.|`$.endDate`
|**DeviceIdExpression**|Expression de chemin d’accès JSON pour extraire l’identificateur d’appareil.|`$.deviceId`
|**PatientIdExpression**|*Facultatif* : Expression de chemin d’accès JSON pour extraire l’identificateur de patient.|`$.patientId`
|**EncounterIdExpression**|*Facultatif* : Expression de chemin d’accès JSON pour extraire l’identificateur de rencontre.|`$.encounterId`
|**Values[].ValueName**|Nom à associer à la valeur extraite par l’expression suivante. Utilisé pour lier la valeur/le composant requis dans le modèle de mappage FHIR. |`hr`
|**Values[].ValueExpression**|Expression de chemin d’accès JSON pour extraire la valeur requise.|`$.heartRate`
|**Values[].Required**|Nécessite que la valeur soit présente dans la charge utile.  Si la valeur est introuvable, la mesure n’est pas générée et une exception InvalidOperationException est levée.|`true`

##### <a name="examples"></a>Exemples
---
**Fréquence cardiaque**

*Message*
```json
{
    "Body": {
        "heartRate": "78",
        "endDate": "2019-02-01T22:46:01.8750000Z",
        "deviceId": "device123"
    },
    "Properties": {},
    "SystemProperties": {}
}
```
*Modèle*
```json
{
    "templateType": "JsonPathContent",
    "template": {
        "typeName": "heartrate",
        "typeMatchExpression": "$..[?(@heartRate)]",
        "deviceIdExpression": "$.deviceId",
        "timestampExpression": "$.endDate",
        "values": [
            {
                "required": "true",
                "valueExpression": "$.heartRate",
                "valueName": "hr"
            }
        ]
    }
}
```
---
**Tension artérielle**

*Message*
```json
{
    "Body": {
        "systolic": "123",
        "diastolic" : "87",
        "endDate": "2019-02-01T22:46:01.8750000Z",
        "deviceId": "device123"
    },
    "Properties": {},
    "SystemProperties": {}
}
```
*Modèle*
```json
{
    "typeName": "bloodpressure",
    "typeMatchExpression": "$..[?(@systolic && @diastolic)]",
    "deviceIdExpression": "$.deviceid",
    "timestampExpression": "$.endDate",
    "values": [
        {
            "required": "true",
            "valueExpression": "$.systolic",
            "valueName": "systolic"
        },
        {
            "required": "true",
            "valueExpression": "$.diastolic",
            "valueName": "diastolic"
        }
    ]
}
```
---

**Projeter plusieurs mesures à partir d’un seul message**

*Message*
```json
{
    "Body": {
        "heartRate": "78",
        "steps": "2",
        "endDate": "2019-02-01T22:46:01.8750000Z",
        "deviceId": "device123"
    },
    "Properties": {},
    "SystemProperties": {}
}
```
*Modèle 1*
```json
{
    "templateType": "JsonPathContent",
    "template": {
        "typeName": "heartrate",
        "typeMatchExpression": "$..[?(@heartRate)]",
        "deviceIdExpression": "$.deviceId",
        "timestampExpression": "$.endDate",
        "values": [
            {
                "required": "true",
                "valueExpression": "$.heartRate",
                "valueName": "hr"
            }
        ]
    }
}
```
*Modèle 2*
```json
{
    "templateType": "JsonPathContent",
    "template": {
        "typeName": "stepcount",
        "typeMatchExpression": "$..[?(@steps)]",
        "deviceIdExpression": "$.deviceId",
        "timestampExpression": "$.endDate",
        "values": [
            {
                "required": "true",
                "valueExpression": "$.steps",
                "valueName": "steps"
            }
        ]
    }
}
```
---

**Projeter plusieurs mesures à partir d’un tableau dans un message**

*Message*
```json
{
    "Body": [
        {
            "heartRate": "78",
            "endDate": "2019-02-01T22:46:01.8750000Z",
            "deviceId": "device123"
        },
        {
            "heartRate": "81",
            "endDate": "2019-02-01T23:46:01.8750000Z",
            "deviceId": "device123"
        },
        {
            "heartRate": "72",
            "endDate": "2019-02-01T24:46:01.8750000Z",
            "deviceId": "device123"
        }
    ],
    "Properties": {},
    "SystemProperties": {}
}
```
*Modèle*
```json
{
    "templateType": "JsonPathContent",
    "template": {
        "typeName": "heartrate",
        "typeMatchExpression": "$..[?(@heartRate)]",
        "deviceIdExpression": "$.deviceId",
        "timestampExpression": "$.endDate",
        "values": [
            {
                "required": "true",
                "valueExpression": "$.heartRate",
                "valueName": "hr"
            }
        ]
    }
}
```

#### <a name="iotjsonpathcontenttemplate"></a>IotJsonPathContentTemplate

IotJsonPathContentTemplate est similaire à JsonPathContentTemplate, sauf que DeviceIdExpression et TimestampExpression ne sont pas requis.

L’hypothèse de l’utilisation de ce modèle est que les messages en cours d’évaluation ont été envoyés à l’aide des [kits de développement logiciel (SDK) d’appareils Azure IoT Hub](../iot-hub/iot-hub-devguide-sdks.md#azure-iot-hub-device-sdks) ou de la fonctionnalité [Exporter des données (héritée)](../iot-central/core/howto-export-data-legacy.md) d’[Azure IoT Central](../iot-central/core/overview-iot-central.md). Lors de l’utilisation de ces kits de développement logiciel (SDK), l’identité de l’appareil (en supposant que l’identificateur d’appareil d’Azure IOT Hub/Central est inscrit comme identificateur pour une ressource d’appareil sur le serveur FHIR de destination) et l’horodateur du message sont connus. Si vous utilisez les kits de développement logiciel (SDK) d’appareils Azure IoT Hub, mais que vous utilisez des propriétés personnalisées dans le corps du message pour l’identité de l’appareil ou l’horodatage de mesure, vous pouvez toujours utiliser JsonPathContentTemplate.

*Remarque : Lors de l’utilisation de IotJsonPathContentTemplate, TypeMatchExpression doit être résolu dans tout le message en tant que JToken. Consultez les exemples ci-dessous.* 
##### <a name="examples"></a>Exemples
---
**Fréquence cardiaque**

*Message*
```json
{
    "Body": {
        "heartRate": "78"        
    },
    "Properties": {
        "iothub-creation-time-utc" : "2019-02-01T22:46:01.8750000Z"
    },
    "SystemProperties": {
        "iothub-connection-device-id" : "device123"
    }
}
```
*Modèle*
```json
{
    "templateType": "JsonPathContent",
    "template": {
        "typeName": "heartrate",
        "typeMatchExpression": "$..[?(@Body.heartRate)]",
        "deviceIdExpression": "$.deviceId",
        "timestampExpression": "$.endDate",
        "values": [
            {
                "required": "true",
                "valueExpression": "$.Body.heartRate",
                "valueName": "hr"
            }
        ]
    }
}
```
---
**Tension artérielle**

*Message*
```json
{
    "Body": {
        "systolic": "123",
        "diastolic" : "87"
    },
    "Properties": {
        "iothub-creation-time-utc" : "2019-02-01T22:46:01.8750000Z"
    },
    "SystemProperties": {
        "iothub-connection-device-id" : "device123"
    }
}
```
*Modèle*
```json
{
    "typeName": "bloodpressure",
    "typeMatchExpression": "$..[?(@Body.systolic && @Body.diastolic)]",
    "values": [
        {
            "required": "true",
            "valueExpression": "$.Body.systolic",
            "valueName": "systolic"
        },
        {
            "required": "true",
            "valueExpression": "$.Body.diastolic",
            "valueName": "diastolic"
        }
    ]
}
```

#### <a name="iotcentraljsonpathcontenttemplate"></a>IotCentralJsonPathContentTemplate

Le paramètre IotCentralJsonPathContentTemplate n’a pas non plus besoin de DeviceIdExpression ni de TimestampExpression, et il est utilisé lorsque les messages en cours d’évaluation sont envoyés via la fonctionnalité [Exporter des données](../iot-central/core/howto-export-data.md) d’[Azure IoT Central](../iot-central/core/overview-iot-central.md). Lors de l’utilisation de cette fonctionnalité, l’identité de l’appareil (en supposant que l’identificateur d’appareil d’Azure IOT Central est inscrit comme identificateur pour une ressource d’appareil sur le serveur FHIR de destination) et l’horodateur du message sont connus. Si vous utilisez la fonctionnalité Exporter des données d’Azure IoT Central, mais que vous utilisez des propriétés personnalisées dans le corps du message pour l’identité de l’appareil ou l’horodatage de mesure, vous pouvez toujours utiliser JsonPathContentTemplate.

*Remarque : Lors de l’utilisation de IotCentralJsonPathContentTemplate, TypeMatchExpression doit être résolu dans tout le message en tant que JToken. Consultez les exemples ci-dessous.* 
##### <a name="examples"></a>Exemples
---
**Fréquence cardiaque**

*Message*
```json
{
    "applicationId": "1dffa667-9bee-4f16-b243-25ad4151475e",
    "messageSource": "telemetry",
    "deviceId": "1vzb5ghlsg1",
    "schema": "default@v1",
    "templateId": "urn:qugj6vbw5:___qbj_27r",
    "enqueuedTime": "2020-08-05T22:26:55.455Z",
    "telemetry": {
        "HeartRate": "88",
    },
    "enrichments": {
      "userSpecifiedKey": "sampleValue"
    },
    "messageProperties": {
      "messageProp": "value"
    }
}
```
*Modèle*
```json
{
    "templateType": "IotCentralJsonPathContent",
    "template": {
        "typeName": "heartrate",
        "typeMatchExpression": "$..[?(@telemetry.HeartRate)]",
        "values": [
            {
                "required": "true",
                "valueExpression": "$.telemetry.HeartRate",
                "valueName": "hr"
            }
        ]
    }
}
```
---
**Tension artérielle**

*Message*
```json
{
    "applicationId": "1dffa667-9bee-4f16-b243-25ad4151475e",
    "messageSource": "telemetry",
    "deviceId": "1vzb5ghlsg1",
    "schema": "default@v1",
    "templateId": "urn:qugj6vbw5:___qbj_27r",
    "enqueuedTime": "2020-08-05T22:26:55.455Z",
    "telemetry": {
        "BloodPressure": {
            "Diastolic": "87",
            "Systolic": "123"
        }
    },
    "enrichments": {
      "userSpecifiedKey": "sampleValue"
    },
    "messageProperties": {
      "messageProp": "value"
    }
}
```
*Modèle*
```json
{
    "templateType": "IotCentralJsonPathContent",
    "template": {
        "typeName": "bloodPressure",
        "typeMatchExpression": "$..[?(@telemetry.BloodPressure.Diastolic && @telemetry.BloodPressure.Systolic)]",
        "values": [
            {
                "required": "true",
                "valueExpression": "$.telemetry.BloodPressure.Diastolic",
                "valueName": "bp_diastolic"
            },
            {
                "required": "true",
                "valueExpression": "$.telemetry.BloodPressure.Systolic",
                "valueName": "bp_systolic"
            }
        ]
    }
}
```

## <a name="fhir-mapping"></a>Mappage FHIR
Une fois que le contenu de l’appareil est extrait dans un modèle normalisé, les données sont collectées et regroupées en fonction de l’identificateur de l’appareil, du type de mesure et de la période de temps. La sortie de ce regroupement est envoyée pour la conversion en ressource FHIR (actuellement [Observation](https://www.hl7.org/fhir/observation.html)). Ici, le modèle de mappage FHIR contrôle la manière dont les données sont mappées à une Observation FHIR. Une observation doit-elle être créée pour un point dans le temps ou sur une période d’une heure ? Quels codes doivent être ajoutés à l’observation ? La valeur doit-elle être représentée en tant que [SampledData](https://www.hl7.org/fhir/datatypes.html#SampledData) ou [Quantity](https://www.hl7.org/fhir/datatypes.html#Quantity) ? Ces types de données sont toutes des options contrôlées par la configuration du mappage FHIR.

### <a name="codevaluefhirtemplate"></a>CodeValueFhirTemplate
CodeValueFhirTemplate est actuellement le seul modèle pris en charge dans le mappage FHIR pour l’instant.  Il vous permet de définir des codes, la période d’effet et la valeur de l’observation. Plusieurs types valeur sont pris en charge : [SampledData](https://www.hl7.org/fhir/datatypes.html#SampledData), [CodeableConcept](https://www.hl7.org/fhir/datatypes.html#CodeableConcept) et [Quantity](https://www.hl7.org/fhir/datatypes.html#Quantity). Outre ces valeurs configurables, l’identificateur de la ressource d’observation et la liaison à l’appareil approprié et aux ressources des patients sont gérés automatiquement.

| Propriété | Description 
| --- | ---
|**TypeName**| Type de mesure auquel ce modèle doit être lié. Il doit y avoir au moins un modèle de mappage d’appareil qui génère ce type.
|**PeriodInterval**|Période pendant laquelle l’observation créée doit être représentée. Les valeurs prises en charge sont 0 (une instance), 60 (une heure), 1440 (un jour).
|**Catégorie**|N’importe quel nombre de [CodeableConcepts](http://hl7.org/fhir/datatypes-definitions.html#codeableconcept) pour classifier le type d’observation créé.
|**Codes**|Un ou plusieurs [Codages](http://hl7.org/fhir/datatypes-definitions.html#coding) à appliquer à l’observation créée.
|**Codes[].Code**|Code du [codage](http://hl7.org/fhir/datatypes-definitions.html#coding).
|**Codes[].System**|Système pour le [Codage](http://hl7.org/fhir/datatypes-definitions.html#coding).
|**Codes[].Display**|Affichage pour le [Codage](http://hl7.org/fhir/datatypes-definitions.html#coding).
|**Valeur**|Valeur à extraire et à représenter dans l’observation. Pour plus d’informations, consultez [Modèles de type valeur](#valuetypes).
|**Composants**|*Facultatif :* Un ou plusieurs composants à créer sur l’observation.
|**Components[].Codes**|Un ou plusieurs [Codages](http://hl7.org/fhir/datatypes-definitions.html#coding) à appliquer au composant.
|**Components[].Value**|Valeur à extraire et à représenter dans le composant. Pour plus d’informations, consultez [Modèles de type valeur](#valuetypes).

### <a name="value-type-templates"></a>Modèles de type valeur <a name="valuetypes"></a>
Vous trouverez ci-dessous les modèles de type valeur actuellement pris en charge. À l’avenir, d’autres modèles pourront être ajoutés.
#### <a name="sampleddata"></a>SampledData
Représente le type de données [SampledData](http://hl7.org/fhir/datatypes.html#SampledData) FHIR. Les mesures d’observation sont écrites dans un flux de valeurs démarrant à un point précis dans le temps et incrémentant le transfert à l’aide de la période définie. Si aucune valeur n’est présente, un `E` est écrit dans le flux de données. Si la période est telle que deux valeurs supplémentaires occupent la même position dans le flux de données, la valeur la plus récente est utilisée. La même logique est appliquée lorsqu’une observation utilisant SampledData est mise à jour.

| Propriété | Description 
| --- | ---
|**DefaultPeriod**|Période par défaut en millisecondes à utiliser. 
|**Unité**|Unité à définir sur l’origine de SampledData. 

#### <a name="quantity"></a>Quantité
Représente le type de données [Quantity](http://hl7.org/fhir/datatypes.html#Quantity) FHIR. Si plusieurs valeurs sont présentes dans le regroupement, seule la première valeur est utilisée. Quand une nouvelle valeur arrive et qu’elle est mappée à la même observation, elle remplace l’ancienne valeur.

| Propriété | Description 
| --- | --- 
|**Unité**| Représentation de l’unité.
|**Code**| Forme codée de l’unité.
|**Système**| Système qui définit la forme d’unité codée.

### <a name="codeableconcept"></a>CodeableConcept
Représente le type de données [CodeableConcept](http://hl7.org/fhir/datatypes.html#CodeableConcept) FHIR. La valeur réelle n’est pas utilisée.

| Propriété | Description 
| --- | --- 
|**Text**|Représentation du texte brut. 
|**Codes**|Un ou plusieurs [Codages](http://hl7.org/fhir/datatypes-definitions.html#coding) à appliquer à l’observation créée.
|**Codes[].Code**|Code du [codage](http://hl7.org/fhir/datatypes-definitions.html#coding).
|**Codes[].System**|Système pour le [Codage](http://hl7.org/fhir/datatypes-definitions.html#coding).
|**Codes[].Display**|Affichage pour le [Codage](http://hl7.org/fhir/datatypes-definitions.html#coding).

### <a name="examples"></a>Exemples
**Fréquence cardiaque : SampledData**
```json
{
    "templateType": "CodeValueFhir",
    "template": {
        "codes": [
            {
                "code": "8867-4",
                "system": "http://loinc.org",
                "display": "Heart rate"
            }
        ],
        "periodInterval": 60,
        "typeName": "heartrate",
        "value": {
            "defaultPeriod": 5000,
            "unit": "count/min",
            "valueName": "hr",
            "valueType": "SampledData"
        }
    }
}
```
---
**Étapes : SampledData**
```json
{
    "templateType": "CodeValueFhir",
    "template": {
        "codes": [
            {
                "code": "55423-8",
                "system": "http://loinc.org",
                "display": "Number of steps"
            }
        ],        
        "periodInterval": 60,
        "typeName": "stepsCount",
        "value": {
            "defaultPeriod": 5000,
            "unit": "",
            "valueName": "steps",
            "valueType": "SampledData"
        }
    }
}
```
---
**Pression artérielle : SampledData**
```json
{
    "templateType": "CodeValueFhir",
    "template": {
        "codes": [
            {
                "code": "85354-9",
                "display": "Blood pressure panel with all children optional",
                "system": "http://loinc.org"
            }
        ],
        "periodInterval": 60,
        "typeName": "bloodpressure",
        "components": [
            {
                "codes": [
                    {
                        "code": "8867-4",
                        "display": "Diastolic blood pressure",
                        "system": "http://loinc.org"
                    }
                ],
                "value": {
                    "defaultPeriod": 5000,
                    "unit": "mmHg",
                    "valueName": "diastolic",
                    "valueType": "SampledData"
                }
            },
            {
                "codes": [
                    {
                        "code": "8480-6",
                        "display": "Systolic blood pressure",
                        "system": "http://loinc.org"
                    }
                ],
                "value": {
                    "defaultPeriod": 5000,
                    "unit": "mmHg",
                    "valueName": "systolic",
                    "valueType": "SampledData"
                }
            }
        ]
    }
}
```
---
**Pression artérielle : Quantity**
```json
{
    "templateType": "CodeValueFhir",
    "template": {
        "codes": [
            {
                "code": "85354-9",
                "display": "Blood pressure panel with all children optional",
                "system": "http://loinc.org"
            }
        ],
        "periodInterval": 0,
        "typeName": "bloodpressure",
        "components": [
            {
                "codes": [
                    {
                        "code": "8867-4",
                        "display": "Diastolic blood pressure",
                        "system": "http://loinc.org"
                    }
                ],
                "value": {
                    "unit": "mmHg",
                    "valueName": "diastolic",
                    "valueType": "Quantity"
                }
            },
            {
                "codes": [
                    {
                        "code": "8480-6",
                        "display": "Systolic blood pressure",
                        "system": "http://loinc.org"
                    }
                ],
                "value": {
                    "unit": "mmHg",
                    "valueName": "systolic",
                    "valueType": "Quantity"
                }
            }
        ]
    }
}
```
---
**Appareil supprimé : CodeableConcept**
```json
{
    "templateType": "CodeValueFhir",
    "template": {
        "codes": [
            {
                "code": "deviceEvent",
                "system": "https://www.mydevice.com/v1",
                "display": "Device Event"
            }
        ],
        "periodInterval": 0,
        "typeName": "deviceRemoved",
        "value": {
            "text": "Device Removed",
            "codes": [
                {
                    "code": "deviceRemoved",
                    "system": "https://www.mydevice.com/v1",
                    "display": "Device Removed"
                }
            ],
            "valueName": "deviceRemoved",
            "valueType": "CodeableConcept"
        }
    }
}
```
---

## <a name="next-steps"></a>Étapes suivantes

Consultez les questions fréquemment posées sur le connecteur Azure IoT pour FHIR (préversion).

>[!div class="nextstepaction"]
>[FAQ sur le Connecteur Azure IoT pour FHIR](fhir-faq.md)

*Sur le Portail Azure, le Connecteur Azure IoT pour FHIR est appelé Connecteur IoT (préversion). Le sigle FHIR est une marque déposée de HL7 et est utilisé avec l’autorisation de HL7.