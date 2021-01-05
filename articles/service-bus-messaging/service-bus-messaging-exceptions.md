---
title: Azure Service Bus - exceptions de la messagerie | Microsoft Docs
description: Cet article fournit la liste des exceptions de messagerie Azure Service Bus et les actions suggérées à entreprendre quand une exception se produit.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: e4aa6d82c20e21caabf0205d7446cf88ed8b7f34
ms.sourcegitcommit: 17b36b13857f573639d19d2afb6f2aca74ae56c1
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/10/2020
ms.locfileid: "94409312"
---
# <a name="service-bus-messaging-exceptions"></a>Exceptions de la messagerie Service Bus
Cet article répertorie les exceptions .NET générées par les API .NET Framework. 

## <a name="exception-categories"></a>Catégories d'exceptions
Les API de messagerie génèrent des exceptions qui peuvent être classées dans les catégories suivantes, ainsi que l'action associée que vous pouvez mener pour essayer de les résoudre. La signification et les causes d’une exception peuvent varier en fonction du type d’entité de messagerie :

1. Erreur de codage utilisateur ([System.ArgumentException](/dotnet/api/system.argumentexception?view=netcore-3.1&preserve-view=true), [System.InvalidOperationException](/dotnet/api/system.invalidoperationexception?view=netcore-3.1&preserve-view=true), [System.OperationCanceledException](/dotnet/api/system.operationcanceledexception?view=netcore-3.1&preserve-view=true), [System.Runtime.Serialization.SerializationException](/dotnet/api/system.runtime.serialization.serializationexception?view=netcore-3.1&preserve-view=true)). Action générale : essayez de corriger le code avant de poursuivre.
2. Erreur d’installation ou de configuration ([Microsoft.ServiceBus.Messaging.MessagingEntityNotFoundException](/dotnet/api/microsoft.azure.servicebus.messagingentitynotfoundexception), [System.UnauthorizedAccessException](/dotnet/api/system.unauthorizedaccessexception?view=netcore-3.1&preserve-view=true). Action générale : révisez la configuration et modifiez-la si besoin.
3. Exceptions temporaires ([Microsoft.ServiceBus.Messaging.MessagingException](/dotnet/api/microsoft.servicebus.messaging.messagingexception), [Microsoft.ServiceBus.Messaging.ServerBusyException](/dotnet/api/microsoft.azure.servicebus.serverbusyexception), [Microsoft.ServiceBus.Messaging.MessagingCommunicationException](/dotnet/api/microsoft.servicebus.messaging.messagingcommunicationexception)). Action générale : relancez l'opération ou avertissez les utilisateurs. La classe `RetryPolicy` du SDK client peut être configurée pour traiter automatiquement les nouvelles tentatives. Pour plus d’informations, consultez les [conseils sur les nouvelles tentatives](/azure/architecture/best-practices/retry-service-specific#service-bus).
4. Autres exceptions ([System.Transactions.TransactionException](/dotnet/api/system.transactions.transactionexception?view=netcore-3.1&preserve-view=true), [System.TimeoutException](/dotnet/api/system.timeoutexception?view=netcore-3.1&preserve-view=true), [Microsoft.ServiceBus.Messaging.MessageLockLostException](/dotnet/api/microsoft.azure.servicebus.messagelocklostexception), [Microsoft.ServiceBus.Messaging.SessionLockLostException](/dotnet/api/microsoft.azure.servicebus.sessionlocklostexception)). Action générale : propre au type d’exception. Reportez-vous au tableau de la section suivante : 

## <a name="exception-types"></a>Types d'exceptions
Le tableau suivant répertorie les types d'exceptions de la messagerie, leurs causes et les propositions d'actions que vous pouvez effectuer.

| **Type d'exception** | **Description/Cause/Exemples** | **Action suggérée** | **Remarques sur la nouvelle tentative automatique/immédiate** |
| --- | --- | --- | --- |
| [TimeoutException](/dotnet/api/system.timeoutexception?view=netcore-3.1&preserve-view=true) |Le serveur n’a pas répondu à l’opération demandée dans le délai spécifié, qui est contrôlé par le paramètre [OperationTimeout](/dotnet/api/microsoft.servicebus.messaging.messagingfactorysettings). Le serveur peut avoir terminé l'opération demandée. Cela peut se produire en raison des délais observés sur le réseau ou une autre infrastructure. |Vérifiez la cohérence de l'état du système et réessayez si nécessaire. Consultez [Exceptions au délai d’expiration](#timeoutexception). |Dans certains cas, l'exécution d'une nouvelle tentative peut aider ; ajouter une logique de nouvelle tentative au code. |
| [InvalidOperationException](/dotnet/api/system.invalidoperationexception?view=netcore-3.1&preserve-view=true) |L’opération utilisateur demandée n’est pas autorisée sur le serveur ou le service. Consultez le message de l'exception pour obtenir plus d'informations. Par exemple, le paramètre [Complete()](/dotnet/api/microsoft.azure.servicebus.queueclient.completeasync) génère cette exception si le message a été reçu en mode [ReceiveAndDelete](/dotnet/api/microsoft.azure.servicebus.receivemode). |Vérifiez le code et consultez la documentation. Assurez-vous que l'opération demandée est valide. |La nouvelle tentative ne résout pas le problème. |
| [OperationCanceledException](/dotnet/api/system.operationcanceledexception?view=netcore-3.1&preserve-view=true) |Une tentative est effectuée pour appeler une opération sur un objet qui a déjà été fermé, abandonné ou supprimé. Dans de rares cas, la transaction ambiante est déjà supprimée. |Vérifiez le code et veillez à ce qu’il n’appelle pas d’opérations sur un objet supprimé. |La nouvelle tentative ne résout pas le problème. |
| [UnauthorizedAccessException](/dotnet/api/system.unauthorizedaccessexception?view=netcore-3.1&preserve-view=true) |L’objet [TokenProvider](/dotnet/api/microsoft.servicebus.tokenprovider) n’a pas pu obtenir de jeton, le jeton n’est pas valide ou le jeton ne contient pas les revendications nécessaires pour exécuter l’opération. |Assurez-vous que le fournisseur de jetons est créé avec les valeurs correctes. Vérifiez la configuration du service ACS (Access Control Service). |Dans certains cas, l'exécution d'une nouvelle tentative peut aider ; ajouter une logique de nouvelle tentative au code. |
| [ArgumentException](/dotnet/api/system.argumentexception?view=netcore-3.1&preserve-view=true)<br /> [ArgumentNullException](/dotnet/api/system.argumentnullexception?view=netcore-3.1&preserve-view=true)<br />[ArgumentOutOfRangeException](/dotnet/api/system.argumentoutofrangeexception?view=netcore-3.1&preserve-view=true) |Un ou plusieurs des arguments fournis à la méthode ne sont pas valides.<br /> L’URI fourni à [NamespaceManager](/dotnet/api/microsoft.servicebus.namespacemanager) ou [Ceate](/dotnet/api/microsoft.servicebus.messaging.messagingfactory) contient un ou plusieurs segments de chemin d’accès.<br /> Le schéma d’URI fourni à [NamespaceManager](/dotnet/api/microsoft.servicebus.namespacemanager) ou [Ceate](/dotnet/api/microsoft.servicebus.messaging.messagingfactory) n’est pas valide. <br />La valeur de la propriété est supérieure à 32 ko. |Vérifiez le code appelant et assurez-vous que les arguments sont corrects. |La nouvelle tentative ne résout pas le problème. |
| [MessagingEntityNotFoundException](/dotnet/api/microsoft.azure.servicebus.messagingentitynotfoundexception) |L’entité associée à l’opération n’existe pas ou a été supprimée. |Assurez-vous que l'entité existe. |La nouvelle tentative ne résout pas le problème. |
| [MessageNotFoundException](/dotnet/api/microsoft.servicebus.messaging.messagenotfoundexception) |Tentative de réception d'un message avec un numéro de séquence spécifique. Ce message est introuvable. |Vérifiez que le message n’a pas déjà été reçu. Vérifiez la file d'attente de lettres mortes pour voir si le message a été désactivé. |La nouvelle tentative ne résout pas le problème. |
| [MessagingCommunicationException](/dotnet/api/microsoft.servicebus.messaging.messagingcommunicationexception) |Le client ne peut pas établir de connexion à Service Bus. |Assurez-vous que le nom d'hôte fourni est correct et que l'hôte est accessible. <p>Si votre code s’exécute dans un environnement avec un pare-feu/proxy, assurez-vous que le trafic vers le domaine Service Bus/l’adresse IP et les ports n’est pas bloqué.</p>|Une nouvelle tentative peut aider en cas de problèmes de connectivité intermittents. |
| [ServerBusyException](/dotnet/api/microsoft.azure.servicebus.serverbusyexception) |Le service n’est pas en mesure de traiter la demande pour l’instant. |Le client peut attendre pendant une période de temps, puis recommencer l'opération. |Le client peut réessayer après un certain temps. Si une nouvelle tentative provoque une exception différente, vérifiez le comportement de nouvelle tentative de cette exception. |
| [MessagingException](/dotnet/api/microsoft.servicebus.messaging.messagingexception) |Exception de messagerie générique qui peut être levée dans les cas suivants :<p>une tentative est effectuée pour créer un [QueueClient](/dotnet/api/microsoft.azure.servicebus.queueclient) à l’aide d’un nom ou d’un chemin d’accès qui appartient à un autre type d’entité (par exemple, une rubrique).</p><p>Une tentative est effectuée pour envoyer un message de taille supérieure à 256 Ko. </p>Le serveur ou le service a rencontré une erreur lors du traitement de la demande. Consultez le message de l'exception pour obtenir plus d'informations. Il s’agit généralement d’une exception temporaire.</p><p>La demande a été arrêtée parce que l’entité est limitée. Code d’erreur : 50001, 50002, 50008. </p> | Vérifiez le code et assurez-vous que seuls les objets sérialisables sont utilisés dans le corps du message (ou utilisez un sérialiseur personnalisé). <p>Consultez la documentation pour connaître les types de valeurs des propriétés pris en charge et utilisez uniquement les types pris en charge.</p><p> Vérifiez la propriété [IsTransient](/dotnet/api/microsoft.servicebus.messaging.messagingexception) . Si sa valeur est **true** , vous pouvez réessayer d’effectuer l’opération. </p>| Si l’exception est due à une limitation, attendez quelques secondes, puis réessayez l’opération. Le comportement de la nouvelle tentative n’est pas défini et peut ne pas être utile dans d’autres scénarios.|
| [MessagingEntityAlreadyExistsException](/dotnet/api/microsoft.servicebus.messaging.messagingentityalreadyexistsexception) |Tentative de création d'une entité dont le nom est déjà utilisé par une autre entité de l'espace de noms de ce service. |Supprimez l'entité existante ou choisissez un autre nom pour l'entité à créer. |La nouvelle tentative ne résout pas le problème. |
| [QuotaExceededException](/dotnet/api/microsoft.azure.servicebus.quotaexceededexception) |L’entité de messagerie a atteint sa taille maximale autorisée, ou le nombre maximal de connexions à un espace de noms a été dépassé. |Créez de l’espace dans l’entité en recevant des messages à partir de l’entité ou de ses files d’attente secondaires. Consultez [QuotaExceededException](#quotaexceededexception). |Une nouvelle tentative peut aider si des messages ont été supprimés entre-temps. |
| [RuleActionException](/dotnet/api/microsoft.servicebus.messaging.ruleactionexception) |Service Bus renvoie cette exception si vous essayez de créer une action de règle non valide. Service Bus associe cette exception à un message désactivé si une erreur se produit lors du traitement de l'action de règle pour ce message. |Vérifiez que l'action de règle est correcte. |La nouvelle tentative ne résout pas le problème. |
| [FilterException](/dotnet/api/microsoft.servicebus.messaging.filterexception) |Service Bus renvoie cette exception si vous essayez de créer un filtre non valide. Service Bus associe cette exception à un message désactivé si une erreur se produit lors du traitement du filtre pour ce message. |Vérifiez l'exactitude du filtre. |La nouvelle tentative ne résout pas le problème. |
| [SessionCannotBeLockedException](/dotnet/api/microsoft.servicebus.messaging.sessioncannotbelockedexception) |Tentative d'acceptation d'une session avec un ID de session spécifique, mais la session est actuellement verrouillée par un autre client. |Assurez-vous que la session est déverrouillée par les autres clients. |Une nouvelle tentative peut aider si la session a été libérée entre-temps. |
| [TransactionSizeExceededException](/dotnet/api/microsoft.servicebus.messaging.transactionsizeexceededexception) |Trop d'opérations font partie de la transaction. |Réduisez le nombre d'opérations qui font partie de cette transaction. |La nouvelle tentative ne résout pas le problème. |
| [MessagingEntityDisabledException](/dotnet/api/microsoft.azure.servicebus.messagingentitydisabledexception) |Demande d'une opération d'exécution sur une entité désactivée. |Activez l'entité. |Une nouvelle tentative peut aider si l'entité a été activée entre-temps. |
| [NoMatchingSubscriptionException](/dotnet/api/microsoft.servicebus.messaging.nomatchingsubscriptionexception) |Service Bus renvoie cette exception si vous envoyez un message à une rubrique avec le filtre préalable activé et si aucun des filtres ne correspond. |Assurez-vous qu'au moins un filtre correspond. |La nouvelle tentative ne résout pas le problème. |
| [MessageSizeExceededException](/dotnet/api/microsoft.servicebus.messaging.messagesizeexceededexception) |Une charge utile de message dépasse la limite de 256 Ko. La limite de 256 Ko correspond à la taille totale du message, qui peut inclure des propriétés système et toute surcharge .NET. |Réduisez la taille de la charge utile de message, puis recommencez l'opération. |La nouvelle tentative ne résout pas le problème. |
| [TransactionException](/dotnet/api/system.transactions.transactionexception) |La transaction ambiante ( *Transaction.Current* ) n'est pas valide. Elle a peut-être été terminée ou annulée. Une exception en interne peut fournir des informations supplémentaires. | |La nouvelle tentative ne résout pas le problème. |
| [TransactionInDoubtException](/dotnet/api/system.transactions.transactionindoubtexception) |Une opération est tentée sur une transaction incertaine, ou une tentative est faite pour valider la transaction et la transaction devient incertaine. |Votre application doit gérer cette exception (comme un cas spécial), car la transaction a peut-être déjà été validée. |- |

## <a name="quotaexceededexception"></a>QuotaExceededException
[QuotaExceededException](/dotnet/api/microsoft.azure.servicebus.quotaexceededexception) indique que le quota d’une entité spécifique a été dépassé.

### <a name="queues-and-topics"></a>Files d’attente et rubriques
Pour les files d’attente et les rubriques, il s’agit souvent de la taille de la file d’attente. La propriété du message d’erreur contient davantage d’informations, comme dans l’exemple suivant :

```Output
Microsoft.ServiceBus.Messaging.QuotaExceededException
Message: The maximum entity size has been reached or exceeded for Topic: 'xxx-xxx-xxx'. 
    Size of entity in bytes:1073742326, Max entity size in bytes:
1073741824..TrackingId:xxxxxxxxxxxxxxxxxxxxxxxxxx, TimeStamp:3/15/2013 7:50:18 AM
```

Ce message indique que la rubrique a dépassé sa limite de taille, dans ce cas 1 Go (limite de taille par défaut). 

### <a name="namespaces"></a>Espaces de noms

En ce qui concerne les espaces de noms, [QuotaExceededException](/dotnet/api/microsoft.azure.servicebus.quotaexceededexception) peut indiquer qu’une application a dépassé le nombre maximal de connexions à un espace de noms. Par exemple :

```Output
Microsoft.ServiceBus.Messaging.QuotaExceededException: ConnectionsQuotaExceeded for namespace xxx.
<tracking-id-guid>_G12 ---> 
System.ServiceModel.FaultException`1[System.ServiceModel.ExceptionDetail]: 
ConnectionsQuotaExceeded for namespace xxx.
```

### <a name="common-causes"></a>Causes courantes
Il existe deux causes courantes pour cette erreur : la file d'attente de lettres mortes et des récepteurs de messages non fonctionnels.

1. **[File d’attente de lettres mortes](service-bus-dead-letter-queues.md)** Un lecteur ne parvient pas à terminer les messages et ceux-ci sont renvoyés à la file d’attente/rubrique après expiration du verrouillage. Cela peut se produire si le lecteur rencontre une exception qui l’empêche d’appeler [BrokeredMessage.Complete](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.complete). Une fois un message lu 10 fois, il passe à la file d'attente de lettres mortes par défaut. Ce comportement est contrôlé par la propriété [QueueDescription.MaxDeliveryCount](/dotnet/api/microsoft.servicebus.messaging.queuedescription.maxdeliverycount) et a une valeur par défaut de 10. Quand les messages s'accumulent dans la file d'attente de lettres mortes, ils prennent de la place.
   
    Pour résoudre ce problème, lisez et terminez les messages de la file d'attente de lettres mortes, comme vous le feriez pour une autre file d'attente. Vous pouvez utiliser la méthode [FormatDeadLetterPath](/dotnet/api/microsoft.azure.servicebus.entitynamehelper.formatdeadletterpath) pour aider à mettre en forme le chemin d’accès de la file d’attente de lettres mortes.
2. **Récepteur arrêté**. Un récepteur a cessé de recevoir des messages d’une file d’attente ou d’un abonnement. Pour l’identifier, examinez la propriété [QueueDescription.MessageCountDetails](/dotnet/api/microsoft.servicebus.messaging.messagecountdetails) , qui affiche la répartition complète des messages. Si la valeur de la propriété [ActiveMessageCount](/dotnet/api/microsoft.servicebus.messaging.messagecountdetails.activemessagecount) est élevée ou augmente, les messages ne sont pas lus aussi rapidement qu’ils sont écrits.

## <a name="timeoutexception"></a>TimeoutException
Une [TimeoutException](/dotnet/api/system.timeoutexception?view=netcore-3.1&preserve-view=true) indique qu’une opération lancée par l’utilisateur dépasse le délai d’expiration de l’opération. 

Vous devez vérifier la valeur de la propriété [ServicePointManager.DefaultConnectionLimit](/dotnet/api/system.net.servicepointmanager.defaultconnectionlimit?view=netcore-3.1&preserve-view=true) car cette limite, si elle est atteinte, peut également entraîner une exception [TimeoutException](/dotnet/api/system.timeoutexception?view=netcore-3.1&preserve-view=true).

Des expirations de délai sont à prévoir pendant ou entre les opérations de maintenance telles que les mises à jour de service Service Bus (ou) les mises à jour de système d’exploitation sur les ressources qui exécutent le service. Pendant les mises à jour du système d’exploitation, les entités sont déplacées et les nœuds sont mis à jour ou redémarrés, ce qui peut entraîner des expirations de délai. Pour plus d’informations sur les contrats de niveau de service (SLA) pour le service Azure Service Bus, consultez [Contrat SLA pour Service Bus](https://azure.microsoft.com/support/legal/sla/service-bus/).


### <a name="queues-and-topics"></a>Files d’attente et rubriques
Pour les files d’attente et les rubriques, le délai d’attente est spécifié dans la propriété [MessagingFactorySettings.OperationTimeout](/dotnet/api/microsoft.servicebus.messaging.messagingfactorysettings), au sein de la chaîne de connexion ou par le biais de [ServiceBusConnectionStringBuilder](/dotnet/api/microsoft.azure.servicebus.servicebusconnectionstringbuilder). Le message d'erreur peut varier, mais il contient toujours la valeur du délai d'attente spécifiée pour l'opération en cours. 

## <a name="messagelocklostexception"></a>MessageLockLostException

### <a name="cause"></a>Cause

L’exception **MessageLockLostException** est levée lors de la réception d’un message en mode de réception [PeekLock](message-transfers-locks-settlement.md#peeklock) quand le verrou détenu par le client expire côté service.

Le verrou sur un message peut expirer pour diverses raisons : 

  * Le minuteur du verrou a expiré avant que l’application cliente l’ait renouvelé.
  * L’application cliente a acquis le verrou, l’a enregistré dans un magasin persistant, puis a redémarré. Après redémarrage, l’application cliente a examiné les messages en cours et tenté de les compléter.

### <a name="resolution"></a>Résolution

En cas d’exception **MessageLockLostException** , l’application cliente ne peut plus traiter le message. Elle peut éventuellement envisager de journaliser l’exception pour analyse, mais le client *doit* supprimer le message.

Étant donné que le verrou sur le message a expiré, celui-ci retourne à la file d’attente (ou à l’abonnement) et peut être traité par l’application cliente suivante qui appelle Receive.

En cas de dépassement de la valeur **MaxDeliveryCount** , le message peut être déplacé vers la file d’attente **DeadLetterQueue**.

## <a name="sessionlocklostexception"></a>SessionLockLostException

### <a name="cause"></a>Cause

L’exception **SessionLockLostException** est levée quand une session est acceptée et que le verrou détenu par le client expire côté service.

Le verrou sur une session peut expirer pour différentes raisons : 

  * Le minuteur du verrou a expiré avant que l’application cliente l’ait renouvelé.
  * L’application cliente a acquis le verrou, l’a enregistré dans un magasin persistant, puis a redémarré. Après redémarrage, l’application cliente a examiné les sessions en cours et tenté de traiter les messages dans celles-ci.

### <a name="resolution"></a>Résolution

En cas d’exception **SessionLockLostException** , l’application cliente ne peut plus traiter les messages de la session. Elle peut envisager de journaliser l’exception pour analyse, mais le client *doit* supprimer le message.

Étant donné que le verrou sur la session a expiré, celle-ci retourne à la file d’attente (ou à l’abonnement) et peut être verrouillée par l’application cliente suivante qui l’accepte. Étant donné que le verrou de session est détenu par une seule application cliente à un moment donné, le traitement dans l’ordre est garanti.

## <a name="socketexception"></a>SocketException

### <a name="cause"></a>Cause

Une exception **SocketException** est levée dans les cas ci-dessous :
   * Quand une tentative de connexion échoue parce que l’hôte n’a pas répondu correctement après une heure spécifiée (code d’erreur TCP 10060).
   * Quand une connexion établie a échoué parce que l’hôte connecté n’a pas pu répondre.
   * Quand une erreur s’est produite lors du traitement du message ou que l’hôte distant a dépassé le délai d’expiration.
   * Quand il y a un problème de ressource réseau sous-jacent.

### <a name="resolution"></a>Résolution

Les erreurs **SocketException** indiquent que la machine virtuelle hébergeant les applications ne peut pas convertir le nom `<mynamespace>.servicebus.windows.net` en l’adresse IP correspondante. 

Vérifiez que la commande ci-dessous parvient à mapper à une adresse IP.

```Powershell
PS C:\> nslookup <mynamespace>.servicebus.windows.net
```

Ce qui doit produire une sortie comme indiqué ci-dessous

```bash
Name:    <cloudappinstance>.cloudapp.net
Address:  XX.XX.XXX.240
Aliases:  <mynamespace>.servicebus.windows.net
```

Si le nom ci-dessus **n’est pas résolu** en adresse IP et en l’alias d’espace de noms, demandez à l’administrateur réseau d’effectuer des recherches plus poussées. La résolution de noms s’effectue au travers d’un serveur DNS qui est généralement une ressource du réseau du client. Si la résolution DNS est effectuée par Azure DNS, contactez le support Azure.

Si la résolution de noms **fonctionne comme prévu** , vérifiez si les connexions à Azure Service Bus sont autorisées [ici](service-bus-troubleshooting-guide.md#connectivity-certificate-or-timeout-issues).


## <a name="messagingexception"></a>MessagingException

### <a name="cause"></a>Cause

**MessagingException** est une exception générique qui peut être levée pour diverses raisons. Certaines de celles-ci sont répertoriées ci-dessous.

   * Une tentative est faite de créer une classe **QueueClient** sur une **Rubrique** ou un **Abonnement**.
   * La taille du message envoyé est supérieure à la limite applicable au niveau donné. Apprenez-en davantage sur les [quotas et limites](service-bus-quotas.md) Service Bus.
   * La demande de plan de données spécifique (envoi, réception, exécution, abandon) a été arrêtée en raison d’une limitation.
   * Des problèmes temporaires se posent en raison de mises à niveau et redémarrages de service.

> [!NOTE]
> La liste d’exceptions ci-dessus n’est pas exhaustive.

### <a name="resolution"></a>Résolution

Les étapes de résolution dépendent de la cause de la levée de l’exception **MessagingException**.

   * Pour des **problèmes temporaires** (où **_isTransient_ *_ a la valeur _* _true_ *_) ou des _* problèmes de limitation** , une nouvelle tentative d’exécution de l’opération peut les résoudre. La stratégie de nouvelle tentative par défaut du Kit de développement logiciel (SDK) peut être utile à cette fin.
   * Pour les autres problèmes, les détails de l’exception indiquent que les étapes du problème et de sa résolution peuvent être déduits de l’exception.

## <a name="next-steps"></a>Étapes suivantes
Pour obtenir des informations complètes sur l’API .NET Service Bus, consultez les [informations de référence sur l’API .NET Azure](/dotnet/api/overview/azure/service-bus).
Pour des conseils relatifs à la résolution des problèmes, consultez le [Guide de résolution des problèmes](service-bus-troubleshooting-guide.md).
