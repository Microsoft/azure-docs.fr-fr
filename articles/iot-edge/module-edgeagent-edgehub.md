---
title: Propriétés des jumeaux de module de l’agent et du hub - Azure IoT Edge
description: Passez en revue les propriétés spécifiques et les valeurs des jumeaux de module edgeAgent et edgeHub.
author: kgremban
manager: philmea
ms.author: kgremban
ms.date: 08/31/2020
ms.topic: conceptual
ms.service: iot-edge
services: iot-edge
ms.openlocfilehash: fa7a56bcad067176d8f9805b418cca45ad144579
ms.sourcegitcommit: d103a93e7ef2dde1298f04e307920378a87e982a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/13/2020
ms.locfileid: "91978694"
---
# <a name="properties-of-the-iot-edge-agent-and-iot-edge-hub-module-twins"></a>Propriétés des jumeaux de module de l’agent IoT Edge et du hub IoT Edge

L’agent IoT Edge et le hub IoT Edge sont les deux modules qui constituent le runtime IoT Edge. Pour plus d’informations sur les responsabilités de chaque module de runtime, voir [Présentation du runtime Azure IoT Edge et de son architecture](iot-edge-runtime.md).

Cet article fournit les propriétés souhaitées et signalées des jumeaux de module du runtime. Pour plus d’informations sur le déploiement de modules sur des appareils IoT Edge, consultez [Déployer des modules et établir des itinéraires dans IoT Edge](module-composition.md).

Un jumeau de module inclut les éléments suivants :

* **Propriétés souhaitées (Desired)** . Le backend de solution peut définir les propriétés souhaitées, et le module peut les lire. Le module peut également recevoir des notifications sur les changements des propriétés souhaitées. Les propriétés souhaitées sont utilisées en même temps que les propriétés signalées pour synchroniser une configuration ou une condition de module.

* **Propriétés signalées (Reported)** . Le module peut définir les propriétés signalées, et le backend de solution peut les lire et les interroger. Les propriétés signalées sont utilisées en même temps que les propriétés souhaitées pour synchroniser une configuration ou une condition de module.

## <a name="edgeagent-desired-properties"></a>Propriétés souhaitées pour EdgeAgent

Le jumeau de module de l’agent IoT Edge est appelé `$edgeAgent` et coordonne les communications entre l’agent IoT Edge exécuté sur un appareil et IoT Hub. Les propriétés souhaitées sont définies lors de l’application d’un manifeste de déploiement sur un appareil spécifique dans le cadre d’un déploiement d’appareil unique ou à grande échelle.

