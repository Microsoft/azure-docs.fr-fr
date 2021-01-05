---
title: Différences entre les services cloud et Service Fabric
description: Aperçu conceptuel pour apprendre à migrer des applications à partir des services cloud vers Service Fabric.
ms.topic: conceptual
ms.date: 11/02/2017
ms.openlocfilehash: c7e7d346b5a39a262d1d55265becadb1c718cc04
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96575769"
---
# <a name="learn-about-the-differences-between-cloud-services-and-service-fabric-before-migrating-applications"></a>Découvrez les différences entre les services cloud et Service Fabric avant de migrer les applications.
Microsoft Azure Service Fabric est la plateforme d’applications cloud nouvelle génération pour les applications distribuées hautement évolutives et fiables. Elle introduit de nombreuses nouvelles fonctionnalités d’empaquetage, de déploiement, de mise à niveau et de gestion des applications cloud distribuées. 

Ceci est un guide pour apprendre à migrer des applications à partir des services cloud vers Service Fabric. Il se concentre essentiellement sur les différences d’architecture et de conception entre les services cloud et Service Fabric.

## <a name="applications-and-infrastructure"></a>Applications et infrastructure
L’une des principales différences entre les services cloud et Service Fabric est la relation entre les machines virtuelles, les charges de travail et les applications. Une charge de travail correspond au code que vous écrivez pour effectuer une tâche spécifique ou fournir un service.

* **Les services cloud visent à déployer des applications en tant que machines virtuelles.** Le code que vous écrivez est étroitement lié à une instance de machine virtuelle, par exemple un rôle web ou de travail. Déployer une charge de travail dans les services cloud revient à déployer une ou plusieurs instances de machines virtuelles qui exécutent la charge de travail. Il n’existe pas de séparation entre les applications et les machines virtuelles, et par conséquent, les applications ne sont pas associées à une définition formelle. Une application peut être considérée comme un ensemble d’instances de rôle web ou de travail au sein d’un déploiement de services cloud ou comme un déploiement complet de services cloud. Dans cet exemple, une application est représentée comme un ensemble d’instances de rôle.

![Topologie et applications de service cloud][1]

* **Service Fabric vise à déployer des applications sur des machines virtuelles existantes ou sur des ordinateurs exécutant Service Fabric sur Windows ou Linux.** Les services que vous écrivez sont complètement dissociés de l’infrastructure sous-jacente, qui est éliminée par la plateforme d’application de Service Fabric. De ce fait, une application peut être déployée dans plusieurs environnements. Une charge de travail dans Service Fabric est appelée un « service », et un ou plusieurs services peuvent être regroupés dans une application formellement définie qui s’exécute sur la plateforme d’application Service Fabric. Plusieurs applications peuvent être déployées dans un seul cluster Service Fabric.

![Topologie et applications Service Fabric][2]

Service Fabric correspond à une couche de plateforme d’application qui s’exécute sur Windows ou Linux, alors que les services cloud constituent un système de déploiement de machines virtuelles gérées par Azure avec des charges de travail jointes.
Le modèle d’application Service Fabric présente plusieurs avantages :

* Temps de déploiement rapide. La création d’instances de machine virtuelle peut prendre beaucoup de temps. Dans Service Fabric, les machines virtuelles sont déployées une seule fois pour former un cluster qui héberge la plateforme d’application Service Fabric. À ce stade, les packages d’application peuvent être déployés vers le cluster très rapidement.
* Hébergement de haute densité. Dans les services cloud, une machine virtuelle de rôle de travail héberge une charge de travail. Dans Service Fabric, les applications sont séparées des machines virtuelles qui les exécutent, ce qui signifie que vous pouvez déployer un grand nombre d’applications sur un petit nombre de machines virtuelles. Résultat : une diminution des coûts globaux pour des déploiements plus importants.
* La plateforme Service Fabric peut s’exécuter sur n’importe quel système incluant des ordinateurs Windows Server ou Linux, qu’il s’agisse d’Azure ou d’un système sur site. La plateforme fournit une couche d’abstraction au-dessus de l’infrastructure sous-jacente pour permettre d’exécuter votre application sur différents environnements. 
* Gestion des applications distribuées. La plateforme Service Fabric ne se contente pas d’héberger les applications distribuées : elle permet également de gérer leur cycle de vie indépendamment de la machine virtuelle hôte ou du cycle de vie de la machine.

