---
title: Caméra de profondeur d’Azure Kinect DK
description: Découvrez les principes de fonctionnement et les fonctionnalités clés de la caméra de profondeur dans votre Azure Kinect DK.
author: tesych
ms.author: tesych
ms.prod: kinect-dk
ms.date: 06/26/2019
ms.topic: conceptual
keywords: kinect, azure, capteur, Kit de développement logiciel (sdk), caméra de profondeur, temps de vol, principes, performances, invalidation
ms.openlocfilehash: 22f04b983ed7c6a2ab19a5c1c709621655ee31c0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85276487"
---
# <a name="azure-kinect-dk-depth-camera"></a>Caméra de profondeur d’Azure Kinect DK

Cette page explique comment utiliser la caméra de profondeur de votre Azure Kinect DK. La caméra de profondeur est la seconde des deux caméras. Comme décrit dans les sections précédentes, l’autre caméra est la caméra RVB (couleur).  

## <a name="operating-principles"></a>Principes de fonctionnement

La caméra de profondeur d’Azure Kinect DK implémente le principe de temps de vol (ToF, Time-of-Flight) d’AMCW (Amplitude Modulated Continuous Wave). Elle convertit l’illumination modulée de la scène dans le spectre proche de l’IR (NIR). Elle enregistre ensuite une mesure indirecte du temps que la lumière prend pour parcourir le trajet aller-retour de la caméra à la scène.

Ces mesures sont traitées pour générer une carte de profondeur. Une carte de profondeur est un ensemble de valeurs de coordonnée Z pour chaque pixel de l’image, mesurée en millimètres.

À côté de la carte de profondeur, nous obtenons également une lecture IR dite propre. La valeur des pixels dans la lecture IR propre est proportionnelle à la quantité de lumière renvoyée par la scène. L’image ressemble à une image IR normale. La figure ci-dessous montre un exemple de carte de profondeur (à gauche) et l’image IR propre correspondante (à droite).

![Profondeur et IR côte à côte](./media/concepts/depth-camera-depth-ir.png)

## <a name="key-features"></a>Fonctionnalités clés

Les caractéristiques techniques de la caméra de profondeur sont les suivantes :

- Un processeur d’image ToF de 1 mégapixel avec une technologie de pixel avancée permettant des fréquences de modulation plus et une précision de profondeur élevées.
- Deux diodes laser NIR permettant d’obtenir des modes de profondeur de champ de vision (FoV) proches et larges.
- Le plus petit pixel ToF au monde, de 3,5 μm sur 3,5 μm.
- Une sélection automatique de gain par pixel activant une vaste plage dynamique permettant une capture propre d’objets proches et distants.
- Un obturateur global permettant d’améliorer les performances à la lumière du jour.
- Une méthode de calcul de profondeur en plusieurs phases qui permet d’obtenir une précision stable, même en présence de variations de processeur, de laser et d’alimentation.
- Un faible taux d’erreurs systématiques et aléatoires.

![Module de profondeur](./media/concepts/depth-camera-depth-module.jpg)

La caméra de profondeur transmet des images IR modulées brutes au PC hôte. Sur le PC, le logiciel du moteur de profondeur accéléré par GPU convertit le signal brut en cartes de profondeur. La caméra de profondeur prend en charge plusieurs modes. Les modes **Champ de vision (FoV) étroit** sont idéaux pour les scènes faiblement étendues dans les dimensions X et Y, mais plus étendues dans la dimension Z. Si la scène comprend des étendues X et Y importantes, mais des plages Z plus petites, les modes **FoV large** sont plus adaptés.

La caméra de profondeur prend en charge les **modes de compartimentage 2 x 2** pour étendre la plage Z par rapport aux **modes sans compartimentage** correspondants. Le compartimentage s’effectue au détriment de la résolution de l’image. Tous les modes peuvent être exécutés jusqu’à une fréquence de 30 images par seconde (i/s), à l’exception du mode 1 mégapixel (MP) qui s’exécute à une fréquence d’images maximale de 15 i/s. La caméra de profondeur offre également un **mode IR passif**. Dans ce mode, les illuminateurs de la caméra ne sont pas actifs et seul l’éclairage ambiant est pris en compte.

## <a name="camera-performance"></a>Performances de la caméra

Les performances de la caméra sont mesurées en tant qu’erreurs systématiques et aléatoires.

### <a name="systematic-error"></a>Erreur systématique

Une erreur systématique est définie comme la différence entre la profondeur mesurée après la suppression du bruit et la profondeur correcte (réalité du terrain). Nous calculons la moyenne temporelle sur de nombreuses images d’une scène statique pour éliminer autant que possible le bruit de profondeur. Plus précisément, l’erreur systématique est définie comme suit :

![Erreur systématique de profondeur](./media/concepts/depth-camera-systematic-error.png)

