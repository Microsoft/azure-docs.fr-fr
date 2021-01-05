---
title: Accès aux données de sécurité et de recommandation
description: Découvrez comment accéder à vos données d’alerte et de recommandation de sécurité lors de l’utilisation de Defender pour IoT.
services: defender-for-iot
ms.service: defender-for-iot
documentationcenter: na
author: mlottner
manager: rkarlin
editor: ''
ms.devlang: na
ms.topic: conceptual
ms.tgt_pltfrm: na
ms.workload: na
ms.date: 09/04/2020
ms.author: mlottner
ms.openlocfilehash: c4f65da4f9a4f7ade94d76e99d57439e62b3c808
ms.sourcegitcommit: 5db975ced62cd095be587d99da01949222fc69a3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/10/2020
ms.locfileid: "97092055"
---
# <a name="access-your-security-data"></a>Accéder à vos données de sécurité

Defender pour IoT stocke les alertes de sécurité, les recommandations et les données de sécurité brutes (si vous choisissez de les enregistrer) dans votre espace de travail Log Analytics.

## <a name="log-analytics"></a>Log Analytics

Pour configurer l’espace de travail Log Analytics utilisé :

1. Ouvrez votre IoT Hub.
1. Cliquez sur le panneau **Paramètres** sous la section **Sécurité**.
1. Cliquez sur **collection de données**, puis changez la configuration de votre espace de travail Log Analytics.

Pour accéder à vos alertes et recommandations dans votre espace de travail Log Analytics après la configuration :

1. Choisissez une alerte ou une recommandation dans Defender pour IoT.
1. Cliquez sur **Investigation poussée**, puis sur **Pour voir quels sont les appareils qui ont cette alerte, cliquez ici et consultez la colonne DeviceId**.

Pour plus d’informations sur l’interrogation des données à partir de Log Analytics, consultez [Bien démarrer avec les requêtes dans Log Analytics](/azure/azure-monitor/log-query/get-started-queries).

## <a name="security-alerts"></a>Alertes de sécurité

Les alertes de sécurité sont stockées dans la table _AzureSecurityOfThings.SecurityAlert_ dans l’espace de travail Log Analytics configuré pour la solution Defender pour IoT.

Nous avons fourni plusieurs requêtes utiles pour vous aider à commencer à explorer les alertes de sécurité.

### <a name="sample-records"></a>Exemples d’enregistrements

Sélectionnez quelques enregistrements aléatoires

```
// Select a few random records
//
SecurityAlert
| project
    TimeGenerated,
    IoTHubId=ResourceId,
    DeviceId=tostring(parse_json(ExtendedProperties)["DeviceId"]),
    AlertSeverity,
    DisplayName,
    Description,
    ExtendedProperties
| take 3
```

| TimeGenerated           | IoTHubId                                                                                                       | deviceId      | AlertSeverity | DisplayName                           | Description                                             | ExtendedProperties                                                                                                                                                             |
|-------------------------|----------------------------------------------------------------------------------------------------------------|---------------|---------------|---------------------------------------|---------------------------------------------------------|--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| 2018-11-18T18:10:29.000 | /subscriptions/<subscription_id>/resourceGroups/<resource_group>/providers/Microsoft.Devices/IotHubs/<iot_hub> | <device_name> | Élevé          | Attaque par force brute réussie           | Une attaque par force brute sur l’appareil a réussi        |    { "Full Source Address": "[\"10.165.12.18:\"]", "User Names": "[\"\"]", "DeviceId": "IoT-Device-Linux" }                                                                       |
| 2018-11-19T12:40:31.000 | /subscriptions/<subscription_id>/resourceGroups/<resource_group>/providers/Microsoft.Devices/IotHubs/<iot_hub> | <device_name> | Élevé          | Connexion locale réussie sur l’appareil      | Une connexion locale réussie à l’appareil a été détectée     | { "Remote Address": "?", "Remote Port": "", "Local Port": "", "Login Shell": "/bin/su", "Login Process Id": "28207", "User Name": "attacker", "DeviceId": "IoT-Device-Linux" } |
| 2018-11-19T12:40:31.000 | /subscriptions/<subscription_id>/resourceGroups/<resource_group>/providers/Microsoft.Devices/IotHubs/<iot_hub> | <device_name> | Élevé          | Échec de la tentative de connexion locale sur l’appareil  | Une tentative échouée de connexion locale à l’appareil a été détectée |    { "Remote Address": "?", "Remote Port": "", "Local Port": "", "Login Shell": "/bin/su", "Login Process Id": "22644", "User Name": "attacker", "DeviceId": "IoT-Device-Linux" } |

