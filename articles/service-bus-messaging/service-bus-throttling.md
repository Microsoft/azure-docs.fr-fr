---
title: Vue d’ensemble de la limitation Azure Service Bus | Microsoft Docs
description: Vue d’ensemble de la limitation Service Bus – Niveaux Standard et Premium.
ms.topic: article
ms.date: 06/23/2020
ms.openlocfilehash: 436f9a40269f7eea4e31b55b9657d38849876eb4
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85340942"
---
# <a name="throttling-operations-on-azure-service-bus"></a>Opérations de limitation dans Azure Service Bus

Les solutions cloud natives laissent à penser qu’elles procurent des ressources illimitées adaptables à votre charge de travail. Même si cette idée vaut plus pour le cloud que pour les systèmes locaux, le cloud présente quand même des limites.

Ces limites peuvent occasionner une limitation des demandes pour les applications clientes aussi bien au niveau Standard qu’au niveau Premium, comme nous le verrons dans cet article. 

## <a name="throttling-in-azure-service-bus-standard-tier"></a>Limitation dans le niveau Standard d’Azure Service Bus

Le niveau Standard d’Azure Service Bus fonctionne comme une installation multilocataire avec un modèle tarifaire basé sur le paiement à l’utilisation. Ici, les différents espaces de noms présents dans un même cluster partagent les ressources allouées. Le niveau Standard est recommandé pour les environnements de développement, de test et d’assurance qualité équipés de systèmes de production à faible rendement.

Par le passé, Azure Service Bus imposait des limitations grossières qui dépendaient strictement de l’utilisation des ressources. Ceci dit, il est possible d’affiner la logique de limitation et de définir un comportement de limitation prévisible pour tous les espaces de noms qui partagent ces ressources.

Pour assurer une utilisation et une répartition équitables des ressources entre les différents espaces de noms Azure Service Bus standard qui utilisent les mêmes ressources, la logique de limitation a été modifiée pour être basée sur les crédits.

> [!NOTE]
> Il est important de noter que la limitation n’est ***pas une nouveauté***, que ce soit pour Azure Service Bus ou n’importe quel service cloud natif.
>
> La limitation basée sur les crédits consiste simplement à affiner la façon dont différents espaces de noms partagent des ressources dans un environnement multilocataire de niveau Standard, ce qui permet une utilisation équitable des ressources partagées par tous les espaces de noms.

### <a name="what-is-credit-based-throttling"></a>Qu’est-ce que la limitation basée sur les crédits ?

La limitation basée sur les crédits limite le nombre d’opérations qui peuvent être effectuées dans un espace de noms donné au cours d’une période déterminée. 

Voici le workflow de la limitation basée sur les crédits – 

  * Au début de chaque période, nous attribuons un certain nombre de crédits à chaque espace de noms.
  * Les opérations exécutées par les applications clientes expéditrices ou réceptrices sont comptabilisées au titre de ces crédits (et soustraites des crédits disponibles).
  * Si les crédits sont épuisés, les opérations suivantes sont limitées jusqu’au début de la période suivante.
  * Les crédits sont réapprovisionnés au début de la période suivante.

### <a name="what-are-the-credit-limits"></a>Quelles sont les limites de crédit ?

Les limites de crédit actuellement définies sont de « 1 000 » crédits par seconde (par espace de noms).

Toutes les opérations ne sont pas égales. Voici le coût en crédits de chacune des opérations – 

| Opération | Coût en crédits|
|-----------|-----------|
| Opérations sur les données (Send, SendAsync, Receive, ReceiveAsync, Peek) |1 crédit par message |
| Opérations de gestion (Créer, Lire, Mettre à jour, Supprimer sur files d’attente, Rubriques, Abonnements, Filtres) | 10 crédits |

> [!NOTE]
> Notez que, lors de l’envoi à une rubrique, chaque message est évalué par rapport à un ou plusieurs filtres avant d’être mis à disposition sur l’abonnement.
> Chaque évaluation de filtre est également décomptée de la limite de crédit (c’est-à-dire 1 crédit par évaluation de filtre).
>

### <a name="how-will-i-know-that-im-being-throttled"></a>Comment savoir si je suis limité ?

Quand les demandes sont limitées pour une application cliente, celle-ci reçoit la réponse ci-dessous du serveur et la journalise.

```
The request was terminated because the entity is being throttled. Error code: 50009. Please wait 2 seconds and try again.
```

### <a name="how-can-i-avoid-being-throttled"></a>Comment éviter d’être limité ?

Dans le cas des ressources partagées, il est important d’instaurer une utilisation équitable entre les différents espaces de noms Service Bus qui partagent ces ressources. La limitation est l’assurance qu’en cas de pic dans une charge de travail, les autres charges de travail utilisant les mêmes ressources ne seront pas limitées.

Comme nous le verrons plus loin dans cet article, le fait que les kits SDK clients (et d’autres offres PaaS Azure) intègrent la stratégie de nouvelle tentative par défaut écarte tout risque de limitation. Les demandes limitées feront l’objet de nouvelles tentatives avec un backoff exponentiel et finiront par passer une fois les crédits réapprovisionnés.

Bien entendu, certaines applications peuvent être sensibles à la limitation. Dans ce cas, il est recommandé de [migrer votre espace de noms Service Bus actuel de Standard à Premium ](service-bus-migrate-standard-premium.md). 

