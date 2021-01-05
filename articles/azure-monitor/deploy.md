---
title: Déployer Azure Monitor
description: Décrit les différentes étapes requises pour une implémentation complète d’Azure Monitor afin de surveiller toutes les ressources de votre abonnement Azure.
ms.subservice: ''
ms.topic: conceptual
author: bwren
ms.author: bwren
ms.date: 07/27/2020
ms.openlocfilehash: a0591825bf187648293d5aabc88597b19ab2436e
ms.sourcegitcommit: b4880683d23f5c91e9901eac22ea31f50a0f116f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/11/2020
ms.locfileid: "94491050"
---
# <a name="deploy-azure-monitor"></a>Déployer Azure Monitor
L’activation d’Azure Monitor pour surveiller toutes vos ressources Azure consiste en une combinaison de configuration de composants Azure Monitor et de configuration des ressources Azure pour générer des données d’analyse collectées par Azure Monitor. Cet article décrit les différentes étapes requises pour une implémentation complète d’Azure Monitor avec une configuration commune afin de surveiller toutes les ressources de votre abonnement Azure. Les descriptions de base de chaque étape sont fournies avec des liens vers d’autres documents pour des exigences de configuration détaillées.

> [!IMPORTANT]
> Les fonctionnalités d’Azure Monitor et leur configuration varient selon les besoins de votre entreprise, en fonction du coût des fonctionnalités activées. Chaque étape ci-dessous indiquera s’il existe des coûts potentiels et vous devez évaluer ces coûts avant de poursuivre cette étape. Pour obtenir des détails sur la tarification, consultez [Tarification Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/).

## <a name="configuration-goals"></a>Objectifs de la configuration
L’objectif d’une implémentation complète d’Azure Monitor consiste à collecter toutes les données disponibles à partir de toutes vos ressources et applications Cloud et à activer autant de fonctionnalités que possible dans Azure Monitor, en fonction de ces données.

Les données collectées par Azure Monitor sont envoyées aux [Métriques Azure Monitor](platform/data-platform-metrics.md) ou aux [Journaux Azure Monitor](platform/data-platform-logs.md). Chacun stocke différents genres de données et permet différents types d’analyses et d’alertes. Pour obtenir une description des différents types d’alerte, consultez [Comparer les métriques et les journaux Azure Monitor](platform/data-platform.md) pour obtenir une comparaison des deux et [Vue d’ensemble des alertes dans Microsoft Azure](platform/alerts-overview.md) pour une description des différents types d’alertes. 

Certaines données peuvent être envoyées aux métriques et aux journaux afin de pouvoir les utiliser à l’aide de différentes fonctionnalités. Dans ce cas, vous devrez peut-être les configurer séparément. Par exemple, les données de métriques sont automatiquement envoyées par les ressources Azure aux métriques, qui prennent en charge les alertes métriques et de l’explorateur de mesures. Vous devez créer un paramètre de diagnostic pour chaque ressource afin d’envoyer les mêmes données de mesure aux journaux, ce qui vous permet d’analyser les tendances des performances avec d’autres données de journal à l’aide de Log Analytics. Les sections ci-dessous identifient où les données sont envoyées et incluent chaque étape nécessaire pour envoyer des données à tous les emplacements possibles.

Vous pouvez avoir des exigences supplémentaires telles que la surveillance des ressources en dehors d’Azure et l’envoi de données en dehors d’Azure Monitor. Des exigences telles que celles-ci peuvent être obtenues avec une configuration supplémentaire des fonctionnalités décrites dans cet article. Suivez les liens vers la documentation de chaque étape pour obtenir des options de configuration supplémentaires.

## <a name="collect-data-from-azure-resources"></a>Recueillir des données à partir de ressources Azure

> [!NOTE]
> Pour obtenir un guide complet sur la surveillance des machines virtuelles avec Azure Monitor, consultez [Surveillance des ressources Azure avec Azure Monitor](insights/monitor-azure-resource.md).

