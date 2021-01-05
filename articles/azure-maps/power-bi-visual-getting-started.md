---
title: Prise en main du visuel Power BI Azure Maps | Microsoft Azure Maps
description: Dans cet article, vous allez apprendre à utiliser le visuel Microsoft Azure Maps pour Power BI.
author: rbrundritt
ms.author: richbrun
ms.date: 12/07/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: cpendle
ms.custom: ''
ms.openlocfilehash: a563a7776e84289e38743057778e8fe10fd17503
ms.sourcegitcommit: 80c1056113a9d65b6db69c06ca79fa531b9e3a00
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/09/2020
ms.locfileid: "96904701"
---
# <a name="getting-started-with-the-azure-maps-power-bi-visual"></a>Prise en main du visuel Power BI Azure Maps

<Token>**S’APPLIQUE À :** ![Coche verte.](media/power-bi-visual/yes.png) Service Power BI pour les *_consommateurs_* _ ![Green check mark.](media/power-bi-visual/yes.png) Service Power BI pour les concepteurs et les développeurs ![Coche verte.](media/power-bi-visual/yes.png) Power BI Desktop ![X signifiant Non.](media/power-bi-visual/no.png) Nécessite une licence Pro ou Premium</Token>

Cet article explique comment utiliser le visuel Microsoft Azure Maps pour Power BI.

> [!NOTE]
> Ce visuel peut être créé et affiché à la fois dans Power BI Desktop et dans le service Power BI. Les étapes et les illustrations dans cet article proviennent de Power BI Desktop.

Le visuel Azure Maps pour Power BI fournit un ensemble complet de visualisations de données pour les données spatiales sur une carte. On estime que plus de 80 % des données d’entreprise possèdent un contexte d’emplacement. Le visuel Azure Maps peut être utilisé pour obtenir des insights sur la façon dont ce contexte d’emplacement est associé aux données de votre entreprise et les influence.

![Power BI Desktop avec le visuel Azure Maps affichant des données d’entreprise](media/power-bi-visual/azure-maps-visual-hero.png)

## <a name="what-is-sent-to-azure"></a>Qu’est-ce qui est envoyé à Azure ?

Le visuel Azure Maps se connecte au service cloud hébergé dans Azure pour récupérer des données d’emplacement, comme les images cartographiques et les coordonnées utilisées pour créer la visualisation de la carte. 

-   Les informations sur la zone sur laquelle la carte est axée sont envoyées à Azure afin de récupérer les images nécessaires au rendu du canevas de la carte (également appelée vignettes de carte). 
-   Les données des compartiments Emplacement, Latitude et Longitude peuvent être envoyées à Azure pour récupérer les coordonnées de la carte (processus appelé géocodage). 
-   Les données de télémétrie peuvent être collectées sur l’intégrité du visuel (c.-à-d. les rapports d’incidents), si l’option de télémétrie est activée dans Power BI.

À part les scénarios décrits ci-dessus, aucune autre donnée superposée sur la carte n’est envoyée aux serveurs Azure Maps. Tout le rendu des données se produit localement au sein du client.

Vous, ou votre administrateur, devrez peut-être mettre à jour votre pare-feu pour autoriser l’accès à la plateforme Azure Maps qui utilise l’URL suivante.

> `https://atlas.microsoft.com`

