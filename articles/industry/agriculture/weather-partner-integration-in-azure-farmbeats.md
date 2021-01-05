---
title: Intégration de partenaire - Données météorologiques
description: Découvrez comment un fournisseur de données météorologiques peut s’intégrer à FarmBeats.
author: sunasing
ms.topic: article
ms.date: 07/09/2020
ms.author: sunasing
ms.openlocfilehash: f0fbd93e2a5f4e92089e10e75dc17e304ff80bf6
ms.sourcegitcommit: 4b76c284eb3d2b81b103430371a10abb912a83f4
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/01/2020
ms.locfileid: "93147077"
---
# <a name="weather-partner-integration-with-farmbeats"></a>Intégration de partenaire de données météorologiques avec FarmBeats

Cet article fournit des informations sur le composant Azure FarmBeats Connector Docker. En tant que fournisseur de données météorologiques, vous pouvez utiliser Connector Docker pour l’intégration à FarmBeats. Utilisez ses API pour envoyer des données météorologiques à FarmBeats. Dans FarmBeats, il est possible d’utiliser les données pour les fusionner et de générer des modèles de Machine Learning ou des modèles d’intelligence artificielle.

 > [!NOTE]
 > Dans cet article, nous utilisons une [implémentation de référence](https://github.com/azurefarmbeats/noaa_docker) qui a été créée à l’aide d’Azure Open Datasets et de données météorologiques de la National Oceanic and Atmospheric Administration (NOAA). Nous utilisons également l’[image Docker](https://hub.docker.com/r/azurefarmbeats/farmbeats-noaa) correspondante.

Vous devez fournir [une image ou un programme Docker approprié](#docker-specifications) et héberger l’image Docker dans un registre de conteneurs auquel les clients peuvent accéder. Fournissez les informations suivantes à vos clients :

- URL de l’image Docker
- Étiquette de l’image Docker
- Clés ou informations d’identification pour accéder à l’image Docker
- Clés ou informations d’identification d’API spécifiques au client pour accéder aux données à partir du système
- Détails de la référence SKU de la VM (fournissez ces détails si votre image Docker a des exigences spécifiques en matière de VM. Dans le cas contraire, les clients peuvent choisir parmi les références SKU de machines virtuelles prises en charge dans Azure.)

Les clients utilisent ces informations Docker pour inscrire un partenaire météorologique dans leur instance FarmBeats. Pour plus d’informations sur la façon dont les clients peuvent utiliser Docker pour ingérer des données météorologiques dans FarmBeats, consultez [Obtenir des données météorologiques auprès de partenaires météo](./get-weather-data-from-weather-partner.md).

## <a name="connector-docker-development"></a>Développement de Docker Connecteur

**Intégration basée sur l’API REST**

Les API FarmBeats incluent une documentation technique Swagger. Pour plus d’informations sur toutes les API et leurs demandes ou réponses correspondantes, consultez [FarmBeats Swagger](https://aka.ms/farmbeatsswagger). 

Si vous avez installé FarmBeats, accédez à votre FarmBeats Swagger à l’adresse `https://yourfarmbeatswebsitename-api.azurewebsites.net/swagger`

Notez que *-api* est ajouté au nom de votre site web FarmBeats. Le point de terminaison de l’API réside à l’adresse : `https://yourfarmbeatswebsitename-api.azurewebsites.net`

### <a name="datahub-lib"></a>Bibliothèque Datahub

FarmBeats fournit une bibliothèque que vous pouvez utiliser. La bibliothèque est actuellement disponible [dans le cadre de l’implémentation de référence](https://github.com/azurefarmbeats/noaa_docker/tree/master/datahub_lib). Ultérieurement, elle sera disponible en tant que Kit SDK pour plusieurs langues.

### <a name="authentication"></a>Authentification

**Authentification avec les API FarmBeats**

FarmBeats utilise l’authentification du porteur. Vous pouvez accéder aux API en fournissant un jeton d’accès dans la section d’en-tête de la requête. Voici un exemple :

```
headers = *{"Authorization": "Bearer " + access_token, …}*
```

Vous pouvez demander le jeton d’accès à partir d’une application Azure Functions qui s’exécute sur l’instance FarmBeats du client. L’URL Azure Functions est fournie au programme Docker en tant qu’argument. Vous pouvez obtenir le jeton d’accès en effectuant une requête `GET` sur l’URL. La réponse de l’URL contient le jeton d’accès. 

Utilisez les fonctions d’assistance de la bibliothèque DataHub pour obtenir le jeton d’accès. Pour plus d’informations, consultez la page [GitHub pour l’image Docker de la NOAA](https://github.com/azurefarmbeats/noaa_docker/blob/master/datahub_lib/auth/partner_auth_helper.py).

Le jeton d’accès est valide uniquement pendant quelques heures. Lorsqu’il expire, vous devez le demander à nouveau.

**Authentification avec les API côté partenaire**

Pour s’authentifier avec des API côté partenaire lors de l’exécution de Docker, les clients doivent fournir les informations d’identification lors de l’inscription du partenaire. Voici un exemple :

```json
{
 "partnerCredentials": {
   "key1": "value1",
   "key2": "value2"
   }
}
```
Le service d’API sérialise ce dictionnaire et le stocke dans un [coffre de clés](../../key-vault/general/basic-concepts.md).

[Azure Data Factory](../../data-factory/introduction.md) permet d’orchestrer des travaux météo. Il permet de faire tourner les ressources pour exécuter le code Docker. Data Factory fournit également un mécanisme pour envoyer (push) les données de façon sécurisée à la machine virtuelle où le travail de Docker s’exécute. Les informations d’identification de l’API sont ensuite stockées de façon sécurisée dans le coffre de clés. 

Les informations d’identification sont lues en tant que chaînes sécurisées à partir du coffre de clés. Elles sont fournies sous forme de propriétés étendues dans le répertoire de travail du conteneur Docker. Le chemin d’accès à ce fichier est */mnt/working_dir/activity.json*. 

Le code Docker peut lire les informations d’identification d’ *activity.json* au moment de l’exécution pour accéder aux API côté partenaire pour le client. Dans le fichier JSON, les informations d’identification se présentent comme dans l’exemple de code suivant :

```json
{ 
   "typeProperties" : {
      "extendedProperties" : { 
         "partnerCredentials": "" } 
   } 
}
```
Les informations d’identification de `partnerCredentials` sont disponibles conformément à ce que le client a fourni lors de l’inscription du partenaire.

La bibliothèque FarmBeats fournit des fonctions d’assistance. Utilisez ces fonctions pour lire les informations d’identification à partir des propriétés de l’activité. Pour plus d’informations, consultez la page [GitHub pour l’image Docker de la NOAA](https://github.com/azurefarmbeats/noaa_docker/blob/master/datahub_lib/auth/partner_adf_helper.py).

Le fichier est utilisé uniquement lors de l’exécution du code Docker. Une fois le code terminé, le fichier est supprimé.

Pour plus d’informations sur le fonctionnement des pipelines et des activités Data Factory, consultez [Mappage du schéma et du type de données](../../data-factory/copy-activity-schema-and-type-mapping.md).

**En-têtes de requête HTTP**

Le tableau suivant présente les en-têtes de demande les plus courants que vous devez spécifier lors d’un appel d’API à FarmBeats.

En-tête | Description et exemple
--- | ---
Content-Type | Format de la requête. Exemple : `Content-Type: application/<format>` <br/>Pour les API FarmBeats Datahub, le format est JSON. Exemple : ` Content-Type: application/json`
Autorisation | Jeton d’accès requis pour effectuer un appel d’API. Exemple : `Authorization: Bearer <Access-Token>`
Acceptation | Format de la réponse. Pour les API FarmBeats Datahub, le format est JSON. Exemple : `Accept: application/json`

## <a name="data-format"></a>Format de données

JSON est un format de données courant, indépendant du langage, qui fournit une représentation textuelle simple de structures de données arbitraires. Pour plus d’informations, consultez [JSON.org](https://json.org).

## <a name="docker-specifications"></a>Spécifications de Docker

Le programme Docker a besoin de deux composants : le démarrage et le travail. Le programme peut avoir plusieurs travaux.

### <a name="bootstrap"></a>Bootstrap

Le composant de démarrage doit s’exécuter lorsque le client démarre l’inscription Docker sur FarmBeats. Les arguments suivants (`arg1` et `arg2`) sont passés au programme :

- **Point de terminaison de l’API FarmBeats**  : Point de terminaison de l’API FarmBeats pour les demandes d’API. Ce point de terminaison effectue des appels d’API au déploiement FarmBeats.
- **URL Azure Functions**  : Votre point de terminaison. Cette URL fournit votre jeton d’accès pour les API FarmBeats. Vous pouvez appeler `GET` sur cette URL pour récupérer le jeton d’accès.

Le démarrage crée les métadonnées dont les utilisateurs ont besoin pour exécuter vos travaux afin d’extraire des données météorologiques. Pour plus d’informations, consultez l’[implémentation de référence](https://github.com/azurefarmbeats/noaa_docker). 

Si vous personnalisez le fichier *bootstrap_manifest.json* , le programme de démarrage de référence crée les métadonnées requises pour vous. Le programme de démarrage crée les métadonnées suivantes : 

 > [!NOTE]
 > Si vous mettez à jour le fichier *bootstrap_manifest.json* comme décrit dans l’[implémentation de référence](https://github.com/azurefarmbeats/noaa_docker), vous n’avez pas besoin de créer les métadonnées suivantes. Le programme de démarrage utilise votre fichier manifeste pour créer les métadonnées nécessaires.

- /**WeatherDataModel**  : Les métadonnées WeatherDataModel représentent des données météorologiques. Elles correspondent aux jeux de données fournis par la source. Par exemple, DailyForecastSimpleModel peut fournir des informations sur la température, l’humidité et les précipitations en moyenne une fois par jour. En revanche, DailyForecastAdvancedModel peut fournir bien plus d’informations avec une précision horaire. Vous pouvez créer autant de modèles de données météorologiques que nécessaire.
- /**JobType**  : FarmBeats dispose d’un système extensible de gestion des travaux. En tant que fournisseur de données météorologiques, vous disposez de plusieurs jeux de données et API (par exemple, GetDailyForecasts). Vous pouvez activer ces jeux de données et ces API dans FarmBeats à l’aide de JobType. Après avoir créé un type de travail, un client peut déclencher des travaux de ce type afin d’obtenir des données météorologiques pour son emplacement ou sa batterie de serveurs. Pour plus d’informations, consultez les API JobType et Job dans [FarmBeats Swagger](https://aka.ms/farmbeatsswagger).

### <a name="jobs"></a>travaux

Ce composant Jobs est appelé à chaque fois qu’un utilisateur de FarmBeats exécute un travail du /JobType que vous avez créé dans le cadre du processus de démarrage. La commande d’exécution de Docker pour le travail est définie dans le cadre du /JobType que vous avez créé.

Le travail extrait les données de la source et les transmet à FarmBeats. Durant le processus de démarrage, les paramètres requis pour l’extraction des données doivent être définis dans le cadre du /JobType.

Pour le travail, le programme doit créer un /WeatherDataLocation en fonction du /WeatherDataModel créé durant le processus de démarrage. /WeatherDataLocation correspond à un emplacement (coordonnées de latitude et de longitude) que l’utilisateur a fourni comme paramètre du travail.

### <a name="object-details"></a>Détails de l’objet

WeatherDataModel | Description |
--- | ---
Nom  | Nom du modèle de données météorologiques. |
Description  | Description explicite du modèle. |
Propriétés  | Propriétés supplémentaires définies par le fournisseur de données. |
weatherMeasures > Name  | Nom de la mesure météorologique. Par exemple, humidity_max. |
weatherMeasures > DataType  | Double ou Énum. Pour Énum, measureEnumDefinition est requis. |
weatherMeasures > measureEnumDefinition  | Obligatoire uniquement si DataType est égal à Énum. Par exemple, `{ "NoRain": 0, "Snow": 1, "Drizzle": 2, "Rain": 3 }` |
weatherMeasures > Type  | Type de données de télémétrie météorologiques. Par exemple, RelativeHumidity. Voici les types définis par le système : AmbientTemperature, NoUnit, CO2, Depth, ElectricalConductivity, LeafWetness, Length, LiquidLevel, Nitrate, O2, PH, Phosphate, PointInTime, Potassium, Pressure, RainGauge, RelativeHumidity, Salinity, SoilMoisture, SoilTemperature, SolarRadiation, State, TimeDuration, UVRadiation, UVIndex, Volume, WindDirection, WindRun, WindSpeed, Evapotranspiration et PAR. Pour ajouter d’autres types, reportez-vous à la section [Ajouter ExtendedType](#add-extendedtype) de cet article.
weatherMeasures > Unit | Unité des données de télémétrie météorologiques. Voici les types définis par le système : NoUnit, Celsius, Fahrenheit, Kelvin, Rankine, Pascal, Mercury, PSI, MilliMeter, CentiMeter, Meter, Inch, Feet, Mile, KiloMeter, MilesPerHour, MilesPerSecond, KMPerHour, KMPerSecond, MetersPerHour, MetersPerSecond, Degree, WattsPerSquareMeter, KiloWattsPerSquareMeter, MilliWattsPerSquareCentiMeter, MilliJoulesPerSquareCentiMeter, VolumetricWaterContent, Percentage, PartsPerMillion, MicroMole, MicroMolesPerLiter, SiemensPerSquareMeterPerMole, MilliSiemensPerCentiMeter, Centibar, DeciSiemensPerMeter, KiloPascal, VolumetricIonContent, Liter, MilliLiter, Seconds, UnixTimestamp, MicroMolePerMeterSquaredPerSecond et InchesPerHour. Pour ajouter d’autres unités, consultez la section [Ajouter ExtendedType](#add-extendedtype) de cet article.
weatherMeasures > AggregationType  | Type d’agrégation. Les valeurs possibles sont None, Average, Maximum, Minimum, StandardDeviation, Sum et Total.
weatherMeasures > Depth  | Profondeur du capteur en centimètres. Par exemple, la mesure de l’humidité à 10 cm dans le sol.
weatherMeasures > Description  | Description explicite de la mesure. 

JobType | Description |
--- | ---
Nom  | Nom du travail. Par exemple, Get_Daily_Forecast. Le client va exécuter ce travail pour récupérer les données météorologiques.|
pipelineDetails > parameters > name  | Nom du paramètre. |
pipelineDetails > parameters > type | Le type du paramètre. Les valeurs possibles sont String, Int, Float, Bool et Array. |
pipelineDetails > parameters > isRequired | Valeur booléenne du paramètre. La valeur est true si le paramètre est obligatoire. Sinon, la valeur est false. La valeur par défaut est true. |
pipelineDetails > parameters > defaultValue | Valeur par défaut du paramètre. |
pipelineDetails > parameters > description | Description du paramètre. |
Propriétés  | Propriétés supplémentaires fournies par le fabricant.
Properties > programRunCommand | Commande d’exécution Docker. Cette commande s’exécute lorsque le client exécute le travail météo. |

WeatherDataLocation | Description |
--- | ---
weatherDataModelId  | ID du modèle WeatherDataModel correspondant qui a été créé au cours du démarrage.|
location  | Latitude, longitude et altitude. |
Nom | Nom de l'objet. |
Description | Description de l’emplacement des données météorologiques. |
farmId | (Facultatif) ID de la batterie de serveurs. Le client fournit cet ID dans le cadre du paramètre du travail. |
Propriétés  | Propriétés supplémentaires fournies par le fabricant.

Pour plus d’informations sur les objets et leurs propriétés, consultez [FarmBeats Swagger](https://aka.ms/FarmBeatsSwagger).

> [!NOTE]
> Les API retournent des ID uniques pour chaque instance créée. Le traducteur pour la gestion des périphériques et la synchronisation des métadonnées doit conserver cet ID.

**Synchronisation des métadonnées**

Le composant Docker Connector doit pouvoir envoyer des mises à jour sur les métadonnées. Par exemple, il doit envoyer des mises à jour lorsque le fournisseur météo ajoute de nouveaux paramètres à un jeu de données ou lorsque de nouvelles fonctionnalités, telles qu’une prévision de 30 jours, sont ajoutées.

> [!NOTE]
> La suppression n’est pas prise en charge pour les métadonnées dans le modèle de données météorologiques.
>
> Pour mettre à jour les métadonnées, vous devez appeler `/Get/{ID}` sur le modèle de données météorologiques. Mettez à jour les propriétés modifiées, puis effectuez un `/Put/{ID}` pour conserver les propriétés que l’utilisateur définit.

## <a name="weather-data-telemetry-specifications"></a>Spécifications des données météorologiques (télémétrie)

Les données météorologiques sont mappées à un message canonique envoyé (push) à Azure Event Hub pour traitement. Azure Event Hubs est un service assurant l’ingestion de données (de télémétrie) en temps réel à partir des appareils et applications connectés. 

Pour envoyer des données météorologiques à FarmBeats, créez un client qui envoie des messages à un Event Hub dans FarmBeats. Pour plus d’informations, consultez [Envoi de données de télémétrie à un Event Hub](../../event-hubs/event-hubs-dotnet-standard-getstarted-send.md).

Voici un exemple de code Python qui envoie des données de télémétrie en tant que client à un Event Hub spécifié.

```python
import azure
from azure.eventhub import EventHubClient, Sender, EventData, Receiver, Offset
EVENTHUBCONNECTIONSTRING = "<EventHub connection string provided by customer>"
EVENTHUBNAME = "<EventHub name provided by customer>"

write_client = EventHubClient.from_connection_string(EVENTHUBCONNECTIONSTRING, eventhub=EVENTHUBNAME, debug=False)
sender = write_client.add_sender(partition="0")
write_client.run()
for i in range(5):
    telemetry = "<Canonical telemetry message>"
    print("Sending telemetry: " + telemetry)
    sender.send(EventData(telemetry))
write_client.stop()

```

Voici le format de message canonique :

```json
{
   "weatherstations": [
   {
   "id": "ID of the WeatherDataLocation.",
   "weatherdata": [
   {
     "timestamp": "Timestamp of the data. For historical purposes, this is the time for which the observations are sent. For forecast, this is the time for which data is forecasted. Format is ISO 8601. Default time zone is UTC.",
     "predictiontimestamp": "Timestamp on which the forecast data is predicted. I.e., the time of prediction. Required only for forecast data. Format is ISO 8601. Default time zone is UTC. ",
     "weathermeasurename1": <value>,
     "weathermeasurename2": <value>
     }
     ]
    }
   ]
}
```

Voici un exemple de message de télémétrie :

```json
{
   "weatherstations": [
   {
     "id": "5e4b34e7-bf9e-4f39-xxxx-f3c06d0366ea",
     "weatherdata": [
     {
      "timestamp": "2019-07-10T00:00:00Z",
      "predictiontimestamp": "2019-07-05T00:00:00Z",
      "PrecipitationTotalLiquidEquivalent": 0,
      "AvgPressure": 0,
      "AvgRelativeHumidity": 72
     }
    ] 
  }
 ]
}

```

## <a name="troubleshooting-and-error-management"></a>Résolution des problèmes et gestion des erreurs

### <a name="error-logging"></a>Journalisation des erreurs

Le travail partenaire s’exécute dans l’infrastructure de travail existante. Par conséquent, les erreurs sont enregistrées de la même façon que les erreurs pour d’autres travaux FarmBeats préexistantes (telles que GetFarmData et SensorPlacement). L’activité Data Factory qui s’exécute dans le pipeline Data Factory journalise à la fois `STDERR` et `STDOUT`. Les deux fichiers sont disponibles dans le compte de stockage `datahublogs-xxx` au sein du groupe de ressources FarmBeats.

La bibliothèque DataHub fournit des fonctions d’assistance pour activer la journalisation dans le cadre des journaux DataHub généraux. Pour plus d’informations, consultez la page [GitHub pour l’image Docker de la NOAA](https://github.com/azurefarmbeats/noaa_docker/blob/master/datahub_lib/framework/logger.py).

### <a name="troubleshooting-and-support"></a>Résolution des problèmes et support

Si le client ne peut pas recevoir de données météorologiques dans l’instance FarmBeats, fournissez un support et un mécanisme pour résoudre le problème.

## <a name="add-extendedtype"></a>Ajouter ExtendedType

FarmBeats prend en charge l’ajout de nouveaux types et unités de mesure de capteur. Vous pouvez ajouter de nouvelles unités/nouveaux types en mettant à jour le fichier *bootstrap_manifest.json* dans l’[implémentation de référence](https://github.com/azurefarmbeats/noaa_docker).

Suivez les étapes ci-dessous pour ajouter un nouveau type WeatherMeasure, par exemple « PrecipitationDepth ».

1. Effectuez une requête `GET` sur /ExtendedType à l’aide de la requête `filter - key = WeatherMeasureType`.
2. Notez l’ID de l’objet retourné.
3. Ajoutez le nouveau type à la liste dans l’objet retourné. Effectuez une requête `PUT` sur /ExtendedType{ID} avec la nouvelle liste suivante. La charge utile d’entrée doit être identique à la réponse que vous avez reçue précédemment. La nouvelle unité doit être ajoutée à la fin de la liste de valeurs.

Pour plus d’informations sur l’API /ExtendedType, consultez le [Swagger FarmBeats](https://aka.ms/FarmBeatsSwagger).

## <a name="next-steps"></a>Étapes suivantes

Vous disposez maintenant d’un composant Docker Connecteur qui s’intègre à FarmBeats. Ensuite, découvrez comment [obtenir des données météorologiques](get-weather-data-from-weather-partner.md) à l’aide de votre image Docker dans FarmBeats. 
