---
title: FAQ Azure Monitor | Microsoft Docs
description: Réponses aux questions fréquemment posées sur Azure Monitor.
services: azure-monitor
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 10/08/2020
ms.openlocfilehash: b770e4488b6edb1c2d3749066315b552c0b5b40a
ms.sourcegitcommit: d22a86a1329be8fd1913ce4d1bfbd2a125b2bcae
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/26/2020
ms.locfileid: "96186165"
---
# <a name="azure-monitor-frequently-asked-questions"></a>Questions fréquemment posées sur Azure Monitor

Cette FAQ Microsoft est une liste de questions fréquemment posées sur Azure Monitor. Si vous avez d’autres questions, rendez-vous sur le [forum de discussion](/answers/questions/topics/single/24223.html) et publiez vos questions. Lorsqu’une question est fréquemment posée, nous l’ajoutons à cet article pour qu’elle soit facile et rapide à trouver.


## <a name="general"></a>Général

### <a name="what-is-azure-monitor"></a>Qu’est-ce qu’Azure Monitor ?
[Azure Monitor](overview.md) est un service Azure qui supervise les performances et la disponibilité des applications et des services dans Azure, d’autres environnements cloud ou en local. Azure Monitor collecte les données de plusieurs sources au sein d'une plateforme de données commune où elles peuvent être analysées afin d'identifier des tendances et des anomalies. Les fonctionnalités enrichies d'Azure Monitor vous aident à identifier les situations critiques susceptibles d’affecter votre application et à y répondre rapidement.

### <a name="whats-the-difference-between-azure-monitor-log-analytics-and-application-insights"></a>Quelle est la différence entre Azure Monitor, Log Analytics et Application Insights ?
En septembre 2018, Microsoft a combiné Azure Monitor, Log Analytics et Application Insights au sein d'un seul service afin d'offrir une supervision de bout en bout de vos applications et des composants sur lesquels elles s’appuient. Les fonctionnalités de Log Analytics et d'Application Insights n’ont pas changé, mais certaines ont été renommées Azure Monitor afin de mieux refléter leur nouvelle étendue. Le moteur de données de journal et le langage de requête de Log Analytics sont désormais appelés Journaux Azure Monitor. Consultez [Mises à jour de la terminologie Azure Monitor](terminology.md).

### <a name="what-does-azure-monitor-cost"></a>Combien coûte Azure Monitor ?
Les fonctionnalités d'Azure Monitor automatiquement activées, telles que la collecte des métriques et des journaux d’activité, sont proposées gratuitement. Un coût est associé à d’autres fonctionnalités telles que les requêtes de journal et les alertes. Pour obtenir des informations de tarification détaillées, consultez la page [Tarification Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/).

### <a name="how-do-i-enable-azure-monitor"></a>Comment activer Azure Monitor ?
Azure Monitor est activé lors de la création d'un nouvel abonnement Azure, et le [journal d'activité](./platform/platform-logs-overview.md), de même que les [métriques](platform/data-platform-metrics.md) de la plateforme sont automatiquement collectés. Créez des [paramètres de diagnostic](platform/diagnostic-settings.md) pour collecter des informations plus détaillées sur le fonctionnement de vos ressources Azure, et ajoutez des [solutions de supervision](insights/solutions.md), de même que des [insights](./monitor-reference.md) afin de fournir des analyses supplémentaires sur les données collectées pour des services spécifiques. 

### <a name="how-do-i-access-azure-monitor"></a>Comment accéder à Azure Monitor ?
Accédez à toutes les fonctionnalités et données Azure Monitor à partir du menu **Superviser** du portail Azure. La section **Supervision** du menu des différents services Azure permet d'accéder aux mêmes outils avec des données filtrées sur une ressource spécifique. Les données Azure Monitor sont également accessibles pour divers scénarios à l’aide de CLI, de PowerShell et d’une API REST.

### <a name="is-there-an-on-premises-version-of-azure-monitor"></a>Existe-t-il une version locale d'Azure Monitor ?
Non. Azure Monitor est un service cloud évolutif qui traite et stocke d'importantes quantités de données, même si Azure Monitor peut superviser des ressources locales, ainsi que dans d'autres clouds.

### <a name="can-azure-monitor-monitor-on-premises-resources"></a>Azure Monitor peut-il superviser des ressources locales ?
Oui, en plus de la collecte des données de supervision à partir des ressources Azure, Azure Monitor peut collecter des données issues de machines virtuelles et d’applications dans d’autres clouds et en local. Consultez [Sources des données de surveillance pour Azure Monitor](platform/data-sources.md).

### <a name="does-azure-monitor-integrate-with-system-center-operations-manager"></a>Azure Monitor s'intègre-t-il à System Center Operations Manager ?
Vous pouvez connecter votre groupe d’administration System Center Operations Manager existant à Azure Monitor afin de collecter des données entre les agents et les journaux Azure Monitor. Cela vous permet d’utiliser des requêtes de journal et une solution pour analyser les données collectées auprès des agents. Vous pouvez également configurer des agents System Center Operations Manager existants pour envoyer des données directement à Azure Monitor. Consultez [Connecter Operations Manager à Azure Monitor](platform/om-agents.md).

### <a name="what-ip-addresses-does-azure-monitor-use"></a>Quelles sont les adresses IP utilisées par Azure Monitor ?
Pour obtenir la liste des adresses IP et ports requis pour permettre aux agents et autres ressources externes d'accéder à Azure Monitor, consultez [Adresses IP utilisées par Application Insights et Log Analytics](app/ip-addresses.md). 

## <a name="monitoring-data"></a>Données de surveillance

### <a name="where-does-azure-monitor-get-its-data"></a>Où Azure Monitor obtient-il ses données ?
Azure Monitor collecte des données à partir de diverses sources, notamment des journaux et des métriques de la plateforme Azure, des ressources, des applications personnalisées et des agents s’exécutant sur des machines virtuelles. D’autres services tels qu'Azure Security Center et Network Watcher collectent des données dans un espace de travail Log Analytics afin de les analyser avec les données Azure Monitor. Vous pouvez également envoyer des données personnalisées à Azure Monitor à l’aide de l’API REST pour les journaux ou métriques. Consultez [Sources des données de surveillance pour Azure Monitor](platform/data-sources.md).

### <a name="what-data-is-collected-by-azure-monitor"></a>Quelles sont les données collectées par Azure Monitor ? 
Azure Monitor collecte des données à partir de diverses sources dans des [journaux](platform/data-platform-logs.md) ou [métriques](platform/data-platform-metrics.md). Chaque type de données présente des avantages qui lui sont propres et prend en charge un ensemble spécifique de fonctionnalités dans Azure Monitor. Il existe une même base de données de métriques pour chaque abonnement Azure, mais vous pouvez créer plusieurs espaces de travail de Log Analytics afin de collecter les journaux selon vos besoins. Consultez [Plateforme de données Azure Monitor](platform/data-platform.md).

