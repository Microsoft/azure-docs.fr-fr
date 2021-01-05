---
title: Ingérer des données de télémétrie historiques
description: Cet article décrit comment ingérer des données de télémétrie historiques.
author: uhabiba04
ms.topic: article
ms.date: 11/04/2019
ms.author: v-umha
ms.custom: has-adal-ref
ms.openlocfilehash: 603f14d2076b5b74dde0b92a732f8fe816f6dd10
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97656782"
---
# <a name="ingest-historical-telemetry-data"></a>Ingérer des données de télémétrie historiques

Cet article explique comment ingérer des données de capteur historiques dans Azure FarmBeats.

L’ingestion de données historiques à partir de ressources IoT (Internet des objets) telles que des appareils et des capteurs est un scénario courant dans FarmBeats. Vous créez des métadonnées pour les appareils et les capteurs, puis vous ingérez les données historiques dans FarmBeats dans un format canonique.

## <a name="before-you-begin"></a>Avant de commencer

Avant de poursuivre cet article, vérifiez que vous avez installé FarmBeats et collecté des données historiques à partir de vos appareils IoT. Vous devez également activer l’accès partenaire comme indiqué dans les étapes suivantes.

## <a name="enable-partner-access"></a>Activer l’accès partenaire

Vous devez activer l’intégration des partenaires à votre instance Azure FarmBeats. Cette étape consiste à créer un client qui a accès à votre instance Azure FarmBeats en tant que partenaire d’appareil. Elle vous permet également d’obtenir les valeurs ci-après qui sont exigées pour les étapes suivantes :

- Point de terminaison d’API : URL du hub de données, par exemple https://\<datahub>.azurewebsites.net
- ID client
- ID client
- Clé secrète client
- Chaîne de connexion du hub d’événements

Procédez comme suit :

> [!NOTE]
> Pour effectuer les étapes suivantes, vous devez être administrateur.

1. Connectez-vous à https://portal.azure.com/.

2. **Si vous êtes sur FarmBeats version 1.2.7 ou ultérieure, ignorez les étapes a, b et c et passez à l’étape 3.** . Vous pouvez vérifier la version de FarmBeats en sélectionnant l’icône **Paramètres** dans le coin supérieur droit de l’interface utilisateur de FarmBeats.

      a.  Accédez à **Azure Active Directory** > **Inscriptions des applications**.

      b. Sélectionnez l’**inscription d’application** qui a été créée dans le cadre de votre déploiement FarmBeats. Elle aura le même nom que votre hub de données FarmBeats.

      c. Sélectionnez **Exposer une API**, sélectionnez **Ajouter une application cliente**, entrez **04b07795-8ddb-461a-bbee-02f9e1bf7b46**, puis cochez **Autoriser l’étendue**. Cela permet d’accéder à l’interface de ligne de commande Azure (Cloud Shell) pour effectuer les étapes ci-dessous :

3. Ouvrez Cloud Shell. Cette option est disponible dans la barre d’outils située en haut à droite du portail Azure.

    ![Barre d’outils du portail Azure](./media/get-drone-imagery-from-drone-partner/navigation-bar-1.png)

4. Vérifiez que l’environnement est défini sur **PowerShell**. Bash est sélectionné par défaut.

    ![Paramètre de la barre d’outils PowerShell](./media/get-sensor-data-from-sensor-partner/power-shell-new-1.png)

5. Accédez à votre répertoire de base.

    ```azurepowershell-interactive
    cd
    ```

6. Exécutez la commande suivante : Cela permet de connecter un compte authentifié et d’utiliser les requêtes Azure AD.

    ```azurepowershell-interactive
    Connect-AzureAD
    ```

7. Exécutez la commande suivante : Cela permet de télécharger un script dans votre répertoire de base.

    ```azurepowershell-interactive 

    wget –q https://aka.ms/farmbeatspartnerscriptv3 -O ./generatePartnerCredentials.ps1

    ```

8. Exécutez le script suivant. Le script invite à fournir l’ID de locataire, qui est disponible dans **Azure Active Directory** > page **Vue d’ensemble**.

    ```azurepowershell-interactive

    ./generatePartnerCredentials.ps1

    ```

