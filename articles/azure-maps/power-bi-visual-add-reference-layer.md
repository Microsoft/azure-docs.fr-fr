---
title: Ajouter un calque de référence au visuel Power BI Azure Maps | Microsoft Azure Maps
description: Dans cet article, vous allez apprendre à utiliser le calque de référence dans le visuel Microsoft Azure Maps pour Power BI.
author: rbrundritt
ms.author: richbrun
ms.date: 06/26/2020
ms.topic: conceptual
ms.service: azure-maps
services: azure-maps
manager: cpendle
ms.custom: ''
ms.openlocfilehash: eb6c9fed42f263ca6cfaa1ea975d31cb8f4a75cf
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "86261406"
---
# <a name="add-a-reference-layer"></a>Ajouter une couche de référence

La fonctionnalité du calque de référence permet de charger un jeu de données spatiales secondaire sur le visuel et de le superposer sur la carte pour fournir un contexte supplémentaire. Ce jeu de données est hébergé par Power BI et doit être un [fichier GeoJSON](https://wikipedia.org/wiki/GeoJSON) avec une extension de fichier `.json` ou `.geojson`.

Pour ajouter un fichier **GeoJSON** comme calque de référence, accédez au volet **Format**, développez la section **Calque de référence** et appuyez sur le bouton **+ Ajouter un fichier local**.

Après l’ajout d’un fichier GeoJSON au calque de référence, le nom du fichier s’affiche à la place du bouton **+ Ajouter un fichier local** avec un **X** en regard. Appuyez sur le bouton **X** pour supprimer les données du visuel et supprimer le fichier GeoJSON de Power BI.

La carte suivante affiche les [secteurs de recensement 2016 pour le Colorado](https://github.com/Azure-Samples/AzureMapsCodeSamples/tree/master/AzureMapsCodeSamples/Common/data/geojson), colorée par population.

> [!div class="mx-imgBorder"]
> ![Carte affichant les secteurs de recensement 2016 pour le Colorado, colorée par population en tant que calque de référence ](media/power-bi-visual/reference-layer-CO-census-tract.png)

Vous trouverez ci-dessous tous les paramètres du **Format** disponibles dans la section **Calque de référence**.

| Paramètre              | Description   |
|----------------------|---------------|
| Données du calque de référence | Fichier GeoJSON à charger dans le visuel en tant que calque supplémentaire dans la carte. Le bouton **+ Ajouter un fichier local** ouvre une boîte de dialogue de fichier que l’utilisateur peut utiliser pour sélectionner un fichier GeoJSON qui a une extension de fichier `.json` ou `.geojson`. |

> [!NOTE]
> Dans cette préversion du visuel Azure Maps, le calque de référence chargera uniquement les 5 000 premières fonctionnalités de forme 5 000 dans la carte. Cette limite sera repoussée dans une mise à jour future.

## <a name="styling-data-in-a-reference-layer"></a>Stylisation de données dans un calque de référence

Des propriétés peuvent être ajoutées à chaque fonctionnalité dans le fichier GeoJSON pour personnaliser la façon dont elle est stylisée sur la carte. Cette fonctionnalité utilise la fonctionnalité de calque de données simple dans le kit de développement logiciel (SDK) Azure Maps Web. Pour plus d’informations, consultez ce document sur les [propriétés de style prises en charge](spatial-io-add-simple-data-layer.md#default-supported-style-properties). Les images d’icônes personnalisées ne sont pas prises en charge dans le visuel Azure Maps par mesure de sécurité.

Voici un exemple de fonctionnalité de point GeoJSON qui définit sa couleur affichée sur rouge.

```json
{
    "type": "Feature",
    "geometry": {
        "type": "Point",
        "coordinates": [-122.13284, 47.63699]
    },
    "properties": {
        "color": "red"
    }
}
```
## <a name="next-steps"></a>Étapes suivantes

Ajouter davantage de contexte à la carte :

> [!div class="nextstepaction"]
> [Ajouter une couche de mosaïques](power-bi-visual-add-tile-layer.md)

> [!div class="nextstepaction"]
> [Afficher le trafic en temps réel](power-bi-visual-show-real-time-traffic.md)
