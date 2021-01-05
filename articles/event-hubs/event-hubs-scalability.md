---
title: Scalabilité - Azure Event Hubs | Microsoft Docs
description: Cet article fournit des informations sur la mise à l'échelle d'Azure Event Hubs à l'aide de partitions et d'unités de débit.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: 4dacb24ace2332f590db54959cbf1f06694b982b
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86521953"
---
# <a name="scaling-with-event-hubs"></a>Mise à l’échelle avec Event Hubs

Deux facteurs influencent la mise à l’échelle avec Event Hubs.
*   Unités de débit
*   Partitions

## <a name="throughput-units"></a>Unités de débit

La capacité de débit des concentrateurs d’événements est contrôlée par les *unités de débit*. Les unités de débit sont des unités de capacité achetées préalablement. Un débit unique vous permet de disposer des capacités suivantes :

* Entrée : jusqu’à 1 Mo par seconde ou 1 000 événements par seconde, selon ce qui se produit en premier.
* Sortie : jusqu’à 2 Mo par seconde ou 4 096 événements par seconde.

En cas de dépassement de la capacité des unités de débit achetées, l’entrée est limitée et une exception [ServerBusyException](/dotnet/api/microsoft.azure.eventhubs.serverbusyexception) est renvoyée. La sortie ne produit aucune exception de limitation, mais reste limitée à la capacité des unités de débit achetées. Si vous recevez des exceptions de vitesse de publication ou si vous attendez une sortie plus élevée, vérifiez le nombre d’unités de débit achetées pour l’espace de noms. Vous pouvez gérer des unités de débit sur le panneau **Mettre à l’échelle** des espaces de noms, dans le [portail Azure](https://portal.azure.com). Vous pouvez également gérer les unités de débit par programmation à l’aide des [API Event Hubs](./event-hubs-samples.md).

Les unités de débit sont achetées à l’avance et facturées à l’heure. Une fois achetées, les unités de débit sont facturées au moins une heure. Vous pouvez acheter jusqu’à 20 unités de débit par espace de noms Azure Event Hubs. Ces unités sont partagées entre tous les hubs d’événements de l’espace de noms en question.

La fonctionnalité **Majoration automatique** d’Event Hubs augmente automatiquement la taille des instances en augmentant le nombre d’unités de débit pour répondre aux besoins d’utilisation. Cette opération permet d’éviter les situations de limitation, dans lesquelles :

- Les taux d’entrée de données sont supérieurs aux unités de débit définies.
- Les taux de demande de sortie de données sont supérieurs aux unités de débit définies.

Le service Event Hubs augmente le débit quand la charge dépasse le seuil minimal, sans qu’aucune demande échoue avec des erreurs ServerBusy. 

Pour plus d’informations sur la fonctionnalité d’augmentation automatique, consultez [Mettre à l’échelle automatiquement les unités de débit](event-hubs-auto-inflate.md).

## <a name="partitions"></a>Partitions
[!INCLUDE [event-hubs-partitions](../../includes/event-hubs-partitions.md)]

### <a name="partition-key"></a>Clé de partition

Vous pouvez utiliser une [clé de partition](event-hubs-programming-guide.md#partition-key) pour mapper des données d’événement entrant dans des partitions spécifiques dans le cadre de l’organisation des données. La clé de partition est une valeur fournie par l’expéditeur transmise dans un concentrateur d’événements. Elle est traitée par le biais d’une fonction de hachage statique, qui crée l’affectation de la partition. Si vous ne spécifiez aucune clé de partition lors de la publication d’un événement, une affectation de type tourniquet (round robin) est utilisée.

L’éditeur d’événements est uniquement informé de sa clé de partition, et non de la partition sur laquelle les événements sont publiés. Grâce à cette dissociation de la clé et de la partition, l’expéditeur n’a pas besoin de connaître trop d’informations sur le traitement en aval. Une identité par appareil ou unique à l'utilisateur constitue une bonne clé de partition, mais d'autres attributs tels que la géographie, peuvent également être utilisés pour regrouper des événements liés dans une seule partition.


## <a name="next-steps"></a>Étapes suivantes
Vous pouvez en apprendre plus sur Event Hubs en consultant les liens suivants :

- [Mettre à l'échelle automatiquement les unités de débit](event-hubs-auto-inflate.md)
- [Présentation du service Event Hubs](./event-hubs-about.md)
