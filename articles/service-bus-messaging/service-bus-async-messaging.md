---
title: Messagerie asynchrone Service Bus | Microsoft Docs
description: Découvrez comment Azure Service Bus prend en charge l’asynchronisme en utilisant un mécanisme d’envoi en différé avec des files d’attente, des rubriques et des abonnements.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: 0ea2a2f51d29f1a65926509581df49b453bde59f
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "88067543"
---
# <a name="asynchronous-messaging-patterns-and-high-availability"></a>Modèles de messagerie asynchrone et haute disponibilité

Une messagerie asynchrone peut être mise en œuvre de différentes façons. Avec les files d’attente, rubriques et abonnements, Azure Service Bus prend en charge des opérations asynchrones en utilisant un mécanisme de stockage et de transfert. En cas de fonctionnement normal (synchrone), des messages sont envoyés à des files d’attente et des rubriques et vous recevez des messages de la part de files d’attente et d’abonnements. Les applications que vous rédigez dépendent de la disponibilité permanente de ces entités. Lorsque, suite à certaines circonstances, l’état de l’entité change, vous devez disposer d’une entité à fonctionnalités réduites pouvant satisfaire la plupart des besoins.

Les applications utilisent en général des modèles de messagerie asynchrones autorisant un certain nombre de scénarios de communication. Vous pouvez créer des applications dans lesquelles les clients peuvent envoyer des messages aux services, même si le service n’est pas en cours d’exécution. Pour les applications soumises à des pics de communication, une file d’attente peut permettre de répartir la charge en fournissant un emplacement pour mettre les communications en mémoire tampon. Enfin, vous avez accès à un équilibreur de charge simple mais efficace pour distribuer les messages sur plusieurs ordinateurs.

Pour maintenir la disponibilité de toutes ces entités, vous devez trouver des moyens de faire apparaître ces entités comme indisponibles pour un système de messagerie durable. En règle générale, l’entité devient indisponible pour les applications que nous écrivons pour différentes raisons :

* Impossible d’envoyer des messages
* Impossible de recevoir des messages
* Impossible de gérer (créer, récupérer, mettre à jour ou supprimer) des entités.
* Impossible de contacter le service.

Pour chacune de ces erreurs, différents modes d’échec existent et permettent à une application de continuer de fonctionner à un niveau de fonctionnalités limité. Par exemple, un système qui peut envoyer des messages mais pas de les recevoir peut encore recevoir des commandes des clients, mais ne peut pas traiter ces commandes. Cette rubrique aborde les problèmes qui peuvent se présenter et explique comment y remédier. Service Bus présente un certain nombre de mesures d’atténuation à votre disposition, et cette rubrique traite également des règles régissant l’utilisation de ces actions.

## <a name="reliability-in-service-bus"></a>Fiabilité de Service Bus
Il existe plusieurs méthodes permettant de gérer les problèmes de messagerie et d’entités et certaines instructions régissent l’utilisation appropriée de ces actions de correction. Pour comprendre ces instructions, vous devez d’abord connaître les défaillances susceptibles de se produire dans Service Bus. En raison de la conception des systèmes Azure, tous ces problèmes sont en général éphémères. À un niveau élevé, les différentes causes d’indisponibilité sont les suivantes :

* Limitation d’un système externe dont Service Bus dépend. La limitation se produit à la suite d’interactions avec les ressources de stockage et de calcul.
* Problème d’un système dont Service Bus dépend. Par exemple, une partie donnée du stockage peut rencontrer des problèmes.
* Défaillance de Service Bus dans un sous-système unique Dans ce cas, un nœud de calcul peut passer à l’état incohérent et redémarrer seul, ce qui fait que toutes les entités desservies doivent équilibrer leur charge sur d’autres nœuds. Pendant un bref laps de temps, le traitement des messages peut se trouver ralenti.
* Échec de Service Bus dans un centre de données Azure. Il s’agit d’une « défaillance irrémédiable », et le système reste inaccessible pendant plusieurs minutes, voire quelques heures.

> [!NOTE]
> Le terme **stockage** peut désigner le stockage Azure et SQL Azure.
> 
> 

Service Bus propose un certain nombre de mesures d’atténuation des problèmes mentionnés ci-dessus. Les sections suivantes décrivent chaque problème et les actions correctives correspondantes.

### <a name="throttling"></a>Limitation
Avec Service Bus, la limitation permet une gestion coopérative de la vitesse des messages. Chaque nœud Service Bus héberge de nombreuses entités. Chacune de ces entités effectue des demandes sur le système en termes de processeur, de mémoire, de stockage et autres. Lorsqu’une de ces facettes détecte une utilisation dépassant les seuils définis, Service Bus peut refuser une demande donnée. L’appelant reçoit alors une exception [ServerBusyException][ServerBusyException] et refait une tentative après 10 secondes.

