---
title: Services et schémas pris en charge pour les journaux de ressource Azure
description: Découvrez les services pris en charge et le schéma d’événement pour les journaux de ressource Azure.
ms.subservice: logs
ms.topic: reference
ms.date: 09/01/2020
ms.openlocfilehash: f5ea6d3f28fe85cf1453f3cf2b9eb0132bda0013
ms.sourcegitcommit: 02ed9acd4390b86c8432cad29075e2204f6b1bc3
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/29/2020
ms.locfileid: "97808298"
---
# <a name="common-and-service-specific-schema-for-azure-resource-logs"></a>Schémas communs et spécifiques de services pour les journaux de ressources Azure

> [!NOTE]
> Les journaux de ressource étaient auparavant appelés journaux de diagnostic. Le nom a été modifié en octobre 2019 parce que les types de journaux collectés par Azure Monitor ont évolué pour inclure plus que la seule ressource Azure. En outre, la liste des catégories de journaux de ressources que vous pouviez était répertoriée dans cet article. Elles ont été déplacées vers les [catégories de journaux de ressources](resource-logs-categories.md). 

Les [journaux de ressource Azure Monitor](./platform-logs-overview.md) sont des journaux d’activité générés par les services Azure, qui décrivent le fonctionnement de ces services ou ressources. Tous les journaux de ressource disponibles via Azure Monitor partagent un schéma commun de niveau supérieur, avec la flexibilité pour chaque service d’émettre des propriétés uniques pour ses propres événements.

Une combinaison du type de ressource (disponible dans la propriété `resourceId`) et du `category` permet d’identifier de manière unique un schéma. Cet article décrit le schéma de niveau supérieur pour les journaux de ressource et les liens vers les schémas pour chaque service.


## <a name="top-level-common-schema"></a>Schéma commun de niveau supérieur

| Nom | Obligatoire ou facultatif | Description |
|---|---|---|
| time | Obligatoire | Horodatage (heure UTC) de l’événement. |
| resourceId | Obligatoire | ID de la ressource qui a émis l’événement. Pour les services abonnés, cet ID prend la forme /tenants/tenant-id/providers/provider-name. |
| tenantId | Obligatoire pour les journaux d’activité de l’abonné | ID d’abonné de l’abonné Active Directory auquel cet événement est lié. Cette propriété est utilisée uniquement pour les journaux d’activité de niveau abonné ; elle n’apparaît pas dans les journaux d’activité de niveau ressource. |
| operationName | Obligatoire | Nom de l’opération représentée par cet événement. Si l’événement représente une opération Azure RBAC, il s’agit du nom de l’opération Azure RBAC (par exemple, Microsoft.Storage/storageAccounts/blobServices/blobs/Read). Généralement modélisée sous la forme d’une opération du Gestionnaire de ressources, même s’il ne s’agit pas d’opérations réelles documentées du Gestionnaire des ressources (`Microsoft.<providerName>/<resourceType>/<subtype>/<Write/Read/Delete/Action>`) |
| operationVersion | Facultatif | Version d’api associée à l’opération, si operationName a été effectuée à l’aide d’une API (par exemple `http://myservice.windowsazure.net/object?api-version=2016-06-01`). S’il n’existe aucune API qui corresponde à cette opération, la version représente la version de cette opération si les propriétés associées à l’opération viennent à changer. |
| catégorie | Obligatoire | Catégorie de journal de l’événement. La catégorie est la granularité selon laquelle vous pouvez activer ou désactiver des journaux d’activité sur une ressource particulière. Les propriétés qui apparaissent dans l’objet blob de propriétés d’un événement sont les mêmes au sein d’un type de ressource et d’une catégorie de journal spécifique. Ls catégories de journaux classiques sont « Audit », « Opérationnel », « Exécution » et « Demande ». |
| resultType | Facultatif | État de l’événement. Les valeurs courantes sont : Started, In Progress, Succeeded, Failed, Active et Resolved. |
| resultSignature | Facultatif | Sous-état de l’événement. Si cette opération correspond à un appel d’API REST, ce champ est le code d’état HTTP de l’appel REST correspondant. |
| resultDescription | Facultatif | Description textuelle statique de cette opération, par exemple « Obtenir le fichier de stockage ». |
| durationMS | Facultatif | Durée de l’opération en millisecondes. |
| callerIpAddress | Facultatif | Adresse IP de l’appelant, si l’opération correspond à un appel d’API qui provient d’une entité avec une adresse IP disponible publiquement. |
| correlationId | Facultatif | GUID utilisé pour regrouper un ensemble d’événements associés. En règle générale, si deux événements ont le même operationName, mais deux états différents (par exemple « Started » et « Succeeded »), ils partagent le même ID de corrélation. Cela peut également représenter d’autres relations entre les événements. |
| identité | Facultatif | Objet blob JSON qui décrit l’identité de l’utilisateur ou d’une application qui a effectué l’opération. En général, ce champ inclut l’autorisation et les revendications/Jeton JWT issus d’active directory. |
| Level | Facultatif | Niveau de gravité de l’événement. Doit être Information, Avertissement, Erreur, ou Critique. |
| location | Facultatif | Région de la ressource émettant l’événement, par exemple « USA Est » ou « France Sud » |
| properties | Facultatif | Toutes les propriétés étendues associées à cette catégorie particulière d’événements spécifiques. Toutes les propriétés personnalisées/uniques doivent être placées à l’intérieur de cette « Partie B » du schéma. |

