---
title: Utiliser le Kit de développement logiciel (SDK) de capteur Azure Kinect pour enregistrer un format de fichier
description: Comprenez comment utiliser le format de fichier enregistré du Kit de développement logiciel (SDK) de capteur Azure Kinect.
author: xthexder
ms.author: jawirth
ms.prod: kinect-dk
ms.date: 06/26/2019
ms.topic: reference
keywords: kinect, azure, capteur, kit de développement logiciel (sdk), profondeur, rvb, enregistrement, lecture, matroska, mkv
ms.openlocfilehash: f4fa14b0841cb76b2ba191310ecbca312d29f805
ms.sourcegitcommit: ad677fdb81f1a2a83ce72fa4f8a3a871f712599f
ms.translationtype: HT
ms.contentlocale: fr-FR
ms.lasthandoff: 12/17/2020
ms.locfileid: "97654589"
---
# <a name="use-azure-kinect-sensor-sdk-to-record-file-format"></a>Utiliser le Kit de développement logiciel (SDK) de capteur Azure Kinect pour enregistrer un format de fichier

Pour enregistrer des données de capteur, le format de conteneur Matroska (.mkv) est utilisé, ce qui permet de stocker plusieurs pistes à l’aide de toute une série de codecs. Le fichier d’enregistrement contient des pistes pour le stockage des images en couleurs, de profondeur et IR, ainsi que des IMU.

Les détails de bas niveau du format de conteneur .mkv sont disponibles sur le [site Web Matroska](https://www.matroska.org/index.html).

| Nom de piste | Format de codec                          |
|------------|---------------------------------------|
| COULEUR      | Dépendant du mode (MJPEG, NV12 ou YUY2) |
| DEPTH      | b16g (nuances de gris, big-endian 16 bit)   |
| IR         | b16g (nuances de gris, codage Big Endian 16 bite)   |
| Unités de mesure inertielle        | Structure personnalisée, consultez la [structure d’échantillon d’IMU](record-file-format.md#imu-sample-structure) ci-dessous. |

## <a name="using-third-party-tools"></a>Utilisation d’outils tiers

Vous pouvez utiliser des outils tels que `ffmpeg` ou la commande `mkvinfo` du kit de ressources [MKVToolNix](https://mkvtoolnix.download/) pour afficher et extraire des informations de fichiers d’enregistrement.

Par exemple, la commande suivante extrait la piste de profondeur sous la forme d’une séquence de png 16 bits dans le même dossier :

```
ffmpeg -i output.mkv -map 0:1 -vsync 0 depth%04d.png
```

Le paramètre `-map 0:1` extrait la piste index 1 qui, pour la plupart des enregistrements, est celle de la profondeur. Si l’enregistrement ne contient pas de piste couleurs, il convient d’utiliser `-map 0:0`.

Le paramètre `-vsync 0` force ffmpeg à extraire les images telles quelles au lieu de tenter de se conformer à une cadence de 30, 15 ou 5 i/s.

## <a name="imu-sample-structure"></a>Structure d’échantillon d’IMU

Si les données d’IMU sont extraites du fichier sans utilisation de l’API de lecture, elles ont une forme binaire.
La structure des données d’IMU est présentée ci-dessous. Tous les champs sont codés en mode Little Endian.

| Champ                        | Type     |
|------------------------------|----------|
| Horodatage de l’accéléromètre (μs) | uint64   |
| Données de l’accéléromètre (x, y, z) | float[3] |
| Horodatage du gyroscope (μs)     | uint64   |
| Données du gyroscope (x, y, z)     | float[3] |

## <a name="identifying-tracks"></a>Identification des pistes

Il peut être nécessaire d’identifier la piste Couleur, Profondeur, IR et ainsi de suite. L’identification des pistes est nécessaire lors de l’utilisation d’outils tiers pour lire un fichier Matroska.
Les numéros de piste varient en fonction du mode de la caméra et de l’ensemble des pistes activées. Des balises sont utilisées pour identifier la signification de chaque piste.

Dans la liste ci-dessous, chaque balise est attachée à un élément Matroska spécifique et peut être utilisée pour rechercher la piste ou la pièce jointe correspondante.

Ces balises sont visibles avec des outils tels que `ffmpeg` et `mkvinfo`.
La liste complète des balises figure dans la page [Enregistrement et lecture](record-playback-api.md).

| Nom de la balise             | Cible de la balise             | Valeur de la balise             |
|----------------------|------------------------|-----------------------|
| K4A_COLOR_TRACK      | Piste Couleur            | UID de piste Matroska    |
| K4A_DEPTH_TRACK      | Piste Profondeur            | UID de piste Matroska    |
| K4A_IR_TRACK         | Piste IR               | UID de piste Matroska    |
| K4A_IMU_TRACK        | Piste IMU              | UID de piste Matroska    |
| K4A_CALIBRATION_FILE | Pièce jointe d’étalonnage | Nom du fichier de pièce jointe   |

## <a name="next-steps"></a>Étapes suivantes

[Enregistrement et lecture](record-playback-api.md)
