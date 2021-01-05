---
title: Mise à l’échelle Azure IoT Hub | Microsoft Docs
description: Comment mettre à l’échelle votre IoT Hub pour gérer le débit de messages prévu et les fonctionnalités souhaitées. Inclut un résumé du débit pris en charge pour chaque niveau et des options pour le partitionnement.
author: wesmc7777
manager: timlt
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 06/28/2019
ms.author: wesmc
ms.custom:
- amqp
- mqtt
- 'Role: Cloud Development'
- 'Role: Operations'
ms.openlocfilehash: bd016fcfe377dc610d5918ad8128fff4f6473fe5
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92538457"
---
# <a name="choose-the-right-iot-hub-tier-for-your-solution"></a>Choisir le niveau IoT Hub correspondant à votre solution

Chaque solution IoT étant différente, Azure IoT Hub offre plusieurs options en fonction de la tarification et de la mise à l’échelle. Cet article vous aide à évaluer vos besoins IoT Hub. Pour plus d’informations sur les niveaux IoT Hub, consultez [Tarification IoT Hub](https://azure.microsoft.com/pricing/details/iot-hub).

Pour déterminer le niveau IoT Hub adapté à votre solution, vous devez vous poser deux questions :

**Quelles sont les fonctionnalités que je prévois d’utiliser ?**

Azure IoT Hub propose deux niveaux : de base et standard. Ils diffèrent quant au nombre de fonctionnalités qu’ils prennent en charge. Si votre solution IoT est basée sur la collecte de données à partir d’appareils et sur leur analyse de manière centralisée, le niveau de base est probablement adapté à vos besoins. Si vous souhaitez utiliser des configurations plus complexes pour le contrôle à distance des appareils IoT ou distribuer une partie de vos charges de travail sur les appareils eux-mêmes, envisagez plutôt le niveau standard. Pour une description détaillée des fonctionnalités incluses dans chaque niveau, consultez [Niveaux de base et standard](#basic-and-standard-tiers).

**Quel volume de données vais-je déplacer au quotidien ?**

Chaque niveau IoT Hub est disponible en trois tailles, en fonction du débit de données à traiter au quotidien. Ces tailles sont identifiées au moyen de chiffre : 1, 2 et 3. Par exemple, chaque unité d’un IoT Hub de niveau 1 peut gérer 400 000 messages par jour, alors qu’une unité de niveau 3 peut en gérer 300 000 000. Pour plus d’informations sur les règles de données, consultez [Débit de messages](#message-throughput).

## <a name="basic-and-standard-tiers"></a>Niveaux de base et standard

Le niveau standard de IoT Hub active toutes les fonctionnalités. Il est requis pour toutes les solutions IoT qui comptent utiliser les fonctionnalités de communication bidirectionnelle. Le niveau de base active un sous-ensemble de fonctionnalités. Il est destiné aux solutions IoT qui nécessitent uniquement une communication unidirectionnelle, des appareils vers le cloud. Ces deux niveaux offrent les mêmes fonctionnalités de sécurité et d’authentification.

Dans un niveau, vous ne pouvez choisir qu’un seul type [d’édition](https://azure.microsoft.com/pricing/details/iot-hub/) par hub IoT. Par exemple, vous pouvez créer un hub IoT avec plusieurs unités de S1, mais pas avec plusieurs unités de différentes éditions, telles que S1 et S2.

| Fonctionnalité | Niveau de base | Niveau gratuit/Standard |
| ---------- | ---------- | ------------- |
| [Télémétrie appareil-à-cloud](iot-hub-devguide-messaging.md) | Oui | Oui |
| [Identité par appareil](iot-hub-devguide-identity-registry.md) | Oui | Oui |
| [Routage de messages](iot-hub-devguide-messages-read-custom.md), [enrichissements des messages](iot-hub-message-enrichments-overview.md) et [intégration Event Grid](iot-hub-event-grid.md) | Oui | Oui |
| [Protocoles HTTP, AMQP et MQTT](iot-hub-devguide-protocols.md) | Oui | Oui |
| [Service Device Provisioning](../iot-dps/about-iot-dps.md) | Oui | Oui |
| [Surveillance et diagnostics](monitor-iot-hub.md) | Oui | Oui |
| [Messages de cloud-à-appareil](iot-hub-devguide-c2d-guidance.md) |   | Oui |
| [Jumeaux d’appareil](iot-hub-devguide-device-twins.md), [Jumeaux de module](iot-hub-devguide-module-twins.md) et [Gestion des appareils](iot-hub-device-management-overview.md) |   | Oui |
| [Flux d’appareils (préversion)](iot-hub-device-streams-overview.md) |   | Oui |
| [Azure IoT Edge](../iot-edge/about-iot-edge.md) |   | Oui |
| [IoT Plug and Play](../iot-pnp/overview-iot-plug-and-play.md) |   | Oui |

IoT Hub propose également un niveau gratuit à des fins de test et d’évaluation. Il possède toutes les fonctionnalités du niveau standard, mais ses allocations en termes de messages sont limitées. Vous ne pouvez pas faire évoluer le niveau gratuit vers le niveau de base ou standard.

## <a name="partitions"></a>Partitions

Les Azure IoT Hubs contiennent de nombreux composants de base [Azure Event Hubs](../event-hubs/event-hubs-features.md), y compris des [partitions](../event-hubs/event-hubs-features.md#partitions). Les flux d’événements pour IoT Hubs sont généralement renseignés à l'aide des données de télémétrie entrantes signalées par différents appareils IoT. Le partitionnement du flux d’événements est utilisé pour réduire les conflits qui se produisent lors des opérations simultanées de lecture et d'écriture dans les flux d’événements.

Le nombre de partitions est choisi lors de la création de l'IoT Hub et ne peut pas être modifié. Le nombre de partitions est limité à 32 pour le niveau De base et le niveau Standard d’IoT Hub. La plupart des hubs IoT n’ont besoin que de 4 partitions. Pour plus d’informations sur les partitions, consultez le Forum aux questions relatif aux Event Hubs [De combien de partitions ai-je besoin ?](../event-hubs/event-hubs-faq.md#how-many-partitions-do-i-need)

## <a name="tier-upgrade"></a>Évolution des niveaux

Une fois votre IoT Hub créé, vous pouvez le faire évoluer et passer du niveau de base au niveau standard sans interrompre vos opérations existantes. Pour plus d’informations, consultez [Comment mettre à niveau votre IoT Hub](iot-hub-upgrade.md).

La configuration des partitions n'évolue pas lorsque vous migrez du niveau De base vers le niveau Standard.

> [!NOTE]
> Le niveau gratuit ne prend pas en charge la mise à niveau vers le niveau De base ou Standard.

## <a name="iot-hub-rest-apis"></a>API REST IoT Hub

La différence de fonctionnalités prises en charge entre les niveaux de base et standard de IoT Hub réside dans le fait que certains appels d’API ne fonctionnent pas avec des hubs utilisant le niveau de base. Le tableau suivant présente les API disponibles :

| API | Niveau de base | Niveau gratuit/Standard |
| --- | ---------- | ------------- |
| [Supprimer un appareil](/javascript/api/azure-iot-digitaltwins-service/registrymanager?view=azure-node-latest#deletedevice-string--models-registrymanagerdeletedeviceoptionalparams-) | Oui | Oui |
| [Obtenir un appareil](/azure/iot-hub/iot-c-sdk-ref/iothub-registrymanager-h/iothubregistrymanager-getdevice) | Oui | Oui |
| [Supprimer le module](/azure/iot-hub/iot-c-sdk-ref/iothub-registrymanager-h/iothubregistrymanager-deletemodule) | Oui | Oui |
| [Obtenir le module](/java/api/com.microsoft.azure.sdk.iot.service.registrymanager.getmodule?view=azure-java-stable) | Oui | Oui |
| [Obtenir les statistiques de Registre](/javascript/api/azure-iot-digitaltwins-service/registrymanager?view=azure-node-latest#getdevicestatistics-msrest-requestoptionsbase-) | Oui | Oui |
| [Obtenir les statistiques de services](/javascript/api/azure-iot-digitaltwins-service/registrymanager?view=azure-node-latest#getservicestatistics-msrest-requestoptionsbase-) | Oui | Oui |
| [Créer ou mettre à jour un appareil](/javascript/api/azure-iot-digitaltwins-service/registrymanager?view=azure-node-latest#createorupdatedevice-string--device--servicecallback-device--) | Oui | Oui |
| [Créer ou mettre à jour un module](/javascript/api/azure-iot-digitaltwins-service/registrymanager?view=azure-node-latest#createorupdatemodule-string--string--module--models-registrymanagercreateorupdatemoduleoptionalparams-) | Oui | Oui |
| [Interroger IoT Hub](/dotnet/api/microsoft.azure.devices.registrymanager?view=azure-dotnet) | Oui | Oui |
| [Créer l’URI SAS de téléchargement des fichiers](/rest/api/iothub/device/createfileuploadsasuri) | Oui | Oui |
| [Recevoir une notification d’appareil lié](/rest/api/iothub/device/receivedeviceboundnotification) | Oui | Oui |
| [Envoyer un événement d’appareil](/rest/api/iothub/device/senddeviceevent) | Oui | Oui |
| Envoyer un événement de module | AMQP et MQTT uniquement | AMQP et MQTT uniquement |
| [Mettre à jour l’état de chargement des fichiers](/rest/api/iothub/device/updatefileuploadstatus) | Oui | Oui |
| [Opération sur l’appareil en bloc](/javascript/api/azure-iot-digitaltwins-service/registrymanager?view=azure-node-latest#bulkdevicecrud-exportimportdevice----msrest-requestoptionsbase-) | Oui, à l’exception des fonctionnalités IoT Edge | Oui |
| [Annuler la tâche d’importation/exportation](/rest/api/iothub/service/jobs/cancelimportexportjob) | Oui | Oui |
| [Créer une tâche d’importation/exportation](/rest/api/iothub/service/jobs/createimportexportjob) | Oui | Oui |
| [Obtenir une tâche d’importation/exportation](/rest/api/iothub/service/jobs/getimportexportjob) | Oui | Oui |
| [Obtenir des tâches d’importation/exportation](/rest/api/iothub/service/jobs/getimportexportjobs) | Oui | Oui |
| [Purger la file d’attente de commandes](/javascript/api/azure-iot-digitaltwins-service/registrymanager?view=azure-node-latest#purgecommandqueue-string--msrest-requestoptionsbase-) |   | Oui |
| [Obtenir un jumeau d’appareil](/java/api/com.microsoft.azure.sdk.iot.device.deviceclient.getdevicetwin?view=azure-java-stable) |   | Oui |
| [Obtenir un jumeau de module](/azure/iot-hub/iot-c-sdk-ref/iothub-devicetwin-h/iothubdevicetwin-getmoduletwin) |   | Oui |
| [Appeler une méthode d’appareil](./iot-hub-devguide-direct-methods.md) |   | Oui |
| [Mettre à jour le jumeau d’appareil](./iot-hub-devguide-device-twins.md) |   | Oui |
| [Mettre à jour le jumeau de module](/azure/iot-hub/iot-c-sdk-ref/iothub-devicetwin-h/iothubdevicetwin-updatemoduletwin) |   | Oui |
| [Abandonner une notification d’appareil lié](/rest/api/iothub/device/abandondeviceboundnotification) |   | Oui |
| [Terminer une notification d’appareil lié](/rest/api/iothub/device/completedeviceboundnotification) |   | Oui |
| [Annuler un travail](/rest/api/media/jobs/canceljob) |   | Oui |
| [Créer un travail](/rest/api/media/jobs/create) |   | Oui |
| [Obtenir un travail](/java/api/com.microsoft.azure.sdk.iot.service.jobs.jobclient.getjob?view=azure-java-stable) |   | Oui |
| [Tâches de requête](/javascript/api/azure-iot-digitaltwins-service/jobclient?view=azure-node-latest#queryjobs-jobclientqueryjobsoptionalparams--servicecallback-queryresult--) |   | Oui |

## <a name="message-throughput"></a>Débit de messages

La meilleure façon de dimensionner une solution IoT Hub consiste à évaluer le trafic sur une base par unité. Prenez plus particulièrement en compte le débit maximal requis pour les catégories d’opérations suivantes :

* Messages appareil-à-cloud
* Messages Cloud vers appareil
* Opérations du registre d’identité

Le trafic se mesure pour votre hub IoT en fonction du nombre d’unités. Lorsque vous créez un hub IoT, vous choisissez son niveau et son édition, et définissez le nombre d’unités disponibles. Vous pouvez acheter jusqu’à 200 unités pour l’édition B1, B2, S1 ou S2, ou jusqu’à 10 unités pour l’édition B3 ou S3. Une fois votre hub IoT créé, vous pouvez changer le nombre d’unités disponibles dans son édition, mettre à niveau ou rétrograder l’édition au sein de son niveau (de B1 à B2) ou effectuer une mise à niveau du niveau de base au niveau standard (de B1 à S1) sans interrompre vos opérations existantes. Pour plus d’informations, consultez [Comment mettre à niveau votre IoT Hub](iot-hub-upgrade.md).  

Voici un exemple du trafic de chaque niveau. Les messages appareil-à-cloud suivent les règles de débit soutenu suivantes :

| Édition de niveau | Débit soutenu | Vitesse de transmission soutenue |
| --- | --- | --- |
| B1, S1 |Jusqu’à 1 111 Ko/minute par unité<br/>(1,5 Go/jour/unité) |Moyenne de 278 messages/minute par unité<br/>(400 000 messages/jour par unité) |
| B2, S2 |Jusqu’à 16 Mo/minute par unité<br/>(22,8 Go/jour/unité) |Moyenne de 4167 messages/minute par unité<br/>(6 millions de messages/jour par unité) |
| B3, S3 |Jusqu’à 814 Mo/minute par unité<br/>(1144,4 Go/jour/unité) |Moyenne de 208 333 messages/minute par unité<br/>(300 millions de messages/jour par unité) |

Le débit appareil-à-cloud ne représente qu’une des métriques que vous devez prendre en compte lors de la conception d’une solution IoT. Pour obtenir des informations plus complètes, consultez [Quotas et limitations IoT Hub](iot-hub-devguide-quotas-throttling.md).

### <a name="identity-registry-operation-throughput"></a>Débit des opérations de registre d’identité

Les opérations de registre d’identité IoT Hub ne sont pas censées être des opérations d’exécution, car elles sont principalement liées à l’approvisionnement d’appareils.

Pour obtenir les pics de performances spécifiques, consultez [Quotas et limitations IoT Hub](iot-hub-devguide-quotas-throttling.md).

## <a name="auto-scale"></a>Mise à l’échelle automatique

Si vous approchez du nombre maximal autorisé de messages sur votre hub IoT, vous pouvez utiliser cette [procédure de mise à l’échelle automatique](https://azure.microsoft.com/resources/samples/iot-hub-dotnet-autoscale/) afin d’incrémenter une unité IoT Hub dans le même niveau IoT Hub.

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur les fonctionnalités IoT Hub et des détails sur les performances, consultez [Tarification IoT Hub](https://azure.microsoft.com/pricing/details/iot-hub) ou [Quotas et limitations IoT Hub](iot-hub-devguide-quotas-throttling.md).

* Pour modifier votre niveau IoT Hub, suivez les étapes décrites dans [Comment mettre à niveau votre IoT Hub](iot-hub-upgrade.md).