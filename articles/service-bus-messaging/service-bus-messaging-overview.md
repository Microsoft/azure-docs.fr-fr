---
title: Présentation de la messagerie Azure Service Bus | Microsoft Docs
description: Cet article offre une vue d’ensemble d’Azure Service Bus, le répartiteur de messages Intégration Entreprise complètement managé.
ms.topic: overview
ms.date: 11/20/2020
ms.openlocfilehash: b795d0c5b429a812d90e0759fbcde3d348b86e22
ms.sourcegitcommit: c4246c2b986c6f53b20b94d4e75ccc49ec768a9a
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/04/2020
ms.locfileid: "96602714"
---
# <a name="what-is-azure-service-bus"></a>Qu’est-ce qu’Azure Service Bus ?
Microsoft Azure Service Bus est un répartiteur de messages d’entreprise complètement managé, avec des files d’attente de messages et des rubriques d’abonnement public. Service Bus est utilisé pour découpler les applications et les services les uns des autres pour offrir les avantages suivants :

- Travail d’équilibrage de charge entre les workers concurrents
- Routage et transfert de façon sécurisée des données et du contrôle au-delà des limites des services et des applications
- Coordination du travail transactionnel qui nécessite un degré élevé de fiabilité 

## <a name="overview"></a>Vue d’ensemble
Les données sont transférées entre différents services et applications à l’aide de *messages*. Un message est un conteneur décoré de métadonnées, qui contient des données. Les données peuvent être n’importe quel type d’informations, y compris des données structurées, codées aux formats courants comme ceux-ci : JSON, XML, Apache Avro, Texte brut.

Les scénarios de messagerie courants sont :

