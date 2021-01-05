---
title: Scénarios et conception d’applications
description: Vue d'ensemble des catégories d'applications cloud dans Service Fabric. Traite de la conception d’applications à l’aide de services avec ou sans état
ms.topic: conceptual
ms.date: 01/08/2020
ms.openlocfilehash: 6c3cc931a85b91fc02b8086ca5c2481153691e54
ms.sourcegitcommit: 16c7fd8fe944ece07b6cf42a9c0e82b057900662
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/03/2020
ms.locfileid: "96575667"
---
# <a name="service-fabric-application-scenarios"></a>Scénarios d'applications Service Fabric

Azure Service Fabric offre une plateforme fiable et flexible où vous pouvez écrire et exécuter de nombreux types de services et d’applications d’entreprise. Ces applications et microservices peuvent être sans état ou avec état, et font l’objet d’un équilibrage des ressources entre les machines virtuelles pour optimiser l’efficacité.

L'architecture unique de Service Fabric vous permet d'exécuter dans vos applications des traitements d'événement, des transactions parallèles, des calculs en mémoire et des analyses de données pratiquement en temps réel. Vous pouvez facilement effectuer un scale-in ou un scale-out de vos applications selon l’évolution de vos besoins en ressources.

Pour obtenir des conseils de conception sur la création d’applications, consultez [Architecture des microservices sur Azure Service Fabric](/azure/architecture/reference-architectures/microservices/service-fabric) et [Bonnes pratiques relatives à la conception des applications à l’aide de Service Fabric](service-fabric-best-practices-applications.md).

Envisagez d’utiliser la plateforme Service Fabric pour les types d’applications suivants :