## <a name="service-specific-schemas"></a>Schémas par service

Le schéma des journaux des ressources varie en fonction de la ressource et de la catégorie de journal d’activité. Cette liste répertorie les services qui mettent des journaux de ressource et des liens à disposition du service et un schéma spécifique de la catégorie lorsqu’il est disponible. Cette liste change en permanence à mesure que de nouveaux services sont ajoutés. Par conséquent, si vous ne voyez pas ce dont vous avez besoin ci-dessous, utilisez un moteur de recherche pour découvrir plus de documentation. N’hésitez pas à ouvrir un problème GitHub à propos de cet article afin que nous le mettions à jour.

| Service | Schéma et documentation |
| --- | --- |
| Azure Active Directory | [Vue d’ensemble](../../active-directory/reports-monitoring/concept-activity-logs-azure-monitor.md), [Schéma des journaux d’audit](../../active-directory/reports-monitoring/reference-azure-monitor-audit-log-schema.md) et [Schéma des connexions](../../active-directory/reports-monitoring/reference-azure-monitor-sign-ins-log-schema.md) |
| Analysis Services | [Azure Analysis Services - Configurer la journalisation des diagnostics](../../analysis-services/analysis-services-logging.md) |
| Gestion des API | [Journaux de ressource de Gestion des API](../../api-management/api-management-howto-use-azure-monitor.md#resource-logs) |
| App Service | [Journaux App Service](../../app-service/troubleshoot-diagnostic-logs.md)
| Passerelles d’application |[Journalisation pour Application Gateway](../../application-gateway/application-gateway-diagnostics.md) |
| Azure Automation |[Log Analytics pour Azure Automation](../../automation/automation-manage-send-joblogs-log-analytics.md) |
| Azure Batch |[Journalisation d’Azure Batch](../../batch/batch-diagnostics.md) |
| Cognitive Services | [Journalisation pour Azure Cognitive Services](../../cognitive-services/diagnostic-logging.md) |
| Container Registry | [Journalisation pour Azure Container Registry](../../container-registry/container-registry-diagnostics-audit-logs.md) |
| Réseau de distribution de contenu | [Journaux Azure pour CDN](../../cdn/cdn-azure-diagnostic-logs.md) |
| CosmosDB | [Journalisation Azure Cosmos DB](../../cosmos-db/monitor-cosmos-db.md) |
| Data Factory | [Surveiller les fabriques de données à l’aide d’Azure Monitor](../../data-factory/monitor-using-azure-monitor.md) |
| Data Lake Analytics |[Accès aux journaux d’Azure Data Lake Analytics](../../data-lake-analytics/data-lake-analytics-diagnostic-logs.md) |
| Data Lake Store |[Accès aux journaux d’Azure Data Lake Storage](../../data-lake-store/data-lake-store-diagnostic-logs.md) |
| Explorateur de données Azure | [Journaux d’Azure Data Explorer](/azure/data-explorer/using-diagnostic-logs) |
| Azure Database pour MySQL | [Journaux de diagnostic Azure Database pour MySQL](../../mysql/concepts-server-logs.md#diagnostic-logs) |
| Azure Database pour PostgreSQL | [Journaux d’Azure Database pour PostgreSQL](../../postgresql/concepts-server-logs.md#resource-logs) |
| Azure Databricks | [Journalisation des diagnostics dans Azure Databricks](/azure/databricks/administration-guide/account-settings/azure-diagnostic-logs) |
| Azure Digital Twins | [Configurer les diagnostics Azure Digital Twins](../../digital-twins/troubleshoot-diagnostics.md#log-schemas)
| Event Hubs |[Journaux d’Azure Event Hubs](../../event-hubs/event-hubs-diagnostic-logs.md) |
| ExpressRoute | Schéma non disponible. |
| Pare-feu Azure | Schéma non disponible. |
| Front Door | [Journalisation pour Front Door](../../frontdoor/front-door-diagnostics.md) |
| IoT Hub | [Opérations IoT Hub](../../iot-hub/monitor-iot-hub-reference.md#resource-logs) |
| Key Vault |[Journalisation d’Azure Key Vault](../../key-vault/general/logging.md) |
| Kubernetes Service |[Journalisation d’Azure Kubernetes](../../aks/view-master-logs.md#log-event-schema) |
| Load Balancer |[Analyse des journaux de l'équilibreur de charge Azure](../../load-balancer/load-balancer-monitor-log.md) |
| Logic Apps |[Schéma de suivi personnalisé Logic Apps B2B](../../logic-apps/logic-apps-track-integration-account-custom-tracking-schema.md) |
| Network Security Group |[Analytique des journaux pour les groupes de sécurité réseau (NSG)](../../virtual-network/virtual-network-nsg-manage-log.md) |
| Protection DDOS | [Gérer le service Azure DDoS Protection standard](../../ddos-protection/diagnostic-logging.md#log-schemas) |
| Power BI dédiées | [Journalisation pour Power BI Embedded dans Azure](/power-bi/developer/azure-pbie-diag-logs) |
| Recovery Services | [Modèle de données pour la sauvegarde Azure](../../backup/backup-azure-reports-data-model.md)|
| Recherche |[Activation et utilisation de la fonctionnalité Rechercher l’analyse du trafic](../../search/search-traffic-analytics.md) |
| Service Bus |[Journaux d’Azure Service Bus](../../service-bus-messaging/service-bus-diagnostic-logs.md) |
| SQL Database | [Journalisation d’Azure SQL Database](../../azure-sql/database/metrics-diagnostic-telemetry-logging-streaming-export-configure.md) |
| Stream Analytics |[Journaux de tâches](../../stream-analytics/stream-analytics-job-diagnostic-logs.md) |
| Traffic Manager | [Schéma des journaux de Traffic Manager](../../traffic-manager/traffic-manager-diagnostic-logs.md) |
| Virtual Network | Schéma non disponible. |
| Passerelles de réseau virtuel | Schéma non disponible. |



## <a name="next-steps"></a>Étapes suivantes

* [Voir les catégories de journaux de ressources que vous pouvez collecter](resource-logs-categories.md)
* [En savoir plus sur les journaux de ressources](./platform-logs-overview.md)
* [Diffuser en continu les journaux de ressource vers **Event Hubs**](./resource-logs.md#send-to-azure-event-hubs)
* [Modifier les paramètres de diagnostic de journal de ressource à l’aide de l’API REST Azure Monitor](/rest/api/monitor/diagnosticsettings)
* [Analyser les journaux d’activité du stockage Azure avec Log Analytics](./resource-logs.md#send-to-log-analytics-workspace)
