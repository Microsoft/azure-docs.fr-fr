---
title: Synchronisation de modèle entre Azure Digital Twins et Time Series Insights | Microsoft Docs
description: Meilleures pratiques et outils utilisés pour traduire un modèle de ressource dans Azure Digital Twins (ADT) en modèle de ressource dans Azure Time Series Insights (STI)
ms.service: time-series-insights
services: time-series-insights
author: msrsrinivasan
ms.author: radhsrin
manager: diviso
ms.workload: big-data
ms.topic: conceptual
ms.date: 09/22/2020
ms.custom: dpalled
ms.openlocfilehash: 0d02a6e3eb2aef4a02c90360b2016e64af579081
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95014728"
---
# <a name="model-synchronization-between-azure-digital-twins-and-time-series-insights-gen2"></a>Synchronisation des modèles entre Azure Digital Twins et Time Series Insights Gen2

Cet article décrit les meilleures pratiques et les outils utilisés pour traduire un modèle de ressource dans Azure Digital Twins (ADT) en modèle de ressource dans Azure Time Series Insights (TSI).  Il constitue la deuxième partie d’une série de tutoriels qui en comprend deux, décrivant l’intégration d’Azure Digital Twins avec Azure Time Series Insights. L’intégration d’Azure Digital Twins avec Time Series Insights permet l’archivage et le suivi de l’historique des télémétries et des propriétés calculées de Digital Twins. Cette série de tutoriels est destinée aux développeurs qui travaillent l’intégration de Time Series Insights avec Azure Digital Twins. La première partie décrit la [mise en place d’un pipeline de données qui intègre les données de série chronologique d’Azure Digital Twins à Azure Time Series Insights](../digital-twins/how-to-integrate-time-series-insights.md), tandis que cette deuxième partie explique la synchronisation de modèles de ressource entre Azure Digital Twins et Azure Time Series Insights. Ce tutoriel décrit les meilleures pratiques en matière de choix et d’établissement d’une convention d’affectation de noms pour l’ID de série chronologique (ID TS) et l’établissement manuel de hiérarchies dans le modèle de série chronologique (TSM).

## <a name="choosing-a-time-series-id"></a>Choix d’un ID de série chronologique

Un ID de série chronologique est un identificateur unique utilisé pour identifier des ressources dans Time Series Insights. Les données de série chronologique (télémétries qui sont des paires heure-valeur) sont représentées à l’aide de variables répertoriées sous ID TS. Dans Azure Digital Twins, des propriétés de jumeau et des télémétries sont utilisées pour représenter respectivement l’état d’un jumeau et les mesures produites par le jumeau. À partir de la conception actuelle de TSM, les ID TS doivent être uniques. L’utilisation d’ID de jumeaux dans Azure Digital Twins ou d’une combinaison de ceux-ci avec un nom de propriété ou de télémétrie produira toujours un ID TS unique dans TSM. Dans un cas typique, le **_`<Twin ID>`_** sera l’ID TS, et les noms de propriété et de télémétrie seront les variables dans TSM. Toutefois, il existe des cas d’utilisation où il est préférable que les hiérarchies de ressources dans Time Series Insights soient aplaties à l’aide d’un format de clés composites, tel que **_`<Twin ID>+ <Delimiter of Choice> + <Name of the Property or Telemetry>`_**. Prenons un exemple pour expliquer ce dernier cas. Imaginez une salle dans un immeuble modélisée en tant que jumeau, dont l’ID de jumeau est Room22. La propriété de son paramètre de température doit être traduite en **_TSID Room22_TempSetting_**, et la mesure de la température doit être traduite en **_Room22_TempMea_** dans TSM.

