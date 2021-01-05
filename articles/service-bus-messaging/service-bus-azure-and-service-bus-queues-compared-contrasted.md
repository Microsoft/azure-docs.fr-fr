---
title: Comparer les files d’attente de stockage Azure et les files d’attente Service Bus
description: Analyse les différences et les similitudes entre les deux types de files d'attente proposés par Azure.
ms.topic: article
ms.date: 11/04/2020
ms.openlocfilehash: 31992aa2012009c51cbeae78010ae8ced65fc872
ms.sourcegitcommit: 1756a8a1485c290c46cc40bc869702b8c8454016
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96928305"
---
# <a name="storage-queues-and-service-bus-queues---compared-and-contrasted"></a>Files d’attente Azure et files d’attente Service Bus : comparaison et différences
Cet article analyse les différences et les similitudes entre les deux types de files d’attente proposés par Microsoft Azure : Files d’attente de stockage et files d’attente Service Bus. À l’aide de ces informations, vous pouvez prendre une décision plus éclairée sur la solution qui répond le mieux à vos besoins.

## <a name="introduction"></a>Introduction
Azure prend en charge deux types de mécanismes de file d’attente : **Files d'attente de stockage** et **files d'attente Service Bus**.

**Les files d’attente Stockage** font partie de l’infrastructure [Stockage Azure](https://azure.microsoft.com/services/storage/). Elles vous permettent de stocker un grand nombre de messages. Vous accédez aux messages depuis n’importe où dans le monde par le biais d’appels authentifiés à l’aide du protocole HTTP ou HTTPS. La taille maximale d’un message de file d’attente est de 64 Ko. Une file d’attente peut contenir des millions de messages, dans la limite de la capacité totale d’un compte de stockage. Les files d’attente sont couramment utilisées pour créer un backlog de travail à traiter de façon asynchrone. Pour plus d’informations, consultez [Que sont les files d’attente Stockage Azure ?](../storage/queues/storage-queues-introduction.md)

Les **files d’attente Service Bus** font partie d’une infrastructure de [messagerie Azure](https://azure.microsoft.com/services/service-bus/) plus large qui prend en charge la mise en file d’attente, la publication/l’abonnement et des modèles d’intégration plus avancés. Elles sont conçues pour intégrer des applications ou des composants d’applications qui peuvent s’étendre sur plusieurs protocoles de communication, contrats de données, domaines de confiance ou environnements réseau. Pour plus d’informations sur les files d’attente/rubriques/abonnements Service Bus, consultez [Files d’attente, rubriques et abonnements Service Bus](service-bus-queues-topics-subscriptions.md).


## <a name="technology-selection-considerations"></a>Considérations relatives à la sélection de la technologie
Les files d’attente Stockage et les files d’attente Service Bus ont un ensemble de fonctionnalités légèrement différent. Vous pouvez choisir l’une ou l’autre, selon les besoins de votre solution spécifique.

Pour déterminer quelle technologie de mise en file d’attente est adaptée à une solution donnée, les développeurs et les architectes de solutions doivent prendre en compte ces recommandations. 

### <a name="consider-using-storage-queues"></a>Envisager d’utiliser des files d’attente Stockage
En tant que développeur/architecte de solutions, **vous devez envisager d’utiliser les files d’attente Azure** dans les cas de figure suivants :

* Votre application doit stocker plus de 80 Go de messages dans une file d’attente.
* Votre application veut suivre la progression du traitement d’un message dans la file d’attente. Cela est utile si le Worker qui traite un message ne répond plus. Un autre Worker peut alors utiliser ces informations pour continuer là où le Worker précédent s’était arrêté.
* Vous avez besoin de journaux d’activité côté serveur de toutes les transactions exécutées sur les files d’attente.

### <a name="consider-using-service-bus-queues"></a>Envisager d’utiliser des files d’attente Service Bus
En tant que développeur/architecte de solutions, **vous devez envisager d’utiliser les files d’attente Service Bus** lorsque :

* Votre solution doit recevoir des messages sans avoir à interroger la file d’attente. Avec Service Bus, vous pouvez y parvenir en utilisant une opération de réception à interrogation longue à l’aide des protocoles TCP pris en charge par Service Bus.
* Votre solution nécessite la file d'attente pour fournir une livraison organisée selon la méthode Premier entré, premier sortie.
* Votre solution doit prendre en charge la détection automatique des doublons.
* Vous voulez que votre application traite les messages sous forme de flux de longue durée parallèles (les messages sont associés à un flux à l’aide de la propriété [SessionId](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.sessionid) du message). Dans ce modèle, chaque nœud de l'application consommatrice entre en concurrence pour les flux, contrairement aux messages. Lorsqu'un flux est donné à un nœud consommateur, le nœud peut examiner l'état du flux de l'application à l'aide de transactions.
* Votre solution nécessite un comportement transactionnel et l'atomicité lors de l'envoi ou de la réception de plusieurs messages à partir d'une file d'attente.
* Votre application gère des messages qui peuvent dépasser 64 ko, mais qui n’atteindront sans doute pas la limite des 256 ko.
* Vous êtes confronté à l'exigence de fournir un modèle d'accès basé sur les rôles aux files d'attente et des droits/autorisations différents pour les expéditeurs et les destinataires. Pour plus d’informations, consultez les articles suivants :
    - [Authentifier avec des identités managées](service-bus-managed-service-identity.md)
    - [Authentifier à partir d’une application](authenticate-application.md)
* La taille de la file d’attente ne sera pas supérieure à 80 Go.
* Vous souhaitez utiliser le protocole de messagerie basé sur les normes AMQP 1.0. Pour plus d’informations sur AMQP, consultez [Présentation d’AMQP Service Bus](service-bus-amqp-overview.md).
* Vous envisagez une migration finale de la communication point à point basée sur une file d’attente vers un modèle de messagerie de type publication-abonnement. Ce modèle permet l’intégration de destinataires supplémentaires (abonnés). Chaque destinataire reçoit des copies indépendantes de certains ou de tous les messages envoyés à la file d’attente. 
* Votre solution de messagerie doit prendre en charge la garantie de remise « Au moins une fois » sans avoir besoin de générer les composants supplémentaires de l’infrastructure.
* Votre solution doit publier et consommer des lots de messages.

## <a name="compare-storage-queues-and-service-bus-queues"></a>Comparer les files d’attente Stockage et les files d’attente Service Bus
Les tableaux des sections suivantes fournissent un regroupement logique des fonctionnalités de file d’attente. Ils vous permettent de comparer, en un coup d’œil, les capacités disponibles dans les files d’attente Stockage Azure et les files d’attente Service Bus.

## <a name="foundational-capabilities"></a>Fonctions de base
Cette section compare certaines des fonctionnalités de base fournies par les files d’attente Azure et les files d’attente Service Bus.

| Critères de comparaison | Files d’attente de stockage | Files d’attente Service Bus |
| --- | --- | --- |
| Garantie de classement |**Non** <br/><br>Pour plus d’informations, consultez la première remarque dans la section [Informations supplémentaires](#additional-information).</br> | **Oui - Premier entré premier sorti (PEPS)**<br/><br>(par le biais de l’utilisation de [sessions de messagerie](message-sessions.md)) |
| Garantie de livraison |**Au moins une fois** |**Au moins une fois** (avec le mode de réception PeekLock. Il s’agit de la valeur par défaut.) <br/><br/>**Une fois au maximum** (avec le mode de réception ReceiveAndDelete) <br/> <br/> En savoir plus sur les différents [modes de réception](service-bus-queues-topics-subscriptions.md#receive-modes)  |
| Prise en charge des opérations atomiques |**Non** |**Oui**<br/><br/> |
| Comportement de réception |**Non bloquant**<br/><br/>(se termine immédiatement si aucun nouveau message n’est trouvé) |**Blocage avec ou sans délai d’expiration**<br/><br/>(offre une interrogation longue, dite [« technique Comet »](https://go.microsoft.com/fwlink/?LinkId=613759))<br/><br/>**Non bloquant**<br/><br/>(via l’utilisation d’une API gérée sur .NET uniquement) |
| API style Push |**Non** |**Oui**<br/><br/>API .NET de sessions [QueueClient.OnMessage](/dotnet/api/microsoft.servicebus.messaging.queueclient.onmessage#Microsoft_ServiceBus_Messaging_QueueClient_OnMessage_System_Action_Microsoft_ServiceBus_Messaging_BrokeredMessage__) et [MessageSessionHandler.OnMessage](/dotnet/api/microsoft.servicebus.messaging.messagesessionhandler.onmessage#Microsoft_ServiceBus_Messaging_MessageSessionHandler_OnMessage_Microsoft_ServiceBus_Messaging_MessageSession_Microsoft_ServiceBus_Messaging_BrokeredMessage__). |
| Mode de réception |**Aperçu et attribution** |**Aperçu et verrouillage**<br/><br/>**Réception et suppression** |
| Mode d'accès exclusif |**Basé sur attribution** |**Basé sur verrouillage** |
| Durée attribution/verrouillage |**30 secondes (par défaut)**<br/><br/>**7 jours (maximum)** (Vous pouvez renouveler ou libérer l’attribution d’un message à l’aide de l’API [UpdateMessage](/dotnet/api/microsoft.azure.storage.queue.cloudqueue.updatemessage).) |**60 secondes (par défaut)**<br/><br/>Vous pouvez renouveler le verrouillage d’un message à l’aide de l’API [RenewLock](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.renewlock#Microsoft_ServiceBus_Messaging_BrokeredMessage_RenewLock). |
| Précision attribution/verrouillage |**Au niveau du message**<br/><br/>Chaque message peut avoir une valeur de délai d’expiration différente, que vous pouvez ensuite mettre à jour en fonction des besoins lors du traitement du message, à l’aide de l’API [UpdateMessage](/dotnet/api/microsoft.azure.storage.queue.cloudqueue.updatemessage). |**Au niveau de la file d’attente**<br/><br/>(pour chaque file d’attente, une précision de verrouillage est appliquée à tous les messages, mais vous pouvez renouveler le verrouillage à l’aide de l’API [RenewLock](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.renewlock#Microsoft_ServiceBus_Messaging_BrokeredMessage_RenewLock).) |
| Réception par lots |**Oui**<br/><br/>(en spécifiant explicitement le nombre de messages lors de la récupération des messages, avec un maximum de 32 messages) |**Oui**<br/><br/>(en activant implicitement une propriété de pré-récupération ou explicitement par le biais de l’utilisation de transactions) |
| Envoi par lots |**Non** |**Oui**<br/><br/>(par le biais de l’utilisation de transactions ou du traitement par lots côté client) |

### <a name="additional-information"></a>Informations supplémentaires
* Les messages dans les files d’attente Stockage sont généralement traités selon le principe du premier entré/premier sorti, mais il arrive qu’ils ne soient pas dans l’ordre correct. Par exemple, lorsque la durée du délai de visibilité d’un message expire parce qu’une application cliente s’est bloquée lors du traitement d’un message. Lorsque le délai de visibilité expire, le message est de nouveau visible dans la file d'attente et un autre processus peut le retirer de la file d'attente. À ce stade, le message nouvellement visible peut être placé dans la file d’attente pour en être de nouveau retiré.
* Le modèle Premier entré, premier sorti garanti dans les files d'attente Service Bus requiert l'utilisation de sessions de messagerie. Si l’application se bloque pendant qu’elle traite un message reçu en mode **Aperçu et verrouillage**, la prochaine fois qu’un destinataire de file d’attente acceptera une session de messagerie, celle-ci démarrera avec le message ayant échoué après l’expiration de la durée de vie (TTL) du message.
* Les files d’attente Stockage sont conçues pour prendre en charge les scénarios de mise en file d’attente standard, tels que les suivants :
    - Découplage des composants d’application pour augmenter la scalabilité et la tolérance aux défaillances.
    - Niveau de charge
    - Génération de workflows de processus.
* Des incohérences relatives à la gestion de message dans le contexte de sessions Service Bus peuvent être évitées via l’utilisation d’état de session pour stocker l’état de l’application relatif à la progression de la gestion de la séquence de message de la session, et via l’utilisation de transactions pour l’installation de messages et la mise à jour de l’état de session. Ce type de fonctionnalité de cohérence est parfois étiqueté *Traitement une seule fois* dans les produits d’autres fournisseurs. Tout échec de transaction entraîne évidemment la nouvelle remise des messages, et c’est pourquoi le terme n’est pas tout à fait adéquat.
* Les files d’attente de stockage fournissent un modèle de programmation uniforme et cohérent entre les files d’attente, les tables et les objets blob, pour les développeurs et les équipes d’exploitation.
* Les files d'attente Service Bus prennent en charge les transactions locales dans le contexte d'une file d'attente unique.
* Le mode **Réception et suppression** pris en charge par Service Bus offre la possibilité de réduire le nombre d’opérations de messagerie (et le coût associé) en échange d’une garantie de livraison réduite.
* Les files d’attente de stockage fournissent des attributions avec la possibilité de les étendre aux messages. Cette fonctionnalité permet aux processus Worker de conserver des baux courts sur les messages. Ainsi, si un Worker se bloque, le message peut être rapidement traité à nouveau par un autre Worker. En outre, un Worker peut étendre le bail d’un message s’il doit le traiter pendant une période plus longue que la durée du bail.
* Les files d’attente de stockage offrent un délai de visibilité que vous pouvez définir au moment de la mise en file d’attente ou du retrait de la file d’attente d’un message. En outre, vous pouvez mettre à jour un message avec des valeurs de bail différentes à l’exécution et mettre à jour des valeurs différentes entre des messages dans la même file d’attente. Les dépassement de délai d’attente de verrou Service Bus sont définis dans les métadonnées de file d’attente. Toutefois, vous pouvez renouveler le verrou en appelant la méthode [RenewLock](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.renewlock#Microsoft_ServiceBus_Messaging_BrokeredMessage_RenewLock).
* Le délai d'attente maximal pour une opération de réception de blocage dans les files d'attente Service Bus est de 24 jours. Toutefois, les délais d'attente basés sur REST ont une valeur maximale de 55 secondes.
* Le traitement par lots côté client fourni par Service Bus permet à un client de file d'attente de traiter par lots plusieurs messages dans une seule opération d'envoi. Le traitement par lots n'est disponible que pour les opérations d'envoi asynchrones.
* Des fonctionnalités telles que le plafond de 200 To pour les files d’attente Stockage (davantage lorsque vous virtualisez des comptes) et un nombre illimité de files d’attente en font une plateforme idéale pour les fournisseurs SaaS.
* Les files d’attente de stockage fournissent un mécanisme performant et flexible de contrôle d’accès délégué.

## <a name="advanced-capabilities"></a>Fonctionnalités avancées
Cette section compare les fonctionnalités avancées des files d’attente Azure et des files d’attente Service Bus.

| Critères de comparaison | Files d’attente de stockage | Files d’attente Service Bus |
| --- | --- | --- |
| Remise planifiée |**Oui** |**Oui** |
| Lettre morte automatique |**Non** |**Oui** |
| Augmenter la valeur de durée de vie de la file d'attente |**Oui**<br/><br/>(par le biais de la mise à jour sur place du délai de visibilité) |**Oui**<br/><br/>(par le biais d’une fonction API dédiée) |
| Prise en charge des messages incohérents |**Oui** |**Oui** |
| Mise à jour sur place |**Oui** |**Oui** |
| Journal des transactions côté serveur |**Oui** |**Non** |
| Métriques de stockage |**Oui**<br/><br/>Les **métriques par minute** fournissent des métriques en temps réel pour la disponibilité, le TPS, le nombre d’appels d’API, le nombre d’erreurs, etc. Elles sont toutes en temps réel, agrégées par minute et signalées dans les quelques minutes qui suivent ce qui vient de se passer en production. Pour plus d’informations, voir la page [À propos des mesures Storage Analytics](/rest/api/storageservices/fileservices/About-Storage-Analytics-Metrics). |**Oui**<br/><br/>(requêtes en bloc en appelant [GetQueues](/dotnet/api/microsoft.servicebus.namespacemanager.getqueues#Microsoft_ServiceBus_NamespaceManager_GetQueues)) |
| Gestion de l'état |**Non** |**Oui**<br/><br/>[Microsoft.ServiceBus.Messaging.EntityStatus.Active](/dotnet/api/microsoft.servicebus.messaging.entitystatus), [Microsoft.ServiceBus.Messaging.EntityStatus.Disabled](/dotnet/api/microsoft.servicebus.messaging.entitystatus), [Microsoft.ServiceBus.Messaging.EntityStatus.SendDisabled](/dotnet/api/microsoft.servicebus.messaging.entitystatus), [Microsoft.ServiceBus.Messaging.EntityStatus.ReceiveDisabled](/dotnet/api/microsoft.servicebus.messaging.entitystatus) |
| Transfert automatique de messages |**Non** |**Oui** |
| Fonction de purge de la file d'attente |**Oui** |**Non** |
| Groupes de messages |**Non** |**Oui**<br/><br/>(par le biais de l’utilisation de sessions de messagerie) |
| État de l'application par groupe de messages |**Non** |**Oui** |
| Détection des doublons |**Non** |**Oui**<br/><br/>(configurable du côté expéditeur) |
| Consultation des groupes de messages |**Non** |**Oui** |
| Récupération des sessions de messagerie par ID |**Non** |**Oui** |

### <a name="additional-information"></a>Informations supplémentaires
* Les deux technologies de mise en file d'attente permettent de planifier la remise d'un message à une date ultérieure.
* Le transfert automatique de files d’attente permet à des milliers de files d’attente de transférer automatiquement leurs messages à une seule file d’attente, à partir de laquelle l’application réceptrice consomme le message. Vous pouvez utiliser ce mécanisme pour garantir la sécurité, contrôler le flux et isoler le stockage entre chaque serveur de publication de message.
* Les files d’attente de stockage prennent en charge la mise à jour du contenu des messages. Vous pouvez utiliser cette fonctionnalité pour les informations d'état persistantes et les mises à jour incrémentielles de progression dans le message, afin que celui-ci puisse être traité à partir du dernier point de contrôle connu, au lieu de recommencer depuis le début. Avec les files d'attente Service Bus, vous pouvez activer le même scénario à l'aide des sessions de messagerie. Les sessions vous permettent d’enregistrer et de récupérer l’état de traitement de l’application (à l’aide de [SetState](/dotnet/api/microsoft.servicebus.messaging.messagesession.setstate#Microsoft_ServiceBus_Messaging_MessageSession_SetState_System_IO_Stream_) et [GetState](/dotnet/api/microsoft.servicebus.messaging.messagesession.getstate#Microsoft_ServiceBus_Messaging_MessageSession_GetState)).
* Les files d’attente Service Bus prennent en charge [la mise en file d’attente de lettres mortes](service-bus-dead-letter-queues.md). Cela peut être utile pour isoler les messages qui répondent aux critères suivants :
    - Les messages ne peuvent pas être traités correctement par l’application réceptrice. 
    - Les messages ne peuvent pas atteindre leur destination en raison d’une propriété de durée de vie (TTL) expirée. La valeur de durée de vie spécifie la durée pendant laquelle un message reste dans la file d'attente. Avec Service Bus, le message est déplacé vers une file d'attente spéciale appelée $DeadLetterQueue lorsque la durée de vie expire.
* Pour trouver les messages « incohérents » dans les files d’attente Azure, lors du retrait d’un message de la file d’attente, l’application examine la propriété [DequeueCount](/dotnet/api/microsoft.azure.storage.queue.cloudqueuemessage.dequeuecount) du message. Si la propriété **DequeueCount** dépasse un seuil donné, l’application déplace le message vers une file d’attente de type « lettre morte » définie par l’application.
* Les files d’attente Stockage vous permettent d’obtenir un journal détaillé de toutes les transactions exécutées sur la file d’attente et des métriques agrégées. Ces deux options sont utiles pour déboguer et comprendre comment votre application utilise les files d’attente de stockage. Elles sont également utiles pour régler les performances de votre application et réduire les coûts d’utilisation des files d’attente.
* Les sessions de messages prises en charge par Service Bus permettent aux messages appartenant à un groupe logique d’être associés à un destinataire. Cela crée une affinité de type session entre les messages et leurs destinataires respectifs. Vous pouvez activer cette fonctionnalité avancée dans Service Bus en définissant la propriété [SessionID](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.sessionid#Microsoft_ServiceBus_Messaging_BrokeredMessage_SessionId) d’un message. Les récepteurs peuvent ensuite écouter par le biais d'un ID de session spécifique et recevoir les messages qui partagent l'identificateur de session spécifié.
* La fonctionnalité de détection des doublons des files d’attente Service Bus supprime automatiquement les messages en double envoyés à une file d’attente ou à une rubrique, selon la valeur de la propriété [MessageId](/dotnet/api/microsoft.servicebus.messaging.brokeredmessage.messageid#Microsoft_ServiceBus_Messaging_BrokeredMessage_MessageId).

## <a name="capacity-and-quotas"></a>Capacité et quotas
Cette section compare les files d’attente de stockage et les files d’attente Service Bus du point de vue [des capacités et des quotas](service-bus-quotas.md) applicables.

| Critères de comparaison | Files d’attente de stockage | Files d’attente Service Bus |
| --- | --- | --- |
| Taille de file d'attente maximale |**500 To**<br/><br/>(limitée à une [capacité de compte de stockage unique](../storage/common/storage-introduction.md#queue-storage)) |**1 Go à 80 Go**<br/><br/>(définie lors de la création d’une file d’attente et d’une [activation du partitionnement](service-bus-partitioning.md) – consultez la section « Informations supplémentaires ») |
| Taille de message maximale |**64 Ko**<br/><br/>(48 Ko avec un codage en **Base64**)<br/><br/>Azure prend en charge les messages volumineux en combinant des files d’attente et des objets blob. Dans ce cas, vous pouvez placer jusqu’à 200 Go en file d’attente pour un seul élément. |**256 Ko** ou **1 Mo**<br/><br/>(y compris l’en-tête et le corps, taille maximale d’en-tête : 64 Ko).<br/><br/>Dépend du [niveau de service](service-bus-premium-messaging.md). |
| Durée de vie maximale des messages |**Infinie** (version d’API 2017-07-27 ou ultérieure) |**TimeSpan.Max** |
| Nombre maximal de files d'attente |**Illimité** |**10,000**<br/><br/>(par espace de noms de service) |
| Nombre maximal de clients simultanés |**Illimité** |**5 000** |

### <a name="additional-information"></a>Informations supplémentaires
* Service Bus applique les limites en termes de taille de file d'attente. La taille maximale de file d’attente est spécifiée lors de la création d’une file d’attente. Elle peut être comprise entre 1 Go et 80 Go. Si la taille de la file d’attente atteint cette limite, les messages entrants supplémentaires sont rejetés et l’appelant reçoit une exception. Pour plus d’informations sur les quotas dans Service Bus, consultez [Quotas Service Bus](service-bus-quotas.md).
* Le partitionnement n’est pas pris en charge dans le [niveau Premium](service-bus-premium-messaging.md). Dans le niveau de messagerie Standard, vous pouvez créer des files d’attente et des rubriques Service Bus avec des tailles de 1 (valeur par défaut), 2, 3, 4 ou 5 Go. Si le partitionnement est activé, Service Bus crée 16 copies (16 partitions) de l’entité, chacune de la même taille spécifiée. Par conséquent, si vous créez une file d’attente de 5 Go, avec 16 partitions, la taille maximale de la file d’attente est (5 * 16) = 80 Go.  Vous pouvez voir la taille maximale de votre file d’attente ou rubrique partitionnée sur le [portail Azure][Azure portal].
* Avec les files d’attente Stockage, si le contenu du message n’est pas sécurisé pour XML, il doit être encodé au format **Base64**. Si vous encodez le message au format **Base64**, la charge utilisateur peut atteindre 48 Ko, au lieu de 64 Ko.
* Avec les files d’attente Service Bus, chaque message stocké dans une file d’attente est composé de deux parties : un en-tête et un corps. La taille totale du message ne peut pas dépasser la taille maximale de message prise en charge par le niveau de service.
* Lorsque des clients communiquent avec des files d'attente Service Bus au moyen du protocole TCP, le nombre maximal de connexions simultanées à une file d'attente Service Bus unique est limité à 100. Ce nombre est partagé entre les expéditeurs et les destinataires. Si ce quota est atteint, les requêtes pour des connexions supplémentaires sont rejetées et une exception sera reçue par le code appelant. Cette limite n’est pas imposée aux clients qui se connectent aux files d’attente à l’aide d’une API REST.
* Si vous avez besoin de plus de 10 000 files d'attente dans un seul espace de noms Service Bus, vous pouvez contacter l'équipe de support Azure et demander l'augmentation du nombre de files d'attente. Pour aller au-delà de 10 000 files d’attente avec Service Bus, vous pouvez également créer des espaces de noms supplémentaires à l’aide du [portail Azure][Azure portal].

## <a name="management-and-operations"></a>Gestion et opérations
Cette section compare les fonctionnalités de gestion fournies par les files d’attente de stockage et les files d’attente Service Bus.

| Critères de comparaison | Files d’attente de stockage | Files d’attente Service Bus |
| --- | --- | --- |
| Protocole de gestion |**REST sur HTTP/HTTPS** |**REST sur HTTPS** |
| Protocole d'exécution |**REST sur HTTP/HTTPS** |**REST sur HTTPS**<br/><br/>**Norme AMQP 1.0 (TCP avec TLS)** |
| API .NET |**Oui**<br/><br/>(API de client de stockage .NET) |**Oui**<br/><br/>(API Service Bus .NET) |
| C++ natif |**Oui** |**Oui** |
| API Java |**Oui** |**Oui** |
| API PHP |**Oui** |**Oui** |
| API Node.js |**Oui** |**Oui** |
| Prise en charge des métadonnées arbitraires |**Oui** |**Non** |
| Règles d'affectation des noms aux files d'attente |**Jusqu’à 63 caractères**<br/><br/>(les lettres dans un nom de file d’attente doivent être en minuscules) |**Jusqu’à 260 caractères**<br/><br/>(les chemins d’accès et noms des files d’attente ne sont pas sensibles à la casse) |
| Fonction d'obtention de la longueur de la file d'attente |**Oui**<br/><br/>(valeur approximative si les messages expirent au-delà de la durée de vie sans être supprimés) |**Oui**<br/><br/>(valeur exacte, à un moment donné) |
| Fonction Peek (aperçu) |**Oui** |**Oui** |

### <a name="additional-information"></a>Informations supplémentaires
* Les files d’attente de stockage prennent en charge les attributs arbitraires qui peuvent s’appliquer à la description de la file d’attente, sous la forme de paires nom/valeur.
* Les deux technologies de file d'attente offrent la possibilité d'obtenir un aperçu d'un message sans avoir à le verrouiller, ce qui peut être utile lors de la mise en œuvre d'un outil de type explorateur/navigateur de file d'attente.
* Les API de messagerie répartie .NET de Service Bus utilisent les connexions TCP en duplex intégral pour améliorer les performances par rapport au protocole REST sur HTTP, et elles prennent en charge le protocole standard AMQP 1.0.
* Les noms des files d’attente de stockage peuvent compter entre 3 et 63 caractères, et contenir des lettres minuscules, des nombres et des traits d’union. Pour plus d’informations, consultez [Affectation de noms pour les files d’attente et les métadonnées](/rest/api/storageservices/fileservices/Naming-Queues-and-Metadata).
* Les noms des files d'attente Service Bus peuvent compter jusqu'à 260 caractères. Les règles d'affectation de noms sont moins restrictives. Les noms des files d’attente Service Bus peuvent contenir des lettres, des nombres, des points (.), des traits d’union (-) et des traits de soulignement (_).

## <a name="authentication-and-authorization"></a>Authentification et autorisation
Cette section décrit les fonctionnalités d’authentification et d’autorisation prises en charge par les files d’attente Service Bus et les files d’attente de stockage.

| Critères de comparaison | Files d’attente de stockage | Files d’attente Service Bus |
| --- | --- | --- |
| Authentification |**Clé symétrique** |**Clé symétrique** |
| Modèle de sécurité |Accès délégué via des jetons SAS. |SAS |
| Fédération de fournisseur d’identité |**Non** |**Oui** |

### <a name="additional-information"></a>Informations supplémentaires
* Chaque requête à l'une des technologies de file d'attente doit être authentifiée. Les files d’attente publiques avec accès anonyme ne sont pas prises en charge. À l’aide de la [SAP](service-bus-sas.md), vous pouvez résoudre ce scénario en publiant une SAP en écriture seule, une SAP en lecture seule ou une SAP à accès total.
* Le schéma d’authentification fourni par les files d’attente Stockage implique l’utilisation d’une clé symétrique. Cette clé est un code d’authentification de message basé sur le hachage (HMAC), calculé avec l’algorithme SHA-256 et encodé sous la forme d’une chaîne **Base64**. Pour plus d’informations sur le protocole respectif, consultez [Authentification pour les services Azure Storage](/rest/api/storageservices/fileservices/Authentication-for-the-Azure-Storage-Services). Les files d'attente Service Bus prennent en charge un modèle similaire utilisant des clés symétriques. Pour plus d’informations, consultez [Authentification par signature d’accès partagé avec Service Bus](service-bus-sas.md).

## <a name="conclusion"></a>Conclusion
En ayant une meilleure compréhension des deux technologies, vous pouvez prendre une décision plus informée sur les technologies de file d’attente à utiliser et à quel moment les utiliser. La décision d’utiliser des files d’attente de stockage ou des files d’attente Service Bus dépend de plusieurs facteurs. Ces facteurs peuvent reposer largement sur les besoins spécifiques de votre application et de son architecture. 

Vous préférerez peut-être choisir des files d’attente Stockage pour des raisons semblables aux suivantes :

- Si votre application utilise déjà les capacités de base de Microsoft Azure 
- Si vous avez besoin d’une communication et d’une messagerie de base entre les services 
- Si vous avez besoin de files d’attente dont la taille peut être supérieure à 80 Go

Les files d’attente Service Bus fournissent un certain nombre de fonctionnalités avancées, telles que les suivantes. Il peut donc s’agir d’un choix privilégié si vous créez une application hybride ou si votre application a besoin de ces fonctionnalités.

- [Sessions](message-sessions.md)
- [Transactions](service-bus-transactions.md)
- [Détection des doublons](duplicate-detection.md)
- [Mise automatique en file d’attente de lettres mortes](service-bus-dead-letter-queues.md)
- [Capacités de publication et d’abonnement durables](service-bus-queues-topics-subscriptions.md#topics-and-subscriptions) 



## <a name="next-steps"></a>Étapes suivantes
Les articles suivants fournissent des conseils et des informations sur l’utilisation des files d’attente de stockage ou Service Bus.

* [Prise en main des files d’attente Service Bus](service-bus-dotnet-get-started-with-queues.md)
* [Utilisation du service de stockage de files d’attente](../storage/queues/storage-dotnet-how-to-use-queues.md)
* [Meilleures pratiques relatives aux améliorations de performances à l’aide de la messagerie répartie Service Bus](service-bus-performance-improvements.md)
* [Présentation des files d’attente et des rubriques dans Azure Service Bus (blog)](https://www.serverless360.com/blog/azure-service-bus-queues-vs-topics)
* [Guide du développeur pour Service Bus](http://www.cloudcasts.net/devguide/Default.aspx?id=11030)
* [Utilisation du service de mise en file d’attente dans Azure](https://www.developerfusion.com/article/120197/using-the-queuing-service-in-windows-azure/)

[Azure portal]: https://portal.azure.com

