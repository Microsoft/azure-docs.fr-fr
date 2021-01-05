---
title: Types de ressource pris en charge par Azure Resource Health | Microsoft Docs
description: Types de ressource pris en charge par Azure Resource Health
ms.topic: conceptual
ms.date: 01/29/2019
ms.openlocfilehash: 79f979b55366d29ca8cf22a1e65ebdcc557ef594
ms.sourcegitcommit: 693df7d78dfd5393a28bf1508e3e7487e2132293
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/28/2020
ms.locfileid: "92899492"
---
# <a name="resource-types-and-health-checks-in-azure-resource-health"></a>Types de ressources et les contrôles d’intégrité dans Azure Resource Health
Voici une liste complète de toutes les vérifications exécutées via Resource Health par type de ressource.

## <a name="microsoftanalysisservicesservers"></a>Microsoft.AnalysisServices/servers
|Vérifications exécutées|
|---|
|<ul><li>Le serveur est-il en cours d’exécution ?</li><li>Le serveur manque-t-il de mémoire ?</li><li>Le serveur démarre-t-il ?</li><li>Le serveur récupère-t-il ?</li></ul>|

## <a name="microsoftapimanagementservice"></a>Microsoft.ApiManagement/service
|Vérifications exécutées|
|---|
|<ul><li>Est-ce que le service Gestion des API est opérationnel ?</li></ul>|

## <a name="microsoftappplatformspring"></a>Microsoft.AppPlatform/Spring
|Vérifications exécutées|
|---|
|<ul><li>L’instance Azure Spring Cloud est-elle disponible ?</li></ul>|

## <a name="microsoftbatchbatchaccounts"></a>Microsoft.Batch/batchAccounts
|Vérifications exécutées|
|---|
|<ul><li>Le compte Batch est-il opérationnel ?</li><li>Le quota de pools a-t-il été dépassé pour ce compte Batch ?</li></ul>|

## <a name="microsoftcacheredis"></a>Microsoft.Cache/Redis
|Vérifications exécutées|
|---|
|<ul><li>Tous les nœuds du cache sont-ils en cours d’exécution ?</li><li>Le cache peut-il être joint à partir du centre de données ?</li><li>Le cache a-t-il atteint le nombre maximal de connexions ?</li><li> Le cache a-t-il épuisé sa mémoire disponible ? </li><li>Le cache rencontre-t-il un grand nombre de défauts de page ?</li><li>Le cache subit-il une charge lourde ?</li></ul>|

## <a name="microsoftcdnprofile"></a>Microsoft.CDN/profile
|Vérifications exécutées|
|---|
|<ul> <li>Le portail supplémentaire est-il accessible pour les opérations de configuration CDN ?</li><li>Existe-t-il des problèmes de livraison continue avec les points de terminaison CDN ?</li><li>Les utilisateurs peuvent-ils modifier la configuration de leurs ressources CDN ?</li><li>Les modifications de configuration sont-elles propagées au rythme attendu ?</li><li>Les utilisateurs peuvent-ils gérer la configuration du CDN via le portail Azure, PowerShell ou l’API ?</li> </ul>|

## <a name="microsoftclassiccomputevirtualmachines"></a>Microsoft.classiccompute/virtualmachines
|Vérifications exécutées|
|---|
|<ul><li>Le serveur hôte est-il en cours d’exécution ?</li><li>Le démarrage du système d’exploitation hôte est-il terminé ?</li><li>Le conteneur de machine virtuelle est-il configuré et sous tension ?</li><li>Existe-t-il une connectivité réseau entre l’hôte et le compte de stockage ?</li><li>Le démarrage du système d’exploitation invité est-il terminé ?</li><li>Y a-t-il une maintenance planifiée régulière ?</li><li>Le matériel hôte est-il détérioré, avec la prévision d’une panne imminente ?</li></ul>|