* *Messagerie*. transfert de données d’entreprise, telles que les ventes ou les bons de commande, les journaux ou les mouvements de stock.
* *Découplage d’applications* : amélioration de la fiabilité et de l’extensibilité de vos applications et services. Il n’est pas nécessaire que les producteurs et les consommateurs soient en ligne ou immédiatement disponibles en même temps. La charge [est nivelée](/azure/architecture/patterns/queue-based-load-leveling) de telle sorte que les pics de trafic ne surtaxent pas un service. 
* *Équilibrage de la charge*. Possibilité pour plusieurs [consommateurs concurrents](/azure/architecture/patterns/competing-consumers) de lire dans une file d’attente en même temps, chacun obtenant de façon sécurisée la propriété exclusive sur des messages spécifiques. 
* *Rubriques et abonnements* : Activation des relations 1:*n* entre [les éditeurs et les abonnés](/azure/architecture/patterns/publisher-subscriber), ce qui permet aux abonnés de sélectionner des messages particuliers dans un flux de messages publiés.
* *Transactions*. Possibilité d’effectuer plusieurs opérations, toutes dans l’étendue d’une transaction atomique. Par exemple, les opérations suivantes peuvent être effectuées dans l’étendue d’une transaction.  

    1. Obtenir un message depuis une file d’attente.
    2. Publier les résultats du traitement dans une ou plusieurs files d’attente différentes.
    3. Déplacer le message d’entrée à partir de la file d’attente d’origine. 
    
    Pour les consommateurs en aval, les résultats ne sont visibles qu’en cas de réussite, ce qui comprend le règlement du message d’entrée autorisant la sémantique de traitement une fois seulement. Ce modèle de transaction constitue une base solide pour le modèle des [transactions de compensation](https://docs.microsoft.com/azure/architecture/patterns/compensating-transaction) dans le contexte de solution supérieur. 
* *Sessions de messagerie* : Implémentez une coordination à grande échelle des flux de travail et des transferts multiplexés qui nécessitent un classement strict des messages ou le report des messages.

Si vous êtes familiarisé avec d’autres répartiteurs de messages, comme Apache ActiveMQ, les concepts de Service Bus sont similaires à ceux que vous connaissez. Service Bus étant une offre PaaS (Platform-as-a-service), la grande différence se situe dans les actions suivantes, dont vous n’avez pas à vous soucier. Azure s’occupe de ces tâches pour vous. 

- Placement des journaux et gestion de l’espace disque
- Gestion des sauvegardes
- Mise à jour des systèmes d’exploitation ou des produits
- Inquiétude par rapport aux défaillances matérielles 
- Basculement vers une machine de réserve

## <a name="compliance-with-standards-and-protocols"></a>Conformité avec les normes et les protocoles

Le protocole filaire principal de Service Bus est [AMQP (Advanced Messaging Queueing Protocol) 1.0](service-bus-amqp-overview.md), une norme ISO/IEC ouverte. Il permet aux clients d’écrire des applications qui fonctionnent sur Service Bus et sur des répartiteurs locaux, tels que ActiveMQ ou RabbitMQ. Le [Guide du protocole AMQP](service-bus-amqp-protocol-guide.md) fournit des informations détaillées au cas où vous souhaiteriez élaborer une telle abstraction.

[Service Bus Premium](service-bus-premium-messaging.md) est entièrement compatible avec l’API [Java Message Service (JMS) 2.0](how-to-use-java-message-service-20.md) de Java/Jakarta EE. De plus, Service Bus standard prend en charge le sous-ensemble JMS 1.1 axé sur les files d’attente. JMS est une abstraction courante pour les répartiteurs de messages, elle s’intègre à de nombreuses applications et frameworks, y compris le populaire framework Spring. Pour passer d’un autre répartiteur à Azure Service Bus, il vous suffit de recréer la topologie des files d’attente et des rubriques, puis de modifier les dépendances et la configuration du fournisseur client. Pour consulter un exemple, reportez-vous au [Guide de migration d’ActiveMQ](migrate-jms-activemq-to-servicebus.md).

## <a name="concepts-and-terminology"></a>Concepts et terminologie 
Cette section traite des concepts et de la terminologie de Service Bus.

### <a name="namespaces"></a>Espaces de noms
Un espace de noms est un conteneur pour tous les composants de messagerie. Plusieurs files d’attente et rubriques peuvent résider dans un seul espace de noms, et les espaces de noms servent souvent de conteneurs d’applications. 

Un espace de noms peut être comparé à un « serveur » dans la terminologie d’autres répartiteurs, mais les concepts ne sont pas directement équivalents. Un espace de noms Service Bus représente votre portion de capacité propre d’un grand cluster constitué de dizaines de machines virtuelles actives. Il peut éventuellement s’étendre sur trois [zones de disponibilité Azure](../availability-zones/az-overview.md). Vous bénéficiez ainsi de tous les avantages de la disponibilité et de la robustesse qui sont liés à l’exécution du répartiteur de messages à très grande échelle. Qui plus est, vous n’avez pas à vous soucier des complexités sous-jacentes. Service Bus est une messagerie « sans serveur ».

### <a name="queues"></a>Files d’attente
Les messages sont envoyés vers et reçus depuis les *files d’attente*. Les files d’attente stockent les messages jusqu’à ce que l’application réceptrice soit disponible pour recevoir et traiter ces messages.

![File d'attente](./media/service-bus-messaging-overview/about-service-bus-queue.png)

Les messages dans les files d’attente sont classés et horodatés à l’arrivée. Une fois accepté par le répartiteur, le message est toujours conservé durablement dans le stockage à triple redondance, réparti entre les zones de disponibilité si l’espace de noms est activé interzone. Service Bus ne laisse jamais des messages en mémoire ni dans un stockage volatile lorsqu’ils ont été signalés au client comme acceptés.

Les messages sont remis en mode *tirage (pull)* , qui remet uniquement les messages lorsqu’ils sont demandés. Contrairement au modèle busy-polling de certaines autres files d’attente cloud, l’opération de tirage peut être longue et ne se terminer qu’une fois un message disponible. 

### <a name="topics"></a>Rubriques

Vous pouvez également utiliser des *rubriques* pour envoyer et recevoir des messages. Si une file d’attente est souvent utilisée pour la communication point à point, les rubriques sont utiles dans les scénarios de publication/abonnement.

![Rubrique](./media/service-bus-messaging-overview/about-service-bus-topic.png)

Les rubriques peuvent avoir plusieurs abonnements indépendants, qui s’attachent à la rubrique et fonctionnent exactement comme des files d’attente côté destinataire. Un abonné à une rubrique peut recevoir une copie de chaque message envoyé à cette rubrique. Les abonnements sont des entités nommées. Les abonnements sont durables par défaut, mais ils peuvent être configurés pour expirer, puis être automatiquement supprimés. Par le biais de l’API JMS, Service Bus Premium vous permet également de créer des abonnements volatiles qui existent le temps de la connexion.

Vous pouvez définir des règles sur un abonnement. Une règle d’abonnement est dotée d’un *filtre* permettant de définir une condition afin que le message soit copié dans l’abonnement, et d’une *action* facultative qui peut modifier les métadonnées du message. Si vous souhaitez en savoir plus, veuillez consulter l’article [Actions et filtres de rubrique](topic-filters.md). Cette fonctionnalité est utile dans les scénarios suivants :

- Lorsqu’il n’est pas souhaitable qu’un abonnement reçoive tous les messages envoyés à une rubrique.
- Quand il est nécessaire de marquer les messages avec des métadonnées supplémentaires lors du passage par un abonnement.

## <a name="advanced-features"></a>Fonctionnalités avancées

Certaines fonctionnalités avancées de Service Bus sont utiles pour résoudre des problèmes de messagerie plus complexes. Les sections suivantes détaillent plusieurs de ces fonctionnalités.

### <a name="message-sessions"></a>Sessions de message

Pour créer une garantie FIFO (pour « First In First Out », ou « Premier entré, premier sorti ») dans Service Bus, utilisez des sessions. Les sessions de message permettent un traitement exclusif et chronologique de séquences illimitées de messages associés. Pour permettre la gestion des sessions dans les systèmes à grande échelle et à haute disponibilité, la fonctionnalité de session permet également de stocker l’état de session, ce qui permet aux sessions de se déplacer de façon sécurisée entre les gestionnaires. Si vous souhaitez en savoir plus, veuillez consulter l’article [Sessions de messagerie : premier entré, premier sorti (FIFO)](message-sessions.md).

### <a name="autoforwarding"></a>Transfert automatique

La fonctionnalité de transfert automatique crée une chaîne reliant une file d’attente ou un abonnement à une autre file d’attente ou rubrique à l’intérieur du même espace de noms. Lorsque vous utilisez cette fonctionnalité, Service Bus déplace automatiquement les messages d’une file d’attente ou d’un abonnement dans une file d’attente ou rubrique cible. Tous ces déplacements sont effectués de manière transactionnelle. Si vous souhaitez en savoir plus, veuillez consulter l’article [Chaînage des entités Service Bus avec transfert automatique](service-bus-auto-forwarding.md).

### <a name="dead-letter-queue"></a>File d’attente de lettres mortes

Tous les abonnements aux rubriques et toutes les files d’attente Service Bus disposent d’une file d’attente de lettres mortes associée. Une file d’attente de lettres mortes contient les messages qui répondent aux critères suivants : 

- Ils ne peuvent pas être remis correctement à un destinataire.
- Leur délai d’attente a expiré.
- Ils sont explicitement écartés par l’application destinataire. 

Les messages, dans la file d’attente de lettres mortes, sont annotés selon la raison pour laquelle ils y ont été placés. La file d’attente de lettres mortes possède un point de terminaison particulier, mais agit sinon comme n’importe quelle file d’attente normale. Une application ou un outil peut parcourir une file d’attente de lettres mortes, ou enlever des message de celle-ci. Vous pouvez également procéder à un transfert automatique à partir d’une file d’attente de lettres mortes. Pour en savoir plus, consultez [Vue d’ensemble des files d’attente de lettres mortes Service Bus](service-bus-dead-letter-queues.md).

### <a name="scheduled-delivery"></a>Remise planifiée

Vous pouvez envoyer des messages à une file d’attente ou à une rubrique pour un traitement différé, en définissant une heure à laquelle le message sera disponible à la consommation. Les messages planifiés peuvent également être annulés. Si vous souhaitez en savoir plus, veuillez consulter la rubrique [Messages planifiés](message-sequencing.md#scheduled-messages).

### <a name="message-deferral"></a>Report de message

Un client de file d’attente ou d’abonnement peut reporter la récupération d’un message reçu à une date ultérieure définie. Le message a peut-être été publié selon un ordre attendu, et le client veut attendre de recevoir un autre message. Les messages différés restent dans la file d’attente ou l’abonnement ; ils doivent être réactivés explicitement au moyen de leur numéro de séquence assigné par le service. Si vous souhaitez en savoir plus, veuillez consulter l’article [Report de message](message-deferral.md).

### <a name="batching"></a>Traitement par lot

Le traitement par lot côté client permet à un client de file d’attente ou de rubrique d’accumuler un ensemble de messages pour les transférer ensemble. Cette pratique est souvent suivie pour économiser la bande passante ou augmenter le débit. Si vous souhaitez en savoir plus, veuillez consulter la rubrique [Traitement par lots côté client](service-bus-performance-improvements.md#client-side-batching).

### <a name="transactions"></a>Transactions

Une transaction regroupe deux ou plusieurs opérations dans une *étendue d’exécution*. Service Bus vous permet de grouper les opérations par rapport à plusieurs entités de messagerie dans l’étendue d’une seule transaction. Une entité de message peut être une file d’attente, une rubrique ou un abonnement. Si vous souhaitez en savoir plus, veuillez consulter l’article [Vue d’ensemble du traitement des transactions Service Bus](service-bus-transactions.md).

### <a name="autodelete-on-idle"></a>Suppression automatique en cas d’inactivité

La fonctionnalité de suppression automatique en cas d’inactivité vous permet de spécifier un intervalle d’inactivité à l’issue duquel une file d’attente ou un abonnement à une rubrique est automatiquement supprimé. La durée minimale est de 5 minutes. Si vous souhaitez en savoir plus, veuillez consulter l’article [Propriété QueueDescription.AutoDeleteOnIdle](/dotnet/api/microsoft.servicebus.messaging.queuedescription.autodeleteonidle).

### <a name="duplicate-detection"></a>Détection des doublons
La fonctionnalité de détection des doublons permet à l’expéditeur de renvoyer le même message une nouvelle fois, et au répartiteur d’éliminer un doublon potentiel. La détection des doublons est basée sur le suivi de la propriété `message-id` d’un message, ce qui signifie que l’application doit veiller à utiliser la même valeur lors du renvoi du message, qui peut peut-être dérivé directement d’un contexte propre à l’application. Pour en savoir plus, consultez [Détection des doublons](duplicate-detection.md).

### <a name="geo-disaster-recovery"></a>Géorécupération d’urgence

Lorsqu’une région Azure subit un temps d’arrêt, la fonctionnalité de reprise d’activité après sinistre permet au traitement des données de continuer à fonctionner dans un autre centre de données ou région. La fonctionnalité conserve le miroir structurel d’un espace de noms disponible dans la région secondaire, et permet à l’identité de l’espace de noms de basculer vers l’espace de noms secondaire. Les messages déjà publiés sont conservés dans l’ancien espace de noms principal, en vue de la reprise d’activité une fois l’incident de disponibilité éloigné. Pour plus d’informations, consultez [Géorécupération d’urgence Azure Service Bus](service-bus-geo-dr.md).

### <a name="security"></a>Sécurité

Service Bus prend en charge les protocoles standard [AMQP 1.0](service-bus-amqp-overview.md) et [HTTP ou REST](/rest/api/servicebus/) ainsi que leurs installations de sécurité respectives, y compris la sécurité au niveau du transport (TLS). Les clients peuvent être autorisés à accéder par l’intermédiaire de la sécurité basée sur les rôles [Signature d’accès partagé](service-bus-sas.md) ou [Azure Active Directory](service-bus-authentication-and-authorization.md). 

Pour la protection contre le trafic indésirable, Service Bus fournit des [fonctionnalités de sécurité](network-security.md), telles que le pare-feu IP et l’intégration aux réseaux virtuels. 

## <a name="client-libraries"></a>Bibliothèques clientes

Les bibliothèques de client Service Bus entièrement prises en charge sont disponibles par le biais du kit SDK Azure.

- [Azure Service Bus pour .NET](/dotnet/api/overview/azure/service-bus?preserve-view=true&view=azure-dotnet)
- [Bibliothèques Azure Service Bus pour Java](/java/api/overview/azure/servicebus?preserve-view=true&view=azure-java-stable)
- [Fournisseur Azure Service Bus pour Java JMS 2.0](how-to-use-java-message-service-20.md)
- [Modules Azure Service Bus pour JavaScript et TypeScript](/javascript/api/overview/azure/service-bus?preserve-view=true&view=azure-node-latest)
- [Bibliothèques Azure Service Bus pour Python](/python/api/overview/azure/servicebus?preserve-view=true&view=azure-python)

[Le protocole principal d’Azure Service Bus est AMQP 1.0](service-bus-amqp-overview.md) ; il est utilisable depuis n’importe quel client de protocole compatible AMQP 1.0. Quelques clients AMQP open source offrent des exemples illustrant de manière explicite l’interopérabilité de Service Bus. Consultez le [Guide du protocole AMQP 1.0](service-bus-amqp-protocol-guide.md) pour comprendre comment utiliser les fonctionnalités Service Bus directement avec les clients AMQP 1.0.

[!INCLUDE [messaging-oss-amqp-stacks.md](../../includes/messaging-oss-amqp-stacks.md)]

## <a name="integration"></a>Intégration

Service Bus s’intègre entièrement à de nombreux services Microsoft et Azure, par exemple :

* [Event Grid](https://azure.microsoft.com/services/event-grid/)
* [Logic Apps](https://azure.microsoft.com/services/logic-apps/)
* [Azure Functions](https://azure.microsoft.com/services/functions/)
* [Power Platform](https://powerplatform.microsoft.com/)
* [Dynamics 365](https://dynamics.microsoft.com)
* [Azure Stream Analytics](https://azure.microsoft.com/services/stream-analytics/)

## <a name="next-steps"></a>Étapes suivantes

Pour commencer à utiliser la messagerie Service Bus, consultez les articles suivants :

* Si vous souhaitez comparer les services de messagerie Azure, veuillez consulter l’article [Choisissez entre les trois services de messagerie Azure - Event Grid, Event Hubs et Service Bus](../event-grid/compare-messaging-services.md?toc=%2fazure%2fservice-bus-messaging%2ftoc.json&bc=%2fazure%2fservice-bus-messaging%2fbreadcrumb%2ftoc.json).
* Essayez les Guides de démarrage rapide pour [.NET](service-bus-dotnet-get-started-with-queues.md), [Java](service-bus-java-how-to-use-queues.md) ou [JMS](service-bus-java-how-to-use-jms-api-amqp.md).
* Si vous devez gérer des ressources Service Bus, veuillez consulter le référentiel GitHub [Explorateur Service Bus](https://github.com/paolosalvatori/ServiceBusExplorer/releases) (contenu en anglais).
* Si vous souhaitez en savoir plus sur les niveaux Standard et Premium et leur tarification, veuillez consulter la page [Tarification Service Bus](https://azure.microsoft.com/pricing/details/service-bus/).
* Si vous souhaitez en savoir plus sur les performances et la latence du niveau Premium, veuillez consulter le billet de blog [Messagerie Premium](https://techcommunity.microsoft.com/t5/Service-Bus-blog/Premium-Messaging-How-fast-is-it/ba-p/370722) (contenu en anglais).