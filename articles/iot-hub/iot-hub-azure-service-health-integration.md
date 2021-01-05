---
title: Vérifier l’intégrité des ressources et du service Azure IoT Hub | Microsoft Docs
description: Utilisez Azure Service Health et Azure Resource Health pour surveiller IoT Hub
author: robinsh
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 04/21/2020
ms.author: robinsh
ms.custom:
- amqp
- 'Role: Cloud Development'
- 'Role: Technical Support'
- devx-track-csharp
ms.openlocfilehash: 27fca7b76ab148fc355eb7d52ee0cbcbd3540458
ms.sourcegitcommit: d767156543e16e816fc8a0c3777f033d649ffd3c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/26/2020
ms.locfileid: "92548399"
---
# <a name="check-iot-hub-service-and-resource-health"></a>Vérifier le service IoT Hub et l’intégrité des ressources

Azure IoT Hub s’intègre au [service Azure Service Health](../service-health/overview.md) pour vous permettre de surveiller l’intégrité au niveau du service pour le service IoT Hub et des hubs IoT individuels. Vous pouvez également configurer des alertes pour être averti lorsque l’état du service IoT Hub ou d’un IoT Hub change. Le service Azure Service Health est une combinaison de trois services plus petits : Azure Resource Health, Azure Service Health et la page de l’état d’Azure. Les sections de cet article décrivent chaque service et sa relation à IoT Hub plus en détail.

Le service Azure Service Health vous aide à surveiller les événements de niveau de service, comme les pannes et les mises à niveau, qui peuvent affecter la disponibilité du service IoT Hub et de vos hubs IoT individuels. IoT Hub s’intègre également à Azure Monitor pour fournir des métriques de plateforme IoT Hub et des journaux de ressources IoT Hub que vous pouvez utiliser pour surveiller les erreurs opérationnelles et les conditions qui se produisent sur un hub IoT spécifique. Pour plus d’informations, consultez [Surveiller IoT Hub](monitor-iot-hub.md).

## <a name="check-health-of-an-iot-hub-with-azure-resource-health"></a>Vérifier l’intégrité d’un hub IoT avec Azure Resource Health

Azure Resource Health fait partie du service Azure Service Health qui effectue le suivi de l’intégrité des ressources individuelles. Vous pouvez vérifier l’état d’intégrité de votre IoT Hub directement à partir du portail.

Pour afficher l’état et l’historique d’état de votre IoT Hub à l’aide du portail, procédez comme suit :

1. Sur le [portail Azure](https://portal.azure.com), accédez à votre déploiement IoT Hub.

1. Dans le volet gauche, sous **Support + résolution des problèmes** , sélectionnez **Resource Health** .

    :::image type="content" source="./media/iot-hub-azure-service-health-integration/iot-hub-resource-health.png" alt-text="Page Resource Health pour un IoT Hub":::

Pour en savoir plus sur Azure Resource Health et l’interprétation des données d’intégrité, consultez la [Vue d’ensemble de Resource Health](../service-health/resource-health-overview.md) dans la documentation Azure Service Health.

Vous pouvez également sélectionner **Ajouter une alerte d’intégrité de ressource** pour configurer des alertes à déclencher lorsque l’état d’intégrité de votre IoT Hub change. Pour plus d’informations, consultez [Configurer des alertes pour des événements Service Health](../service-health/alerts-activity-log-service-notifications-portal.md) et les rubriques connexes dans la documentation Azure Service Health.

## <a name="check-health-of-iot-hubs-in-your-subscription-with-azure-service-health"></a>Vérifier l’intégrité des hubs IoT dans votre abonnement avec Azure Service Health

Avec Azure Service Health, vous pouvez vérifier l’état d’intégrité de tous les hubs IoT dans votre abonnement.

Pour vérifier l’intégrité de vos IoT Hubs, procédez comme suit :

1. Connectez-vous au [portail Azure](https://portal.azure.com).

2. Accédez à **Service Health** > **Resource Health** .

3. Dans les champs déroulants, sélectionnez votre abonnement, puis **IoT Hub** comme type de ressource.

Pour en savoir plus sur Azure Service Health et l’interprétation des données d’intégrité, consultez la [Vue d’ensemble de Service Health](../service-health/service-health-overview.md) dans la documentation Azure Service Health.

Pour savoir comment configurer des alertes avec Azure Service Health, consultez [Configurer des alertes pour des événements Service Health](../service-health/alerts-activity-log-service-notifications-portal.md) et les rubriques connexes dans la documentation Azure Service Health.

## <a name="check-health-of-the-iot-hub-service-by-region-on-azure-status-page"></a>Vérifier l’intégrité du service IoT Hub par région sur la page de l’état Azure

Pour afficher l’état d’IoT Hub et d’autres services par région à l’échelle mondiale, vous pouvez utiliser la [Page de l’état Azure](https://status.azure.com/status). Pour plus d’informations sur la page de l’état Azure, consultez [Vue d’ensemble de l’état Azure](../service-health/azure-status-overview.md) dans la documentation Azure Service Health.

## <a name="next-steps"></a>Étapes suivantes

* Pour plus d’informations sur Azure Service Health, Azure Resource Health et la page de l’état Azure, consultez [service Azure Service Health](../service-health/overview.md).
* Pour obtenir une description de la surveillance d’Azure IoT Hub, consultez [Surveiller Azure IoT Hub](monitor-iot-hub.md).
