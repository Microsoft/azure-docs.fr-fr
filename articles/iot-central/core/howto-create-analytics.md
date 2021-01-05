---
title: Analyser les données d’appareil dans votre application Azure IoT Central | Microsoft Docs
description: Analysez les données d’appareil dans votre application Azure IoT Central.
author: ankitscribbles
ms.author: ankitgup
ms.date: 11/27/2019
ms.topic: how-to
ms.service: iot-central
services: iot-central
manager: abjork
ms.openlocfilehash: 8f329b6377915a3df6e3f3e212cbd8d41f9ed80d
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90017658"
---
# <a name="how-to-use-analytics-to-analyze-device-data"></a>Guide pratique pour utiliser une analytique permettant d’analyser les données des appareils

*Cet article s’applique aux opérateurs, aux créateurs et aux administrateurs.*

Azure IoT Central fournit des fonctionnalités d’analyse enrichies pour analyser les tendances historiques et mettre en corrélation différentes données de télémétrie de vos appareils. Pour commencer, accédez à la zone **Analytique** du volet de gauche.

## <a name="understanding-the-analytics-ui"></a>Fonctionnement de l’interface utilisateur Analytique
L’interface utilisateur Analytique est constituée de trois composants principaux :
- **Panneau de configuration des données :** Dans le panneau de configuration, commencez par sélectionner le groupe d’appareils pour lequel vous souhaitez analyser les données. Ensuite, sélectionnez la télémétrie que vous souhaitez analyser et sélectionnez la méthode d’agrégation pour chaque télémétrie. Le contrôle **Diviser par** permet de regrouper les données à l’aide des propriétés de l’appareil en tant que dimensions.

- **Contrôle du temps :** Le contrôle du temps est utilisé pour sélectionner la durée pendant laquelle vous souhaitez analyser les données. Vous pouvez faire glisser l’une des extrémités du curseur de temps pour sélectionner l’intervalle de temps. Le contrôle d’heure a également un curseur **Taille d’intervalle** qui contrôle le compartiment ou la taille d’intervalle utilisée pour agréger les données. 

- **Contrôle de graphique :** Le contrôle de graphique visualise les données sous forme de graphique en courbes. Vous pouvez activer ou désactiver la visibilité de lignes spécifiques en interagissant avec la légende du graphique. 


  ![Vue d’ensemble de l’interface utilisateur Analytique](media/howto-create-analytics/analyticsui.png)


## <a name="querying-your-data"></a>Interrogation de vos données

Vous devez commencer par choisir un **groupe d’appareils** et les données de télémétrie que vous souhaitez analyser. Quand vous avez terminé, sélectionnez **Analyser** pour commencer à visualiser vos données.

- **Groupe d’appareils :** Un [groupe d’appareils](tutorial-use-device-groups.md) est un groupe de vos appareils défini par l’utilisateur. Par exemple, tous les réfrigérateurs de Paris ou la totalité des éoliennes version 2.0.

- **Données de télémétrie :** Sélectionnez les données de télémétrie que vous souhaitez analyser et explorer. Vous pouvez sélectionner plusieurs données de télémétrie à analyser ensemble. La méthode d’agrégation par défaut est définie sur Moyenne pour les valeurs numériques et Nombre pour le type de données chaîne, respectivement. Les méthodes d’agrégation prises en charge pour les types de données numériques sont Moyenne, Maximum, Minimum, Nombre et Somme.  Les méthodes d’agrégation prises en charge pour le type de données chaîne sont Nombre.

- **Fractionner par** : Le contrôle Diviser par permet de regrouper les données à l’aide des propriétés de l’appareil en tant que dimensions. Les valeurs des propriétés de l’appareil et du cloud sont jointes à la télémétrie telles qu’envoyées par l’appareil. Si une propriété cloud ou de l’appareil est mise à jour, vous verrez les données de télémétrie regroupées par différentes valeurs sur le graphique.

    > [!TIP]
    > Pour afficher les données de chaque appareil séparément, sélectionnez l’ID de l’appareil dans le contrôle « Diviser par ».

## <a name="interacting-with-your-data"></a>Interaction avec vos données

Une fois que vous avez interrogé vos données, vous pouvez commencer à les visualiser sur le graphique en courbes. Vous pouvez afficher/masquer les données de télémétrie, modifier la durée et afficher la télémétrie dans une grille de données.

