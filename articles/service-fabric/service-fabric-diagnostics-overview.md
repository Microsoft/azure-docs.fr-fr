---
title: Vue d’ensemble de la surveillance et des diagnostics Azure Service Fabric
description: Découvrez la surveillance et les diagnostics pour les clusters, applications et services Azure Service Fabric.
author: srrengar
ms.topic: conceptual
ms.date: 1/17/2019
ms.author: srrengar
ms.openlocfilehash: 25a50a2841a03929804be45be8012f9b5d0457ff
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91357129"
---
# <a name="monitoring-and-diagnostics-for-azure-service-fabric"></a>Surveillance et diagnostics pour Azure Service Fabric

Cet article fournit une vue d’ensemble du monitoring et des diagnostics dans Azure Service Fabric. Le monitoring et les diagnostics sont essentiels au développement, au test et au déploiement de charges de travail dans tout environnement cloud. Par exemple, vous pouvez suivre la façon dont vos applications sont utilisées, les actions effectuées par la plateforme Service Fabric, votre utilisation des ressources avec des compteurs de performances et l’intégrité globale de votre cluster. Vous pouvez utiliser ces informations pour diagnostiquer et corriger les problèmes, et éviter qu’ils ne se reproduisent. Les sections suivantes décrivent brièvement chaque zone de la supervision de Service Fabric à prendre en compte pour les charges de travail de production. 

[!INCLUDE [azure-monitor-log-analytics-rebrand](../../includes/azure-monitor-log-analytics-rebrand.md)]

## <a name="application-monitoring"></a>Monitoring des applications
Le monitoring des applications permet de suivre l’utilisation des fonctionnalités et des composants de votre application. L’objectif est d’intercepter les problèmes qui impactent les utilisateurs. La responsabilité de la supervision des applications revient aux utilisateurs qui développent une application et ses services dans la mesure où il est unique à la logique métier de l’application. Le monitoring des applications peut être utile dans les scénarios suivants :
* Quel est le trafic que rencontre mon application ? - Devez-vous mettre à l’échelle vos services pour répondre aux demandes des utilisateurs ou traiter un goulot d’étranglement potentiel dans votre application ?
* Mes appels de service à service réussissent-ils et font-ils l’objet d’un suivi ?
* Quelles actions sont prises par les utilisateurs de mon application ? - La collecte de données de télémétrie peut guider le développement de nouvelles fonctionnalités et améliorer les diagnostics des erreurs d’application
* Mon application lève-t-elle des exceptions non gérées ? 
* Que se passe-t-il dans les services qui s’exécutent au sein de mes conteneurs ?

L’avantage de la supervision des applications est que les développeurs peuvent utiliser les outils et le framework qu’ils souhaitent dans la mesure où il réside dans le contexte de votre application. Vous pouvez en apprendre plus sur la solution Azure pour la supervision des applications avec Azure Monitor - Application Insights dans [Analyse d’événements avec Application Insights](service-fabric-diagnostics-event-analysis-appinsights.md).
Nous disposons également d’un tutoriel qui explique comment [configurer ceci pour les applications .NET](service-fabric-tutorial-monitoring-aspnet.md). Ce tutoriel aborde la façon d’installer les outils appropriés, un exemple pour écrire des données de télémétrie personnalisées dans votre application, et l’affichage des données de télémétrie et de diagnostic de l’application dans le portail Azure. 


## <a name="platform-cluster-monitoring"></a>Monitoring de la plateforme (cluster)
Un utilisateur contrôle les données de télémétrie qui proviennent de l’application dans la mesure où un utilisateur écrit le code lui-même, mais qu’en est-il du diagnostic provenant de la plateforme Service Fabric ? L’un des objectifs de Service Fabric est d’assurer le bon fonctionnement des applications même en cas de défaillances matérielles. Cet objectif repose sur la capacité des services système de la plateforme à détecter les problèmes d’infrastructure et à basculer rapidement les charges de travail sur d’autres nœuds du cluster. Mais dans ce cas précis, que se passe-t-il si les services système subissent eux aussi des problèmes ? Que se passe-t-il si, durant une tentative de déploiement ou de déplacement d’une charge de travail, les règles de placement des services sont enfreintes ? Service Fabric fournit des diagnostics pour cela et pour vous garantir d’être informé de l’activité en cours dans votre cluster. Voici quelques exemples de scénarios pour la surveillance du cluster :