Où *d<sub>t</sub>* indique la profondeur de mesure au moment *t*, *N* le nombre d’images utilisées dans la procédure de calcul de moyenne, et *d<sub>gt</sub>* la profondeur correspondant à la réalité du terrain.

La spécification d’erreur systématique de la caméra de profondeur exclut l’interférence multichemin (MPI). Une interférence multichemin se produit quand un pixel de capteur intègre la lumière reflétée par plusieurs objets. Elle est en partie atténuée dans notre caméra de profondeur à l’aide de fréquences de modulation supérieures, ainsi que de l’invalidation de profondeur que nous présenterons plus tard.

### <a name="random-error"></a>Erreur aléatoire

Supposons que nous prenons 100 images du même objet sans déplacer la caméra. La profondeur de l’objet sera légèrement différente dans chacune des images. Cette différence est due au bruit de grenaille. Le bruit de grenaille est dû au fait que le nombre de photons qui atteignent le capteur varie dans une proportion aléatoire au fil du temps. Nous définissons cette erreur aléatoire sur une scène statique comme l’écart type de profondeur dans le temps calculé comme suit :

![Erreur aléatoire de profondeur](./media/concepts/depth-camera-random-error.png)

Où *N* indique le nombre de mesures de profondeur, *d<sub>t</sub>* la mesure de profondeur au moment *t*, et *d* la valeur moyenne calculée sur toutes les mesures de profondeur *d<sub>t</sub>* .

## <a name="invalidation"></a>Invalidation

Dans certains cas, la caméra de profondeur ne peut pas fournir de valeurs correctes pour certains pixels. Les pixels de profondeur sont alors invalidés. Les pixels non valides sont indiqués par une valeur de profondeur égale à 0. Les raisons pour lesquelles le moteur de profondeur ne peut pas produire des valeurs correctes sont les suivantes :

- Hors du masque d’illumination IR actif
- Signal IR saturé
- Signal IR faible
- Valeur hors norme de filtrer
- Interférence multichemin

### <a name="illumination-mask"></a>Masque d’illumination

Des pixels sont invalidés quand ils sont hors du masque d’illumination IR actif. Nous vous déconseillons d’utiliser le signal de tels pixels pour calculer la profondeur. La figure ci-dessous illustre l’exemple d’invalidation par masque d’illumination. Les pixels invalidés sont ceux de couleur noire situés hors du cercle dans les modes FoV large (à gauche), et de l’hexagone dans les FoV étroit (à droite).

![Invalidation hors du masque d’illumination](./media/concepts/depth-camera-invalidation-illumination-mask.png)

### <a name="signal-strength"></a>Force du signal

Des pixels sont invalidés quand ils contiennent un signal IR saturé. Quand les pixels sont saturés, les informations de phase sont perdues. L’image ci-dessous montre l’exemple d’invalidation d’un signal IR saturé. Consultez les flèches pointant vers les exemples de pixels dans les images de profondeur et IR.

![Invalidation pour saturation](./media/concepts/depth-camera-invalidation-saturation.png)

Une invalidation peut également se produire lorsque le signal IR n’est pas suffisamment fort pour générer une profondeur. La figure ci-dessous illustre l’exemple d’invalidation par un signal IR faible. Consultez les flèches pointant vers des exemples de pixels dans les images de profondeur et IR.

![Invalidation pour signal faible](./media/concepts/depth-camera-invalidation-low-signal.png)

### <a name="ambiguous-depth"></a>Profondeur ambiguë

Des pixels peuvent également être invalidés s’ils ont reçu des signaux de plusieurs objets dans la scène. Un cas courant où ce type d’invalidation peut être constaté est celui des coins.  En raison de la géométrie de la scène, la lumière IR de la caméra se réfléchit d’un mur sur l’autre. Cette lumière réfléchie entraîne une ambiguïté dans la profondeur mesurée du pixel. Les filtres de l’algorithme de profondeur détectent ces signaux ambigus et invalident les pixels.

Les figures ci-dessous présentent des exemples d’invalidation par détection multichemin. Vous pouvez également voir comment la surface d’exposition invalidée dans une vue de caméra (ligne supérieure) peut réapparaître dans une autre vue caméra (ligne inférieure). Cette image montre que les surfaces invalidées dans une perspective peuvent être visibles dans une autre.

![Invalidation pour interférence multichemin – Coin](./media/concepts/depth-camera-invalidation-multipath.png)

Un autre cas courant d’invalidation multichemin est quand des pixels qui contiennent un signal mixte pour le premier plan et l’arrière-plan (par exemple autour des bords d’objets). En cas de mouvement rapide, vous pouvez constater que davantage de pixels sont invalidés autour des bords. Les pixels supplémentaires sont invalidés en raison de l’intervalle d’exposition de la capture de profondeur brute.

![Invalidation pour interférence multichemin – Bords](./media/concepts/depth-camera-invalidation-edge.png)

## <a name="next-steps"></a>Étapes suivantes

[Systèmes de coordonnées](coordinate-systems.md)
