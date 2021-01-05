---
title: Utiliser les transformations d’image du Kit de développement logiciel (SDK) du capteur Azure Kinect
description: Découvrez comment utiliser les fonctions de transformation d’image du Kit de développement logiciel (SDK) du capteur Azure Kinect.
author: cdedmonds
ms.author: cedmonds
ms.prod: kinect-dk
ms.topic: conceptual
ms.date: 06/26/2019
keywords: kinect, azure, capteur, kit de développement logiciel (sdk), système de coordonnées, étalonnage, projet, annuler un projet, transformation, rvb-d, nuage de points
ms.openlocfilehash: df7f2aa13c0e9c0241494e96e720b30f3ff1d8a5
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85276456"
---
# <a name="use-azure-kinect-sensor-sdk-image-transformations"></a>Utiliser les transformations d’image du Kit de développement logiciel (SDK) du capteur Azure Kinect

Suivez les fonctions spécifiques pour l’utilisation et la transformation d’images entre systèmes de caméras coordonnés dans Azure Kinect DK.

## <a name="k4a_transformation-functions"></a>Fonctions k4a_transformation

 Toutes les fonctions dont le nom commence par le préfixe *k4a_transformation* opèrent sur des images entières. Elles requièrent le descripteur de transformation [k4a_transformation_t](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__transformation__t.html) obtenu via la fonction [k4a_transformation_create()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga853a1a5b6d521bbdf523a69e890c4f10.html#ga853a1a5b6d521bbdf523a69e890c4f10), et sont non allouées via la fonction [k4a_transformation_destroy()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga7d3ecaae66f26c1a89da9042b1bc6d44.html#ga7d3ecaae66f26c1a89da9042b1bc6d44). Vous pouvez également consulter dans cette rubrique l’[exemple de transformation](https://github.com/Microsoft/Azure-Kinect-Sensor-SDK/tree/develop/examples/transformation) du Kit de développement logiciel (SDK) qui montre comment utiliser les trois fonctions.

Voici les fonctions qui sont abordées ici :

* [`k4a_transformation_depth_image_to_color_camera()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gafacffb5f781a9c2df30d4a16241cd514.html#gafacffb5f781a9c2df30d4a16241cd514)
* [`k4a_transformation_depth_image_to_color_camera_custom()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gac00dd00e7612a86382e3d0a130f276bb.html#gac00dd00e7612a86382e3d0a130f276bb)
* [`k4a_transformation_color_image_to_depth_camera()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gaf3a941f07bb0185cd7a72699a648fc29.html#gaf3a941f07bb0185cd7a72699a648fc29)
* [`k4a_transformation_depth_image_to_point_cloud()`](
https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga7385eb4beb9d8892e8a88cf4feb3be70.html#ga7385eb4beb9d8892e8a88cf4feb3be70)

### <a name="k4a_transformation_depth_image_to_color_camera"></a>k4a_transformation_depth_image_to_color_camera

#### <a name="overview"></a>Vue d’ensemble

 La fonction [k4a_transformation_depth_image_to_color_camera()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gafacffb5f781a9c2df30d4a16241cd514.html#gafacffb5f781a9c2df30d4a16241cd514) transforme la carte de profondeur du point de vue de la caméra de profondeur en point de vue de la caméra couleur. Cette fonction est conçue pour produire des images RVB-D, où D représente un canal d’image supplémentaire qui enregistre la valeur de profondeur. Comme illustré ci-dessous, l’image en couleurs et la sortie de la fonction [k4a_transformation_depth_image_to_color_camera()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gafacffb5f781a9c2df30d4a16241cd514.html#gafacffb5f781a9c2df30d4a16241cd514) semblent avoir été prises du même point de vue, c’est-à-dire celui de la caméra couleur.

   ![Transformation d’image](./media/how-to-guides/image-transformation.png)

#### <a name="implementation"></a>Implémentation

 Cette fonction de transformation est plus complexe que le simple appel de la fonction [k4a_calibration_2d_to_2d()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga3b6bf6dedbfe67468e2f895dcce68ed4.html#ga3b6bf6dedbfe67468e2f895dcce68ed4) pour chaque pixel. Elle distord le maillage triangulaire de la géométrie de la caméra de profondeur pour l’adapter à la géométrie de la caméra couleur. Le maillage triangulaire est utilisé pour éviter de générer des trous dans l’image de profondeur transformée. Un tampon z-buffer garantit que les occlusions sont gérées correctement. Par défaut , l’accélération GPU est activée pour cette fonction.

#### <a name="parameters"></a>Paramètres

 Les paramètres d’entrée sont le descripteur de transformation et une image de profondeur. La résolution de l’image de profondeur doit correspondre au ```depth_mode``` spécifié lors de la création du descripteur de transformation. Par exemple, si le descripteur de transformation a été créé à l’aide du mode ```K4A_DEPTH_MODE_WFOV_UNBINNED``` 1024x1024, la résolution de l’image de profondeur doit être de 1024 x 1024 pixels. La sortie est une image de profondeur transformée qui doit être allouée par l’utilisateur via l’appel de [k4a_image_create()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga859554581bb97a620ff8e92a893e71ef.html#ga859554581bb97a620ff8e92a893e71ef). La résolution de l’image de profondeur transformée doit correspondre à la ```color_resolution``` spécifiée lors de la création du descripteur de transformation. Par exemple, si la résolution des couleurs a été définie sur `K4A_COLOR_RESOLUTION_1080P`, la résolution de l’image de sortie doit être de 1920 x 1080 pixels. Le stride de l’image de sortie est défini sur `width * sizeof(uint16_t)`, car l’image stocke des valeurs de profondeur de 16 bits.

### <a name="k4a_transformation_depth_image_to_color_camera_custom"></a>k4a_transformation_depth_image_to_color_camera_custom

#### <a name="overview"></a>Vue d’ensemble

 La fonction [k4a_transformation_depth_image_to_color_camera_custom()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gac00dd00e7612a86382e3d0a130f276bb.html#gac00dd00e7612a86382e3d0a130f276bb) transforme la carte de profondeur et une image personnalisée du point de vue de la caméra de profondeur en point de vue de la caméra couleur. En tant qu’extension de la fonction [k4a_transformation_depth_image_to_color_camera()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gafacffb5f781a9c2df30d4a16241cd514.html#gafacffb5f781a9c2df30d4a16241cd514), cette fonction est conçue pour produire une image personnalisée dont chaque pixel correspond à des coordonnées de pixel de la caméra couleur, en plus de l’image de profondeur transformée.

#### <a name="implementation"></a>Implémentation

 Cette fonction de transformation produit l’image de profondeur transformée de la même façon que la fonction [k4a_transformation_depth_image_to_color_camera()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gafacffb5f781a9c2df30d4a16241cd514.html#gafacffb5f781a9c2df30d4a16241cd514). Pour transformer l’image personnalisée, cette fonction fournit des options d’interpolation linéaire ou d’interpolation du voisin le plus proche. Une interpolation linéaire pourrait créer de nouvelles valeurs dans l’image personnalisée transformée. Une interpolation du voisin le plus proche empêche l’affichage dans l’image de sortie de valeurs non présentes dans l’image d’origine, mais produit une image moins lisse. L’image personnalisée doit être un canal unique 8 bits ou 16 bits. Par défaut , l’accélération GPU est activée pour cette fonction.

#### <a name="parameters"></a>Paramètres

 Les paramètres d’entrée sont le descripteur de transformation, une image de profondeur, une image personnalisée et le type d’interpolation. Les résolutions de l’image de profondeur de l’image personnalisée doivent correspondre au `depth_mode` spécifié lors de la création du descripteur de transformation. Par exemple, si le descripteur de transformation a été créé à l’aide du mode `K4A_DEPTH_MODE_WFOV_UNBINNED` 1024x1024, les résolutions de l’image de profondeur et de l’image personnalisée doivent être de 1024 x 1024 pixels. Le `interpolation_type` doit être `K4A_TRANSFORMATION_INTERPOLATION_TYPE_LINEAR` ou `K4A_TRANSFORMATION_INTERPOLATION_TYPE_NEAREST`. La sortie est une image de profondeur transformée et une image personnalisée transformée qui doivent être allouées par l’utilisateur via l’appel de la fonction [k4a_image_create()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga859554581bb97a620ff8e92a893e71ef.html#ga859554581bb97a620ff8e92a893e71ef). Les résolutions de l’image de profondeur transformée et de l’image personnalisée transformée doivent correspondre à la valeur `color_resolution` spécifiée lors de la création du descripteur de transformation. Par exemple, si la résolution des couleurs a été définie sur `K4A_COLOR_RESOLUTION_1080P`, la résolution de l’image de sortie doit être de 1920 x 1080 pixels. Le stride de l’image de profondeur sortie est défini sur `width * sizeof(uint16_t)`, car l’image stocke des valeurs de profondeur de 16 bits. L’image personnalisée d’entrée et l’image personnalisée transformée doivent être au format `K4A_IMAGE_FORMAT_CUSTOM8` ou `K4A_IMAGE_FORMAT_CUSTOM16`, et le stride d’image correspondant doit être défini en conséquence. 

### <a name="k4a_transformation_color_image_to_depth_camera"></a>k4a_transformation_color_image_to_depth_camera

#### <a name="overview"></a>Vue d’ensemble

 La fonction [k4a_transformation_color_image_to_depth_camera()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gaf3a941f07bb0185cd7a72699a648fc29.html#gaf3a941f07bb0185cd7a72699a648fc29) transforme l’image en couleurs du point de vue de la caméra couleur en point de vue de la caméra de profondeur (voir la figure ci-dessus). Vous pouvez l’utiliser pour générer des images RVB-D.

#### <a name="implementation"></a>Implémentation

 Pour chaque pixel de la carte de profondeur, la fonction utilise la valeur de profondeur du pixel pour calculer la coordonnée de sous-pixel correspondante dans l’image en couleurs. Nous recherchons ensuite la valeur de couleur à cette coordonnée dans l’image en couleurs. Une interpolation bilinéaire est effectuée dans l’image en couleurs pour obtenir la valeur de couleur avec une précision de sous-pixel. Un pixel qui n’est pas associé à une mesure de profondeur est affecté à une valeur BGRA de `[0,0,0,0]` dans l’image de sortie. Par défaut , l’accélération GPU est activée pour cette fonction. Étant donné que cette méthode produit des trous dans l’image en couleurs transformée et ne gère pas les occlusions, nous vous recommandons d’utiliser la fonction [k4a_transformation_depth_image_to_color_camera()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gafacffb5f781a9c2df30d4a16241cd514.html#gafacffb5f781a9c2df30d4a16241cd514) à la place.

#### <a name="parameters"></a>Paramètres

Les paramètres d’entrée sont le descripteur de transformation, une image de profondeur et une image en couleurs. Les résolutions des images de profondeur et en couleurs doivent correspondre aux valeurs depth_mode et color_resolution spécifiées lors de la création du descripteur de transformation. La sortie est une image en couleurs transformée qui doit être allouée par l’utilisateur via l’appel de [k4a_image_create()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga859554581bb97a620ff8e92a893e71ef.html#ga859554581bb97a620ff8e92a893e71ef). La résolution de l’image en couleurs transformée doit correspondre à la valeur depth_resolution spécifiée lors de la création du descripteur de transformation. L’image de sortie stocke quatre valeurs 8 bits représentant BGRA pour chaque pixel. Par conséquent, le stride de l’image est ```width * 4 * sizeof(uint8_t)```. L’ordre des données est celui des pixels entrelacés, c’est-à-dire la valeur bleu -pixel 0, la valeur verte -pixel 0, la valeur rouge -pixel 0, la valeur alpha -pixel 0, la valeur bleue -pixel 1, et ainsi de suite.

### <a name="k4a_transformation_depth_image_to_point_cloud"></a>k4a_transformation_depth_image_to_point_cloud

#### <a name="overview"></a>Vue d’ensemble

La fonction [k4a_transformation_depth_image_to_point_cloud()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga7385eb4beb9d8892e8a88cf4feb3be70.html#ga7385eb4beb9d8892e8a88cf4feb3be70) convertit une carte de profondeur 2D prise par une caméra en nuage de points 3D dans le système de coordonnées de la même caméra. La caméra peut donc être la caméra de profondeur ou la caméra couleur.

#### <a name="implementation"></a>Implémentation

 La fonction produit des résultats équivalents à ceux que produit la fonction [k4a_calibration_2d_to_2d()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga3b6bf6dedbfe67468e2f895dcce68ed4.html#ga3b6bf6dedbfe67468e2f895dcce68ed4) pour chaque pixel, mais elle est plus efficace en termes de calcul. Lors de l’appel de la fonction [k4a_transformation_create()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga853a1a5b6d521bbdf523a69e890c4f10.html#ga853a1a5b6d521bbdf523a69e890c4f10), nous précalculons une table de recherche-xy qui stocke les facteurs d’échelle x et y pour chaque pixel d’image. Lors de l’appel de la fonction [k4a_transformation_depth_image_to_point_cloud(](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga7385eb4beb9d8892e8a88cf4feb3be70.html#ga7385eb4beb9d8892e8a88cf4feb3be70)), nous obtenons la coordonnée 3D x d’un pixel en multipliant le facteur d’échelle x du pixel par la coordonnée z du pixel. De même, la coordonnée 3D y est calculée via une multiplication avec le facteur d’échelle y. L’[exemple Fast point Cloud](https://github.com/Microsoft/Azure-Kinect-Sensor-SDK/tree/develop/examples/fastpointcloud) du Kit de développement logiciel (SDK) illustre le mode de calcul de la table xy. Les utilisateurs peuvent se référer à l’exemple de code pour implémenter leur propre version de cette fonction, par exemple, pour accélérer leur pipeline GPU.

#### <a name="parameters"></a>Paramètres

 Les paramètres d’entrée sont le descripteur de transformation, un spécificateur de caméra et une image de profondeur. Si le spécificateur de caméra défini est la profondeur, la résolution de l’image de profondeur doit correspondre à la valeur depth_mode spécifiée lors de la création du descripteur de transformation. Si le spécificateur défini est la caméra couleur, la résolution doit correspondre à la valeur color_resolution choisie. Le paramètre de sortie est une image XYZ que l’utilisateur doit allouer via l’appel de la fonction [k4a_image_create()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga859554581bb97a620ff8e92a893e71ef.html#ga859554581bb97a620ff8e92a893e71ef). La résolution d’image XYZ doit correspondre à celle de la carte de profondeur d’entrée. Nous stockons trois valeurs de coordonnées 16 bits signées exprimées en millimètres pour chaque pixel. Le stride d’image XYZ est donc défini sur `width * 3 * sizeof(int16_t)`. L’ordre des données est celui des pixels entrelacés, c’est-à-dire la coordonnée X -pixel 0, la coordonnée Y -pixel 0, la coordonnée Z -pixel 0, la coordonnée X -pixel 1 et ainsi de suite. Si un pixel ne peut pas être converti en 3D, la fonction affecte au pixel les valeurs `[0,0,0]`.

## <a name="samples"></a>Exemples

[Exemple de transformation](https://github.com/Microsoft/Azure-Kinect-Sensor-SDK/tree/develop/examples/transformation)

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous savez comment utiliser les fonctions de transformation d’image du Kit de développement logiciel (SDK) du capteur Azure Kinect, vous pouvez également en apprendre davantage sur

>[!div class="nextstepaction"]
>[Fonctions d’étalonnage du Kit de développement logiciel (SDK) du capteur Kinect Azure](use-calibration-functions.md)

Vous pouvez également consulter

[Systèmes de coordonnées](coordinate-systems.md)
