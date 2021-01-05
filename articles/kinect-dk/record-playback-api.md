---
title: API de lecture Azure Kinect
description: Découvrez comment utiliser le Kit de développement logiciel (SDK) de capteur Azure Kinect pour ouvrir un fichier d’enregistrement à l’aide de l’API de lecture.
author: xthexder
ms.author: jawirth
ms.prod: kinect-dk
ms.date: 06/26/2019
ms.topic: conceptual
keywords: kinect, azure, capteur, kit de développement logiciel (sdk), profondeur, rvb, enregistrement, lecture, matroska, mkv
ms.openlocfilehash: fe403f314c1df415537d090433f34627eb1249e7
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85276459"
---
# <a name="the-azure-kinect-playback-api"></a>API de lecture Azure Kinect

Le Kit de développement logiciel (SDK) de capteur fournit une API pour l’enregistrement des données de l’appareil dans un fichier Matroska (.mkv). Le format de conteneur Matroska stocke les pistes vidéo, les échantillons d’IMU et l’étalonnage des appareils. Vous pouvez générer des enregistrements à l’aide de l’utilitaire de ligne de commande [k4arecorder](record-sensor-streams-file.md) fourni. Vous pouvez également personnaliser et enregistrer des enregistrements directement à l’aide de l’API d’enregistrement.

