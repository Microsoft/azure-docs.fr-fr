---
title: Comportement d’exportation des métriques avec les valeurs NULL et zéro
description: Discussion relative aux valeurs NULL et zéro lors de l’exportation de métriques et pointeur vers la liste des métriques non exportables.
services: azure-monitor
ms.topic: reference
ms.date: 07/22/2020
ms.subservice: metrics
ms.openlocfilehash: ca6acb97e52123a0663d988b3f217d305bce2c4b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87131682"
---
# <a name="azure-monitor-platform-metrics-exportable-via-diagnostic-settings"></a>Métriques de plateforme Azure Monitor exportables par le biais des paramètres de diagnostic

Azure Monitor fournit des métriques de [plateforme](data-platform-metrics.md) par défaut sans configuration. Il offre plusieurs moyens d’interagir avec les métriques de plateforme, y compris en créant des graphiques dans le portail, en y accédant via l’API REST ou en envoyant des requêtes avec PowerShell ou l’interface CLI. Pour obtenir la liste complète des métriques de plateforme disponibles avec le pipeline de métriques consolidées d’Azure Monitor, consultez [Métriques prises en charge](metrics-supported.md). Pour rechercher ces métriques et y accéder, veuillez utiliser [2018-01-01 api-version](/rest/api/monitor/metricdefinitions). D’autres métriques peuvent être disponibles dans le portail ou via les API héritées.

## <a name="metrics-not-exportable-via-diagnostic-settings"></a>Métriques non exportables par le biais des paramètres de diagnostic