## <a name="microsoftclassiccomputedomainnames"></a>Microsoft.classiccompute/domainnames
|Vérifications exécutées|
|---|
|<ul><li>Le déploiement de l’emplacement de production est-il sain dans toutes les instances de rôle ?</li><li>Le rôle est-il sain dans toutes ses instances de machine virtuelle ?</li><li>Quel est l’état d’intégrité de chaque machine virtuelle au sein du rôle d’un service cloud ?</li><li>La modification de l’état de la machine virtuelle était-il dû à une opération initiée par la plateforme ou par le client ?</li><li>Le démarrage du système d’exploitation invité est-il terminé ?</li><li>Y a-t-il une maintenance planifiée régulière ?</li><li>Le matériel hôte est-il détérioré, avec la prévision d’une panne imminente ?</li><li>[En savoir plus](../cloud-services/resource-health-for-cloud-services.md) sur les vérifications exécutées</li></ul>|

## <a name="microsoftcognitiveservicesaccounts"></a>Microsoft.cognitiveservices/accounts
|Vérifications exécutées|
|---|
|<ul><li>Le compte peut-il être joint à partir du centre de données ?</li><li>Le fournisseur de ressources Cognitive Services est-il disponible ?</li><li>Cognitive Service est-il disponible dans la région appropriée ?</li><li>Les opérations de lecture peuvent-elles être effectuées sur le compte de stockage contenant les métadonnées des ressources ?</li><li>Le quota d’appels à l’API a-t-il été atteint ?</li><li>La limite de lecture d’appels à l’API a-t-elle été atteinte ?</li></ul>|

## <a name="microsoftcomputehostgroupshosts"></a>Microsoft.compute/hostgroups/hosts
|Vérifications exécutées|
|---|
|<ul><li>L’hôte est-il opérationnel ?</li><li>Le matériel hôte est-il détérioré ?</li><li>L’hôte est-il désalloué ?</li><li>Le matériel hôte a-t-il été réparé par service sur un autre matériel ?</li></ul>|

## <a name="microsoftcomputevirtualmachines"></a>Microsoft.compute/virtualmachines
|Vérifications exécutées|
|---|
|<ul><li>Est-ce que le serveur d’hébergement de cette machine virtuelle est en cours d’exécution ?</li><li>Le démarrage du système d’exploitation hôte est-il terminé ?</li><li>Le conteneur de machine virtuelle est-il configuré et sous tension ?</li><li>Existe-t-il une connectivité réseau entre l’hôte et le compte de stockage ?</li><li>Le démarrage du système d’exploitation invité est-il terminé ?</li><li>Y a-t-il une maintenance planifiée régulière ?</li><li>Le matériel hôte est-il détérioré, avec la prévision d’une panne imminente ?</li></ul>|

## <a name="microsoftcontainerservicemanagedclusters"></a>Microsoft.ContainerService/managedClusters
|Vérifications exécutées|
|---|
|<ul><li>Le cluster est-il opérationnel ?</li><li>Les services principaux sont-ils disponibles dans le cluster ?</li><li>Tous les nœuds de cluster sont-ils prêts ?</li><li>Le principal du service est-il actuel et valide ?</li></ul>|

## <a name="microsoftdatafactoryfactories"></a>Microsoft.datafactory/factories
|Vérifications exécutées|
|---|
|<ul><li>Y a-t-il eu des échecs d’exécution du pipeline ?</li><li>Le cluster hébergeant la fabrique de données est-il sain ?</li></ul>|

## <a name="microsoftdatalakeanalyticsaccounts"></a>Microsoft.datalakeanalytics/accounts
|Vérifications exécutées|
|---|
|<ul><li>Les utilisateurs ont-ils rencontré des problèmes lors de l’envoi ou du listage de leurs travaux Data Lake Analytics ?</li><li>Les travaux Data Lake Analytics sont-ils incapables de se terminer en raison d’erreurs système ?</li></ul>|


## <a name="microsoftdatalakestoreaccounts"></a>Microsoft.datalakestore/accounts
|Vérifications exécutées|
|---|
|<ul><li>Les utilisateurs ont-ils rencontré des problèmes lors du chargement de données vers le Data Lake Store ?</li><li>Les utilisateurs ont-ils rencontré des problèmes lors du téléchargement de données à partir du Data Lake Store ?</li></ul>|