Pour la migration, vous pouvez allouer des ressources dédiées à votre espace de noms Service Bus et effectuer un scale-up sur les ressources de telle sorte que si votre charge de travail connaît un pic, vous aurez moins de risques d’être limité. Par ailleurs, une fois que votre charge de travail est redescendue à des niveaux normaux, vous pouvez effectuer un scale-down sur les ressources allouées à votre espace de noms.

## <a name="throttling-in-azure-service-bus-premium-tier"></a>Limitation dans le niveau Premium d’Azure Service Bus

Le niveau [Premium d’Azure Service Bus](service-bus-premium-messaging.md) alloue des ressources dédiées, sous la forme d’unités de messagerie, à chaque espace de noms configuré par le client. Ces ressources dédiées offrent un débit et une latence prévisibles et sont recommandées pour les systèmes de production sensibles ou à haut rendement.

De plus, le niveau Premium permet aussi aux clients d’effectuer le scale-up de leur capacité de rendement quand la charge de travail connaît un pic.

### <a name="how-does-throttling-work-in-service-bus-premium"></a>Comment fonctionne la limitation dans Service Bus Premium ?

Du fait de l’allocation exclusive de ressources dans le cas de Service Bus Premium, la limitation est intimement liée aux limites des ressources allouées à l’espace de noms.

Si le nombre de demandes est supérieur à la capacité de traitement actuelle des ressources, les demandes sont limitées.

### <a name="how-will-i-know-that-im-being-throttled"></a>Comment savoir si je suis limité ?

Il existe différentes façons d’identifier la limitation dans Azure Service Bus Premium – 
  * Les **demandes limitées**  figurent dans les [métriques des demandes d’Azure Monitor](service-bus-metrics-azure-monitor.md#request-metrics) pour identifier le nombre de demandes qui ont été limitées.
  * Une **utilisation du processeur** élevée indique que l’allocation de ressources actuelle est élevée et que les demandes risquent d’être limitées si la charge de travail actuelle ne diminue pas.
  * Une **utilisation de la mémoire** élevée indique que l’allocation de ressources actuelle est élevée et que les demandes risquent d’être limitées si la charge de travail actuelle ne diminue pas.

### <a name="how-can-i-avoid-being-throttled"></a>Comment éviter d’être limité ?

Étant donné que l’espace de noms Service Bus Premium a déjà des ressources dédiées, vous pouvez réduire la possibilité d’être limité en effectuant un scale-up sur le nombre d’unités de messagerie allouées à votre espace de noms dans l’hypothèse (ou en anticipation) d’un pic dans la charge de travail.

Un scale-up/down peut être effectué en créant des [runbooks](../automation/automation-create-alert-triggered-runbook.md) qui peuvent être déclenchés par des changements dans les métriques ci-dessus.

## <a name="faqs"></a>FAQ

### <a name="how-does-throttling-affect-my-application"></a>En quoi la limitation affecte-t-elle mon application ?

Quand une demande est limitée, cela sous-entend que le service est occupé ; il fait face à plus de demandes que les ressources ne l’autorisent. Si la même opération est retentée après quelques instants, la demande est honorée une fois que le service est sorti de sa charge de travail actuelle.

La limitation étant le comportement attendu de tous les services cloud natifs, nous avons mis en place la logique de nouvelle tentative dans le SDK Azure Service Bus lui-même. Par défaut, les nouvelles tentatives automatiques sont définies avec un backoff exponentiel pour éviter qu’une même demande soit limitée à chaque fois.

La logique de nouvelle tentative par défaut s’applique à chaque opération.

### <a name="does-throttling-result-in-data-loss"></a>La limitation entraîne-t-elle une perte de données ?

Azure Service Bus est optimisé pour la persistance. Nous veillons en effet à ce que toutes les données envoyées à Service Bus soient validées dans le stockage avant que le service ne reconnaisse l’issue favorable de la demande.

Une fois que Service Bus a accusé réception de la demande (statut « ACK »), cela signifie que Service Bus a correctement traité la demande. Si Service Bus retourne « NACK » (échec), cela signifie que Service Bus n’a pas pu traiter la demande et que l’application cliente doit tenter à nouveau la demande.

Cependant, quand une demande est limitée, cela sous-entend que le service ne peut pas accepter et traiter la demande dans l’immédiat en raison des limites de ressources. Cela **n’est pas** le signe d’une perte de données ; cela signifie simplement que Service Bus n’a pas examiné la demande. Dans ce cas, la stratégie de nouvelle tentative par défaut du SDK Service Bus est l’assurance que la demande sera finalement traitée.

## <a name="next-steps"></a>Étapes suivantes

Pour plus d’informations et des exemples d’utilisation de la messagerie Service Bus, consultez les rubriques avancées suivantes :

* [Présentation de la messagerie Service Bus](service-bus-messaging-overview.md)
* [Démarrage rapide : Envoyer et recevoir des messages avec le Portail Azure et .NET](service-bus-quickstart-portal.md)
* [Tutoriel : Mettre à jour l’inventaire avec le Portail Azure et des rubriques/abonnements](service-bus-tutorial-topics-subscriptions-portal.md)