- **Panneau de l’éditeur d’heure :** Par défaut, nous allons récupérer les données du jour passé. Vous pouvez faire glisser l’une des extrémités du curseur de temps pour modifier l’intervalle de temps. Vous pouvez également utiliser le contrôle de calendrier pour sélectionner un des intervalles de temps prédéfinis ou sélectionner un intervalle de temps personnalisé. Le contrôle d’heure a également un curseur **Taille d’intervalle** qui contrôle le compartiment ou la taille d’intervalle utilisée pour agréger les données.

    ![Éditeur d’heure](media/howto-create-analytics/timeeditorpanel.png)

    - **Outil de curseur de plage de dates interne** : utilisez les deux contrôles de point de terminaison en les faisant glisser sur l’intervalle de temps souhaité. Cette plage de dates interne est limitée par le contrôle de curseur de plage de dates externe.
    
   
    - **Contrôle de curseur de plage de dates externe** : utilisez les contrôles de point de terminaison pour sélectionner la plage de dates externe, qui sera disponible pour votre contrôle de plage de dates interne.

    - **Boutons pour augmenter ou diminuer la plage de dates** : augmentez ou réduisez votre intervalle de temps en sélectionnant l’un des boutons pour l’intervalle souhaité.

    - **Curseur de taille d’intervalle** : permet d’effectuer un zoom avant et arrière sur des intervalles d’un même intervalle de temps. Cette action permet un contrôle plus précis du déplacement entre les tranches de temps volumineuses. Vous pouvez l’utiliser pour afficher des vues granulaires de vos données en haute résolution, à jusqu’à la milliseconde près. Le point de départ par défaut du curseur est défini comme la vue optimale des données à partir de votre sélection, avec un équilibre entre résolution, vitesse de requête et granularité.
    
    - **Sélecteur de plages de dates** : vous permet de sélectionner facilement les plages de dates et d’heures souhaitées. Vous pouvez également utiliser le contrôle pour basculer entre les différents fuseaux horaires. Une fois que vous avez apporté des modifications, pour les appliquer à votre espace de travail actuel, sélectionnez Enregistrer.

    > [!TIP]
    > La taille de l’intervalle est déterminée dynamiquement en fonction de l’intervalle de temps sélectionné. Des délais plus petits permettront d’agréger les données en intervalles très granulaires de jusqu’à quelques secondes.


- **Légende du graphique :** La légende du graphique affiche les données de télémétrie sélectionnées sur le graphique. Vous pouvez pointer sur chaque élément de la légende pour l’activer sur le graphique. Lors de l’utilisation de l’option « Diviser par », les données de télémétrie sont regroupées par valeurs respectives de la dimension sélectionnée. Vous pouvez activer ou désactiver la visibilité de chaque donnée de télémétrie spécifique ou de l’ensemble du groupe en cliquant sur le nom du groupe.  


- **Contrôle du format de l’axe Y :** le mode de l’axe Y parcourt les options d’affichage de l’axe Y disponibles. Ce contrôle est disponible uniquement lorsque différentes données de télémétrie sont visualisées. Vous pouvez définir l’axe Y en choisissant un des trois modes suivants :

    - **Empilé :** un graphique pour chaque donnée de télémétrie est empilé et chacun des graphiques a son propre axe Y. Ce mode est défini par défaut.
    - **Partagé :** Un graphique pour chaque données de télémétrie est tracé sur le même axe Y.
    - **Chevauchement :** permet d’empiler plusieurs lignes sur le même axe y, celui-ci étant modifié en fonction de la ligne sélectionnée.

  ![Organiser les données sur l’axe Y avec différents modes de visualisation](media/howto-create-analytics/yaxiscontrol.png)

- **Contrôle de zoom :** Le zoom vous permet d’analyser vos données de plus près. Si vous trouvez une période sur laquelle vous souhaitez vous concentrer dans votre jeu de résultats, utilisez le pointeur de la souris pour saisir la zone, puis faites-la glisser vers le point de terminaison de votre choix. Cliquez ensuite avec le bouton droit sur la zone sélectionnée, puis cliquez sur Zoom.

  ![Zoom sur les données](media/howto-create-analytics/zoom.png)

Sous les points de suspension, il y a plus de contrôles de graphique pour interagir avec les données :

- **Afficher la grille :** vos résultats sont disponibles sous forme de tableau, ce qui vous permet de voir la valeur de chaque point de données.

- **Télécharger au format CSV :** Vos résultats peuvent être exportés sous la forme d’un fichier de valeurs séparées par des virgules (CSV). Ce fichier CSV contient des données pour chaque appareil. Les résultats sont exportés en utilisant l’intervalle et le délai spécifiés. 

- **Placer un marqueur :** Le contrôle « Placer un marqueur » permet d’ancrer certains points de données sur le graphique. Il est utile lorsque vous tentez de comparer des données pour plusieurs lignes sur différentes périodes.

  ![Affichage du mode grille pour votre analytique](media/howto-create-analytics/additionalchartcontrols.png)