## <a name="microsoftdatamigrationservices"></a>Microsoft.datamigration/services
|Vérifications exécutées|
|---|
|<ul><li>Votre service de migration de base de données a-t-il été incapable de procéder au provisionnement ?</li><li>Votre service de migration de base de données s’est-il arrêté pour cause d’inactivité ou à la suite d’une requête effectuée par l’utilisateur ?</li></ul>|

## <a name="microsoftdatashareaccounts"></a>Microsoft.DataShare/accounts
|Vérifications exécutées|
|---|
|<ul><li>Le compte Data Share est-il opérationnel ?</li><li>Le cluster hébergeant Data Share est-il disponible ?</li></ul>|

## <a name="microsoftdbformariadbservers"></a>Microsoft.DBforMariaDB/servers
|Vérifications exécutées|
|---|
|<ul><li>Le serveur est-il non disponible en raison d’une maintenance ?</li><li>Le serveur est-il non disponible en raison d’une reconfiguration ?</li></ul>|

## <a name="microsoftdbformysqlservers"></a>Microsoft.DBforMySQL/servers
|Vérifications exécutées|
|---|
|<ul><li>Le serveur est-il non disponible en raison d’une maintenance ?</li><li>Le serveur est-il non disponible en raison d’une reconfiguration ?</li></ul>|

## <a name="microsoftdbforpostgresqlservers"></a>Microsoft.DBforPostgreSQL/servers
|Vérifications exécutées|
|---|
|<ul><li>Le serveur est-il non disponible en raison d’une maintenance ?</li><li>Le serveur est-il non disponible en raison d’une reconfiguration ?</li></ul>|

## <a name="microsoftdevicesiothubs"></a>Microsoft.devices/iothubs
|Vérifications exécutées|
|---|
|<ul><li>Le hub IoT est-il opérationnel ?</li></ul>|

## <a name="microsoftdigitaltwinsdigitaltwinsinstances"></a>Microsoft.DigitalTwins/DigitalTwinsInstances
|Vérifications exécutées|
|---|
|<ul><li>L’instance Azure Digital Twins est-elle opérationnelle ?</li></ul>|

## <a name="microsoftdocumentdbdatabaseaccounts"></a>Microsoft.documentdb/databaseAccounts
|Vérifications exécutées|
|---|
|<ul><li>Y a-t-il eu des demandes de base de données ou de collection non servies en raison d’une indisponibilité du service Azure Cosmos DB ?</li><li>Y a-t-il eu des demandes de document non servies en raison d’une indisponibilité du service Azure Cosmos DB ?</li></ul>|

## <a name="microsofteventhubnamespaces"></a>Microsoft.eventhub/namespaces
|Vérifications exécutées|
|---|
|<ul><li>L’espace de noms Event Hubs rencontre-t-il des erreurs générées par l’utilisateur ?</li><li>L’espace de noms Event Hubs est-il en cours de mise à niveau ?</li></ul>|

## <a name="microsofthdinsightclusters"></a>Microsoft.hdinsight/clusters
|Vérifications exécutées|
|---|
|<ul><li>Les services principaux sont-ils disponibles dans le cluster HDInsight ?</li><li>Le cluster HDInsight peut-il accéder à la clé pour le chiffrement BYOK au repos ?</li></ul>|

## <a name="microsoftiotcentraliotapps"></a>Microsoft.IoTCentral/IoTApps
|Vérifications exécutées|
|---|
|<ul><li>L’application IoT Central est-elle disponible ?</li></ul>|

## <a name="microsoftkeyvaultvaults"></a>Microsoft.KeyVault/vaults
|Vérifications exécutées|
|---|
|<ul><li>Les demandes de coffre de clés échouent-elles en raison de problèmes de plateforme Azure KeyVault ?</li><li>Les demandes de coffre de clés sont-elles limitées en raison de trop de requêtes effectuées par le client ?</li></ul>|

## <a name="microsoftkustoclusters"></a>Microsoft.Kusto/clusters
|Vérifications exécutées|
|---|
|<ul><li>Le cluster connaît-il de taux de réussite d’ingestion des données faibles ?</li><li>Le cluster connaît-il une lance d’ingestion élevée ?</li><li>Le cluster rencontre-t-il un grand nombre d’échecs de requêtes ?</li></ul>|

