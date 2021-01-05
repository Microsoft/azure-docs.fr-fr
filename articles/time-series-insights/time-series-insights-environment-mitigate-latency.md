---
title: 'Guide pratique pour surveiller et réduire la limitation : Azure Time Series Insights | Microsoft Docs'
description: Découvrez comment analyser, diagnostiquer et réduire les problèmes de performances qui provoquent la latence et la limitation dans Azure Time Series Insights.
ms.service: time-series-insights
services: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.reviewer: v-mamcge, jasonh, kfile
ms.devlang: csharp
ms.workload: big-data
ms.topic: troubleshooting
ms.date: 09/29/2020
ms.custom: seodec18
ms.openlocfilehash: e89189b22b144d9e92ee8315bc6fd9aabe699eec
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91531647"
---
# <a name="monitor-and-mitigate-throttling-to-reduce-latency-in-azure-time-series-insights-gen1"></a>Surveiller et réduire la limitation afin d'éviter la latence dans Azure Time Series Insights Gen1

> [!CAUTION]
> Il s’agit d’un article Gen1.

Lorsque la quantité de données entrantes dépasse la configuration de votre environnement, vous pouvez rencontrer une latence ou une limitation dans Azure Time Series Insights.

Vous pouvez éviter la latence et la limitation en configurant correctement votre environnement pour la quantité de données que vous souhaitez analyser.

Vous êtes susceptible de rencontrer une latence et une limitation lorsque vous :

- Ajoutez une source d’événement qui contient des données anciennes qui peuvent dépasser le taux d’entrée alloué (qu’Azure Time Series Insights devra rattraper).
- Ajoutez plusieurs sources d’événements dans un environnement, ce qui entraîne un pic d’activité dû aux événements supplémentaires (cela risque de dépasser la capacité de votre environnement).
- Envoyez de grandes quantités d’événements historiques à une source d’événements, ce qui entraîne un décalage (qu’Azure Time Series Insights devra rattraper).
- Joignez des données de référence avec la télémétrie, ce qui entraîne une plus grande taille d’événement. La taille maximale autorisée du paquet est 32 Ko ; les paquets de données d’une taille supérieure à 32 Ko sont tronqués.

## <a name="video"></a>Vidéo

### <a name="learn-about-azure-time-series-insights-data-ingress-behavior-and-how-to-plan-for-itbr"></a>Apprenez-en plus sur le comportement d’entrée de données Azure Time Series Insights et comment le planifier.</br>

