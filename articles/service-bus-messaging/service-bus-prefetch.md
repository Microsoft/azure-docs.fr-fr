---
title: Prérécupérer les messages Azure Service Bus | Microsoft Docs
description: Améliorez les performances en prérécupérant les messages Azure Service Bus. Les messages sont immédiatement récupérables localement, avant que l'application les demande.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: 05e23b0590f0c04171efda8fb561b4c2664ed096
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85341046"
---
# <a name="prefetch-azure-service-bus-messages"></a>Prérécupérer les messages Azure Service Bus

Lorsque l’option *Prérécupération* est activée dans l’un des clients Service Bus officiels, le destinataire acquiert silencieusement davantage de messages, dans la limite de la valeur [PrefetchCount](/dotnet/api/microsoft.azure.servicebus.queueclient.prefetchcount#Microsoft_Azure_ServiceBus_QueueClient_PrefetchCount), que le nombre de messages initialement demandé par l’application.

Par conséquent, un seul appel [Receive](/dotnet/api/microsoft.servicebus.messaging.queueclient.receive) ou [ReceiveAsync](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver.receiveasync) initial acquiert un message pour consommation immédiate qui est renvoyé dès qu’il est disponible. Ensuite, le client acquiert d’autres messages en arrière-plan pour remplir la mémoire tampon de prérécupération.

## <a name="enable-prefetch"></a>Activer la prérécupération

Avec .NET, vous activez la fonctionnalité de prérécupération en affectant une valeur supérieure à zéro à la propriété [PrefetchCount](/dotnet/api/microsoft.azure.servicebus.queueclient.prefetchcount#Microsoft_Azure_ServiceBus_QueueClient_PrefetchCount) d’un élément **MessageReceiver**, **QueueClient** ou **SubscriptionClient**. La définition de cette propriété sur la valeur zéro désactive la prérécupération.

Vous pouvez facilement ajouter ce paramètre au côté réception des exemples de paramètres [QueuesGettingStarted](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.ServiceBus.Messaging/QueuesGettingStarted) ou [ReceiveLoop](https://github.com/Azure/azure-service-bus/tree/master/samples/DotNet/Microsoft.ServiceBus.Messaging/ReceiveLoop) pour en découvrir l’effet dans ces contextes.

Pendant que les messages sont disponibles dans la mémoire tampon de prérécupération, tous les appels **Receive**/**ReceiveAsync** ultérieurs sont immédiatement traités à partir de la mémoire tampon, et cette dernière est réapprovisionnée en arrière-plan dès que de l’espace devient disponible. Si plus aucun message n’est prêt à être remis, l’opération de réception vide la mémoire tampon, puis attend ou se bloque, comme attendu.

La prérécupération fonctionne de la même façon avec les API [OnMessage](/dotnet/api/microsoft.servicebus.messaging.queueclient.onmessage) et [OnMessageAsync](/dotnet/api/microsoft.servicebus.messaging.queueclient.onmessageasync).

## <a name="if-it-is-faster-why-is-prefetch-not-the-default-option"></a>Si la prérécupération se révèle plus rapide, pourquoi n’est-elle pas activée par défaut ?

La prérécupération accélère le flux de messages en faisant en sorte qu’un message soit immédiatement récupérable localement dès ou avant que l’application le demande. Ce gain de débit résulte d’un compromis que l’auteur de l’application doit effectuer de manière explicite :

En mode de réception [ReceiveAndDelete](/dotnet/api/microsoft.servicebus.messaging.receivemode), tous les messages qui sont acquis dans la mémoire tampon de prérécupération ne sont plus disponibles dans la file d’attente et résident uniquement dans la mémoire tampon de prérécupération en mémoire jusqu’à ce qu’ils soient reçus dans l’application par le biais des API **Receive**/**ReceiveAsync** ou **OnMessage**/**OnMessageAsync**. Si l’application s’arrête avant d’avoir reçu les messages, ces derniers sont définitivement perdus.

En mode de réception [PeekLock](/dotnet/api/microsoft.servicebus.messaging.receivemode#Microsoft_ServiceBus_Messaging_ReceiveMode_PeekLock), les messages récupérés dans la mémoire tampon de prérécupération sont acquis dans la mémoire tampon à l’état verrouillé et déclenchent l’horloge de délai d’expiration du verrouillage. Si la mémoire tampon de prérécupération présente une taille conséquente, et que le traitement est si long que le verrouillage des messages arrive à expiration pendant que les messages résident dans la mémoire tampon de prérécupération ou même pendant que l’application traite les messages, l’application peut faire face à certains événements déroutants.

Il est possible que l’application acquière un message dont le verrouillage est arrivé à expiration ou doit l’être prochainement. Dans ce cas, l’application peut commencer à traiter le message, puis se trouver dans l’impossibilité d’achever le traitement en raison de l’expiration d’un verrouillage. L’application peut vérifier la propriété [LockedUntilUtc](/dotnet/api/microsoft.azure.servicebus.message.systempropertiescollection.lockeduntilutc) (qui est soumise aux variations d’horloge entre l’horloge du répartiteur et celle de la machine locale). Si le verrouillage d’un message est arrivé à expiration, l’application doit ignorer le message, et aucun appel d’API sur ou avec le message ne doit être effectué. Si le message n’est pas arrivé à expiration, mais que son expiration est imminente, il est possible de renouveler et prolonger le verrouillage avec une autre période de verrouillage par défaut en appelant [message. RenewLock()](/dotnet/api/microsoft.azure.servicebus.core.messagereceiver.renewlockasync#Microsoft_Azure_ServiceBus_Core_MessageReceiver_RenewLockAsync_System_String_).

Si le verrouillage arrive silencieusement à expiration dans la mémoire tampon de prérécupération, le message est considéré comme abandonné et redevient récupérable à partir de la file d’attente. Ceci peut alors entraîner sa récupération dans la mémoire tampon de prérécupération, en dernière position. Si la mémoire tampon de prérécupération n’est généralement pas utilisable pendant l’arrivée à expiration des messages, les messages sont prérécupérés de façon répétée, mais ne sont en fait jamais remis dans un état exploitable (correctement verrouillés), et sont finalement déplacés vers la file d’attente de lettres mortes une fois que le nombre de remises maximal a été atteint.

Si vous exigez un haut niveau de fiabilité pour le traitement des messages et que ce traitement demande des efforts et un temps considérables, il est recommandé d’utiliser la fonctionnalité de prérécupération avec prudence ou de ne pas l’employer du tout.

Si vous avez besoin d’un haut débit et que votre processus de traitement des messages est généralement peu coûteux, la prérécupération entraîne des avantages substantiels en matière de débit.

Le nombre maximal de prérécupérations et la durée de verrouillage configurés sur la file d’attente ou l’abonnement doivent être équilibrés, de sorte que le délai d’expiration du verrouillage dépasse au moins le temps de traitement des messages prévu cumulé pour la taille maximale de la mémoire tampon de prérécupération, plus un message. Parallèlement, il convient d’éviter que le délai d’expiration du verrouillage soit si long que les messages risquent de dépasser leur valeur [TimeToLive](/dotnet/api/microsoft.azure.servicebus.message.timetolive#Microsoft_Azure_ServiceBus_Message_TimeToLive) maximale lorsqu’ils sont accidentellement abandonnés, ce qui nécessite alors d’attendre l’arrivée à expiration du verrouillage des messages pour que ces derniers puissent faire l’objet d’une nouvelle tentative de remise.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations sur la messagerie Service Bus, consultez les articles suivants :

* [Files d’attente, rubriques et abonnements Service Bus](service-bus-queues-topics-subscriptions.md)
* [Prise en main des files d’attente Service Bus](service-bus-dotnet-get-started-with-queues.md)
* [Utilisation des rubriques et abonnements Service Bus](service-bus-dotnet-how-to-use-topics-subscriptions.md)
