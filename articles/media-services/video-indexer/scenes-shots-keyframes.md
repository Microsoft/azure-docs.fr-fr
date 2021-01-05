---
title: Scènes, plans et images clés Video Indexer
titleSuffix: Azure Media Services
description: Cette rubrique offre une vue d'ensemble des scènes, captures et images clés Video Indexer.
services: media-services
author: Juliako
manager: femila
ms.service: media-services
ms.subservice: video-indexer
ms.topic: article
ms.date: 07/05/2019
ms.author: juliako
ms.openlocfilehash: 248799d70e0741efcaea1714c12f4d92a42cef25
ms.sourcegitcommit: 4f4a2b16ff3a76e5d39e3fcf295bca19cff43540
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/30/2020
ms.locfileid: "93041906"
---
# <a name="scenes-shots-and-keyframes"></a>Scènes, captures et images clés

Video Indexer prend en charge la segmentation de vidéos en unités temporelles basées sur des propriétés sémantiques et structurelles. Cette fonctionnalité permet aux clients de facilement parcourir, gérer et modifier leur contenu vidéo en fonction des niveaux de détail. Par exemple, en fonction des scènes, des captures et des images clés, décrites dans cette rubrique.   

![Scènes, captures et images clés](./media/scenes-shots-keyframes/scenes-shots-keyframes.png)
 
## <a name="scene-detection"></a>Détection de scène  
 
Video Indexer détermine quand une scène change dans la vidéo via des signaux visuels. Une scène représente un événement unique et est composée d’une série de captures consécutives, qui sont sémantiquement liées. Une miniature de la scène constitue la première image clé de sa capture sous-jacente. Video Indexer segmente une vidéo en scènes en fonction de la cohérence des couleurs sur des captures consécutives et récupère le début et la fin de chaque scène. La détection de scène est considérée comme étant une tâche ardue, car elle implique de quantifier des aspects sémantiques d’une vidéo.

> [!NOTE]
> S’applique aux vidéos qui contiennent au moins 3 scènes.

## <a name="shot-detection"></a>Détection de plan

Video Indexer détermine quand une capture change dans la vidéo selon des signaux visuels, en effectuant le suivi des transitions brusques et progressives du jeu de couleurs des images adjacentes. Les métadonnées de la capture incluent un temps de début et de fin, ainsi que la liste des images clés inclues dans cette capture. Les captures sont des images consécutives prises de la même caméra et en même temps.

## <a name="keyframe-detection"></a>Détection d’une image clé

Video Indexer sélectionne les images représentant au mieux chaque capture. Les images clés sont des images représentatives sélectionnées à partir de la vidéo complète, en fonction des propriétés esthétiques (par exemple, contraste et stabilité). Video Indexer récupère une liste d’ID d’images clés faisant partie des métadonnées de la capture, en fonction des clients qui peuvent extraire l’image clé en tant qu’image haute résolution.  

### <a name="extracting-keyframes"></a>Extraction d’images clés

Pour extraire des images clés haute résolution pour votre vidéo, vous devez d’abord charger et indexer la vidéo.

![Images clés](./media/scenes-shots-keyframes/extracting-keyframes.png)

#### <a name="with-the-video-indexer-website"></a>Avec le site web Video Indexer

Pour extraire des images clés à l’aide du site web Video Indexer, chargez et indexez votre vidéo. Une fois la tâche d’indexation terminée, cliquez sur le bouton **Télécharger** et sélectionnez **Artefacts (ZIP)** . Cela permet de télécharger le dossier d’artefacts sur votre ordinateur. 

![Capture d’écran montrant la liste déroulante « Télécharger » avec l’option « Artefacts » sélectionnée.](./media/scenes-shots-keyframes/extracting-keyframes2.png)
 
Décompressez et ouvrez le dossier. Dans le dossier *_KeyframeThumbnail* , vous trouverez toutes les images clés qui ont été extraites de votre vidéo. 

#### <a name="with-the-video-indexer-api"></a>Avec l’API Video Indexer

Pour récupérer des images clés à l’aide de l’API Video Indexer, chargez et indexez votre vidéo à l’aide de l’appel [Charger la vidéo](https://api-portal.videoindexer.ai/docs/services/Operations/operations/Upload-Video?). Une fois la tâche d’indexation terminée, appelez [Obtenir l’index vidéo](https://api-portal.videoindexer.ai/docs/services/Operations/operations/Get-Video-Index?). Vous obtiendrez ainsi toutes les informations que Video Indexer a extraites de votre contenu dans un fichier JSON.  

Vous obtiendrez une liste d’ID d’images clés dans les métadonnées de chaque plan. 

```json
"shots":[  
    {  
      "id":0,
      "keyFrames":[  
          {  
            "id":0,
            "instances":[  
                {  
                  "thumbnailId":"00000000-0000-0000-0000-000000000000",
                  "start":"0:00:00.209",
                  "end":"0:00:00.251",
                  "duration":"0:00:00.042"
                }
            ]
          },
          {  
            "id":1,
            "instances":[  
                {  
                  "thumbnailId":"00000000-0000-0000-0000-000000000000",
                  "start":"0:00:04.755",
                  "end":"0:00:04.797",
                  "duration":"0:00:00.042"
                }
            ]
          }
      ],
      "instances":[  
          {  
            "start":"0:00:00",
            "end":"0:00:06.34",
            "duration":"0:00:06.34"
          }
      ]
    },

]
```

Vous devrez ensuite exécuter chacun de ces ID d’images clés sur l’appel [Obtenir des miniatures](https://api-portal.videoindexer.ai/docs/services/Operations/operations/Get-Video-Thumbnail?). Chacune des images clés sera alors téléchargée sur votre ordinateur. 

## <a name="editorial-shot-type-detection"></a>Détection du type de plan éditorial

Les images clés sont associées à des captures dans la sortie JSON. 

Le type de plan associé à un plan individuel dans les insights JSON représente son type éditorial. Vous pouvez trouver ces caractéristiques de type de plan utiles lors de la modification de vidéos dans des clips ou bandes-annonces, ou lors de la recherche d’un style spécifique d’image clé à des fins artistiques. Les différents types sont déterminés en fonction de l’analyse de la première image clé de chaque plan. Les plans sont identifiés par l’échelle, la taille et l’emplacement des visages apparaissant dans leur première image clé. 

La taille et l’échelle du plan sont déterminées en fonction de la distance entre l’appareil photo et les visages apparaissant dans l’image. À l’aide de ces propriétés, Video Indexer détecte les types de plans suivants :

* Large : montre le corps tout entier d’une personne.
* Moyen : montre le haut du corps et le visage d’une personne.
* Gros plan : montre principalement le visage d’une personne.
* Très gros plan : montre le visage d’une personne qui remplit l’écran. 

Les types de plans peuvent également être déterminés par l’emplacement des personnages par rapport au centre de l’image. Cette propriété définit les types de plans suivants dans Video Indexer :

* Visage à gauche : une personne apparaît sur la gauche de l’image.
* Visage au centre : une personne apparaît dans la partie centrale de l’image.
* Visage à droite : une personne apparaît sur la droite de l’image.
* Extérieur : une personne apparaît dans un décor extérieur.
* Intérieur : une personne apparaît dans un décor intérieur.

Caractéristiques supplémentaires :

* Deux plans : montre les visages de taille moyenne de deux personnes.
* Plusieurs visages : plus de deux personnes.


## <a name="next-steps"></a>Étapes suivantes

[Examiner la sortie de Video Indexer générée par l’API](video-indexer-output-json-v2.md#scenes)