## <a name="application-architecture"></a>Architecture de l'application
L’architecture d’une application de service cloud inclut généralement de nombreuses dépendances de services externes, comme Service Bus, les tables et le stockage d’objets blob Azure, SQL, Redis etc., pour gérer l’état et les données d’une application et la communication entre les rôles web et de travail dans un déploiement de services cloud. Voici un exemple d’application de services cloud complète :  

![Architecture des services cloud][9]

Les applications Service Fabric peuvent également opter pour l’utilisation des mêmes services externes dans l’ensemble d’une application. Cet exemple d’architecture de services cloud montre que la manière la plus simple d’effectuer une migration des services cloud vers Service Fabric est de remplacer uniquement le déploiement des services cloud par une application Service Fabric, tout en conservant l’architecture globale. Les rôles web et de travail peuvent être transférés vers les services sans état de Service Fabric avec peu de modifications du code.

![Architecture Service Fabric après une migration simple][10]

À ce stade, le système doit continuer à fonctionner comme avant. En tirant profit des fonctionnalités avec état de Service Fabric, les magasins d’état externe peuvent être internalisés sous forme de services avec état le cas échéant. Cela est plus complexe qu’une simple migration de rôles web et de travail vers les services sans état Service Fabric, car il est nécessaire d’écrire les services personnalisés qui fournissent à votre application des fonctionnalités équivalentes à celles fournies auparavant par les services externes. Les avantages sont les suivants : 

* Suppression des dépendances externes 
* Unification des modèles de mise à niveau, de gestion et de déploiement 

Voici un exemple d’architecture résultant de l’internalisation de ces services :

![Architecture Service Fabric après une migration complète][11]

## <a name="communication-and-workflow"></a>Communication et workflow
La plupart des applications de service cloud sont constituées de plusieurs niveaux. De même, une application Service Fabric se compose de plusieurs services (généralement de nombreux services). La communication directe et la communication via un stockage durable externe constituent deux modèles de communication courants.

### <a name="direct-communication"></a>Communication directe
Avec la communication directe, les niveaux peuvent communiquer directement par le biais du point de terminaison exposé par chacun d’entre eux. Dans les environnements sans état tels que les services cloud, cela signifie qu’une instance d’un rôle de machine virtuelle doit être sélectionnée, soit au hasard, soit par tourniquet (round-robin), afin d’équilibrer la charge, et que la connexion au point de terminaison doit se faire directement.

![Communication directe des services cloud][5]

 La communication directe est un modèle de communication courant dans Service Fabric. La principale différence entre Service Fabric et les services cloud est que dans les services cloud, vous vous connectez à une machine virtuelle, tandis que dans Service Fabric, vous vous connectez à un service. Cette distinction est importante pour plusieurs raisons :

* Les services dans Service Fabric ne sont pas liés aux machines virtuelles qui les hébergent ; les services peuvent se déplacer au sein du cluster et en réalité, ils doivent même se déplacer pour différentes raisons : équilibrage des ressources, basculement, mises à niveau de l’application et de l’infrastructure, contraintes de positionnement ou de charge. Cela signifie que l’adresse d’une instance de service peut changer à tout moment. 
* Une machine virtuelle dans Service Fabric peut héberger plusieurs services, chacun avec des points de terminaison uniques.

Service Fabric fournit un mécanisme de découverte de service, appelé service d’affectation de noms, qui peut être utilisé pour résoudre les adresses de point de terminaison des services. 

![Diagramme qui montre comment Service Fabric fournit un mécanisme de découverte de service, appelé service d’affectation de noms, qui peut être utilisé pour résoudre les adresses de point de terminaison des services.][6]

### <a name="queues"></a>Files d’attente
Pour communiquer entre les niveaux dans des environnements sans état tels que les services cloud, il est courant d’utiliser une file d’attente de stockage externe pour enregistrer durablement les tâches de travail d’un niveau à l’autre. Il est fréquent qu’un niveau web envoie les travaux à un Service Bus ou à une file d’attente Azure où les instances de rôle de travail peuvent enlever les travaux de la file d’attente et les traiter.

