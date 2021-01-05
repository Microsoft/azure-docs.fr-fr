---
title: Bonnes pratiques de supervision pour Azure Service Fabric
description: Bonnes pratiques et considérations de conception pour la surveillance des clusters et des applications à l’aide d’Azure Service Fabric.
author: peterpogorski
ms.topic: conceptual
ms.date: 01/23/2019
ms.author: pepogors
ms.openlocfilehash: a7b1c1b3fc3196557b862c488ee01af8b8e1f04f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86529248"
---
# <a name="monitoring-and-diagnostic-best-practices-for-azure-service-fabric"></a>Meilleures pratiques en matière de surveillance et de diagnostic pour Azure Service Fabric

[La supervision et les diagnostics](./service-fabric-diagnostics-overview.md) sont essentiels au développement, au test et au déploiement des charges de travail dans tout environnement cloud. Par exemple, vous pouvez suivre la façon dont vos applications sont utilisées, les actions effectuées par la plateforme Service Fabric, votre utilisation des ressources avec des compteurs de performances et l’intégrité globale de votre cluster. Vous pouvez utiliser ces informations pour diagnostiquer et corriger les problèmes, et éviter qu’ils ne se reproduisent.

## <a name="application-monitoring"></a>Monitoring des applications

Le monitoring des applications permet de suivre l’utilisation des fonctionnalités et des composants de votre application. Supervisez vos applications pour intercepter les problèmes qui impactent les utilisateurs. La responsabilité de la supervision des applications revient aux utilisateurs qui développent l’application et ses services, car elle s’applique uniquement à la logique métier de l’application. Il est recommandé de configurer la supervision des applications avec [Application Insights](./service-fabric-tutorial-monitoring-aspnet.md), qui est l’outil de supervision des applications d’Azure.

## <a name="cluster-monitoring"></a>Monitoring du cluster

L’un des objectifs de Service Fabric est d’assurer le bon fonctionnement des applications, même en cas de défaillances matérielles. Cet objectif repose sur la capacité des services système de la plateforme à détecter les problèmes d’infrastructure et à basculer rapidement les charges de travail sur d’autres nœuds du cluster. Mais que se passe-t-il si les services système subissent eux aussi des problèmes ? Que se passe-t-il si, durant une tentative de déploiement ou de déplacement d’une charge de travail, les règles de placement des services sont enfreintes ? Dans ce cas, Service Fabric fournit des diagnostics pour vous informer de la façon dont la plateforme Service Fabric interagit avec vos applications, services, conteneurs et nœuds.

Pour des clusters Windows, il est recommandé de définir la supervision des clusters avec l’[agent Diagnostics](./service-fabric-diagnostics-event-aggregation-wad.md) et les [journaux Azure Monitor](./service-fabric-diagnostics-oms-setup.md).

Pour des clusters Linux, les journaux Azure Monitor sont également l’outil recommandé pour la supervision de l’infrastructure et de la plateforme Azure. Les diagnostics de plateforme Linux nécessitent une configuration différente, comme indiqué dans [Événements de cluster Linux Service Fabric dans Syslog](./service-fabric-diagnostics-oms-syslog.md).

## <a name="infrastructure-monitoring"></a>Supervision des infrastructures

Les [journaux Azure Monitor](./service-fabric-diagnostics-oms-agent.md) sont recommandé pour la supervision des événements au niveau du cluster. Une fois que vous aurez configuré l’agent Log Analytics avec votre espace de travail comme décrit dans le lien précédent, vous pourrez collecter des métriques de performances telles que l’utilisation du processeur, des compteurs de performances .NET tels que l’utilisation du processeur au niveau du processus, des compteurs performances Service Fabric tels que le nombre d’exceptions d’un service fiable, et des métriques de conteneur, telles que l’utilisation du processeur.  Vous devez écrire les journaux du conteneur dans stdout ou stderr pour les rendre disponibles dans les journaux Azure Monitor.

## <a name="watchdogs"></a>Agents de surveillance

En général, un agent de surveillance est un service distinct capable de surveiller l’intégrité et la charge des services, d’effectuer des tests ping sur les points de terminaison et de créer des rapports à partir des événements non sains du cluster. Cela vous permet de détecter plus facilement les erreurs que vous n’auriez pas pu détecter en vous basant uniquement sur les performances d’un seul service. Les agents de surveillance constituent également un bon emplacement pour héberger du code qui exécute des actions correctives sans intervention de l’utilisateur (par exemple, le nettoyage de fichiers journaux dans le stockage à intervalles réguliers). Pour voir un exemple d’implémentation de l’agent de surveillance, consultez [Événements de cluster Linux Service Fabric dans Syslog](https://github.com/Azure-Samples/service-fabric-watchdog-service).

## <a name="next-steps"></a>Étapes suivantes

* Pour instrumenter vos applications : [Événements au niveau de l’application et génération de journaux](service-fabric-diagnostics-event-generation-app.md).
* Suivez les étapes de configuration d’Application Insights pour votre application dans [Surveiller et diagnostiquer une application ASP.NET Core dans Service Fabric](service-fabric-tutorial-monitoring-aspnet.md).
* Pour plus d’informations sur la supervision de la plateforme et les événements fournis par Service Fabric : [Événements au niveau de la plateforme et génération de journaux](service-fabric-diagnostics-event-generation-infra.md).
* Configurer l’intégration des journaux Azure Monitor avec Service Fabric : [Configurer les journaux Azure Monitor pour un cluster](service-fabric-diagnostics-oms-setup.md)
* Découvrez comment configurer les journaux Azure Monitor pour superviser des conteneurs : [Supervision et diagnostic des conteneurs Windows dans Azure Service Fabric](service-fabric-tutorial-monitoring-wincontainers.md).
* Pour consulter des exemples de problèmes de diagnostic et de solutions Service Fabric : [Diagnostiquer des scénarios courants](service-fabric-diagnostics-common-scenarios.md)
* Découvrez les recommandations générales sur la supervision des ressources Azure : [Bonnes pratiques : Supervision et diagnostics](/azure/architecture/best-practices/monitoring).