Pour plus d’informations sur l’API d’enregistrement, consultez [`k4a_record_create()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gae14f4181e9688e710d1c80b215413831.html#gae14f4181e9688e710d1c80b215413831).

Pour plus d’informations sur les spécifications de format de fichier Matroska, consultez la page [Format de fichier d’enregistrement](record-file-format.md).

## <a name="use-the-playback-api"></a>Utiliser l’API de lecture

Vous pouvez ouvrir des fichiers d’enregistrement à l’aide de l’API de lecture. L’API de lecture donne accès aux données de capteur dans le même format que le reste du Kit de développement logiciel (SDK) du capteur.

### <a name="open-a-record-file"></a>Ouvrir un fichier d’enregistrement

Dans l’exemple suivant, nous ouvrons un enregistrement à l’aide de la commande [`k4a_playback_open()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gacb254ac941b2ab3c202ca68f4537f368.html#gacb254ac941b2ab3c202ca68f4537f368), imprimons la longueur de l’enregistrement, puis fermons le fichier à l’aide de la commande [`k4a_playback_close()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga76f415f2076f1c8c544e094a649306ff.html#ga76f415f2076f1c8c544e094a649306ff).

```C
k4a_playback_t playback_handle = NULL;
if (k4a_playback_open("recording.mkv", &playback_handle) != K4A_RESULT_SUCCEEDED)
{
    printf("Failed to open recording\n");
    return 1;
}

uint64_t recording_length = k4a_playback_get_last_timestamp_usec(playback_handle);
printf("Recording is %lld seconds long\n", recording_length / 1000000);

k4a_playback_close(playback_handle);
```

### <a name="read-captures"></a>Lire les captures

Une fois le fichier ouvert, nous pouvons commencer à lire les captures de l’enregistrement. L’exemple suivant lit chacune des captures dans le fichier.

```C
k4a_capture_t capture = NULL;
k4a_stream_result_t result = K4A_STREAM_RESULT_SUCCEEDED;
while (result == K4A_STREAM_RESULT_SUCCEEDED)
{
    result = k4a_playback_get_next_capture(playback_handle, &capture);
    if (result == K4A_STREAM_RESULT_SUCCEEDED)
    {
        // Process capture here
        k4a_capture_release(capture);
    }
    else if (result == K4A_STREAM_RESULT_EOF)
    {
        // End of file reached
        break;
    }
}
if (result == K4A_STREAM_RESULT_FAILED)
{
    printf("Failed to read entire recording\n");
    return 1;
}
```

### <a name="seek-within-a-recording"></a>Rechercher dans un enregistrement

Une fois que nous avons atteint la fin du fichier, nous pouvons revenir en arrière et le relire. Vous pouvez effectuer ce processus en lisant en arrière à l’aide de la commande [`k4a_playback_get_previous_capture()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga54732e3aa0717e1ca4eb76ee385e878c.html#ga54732e3aa0717e1ca4eb76ee385e878c), mais cela peut prendre beaucoup de temps selon la durée de l’enregistrement.
Au lieu de cela, nous pouvons utiliser la fonction [`k4a_playback_seek_timestamp()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gaea748994a121543bd77f90417cf428f6.html#gaea748994a121543bd77f90417cf428f6) pour atteindre un point spécifique dans le fichier.

Dans cet exemple, nous spécifions des horodateurs en microsecondes pour rechercher plusieurs points dans le fichier.

```C
// Seek to the beginning of the file
if (k4a_playback_seek_timestamp(playback_handle, 0, K4A_PLAYBACK_SEEK_BEGIN) != K4A_RESULT_SUCCEEDED)
{
    return 1;
}

// Seek to the end of the file
if (k4a_playback_seek_timestamp(playback_handle, 0, K4A_PLAYBACK_SEEK_END) != K4A_RESULT_SUCCEEDED)
{
    return 1;
}

// Seek to 10 seconds from the start
if (k4a_playback_seek_timestamp(playback_handle, 10 * 1000000, K4A_PLAYBACK_SEEK_BEGIN) != K4A_RESULT_SUCCEEDED)
{
    return 1;
}

// Seek to 10 seconds from the end
if (k4a_playback_seek_timestamp(playback_handle, -10 * 1000000, K4A_PLAYBACK_SEEK_END) != K4A_RESULT_SUCCEEDED)
{
    return 1;
}
```

### <a name="read-tag-information"></a>Lire les Informations sur les balises

Les enregistrements peuvent également contenir diverses métadonnées, telles que le numéro de série et les versions du microprogramme de l’appareil. Ces métadonnées sont stockées dans des balises d’enregistrement, accessibles à l’aide de la fonction [`k4a_playback_get_tag()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga320f966fc89b4ba0d758f787f70d5143.html#ga320f966fc89b4ba0d758f787f70d5143).

```C
// Print the serial number of the device used to record
char serial_number[256];
size_t serial_number_size = 256;
k4a_buffer_result_t buffer_result = k4a_playback_get_tag(playback_handle, "K4A_DEVICE_SERIAL_NUMBER", &serial_number, &serial_number_size);
if (buffer_result == K4A_BUFFER_RESULT_SUCCEEDED)
{
    printf("Device serial number: %s\n", serial_number);
}
else if (buffer_result == K4A_BUFFER_RESULT_TOO_SMALL)
{
    printf("Device serial number too long.\n");
}
else
{
    printf("Tag does not exist. Device serial number was not recorded.\n");
}
```

### <a name="record-tag-list"></a>Liste de balises d’enregistrement

Vous trouverez ci-dessous la liste de toutes les balises par défaut qui peuvent être incluses dans un fichier d’enregistrement. La plupart de ces valeurs sont disponibles dans le cadre de la structure [`k4a_record_configuration_t`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__record__configuration__t.html), et peuvent être lues avec la fonction [`k4a_playback_get_record_configuration()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_gaab54a85c1f1e98d170d009042b449255.html#gaab54a85c1f1e98d170d009042b449255).

Si une balise n’existe pas, elle est supposée avoir la valeur par défaut.

| Nom de la balise                     | Valeur par défaut      | [`k4a_record_configuration_t`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__record__configuration__t.html) Champ | Notes     |
|------------------------------|--------------------|--------------------------------------|----------------------------------------------------------------------------------------------------------------|
| `K4A_COLOR_MODE`             | "OFF"              | `color_format` / `color_resolution`  | Valeurs possibles : "OFF", "MJPG_1080P", "NV12_720P", "YUY2_720P", etc.                                      |
| `K4A_DEPTH_MODE`             | "OFF"              | `depth_mode` / `depth_track_enabled` | Valeurs possibles : "OFF, "NFOV_UNBINNED", "PASSIVE_IR", etc.                                                |
| `K4A_IR_MODE`                | "OFF"              | `depth_mode` / `ir_track_enabled`    | Valeurs possibles : "OFF", "ACTIVE", "PASSIVE"                                                                    |
| `K4A_IMU_MODE`               | "OFF"              | `imu_track_enabled`                  | Valeurs possibles : "ON", "OFF"                                                                                   |
| `K4A_CALIBRATION_FILE`       | "calibration.json" | N/A                                  | Consultez [`k4a_device_get_raw_calibration()`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/group___functions_ga8c4e46642cee3115aeb0b33e2b43b24f.html#ga8c4e46642cee3115aeb0b33e2b43b24f) |
| `K4A_DEPTH_DELAY_NS`         | "0"                | `depth_delay_off_color_usec`         | Valeur stockée en nanosecondes, l’API fournit des microsecondes.                                                        |
| `K4A_WIRED_SYNC_MODE`        | "STANDALONE"       | `wired_sync_mode`                    | Valeurs possibles : "STANDALONE", "MASTER", "SUBORDINATE"                                                         |
| `K4A_SUBORDINATE_DELAY_NS`   | "0"                | `subordinate_delay_off_master_usec`  | Valeur stockée en nanosecondes, l’API fournit des microsecondes.                                                        |
| `K4A_COLOR_FIRMWARE_VERSION` | ""                 | N/A                                  | Version du microprogramme de couleur de l’appareil, par exemple « 1.x.xx »                                                            |
| `K4A_DEPTH_FIRMWARE_VERSION` | ""                 | N/A                                  | Version du microprogramme de profondeur de l’appareil, par exemple « 1.x.xx »                                                            |
| `K4A_DEVICE_SERIAL_NUMBER`   | ""                 | N/A                                  | Numéro de série de l’appareil d’enregistrement                                                                                 |
| `K4A_START_OFFSET_NS`        | "0"                | `start_timestamp_offset_usec`        | Consultez [Synchronisation d’horodatage](record-playback-api.md#timestamp-synchronization) ci-dessous.                       |
| `K4A_COLOR_TRACK`            | None               | N/A                                  | Consultez [Format de fichier d’enregistrement – Identification de pistes](record-file-format.md#identifying-tracks).                     |
| `K4A_DEPTH_TRACK`            | None               | N/A                                  | Consultez [Format de fichier d’enregistrement – Identification de pistes](record-file-format.md#identifying-tracks).                     |
| `K4A_IR_TRACK`               | None               | N/A                                  | Consultez [Format de fichier d’enregistrement – Identification de pistes](record-file-format.md#identifying-tracks).                     |
| `K4A_IMU_TRACK`              | None               | N/A                                  | Consultez [Format de fichier d’enregistrement – Identification de pistes](record-file-format.md#identifying-tracks).                     |

## <a name="timestamp-synchronization"></a>Synchronisation d’horodatage

Le format Matroska nécessite que les enregistrements commencent par un horodateur à zéro. Lors de la [synchronisation externe des caméras](record-external-synchronized-units.md), le premier horodatage de chaque appareil peut être différent de zéro.

Pour conserver les horodatages d’origine des appareils entre l’enregistrement et la lecture, le fichier stocke un décalage à appliquer aux horodatages.

La balise `K4A_START_OFFSET_NS` permet de spécifier un décalage d’horodatage afin que les fichiers puissent être resynchronisés après l’enregistrement. Vous pouvez ajouter ce décalage d’horodatage à chaque horodatage dans le fichier pour reconstruire les horodatages de l’appareil d’origine.

Le décalage de début est également disponible dans la structure [`k4a_record_configuration_t`](https://microsoft.github.io/Azure-Kinect-Sensor-SDK/master/structk4a__record__configuration__t.html).
