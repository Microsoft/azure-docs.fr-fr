---
title: Positionnement de la caméra d’analyse spatiale
titleSuffix: Azure Cognitive Services
description: Découvrez comment configurer une caméra en vue d’une analyse spatiale
services: cognitive-services
author: aahill
manager: nitinme
ms.service: cognitive-services
ms.subservice: computer-vision
ms.topic: conceptual
ms.date: 09/11/2020
ms.author: aahi
ms.openlocfilehash: 8e951e6cb18596b19f49bf42179297f656e3fa5f
ms.sourcegitcommit: 32c521a2ef396d121e71ba682e098092ac673b30
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 09/25/2020
ms.locfileid: "91304290"
---
# <a name="camera-placement-guide"></a>Guide de positionnement de la caméra

Cet article fournit des recommandations relatives au positionnement de la caméra en vue d’une analyse spatiale (version préliminaire publique). Il inclut des instructions générales ainsi que des recommandations spécifiques concernant la hauteur, l’angle et la distance entre le point focal et la caméra pour toutes les opérations incluses. 

> [!NOTE]
> Ce guide est conçu pour la caméra Axis M3045-V. Cette caméra utilise une résolution de 1 920 x 1 080, un champ horizontal de 106 degrés, un champ de vision vertical de 59 degrés, et une longueur focale fixe de 2,8 mm. Les principes ci-dessous s’appliquent à toutes les caméras, mais des instructions spécifiques sur la hauteur de la caméra et la distance entre le point focal et la caméra doivent être ajustées pour une utilisation avec d’autres caméras. 

## <a name="general-guidelines"></a>Recommandations générales

Tenez compte des recommandations générales suivantes lorsque vous positionnez des caméras en vue d’une analyse spatiale :

* **Hauteur de l’éclairage.** Placez les caméras sous les systèmes d’éclairage afin ces derniers n’obstruent pas la vue des caméras.
* **Obstructions.** Pour éviter d’obstruer les vues de la caméra, prenez note des éventuels obstacles, par exemple des poteaux, des panneaux, des étagères, des murs et d’autres caméras LP existantes.
* **Rétroéclairage environnemental.** Le rétroéclairage extérieur affecte la qualité de l’image de la caméra. Pour éviter d’éventuelles mauvaises conditions de rétroéclairage, n’orientez pas les caméras vers des fenêtres et des portes en verre externes.
* **Réglementations locales en matière de respect de la vie privée.** Les réglementations locales peuvent limiter le champ de vision des caméras. Assurez-vous que vous comprenez les réglementations locales avant de placer les caméras.
* **Structure du bâtiment.** Les systèmes de chauffage, ventilation et climatisation (HVAC), les gicleurs et le câblage existant peuvent limiter l’installation des caméras.
* **Gestion du câblage.** Veillez à installer un câble Ethernet entre les emplacements prévus pour l’installation des caméras et le commutateur PoE (Power on Internet).

## <a name="height-focal-point-distance-and-angle"></a>Hauteur, distance du point focal et angle

Vous devez prendre en compte trois éléments lorsque vous installez une caméra en vue d’une analyse spatiale :
- Hauteur de la caméra
- Distance entre le point focal et la caméra
- Angle de la caméra par rapport au sol

Il est également important de connaître la direction dans laquelle la majorité des personnes marchent (direction de la marche d’une personne) par rapport au champ de vision de la caméra, si possible. Cette direction est importante car elle affecte les performances du système.

![Image d’une personne qui marche dans une direction](./media/spatial-analysis/person-walking-direction.png)

L’illustration suivante montre la vue en hauteur de la direction de marche d’une personne.

![Vue en hauteur et de plan](./media/spatial-analysis/person-walking-direction-diagram.png)

## <a name="camera-height"></a>Hauteur de la caméra

En règle générale, les caméras doivent être installées à environ 12-14 pieds (3-4 mètres) du sol. Lorsque vous planifiez l’installation de votre caméra dans cette plage, pensez aux éventuels obstacles (étagères, luminaires, panneaux suspendus et écrans, par exemple) susceptibles d’obstruer la vue de la caméra, puis ajustez la hauteur si nécessaire.

