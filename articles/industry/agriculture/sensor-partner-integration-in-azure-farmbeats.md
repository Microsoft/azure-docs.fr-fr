---
title: Intégration de partenaire de capteur
description: Cet article décrit l’intégration de partenaires de capteur.
author: uhabiba04
ms.topic: article
ms.date: 11/04/2019
ms.author: v-umha
ms.openlocfilehash: ef74c4b799c3a24636f88a8e704bf726104b034f
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96001590"
---
# <a name="sensor-partner-integration"></a>Intégration de partenaire de capteur

Cet article fournit des informations sur le composant Azure FarmBeats **Translator**, qui permet l'intégration de partenaire de capteur.

À l’aide de ce composant, les partenaires peuvent s’intégrer à FarmBeats en utilisant les API FarmBeats Datahub et envoyer les données de télémétrie et les données d’appareils des clients à FarmBeats Datahub. Une fois que les données sont disponibles dans FarmBeats, il est possible de les visualiser avec l’accélérateur FarmBeats et de s’en servir pour fusionner des données et générer des modèles de Machine Learning et d’intelligence artificielle.

## <a name="before-you-start"></a>Avant de commencer

Pour développer le composant Translator, vous aurez besoin des informations d’identification suivantes pour accéder aux API FarmBeats.

- Point de terminaison d’API
- ID client
- ID client
- Clé secrète client
- Chaîne de connexion du hub d’événements

