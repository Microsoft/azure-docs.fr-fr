---
title: Recevoir des événements à l’aide de l’hôte de processeur d’événements - Azure Event Hubs | Microsoft Docs
description: Cet article décrit l’hôte de processeur d’événements d’Azure Event Hubs, qui simplifie la gestion des points de contrôle, de la location et des lecteurs d’événements parallèles.
ms.topic: conceptual
ms.date: 06/23/2020
ms.custom: devx-track-csharp
ms.openlocfilehash: a05f2172b266301919d0a800fb863b8f0dbe5884
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89319500"
---
# <a name="event-processor-host"></a>Hôte du processeur d’événements
> [!NOTE]
> Cet article s’applique à l’ancienne version du kit de développement logiciel (SDK) Azure Event Hubs. Pour une version actuelle du Kit de développement logiciel (SDK), consultez [Équilibrer la charge de partition sur plusieurs instances de votre application](event-processor-balance-partition-load.md). Pour savoir comment migrer votre code vers la version la plus récente du kit de développement logiciel (SDK), consultez ces guides de migration. 
> - [.NET](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/eventhub/Azure.Messaging.EventHubs/MigrationGuide.md)
> - [Java](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/eventhubs/azure-messaging-eventhubs/migration-guide.md)
> - [Python](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/eventhub/azure-eventhub/migration_guide.md)
> - [Script Java](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/eventhub/event-hubs/migrationguide.md)

Azure Event Hubs est un puissant service d’ingestion de télémétrie qui permet de diffuser des millions d’événements à moindre coût. Cet article décrit comment consommer les événements ingérés à l’aide de l’*hôte de processeur d’événements* (EPH) ; un agent consommateur intelligent qui simplifie la gestion des points de contrôle, de la location et des lecteurs d’événements parallèles.  

La clé de la mise à l’échelle des instances Event Hubs réside dans la notion de consommateurs partitionnés. Contrairement au modèle de [consommateurs concurrents](/previous-versions/msp-n-p/dn568101(v=pandp.10)), le modèle de consommateurs partitionnés permet de travailler à grande échelle en supprimant le goulot d’étranglement de contention et en facilitant le parallélisme de bout en bout.

## <a name="home-security-scenario"></a>Scénario de sécurité pour domicile

Comme exemple de scénario, prenons une société de sécurité qui surveille 100 000 maisons. Chaque minute, elle obtient des données de différents capteurs (détecteur de mouvement, capteur d’ouverture de porte/fenêtre, détecteur de bris de verre, etc.) installés dans chaque maison. La société fournit un site web sur lequel les résidents peuvent surveiller l’activité de la maison en temps quasi réel.

Chaque capteur envoie des données à un hub d’événements. Le hub d’événements est configuré avec 16 partitions. Côté consommation, vous avez besoin d’un mécanisme qui peut lire ces événements, les consolider (filtrage, agrégation, etc.) et sauvegarder l’agrégat dans un objet blob de stockage, qui est ensuite projeté sur une page web conviviale.

## <a name="write-the-consumer-application"></a>Écrire l’application consommateur

Lorsque vous concevez le consommateur dans un environnement distribué, le scénario doit gérer les exigences suivantes :

1. **Mise à l’échelle :** créez plusieurs consommateurs, chacun d’entre eux assurant la lecture à partir de quelques partitions Event Hubs.
2. **Équilibrage de charge :** augmentez ou réduisez le nombre de consommateurs dynamiquement. Par exemple, lorsqu’un nouveau type de capteur (comme un détecteur de monoxyde de carbone) est ajouté à chaque maison, le nombre d’événements augmente. Dans ce cas, l’opérateur (un humain) augmente le nombre d’instances de consommateur. Ensuite, le pool de consommateurs peut rééquilibrer le nombre de partitions pour partager la charge avec les consommateurs qui viennent d’être ajoutés.
3. **Reprise transparente en cas d’échec :** si un consommateur (**consommateur A**) échoue (par exemple, la machine virtuelle qui héberge le consommateur tombe soudainement en panne), les autres consommateurs doivent être en mesure de récupérer les partitions appartenant au **consommateur A** et continuer. En outre, le point de continuation, appelé *point de contrôle* ou *décalage*, doit se trouver exactement là où le **consommateur A** a échoué ou légèrement avant.
4. **Consommer des événements :** alors que les trois points précédents concernent la gestion du consommateur, il doit y avoir du code pour consommer les événements et faire quelque chose d’utile avec eux ; par exemple, les agréger et les télécharger vers le stockage blob.