Le contenu qui se trouvait à cet emplacement a été déplacé dans [Liste des métriques Azure Monitor prises en charge](metrics-supported.md#exporting-platform-metrics-to-other-locations).

L’exportation de métriques par le biais des paramètres de diagnostic fait l’objet de limitations. Toutes les métriques sont exportables à l’aide de l’API REST. 

## <a name="exported-zero-vs-null-values"></a>Valeurs zéro et NULL exportées 

Les métriques ont un comportement différent pour la gestion des valeurs 0 et NULL.  Certaines métriques (par exemple, les métriques sur les échecs HTTP) indiquent 0 quand aucune donnée n’est récupérée. D’autres stockent des valeurs NULL quand elles n’obtiennent aucune donnée, car cela peut indiquer que la ressource est hors connexion. Vous pouvez voir la différence lorsque vous représentez ces métriques sous forme graphique, les valeurs NULL apparaissant sous forme de [lignes en pointillés](metrics-troubleshoot.md#chart-shows-dashed-line). 

Les métriques de plateforme exportables par le biais des paramètres de diagnostic présentent le même comportement que la métrique. Autrement dit, elles exportent des valeurs NULL lorsque la ressource n’envoie aucune donnée.  Elles n’exportent des « 0 » que s’ils ont réellement été émis par la ressource sous-jacente. 

Si vous supprimez un groupe de ressources ou une ressource spécifique, les données des métriques des ressources concernées ne sont plus envoyées aux destinations d’exportation des paramètres de diagnostic. Autrement dit, elles n’apparaissent plus dans Event Hubs, les comptes de stockage Azure ni les espaces de travail Log Analytics.

### <a name="metrics-that-used-to-export-zero-for-null"></a>Métriques qui exportaient zéro pour NULL

**Avant le 1er juin 2020**, les métriques ci-dessous émettaient des « 0 » lorsqu’il n’y avait pas de données. Ces zéros étaient ensuite exportables dans des systèmes en aval comme Log Analytics et le Stockage Azure.  Ce comportement entraînait une confusion entre les véritables « 0 » (émis par la ressource) et les « 0 » interprétés (valeurs NULL) ; le comportement a été modifié pour correspondre à celui de la métrique sous-jacente (cf. section précédente). 

La modification s’est produite dans tous les clouds publics et privés.

Le changement n’a pas eu d’impact sur le comportement des expériences suivantes : 
   - Journaux de ressources de plateforme exportés par le biais des paramètres de diagnostic
   - Graphiques de métriques dans Metrics Explorer
   - Alertes sur les métriques de plateforme
 
Voici la liste des métriques dont le comportement a changé. 

| ResourceType                    | Métrique               |  Nom d’affichage de la métrique  | 
|---------------------------------|----------------------|---------------------|
| Microsoft.ApiManagement/service | UnauthorizedRequests |  Demandes de la passerelle non autorisées (déprécié)  | 
| Microsoft.ApiManagement/service | TotalRequests |  Nombre total de demandes de la passerelle (déprécié)  | 
| Microsoft.ApiManagement/service | SuccessfulRequests |  Demandes de la passerelle ayant abouti (déprécié)  | 
| Microsoft.ApiManagement/service | Demandes |  Demandes  | 
| Microsoft.ApiManagement/service | OtherRequests |  Autres demandes de la passerelle (déprécié)  | 
| Microsoft.ApiManagement/service | FailedRequests |  Demandes de la passerelle ayant échoué (déprécié)  | 
| Microsoft.ApiManagement/service | EventHubTotalFailedEvents |  Événements EventHub non réussis  | 
| Microsoft.ApiManagement/service | EventHubTotalEvents |  Nombre total d'événements EventHub  | 
| Microsoft.ApiManagement/service | EventHubTotalBytesSent |  Taille des événements EventHub  | 
| Microsoft.ApiManagement/service | EventHubTimedoutEvents |  Événements EventHub expirés  | 
| Microsoft.ApiManagement/service | EventHubThrottledEvents |  Événements EventHub limités  | 
| Microsoft.ApiManagement/service | EventHubSuccessfulEvents |  Événements EventHub réussis  | 
| Microsoft.ApiManagement/service | EventHubRejectedEvents |  Événements EventHub rejetés  | 
| Microsoft.ApiManagement/service | EventHubDroppedEvents |  Événements EventHub supprimés  | 
| Microsoft.ApiManagement/service | Duration |  Durée globale des demandes de passerelle  | 
| Microsoft.ApiManagement/service | BackendDuration |  Durées des demandes back-end  | 
| Microsoft.DBforMariaDB/servers | storage_used |  Stockage utilisé  | 
| Microsoft.DBforMariaDB/servers | storage_percent |  Pourcentage de stockage  | 
| Microsoft.DBforMariaDB/servers | storage_limit |  Limite de stockage  | 
| Microsoft.DBforMariaDB/servers | serverlog_storage_usage |  Stockage du journal du serveur utilisé  | 
| Microsoft.DBforMariaDB/servers | serverlog_storage_percent |  Pourcentage de stockage du journal du serveur  | 
| Microsoft.DBforMariaDB/servers | serverlog_storage_limit |  Limite de stockage du journal du serveur  | 
| Microsoft.DBforMariaDB/servers | seconds_behind_master |  Décalage de la réplication en secondes  | 
| Microsoft.DBforMariaDB/servers | network_bytes_ingress |  Network In  | 
| Microsoft.DBforMariaDB/servers | network_bytes_egress |  Network Out  | 
| Microsoft.DBforMariaDB/servers | memory_percent |  Pourcentage de mémoire  | 
| Microsoft.DBforMariaDB/servers | io_consumption_percent |  Pourcentage d’E/S  | 
| Microsoft.DBforMariaDB/servers | cpu_percent |  Pourcentage d’UC  | 
| Microsoft.DBforMariaDB/servers | connections_failed |  Connexions ayant échoué  | 
| Microsoft.DBforMariaDB/servers | backup_storage_used |  Stockage de sauvegarde utilisé  | 
| Microsoft.DBforMariaDB/servers | active_connections |  Connexions actives  | 
| Microsoft.DBforMySQL/servers | storage_used |  Stockage utilisé  | 
| Microsoft.DBforMySQL/servers | storage_percent |  Pourcentage de stockage  | 
| Microsoft.DBforMySQL/servers | storage_limit |  Limite de stockage  | 
| Microsoft.DBforMySQL/servers | serverlog_storage_usage |  Stockage du journal du serveur utilisé  | 
| Microsoft.DBforMySQL/servers | serverlog_storage_percent |  Pourcentage de stockage du journal du serveur  | 
| Microsoft.DBforMySQL/servers | serverlog_storage_limit |  Limite de stockage du journal du serveur  | 
| Microsoft.DBforMySQL/servers | seconds_behind_master |  Décalage de la réplication en secondes  | 
| Microsoft.DBforMySQL/servers | network_bytes_ingress |  Network In  | 
| Microsoft.DBforMySQL/servers | network_bytes_egress |  Network Out  | 
| Microsoft.DBforMySQL/servers | memory_percent |  Pourcentage de mémoire  | 
| Microsoft.DBforMySQL/servers | io_consumption_percent |  Pourcentage d’E/S  | 
| Microsoft.DBforMySQL/servers | cpu_percent |  Pourcentage d’UC  | 
| Microsoft.DBforMySQL/servers | connections_failed |  Connexions ayant échoué  | 
| Microsoft.DBforMySQL/servers | backup_storage_used |  Stockage de sauvegarde utilisé  | 
| Microsoft.DBforMySQL/servers | active_connections |  Connexions actives  | 
| Microsoft.Devices/Account | digitaltwins.telemetry.nodes |  Espace réservé des données de télémétrie des nœuds Digital Twins  | 
| Microsoft.Devices/IotHubs | twinQueries.success |  Requêtes de représentations réussies  | 
| Microsoft.Devices/IotHubs | twinQueries.resultSize |  Taille du résultat des requêtes de représentations  | 
| Microsoft.Devices/IotHubs | twinQueries.failure |  Requêtes de représentations en échec  | 
| Microsoft.Devices/IotHubs | jobs.queryJobs.success |  Requêtes de travaux réussies  | 
| Microsoft.Devices/IotHubs | jobs.queryJobs.failure |  Requêtes de travaux en échec  | 
| Microsoft.Devices/IotHubs | jobs.listJobs.success |  Appels réussis pour répertorier les travaux  | 
| Microsoft.Devices/IotHubs | jobs.listJobs.failure |  Appels en échec pour répertorier les travaux  | 
| Microsoft.Devices/IotHubs | jobs.failed |  Travaux en échec  | 
| Microsoft.Devices/IotHubs | jobs.createTwinUpdateJob.success |  Créations réussies des travaux de mises à jour de représentations  | 
| Microsoft.Devices/IotHubs | jobs.createTwinUpdateJob.failure |  Créations des travaux de mises à jour de représentations en échec  | 
| Microsoft.Devices/IotHubs | jobs.createDirectMethodJob.success |  Créations réussies des travaux d’appel de méthode  | 
| Microsoft.Devices/IotHubs | jobs.createDirectMethodJob.failure |  Créations des travaux d’appel de méthode en échec  | 
| Microsoft.Devices/IotHubs | jobs.completed |  Travaux terminés  | 
| Microsoft.Devices/IotHubs | jobs.cancelJob.success |  Annulations de travaux réussies  | 
| Microsoft.Devices/IotHubs | jobs.cancelJob.failure |  Annulations de travaux en échec  | 
| Microsoft.Devices/IotHubs | EventGridLatency |  Latence d’Event Grid (préversion)  | 
| Microsoft.Devices/IotHubs | EventGridDeliveries |  Remises Event Grid (préversion)  | 
| Microsoft.Devices/IotHubs | devices.totalDevices |  Nombre total d’appareils (déconseillé)  | 
| Microsoft.Devices/IotHubs | devices.connectedDevices.allProtocol |  Appareils connectés (déconseillé)   | 
| Microsoft.Devices/IotHubs | deviceDataUsageV2 |  Utilisation totale des données d’appareil (préversion)  | 
| Microsoft.Devices/IotHubs | deviceDataUsage |  Utilisation totale des données d’appareil  | 
| Microsoft.Devices/IotHubs | dailyMessageQuotaUsed |  Nombre total de messages utilisés  | 
| Microsoft.Devices/IotHubs | d2c.twin.update.success |  Mises à jour de représentations réussies d’appareils  | 
| Microsoft.Devices/IotHubs | d2c.twin.update.size |  Taille des mises à jour de représentations d’appareils  | 
| Microsoft.Devices/IotHubs | d2c.twin.update.failure |  Mises à jour de représentations d’appareils en échec  | 
| Microsoft.Devices/IotHubs | d2c.twin.read.success |  Lectures de représentations réussies d’appareils  | 
| Microsoft.Devices/IotHubs | d2c.twin.read.size |  Taille de la réponse des lectures de représentations des appareils  | 
| Microsoft.Devices/IotHubs | d2c.twin.read.failure |  Lectures de représentations d’appareils en échec  | 
| Microsoft.Devices/IotHubs | d2c.telemetry.ingress.success |  Messages de télémétrie envoyés  | 
| Microsoft.Devices/IotHubs | d2c.telemetry.ingress.sendThrottle |  Nombre d’erreurs de limitation  | 
| Microsoft.Devices/IotHubs | d2c.telemetry.ingress.allProtocol |  Tentatives d’envoi de message de télémétrie  | 
| Microsoft.Devices/IotHubs | d2c.telemetry.egress.success |  Routage : messages de télémétrie remis  | 
| Microsoft.Devices/IotHubs | d2c.telemetry.egress.orphaned |  Routage : messages de télémétrie orphelins   | 
| Microsoft.Devices/IotHubs | d2c.telemetry.egress.invalid |  Routage : messages de télémétrie incompatibles  | 
| Microsoft.Devices/IotHubs | d2c.telemetry.egress.fallback |  Routage : messages remis à l’itinéraire de secours  | 
| Microsoft.Devices/IotHubs | d2c.telemetry.egress.dropped |  Routage : messages de télémétrie annulés   | 
| Microsoft.Devices/IotHubs | d2c.endpoints.latency.storage |  Routage : latence des messages du stockage  | 
| Microsoft.Devices/IotHubs | d2c.endpoints.latency.serviceBusTopics |  Routage : latence des messages de la rubrique Service Bus  | 
| Microsoft.Devices/IotHubs | d2c.endpoints.latency.serviceBusQueues |  Routage : latence des messages de la file d’attente Service Bus  | 
| Microsoft.Devices/IotHubs | d2c.endpoints.latency.eventHubs |  Routage : latence des messages d’Event Hub  | 
| Microsoft.Devices/IotHubs | d2c.endpoints.latency.builtIn.events |  Routage : latence des messages de messages/d’événements  | 
| Microsoft.Devices/IotHubs | d2c.endpoints.egress.storage.bytes |  Routage : données remises au stockage  | 
| Microsoft.Devices/IotHubs | d2c.endpoints.egress.storage.blobs |  Routage : objets blob remis au stockage  | 
| Microsoft.Devices/IotHubs | d2c.endpoints.egress.storage |  Routage : messages remis au stockage  | 
| Microsoft.Devices/IotHubs | d2c.endpoints.egress.serviceBusTopics |  Routage : messages remis à la rubrique Service Bus  | 
| Microsoft.Devices/IotHubs | d2c.endpoints.egress.serviceBusQueues |  Routage : messages remis à la file d’attente Service Bus  | 
| Microsoft.Devices/IotHubs | d2c.endpoints.egress.eventHubs |  Routage : messages remis à Event Hub  | 
| Microsoft.Devices/IotHubs | d2c.endpoints.egress.builtIn.events |  Routage : messages remis à des messages/événements  | 
| Microsoft.Devices/IotHubs | configurations |  Métriques de configuration  | 
| Microsoft.Devices/IotHubs | C2DMessagesExpired |  Messages cloud vers appareil ayant expiré (préversion)  | 
| Microsoft.Devices/IotHubs | c2d.twin.update.success |  Mises à jour de représentations réussies de serveur principal  | 
| Microsoft.Devices/IotHubs | c2d.twin.update.size |  Taille des mises à jour de représentations de serveur principal  | 
| Microsoft.Devices/IotHubs | c2d.twin.update.failure |  Mises à jour de représentations de serveur principal en échec  | 
| Microsoft.Devices/IotHubs | c2d.twin.read.success |  Lectures de représentations réussies de serveur principal  | 
| Microsoft.Devices/IotHubs | c2d.twin.read.size |  Taille de la réponse des lectures de représentations de serveur principal  | 
| Microsoft.Devices/IotHubs | c2d.twin.read.failure |  Lectures de représentations de serveur principal en échec  | 
| Microsoft.Devices/IotHubs | c2d.methods.success |  Appels de méthode directe réussis  | 
| Microsoft.Devices/IotHubs | c2d.methods.responseSize |  Taille de réponse des appels de méthode directe  | 
| Microsoft.Devices/IotHubs | c2d.methods.requestSize |  Taille de demande des appels de méthode directe  | 
| Microsoft.Devices/IotHubs | c2d.methods.failure |  Appels de méthode directe en échec  | 
| Microsoft.Devices/IotHubs | c2d.commands.egress.reject.success |  Messages cloud vers appareil rejetés  | 
| Microsoft.Devices/IotHubs | c2d.commands.egress.complete.success |  Remises de messages cloud vers appareil terminées  | 
| Microsoft.Devices/IotHubs | c2d.commands.egress.abandon.success |  Messages cloud vers appareil abandonnés  | 
| Microsoft.Devices/provisioningServices | RegistrationAttempts |  Tentatives d’enregistrement  | 
| Microsoft.Devices/provisioningServices | DeviceAssignments |  Appareils attribués  | 
| Microsoft.Devices/provisioningServices | AttestationAttempts |  Tentatives d’attestation  | 
| Microsoft.DocumentDB/databaseAccounts | TotalRequestUnits |  Unités de requête totales  | 
| Microsoft.DocumentDB/databaseAccounts | TotalRequests |  Total de requêtes  | 
| Microsoft.DocumentDB/databaseAccounts | MongoRequests |  Requêtes Mongo  | 
| Microsoft.DocumentDB/databaseAccounts | MongoRequestCharge |  Frais des requêtes Mongo  | 
| Microsoft.EventGrid/domains | PublishSuccessLatencyInMs |  Latence de réussite de la publication  | 
| Microsoft.EventGrid/domains | PublishSuccessCount |  Événements publiés  | 
| Microsoft.EventGrid/domains | PublishFailCount |  Événements d'échec de la publication  | 
| Microsoft.EventGrid/domains | MatchedEventCount |  Événements correspondants  | 
| Microsoft.EventGrid/domains | DroppedEventCount |  Événements annulés  | 
| Microsoft.EventGrid/domains | DeliverySuccessCount |  Événements remis  | 
| Microsoft.EventGrid/domains | DeadLetteredCount |  Événements de lettres mortes  | 
| Microsoft.EventGrid/eventSubscriptions | MatchedEventCount |  Événements correspondants  | 
| Microsoft.EventGrid/eventSubscriptions | DroppedEventCount |  Événements annulés  | 
| Microsoft.EventGrid/eventSubscriptions | DeliverySuccessCount |  Événements remis  | 
| Microsoft.EventGrid/eventSubscriptions | DeadLetteredCount |  Événements de lettres mortes  | 
| Microsoft.EventGrid/extensionTopics | UnmatchedEventCount |  Événements sans correspondance  | 
| Microsoft.EventGrid/extensionTopics | PublishSuccessLatencyInMs |  Latence de réussite de la publication  | 
| Microsoft.EventGrid/extensionTopics | PublishSuccessCount |  Événements publiés  | 
| Microsoft.EventGrid/extensionTopics | PublishFailCount |  Événements d'échec de la publication  | 
| Microsoft.EventGrid/topics | UnmatchedEventCount |  Événements sans correspondance  | 
| Microsoft.EventGrid/topics | PublishSuccessLatencyInMs |  Latence de réussite de la publication  | 
| Microsoft.EventGrid/topics | PublishSuccessCount |  Événements publiés  | 
| Microsoft.EventGrid/topics | PublishFailCount |  Événements d'échec de la publication  | 
| Microsoft.EventHub/clusters | OutgoingMessages |  Messages sortants  | 
| Microsoft.EventHub/clusters | OutgoingBytes |  Octets sortants.  | 
| Microsoft.EventHub/clusters | IncomingRequests |  Demandes entrantes  | 
| Microsoft.EventHub/clusters | IncomingMessages |  Messages entrants  | 
| Microsoft.EventHub/clusters | IncomingBytes |  Octets entrants.  | 
| Microsoft.EventHub/namespaces | SVRBSY |  Erreurs de serveur occupé (déconseillé)  | 
| Microsoft.EventHub/namespaces | SUCCREQ |  Requêtes réussies (déconseillé)  | 
| Microsoft.EventHub/namespaces | OUTMSGS |  Messages sortants (obsolète) (déprécié)  | 
| Microsoft.EventHub/namespaces | OutgoingMessages |  Messages sortants  | 
| Microsoft.EventHub/namespaces | OutgoingBytes |  Octets sortants.  | 
| Microsoft.EventHub/namespaces | MISCERR |  Autres erreurs (déconseillé)  | 
| Microsoft.EventHub/namespaces | INTERR |  Erreurs internes du serveur (déconseillé)  | 
| Microsoft.EventHub/namespaces | INREQS |  Requêtes entrantes (déconseillé)  | 
| Microsoft.EventHub/namespaces | INMSGS |  Messages entrants (obsolète) (déprécié)  | 
| Microsoft.EventHub/namespaces | IncomingRequests |  Demandes entrantes  | 
| Microsoft.EventHub/namespaces | IncomingMessages |  Messages entrants  | 
| Microsoft.EventHub/namespaces | IncomingBytes |  Octets entrants.  | 
| Microsoft.EventHub/namespaces | FAILREQ |  Requêtes non réussies (déconseillé)  | 
| Microsoft.EventHub/namespaces | EHOUTMSGS |  Messages sortants (déconseillé)  | 
| Microsoft.EventHub/namespaces | EHOUTMBS |  Octets sortants (obsolète) (déprécié)  | 
| Microsoft.EventHub/namespaces | EHOUTBYTES |  Octets sortants (déconseillé)  | 
| Microsoft.EventHub/namespaces | EHINMSGS |  Messages entrants (déconseillé)  | 
| Microsoft.EventHub/namespaces | EHINMBS |  Octets entrants (obsolète) (déprécié)  | 
| Microsoft.EventHub/namespaces | EHINBYTES |  Octets entrants (déconseillé)  | 
| Microsoft.EventHub/namespaces | EHAMSGS |  Messages archivés (déconseillé)  | 
| Microsoft.EventHub/namespaces | EHAMBS |  Archiver le débit des messages (déconseillé)  | 
| Microsoft.EventHub/namespaces | EHABL |  Archiver les messages de backlog (déconseillé)  | 
| Microsoft.HDInsight/clusters | NumActiveWorkers |  Nombre de collaborateurs actifs  | 
| Microsoft.HDInsight/clusters | GatewayRequests |  Demandes de la passerelle  | 
| Microsoft.HDInsight/clusters | CategorizedGatewayRequests |  Demandes de la passerelle par catégorie  | 
| Microsoft.Insights/AutoscaleSettings | ScaleActionsInitiated |  Actions de mise à l’échelle initiées  | 
| Microsoft.Insights/Components | traces/count |  Traces  | 
| Microsoft.Insights/Components | performanceCounters/requestsPerSecond |  Taux de requêtes HTTP  | 
| Microsoft.Insights/Components | performanceCounters/requestsInQueue |  Requêtes HTTP dans la file d’attente de l’application  | 
| Microsoft.Insights/Components | performanceCounters/exceptionsPerSecond |  Taux d’exceptions  | 
| Microsoft.Insights/Components | pageViews/count |  Affichages de page  | 
| Microsoft.Insights/Components | exceptions/count |  Exceptions  | 
| Microsoft.Kusto/Clusters | StreamingIngestResults |  Résultat de l’ingestion en streaming  | 
| Microsoft.Kusto/Clusters | StreamingIngestDuration |  Durée de l’ingestion en streaming  | 
| Microsoft.Kusto/Clusters | StreamingIngestDataRate |  Taux de données d’ingestion en streaming  | 
| Microsoft.Kusto/Clusters | SteamingIngestRequestRate |  Taux de demandes d’ingestion en streaming  | 
| Microsoft.Kusto/Clusters | QueryDuration |  Durée de la requête  | 
| Microsoft.Kusto/Clusters | KeepAlive |  Keep alive  | 
| Microsoft.Kusto/Clusters | IngestionVolumeInMB |  Volume d’ingestion (en Mo)  | 
| Microsoft.Kusto/Clusters | IngestionUtilization |  Utilisation de l’ingestion  | 
| Microsoft.Kusto/Clusters | IngestionResult |  Résultat de l’ingestion  | 
| Microsoft.Kusto/Clusters | IngestionLatencyInSeconds |  Latence d’ingestion (en secondes)  | 
| Microsoft.Kusto/Clusters | ExportUtilization |  Utilisation de l’exportation  | 
| Microsoft.Kusto/Clusters | EventsProcessedForEventHubs |  Événements traités (pour Event/IoT Hubs)  | 
| Microsoft.Kusto/Clusters | UC |  UC  | 
| Microsoft.Kusto/Clusters | ContinuousExportResult |  Résultat de l’exportation continue  | 
| Microsoft.Kusto/Clusters | ContinuousExportPendingCount |  Nombre en attente d’exportations continues  | 
| Microsoft.Kusto/Clusters | ContinuousExportNumOfRecordsExported |  Exportation continue : nombre d’enregistrements exportés  | 
| Microsoft.Kusto/Clusters | ContinuousExportMaxLatenessMinutes |  Minutes de retard max. pour l’exportation continue  | 
| Microsoft.Kusto/Clusters | CacheUtilization |  Utilisation du cache  | 
| Microsoft.Logic/integrationServiceEnvironments | TriggerThrottledEvents |  Événements limités par déclencheur  | 
| Microsoft.Logic/integrationServiceEnvironments | TriggersSucceeded |  Déclenchements ayant réussi   | 
| Microsoft.Logic/integrationServiceEnvironments | TriggersStarted |  Déclenchements lancés   | 
| Microsoft.Logic/integrationServiceEnvironments | TriggersSkipped |  Déclenchements ignorés  | 
| Microsoft.Logic/integrationServiceEnvironments | TriggersFired |  Déclenchements activés   | 
| Microsoft.Logic/integrationServiceEnvironments | TriggersFailed |  Déclenchements ayant échoué   | 
| Microsoft.Logic/integrationServiceEnvironments | TriggersCompleted |  Déclenchements terminés   | 
| Microsoft.Logic/integrationServiceEnvironments | RunThrottledEvents |  Exécuter événements limités  | 
| Microsoft.Logic/integrationServiceEnvironments | RunStartThrottledEvents |  Événements limités démarrés par l’exécution  | 
| Microsoft.Logic/integrationServiceEnvironments | RunsSucceeded |  Exécutions réussies  | 
| Microsoft.Logic/integrationServiceEnvironments | RunsStarted |  Exécutions démarrées  | 
| Microsoft.Logic/integrationServiceEnvironments | RunsFailed |  Exécutions ayant échoué  | 
| Microsoft.Logic/integrationServiceEnvironments | RunsCompleted |  Exécutions terminées  | 
| Microsoft.Logic/integrationServiceEnvironments | RunsCancelled |  Exécutions annulées  | 
| Microsoft.Logic/integrationServiceEnvironments | RunFailurePercentage |  Pourcentage d’échec des exécutions  | 
| Microsoft.Logic/integrationServiceEnvironments | ActionThrottledEvents |  Événements d’action limitée  | 
| Microsoft.Logic/integrationServiceEnvironments | ActionsSucceeded |  Actions ayant réussi   | 
| Microsoft.Logic/integrationServiceEnvironments | ActionsStarted |  Actions démarrées   | 
| Microsoft.Logic/integrationServiceEnvironments | ActionsSkipped |  Actions ignorées   | 
| Microsoft.Logic/integrationServiceEnvironments | ActionsFailed |  Actions ayant échoué   | 
| Microsoft.Logic/integrationServiceEnvironments | ActionsCompleted |  Actions terminées   | 
| Microsoft.Logic/workflows | TriggerThrottledEvents |  Événements limités par déclencheur  | 
| Microsoft.Logic/workflows | TriggersSucceeded |  Déclenchements ayant réussi   | 
| Microsoft.Logic/workflows | TriggersStarted |  Déclenchements lancés   | 
| Microsoft.Logic/workflows | TriggersSkipped |  Déclenchements ignorés  | 
| Microsoft.Logic/workflows | TriggersFired |  Déclenchements activés   | 
| Microsoft.Logic/workflows | TriggersFailed |  Déclenchements ayant échoué   | 
| Microsoft.Logic/workflows | TriggersCompleted |  Déclenchements terminés   | 
| Microsoft.Logic/workflows | TotalBillableExecutions |  Nombre total d’exécutions facturables  | 
| Microsoft.Logic/workflows | RunThrottledEvents |  Exécuter événements limités  | 
| Microsoft.Logic/workflows | RunStartThrottledEvents |  Événements limités démarrés par l’exécution  | 
| Microsoft.Logic/workflows | RunsSucceeded |  Exécutions réussies  | 
| Microsoft.Logic/workflows | RunsStarted |  Exécutions démarrées  | 
| Microsoft.Logic/workflows | RunsFailed |  Exécutions ayant échoué  | 
| Microsoft.Logic/workflows | RunsCompleted |  Exécutions terminées  | 
| Microsoft.Logic/workflows | RunsCancelled |  Exécutions annulées  | 
| Microsoft.Logic/workflows | RunFailurePercentage |  Pourcentage d’échec des exécutions  | 
| Microsoft.Logic/workflows | BillingUsageStorageConsumption |  Utilisation de la facturation pour les exécutions de consommation du stockage  | 
| Microsoft.Logic/workflows | BillingUsageStorageConsumption |  Utilisation de la facturation pour les exécutions de consommation du stockage  | 
| Microsoft.Logic/workflows | BillingUsageStandardConnector |  Utilisation de la facturation pour les exécutions de connecteurs Standard  | 
| Microsoft.Logic/workflows | BillingUsageStandardConnector |  Utilisation de la facturation pour les exécutions de connecteurs Standard  | 
| Microsoft.Logic/workflows | BillingUsageNativeOperation |  Utilisation de la facturation pour les exécutions d’opérations natives  | 
| Microsoft.Logic/workflows | BillingUsageNativeOperation |  Utilisation de la facturation pour les exécutions d’opérations natives  | 
| Microsoft.Logic/workflows | BillableTriggerExecutions |  Exécutions de déclencheurs facturables  | 
| Microsoft.Logic/workflows | BillableActionExecutions |  Exécutions d’actions facturables  | 
| Microsoft.Logic/workflows | ActionThrottledEvents |  Événements d’action limitée  | 
| Microsoft.Logic/workflows | ActionsSucceeded |  Actions ayant réussi   | 
| Microsoft.Logic/workflows | ActionsStarted |  Actions démarrées   | 
| Microsoft.Logic/workflows | ActionsSkipped |  Actions ignorées   | 
| Microsoft.Logic/workflows | ActionsFailed |  Actions ayant échoué   | 
| Microsoft.Logic/workflows | ActionsCompleted |  Actions terminées   | 
| Microsoft.Network/frontdoors | WebApplicationFirewallRequestCount |  Nombre de requêtes du pare-feu d’applications web  | 
| Microsoft.Network/frontdoors | TotalLatency |  Latence totale  | 
| Microsoft.Network/frontdoors | ResponseSize |  Taille de la réponse  | 
| Microsoft.Network/frontdoors | RequestSize |  Taille de la requête  | 
| Microsoft.Network/frontdoors | RequestCount |  Nombre de requêtes  | 
| Microsoft.Network/frontdoors | BillableResponseSize |  Taille de la réponse facturable  | 
| Microsoft.Network/frontdoors | BackendRequestLatency |  Latence de requête du backend  | 
| Microsoft.Network/frontdoors | BackendRequestCount |  Nombre de requêtes de backend  | 
| Microsoft.Network/frontdoors | BackendHealthPercentage |  Pourcentage d’intégrité du backend  | 
| Microsoft.Network/trafficManagerProfiles | QpsByEndpoint |  Requêtes par point de terminaison renvoyé  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | scheduled.pending |  Notifications planifiées en attente  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | registration.update |  Opérations de mise à jour d’inscription  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | registration.get |  Opérations de lecture d’inscription  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | registration.delete |  Opérations de suppression d’inscription  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | registration.create |  Opérations de création d’inscription  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | registration.all |  Opérations d’inscription  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.wrongtoken |  Erreurs d’autorisation WNS (jeton incorrect)  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.tokenproviderunreachable |  Erreurs d’autorisation WNS (inaccessible)  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.throttled |  Notifications WNS limitées  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.success |  Notifications WNS réussies  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.pnserror |  Erreurs WNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.invalidtoken |  Erreurs d’autorisation WNS (jeton non valide)  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.invalidnotificationsize |  Erreur de taille de notification WNS non valide  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.invalidnotificationformat |  Format de notification WNS non valide  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.invalidcredentials |  Erreurs d’autorisation WNS (informations d’identification non valides)  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.expiredchannel |  Erreur de canal WNS arrivé à expiration  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.dropped |  Notifications WNS supprimées  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.channelthrottled |  Canal WNS limité  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.channeldisconnected |  Canal WNS déconnecté  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.badchannel |  Erreur de canal WNS incorrect  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.wns.authenticationerror |  Erreurs d’authentification WNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.mpns.throttled |  Notifications MPNS limitées  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.mpns.success |  Notifications MPNS réussies  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.mpns.pnserror |  Erreurs MPNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.mpns.invalidnotificationformat |  Format de notification MPNS non valide  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.mpns.invalidcredentials |  Informations d’identification MPNS non valides  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.mpns.dropped |  Notifications MPNS supprimées  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.mpns.channeldisconnected |  Canal MPNS déconnecté  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.mpns.badchannel |  Erreur de canal incorrect MPNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.mpns.authenticationerror |  Erreurs d’authentification MPNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.gcm.wrongchannel |  Erreur de canal GCM incorrect  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.gcm.throttled |  Notifications GCM limitées  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.gcm.success |  Notifications GCM réussies  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.gcm.pnserror |  Erreurs GCM  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.gcm.invalidnotificationsize |  Erreur de taille de notification GCM non valide  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.gcm.invalidnotificationformat |  Format de notification GCM non valide  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.gcm.invalidcredentials |  Erreurs d’autorisation GCM (informations d’identification non valides)  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.gcm.expiredchannel |  Erreur de canal GCM arrivé à expiration  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.gcm.badchannel |  Erreur de canal GCM incorrect  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.gcm.authenticationerror |  Erreurs d’authentification GCM  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.apns.success |  Notifications APNS réussies  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.apns.pnserror |  Erreurs APNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.apns.invalidnotificationsize |  Erreur de taille de notification APNS non valide  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.apns.invalidcredentials |  Erreurs d’autorisation APNS  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.apns.expiredchannel |  Erreur de canal APNS arrivé à expiration  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.apns.badchannel |  Erreur de canal APNS incorrect  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.allpns.success |  Notifications réussies  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.allpns.pnserror |  Erreurs système de notification externe  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.allpns.invalidpayload |  Erreurs de charge utile  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | outgoing.allpns.channelerror |  Erreurs de canal  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | notificationhub.pushes |  Toutes les notifications sortantes  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | installation.upsert |  Opérations de création ou de mise à jour d’installation  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | installation.patch |  Opérations d’installation de correctif  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | installation.get |  Opérations d’obtention d’installation  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | installation.delete |  Opérations de suppression d’installation  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | installation.all |  Opérations de gestion de l’installation  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | incoming.scheduled.cancel |  Notifications Push planifiées annulées  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | incoming.scheduled |  Notifications Push planifiées envoyées  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | incoming.all.requests |  Toutes les demandes entrantes  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | incoming.all.failedrequests |  Toutes les requêtes entrantes ayant échoué  | 
| Microsoft.NotificationHubs/Namespaces/NotificationHubs | incoming |  Messages entrants  | 
| Microsoft.OperationalInsights/workspaces | Heartbeat |  Heartbeat  | 
| Microsoft.Relay/namespaces | BytesTransferred |  BytesTransferred  | 
| Microsoft.ServiceBus/namespaces | OutgoingMessages |  Messages sortants  | 
| Microsoft.ServiceBus/namespaces | IncomingRequests |  Demandes entrantes  | 
| Microsoft.ServiceBus/namespaces | IncomingMessages |  Messages entrants  | 
| Microsoft.SignalRService/SignalR | UserErrors |  Erreurs d’utilisateur  | 
| Microsoft.SignalRService/SignalR | SystemErrors |  Erreurs système  | 
| Microsoft.SignalRService/SignalR | OutboundTraffic |  Trafic sortant  | 
| Microsoft.SignalRService/SignalR | MessageCount |  Nombre de messages  | 
| Microsoft.SignalRService/SignalR | InboundTraffic |  Trafic entrant  | 
| Microsoft.SignalRService/SignalR | ConnectionCount |  Nombre de connexions  | 
| Microsoft.Sql/managedInstances | avg_cpu_percent |  Pourcentage d’UC moyenne  | 
| Microsoft.Sql/servers | dtu_used |  DTU utilisé  | 
| Microsoft.Sql/servers | dtu_consumption_percent |  Pourcentage DTU  | 
| Microsoft.Sql/servers/databases | xtp_storage_percent |  Pourcentage de stockage OLTP en mémoire  | 
| Microsoft.Sql/servers/databases | workers_percent |  Pourcentage de travaux  | 
| Microsoft.Sql/servers/databases | sessions_percent |  Pourcentage de sessions  | 
| Microsoft.Sql/servers/databases | physical_data_read_percent |  Pourcentage E/S des données  | 
| Microsoft.Sql/servers/databases | log_write_percent |  Pourcentage E/S du journal  | 
| Microsoft.Sql/servers/databases | dwu_used |  DWU utilisé  | 
| Microsoft.Sql/servers/databases | dwu_consumption_percent |  Pourcentage DWU  | 
| Microsoft.Sql/servers/databases | dtu_used |  DTU utilisé  | 
| Microsoft.Sql/servers/databases | dtu_consumption_percent |  Pourcentage DTU  | 
| Microsoft.Sql/servers/databases | deadlock |  Blocages  | 
| Microsoft.Sql/servers/databases | cpu_used |  UC utilisée  | 
| Microsoft.Sql/servers/databases | cpu_percent |  Pourcentage UC  | 
| Microsoft.Sql/servers/databases | connection_successful |  Connexions réussies  | 
| Microsoft.Sql/servers/databases | connection_failed |  Connexions ayant échoué  | 
| Microsoft.Sql/servers/databases | cache_hit_percent |  Pourcentage d’accès au cache  | 
| Microsoft.Sql/servers/databases | blocked_by_firewall |  Bloqué par le pare-feu  | 
| Microsoft.Sql/servers/elasticPools | xtp_storage_percent |  Pourcentage de stockage OLTP en mémoire  | 
| Microsoft.Sql/servers/elasticPools | workers_percent |  Pourcentage de travaux  | 
| Microsoft.Sql/servers/elasticPools | sessions_percent |  Pourcentage de sessions  | 
| Microsoft.Sql/servers/elasticPools | physical_data_read_percent |  Pourcentage E/S des données  | 
| Microsoft.Sql/servers/elasticPools | log_write_percent |  Pourcentage E/S du journal  | 
| Microsoft.Sql/servers/elasticPools | eDTU_used |  eDTU utilisé  | 
| Microsoft.Sql/servers/elasticPools | dtu_consumption_percent |  Pourcentage DTU  | 
| Microsoft.Sql/servers/elasticPools | cpu_percent |  Pourcentage UC  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | TotalFrontEnds |  Nombre total de serveurs frontaux  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | SmallAppServicePlanInstances |  Workers de plan App Service de petite taille  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | Demandes |  Demandes  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | MemoryPercentage |  Pourcentage de mémoire  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | MediumAppServicePlanInstances |  Workers de plan App Service de taille moyenne  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | LargeAppServicePlanInstances |  Workers de plan App Service de grande taille  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | HttpQueueLength |  Longueur de la file d’attente HTTP  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | Http5xx |  Erreurs de serveur http  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | Http4xx |  Http 4xx  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | Http406 |  Http 406  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | Http404 |  Http 404  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | Http403 |  Http 403  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | Http401 |  Http 401  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | Http3xx |  Http 3xx  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | Http2xx |  Http 2xx  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | Http101 |  HTTP 101  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | DiskQueueLength |  Longueur de file d'attente de disque  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | CpuPercentage |  Pourcentage UC  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | BytesSent |  Données sortantes  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | BytesReceived |  Données entrantes  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | AverageResponseTime |  Temps de réponse moyen  | 
| Microsoft.Web/hostingEnvironments/multiRolePools | ActiveRequests |  Requêtes actives  | 
| Microsoft.Web/hostingEnvironments/workerPools | WorkersUsed |  Workers utilisés  | 
| Microsoft.Web/hostingEnvironments/workerPools | WorkersTotal |  Nombre total de workers  | 
| Microsoft.Web/hostingEnvironments/workerPools | WorkersAvailable |  Workers disponibles  | 
| Microsoft.Web/hostingEnvironments/workerPools | MemoryPercentage |  Pourcentage de mémoire  | 
| Microsoft.Web/hostingEnvironments/workerPools | CpuPercentage |  Pourcentage UC  | 
| Microsoft.Web/serverfarms | TcpTimeWait |  Time Wait - TCP  | 
| Microsoft.Web/serverfarms | TcpSynSent |  Syn Sent - TCP  | 
| Microsoft.Web/serverfarms | TcpSynReceived |  Syn Received - TCP  | 
| Microsoft.Web/serverfarms | TcpLastAck |  Last Ack - TCP  | 
| Microsoft.Web/serverfarms | TcpFinWait2 |  Fin Wait 2 - TCP  | 
| Microsoft.Web/serverfarms | TcpFinWait1 |  Fin Wait 1 - TCP  | 
| Microsoft.Web/serverfarms | TcpEstablished |  Established - TCP  | 
| Microsoft.Web/serverfarms | TcpClosing |  Closing - TCP  | 
| Microsoft.Web/serverfarms | TcpCloseWait |  Close Wait - TCP  | 
| Microsoft.Web/serverfarms | MemoryPercentage |  Pourcentage de mémoire  | 
| Microsoft.Web/serverfarms | HttpQueueLength |  Longueur de la file d’attente HTTP  | 
| Microsoft.Web/serverfarms | DiskQueueLength |  Longueur de file d'attente de disque  | 
| Microsoft.Web/serverfarms | CpuPercentage |  Pourcentage UC  | 
| Microsoft.Web/serverfarms | BytesSent |  Données sortantes  | 
| Microsoft.Web/serverfarms | BytesReceived |  Données entrantes  | 
| Microsoft.Web/sites | TotalAppDomainsUnloaded |  Total des domaines d’application déchargés  | 
| Microsoft.Web/sites | TotalAppDomains |  Total des domaines d’application  | 
| Microsoft.Web/sites | Threads |  Nombre de threads  | 
| Microsoft.Web/sites | RequestsInApplicationQueue |  Demandes dans la file d’attente d’application  | 
| Microsoft.Web/sites | Demandes |  Demandes  | 
| Microsoft.Web/sites | PrivateBytes |  Octets privés  | 
| Microsoft.Web/sites | MemoryWorkingSet |  Plage de travail de la mémoire  | 
| Microsoft.Web/sites | IoWriteOperationsPerSecond |  Opérations d’écriture par seconde (E/S)  | 
| Microsoft.Web/sites | IoWriteBytesPerSecond |  Octets écrits par seconde (E/S)  | 
| Microsoft.Web/sites | IoReadOperationsPerSecond |  Opérations de lecture par seconde (E/S)  | 
| Microsoft.Web/sites | IoReadBytesPerSecond |  Octets lus par seconde (E/S)  | 
| Microsoft.Web/sites | IoOtherOperationsPerSecond |  Autres opérations par seconde (E/S)  | 
| Microsoft.Web/sites | IoOtherBytesPerSecond |  Autres octets par seconde (E/S)  | 
| Microsoft.Web/sites | HttpResponseTime |  Temps de réponse  | 
| Microsoft.Web/sites | Http5xx |  Erreurs de serveur http  | 
| Microsoft.Web/sites | Http4xx |  Http 4xx  | 
| Microsoft.Web/sites | Http406 |  Http 406  | 
| Microsoft.Web/sites | Http404 |  Http 404  | 
| Microsoft.Web/sites | Http403 |  Http 403  | 
| Microsoft.Web/sites | Http401 |  Http 401  | 
| Microsoft.Web/sites | Http3xx |  Http 3xx  | 
| Microsoft.Web/sites | Http2xx |  Http 2xx  | 
| Microsoft.Web/sites | Http101 |  HTTP 101  | 
| Microsoft.Web/sites | HealthCheckStatus |  État de contrôle d’intégrité  | 
| Microsoft.Web/sites | Poignées |  Nombre de descripteurs  | 
| Microsoft.Web/sites | Gen2Collections |  Garbage collections de génération 2  | 
| Microsoft.Web/sites | Gen1Collections |  Garbage collections de génération 1  | 
| Microsoft.Web/sites | Gen0Collections |  Garbage collections de génération 0  | 
| Microsoft.Web/sites | FunctionExecutionUnits |  Unités d’exécution de fonctions  | 
| Microsoft.Web/sites | FunctionExecutionCount |  Nombre d’exécutions de fonctions  | 
| Microsoft.Web/sites | CurrentAssemblies |  Assemblys actuels  | 
| Microsoft.Web/sites | CpuTime |  Temps processeur  | 
| Microsoft.Web/sites | BytesSent |  Données sortantes  | 
| Microsoft.Web/sites | BytesReceived |  Données entrantes  | 
| Microsoft.Web/sites | AverageResponseTime |  Temps de réponse moyen  | 
| Microsoft.Web/sites | AverageMemoryWorkingSet |  Plage de travail moyenne de la mémoire  | 
| Microsoft.Web/sites | AppConnections |  Connexions  | 
| Microsoft.Web/sites/slots | TotalAppDomainsUnloaded |  Total des domaines d’application déchargés  | 
| Microsoft.Web/sites/slots | TotalAppDomains |  Total des domaines d’application  | 
| Microsoft.Web/sites/slots | Threads |  Nombre de threads  | 
| Microsoft.Web/sites/slots | RequestsInApplicationQueue |  Demandes dans la file d’attente d’application  | 
| Microsoft.Web/sites/slots | Demandes |  Demandes  | 
| Microsoft.Web/sites/slots | PrivateBytes |  Octets privés  | 
| Microsoft.Web/sites/slots | MemoryWorkingSet |  Plage de travail de la mémoire  | 
| Microsoft.Web/sites/slots | IoWriteOperationsPerSecond |  Opérations d’écriture par seconde (E/S)  | 
| Microsoft.Web/sites/slots | IoWriteBytesPerSecond |  Octets écrits par seconde (E/S)  | 
| Microsoft.Web/sites/slots | IoReadOperationsPerSecond |  Opérations de lecture par seconde (E/S)  | 
| Microsoft.Web/sites/slots | IoReadBytesPerSecond |  Octets lus par seconde (E/S)  | 
| Microsoft.Web/sites/slots | IoOtherOperationsPerSecond |  Autres opérations par seconde (E/S)  | 
| Microsoft.Web/sites/slots | IoOtherBytesPerSecond |  Autres octets par seconde (E/S)  | 
| Microsoft.Web/sites/slots | HttpResponseTime |  Temps de réponse  | 
| Microsoft.Web/sites/slots | Http5xx |  Erreurs de serveur http  | 
| Microsoft.Web/sites/slots | Http4xx |  Http 4xx  | 
| Microsoft.Web/sites/slots | Http406 |  Http 406  | 
| Microsoft.Web/sites/slots | Http404 |  Http 404  | 
| Microsoft.Web/sites/slots | Http403 |  Http 403  | 
| Microsoft.Web/sites/slots | Http401 |  Http 401  | 
| Microsoft.Web/sites/slots | Http3xx |  Http 3xx  | 
| Microsoft.Web/sites/slots | Http2xx |  Http 2xx  | 
| Microsoft.Web/sites/slots | Http101 |  HTTP 101  | 
| Microsoft.Web/sites/slots | HealthCheckStatus |  État de contrôle d’intégrité  | 
| Microsoft.Web/sites/slots | Poignées |  Nombre de descripteurs  | 
| Microsoft.Web/sites/slots | Gen2Collections |  Garbage collections de génération 2  | 
| Microsoft.Web/sites/slots | Gen1Collections |  Garbage collections de génération 1  | 
| Microsoft.Web/sites/slots | Gen0Collections |  Garbage collections de génération 0  | 
| Microsoft.Web/sites/slots | FunctionExecutionUnits |  Unités d’exécution de fonctions  | 
| Microsoft.Web/sites/slots | FunctionExecutionCount |  Nombre d’exécutions de fonctions  | 
| Microsoft.Web/sites/slots | CurrentAssemblies |  Assemblys actuels  | 
| Microsoft.Web/sites/slots | CpuTime |  Temps processeur  | 
| Microsoft.Web/sites/slots | BytesSent |  Données sortantes  | 
| Microsoft.Web/sites/slots | BytesReceived |  Données entrantes  | 
| Microsoft.Web/sites/slots | AverageResponseTime |  Temps de réponse moyen  | 
| Microsoft.Web/sites/slots | AverageMemoryWorkingSet |  Plage de travail moyenne de la mémoire  | 
| Microsoft.Web/sites/slots | AppConnections |  Connexions  | 
| Microsoft.Sql/servers/databases | cpu_percent | Pourcentage UC | 
| Microsoft.Sql/servers/databases | physical_data_read_percent | Pourcentage E/S des données | 
| Microsoft.Sql/servers/databases | log_write_percent | Pourcentage E/S du journal | 
| Microsoft.Sql/servers/databases | dtu_consumption_percent | Pourcentage DTU | 
| Microsoft.Sql/servers/databases | connection_successful | Connexions réussies | 
| Microsoft.Sql/servers/databases | connection_failed | Connexions ayant échoué | 
| Microsoft.Sql/servers/databases | blocked_by_firewall | Bloqué par le pare-feu | 
| Microsoft.Sql/servers/databases | deadlock | Blocages | 
| Microsoft.Sql/servers/databases | xtp_storage_percent | Pourcentage de stockage OLTP en mémoire | 
| Microsoft.Sql/servers/databases | workers_percent | Pourcentage de travaux | 
| Microsoft.Sql/servers/databases | sessions_percent | Pourcentage de sessions | 
| Microsoft.Sql/servers/databases | dtu_used | DTU utilisé | 
| Microsoft.Sql/servers/databases | cpu_used | UC utilisée | 
| Microsoft.Sql/servers/databases | dwu_consumption_percent | Pourcentage DWU | 
| Microsoft.Sql/servers/databases | dwu_used | DWU utilisé | 
| Microsoft.Sql/servers/databases | cache_hit_percent | Pourcentage d’accès au cache | 
| Microsoft.Sql/servers/databases | wlg_allocation_relative_to_system_percent | Allocation du groupe de charge de travail par pourcentage système | 
| Microsoft.Sql/servers/databases | wlg_allocation_relative_to_wlg_effective_cap_percent | Allocation du groupe de charge de travail par pourcentage maximal de ressources | 
| Microsoft.Sql/servers/databases | wlg_active_queries | Requêtes actives du groupe de charge de travail | 
| Microsoft.Sql/servers/databases | wlg_queued_queries | Requêtes en file d’attente du groupe de charge de travail | 
| Microsoft.Sql/servers/databases | active_queries | Requêtes actives | 
| Microsoft.Sql/servers/databases | queued_queries | Requêtes mises en file d’attente | 
| Microsoft.Sql/servers/databases | wlg_active_queries_timeouts | Délais d’expiration des requêtes du groupe de charge de travail | 
| Microsoft.Sql/servers/databases | wlg_queued_queries_timeouts | Délais d’expiration des requêtes du groupe de charge de travail mis en file d’attente | 
| Microsoft.Sql/servers/databases | wlg_effective_min_resource_percent | Pourcentage minimal de ressources réelles | 
| Microsoft.Sql/servers/databases | wlg_effective_cap_resource_percent | Pourcentage de ressources limitées réelles | 
| Microsoft.Sql/servers/elasticPools | cpu_percent | Pourcentage UC | 
| Microsoft.Sql/servers/elasticPools | physical_data_read_percent | Pourcentage E/S des données | 
| Microsoft.Sql/servers/elasticPools | log_write_percent | Pourcentage E/S du journal | 
| Microsoft.Sql/servers/elasticPools | dtu_consumption_percent | Pourcentage DTU | 
| Microsoft.Sql/servers/elasticPools | workers_percent | Pourcentage de travaux | 
| Microsoft.Sql/servers/elasticPools | sessions_percent | Pourcentage de sessions | 
| Microsoft.Sql/servers/elasticPools | eDTU_used | eDTU utilisé | 
| Microsoft.Sql/servers/elasticPools | xtp_storage_percent | Pourcentage de stockage OLTP en mémoire | 
| Microsoft.Sql/servers | dtu_consumption_percent | Pourcentage DTU | 
| Microsoft.Sql/servers | dtu_used | DTU utilisé | 
| Microsoft.Sql/managedInstances | avg_cpu_percent | Pourcentage d’UC moyenne | 