Consultez cette section pour obtenir les informations d’identification précédentes : [Permettre l’intégration d’appareils](get-sensor-data-from-sensor-partner.md#enable-device-integration-with-farmbeats)

## <a name="translator-development"></a>Développement de Translator

**Intégration basée sur l’API REST**

Les fonctionnalités d’intégration des données de capteur de FarmBeats sont exposées via l’API REST. Les fonctionnalités incluent la définition des métadonnées, l’approvisionnement de l’appareil et du capteur ainsi que la gestion de l’appareil et du capteur.

**Ingestion de données de télémétrie**

Les données de télémétrie sont mappées à un message canonique publié sur Azure Event Hubs pour traitement. Azure Event Hubs est un service assurant l’ingestion de données (de télémétrie) en temps réel à partir des appareils et applications connectés.

**Développement d’API**

Les API incluent une documentation technique Swagger. Pour plus d’informations sur les API et leurs demandes ou réponses correspondantes, consultez [Swagger](https://aka.ms/FarmBeatsSwagger).

**Authentification**

FarmBeats utilise l’authentification Microsoft Azure Active Directory. Azure App Service offre une prise en charge intégrée de l’authentification et de l’autorisation.

Pour plus d’informations, consultez [Azure Active Directory](../../app-service/overview-authentication-authorization.md).

FarmBeats Datahub utilise l’authentification du porteur, qui nécessite les informations d’identification suivantes :
   - ID client
   - Clé secrète client
   - ID client

À l’aide de ces informations d’identification, l’appelant peut demander un jeton d’accès. Le jeton doit être envoyé dans les demandes d’API suivantes, dans la section d’en-tête, comme suit :

```
headers = {"Authorization": "Bearer " + access_token, …} 
```

L’exemple de code Python ci-après donne le jeton d’accès, qui peut être utilisé pour les appels d’API suivants à FarmBeats.

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

Voici les en-têtes de demande les plus courants qui doivent être spécifiés lors d’un appel d’API à FarmBeats Datahub.


**En-tête** | **Description et exemple**
--- | ---
Content-Type | Format de la demande (Content-Type: application/<format>). Pour les API FarmBeats Datahub, le format est JSON. Content-Type: application/json
Autorisation | Spécifie le jeton d’accès requis pour effectuer un appel d’API. Autorisation : Porteur <jeton-accès>
Acceptation | Format de la réponse. Pour les API FarmBeats Datahub, le format est JSON. Accept: application/json

**Requêtes d’API**

Pour effectuer une demande d’API REST, vous associez la méthode HTTP (GET, POST ou PUT), l’URL du service API, l’URI (Uniform Resource Identifier) d’une ressource à interroger, vers laquelle envoyer des données, à mettre à jour ou à supprimer et un ou plusieurs en-têtes de requête HTTP. L’URL du service API est le point de terminaison API que vous fournissez. Voici un exemple : https://\<yourdatahub-website-name>.azurewebsites.net

Si vous le souhaitez, vous pouvez inclure des paramètres de requête à des appels GET pour filtrer les données dans les réponses, en limiter la taille et les trier.

L’exemple de demande suivant porte sur l’obtention de la liste des appareils.

```bash
curl -X GET "https://microsoft-farmbeats.azurewebsites.net/Device" -H "Content-Type: application/json" -H "Authorization: Bearer <Access-Token>"
```
La plupart des appels GET, POST et PUT nécessitent un corps de demande JSON.

L’exemple de demande suivant porte sur la création d’un appareil. (Cet exemple contient un code JSON d’entrée avec le corps de la demande.)

```bash
curl -X POST "https://microsoft-farmbeats.azurewebsites.net/Device" -H  "accept: application/json" -H  "Content-Type: application/json" -H "Authorization: Bearer <Access-Token>" -d "{  \"deviceModelId\": \"ID123\",  \"hardwareId\": \"MHDN123\",  \"reportingInterval\": 900,  \"name\": \"Device123\",  \"description\": \"Test Device 123\",}"
```

## <a name="data-format"></a>Format de données

JSON est un format de données courant, indépendant du langage, qui fournit une représentation textuelle simple de structures de données arbitraires. Pour plus d’informations, consultez [json.org](http://json.org).

## <a name="metadata-specifications"></a>Spécifications de métadonnées

FarmBeats Datahub offre les API suivantes, qui permettent aux partenaires d’appareil de créer et de gérer des métadonnées d’appareil ou de capteur.

- /**DeviceModel** : DeviceModel correspond aux métadonnées de l’appareil, telles que le fabricant et le type d’appareil (passerelle ou nœud).
- /**Device** : Device correspond à un appareil physique présent dans l’exploitation agricole.
- /**SensorModel** : SensorModel correspond aux métadonnées du capteur, telles que le fabricant, le type de capteur (analogique ou numérique) et la mesure effectuée par le capteur (température ambiante, pression, etc.).
- /**Sensor** : Sensor correspond à un capteur physique qui enregistre des valeurs. Un capteur est généralement connecté à un appareil avec une identité d’appareil.

  DeviceModel | Description |
  --- | ---
  Type (nœud, passerelle)  | Type de l’appareil – Nœud ou passerelle |
  Fabricant  | Nom du fabricant |
  ProductCode  | Code produit de l’appareil ou nom ou numéro du modèle. Par exemple, EnviroMonitor#6800. |
  Ports  | Nom et type du port (numérique ou analogique).  |
  Nom  | Nom destiné à identifier la ressource. Par exemple, le nom du modèle ou du produit. |
  Description  | Description explicite du modèle. |
  Propriétés  | Propriétés supplémentaires fournies par le fabricant. |
  **Appareil** | **Description** |
  DeviceModelId  |ID du modèle d’appareil associé. |
  HardwareId   |ID unique de l’appareil, par exemple l’adresse MAC.  |
  ReportingInterval |Intervalle de rapport en secondes. |
  Emplacement    |Latitude (-90 à +90), longitude (-180 à 180) et élévation (en mètres) de l’appareil. |
  ParentDeviceId | ID de l’appareil parent auquel cet appareil est connecté. Par exemple, si un nœud est connecté à une passerelle, le nœud a parentDeviceID comme passerelle. |
  Nom  | Nom destiné à identifier la ressource. Les partenaires d’appareil doivent envoyer un nom correspondant à celui de l’appareil côté partenaire. Si le nom de l’appareil est défini par l’utilisateur côté partenaire, ce nom doit être propagé sur FarmBeats.  |
  Description  | Description explicite.  |
  Propriétés  |Propriétés supplémentaires fournies par le fabricant.  |
  **SensorModel** | **Description** |
  Type (analogique, numérique)  |Mentionne le type de capteur (analogique ou numérique).|
  Fabricant  | Nom du fabricant. |
  ProductCode  | Code produit ou nom ou numéro du modèle. Par exemple, RS-CO2-N01.  |
  SensorMeasures > Name  | Nom de la mesure du capteur. Seules les minuscules sont prises en charge. Pour des mesures effectuées à différentes profondeurs, spécifiez la profondeur. Par exemple, humidité_sol_15cm. Ce nom doit être cohérent avec les données de télémétrie. |
  SensorMeasures > DataType  | Type de données de télémétrie. À l’heure actuelle, le type double est pris en charge. |
  SensorMeasures > Type  | Type de mesure des données de télémétrie de capteur. Voici les types définis par le système : AmbientTemperature, CO2, Depth, ElectricalConductivity, LeafWetness, Length, LiquidLevel, Nitrate, O2, PH, Phosphate, PointInTime, Potassium, Pressure, RainGauge, RelativeHumidity, Salinity, SoilMoisture, SoilTemperature, SolarRadiation, State, TimeDuration, UVRadiation, UVIndex, Volume, WindDirection, WindRun, WindSpeed, Evapotranspiration, PAR. Pour en ajouter d’autres, reportez-vous à l’API /ExtendedType.
  SensorMeasures > Unit | Unité des données de télémétrie du capteur. Voici les unités définies par le système : NoUnit, Celsius, Fahrenheit, Kelvin, Rankine, Pascal, Mercury, PSI, MilliMeter, CentiMeter, Meter, Inch, Feet, Mile, KiloMeter, MilesPerHour, MilesPerSecond, KMPerHour, KMPerSecond, MetersPerHour, MetersPerSecond, Degree, WattsPerSquareMeter, KiloWattsPerSquareMeter, MilliWattsPerSquareCentiMeter, MilliJoulesPerSquareCentiMeter, VolumetricWaterContent, Percentage, PartsPerMillion, MicroMol, MicroMolesPerLiter, SiemensPerSquareMeterPerMole, MilliSiemensPerCentiMeter, Centibar, DeciSiemensPerMeter, KiloPascal, VolumetricIonContent, Liter, MilliLiter, Seconds, UnixTimestamp, MicroMolPerMeterSquaredPerSecond et InchesPerHour. Pour en ajouter d’autres, reportez-vous à l’API /ExtendedType.
  SensorMeasures > AggregationType  | Peut être none, average, maximum, minimum ou StandardDeviation.
  SensorMeasures > Depth  | Profondeur du capteur en centimètres. Par exemple, la mesure de l’humidité à 10 cm dans le sol.
  SensorMeasures > Description  | Description explicite de la mesure.
  Nom  | Nom destiné à identifier la ressource. Par exemple, le nom du modèle ou du produit.
  Description  | Description explicite du modèle.
  Propriétés  | Propriétés supplémentaires fournies par le fabricant.
  **Capteur**  | **Description** |
  HardwareId  | ID unique du capteur défini par le fabricant.
  SensorModelId  | ID du modèle de capteur associé.
  Emplacement  | Latitude (-90 à +90), longitude (-180 à 180) et élévation (en mètres) du capteur.
  Port > Name  |Nom et type du port auquel le capteur est connecté sur l’appareil. Ce nom doit être identique à celui défini dans le modèle d’appareil.
  deviceId  | ID de l’appareil auquel le capteur est connecté.
  Nom  | Nom destiné à identifier la ressource. Par exemple, nom du capteur ou du produit et numéro de modèle ou code produit.
  Description  | Description explicite.
  Propriétés  | Propriétés supplémentaires fournies par le fabricant.

 Pour plus d’informations sur chacun des objets et leurs propriétés, consultez [Swagger](https://aka.ms/FarmBeatsDatahubSwagger).

 > [!NOTE]
 > Les API retournent des ID uniques pour chaque instance créée. Cet ID doit être conservé par le traducteur pour la gestion des périphériques et la synchronisation des métadonnées.


**Synchronisation des métadonnées**

Le traducteur doit envoyer des mises à jour sur les métadonnées. Par exemple, les scénarios de mise à jour sont la modification du nom de l’appareil ou du capteur et la modification de l’emplacement de l’appareil ou du capteur.

Le traducteur doit avoir la possibilité d’ajouter des appareils ou capteurs installés par l’utilisateur après la liaison de FarmBeats. De même, si un appareil ou capteur a été mis à jour par l’utilisateur, l’appareil ou capteur correspondant doit être mis à jour dans FarmBeats. Les scénarios classiques qui nécessitent la mise à jour d’un appareil ou d’un capteur correspondent à une modification de l’emplacement de l’appareil ou à l’ajout de capteurs dans un nœud.


> [!NOTE]
> La suppression n’est pas prise en charge pour les métadonnées d’appareil ou de capteur.
>
> Pour mettre à jour les métadonnées, il est obligatoire d’appeler /Get/{id} sur l’appareil ou le capteur, de mettre à jour les propriétés modifiées, puis d’effectuer une commande /Put/{id} afin que les propriétés définies par l’utilisateur ne soient pas perdues.

### <a name="add-new-types-and-units"></a>Ajouter de nouveaux types et unités

FarmBeats prend en charge l’ajout de nouveaux types et unités de mesure de capteur. Pour plus d’informations sur l’API /ExtendedType, consultez [Swagger](https://aka.ms/FarmBeatsSwagger).

## <a name="telemetry-specifications"></a>Spécifications des données de télémétrie

Les données de télémétrie sont mappées à un message canonique publié sur Azure Event Hubs pour traitement. Azure Event Hubs est un service assurant l’ingestion de données (de télémétrie) en temps réel à partir des appareils et applications connectés.

## <a name="send-telemetry-data-to-farmbeats"></a>Envoyer des données de télémétrie à FarmBeats

Pour envoyer des données de télémétrie à FarmBeats, créez un client qui envoie des messages à un Event Hub dans FarmBeats. Pour plus d’informations sur les données de télémétrie, consultez [Envoi de données de télémétrie à un Event Hub](../../event-hubs/event-hubs-dotnet-standard-getstarted-send.md).

Voici un exemple de code Python qui envoie des données de télémétrie en tant que client à un Event Hub spécifié.

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

Le format de message canonique se présente comme suit :

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
Tous les noms de clé dans le JSON de télémétrie doivent être en minuscules. Les noms deviceid et sensordata sont des exemples.

Par exemple, voici un message de télémétrie :


```json
{
  "deviceid": "7f9b4b92-ba45-4a1d-a6ae-c6eda3a5bd12",
  "timestamp": "2019-06-22T06:55:02.7279559Z",
  "version" : "1",
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

> [!NOTE]
> Les sections suivantes portent sur d’autres modifications (par exemple, interface utilisateur, gestion des erreurs, etc.) auxquelles le partenaire de capteur peut faire référence pendant le développement du composant Translator.


## <a name="link-a-farmbeats-account"></a>Lier un compte FarmBeats

Après avoir acheté et déployé des appareils ou des capteurs, les clients peuvent accéder aux données et à la télémétrie de ces appareils sur le portail SaaS (software as a service) des partenaires d’appareils. Les partenaires d’appareils peuvent permettre aux clients de lier leurs comptes à leurs instance FarmBeats sur Azure en offrant un moyen d’entrer les informations d’identification suivantes :

   - Nom d’affichage (champ facultatif permettant aux utilisateurs de définir un nom pour cette intégration)
   - Point de terminaison d’API
   - ID client
   - ID client
   - Clé secrète client
   - Chaîne de connexion du hub d’événements
   - Date de début

   > [!NOTE]
   > La date de début active le flux de données d’historique, autrement dit, les données à partir de la date spécifiée par l’utilisateur.

## <a name="unlink-farmbeats"></a>Dissocier FarmBeats

Les partenaires d’appareils peuvent permettre aux clients de dissocier une intégration FarmBeats existante. En principe, la dissociation de FarmBeats ne supprime pas les métadonnées d’appareils ou de capteurs qui ont été créées dans FarmBeats Datahub. La dissociation les opérations suivantes :

   - Arrête le workflow des données de télémétrie.
   - Supprime et efface les informations d’identification d’intégration sur le partenaire d’appareil.

## <a name="edit-farmbeats-integration"></a>Modifier l’intégration de FarmBeats

Les partenaires d’appareils peuvent autoriser les clients à modifier les paramètres d’intégration de FarmBeats si le secret client ou la chaîne de connexion change. Dans ce cas, seuls les champs suivants peuvent être modifiés :

   - Nom complet (le cas échéant)
   - Clé secrète client (doit être affichée dans le format « 2x8*********** » ou dans la fonctionnalité Afficher/masquer au lieu du texte en clair)
   - Chaîne de connexion (doit être affichée dans le format « 2x8*********** » ou dans la fonctionnalité Afficher/masquer au lieu du texte en clair)

## <a name="view-the-last-telemetry-sent"></a>Afficher les dernières données de télémétrie envoyées

Les partenaires d’appareils peuvent autoriser les clients à afficher le timestamp des dernières données de télémétrie envoyées, qui se trouvent sous **Telemetry Sent** (Données de télémétrie envoyées). Il s’agit de l’heure à laquelle les données de télémétrie les plus récentes ont été correctement envoyées à FarmBeats.

## <a name="troubleshooting-and-error-management"></a>Résolution des problèmes et gestion des erreurs

**Support ou option de résolution de problèmes**

Si le client ne peut pas recevoir des données de télémétrie ou d’appareils dans l’instance FarmBeats spécifiée, le partenaire d’appareils doit fournir un support technique et un mécanisme pour résoudre le problème.

**Conservation des données de télémétrie**

Les données de télémétrie doivent également être conservées pour une période prédéfinie, car elles sont utiles pour le débogage ou le renvoi de la télémétrie en cas d’erreur ou de perte de données.

**Gestion d’erreurs ou notification d’erreurs**

Si une erreur affecte les métadonnées d’appareils ou de capteurs ou l’intégration des données ou le flux de données de télémétrie dans le système du partenaire d’appareils, le client doit recevoir une notification. Un mécanisme permettant de résoudre les erreurs doit également être conçu et implémenté.

**Liste de contrôle de connexion**

Les fabricants ou partenaires d’appareils peuvent utiliser la check-list suivante pour s’assurer que les informations d’identification fournies par le client sont exactes :

   - Vérifier si un jeton d’accès est reçu avec les informations d’identification fournies.
   - Vérifier si un appel d’API a échoué avec le jeton d’accès reçu.
   - Vérifier si la connexion cliente EventHub est établie.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur l’API REST, consultez [API REST](rest-api-in-azure-farmbeats.md).