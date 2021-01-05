---
title: Enregistreur Azure Kinect DK
description: Découvrez comment enregistrer des flux de données à partir du Kit de développement logiciel (SDK) du capteur dans un fichier à l’aide de l’enregistreur Azure Kinect.
author: tesych
ms.author: tesych
ms.prod: kinect-dk
ms.date: 06/26/2019
ms.topic: conceptual
keywords: kinect, enregistrement, lecture, lecteur, matroska, mkv, flux, profondeur, rvb, caméra, couleur, imu, audio
ms.openlocfilehash: 5fb6895d4102956a991c67ffab836a26b7a3abb0
ms.sourcegitcommit: 829d951d5c90442a38012daaf77e86046018e5b9
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 10/09/2020
ms.locfileid: "85276491"
---
# <a name="azure-kinect-dk-recorder"></a>Enregistreur Azure Kinect DK

Cet article explique comment utiliser l’utilitaire de ligne de commande `k4arecorder` pour enregistrer des flux de données à partir du Kit de développement logiciel (SDK) du capteur dans un fichier.

>[!NOTE]
>L’enregistreur Azure Kinect n’enregistre pas de signal audio.

## <a name="recorder-options"></a>Options de l’enregistreur

Le `k4arecorder` possède différents arguments de ligne de commande pour spécifier le fichier de sortie et les modes d’enregistrement.

Les enregistrements sont stockés au [format .mkv de Matroska](record-file-format.md). L’enregistrement utilise plusieurs pistes vidéo pour la couleur et la profondeur, ainsi que des informations supplémentaires telles que l’étalonnage et les métadonnées de la caméra.

```console
k4arecorder [options] output.mkv

 Options:
  -h, --help              Prints this help
  --list                  List the currently connected K4A devices
  --device                Specify the device index to use (default: 0)
  -l, --record-length     Limit the recording to N seconds (default: infinite)
  -c, --color-mode        Set the color sensor mode (default: 1080p), Available options:
                            3072p, 2160p, 1536p, 1440p, 1080p, 720p, 720p_NV12, 720p_YUY2, OFF
  -d, --depth-mode        Set the depth sensor mode (default: NFOV_UNBINNED), Available options:
                            NFOV_2X2BINNED, NFOV_UNBINNED, WFOV_2X2BINNED, WFOV_UNBINNED, PASSIVE_IR, OFF
  --depth-delay           Set the time offset between color and depth frames in microseconds (default: 0)
                            A negative value means depth frames will arrive before color frames.
                            The delay must be less than 1 frame period.
  -r, --rate              Set the camera frame rate in Frames per Second
                            Default is the maximum rate supported by the camera modes.
                            Available options: 30, 15, 5
  --imu                   Set the IMU recording mode (ON, OFF, default: ON)
  --external-sync         Set the external sync mode (Master, Subordinate, Standalone default: Standalone)
  --sync-delay            Set the external sync delay off the master camera in microseconds (default: 0)
                            This setting is only valid if the camera is in Subordinate mode.
  -e, --exposure-control  Set manual exposure value (-11 to 1) for the RGB camera (default: auto exposure)
```

## <a name="record-files"></a>Fichiers d’enregistrement

Exemple 1. Enregistrer la profondeur en mode NFOV sans compartimentation (640x576), RVB 1080p à 30 i/s avec IMU.
Appuyez sur les touches **CTRL-C** pour arrêter l’enregistrement.

```
k4arecorder.exe output.mkv
```

Exemple 2. Enregistrer WFOV sans compartimentation (1 MP), RVB 3072p à 15 i/s sans IMU, pendant 10 secondes.

```
k4arecorder.exe -d WFOV_UNBINNED -c 3072p -r 15 -l 10 --imu OFF output.mkv
```

Exemple 3. Enregistrer WFOV 2x2 avec compartimentation à 30 i/s pendant 5 secondes, puis sauvegarder dans le fichier output.mkv.

```
k4arecorder.exe -d WFOV_2X2BINNED -c OFF --imu OFF -l 5 output.mkv
```

>[!TIP]
>Vous pouvez utiliser la [visionneuse Azure Kinect](azure-kinect-viewer.md) pour configurer les contrôles de caméra RVB avant l’enregistrement (par exemple, pour régler la balance des blancs manuellement).

## <a name="verify-recording"></a>Vérifier l’enregistrement

Vous pouvez ouvrir le fichier output.mkv avec la [visionneuse Azure Kinect](azure-kinect-viewer.md).

Pour extraire des pistes ou afficher les informations du fichier, des outils tels que `mkvinfo` sont disponibles dans le kit de ressources [MKVToolNix](https://mkvtoolnix.download/).

## <a name="next-steps"></a>Étapes suivantes

[Utiliser l’enregistreur avec des unités synchronisées externes](record-external-synchronized-units.md)