Pour remédier au problème, le code doit lire l’erreur et stopper toute nouvelle tentative du message pendant au moins 10 secondes. Dans la mesure où l’erreur peut se produire sur différents éléments de l’application client, chaque élément doit exécuter la nouvelle tentative de manière indépendante. Le code peut réduire la probabilité de limitation en activant le partitionnement sur une file d’attente ou une rubrique.

### <a name="issue-for-an-azure-dependency"></a>Problème sur une dépendance Azure
D’autres composants dans Azure peuvent parfois rencontrer des problèmes de service. Par exemple, lorsqu’un système Service Bus est en cours de mise à niveau, ce système peut provisoirement présenter des fonctionnalités réduites. Pour résoudre ces types de problèmes, Service Bus procède régulièrement à des analyses et met en œuvre les solutions de correction. Des effets secondaires de ces mesures de correction peuvent apparaître. Par exemple, pour gérer les problèmes temporaires liés au stockage, Service Bus fait appel à un système qui permet aux opérations d’envoi de message de continuer de fonctionner manière cohérente. En raison de la nature de cette mesure de correction, un message envoyé peut prendre jusqu’à 15 minutes pour apparaître dans la file d’attente ou l’abonnement affecté et être prêt pour une opération de réception. En règle générale, la plupart des entités ne rencontrent pas ce problème. Toutefois, étant donné le nombre d’entités présentes dans Service Bus au sein d’Azure, cette opération est parfois nécessaire pour un petit sous-ensemble de clients de Service Bus.

### <a name="service-bus-failure-on-a-single-subsystem"></a>Défaillance Service Bus dans un sous-système unique
Quelle que soit l’application, les circonstances peuvent générer une défaillance chez un composant interne de Service Bus. Lorsque Service Bus détecte cela, il rassemble les données de l’application pour vous aider à diagnostiquer ce qui est arrivé. Une fois les données collectées, l’application est redémarrée afin de revenir à un état cohérent. Ce processus est assez rapidement et peut faire en sorte qu’une entité semble indisponible pendant quelques minutes, bien que les interruptions de service par défaut soient beaucoup plus courtes.

Dans ce cas, l’application cliente génère une exception [System.TimeoutException][System.TimeoutException] ou [MessagingException][MessagingException]. Service Bus contient un correctif pour ce problème sous la forme d’un programme de nouvelle tentative cliente automatisée. Une fois la période de nouvelle tentative écoulée, si le message n’est pas remis, vous pouvez entamer des recherches à l’aide d’autres fonctionnalités mentionnées dans l’article [Gestion des urgences et des pannes][handling outages and disasters].

## <a name="next-steps"></a>Étapes suivantes
Maintenant que vous avez appris les principes fondamentaux de la messagerie asynchrone dans Service Bus, apprenez-en davantage sur la [gestion des urgences et des pannes][handling outages and disasters].

[ServerBusyException]: /dotnet/api/microsoft.servicebus.messaging.serverbusyexception
[System.TimeoutException]: /dotnet/api/system.timeoutexception?view=netcore-3.1
[MessagingException]: /dotnet/api/microsoft.servicebus.messaging.messagingexception
[Best practices for insulating applications against Service Bus outages and disasters]: service-bus-outages-disasters.md
[Microsoft.ServiceBus.Messaging.MessagingFactory]: /dotnet/api/microsoft.servicebus.messaging.messagingfactory
[MessageReceiver]: /dotnet/api/microsoft.servicebus.messaging.messagereceiver
[QueueClient]: /dotnet/api/microsoft.servicebus.messaging.queueclient
[TopicClient]: /dotnet/api/microsoft.servicebus.messaging.topicclient
[Microsoft.ServiceBus.Messaging.PairedNamespaceOptions]: /dotnet/api/microsoft.servicebus.messaging.pairednamespaceoptions
[MessagingFactory]: /dotnet/api/microsoft.servicebus.messaging.messagingfactory
[SendAvailabilityPairedNamespaceOptions]: /dotnet/api/microsoft.servicebus.messaging.sendavailabilitypairednamespaceoptions
[NamespaceManager]: /dotnet/api/microsoft.servicebus.namespacemanager
[PairNamespaceAsync]: /dotnet/api/microsoft.servicebus.messaging.messagingfactory
[EnableSyphon]: /dotnet/api/microsoft.servicebus.messaging.sendavailabilitypairednamespaceoptions
[System.TimeSpan.Zero]: /dotnet/api/system.timespan.zero?view=netcore-3.1
[IsTransient]: /dotnet/api/microsoft.servicebus.messaging.messagingexception
[UnauthorizedAccessException]: /dotnet/api/system.unauthorizedaccessexception?view=netcore-3.1
[BacklogQueueCount]: /dotnet/api/microsoft.servicebus.messaging.sendavailabilitypairednamespaceoptions?redirectedfrom=MSDN
[handling outages and disasters]: service-bus-outages-disasters.md