---
title: 'Démarrage rapide : Explorateur Azure Time Series Insights - Azure Time Series Insights | Microsoft Docs'
description: Découvrez comment commencer à utiliser l’Explorateur Azure Time Series Insights. Visualisez de grands volumes de données IoT et découvrez les principales fonctionnalités de votre environnement.
ms.service: time-series-insights
services: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.topic: quickstart
ms.workload: big-data
ms.custom: mvc seodec18
ms.date: 09/30/2020
ms.openlocfilehash: dd5ed52187a7cd23142b59e20c91c6d125946a72
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "91613779"
---
# <a name="quickstart-explore-azure-time-series-insights-gen1"></a>Démarrage rapide : Explorer Azure Time Series Insights Gen1

> [!CAUTION]
> Il s’agit d’un article Gen1.

Ce guide de démarrage rapide consacré à l’Explorateur Azure Time Series Insights vous aide à démarrer avec Azure Time Series Insights dans un environnement de démonstration gratuit. Ce guide de démarrage rapide explique comment utiliser votre navigateur web pour visualiser de grands volumes de données IoT avant de faire le tour de fonctionnalités clés désormais en disponibilité générale.

Azure Time Series Insights est un service entièrement managé d’analyse, de stockage et de visualisation qui simplifie la découverte et l’analyse simultanées de milliards d’événements IoT. Cette solution vous donne une vue globale de vos données afin que vous puissiez valider rapidement votre solution IoT et éviter des temps morts coûteux d’appareils stratégiques. Azure Time Series Insights vous aide à découvrir des tendances masquées, détecter les anomalies et effectuer des analyses de cause première quasiment en temps réel.

Pour plus de flexibilité, vous pouvez ajouter Azure Time Series Insights à des applications préexistantes via ses [API REST](./concepts-query-overview.md) puissantes et son [kit SDK client](https://github.com/microsoft/tsiclient). Vous pouvez utiliser les API pour stocker, interroger et consommer des données de série chronologique dans l’application cliente de votre choix. Vous pouvez également utiliser le kit SDK client pour ajouter des composants d’interface utilisateur à votre application existante.

Ce guide de démarrage rapide consacré à l’Explorateur Azure Time Series Insights propose une visite guidée des fonctionnalités.

> [!IMPORTANT]
> Créez un [compte Azure gratuit](https://azure.microsoft.com/free/?ref=microsoft.com&utm_source=microsoft.com&utm_medium=docs&utm_campaign=visualstudio) si ce n’est déjà fait.

## <a name="prepare-the-demo-environment"></a>Préparer l’environnement de démonstration

1. Dans votre navigateur, accédez à la [démonstration Gen1](https://insights.timeseries.azure.com/demo).

1. Si vous y êtes invité, connectez-vous à l’Explorateur Azure Time Series Insights à l’aide de vos informations d’identification de compte Azure.

1. La page de présentation Azure Time Series Insights s’affiche. Sélectionnez **Suivant** pour commencer la visite guidée.

   [![Bienvenue dans le tutoriel de démarrage rapide - sélectionnez Suivant](media/quickstart/quickstart-welcome.png)](media/quickstart/quickstart-welcome.png#lightbox)

## <a name="explore-the-demo-environment"></a>Explorer l’environnement de démonstration

1. Le **Volet de sélection de temps** s’affiche. Utilisez ce volet pour sélectionner un intervalle de temps à visualiser.

   [![Volet de sélection d’heure](media/quickstart/quickstart-time-selection-panel.png)](media/quickstart/quickstart-time-selection-panel.png#lightbox)

1. Sélectionnez un intervalle de temps et faites-le glisser dans la région. Ensuite, sélectionner **Rechercher**.

   [![Sélectionner un délai d’exécution](media/quickstart/quickstart-select-time.png)](media/quickstart/quickstart-select-time.png#lightbox)

   Azure Time Series Insights affiche une visualisation de graphique pour l’intervalle de temps spécifié. Vous pouvez effectuer diverses actions dans le graphique en courbes. Par exemple, vous pouvez filtrer, épingler, trier et empiler.

   Pour revenir au **volet de sélection de temps**, sélectionnez la flèche vers le bas, comme illustré :

   [![Graphique](media/quickstart/quickstart-select-down-arrow.png)](media/quickstart/quickstart-select-down-arrow.png#lightbox)

1. Sélectionnez **Ajouter** dans le **Volet de temps** pour ajouter un nouveau terme de recherche.

   [![Ajouter un volet de termes de recherche](media/quickstart/quickstart-add-terms.png)](media/quickstart/quickstart-add-terms.png#lightbox)

1. Dans le graphique, vous pouvez sélectionner une région en cliquant avec le bouton droit de la région et en sélectionnant **Explorer les événements**.

   [![Explorer les événements](media/quickstart/quickstart-explore-events.png)](media/quickstart/quickstart-explore-events.png#lightbox)

   Une grille de vos données brutes s’affiche à partir de la région que vous explorez.

   [![Explorer les événements - Vue des données de la grille](media/quickstart/quickstart-explore-events-grid-data.png)](media/quickstart/quickstart-explore-events-grid-data.png#lightbox)

## <a name="select-and-filter-data"></a>Sélectionner et filtrer les données

1. Modifiez vos termes pour changer les valeurs dans le graphique. Ajoutez un autre terme pour effectuer une corrélation croisée entre différents types de valeurs.

   [![Ajouter un terme](media/quickstart/quickstart-add-a-term.png)](media/quickstart/quickstart-add-a-term.png#lightbox)

1. Laissez la zone **Filter la série** vide pour afficher tous les termes de recherche sélectionnés ou entrez un terme de filtre dans la zone **Filtrer la série** pour le filtrage de série improvisé.

   [![Filtrer les séries](media/quickstart/quickstart-filter-series.png)](media/quickstart/quickstart-filter-series.png#lightbox)

   Pour le démarrage rapide, saisissez **Station5** pour avoir une corrélation croisée de la température et de la pression pour cette station.

Après avoir terminé le démarrage rapide, vous pouvez tester l’exemple de jeu de données pour créer différentes visualisations.

## <a name="clean-up-resources"></a>Nettoyer les ressources

Maintenant que vous avez terminé le didacticiel, nettoyez les ressources que vous avez créées :

1. Dans le menu de gauche du [portail Azure](https://portal.azure.com), sélectionnez **Toutes les ressources**, puis votre groupe de ressources Azure Time Series Insights.
1. Vous pouvez supprimer l’intégralité du groupe de ressources (et toutes les ressources qu’il contient) en sélectionnant **Supprimer** ou supprimer chaque ressource individuellement.

## <a name="next-steps"></a>Étapes suivantes

* Vous êtes prêt à créer votre propre environnement Azure Time Series Insights. Consultez [Planifier votre environnement Azure Time Series Insights](time-series-insights-environment-planning.md).
