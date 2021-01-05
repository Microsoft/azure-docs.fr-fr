---
title: Examiner un appareil suspect
description: Ce guide pratique explique comment utiliser Defender pour IoT afin d’examiner un appareil IoT suspect avec Log Analytics.
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
ms.openlocfilehash: cd7802fa4a88c3b05140d30ab4d8b2bbd4adc8f4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90930648"
---
# <a name="investigate-a-suspicious-iot-device"></a>Examiner un appareil IoT suspect

Les alertes du service Defender pour IoT donnent des indications claires lorsqu’il est possible qu’un appareil IoT soit impliqué dans des activités suspectes ou compromis.

Dans ce guide, utilisez les suggestions d’enquête fournies afin de déterminer les risques potentiels pour votre organisation, de choisir le mode de correction adéquat et de vous prémunir au mieux contre des attaques similaires.

> [!div class="checklist"]
> * Rechercher les données de votre appareil
> * Effectuer l’examen à l’aide de requêtes kql

## <a name="how-can-i-access-my-data"></a>Accéder à ses données

Par défaut, Defender pour IoT stocke vos suggestions et alertes de sécurité dans votre espace de travail Log Analytics. Vous pouvez également choisir de stocker vos données de sécurité brutes.

Pour rechercher votre espace de travail Log Analytics pour le stockage des données :

1. Ouvrez votre hub IoT.
1. Sous **Security** (Sécurité), sélectionnez **Settings** (Paramètres), puis **Data Collection** (Collection de données).
1. Changez les détails de la configuration de votre espace de travail Log Analytics.
1. Cliquez sur **Enregistrer**.

Une fois la configuration effectuée, procédez comme suit pour accéder aux données stockées dans votre espace de travail Log Analytics :

1. Dans votre hub IoT, cliquez sur une alerte Defender pour IoT.
1. Cliquez sur **Investigation poussée**.
1. Sélectionnez **Pour voir quels sont les appareils qui ont cette alerte, cliquez ici et consultez la colonne DeviceId**.

## <a name="investigation-steps-for-suspicious-iot-devices"></a>Étapes de l’examen des appareils IoT suspects

