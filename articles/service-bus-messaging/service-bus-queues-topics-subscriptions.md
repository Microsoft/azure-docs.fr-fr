---
title: Messagerie Azure Service Bus - files d’attente, rubriques et abonnements
description: Cet article fournit une vue d’ensemble des entités de messagerie Azure Service Bus (file d’attente, rubriques et abonnements).
ms.topic: article
ms.date: 11/04/2020
ms.openlocfilehash: 54b6a1fd2d4e8e5ef5bb6522374646257213e4b4
ms.sourcegitcommit: 6a770fc07237f02bea8cc463f3d8cc5c246d7c65
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/24/2020
ms.locfileid: "95791604"
---
# <a name="service-bus-queues-topics-and-subscriptions"></a>Files d’attente, rubriques et abonnements Service Bus
Azure Service Bus prend en charge un ensemble de technologies interlogicielles Cloud orientées messages, notamment une mise en file d'attente des messages fiable et une messagerie de publication/abonnement durable. Ces capacités de messagerie répartie peuvent être considérées comme des fonctionnalités de messagerie découplée prenant en charge la publication-abonnement, le découplage temporel et les scénarios d’équilibrage de charge qui utilisent la charge de travail de messagerie Service Bus. La communication découplée présente de nombreux avantages. Par exemple, les clients et les serveurs peuvent se connecter en fonction des besoins et exécuter leurs opérations de manière asynchrone.

Les entités de messagerie qui constituent l’essentiel des capacités de messagerie dans Service Bus sont **les files d’attente**, **les rubriques et abonnements**, les règles et les actions.

## <a name="queues"></a>Files d’attente
Les files d’attente permettent la livraison de messages sur le principe du **premier entré, premier sorti (FIFO)** à un ou plusieurs destinataires concurrents. Autrement dit, les messages sont en général reçus et traités par les destinataires dans l’ordre dans lequel ils ont été ajoutés à la file d’attente. Chaque message est reçu et traité par un seul consommateur de message. Un des principaux avantages de l’utilisation des files d’attente consiste à réaliser le **découplage temporel des composants d’application**. En d’autres termes, les producteurs (expéditeurs) et les consommateurs (destinataires) n’ont pas besoin d’envoyer et de recevoir des messages en même temps. Cela est dû au fait que les messages sont stockés durablement dans la file d’attente. En outre, le producteur n’a pas à attendre de réponse du consommateur pour continuer de traiter et d’envoyer d’autres messages.

Un des avantages associés est le **nivellement de charge**, qui permet aux producteurs et aux consommateurs d’envoyer et de recevoir des messages à des vitesses différentes. Dans de nombreuses applications, la charge système varie au fil du temps. Toutefois, le temps de traitement requis pour chaque unité de travail est généralement constant. L’ajout d’une file d’attente entre les producteurs et les consommateurs des messages fait que l’application de destination doit être en mesure de gérer seulement une charge moyenne, plutôt que la charge de travail maximale. La file d’attente s’allonge et se raccourcit en fonction de la charge entrante. Cette fonctionnalité permet de faire des économies concernant les infrastructures nécessaires pour faire face à la charge de travail de l’application. À mesure que la charge augmente, d’autres processus de travail peuvent être ajoutés pour lire les éléments de la file d’attente. Chaque message est traité par un seul des processus de travail. De plus, cet équilibrage de la charge basé sur le tirage (pull) permet d’optimiser l’utilisation des ordinateurs de travail, même si les ordinateurs de travail dotés d’une puissance de traitement tirent les messages au maximum de leur capacité. Ce modèle est souvent appelé modèle **consommateur concurrent**.

L’utilisation de files d’attente comme intermédiaire entre les producteurs et les consommateurs de message fournit un couplage souple inhérent entre les composants. Producteurs et consommateurs étant indépendants les uns des autres, il est possible de mettre à niveau un consommateur sans que cela affecte le producteur.

