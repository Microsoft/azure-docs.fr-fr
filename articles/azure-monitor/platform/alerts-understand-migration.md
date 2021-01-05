---
title: Comprendre la migration pour les alertes Azure Monitor
description: Comprendre le fonctionnement de la migration des alertes et résoudre les problèmes.
ms.topic: conceptual
ms.date: 07/10/2019
ms.author: yalavi
author: yalavi
ms.subservice: alerts
ms.openlocfilehash: 6509425f11b09a2fa5229f9dd68a508241391925
ms.sourcegitcommit: fbb620e0c47f49a8cf0a568ba704edefd0e30f81
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91875918"
---
# <a name="understand-migration-options-to-newer-alerts"></a>Comprendre les options de migration vers les alertes plus récentes

Les alertes classiques ne sont [mises hors service](./monitoring-classic-retirement.md), bien qu’elles soient toujours utilisées pour les ressources qui ne prennent pas encore en charge les nouvelles alertes. Une nouvelle date sera annoncée bientôt pour la migration des alertes restantes ([cloud Azure Government](../../azure-government/documentation-government-welcome.md) et [Azure China 21Vianet](https://docs.azure.cn/)).

Cet article explique comment fonctionne l’outil de migration manuelle et de migration volontaire, qui sera utilisé pour migrer les règles d’alerte restantes. Il décrit également les solutions pour certains problèmes courants.

> [!IMPORTANT]
> Les alertes de journal d’activité (y compris les alertes d’intégrité de service) et les alertes de journal ne sont pas affectées par la migration. La migration s’applique uniquement aux règles d’alerte classiques décrites [ici](monitoring-classic-retirement.md#retirement-of-classic-monitoring-and-alerting-platform).

> [!NOTE]
> Si vos règles d’alerte classiques ne sont pas valides, par exemple si elles portent sur des [métriques déconseillées](#classic-alert-rules-on-deprecated-metrics) ou des ressources qui ont été supprimées, elles ne seront pas migrées et ne seront pas disponibles une fois le service mis hors service.

## <a name="manually-migrating-classic-alerts-to-newer-alerts"></a>Migration manuelle des alertes classiques vers des alertes plus récentes

Les clients qui souhaitent migrer manuellement leurs alertes restantes peuvent déjà le faire à l’aide des sections suivantes. Ces sections définissent également les métriques qui sont mises hors service par le fournisseur de ressources et qui ne peuvent actuellement pas être migrées directement.

### <a name="guest-metrics-on-virtual-machines"></a>Métriques d’invité sur les machines virtuelles

Avant de pouvoir créer de nouvelles alertes de métrique sur les métriques d’invité, les métriques d’invité doivent être envoyées au magasin de métriques personnalisées Azure Monitor. Suivez ces instructions pour activer le récepteur Azure Monitor dans les paramètres de diagnostic :

- [Activer les métriques invitées pour les machines virtuelles Windows](collect-custom-metrics-guestos-resource-manager-vm.md)
- [Activer les métriques invitées pour les machines virtuelles Linux](collect-custom-metrics-linux-telegraf.md)

Une fois ces étapes effectuées, vous pouvez créer de nouvelles alertes de métrique sur les métriques d’invité. Et une fois que vous avez créé les nouvelles alertes de métrique, vous pouvez supprimer les alertes classiques.

### <a name="storage-account-metrics"></a>Métriques du compte de stockage

Toutes les alertes classiques sur les comptes de stockage peuvent être migrées, à l’exception des alertes sur ces métriques :

- PercentAuthorizationError
- PercentClientOtherError
- PercentNetworkError
- PercentServerOtherError
- PercentSuccess
- PercentThrottlingError
- PercentTimeoutError
- AnonymousThrottlingError
- SASThrottlingError
- ThrottlingError

Les règles d’alerte classiques sur les métriques de pourcentage doivent être migrées selon [le mappage entre les anciennes et nouvelles métriques de stockage](../../storage/common/storage-metrics-migration.md#metrics-mapping-between-old-metrics-and-new-metrics). Les seuils devront être modifiés en conséquence, car la nouvelle métrique disponible est absolue.

Les règles d’alerte classiques sur AnonymousThrottlingError, SASThrottlingError et ThrottlingError doivent être fractionnées en deux nouvelles alertes, car il n’existe aucune métrique combinée qui fournit les mêmes fonctionnalités. Les seuils devront être adaptés de manière appropriée.

### <a name="cosmos-db-metrics"></a>Métriques Cosmos DB

Toutes les alertes classiques sur les mesures de Cosmos DB peuvent être migrées, à l’exception des alertes sur ces mesures :

- Nombre moyen de requêtes par seconde
- Niveau de cohérence
- Http 2xx
- Http 3xx
- HTTP 400
- Http 401
- Erreur interne du serveur
- Nombre maximal de RUPM consommées par minute
- Nombre maximal de RU par seconde
- Nombre d’échecs de requêtes Mongo
- Requêtes ayant échoué supprimées Mongo
- Requêtes d’insertion ayant échoué Mongo
- Autres demandes ayant échoué Mongo
- Autres frais des requêtes Mongo
- Autres taux des requêtes Mongo
- Requêtes interrogées ayant échoué Mongo
- Requêtes ayant échoué mises à jour Mongo
- Latence de lecture observée
- Latence d’écriture observée
- Disponibilité des services
- Capacité de stockage
- Requêtes limitées
- Total de requêtes

Le nombre moyen de demandes par seconde, le niveau de cohérence, le nombre maximal de RUPM consommées par minute, le nombre maximal de RU par seconde, la latence de lecture observée, la latence d’écriture observée et la capacité de stockage ne sont pas disponible actuellement dans le [nouveau système](metrics-supported.md#microsoftdocumentdbdatabaseaccounts).

Les alertes sur les mesures de requête comme HTTP 2xx, HTTP 3xx, HTTP 400, HTTP 401, erreur de serveur interne, disponibilité du service, demandes limitées et nombre total de requêtes ne sont pas migrées, car la manière dont les requêtes sont comptées est différente entre les mesures classiques et les nouvelles mesures. Les alertes doivent être recréées manuellement avec des seuils ajustés.

Les alertes sur les mesures de requêtes Mongo ayant échoué doivent être fractionnées en plusieurs alertes, car il n’existe aucune mesure combinée qui offre les mêmes fonctionnalités. Les seuils devront être adaptés de manière appropriée.

### <a name="classic-compute-metrics"></a>Calcul classique de mesures

Les alertes sur les mesures de calcul classiques ne seront pas migrées à l’aide de l’outil de migration, car les ressources de calcul classiques ne sont pas encore prises en charge avec les nouvelles alertes. La prise en charge des nouvelles alertes sur ces types de ressources est actuellement disponible en version préliminaire publique, et les clients peuvent recréer de nouvelles règles d’alerte équivalentes basées sur leurs règles d’alerte classiques.

### <a name="classic-alert-rules-on-deprecated-metrics"></a>Règles d’alerte classiques sur des métriques déconseillées

Il s’agit de règles d’alerte classiques sur des métriques qui étaient précédemment prises en charge, mais qui ont été déconseillées par la suite. Un petit pourcentage de clients peut avoir des règles d’alerte classiques non valides sur de telles métriques. Dans la mesure où ces règles d’alerte ne sont pas valides, elles ne seront pas migrées.

| Type de ressource| Métriques déconseillées |
|-------------|----------------- |
| Microsoft.DBforMySQL/servers | compute_consumption_percent, compute_limit |
| Microsoft.DBforPostgreSQL/servers | compute_consumption_percent, compute_limit |
| Microsoft.Network/publicIPAddresses | defaultddostriggerrate |
| Microsoft.SQL/servers/databases | service_level_objective, storage_limit, storage_used, throttling, dtu_consumption_percent, storage_used |
| Microsoft.Web/hostingEnvironments/multirolepools | averagememoryworkingset |
| Microsoft.Web/hostingEnvironments/workerpools | bytesreceived, httpqueuelength |

## <a name="how-equivalent-new-alert-rules-and-action-groups-are-created"></a>Comment les nouvelles règles d’alerte et les groupes d’actions équivalents sont créés

L’outil de migration convertit vos règles d’alerte classiques en nouvelles règles d’alerte et groupes d’actions équivalents. Pour la plupart des règles d’alerte classiques, les nouvelles règles d’alerte équivalentes portent sur la même métrique avec les mêmes propriétés, comme `windowSize` et `aggregationType`. Toutefois, il existe des règles d’alerte classiques portant sur des métriques qui ont une métrique équivalente différente dans le nouveau système. Les principes suivants s’appliquent à la migration des alertes classiques, sauf mention contraire dans la section ci-dessous :

- **Fréquence** : Définit la fréquence de vérification de la condition pour une règle d’alerte classique ou nouvelle. La `frequency` dans les règles d’alerte classiques n’était pas configurable par l’utilisateur et était toujours de 5 minutes pour tous les types de ressources à l’exception des composants d’Application Insights, pour lesquels elle était de 1 minute. La fréquence des règles équivalentes est donc également définie sur 5 minutes et 1 minute respectivement.
- **Type d’agrégation** : Définit la façon dont les métriques sont agrégées pendant la fenêtre d’intérêt. Le `aggregationType` est également le même entre les alertes classiques et les nouvelles alertes pour la plupart des métriques. Dans certains cas, étant donné que la métrique est différente entre les alertes classiques et les nouvelles alertes, le `aggregationType` équivalent ou le `primary Aggregation Type` défini pour la métrique est utilisé.
- **Unités** : La propriété de la métrique sur laquelle l’alerte est créée. Certaines métriques équivalentes ont des unités différentes. Le seuil est ajusté de manière appropriée en fonction des besoins. Par exemple, si la métrique d’origine utilise les secondes comme unité mais que la nouvelle métrique équivalente utilise les millisecondes en tant qu’unité, le seuil d’origine est multiplié par 1000 pour garantir le même comportement.
- **Taille de la fenêtre** : Définit la fenêtre pendant laquelle les données de métrique sont agrégées pour être comparées au seuil. Pour les valeurs de `windowSize` standard telles que 5 minutes, 15 minutes, 30 minutes, 1 heure, 3 heures, 6 heures, 12 heures ou 1 jour, aucune modification n’est effectuée pour la nouvelle règle d’alerte équivalente. Pour les autres valeurs, la `windowSize` la plus proche est choisie. Pour la plupart des clients, cette modification n’a aucun impact. Pour un petit pourcentage de clients, il pourrait être nécessaire d’ajuster le seuil pour obtenir un comportement rigoureusement identique.

Dans les sections suivantes, nous détaillons les métriques qui ont une métrique différente et équivalente dans le nouveau système. Les métriques identiques pour les règles d’alerte classiques et nouvelles ne sont pas répertoriées. Vous trouverez une liste des métriques prises en charge dans le nouveau système [ici](metrics-supported.md).

### <a name="microsoftstorageaccountsservices"></a>Microsoft.StorageAccounts/services

Pour les services de compte de stockage comme les objets blob, les tables, les fichiers et les files d’attente, les métriques suivantes sont mappées aux métriques équivalentes, comme indiqué ci-dessous :

| Métrique dans les alertes classiques | Métrique équivalente dans les nouvelles alertes | Commentaires|
|--------------------------|---------------------------------|---------|
| AnonymousAuthorizationError| Métrique de transactions avec les dimensions « ResponseType » = « AuthorizationError » et « Authentication » = « Anonymous »| |
| AnonymousClientOtherError | Métrique de transactions avec les dimensions « ResponseType » = « ClientOtherError » et « Authentication » = « Anonymous » | |
| AnonymousClientTimeOutError| Métrique de transactions avec les dimensions « ResponseType » = « ClientTimeOutError » et « Authentication » = « Anonymous » | |
| AnonymousNetworkError | Métrique de transactions avec les dimensions « ResponseType » = « NetworkError » et « Authentication » = « Anonymous » | |
| AnonymousServerOtherError | Métrique de transactions avec les dimensions « ResponseType » = « ServerOtherError » et « Authentication » = « Anonymous » | |
| AnonymousServerTimeOutError | Métrique de transactions avec les dimensions « ResponseType » = « ServerTimeOutError » et « Authentication » = « Anonymous » | |
| AnonymousSuccess | Métrique de transactions avec les dimensions « ResponseType » = « Success » et « Authentication » = « Anonymous » | |
| AuthorizationError | Métrique de transactions avec les dimensions « ResponseType » = « AuthorizationError » | |
| AverageE2ELatency | SuccessE2ELatency | |
| AverageServerLatency | SuccessServerLatency | |
| Capacité | BlobCapacity | Utilisez 'average' au lieu de 'last' pour `aggregationType`. Les métriques s’appliquent uniquement aux services blob |
| ClientOtherError | Métrique de transactions avec les dimensions « ResponseType » = « ClientOtherError »  | |
| ClientTimeoutError | Métrique de transactions avec les dimensions « ResponseType » = « ClientTimeOutError » | |
| ContainerCount | ContainerCount | Utilisez 'average' au lieu de 'last' pour `aggregationType`. Les métriques s’appliquent uniquement aux services blob |
| NetworkError | Métrique de transactions avec les dimensions « ResponseType » = « NetworkError » | |
| ObjectCount | BlobCount| Utilisez 'average' au lieu de 'last' pour `aggregationType`. Les métriques s’appliquent uniquement aux services blob |
| SASAuthorizationError | Métrique de transactions avec les dimensions « ResponseType » = « AuthorizationError » et « Authentication » = « SAS » | |
| SASClientOtherError | Métrique de transactions avec les dimensions « ResponseType » = « ClientOtherError » et « Authentication » = « SAS » | |
| SASClientTimeOutError | Métrique de transactions avec les dimensions « ResponseType » = « ClientTimeOutError » et « Authentication » = « SAS » | |
| SASNetworkError | Métrique de transactions avec les dimensions « ResponseType » = « NetworkError » et « Authentication » = « SAS » | |
| SASServerOtherError | Métrique de transactions avec les dimensions « ResponseType » = « ServerOtherError » et « Authentication » = « SAS » | |
| SASServerTimeOutError | Métrique de transactions avec les dimensions « ResponseType » = « ServerTimeOutError » et « Authentication » = « SAS » | |
| SASSuccess | Métrique de transactions avec les dimensions « ResponseType » = « Success » et « Authentication » = « SAS » | |
| ServerOtherError | Métrique de transactions avec les dimensions « ResponseType » = « ServerOtherError » | |
| ServerTimeOutError | Métrique de transactions avec les dimensions « ResponseType » = « ServerTimeOutError »  | |
| Succès | Métrique de transactions avec les dimensions « ResponseType » = « Success » | |
| TotalBillableRequests| Transactions | |
| TotalEgress | Sortie | |
| TotalIngress | Entrée | |
| TotalRequests | Transactions | |

### <a name="microsoftinsightscomponents"></a>Microsoft.insights/components

Pour Application Insights, les métriques équivalentes sont indiquées ci-dessous :

| Métrique dans les alertes classiques | Métrique équivalente dans les nouvelles alertes | Commentaires|
|--------------------------|---------------------------------|---------|
| availability.availabilityMetric.value | availabilityResults/availabilityPercentage|   |
| availability.durationMetric.value | availabilityResults/duration| Multipliez le seuil d’origine par 1000, car les unités de mesure classiques sont exprimées en secondes et les nouvelles en millisecondes.  |
| basicExceptionBrowser.count | exceptions/browser|  Utilisez 'count' au lieu de 'sum' pour `aggregationType`. |
| basicExceptionServer.count | exceptions/server| Utilisez 'count' au lieu de 'sum' pour `aggregationType`.  |
| clientPerformance.clientProcess.value | browserTimings/processingDuration| Multipliez le seuil d’origine par 1000, car les unités de mesure classiques sont exprimées en secondes et les nouvelles en millisecondes.  |
| clientPerformance.networkConnection.value | browserTimings/networkDuration|  Multipliez le seuil d’origine par 1000, car les unités de mesure classiques sont exprimées en secondes et les nouvelles en millisecondes. |
| clientPerformance.receiveRequest.value | browserTimings/receiveDuration| Multipliez le seuil d’origine par 1000, car les unités de mesure classiques sont exprimées en secondes et les nouvelles en millisecondes.  |
| clientPerformance.sendRequest.value | browserTimings/sendDuration| Multipliez le seuil d’origine par 1000, car les unités de mesure classiques sont exprimées en secondes et les nouvelles en millisecondes.  |
| clientPerformance.total.value | browserTimings/totalDuration| Multipliez le seuil d’origine par 1000, car les unités de mesure classiques sont exprimées en secondes et les nouvelles en millisecondes.  |
| performanceCounter.available_bytes.value | performanceCounters/memoryAvailableBytes|   |
| performanceCounter.io_data_bytes_per_sec.value | performanceCounters/processIOBytesPerSecond|   |
| performanceCounter.number_of_exceps_thrown_per_sec.value | performanceCounters/exceptionsPerSecond|   |
| performanceCounter.percentage_processor_time_normalized.value | performanceCounters/processCpuPercentage|   |
| performanceCounter.percentage_processor_time.value | performanceCounters/processCpuPercentage| Le seuil doit être modifié en conséquence, car la métrique d’origine était partagée sur tous les cœurs et la nouvelle métrique est normalisée sur un seul cœur. L’outil de migration ne modifie pas les seuils.  |
| performanceCounter.percentage_processor_total.value | performanceCounters/processorCpuPercentage|   |
| performanceCounter.process_private_bytes.value | performanceCounters/processPrivateBytes|   |
| performanceCounter.request_execution_time.value | performanceCounters/requestExecutionTime|   |
| performanceCounter.requests_in_application_queue.value | performanceCounters/requestsInQueue|   |
| performanceCounter.requests_per_sec.value | performanceCounters/requestsPerSecond|   |
| request.duration | requests/duration| Multipliez le seuil d’origine par 1000, car les unités de mesure classiques sont exprimées en secondes et les nouvelles en millisecondes.  |
| request.rate | requests/taux|   |
| requestFailed.count | requests/failed| Utilisez 'count' au lieu de 'sum' pour `aggregationType`.   |
| view.count | pageViews/count| Utilisez 'count' au lieu de 'sum' pour `aggregationType`.   |

### <a name="microsoftdocumentdbdatabaseaccounts"></a>Microsoft.DocumentDB/databaseAccounts

Pour Cosmos DB, les mesures équivalentes sont indiquées ci-dessous :

| Métrique dans les alertes classiques | Métrique équivalente dans les nouvelles alertes | Commentaires|
|--------------------------|---------------------------------|---------|
| AvailableStorage     |AvailableStorage|   |
| Taille des données | DataUsage| |
| Nombre de documents | DocumentCount||
| Taille d'index | IndexUsage||
| Nombre de frais des requêtes Mongo| MongoRequestCharge avec la dimension « CommandName » = « count »||
| Nombre de taux des requêtes Mongo | MongoRequestCount avec la dimension « CommandName » = « count »||
| Frais de requête de suppression Mongo | MongoRequestCharge avec la dimension « CommandName » = « delete »||
| Taux de requête de suppression Mongo | MongoRequestsCount avec la dimension « CommandName » = « delete »||
| Frais de requête d’insertion Mongo | MongoRequestCharge avec la dimension « CommandName » = « insert »||
| Taux de requête d’insertion Mongo | MongoRequestsCount avec la dimension « CommandName » = « insert »||
| Frais de requêtes d’interrogation Mongo | MongoRequestCharge avec la dimension « CommandName » = « find »||
| Taux de requêtes d’interrogation Mongo | MongoRequestsCount avec la dimension « CommandName » = « find »||
| Frais de requête de mise à jour Mongo | MongoRequestCharge avec la dimension « CommandName » = « update »||
| Service indisponible| ServiceAvailability||
| TotalRequestUnits | TotalRequestUnits||

### <a name="how-equivalent-action-groups-are-created"></a>Comment les groupes d’actions équivalents sont créés

Les règles d’alerte classiques avaient les actions d’e-mail, de webhook, d’application logique et de runbook liées à la règle d’alerte elle-même. Les nouvelles règles d’alerte utilisent des groupes d’actions qui peuvent être réutilisés dans plusieurs règles d’alerte. L’outil de migration crée un groupe d’actions unique pour les actions similaires, quel que soit le nombre de règles d’alerte utilisant l’action. Les groupes d’actions créés par l’outil de migration utilisent le format d’attribution de noms « Migrated_AG* ».

> [!NOTE]
> Les alertes classiques ont envoyé des e-mails localisés en fonction des paramètres régionaux de l’administrateur classique lorsqu’ils sont utilisés pour notifier les rôles d’administrateur classiques. Les nouveaux e-mails d’alerte sont envoyés par le biais de groupes d’actions et sont uniquement en anglais.

## <a name="rollout-phases"></a>Phases de déploiement

L’outil de migration est livré en plusieurs phases aux clients qui utilisent des règles d’alerte classiques. Les propriétaires d’abonnements recevront un e-mail lorsque leur abonnement sera prêt à être migré à l’aide de l’outil.

> [!NOTE]
> Étant donné que l’outil est déployé en plusieurs phases, vous pourriez constater que certains de vos abonnements ne sont pas encore prêts à être migrés lors des premières phases.

La plupart des abonnements sont actuellement marqués comme prêts pour la migration. Seuls les abonnements qui ont des alertes classiques sur les types de ressources suivants ne sont pas encore prêts pour la migration.

- Microsoft.classicCompute/domainNames/slots/roles
- Microsoft.insights/components

## <a name="who-can-trigger-the-migration"></a>Qui peut déclencher la migration ?

Tout utilisateur ayant le rôle intégré de contributeur de surveillance au niveau de l’abonnement peut déclencher la migration. Les utilisateurs qui ont un rôle personnalisé avec les autorisations suivantes peuvent également déclencher la migration :

- */read
- Microsoft.Insights/actiongroups/*
- Microsoft.Insights/AlertRules/*
- Microsoft.Insights/metricAlerts/*
- Microsoft.AlertsManagement/smartDetectorAlertRules/*

> [!NOTE]
> En plus d’avoir les autorisations ci-dessus, votre abonnement doit être inscrit avec le fournisseur de ressources Microsoft.AlertsManagement. Cela est nécessaire pour réussir la migration des alertes d’anomalies d’échec sur Application Insights. 

## <a name="common-problems-and-remedies"></a>Problèmes courants et solutions

Après avoir [déclenché la migration](alerts-using-migration-tool.md), vous recevez un e-mail aux adresses que vous avez fournies pour vous avertir que la migration est terminée ou si une action est nécessaire de votre part. Cette section décrit certains problèmes courants et comment les résoudre.

### <a name="validation-failed"></a>Échec de validation

En raison des modifications apportées récemment aux règles d’alerte classiques dans votre abonnement, l’abonnement ne peut pas être migré. Ce problème est temporaire. Vous pourrez redémarrer la migration une fois que l’état de migration sera de nouveau **prêt pour la migration**, dans quelques jours.

### <a name="scope-lock-preventing-us-from-migrating-your-rules"></a>Un verrou d’étendue nous empêche de migrer vos règles

Dans le cadre de la migration, de nouvelles alertes de métrique et de nouveaux groupes d’actions seront créés, et les règles d’alerte classiques seront alors supprimées. Toutefois, un verrou d’étendue peut nous empêcher de créer ou de supprimer des ressources. En fonction du verrou d’étendue, certaines ou l’intégralité des règles n’ont pas pu être migrées. Vous pouvez résoudre ce problème en supprimant le verrou d’étendue pour l’abonnement, le groupe de ressources ou la ressource, lequel est listé dans l’[outil de migration](https://portal.azure.com/#blade/Microsoft_Azure_Monitoring/MigrationBladeViewModel), puis en redéclenchant la migration. Le verrou d’étendue ne peut pas être désactivé et doit être supprimé pendant la durée du processus de migration. [En savoir plus sur la gestion des verrous d’étendue](../../azure-resource-manager/management/lock-resources.md#portal).

### <a name="policy-with-deny-effect-preventing-us-from-migrating-your-rules"></a>Une stratégie avec effet de refus nous empêche de migrer vos règles

Dans le cadre de la migration, de nouvelles alertes de métrique et de nouveaux groupes d’actions seront créés, et les règles d’alerte classiques seront alors supprimées. Toutefois, une affectation [Azure Policy](../../governance/policy/index.yml) peut nous empêcher de créer des ressources. En fonction de l’affectation de stratégie, certaines ou l’intégralité des règles n’ont pas pu être migrées. Les affectations de stratégie qui bloquent le processus sont listées dans l’[outil de migration](https://portal.azure.com/#blade/Microsoft_Azure_Monitoring/MigrationBladeViewModel). Pour résoudre ce problème, effectuez l’une ou l’autre des étapes suivantes :

- Excluez les abonnements, les groupes de ressources ou les ressources individuelles pendant la durée du processus de migration à partir de l’affectation de stratégie. [En savoir plus sur la gestion des étendues de l’exclusion de stratégie](../../governance/policy/tutorials/create-and-manage.md#remove-a-non-compliant-or-denied-resource-from-the-scope-with-an-exclusion).
- Définissez le « mode d’application » sur **Désactivé** sur l’affectation de stratégie. [En savoir plus sur la propriété enforcementMode de l’affectation de stratégie](../../governance/policy/concepts/assignment-structure.md#enforcement-mode).
- Définissez une exemption Azure Policy (préversion) sur les abonnements, les groupes de ressources ou les ressources individuelles pour l’affectation de stratégie. [En savoir plus sur la structure d’exemption Azure Policy](../../governance/policy/concepts/exemption-structure.md).
- Supprimez l’effet ou modifiez-le en le remplaçant par un effet désactivé, d’audit, d’ajout ou de modification (qui, par exemple, permet de résoudre les problèmes liés aux balises manquantes). [En savoir plus sur la gestion des effets de stratégie](../../governance/policy/concepts/definition-structure.md#policy-rule).

## <a name="next-steps"></a>Étapes suivantes

- [Guide pratique pour utiliser l’outil de migration](alerts-using-migration-tool.md)
- [Préparer la migration](alerts-prepare-migration.md)
