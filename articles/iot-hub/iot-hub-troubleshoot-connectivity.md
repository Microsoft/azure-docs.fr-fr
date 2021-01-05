---
title: Analyser et résoudre les problèmes de déconnexion d’Azure IoT Hub
description: Apprendre à analyser et résoudre les erreurs courantes en matière de connectivité des appareils pour Azure IoT Hub
author: jlian
manager: briz
ms.service: iot-hub
services: iot-hub
ms.topic: conceptual
ms.date: 11/06/2020
ms.author: jlian
ms.custom:
- mqtt
- 'Role: Cloud Development'
- 'Role: IoT Device'
- 'Role: Technical Support'
- fasttrack-edit
- iot
ms.openlocfilehash: 81558526500f3c01e975d9a963b4a6a98ec6d753
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96446411"
---
# <a name="monitor-diagnose-and-troubleshoot-disconnects-with-azure-iot-hub"></a>Analyser et résoudre les problèmes de déconnexion avec Azure IoT Hub

Les problèmes de connectivité des appareils IoT sont parfois difficiles à résoudre, car il existe de nombreux points de défaillance possibles. La logique d’application, les réseaux physiques, les protocoles, le matériel, IoT Hub et d'autres services cloud sont tous susceptibles d’occasionner des problèmes. La possibilité de détecter et d’identifier la source d’un problème est essentielle. Cela étant, une solution IoT à grande échelle peut impliquer des milliers d’appareils et il n'est pas pratique de vérifier chaque appareil individuellement. IoT Hub intègre deux services Azure pour vous aider à :

* **Azure Monitor** Pour vous aider à détecter, diagnostiquer et résoudre ces problèmes à grande échelle, utilisez les fonctionnalités de supervision offertes par IoT Hub via Azure Monitor. Cela comprend la configuration des alertes pour déclencher des notifications et des actions lorsque des déconnexions se produisent, et la configuration des journaux que vous pouvez utiliser pour découvrir les conditions qui ont provoqué ces déconnexions.

* **Azure Event Grid** : pour l’infrastructure critique et les déconnexions par appareil, utilisez Azure Event Grid pour vous abonner aux événements de connexion et de déconnexion des appareils émis par IoT Hub.

Dans les deux cas, ces fonctionnalités se limitant à ce que IoT Hub peut observer, nous vous recommandons également de suivre les meilleures pratiques de supervision pour vos appareils et autres services Azure.

## <a name="event-grid-vs-azure-monitor"></a>Event Grid ou Azure Monitor

Azure Event Grid offre une solution de supervision par appareil à faible latence qui vous permet de suivre les connexions pour les appareils critiques et l’infrastructure. Azure Monitor propose une métrique, *Appareils connectés*, que vous pouvez utiliser pour superviser le nombre d’appareils connectés à votre instance IoT Hub et déclencher une alerte dès que ce nombre descend en dessous d’un seuil statique.

Tenez compte des éléments suivants lorsque vous décidez d’utiliser Event Grid ou Azure Monitor pour un scénario particulier :

* Latence de l'alerte : les événements de connexion IoT Hub sont remis plus rapidement via Event Grid. Event Grid constitue donc un meilleur choix pour les scénarios où une notification rapide est préférable.

* Notifications par appareil : Event Grid offre la possibilité de suivre les connexions et les déconnexions pour des appareils individuels. Event Grid constitue donc un meilleur choix pour les scénarios où vous devez surveiller les connexions d’appareils critiques.

* Installation simplifiée : Les alertes de métrique Azure Monitor fournissent une expérience d’installation simplifiée qui ne nécessite pas d’intégration avec d’autres services pour envoyer des notifications par courrier électronique, SMS, voix et autres notifications.  Avec Event Grid, vous devez intégrer d’autres services Azure pour remettre des notifications. Ces deux services peuvent s’intégrer à d’autres services pour déclencher des actions plus complexes.

En raison de ses capacités de faible latence par appareil, pour les environnements de production, nous vous recommandons fortement d’utiliser Event Grid afin de surveiller les connexions. Bien entendu, ce choix n’est pas exclusif et vous pouvez utiliser les alertes de métrique Azure Monitor et Event Grid. Quel que soit votre choix pour le suivi des déconnexions, vous utiliserez probablement des journaux de ressources Azure Monitor pour vous aider à résoudre les problèmes de déconnexions inattendues d’appareils. Les sections suivantes décrivent chacune de ces étapes plus en détail.

