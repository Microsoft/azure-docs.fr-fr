---
title: Configurer votre tableau de bord Azure IoT Central | Microsoft Docs
description: En tant que concepteur, découvrez comment configurer le tableau de bord de l’application d’Azure IoT Central par défaut avec des vignettes.
author: TheJasonAndrew
ms.author: v-anjaso
ms.date: 12/19/2020
ms.topic: how-to
ms.service: iot-central
ms.openlocfilehash: 73c78f514f974b97eb178177f12dc4ceff7ede63
ms.sourcegitcommit: ab829133ee7f024f9364cd731e9b14edbe96b496
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/28/2020
ms.locfileid: "97797007"
---
# <a name="configure-the-application-dashboard"></a>Configurer le tableau de bord de l’application

Le **tableau de bord** est la première page que vous voyez lorsque vous vous connectez à une application Azure IoT Central. Si vous avez créé votre application à partir de l’un des [modèles d’application](./concepts-app-templates.md) axés sur le secteur, votre application dispose d’un tableau de bord prédéfini pour commencer. Si vous créez votre application à partir d’un [modèle d’application](./concepts-app-templates.md) personnalisé, votre tableau de bord affiche quelques conseils pour commencer.

> [!TIP]
> Les utilisateurs peuvent [créer plusieurs tableaux de bord](howto-create-personal-dashboards.md) en plus du tableau de bord par défaut de l’application. Ces tableaux de bord peuvent être personnels pour l’utilisateur ou partagés par tous les utilisateurs de l’application.  

## <a name="add-tiles"></a>Ajouter des vignettes

La capture d’écran suivante montre le tableau de bord dans une application créée à partir du modèle **Application personnalisée**. Pour personnaliser le tableau de bord actuel, sélectionnez **Modifier**, pour ajouter un tableau de bord personnalisé ou partagé, sélectionnez **Nouveau**:

:::image type="content" source="media/howto-add-tiles-to-your-dashboard/dashboard-sample-contoso.png" alt-text="Tableau de bord des applications basées sur le modèle d’application personnalisée":::

Une fois que vous avez sélectionné **Modifier** ou **Nouveau**, le tableau de bord est en *mode d’édition*. Vous pouvez utiliser les outils du panneau **Modifier le tableau de bord** pour ajouter des vignettes au tableau de bord, personnaliser et supprimer des vignettes sur le tableau de bord lui-même. Par exemple, pour ajouter une vignette **Télémétrie** afin d’afficher la température actuelle signalée par un ou plusieurs appareils :

1. Sélectionnez un **groupe d’appareils**, puis choisissez vos appareils à afficher sur la vignette dans la liste déroulante **Appareils**. Vous voyez maintenant les données de télémétrie, propriétés et commandes disponibles à partir des appareils.

1. Si nécessaire, utilisez la liste déroulante pour sélectionner une valeur de télémétrie à afficher sur la vignette. Vous pouvez ajouter d’autres éléments à la vignette en sélectionnant **+ Télémétrie**, **+ Propriété** ou **+ Propriété cloud**.

:::image type="content" source="media/howto-add-tiles-to-your-dashboard/device-details.png" alt-text="Ajouter une vignette de télémétrie de température au tableau de bord":::

Une fois que vous avez sélectionné toutes les valeurs à afficher sur la vignette, cliquez sur **Ajouter la vignette**. La vignette s’affiche maintenant dans le tableau de bord où vous pouvez modifier la visualisation, redimensionner la vignette, la déplacer et la configurer.

Une fois que vous avez fini d’ajouter et de personnaliser des vignettes sur le tableau de bord, sélectionnez **Enregistrer** pour enregistrer les modifications apportées au tableau de bord, ce qui vous permet de sortir du mode d’édition.

## <a name="customize-tiles"></a>Personnaliser des vignettes

