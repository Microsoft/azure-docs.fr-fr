---
title: Équilibrer la charge de partition sur plusieurs instances – Azure Event Hubs | Microsoft Docs
description: Décrit comment équilibrer la charge de partition sur plusieurs instances de votre application à l’aide d’un processeur d’événements et du Kit de développement logiciel (SDK) Azure Event Hubs.
ms.topic: conceptual
ms.date: 06/23/2020
ms.openlocfilehash: af307058d0eda6b96c0811bccc245c09e2bdd27d
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95025042"
---
# <a name="balance-partition-load-across-multiple-instances-of-your-application"></a>Équilibrer la charge de partition sur plusieurs instances de votre application
Pour mettre à l’échelle votre application de traitement des événements, vous pouvez exécuter plusieurs instances de l’application et faire en sorte que celle-ci équilibre la charge entre elles. Dans les versions plus anciennes, [EventProcessorHost](event-hubs-event-processor-host.md) vous permettait d’équilibrer la charge entre plusieurs instances de votre programme et des événements de point de contrôle lors de la réception. Dans les versions plus récentes (à partir de 5.0), **EventProcessorClient** (.NET et Java) ou **EventHubConsumerClient** (Python et JavaScript) vous permet d’en faire de même. Le modèle de développement est simplifié par l’utilisation d’événements. Vous vous abonnez aux événements qui vous intéressent en inscrivant un gestionnaire d’événements. Si vous utilisez l’ancienne version de la bibliothèque de client, consultez les guides de migration suivants : [.NET](https://github.com/Azure/azure-sdk-for-net/blob/master/sdk/eventhub/Azure.Messaging.EventHubs/MigrationGuide.md), [Java](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/servicebus/azure-messaging-servicebus/migration-guide.md), [Python](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/servicebus/azure-servicebus/migration_guide.md)et [JavaScript](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/servicebus/service-bus/migrationguide.md).

Cet article décrit un exemple de scénario permettant d’utiliser plusieurs instances pour lire des événements à partir d’un hub d’événements, puis vous donner des détails sur les fonctionnalités du client du processeur d’événements, ce qui vous permet de recevoir des événements de plusieurs partitions en même temps et d’équilibrer la charge avec d’autres consommateurs qui utilisent les mêmes hub d’événements et groupe de consommateurs.

> [!NOTE]
> La clé de la mise à l’échelle des instances Event Hubs réside dans la notion de consommateurs partitionnés. Contrairement au modèle de [consommateurs concurrents](/previous-versions/msp-n-p/dn568101(v=pandp.10)), le modèle de consommateurs partitionnés permet de travailler à grande échelle en supprimant le goulot d’étranglement de contention et en facilitant le parallélisme de bout en bout.

## <a name="example-scenario"></a>Exemple de scénario

Comme exemple de scénario, prenons une société de sécurité qui surveille 100 000 maisons. À chaque minute, elle reçoit des données de différents capteurs (détecteur de mouvement, capteur d’ouverture de porte/fenêtre, détecteur de bris de verre, etc.) installés dans chaque maison. La société fournit un site web sur lequel les résidents peuvent surveiller l’activité de la maison en temps quasi réel.

Chaque capteur envoie des données à un hub d’événements. Le hub d’événements est configuré avec 16 partitions. Côté consommation, vous avez besoin d’un mécanisme pouvant lire ces événements, les consolider (filtrage, agrégation, etc.) et sauvegarder l’agrégat dans un blob de stockage qui est ensuite projeté sur une page web conviviale.

## <a name="write-the-consumer-application"></a>Écrire l’application consommateur

Lorsque vous concevez le consommateur dans un environnement distribué, le scénario doit gérer les exigences suivantes :

1. **Mise à l’échelle :** créez plusieurs consommateurs, chacun d’entre eux assurant la lecture à partir de quelques partitions Event Hubs.
2. **Équilibrage de charge :** augmentez ou réduisez le nombre de consommateurs dynamiquement. Par exemple, lorsqu’un nouveau type de capteur (comme un détecteur de monoxyde de carbone) est ajouté à chaque maison, le nombre d’événements augmente. Dans ce cas, l’opérateur (un humain) augmente le nombre d’instances de consommateur. Ensuite, le pool de consommateurs peut rééquilibrer le nombre de partitions pour partager la charge avec les consommateurs qui viennent d’être ajoutés.
3. **Reprise transparente en cas d’échec :** si un consommateur (**consommateur A**) échoue (par exemple, la machine virtuelle qui héberge le consommateur tombe soudainement en panne), d’autres consommateurs peuvent récupérer les partitions appartenant au **consommateur A** et continuer. En outre, le point de continuation, appelé *point de contrôle* ou *décalage*, doit se trouver exactement là où le **consommateur A** a échoué ou légèrement avant.
4. **Consommer des événements :** alors que les trois points précédents concernent la gestion du consommateur, il doit y avoir du code pour consommer les événements et en faire quelque chose d’utile. Par exemple, l’agréger et le charger dans le stockage d’objets blob.

## <a name="event-processor-or-consumer-client"></a>Processeur d’événements ou client consommateur

Vous n’avez pas besoin de créer votre propre solution pour répondre à ces exigences. Les kits de développement logiciel (SDK) Azure Event Hubs offrent cette fonctionnalité. Dans les kits de développement logiciel (SDK) .NET ou Java, vous utilisez un client de processeur d'événements (EventProcessorClient) et, dans les SDK Python et JavaScript, vous utilisez EventHubConsumerClient. Dans l’ancienne version du Kit de développement logiciel (SDK), c’était l’hôte du processeur d’événements (EventProcessorHost) qui prenait en charge ces fonctionnalités.

Pour la majorité des scénarios de production, nous vous recommandons d’utiliser le client de processeur d’événements pour la lecture et le traitement des événements. Le client de processeur est conçu pour fournir une expérience solide pour le traitement des événements dans toutes les partitions d’un hub d’événements de façon performante et tolérante aux pannes, tout en fournissant un moyen de marquer sa progression d’un point de contrôle. Les clients de processeur d’événements peuvent travailler de façon coopérative dans le contexte d’un groupe de consommateurs pour un hub d’événements donné. Les clients gèrent automatiquement la distribution et l’équilibrage du travail à mesure que des instances deviennent disponibles ou indisponibles pour le groupe.

## <a name="partition-ownership-tracking"></a>Suivi de la propriété d’une partition

Une instance de processeur d’événements possède généralement et traite des événements d’une ou plusieurs partitions. La propriété des partitions est répartie uniformément entre toutes les instances de processeur d’événements actives associées à une combinaison de hub d’événements et de groupe de consommateurs. 

Chaque processeur d’événements reçoit un identificateur unique et revendique la propriété des partitions en ajoutant ou en mettant à jour une entrée dans un magasin de point de contrôle. Toutes les instances de processeur d’événements communiquent périodiquement avec ce magasin pour mettre à jour leur propre état de traitement, ainsi que pour en savoir plus sur les autres instances actives. Ces données sont ensuite utilisées pour équilibrer la charge entre les processeurs actifs. De nouvelles instances peuvent rejoindre le pool de traitement pour le faire monter en puissance. Lorsque des instances s’arrêtent, en raison de défaillances ou d’une descente en puissance, la propriété de la partition est transférée proprement vers d’autres processeurs actifs.

Les enregistrements de propriété des partitions disponibles dans le magasin de points de contrôle permettent de suivre l’espace de noms Event Hubs, le nom du hub d’événements, le groupe de consommateurs, l’identificateur du processeur d’événements (également appelé propriétaire), l’ID de la partition et l’heure de la dernière modification.



| Espace de noms Event Hubs               | Nom de l’Event Hub | **Groupe de consommateurs** | Propriétaire                                | ID de partition (Partition ID) | Heure de dernière modification  |
| ---------------------------------- | -------------- | :----------------- | :----------------------------------- | :----------- | :------------------ |
| mynamespace.servicebus.windows.net | myeventhub     | myconsumergroup    | 3be3f9d3-9d9e-4c50-9491-85ece8334ff6 | 0            | 2020-01-15T01:22:15 |
| mynamespace.servicebus.windows.net | myeventhub     | myconsumergroup    | f5cc5176-ce96-4bb4-bbaa-a0e3a9054ecf | 1            | 2020-01-15T01:22:17 |
| mynamespace.servicebus.windows.net | myeventhub     | myconsumergroup    | 72b980e9-2efc-4ca7-ab1b-ffd7bece8472 | 2            | 2020-01-15T01:22:10 |
|                                    |                | :                  |                                      |              |                     |
|                                    |                | :                  |                                      |              |                     |
| mynamespace.servicebus.windows.net | myeventhub     | myconsumergroup    | 844bd8fb-1f3a-4580-984d-6324f9e208af | 15           | 2020-01-15T01:22:00 |

Chaque instance de processeur d’événements acquiert la propriété d’une partition et commence à traiter celle-ci à partir du dernier [point de contrôle](# Checkpointing) connu. En cas de défaillance d’un processeur (machine virtuelle qui s’arrête), d’autres instances détectent l’événement en examinant l’heure de dernière modification. D’autres instances essaient d’obtenir la propriété des partitions appartenant précédemment à l’instance inactive, et le magasin de points de contrôle garantit qu’une seule des instances réussit à revendiquer la propriété d’une partition. Ainsi, à un moment donné, il n’y a qu’un seul processeur recevant les événements d’une partition.

## <a name="receive-messages"></a>Recevoir des messages

Lorsque vous créez un processeur d’événements, vous spécifiez les fonctions devant traiter les événements et les erreurs. Chaque appel de la fonction qui traite des événements remet un événement unique à partir d’une partition spécifique. Il vous incombe de gérer cet événement. Si vous souhaitez vous assurer que le consommateur traite chaque message au moins une fois, vous devez écrire votre propre code avec une nouvelle tentative. Méfiez-vous cependant des messages empoisonnés.

Nous vous recommandons d’opérer assez rapidement. Autrement dit, effectuez le moins de traitement possible. Si vous avez besoin d’écrire dans le stockage et d’effectuer du routage, mieux vaut utiliser deux groupes de consommateurs et avoir deux processeurs d’événements.

## <a name="checkpointing"></a>Points de contrôle

Le *marquage de point de contrôle* est un processus par lequel un processeur d’événements marque ou valide la position du dernier événement correctement traité dans une partition. Le marquage d’un point de contrôle est généralement effectué à l’intérieur de la fonction qui traite les événements et se produit par partition au sein d’un groupe de consommateurs. 

Si un processeur d’événements se déconnecte d’une partition, une autre instance peut reprendre le traitement de la partition au point de contrôle précédemment validé par le dernier processeur de cette partition dans ce groupe de consommateurs. Lorsque le processeur se connecte, il transmet le décalage au hub d’événements pour spécifier l’emplacement où commencer la lecture. De cette façon, vous pouvez utiliser des points de contrôle pour marquer des événements comme « terminés » par des applications en aval et pour offrir une résilience en cas d’arrêt d’un processeur d’événements. Il est possible de revenir à des données plus anciennes en spécifiant un décalage inférieur à partir de ce processus de vérification. 

Lorsque le point de contrôle est appliqué pour marquer un événement comme traité, une entrée dans le magasin de points de contrôle est ajoutée ou mise à jour avec le décalage et le numéro de séquence de l’événement. Les utilisateurs doivent décider de la fréquence de mise à jour du point de contrôle. Une mise à jour après chaque événement traité avec succès peut avoir une incidence sur les performances et le coût, car elle déclenche une opération d’écriture dans le magasin de points de contrôle sous-jacent. De plus, l’application d’un point de contrôle à chaque événement indique un profil de courrier mis en file d’attente pour lequel une file d’attente Service Bus pourrait être une meilleure option qu’un hub d’événements. L’idée derrière Event Hubs est que vous obtenez une livraison « au moins une fois » à grande échelle. En octroyant la même puissance à vos systèmes en aval, il est facile de récupérer après des pannes ou des redémarrages qui se traduisent par la réception répétée des mêmes événements.

> [!NOTE]
> Si vous utilisez Stockage Blob Azure comme magasin de points de contrôle dans un environnement qui prend en charge une autre version du SDK de stockage Blob que celle généralement disponible sur Azure, vous devez utiliser le code pour remplacer la version de l’API de service de stockage par la version prise en charge par cet environnement. Par exemple, si vous exécutez [Event Hubs sur Azure Stack Hub version 2002](/azure-stack/user/event-hubs-overview), la version la plus élevée disponible pour le service Stockage est la version 2017-11-09. Dans ce cas, vous devez utiliser le code pour cibler la version de l’API de service de stockage 2017-11-09. Pour obtenir un exemple sur la façon de cibler une version spécifique de l’API de stockage, consultez les exemples suivants sur GitHub : 
> - [.NET](https://github.com/Azure/azure-sdk-for-net/tree/master/sdk/eventhub/Azure.Messaging.EventHubs.Processor/samples/). 
> - [Java](https://github.com/Azure/azure-sdk-for-java/blob/master/sdk/eventhubs/azure-messaging-eventhubs-checkpointstore-blob/src/samples/java/com/azure/messaging/eventhubs/checkpointstore/blob/)
> - [JavaScript](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/eventhub/eventhubs-checkpointstore-blob/samples/javascript) ou [TypeScript](https://github.com/Azure/azure-sdk-for-js/blob/master/sdk/eventhub/eventhubs-checkpointstore-blob/samples/typescript)
> - [Python](https://github.com/Azure/azure-sdk-for-python/blob/master/sdk/eventhub/azure-eventhub-checkpointstoreblob-aio/samples/)

## <a name="thread-safety-and-processor-instances"></a>Cohérence de thread et instances de processeur

Par défaut, la fonction qui traite les événements est appelée séquentiellement pour une partition donnée. Les événements et appels subséquents adressés à cette fonction à partir de la même partition s'accumulent en coulisses alors que la pompe d'événements continue à s'exécuter en arrière-plan sur d'autres threads. Les événements provenant de différentes partitions peuvent être traités simultanément et tout état partagé accessible sur plusieurs partitions doit être synchronisé.

## <a name="next-steps"></a>Étapes suivantes
Consultez les démarrages rapides suivants :

- [.NET Core](event-hubs-dotnet-standard-getstarted-send.md)
- [Java](event-hubs-java-get-started-send.md)
- [Python](event-hubs-python-get-started-send.md)
- [JavaScript](event-hubs-node-get-started-send.md)
