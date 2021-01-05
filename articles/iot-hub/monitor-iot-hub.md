---
title: Surveillance d’Azure IoT Hub
description: Commencer ici pour découvrir comment surveiller Azure IoT Hub
author: robinsh
ms.author: robinsh
ms.topic: conceptual
ms.service: iot-hub
ms.date: 11/06/2020
ms.openlocfilehash: 2fabf75f74ccf57f57e7b47720ceda8885420842
ms.sourcegitcommit: 6a350f39e2f04500ecb7235f5d88682eb4910ae8
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/01/2020
ms.locfileid: "96460904"
---
# <a name="monitoring-azure-iot-hub"></a>Surveillance d’Azure IoT Hub

Lorsque vous avez des applications critiques et des processus métier basés sur des ressources Azure, vous voulez superviser ces ressources pour connaître leur disponibilité, leurs performances et leur fonctionnement. Cet article décrit les données de supervision générées par Azure IoT Hub et comment vous pouvez utiliser les fonctionnalités d’Azure Monitor pour analyser ces données et créer des alertes.

## <a name="monitor-overview"></a>Présentation de Monitor

La page **Vue d’ensemble** sur le portail Azure pour chaque hub IoT inclut des graphiques qui fournissent des métriques d’utilisation, telles que le nombre de messages utilisés et le nombre d’appareils connectés au hub IoT.

:::image type="content" source="media/monitor-iot-hub/overview-portal.png" alt-text="Graphiques des métriques par défaut sur la page récapitulative du hub IoT.":::

Sachez que la valeur du nombre de messages peut être retardée d’une minute et que, pour des raisons liées à l’infrastructure de service IoT Hub, la valeur peut parfois rebondir entre des valeurs supérieures et inférieures lors de l’actualisation. Ce compteur ne doit être incorrect que pour les valeurs accumulées au cours de la dernière minute.

Les informations présentées dans le volet Vue d’ensemble sont utiles, mais elles ne constituent qu’une petite quantité des données de supervision disponibles pour un hub IoT. Certaines données de surveillance sont collectées automatiquement et peuvent être analysées dès que vous créez le hub IoT. Vous pouvez activer d’autres types de collecte de données avec une certaine configuration.

## <a name="what-is-azure-monitor"></a>Qu’est-ce qu’Azure Monitor ?

Azure IoT Hub crée des données de supervision avec [Azure Monitor](../azure-monitor/overview.md), un service de supervision de pile complète dans Azure qui fournit un ensemble complet de fonctionnalités pour superviser vos ressources Azure en plus des ressources figurant dans d’autres clouds et sur site.

Commencez avec l’article [Supervision de ressources Azure avec Azure Monitor](../azure-monitor/insights/monitor-azure-resource.md), qui décrit les concepts suivants :

- Qu’est-ce qu’Azure Monitor ?
- Coûts associés à la supervision
- Analyse des données collectées dans Azure
- Configuration de la collecte des données
- Outils standard dans Azure pour l’analyse et la génération d’alertes sur les données analysées

Les sections suivantes s’appuient sur cet article en décrivant les données spécifiques collectées à partir d’Azure IoT Hub et en fournissant des exemples de configuration de collecte de données et d’analyse de ces données avec les outils Azure.

## <a name="monitoring-data"></a>Données de surveillance

