---
title: Démarrage rapide – Créer votre première application Azure Kinect
description: Ce guide de démarrage rapide accompagne l’utilisateur d’Azure Kinect DK tout au long du processus de création d’une application.
author: tesych
ms.author: tesych
ms.prod: kinect-dk
ms.date: 06/26/2019
ms.topic: quickstart
keywords: kinect, azure, capteur, sdk, microphone, accéder aux micros, données de micro
ms.openlocfilehash: 3632145b3f3b63023e0c66e3cf99903231802edf
ms.sourcegitcommit: eb6bef1274b9e6390c7a77ff69bf6a3b94e827fc
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/05/2020
ms.locfileid: "85277788"
---
# <a name="quickstart-build-your-first-azure-kinect-application"></a>Démarrage rapide : Créer votre première application Azure Kinect

Vous débutez avec Azure Kinect DK ? Ce guide de démarrage rapide va vous aider à devenir opérationnel avec l’appareil.

Si vous n’avez pas d’abonnement Azure, créez un [compte gratuit](https://azure.microsoft.com/free/?WT.mc_id=A261C142F) avant de commencer.

Voici les fonctions qui sont abordées ici :

- [`k4a_device_get_installed_count()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gaf7d19df0f73f8e4dfaa21e1b4b719ecc.html#gaf7d19df0f73f8e4dfaa21e1b4b719ecc)
- [`k4a_device_open()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga3d4eb5dfbf4d576d4978b66ea419f113.html#ga3d4eb5dfbf4d576d4978b66ea419f113)
- [`k4a_device_get_serialnum()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga798489af207ff1c99f2285ff6b08bc22.html#ga798489af207ff1c99f2285ff6b08bc22)
- [`k4a_device_start_cameras()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gaad7a85e1e5471810262442fc4a8e217a.html#gaad7a85e1e5471810262442fc4a8e217a)
- [`k4a_device_stop_cameras()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga4fa0e0a011a7105309ad97f081a5d6b8.html#ga4fa0e0a011a7105309ad97f081a5d6b8)
- [`k4a_device_close()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga7a3931d9a690b3971caaac83b43f9423.html#ga7a3931d9a690b3971caaac83b43f9423)

## <a name="prerequisites"></a>Prérequis

1. [Configurer l’appareil Azure Kinect DK](set-up-azure-kinect-dk.md).
2. [Télécharger](sensor-sdk-download.md) et installer le SDK Capteur d’Azure Kinect.

## <a name="headers"></a>En-têtes

Le seul en-tête dont vous avez besoin est `k4a.h`. Veillez à ce que le compilateur que vous avez choisi est configuré avec les dossiers lib et include du SDK. Vous devez aussi lier les fichiers `k4a.lib` et `k4a.dll`. Consultez éventuellement [Ajouter la bibliothèque Azure Kinect à votre projet](add-library-to-project.md).

```C
#include <k4a/k4a.h>
```

## <a name="finding-an-azure-kinect-dk-device"></a>Rechercher un appareil Azure Kinect DK

Plusieurs appareils Azure Kinect DK peuvent être connectés à votre ordinateur. Nous allons commencer par déterminer leur nombre, si tant est qu’il y en ait, à l’aide de la fonction [`k4a_device_get_installed_count()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gaf7d19df0f73f8e4dfaa21e1b4b719ecc.html#gaf7d19df0f73f8e4dfaa21e1b4b719ecc). Cette fonction doit en principe fonctionner directement, sans étapes de configuration supplémentaires.

```C
uint32_t count = k4a_device_get_installed_count();
```

Dès lors que vous avez déterminé qu’un appareil est connecté à l’ordinateur, vous pouvez l’ouvrir avec [`k4a_device_open()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga3d4eb5dfbf4d576d4978b66ea419f113.html#ga3d4eb5dfbf4d576d4978b66ea419f113). Vous pouvez fournir l’index de l’appareil à ouvrir ou utiliser simplement `K4A_DEVICE_DEFAULT` pour le premier.

```C
// Open the first plugged in Kinect device
k4a_device_t device = NULL;
k4a_device_open(K4A_DEVICE_DEFAULT, &device);
```
Comme c’est généralement le cas avec la bibliothèque Azure Kinect, quand vous ouvrez un de ses composants, vous devez aussi le refermer après l’avoir utilisé. Pour cela, pensez à appeler [`k4a_device_close()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga7a3931d9a690b3971caaac83b43f9423.html#ga7a3931d9a690b3971caaac83b43f9423).

```C
k4a_device_close(device);
```

Une fois l’appareil ouvert, vous pouvez le tester pour vérifier qu’il fonctionne. C’est le cas si vous pouvez lire le numéro de série de l’appareil.

```C
// Get the size of the serial number
size_t serial_size = 0;
k4a_device_get_serialnum(device, NULL, &serial_size);

// Allocate memory for the serial, then acquire it
char *serial = (char*)(malloc(serial_size));
k4a_device_get_serialnum(device, serial, &serial_size);
printf("Opened device: %s\n", serial);
free(serial);
```

## <a name="starting-the-cameras"></a>Démarrer les caméras

Une fois que vous avez ouvert l’appareil, vous devez configurer la caméra avec un objet [`k4a_device_configuration_t`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__device__configuration__t.html). Il existe différentes options de configuration de la caméra. Choisissez les paramètres qui correspondent le mieux à votre propre scénario.

```C
// Configure a stream of 4096x3072 BRGA color data at 15 frames per second
k4a_device_configuration_t config = K4A_DEVICE_CONFIG_INIT_DISABLE_ALL;
config.camera_fps       = K4A_FRAMES_PER_SECOND_15;
config.color_format     = K4A_IMAGE_FORMAT_COLOR_BGRA32;
config.color_resolution = K4A_COLOR_RESOLUTION_3072P;

// Start the camera with the given configuration
k4a_device_start_cameras(device, &config);

// ...Camera capture and application specific code would go here...

// Shut down the camera when finished with application logic
k4a_device_stop_cameras(device);
```

## <a name="error-handling"></a>Gestion des erreurs

Dans un souci de clarté et de concision, la gestion des erreurs n’apparaît pas dans les exemples fournis. Cependant, la gestion des erreurs est toujours importante ! De nombreuses fonctions peuvent retourner un type de réussite/échec général [`k4a_result_t`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___enumerations_ga4b419a99aa2220b076a4520dc2afd1e5.html#ga4b419a99aa2220b076a4520dc2afd1e5) ou une variante plus spécifiques avec des informations détaillées comme [`k4a_wait_result_t`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___enumerations_ga44c7c0c1cfba7c879e9e2da1a869e4ee.html#ga44c7c0c1cfba7c879e9e2da1a869e4ee). Consultez la documentation ou IntelliSense pour chaque fonction de façon à vous faire une idée des messages d’erreur que vous êtes susceptible de rencontrer.

Vous pouvez utiliser les macros [`K4A_SUCCEEDED`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___macros_ga8e5b48150bc243c6052793bd830c2fcd.html#ga8e5b48150bc243c6052793bd830c2fcd) et [`K4A_FAILED`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___macros_ga7c2e32349135d008b6f836c571d434b4.html#ga7c2e32349135d008b6f836c571d434b4) pour vérifier le résultat d’une fonction. Au lieu d’ouvrir simplement un appareil Azure Kinect DK, vous pouvez protéger l’appel de fonction comme ceci :

```C
// Open the first plugged in Kinect device
k4a_device_t device = NULL;
if ( K4A_FAILED( k4a_device_open(K4A_DEVICE_DEFAULT, &device) ) )
{
    printf("Failed to open k4a device!\n");
    return;
}
```

## <a name="full-source"></a>Source complète

```C
#pragma comment(lib, "k4a.lib")
#include <k4a/k4a.h>

#include <stdio.h>
#include <stdlib.h>

int main()
{
    uint32_t count = k4a_device_get_installed_count();
    if (count == 0)
    {
        printf("No k4a devices attached!\n");
        return 1;
    }

    // Open the first plugged in Kinect device
    k4a_device_t device = NULL;
    if (K4A_FAILED(k4a_device_open(K4A_DEVICE_DEFAULT, &device)))
    {
        printf("Failed to open k4a device!\n");
        return 1;
    }

    // Get the size of the serial number
    size_t serial_size = 0;
    k4a_device_get_serialnum(device, NULL, &serial_size);

    // Allocate memory for the serial, then acquire it
    char *serial = (char*)(malloc(serial_size));
    k4a_device_get_serialnum(device, serial, &serial_size);
    printf("Opened device: %s\n", serial);
    free(serial);

    // Configure a stream of 4096x3072 BRGA color data at 15 frames per second
    k4a_device_configuration_t config = K4A_DEVICE_CONFIG_INIT_DISABLE_ALL;
    config.camera_fps       = K4A_FRAMES_PER_SECOND_15;
    config.color_format     = K4A_IMAGE_FORMAT_COLOR_BGRA32;
    config.color_resolution = K4A_COLOR_RESOLUTION_3072P;

    // Start the camera with the given configuration
    if (K4A_FAILED(k4a_device_start_cameras(device, &config)))
    {
        printf("Failed to start cameras!\n");
        k4a_device_close(device);
        return 1;
    }

    // Camera capture and application specific code would go here

    // Shut down the camera when finished with application logic
    k4a_device_stop_cameras(device);
    k4a_device_close(device);

    return 0;
}

```

## <a name="next-steps"></a>Étapes suivantes

Découvrir comment rechercher et ouvrir un appareil Azure Kinect DK à l’aide du SDK Capteur
> [!div class="nextstepaction"]
>[Rechercher et ouvrir un appareil](find-then-open-device.md)