## <a name="microsoftmachinelearningwebservices"></a>Microsoft.MachineLearning/webServices
|Vérifications exécutées|
|---|
|<ul><li>Le service web est-il opérationnel ?</li></ul>|

## <a name="microsoftmediamediaservices"></a>Microsoft.Media/mediaservices
|Vérifications exécutées|
|---|
|<ul><li>Le service multimédia est-il opérationnel ?</li></ul>|

## <a name="microsoftnetworkapplicationgateways"></a>Microsoft.network/applicationgateways
|Vérifications exécutées|
|---|
|<ul><li>Les performances de la passerelle d’application se sont-elles dégradées ?</li><li>La passerelle d’application est-elle disponible ?</li></ul>|

## <a name="microsoftnetworkconnections"></a>Microsoft.network/connections
|Vérifications exécutées|
|---|
|<ul><li>Le tunnel VPN est-il connecté ?</li><li>Existe-t-il des conflits de configuration dans la connexion ?</li><li>Les clés prépartagées sont-elles correctement configurées ?</li><li>L’appareil VPN local est-il accessible ?</li><li>Existe-t-il des non-correspondances dans la stratégie de sécurité IPSec/IKE ?</li><li>La connexion VPN S2S est-elle correctement configurée ou dans un état d’échec ?</li><li>La connexion de réseau virtuel à réseau virtuel est-elle correctement configurée ou dans un état d’échec ?</li></ul>|

## <a name="microsoftnetworkexpressroutecircuits"></a>Microsoft.network/expressroutecircuits
|Vérifications exécutées|
|---|
|<ul><li>Le circuit ExpressRoute est-il sain ?</li></ul>|

## <a name="microsoftnetworkfrontdoors"></a>Microsoft.network/frontdoors
|Vérifications exécutées|
|---|
|<ul><li>Les back-ends Front Door répondent-ils avec des erreurs aux sondes d’intégrité ?</li><li>Les modifications de configuration sont-elles retardées ?</li></ul>|

## <a name="microsoftnetworkloadbalancers"></a>Microsoft.network/LoadBalancers
|Vérifications exécutées|
|---|
|<ul><li>Les points de terminaison d’équilibrage de charge sont-ils disponibles ?</li></ul>|

## <a name="microsoftnetworktrafficmanagerprofiles"></a>Microsoft.network/trafficmanagerprofiles
|Vérifications exécutées|
|---|
|<ul><li>Y a-t-il des problèmes affectant le profil de Traffic Manager ?</li></ul>|

## <a name="microsoftnetworkvirtualnetworkgateways"></a>Microsoft.network/virtualNetworkGateways
|Vérifications exécutées|
|---|
|<ul><li>La passerelle VPN est-elle accessible à partir d’internet ?</li><li>La passerelle VPN est-elle en mode veille ?</li><li>Le service VPN est-il en cours d’exécution sur la passerelle ?</li></ul>|

## <a name="microsoftnotificationhubsnamespace"></a>Microsoft.NotificationHubs/namespace
|Vérifications exécutées|
|---|
|<ul><li>Les opérations de runtime comme l’inscription, l’installation ou l’envoi peuvent-elles être effectuées sur l’espace de noms ?</li></ul>|

## <a name="microsoftoperationalinsightsworkspaces"></a>Microsoft.operationalinsights/workspaces
|Vérifications exécutées|
|---|
|<ul><li>Y a-t-il des retards d’indexation pour l’espace de travail ?</li></ul>|

## <a name="microsoftpowerbidedicatedcapacities"></a>Microsoft.PowerBIDedicated/Capacities
|Vérifications exécutées|
|---|
|<ul><li>La ressource de capacité est-elle opérationnelle ?</li><li>Toutes les charges de travail sont-elles opérationnelles ?</li></ul>

## <a name="microsoftsearchsearchservices"></a>Microsoft.search/searchServices
|Vérifications exécutées|
|---|
|<ul><li>Les opérations de diagnostic peuvent-elles être exécutées sur le cluster ?</li></ul>|

