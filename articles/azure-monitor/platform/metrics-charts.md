---
title: Fonctionnalités avancées d’Azure Metrics Explorer
description: En savoir plus sur les fonctionnalités avancées d’Azure Monitor Metrics Explorer
author: vgorbenko
services: azure-monitor
ms.topic: conceptual
ms.date: 01/22/2019
ms.author: vitalyg
ms.subservice: metrics
ms.openlocfilehash: be3d3f11e90c17bd8c4792418500da651039e480
ms.sourcegitcommit: 77ab078e255034bd1a8db499eec6fe9b093a8e4f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/16/2020
ms.locfileid: "97562801"
---
# <a name="advanced-features-of-azure-metrics-explorer"></a>Fonctionnalités avancées d’Azure Metrics Explorer

> [!NOTE]
> Cet article part du principe que vous êtes familiarisé avec les fonctionnalités de base de Metrics Explorer. Si vous êtes un nouvel utilisateur et que vous souhaitez apprendre à créer votre premier graphique de métriques, veuillez consulter l’article [Prise en main d’Azure Metrics Explorer](metrics-getting-started.md).

## <a name="metrics-in-azure"></a>Métriques dans Azure

Les [métriques dans Azure Monitor](data-platform-metrics.md) sont les séries de valeurs et de comptes mesurés qui sont collectées et stockées au fil du temps. Il existe des métriques standard (ou de « plateforme ») et des métriques personnalisées. Les métriques standards vous sont fournies par la plateforme Azure elle-même. Les métriques standards reflètent les statistiques d’intégrité et d’utilisation de vos ressources Azure. Tandis que vos applications envoient les mesures personnalisées à Azure à l’aide de l’[API Application Insights pour les événements et mesures personnalisés](../app/api-custom-events-metrics.md), [Extension Windows Azure Diagnostics (WAD)](./diagnostics-extension-overview.md), ou par l’[API REST Azure Monitor](./metrics-store-custom-rest-api.md).

## <a name="create-views-with-multiple-metrics-and-charts"></a>Créer des vues contenant plusieurs métriques et graphiques

Vous pouvez créer des graphiques constitués de plusieurs lignes de métriques ou afficher plusieurs graphiques de métriques à la fois. Cette fonctionnalité vous permet de :

- Mettre en corrélation les mesures associées sur le même graphique pour voir comment une valeur est liée à une autre
- Afficher les métriques avec différentes unités de mesure à proximité les unes des autres
- Agréger visuellement et comparer les métriques de plusieurs ressources

Par exemple, si vous avez 5 comptes de stockage et que vous souhaitez connaître la quantité d’espace total qu’ils consomment, vous pouvez créer un graphique en aires (empilées) qui illustre chaque valeur, individuellement, ainsi que leur somme, à différents moments dans le temps.

### <a name="multiple-metrics-on-the-same-chart"></a>Plusieurs métriques sur le même graphique

