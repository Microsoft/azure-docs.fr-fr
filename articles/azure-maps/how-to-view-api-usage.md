---
title: Afficher les métriques d’utilisation de l’API Azure Maps | Microsoft Azure Maps
description: Découvrez comment afficher les métriques d’utilisation de l’API Azure Maps, comme le nombre total de demandes, le nombre total d’erreurs et la disponibilité. Découvrez comment filtrer les données et fractionner les résultats.
author: anastasia-ms
ms.author: v-stharr
ms.date: 08/06/2018
ms.topic: how-to
ms.service: azure-maps
services: azure-maps
manager: timlt
ms.openlocfilehash: c9b732bd25e7ef8aa084c98d5b059d422f86a4b0
ms.sourcegitcommit: a43a59e44c14d349d597c3d2fd2bc779989c71d7
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/25/2020
ms.locfileid: "96003512"
---
# <a name="view-azure-maps-api-usage-metrics"></a>Afficher les métriques d’utilisation de l’API Azure Maps

Cet article vous montre comment afficher les mesures d’utilisation de l’API pour votre compte Azure Maps dans le [Portail Azure](https://portal.azure.com). Les métriques sont affichées dans un format graphique pratique le long d’une chronologie personnalisable.

## <a name="view-metric-snapshot"></a>Afficher l’instantané d’une métrique

Vous pouvez voir certaines métriques courantes sur la page **Vue d’ensemble** de votre compte Azure Maps. Elle affiche actuellement le *nombre total de demandes*, le *nombre total d’erreurs* et la *disponibilité* sur une période de temps sélectionnable.

![Vue d’ensemble des métriques d’utilisation d’Azure Maps](media/how-to-view-api-usage/portal-overview.png)

Si vous avez besoin de personnaliser ces graphiques pour votre analyse, passez à la section suivante.

## <a name="view-detailed-metrics"></a>Afficher des métriques détaillées

1. Connectez-vous à votre abonnement Azure sur le [portail](https://portal.azure.com).

2. Cliquez sur l’option de menu **Toutes les ressources** à gauche, puis accédez à votre *Compte Azure Maps*.

3. Une fois votre compte Azure Maps ouvert, cliquez sur le menu **Métriques** à gauche.

4. Dans le volet **Métriques**, choisissez l’une des options suivantes :

   1. **Disponibilité**, qui affiche la *Moyenne* de disponibilité de l’API sur une période de temps.
   2. **Utilisation**, qui affiche le *nombre* d’utilisations de votre compte.

      ![Volet des métriques d’utilisation d’Azure Maps](media/how-to-view-api-usage/portal-metrics.png)

5. Ensuite, vous pouvez sélectionner l’*Intervalle de temps* en cliquant sur **Dernières 24 heures (automatique)**. Par défaut, l’intervalle de temps est défini sur 24 heures. Après avoir cliqué, vous voyez tous les intervalles de temps sélectionnables. Vous pouvez sélectionner la *Granularité temporelle*, ainsi que choisir d’afficher l’heure *locale* ou *GMT* dans la même liste déroulante. Cliquez sur **Appliquer**.

    ![Intervalle de temps des métriques d’Azure Maps](media/how-to-view-api-usage/time-range.png)

6. Une fois que vous avez ajouté votre métrique, vous pouvez **Ajouter un filtre** à partir des propriétés correspondant à cette métrique. Ensuite, sélectionnez la valeur de la propriété que vous souhaitez voir reflétée sur le graphique.

    ![Filtre des métriques d’utilisation d’Azure Maps](media/how-to-view-api-usage/filter.png)

7. Vous pouvez également **Appliquer un fractionnement** à votre métrique en fonction de la propriété de la métrique sélectionnée. Cela permet de fractionner le graphique en plusieurs graphiques, un pour chaque valeur de cette propriété. Dans l’image suivante, la couleur de chaque graphique correspond à la valeur de propriété affichée au bas du graphique.

    ![Fractionnement des métriques d’utilisation d’Azure Maps](media/how-to-view-api-usage/splitting.png)

8. Vous pouvez également observer plusieurs métriques sur le même graphique, en cliquant simplement sur le bouton **Ajouter une métrique** en haut.

## <a name="next-steps"></a>Étapes suivantes

En savoir plus sur les API Azure Maps pour lesquelles vous souhaitez suivre l’utilisation :
> [!div class="nextstepaction"] 
> [Procédure SDK Web Azure Maps](how-to-use-map-control.md)

> [!div class="nextstepaction"] 
> [Procédure Android SDK Azure Maps](how-to-use-android-map-control-library.md)

> [!div class="nextstepaction"]
> [Documentation de l’API REST Azure Maps](/rest/api/maps)