### <a name="device-summary"></a>Récapitulatif de l’appareil

Obtenez le nombre d’alertes de sécurité distinctes détectées la semaine dernière, groupées par IoT Hub, appareil, gravité d’alerte et type d’alerte.

```
// Get the number of distinct security alerts detected in the last week, grouped by
//   IoT hub, device, alert severity, alert type
//
SecurityAlert
| where TimeGenerated > ago(7d)
| summarize Cnt=dcount(SystemAlertId) by
    IoTHubId=ResourceId,
    DeviceId=tostring(parse_json(ExtendedProperties)["DeviceId"]),
    AlertSeverity,
    DisplayName
```

| IoTHubId                                                                                                       | deviceId      | AlertSeverity | DisplayName                           | Count |
|----------------------------------------------------------------------------------------------------------------|---------------|---------------|---------------------------------------|-----|
| /subscriptions/<subscription_id>/resourceGroups/<resource_group>/providers/Microsoft.Devices/IotHubs/<iot_hub> | <device_name> | Élevé          | Attaque par force brute réussie           | 9   |
| /subscriptions/<subscription_id>/resourceGroups/<resource_group>/providers/Microsoft.Devices/IotHubs/<iot_hub> | <device_name> | Moyenne        | Échec de la tentative de connexion locale sur l’appareil  | 242 |
| /subscriptions/<subscription_id>/resourceGroups/<resource_group>/providers/Microsoft.Devices/IotHubs/<iot_hub> | <device_name> | Élevé          | Connexion locale réussie sur l’appareil      | 31  |
| /subscriptions/<subscription_id>/resourceGroups/<resource_group>/providers/Microsoft.Devices/IotHubs/<iot_hub> | <device_name> | Moyenne        | Crypto Coin Miner                     | 4   |

### <a name="iot-hub-summary"></a>Récapitulatif IoT Hub

Sélectionnez plusieurs appareils distincts ayant eu des alertes la dernière semaine, par IoT Hub, gravité d’alerte, type d’alerte

```
// Select number of distinct devices which had alerts in the last week, by
//   IoT hub, alert severity, alert type
//
SecurityAlert
| where TimeGenerated > ago(7d)
| extend DeviceId=tostring(parse_json(ExtendedProperties)["DeviceId"])
| summarize CntDevices=dcount(DeviceId) by
    IoTHubId=ResourceId,
    AlertSeverity,
    DisplayName
```

| IoTHubId                                                                                                       | AlertSeverity | DisplayName                           | CntDevices |
|----------------------------------------------------------------------------------------------------------------|---------------|---------------------------------------|------------|
| /subscriptions/<subscription_id>/resourceGroups/<resource_group>/providers/Microsoft.Devices/IotHubs/<iot_hub> | Élevé          | Attaque par force brute réussie           | 1          |
| /subscriptions/<subscription_id>/resourceGroups/<resource_group>/providers/Microsoft.Devices/IotHubs/<iot_hub> | Moyenne        | Échec de la tentative de connexion locale sur l’appareil  | 1          |
| /subscriptions/<subscription_id>/resourceGroups/<resource_group>/providers/Microsoft.Devices/IotHubs/<iot_hub> | Élevé          | Connexion locale réussie sur l’appareil      | 1          |
| /subscriptions/<subscription_id>/resourceGroups/<resource_group>/providers/Microsoft.Devices/IotHubs/<iot_hub> | Moyenne        | Crypto Coin Miner                     | 1          |

## <a name="security-recommendations"></a>Recommandations de sécurité

