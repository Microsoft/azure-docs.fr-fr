---
title: Meilleures pratiques de conception d’application Azure Service Fabric
description: Bonnes pratiques et considérations relatives à la conception pour développer des applications et services à l’aide d’Azure Service Fabric.
ms.topic: conceptual
ms.date: 06/18/2019
ms.openlocfilehash: ddf846e9e3ac6add7cf3f584b702de5accfb22af
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91538496"
---
# <a name="azure-service-fabric-application-design-best-practices"></a>Meilleures pratiques de conception d’application Azure Service Fabric

Cet article fournit des conseils sur les bonnes pratiques de création d’applications et de services sur Azure Service Fabric.
 
## <a name="get-familiar-with-service-fabric"></a>Se familiariser avec Service Fabric
* Lisez l’article [Vous voulez en savoir plus sur Service Fabric ?](service-fabric-content-roadmap.md).
* Découvrez les [Scénarios d’applications Service Fabric](service-fabric-application-scenarios.md).
* Découvrez les choix de modèle de programmation en lisant la [Vue d’ensemble des modèles de programmation Service Fabric](service-fabric-choose-framework.md).



## <a name="application-design-guidance"></a>Guide de conception d’application
Familiarisez-vous avec l’[architecture générale](/azure/architecture/reference-architectures/microservices/service-fabric) des applications Service Fabric et leurs [considérations relatives à la conception](/azure/architecture/reference-architectures/microservices/service-fabric#design-considerations).

### <a name="choose-an-api-gateway"></a>Choisir une passerelle d’API
Utilisez un service de passerelle d’API qui communique avec les services principaux qui peuvent ensuite être montés en charge. Les services de passerelle d’API les plus couramment utilisés sont :

- [Gestion des API Azure](./service-fabric-api-management-overview.md), qui est [intégrée à Service Fabric](./service-fabric-tutorial-deploy-api-management.md).
- [Azure IoT Hub](../iot-hub/index.yml) ou [Azure Event Hubs](../event-hubs/index.yml) à l’aide de [ServiceFabricProcessor](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/eventhub/Microsoft.Azure.EventHubs.ServiceFabricProcessor) pour lire à partir de partitions Event Hub.
- [Proxy inverse Træfik](https://techcommunity.microsoft.com/t5/azure-service-fabric/bg-p/Service-Fabric) à l’aide du [fournisseur Azure Service Fabric](https://docs.traefik.io/v1.6/configuration/backends/servicefabric/).
- [Azure Application Gateway](../application-gateway/index.yml) :

   > [!NOTE] 
   > Azure Application Gateway n’est pas directement intégré à Service Fabric. Gestion des API Azure est généralement le choix privilégié.
- Créez votre propre passerelle d’application web [ASP.NET Core](./service-fabric-reliable-services-communication-aspnetcore.md) personnalisée.

### <a name="stateless-services"></a>Service sans état
Nous vous recommandons de toujours commencer par créer des services sans état à l’aide de [Reliable Services](./service-fabric-reliable-services-introduction.md), et de stocker l’état dans une base de données Azure, Azure CosmosDB ou Stockage Azure. L’état externalisé est l’approche la plus familière pour la plupart des développeurs. Cette approche vous permet également de tirer parti des fonctionnalités de requête sur le magasin.  

### <a name="when-to-use-stateful-services"></a>Quand utiliser des services avec état ?
Envisagez des services avec état lorsque votre scénario implique une faible latence et que vous devez conserver les données à proximité des ressources de calcul. Il peut s’agir de scénarios tels que des appareils jumeaux numériques IoT, l’état du jeu, l’état de la session, la mise en cache de données à partir d’une base de données et les flux de travail durables pour suivre les appels d’autres services.

Choisissez la période de conservation des données :

- **Données mises en cache**. Utilisez la mise en cache quand la latence vers des magasins externes est un problème. Utilisez un service avec état comme cache de données ou envisagez d’utiliser le [cache distribué SoCreate Service Fabric open source](https://github.com/SoCreate/service-fabric-distributed-cache). Dans ce scénario, vous n’avez pas à vous inquiéter de perdre toutes les données dans le cache.
- **Données associées à un délai**. Dans ce scénario, vous devez conserver les données à proximité des ressources de calcul pendant un certain délai pour des raisons de latence, mais vous pouvez vous permettre de perdre ces données en cas de *sinistre*. Par exemple, dans de nombreuses solutions IoT, les données doivent être proches des ressources de calcul (par exemple en cas de calcul de la température moyenne sur les derniers jours), mais si ces données sont perdues, les points de données spécifiques enregistrés ne sont pas si importants. En outre, dans ce scénario vous vous souciez généralement peu de la sauvegarde de points de données individuels. Vous sauvegardez uniquement les valeurs moyennes calculées qui sont régulièrement écrites vers le stockage externe.  
- **Données à long terme**. Des collections fiables peuvent stocker vos données de manière permanente. Dans ce cas, vous devez toutefois vous [préparer à la reprise d’activité après sinistre](./service-fabric-disaster-recovery.md), notamment à la [configuration de stratégies de sauvegarde périodiques](./service-fabric-backuprestoreservice-configure-periodic-backup.md) pour vos clusters. Dans les faits, vous configurez ce qu’il se passe si votre cluster est détruit lors d’un sinistre, auquel cas vous devez créer un nouveau cluster, et comment déployer de nouvelles instances d’application et récupérer à partir de la dernière sauvegarde.

Réduire les coûts et améliorer la disponibilité :
- Vous pouvez réduire les coûts grâce à des services avec état, car vous n’engagez pas de coûts d’accès aux données et de transactions à partir du magasin distant, et car vous n’avez pas besoin d’utiliser un autre service, tel que le cache Azure pour Redis.
- L’utilisation de services avec état principalement pour le stockage et non pour le calcul est coûteuse, et nous la déconseillons. Vous pouvez considérer les services avec état comme du calcul avec un stockage local économique.
- En supprimant les dépendances à d’autres services, vous pouvez améliorer la disponibilité de votre service. La gestion de l’état avec haute disponibilité dans le cluster permet de vous isoler d’autres temps d’arrêt du service ou problèmes de latence.

## <a name="how-to-work-with-reliable-services"></a>Comment utiliser Reliable Services
Service Fabric Reliable Services vous permet de créer facilement des services sans et avec état. Pour plus d’informations, consultez l’[introduction à Reliable Services](./service-fabric-reliable-services-introduction.md).
- Utilisez toujours le [jeton d’annulation](./service-fabric-reliable-services-lifecycle.md#stateful-service-primary-swaps) de la méthode `RunAsync()` pour les services sans et avec état et de la méthode `ChangeRole()` pour les services avec état. Sans cela, Service Fabric ne sait pas si votre service peut être fermé. Le fait de ne pas utiliser le jeton d’annulation peut, par exemple, entraîner des temps de mise à niveau d’applications beaucoup plus longs.
-    Ouvrez et fermez les [écouteurs de communications](./service-fabric-reliable-services-communication.md) aux moments opportuns et utilisez les jetons d’annulation.
-    Ne combinez jamais du code synchrone et du code asynchrone. Par exemple, n’utilisez pas `.GetAwaiter().GetResult()` dans vos appels asynchrones. Utiliser async *à travers toute* la pile des appels.

## <a name="how-to-work-with-reliable-actors"></a>Comment utiliser Reliable Actors
Service Fabric Reliable Actors vous permet de créer facilement des acteurs virtuels avec état. Pour plus d’informations, consultez l’[introduction à Reliable Actors](./service-fabric-reliable-actors-introduction.md).

- L’utilisation de la messagerie de publication/d’abonnement entre vos acteurs pour la mise à l’échelle de l’application est vivement recommandée. Parmi les outils qui offrent ce service, citons l’[open-source SoCreate Service Fabric Pub/Sub](https://service-fabric-pub-sub.socreate.it/) et [Azure Service Bus](/azure/service-bus/).
- Rendez l’état d’acteur aussi [précis que possible](./service-fabric-reliable-actors-state-management.md#best-practices).
- Gérez le [cycle de vie de l’acteur](./service-fabric-reliable-actors-state-management.md#best-practices). Supprimez des acteurs si vous ne prévoyez pas de les réutiliser. La suppression des acteurs inutiles est particulièrement importante quand vous utilisez le [fournisseur d’état volatile](./service-fabric-reliable-actors-state-management.md#state-persistence-and-replication), car tout l’état est stocké en mémoire.
- En raison de leur [concurrence alternée](./service-fabric-reliable-actors-introduction.md#concurrency), il est préférable d’utiliser les acteurs comme des objets indépendants. Ne créez pas de graphes d’appels de méthode synchrones à plusieurs acteurs (chacun d’entre eux constituera probablement un appel réseau distinct) ni de requêtes d’acteur circulaires. Celles-ci affectent considérablement les performances et la mise à l’échelle.
- Ne combinez pas du code synchrone et du code asynchrone. Utilisez async de manière cohérente afin d’éviter les problèmes de performances.
- N’effectuez pas d’appels longs dans des acteurs. Les appels longs bloquent les autres appels vers le même acteur à cause de la concurrence alternée.
- Si vous communiquez avec d’autres services à l’aide de la [communication à distance Service Fabric](./service-fabric-reliable-services-communication-remoting.md) et que vous créez un `ServiceProxyFactory`, créez la fabrique au niveau du [service d’acteur](./service-fabric-reliable-actors-using.md) et *non* au niveau de l’acteur.


## <a name="application-diagnostics"></a>Diagnostic d'application
Veillez à ajouter la [journalisation des applications](./service-fabric-diagnostics-event-generation-app.md) dans les appels de service. Cela vous aidera à diagnostiquer les scénarios dans lesquels les services s’appellent mutuellement. Par exemple, quand A appelle B appelle C appelle D, l’appel peut échouer n’importe où. Si vous n’avez pas suffisamment de journalisation, les échecs sont difficiles à diagnostiquer. Si la journalisation des services est trop importante en raison des volumes d’appel, veillez à enregistrer au minimum les erreurs et avertissements.

## <a name="iot-and-messaging-applications"></a>Applications IoT et de messagerie
Quand vous lisez des messages à partir d’[Azure IoT Hub](../iot-hub/index.yml) ou d’[Azure Event Hubs](../event-hubs/index.yml), utilisez [ServiceFabricProcessor](https://github.com/Azure/azure-event-hubs/tree/master/samples/DotNet/Microsoft.Azure.EventHubs/ServiceFabricProcessor). ServiceFabricProcessor s’intègre à Service Fabric Reliable Services pour maintenir l’état de lecture des partitions de hub d’événements, et envoie (push) les nouveaux messages à vos services par le biais de la méthode `IEventProcessor::ProcessEventsAsync()`.


## <a name="design-guidance-on-azure"></a>Guide de conception sur Azure
* Accédez au [centre des architectures Azure](/azure/architecture/microservices/) pour obtenir un guide sur la [création de microservices sur Azure](/azure/architecture/microservices/).

* Consultez la section de [prise en main Azure pour les jeux](/gaming/azure/) pour obtenir un guide sur l’[utilisation de Service Fabric dans des services de gaming](/gaming/azure/reference-architectures/multiplayer-synchronous-sf).
