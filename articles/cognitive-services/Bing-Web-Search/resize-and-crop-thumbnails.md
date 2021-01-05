---
title: Redimensionner et rogner les images miniatures - API Recherche Web Bing
titleSuffix: Azure Cognitive Services
description: Certaines réponses des API Recherche Bing incluent des URL vers les images miniatures affichées par Bing, que vous pouvez redimensionner et rogner, et qui peuvent contenir des paramètres de requête.
services: cognitive-services
author: aahill
manager: nitinme
ms.assetid: 05A08B01-89FF-4781-AFE7-08DA92F25047
ms.service: cognitive-services
ms.subservice: bing-web-search
ms.topic: conceptual
ms.date: 07/08/2019
ms.author: aahi
ms.openlocfilehash: a85c5b2333418367742678a529b69c95164eda53
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96350481"
---
# <a name="resize-and-crop-thumbnail-images"></a>Redimensionner et rogner les images miniatures

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020**, toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Accord Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

Certaines réponses des API Recherche Bing incluent des URL vers les images miniatures affichées par Bing, que vous pouvez redimensionner et rogner, et qui peuvent contenir des paramètres de requête. Par exemple :

`https://<host>/th?id=AMMS_92772df988...&w=110&h=73&rs=1&qlt=80&cdv=1&pid=16.1`

Si vous affichez un sous-ensemble de ces miniatures, offrez une option pour afficher les images restantes.

> [!NOTE]
> Vérifiez que le rognage et le redimensionnement des images miniatures fournissent un scénario de recherche et respectent les droits tiers, comme requis par les [conditions d’utilisation et d’affichage](use-display-requirements.md) de l’API Recherche Bing.

## <a name="resize-a-thumbnail"></a>Redimensionner une miniature 

Pour redimensionner une miniature, Bing vous recommande de ne spécifier qu'un seul paramètre de requête `w` (largeur) ou `h` (hauteur) dans l'URL de la miniature. Spécifier uniquement la hauteur ou la largeur permet à Bing de conserver l'aspect original de l'image. Spécifiez la largeur et la hauteur en pixels. 

Par exemple, si la miniature originale est au format 480x620 :

`https://<host>/th?id=JN.5l3yzwy%2f%2fHj59U6XhssIQ&pid=Api&w=480&h=620`

Et que vous voulez diminuer sa taille, réglez le paramètre `w` sur une nouvelle valeur (par exemple `336`), et supprimez le paramètre `h` :

`https://<host>/th?id=JN.5l3yzwy%2f%2fHj59U6XhssIQ&pid=Api&w=336`

Si vous spécifiez uniquement la hauteur ou la largeur d'une miniature, le format d'origine de l'image sera conservé. Si vous spécifiez ces deux paramètres et que le format d'image n'est pas respecté, Bing ajoutera une marge intérieure blanche à la bordure de l'image.

Exemple : si vous redimensionnez une image de 480x359 sur 200x200 sans la rogner, la largeur totale contiendra l’image, mais la hauteur contiendra une marge intérieure blanche de 25 pixels en haut et en bas de l’image. Si l’image avait une taille de 359x480, les bordures gauche et droite contiendraient une marge intérieure blanche. Si vous rognez l’image, aucune marge intérieure blanche n’est ajoutée.  

L’illustration suivante présente la taille d’origine d’une image miniature (480x300).  
  
![Image d’origine en mode Paysage](./media/resize-crop/bing-resize-crop-landscape.png)  
  
L’illustration suivante présente l’image redimensionnée sur 200x200. Les proportions sont conservées, et les bordures supérieure et inférieure sont remplies de blanc (la bordure noire est incluse ici afin de montrer la marge intérieure).  
  
![Image redimensionnée en mode Paysage](./media/resize-crop/bing-resize-crop-landscape-resized.png)  

Si vous spécifiez des dimensions supérieures à la largeur et à la hauteur d’origine de l’image, Bing ajoutera une marge intérieure blanche aux bordures gauche et supérieure.  

## <a name="request-different-thumbnail-sizes"></a>Demander différentes tailles de miniatures

Pour demander une taille de miniature différente, supprimez tous les paramètres de requête de l'URL de la miniature, à l'exception des paramètres `id` et `pid`. Ajoutez ensuite le paramètre `&w` (largeur) ou `&h` (hauteur) de la requête avec la taille d'image souhaitée en pixels, mais pas les deux. Bing conservera les proportions d'origine de l'image. 

Pour augmenter la largeur de l'image spécifiée par l'URL ci-dessus à 165 pixels, vous devez utiliser l'URL suivante :

`https://<host>/th?id=AMMS_92772df988...&w=165&pid=16.1`

Si vous demandez une image plus grande que sa taille originale, Bing ajoute une marge intérieure blanche autour de l'image si nécessaire. Par exemple, si la taille originale de l'image est 474x316 et que vous réglez `&w` sur 500, Bing retournera une image de 500x333. Cette image affichera une marge intérieure blanche de 8,5 pixels aux bordures supérieure et inférieure, et une marge intérieure blanche de 13 pixels aux bordures gauche et droite.

Pour empêcher Bing d'ajouter une marge intérieure blanche si la taille demandée est supérieure à la taille originale de l'image, définissez le paramètre `&p` de la requête sur 0. Par exemple, si vous incluez le paramètre `&p=0` dans l'URL ci-dessus, Bing retournera une image 474x316 au lieu d'une image 500x333 :