Les recommandations de sécurité sont stockées dans la table _AzureSecurityOfThings.SecurityRecommendation_ dans l’espace de travail Log Analytics configuré pour la solution Defender pour IoT.

Nous avons fourni plusieurs requêtes utiles pour vous aider à commencer à explorer les recommandations de sécurité.

### <a name="sample-records"></a>Exemples d’enregistrements

Sélectionnez quelques enregistrements aléatoires

```
// Select a few random records
//
SecurityRecommendation
| project
    TimeGenerated,
    IoTHubId=AssessedResourceId,
    DeviceId,
    RecommendationSeverity,
    RecommendationState,
    RecommendationDisplayName,
    Description,
    RecommendationAdditionalData
| take 2
```

| TimeGenerated | IoTHubId | deviceId | RecommendationSeverity | RecommendationState | RecommendationDisplayName | Description | RecommendationAdditionalData |
|---------------|----------|----------|------------------------|---------------------|---------------------------|-------------|------------------------------|
| 2019-03-22T10:21:06.060 |    /subscriptions/<subscription_id>/resourceGroups/<resource_group>/providers/Microsoft.Devices/IotHubs/<iot_hub> | <device_name> | Moyenne | Actif | Règle de pare-feu permissive détectée dans la chaîne d’entrée | Une règle dans le pare-feu contenant un modèle permissif pour un large éventail d’adresses IP ou de ports a été trouvée | {"Rules":"[{\"SourceAddress\":\"\",\"SourcePort\":\"\",\"DestinationAddress\":\"\",\"DestinationPort\":\"1337\"}]"} |
| 2019-03-22T10:50:27.237 | /subscriptions/<subscription_id>/resourceGroups/<resource_group>/providers/Microsoft.Devices/IotHubs/<iot_hub> | <device_name> | Moyenne | Actif | Règle de pare-feu permissive détectée dans la chaîne d’entrée | Une règle dans le pare-feu contenant un modèle permissif pour un large éventail d’adresses IP ou de ports a été trouvée | {"Rules":"[{\"SourceAddress\":\"\",\"SourcePort\":\"\",\"DestinationAddress\":\"\",\"DestinationPort\":\"1337\"}]"} |

### <a name="device-summary"></a>Récapitulatif de l’appareil

Obtenez le nombre de recommandations de sécurité actives distinctes, groupées par IoT Hub, appareil, gravité de la recommandation et type.

```
// Get the number of distinct active security recommendations, grouped by by
//   IoT hub, device, recommendation severity and type
//
SecurityRecommendation
| extend IoTHubId=AssessedResourceId
| summarize CurrentState=arg_max(RecommendationState, DiscoveredTimeUTC) by IoTHubId, DeviceId, RecommendationSeverity, RecommendationDisplayName
| where CurrentState == "Active"
| summarize Cnt=count() by IoTHubId, DeviceId, RecommendationSeverity
```

| IoTHubId                                                                                                       | deviceId      | RecommendationSeverity | Count |
|----------------------------------------------------------------------------------------------------------------|---------------|------------------------|-----|
| /subscriptions/<subscription_id>/resourceGroups/<resource_group>/providers/Microsoft.Devices/IotHubs/<iot_hub> | <device_name> | Élevé          | 2   |
| /subscriptions/<subscription_id>/resourceGroups/<resource_group>/providers/Microsoft.Devices/IotHubs/<iot_hub> | <device_name> | Moyenne        | 1 |
| /subscriptions/<subscription_id>/resourceGroups/<resource_group>/providers/Microsoft.Devices/IotHubs/<iot_hub> | <device_name> | Élevé          | 1  |
| /subscriptions/<subscription_id>/resourceGroups/<resource_group>/providers/Microsoft.Devices/IotHubs/<iot_hub> | <device_name> | Moyenne        | 4   |

## <a name="next-steps"></a>Étapes suivantes

- Lire la [Vue d’ensemble de Defender pour IoT](overview.md)
- Découvrir l’[architecture](architecture.md) de Defender pour IoT
- Comprendre et explorer les [alertes Defender pour IoT](concept-security-alerts.md)
- Comprendre et explorer les [recommandations Defender pour IoT](concept-recommendations.md)