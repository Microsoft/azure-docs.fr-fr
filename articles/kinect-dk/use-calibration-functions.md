---
title: Utiliser les fonctions d’étalonnage d’Azure Kinect
description: Découvrez comment utiliser les fonctions d’étalonnage pour Azure Kinect DK.
author: cdedmonds
ms.author: cedmonds
ms.prod: kinect-dk
ms.date: 06/26/2019
ms.topic: conceptual
keywords: kinect, azure, capteur, Kit de développement logiciel (sdk), étalonnage, fonctions, caméra, intrinsèque, extrinsèque, projection, déprojection, transformation, rvb-d, nuage de points
ms.openlocfilehash: a501765222154bde4ed35e878a9d18042893fca2
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85276476"
---
# <a name="use-azure-kinect-calibration-functions"></a>Utiliser les fonctions d’étalonnage d’Azure Kinect

Les fonctions d’étalonnage permettent de transformer des points entre les systèmes de coordonnées de chaque capteur sur l’appareil Azure Kinect. Les applications nécessitant une conversion d’images entières peuvent tirer parti des opérations accélérées disponibles dans les [fonctions de transformation](use-image-transformation.md).

## <a name="retrieve-calibration-data"></a>Récupérer les données d’étalonnage

Il est nécessaire de récupérer l’étalonnage de l’appareil pour effectuer des transformations du système de coordonnées. Les données d’étalonnage sont stockées dans le type de données [k4a_calibration_t](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__calibration__t.html#details). Elles sont obtenue de l’appareil via la fonction [k4a_device_get_calibration()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga4e43940d8d8db48da266c7a7842c8d78.html#ga4e43940d8d8db48da266c7a7842c8d78). Les données d’étalonnage sont spécifiques non seulement de chaque appareil, mais aussi du mode de fonctionnement des caméras. Par conséquent, la fonction [k4a_device_get_calibration()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga4e43940d8d8db48da266c7a7842c8d78.html#ga4e43940d8d8db48da266c7a7842c8d78) requiert les paramètres `depth_mode` et `color_resolution` en entrée.

### <a name="opencv-compatibility"></a>Compatibilité OpenCV

Les paramètres d’étalonnage sont compatibles avec la bibliothèque graphique [OpenCV](https://opencv.org/). Pour plus d’informations sur les paramètres d’étalonnage de la caméra, consultez aussi la [documentation sur la bibliothèque graphique OpenCV](https://docs.opencv.org/3.2.0/d9/d0c/group__calib3d.html#gga7041b2a9c8f9f8ee93a2796981bc5546a204766e24f2e413e7a7c9f8b9e93f16c). Consultez également l’[exemple de compatibilité OpenCV](https://github.com/Microsoft/Azure-Kinect-Sensor-SDK/tree/develop/examples/opencv_compatibility) du Kit de développement logiciel (SDK) qui illustre la conversion entre le type [k4a_calibration_t](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__calibration__t.html#details) et les structures de données OpenCV correspondantes.

## <a name="coordinate-transformation-functions"></a>Fonctions de transformation de coordonnées

La figure ci-dessous présente les différents systèmes de coordonnées d’Azure Kinect, ainsi que les fonctions permettant de les convertir entre eux. Pour que la figure reste simple, nous n’indiquons pas les systèmes de coordonnées 3D de gyroscope et d’accéléromètre.

   ![Transformation de coordonnées](./media/how-to-guides/coordinate-transformation.png)

Remarque concernant la distorsion de l’objectif : les coordonnées 2D font toujours référence à l’image distordue dans le Kit de développement logiciel (SDK). L’[exemple de correction de distorsion](https://github.com/Microsoft/Azure-Kinect-Sensor-SDK/tree/develop/examples/undistort) du Kit de développement logiciel (SDK) montre la correction de distorsion d’image. En général, les points 3D ne sont jamais affectés par la distorsion de l’objectif.

### <a name="convert-between-3d-coordinate-systems"></a>Opérer une conversion entre systèmes de coordonnées 3D

La fonction [k4a_calibration_3d_to_3d()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gaacd1eab997ef964b5e436afa5295726e.html#gaacd1eab997ef964b5e436afa5295726e) convertit un point 3D du système de coordonnées source en point 3D du système de coordonnées cible en utilisant l’étalonnage extrinsèque de la caméra. La source et la cible peuvent être définies sur l’un des quatre systèmes de coordonnées 3D, à savoir, caméra couleur, caméra de profondeur, gyroscope ou accéléromètre. Si la source et la cible sont identiques, le point 3D d’entrée non modifié est retourné en tant que sortie.

### <a name="convert-between-2d-and-3d-coordinate-systems"></a>Conversion entre systèmes de coordonnées 2D et 3D

La fonction [k4a_calibration_3d_to_2d()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga2ed8b51d727425caa942aab190fc2ba9.html#ga2ed8b51d727425caa942aab190fc2ba9) convertit un point 3D du système de coordonnées source en coordonnée de pixel 2D de la caméra cible. Cette fonction est souvent appelée fonction de projection. Si la source peut être définie sur l’un des quatre systèmes de coordonnées 3D, la cible doit être la caméra de profondeur ou couleur. Si la source et la cible sont différentes, le point d’entrée 3D est converti dans le système de coordonnées 3D de la caméra cible à l’aide de la fonction [k4a_calibration_3d_to_3d()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gaacd1eab997ef964b5e436afa5295726e.html#gaacd1eab997ef964b5e436afa5295726e). Une fois le point 3D représenté dans le système de coordonnées de la caméra cible, les coordonnées de pixel 2D correspondantes sont calculées en utilisant l’étalonnage intrinsèque de la caméra cible. Si un point 3D se trouve en dehors de la zone visible de la caméra cible, la valeur valide est définie sur 0.

La fonction [k4a_calibration_2d_to_3d()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga664602bdb48dab38117a6c1d14b880de.html#ga664602bdb48dab38117a6c1d14b880de) convertit une coordonnée de pixel 2D de la caméra source en un point 3D du système de coordonnées de la caméra cible. La source doit être une caméra couleur ou de profondeur. La cible peut être définie sur l’un des quatre systèmes de coordonnées 3D. En plus de la coordonnée de pixel 2D, la valeur de profondeur du pixel (en millimètres) dans l’image de la caméra source est requise comme entrée pour la fonction. Une façon de dériver la valeur de profondeur dans la géométrie de la caméra couleur consiste à utiliser la fonction [k4a_transformation_depth_image_to_color_camera()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gafacffb5f781a9c2df30d4a16241cd514.html#gafacffb5f781a9c2df30d4a16241cd514). La fonction calcule le rayon 3D qui mène du point focal de la caméra source à la coordonnée de pixel spécifiée en utilisant l’étalonnage intrinsèque de la caméra source. La valeur de profondeur est ensuite utilisée pour rechercher l’emplacement exact du point 3D sur ce rayon. Cette opération est souvent appelée fonction de déprojection. Si les caméras source et cible sont différentes, la fonction transforme le point 3D au système de coordonnées de la cible via la fonction [k4a_calibration_3d_to_3d()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gaacd1eab997ef964b5e436afa5295726e.html#gaacd1eab997ef964b5e436afa5295726e). Si une coordonnée de pixel 2D se trouve en dehors de la zone visible de la caméra source, la valeur valide est définie sur 0.

### <a name="converting-between-2d-coordinate-systems"></a>Conversion entre systèmes de coordonnées 2D

La fonction [k4a_calibration_2d_to_2d()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga3b6bf6dedbfe67468e2f895dcce68ed4.html#ga3b6bf6dedbfe67468e2f895dcce68ed4) convertit une coordonnée de pixel 2D de la caméra source en coordonnée de pixel 2D de la caméra cible. La source et la cible définies doivent être une caméra couleur ou de profondeur. La fonction requiert la valeur de profondeur du pixel (en millimètres) dans l’image de la caméra source en entrée. Une façon de dériver la valeur de profondeur dans la géométrie de la caméra de couleur consiste à utiliser la fonction [k4a_transformation_depth_image_to_color_camera()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gafacffb5f781a9c2df30d4a16241cd514.html#gafacffb5f781a9c2df30d4a16241cd514). Elle appelle la fonction [k4a_calibration_2d_to_3d()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga664602bdb48dab38117a6c1d14b880de.html#ga664602bdb48dab38117a6c1d14b880de) pour effectuer la conversion en un point 3D du système de la caméra source. Elle appelle ensuite la fonction [k4a_calibration_3d_to_2d()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga2ed8b51d727425caa942aab190fc2ba9.html#ga2ed8b51d727425caa942aab190fc2ba9) pour convertir en coordonnée de pixel 2D de l’image de la caméra cible. La valeur valide est définie sur 0, si la fonction [k4a_calibration_2d_to_3d()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga664602bdb48dab38117a6c1d14b880de.html#ga664602bdb48dab38117a6c1d14b880ded) ou [k4a_calibration_3d_to_2d()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga2ed8b51d727425caa942aab190fc2ba9.html#ga2ed8b51d727425caa942aab190fc2ba9) retourne un résultat non valide.

## <a name="related-samples"></a>Exemples connexes

- [Exemple de compatibilité OpenCV](https://github.com/Microsoft/Azure-Kinect-Sensor-SDK/tree/develop/examples/opencv_compatibility)
- [Exemple de correction de distorsion](https://github.com/Microsoft/Azure-Kinect-Sensor-SDK/tree/develop/examples/undistort)
- [Exemple de nuage de points rapide](https://github.com/Microsoft/Azure-Kinect-Sensor-SDK/tree/develop/examples/fastpointcloud)

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous savez ce que sont les étalonnages de caméra, vous pouvez également apprendre à
>[!div class="nextstepaction"]
>[Capturer la synchronisation des appareils](capture-device-synchronization.md)

Vous pouvez également consulter

[Systèmes de coordonnées](coordinate-systems.md)
