---
title: Obtenir des données météorologiques auprès de partenaires météo
description: Cet article décrit comment obtenir des données météorologiques auprès de partenaires.
author: sunasing
ms.topic: article
ms.date: 03/31/2020
ms.author: sunasing
ms.openlocfilehash: bb28c517e353af6b8c1ee0cad788ff41b971918c
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96460882"
---
# <a name="get-weather-data-from-weather-partners"></a>Obtenir des données météorologiques auprès de partenaires météo

Azure FarmBeats vous permet d’importer des données météorologiques à partir de vos fournisseurs de données météorologiques à l’aide d’un framework de connecteur basé sur Docker. À l’aide de ce framework, les fournisseurs de données météorologiques implémentent un Docker qui peut être intégré à FarmBeats. Actuellement, le fournisseur de données météorologiques suivant est pris en charge.

  ![Partenaires de FarmBeats](./media/get-sensor-data-from-sensor-partner/dtn-logo.png)
  
   [DTN](https://www.dtn.com/dtn-content-integration/)

Les données météorologiques peuvent être utilisées pour générer des insights actionnables et créer des modèles IA/ML sur FarmBeats.

## <a name="before-you-start"></a>Avant de commencer

Pour obtenir des données météorologiques, vérifiez que vous avez [installé FarmBeats](./install-azure-farmbeats.md). L’intégration des données météorologiques est prise en charge dans les versions 1.2.11 et supérieures. 

## <a name="enable-weather-integration-with-farmbeats"></a>Activer l’intégration des données météorologiques avec FarmBeats

Pour commencer à obtenir des données météorologiques sur votre hub de données FarmBeats :

1. Accédez à votre Swagger de hub de données FarmBeats `https://farmbeatswebsite-api.azurewebsites.net/swagger`.

2. Accédez à l’API /Partner, puis créez une requête POST. Utilisez la charge utile d'entrée suivante :

   ```json
   {  
 
     "dockerDetails": {  
       "credentials": { 
         "username": "<credentials to access private docker - not required for public docker>", 
         "password": "<credentials to access private docker – not required for public docker>"  
       },  
       "imageName" : "<docker image name",
       "imageTag" : "<docker image tag, default:latest>",
       "azureBatchVMDetails": {  
         "batchVMSKU" : "<VM SKU. Default is standard_d2_v2>",  
         "dedicatedComputerNodes" : 1,
         "nodeAgentSKUID": "<Node SKU. Default is batch.node.ubuntu 18.04>"  
       }
     },
     "partnerCredentials": {  
       "key1": "value1",  
       "key2": "value2"  
     },  
     "partnerType": "Weather",  
     "name": "<Name of the partner>",  
     "description": "<Description>",
     "properties": {  }  
   }  
   ```

   Par exemple, pour obtenir des données météorologiques de DTN, utilisez la charge utile suivante. Vous pouvez modifier le nom et la description en fonction de vos préférences.

   > [!NOTE]
   > L’étape suivante nécessite une clé API. Pour obtenir une clé pour votre abonnement DTN, contactez DTN.

   ```json
   {
 
     "dockerDetails": {
       "imageName": "dtnweather/dtn-farm-beats",
       "imageTag": "latest",
       "azureBatchVMDetails": {
         "batchVMSKU": "standard_d2_v2",
         "dedicatedComputerNodes": 1,
         "nodeAgentSKUID": "batch.node.ubuntu 18.04"
       }
     },
     "partnerCredentials": {
      "apikey": "<API key from DTN>"
      },
     "partnerType": "Weather",
     "name": "dtn-weather",
     "description": "DTN registered as a Weather Partner in FarmBeats"
   }  
   ```

   > [!NOTE]
   > Pour plus d’informations sur l’objet Partner, consultez [Annexe](get-weather-data-from-weather-partner.md#appendix) dans cet article.

   L’étape précédente provisionne les ressources pour permettre à Docker de s’exécuter dans l’environnement FarmBeats du client.  

   Il faut environ 10 à 15 minutes pour provisionner les ressources.

3. Vérifiez l’état de l’objet /Partner que vous avez créé à l’étape précédente. Pour vérifier l’état, créez une requête GET sur l’API /Partner et vérifiez l’état de l’objet Partner. Une fois que FarmBeats a provisionné le partenaire, l’état devient **Active**.

4. Sur l’API /JobType, créez une requête GET. Vérifiez les travaux météorologiques que vous avez créés précédemment dans le cadre du processus d’ajout de partenaires. Dans les travaux météorologiques, le champ **pipelineName** présente le format suivant : **nom-partenaire_type-partenaire_nom-travail**.

      Votre instance FarmBeats a maintenant un partenaire de données météorologiques actif. Vous pouvez exécuter des travaux pour demander des données météorologiques pour un lieu particulier (latitude et longitude) et une plage de dates spécifique. Les types de travaux contiennent des détails sur les paramètres requis pour exécuter des travaux météorologiques.

      Par exemple, pour DTN, les types de travaux suivants vont être créés :
   
      - **get_dtn_daily_observations** : Obtenir des observations quotidiennes pour un lieu et une période.
      - **get_dtn_daily_forecasts** : Obtenir des prévisions quotidiennes pour un lieu et une période.
      - **get_dtn_hourly_observations** : Obtenir des observations par heure pour un lieu et une période.
      - **get_dtn_hourly_forecasts** : Obtenir des prévisions par heure pour un lieu et une période.

6. Prenez note de l’ID et des paramètres des types de travaux.

7. Accédez à l’API /Jobs et créez une requête POST sur /Jobs. Utilisez la charge utile d'entrée suivante :

   ```json
    {
         "typeId": "<id of the JobType>",
         "arguments": {
           "additionalProp1": {},
           "additionalProp2": {},
           "additionalProp3": {}
         },
         "name": "<name of the job>",
         "description": "<description>",
         "properties": {}
       }
   ```

   Par exemple, pour exécuter **get_dtn_daily_observations**, utilisez la charge utile suivante :

   ```json
   { 
         "typeId": "<id of the JobType>",
         "arguments": {
               "latitude": 47.620422,
               "longitude": -122.349358,
               "days": 5
         },
         "name": "<name of the job>",
         "description": "<description>",
         "properties": {}
   }
   ```

8. L’étape précédente exécute les travaux météorologiques tels qu’ils sont définis dans le Docker du partenaire et ingère les données météorologiques dans FarmBeats. Vous pouvez vérifier l’état du travail en créant une requête GET sur /Jobs. Dans la réponse, recherchez **currentState**. Une fois l’opération terminée, **currentState** est défini sur **Succeeded**.

## <a name="query-ingested-weather-data"></a>Données météorologiques ingérées par la requête

Une fois les travaux météorologiques effectués, vous pouvez interroger les données météorologiques ingérées pour générer des modèles ou des insights actionnables à l’aide des API REST FarmBeats Datahub.

Pour interroger les données météorologiques à l’aide d’une API REST FarmBeats :

1. Dans votre [Swagger](https://yourdatahub.azurewebsites.net/swagger) FarmBeats Datahub, accédez à l’API /WeatherDataLocation et créez une requête GET. La réponse inclut des objets /WeatherDataLocation créés pour le lieu (latitude et longitude) que le travail spécifiait. Prenez note de l’**ID** et du **weatherDataModelId** des objets.

2. Créez une requête GET/{id} sur l’API /WeatherDataModel pour le **weatherDataModelId** comme vous l’avez fait précédemment. Le modèle de données météorologiques contient toutes les métadonnées et les détails sur les données météorologiques ingérées. Par exemple, dans l’objet Weather Data Model, la mesure météorologique détaille les informations météorologiques prises en charge, ainsi que les types et unités. Par exemple :

   ```json
   {
       "name": "Temperature <name of the weather measure - this is what we will receive as part of the queried weather data>",
       "dataType": "Double <Data Type - eg. Double, Enum>",
       "type": "AmbientTemperature <Type of measure eg. AmbientTemperature, Wind speed etc.>",
       "unit": "Celsius <Unit of measure eg. Celsius, Percentage etc.>",
       "aggregationType": "None <either of None, Average, Maximum, Minimum, StandardDeviation, Sum, Total>",
       "description": "<Description of the measure>"
   }
   ```

   Prenez note de la réponse de l’appel GET/{id} pour le modèle de données météorologiques.

3. Accédez à l’API Telemetry et créez une requête POST. Utilisez la charge utile d'entrée suivante :

   ```json
   {
       "weatherDataLocationId": "<id from step 1 above>",
     "searchSpan": {
       "from": "2020-XX-XXT07:30:00Z",
       "to": "2020-XX-XXT07:45:00Z"
     }
   }
   ```

    La réponse affiche les données météorologiques disponibles pour la période spécifiée :

   ```json
   {
     "timestamps": [
       "2020-XX-XXT07:30:00Z",
       "2020-XX-XXT07:45:00Z"
     ],
     "properties": [
       {
         "values": [
           "<id of the weatherDataLocation>",
           "<id of the weatherDataLocation>"
         ],
         "name": "Id",
         "type": "String"
       },
       {
         "values": [
           29.1,
           30.2
         ],
         "name": "Temperature <name of the WeatherMeasure as defined in the WeatherDataModel object>",
         "type": "Double <Data Type of the value - eg. Double>"
       }
     ]
   }
   ```

Dans l’exemple précédent, la réponse affiche des données pour deux horodatages. Elle indique également le nom de la mesure (température) et les valeurs des données météorologiques signalées dans les deux horodatages. Reportez-vous au modèle de données météorologiques associé pour interpréter le type et l’unité des valeurs signalées.

## <a name="troubleshoot-job-failures"></a>Résoudre les échecs liés aux travaux

Pour résoudre les échecs liés aux travaux, [consultez les journaux des travaux](troubleshoot-azure-farmbeats.md#weather-data-job-failures).


## <a name="appendix"></a>Annexe

|        Partenaire   |  Détails   |
| ------- | -------             |
|     DockerDetails - imageName         |          Nom de l’image Docker. Par exemple docker.io/mydockerimage (image dans hub.docker.com) ou myazureacr.azurecr.io/mydockerimage (image dans Azure Container Registry), etc. Si aucun registre n’est fourni, la valeur par défaut est hub.docker.com.      |
|          DockerDetails - imageTag             |         Nom d’étiquette de l’image Docker. La valeur par défaut est « latest ».     |
|  DockerDetails - credentials      |  Informations d’identification pour accéder au Docker privé. Le partenaire fournit les informations d’identification.   |
|  DockerDetails - azureBatchVMDetails - batchVMSKU     |    Référence SKU de machine virtuelle Azure Batch Pour plus d’informations, consultez [toutes les machines virtuelles Linux disponibles](../../virtual-machines/sizes.md?toc=%2fazure%2fvirtual-machines%2flinux%2ftoc.json). <BR> <BR> Les valeurs valides sont les suivantes : « Small », « ExtraLarge », « Large », « A8 », « A9 », « Medium », « A5 », « A6 », « A7 », « STANDARD_D1 », « STANDARD_D2 », « STANDARD_D3 », « STANDARD_D4 », « STANDARD_D11 », « STANDARD_D12 », « STANDARD_D13 », « STANDARD_D14 », « A10 », « A11 », « STANDARD_D1_V2 », « STANDARD_D2_V2 », « STANDARD_D3_V2 », « STANDARD_D4_V2 », « STANDARD_D11_V2 », « STANDARD_D12_V2 », « STANDARD_D13_V2 », « STANDARD_D14_V2 », « STANDARD_G1 », « STANDARD_G2 », « STANDARD_G3 », « STANDARD_G4 », « STANDARD_G5 », « STANDARD_D5_V2 », « BASIC_A1 », « BASIC_A2 », « BASIC_A3 », « BASIC_A4 », « STANDARD_A1 », « STANDARD_A2 », « STANDARD_A3 », « STANDARD_A4 », « STANDARD_A5 », « STANDARD_A6 », « STANDARD_A7 », « STANDARD_A8 », « STANDARD_A9 », « STANDARD_A10 », « STANDARD_A11 », « STANDARD_D15_V2 », « STANDARD_F1 », « STANDARD_F2 », « STANDARD_F4 », « STANDARD_F8 », « STANDARD_F16 », « STANDARD_NV6 », « STANDARD_NV12 », « STANDARD_NV24 », « STANDARD_NC6 », « STANDARD_NC12 », « STANDARD_NC24 », « STANDARD_NC24r », « STANDARD_H8 », « STANDARD_H8m », « STANDARD_H16 », « STANDARD_H16m », « STANDARD_H16mr », « STANDARD_H16r », « STANDARD_A1_V2 », « STANDARD_A2_V2 », « STANDARD_A4_V2 », « STANDARD_A8_V2 », « STANDARD_A2m_V2 », « STANDARD_A4m_V2 », « STANDARD_A8m_V2 », « STANDARD_M64ms », « STANDARD_M128s » et « STANDARD_D2_V3 ». *La valeur par défaut est « STANDARD_D2_V2 »* .  |
|    DockerDetails - azureBatchVMDetails - dedicatedComputerNodes   |  Nombre de nœuds d’ordinateur dédiés pour le pool Batch. La valeur par défaut est 1. |
|    DockerDetails - azureBatchVMDetails - nodeAgentSKUID          |    ID de référence SKU de l’agent du nœud Azure Batch. Actuellement, seul l’agent de nœud Batch « batch.node.ubuntu 18.04 » est pris en charge.    |
| DockerDetails - partnerCredentials | Informations d’identification pour l’appel de l’API partenaire dans Docker. Le partenaire fournit ces informations en fonction du mécanisme d’autorisation pris en charge. Il s’agit par exemple du nom d’utilisateur et du mot de passe, ou de clés API. |
| partnerType | « Weather ». Les autres types de partenaires dans FarmBeats sont « Sensor » et « Imagery ».  |
|  name   |   Nom souhaité du partenaire dans le système FarmBeats.   |
|  description |  Description.   |

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous avez interrogé les données de capteur à partir de votre instance Azure FarmBeats, découvrez comment [générer des mappages](generate-maps-in-azure-farmbeats.md#generate-maps) pour vos batteries de serveurs.