## <a name="microsoftservicebusnamespaces"></a>Microsoft.ServiceBus/namespaces
|Vérifications exécutées|
|---|
|<ul><li>Les clients rencontrent-ils des erreurs Service Bus générées par les utilisateurs ?</li><li>Les utilisateurs rencontrent-ils une augmentation des erreurs transitoires dues à une mise à niveau de l’espace de nommage Service Bus ?</li></ul>|

## <a name="microsoftservicefabricclusters"></a>Microsoft.ServiceFabric/clusters
|Vérifications exécutées|
|---|
|<ul><li>Le cluster Service Fabric est-il opérationnel ?</li><li>Le cluster Service Fabric peut-il être géré via Azure Resource Manager ?</li></ul>|

## <a name="microsoftsqlmanagedinstancesdatabases"></a>Microsoft.SQL/managedInstances/databases
|Vérifications exécutées|
|---|
|<ul><li>La base de données est-elle opérationnelle ?</li></ul>|

## <a name="microsoftsqlserversdatabases"></a>Microsoft.SQL/servers/databases
|Vérifications exécutées|
|---|
|<ul><li>Y a-t-il eu des connexions à la base de données ?</li></ul>|

## <a name="microsoftstoragestorageaccounts"></a>Microsoft.Storage/storageAccounts
|Vérifications exécutées|
|---|
|<ul><li>Les demandes de lecture des données à partir du compte de stockage échouent-elles en raison de problèmes de la plateforme de Stockage Azure ?</li><li>Les demandes d’écriture des données sur le compte de stockage échouent-elles en raison de problèmes de la plateforme de Stockage Azure ?</li><li>Le cluster de stockage où réside le compte de stockage est-il indisponible ?</li></ul>|

## <a name="microsoftstreamanalyticsstreamingjobs"></a>Microsoft.StreamAnalytics/streamingjobs
|Vérifications exécutées|
|---|
|<ul><li>Tous les hôtes sur lesquels le travail est exécuté sont-ils en cours d’exécution ?</li><li>Le travail a-t-il échoué à démarrer ?</li><li>Y a-t-il des mises à niveau du runtime en cours ?</li><li>Le travail est-il dans un état prévu (par exemple en cours d’exécution ou arrêtée par le client) ?</li><li>Le travail a-t-il rencontré des exceptions de mémoire insuffisante ?</li><li>Y a-t-il des mises à jour de calcul planifiées en cours ?</li><li>Le Gestionnaire d’exécution (plan de contrôle) est-il disponible ?</li></ul>|

## <a name="microsoftwebserverfarms"></a>Microsoft.web/serverFarms
|Vérifications exécutées|
|---|
|<ul><li>Le serveur hôte est-il en cours d’exécution ?</li><li>Internet Information Services (IIS) est-il en cours d’exécution ?</li><li>L’équilibreur de charge est-il en cours d’exécution ?</li><li>Le plan App Service peut-il être joint à partir du centre de données ?</li><li>Le compte de stockage hébergeant le contenu des sites de la batterie de serveurs est-il disponible ?</li></ul>|

## <a name="microsoftwebsites"></a>Microsoft.web/sites
|Vérifications exécutées|
|---|
|<ul><li>Le serveur hôte est-il en cours d’exécution ?</li><li>Le serveur Internet Information est-il en cours d’exécution ?</li><li>L’équilibreur de charge est-il en cours d’exécution ?</li><li>L’application web peut-elle être jointe à partir du centre de données ?</li><li>Le compte de stockage hébergeant le contenu du site est-il disponible ?</li></ul>|

## <a name="next-steps"></a>Étapes suivantes
-  Pour en savoir plus à leur sujet, voir [Présentation du tableau de bord d’Azure Service Health](service-health-overview.md) et [Présentation d’Azure Resource Health](resource-health-overview.md). 
-  [Forum aux questions sur Azure Resource Health](resource-health-faq.md)
- Configurez des alertes afin d’être averti des problèmes d’intégrité. Pour plus d’informations, voir [Configurer des alertes pour les événements d’intégrité du service](./alerts-activity-log-service-notifications-portal.md).