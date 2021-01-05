---
title: Récupérer des échantillons d’IMU Azure Kinect
description: Découvrez comment récupérer des échantillons d’IMU Azure Kinect à l’aide du Kit de développement logiciel (SDK) Azure Kinect.
author: qm13
ms.author: cedmonds
ms.date: 06/26/2019
ms.prod: kinect-dk
ms.topic: conceptual
keywords: kinect, azure, configurer, profondeur, couleur, rvg, caméra, capteur, kit de développement logiciel (sdk), imu, capteur de mouvement, mouvement, gyroscope, gyro, accéléromètre, i/s, fps
ms.openlocfilehash: 649dd5b9de62d43d59e74d53adff1ec7de8dfd32
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85276479"
---
# <a name="retrieve-azure-kinect-imu-samples"></a>Récupérer des échantillons d’IMU Azure Kinect

L’appareil Azure Kinect donne accès aux unités de mesure inertielle (IMU), notamment des types accéléromètre et gyroscope. Pour accéder aux échantillons d’IMU, vous devez ouvrir et configurer votre appareil, puis capturer des données d’IMU. Pour plus d’informations, consultez [Rechercher et ouvrir un appareil](find-then-open-device.md).

Les échantillons d’IMU sont générés à une fréquence sensiblement plus élevée que les images. Les échantillons sont signalés à l’hôte à un débit inférieur à celui de leur collecte. Lors de l’attente d’un échantillon d’IMU, il est fréquent que plusieurs échantillons deviennent disponibles en même temps.

Pour plus d’informations sur le taux de création de rapports sur les IMU, consultez les [spécifications matérielles](hardware-specification.md) d’Azure Kinect DK.

## <a name="configure-and-start-cameras"></a>Configurer et démarrer des caméras

> [!NOTE]
> Les capteurs IMU ne peuvent fonctionner que lorsque les caméras couleur et/ou de profondeur sont en cours d’exécution. Les capteurs IMU ne peuvent pas fonctionner isolément.

Pour démarrer les caméras, utilisez [k4a_device_start_cameras()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gaad7a85e1e5471810262442fc4a8e217a.html#gaad7a85e1e5471810262442fc4a8e217a).

```C
k4a_device_configuration_t config = K4A_DEVICE_CONFIG_INIT_DISABLE_ALL;
config.camera_fps = K4A_FRAMES_PER_SECOND_30;
config.color_format = K4A_IMAGE_FORMAT_COLOR_MJPG;
config.color_resolution = K4A_COLOR_RESOLUTION_2160P;

if (K4A_RESULT_SUCCEEDED != k4a_device_start_cameras(device, &config))
{
    printf("Failed to start cameras\n");
    goto Exit;
}

if (K4A_RESULT_SUCCEEDED != k4a_device_start_imu(device))
{
    printf("Failed to start imu\n");
    goto Exit;
}
```

## <a name="access-imu-samples"></a>Accéder aux échantillons d’IMU

 Chaque échantillon [k4a_imu_sample_t](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__imu__sample__t.html#details) contient des lectures d’accéléromètre et de gyroscope capturées presque en même temps.

Vous pouvez recevoir les échantillons d’IMU sur le même thread que celui sur lequel vous recevez les captures d’images ou sur des threads distincts.

Pour récupérer les échantillons d’IMU dès qu’ils sont disponibles, vous pouvez appeler [`k4a_device_get_imu_sample()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga8e5913b3bb94a453c7143bbd6e399a0e.html#ga8e5913b3bb94a453c7143bbd6e399a0e) sur son propre thread. L’API dispose également d’une mise en file d’attente interne suffisante pour vous permettre de vérifier les échantillons uniquement après le retour de chaque capture d’image.

Étant donné qu’il existe une mise en file d’attente interne des échantillons d’IMU, vous pouvez utiliser le modèle suivant sans supprimer de données :

1. Attendez une capture, quelle que soit la fréquence des images.
2. Traitez la capture.
3. Récupérez tous les échantillons d’IMU mis en file d’attente.
4. Répétez l’attente pour la capture suivante.

Pour récupérer tous les échantillons d’IMU actuellement en file d’attente, vous pouvez appeler la fonction [`k4a_device_get_imu_sample()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga8e5913b3bb94a453c7143bbd6e399a0e.html#ga8e5913b3bb94a453c7143bbd6e399a0e) avec un `timeout_in_ms` de 0 dans une boucle jusqu’à ce que la fonction retourne `K4A_WAIT_RESULT_TIMEOUT`. `K4A_WAIT_RESULT_TIMEOUT` indique qu’aucun échantillon n’est en file d’attente et n’est arrivé dans le délai spécifié.

## <a name="usage-example"></a>Exemple d’utilisation

```C
k4a_imu_sample_t imu_sample;

// Capture a imu sample
switch (k4a_device_get_imu_sample(device, &imu_sample, TIMEOUT_IN_MS))
{
case K4A_WAIT_RESULT_SUCCEEDED:
    break;
case K4A_WAIT_RESULT_TIMEOUT:
    printf("Timed out waiting for a imu sample\n");
    continue;
    break;
case K4A_WAIT_RESULT_FAILED:
    printf("Failed to read a imu sample\n");
    goto Exit;
}

// Access the accelerometer readings
if (imu_sample != NULL)
{
    printf(" | Accelerometer temperature:%.2f x:%.4f y:%.4f z: %.4f\n",
            imu_sample.temperature,
            imu_sample.acc_sample.xyz.x,
            imu_sample.acc_sample.xyz.y,
            imu_sample.acc_sample.xyz.z);
}

```

## <a name="next-steps"></a>Étapes suivantes

Maintenant que vous savez comment utiliser les échantillons d’IMU, vous pouvez également
>[!div class="nextstepaction"]
>[Accéder aux données d’entrée du microphone](access-mics.md)
