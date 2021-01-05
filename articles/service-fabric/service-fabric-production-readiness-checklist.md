---
title: Liste de contrôle de disponibilité de production Azure Service Fabric
description: Préparez votre application Service Fabric et votre cluster de production en suivant les meilleures pratiques.
ms.topic: conceptual
ms.date: 6/05/2019
ms.openlocfilehash: a03df40a8ce213c5de9ed7017d47713c4de3449d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87835865"
---
# <a name="production-readiness-checklist"></a>Liste de vérification de disponibilité de la production

Votre application et le cluster sont prêts à accepter le trafic de production ? Le fait que vous exécutiez et testiez votre application et votre cluster ne signifie pas nécessairement qu’il sont prêts à passer en production. Continuez à exécuter votre application et votre cluster en douceur tout en passant en revue la liste de contrôle suivante. Nous recommandons vivement de vérifier tous ces points. Bien sûr, vous pouvez choisir d’utiliser d’autres solutions pour une ligne (par exemple, vos propres infrastructures de diagnostic).


## <a name="prerequisites-for-production"></a>Conditions préalables à la production
1. Les meilleures pratiques Service Fabric sont : [Conception d’applications](./service-fabric-best-practices-applications.md), [Sécurité](./service-fabric-best-practices-security.md), [Mise en réseau](./service-fabric-best-practices-networking.md), [Planification de la capacité et mise à l’échelle](./service-fabric-best-practices-capacity-scaling.md), [Infrastructure as code](./service-fabric-best-practices-infrastructure-as-code.md), et [Surveillance et diagnostics](./service-fabric-best-practices-monitoring.md). 
1. [Configurez les paramètres FabricTransport](./service-fabric-reliable-actors-fabrictransportsettings.md) si vous utilisez le modèle de programmation Reliable Actors et que vous devez sécuriser la communication entre les services.
1. Pour les clusters comprenant plus de 20 cœurs ou 10 nœuds, créez un type de nœud principal dédié aux services système. Ajoutez des [contraintes de placement](service-fabric-cluster-resource-manager-advanced-placement-rules-placement-policies.md) pour réserver le type de nœud principal aux services système.
1. Utilisez une référence (SKU) D2v2 ou supérieure pour le type de nœud principal. Il est recommandé de choisir une référence (SKU) avec une capacité de disque dur d’au moins 50 Go.
1. Les clusters de production doivent être [sécurisés](service-fabric-cluster-security.md). Pour un exemple de configuration de cluster sécurisé, regardez ce [modèle de cluster](https://github.com/Azure-Samples/service-fabric-cluster-templates/tree/master/7-VM-Windows-3-NodeTypes-Secure-NSG). Utilisez des noms communs pour les certificats, et évitez d’utiliser des certificats auto-signés.
1. Ajoutez [des contraintes de ressources sur les conteneurs et les services](service-fabric-resource-governance.md), afin qu’ils ne consomment pas plus de 75 % des ressources du nœud. 
1. Comprenez et définissez le [niveau de durabilité](service-fabric-cluster-capacity.md#durability-characteristics-of-the-cluster). Le niveau de durabilité Silver ou un niveau supérieur sont recommandés pour les types de nœuds exécutant des charges de travail avec état.
1. Comprenez et choisissez le [niveau de fiabilité](service-fabric-cluster-capacity.md#reliability-characteristics-of-the-cluster) du type de nœud. Une fiabilité de niveau Silver ou d’un niveau supérieur est recommandée.
1. Chargez vos charges de travail et testez-les à l’échelle afin d’identifier les [besoins en capacité](service-fabric-cluster-capacity.md) pour votre cluster. 
1. Vos services et applications sont surveillés, et les journaux des applications sont générés et stockés avec génération d’alertes. Consultez, par exemple, [Ajouter la journalisation à votre application Service Fabric](service-fabric-how-to-diagnostics-log.md) et [Surveiller les conteneurs avec les journaux Azure Monitor](service-fabric-diagnostics-oms-containers.md).
1. Le cluster est surveillé avec un dispositif de génération d’alertes (par exemple, les [journaux Azure Monitor](service-fabric-diagnostics-event-analysis-oms.md)). 
1. L’infrastructure de groupe de machines virtuelles identiques sous-jacente est surveillée avec un dispositif de génération d’alertes (par exemple, les [journaux Azure Monitor](service-fabric-diagnostics-oms-agent.md)).
1. Le cluster a toujours des [certificats primaire et secondaire](service-fabric-cluster-security-update-certs-azure.md) (de sorte que vous n’êtes verrouillé).
1. Conservez des clusters distincts pour le développement, la préproduction et la production. 
1. Les [mises à niveau d’application](service-fabric-application-upgrade.md) et les [mises à niveau de cluster](service-fabric-tutorial-upgrade-cluster.md) sont testées d’abord dans des clusters de développement et de préproduction. 
1. Désactivez les mises à niveau automatiques dans les clusters de production, et activez-les pour les clusters de développement et de préproduction (restaurez au besoin). 
1. Fixez un objectif de point de récupération (RPO) pour votre service, configurez un [processus de récupération d’urgence](service-fabric-disaster-recovery.md), puis testez-le.
1. Planifiez la [mise à l’échelle](service-fabric-cluster-scaling.md) de votre cluster manuellement ou par programmation.
1. Planifiez la [mise à jour corrective](service-fabric-patch-orchestration-application.md) de vos nœuds de cluster. 
1. Établissez un pipeline d’intégration/livraison continue afin que vos dernières modifications soient testées en permanence. Par exemple, en utilisant [Azure DevOps](service-fabric-tutorial-deploy-app-with-cicd-vsts.md) ou [Jenkins](/azure/developer/jenkins/deploy-to-service-fabric-cluster)
1. Testez vos clusters de développement et de préproduction sous charge avec le [service d’analyse des défaillances](service-fabric-testability-overview.md), et provoquez un [chaos](service-fabric-controlled-chaos.md) contrôlé. 
1. Planifiez la [mise à l’échelle](service-fabric-concepts-scalability.md) de vos applications. 


Si vous utilisez le modèle de programmation Reliable Services ou Reliable Actors de Service Fabric, vous devez cocher les éléments suivants :
1. Mettez à niveau les applications pendant le développement local pour vérifier que votre code de service honore le jeton d’annulation dans la méthode `RunAsync` et ferme les écouteurs de communication personnalisés.
1. Évitez les [pièges courants](service-fabric-work-with-reliable-collections.md) lors de l’utilisation de Collections fiables.
1. Surveillez les compteurs de performances de mémoire CLR .NET lors de l’exécution des tests de charge, et vérifiez la présence de taux élevés de nettoyage de la mémoire ou de croissance de pile d’échappement.
1. Conservez une sauvegarde hors connexion de [Reliable Services et de Reliable Actors](service-fabric-reliable-services-backup-restore.md), puis testez le processus de restauration.
1. Idéalement, le nombre d'instances de machines virtuelles NodeType principales doit être égal au niveau de fiabilité minimum des clusters ; conditions nécessaires pour dépasser le niveau minimum : temporairement, lors de la mise à l'échelle verticale de la référence SKU du groupe principal de machines virtuelles identiques NodeType.

## <a name="optional-best-practices"></a>Meilleures pratiques facultatives

Si les listes ci-dessus sont des conditions préalables à la mise en production, les éléments suivants doivent également être pris en considération :
1. Connectez-vous au [modèle d’intégrité Service Fabric](service-fabric-health-introduction.md) pour étendre l’évaluation d’intégrité et les rapports intégrés.
1. Déployez une surveillance personnalisée qui analyse votre application et rende compte de la [charge](service-fabric-cluster-resource-manager-metrics.md) pour [l’équilibrage des ressources](service-fabric-cluster-resource-manager-balancing.md). 


## <a name="next-steps"></a>Étapes suivantes
* [Déployez un cluster Windows Service Fabric](service-fabric-tutorial-create-vnet-and-windows-cluster.md)
* [Déployez un cluster Linux Service Fabric](service-fabric-tutorial-create-vnet-and-linux-cluster.md)
* En savoir plus sur le [cycle de vie des applications](service-fabric-application-lifecycle.md) Service Fabric.