[![Choix d’un ID de série chronologique](media/tutorials-model-sync/choosing-tsid.png)](media/tutorials-model-sync/choosing-tsid.png#lightbox)

## <a name="contextualizing-time-series"></a>Contextualisation de séries chronologiques

La Contextualization des données (essentiellement spatiales par nature) dans Time Series Insights est obtenue via des hiérarchies de ressources, et utilisée pour faciliter la navigation dans les données via une arborescence dans l’Explorateur Time Series Insights. Les types de séries chronologiques et les hiérarchies sont définis à l’aide d’un modèle de série chronologique (TSM) dans Time Series Insights. Des types dans TSM permettent de définir des variables, tandis que des niveaux de hiérarchie et des valeurs de champ d’instance sont utilisés pour construire l’arborescence dans l’Explorateur Time Series Insights. Pour plus d’informations sur TSM, consultez la [documentation en ligne sur Time Series Insights](./concepts-model-overview.md).

Dans Azure Digital Twins, la connexion entre des ressources est exprimée à l’aide de relations entre jumeaux. Les relations entre jumeaux sont simplement un graphique de ressources connectées. Toutefois, dans Time Series Insight, les relations entre ressources sont hiérarchiques par nature. Autrement dit, les ressources partagent une relation de type parent-enfant et sont représentées à l’aide d’une structure arborescente. Pour traduire les informations de relation d’Azure Digital Twins en hiérarchies Time Series Insights, nous devons choisir des relations hiérarchiques pertinentes d’Azure Digital Twins. Azure Digital Twins utilise un langage de modélisation standard ouvert, appelé DTDL (Digital Twin Definition Language). Les modèles DTDL sont décrits à l’aide d’une variante de JSON appelée JSON-LD. Pour plus d’informations sur la spécification, consultez la [documentation sur le langage DTDL](https://github.com/Azure/opendigitaltwins-dtdl/blob/master/DTDL/v2/dtdlv2.md).

[![Connexion entre ressources](media/tutorials-model-sync/asset-connection.png)](media/tutorials-model-sync/asset-connection.png#lightbox)

## <a name="translating-graph-representation-in-azure-digital-twins-to-tree-structure-in-time-series-insights"></a>Traduction de représentation graphique dans Azure Digital Twins en structure arborescente dans Time Series Insights

Les sections suivantes du tutoriel illustrent quelques scénarios de base de traduction manuelle de la structure graphique dans Azure Digital Twins en structure arborescente dans Time Series Insights.

Exemple de système : Ce tutoriel utilise l’exemple suivant pour expliquer les concepts abordés ci-dessous. Il s’agit d’un système de gestion de bâtiment fictif simple, pour un étage comportant deux pièces. Il compte trois thermostats, un dans chacune des pièces, et un autre commun à l’étage. Il dispose également d’un débitmètre mesurant le flux d’eau s’écoulant de la pièce Room21 à la pièce Room22 via une canalisation. En examinant la relation spatiale entre jumeaux, on observe deux types de relations.

1. Relation hiérarchique, la plus courante. Par exemple, Building40-> Floor01-> FloorTS*-> Temperature
1. Relation circulaire, peu courante. Par exemple, à partir de Building40, le débitmètre FlowMtr peut être suivi via deux chemins différents.

   1. Building40-> Floor01-> Room21-> FlowMtr*-> Flow
   1. Building40-> Floor01-> Room22-> FlowMtr*-> Flow  
      Où « Flow » est la télémétrie réelle mesurant le flux d’eau circulant entre les pièces Room21 et Room22

> [!Note]
>
> `*` FloorTS est le thermostat de l’étage, et FlowMtr le débitmètre généralement choisi comme ID TS. Temperature et Flow sont des télémétries brutes de température et de flux, appelées variables dans Time Series Insights.

Compte tenu de la limitation actuelle dans Time Series Insights en vertu de laquelle une ressource ne peut pas être représentée dans plusieurs branches, les sections suivantes expliquent la modélisation des relations hiérarchiques et circulaires dans Time Series Insights.

## <a name="case-1-hierarchical-parent-child-relationship"></a>Cas n° 1 : Relation hiérarchique (parent-enfant)

Il s’agit du type de relation le plus courant entre jumeaux. La modélisation de la relation parent-enfant pure est expliquée dans l’illustration suivante. Les champs d’instance et l’ID TS sont dérivés des ID de jumeaux, comme illustré ci-dessous. Bien qu’il soit possible de mettre à jour les champs d’instance manuellement à l’aide de l’Explorateur Time Series Insights, la section ci-dessous, intitulée « Mise à jour des champs d’instance à l’aide d’API » explique comment écouter les changements de modèle dans Azure Digital Twins et mettre à jour les champs d’instance dans Time Series Insights en utilisant des fonctions Azure.

[![Mappage d’ID de jumeaux](media/tutorials-model-sync/mapping-twin-ids.png)](media/tutorials-model-sync/mapping-twin-ids.png#lightbox)

[![Mappage d’ID de jumeaux 2](media/tutorials-model-sync/mapping-twin-ids2.png)](media/tutorials-model-sync/mapping-twin-ids2.png#lightbox)

## <a name="case-2-circular-relationship"></a>Cas n° 2 : Relation circulaire

### <a name="circular-relationship-in-azure-digital-twins-to-single-hierarchy-relationship-in-time-series-insights"></a>Relation circulaire dans Azure Digital Twins vers relation hiérarchique dans Time Series Insights

Étant donné que l’ID TS doit être unique et ne peut être représenté que dans une seule hiérarchie, ce cas représente le _« FlowMtr »_ avec une télémétrie nommée _« Flow »_ juste sous le jumeau _« Room21 »_ . À l’avenir, lorsque Time Series Insights pourra prendre en charge plusieurs représentations de séries chronologiques dans TSM, la télémétrie _« Flow »_ sera représentée sous _« Room 21 »_ et _« Room 22 »_ .

La capture d’écran suivante montre le mappage manuel d’ID de jumeaux dans Azure Digital Twins vers un champ Instance dans TSM, et la hiérarchie qui en résulte dans Time Series Insights.

[![Mappage d’ID de jumeaux dans Azure Digital Twins](media/tutorials-model-sync/mapping-twin-ids-adt.png)](media/tutorials-model-sync/mapping-twin-ids-adt.png#lightbox)

### <a name="circular-relationship-in-azure-digital-twins-to-multiple-hierarchies-in-time-series-insights-using-duplicates"></a>Relation circulaire dans Azure Digital Twins vers hiérarchies multiples dans Time Series Insights, utilisant des doublons

La première partie du tutoriel explique comment un programme client (une fonction Azure) permet de transférer des données de télémétrie à partir de sources d’événements IoT Hub ou autres vers Azure Digital Twins. Cette approche suggère d’utiliser le même programme client pour mettre à jour les propriétés pertinentes des jumeaux parents. Dans l’exemple donné, lors de la lecture de la télémétrie du débitmètre FlowMtr à partir d’IoT Hub et de la mise à jour de la propriété « Flow » dans le débitmètre FlowMtr jumeau, le programme peut également mettre à jour les propriétés correspondantes dans tous les jumeaux parents possibles de la source. Dans notre exemple, il s’agit de la propriété « outflowmea » (identifiée à l’aide de la relation « outflow ») de la pièce Room21, et de la propriété « inflowmea » de la pièce Room22.  La capture d’écran ci-dessous présente l’expérience utilisateur finale dans l’Explorateur Time Series Insights. Il convient de noter qu’en adoptant cette approche, nous avons des doublons de données.

[![Explorateur Time Series Insights](media/tutorials-model-sync/tsi-explorer.png)](media/tutorials-model-sync/tsi-explorer.png#lightbox)

L’extrait de code ci-dessous montre comment l’application cliente a pu naviguer dans la relation entre jumeaux à l’aide d’API Azure Digital Twins.

> [!Note]
>
> Cet exemple d’extrait de code suppose que les lecteurs ont pris connaissance de la [première partie](../digital-twins/tutorial-end-to-end.md#set-up-the-sample-function-app) du tutoriel, et que ce changement de code a été effectué à l’intérieur de la fonction « ProcessHubToDTEvents ».

```csharp
if (propertyPath.Equals("/Flow"))
{
//Update the flow value property of the flow meter
await AdtUtilities.UpdateTwinProperty(client, twinId, "replace",
propertyPath, "double", propertyValue, log);

//also update the sending end flow
string parentIdOutflow = await AdtUtilities.FindParent(client, twinId,
"outflow", log);
if (parentIdOutflow != null)
await AdtUtilities.UpdateTwinProperty(client, parentIdOutflow, "replace", "outflow", "double", propertyValue, log);
else
log.LogInformation("Unable to find Parent with outflow
relationship for " + twinId );
//and receiving end flow value
string parentIdinflow = await AdtUtilities.FindParent(client, twinId,
"inflow", log);
if (parentIdinflow != null)

await AdtUtilities.UpdateTwinProperty(client, parentIdinflow,
"replace", "inflow", "double", propertyValue, log);
else
log.LogInformation("Unable to find Parent with inflow
relationship for " + twinId);
}
```

## <a name="updating-instance-fields-using-apis"></a>Mise à jour des champs d’instance à l’aide d’API

Cette section du tutoriel explique comment écouter des changements de modèle dans Azure Digital Twins, tels que la création et la suppression de jumeaux, ou la modification des relations entre jumeaux, ainsi que mettre à jour des champs d’instance et hiérarchies par programmation à l’aide d’API de modèle Time Series Insights. Cette méthode de mise à jour de modèle Time Series Insights est généralement obtenue via des fonctions Azure. Dans Azure Digital Twins, des notifications d’événements, telles que des ajouts ou suppressions de jumeaux, peuvent être des services routés en aval tels que des Event Hubs qui peuvent à leur tour alimenter des fonctions Azure. Vous trouverez des informations supplémentaires sur le routage et le filtrage d’événements [ici](../digital-twins/how-to-manage-routes-portal.md).  Le reste de cette section explique comment utiliser des API de modèle Time Series Insights dans des fonctions Azure pour mettre à jour un modèle Time Series Insights en réponse à l’ajout de jumeau (un type de modification de modèle) dans Azure Digital Twins.

### <a name="receiving-and-identifying-twin-addition-event-notification"></a>Réception et identification de notification d’événement d’ajout de jumeau

```csharp
[FunctionName("RouteEventsToTsi")]
public async Task Run([EventGridTrigger]EventGridEvent eventGridEvent)
{
    try
    {
        if (eventGridEvent != null && eventGridEvent.Data != null)
        {
            logger.LogInformation($"EventType: {eventGridEvent.EventType}");
            logger.LogInformation($"EventGridEvent: {JsonConvert.SerializeObject(eventGridEvent)}");

            //Shape event and Send event data to event hub and tsi
            await SendEventToEventHubAsync(eventGridEvent).ConfigureAwait(false);

            //If a new twin was created, update the newly created instance in TSI with info retrieved from ADT
            if (eventGridEvent.EventType == Constants.TwinCreateEventType)
            {
                //retrieve building, floor and room of value twin
                var twinInfo = await RetrieveTwinInfoAsync(eventGridEvent).ConfigureAwait(false);
                //Update Tsi instance with type(sensor type), hierarchy(space hierarchy) and instance fields(twin info retrieved above)
                var instance = await CreateInstanceToSendAsync(twinInfo).ConfigureAwait(false);
                var instanceToUpdate = new List<TimeSeriesInstance>() { instance };
                var response = await tsiClient.TimeSeriesInstances.ExecuteBatchAsync(new InstancesBatchRequest(update: instanceToUpdate)).ConfigureAwait(false);
            }
        }
    }
    catch (Exception ex)
    {
        logger.LogError($"Exception: {ex.Message}");
    }
}
```

### <a name="creating-time-series-insights-client-and-adding-instance-details"></a>Création de client Time Series Insights et ajout de détails d’instance

```csharp
private async Task<TimeSeriesInstance> CreateInstanceToSendAsync(Dictionary<string, string> twinInfo)
{
    try
    {
        tsiClient = await GetTSIClientAsync().ConfigureAwait(false);

        var timeSeriesId = new object[] { twinInfo[Constants.DtId] };
        var instances = await tsiClient.TimeSeriesInstances.ExecuteBatchAsync(
            new InstancesBatchRequest(
                get: new InstancesRequestBatchGetOrDelete(
                    new IList<object>[] { timeSeriesId }))).ConfigureAwait(false);
        var instance = instances.Get.First().Instance;

        if (instance != null)
        {
            instance = await AddHierarchyToInstanceAsync(instance).ConfigureAwait(false);
            instance = await AddTypeToInstanceAsync(instance, twinInfo[Constants.TwinType]).ConfigureAwait(false);

            instance.InstanceFields = new Dictionary<string, object>
                            {
                                { "Building", twinInfo[Constants.BuildingName] },
                                { "Floor", twinInfo[Constants.FloorName] },
                                { "Room", twinInfo[Constants.ParentName] }
                            };

            //If value twin is a sensor value twin, add sensor type instance field to diff from spatial value twin
            if (twinInfo[Constants.ParentType] == Constants.Sensor)
            {
                instance.InstanceFields.Add(Constants.SensorType, twinInfo[Constants.TwinType]);
            }
            return instance;
        }
        else
        {
            logger.LogError($"instance with id {twinInfo[Constants.DtId]} not found");
            return new TimeSeriesInstance();
        }
    }
    catch (Exception e)
    {
        logger.LogError(e.Message);
        throw;
    }
}
```

### <a name="applying-hierarchy-info-to-instance"></a>Application d’informations de hiérarchie à une instance

```csharp
private async Task<TimeSeriesInstance> AddHierarchyToInstanceAsync(TimeSeriesInstance instance)
{
    if (instance.HierarchyIds == null)
    {
        TimeSeriesHierarchy spacesHierarchy = null;
        try
        {
            var hierarchy = await RunGetHierarchiesAsync(Constants.SpaceHierarchy).ConfigureAwait(false);
            spacesHierarchy = hierarchy.First(h => h.Name.Equals(Constants.SpaceHierarchy));
            instance.HierarchyIds = new List<Guid?>();
            instance.HierarchyIds.Add(spacesHierarchy.Id);
        }
        catch (Exception ex)
        {
            logger.LogWarning($"Hierarchy 'space hierarchy' not found, {ex}");
            throw;
        }
    }
    return instance;
}
```

## <a name="next-steps"></a>Étapes suivantes

Le troisième tutoriel de la série montre comment interroger des données historiques d’Azure Digital Twins à l’aide d’API Time Series Insights. Il s’agit d’un travail en cours et la section sera mise à jour quand elle sera prête. En attendant, les lecteurs sont encouragés à consulter la [documentation sur l’API de requête de données Time Series Insights](./concepts-query-overview.md).