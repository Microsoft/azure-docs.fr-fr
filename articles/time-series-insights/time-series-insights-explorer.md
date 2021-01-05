---
title: Explorer des données avec l’Explorateur – Azure Time Series Insights | Microsoft Docs
description: Découvrez comment utiliser l’Explorateur Azure Time Series Insights pour afficher vos données IoT.
ms.service: time-series-insights
services: time-series-insights
author: deepakpalled
ms.author: dpalled
manager: diviso
ms.reviewer: v-mamcge, jasonh, kfile
ms.devlang: csharp
ms.workload: big-data
ms.topic: conceptual
ms.date: 09/29/2020
ms.custom: seodec18
ms.openlocfilehash: 639f2ef12d190a56e04a9b48d96ea0a6537d243c
ms.sourcegitcommit: 10d00006fec1f4b69289ce18fdd0452c3458eca5
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/21/2020
ms.locfileid: "95020110"
---
# <a name="azure-time-series-insights-gen1-explorer"></a>Explorateur Azure Time Series Insights Gen1

> [!CAUTION]
> Il s’agit d’un article Gen1.

Cet article décrit les fonctionnalités et options pour l’[application web Explorateur](https://insights.timeseries.azure.com/) Azure Time Series Insights Gen1. L’Explorateur Azure Time Series Insights fait preuve de puissantes capacités de visualisation de données fournies par le service et est accessible au sein de votre propre environnement.

Azure Time Series Insights est un service entièrement managé d’analyse, de stockage et de visualisation qui simplifie la découverte et l’analyse simultanées de milliards d’événements IoT. Cette solution vous donne une vue globale de vos données, ce qui vous permet de valider rapidement votre solution IoT et d’éviter des temps morts coûteux d’appareils stratégiques. Vous pouvez découvrir des tendances masquées, détecter les anomalies et effectuer des analyses de cause première quasiment en temps réel.

> [!TIP]
> Pour une visite guidée de l’environnement de démonstration, consultez le [Guide de démarrage rapide Azure Time Series Insights](time-series-quickstart.md).

## <a name="video"></a>Vidéo

### <a name="learn-about-querying-data-by-using-the-azure-time-series-insights-explorer-br"></a>Apprenez-en davantage sur l’interrogation de données avec l’Explorateur Azure Time Series Insights. </br>

> [!VIDEO https://www.youtube.com/embed/SHFPZvrR71s]

>[!NOTE]
>Regardez la vidéo précédente <a href="https://www.youtube.com/watch?v=6ehNf6AJkFo">« Getting started with Azure Time Series Insights by using an Azure IoT Solution Accelerator »</a> (Bien démarrer avec Azure Time Series Insights en utilisant un accélérateur de solution Azure IoT).

## <a name="prerequisites"></a>Prérequis

Avant de pouvoir utiliser l’Explorateur Azure Time Series Insights, vous devez :

- Créez un environnement Azure Time Series Insights. Pour plus d’informations, consultez [Bien démarrer avec Azure Time Series Insights](./time-series-insights-get-started.md).
- [Fournissez l’accès](./concepts-access-policies.md) à votre compte dans l’environnement.
- Ajoutez-y une source [hub IoT](./how-to-ingest-data-iot-hub.md) ou un [hub d’événement](./how-to-ingest-data-event-hub.md).

## <a name="explore-and-query-data"></a>Explorer et interroger les données

Quelques minutes après la connexion de votre source d’événements à votre environnement Azure Time Series Insights, vous pouvez explorer et interroger les données de votre série chronologique.

1. Pour démarrer, ouvrez l’[Explorateur Azure Time Series Insights](https://insights.timeseries.azure.com/) dans votre navigateur web. Sur le côté gauche de la fenêtre, sélectionnez un environnement. Tous les environnements auxquels vous avez accès sont répertoriés par ordre alphabétique.

1. Après avoir sélectionné un environnement, utilisez les configurations **From** (À partir de) et **To** (Vers) en haut, ou sélectionnez et faites glisser l’intervalle de temps souhaité. Sélectionnez la loupe en haut à droite, ou cliquez avec le bouton droit sur l’intervalle de temps sélectionné et sélectionnez **Rechercher**.

1. Vous pouvez également actualiser la disponibilité automatiquement toutes les minutes, en sélectionnant le bouton **Activer automatiquement**. Le bouton **Activer automatiquement** s’applique uniquement au graphique de disponibilité, pas au contenu de la visualisation principale.

1. L’icône Azure Cloud vous permet d’accéder à votre environnement dans le portail Azure.

   [![Sélection de l’environnement Azure Time Series Insights](media/time-series-insights-explorer/tsi-ga-explorer-environments.png)](media/time-series-insights-explorer/tsi-ga-explorer-environments.png#lightbox)

1. Ensuite, un graphique s’affiche, montrant le nombre total d’événements pendant la période sélectionnée. Vous avez plusieurs commandes disponibles :

    - **Panneau de l’éditeur de termes** : L’espace de terme correspond à l’endroit où vous interrogez votre environnement. Vous le trouverez sur le côté gauche de l’écran :
      - **MESURE** : Cette liste déroulante affiche toutes les colonnes numériques (**doubles**).
      - **FRACTIONNER PAR** : Cette liste déroulante affiche les colonnes catégorielles (**chaînes**).
      - Vous pouvez activer une interpolation par étape, afficher les valeurs minimale et maximale et ajuster l’axe des ordonnées à partir du panneau de configuration à côté de **MESURE**. Vous pouvez aussi choisir si les données indiquées sont un nombre, une moyenne ou une somme des données.
      - Vous pouvez ajouter jusqu’à cinq conditions à afficher sur le même axe X. Sélectionnez **Ajouter** pour ajouter un nouveau terme ou utilisez le bouton **Cloner ce terme** pour ajouter la copie d’un terme existant.

        [![Sélection des termes, filtrage et panneau de requête](media/time-series-insights-explorer/tsi-ga-explorer-add-or-clone.png)](media/time-series-insights-explorer/tsi-ga-explorer-add-or-clone.png#lightbox)

      - **Prédicat** : Utilisez le prédicat pour filtrer rapidement les événements à l’aide de l’ensemble d’opérandes répertoriés dans le tableau ci-dessous. Si vous effectuez une recherche en la sélectionnant ou en cliquant dessus, le prédicat est automatiquement mis à jour selon cette recherche. Les types d’opérandes pris en charge comprennent les suivants :

         |Opération  |Types pris en charge  |Notes  |
         |---------|---------|---------|
         |**<**, **>**, **<=**, **>=**    |  **Double**, **DateTime**, **TimeSpan**       |         |
         |**=** , **!=** , **<>**     | **String**, **Bool**, **Double**, **DateTime**, **TimeSpan**, **NULL**        |         |
         |**IN**     | **String**, **Bool**, **Double**, **DateTime**, **TimeSpan**, **NULL**        |  Tous les opérandes doivent être du même type ou être la constante **NULL**.        |
         |**HAS**     | **Chaîne**        |  Seuls les littéraux de chaîne constante sont autorisés à droite. Les chaînes vides et **NULL** ne sont pas autorisées.       |

      - **Exemples de requêtes**

         [![Exemples de requêtes Gen1](media/time-series-insights-explorer/tsi-ga-example-queries.png)](media/time-series-insights-explorer/tsi-ga-example-queries.png#lightbox)

1. Vous pouvez utiliser l’outil curseur **Taille de l’intervalle** pour effectuer un zoom/zoom arrière sur les intervalles pour le même intervalle. Ce curseur fournit un contrôle plus précis du déplacement entre des grandes tranches de temps qui montrent des tendances lissées pour des tranches se réduisant jusqu’à la milliseconde, ce qui vous permet de visualiser et d’analyser des segments précis de vos donnés avec une haute résolution. Le point de départ par défaut du curseur est défini comme la vue optimale des données à partir de votre sélection afin d’équilibrer la résolution, la vitesse de la requête et la granularité.

1. L’outil **Balayage temporel** simplifie la navigation entre intervalles.

1. Sélectionnez l’icône **Enregistrer** pour enregistrer votre requête actuelle et la partager avec d’autres utilisateurs de l’environnement. Quand vous sélectionnez l’icône **Ouvrir**, vous pouvez passer en revue toutes vos requêtes enregistrées et toutes les requêtes partagées des autres utilisateurs dans les environnements auxquels vous avez accès.

   [![Requêtes](media/time-series-insights-explorer/tsi-ga-explorer-saved-queries.png)](media/time-series-insights-explorer/tsi-ga-explorer-saved-queries.png#lightbox)

## <a name="visualize-data"></a>Visualiser les données

1. Utilisez l’outil **Vue en perspective** pour obtenir une vue simultanée de jusqu’à quatre requêtes uniques. Le bouton **Vue en perspective** se situe dans le coin supérieur droit du graphique.

   [![Sélectionner les requêtes à ajouter au volet de perspective](media/time-series-insights-explorer/tsi-ga-explorer-perspective-panes.png)](media/time-series-insights-explorer/tsi-ga-explorer-perspective-panes.png#lightbox)

1. Affichez un graphique pour explorer visuellement vos données, et utilisez les outils du **Graphique** :

    - **Sélectionnez** ou **cliquez sur** un intervalle de temps spécifique ou une série de données unique.
    - Dans une sélection d’intervalle, vous pouvez effectuer un zoom ou explorer les événements.
    - Au sein d’une série de données, vous pouvez fractionner la série par une autre colonne, ajouter la série en tant que nouveau terme, afficher uniquement la série sélectionnée, exclure les séries sélectionnées, effectuer un test ping sur cette série ou explorer les événements de la série sélectionnée.
    - Dans la zone de filtre à gauche du graphique, vous pouvez passer en revue toutes les séries de données affichées, et les réorganiser par valeur ou par nom. Vous pouvez aussi afficher toutes les séries de données ou des séries épinglées ou non épinglées. Vous pouvez sélectionner une seule série de données et fractionner la série par une autre colonne, ajouter la série en tant que nouveau terme, afficher uniquement la série sélectionnée, exclure les séries sélectionnées, effectuer un test ping sur cette série ou explorer les événements de la série sélectionnée.
    - Quand vous affichez plusieurs termes simultanément, vous pouvez empiler, désempiler et passer en revue des données supplémentaires sur une série de données, et utiliser le même axe des ordonnées sur tous les termes. Utilisez les boutons dans le coin supérieur droit du graphique.

    [![Paramètres des options de l’outil graphique dans le coin supérieur droit ](media/time-series-insights-explorer/tsi-ga-example-chart-options.png)](media/time-series-insights-explorer/tsi-ga-example-chart-options.png#lightbox)

1. Utilisez la **carte thermique** pour identifier rapidement les séries de données uniques ou anormales dans une requête donnée. Un seul terme de recherche peut être visualisé comme une carte thermique.

    [Graphiques de carte thermique d’exploration Time Series Insights](media/time-series-insights-explorer/tsi-ga-example-heatmap-charting.png)] (media/time-series-insights-explorer/tsi-ga-example-heatmap-charting.png#lightbox)

1. Lorsque vous explorez des événements en les sélectionnant ou en cliquant dessus, le panneau **ÉVÉNEMENTS** est affiché. Ici, vous pouvez passer en revue tous les événements bruts, et exporter vos événements sous forme de fichiers JSON ou CSV. Azure Time Series Insights stocke toutes les données brutes.

    [![Événements](media/time-series-insights-explorer/tsi-ga-explorer-events-panel.png)](media/time-series-insights-explorer/tsi-ga-explorer-events-panel.png#lightbox)

1. Sélectionnez l’onglet **Statistiques** après avoir exploré les événements pour exposer des modèles et les statistiques de colonne.

    - **Modèles** : cette fonctionnalité fait ressortir de façon proactive les modèles statistiquement les plus significatifs dans une région de données sélectionnée. Vous n’avez pas à examiner plusieurs milliers d’événements pour comprendre les modèles qui nécessitent le plus de temps et d’énergie. Avec Azure Time Series Insights, vous pouvez accéder directement à ces modèles statistiquement significatifs pour poursuivre la réalisation d’une analyse. Cette fonctionnalité est également utile pour les enquêtes post mortem des données historiques.
    - **Statistiques de colonne** : Les statistiques de colonne fournissent des graphiques et des tables qui décomposent les données de chaque colonne de la série de données sélectionnée sur l’intervalle de temps sélectionné.

      [![Graphiques et options de la colonne STATISTIQUES](media/time-series-insights-explorer/tsi-ga-explorer-stat-column.png)](media/time-series-insights-explorer/tsi-ga-explorer-stat-column.png#lightbox)

Nous vous avons montré les fonctionnalités clés, les paramètres de configuration et les options d’affichage disponibles dans l’application web Explorateur Azure Time Series Insights.

## <a name="next-steps"></a>Étapes suivantes

- Découvrez comment [Diagnostiquer et résoudre les problèmes](time-series-insights-diagnose-and-solve-problems.md) dans votre environnement Azure Time Series Insights.

- Suivez la visite guidée du [démarrage rapide d’Azure Time Series Insights](time-series-quickstart.md).