Service Fabric fournit un ensemble complet d’événements prêts à l’emploi. Ces [événements Service Fabric](service-fabric-diagnostics-events.md) sont accessibles via EventStore ou le canal opérationnel (canal d’événements exposé par la plateforme). 

* Canaux d’événements Service Fabric : sur Windows, les événements Service Fabric sont disponibles à partir d’un seul fournisseur ETW avec un ensemble de filtres `logLevelKeywordFilters` pertinents permettant de choisir entre le canal opérationnel et le canal de données et de messagerie. Il s’agit de la méthode à l’aide de laquelle nous séparons des événements Service Fabric sortants à filtrer en fonction de vos besoins. Sur Linux, les événements Service Fabric transitent par LTTng et sont placés dans une table de stockage où ils peuvent être filtrés selon les besoins. Ces canaux contiennent des événements organisés et structurés que vous pouvez utiliser pour mieux comprendre l’état de votre cluster. Les diagnostics sont activés par défaut au moment de la création du cluster. Vous disposez ainsi d’une table Stockage Azure où sont envoyés les événements de ces canaux que vous pourrez interroger ultérieurement. 

* EventStore - EventStore est une fonctionnalité offerte par la plateforme qui fournit les événements de la plateforme Service Fabric disponibles dans Service Fabric Explorer et via l’API REST. Vous pouvez obtenir une vue de capture de ce qui se passe dans votre cluster pour chaque entité (nœud, service, application et requête) basée sur l’heure de l’événement. Vous pouvez en savoir plus sur EventStore dans [Vue d’ensemble d’EventStore](service-fabric-diagnostics-eventstore.md).    

![Capture d'écran représentant l'onglet ÉVÉNEMENTS du volet Nœuds et affichant plusieurs événements, dont un événement NodeDown.](media/service-fabric-diagnostics-overview/eventstore.png)

Les diagnostics fournis sont sous la forme d’un ensemble complet d’événements prêts à l’emploi. Ces [événements Service Fabric](service-fabric-diagnostics-events.md) illustrent les actions effectuées par la plateforme sur différentes entités telles que les nœuds, les applications, les services, les partitions, etc. Dans le dernier scénario ci-dessus, si un nœud venait à tomber en panne, la plateforme émettrait un événement `NodeDown` et vous pourriez être informé immédiatement par votre outil de supervision préféré. D’autres exemples courants incluent `ApplicationUpgradeRollbackStarted` ou `PartitionReconfigured` lors d’un basculement. **Les mêmes événements sont disponibles sur les clusters Windows et Linux.**

Les événements sont envoyés par le biais des canaux standard sur Windows et Linux, et peuvent être lus par n’importe quel outil de supervision qui les prend en charge. La solution Azure Monitor correspond aux journaux Azure Monitor. Renseignez-vous sur notre [intégration des journaux Azure Monitor](service-fabric-diagnostics-event-analysis-oms.md), qui comporte un tableau de bord opérationnel personnalisé pour votre cluster et quelques exemples de requêtes permettant de créer des alertes. D’autres concepts de supervision de cluster sont disponibles dans [Événement au niveau de la plateforme et génération de journal](service-fabric-diagnostics-event-generation-infra.md).

### <a name="health-monitoring"></a>Surveillance de l’intégrité
La plateforme Service Fabric inclut un modèle d’intégrité qui fournit des rapports d’intégrité extensibles sur l’état des entités dans un cluster. Chaque nœud, application, service, partition, réplica ou instance a un état d’intégrité qui est mis à jour en permanence. L’état d’intégrité peut avoir la valeur « OK », « Avertissement » ou « Erreur ». Considérez les événements Service Fabric comme des verbes appliqués par le cluster aux diverses entités et l’intégrité comme un adjectif pour chaque entité. Chaque fois que l’intégrité d’une entité particulière change, un événement est également émis. De cette façon, vous pouvez définir des requêtes et des alertes pour les événements d’intégrité dans votre outil de supervision préféré, tout comme pour n’importe quel autre événement. 

