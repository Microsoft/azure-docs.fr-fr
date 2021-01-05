---
title: Comment utiliser des marqueurs d’ornement pour mettre en surbrillance du texte - API Recherche Web Bing
titleSuffix: Azure Cognitive Services
description: Découvrez comment utiliser des ornements de texte et une mise en surbrillance dans vos résultats de recherche à l’aide de l’API Recherche Web Bing.
services: cognitive-services
author: swhite-msft
manager: nitinme
ms.assetid: 5365B568-EA55-4D97-8FBE-0AF60158D4D5
ms.service: cognitive-services
ms.subservice: bing-web-search
ms.topic: conceptual
ms.date: 07/30/2019
ms.author: scottwhi
ms.openlocfilehash: 32c3279cacddf10e77e8d245ba525ab766efd6fd
ms.sourcegitcommit: 9eda79ea41c60d58a4ceab63d424d6866b38b82d
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 11/30/2020
ms.locfileid: "96351876"
---
# <a name="using-decoration-markers-to-highlight-text"></a>Utilisation de marqueurs d’ornement pour mettre en surbrillance un texte

> [!WARNING]
> Les API Recherche Bing passent de Cognitive Services aux services de recherche Bing. À compter du **30 octobre 2020**, toutes les nouvelles instances de Recherche Bing doivent être provisionnées en suivant le processus documenté [ici](/bing/search-apis/bing-web-search/create-bing-search-service-resource).
> Les API Recherche Bing provisionnées à l’aide de Cognitive Services seront prises en charge les trois prochaines années ou jusqu’à la fin de votre Accord Entreprise, selon la première éventualité.
> Pour obtenir des instructions de migration, consultez [Services de recherche Bing](/bing/search-apis/bing-web-search/create-bing-search-service-resource).

Bing prend en charge la mise en surbrillance des correspondances qui marque les termes de requête (ou d’autres termes que Bing estime pertinents) dans les chaînes d’affichage de certaines réponses. Par exemple, les champs `name`, `displayUrl` et `snippet` des résultats d’une page web peuvent contenir des termes de requête marqués. 

Par défaut, Bing n’inclut pas les marqueurs de mise en surbrillance dans les chaînes d’affichage. Pour activer les marqueurs, ajoutez le paramètre de requête `textDecorations` à votre requête et définissez-le sur `true`.

## <a name="hit-highlighting-example"></a>Exemple de mise en surbrillance des correspondances

L’exemple suivant montre les résultats web de `Sailing Dinghy`. Bing marque le début et la fin du terme de requête à l’aide des caractères Unicode E000 et E001.
  
![Mise en surbrillance des correspondances](./media/cognitive-services-bing-web-api/bing-hit-highlighting.png) 

Avant d’afficher le résultat dans votre interface utilisateur, remplacez les caractères Unicode par des caractères adaptés à votre format d’affichage.

## <a name="marker-formatting"></a>Mise en forme des marqueurs

Bing vous offre la possibilité d’utiliser des caractères Unicode ou des balises HTML en tant que marqueurs. Pour spécifier les marqueurs à utiliser, ajoutez le paramètre de requête [textFormat](/rest/api/cognitiveservices-bingsearch/bing-web-api-v7-reference#textformat) : 

| Valeur             | Marqueur                       |
|-------------------|------------------------------|
| `textFormat=Raw`  | Caractères Unicode (par défaut) |
| `textFormat=HTML` | Caractères HTML              |

## <a name="additional-text-decorations"></a>Autres ornements de texte

Bing peut retourner plusieurs ornements de texte différents. Par exemple, le champ `expression` d’une réponse `Computation` peut contenir des marqueurs d’indice pour le terme de requête `log(2)`.

![marqueurs de calcul](./media/cognitive-services-bing-web-api/bing-markers-computation.png) 

Si la requête n’a pas spécifié d’ornements, le champ `expression` contient `log10(2)`. 

Si `textDecorations` est `true`, Bing peut inclure les marqueurs suivants dans les chaînes d’affichage des réponses. S’il n’existe pas de balise HTML équivalente, la cellule du tableau sera vide.

|Unicode|HTML|Description
|-|-|-
|U+E000|\<b>|Marque le début du terme de requête (mise en surbrillance des correspondances)
|U+E001|\</b>|Marque la fin du terme de requête
|U+E002|\<i>|Marque le début du contenu en italique 
|U+E003|\</i>|Marque la fin du contenu en italique
|U+E004|\<br/>|Marque un saut de ligne
|U+E005||Marque le début d’un numéro de téléphone
|U+E006||Marque la fin d’un numéro de téléphone
|U+E007||Marque le début d’une adresse
|U+E008||Marque la fin d’une adresse
|U+E009|\&nbsp;|Marque une espace insécable
|U+E00C|\<strong>|Marque le début du contenu en gras
|U+E00D|\</strong>|Marque la fin du contenu en gras
|U+E00E||Marque le début du contenu dont l’arrière-plan doit être plus clair que son arrière-plan qui l’entoure
|U+E00F||Marque la fin du contenu dont l’arrière-plan doit être plus clair que son arrière-plan qui l’entoure
|U+E010||Marque le début du contenu dont l’arrière-plan doit être plus foncé que son arrière-plan qui l’entoure
|U+E011||Marque la fin du contenu dont l’arrière-plan doit être plus foncé que son arrière-plan qui l’entoure
|U+E012|\<del>|Marque le début du contenu qui doit être barré
|U+E013|\</del>|Marque la fin du contenu qui doit être barré
|U+E016|\<sub>|Marque le début du contenu en indice
|U+E017|\</sub>|Marque la fin du contenu en indice
|U+E018|\<sup>|Marque le début du contenu en exposant
|U+E019|\</sup>|Marque la fin du contenu en exposant

## <a name="next-steps"></a>Étapes suivantes

* [Qu’est-ce que l’API Recherche Web Bing ?](overview.md) 
* [Redimensionner et rogner les miniatures](resize-and-crop-thumbnails.md)