---
title: Intégration de partenaire d’imagerie
description: Cet article décrit l’intégration de partenaire d’imagerie.
author: uhabiba04
ms.topic: article
ms.date: 11/04/2019
ms.author: v-umha
ms.custom: has-adal-ref
ms.openlocfilehash: 40e17a9f73d98f24c6c7fbb55ef38cc2bc088861
ms.sourcegitcommit: 4cb89d880be26a2a4531fedcc59317471fe729cd
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/27/2020
ms.locfileid: "92675933"
---
# <a name="imagery-partner-integration"></a>Intégration de partenaire d’imagerie

Cet article explique comment utiliser le composant Azure FarmBeats Translator pour envoyer des données d’imagerie à FarmBeats. Les données d’imagerie agricole peuvent être générées à partir de sources diverses, notamment des caméras multispectrales, des satellites et des drones. Les partenaires d’imagerie agricole peuvent s’intégrer à FarmBeats afin de fournir aux clients des cartes personnalisées pour leurs fermes.

Une fois les données disponibles, elles peuvent être visualisées à l’aide de FarmBeats Accelerator. Par ailleurs, les entreprises agricoles et intégrateurs systèmes clients peuvent les utiliser pour la fusion de données et la création de modèles ML/IA (machine learning/intelligence artificielle).

FarmBeats offre les possibilités suivantes :

- Définir des types, une source et un format de fichier d’image personnalisés à l’aide des API /ExtendedType.
- Ingérer des données d’imagerie de diverses sources avec les API /Scene et /SceneFile.

Les informations suivantes expliquent comment récupérer tout type d’imagerie dans le système FarmBeats.

Quand vous sélectionnez la section **Drone Imagery** , une fenêtre indépendante s’ouvre avec une image haute résolution de l’orthophotographie du drone. Vous pouvez accéder au logiciel du partenaire, qui vous permet de planifier les vols de drone et de récupérer les données brutes. Vous l’utiliserez également pour la planification des trajectoires et l’assemblage des images orthophotographiques.

Les partenaires de drone doivent permettre aux clients de lier leur compte client à leur instance FarmBeats sur Azure.

Vous devez utiliser les informations d’identification suivantes dans le logiciel du partenaire de drone pour lier FarmBeats :

- Point de terminaison d’API
- ID client
- ID client
- Clé secrète client

## <a name="api-development"></a>Développement d’API