De plus, nous laissons même les utilisateurs remplacer l’intégrité des entités. Si votre application est en cours de mise à niveau et que des tests de validation échouent, vous pouvez écrire dans le composant d’intégrité de Service Fabric à l’aide de l’API Intégrité pour indiquer que votre application n’est plus intègre, et Service Fabric restaurera automatiquement la mise à niveau. Pour plus d’informations sur le modèle d’intégrité, consultez [Présentation du contrôle d’intégrité de Service Fabric](service-fabric-health-introduction.md).

![Tableau de bord d’intégrité SFX](media/service-fabric-diagnostics-overview/sfx-healthstatus.png)


### <a name="watchdogs"></a>Agents de surveillance
En général, un agent de surveillance est un service distinct capable de surveiller l’intégrité et la charge des services, d’effectuer des tests ping sur les points de terminaison et de créer des rapports d’intégrité pour n’importe quel élément du cluster. Cela permet d’éviter des erreurs qui ne seraient pas détectées à partir de l’affichage d’un service unique. Les agents de surveillance sont également un bon emplacement pour héberger du code qui exécute des actions correctives sans intervention de l’utilisateur (par exemple, le nettoyage de fichiers journaux dans le stockage à intervalles réguliers). Vous trouverez un exemple d’implémentation de service d’agent de surveillance [ici](https://github.com/Azure-Samples/service-fabric-watchdog-service).

## <a name="infrastructure-performance-monitoring"></a>Supervision de l’infrastructure (performances)
Maintenant que nous avons couvert les diagnostics dans votre application et sur la plateforme, comment savons-nous que le matériel fonctionne comme prévu ? Le monitoring de l’infrastructure sous-jacente est essentiel pour comprendre l’état de votre cluster et l’utilisation de vos ressources. La mesure des performances système dépend de nombreux facteurs qui peuvent être subjectifs en fonction de vos charges de travail. Ces facteurs sont généralement mesurés via des compteurs de performances. Ces compteurs de performances peuvent provenir de diverses sources, y compris du système d’exploitation, du .NET Framework ou de la plateforme Service Fabric proprement dite. Voici quelques scénarios dans lesquels ils peuvent être utiles :

* Est-ce que j’utilise efficacement mon matériel ? Voulez-vous utiliser votre matériel à 90 % ou 10 % du processeur ? Cela peut être pratique lors de la mise à l’échelle de votre cluster ou de l’optimisation des processus de votre application.
* Puis-je prévoir des problèmes d’infrastructure de façon proactive ? - de nombreux problèmes sont précédés de baisses (chutes) soudaines des performances. Vous pouvez donc utiliser des compteurs de performances comme E/S réseau et Utilisation de l’UC pour prédire et diagnostiquer les problèmes de façon proactive.

Vous trouverez la liste des compteurs de performances à collecter au niveau infrastructure dans [Métriques de performances](service-fabric-diagnostics-event-generation-perf.md). 

Service Fabric offre également une série de compteurs de performances pour les modèles de programmation Reliable Services et Reliable Actors. Si vous utilisez l’un de ces modèles, ces compteurs de performances peuvent fournir des informations garantissant que les procédures de « spin up » et de « spin down » de vos acteurs se déroulent correctement ou que vos demandes de services fiables sont gérées assez rapidement. Pour plus d’informations, consultez [Surveillance pour Reliable Service Remoting](service-fabric-reliable-serviceremoting-diagnostics.md#performance-counters) et [Surveillance des performances pour Reliable Actors](service-fabric-reliable-actors-diagnostics.md#performance-counters). 

La solution Azure Monitor permettant de les collecter correspond aux journaux Azure Monitor, tout comme le monitoring au niveau de la plateforme. Utilisez [l’agent Log Analytics](service-fabric-diagnostics-oms-agent.md) pour collecter les compteurs de performances souhaités et les afficher dans les journaux Azure Monitor.

## <a name="recommended-setup"></a>Configuration recommandée
Maintenant que nous avons passé en revue chaque zone de supervision et les exemples de scénarios, voici un résumé des outils de supervision Azure et de leur configuration nécessaire pour superviser toutes les zones ci-dessus. 

* Supervision des applications avec [Application Insights](service-fabric-tutorial-monitoring-aspnet.md)
* Monitoring du cluster avec [l’agent Diagnostics](service-fabric-diagnostics-event-aggregation-wad.md) et les [journaux Azure Monitor](service-fabric-diagnostics-oms-setup.md)
* Monitoring de l’infrastructure avec les journaux [Azure Monitor](service-fabric-diagnostics-oms-agent.md)

Vous pouvez également utiliser et modifier l’exemple de modèle ARM situé [ici](service-fabric-diagnostics-oms-setup.md#deploy-azure-monitor-logs-with-azure-resource-manager) pour automatiser le déploiement de tous les agents et ressources nécessaires. 

## <a name="other-logging-solutions"></a>Autres solutions de journalisation

Même si les deux solutions que nous recommandons, les [journaux Azure Monitor](service-fabric-diagnostics-event-analysis-oms.md) et [Application Insights](service-fabric-diagnostics-event-analysis-appinsights.md), s’intègrent à Service Fabric, de nombreux événements sont écrits par les fournisseurs ETW et peuvent être étendus avec d’autres solutions de journalisation. Intéressez-vous également à [Elastic Stack](https://www.elastic.co/products) (notamment si vous envisagez d’exécuter un cluster dans un environnement hors connexion), à [Dynatrace](https://www.dynatrace.com/) ou à toute autre plateforme de votre choix. Vous trouverez la liste des partenaires intégrés disponibles [ici](service-fabric-diagnostics-partners.md).

Quelle que soit la plateforme que vous choisissez, ses points clés doivent inclure la convivialité de l’interface utilisateur, les fonctions d’interrogation, les visualisations et tableaux de bord personnalisés disponibles, ainsi que les outils supplémentaires qu’elle met à votre disposition pour améliorer votre expérience de supervision. 

## <a name="next-steps"></a>Étapes suivantes

* Pour commencer à instrumenter vos applications, consultez [Génération d’événements et de journaux au niveau application](service-fabric-diagnostics-event-generation-app.md).
* Suivez les étapes de configuration d’Application Insights pour votre application dans [Surveiller et diagnostiquer une application ASP.NET Core dans Service Fabric](service-fabric-tutorial-monitoring-aspnet.md).
* Pour plus d’informations sur la surveillance de la plateforme et des événements fournis par Service Fabric, consultez [Génération d’événements et de journaux au niveau plateforme](service-fabric-diagnostics-event-generation-infra.md).
* Configurez l’intégration des journaux Azure Monitor à Service Fabric, dans [Configurer les journaux Azure Monitor pour un cluster](service-fabric-diagnostics-oms-setup.md).
* Découvrez comment configurer les journaux Azure Monitor pour le monitoring des conteneurs dans [Monitoring et diagnostics des conteneurs Windows dans Azure Service Fabric](service-fabric-tutorial-monitoring-wincontainers.md).
* Consultez des exemples de problèmes de diagnostic et de solutions avec Service Fabric dans l’article [Diagnostiquer des scénarios courants](service-fabric-diagnostics-common-scenarios.md).
* Découvrez d’autres produits de diagnostic qui s’intègrent à Service Fabric dans l’article [Solutions de partenaires pour la surveillance d’Azure Service Fabric](service-fabric-diagnostics-partners.md).
* Découvrez les recommandations générales sur la surveillance des ressources Azure : [Bonnes pratiques : Surveillance et diagnostics](/azure/architecture/best-practices/monitoring). 