> [!VIDEO https://www.youtube.com/embed/npeZLAd9lxo]

## <a name="monitor-latency-and-throttling-with-alerts"></a>Analyser la latence et la limitation à l’aide d’alertes

Les alertes peuvent vous aider à diagnostiquer et réduire les problèmes de latence au sein de votre environnement.

1. Dans le portail Azure, sélectionnez votre environnement Azure Time Series Insights. Ensuite, sélectionnez **Alertes**.

   [![Ajouter une alerte à votre environnement Azure Time Series Insights](media/environment-mitigate-latency/mitigate-latency-add-alert.png)](media/environment-mitigate-latency/mitigate-latency-add-alert.png#lightbox)

1. Sélectionnez **+ Nouvelle règle d’alerte**. Le panneau **Créer une règle** s’affiche. Sélectionnez **Ajouter** sous **CONDITION**.

   [![Ajouter un volet d’alerte](media/environment-mitigate-latency/mitigate-latency-add-pane.png)](media/environment-mitigate-latency/mitigate-latency-add-pane.png#lightbox)

1. Ensuite, configurez les conditions exactes de la logique du signal.

   [![Configurer la logique du signal](media/environment-mitigate-latency/configure-alert-rule.png)](media/environment-mitigate-latency/configure-alert-rule.png#lightbox)

   À partir de là, vous pouvez configurer des alertes à l’aide des conditions suivantes :

   |Métrique  |Description  |
   |---------|---------|
   |**Octets reçus en entrée**     | Nombre d’octets bruts lus à partir des sources d’événements. Le nombre brut inclut généralement le nom de la propriété et la valeur.  |  
   |**Messages non valides reçus en entrée**     | Nombre de messages non valides lus à partir de la totalité des Event Hubs Azure ou des sources d’événements Azure IoT Hub.      |
   |**Messages reçus en entrée**   | Nombre de messages lus à partir de la totalité des Event Hubs ou des sources d’événements IoT Hub.        |
   |**Octets stockés en entrée**     | Taille totale des événements stockés et disponibles pour la requête. La taille est calculée uniquement sur la valeur de propriété.        |
   |**Événements stockés en entrée**     |   Nombre d’événements aplatis stockés et disponibles pour la requête.      |
   |**Retard des messages reçus en entrée**    |  Différence en secondes entre l’heure à laquelle le message est placé en file d’attente dans la source d’événement et l’heure à laquelle il est traité en entrée.      |
   |**Décalage du nombre des messages reçus en entrée**    |  Différence entre le numéro de séquence du dernier message placé en file d’attente dans la partition source de l’événement et le numéro de séquence du message traité en entrée.      |

   Sélectionnez **Terminé**.

1. Après avoir configuré la logique du signal souhaitée, examinez la règle d’alerte choisie visuellement.

   [![Vue de la latence et graphiques](media/environment-mitigate-latency/mitigate-latency-view-and-charting.png)](media/environment-mitigate-latency/mitigate-latency-view-and-charting.png#lightbox)

## <a name="throttling-and-ingress-management"></a>Limitation et gestion des entrées

- Si vous êtes limité, une valeur s’affichera pour la mesure *Retard des messages reçus en entrée*, vous informant du décalage en secondes entre votre environnement Azure Time Series Insights et l’horodatage réel de l’arrivée du message dans la source d’événement (sans tenir compte du délai d’indexation d’environ 30 à 60 secondes).  

  Le champ *Décalage de nombre des messages reçus en entrée* doit également comporter une valeur, vous octroyant une visibilité sur le nombre de messages de retard que vous accusez.  Le moyen le plus simple de rattraper le retard est d’augmenter la capacité de votre environnement à une taille qui vous permettra de combler la différence.  

  Par exemple, si votre environnement S1 montre un décalage de 5 000 000 de messages, vous pouvez augmenter la taille de votre environnement à 6 unités pendant environ une journée pour refaire votre retard.  Vous pouvez même augmenter davantage pour rattraper plus vite le retard. La période de rattrapage se produit souvent lors du provisionnement initial d’un environnement, plus particulièrement lorsque vous le connectez à une source d’événement qui présente déjà des événements ou quand vous chargez de manière groupée de gros volumes de données d’historique.

- Une autre technique est de définir une alerte d’**événements stockés en entrée** >= à un seuil légèrement inférieur à la capacité totale de votre environnement pour une période de 2 heures.  Cette alerte peut vous aider à comprendre si vous êtes en permanence à votre capacité maximale, ce qui implique une forte probabilité de latence.

  Par exemple, si vous avez trois unités S1 configurées (ou 2 100 événements de capacité d’entrée par minute), vous pouvez définir une alerte d’**événements stockés en entrée** pour >= 1 900 événements pendant 2 heures. Si vous dépassez en permanence ce seuil et par conséquent, vous déclenchez l’alerte, vous êtes probablement sous-configuré.  

- Si vous pensez que vous êtes limité, vous pouvez comparer vos **messages reçus en entrée** avec les messages de sortie de votre source d’événement.  Si l’entrée dans votre Event Hub est supérieure à vos **messages reçus en entrée**, vos informations Azure Time Series Insights sont probablement limitées.

## <a name="improving-performance"></a>Améliorer les performances

La meilleure façon de réduire la limitation ou la latence est de d’augmenter la capacité de votre environnement.

Vous pouvez éviter la latence et la limitation en configurant correctement votre environnement pour la quantité de données que vous souhaitez analyser. Pour plus d’informations l’augmentation de la capacité de votre environnement, consultez [Mettre à l’échelle votre environnement](time-series-insights-how-to-scale-your-environment.md).

## <a name="next-steps"></a>Étapes suivantes

- Consultez [Diagnostiquer et résoudre les problèmes dans votre environnement Azure Time Series Insights](time-series-insights-diagnose-and-solve-problems.md).

- Apprenez [comment mettre à l’échelle votre environnement Azure Time Series Insights](time-series-insights-how-to-scale-your-environment.md).