### <a name="is-there-a-maximum-amount-of-data-that-i-can-collect-in-azure-monitor"></a>Existe-t-il une quantité maximale de données que je peux collecter dans Azure Monitor ?
Il n’existe aucune limite en termes de données métriques collectées, mais ces données sont stockées pendant un maximum de 93 jours. Consultez [Rétention des métriques](platform/data-platform-metrics.md#retention-of-metrics). Il n’existe aucune limite quant à la quantité de données de journal collectées, en dehors du fait qu’elles peuvent affecter le niveau tarifaire que vous choisissez pour l’espace de travail Log Analytics. Consultez les [détails de la tarification](https://azure.microsoft.com/pricing/details/monitor/).

### <a name="how-do-i-access-data-collected-by-azure-monitor"></a>Comment accéder aux données collectées par Azure Monitor ?
Les insights et les solutions permettent une expérience d'utilisation personnalisée des données stockées dans Azure Monitor. Vous pouvez utiliser directement les données de journal à l’aide d’une requête de journal écrite en langage de requête Kusto (KQL). Dans le portail Azure, vous pouvez écrire et exécuter des requêtes, et analyser les données de manière interactive à l’aide de Log Analytics. Dans le portail Azure, analysez des métriques avec Metrics Explorer. Consultez [Analyser les données de journal dans Azure Monitor](log-query/log-query-overview.md) et [Prise en main d’Azure Metrics Explorer](platform/metrics-getting-started.md).

## <a name="solutions-and-insights"></a>Solutions et insights

### <a name="what-is-an-insight-in-azure-monitor"></a>Qu’est-ce qu’un insight dans Azure Monitor ?
Les insights offrent une expérience de supervision personnalisée pour des services Azure spécifiques. Ils utilisent les mêmes métriques et journaux que les autres fonctionnalités d'Azure Monitor, mais peuvent collecter des données supplémentaires et offrir une expérience unique dans le portail Azure. Consultez [Insights dans Azure Monitor](./monitor-reference.md).

Pour afficher des insights dans le portail Azure, consultez la section **Insights** du menu **Superviser** ou la section **Supervision** du menu du service.

### <a name="what-is-a-solution-in-azure-monitor"></a>Qu’est-ce qu’une solution dans Azure Monitor ?
Les solutions de supervision sont des jeux de logique empaquetés permettant de superviser une application ou un service spécifique selon les fonctionnalités Azure Monitor. Elles collectent les données de journal dans Azure Monitor et proposent des requêtes de journal et des affichages à des fins d'analyse, avec une expérience commune dans le portail Azure. Consultez [Solutions de supervision dans Azure Monitor](insights/solutions.md).

Pour afficher les solutions dans le portail Azure, cliquez sur **Plus** dans la section **Insights** du menu **Superviser**. Cliquez sur **Ajouter** pour ajouter des solutions supplémentaires à l’espace de travail.

## <a name="logs"></a>Journaux d’activité

### <a name="whats-the-difference-between-azure-monitor-logs-and-azure-data-explorer"></a>Quelle est la différence entre les journaux Azure Monitor et Azure Data Explorer ?
L’Explorateur de données Azure est un service d’exploration de données rapide et hautement évolutive pour les données des journaux et les données de télémétrie. Les journaux Azure Monitor s’appuient sur Azure Data Explorer et utilisent le même langage de requête Kusto (KQL), à quelques différences mineures près. Consultez [Différences propres au langage de requête de journal d'Azure Monitor](/azure/data-explorer/kusto/query/).

### <a name="how-do-i-retrieve-log-data"></a>Comment récupérer des données de journal ?
Toutes les données sont récupérées à partir d’un espace de travail Log Analytics à l’aide d’une requête de journal écrite en langage de requête Kusto (KQL). Vous pouvez écrire vos propres requêtes ou utiliser des solutions et insights incluant des requêtes de journal pour une application ou un service spécifique. Consultez [Vue d’ensemble des requêtes de journal dans Azure Monitor](log-query/log-query-overview.md).
p
### <a name="can-i-delete-data-from-a-log-analytics-workspace"></a>Puis-je supprimer des données à partir d’un espace de travail Log Analytics ?
Les données sont supprimées d’un espace de travail en fonction de sa [période de rétention](platform/manage-cost-storage.md#change-the-data-retention-period). Vous pouvez supprimer des données spécifiques pour des raisons de confidentialité ou de conformité. Consultez [Comment exporter et supprimer des données privées](platform/personal-data-mgmt.md#how-to-export-and-delete-private-data) pour plus d’informations.


### <a name="what-is-a-log-analytics-workspace"></a>Qu'est-ce qu'un espace de travail Log Analytics ?
Toutes les données de journal collectées par Azure Monitor sont stockées dans un espace de travail Log Analytics. Un espace de travail est essentiellement un conteneur dans lequel les données de journal issues de différentes sources sont collectées. Vous pouvez disposer d'un même espace de travail Log Analytics pour toutes vos données de supervision ou plus, selon vos besoins. Consultez [Conception de votre déploiement de journaux Azure Monitor](platform/design-logs-deployment.md).

### <a name="can-you-move-an-existing-log-analytics-workspace-to-another-azure-subscription"></a>Est-il possible de déplacer un espace de travail Log Analytics existant vers un autre abonnement Azure ?
Vous pouvez déplacer un espace de travail entre groupes de ressources ou abonnements, mais pas vers une autre région. Consultez [Déplacer un espace de travail Log Analytics vers un autre abonnement ou groupe de ressources](platform/move-workspace.md).

### <a name="why-cant-i-see-query-explorer-and-save-buttons-in-log-analytics"></a>Pourquoi ne puis-je pas voir les boutons Explorateur de requêtes et Enregistrer dans Log Analytics ?

Les boutons **Explorateur de requêtes**, **Enregistrer** et **Nouvelle règle d’alerte** ne sont pas disponibles lorsque l’[étendue de la requête](log-query/scope.md) est définie sur une ressource spécifique. Pour créer des alertes, ainsi qu’enregistrer ou charger une requête, Log Analytics doit avoir une étendue définie sur un espace de travail. Pour ouvrir Log Analytics dans le contexte d’un espace de travail, sélectionnez **Journaux** dans le menu **Azure Monitor**. Le dernier espace de travail utilisée est sélectionné, mais vous pouvez sélectionner n’importe quel autre espace de travail. Voir [Étendue de requête de journal et intervalle de temps dans la fonctionnalité Log Analytics d’Azure Monitor](log-query/scope.md)

### <a name="why-am-i-getting-the-error-register-resource-provider-microsoftinsights-for-this-subscription-to-enable-this-query-when-opening-log-analytics-from-a-vm"></a>Pourquoi l’erreur : « Inscrivez le fournisseur de ressources ’Microsoft.Insights’ pour cet abonnement afin d’activer cette requête » s’affiche-t-elle lors de l’ouverture de Log Analytics à partir d’une machine virtuelle ? 
De nombreux fournisseurs de ressources sont inscrits automatiquement, mais il se peut que vous deviez en inscrire certains manuellement. L’étendue pour l’inscription est toujours l’abonnement. Pour plus d’informations, consultez [Fournisseurs et types de ressources](../azure-resource-manager/management/resource-providers-and-types.md#azure-portal).

### <a name="why-am-i-getting-no-access-error-message-when-opening-log-analytics-from-a-vm"></a>Pourquoi un message d’erreur indiquant une absence d’accès s’affiche-t-il lors de l’ouverture de Log Analytics à partir d’une machine virtuelle ? 
Pour voir les journaux d’activité de machine virtuelle, vous devez disposer de l’autorisation de lecture sur les espaces de travail qui les stockent. Dans ce cas, votre administrateur doit vous accorder des autorisations dans Azure.

## <a name="metrics"></a>Mesures

### <a name="why-are-metrics-from-the-guest-os-of-my-azure-virtual-machine-not-showing-up-in-metrics-explorer"></a>Pourquoi les métriques du système d’exploitation invité de ma machine virtuelle Azure n’apparaissent pas dans Metrics Explorer ?
Les [métriques de la plateforme](insights/monitor-azure-resource.md#monitoring-data) sont collectées automatiquement pour les ressources Azure. Vous devez toutefois configurer certains paramètres pour collecter les métriques du système d’exploitation invité d’une machine virtuelle. Pour une machine virtuelle Windows, installez l’extension de diagnostic et configurez le récepteur Azure Monitor comme décrit dans [Installer et configurer l’extension Diagnostics Azure pour Windows (WAD)](platform/diagnostics-extension-windows-install.md). Pour Linux, installez l’agent Telegraf comme décrit dans [Collecter des métriques personnalisées pour une machine virtuelle Linux avec l’agent InfluxData Telegraf](platform/collect-custom-metrics-linux-telegraf.md).

## <a name="alerts"></a>Alertes

### <a name="what-is-an-alert-in-azure-monitor"></a>Qu’est-ce qu’une alerte dans Azure Monitor ?
Les alertes vous avertissent de façon proactive lorsque des conditions significatives sont détectées dans vos données de surveillance. Elles permettent d’identifier et de résoudre les problèmes avant que les utilisateurs de votre système ne les remarquent. Il existe plusieurs types d’alertes :

- Métrique : la valeur de métrique dépasse un certain seuil.
- Requête de journal : résultats d'une requête de journal correspondant à des critères définis.
- Journal d’activité : événement de journal d’activité correspondant à des critères définis.
- Test web : résultats d'un test de disponibilité correspondant à des critères définis.


Consultez [Vue d’ensemble des alertes dans Microsoft Azure](platform/alerts-overview.md).


### <a name="what-is-an-action-group"></a>Qu'est-ce qu'un groupe d'actions ?
Un groupe d’actions est une collection de notifications et d’actions qui peuvent être déclenchées par une alerte. Plusieurs alertes peuvent utiliser un même groupe d’actions, ce qui vous permet de tirer parti des ensembles communs de notifications et d’actions. Consultez [Créer et gérer des groupes d’actions dans le portail Azure](platform/action-groups.md).


### <a name="what-is-an-action-rule"></a>Qu'est-ce qu'une règle d'action ?
Une règle d’action vous permet de modifier le comportement d’un ensemble d’alertes correspondant à certains critères. Vous pouvez ainsi répondre à certaines exigences, comme désactiver des actions d’alerte lors d’une fenêtre de maintenance. Vous pouvez également appliquer un groupe d’actions à un ensemble d’alertes plutôt que de les appliquer directement aux règles d’alerte. Consultez [Règles d’action](platform/alerts-action-rules.md).

## <a name="agents"></a>Agents

### <a name="does-azure-monitor-require-an-agent"></a>Azure Monitor requiert-il un agent ?
Un agent est uniquement requis pour collecter les données du système d’exploitation et les charges de travail des machines virtuelles. Les machines virtuelles peuvent se trouver dans Azure, dans un autre environnement cloud ou en local. Consultez [Vue d’ensemble des agents Azure Monitor](platform/agents-overview.md).


### <a name="whats-the-difference-between-the-azure-monitor-agents"></a>Quelle est la différence entre les agents Azure Monitor ?
L’extension Azure Diagnostic est destinée aux machines virtuelles Azure et collecte des données vers les métriques Azure Monitor, Stockage Azure et Azure Event Hubs. L’agent Log Analytics est destiné aux machines virtuelles dans Azure, à un autre environnement cloud ou en local, et collecte des données dans les journaux Azure Monitor. Dependency Agent requiert l’agent Log Analytics, ainsi que les détails et dépendances du processus. Consultez [Vue d’ensemble des agents Azure Monitor](platform/agents-overview.md).


### <a name="does-my-agent-traffic-use-my-expressroute-connection"></a>Le trafic de mon agent utilise-t-il ma connexion ExpressRoute ?
Le trafic vers Azure Monitor utilise le circuit ExpressRoute de peering Microsoft. Consultez la [documentation ExpressRoute](../expressroute/expressroute-faqs.md#supported-services) pour une description des différents types de trafic ExpressRoute. 

### <a name="how-can-i-confirm-that-the-log-analytics-agent-is-able-to-communicate-with-azure-monitor"></a>Comment vérifier qu’un agent Log Analytics est en mesure de communiquer avec Azure Monitor ?
Dans le Panneau de configuration de l’ordinateur de l’agent, sélectionnez **Paramètres et sécurité**, **Microsoft Monitoring Agent. Sous l'onglet **Azure Log Analytics (OMS)** , une icône en forme de coche verte confirme que l’agent est en mesure de communiquer avec le Azure Monitor. Une icône d’avertissement jaune signifie que l’agent rencontre des problèmes. L'arrêt du service **Microsoft Monitoring Agent** constitue une cause courante. Utilisez le Gestionnaire de contrôle des services pour redémarrer le service.

### <a name="how-do-i-stop-the-log-analytics-agent-from-communicating-with-azure-monitor"></a>Comment interrompre la communication de l'agent Log Analytics avec Azure Monitor ?
Pour les agents connectés directement à Log Analytics, ouvrez le Panneau de configuration et sélectionnez **Sécurité et paramètres**, **Microsoft Monitoring Agent**. Sous l'onglet **Azure Log Analytics (OMS)** , supprimez tous les espaces de travail répertoriés. Dans System Center Operations Manager, supprimez l’ordinateur de la liste des ordinateurs gérés par Log Analytics. Operations Manager met à jour la configuration de l’agent pour qu’il ne fasse plus de rapports à Log Analytics. 

### <a name="how-much-data-is-sent-per-agent"></a>Quelle est la quantité de données envoyées par agent ?
La quantité de données envoyées par agent dépend :

* des solutions activées ;
* du nombre de journaux d’activité et de compteurs de performances collectés ;
* du volume de données des journaux d’activité.

Pour plus d'informations, consultez [Gérer l’utilisation et les coûts avec les journaux Azure Monitor](platform/manage-cost-storage.md).

Pour les ordinateurs en mesure d’exécuter l’agent Wire Data, lancez la requête suivante afin de connaître la quantité de données envoyées :

```Kusto
WireData
| where ProcessName == "C:\\Program Files\\Microsoft Monitoring Agent\\Agent\\MonitoringHost.exe" 
| where Direction == "Outbound" 
| summarize sum(TotalBytes) by Computer 
```

### <a name="how-much-network-bandwidth-is-used-by-the-microsoft-management-agent-mma-when-sending-data-to-azure-monitor"></a>Quelle est la quantité de bande passante réseau utilisée par Microsoft Management Agent (MMA) lorsqu’il envoie des données à Azure Monitor ?
La bande passante est fonction de la quantité de données envoyées. Les données sont compressées à mesure de leur envoi sur le réseau


### <a name="how-can-i-be-notified-when-data-collection-from-the-log-analytics-agent-stops"></a>Comment être informé lorsque la collecte de données issues de l'agent Log Analytics s’arrête ?

Suivez les étapes décrites dans [Créer une nouvelle alerte de journal](platform/alerts-metric.md) pour être averti lorsque la collecte de données s’arrête. Utilisez les paramètres suivants pour la règle d'alerte :

- **Définir la condition d’alerte** : spécifiez votre espace de travail Log Analytics en tant que cible de la ressource.
- **Critères d’alerte** 
   - **Nom du signal** : *Recherche personnalisée dans les journaux*
   - **Requête de recherche** : `Heartbeat | summarize LastCall = max(TimeGenerated) by Computer | where LastCall < ago(15m)`
   - **Logique d'alerte** : **Selon le** *nombre de résultats*, la **condition** *Supérieur à*, **Valeur de seuil** *0*
   - **Évaluée sur la base de** : **Période (en minutes)** *30*, **fréquence (en minutes)** *10*
- **Définir les détails de l’alerte** 
   - **Name** : *Collecte de données arrêtée*
   - **Gravité** : *Avertissement*

Spécifiez un [groupe d’actions](platform/action-groups.md) nouveau ou existant afin que l’alerte de journal corresponde aux critères, vous êtes informé si vous disposez d’une pulsation manquante pendant plus de 15 minutes.


### <a name="what-are-the-firewall-requirements-for-azure-monitor-agents"></a>Quelles sont les exigences en matière de pare-feu des agents Azure Monitor ?
Pour plus d'informations sur les exigences liées au pare-feu, consultez [Configuration requise du pare-feu réseau](platform/log-analytics-agent.md#network-requirements).


## <a name="visualizations"></a>Visualisations

### <a name="why-cant-i-see-view-designer"></a>Pourquoi ne puis-je pas voir le Concepteur de vues ?

Le Concepteur de vues n’est disponible que pour des utilisateurs disposant d’autorisations de niveau Contributeur ou supérieur dans l’espace de travail Log Analytics.

## <a name="application-insights"></a>Application Insights

### <a name="configuration-problems"></a>Problèmes de configuration
*J’ai des difficultés à configurer :*

* [Application .NET](app/asp-net-troubleshoot-no-data.md)
* [Analyse d’une application déjà en cours d’exécution](app/monitor-performance-live-website-now.md#troubleshoot)
* [Diagnostics Azure](platform/diagnostics-extension-to-application-insights.md)
* [Applications web Java](app/java-troubleshoot.md)

*Je ne reçois aucune donnée de mon serveur :*

* [Définir les exceptions de pare-feu](app/ip-addresses.md)
* [Configurer un serveur ASP.NET](app/monitor-performance-live-website-now.md)
* [Configurer un serveur Java](app/java-agent.md)

*Combien de ressources Application Insights déployer ?*

* [Guide pratique pour concevoir votre déploiement Application Insights : une ou plusieurs ressources Application Insights ?](app/separate-resources.md)

### <a name="can-i-use-application-insights-with-"></a>Puis-je utiliser Application Insights avec... ?

* [Applications web sur un serveur IIS sur une machine virtuelle Azure ou un groupe de machines virtuelles identiques Azure](app/azure-vm-vmss-apps.md)
* [Applications web sur un serveur IIS - en local ou sur une machine virtuelle](app/asp-net.md)
* [Applications web Java](app/java-get-started.md)
* [Applications Node.js](app/nodejs.md)
* [Applications web sur Azure](app/azure-web-apps.md)
* [Services cloud sur Azure](app/cloudservices.md)
* [Serveurs d’applications exécutés dans Docker](./azure-monitor-app-hub.yml)
* [Applications web d’une seule page](app/javascript.md)
* [SharePoint](app/sharepoint.md)
* [Applications de bureau Windows](app/windows-desktop.md)
* [autres plateformes](app/platforms.md)

### <a name="is-it-free"></a>Est-ce gratuit ?

Oui, pour une utilisation expérimentale. Dans le plan de tarification de base, votre application peut envoyer un certain quota de données chaque mois sans frais. Le quota gratuit est suffisamment grand pour couvrir le développement et la publication d’une application pour un petit nombre d’utilisateurs. Vous pouvez définir un plafond pour empêcher le traitement des données au-delà d’un certain seuil.

Les grands volumes de télémétrie sont facturés au Go. Nous vous offrons quelques conseils sur la façon de [limiter vos frais](app/pricing.md).

Le plan Entreprise implique un coût journalier pour chaque nœud de serveur web qui envoie des données de télémétrie. Il est adapté si vous souhaitez utiliser l’exportation continue à grande échelle.

[Lire le plan de tarification](https://azure.microsoft.com/pricing/details/application-insights/).

### <a name="how-much-does-it-cost"></a>Quel est le coût ?

* Ouvrez la **page Utilisation et estimation des coûts** dans une ressource Application Insights. Cette page contient un graphique reflétant l’utilisation récente. Vous pouvez définir une limite de volume de données si vous le souhaitez.
* Ouvrez le panneau [Facturation Azure](https://portal.azure.com/#blade/Microsoft_Azure_Billing/BillingBlade/Overview) pour voir vos factures sur l’ensemble des ressources.

### <a name="what-does-application-insights-modify-in-my-project"></a><a name="q14"></a>Que modifie Application Insights dans mon projet ?
Cela dépend du type de projet. Pour une application web :

* Ajoute ces fichiers à votre projet :
  * ApplicationInsights.config
  * ai.js
* Installe ces packages NuGet :
  * *Application Insights API* - API de base
  * *Application Insights API for Web Applications* - permet d'envoyer des données de télémétrie depuis le serveur
  * *Application Insights API for JavaScript Applications* - permet d'envoyer des données de télémétrie depuis le client
* Ces packages comprennent les assemblys suivants :
  * Microsoft.ApplicationInsights
  * Microsoft.ApplicationInsights.Platform
* Insère des éléments dans :
  * Web.config
  * packages.config
* (Nouveaux projets uniquement - si vous [ajoutez Application Insights à un projet existant][start], vous devez le faire manuellement). Insère des extraits de code dans le code du client et du serveur, afin de les initialiser avec l'ID de la ressource Application Insights. Par exemple, dans une application MVC, le code est inséré dans la page maître Views/Shared/\_Layout.cshtml

### <a name="how-do-i-upgrade-from-older-sdk-versions"></a>Comment mettre à niveau à partir d'anciennes versions du Kit de développement logiciel (SDK) ?
Consultez les [notes de publication](app/release-notes.md) du Kit de développement logiciel (SDK) adapté à votre type d'application.

### <a name="how-can-i-change-which-azure-resource-my-project-sends-data-to"></a><a name="update"></a>Comment puis-je changer la ressource Azure à laquelle mon projet envoie des données ?
Dans l’Explorateur de solutions, cliquez avec le bouton droit sur `ApplicationInsights.config` , puis sélectionnez **Mettre à jour Application Insights**. Vous pouvez envoyer les données à une ressource existante ou à une nouvelle ressource dans Azure. L'Assistant Mise à jour modifie la clé d'instrumentation dans ApplicationInsights.config, qui détermine où le Kit de développement logiciel (SDK) du serveur envoie vos données. Cela modifie également la clé où de l'affichage dans vos pages web, sauf si vous désactivez l'option « Tout mettre à jour ».

### <a name="can-i-use-providersmicrosoftinsights-componentsapiversions0-in-my-azure-resource-manager-deployments"></a>Puis-je utiliser `providers('Microsoft.Insights', 'components').apiVersions[0]` dans mes déploiements d’Azure Resource Manager ?

Nous vous déconseillons d’utiliser cette méthode pour renseigner la version de l’API. La version la plus récente peut représenter des publications de préversions susceptibles de contenir des changements radicaux. Même avec des publications plus récentes autres que de préversions, les versions API ne sont pas toujours rétrocompatibles avec des modèles existants, ou, dans certains cas, la version API peut ne pas être disponible pour tous les abonnements.

### <a name="what-is-status-monitor"></a>Qu’est-ce que Status Monitor ?

Une application de bureau que vous pouvez utiliser dans votre serveur web IIS pour faciliter la configuration d’Application Insights dans les applications web. Cette application ne collecte pas de données de télémétrie : vous pouvez l’arrêter lorsque vous ne configurez pas une application. 

[Plus d’informations](app/monitor-performance-live-website-now.md#questions)

### <a name="what-telemetry-is-collected-by-application-insights"></a>Quelles sont les données de télémétrie recueillies par Application Insights ?

À partir d’applications web serveur :

* Des requêtes HTTP
* [Dépendances](app/asp-net-dependencies.md). Appels passés aux instances SQL Database ; appels HTTP vers des services externes ; Azure Cosmos DB, table, stockage d’objets blob et file d’attente. 
* [Exceptions](app/asp-net-exceptions.md) et arborescences des appels de procédure.
* [Compteurs de performances](app/performance-counters.md) : si vous utilisez [Status Monitor](app/monitor-performance-live-website-now.md), la [surveillance Azure pour App Services](app/azure-web-apps.md), la [surveillance Azure pour le groupe de machines virtuelles identiques](app/azure-vm-vmss-apps.md) ou le [writer collectd d’Application Insights](app/java-collectd.md).
* [Événements et mesures personnalisés](app/api-custom-events-metrics.md) que vous codez.
* [Journaux d’activité de suivi](app/asp-net-trace-logs.md) si vous configurez le collecteur approprié.

À partir des [ pages web client](app/javascript.md) :

* [Nombre de pages consultées](app/usage-overview.md)
* [Appels AJAX](app/asp-net-dependencies.md). Requêtes transmises à partir d’un script en cours d’exécution.
* Données relatives au chargement de pages
* Nombre de sessions et d’utilisateurs
* [ID des utilisateurs authentifiés](app/api-custom-events-metrics.md#authenticated-users)

À partir d’autres sources, si vous les configurez :

* [Diagnostics Azure](platform/diagnostics-extension-to-application-insights.md)
* [Importer vers Analytics](platform/data-collector-api.md)
* [Log Analytics](platform/data-collector-api.md)
* [Logstash](platform/data-collector-api.md)

### <a name="can-i-filter-out-or-modify-some-telemetry"></a>Puis-je filtrer ou modifier des données de télémétrie ?

Oui, dans le serveur, vous pouvez écrire :

* Un processeur de télémétrie, pour filtrer ou ajouter des propriétés aux éléments de télémétrie sélectionnés avant de les envoyer à partir de votre application.
* Un initialiseur de télémétrie, pour ajouter des propriétés à tous les éléments de télémétrie.

En savoir plus pour [ASP.NET](app/api-filtering-sampling.md) ou [Java](app/java-filter-telemetry.md).

### <a name="how-are-city-countryregion-and-other-geo-location-data-calculated"></a>Comment sont calculées les données relative à la ville, au pays/à la région et aux autres emplacements géographiques ?

Nous recherchons l’adresse IP (IPv4 ou IPv6) du client web à l’aide de [GeoLite2](https://dev.maxmind.com/geoip/geoip2/geolite2/).

* Télémétrie de navigateur : nous collectons l’adresse IP de l’expéditeur.
* Télémétrie de serveur : le module Application Insights collecte l’adresse IP du client. Elle n’est pas collectée si `X-Forwarded-For` est défini.
* Pour en savoir plus sur la façon dont les données d’adresse IP et de géolocalisation sont collectées dans Application Insights, consultez cet [article](./app/ip-collection.md).

Vous pouvez configurer le `ClientIpHeaderTelemetryInitializer` pour récupérer l’adresse IP à partir d’un autre en-tête. Dans certains systèmes, par exemple, elle est déplacée vers `X-Originating-IP` par un proxy, un équilibreur de charge ou un CDN. [Plus d’informations](https://apmtips.com/posts/2016-07-05-client-ip-address/)

Vous pouvez [utiliser Power BI](app/export-power-bi.md ) pour afficher les données de télémétrie de votre requête sur une carte.


### <a name="how-long-is-data-retained-in-the-portal-is-it-secure"></a><a name="data"></a>Combien de temps les données sont-elles conservées dans le portail ? Sont-elles sécurisées ?
Consultez [Rétention de données et confidentialité][data].

### <a name="what-happens-to-application-insights-telemetry-when-a-server-or-device-loses-connection-with-azure"></a>Qu’advient-il de la télémétrie d’Application Insight lorsqu’un serveur ou un appareil perd la connexion avec Azure ?

Tous nos Kits de développement logiciel (SDK), y compris le Kit de développement logiciel (SDK) web, incluent le protocole « reliable transport » ou « robust transport ». Lorsque le serveur ou l’appareil perd la connexion avec Azure, la télémétrie est [stockée localement sur le système de fichiers](./app/data-retention-privacy.md#does-the-sdk-create-temporary-local-storage) (Kits de développement logiciel [SDK] de serveur) ou dans le stockage de session HTML5 (Kit de développement logiciel [SDK] web). Le Kit de développement logiciel (SDK) réessaiera régulièrement d’envoyer cette télémétrie jusqu’à ce que notre service d’ingestion la considère « obsolète » (48 heures pour les journaux, 30 minutes pour les métriques). La télémétrie obsolète sera supprimée. Dans certains cas, par exemple lorsque le stockage local est plein, aucune nouvelle tentative ne se produit.


### <a name="could-personal-data-be-sent-in-the-telemetry"></a>Est-ce que des informations personnelles peuvent être envoyées dans les données de télémétrie ?

Cela est possible si votre code envoie ce type de données. Cela peut également se produire si les variables dans les arborescences des appels de procédure incluent des données personnelles. Votre équipe de développement doit mener une évaluation des risques afin de s’assurer que les données personnelles sont correctement gérées. [En savoir plus sur la rétention et la confidentialité des données](app/data-retention-privacy.md).

**Tous** les octets de l’adresse web du client ont toujours la valeur 0 lors de la recherche des attributs d’emplacement géographique.

### <a name="my-instrumentation-key-is-visible-in-my-web-page-source"></a>La clé d’instrumentation est visible dans la source de ma page web. 

* Il s’agit d’une pratique courante dans les solutions de surveillance.
* Il ne peut pas être utilisé pour dérober vos données.
* Il peut servir à fausser vos alertes de données ou à déclencher des alertes.
* Aucun client ne nous a jamais signalé ce type de problème.

Vous pouvez :

* Utiliser deux clés d’instrumentation distinctes (ressources Application Insights distinctes) pour les données client et serveur. ou
* Écrire un proxy qui s’exécute sur votre serveur et que le client web utilise pour envoyer des données.

### <a name="how-do-i-see-post-data-in-diagnostic-search"></a><a name="post"></a>Comment consulter les données POST dans la fonction Recherche de diagnostic ?
Les données POST ne sont pas automatiquement consignées, mais vous pouvez utiliser un appel TrackTrace ; placez les données dans le paramètre message. Ce dernier présente une limite de taille plus longue que les limites relatives aux propriétés de type chaîne, bien que vous ne puissiez pas lui appliquer de filtres.

### <a name="should-i-use-single-or-multiple-application-insights-resources"></a>Dois-je utiliser une ou plusieurs ressources Application Insights ?

Utilisez une ressource unique pour tous les composants ou rôles dans un système métier unique. Utilisez des ressources distinctes pour les versions de développement, de test et de publication, et pour les applications indépendantes.

* [Suivez la discussion ici](app/separate-resources.md)
* [Exemple : service cloud avec des rôles web et worker](app/cloudservices.md)

### <a name="how-do-i-dynamically-change-the-instrumentation-key"></a>Comment modifier dynamiquement la clé d’instrumentation ?

* [Discussion ici](app/separate-resources.md)
* [Exemple : service cloud avec des rôles web et worker](app/cloudservices.md)

### <a name="what-are-the-user-and-session-counts"></a>Que désignent les nombres d’utilisateurs et de sessions ?

* Le SDK JavaScript définit un cookie utilisateur sur le client web pour identifier les utilisateurs récurrents, ainsi qu’un cookie de session pour regrouper des activités.
* S’il n’existe aucun script côté client, vous pouvez [définir des cookies sur le serveur](https://apmtips.com/posts/2016-07-09-tracking-users-in-api-apps/).
* Si un utilisateur réel utilise votre site dans différents navigateurs, ou s’il utilise une navigation privée ou encore des ordinateurs différents, il sera comptabilisé plusieurs fois.
* Pour identifier un utilisateur connecté sur différents ordinateurs et navigateurs, ajoutez un appel à [setAuthenticatedUserContext()](app/api-custom-events-metrics.md#authenticated-users).

### <a name="have-i-enabled-everything-in-application-insights"></a><a name="q17"></a> Comment savoir si j'ai activé tout ce qu'il faut pour utiliser Application Insights ?
| Ce qui suit doit s'afficher | Comment y accéder | Utilité |
| --- | --- | --- |
| Graphiques de disponibilité |[Tests web](app/monitor-web-app-availability.md) |Savoir si votre application web est active |
| Performances des applications de serveur (temps de réponse, etc.) |[Ajout d’Application Insights à votre projet](app/asp-net.md) ou [Installation d’AI Status Monitor sur le serveur](app/monitor-performance-live-website-now.md) (ou écrivez votre propre code pour [suivre des dépendances](app/api-custom-events-metrics.md#trackdependency)) |Détecter les problèmes de performances |
| Télémétrie des dépendances |[Installation d’AI Status Monitor sur le serveur](app/monitor-performance-live-website-now.md) |Diagnostiquer les problèmes relatifs à des bases de données ou à d'autres composants externes |
| Obtention de l'arborescence des appels de procédure à partir des exceptions |[Insertion d’appels TrackException dans votre code](app/asp-net-exceptions.md) (certains d’entre eux sont cependant signalés automatiquement) |Détecter et diagnostiquer les exceptions |
| Recherche des données de suivi des journaux |[Ajout d’un enregistreur de données](app/asp-net-trace-logs.md) |Diagnostiquer les exceptions et problèmes de performances |
| Principes fondamentaux d'utilisation des clients (vues de page, sessions, etc.) |[Initialiseur JavaScript dans les pages web](app/javascript.md) |Analyse de l'utilisation |
| Mesures personnalisées des clients |[Appels de suivi dans les pages web](app/api-custom-events-metrics.md) |Améliorer l'expérience utilisateur |
| Mesures personnalisées des serveurs |[Appels de suivi dans le serveur](app/api-custom-events-metrics.md) |Décisionnel |

### <a name="why-are-the-counts-in-search-and-metrics-charts-unequal"></a>Pourquoi les graphiques de recherche et de mesures contiennent-ils des valeurs différentes ?

[L’échantillonnage](app/sampling.md) réduit le nombre d’éléments de télémétrie (demandes, événements personnalisés, etc.) qui sont réellement envoyés à partir de votre application sur le portail. Dans la Recherche, vous pouvez voir le nombre d’éléments réellement reçus. Dans les graphiques de mesure qui affichent un nombre d’événements, vous obtenez le nombre d’événements d’origine qui se sont produits. 

Chaque élément transmis comporte une propriété `itemCount` qui indique le nombre d’événements d’origine que représente cet élément. Pour observer l’échantillonnage en action, vous pouvez exécuter cette requête dans Analytics :

```
    requests | summarize original_events = sum(itemCount), transmitted_events = count()
```

### <a name="how-do-i-move-an-application-insights-resource-to-a-new-region"></a>Comment déplacer une ressource Application Insights vers une nouvelle région ?

Le déplacement de ressources Application Insights existantes d’une région vers une autre n’est **actuellement pas pris en charge**. Les données d’historique que vous avez collectées **ne peuvent pas être migrées** vers une nouvelle région. La seule solution de contournement partiel consiste à procéder comme suit :

1. Créez une toute nouvelle ressource Application Insights ([classique](app/create-new-resource.md) ou [basée sur l’espace de travail](./app/create-workspace-resource.md)) dans la nouvelle région.
2. Recréez toutes les personnalisations uniques propres à la ressource d’origine dans la nouvelle ressource.
3. Modifiez votre application de façon à utiliser la [clé d’instrumentation](app/create-new-resource.md#copy-the-instrumentation-key) ou la [chaîne de connexion](app/sdk-connection-string.md) de la ressource de la nouvelle région.  
4. Testez pour confirmer que tout continue à fonctionner comme prévu avec votre nouvelle ressource Application Insights. 
5. À ce stade, vous pouvez supprimer la ressource d’origine, ce qui entraînera la **perte de toutes les données d’historique**. Ou vous pouvez conserver la ressource d’origine à des fins de création de rapports historiques selon la durée de ses paramètres de conservation des données.

Les personnalisations uniques qui doivent généralement être recréées ou mises à jour manuellement pour la ressource dans la nouvelle région incluent, mais sans s’y limiter :

- Recréer des tableaux de bord et des workbooks personnalisés. 
- Recréer ou mettre à jour l’étendue des alertes de journal/métrique personnalisées. 
- Recréer des alertes de disponibilité.
- Recréer les paramètres de contrôle d’accès en fonction du rôle Azure (Azure RBAC) personnalisés qui sont requis pour que les utilisateurs accèdent à la nouvelle ressource. 
- Répliquer les paramètres impliquant l’échantillonnage d’ingestion, la conservation des données, la limite quotidienne et l’activation des métriques personnalisées. Ces paramètres sont contrôlés via le volet **Utilisation et coûts estimés**.
- Toute intégration reposant sur des clés d’API telles que [annotations de version](./app/annotations.md), [canal de contrôle sécurisé des métriques en temps réel](app/live-stream.md#secure-the-control-channel), etc. Vous devrez générer de nouvelles clés d’API et mettre à jour l’intégration associée. 
- L’exportation continue dans les ressources classiques doit être reconfigurée.
- Les paramètres de diagnostic des ressources basées sur l’espace de travail doivent être à nouveau configurés.

> [!NOTE]
> Si la ressource que vous créez dans une nouvelle région remplace une ressource classique, nous vous recommandons d’explorer les avantages de la [création d’une ressource basée sur l’espace de travail](app/create-workspace-resource.md) ou de la [migration de votre ressource existante vers une ressource basée sur l’espace de travail](app/convert-classic-resource.md). 

### <a name="automation"></a>Automatisation

#### <a name="configuring-application-insights"></a>Configuration d'Application Insights

Vous pouvez [écrire des scripts PowerShell](app/powershell.md) à l’aide du moniteur de ressources Azure pour :

* Créer et mettre à jour des ressources Application Insights.
* Définir le plan de tarification.
* Récupérer la clé d’instrumentation.
* Ajouter une alerte métrique.
* Ajouter un test de disponibilité.

Vous ne peut pas définir un rapport état Metrics Explorer ou configurer une exportation continue.

#### <a name="querying-the-telemetry"></a>Interrogation des données de télémétrie

Utilisez [l’API REST](https://dev.applicationinsights.io/) pour exécuter des requêtes [Analytics](./log-query/log-query-overview.md).

### <a name="how-can-i-set-an-alert-on-an-event"></a>Comment puis-je définir une alerte sur un événement ?

Les alertes Azure portent uniquement sur les mesures. Créez une mesure personnalisée qui dépasse un seuil de valeur chaque fois que l’événement se produit. Puis définissez une alerte sur la mesure. Vous recevez une notification chaque fois que la mesure dépasse le seuil dans les deux directions. Vous ne recevez pas de notification avant le dernier dépassement, que la valeur initiale soit élevée ou faible ; il y a toujours une latence de quelques minutes.

### <a name="are-there-data-transfer-charges-between-an-azure-web-app-and-application-insights"></a>Existe-t-il des frais de transfert de données entre une application web Azure et Application Insights ?

* Si votre application web Azure est hébergée dans un centre de données qui comporte un point de terminaison de collecte Application Insights, aucuns frais ne sont appliqués. 
* S’il n’existe aucun point de terminaison de collecte dans votre centre de données hôte, les données de télémétrie de votre application généreront des [frais Azure de trafic sortant](https://azure.microsoft.com/pricing/details/bandwidth/).

Cela ne dépend pas de l’emplacement où est hébergée votre ressource Application Insights. Cela dépend simplement de la distribution de nos points de terminaison.

### <a name="can-i-send-telemetry-to-the-application-insights-portal"></a>Puis-je envoyer des données de télémétrie au portail Application Insights ?

Nous vous recommandons d’utiliser nos Kits de développement logiciel (SDK) et [l’API du Kit SDK](app/api-custom-events-metrics.md). Il existe des variantes du SDK pour les différentes [plateformes](app/platforms.md). Ces kits de développement logiciel gèrent la mise en mémoire tampon, la compression, la limitation, les nouvelles tentatives, etc. Le [schéma d’ingestion](https://github.com/microsoft/ApplicationInsights-dotnet/tree/master/BASE/Schema/PublicSchema) et le [protocole de point de terminaison](https://github.com/MohanGsk/ApplicationInsights-Home/blob/master/EndpointSpecs/ENDPOINT-PROTOCOL.md) sont toutefois publics.

### <a name="can-i-monitor-an-intranet-web-server"></a>Puis-je surveiller un serveur web intranet ?

Oui, mais vous devez autoriser le trafic vers nos services à l’aide d’exceptions de pare-feu ou de redirections proxy.
- QuickPulse `https://rt.services.visualstudio.com:443` 
- ApplicationIdProvider `https://dc.services.visualstudio.com:443` 
- TelemetryChannel `https://dc.services.visualstudio.com:443` 


Passez en revue la liste complète des services et adresses IP en cliquant [ici](app/ip-addresses.md).

#### <a name="firewall-exception"></a>Exception de pare-feu

Permettez à votre serveur web d’envoyer la télémétrie à nos points de terminaison. 

#### <a name="gateway-redirect"></a>Redirection de passerelle

Acheminez le trafic entre votre serveur et une passerelle sur votre intranet en remplaçant les points de terminaison dans votre configuration. Si ces propriétés « Endpoint » ne sont pas présentes dans votre configuration, ces classes utiliseront les valeurs par défaut indiquées dans l’exemple ApplicationInsights.config ci-dessous. 

Votre passerelle doit acheminer le trafic vers l’adresse de base de notre point de terminaison. Dans votre configuration, remplacez les valeurs par défaut par `http://<your.gateway.address>/<relative path>`.


##### <a name="example-applicationinsightsconfig-with-default-endpoints"></a>Exemple ApplicationInsights.config avec les points de terminaison par défaut :
```xml
<ApplicationInsights>
  ...
  <TelemetryModules>
    <Add Type="Microsoft.ApplicationInsights.Extensibility.PerfCounterCollector.QuickPulse.QuickPulseTelemetryModule, Microsoft.AI.PerfCounterCollector">
      <QuickPulseServiceEndpoint>https://rt.services.visualstudio.com/QuickPulseService.svc</QuickPulseServiceEndpoint>
    </Add>
  </TelemetryModules>
    ...
  <TelemetryChannel>
     <EndpointAddress>https://dc.services.visualstudio.com/v2/track</EndpointAddress>
  </TelemetryChannel>
  ...
  <ApplicationIdProvider Type="Microsoft.ApplicationInsights.Extensibility.Implementation.ApplicationId.ApplicationInsightsApplicationIdProvider, Microsoft.ApplicationInsights">
    <ProfileQueryEndpoint>https://dc.services.visualstudio.com/api/profiles/{0}/appId</ProfileQueryEndpoint>
  </ApplicationIdProvider>
  ...
</ApplicationInsights>
```

> [!NOTE]
> ApplicationIdProvider est disponible à partir de la version 2.6.0.



#### <a name="proxy-passthrough"></a>Pass-through du proxy

Vous pouvez obtenir le pass-through du proxy en configurant un proxy au niveau de l’ordinateur ou de l’application.
Pour en savoir plus, consultez l’article de dotnet sur [DefaultProxy](/dotnet/framework/configure-apps/file-schema/network/defaultproxy-element-network-settings).
 
 Exemple de fichier Web.config :
 ```xml
<system.net>
    <defaultProxy>
      <proxy proxyaddress="http://xx.xx.xx.xx:yyyy" bypassonlocal="true"/>
    </defaultProxy>
</system.net>
```
 

### <a name="can-i-run-availability-web-tests-on-an-intranet-server"></a>Puis-je exécuter des tests web de disponibilité sur un serveur intranet ?

Nos [tests web](app/monitor-web-app-availability.md) s’exécutent sur des points de présence qui sont répartis dans le monde entier. Il existe deux solutions :

* Pare-feu : autorisez les demandes envoyées à votre serveur parmi [la longue liste modifiable d’agents de test web](app/ip-addresses.md).
* Écrivez votre propre code pour envoyer des demandes régulières à votre serveur depuis votre intranet. Vous pouvez exécuter des tests web Visual Studio à cet effet. Le testeur peut envoyer les résultats à Application Insights à l’aide de l’API TrackAvailability().

### <a name="how-long-does-it-take-for-telemetry-to-be-collected"></a>Combien de temps dure la collecte de données de télémétrie ?

La plupart des données Application Insights ont une latence de moins de 5 minutes. Certaines données peuvent prendre plus de temps, en général les fichiers journaux plus volumineux. Pour plus d’informations, consultez le [contrat SLA Application Insights](https://azure.microsoft.com/support/legal/sla/application-insights/v1_2/).



<!--Link references-->

[data]: app/data-retention-privacy.md
[platforms]: app/platforms.md
[start]: app/app-insights-overview.md
[windows]: app/app-insights-windows-get-started.md

### <a name="http-502-and-503-responses-are-not-always-captured-by-application-insights"></a>Les réponses HTTP 502 et 503 ne sont pas toujours capturées par Application Insights

Les erreurs « 502 Passerelle incorrecte » et « 503 Service indisponible » ne sont pas toujours capturées par Application Insights. Si seul le code JavaScript côté client est utilisé pour l’analyse, ce comportement est prévisible, car la réponse d’erreur est retournée avant la page contenant l’en-tête HTML avec l’extrait de code JavaScript d’analyse rendu. 

Si la réponse 502 ou 503 a été envoyée à partir d’un serveur sur lequel l’analyse côté serveur est activée, les erreurs sont collectées par le Kit de développement logiciel (SDK) Application Insights. 

Toutefois, dans certains cas, même lorsque l’analyse côté serveur est activée sur le serveur web d’une application, une erreur 502 ou 503 n’est pas capturée par Application Insights. De nombreux serveurs web modernes n’autorisent pas un client à communiquer directement, mais utilisent plutôt des solutions comme les proxys inverses pour transmettre des informations entre le client et les serveurs web frontaux. 

Dans ce scénario, une réponse 502 ou 503 peut être renvoyée à un client en raison d’un problème au niveau de la couche du proxy inverse et cela n’est pas capturé par Application Insights. Pour aider à détecter les problèmes au niveau de cette couche, vous devrez peut-être transférer les journaux de votre proxy inverse vers Log Analytics et créer une règle personnalisée pour vérifier les réponses 502/503. Pour en savoir plus sur les causes courantes des erreurs 502 et 503, consultez [l’article de résolution des problèmes d’Azure App Service pour les erreurs « 502 Passerelle incorrecte » et « 503 Service indisponible »](../app-service/troubleshoot-http-502-http-503.md).     


## <a name="opentelemetry"></a>OpenTelemetry

### <a name="what-is-opentelemetry"></a>Qu’est-ce qu’OpenTelemetry ?

Une nouvelle norme open source pour l’observabilité. Pour en savoir plus, rendez-vous sur [https://opentelemetry.io/](https://opentelemetry.io/).

### <a name="why-is-microsoft--azure-monitor-investing-in-opentelemetry"></a>Pourquoi Microsoft/Azure Monitor investit-il dans OpenTelemetry ?

Nous pensons qu’elle répond mieux aux besoins de nos clients pour trois raisons :
   1. Prise en charge d’un plus grand nombre de scénarios client.
   2. Instrument sans crainte d’un verrouillage du fournisseur.
   3. Plus de transparence et d’engagement pour les clients.

Elle s’aligne également sur la stratégie de Microsoft visant à [adopter l’open source](https://opensource.microsoft.com/).

### <a name="what-additional-value-does-opentelemetry-give-me"></a>Quelle valeur ajoutée m’apporte OpenTelemetry ?

Outre les raisons ci-dessus, OpenTelemetry est plus efficace à grande échelle et fournit une conception et des configurations cohérentes dans les différentes langues.

### <a name="how-can-i-test-out-opentelemetry"></a>Comment puis-je tester OpenTelemetry ?

Inscrivez-vous à l’adresse [https://aka.ms/AzMonOtel](https://aka.ms/AzMonOtel) pour rejoindre notre communauté d’utilisateurs précoces d’Azure Monitor Application Insights.

### <a name="what-does-ga-mean-in-the-context-of-opentelemetry"></a>Que signifie GA dans le contexte d’OpenTelemetry ?

La communauté OpenTelemetry définit la disponibilité générale (GA) [ici](https://medium.com/opentelemetry/ga-planning-f0f6d7b5302). Toutefois, la « disponibilité générale » d’OpenTelemetry ne signifie pas la parité des fonctionnalités avec les Kits de développement logiciel (SDK) Application Insights existants. Azure Monitor continuera à recommander nos Kits de développement logiciel (SDK) Application Insights actuels pour les clients nécessitant des fonctionnalités telles que les [métriques pré-agrégées](app/pre-aggregated-metrics-log-metrics.md#pre-aggregated-metrics), les [métriques en temps réel](app/live-stream.md), l’[échantillonnage adaptatif](app/sampling.md#adaptive-sampling), le [profileur](app/profiler-overview.md) et le [débogueur de capture instantanée](app/snapshot-debugger.md) jusqu’à ce que les SDK OpenTelemetry atteignent la maturité des fonctionnalités.

### <a name="can-i-use-preview-builds-in-production-environments"></a>Puis-je utiliser des builds en préversion dans des environnements de production ?

Cela n’est pas recommandé. Pour plus d’informations, consultez [Conditions d’utilisation supplémentaires des Préversions Microsoft Azure](https://azure.microsoft.com/support/legal/preview-supplemental-terms/).

### <a name="whats-the-difference-between-opentelemetry-sdk-and-auto-instrumentation"></a>Quelle est la différence entre le Kit de développement logiciel (SDK) OpenTelemetry et l’auto-instrumentation ?

La spécification OpenTelemetry définit le [SDK](https://github.com/open-telemetry/opentelemetry-specification/blob/master/specification/glossary.md#telemetry-sdk). En bref, le « SDK » est un package spécifique au langage qui collecte les données de télémétrie sur les différents composants de votre application et envoie les données à Azure Monitor via un exportateur.

Le concept d’auto-instrumentation (parfois appelé injection de bytecode, sans code ou basé sur un agent) fait référence à la capacité d’instrumenter votre application sans modifier votre code. Par exemple, consultez le [fichier Lisez-moi de l’auto-instrumentation OpenTelemetry pour Java ](https://github.com/open-telemetry/opentelemetry-java-instrumentation/blob/master/README.md) pour plus d’informations.

### <a name="whats-the-opentelemetry-collector"></a>Qu’est-ce que le collecteur OpenTelemetry ?

Le collecteur OpenTelemetry est décrit dans son [fichier Lisez-moi GitHub](https://github.com/open-telemetry/opentelemetry-collector#opentelemetry-collector). Actuellement, Microsoft n’utilise pas le collecteur OpenTelemetry et dépend des exportateurs directs pour envoyer des données à Application Insights d’Azure Monitor.

### <a name="whats-the-difference-between-opencensus-and-opentelemetry"></a>Quelle est la différence entre OpenCensus et OpenTelemetry ?

[OpenCensus](https://opencensus.io/) est le précurseur d’[OpenTelemetry](https://opentelemetry.io/). Microsoft a contribué à réunir [OpenTracing](https://opentracing.io/) et OpenCensus pour créer OpenTelemetry, une norme d’observabilité unique et disponible à l’échelle mondiale. Le [Kit de développement logiciel (SDK) Python recommandé pour la production](app/opencensus-python.md) d’Azure Monitor est basé sur OpenCensus, mais, à terme, tous les SDK d’Azure Monitor seront basés sur OpenTelemetry.


## <a name="azure-monitor-for-containers"></a>Azure Monitor pour des conteneurs

### <a name="what-does-other-processes-represent-under-the-node-view"></a>Que représente *Autres processus* sous la vue Nœud ?

Les **autres processus** sont destinés à vous aider à comprendre clairement la cause racine de l’utilisation intensive des ressources sur votre nœud. Cela vous permet de distinguer l’utilisation des processus conteneurisés et non conteneurisés.

Quels sont ces **autres processus** ? 

Il s’agit de processus non conteneurisés qui s’exécutent sur votre nœud.  

Comment les calculer ?

**Autres processus** = *Utilisation totale par CAdvisor* - *Utilisation par un processus conteneurisé*

Les **autres processus** sont notamment :

- Processus non conteneurisés auto-managés ou Kubernetes managés 

- Processus à l’exécution du conteneur  

- Kubelet  

- Processus système en cours d’exécution sur votre nœud 

- Autres charges de travail non-Kubernetes en cours d’exécution sur le matériel ou la machine virtuelle du nœud 

### <a name="i-dont-see-image-and-name-property-values-populated-when-i-query-the-containerlog-table"></a>Je ne vois pas les valeurs de propriété Image et Name renseignées quand j’interroge la table ContainerLog.

Pour la version de l’agent ciprod12042019 et les versions ultérieures, par défaut, ces deux propriétés ne sont pas remplies pour chaque ligne de journal afin de réduire les coûts liés aux données de journal collectées. Il existe deux options pour interroger la table qui inclut ces propriétés avec leurs valeurs :

#### <a name="option-1"></a>Option 1 : 

Joignez d’autres tables pour inclure ces valeurs de propriétés dans les résultats.

Modifiez vos requêtes pour inclure les propriétés Image et ImageTag à partir de la table ```ContainerInventory``` en joignant la propriété ContainerID. Vous pouvez inclure la propriété Name (telle qu’elle apparaissait dans la table ```ContainerLog```) à partir du champ ContaineName de la table KubepodInventory en effectuant une jointure sur la propriété ContainerID. C'est l'option recommandée.

L’exemple suivant illustre une requête détaillée qui explique comment obtenir ces valeurs de champs avec des jointures.

```
//lets say we are querying an hour worth of logs
let startTime = ago(1h);
let endTime = now();
//below gets the latest Image & ImageTag for every containerID, during the time window
let ContainerInv = ContainerInventory | where TimeGenerated >= startTime and TimeGenerated < endTime | summarize arg_max(TimeGenerated, *)  by ContainerID, Image, ImageTag | project-away TimeGenerated | project ContainerID1=ContainerID, Image1=Image ,ImageTag1=ImageTag;
//below gets the latest Name for every containerID, during the time window
let KubePodInv  = KubePodInventory | where ContainerID != "" | where TimeGenerated >= startTime | where TimeGenerated < endTime | summarize arg_max(TimeGenerated, *)  by ContainerID2 = ContainerID, Name1=ContainerName | project ContainerID2 , Name1;
//now join the above 2 to get a 'jointed table' that has name, image & imagetag. Outer left is safer in-case there are no kubepod records are if they are latent
let ContainerData = ContainerInv | join kind=leftouter (KubePodInv) on $left.ContainerID1 == $right.ContainerID2;
//now join ContainerLog table with the 'jointed table' above and project-away redundant fields/columns and rename columns that were re-written
//Outer left is safer so you dont lose logs even if we cannot find container metadata for loglines (due to latency, time skew between data types etc...)
ContainerLog
| where TimeGenerated >= startTime and TimeGenerated < endTime 
| join kind= leftouter (
   ContainerData
) on $left.ContainerID == $right.ContainerID2 | project-away ContainerID1, ContainerID2, Name, Image, ImageTag | project-rename Name = Name1, Image=Image1, ImageTag=ImageTag1 

```

#### <a name="option-2"></a>Option 2 :

Réactivez la collecte pour ces propriétés pour chaque ligne de journal de conteneur.

Si la première option n’est pas pratique en raison des modifications de requête qu’elle implique, vous pouvez réactiver la collecte de ces champs en activant le paramètre ```log_collection_settings.enrich_container_logs``` dans le mappage de configuration de l’agent, comme décrit dans les [paramètres de configuration de la collecte de données](insights/container-insights-agent-config.md).

> [!NOTE]
> La seconde option n’est pas recommandée avec les grands clusters qui contiennent plus de 50 nœuds, car elle génère des appels de serveur d’API à partir de chaque nœud du cluster pour effectuer cet enrichissement. En outre, cette option augmente la taille des données pour chaque ligne de journal collectée.

### <a name="can-i-view-metrics-collected-in-grafana"></a>Puis-je consulter les métriques collectées dans Grafana ?

Azure Monitor pour conteneurs prend en charge la consultation des métriques stockées dans votre espace de travail Log Analytics, dans les tableaux de bord Grafana. Nous avons fourni un modèle que vous pouvez télécharger à partir du [référentiel des tableaux de bord](https://grafana.com/grafana/dashboards?dataSource=grafana-azure-monitor-datasource&category=docker) de Grafana pour vous aider à démarrer, et une référence permettant de vous familiariser avec l’interrogation des données supplémentaires depuis vos clusters supervisés, pour une visualisation dans des tableaux de bord Grafana personnalisés. 

### <a name="can-i-monitor-my-aks-engine-cluster-with-azure-monitor-for-containers"></a>Puis-je surveiller mon cluster AKS-Engine avec Azure Monitor pour les conteneurs ?

Azure Monitor pour conteneurs prend en charge la surveillance des charges de travail de conteneur déployées sur un ou plusieurs clusters AKS-Engine (anciennement ACS-Engine) hébergés sur Azure. Pour plus d’informations et une vue d’ensemble des étapes requises pour l’activation de la surveillance dans le cadre de ce scénario, voir [Utilisation d’Azure Monitor pour conteneurs pour AKS-Engine ](https://github.com/microsoft/OMS-docker/tree/aks-engine).

### <a name="why-dont-i-see-data-in-my-log-analytics-workspace"></a>Pourquoi est-ce que je ne vois pas de données dans mon espace de travail Log Analytics ?

Si vous ne voyez pas de données dans l’espace de travail Log Analytics à un moment donné de la journée, il est possible que vous ayez atteint la limite de 500 Mo par défaut ou la limite quotidienne spécifiée pour contrôler la quantité de données à collecter chaque jour. Lorsque la limite est atteinte pour la journée, la collecte de données s’interrompt et ne reprend que sur le jour suivant. Pour vérifier votre utilisation des données et mettre à jour vers un autre niveau tarifaire en fonction de vos modèles d’utilisation anticipée, consultez [Log data usage and cost](platform/manage-cost-storage.md) (Utilisation et coût de données de journal). 

### <a name="what-are-the-container-states-specified-in-the-containerinventory-table"></a>Quels sont les états des conteneurs spécifiés dans la table ContainerInventory ?

La table ContainerInventory contient des informations sur les conteneurs arrêtés et en cours d'exécution. La table est renseignée par un flux de travail situé au sein de l’agent qui interroge le docker pour tous les conteneurs (en cours d’exécution et arrêtés) et transmet ces données à l’espace de travail Log Analytics.
 
### <a name="how-do-i-resolve-missing-subscription-registration-error"></a>Comment puis-je résoudre l’erreur *Missing Subscription registration* (Inscription d’abonnement manquante) ?

Si vous recevez l’erreur **Missing Subscription registration** (Inscription d’abonnement manquante), vous pouvez y remédier en inscrivant le fournisseur de ressources **Microsoft.OperationsManagement** dans l’abonnement où l’espace de travail est défini. La documentation contenant la procédure à suivre est disponible [ici](../azure-resource-manager/templates/error-register-resource-provider.md).

### <a name="is-there-support-for-kubernetes-rbac-enabled-aks-clusters"></a>Existe-t-il une prise en charge pour les clusters AKS avec Kubernetes RBAC activé ?

La solution Container Monitoring ne prend pas en charge Kubernetes RBAC, mais il est pris en charge avec Azure Monitor pour conteneurs. La page des détails de la solution peut ne pas afficher les bonnes informations sur les panneaux qui présentent les données de ces clusters.

### <a name="how-do-i-enable-log-collection-for-containers-in-the-kube-system-namespace-through-helm"></a>Comment puis-je activer la collecte des journaux pour les conteneurs de l'espace de noms kube-system via Helm ?

Par défaut, la collecte des journaux des conteneurs de l'espace de noms kube-system est désactivée. La collecte des journaux peut être activée en définissant une variable d'environnement sur l'agent omsagent. Pour plus d’informations, consultez la page GitHub [Azure Monitor pour conteneurs](https://aka.ms/azuremonitor-containers-helm-chart). 

### <a name="how-do-i-update-the-omsagent-to-the-latest-released-version"></a>Comment mettre à jour l'agent omsagent vers la dernière version disponible ?

Pour savoir comment procéder à la mise à niveau de l'agent, consultez [Gestion de l'agent](insights/container-insights-manage-agent.md).

### <a name="how-do-i-enable-multi-line-logging"></a>Comment puis-je activer la journalisation multiligne?

Actuellement, Azure Monitor pour conteneurs ne prend pas en charge la journalisation multiligne, mais il existe des solutions de contournement. Vous pouvez configurer tous les services sur le format JSON. Docker/Moby utilisera ensuite une seule ligne pour l'écriture.

Par exemple, vous pouvez inclure votre journal dans un objet JSON, comme indiqué dans l'exemple d'application node.js ci-dessous :

```
console.log(json.stringify({ 
      "Hello": "This example has multiple lines:",
      "Docker/Moby": "will not break this into multiple lines",
      "and you will receive":"all of them in log analytics",
      "as one": "log entry"
      }));
```

Lorsque vous interrogerez les données dans Azure Monitor pour les journaux d’activité, celles-ci se présenteront comme dans l’exemple suivant :

```
LogEntry : ({"Hello": "This example has multiple lines:","Docker/Moby": "will not break this into multiple lines", "and you will receive":"all of them in log analytics", "as one": "log entry"}

```

Pour en savoir plus sur ce problème, utilisez le [lien GitHub](https://github.com/moby/moby/issues/22920) suivant.

### <a name="how-do-i-resolve-azure-ad-errors-when-i-enable-live-logs"></a>Comment puis-je résoudre les erreurs Azure AD lorsque j’active les journaux d’activité dynamiques ? 

L'erreur suivante peut apparaître : **L’URL de réponse spécifiée dans la requête ne correspond pas aux URL de réponse configurées pour l’application : « <ID application\> »** . Sa solution se trouve dans l'article [Affichage des données de conteneur en temps réel avec Azure Monitor pour conteneurs](insights/container-insights-livedata-setup.md#configure-ad-integrated-authentication). 

### <a name="why-cant-i-upgrade-cluster-after-onboarding"></a>Pourquoi ne puis-je pas mettre à niveau un cluster après l’intégration ?

Après avoir activé Azure Monitor pour conteneurs pour un cluster AKS, si vous supprimez l’espace de travail Log Analytics auquel le cluster envoyait ses données, le cluster échoue lorsque vous tentez de le mettre à niveau. Pour contourner ce problème, vous devez désactiver la supervision, puis la réactiver en faisant référence à un autre espace de travail valide dans votre abonnement. Lorsque vous réessayez de mettre le cluster à niveau, il doit être traité et s’effectuer correctement.  

### <a name="which-ports-and-domains-do-i-need-to-openallow-for-the-agent"></a>Quels ports et domaines dois-je ouvrir/autoriser pour l’agent ?

Pour plus d'informations sur les configurations de proxy et de pare-feu requises pour l'agent conteneurisé avec les clouds Azure, Azure US Government et Azure China 21Vianet, consultez [Configuration requise pour le pare-feu réseau](insights/container-insights-onboard.md#network-firewall-requirements).


## <a name="azure-monitor-for-vms"></a>Azure Monitor pour machines virtuelles

### <a name="can-i-onboard-to-an-existing-workspace"></a>Puis-je intégrer un espace de travail existant ?
Si vos machines virtuelles sont déjà connectées à un espace de travail Log Analytics, vous pouvez continuer à l’utiliser lors de l’intégration à Azure Monitor pour machines virtuelles, à condition qu’il se trouve dans l’une des [régions prises en charge](insights/vminsights-configure-workspace.md#supported-regions).


### <a name="can-i-onboard-to-a-new-workspace"></a>Puis-je intégrer un nouvel espace de travail ? 
Si vos machines virtuelles ne sont pas actuellement connectées à un espace de travail Log Analytics existant, vous devez créer un nouvel espace de travail pour stocker vos données. La création d’un nouvel espace de travail par défaut se fait automatiquement lorsque vous configurez une seule machine virtuelle Azure pour Azure Monitor pour machines virtuelles via le portail Azure.

Si vous choisissez d’utiliser la méthode basée sur un script, ces étapes sont décrites dans l’article [Activer Azure Monitor pour machines virtuelles à l’aide d’Azure PowerShell ou d’un modèle Resource Manager](./insights/vminsights-enable-powershell.md). 

### <a name="what-do-i-do-if-my-vm-is-already-reporting-to-an-existing-workspace"></a>Que dois-je faire si ma machine virtuelle envoie déjà des rapports à un espace de travail existant ?
Si vous collectez déjà des données à partir de vos machines virtuelles, il se peut que vous l’ayez déjà configuré de façon à ce que les rapports des données soient transmis à un espace de travail Log Analytics existant.  Si cet espace de travail se trouve dans l’une des régions que nous prenons en charge, vous pouvez y activer Azure Monitor pour machines virtuelles.  Si l’espace de travail que vous utilisez déjà ne se trouve pas dans l’une des régions que nous prenons en charge, vous ne pouvez pas intégrer Azure Monitor pour machines virtuelles pour l’instant.  Nous mettons tout en œuvre pour prendre en charge de nouvelles régions.


### <a name="why-did-my-vm-fail-to-onboard"></a>Pourquoi l’intégration de ma machine virtuelle a-t-elle échoué ?
Voici les étapes effectuées lors de l’intégration d’une machine virtuelle Azure depuis le portail Azure :

* Un espace de travail Log Analytics par défaut est créé, si cette option a été sélectionnée.
* L’agent Log Analytics est installé sur les machines virtuelles Azure à l’aide d’une extension machine virtuelle, au besoin.  
* L’agent Map Dependency d’Azure Monitor pour machines virtuelles est installé sur les machines virtuelles Azure à l’aide d’une extension, au besoin. 

Pendant le processus d’intégration, l’état d’avancement de chacun des éléments ci-dessus est vérifié afin de vous renvoyer un état de notification sur le portail. La configuration de l’espace de travail et l’installation de l’agent prennent généralement de 5 à 10 minutes. La consultation des données de supervision dans le portail prend 5 à 10 minutes supplémentaires.  

Si vous avez commencé l’intégration et que des messages indiquant que la machine virtuelle doit être intégrée s’affichent, attendez jusqu’à 30 minutes pour que la machine virtuelle puisse terminer le processus. 


### <a name="i-dont-see-some-or-any-data-in-the-performance-charts-for-my-vm"></a>Je ne vois pas de données dans les graphiques de performances de ma machine virtuelle ou certaines sont manquantes
Nos graphiques de performances ont été mis à jour pour utiliser les données stockées dans la table *InsightsMetrics*.  Pour afficher les données de ces graphiques, vous devez effectuer une mise à niveau pour utiliser la nouvelle solution VM Insights.  Pour plus d’informations, reportez-vous à notre [FAQ sur la disponibilité générale](insights/vminsights-ga-release-faq.md).

Si vous ne voyez pas les données de performances dans le tableau du disque ou dans certains graphiques de performances, vos compteurs de performances ne sont peut-être pas configurés dans l’espace de travail. Pour résoudre ce problème, exécutez le [script PowerShell](./insights/vminsights-enable-powershell.md) suivant.


### <a name="how-is-azure-monitor-for-vms-map-feature-different-from-service-map"></a>Quelle est la différence entre la fonctionnalité de mappage d’Azure Monitor pour machines virtuelles et celle de Service Map ?
La fonctionnalité de mappage d’Azure Monitor pour machines virtuelles est basée sur celle de Service Map, mais présente les différences suivantes :

* La fenêtre Mappage (Map) est accessible depuis le panneau de la machine virtuelle et depuis Azure Monitor pour machines virtuelles sous Azure Monitor.
* Il est maintenant possible de cliquer sur les connexions dans la fenêtre Mappage pour afficher les données métriques de connexion dans le panneau latéral de la connexion sélectionnée.
* Une nouvelle API permet de créer les mappages afin de mieux prendre en charge les plus complexes d’entre eux.
* Les machines virtuelles supervisées font maintenant partie du nœud du groupe de clients, et le graphique en secteurs montre la répartition entre les machines virtuelles supervisées et non supervisées dans ce groupe.  Il permet également de filtrer la liste des machines lorsque le groupe est développé.
* Les machines virtuelles supervisées font désormais partie des nœuds du groupe de ports du serveur, et le graphique en secteurs montre la répartition entre les machines supervisées et non supervisées dans ce groupe.  Il permet également de filtrer la liste des machines lorsque le groupe est développé.
* Le style de mappage a été mis à jour pour assurer une meilleure cohérence avec le mappage des applications (App Map) dans les insights des applications (Application insights).
* Les panneaux latéraux ont été mis à jour, mais ne disposent pas de l’ensemble d’intégrations complet pris en charge dans Service Map : Update Management, Change Tracking, Security et Service Desk. 
* L’option de sélection des groupes et des machines à mapper a été mise à jour et prend désormais en charge les abonnements, les groupes de ressources, les groupes de machines virtuelles identiques Azure et les services cloud.
* Vous ne pouvez pas créer de nouveaux groupes de machines Service Map dans la fonctionnalité de mappage d’Azure Monitor pour machines virtuelles.  

### <a name="why-do-my-performance-charts-show-dotted-lines"></a>Pourquoi des lignes pointillées apparaissent sur mes graphiques de performances ?
Cela peut se produire pour plusieurs raisons.  Lorsqu’il manque des données, les lignes sont marquées en pointillés.  Si vous avez modifié la fréquence d’échantillonnage des données sur les compteurs de performance activés (le réglage par défaut correspond à une collecte de données toutes les 60 secondes), des lignes en pointillés peuvent apparaître sur le graphique si vous choisissez une plage horaire courte et une fréquence inférieure à celle utilisée sur le graphique (par exemple, la fréquence d’échantillonnage est de 10 minutes et chacun de ses compartiments correspond à 5 minutes).  Si vous choisissez un plus grand intervalle de temps pour la visualisation, les lignes du graphique devraient apparaître sous forme de lignes pleines plutôt que de points dans ce cas.

### <a name="are-groups-supported-with-azure-monitor-for-vms"></a>Les groupes sont-ils pris en charge avec Azure Monitor pour machines virtuelles ?
Oui, une fois que vous installez Dependency Agent, nous collectons des informations depuis les machines virtuelles pour afficher les groupes en fonction de l’abonnement, du groupe de ressources, des groupes de machines virtuelles identiques et des services cloud.  Si vous utilisez Service Map et que vous avez créé des groupes d’ordinateurs, ceux-ci sont également affichés.  Les groupes d’ordinateurs apparaissent également dans le filtre de groupes si vous les avez créés pour l’espace de travail que vous affichez. 

### <a name="how-do-i-see-the-details-for-what-is-driving-the-95th-percentile-line-in-the-aggregate-performance-charts"></a>Comment faire pour voir les détails de ce qui détermine la ligne du 95e percentile dans les graphiques de performance globale ?
Par défaut, la liste est triée de façon à afficher les machines virtuelles ayant la valeur la plus élevée pour le 95e percentile de la métrique sélectionnée, à l’exception du graphique de la mémoire disponible qui affiche les machines ayant la valeur la plus faible pour le 5e percentile.  Si vous cliquez sur le graphique, la fenêtre **Top N List** s’ouvre avec la métrique appropriée sélectionnée.

### <a name="how-does-the-map-feature-handle-duplicate-ips-across-different-vnets-and-subnets"></a>Comment la fonctionnalité Map (Mappage) gère-t-elle les adresses IP dupliquées sur différents réseaux et sous-réseaux ?
Si vous dupliquez des plages d’adresses IP avec des machines virtuelles ou des groupes d’échelles de machines virtuelles identiques Azure sur des sous-réseaux et des réseaux virtuels, il se peut que la mappage d’Azure Monitor pour machines virtuelles affiche des informations erronées à la place. Il s’agit d’un problème connu et nous étudions des options pour offrir une meilleure expérience.

### <a name="does-map-feature-support-ipv6"></a>La fonctionnalité de mappage prend-elle en charge le protocole IPv6 ?
La fonctionnalité de mappage ne prend actuellement en charge que le protocole IPv4, mais nous étudions actuellement la prise en charge du protocole IPv6. Nous prenons également en charge le protocole IPv4 tunnelisé dans le protocole IPv6.

### <a name="when-i-load-a-map-for-a-resource-group-or-other-large-group-the-map-is-difficult-to-view"></a>Lorsque je charge un mappage pour un groupe de ressources ou un autre grand groupe, celui-ci est difficile à visualiser
Bien que nous ayons apporté des améliorations au mappage pour la gestion de configurations grandes et complexes, nous savons qu’il peut comporter un grand nombre de nœuds, de connexions et de nœuds fonctionnant en cluster.  Nous continuons d’accroître notre soutien afin de renforcer la mise à l’échelle.   

### <a name="why-does-the-network-chart-on-the-performance-tab-look-different-than-the-network-chart-on-the-azure-vm-overview-page"></a>Pourquoi le graphique du réseau dans l’onglet Performance est-il différent de celui de la page d’aperçu de la machine virtuelle Azure ?

La page d’aperçu d’une machine virtuelle Azure affiche des graphiques basés sur la mesure de l’activité de l’hôte dans la machine virtuelle hôte.  Dans le cas du graphique du réseau sur l’aperçu de la machine virtuelle Azure, seul le trafic réseau qui sera facturé est affiché.  Cela n'inclut pas le trafic entre réseaux virtuels.  Les données et les graphiques présentés dans Azure Monitor pour machines virtuelles sont basés sur les données de la machine virtuelle invitée et le graphique du réseau affiche tout le trafic TCP/IP entrant et sortant vers cette machine virtuelle, y compris le trafic entre réseaux virtuels.

### <a name="how-is-response-time-measured-for-data-stored-in-vmconnection-and-displayed-in-the-connection-panel-and-workbooks"></a>Comment le temps de réponse des données stockées dans VMConnection est -il mesuré et affiché dans le volet de connexion et dans les classeurs ?

Ce temps de réponse est une approximation. Comme nous n’instrumentons pas le code de l’application, nous ne savons pas réellement à quel moment une requête débute et quand la réponse nous parvient. Au lieu de cela, nous observons les données envoyées sur une connexion, puis celles qui sont renvoyées sur cette connexion. Notre agent effectue le suivi de ces envois et réceptions et tente de les associer : une séquence d’envois suivie d’une séquence de réceptions est interprétée comme une paire requête/réponse. Le délai entre ces opérations correspond au temps de réponse. Il inclut la latence du réseau et le temps de traitement par le serveur.

Cette approximation fonctionne bien pour les protocoles basés sur les paires requête/réponse : une seule requête est émise sur la connexion, et une seule réponse nous parvient. Cela se produit pour le protocole HTTP(S) (sans traitement « pipeline »), mais non pour les autres.

### <a name="are-there-limitations-if-i-am-on-the-log-analytics-free-pricing-plan"></a>Existe-t-il des limites avec l’utilisation du plan tarifaire Gratuit de Log Analytics ?
Si vous avez configuré Azure Monitor avec un espace de travail Log Analytics en utilisant le niveau tarifaire *Gratuit*, la fonctionnalité de mappage d’Azure Monitor pour machines virtuelles ne prend en charge que cinq machines connectées à l’espace de travail. Ainsi, si cinq machines virtuelles sont connectées à un espace de travail gratuit et que vous déconnectez l'une d'elle, puis connectez une nouvelle machine virtuelle, la nouvelle machine virtuelle n'est ni surveillée ni reflétée sur la page dédiée aux mappages.  

Dans ce cas, l’option **Essayer maintenant** s’affiche quand vous ouvrez la machine virtuelle et sélectionnez **Insights** dans le volet de gauche, même après son installation sur la machine virtuelle.  Mais aucune option ne vous est proposée, contrairement à ce qui aurait eu lieu si cette machine virtuelle n'avait pas été intégrée à Azure Monitor pour machines virtuelles. 


## <a name="next-steps"></a>Étapes suivantes
Si vous ne trouvez pas de réponse à votre question ici, vous pouvez consulter les forums suivants pour plus de questions et réponses.

- [Log Analytics](/answers/topics/azure-monitor.html)
- [Application Insights](/answers/topics/azure-monitor.html)

Pour obtenir des commentaires généraux sur Azure Monitor visitez le [forum de commentaires](https://feedback.azure.com/forums/34192--general-feedback).