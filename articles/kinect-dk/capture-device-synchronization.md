---
title: Capturer la synchronisation d’appareils Azure Kinect
description: Découvrez comment synchroniser des appareils de capture Azure Kinect à l’aide du Kit de développement logiciel (SDK) Sensor Azure Kinect.
author: xthexder
ms.author: jawirth
ms.prod: kinect-dk
ms.date: 06/26/2019
ms.topic: conceptual
keywords: kinect, azure, sensor, sdk, depth, rgb, internal, external, synchronization, daisy chain, phase offset
ms.openlocfilehash: 914a29882277c11c388d3f81f0b1fe827fc2f156
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85276488"
---
# <a name="capture-azure-kinect-device-synchronization"></a>Capturer la synchronisation d’appareils Azure Kinect

Le matériel Azure Kinect permet d’aligner le temps de capture des images de couleur et de profondeur. L’alignement entre les caméras d’un même appareil est une **synchronisation interne**. L’alignement du temps de capture sur plusieurs appareils connectés est une **synchronisation externe**.

## <a name="device-internal-synchronization"></a>Synchronisation interne de l’appareil

La capture d’images entre les différentes caméras est synchronisée sur le matériel. Dans chaque [k4a_capture_t](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__capture__t.html) qui contient des images provenant du capteur de couleur et de profondeur, les horodatages des images sont alignés en fonction du mode de fonctionnement du matériel. Par défaut, les images d’une capture sont centrées sur l’exposition. La chronologie relative des captures de profondeur et de couleur peut être ajustée à l’aide du champ `depth_delay_off_color_usec` de [k4a_device_configuration_t](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__device__configuration__t.html).

## <a name="device-external-synchronization"></a>Synchronisation externe de l’appareil

Consultez [Configurer la synchronisation externe](https://support.microsoft.com/help/4494429/sync-multiple-azure-kinect-dk-devices) pour en savoir plus sur la configuration matérielle.

Le logiciel de chaque appareil connecté doit être configuré pour fonctionner en mode **principal** ou **subordonné**. Ce paramètre est configuré dans la structure [k4a_device_configuration_t](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__device__configuration__t.html).

Lorsque vous utilisez la synchronisation externe, les caméras subordonnées doivent toujours être démarrées avant la caméra principale pour que les horodatages s’alignent correctement.

### <a name="subordinate-mode"></a>Mode subordonné

```C
k4a_device_configuration_t deviceConfig;
deviceConfig.wired_sync_mode = K4A_WIRED_SYNC_MODE_SUBORDINATE
```

### <a name="master-mode"></a>Mode principal

```C
k4a_device_configuration_t deviceConfig;
deviceConfig.wired_sync_mode = K4A_WIRED_SYNC_MODE_MASTER;
```

### <a name="retrieving-synchronization-jack-state"></a>Récupération de l’état des prises de synchronisation

Pour récupérer par programmation l’état actuel des prises d’entrée et de sortie de synchronisation, utilisez la fonction [k4a_device_get_sync_jack](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga0209ac87bfd055163677321b0304e962.html#ga0209ac87bfd055163677321b0304e962).

## <a name="next-steps"></a>Étapes suivantes

Vous savez maintenant comment activer et capturer la synchronisation des appareils. Vous pouvez également consulter la rubrique suivante : 

>[!div class="nextstepaction"]
>[API d’enregistrement et de lecture du Kit de développement logiciel (SDK) Sensor Azure Kinect](record-playback-api.md)