Azure IoT Hub collecte les mêmes types de données de supervision que d’autres ressources Azure, lesquelles sont décrites dans [Supervision des données de ressources Azure](../azure-monitor/insights/monitor-azure-resource.md#monitoring-data).

Pour obtenir des informations détaillées sur les métriques et les métriques de journaux créées par le service Azure IoT Hub, consultez [Informations de référence sur les données de supervision d’Azure IoT Hub](monitor-iot-hub-reference.md).

> [!IMPORTANT]
> Ni la fiabilité ni l’ordre des événements émis par le service IoT Hub à l’aide des journaux de ressources Azure Monitor ne sont garantis. Certains événements peuvent être perdus ou remis de manière désordonnée. De même, les journaux de ressources ne sont pas censés être en temps réel, et l’enregistrement des événements dans la destination de votre choix peut prendre plusieurs minutes.

## <a name="collection-and-routing"></a>Collecte et routage

Les métriques de plateforme et le journal d’activité sont collectés et stockés automatiquement, mais ils peuvent être acheminés vers d’autres emplacements à l’aide d’un paramètre de diagnostic.

Les journaux de ressources ne sont pas collectés ni stockés tant que vous n’avez pas créé un paramètre de diagnostic et que vous ne les acheminez pas vers un ou plusieurs emplacements.

Les métriques et les journaux peuvent être acheminés vers plusieurs emplacements, notamment :
- Le magasin Journaux Azure Monitor via un espace de travail Log Analytics associé. Ils peuvent y être analysés à l’aide de Log Analytics.
- Stockage Azure pour l’archivage et l’analyse hors connexion 
- Un point de terminaison Event Hubs dans lequel ils peuvent être lus par des applications externes, par exemple des outils SIEM tiers.

Sur le portail Azure, vous pouvez sélectionner **Paramètres de diagnostic** sous **Surveillance** dans le volet de gauche de votre hub IoT, puis **Ajouter un paramètre de diagnostic** pour créer des paramètres de diagnostic étendus aux métriques de journaux et de plateforme émises par votre hub IoT.

La capture d’écran suivante montre un paramètre de diagnostic pour le routage du type de journal de ressources *Opérations de connexion* et de toutes les métriques de plateforme vers un espace de travail Log Analytics.

:::image type="content" source="media/monitor-iot-hub/diagnostic-setting-portal.png" alt-text="Volet Paramètres de diagnostic pour un hub IoT.":::

Pour plus d’informations sur la création d’un paramètre de diagnostic à l’aide du portail Azure, de l’interface CLI ou de PowerShell, consultez [Créer un paramètre de diagnostic pour collecter des journaux et métriques de plateforme dans Azure](../azure-monitor/platform/diagnostic-settings.md). Lorsque vous créez un paramètre de diagnostic, vous spécifiez les catégories de journaux à collecter. Les catégories pour Azure IoT Hub sont répertoriées sous [Journaux des ressources dans les informations de référence des données de surveillance d’Azure IoT Hub](monitor-iot-hub-reference.md#resource-logs).

Lors du routage de la plateforme IoT Hub vers d’autres emplacements, rappelez-vous que :

- Les métriques de plateforme suivantes ne sont pas exportables par le biais des paramètres de diagnostic : *Périphériques connectés (version préliminaire)* et *Nombre total d’appareils (version préliminaire)* .

- Les métriques multidimensionnelles, par exemple certaines [métriques de routage](monitor-iot-hub-reference.md#routing-metrics), sont actuellement exportées sous forme de métriques unidimensionnelles aplaties agrégées entre les valeurs de dimension. Pour plus d’informations, consultez [Exportation des métriques de plateforme vers d’autres emplacements](../azure-monitor/platform/metrics-supported.md#exporting-platform-metrics-to-other-locations).

## <a name="analyzing-metrics"></a>Analyse des métriques

Vous pouvez analyser les métriques d’Azure IoT Hub avec les métriques d’autres services Azure à l’aide de Metrics Explorer en ouvrant **Métriques** dans le menu **Azure Monitor**. Pour plus d’informations sur l’utilisation de cet outil, consultez [Prise en main d’Azure Metrics Explorer](../azure-monitor/platform/metrics-getting-started.md).

Dans le portail Azure, vous pouvez sélectionner **Métriques** sous **Surveillance** dans le volet de gauche de votre hub IoT pour ouvrir Metrics explorer étendu, par défaut, aux métriques de la plateforme émises par votre hub IoT :

:::image type="content" source="media/monitor-iot-hub/metrics-portal.png" alt-text="Page de Metrics Explorer pour un hub IoT.":::

Pour obtenir la liste des métriques de plateforme collectées pour Azure IoT Hub, consultez [Métriques dans les informations de référence des données de surveillance Azure IoT Hub](monitor-iot-hub-reference.md#metrics). Pour obtenir la liste des métriques de plateforme collectées pour tous les services Azure, consultez [Métriques prises en charge avec Azure Monitor](../azure-monitor/platform/metrics-supported.md).

Pour les métriques de la plateforme IoT Hub qui sont collectées à l’unité, certaines agrégations peuvent ne pas être disponibles ou utilisables. Pour plus d’informations, consultez [Agrégations prises en charge dans les informations de référence des données de surveillance Azure IoT Hub](monitor-iot-hub-reference.md#supported-aggregations).

Certaines mesures IoT Hub, comme les [métriques de routage](monitor-iot-hub-reference.md#routing-metrics), sont multidimensionnelles. Pour ces métriques, vous pouvez appliquer des [filtres](../azure-monitor/platform/metrics-charts.md#apply-filters-to-charts) et un [fractionnement](../azure-monitor/platform/metrics-charts.md#apply-splitting-to-a-chart) sur vos graphiques en fonction d’une dimension.

## <a name="analyzing-logs"></a>Analyse des journaux d’activité

Les données des journaux Azure Monitor sont stockées dans des tables, chacune ayant son propre ensemble de propriétés uniques. Les données de ces tables sont associées à un espace de travail Log Analytics et peuvent être interrogées dans Log Analytics. Pour en savoir plus sur Azure Monitor Logs, consultez [Vue d’ensemble d’Azure Monitor Logs](../azure-monitor/platform/data-platform-logs.md) dans la documentation Azure Monitor. 

Pour router des données vers Azure Monitor Logs, vous devez créer un paramètre de diagnostic pour envoyer les journaux de ressources ou les métriques de plateforme à un espace de travail Log Analytics. Pour plus d’informations, consultez [Collecte et routage](#collection-and-routing).

Dans le portail Azure, vous pouvez sélectionner **Journaux** sous **Surveillance** dans le volet de gauche de votre hub IoT pour effectuer des requêtes Log Analytics étendues, par défaut, aux journaux et aux métriques collectés dans Azure Monitor Logs pour votre hub IoT.

:::image type="content" source="media/monitor-iot-hub/logs-portal.png" alt-text="Page des journaux pour un hub IoT.":::

Pour obtenir la liste des tables utilisées par Azure Monitor Logs et interrogeables par Log Analytics, consultez [Tableaux Azure Monitor Logs dans les informations de référence des données de surveillance Azure IoT Hub](monitor-iot-hub-reference.md#azure-monitor-logs-tables).

Tous les journaux de ressources dans Azure Monitor ont les mêmes champs suivis de champs spécifiques au service. Le schéma commun est décrit dans [Schéma des journaux des ressources Azure Monitor](../azure-monitor/platform/resource-logs-schema.md#top-level-common-schema). Vous pouvez trouver le schéma et les catégories de journaux de ressources collectés pour Azure IoT Hub dans [Journaux des ressources dans les informations de référence des données de surveillance Azure IoT Hub](monitor-iot-hub-reference.md#resource-logs).

Le [journal d’activité](../azure-monitor/platform/activity-log.md) est un journal de plateforme dans Azure qui fournit des insights de tous les événements de niveau abonnement. Vous pouvez l’afficher indépendamment ou le router vers Azure Monitor Logs, où vous pouvez effectuer des requêtes bien plus complexes à l’aide de Log Analytics.  

Lors du routage de la plateforme IoT Hub vers Azure Monitor Logs, rappelez-vous que :

- Les métriques de plateforme suivantes ne sont pas exportables par le biais des paramètres de diagnostic : *Périphériques connectés (version préliminaire)* et *Nombre total d’appareils (version préliminaire)* .

- Les métriques multidimensionnelles, par exemple certaines [métriques de routage](monitor-iot-hub-reference.md#routing-metrics), sont actuellement exportées sous forme de métriques unidimensionnelles aplaties agrégées entre les valeurs de dimension. Pour plus d’informations, consultez [Exportation des métriques de plateforme vers d’autres emplacements](../azure-monitor/platform/metrics-supported.md#exporting-platform-metrics-to-other-locations).

Pour certaines requêtes courantes avec IoT Hub, consultez [Exemples de requêtes Kusto](#sample-kusto-queries). Pour des informations détaillées sur les requêtes Log Analytics, consultez [Vue d’ensemble des requêtes de journal dans Azure Monitor](../azure-monitor/log-query/log-query-overview.md).

### <a name="sdk-version-in-iot-hub-logs"></a>Version du kit de développement logiciel dans les journaux IoT Hub

Certaines opérations dans les journaux de ressources IoT Hub retournent une propriété `sdkVersion` dans leur objet `properties`. Pour ces opérations, quand une application d’appareil ou back-end utilise l’un des kits SDK Azure IoT, cette propriété contient des informations sur le kit SDK utilisé, sa version et la plateforme sur laquelle il s’exécute. L’exemple suivant montre la propriété `sdkVersion` émise pour une opération [`deviceConnect`](monitor-iot-hub-reference.md#connections) lors de l’utilisation du kit SDK d’appareil Node.js : `"azure-iot-device/1.17.1 (node v10.16.0; Windows_NT 10.0.18363; x64)"`. Voici un exemple de la valeur émise pour le kit SDK .NET (C#) : `".NET/1.21.2 (.NET Framework 4.8.4200.0; Microsoft Windows 10.0.17763 WindowsProduct:0x00000004; X86)"`.

Le tableau suivant indique le nom du kit SDK utilisé pour différents kits SDK Azure IoT :

| Nom du kit SDK dans la propriété sdkVersion | Langage |
|----------|----------|
| .NET | .NET (C#) |
| microsoft.azure.devices | Kit SDK de service .NET (C#) |
| microsoft.azure.devices.client | Kit SDK d’appareil .NET (C#) |
| iothubclient | Kit SDK d’appareil C ou Python v1 (obsolète) |
| iothubserviceclient | Kit SDK de service C ou Python v1 (obsolète) |
| azure-iot-device-iothub-py | Kit SDK d’appareil Python |
| azure-iot-device | Kit SDK d’appareil Node.js |
| azure-iothub | Kit SDK de service Node.js |
| com.microsoft.azure.iothub-java-client | Kit SDK d’appareil Java |
| com.microsoft.azure.iothub.service.sdk | Kit SDK de service Java |
| com.microsoft.azure.sdk.iot.iot-device-client | Kit SDK d’appareil Java |
| com.microsoft.azure.sdk.iot.iot-service-client | Kit SDK de service Java |
| C | Embedded C |
| C + (OSSimplified = Azure RTOS) | Azure RTOS |

Vous pouvez extraire la propriété de version du kit SDK lorsque vous exécutez des requêtes sur des journaux de ressources IoT Hub. À titre d’exemple, la requête suivante extrait la propriété de version du kit SDK (et l’ID de l’appareil) à partir des propriétés retournées par les opérations de connexion. Ces deux propriétés sont écrites dans les résultats avec l’heure de l’opération et l’ID de ressource du hub IoT auquel l’appareil se connecte.

```kusto
// SDK version of devices
// List of devices and their SDK versions that connect to IoT Hub
AzureDiagnostics
| where ResourceProvider == "MICROSOFT.DEVICES" and ResourceType == "IOTHUBS"
| where Category == "Connections"
| extend parsed_json = parse_json(properties_s) 
| extend SDKVersion = tostring(parsed_json.sdkVersion) , DeviceId = tostring(parsed_json.deviceId)
| distinct DeviceId, SDKVersion, TimeGenerated, _ResourceId
```

### <a name="sample-kusto-queries"></a>Exemples de requêtes Kusto

> [!IMPORTANT]
> Quand vous sélectionnez **Journaux** dans le menu du hub IoT, Log Analytics est ouvert avec l’étendue de requête définie sur le hub IoT actuel. Cela signifie que les requêtes de journal n’incluront que les données de cette ressource. Si vous voulez exécuter une requête qui inclut des données provenant d’autres hubs IoT ou d’autres services Azure, sélectionnez **Journaux** dans le menu **Azure Monitor**. Pour plus d’informations, consultez [Étendue de requête de journal et intervalle de temps dans la fonctionnalité Log Analytics d’Azure Monitor](../azure-monitor/log-query/scope.md).

Voici des requêtes que vous pouvez utiliser pour vous aider à superviser votre hub IoT.

- Erreurs de connectivité : Identifiez les erreurs de connexion des appareils.

    ```kusto
    AzureDiagnostics
    | where ResourceProvider == "MICROSOFT.DEVICES" and ResourceType == "IOTHUBS"
    | where Category == "Connections" and Level == "Error"
    ```

- Erreurs de limitation : Identifiez les appareils qui ont effectué le plus de requêtes et engendré des erreurs de limitation.

    ```kusto
    AzureDiagnostics
    | where ResourceProvider == "MICROSOFT.DEVICES" and ResourceType == "IOTHUBS"
    | where ResultType == "429001"
    | extend DeviceId = tostring(parse_json(properties_s).deviceId)
    | summarize count() by DeviceId, Category, _ResourceId
    | order by count_ desc
    ```

- Points de terminaison inactifs : Identifiez les points de terminaison inactifs ou non sains en fonction du nombre de fois où le problème a été signalé, ainsi que de la raison de celui-ci.

    ```kusto
    AzureDiagnostics
    | where ResourceProvider == "MICROSOFT.DEVICES" and ResourceType == "IOTHUBS"
    | where Category == "Routes" and OperationName in ("endpointDead", "endpointUnhealthy")
    | extend parsed_json = parse_json(properties_s)
    | extend Endpoint = tostring(parsed_json.endpointName), Reason = tostring(parsed_json.details) 
    | summarize count() by Endpoint, OperationName, Reason, _ResourceId
    | order by count_ desc
    ```

- Récapitulatif des erreurs : Nombre d’erreurs sur l’ensemble des opérations par type.

    ```kusto
    AzureDiagnostics
    | where ResourceProvider == "MICROSOFT.DEVICES" and ResourceType == "IOTHUBS"
    | where Level == "Error"
    | summarize count() by ResultType, ResultDescription, Category, _ResourceId
    ```

- Appareils récemment connectés : Liste des appareils que IoT Hub a détectés comme connectés au cours de la période spécifiée.

    ```kusto
    AzureDiagnostics
    | where ResourceProvider == "MICROSOFT.DEVICES" and ResourceType == "IOTHUBS"
    | where Category == "Connections" and OperationName == "deviceConnect"
    | extend DeviceId = tostring(parse_json(properties_s).deviceId)
    | summarize max(TimeGenerated) by DeviceId, _ResourceId
    ```

- Version du kit de développement logiciel (SDK) des appareils : Liste des appareils et leurs versions de kit de développement logiciel (SDK) pour les connexions d’appareils ou les opérations jumelles de l’appareil vers le cloud.

    ```kusto
    AzureDiagnostics
    | where ResourceProvider == "MICROSOFT.DEVICES" and ResourceType == "IOTHUBS"
    | where Category == "Connections" or Category == "D2CTwinOperations"
    | extend parsed_json = parse_json(properties_s)
    | extend SDKVersion = tostring(parsed_json.sdkVersion) , DeviceId = tostring(parsed_json.deviceId)
    | distinct DeviceId, SDKVersion, TimeGenerated, _ResourceId
    ```

### <a name="read-logs-from-azure-event-hubs"></a>Lecture de journaux d’activité à partir d’Azure Event Hubs

Après avoir configuré la journalisation des événements via les paramètres de diagnostic, vous pouvez créer des applications qui lisent les journaux d’activité, pour vous permettre d’agir en fonction des informations qu’ils contiennent. Cet exemple de code extrait les journaux d’activité à partir d’un concentrateur d’événements :

```csharp
class Program
{ 
    static string connectionString = "{your AMS eventhub endpoint connection string}";
    static string monitoringEndpointName = "{your AMS event hub endpoint name}";
    static EventHubClient eventHubClient;
    //This is the Diagnostic Settings schema
    class AzureMonitorDiagnosticLog
    {
        string time { get; set; }
        string resourceId { get; set; }
        string operationName { get; set; }
        string category { get; set; }
        string level { get; set; }
        string resultType { get; set; }
        string resultDescription { get; set; }
        string durationMs { get; set; }
        string callerIpAddress { get; set; }
        string correlationId { get; set; }
        string identity { get; set; }
        string location { get; set; }
        Dictionary<string, string> properties { get; set; }
    };

    static void Main(string[] args)
    {
        Console.WriteLine("Monitoring. Press Enter key to exit.\n");
        eventHubClient = EventHubClient.CreateFromConnectionString(connectionString, monitoringEndpointName);
        var d2cPartitions = eventHubClient.GetRuntimeInformationAsync().PartitionIds;
        CancellationTokenSource cts = new CancellationTokenSource();
        var tasks = new List<Task>();
        foreach (string partition in d2cPartitions)
        {
            tasks.Add(ReceiveMessagesFromDeviceAsync(partition, cts.Token));
        }
        Console.ReadLine();
        Console.WriteLine("Exiting...");
        cts.Cancel();
        Task.WaitAll(tasks.ToArray());
    }

    private static async Task ReceiveMessagesFromDeviceAsync(string partition, CancellationToken ct)
    {
        var eventHubReceiver = eventHubClient.GetDefaultConsumerGroup().CreateReceiver(partition, DateTime.UtcNow);
        while (true)
        {
            if (ct.IsCancellationRequested)
            {
                await eventHubReceiver.CloseAsync();
                break;
            }
            EventData eventData = await eventHubReceiver.ReceiveAsync(new TimeSpan(0,0,10));
            if (eventData != null)
            {
                string data = Encoding.UTF8.GetString(eventData.GetBytes());
                Console.WriteLine("Message received. Partition: {0} Data: '{1}'", partition, data);
                var deserializer = new JavaScriptSerializer();
                //deserialize json data to azure monitor object
                AzureMonitorDiagnosticLog message = new JavaScriptSerializer().Deserialize<AzureMonitorDiagnosticLog>(result);
            }
        }
    }
}
```

## <a name="alerts"></a>Alertes

Azure Monitor vous avertit de façon proactive lorsque des conditions significatives sont détectées dans vos données de surveillance. Elles permettent d’identifier et de résoudre les problèmes affectant votre système avant que vos clients ne les remarquent. Vous pouvez définir des alertes sur des [métriques](../azure-monitor/platform/alerts-metric-overview.md), sur des [journaux](../azure-monitor/platform/alerts-unified-log.md) et sur le [journal d’activité](../azure-monitor/platform/activity-log-alerts.md). Les différents types d’alertes présentent des avantages et des inconvénients.

Au moment de la création d’une règle d’alerte basée sur les métriques de la plateforme IoT Hub qui sont collectées à l’unité, certaines agrégations peuvent ne pas être disponibles ou utilisables. Pour plus d’informations, consultez [Agrégations prises en charge dans les informations de référence des données de surveillance Azure IoT Hub](monitor-iot-hub-reference.md#supported-aggregations).

## <a name="monitor-per-device-disconnects-with-event-grid"></a>Superviser les déconnexions par appareil avec Event Grid

Azure Monitor propose une métrique, *Appareils connectés*, que vous pouvez utiliser pour superviser le nombre d’appareils connectés à votre instance IoT Hub et déclencher une alerte dès que ce nombre descend en dessous d’un certain seuil. Même si cela peut suffire pour certains scénarios, [Azure Event Grid](../event-grid/index.yml) offre une solution de supervision par appareil à faible latence qui vous permet de suivre les connexions pour les appareils critiques et l’infrastructure.

Avec Event Grid, vous pouvez vous abonner aux [événements IoT Hub **DeviceConnected** et **DeviceDisconnected**](iot-hub-event-grid.md#event-types) pour déclencher des alertes et superviser l’état de connexion des appareils. Event Grid offre une latence bien moindre à celle d’Azure Monitor, et vous pouvez assurer une supervision par appareil et non sur le nombre total d’appareils connectés. Ces aspects font d’Event Grid la méthode recommandée pour la supervision des connexions des appareils critiques et de l’infrastructure. Nous vous recommandons vivement d’utiliser Event Grid pour superviser les connexions d’appareils dans les environnements de production.

Pour plus d’informations sur la supervision des connexions d’appareils avec Event Grid et Azure Monitor, consultez [Analyser et résoudre les problèmes de déconnexion avec Azure IoT Hub](iot-hub-troubleshoot-connectivity.md).

## <a name="next-steps"></a>Étapes suivantes

- Pour plus d’informations sur les métriques, les journaux et d’autres valeurs importantes créées par [nom du service], consultez [Surveillance des informations de référence sur les données de surveillance d’Azure IoT Hub](monitor-iot-hub-reference.md) .

- Pour plus d’informations sur le monitoring des ressources Azure, voir [Monitoring des ressources Azure avec Azure Monitor](../azure-monitor/insights/monitor-azure-resource.md).