## <a name="camera-to-focal-point-distance"></a>Distance entre le point focal et la caméra

La _distance entre le point focal et la caméra_ représente la distance linéaire entre le point focal (ou le centre de l’image de la caméra) et la caméra mesurée au sol.

![Distance entre le point focal et la caméra](./media/spatial-analysis/camera-focal-point.png)

Cette distance se mesure au niveau du sol.

![Méthode pour mesurer la distance entre le point focal et la caméra au niveau du sol](./media/spatial-analysis/camera-focal-point-floor-plane.png)

Vue de haut, la mesure ressemble à ceci :

![Méthode pour mesurer la distance entre le point focal et la caméra, vue d’en haut](./media/spatial-analysis/camera-focal-point-above.png)

Utilisez le tableau ci-dessous pour déterminer la distance de la caméra par rapport au point focal en fonction de hauteurs d’installation spécifiques. Ces distances garantissent un positionnement optimal. Notez que le tableau fournit des conseils pour une installation sous la recommandation « 12'-14'», car certains plafonds peuvent limiter la hauteur.

| Hauteur de la caméra | Distance entre le point focal et la caméra (min/max) |  
| ------------- | ---------------------------------------- |  
| 8'            | 10'-13'                                  |  
| 10'           | 7'-13'                                   |  
| 12'           | 10'-17'                                  |  
| 14'           | 11'-18'                                  |  
| 16'           | 12'-22'                                  |  
| 20'           | 15'-30'                                  |  

L’illustration suivante simule les vues d’une caméra selon la distance la plus proche et la distance la plus éloignée entre le point focal et la caméra.

| La plus proche                                      | La plus éloignée                                      |  
| -------------------------------------------- | --------------------------------------------- |  
| ![Distance la plus proche entre le point focal et la caméra](./media/spatial-analysis/focal-point-closest.png) | ![Distance la plus éloignée entre le point focal et la caméra](./media/spatial-analysis/focal-point-farthest.png) |  

## <a name="camera-angle-mounting-ranges"></a>Plages de montage de l’angle de la caméra

Cette section décrit les plages de montage acceptables pour l’angle de caméra. Ces plages de montage indiquent la plage acceptable pour un positionnement optimal.

### <a name="line-configuration"></a>Configuration de ligne

Le tableau suivant indique les recommandations pour les caméras configurées en mode **cognitiveservices.vision.spatialanalysis-personcrossingline**. 

| Hauteur de la caméra | Distance entre le point focal et la caméra | Angle de montage optimal de la caméra (min/max) |  
| ------------- | ------------------------------ | ------------------------------------------ |  
| 8'            | 9'                             | +/-40°                                     |  
| 10'           | 10'                            | +/-30°                                     |  
| 12'           | 13'                            | +/-20°                                     |  
| 16'           | 18'                            | +/-10°                                     |  
| 20'           | 22'                            | +/-10°                                     |  

L’illustration suivante simule les vues de caméra à l’aide des recommandations d’angle le plus à gauche (-) et le plus à droite (+) en mode **cognitiveservices.vision.spatialanalysis-personcrossingline** pour compter les personnes qui passent une porte.

| Vue la plus à gauche                | Vue la plus à droite                |  
| ---------------------------- | ----------------------------- |  
| ![Angle de caméra le plus à gauche](./media/spatial-analysis/camera-angle-left.png) | ![Angle de caméra le plus à droite](./media/spatial-analysis/camera-angle-right.png) |  

L’illustration suivante montre le positionnement d’une caméra et les angles de montage sous une vue aérienne.

![Vue aérienne](./media/spatial-analysis/camera-angle-top.png)

### <a name="zone-configuration"></a>Configuration de la zone

Nous vous recommandons de placer les caméras à 10 pieds (3 m) ou plus du sol pour garantir une zone de couverture suffisamment grande. 