### <a name="create-queues"></a>Créer des files d’attente
Vous pouvez créer des files d’attente en utilisant le [portail Azure](service-bus-quickstart-portal.md), [PowerShell](service-bus-quickstart-powershell.md), [l’interface CLI](service-bus-quickstart-cli.md) ou les [modèles Resource Manager](service-bus-resource-manager-namespace-queue.md). Ensuite, envoyez et recevez des messages à l’aide de clients écrits en [C#](service-bus-dotnet-get-started-with-queues.md), [Java](service-bus-java-how-to-use-queues.md), [Python](service-bus-python-how-to-use-queues.md), [JavaScript](service-bus-nodejs-how-to-use-queues.md), [PHP](service-bus-php-how-to-use-queues.md) et [Ruby](service-bus-ruby-how-to-use-queues.md). 

### <a name="receive-modes"></a>Modes de réception
Service Bus propose deux modes différents de réception des messages : **ReceiveAndDelete** et **PeekLock**. En mode [ReceiveAndDelete](/dotnet/api/microsoft.azure.servicebus.receivemode), lorsque Service Bus reçoit la demande du consommateur, il marque ce message comme étant consommé et le renvoie à l’application du consommateur. Ce mode est le modèle le plus simple. Il fonctionne mieux pour les scénarios dans lesquels l’application peut tolérer l’absence de traitement d’un message en cas d’échec. Pour comprendre ce comportement, imaginez un scénario selon lequel le consommateur émet la demande de réception et subit un incident avant de la traiter. Comme Service Bus marque le message comme étant consommé, l’application commence à consommer des messages lors du redémarrage. Elle manquera le message qu’elle a consommé avant l’incident.

En mode [PeekLock](/dotnet/api/microsoft.azure.servicebus.receivemode), la réception devient une opération en deux étapes, qui autorise une prise en charge des applications qui ne peuvent pas se permettre de manquer des messages. Lorsque Service Bus reçoit la demande, il effectue les opérations suivantes :

1. Recherche le message suivant à consommer.
1. Le verrouille pour empêcher d’autres consommateurs de le recevoir.
1. Ensuite, retourne le message à l’application. 

Une fois que l’application a terminé de traiter le message ou qu’elle l’a stocké de façon fiable pour un traitement ultérieur, elle exécute la deuxième étape du processus de réception en appelant [`CompleteAsync`](/dotnet/api/microsoft.azure.servicebus.queueclient.completeasync) sur le message. Lorsque Service Bus reçoit la requête **CompleteAsync**, il marque le message comme étant consommé.

Si l’application ne peut pas traiter le message pour une raison quelconque, elle peut appeler la méthode [`AbandonAsync`](/dotnet/api/microsoft.azure.servicebus.queueclient.abandonasync) sur le message reçu (au lieu de [`CompleteAsync`](/dotnet/api/microsoft.azure.servicebus.queueclient.completeasync)). Cette méthode permet à Service Bus de déverrouiller le message et de faire en sorte qu’il puisse être à nouveau reçu, soit par le même consommateur, soit par un consommateur concurrent. Ensuite, il y a un délai d’attente associé au verrou. Si l’application ne parvient pas à traiter le message dans le temps imparti, Service Bus déverrouille le message et le rend à nouveau disponible en réception.

Si l’application subit un incident après le traitement du message, mais avant qu’elle appelle [`CompleteAsync`](/dotnet/api/microsoft.azure.servicebus.queueclient.completeasync), Service Bus livre à nouveau le message à l’application lorsqu’elle redémarre. Ce processus est souvent appelé le traitement **Au moins une fois**. Autrement dit, chaque message est traité au moins une fois. Toutefois, dans certaines circonstances, un même message peut être remis une nouvelle fois. Si votre scénario ne peut pas tolérer un traitement dupliqué, ajoutez une logique supplémentaire dans votre application pour détecter les doublons. Pour ce faire, vous pouvez utiliser la propriété [MessageId](/dotnet/api/microsoft.azure.servicebus.message.messageid) du message, qui reste constante entre les tentatives de remise. Cette fonctionnalité est connue sous le nom de traitement **une seule fois**.

## <a name="topics-and-subscriptions"></a>Rubriques et abonnements
Une file d’attente autorise le traitement d’un message par un seul consommateur. Contrairement aux files d’attente, les rubriques et les abonnements fournissent une forme de communication de type un-à-plusieurs dans un modèle de **publication et abonnement**. Cela est utile pour la mise à l’échelle d’un grand nombre de destinataires. Chaque message publié est mis à disposition de tous les abonnements inscrits auprès de la rubrique. L’éditeur envoie un message à une rubrique et un ou plusieurs abonnés reçoivent une copie du message, en fonction des règles de filtre définies pour ces abonnements. Les abonnements peuvent utiliser des filtres supplémentaires pour restreindre les messages qu’ils souhaitent recevoir. Les éditeurs envoient des messages à une rubrique de la même façon qu’ils envoient des messages à une file d’attente. Toutefois, les consommateurs ne reçoivent pas de messages directement de la rubrique. Au lieu de cela, les consommateurs reçoivent des messages des abonnements de la rubrique. Un abonnement à une rubrique ressemble à une file d’attente virtuelle recevant des copies des messages envoyés à la rubrique. Les consommateurs reçoivent des messages d’un abonnement de la même manière qu’ils reçoivent des messages d’une file d’attente.

La fonctionnalité d’envoi de messages d’une file d’attente mappe directement sur une rubrique, et sa fonctionnalité de réception de messages est mappé sur un abonnement. Cette fonctionnalité implique, entre autres, que les abonnements prennent en charge les modèles décrits plus haut dans cette section concernant les files d’attente : consommateurs simultanés, découplage temporel, nivellement et équilibrage de charge.

### <a name="create-topics-and-subscriptions"></a>Créer des rubriques et des abonnements
La création d’une rubrique est similaire à la création d’une file d’attente, comme le montre la section précédente. Vous pouvez créer des rubriques et des abonnements en utilisant le [portail Azure](service-bus-quickstart-topics-subscriptions-portal.md), [PowerShell](service-bus-quickstart-powershell.md), [l’interface CLI](service-bus-tutorial-topics-subscriptions-cli.md) ou les [modèles Resource Manager](service-bus-resource-manager-namespace-topic.md). Ensuite, envoyez des messages à une rubrique et recevez des messages d’abonnements à l’aide de clients écrits en [C#](service-bus-dotnet-how-to-use-topics-subscriptions.md), [Java](service-bus-java-how-to-use-topics-subscriptions.md), [Python](service-bus-python-how-to-use-topics-subscriptions.md), [JavaScript](service-bus-nodejs-how-to-use-topics-subscriptions.md), [PHP](service-bus-php-how-to-use-topics-subscriptions.md) et [Ruby](service-bus-ruby-how-to-use-topics-subscriptions.md). 

### <a name="rules-and-actions"></a>Règles et actions
Dans de nombreux scénarios, les messages ayant des caractéristiques spécifiques doivent être traités différemment. Pour mettre en place ce traitement, vous pouvez configurer des abonnements de sorte qu’ils recherchent les messages présentant les propriétés souhaitées, puis apporter certaines modifications à ces propriétés. Bien que les abonnements Service Bus voient tous les messages envoyés à la rubrique, vous pouvez uniquement copier un sous-ensemble de ces messages dans la file d’attente d’abonnement virtuelle. On utilise pour cela des filtres d’abonnement. Ces modifications sont appelées **actions de filtrage**. Lorsqu’un abonnement est créé, vous pouvez fournir une expression de filtre qui opère sur les propriétés du message. Les propriétés peuvent être à la fois des propriétés système (par exemple, **Étiquette**) et des propriétés d’application personnalisées (par exemple, **StoreName**). L’expression de filtre SQL est facultative dans ce cas. Sans expression de filtre SQL, toute action de filtre définie sur un abonnement sera effectuée sur tous les messages de cet abonnement.

Pour un exemple complet et concret, voir [TopicSubscriptionWithRuleOperationsSample](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/GettingStarted/Microsoft.Azure.ServiceBus/TopicSubscriptionWithRuleOperationsSample) sur GitHub.

Pour plus d’informations sur les valeurs de filtre possibles, consultez la documentation des classes [SqlFilter](/dotnet/api/microsoft.azure.servicebus.sqlfilter) et [SqlRuleAction](/dotnet/api/microsoft.azure.servicebus.sqlruleaction).

## <a name="java-message-service-jms-20-entities-preview"></a>Entités de Java Message Service (JMS) 2.0 (préversion)
Les entités suivantes sont accessibles via l’API Java Message Service (JMS) 2.0.

  * Files d’attente temporaires
  * Rubriques temporaires
  * Abonnements durables partagés
  * Abonnements durables non partagés
  * Abonnements non durables partagés
  * Abonnements non durables non partagés

En savoir plus sur les [entités JMS 2.0](java-message-service-20-entities.md) et sur la façon de [les utiliser](how-to-use-java-message-service-20.md).

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations et des exemples d’utilisation de la messagerie Service Bus, consultez les rubriques avancées suivantes :

* [Présentation de la messagerie Service Bus](service-bus-messaging-overview.md)
* [Démarrage rapide : Envoyer et recevoir des messages à l’aide du portail Azure et de .NET](service-bus-quickstart-portal.md)
* [Tutoriel : Mettre à jour l’inventaire à l’aide du portail Azure et des rubriques/abonnements](service-bus-tutorial-topics-subscriptions-portal.md)