Au lieu de générer votre propre solution, Event Hubs fournit cette fonctionnalité via l’interface [IEventProcessor](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor) et la classe [EventProcessorHost](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessorhost).

## <a name="ieventprocessor-interface"></a>Interface IEventProcessor

Tout d’abord, les applications de consommation implémentent l’interface [IEventProcessor](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor), qui comporte quatre méthodes : [OpenAsync, CloseAsync, ProcessErrorAsync et ProcessEventsAsync](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor?view=azure-dotnet#methods). Cette interface contient le code réel permettant de consommer les événements envoyés par Event Hubs. Le code suivant montre une implémentation simple :

```csharp
public class SimpleEventProcessor : IEventProcessor
{
    public Task CloseAsync(PartitionContext context, CloseReason reason)
    {
       Console.WriteLine($"Processor Shutting Down. Partition '{context.PartitionId}', Reason: '{reason}'.");
       return Task.CompletedTask;
    }

    public Task OpenAsync(PartitionContext context)
    {
       Console.WriteLine($"SimpleEventProcessor initialized. Partition: '{context.PartitionId}'");
       return Task.CompletedTask;
     }

    public Task ProcessErrorAsync(PartitionContext context, Exception error)
    {
       Console.WriteLine($"Error on Partition: {context.PartitionId}, Error: {error.Message}");
       return Task.CompletedTask;
    }

    public Task ProcessEventsAsync(PartitionContext context, IEnumerable<EventData> messages)
    {
       foreach (var eventData in messages)
       {
          var data = Encoding.UTF8.GetString(eventData.Body.Array, eventData.Body.Offset, eventData.Body.Count);
             Console.WriteLine($"Message received. Partition: '{context.PartitionId}', Data: '{data}'");
       }
       return context.CheckpointAsync();
    }
}
```

Ensuite, créez une instance [EventProcessorHost](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessorhost). Selon la surcharge, lorsque vous créez l’instance [EventProcessorHost](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessorhost) dans le constructeur, les paramètres suivants sont utilisés :

- **hostName :** nom de chaque instance de consommateur. Chaque instance d’**EventProcessorHost** doit avoir une valeur unique pour cette variable dans un groupe de consommateurs ; veillez donc à ne pas coder cette valeur en dur.
- **eventHubPath :** Nom du hub d’événements.
- **consumerGroupName :** Event Hubs utilise **$Default** comme nom du groupe de consommateurs par défaut, mais il est conseillé de créer un groupe de consommateurs pour votre aspect spécifique du traitement.
- **eventHubConnectionString :** chaîne de connexion au hub d’événements, qui peut être récupérée à partir du Portail Azure. Cette chaîne de connexion doit disposer d’autorisations d’**écoute** sur le hub d’événements.
- **storageConnectionString :** compte de stockage utilisé pour la gestion des ressources internes.

Enfin, les consommateurs inscrivent l’instance [EventProcessorHost](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessorhost) auprès du service Event Hubs. L’inscription d’une classe de processeur d’événements avec une instance EventProcessorHost entraîne le démarrage du traitement des événements. L’inscription ordonne au service Event Hubs d’attendre que l’application consommateur consomme les événements provenant de certaines de ses partitions, puis d’appeler le code d’implémentation [IEventProcessor](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor) chaque fois qu’il envoie (push) des événements à consommer. 

> [!NOTE]
> La valeur de ConsumerGroupName respecte la casse.  Les modifications apportées à ConsumerGroupName peuvent entraîner la lecture de toutes les partitions à partir du début du flux.

### <a name="example"></a>Exemple

Par exemple, imaginez qu’il y a 5 machines virtuelles dédiées à la consommation d’événements et une simple application console dans chaque machine virtuelle, qui effectue les tâches de consommation proprement dites. Chaque application console crée alors une instance [EventProcessorHost](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessorhost) et l’inscrit auprès du service Event Hubs.

Dans cet exemple de scénario, supposons que 16 partitions sont allouées aux 5 instances **EventProcessorHost**. Certaines instances **EventProcessorHost** peuvent posséder quelques partitions de plus que d’autres. Pour chacune de ses partitions, l’instance **EventProcessorHost** crée une instance de la classe `SimpleEventProcessor`. Par conséquent, il existe 16 instances de `SimpleEventProcessor` en tout (une affectée à chaque partition).

La liste suivante récapitule cet exemple :

- 16 partitions Event Hubs.
- 5 machines virtuelles, 1 application consommateur (par exemple, Consumer.exe) dans chaque machine virtuelle.
- 5 instances EPH inscrites, 1 dans chaque machine virtuelle par Consumer.exe.
- 16 objets `SimpleEventProcessor` créés par les 5 instances EPH.
- Une application Consumer.exe peut contenir 4 objets `SimpleEventProcessor`, dans la mesure où une instance EPH peut posséder 4 partitions.

## <a name="partition-ownership-tracking"></a>Suivi de la propriété d’une partition

La propriété d’une partition pour une instance EPH (ou un consommateur) est suivie par le biais du compte de stockage Azure fourni pour le suivi. Vous pouvez visualiser le suivi en tant que simple tableau, comme suit. Vous pouvez voir l’implémentation réelle en examinant les objets BLOB sous le compte de stockage spécifié :

| **Nom du groupe de consommateurs** | **ID de partition** | **Nom d’hôte (propriétaire)** | **Date d’acquisition du bail (ou la propriété)** | **Décalage dans la partition (point de contrôle)** |
| --- | --- | --- | --- | --- |
| $Default | 0 | Consumer\_VM3 | 2018-04-15T01:23:45 | 156 |
| $Default | 1 | Consumer\_VM4 | 2018-04-15T01:22:13 | 734 |
| $Default | 2 | Consumer\_VM0 | 2018-04-15T01:22:56 | 122 |
| : |   |   |   |   |
| : |   |   |   |   |
| $Default | 15 | Consumer\_VM3 | 2018-04-15T01:22:56 | 976 |

Ici, chaque hôte acquiert la possession d’une partition pour une certaine durée (la durée du bail). Si un hôte échoue (la machine virtuelle s’arrête), le bail expire. D’autres hôtes tentent d’obtenir la propriété de la partition, et un des hôtes réussit. Ce processus réinitialise le bail sur la partition avec un nouveau propriétaire. De cette façon, un seul lecteur à la fois peut utiliser une partition donnée au sein d’un groupe de consommateurs.

## <a name="receive-messages"></a>Recevoir des messages

Chaque appel à [ProcessEventsAsync](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor.processeventsasync) offre une collection d’événements. Il vous incombe de gérer ces événements. Si vous souhaitez vous assurer que l’hôte du processeur traite chaque message au moins une fois, vous devez écrire votre propre code de nouvelle tentative. Méfiez-vous cependant des messages empoisonnés.

Il est recommandé de procéder relativement vite. Autrement dit, de faire aussi peu de traitement que possible. Pour cela, utilisez des groupes de consommateurs. Si vous avez besoin d’écrire dans le stockage et de procéder à un routage, mieux vaut utiliser deux groupes de consommateurs et avoir deux implémentations [IEventProcessor](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor) qui s’exécutent séparément.

À un moment donné pendant le traitement, vous souhaiterez peut-être effectuer le suivi de ce que vous avez lu et accompli. Cela est essentiel si vous devez redémarrer la lecture, afin de ne pas revenir au début du flux de données. [EventProcessorHost](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessorhost) simplifie ce suivi à l’aide de *points de contrôle*. Un point de contrôle est un emplacement (ou un décalage) d’une partition donnée au sein d’un groupe de consommateurs, où vous êtes satisfait du traitement des messages. La création d’un point de contrôle dans **EventProcessorHost** nécessite d’appeler la méthode [CheckpointAsync](/dotnet/api/microsoft.azure.eventhubs.processor.partitioncontext.checkpointasync) sur l’objet [PartitionContext](/dotnet/api/microsoft.azure.eventhubs.processor.partitioncontext). Cette opération s’effectue dans la méthode [ProcessEventsAsync](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor.processeventsasync), mais également dans [CloseAsync](/dotnet/api/microsoft.azure.eventhubs.eventhubclient.closeasync).

## <a name="checkpointing"></a>Points de contrôle

La méthode [CheckpointAsync](/dotnet/api/microsoft.azure.eventhubs.processor.partitioncontext.checkpointasync) a deux surcharges : la première, sans paramètres, crée un point de contrôle au décalage d’événement le plus élevé au sein de la collection renvoyée par [ProcessEventsAsync](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor.processeventsasync). Ce décalage est une marque de « borne haute » ; il part du principe que vous avez traité tous les événements récents lorsque vous l’appelez. Si vous utilisez la méthode de cette façon, n’oubliez pas que vous êtes censé l’appeler une fois que votre autre code de traitement d’événements a été renvoyé. La deuxième surcharge vous permet de spécifier une instance [EventData](/dotnet/api/microsoft.azure.eventhubs.eventdata) pour le point de contrôle. Cette méthode vous permet d’utiliser un autre type de marque pour le point de contrôle. Avec cette marque, vous pouvez implémenter une marque de « niveau faible » : l’événement séquencé le plus bas qui a été traité. Cette surcharge est fournie pour permettre une certaine flexibilité dans la gestion du décalage.

Lorsque le point de contrôle est créé, un fichier JSON contenant des informations propres à la partition (plus précisément, le décalage) est écrit dans le compte de stockage fourni dans le constructeur pour [EventProcessorHost](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessorhost). Ce fichier est continuellement mis à jour. Il est essentiel à prendre en compte le contexte dan la création des points de contrôle : il est déconseillé de créer un point de contrôle pour chaque message. Le compte de stockage utilisé pour la création des points de contrôle ne gère probablement pas cette charge. Mais, plus important encore, créer un point de contrôle pour chaque événement est signe d’un modèle de messagerie avec file d’attente pour lequel une file d’attente Service Bus peut être une meilleure option qu’un hub d’événements. L’idée derrière Event Hubs est que vous obtenez une livraison « au moins une fois » à grande échelle. En octroyant la même puissance à vos systèmes en aval, il est facile de récupérer après des pannes ou des redémarrages qui se traduisent par la réception répétée des mêmes événements.

## <a name="thread-safety-and-processor-instances"></a>Cohérence de thread et instances de processeur

Par défaut, [EventProcessorHost](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessorhost) est thread-safe et se comporte de façon synchrone en ce qui concerne l’instance de [IEventProcessor](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor). Lorsque des événements arrivent pour une partition, [ProcessEventsAsync](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor.processeventsasync) est appelée sur l’instance **IEventProcessor** de cette partition et bloque les autres appels destinés à l’instance **ProcessEventsAsync** pour la partition. Les messages et appels suivants destinés à **ProcessEventsAsync** sont placés en file d’attente en arrière-plan tandis que l’acquisition des messages continue à s’exécuter en arrière-plan sur d’autres threads. Avec cette cohérence de thread, les collections thread-safe ne sont plus nécessaires, et les performances augmentent considérablement.

## <a name="shut-down-gracefully"></a>Arrêt correct

Enfin, [EventProcessorHost.UnregisterEventProcessorAsync](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessorhost.unregistereventprocessorasync) permet un arrêt normal de tous les lecteurs de partition et doit toujours être appelé lorsque vous arrêtez une instance de [EventProcessorHost](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessorhost). Dans le cas contraire, cela peut provoquer des retards lors du démarrage d’autres instances de **EventProcessorHost** en raison de l’expiration du bail et de conflits d’époque. La gestion de l’époque est abordée en détail dans la section [Époque](#epoch) de cet article. 

## <a name="lease-management"></a>Gestion de bail
L’inscription d’une classe de processeur d’événements avec une instance EventProcessorHost entraîne le démarrage du traitement des événements. L’instance d’hôte obtient des baux sur certaines partitions d’Event Hub, probablement en en récupérant certaines à partir d’autres instances d’hôte, d’une manière qui converge vers une répartition uniforme des partitions dans toutes les instances d’hôte. Pour chaque partition allouée par bail, l’instance d’hôte crée une instance de la classe de processeur d’événements fournie, puis reçoit les événements de cette partition et les transmet à l’instance du processeur d’événements. Lorsque d’autres instances sont ajoutées et d’autres baux sont récupérés, EventProcessorHost finit par équilibrer la charge entre tous les consommateurs.

Comme expliqué précédemment, le tableau de suivi simplifie considérablement la nature de mise à l’échelle automatique de [EventProcessorHost.UnregisterEventProcessorAsync](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessorhost.unregistereventprocessorasync). Quand une instance de **EventProcessorHost** démarre, elle acquiert autant de baux que possible et commence la lecture des événements. Quand l’expiration des baux approche, **EventProcessorHost** tente de les renouveler en plaçant une réservation. Si le bail est disponible pour le renouvellement, le processeur poursuit la lecture, mais si ce n’est pas le cas, le lecteur est fermé et [CloseAsync](/dotnet/api/microsoft.azure.eventhubs.eventhubclient.closeasync) est appelé. **CloseAsync** est un bon moment pour effectuer un nettoyage final pour cette partition.

**EventProcessorHost** inclut une propriété [PartitionManagerOptions](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessorhost.partitionmanageroptions). Cette propriété permet de contrôler la gestion de bail. Définissez ces options avant d’inscrire votre implémentation [IEventProcessor](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor).

## <a name="control-event-processor-host-options"></a>Options de contrôle pour l’hôte du processeur d’événements

En outre, une surcharge de [RegisterEventProcessorAsync](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessorhost.registereventprocessorasync?view=azure-dotnet#Microsoft_Azure_EventHubs_Processor_EventProcessorHost_RegisterEventProcessorAsync__1_Microsoft_Azure_EventHubs_Processor_EventProcessorOptions_) prend un objet [EventProcessorOptions](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessorhost.registereventprocessorasync?view=azure-dotnet#Microsoft_Azure_EventHubs_Processor_EventProcessorHost_RegisterEventProcessorAsync__1_Microsoft_Azure_EventHubs_Processor_EventProcessorOptions_) comme paramètre. Utilisez ce paramètre pour contrôler le comportement de [EventProcessorHost.UnregisterEventProcessorAsync](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessorhost.unregistereventprocessorasync) lui-même. [EventProcessorOptions](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessoroptions) définit quatre propriétés et un événement :

- [MaxBatchSize](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessoroptions.maxbatchsize) : taille maximale de la collection que vous souhaitez recevoir dans un appel de [ProcessEventsAsync](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor.processeventsasync). Cette taille n’est pas la valeur minimale, mais seulement la taille maximale. Si moins de messages sont reçus, **ProcessEventsAsync** s’exécute avec tous les messages disponibles.
- [PrefetchCount](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessoroptions.prefetchcount) : valeur utilisée par le canal AMQP sous-jacent pour déterminer la limite supérieure du nombre de messages que le client doit recevoir. Cette valeur doit être supérieure ou égale à [MaxBatchSize](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessoroptions.maxbatchsize).
- [InvokeProcessorAfterReceiveTimeout](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessoroptions.invokeprocessorafterreceivetimeout) : si ce paramètre a la valeur **true**, [ProcessEventsAsync](/dotnet/api/microsoft.azure.eventhubs.processor.ieventprocessor.processeventsasync) est appelé lorsque l’appel sous-jacent servant à recevoir des événements sur une partition arrive à expiration. Cette méthode est utile pour réaliser les actions liées au temps pendant les périodes d’inactivité sur la partition.
- [InitialOffsetProvider](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessoroptions.initialoffsetprovider) : permet de définir un pointeur de fonction ou une expression lambda, qui est appelé pour fournir le décalage initial lorsqu’un lecteur commence à lire une partition. Sans spécifier ce décalage, le lecteur commence par l’événement le plus ancien, sauf si un fichier JSON avec un décalage a déjà été enregistré dans le compte de stockage fourni au constructeur [EventProcessorHost](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessorhost). Cette méthode est utile lorsque vous souhaitez modifier le comportement du démarrage du lecteur. Lorsque cette méthode est appelée, le paramètre d’objet contient l’ID de partition pour lequel le lecteur est démarré.
- [ExceptionReceivedEventArgs](/dotnet/api/microsoft.azure.eventhubs.processor.exceptionreceivedeventargs) : permet de recevoir une notification pour les exceptions sous-jacentes qui se produisent dans [EventProcessorHost](/dotnet/api/microsoft.azure.eventhubs.processor.eventprocessorhost). Si les choses ne fonctionnent pas comme prévu, cet événement est un bon point pour commencer la recherche.

## <a name="epoch"></a>Époque

Voici le fonctionnement de la réception d’époque :

### <a name="with-epoch"></a>Avec époque
L’époque est un identificateur unique (valeur d’époque) utilisé par le service pour appliquer la propriété de partition/bail. Vous créez un récepteur basé sur une époque à l’aide de la méthode [CreateEpochReceiver](/dotnet/api/microsoft.azure.eventhubs.eventhubclient.createepochreceiver?view=azure-dotnet). Cette méthode crée un récepteur basé sur une époque. Le récepteur est créé pour une partition de hub d’événements spécifique dans le groupe de consommateurs spécifié.

La fonctionnalité d’époque permet aux utilisateurs de s’assurer qu’il n’existe qu’un seul récepteur sur un groupe de consommateurs à tout moment dans le temps, avec les règles suivantes :

- S’il n’y a aucun récepteur existant sur un groupe de consommateurs, l’utilisateur peut créer un récepteur avec n’importe quelle valeur d’époque.
- S’il existe un récepteur avec une valeur d’époque e1 et qu’un récepteur est créé avec une valeur d’époque e2, où e1 <= e2, le récepteur avec e1 est automatiquement déconnecté et le récepteur avec e2 est créé avec succès.
- S’il existe un récepteur avec une valeur d’époque e1 et qu’un récepteur est créé avec une valeur d’époque e2, où e1 > e2, la création d’e2 échoue avec une erreur signalant qu’il existe déjà un récepteur avec l’époque e1.

### <a name="no-epoch"></a>Sans époque
Vous créez un récepteur non basé sur une époque à l’aide de la méthode [CreateReceiver](/dotnet/api/microsoft.azure.eventhubs.eventhubclient.createreceiver?view=azure-dotnet). 

Il existe certains scénarios de flux de traitement où les utilisateurs souhaitent créer plusieurs récepteurs sur un même groupe de consommateurs. Pour prendre en charge ces scénarios, nous pouvons créer un récepteur sans époque ; dans ce cas, nous autorisons jusqu’à cinq récepteurs simultanés sur le groupe de consommateurs.

### <a name="mixed-mode"></a>Mode mixte
Nous déconseillons les cas d’usage où vous créez un récepteur avec époque puis basculez vers un récepteur sans époque (ou inversement) sur le même groupe de consommateurs. Toutefois, quand ce comportement se présente, le service le gère en appliquant les règles suivantes :

- Si un récepteur a déjà été créé avec une époque e1 et reçoit activement des événements, et qu’un nouveau récepteur est créé sans époque, la création du nouveau récepteur échoue. Les récepteurs avec époque sont toujours prioritaires dans le système.
- S’il existait déjà un récepteur avec une époque e1 qu’il a été déconnecté, et qu’un récepteur est créé sans époque sur un nouveau MessagingFactory, la création du nouveau récepteur réussit. Il existe ici un inconvénient : notre système détectera la « déconnexion du récepteur » après environ 10 minutes.
- Si un ou plusieurs récepteurs ont été créés sans époque, et qu’un destinataire est créé avec l’époque e1, tous les anciens récepteurs sont déconnectés.


> [!NOTE]
> Nous recommandons d'utiliser différents groupes de consommateurs pour les applications qui utilisent des époques et pour celles qui n'en utilisent pas afin d'éviter les erreurs. 


## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous vous êtes familiarisé avec l’hôte du processeur d’événements, consultez les articles suivants pour en savoir plus sur Event Hubs :

- Prise en main des hubs d’événements
    - [.NET Core](event-hubs-dotnet-standard-getstarted-send.md)
    - [Java](event-hubs-java-get-started-send.md)
    - [Python](event-hubs-python-get-started-send.md)
    - [JavaScript](event-hubs-node-get-started-send.md)
* [Guide de programmation Event Hubs](event-hubs-programming-guide.md)
* [Disponibilité et cohérence dans Event Hubs](event-hubs-availability-and-consistency.md)
* [FAQ sur les hubs d’événements](event-hubs-faq.md)
* [Exemples Event Hubs sur GitHub](https://github.com/Azure/azure-event-hubs/tree/master/samples)
