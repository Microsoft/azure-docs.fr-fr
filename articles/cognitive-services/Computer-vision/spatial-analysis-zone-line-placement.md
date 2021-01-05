---
title: Positionnement de la ligne et de la zone d’analyse spatiale
titleSuffix: Azure Cognitive Services
description: Découvrez comment configurer des zones et des lignes avec l’analyse spatiale
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: conceptual
ms.date: 09/01/2020
ms.author: aahi
ms.openlocfilehash: 7e2a64c14d7d7a1d20b64b746969aca1e60ab218
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "90930241"
---
# <a name="zone-and-line-placement-guide"></a>Guide de positionnement de zone et de ligne

Cet article fournit des instructions sur la façon de définir des zones et des lignes pour les opérations d’analyse spatiale, afin d’obtenir une analyse précise des mouvements de populations dans un espace. Ceci concerne toutes les opérations. 

Les zones et les lignes sont définies à l’aide du paramètre JSON SPACEANALYSIS_CONFIG. Pour plus d’informations, consultez l’article [Opérations d’analyse spatiale](spatial-analysis-operations.md).

## <a name="guidelines-for-drawing-zones"></a>Instructions pour le dessin de zones

N’oubliez pas que chaque espace est différent ; vous devez mettre à jour la position ou la taille en fonction de vos besoins.

Si vous souhaitez afficher une section spécifique de la vue de votre caméra, créez la zone la plus grande possible, qui couvre la zone de plancher spécifique qui vous intéresse, mais n’inclut pas d’autres zones qui ne vous intéressent pas. Cela augmente la précision des données collectées et empêche les faux positifs des zones que vous ne souhaitez pas suivre. Soyez prudent lorsque vous placez les coins de votre Polygone et assurez-vous qu’ils ne se trouvent pas à l’extérieur de la zone que vous souhaitez suivre.  

### <a name="example-of-a-well-shaped-zone"></a>Exemple de zone correctement mise en forme

La zone doit être suffisamment grande pour accueillir trois personnes sur chaque bord et se concentrer sur le domaine qui vous intéresse. L’analyse spatiale identifie les personnes dont les pieds sont placés dans la zone. Par conséquent, lorsque vous dessinez des zones sur l’image 2D, imaginez la zone comme un tapis sur le plancher.

![Zone correctement mise en forme](./media/spatial-analysis/zone-good-example.png)

### <a name="examples-of-zones-that-arent-well-shaped"></a>Exemples de zones qui ne sont pas correctement mises en forme

Les exemples suivants illustrent des zones mal formées. Dans ces exemples, la zone d’intérêt est l’espace situé devant l’affichage *It's Game Time*.

**La zone n’est pas sur le plancher.**

![Zone mal mise en forme](./media/spatial-analysis/zone-not-on-floor.png) 

**La zone est trop petite.**

![la zone est trop petite](./media/spatial-analysis/zone-too-small.png)

**La zone ne capture pas complètement la zone autour de l’affichage.**

![la zone ne capture pas complètement la zone autour de l’extrémité de fin](./media/spatial-analysis/zone-bad-capture.png)

**La zone est trop proche du bord de l’image de la caméra photo et ne capture pas l’affichage à droite.**

![la zone est trop proche du bord de l’image de la caméra photo et ne capture pas l’affichage à droite](./media/spatial-analysis/zone-edge.png)

**La zone est partiellement bloquée par l’étagère, donc les personnes et le plancher ne sont pas entièrement visibles.**

![la zone est partiellement bloquée, donc les personnes ne sont pas entièrement visibles](./media/spatial-analysis/zone-partially-blocked.png)

### <a name="example-of-a-well-shaped-line"></a>Exemple de ligne correctement mise en forme

La ligne doit être suffisamment longue pour accueillir la totalité de l’entrée. L’analyse spatiale identifiera les personnes dont les pieds franchissent la ligne. Par conséquent, lorsque vous dessinez des lignes sur l’image 2D, imaginez que vous les dessinez comme si elles se trouvaient sur le plancher. 

Si possible, tracez une ligne plus large que l’entrée réelle. Si cela ne provoque pas d’intersections supplémentaires (comme sur l’image ci-dessous lorsque la ligne est sur un mur), vous pouvez la prolonger.

![Ligne correctement mise en forme](./media/spatial-analysis/zone-line-good-example.png)

### <a name="examples-of-lines-that-arent-well-shaped"></a>Exemples de lignes qui ne sont pas correctement mises en forme

Les exemples suivants illustrent des lignes mal définies.

**La ligne ne couvre pas l’intégralité du chemin d’entrée sur le plancher.**

![La ligne ne couvre pas l’intégralité du chemin d’entrée sur le plancher](./media/spatial-analysis/zone-line-bad-coverage.png)

**La ligne est trop haute et ne couvre pas l’intégralité de la porte.**

![La ligne est trop haute et ne couvre pas l’intégralité de la porte](./media/spatial-analysis/zone-line-too-high.png)

## <a name="next-steps"></a>Étapes suivantes

* [Déployer une application web de comptage de personnes](spatial-analysis-web-app.md)
* [Configurer les opérations d’analyse spatiale](./spatial-analysis-operations.md)
* [Journalisation et résolution des problèmes](spatial-analysis-logging.md)
* [Guide de positionnement de la caméra](spatial-analysis-camera-placement.md)