Pour accéder aux données brutes et aux insights de vos appareils IoT, allez dans votre espace de travail Log Analytics [pour accéder à vos données](#how-can-i-access-my-data).

Consultez les exemples de requêtes kql ci-dessous pour commencer à examiner les alertes et les activités sur votre appareil.

### <a name="related-alerts"></a>Alertes associées

Pour savoir si d’autres alertes ont été déclenchées à peu près au même moment, utilisez la requête kql suivante :

  ```
  let device = "YOUR_DEVICE_ID";
  let hub = "YOUR_HUB_NAME";
  SecurityAlert
  | where ExtendedProperties contains device and ResourceId contains tolower(hub)
  | project TimeGenerated, AlertName, AlertSeverity, Description, ExtendedProperties
  ```

### <a name="users-with-access"></a>Utilisateurs disposant d’un accès

Pour savoir quels utilisateurs ont accès à un appareil spécifique, utilisez la requête kql suivante :

 ```
  let device = "YOUR_DEVICE_ID";
  let hub = "YOUR_HUB_NAME";
  SecurityIoTRawEvent
  | where
     DeviceId == device and AssociatedResourceId contains tolower(hub)
     and RawEventName == "LocalUsers"
  | project
     TimestampLocal=extractjson("$.TimestampLocal", EventDetails, typeof(datetime)),
     GroupNames=extractjson("$.GroupNames", EventDetails, typeof(string)),
     UserName=extractjson("$.UserName", EventDetails, typeof(string))
  | summarize FirstObserved=min(TimestampLocal) by GroupNames, UserName
 ```
Ces données répondent aux questions suivantes :

- Quels sont les utilisateurs ayant accès à l’appareil ?
- Les utilisateurs y ayant accès ont-ils les niveaux d’autorisation prévus ?

### <a name="open-ports"></a>Ouvrir les ports

Pour savoir quels ports de l’appareil sont en cours d’utilisation ou ont été utilisés, exécutez la requête kql suivante :

 ```
  let device = "YOUR_DEVICE_ID";
  let hub = "YOUR_HUB_NAME";
  SecurityIoTRawEvent
  | where
     DeviceId == device and AssociatedResourceId contains tolower(hub)
     and RawEventName == "ListeningPorts"
     and extractjson("$.LocalPort", EventDetails, typeof(int)) <= 1024 // avoid short-lived TCP ports (Ephemeral)
  | project
     TimestampLocal=extractjson("$.TimestampLocal", EventDetails, typeof(datetime)),
     Protocol=extractjson("$.Protocol", EventDetails, typeof(string)),
     LocalAddress=extractjson("$.LocalAddress", EventDetails, typeof(string)),
     LocalPort=extractjson("$.LocalPort", EventDetails, typeof(int)),
     RemoteAddress=extractjson("$.RemoteAddress", EventDetails, typeof(string)),
     RemotePort=extractjson("$.RemotePort", EventDetails, typeof(string))
  | summarize MinObservedTime=min(TimestampLocal), MaxObservedTime=max(TimestampLocal), AllowedRemoteIPAddress=makeset(RemoteAddress), AllowedRemotePort=makeset(RemotePort) by Protocol, LocalPort
 ```

Ces données répondent aux questions suivantes :

- Quels sont les sockets d’écoute actifs sur l’appareil ?
- Les sockets d’écoute actuellement actifs sont-ils censés être autorisés ?
- Y a-t-il des adresses distantes suspectes connectées à l’appareil ?

### <a name="user-logins"></a>Connexions utilisateur

Pour savoir quels utilisateurs se sont connectés à l’appareil, utilisez la requête kql suivante :

 ```
  let device = "YOUR_DEVICE_ID";
  let hub = "YOUR_HUB_NAME";
  SecurityIoTRawEvent
  | where
     DeviceId == device and AssociatedResourceId contains tolower(hub)
     and RawEventName == "Login"
     // filter out local, invalid and failed logins
     and EventDetails contains "RemoteAddress"
     and EventDetails !contains '"RemoteAddress":"127.0.0.1"'
     and EventDetails !contains '"UserName":"(invalid user)"'
     and EventDetails !contains '"UserName":"(unknown user)"'
     //and EventDetails !contains '"Result":"Fail"'
  | project
     TimestampLocal=extractjson("$.TimestampLocal", EventDetails, typeof(datetime)),
     UserName=extractjson("$.UserName", EventDetails, typeof(string)),
     LoginHandler=extractjson("$.Executable", EventDetails, typeof(string)),
     RemoteAddress=extractjson("$.RemoteAddress", EventDetails, typeof(string)),
     Result=extractjson("$.Result", EventDetails, typeof(string))
  | summarize CntLoginAttempts=count(), MinObservedTime=min(TimestampLocal), MaxObservedTime=max(TimestampLocal), CntIPAddress=dcount(RemoteAddress), IPAddress=makeset(RemoteAddress) by UserName, Result, LoginHandler
 ```

Les résultats de la requête répondent aux questions suivantes :

- Quels sont les utilisateurs qui se sont connectés à l’appareil ?
- Les utilisateurs qui se sont connectés sont-ils censés le faire ?
- Ces utilisateurs se sont-ils connectés à partir d’adresses IP attendues ou inattendues ?

### <a name="process-list"></a>Liste de processus

Pour savoir si la liste de processus est conforme à ce qui est attendu, utilisez la requête kql suivante :

 ```
  let device = "YOUR_DEVICE_ID";
  let hub = "YOUR_HUB_NAME";
  SecurityIoTRawEvent
  | where
     DeviceId == device and AssociatedResourceId contains tolower(hub)
     and RawEventName == "ProcessCreate"
  | project
     TimestampLocal=extractjson("$.TimestampLocal", EventDetails, typeof(datetime)),
     Executable=extractjson("$.Executable", EventDetails, typeof(string)),
     UserId=extractjson("$.UserId", EventDetails, typeof(string)),
     CommandLine=extractjson("$.CommandLine", EventDetails, typeof(string))
  | join kind=leftouter (
     // user UserId details
     SecurityIoTRawEvent
     | where
        DeviceId == device and AssociatedResourceId contains tolower(hub)
        and RawEventName == "LocalUsers"
     | project
        UserId=extractjson("$.UserId", EventDetails, typeof(string)),
        UserName=extractjson("$.UserName", EventDetails, typeof(string))
     | distinct UserId, UserName
  ) on UserId
  | extend UserIdName = strcat("Id:", UserId, ", Name:", UserName)
  | summarize CntExecutions=count(), MinObservedTime=min(TimestampLocal), MaxObservedTime=max(TimestampLocal), ExecutingUsers=makeset(UserIdName), ExecutionCommandLines=makeset(CommandLine) by Executable
```

Les résultats de la requête répondent aux questions suivantes :

- Des processus suspects se sont-ils exécutés sur l’appareil ?
- Les processus ont-ils été exécutés par les utilisateurs appropriés ?
- Les exécutions de ligne de commande contenaient-elles les arguments corrects et attendus ?

## <a name="next-steps"></a>Étapes suivantes

Après avoir examiné un appareil pour mieux comprendre les risques que vous encourez, vous pouvez envisager de [configurer des alertes personnalisées](quickstart-create-custom-alerts.md) pour améliorer la sécurité de votre solution IoT. Si vous ne disposez pas d’un agent d’appareil, envisagez de [déployer un agent de sécurité](how-to-deploy-agent.md) ou de [changer la configuration d’un agent d’appareil existant](how-to-agent-configuration.md) afin d’améliorer vos résultats.