Une analyse des ressources Azure est disponible automatiquement sans aucune configuration requise, tandis que vous devez effectuer des étapes de configuration pour collecter des données d’analyse supplémentaires. Le tableau suivant illustre les étapes de configuration requises pour collecter toutes les données disponibles à partir de vos ressources Azure, notamment les données d’étape envoyées aux métriques Azure Monitor et aux journaux Azure Monitor. Chaque étape est décrite en détail dans les sections ci-dessous.

[ ![Déployer une surveillance des ressources Azure](media/deploy/deploy-azure-resources.png) ](media/deploy/deploy-azure-resources.png#lightbox)

### <a name="no-configuration"></a>Pas de configuration
Les fonctionnalités suivantes d’Azure Monitor sont activées sans configuration requise lorsque vous créez un abonnement Azure. Aucun coût n’est associé à cette surveillance.

[Journaux Azure Active Directory](../active-directory/reports-monitoring/overview-reports.md) - Fournit un historique des activités de connexion au niveau du locataire et une piste d'audit des modifications effectuées dans Azure Active Directory. Pour plus d’informations sur les journaux d’Azure Active Directory et sur la façon de les afficher dans le portail, consultez [Rapports d’activité d’audit dans le portail Azure Active Directory](../active-directory/reports-monitoring/concept-audit-logs.md) et [Rapports d’activité de connexion dans le portail Azure Active Directory](../active-directory/reports-monitoring/concept-sign-ins.md).

[Journal d’activité](platform/platform-logs-overview.md) - Apporte des insights sur la gestion de groupe les événements liés aux abonnements qui se sont produits dans Azure. Les événements sont automatiquement écrits dans le journal d’activité lorsque vous créez une nouvelle ressource Azure, modifiez une ressource ou effectuez une activité importante. Vous pouvez afficher les événements dans le Portail Azure et créer des alertes de journal d’activité lorsque des événements particuliers sont créés. Pour plus d’informations sur le journal d’activité et sur son affichage dans le Portail Azure, consultez [Journal d’activité Azure](platform/activity-log.md).

[Métriques de plateforme](platform/metrics-supported.md) - Collectées automatiquement dans les [Métriques Azure Monitor](platform/data-platform-metrics.md). Ces données sont souvent présentées sur la page **Vue d’ensemble** du Portail Azure pour différents services. Pour plus d’informations sur l’analyse des mesures de plate-forme dans le Portail Azure, consultez [Prise en main d’Azure Metrics Explorer](platform/metrics-getting-started.md). 


### <a name="create-log-analytics-workspace"></a>Créer un espace de travail Log Analytics
Vous avez besoin d’au moins un espace de travail Log Analytics pour activer les [Journaux Azure Monitor](platform/data-platform-logs.md), requis pour la collecte de ces données sous forme de journaux à partir de ressources Azure, la collecte de données à partir du système d’exploitation invité des machines virtuelles Azure et la plupart des Azure Monitor Insights. D’autres services tels que Azure Sentinel et Azure Security Center utilisent également un espace de travail Log Analytics et peuvent partager le même que celui utilisé pour Azure Monitor. Vous pouvez commencer avec un seul espace de travail pour prendre en charge cette analyse, mais consultez [Conception de votre déploiement de journaux Azure Monitor](platform/design-logs-deployment.md) pour obtenir des conseils sur l’utilisation de plusieurs espaces de travail.

Il n’y a aucun coût pour créer un espace de travail Log Analytics, mais il y a des frais potentiels une fois que vous avez configuré les données à collecter dans celui-ci. Pour plus d'informations, consultez [Gérer l’utilisation et les coûts avec les journaux Azure Monitor](platform/manage-cost-storage.md).  

Pour créer un espace de travail Log Analytics, consultez [Créer un espace de travail Log Analytics dans le portail Azure](learn/quick-create-workspace.md). Voir [Gérer l’accès aux données de journal et aux espaces de travail dans Azure Monitor](platform/manage-access.md) pour configurer l’accès. 

### <a name="create-diagnostic-setting-to-collect-tenant-and-subscription-logs"></a>Créer un paramètre de diagnostic pour collecter les journaux des locataires et des abonnements
Tandis que les journaux [Azure Active Directory](../active-directory/reports-monitoring/overview-reports.md) pour votre locataire et que le [journal d’activité](platform/platform-logs-overview.md) pour votre abonnement sont collectées automatiquement, leur envoi à un espace de travail Log Analytics vous permet d’analyser ces événements avec d’autres données de journal à l’aide des requêtes de journal dans Log Analytics. Cela vous permet également de créer des alertes de requête de journal qui sont le seul moyen d’alerter sur les journaux Azure Active Directory et de fournir une logique plus complexe que les alertes du journal d’activité.

Il n’y a aucun coût pour envoyer le journal d’activité à un espace de travail, mais il existe une ingestion de données et des frais de rétention pour les journaux Azure Active Directory. 

Consultez [Intégrer des journaux Azure AD avec des journaux Azure Monitor](../active-directory/reports-monitoring/howto-integrate-activity-logs-with-log-analytics.md) et [Créer des paramètres de diagnostic pour envoyer des journaux et des métriques de plateforme vers différentes destinations](platform/diagnostic-settings.md) pour créer un paramètre de diagnostic pour votre locataire et votre abonnement afin d’envoyer des entrées de journal à votre espace de travail Log Analytics. 

### <a name="create-diagnostic-setting-to-collect-resource-logs-and-platform-metrics"></a>Créer un paramètre de diagnostic pour collecter les journaux de ressources et des métriques de plateforme
Les ressources dans Azure génèrent automatiquement des [journaux de ressources](platform/platform-logs-overview.md) qui fournissent des détails sur les opérations effectuées au sein de la ressource. Contrairement aux métriques de plateforme, vous devez cependant configurer les journaux de ressources à collecter. Créez un paramètre de diagnostic pour les envoyer à un espace de travail Log Analytics pour les combiner avec les autres données utilisées avec les journaux Azure Monitor. Le même paramètre de diagnostic peut être utilisé pour envoyer également les métriques de la plateforme pour la plupart des ressources vers le même espace de travail, ce qui vous permet d’analyser les données de métriques à l’aide de requêtes de journal avec d’autres données collectées.

Il y a un coût pour ce regroupement.par conséquent, reportez-vous à [Tarification Azure Monitor](https://azure.microsoft.com/pricing/details/monitor/) avant de l’implémenter sur un nombre important de ressources. Consultez également [Gérer l’utilisation et les coûts avec les journaux Azure Monitor](platform/manage-cost-storage.md) pour plus d’informations sur l’optimisation du coût de votre collection de journaux.

Pour créer des paramètres de diagnostic dans une ressource Azure, consultez [Créer un paramètre de diagnostic pour collecter les journaux et les métriques des ressources dans Azure](platform/diagnostic-settings.md#create-in-azure-portal). Étant donné qu’un paramètre de diagnostic doit être créé pour chaque ressource Azure, consultez [Déployer Azure Monitor à l’échelle](deploy-scale.md) pour plus d’informations sur l’utilisation de la [Stratégie Azure](../governance/policy/overview.md) afin de créer automatiquement des paramètres chaque fois qu’une ressource Azure est créée.

### <a name="enable-insights-and-solutions"></a>Activer les Insights et solutions
Les insights et solutions offrent une surveillance spécialisée pour un service ou une solution spécifique. Les Insights utilisent des fonctionnalités plus récentes d’Azure Monitor telles que des classeurs. Vous devez donc utiliser un aperçu s’il est disponible pour votre service. Ils sont automatiquement disponibles dans chaque abonnement Azure, mais ils peuvent nécessiter une certaine configuration pour bénéficier de toutes les fonctionnalités. Ils utilisent généralement les métriques de plateforme et les journaux de ressources que vous avez configurés précédemment et peuvent collecter des données supplémentaires.

Les solutions doivent être ajoutées à chaque abonnement et travailler exclusivement avec les données des journaux Azure Monitor et peuvent collecter des données de journal supplémentaires.

Il n’y a aucun coût pour les solutions ou les insights, mais vous pouvez être facturé pour toutes les données recueillies.

Voir [Qu’est-ce qui est surveillé par Azure Monitor ?](monitor-reference.md) pour obtenir la liste des analyses et des solutions disponibles dans Azure Monitor. Reportez-vous à la documentation de chacune pour obtenir des informations de configuration ou de tarification spécifiques. 

## <a name="collect-data-from-virtual-machines"></a>Collecter des données à partir de machines virtuelles

> [!NOTE]
> Pour obtenir un guide complet sur la surveillance des machines virtuelles avec Azure Monitor, consultez [Surveillance des ressources Azure avec Azure Monitor](insights/monitor-vm-azure.md). 

Les machines virtuelles génèrent des données similaires à celles des autres ressources Azure, mais vous avez besoin d’un agent pour collecter les données du système d’exploitation invité. Pour une comparaison détaillée des agents Azure Monitor, consultez [Vue d’ensemble des agents Azure Monitor](platform/agents-overview.md). 

[Azure Monitor pour machines virtuelles](insights/vminsights-overview.md) utilise l’agent de Log Analytics et l’agent de Dépendance pour collecter des données à partir du système d’exploitation invité des machines virtuelles. vous pouvez donc déployer ces agents dans le cadre de l’implémentation de cet aperçu. Cela active l’agent Log Analytics pour d’autres services qui l’utilisent, comme Azure Security Center.


[ ![Déployer une machine virtuelle Azure](media/deploy/deploy-azure-vm.png) ](media/deploy/deploy-azure-vm.png#lightbox)


### <a name="configure-workspace-for-azure-monitor-for-vms"></a>Configurer l’espace de travail d’Azure Monitor pour machines virtuelles
Azure Monitor pour machines virtuelles nécessite un espace de travail Log Analytics qui est généralement identique à celui créé pour collecter des données à partir d’autres ressources Azure. Avant d’activer des machines virtuelles, vous devez ajouter la solution requise pour Azure Monitor pour machines virtuelles à l’espace de travail.

Pour plus d’informations sur la configuration de votre espace de travail Log Analytics pour Azure Monitor pour machines virtuelles, consultez [Configurer l’espace de travail Log Analytics d’Azure Monitor pour machines virtuelles](insights/vminsights-configure-workspace.md).

### <a name="enable-azure-monitor-for-vms-on-each-virtual-machine"></a>Activer Azure Monitor pour machines virtuelles sur chaque machine virtuelle
Une fois qu’un espace de travail a été configuré, vous pouvez activer chaque machine virtuelle en installant l’agent de Log Analytics et l’agent de Dépendance. Il existe plusieurs méthodes pour installer ces agents, y compris les stratégies Azure qui vous permettent de configurer automatiquement chaque machine virtuelle lors de sa création. Les données de performance et les détails de processus recueillis par Azure Monitor pour les machines virtuelles sont stockés dans les journaux Azure Monitor.

Consultez [Vue d’ensemble de l’activation Azure Monitor pour machines virtuelles](insights/vminsights-enable-overview.md) pour obtenir des options de déploiement des agents sur vos machines virtuelles et les activer pour la surveillance.

### <a name="configure-workspace-to-collect-events"></a>Configurer l’espace de travail pour collecter des événements
Azure Monitor pour machines virtuelles recueillera les données de performances, ainsi que les détails et les dépendances des processus du système d’exploitation invité de chaque machines virtuelles. L’agent Log Analytics peut également collecter des journaux à partir de l’invité, y compris le journal des événements de Windows et syslog de Linux. Il récupère la configuration de ces journaux à partir de l’espace de travail Log Analytics auquel il est connecté. Vous ne devez configurer l’espace de travail qu’une seule fois, et chaque fois qu’un agent se connecte, il télécharge les modifications de configuration. 

Pour plus d’informations sur la configuration de votre espace de travail Log Analytics afin de collecter des données supplémentaires à partir des machines virtuelles de l’agent, consultez [Sources de données de l’agent dans Azure Monitor](platform/agent-data-sources.md).

> [!NOTE]
> Vous pouvez également configurer l’espace de travail pour collecter des compteurs de performance, mais cela sera probablement redondant avec les données de performances collectées par Azure Monitor pour machines virtuelles. Les données de performances collectées par l’espace de travail sont stockées dans la table *Perf*, tandis que les données de performances collectées par Azure Monitor pour machines virtuelles sont stockées dans la table *InsightsMetrics*. Configurez la collecte des performances dans l’espace de travail uniquement si vous avez besoin de compteurs qui ne sont pas déjà collectés par Azure Monitor pour machines virtuelles.

### <a name="diagnostic-extension-and-telegraf-agent"></a>Extension de diagnostic et l’agent Telegraf
Azure Monitor pour machines virtuelles utilise l’agent Log Analytics qui envoie les données de performances à un espace de travail Log Analytics, mais pas aux métriques Azure Monitor. L’envoi de ces données aux mesures permet de les analyser avec Metrics Explorer et de les utiliser avec des alertes de métrique. Cela nécessite l’extension de diagnostic sur Windows et l’agent Telegraf sur Linux.

Pour plus d’informations sur l’installation et la configuration de ces agents, consultez [Installer et configurer l’extension des Diagnostics Windows Azure (WAD)](platform/diagnostics-extension-windows-install.md) et [Collecter des métriques personnalisées pour une machine virtuelle Linux avec l’](platform/collect-custom-metrics-linux-telegraf.md)agent InfluxData Telegraf.


## <a name="monitor-applications"></a>Surveillance des applications
Azure Monitor surveille vos applications personnalisées à l’aide d’[Application Insights](app/app-insights-overview.md), que vous devez configurer pour chaque application que vous souhaitez analyser. Le processus de configuration varie selon le type d’application surveillé et le type d’analyse que vous souhaitez effectuer. Les données collectées par Application Insights sont stockées dans les mesures Azure Monitor, les journaux Azure Monitor et le stockage d’objets BLOB Azure, selon la fonctionnalité. Les données de performances sont stockées à la fois dans les métriques Azure Monitor et dans les journaux Azure Monitor sans aucune configuration supplémentaire.

### <a name="create-an-application-resource"></a>Créer une ressource d’application
Vous devez créer une ressource dans Application Insights pour chaque application que vous allez analyser. Les données de journal collectées par Application Insights sont stockées dans des journaux d’activité Azure Monitor pour une application basée sur un espace de travail. Les données de journal pour les applications classiques sont stockées séparément de votre espace de travail Log Analytics, comme décrit dans la section [Structure des données](platform/data-platform-logs.md#data-structure).

 Lorsque vous créez l’application, vous devez choisir entre Classique ou Basée sur un espace de travail. Consultez [Créer une ressource Application Insights](app/create-new-resource.md) pour créer une application classique. Pour créer une application basée sur un espace de travail, consultez [Ressources Application Insights basées sur un espace de travail (version préliminaire)](app/create-workspace-resource.md).

### <a name="configure-codeless-or-code-based-monitoring"></a>Configurer la surveillance sans code ou basée sur le code
Pour activer l’analyse pour une application, vous devez décider si vous allez utiliser l’analyse sans code ou basée sur le code. Le processus de configuration varie en fonction de cette décision et du type d’application que vous allez surveiller.

**La surveillance sans code** est plus facile à implémenter et peut être configurée après le développement de votre code. Elle ne nécessite aucune mise à jour de votre code. Pour plus d’informations sur l’activation de la surveillance en fonction de votre application, consultez les ressources suivantes.

- [Applications hébergées sur Azure Web Apps](app/azure-web-apps.md)
- [Applications Java](app/java-in-process-agent.md)
- [Applications ASP.NET hébergées dans IIS sur une machine virtuelle Azure ou un groupe de machines virtuelles identiques Azure](app/azure-vm-vmss-apps.md)
- [Applications ASP.NET hébergées par IIS sur une machine virtuelle locale](app/monitor-performance-live-website-now.md)


**La surveillance basée sur le code** est plus personnalisable et collecte des données de télémétrie supplémentaires, mais elle nécessite l’ajout d’une dépendance à votre code sur les packages NuGet du kit de développement logiciel (SDK) Application Insights. Pour plus d’informations sur l’activation de la surveillance en fonction de votre application, consultez les ressources suivantes.

- [Applications ASP.NET](app/asp-net.md)
- [Applications ASP.NET Core](app/asp-net-core.md)
- [Applications de console .NET](app/console.md)
- [Java](app/java-get-started.md)
- [Node.JS](app/nodejs.md)
- [Python](app/opencensus-python.md)
- [autres plateformes](app/platforms.md)

### <a name="configure-availability-testing"></a>Configurer un test de disponibilité
Les tests de disponibilité dans Application Insights sont des tests périodiques qui surveillent la disponibilité et la réactivité de votre application à intervalles réguliers à partir de points situés dans le monde entier. Vous pouvez créer un test ping simple gratuit ou créer une séquence de requêtes Web pour simuler des transactions utilisateur associées à un coût. 

Consultez [Surveiller la disponibilité de tout site Web](app/monitor-web-app-availability.md) pour résumer les différents types de test et les détails de leur création.

### <a name="configure-profiler"></a>Configurer Profiler
Profiler dans Application Insights fournit des traces de performances pour les applications .NET. Profiler vous aide à identifier le chemin de code « chaud » qui prend le plus de temps pendant le traitement d’une demande web particulière. Le processus de configuration de Profiler varie selon le type d’application. 

Voir [Applications de production Profile dans Azure avec Application Insights](app/profiler-overview.md) pour plus de détails sur la configuration de Profiler.

### <a name="configure-snapshot-debugger"></a>Configurer le Débogueur de capture instantanée
Le débogueur de capture instantanée dans Application Insights surveille la télémétrie des exceptions à partir de votre application .NET et recueille des instantanés de vos exceptions les plus fréquentes afin que vous disposiez des informations dont vous avez besoin pour diagnostiquer les problèmes en production. Le processus de configuration du Débogueur de capture instantanée varie selon le type d’application. 

Pour plus d’informations sur la configuration du Débogueur de capture instantanée, consultez [Instantanés de débogage sur les exceptions dans les applications .NET](app/snapshot-debugger.md).


## <a name="visualize-data"></a>Visualiser les données
Les insights et les solutions incluront leurs propres classeurs et vues pour analyser leurs données. En plus de ces informations, vous pouvez créer vos propres [visualisations](visualizations.md) y compris des données et tableaux de bord de classeurs pour Azure Monitor pour combiner des données Azure Monitor à des données provenant d’autres services dans Azure.


### <a name="create-workbooks"></a>Créer des classeurs
[Les classeurs](platform/workbooks-overview.md) dans Azure Monitor vous permettent de créer des rapports visuels enrichis dans le Portail Azure. Vous pouvez combiner différents jeux de données à partir des métriques Azure Monitor et des journaux Azure Monitor pour créer des expériences interactives unifiées. Vous pouvez accéder à une galerie de classeurs sous l’onglet **Classeurs** du menu Azure Monitor. 

Pour plus d’informations sur la création de classeurs personnalisés, consultez [classeurs Azure Monitor](platform/workbooks-overview.md).

### <a name="create-dashboards"></a>Créer des tableaux de bord
[Les tableaux de bord Azure](../azure-portal/azure-portal-dashboards.md) sont la principale technologie de tableau de bord pour Azure et vous permettent de combiner des données Azure Monitor avec des données provenant d’autres services pour fournir un seul volet de transparence à votre infrastructure Azure. Pour plus d’informations sur la création d’un tableau de bord qui comprend des données de journaux Azure Monitor, consultez [Créer et partager des tableaux de bord de données Log Analytics](learn/tutorial-logs-dashboards.md). 

Pour plus d’informations sur la création d’un tableau de bord qui comprend des données Application Insights, consultez [Créer des tableaux de bord d’indicateurs de performance clés personnalisés à l’aide d’Application Insights](learn/tutorial-app-dashboards.md). 

## <a name="alerts"></a>Alertes
Les alertes dans Azure Monitor vous informent de façon proactive des données et des modèles importants dans vos données de surveillance. Certains Insights génèrent des alertes sans configuration. Pour les autres scénarios, vous devez créer des [Règles d’alerte](platform/alerts-overview.md) qui incluent les données à analyser et les critères de génération d’une alerte, ainsi que des groupes d’actions qui définissent l’action à entreprendre lors de la génération d’une alerte. 


### <a name="create-action-groups"></a>Créer des groupes d’actions
[Les groupes d’actions](platform/action-groups.md) sont un ensemble de préférences de notification utilisées par les règles d’alerte pour déterminer l’action à effectuer lorsqu’une alerte est déclenchée. Voici quelques exemples d’actions : envoyer un message électronique ou du texte, appeler un webhook ou envoyer des données à un outil ITSM. Chaque règle d’alerte requiert au moins un groupe d’actions, et un groupe d’actions unique peut être utilisé par plusieurs règles d’alerte.

Consultez [Créer et gérer des groupes d’actions sur le Portail Azure](platform/action-groups.md) pour plus d’informations sur la création d’un groupe d’actions vous permettant de gérer les réponses aux alertes.


### <a name="create-alert-rules"></a>Créer des règles d'alerte
Plusieurs types de règles d’alerte sont définis par le type de données qu’ils utilisent. Chacun a des fonctionnalités différentes et un coût différent. La stratégie de base que vous devez suivre consiste à utiliser le type de règle d’alerte avec le coût le plus bas fournissant la logique dont vous avez besoin.

- [Règles du journal d’activité](platform/activity-log-alerts.md). Crée une alerte en réponse à un nouvel événement du journal d’activité qui correspond aux conditions spécifiées. Il n’y a aucun coût pour ces alertes, elles doivent être votre premier choix. Pour plus d’informations sur la création d’alertes de journal d'activité, consultez [Créer, afficher et gérer des alertes de métriques à l’aide d’Azure Monitor](platform/alerts-activity-log.md).
- [Règles d’alerte de métriques](platform/alerts-metric-overview.md). Crée une alerte en réponse à une ou plusieurs valeurs métriques dépassant un seuil. Les alertes métriques sont des alertes par état, c'est-à-dire qu'elles se ferment automatiquement lorsque la valeur passe en dessous du seuil, et qu'elles n'envoient des notifications que lorsque l'état change. Il y a un coût pour les alertes de métriques, mais cela est beaucoup moins important que les alertes de journal. Pour plus d’informations sur la création d’alertes de métriques, consultez [Créer, afficher et gérer des alertes de métrique à l’aide d’Azure Monitor](platform/alerts-metric.md).
- [Règles d’alerte de journal](platform/alerts-unified-log.md). Crée une alerte lorsque les résultats d’une requête de planification correspondent à des critères spécifiés. Il s’agit des règles d’alerte les plus coûteuses, mais elles autorisent les critères les plus complexes. Pour plus d’informations sur la création d’alertes de requête de journal, consultez [Créer, afficher et gérer des alertes de métrique à l’aide d’Azure Monitor](platform/alerts-log.md).
- [Les alertes d’application](app/monitor-web-app-availability.md) vous permettent d’effectuer des tests de performances et de disponibilité proactifs de votre application Web. Vous pouvez effectuer un test ping simple gratuitement, mais un test plus complexe engendrera des coûts. Consultez [Surveiller la disponibilité de tout site Web](app/monitor-web-app-availability.md) pour une description des différents types de test et des détails de leur création.


## <a name="next-steps"></a>Étapes suivantes

- Voir [Déployer les fonctionnalités Azure Monitor à la bonne échelle à l’aide d’Azure Policy](deploy-scale.md).