![Communication par file d’attente des services cloud][7]

Le même modèle de communication peut être utilisé dans Service Fabric. Cela peut être utile lors de la migration d’une application existante d’un service cloud vers Service Fabric. 

![Communication directe Service Fabric][8]

## <a name="parity"></a>Parité
[Cloud Services est similaire à Service Fabric en termes de niveau de contrôle et de facilité d’utilisation, mais il s’agit désormais d’un service hérité et Service Fabric est recommandé pour un nouveau développement](/azure/architecture/guide/technology-choices/compute-decision-tree) ; voici une comparaison entre les API :


| **API Service Cloud** | **API Service Fabric** | **Remarques** |
| --- | --- | --- |
| RoleInstance.GetID | FabricRuntime.GetNodeContext.NodeId ou .NodeName | ID est une propriété de NodeName |
| RoleInstance.GetFaultDomain | FabricClient.QueryManager.GetNodeList | Filtrez sur NodeName et utilisez la propriété FD |
| RoleInstance.GetUpgradeDomain | FabricClient.QueryManager.GetNodeList | Filtrez sur NodeName et utilisez la propriété Upgrade |
| RoleInstance.GetInstanceEndpoints | FabricRuntime.GetActivationContext ou Naming (ResolveService) | CodePackageActivationContext, fournie par FabricRuntime.GetActivationContext et au sein des réplicas via ServiceInitializationParameters.CodePackageActivationContext, fournie pendant . Initialize |
| RoleEnvironment.GetRoles | FabricClient.QueryManager.GetNodeList | Si vous souhaitez effectuer le même type de filtrage par type, vous pouvez obtenir la liste des types de nœuds du manifeste du cluster via FabricClient.ClusterManager.GetClusterManifest et récupérer les types de nœud/rôle à partir de cet emplacement. |
| RoleEnvironment.GetIsAvailable | Connect-WindowsFabricCluster, ou créez un FabricRuntime pointant vers un nœud spécifique | * |
| RoleEnvironment.GetLocalResource | CodePackageActivationContext.Log/Temp/Work | * |
| RoleEnvironment.GetCurrentRoleInstance | CodePackageActivationContext.Log/Temp/Work | * |
| LocalResource.GetRootPath | CodePackageActivationContext.Log/Temp/Work | * |
| Role.GetInstances | FabricClient.QueryManager.GetNodeList ou ResolveService | * |
| RoleInstanceEndpoint.GetIPEndpoint | FabricRuntime.GetActivationContext ou Naming (ResolveService) | * |

## <a name="next-steps"></a>Étapes suivantes
Le moyen le plus simple de migrer des services cloud vers Service Fabric est de remplacer uniquement le déploiement des services cloud par une application Service Fabric, tout en conservant l’architecture globale de votre application presque inchangée. L’article suivant fournit un guide pour vous aider à convertir un rôle web ou de travail en service sans état Service Fabric.

* [Migration simple : convertir un rôle web ou de travail en service sans état Service Fabric](service-fabric-cloud-services-migration-worker-role-stateless-service.md)

<!--Image references-->
[1]: ./media/service-fabric-cloud-services-migration-differences/topology-cloud-services.png
[2]: ./media/service-fabric-cloud-services-migration-differences/topology-service-fabric.png
[5]: ./media/service-fabric-cloud-services-migration-differences/cloud-service-communication-direct.png
[6]: ./media/service-fabric-cloud-services-migration-differences/service-fabric-communication-direct.png
[7]: ./media/service-fabric-cloud-services-migration-differences/cloud-service-communication-queues.png
[8]: ./media/service-fabric-cloud-services-migration-differences/service-fabric-communication-queues.png
[9]: ./media/service-fabric-cloud-services-migration-differences/cloud-services-architecture.png
[10]: ./media/service-fabric-cloud-services-migration-differences/service-fabric-architecture-simple.png
[11]: ./media/service-fabric-cloud-services-migration-differences/service-fabric-architecture-full.png