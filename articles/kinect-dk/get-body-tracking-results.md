---
title: Résultats de suivi de corps Azure Kinect
description: Découvrez comment obtenir les résultats de suivi de corps à l’aide du Kit de développement logiciel (SDK) de suivi de corps Azure Kinect.
author: qm13
ms.prod: kinect-dk
ms.author: quentinm
ms.reviewer: yijwan
ms.date: 06/26/2019
ms.topic: conceptual
keywords: kinect, azure, capture, sdk, corps, suivi, jointure
ms.openlocfilehash: 1b62022242144d5db51455a32ac04b67c3e5dd7a
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85276532"
---
# <a name="get-body-tracking-results"></a>Obtenir les résultats Body Tracking

Le Kit de développement logiciel (SDK) de suivi de corps utilise un objet dédié au suivi de corps pour traiter les captures Azure Kinect DK, et génère des résultats de suivi de corps. Il gère également l’état global du dispositif de suivi, le traitement des files d’attente et la file d’attente de sortie. L’utilisation du dispositif de suivi de corps comporte trois étapes :

- Créer un dispositif de suivi
- Capturer des images de profondeur et IR à l’aide d’un appareil Azure Kinect DK
- Mettre en file d’attente la capture et afficher les résultats

## <a name="create-a-tracker"></a>Créer un dispositif de suivi


La première étape de l’utilisation du suivi de corps consiste à créer un dispositif de suivi et à suivre la procédure d’étalonnage du capteur [k4a_calibration_t](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__calibration__t.html). L’étalonnage du capteur peut être interrogé à l’aide de la fonction [k4a_device_get_calibration()](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga4e43940d8d8db48da266c7a7842c8d78.html#ga4e43940d8d8db48da266c7a7842c8d78) du Kit de développement logiciel (SDK) du capteur Azure Kinect.

```C
k4a_calibration_t sensor_calibration;
if (K4A_RESULT_SUCCEEDED != k4a_device_get_calibration(device, device_config.depth_mode, K4A_COLOR_RESOLUTION_OFF, &sensor_calibration))
{
    printf("Get depth camera calibration failed!\n");
    return 0;
}

k4abt_tracker_t tracker = NULL;
k4abt_tracker_configuration_t tracker_config = K4ABT_TRACKER_CONFIG_DEFAULT;
if (K4A_RESULT_SUCCEEDED != k4abt_tracker_create(&sensor_calibration, tracker_config, &tracker))
{
    printf("Body tracker initialization failed!\n");
    return 0;
}
```

## <a name="capture-depth-and-ir-images"></a>Capturer des images de profondeur et IR

La capture d’images à l’aide d’Azure Kinect DK est traitée dans la page [Récupérer les images](retrieve-images.md).

>[!NOTE]
> Les modes `K4A_DEPTH_MODE_NFOV_UNBINNED` ou `K4A_DEPTH_MODE_WFOV_2X2BINNED` sont recommandés pour bénéficier de performances et d’une précision optimales. N’utilisez pas les modes `K4A_DEPTH_MODE_OFF` ou `K4A_DEPTH_MODE_PASSIVE_IR`.

Les modes d’Azure Kinect DK pris en charge sont décrits dans les [spécifications matérielles](hardware-specification.md) d’Azure Kinect DK et les énumérations [k4a_depth_mode_t](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___enumerations_ga3507ee60c1ffe1909096e2080dd2a05d.html#ga3507ee60c1ffe1909096e2080dd2a05d).

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

## <a name="enqueue-the-capture-and-pop-the-results"></a>Empiler la capture et dépiler les résultats

Le dispositif de suivi gère en interne une file d’attente d’entrée et une file d’attente de sortie pour traiter de manière asynchrone et plus efficace les captures d’Azure Kinect DK. Utilisez la fonction [k4abt_tracker_enqueue_capture()](https://microsoft.github.io/Azure-Kinect-Body-Tracking/release/1.x.x/group__btfunctions_ga093becd9bb4a63f5f4d56f58097a7b1e.html#ga093becd9bb4a63f5f4d56f58097a7b1e) pour ajouter une nouvelle capture à la file d’attente d’entrée. Utilisez la fonction [k4abt_tracker_pop_result ()](https://microsoft.github.io/Azure-Kinect-Body-Tracking/release/1.x.x/group__btfunctions_gaaf446fb1579cbbe0b6af824ee0a7458b.html#gaaf446fb1579cbbe0b6af824ee0a7458b) pour afficher un résultat de la file d’attente de sortie. L’utilisation de la valeur de délai d’attente dépend de l’application et contrôle la valeur de délai d’attente de la file d’attente.

### <a name="real-time-processing"></a>Traitement en temps réel
Utilisez ce modèle pour les applications monothread qui ont besoin de résultats en temps réel et peuvent s’accommoder de l’absence de certaines images. L’exemple `simple_3d_viewer` situé dans [GitHub Azure-Kinect-Samples](https://github.com/microsoft/Azure-Kinect-Samples) est un exemple de traitement en temps réel.

```C
k4a_wait_result_t queue_capture_result = k4abt_tracker_enqueue_capture(tracker, sensor_capture, 0);
k4a_capture_release(sensor_capture); // Remember to release the sensor capture once you finish using it
if (queue_capture_result == K4A_WAIT_RESULT_FAILED)
{
    printf("Error! Adding capture to tracker process queue failed!\n");
    break;
}

k4abt_frame_t body_frame = NULL;
k4a_wait_result_t pop_frame_result = k4abt_tracker_pop_result(tracker, &body_frame, 0);
if (pop_frame_result == K4A_WAIT_RESULT_SUCCEEDED)
{
    // Successfully popped the body tracking result. Start your processing
    ...

    k4abt_frame_release(body_frame); // Remember to release the body frame once you finish using it
}
```

### <a name="synchronous-processing"></a>Traitement synchrone
Utilisez ce modèle pour les applications qui n’ont pas besoin de résultats en temps réel ou qui ne peuvent pas s’accommoder de l’absence de certaines images.

Le débit de traitement peut être limité.

L’exemple `simple_sample.exe` situé dans [GitHub Azure-Kinect-Samples](https://github.com/microsoft/Azure-Kinect-Samples) est un exemple de traitement synchrone.

```C
k4a_wait_result_t queue_capture_result = k4abt_tracker_enqueue_capture(tracker, sensor_capture, K4A_WAIT_INFINITE);
k4a_capture_release(sensor_capture); // Remember to release the sensor capture once you finish using it
if (queue_capture_result != K4A_WAIT_RESULT_SUCCEEDED)
{
    // It should never hit timeout or error when K4A_WAIT_INFINITE is set.
    printf("Error! Adding capture to tracker process queue failed!\n");
    break;
}

k4abt_frame_t body_frame = NULL;
k4a_wait_result_t pop_frame_result = k4abt_tracker_pop_result(tracker, &body_frame, K4A_WAIT_INFINITE);
if (pop_frame_result != K4A_WAIT_RESULT_SUCCEEDED)
{
    // It should never hit timeout or error when K4A_WAIT_INFINITE is set.
    printf("Error! Popping body tracking result failed!\n");
    break;
}
// Successfully popped the body tracking result. Start your processing
...

k4abt_frame_release(body_frame); // Remember to release the body frame once you finish using it
```

## <a name="next-steps"></a>Étapes suivantes

> [!div class="nextstepaction"]
>[Accéder aux données de l’ossature du corps](access-data-body-frame.md)
