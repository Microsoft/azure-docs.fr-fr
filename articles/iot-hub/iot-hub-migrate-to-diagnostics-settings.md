---
title: Migrer la surveillance des opérations Azure IoT Hub vers les journaux de ressources IoT Hub dans Azure Monitor | Microsoft Docs
description: Découvrez comment mettre à jour Azure IoT Hub pour utiliser Azure Monitor plutôt que la surveillance des opérations servant à superviser l’état des opérations sur votre hub IoT en temps réel.
author: kgremban
manager: philmea
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 03/11/2019
ms.author: kgremban
ms.openlocfilehash: dfd819d82f5e35183802e33e5d423cad4de36c38
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96461590"
---
# <a name="migrate-your-iot-hub-from-operations-monitoring-to-azure-monitor-resource-logs"></a>Migrer votre IoT Hub de la surveillance des opérations vers les journaux de ressources Azure Monitor

Les clients qui utilisent la [surveillance des opérations](iot-hub-operations-monitoring.md) pour suivre l’état des opérations dans IoT Hub peuvent migrer ce workflow vers les [journaux de ressources Azure Monitor](../azure-monitor/platform/platform-logs-overview.md), une fonctionnalité d’Azure Monitor. Les journaux de ressources fournissent des informations de diagnostic au niveau des ressources pour de nombreux services Azure.

**La fonctionnalité de surveillance des opérations d’IoT Hub est déconseillée** et a été supprimée du portail. Cet article montre les étapes à effectuer pour déplacer vos charges de travail de la surveillance des opérations vers les journaux de ressources Azure Monitor. Pour plus d’informations sur la chronologie de dépréciation, consultez l’article [Surveiller vos solutions Azure IoT avec Azure Monitor et Azure Resource Health](https://azure.microsoft.com/blog/monitor-your-azure-iot-solutions-with-azure-monitor-and-azure-resource-health/).

## <a name="update-iot-hub"></a>Mettre à jour IoT Hub

Pour mettre à jour votre IoT Hub dans le portail Azure, commencez par activer un paramètre de diagnostic, puis désactivez la surveillance des opérations.  

### <a name="create-a--diagnostic-setting"></a>Créer un paramètre de diagnostic

1. Connectez-vous au [portail Azure](https://portal.azure.com) et accédez à votre IoT Hub.

1. Dans le volet de gauche, sous **Supervision**, sélectionnez **Paramètres de diagnostic**. Sélectionnez ensuite **Ajouter un paramètre de diagnostic**.

   :::image type="content" source="media/iot-hub-migrate-to-diagnostics-settings/open-diagnostic-settings.png" alt-text="Capture d’écran qui met en évidence Paramètres de diagnostic dans la section Supervision.":::

1. Dans le volet **Paramètre de diagnostic**, attribuez un nom au paramètre.

1. Sous **Détails de la catégorie**, sélectionnez les catégories pour les opérations que vous souhaitez analyser. Pour plus d’informations sur les catégories d’opérations disponibles avec IoT Hub, consultez [Journaux de ressources](monitor-iot-hub-reference.md#resource-logs).

1. Sous **Détails de la destination**, choisissez l’emplacement où vous souhaitez envoyer les journaux. Vous pouvez sélectionner n’importe quelle combinaison des destinations suivantes :

   * Archiver dans un compte de stockage
   * Diffuser vers un hub d’événements
   * Envoyer aux journaux Azure Monitor via l’espace de travail Log Analytics

   La capture d’écran suivante montre un paramètre de diagnostic qui achemine les opérations des catégories Connexions et Télémétrie d’appareil vers un espace de travail Log Analytics :

   :::image type="content" source="media/iot-hub-migrate-to-diagnostics-settings/add-diagnostic-setting.png" alt-text="Capture d’écran montrant un paramètre de diagnostic terminé.":::

1. Sélectionnez **Enregistrer** pour enregistrer les paramètres.

Les nouveaux paramètres prennent effet au bout de 10 minutes environ. Après cela, les journaux apparaissent dans la destination configurée. Pour en savoir plus sur la configuration des diagnostics, consultez [Collecter et utiliser des données de journaux à partir de vos ressources Azure](../azure-monitor/platform/platform-logs-overview.md).

Pour obtenir des informations plus détaillées sur la procédure de création de paramètres de diagnostic, notamment avec PowerShell et Azure CLI, consultez [Paramètres de diagnostic](../azure-monitor/platform/diagnostic-settings.md) dans la documentation Azure Monitor.

### <a name="turn-off-operations-monitoring"></a>Désactiver la surveillance des opérations

> [!NOTE]
> À compter du 11 mars 2019, la fonctionnalité de surveillance des opérations est supprimée de l’interface du portail Azure d’IoT Hub. La procédure décrite ci-dessous ne s’applique plus. Pour effectuer une migration, assurez-vous que les catégories appropriées sont acheminées vers une destination avec le paramètre de diagnostic Azure Monitor ci-dessus.

Une fois que vous avez testé les nouveaux paramètres de diagnostic sur votre flux de travail, vous pouvez désactiver la fonctionnalité de surveillance des opérations. 

1. Dans le menu de votre IoT Hub, sélectionnez **Surveillance des opérations**.

2. Sous chaque catégorie de surveillance, sélectionnez **Aucune**.

3. Enregistrez les modifications apportées à la surveillance des opérations.

## <a name="update-applications-that-use-operations-monitoring"></a>Mettre à jour les applications qui utilisent la surveillance des opérations

Les schémas de surveillance des opérations et les journaux de ressources varient légèrement. Il est important que vous mettiez à jour les applications qui utilisent la surveillance des opérations dès aujourd’hui afin qu’elles mappent au schéma utilisé par les journaux de ressources.

De plus, les journaux de ressources offrent cinq nouvelles catégories de suivi. Une fois que vous avez mis à jour les applications pour le schéma existant, ajoutez aussi les nouvelles catégories :

* Opérations jumelles cloud à appareil
* Opérations jumelles appareil à cloud
* Requêtes de jumeaux
* Opérations de travaux
* Méthodes directes

Pour connaître les structures de schéma spécifiques, consultez [Journaux de ressources](monitor-iot-hub-reference.md#resource-logs).

## <a name="monitoring-device-connect-and-disconnect-events-with-low-latency"></a>Supervision des événements de connexion et de déconnexion des appareils avec une faible latence

Pour surveiller les événements de connexion et de déconnexion des appareils dans les environnements de production, nous vous recommandons de vous abonner à [l’événement **Appareil déconnecté**](iot-hub-event-grid.md#event-types) sur Event Grid pour recevoir des alertes et suivre l’état de connexion de l’appareil. Suivez ce [tutoriel](iot-hub-how-to-order-connection-state-events.md) pour savoir comment intégrer les événements Appareil connecté et Appareil déconnecté d’IoT Hub à votre solution IoT.

## <a name="next-steps"></a>Étapes suivantes

[Superviser avec IoT Hub](monitor-iot-hub.md)