Pour modifier une vignette, vous devez être en mode d’édition.  Les options de personnalisation disponibles dépendent du [type de vignette](#tile-types) :

* L’icône de règle d’une vignette vous permet de modifier la visualisation. Les visualisations incluent des graphiques en courbes, des graphiques à barres, des graphiques à secteurs, les dernières valeurs connues (LKV, last known values), des indicateurs de performance clés (KPI), des cartes thermiques et des cartes.

* L’icône carrée vous permet de redimensionner la vignette.

* L’icône d’engrenage vous permet de configurer la visualisation. Par exemple, pour une visualisation de graphique en courbes, vous pouvez choisir d’afficher la légende et les axes, puis choisir l’intervalle de temps à tracer.


## <a name="tile-types"></a>Types de vignettes

Le tableau suivant décrit les différents types de vignettes que vous pouvez ajouter à un tableau de bord :

| Vignette             | Description |
| ---------------- | ----------- |
| Markdown         | Les vignettes Markdown sont des vignettes interactives qui affichent un en-tête et un texte de description formatés à l’aide de Markdown. L’URL peut être un lien relatif menant à une autre page de l’application, ou un lien absolu menant à un site externe.|
| Image            | Les vignettes d’image affichent une image personnalisée et peuvent être interactives. L’URL peut être un lien relatif menant à une autre page de l’application, ou un lien absolu menant à un site externe.|
| Étiquette            | Les vignettes d’étiquette affichent un texte personnalisé sur un tableau de bord. Vous pouvez choisir la taille du texte. Utilisez une vignette d’étiquette pour ajouter des informations pertinentes au tableau de bord, telles que des descriptions, des informations de contact ou une assistance.|
| Count            | Les vignettes Nombre affichent le nombre d’appareils qui se trouvent dans un groupe d’appareils.|
| Mappage              | Les vignettes Carte affichent l’emplacement d’un appareil sur une carte. Vous pouvez également afficher jusqu'à 100 points de l'historique des emplacements d'un appareil. Par exemple, vous pouvez afficher un itinéraire détaillant les emplacements d’un appareil au cours de la semaine dernière.|
| KPI              |  Les vignettes KPI affichent des valeurs de télémétrie agrégées pour un ou plusieurs appareils sur une période donnée. Par exemple, vous pouvez les utiliser pour afficher la température et la pression maximales atteintes par un ou plusieurs appareils durant la dernière heure.|
| Graphique en courbes       | Les vignettes Graphique en courbes tracent une ou plusieurs valeurs de télémétrie agrégées pour un ou plusieurs appareils sur une période donnée. Par exemple, vous pouvez afficher un graphique en courbes pour tracer la température et la pression moyennes d’un ou plusieurs appareils au cours de la dernière heure.|
| Graphique à barres        | Les vignettes Graphique à barres tracent une ou plusieurs valeurs de télémétrie agrégées pour un ou plusieurs appareils sur une période donnée. Par exemple, vous pouvez afficher un graphique à barres pour tracer la température et la pression moyennes d’un ou plusieurs appareils au cours de la dernière heure.|
| Graphique à secteurs        | Les vignettes Graphique à secteurs affichent une ou plusieurs valeurs de télémétrie agrégées pour un ou plusieurs appareils sur une période donnée.|
| Carte thermique         | Les vignettes Carte thermique présentent des informations sur un ou plusieurs appareils sous forme de couleurs.|
| Dernière valeur connue | Les vignettes Dernière valeur connue affichent les dernières valeurs de télémétrie pour un ou plusieurs appareils. Par exemple, vous pouvez utiliser cette vignette pour afficher les valeurs de température, de pression et d’humidité les plus récentes d’un ou plusieurs appareils. |
| Historique des événements    | Les vignettes d’historique des événements affichent les événements d’un appareil sur une période de temps. Par exemple, vous pouvez l’utiliser pour afficher tous les événements d’ouverture et de fermeture de vanne pour un ou plusieurs appareils au cours de la dernière heure.|
| Propriété         |  Les vignettes Propriété affichent la valeur actuelle des propriétés et propriétés cloud d’un ou plusieurs appareils. Par exemple, vous pouvez utiliser cette vignette pour afficher les propriétés d’un appareil, comme son fabricant ou la version de son microprogramme. |

Actuellement, vous pouvez ajouter jusqu’à 10 appareils à des vignettes prenant en charge plusieurs appareils.

### <a name="customizing-visualizations"></a>Personnalisation des visualisations

Par défaut, les graphiques en courbes affichent les données sur un intervalle de temps. L’intervalle de temps sélectionné est fractionné en 50 compartiments de taille égale, et les données de l’appareil sont ensuite regroupées par compartiment pour fournir 50 points de données sur l’intervalle de temps sélectionné. Si vous souhaitez afficher les données brutes, vous pouvez modifier votre sélection pour afficher les 100 dernières valeurs. Pour modifier l’intervalle de temps ou pour sélectionner la visualisation des données brutes, utilisez la liste déroulante Intervalle d’affichage dans le panneau **Configurer le graphique**.

:::image type="content" source="media/howto-add-tiles-to-your-dashboard/display-range.png" alt-text="Modifier l’intervalle d’affichage d’un graphique en courbes":::

Pour les vignettes affichant des valeurs agrégées, sélectionnez l’icône d’engrenage en regard du type de données de télémétrie dans le panneau **Configurer le graphique** afin de choisir l’agrégation. Vous pouvez choisir entre moyenne, somme, maximum, minimum et nombre.

Pour les graphiques en courbes, à barres et à secteurs, vous pouvez personnaliser la couleur des différentes valeurs de télémétrie. Sélectionnez l’icône de palette en regard des données de télémétrie que vous souhaitez personnaliser :

:::image type="content" source="media/howto-add-tiles-to-your-dashboard/color-customization.png" alt-text="Modifier la couleur d’une valeur de télémétrie":::

Pour les vignettes affichant des propriétés de chaîne ou des valeurs de télémétrie, vous pouvez choisir comment afficher le texte. Par exemple, si l’appareil stocke une URL dans une propriété de chaîne, vous pouvez l’afficher sous forme de lien sur lequel vous pouvez cliquer. Si l’URL fait référence à une image, vous pouvez restituer celle-ci dans une vignette de dernière valeur connue ou de propriété. Pour modifier la manière dont une chaîne s’affiche, dans la configuration de la vignette, sélectionnez l’icône d’engrenage en regard du type de télémétrie ou de la propriété  :

:::image type="content" source="media/howto-add-tiles-to-your-dashboard/string-customization.png" alt-text="Modifier la manière dont une chaîne s’affiche sur une vignette":::

Pour les vignettes **KPI**, **Dernière valeur connue**, et **Propriété**, vous pouvez utiliser la mise en forme conditionnelle pour personnaliser la couleur de la vignette en fonction de sa valeur actuelle. Pour ajouter une mise en forme conditionnelle, sélectionnez **Configurer** sur la vignette, puis sélectionnez l’icône **Mise en forme conditionnelle** en regard de la valeur à personnaliser :

:::image type="content" source="media/howto-add-tiles-to-your-dashboard/conditional-formatting-1.png" alt-text="Capture d’écran montrant comment trouver l’option Configurer pour une vignette, puis l’icône de mise en forme conditionnelle":::

Ajoutez vos règles de mise en forme conditionnelle :

:::image type="content" source="media/howto-add-tiles-to-your-dashboard/conditional-formatting-2.png" alt-text="Capture d’écran montrant les règles de mise en forme conditionnelle pour le flux moyen. Il existe trois règles : moins de 20 est vert, moins de 50 est jaune et tout ce qui se trouve au-dessus de 50 est rouge":::
   
La capture d’écran suivante montre l’effet de la règle de mise en forme conditionnelle :

:::image type="content" source="media/howto-add-tiles-to-your-dashboard/conditional-formatting-3.png" alt-text="Capture d’écran montrant la couleur d’arrière-plan rouge sur la vignette Débit d’eau moyen. Le nombre sur la vignette est 50,54":::

### <a name="tile-formatting"></a>Mise en forme de « vignette »
Cette fonctionnalité, disponible dans les vignettes KPI, LKV et Propriété, permet aux utilisateurs d’ajuster la taille de police, de choisir la précision décimale, d’abréger les valeurs numériques (par exemple, format 1 700 en 1,7K) ou de renvoyer à la ligne les valeurs de chaîne dans leurs vignettes.

:::image type="content" source="media/howto-add-tiles-to-your-dashboard/tile-format.png" alt-text="Format de la vignette":::

## <a name="next-steps"></a>Étapes suivantes

À présent que vous savez comment configurer votre tableau de bord d’application par défaut Azure IoT Central, découvrez comment [Créer un tableau de bord personnel](howto-create-personal-dashboards.md).
