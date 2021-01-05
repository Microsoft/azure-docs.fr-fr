---
title: Niveaux de service Premium et Standard d’Azure Service Bus
description: Cet article décrit les niveaux de service Standard et Premium d’Azure Service Bus. Compare ces niveaux de service et indique les différences techniques.
ms.topic: conceptual
ms.date: 07/28/2020
ms.openlocfilehash: 31c53a1375078cd5d185945cba55a6e5a6dd5ffb
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90966795"
---
# <a name="service-bus-premium-and-standard-messaging-tiers"></a>Couches messagerie Service Bus Premium et Standard

La messagerie Service Bus, qui comprend des entités telles que les files d’attente et les rubriques, associe des fonctionnalités de messagerie d’entreprise à une sémantique riche de publication et d’abonnement à l’échelle du cloud. La messagerie Service Bus sert d’infrastructure de communication à de nombreuses solutions cloud sophistiquées.

La couche *Premium* de la messagerie Service Bus répond aux demandes couramment formulées par les clients concernant la mise à l’échelle, les performances et la disponibilité de leurs applications critiques. Le niveau Premium est recommandé pour les scénarios de production. Bien qu’offrant des ensembles de fonctionnalités quasiment identiques, les deux couches de messagerie Service Bus sont conçues pour différents cas de figure.

Les principales différences sont répertoriées dans le tableau suivant.

