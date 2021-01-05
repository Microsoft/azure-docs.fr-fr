---
title: 'Tutoriel : Recevoir des données d’appareil via Azure IoT Hub'
description: Ce tutoriel vous explique comment activer le routage de données d’appareil depuis IoT Hub vers l’API Azure par le biais du Connecteur Azure IoT pour FHIR.
services: healthcare-apis
author: ms-puneet-nagpal
ms.service: healthcare-apis
ms.subservice: iomt
ms.topic: tutorial
ms.date: 11/13/2020
ms.author: punagpal
ms.openlocfilehash: 6c364cf84bada2a951ef3f224ea836885f0e3c1e
ms.sourcegitcommit: 295db318df10f20ae4aa71b5b03f7fb6cba15fc3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/15/2020
ms.locfileid: "94636314"
---
# <a name="tutorial-receive-device-data-through-azure-iot-hub"></a>Tutoriel : Recevoir des données d’appareil via Azure IoT Hub

Le Connecteur Azure IoT pour Fast Healthcare Interoperability Resources (FHIR&#174;)* vous offre la possibilité d’ingérer des données à partir d’appareils de l’Internet des objets médicaux (IoMT, Internet of Medical Things) dans l’API Azure pour FHIR. Le guide de démarrage rapide [Déployer le Connecteur Azure IoT pour FHIR (préversion) à l’aide du portail Azure](iot-fhir-portal-quickstart.md) montre un exemple d’appareil managé par Azure IoT Central [envoyant des données de télémétrie](iot-fhir-portal-quickstart.md#connect-your-devices-to-iot) au Connecteur Azure IoT pour FHIR . Le Connecteur IoT pour FHIR peut également fonctionner avec des appareils approvisionnés et managés par Azure IoT Hub. Ce tutoriel présente la procédure à suivre pour connecter et router des données d’appareil depuis Azure IoT Hub vers le Connecteur Azure IoT pour FHIR.

## <a name="prerequisites"></a>Prérequis

- Un abonnement Azure actif - [En créer un gratuitement](https://azure.microsoft.com/free/?WT.mc_id=A261C142F)
- Une ressource d’API Azure pour FHIR avec au moins un Connecteur Azure IoT pour FHIR - [Déployer le Connecteur Azure IoT pour FHIR (préversion) à l’aide du portail Azure](iot-fhir-portal-quickstart.md)
- Une ressource Azure IoT Hub connectée à un ou plusieurs appareils réels ou simulés - [Créer un hub IoT à l’aide du portail Azure](../iot-hub/quickstart-send-telemetry-dotnet.md)

> [!TIP]
> Si vous utilisez une application d’appareil simulé Azure IoT Hub, vous pouvez librement choisir entre les différents langages et systèmes pris en charge.

## <a name="get-connection-string-for-azure-iot-connector-for-fhir-preview"></a>Obtenir une chaîne de connexion pour le Connecteur Azure IoT pour FHIR (préversion)

Azure IoT Hub nécessite une chaîne de connexion pour se connecter à votre Connecteur Azure IoT pour FHIR de manière sécurisée. Créez une chaîne de connexion pour votre Connecteur Azure IoT pour FHIR comme décrit dans [Générer une chaîne de connexion](iot-fhir-portal-quickstart.md#generate-a-connection-string). Conservez cette chaîne de connexion, qui sera utilisée à la prochaine étape.

En arrière-plan, le Connecteur Azure IoT pour FHIR utilise une instance Azure Event Hub pour recevoir les messages d’appareil. La chaîne de connexion créée précédemment est, fondamentalement, la chaîne de connexion à ce hub d’événements sous-jacent.

## <a name="connect-azure-iot-hub-with-the-azure-iot-connector-for-fhir-preview"></a>Connecter Azure IoT Hub au Connecteur Azure IoT pour FHIR (préversion)

Azure IoT Hub prend en charge une fonctionnalité appelée [routage des messages](../iot-hub/iot-hub-devguide-messages-d2c.md), qui permet d’envoyer des données d’appareil à différents services Azure comme le compte de stockage, Event Hub et Service Bus. Le Connecteur Azure IoT pour FHIR utilise cette fonctionnalité pour connecter Azure IoT Hub et envoyer des données d’appareil d’Azure IoT Hub à son point de terminaison Event Hub.

> [!NOTE] 
> À ce stade, vous pouvez uniquement utiliser une commande PowerShell ou CLI pour [créer un routage de messages](../iot-hub/tutorial-routing.md). En effet, le hub d’événements du Connecteur Azure IoT pour FHIR n’est pas hébergé dans l’abonnement du client. Vous ne pourrez donc pas le voir sur le portail Azure. Toutefois, quand les objets de routage de message sont ajoutés à l’aide de PowerShell ou de l’interface CLI, vous pouvez les voir sur le portail Azure et vous pouvez les gérer à partir du portail.

La configuration d’un routage de message s’effectue en deux étapes.

### <a name="add-an-endpoint"></a>Ajout d’un point de terminaison
Cette étape consiste à définir un point de terminaison vers lequel le hub IoT Hub va router les données. Créez ce point de terminaison en utilisant la commande PowerShell [Add-AzIotHubRoutingEndpoint](/powershell/module/az.iothub/Add-AzIotHubRoutingEndpoint) ou la commande CLI [az iot hub routing-endpoint create](/cli/azure/iot/hub/routing-endpoint#az-iot-hub-routing-endpoint-create) selon vos préférences.

Voici la liste des paramètres à utiliser avec la commande pour créer un point de terminaison :

|Paramètre PowerShell|Paramètre CLI|Description|
|---|---|---|
|ResourceGroupName|resource-group|Nom du groupe de ressources de votre ressource IoT Hub.|
|Nom|hub-name|Nom de votre ressource IoT Hub.|
|EndpointName|endpoint-name|Utilisez un nom que vous souhaitez attribuer au point de terminaison en cours de création.|
|EndpointType|endpoint-type|Entrez un point de terminaison auquel IoT Hub doit se connecter. Utilisez la valeur littérale « EventHub » pour PowerShell et « eventhub » pour l’interface CLI.|
|EndpointResourceGroup|endpoint-resource-group|Nom du groupe de ressources de la ressource d’API Azure pour FHIR de votre Connecteur Azure IoT pour FHIR. Vous pouvez obtenir cette valeur à partir de la page Vue d’ensemble de l’API Azure pour FHIR.|
|EndpointSubscriptionId|endpoint-subscription-id|ID d’abonnement de la ressource d’API Azure pour FHIR de votre Connecteur Azure IoT pour FHIR. Vous pouvez obtenir cette valeur à partir de la page Vue d’ensemble de l’API Azure pour FHIR.|
|ConnectionString|connection-string|Chaîne de connexion à votre Connecteur Azure oT pour FHIR. Utilisez la valeur que vous avez obtenue à l’étape précédente.|

### <a name="add-a-message-route"></a>Ajout d’une route de message
Cette étape consiste à définir une route de message en utilisant le point de terminaison créé précédemment. Créez une route en utilisant la commande PowerShell [Add-AzIotHubRoute](/powershell/module/az.iothub/Add-AzIoTHubRoute) ou la commande CLI [az iot hub route create](/cli/azure/iot/hub/route#az-iot-hub-route-create), selon vos préférences.

Voici la liste des paramètres à utiliser avec la commande pour ajouter un itinéraire de message :

|Paramètre PowerShell|Paramètre CLI|Description|
|---|---|---|
|ResourceGroupName|g|Nom du groupe de ressources de votre ressource IoT Hub.|
|Nom|hub-name|Nom de votre ressource IoT Hub.|
|EndpointName|endpoint-name|Nom du point de terminaison que vous avez créé précédemment.|
|RouteName|route-name|Nom que vous souhaitez attribuer à la route de message en cours de création.|
|Source|source-type|Type de donnée à envoyer au point de terminaison. Utilisez la valeur littérale « DeviceMessages » pour PowerShell et « devicemessages » pour l’interface CLI.|

## <a name="send-device-message-to-iot-hub"></a>Envoyer un message d’appareil à IoT Hub

Utilisez votre appareil (réel ou simulé) pour envoyer l’exemple de message de fréquence cardiaque présenté ci-dessous à Azure IoT Hub. Ce message est routé vers le Connecteur Azure IoT pour FHIR, où il est transformé en ressource d’observation FHIR et stocké dans l’API Azure pour FHIR.

```json
{
  "HeartRate": 80,
  "RespiratoryRate": 12,
  "HeartRateVariability": 64,
  "BodyTemperature": 99.08839032397609,
  "BloodPressure": {
    "Systolic": 23,
    "Diastolic": 34
  },
  "Activity": "walking"
}
```
> [!IMPORTANT]
> Veillez à envoyer le message d’appareil conforme aux [modèles de mappage](iot-mapping-templates.md) configurés avec votre Connecteur Azure IoT pour FHIR.

## <a name="view-device-data-in-azure-api-for-fhir"></a>Voir les données d’appareil dans l’API Azure pour FHIR

Vous pouvez voir les ressources d’observation FHIR créées par le Connecteur Azure IoT pour FHIR sur l’API Azure pour FHIR à l’aide de Postman. Configurez [Postman pour accéder à l’API Azure pour FHIR](access-fhir-postman-tutorial.md) et envoyez une requête `GET` sur `https://your-fhir-server-url/Observation?code=http://loinc.org|8867-4` pour voir les ressources d’observation FHIR avec la valeur de fréquence cardiaque envoyée dans l’exemple de message ci-dessus.

> [!TIP]
> Vérifiez que votre utilisateur dispose de l’accès approprié au plan de données de l’API Azure pour FHIR. Utilisez le [contrôle d’accès en fonction du rôle Azure](configure-azure-rbac.md) (RBAC Azure) pour attribuer les rôles de plan de données nécessaires.


## <a name="next-steps"></a>Étapes suivantes

Dans ce guide de démarrage rapide, vous avez configuré Azure IoT Hub pour router les données d’appareil vers le Connecteur Azure IoT pour FHIR. Choisissez l’une des étapes suivantes pour en savoir plus sur le Connecteur Azure IoT pour FHIR :

Appréhendez les différentes étapes du workflow dans le Connecteur Azure IoT pour FHIR.

>[!div class="nextstepaction"]
>[Flux de données du Connecteur Azure IoT pour FHIR](iot-data-flow.md)

Découvrez comment configurer le connecteur IoT à l’aide de modèles de mappage d’appareil et FHIR.

>[!div class="nextstepaction"]
>[Modèles de mappage du Connecteur Azure IoT pour FHIR](iot-mapping-templates.md)

*Sur le portail Azure, le Connecteur Azure IoT pour FHIR est appelé Connecteur IoT (préversion). Le sigle FHIR est une marque déposée de HL7 et est utilisé avec l’autorisation de HL7.