Pour en savoir plus à propos de la confidentialité et des conditions d’utilisation associées au visuel Azure Maps, consultez [Mentions légales Microsoft Azure](https://azure.microsoft.com/support/legal/).

## <a name="azure-maps-visual-preview-behavior-and-requirements"></a>Comportement et configuration requise du visuel Azure Maps (préversion)

Voici quelques considérations et exigences concernant le visuel _ *Azure Maps** :

-   Le visuel **Azure Maps** (préversion) doit être activé dans Power BI Desktop. Pour activer le visuel **Azure Maps**, sélectionnez **Fichier** &gt; **Options et paramètres** &gt; **Options** &gt; **Fonctionnalités en préversion**, puis cochez la case **Visuel Azure Maps**. Si le visuel Azure Maps n’est pas disponible après cela, cela signifie sans doute qu’un commutateur administrateur de locataire dans le Portail d’administration doit être activé.
-   Le jeu de données doit inclure des champs qui contiennent des informations sur la **latitude** et la **longitude**. Le géocodage des champs d’emplacement sera ajouté dans une prochaine mise à jour.
-   Le contrôle de légende intégré pour Power BI n’apparaît pas dans cette préversion. Il sera ajouté dans une prochaine mise à jour.

## <a name="use-the-azure-maps-visual-preview"></a>Utiliser le visuel Azure Maps (préversion)

Une fois que le visuel **Azure Maps** est activé, sélectionnez l’icône **Azure Maps** dans le volet **Visualisations**.

![Bouton du visuel Azure Maps dans le volet Visualisations](media/power-bi-visual/azure-maps-in-visualizations-pane.png)

Power BI crée un canevas de conception de visuel Azure Maps vide. Dans la préversion, une clause d’exclusion de responsabilité supplémentaire est affichée.

![Power BI Desktop avec le visuel Azure Maps chargé dans son état initial](media/power-bi-visual/visual-initial-load.png)

Procédez comme suit pour charger le visuel Azure Maps :

1.  Dans le volet **Champs**, faites glisser les champs de données qui contiennent des informations sur les coordonnées de latitude et de longitude vers les compartiments **Latitude** et/ou **Longitude**. Il s’agit des données minimales nécessaires au chargement du visuel Azure Maps.
    
    > [!div class="mx-imgBorder"]
    > ![Visuel Azure Maps affichant des points en tant que bulles sur la carte une fois les champs de latitude et de longitude renseignés](media/power-bi-visual/bubble-layer.png)

2.  Pour coloriser les données en fonction de la catégorisation, faites glisser un champ de catégorie vers le compartiment **Légende** du volet **Champs**. Dans cet exemple, nous utilisons la colonne **AdminDistrict** (également appelée État ou Province).  
    
    > [!div class="mx-imgBorder"]
    > ![Visuel Azure Maps affichant des points en tant que bulles colorées sur la carte une fois le champ de légende renseigné](media/power-bi-visual/bubble-layer-with-legend-color.png)

    > [!NOTE]
    > Le contrôle de légende intégré pour Power BI n’apparaît pas dans cette préversion. Il sera ajouté dans une prochaine mise à jour.

3.  Pour mettre à l’échelle les données de manière relative, faites glisser une mesure vers le compartiment **Taille** du volet **Champs**. Dans cet exemple, nous utilisons la colonne **Sales** (Ventes).  
    
    > [!div class="mx-imgBorder"]
    > ![Visuel Azure Maps affichant des points en tant que bulles colorées et à l’échelle sur la carte une fois le champ de taille renseigné.](media/power-bi-visual/bubble-layer-with-legend-color-and-size.png)

4.  Utilisez les options du volet **Format** pour personnaliser le rendu des données. L’image suivante est identique à la carte ci-dessus, mais avec l’option de transparence de remplissage des calques de bulles définie sur 50 % et l’option de contour à contraste élevé activée.  
    
    > [!div class="mx-imgBorder"]
    > ![Visuel Azure Maps affichant des points en tant que bulles sur la carte avec un style personnalisé](media/power-bi-visual/bubble-layer-styled.png)

## <a name="fields-pane-buckets"></a>Compartiments du volet Champs

Les compartiments de données suivants sont disponibles dans le volet **Champs** du visuel Azure Maps.

| Champ     | Description  |
|-----------|--------------|
| Latitude  | Champ utilisé pour spécifier la valeur de la latitude des points de données. Les valeurs de latitude doivent être comprises entre -90 et 90 degrés au format décimal.  |
| Longitude | Champ utilisé pour spécifier la valeur de la longitude des points de données. Les valeurs de longitude doivent être comprises entre -180 et 180 degrés au format décimal.  |
| Légende    | Champ utilisé pour catégoriser les données et assigner une couleur unique aux points de données dans chaque catégorie. Quand ce compartiment est rempli, une section **Couleurs des données** permettant d’ajuster les couleurs s’affiche dans le volet **Format**. |
| Taille      | Mesure utilisée pour le dimensionnement relatif des points de données sur la carte.   |
| Info-bulles  | Champs de données supplémentaires affichés dans les info-bulles lorsque les formes sont survolées avec la souris. |

## <a name="map-settings"></a>Paramètres de la carte

La section **Paramètres de la carte** du volet Format fournit des options pour personnaliser l’affichage de la carte et sa réaction aux mises à jour.

| Paramètre             | Description  |
|---------------------|--------------|
| Zoom automatique           | Effectue automatiquement un zoom de la carte sur les données chargées via le volet **Champs** du visuel. À mesure que les données changent, la carte met à jour sa position en conséquence. Lorsque le curseur se trouve dans la position **Off** (désactivé), des paramètres de vue cartographique supplémentaires sont affichés pour la vue cartographique par défaut. |
| Enveloppe mondiale          | Permet à l’utilisateur de faire pivoter la carte horizontalement de manière infinie. |
| Sélecteur de styles        | Ajoute un bouton à la carte qui permet aux lecteurs de rapport de modifier le style de la carte. |
| Commandes de navigation | Ajoute des boutons à la carte comme méthode alternative pour permettre aux lecteurs de rapport de faire un zoom, une rotation et de modifier l’inclinaison de la carte. Pour plus d’informations, consultez ce document [Navigation dans la carte](map-accessibility.md#navigating-the-map) pour obtenir des détails sur les différentes façons dont les utilisateurs peuvent naviguer dans la carte. |
| Style de carte           | Style de la carte. Pour plus d’informations, consultez ce document sur [les styles de carte pris en charge](supported-map-styles.md). |

### <a name="map-view-settings"></a>Paramètres de l’affichage de la carte

Si le curseur **Zoom automatique** se trouve dans la position **Off** (désactivé), les paramètres suivants s’affichent et permettent à l’utilisateur de spécifier les informations d’affichage de la carte par défaut.

| Paramètre          | Description   |
|------------------|---------------|
| Zoom             | Niveau de zoom par défaut de la carte. Peut être un nombre compris entre 0 et 22. |
| Latitude du centre  | Latitude par défaut au centre de la carte. |
| Longitude du centre | Longitude par défaut au centre de la carte. |
| Direction          | Orientation par défaut de la carte en degrés, où 0 correspond au nord, 90 à l’est, 180 au sud et 270 à l’ouest. Peut être un nombre compris entre 0 et 360. |
| Inclinaison            | Inclinaison par défaut de la carte, en degrés, entre 0 et 60, où 0 correspond à un plan parfaitement horizontal de la carte. |

## <a name="considerations-and-limitations"></a>Observations et limitations

Le visuel Azure Maps est disponible dans les services et applications suivants :

| Service/Application                              | Disponibilité |
|------------------------------------------|--------------|
| Power BI Desktop                         | Oui          |
| Service Power BI (app.powerbi.com)       | Oui          |
| Applications mobiles Power BI             | Oui          |
| Publier sur le web Power BI                  | Non           |
| Power BI Embedded                        | Non           |
| Intégration de service Power BI (PowerBI.com) | Oui          |

La prise en charge de services/applications Power BI supplémentaires sera ajoutée dans les futures mises à jour.

**Où la fonctionnalité Azure Maps est-elle disponible ?**

Pour le moment, Azure Maps est disponible dans tous les pays et régions, à l’exception de :

- Chine
- Corée du Sud

Pour plus d’informations sur la couverture des différents services Azure Maps sur lesquels repose ce visuel, consultez le document [Informations sur la couverture géographique](geographic-coverage.md).

**Quels sont les navigateurs web pris en charge par le visuel Azure Maps ?**

Consultez cette documentation pour plus d’informations sur [les navigateurs pris en charge par le kit SDK web Azure Maps](supported-browsers.md).

**Combien de points de données puis-je visualiser ?**

Ce visuel prend en charge jusqu’à 30 000 points de données.

**Est-il possible d’utiliser des adresses ou d’autres chaînes d’emplacement dans ce visuel ?**

La préversion initiale de ce visuel ne prend en charge que les valeurs de latitude et de longitude en degrés décimaux. Une mise à jour ultérieure ajoutera la prise en charge des adresses et autres chaînes d’emplacement.

## <a name="next-steps"></a>Étapes suivantes

Découvrez-en plus sur le visuel Power BI Azure Maps :

> [!div class="nextstepaction"]
> [Présentation des calques dans le visuel Power BI Azure Maps](power-bi-visual-understanding-layers.md)

> [!div class="nextstepaction"]
> [Gérer le visuel Azure Maps au sein de votre organisation](power-bi-visual-manage-access.md)

Personnaliser le visuel :

> [!div class="nextstepaction"]
> [Astuces et conseils pour la mise en forme des couleurs dans Power BI](/power-bi/visuals/service-tips-and-tricks-for-color-formatting)

> [!div class="nextstepaction"]
> [Personnaliser les titres, les légendes et les arrière-plans des visualisations](/power-bi/visuals/power-bi-visualization-customize-title-background-and-legend)