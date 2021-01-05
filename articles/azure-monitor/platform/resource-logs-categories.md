---
title: Services et catégories pris en charge des journaux de ressources Azure Monitor
description: Référence d’Azure Monitor Comprendre les services et le schéma d’événement pris en charge pour les journaux de ressource Azure.
ms.subservice: logs
ms.topic: reference
ms.date: 12/09/2020
ms.openlocfilehash: c7b2d48b40843930bba78f54d2294769d952daf6
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96931226"
---
# <a name="supported-categories-for-azure-resource-logs"></a>Catégories prises en charge pour les journaux de ressources Azure

> [!NOTE]
> Les journaux de ressources étaient auparavant appelés journaux de diagnostic. Le nom a été modifié en octobre 2019 parce que les types de journaux collectés par Azure Monitor ont évolué pour inclure plus que la seule ressource Azure.

Les [journaux de ressource Azure Monitor](./platform-logs-overview.md) sont des journaux d’activité générés par les services Azure, qui décrivent le fonctionnement de ces services ou ressources. Tous les journaux de ressource disponibles via Azure Monitor partagent un schéma commun de niveau supérieur, avec la flexibilité pour chaque service d’émettre des propriétés uniques pour ses propres événements.

Une combinaison du type de ressource (disponible dans la propriété `resourceId`) et du `category` permet d’identifier de manière unique un schéma. Il existe un schéma commun pour tous les journaux de ressources avec des champs spécifiques de services ajoutés pour différentes catégories de journaux. Pour plus d’informations, consultez l’article sur les [schémas commun et spécifiques de services pour les journaux de ressources Azure]().


## <a name="costs"></a>Coûts

