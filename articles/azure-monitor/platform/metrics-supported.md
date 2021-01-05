---
title: Métriques prises en charge par Azure Monitor par type de ressource
description: Liste des métriques disponibles pour chaque type de ressource avec Azure Monitor.
author: rboucher
services: azure-monitor
ms.topic: reference
ms.date: 07/16/2020
ms.author: robb
ms.subservice: metrics
ms.openlocfilehash: 1741331c57427c57507376afdb878edc18259b72
ms.sourcegitcommit: 96918333d87f4029d4d6af7ac44635c833abb3da
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/04/2020
ms.locfileid: "93306837"
---
# <a name="supported-metrics-with-azure-monitor"></a>Métriques prises en charge avec Azure Monitor

> [!NOTE]
> Cette liste est en grande partie générée automatiquement à partir de l’API REST de métriques Azure Monitor. Toute modification apportée à cette liste via GitHub peut être remplacée sans avertissement. Pour plus d’informations sur la façon d’effectuer des mises à jour permanentes, contactez l’auteur de cet article.

Azure Monitor offre plusieurs moyens d’interagir avec les métriques, y compris en créant des graphiques dans le portail, en y accédant via l’API REST ou en envoyant des requêtes avec PowerShell ou l’interface CLI. 

Cet article est une liste complète de toutes les métriques de plateforme (c’est-à-dire collectées automatiquement) actuellement disponibles avec le pipeline de métriques consolidées d’Azure Monitor. La liste a été mise à jour pour la dernière fois le 27 mars 2020. Les métriques modifiées ou ajoutées après cette date peuvent ne pas apparaître ci-dessous. Pour interroger cette liste de métriques et y accéder programmatiquement, veuillez utiliser [2018-01-01 api-version](/rest/api/monitor/metricdefinitions). D’autres métriques ne figurant pas dans cette liste peuvent être disponibles dans le portail ou via les API héritées.

Les métriques sont organisées par fournisseurs de ressources et par type de ressource. Pour obtenir la liste des services et des fournisseurs de ressources qui leur appartiennent, consultez [Fournisseurs de ressources pour les services Azure](../../azure-resource-manager/management/azure-services-resource-providers.md). 

## <a name="exporting-platform-metrics-to-other-locations"></a>Exportation des métriques de plateforme vers d’autres emplacements

Vous pouvez exporter les métriques de plateforme à partir du pipeline Azure Monitor vers d’autres emplacements de l’une des deux façons suivantes.
1. Utilisation de l’[API REST pour les métriques](/rest/api/monitor/metrics/list)
2. Utilisez les [paramètres de diagnostic](diagnostic-settings.md) pour acheminer les métriques de plateforme vers les emplacements suivants : 
    - Stockage Azure
    - Journaux Azure Monitor (et donc Log Analytics)
    - Event Hubs, qui permet de les transmettre à des systèmes autres que Microsoft 

Si le recours aux paramètres de diagnostic constitue le moyen le plus simple d’acheminer les métriques, certaines limitations s’appliquent : 

- **Métriques non exportables** : toutes les métriques sont exportables à l’aide de l’API REST, mais certaines ne peuvent pas être exportées à l’aide des paramètres de diagnostic en raison de subtilités dans le système backend Azure Monitor. La colonne *Exportable au moyen des paramètres de diagnostic* des tableaux ci-dessous indique quelles métriques peuvent être exportées de cette façon.  

- **Métriques multidimensionnelles** : il n’est pas possible à l’heure actuelle d’envoyer des métriques multidimensionnelles à d’autres emplacements par le biais des paramètres de diagnostic. Les métriques à plusieurs dimensions sont exportées en tant que métriques dimensionnelles uniques aplaties, puis agrégées dans les valeurs de la dimension. *Par exemple* : La métrique « Messages entrants » sur un Event Hub peut être examinée et représentée sur un niveau par file d’attente. Toutefois, lors de l’exportation via les paramètres de diagnostic, la métrique est représentée sous la forme de tous les messages entrants, dans toutes les files d’attente de l’Event Hub.

## <a name="guest-os-and-host-os-metrics"></a>Métriques du système d’exploitation invité et du système d’exploitation hôte

> [!WARNING]
> Les métriques du système d’exploitation invité (SE invité) qui s’exécutent dans les Machines virtuelles Azure, Service Fabric et les Services cloud ne sont **PAS** répertoriées ici. Les métriques du SE invité doivent être collectées au moyen du ou des agents qui s’exécutent sur ou dans le cadre du SE invité.  Parmi ces métriques, on peut citer les compteurs de performances qui effectuent le suivi du pourcentage de processeur invité et de l’utilisation de la mémoire, fréquemment utilisés pour la mise à l’échelle automatique et la génération d’alertes. 
>
> **Les métriques du SE hôte SONT disponibles et répertoriées ci-dessous.** Ce ne sont pas les mêmes. Les métriques du SE hôte sont liées à la session Hyper-V qui héberge la session du SE invité. 

> [!TIP]
> Il est recommandé d’utiliser et de configurer [l’extension Diagnostics Azure](diagnostics-extension-overview.md) pour envoyer les métriques de performance du SE invité dans la base de données de métriques Azure Monitor où sont stockées les métriques de la plateforme. L’extension achemine les métriques du SE invité par le biais de l’API de [métriques personnalisées](metrics-custom-overview.md). Vous pouvez alors représenter ces métriques (par exemple, les métriques de plateforme) sous forme graphique, générer des alertes, etc. Il est également possible d’utiliser l’agent Log Analytics pour envoyer les métriques du SE invité aux journaux Azure Monitor/à Log Analytics. Vous pouvez alors interroger ces métriques en association avec d’autres types de données. 

Pour obtenir des informations complémentaires importantes, consultez [Vue d’ensemble des agents de surveillance](agents-overview.md).    

## <a name="table-formatting"></a>Mise en forme des tableaux

> [!IMPORTANT] 
> Dans cette dernière mise à jour a été ajoutée une nouvelle colonne ; les métriques ont par ailleurs été réorganisées par ordre alphabétique. En raison de ces informations complémentaires, une barre de défilement horizontale est susceptible d’apparaître en bas des tableaux ci-dessous, en fonction de la largeur de votre fenêtre de navigateur. Si vous pensez qu’il manque des informations, utilisez la barre de défilement pour afficher l’intégralité du tableau.


## <a name="microsoftanalysisservicesservers"></a>Microsoft.AnalysisServices/servers

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|CleanerCurrentPrice|Oui|Mémoire : prix actuel du nettoyage|Count|Average|Prix actuel de la mémoire, $/octet/temps, normalisé à 1000.|ServerResourceType|
|CleanerMemoryNonshrinkable|Oui|Mémoire : mémoire de nettoyage non réductible|Octets|Average|Quantité de mémoire, en octets, qui ne doit pas être vidée par le nettoyage en arrière-plan.|ServerResourceType|
|CleanerMemoryShrinkable|Oui|Mémoire : mémoire de nettoyage réductible|Octets|Average|Quantité de mémoire, en octets, qui doit être vidée par le nettoyage en arrière-plan.|ServerResourceType|
|CommandPoolBusyThreads|Oui|Threads : threads occupés du pool de commandes|Count|Average|Nombre de threads occupés dans le pool de threads de commandes.|ServerResourceType|
|CommandPoolIdleThreads|Oui|Threads : threads inactifs du pool de commandes|Count|Average|Nombre de threads inactifs dans le pool de threads de commandes.|ServerResourceType|
|CommandPoolJobQueueLength|Oui|Longueur de la file d’attente des travaux du pool de commandes|Count|Average|Nombre de travaux contenus dans la file d’attente du pool de threads de commandes.|ServerResourceType|
|CurrentConnections|Oui|Connexion : Connexions en cours|Count|Average|Nombre actuel de connexions client établies.|ServerResourceType|
|CurrentUserSessions|Oui|Sessions utilisateur actuelles|Count|Average|Nombre actuel de sessions utilisateur établies.|ServerResourceType|
|LongParsingBusyThreads|Oui|Threads : threads occupés d'analyse longue|Count|Average|Nombre de threads occupés dans le pool de threads d’analyse longue.|ServerResourceType|
|LongParsingIdleThreads|Oui|Threads : threads inactifs d'analyse longue|Count|Average|Nombre de threads inactifs dans le pool de threads d’analyse longue.|ServerResourceType|
|LongParsingJobQueueLength|Oui|Threads : longueur de la file d'attente des travaux d'analyse longue|Count|Average|Nombre de travaux contenus dans la file d’attente du pool de threads d’analyse longue.|ServerResourceType|
|mashup_engine_memory_metric|Oui|Mémoire du moteur M|Octets|Average|Utilisation de la mémoire par les processus de moteur mashup|ServerResourceType|
|mashup_engine_private_bytes_metric|Oui|Octets privés du moteur M|Octets|Average|Utilisation des octets privés par les processus de moteur mashup.|ServerResourceType|
|mashup_engine_qpu_metric|Oui|QPU du moteur M|Count|Average|Utilisation des QPU par les processus de moteur mashup|ServerResourceType|
|mashup_engine_virtual_bytes_metric|Oui|Octets virtuels du moteur M|Octets|Average|Utilisation des octets virtuels par les processus de moteur mashup.|ServerResourceType|
|memory_metric|Oui|Mémoire|Octets|Average|Mémoire. Plage de 0 à 25 Go pour S1, de 0 à 50 Go pour S2 et de 0 à 100 Go pour S4|ServerResourceType|
|memory_thrashing_metric|Oui|Vidage de mémoire|Pourcentage|Average|Vidage de mémoire moyenne.|ServerResourceType|
|MemoryLimitHard|Oui|Mémoire : limite inconditionnelle de la mémoire|Octets|Average|Limite de mémoire physique, du fichier de configuration.|ServerResourceType|
|MemoryLimitHigh|Oui|Mémoire : limite haute de la mémoire|Octets|Average|Limite de mémoire élevée, du fichier de configuration.|ServerResourceType|
|MemoryLimitLow|Oui|Mémoire : limite basse de la mémoire|Octets|Average|Limite de mémoire basse, du fichier de configuration.|ServerResourceType|
|MemoryLimitVertiPaq|Oui|Mémoire : limite de la mémoire VertiPaq|Octets|Average|Limite en mémoire, du fichier de configuration.|ServerResourceType|
|MemoryUsage|Oui|Mémoire : Utilisation de la mémoire|Octets|Average|Utilisation de la mémoire du processus serveur telle qu’utilisée dans le calcul du coût de la mémoire de nettoyage. Équivaut au compteur Process\PrivateBytes, plus la taille des données mappées en mémoire, en ignorant la mémoire mappée ou allouée par le moteur d’analyse de mémoire xVelocity (VertiPaq) dépassant la limite de mémoire du moteur xVelocity.|ServerResourceType|
|private_bytes_metric|Oui|Octets privés|Octets|Average|Octets privés.|ServerResourceType|
|ProcessingPoolBusyIOJobThreads|Oui|Threads : threads des travaux d'E/S occupés du pool de traitement|Count|Average|Nombre de threads pour les travaux d’E/S en cours d’exécution dans le pool de threads de traitement.|ServerResourceType|
|ProcessingPoolBusyNonIOThreads|Oui|Threads : threads autres que les threads d'E/S occupés du pool de traitement|Count|Average|Nombre de threads pour les travaux autres que d’E/S en cours d’exécution dans le pool de threads de traitement.|ServerResourceType|
|ProcessingPoolIdleIOJobThreads|Oui|Threads : threads des travaux d'E/S inactifs du pool de traitement|Count|Average|Nombre de threads inactifs pour les travaux d’E/S le pool de threads de traitement.|ServerResourceType|
|ProcessingPoolIdleNonIOThreads|Oui|Threads : threads autres que les threads d'E/S inactifs du pool de traitement|Count|Average|Nombre de threads inactifs le pool de threads de traitement dédiés aux travaux autres qu’E/S.|ServerResourceType|
|ProcessingPoolIOJobQueueLength|Oui|Threads : longueur de la file des travaux d'E/S du pool de traitement|Count|Average|Nombre de travaux d’E/S contenus dans la file d’attente du pool de threads de traitement.|ServerResourceType|
|ProcessingPoolJobQueueLength|Oui|Longueur de la file d’attente des travaux du pool de traitement|Count|Average|Nombre de travaux autres que d’E/S contenus dans la file d’attente du pool de threads de traitement.|ServerResourceType|
|qpu_metric|Oui|QPU|Count|Average|QPU. Plage de 0 à 100 pour S1, de 0 à 200 pour S2 et de 0 à 400 pour S4|ServerResourceType|
|QueryPoolBusyThreads|Oui|Threads occupés du pool de threads de requêtes|Count|Average|Nombre de threads occupés dans le pool de threads de requêtes.|ServerResourceType|
|QueryPoolIdleThreads|Oui|Threads : threads inactifs du pool de requêtes|Count|Average|Nombre de threads inactifs pour les travaux d’E/S le pool de threads de traitement.|ServerResourceType|
|QueryPoolJobQueueLength|Oui|Threads : longueur de la file d'attente des travaux du pool de requêtes|Count|Average|Nombre de travaux contenus dans la file d’attente du pool de threads de requêtes.|ServerResourceType|
|Quota|Oui|Mémoire : Quota|Octets|Average|Quota de mémoire actuel, en octets. Le quota de mémoire est également appelé réserve de mémoire ou d’allocation.|ServerResourceType|
|QuotaBlocked|Oui|Mémoire : quota bloqué|Count|Average|Nombre actuel de requêtes de quota qui sont bloquées en attendant la libération d’autres quotas de mémoire.|ServerResourceType|
|RowsConvertedPerSec|Oui|Traitement : lignes converties par seconde|CountPerSecond|Average|Taux de lignes converties lors du traitement.|ServerResourceType|
|RowsReadPerSec|Oui|Traitement : lignes lues par seconde|CountPerSecond|Average|Taux de lignes lues à partir de toutes les bases de données relationnelles.|ServerResourceType|
|RowsWrittenPerSec|Oui|Traitement : lignes écrites par seconde|CountPerSecond|Average|Taux de lignes écrites lors du traitement.|ServerResourceType|
|ShortParsingBusyThreads|Oui|Threads : threads occupés d'analyse courte|Count|Average|Nombre de threads occupés dans le pool de threads d’analyse courte.|ServerResourceType|
|ShortParsingIdleThreads|Oui|Threads : threads inactifs d'analyse courte|Count|Average|Nombre de threads inactifs dans le pool de threads d’analyse courte.|ServerResourceType|
|ShortParsingJobQueueLength|Oui|Threads : longueur de la file d'attente des travaux d'analyse courte|Count|Average|Nombre de travaux contenus dans la file d’attente du pool de threads d’analyse courte.|ServerResourceType|
|SuccessfullConnectionsPerSec|Oui|Connexions réussies par seconde|CountPerSecond|Average|Taux de connexions terminées réussies.|ServerResourceType|
|TotalConnectionFailures|Oui|Nombre total d’échecs de connexion|Count|Average|Total des échecs de tentatives de connexion.|ServerResourceType|
|TotalConnectionRequests|Oui|Nombre total de demandes de connexion|Count|Average|Nombre total de demandes de connexion. Il s’agit des arrivées.|ServerResourceType|
|VertiPaqNonpaged|Oui|Mémoire : mémoire non paginée VertiPaq|Octets|Average|Octets de mémoire verrouillée dans la plage de travail pour utilisation par le moteur en mémoire.|ServerResourceType|
|VertiPaqPaged|Oui|Mémoire : mémoire paginée VertiPaq|Octets|Average|Octets de mémoire paginée utilisée pour les données en mémoire.|ServerResourceType|
|virtual_bytes_metric|Oui|Octets virtuels|Octets|Average|Octets virtuels.|ServerResourceType|


## <a name="microsoftapimanagementservice"></a>Microsoft.ApiManagement/service

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|BackendDuration|Oui|Durées des demandes back-end|Millisecondes|Average|Durée des demandes back-end, en millisecondes|Emplacement, nom d’hôte|
|Capacité|Oui|Capacité|Pourcentage|Average|Métrique d’utilisation pour le service ApiManagement|Emplacement|
|Duration|Oui|Durée globale des demandes de passerelle|Millisecondes|Average|Durée globale des demandes de passerelle en millisecondes|Emplacement, nom d’hôte|
|EventHubDroppedEvents|Oui|Événements EventHub supprimés|Count|Total|Nombre d’événements ignorés car la limite de taille de la file d’attente a été atteinte|Emplacement|
|EventHubRejectedEvents|Oui|Événements EventHub rejetés|Count|Total|Nombre d’événements EventHub rejetés (configuration incorrecte ou non autorisée)|Emplacement|
|EventHubSuccessfulEvents|Oui|Événements EventHub réussis|Count|Total|Nombre d’événements EventHub réussis|Emplacement|
|EventHubThrottledEvents|Oui|Événements EventHub limités|Count|Total|Nombre d’événements EventHub limités|Emplacement|
|EventHubTimedoutEvents|Oui|Événements EventHub expirés|Count|Total|Nombre d'événements EventHub expirés|Emplacement|
|EventHubTotalBytesSent|Oui|Taille des événements EventHub|Octets|Total|Taille totale des événements EventHub en octets|Emplacement|
|EventHubTotalEvents|Oui|Nombre total d'événements EventHub|Count|Total|Nombre d’événements envoyés à EventHub|Emplacement|
|EventHubTotalFailedEvents|Oui|Événements EventHub non réussis|Count|Total|Nombre d’événements EventHub non réussis|Emplacement|
|FailedRequests|Oui|Demandes de la passerelle ayant échoué (déprécié)|Count|Total|Nombre de défaillances des demandes de la passerelle : utilisez une métrique de demande multi-dimension avec la dimension GatewayResponseCodeCategory à la place|Emplacement, nom d’hôte|
|NetworkConnectivity|Oui|État de connectivité réseau des ressources (préversion)|Count|Average|État de connectivité réseau des types de ressources dépendants du service Gestion des API|Location, ResourceType|
|OtherRequests|Oui|Autres demandes de la passerelle (déprécié)|Count|Total|Nombre d’autres demandes de la passerelle : utilisez une métrique de demande multi-dimension avec la dimension GatewayResponseCodeCategory à la place|Emplacement, nom d’hôte|
|Demandes|Oui|Demandes|Count|Total|Métriques de demande de la passerelle avec plusieurs dimensions|Location, Hostname, LastErrorReason, BackendResponseCode, GatewayResponseCode, BackendResponseCodeCategory, GatewayResponseCodeCategory|
|SuccessfulRequests|Oui|Demandes de la passerelle ayant abouti (déprécié)|Count|Total|Nombre de demandes de la passerelle ayant abouti : utilisez une métrique de demande multi-dimension avec la dimension GatewayResponseCodeCategory à la place|Emplacement, nom d’hôte|
|TotalRequests|Oui|Nombre total de demandes de la passerelle (déprécié)|Count|Total|Nombre de demandes de la passerelle : utilisez une métrique de demande multi-dimension avec la dimension GatewayResponseCodeCategory à la place|Emplacement, nom d’hôte|
|UnauthorizedRequests|Oui|Demandes de la passerelle non autorisées (déprécié)|Count|Total|Nombre de demandes de la passerelle non autorisées : utilisez une métrique de demande multi-dimension avec la dimension GatewayResponseCodeCategory à la place|Emplacement, nom d’hôte|


## <a name="microsoftappconfigurationconfigurationstores"></a>Microsoft.AppConfiguration/configurationStores

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|HttpIncomingRequestCount|Oui|HttpIncomingRequestCount|Count|Count|Nombre total de requêtes HTTP entrantes.|StatusCode, Authentication|
|HttpIncomingRequestDuration|Oui|HttpIncomingRequestDuration|Count|Average|Latence sur une requête HTTP.|StatusCode, Authentication|
|ThrottledHttpRequestCount|Oui|ThrottledHttpRequestCount|Count|Count|Requêtes HTTP limitées.|Aucune dimension|


## <a name="microsoftappplatformspring"></a>Microsoft.AppPlatform/Spring

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|jvm.gc.live.data.size|Oui|jvm.gc.live.data.size|Octets|Average|Taille du pool de mémoire d’ancienne génération après un GC complet|Deployment, AppName, Pod|
|jvm.gc.max.data.size|Oui|jvm.gc.max.data.size|Octets|Average|Taille maximale du pool de mémoire d’ancienne génération|Deployment, AppName, Pod|
|jvm.gc.memory.allocated|Oui|jvm.gc.memory.allocated|Octets|Maximale|Incrémenté pour une augmentation de la taille du pool de mémoire de nouvelle génération après un GC avant le suivant|Deployment, AppName, Pod|
|jvm.gc.memory.promoted|Oui|jvm.gc.memory.promoted|Octets|Maximale|Nombre d’augmentations positives de la taille du pool de mémoire d’ancienne génération avant l’application du GC jusqu’au terme de cette application|Deployment, AppName, Pod|
|jvm.gc.pause.total.count|Oui|jvm.gc.pause.total.count|Count|Total|Nombre d’interruptions du GC|Deployment, AppName, Pod|
|jvm.gc.pause.total.count|Oui|jvm.gc.pause.total.count|Millisecondes|Total|Durée totale de pause du GC|Deployment, AppName, Pod|
|jvm.memory.committed|Oui|jvm.memory.committed|Octets|Average|Mémoire affectée à JVM en octets|Deployment, AppName, Pod|
|jvm.memory.max|Oui|jvm.memory.max|Octets|Maximale|Quantité maximale de mémoire en octets utilisable pour la gestion de la mémoire|Deployment, AppName, Pod|
|jvm.memory.used|Oui|jvm.memory.used|Octets|Average|Mémoire d’application utilisée en octets|Deployment, AppName, Pod|
|process.cpu.usage|Oui|process.cpu.usage|Pourcentage|Average|Pourcentage d’utilisation du processeur JVM d’application|Deployment, AppName, Pod|
|system.cpu.usage|Oui|system.cpu.usage|Pourcentage|Average|Utilisation récente du processeur pour l’ensemble du système|Deployment, AppName, Pod|
|tomcat.global.error|Oui|tomcat.global.error|Count|Total|Erreur globale Tomcat|Deployment, AppName, Pod|
|tomcat.global.received|Oui|tomcat.global.received|Octets|Total|Nombre total d’octets reçus par Tomcat|Deployment, AppName, Pod|
|tomcat.global.request.avg.time|Oui|tomcat.global.request.avg.time|Millisecondes|Average|Durée moyenne des demandes Tomcat|Deployment, AppName, Pod|
|tomcat.global.request.max|Oui|tomcat.global.request.max|Millisecondes|Maximale|Durée maximale des demandes Tomcat|Deployment, AppName, Pod|
|tomcat.global.request.total.count|Oui|tomcat.global.request.total.count|Count|Total|Nombre total de demandes Tomcat|Deployment, AppName, Pod|
|tomcat.global.request.total.time|Oui|tomcat.global.request.total.time|Millisecondes|Total|Durée totale des requêtes Tomcat|Deployment, AppName, Pod|
|tomcat.global.sent|Oui|tomcat.global.sent|Octets|Total|Nombre total d’octets envoyés par Tomcat|Deployment, AppName, Pod|
|tomcat.sessions.active.current|Oui|tomcat.sessions.active.current|Count|Total|Nombre de sessions actives Tomcat|Deployment, AppName, Pod|
|tomcat.sessions.active.max|Oui|tomcat.sessions.active.max|Count|Total|Nombre maximal de sessions actives Tomcat|Deployment, AppName, Pod|
|tomcat.sessions.alive.max|Oui|tomcat.sessions.alive.max|Millisecondes|Maximale|Durée maximale des sessions actives Tomcat|Deployment, AppName, Pod|
|tomcat.sessions.created|Oui|tomcat.sessions.created|Count|Total|Nombre de sessions Tomcat créées|Deployment, AppName, Pod|
|tomcat.sessions.expired|Oui|tomcat.sessions.expired|Count|Total|Nombre de sessions Tomcat ayant expiré|Deployment, AppName, Pod|
|tomcat.sessions.rejected|Oui|tomcat.sessions.rejected|Count|Total|Nombre de sessions Tomcat rejetées|Deployment, AppName, Pod|
|tomcat.threads.config.max|Oui|tomcat.threads.config.max|Count|Total|Nombre de threads maximal de la configuration Tomcat|Deployment, AppName, Pod|
|tomcat.threads.current|Oui|tomcat.threads.current|Count|Total|Nombre de threads actuel de Tomcat|Deployment, AppName, Pod|


## <a name="microsoftautomationautomationaccounts"></a>Microsoft.Automation/automationAccounts

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|TotalJob|Oui|Nombre total de travaux|Count|Total|Nombre total de travaux|Runbook, état|
|TotalUpdateDeploymentMachineRuns|Oui|Nombre total d’exécutions de déploiement de mises à jour de machines|Count|Total|Nombre total d'exécutions de déploiement de mises à jour logicielles de machines lors d'une exécution de déploiement de mises à jour logicielles|SoftwareUpdateConfigurationName, Status, TargetComputer, SoftwareUpdateConfigurationRunId|
|TotalUpdateDeploymentRuns|Oui|Nombre total d’exécutions de déploiement de mises à jour|Count|Total|Nombre total d’exécutions de déploiement de mises à jour logicielles|SoftwareUpdateConfigurationName, Status|


## <a name="microsoftbatchbatchaccounts"></a>Microsoft.Batch/batchAccounts

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|CoreCount|Non|Nombre de cœurs dédiés|Count|Total|Nombre total de cœurs dédiés dans le compte Batch|Aucune dimension|
|CreatingNodeCount|Non|Nombre de nœuds créés|Count|Total|Nombre de nœuds en cours de création|Aucune dimension|
|IdleNodeCount|Non|Nombre de nœuds inactifs|Count|Total|Le nombre de nœuds inactifs|Aucune dimension|
|JobDeleteCompleteEvent|Oui|Événements de fin de suppression de travail|Count|Total|Nombre total de travaux correctement supprimés.|jobId|
|JobDeleteStartEvent|Oui|Événements de démarrage de suppression de travail|Count|Total|Nombre total de travaux demandés pour être supprimés.|jobId|
|JobDisableCompleteEvent|Oui|Événements de fin de désactivation de travail|Count|Total|Nombre total de travaux correctement supprimés.|jobId|
|JobDisableStartEvent|Oui|Événements de démarrage de désactivation de travail|Count|Total|Nombre total de travaux demandés pour être désactivés.|jobId|
|JobStartEvent|Oui|Événements de démarrage de travail|Count|Total|Nombre total de travaux correctement démarrés.|jobId|
|JobTerminateCompleteEvent|Oui|Événements de fin de travail terminé|Count|Total|Nombre total de travaux correctement terminés.|jobId|
|JobTerminateCompleteEvent|Oui|Événements de démarrage de fin de travail|Count|Total|Nombre total de travaux demandés pour être terminés.|jobId|
|LeavingPoolNodeCount|Non|Nombre de nœuds quittant le pool|Count|Total|Le nombre de nœuds quittant le pool|Aucune dimension|
|LowPriorityCoreCount|Non|Nombre de cœurs à priorité basse|Count|Total|Nombre total de cœurs à priorité basse dans le compte Batch|Aucune dimension|
|OfflineNodeCount|Non|Nombre de nœuds hors ligne|Count|Total|Le nombre de nœuds hors ligne|Aucune dimension|
|PoolCreateEvent|Oui|Événements de création de pool|Count|Total|Nombre total de pools créés|poolId|
|PoolDeleteCompleteEvent|Oui|Événements de suppression de pool terminés|Count|Total|Nombre total de suppressions de pool terminées|poolId|
|PoolDeleteStartEvent|Oui|Événements de démarrage de suppression de pool|Count|Total|Nombre total de suppressions de pool ayant démarré|poolId|
|PoolResizeCompleteEvent|Oui|Événements de fin de redimensionnement de pool|Count|Total|Nombre total de redimensionnements de pool terminés|poolId|
|PoolResizeStartEvent|Oui|Événements de démarrage de redimensionnement de pool|Count|Total|Nombre total de redimensionnements de pool ayant démarré|poolId|
|PreemptedNodeCount|Non|Nombre de nœuds reportés|Count|Total|Nombre de nœuds reportés|Aucune dimension|
|RebootingNodeCount|Non|Nombre de nœuds en cours de redémarrage|Count|Total|Le nombre de nœuds en cours de redémarrage|Aucune dimension|
|ReimagingNodeCount|Non|Nombre de nœuds en cours de réimageage|Count|Total|Le nombre de nœuds en cours de réimageage|Aucune dimension|
|RunningNodeCount|Non|Nombre de nœuds en cours d’exécution|Count|Total|Le nombre de nœuds en cours d’exécution|Aucune dimension|
|StartingNodeCount|Non|Nombre de nœuds de départ|Count|Total|Nœuds de nœuds de départ|Aucune dimension|
|StartTaskFailedNodeCount|Non|Nombre de nœuds pour lesquels le démarrage d’une tâche a échoué|Count|Total|Le nombre de nœuds pour lesquels le démarrage d’une tâche a échoué|Aucune dimension|
|TaskCompleteEvent|Oui|Événements de fin de tâche|Count|Total|Le nombre total de tâches terminées|poolId, jobId|
|TaskFailEvent|Oui|Événements d’échec de tâches|Count|Total|Le nombre total de tâches terminées dans un état d’échec|poolId, jobId|
|TaskStartEvent|Oui|Événements de lancement de tâche|Count|Total|Nombre total de tâches ayant démarré|poolId, jobId|
|TotalLowPriorityNodeCount|Non|Nombre de nœuds à priorité basse|Count|Total|Nombre total de nœuds à priorité basse dans le compte Batch|Aucune dimension|
|TotalNodeCount|Non|Nombre de nœuds dédiés|Count|Total|Nombre total de nœuds dédiés dans le compte Batch|Aucune dimension|
|UnusableNodeCount|Non|Nombre de nœuds inutilisables|Count|Total|Le nombre de nœuds inutilisables|Aucune dimension|
|WaitingForStartTaskNodeCount|Non|Nombre de nœuds en attente de démarrage de tâche|Count|Total|Nombre de nœuds en attente de la fin d’une tâche de démarrage|Aucune dimension|


## <a name="microsoftbatchaiworkspaces"></a>Microsoft.BatchAI/workspaces

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|Cœurs actifs|Oui|Cœurs actifs|Count|Average|Nombre de cœurs actifs|Scenario, ClusterName|
|Nœuds actifs|Oui|Nœuds actifs|Count|Average|Le nombre de nœuds en cours d’exécution|Scenario, ClusterName|
|Cœurs inactifs|Oui|Cœurs inactifs|Count|Average|Nombre de cœurs inactifs|Scenario, ClusterName|
|Nœuds inactifs|Oui|Nœuds inactifs|Count|Average|Le nombre de nœuds inactifs|Scenario, ClusterName|
|Travail effectué|Oui|Travail effectué|Count|Total|Nombre de travaux effectués|Scenario, ClusterName, ResultType|
|Travail envoyé|Oui|Travail envoyé|Count|Total|Nombre de travaux envoyés|Scenario, ClusterName|
|Cœurs sortants|Oui|Cœurs sortants|Count|Average|Nombre de cœurs sortants|Scenario, ClusterName|
|Nœuds sortants|Oui|Nœuds sortants|Count|Average|Nombre de nœuds sortants|Scenario, ClusterName|
|Cœurs réquisitionnés|Oui|Cœurs réquisitionnés|Count|Average|Nombre de cœurs réquisitionnés|Scenario, ClusterName|
|Nœuds reportés|Oui|Nœuds reportés|Count|Average|Nombre de nœuds reportés|Scenario, ClusterName|
|Pourcentage d’utilisation du quota|Oui|Pourcentage d’utilisation du quota|Count|Average|Pourcentage de quota utilisé|Scenario, ClusterName, VmFamilyName, VmPriority|
|Nombre total de cœurs|Oui|Nombre total de cœurs|Count|Average|Nombre total de cœurs|Scenario, ClusterName|
|Nombre total de nœuds|Oui|Nombre total de nœuds|Count|Average|Nombre total de nœuds|Scenario, ClusterName|
|Cœurs inutilisables|Oui|Cœurs inutilisables|Count|Average|Nombre de cœurs inutilisables|Scenario, ClusterName|
|Nœuds inutilisables|Oui|Nœuds inutilisables|Count|Average|Le nombre de nœuds inutilisables|Scenario, ClusterName|


## <a name="microsoftblockchainblockchainmembers"></a>Microsoft.Blockchain/blockchainMembers

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|BroadcastProcessedCount|Oui|Nombre de diffusions traitées|Count|Average|Nombre de transactions traitées|Node, channel, type, status|
|ConnectionAccepted|Oui|Connexions acceptées|Count|Total|Connexions acceptées|Nœud|
|ConnectionActive|Oui|Connexions actives|Count|Average|Connexions actives|Nœud|
|ConnectionHandled|Oui|Connexions gérées|Count|Total|Connexions gérées|Nœud|
|ConsensusEtcdraftCommittedBlockNumber|Oui|Numéro de bloc validé Consensus Etcdraft|Count|Average|Numéro du dernier bloc validé|Node, channel|
|CpuUsagePercentageInDouble|Oui|Pourcentage d’utilisation du processeur|Pourcentage|Maximale|Pourcentage d’utilisation du processeur|Nœud|
|EndorserEndorsementFailures|Oui|Échecs d’approbation des approuveurs|Count|Average|Nombre d’approbations ayant échoué|Node, channel, chaincode, chaincodeerror|
|GossipLeaderElectionLeader|Oui|Leader choisi Gossip|Count|Total|Le pair est leader (1) ou suiveur (0)|Node, channel|
|GossipMembershipTotalPeersKnown|Oui|Nombre total de pairs connus appartenant à Gossip|Count|Average|Nombre total de pairs connus|Node, channel|
|GossipStateHeight|Oui|Hauteur de l’état Gossip|Count|Average|Hauteur actuelle du registre|Node, channel|
|IOReadBytes|Oui|E/S de lecture d’octets|Octets|Total|E/S de lecture d’octets|Nœud|
|IOWriteBytes|Oui|E/S d’écriture d’octets|Octets|Total|E/S d’écriture d’octets|Nœud|
|LedgerTransactionCount|Oui|Nombre de transactions du registre|Count|Average|Nombre de transactions traitées|Node, channel, transaction_type, chaincode, validation_code|
|MemoryLimit|Oui|Limite de mémoire|Octets|Average|Limite de mémoire|Nœud|
|MemoryUsage|Oui|Utilisation de la mémoire|Octets|Average|Utilisation de la mémoire|Nœud|
|MemoryUsagePercentageInDouble|Oui|Pourcentage d’utilisation de la mémoire|Pourcentage|Average|Pourcentage d’utilisation de la mémoire|Nœud|
|PendingTransactions|Oui|Transactions en attente|Count|Average|Transactions en attente|Nœud|
|ProcessedBlocks|Oui|Blocs traités|Count|Total|Blocs traités|Nœud|
|ProcessedTransactions|Oui|Transactions traitées|Count|Total|Transactions traitées|Nœud|
|QueuedTransactions|Oui|Transactions en file d’attente|Count|Average|Transactions en file d’attente|Nœud|
|RequestHandled|Oui|Requêtes traitées|Count|Total|Demandes traitées|Nœud|
|StorageUsage|Oui|Utilisation du stockage|Octets|Average|Utilisation du stockage|Nœud|


## <a name="microsoftcacheredis"></a>Microsoft.Cache/redis

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|cachehits|Oui|Présences dans le cache|Count|Total||ShardId|
|cachehits0|Oui|Présences dans le cache (Shard 0)|Count|Total||Aucune dimension|
|cachehits1|Oui|Présences dans le cache (Shard 1)|Count|Total||Aucune dimension|
|cachehits2|Oui|Présences dans le cache (Shard 2)|Count|Total||Aucune dimension|
|cachehits3|Oui|Présences dans le cache (Shard 3)|Count|Total||Aucune dimension|
|cachehits4|Oui|Présences dans le cache (Shard 4)|Count|Total||Aucune dimension|
|cachehits5|Oui|Présences dans le cache (Shard 5)|Count|Total||Aucune dimension|
|cachehits6|Oui|Présences dans le cache (Shard 6)|Count|Total||Aucune dimension|
|cachehits7|Oui|Présences dans le cache (Shard 7)|Count|Total||Aucune dimension|
|cachehits8|Oui|Présences dans le cache (Shard 8)|Count|Total||Aucune dimension|
|cachehits9|Oui|Présences dans le cache (Shard 9)|Count|Total||Aucune dimension|
|cacheLatency|Oui|Latence de cache en microsecondes (préversion)|Count|Average||ShardId|
|cachemisses|Oui|Absences dans le cache|Count|Total||ShardId|
|cachemisses0|Oui|Absences dans le cache (Shard 0)|Count|Total||Aucune dimension|
|cachemisses1|Oui|Absences dans le cache (Shard 1)|Count|Total||Aucune dimension|
|cachemisses2|Oui|Absences dans le cache (Shard 2)|Count|Total||Aucune dimension|
|cachemisses3|Oui|Absences dans le cache (Shard 3)|Count|Total||Aucune dimension|
|cachemisses4|Oui|Absences dans le cache (Shard 4)|Count|Total||Aucune dimension|
|cachemisses5|Oui|Absences dans le cache (Shard 5)|Count|Total||Aucune dimension|
|cachemisses6|Oui|Absences dans le cache (Shard 6)|Count|Total||Aucune dimension|
|cachemisses7|Oui|Absences dans le cache (Shard 7)|Count|Total||Aucune dimension|
|cachemisses8|Oui|Absences dans le cache (Shard 8)|Count|Total||Aucune dimension|
|cachemisses9|Oui|Absences dans le cache (Shard 9)|Count|Total||Aucune dimension|
|cachemissrate|Oui|Taux d’échec d’accès au cache|Pourcentage|cachemissrate||ShardId|
|cacheRead|Oui|Lecture du cache|BytesPerSecond|Maximale||ShardId|
|cacheRead0|Oui|Cache de lecture (Shard 0)|BytesPerSecond|Maximale||Aucune dimension|
|cacheRead1|Oui|Lecture du cache (Shard 1)|BytesPerSecond|Maximale||Aucune dimension|
|cacheRead2|Oui|Lecture du cache (Shard 2)|BytesPerSecond|Maximale||Aucune dimension|
|cacheRead3|Oui|Lecture du cache (Shard 3)|BytesPerSecond|Maximale||Aucune dimension|
|cacheRead4|Oui|Lecture du cache (Shard 4)|BytesPerSecond|Maximale||Aucune dimension|
|cacheRead5|Oui|Lecture du cache (Shard 5)|BytesPerSecond|Maximale||Aucune dimension|
|cacheRead6|Oui|Lecture du cache (Shard 6)|BytesPerSecond|Maximale||Aucune dimension|
|cacheRead7|Oui|Lecture du cache (Shard 7)|BytesPerSecond|Maximale||Aucune dimension|
|cacheRead8|Oui|Lecture du cache (Shard 8)|BytesPerSecond|Maximale||Aucune dimension|
|cacheRead9|Oui|Cache de lecture (Shard 9)|BytesPerSecond|Maximale||Aucune dimension|
|cacheWrite|Oui|Cache d’écriture|BytesPerSecond|Maximale||ShardId|
|cacheWrite0|Oui|Cache d’écriture (Shard 0)|BytesPerSecond|Maximale||Aucune dimension|
|cacheWrite1|Oui|Cache d’écriture (Shard 1)|BytesPerSecond|Maximale||Aucune dimension|
|cacheWrite2|Oui|Cache d’écriture (Shard 2)|BytesPerSecond|Maximale||Aucune dimension|
|cacheWrite3|Oui|Cache d’écriture (Shard 3)|BytesPerSecond|Maximale||Aucune dimension|
|cacheWrite4|Oui|Cache d’écriture (Shard 4)|BytesPerSecond|Maximale||Aucune dimension|
|cacheWrite5|Oui|Cache d’écriture (Shard 5)|BytesPerSecond|Maximale||Aucune dimension|
|cacheWrite6|Oui|Cache d’écriture (Shard 6)|BytesPerSecond|Maximale||Aucune dimension|
|cacheWrite7|Oui|Cache d’écriture (Shard 7)|BytesPerSecond|Maximale||Aucune dimension|
|cacheWrite8|Oui|Cache d’écriture (Shard 8)|BytesPerSecond|Maximale||Aucune dimension|
|cacheWrite9|Oui|Cache d’écriture (Shard 9)|BytesPerSecond|Maximale||Aucune dimension|
|connectedclients|Oui|Clients connectés|Count|Maximale||ShardId|
|connectedclients0|Oui|Clients connectés (Shard 0)|Count|Maximale||Aucune dimension|
|connectedclients1|Oui|Clients connectés (Shard 1)|Count|Maximale||Aucune dimension|
|connectedclients2|Oui|Clients connectés (Shard 2)|Count|Maximale||Aucune dimension|
|connectedclients3|Oui|Clients connectés (Shard 3)|Count|Maximale||Aucune dimension|
|connectedclients4|Oui|Clients connectés (Shard 4)|Count|Maximale||Aucune dimension|
|connectedclients5|Oui|Clients connectés (Shard 5)|Count|Maximale||Aucune dimension|
|connectedclients6|Oui|Clients connectés (Shard 6)|Count|Maximale||Aucune dimension|
|connectedclients7|Oui|Clients connectés (Shard 7)|Count|Maximale||Aucune dimension|
|connectedclients8|Oui|Clients connectés (Shard 8)|Count|Maximale||Aucune dimension|
|connectedclients9|Oui|Clients connectés (Shard 9)|Count|Maximale||Aucune dimension|
|erreurs|Oui|Erreurs|Count|Maximale||ShardId, ErrorType|
|evictedkeys|Oui|Clés exclues|Count|Total||ShardId|
|evictedkeys0|Oui|Clés exclues (Shard 0)|Count|Total||Aucune dimension|
|evictedkeys1|Oui|Clés exclues (Shard 1)|Count|Total||Aucune dimension|
|evictedkeys2|Oui|Clés exclues (Shard 2)|Count|Total||Aucune dimension|
|evictedkeys3|Oui|Clés exclues (Shard 3)|Count|Total||Aucune dimension|
|evictedkeys4|Oui|Clés exclues (Shard 4)|Count|Total||Aucune dimension|
|evictedkeys5|Oui|Clés exclues (Shard 5)|Count|Total||Aucune dimension|
|evictedkeys6|Oui|Clés exclues (Shard 6)|Count|Total||Aucune dimension|
|evictedkeys7|Oui|Clés exclues (Shard 7)|Count|Total||Aucune dimension|
|evictedkeys8|Oui|Clés exclues (Shard 8)|Count|Total||Aucune dimension|
|evictedkeys9|Oui|Clés exclues (Shard 9)|Count|Total||Aucune dimension|
|expiredkeys|Oui|Clés expirées|Count|Total||ShardId|
|expiredkeys0|Oui|Clés expirées (Shard 0)|Count|Total||Aucune dimension|
|expiredkeys1|Oui|Clés expirées (Shard 1)|Count|Total||Aucune dimension|
|expiredkeys2|Oui|Clés expirées (Shard 2)|Count|Total||Aucune dimension|
|expiredkeys3|Oui|Clés expirées (Shard 3)|Count|Total||Aucune dimension|
|expiredkeys4|Oui|Clés expirées (Shard 4)|Count|Total||Aucune dimension|
|expiredkeys5|Oui|Clés expirées (Shard 5)|Count|Total||Aucune dimension|
|expiredkeys6|Oui|Clés expirées (Shard 6)|Count|Total||Aucune dimension|
|expiredkeys7|Oui|Clés expirées (Shard 7)|Count|Total||Aucune dimension|
|expiredkeys8|Oui|Clés expirées (Shard 8)|Count|Total||Aucune dimension|
|expiredkeys9|Oui|Clés expirées (Shard 9)|Count|Total||Aucune dimension|
|getcommands|Oui|Gets|Count|Total||ShardId|
|getcommands0|Oui|Gets (Shard 0)|Count|Total||Aucune dimension|
|getcommands1|Oui|Gets (Shard 1)|Count|Total||Aucune dimension|
|getcommands2|Oui|Gets (Shard 2)|Count|Total||Aucune dimension|
|getcommands3|Oui|Gets (Shard 3)|Count|Total||Aucune dimension|
|getcommands4|Oui|Gets (Shard 4)|Count|Total||Aucune dimension|
|getcommands5|Oui|Gets (Shard 5)|Count|Total||Aucune dimension|
|getcommands6|Oui|Gets (Shard 6)|Count|Total||Aucune dimension|
|getcommands7|Oui|Gets (Shard 7)|Count|Total||Aucune dimension|
|getcommands8|Oui|Gets (Shard 8)|Count|Total||Aucune dimension|
|getcommands9|Oui|Gets (Shard 9)|Count|Total||Aucune dimension|
|operationsPerSecond|Oui|Opérations par seconde|Count|Maximale||ShardId|
|operationsPerSecond0|Oui|Opérations par seconde (Partition 0)|Count|Maximale||Aucune dimension|
|operationsPerSecond1|Oui|Opérations par seconde (Partition 1)|Count|Maximale||Aucune dimension|
|operationsPerSecond2|Oui|Opérations par seconde (Partition 2)|Count|Maximale||Aucune dimension|
|operationsPerSecond3|Oui|Opérations par seconde (Partition 3)|Count|Maximale||Aucune dimension|
|operationsPerSecond4|Oui|Opérations par seconde (Partition 4)|Count|Maximale||Aucune dimension|
|operationsPerSecond5|Oui|Opérations par seconde (Partition 5)|Count|Maximale||Aucune dimension|
|operationsPerSecond6|Oui|Opérations par seconde (Partition 6)|Count|Maximale||Aucune dimension|
|operationsPerSecond7|Oui|Opérations par seconde (Partition 7)|Count|Maximale||Aucune dimension|
|operationsPerSecond8|Oui|Opérations par seconde (Partition 8)|Count|Maximale||Aucune dimension|
|operationsPerSecond9|Oui|Opérations par seconde (Partition 9)|Count|Maximale||Aucune dimension|
|percentProcessorTime|Oui|UC|Pourcentage|Maximale||ShardId|
|percentProcessorTime0|Oui|UC (Shard 0)|Pourcentage|Maximale||Aucune dimension|
|percentProcessorTime1|Oui|UC (Shard 1)|Pourcentage|Maximale||Aucune dimension|
|percentProcessorTime2|Oui|UC (Shard 2)|Pourcentage|Maximale||Aucune dimension|
|percentProcessorTime3|Oui|UC (Shard 3)|Pourcentage|Maximale||Aucune dimension|
|percentProcessorTime4|Oui|UC (Shard 4)|Pourcentage|Maximale||Aucune dimension|
|percentProcessorTime5|Oui|UC (Shard 5)|Pourcentage|Maximale||Aucune dimension|
|percentProcessorTime6|Oui|UC (Shard 6)|Pourcentage|Maximale||Aucune dimension|
|percentProcessorTime7|Oui|UC (Shard 7)|Pourcentage|Maximale||Aucune dimension|
|percentProcessorTime8|Oui|UC (Shard 8)|Pourcentage|Maximale||Aucune dimension|
|percentProcessorTime9|Oui|UC (Shard 9)|Pourcentage|Maximale||Aucune dimension|
|serverLoad|Oui|Charge du serveur|Pourcentage|Maximale||ShardId|
|serverLoad0|Oui|Charge du serveur (Shard 0)|Pourcentage|Maximale||Aucune dimension|
|serverLoad1|Oui|Charge du serveur (Shard 1)|Pourcentage|Maximale||Aucune dimension|
|serverLoad2|Oui|Charge du serveur (Shard 2)|Pourcentage|Maximale||Aucune dimension|
|serverLoad3|Oui|Charge du serveur (Shard 3)|Pourcentage|Maximale||Aucune dimension|
|serverLoad4|Oui|Charge du serveur (Shard 4)|Pourcentage|Maximale||Aucune dimension|
|serverLoad5|Oui|Charge du serveur (Shard 5)|Pourcentage|Maximale||Aucune dimension|
|serverLoad6|Oui|Charge du serveur (Shard 6)|Pourcentage|Maximale||Aucune dimension|
|serverLoad7|Oui|Charge du serveur (Shard 7)|Pourcentage|Maximale||Aucune dimension|
|serverLoad8|Oui|Charge du serveur (Shard 8)|Pourcentage|Maximale||Aucune dimension|
|serverLoad9|Oui|Charge du serveur (Shard 9)|Pourcentage|Maximale||Aucune dimension|
|setcommands|Oui|Jeux|Count|Total||ShardId|
|setcommands0|Oui|Sets (Shard 0)|Count|Total||Aucune dimension|
|setcommands1|Oui|Sets (Shard 1)|Count|Total||Aucune dimension|
|setcommands2|Oui|Sets (Shard 2)|Count|Total||Aucune dimension|
|setcommands3|Oui|Sets (Shard 3)|Count|Total||Aucune dimension|
|setcommands4|Oui|Sets (Shard 4)|Count|Total||Aucune dimension|
|setcommands5|Oui|Sets (Shard 5)|Count|Total||Aucune dimension|
|setcommands6|Oui|Sets (Shard 6)|Count|Total||Aucune dimension|
|setcommands7|Oui|Sets (Shard 7)|Count|Total||Aucune dimension|
|setcommands8|Oui|Sets (Shard 8)|Count|Total||Aucune dimension|
|setcommands9|Oui|Sets (Shard 9)|Count|Total||Aucune dimension|
|totalcommandsprocessed|Oui|Total des opérations|Count|Total||ShardId|
|totalcommandsprocessed0|Oui|Total des opérations (Shard 0)|Count|Total||Aucune dimension|
|totalcommandsprocessed1|Oui|Total des opérations (Shard 1)|Count|Total||Aucune dimension|
|totalcommandsprocessed2|Oui|Total des opérations (Shard 2)|Count|Total||Aucune dimension|
|totalcommandsprocessed3|Oui|Total des opérations (Shard 3)|Count|Total||Aucune dimension|
|totalcommandsprocessed4|Oui|Total des opérations (Shard 4)|Count|Total||Aucune dimension|
|totalcommandsprocessed5|Oui|Total des opérations (Shard 5)|Count|Total||Aucune dimension|
|totalcommandsprocessed6|Oui|Total des opérations (Shard 6)|Count|Total||Aucune dimension|
|totalcommandsprocessed7|Oui|Total des opérations (Shard 7)|Count|Total||Aucune dimension|
|totalcommandsprocessed8|Oui|Total des opérations (Shard 8)|Count|Total||Aucune dimension|
|totalcommandsprocessed9|Oui|Total des opérations (Shard 9)|Count|Total||Aucune dimension|
|totalkeys|Oui|Nombre total de clés|Count|Maximale||ShardId|
|totalkeys0|Oui|Nombre total de clés (Shard 0)|Count|Maximale||Aucune dimension|
|totalkeys1|Oui|Nombre total de clés (Shard 1)|Count|Maximale||Aucune dimension|
|totalkeys2|Oui|Nombre total de clés (Shard 2)|Count|Maximale||Aucune dimension|
|totalkeys3|Oui|Nombre total de clés (Shard 3)|Count|Maximale||Aucune dimension|
|totalkeys4|Oui|Nombre total de clés (Shard 4)|Count|Maximale||Aucune dimension|
|totalkeys5|Oui|Nombre total de clés (Shard 5)|Count|Maximale||Aucune dimension|
|totalkeys6|Oui|Nombre total de clés (Shard 6)|Count|Maximale||Aucune dimension|
|totalkeys7|Oui|Nombre total de clés (Shard 7)|Count|Maximale||Aucune dimension|
|totalkeys8|Oui|Nombre total de clés (Shard 8)|Count|Maximale||Aucune dimension|
|totalkeys9|Oui|Nombre total de clés (Shard 9)|Count|Maximale||Aucune dimension|
|usedmemory|Oui|Mémoire utilisée|Octets|Maximale||ShardId|
|usedmemory0|Oui|Mémoire utilisée (Shard 0)|Octets|Maximale||Aucune dimension|
|usedmemory1|Oui|Mémoire utilisée (Shard 1)|Octets|Maximale||Aucune dimension|
|usedmemory2|Oui|Mémoire utilisée (Shard 2)|Octets|Maximale||Aucune dimension|
|usedmemory3|Oui|Mémoire utilisée (Shard 3)|Octets|Maximale||Aucune dimension|
|usedmemory4|Oui|Mémoire utilisée (Shard 4)|Octets|Maximale||Aucune dimension|
|usedmemory5|Oui|Mémoire utilisée (Shard 5)|Octets|Maximale||Aucune dimension|
|usedmemory6|Oui|Mémoire utilisée (Shard 6)|Octets|Maximale||Aucune dimension|
|usedmemory7|Oui|Mémoire utilisée (Shard 7)|Octets|Maximale||Aucune dimension|
|usedmemory8|Oui|Mémoire utilisée (Shard 8)|Octets|Maximale||Aucune dimension|
|usedmemory9|Oui|Mémoire utilisée (Shard 9)|Octets|Maximale||Aucune dimension|
|usedmemorypercentage|Oui|Pourcentage de mémoire utilisé|Pourcentage|Maximale||ShardId|
|usedmemoryRss|Oui|Taille de la mémoire résidente utilisée|Octets|Maximale||ShardId|
|usedmemoryRss0|Oui|Mémoire résidente utilisée (Shard 0)|Octets|Maximale||Aucune dimension|
|usedmemoryRss1|Oui|Mémoire résidente utilisée (Shard 1)|Octets|Maximale||Aucune dimension|
|usedmemoryRss2|Oui|Mémoire résidente utilisée (Shard 2)|Octets|Maximale||Aucune dimension|
|usedmemoryRss3|Oui|Mémoire résidente utilisée (Shard 3)|Octets|Maximale||Aucune dimension|
|usedmemoryRss4|Oui|Mémoire résidente utilisée (Shard 4)|Octets|Maximale||Aucune dimension|
|usedmemoryRss5|Oui|Mémoire résidente utilisée (Shard 5)|Octets|Maximale||Aucune dimension|
|usedmemoryRss6|Oui|Mémoire résidente utilisée (Shard 6)|Octets|Maximale||Aucune dimension|
|usedmemoryRss7|Oui|Mémoire résidente utilisée (Shard 7)|Octets|Maximale||Aucune dimension|
|usedmemoryRss8|Oui|Mémoire résidente utilisée (Shard 8)|Octets|Maximale||Aucune dimension|
|usedmemoryRss9|Oui|Mémoire résidente utilisée (Shard 9)|Octets|Maximale||Aucune dimension|


## <a name="microsoftcdncdnwebapplicationfirewallpolicies"></a>Microsoft.Cdn/cdnwebapplicationfirewallpolicies

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|WebApplicationFirewallRequestCount|Oui|Nombre de requêtes du pare-feu d’applications web|Count|Total|Nombre de requêtes clients traitées par le pare-feu d’applications web|PolicyName, RuleName, Action|


## <a name="microsoftclassiccomputedomainnamesslotsroles"></a>Microsoft.ClassicCompute/domainNames/slots/roles

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|Disk Read Bytes/Sec|Non|Lecture du disque|BytesPerSecond|Average|Moyenne d’octets lus à partir du disque pendant la période d’analyse.|RoleInstanceId|
|Disk Read Operations/Sec|Oui|Disk Read Operations/Sec|CountPerSecond|Average|E/S de lecture disque par seconde.|RoleInstanceId|
|Disk Write Bytes/Sec|Non|Écriture sur le disque|BytesPerSecond|Average|Moyenne d’octets écrits sur le disque pendant la période d’analyse.|RoleInstanceId|
|Disk Write Operations/Sec|Oui|Disk Write Operations/Sec|CountPerSecond|Average|E/S d’écriture sur disque par seconde.|RoleInstanceId|
|Network In|Oui|Network In|Octets|Total|Nombre d’octets reçus sur toutes les interfaces réseau par les machines virtuelles (trafic entrant).|RoleInstanceId|
|Network Out|Oui|Network Out|Octets|Total|Nombre d’octets envoyés sur toutes les interfaces réseau par les machines virtuelles (trafic sortant).|RoleInstanceId|
|Percentage CPU|Oui|Percentage CPU|Pourcentage|Average|Pourcentage d’unités de calcul affectées actuellement utilisées par des machines virtuelles.|RoleInstanceId|


## <a name="microsoftclassiccomputevirtualmachines"></a>Microsoft.ClassicCompute/virtualMachines

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|Disk Read Bytes/Sec|Non|Lecture du disque|BytesPerSecond|Average|Moyenne d’octets lus à partir du disque pendant la période d’analyse.|Aucune dimension|
|Disk Read Operations/Sec|Oui|Disk Read Operations/Sec|CountPerSecond|Average|E/S de lecture disque par seconde.|Aucune dimension|
|Disk Write Bytes/Sec|Non|Écriture sur le disque|BytesPerSecond|Average|Moyenne d’octets écrits sur le disque pendant la période d’analyse.|Aucune dimension|
|Disk Write Operations/Sec|Oui|Disk Write Operations/Sec|CountPerSecond|Average|E/S d’écriture sur disque par seconde.|Aucune dimension|
|Network In|Oui|Network In|Octets|Total|Nombre d’octets reçus sur toutes les interfaces réseau par les machines virtuelles (trafic entrant).|Aucune dimension|
|Network Out|Oui|Network Out|Octets|Total|Nombre d’octets envoyés sur toutes les interfaces réseau par les machines virtuelles (trafic sortant).|Aucune dimension|
|Percentage CPU|Oui|Percentage CPU|Pourcentage|Average|Pourcentage d’unités de calcul affectées actuellement utilisées par des machines virtuelles.|Aucune dimension|


## <a name="microsoftclassicstoragestorageaccounts"></a>Microsoft.ClassicStorage/storageAccounts

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|Disponibilité|Oui|Disponibilité|Pourcentage|Average|Pourcentage de disponibilité pour le service de stockage ou l’opération API spécifiée. La disponibilité est calculée en prenant la valeur TotalBillableRequests puis en la divisant par le nombre de requêtes applicables, y compris celles qui ont généré des erreurs inattendues. Toutes erreurs inattendues réduisent la disponibilité du service de stockage ou de l’opération API spécifiée.|GeoType, ApiName, Authentication|
|Sortie|Oui|Sortie|Octets|Total|Quantité de données de sortie, en octets. Ce nombre inclut les sorties d’un client externe dans Stockage Microsoft Azure ainsi que les sorties dans Azure. Par conséquent, ce nombre ne reflète pas les sorties facturables.|GeoType, ApiName, Authentication|
|Entrée|Oui|Entrée|Octets|Total|Quantité de données d’entrée, en octets. Ce nombre inclut les entrées d’un client externe dans Stockage Microsoft Azure ainsi que les entrées dans Azure.|GeoType, ApiName, Authentication|
|SuccessE2ELatency|Oui|Latence E2E de réussite|Millisecondes|Average|Latence de bout en bout des requêtes réussies envoyées à un service de stockage ou à l’opération API spécifiée, en millisecondes. Cette valeur inclut le temps de traitement requis au sein de Stockage Microsoft Azure pour lire la requête, envoyer la réponse et recevoir un accusé de réception de la réponse.|GeoType, ApiName, Authentication|
|SuccessServerLatency|Oui|Latence du serveur avec requête réussie|Millisecondes|Average|Latence utilisée par Stockage Microsoft Azure pour traiter une requête réussie, en millisecondes. Cette valeur n’inclut pas la latence réseau spécifiée dans SuccessE2ELatency.|GeoType, ApiName, Authentication|
|Transactions|Oui|Transactions|Count|Total|Nombre de requêtes envoyées à un service de stockage ou à l’opération API spécifiée. Ce nombre inclut les requêtes réussies et celles ayant échoué, ainsi que les requêtes qui ont généré des erreurs. Utilisez la dimension ResponseType pour connaître le nombre des différents types de réponses.|ResponseType, GeoType, ApiName, Authentication|
|UsedCapacity|Non|Capacité utilisée|Octets|Average|Capacité de compte utilisée|Aucune dimension|


## <a name="microsoftclassicstoragestorageaccountsblobservices"></a>Microsoft.ClassicStorage/storageAccounts/blobServices

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|Disponibilité|Oui|Disponibilité|Pourcentage|Average|Pourcentage de disponibilité pour le service de stockage ou l’opération API spécifiée. La disponibilité est calculée en prenant la valeur TotalBillableRequests puis en la divisant par le nombre de requêtes applicables, y compris celles qui ont généré des erreurs inattendues. Toutes erreurs inattendues réduisent la disponibilité du service de stockage ou de l’opération API spécifiée.|GeoType, ApiName, Authentication|
|BlobCapacity|Non|Capacité d’objet blob|Octets|Average|Quantité de stockage utilisée par le service BLOB du compte de stockage, en octets.|BlobType, Tier|
|BlobCount|Non|Nombre d’objets blob|Count|Average|Nombre d’objets blob dans le service BLOB du compte de stockage.|BlobType, Tier|
|ContainerCount|Oui|Nombre de conteneurs d’objets blob|Count|Average|Nombre de conteneurs d’objets blob dans le service BLOB du compte de stockage.|Aucune dimension|
|Sortie|Oui|Sortie|Octets|Total|Quantité de données de sortie, en octets. Ce nombre inclut les sorties d’un client externe dans Stockage Microsoft Azure ainsi que les sorties dans Azure. Par conséquent, ce nombre ne reflète pas les sorties facturables.|GeoType, ApiName, Authentication|
|IndexCapacity|Non|Capacité d'index|Octets|Average|Stockage utilisé par l'index (hiérarchique) d'ADLS Gen2 en octets.|Aucune dimension|
|Entrée|Oui|Entrée|Octets|Total|Quantité de données d’entrée, en octets. Ce nombre inclut les entrées d’un client externe dans Stockage Microsoft Azure ainsi que les entrées dans Azure.|GeoType, ApiName, Authentication|
|SuccessE2ELatency|Oui|Latence E2E de réussite|Millisecondes|Average|Latence de bout en bout des requêtes réussies envoyées à un service de stockage ou à l’opération API spécifiée, en millisecondes. Cette valeur inclut le temps de traitement requis au sein de Stockage Microsoft Azure pour lire la requête, envoyer la réponse et recevoir un accusé de réception de la réponse.|GeoType, ApiName, Authentication|
|SuccessServerLatency|Oui|Latence du serveur avec requête réussie|Millisecondes|Average|Latence utilisée par Stockage Microsoft Azure pour traiter une requête réussie, en millisecondes. Cette valeur n’inclut pas la latence réseau spécifiée dans SuccessE2ELatency.|GeoType, ApiName, Authentication|
|Transactions|Oui|Transactions|Count|Total|Nombre de requêtes envoyées à un service de stockage ou à l’opération API spécifiée. Ce nombre inclut les requêtes réussies et celles ayant échoué, ainsi que les requêtes qui ont généré des erreurs. Utilisez la dimension ResponseType pour connaître le nombre des différents types de réponses.|ResponseType, GeoType, ApiName, Authentication|


## <a name="microsoftclassicstoragestorageaccountsfileservices"></a>Microsoft.ClassicStorage/storageAccounts/fileServices

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|Disponibilité|Oui|Disponibilité|Pourcentage|Average|Pourcentage de disponibilité pour le service de stockage ou l’opération API spécifiée. La disponibilité est calculée en prenant la valeur TotalBillableRequests puis en la divisant par le nombre de requêtes applicables, y compris celles qui ont généré des erreurs inattendues. Toutes erreurs inattendues réduisent la disponibilité du service de stockage ou de l’opération API spécifiée.|GeoType, ApiName, Authentication, FileShare|
|Sortie|Oui|Sortie|Octets|Total|Quantité de données de sortie, en octets. Ce nombre inclut les sorties d’un client externe dans Stockage Microsoft Azure ainsi que les sorties dans Azure. Par conséquent, ce nombre ne reflète pas les sorties facturables.|GeoType, ApiName, Authentication, FileShare|
|FileCapacity|Non|Capacité de fichiers|Octets|Average|Quantité de stockage utilisée par le service de Fichier du compte de stockage, en octets.|FileShare|
|FileCount|Non|Nombre de fichiers|Count|Average|Nombre de fichiers dans le service de Fichier du compte de stockage.|FileShare|
|FileShareCount|Non|Nombre de partages de fichiers|Count|Average|Nombre de partage de fichiers dans le service de Fichier du compte de stockage.|Aucune dimension|
|FileShareQuota|Non|Taille du quota de partage de fichiers|Octets|Average|Limite supérieure de la quantité de stockage pouvant être utilisée par le service Azure Files, en octets.|FileShare|
|FileShareSnapshotCount|Non|Nombre d’instantanés de partage de fichiers|Count|Average|Nombre d’instantanés présents sur le partage dans le service Files du compte de stockage.|FileShare|
|FileShareSnapshotSize|Non|Taille d’instantané de partage de fichiers|Octets|Average|Quantité de stockage utilisée par les instantanés dans le service Fichier du compte de stockage, en octets.|FileShare|
|Entrée|Oui|Entrée|Octets|Total|Quantité de données d’entrée, en octets. Ce nombre inclut les entrées d’un client externe dans Stockage Microsoft Azure ainsi que les entrées dans Azure.|GeoType, ApiName, Authentication, FileShare|
|SuccessE2ELatency|Oui|Latence E2E de réussite|Millisecondes|Average|Latence de bout en bout des requêtes réussies envoyées à un service de stockage ou à l’opération API spécifiée, en millisecondes. Cette valeur inclut le temps de traitement requis au sein de Stockage Microsoft Azure pour lire la requête, envoyer la réponse et recevoir un accusé de réception de la réponse.|GeoType, ApiName, Authentication, FileShare|
|SuccessServerLatency|Oui|Latence du serveur avec requête réussie|Millisecondes|Average|Latence utilisée par Stockage Microsoft Azure pour traiter une requête réussie, en millisecondes. Cette valeur n’inclut pas la latence réseau spécifiée dans SuccessE2ELatency.|GeoType, ApiName, Authentication, FileShare|
|Transactions|Oui|Transactions|Count|Total|Nombre de requêtes envoyées à un service de stockage ou à l’opération API spécifiée. Ce nombre inclut les requêtes réussies et celles ayant échoué, ainsi que les requêtes qui ont généré des erreurs. Utilisez la dimension ResponseType pour connaître le nombre des différents types de réponses.|ResponseType, GeoType, ApiName, Authentication, FileShare|


## <a name="microsoftclassicstoragestorageaccountsqueueservices"></a>Microsoft.ClassicStorage/storageAccounts/queueServices

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|Disponibilité|Oui|Disponibilité|Pourcentage|Average|Pourcentage de disponibilité pour le service de stockage ou l’opération API spécifiée. La disponibilité est calculée en prenant la valeur TotalBillableRequests puis en la divisant par le nombre de requêtes applicables, y compris celles qui ont généré des erreurs inattendues. Toutes erreurs inattendues réduisent la disponibilité du service de stockage ou de l’opération API spécifiée.|GeoType, ApiName, Authentication|
|Sortie|Oui|Sortie|Octets|Total|Quantité de données de sortie, en octets. Ce nombre inclut les sorties d’un client externe dans Stockage Microsoft Azure ainsi que les sorties dans Azure. Par conséquent, ce nombre ne reflète pas les sorties facturables.|GeoType, ApiName, Authentication|
|Entrée|Oui|Entrée|Octets|Total|Quantité de données d’entrée, en octets. Ce nombre inclut les entrées d’un client externe dans Stockage Microsoft Azure ainsi que les entrées dans Azure.|GeoType, ApiName, Authentication|
|QueueCapacity|Oui|Capacité de la file d’attente|Octets|Average|Quantité de stockage utilisée par le service de File d’attente du compte de stockage, en octets.|Aucune dimension|
|QueueCount|Oui|Nombre de files d’attente|Count|Average|Nombre de files d’attente dans le service de File d’attente du compte de stockage.|Aucune dimension|
|QueueMessageCount|Oui|Nombre de messages dans la file d’attente|Count|Average|Nombre approximatif de messages en file d’attente dans le service de File d’attente du compte de stockage.|Aucune dimension|
|SuccessE2ELatency|Oui|Latence E2E de réussite|Millisecondes|Average|Latence de bout en bout des requêtes réussies envoyées à un service de stockage ou à l’opération API spécifiée, en millisecondes. Cette valeur inclut le temps de traitement requis au sein de Stockage Microsoft Azure pour lire la requête, envoyer la réponse et recevoir un accusé de réception de la réponse.|GeoType, ApiName, Authentication|
|SuccessServerLatency|Oui|Latence du serveur avec requête réussie|Millisecondes|Average|Latence utilisée par Stockage Microsoft Azure pour traiter une requête réussie, en millisecondes. Cette valeur n’inclut pas la latence réseau spécifiée dans SuccessE2ELatency.|GeoType, ApiName, Authentication|
|Transactions|Oui|Transactions|Count|Total|Nombre de requêtes envoyées à un service de stockage ou à l’opération API spécifiée. Ce nombre inclut les requêtes réussies et celles ayant échoué, ainsi que les requêtes qui ont généré des erreurs. Utilisez la dimension ResponseType pour connaître le nombre des différents types de réponses.|ResponseType, GeoType, ApiName, Authentication|


## <a name="microsoftclassicstoragestorageaccountstableservices"></a>Microsoft.ClassicStorage/storageAccounts/tableServices

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|Disponibilité|Oui|Disponibilité|Pourcentage|Average|Pourcentage de disponibilité pour le service de stockage ou l’opération API spécifiée. La disponibilité est calculée en prenant la valeur TotalBillableRequests puis en la divisant par le nombre de requêtes applicables, y compris celles qui ont généré des erreurs inattendues. Toutes erreurs inattendues réduisent la disponibilité du service de stockage ou de l’opération API spécifiée.|GeoType, ApiName, Authentication|
|Sortie|Oui|Sortie|Octets|Total|Quantité de données de sortie, en octets. Ce nombre inclut les sorties d’un client externe dans Stockage Microsoft Azure ainsi que les sorties dans Azure. Par conséquent, ce nombre ne reflète pas les sorties facturables.|GeoType, ApiName, Authentication|
|Entrée|Oui|Entrée|Octets|Total|Quantité de données d’entrée, en octets. Ce nombre inclut les entrées d’un client externe dans Stockage Microsoft Azure ainsi que les entrées dans Azure.|GeoType, ApiName, Authentication|
|SuccessE2ELatency|Oui|Latence E2E de réussite|Millisecondes|Average|Latence de bout en bout des requêtes réussies envoyées à un service de stockage ou à l’opération API spécifiée, en millisecondes. Cette valeur inclut le temps de traitement requis au sein de Stockage Microsoft Azure pour lire la requête, envoyer la réponse et recevoir un accusé de réception de la réponse.|GeoType, ApiName, Authentication|
|SuccessServerLatency|Oui|Latence du serveur avec requête réussie|Millisecondes|Average|Latence utilisée par Stockage Microsoft Azure pour traiter une requête réussie, en millisecondes. Cette valeur n’inclut pas la latence réseau spécifiée dans SuccessE2ELatency.|GeoType, ApiName, Authentication|
|TableCapacity|Oui|Capacité de la table|Octets|Average|Quantité de stockage utilisée par le service de Table du compte de stockage, en octets.|Aucune dimension|
|TableCount|Oui|Nombre de tables|Count|Average|Nombre de tables dans le service de Table du compte de stockage.|Aucune dimension|
|TableEntityCount|Oui|Nombre d’entités de table|Count|Average|Nombre d’entités de table dans le service de Table du compte de stockage.|Aucune dimension|
|Transactions|Oui|Transactions|Count|Total|Nombre de requêtes envoyées à un service de stockage ou à l’opération API spécifiée. Ce nombre inclut les requêtes réussies et celles ayant échoué, ainsi que les requêtes qui ont généré des erreurs. Utilisez la dimension ResponseType pour connaître le nombre des différents types de réponses.|ResponseType, GeoType, ApiName, Authentication|


## <a name="microsoftcognitiveservicesaccounts"></a>Microsoft.CognitiveServices/accounts

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|BlockedCalls|Oui|Appels bloqués|Count|Total|Nombre d’appels ayant dépassé la limite de débit ou de quota.|ApiName, OperationName, Region|
|CharactersTrained|Oui|Caractères formés|Count|Total|Nombre total de caractères formés.|ApiName, OperationName, Region|
|CharactersTranslated|Oui|Caractères traduits|Count|Total|Nombre total de caractères dans la requête de texte entrante.|ApiName, OperationName, Region|
|ClientErrors|Oui|Erreurs de client|Count|Total|Nombre d’appels avec erreur côté client (code de réponse HTTP : 4xx).|ApiName, OperationName, Region|
|DataIn|Oui|Données entrantes|Octets|Total|Taille des données entrantes en octets.|ApiName, OperationName, Region|
|DataOut|Oui|Données sortantes|Octets|Total|Taille des données sortantes en octets.|ApiName, OperationName, Region|
|Latence|Oui|Latence|Millisecondes|Average|Latence en millisecondes.|ApiName, OperationName, Region|
|ProcessedImages|Oui|Images traitées|Count|Total| Nombre de transactions pour le traitement des images.|ApiName, FeatureName, UsageChannel, Region|
|ServerErrors|Oui|Erreurs de serveur|Count|Total|Nombre d’appels avec erreur interne du service (code de réponse HTTP : 5xx).|ApiName, OperationName, Region|
|SpeechSessionDuration|Oui|Durée de la session vocale|Secondes|Total|Durée totale de la session vocale en secondes.|ApiName, OperationName, Region|
|SuccessfulCalls|Oui|Appels réussis|Count|Total|Nombre d’appels réussis.|ApiName, OperationName, Region|
|TotalCalls|Oui|Nombre total d’appels|Count|Total|Nombre total d’appels.|ApiName, OperationName, Region|
|TotalErrors|Oui|Nombre total d’erreurs|Count|Total|Nombre total d’appels avec réponse d’erreur (code de réponse HTTP : 4xx ou 5xx).|ApiName, OperationName, Region|
|TotalTokenCalls|Oui|Total d’appels de jeton|Count|Total|Nombre total d’appels de jeton.|ApiName, OperationName, Region|
|TotalTransactions|Oui|Nombre total de transactions|Count|Total|Nombre total de transactions.|Aucune dimension|


## <a name="microsoftcomputevirtualmachines"></a>Microsoft.Compute/virtualMachines

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|Crédits de processeur consommés|Oui|Crédits de processeur consommés|Count|Average|Nombre total de crédits consommés par la machine virtuelle|Aucune dimension|
|Crédits de processeurs restants|Oui|Crédits de processeurs restants|Count|Average|Nombre total de crédits pouvant être consommés|Aucune dimension|
|Longueur de file d’attente du disque de données|Oui|Longueur de file d’attente du disque de données (préversion)|Count|Average|Longueur de file d’attente (ou QD) du disque de données|Numéro d'unité logique|
|Octets lus/s sur disque de données|Oui|Octets lus/s sur disque de données (préversion)|CountPerSecond|Average|Octets/s lus sur un seul disque pendant la période d'analyse|Numéro d'unité logique|
|Opérations de lecture/s sur disque de données|Oui|Opérations de lecture/s sur disque de données (préversion)|CountPerSecond|Average|IOPS en lecture effectuées sur un seul disque pendant la période d'analyse|Numéro d'unité logique|
|Octets écrits/s sur disque de données|Oui|Octets/s écrits sur disque de données (préversion)|CountPerSecond|Average|Octets/s écrits sur un seul disque pendant la période d'analyse|Numéro d'unité logique|
|Opérations d’écriture/s sur disque de données|Oui|Opérations d’écriture/s sur disque de données (préversion)|CountPerSecond|Average|IOPS en écriture effectuées sur un seul disque pendant la période d'analyse|Numéro d'unité logique|
|Disk Read Bytes|Oui|Disk Read Bytes|Octets|Total|Octets lus à partir du disque pendant la période d’analyse|Aucune dimension|
|Disk Read Operations/Sec|Oui|Disk Read Operations/Sec|CountPerSecond|Average|E/S de lecture disque par seconde|Aucune dimension|
|Disk Write Bytes|Oui|Disk Write Bytes|Octets|Total|Octets écrits sur le disque pendant la période d’analyse|Aucune dimension|
|Disk Write Operations/Sec|Oui|Disk Write Operations/Sec|CountPerSecond|Average|E/S d’écriture disque par seconde|Aucune dimension|
|Flux entrants|Oui|Flux entrants|Count|Average|Les flux entrants correspondent aux flux actuels dans le sens entrant (trafic entrant dans la machine virtuelle)|Aucune dimension|
|Taux de création maximal de flux entrants|Oui|Taux de création maximal de flux entrants|CountPerSecond|Average|Taux de création maximal de flux entrants (trafic entrant dans la machine virtuelle)|Aucune dimension|
|Network In|Oui|Octets entrants réseau facturables (déprécié)|Octets|Total|Nombre d’octets facturables reçus sur toutes les interfaces réseau par la ou les machines virtuelles (trafic entrant) (déprécié)|Aucune dimension|
|Octets entrants réseau totaux|Oui|Octets entrants réseau totaux|Octets|Total|Le nombre d’octets reçus sur toutes les interfaces réseau par les ordinateurs virtuels (trafic entrant)|Aucune dimension|
|Network Out|Oui|Octets sortants réseau facturables (déprécié)|Octets|Total|Nombre d’octets facturables envoyés sur toutes les interfaces réseau par la ou les machines virtuelles (trafic sortant) (déprécié)|Aucune dimension|
|Octets sortants réseau totaux|Oui|Octets sortants réseau totaux|Octets|Total|Le nombre d’octets envoyés sur toutes les interfaces réseau par les ordinateurs virtuels (trafic sortant)|Aucune dimension|
|Longueur de file d’attente du disque du système d’exploitation|Oui|Longueur de file d’attente du disque de données (préversion)|Count|Average|Longueur de file d’attente (ou QD) du disque du système d’exploitation|Aucune dimension|
|Octets lus/s sur disque du système d’exploitation|Oui|Octets lus/s sur disque du système d’exploitation (préversion)|CountPerSecond|Average|Octets/s lus sur un seul disque pendant la période d'analyse du disque du système d’exploitation|Aucune dimension|
|Opérations de lecture/s sur disque du système d’exploitation|Oui|Opérations de lecture/s sur disque du système d’exploitation (préversion)|CountPerSecond|Average|IOPS en lecture effectuées sur un seul disque pendant la période d'analyse du disque du système d’exploitation|Aucune dimension|
|Octets/s écrits sur disque du système d’exploitation|Oui|Octets/s écrits sur disque du système d’exploitation (préversion)|CountPerSecond|Average|Octets/s écrits sur un seul disque pendant la période d'analyse du disque du système d’exploitation|Aucune dimension|
|Opérations d’écriture/s sur disque du système d’exploitation|Oui|Opérations d’écriture/s sur disque du système d’exploitation (préversion)|CountPerSecond|Average|IOPS en écriture effectuées sur un seul disque pendant la période d'analyse du disque du système d’exploitation|Aucune dimension|
|QD sur système d’exploitation par disque|Oui|QD du disque du système d'exploitation (déconseillé)|Count|Average|Longueur de file d’attente (ou QD) du disque du système d’exploitation|Aucune dimension|
|Octets lus/s sur système d’exploitation par disque|Oui|Octets lus/s sur système d'exploitation par disque (déconseillé)|CountPerSecond|Average|Octets/s lus sur un seul disque pendant la période d'analyse du disque du système d’exploitation|Aucune dimension|
|Opérations de lecture/s sur système d’exploitation par disque|Oui|Opérations de lecture/s sur disque du système d'exploitation (déconseillé)|CountPerSecond|Average|IOPS en lecture effectuées sur un seul disque pendant la période d'analyse du disque du système d’exploitation|Aucune dimension|
|Octets écrits/s sur système d’exploitation par disque|Oui|Octets écrits/s sur disque du système d'exploitation (déconseillé)|CountPerSecond|Average|Octets/s écrits sur un seul disque pendant la période d'analyse du disque du système d’exploitation|Aucune dimension|
|Opérations d’écriture/s sur système d’exploitation par disque|Oui|Opérations d’écriture/s sur disque du système d'exploitation (déconseillé)|CountPerSecond|Average|IOPS en écriture effectuées sur un seul disque pendant la période d'analyse du disque du système d’exploitation|Aucune dimension|
|Flux sortants|Oui|Flux sortants|Count|Average|Les flux sortants correspondent aux flux actuels dans le sens sortant (trafic sortant de la machine virtuelle)|Aucune dimension|
|Taux de création maximal de flux sortants|Oui|Taux de création maximal de flux sortants|CountPerSecond|Average|Taux de création maximal de flux sortants (trafic sortant de la machine virtuelle)|Aucune dimension|
|QD par disque|Oui|QD par disque de données (déconseillé)|Count|Average|Longueur de file d’attente (ou QD) du disque de données|SlotId|
|Octets lus/s par disque|Oui|Octets lus/s sur disque de données (déconseillé)|CountPerSecond|Average|Octets/s lus sur un seul disque pendant la période d'analyse|SlotId|
|Opérations de lecture/s par disque|Oui|Opérations de lecture/s sur disque de données (déconseillé)|CountPerSecond|Average|IOPS en lecture effectuées sur un seul disque pendant la période d'analyse|SlotId|
|Octets écrits/s par disque|Oui|Octets écrits/s sur disque de données (déconseillé)|CountPerSecond|Average|Octets/s écrits sur un seul disque pendant la période d'analyse|SlotId|
|Opérations d’écriture/s par disque|Oui|Opérations d’écriture/s sur disque de données (déconseillé)|CountPerSecond|Average|IOPS en écriture effectuées sur un seul disque pendant la période d'analyse|SlotId|
|Percentage CPU|Oui|Percentage CPU|Pourcentage|Average|Le pourcentage d’unités de calcul affectées actuellement utilisées par des machines virtuelles|Aucune dimension|
|Accès en lecture dans le cache de disque de données Premium|Oui|Accès en lecture dans le cache de disque de données Premium (préversion)|Pourcentage|Average|Accès en lecture dans le cache de disque de données Premium|Numéro d'unité logique|
|Échec de lecture dans le cache de disque de données Premium|Oui|Échec de lecture dans le cache de disque de données Premium (préversion)|Pourcentage|Average|Échec de lecture dans le cache de disque de données Premium|Numéro d'unité logique|
|Accès en lecture dans le cache de disque OS Premium|Oui|Accès en lecture dans le cache de disque OS Premium (préversion)|Pourcentage|Average|Accès en lecture dans le cache de disque OS Premium|Aucune dimension|
|Échec de lecture dans le cache de disque OS Premium|Oui|Échec de lecture dans le cache de disque OS Premium (préversion)|Pourcentage|Average|Échec de lecture dans le cache de disque OS Premium|Aucune dimension|


## <a name="microsoftcomputevirtualmachinescalesets"></a>Microsoft.Compute/virtualMachineScaleSets

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|Crédits de processeur consommés|Oui|Crédits de processeur consommés|Count|Average|Nombre total de crédits consommés par la machine virtuelle|Aucune dimension|
|Crédits de processeurs restants|Oui|Crédits de processeurs restants|Count|Average|Nombre total de crédits pouvant être consommés|Aucune dimension|
|Longueur de file d’attente du disque de données|Oui|Longueur de file d’attente du disque de données (préversion)|Count|Average|Longueur de file d’attente (ou QD) du disque de données|Numéro d'unité logique, VMName|
|Octets lus/s sur disque de données|Oui|Octets lus/s sur disque de données (préversion)|CountPerSecond|Average|Octets/s lus sur un seul disque pendant la période d'analyse|Numéro d'unité logique, VMName|
|Opérations de lecture/s sur disque de données|Oui|Opérations de lecture/s sur disque de données (préversion)|CountPerSecond|Average|IOPS en lecture effectuées sur un seul disque pendant la période d'analyse|Numéro d'unité logique, VMName|
|Octets écrits/s sur disque de données|Oui|Octets/s écrits sur disque de données (préversion)|CountPerSecond|Average|Octets/s écrits sur un seul disque pendant la période d'analyse|Numéro d'unité logique, VMName|
|Opérations d’écriture/s sur disque de données|Oui|Opérations d’écriture/s sur disque de données (préversion)|CountPerSecond|Average|IOPS en écriture effectuées sur un seul disque pendant la période d'analyse|Numéro d'unité logique, VMName|
|Disk Read Bytes|Oui|Disk Read Bytes|Octets|Total|Octets lus à partir du disque pendant la période d’analyse|VMName|
|Disk Read Operations/Sec|Oui|Disk Read Operations/Sec|CountPerSecond|Average|E/S de lecture disque par seconde|VMName|
|Disk Write Bytes|Oui|Disk Write Bytes|Octets|Total|Octets écrits sur le disque pendant la période d’analyse|VMName|
|Disk Write Operations/Sec|Oui|Disk Write Operations/Sec|CountPerSecond|Average|E/S d’écriture disque par seconde|VMName|
|Flux entrants|Oui|Flux entrants|Count|Average|Les flux entrants correspondent aux flux actuels dans le sens entrant (trafic entrant dans la machine virtuelle)|VMName|
|Taux de création maximal de flux entrants|Oui|Taux de création maximal de flux entrants|CountPerSecond|Average|Taux de création maximal de flux entrants (trafic entrant dans la machine virtuelle)|VMName|
|Network In|Oui|Octets entrants réseau facturables (déprécié)|Octets|Total|Nombre d’octets facturables reçus sur toutes les interfaces réseau par la ou les machines virtuelles (trafic entrant) (déprécié)|VMName|
|Octets entrants réseau totaux|Oui|Octets entrants réseau totaux|Octets|Total|Le nombre d’octets reçus sur toutes les interfaces réseau par les ordinateurs virtuels (trafic entrant)|VMName|
|Network Out|Oui|Octets sortants réseau facturables (déprécié)|Octets|Total|Nombre d’octets facturables envoyés sur toutes les interfaces réseau par la ou les machines virtuelles (trafic sortant) (déprécié)|VMName|
|Octets sortants réseau totaux|Oui|Octets sortants réseau totaux|Octets|Total|Le nombre d’octets envoyés sur toutes les interfaces réseau par les ordinateurs virtuels (trafic sortant)|VMName|
|Longueur de file d’attente du disque du système d’exploitation|Oui|Longueur de file d’attente du disque de données (préversion)|Count|Average|Longueur de file d’attente (ou QD) du disque du système d’exploitation|VMName|
|Octets lus/s sur disque du système d’exploitation|Oui|Octets lus/s sur disque du système d’exploitation (préversion)|CountPerSecond|Average|Octets/s lus sur un seul disque pendant la période d'analyse du disque du système d’exploitation|VMName|
|Opérations de lecture/s sur disque du système d’exploitation|Oui|Opérations de lecture/s sur disque du système d’exploitation (préversion)|CountPerSecond|Average|IOPS en lecture effectuées sur un seul disque pendant la période d'analyse du disque du système d’exploitation|VMName|
|Octets/s écrits sur disque du système d’exploitation|Oui|Octets/s écrits sur disque du système d’exploitation (préversion)|CountPerSecond|Average|Octets/s écrits sur un seul disque pendant la période d'analyse du disque du système d’exploitation|VMName|
|Opérations d’écriture/s sur disque du système d’exploitation|Oui|Opérations d’écriture/s sur disque du système d’exploitation (préversion)|CountPerSecond|Average|IOPS en écriture effectuées sur un seul disque pendant la période d'analyse du disque du système d’exploitation|VMName|
|QD sur système d’exploitation par disque|Oui|QD du disque du système d'exploitation (déconseillé)|Count|Average|Longueur de file d’attente (ou QD) du disque du système d’exploitation|Aucune dimension|
|Octets lus/s sur système d’exploitation par disque|Oui|Octets lus/s sur système d'exploitation par disque (déconseillé)|CountPerSecond|Average|Octets/s lus sur un seul disque pendant la période d'analyse du disque du système d’exploitation|Aucune dimension|
|Opérations de lecture/s sur système d’exploitation par disque|Oui|Opérations de lecture/s sur disque du système d'exploitation (déconseillé)|CountPerSecond|Average|IOPS en lecture effectuées sur un seul disque pendant la période d'analyse du disque du système d’exploitation|Aucune dimension|
|Octets écrits/s sur système d’exploitation par disque|Oui|Octets écrits/s sur disque du système d'exploitation (déconseillé)|CountPerSecond|Average|Octets/s écrits sur un seul disque pendant la période d'analyse du disque du système d’exploitation|Aucune dimension|
|Opérations d’écriture/s sur système d’exploitation par disque|Oui|Opérations d’écriture/s sur disque du système d'exploitation (déconseillé)|CountPerSecond|Average|IOPS en écriture effectuées sur un seul disque pendant la période d'analyse du disque du système d’exploitation|Aucune dimension|
|Flux sortants|Oui|Flux sortants|Count|Average|Les flux sortants correspondent aux flux actuels dans le sens sortant (trafic sortant de la machine virtuelle)|VMName|
|Taux de création maximal de flux sortants|Oui|Taux de création maximal de flux sortants|CountPerSecond|Average|Taux de création maximal de flux sortants (trafic sortant de la machine virtuelle)|VMName|
|QD par disque|Oui|QD par disque de données (déconseillé)|Count|Average|Longueur de file d’attente (ou QD) du disque de données|SlotId|
|Octets lus/s par disque|Oui|Octets lus/s sur disque de données (déconseillé)|CountPerSecond|Average|Octets/s lus sur un seul disque pendant la période d'analyse|SlotId|
|Opérations de lecture/s par disque|Oui|Opérations de lecture/s sur disque de données (déconseillé)|CountPerSecond|Average|IOPS en lecture effectuées sur un seul disque pendant la période d'analyse|SlotId|
|Octets écrits/s par disque|Oui|Octets écrits/s sur disque de données (déconseillé)|CountPerSecond|Average|Octets/s écrits sur un seul disque pendant la période d'analyse|SlotId|
|Opérations d’écriture/s par disque|Oui|Opérations d’écriture/s sur disque de données (déconseillé)|CountPerSecond|Average|IOPS en écriture effectuées sur un seul disque pendant la période d'analyse|SlotId|
|Percentage CPU|Oui|Percentage CPU|Pourcentage|Average|Le pourcentage d’unités de calcul affectées actuellement utilisées par des machines virtuelles|VMName|
|Accès en lecture dans le cache de disque de données Premium|Oui|Accès en lecture dans le cache de disque de données Premium (préversion)|Pourcentage|Average|Accès en lecture dans le cache de disque de données Premium|Numéro d'unité logique, VMName|
|Échec de lecture dans le cache de disque de données Premium|Oui|Échec de lecture dans le cache de disque de données Premium (préversion)|Pourcentage|Average|Échec de lecture dans le cache de disque de données Premium|Numéro d'unité logique, VMName|
|Accès en lecture dans le cache de disque OS Premium|Oui|Accès en lecture dans le cache de disque OS Premium (préversion)|Pourcentage|Average|Accès en lecture dans le cache de disque OS Premium|VMName|
|Échec de lecture dans le cache de disque OS Premium|Oui|Échec de lecture dans le cache de disque OS Premium (préversion)|Pourcentage|Average|Échec de lecture dans le cache de disque OS Premium|VMName|


## <a name="microsoftcomputevirtualmachinescalesetsvirtualmachines"></a>Microsoft.Compute/virtualMachineScaleSets/virtualMachines

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|Crédits de processeur consommés|Oui|Crédits de processeur consommés|Count|Average|Nombre total de crédits consommés par la machine virtuelle|Aucune dimension|
|Crédits de processeurs restants|Oui|Crédits de processeurs restants|Count|Average|Nombre total de crédits pouvant être consommés|Aucune dimension|
|Longueur de file d’attente du disque de données|Oui|Longueur de file d’attente du disque de données (préversion)|Count|Average|Longueur de file d’attente (ou QD) du disque de données|Numéro d'unité logique|
|Octets lus/s sur disque de données|Oui|Octets lus/s sur disque de données (préversion)|CountPerSecond|Average|Octets/s lus sur un seul disque pendant la période d'analyse|Numéro d'unité logique|
|Opérations de lecture/s sur disque de données|Oui|Opérations de lecture/s sur disque de données (préversion)|CountPerSecond|Average|IOPS en lecture effectuées sur un seul disque pendant la période d'analyse|Numéro d'unité logique|
|Octets écrits/s sur disque de données|Oui|Octets/s écrits sur disque de données (préversion)|CountPerSecond|Average|Octets/s écrits sur un seul disque pendant la période d'analyse|Numéro d'unité logique|
|Opérations d’écriture/s sur disque de données|Oui|Opérations d’écriture/s sur disque de données (préversion)|CountPerSecond|Average|IOPS en écriture effectuées sur un seul disque pendant la période d'analyse|Numéro d'unité logique|
|Disk Read Bytes|Oui|Disk Read Bytes|Octets|Total|Octets lus à partir du disque pendant la période d’analyse|Aucune dimension|
|Disk Read Operations/Sec|Oui|Disk Read Operations/Sec|CountPerSecond|Average|E/S de lecture disque par seconde|Aucune dimension|
|Disk Write Bytes|Oui|Disk Write Bytes|Octets|Total|Octets écrits sur le disque pendant la période d’analyse|Aucune dimension|
|Disk Write Operations/Sec|Oui|Disk Write Operations/Sec|CountPerSecond|Average|E/S d’écriture disque par seconde|Aucune dimension|
|Flux entrants|Oui|Flux entrants|Count|Average|Les flux entrants correspondent aux flux actuels dans le sens entrant (trafic entrant dans la machine virtuelle)|Aucune dimension|
|Taux de création maximal de flux entrants|Oui|Taux de création maximal de flux entrants|CountPerSecond|Average|Taux de création maximal de flux entrants (trafic entrant dans la machine virtuelle)|Aucune dimension|
|Network In|Oui|Octets entrants réseau facturables (déprécié)|Octets|Total|Nombre d’octets facturables reçus sur toutes les interfaces réseau par la ou les machines virtuelles (trafic entrant) (déprécié)|Aucune dimension|
|Octets entrants réseau totaux|Oui|Octets entrants réseau totaux|Octets|Total|Le nombre d’octets reçus sur toutes les interfaces réseau par les ordinateurs virtuels (trafic entrant)|Aucune dimension|
|Network Out|Oui|Octets sortants réseau facturables (déprécié)|Octets|Total|Nombre d’octets facturables envoyés sur toutes les interfaces réseau par la ou les machines virtuelles (trafic sortant) (déprécié)|Aucune dimension|
|Octets sortants réseau totaux|Oui|Octets sortants réseau totaux|Octets|Total|Le nombre d’octets envoyés sur toutes les interfaces réseau par les ordinateurs virtuels (trafic sortant)|Aucune dimension|
|Longueur de file d’attente du disque du système d’exploitation|Oui|Longueur de file d’attente du disque de données (préversion)|Count|Average|Longueur de file d’attente (ou QD) du disque du système d’exploitation|Aucune dimension|
|Octets lus/s sur disque du système d’exploitation|Oui|Octets lus/s sur disque du système d’exploitation (préversion)|CountPerSecond|Average|Octets/s lus sur un seul disque pendant la période d'analyse du disque du système d’exploitation|Aucune dimension|
|Opérations de lecture/s sur disque du système d’exploitation|Oui|Opérations de lecture/s sur disque du système d’exploitation (préversion)|CountPerSecond|Average|IOPS en lecture effectuées sur un seul disque pendant la période d'analyse du disque du système d’exploitation|Aucune dimension|
|Octets/s écrits sur disque du système d’exploitation|Oui|Octets/s écrits sur disque du système d’exploitation (préversion)|CountPerSecond|Average|Octets/s écrits sur un seul disque pendant la période d'analyse du disque du système d’exploitation|Aucune dimension|
|Opérations d’écriture/s sur disque du système d’exploitation|Oui|Opérations d’écriture/s sur disque du système d’exploitation (préversion)|CountPerSecond|Average|IOPS en écriture effectuées sur un seul disque pendant la période d'analyse du disque du système d’exploitation|Aucune dimension|
|QD sur système d’exploitation par disque|Oui|QD du disque du système d'exploitation (déconseillé)|Count|Average|Longueur de file d’attente (ou QD) du disque du système d’exploitation|Aucune dimension|
|Octets lus/s sur système d’exploitation par disque|Oui|Octets lus/s sur système d'exploitation par disque (déconseillé)|CountPerSecond|Average|Octets/s lus sur un seul disque pendant la période d'analyse du disque du système d’exploitation|Aucune dimension|
|Opérations de lecture/s sur système d’exploitation par disque|Oui|Opérations de lecture/s sur disque du système d'exploitation (déconseillé)|CountPerSecond|Average|IOPS en lecture effectuées sur un seul disque pendant la période d'analyse du disque du système d’exploitation|Aucune dimension|
|Octets écrits/s sur système d’exploitation par disque|Oui|Octets écrits/s sur disque du système d'exploitation (déconseillé)|CountPerSecond|Average|Octets/s écrits sur un seul disque pendant la période d'analyse du disque du système d’exploitation|Aucune dimension|
|Opérations d’écriture/s sur système d’exploitation par disque|Oui|Opérations d’écriture/s sur disque du système d'exploitation (déconseillé)|CountPerSecond|Average|IOPS en écriture effectuées sur un seul disque pendant la période d'analyse du disque du système d’exploitation|Aucune dimension|
|Flux sortants|Oui|Flux sortants|Count|Average|Les flux sortants correspondent aux flux actuels dans le sens sortant (trafic sortant de la machine virtuelle)|Aucune dimension|
|Taux de création maximal de flux sortants|Oui|Taux de création maximal de flux sortants|CountPerSecond|Average|Taux de création maximal de flux sortants (trafic sortant de la machine virtuelle)|Aucune dimension|
|QD par disque|Oui|QD par disque de données (déconseillé)|Count|Average|Longueur de file d’attente (ou QD) du disque de données|SlotId|
|Octets lus/s par disque|Oui|Octets lus/s sur disque de données (déconseillé)|CountPerSecond|Average|Octets/s lus sur un seul disque pendant la période d'analyse|SlotId|
|Opérations de lecture/s par disque|Oui|Opérations de lecture/s sur disque de données (déconseillé)|CountPerSecond|Average|IOPS en lecture effectuées sur un seul disque pendant la période d'analyse|SlotId|
|Octets écrits/s par disque|Oui|Octets écrits/s sur disque de données (déconseillé)|CountPerSecond|Average|Octets/s écrits sur un seul disque pendant la période d'analyse|SlotId|
|Opérations d’écriture/s par disque|Oui|Opérations d’écriture/s sur disque de données (déconseillé)|CountPerSecond|Average|IOPS en écriture effectuées sur un seul disque pendant la période d'analyse|SlotId|
|Percentage CPU|Oui|Percentage CPU|Pourcentage|Average|Le pourcentage d’unités de calcul affectées actuellement utilisées par des machines virtuelles|Aucune dimension|
|Accès en lecture dans le cache de disque de données Premium|Oui|Accès en lecture dans le cache de disque de données Premium (préversion)|Pourcentage|Average|Accès en lecture dans le cache de disque de données Premium|Numéro d'unité logique|
|Échec de lecture dans le cache de disque de données Premium|Oui|Échec de lecture dans le cache de disque de données Premium (préversion)|Pourcentage|Average|Échec de lecture dans le cache de disque de données Premium|Numéro d'unité logique|
|Accès en lecture dans le cache de disque OS Premium|Oui|Accès en lecture dans le cache de disque OS Premium (préversion)|Pourcentage|Average|Accès en lecture dans le cache de disque OS Premium|Aucune dimension|
|Échec de lecture dans le cache de disque OS Premium|Oui|Échec de lecture dans le cache de disque OS Premium (préversion)|Pourcentage|Average|Échec de lecture dans le cache de disque OS Premium|Aucune dimension|


## <a name="microsoftcontainerinstancecontainergroups"></a>Microsoft.ContainerInstance/containerGroups

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|CpuUsage|Oui|Utilisation du processeur|Count|Average|Utilisation du processeur sur tous les cœurs en millicores.|containerName|
|MemoryUsage|Oui|Utilisation de la mémoire|Octets|Average|Utilisation de la mémoire totale en octets.|containerName|
|NetworkBytesReceivedPerSecond|Oui|Octets réseau reçus par seconde|Octets|Average|Octets réseau reçus par seconde.|Aucune dimension|
|NetworkBytesTransmittedPerSecond|Oui|Octets réseau transmis par seconde|Octets|Average|Octets réseau transmis par seconde.|Aucune dimension|


## <a name="microsoftcontainerregistryregistries"></a>Microsoft.ContainerRegistry/registries

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|AgentPoolCPUTime|Oui|Temps processeur du pool d’agents|Secondes|Total|Temps processeur du pool d’agents en secondes|Aucune dimension|
|RunDuration|Oui|Durée d’exécution|Millisecondes|Total|Durée d’exécution en millisecondes|Aucune dimension|
|SuccessfulPullCount|Oui|Nombre d'extractions réussies|Count|Average|Nombre d'images extraites réussies|Aucune dimension|
|SuccessfulPushCount|Oui|Nombre d'envois (push) réussis|Count|Average|Nombre d'images envoyées (push) réussies|Aucune dimension|
|TotalPullCount|Oui|Nombre total d'extractions|Count|Average|Nombre total d'images extraites|Aucune dimension|
|TotalPushCount|Oui|Nombre total d'envois (push)|Count|Average|Nombre total d'images envoyées (push)|Aucune dimension|


## <a name="microsoftcontainerservicemanagedclusters"></a>Microsoft.ContainerService/managedClusters

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|kube_node_status_allocatable_cpu_cores|Non|Nombre total de cœurs d’unité centrale disponibles dans un cluster géré|Count|Average|Nombre total de cœurs d’unité centrale disponibles dans un cluster géré|Aucune dimension|
|kube_node_status_allocatable_memory_bytes|Non|Quantité totale de mémoire disponible dans un cluster géré|Octets|Average|Quantité totale de mémoire disponible dans un cluster géré|Aucune dimension|
|kube_node_status_condition|Non|États pour les différentes conditions de nœud|Count|Average|États pour les différentes conditions de nœud|condition, status, status2, node|
|kube_pod_status_phase|Non|Nombre de pods par phase|Count|Average|Nombre de pods par phase|phase, espace de noms, pod|
|kube_pod_status_ready|Non|Nombre de pods en état Prêt|Count|Average|Nombre de pods en état Prêt|espace de noms, pod|


## <a name="microsoftcustomprovidersresourceproviders"></a>Microsoft.CustomProviders/resourceproviders

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|FailedRequests|Oui|Demandes ayant échoué|Count|Total|Obtient les journaux disponibles pour les fournisseurs de ressources personnalisés|HttpMethod, CallPath, StatusCode|
|SuccessfullRequests|Oui|Requêtes ayant réussi|Count|Total|Demandes réussies effectuées par le fournisseur personnalisé|HttpMethod, CallPath, StatusCode|


## <a name="microsoftdataboxedgedataboxedgedevices"></a>Microsoft.DataBoxEdge/dataBoxEdgeDevices

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|AvailableCapacity|Oui|Capacité disponible|Octets|Average|Capacité disponible en octets pendant la période du rapport.|Aucune dimension|
|BytesUploadedToCloud|Oui|Octets cloud chargés (appareil)|Octets|Average|Nombre total d’octets chargés vers Azure à partir d’un appareil pendant la période du rapport.|Aucune dimension|
|BytesUploadedToCloudPerShare|Oui|Octets cloud chargés (partage)|Octets|Average|Nombre total d’octets chargés vers Azure à partir d’un partage pendant la période du rapport.|Partager|
|CloudReadThroughput|Oui|Débit de téléchargement cloud|BytesPerSecond|Average|Débit de téléchargement cloud vers Azure pendant la période du rapport.|Aucune dimension|
|CloudReadThroughputPerShare|Oui|Débit de téléchargement cloud (partage)|BytesPerSecond|Average|Débit de téléchargement vers Azure à partir d’un partage pendant la période du rapport.|Partager|
|CloudUploadThroughput|Oui|Débit de chargement cloud|BytesPerSecond|Average|Débit de chargement cloud vers Azure pendant la période du rapport.|Aucune dimension|
|CloudUploadThroughputPerShare|Oui|Débit de chargement cloud (partage)|BytesPerSecond|Average|Débit de chargement vers Azure à partir d’un partage pendant la période du rapport.|Partager|
|HyperVMemoryUtilization|Oui|Computing en périphérie - Utilisation de la mémoire|Pourcentage|Average|Quantité de RAM utilisée|InstanceName|
|HyperVVirtualProcessorUtilization|Oui|Computing en périphérie - Pourcentage du processeur|Pourcentage|Average|Pourcentage d’utilisation du processeur|InstanceName|
|NICReadThroughput|Oui|Débit de lecture (réseau)|BytesPerSecond|Average|Débit de lecture de l’interface réseau de l’appareil dans la période du rapport pour tous les volumes de la passerelle.|InstanceName|
|NICWriteThroughput|Oui|Débit d’écriture (réseau)|BytesPerSecond|Average|Débit d'écriture de l’interface réseau de l’appareil dans la période du rapport pour tous les volumes de la passerelle.|InstanceName|
|TotalCapacity|Oui|Capacité totale|Octets|Average|Capacité totale|Aucune dimension|


## <a name="microsoftdatafactorydatafactories"></a>Microsoft.DataFactory/datafactories

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|FailedRuns|Oui|Exécutions échouées|Count|Total||pipelineName, activityName|
|SuccessfulRuns|Oui|Exécutions réussies|Count|Total||pipelineName, activityName|


## <a name="microsoftdatafactoryfactories"></a>Microsoft.DataFactory/factories

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|ActivityCancelledRuns|Oui|Métriques d’exécutions d’activité annulées|Count|Total||ActivityType, PipelineName, FailureType, Name|
|ActivityFailedRuns|Oui|Métriques d’exécutions d’activité ayant échoué|Count|Total||ActivityType, PipelineName, FailureType, Name|
|ActivitySucceededRuns|Oui|Métriques d’exécutions d’activité ayant abouti|Count|Total||ActivityType, PipelineName, FailureType, Name|
|FactorySizeInGbUnits|Oui|Taille de fabrique totale (unité Go)|Count|Maximale||Aucune dimension|
|IntegrationRuntimeAvailableMemory|Oui|Mémoire disponible du runtime d’intégration|Octets|Average||IntegrationRuntimeName, NodeName|
|IntegrationRuntimeAvailableNodeNumber|Oui|Nombre de nœuds disponibles pour le runtime d’intégration|Count|Average||IntegrationRuntimeName|
|IntegrationRuntimeAverageTaskPickupDelay|Oui|Durée de la file d’attente du runtime d’intégration|Secondes|Average||IntegrationRuntimeName|
|IntegrationRuntimeCpuPercentage|Oui|Utilisation du processeur du runtime d’intégration|Pourcentage|Average||IntegrationRuntimeName, NodeName|
|IntegrationRuntimeQueueLength|Oui|Longueur de la file d’attente du runtime d’intégration|Count|Average||IntegrationRuntimeName|
|MaxAllowedFactorySizeInGbUnits|Oui|Taille maximale de fabrique autorisée (unité Go)|Count|Maximale||Aucune dimension|
|MaxAllowedResourceCount|Oui|Nombre maximal d’entités autorisées|Count|Maximale||Aucune dimension|
|PipelineCancelledRuns|Oui|Métriques d’exécutions de pipeline annulées|Count|Total||FailureType, Name|
|PipelineFailedRuns|Oui|Métriques d’exécutions de pipeline ayant échoué|Count|Total||FailureType, Name|
|PipelineSucceededRuns|Oui|Métriques d’exécutions de pipeline ayant abouti|Count|Total||FailureType, Name|
|ResourceCount|Oui|Nombre total d’entités|Count|Maximale||Aucune dimension|
|TriggerCancelledRuns|Oui|Métriques d’exécutions de déclencheur annulées|Count|Total||Name, FailureType|
|TriggerFailedRuns|Oui|Métriques d’exécutions de déclencheur ayant échoué|Count|Total||Name, FailureType|
|TriggerSucceededRuns|Oui|Métriques d’exécutions de déclencheur ayant abouti|Count|Total||Name, FailureType|


## <a name="microsoftdatalakestoreaccounts"></a>Microsoft.DataLakeStore/accounts

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|DataRead|Oui|Données lues|Octets|Total|Volume total de données lues à partir du compte.|Aucune dimension|
|DataWritten|Oui|Données écrites|Octets|Total|Volume total de données écrites dans le compte.|Aucune dimension|
|ReadRequests|Oui|Demandes de lecture|Count|Total|Nombre de demandes de lecture de données sur le compte.|Aucune dimension|
|TotalStorage|Oui|Stockage total|Octets|Maximale|Volume total de données stockées dans le compte.|Aucune dimension|
|WriteRequests|Oui|Demandes d’écriture|Count|Total|Nombre de demandes d’écriture de données sur le compte.|Aucune dimension|


## <a name="microsoftdbformariadbservers"></a>Microsoft.DBforMariaDB/servers

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|active_connections|Oui|Connexions actives|Count|Average|Connexions actives|Aucune dimension|
|backup_storage_used|Oui|Stockage de sauvegarde utilisé|Octets|Average|Stockage de sauvegarde utilisé|Aucune dimension|
|connections_failed|Oui|Connexions ayant échoué|Count|Total|Connexions ayant échoué|Aucune dimension|
|cpu_percent|Oui|Pourcentage d’UC|Pourcentage|Average|Pourcentage d’UC|Aucune dimension|
|io_consumption_percent|Oui|Pourcentage d’E/S|Pourcentage|Average|Pourcentage d’E/S|Aucune dimension|
|memory_percent|Oui|Pourcentage de mémoire|Pourcentage|Average|Pourcentage de mémoire|Aucune dimension|
|network_bytes_egress|Oui|Network Out|Octets|Total|Sortie réseau entre connexions actives|Aucune dimension|
|network_bytes_ingress|Oui|Network In|Octets|Total|Entrée réseau entre connexions actives|Aucune dimension|
|seconds_behind_master|Oui|Décalage de la réplication en secondes|Count|Maximale|Décalage de la réplication en secondes|Aucune dimension|
|serverlog_storage_limit|Oui|Limite de stockage du journal du serveur|Octets|Average|Limite de stockage du journal du serveur|Aucune dimension|
|serverlog_storage_percent|Oui|Pourcentage de stockage du journal du serveur|Pourcentage|Average|Pourcentage de stockage du journal du serveur|Aucune dimension|
|serverlog_storage_usage|Oui|Stockage du journal du serveur utilisé|Octets|Average|Stockage du journal du serveur utilisé|Aucune dimension|
|storage_limit|Oui|Limite de stockage|Octets|Maximale|Limite de stockage|Aucune dimension|
|storage_percent|Oui|Pourcentage de stockage|Pourcentage|Average|Pourcentage de stockage|Aucune dimension|
|storage_used|Oui|Stockage utilisé|Octets|Average|Stockage utilisé|Aucune dimension|


## <a name="microsoftdbformysqlservers"></a>Microsoft.DBforMySQL/servers

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|active_connections|Oui|Connexions actives|Count|Average|Connexions actives|Aucune dimension|
|backup_storage_used|Oui|Stockage de sauvegarde utilisé|Octets|Average|Stockage de sauvegarde utilisé|Aucune dimension|
|connections_failed|Oui|Connexions ayant échoué|Count|Total|Connexions ayant échoué|Aucune dimension|
|cpu_percent|Oui|Pourcentage d’UC|Pourcentage|Average|Pourcentage d’UC|Aucune dimension|
|io_consumption_percent|Oui|Pourcentage d’E/S|Pourcentage|Average|Pourcentage d’E/S|Aucune dimension|
|memory_percent|Oui|Pourcentage de mémoire|Pourcentage|Average|Pourcentage de mémoire|Aucune dimension|
|network_bytes_egress|Oui|Network Out|Octets|Total|Sortie réseau entre connexions actives|Aucune dimension|
|network_bytes_ingress|Oui|Network In|Octets|Total|Entrée réseau entre connexions actives|Aucune dimension|
|seconds_behind_master|Oui|Décalage de la réplication en secondes|Count|Maximale|Décalage de la réplication en secondes|Aucune dimension|
|serverlog_storage_limit|Oui|Limite de stockage du journal du serveur|Octets|Maximale|Limite de stockage du journal du serveur|Aucune dimension|
|serverlog_storage_percent|Oui|Pourcentage de stockage du journal du serveur|Pourcentage|Average|Pourcentage de stockage du journal du serveur|Aucune dimension|
|serverlog_storage_usage|Oui|Stockage du journal du serveur utilisé|Octets|Average|Stockage du journal du serveur utilisé|Aucune dimension|
|storage_limit|Oui|Limite de stockage|Octets|Maximale|Limite de stockage|Aucune dimension|
|storage_percent|Oui|Pourcentage de stockage|Pourcentage|Average|Pourcentage de stockage|Aucune dimension|
|storage_used|Oui|Stockage utilisé|Octets|Average|Stockage utilisé|Aucune dimension|


## <a name="microsoftdbforpostgresqlservers"></a>Microsoft.DBforPostgreSQL/servers

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|active_connections|Oui|Connexions actives|Count|Average|Connexions actives|Aucune dimension|
|backup_storage_used|Oui|Stockage de sauvegarde utilisé|Octets|Average|Stockage de sauvegarde utilisé|Aucune dimension|
|connections_failed|Oui|Connexions ayant échoué|Count|Total|Connexions ayant échoué|Aucune dimension|
|cpu_percent|Oui|Pourcentage d’UC|Pourcentage|Average|Pourcentage d’UC|Aucune dimension|
|io_consumption_percent|Oui|Pourcentage d’E/S|Pourcentage|Average|Pourcentage d’E/S|Aucune dimension|
|memory_percent|Oui|Pourcentage de mémoire|Pourcentage|Average|Pourcentage de mémoire|Aucune dimension|
|network_bytes_egress|Oui|Network Out|Octets|Total|Sortie réseau entre connexions actives|Aucune dimension|
|network_bytes_ingress|Oui|Network In|Octets|Total|Entrée réseau entre connexions actives|Aucune dimension|
|pg_replica_log_delay_in_bytes|Oui|Retard maximum entre réplicas|Octets|Maximale|Retard en octets du réplica le plus en retard|Aucune dimension|
|pg_replica_log_delay_in_seconds|Oui|Retard du réplica|Secondes|Maximale|Retard du réplica en secondes|Aucune dimension|
|serverlog_storage_limit|Oui|Limite de stockage du journal du serveur|Octets|Maximale|Limite de stockage du journal du serveur|Aucune dimension|
|serverlog_storage_percent|Oui|Pourcentage de stockage du journal du serveur|Pourcentage|Average|Pourcentage de stockage du journal du serveur|Aucune dimension|
|serverlog_storage_usage|Oui|Stockage du journal du serveur utilisé|Octets|Average|Stockage du journal du serveur utilisé|Aucune dimension|
|storage_limit|Oui|Limite de stockage|Octets|Maximale|Limite de stockage|Aucune dimension|
|storage_percent|Oui|Pourcentage de stockage|Pourcentage|Average|Pourcentage de stockage|Aucune dimension|
|storage_used|Oui|Stockage utilisé|Octets|Average|Stockage utilisé|Aucune dimension|


## <a name="microsoftdbforpostgresqlserversv2"></a>Microsoft.DBforPostgreSQL/serversv2

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|active_connections|Oui|Connexions actives|Count|Average|Connexions actives|Aucune dimension|
|cpu_percent|Oui|Pourcentage d’UC|Pourcentage|Average|Pourcentage d’UC|Aucune dimension|
|iops|Oui|E/S par seconde|Count|Average|Opérations d'E/S par seconde|Aucune dimension|
|memory_percent|Oui|Pourcentage de mémoire|Pourcentage|Average|Pourcentage de mémoire|Aucune dimension|
|network_bytes_egress|Oui|Network Out|Octets|Total|Sortie réseau entre connexions actives|Aucune dimension|
|network_bytes_ingress|Oui|Network In|Octets|Total|Entrée réseau entre connexions actives|Aucune dimension|
|storage_percent|Oui|Pourcentage de stockage|Pourcentage|Average|Pourcentage de stockage|Aucune dimension|
|storage_used|Oui|Stockage utilisé|Octets|Average|Stockage utilisé|Aucune dimension|


## <a name="microsoftdbforpostgresqlsingleservers"></a>Microsoft.DBforPostgreSQL/singleservers

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|active_connections|Oui|Connexions actives|Count|Average|Connexions actives|Aucune dimension|
|connections_failed|Oui|Connexions ayant échoué|Count|Total|Connexions ayant échoué|Aucune dimension|
|connections_succeeded|Oui|Connexions ayant abouti|Count|Total|Connexions ayant abouti|Aucune dimension|
|cpu_percent|Oui|Pourcentage d’UC|Pourcentage|Average|Pourcentage d’UC|Aucune dimension|
|iops|Oui|E/S par seconde|Count|Average|Opérations d'E/S par seconde|Aucune dimension|
|maximum_used_transactionIDs|Oui|Nombre maximal d’ID de transaction utilisés|Count|Average|Nombre maximal d’ID de transaction utilisés|Aucune dimension|
|memory_percent|Oui|Pourcentage de mémoire|Pourcentage|Average|Pourcentage de mémoire|Aucune dimension|
|network_bytes_egress|Oui|Network Out|Octets|Total|Sortie réseau entre connexions actives|Aucune dimension|
|network_bytes_ingress|Oui|Network In|Octets|Total|Entrée réseau entre connexions actives|Aucune dimension|
|storage_percent|Oui|Pourcentage de stockage|Pourcentage|Average|Pourcentage de stockage|Aucune dimension|
|storage_used|Oui|Stockage utilisé|Octets|Average|Stockage utilisé|Aucune dimension|


## <a name="microsoftdevicesiothubs"></a>Microsoft.Devices/IotHubs

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|c2d.commands.egress.abandon.success|Oui|Messages cloud vers appareil abandonnés|Count|Total|Nombre de messages cloud vers appareil abandonnés par l’appareil|Aucune dimension|
|c2d.commands.egress.complete.success|Oui|Remises de messages cloud vers appareil terminées|Count|Total|Nombre de remises de messages cloud vers appareil terminées avec succès par l’appareil|Aucune dimension|
|c2d.commands.egress.reject.success|Oui|Messages cloud vers appareil rejetés|Count|Total|Nombre de messages cloud vers appareil rejetés par l’appareil|Aucune dimension|
|c2d.methods.failure|Oui|Appels de méthode directe en échec|Count|Total|Total des appels de méthode directe en échec.|Aucune dimension|
|c2d.methods.requestSize|Oui|Taille de demande des appels de méthode directe|Octets|Average|Moyenne, minimum et maximum de toutes les demandes de méthode directe réussies.|Aucune dimension|
|c2d.methods.responseSize|Oui|Taille de réponse des appels de méthode directe|Octets|Average|Moyenne, minimum et maximum de toutes les réponses de méthode directe réussies.|Aucune dimension|
|c2d.methods.success|Oui|Appels de méthode directe réussis|Count|Total|Total des appels de méthode directe réussis.|Aucune dimension|
|c2d.twin.read.failure|Oui|Lectures de représentations de serveur principal en échec|Count|Total|Total des lectures de représentations en échec initiées par un serveur principal.|Aucune dimension|
|c2d.twin.read.size|Oui|Taille de la réponse des lectures de représentations de serveur principal|Octets|Average|Moyenne, minimum et maximum de toutes les lectures de représentations réussies initiées par un serveur principal.|Aucune dimension|
|c2d.twin.read.success|Oui|Lectures de représentations réussies de serveur principal|Count|Total|Total des lectures de représentations réussies initiées par un serveur principal.|Aucune dimension|
|c2d.twin.update.failure|Oui|Mises à jour de représentations de serveur principal en échec|Count|Total|Total des mises à jour de représentations en échec initiées par un serveur principal.|Aucune dimension|
|c2d.twin.update.size|Oui|Taille des mises à jour de représentations de serveur principal|Octets|Average|Taille moyenne, minimale et maximale de toutes les mises à jour de représentations réussies initiées par un serveur principal.|Aucune dimension|
|c2d.twin.update.success|Oui|Mises à jour de représentations réussies de serveur principal|Count|Total|Total des mises à jour de représentations réussies initiées par un serveur principal.|Aucune dimension|
|C2DMessagesExpired|Oui|Messages cloud vers appareil ayant expiré (préversion)|Count|Total|Nombre de messages cloud vers appareil ayant expiré|Aucune dimension|
|configurations|Oui|Métriques de configuration|Count|Total|Métriques pour les opérations de configuration|Aucune dimension|
|connectedDeviceCount|Non|Appareils connectés (préversion)|Count|Average|Nombre d’appareils connectés à votre hub IoT|Aucune dimension|
|d2c.endpoints.egress.builtIn.events|Oui|Routage : messages remis à des messages/événements|Count|Total|Nombre de fois où le routage IoT Hub a correctement remis des messages au point de terminaison intégré (messages/événements).|Aucune dimension|
|d2c.endpoints.egress.eventHubs|Oui|Routage : messages remis à Event Hub|Count|Total|Nombre de fois où le routage IoT Hub a correctement remis des messages à des points de terminaison Event Hub.|Aucune dimension|
|d2c.endpoints.egress.serviceBusQueues|Oui|Routage : messages remis à la file d’attente Service Bus|Count|Total|Nombre de fois où le routage IoT Hub a correctement remis des messages aux points de terminaison de file d’attente Service Bus.|Aucune dimension|
|d2c.endpoints.egress.serviceBusTopics|Oui|Routage : messages remis à la rubrique Service Bus|Count|Total|Nombre de fois où le routage IoT Hub a correctement remis des messages à des points de terminaison de rubrique Service Bus.|Aucune dimension|
|d2c.endpoints.egress.storage|Oui|Routage : messages remis au stockage|Count|Total|Nombre de fois où le routage IoT Hub a correctement remis des messages à des points de terminaison de stockage.|Aucune dimension|
|d2c.endpoints.egress.storage.blobs|Oui|Routage : objets blob remis au stockage|Count|Total|Nombre de fois où le routage IoT Hub a remis des objets blob à des points de terminaison de stockage.|Aucune dimension|
|d2c.endpoints.egress.storage.bytes|Oui|Routage : données remises au stockage|Octets|Total|Quantité de données (octets) que le routage IoT Hub a remis aux points de terminaison de stockage.|Aucune dimension|
|d2c.endpoints.latency.builtIn.events|Oui|Routage : latence des messages de messages/d’événements|Millisecondes|Average|Latence moyenne (en millisecondes) entre les entrées de messages vers IoT Hub et de télémétrie dans le point de terminaison intégré (messages/événements).|Aucune dimension|
|d2c.endpoints.latency.eventHubs|Oui|Routage : latence des messages d’Event Hub|Millisecondes|Average|Latence moyenne (en millisecondes) entre les entrées de messages vers IoT Hub et dans un point de terminaison Event Hub.|Aucune dimension|
|d2c.endpoints.latency.serviceBusQueues|Oui|Routage : latence des messages de la file d’attente Service Bus|Millisecondes|Average|Latence moyenne (en millisecondes) entre les entrées de messages vers IoT Hub et de télémétrie dans un point de terminaison de file d’attente Service Bus.|Aucune dimension|
|d2c.endpoints.latency.serviceBusTopics|Oui|Routage : latence des messages de la rubrique Service Bus|Millisecondes|Average|Latence moyenne (en millisecondes) entre les entrées de messages vers IoT Hub et de télémétrie dans un point de terminaison de rubrique Service Bus.|Aucune dimension|
|d2c.endpoints.latency.storage|Oui|Routage : latence des messages du stockage|Millisecondes|Average|Latence moyenne (en millisecondes) entre les entrées de messages vers IoT Hub et de télémétrie dans un point de terminaison de stockage.|Aucune dimension|
|d2c.telemetry.egress.dropped|Oui|Routage : messages de télémétrie annulés |Count|Total|Nombre de fois où des messages ont été annulés par le routage IoT Hub en raison de points de terminaison morts. Cette valeur ne compte pas les messages remis à un itinéraire de secours, car les messages annulés n’y sont pas remis.|Aucune dimension|
|d2c.telemetry.egress.fallback|Oui|Routage : messages remis à l’itinéraire de secours|Count|Total|Nombre de fois où le routage IoT Hub a remis des messages au point de terminaison associé à l’itinéraire de secours.|Aucune dimension|
|d2c.telemetry.egress.invalid|Oui|Routage : messages de télémétrie incompatibles|Count|Total|Nombre de fois où le routage IoT Hub n’a pas réussi à remettre des messages en raison d’une incompatibilité avec le point de terminaison. Cette valeur n’inclut pas les nouvelles tentatives.|Aucune dimension|
|d2c.telemetry.egress.orphaned|Oui|Routage : messages de télémétrie orphelins |Count|Total|Nombre de fois où des messages ont été définis comme orphelins par le routage IoT Hub car ils ne correspondaient à aucune règle de routage (y compris la règle de secours). |Aucune dimension|
|d2c.telemetry.egress.success|Oui|Routage : messages de télémétrie remis|Count|Total|Nombre de fois où des messages ont été correctement remis à tous les points de terminaison à l’aide du routage IoT Hub. Si un message est routé vers plusieurs points de terminaison, cette valeur augmente d’une unité pour chaque remise réussie. Si un message est routé plusieurs fois vers le même point de terminaison, cette valeur augmente d’une unité pour chaque remise réussie.|Aucune dimension|
|d2c.telemetry.ingress.allProtocol|Oui|Tentatives d’envoi de message de télémétrie|Count|Total|Nombre de tentatives d’envoi de messages de télémétrie appareil vers cloud à votre hub IoT|Aucune dimension|
|d2c.telemetry.ingress.sendThrottle|Oui|Nombre d’erreurs de limitation|Count|Total|Nombre d’erreurs de limitation causées par des limitations de débit d’appareil|Aucune dimension|
|d2c.telemetry.ingress.success|Oui|Messages de télémétrie envoyés|Count|Total|Nombre de messages de télémétrie appareil vers cloud envoyés avec succès à votre hub IoT|Aucune dimension|
|d2c.twin.read.failure|Oui|Lectures de représentations d’appareils en échec|Count|Total|Total des lectures de représentations en échec initiées par un appareil.|Aucune dimension|
|d2c.twin.read.size|Oui|Taille de la réponse des lectures de représentations des appareils|Octets|Average|Moyenne, minimum et maximum de toutes les lectures de représentations réussies initiées par un appareil.|Aucune dimension|
|d2c.twin.read.success|Oui|Lectures de représentations réussies d’appareils|Count|Total|Total des lectures de représentations réussies initiées par un appareil.|Aucune dimension|
|d2c.twin.update.failure|Oui|Mises à jour de représentations d’appareils en échec|Count|Total|Total des mises à jour de représentations en échec initiées par un appareil.|Aucune dimension|
|d2c.twin.update.size|Oui|Taille des mises à jour de représentations d’appareils|Octets|Average|Taille moyenne, minimale et maximale de toutes les mises à jour de représentations réussies initiées par un appareil.|Aucune dimension|
|d2c.twin.update.success|Oui|Mises à jour de représentations réussies d’appareils|Count|Total|Total des mises à jour de représentations réussies initiées par un appareil.|Aucune dimension|
|dailyMessageQuotaUsed|Oui|Nombre total de messages utilisés|Count|Maximale|Nombre total de messages utilisés aujourd'hui|Aucune dimension|
|deviceDataUsage|Oui|Utilisation totale des données d’appareil|Octets|Total|Nombre d’octets transférés vers et depuis tous les appareils connectés à IotHub|Aucune dimension|
|deviceDataUsageV2|Oui|Utilisation totale des données d’appareil (préversion)|Octets|Total|Nombre d’octets transférés vers et depuis tous les appareils connectés à IotHub|Aucune dimension|
|devices.connectedDevices.allProtocol|Oui|Appareils connectés (déconseillé) |Count|Total|Nombre d’appareils connectés à votre hub IoT|Aucune dimension|
|devices.totalDevices|Oui|Nombre total d’appareils (déconseillé)|Count|Total|Nombre d’appareils enregistrés sur votre hub IoT|Aucune dimension|
|EventGridDeliveries|Oui|Remises Event Grid (préversion)|Count|Total|Nombre d’événements IoT Hub publiés dans Event Grid. Utilisez la dimension Résultat pour le nombre de requêtes ayant réussi et ayant échoué. La dimension EventType représente le type de l’événement (https://aka.ms/ioteventgrid) ).|Résultat, Type d’événement|
|EventGridLatency|Oui|Latence d’Event Grid (préversion)|Millisecondes|Average|Latence moyenne (en millisecondes) entre le moment où l’événement Iot Hub a été généré et le moment où l’événement a été publié dans Event Grid. Ce nombre est une moyenne de tous les types d’événement. Utilisez la dimension Type d’événement pour afficher la latence d’un type d’événement spécifique.|Type d’événement|
|jobs.cancelJob.failure|Oui|Annulations de travaux en échec|Count|Total|Total des appels en échec pour annuler un travail.|Aucune dimension|
|jobs.cancelJob.success|Oui|Annulations de travaux réussies|Count|Total|Total des appels réussis pour annuler un travail.|Aucune dimension|
|jobs.completed|Oui|Travaux terminés|Count|Total|Total des travaux terminés.|Aucune dimension|
|jobs.createDirectMethodJob.failure|Oui|Créations des travaux d’appel de méthode en échec|Count|Total|Total des créations en échec des travaux d’appel de méthode directe.|Aucune dimension|
|jobs.createDirectMethodJob.success|Oui|Créations réussies des travaux d’appel de méthode|Count|Total|Total des créations réussies des travaux d’appel de méthode directe.|Aucune dimension|
|jobs.createTwinUpdateJob.failure|Oui|Créations des travaux de mises à jour de représentations en échec|Count|Total|Total des créations en échec des travaux de mises à jour de représentations.|Aucune dimension|
|jobs.createTwinUpdateJob.success|Oui|Créations réussies des travaux de mises à jour de représentations|Count|Total|Total des créations réussies de travaux de mises à jour de représentations.|Aucune dimension|
|jobs.failed|Oui|Travaux en échec|Count|Total|Total des travaux en échec.|Aucune dimension|
|jobs.listJobs.failure|Oui|Appels en échec pour répertorier les travaux|Count|Total|Total des appels en échec pour répertorier les travaux.|Aucune dimension|
|jobs.listJobs.success|Oui|Appels réussis pour répertorier les travaux|Count|Total|Total des appels réussis pour répertorier les travaux.|Aucune dimension|
|jobs.queryJobs.failure|Oui|Requêtes de travaux en échec|Count|Total|Total des appels en échec pour interroger les travaux.|Aucune dimension|
|jobs.queryJobs.success|Oui|Requêtes de travaux réussies|Count|Total|Total des appels réussis pour interroger les travaux.|Aucune dimension|
|RoutingDataSizeInBytesDelivered|Oui|Taille des messages de distribution de l’acheminement en octets (préversion)|Octets|Total|Taille totale, en octets, des messages remis par IoT Hub à un point de terminaison. Vous pouvez utiliser les dimensions EndpointName et EndpointType pour afficher la taille, en octets, des messages remis à vos différents points de terminaison. La valeur de la métrique augmente à chaque message remis, y compris à plusieurs points de terminaison ou plusieurs fois au même point de terminaison.|EndpointType, EndpointName, RoutingSource|
|RoutingDeliveries|Oui|Livraisons de routage (préversion)|Count|Total|Nombre de fois où IoT Hub a tenté de remettre des messages à tous les points de terminaison à l’aide du routage. Pour afficher le nombre de tentatives ayant abouti ou échoué, utilisez la dimension Résultat. Pour voir la raison de l’échec, comme non valide, supprimé ou orphelin, utilisez la dimension FailureReasonCategory. Vous pouvez également utiliser les dimensions EndpointName et EndpointType pour connaître le nombre de messages remis à vos différents points de terminaison. La valeur de métrique augmente d’une unité à chaque tentative de remise, notamment si le message est remis à plusieurs points de terminaison ou plusieurs fois au même point de terminaison.|EndpointType, EndpointName, FailureReasonCategory, Result, RoutingSource|
|RoutingDeliveryLatency|Oui|Latence de livraison du routage (préversion)|Millisecondes|Average|Latence moyenne (en millisecondes) entre les entrées de messages vers IoT Hub et de télémétrie dans un point de terminaison. Vous pouvez utiliser les dimensions EndpointName et EndpointType pour connaître la latence à vos différents points de terminaison.|EndpointType, EndpointName, RoutingSource|
|totalDeviceCount|Non|Nombre total d’appareils (préversion)|Count|Average|Nombre d’appareils enregistrés sur votre hub IoT|Aucune dimension|
|twinQueries.failure|Oui|Requêtes de représentations en échec|Count|Total|Total des requêtes de représentations en échec.|Aucune dimension|
|twinQueries.resultSize|Oui|Taille du résultat des requêtes de représentations|Octets|Average|Moyenne, minimum et maximum de la taille du résultat de toutes les requêtes de représentations réussies.|Aucune dimension|
|twinQueries.success|Oui|Requêtes de représentations réussies|Count|Total|Total des requêtes de représentations réussies.|Aucune dimension|


## <a name="microsoftdevicesprovisioningservices"></a>Microsoft.Devices/provisioningServices

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|AttestationAttempts|Oui|Tentatives d’attestation|Count|Total|Nombre d’attestations d’appareils tentées|ProvisioningServiceName, Status, Protocol|
|DeviceAssignments|Oui|Appareils attribués|Count|Total|Nombre d’appareils affectés à un hub IoT|ProvisioningServiceName, IotHubName|
|RegistrationAttempts|Oui|Tentatives d’enregistrement|Count|Total|Nombre d’inscriptions d’appareils tentées|ProvisioningServiceName, IotHubName, Status|


## <a name="microsoftdocumentdbdatabaseaccounts"></a>Microsoft.DocumentDB/databaseAccounts

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|AddRegion|Oui|Région ajoutée|Count|Count|Région ajoutée|Région|
|AutoscaleMaxThroughput|Non|Débit maximal de mise à l’échelle automatique|Count|Maximale|Débit maximal de mise à l’échelle automatique|DatabaseName, CollectionName|
|AvailableStorage|Non|(déconseillé) Stockage disponible|Octets|Total|« Stockage disponible » sera supprimé d’Azure Monitor à la fin du mois de septembre 2020. La taille de stockage de collection Cosmos DB est maintenant illimitée. La seule restriction est que la taille de stockage est de 20 Go par clé de partition logique. Vous pouvez activer PartitionKeyStatistics dans le journal de diagnostic pour connaître la consommation de stockage des principales clés de partition. Pour plus d’informations sur le quota de stockage Cosmos DB, consultez ce document : https://docs.microsoft.com/azure/cosmos-db/concepts-limits. À la date où la métrique sera déconseillée, les règles d’alerte qui seront encore définies dessus seront automatiquement désactivées.|CollectionName, DatabaseName, Region|
|CassandraConnectionClosures|Non|Fermetures de connexion Cassandra|Count|Total|Nombre de connexions de Cassandra fermées, signalées à une granularité d'une minute|Region, ClosureReason|
|CassandraConnectorAvgReplicationLatency|Non|Latence de réplication moyenne du connecteur Cassandra|Millisecondes|Average|Latence de réplication moyenne du connecteur Cassandra|Aucune dimension|
|CassandraConnectorReplicationHealthStatus|Non|État d’intégrité de la réplication du connecteur Cassandra|Count|Count|État d’intégrité de la réplication du connecteur Cassandra|NotStarted, ReplicationInProgress, Error|
|CassandraKeyspaceCreate|Non|Espace de clés Cassandra créé|Count|Count|Espace de clés Cassandra créé|ResourceName |
|CassandraKeyspaceDelete|Non|Espace de clés Cassandra supprimé|Count|Count|Espace de clés Cassandra supprimé|ResourceName |
|CassandraKeyspaceThroughputUpdate|Non|Débit d’espace de clés Cassandra mis à jour|Count|Count|Débit d’espace de clés Cassandra mis à jour|ResourceName |
|CassandraKeyspaceUpdate|Non|Espace de clés Cassandra mis à jour|Count|Count|Espace de clés Cassandra mis à jour|ResourceName |
|CassandraRequestCharges|Non|Frais de requête Cassandra|Count|Total|Unités de requête consommées pour les requêtes Cassandra effectuées|DatabaseName, CollectionName, Region, OperationType, ResourceType|
|CassandraRequests|Non|Requêtes Cassandra|Count|Count|Nombre de requêtes Cassandra effectuées|DatabaseName, CollectionName, Region, OperationType, ResourceType, ErrorCode|
|CassandraTableCreate|Non|Table Cassandra créée|Count|Count|Table Cassandra créée|ResourceName, ChildResourceName |
|CassandraTableDelete|Non|Table Cassandra supprimée|Count|Count|Table Cassandra supprimée|ResourceName, ChildResourceName |
|CassandraTableThroughputUpdate|Non|Débit de table Cassandra mis à jour|Count|Count|Débit de table Cassandra mis à jour|ResourceName, ChildResourceName |
|CassandraTableUpdate|Non|Table Cassandra mise à jour|Count|Count|Table Cassandra mise à jour|ResourceName, ChildResourceName |
|CreateAccount|Oui|Compte créé|Count|Count|Compte créé|Aucune dimension|
|DataUsage|Non|Utilisation des données|Octets|Total|Utilisation totale des données signalée à une granularité de 5 minutes|CollectionName, DatabaseName, Region|
|DeleteAccount|Oui|Compte supprimé|Count|Count|Compte supprimé|Aucune dimension|
|DocumentCount|Non|Nombre de documents|Count|Total|Nombre total de documents signalé à une granularité de 5 minutes|CollectionName, DatabaseName, Region|
|DocumentQuota|Non|Quota de document|Octets|Total|Quota de stockage total signalé à une granularité de 5 minutes|CollectionName, DatabaseName, Region|
|GremlinDatabaseCreate|Non|Base de données Gremlin créée|Count|Count|Base de données Gremlin créée|ResourceName |
|GremlinDatabaseDelete|Non|Base de données Gremlin supprimée|Count|Count|Base de données Gremlin supprimée|ResourceName |
|GremlinDatabaseThroughputUpdate|Non|Débit de base de données Gremlin mis à jour|Count|Count|Débit de base de données Gremlin mis à jour|ResourceName |
|GremlinDatabaseUpdate|Non|Base de données Gremlin mise à jour|Count|Count|Base de données Gremlin mise à jour|ResourceName |
|GremlinGraphCreate|Non|Graphique Gremlin créé|Count|Count|Graphique Gremlin créé|ResourceName, ChildResourceName |
|GremlinGraphDelete|Non|Graphique Gremlin supprimé|Count|Count|Graphique Gremlin supprimé|ResourceName, ChildResourceName |
|GremlinGraphThroughputUpdate|Non|Débit de graphique Gremlin mis à jour|Count|Count|Débit de graphique Gremlin mis à jour|ResourceName, ChildResourceName |
|GremlinGraphUpdate|Non|Graphique Gremlin mis à jour|Count|Count|Graphique Gremlin mis à jour|ResourceName, ChildResourceName |
|IndexUsage|Non|Utilisation d'index|Octets|Total|Utilisation d’index totale signalée à une granularité de 5 minutes|CollectionName, DatabaseName, Region|
|MetadataRequests|Non|Demandes de métadonnées|Count|Count|Nombre de demandes de métadonnées. Cosmos DB gère la collection des métadonnées système pour chaque compte, ce qui vous permet d’énumérer les collections, les bases de données, etc., ainsi que leur configuration, et ce gratuitement.|DatabaseName, CollectionName, Region, StatusCode, |
|MongoCollectionCreate|Non|Collection Mongo créée|Count|Count|Collection Mongo créée|ResourceName, ChildResourceName |
|MongoCollectionDelete|Non|Collection Mongo supprimée|Count|Count|Collection Mongo supprimée|ResourceName, ChildResourceName |
|MongoCollectionThroughputUpdate|Non|Débit de collection Mongo mis à jour|Count|Count|Débit de collection Mongo mis à jour|ResourceName, ChildResourceName |
|MongoCollectionUpdate|Non|Collection Mongo mise à jour|Count|Count|Collection Mongo mise à jour|ResourceName, ChildResourceName |
|MongoDBDatabaseUpdate|Non|Base de données Mongo supprimée|Count|Count|Base de données Mongo supprimée|ResourceName |
|MongoDatabaseThroughputUpdate|Non|Débit de base de données Mongo mis à jour|Count|Count|Débit de base de données Mongo mis à jour|ResourceName |
|MongoDBDatabaseCreate|Non|Base de données Mongo créée|Count|Count|Base de données Mongo créée|ResourceName |
|MongoDBDatabaseUpdate|Non|Base de données Mongo mise à jour|Count|Count|Base de données Mongo mise à jour|ResourceName |
|MongoRequestCharge|Oui|Frais des requêtes Mongo|Count|Total|Unités de requête Mongo consommées|DatabaseName, CollectionName, Region, CommandName, ErrorCode, Status|
|MongoRequests|Oui|Requêtes Mongo|Count|Count|Nombre de requêtes Mongo effectuées|DatabaseName, CollectionName, Region, CommandName, ErrorCode, Status|
|MongoRequestsCount|Non|Taux de demandes Mongo|CountPerSecond|Average|Nombre de demandes Mongo par seconde|DatabaseName, CollectionName, Region, ErrorCode|
|MongoRequestsDelete|Non|Taux de requête de suppression Mongo|CountPerSecond|Average|Demandes de suppression Mongo par seconde|DatabaseName, CollectionName, Region, ErrorCode|
|MongoRequestsInsert|Non|Taux de requête d’insertion Mongo|CountPerSecond|Average|Nombre d’insertions Mongo par seconde|DatabaseName, CollectionName, Region, ErrorCode|
|MongoRequestsQuery|Non|Taux de requêtes d’interrogation Mongo|CountPerSecond|Average|Demandes de requête Mongo par seconde|DatabaseName, CollectionName, Region, ErrorCode|
|MongoRequestsUpdate|Non|Taux de demande de mise à jour Mongo|CountPerSecond|Average|Demandes de mise à jour Mongo par seconde|DatabaseName, CollectionName, Region, ErrorCode|
|NormalizedRUConsumption|Non|Consommation d’unités de requête normalisée|Pourcentage|Maximale|Pourcentage maximal de consommation d’unités de requête par minute|CollectionName, DatabaseName, Region, PartitionKeyRangeId|
|ProvisionedThroughput|Non|Débit approvisionné|Count|Maximale|Débit approvisionné|DatabaseName, CollectionName|
|RegionFailover|Oui|Région basculée|Count|Count|Région basculée|Aucune dimension|
|RemoveRegion|Oui|Région supprimée|Count|Count|Région supprimée|Région|
|ReplicationLatency|Oui|Latence de réplication P99|Millisecondes|Average|Latence de réplication P99 des régions source et cible pour le compte géolocalisé|SourceRegion, TargetRegion|
|ServerSideLatency|Non|Latence côté serveur|Millisecondes|Average|Latence côté serveur|DatabaseName, CollectionName, Region, ConnectionMode, OperationType, PublicAPIType|
|ServiceAvailability|Non|Disponibilité des services|Pourcentage|Average|Disponibilité des requêtes de compte à une granularité d’une heure, d'un jour ou d'un mois|Aucune dimension|
|SqlContainerCreate|Non|Conteneur SQL créé|Count|Count|Conteneur SQL créé|ResourceName, ChildResourceName |
|SqlContainerDelete|Non|Conteneur SQL supprimé|Count|Count|Conteneur SQL supprimé|ResourceName, ChildResourceName |
|SqlContainerThroughputUpdate|Non|Débit de conteneur SQL mis à jour|Count|Count|Débit de conteneur SQL mis à jour|ResourceName, ChildResourceName |
|SqlContainerUpdate|Non|Conteneur SQL mis à jour|Count|Count|Conteneur SQL mis à jour|ResourceName, ChildResourceName |
|SqlDatabaseCreate|Non|Base de données SQL créée|Count|Count|Base de données SQL créée|ResourceName |
|SqlDatabaseDelete|Non|Base de données SQL supprimée|Count|Count|Base de données SQL supprimée|ResourceName |
|SqlDatabaseThroughputUpdate|Non|Débit de base de données SQL mis à jour|Count|Count|Débit de base de données SQL mis à jour|ResourceName |
|SqlDatabaseUpdate|Non|Base de données SQL mise à jour|Count|Count|Base de données SQL mise à jour|ResourceName |
|TableTableCreate|Non|Table AzureTable créée|Count|Count|Table AzureTable créée|ResourceName |
|TableTableDelete|Non|Table AzureTable supprimée|Count|Count|Table AzureTable supprimée|ResourceName |
|TableTableThroughputUpdate|Non|Débit de table AzureTable mis à jour|Count|Count|Débit de table AzureTable mis à jour|ResourceName |
|TableTableUpdate|Non|Table AzureTable mise à jour|Count|Count|Table AzureTable mise à jour|ResourceName |
|TotalRequests|Oui|Total de requêtes|Count|Count|Nombre de requêtes effectuées|DatabaseName, CollectionName, Region, StatusCode, OperationType, Status|
|TotalRequestUnits|Oui|Unités de requête totales|Count|Total|Unités de requête consommées|DatabaseName, CollectionName, Region, StatusCode, OperationType, Status|
|UpdateAccountKeys|Oui|Clés de compte mises à jour|Count|Count|Clés de compte mises à jour|KeyType|
|UpdateAccountNetworkSettings|Oui|Paramètres réseau de compte mis à jour|Count|Count|Paramètres réseau de compte mis à jour|Aucune dimension|
|UpdateAccountReplicationSettings|Oui|Paramètres de réplication de compte mis à jour|Count|Count|Paramètres de réplication de compte mis à jour|Aucune dimension|
|UpdateDiagnosticsSettings|Non|Paramètres de diagnostic du compte mis à jour|Count|Count|Paramètres de diagnostic du compte mis à jour|DiagnosticSettingsName, ResourceGroupName|


## <a name="microsofteventgriddomains"></a>Microsoft.EventGrid/domains

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|DeadLetteredCount|Oui|Événements de lettres mortes|Count|Total|Nombre total d’événements de lettres mortes correspondant à cet abonnement aux événements|Topic, EventSubscriptionName, DomainEventSubscriptionName, DeadLetterReason|
|DeliveryAttemptFailCount|Non|Événements d’échec de la remise|Count|Total|Nombre total d’événements ayant échoué dans la remise à cet abonnement aux événements|Topic, EventSubscriptionName, DomainEventSubscriptionName, Error, ErrorType|
|DeliverySuccessCount|Oui|Événements remis|Count|Total|Nombre total d’événements remis à cet abonnement aux événements|Topic, EventSubscriptionName, DomainEventSubscriptionName|
|DestinationProcessingDurationInMs|Non|Durée du traitement de la destination|Millisecondes|Average|Durée du traitement de la destination en millisecondes|Topic, EventSubscriptionName, DomainEventSubscriptionName|
|DroppedEventCount|Oui|Événements annulés|Count|Total|Nombre total d’événements annulés correspondant à cet abonnement aux événements|Topic, EventSubscriptionName, DomainEventSubscriptionName, DropReason|
|MatchedEventCount|Oui|Événements correspondants|Count|Total|Nombre total d’événements correspondant à cet abonnement aux événements|Topic, EventSubscriptionName, DomainEventSubscriptionName|
|PublishFailCount|Oui|Événements d'échec de la publication|Count|Total|Nombre total d’événements ayant échoué à publier dans cette rubrique|Topic, ErrorType, Error|
|PublishSuccessCount|Oui|Événements publiés|Count|Total|Nombre total d’événements publiés dans cette rubrique|Rubrique|
|PublishSuccessLatencyInMs|Oui|Latence de réussite de la publication|Millisecondes|Total|Latence de réussite de la publication en millisecondes|Aucune dimension|


## <a name="microsofteventgrideventsubscriptions"></a>Microsoft.EventGrid/eventSubscriptions

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|DeadLetteredCount|Oui|Événements de lettres mortes|Count|Total|Nombre total d’événements de lettres mortes correspondant à cet abonnement aux événements|DeadLetterReason|
|DeliveryAttemptFailCount|Non|Événements d’échec de la remise|Count|Total|Nombre total d’événements ayant échoué dans la remise à cet abonnement aux événements|Error, ErrorType|
|DeliverySuccessCount|Oui|Événements remis|Count|Total|Nombre total d’événements remis à cet abonnement aux événements|Aucune dimension|
|DestinationProcessingDurationInMs|Non|Durée du traitement de la destination|Millisecondes|Average|Durée du traitement de la destination en millisecondes|Aucune dimension|
|DroppedEventCount|Oui|Événements annulés|Count|Total|Nombre total d’événements annulés correspondant à cet abonnement aux événements|DropReason|
|MatchedEventCount|Oui|Événements correspondants|Count|Total|Nombre total d’événements correspondant à cet abonnement aux événements|Aucune dimension|


## <a name="microsofteventgridextensiontopics"></a>Microsoft.EventGrid/extensionTopics

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|PublishFailCount|Oui|Événements d'échec de la publication|Count|Total|Nombre total d’événements ayant échoué à publier dans cette rubrique|ErrorType, Error|
|PublishSuccessCount|Oui|Événements publiés|Count|Total|Nombre total d’événements publiés dans cette rubrique|Aucune dimension|
|PublishSuccessLatencyInMs|Oui|Latence de réussite de la publication|Millisecondes|Total|Latence de réussite de la publication en millisecondes|Aucune dimension|
|UnmatchedEventCount|Oui|Événements sans correspondance|Count|Total|Nombre total d’événements ne correspondant à aucun des abonnements aux événements pour cette rubrique|Aucune dimension|


## <a name="microsofteventgridsystemtopics"></a>Microsoft.EventGrid/systemTopics

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|DeadLetteredCount|Oui|Événements de lettres mortes|Count|Total|Nombre total d’événements de lettres mortes correspondant à cet abonnement aux événements|DeadLetterReason, EventSubscriptionName|
|DeliveryAttemptFailCount|Non|Événements d’échec de la remise|Count|Total|Nombre total d’événements ayant échoué dans la remise à cet abonnement aux événements|Error, ErrorType, EventSubscriptionName|
|DeliverySuccessCount|Oui|Événements remis|Count|Total|Nombre total d’événements remis à cet abonnement aux événements|EventSubscriptionName|
|DestinationProcessingDurationInMs|Non|Durée du traitement de la destination|Millisecondes|Average|Durée du traitement de la destination en millisecondes|EventSubscriptionName|
|DroppedEventCount|Oui|Événements annulés|Count|Total|Nombre total d’événements annulés correspondant à cet abonnement aux événements|DropReason, EventSubscriptionName|
|MatchedEventCount|Oui|Événements correspondants|Count|Total|Nombre total d’événements correspondant à cet abonnement aux événements|EventSubscriptionName|
|PublishFailCount|Oui|Événements d'échec de la publication|Count|Total|Nombre total d’événements ayant échoué à publier dans cette rubrique|ErrorType, Error|
|PublishSuccessCount|Oui|Événements publiés|Count|Total|Nombre total d’événements publiés dans cette rubrique|Aucune dimension|
|PublishSuccessLatencyInMs|Oui|Latence de réussite de la publication|Millisecondes|Total|Latence de réussite de la publication en millisecondes|Aucune dimension|
|UnmatchedEventCount|Oui|Événements sans correspondance|Count|Total|Nombre total d’événements ne correspondant à aucun des abonnements aux événements pour cette rubrique|Aucune dimension|


## <a name="microsofteventgridtopics"></a>Microsoft.EventGrid/topics

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|DeadLetteredCount|Oui|Événements de lettres mortes|Count|Total|Nombre total d’événements de lettres mortes correspondant à cet abonnement aux événements|DeadLetterReason, EventSubscriptionName|
|DeliveryAttemptFailCount|Non|Événements d’échec de la remise|Count|Total|Nombre total d’événements ayant échoué dans la remise à cet abonnement aux événements|Error, ErrorType, EventSubscriptionName|
|DeliverySuccessCount|Oui|Événements remis|Count|Total|Nombre total d’événements remis à cet abonnement aux événements|EventSubscriptionName|
|DestinationProcessingDurationInMs|Non|Durée du traitement de la destination|Millisecondes|Average|Durée du traitement de la destination en millisecondes|EventSubscriptionName|
|DroppedEventCount|Oui|Événements annulés|Count|Total|Nombre total d’événements annulés correspondant à cet abonnement aux événements|DropReason, EventSubscriptionName|
|MatchedEventCount|Oui|Événements correspondants|Count|Total|Nombre total d’événements correspondant à cet abonnement aux événements|EventSubscriptionName|
|PublishFailCount|Oui|Événements d'échec de la publication|Count|Total|Nombre total d’événements ayant échoué à publier dans cette rubrique|ErrorType, Error|
|PublishSuccessCount|Oui|Événements publiés|Count|Total|Nombre total d’événements publiés dans cette rubrique|Aucune dimension|
|PublishSuccessLatencyInMs|Oui|Latence de réussite de la publication|Millisecondes|Total|Latence de réussite de la publication en millisecondes|Aucune dimension|
|UnmatchedEventCount|Oui|Événements sans correspondance|Count|Total|Nombre total d’événements ne correspondant à aucun des abonnements aux événements pour cette rubrique|Aucune dimension|


## <a name="microsofteventhubclusters"></a>Microsoft.EventHub/clusters

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|ActiveConnections|Non|ActiveConnections|Count|Average|Nombre total de connexions actives pour Microsoft.EventHub.|Aucune dimension|
|AvailableMemory|Non|Mémoire disponible|Pourcentage|Maximale|Mémoire disponible pour le cluster Event Hub, en pourcentage de la mémoire totale.|Role|
|CaptureBacklog|Non|Backlog des captures.|Count|Total|Backlog des captures de Microsoft.EventHub.|Aucune dimension|
|CapturedBytes|Non|Octets capturés.|Octets|Total|Octets capturés pour Microsoft.EventHub.|Aucune dimension|
|CapturedMessages|Non|Messages capturés.|Count|Total|Messages capturés pour Microsoft.EventHub.|Aucune dimension|
|ConnectionsClosed|Non|Connexions fermées.|Count|Average|Connexions fermées pour Microsoft.EventHub.|Aucune dimension|
|ConnectionsOpened|Non|Connexions ouvertes.|Count|Average|Connexions ouvertes pour Microsoft.EventHub.|Aucune dimension|
|UC|Non|UC|Pourcentage|Maximale|Utilisation de l’UC en pourcentage du cluster Event Hub|Role|
|IncomingBytes|Oui|Octets entrants.|Octets|Total|Octets entrants pour Microsoft.EventHub.|Aucune dimension|
|IncomingMessages|Oui|Messages entrants|Count|Total|Messages entrants pour Microsoft.EventHub.|Aucune dimension|
|IncomingRequests|Oui|Demandes entrantes|Count|Total|Demandes entrantes pour Microsoft.EventHub.|Aucune dimension|
|OutgoingBytes|Oui|Octets sortants.|Octets|Total|Octets sortants pour Microsoft.EventHub.|Aucune dimension|
|OutgoingMessages|Oui|Messages sortants|Count|Total|Messages sortants pour Microsoft.EventHub.|Aucune dimension|
|QuotaExceededErrors|Non|Erreurs de dépassement de quota.|Count|Total|Erreurs de dépassement de quota pour Microsoft.EventHub.|Aucune dimension|
|ServerErrors|Non|Erreurs de serveur.|Count|Total|Erreurs de serveur pour Microsoft.EventHub.|Aucune dimension|
|Taille|Non|Taille|Octets|Average|Taille d’un EventHub en octets.|Role|
|SuccessfulRequests|Non|Requêtes ayant réussi|Count|Total|Requête réussies pour Microsoft.EventHub.|Aucune dimension|
|ThrottledRequests|Non|Demandes limitées.|Count|Total|Demandes limitées pour Microsoft.EventHub.|Aucune dimension|
|UserErrors|Non|Erreurs d’utilisateur.|Count|Total|Erreurs d’utilisateur pour Microsoft.EventHub.|Aucune dimension|


## <a name="microsofteventhubnamespaces"></a>Microsoft.EventHub/namespaces

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|ActiveConnections|Non|ActiveConnections|Count|Average|Nombre total de connexions actives pour Microsoft.EventHub.|Aucune dimension|
|CaptureBacklog|Non|Backlog des captures.|Count|Total|Backlog des captures de Microsoft.EventHub.|EntityName|
|CapturedBytes|Non|Octets capturés.|Octets|Total|Octets capturés pour Microsoft.EventHub.|EntityName|
|CapturedMessages|Non|Messages capturés.|Count|Total|Messages capturés pour Microsoft.EventHub.|EntityName|
|ConnectionsClosed|Non|Connexions fermées.|Count|Average|Connexions fermées pour Microsoft.EventHub.|EntityName|
|ConnectionsOpened|Non|Connexions ouvertes.|Count|Average|Connexions ouvertes pour Microsoft.EventHub.|EntityName|
|EHABL|Oui|Archiver les messages de backlog (déconseillé)|Count|Total|Messages archivés Event Hub dans le backlog pour un espace de noms (déconseillé)|Aucune dimension|
|EHAMBS|Oui|Archiver le débit des messages (déconseillé)|Octets|Total|Débit de messages archivés Event Hub dans un espace de noms (déconseillé)|Aucune dimension|
|EHAMSGS|Oui|Messages archivés (déconseillé)|Count|Total|Messages archivés Event Hub pour un espace de noms (déconseillé)|Aucune dimension|
|EHINBYTES|Oui|Octets entrants (déconseillé)|Octets|Total|Débit de messages entrants Event Hub pour un espace de noms (déconseillé)|Aucune dimension|
|EHINMBS|Oui|Octets entrants (obsolète) (déprécié)|Octets|Total|Débit de messages entrants Event Hub pour un espace de noms. Cette métrique est déconseillée. Utilisez plutôt la métrique Octets entrants (déconseillé)|Aucune dimension|
|EHINMSGS|Oui|Messages entrants (déconseillé)|Count|Total|Nombre total de messages entrants pour un espace de noms (déconseillé)|Aucune dimension|
|EHOUTBYTES|Oui|Octets sortants (déconseillé)|Octets|Total|Débit de messages sortants Event Hub pour un espace de noms (déconseillé)|Aucune dimension|
|EHOUTMBS|Oui|Octets sortants (obsolète) (déprécié)|Octets|Total|Débit de messages sortants Event Hub pour un espace de noms. Cette métrique est déconseillée. Utilisez plutôt la métrique Octets sortants (déconseillé)|Aucune dimension|
|EHOUTMSGS|Oui|Messages sortants (déconseillé)|Count|Total|Nombre total de messages sortants pour un espace de noms (déconseillé)|Aucune dimension|
|FAILREQ|Oui|Requêtes non réussies (déconseillé)|Count|Total|Nombre total de requêtes non réussies pour un espace de noms (déconseillé)|Aucune dimension|
|IncomingBytes|Oui|Octets entrants.|Octets|Total|Octets entrants pour Microsoft.EventHub.|EntityName|
|IncomingMessages|Oui|Messages entrants|Count|Total|Messages entrants pour Microsoft.EventHub.|EntityName|
|IncomingRequests|Oui|Demandes entrantes|Count|Total|Demandes entrantes pour Microsoft.EventHub.|EntityName|
|INMSGS|Oui|Messages entrants (obsolète) (déprécié)|Count|Total|Nombre total de messages entrants pour un espace de noms. Cette métrique est déconseillée. Utilisez plutôt la métrique Messages entrants (déconseillé)|Aucune dimension|
|INREQS|Oui|Requêtes entrantes (déconseillé)|Count|Total|Nombre total des requêtes d’envoi entrantes pour un espace de noms (déconseillé)|Aucune dimension|
|INTERR|Oui|Erreurs internes du serveur (déconseillé)|Count|Total|Nombre total d’erreurs internes du serveur pour un espace de noms (déconseillé)|Aucune dimension|
|MISCERR|Oui|Autres erreurs (déconseillé)|Count|Total|Nombre total de requêtes non réussies pour un espace de noms (déconseillé)|Aucune dimension|
|OutgoingBytes|Oui|Octets sortants.|Octets|Total|Octets sortants pour Microsoft.EventHub.|EntityName|
|OutgoingMessages|Oui|Messages sortants|Count|Total|Messages sortants pour Microsoft.EventHub.|EntityName|
|OUTMSGS|Oui|Messages sortants (obsolète) (déprécié)|Count|Total|Nombre total de messages sortants pour un espace de noms. Cette métrique est déconseillée. Utilisez plutôt la métrique Messages sortants (déconseillé)|Aucune dimension|
|QuotaExceededErrors|Non|Erreurs de dépassement de quota.|Count|Total|Erreurs de dépassement de quota pour Microsoft.EventHub.|EntityName, |
|ServerErrors|Non|Erreurs de serveur.|Count|Total|Erreurs de serveur pour Microsoft.EventHub.|EntityName, |
|Taille|Non|Taille|Octets|Average|Taille d’un EventHub en octets.|EntityName|
|SuccessfulRequests|Non|Requêtes ayant réussi|Count|Total|Requête réussies pour Microsoft.EventHub.|EntityName, |
|SUCCREQ|Oui|Requêtes réussies (déconseillé)|Count|Total|Nombre total de requêtes réussies pour un espace de noms (déconseillé)|Aucune dimension|
|SVRBSY|Oui|Erreurs de serveur occupé (déconseillé)|Count|Total|Nombre total d’erreurs de serveur occupé pour un espace de noms (déconseillé)|Aucune dimension|
|ThrottledRequests|Non|Demandes limitées.|Count|Total|Demandes limitées pour Microsoft.EventHub.|EntityName, |
|UserErrors|Non|Erreurs d’utilisateur.|Count|Total|Erreurs d’utilisateur pour Microsoft.EventHub.|EntityName, |


## <a name="microsofthdinsightclusters"></a>Microsoft.HDInsight/clusters

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|CategorizedGatewayRequests|Oui|Demandes de la passerelle par catégorie|Count|Total|Nombre de demandes de la passerelle par catégories (1xx/2xx/3xx/4xx/5xx)|httpStatus|
|GatewayRequests|Oui|Demandes de la passerelle|Count|Total|Nombre de demandes de la passerelle|httpStatus|
|NumActiveWorkers|Oui|Nombre de collaborateurs actifs|Count|Maximale|Nombre de collaborateurs actifs|MetricName|


## <a name="microsoftinsightsautoscalesettings"></a>Microsoft.Insights/AutoscaleSettings

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|MetricThreshold|Oui|Seuil de métrique|Count|Average|Seuil de mise à l’échelle automatique configurée lors de l’exécution de la mise à l’échelle automatique.|MetricTriggerRule|
|ObservedCapacity|Oui|Capacité observée|Count|Average|Capacité envoyée à la mise à l’échelle automatique lors de l’exécution.|Aucune dimension|
|ObservedMetricValue|Oui|Valeur de métrique observée|Count|Average|Valeur calculée par mise à l’échelle automatique lors de l’exécution|MetricTriggerSource|
|ScaleActionsInitiated|Oui|Actions de mise à l’échelle initiées|Count|Total|Direction de l’opération de mise à l’échelle.|ScaleDirection|


## <a name="microsoftinsightscomponents"></a>Microsoft.Insights/Components

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|availabilityResults/availabilityPercentage|Oui|Disponibilité|Pourcentage|Average|Pourcentage de tests de disponibilité terminés|availabilityResult/name, availabilityResult/location|
|availabilityResults/count|Non|Tests de disponibilité|Count|Count|Nombre de tests de disponibilité|availabilityResult/name, availabilityResult/location, availabilityResult/success|
|availabilityResults/duration|Oui|Durée du test de disponibilité|Millisecondes|Average|Durée du test de disponibilité|availabilityResult/name, availabilityResult/location, availabilityResult/success|
|browserTimings/networkDuration|Oui|Temps de connexion au réseau pour le chargement de page|Millisecondes|Average|Temps écoulé entre la requête de l’utilisateur et la connexion réseau. Inclut la connexion de recherche DNS et de transport.|Aucune dimension|
|browserTimings/processingDuration|Oui|Temps de traitement du client|Millisecondes|Average|Temps écoulé entre la réception du dernier octet d’un document et le chargement du modèle DOM. Les demandes asynchrones peuvent encore être en cours de traitement.|Aucune dimension|
|browserTimings/receiveDuration|Oui|Temps de réception de réponse|Millisecondes|Average|Temps écoulé entre le premier et le dernier octet, ou jusqu’à la déconnexion.|Aucune dimension|
|browserTimings/sendDuration|Oui|Temps d’envoi de demande|Millisecondes|Average|Temps écoulé entre la connexion réseau et la réception du premier octet.|Aucune dimension|
|browserTimings/totalDuration|Oui|Temps de chargement de la page de navigateur|Millisecondes|Average|Temps s’écoulant entre la demande de l’utilisateur et le chargement du DOM, des feuilles de style, des scripts et des images.|Aucune dimension|
|dependencies/count|Non|Appels de dépendance|Count|Count|Nombre d’appels effectués par l’application à des ressources externes.|dependency/type, dependency/performanceBucket, dependency/success, dependency/target, dependency/resultCode, operation/synthetic, cloud/roleInstance, cloud/roleName|
|dependencies/duration|Oui|Durée de la dépendance|Millisecondes|Average|Durée des appels effectués par l’application aux ressources externes.|dependency/type, dependency/performanceBucket, dependency/success, dependency/target, dependency/resultCode, operation/synthetic, cloud/roleInstance, cloud/roleName|
|dependencies/failed|Non|Échecs d'appel de dépendance|Count|Count|Nombre d’échecs des appels de dépendance effectués par l’application à des ressources externes.|dependency/type, dependency/performanceBucket, dependency/target, dependency/resultCode, operation/synthetic, cloud/roleInstance, cloud/roleName|
|exceptions/browser|Non|Exceptions du navigateur|Count|Count|Nombre d’exceptions non interceptées levées dans le navigateur.|cloud/roleName|
|exceptions/count|Oui|Exceptions|Count|Count|Nombre combiné de toutes les exceptions non interceptées.|cloud/roleName, cloud/roleInstance, client/type|
|exceptions/server|Non|Exceptions de serveur|Count|Count|Nombre d’exceptions non interceptées levées dans l’application serveur.|cloud/roleName, cloud/roleInstance|
|pageViews/count|Oui|Affichages de page|Count|Count|Nombre de pages consultées.|operation/synthetic, cloud/roleName|
|pageViews/duration|Oui|Temps de chargement de la page consultée|Millisecondes|Average|Temps de chargement de la page consultée|operation/synthetic, cloud/roleName|
|performanceCounters/exceptionsPerSecond|Oui|Taux d’exceptions|CountPerSecond|Average|Nombre d’exceptions prises en charge et non prises en charge signalées à Windows, notamment les exceptions .NET et les exceptions non managées converties en exceptions .NET.|cloud/roleInstance|
|performanceCounters/memoryAvailableBytes|Oui|Mémoire disponible|Octets|Average|Mémoire physique prête à être allouée immédiatement pour un processus ou pour le système.|cloud/roleInstance|
|performanceCounters/processCpuPercentage|Oui|Processeur de processus|Pourcentage|Average|Temps écoulé en pourcentage que tous les threads de processus a passé à utiliser le processeur pour exécuter des instructions. Il peut varier de 0 à 100. Cette métrique indique les performances du processus w3wp uniquement.|cloud/roleInstance|
|performanceCounters/processIOBytesPerSecond|Oui|Taux d’E/S du processus|BytesPerSecond|Average|Nombre total d’octets par seconde lus et écrits sur des fichiers, un réseau et des appareils.|cloud/roleInstance|
|performanceCounters/processorCpuPercentage|Oui|Temps processeur|Pourcentage|Average|Temps en pourcentage que le processus a passé dans des threads actifs.|cloud/roleInstance|
|performanceCounters/processPrivateBytes|Oui|Octets privés du processus|Octets|Average|Mémoire allouée exclusivement aux processus de l’application analysée.|cloud/roleInstance|
|performanceCounters/requestExecutionTime|Oui|Durée d’exécution de la requête HTTP|Millisecondes|Average|Durée d’exécution de la demande la plus récente.|cloud/roleInstance|
|performanceCounters/requestsInQueue|Oui|Requêtes HTTP dans la file d’attente de l’application|Count|Average|Longueur de la file d'attente des demandes d'application.|cloud/roleInstance|
|performanceCounters/requestsPerSecond|Oui|Taux de requêtes HTTP|CountPerSecond|Average|Taux par seconde de l'ensemble des demandes à l'application provenant d'ASP.NET.|cloud/roleInstance|
|requests/count|Non|Requêtes serveur|Count|Count|Nombre de requêtes HTTP terminées.|request/performanceBucket, request/resultCode, operation/synthetic, cloud/roleInstance, request/success, cloud/roleName|
|requests/duration|Oui|Temps de réponse du serveur|Millisecondes|Average|Temps écoulé entre la réception d'une requête HTTP et la fin de l'envoi de la réponse.|request/performanceBucket, request/resultCode, operation/synthetic, cloud/roleInstance, request/success, cloud/roleName|
|requests/failed|Non|Demandes ayant échoué|Count|Count|Nombre de requêtes HTTP marquées comme ayant échoué. Dans la plupart des cas, il s’agit de requêtes avec un code de réponse >= 400 et différent de 401.|request/performanceBucket, request/resultCode, operation/synthetic, cloud/roleInstance, cloud/roleName|
|requests/taux|Non|Taux de requêtes du serveur|CountPerSecond|Average|Taux de requêtes du serveur par seconde|request/performanceBucket, request/resultCode, operation/synthetic, cloud/roleInstance, request/success, cloud/roleName|
|traces/count|Oui|Traces|Count|Count|Nombre de documents de traces|trace/severityLevel, operation/synthetic, cloud/roleName, cloud/roleInstance|

## <a name="microsoftiotcentraliotapps"></a>Microsoft.IoTCentral/IoTApps

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|c2d.property.read.failure|Oui|Lectures des propriétés d’appareil ayant échoué à partir d’IoT Central|Count|Total|Nombre total de lectures des propriétés lancées à partir d’IoT Central ayant échoué|Aucune dimension|
|c2d.property.read.success|Oui|Lectures des propriétés d’appareil ayant abouti à partir d’IoT Central|Count|Total|Nombre total de lectures des propriétés lancées à partir d’IoT Central ayant abouti|Aucune dimension|
|c2d.property.update.failure|Oui|Mises à jour des propriétés d’appareil ayant échoué à partir d’IoT Central|Count|Total|Nombre total de mises à jour des propriétés lancées à partir d’IoT Central ayant échoué|Aucune dimension|
|c2d.property.update.success|Oui|Mises à jour des propriétés d’appareil ayant abouti à partir d’IoT Central|Count|Total|Nombre total de mises à jour des propriétés lancées à partir d’IoT Central ayant abouti|Aucune dimension|
|connectedDeviceCount|Non|Nombre total d’appareils connectés|Count|Average|Nombre d’appareils connectés à IoT Central|Aucune dimension|
|d2c.property.read.failure|Oui|Lectures des propriétés d’appareil ayant échoué à partir d’appareils|Count|Total|Nombre total de lectures des propriétés lancées à partir d’appareils ayant échoué|Aucune dimension|
|d2c.property.read.success|Oui|Lectures des propriétés d’appareil ayant abouti à partir d’appareils|Count|Total|Nombre total de lectures des propriétés lancées à partir d’appareils ayant abouti|Aucune dimension|
|d2c.property.update.failure|Oui|Mises à jour des propriétés d’appareil ayant échoué à partir d’appareils|Count|Total|Nombre total de mises à jour des propriétés lancées à partir d’appareils ayant échoué|Aucune dimension|
|d2c.property.update.success|Oui|Mises à jour des propriétés d’appareil ayant abouti à partir d’appareils|Count|Total|Nombre total de mises à jour des propriétés lancées à partir d’appareils ayant abouti|Aucune dimension|


## <a name="microsoftkeyvaultvaults"></a>Microsoft.KeyVault/vaults

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|Disponibilité|Oui|Disponibilité globale du coffre|Pourcentage|Average|Disponibilité des demandes de coffre|ActivityType, ActivityName, StatusCode, StatusCodeClass|
|SaturationShoebox|Non|Saturation globale du coffre|Pourcentage|Average|Capacité de coffre utilisée|ActivityType, ActivityName, TransactionType|
|ServiceApiHit|Oui|Correspondances totales de l'API de service|Count|Count|Nombre total de correspondances de l'API de service|ActivityType, ActivityName|
|ServiceApiLatency|Oui|Latence globale de l'API de service|Millisecondes|Average|Latence globale des demandes de l'API de service|ActivityType, ActivityName, StatusCode, StatusCodeClass|
|ServiceApiResult|Oui|Résultats totaux de l'API de service|Count|Count|Nombre total de résultats de l'API de service|ActivityType, ActivityName, StatusCode, StatusCodeClass|


## <a name="microsoftkustoclusters"></a>Microsoft.Kusto/Clusters

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|BatchBlobCount|Oui|Nombre d’objets blob du lot|Count|Average|Nombre de sources de données d’un lot agrégé pour l’ingestion|Base de données|
|BatchDuration|Oui|Durée du lot|Secondes|Average|Durée de la phase d’agrégation du flux d’ingestion|Base de données|
|BatchesProcessed|Oui|Lots traités|Count|Average|Nombre de lots agrégés pour l’ingestion. Motif de la fin du lot : si le lot a atteint la limite du temps de traitement par lot, de taille des données ou de nombre de fichiers définie par la stratégie de traitement par lot|Database, SealReason|
|BatchSize|Oui|Taille du lot|Octets|Average|Taille de données attendue non compressée dans un lot agrégé pour l’ingestion|Base de données|
|CacheUtilization|Oui|Utilisation du cache|Pourcentage|Average|Niveau d’utilisation dans l’étendue du cluster|Aucune dimension|
|ContinuousExportMaxLatenessMinutes|Oui|Retard max. pour l’exportation continue|Count|Maximale|Retard (en minutes) signalé par les travaux d’exportation continue dans le cluster|Aucune dimension|
|ContinuousExportNumOfRecordsExported|Oui|Exportation continue : nombre d’enregistrements exportés|Count|Total|Nombre d’enregistrements exportés, déclenchés pour chaque artefact de stockage écrit pendant l’opération d’exportation|ContinuousExportName, Database|
|ContinuousExportPendingCount|Oui|Nombre en attente d’exportations continues|Count|Maximale|Nombre de travaux d’exportation continue en attente prêts pour l’exécution|Aucune dimension|
|ContinuousExportResult|Oui|Résultat de l’exportation continue|Count|Count|Indique si l’exportation continue a réussi ou échoué|ContinuousExportName, Result, Database|
|UC|Oui|UC|Pourcentage|Average|Niveau d’utilisation de l’UC|Aucune dimension|
|EventsProcessedForEventHubs|Oui|Événements traités (pour Event/IoT Hubs)|Count|Total|Nombre d’événements traités par le cluster lors de l’ingestion à partir d’Event/IoT Hub|EventStatus|
|ExportUtilization|Oui|Utilisation de l’exportation|Pourcentage|Maximale|Utilisation de l’exportation|Aucune dimension|
|IngestionLatencyInSeconds|Oui|Latence d’ingestion (en secondes)|Secondes|Average|Durée d’ingestion entre la source (par ex., message dans Event Hub) et le cluster en secondes|Aucune dimension|
|IngestionResult|Oui|Résultat de l’ingestion|Count|Count|Nombre d’opérations d’ingestion|IngestionResultDetails|
|IngestionUtilization|Oui|Utilisation de l’ingestion|Pourcentage|Average|Ratio d’emplacements d’ingestion utilisés dans le cluster|Aucune dimension|
|IngestionVolumeInMB|Oui|Volume d’ingestion (en Mo)|Count|Total|Volume total de données ingérées dans le cluster (en Mo)|Aucune dimension|
|InstanceCount|Oui|Nombre d'instances|Count|Average|Nombre total d’instances|Aucune dimension|
|KeepAlive|Oui|Keep alive|Count|Average|Contrôle d’intégrité indiquant que le cluster répond aux requêtes|Aucune dimension|
|QueryDuration|Oui|Durée de la requête|Millisecondes|Average|Durée des requêtes en secondes|QueryStatus|
|SteamingIngestRequestRate|Oui|Taux de demandes d’ingestion en streaming|Count|RateRequestsPerSecond|Taux de demandes d’ingestion en streaming (Mo par seconde)|Aucune dimension|
|StreamingIngestDataRate|Oui|Taux de données d’ingestion en streaming|Count|Average|Taux de données d’ingestion en streaming (Mo par seconde)|Aucune dimension|
|StreamingIngestDuration|Oui|Durée de l’ingestion en streaming|Millisecondes|Average|Durée de l’ingestion en streaming en millisecondes|Aucune dimension|
|StreamingIngestResults|Oui|Résultat de l’ingestion en streaming|Count|Average|Résultat de l’ingestion en streaming|Résultats|
|TotalNumberOfConcurrentQueries|Oui|Nombre total de demandes simultanées|Count|Total|Nombre total de demandes simultanées|Aucune dimension|
|TotalNumberOfExtents|Oui|Nombre total d’étendues|Count|Total|Nombre total d’étendues de données|Aucune dimension|
|TotalNumberOfThrottledCommands|Oui|Nombre total de commandes limitées|Count|Total|Nombre total de commandes limitées|CommandType|
|TotalNumberOfThrottledQueries|Oui|Nombre total de demandes limitées|Count|Total|Nombre total de demandes limitées|Aucune dimension|


## <a name="microsoftlogicintegrationserviceenvironments"></a>Microsoft.Logic/integrationServiceEnvironments

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|ActionLatency|Oui|Latence de l’action |Secondes|Average|Latence des actions de flux de travail terminés.|Aucune dimension|
|ActionsCompleted|Oui|Actions terminées |Count|Total|Nombre d’actions de flux de travail terminées.|Aucune dimension|
|ActionsFailed|Oui|Actions ayant échoué |Count|Total|Nombre d’actions de flux de travail ayant échoué.|Aucune dimension|
|ActionsSkipped|Oui|Actions ignorées |Count|Total|Nombre d’actions de flux de travail ignorées.|Aucune dimension|
|ActionsStarted|Oui|Actions démarrées |Count|Total|Nombre d’actions de flux de travail démarrées.|Aucune dimension|
|ActionsSucceeded|Oui|Actions ayant réussi |Count|Total|Nombre d’actions de flux de travail ayant réussi.|Aucune dimension|
|ActionSuccessLatency|Oui|Latence de réussite d’action |Secondes|Average|Latence des actions de flux de travail ayant réussi.|Aucune dimension|
|ActionThrottledEvents|Oui|Événements d’action limitée|Count|Total|Nombre d’événements limités d’action de flux de travail.|Aucune dimension|
|IntegrationServiceEnvironmentConnectorMemoryUsage|Oui|Utilisation de la mémoire du connecteur pour l’environnement de service d’intégration|Pourcentage|Average|Utilisation de la mémoire du connecteur pour l’environnement de service d’intégration.|Aucune dimension|
|IntegrationServiceEnvironmentConnectorProcessorUsage|Oui|Utilisation du processeur du connecteur pour l’environnement de service d’intégration|Pourcentage|Average|Utilisation du processeur du connecteur pour l’environnement de service d’intégration.|Aucune dimension|
|IntegrationServiceEnvironmentWorkflowMemoryUsage|Oui|Utilisation de la mémoire de flux de travail pour l’environnement de service d’intégration|Pourcentage|Average|Utilisation de la mémoire de flux de travail pour l’environnement de service d’intégration.|Aucune dimension|
|IntegrationServiceEnvironmentWorkflowProcessorUsage|Oui|Utilisation du processeur de flux de travail pour l’environnement de service d’intégration|Pourcentage|Average|Utilisation du processeur de flux de travail pour l’environnement de service d’intégration.|Aucune dimension|
|RunFailurePercentage|Oui|Pourcentage d’échec des exécutions|Pourcentage|Total|Pourcentage d’exécutions de flux de travail ayant échoué.|Aucune dimension|
|RunLatency|Oui|Latence d’exécution|Secondes|Average|Latence des exécutions de flux de travail terminées.|Aucune dimension|
|RunsCancelled|Oui|Exécutions annulées|Count|Total|Nombre d’exécutions de flux de travail annulées.|Aucune dimension|
|RunsCompleted|Oui|Exécutions terminées|Count|Total|Nombre d’exécutions de flux de travail terminées.|Aucune dimension|
|RunsFailed|Oui|Exécutions ayant échoué|Count|Total|Nombre d’exécutions de flux de travail ayant échoué.|Aucune dimension|
|RunsStarted|Oui|Exécutions démarrées|Count|Total|Nombre d’exécutions de flux de travail démarrées.|Aucune dimension|
|RunsSucceeded|Oui|Exécutions réussies|Count|Total|Nombre d’exécutions de flux de travail ayant réussi.|Aucune dimension|
|RunStartThrottledEvents|Oui|Événements limités démarrés par l’exécution|Count|Total|Nombre d’événements limités de démarrage de l’exécution de flux de travail.|Aucune dimension|
|RunSuccessLatency|Oui|Latence d’exécution ayant réussi|Secondes|Average|Latence des exécutions de flux de travail ayant réussi.|Aucune dimension|
|RunThrottledEvents|Oui|Exécuter événements limités|Count|Total|Nombre d’actions de flux de travail ou d’événements limités par déclencheur.|Aucune dimension|
|TriggerFireLatency|Oui|Latence d’activation de déclencheur |Secondes|Average|Latence des déclencheurs de flux de travail activés.|Aucune dimension|
|TriggerLatency|Oui|Latence de déclencheur |Secondes|Average|Latence des déclenchements de flux de travail terminés.|Aucune dimension|
|TriggersCompleted|Oui|Déclenchements terminés |Count|Total|Nombre de déclencheurs de flux de travail terminés.|Aucune dimension|
|TriggersFailed|Oui|Déclenchements ayant échoué |Count|Total|Nombre de déclencheurs de flux de travail ayant échoué.|Aucune dimension|
|TriggersFired|Oui|Déclenchements activés |Count|Total|Nombre de déclencheurs de flux de travail activés.|Aucune dimension|
|TriggersSkipped|Oui|Déclenchements ignorés|Count|Total|Nombre de déclencheurs de flux de travail ignorés.|Aucune dimension|
|TriggersStarted|Oui|Déclenchements lancés |Count|Total|Nombre de déclencheurs de flux de travail démarrés.|Aucune dimension|
|TriggersSucceeded|Oui|Déclenchements ayant réussi |Count|Total|Nombre de déclencheurs de flux de travail ayant réussi.|Aucune dimension|
|TriggerSuccessLatency|Oui|Latence déclencheur ayant réussi |Secondes|Average|Latence des déclencheurs de flux de travail ayant réussi.|Aucune dimension|
|TriggerThrottledEvents|Oui|Événements limités par déclencheur|Count|Total|Nombre d’événements de flux de travail limités par déclencheur.|Aucune dimension|


## <a name="microsoftlogicworkflows"></a>Microsoft.Logic/workflows

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|ActionLatency|Oui|Latence de l’action |Secondes|Average|Latence des actions de flux de travail terminés.|Aucune dimension|
|ActionsCompleted|Oui|Actions terminées |Count|Total|Nombre d’actions de flux de travail terminées.|Aucune dimension|
|ActionsFailed|Oui|Actions ayant échoué |Count|Total|Nombre d’actions de flux de travail ayant échoué.|Aucune dimension|
|ActionsSkipped|Oui|Actions ignorées |Count|Total|Nombre d’actions de flux de travail ignorées.|Aucune dimension|
|ActionsStarted|Oui|Actions démarrées |Count|Total|Nombre d’actions de flux de travail démarrées.|Aucune dimension|
|ActionsSucceeded|Oui|Actions ayant réussi |Count|Total|Nombre d’actions de flux de travail ayant réussi.|Aucune dimension|
|ActionSuccessLatency|Oui|Latence de réussite d’action |Secondes|Average|Latence des actions de flux de travail ayant réussi.|Aucune dimension|
|ActionThrottledEvents|Oui|Événements d’action limitée|Count|Total|Nombre d’événements limités d’action de flux de travail.|Aucune dimension|
|BillableActionExecutions|Oui|Exécutions d’actions facturables|Count|Total|Nombre d’exécutions d’actions de flux de travail facturées.|Aucune dimension|
|BillableTriggerExecutions|Oui|Exécutions de déclencheurs facturables|Count|Total|Nombre d’exécutions de déclencheurs de flux de travail facturées.|Aucune dimension|
|BillingUsageNativeOperation|Oui|Utilisation de la facturation pour les exécutions d’opérations natives|Count|Total|Nombre d’exécutions d’opérations natives facturées.|Aucune dimension|
|BillingUsageStandardConnector|Oui|Utilisation de la facturation pour les exécutions de connecteurs Standard|Count|Total|Nombre d’exécutions de connecteurs Standard facturées.|Aucune dimension|
|BillingUsageStorageConsumption|Oui|Utilisation de la facturation pour les exécutions de consommation du stockage|Count|Total|Nombre d’exécutions de consommation du stockage facturées.|Aucune dimension|
|RunFailurePercentage|Oui|Pourcentage d’échec des exécutions|Pourcentage|Total|Pourcentage d’exécutions de flux de travail ayant échoué.|Aucune dimension|
|RunLatency|Oui|Latence d’exécution|Secondes|Average|Latence des exécutions de flux de travail terminées.|Aucune dimension|
|RunsCancelled|Oui|Exécutions annulées|Count|Total|Nombre d’exécutions de flux de travail annulées.|Aucune dimension|
|RunsCompleted|Oui|Exécutions terminées|Count|Total|Nombre d’exécutions de flux de travail terminées.|Aucune dimension|
|RunsFailed|Oui|Exécutions ayant échoué|Count|Total|Nombre d’exécutions de flux de travail ayant échoué.|Aucune dimension|
|RunsStarted|Oui|Exécutions démarrées|Count|Total|Nombre d’exécutions de flux de travail démarrées.|Aucune dimension|
|RunsSucceeded|Oui|Exécutions réussies|Count|Total|Nombre d’exécutions de flux de travail ayant réussi.|Aucune dimension|
|RunStartThrottledEvents|Oui|Événements limités démarrés par l’exécution|Count|Total|Nombre d’événements limités de démarrage de l’exécution de flux de travail.|Aucune dimension|
|RunSuccessLatency|Oui|Latence d’exécution ayant réussi|Secondes|Average|Latence des exécutions de flux de travail ayant réussi.|Aucune dimension|
|RunThrottledEvents|Oui|Exécuter événements limités|Count|Total|Nombre d’actions de flux de travail ou d’événements limités par déclencheur.|Aucune dimension|
|TotalBillableExecutions|Oui|Nombre total d’exécutions facturables|Count|Total|Nombre d’exécutions de flux de travail facturées.|Aucune dimension|
|TriggerFireLatency|Oui|Latence d’activation de déclencheur |Secondes|Average|Latence des déclencheurs de flux de travail activés.|Aucune dimension|
|TriggerLatency|Oui|Latence de déclencheur |Secondes|Average|Latence des déclenchements de flux de travail terminés.|Aucune dimension|
|TriggersCompleted|Oui|Déclenchements terminés |Count|Total|Nombre de déclencheurs de flux de travail terminés.|Aucune dimension|
|TriggersFailed|Oui|Déclenchements ayant échoué |Count|Total|Nombre de déclencheurs de flux de travail ayant échoué.|Aucune dimension|
|TriggersFired|Oui|Déclenchements activés |Count|Total|Nombre de déclencheurs de flux de travail activés.|Aucune dimension|
|TriggersSkipped|Oui|Déclenchements ignorés|Count|Total|Nombre de déclencheurs de flux de travail ignorés.|Aucune dimension|
|TriggersStarted|Oui|Déclenchements lancés |Count|Total|Nombre de déclencheurs de flux de travail démarrés.|Aucune dimension|
|TriggersSucceeded|Oui|Déclenchements ayant réussi |Count|Total|Nombre de déclencheurs de flux de travail ayant réussi.|Aucune dimension|
|TriggerSuccessLatency|Oui|Latence déclencheur ayant réussi |Secondes|Average|Latence des déclencheurs de flux de travail ayant réussi.|Aucune dimension|
|TriggerThrottledEvents|Oui|Événements limités par déclencheur|Count|Total|Nombre d’événements de flux de travail limités par déclencheur.|Aucune dimension|


## <a name="microsoftmachinelearningservicesworkspaces"></a>Microsoft.MachineLearningServices/workspaces

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|Cœurs actifs|Oui|Cœurs actifs|Count|Average|Nombre de cœurs actifs|Scenario, ClusterName|
|Nœuds actifs|Oui|Nœuds actifs|Count|Average|Nombre de nœuds actifs. Ce sont les nœuds qui exécutent activement un travail.|Scenario, ClusterName|
|Annuler les exécutions demandées|Oui|Annuler les exécutions demandées|Count|Total|Nombre d’exécutions pour lesquelles l’annulation a été demandée dans cet espace de travail. Le nombre est mis à jour à la réception d’une demande d’annulation pour une exécution.|Scenario, RunType, PublishedPipelineId, ComputeType, PipelineStepType|
|Exécutions annulées|Oui|Exécutions annulées|Count|Total|Nombre d’exécutions annulées pour cet espace de travail. Le nombre est mis à jour lorsque l’annulation d’une exécution aboutit.|Scenario, RunType, PublishedPipelineId, ComputeType, PipelineStepType|
|Exécutions terminées|Oui|Exécutions terminées|Count|Total|Nombre d’exécutions réussies pour cet espace de travail. Le nombre est mis à jour lorsqu’une exécution est terminée et que la sortie a été collectée.|Scenario, RunType, PublishedPipelineId, ComputeType, PipelineStepType|
|CpuUtilization|Oui|CpuUtilization|Count|Average|Pourcentage d’utilisation de la mémoire sur un nœud de processeur. L’utilisation est rapportée à intervalles d’une minute.|Scenario, runId, NodeId, ClusterName|
|Erreurs|Oui|Erreurs|Count|Total|Nombre d’erreurs d’exécution dans cet espace de travail. Le nombre est mis à jour chaque fois que l’exécution rencontre une erreur.|Scénario|
|Exécutions échouées|Oui|Exécutions échouées|Count|Total|Nombre d’exécutions en échec pour cet espace de travail. Le nombre est mis à jour lorsqu’une exécution échoue.|Scenario, RunType, PublishedPipelineId, ComputeType, PipelineStepType|
|Exécutions en cours de finalisation|Oui|Exécutions en cours de finalisation|Count|Total|Nombre d’exécutions entrées en état de finalisation pour cet espace de travail. Le nombre est mis à jour lorsqu’une exécution est terminée, mais que la collecte de sortie est toujours en cours.|Scenario, RunType, PublishedPipelineId, ComputeType, PipelineStepType|
|GpuUtilization|Oui|GpuUtilization|Count|Average|Pourcentage d’utilisation de la mémoire sur un nœud de GPU. L’utilisation est rapportée à intervalles d’une minute.|Scenario, runId, NodeId, DeviceId, ClusterName|
|Cœurs inactifs|Oui|Cœurs inactifs|Count|Average|Nombre de cœurs inactifs|Scenario, ClusterName|
|Nœuds inactifs|Oui|Nœuds inactifs|Count|Average|Nombre de nœuds inactifs. Les nœuds inactifs sont les nœuds qui n’exécutent aucun travail, mais qui peuvent accepter un nouveau travail, le cas échéant.|Scenario, ClusterName|
|Cœurs sortants|Oui|Cœurs sortants|Count|Average|Nombre de cœurs sortants|Scenario, ClusterName|
|Nœuds sortants|Oui|Nœuds sortants|Count|Average|Nombre de nœuds sortants. Les nœuds sortants sont les nœuds qui viennent de terminer le traitement d’un travail et qui passent à l’état inactif.|Scenario, ClusterName|
|Échec du déploiement de modèle|Oui|Échec du déploiement de modèle|Count|Total|Nombre de déploiements de modèles ayant échoué dans cet espace de travail|Scenario, StatusCode|
|Déploiements de modèles commencés|Oui|Déploiements de modèles commencés|Count|Total|Nombre de déploiements de modèles ayant commencé dans cet espace de travail|Scénario|
|Déploiements de modèles réussis|Oui|Déploiements de modèles réussis|Count|Total|Nombre de déploiements de modèles ayant réussi dans cet espace de travail|Scénario|
|Enregistrements de modèles ayant échoué|Oui|Enregistrements de modèles ayant échoué|Count|Total|Nombre d’enregistrements de modèles ayant échoué dans cet espace de travail|Scenario, StatusCode|
|Enregistrements de modèles réussis|Oui|Enregistrements de modèles réussis|Count|Total|Nombre d’enregistrements de modèles ayant réussi dans cet espace de travail|Scénario|
|Exécutions sans réponse|Oui|Exécutions sans réponse|Count|Total|Nombre d’exécutions sans réponse pour cet espace de travail. Le nombre est mis à jour lorsqu’une exécution passe à l’état Sans réponse.|Scenario, RunType, PublishedPipelineId, ComputeType, PipelineStepType|
|Exécutions non démarrées|Oui|Exécutions non démarrées|Count|Total|Nombre d’exécutions à l’état Non démarré pour cet espace de travail. Le nombre est mis à jour lorsqu’une demande de création d’une exécution est reçue, mais que les informations d’exécution n’ont pas encore été remplies. |Scenario, RunType, PublishedPipelineId, ComputeType, PipelineStepType|
|Cœurs réquisitionnés|Oui|Cœurs réquisitionnés|Count|Average|Nombre de cœurs réquisitionnés|Scenario, ClusterName|
|Nœuds reportés|Oui|Nœuds reportés|Count|Average|Nombre de nœuds reportés. Ces nœuds sont les nœuds de basse priorité qui sont éloignés du pool de nœuds disponibles.|Scenario, ClusterName|
|Exécutions en cours de préparation|Oui|Exécutions en cours de préparation|Count|Total|Nombre d’exécutions en cours de préparation pour cet espace de travail. Le nombre est mis à jour lorsqu’une exécution passe à l’état En préparation pendant la préparation de l’environnement d’exécution.|Scenario, RunType, PublishedPipelineId, ComputeType, PipelineStepType|
|Exécutions en cours de provisionnement|Oui|Exécutions en cours de provisionnement|Count|Total|Nombre d’exécutions en cours de provisionnement pour cet espace de travail. Le nombre est mis à jour lorsqu’une exécution attend la création ou le provisionnement d’une cible de calcul.|Scenario, RunType, PublishedPipelineId, ComputeType, PipelineStepType|
|Exécutions en file d’attente|Oui|Exécutions en file d’attente|Count|Total|Nombre d’exécutions mises en file d’attente pour cet espace de travail. Le nombre est mis à jour lorsqu’une exécution est mise en file d’attente dans la cible de calcul. Peut se produire en attendant que les nœuds de calcul nécessaires soient prêts.|Scenario, RunType, PublishedPipelineId, ComputeType, PipelineStepType|
|Pourcentage d’utilisation du quota|Oui|Pourcentage d’utilisation du quota|Count|Average|Pourcentage de quota utilisé|Scenario, ClusterName, VmFamilyName, VmPriority|
|Exécutions démarrées|Oui|Exécutions démarrées|Count|Total|Nombre d’exécutions en cours pour cet espace de travail. Le nombre est mis à jour lorsque l’exécution commence sur les ressources requises.|Scenario, RunType, PublishedPipelineId, ComputeType, PipelineStepType|
|Exécutions en cours de démarrage|Oui|Exécutions en cours de démarrage|Count|Total|Nombre d’exécutions démarrées pour cet espace de travail. Le nombre est mis à jour après que la demande de création de l’exécution et des informations d’exécution (par exemple, l’ID d’exécution) a été remplie.|Scenario, RunType, PublishedPipelineId, ComputeType, PipelineStepType|
|Nombre total de cœurs|Oui|Nombre total de cœurs|Count|Average|Nombre total de cœurs|Scenario, ClusterName|
|Nombre total de nœuds|Oui|Nombre total de nœuds|Count|Average|Nombre total de nœuds. Ce total comprend certains des nœuds actifs, des nœuds inactifs, des nœuds inutilisables, des nœuds reportés et des nœuds sortants|Scenario, ClusterName|
|Cœurs inutilisables|Oui|Cœurs inutilisables|Count|Average|Nombre de cœurs inutilisables|Scenario, ClusterName|
|Nœuds inutilisables|Oui|Nœuds inutilisables|Count|Average|Nombre de nœuds inutilisables. Les nœuds inutilisables ne sont pas fonctionnels en raison d’un problème insoluble. Azure recycle ces nœuds.|Scenario, ClusterName|
|Avertissements|Oui|Avertissements|Count|Total|Nombre d’avertissements d’exécution dans cet espace de travail. Le nombre est mis à jour chaque fois qu’une exécution rencontre un avertissement.|Scénario|


## <a name="microsoftmapsaccounts"></a>Microsoft.Maps/accounts

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|Disponibilité|Oui|Disponibilité|Pourcentage|Average|Disponibilité des API|ApiCategory, ApiName|
|Usage|Non|Usage|Count|Count|Nombre d’appels d’API|ApiCategory, ApiName, ResultType, ResponseCode|


## <a name="microsoftmediamediaservices"></a>Microsoft.Media/mediaservices

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|AssetCount|Oui|Nombre de ressources|Count|Average|Nombre de ressources déjà créées dans le compte de service multimédia actuel|Aucune dimension|
|AssetQuota|Oui|Quota de ressources|Count|Average|Nombre de ressources autorisées pour le compte de service multimédia actuel|Aucune dimension|
|AssetQuotaUsedPercentage|Oui|Pourcentage du quota de ressources utilisé|Pourcentage|Average|Pourcentage de ressources utilisées dans le compte de service multimédia actuel|Aucune dimension|
|ContentKeyPolicyCount|Oui|Nombre de stratégies de clé de contenu|Count|Average|Nombre de stratégies de clé de contenu déjà créées dans le compte de service multimédia actuel|Aucune dimension|
|ContentKeyPolicyQuota|Oui|Quota de stratégies de clé de contenu|Count|Average|Nombre de stratégies de clé de contenu autorisées pour le compte de service multimédia actuel|Aucune dimension|
|ContentKeyPolicyQuotaUsedPercentage|Oui|Pourcentage du quota de stratégies de clé de contenu utilisé|Pourcentage|Average|Pourcentage de stratégies de clé de contenu utilisées dans le compte de service multimédia actuel|Aucune dimension|
|StreamingPolicyCount|Oui|Nombre de stratégies de diffusion en continu|Count|Average|Nombre de stratégies de streaming déjà créées dans le compte de service multimédia actuel|Aucune dimension|
|StreamingPolicyQuota|Oui|Quota de stratégies de diffusion en continu|Count|Average|Nombre de stratégies de streaming autorisées pour le compte de service multimédia actuel|Aucune dimension|
|StreamingPolicyQuotaUsedPercentage|Oui|Pourcentage du quota de stratégies de diffusion en continu utilisé|Pourcentage|Average|Pourcentage de stratégies de streaming utilisées dans le compte de service multimédia actuel|Aucune dimension|


## <a name="microsoftmediamediaservicesstreamingendpoints"></a>Microsoft.Media/mediaservices/streamingEndpoints

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|Sortie|Oui|Sortie|Octets|Total|Quantité de données de sortie, en octets.|OutputFormat|
|Demandes|Oui|Demandes|Count|Total|Demandes à un point de terminaison de streaming.|OutputFormat, HttpStatusCode, ErrorCode|
|SuccessE2ELatency|Oui|Latence de réussite de bout en bout|Millisecondes|Average|Latence moyenne des demandes réussies, en millisecondes.|OutputFormat|


## <a name="microsoftnetappnetappaccountscapacitypools"></a>Microsoft.NetApp/netAppAccounts/capacityPools

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|VolumePoolAllocatedSize|Oui|Taille allouée au pool|Octets|Average|Taille provisionnée de ce pool|Aucune dimension|
|VolumePoolAllocatedUsed|Oui|Pool alloué à la taille du volume|Octets|Average|Taille allouée utilisée du pool|Aucune dimension|
|VolumePoolTotalLogicalSize|Oui|Taille du pool consommée|Octets|Average|Somme de la taille logique de tous les volumes appartenant au pool|Aucune dimension|
|VolumePoolTotalSnapshotSize|Oui|Taille totale des instantanés du pool|Octets|Average|Somme de la taille des instantanés de tous les volumes de ce pool|Aucune dimension|


## <a name="microsoftnetappnetappaccountscapacitypoolsvolumes"></a>Microsoft.NetApp/netAppAccounts/capacityPools/volumes

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|AverageReadLatency|Oui|Latence de lecture moyenne|Millisecondes|Average|Latence de lecture moyenne en millisecondes par opération|Aucune dimension|
|AverageWriteLatency|Oui|Latence d’écriture moyenne|Millisecondes|Average|Latence d’écriture moyenne en millisecondes par opération|Aucune dimension|
|CbsVolumeBackupActive|Oui|État actif de la sauvegarde du volume|Count|Average|La sauvegarde est-elle actuellement suspendue pour le volume|Aucune dimension|
|CbsVolumeLogicalBackupBytes|Oui|Octets logiques sauvegardés|Octets|Average|Nombre total d’octets non compressés/non chiffrés sauvegardés pour ce volume|Aucune dimension|
|CbsVolumeOperationComplete|Oui|État de l’opération|Count|Average|La dernière opération de sauvegarde/restauration a-t-elle réussi|Aucune dimension|
|CbsVolumeOperationTransferredBytes|Oui|Octets transférés pour l’opération|Octets|Average|Nombre total d’octets transférés pour la dernière opération de sauvegarde/restauration|Aucune dimension|
|CbsVolumeProtected|Oui|État de protection du volume|Count|Average|Le volume est-il protégé par le service de sauvegarde dans le cloud|Aucune dimension|
|ReadIops|Oui|E/S par seconde en lecture|CountPerSecond|Average|Opérations d’E/S en lecture par seconde|Aucune dimension|
|VolumeAllocatedSize|Oui|Taille allouée de volume|Octets|Average|Taille provisionnée d’un volume|Aucune dimension|
|VolumeLogicalSize|Oui|Taille du volume consommée|Octets|Average|Taille logique du volume (octets utilisés)|Aucune dimension|
|VolumeSnapshotSize|Oui|Taille d’instantané du volume|Octets|Average|Taille de tous les instantanés dans le volume|Aucune dimension|
|WriteIops|Oui|E/S par seconde en écriture|CountPerSecond|Average|Opérations d’E/S en écriture par seconde|Aucune dimension|
|XregionReplicationHealthy|Oui|L’état de la réplication de volume est-il sain|Count|Average|État de la relation (1 ou 0)|Aucune dimension|
|XregionReplicationLagTime|Oui|Décalage de la réplication de volume|Secondes|Average|Durée, en secondes, du retard des données présentes sur le miroir par rapport à la source|Aucune dimension|
|XregionReplicationLastTransferDuration|Oui|Durée du dernier transfert de réplication de volume|Secondes|Average|Durée, en secondes, nécessaire au dernier transfert|Aucune dimension|
|XregionReplicationLastTransferSize|Oui|Taille du dernier transfert de réplication de volume|Octets|Average|Nombre total d’octets transférés dans le cadre du dernier transfert|Aucune dimension|
|XregionReplicationRelationshipProgress|Oui|Progression de la réplication de volume|Octets|Average|Volume total de données transférées pour l’opération de transfert en cours|Aucune dimension|
|XregionReplicationRelationshipTransferring|Oui|La réplication de volume est-elle en cours de transfert|Count|Average|L’état de la réplication de volume est-il « En cours de transfert »|Aucune dimension|
|XregionReplicationTotalTransferBytes|Oui|Transfert total de la réplication de volume|Octets|Average|Octets cumulés transférés pour la relation|Aucune dimension|


## <a name="microsoftnetworkapplicationgateways"></a>Microsoft.Network/applicationGateways

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|ApplicationGatewayTotalTime|Non|Durée totale d’Application Gateway|Millisecondes|Average|Temps moyen nécessaire pour le traitement d’une requête et l’envoi de la réponse. Elle est calculée en fonction de l’intervalle moyen entre le moment où Application Gateway reçoit le premier octet d’une requête HTTP et le moment où l’opération d’envoi d’une réponse se termine. Il est important de noter que cela implique généralement le temps de traitement d’Application Gateway, le temps pendant lequel les paquets de requête et de réponse transitent sur le réseau et le temps que le serveur principal a mis pour répondre.|Écouteur|
|AvgRequestCountPerHealthyHost|Non|Demandes par minute par hôte sain|Count|Average|Nombre moyen de demandes par minute par hôte back-end sain dans un pool|BackendSettingsPool|
|BackendConnectTime|Non|Temps de connexion au back-end|Millisecondes|Average|Temps passé à établir une connexion avec un back-end|Listener, BackendServer, BackendPool, BackendHttpSetting|
|BackendFirstByteResponseTime|Non|Temps de réponse du premier octet du back-end|Millisecondes|Average|Intervalle de temps entre le début de l’établissement d’une connexion au principal et la réception du premier octet de l’en-tête de réponse, ce qui correspond approximativement au temps de traitement du principal|Listener, BackendServer, BackendPool, BackendHttpSetting|
|BackendLastByteResponseTime|Non|Temps de réponse du dernier octet du back-end|Millisecondes|Average|Intervalle de temps entre le début de l’établissement d’une connexion au principal et la réception du dernier octet du corps de la réponse|Listener, BackendServer, BackendPool, BackendHttpSetting|
|BackendResponseStatus|Oui|État de la réponse du back-end|Count|Total|Nombre de codes de réponse HTTP générés par les membres du back-end. Cela n’inclut pas les codes de réponse générés par Application Gateway.|BackendServer, BackendPool, BackendHttpSetting, HttpStatusGroup|
|BlockedCount|Oui|Distribution des règles de demandes bloquées du pare-feu d’applications web|Count|Total|Distribution des règles de demandes bloquées du pare-feu d’applications web|RuleGroup, RuleId|
|BlockedReqCount|Oui|Nombre de demandes bloquées du pare-feu d’applications web|Count|Total|Nombre de demandes bloquées du pare-feu d’applications web|Aucune dimension|
|BytesReceived|Oui|Octets reçus|Octets|Total|Nombre total d’octets reçus par Application Gateway à partir des clients|Écouteur|
|BytesSent|Oui|Octets envoyés|Octets|Total|Nombre total d’octets envoyés par Application Gateway aux clients|Écouteur|
|CapacityUnits|Non|Unités de capacité actuelles|Count|Average|Unités de capacité consommées|Aucune dimension|
|ClientRtt|Non|RTT client|Millisecondes|Average|Durée moyenne des allers-retours entre les clients et Application Gateway. Indique le temps nécessaire à l’établissement des connexions et à l’envoi d’accusés de réception|Écouteur|
|ComputeUnits|Non|Unités Compute actuelles|Count|Average|Unités Compute consommées|Aucune dimension|
|CpuUtilization|Non|Utilisation du processeur|Pourcentage|Average|Utilisation actuelle du processeur d’Application Gateway|Aucune dimension|
|CurrentConnections|Oui|Connexions courantes|Count|Total|Nombre de connexions courantes établies avec Application Gateway|Aucune dimension|
|EstimatedBilledCapacityUnits|Non|Unités de capacité facturées estimées|Count|Average|Unités de capacité estimées qui seront facturées|Aucune dimension|
|FailedRequests|Oui|Demandes ayant échoué|Count|Total|Nombre de requêtes en échec servies par Application Gateway|BackendSettingsPool|
|FixedBillableCapacityUnits|Non|Unités de capacité facturables fixes|Count|Average|Unités de capacité minimales qui seront facturées|Aucune dimension|
|HealthyHostCount|Oui|Nombre d’hôtes intègres|Count|Average|Nombre d’hôtes principaux intègres|BackendSettingsPool|
|MatchedCount|Oui|Distribution totale des règles du pare-feu d’applications web|Count|Total|Distribution totale des règles du pare-feu d’applications web pour le trafic entrant|RuleGroup, RuleId|
|NewConnectionsPerSecond|Non|Nouvelles connexions par seconde|CountPerSecond|Average|Nouvelles connexions par seconde établies avec Application Gateway|Aucune dimension|
|ResponseStatus|Oui|État de la réponse|Count|Total|État de la réponse HTTP retourné par Application Gateway|HttpStatusGroup|
|Débit|Non|Débit|BytesPerSecond|Average|Nombre d’octets par seconde servis par Application Gateway|Aucune dimension|
|TlsProtocol|Oui|Protocole TLS du client|Count|Total|Nombre de demandes TLS et non-TLS lancées par le client qui a établi la connexion avec Application Gateway. Pour afficher la distribution du protocole TLS, filtrez par la dimension Protocole TLS.|Listener, TlsProtocol|
|TotalRequests|Oui|Total de requêtes|Count|Total|Nombre de requêtes réussies servies par Application Gateway|BackendSettingsPool|
|UnhealthyHostCount|Oui|Nombre d’hôtes non intègres|Count|Average|Nombre d’hôtes principaux non intègres|BackendSettingsPool|


## <a name="microsoftnetworkazurefirewalls"></a>Microsoft.Network/azurefirewalls

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|ApplicationRuleHit|Oui|Nombre d'accès aux règles d'application|Count|Total|Nombre d'accès aux règles d'application|Status, Reason, Protocol|
|DataProcessed|Oui|Données traitées|Octets|Total|Volume total de données traitées par ce pare-feu|Aucune dimension|
|FirewallHealth|Oui|État d’intégrité du pare-feu|Pourcentage|Average|Intégrité globale de ce pare-feu|Statut, Motif|
|NetworkRuleHit|Oui|Nombre d’accès aux règles de réseau|Count|Total|Nombre d’accès aux règles de réseau|Status, Reason, Protocol|
|SNATPortUtilization|Oui|Utilisation des ports SNAT|Pourcentage|Average|Pourcentage de ports SNAT sortants en cours d’utilisation|Protocol|
|Débit|Non|Débit|BitsPerSecond|Average|Débit traité par ce pare-feu|Aucune dimension|


## <a name="microsoftnetworkconnections"></a>Microsoft.Network/connections

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|BitsInPerSecond|Oui|BitsInPerSecond|CountPerSecond|Average|Bits entrant dans Azure par seconde|Aucune dimension|
|BitsOutPerSecond|Oui|BitsOutPerSecond|CountPerSecond|Average|Bits sortant d’Azure par seconde|Aucune dimension|


## <a name="microsoftnetworkdnszones"></a>Microsoft.Network/dnszones

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|QueryVolume|Oui|Volume de requête|Count|Total|Nombre de requêtes servies pour une zone DNS|Aucune dimension|
|RecordSetCapacityUtilization|Non|Utilisation de la capacité du jeu d’enregistrements|Pourcentage|Maximale|Pourcentage de la capacité du jeu d’enregistrements utilisée par une zone DNS|Aucune dimension|
|RecordSetCount|Oui|Nombre de jeux d’enregistrements|Count|Maximale|Nombre de jeux d’enregistrements dans une zone DNS|Aucune dimension|


## <a name="microsoftnetworkexpressroutecircuits"></a>Microsoft.Network/expressRouteCircuits

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|ArpAvailability|Oui|Disponibilité du protocole ARP|Pourcentage|Average|Disponibilité du protocole ARP entre MSEE et tous les pairs.|PeeringType, Peer|
|BgpAvailability|Oui|Disponibilité du protocole BGP|Pourcentage|Average|Disponibilité du protocole BGP entre MSEE et tous les pairs.|PeeringType, Peer|
|BitsInPerSecond|Non|BitsInPerSecond|CountPerSecond|Average|Bits entrant dans Azure par seconde|PeeringType|
|BitsOutPerSecond|Non|BitsOutPerSecond|CountPerSecond|Average|Bits sortant d’Azure par seconde|PeeringType|
|GlobalReachBitsInPerSecond|Non|GlobalReachBitsInPerSecond|CountPerSecond|Average|Bits entrant dans Azure par seconde|PeeredCircuitSKey|
|GlobalReachBitsOutPerSecond|Non|GlobalReachBitsOutPerSecond|CountPerSecond|Average|Bits sortant d’Azure par seconde|PeeredCircuitSKey|
|QosDropBitsInPerSecond|Non|DroppedInBitsPerSecond|CountPerSecond|Average|Octets de données d’entrée abandonnés par seconde|Aucune dimension|
|QosDropBitsOutPerSecond|Non|DroppedOutBitsPerSecond|CountPerSecond|Average|Octets de données de sortie abandonnés par seconde|Aucune dimension|


## <a name="microsoftnetworkexpressroutecircuitspeerings"></a>Microsoft.Network/expressRouteCircuits/peerings

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|BitsInPerSecond|Oui|BitsInPerSecond|CountPerSecond|Average|Bits entrant dans Azure par seconde|Aucune dimension|
|BitsOutPerSecond|Oui|BitsOutPerSecond|CountPerSecond|Average|Bits sortant d’Azure par seconde|Aucune dimension|


## <a name="microsoftnetworkexpressroutegateways"></a>Microsoft.Network/expressRouteGateways

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|ErGatewayConnectionBitsInPerSecond|Non|BitsInPerSecond|CountPerSecond|Average|Bits entrant dans Azure par seconde|ConnectionName|
|ErGatewayConnectionBitsOutPerSecond|Non|BitsOutPerSecond|CountPerSecond|Average|Bits sortant d’Azure par seconde|ConnectionName|


## <a name="microsoftnetworkexpressrouteports"></a>Microsoft.Network/expressRoutePorts

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|AdminState|Oui|AdminState|Count|Average|État administrateur du port|Lien|
|LineProtocol|Oui|LineProtocol|Count|Average|État du protocole de ligne du port|Lien|
|PortBitsInPerSecond|Oui|BitsInPerSecond|CountPerSecond|Average|Bits entrant dans Azure par seconde|Lien|
|PortBitsOutPerSecond|Oui|BitsOutPerSecond|CountPerSecond|Average|Bits sortant d’Azure par seconde|Lien|
|RxLightLevel|Oui|RxLightLevel|Count|Average|Niveau d’éclairage de réception en dBm|Link, Lane|
|TxLightLevel|Oui|TxLightLevel|Count|Average|Niveau d’éclairage de transmission en dBm|Link, Lane|


## <a name="microsoftnetworkfrontdoors"></a>Microsoft.Network/frontdoors

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|BackendHealthPercentage|Oui|Pourcentage d’intégrité du backend|Pourcentage|Average|Pourcentage de sondes d’intégrité réussies du proxy HTTP/S vers les serveurs principaux|Backend, BackendPool|
|BackendRequestCount|Oui|Nombre de requêtes de backend|Count|Total|Nombre de requêtes envoyées du proxy HTTP/S aux serveurs principaux|HttpStatus, HttpStatusGroup, Backend|
|BackendRequestLatency|Oui|Latence de requête du backend|Millisecondes|Average|Temps calculé à partir du moment où la requête est envoyée par le proxy HTTP/S au serveur principal jusqu’à ce que le proxy HTTP/S reçoive le dernier octet de la réponse du serveur principal|Backend|
|BillableResponseSize|Oui|Taille de la réponse facturable|Octets|Total|Nombre d’octets facturables (2 Ko minimum par demande) envoyés en tant que réponses du proxy HTTP/S aux clients.|HttpStatus, HttpStatusGroup, ClientRegion, ClientCountry|
|RequestCount|Oui|Nombre de requêtes|Count|Total|Nombre de requêtes clients prises en charge par le proxy HTTP/S|HttpStatus, HttpStatusGroup, ClientRegion, ClientCountry|
|RequestSize|Oui|Taille de la requête|Octets|Total|Nombre d’octets envoyés en tant que requêtes de clients au proxy HTTP/S|HttpStatus, HttpStatusGroup, ClientRegion, ClientCountry|
|ResponseSize|Oui|Taille de la réponse|Octets|Total|Nombre d’octets envoyés en tant que réponses du proxy HTTP/S aux clients|HttpStatus, HttpStatusGroup, ClientRegion, ClientCountry|
|TotalLatency|Oui|Latence totale|Millisecondes|Average|Temps calculé à partir du moment où la requête du client est reçue par le proxy HTTP/S jusqu’à ce que le client accuse réception du dernier octet de la réponse du proxy HTTP/S|HttpStatus, HttpStatusGroup, ClientRegion, ClientCountry|
|WebApplicationFirewallRequestCount|Oui|Nombre de requêtes du pare-feu d’applications web|Count|Total|Nombre de requêtes clients traitées par le pare-feu d’applications web|PolicyName, RuleName, Action|


## <a name="microsoftnetworkloadbalancers"></a>Microsoft.Network/loadBalancers

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|AllocatedSnatPorts|Non|Ports SNAT alloués|Count|Average|Nombre total de ports SNAT alloués pendant la période|FrontendIPAddress, BackendIPAddress, ProtocolType |
|ByteCount|Oui|Nombre d’octets|Count|Total|Nombre total d’octets transmis dans une période de temps|FrontendIPAddress, FrontendPort, Direction|
|DipAvailability|Oui|État de la sonde d’intégrité|Count|Average|État de la sonde d’intégrité d’équilibreur de charge moyen par durée|ProtocolType, BackendPort, FrontendIPAddress, FrontendPort, BackendIPAddress|
|PacketCount|Oui|Nombre de paquets|Count|Total|Nombre total de paquets transmis dans une période de temps|FrontendIPAddress, FrontendPort, Direction|
|SnatConnectionCount|Oui|Nombre de connexions SNAT|Count|Total|Nombre total de connexions SNAT créées dans une période de temps|FrontendIPAddress, BackendIPAddress, ConnectionState|
|SYNCount|Oui|Nombre de SYN|Count|Total|Nombre total de paquets SYN transmis dans une période de temps|FrontendIPAddress, FrontendPort, Direction|
|UsedSnatPorts|Non|Ports SNAT utilisés|Count|Average|Nombre total de ports SNAT utilisés pendant une période|FrontendIPAddress, BackendIPAddress, ProtocolType |
|VipAvailability|Oui|Disponibilité du chemin d’accès aux données|Count|Average|Disponibilité du chemin d’accès aux données d’équilibreur de charge moyenne par durée|FrontendIPAddress, FrontendPort|


## <a name="microsoftnetworknetworkinterfaces"></a>Microsoft.Network/networkInterfaces

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|BytesReceivedRate|Oui|Octets reçus|Octets|Total|Nombre d’octets reçus par l’interface réseau|Aucune dimension|
|BytesSentRate|Oui|Octets envoyés|Octets|Total|Nombre d’octets envoyés par l’interface réseau|Aucune dimension|
|PacketsReceivedRate|Oui|Paquets reçus|Count|Total|Nombre de paquets reçus par l’interface réseau|Aucune dimension|
|PacketsSentRate|Oui|Paquets envoyés|Count|Total|Nombre de paquets envoyés par l’interface réseau|Aucune dimension|


## <a name="microsoftnetworknetworkwatchersconnectionmonitors"></a>Microsoft.Network/networkWatchers/connectionMonitors

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|AverageRoundtripMs|Oui|Avg. Durée aller-retour (ms)|Millisecondes|Average|Durée aller-retour réseau moyenne (ms) pour les sondes de surveillance de connectivité envoyées entre la source et la destination|Aucune dimension|
|ChecksFailedPercent|Oui|Pourcentage de vérifications ayant échoué (préversion)|Pourcentage|Average|% de vérifications de supervision de connectivité ayant échoué|SourceAddress, SourceName, SourceResourceId, SourceType, Protocol, DestinationAddress, DestinationName, DestinationResourceId, DestinationType, DestinationPort, TestGroupName, TestConfigurationName|
|ProbesFailedPercent|Oui|% de sondes ayant échoué|Pourcentage|Average|% de sondes de surveillance de connectivité ayant échoué|Aucune dimension|
|RoundTripTimeMs|Oui|Durée aller-retour (ms) (préversion)|Millisecondes|Average|Durée aller-retour en millisecondes pour les vérifications de supervision de la connectivité|SourceAddress, SourceName, SourceResourceId, SourceType, Protocol, DestinationAddress, DestinationName, DestinationResourceId, DestinationType, DestinationPort, TestGroupName, TestConfigurationName|


## <a name="microsoftnetworkpublicipaddresses"></a>Microsoft.Network/publicIPAddresses

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|ByteCount|Oui|Nombre d’octets|Count|Total|Nombre total d’octets transmis dans une période de temps|Port, Direction|
|BytesDroppedDDoS|Oui|Octets entrants supprimés DDoS|BytesPerSecond|Maximale|Octets entrants supprimés DDoS|Aucune dimension|
|BytesForwardedDDoS|Oui|Octets entrants transférés DDoS|BytesPerSecond|Maximale|Octets entrants transférés DDoS|Aucune dimension|
|BytesInDDoS|Oui|Octets entrants DDoS|BytesPerSecond|Maximale|Octets entrants DDoS|Aucune dimension|
|DDoSTriggerSYNPackets|Oui|Paquets SYN entrants pour déclencher l’atténuation DDoS|CountPerSecond|Maximale|Paquets SYN entrants pour déclencher l’atténuation DDoS|Aucune dimension|
|DDoSTriggerTCPPackets|Oui|Paquets TCP entrants pour déclencher l’atténuation DDoS|CountPerSecond|Maximale|Paquets TCP entrants pour déclencher l’atténuation DDoS|Aucune dimension|
|DDoSTriggerUDPPackets|Oui|Paquets UDP entrants pour déclencher l’atténuation DDoS|CountPerSecond|Maximale|Paquets UDP entrants pour déclencher l’atténuation DDoS|Aucune dimension|
|IfUnderDDoSAttack|Oui|Sous attaque DDoS ou non|Count|Maximale|Sous attaque DDoS ou non|Aucune dimension|
|PacketCount|Oui|Nombre de paquets|Count|Total|Nombre total de paquets transmis dans une période de temps|Port, Direction|
|PacketsDroppedDDoS|Oui|Paquets entrants ignorés DDoS|CountPerSecond|Maximale|Paquets entrants ignorés DDoS|Aucune dimension|
|PacketsForwardedDDoS|Oui|Paquets entrants transférés DDoS|CountPerSecond|Maximale|Paquets entrants transférés DDoS|Aucune dimension|
|PacketsInDDoS|Oui|Paquets entrants DDoS|CountPerSecond|Maximale|Paquets entrants DDoS|Aucune dimension|
|SynCount|Oui|Nombre de SYN|Count|Total|Nombre total de paquets SYN transmis dans une période de temps|Port, Direction|
|TCPBytesDroppedDDoS|Oui|Octets TCP entrants supprimés DDoS|BytesPerSecond|Maximale|Octets TCP entrants supprimés DDoS|Aucune dimension|
|TCPBytesForwardedDDoS|Oui|Octets TCP entrants transférés DDoS|BytesPerSecond|Maximale|Octets TCP entrants transférés DDoS|Aucune dimension|
|TCPBytesInDDoS|Oui|Octets TCP entrants DDoS|BytesPerSecond|Maximale|Octets TCP entrants DDoS|Aucune dimension|
|TCPPacketsDroppedDDoS|Oui|Paquets TCP entrants TCP ignorés DDoS|CountPerSecond|Maximale|Paquets TCP entrants TCP ignorés DDoS|Aucune dimension|
|TCPPacketsForwardedDDoS|Oui|Paquets TCP entrants transférés DDoS|CountPerSecond|Maximale|Paquets TCP entrants transférés DDoS|Aucune dimension|
|TCPPacketsInDDoS|Oui|Paquets TCP entrants DDoS|CountPerSecond|Maximale|Paquets TCP entrants DDoS|Aucune dimension|
|UDPBytesDroppedDDoS|Oui|Octets UDP entrants supprimés DDoS|BytesPerSecond|Maximale|Octets UDP entrants supprimés DDoS|Aucune dimension|
|UDPBytesForwardedDDoS|Oui|Octets UDP entrants transférés DDoS|BytesPerSecond|Maximale|Octets UDP entrants transférés DDoS|Aucune dimension|
|UDPBytesInDDoS|Oui|Octets UDP entrants DDoS|BytesPerSecond|Maximale|Octets UDP entrants DDoS|Aucune dimension|
|UDPPacketsDroppedDDoS|Oui|Paquets UDP entrants ignorés DDoS|CountPerSecond|Maximale|Paquets UDP entrants ignorés DDoS|Aucune dimension|
|UDPPacketsForwardedDDoS|Oui|Paquets UDP entrants transférés DDoS|CountPerSecond|Maximale|Paquets UDP entrants transférés DDoS|Aucune dimension|
|UDPPacketsInDDoS|Oui|Paquets UDP entrants DDoS|CountPerSecond|Maximale|Paquets UDP entrants DDoS|Aucune dimension|
|VipAvailability|Oui|Disponibilité du chemin d’accès aux données|Count|Average|Disponibilité d’adresse IP moyenne par durée|Port|


## <a name="microsoftnetworktrafficmanagerprofiles"></a>Microsoft.Network/trafficManagerProfiles

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|ProbeAgentCurrentEndpointStateByProfileResourceId|Oui|État du point de terminaison par point de terminaison|Count|Maximale|1 si l’état de sondage d’un point de terminaison est « activé », 0 dans le cas contraire.|EndpointName|
|QpsByEndpoint|Oui|Requêtes par point de terminaison renvoyé|Count|Total|Nombre de fois où un point de terminaison Traffic Manager a été renvoyé dans le laps de temps donné|EndpointName|


## <a name="microsoftnetworkvirtualnetworkgateways"></a>Microsoft.Network/virtualNetworkGateways

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|AverageBandwidth|Oui|Bande passante S2S de passerelle|BytesPerSecond|Average|Bande passante site à site moyenne d’une passerelle en octets par seconde|Aucune dimension|
|P2SBandwidth|Oui|Bande passante P2S de passerelle|BytesPerSecond|Average|Bande passante point à site moyenne d’une passerelle en octets par seconde|Aucune dimension|
|P2SConnectionCount|Oui|Nombre de connexions P2S|Count|Maximale|Nombre de connexions point à site d’une passerelle|Protocol|
|TunnelAverageBandwidth|Oui|Bande passante de tunnel|BytesPerSecond|Average|Bande passante moyenne d’un tunnel en octets par seconde|ConnectionName, RemoteIP|
|TunnelEgressBytes|Oui|Octets de sortie de tunnel|Octets|Total|Octets sortants d’un tunnel|ConnectionName, RemoteIP|
|TunnelEgressPacketDropTSMismatch|Oui|Rejet de paquets TS de sortie de tunnel pour incompatibilité|Count|Total|Nombre de rejets de paquets sortants du sélecteur de trafic pour incompatibilité dans un tunnel|ConnectionName, RemoteIP|
|TunnelEgressPackets|Oui|Paquets de sortie de tunnel|Count|Total|Nombre de paquets sortants d’un tunnel|ConnectionName, RemoteIP|
|TunnelIngressBytes|Oui|Octets d’entrée de tunnel|Octets|Total|Octets entrants d’un tunnel|ConnectionName, RemoteIP|
|TunnelIngressPacketDropTSMismatch|Oui|Rejet de paquets TS d’entrée de tunnel pour incompatibilité|Count|Total|Nombre de rejets de paquets entrants du sélecteur de trafic pour incompatibilité dans un tunnel|ConnectionName, RemoteIP|
|TunnelIngressPackets|Oui|Paquets en entrée de tunnel|Count|Total|Nombre de paquets entrants d’un tunnel|ConnectionName, RemoteIP|


## <a name="microsoftnetworkvirtualnetworks"></a>Microsoft.Network/virtualNetworks

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|PingMeshAverageRoundtripMs|Oui|Durée aller-retour pour les commandes ping à destination d’une machine virtuelle|Millisecondes|Average|Durée aller-retour pour les commandes ping envoyées à une machine virtuelle de destination|SourceCustomerAddress, DestinationCustomerAddress|
|PingMeshProbesFailedPercent|Oui|Commandes ping à destination d’une machine virtuelle qui ont échoué|Pourcentage|Average|Pourcentage du nombre d’échecs de commandes ping par rapport au total de commandes ping envoyées à une machine virtuelle de destination|SourceCustomerAddress, DestinationCustomerAddress|


## <a name="microsoftnotificationhubsnamespacesnotificationhubs"></a>Microsoft.NotificationHubs/Namespaces/NotificationHubs

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|incoming|Oui|Messages entrants|Count|Total|Total des appels d’API d’envoi réussis. |Aucune dimension|
|incoming.all.failedrequests|Oui|Toutes les requêtes entrantes ayant échoué|Count|Total|Nombre total des demandes entrantes ayant échoué pour un hub de notification|Aucune dimension|
|incoming.all.requests|Oui|Toutes les demandes entrantes|Count|Total|Nombre total des demandes entrantes pour un hub de notification|Aucune dimension|
|incoming.scheduled|Oui|Notifications Push planifiées envoyées|Count|Total|Notifications Push planifiées annulées|Aucune dimension|
|incoming.scheduled.cancel|Oui|Notifications Push planifiées annulées|Count|Total|Notifications Push planifiées annulées|Aucune dimension|
|installation.all|Oui|Opérations de gestion de l’installation|Count|Total|Opérations de gestion de l’installation|Aucune dimension|
|installation.delete|Oui|Opérations de suppression d’installation|Count|Total|Opérations de suppression d’installation|Aucune dimension|
|installation.get|Oui|Opérations d’obtention d’installation|Count|Total|Opérations d’obtention d’installation|Aucune dimension|
|installation.patch|Oui|Opérations d’installation de correctif|Count|Total|Opérations d’installation de correctif|Aucune dimension|
|installation.upsert|Oui|Opérations de création ou de mise à jour d’installation|Count|Total|Opérations de création ou de mise à jour d’installation|Aucune dimension|
|notificationhub.pushes|Oui|Toutes les notifications sortantes|Count|Total|Toutes les notifications sortantes du hub de notification|Aucune dimension|
|outgoing.allpns.badorexpiredchannel|Oui|Erreurs de canal incorrect ou arrivé à expiration|Count|Total|Nombre d’opérations Push en échec, car le canal/jeton/registrationId indiqué dans l’inscription est arrivé à expiration ou non valide.|Aucune dimension|
|outgoing.allpns.channelerror|Oui|Erreurs de canal|Count|Total|Nombre d’opérations Push en échec, car le canal n’était pas valide ni associé à l’application appropriée, limité ou arrivé à expiration.|Aucune dimension|
|outgoing.allpns.invalidpayload|Oui|Erreurs de charge utile|Count|Total|Nombre d’opérations Push en échec, car le PNS a retourné une erreur de charge utile incorrecte.|Aucune dimension|
|outgoing.allpns.pnserror|Oui|Erreurs système de notification externe|Count|Total|Nombre d’opérations Push en échec en raison d’un problème de communication avec le PNS (à l’exclusion des problèmes d’authentification).|Aucune dimension|
|outgoing.allpns.success|Oui|Notifications réussies|Count|Total|Total des notifications réussies.|Aucune dimension|
|outgoing.apns.badchannel|Oui|Erreur de canal APNS incorrect|Count|Total|Nombre d'opérations d'envoi (push) qui ont échoué car le jeton n'est pas valide (code d'état APNS : 8).|Aucune dimension|
|outgoing.apns.expiredchannel|Oui|Erreur de canal APNS arrivé à expiration|Count|Total|Nombre de jetons qui ont été invalidés par le canal de commentaires APNS.|Aucune dimension|
|outgoing.apns.invalidcredentials|Oui|Erreurs d’autorisation APNS|Count|Total|Nombre d’opérations Push en échec, car le PNS n’a pas accepté les informations d’identification fournies ou celles-ci sont bloquées.|Aucune dimension|
|outgoing.apns.invalidnotificationsize|Oui|Erreur de taille de notification APNS non valide|Count|Total|Nombre d'opérations d'envoi (push) qui ont échoué car la charge utile était trop importante (code d'état APNS : 7).|Aucune dimension|
|outgoing.apns.pnserror|Oui|Erreurs APNS|Count|Total|Nombre d’opérations Push en échec en raison d’erreurs de communication avec APNS.|Aucune dimension|
|outgoing.apns.success|Oui|Notifications APNS réussies|Count|Total|Total des notifications réussies.|Aucune dimension|
|outgoing.gcm.authenticationerror|Oui|Erreurs d’authentification GCM|Count|Total|Nombre d'opérations d'envoi (push) qui ont échoué car le PNS n'a pas accepté les informations d'identification fournies, celles-ci sont bloquées ou l'élément SenderId n'est pas correctement configuré dans l'application (résultat GCM : MismatchedSenderId).|Aucune dimension|
|outgoing.gcm.badchannel|Oui|Erreur de canal GCM incorrect|Count|Total|Nombre d'opérations d'envoi (push) qui ont échoué car l'élément registrationId contenu dans l'inscription n'a pas été reconnu (résultat GCM : inscription non valide).|Aucune dimension|
|outgoing.gcm.expiredchannel|Oui|Erreur de canal GCM arrivé à expiration|Count|Total|Nombre d'opérations d'envoi (push) qui ont échoué car l'élément registrationId contenu dans l'inscription n'a pas été reconnu (résultat GCM : NotRegistered).|Aucune dimension|
|outgoing.gcm.invalidcredentials|Oui|Erreurs d’autorisation GCM (informations d’identification non valides)|Count|Total|Nombre d’opérations Push en échec, car le PNS n’a pas accepté les informations d’identification fournies ou celles-ci sont bloquées.|Aucune dimension|
|outgoing.gcm.invalidnotificationformat|Oui|Format de notification GCM non valide|Count|Total|Nombre d'opérations d'envoi (push) qui ont échoué car la charge utile n'était pas correctement mise en forme (résultat GCM : InvalidDataKey ou InvalidTtl).|Aucune dimension|
|outgoing.gcm.invalidnotificationsize|Oui|Erreur de taille de notification GCM non valide|Count|Total|Nombre d'opérations d'envoi (push) qui ont échoué car la charge utile était trop importante (résultat GCM : MessageTooBig).|Aucune dimension|
|outgoing.gcm.pnserror|Oui|Erreurs GCM|Count|Total|Nombre d’opérations Push en échec en raison d’erreurs de communication avec GCM.|Aucune dimension|
|outgoing.gcm.success|Oui|Notifications GCM réussies|Count|Total|Total des notifications réussies.|Aucune dimension|
|outgoing.gcm.throttled|Oui|Notifications GCM limitées|Count|Total|Nombre d'opérations d'envoi (push) qui ont échoué car GCM a limité cette application (code d'état GCM : 501-599 ou résultat : non disponible).|Aucune dimension|
|outgoing.gcm.wrongchannel|Oui|Erreur de canal GCM incorrect|Count|Total|Nombre d'opérations d'envoi (push) qui ont échoué car l'élément registrationId contenu dans l'inscription n'est pas associé à l'application actuelle (résultat GCM : InvalidPackageName).|Aucune dimension|
|outgoing.mpns.authenticationerror|Oui|Erreurs d’authentification MPNS|Count|Total|Nombre d’opérations Push en échec, car le PNS n’a pas accepté les informations d’identification fournies ou celles-ci sont bloquées.|Aucune dimension|
|outgoing.mpns.badchannel|Oui|Erreur de canal incorrect MPNS|Count|Total|Nombre d'opérations d'envoi (push) qui ont échoué car l'élément ChannelURI contenu dans l'inscription n'a pas été reconnu (état MPNS : 404 - Introuvable).|Aucune dimension|
|outgoing.mpns.channeldisconnected|Oui|Canal MPNS déconnecté|Count|Total|Nombre d'opérations d'envoi (push) qui ont échoué car l'élément ChannelURI contenu dans l'inscription était déconnecté (état MPNS : 412 introuvable).|Aucune dimension|
|outgoing.mpns.dropped|Oui|Notifications MPNS supprimées|Count|Total|Nombre d'opérations d'envoi (push) qui ont été abandonnées par MPNS (en-tête de réponse MPNS : X-NotificationStatus: QueueFull ou Suppressed).|Aucune dimension|
|outgoing.mpns.invalidcredentials|Oui|Informations d’identification MPNS non valides|Count|Total|Nombre d’opérations Push en échec, car le PNS n’a pas accepté les informations d’identification fournies ou celles-ci sont bloquées.|Aucune dimension|
|outgoing.mpns.invalidnotificationformat|Oui|Format de notification MPNS non valide|Count|Total|Nombre d’opérations Push en échec, car la charge utile de la notification était trop importante.|Aucune dimension|
|outgoing.mpns.pnserror|Oui|Erreurs MPNS|Count|Total|Nombre d’opérations Push en échec en raison d’erreurs de communication avec MPNS.|Aucune dimension|
|outgoing.mpns.success|Oui|Notifications MPNS réussies|Count|Total|Total des notifications réussies.|Aucune dimension|
|outgoing.mpns.throttled|Oui|Notifications MPNS limitées|Count|Total|Nombre d'opérations d'envoi (push) qui ont échoué car MPNS limite cette application (WNS MPNS : 406 Non acceptable).|Aucune dimension|
|outgoing.wns.authenticationerror|Oui|Erreurs d’authentification WNS|Count|Total|La notification n’a pas été remise en raison d’erreurs de communication avec Windows Live, d’informations d’identification non valides ou d’un jeton incorrect.|Aucune dimension|
|outgoing.wns.badchannel|Oui|Erreur de canal WNS incorrect|Count|Total|Nombre d'opérations d'envoi (push) qui ont échoué car l'élément ChannelURI indiqué dans l'inscription n'a pas été reconnu (état WNS : 404 - Introuvable).|Aucune dimension|
|outgoing.wns.channeldisconnected|Oui|Canal WNS déconnecté|Count|Total|La notification a été supprimée car l'élément ChannelURI contenu dans l'inscription est limité (en-tête de réponse WNS : X-WNS-DeviceConnectionStatus: déconnecté).|Aucune dimension|
|outgoing.wns.channelthrottled|Oui|Canal WNS limité|Count|Total|La notification a été supprimée car l'élément ChannelURI contenu dans l'inscription est limité (en-tête de réponse WNS : X-WNS-NotificationStatus:channelThrottled).|Aucune dimension|
|outgoing.wns.dropped|Oui|Notifications WNS supprimées|Count|Total|La notification a été supprimée, car l’élément ChannelURI indiqué dans l’inscription est limité (X-WNS-NotificationStatus : supprimé, mais pas X-WNS-DeviceConnectionStatus : déconnecté).|Aucune dimension|
|outgoing.wns.expiredchannel|Oui|Erreur de canal WNS arrivé à expiration|Count|Total|Nombre d'opérations d'envoi (push) qui ont échoué car l'élément ChannelURI est arrivé à expiration (état WNS : 410 Supprimé).|Aucune dimension|
|outgoing.wns.invalidcredentials|Oui|Erreurs d’autorisation WNS (informations d’identification non valides)|Count|Total|Nombre d’opérations Push en échec, car le PNS n’a pas accepté les informations d’identification fournies ou celles-ci sont bloquées. (Windows Live ne reconnaît pas les informations d’identification).|Aucune dimension|
|outgoing.wns.invalidnotificationformat|Oui|Format de notification WNS non valide|Count|Total|Le format de la notification n'est pas valide (état WNS : 400). Notez que WNS ne rejette pas toutes les charges utiles non valides.|Aucune dimension|
|outgoing.wns.invalidnotificationsize|Oui|Erreur de taille de notification WNS non valide|Count|Total|La charge utile de notification est trop grande (état WNS : 413).|Aucune dimension|
|outgoing.wns.invalidtoken|Oui|Erreurs d’autorisation WNS (jeton non valide)|Count|Total|Le jeton fourni à WNS n'est pas valide (état WNS : 401 - Non autorisé).|Aucune dimension|
|outgoing.wns.pnserror|Oui|Erreurs WNS|Count|Total|La notification n’a pas été remise en raison d’erreurs de communication avec WNS.|Aucune dimension|
|outgoing.wns.success|Oui|Notifications WNS réussies|Count|Total|Total des notifications réussies.|Aucune dimension|
|outgoing.wns.throttled|Oui|Notifications WNS limitées|Count|Total|Nombre d'opérations d'envoi (push) qui ont échoué car WNS limite cette application (état WNS : 406 Non acceptable).|Aucune dimension|
|outgoing.wns.tokenproviderunreachable|Oui|Erreurs d’autorisation WNS (inaccessible)|Count|Total|Windows Live n’est pas accessible.|Aucune dimension|
|outgoing.wns.wrongtoken|Oui|Erreurs d’autorisation WNS (jeton incorrect)|Count|Total|Le jeton fourni à WNS est valide mais il concerne une autre application (état WNS : 403 - Interdit). Cela peut se produire si l’élément ChannelURI indiqué dans l’inscription est associé à une autre application. Vérifiez que l’application cliente est associée à l’application dont les informations d’identification figurent dans le hub de notification.|Aucune dimension|
|registration.all|Oui|Opérations d’inscription|Count|Total|Total des opérations d’inscription réussies (requêtes de mises à jour de créations et suppressions). |Aucune dimension|
|registration.create|Oui|Opérations de création d’inscription|Count|Total|Total des créations d’inscription réussies.|Aucune dimension|
|registration.delete|Oui|Opérations de suppression d’inscription|Count|Total|Total des suppressions d’inscription réussies.|Aucune dimension|
|registration.get|Oui|Opérations de lecture d’inscription|Count|Total|Total des requêtes d’inscription réussies.|Aucune dimension|
|registration.update|Oui|Opérations de mise à jour d’inscription|Count|Total|Total des mises à jour d’inscription réussies.|Aucune dimension|
|scheduled.pending|Oui|Notifications planifiées en attente|Count|Total|Notifications planifiées en attente|Aucune dimension|


## <a name="microsoftoperationalinsightsworkspaces"></a>Microsoft.OperationalInsights/workspaces

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|Average_% Available Memory|Oui|% Available Memory|Count|Average|Average_% Available Memory|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_% Available Swap Space|Oui|% Available Swap Space|Count|Average|Average_% Available Swap Space|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Octets validés en cours d’utilisation Average_%|Oui|% d’octets validés utilisés|Count|Average|Octets validés en cours d’utilisation Average_%|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_% DPC Time|Oui|% DPC Time|Count|Average|Average_% DPC Time|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Inodes libres Average_%|Oui|% Free Inodes|Count|Average|Inodes libres Average_%|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_% Free Space|Oui|% Free Space|Count|Average|Average_% Free Space|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_% Idle Time|Oui|% Idle Time|Count|Average|Average_% Idle Time|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_% Interrupt Time|Oui|% Interrupt Time|Count|Average|Average_% Interrupt Time|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_% IO Wait Time|Oui|% IO Wait Time|Count|Average|Average_% IO Wait Time|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_% Nice Time|Oui|% Nice Time|Count|Average|Average_% Nice Time|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_% Privileged Time|Oui|% Privileged Time|Count|Average|Average_% Privileged Time|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Temps processeur Average_%|Oui|% temps processeur|Count|Average|Temps processeur Average_%|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Inodes utilisés Average_%|Oui|% Used Inodes|Count|Average|Inodes utilisés Average_%|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_% Used Memory|Oui|% Used Memory|Count|Average|Average_% Used Memory|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Inodes utilisés Average_%|Oui|% Used Space|Count|Average|Inodes utilisés Average_%|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_% Used Swap Space|Oui|% Used Swap Space|Count|Average|Average_% Used Swap Space|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_% User Time|Oui|% User Time|Count|Average|Average_% User Time|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Available MBytes|Oui|Nombre d’octets disponibles|Count|Average|Average_Available MBytes|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Available MBytes Memory|Oui|Available MBytes Memory|Count|Average|Average_Available MBytes Memory|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Available MBytes Swap|Oui|Available MBytes Swap|Count|Average|Average_Available MBytes Swap|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Avg. Disk sec/Read|Oui|Avg. Disk sec/Read|Count|Average|Average_Avg. Disk sec/Read|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Avg. Disk sec/Transfer|Oui|Avg. Disk sec/Transfer|Count|Average|Average_Avg. Disk sec/Transfer|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Avg. Disk sec/Write|Oui|Avg. Disk sec/Write|Count|Average|Average_Avg. Disk sec/Write|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Bytes Received/sec|Oui|Octets reçus/s|Count|Average|Average_Bytes Received/sec|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Bytes Sent/sec|Oui|Octets envoyés/s|Count|Average|Average_Bytes Sent/sec|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Bytes Total/sec|Oui|Nombre total d’octets/s|Count|Average|Average_Bytes Total/sec|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Longueur de file d’attente du disque Average_Current|Oui|Longueur actuelle de la file d'attente du disque|Count|Average|Longueur de file d’attente du disque Average_Current|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Octets de lecture/s Average_Disk|Oui|Nb d’octets de lecture de disque/s|Count|Average|Octets de lecture/s Average_Disk|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Disk Reads/sec|Oui|Nb d’opérations de lectures de disque/s|Count|Average|Average_Disk Reads/sec|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Disk Transfers/sec|Oui|Disk Transfers/sec|Count|Average|Average_Disk Transfers/sec|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Octets d’écriture/s Average_Disk|Oui|Nb d’octets d’écriture de disque/s|Count|Average|Octets d’écriture/s Average_Disk|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Disk Writes/sec|Oui|Nb d’opération d’écriture de disque/s|Count|Average|Average_Disk Writes/sec|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Mégaoctets Average_Free|Oui|Free Megabytes|Count|Average|Mégaoctets Average_Free|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Free Physical Memory|Oui|Free Physical Memory|Count|Average|Average_Free Physical Memory|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Free Space in Paging Files|Oui|Free Space in Paging Files|Count|Average|Average_Free Space in Paging Files|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Free Virtual Memory|Oui|Free Virtual Memory|Count|Average|Average_Free Virtual Memory|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Logical Disk Bytes/sec|Oui|Logical Disk Bytes/sec|Count|Average|Average_Logical Disk Bytes/sec|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Page Reads/sec|Oui|Page Reads/sec|Count|Average|Average_Page Reads/sec|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Page Writes/sec|Oui|Page Writes/sec|Count|Average|Average_Page Writes/sec|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Pages/sec|Oui|Pages/sec|Count|Average|Average_Pages/sec|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Pct Privileged Time|Oui|Pct Privileged Time|Count|Average|Average_Pct Privileged Time|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Pct User Time|Oui|Pct User Time|Count|Average|Average_Pct User Time|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Physical Disk Bytes/sec|Oui|Physical Disk Bytes/sec|Count|Average|Average_Physical Disk Bytes/sec|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Processes|Oui|Processus|Count|Average|Average_Processes|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Longueur de la file d’attente Average_Processor|Oui|Longueur de la file d’attente du processeur|Count|Average|Longueur de la file d’attente Average_Processor|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Size Stored In Paging Files|Oui|Size Stored In Paging Files|Count|Average|Average_Size Stored In Paging Files|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Total Bytes|Oui|Total Bytes|Count|Average|Average_Total Bytes|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Total Bytes Received|Oui|Total Bytes Received|Count|Average|Average_Total Bytes Received|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Total Bytes Transmitted|Oui|Total Bytes Transmitted|Count|Average|Average_Total Bytes Transmitted|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Total Collisions|Oui|Total Collisions|Count|Average|Average_Total Collisions|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Total Bytes Received|Oui|Total Packets Received|Count|Average|Average_Total Bytes Received|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Total Packets Transmitted|Oui|Total Packets Transmitted|Count|Average|Average_Total Packets Transmitted|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Total Rx Errors|Oui|Total Rx Errors|Count|Average|Average_Total Rx Errors|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Total Rx Errors|Oui|Total Tx Errors|Count|Average|Average_Total Rx Errors|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Uptime|Oui|Uptime|Count|Average|Average_Uptime|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Used MBytes Swap Space|Oui|Used MBytes Swap Space|Count|Average|Average_Used MBytes Swap Space|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Used Memory MBytes|Oui|Used Memory kBytes|Count|Average|Average_Used Memory MBytes|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Used Memory MBytes|Oui|Used Memory MBytes|Count|Average|Average_Used Memory MBytes|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Users|Oui|Utilisateurs|Count|Average|Average_Users|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Average_Virtual Shared Memory|Oui|Virtual Shared Memory|Count|Average|Average_Virtual Shared Memory|Computer, ObjectName, InstanceName, CounterPath, SourceSystem|
|Événement|Oui|Événement|Count|Average|Événement|Source, EventLog, Computer, EventCategory, EventLevel, EventLevelName, EventID|
|Heartbeat|Oui|Heartbeat|Count|Total|Heartbeat|Computer, OSType, Version, SourceComputerId|
|Update|Oui|Update|Count|Average|Update|Computer, Product, Classification, UpdateState, Optional, Approved|


## <a name="microsoftpeeringpeerings"></a>Microsoft.Peering/peerings

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|EgressTrafficRate|Oui|Débit du trafic de sortie|BitsPerSecond|Average|Débit du trafic de sortie en bits par seconde|ConnectionId|
|IngressTrafficRate|Oui|Débit du trafic d’entrée|BitsPerSecond|Average|Débit du trafic d’entrée en bits par seconde|ConnectionId|
|SessionAvailabilityV4|Oui|Disponibilité de la session V4|Pourcentage|Average|Disponibilité de la session V4|ConnectionId|
|SessionAvailabilityV6|Oui|Disponibilité de la session V6|Pourcentage|Average|Disponibilité de la session V6|ConnectionId|


## <a name="microsoftpeeringpeeringservices"></a>Microsoft.Peering/peeringServices

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|PrefixLatency|Oui|Latence du préfixe|Millisecondes|Average|Latence du préfixe médiane|PrefixName|


## <a name="microsoftpowerbidedicatedcapacities"></a>Microsoft.PowerBIDedicated/capacities

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|memory_metric|Oui|Mémoire|Octets|Average|Mémoire. Plage de 0-3 Go pour A1, de 0-5 Go pour A2, de 0-10 Go pour A3, de 0-25 Go pour A4, de 0-50 Go pour A5 et de 0-100 Go pour A6|Aucune dimension|
|memory_thrashing_metric|Oui|Vidage de mémoire (jeux de données)|Pourcentage|Average|Vidage de mémoire moyenne.|Aucune dimension|
|qpu_high_utilization_metric|Oui|Utilisation élevée du QPU|Count|Total|Utilisation élevée du QPU pendant la dernière minute, 1 pour une utilisation élevée du QPU, sinon 0|Aucune dimension|
|QueryDuration|Oui|Durée de la requête (jeux de données)|Millisecondes|Average|Durée de la requête DAX dans le dernier intervalle|Aucune dimension|
|QueryPoolJobQueueLength|Oui|Longueur de la file d’attente des travaux du pool de requêtes (jeux de données)|Count|Average|Nombre de travaux contenus dans la file d’attente du pool de threads de requêtes.|Aucune dimension|


## <a name="microsoftrelaynamespaces"></a>Microsoft.Relay/namespaces

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|ActiveConnections|Non|ActiveConnections|Count|Total|Nombre total d’ActiveConnections pour Microsoft.Relay.|EntityName|
|ActiveListeners|Non|ActiveListeners|Count|Total|Nombre total d’ActiveListeners pour Microsoft.Relay.|EntityName|
|BytesTransferred|Oui|BytesTransferred|Count|Total|Nombre total de BytesTransferred pour Microsoft.Relay.|EntityName|
|ListenerConnections-ClientError|Non|ListenerConnections-ClientError|Count|Total|ClientError sur ListenerConnections pour Microsoft.Relay.|EntityName, |
|ListenerConnections-ServerError|Non|ListenerConnections-ServerError|Count|Total|ServerError sur ListenerConnections pour Microsoft.Relay.|EntityName, |
|ListenerConnections-Success|Non|ListenerConnections-Success|Count|Total|ListenerConnections ayant abouti pour Microsoft.Relay.|EntityName, |
|ListenerConnections-TotalRequests|Non|ListenerConnections-TotalRequests|Count|Total|Nombre total de ListenerConnections pour Microsoft.Relay.|EntityName|
|ListenerDisconnects|Non|ListenerDisconnects|Count|Total|Nombre total de ListenerDisconnects pour Microsoft.Relay.|EntityName|
|SenderConnections-ClientError|Non|SenderConnections-ClientError|Count|Total|ClientError sur SenderConnections pour Microsoft.Relay.|EntityName, |
|SenderConnections-ServerError|Non|SenderConnections-ServerError|Count|Total|ServerError sur SenderConnections pour Microsoft.Relay.|EntityName, |
|SenderConnections-Success|Non|SenderConnections-Success|Count|Total|SenderConnections ayant abouti pour Microsoft.Relay.|EntityName, |
|SenderConnections-TotalRequests|Non|SenderConnections-TotalRequests|Count|Total|Nombre total de demandes SenderConnections pour Microsoft.Relay.|EntityName|
|SenderDisconnects|Non|SenderDisconnects|Count|Total|Nombre total de SenderDisconnects pour Microsoft.Relay.|EntityName|


## <a name="microsoftsearchsearchservices"></a>Microsoft.Search/searchServices

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|SearchLatency|Oui|Latence de recherche|Secondes|Average|Latence moyenne de recherche du service de recherche|Aucune dimension|
|SearchQueriesPerSecond|Oui|Requêtes de recherche par seconde|CountPerSecond|Average|Requêtes de recherche par seconde pour le service de recherche|Aucune dimension|
|ThrottledSearchQueriesPercentage|Oui|Pourcentage de requêtes de recherche limitées|Pourcentage|Average|Pourcentage de requêtes de recherche limitées par le service de recherche|Aucune dimension|


## <a name="microsoftservicebusnamespaces"></a>Microsoft.ServiceBus/namespaces

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|ActiveConnections|Non|ActiveConnections|Count|Total|Nombre total de connexions actives pour Microsoft.ServiceBus.|Aucune dimension|
|ActiveMessages|Non|Nombre de messages actifs dans une file d’attente/rubrique.|Count|Average|Nombre de messages actifs dans une file d’attente/rubrique.|EntityName|
|ConnectionsClosed|Non|Connexions fermées.|Count|Average|Connexions fermées pour Microsoft.ServiceBus.|EntityName|
|ConnectionsOpened|Non|Connexions ouvertes.|Count|Average|Connexions ouvertes pour Microsoft.ServiceBus.|EntityName|
|CPUXNS|Non|Processeur (déprécié)|Pourcentage|Maximale|Métrique d’utilisation du processeur de l’espace de noms Service Bus Premium. Cette métrique est déconseillée. Utilisez la métrique de processeur (NamespaceCpuUsage) à la place.|Aucune dimension|
|DeadletteredMessages|Non|Nombre de messages de lettres mortes dans une file d’attente/rubrique.|Count|Average|Nombre de messages de lettres mortes dans une file d’attente/rubrique.|EntityName|
|IncomingMessages|Oui|Messages entrants|Count|Total|Messages entrants pour Microsoft.ServiceBus.|EntityName|
|IncomingRequests|Oui|Demandes entrantes|Count|Total|Demandes entrantes pour Microsoft.ServiceBus.|EntityName|
|Messages|Non|Nombre de messages dans une file d’attente/rubrique.|Count|Average|Nombre de messages dans une file d’attente/rubrique.|EntityName|
|NamespaceCpuUsage|Non|UC|Pourcentage|Maximale|Métrique d’utilisation du processeur de l’espace de noms Service Bus Premium.|Aucune dimension|
|NamespaceMemoryUsage|Non|Utilisation de la mémoire|Pourcentage|Maximale|Métrique d’utilisation de la mémoire de l’espace de noms Service Bus Premium|Aucune dimension|
|OutgoingMessages|Oui|Messages sortants|Count|Total|Messages sortants pour Microsoft.ServiceBus.|EntityName|
|ScheduledMessages|Non|Nombre de messages planifiés dans une file d’attente/rubrique.|Count|Average|Nombre de messages planifiés dans une file d’attente/rubrique.|EntityName|
|ServerErrors|Non|Erreurs de serveur.|Count|Total|Erreurs de serveur pour Microsoft.ServiceBus.|EntityName, |
|Taille|Non|Taille|Octets|Average|Taille d’une file d’attente/rubrique en octets.|EntityName|
|SuccessfulRequests|Non|Requêtes ayant réussi|Count|Total|Nombre total de demandes ayant réussi pour un espace de noms|EntityName, |
|ThrottledRequests|Non|Demandes limitées.|Count|Total|Demandes limitées pour Microsoft.ServiceBus.|EntityName, |
|UserErrors|Non|Erreurs d’utilisateur.|Count|Total|Erreurs d’utilisateur pour Microsoft.ServiceBus.|EntityName, |
|WSXNS|Non|Utilisation de la mémoire (déprécié)|Pourcentage|Maximale|Métrique d’utilisation de la mémoire de l’espace de noms Service Bus Premium Cette métrique est déconseillée. Utilisez la métrique d’utilisation de la mémoire (NamespaceMemoryUsage) à la place.|Aucune dimension|


## <a name="microsoftservicefabricmeshapplications"></a>Microsoft.ServiceFabricMesh/applications

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|ActualCpu|Non|ActualCpu|Count|Average|Utilisation réelle du processeur en millicoeurs|ApplicationName, ServiceName, CodePackageName, ServiceReplicaName|
|ActualMemory|Non|ActualMemory|Octets|Average|Utilisation réelle de la mémoire en octets|ApplicationName, ServiceName, CodePackageName, ServiceReplicaName|
|AllocatedCpu|Non|AllocatedCpu|Count|Average|Processeur alloué à ce conteneur en millicoeurs|ApplicationName, ServiceName, CodePackageName, ServiceReplicaName|
|AllocatedMemory|Non|AllocatedMemory|Octets|Average|Mémoire allouée à ce conteneur en Mo|ApplicationName, ServiceName, CodePackageName, ServiceReplicaName|
|ApplicationStatus|Non|ApplicationStatus|Count|Average|État de l'application Service Fabric Mesh|ApplicationName, Status|
|ContainerStatus|Non|ContainerStatus|Count|Average|État du conteneur dans l'application Service Fabric Mesh|ApplicationName, ServiceName, CodePackageName, ServiceReplicaName, Status|
|CpuUtilization|Non|CpuUtilization|Pourcentage|Average|Utilisation du processeur pour ce conteneur en pourcentage de AllocatedCpu|ApplicationName, ServiceName, CodePackageName, ServiceReplicaName|
|MemoryUtilization|Non|MemoryUtilization|Pourcentage|Average|Utilisation du processeur pour ce conteneur en pourcentage de AllocatedCpu|ApplicationName, ServiceName, CodePackageName, ServiceReplicaName|
|RestartCount|Non|RestartCount|Count|Average|Nombre de redémarrage d'un conteneur dans l'application Service Fabric Mesh|ApplicationName, Status, ServiceName, ServiceReplicaName, CodePackageName|
|ServiceReplicaStatus|Non|ServiceReplicaStatus|Count|Average|État d’intégrité d’un réplica de service dans l'application Service Fabric Mesh|ApplicationName, Status, ServiceName, ServiceReplicaName|
|ServiceStatus|Non|ServiceStatus|Count|Average|État d’intégrité d’un service dans l'application Service Fabric Mesh|ApplicationName, Status, ServiceName|


## <a name="microsoftsignalrservicesignalr"></a>Microsoft.SignalRService/SignalR

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|ConnectionCount|Oui|Nombre de connexions|Count|Maximale|Nombre de connexions utilisateur|Point de terminaison|
|InboundTraffic|Oui|Trafic entrant|Octets|Total|Trafic entrant de service|Aucune dimension|
|MessageCount|Oui|Nombre de messages|Count|Total|Nombre total de messages.|Aucune dimension|
|OutboundTraffic|Oui|Trafic sortant|Octets|Total|Trafic sortant de service|Aucune dimension|
|SystemErrors|Oui|Erreurs système|Pourcentage|Maximale|Pourcentage d’erreurs système|Aucune dimension|
|UserErrors|Oui|Erreurs d’utilisateur|Pourcentage|Maximale|Pourcentage d’erreurs d’utilisateur|Aucune dimension|


## <a name="microsoftsqlmanagedinstances"></a>Microsoft.Sql/managedInstances

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|avg_cpu_percent|Oui|Pourcentage d’UC moyenne|Pourcentage|Average|Pourcentage d’UC moyenne|Aucune dimension|
|io_bytes_read|Oui|Octets d’E/S lus|Octets|Average|Octets d’E/S lus|Aucune dimension|
|io_bytes_written|Oui|Octets d’E/S écrits|Octets|Average|Octets d’E/S écrits|Aucune dimension|
|io_requests|Oui|Nombre de requêtes d’E/S|Count|Average|Nombre de requêtes d’E/S|Aucune dimension|
|reserved_storage_mb|Oui|Espace de stockage réservé|Count|Average|Espace de stockage réservé|Aucune dimension|
|storage_space_used_mb|Oui|Espace de stockage utilisé|Count|Average|Espace de stockage utilisé|Aucune dimension|
|virtual_core_count|Oui|Nombre de cœurs virtuels|Count|Average|Nombre de cœurs virtuels|Aucune dimension|


## <a name="microsoftsqlserversdatabases"></a>Microsoft.Sql/servers/databases

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|active_queries|Oui|Requêtes actives|Count|Total|Requêtes actives sur tous les groupes de charges de travail. S’applique uniquement aux entrepôts de données.|Aucune dimension|
|allocated_data_storage|Oui|Espace de données alloué|Octets|Average|Stockage des données alloué. Non applicable aux entrepôts de données.|Aucune dimension|
|app_cpu_billed|Oui|Processeur d'application facturé|Count|Total|Processeur d’application facturé. S’applique aux bases de données serverless.|Aucune dimension|
|app_cpu_percent|Oui|Pourcentage processeur d'application|Pourcentage|Average|Pourcentage processeur d'application. S’applique aux bases de données serverless.|Aucune dimension|
|app_memory_percent|Oui|Pourcentage de mémoire de l’application|Pourcentage|Average|Pourcentage de mémoire de l’application. S’applique aux bases de données serverless.|Aucune dimension|
|base_blob_size_bytes|Oui|Taille de stockage d’objet blob de base|Octets|Maximale|Taille de stockage d’objet blob de base. S’applique aux bases de données Hyperscale.|Aucune dimension|
|blocked_by_firewall|Oui|Bloqué par le pare-feu|Count|Total|Bloqué par le pare-feu|Aucune dimension|
|cache_hit_percent|Oui|Pourcentage d’accès au cache|Pourcentage|Maximale|Pourcentage d’accès au cache. S’applique uniquement aux entrepôts de données.|Aucune dimension|
|cache_used_percent|Oui|Pourcentage de cache utilisé|Pourcentage|Maximale|Pourcentage de cache utilisé. S’applique uniquement aux entrepôts de données.|Aucune dimension|
|connection_failed|Oui|Connexions ayant échoué|Count|Total|Connexions ayant échoué|Aucune dimension|
|connection_successful|Oui|Connexions réussies|Count|Total|Connexions réussies|Aucune dimension|
|cpu_limit|Oui|Limite UC|Count|Average|Limite UC. S’applique aux bases de données basées sur vCore.|Aucune dimension|
|cpu_percent|Oui|Pourcentage UC|Pourcentage|Average|Pourcentage UC|Aucune dimension|
|cpu_used|Oui|UC utilisée|Count|Average|UC utilisée. S’applique aux bases de données basées sur vCore.|Aucune dimension|
|deadlock|Oui|Blocages|Count|Total|Interblocages. Non applicable aux entrepôts de données.|Aucune dimension|
|diff_backup_size_bytes|Oui|Taille de stockage de sauvegarde différentielle|Octets|Maximale|Taille de stockage de sauvegarde différentielle cumulée. S’applique aux bases de données basées sur vCore. Ne s'applique pas aux bases de données Hyperscale.|Aucune dimension|
|dtu_consumption_percent|Oui|Pourcentage DTU|Pourcentage|Average|Pourcentage DTU. S’applique aux bases de données basées sur DTU.|Aucune dimension|
|dtu_limit|Oui|Limite DTU|Count|Average|Limite DTU. S’applique aux bases de données basées sur DTU.|Aucune dimension|
|dtu_used|Oui|DTU utilisé|Count|Average|DTU utilisé. S’applique aux bases de données basées sur DTU.|Aucune dimension|
|dw_backup_size_gb|Oui|Taille du stockage de données (en Go)|Count|Total|La taille de stockage des données comprend la taille de vos données et le journal des transactions. La métrique est comptabilisée dans la partie « Stockage » de votre facture. S’applique uniquement aux entrepôts de données.|Aucune dimension|
|dw_geosnapshot_size_gb|Oui|Taille de stockage de récupération d’urgence (en Go)|Count|Total|La taille de stockage de récupération d’urgence apparaît comme « Stockage de récupération d’urgence » sur votre facture. S’applique uniquement aux entrepôts de données.|Aucune dimension|
|dw_snapshot_size_gb|Oui|Taille du stockage des captures instantanées (en Go)|Count|Total|La taille du stockage des captures instantanées correspond à la taille des modifications incrémentielles capturées par les captures instantanées pour créer des points de restauration automatiques et définis par l’utilisateur. La métrique est comptabilisée dans la partie « Stockage » de votre facture. S’applique uniquement aux entrepôts de données.|Aucune dimension|
|dwu_consumption_percent|Oui|Pourcentage DWU|Pourcentage|Maximale|Pourcentage DWU. S’applique uniquement aux entrepôts de données.|Aucune dimension|
|dwu_limit|Oui|Limite DWU|Count|Maximale|Limite DWU. S’applique uniquement aux entrepôts de données.|Aucune dimension|
|dwu_used|Oui|DWU utilisé|Count|Maximale|DWU utilisé. S’applique uniquement aux entrepôts de données.|Aucune dimension|
|full_backup_size_bytes|Oui|Taille de stockage de sauvegarde complète|Octets|Maximale|Taille de stockage de sauvegarde complète cumulée. S’applique aux bases de données basées sur vCore. Ne s'applique pas aux bases de données Hyperscale.|Aucune dimension|
|local_tempdb_usage_percent|Oui|Pourcentage de tempdb locale|Pourcentage|Average|Pourcentage de tempdb locale. S’applique uniquement aux entrepôts de données.|Aucune dimension|
|log_backup_size_bytes|Oui|Taille de stockage de fichier journal|Octets|Maximale|Taille de stockage de fichier journal cumulée. S’applique aux bases de données vCore et Hyperscale.|Aucune dimension|
|log_write_percent|Oui|Pourcentage E/S du journal|Pourcentage|Average|Pourcentage E/S du journal. Non applicable aux entrepôts de données.|Aucune dimension|
|memory_usage_percent|Oui|Pourcentage de mémoire|Pourcentage|Maximale|Pourcentage de mémoire. S’applique uniquement aux entrepôts de données.|Aucune dimension|
|physical_data_read_percent|Oui|Pourcentage E/S des données|Pourcentage|Average|Pourcentage E/S des données|Aucune dimension|
|queued_queries|Oui|Requêtes mises en file d’attente|Count|Total|Requêtes mises en file d’attente sur tous les groupes de charges de travail. S’applique uniquement aux entrepôts de données.|Aucune dimension|
|sessions_percent|Oui|Pourcentage de sessions|Pourcentage|Average|Pourcentage de sessions. Non applicable aux entrepôts de données.|Aucune dimension|
|snapshot_backup_size_bytes|Oui|Taille de stockage de sauvegarde instantanée|Octets|Maximale|Taille de stockage de sauvegarde instantanée cumulée. S’applique aux bases de données Hyperscale.|Aucune dimension|
|sqlserver_process_core_percent|Oui|Pourcentage de cœurs de processus SQL Server|Pourcentage|Maximale|Utilisation du processeur en pourcentage du processus SQL DB. Non applicable aux entrepôts de données.|Aucune dimension|
|sqlserver_process_memory_percent|Oui|Pourcentage de mémoire de processus SQL Server|Pourcentage|Maximale|Utilisation de la mémoire en pourcentage du processus SQL DB. Non applicable aux entrepôts de données.|Aucune dimension|
|storage|Oui|Espace de données utilisé|Octets|Maximale|Espace de données utilisé. Non applicable aux entrepôts de données.|Aucune dimension|
|storage_percent|Oui|Pourcentage d'espace de données utilisé|Pourcentage|Maximale|Pourcentage d’espace de données utilisé. Non applicable aux entrepôts de données ou base de données hyperscale.|Aucune dimension|
|tempdb_data_size|Oui|Taille du fichier de données tempdb en kilo-octets|Count|Maximale|Taille du fichier de données tempdb en kilo-octets. Non applicable aux entrepôts de données.|Aucune dimension|
|tempdb_log_size|Oui|Taille du fichier journal de tempdb en kilo-octets|Count|Maximale|Taille du fichier journal de tempdb en kilo-octets. Non applicable aux entrepôts de données.|Aucune dimension|
|tempdb_log_used_percent|Oui|Pourcentage d’utilisation du journal tempdb|Pourcentage|Maximale|Pourcentage d’utilisation du journal tempdb. Non applicable aux entrepôts de données.|Aucune dimension|
|wlg_active_queries|Oui|Requêtes actives du groupe de charge de travail|Count|Total|Requêtes actives dans le groupe de charges de travail. S’applique uniquement aux entrepôts de données.|WorkloadGroupName, IsUserDefined|
|wlg_active_queries_timeouts|Oui|Délais d’expiration des requêtes du groupe de charge de travail|Count|Total|Requêtes ayant dépassé le délai d’attente pour le groupe de charges de travail. S’applique uniquement aux entrepôts de données.|WorkloadGroupName, IsUserDefined|
|wlg_allocation_relative_to_system_percent|Oui|Allocation du groupe de charge de travail par pourcentage système|Pourcentage|Maximale|Pourcentage alloué de ressources par rapport à l’ensemble du système par groupe de charges de travail. S’applique uniquement aux entrepôts de données.|WorkloadGroupName, IsUserDefined|
|wlg_allocation_relative_to_wlg_effective_cap_percent|Oui|Allocation de groupe de charges de travail par pourcentage de ressources limitées|Pourcentage|Maximale|Pourcentage alloué de ressources par rapport aux ressources limitées indiquées par groupe de charges de travail. S’applique uniquement aux entrepôts de données.|WorkloadGroupName, IsUserDefined|
|wlg_effective_cap_resource_percent|Oui|Pourcentage de ressources limitées réelles|Pourcentage|Maximale|Limite inconditionnelle sur le pourcentage de ressources autorisées pour le groupe de charges de travail, en tenant compte du pourcentage minimal de ressources réelles alloué pour d’autres groupes de charges de travail. S’applique uniquement aux entrepôts de données.|WorkloadGroupName, IsUserDefined|
|wlg_effective_min_resource_percent|Oui|Pourcentage minimal de ressources réelles|Pourcentage|Maximale|Pourcentage minimal de ressources réservées et isolées pour le groupe de charges de travail, en tenant compte du niveau de service minimum. S’applique uniquement aux entrepôts de données.|WorkloadGroupName, IsUserDefined|
|wlg_queued_queries|Oui|Requêtes en file d’attente du groupe de charge de travail|Count|Total|Requêtes mises en file d’attente dans le groupe de charges de travail. S’applique uniquement aux entrepôts de données.|WorkloadGroupName, IsUserDefined|
|workers_percent|Oui|Pourcentage de travaux|Pourcentage|Average|Pourcentage de Workers. Non applicable aux entrepôts de données.|Aucune dimension|
|xtp_storage_percent|Oui|Pourcentage de stockage OLTP en mémoire|Pourcentage|Average|Pourcentage de stockage OLTP en mémoire. Non applicable aux entrepôts de données.|Aucune dimension|


## <a name="microsoftsqlserverselasticpools"></a>Microsoft.Sql/servers/elasticPools

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|allocated_data_storage|Oui|Espace de données alloué|Octets|Average|Espace de données alloué|Aucune dimension|
|allocated_data_storage_percent|Oui|Pourcentage d'espace de données alloué|Pourcentage|Maximale|Pourcentage d'espace de données alloué|Aucune dimension|
|cpu_limit|Oui|Limite UC|Count|Average|Limite UC. S’applique aux pools élastiques basés sur vCore.|Aucune dimension|
|cpu_percent|Oui|Pourcentage UC|Pourcentage|Average|Pourcentage UC|Aucune dimension|
|cpu_used|Oui|UC utilisée|Count|Average|UC utilisée. S’applique aux pools élastiques basés sur vCore.|Aucune dimension|
|database_allocated_data_storage|Non|Espace de données alloué|Octets|Average|Espace de données alloué|DatabaseResourceId|
|database_cpu_limit|Non|Limite UC|Count|Average|Limite UC|DatabaseResourceId|
|database_cpu_percent|Non|Pourcentage UC|Pourcentage|Average|Pourcentage UC|DatabaseResourceId|
|database_cpu_used|Non|UC utilisée|Count|Average|UC utilisée|DatabaseResourceId|
|database_dtu_consumption_percent|Non|Pourcentage DTU|Pourcentage|Average|Pourcentage DTU|DatabaseResourceId|
|database_eDTU_used|Non|eDTU utilisé|Count|Average|eDTU utilisé|DatabaseResourceId|
|database_log_write_percent|Non|Pourcentage E/S du journal|Pourcentage|Average|Pourcentage E/S du journal|DatabaseResourceId|
|database_physical_data_read_percent|Non|Pourcentage E/S des données|Pourcentage|Average|Pourcentage E/S des données|DatabaseResourceId|
|database_sessions_percent|Non|Pourcentage de sessions|Pourcentage|Average|Pourcentage de sessions|DatabaseResourceId|
|database_storage_used|Non|Espace de données utilisé|Octets|Average|Espace de données utilisé|DatabaseResourceId|
|database_workers_percent|Non|Pourcentage de travaux|Pourcentage|Average|Pourcentage de travaux|DatabaseResourceId|
|dtu_consumption_percent|Oui|Pourcentage DTU|Pourcentage|Average|Pourcentage DTU. S’applique aux pools élastiques basés sur DTU.|Aucune dimension|
|eDTU_limit|Oui|Limite eDTU|Count|Average|Limite eDTU. S’applique aux pools élastiques basés sur DTU.|Aucune dimension|
|eDTU_used|Oui|eDTU utilisé|Count|Average|eDTU utilisé. S’applique aux pools élastiques basés sur DTU.|Aucune dimension|
|log_write_percent|Oui|Pourcentage E/S du journal|Pourcentage|Average|Pourcentage E/S du journal|Aucune dimension|
|physical_data_read_percent|Oui|Pourcentage E/S des données|Pourcentage|Average|Pourcentage E/S des données|Aucune dimension|
|sessions_percent|Oui|Pourcentage de sessions|Pourcentage|Average|Pourcentage de sessions|Aucune dimension|
|sqlserver_process_core_percent|Oui|Pourcentage de cœurs de processus SQL Server|Pourcentage|Maximale|Utilisation du processeur en pourcentage du processus SQL DB. S’applique aux pools élastiques.|Aucune dimension|
|sqlserver_process_memory_percent|Oui|Pourcentage de mémoire de processus SQL Server|Pourcentage|Maximale|Utilisation de la mémoire en pourcentage du processus SQL DB. S’applique aux pools élastiques.|Aucune dimension|
|storage_limit|Oui|Taille maximale des données|Octets|Average|Taille maximale des données|Aucune dimension|
|storage_percent|Oui|Pourcentage d'espace de données utilisé|Pourcentage|Average|Pourcentage d'espace de données utilisé|Aucune dimension|
|storage_used|Oui|Espace de données utilisé|Octets|Average|Espace de données utilisé|Aucune dimension|
|tempdb_data_size|Oui|Taille du fichier de données tempdb en kilo-octets|Count|Maximale|Taille du fichier de données tempdb en kilo-octets|Aucune dimension|
|tempdb_log_size|Oui|Taille du fichier journal de tempdb en kilo-octets|Count|Maximale|Taille du fichier journal de tempdb en kilo-octets|Aucune dimension|
|tempdb_log_used_percent|Oui|Pourcentage d’utilisation du journal tempdb|Pourcentage|Maximale|Pourcentage d’utilisation du journal tempdb|Aucune dimension|
|workers_percent|Oui|Pourcentage de travaux|Pourcentage|Average|Pourcentage de travaux|Aucune dimension|
|xtp_storage_percent|Oui|Pourcentage de stockage OLTP en mémoire|Pourcentage|Average|Pourcentage de stockage OLTP en mémoire|Aucune dimension|


## <a name="microsoftstoragestorageaccounts"></a>Microsoft.Storage/storageAccounts

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|Disponibilité|Oui|Disponibilité|Pourcentage|Average|Pourcentage de disponibilité pour le service de stockage ou l’opération API spécifiée. La disponibilité est calculée en prenant la valeur TotalBillableRequests puis en la divisant par le nombre de requêtes applicables, y compris celles qui ont généré des erreurs inattendues. Toutes erreurs inattendues réduisent la disponibilité du service de stockage ou de l’opération API spécifiée.|GeoType, ApiName, Authentication|
|Sortie|Oui|Sortie|Octets|Total|Quantité de données de sortie. Ce volume inclut les sorties vers un client externe provenant du Stockage Azure ainsi que les sorties dans Azure. Par conséquent, ce nombre ne reflète pas les sorties facturables.|GeoType, ApiName, Authentication|
|Entrée|Oui|Entrée|Octets|Total|Quantité de données d’entrée, en octets. Ce nombre inclut les entrées d’un client externe dans Stockage Microsoft Azure ainsi que les entrées dans Azure.|GeoType, ApiName, Authentication|
|SuccessE2ELatency|Oui|Latence E2E de réussite|Millisecondes|Average|Latence moyenne de bout en bout des requêtes réussies envoyées à un service de stockage ou à l’opération API spécifiée, en millisecondes. Cette valeur inclut le temps de traitement requis au sein de Stockage Microsoft Azure pour lire la requête, envoyer la réponse et recevoir un accusé de réception de la réponse.|GeoType, ApiName, Authentication|
|SuccessServerLatency|Oui|Latence du serveur avec requête réussie|Millisecondes|Average|Durée moyenne utilisée pour traiter une requête réussie par Stockage Azure. Cette valeur n’inclut pas la latence réseau spécifiée dans SuccessE2ELatency.|GeoType, ApiName, Authentication|
|Transactions|Oui|Transactions|Count|Total|Nombre de requêtes envoyées à un service de stockage ou à l’opération API spécifiée. Ce nombre inclut les requêtes réussies et celles ayant échoué, ainsi que les requêtes qui ont généré des erreurs. Utilisez la dimension ResponseType pour connaître le nombre des différents types de réponses.|ResponseType, GeoType, ApiName, Authentication|
|UsedCapacity|Non|Capacité utilisée|Octets|Average|Quantité de stockage utilisée par le compte de stockage. Pour les comptes de stockage standard, il s’agit de la somme de la capacité utilisée par les objets blob, tables, fichiers et files d’attente. Pour les comptes de Stockage Premium et Blob, elle équivaut à BlobCapacity ou à FileCapacity.|Aucune dimension|


## <a name="microsoftstoragestorageaccountsblobservices"></a>Microsoft.Storage/storageAccounts/blobServices

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|Disponibilité|Oui|Disponibilité|Pourcentage|Average|Pourcentage de disponibilité pour le service de stockage ou l’opération API spécifiée. La disponibilité est calculée en prenant la valeur TotalBillableRequests puis en la divisant par le nombre de requêtes applicables, y compris celles qui ont généré des erreurs inattendues. Toutes erreurs inattendues réduisent la disponibilité du service de stockage ou de l’opération API spécifiée.|GeoType, ApiName, Authentication|
|BlobCapacity|Non|Capacité d’objet blob|Octets|Average|Quantité de stockage utilisée par le service BLOB du compte de stockage, en octets.|BlobType, Tier|
|BlobCount|Non|Nombre d’objets blob|Count|Average|Nombre d’objets blob stockés dans le compte de stockage.|BlobType, Tier|
|BlobProvisionedSize|Non|Taille provisionnée du service blob|Octets|Average|Quantité de stockage provisionnée dans le service blob du compte de stockage, en octets|BlobType, Tier|
|ContainerCount|Oui|Nombre de conteneurs d’objets blob|Count|Average|Nombre de conteneurs dans le compte de stockage.|Aucune dimension|
|Sortie|Oui|Sortie|Octets|Total|Quantité de données de sortie. Ce volume inclut les sorties vers un client externe provenant du Stockage Azure ainsi que les sorties dans Azure. Par conséquent, ce nombre ne reflète pas les sorties facturables.|GeoType, ApiName, Authentication|
|IndexCapacity|Non|Capacité d'index|Octets|Average|Quantité de stockage utilisée par l’index hiérarchique Azure Data Lake Storage Gen2|Aucune dimension|
|Entrée|Oui|Entrée|Octets|Total|Quantité de données d’entrée, en octets. Ce nombre inclut les entrées d’un client externe dans Stockage Microsoft Azure ainsi que les entrées dans Azure.|GeoType, ApiName, Authentication|
|SuccessE2ELatency|Oui|Latence E2E de réussite|Millisecondes|Average|Latence moyenne de bout en bout des requêtes réussies envoyées à un service de stockage ou à l’opération API spécifiée, en millisecondes. Cette valeur inclut le temps de traitement requis au sein de Stockage Microsoft Azure pour lire la requête, envoyer la réponse et recevoir un accusé de réception de la réponse.|GeoType, ApiName, Authentication|
|SuccessServerLatency|Oui|Latence du serveur avec requête réussie|Millisecondes|Average|Durée moyenne utilisée pour traiter une requête réussie par Stockage Azure. Cette valeur n’inclut pas la latence réseau spécifiée dans SuccessE2ELatency.|GeoType, ApiName, Authentication|
|Transactions|Oui|Transactions|Count|Total|Nombre de requêtes envoyées à un service de stockage ou à l’opération API spécifiée. Ce nombre inclut les requêtes réussies et celles ayant échoué, ainsi que les requêtes qui ont généré des erreurs. Utilisez la dimension ResponseType pour connaître le nombre des différents types de réponses.|ResponseType, GeoType, ApiName, Authentication|


## <a name="microsoftstoragestorageaccountsfileservices"></a>Microsoft.Storage/storageAccounts/fileServices

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|Disponibilité|Oui|Disponibilité|Pourcentage|Average|Pourcentage de disponibilité pour le service de stockage ou l’opération API spécifiée. La disponibilité est calculée en prenant la valeur TotalBillableRequests puis en la divisant par le nombre de requêtes applicables, y compris celles qui ont généré des erreurs inattendues. Toutes erreurs inattendues réduisent la disponibilité du service de stockage ou de l’opération API spécifiée.|GeoType, ApiName, Authentication, FileShare|
|Sortie|Oui|Sortie|Octets|Total|Quantité de données de sortie. Ce volume inclut les sorties vers un client externe provenant du Stockage Azure ainsi que les sorties dans Azure. Par conséquent, ce nombre ne reflète pas les sorties facturables.|GeoType, ApiName, Authentication, FileShare|
|FileCapacity|Non|Capacité de fichiers|Octets|Average|Quantité de stockage de fichiers utilisée par le compte de stockage.|FileShare|
|FileCount|Non|Nombre de fichiers|Count|Average|Nombre de fichiers dans le compte de stockage.|FileShare|
|FileShareCapacityQuota|Non|Quota de capacité du partage de fichiers|Octets|Average|Limite supérieure de la quantité de stockage pouvant être utilisée par le service Azure Files, en octets.|FileShare|
|FileShareCount|Non|Nombre de partages de fichiers|Count|Average|Nombre de partages de fichiers dans le compte de stockage.|Aucune dimension|
|FileShareProvisionedIOPS|Non|IOPS provisionnées du partage de fichiers|Octets|Average|Nombre de référence d’IOPS provisionnées pour le partage de fichiers Premium dans le compte de stockage de fichiers Premium. Ce nombre est calculé en fonction de la taille provisionnée (quota) de la capacité de partage.|FileShare|
|FileShareSnapshotCount|Non|Nombre d’instantanés de partage de fichiers|Count|Average|Nombre d’instantanés présents sur le partage dans le service Files du compte de stockage.|FileShare|
|FileShareSnapshotSize|Non|Taille d’instantané de partage de fichiers|Octets|Average|Quantité de stockage utilisée par les instantanés dans le service Fichier du compte de stockage, en octets.|FileShare|
|Entrée|Oui|Entrée|Octets|Total|Quantité de données d’entrée, en octets. Ce nombre inclut les entrées d’un client externe dans Stockage Microsoft Azure ainsi que les entrées dans Azure.|GeoType, ApiName, Authentication, FileShare|
|SuccessE2ELatency|Oui|Latence E2E de réussite|Millisecondes|Average|Latence moyenne de bout en bout des requêtes réussies envoyées à un service de stockage ou à l’opération API spécifiée, en millisecondes. Cette valeur inclut le temps de traitement requis au sein de Stockage Microsoft Azure pour lire la requête, envoyer la réponse et recevoir un accusé de réception de la réponse.|GeoType, ApiName, Authentication, FileShare|
|SuccessServerLatency|Oui|Latence du serveur avec requête réussie|Millisecondes|Average|Durée moyenne utilisée pour traiter une requête réussie par Stockage Azure. Cette valeur n’inclut pas la latence réseau spécifiée dans SuccessE2ELatency.|GeoType, ApiName, Authentication, FileShare|
|Transactions|Oui|Transactions|Count|Total|Nombre de requêtes envoyées à un service de stockage ou à l’opération API spécifiée. Ce nombre inclut les requêtes réussies et celles ayant échoué, ainsi que les requêtes qui ont généré des erreurs. Utilisez la dimension ResponseType pour connaître le nombre des différents types de réponses.|ResponseType, GeoType, ApiName, Authentication, FileShare|


## <a name="microsoftstoragestorageaccountsqueueservices"></a>Microsoft.Storage/storageAccounts/queueServices

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|Disponibilité|Oui|Disponibilité|Pourcentage|Average|Pourcentage de disponibilité pour le service de stockage ou l’opération API spécifiée. La disponibilité est calculée en prenant la valeur TotalBillableRequests puis en la divisant par le nombre de requêtes applicables, y compris celles qui ont généré des erreurs inattendues. Toutes erreurs inattendues réduisent la disponibilité du service de stockage ou de l’opération API spécifiée.|GeoType, ApiName, Authentication|
|Sortie|Oui|Sortie|Octets|Total|Quantité de données de sortie. Ce volume inclut les sorties vers un client externe provenant du Stockage Azure ainsi que les sorties dans Azure. Par conséquent, ce nombre ne reflète pas les sorties facturables.|GeoType, ApiName, Authentication|
|Entrée|Oui|Entrée|Octets|Total|Quantité de données d’entrée, en octets. Ce nombre inclut les entrées d’un client externe dans Stockage Microsoft Azure ainsi que les entrées dans Azure.|GeoType, ApiName, Authentication|
|QueueCapacity|Oui|Capacité de la file d’attente|Octets|Average|Quantité de stockage de files d’attente utilisée par le compte de stockage.|Aucune dimension|
|QueueCount|Oui|Nombre de files d’attente|Count|Average|Nombre de files d’attente dans le compte de stockage.|Aucune dimension|
|QueueMessageCount|Oui|Nombre de messages dans la file d’attente|Count|Average|Nombre de messages de file d’attente non expirés dans le compte de stockage.|Aucune dimension|
|SuccessE2ELatency|Oui|Latence E2E de réussite|Millisecondes|Average|Latence moyenne de bout en bout des requêtes réussies envoyées à un service de stockage ou à l’opération API spécifiée, en millisecondes. Cette valeur inclut le temps de traitement requis au sein de Stockage Microsoft Azure pour lire la requête, envoyer la réponse et recevoir un accusé de réception de la réponse.|GeoType, ApiName, Authentication|
|SuccessServerLatency|Oui|Latence du serveur avec requête réussie|Millisecondes|Average|Durée moyenne utilisée pour traiter une requête réussie par Stockage Azure. Cette valeur n’inclut pas la latence réseau spécifiée dans SuccessE2ELatency.|GeoType, ApiName, Authentication|
|Transactions|Oui|Transactions|Count|Total|Nombre de requêtes envoyées à un service de stockage ou à l’opération API spécifiée. Ce nombre inclut les requêtes réussies et celles ayant échoué, ainsi que les requêtes qui ont généré des erreurs. Utilisez la dimension ResponseType pour connaître le nombre des différents types de réponses.|ResponseType, GeoType, ApiName, Authentication|


## <a name="microsoftstoragestorageaccountstableservices"></a>Microsoft.Storage/storageAccounts/tableServices

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|Disponibilité|Oui|Disponibilité|Pourcentage|Average|Pourcentage de disponibilité pour le service de stockage ou l’opération API spécifiée. La disponibilité est calculée en prenant la valeur TotalBillableRequests puis en la divisant par le nombre de requêtes applicables, y compris celles qui ont généré des erreurs inattendues. Toutes erreurs inattendues réduisent la disponibilité du service de stockage ou de l’opération API spécifiée.|GeoType, ApiName, Authentication|
|Sortie|Oui|Sortie|Octets|Total|Quantité de données de sortie. Ce volume inclut les sorties vers un client externe provenant du Stockage Azure ainsi que les sorties dans Azure. Par conséquent, ce nombre ne reflète pas les sorties facturables.|GeoType, ApiName, Authentication|
|Entrée|Oui|Entrée|Octets|Total|Quantité de données d’entrée, en octets. Ce nombre inclut les entrées d’un client externe dans Stockage Microsoft Azure ainsi que les entrées dans Azure.|GeoType, ApiName, Authentication|
|SuccessE2ELatency|Oui|Latence E2E de réussite|Millisecondes|Average|Latence moyenne de bout en bout des requêtes réussies envoyées à un service de stockage ou à l’opération API spécifiée, en millisecondes. Cette valeur inclut le temps de traitement requis au sein de Stockage Microsoft Azure pour lire la requête, envoyer la réponse et recevoir un accusé de réception de la réponse.|GeoType, ApiName, Authentication|
|SuccessServerLatency|Oui|Latence du serveur avec requête réussie|Millisecondes|Average|Durée moyenne utilisée pour traiter une requête réussie par Stockage Azure. Cette valeur n’inclut pas la latence réseau spécifiée dans SuccessE2ELatency.|GeoType, ApiName, Authentication|
|TableCapacity|Oui|Capacité de la table|Octets|Average|Quantité de stockage de tables utilisée par le compte de stockage.|Aucune dimension|
|TableCount|Oui|Nombre de tables|Count|Average|Nombre de tables dans le compte de stockage.|Aucune dimension|
|TableEntityCount|Oui|Nombre d’entités de table|Count|Average|Nombre d’entités de table dans le compte de stockage.|Aucune dimension|
|Transactions|Oui|Transactions|Count|Total|Nombre de requêtes envoyées à un service de stockage ou à l’opération API spécifiée. Ce nombre inclut les requêtes réussies et celles ayant échoué, ainsi que les requêtes qui ont généré des erreurs. Utilisez la dimension ResponseType pour connaître le nombre des différents types de réponses.|ResponseType, GeoType, ApiName, Authentication|


## <a name="microsoftstoragesyncstoragesyncservices"></a>microsoft.storagesync/storageSyncServices

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|ServerSyncSessionResult|Oui|Résultat de session de synchronisation|Count|Average|Métrique consignant une valeur de 1 chaque fois que le point de terminaison de serveur termine une session de synchronisation avec le point de terminaison cloud|SyncGroupName, ServerEndpointName, SyncDirection|
|StorageSyncBatchTransferredFileBytes|Oui|Octets synchronisés|Octets|Total|Taille totale des fichiers transférés pour les sessions de synchronisation|SyncGroupName, ServerEndpointName, SyncDirection|
|StorageSyncRecalledNetworkBytesByApplication|Oui|Taille de rappel de hiérarchisation cloud par application|Octets|Total|Taille des données rappelées par application|SyncGroupName, ServerName, ApplicationName|
|StorageSyncRecalledTotalNetworkBytes|Oui|Taille de rappel de la hiérarchisation cloud|Octets|Total|taille des données rappelées ;|SyncGroupName, ServerName|
|StorageSyncRecallIOTotalSizeBytes|Oui|Rappel de hiérarchisation cloud|Octets|Total|Taille totale des données rappelées par le serveur|ServerName|
|StorageSyncRecallThroughputBytesPerSecond|Oui|Débit de rappel de la hiérarchisation cloud|BytesPerSecond|Average|Taille de débit de rappel des données|SyncGroupName, ServerName|
|StorageSyncServerHeartbeat|Oui|État du serveur en ligne|Count|Maximale|Métrique consignant une valeur de 1 chaque fois le serveur inscrit enregistre une pulsation avec le point de terminaison cloud|ServerName|
|StorageSyncSyncSessionAppliedFilesCount|Oui|Fichiers synchronisés|Count|Total|Nombre de fichiers synchronisés|SyncGroupName, ServerEndpointName, SyncDirection|
|StorageSyncSyncSessionPerItemErrorsCount|Oui|Fichiers ne se synchronisant pas|Count|Total|Nombre de fichiers dont la synchronisation a échoué|SyncGroupName, ServerEndpointName, SyncDirection|


## <a name="microsoftstoragesyncstoragesyncservicesregisteredservers"></a>microsoft.storagesync/storageSyncServices/registeredServers

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|ServerHeartbeat|Oui|État du serveur en ligne|Count|Maximale|Métrique consignant une valeur de 1 chaque fois le serveur inscrit enregistre une pulsation avec le point de terminaison cloud|ServerResourceId, ServerName|
|ServerRecallIOTotalSizeBytes|Oui|Rappel de hiérarchisation cloud|Octets|Total|Taille totale des données rappelées par le serveur|ServerResourceId, ServerName|


## <a name="microsoftstoragesyncstoragesyncservicessyncgroups"></a>microsoft.storagesync/storageSyncServices/syncGroups

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|SyncGroupBatchTransferredFileBytes|Oui|Octets synchronisés|Octets|Total|Taille totale des fichiers transférés pour les sessions de synchronisation|SyncGroupName, ServerEndpointName, SyncDirection|
|SyncGroupSyncSessionAppliedFilesCount|Oui|Fichiers synchronisés|Count|Total|Nombre de fichiers synchronisés|SyncGroupName, ServerEndpointName, SyncDirection|
|SyncGroupSyncSessionPerItemErrorsCount|Oui|Fichiers ne se synchronisant pas|Count|Total|Nombre de fichiers dont la synchronisation a échoué|SyncGroupName, ServerEndpointName, SyncDirection|


## <a name="microsoftstoragesyncstoragesyncservicessyncgroupsserverendpoints"></a>microsoft.storagesync/storageSyncServices/syncGroups/serverEndpoints

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|ServerEndpointBatchTransferredFileBytes|Oui|Octets synchronisés|Octets|Total|Taille totale des fichiers transférés pour les sessions de synchronisation|ServerEndpointName, SyncDirection|
|ServerEndpointSyncSessionAppliedFilesCount|Oui|Fichiers synchronisés|Count|Total|Nombre de fichiers synchronisés|ServerEndpointName, SyncDirection|
|ServerEndpointSyncSessionPerItemErrorsCount|Oui|Fichiers ne se synchronisant pas|Count|Total|Nombre de fichiers dont la synchronisation a échoué|ServerEndpointName, SyncDirection|


## <a name="microsoftstreamanalyticsstreamingjobs"></a>Microsoft.StreamAnalytics/streamingjobs

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|AMLCalloutFailedRequests|Oui|Requêtes de fonction ayant échoué|Count|Total|Requêtes de fonction ayant échoué|LogicalName, PartitionId|
|AMLCalloutInputEvents|Oui|Événements de fonction|Count|Total|Événements de fonction|LogicalName, PartitionId|
|AMLCalloutRequests|Oui|Requêtes de fonction|Count|Total|Requêtes de fonction|LogicalName, PartitionId|
|ConversionErrors|Oui|Erreurs de conversion de données|Count|Total|Erreurs de conversion de données|LogicalName, PartitionId|
|DeserializationError|Oui|Erreurs de désérialisation d’entrée|Count|Total|Erreurs de désérialisation d’entrée|LogicalName, PartitionId|
|DroppedOrAdjustedEvents|Oui|Événements en désordre|Count|Total|Événements en désordre|LogicalName, PartitionId|
|EarlyInputEvents|Oui|Événements d’entrée précoces|Count|Total|Événements d’entrée précoces|LogicalName, PartitionId|
|Erreurs|Oui|Erreurs d’exécution|Count|Total|Erreurs d’exécution|LogicalName, PartitionId|
|InputEventBytes|Oui|Octets des événements d’entrée|Octets|Total|Octets des événements d’entrée|LogicalName, PartitionId|
|InputEvents|Oui|Événements d’entrée|Count|Total|Événements d’entrée|LogicalName, PartitionId|
|InputEventsSourcesBacklogged|Oui|Événements d'entrée en backlog|Count|Maximale|Événements d'entrée en backlog|LogicalName, PartitionId|
|InputEventsSourcesPerSecond|Oui|Sources d'entrée reçues|Count|Total|Sources d'entrée reçues|LogicalName, PartitionId|
|LateInputEvents|Oui|Événements d’entrée tardifs|Count|Total|Événements d’entrée tardifs|LogicalName, PartitionId|
|OutputEvents|Oui|Événements de sortie|Count|Total|Événements de sortie|LogicalName, PartitionId|
|OutputWatermarkDelaySeconds|Oui|Délai en filigrane|Secondes|Maximale|Délai en filigrane|LogicalName, PartitionId|
|ResourceUtilization|Oui|Utilisation de % d’unités de diffusion|Pourcentage|Maximale|Utilisation de % d’unités de diffusion|LogicalName, PartitionId|


## <a name="microsoftsynapseworkspaces"></a>Microsoft.Synapse/workspaces

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|OrchestrationActivityRunsEnded|Non|Exécutions d’activité terminées|Count|Total|Nombre d’activités d’orchestration ayant abouti, échoué ou qui ont été annulées|Result, FailureType, Activity, ActivityType, Pipeline|
|OrchestrationPipelineRunsEnded|Non|Exécutions de pipeline terminées|Count|Total|Nombre d’exécutions de pipeline d’orchestration ayant abouti, échoué ou qui ont été annulées|Result, FailureType, Pipeline|
|OrchestrationTriggersEnded|Non|Déclencheurs terminés|Count|Total|Nombre de déclencheurs d’orchestration ayant abouti, échoué ou qui ont été annulés|Result, FailureType, Trigger|
|SQLOnDemandLoginAttempts|Non|Tentatives de connexion|Count|Total|Nombre de tentatives de connexion ayant abouti ou échoué|Résultats|
|SQLOnDemandQueriesEnded|Non|Requêtes terminées|Count|Total|Nombre de requêtes ayant abouti, échoué ou qui ont été annulées|Résultats|
|SQLOnDemandQueryProcessedBytes|Non|Données traitées|Octets|Total|Quantité de données traitées par les requêtes|Aucune dimension|


## <a name="microsoftsynapseworkspacesbigdatapools"></a>Microsoft.Synapse/workspaces/bigDataPools

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|CoresCapacity|Non|Capacité des cœurs|Count|Maximale|Capacité des cœurs|Aucune dimension|
|MemoryCapacityGB|Non|Capacité de mémoire (Go)|Count|Maximale|Capacité de mémoire (Go)|Aucune dimension|
|SparkJobsEnded|Oui|Applications terminées|Count|Total|Nombre d’applications terminées|JobType, JobResult|


## <a name="microsoftsynapseworkspacessqlpools"></a>Microsoft.Synapse/workspaces/sqlPools

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|AdaptiveCacheHitPercent|Non|Pourcentage d’accès au cache adaptatif|Pourcentage|Maximale|Mesure avec quelle efficacité les charges de travail utilisent le cache adaptatif. Utilisez cette métrique avec celle du pourcentage d’accès au cache pour déterminer s’il convient d’effectuer une mise à l’échelle pour une capacité supplémentaire ou de réexécuter les charges de travail pour alimenter le cache|Aucune dimension|
|AdaptiveCacheUsedPercent|Non|Pourcentage d’utilisation du cache adaptatif|Pourcentage|Maximale|Mesure avec quelle efficacité les charges de travail utilisent le cache adaptatif. Utilisez cette métrique avec celle du pourcentage d’utilisation du cache pour déterminer s’il convient d’effectuer une mise à l’échelle pour une capacité supplémentaire ou de réexécuter les charges de travail pour alimenter le cache|Aucune dimension|
|Connexions|Oui|Connexions|Count|Total|Nombre total de connexions au pool SQL dédié|Résultats|
|ConnectionsBlockedByFirewall|Non|Connexions bloquées par le pare-feu|Count|Total|Nombre de connexions bloquées par les règles de pare-feu. Revisitez les stratégies de contrôle d’accès de votre pool SQL dédié et supervisez ces connexions si le nombre est élevé|Aucune dimension|
|DWULimit|Non|Limite DWU|Count|Maximale|Objectif de niveau de service du pool SQL dédié|Aucune dimension|
|DWUUsed|Non|DWU utilisé|Count|Maximale|Représentation générale de l’utilisation dans l’ensemble du pool SQL dédié. Mesurée par Limite DWU * Pourcentage DWU|Aucune dimension|
|DWUUsedPercent|Non|Pourcentage d’utilisation de DWU|Pourcentage|Maximale|Représentation générale de l’utilisation dans l’ensemble du pool SQL dédié. Mesurée en prenant la valeur maximale entre le pourcentage de processeur et le pourcentage d’E/S des données|Aucune dimension|
|LocalTempDBUsedPercent|Non|Pourcentage utilisé de tempdb locale|Pourcentage|Maximale|Utilisation de la tempdb locale sur tous les nœuds de calcul. Des valeurs sont émises toutes les cinq minutes|Aucune dimension|
|MemoryUsedPercent|Non|Pourcentage de mémoire utilisé|Pourcentage|Maximale|Utilisation de la mémoire sur tous les nœuds du pool SQL dédié|Aucune dimension|
|wlg_effective_min_resource_percent|Oui|Pourcentage minimal de ressources réelles|Pourcentage|Minimum|Paramètre de pourcentage minimal de ressources réelles autorisé en tenant compte du niveau de service et des paramètres de groupe de charges de travail. Le min_percentage_resource effectif peut être ajusté à une valeur supérieure sur des niveaux de service inférieurs|IsUserDefined, WorkloadGroup|
|WLGActiveQueries|Non|Requêtes actives du groupe de charge de travail|Count|Total|Requêtes actives dans le groupe de charges de travail. L’utilisation de cette métrique non filtrée et non fractionnée affiche toutes les requêtes actives en cours d’exécution sur le système|IsUserDefined, WorkloadGroup|
|WLGActiveQueriesTimeouts|Non|Délais d’expiration des requêtes du groupe de charge de travail|Count|Total|Requêtes pour le groupe de charge de travail dont le délai a expiré. Les délais d’expiration des requêtes signalés par cette métrique démarrent uniquement une fois que l’exécution de la requête a commencé (cela n’inclut pas le temps d’attente dû au verrouillage ou à l’attente des ressources)|IsUserDefined, WorkloadGroup|
|WLGAllocationByMaxResourcePercent|Non|Allocation du groupe de charge de travail par pourcentage maximal de ressources|Pourcentage|Maximale|Affiche l’allocation de pourcentage des ressources par rapport au pourcentage de ressources limitées réelles par groupe de charges de travail. Cette métrique fournit l’utilisation effective du groupe de charges de travail|IsUserDefined, WorkloadGroup|
|WLGAllocationBySystemPercent|Non|Allocation du groupe de charge de travail par pourcentage système|Pourcentage|Maximale|Allocation de pourcentage de ressources par rapport à l’ensemble du système|IsUserDefined, WorkloadGroup|
|WLGEffectiveCapResourcePercent|Oui|Pourcentage de ressources limitées réelles|Pourcentage|Maximale|Pourcentage de ressources limitées réelles pour le groupe de charges de travail. S’il existe d’autres groupes de charges de travail avec min_percentage_resource > 0, effective_cap_percentage_resource est abaissé proportionnellement|IsUserDefined, WorkloadGroup|
|WLGQueuedQueries|Non|Requêtes en file d’attente du groupe de charge de travail|Count|Total|Nombre cumulatif de demandes mises en file d’attente une fois la limite de concurrence maximale atteinte|IsUserDefined, WorkloadGroup|


## <a name="microsofttimeseriesinsightsenvironments"></a>Microsoft.TimeSeriesInsights/environments

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|IngressReceivedBytes|Oui|Octets reçus en entrée|Octets|Total|Nombre d’octets lus à partir de toutes les sources d’événements|Aucune dimension|
|IngressReceivedInvalidMessages|Oui|Messages non valides reçus en entrée|Count|Total|Nombre de messages non valides lus à partir de la totalité des Event Hubs ou des sources d’événements IoT Hub|Aucune dimension|
|IngressReceivedMessages|Oui|Messages reçus en entrée|Count|Total|Nombre de messages lus à partir de la totalité des Event Hubs ou des sources d’événements IoT Hub|Aucune dimension|
|IngressReceivedMessagesCountLag|Oui|Décalage de nombre des messages reçus en entrée|Count|Average|Différence entre le numéro de séquence du dernier message placé en file d’attente dans la partition source de l’événement et le numéro de séquence des messages traités en entrée|Aucune dimension|
|IngressReceivedMessagesTimeLag|Oui|Retard des messages reçus en entrée|Secondes|Maximale|Différence entre l’heure à laquelle le message est placé en file d’attente dans la source d’événement et l’heure à laquelle il est traité en entrée|Aucune dimension|
|IngressStoredBytes|Oui|Octets stockés en entrée|Octets|Total|Taille totale des événements traités correctement et disponibles pour une requête|Aucune dimension|
|IngressStoredEvents|Oui|Événements stockés en entrée|Count|Total|Nombre d’événements aplatis traités correctement et disponibles pour une requête|Aucune dimension|
|WarmStorageMaxProperties|Oui|Propriétés max de stockage chaud|Count|Maximale|Nombre maximal de propriétés utilisées autorisées par l’environnement pour la référence SKU S1/S2 et nombre maximal de propriétés autorisées par le magasin Warm pour la référence SKU PAYG|Aucune dimension|
|WarmStorageUsedProperties|Oui|Propriétés de stockage chaud utilisées |Count|Maximale|Nombre de propriétés utilisées par l’environnement pour la référence SKU S1/S2 et nombre de propriétés utilisées par le magasin Warm pour la référence SKU PAYG|Aucune dimension|


## <a name="microsofttimeseriesinsightsenvironmentseventsources"></a>Microsoft.TimeSeriesInsights/environments/eventsources

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|IngressReceivedBytes|Oui|Octets reçus en entrée|Octets|Total|Nombre d’octets lus à partir de la source de l’événement|Aucune dimension|
|IngressReceivedInvalidMessages|Oui|Messages non valides reçus en entrée|Count|Total|Nombre de messages non valides lus à partir de la source de l’événement|Aucune dimension|
|IngressReceivedMessages|Oui|Messages reçus en entrée|Count|Total|Nombre de messages lus à partir de la source de l’événement|Aucune dimension|
|IngressReceivedMessagesCountLag|Oui|Décalage de nombre des messages reçus en entrée|Count|Average|Différence entre le numéro de séquence du dernier message placé en file d’attente dans la partition source de l’événement et le numéro de séquence des messages traités en entrée|Aucune dimension|
|IngressReceivedMessagesTimeLag|Oui|Retard des messages reçus en entrée|Secondes|Maximale|Différence entre l’heure à laquelle le message est placé en file d’attente dans la source d’événement et l’heure à laquelle il est traité en entrée|Aucune dimension|
|IngressStoredBytes|Oui|Octets stockés en entrée|Octets|Total|Taille totale des événements traités correctement et disponibles pour une requête|Aucune dimension|
|IngressStoredEvents|Oui|Événements stockés en entrée|Count|Total|Nombre d’événements aplatis traités correctement et disponibles pour une requête|Aucune dimension|
|WarmStorageMaxProperties|Oui|Propriétés max de stockage chaud|Count|Maximale|Nombre maximal de propriétés utilisées autorisées par l’environnement pour la référence SKU S1/S2 et nombre maximal de propriétés autorisées par le magasin Warm pour la référence SKU PAYG|Aucune dimension|
|WarmStorageUsedProperties|Oui|Propriétés de stockage chaud utilisées |Count|Maximale|Nombre de propriétés utilisées par l’environnement pour la référence SKU S1/S2 et nombre de propriétés utilisées par le magasin Warm pour la référence SKU PAYG|Aucune dimension|


## <a name="microsoftvmwarecloudsimplevirtualmachines"></a>Microsoft.VMwareCloudSimple/virtualMachines

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|Disk Read Bytes|Oui|Disk Read Bytes|Octets|Total|Débit total du disque en raison des opérations de lecture au cours de la période d’échantillonnage.|Aucune dimension|
|Disk Read Operations/Sec|Oui|Disk Read Operations/Sec|CountPerSecond|Average|Nombre moyen d'opérations de lecture (E/S) au cours de la précédente période d’échantillonnage. Notez que ces opérations peuvent varier en taille.|Aucune dimension|
|Disk Write Bytes|Oui|Disk Write Bytes|Octets|Total|Débit total du disque en raison des opérations d'écriture au cours de la période d’échantillonnage.|Aucune dimension|
|Disk Write Operations/Sec|Oui|Disk Write Operations/Sec|CountPerSecond|Average|Nombre moyen d'opérations d'écriture (E/S) au cours de la précédente période d’échantillonnage. Notez que ces opérations peuvent varier en taille.|Aucune dimension|
|DiskReadBytesPerSecond|Oui|Disk Read Bytes/Sec|BytesPerSecond|Average|Débit moyen du disque en raison des opérations de lecture pendant la période d’échantillonnage.|Aucune dimension|
|DiskReadLatency|Oui|Latence de lecture sur disque|Millisecondes|Average|Latence de lecture totale. Somme des latences de lecture sur l'appareil et le noyau.|Aucune dimension|
|DiskReadOperations|Oui|Opérations de lecture sur disque|Count|Total|Nombre d'opérations de lecture (E/S) au cours de la précédente période d’échantillonnage. Notez que ces opérations peuvent varier en taille.|Aucune dimension|
|DiskWriteBytesPerSecond|Oui|Disk Write Bytes/Sec|BytesPerSecond|Average|Débit moyen du disque en raison des opérations d'écriture pendant la période d’échantillonnage.|Aucune dimension|
|DiskWriteLatency|Oui|Latence d'écriture sur disque|Millisecondes|Average|Latence d'écriture totale. Somme des latences d'écriture sur l'appareil et le noyau.|Aucune dimension|
|DiskWriteOperations|Oui|Opérations d'écriture sur disque|Count|Total|Nombre d'opérations d'écriture (E/S) au cours de la précédente période d’échantillonnage. Notez que ces opérations peuvent varier en taille.|Aucune dimension|
|MemoryActive|Oui|Mémoire active|Octets|Average|Quantité de mémoire utilisée par la machine virtuelle dans la petite fenêtre de temps passée. Elle correspond à la quantité de mémoire « vraie » dont la machine virtuelle a actuellement besoin. La mémoire supplémentaire non utilisée peut être échangée ou désallouée sans incidence sur le niveau de performance de l'invité.|Aucune dimension|
|MemoryGranted|Oui|Mémoire allouée|Octets|Average|Quantité de mémoire allouée par l'hôte à la machine virtuelle. La mémoire n’est pas allouée à l’hôte avant un premier accès et la mémoire allouée peut être échangée ou désallouée si le VMkernel a besoin de mémoire.|Aucune dimension|
|MemoryUsed|Oui|Mémoire utilisée|Octets|Average|Quantité de mémoire utilisée par la machine virtuelle.|Aucune dimension|
|Network In|Oui|Network In|Octets|Total|Débit réseau total pour le trafic reçu.|Aucune dimension|
|Network Out|Oui|Network Out|Octets|Total|Débit réseau total pour le trafic transmis.|Aucune dimension|
|NetworkInBytesPerSecond|Oui|Octets entrants réseau/s|BytesPerSecond|Average|Débit réseau moyen pour le trafic reçu.|Aucune dimension|
|NetworkOutBytesPerSecond|Oui|Octets sortants réseau/s|BytesPerSecond|Average|Débit réseau moyen pour le trafic transmis.|Aucune dimension|
|Percentage CPU|Oui|Percentage CPU|Pourcentage|Average|Utilisation du processeur. Cette valeur est signalée avec 100 % représentant tous les cœurs de processeur sur le système. Par exemple, une machine virtuelle bidirectionnelle utilisant 50 % d'un système à quatre cœurs utilise complètement deux cœurs.|Aucune dimension|
|PercentageCpuReady|Oui|Pourcentage de processeurs prêts|Millisecondes|Total|Correspond au temps passé à attendre que les processeurs soient disponibles dans l'intervalle de la dernière mise à jour.|Aucune dimension|


## <a name="microsoftwebhostingenvironmentsmultirolepools"></a>Microsoft.Web/hostingEnvironments/multiRolePools

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|ActiveRequests|Oui|Requêtes actives|Count|Total|Requêtes actives|Instance|
|AverageResponseTime|Oui|Temps de réponse moyen|Secondes|Average|Temps de réponse moyen|Instance|
|BytesReceived|Oui|Données entrantes|Octets|Total|Données entrantes|Instance|
|BytesSent|Oui|Données sortantes|Octets|Total|Données sortantes|Instance|
|CpuPercentage|Oui|Pourcentage UC|Pourcentage|Average|Pourcentage UC|Instance|
|DiskQueueLength|Oui|Longueur de file d'attente de disque|Count|Average|Longueur de file d'attente de disque|Instance|
|Http101|Oui|HTTP 101|Count|Total|HTTP 101|Instance|
|Http2xx|Oui|Http 2xx|Count|Total|Http 2xx|Instance|
|Http3xx|Oui|Http 3xx|Count|Total|Http 3xx|Instance|
|Http401|Oui|Http 401|Count|Total|Http 401|Instance|
|Http403|Oui|Http 403|Count|Total|Http 403|Instance|
|Http404|Oui|Http 404|Count|Total|Http 404|Instance|
|Http406|Oui|Http 406|Count|Total|Http 406|Instance|
|Http4xx|Oui|Http 4xx|Count|Total|Http 4xx|Instance|
|Http5xx|Oui|Erreurs de serveur http|Count|Total|Erreurs de serveur http|Instance|
|HttpQueueLength|Oui|Longueur de la file d’attente HTTP|Count|Average|Longueur de la file d’attente HTTP|Instance|
|LargeAppServicePlanInstances|Oui|Workers de plan App Service de grande taille|Count|Average|Workers de plan App Service de grande taille|Aucune dimension|
|MediumAppServicePlanInstances|Oui|Workers de plan App Service de taille moyenne|Count|Average|Workers de plan App Service de taille moyenne|Aucune dimension|
|MemoryPercentage|Oui|Pourcentage de mémoire|Pourcentage|Average|Pourcentage de mémoire|Instance|
|Demandes|Oui|Demandes|Count|Total|Demandes|Instance|
|SmallAppServicePlanInstances|Oui|Workers de plan App Service de petite taille|Count|Average|Workers de plan App Service de petite taille|Aucune dimension|
|TotalFrontEnds|Oui|Nombre total de serveurs frontaux|Count|Average|Nombre total de serveurs frontaux|Aucune dimension|


## <a name="microsoftwebhostingenvironmentsworkerpools"></a>Microsoft.Web/hostingEnvironments/workerPools

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|CpuPercentage|Oui|Pourcentage UC|Pourcentage|Average|Pourcentage UC|Instance|
|MemoryPercentage|Oui|Pourcentage de mémoire|Pourcentage|Average|Pourcentage de mémoire|Instance|
|WorkersAvailable|Oui|Workers disponibles|Count|Average|Workers disponibles|Aucune dimension|
|WorkersTotal|Oui|Nombre total de workers|Count|Average|Nombre total de workers|Aucune dimension|
|WorkersUsed|Oui|Workers utilisés|Count|Average|Workers utilisés|Aucune dimension|


## <a name="microsoftwebserverfarms"></a>Microsoft.Web/serverfarms

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|BytesReceived|Oui|Données entrantes|Octets|Total|Données entrantes|Instance|
|BytesSent|Oui|Données sortantes|Octets|Total|Données sortantes|Instance|
|CpuPercentage|Oui|Pourcentage UC|Pourcentage|Average|Pourcentage UC|Instance|
|DiskQueueLength|Oui|Longueur de file d'attente de disque|Count|Average|Longueur de file d'attente de disque|Instance|
|HttpQueueLength|Oui|Longueur de la file d’attente HTTP|Count|Average|Longueur de la file d’attente HTTP|Instance|
|MemoryPercentage|Oui|Pourcentage de mémoire|Pourcentage|Average|Pourcentage de mémoire|Instance|
|SocketInboundAll|Oui|SocketInboundAll|Count|Average|SocketInboundAll|Instance|
|SocketLoopback|Oui|SocketLoopback|Count|Average|SocketLoopback|Instance|
|SocketOutboundAll|Oui|SocketOutboundAll|Count|Average|SocketOutboundAll|Instance|
|SocketOutboundEstablished|Oui|SocketOutboundEstablished|Count|Average|SocketOutboundEstablished|Instance|
|SocketOutboundTimeWait|Oui|SocketOutboundTimeWait|Count|Average|SocketOutboundTimeWait|Instance|
|TcpCloseWait|Oui|Close Wait - TCP|Count|Average|Close Wait - TCP|Instance|
|TcpClosing|Oui|Closing - TCP|Count|Average|Closing - TCP|Instance|
|TcpEstablished|Oui|Established - TCP|Count|Average|Established - TCP|Instance|
|TcpFinWait1|Oui|Fin Wait 1 - TCP|Count|Average|Fin Wait 1 - TCP|Instance|
|TcpFinWait2|Oui|Fin Wait 2 - TCP|Count|Average|Fin Wait 2 - TCP|Instance|
|TcpLastAck|Oui|Last Ack - TCP|Count|Average|Last Ack - TCP|Instance|
|TcpSynReceived|Oui|Syn Received - TCP|Count|Average|Syn Received - TCP|Instance|
|TcpSynSent|Oui|Syn Sent - TCP|Count|Average|Syn Sent - TCP|Instance|
|TcpTimeWait|Oui|Time Wait - TCP|Count|Average|Time Wait - TCP|Instance|

## <a name="microsoftwebsites-excluding-functions"></a>Microsoft.Web/sites (à l’exclusion de Functions) 

> [!NOTE]
> **Utilisation de systèmes de fichiers** est une nouvelle métrique déployée à l’échelle mondiale. Vous n’avez aucune donnée à attendre sauf si vous figurez sur la liste verte de la préversion privée.

> [!IMPORTANT]
> Le **temps de réponse moyen** sera bientôt déprécié afin d’éviter toute confusion avec les agrégations de métriques. Utilisez le **Temps de réponse** à la place.

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|AppConnections|Oui|Connexions|Count|Average|Connexions|Instance|
|AverageMemoryWorkingSet|Oui|Plage de travail moyenne de la mémoire|Octets|Average|Plage de travail moyenne de la mémoire|Instance|
|AverageResponseTime|Oui|Temps de réponse moyen **(déconseillé)**|Secondes|Average|Temps de réponse moyen|Instance|
|BytesReceived|Oui|Données entrantes|Octets|Total|Données entrantes|Instance|
|BytesSent|Oui|Données sortantes|Octets|Total|Données sortantes|Instance|
|CpuTime|Oui|Temps processeur|Secondes|Total|Temps processeur|Instance|
|CurrentAssemblies|Oui|Assemblys actuels|Count|Average|Assemblys actuels|Instance|
|FileSystemUsage|Oui|Utilisation de systèmes de fichiers|Octets|Average|Utilisation de systèmes de fichiers|Aucune dimension|
|Gen0Collections|Oui|Garbage collections de génération 0|Count|Total|Garbage collections de génération 0|Instance|
|Gen1Collections|Oui|Garbage collections de génération 1|Count|Total|Garbage collections de génération 1|Instance|
|Gen2Collections|Oui|Garbage collections de génération 2|Count|Total|Garbage collections de génération 2|Instance|
|Poignées|Oui|Nombre de descripteurs|Count|Average|Nombre de descripteurs|Instance|
|HealthCheckStatus|Oui|État de contrôle d’intégrité|Count|Average|État de contrôle d’intégrité|Instance|
|Http101|Oui|HTTP 101|Count|Total|HTTP 101|Instance|
|Http2xx|Oui|Http 2xx|Count|Total|Http 2xx|Instance|
|Http3xx|Http 3xx|Count|Total|Http 3xx|Instance|
|Http401|Oui|Http 401|Count|Total|Http 401|Instance|
|Http403|Oui|Http 403|Count|Total|Http 403|Instance|
|Http404|Oui|Http 404|Count|Total|Http 404|Instance|
|Http406|Oui|Http 406|Count|Total|Http 406|Instance|
|Http4xx|Oui|Http 4xx|Count|Total|Http 4xx|Instance|
|Http5xx|Oui|Erreurs de serveur http|Count|Total|Erreurs de serveur http|Instance|
|HttpResponseTime|Oui|Temps de réponse|Secondes|Average|Temps de réponse|Instance|
|IoOtherBytesPerSecond|Oui|Autres octets par seconde (E/S)|BytesPerSecond|Total|Autres octets par seconde (E/S)|Instance|
|IoOtherOperationsPerSecond|Oui|Autres opérations par seconde (E/S)|BytesPerSecond|Total|Autres opérations par seconde (E/S)|Instance|
|IoReadBytesPerSecond|Oui|Octets lus par seconde (E/S)|BytesPerSecond|Total|Octets lus par seconde (E/S)|Instance|
|IoReadOperationsPerSecond|Oui|Opérations de lecture par seconde (E/S)|BytesPerSecond|Total|Opérations de lecture par seconde (E/S)|Instance|
|IoWriteBytesPerSecond|Oui|Octets écrits par seconde (E/S)|BytesPerSecond|Total|Octets écrits par seconde (E/S)|Instance|
|IoWriteOperationsPerSecond|Oui|Opérations d’écriture par seconde (E/S)|BytesPerSecond|Total|Opérations d’écriture par seconde (E/S)|Instance|
|MemoryWorkingSet|Oui|Plage de travail de la mémoire|Octets|Average|Plage de travail de la mémoire|Instance|
|PrivateBytes|Oui|Octets privés|Octets|Average|Octets privés|Instance|
|Demandes|Oui|Demandes|Count|Total|Demandes|Instance|
|RequestsInApplicationQueue|Oui|Demandes dans la file d’attente d’application|Count|Average|Demandes dans la file d’attente d’application|Instance|
|Threads|Oui|Nombre de threads|Count|Average|Nombre de threads|Instance|
|TotalAppDomains|Oui|Total des domaines d’application|Count|Average|Total des domaines d’application|Instance|
|TotalAppDomainsUnloaded|Oui|Total des domaines d’application déchargés|Count|Average|Total des domaines d’application déchargés|Instance|

## <a name="microsoftwebsites-functions"></a>Microsoft.Web/sites (Functions)

> [!NOTE]
> **Utilisation de systèmes de fichiers** est une nouvelle métrique déployée à l’échelle mondiale. Vous n’avez aucune donnée à attendre sauf si vous figurez sur la liste verte de la préversion privée.

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|AverageMemoryWorkingSet|Oui|Plage de travail moyenne de la mémoire|Octets|Average|Plage de travail moyenne de la mémoire|Instance|
|BytesReceived|Oui|Données entrantes|Octets|Total|Données entrantes|Instance|
|BytesSent|Oui|Données sortantes|Octets|Total|Données sortantes|Instance|
|CurrentAssemblies|Oui|Assemblys actuels|Count|Average|Assemblys actuels|Instance|
|FileSystemUsage|Oui|Utilisation de systèmes de fichiers|Octets|Average|Utilisation de systèmes de fichiers|Aucune dimension|
|FunctionExecutionCount|Oui|Nombre d’exécutions de fonctions|Count|Total|Nombre d’exécutions de fonctions|Instance|
|FunctionExecutionUnits|Oui|Unités d’exécution de fonctions|Count|Total|[Unités d’exécution de fonctions](https://github.com/Azure/Azure-Functions/wiki/Consumption-Plan-Cost-Billing-FAQ#how-can-i-view-graphs-of-execution-count-and-gb-seconds)|Instance|
|Gen0Collections|Oui|Garbage collections de génération 0|Count|Total|Garbage collections de génération 0|Instance|
|Gen1Collections|Oui|Garbage collections de génération 1|Count|Total|Garbage collections de génération 1|Instance|
|Gen2Collections|Oui|Garbage collections de génération 2|Count|Total|Garbage collections de génération 2|Instance|
|HealthCheckStatus|Oui|État de contrôle d’intégrité|Count|Average|État de contrôle d’intégrité|Instance|
|Http5xx|Oui|Erreurs de serveur http|Count|Total|Erreurs de serveur http|Instance|
|IoOtherBytesPerSecond|Oui|Autres octets par seconde (E/S)|BytesPerSecond|Total|Autres octets par seconde (E/S)|Instance|
|IoOtherOperationsPerSecond|Oui|Autres opérations par seconde (E/S)|BytesPerSecond|Total|Autres opérations par seconde (E/S)|Instance|
|IoReadBytesPerSecond|Oui|Octets lus par seconde (E/S)|BytesPerSecond|Total|Octets lus par seconde (E/S)|Instance|
|IoReadOperationsPerSecond|Oui|Opérations de lecture par seconde (E/S)|BytesPerSecond|Total|Opérations de lecture par seconde (E/S)|Instance|
|IoWriteBytesPerSecond|Oui|Octets écrits par seconde (E/S)|BytesPerSecond|Total|Octets écrits par seconde (E/S)|Instance|
|IoWriteOperationsPerSecond|Oui|Opérations d’écriture par seconde (E/S)|BytesPerSecond|Total|Opérations d’écriture par seconde (E/S)|Instance|
|MemoryWorkingSet|Oui|Plage de travail de la mémoire|Octets|Average|Plage de travail de la mémoire|Instance|
|PrivateBytes|Oui|Octets privés|Octets|Average|Octets privés|Instance|
|RequestsInApplicationQueue|Oui|Demandes dans la file d’attente d’application|Count|Average|Demandes dans la file d’attente d’application|Instance|
|TotalAppDomains|Oui|Total des domaines d’application|Count|Average|Total des domaines d’application|Instance|
|TotalAppDomainsUnloaded|Oui|Total des domaines d’application déchargés|Count|Average|Total des domaines d’application déchargés|Instance|

## <a name="microsoftwebsitesslots"></a>Microsoft.Web/sites/slots

|Métrique|Exportable par le biais des paramètres de diagnostic ?|Nom d’affichage de la métrique|Unité|Type d’agrégation|Description|Dimensions|
|---|---|---|---|---|---|---|
|AppConnections|Oui|Connexions|Count|Average|Connexions|Instance|
|AverageMemoryWorkingSet|Oui|Plage de travail moyenne de la mémoire|Octets|Average|Plage de travail moyenne de la mémoire|Instance|
|AverageResponseTime|Oui|Temps de réponse moyen|Secondes|Average|Temps de réponse moyen|Instance|
|BytesReceived|Oui|Données entrantes|Octets|Total|Données entrantes|Instance|
|BytesSent|Oui|Données sortantes|Octets|Total|Données sortantes|Instance|
|CpuTime|Oui|Temps processeur|Secondes|Total|Temps processeur|Instance|
|CurrentAssemblies|Oui|Assemblys actuels|Count|Average|Assemblys actuels|Instance|
|FileSystemUsage|Oui|Utilisation de systèmes de fichiers|Octets|Average|Utilisation de systèmes de fichiers|Aucune dimension|
|FunctionExecutionCount|Oui|Nombre d’exécutions de fonctions|Count|Total|Nombre d’exécutions de fonctions|Instance|
|FunctionExecutionUnits|Oui|Unités d’exécution de fonctions|Count|Total|Unités d’exécution de fonctions|Instance|
|Gen0Collections|Oui|Garbage collections de génération 0|Count|Total|Garbage collections de génération 0|Instance|
|Gen1Collections|Oui|Garbage collections de génération 1|Count|Total|Garbage collections de génération 1|Instance|
|Gen2Collections|Oui|Garbage collections de génération 2|Count|Total|Garbage collections de génération 2|Instance|
|Poignées|Oui|Nombre de descripteurs|Count|Average|Nombre de descripteurs|Instance|
|HealthCheckStatus|Oui|État de contrôle d’intégrité|Count|Average|État de contrôle d’intégrité|Instance|
|Http101|Oui|HTTP 101|Count|Total|HTTP 101|Instance|
|Http2xx|Oui|Http 2xx|Count|Total|Http 2xx|Instance|
|Http3xx|Oui|Http 3xx|Count|Total|Http 3xx|Instance|
|Http401|Oui|Http 401|Count|Total|Http 401|Instance|
|Http403|Oui|Http 403|Count|Total|Http 403|Instance|
|Http404|Oui|Http 404|Count|Total|Http 404|Instance|
|Http406|Oui|Http 406|Count|Total|Http 406|Instance|
|Http4xx|Oui|Http 4xx|Count|Total|Http 4xx|Instance|
|Http5xx|Oui|Erreurs de serveur http|Count|Total|Erreurs de serveur http|Instance|
|HttpResponseTime|Oui|Temps de réponse|Secondes|Average|Temps de réponse|Instance|
|IoOtherBytesPerSecond|Oui|Autres octets par seconde (E/S)|BytesPerSecond|Total|Autres octets par seconde (E/S)|Instance|
|IoOtherOperationsPerSecond|Oui|Autres opérations par seconde (E/S)|BytesPerSecond|Total|Autres opérations par seconde (E/S)|Instance|
|IoReadBytesPerSecond|Oui|Octets lus par seconde (E/S)|BytesPerSecond|Total|Octets lus par seconde (E/S)|Instance|
|IoReadOperationsPerSecond|Oui|Opérations de lecture par seconde (E/S)|BytesPerSecond|Total|Opérations de lecture par seconde (E/S)|Instance|
|IoWriteBytesPerSecond|Oui|Octets écrits par seconde (E/S)|BytesPerSecond|Total|Octets écrits par seconde (E/S)|Instance|
|IoWriteOperationsPerSecond|Oui|Opérations d’écriture par seconde (E/S)|BytesPerSecond|Total|Opérations d’écriture par seconde (E/S)|Instance|
|MemoryWorkingSet|Oui|Plage de travail de la mémoire|Octets|Average|Plage de travail de la mémoire|Instance|
|PrivateBytes|Oui|Octets privés|Octets|Average|Octets privés|Instance|
|Demandes|Oui|Demandes|Count|Total|Demandes|Instance|
|RequestsInApplicationQueue|Oui|Demandes dans la file d’attente d’application|Count|Average|Demandes dans la file d’attente d’application|Instance|
|Threads|Oui|Nombre de threads|Count|Average|Nombre de threads|Instance|
|TotalAppDomains|Oui|Total des domaines d’application|Count|Average|Total des domaines d’application|Instance|
|TotalAppDomainsUnloaded|Oui|Total des domaines d’application déchargés|Count|Average|Total des domaines d’application déchargés|Instance|


## <a name="next-steps"></a>Étapes suivantes
* [En savoir plus sur les métriques dans Azure Monitor](data-platform.md)
* [Créer des alertes sur les métriques](alerts-overview.md)
* [Exporter des métriques vers le stockage, un hub d’événements ou Log Analytics](platform-logs-overview.md)