9. Suivez les instructions à l’écran pour capturer les valeurs suivantes : **API Endpoint** (Point de terminaison d’API), **Tenant ID** (ID de locataire), **Client ID** (ID client), **Client Secret** (Secret client) et **EventHub Connection String** (Chaîne de connexion du hub d’événements).


## <a name="create-device-or-sensor-metadata"></a>Créer des métadonnées d’appareils ou de capteurs

 Maintenant que vous disposez des informations d’identification nécessaires, vous pouvez définir les appareils et les capteurs. Pour cela, créez les métadonnées en appelant les API FarmBeats. Veillez à appeler les API en tant qu’application cliente créée dans la section ci-dessus.

 FarmBeats Datahub propose les API suivantes qui permettent de créer et de gérer les métadonnées d’appareils ou de capteurs.

 > [!NOTE]
 > En tant que partenaire, vous avez uniquement accès aux option de lecture, de création et de mise à jour des métadonnées. **L’option de suppression est interdite au partenaire.**

- /**DeviceModel** : DeviceModel correspond aux métadonnées de l’appareil, telles que le fabricant et le type d’appareil (passerelle ou nœud).
- /**Device** : Device correspond à un appareil physique présent dans l’exploitation agricole.
- /**SensorModel** : SensorModel correspond aux métadonnées du capteur, telles que le fabricant, le type de capteur (analogique ou numérique) et la mesure effectuée par le capteur (température ambiante, pression, etc.).
- /**Sensor** : Sensor correspond à un capteur physique qui enregistre des valeurs. Un capteur est généralement connecté à un appareil avec une identité d’appareil.

| DeviceModel | Suggestions |
|--|--|
| Type (nœud, passerelle) | Type de l’appareil (nœud ou passerelle) |
| Fabricant | Nom du fabricant |
| ProductCode | Code produit de l’appareil ou nom ou numéro du modèle. Par exemple, EnviroMonitor#6800. |
| Ports | Nom et type du port (numérique ou analogique). |
| Nom | Nom destiné à identifier la ressource. Par exemple, le nom du modèle ou du produit. |
| Description | Description explicite du modèle. |
| Propriétés | Propriétés supplémentaires fournies par le fabricant. |
| **Appareil** |  |
| DeviceModelId | ID du modèle d’appareil associé. |
| HardwareId | ID unique de l’appareil, par exemple l’adresse MAC. |
| ReportingInterval | Intervalle de rapport en secondes. |
| Emplacement | Latitude (-90 à +90), longitude (-180 à 180) et élévation (en mètres) de l’appareil. |
| ParentDeviceId | ID de l’appareil parent auquel cet appareil est connecté. Par exemple, un nœud connecté à une passerelle. Pour un nœud, le parentDeviceId est l’ID d’une passerelle. |
| Nom | Nom destiné à identifier la ressource. Les partenaires d’appareil doivent envoyer un nom cohérent avec celui de l’appareil côté partenaire. Si le nom de l’appareil partenaire est défini par l’utilisateur, ce nom doit être propagé sur FarmBeats. |
| Description | Description explicite. |
| Propriétés | Propriétés supplémentaires fournies par le fabricant. |
| **SensorModel** |  |
| Type (analogique, numérique) | Type de capteur (analogique ou numérique). |
| Fabricant | Fabricant du capteur. |
| ProductCode | Code produit ou nom ou numéro du modèle. Par exemple, RS-CO2-N01. |
| SensorMeasures > Name | Nom de la mesure du capteur. Seules les minuscules sont prises en charge. Pour des mesures effectuées à différentes profondeurs, spécifiez la profondeur. Par exemple, humidité_sol_15cm. Ce nom doit être cohérent avec les données de télémétrie. |
| SensorMeasures > DataType | Type de données de télémétrie. À l’heure actuelle, le type double est pris en charge. |
| SensorMeasures > Type | Type de mesure des données de télémétrie de capteur. Voici les types définis par le système : AmbientTemperature, CO2, Depth, ElectricalConductivity, LeafWetness, Length, LiquidLevel, Nitrate, O2, PH, Phosphate, PointInTime, Potassium, Pressure, RainGauge, RelativeHumidity, Salinity, SoilMoisture, SoilTemperature, SolarRadiation, State, TimeDuration, UVRadiation, UVIndex, Volume, WindDirection, WindRun, WindSpeed, Evapotranspiration, PAR. Pour en ajouter d’autres, reportez-vous à l’API /ExtendedType. |
| SensorMeasures > Unit | Unité des données de télémétrie du capteur. Voici les unités définies par le système : NoUnit, Celsius, Fahrenheit, Kelvin, Rankine, Pascal, Mercury, PSI, MilliMeter, CentiMeter, Meter, Inch, Feet, Mile, KiloMeter, MilesPerHour, MilesPerSecond, KMPerHour, KMPerSecond, MetersPerHour, MetersPerSecond, Degree, WattsPerSquareMeter, KiloWattsPerSquareMeter, MilliWattsPerSquareCentiMeter, MilliJoulesPerSquareCentiMeter, VolumetricWaterContent, Percentage, PartsPerMillion, MicroMol, MicroMolesPerLiter, SiemensPerSquareMeterPerMole, MilliSiemensPerCentiMeter, Centibar, DeciSiemensPerMeter, KiloPascal, VolumetricIonContent, Liter, MilliLiter, Seconds, UnixTimestamp, MicroMolPerMeterSquaredPerSecond, InchesPerHour. Pour en ajouter d’autres, reportez-vous à la documentation sur l’API /ExtendedType. |
| SensorMeasures > AggregationType | Les valeurs peuvent être : none, average, maximum, minimum ou StandardDeviation. |
| Nom | Nom destiné à identifier la ressource. Par exemple, le nom du modèle ou du produit. |
| Description | Description explicite du modèle. |
| Propriétés | Propriétés supplémentaires fournies par le fabricant. |
| **Capteur** |  |
| HardwareId | ID unique du capteur défini par le fabricant. |
| SensorModelId | ID du modèle de capteur associé. |
| Emplacement | Latitude (-90 à +90), longitude (-180 à 180) et élévation (en mètres) du capteur. |
| Port > Name | Nom et type du port auquel le capteur est connecté sur l’appareil. Ce nom doit être identique à celui défini dans le modèle d’appareil. |
| DeviceID | ID de l’appareil auquel le capteur est connecté. |
| Nom | Nom destiné à identifier la ressource. Par exemple, nom du capteur ou nom du produit et numéro de modèle ou code produit. |
| Description | Description explicite. |
| Propriétés | Propriétés supplémentaires fournies par le fabricant. |

Pour plus d’informations sur les objets, consultez [Swagger](https://aka.ms/FarmBeatsDatahubSwagger).

### <a name="api-request-to-create-metadata"></a>Requête d’API pour la création de métadonnées

Pour effectuer une requête d’API, vous devez associer la méthode HTTP (POST), l’URL du service d’API et l’URI d’une ressource (pour l’interrogation, l’envoi des données, la création ou la suppression d’une requête). Vous ajoutez ensuite un ou plusieurs en-têtes de requête HTTP. L’URL du service d’API est le point de terminaison d’API, c’est-à-dire l’URL du hub de données (https://\<yourdatahub>.azurewebsites.net).

### <a name="authentication"></a>Authentification

FarmBeats Datahub utilise l’authentification de porteur, qui nécessite les informations d’identification suivantes (générées dans la section précédente) :

- ID client
- Clé secrète client
- ID client

À l’aide de ces informations d’identification, l’appelant peut demander un jeton d’accès. Le jeton doit être envoyé dans les requêtes d’API suivantes, dans la section d’en-tête, comme suit :

```
headers = *{"Authorization": "Bearer " + access_token, …}*
```

L’exemple de code Python ci-après donne le jeton d’accès, qui peut être utilisé pour les appels d’API suivants à FarmBeats : 

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

**En-têtes de requête HTTP**

Voici les en-têtes de requête les plus courants qui doivent être spécifiés lors d’un appel d’API à FarmBeats Datahub :

- **Content-Type**: application/json
- **Autorisation** : Porteur <jeton-accès>
- **Accept**: application/json

### <a name="input-payload-to-create-metadata"></a>Charge utile d’entrée pour la création des métadonnées

DeviceModel


```json
{
  "type": "Node",
  "manufacturer": "string",
  "productCode": "string",
  "ports": [
    {
      "name": "string",
      "type": "Analog"
    }
  ],
  "name": "string",
  "description": "string",
  "properties": {
    "additionalProp1": {},
    "additionalProp2": {},
    "additionalProp3": {}
  }
}
```

Appareil

```json
{
  "deviceModelId": "string",
  "hardwareId": "string",
  "farmId": "string",
  "reportingInterval": 0,
  "location": {
    "latitude": 0,
    "longitude": 0,
    "elevation": 0
  },
  "parentDeviceId": "string",
  "name": "string",
  "description": "string",
  "properties": {
    "additionalProp1": {},
    "additionalProp2": {},
    "additionalProp3": {}
  }
}
```

SensorModel

```json
{
  "type": "Analog",
  "manufacturer": "string",
  "productCode": "string",
  "sensorMeasures": [
    {
      "name": "string",
      "dataType": "Double",
      "type": "string",
      "unit": "string",
      "aggregationType": "None",
      "depth": 0,
      "description": "string"
    }
  ],
  "name": "string",
  "description": "string",
  "properties": {
    "additionalProp1": {},
    "additionalProp2": {},
    "additionalProp3": {}
  }
}

```
Capteur

```json
{
  "hardwareId": "string",
  "sensorModelId": "string",
  "location": {
    "latitude": 0,
    "longitude": 0,
    "elevation": 0
  },
  "depth": 0,
  "port": {
    "name": "string",
    "type": "Analog"
  },
  "deviceId": "string",
  "name": "string",
  "description": "string",
  "properties": {
    "additionalProp1": {},
    "additionalProp2": {},
    "additionalProp3": {}
  }
}
```

L’exemple de requête suivant crée un appareil. Cette requête comporte du code JSON d’entrée en tant que charge utile avec le corps de la requête.

```bash
curl -X POST "https://<datahub>.azurewebsites.net/Device" -H
"accept: application/json" -H  "Content-Type: application/json" -H
"Authorization: Bearer <Access-Token>" -d "{  \"deviceModelId\": \"ID123\",  \"hardwareId\": \"MHDN123\",
\"reportingInterval\": 900,  \"name\": \"Device123\",
\"description\": \"Test Device 123\"}" *
```

Vous trouverez ci-dessous un exemple de code dans Python. Le jeton d’accès utilisé dans cet exemple est le même que celui reçu pendant l’authentification.

```python
import requests
import json

# Got access token - Calling the Device Model API

headers = {
    "Authorization": "Bearer " + access_token,
    "Content-Type" : "application/json"
    }
payload = '{"type" : "Node", "productCode" : "TestCode", "ports": [{"name": "port1","type": "Analog"}], "name" : "DummyDevice"}'
response = requests.post(ENDPOINT + "/DeviceModel", data=payload, headers=headers)
```

> [!NOTE]
> Les API retournent des ID uniques pour chaque instance créée. Vous devez conserver les ID pour envoyer des messages de télémétrie correspondants.

### <a name="send-telemetry"></a>Envoyer des données de télémétrie

Maintenant que vous avez créé les appareils et les capteurs dans FarmBeats, vous pouvez envoyer les messages de télémétrie associés.

### <a name="create-a-telemetry-client"></a>Créer un client de télémétrie

Vous devez envoyer les données de télémétrie à Azure Event Hubs pour traitement. Azure Event Hubs est un service assurant l’ingestion de données (de télémétrie) en temps réel à partir des appareils et applications connectés. Pour envoyer des données de télémétrie à FarmBeats, créez un client qui envoie des messages à un Event Hub dans FarmBeats. Pour plus d’informations sur l’envoi de données de télémétrie, consultez [Azure Event Hubs](../../event-hubs/event-hubs-dotnet-standard-getstarted-send.md).

### <a name="send-a-telemetry-message-as-the-client"></a>Envoyer un message de télémétrie comme client

Après avoir établi une connexion en tant que client Event Hubs, vous pouvez envoyer des messages au hub d’événements sous forme de code JSON.

Voici un exemple de code Python qui envoie des données de télémétrie en tant que client à un hub d’événements spécifié :

```python
import azure
from azure.eventhub import EventHubClient, Sender, EventData, Receiver, Offset
EVENTHUBCONNECTIONSTRING = "<EventHub Connection String provided by customer>"
EVENTHUBNAME = "<EventHub Name provided by customer>"

write_client = EventHubClient.from_connection_string(EVENTHUBCONNECTIONSTRING, eventhub=EVENTHUBNAME, debug=False)
sender = write_client.add_sender(partition="0")
write_client.run()
for i in range(5):
    telemetry = "<Canonical Telemetry message>"
    print("Sending telemetry: " + telemetry)
    sender.send(EventData(telemetry))
write_client.stop()

```

Convertissez les données de capteur historiques dans un format canonique compréhensible par Azure FarmBeats. Le format de message canonique se présente comme suit :

```json
{
"deviceid": "<id of the Device created>",
"timestamp": "<timestamp in ISO 8601 format>",
"version" : "1",
"sensors": [
    {
      "id": "<id of the sensor created>",
      "sensordata": [
        {
          "timestamp": "< timestamp in ISO 8601 format >",
          "<sensor measure name (as defined in the Sensor Model)>": <value>
        },
        {
          "timestamp": "<timestamp in ISO 8601 format>",
          "<sensor measure name (as defined in the Sensor Model)>": <value>
        }
      ]
    }
 ]
}
```

Après avoir ajouté les appareils et capteurs correspondants, récupérez les ID d’appareil et de capteur dans le message de télémétrie comme décrit dans la section précédente.

Voici un exemple de message de télémétrie :


 ```json
{
  "deviceid": "7f9b4b92-ba45-4a1d-a6ae-c6eda3a5bd12",
  "timestamp": "2019-06-22T06:55:02.7279559Z",
  "version": "1",
  "sensors": [
    {
      "id": "d8e7beb4-72de-4eed-9e00-45e09043a0b3",
      "sensordata": [
        {
          "timestamp": "2019-06-22T06:55:02.7279559Z",
          "hum_max": 15
        },
        {
          "timestamp": "2019-06-22T06:55:02.7279559Z",
          "hum_min": 42
        }
      ]
    },
    {
      "id": "d8e7beb4-72de-4eed-9e00-45e09043a0b3",
      "sensordata": [
        {
          "timestamp": "2019-06-22T06:55:02.7279559Z",
          "hum_max": 20
        },
        {
          "timestamp": "2019-06-22T06:55:02.7279559Z",
          "hum_min": 89
        }
      ]
    }
  ]
}
```

## <a name="troubleshooting"></a>Dépannage

### <a name="cant-view-telemetry-data-after-ingesting-historicalstreaming-data-from-your-sensors"></a>Impossible d’afficher les données de télémétrie après l’ingestion des données historique/de diffusion en continu à partir de vos capteurs

**Symptôme** : Des appareils ou des capteurs sont déployés. Vous avez créé les appareils/capteurs sur FarmBeats et ingéré la télémétrie vers l’EventHub, mais vous ne pouvez pas obtenir ni visualiser les données de télémétrie sur FarmBeats.

**Action corrective** :

1. Assurez-vous d’avoir effectué l’inscription du partenaire appropriée. Pour le vérifier, accédez à votre Swagger DataHub et à /Partner API, entrez la commande GET et regardez si le partenaire est bien inscrit. Si ce n’est pas le cas, suivez les [étapes indiquées ici](get-sensor-data-from-sensor-partner.md#enable-device-integration-with-farmbeats) pour ajouter un partenaire.

2. Assurez-vous que vous avez créé les métadonnées (DeviceModel, Appareil, SensorModel, Capteur) à l’aide des informations d’identification du client partenaire.

3. Vérifiez que vous avez bien utilisé le bon format de message de télémétrie (comme indiqué ci-dessous) :

```json
{
"deviceid": "<id of the Device created>",
"timestamp": "<timestamp in ISO 8601 format>",
"version" : "1",
"sensors": [
    {
      "id": "<id of the sensor created>",
      "sensordata": [
        {
          "timestamp": "< timestamp in ISO 8601 format >",
          "<sensor measure name (as defined in the Sensor Model)>": <value>
        },
        {
          "timestamp": "<timestamp in ISO 8601 format>",
          "<sensor measure name (as defined in the Sensor Model)>": <value>
        }
      ]
    }
 ]
}
```

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’intégration basée sur l’API REST, consultez [API REST](rest-api-in-azure-farmbeats.md).