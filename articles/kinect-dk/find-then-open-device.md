---
title: Rechercher et ouvrir l’appareil Azure Kinect
description: Découvrez comment rechercher et ouvrir un appareil Azure Kinect à l’aide du Kit de développement logiciel (SDK) Azure Kinect Senor.
author: cedmonds
ms.author: cedmonds
ms.prod: kinect-dk
ms.date: 06/26/2019
ms.topic: conceptual
keywords: kinect, azure, capteur, kit de développement logiciel (sdk), profondeur, rvb, appareil, rechercher, ouvrir
ms.openlocfilehash: 67fc93b924d5d663bb43098969c54d1975bd5895
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85276463"
---
# <a name="find-then-open-the-azure-kinect-device"></a>Rechercher et ouvrir l’appareil Azure Kinect

Cet article décrit comment trouver, puis ouvrir votre Azure Kinect DK. Il article explique comment gérer le cas où plusieurs appareils sont connectés à votre ordinateur.

Vous pouvez également consulter l’[exemple d’énumération du Kit de développement logiciel (SDK)](https://github.com/microsoft/Azure-Kinect-Sensor-SDK/tree/develop/examples/enumerate) qui montre comment utiliser les fonctions décrites dans cet article.

Voici les fonctions qui sont abordées ici :
 * [`k4a_device_get_installed_count()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gaf7d19df0f73f8e4dfaa21e1b4b719ecc.html#gaf7d19df0f73f8e4dfaa21e1b4b719ecc)
 * [`k4a_device_open()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga3d4eb5dfbf4d576d4978b66ea419f113.html#ga3d4eb5dfbf4d576d4978b66ea419f113)
 * [`k4a_device_get_serialnum()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga798489af207ff1c99f2285ff6b08bc22.html#ga798489af207ff1c99f2285ff6b08bc22)
 * [`k4a_device_close()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga7a3931d9a690b3971caaac83b43f9423.html#ga7a3931d9a690b3971caaac83b43f9423)

## <a name="discover-the-number-of-connected-devices"></a>Découvrir le nombre d’appareils connectés

Commencez par récupérer le nombre d’appareils Azure Kinect actuellement connectés en utilisant [`k4a_device_get_installed_count()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gaf7d19df0f73f8e4dfaa21e1b4b719ecc.html#gaf7d19df0f73f8e4dfaa21e1b4b719ecc).

```C
uint32_t device_count = k4a_device_get_installed_count();

printf("Found %d connected devices:\n", device_count);
```

## <a name="open-a-device"></a>Ouvrir un appareil

Pour obtenir des informations sur un appareil ou pour y lire des données, vous devez d’abord ouvrir un descripteur de l’appareil en utilisant [`k4a_device_open()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga3d4eb5dfbf4d576d4978b66ea419f113.html#ga3d4eb5dfbf4d576d4978b66ea419f113).

```C
k4a_device_t device = NULL;

for (uint8_t deviceIndex = 0; deviceIndex < device_count; deviceIndex++)
{
    if (K4A_RESULT_SUCCEEDED != k4a_device_open(deviceIndex, &device))
    {
        printf("%d: Failed to open device\n", deviceIndex);
        continue;
    }

    ...

    k4a_device_close(device);
}
```

Le paramètre `index` de [`k4a_device_open()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga3d4eb5dfbf4d576d4978b66ea419f113.html#ga3d4eb5dfbf4d576d4978b66ea419f113) indique l’appareil à ouvrir si plusieurs appareils sont connectés. Si vous vous attendez à ce qu’un seul appareil soit connecté, vous pouvez passer un argument `K4A_DEVICE_DEFAULT` ou 0 pour indiquer le premier appareil.

Chaque fois que vous ouvrez un appareil, vous devez appeler [`k4a_device_close()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga7a3931d9a690b3971caaac83b43f9423.html#ga7a3931d9a690b3971caaac83b43f9423) lorsque vous avez fini d’utiliser le descripteur. Vous ne pouvez ouvrir aucun autre descripteur sur le même appareil tant que vous n’avez pas fermé le descripteur ouvert.

## <a name="identify-a-specific-device"></a>Identifier un appareil spécifique

L’ordre d’énumération des appareils par index ne change pas tant que vous ne connectez ou ne déconnectez pas un appareil. Pour identifier un appareil physique, vous devez utiliser son numéro de série.

Pour lire le numéro de série de l’appareil, utilisez la fonction [`k4a_device_get_serialnum()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga798489af207ff1c99f2285ff6b08bc22.html#ga798489af207ff1c99f2285ff6b08bc22) après avoir ouvert un descripteur.

Cet exemple montre comment allouer la quantité de mémoire appropriée pour stocker le numéro de série.

```C
char *serial_number = NULL;
size_t serial_number_length = 0;

if (K4A_BUFFER_RESULT_TOO_SMALL != k4a_device_get_serialnum(device, NULL, &serial_number_length))
{
    printf("%d: Failed to get serial number length\n", deviceIndex);
    k4a_device_close(device);
    device = NULL;
    continue;
}

serial_number = malloc(serial_number_length);
if (serial_number == NULL)
{
    printf("%d: Failed to allocate memory for serial number (%zu bytes)\n", deviceIndex, serial_number_length);
    k4a_device_close(device);
    device = NULL;
    continue;
}

if (K4A_BUFFER_RESULT_SUCCEEDED != k4a_device_get_serialnum(device, serial_number, &serial_number_length))
{
    printf("%d: Failed to get serial number\n", deviceIndex);
    free(serial_number);
    serial_number = NULL;
    k4a_device_close(device);
    device = NULL;
    continue;
}

printf("%d: Device \"%s\"\n", deviceIndex, serial_number);
```

## <a name="open-the-default-device"></a>Ouvrir l’appareil par défaut

Dans la plupart des applications, il n’y a qu’un seul appareil Azure Kinect DK connecté au même ordinateur. Si vous avez uniquement besoin de vous connecter au seul appareil attendu, vous pouvez appeler la fonction [`k4a_device_open()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga3d4eb5dfbf4d576d4978b66ea419f113.html#ga3d4eb5dfbf4d576d4978b66ea419f113) avec l’`index` de `K4A_DEVICE_DEFAULT` pour ouvrir le premier appareil.

```C
k4a_device_t device = NULL;
uint32_t device_count = k4a_device_get_installed_count();

if (device_count != 1)
{
    printf("Unexpected number of devices found (%d)\n", device_count);
    goto Exit;
}

if (K4A_RESULT_SUCCEEDED != k4a_device_open(K4A_DEVICE_DEFAULT, &device))
{
    printf("Failed to open device\n");
    goto Exit;
}
```

## <a name="next-steps"></a>Étapes suivantes

>[!div class="nextstepaction"]
>[Récupérer les images](retrieve-images.md)

>[!div class="nextstepaction"]
>[Récupérer des exemples IMU](retrieve-imu-samples.md)

