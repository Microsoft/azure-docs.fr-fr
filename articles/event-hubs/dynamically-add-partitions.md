---
title: Ajouter dynamiquement des partitions à un Event Hub dans Azure Event Hubs
description: Cet article vous montre comment ajouter dynamiquement des partitions à un Event Hub dans Azure Event Hubs.
ms.topic: how-to
ms.date: 06/23/2020
ms.openlocfilehash: 4a729147eaa11497c66f82a9764dfee9492786b9
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87002537"
---
# <a name="dynamically-add-partitions-to-an-event-hub-apache-kafka-topic-in-azure-event-hubs"></a>Ajouter dynamiquement des partitions à un Event Hub (rubrique Apache Kafka) dans Azure Event Hubs
Azure Event Hubs diffuse des messages via un modèle de consommateur partitionné, dans lequel chaque consommateur lit uniquement un sous-ensemble spécifique, ou partition, du flux de messages. Ce modèle permet la mise à l’échelle horizontale pour le traitement des événements et fournit d’autres fonctionnalités de flux qui ne sont pas disponibles dans les rubriques et les files d’attente. Une partition est une séquence ordonnée d’événements qui est conservée dans un concentrateur d’événements. Les événements les plus récents sont ajoutés à la fin de cette séquence. Pour plus d’informations sur les partitions en général, consultez [Partitions](event-hubs-scalability.md#partitions).

Vous pouvez spécifier le nombre de partitions au moment de la création d’un Event Hub. Dans certains scénarios, vous aurez peut-être besoin d’ajouter des partitions après la création de l’Event Hub. Cet article explique comment ajouter dynamiquement des partitions à un Event Hub existant. 

> [!IMPORTANT]
> Les ajouts dynamiques de partitions sont disponibles uniquement sur des clusters Event Hubs **dédiés**.

> [!NOTE]
> Pour les clients Apache Kafka, un **Event Hub** est mappé à une **rubrique Kafka**. Pour plus d’informations sur les mappages entre Azure Event Hubs et Apache Kafka, consultez [le mappage conceptuel de Kafka et Event Hubs](event-hubs-for-kafka-ecosystem-overview.md#kafka-and-event-hub-conceptual-mapping).


## <a name="update-the-partition-count"></a>Mettre à jour le nombre de partitions
Cette section vous montre comment mettre à jour le nombre de partitions d’un Event Hub de différentes façons (PowerShell, CLI, etc.).

### <a name="powershell"></a>PowerShell
Utilisez la commande PowerShell [Set-AzureRmEventHub](/powershell/module/azurerm.eventhub/Set-AzureRmEventHub?view=azurermps-6.13.0) pour mettre à jour les partitions d’un Event Hub. 

```azurepowershell-interactive
Set-AzureRmEventHub -ResourceGroupName MyResourceGroupName -Namespace MyNamespaceName -Name MyEventHubName -partitionCount 12
```

### <a name="cli"></a>Interface de ligne de commande
Utilisez la commande CLI [`az eventhubs eventhub update`](/cli/azure/eventhubs/eventhub?view=azure-cli-latest#az-eventhubs-eventhub-update) pour mettre à jour les partitions d’un Event Hub. 

```azurecli-interactive
az eventhubs eventhub update --resource-group MyResourceGroupName --namespace-name MyNamespaceName --name MyEventHubName --partition-count 12
```

### <a name="resource-manager-template"></a>Modèle Resource Manager
Mettez à jour la valeur de la propriété `partitionCount` dans le modèle Resource Manager et redéployez ce dernier pour mettre à jour la ressource. 

```json
    {
        "apiVersion": "2017-04-01",
        "type": "Microsoft.EventHub/namespaces/eventhubs",
        "name": "[concat(parameters('namespaceName'), '/', parameters('eventHubName'))]",
        "location": "[parameters('location')]",
        "dependsOn": [
            "[resourceId('Microsoft.EventHub/namespaces', parameters('namespaceName'))]"
        ],
        "properties": {
            "messageRetentionInDays": 7,
            "partitionCount": 12
        }
    }
```

### <a name="apache-kafka"></a>Apache Kafka
Utilisez l’API `AlterTopics` (par exemple, via l’outil CLI**kafka-topics**) pour augmenter le nombre de partitions. Pour plus d’informations, consultez [la modification des rubriques Kafka](http://kafka.apache.org/documentation/#basic_ops_modify_topic). 

## <a name="event-hubs-clients"></a>Clients Event Hubs
Voyons comment des clients Event Hubs se comportent lorsque le nombre de partitions est mis à jour sur un Event Hub. 

Lorsque vous ajoutez une partition à un Event Hub existant, le client Event Hub reçoit un « `MessagingException` » du service informant les clients que les métadonnées d’entité (l’entité est votre Event Hub et les métadonnées sont les informations de partition) ont été modifiées. Les clients rouvrent automatiquement les liens AMQP, qui récupèrent alors les informations de métadonnées modifiées. Les clients fonctionnent ensuite normalement.

### <a name="senderproducer-clients"></a>Clients expéditeur/producteur
Event Hubs fournit trois options d’expéditeur :

- **Expéditeur de partition** : dans ce scénario, les clients envoient des événements directement à une partition. Bien que les partitions soient identifiables et que les événements puissent leur être envoyés directement, nous ne recommandons pas ce modèle. L’ajout de partitions n’a pas d’impact sur ce scénario. Nous vous recommandons de redémarrer les applications afin qu’elles puissent détecter les partitions nouvellement ajoutées. 
- **Expéditeur de clé de partition** : dans ce scénario, les clients envoient les événements avec une clé afin que tous les événements appartenant à cette clé finissent dans la même partition. Dans ce cas, le service hache la clé et les itinéraires vers la partition correspondante. La mise à jour du nombre de partitions peut entraîner des problèmes de désordre dus à un changement de hachage. Ainsi, si vous vous souciez de l’ordre, assurez-vous que votre application consomme tous les événements des partitions existantes avant d’augmenter le nombre de partitions.
- **Expéditeur par tourniquet (par défaut)**  : dans ce scénario, le service Event Hubs envoie par tourniquet (round robin) les événements sur les partitions. Le service Event Hubs prend en charge les modifications du nombre de partitions et envoie les événements aux nouvelles partitions dans les secondes qui suivent la modification du nombre de partitions.

### <a name="receiverconsumer-clients"></a>Clients récepteur/consommateur
Event Hubs fournit des récepteurs directs et une bibliothèque de consommateur simple appelée [hôte du processeur d’événements (ancien SDK)](event-hubs-event-processor-host.md) ou [processeur d’événements (nouveau SDK)](event-processor-balance-partition-load.md).

- **Récepteurs directs** : les destinataires directs écoutent des partitions spécifiques. Leur comportement d’exécution n’est pas perturbé lorsque des partitions effectuent un scale-out pour un Event Hub. L’application qui utilise des récepteurs directs doit prendre en charge la sélection des nouvelles partitions et l’attribution des récepteurs en conséquence.
- **Hôte du processeur d’événements** : ce client n’actualise pas automatiquement les métadonnées de l’entité. Ainsi, l’augmentation du nombre de partitions n’est pas prise en compte. La recréation d’une instance de processeur d’événements entraîne une extraction de métadonnées d’entité qui, à son tour, crée de nouveaux blobs pour les partitions nouvellement ajoutées. Les blobs préexistants ne seront pas concernés. Il est recommandé de redémarrer toutes les instances de processeur d’événements pour s’assurer que toutes les instances prennent en charge les partitions nouvellement ajoutées et que l’équilibrage de charge est traité correctement entre les consommateurs.

    Si vous utilisez l’ancienne version du Kit de développement logiciel (SDK) .NET ([WindowsAzure.ServiceBus](https://www.nuget.org/packages/WindowsAzure.ServiceBus/)), l’hôte du processeur d’événements supprime un point de contrôle existant lors du redémarrage si le nombre de partitions dans le point de contrôle ne correspond pas au nombre de partitions extraites du service. Ce comportement peut avoir un impact sur votre application. 

## <a name="apache-kafka-clients"></a>Clients Apache Kafka
Cette section décrit la façon dont les clients Apache Kafka qui utilisent le point de terminaison Kafka d’Azure Event Hubs se comportent lorsque le nombre de partitions est mis à jour pour un Event Hub. 

Les clients Kafka qui utilisent Event Hubs avec le protocole Apache Kafka se comportent différemment des clients Event Hub qui utilisent le protocole AMQP. Les clients Kafka mettent à jour leurs métadonnées une fois toutes les `metadata.max.age.ms` millisecondes. Vous spécifiez cette valeur dans les configurations des clients. Les bibliothèques `librdkafka` utilisent également la même configuration. Les mises à jour des métadonnées informent les clients des modifications du service, notamment l’augmentation du nombre de partitions. Pour connaître la liste des configurations, consultez les [configurations Apache Kafka pour Event Hubs](apache-kafka-configurations.md).

### <a name="senderproducer-clients"></a>Clients expéditeur/producteur
Les producteurs imposent toujours que les demandes d’envoi contiennent la destination de la partition pour chaque ensemble d’enregistrements générés. Ainsi, tout le partitionnement de la production est effectué côté client avec la vue du producteur sur les métadonnées du répartiteur. Une fois les nouvelles partitions ajoutées à la vue du producteur sur les métadonnées, elles sont disponibles pour les demandes des producteurs.

### <a name="consumerreceiver-clients"></a>Clients consommateur/récepteur
Lorsqu’un membre du groupe de consommateurs effectue une actualisation des métadonnées et récupère les partitions nouvellement créées, ce membre lance un rééquilibrage du groupe. Les métadonnées du consommateur sont ensuite actualisées pour tous les membres du groupe, et les nouvelles partitions sont attribuées par le responsable du rééquilibrage alloti.

## <a name="recommendations"></a>Recommandations

- Si vous utilisez une clé de partition avec vos applications de producteur et que vous dépendez du hachage de clé pour garantir le classement dans une partition, l’ajout dynamique de partitions n’est pas recommandé. 

    > [!IMPORTANT]
    > Tandis que les données existantes préservent le classement, le hachage de partition est interrompu pour les messages hachés après que le nombre de partitions a été modifié en raison de l’ajout de partitions.
- L’ajout d’une partition à une rubrique existante ou à une instance Event Hub est recommandé dans les cas suivants :
    - Quand vous utilisez la méthode du tourniquet (round robin) [par défaut] pour envoyer des événements
     - Stratégies de partitionnement par défaut Kafka, par exemple : stratégie Sticky Assignor


## <a name="next-steps"></a>Étapes suivantes
Pour plus d’informations sur les partitions, consultez [Partitions](event-hubs-scalability.md#partitions).

