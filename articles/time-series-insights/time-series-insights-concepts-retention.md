---
title: Comprendre la conservation des données dans votre environnement - Azure Time Series Insights | Microsoft Docs
description: Cet article décrit deux paramètres qui contrôlent la conservation des données dans votre environnement Azure Time Series Insights.
ms.service: time-series-insights
services: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.reviewer: jasonh, kfile
ms.workload: big-data
ms.topic: conceptual
ms.date: 09/29/2020
ms.custom: seodec18
ms.openlocfilehash: 4f236679d0662df852581a6a8408ed6bc0d4e3fe
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "91535687"
---
# <a name="understand-data-retention-in-azure-time-series-insights-gen1"></a>Comprendre la procédure de conservation des données dans Azure Time Series Insights Gen1

> [!CAUTION]
> Il s’agit d’un article Gen1.

Cet article décrit deux paramètres principaux qui affectent la conservation des données dans votre environnement Azure Time Series Insights.

## <a name="video"></a>Vidéo

### <a name="the-following-video-summarizes-azure-time-series-insights-data-retention-and-how-to-plan-for-itbr"></a>Dans cette vidéo, nous traitons de la conservation des données dans Azure Time Series Insights et de sa planification.</br>

> [!VIDEO https://www.youtube.com/embed/03x6zKDQ6DU]

Chacun de vos environnements Azure Time Series Insights inclut un paramètre qui contrôle la **durée de conservation des données**. La valeur est comprise entre 1 et 400 jours. Les données sont supprimées en fonction de la capacité de stockage ou de la durée de conservation de l’environnement, selon ce qui se présente en premier.

De plus, votre environnement Azure Time Series Insights inclut un paramètre **Comportement de limite de stockage dépassée**. Il contrôle le comportement relatif aux entrées et à la purge lorsque la capacité maximale d’un environnement est atteinte. Vous pouvez choisir entre deux comportements lors de la configuration :

- **Vidage des données anciennes** (par défaut)  
- **Suspendre l’entrée**

> [!NOTE]
> Par défaut, lorsque vous créez un nouvel environnement, la conservation est configurée sur **Vidage des données anciennes**. Ce paramètre peut être activé ou désactivé après la création à l’aide du portail Microsoft Azure, sur la page **Configurer** de l’environnement Azure Time Series Insights.
>
> - Pour plus d’informations sur la configuration des stratégies de rétention, consultez [Configuration de la conservation des données dans Azure Time Series Insights](time-series-insights-how-to-configure-retention.md).

Les deux stratégies de rétention des données sont décrites plus en détail ci-dessous.

## <a name="purge-old-data"></a>Vidage des données anciennes

- **Vidage des anciennes données** est le paramètre par défaut pour les environnements Azure Time Series Insights.  
- **Supprimer définitivement des données anciennes** est préférable lorsque les utilisateurs souhaitent avoir systématiquement leurs *données les plus récentes* dans leur environnement Azure Time Series Insights.
- Le paramètre **Vidage des données anciennes** *purge* les données une fois que les limites de l’environnement (durée de conservation, taille ou nombre, selon ce qui se présente en premier) sont atteintes. Par défaut, la conservation est définie sur 30 jours.
- Les données ingérées les plus anciennes sont vidées en premier (approche « premier arrivé, premier sorti »).

### <a name="example-one"></a>Premier exemple

Prenons l’exemple d’un environnement configuré avec le comportement de conservation **Poursuivre l’entrée et vider les données anciennes** :

La **durée de conservation des données** est définie sur 400 jours. La **Capacité** est définie sur l’unité S1, avec une capacité totale de 30 Go. Supposons que les données entrantes atteignent en moyenne 500 Mo par jour. Cet environnement ne peut conserver que 60 jours de données étant donné le taux de données entrantes, la capacité maximale étant atteinte après 60 jours. Les données entrantes s’accumulent comme suit : 500 Mo chaque jour x 60 jours = 30 Go.

Le 61ème jour, l’environnement affiche les données les plus récentes, mais vide les données les plus anciennes, datant de plus de 60 jours. Le vidage fait de la place pour les nouvelles données entrantes, afin qu’elles puissent toujours être explorées. Si l’utilisateur souhaite conserver les données plus longtemps, ils peut augmenter la taille de l’environnement en ajoutant des unités supplémentaires ou il peut envoyer (push) moins de données.  

### <a name="example-two"></a>Deuxième exemple

Prenons l’exemple d’un environnement configuré également avec le comportement de conservation **Poursuivre l’entrée et vider les données anciennes**. Dans cet exemple, la **Durée de conservation des données** est définie sur une valeur inférieure de 180 jours. La **Capacité** est définie sur l’unité S1, avec une capacité totale de 30 Go. Pour stocker des données durant les 180 jours complets, l’entrée quotidienne ne peut pas dépasser 0,166 Go (166 Mo) par jour.  

Quand le taux d’entrée quotidien de cet environnement dépasse 0,166 Go par jour, les données ne peuvent pas être stockées pendant 180 jours, étant donné que certaines données sont vidées. Considérez ce même environnement pendant un laps de temps occupé. Supposons que le taux d’entrée de l’environnement atteigne 0,189 Go en moyenne par jour. Durant ce laps de temps occupé, 158 jours environ de données sont conservés (30 Go/0,189 = 158,73 jours de conservation). Cette durée est inférieure à la durée de conservation des données souhaitée.

## <a name="pause-ingress"></a>Suspendre l’entrée

- Le paramètre **Suspendre l’entrée** est conçu pour s’assurer que les données ne sont pas vidées si les limites de taille et de nombre sont atteintes avant leur durée de conservation.  
- Le paramètre **Suspendre l’entrée** offre plus de temps aux utilisateurs pour augmenter la capacité de leur environnement avant que les données ne soient vidées en raison de la violation de la durée de conservation.
- Il offre une protection contre la perte de données, mais peut créer une opportunité en cas de perte de vos données les plus récentes si l’entrée est suspendue au-delà de la durée de conservation de votre source d’événements.
- Toutefois, quand la capacité maximale d’un environnement est atteinte, l’environnement suspend l’entrée des données jusqu’à ce que des actions supplémentaires se produisent :

  - Vous augmentez la capacité maximale de l’environnement pour ajouter des unités d’échelle comme décrit dans la section [Mise à l’échelle de votre environnement Azure Time Series Insights](time-series-insights-how-to-scale-your-environment.md).
  - La période de conservation des données est atteinte et les données sont vidées ; l’environnement est ainsi ramené sous sa capacité maximale.

### <a name="example-three"></a>Troisième exemple

Prenons l’exemple d’un environnement avec le comportement de conservation configuré sur **Suspendre l’entrée**. Dans cet exemple, la **Durée de conservation des données** est configurée sur 60 jours. La **capacité** est définie sur trois (3) unités S1. Supposons que cet environnement connaît une entrée de données de 2 Go par jour. Dans cet environnement, l’entrée est suspendue une fois la capacité maximale atteinte.

À ce stade, l’environnement affiche le même jeu de données jusqu’à ce que l’entrée reprenne ou que l’option **Poursuivre l’entrée** soit activée (vidant les données les plus anciennes pour faire de la place aux nouvelles données).

Lorsque l’entrée reprend :

- Les données circulent dans l’ordre où elles ont été reçues par la source d’événements
- Les événements sont indexés en fonction de leur horodatage, sauf si vous avez dépassé les stratégies de conservation sur votre source d’événements. Pour plus d’informations sur la configuration de la conservation de la source d’événements, consultez [Forum Aux Questions (FAQ) sur Event Hubs](../event-hubs/event-hubs-faq.md)

> [!IMPORTANT]
> Nous vous conseillons de définir des alertes pour éviter que l’entrée ne soit suspendue. La perte de données est possible, car la conservation par défaut est définie sur 1 jour pour les sources d’événements Azure. Par conséquent, une fois que l’entrée est suspendue, vous perdrez probablement les données les plus récentes, sauf si une action supplémentaire est effectuée. Vous devez augmenter la capacité ou définir le comportement sur **Vider les données anciennes** afin d’éviter la perte de données potentielle.

Dans les Event Hubs concernés, envisagez d’ajuster la propriété **Conservation des messages** afin de minimiser la perte de données lorsque l’entrée est suspendue dans Azure Time Series Insights.

[![Conservation des messages Event Hub.](media/time-series-insights-concepts-retention/event-hub-retention.png)](media/time-series-insights-concepts-retention/event-hub-retention.png#lightbox)

Si aucune propriété n’est configurée sur la source d’événements (`timeStampPropertyName`), Azure Time Series Insights utilise par défaut l’horodatage d’arrivée dans Event Hub comme axe des abscisses. Si le paramètre `timeStampPropertyName` est configuré sur une autre valeur, l’environnement recherche la propriété `timeStampPropertyName` configurée dans le paquet de données lorsque les événements sont analysés.

Consultez [Mise à l’échelle de votre environnement Azure Time Series Insights](time-series-insights-how-to-scale-your-environment.md) si vous devez mettre votre environnement à l’échelle pour prendre en charge une capacité supplémentaire ou pour augmenter la durée de conservation.

## <a name="next-steps"></a>Étapes suivantes

- Pour en savoir plus sur la commutation ou la modification des paramètres de conservation de données, voir [Configuration de la conservation dans Azure Time Series Insights](time-series-insights-how-to-configure-retention.md).

- En savoir plus sur la [Réduction de la latence dans Azure Time Series Insights](time-series-insights-environment-mitigate-latency.md).