Les API incluent une documentation technique Swagger. Pour plus d’informations sur les API et les requêtes ou réponses correspondantes, consultez [Swagger](https://aka.ms/FarmBeatsDatahubSwagger).

## <a name="authentication"></a>Authentification

FarmBeats utilise Microsoft Azure [Active Directory](../../app-service/overview-authentication-authorization.md) (Azure AD). Azure App Service offre une prise en charge intégrée de l’authentification et de l’autorisation. 

Pour plus d’informations sur Azure AD, consultez [Azure Active Directory](../../app-service/overview-authentication-authorization.md).   

FarmBeats Datahub utilise l’authentification du porteur, qui nécessite les informations d’identification suivantes :

- ID client
- Clé secrète client
- ID client

À l’aide des informations d’identification précédentes, l’appelant peut demander un jeton d’accès, qui doit être envoyé dans les requêtes d’API suivantes (dans la section d’en-tête) comme suit :

```
headers = {"Authorization": "Bearer " + access_token, …} 
```

L’exemple de code Python suivant récupère le jeton d’accès. Vous pouvez ensuite utiliser le jeton pour les appels d’API suivants à FarmBeats.

```python
import requests
import json
import msal

# Your service principal App ID
CLIENT_ID = "<CLIENT_ID>"
# Your service principal password
CLIENT_SECRET = "<CLIENT_SECRET>"
# Tenant ID for your Azure subscription
TENANT_ID = "<TENANT_ID>"

AUTHORITY_HOST = 'https://login.microsoftonline.com'
AUTHORITY = AUTHORITY_HOST + '/' + TENANT_ID

ENDPOINT = "https://<yourfarmbeatswebsitename-api>.azurewebsites.net"
SCOPE = ENDPOINT + "/.default"

context = msal.ConfidentialClientApplication(CLIENT_ID, authority=AUTHORITY, client_credential=CLIENT_SECRET)
token_response = context.acquire_token_for_client(SCOPE)
# We should get an access token here
access_token = token_response.get('access_token')
```

## <a name="http-request-headers"></a>En-têtes de requête HTTP

Voici les en-têtes de demande les plus courants qui doivent être spécifiés lors d’un appel d’API à FarmBeats Datahub.

**En-tête** | **Description et exemple**
--- | ---
Content-Type  | Format de la demande (Content-Type: application/<format>). Pour les API FarmBeats Datahub, le format est JSON. Content-Type: application/json
Autorisation | Spécifie le jeton d’accès requis pour effectuer un appel d’API. Autorisation : Porteur <jeton-accès>
Acceptation  | Format de la réponse. Pour les API FarmBeats Datahub, le format est JSON. Accept: application/json


## <a name="api-requests"></a>Requêtes d’API

Pour effectuer une requête d’API REST, vous combinez :

- La méthode HTTP (GET, POST et PUT).
- L’URL du service d’API.
- L’URI de la ressource (pour interroger, envoyer des données, mettre à jour ou supprimer).
- Un ou plusieurs en-têtes de requête HTTP.

Si vous le souhaitez, vous pouvez inclure des paramètres de requête à des appels GET pour filtrer les données dans les réponses, en limiter la taille et les trier.

L’exemple de requête suivant porte sur l’obtention de la liste des appareils :

```bash
curl -X GET "https://microsoft-farmbeats.azurewebsites.net/Device" -H
"Content-Type: application/json" -H
"Authorization: Bearer <Access-Token>”
```

La plupart des appels GET, POST et PUT nécessitent un corps de demande JSON.

L’exemple de demande suivant porte sur la création d’un appareil. Cet exemple contient un code JSON d’entrée avec le corps de la demande.


```bash
curl -X POST "https://microsoft-farmbeats.azurewebsites.net/Device" -H
"accept: application/json" -H
"Content-Type: application/json" -H "Authorization: Bearer <Access-Token>" -d
"{  \"deviceModelId\": \"ID123\",  \"hardwareId\": \"MHDN123\",  \"reportingInterval\": 900,
\"name\": \"Device123\",  \"description\": \"Test Device 123\",}"
```

## <a name="data-format"></a>Format de données

JSON est un format de données courant, indépendant du langage, qui fournit une représentation textuelle simple de structures de données arbitraires. Pour plus d’informations, consultez [JSON.org](https://JSON.org).

## <a name="ingest-imagery-into-farmbeats"></a>Ingérer des données d’imagerie dans FarmBeats

Une fois que le partenaire dispose d’informations d’identification pour se connecter à FarmBeats Datahub, il effectue les étapes suivantes dans le composant Translator.

1.  Il crée un type étendu pour les champs suivants en fonction du type de données imagerie à charger :

    - **Source de la scène**  : Par exemple, drone_partner_name
    - **Type de scène**  : Par exemple, drone
    - **Type de fichier de scène**  : Par exemple, chlorophyll index
    - **Type de contenu de fichier de scène**  : Par exemple, image/tiff

2.  Il appelle l’API /Farms pour obtenir la liste des fermes dans le système Azure FarmBeats.
3.  Il donne au client la possibilité de choisir une ferme unique dans la liste des fermes.

    Le système partenaire doit afficher la ferme dans le logiciel partenaire. Il permet ainsi de planifier les trajectoires, de procéder au vol du drone et de collecter les images.

4.  Il appelle l’API /Scene et fournit les informations nécessaires pour créer une scène avec un ID de scène unique.
5.  Il reçoit une URL SAS d’objet blob pour charger les images requises dans FarmBeats Datahub dans le contexte de la ferme choisie, dans le système FarmBeats.

Voici un flux détaillé sur les appels d’API.

### <a name="step-1-extendedtype"></a>Étape 1 : ExtendedType

Dans l’API /ExtendedType, vérifiez si le type et la source du fichier sont disponibles sur FarmBeats. Pour ce faire, appelez un GET sur l’API /ExtendedType.

Voici les valeurs définies par le système :

```json
{
  "items": [
    {
      "id": "ae2aa527-7d27-42e2-8fcf-841937c651d7",
      "createdAt": "2019-06-17T07:32:30.1135716Z",
      "lastModifiedAt": "2019-10-25T09:47:42.5910344Z",
      "key": "SceneFileType",
      "value": [
        "evi",
        "ndvi",
        "ndwi",
        "tci",
        "soil-moisture",
        "sensor-placement",
        "sentinel-b01",
        "sentinel-b02",
        "sentinel-b03",
        "sentinel-b04",
        "sentinel-b05",
        "sentinel-b06",
        "sentinel-b07",
        "sentinel-b08",
        "sentinel-b8a",
        "sentinel-b09",
        "sentinel-b10",
        "sentinel-b11",
        "sentinel-b12",
        "cloud-mask",
        "farm-mask",
        "image/tiff",
        "orthomosaic"
      ],
      "description": "List of scene file types available in system. User can add more values."
    },
    {
      "id": "b7824d2e-3e43-46d3-8fd4-ec7ba8918d84",
      "createdAt": "2019-06-17T07:32:30.1135716Z",
      "lastModifiedAt": "2019-10-25T09:45:44.8914691Z",
      "key": "SceneFileContentType",
      "value": [
        "image/tiff",
        "image/png",
        "image/jpeg",
        "text/csv",
        "text/plain",
        "text/tab-separated-values",
        "application/json",
        "application/octet-stream",
        "orthomosaic"
      ],
      "description": "List of scene file content types available in system. User can add more values."
    },
    {
      "id": "21d2924d-8b3e-4ce4-a7f8-318e85c0f7f2",
      "createdAt": "2019-06-17T07:32:30.1135716Z",
      "lastModifiedAt": "2019-10-25T09:45:37.5096445Z",
      "key": "SceneSource",
      "value": [
        "sentinel-l1c",
        "sentinel-l2a",
        "farmbeats-model",
        "dji",
        "SlantRange-3P",
        "DJI"
      ],
      "description": "List of scene sources available in system. User can add more values."
    },
    {
      "id": "6e09ca69-cacf-4b53-b63a-53c659aae4a4",
      "createdAt": "2019-06-17T07:32:30.1135716Z",
      "lastModifiedAt": "2019-10-25T09:45:34.7483899Z",
      "key": "SceneType",
      "value": [
        "base-bands",
        "sensor-placement",
        "soil-moisture",
        "evi",
        "ndwi",
        "ndvi",
        "drone",
        "orthomosaic-drone"
      ],
      "description": "List of scene types available in system. User can add more values."
    },
    {
      "id": "55d8436b-3a86-4bea-87ab-e1b51226525f",
      "createdAt": "2019-06-17T07:32:30.1135716Z",
      "lastModifiedAt": "2019-06-17T07:32:30.1135716Z",
      "key": "SensorMeasureType",
      "value": [
        "AmbientTemperature",
        "CO2",
        "Depth",
        "ElectricalConductivity",
        "LeafWetness",
        "Length",
        "LiquidLevel",
        "Nitrate",
        "O2",
        "PH",
        "Phosphate",
        "PointInTime",
        "Potassium",
        "Pressure",
        "RainGauge",
        "RelativeHumidity",
        "Salinity",
        "SoilMoisture",
        "SoilTemperature",
        "SolarRadiation",
        "State",
        "TimeDuration",
        "UVRadiation",
        "UVIndex",
        "Volume",
        "WindDirection",
        "WindRun",
        "WindSpeed",
        "Evapotranspiration",
        "PAR"
      ],
      "description": "List of sensor measure types available in system. User can add more values."
    },
    {
      "id": "0dfd6e6b-df58-428f-9ab8-a0674bfdcbe5",
      "createdAt": "2019-06-17T07:32:30.1135716Z",
      "lastModifiedAt": "2019-06-17T07:32:30.1135716Z",
      "key": "SensorMeasureUnit",
      "value": [
        "NoUnit",
        "Celsius",
        "Fahrenheit",
        "Kelvin",
        "Rankine",
        "Pascal",
        "Mercury",
        "PSI",
        "MilliMeter",
        "CentiMeter",
        "Meter",
        "Inch",
        "Feet",
        "Mile",
        "KiloMeter",
        "MilesPerHour",
        "MilesPerSecond",
        "KMPerHour",
        "KMPerSecond",
        "MetersPerHour",
        "MetersPerSecond",
        "Degree",
        "WattsPerSquareMeter",
        "KiloWattsPerSquareMeter",
        "MilliWattsPerSquareCentiMeter",
        "MilliJoulesPerSquareCentiMeter",
        "VolumetricWaterContent",
        "Percentage",
        "PartsPerMillion",
        "MicroMol",
        "MicroMolesPerLiter",
        "SiemensPerSquareMeterPerMole",
        "MilliSiemensPerCentiMeter",
        "Centibar",
        "DeciSiemensPerMeter",
        "KiloPascal",
        "VolumetricIonContent",
        "Liter",
        "MilliLiter",
        "Seconds",
        "UnixTimestamp",
        "MicroMolPerMeterSquaredPerSecond",
        "InchesPerHour"
      ],
      "description": "List of sensor measure units available in system. User can add more values."
    }
  ]
}
```

Il s’agit d’une configuration unique. La portée de ce nouveau type de scène est limitée à l’abonnement dans lequel Azure FarmBeats est installé.

Par exemple, pour ajouter SceneSource : "SlantRange", vous effectuez un PUT sur l’ID de l’API /ExtendedType avec la charge utile d’entrée "SceneSource" comme clé.

```json
{

      "key": "SceneSource",
      "value": [
        "sentinel-l1c",
        "sentinel-l2a",
        "farmbeats-model",
        "dji",
        "SlantRange-3P"
      ],
      "description": "List of scene sources available in system. User can add more values."
}

```

Le champ vert est le nouvel ajout aux valeurs de source de scène définies par le système.

### <a name="step-2-get-farm-details"></a>Étape 2 : Obtenir les détails de la ferme

Les scènes (fichiers .tiff ou .csv) sont associées au contexte d’une ferme. Vous devez obtenir les informations sur la ferme en effectuant un GET sur l’API /Farm. L’API retourne la liste des fermes disponibles dans FarmBeats. Vous pouvez sélectionner la ferme pour laquelle vous souhaitez ingérer les données.

Réponse GET sur /Farm :

```json
{
  "id": "07f3e09c-89aa-4619-8d50-e57fb083d8f9",
  "createdAt": "2019-11-01T13:55:41.8804663Z",
  "lastModifiedAt": "2019-11-01T13:55:41.8804663Z",
  "geometry": {
    "type": "Polygon",
    "coordinates": [
      [
        [
          78.34252251428694,
          17.402556352862675
        ],
        [
          78.34278000635095,
          17.407920852463022
        ],
        [
          78.34883106989963,
          17.40878079576528
        ],
        [
          78.3498181228054,
          17.403539173730607
        ],
        [
          78.34805859369493,
          17.39977166504127
        ],
        [
          78.34252251428694,
          17.402556352862675
        ]
      ]
    ]
  },
  "name": "Samplefarm",
  "properties": {
    "crops": "Corn",
    "address": "Sample street"
  }
}
 ```

### <a name="step-3-create-a-scene-id-post-call"></a>Étape 3 : Créer un ID de scène (appel POST)

Créez une scène (fichier .tiff ou .csv) avec les informations données qui fournissent la date, la séquence et l’ID de ferme auxquels la scène est associée. Les métadonnées associées à la scène peuvent être définies dans les propriétés, notamment la durée et le type de mesure.

La création d’une scène crée un ID de scène qui est associé à la ferme. Une fois l’ID de scène créé, l’utilisateur peut s’en servir pour créer un fichier (.tiff ou .csv) et stocker le contenu du fichier.

Exemple de charge utile d’entrée pour l’appel POST sur l’API /Scene :

```json
{
  "sceneId": "a0505928-c480-491b-ba31-d38285a28c1d",
  "type": "newtype",
  "contentType": "image/tiff",
  "name": "test scene file",
  "description": "test scene file description",
  "properties": {
    "additionalProp1": {},
    "additionalProp2": {},
    "additionalProp3": {}
  }
}
```

Réponse de l’API :

```json
{
  "id": "a0505928-c480-491b-ba31-d38285a28c1d",
  "createdAt": "2019-10-04T16:19:12.4838584Z",
  "lastModifiedAt": "2019-10-04T16:19:12.4838584Z",
  "type": "new type",
  "source": "SlantRange-3P",
  "farmId": "d41a33e7-b73e-480e-9279-0fcb3207332b",
  "date": "2019-10-04T16:13:39.064Z",
  "sequence": 5,
  "name": "test scene",
  "description": "test scene description",
  "properties": {}
}

```

**Créer un fichier de scène**

L’ID de scène retourné à l’étape 3 est l’entrée pour le fichier de scène. Le fichier de scène retourne un jeton d’URL SAS valide pendant 24 heures.

Si l’utilisateur a besoin d’une méthode programmatique pour charger un flux d’images, le SDK de stockage d’objets blob peut être utilisé pour définir une méthode avec l’ID, l’emplacement et l’URL du fichier de scène.

Exemple de charge utile d’entrée pour l’appel POST sur l’API /SceneFile :

```json
{
  "sceneId": "a0505928-c480-491b-ba31-d38285a28c1d",
  "type": "newtype",
  "contentType": "image/tiff",
  "name": "test scene file",
  "description": "test scene file description",
  "properties": {
    "additionalProp1": {},
    "additionalProp2": {},
    "additionalProp3": {}
  }
}
```
Réponse de l’API :

```json
{
  "uploadSASUrl": "https://storagej2lho.blob.core.windows.net/farm-scene/2019/a0505928-c480-491b-ba31-d38285a28c1d/e91139a7-4ebd-4e2f-b17c-c677822dc840?sv=2018-03-28&sr=b&sig=%2F1426JkDcIFE5g3d%2BjOevCVMIn%2FJo9YKwBn3La5zL8Y%3D&se=2019-10-05T16%3A23%3A57Z&sp=w",
  "id": "e91139a7-4ebd-4e2f-b17c-c677822dc840",
  "createdAt": "2019-10-04T16:23:57.1192916Z",
  "lastModifiedAt": "2019-10-04T16:23:57.1192916Z",
  "blobUrl": "https://storagej2lho.blob.core.windows.net/farm-scene/2019/a0505928-c480-491b-ba31-d38285a28c1d/e91139a7-4ebd-4e2f-b17c-c677822dc840",
  "sceneId": "a0505928-c480-491b-ba31-d38285a28c1d",
  "type": "newtype",
  "contentType": "image/tiff",
  "name": "test scene file",
  "description": "test scene file description",
  "properties": {}
}


```

L’appel POST à l’API /SceneFile retourne une URL de chargement SAS qui peut être utilisée pour charger le fichier .csv ou .tiff à l’aide du client ou de la bibliothèque de stockage d’objets blob Azure.


## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’intégration basée sur l’API REST, consultez [API REST](rest-api-in-azure-farmbeats.md).