`https://<host>/th?id=AMMS_92772df988...&w=500&p=0&pid=16.1`

Si vous spécifiez les paramètres de requête `&w` et `&h`, Bing conservera les proportions de l'image et ajoutera une marge intérieure blanche si nécessaire. Par exemple, si la taille originale de l'image est 474x316 et que vous réglez les paramètres de largeur et de hauteur sur 200x200 (`&w=200&h=200`), Bing retourne une image contenant une marge intérieure blanche de 33 pixels en haut et en bas. Si vous incluez le paramètre de requête `&p`, Bing retourne une image 200x134.

## <a name="crop-a-thumbnail"></a>Rogner une miniature 

Pour rogner une image, incluez le paramètre de requête `c` (rogner). Vous pouvez utiliser les valeurs suivantes :
  
- `4` &mdash; Blind Ratio  
- `7` &mdash; Smart Ratio  

### <a name="smart-ratio-cropping"></a>Rognage Smart Ratio

Si vous demandez un rognage Smart Ratio (Ratio intelligent) en définissant le paramètre `c` sur `7`, Bing rognera une image partir du centre de la région d’intérêt de l’image vers l’extérieur tout en conservant les proportions de l’image. La région d’intérêt est la zone de l’image qui, d’après Bing, contient les parties les plus importantes. L’illustration suivante présente un exemple de région d’intérêt :  
  
![Région d’intérêt](./media/resize-crop/bing-resize-crop-regionofinterest.png)

Si vous redimensionnez une image et demandez un rognage Smart Ratio (Ratio intelligent), Bing réduire l’image jusqu’à la taille demandée, tout en conservant ses proportions. Bing rognera ensuite l’image en fonction des nouvelles dimensions. Exemple : si la largeur redimensionnée est inférieure ou égale à la hauteur, Bing rognera l’image à gauche et à droite du centre de la région d’intérêt. Dans le cas contraire, Bing la rognera en haut et en bas du centre de la région d’intérêt.  
  
 
L’illustration suivante présente l’image réduite à une taille de 200x200 à l’aide du rognage Smart Ratio (Ratio intelligent). Comme Bing mesure l’image à partir du coin supérieur gauche, la partie inférieure de cette image est rognée. 
  
![Image en mode Paysage rognée à 200x200](./media/resize-crop/bing-resize-crop-landscape200x200c7.png) 
  
L’illustration suivante présente l’image réduite à une taille de 200x100 à l’aide du rognage Smart Ratio (Ratio intelligent). Comme Bing mesure l’image à partir du coin supérieur gauche, la partie inférieure de cette image est rognée. 
   
![Image en mode Paysage rognée à 200x100](./media/resize-crop/bing-resize-crop-landscape200x100c7.png)
  
L’illustration suivante présente l’image réduite à une taille de 100x200 à l’aide du rognage Smart Ratio (Ratio intelligent). Comme Bing mesure l’image à partir du centre, les parties gauche et droite de cette image sont rognées.
  
![Image en mode Paysage rognée à 100x200](./media/resize-crop/bing-resize-crop-landscape100x200c7.png) 

Si Bing ne peut pas déterminer la région d’intérêt de l’image, le service utilisera le rognage Blind Ratio (Ratio aveugle).  

### <a name="blind-ratio-cropping"></a>Rognage Blind Ratio

Si vous demandez le rognage Blind Ratio (Ratio aveugle) en définissant le paramètre `c` sur `4`, Bing utilise les règles ci-dessous pour rogner l’image.  
  
- Avec `(Original Image Width / Original Image Height) < (Requested Image Width / Requested Image Height)`, l'image est mesurée à partir du coin supérieur gauche puis rognée en bas.  
- Avec `(Original Image Width / Original Image Height) > (Requested Image Width / Requested Image Height)`, l'image est mesurée à partir du centre puis rognée à gauche et à droite.  

L’illustration suivante présente une image en mode Portrait de 225x300.  
  
![Image d’origine de tournesol](./media/resize-crop/bing-resize-crop-sunflower.png)
  
L’illustration suivante présente l’image réduite à une taille de 200x200 à l’aide du rognage Blind Ratio (Ratio aveugle). L’image est mesurée à partir du coin supérieur gauche, ce qui entraîne le rognage de sa partie inférieure.  
  
![Image de tournesol rognée à 200x200](./media/resize-crop/bing-resize-crop-sunflower200x200c4.png)
  
L’illustration suivante présente l’image réduite à une taille de 200x100 à l’aide du rognage Blind Ratio (Ratio aveugle). L’image est mesurée à partir du coin supérieur gauche, ce qui entraîne le rognage de sa partie inférieure.  
  
![Image de tournesol rognée à 200x100](./media/resize-crop/bing-resize-crop-sunflower200x100c4.png)
  
L’illustration suivante présente l’image réduite à une taille de 100x200 à l’aide du rognage Blind Ratio (Ratio aveugle). L’image est mesurée à partir du centre, ce qui entraîne le rognage de ses parties gauche et droite.  
  
![Image de tournesol rognée à 100x200](./media/resize-crop/bing-resize-crop-sunflower100x200c4.png)

## <a name="next-steps"></a>Étapes suivantes

* [Présentation des API Recherche Bing](bing-api-comparison.md)
* [Conditions d’utilisation et d’affichage de l’API Recherche Bing](use-display-requirements.md)