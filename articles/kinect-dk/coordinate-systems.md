---
title: Systèmes de coordonnées Azure Kinect DK
description: Description des systèmes de coordonnées Azure Kinect DK associés aux capteurs Azure DK
author: tesych
ms.author: tesych
ms.prod: kinect-dk
ms.date: 06/26/2019
ms.topic: conceptual
keywords: kinect, azure, capteur, Kit de développement logiciel (sdk), caméra de profondeur, temps de vol, principes, performances, invalidation
ms.openlocfilehash: 4bb1c3b79862b918870cff786042d9b4c66270d1
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85276452"
---
# <a name="azure-kinect-dk-coordinate-systems"></a>Systèmes de coordonnées Azure Kinect DK

Dans cet article, nous décrivons les conventions utilisées pour les systèmes de coordonnées 2D et 3D.  Il existe des systèmes de coordonnées distincts associés à chaque capteur d’appareil et aux [fonctions d’étalonnage](use-calibration-functions.md) autorisées à convertir des points entre eux. Les [fonctions de transformation](use-image-transformation.md) convertissent des images entières entre systèmes de coordonnées.  

## <a name="2d-coordinate-systems"></a>Systèmes de coordonnées 2D

 Les caméras de profondeur et couleur sont associées à un système de coordonnées 2D indépendant. Une coordonnée [x,y] est représentée en unités de pixels où *x* est compris entre 0 et largeur-1 et *y* entre 0 et hauteur-1. La largeur et la hauteur dépendent du choix du mode dans lequel les caméras de profondeur et couleur sont utilisées. La coordonnée de pixel `[0,0]` correspond au pixel supérieur gauche de l’image. Des coordonnées de pixels peuvent être fractionnelles et représenter des coordonnées de sous-pixel.

Le système de coordonnées 2D est centré sur 0. Autrement dit, la coordonnée de sous-pixel `[0.0, 0.0]` représente le centre, et la coordonnée `[0.5,0.5]` l’angle inférieur droit du pixel, comme illustré ci-dessous.

   ![Système de coordonnées 2D](./media/concepts/concepts-coordinate-systems/coordinate-systems-sdk-2d-system.png)

## <a name="3d-coordinate-systems"></a>Systèmes de coordonnées 3D

Chaque caméra, l’accéléromètre et le gyroscope sont associés à un système d’espace de coordonnées 3D indépendant.

Les points dans les systèmes de coordonnées 3D sont représentés en tant que triplets de coordonnées [X,Y,Z] de métrique avec des unités exprimées en millimètres.

### <a name="depth-and-color-camera"></a>Caméra de profondeur et couleur

L’origine `[0,0,0]` se trouve au point focal de la caméra. Le système de coordonnées est orienté de manière à ce que l’axe X positif pointe vers la droite, l’axe Y positif vers le bas, et l’axe Z positif vers l’avant.

La caméra de profondeur est inclinée de 6 degrés vers le bas de la caméra de couleur, comme illustré ci-dessous. 

La caméra de profondeur utilise deux illuminateurs. L’illuminateur utilisé dans les modes de champ de vision étroit (NFOV, narrow field-of-view) étant aligné avec le boîtier de la caméra de profondeur, l’illuminateur n’est pas incliné. L’illuminateur utilisé dans les modes de champ de vision large (WFOV, wide field-of-view ) est incliné de 1,3 degrés vers le bas par rapport à la caméra de profondeur.

![conventions de coordonnée 3D](./media/concepts/concepts-coordinate-systems/coordinate-systems-camera-features.png)

### <a name="gyroscope-and-accelerometer"></a>Gyroscope et accéléromètre

L’origine `[0,0,0]` du gyroscope est identique à celle de la caméra de profondeur. L’origine de l’accéléromètre coïncide avec son emplacement physique. Les systèmes de coordonnées de l’accéléromètre et du gyroscope sont droitiers. L’axe X positif du système de coordonnées pointe vers l’arrière, l’axe Y positif vers la gauche et l’axe Z positif vers le bas, comme illustré ci-dessous.

![Système de coordonnées d’IMU.](./media/concepts/concepts-coordinate-systems/coordinate-systems-gyroscope.png)

## <a name="next-steps"></a>Étapes suivantes

[Découvrir le Kit de développement logiciel (SDK) de capteur Kinect Azure](about-sensor-sdk.md)