| Propriété | Description | Obligatoire |
| -------- | ----------- | -------- |
| schemaVersion | « 1.0 » ou « 1.1 ». La version 1.1 a été introduite avec IoT Edge version 1.0.10, et est celle recommandée. | Oui |
| runtime.type | Doit être "docker" | Oui |
| runtime.settings.minDockerVersion | Définie sur la version Docker minimale requise par ce manifeste de déploiement | Oui |
| runtime.settings.loggingOptions | JSON converti en chaînes et contenant les options de journalisation du conteneur d’agent IoT Edge. [Options de journalisation Docker](https://docs.docker.com/engine/admin/logging/overview/) | Non |
| runtime.settings.registryCredentials<br>.{registryId}.username | Nom d’utilisateur du registre de conteneurs. Pour Azure Container Registry, le nom d’utilisateur est généralement le nom du registre.<br><br> Les informations d’identification du Registre sont nécessaires pour toutes les images de module privé. | Non |
| runtime.settings.registryCredentials<br>.{registryId}.password | Mot de passe du registre de conteneurs. | Non |
| runtime.settings.registryCredentials<br>.{registryId}.address | Adresse du registre de conteneurs. Pour Azure Container Registry, l’adresse est généralement *{nom du registre}.azurecr.io*. | Non |  
| systemModules.edgeAgent.type | Doit être "docker" | Oui |
| systemModules.edgeAgent.settings.image | URI de l’image de l’agent IoT Edge. Actuellement, l’agent IoT Edge ne peut pas se mettre à jour lui-même. | Oui |
| systemModules.edgeAgent.settings<br>.createOptions | JSON converti en chaînes et contenant les options de création du conteneur d’agent IoT Edge. [Options de création Docker](https://docs.docker.com/engine/api/v1.32/#operation/ContainerCreate) | Non |
| systemModules.edgeAgent.configuration.id | ID du déploiement ayant déployé ce module. | IoT Hub définit cette propriété quand le manifeste est appliqué à l’aide d’un déploiement. Ne fait pas partie d’un manifeste de déploiement. |
| systemModules.edgeHub.type | Doit être "docker" | Oui |
| systemModules.edgeHub.type | Doit être "running" | Oui |
| systemModules.edgeHub.restartPolicy | Doit être "always" | Oui |
| systemModules.edgeHub.startupOrder | Valeur entière pour l’emplacement d’un module dans l’ordre de démarrage. Le chiffre 0 est la première valeur et l’entier maximum (4 294 967 295) est la dernière. Si aucune valeur n’est fournie, la valeur par défaut est l’entier maximum.  | Non |
| systemModules.edgeHub.settings.image | URI de l’image du hub IoT Edge. | Oui |
| systemModules.edgeHub.settings<br>.createOptions | JSON converti en chaînes et contenant les options de création du conteneur du hub IoT Edge. [Options de création Docker](https://docs.docker.com/engine/api/v1.32/#operation/ContainerCreate) | Non |
| systemModules.edgeHub.configuration.id | ID du déploiement ayant déployé ce module. | IoT Hub définit cette propriété quand le manifeste est appliqué à l’aide d’un déploiement. Ne fait pas partie d’un manifeste de déploiement. |
| modules.{moduleId}.version | Chaîne définie par l’utilisateur représentant la version de ce module. | Oui |
| modules.{moduleId}.type | Doit être "docker" | Oui |
| modules.{moduleId}.status | {"running" \| "stopped"} | Oui |
| modules.{moduleId}.restartPolicy | {"never" \| "on-failure" \| "on-unhealthy" \| "always"} | Oui |
| modules.{moduleId}.startupOrder | Valeur entière pour l’emplacement d’un module dans l’ordre de démarrage. Le chiffre 0 est la première valeur et l’entier maximum (4 294 967 295) est la dernière. Si aucune valeur n’est fournie, la valeur par défaut est l’entier maximum.  | Non |
| modules.{moduleId}.imagePullPolicy | {"on-create" \| "never"} | Non |
| modules.{moduleId}.env | Liste de variables d’environnement à passer au module. Prend le format `"<name>": {"value": "<value>"}` | Non |
| modules.{moduleId}.settings.image | URI de l’image du module. | Oui |
| modules.{moduleId}.settings.createOptions | Champ de chaîne JSON contenant les options de création du conteneur de module. [Options de création Docker](https://docs.docker.com/engine/api/v1.32/#operation/ContainerCreate) | Non |
| modules.{moduleId}.configuration.id | ID du déploiement ayant déployé ce module. | IoT Hub définit cette propriété quand le manifeste est appliqué à l’aide d’un déploiement. Ne fait pas partie d’un manifeste de déploiement. |

## <a name="edgeagent-reported-properties"></a>Propriétés signalées pour EdgeAgent

Les propriétés signalées pour l’agent IoT Edge incluent trois informations principales :

1. l’état de l’application de dernières propriétés souhaitées affichées ;
2. L’état des modules en cours d’exécution sur l’appareil, comme signalé par l’agent IoT Edge
3. une copie des propriétés souhaitées en cours d’exécution sur l’appareil.

La copie des propriétés souhaitées actuelles est utile pour déterminer si l’appareil a appliqué le dernier déploiement ou exécute encore un manifeste de déploiement antérieur.

> [!NOTE]
> Les propriétés signalées de l’agent IoT Edge sont utiles, car elles peuvent être interrogées avec le [langage de requête IoT Hub](../iot-hub/iot-hub-devguide-query-language.md) afin de connaître l’état des déploiements à grande échelle. Pour plus d’informations sur l’utilisation des propriétés de l’agent IoT Edge pour l’état, consultez [Comprendre les déploiements IoT Edge pour les appareils uniques ou à grande échelle](module-deployment-monitoring.md).

Le tableau suivant n’inclut pas les informations copiées à partir des propriétés souhaitées.

| Propriété | Description |
| -------- | ----------- |
| lastDesiredVersion | Cet entier référence la dernière version des propriétés souhaitées traitées par l’agent IoT Edge. |
| lastDesiredStatus.code | Ce code d’état fait référence aux dernières propriétés souhaitées observées par l’agent IoT Edge. Valeurs autorisées : `200` Réussite, `400` Configuration non valide, `412` Version de schéma non valide, `417` les propriétés souhaitées sont vides, `500` Échec |
| lastDesiredStatus.description | Texte de description de l’état |
| deviceHealth | `healthy` si l’état du runtime de tous les modules est `running` ou `stopped`, sinon, `unhealthy` |
| configurationHealth.{deploymentId}.health | `healthy` si l’état du runtime de tous les modules définis par le déploiement {deploymentId} est `running` ou `stopped`, sinon, `unhealthy` |
| runtime.platform.OS | Signalement du système d’exploitation exécuté sur l’appareil |
| runtime.platform.architecture | Signalement de l’architecture de l’UC sur l’appareil |
| systemModules.edgeAgent.runtimeStatus | État signalé de l’agent IoT Edge : {"running" \| "unhealthy"} |
| systemModules.edgeAgent.statusDescription | Texte de description de l’état signalé de l’agent IoT Edge. |
| systemModules.edgeHub.runtimeStatus | État du hub IoT Edge : { "running" \| "stopped" \| "failed" \| "backoff" \| "unhealthy" } |
| systemModules.edgeHub.statusDescription | Texte de description de l’état du hub IoT Edge si celui-ci est non sain. |
| systemModules.edgeHub.exitCode | En cas de sortie, le code de sortie signalé par le conteneur du hub IoT Edge. |
| systemModules.edgeHub.startTimeUtc | Heure du dernier démarrage du hub IoT Edge |
| systemModules.edgeHub.lastExitTimeUtc | Heure à laquelle l’utilisateur a quitté le hub IoT Edge pour la dernière fois |
| systemModules.edgeHub.lastRestartTimeUtc | Heure du dernier redémarrage du hub IoT Edge |
| systemModules.edgeHub.restartCount | Nombre de redémarrages de ce module dans le cadre de la stratégie de redémarrage. |
| modules.{moduleId}.runtimeStatus | État du module : { "running" \| "stopped" \| "failed" \| "backoff" \| "unhealthy" } |
| modules.{moduleId}.statusDescription | Texte de description de l’état du module si celui-ci est non sain. |
| modules.{moduleId}.exitCode | En cas de sortie, le code de sortie signalé par le conteneur du module. |
| modules.{moduleId}.startTimeUtc | Heure du dernier démarrage du module |
| modules.{moduleId}.lastExitTimeUtc | Heure de la dernière sortie du module |
| modules.{moduleId}.lastRestartTimeUtc | Heure du dernier redémarrage du module |
| modules.{moduleId}.restartCount | Nombre de redémarrages de ce module dans le cadre de la stratégie de redémarrage. |

## <a name="edgehub-desired-properties"></a>Propriétés souhaitées pour EdgeHub

Le jumeau de module du hub IoT Edge est appelé `$edgeHub` et coordonne les communications entre le hub IoT Edge exécuté sur un appareil et IoT Hub. Les propriétés souhaitées sont définies lors de l’application d’un manifeste de déploiement sur un appareil spécifique dans le cadre d’un déploiement d’appareil unique ou à grande échelle.

| Propriété | Description | Requise dans le manifeste de déploiement |
| -------- | ----------- | -------- |
| schemaVersion | « 1.0 » ou « 1.1 ». La version 1.1 a été introduite avec IoT Edge version 1.0.10, et est celle recommandée. | Oui |
| routes.{routeName} | Chaîne représentant la route d’un hub IoT Edge. Pour plus d’informations, consultez [Déclarer des itinéraires](module-composition.md#declare-routes). | L’élément `routes` peut être présent mais vide. |
| storeAndForwardConfiguration.timeToLiveSecs | Durée en secondes pendant laquelle le hub IoT Edge conserve les messages en cas de points de terminaison de routage déconnectés, par exemple, déconnectés d’IoT Hub ou d’un module local. La valeur peut être un entier positif. | Oui |

## <a name="edgehub-reported-properties"></a>Propriétés signalées pour EdgeHub

| Propriété | Description |
| -------- | ----------- |
| lastDesiredVersion | Cet entier référence la dernière version des propriétés souhaitées traitées par le hub IoT Edge. |
| lastDesiredStatus.code | Ce code d’état fait référence aux dernières propriétés souhaitées observées par le hub IoT Edge. Valeurs autorisées : `200` Réussite, `400` Configuration non valide, `500` Échec |
| lastDesiredStatus.description | Texte de description de l’état. |
| clients.{device or moduleId}.status | État de connectivité de cet appareil ou module. Valeurs possibles {"connected" \| "disconnected"}. Seules les identités de module peuvent être à l’état déconnecté. Les appareils en aval se connectant au hub IoT Edge ne s’affichent que lorsqu’ils sont connectés. |
| clients.{device or moduleId}.lastConnectTime | Dernière connexion de l’appareil ou du module. |
| clients.{device or moduleId}.lastDisconnectTime | Dernière déconnexion de l’appareil ou du module. |

## <a name="next-steps"></a>Étapes suivantes

Pour savoir comment utiliser ces propriétés pour générer des manifestes de déploiement, consultez [Comprendre comment les modules IoT Edge peuvent être utilisés, configurés et réutilisées](module-composition.md).