* **Collecte des données, traitement et IoT** : Service Fabric, par l’intermédiaire de ses services avec état, gère une échelle importante et possède une faible latence. Il peut s’avérer utile pour traiter des données sur des millions d’appareils sur lesquels les données de l’appareil et le calcul sont colocalisés.

    Les clients qui ont créé des services IoT avec Service Fabric sont notamment [Honeywell](https://customers.microsoft.com/story/honeywell-builds-microservices-based-thermostats-on-azure), [PCL Construction](https://customers.microsoft.com/story/pcl-construction-professional-services-azure), [Crestron](https://customers.microsoft.com/story/crestron-partner-professional-services-azure), [BMW](https://customers.microsoft.com/story/bmw-enables-driver-mobility-via-azure-service-fabric/), [Schneider Electric](https://customers.microsoft.com/story/schneider-electric-powers-engergy-solutions-on-azure-service-fabric) et [Mesh Systems](https://customers.microsoft.com/story/mesh-systems-lights-up-the-market-with-iot-based-azure-solutions).

* **Gaming et applications interactives basées sur les sessions** : Service Fabric est utile si votre application a besoin d’effectuer des opérations de lecture et d’écriture avec une faible latence, comme le gaming en ligne ou la messagerie instantanée. Service Fabric vous permet de générer ces applications interactives avec état sans avoir à créer de cache ou de magasin distinct. Consultez les [solutions de gaming Azure](https://azure.microsoft.com/solutions/gaming/) pour obtenir des conseils de conception sur l’[utilisation de Service Fabric dans des services de gaming](/gaming/azure/reference-architectures/multiplayer-synchronous-sf).

    Les clients qui ont créé des services de gaming sont notamment [Next Games](https://customers.microsoft.com/story/next-games-media-telecommunications-azure) et [Digamore](https://customers.microsoft.com/story/digamore-entertainment-scores-with-a-new-gaming-platform-based-on-azure-service-fabric/). Les clients qui ont créé des sessions interactives sont notamment [Honeywell avec Hololens](https://customers.microsoft.com/story/honeywell-manufacturing-hololens).

* **Analytique de données et traitement de workflow** : les applications qui doivent traiter des événements ou des flux de données de manière fiable bénéficient des opérations de lecture et d’écriture optimisées de Service Fabric. Service Fabric prend également en charge les pipelines de traitement d’applications où les résultats doivent être fiables et transmis à l’étape de traitement suivante sans aucune perte. Ces pipelines incluent les systèmes transactionnels et financiers, qui imposent des garanties de calcul et de cohérence des données.

    Les clients qui ont créé des services de workflow d’entreprise sont notamment [Zeiss Group](https://customers.microsoft.com/story/zeiss-group-focuses-on-azure-service-fabric-for-key-integration-platform), [Quorum Business Solutions](https://customers.microsoft.com/en-us/story/quorum-business-solutions-expand-energy-managemant-solutions-using-azure-service-fabric) et [Société Générale](https://customers.microsoft.com/en-us/story/societe-generale-speeds-real-time-market-quotes-using-azure-service-fabric).

* **Calcul sur des données** : Service Fabric vous permet de créer des applications avec état qui effectuent des calculs de données nécessitant une utilisation importante du processeur. Service Fabric permet la colocation du traitement (calcul) et des données dans les applications. 

   Normalement, lorsque votre application a besoin d’un accès aux données, une latence réseau associée à un niveau de stockage ou de cache de données externe limite le temps de calcul. Les services Service Fabric avec état éliminent cette latence, ce qui permet des lectures et écritures plus optimisées.

   Imaginez, par exemple, une application qui effectue des sélections de recommandations en temps quasi-réel pour des clients, avec un délai d’aller-retour de moins de 100 millisecondes. Les caractéristiques de latence et les performances des services Service Fabric offrent une expérience réactive à l’utilisateur par rapport au modèle d’implémentation standard consistant à extraire les données nécessaires depuis le stockage distant. Le système est plus réactif, car le calcul de sélection de la recommandation est colocalisé avec les données et les règles.

    Les clients qui ont créé des services de calcul sont notamment [Solidsoft Reply](https://customers.microsoft.com/story/solidsoft-reply-platform-powers-e-verification-of-pharmaceuticals) et [Infosupport](https://customers.microsoft.com/story/service-fabric-customer-profile-info-support-and-fudura).

* **Services hautement disponibles** : Service Fabric assure un basculement rapide en créant plusieurs réplicas de services secondaires. Si un nœud, un processus ou un service individuel tombe en panne en raison d’une défaillance matérielle ou autre, un des réplicas secondaires est promu réplica principal avec une perte de service minimale.

* **Services évolutifs** : les différents services peuvent être partitionnés pour permettre une montée en charge de l'état à l'ensemble du cluster. Des services individuels peuvent aussi être créés et supprimés à la volée. Vous pouvez effectuer un scale-out des services à partir de plusieurs instances sur quelques nœuds jusqu’à des milliers d’instances sur un grand nombre de nœuds, puis réexécuter un scale-in en fonction des besoins. Vous pouvez utiliser Service Fabric pour générer ces services et gérer l’ensemble de leur cycle de vie.

## <a name="application-design-case-studies"></a>Études de cas de conception d’applications

Des études de cas qui illustrent comment Service Fabric permet de concevoir des applications sont publiées sur les sites [Témoignages client](https://customers.microsoft.com/search?sq=%22Azure%20Service%20Fabric%22&ff=&p=2&so=story_publish_date%20desc) et [Microservices dans Azure](https://azure.microsoft.com/solutions/microservice-applications/).

## <a name="designing-applications-composed-of-stateless-and-stateful-microservices"></a>Conception d’applications composées de microservices avec et sans état

La création d’applications avec des rôles de travail Azure Cloud Services est un exemple de service sans état. À l’opposé, les microservices avec état maintiennent leur état faisant autorité au-delà de la demande et de la réponse correspondante. Cette fonctionnalité garantit une haute disponibilité et une cohérence de l’état grâce à des API simples qui fournissent des garanties transactionnelles soutenues par la réplication.

Les services avec état de Service Fabric apportent une haute disponibilité à tous les types d’applications, sans se limiter aux bases de données et autres magasins de données. Il s’agit d’une évolution naturelle. Les applications sont déjà passées d’une simple utilisation de bases de données purement relationnelles pour la haute disponibilité à l’emploi de bases de données NoSQL. Les applications elles-mêmes peuvent maintenant avoir un état « actif » et bénéficier d’une gestion des données en interne pour optimiser leurs performances, sans pour autant sacrifier la fiabilité, la cohérence ou la disponibilité des données.

Quand vous créez des applications composées de microservices, vous avez généralement une combinaison d’applications web sans état (ASP.NET, Node.js, etc.) qui appellent des services de couche intermédiaire métier sans et avec état. Les applications et les services sont tous déployés dans le même cluster Service Fabric au moyen des commandes de déploiement Service Fabric. Chacun de ces services est indépendant en ce qui concerne la mise à l’échelle, la fiabilité et l’utilisation des ressources. Cette indépendance améliore la souplesse du développement et de la gestion du cycle de vie.

Les microservices avec état simplifient les conceptions d’applications, car ils suppriment les caches et files d’attente supplémentaires, jusque-là indispensables pour répondre aux exigences de disponibilité et de latence d’une application purement sans état. Étant donné que les services avec état ont une haute disponibilité et une faible latence, il existe moins de détails à gérer dans votre application.

Les schémas suivants comparent la conception d’une application sans état à la conception d’une application avec état. Grâce aux modèles de programmation [Reliable Services](service-fabric-reliable-services-introduction.md) et [Reliable Actors](service-fabric-reliable-actors-introduction.md), les services avec état réduisent la complexité de l’application, tout en atteignant un débit élevé et une faible latence.

Voici un exemple d’application qui utilise les services sans état : ![Application qui utilise les services avec état][Image1]

Voici un exemple d’application qui utilise les services avec état : ![Application qui utilise les services avec état][Image2]

## <a name="next-steps"></a>Étapes suivantes

* Commencez à créer des services sans et avec état au moyen des modèles de programmation [Reliable Services](service-fabric-reliable-services-quick-start.md) et [Reliable Actors](service-fabric-reliable-actors-get-started.md) de Service Fabric.
* Accédez au Centre des architectures Azure pour obtenir des conseils sur la [création de microservices sur Azure](/azure/architecture/microservices/).
* Accédez aux [bonnes pratiques relatives aux applications et aux clusters Azure Service Fabric](service-fabric-best-practices-overview.md) pour obtenir des conseils sur la conception des applications.

* Voir aussi :
  * [Compréhension des microservices](service-fabric-overview-microservices.md)
  * [Définition et gestion de l’état du service](service-fabric-concepts-state.md)
  * [Disponibilité des services](service-fabric-availability-services.md)
  * [Mettre à l’échelle les services](service-fabric-concepts-scalability.md)
  * [Partitionner les services](service-fabric-concepts-partitioning.md)

[Image1]: media/service-fabric-application-scenarios/AppwithStatelessServices.png
[Image2]: media/service-fabric-application-scenarios/AppwithStatefulServices.png
