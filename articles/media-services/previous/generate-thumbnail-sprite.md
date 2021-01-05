---
title: Générer un sprite de miniatures avec Azure Media Services | Microsoft Docs
description: Cette rubrique montre comment générer un sprite de miniatures avec Azure Media Services.
services: media-services
documentationcenter: ''
author: Juliako
manager: femila
editor: ''
ms.service: media-services
ms.workload: ''
ms.topic: article
ms.date: 08/31/2020
ms.author: juliako
ms.openlocfilehash: 1d05e73bc9aa51bf20121790de94b1b6d0e26615
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "89268568"
---
# <a name="generate-a-thumbnail-sprite"></a>Générer un sprite de miniatures

[!INCLUDE [media services api v2 logo](./includes/v2-hr.md)]

Vous pouvez utiliser Media Encoder Standard pour générer un sprite de miniatures, qui est un fichier JPEG contenant plusieurs miniatures de faible résolution rassemblées en une seule (grande) image, avec un fichier VTT. Ce fichier VTT spécifie la plage de temps dans la vidéo d’entrée représentée par chaque miniature, ainsi que la taille et les coordonnées de cette miniature dans le grand fichier JPEG. Les lecteurs vidéo utilisent le fichier VTT et l’image du sprite pour afficher une barre de recherche « visuelle », qui constitue une visionneuse avec un feedback visuel lors de la lecture à vitesse variable vers l’arrière et vers l’avant dans la chronologie des vidéos.

Pour pouvoir utiliser Media Encoder Standard pour générer un sprite de miniatures, le préréglage :

1. Doit utiliser un format d’image miniature JPG
2. Doit spécifier des valeurs de début/étape/plage sous forme d’horodatages ou de valeurs en pour cent (et non pas des nombres de trames) 
    
    1. Il est possible de combiner des horodatages et des valeurs en pour cent.

3. La valeur SpriteColumn doit être spécifiée, et sa valeur doit être supérieure ou égale à 1

    1. Si SpriteColumn est définie sur M >= 1, l’image de sortie est un rectangle avec M colonnes. Si le nombre de miniatures générées via #2 n’est pas un multiple exact de M, la dernière ligne est incomplète et remplie avec des pixels noirs.  

Voici un exemple :

```json
{
    "Version": 1.0,
    "Codecs": [
    {
      "Start": "00:00:01",
      "Type": "JpgImage",
      "Step": "5%",
      "Range": "100%",
      "JpgLayers": [
        {
          "Type": "JpgLayer",
          "Width": "10%",
          "Height": "10%",
          "Quality": 90
        }
      ],
      "SpriteColumn": 10
    }
      ],
      "Outputs": [
        {
          "FileName": "{Basename}_{Index}{Extension}",
          "Format": {
            "Type": "JpgFormat"
          }
        }
   ]
}
```

## <a name="known-issues"></a>Problèmes connus

1.  Il n’est pas possible de générer une image sprite avec une seule ligne d’images (SpriteColumn = 1 produit une image avec une seule colonne).
2.  La segmentation des images sprite en images JPEG de taille modérée n’est pas encore prise en charge. Vous devez donc veiller à limiter le nombre de miniatures et leur taille, afin que le sprite de miniatures en panorama résultant soit de 8 mégapixels ou moins.
3.  Le lecteur multimédia Azure prend en charge les sprites sur les navigateurs Microsoft Edge, Chrome et Firefox. L’analyse des fichiers VTT n’est pas prise en charge dans IE11.

## <a name="next-steps"></a>Étapes suivantes

[Encodage de contenu](media-services-encode-asset.md)
