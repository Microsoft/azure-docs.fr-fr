---
title: Récupérer des données d’image Azure Kinect
description: Découvrez comment récupérer des données d’image Azure Kinect à l’aide du Kit de développement logiciel (SDK) de capteur Kinect.
author: cdedmonds
ms.author: cedmonds
ms.prod: kinect-dk
ms.date: 06/26/2019
ms.topic: conceptual
keywords: kinect, azure, récupérer, capteur, caméra, kit de développement logiciel (sdk), profondeur, rvb, images, couleur, capture, résolution, tampon
ms.openlocfilehash: fed5c7340d287d9103ba35f0fd3d80c0fff6e3ec
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "87538912"
---
# <a name="retrieve-azure-kinect-image-data"></a>Récupérer des données d’image Azure Kinect

Cette page fournit des informations détaillées sur la façon de récupérer des images à partir de l’appareil Azure Kinect. L’article montre comment capturer des images coordonnées et y accéder entre la couleur et la profondeur de l’appareil. Pour accéder aux images, vous devez commencer par ouvrir et configurer l’appareil. Vous pouvez ensuite capturer des images.
Avant de configurer et de capturer une image, vous devez [Rechercher et ouvrir un appareil](find-then-open-device.md).

Vous pouvez également consulter l’[exemple de diffusion en continu du Kit de développement logiciel (SDK)](https://github.com/microsoft/Azure-Kinect-Sensor-SDK/tree/develop/examples/streaming) qui montre comment utiliser les fonctions abordées dans cet article.

Voici les fonctions qui sont abordées ici :

- [`k4a_device_start_cameras()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gaad7a85e1e5471810262442fc4a8e217a.html#gaad7a85e1e5471810262442fc4a8e217a)
- [`k4a_device_get_capture()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga4dac757a33657f4d3dbf1ae8b21c158a.html#ga4dac757a33657f4d3dbf1ae8b21c158a)
- [`k4a_capture_get_depth_image()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gafa03513da96bf6b8c254fc11a04ee6d6.html#gafa03513da96bf6b8c254fc11a04ee6d6)
- [`k4a_image_get_buffer()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga2ef070cf4e543cd0f726478af332546e.html#ga2ef070cf4e543cd0f726478af332546e)
- [`k4a_image_release()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga27c81863b13fafc3934a32935a014e9f.html#ga27c81863b13fafc3934a32935a014e9f)
- [`k4a_capture_release()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga0ed6f74ec403c3eac1b8ef3afb32cee6.html#ga0ed6f74ec403c3eac1b8ef3afb32cee6)
- [`k4a_device_stop_cameras()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga4fa0e0a011a7105309ad97f081a5d6b8.html#ga4fa0e0a011a7105309ad97f081a5d6b8)
  
## <a name="configure-and-start-the-device"></a>Configurer et démarrer l’appareil

Les deux caméras disponibles sur votre appareil Kinect prennent en charge plusieurs modes, résolutions et formats de sortie. Pour obtenir une liste complète, reportez-vous au Kit de développement Azure Kinect [Spécifications matérielles](hardware-specification.md).

La configuration de la diffusion en continu est définie à l’aide de valeurs de la structure [`k4a_device_configuration_t`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__device__configuration__t.html).

```C
k4a_device_configuration_t config = K4A_DEVICE_CONFIG_INIT_DISABLE_ALL;
config.camera_fps = K4A_FRAMES_PER_SECOND_30;
config.color_format = K4A_IMAGE_FORMAT_COLOR_MJPG;
config.color_resolution = K4A_COLOR_RESOLUTION_2160P;
config.depth_mode = K4A_DEPTH_MODE_NFOV_UNBINNED;

if (K4A_RESULT_SUCCEEDED != k4a_device_start_cameras(device, &config))
{
    printf("Failed to start device\n");
    goto Exit;
}
```

Une fois les caméras démarrées, elles continuent à capturer des données jusqu’à l’appel de la fonction [`k4a_device_stop_cameras()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga4fa0e0a011a7105309ad97f081a5d6b8.html#ga4fa0e0a011a7105309ad97f081a5d6b8) ou à la fermeture de l’appareil.

## <a name="stabilization"></a>Stabilisation

Lors du démarrage d’appareils à l’aide de la fonctionnalité de synchronisation d’appareils multiples, il est fortement recommandé de le faire à l’aide d’un paramètre à exposition fixe.
Avec une exposition manuelle, jusqu’à huit captures de l’appareil peuvent être nécessaires avant la stabilisation des images et des taux de trames. Avec une exposition automatique, jusqu’à 20 captures avant que peuvent être nécessaires avant la stabilisation des images et des taux de trames.

## <a name="get-a-capture-from-the-device"></a>Obtenir une capture à partir de l’appareil

Les images sont capturées à partir de l’appareil de manière corrélée. Chaque image capturée contient une image de profondeur, une image IR, une image en couleur ou une combinaison d’images.

Par défaut, l’API retourne une capture uniquement une fois qu’elle a reçu toutes les images demandées pour le mode de diffusion en continu. Vous pouvez configurer l’API de sorte qu’elle retourne des captures partielles avec uniquement des images de profondeur ou en couleurs dès qu’elles sont disponibles en désactivant le paramètre [`synchronized_images_only`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__device__configuration__t_a8208974f05d89fc1362c6a0900bdef4d.html#a8208974f05d89fc1362c6a0900bdef4d) de la fonction [`k4a_device_configuration_t`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__device__configuration__t.html).

```C
// Capture a depth frame
switch (k4a_device_get_capture(device, &capture, TIMEOUT_IN_MS))
{
case K4A_WAIT_RESULT_SUCCEEDED:
    break;
case K4A_WAIT_RESULT_TIMEOUT:
    printf("Timed out waiting for a capture\n");
    continue;
    break;
case K4A_WAIT_RESULT_FAILED:
    printf("Failed to read a capture\n");
    goto Exit;
}
```

Une fois que l’API a correctement retourné une capture, vous devez appeler [`k4a_capture_release()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga0ed6f74ec403c3eac1b8ef3afb32cee6.html#ga0ed6f74ec403c3eac1b8ef3afb32cee6) lorsque vous avez fini d’utiliser l’objet de capture.

## <a name="get-an-image-from-the-capture"></a>Obtenir une image de la capture

Pour récupérer une image capturée, appelez la fonction appropriée pour chaque type d’image. Valeurs possibles :

- [`k4a_capture_get_color_image()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga683e440b5f22215a2de58d7fa140488c.html#ga683e440b5f22215a2de58d7fa140488c)
- [`k4a_capture_get_depth_image()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gafa03513da96bf6b8c254fc11a04ee6d6.html#gafa03513da96bf6b8c254fc11a04ee6d6)
- [`k4a_capture_get_ir_image()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga1531c3fa76a7c174b8f2eab24de91794.html#ga1531c3fa76a7c174b8f2eab24de91794)

Une fois que vous avez fini d’utiliser l’image, vous devez appeler la fonction [`k4a_image_release()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga27c81863b13fafc3934a32935a014e9f.html#ga27c81863b13fafc3934a32935a014e9f) sur tout descripteur [`k4a_image_t`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__image__t.html) retourné par ces fonctions.

## <a name="access-image-buffers"></a>Accéder aux tampons d’image

[`k4a_image_t`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__image__t.html) possède de nombreuses fonctions d’accesseur pour l’extraction de propriétés de l’image.

Pour accéder à la mémoire tampon de l’image, utilisez [k4a_image_get_buffer](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga2ef070cf4e543cd0f726478af332546e.html#ga2ef070cf4e543cd0f726478af332546e).

L’exemple suivant montre comment accéder à une image de profondeur capturée. Ce même principe s’applique à d’autres types d’images. Toutefois, veillez à remplacer la variable de type image par le type d’image correct, tel que IR ou Couleur.

```C
// Access the depth16 image
k4a_image_t image = k4a_capture_get_depth_image(capture);
if (image != NULL)
{
    printf(" | Depth16 res:%4dx%4d stride:%5d\n",
            k4a_image_get_height_pixels(image),
            k4a_image_get_width_pixels(image),
            k4a_image_get_stride_bytes(image));

    // Release the image
    k4a_image_release(image);
}

// Release the capture
k4a_capture_release(capture);
```

## <a name="next-steps"></a>Étapes suivantes

Vous savez maintenant comment capturer et coordonner les images des caméras couleur et de profondeur à l’aide de votre appareil Azure Kinect. Vous pouvez également :

>[!div class="nextstepaction"]
>[Récupérer des échantillons IMU](retrieve-imu-samples.md)

>[!div class="nextstepaction"]
>[Accéder aux microphones](access-mics.md)