Lorsque la zone se trouve à côté d’un obstacle comme un mur ou une étagère, installez les caméras à la distance spécifiée depuis la cible dans la plage d’angles de 120 degrés acceptable, comme indiqué dans l’illustration suivante.

![Angle de caméra acceptable](./media/spatial-analysis/camera-angle-acceptable.png)

L’illustration suivante fournit des simulations pour les vues de caméra gauche et droite d’une zone à proximité d’une étagère.

| Vue de gauche        | Vue de droite        |  
| ---------------- | ----------------- |  
| ![Vue de gauche](./media/spatial-analysis/end-cap-left.png) | ![Vue de droite](./media/spatial-analysis/end-cap-right.png) |  

#### <a name="queues"></a>Files d’attente

Les compétences **cognitiveservices.vision.spatialanalysis-personcount**, **cognitiveservices.vision.spatialanalysis-persondistance** et **cognitiveservices.vision.spatialanalysis-personcrossingpolygon** peuvent être utilisées pour surveiller des files d’attente. Pour optimiser la qualité des données d’une file d’attente, il est recommandé de mettre en place des barrières à ceinture rétractables pour réduire l’occlusion des personnes dans la file d’attente et ainsi garantir la cohérence de l’emplacement des files d’attente dans le temps.

![File d’attente avec barrière à ceinture rétractable](./media/spatial-analysis/retractable-belt-queue.png)

Ce type de barrière est préféré aux barrières opaques pour canaliser une file d’attente car il optimise la précision des insights provenant du système.

Il existe deux types de files d’attente : linéaire et à ondulations.

L’illustration suivante montre les recommandations pour des files d’attente linéaires :

![Recommandation de file d’attente linéaire](./media/spatial-analysis/camera-angle-linear-queue.png)

L’illustration suivante fournit des simulations pour les vues de caméra gauche et droite de files d’attente linéaires. Notez que vous pouvez installer la caméra sur le côté opposé de la file d’attente.

| Vue de gauche                          | Vue de droite                          |  
| ---------------------------------- | ----------------------------------- |  
| ![Angle gauche pour une file d’attente linéaire](./media/spatial-analysis/camera-angle-linear-left.png) | ![Angle droit pour une file d’attente linéaire](./media/spatial-analysis/camera-angle-linear-right.png) |  

Pour les files d’attente à ondulations, il est préférable d’éviter de placer la caméra directement dans le sens de la file d’attente, comme indiqué dans l’illustration suivante. Notez que chacun des quatre exemples de positionnement de caméra dans l’illustration offre une vue idéale avec un écart acceptable de +/-15 degrés dans chaque direction.

Les illustrations suivantes simulent la vue d’une caméra placée à des positions optimales pour une file d’attente à ondulations.

| Vue 1        | Vue 2        |  
| ------------- | ------------- |  
| ![Vue 1](./media/spatial-analysis/camera-angle-ideal-location-right.png) | ![Vue 2](./media/spatial-analysis/camera-angle-ideal-location-left.png) |  

| Vue 3 |  Vue 4 |  
| ---- | ----  |
| ![Vue 3](./media/spatial-analysis/camera-angle-ideal-location-back.png) |  ![Vue 4](./media/spatial-analysis/camera-angle-ideal-location-back-left.png) | 

##### <a name="organic-queues"></a>Files d’attente organiques

Les files d’attente organiques se forment de manière organique. Ce style de file d’attente est acceptable si les files d’attente ne dépassent pas 2 à 3 personnes et que la ligne reste dans la définition de la zone. Si la longueur de la file d’attente dépasse généralement à 2 à 3 personnes, nous vous recommandons d’utiliser une barrière à ceinture rétractable pour guider la file d’attente et vous assurer que la ligne reste dans la définition de la zone.

## <a name="next-steps"></a>Étapes suivantes

* [Déployer une application web de comptage de personnes](spatial-analysis-web-app.md)
* [Configurer les opérations d’analyse spatiale](./spatial-analysis-operations.md)
* [Journalisation et résolution des problèmes](spatial-analysis-logging.md)
* [Guide de positionnement de zone et de ligne](spatial-analysis-zone-line-placement.md)
