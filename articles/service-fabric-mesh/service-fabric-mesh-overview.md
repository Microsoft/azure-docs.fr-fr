---
title: Vue d’ensemble d’Azure Service Fabric Mesh
description: En savoir plus sur Azure Service Fabric Mesh. Avec Service Fabric Mesh, vous pouvez déployer et mettre à l’échelle votre application sans vous soucier des besoins d’infrastructure de votre application.
author: georgewallace
ms.author: gwallace
ms.date: 10/1/2018
ms.topic: overview
ms.openlocfilehash: 32e855ef1580dd065ee681e635994064a9038f77
ms.sourcegitcommit: b87c7796c66ded500df42f707bdccf468519943c
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/08/2020
ms.locfileid: "91840266"
---
# <a name="what-is-service-fabric-mesh"></a>Qu’est-ce que Service Fabric Mesh ?

Cette vidéo fournit une vue d’ensemble rapide de Service Fabric Mesh.
> [!VIDEO https://www.youtube.com/embed/7qWeVGzAid0]

Azure Service Fabric mesh est un service entièrement géré qui permet aux développeurs de déployer des applications de microservices sans gestion de machines virtuelles, de stockage ou de mise en réseau. Les applications hébergées sur Service Fabric Mesh sont exécutées et mises à l’échelle sans que vous ayez à vous soucier de l’infrastructure sous-jacente.  Service Fabric Mesh se compose de clusters constitués de milliers de machines.  Toutes les opérations de cluster sont cachées au développeur. Chargez votre code et spécifiez les ressources dont vous avez besoin, les exigences de disponibilité et les limites des ressources.  Service Fabric Mesh alloue automatiquement l’infrastructure et gère les défaillances d’infrastructure. Vous êtes ainsi assuré que vos applications sont hautement disponibles. Il vous suffit de veiller à l’intégrité et à la réactivité de votre application, mais pas à l’infrastructure.  

[!INCLUDE [preview note](./includes/include-preview-note.md)]

Cet article contient une vue d’ensemble des principaux avantages de Service Fabric Mesh.

## <a name="great-developer-experience"></a>Expérience de développement améliorée

Service Fabric Mesh prend en charge tout langage ou infrastructure de programmation pouvant être exécuté dans un conteneur. La prise en charge des outils de Visual Studio 2019 et Visual Studio Code assure une puissante expérience de modification et de débogage pour les applications .NET et .NET Core. 

Avec Service Fabric Mesh, vous pouvez :

- Effectuez une migration « lift-and-shift » des applications existantes dans des conteneurs pour moderniser et exécuter vos applications actuelles à grande échelle.
- Générer et déployer de nouvelles applications de microservices à grande échelle dans Azure.  Intégrez-les à d’autres services Azure ou applications existantes qui s’exécutent dans des conteneurs. Chaque microservice fait partie d’une application sécurisée et isolée du réseau. Le microservice applique des stratégies de gouvernance de ressources définies pour les cœurs de processeurs, la mémoire, l’espace disque, etc.
- Intégrer et étendre des applications existantes sans leur apporter de modifications. Utilisez votre propre réseau virtuel pour connecter une application existante à la nouvelle application.  
- Moderniser vos applications Services Cloud existantes en procédant à une migration vers Service Fabric Mesh.  

## <a name="simple-operational-lifecycle"></a>Cycle de vie opérationnel simple

Gérez facilement des applications en cours d’exécution, leur supervision ainsi que leur débogage dans des environnements de production. Cette gestion inclut les mises à niveau et les versions des applications. Ces applications peuvent contenir un seul microservice ou plusieurs microservices isolés dans leur propre réseau. Les applications sont exécutées efficacement avec des délais rapides de déploiement, de positionnement et de basculement.

Avec Service Fabric Mesh, vous pouvez :

- Déployer et gérer des applications sans devoir explicitement approvisionner et gérer l’infrastructure.  Service Fabric Mesh approvisionne, met à niveau, corrige et gère l’infrastructure sous-jacente à votre place.
- Configurer l’intégration continue en utilisant les outils intégrés pour empaqueter et déployer facilement des applications.
- Exploitez toutes les fonctionnalités des ressources Azure Resource Manager. Une piste d’audit et un [Contrôle d’accès en fonction du rôle (RBAC) Azure](../role-based-access-control/overview.md) sont des exemples de ces fonctionnalités. Toutes les ressources que vous déployez sur le service Service Fabric Mesh dans Azure sont des ressources Azure Resource Manager. Ces ressources incluent des applications, des services, des secrets, etc.
- Déployer et gérer des ressources à l’aide du [portail Azure](https://portal.azure.com), des modèles Resource Manager ou des bibliothèques d’Azure CLI/PowerShell.
- Configurer la supervision opérationnelle et la génération d’alertes à l’aide [d’Application Insights](/azure/application-insights/) (ou de l’outil de votre choix) pour capturer des traces opérationnelles et de diagnostic à partir de la plateforme.
- Accéder aux informations de diagnostic des applications émises à partir du modèle d’application à l’aide [d’Application Insights](/azure/application-insights/) ou de l’outil de votre choix.
- Optimisez l’utilisation de ressources en spécifiant des règles de mise à l’échelle automatique pour les services dans la définition d’application.

## <a name="mission-critical-platform-capabilities"></a>Fonctionnalités stratégiques de la plateforme

Service Fabric Mesh crée une collection de clusters qui s’étend sur les [Zones de disponibilité Azure](../availability-zones/az-overview.md) et/ou les limites régionales géopolitiques. Service Fabric Mesh représente des applications avec un ensemble d’intentions comme les besoins de mise à l’échelle, la configuration matérielle requise, les spécifications de durabilité et les stratégies de sécurité.  Lorsque l’application est déployée, Service Fabric Mesh recherche l’emplacement optimal pour l’exécuter.

Avec Service Fabric Mesh, vous pouvez :

- Tirer parti des caractéristiques suivantes : haute disponibilité, diminution/augmentation du nombre d’instances, détectabilité, orchestration, routage des messages, messagerie fiable, mises à niveau sans temps d’arrêt, gestion de la sécurité/des secrets, récupération d’urgence, gestion d’état, gestion de la configuration, et transactions distribuées.
- Choisir entre plusieurs modèles d’application lors de la création d’applications.
- Utiliser les fonctionnalités de la plateforme exposées via des points de terminaison REST en consommant des liaisons propres au langage générées à l’aide de Swagger.
- Déployer des applications entre les [Zones de disponibilité](../availability-zones/az-overview.md) et plusieurs régions pour la fiabilité géographique.
- Utiliser toutes les fonctionnalités de sécurité et de conformité fournies par Azure.

## <a name="next-steps"></a>Étapes suivantes

Le déploiement d’un exemple de projet avec Visual Studio ne prend que quelques étapes. Pour plus d’informations, consultez [Create an ASP.NET Core website](service-fabric-mesh-quickstart-dotnet-core.md) (Créer un site web ASP.NET Core). 

Lisez les réponses aux [questions fréquentes](service-fabric-mesh-faq.md).


<!-- Links -->

[service-fabric-overview]: ../service-fabric/service-fabric-overview.md