| Premium | standard |
| --- | --- |
| Débit élevé |Débit variable |
| Performances prévisibles |Latence variable |
| Prix fixe |Tarification à l’utilisation variable |
| Possibilité de faire évoluer la charge de travail |N/A |
| Taille de message maximale de 1 Mo. Cette limite pourrait être augmentée à l’avenir. Pour obtenir les dernières mises à jour importantes du service, consultez le [blog Messaging on Azure](https://techcommunity.microsoft.com/t5/messaging-on-azure/bg-p/MessagingonAzureBlog). |Taille de message maximale de 256 Ko |

La **messagerie Service Bus Premium** isole les ressources au niveau processeur et mémoire, ce qui permet d’exécuter chaque charge de travail client de manière isolée. Ce conteneur de ressources est appelé *unité de messagerie*. Au moins une unité de messagerie est allouée à chaque espace de noms premium. Vous pouvez acheter 1, 2, 4 ou 8 unités de messagerie pour chaque espace de noms Service Bus Premium. Une entité ou une charge de travail unique peut couvrir plusieurs unités de messagerie et le nombre d’unités de messagerie peut être modifié à volonté. Au final, les performances de votre solution Service Bus sont prévisibles et répétables.

Au final, les performances de votre solution Service Bus sont non seulement prévisibles et répétables, mais aussi supérieures. La messagerie Service Bus Premium s’appuie sur le moteur de stockage introduit dans [Azure Event Hubs](https://azure.microsoft.com/services/event-hubs/). La messagerie Premium offre des performances de pointe bien supérieures à celles de la couche Standard.

## <a name="premium-messaging-technical-differences"></a>Différences techniques de la messagerie Premium

Les sections suivantes décrivent les quelques différences entre les couches de messagerie Standard et Premium.

### <a name="partitioned-queues-and-topics"></a>Files d’attente et rubriques partitionnées

La messagerie Premium-Messaging ne prend pas en charge les files d’attente et rubriques partitionnées. Pour plus d’informations sur le partitionnement, voir [Files d’attentes et rubriques partitionnées](service-bus-partitioning.md).

### <a name="express-entities"></a>Entités Express

La messagerie Premium-Messaging s’exécutant dans un environnement d’exécution isolé, les entités express ne sont pas prises en charge dans les espaces de noms Premium. Pour en savoir plus sur la fonctionnalité express, consultez la propriété [QueueDescription.EnableExpress](/dotnet/api/microsoft.servicebus.messaging.queuedescription.enableexpress#Microsoft_ServiceBus_Messaging_QueueDescription_EnableExpress).

Si vous avez du code s’exécutant dans la messagerie Standard et que vous souhaitez le faire passer au niveau Premium, vérifiez que la propriété [EnableExpress](/dotnet/api/microsoft.servicebus.messaging.queuedescription.enableexpress#Microsoft_ServiceBus_Messaging_QueueDescription_EnableExpress) est définie sur **false** (la valeur par défaut).

## <a name="premium-messaging-resource-usage"></a>Utilisation des ressources Premium Messaging
En règle générale, toute opération sur une entité peut entraîner l’utilisation du processeur et de la mémoire. Voici quelques-unes de ces opérations : 

- Les opérations de gestion comme les opérations CRUD (Create, Retrieve, Update et Delete) sur les files d’attente, les sujets et les abonnements.
- Les opérations d’exécution (envoi et réception de messages)
- Les opérations de surveillance et d’alerte

L’utilisation supplémentaire de mémoire et de processeur n’est cependant pas facturée. Pour la messagerie Premium Messaging, il existe un prix unique par unité de message.

L’utilisation du processeur et de la mémoire est suivie et affichée pour les raisons suivantes : 

- Transparence sur le fonctionnement interne du système
- Capacité des ressources achetées.
- Planification de la capacité vous décider de monter en puissance ou non.

## <a name="messaging-unit---how-many-are-needed"></a>Unité de messagerie – Combien sont nécessaires ?

Lors du provisionnement d’un espace de noms Azure Service Bus Premium, le nombre d’unités de messagerie allouées doit être spécifié. Ces unités de messagerie sont des ressources dédiées qui sont allouées à l’espace de noms.

Le nombre d’unités de messagerie allouées à l’espace de noms Service Bus Premium peut être **ajusté dynamiquement** pour prendre en compte la variation (augmentation ou diminution) des charges de travail.

Un certain nombre de facteurs doivent être pris en compte pour déterminer le nombre d’unités de messagerie pour votre architecture :

- Commencez par ***1 ou 2 unités de messagerie*** allouées à votre espace de noms.
- Étudiez les métriques d’utilisation du processeur dans les [métriques d’utilisation des ressources](service-bus-metrics-azure-monitor.md#resource-usage-metrics) pour votre espace de noms.
    - Si l’utilisation du processeur est ***inférieure à 20 %***, vous pourrez peut-être ***réduire*** le nombre d’unités de messagerie allouées à votre espace de noms.
    - Si l’utilisation du processeur est ***supérieure à 70 %***, votre application profitera d’une ***augmentation*** du nombre d’unités de messagerie allouées à votre espace de noms.

Pour configurer un espace de noms Service Bus afin qu'il se mette automatiquement à l'échelle (augmentation ou diminution des unités de messagerie), consultez [Mettre automatiquement à jour les unités de messagerie](automate-update-messaging-units.md).

> [!NOTE]
> La **mise à l’échelle** des ressources allouées à l’espace de noms peut être préemptive ou réactive.
>
>  * **Préemptive** : si une charge de travail supplémentaire est attendue (en raison de la saisonnalité ou des tendances), vous pouvez allouer plus d’unités de messagerie à l’espace de noms avant la hausse des charges de travail.
>
>  * **Réactive** : si des charges de travail supplémentaires sont identifiées en examinant les métriques d’utilisation des ressources, des ressources supplémentaires peuvent être allouées à l’espace de noms pour incorporer la hausse de la demande.
>
> Les compteurs de facturation pour Service Bus sont horaires. Dans le cas d’un scale-up, vous payez uniquement pour les ressources supplémentaires pendant les heures où celles-ci sont utilisées.
>

## <a name="get-started-with-premium-messaging"></a>Prise en main de Premium Messaging

La prise en main de Premium Messaging est immédiate ; le processus est similaire à celui de la messagerie Standard. Commencez par [créer un espace de noms](service-bus-create-namespace-portal.md) dans le [portail Azure](https://portal.azure.com). Veillez à sélectionner **Premium** sous **Niveau de tarification**. Cliquez sur **Afficher tous les détails de la tarification** pour voir plus d’informations sur chaque niveau.

![create-premium-namespace][create-premium-namespace]

Vous pouvez également créer des [espaces de noms Premium à l’aide de modèles Azure Resource Manager](https://azure.microsoft.com/resources/templates/101-servicebus-pn-ar/).

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la messagerie Service Bus, consultez les liens suivants :

- [Mettre automatiquement à jour les unités de messagerie](automate-update-messaging-units.md)
- [Introducing Azure Service Bus Premium Messaging (Présentation de la messagerie Azure Service Bus Premium (billet de blog))](https://azure.microsoft.com/blog/introducing-azure-service-bus-premium-messaging/)
- [Introducing Azure Service Bus Premium Messaging (Présentation de la messagerie Azure Service Bus Premium (Channel9))](https://channel9.msdn.com/Blogs/Subscribe/Introducing-Azure-Service-Bus-Premium-Messaging)

<!--Image references-->

[create-premium-namespace]: ./media/service-bus-premium-messaging/select-premium-tier.png