D’abord, [créez un graphique](metrics-getting-started.md#create-your-first-metric-chart). Cliquez sur **Ajouter une métrique** et répétez les étapes pour ajouter une autre métrique sur le même graphique.

   > [!NOTE]
   > Vous ne souhaiterez normalement pas avoir des métriques avec des unités de mesure différentes (par exemple, « millisecondes » et « Ko ») ou d’échelles très différentes sur un même graphique. Au lieu de cela, envisagez d’utiliser plusieurs graphiques. Pour créer plusieurs graphiques dans Metrics Explorer, cliquez sur le bouton Ajouter un graphique.

### <a name="multiple-charts"></a>Plusieurs graphiques

Cliquez sur **Ajouter un graphique** et créez un autre graphique avec une autre métrique.

### <a name="order-or-delete-multiple-charts"></a>Ordonner ou supprimer plusieurs graphiques

Pour ordonner ou supprimer plusieurs graphiques, cliquez sur le bouton de sélection ( **...** ) pour ouvrir le menu des graphiques, puis choisissez l’élément de menu approprié : **Monter**, **Descendre**, ou **Supprimer**.

## <a name="changing-aggregation"></a>Modification de l’agrégation

Lorsque vous ajoutez une métrique à un graphique, Metrics Explorer présélectionne automatiquement son agrégation par défaut. La valeur par défaut est logique dans les scénarios de base, mais vous pouvez utiliser une autre agrégation pour obtenir des informations supplémentaires sur la métrique. Pour afficher différentes agrégations sur un graphique, vous devez comprendre comment Metrics Explorer les gère. 

Les métriques sont la série de mesures (ou « valeurs métriques ») capturées au cours d’une période donnée. Lorsque vous tracez un graphique, les valeurs de la métrique sélectionnée sont agrégées séparément sur le *fragment de temps*. Vous sélectionnez la taille du fragment de temps [à l’aide du panneau du sélecteur d’heure Metrics Explorer](metrics-getting-started.md#select-a-time-range). Si vous n’effectuez pas de sélection explicite du fragment de temps, la granularité du temps est automatiquement sélectionnée en fonction de l’intervalle de temps actuellement sélectionné. Une fois que le fragment de temps est déterminé, les valeurs métriques qui ont été capturées pendant chaque intervalle de fragment de temps sont agrégées et placées sur le graphique (un point de données par fragment de temps).

Par exemple, supposons que le graphique indique la métrique **Temps de réponse du serveur** à l’aide de l’agrégation **Moyenne** au cours de l’intervalle de temps des **dernières 24 heures** :

- Si la granularité du temps est définie sur 30 minutes, le graphique est dessiné à partir de 48 points de données agrégés (par exemple, le graphique linéaire connecte 48 points dans la zone de traçage du graphique). Autrement dit, 24 heures x 2 points de données par heure. Chaque point de données représente la *moyenne* de tous les temps de réponse capturés pour les requêtes au serveur qui se sont produites pendant chacune des périodes de 30 minutes pertinentes.
- Si vous réglez la granularité du temps à 15 minutes, vous obtenez 96 points de données agrégés.  Autrement dit, 24 heures x 4 points de données par heure.

Metrics Explorer propose cinq types d’agrégation de statistiques de base : **Somme**, **Nombre**, **Min**, **Max** et **Moyenne**. L’agrégation **Somme** est parfois appelée l’agrégation **Total**. Pour de nombreuses mesures, Metrics Explorer masque les agrégations qui sont totalement inutiles et ne peuvent pas être utilisées.

- **Somme** : somme de toutes les valeurs capturées sur l’intervalle d’agrégation
- **Nombre** : nombre de mesures capturées au cours de l’intervalle d’agrégation. Notez que **Nombre** sera égal à **Somme** dans le cas où la mesure est toujours capturée avec la valeur 1. Cela est courant lorsque la métrique effectue le suivi du nombre d’événements distincts et que chaque mesure représente un événement (autrement dit, le code déclenche un enregistrement de métrique chaque fois qu’une nouvelle requête arrive)
- **Moyenne** : moyenne des valeurs métriques capturées sur l’intervalle d’agrégation
- **Min** : la plus petite valeur capturée sur l’intervalle d’agrégation
- **Max** : la plus grande valeur capturée sur l’intervalle d’agrégation

## <a name="apply-filters-to-charts"></a>Appliquer des filtres aux graphiques

Vous pouvez appliquer des filtres à des graphiques qui montrent des métriques associées à des dimensions. Par exemple, si la métrique « Nombre de transactions » a une dimension « Type de réponse » indiquant si la réponse provient de transactions qui ont réussi ou échoué, un filtrage sur cette dimension trace une courbe de graphique uniquement pour les transactions qui ont réussi ou échoué. 

### <a name="to-add-a-filter"></a>Pour ajouter un filtre

1. Sélectionnez **Ajouter un filtre** au-dessus du graphique

2. Sélectionnez la dimension (propriété) que vous souhaitez filtrer.

   ![Capture d’écran montrant les dimensions (propriétés) que vous pouvez filtrer.](./media/metrics-charts/028.png)

3. Sélectionnez les valeurs de dimension que vous souhaitez inclure lors du traçage du graphique (cet exemple illustre le filtrage des transactions de stockage qui ont réussi) :

   ![Capture d’écran montrant le filtrage des transactions de stockage réussies.](./media/metrics-charts/029.png)

4. Après avoir sélectionné les valeurs de filtre, cliquez à côté du sélecteur de filtre pour fermer celui-ci. Le graphique montre à présent le nombre de transactions de stockage qui ont échoué :

   ![Capture d’écran montrant le nombre de transactions de stockage ayant échoué.](./media/metrics-charts/030.png)

5. Vous pouvez répéter les étapes 1 à 4 pour appliquer plusieurs filtres aux mêmes graphiques.



## <a name="apply-splitting-to-a-chart"></a>Appliquer un fractionnement à un graphique

Vous pouvez fractionner une métrique par dimension afin de visualiser la façon dont les différents segments de la métrique se comparent entre eux, et d’identifier des segments éloignés d’une dimension.

### <a name="apply-splitting"></a>Appliquer la division

1. Cliquez sur **Appliquer un fractionnement** au-dessus du graphique.
 
   > [!NOTE]
   > Le fractionnement ne peut pas être utilisé avec des graphiques qui ont plusieurs métriques. En outre, vous pouvez avoir plusieurs filtres mais une seule dimension de fractionnement appliquée à n’importe quel graphique unique.

2. Choisissez la dimension sur laquelle vous souhaitez segmenter votre graphique :

   ![Capture d’écran montrant la dimension sélectionnée sur laquelle vous segmentez votre graphique.](./media/metrics-charts/031.png)

   Le graphique montre à présent plusieurs lignes, une par segment de la dimension :

   ![Capture d’écran montrant plusieurs lignes, une pour chaque segment de dimension.](./media/metrics-charts/032.png)

3. Cliquez à côté du **sélecteur de regroupement** pour fermer celui-ci.

   > [!NOTE]
   > Pour masquer les segments non pertinents pour votre scénario et faciliter la lecture des graphiques, utilisez un Filtrage et un Fractionnement sur la même dimension.

## <a name="lock-boundaries-of-chart-y-axis"></a>Verrouiller les limites de l’axe y du graphique

Lorsque le graphique montre des fluctuations plus faibles sur de plus grandes valeurs, il est important de verrouiller la plage de l’axe y. 

Par exemple, imaginons que le volume de requêtes réussies passe de 99,99 % à 99,95 %. Ces chiffres peuvent correspondre à une baisse significative de la qualité de service. Toutefois, avec les paramètres du graphique par défaut, il est très difficile voire impossible de remarquer une faible fluctuation des valeurs numériques. Dans ce cas, vous pouvez verrouiller la limite inférieure du graphique à 99 %. Cette petite baisse apparaît alors plus évidente. 

Autre exemple avec la fluctuation de la mémoire disponible, pour laquelle la valeur ne peut pas atteindre zéro (techniquement). Si la plage est définie sur une valeur plus élevée, il se peut que les chutes de la mémoire disponible soient plus faciles à repérer. 

Pour contrôler la plage de l’axe des y, utilisez le menu du graphique « … », puis sélectionnez **Paramètres du graphique** pour accéder aux paramètres avancés du graphique.

![Capture d’écran mettant en évidence l’option des paramètres du graphique.](./media/metrics-charts/033.png)

 Modifiez les valeurs de la section de la plage de l’axe y, ou utilisez le bouton **Auto** pour rétablir les valeurs par défaut.
 
 ![Capture d’écran mettant en évidence la section de la plage de l’axe Y.](./media/metrics-charts/034.png)

> [!WARNING]
> En général, pour verrouiller les limites de l’axe y pour les graphiques qui effectuent le suivi de nombres ou de sommes différents sur une période (et par conséquent, utilisent les agrégations de compte, somme, minimum ou maximum), vous devez spécifier une granularité temporelle fixe plutôt que de conserver les valeurs définies automatiquement par défaut. En effet, les valeurs des graphiques changent lorsque la granularité temporelle est modifiée automatiquement par l’utilisateur qui redimensionne sa fenêtre de navigation ou qui change de résolution d’écran. La modification de la granularité temporelle qui en découle affecte l’apparence du graphique et invalide la sélection actuelle de la plage de l’axe y.

## <a name="change-colors-of-chart-lines"></a>Modifier les couleurs des lignes des graphiques

Une fois les graphiques configurés, une couleur est automatiquement attribuée à leurs lignes à partir d'une palette par défaut. Vous pouvez modifier ces couleurs.

Pour changer la couleur d'une ligne, cliquez sur la barre de couleur dans la légende qui correspond au graphique. La boîte de dialogue du sélecteur de couleurs s'ouvre. Utilisez le sélecteur de couleurs pour configurer la couleur de la ligne.

![Capture d’écran montrant comment modifier la couleur.](./media/metrics-charts/035.png)

Une fois configurées, les couleurs sont conservées lorsque vous épinglez le graphique sur un tableau de bord. La section suivante explique comment épingler un graphique.

## <a name="pin-charts-to-dashboards"></a>Épingler des graphiques à des tableaux de bord

Après avoir configuré les graphiques, vous pouvez les ajouter aux tableaux de bord afin de pouvoir les réafficher, par exemple, dans le contexte d’une autre télémétrie de surveillance ou d’un partage avec votre équipe.

Pour épingler un graphique configuré à un tableau de bord :

Après avoir configuré votre graphique, cliquez sur **Épingler au tableau de bord** dans l’angle supérieur droit du graphique.

![Capture d’écran montrant comment épingler un graphique.](./media/metrics-charts/036.png)

## <a name="create-alert-rules"></a>Créer des règles d'alerte

Vous pouvez utiliser les critères que vous avez définis pour visualiser vos métriques en tant que base d’une règle d’alerte basée sur une métrique. La nouvelle règle d’alerte inclura les dimensions de la ressource, de la métrique, du fractionnement et du filtre cibles de votre graphique. Vous ne pourrez pas modifier ces paramètres ultérieurement dans le volet de création de règles d’alerte.

### <a name="to-create-a-new-alert-rule-click-new-alert-rule"></a>Pour créer une nouvelle règle d’alerte, cliquez sur **Nouvelle règle d’alerte**

![Bouton Nouvelle règle d’alerte surligné en rouge](./media/metrics-charts/042.png)

Vous êtes redirigé vers le volet de la création de règle d’alerte avec les dimensions de métriques sous-jacentes de votre graphique préremplies pour faciliter la génération de règles d’alerte personnalisées.

![Créer une règle d’alerte](./media/metrics-charts/041.png)

Lisez cet [article](alerts-metric.md) pour en savoir plus sur la configuration d’alertes de métrique.

## <a name="troubleshooting"></a>Dépannage

*Mon graphique ne contient aucune donnée.*

* Les filtres s’appliquent à tous les graphiques du volet. Lorsque vous vous concentrez sur un graphique, vérifiez que vous n’avez pas défini un filtre qui exclut toutes les données d’un autre graphique.

* Si vous souhaitez définir des filtres différents sur différents graphiques, créez-les dans des panneaux différents, enregistrez-les sous forme de favoris distincts. Si vous le souhaitez, vous pouvez les épingler au tableau de bord afin de les afficher parallèlement.

* Si vous segmentez un graphique en fonction d’une propriété qui n’est pas définie dans la mesure, alors il n’y aura rien à afficher sur le graphique. Essayez d’effacer la segmentation (fractionnement) ou sélectionnez une autre propriété.

## <a name="next-steps"></a>Étapes suivantes

  Pour découvrir les meilleures pratiques en matière de création de tableaux de bord actionnables avec des métriques, Voir [Créer des tableaux de bord d’indicateur de performance clé](../learn/tutorial-app-dashboards.md).