Des coûts sont associés à l’envoi et au stockage de données dans Log Analytics, le Stockage Azure et/ou un hub d’événements. Vous pouvez être amené à supporter le coût lié à l’envoi des données vers ces emplacements et à leur conservation.  Les journaux de ressources sont un type de données que vous pouvez envoyer vers ces emplacements. Un [coût supplémentaire pour exporter certaines catégories de journaux de ressources](https://azure.microsoft.com/pricing/details/monitor/) vers ces emplacements s’applique, tandis que d’autres sont exemptes de coûts d’exportation. Les détails des coûts d’exportation sont listés dans le tableau ci-dessous.

## <a name="supported-log-categories-per-resource-type"></a>Catégories de journaux prises en charge par type de ressource

Voici la liste des types de journaux disponibles pour chaque type de ressource. 

Il est possible que certaines catégories soient prises en charge uniquement pour des types de ressources spécifiques. Si vous avez le sentiment qu’il vous manque une ressource, consultez la documentation spécifique de celle-ci. Par exemple, les catégories Microsoft.Sql/servers/databases ne sont pas disponibles pour tous les types de bases de données. Pour plus d’informations, consultez les [informations sur la journalisation des diagnostics SQL Database](../../azure-sql/database/metrics-diagnostic-telemetry-logging-streaming-export-configure.md). 

S’il vous manque toujours quelque chose, vous pouvez ouvrir un commentaire GitHub au bas de cet article.
## <a name="microsoftanalysisservicesservers"></a>Microsoft.AnalysisServices/servers

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|Moteur|Moteur|
|Service|Service|


## <a name="microsoftapimanagementservice"></a>Microsoft.ApiManagement/service

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|GatewayLogs|Journaux d’activité relatifs à la passerelle ApiManagement|


## <a name="microsoftappplatformspring"></a>Microsoft.AppPlatform/Spring

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|ApplicationConsole|Application console|
|SystemLogs|Journaux système|


## <a name="microsoftautomationautomationaccounts"></a>Microsoft.Automation/automationAccounts

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|DscNodeStatus|État du nœud DSC|
|JobLogs|Journaux d’activité de travail|
|JobStreams|Flux de travail|


## <a name="microsoftbatchbatchaccounts"></a>Microsoft.Batch/batchAccounts

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|ServiceLog|Journaux d’activité de service|


## <a name="microsoftbatchaiworkspaces"></a>Microsoft.BatchAI/workspaces

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|BaiClusterEvent|BaiClusterEvent|
|BaiClusterNodeEvent|BaiClusterNodeEvent|
|BaiJobEvent|BaiJobEvent|


## <a name="microsoftblockchainblockchainmembers"></a>Microsoft.Blockchain/blockchainMembers

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|BlockchainApplication|Application blockchain|
|FabricOrderer|Ordonnanceur de structure|
|FabricPeer|Pair de structure|
|Proxy|Proxy|


## <a name="microsoftblockchaincordamembers"></a>Microsoft.Blockchain/cordaMembers

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|BlockchainApplication|Application blockchain|


## <a name="microsoftcdncdnwebapplicationfirewallpolicies"></a>Microsoft.Cdn/cdnwebapplicationfirewallpolicies

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|WebApplicationFirewallLogs|Journaux du pare-feu d’applications web|


## <a name="microsoftcdnprofiles"></a>Microsoft.Cdn/profiles

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|AzureCdnAccessLog|Journal d’accès à Azure CDN|


## <a name="microsoftcdnprofilesendpoints"></a>Microsoft.Cdn/profiles/endpoints

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|CoreAnalytics|Obtient les métriques du point de terminaison, par exemple, la bande passante, la sortie, etc.|


## <a name="microsoftclassicnetworknetworksecuritygroups"></a>Microsoft.ClassicNetwork/networksecuritygroups

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|Événement de flux de règle de groupe de sécurité réseau|Événement de flux de règle de groupe de sécurité réseau|


## <a name="microsoftcognitiveservicesaccounts"></a>Microsoft.CognitiveServices/accounts

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|Audit|Journaux d’audit|
|RequestResponse|Journaux d’activité des requêtes et des réponses|
|Trace|Journaux de suivi|


## <a name="microsoftcontainerregistryregistries"></a>Microsoft.ContainerRegistry/registries

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|ContainerRegistryLoginEvents|Événements de connexion|
|ContainerRegistryRepositoryEvents|Journaux RepositoryEvent|


## <a name="microsoftcontainerservicemanagedclusters"></a>Microsoft.ContainerService/managedClusters

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|cluster-autoscaler|Autoscaler de cluster Kubernetes|
|kube-apiserver|Serveur d’API Kubernetes|
|kube-audit|Audit Kubernetes|
|kube-controller-manager|Gestionnaire de contrôleur Kubernetes|
|kube-scheduler|Planificateur Kubernetes|


## <a name="microsoftcustomprovidersresourceproviders"></a>Microsoft.CustomProviders/resourceproviders

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|AuditLogs|Journaux d’audit pour les appels MiniRP|


## <a name="microsoftdatabricksworkspaces"></a>Microsoft.Databricks/workspaces

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|accounts|Comptes Databricks|
|clusters|Clusters Databricks|
|dbfs|Système de fichiers Databricks|
|instancePools|Pools d’instances|
|jobs|Travaux Databricks|
|notebook|Databricks Notebook|
|secrets|Secrets Databricks|
|sqlPermissions|SQLPermissions Databricks|
|ssh|SSH Databricks|
|espace de travail|Espace de travail Databricks|


## <a name="microsoftdatafactoryfactories"></a>Microsoft.DataFactory/factories

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|ActivityRuns|Journal d’exécutions d’activités de pipeline|
|PipelineRuns|Journal d’exécutions de pipeline|
|TriggerRuns|Journal d’exécutions de déclencheur|


## <a name="microsoftdatalakestoreaccounts"></a>Microsoft.DataLakeStore/accounts

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|Audit|Journaux d’audit|
|Demandes|Journaux d’activité de requête|


## <a name="microsoftdatashareaccounts"></a>Microsoft.DataShare/accounts

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|ReceivedShareSnapshots|Instantanés de partage reçus|
|SentShareSnapshots|Instantanés de partage envoyés|
|Partages|Partages|
|ShareSubscriptions|Abonnements de partage|


## <a name="microsoftdbformariadbservers"></a>Microsoft.DBforMariaDB/servers

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|MySqlAuditLogs|Journaux d’audit MariaDB|
|MySqlSlowLogs|Journaux du serveur MariaDB|


## <a name="microsoftdbformysqlflexibleservers"></a>Microsoft.DBforMySQL/flexibleServers

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|MySqlAuditLogs|Journaux d’audit MySQL|
|MySqlSlowLogs|Journaux des requêtes lentes MySQL|


## <a name="microsoftdbformysqlservers"></a>Microsoft.DBforMySQL/servers

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|MySqlAuditLogs|Journaux d’audit MySQL|
|MySqlSlowLogs|Journaux d’activité MySQL Server|


## <a name="microsoftdbforpostgresqlflexibleservers"></a>Microsoft.DBforPostgreSQL/flexibleServers

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|PostgreSQLLogs|Journaux d’activité de serveur PostgreSQL|


## <a name="microsoftdbforpostgresqlservers"></a>Microsoft.DBforPostgreSQL/servers

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|PostgreSQLLogs|Journaux d’activité de serveur PostgreSQL|
|QueryStoreRuntimeStatistics|Statistiques de runtime du magasin des requêtes PostgreSQL|
|QueryStoreWaitStatistics|Statistiques d’attente du magasin des requêtes PostgreSQL|


## <a name="microsoftdbforpostgresqlserversv2"></a>Microsoft.DBforPostgreSQL/serversv2

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|PostgreSQLLogs|Journaux d’activité de serveur PostgreSQL|


## <a name="microsoftdesktopvirtualizationapplicationgroups"></a>Microsoft.DesktopVirtualization/applicationgroups

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|Point de contrôle|Point de contrôle|
|Error|Error|
|Gestion|Gestion|


## <a name="microsoftdesktopvirtualizationhostpools"></a>Microsoft.DesktopVirtualization/hostpools

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|Point de contrôle|Point de contrôle|
|Connexion|Connexion|
|Error|Error|
|HostRegistration|HostRegistration|
|Gestion|Gestion|


## <a name="microsoftdesktopvirtualizationworkspaces"></a>Microsoft.DesktopVirtualization/workspaces

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|Point de contrôle|Point de contrôle|
|Error|Error|
|Flux|Flux|
|Gestion|Gestion|


## <a name="microsoftdevicesiothubs"></a>Microsoft.Devices/IotHubs

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|C2DCommands|Commandes C2D|
|C2DTwinOperations|Opérations de jumeaux C2D|
|Configurations|Configurations|
|Connexions|Connexions|
|Opérations de jumeaux D2C|Opérations de jumeaux D2C|
|DeviceIdentityOperations|Opérations d’identité des appareils|
|DeviceStreams|Flux d’appareils (préversion)|
|DeviceTelemetry|Télémétrie d’appareil|
|DirectMethods|Méthodes directes|
|DistributedTracing|Traçage distribué (préversion)|
|FileUploadOperations|Opérations de chargement de fichier|
|JobsOperations|Opérations de travaux|
|Itinéraires|Itinéraires|
|TwinQueries|Requêtes de jumeaux|


## <a name="microsoftdevicesprovisioningservices"></a>Microsoft.Devices/provisioningServices

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|DeviceOperations|Opérations sur les appareils|
|ServiceOperations|Opérations de service|


## <a name="microsoftdocumentdbdatabaseaccounts"></a>Microsoft.DocumentDB/databaseAccounts

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|CassandraRequests|CassandraRequests|
|ControlPlaneRequests|ControlPlaneRequests|
|DataPlaneRequests|DataPlaneRequests|
|GremlinRequests|GremlinRequests|
|MongoRequests|MongoRequests|
|PartitionKeyRUConsumption|PartitionKeyRUConsumption|
|PartitionKeyStatistics|PartitionKeyStatistics|
|QueryRuntimeStatistics|QueryRuntimeStatistics|


## <a name="microsofteventgriddomains"></a>Microsoft.EventGrid/domains

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|DeliveryFailures|Journaux d’échec de remise|
|PublishFailures|Journaux d’échec de publication|


## <a name="microsofteventgridsystemtopics"></a>Microsoft.EventGrid/systemTopics

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|DeliveryFailures|Journaux d’échec de remise|


## <a name="microsofteventgridtopics"></a>Microsoft.EventGrid/topics

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|DeliveryFailures|Journaux d’échec de remise|
|PublishFailures|Journaux d’échec de publication|


## <a name="microsofteventhubnamespaces"></a>Microsoft.EventHub/namespaces

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|ArchiveLogs|Journaux d’activité d’archivage|
|AutoScaleLogs|Journaux d’activité de mise à l’échelle automatique|
|CustomerManagedKeyUserLogs|Journaux de clés gérées par le client|
|EventHubVNetConnectionEvent|Journaux de connexion de réseau virtuel/Filtrage IP|
|KafkaCoordinatorLogs|Journaux du coordinateur Kafka|
|KafkaUserErrorLogs|Journaux des erreurs d’utilisateur Kafka|
|OperationalLogs|Journaux d’activité des opérations|


## <a name="microsofthealthcareapisservices"></a>Microsoft.HealthcareApis/services

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|AuditLogs|Journaux d’audit|


## <a name="microsoftinsightsautoscalesettings"></a>Microsoft.Insights/AutoscaleSettings

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|AutoscaleEvaluations|Évaluations de mise à l’échelle automatique|
|AutoscaleScaleActions|Actions de mise à l’échelle automatique|


## <a name="microsoftinsightscomponents"></a>Microsoft.Insights/Components

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|AppAvailabilityResults|Résultats de la disponibilité|
|AppBrowserTimings|Minutages du navigateur|
|AppDependencies|Les dépendances|
|AppEvents|Événements|
|AppExceptions|Exceptions|
|AppMetrics|Mesures|
|AppPageViews|Affichages de page|
|AppPerformanceCounters|Compteurs de performance|
|AppRequests|Demandes|
|AppSystemEvents|Événements système|
|AppTraces|Traces|


## <a name="microsoftkeyvaultvaults"></a>Microsoft.KeyVault/vaults

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|AuditEvent|Journaux d’audit|


## <a name="microsoftkustoclusters"></a>Microsoft.Kusto/Clusters

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|Commande|Commande|
|FailedIngestion|Opérations d’ingestion ayant échoué|
|IngestionBatching|Traitement par lot de l’ingestion|
|Requête|Requête|
|SucceededIngestion|Opérations d’ingestion réussies|
|TableDetails|Détails de la table|
|TableUsageStatistics|Statistiques d’utilisation de la table|


## <a name="microsoftlogicintegrationaccounts"></a>Microsoft.Logic/integrationAccounts

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|IntegrationAccountTrackingEvents|Suivi des événements de compte d’intégration|


## <a name="microsoftlogicworkflows"></a>Microsoft.Logic/workflows

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|WorkflowRuntime|Événements de diagnostic de runtime de workflow|


## <a name="microsoftmachinelearningservicesworkspaces"></a>Microsoft.MachineLearningServices/workspaces

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|AmlComputeClusterEvent|AmlComputeClusterEvent|
|AmlComputeClusterNodeEvent|AmlComputeClusterNodeEvent|
|AmlComputeCpuGpuUtilization|AmlComputeCpuGpuUtilization|
|AmlComputeJobEvent|AmlComputeJobEvent|
|AmlRunStatusChangedEvent|AmlRunStatusChangedEvent|


## <a name="microsoftmediamediaservices"></a>Microsoft.Media/mediaservices

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|KeyDeliveryRequests|Service de remise de clé|


## <a name="microsoftnetworkapplicationgateways"></a>Microsoft.Network/applicationGateways

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|ApplicationGatewayAccessLog|Journal d’accès à la passerelle d’application|
|ApplicationGatewayFirewallLog|Journal de pare-feu de la passerelle d’application|
|ApplicationGatewayPerformanceLog|Journal de performance de la passerelle d’application|


## <a name="microsoftnetworkazurefirewalls"></a>Microsoft.Network/azurefirewalls

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|AzureFirewallApplicationRule|Règle d’application de pare-feu Azure|
|AzureFirewallNetworkRule|Règle de réseau de pare-feu Azure|


## <a name="microsoftnetworkbastionhosts"></a>Microsoft.Network/bastionHosts

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|BastionAuditLogs|Journaux d’audit de Bastion|


## <a name="microsoftnetworkexpressroutecircuits"></a>Microsoft.Network/expressRouteCircuits

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|PeeringRouteLog|Journaux d’activité de table de routage de Peering|


## <a name="microsoftnetworkfrontdoors"></a>Microsoft.Network/frontdoors

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|FrontdoorAccessLog|Journal des accès Frontdoor|
|FrontdoorWebApplicationFirewallLog|Journal de pare-feu d’applications web Frontdoor|


## <a name="microsoftnetworkloadbalancers"></a>Microsoft.Network/loadBalancers

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|LoadBalancerAlertEvent|Événements d’alerte d’équilibreur de charge|
|LoadBalancerProbeHealthStatus|État d’intégrité de la sonde d’équilibreur de charge|


## <a name="microsoftnetworknetworksecuritygroups"></a>Microsoft.Network/networksecuritygroups

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|NetworkSecurityGroupEvent|Événement de groupe de sécurité réseau|
|NetworkSecurityGroupFlowEvent|Événement de flux de règle de groupe de sécurité réseau|
|NetworkSecurityGroupRuleCounter|Compteur de règle de groupe de sécurité réseau|


## <a name="microsoftnetworkpublicipaddresses"></a>Microsoft.Network/publicIPAddresses

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|DDoSMitigationFlowLogs|Journaux d’activité des flux des décisions d’atténuation DDoS|
|DDoSMitigationReports|Rapports des mesures d’atténuation DDoS|
|DDoSProtectionNotifications|Notifications de protection DDoS|


## <a name="microsoftnetworktrafficmanagerprofiles"></a>Microsoft.Network/trafficManagerProfiles

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|ProbeHealthStatusEvents|Événement de résultats d’intégrité de sondage Traffic Manager|


## <a name="microsoftnetworkvirtualnetworkgateways"></a>Microsoft.Network/virtualNetworkGateways

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|GatewayDiagnosticLog|Journaux de diagnostic de passerelle|
|IKEDiagnosticLog|Journaux de diagnostics IKE|
|P2SDiagnosticLog|Journaux de diagnostics PS2|
|RouteDiagnosticLog|Journaux de diagnostic d’itinéraires|
|TunnelDiagnosticLog|Journaux de diagnostic de tunnel|


## <a name="microsoftnetworkvirtualnetworks"></a>Microsoft.Network/virtualNetworks

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|VMProtectionAlerts|Alertes de protection de machine virtuelle|


## <a name="microsoftpowerbidedicatedcapacities"></a>Microsoft.PowerBIDedicated/capacities

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|Moteur|Moteur|


## <a name="microsoftrecoveryservicesvaults"></a>Microsoft.RecoveryServices/Vaults

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|AddonAzureBackupAlerts|Extension Données d’alerte de sauvegarde Azure|
|AddonAzureBackupJobs|Extension Données du travail de sauvegarde Azure|
|AddonAzureBackupPolicy|Extension Données de stratégie de sauvegarde Azure|
|AddonAzureBackupProtectedInstance|Extension Données d’instance protégée de sauvegarde Azure|
|AddonAzureBackupStorage|Extension Données de stockage de sauvegarde Azure|
|AzureBackupReport|Données de rapport de sauvegarde Azure|
|AzureSiteRecoveryEvents|Événements Azure Site Recovery|
|AzureSiteRecoveryJobs|Travaux Azure Site Recovery|
|AzureSiteRecoveryProtectedDiskDataChurn|Activité des données de disque protégé Azure Site Recovery|
|AzureSiteRecoveryRecoveryPoints|Points de récupération Azure Site Recovery|
|AzureSiteRecoveryReplicatedItems|Éléments répliqués d’Azure Site Recovery|
|AzureSiteRecoveryReplicationDataUploadRate|Taux de chargement de données de réplication Azure Site Recovery|
|AzureSiteRecoveryReplicationStats|Statistiques de réplication Azure Site Recovery|
|CoreAzureBackup|Données de sauvegarde Azure de base|


## <a name="microsoftrelaynamespaces"></a>Microsoft.Relay/namespaces

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|HybridConnectionsEvent|Événements HybridConnections|


## <a name="microsoftsearchsearchservices"></a>Microsoft.Search/searchServices

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|OperationLogs|Journaux d’activité des opérations|


## <a name="microsoftservicebusnamespaces"></a>Microsoft.ServiceBus/namespaces

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|OperationalLogs|Journaux d’activité des opérations|


## <a name="microsoftsignalrservicesignalr"></a>Microsoft.SignalRService/SignalR

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|AllLogs|Journaux d’Azure SignalR Service.|


## <a name="microsoftsqlmanagedinstances"></a>Microsoft.Sql/managedInstances

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|DevOpsOperationsAudit|Journaux d’audit des opérations de Devops|
|ResourceUsageStats|Statistiques d’utilisation des ressources|
|SQLSecurityAuditEvents|Événement d’audit de sécurité SQL|


## <a name="microsoftsqlmanagedinstancesdatabases"></a>Microsoft.Sql/managedInstances/databases

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|Erreurs|Erreurs|
|QueryStoreRuntimeStatistics|Statistiques d’exécution du magasin de données des requêtes|
|QueryStoreWaitStatistics|Statistiques d’attente du magasin de requêtes|
|SQLInsights|SQL Insights|


## <a name="microsoftsqlserversdatabases"></a>Microsoft.Sql/servers/databases

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|AutomaticTuning|Réglage automatique|
|Blocs|Blocs|
|DatabaseWaitStatistics|Statistiques d’attente de base de données|
|Blocages|Blocages|
|DevOpsOperationsAudit|Journaux d’audit des opérations de Devops|
|DmsWorkers|Rôles de travail Dms|
|Erreurs|Erreurs|
|ExecRequests|Requêtes d’exécution|
|QueryStoreRuntimeStatistics|Statistiques d’exécution du magasin de données des requêtes|
|QueryStoreWaitStatistics|Statistiques d’attente du magasin de requêtes|
|RequestSteps|Étapes de la requête|
|SQLInsights|SQL Insights|
|SqlRequests|Requêtes SQL|
|SQLSecurityAuditEvents|Événement d’audit de sécurité SQL|
|Délais d’expiration|Délais d’expiration|
|Attend|Attend|


## <a name="microsoftstoragestorageaccountsblobservices"></a>Microsoft.Storage/storageAccounts/blobServices

Coût : Payé comme indiqué dans la section Journaux de plateforme de la [page Tarification Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/). 

|Category |Nom d’affichage de la catégorie|
|---|---|
|StorageDelete|StorageDelete|
|StorageRead|StorageRead|
|StorageWrite|StorageWrite|


## <a name="microsoftstoragestorageaccountsfileservices"></a>Microsoft.Storage/storageAccounts/fileServices

Coût : Payé comme indiqué dans la section Journaux de plateforme de la [page Tarification Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/). 

|Category |Nom d’affichage de la catégorie|
|---|---|
|StorageDelete|StorageDelete|
|StorageRead|StorageRead|
|StorageWrite|StorageWrite|


## <a name="microsoftstoragestorageaccountsqueueservices"></a>Microsoft.Storage/storageAccounts/queueServices

Coût : Payé comme indiqué dans la section Journaux de plateforme de la [page Tarification Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/). 
 
|Category |Nom d’affichage de la catégorie|
|---|---|
|StorageDelete|StorageDelete|
|StorageRead|StorageRead|
|StorageWrite|StorageWrite|


## <a name="microsoftstoragestorageaccountstableservices"></a>Microsoft.Storage/storageAccounts/tableServices

Coût : Payé comme indiqué dans la section Journaux de plateforme de la [page Tarification Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/). 
 
|Category |Nom d’affichage de la catégorie|
|---|---|
|StorageDelete|StorageDelete|
|StorageRead|StorageRead|
|StorageWrite|StorageWrite|


## <a name="microsoftstreamanalyticsstreamingjobs"></a>Microsoft.StreamAnalytics/streamingjobs

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|Création|Création|
|Exécution|Exécution|


## <a name="microsoftsynapseworkspaces"></a>Microsoft.Synapse/workspaces

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|BuiltinSqlReqsEnded|Demandes terminées pour le pool SQL intégré|
|GatewayApiRequests|Demandes d’API de passerelle Synapse|
|SQLSecurityAuditEvents|Événement d’audit de sécurité SQL|
|SynapseRbacOperations|Opérations RBAC Synapse|


## <a name="microsoftsynapseworkspacesbigdatapools"></a>Microsoft.Synapse/workspaces/bigDataPools

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|BigDataPoolAppsEnded|Applications terminées pour le pool Big Data|


## <a name="microsoftsynapseworkspacessqlpools"></a>Microsoft.Synapse/workspaces/sqlPools

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|DmsWorkers|Rôles de travail Dms|
|ExecRequests|Requêtes d’exécution|
|RequestSteps|Étapes de la requête|
|SqlRequests|Requêtes SQL|
|SQLSecurityAuditEvents|Événement d’audit de sécurité SQL|
|Attend|Attend|


## <a name="microsoftwebhostingenvironments"></a>microsoft.web/hostingenvironments

Coût : Gratuit 

|Category |Nom d’affichage de la catégorie|
|---|---|
|AppServiceEnvironmentPlatformLogs|Journaux de plateforme de l’environnement App Service|


## <a name="microsoftwebsites"></a>microsoft.web/sites

Coût : Gratuit 


|Category |Nom d’affichage de la catégorie|
|---|---|
|AppServiceAppLogs|Journaux d’application App Service|
|AppServiceAuditLogs|Journaux d’audit d’accès|
|AppServiceConsoleLogs|Journaux de la console App Service|
|AppServiceFileAuditLogs|Journaux d’audit de modification de contenu de site|
|AppServiceHTTPLogs|Journaux HTTP|
|FunctionAppLogs|Journaux d’application de fonction|


## <a name="microsoftwebsitesslots"></a>microsoft.web/sites/slots

Coût : Gratuit 


|Category |Nom d’affichage de la catégorie|
|---|---|
|AppServiceAppLogs|Journaux d’application App Service|
|AppServiceAuditLogs|Journaux d’audit d’accès|
|AppServiceConsoleLogs|Journaux de la console App Service|
|AppServiceFileAuditLogs|Journaux d’audit de modification de contenu de site|
|AppServiceHTTPLogs|Journaux HTTP|
|FunctionAppLogs|Journaux d’application de fonction|


## <a name="next-steps"></a>Étapes suivantes

* [En savoir plus sur les journaux de ressources](./platform-logs-overview.md)
* [Diffuser en continu les journaux de ressource vers **Event Hubs**](./resource-logs.md#send-to-azure-event-hubs)
* [Modifier les paramètres de diagnostic de journal de ressource à l’aide de l’API REST Azure Monitor](/rest/api/monitor/diagnosticsettings)
* [Analyser les journaux d’activité du stockage Azure avec Log Analytics](./resource-logs.md#send-to-log-analytics-workspace)