## <a name="event-grid-monitor-device-connect-and-disconnect-events"></a>Event Grid : Superviser des événements de connexion et de déconnexion des appareils

Pour surveiller les événements de connexion et de déconnexion des appareils dans les environnements de production, nous vous recommandons de vous abonner aux [événements **DeviceConnected** et **DeviceDisconnected**](iot-hub-event-grid.md#event-types) sur Event Grid pour déclencher des alertes et suivre l’état de connexion de l’appareil. Event Grid offre une latence bien moindre à celle d’Azure Monitor, et vous pouvez assurer une supervision par appareil et non sur le nombre total d’appareils connectés. Ces aspects font d’Event Grid la méthode recommandée pour la supervision des appareils critiques et de l’infrastructure.

Lorsque vous utilisez Event Grid pour surveiller ou déclencher des alertes en cas de déconnexion d’un appareil, veillez à créer une méthode de filtrage des déconnexions périodiques dues à un renouvellement de jeton SAS sur les appareils qui utilisent les kits de développement logiciel (SDK) Azure IoT. Pour plus d’informations, consultez [Comportement de déconnexion d’appareil MQTT avec les kits de développement logiciel (SDK) Azure IoT](#mqtt-device-disconnect-behavior-with-azure-iot-sdks).

Explorez les rubriques suivantes pour en savoir plus sur la surveillance des événements de connexion des appareils avec Event Grid :

* Pour obtenir une vue d’ensemble de l’utilisation d’Event Grid avec IoT Hub, consultez [Réagir aux événements IoT Hub avec Event Grid](iot-hub-event-grid.md). Prêtez une attention particulière à la section [Limitations pour les événements d’état de la connexion et de la déconnexion d’appareils](iot-hub-event-grid.md#limitations-for-device-connected-and-device-disconnected-events).

* Pour consulter un tutoriel sur la commande des événements de connexion d’appareils, consultez [Commander des événements de connexion d’appareils depuis Azure IoT Hub à l’aide d’Azure Cosmos DB](iot-hub-how-to-order-connection-state-events.md).

* Pour consulter un tutoriel sur l’envoi de notifications par e-mail, consultez [Envoyer des notifications par e-mail concernant des événements Azure IoT Hub à l’aide d’Event Grid et de Logic Apps](../event-grid/publish-iot-hub-events-to-logic-apps.md) dans la documentation Event Grid.

## <a name="azure-monitor-route-connection-events-to-logs"></a>Azure Monitor : acheminer les événements de connexion vers les journaux

IoT Hub émet en continu des journaux de ressources pour plusieurs catégories d’opérations. Toutefois, pour collecter ces données de journal, vous devez créer un paramètre de diagnostic afin de les acheminer vers une destination où elles peuvent être analysées ou archivées. L’une de ces destinations est Azure Monitor Logs via un espace de travail Log Analytics ([voir la tarification](https://azure.microsoft.com/pricing/details/log-analytics/)), où vous pouvez analyser les données à l’aide de requêtes Kusto.

La [catégorie des connexions des journaux de ressources](monitor-iot-hub-reference.md#connections) d’IoT Hub émet des opérations et des erreurs liées aux connexions des appareils. La capture d’écran suivante montre un paramètre de diagnostic permettant d’acheminer ces journaux vers un espace de travail Log Analytics :

:::image type="content" source="media/iot-hub-troubleshoot-connectivity/create-diagnostic-setting.png" alt-text="Paramètre recommandé pour envoyer des journaux de connectivité à l’espace de travail Log Analytics.":::

Nous vous recommandons de créer un paramètre de diagnostic le plus tôt possible après avoir créé votre IoT Hub, car, même si IoT Hub émet toujours les journaux de ressources, ceux-ci ne sont pas collectés par Azure Monitor tant que vous ne les acheminez pas vers une destination.

Pour en savoir plus sur le routage des journaux vers une destination, consultez [Collecte et routage](monitor-iot-hub.md#collection-and-routing). Pour obtenir des instructions détaillées sur la création d’un paramètre de diagnostic, consultez le tutoriel [Utiliser les métriques et les journaux](tutorial-use-metrics-and-diags.md).

## <a name="azure-monitor-set-up-metric-alerts-for-device-disconnect-at-scale"></a>Azure Monitor : Configurer des alertes de métrique en cas de déconnexion d'appareils à grande échelle

Vous pouvez configurer des alertes basées sur les métriques de plateforme émises par IoT Hub. Les alertes de métrique vous permettent d’avertir les utilisateurs qu’une condition importante s’est produite et de déclencher des actions pour répondre automatiquement à cette condition.

La métrique [*Appareils connectés (préversion)*](monitor-iot-hub-reference.md#device-metrics) vous indique le nombre d’appareils connectés à IoT Hub. Vous pouvez créer des alertes à déclencher si cette métrique descend en dessous d’une valeur de seuil :

:::image type="content" source="media/iot-hub-troubleshoot-connectivity/configure-alert-logic.png" alt-text="Paramètres de logique d’alerte pour la métrique des appareils connectés.":::

Vous pouvez utiliser des règles d’alerte de métrique pour surveiller les anomalies de déconnexion d’appareil à grande échelle. Autrement dit, lorsqu’un nombre significatif d’appareils se déconnectent de façon inattendue. Quand une telle occurrence est détectée, vous pouvez consulter les journaux pour vous aider à résoudre le problème. Toutefois, vous devez utiliser Event Grid pour surveiller les déconnexions par appareil et les déconnexions d’appareils critiques. Event Grid offre également une meilleure expérience en temps réel que les métriques Azure.

Pour en savoir plus sur les alertes avec IoT Hub, consultez [Alertes dans Superviser IoT Hub](monitor-iot-hub.md#alerts). Pour obtenir une procédure pas à pas de création d’alertes dans IoT Hub, consultez le tutoriel [Utiliser les métriques et les journaux](tutorial-use-metrics-and-diags.md). Pour obtenir une vue d’ensemble plus détaillée des alertes, consultez [vue d’ensemble des alertes dans Microsoft Azure](../azure-monitor/platform/alerts-overview.md) dans la documentation Azure Monitor.

## <a name="azure-monitor-use-logs-to-resolve-connectivity-errors"></a>Azure Monitor : utiliser les journaux pour résoudre les erreurs de connectivité

Lorsque vous détectez des déconnexions d’appareils, que ce soit avec des alertes de métrique Azure Monitor ou avec Event Grid, vous pouvez utiliser les journaux pour résoudre le problème. Cette section décrit comment examiner les problèmes courants dans Azure Monitor Logs. La procédure ci-dessous part du principe que vous avez déjà créé un [paramètre de diagnostic](#azure-monitor-route-connection-events-to-logs) pour envoyer les journaux des connexions IoT Hub à un espace de travail Log Analytics.

Une fois que vous avez créé un paramètre de diagnostic pour acheminer des journaux de ressources IoT Hub vers Azure Monitor Logs, procédez comme suit pour afficher les journaux dans le portail Azure.

1. Accédez à IoT hub dans le [portail Azure](https://portal.azure.com).

1. Sous **Supervision** dans le volet gauche d’IoT hub, sélectionnez **Journaux**.

1. Pour isoler les journaux d’activité d’erreurs de connectivité pour IoT Hub, entrez la requête suivante dans l’éditeur de requête, puis sélectionnez **Exécuter** :

    ```kusto
    AzureDiagnostics
    | where ( ResourceType == "IOTHUBS" and Category == "Connections" and Level == "Error")
    ```

1. Si des résultats s’affichent, recherchez les éléments `OperationName`, `ResultType` (code d’erreur) et `ResultDescription` (message d’erreur) pour obtenir plus de détails sur l’erreur.

   ![Exemple de journal des erreurs](./media/iot-hub-troubleshoot-connectivity/diag-logs.png)

Une fois que vous avez identifié l’erreur, suivez les guides de résolution des problèmes pour obtenir de l’aide sur les erreurs les plus courantes :

* [400027 ConnectionForcefullyClosedOnNewConnection](iot-hub-troubleshoot-error-400027-connectionforcefullyclosedonnewconnection.md)

* [404104 DeviceConnectionClosedRemotely](iot-hub-troubleshoot-error-404104-deviceconnectionclosedremotely.md)

* [401003 IoTHubUnauthorized](iot-hub-troubleshoot-error-401003-iothubunauthorized.md)

* [409002 LinkCreationConflict](iot-hub-troubleshoot-error-409002-linkcreationconflict.md)

* [500001 ServerError](iot-hub-troubleshoot-error-500xxx-internal-errors.md)

* [500008 GenericTimeout](iot-hub-troubleshoot-error-500xxx-internal-errors.md)

## <a name="mqtt-device-disconnect-behavior-with-azure-iot-sdks"></a>Comportement de déconnexion d’appareil MQTT avec les kits de développement logiciel (SDK) Azure IoT

Les kits de développement logiciel (SDK) d’appareil Azure IoT se déconnectent d’IoT Hub puis se reconnectent lorsqu’ils renouvellent les jetons SAS via le protocole MQTT (et MQTT via WebSockets). Dans les journaux, ces informations s’affichent comme des événements de déconnexion et de connexion, parfois accompagnés d’événements d’erreur.

Par défaut, la durée de vie des jetons est de 60 minutes pour tous les kits de développement logiciel (SDK) ; toutefois, cette valeur peut être modifiée par les développeurs dans certains SDK. Le tableau suivant résume la durée de vie des jetons, le renouvellement des jetons et le comportement du renouvellement des jetons pour chacun des kits de développement logiciel (SDK) :

| Kit SDK | Durée de vie des jetons | Renouvellement des jetons | Comportement du renouvellement |
|-----|----------|---------------------|---------|
| .NET | 60 minutes, configurable | 85 % de la durée de vie, configurable | Le kit de développement logiciel (SDK) se connecte et se déconnecte dès la fin de vie du jeton plus une période de grâce de 10 minutes. Événements d’information et erreurs générés dans les journaux. |
| Java | 60 minutes, configurable | 85 % de la durée de vie, non configurable | Le kit de développement logiciel (SDK) se connecte et se déconnecte dès la fin de vie du jeton plus une période de grâce de 10 minutes. Événements d’information et erreurs générés dans les journaux. |
| Node.js | 60 minutes, configurable | configurable | Le SDK se connecte et se déconnecte lors du renouvellement du jeton. Seuls les événements d’information sont générés dans les journaux. |
| Python | 60 minutes, non configurable | -- | Le SDK se connecte et se déconnecte à la fin de la durée de vie du jeton. |

Les captures d’écran suivantes montrent le comportement de renouvellement des jetons dans les journaux Azure Monitor pour différents kits de développement logiciel (SDK). La durée de vie et le seuil de renouvellement des jetons ont été modifiés par rapport à leurs valeurs par défaut, comme indiqué.

* Le kit de développement logiciel (SDK) pour appareils .NET avec une durée de vie et un renouvellement de jeton de 1 200 secondes (20 minutes) est défini à 90 % de la durée de vie. Les déconnexions se produisent toutes les 30 minutes :

    :::image type="content" source="media/iot-hub-troubleshoot-connectivity/net-mqtt.png" alt-text="Comportement d’erreur pour le renouvellement de jetons via MQTT dans Azure Monitor Logs avec le kit de développement logiciel (SDK) .NET.":::

* Le Kit de développement logiciel (SDK) Java avec une durée de vie de jeton de 300 secondes (5 minutes) et un renouvellement de durée de vie par défaut de 85 %. Les déconnexions se produisent toutes les 15 minutes :

    :::image type="content" source="media/iot-hub-troubleshoot-connectivity/java-mqtt.png" alt-text="Comportement d’erreur pour le renouvellement de jetons via MQTT dans Azure Monitor Logs avec le kit de développement logiciel (SDK) Java.":::

* Le kit de développement logiciel (SDK) Node avec une durée de vie et un renouvellement de jeton de 300 secondes (5 minutes) est défini à 3 minutes. Les déconnexions se produisent lors du renouvellement du jeton. En outre, il n’y a pas d’erreur ; seuls les événements de connexion/déconnexion informationnels sont émis :

    :::image type="content" source="media/iot-hub-troubleshoot-connectivity/node-mqtt.png" alt-text="Comportement d’erreur pour le renouvellement de jetons via MQTT dans Azure Monitor Logs avec le kit de développement logiciel (SDK) Node.":::

La requête suivante a été utilisée pour collecter les résultats. La requête extrait le nom et la version du kit de développement logiciel (SDK) à partir du conteneur de propriétés. Pour plus d’informations, consultez [Version du kit de développement logiciel dans les journaux IoT Hub](monitor-iot-hub.md#sdk-version-in-iot-hub-logs).

```kusto
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.DEVICES" and ResourceType == "IOTHUBS"
| where Category == "Connections"
| extend parsed_json = parse_json(properties_s)
| extend SDKVersion = tostring(parsed_json.sdkVersion) , DeviceId = tostring(parsed_json.deviceId) , Protocol =  tostring(parsed_json.protocol)
| distinct TimeGenerated, OperationName, Level, ResultType, ResultDescription, DeviceId, Protocol, SDKVersion

```

En tant que développeur ou opérateur de solutions IoT, vous devez être conscient de ce comportement afin d’interpréter les événements de connexion/déconnexion et les erreurs associées consignées dans les journaux. Si vous souhaitez modifier la durée de vie ou le comportement de renouvellement des jetons pour les appareils, vérifiez que l’appareil implémente un paramètre de jumeau d’appareil ou une méthode d’appareil qui rend cela possible.

Si vous surveillez les connexions d’appareils avec Event Hub, assurez-vous de créer une méthode de filtrage des déconnexions périodiques dues à un renouvellement de jeton SAP, par exemple en ne déclenchant pas d’actions basées sur des déconnexions tant que l’événement de déconnexion est suivi d’un événement de connexion dans un intervalle de temps donné.

> [!NOTE]
> IoT Hub ne prend en charge qu’une seule connexion MQTT active par appareil. Toute nouvelle connexion MQTT au nom du même ID d’appareil entraîne l’interruption de la connexion existante par IoT Hub.
>
> Le message 400027 ConnectionForcefullyClosedOnNewConnection sera consigné dans les journaux IoT Hub

## <a name="i-tried-the-steps-but-they-didnt-work"></a>J’ai suivi les différentes étapes, mais cela n'a pas fonctionné.

Si les étapes ci-dessus n’ont pas permis de résoudre le problème, essayez ce qui suit :

* Si vous avez accès aux appareils problématiques, que ce soit physiquement ou à distance (comme SSH), consultez le [guide de résolution des problèmes côté appareil](https://github.com/Azure/azure-iot-sdk-node/wiki/Troubleshooting-Guide-Devices) pour poursuivre le dépannage.

* Vérifiez que vos appareils présentent l’état **Activé** en accédant au portail Azure > votre hub IoT > Appareils IoT.

* Si votre appareil utilise le protocole MQTT, vérifiez que le port 8883 est ouvert. Pour plus d’informations, consultez [Connexion à IoT Hub (MQTT)](iot-hub-mqtt-support.md#connecting-to-iot-hub).

* Obtenez de l’aide sur [Page de questions Microsoft Q&A sur Azure IoT Hub](/answers/topics/azure-iot-hub.html), [Stack Overflow](https://stackoverflow.com/questions/tagged/azure-iot-hub)ou [support Azure](https://azure.microsoft.com/support/options/).

Pour contribuer à améliorer la documentation à l’intention de tous les utilisateurs, laissez un commentaire dans la section des commentaires ci-après si ce guide ne vous a pas aidé.

## <a name="next-steps"></a>Étapes suivantes

* Pour en savoir plus sur la résolution des problèmes temporaires, consultez [Gestion des erreurs temporaires](/azure/architecture/best-practices/transient-faults).

* Pour explorer davantage le SDK Azure IoT et la gestion des nouvelles tentatives, consultez [Guide pratique pour gérer la connectivité et la messagerie fiable à l’aide des kits Azure IoT Hub device SDK](iot-hub-reliability-features-in-sdks.md#connection-and-retry).